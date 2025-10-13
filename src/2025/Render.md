# LINE Bot を Render でデプロイする手順
<!--
date = "2025-10-09"
-->


## 概要

- 所要時間: 1-2時間
- 前提条件: GitHubアカウント、LINE Developersアカウント、動作するLINE Botのコード
- 目的: ngrokを使わず常時稼働するLINE Botを構築

## 必要なもの

- GitHubアカウント
- Renderアカウント
- LINE Bot のソースコード
- LINE Channel Access Token
- LINE Channel Secret

## 1. GitHubリポジトリ作成

### リポジトリ作成

1. https://github.com にログイン
2. 右上の「+」→「New repository」
3. 以下を入力:
   - Repository name: line-booking-bot
   - Description: LINE予約管理Bot
   - Visibility: Private
4. 「Create repository」をクリック

### 必要なファイルの準備

目的: デプロイに必要な設定ファイルを作成し、Pythonバージョンと依存パッケージを明示する。

プロジェクトフォルダに以下のファイルが存在することを確認:

#### runtime.txt

目的: Python 3.13との互換性問題を回避するため、Python 3.11を指定。
```
python-3.11.9
```

#### requirements.txt

目的: 依存パッケージのバージョンを統一し、ビルドエラーを防ぐ。
```
# FastAPI関連
fastapi==0.115.0
uvicorn[standard]==0.32.0

# LINE Bot SDK
line-bot-sdk==3.6.0

# データベース関連
sqlalchemy==2.0.35
psycopg2-binary==2.9.10
alembic==1.13.3

# 環境変数管理
python-dotenv==1.0.1

# その他ユーティリティ
python-multipart==0.0.17
pydantic==2.10.2
pydantic-settings==2.6.1

# スケジューラー（リマインド機能用・後で使用）
apscheduler==3.10.4

# 開発用
pytest==8.3.3
httpx==0.27.2
```

#### .gitignore

目的: 機密情報やローカル環境固有のファイルをGitHubにアップロードしないようにする。
```
# 環境変数（重要！公開しない）
.env
.env.local

# Python関連
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
venv/
env/
ENV/
build/
dist/
*.egg-info/

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# データベース
*.db
*.sqlite
*.sqlite3

# ログ
*.log

# OS
.DS_Store
Thumbs.db

# テスト
.pytest_cache/
.coverage
htmlcov/

# その他
.env.backup
temp/
tmp/
```

#### main.py

FastAPIアプリケーションのエントリーポイント。LINE Webhookを処理する。

#### database.py

Supabase PostgreSQL接続設定。環境変数からDATABASE_URLを読み込む。

#### models.py

SQLAlchemyデータモデル定義。User、Staff、Booking、BusinessSettingsテーブル。

#### README.md

プロジェクトのセットアップガイド。

### GitHubにpush

目的: ローカルのコードをGitHubリポジトリにアップロードし、Renderからアクセスできるようにする。
```bash
# Gitの初期化
git init

# .gitignoreの確認（.envが含まれているか）
cat .gitignore

# すべてのファイルをステージング
git add .

# コミット
git commit -m "Initial commit: LINE Bot基本実装"

# GitHubリポジトリを追加（USERNAMEを自分のユーザー名に変更）
git remote add origin https://github.com/USERNAME/line-booking-bot.git

# メインブランチに変更
git branch -M main

# プッシュ
git push -u origin main
```

pushが完了したら、GitHubのリポジトリページ（https://github.com/USERNAME/line-booking-bot）で以下のファイルが存在することを確認:

- main.py
- database.py
- models.py
- requirements.txt
- runtime.txt
- .gitignore
- README.md

.envファイルが含まれていないことを必ず確認すること。

## 2. Renderアカウント作成

1. https://render.com/ にアクセス
2. 「Get Started」をクリック
3. 「Sign up with GitHub」を選択
4. GitHubアカウントで認証

## 3. Web Service作成

### サービス設定

1. Dashboard →「New +」→「Web Service」
2. GitHubリポジトリを接続
3. line-booking-bot を選択
4. 以下を入力:
   - Name: line-booking-bot
   - Region: Singapore または Oregon
   - Branch: main
   - Runtime: Python 3
   - Build Command: pip install -r requirements.txt
   - Start Command: uvicorn main:app --host 0.0.0.0 --port $PORT
   - Instance Type: Free

### 環境変数設定

「Environment Variables」で以下を追加:

| Key | Value |
|-----|-------|
| LINE_CHANNEL_ACCESS_TOKEN | LINEのアクセストークン |
| LINE_CHANNEL_SECRET | LINEのチャネルシークレット |
| DATABASE_URL | (空欄可) |

「Create Web Service」をクリックしてデプロイ開始。

## 4. デプロイ確認

### ログ確認

以下のメッセージが表示されれば成功:
```
==> Using Python version 3.11.9
==> Successfully installed ...
==> Your service is live 🎉
```

### URL取得

画面上部に表示されるURLをコピー:
```
https://line-booking-bot-xxxx.onrender.com
```

### 動作確認

ブラウザで上記URLにアクセスし、JSONレスポンスが返ることを確認。
```json
{
  "message": "LINE Booking Bot API",
  "status": "running"
}
```

## 5. LINE Webhook URL更新

1. https://developers.line.biz/console/ にアクセス
2. 該当チャネルを選択
3. 「Messaging API設定」タブを開く
4. Webhook URLを以下に変更:
```
https://your-app-url.onrender.com/webhook
```

5. 「更新」→「検証」で成功を確認

## 6. 動作テスト

LINE Botにメッセージを送信して応答を確認。

## 注意事項

- Freeプランは30分アクセスがないとスリープする
- 初回リクエストは起動に時間がかかる場合がある
- .envファイルをGitHubにpushしないこと
- 環境変数はRenderの管理画面から設定すること

## トラブルシューティング

### デプロイエラー: ビルド失敗

原因: Python 3.13との互換性問題、またはパッケージバージョンの不一致。

対処法1: runtime.txtでPython 3.11を指定

目的: 安定したPythonバージョンを使用してビルドエラーを回避。
```bash
echo "python-3.11.9" > runtime.txt
git add runtime.txt
git commit -m "Fix: Python 3.11を指定"
git push origin main
```

対処法2: requirements.txtを更新

目的: 依存パッケージを互換性のあるバージョンに統一。

上記の requirements.txt の内容をコピーして更新:
```bash
git add requirements.txt
git commit -m "Update: requirements.txtのバージョン更新"
git push origin main
```

Renderが自動で再デプロイを開始する。デプロイログで以下を確認:
```
==> Using Python version 3.11.9
==> Successfully installed ...
==> Your service is live 🎉
```

### runtime.txtに余分な文字が含まれる

症状: runtime.txtの内容に不要な文字（例: T）が混入している。

確認方法:
```bash
cat runtime.txt
```

修正方法:
```bash
echo "python-3.11.9" > runtime.txt
git add runtime.txt
git commit -m "Fix: Remove extra character"
git push origin main
```

### デプロイが失敗する

- requirements.txtの内容を確認
- Build Commandが正しいか確認
- ログでエラー内容を確認

### Webhookが応答しない

- Webhook URLが正しいか確認
- 環境変数が設定されているか確認
- Renderのログでエラーを確認

### 応答が遅い

- Freeプランのスリープ状態から復帰している可能性
- 有料プランへのアップグレードを検討

## 参考情報

- Render Documentation: https://render.com/docs
- LINE Messaging API: https://developers.line.biz/ja/docs/messaging-api/

---
<details><summary>履歴</summary>

- [2025-10-9 Thu] Renderの登録情報を追加

</details>