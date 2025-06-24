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

## View

views.py の仕事  
- HTTPリクエストの処理
- HTTPレスポンスの作成
- HTTPステータスコード（404等）
- Webに関する例外（Http404等）

##### URLにアクセスした際の処理を記述
```
Viewの基本的な流れ：
1. ユーザーがURLにアクセスする
   例: http://localhost:8000/polls/
   ↓
2. DjangoがURLを解析
   - プロジェクトのurls.py → アプリのurls.py
   ↓
3. urls.pyで対応するView関数を特定
   path('', views.index) → index関数を使うと判断
   ↓
4. DjangoがView関数を呼び出す
   views.index(request) を実行
   ↓
5. View関数が処理を実行
   - データベースからデータ取得
   - テンプレートにデータを渡す
   - レスポンスを作成
   ↓
6. Viewがレスポンスを返す
   return HttpResponse("Hello, world")
   ↓
7. ユーザーのブラウザに結果が表示される
```

##### viewを設定したらurls.pyを編集する
アプリケーションを始めて追加した際は、プロジェクトのurls.pyにアプリケーションのurlを登録する
ユーザーのアクセスしたurlが/app1/page1だった場合
1. プロジェクトのurls.pyによりurlの確認が行われる
2. /app1/から始まるurlだった場合、app1のurls.pyに処理を振り分ける
3. app1のurls.pyで定義されたurlにより処理を実行する
![イメージ図](./.drawio.png)


