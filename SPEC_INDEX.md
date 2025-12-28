# Barcode Genesis - 完全仕様書インデックス

**プロジェクト名**: Barcode Genesis  
**バージョン**: 1.1  
**最終更新**: 2024-12-28  
**開発期間**: 30日間（2人）

---

## 📚 仕様書一覧

### 1. コア仕様

| ファイル名 | 説明 | 対象者 |
|-----------|------|--------|
| [ROBOT_GENERATION_LOGIC.md](./ROBOT_GENERATION_LOGIC.md) | バーコードからロボットを生成するアルゴリズム | Backend開発者 |
| [BATTLE_LOGIC.md](./BATTLE_LOGIC.md) | バトルシステムの詳細ロジック | Backend開発者 |
| [CONSTANTS.md](./CONSTANTS.md) | 定数定義（レアリティ、属性、スキルなど） | 全開発者 |
| [TYPES.md](./TYPES.md) | TypeScript型定義 | 全開発者 |

### 2. 参照資料

| ファイル名 | 説明 |
|-----------|------|
| [index.html](./index.html) | 仕様書ウェブサイト（ブラウザで閲覧可能） |
| [README.md](./README.md) | プロジェクト概要 |

---

## 🎯 開発フロー

### Phase 1: 基盤構築（Day 1-7）

**Backend（Person A）**:
1. Firebase プロジェクトセットアップ
2. Firestore コレクション作成
3. `generateRobot` Cloud Function 実装
4. 認証ミドルウェア実装

**Frontend（Person B）**:
1. Vite + React + TypeScript セットアップ
2. TailwindCSS + Zustand 設定
3. Firebase SDK 統合
4. 認証UI実装

### Phase 2: コア機能（Day 8-14）

**Backend（Person A）**:
1. `matchBattle` Cloud Function 実装
2. バトルエンジン実装
3. ELOレーティング計算
4. ランキング更新 Scheduled Function

**Frontend（Person B）**:
1. バーコードスキャナー実装
2. RobotSVG コンポーネント実装
3. ロボット一覧画面
4. バトル画面（ログ再生）

### Phase 3: ゲーム性追加（Day 15-21）

**Backend（Person A）**:
1. スキルシステム完成
2. 実績システム実装
3. レベルアップ処理

**Frontend（Person B）**:
1. スキルアニメーション
2. 実績画面
3. ランキング画面

### Phase 4: ブラッシュアップ（Day 22-28）

**両者**:
1. バグ修正
2. パフォーマンス最適化
3. UI/UX改善
4. テスト作成

### Phase 5: リリース（Day 29-30）

**両者**:
1. E2Eテスト
2. 本番デプロイ
3. モニタリング設定

---

## 🔧 技術スタック

### Frontend
- React 18.2 + TypeScript 5.3
- Vite 5.0
- Zustand 4.4（状態管理）
- TailwindCSS 3.4
- Framer Motion 10.16
- Firebase SDK 10.7
- html5-qrcode 2.3

### Backend
- Firebase Cloud Functions（Node.js 20）
- Cloud Firestore
- Firebase Authentication
- Cloud Scheduler

### デプロイ
- Vercel（Frontend）
- Firebase（Backend）

---

## 📋 チェックリスト

### 実装前確認
- [ ] Firebase プロジェクト作成
- [ ] Vercel アカウント設定
- [ ] GitHub リポジトリ作成
- [ ] 環境変数設定

### 実装完了確認
- [ ] 全Cloud Functions動作
- [ ] 全画面実装完了
- [ ] バトルシステム動作
- [ ] ランキングシステム動作
- [ ] テストカバレッジ80%以上
- [ ] Lighthouse 80点以上

---

## 🚀 クイックスタート

```bash
# リポジトリクローン
git clone https://github.com/kojima1459/barcode-genesis-spec.git
cd barcode-genesis-spec

# 仕様書をブラウザで確認
open index.html
```

---

## 📞 サポート

質問や提案がある場合は、GitHub Issues でお知らせください。
