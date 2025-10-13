# LINE Bot Webhook検証エラーの解決方法
<!--
date = "2025-10-13"
-->


## 症状

LINE Developers ConsoleでWebhook検証を実行すると、タイムアウトエラーが発生する。

## 原因

Renderの無料プランは初回起動に時間がかかるため、Webhook検証時にレスポンスが間に合わない。

## 解決手順

### 1. サーバー起動を確認

目的: サーバーが完全に起動してからWebhook検証を実行する。

Render Dashboardでログを確認:

1. Render Dashboard → YourURL → Logs
2. 最新のログで以下のメッセージを確認:
```
INFO: Uvicorn running on http://0.0.0.0:10000
==> Your service is live 🎉
```

3. このメッセージが表示されてから2-3分待機

### 2. ブラウザで動作確認

目的: サーバーが正常に応答しているか確認する。

ブラウザで以下のURLにアクセス:
```
https://YourURL.onrender.com
```

以下のJSONレスポンスが表示されれば正常:
```json
{
  "message": "LINE Booking Bot API",
  "status": "running"
}
```

#### ヘルスチェックでも確認
```
https://YourURL.onrender.com/health
```

以下が表示されれば正常:
```json
{
  "status": "healthy"
}
```

### 3. Webhook検証を再実行

1. LINE Developers Console にアクセス
2. 該当チャネルを選択
3. Messaging API設定タブを開く
4. Webhook URLを確認: `https://YourURL.onrender.com/webhook`
5. 「更新」をクリック
6. 30秒待機
7. 「検証」をクリック

成功と表示されれば完了。

### 4. それでも失敗する場合

#### 方法1: Webhookの再設定

目的: Webhook設定をリセットして接続を再確立する。

1. LINE Developers Console → Messaging API設定
2. 「Webhookの利用」をOFFにする
3. 10秒待つ
4. 「Webhookの利用」をONにする
5. 再度検証を実行

#### 方法2: タイムアウト設定の追加

目的: FastAPIのミドルウェアを追加してタイムアウトを調整する。

ローカルの main.py を編集:
```python
# main.pyの app = FastAPI() の後に追加

from fastapi.middleware.trustedhost import TrustedHostMiddleware

app.add_middleware(TrustedHostMiddleware, allowed_hosts=["*"])
```

GitHubにpush:
```bash
git add main.py
git commit -m "Add: timeout configuration"
git push origin main
```

Renderが自動で再デプロイを開始する。デプロイ完了後、再度Webhook検証を実行。

## トラブルシューティング

### 502 Bad Gateway エラーが表示される

原因: サーバーがまだ起動中。

対処法: 2-3分待ってから再度アクセス。

### Webhook検証が毎回タイムアウトする

原因: Renderの無料プランはスリープ状態からの復帰に時間がかかる。

対処法:

1. ブラウザで `https://YourURL.onrender.com` にアクセス
2. JSONが表示されるまで待つ
3. 表示されたら即座にLINEでWebhook検証を実行

### デプロイは成功しているが応答がない

確認項目:

- 環境変数が正しく設定されているか確認
- Renderのログでエラーが出ていないか確認
- Start Commandが正しいか確認: `uvicorn main:app --host 0.0.0.0 --port $PORT`

## 注意事項

- Renderの無料プランは30分アクセスがないとスリープする
- スリープ状態からの復帰には30秒～1分かかる
- 初回デプロイ後は特に起動が遅い場合がある
- Webhook検証前に必ずブラウザでアクセスして起動させる

## 参考情報

- Render Documentation: https://render.com/docs/web-services#request-timeouts
- LINE Messaging API - Webhook: https://developers.line.biz/ja/docs/messaging-api/receiving-messages/

---
<details><summary>履歴</summary>

- [0000-00-00 Fri] 

</details>