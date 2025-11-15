# 5分でできる! Next.jsアプリを無料で世界に公開する方法

<!--
date = "2025-11-15"
-->

## あなたのアプリを今すぐ世界に公開できます

あなたが`localhost:3000`で確認しているNext.jsアプリ。それを、たった5分で`https://your-app.netlify.app`のような本物のURLにして、世界中の人にアクセスしてもらえる状態にできます。しかも無料です。

このブログでは、Next.jsアプリを公開する最も簡単な方法を、プログラミング初心者向けに解説します。

## そもそも「デプロイ」とは?

デプロイとは、あなたのパソコンの中だけで動いているアプリを、インターネット上に公開することです。

例えば:
- **デプロイ前**: あなたのパソコンでしか見れない
- **デプロイ後**: スマホでもタブレットでも、世界中のどこからでもアクセスできる

つまり、デプロイ=「自分だけのアプリ」から「みんなのアプリ」への変身です。

## なぜNetlifyを使うのか?

Netlifyは、Webアプリを公開するためのプラットフォームです。本来なら以下のような面倒な作業が必要です:

- サーバーを借りる
- サーバーの設定をする
- ドメインを取得する
- HTTPSを設定する

しかし、Netlifyはこれらをすべて自動でやってくれます。あなたがすることは「ファイルをアップロードするだけ」です。

**料金**: 個人利用なら無料。商用利用でも月19ドルから。

## 【重要】準備するもの3つ

デプロイを始める前に、以下3つを用意します。

1. **Netlifyアカウント** - 無料で作成: https://app.netlify.com
2. **Node.js 18以上** - すでにNext.jsを使っているならインストール済みのはず
3. **インターネット接続** - 当たり前ですが念のため

Node.jsのバージョン確認方法:
```bash
node -v
```

`v18.0.0`以上の数字が表示されればOKです。

---

## ステップ1: Next.jsの設定を変更する(所要時間: 1分)

Next.jsアプリをインターネット上で動かすには、「静的ファイル」という形式に変換する必要があります。

### 静的ファイルとは?

静的ファイルとは、サーバー側で特別な処理をしなくても動くファイルのことです。HTMLやCSS、JavaScriptだけで動くファイルと考えてください。

### next.config.tsを開いて2行追加

プロジェクトのルートにある`next.config.ts`というファイルを開き、以下のように変更します。
```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  output: 'export',  // この行を追加
  images: {
    unoptimized: true,  // この行を追加
  },
};

export default nextConfig;
```

**なぜこの設定が必要?**
- `output: 'export'` → Next.jsを静的ファイルに変換するため
- `images: { unoptimized: true }` → 画像の最適化機能を無効化するため(静的ファイルでは動かないため)

### 変更後、ビルドして確認

ターミナルを開き、以下のコマンドを実行します。
```bash
npm run build
```

エラーが出なければ成功です。プロジェクトに`out`という新しいフォルダが作成されているはずです。この`out`フォルダの中身が、これからアップロードするファイルです。

---

## ステップ2: デプロイ方法を選ぶ(ここが分かれ道)

Netlifyへのアップロード方法は4つあります。あなたに合った方法を選んでください。

| 方法 | 難易度 | 向いている人 | 自動更新 |
|-----|-------|------------|---------|
| **GitHub連携** | ★★★ | チーム開発をする。自動デプロイしたい | ◎ 自動 |
| **CLI** | ★★☆ | コマンド操作に抵抗がない | △ 手動 |
| **ドラッグ&ドロップ** | ★☆☆ | コマンドは苦手。とにかく簡単 | × 手動 |
| **ZIP上アップロード** | ★☆☆ | ドラッグ&ドロップと同じ | × 手動 |

**おすすめ**: 
- **本格的に開発するなら「GitHub連携」** - 一度設定すれば、コードを更新するだけで自動デプロイ
- **今すぐ試したいなら「ドラッグ&ドロップ」** - 最も簡単で即座にデプロイ可能

