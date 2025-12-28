# Barcode Genesis - バトルロジック詳細仕様書 v1.1

## 1. バトル概要

- **形式**: 1対1、非同期オートバトル
- **ターン制限**: 最大30ターン
- **勝利条件**: 相手のHPを0にする、または30ターン終了時にHP割合が高い方

---

## 2. バトルパラメータ計算

```typescript
// backend/src/services/battleEngine.ts

interface BattleRobot {
  // 基本情報
  id: string;
  name: string;
  level: number;
  elementType: number;
  
  // バトル用ステータス (レベル補正済み)
  maxHp: number;
  currentHp: number;
  attack: number;
  defense: number;
  speed: number;
  
  // スキル
  skills: number[];
  skillCooldowns: Record<number, number>;
  
  // バフ/デバフ
  buffs: Buff[];
}

function initializeBattleRobot(robot: RobotDocument): BattleRobot {
  const levelMultiplier = 1 + (robot.level - 1) * 0.02;
  
  return {
    id: robot.id,
    name: robot.name,
    level: robot.level,
    elementType: robot.elementType,
    
    maxHp: Math.floor(robot.baseHp * levelMultiplier * 10),
    currentHp: Math.floor(robot.baseHp * levelMultiplier * 10),
    attack: Math.floor(robot.baseAttack * levelMultiplier),
    defense: Math.floor(robot.baseDefense * levelMultiplier),
    speed: Math.floor(robot.baseSpeed * levelMultiplier),
    
    skills: robot.skills,
    skillCooldowns: {},
    buffs: [],
  };
}
```

---

## 3. ターン進行ロジック

```typescript
// backend/src/services/battleEngine.ts

interface BattleState {
  turn: number;
  player: BattleRobot;
  enemy: BattleRobot;
  log: BattleLogEntry[];
  isFinished: boolean;
  winner: 'player' | 'enemy' | 'draw' | null;
}

interface BattleLogEntry {
  turn: number;
  actor: 'player' | 'enemy';
  action: 'attack' | 'skill' | 'heal' | 'buff' | 'debuff';
  skillId?: number;
  skillName?: string;
  damage?: number;
  heal?: number;
  effect?: string;
  isCritical?: boolean;
  elementModifier?: number;
  targetHpBefore: number;
  targetHpAfter: number;
}

function executeTurn(state: BattleState): void {
  state.turn++;
  
  // 行動順決定 (速度が高い方が先攻)
  const [first, second] = state.player.speed >= state.enemy.speed
    ? ['player', 'enemy'] as const
    : ['enemy', 'player'] as const;
  
  // 先攻の行動
  executeAction(state, first);
  if (state.isFinished) return;
  
  // 後攻の行動
  executeAction(state, second);
  if (state.isFinished) return;
  
  // ターン終了処理
  processEndOfTurn(state);
  
  // 勝敗判定
  checkBattleEnd(state);
}
```

---

## 4. アクション実行ロジック

```typescript
// backend/src/services/battleEngine.ts

function executeAction(state: BattleState, actor: 'player' | 'enemy'): void {
  const attacker = state[actor];
  const defender = actor === 'player' ? state.enemy : state.player;
  
  // スキル発動判定
  const availableSkill = getAvailableSkill(attacker);
  
  if (availableSkill && Math.random() < BATTLE_CONSTANTS.SKILL_ACTIVATION_CHANCE) {
    executeSkill(state, actor, availableSkill, attacker, defender);
  } else {
    executeNormalAttack(state, actor, attacker, defender);
  }
}

function getAvailableSkill(robot: BattleRobot): SkillDefinition | null {
  for (const skillId of robot.skills) {
    const cooldown = robot.skillCooldowns[skillId] || 0;
    if (cooldown <= 0) {
      return SKILL_DEFINITIONS.find(s => s.id === skillId) || null;
    }
  }
  return null;
}
```

---

