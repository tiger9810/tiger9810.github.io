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

### コードをpush
```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/USERNAME/line-booking-bot.git
git branch -M main
git push -u origin main
```

.gitignoreに.envが含まれていることを確認すること。

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