---

## 方法A: GitHub連携で自動デプロイ(最もプロフェッショナル)

GitHubと連携すると、コードを更新してGitHubにプッシュするだけで、自動的にNetlifyがデプロイしてくれます。チーム開発や継続的な開発に最適です。

### GitHubとは?

GitHubは、コードを保存・管理するためのオンラインサービスです。以下のような機能があります:

- コードのバージョン管理(過去の状態に戻せる)
- チームでのコード共有
- コードのバックアップ

GitHubアカウントがない場合は、https://github.com で無料登録できます。

### A-1. Gitのインストール確認

まず、あなたのパソコンにGitがインストールされているか確認します。
```bash
git --version
```

バージョンが表示されればOKです。表示されない場合は、以下からインストールしてください:

- **Windows**: https://git-scm.com/download/win
- **Mac**: ターミナルで`git`と入力すると、自動でインストールが始まります

### A-2. GitHubリポジトリを作成

1. GitHubにログイン
2. 右上の「+」アイコン → 「New repository」をクリック
3. 以下を入力:
   - **Repository name**: `my-next-app`(好きな名前でOK)
   - **Public/Private**: Publicを選択(無料アカウントの場合)
   - 他はそのまま
4. 「Create repository」をクリック

### A-3. ローカルのコードをGitHubにプッシュ

プロジェクトのフォルダで、ターミナルを開きます。

#### 初回のみ: Gitの初期設定
```bash
# Gitユーザー名を設定(GitHubのユーザー名と同じにする)
git config --global user.name "あなたのユーザー名"

# Gitメールアドレスを設定(GitHubのメールアドレスと同じにする)
git config --global user.email "your.email@example.com"
```

#### Gitリポジトリを初期化
```bash
# Gitリポジトリとして初期化
git init

# すべてのファイルをステージングエリアに追加
git add .

# 最初のコミット
git commit -m "Initial commit"

# メインブランチの名前を設定
git branch -M main

# GitHubリポジトリと連携(URLは自分のリポジトリのURLに置き換える)
git remote add origin https://github.com/あなたのユーザー名/my-next-app.git

# GitHubにプッシュ
git push -u origin main
```

**エラーが出た場合**: GitHubのパスワード入力を求められることがあります。最近のGitHubでは、パスワードではなく「Personal Access Token」が必要です。

#### Personal Access Tokenの作成方法

1. GitHub → 右上のアイコン → 「Settings」
2. 左下の「Developer settings」
3. 「Personal access tokens」 → 「Tokens (classic)」
4. 「Generate new token」 → 「Generate new token (classic)」
5. 「Note」に「Netlify deploy」など任意の名前を入力
6. 「repo」にチェックを入れる
7. 「Generate token」をクリック
8. 表示されたトークンをコピー(二度と表示されないので注意)

パスワードを求められたら、このトークンを貼り付けます。

### A-4. NetlifyとGitHubを連携

1. https://app.netlify.com にアクセス
2. 「Add new site」 → 「Import an existing project」をクリック
3. 「Deploy with GitHub」を選択
4. GitHubへの連携を承認(初回のみ)
5. リポジトリ一覧から`my-next-app`を選択
6. ビルド設定を入力:
   - **Build command**: `npm run build`
   - **Publish directory**: `out`
7. 「Deploy site」をクリック

Base directoryは空欄のままでOKです。
Base directoryとは何か
Base directoryは、Netlifyがビルドコマンド（npm run buildなど）を実行する際の「作業ディレクトリ」を指定する設定です。
通常のNext.jsプロジェクトの場合
あなたのプロジェクト（click-counter）は通常のNext.jsプロジェクトなので、Base directoryは空欄にしてください。
なぜ空欄でいいのか？

GitHubリポジトリのルート（最上位階層）にpackage.jsonがある
プロジェクトがサブディレクトリに分かれていない
Netlifyはリポジトリのルートからビルドコマンドを実行すれば良い