## 5. 通常攻撃ダメージ計算

```typescript
// backend/src/services/battleEngine.ts

function executeNormalAttack(
  state: BattleState,
  actor: 'player' | 'enemy',
  attacker: BattleRobot,
  defender: BattleRobot
): void {
  const hpBefore = defender.currentHp;
  
  // 基本ダメージ計算
  let damage = Math.max(
    BATTLE_CONSTANTS.MINIMUM_DAMAGE,
    attacker.attack - defender.defense * 0.5
  );
  
  // 属性相性補正
  const elementModifier = ELEMENT_ADVANTAGE[attacker.elementType][defender.elementType];
  damage = Math.floor(damage * elementModifier);
  
  // クリティカル判定
  const isCritical = Math.random() < BATTLE_CONSTANTS.CRITICAL_HIT_CHANCE;
  if (isCritical) {
    damage = Math.floor(damage * BATTLE_CONSTANTS.CRITICAL_HIT_MULTIPLIER);
  }
  
  // バフ/デバフ補正
  damage = applyBuffsToAttack(attacker, damage);
  damage = applyBuffsToDefense(defender, damage);
  
  // ダメージ適用
  defender.currentHp = Math.max(0, defender.currentHp - damage);
  
  // ログ記録
  state.log.push({
    turn: state.turn,
    actor,
    action: 'attack',
    damage,
    isCritical,
    elementModifier,
    targetHpBefore: hpBefore,
    targetHpAfter: defender.currentHp,
  });
  
  // 勝敗判定
  if (defender.currentHp <= 0) {
    state.isFinished = true;
    state.winner = actor;
  }
}
```

---

## 6. スキル実行ロジック

```typescript
// backend/src/services/battleEngine.ts

function executeSkill(
  state: BattleState,
  actor: 'player' | 'enemy',
  skill: SkillDefinition,
  attacker: BattleRobot,
  defender: BattleRobot
): void {
  const hpBefore = skill.targetType === 'self' 
    ? attacker.currentHp 
    : defender.currentHp;
  
  // クールダウン設定
  attacker.skillCooldowns[skill.id] = skill.cooldown;
  
  let damage = 0;
  let heal = 0;
  let effect = '';
  
  switch (skill.type) {
    case 'attack':
      damage = calculateSkillDamage(attacker, defender, skill);
      defender.currentHp = Math.max(0, defender.currentHp - damage);
      break;
      
    case 'heal':
      heal = Math.floor(attacker.maxHp * skill.effectValue);
      attacker.currentHp = Math.min(attacker.maxHp, attacker.currentHp + heal);
      break;
      
    case 'buff':
    case 'defense':
      effect = skill.effectType;
      attacker.buffs.push({
        type: skill.effectType,
        value: skill.effectValue,
        duration: 3,
      });
      break;
      
    case 'debuff':
      effect = skill.effectType;
      defender.buffs.push({
        type: skill.effectType,
        value: -skill.effectValue,
        duration: 3,
      });
      break;
  }
  
  // ログ記録
  state.log.push({
    turn: state.turn,
    actor,
    action: 'skill',
    skillId: skill.id,
    skillName: skill.name,
    damage: damage > 0 ? damage : undefined,
    heal: heal > 0 ? heal : undefined,
    effect: effect || undefined,
    targetHpBefore: hpBefore,
    targetHpAfter: skill.targetType === 'self' 
      ? attacker.currentHp 
      : defender.currentHp,
  });
  
  // 勝敗判定
  if (defender.currentHp <= 0) {
    state.isFinished = true;
    state.winner = actor;
  }
}

function calculateSkillDamage(
  attacker: BattleRobot,
  defender: BattleRobot,
  skill: SkillDefinition
): number {
  // 基本ダメージ = 攻撃力 * スキル倍率
  let damage = attacker.attack * skill.effectValue;
  
  // 防御力軽減
  damage = Math.max(BATTLE_CONSTANTS.MINIMUM_DAMAGE, damage - defender.defense * 0.3);
  
  // 属性相性 (スキルの属性を使用)
  if (skill.element) {
    const elementModifier = ELEMENT_ADVANTAGE[skill.element][defender.elementType];
    damage = damage * elementModifier;
  }
  
  return Math.floor(damage);
}
```

