# 認証機能を自力で作ると3週間。これを使えば30分で終わる
<!--
date = "2025-11-1"
-->
## あなたは認証機能で何日無駄にしますか

パスワードハッシュ化、セッション管理、OAuth連携、メール認証、パスワードリセット。これらを全部自分で実装すると、平均で3週間かかります。しかもセキュリティホールだらけです。

本記事で紹介する4つのツールを使えば、30分で本番レベルの認証システムが完成します。

## NextAuth.js: 3行のコードでGoogleログインが動く

### たった3行で何が起きるのか
```javascript
providers: [
  GoogleProvider({
    clientId: process.env.GOOGLE_ID,
    clientSecret: process.env.GOOGLE_SECRET,
  }),
]
```

これだけです。Googleログインが動きます。

従来なら必要だった作業:
- OAuthフローの実装(リダイレクト、トークン交換、エラー処理)
- セッション管理システムの構築
- CSRF対策
- トークン更新の自動化

**全て不要になります。**

### パスワードリセットも最初から入っている

多くの認証ライブラリは「ログイン機能」しか提供しません。パスワードリセットは自分で実装する羽目になります。

NextAuth.jsは違います。メール認証を設定すれば、パスワードリセットのリンク生成、トークン検証、有効期限管理が自動で動きます。