Base directoryを設定する必要があるケース
以下のような特殊な構成の場合のみ、Base directoryの設定が必要です：
ケース1: モノレポ構成
my-repo/
├── apps/
│   ├── web/          ← Next.jsアプリがここにある
│   │   ├── package.json
│   │   └── next.config.ts
│   └── mobile/
└── package.json
この場合、Base directoryに apps/web と入力します。
ケース2: プロジェクトがサブディレクトリにある
my-repo/
├── docs/
├── frontend/         ← Next.jsアプリがここにある
│   ├── package.json
│   └── next.config.ts
└── README.md
この場合、Base directoryに frontend と入力します。
デプロイが開始されます。1〜2分で完了し、URLが表示されます。

### A-5. 自動デプロイの仕組み

これで設定完了です。今後、コードを変更してGitHubにプッシュすると、自動的にNetlifyがデプロイしてくれます。
```bash
# コードを変更した後
git add .
git commit -m "Update: 機能を追加"
git push
```

これだけで、最新版が自動的に公開されます。

### GitHub連携のメリット

- **自動デプロイ**: コードをプッシュするだけで自動更新
- **バージョン管理**: 過去のコードに戻せる
- **チーム開発**: 複数人で同じコードを編集できる
- **プレビュー環境**: プルリクエストごとにプレビューURLが作成される

### GitHub連携のデメリット

- **学習コスト**: Gitとコマンドの基礎知識が必要
- **初期設定**: 最初の設定がやや複雑

---

## 方法B: コマンドで一発デプロイ(Netlify CLI)

コマンドライン操作に慣れている人向けです。一度設定すれば、次回からは1コマンドでデプロイできます。

### B-1. Netlify CLIをインストール

ターミナルで以下を実行:
```bash
npm install -g netlify-cli
```

`-g`は「グローバルインストール」の意味で、どのフォルダからでもNetlifyコマンドを使えるようになります。

### B-2. Netlifyにログイン
```bash
netlify login
```

ブラウザが自動で開き、Netlifyのログイン画面が表示されます。ログインすると、ターミナルに「You are now logged in」と表示されます。

### B-3. 初期設定(初回のみ)

プロジェクトフォルダで以下を実行:
```bash
netlify init
```

いくつか質問されるので、以下のように答えます:

1. **Create & configure a new site** を選択
2. **Team**: 個人アカウントを選択
3. **Site name**: 好きな名前を入力(例: `my-first-app`)。空欄でもOK
4. **Build command**: `npm run build`
5. **Directory to deploy**: `out`
6. **Functions folder**: Enterでスキップ

### B-4. デプロイ実行
```bash
netlify deploy --prod
```

完了すると、以下のようなURLが表示されます:
```
✔ Deploy is live!
Website URL: https://my-first-app.netlify.app
```

このURLをブラウザで開けば、あなたのアプリが表示されます。

### 2回目以降は超簡単

コードを変更した後:
```bash
npm run build
netlify deploy --prod
```

これだけで最新版が公開されます。

---

## 方法C: ドラッグ&ドロップで秒速デプロイ(最も簡単)

コマンド操作が苦手な人はこちら。マウスだけで完結します。

### C-1. ビルドする

ターミナルで:
```bash
npm run build
```

これで`out`フォルダが作成されます。

### C-2. Netlify Dropを開く

ブラウザで以下にアクセス:

https://app.netlify.com/drop

ログインしていない場合は、Netlifyアカウントでログインします。

### C-3. ファイルをドラッグ&ドロップ

1. `out`フォルダを開く
2. フォルダ内の**すべてのファイル**を選択
3. ブラウザのNetlify Dropエリアにドラッグ&ドロップ

アップロードが始まり、数秒で完了します。完了すると、以下のようなURLが表示されます:
```
https://random-name-12345.netlify.app
```

### 注意点

この方法は、更新のたびに手動でアップロードが必要です。サイト名もランダムになります(後で変更可能)。

