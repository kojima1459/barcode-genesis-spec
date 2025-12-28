# Barcode Genesis - 定数定義

## 1. レアリティ定義

```typescript
// shared/constants.ts

export const RARITY_LEVELS: Record<number, string> = {
  1: 'Common',
  2: 'Uncommon',
  3: 'Rare',
  4: 'Epic',
  5: 'Legendary',
};

export const RARITY_COLORS: Record<number, string> = {
  1: '#9CA3AF', // Gray
  2: '#22C55E', // Green
  3: '#3B82F6', // Blue
  4: '#A855F7', // Purple
  5: '#F59E0B', // Gold
};

export const RARITY_GLOW_INTENSITY: Record<number, number> = {
  1: 0,
  2: 0.2,
  3: 0.4,
  4: 0.6,
  5: 1.0,
};
```

---

## 2. 属性定義

```typescript
// shared/constants.ts

export const ELEMENT_TYPES: Record<number, string> = {
  1: '火',
  2: '水',
  3: '風',
  4: '土',
  5: '光',
  6: '闇',
  7: '機械',
};

export const ELEMENT_COLORS: Record<number, string> = {
  1: '#EF4444', // 火: Red
  2: '#3B82F6', // 水: Blue
  3: '#22C55E', // 風: Green
  4: '#A16207', // 土: Brown
  5: '#FBBF24', // 光: Yellow
  6: '#6B21A8', // 闇: Purple
  7: '#6B7280', // 機械: Gray
};

// 属性相性表 (攻撃側 → 防御側)
// 1.5 = 有利, 0.75 = 不利, 1.0 = 等倍
export const ELEMENT_ADVANTAGE: Record<number, Record<number, number>> = {
  1: { 1: 1.0, 2: 0.75, 3: 1.5, 4: 1.0, 5: 1.0, 6: 1.0, 7: 1.5 },  // 火
  2: { 1: 1.5, 2: 1.0, 3: 0.75, 4: 1.0, 5: 1.0, 6: 1.0, 7: 1.0 },  // 水
  3: { 1: 0.75, 2: 1.5, 3: 1.0, 4: 1.0, 5: 1.0, 6: 1.0, 7: 1.0 },  // 風
  4: { 1: 1.0, 2: 1.0, 3: 1.0, 4: 1.0, 5: 0.75, 6: 1.5, 7: 1.0 },  // 土
  5: { 1: 1.0, 2: 1.0, 3: 1.0, 4: 1.5, 5: 1.0, 6: 1.5, 7: 1.0 },  // 光
  6: { 1: 1.0, 2: 1.0, 3: 1.0, 4: 0.75, 5: 0.75, 6: 1.0, 7: 1.5 }, // 闇
  7: { 1: 0.75, 2: 1.0, 3: 1.0, 4: 1.0, 5: 1.0, 6: 0.75, 7: 1.0 }, // 機械
};
```

---

## 3. スキル定義 (MVP: 10種類)

