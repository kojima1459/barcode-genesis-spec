## 🤖 Barcode Genesis - ロボット生成アルゴリズム v1.1

**目的**: 13桁のバーコードから、決定論的かつユニークなロボットデータを生成する。

**入力**: `barcode` (string, 13桁)
**出力**: `RobotGenerationResult` (object)

---

### 1. 基本構造

```typescript
// backend/src/services/robotGenerator.ts

import { RobotData, RobotParts, RobotColors, Skill } from 'shared/types/robot';
import { ELEMENT_TYPES, RARITY_LEVELS, SKILL_DEFINITIONS } from 'shared/constants';

// バーコードから生成されるデータのインターフェース
export interface RobotGenerationResult {
  name: string;
  rarity: number;
  rarityName: string;
  baseHp: number;
  baseAttack: number;
  baseDefense: number;
  baseSpeed: number;
  elementType: number;
  elementName: string;
  parts: RobotParts;
  colors: RobotColors;
  skills: number[];
}

// メイン関数
export function generateRobotFromBarcode(barcode: string): RobotGenerationResult {
  // ... 実装 ...
}
```

---

### 2. バーコード分解

13桁のJANコードを以下の構造に分解する。

- **C1, C2**: 国コード
- **M1, M2, M3, M4, M5**: メーカーコード
- **P1, P2, P3, P4, P5**: 製品コード
- **D**: チェックデジット

```typescript
// backend/src/services/robotGenerator.ts

function decomposeBarcode(barcode: string): number[] {
  if (!/^[0-9]{13}$/.test(barcode)) {
    throw new Error('Invalid barcode format. Must be 13 digits.');
  }
  return barcode.split('').map(Number);
}

// ...
const digits = decomposeBarcode(barcode);
const [c1, c2, m1, m2, m3, m4, m5, p1, p2, p3, p4, p5, d] = digits;
```

---

### 3. レアリティ決定ロジック

メーカーコードと製品コードの一部を組み合わせてスコアを算出し、レアリティを決定する。

- **計算式**: `((メーカーコードの奇数桁の和 % 10) + (製品コードの偶数桁の和 % 10)) % 100`
- **目的**: 特定のメーカーや製品に偏らず、分散したスコアを生成する。

```typescript
// backend/src/services/robotGenerator.ts

function calculateRarity(digits: number[]): { rarity: number; rarityName: string } {
  const [c1, c2, m1, m2, m3, m4, m5, p1, p2, p3, p4, p5, d] = digits;

  const manufacturerScore = (m1 + m3 + m5) % 10;
  const productScore = (p2 + p4) % 10;
  const totalScore = (manufacturerScore * 10 + productScore) % 100;

  let rarity = 1;
  if (totalScore >= 99) rarity = 5; // 1%
  else if (totalScore >= 95) rarity = 4; // 4%
  else if (totalScore >= 85) rarity = 3; // 10%
  else if (totalScore >= 60) rarity = 2; // 25%
  else rarity = 1; // 60%

  return { rarity, rarityName: RARITY_LEVELS[rarity] };
}

// ...
const { rarity, rarityName } = calculateRarity(digits);
```

---

### 4. 基本ステータス計算

レアリティに応じて基本ステータスポイントの総量を決定し、製品コードを基に各ステータス（HP, 攻撃, 防御, 速度）に配分する。

- **ステータス総量**: `100 + (レアリティ * 25)`
- **配分比率**: 製品コードの各桁を使い、比率を生成する。

```typescript
// backend/src/services/robotGenerator.ts

function calculateBaseStats(digits: number[], rarity: number): { baseHp: number; baseAttack: number; baseDefense: number; baseSpeed: number } {
  const [c1, c2, m1, m2, m3, m4, m5, p1, p2, p3, p4, p5, d] = digits;

  const totalStatusPoints = 100 + (rarity * 25);

  // 製品コードから比率を生成
  const ratioHp = (p1 + 1) / 10;
  const ratioAttack = (p2 + 1) / 10;
  const ratioDefense = (p3 + 1) / 10;
  const ratioSpeed = (p4 + 1) / 10;
  const totalRatio = ratioHp + ratioAttack + ratioDefense + ratioSpeed;

  const hp = Math.floor(((totalStatusPoints * ratioHp) / totalRatio) * 10);
  const attack = Math.floor(((totalStatusPoints * ratioAttack) / totalRatio));
  const defense = Math.floor(((totalStatusPoints * ratioDefense) / totalRatio));
  const speed = Math.floor(((totalStatusPoints * ratioSpeed) / totalRatio));

  return {
    baseHp: Math.max(50, hp), // 最低保証
    baseAttack: Math.max(5, attack),
    baseDefense: Math.max(5, defense),
    baseSpeed: Math.max(5, speed),
  };
}

// ...
const { baseHp, baseAttack, baseDefense, baseSpeed } = calculateBaseStats(digits, rarity);
```

---

### 5. 属性決定ロジック

メーカーコードの最初の桁を使い、7種類の属性から1つを決定する。

```typescript
// backend/src/services/robotGenerator.ts

function determineElement(digits: number[]): { elementType: number; elementName: string } {
  const m1 = digits[2];
  const elementType = (m1 % 7) + 1;
  return { elementType, elementName: ELEMENT_TYPES[elementType] };
}

// ...
const { elementType, elementName } = determineElement(digits);
```

---

### 6. パーツ選択ロジック

10部位のパーツを、それぞれ異なるバーコードの桁を基に決定する。これにより、全身が同じようなパーツになることを防ぐ。

- **パーツ数**: 各部位10種類 (ID: 1-10)