---

## 方法D: ZIPファイルでアップロード

ドラッグ&ドロップとほぼ同じですが、ZIPにまとめてアップロードします。

### D-1. ビルドしてZIP化
```bash
npm run build
```

`out`フォルダができたら、以下の手順でZIPにします:

- **Windows**: `out`フォルダを右クリック → 「送る」 → 「圧縮(zip形式)フォルダー」
- **Mac**: `out`フォルダを右クリック → 「"out"を圧縮」

### D-2. Netlifyダッシュボードでアップロード

1. https://app.netlify.com にアクセス
2. 「Add new site」 → 「Deploy manually」
3. 作成したZIPファイルをドラッグ&ドロップ

自動で展開され、デプロイされます。

---

## 各デプロイ方法の比較表

| 項目 | GitHub連携 | CLI | ドラッグ&ドロップ | ZIP |
|-----|----------|-----|----------------|-----|
| **初期設定の難易度** | 高 | 中 | 低 | 低 |
| **2回目以降の手間** | 最小 | 小 | 大 | 大 |
| **自動デプロイ** | ○ | × | × | × |
| **チーム開発** | ○ | △ | × | × |
| **バージョン管理** | ○ | × | × | × |
| **プレビュー環境** | ○ | × | × | × |

**結論**: 
- 継続的に開発するなら **GitHub連携**
- 今すぐ試したいだけなら **ドラッグ&ドロップ**

---

## デプロイ後にやること

### サイト名を変更する

Netlifyが自動生成した名前(例: `random-name-12345`)を変更できます。

1. Netlifyダッシュボードを開く
2. 「Site settings」をクリック
3. 「Site details」 → 「Change site name」
4. 新しい名前を入力(例: `my-counter-app`)

変更後、URLは`https://my-counter-app.netlify.app`になります。

### 独自ドメインを設定する(オプション)

`example.com`のような独自ドメインを持っている場合:

1. 「Site settings」 → 「Domain management」
2. 「Add custom domain」
3. ドメイン名を入力
4. DNS設定の指示に従う

DNS設定は、ドメインを購入したサービス(お名前.comなど)で行います。

---

## うまくいかない時のチェックリスト

### ビルドが失敗する

**症状**: `npm run build`でエラーが出る

**解決策**:
```bash
# 1. node_modulesを削除して再インストール
rm -rf node_modules package-lock.json
npm install

# 2. キャッシュを削除
rm -rf .next out

# 3. npmのキャッシュをクリア
npm cache clean --force
```

### デプロイ後に真っ白なページが表示される

**症状**: URLにアクセスしても404エラーまたは真っ白

**確認すること**:
- `out`フォルダに`index.html`が存在するか
- デプロイしたフォルダ名が`out`になっているか
- `next.config.ts`の設定が正しいか

### 画像が表示されない

**症状**: テキストは表示されるが、画像だけ表示されない

**確認すること**:
- `next.config.ts`に`images: { unoptimized: true }`があるか
- 画像のパスが`/images/photo.jpg`のように`/`から始まっているか

### GitHubへのプッシュが失敗する

**症状**: `git push`でエラーが出る

**よくある原因と解決策**:

1. **認証エラー**: Personal Access Tokenを使用する(上記参照)
2. **リモートリポジトリが設定されていない**:
```bash
   git remote -v  # 設定を確認
   git remote add origin https://github.com/ユーザー名/リポジトリ名.git
```
3. **ブランチ名が違う**:
```bash
   git branch -M main  # ブランチ名をmainに統一
```

### Netlifyのビルドが失敗する(GitHub連携時)

**症状**: GitHubにプッシュしたがNetlifyでビルドエラー

**確認すること**:
1. Netlifyダッシュボード → 「Deploys」タブでエラーログを確認
2. Build commandが`npm run build`になっているか
3. Publish directoryが`out`になっているか
4. ローカルで`npm run build`が成功するか

### Netlify CLIのログインができない

