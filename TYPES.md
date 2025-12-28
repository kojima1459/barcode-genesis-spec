# Barcode Genesis - TypeScript型定義

## 1. ロボット関連

```typescript
// shared/types/robot.ts

export interface RobotParts {
  head: number;      // 1-10
  face: number;      // 1-10
  body: number;      // 1-10
  armLeft: number;   // 1-10
  armRight: number;  // 1-10
  legLeft: number;   // 1-10
  legRight: number;  // 1-10
  backpack: number;  // 1-10
  weapon: number;    // 1-10
  accessory: number; // 1-10
}

export interface RobotColors {
  primary: string;   // HEX color (e.g., "#FF5733")
  secondary: string; // HEX color
  accent: string;    // HEX color
  glow: string;      // HEX color
}

export interface RobotData {
  id: string;
  userId: string;
  name: string;
  sourceBarcode: string;
  
  // ステータス
  rarity: number;           // 1-5
  rarityName: string;       // "Common" | "Uncommon" | "Rare" | "Epic" | "Legendary"
  baseHp: number;
  baseAttack: number;
  baseDefense: number;
  baseSpeed: number;
  elementType: number;      // 1-7
  elementName: string;
  
  // レベル
  level: number;
  experience: number;
  experienceToNext: number;
  
  // 外観
  parts: RobotParts;
  colors: RobotColors;
  
  // スキル
  skills: number[];         // スキルIDの配列
  
  // メタデータ
  createdAt: Date;
  updatedAt: Date;
  
  // 統計
  totalBattles: number;
  totalWins: number;
  isFavorite: boolean;
}
```

---

## 2. ユーザー関連

```typescript
// shared/types/user.ts

export interface UserData {
  uid: string;
  email: string;
  username: string;
  avatarUrl: string | null;
  
  // レベル
  level: number;
  experience: number;
  experienceToNext: number;
  
  // ランキング
  rankingPoints: number;
  
  // 統計
  totalBattles: number;
  totalWins: number;
  totalLosses: number;
  totalDraws: number;
  winRate: number;
  currentWinStreak: number;
  maxWinStreak: number;
  
  // ログイン
  loginStreak: number;
  lastLoginAt: Date;
  
  // ロボット
  totalRobots: number;
  favoriteRobotId: string | null;
  
  // メタデータ
  createdAt: Date;
  updatedAt: Date;
}
```

---

## 3. バトル関連

```typescript
// shared/types/battle.ts

export type BattleWinner = 'player' | 'enemy' | 'draw';

export interface BattleLogEntry {
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

export interface BattleResult {
  id: string;
  
  // 参加者
  playerId: string;
  playerRobotId: string;
  playerRobotName: string;
  opponentId: string;
  opponentRobotId: string;
  opponentRobotName: string;
  
  // 結果
  winner: BattleWinner;
  turnCount: number;
  playerFinalHp: number;
  enemyFinalHp: number;
  
  // 報酬
  experienceGained: number;
  rankingChange: number;
  
  // ログ
  log: BattleLogEntry[];
  
  // メタデータ
  createdAt: Date;
}

export interface Buff {
  type: string;
  value: number;
  duration: number;
}
```

---

## 4. API関連

```typescript
// shared/types/api.ts

// ロボット生成
export interface GenerateRobotRequest {
  barcode: string;
}

export interface GenerateRobotResponse {
  success: boolean;
  robot?: RobotData;
  error?: string;
}

// バトルマッチング
export interface MatchBattleRequest {
  robotId: string;
}

export interface MatchBattleResponse {
  success: boolean;
  battle?: BattleResult;
  error?: string;
}

// ランキング取得
export interface GetRankingResponse {
  success: boolean;
  ranking?: RankingEntry[];
  error?: string;
}

export interface RankingEntry {
  rank: number;
  userId: string;
  username: string;
  avatarUrl: string | null;
  rankingPoints: number;
  totalWins: number;
  winRate: number;
}
```

---

## 5. Firestore ドキュメント型

