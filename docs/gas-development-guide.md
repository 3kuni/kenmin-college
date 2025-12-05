# Google Apps Script (GAS) ローカル開発ガイド

Cursor + Claude Code を使用した GAS 開発のベストプラクティス

---

## 概要

Google Apps Script は通常ブラウザ上のエディタで開発しますが、**clasp** を使用することでローカル環境（Cursor/VS Code）で開発できます。これにより以下のメリットが得られます：

- Git によるバージョン管理
- AI アシスタント（Claude Code）の活用
- TypeScript による型安全な開発
- ESLint/Prettier によるコード品質管理

---

## 環境構築

### 1. 前提条件

- Node.js v4.7.4 以上
- npm
- Google アカウント

### 2. clasp のインストール

```bash
npm install -g @google/clasp
```

### 3. Apps Script API の有効化

1. [Apps Script Settings](https://script.google.com/home/usersettings) にアクセス
2. 「Google Apps Script API」を **オン** にする

### 4. clasp にログイン

```bash
clasp login
```

ブラウザが開き、Google アカウントでの認証が求められます。

---

## プロジェクトのセットアップ

### 新規プロジェクトの作成

```bash
mkdir my-gas-project
cd my-gas-project
clasp create --title "My Project" --type standalone
```

`--type` オプション:
- `standalone`: スタンドアロンスクリプト
- `sheets`: スプレッドシート連携
- `docs`: ドキュメント連携
- `forms`: フォーム連携

### 既存プロジェクトのクローン

```bash
clasp clone <scriptId>
```

Script ID は GAS エディタの URL から取得できます。

---

## TypeScript での開発（推奨）

### 型定義のインストール

```bash
npm init -y
npm install -D @types/google-apps-script typescript
```

### tsconfig.json の作成

```json
{
  "compilerOptions": {
    "lib": ["esnext"],
    "target": "ES2019",
    "module": "None",
    "strict": true,
    "skipLibCheck": true,
    "experimentalDecorators": true
  },
  "include": ["src/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### 注意: Clasp 3.x の変更点

Clasp 3.x 以降は TypeScript の自動トランスパイルが廃止されました。
バンドラー（Rollup, Webpack）を使用するか、テンプレートプロジェクトを利用してください：

- [apps-script-engine-template](https://github.com/WildH0g/apps-script-engine-template)
- [clasp-typescript-template](https://github.com/tomoyanakano/clasp-typescript-template)
- [apps-script-starter](https://github.com/labnol/apps-script-starter)

---

## 基本的なワークフロー

### コードの同期

```bash
# ローカル → Google Drive にプッシュ
clasp push

# 変更を監視して自動プッシュ
clasp push -w

# Google Drive → ローカル にプル
clasp pull
```

### ブラウザでエディタを開く

```bash
clasp open
```

### デプロイメント管理

```bash
# 新しいバージョンを作成
clasp version "v1.0.0"

# デプロイ一覧
clasp deployments

# 新しいデプロイを作成
clasp deploy --versionNumber 1 --description "Initial deploy"
```

---

## プロジェクト構成例

```
my-gas-project/
├── .clasp.json          # clasp 設定
├── .claspignore         # プッシュ除外ファイル
├── appsscript.json      # GAS マニフェスト
├── package.json
├── tsconfig.json
├── src/
│   ├── Code.ts          # メインコード
│   ├── Utils.ts         # ユーティリティ
│   └── types.d.ts       # 型定義（IDE用）
└── tests/               # テストファイル
```

### .claspignore の例

```
**/**
!src/**/*.ts
!appsscript.json
tests/
node_modules/
```

---

## Cursor + Claude Code での開発Tips

### 1. 型定義ファイルでオートコンプリートを有効化

`src/types.d.ts` を作成：

```typescript
/// <reference types="google-apps-script" />
```

このファイルを開いておくことで、IDE のオートコンプリートが有効になります。
**注意**: このファイルは `.claspignore` に追加して、GAS にプッシュしないようにします。

### 2. GAS 特有の制約

- **モジュールシステム**: GAS は ES Modules をサポートしていません。すべてのコードはグローバル名前空間で動作します。
- **import/export**: バンドラーなしでは使用できません。
- **Node.js モジュール**: 直接は使用できませんが、Webpack でバンドルすることで一部使用可能。

### 3. ローカルとオンラインの同時編集を避ける

clasp を使う場合、オンラインエディタでの編集は避けてください。コンフリクトの原因になります。

### 4. Claude Code へのプロンプト例

```
GAS でスプレッドシートの A 列を取得して処理する関数を書いて。
SpreadsheetApp を使用してください。
```

```
GAS でフォーム送信時にメール通知を送るトリガー関数を作成して。
```

---

## テスト戦略

### Jasmine を使用したテスト

```bash
npm install -D jasmine @types/jasmine
```

`tests/` ディレクトリにテストを作成し、`.claspignore` で除外します。

### ローカルテスト用のモック

[gas-fakes](https://github.com/nickebbitt/gas-fakes) などを使用してローカルでテスト可能。

---

## CI/CD（GitHub Actions）

```yaml
name: Deploy GAS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Setup clasp
        run: |
          npm install -g @google/clasp
          echo '${{ secrets.CLASPRC_JSON }}' > ~/.clasprc.json

      - name: Push to GAS
        run: clasp push
```

---

## 参考リンク

- [clasp 公式ドキュメント](https://developers.google.com/apps-script/guides/clasp)
- [clasp GitHub リポジトリ](https://github.com/google/clasp)
- [TypeScript での開発ガイド](https://developers.google.com/apps-script/guides/typescript)
- [apps-script-starter テンプレート](https://github.com/labnol/apps-script-starter)
- [Cursor Forum: GAS 開発の議論](https://forum.cursor.com/t/coding-with-google-apps-script/29245)

---

## トラブルシューティング

### clasp push が失敗する

1. `clasp login` で再認証
2. Apps Script API が有効か確認
3. `.clasp.json` の scriptId が正しいか確認

### 型定義が効かない

1. `@types/google-apps-script` がインストールされているか確認
2. `tsconfig.json` の設定を確認
3. VS Code/Cursor を再起動

### デプロイ後に動作しない

1. `clasp open` でオンラインエディタを開き、エラーログを確認
2. 権限スコープが `appsscript.json` に正しく設定されているか確認