---

## 7. ターン終了処理

```typescript
// backend/src/services/battleEngine.ts

function processEndOfTurn(state: BattleState): void {
  // クールダウン減少
  decreaseCooldowns(state.player);
  decreaseCooldowns(state.enemy);
  
  // バフ/デバフ持続時間減少
  processBuffDuration(state.player);
  processBuffDuration(state.enemy);
}

function decreaseCooldowns(robot: BattleRobot): void {
  for (const skillId in robot.skillCooldowns) {
    if (robot.skillCooldowns[skillId] > 0) {
      robot.skillCooldowns[skillId]--;
    }
  }
}

function processBuffDuration(robot: BattleRobot): void {
  robot.buffs = robot.buffs
    .map(buff => ({ ...buff, duration: buff.duration - 1 }))
    .filter(buff => buff.duration > 0);
}
```

---

## 8. 勝敗判定

```typescript
// backend/src/services/battleEngine.ts

function checkBattleEnd(state: BattleState): void {
  // HP 0 による勝敗
  if (state.player.currentHp <= 0) {
    state.isFinished = true;
    state.winner = 'enemy';
    return;
  }
  if (state.enemy.currentHp <= 0) {
    state.isFinished = true;
    state.winner = 'player';
    return;
  }
  
  // ターン上限による勝敗
  if (state.turn >= BATTLE_CONSTANTS.MAX_TURNS) {
    state.isFinished = true;
    
    const playerHpRatio = state.player.currentHp / state.player.maxHp;
    const enemyHpRatio = state.enemy.currentHp / state.enemy.maxHp;
    
    if (playerHpRatio > enemyHpRatio) {
      state.winner = 'player';
    } else if (enemyHpRatio > playerHpRatio) {
      state.winner = 'enemy';
    } else {
      state.winner = 'draw';
    }
  }
}
```

---

## 9. ELOレーティング計算

```typescript
// backend/src/services/battleEngine.ts

interface EloResult {
  playerNewRating: number;
  enemyNewRating: number;
  playerChange: number;
  enemyChange: number;
}

function calculateEloChange(
  playerRating: number,
  enemyRating: number,
  winner: 'player' | 'enemy' | 'draw'
): EloResult {
  const K = ELO_CONSTANTS.K_FACTOR;
  
  // 期待勝率
  const expectedPlayer = 1 / (1 + Math.pow(10, (enemyRating - playerRating) / 400));
  const expectedEnemy = 1 - expectedPlayer;
  
  // 実際の結果
  let actualPlayer: number;
  let actualEnemy: number;
  
  if (winner === 'player') {
    actualPlayer = 1;
    actualEnemy = 0;
  } else if (winner === 'enemy') {
    actualPlayer = 0;
    actualEnemy = 1;
  } else {
    actualPlayer = 0.5;
    actualEnemy = 0.5;
  }
  
  // レーティング変動
  const playerChange = Math.round(K * (actualPlayer - expectedPlayer));
  const enemyChange = Math.round(K * (actualEnemy - expectedEnemy));
  
  // 新レーティング (範囲制限)
  const playerNewRating = Math.max(
    ELO_CONSTANTS.MIN_RATING,
    Math.min(ELO_CONSTANTS.MAX_RATING, playerRating + playerChange)
  );
  const enemyNewRating = Math.max(
    ELO_CONSTANTS.MIN_RATING,
    Math.min(ELO_CONSTANTS.MAX_RATING, enemyRating + enemyChange)
  );
  
  return {
    playerNewRating,
    enemyNewRating,
    playerChange,
    enemyChange,
  };
}
```