```typescript
// shared/constants.ts

export interface SkillDefinition {
  id: number;
  name: string;
  description: string;
  type: 'attack' | 'defense' | 'heal' | 'buff' | 'debuff';
  element: number | null; // null = 全属性
  power: number;
  cooldown: number;
  targetType: 'self' | 'enemy';
  effectType: string;
  effectValue: number;
  animationType: string;
}

export const SKILL_DEFINITIONS: SkillDefinition[] = [
  // 攻撃スキル (5種類)
  {
    id: 1,
    name: 'ファイアブラスト',
    description: '炎の弾を放ち、敵に火属性ダメージを与える',
    type: 'attack',
    element: 1,
    power: 120,
    cooldown: 3,
    targetType: 'enemy',
    effectType: 'damage',
    effectValue: 1.2,
    animationType: 'blast_fire',
  },
  {
    id: 2,
    name: 'アクアスラッシュ',
    description: '水の刃で敵を切り裂き、水属性ダメージを与える',
    type: 'attack',
    element: 2,
    power: 110,
    cooldown: 2,
    targetType: 'enemy',
    effectType: 'damage',
    effectValue: 1.1,
    animationType: 'slash_water',
  },
  {
    id: 3,
    name: 'サンダーストライク',
    description: '雷を落とし、風属性ダメージを与える',
    type: 'attack',
    element: 3,
    power: 130,
    cooldown: 4,
    targetType: 'enemy',
    effectType: 'damage',
    effectValue: 1.3,
    animationType: 'strike_thunder',
  },
  {
    id: 4,
    name: 'ロックスマッシュ',
    description: '岩を投げつけ、土属性ダメージを与える',
    type: 'attack',
    element: 4,
    power: 100,
    cooldown: 2,
    targetType: 'enemy',
    effectType: 'damage',
    effectValue: 1.0,
    animationType: 'smash_rock',
  },
  {
    id: 5,
    name: 'レーザーキャノン',
    description: 'レーザーを発射し、機械属性ダメージを与える',
    type: 'attack',
    element: 7,
    power: 140,
    cooldown: 5,
    targetType: 'enemy',
    effectType: 'damage',
    effectValue: 1.4,
    animationType: 'laser_beam',
  },

  // 防御スキル (2種類)
  {
    id: 6,
    name: 'アイアンガード',
    description: '3ターンの間、防御力を30%上昇させる',
    type: 'defense',
    element: null,
    power: 0,
    cooldown: 5,
    targetType: 'self',
    effectType: 'buff_defense',
    effectValue: 0.3,
    animationType: 'shield_up',
  },
  {
    id: 7,
    name: 'バリアシールド',
    description: '次の攻撃のダメージを50%軽減する',
    type: 'defense',
    element: null,
    power: 0,
    cooldown: 4,
    targetType: 'self',
    effectType: 'damage_reduction',
    effectValue: 0.5,
    animationType: 'barrier',
  },

  // 回復スキル (2種類)
  {
    id: 8,
    name: 'リペア',
    description: '最大HPの20%を回復する',
    type: 'heal',
    element: null,
    power: 0,
    cooldown: 4,
    targetType: 'self',
    effectType: 'heal_percent',
    effectValue: 0.2,
    animationType: 'heal_green',
  },
  {
    id: 9,
    name: 'エマージェンシーリペア',
    description: '最大HPの40%を回復するが、次のターン行動不能',
    type: 'heal',
    element: null,
    power: 0,
    cooldown: 6,
    targetType: 'self',
    effectType: 'heal_percent_stun',
    effectValue: 0.4,
    animationType: 'heal_emergency',
  },

  // バフスキル (1種類)
  {
    id: 10,
    name: 'パワーチャージ',
    description: '3ターンの間、攻撃力を25%上昇させる',
    type: 'buff',
    element: null,
    power: 0,
    cooldown: 5,
    targetType: 'self',
    effectType: 'buff_attack',
    effectValue: 0.25,
    animationType: 'power_up',
  },
];
```

---

## 4. レベルシステム定義

```typescript
// shared/constants.ts

export const MAX_LEVEL = 50;

// レベルアップに必要な経験値 (累積)
export function getRequiredExperience(level: number): number {
  return Math.floor(100 * Math.pow(level, 1.5));
}

// レベルアップ時のステータス上昇率
export const LEVEL_UP_STAT_MULTIPLIER = 0.02; // 2% per level

// 経験値テーブル (レベル1-10の例)
// Level 1: 0 XP (start)
// Level 2: 100 XP
// Level 3: 283 XP
// Level 4: 520 XP
// Level 5: 800 XP
// Level 6: 1118 XP
// Level 7: 1470 XP
// Level 8: 1852 XP
// Level 9: 2263 XP
// Level 10: 2700 XP
```

---

## 5. バトル定数

```typescript
// shared/constants.ts

export const BATTLE_CONSTANTS = {
  MAX_TURNS: 30,
  MINIMUM_DAMAGE: 1,
  CRITICAL_HIT_CHANCE: 0.1, // 10%
  CRITICAL_HIT_MULTIPLIER: 1.5,
  SKILL_ACTIVATION_CHANCE: 0.3, // 30% (クールダウン中でない場合)
  TURN_DELAY_MS: 500, // アニメーション用
};

// ELOレーティング定数
export const ELO_CONSTANTS = {
  K_FACTOR: 32,
  INITIAL_RATING: 1000,
  MIN_RATING: 100,
  MAX_RATING: 3000,
};
```

---

## 6. ロボット名生成用定数

```typescript
// shared/constants.ts

export const ROBOT_NAME_PREFIXES = [
  'ZG', 'X', 'GN', 'MS', 'RX', 'VF', 'EVA', 'AC', 'LBX', 'Z'
];

export const ROBOT_NAME_SUFFIXES = [
  'Alpha', 'Beta', 'Gamma', 'Delta', 'Zeta', 
  'Omega', 'Sigma', 'Epsilon', 'Orion', 'Leo'
];
```

---

## 7. SVGパーツ数定義

```typescript
// shared/constants.ts

export const SVG_PARTS_CONFIG = {
  NUM_PARTS_PER_CATEGORY: 10,
  CATEGORIES: [
    'head',
    'face',
    'body',
    'armLeft',
    'armRight',
    'legLeft',
    'legRight',
    'backpack',
    'weapon',
    'accessory',
  ] as const,
};
```
