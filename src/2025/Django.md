<!-- # 腕立て伏せ100回を30日行うとどのような変化が起きたか？ -->
# DjangoでのWebサイト構築

<!--
date = "2025-06-22"
-->

### 使ったもの

- Claude AI

### やりたいこと

pythonのDjangoを使用してWebサイトを構築する

### なぜやるのか

プログラミング学習

## Djangoについて

### 学習の流れ

[Django学習のロードマップ](https://qiita.com/nossey/items/03e114a7b728a8b2ae55)のロードマップに従う

[クイックインストールガイド](https://docs.djangoproject.com/ja/2.0/intro/install/)を参考にDjangoをインストール


### memo

バージョン確認
``` zsh
python -m django --version
```

mysiteという[プロジェクトを作成](https://docs.djangoproject.com/ja/2.0/intro/tutorial01/#creating-a-project)
```zsh
django-admin startproject mysite
```

アプリケーションを作成する
アプリケーションを作るには、 manage.py と同じディレクトリに入って、このコマンドを実行
```
python manage.py startapp applicationNmae
```

作成したprojectの構成
``` python
mysite/             # プロジェクト名、自由に変更できる。
    manage.py       # コマンドラインユーティリティ
    mysite/.        # このプロジェクトに関連するPythonファイル（モジュール）をまとめて整理するためのフォルダ
        __init__.py # このディレクトリが Python パッケージであることを Python に知らせるための空のファイル
        settings.py # Django プロジェクトの設定ファイル
        urls.py     # Django プロジェクトの URL 宣言、いうなれば Django サイトにおける「目次」に相当
        wsgi.py     # プロジェクトをサーブするためのWSGI互換Webサーバーとのエントリーポイント
```

[開発用サーバーの起動](https://docs.djangoproject.com/ja/2.0/intro/tutorial01/#the-development-server)
Django 開発サーバは Python だけで書かれた軽量な Web サーバのこと
```zsh
python manage.py runserver
```
##### プロジェクトとアプリケーションの違い

アプリケーション：実際に処理を行うWebアプリケーションのこと（例：ブログシステム、データベース、投票アプリなど）
プロジェクト：特定のウェブサイト用に設定とアプリケーションをまとめたもの

##### プロジェクトとアプリケーションの関係

1つのプロジェクトに複数のアプリケーションを含めることができる
1つのアプリケーションを複数の異なるプロジェクトで再利用できる

アプリケーション＝機能、アプリケーションは、特定の機能や役割を持つ独立したコンポーネント
各アプリケーションは1つの明確な目的を持つ（単一責任の原則）

##### コマンドラインユーティリティ
コマンドラインユーティリティとは下記のサーバー起動のコマンドのようなサブコマンドを持つコマンドのまとまりのこと
```
python manage.py runserver 8080
└────┘ └───────┘ └──────┘ └──┘
Python  管理スクリプト サーバー起動 ポート番号
```

## View = Webページを表示するための処理を行う部分
##### View = Webページを表示するための処理を行う部分

Viewの基本的な流れ：
1. ユーザーがURLにアクセスする
2. DjangoがそのURLに対応するViewを呼び出す
3. Viewが処理を実行する
4. Viewがレスポンス（HTMLやJSONなど）を返す
5. ユーザーのブラウザに結果が表示される

##### viewを設定したらurls.pyを編集する
アプリケーションを始めて追加した際は、プロジェクトのurls.pyにアプリケーションのurlを登録する
ユーザーのアクセスしたurlが/app1/page1だった場合
1. プロジェクトのurls.pyによりurlの確認が行われる
2. /app1/から始まるurlだった場合、app1のurls.pyに処理を振り分ける
3. app1のurls.pyで定義されたurlにより処理を実行する
![イメージ図](./.drawio.png)

##### path()関数の引数について
- path() 関数は4つの引数を受け取る。引数のうち route と view の2つは必須で、kwargs、name の2つは省略可能
- route...route は URL パターンを含む文字列
- 

## migrate
```zsh
python manage.py migrate
```
migrate コマンドはmysite/settings.py ファイルのデータベース設定に従って必要なすべてのデータベースのテーブルを作成します
実行すると、以下のようなテーブルが自動的に作られます：
```sql
auth_user          -- ユーザー情報を保存
auth_group         -- グループ情報を保存
auth_permission    -- 権限情報を保存
django_session     -- セッション情報を保存
django_content_type -- コンテンツタイプ情報
django_admin_log   -- 管理サイトのログ
django_migrations  -- 実行済みマイグレーションの記録
```
```
【auth_userテーブル(イメージ)】
| ID | ユーザー名 | メール | パスワード | 登録日 |
|----|----------|--------|-----------|---------|
| 1  | tanaka   | t@mail | xxxxx     | 2024/1/1|
| 2  | suzuki   | s@mail | yyyyy     | 2024/1/2|
| 3  | sato     | sa@mail| zzzzz     | 2024/1/3|
```
## モデル
```
【流れ】
モデル定義（設計図）
    ↓
makemigrations（変更を記録）
    ↓
migrate（データベースに反映）
    ↓
テーブル作成（構造も含めて完成）

【データベースの状態】
migrate前：テーブルなし
migrate後：定義されたモデルよりテーブル作成済み（カラム構造も完成、データは空
```

モデルはクラスで定義する。クラスは「データ＋処理」をパッケージ化した再利用可能な設計図。これにより、同じ構造を持つ異なるデータを効率的に扱える。


```
モデル（クラス）     = テーブル
クラス変数          = フィールド = カラム（列）

# インスタンス作成 = レコード（行）追加
q1 = Question(
    question_text="好きな色は？",
    pub_date="2024-01-01"
)
q1.save()

polls_questionテーブル
| id | question_text | pub_date    |
|----|--------------|-------------|
| 1  | 好きな色は？   | 2024-01-01  | ← この1行がインスタンス
```
IDとインスタンスが格納された変数名(ここではq1)はインスタンス作成時に紐付けされていて、その変数に実際にデータが格納されているわけではなくて、変数はデータベースを参照しているだけ

モデルを作成し、マイグレーションを実行することで、データベーススキーマ = データベースの「全体構造」が生成される。
データベーススキーマとは以下を含むデータ構造のこと
- テーブルの定義
- カラム（フィールド）の定義
- データ型の定義
- 制約（最大文字数など）の定義
- テーブル間の関係性


##### モデルを有効にする
アプリケーションをプロジェクトに含めるには、構成クラスへの参照を [INSTALLED_APPS](https://docs.djangoproject.com/ja/2.0/intro/tutorial02/#activating-models) 設定に追加する必要がある。
AppConfigは「このアプリの名前は何で、どんな設定で動かすか」を定義する設定ファイルのこと。
```
1. アプリケーション作成
   └── polls/
       ├── apps.py（構成クラス：PollsConfig）
       ├── models.py（モデル：Question, Choice）
       └── ...

2. INSTALLED_APPSに追加
   'polls.apps.PollsConfig' を追加

3. これにより以下のことがわかる：
   - pollsというアプリが存在することが認識される
   - models.pyのモデルも自動的に認識される
   - IDフィールドの型（BigAutoFieldを使う）
   - アプリの場所（pollsフォルダ）
   - マイグレーション対象になる
```
## 構成クラス（AppConfig）の構造
```python
# polls/apps.py
from django.apps import AppConfig

class PollsConfig(AppConfig):  # ← クラス名（任意だが慣習的に「アプリ名+Config」）
    default_auto_field = 'django.db.models.BigAutoField'  # ← 自動生成されるIDフィールドの型(各レコードを識別するidの最大値を決める)
    name = 'polls'  # ← アプリケーションの名前（必須）
```

```
python manage.py makemigrations polls
```

makemigrations を実行することで、Djangoにモデルに変更があったこと(この場合、新しいものを作成しました)を伝え、そして変更を マイグレーション の形で保存することができ

---
<details><summary>履歴</summary>


</details>