```typescript
// backend/src/services/robotGenerator.ts

function selectParts(digits: number[]): RobotParts {
  const [c1, c2, m1, m2, m3, m4, m5, p1, p2, p3, p4, p5, d] = digits;
  const NUM_PARTS_PER_CATEGORY = 10;

  return {
    head: (p1 % NUM_PARTS_PER_CATEGORY) + 1,
    face: (p2 % NUM_PARTS_PER_CATEGORY) + 1,
    body: (p3 % NUM_PARTS_PER_CATEGORY) + 1,
    armLeft: (p4 % NUM_PARTS_PER_CATEGORY) + 1,
    armRight: (p5 % NUM_PARTS_PER_CATEGORY) + 1,
    legLeft: (m1 % NUM_PARTS_PER_CATEGORY) + 1,
    legRight: (m2 % NUM_PARTS_PER_CATEGORY) + 1,
    backpack: (m3 % NUM_PARTS_PER_CATEGORY) + 1,
    weapon: (m4 % NUM_PARTS_PER_CATEGORY) + 1,
    accessory: (m5 % NUM_PARTS_PER_CATEGORY) + 1,
  };
}

// ...
const parts = selectParts(digits);
```

---

### 7. カラー生成ロジック

HSL色空間を使い、調和の取れた色の組み合わせを生成する。バーコードの各部分を色相(H)、彩度(S)、輝度(L)に割り当てる。

```typescript
// backend/src/utils/color.ts

// HSL to HEX converter
export function hslToHex(h: number, s: number, l: number): string {
  l /= 100;
  const a = (s * Math.min(l, 1 - l)) / 100;
  const f = (n: number) => {
    const k = (n + h / 30) % 12;
    const color = l - a * Math.max(Math.min(k - 3, 9 - k, 1), -1);
    return Math.round(255 * color).toString(16).padStart(2, '0');
  };
  return `#${f(0)}${f(8)}${f(4)}`;
}

// backend/src/services/robotGenerator.ts
import { hslToHex } from '../utils/color';

function generateColors(digits: number[]): RobotColors {
  const [c1, c2, m1, m2, m3, m4, m5, p1, p2, p3, p4, p5, d] = digits;

  const baseHue = (m1 * 30 + p1 * 6) % 360;

  const primary = hslToHex(baseHue, 70, 50);
  const secondary = hslToHex((baseHue + 30) % 360, 60, 60);
  const accent = hslToHex((baseHue + 180) % 360, 90, 70); // 補色
  const glow = hslToHex((baseHue + 60) % 360, 100, 80);

  return { primary, secondary, accent, glow };
}

// ...
const colors = generateColors(digits);
```

---

### 8. スキル選択ロジック

レアリティに応じて習得スキル数が決まり、製品コードを基に決定論的にスキルを選択する。

- **スキル数**: `レアリティ - 1` (最大4つ)
- **スキル定義**: `shared/constants.ts` にマスタデータを定義

```typescript
// backend/src/services/robotGenerator.ts

function selectSkills(digits: number[], rarity: number): number[] {
  const [c1, c2, m1, m2, m3, m4, m5, p1, p2, p3, p4, p5, d] = digits;
  const NUM_TOTAL_SKILLS = 10; // MVPでは10種類

  const numSkills = Math.min(rarity - 1, 4);
  if (numSkills <= 0) return [];

  const skillSeed = p1 * 1 + p2 * 2 + p3 * 3 + p4 * 4 + p5 * 5;
  
  const skills: number[] = [];
  for (let i = 0; i < numSkills; i++) {
    const skillId = ((skillSeed + i * 13) % NUM_TOTAL_SKILLS) + 1;
    if (!skills.includes(skillId)) {
      skills.push(skillId);
    }
  }
  return skills;
}

// ...
const skills = selectSkills(digits, rarity);
```

---

### 9. ロボット名生成ロジック

メーカーコードと製品コードから、ユニークなモデル名を生成する。

```typescript
// backend/src/services/robotGenerator.ts

function generateRobotName(digits: number[]): string {
  const [c1, c2, m1, m2, m3, m4, m5, p1, p2, p3, p4, p5, d] = digits;

  const prefixMap = ['ZG', 'X', 'GN', 'MS', 'RX', 'VF', 'EVA', 'AC', 'LBX', 'Z'];
  const modelMap = ['Alpha', 'Beta', 'Gamma', 'Delta', 'Zeta', 'Omega', 'Sigma', 'Epsilon', 'Orion', 'Leo'];

  const prefix = prefixMap[m1];
  const modelNumber = `${m2}${m3}${m4}`;
  const modelName = modelMap[p1];

  return `${prefix}-${modelNumber} ${modelName}`;
}

// ...
const name = generateRobotName(digits);
```

---

### 10. 最終組み立て

すべての生成ロジックを統合し、最終的な`RobotGenerationResult`オブジェクトを返す。

```typescript
// backend/src/services/robotGenerator.ts

export function generateRobotFromBarcode(barcode: string): RobotGenerationResult {
  const digits = decomposeBarcode(barcode);
  
  const { rarity, rarityName } = calculateRarity(digits);
  const { baseHp, baseAttack, baseDefense, baseSpeed } = calculateBaseStats(digits, rarity);
  const { elementType, elementName } = determineElement(digits);
  const parts = selectParts(digits);
  const colors = generateColors(digits);
  const skills = selectSkills(digits, rarity);
  const name = generateRobotName(digits);

  return {
    name,
    rarity,
    rarityName,
    baseHp,
    baseAttack,
    baseDefense,
    baseSpeed,
    elementType,
    elementName,
    parts,
    colors,
    skills,
  };
}
```