[renderショートカット](https://docs.djangoproject.com/ja/2.0/intro/tutorial03/#a-shortcut-render)
```
 order_by('-pub_date')
```
order_by : 並び替えの指定
pub_date：公開日で並び替え
-（マイナス）：降順（新しい順）
マイナスなし：昇順（古い順）

##### [簡単なフォームを書く](https://docs.djangoproject.com/ja/2.0/intro/tutorial04/#write-a-simple-form)
```python
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question
# ...
def vote(request, question_id):
    # 処理：指定されたIDの質問を取得
    # エラー時：質問が存在しない → 404エラーページ表示
    question = get_object_or_404(Question, pk=question_id)
    try:
        # question.choice_setはDjangoによって自動的に作成され、Questionに関連づけられたChoice全てにアクセスできるようになる。
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

### urls
urlsファイルでは同じ階層のviews.pyをimportしていて、urlsで対応させたviewsの関数を実行する。urls->viewsの順番。


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
良い設計（疎結合）：
┌─────────────┐     ┌─────────────┐
│  models.py  │     │  views.py   │
├─────────────┤     ├─────────────┤
│ DBの操作のみ │     │ HTTPの処理  │
│ DoesNotExist│     │ Http404     │
└─────────────┘     └─────────────┘
      ↑                    ↑
      └────────┬───────────┘
               │
         各自の仕事に専念！

悪い設計（密結合）：
┌─────────────┐
│  models.py  │
├─────────────┤
│ DBの操作    │
│ Http404 ←───┼─ なんでHTTPの処理してるの？
└─────────────┘
```


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

models.py の仕事
- データベースの操作
- データの検証
- ビジネスロジック
- データに関する例外（DoesNotExist等）

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

    # ↓自動生成されるIDフィールドの型(各レコードを識別するidの最大値を決める)
    default_auto_field = 'django.db.models.BigAutoField' 
    name = 'polls'  # ← アプリケーションの名前（必須）
```

```
python manage.py makemigrations polls
```

makemigrations を実行することで、Djangoにモデルに変更があったこと(この場合、新しいものを作成しました)を伝え、そして変更を マイグレーション の形で保存することができる。
マイグレーションはDjangoがモデル(データベース)の変更を保存する方法

```zsh
python manage.py sqlmigrate polls 0001
```

モデルから生成されたマイグレーションファイル（0001_initial.py）が
実際にどんなSQL文を実行するかを表示する
まだ実行はしない（確認だけ）

```
python manage.py shell
```
manage.pyからpythonシェルの起動

データベースの確認

##### Django Admin
Django adminサイトにアクセスできるアクセスできるユーザーの作成
```
python manage.py createsuperuser
```


##### テンプレート
ユーザーがurlにアクセスして、index.htmlが表示されるまでの流れ
```
【1. ユーザーがURLにアクセス】
ブラウザ: http://localhost:8000/polls/5/
                                    ↓

【2. Djangoがリクエストを受信】
Django: "GET /polls/5/ というリクエストが来た"
                                    ↓

【3. メインのURLconf確認】
mysite/urls.py:
    path('polls/', include('polls.urls'))  ← /polls/で始まる！
                                    ↓

【4. アプリのURLconf確認】
polls/urls.py:
    path('<int:question_id>/', views.detail, name='detail')
    ← パターンマッチ！question_id=5
                                    ↓

【5. View関数の呼び出し】
polls/views.py:
    def detail(request, question_id=5):
        question = get_object_or_404(Question, pk=5)
        return render(request, 'polls/detail.html', {'question': question})
                                    ↓

【6. データベースアクセス】
Question.objects.get(pk=5) → 質問データ取得
                                    ↓

【7. テンプレートの読み込み】
polls/templates/polls/detail.html を探す
                                    ↓

【8. テンプレートのレンダリング】
<h1>{{ question.question_text }}</h1>  ← データを埋め込みurls.pyで定義した名前を元に参照している
↓
<h1>好きな色は？</h1>  ← 実際のHTMLに変換
                                    ↓

【9. HTTPレスポンス返却】
Django → ブラウザ: HTMLデータを送信
                                    ↓

【10. ブラウザが表示】
ユーザーに質問詳細ページが表示される
```
##### テンプレート構文
Djangoテンプレート構文の基本
```python 
{{ }} = 値を表示する場所
        - 変数
        - 簡単な式（item.name、forloop.counter）
        - フィルター（{{ text|upper }}）

{% %} = Djangoテンプレート専用のタグ
        - if, for, while
        - url, csrf_token
        - include, extends
        - その他Djangoが用意したタグのみ

{% csrf_token %} - セキュリティタグ
役割：CSRF攻撃を防ぐ
動作：隠しフィールドを自動生成
生成されるHTML：

{% url %} - URL逆引きタグ
django<form action="{% url 'polls:vote' question.id %}" method="post">
役割：名前からURLを生成
構文：{% url '名前空間:URL名' パラメータ %}
結果：/polls/5/vote/ のようなURLを生成
```

##### [APIで遊んでみる](https://docs.djangoproject.com/ja/2.0/intro/tutorial02/#playing-with-the-api)
Djangoのmanage.pyからシェルを呼び出して、データベースにデータを追加する

##### [汎用ビューを使う](https://docs.djangoproject.com/ja/2.0/intro/tutorial04/#use-generic-views-less-code-is-better)
```
return Question.objects.order_by('-pub_date')[:5]
```
このコードは3つの部分に分けられます：

Question.objects → 何を取得？
.order_by('-pub_date') → どう並べる？
[:5] → どれだけ取得？

###### objectsはカラム（フィールド）ではない
もしQuestionモデルがこうだったら：
```
pythonclass Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')
```
フィールドは`question_text`と`pub_date`だけ

objectsは「データベースへの問い合わせ窓口」のようなもの。
例えば：
```python
python# 全ての質問を取得
Question.objects.all()

# 特定の質問を取得
Question.objects.get(id=1)

# 条件に合う質問を検索
Question.objects.filter(pub_date__year=2024)
```
##### →Question.objects:「Questionテーブル全体を操作するツール」
本1冊 = インスタンス（具体的なデータ）
司書さん = マネージャー（本を探したり、整理したりする人）
☝️これが.objects






---
<details><summary>履歴</summary>


</details>