```typescript
// shared/types/firestore.ts

import { Timestamp } from 'firebase/firestore';

// Firestoreに保存される形式 (DateはTimestampに変換)
export interface UserDocument {
  uid: string;
  email: string;
  username: string;
  avatarUrl: string | null;
  
  level: number;
  experience: number;
  experienceToNext: number;
  
  rankingPoints: number;
  
  totalBattles: number;
  totalWins: number;
  totalLosses: number;
  totalDraws: number;
  winRate: number;
  currentWinStreak: number;
  maxWinStreak: number;
  
  loginStreak: number;
  lastLoginAt: Timestamp;
  
  totalRobots: number;
  favoriteRobotId: string | null;
  
  createdAt: Timestamp;
  updatedAt: Timestamp;
}

export interface RobotDocument {
  id: string;
  userId: string;
  name: string;
  sourceBarcode: string;
  
  rarity: number;
  rarityName: string;
  baseHp: number;
  baseAttack: number;
  baseDefense: number;
  baseSpeed: number;
  elementType: number;
  elementName: string;
  
  level: number;
  experience: number;
  experienceToNext: number;
  
  parts: RobotParts;
  colors: RobotColors;
  skills: number[];
  
  createdAt: Timestamp;
  updatedAt: Timestamp;
  
  totalBattles: number;
  totalWins: number;
  isFavorite: boolean;
}

export interface BattleDocument {
  id: string;
  playerId: string;
  playerRobotId: string;
  playerRobotName: string;
  opponentId: string;
  opponentRobotId: string;
  opponentRobotName: string;
  
  winner: BattleWinner;
  turnCount: number;
  playerFinalHp: number;
  enemyFinalHp: number;
  
  experienceGained: number;
  rankingChange: number;
  
  log: BattleLogEntry[];
  
  createdAt: Timestamp;
}

export interface RankingDocument {
  entries: RankingEntry[];
  lastUpdated: Timestamp;
}
```

---

## 6. Zustand Store 型

```typescript
// frontend/src/stores/types.ts

export interface AuthState {
  user: UserData | null;
  isLoading: boolean;
  isAuthenticated: boolean;
  
  // Actions
  setUser: (user: UserData | null) => void;
  setLoading: (loading: boolean) => void;
  logout: () => void;
}

export interface RobotState {
  robots: RobotData[];
  selectedRobot: RobotData | null;
  isLoading: boolean;
  
  // Actions
  setRobots: (robots: RobotData[]) => void;
  addRobot: (robot: RobotData) => void;
  selectRobot: (robot: RobotData | null) => void;
  setLoading: (loading: boolean) => void;
}

export interface BattleState {
  currentBattle: BattleResult | null;
  isMatching: boolean;
  isPlaying: boolean;
  currentLogIndex: number;
  
  // Actions
  setBattle: (battle: BattleResult | null) => void;
  setMatching: (matching: boolean) => void;
  setPlaying: (playing: boolean) => void;
  nextLog: () => void;
  resetLog: () => void;
}
```

---

## 7. コンポーネント Props 型

```typescript
// frontend/src/components/types.ts

export interface RobotCardProps {
  robot: RobotData;
  onClick?: (robot: RobotData) => void;
  isSelected?: boolean;
  showStats?: boolean;
  size?: 'small' | 'medium' | 'large';
}

export interface RobotSVGProps {
  parts: RobotParts;
  colors: RobotColors;
  size?: number;
  animate?: boolean;
  glowIntensity?: number;
}

export interface BattleScreenProps {
  battle: BattleResult;
  onComplete?: () => void;
  autoPlay?: boolean;
}

export interface HPBarProps {
  current: number;
  max: number;
  color?: string;
  showLabel?: boolean;
  animate?: boolean;
}

export interface SkillBadgeProps {
  skillId: number;
  size?: 'small' | 'medium' | 'large';
  showTooltip?: boolean;
}

export interface RarityBadgeProps {
  rarity: number;
  showLabel?: boolean;
}

export interface ElementBadgeProps {
  elementType: number;
  showLabel?: boolean;
}
```