**症状**: `netlify login`を実行してもログインできない

**解決策**:
- ブラウザのポップアップブロッカーを無効化
- 以下のコマンドを試す:
```bash
netlify login --browser
```

---

## 更新する時はどうする?

アプリを修正した後、再度デプロイする方法です。

### GitHub連携を使った場合
```bash
# コードを変更後
git add .
git commit -m "Update: 変更内容の説明"
git push
```

これだけで自動的にNetlifyがデプロイしてくれます。デプロイの進行状況は、Netlifyダッシュボードの「Deploys」タブで確認できます。

### Netlify CLIを使った場合
```bash
npm run build
netlify deploy --prod
```

### ドラッグ&ドロップを使った場合

1. コードを変更
2. `npm run build`を実行
3. 新しい`out`フォルダの中身を再度ドラッグ&ドロップ

### ZIP上アップロードを使った場合

1. コードを変更
2. `npm run build`を実行
3. `out`フォルダを再度ZIP化
4. Netlifyダッシュボードからアップロード

---

## 最終確認: デプロイ成功のチェックリスト

デプロイが完了したら、以下を確認します。

- [ ] URLにアクセスして、アプリが表示される
- [ ] ボタンやフォームなど、すべての機能が動作する
- [ ] スマートフォンでも正常に表示される
- [ ] URLが`https://`から始まっている(Netlifyは自動でHTTPSを有効化)
- [ ] 画像やスタイルが正しく表示される
- [ ] (GitHub連携の場合)コードをプッシュすると自動デプロイされる

すべてチェックできれば、デプロイ完了です。

---

## まとめ: あなたに合ったデプロイ方法

### GitHub連携がおすすめな人
- チームで開発する
- 継続的にアプリを更新する予定
- バージョン管理をしたい
- 自動デプロイしたい

**手順**: Git設定 → GitHubリポジトリ作成 → コードをプッシュ → Netlify連携

### Netlify CLIがおすすめな人
- コマンド操作に抵抗がない
- 個人開発
- 手動デプロイでも問題ない

**手順**: CLI インストール → ログイン → 初期設定 → デプロイ

### ドラッグ&ドロップがおすすめな人
- コマンド操作が苦手
- とにかく今すぐ試したい
- 一度だけデプロイできればOK

**手順**: ビルド → Netlify Dropにアクセス → ファイルをドラッグ&ドロップ

---

## 参考になるWeb記事

デプロイについてさらに詳しく知りたい方は、以下の公式ドキュメントを参照してください。

### Netlify公式
- [Netlify公式ドキュメント](https://docs.netlify.com/) - Netlifyの全機能を網羅
- [Netlify CLIリファレンス](https://cli.netlify.com/) - CLIコマンドの詳細
- [Netlify Drop](https://app.netlify.com/drop) - ドラッグ&ドロップデプロイ
- [NetlifyとGitHubの連携ガイド](https://docs.netlify.com/integrations/github/) - GitHub連携の詳細

### Next.js公式
- [Next.js静的エクスポート公式ガイド](https://nextjs.org/docs/app/building-your-application/deploying/static-exports) - `output: 'export'`の詳細説明
- [Next.jsデプロイドキュメント](https://nextjs.org/docs/deployment) - 各種プラットフォームへのデプロイ方法

### Git/GitHub
- [GitHub公式ドキュメント](https://docs.github.com/ja) - GitHubの使い方
- [Git入門](https://git-scm.com/book/ja/v2) - Gitの基礎から応用まで
- [GitHub Personal Access Tokenの作成方法](https://docs.github.com/ja/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) - 認証トークンの作成手順

### トラブルシューティング
- [Netlifyコミュニティフォーラム](https://answers.netlify.com/) - エラー解決のQ&A
- [NetlifyステータスページPage](https://www.netlifystatus.com/) - サービス障害情報

これらのリソースを活用すれば、より高度なデプロイ設定やカスタマイズが可能になります。