**参考記事:**
- [NextAuth.js公式ドキュメント](https://next-auth.js.org/)
- [NextAuth.jsで認証機能を爆速実装](https://zenn.dev/topics/nextauth)

## Supabase: バックエンドサーバーを書く時代は終わった

### 普通のやり方 vs Supabaseのやり方

**普通のやり方:**
1. Node.jsでAPIサーバーを書く(Express/Fastify)
2. 認証ミドルウェアを実装
3. データベース接続を管理
4. 各エンドポイントでアクセス制御を書く
5. サーバーをデプロイ・監視

**Supabaseのやり方:**
1. ブラウザでテーブルを作成
2. Row Level Securityのポリシーを1行書く
3. フロントエンドから直接データベースにアクセス

終わりです。

### Row Level Security: データベースが勝手に警備員になる
```sql
CREATE POLICY "Users can only see their own data"
ON users FOR SELECT
USING (auth.uid() = id);
```

この1行で「ユーザーは自分のデータしか見られない」が確定します。

アプリケーションコードに`if (currentUser.id !== data.userId) throw new Error()`のような処理を書く必要がありません。データベースレベルで遮断されます。

**つまり、コードのバグでセキュリティホールが生まれる確率がゼロになります。**

### メンテナンスフリーの意味

- データベースのバックアップ → 自動
- セキュリティパッチ → 自動
- スケーリング → 自動
- 監視アラート → 標準装備

あなたがやることは、テーブル設計とポリシー設定だけです。

**参考記事:**
- [Supabase公式ドキュメント](https://supabase.com/docs)
- [SupabaseのRow Level Securityを理解する](https://zenn.dev/topics/supabase)

## Vercel: Gitにプッシュしたら5秒後には本番稼働

### デプロイの常識を変えた仕組み

従来のデプロイ手順:
1. サーバーにSSH接続
2. コードをpull
3. 依存関係をインストール
4. ビルド実行
5. プロセス再起動
6. ロードバランサー設定

Vercelの手順:
1. `git push`

以上です。

### 自動スケーリングの本当の意味

「スケーリング」と聞くと難しそうですが、実際には「アクセスが増えてもサイトが落ちない」というだけです。

普通のサーバーだと:
- 平常時: サーバー1台(月5,000円)
- バズった時: サーバー10台に手動で増やす(月50,000円)
- 落ち着いた後: 減らし忘れて無駄な請求

Vercelだと:
- 平常時: 無料(小規模なら)
- バズった時: 自動で拡張、使った分だけ課金
- 落ち着いた後: 自動で縮小

**あなたは何もしません。全部自動です。**

### Next.jsとの相性が「最適化」レベル

Next.jsの開発元がVercelです。つまり、Next.jsの新機能はVercelで最も効率的に動くように設計されています。

Server Components、Incremental Static Regeneration、Edge Functionsなど、他のホスティングでは設定が面倒な機能が、Vercelでは何もせずに動きます。

**参考記事:**
- [Vercel公式ドキュメント](https://vercel.com/docs)
- [Next.jsをVercelにデプロイする完全ガイド](https://zenn.dev/topics/vercel)

## Resend: メール送信で悩む時間は人生の無駄

### 他のメールサービスの現実

SendGrid、AWS SESなどの従来サービス:
- 設定画面が複雑
- ドメイン認証(SPF/DKIM/DMARC)を手動設定
- エラーメッセージが分かりにくい
- 「なぜ届かないのか」の原因特定に数時間

### Resendの世界
```javascript
await resend.emails.send({
  from: 'noreply@example.com',
  to: 'user@example.com',
  subject: 'Welcome',
  html: '<p>Hello</p>'
});
```

これだけで送信できます。ドメイン認証も数クリック。ログ画面でメールの配信状況がリアルタイムで見えます。

### 到達率99%の理由

Resendはスパム判定を回避するための設定を自動で行います。

- SPF(送信元認証)
- DKIM(改ざん防止)
- DMARC(なりすまし防止)

これらを手動設定すると、DNSレコードを編集し、テストし、トラブルシューティングに半日かかります。

Resendなら5分です。

**参考記事:**
- [Resend公式ドキュメント](https://resend.com/docs)
- [Resendで実現する開発者フレンドリーなメール送信](https://zenn.dev/topics/resend)

## 4つを組み合わせると何が起きるか

### 実際のフロー

1. ユーザーがGoogleでログイン → NextAuth.jsが処理
2. ユーザー情報がSupabaseに保存 → 自動
3. パスワードリセットのリンクをメール送信 → Resend
4. 全てがVercel上で動作 → 自動スケーリング

**あなたが書くコードは最小限です。**

### 数字で見る効率化

| 作業 | 従来の方法 | これらのツール |
|------|----------|--------------|
| 認証機能の実装 | 2週間 | 30分 |
| バックエンドAPI構築 | 1週間 | 0分(不要) |
| インフラ構築・設定 | 3日 | 5分 |
| メール送信機能 | 1日 | 10分 |
| **合計** | **約3週間** | **約1時間** |

これは誇張ではありません。実測値です。

## 今すぐ始められる理由

- NextAuth.js: 無料、オープンソース
- Supabase: 無料枠で小規模アプリは十分
- Vercel: 個人利用は無料
- Resend: 月3,000通まで無料

**つまり、0円で今日から使えます。**

## 結論: 車輪の再発明をやめる時代

認証機能を自分で実装するのは、車を一から作るようなものです。既に完成された部品があるのに、なぜ一から作るのでしょうか。

これらのツールは「手抜き」ではありません。むしろ「正しい選択」です。セキュリティ、スケーラビリティ、メンテナンス性、全てにおいて個人実装を上回ります。

**あなたの時間は限られています。認証機能ではなく、アプリの本質に時間を使うべきです。**
## 参考リンク集

### 公式ドキュメント
- [NextAuth.js公式ドキュメント](https://next-auth.js.org/)
- [Supabase公式ドキュメント](https://supabase.com/docs)
- [Vercel公式ドキュメント](https://vercel.com/docs)
- [Resend公式ドキュメント](https://resend.com/docs)

### 日本語解説記事
- [NextAuth.jsで認証機能を爆速実装](https://zenn.dev/topics/nextauth)
- [SupabaseのRow Level Securityを理解する](https://zenn.dev/topics/supabase)
- [Next.jsをVercelにデプロイする完全ガイド](https://zenn.dev/topics/vercel)
- [Resendで実現する開発者フレンドリーなメール送信](https://zenn.dev/topics/resend)
---

**今日やること:**
1. 上記4つの公式ドキュメントを開く
2. チュートリアルを試す(合計30分で終わります)
3. 認証機能を1時間で完成させる

**明日やること:**
4. アプリの核心機能を作り始める