---

## 10. 経験値計算

```typescript
// backend/src/services/battleEngine.ts

interface ExperienceResult {
  robotExpGained: number;
  userExpGained: number;
}

function calculateExperience(
  winner: 'player' | 'enemy' | 'draw',
  playerLevel: number,
  enemyLevel: number
): ExperienceResult {
  const baseExp = 50;
  const levelDiff = enemyLevel - playerLevel;
  
  // レベル差ボーナス (高レベルの敵を倒すと多くもらえる)
  const levelBonus = Math.max(0.5, 1 + levelDiff * 0.1);
  
  let robotExpGained: number;
  let userExpGained: number;
  
  if (winner === 'player') {
    robotExpGained = Math.floor(baseExp * levelBonus * 1.5);
    userExpGained = Math.floor(baseExp * levelBonus);
  } else if (winner === 'enemy') {
    robotExpGained = Math.floor(baseExp * 0.5);
    userExpGained = Math.floor(baseExp * 0.3);
  } else {
    robotExpGained = Math.floor(baseExp * 0.75);
    userExpGained = Math.floor(baseExp * 0.5);
  }
  
  return { robotExpGained, userExpGained };
}
```

---

## 11. バトルエンジン統合

```typescript
// backend/src/services/battleEngine.ts

export interface BattleResult {
  winner: 'player' | 'enemy' | 'draw';
  turnCount: number;
  log: BattleLogEntry[];
  playerFinalHp: number;
  enemyFinalHp: number;
  experienceGained: number;
  rankingChange: number;
}

export class BattleEngine {
  private state: BattleState;
  
  constructor(playerRobot: RobotDocument, enemyRobot: RobotDocument) {
    this.state = {
      turn: 0,
      player: initializeBattleRobot(playerRobot),
      enemy: initializeBattleRobot(enemyRobot),
      log: [],
      isFinished: false,
      winner: null,
    };
  }
  
  executeBattle(): BattleResult {
    while (!this.state.isFinished && this.state.turn < BATTLE_CONSTANTS.MAX_TURNS) {
      executeTurn(this.state);
    }
    
    // 最終判定
    if (!this.state.isFinished) {
      checkBattleEnd(this.state);
    }
    
    return {
      winner: this.state.winner!,
      turnCount: this.state.turn,
      log: this.state.log,
      playerFinalHp: this.state.player.currentHp,
      enemyFinalHp: this.state.enemy.currentHp,
      experienceGained: 0, // 呼び出し元で計算
      rankingChange: 0, // 呼び出し元で計算
    };
  }
}
```

---

## 12. 使用例

```typescript
// Cloud Function内での使用例

import { BattleEngine, calculateEloChange, calculateExperience } from './battleEngine';

async function matchBattle(playerId: string, playerRobotId: string) {
  // プレイヤーとロボットの取得
  const playerDoc = await db.collection('users').doc(playerId).get();
  const playerRobotDoc = await db.collection('users').doc(playerId)
    .collection('robots').doc(playerRobotId).get();
  
  // ランダムな対戦相手を取得
  const opponent = await findRandomOpponent(playerId);
  const opponentRobot = await getRandomRobotFromUser(opponent.id);
  
  // バトル実行
  const engine = new BattleEngine(playerRobotDoc.data(), opponentRobot);
  const result = engine.executeBattle();
  
  // ELO計算
  const eloResult = calculateEloChange(
    playerDoc.data().rankingPoints,
    opponent.rankingPoints,
    result.winner
  );
  
  // 経験値計算
  const expResult = calculateExperience(
    result.winner,
    playerRobotDoc.data().level,
    opponentRobot.level
  );
  
  // 結果を返す
  return {
    ...result,
    experienceGained: expResult.robotExpGained,
    rankingChange: eloResult.playerChange,
  };
}
```
