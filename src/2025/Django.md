<!-- # 腕立て伏せ100回を30日行うとどのような変化が起きたか？ -->
# DjangoでのWebサイト構築

<!--
date = "2025-06-22"
-->

### 使ったもの

[Django学習のロードマップ](https://qiita.com/nossey/items/03e114a7b728a8b2ae55)

[クイックインストールガイド](https://docs.djangoproject.com/ja/2.0/intro/install/)

[Djangoロードマップ](https://zenn.dev/suirunakamura/articles/bee05059ea7fad)

[Djangobrothersのチュートリアル](https://djangobrothers.com/tutorials/blog_app/first_app/)←これが一番わかりやすかった

[Django ウェブフレームワーク (Python)(MDN)](https://developer.mozilla.org/ja/docs/Learn_web_development/Extensions/Server-side/Django)

[Django最速マスター](https://qiita.com/gragragrao/items/373057783ba8856124f3#%E4%BD%95%E3%82%92%E4%BD%9C%E3%82%8B%E3%81%AE)

[Djangoの学習ロードマップ](https://akiyoko.hatenablog.jp/entry/2018/12/01/133427)
### やりたいこと

pythonのDjangoを使用してWebサイトを構築する


## Djangoについて

### memo

バージョン確認
``` zsh
python -m django --version
```

- プロジェクト作成
```
django-admin startproject project_name
```
##### 基本設定
- タイムゾーンと言語の変更`settings.py`
- マイグレーション`python manage.py migrate`
- スーパーユーザー設定`python manage.py createsuperuser`
- 開発用サーバーの起動`python manage.py runserver`
- アプリケーション作成`python manage.py startapp app_name`
- urls.pyにアプリケーションを追加`path('', include('app.urls')),`
- settings.pyにアプリケーション登録`INSTALLED_APPS`
- プロジェクトにアプリケーションを登録(モデルの有効化)``
- モデルの定義
- マイグレーションファイルを作成`python manage.py makemigrations cms`
- データベースに反映`python manage.py migrate cms`
- 管理サイトからデータの追加
- modelをadmin上で編集できるようにする(admin.pyにモデルを追加する)
- [管理サイトの一覧ページをカスタマイズする](https://qiita.com/kaki_k/items/7b178ad39394a031b50d#%E7%AE%A1%E7%90%86%E3%82%B5%E3%82%A4%E3%83%88%E3%81%AE%E4%B8%80%E8%A6%A7%E3%83%9A%E3%83%BC%E3%82%B8%E3%82%92%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%9E%E3%82%A4%E3%82%BA%E3%81%99%E3%82%8B)
- [Bootstrapの導入](https://qiita.com/kaki_k/items/6e17597804437ef170ae#bootstrap%E3%81%AE%E5%B0%8E%E5%85%A5)
- CRUDの作成


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
アプリケーション作成
```
python manage.py startapp app_name
```

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

### modelの型指定
```python
Django モデルフィールド チートシート

よく使う実装パターン
==================================================
名前・タイトル              → models.CharField(max_length=200)
説明・本文                  → models.TextField()
価格・金額                  → models.DecimalField(max_digits=10, decimal_places=2)
数量・カウント              → models.PositiveIntegerField(default=0)
フラグ・状態                → models.BooleanField(default=False)
作成日時                    → models.DateTimeField(auto_now_add=True)
更新日時                    → models.DateTimeField(auto_now=True)
カテゴリ（1つ）             → models.ForeignKey(Category, on_delete=models.CASCADE)
タグ（複数）                → models.ManyToManyField(Tag)
画像                       → models.ImageField(upload_to='images/')
==================================================

フィールドオプション
==================================================
必須項目                    → （デフォルト）
任意項目                    → blank=True, null=True
重複禁止                    → unique=True
デフォルト値                → default='値'
選択肢                      → choices=CHOICES
ヘルプテキスト              → help_text='説明'
管理画面の表示名            → verbose_name='表示名'
==================================================

ForeignKeyのon_delete
==================================================
親と一緒に削除              → on_delete=models.CASCADE
親が削除されてもNULL         → on_delete=models.SET_NULL, null=True
親の削除を禁止              → on_delete=models.PROTECT
デフォルト値に設定          → on_delete=models.SET_DEFAULT, default=値
==================================================

命名規則のヒント
==================================================
is_xxx, has_xxx            → BooleanField
xxx_count                  → PositiveIntegerField
xxx_at                     → DateTimeField
xxx_date                   → DateField
xxx_time                   → TimeField
price, amount, cost        → DecimalField
==================================================
```
データ列名→first_name, 


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

## モデルの定義
- models.Modelを継承したclassを定義して、その中でmodelsモジュールで定義されているclassでフィールドの型を指定する
    - models.ModelはDjangoが提供するmodelsモジュールで定義されている基底クラス
    - データベースの保存や、登録等の機能を備えている
- objectはデータとデータを操作する関数が定義されたまとまり
- 型 ≈ クラス ≈ オブジェクト（Pythonでは）
- 「型」は概念的な話、「オブジェクト」は実装的な話
- でも実際には同じものを違う角度から見ているだけ

#### 例えば投稿に関するモデルであるPostモデルを作成する場合
Post（投稿のモデル）といいねモデルの作成
- id: 主キー
- title: 投稿タイトル
- content: 投稿本文（任意）
- created_at: 投稿日時
などを定義する。これらはエクセルでいうところの縦の列に相当する。
```python

class Post(models.Model):
    # 投稿タイトル(100文字以内に制限)
    title = models.CharField(max_length=100)

    # 投稿本文
    content = models.TextField()

    # 投稿日時
    created_at = models.DateTimeField(auto_now_add=True)

class Vote()
    post = models.ForeignKey(
        'Post',
        on_delete=models.CASCADE,  # 投稿が削除されたら投票も削除
    ),



```


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

### 構成クラス（AppConfig）の構造
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

## クエリセット
##### [APIで遊んでみる](https://docs.djangoproject.com/ja/2.0/intro/tutorial02/#playing-with-the-api)

```
python manage.py shell
```
manage.pyからpythonシェルの起動

```
Django QuerySet チートシート

基本的な取得
==================================================
全件取得                   → Model.objects.all()
最初の1件                  → Model.objects.first()
最後の1件                  → Model.objects.last()
件数を取得                 → Model.objects.count()
存在確認                   → Model.objects.exists()
==================================================

単一オブジェクトの取得
==================================================
主キーで取得               → Model.objects.get(pk=1)
                          → Model.objects.get(id=1)
条件で取得                 → Model.objects.get(name='Django')
複数条件                   → Model.objects.get(name='Django', status='active')

# 注意: get()は必ず1件を返す。0件または複数件の場合はエラー
存在しない場合             → Model.DoesNotExist
複数存在する場合           → Model.MultipleObjectsReturned
==================================================

フィルタリング（filter）
==================================================
基本フィルタ               → Model.objects.filter(status='published')
複数条件（AND）            → Model.objects.filter(status='published', author='John')
チェーン                   → Model.objects.filter(status='published').filter(author='John')
除外                       → Model.objects.exclude(status='draft')
filter + exclude          → Model.objects.filter(category='tech').exclude(status='draft')
==================================================

フィールド検索（Field lookups）
==================================================
完全一致                   → filter(name='Django')
                          → filter(name__exact='Django')
大文字小文字を無視         → filter(name__iexact='django')
含む                       → filter(title__contains='Django')
含む（大文字小文字無視）   → filter(title__icontains='django')
前方一致                   → filter(name__startswith='Dj')
後方一致                   → filter(name__endswith='go')
IN検索                     → filter(id__in=[1, 2, 3])
                          → filter(status__in=['draft', 'published'])
==================================================

数値の比較
==================================================
より大きい                 → filter(price__gt=100)        # price > 100
以上                       → filter(price__gte=100)       # price >= 100
より小さい                 → filter(price__lt=100)        # price < 100
以下                       → filter(price__lte=100)       # price <= 100
範囲                       → filter(price__range=(10, 50)) # 10 <= price <= 50
==================================================

日付の検索
==================================================
特定の年                   → filter(created_at__year=2024)
特定の月                   → filter(created_at__month=12)
特定の日                   → filter(created_at__day=25)
日付の範囲                 → filter(created_at__date=date(2024, 12, 25))
                          → filter(created_at__gte=datetime(2024, 1, 1))
今日                       → filter(created_at__date=timezone.now().date())
今週                       → filter(created_at__week=52)
曜日（1=月曜、7=日曜）    → filter(created_at__week_day=2)  # 月曜日
==================================================

NULL値の検索
==================================================
NULLである                 → filter(description__isnull=True)
NULLでない                 → filter(description__isnull=False)
空文字列                   → filter(name__exact='')
空またはNULL               → filter(Q(name='') | Q(name__isnull=True))
===============================================
```



## Django Admin
Django adminサイトにアクセスできるアクセスできるユーザーの作成
```
Django Admin チートシート

必須インポート
==================================================
基本のインポート           → 
from django.contrib import admin
from .models import Model

フォーマット用             → 
from django.utils.html import format_html

管理コマンド用             → 
from django.urls import path

カスタムフォーム用         → 
from django import forms
==================================================

基本の登録（最小構成）
==================================================
# admin.py
from django.contrib import admin
from .models import Post

admin.site.register(Post)
==================================================

基本の登録パターン
==================================================
シンプルな登録             → from django.contrib import admin
                          from .models import Model
                          
                          admin.site.register(Model)

デコレータを使った登録     → from django.contrib import admin
                          from .models import Model
                          
                          @admin.register(Model)
                          class ModelAdmin(admin.ModelAdmin):
                              pass

複数モデル一括登録         → from django.contrib import admin
                          from .models import Model1, Model2, Model3
                          
                          admin.site.register([Model1, Model2, Model3])
==================================================

リスト表示のカスタマイズ
==================================================
from django.contrib import admin
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
   list_display = ['title', 'author', 'created_at', 'status']
   list_filter = ['status', 'created_at', 'author']
   search_fields = ['title', 'content']
   date_hierarchy = 'created_at'
   ordering = ['-created_at']
   list_per_page = 50
   list_editable = ['status']

admin.site.register(Post, PostAdmin)
==================================================

カスタムカラムの追加
==================================================
from django.contrib import admin
from django.utils.html import format_html
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
   list_display = ['title', 'colored_status', 'view_count']
   
   def colored_status(self, obj):
       colors = {
           'draft': 'gray',
           'published': 'green',
           'archived': 'red'
       }
       return format_html(
           '<span style="color: {};">{}</span>',
           colors.get(obj.status, 'black'),
           obj.get_status_display()
       )
   colored_status.short_description = 'ステータス'
==================================================

インライン編集の設定
==================================================
from django.contrib import admin
from .models import Post, Comment, Tag

class CommentInline(admin.TabularInline):
   model = Comment
   extra = 1

class TagInline(admin.StackedInline):
   model = Tag
   extra = 0

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
   inlines = [CommentInline, TagInline]
==================================================

アクションの追加
==================================================
from django.contrib import admin
from django.contrib import messages
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
   actions = ['make_published', 'make_draft']
   
   def make_published(self, request, queryset):
       updated = queryset.update(status='published')
       messages.success(request, f'{updated}件の記事を公開しました。')
   make_published.short_description = '選択した記事を公開'
   
   def make_draft(self, request, queryset):
       updated = queryset.update(status='draft')
       messages.info(request, f'{updated}件の記事を下書きに戻しました。')
   make_draft.short_description = '選択した記事を下書きに戻す'
==================================================

詳細画面のカスタマイズ
==================================================
from django.contrib import admin
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
   fieldsets = [
       (None, {
           'fields': ['title', 'slug', 'author']
       }),
       ('コンテンツ', {
           'fields': ['content', 'excerpt'],
           'classes': ['wide']
       }),
       ('公開設定', {
           'fields': ['status', 'published_at'],
           'classes': ['collapse']
       }),
       ('メタ情報', {
           'fields': ['created_at', 'updated_at'],
           'classes': ['collapse'],
           'description': 'システムが自動的に管理する情報'
       }),
   ]
   readonly_fields = ['created_at', 'updated_at']
   prepopulated_fields = {'slug': ('title',)}
==================================================

カスタムフォームの使用
==================================================
from django.contrib import admin
from django import forms
from .models import Post

class PostAdminForm(forms.ModelForm):
   class Meta:
       model = Post
       fields = '__all__'
       widgets = {
           'content': forms.Textarea(attrs={'rows': 20, 'cols': 80}),
           'excerpt': forms.Textarea(attrs={'rows': 3}),
       }

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
   form = PostAdminForm
==================================================

権限の制御
==================================================
from django.contrib import admin
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
   def get_queryset(self, request):
       qs = super().get_queryset(request)
       if request.user.is_superuser:
           return qs
       return qs.filter(author=request.user)
   
   def save_model(self, request, obj, form, change):
       if not change:  # 新規作成時
           obj.author = request.user
       super().save_model(request, obj, form, change)
   
   def has_delete_permission(self, request, obj=None):
       if obj and obj.author != request.user:
           return False
       return super().has_delete_permission(request, obj)
==================================================

管理サイトのカスタマイズ
==================================================
# admin.py の最後に追加
admin.site.site_header = 'My Project 管理画面'
admin.site.site_title = 'My Project Admin'
admin.site.index_title = 'サイト管理'
==================================================

完全な例（ブログ投稿）
==================================================
from django.contrib import admin
from django.utils.html import format_html
from django.urls import reverse
from django.contrib import messages
from .models import Post, Category, Tag, Comment

# インライン
class CommentInline(admin.TabularInline):
   model = Comment
   extra = 0
   fields = ['author', 'content', 'is_approved']
   readonly_fields = ['created_at']

# カテゴリ管理
@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
   list_display = ['name', 'slug', 'post_count']
   prepopulated_fields = {'slug': ('name',)}
   
   def post_count(self, obj):
       return obj.posts.count()
   post_count.short_description = '記事数'

# 投稿管理
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
   # リスト表示
   list_display = ['title', 'author', 'category', 'status_colored', 'created_at']
   list_filter = ['status', 'category', 'created_at']
   search_fields = ['title', 'content']
   date_hierarchy = 'created_at'
   ordering = ['-created_at']
   list_per_page = 20
   
   # 詳細画面
   fieldsets = [
       (None, {
           'fields': ['title', 'slug', 'author', 'category']
       }),
       ('コンテンツ', {
           'fields': ['content', 'excerpt'],
           'classes': ['wide']
       }),
       ('公開設定', {
           'fields': ['status', 'published_at', 'tags'],
           'classes': ['collapse']
       }),
   ]
   
   prepopulated_fields = {'slug': ('title',)}
   readonly_fields = ['created_at', 'updated_at']
   filter_horizontal = ['tags']
   inlines = [CommentInline]
   
   # カスタムメソッド
   def status_colored(self, obj):
       colors = {
           'draft': '#999999',
           'published': '#008000',
           'archived': '#ff0000'
       }
       return format_html(
           '<span style="color: {}; font-weight: bold;">{}</span>',
           colors.get(obj.status, '#000000'),
           obj.get_status_display()
       )
   status_colored.short_description = 'ステータス'
   
   # アクション
   actions = ['make_published']
   
   def make_published(self, request, queryset):
       updated = queryset.update(status='published')
       messages.success(request, f'{updated}件の記事を公開しました。')
   make_published.short_description = '選択した記事を公開'

# 管理サイトのカスタマイズ
admin.site.site_header = 'ブログ管理システム'
admin.site.site_title = 'ブログ管理'
admin.site.index_title = 'ダッシュボード'
==================================================
```
Djangoには、パスワードをリセットするための安全なコマンドが用意されています。changepasswordという管理コマンドを使用します。



## View
```python
Django Views チートシート

from .models import Post

関数ベースビュー（FBV）
==================================================
from .models import Post
基本のビュー               → def index(request):
                             return render(request, 'index.html')

クエリセットを渡す         → def post_list(request):
                             posts = Post.objects.all()
                             return render(request, 'posts.html', {'posts': posts})
                             # テンプレートでは {{ posts }} で使用可能

パラメータ受け取り         → def post_detail(request, pk):
                             post = get_object_or_404(Post, pk=pk)
                             return render(request, 'detail.html', {'post': post})
==================================================

クエリセットの渡し方
==================================================
基本パターン               → def view_name(request):
                             queryset = Model.objects.all()
                             return render(request, 'template.html', {'変数名': queryset})
                             # テンプレートで {{ 変数名 }} として使用

複数のクエリセット         → def dashboard(request):
                             posts = Post.objects.all()
                             users = User.objects.all()
                             return render(request, 'dashboard.html', {
                                 'posts': posts,
                                 'users': users
                             })
                             # テンプレートで {{ posts }} と {{ users }} が使用可能

フィルター済みを渡す       → def published_posts(request):
                             posts = Post.objects.filter(status='published')
                             return render(request, 'posts.html', {'posts': posts})
                             # テンプレートで {{ posts }} はpublishedのみ
==================================================

HTTPメソッドの処理
==================================================
GETとPOSTの分岐            → if request.method == 'POST':
                             # フォーム処理
                          else:
                             # 表示処理

POSTのみ許可               → @require_POST
                          def delete_view(request):

特定メソッドのみ           → @require_http_methods(['GET', 'POST'])
==================================================

よく使うレスポンス
==================================================
HTMLを返す                 → return render(request, 'template.html', context)
リダイレクト               → return redirect('app:view_name')
                          → return redirect('/some/url/')
404エラー                  → raise Http404("メッセージ")
                          → get_object_or_404(Model, pk=pk)
JSONを返す                 → return JsonResponse({'key': 'value'})
==================================================

フォーム処理の基本パターン
==================================================
def create_view(request):
   if request.method == 'POST':
       form = MyForm(request.POST)
       if form.is_valid():
           form.save()
           return redirect('success')
   else:
       form = MyForm()
   return render(request, 'form.html', {'form': form})
==================================================

クラスベースビュー（CBV）
==================================================
一覧表示                   → class PostListView(ListView):
                             model = Post
                             template_name = 'post_list.html'
                             context_object_name = 'posts'  # テンプレートで使う変数名

詳細表示                   → class PostDetailView(DetailView):
                             model = Post
                             template_name = 'post_detail.html'
                             # デフォルトでは {{ post }} として使用可能

作成                       → class PostCreateView(CreateView):
                             model = Post
                             fields = ['title', 'content']
                             success_url = reverse_lazy('post_list')

更新                       → class PostUpdateView(UpdateView):
                             model = Post
                             fields = ['title', 'content']
                             success_url = reverse_lazy('post_list')

削除                       → class PostDeleteView(DeleteView):
                             model = Post
                             success_url = reverse_lazy('post_list')
==================================================

デコレータ
==================================================
ログイン必須               → @login_required
                          def my_view(request):

権限チェック               → @permission_required('app.add_post')
キャッシュ                 → @cache_page(60 * 15)
CSRF除外                   → @csrf_exempt
==================================================

便利な関数・クラス
==================================================
404取得                    → post = get_object_or_404(Post, pk=pk)
リスト404                  → posts = get_list_or_404(Post, published=True)
ページネーション           → from django.core.paginator import Paginator
                          paginator = Paginator(queryset, 10)
                          page = paginator.get_page(request.GET.get('page'))
==================================================

リクエストオブジェクト
==================================================
GETパラメータ              → request.GET.get('q')
POSTデータ                 → request.POST.get('field_name')
ファイル                   → request.FILES.get('file')
ユーザー                   → request.user
メソッド                   → request.method
パス                       → request.path
Ajax判定                   → request.is_ajax()
==================================================

実践例：クエリセットを活用したビュー
==================================================
def blog_index(request):
   # 複数のクエリセットを準備
   recent_posts = Post.objects.filter(status='published').order_by('-created_at')[:5]
   popular_posts = Post.objects.filter(status='published').order_by('-view_count')[:5]
   categories = Category.objects.annotate(post_count=Count('post'))
   
   # テンプレートに渡す
   context = {
       'recent_posts': recent_posts,      # {{ recent_posts }} で使用
       'popular_posts': popular_posts,    # {{ popular_posts }} で使用
       'categories': categories,          # {{ categories }} で使用
       'total_posts': Post.objects.count()  # {{ total_posts }} で使用
   }
   
   return render(request, 'blog/index.html', context)

# テンプレート側では：
# {% for post in recent_posts %}
#     {{ post.title }}
# {% endfor %}
==================================================
```
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

## urls
urlsファイルでは同じ階層のviews.pyをimportしていて、urlsで対応させたviewsの関数を実行する。urls->viewsの順番。
```python
Django URLs チートシート

URLパターンの基本
==================================================
from django.urls import path
from . import views

基本のURL                   → path('', views.index, name='index')
詳細ページ                  → path('<int:pk>/', views.detail, name='detail')
文字列パラメータ            → path('<str:slug>/', views.post, name='post')
複数パラメータ              → path('<int:year>/<int:month>/', views.archive)
任意のパス                  → path('<path:url>/', views.redirect_view)
==================================================

include を使った分割
==================================================
アプリのURL読み込み         → path('blog/', include('blog.urls'))
名前空間付き               → path('blog/', include('blog.urls', namespace='blog'))
==================================================

パスコンバータ
==================================================
整数                       → <int:id>
文字列                     → <str:username>
スラッグ                   → <slug:post_slug>
UUID                       → <uuid:token>
パス（/を含む）            → <path:file_path>
==================================================

よく使うURLパターン
==================================================
一覧                       → path('', views.post_list, name='post_list')
詳細                       → path('<int:pk>/', views.post_detail, name='post_detail')
作成                       → path('create/', views.post_create, name='post_create')
編集                       → path('<int:pk>/edit/', views.post_edit, name='post_edit')
削除                       → path('<int:pk>/delete/', views.post_delete, name='post_delete')
==================================================

クラスベースビューのURL
==================================================
ListView                   → path('', PostListView.as_view(), name='post_list')
DetailView                 → path('<int:pk>/', PostDetailView.as_view(), name='post_detail')
CreateView                 → path('create/', PostCreateView.as_view(), name='post_create')
UpdateView                 → path('<int:pk>/edit/', PostUpdateView.as_view(), name='post_edit')
DeleteView                 → path('<int:pk>/delete/', PostDeleteView.as_view(), name='post_delete')
==================================================

正規表現を使う場合（re_path）
==================================================
4桁の年                    → re_path(r'^(?P<year>[0-9]{4})/$', views.year_archive)
電話番号                   → re_path(r'^(?P<phone>\d{3}-\d{4}-\d{4})/$', views.phone)
==================================================
```


##### [汎用ビューを使う](https://docs.djangoproject.com/ja/2.0/intro/tutorial04/#use-generic-views-less-code-is-better)

汎用ビューは、「View関数が処理を実行」の部分を自動化してくれる仕組み
DetailViewの場合：

- URLからpkを受け取る
- Question.objects.get(pk=pk)を実行
- 404エラー処理
- テンプレートにquestionという名前で渡す
- レスポンスを返す
等の処理をmodelとhtmlファイル場所を指定するだけで行ってくれる

```
return Question.objects.order_by('-pub_date')[:5]
```
このコードは3つの部分に分けられます：

Question.objects → 何を取得？
.order_by('-pub_date') → どう並べる？
[:5] → どれだけ取得？

## テンプレート
```
Django テンプレートタグ チートシート

基本的な書き方
==================================================
変数の表示                 → {{ variable }}
タグの使用                 → {% tag %}
フィルターの適用           → {{ variable|filter }}
コメント                   → {# コメント #}

3種類の記法：
{{ }}  → 変数や式を表示
{% %}  → タグ（制御構造など）
{# #}  → コメント
==================================================

基本的な使い方の例
==================================================
<!-- 変数の表示 -->
<h1>{{ title }}</h1>
<p>{{ user.username }}さん、こんにちは</p>

<!-- タグの使用 -->
{% if user.is_authenticated %}
   ログイン済み
{% endif %}

<!-- フィルターの適用 -->
<p>{{ text|truncatewords:10 }}</p>
<p>{{ price|floatformat:2 }}円</p>

<!-- コメント -->
{# TODO: ここに説明を追加 #}
==================================================

URL逆引き
==================================================
【テンプレートタグ → 実際のURL】
{% url 'blogs:index' %}
→ /blogs/

{% url 'blogs:detail' blog_id=1 %}
→ /blogs/1/

{% url 'blogs:detail' blog_id=blog.id %}
→ /blogs/42/  （blog.idが42の場合）

{% url 'blogs:edit' blog_id=blog.id %}
→ /blogs/42/edit/

{% url 'blogs:create' %}
→ /blogs/create/

{% url 'blogs:tag' tag_slug='django' %}
→ /blogs/tag/django/

基本のURL                  → {% url 'view_name' %}
パラメータ付き             → {% url 'view_name' pk=object.pk %}
                          → {% url 'view_name' id=1 %}
複数パラメータ             → {% url 'view_name' year=2024 month=12 %}
名前空間付き               → {% url 'app_name:view_name' %}
                          → {% url 'blogs:detail' blog_id=blog.id %}
位置引数                   → {% url 'view_name' arg1 arg2 %}
                          → {% url 'blog:archive' 2024 12 %}
変数として保存             → {% url 'view_name' pk=object.pk as the_url %}
                          → <a href="{{ the_url }}">リンク</a>
==================================================

テンプレート変数の展開
==================================================
変数                       → {{ variable }}
プロパティアクセス         → {{ object.property }}
                          → {{ user.username }}
メソッド呼び出し           → {{ object.get_absolute_url }}
                          → {{ user.get_full_name }}
辞書のキー                 → {{ dict.key }}
                          → {{ data.title }}
リストのインデックス       → {{ list.0 }}
                          → {{ items.1 }}
ネストしたアクセス         → {{ blog.author.profile.bio }}
==================================================

フィルター
==================================================
デフォルト値               → {{ value|default:"デフォルト値" }}
                          → {{ title|default:"無題" }}
空の場合のデフォルト       → {{ value|default_if_none:"N/A" }}
長さ/件数                  → {{ items|length }}
                          → {{ queryset|length }}
最初/最後                  → {{ list|first }}
                          → {{ list|last }}
大文字/小文字              → {{ name|upper }}
                          → {{ name|lower }}
                          → {{ title|title }}
文字数制限                 → {{ text|truncatechars:30 }}
単語数制限                 → {{ text|truncatewords:10 }}
改行をbrタグに             → {{ text|linebreaks }}
                          → {{ text|linebreaksbr }}
HTMLエスケープ             → {{ html|escape }}
HTMLエスケープ解除         → {{ html|safe }}
日付フォーマット           → {{ date|date:"Y年m月d日" }}
                          → {{ datetime|date:"Y-m-d H:i:s" }}
結合                       → {{ list|join:", " }}
                          → {{ tags|join:" / " }}
==================================================

条件分岐タグ
==================================================
基本のif文                 → {% if condition %}
                             内容
                          {% endif %}

if-else                    → {% if user.is_authenticated %}
                             ログイン中
                          {% else %}
                             未ログイン
                          {% endif %}

if-elif-else              → {% if value > 100 %}
                             高い
                          {% elif value > 50 %}
                             普通
                          {% else %}
                             安い
                          {% endif %}

and/or条件                 → {% if user.is_authenticated and user.is_staff %}
                          → {% if value > 0 or override %}

not条件                    → {% if not user.is_authenticated %}
                          → {% if value not in list %}

in演算子                   → {% if item in list %}
                          → {% if "django" in title|lower %}
==================================================

ループタグ
==================================================
基本のfor文                → {% for item in items %}
                             {{ item }}
                          {% endfor %}

インデックス付き           → {% for item in items %}
                             {{ forloop.counter }}: {{ item }}
                          {% endfor %}

空の場合の処理             → {% for item in items %}
                             {{ item }}
                          {% empty %}
                             アイテムがありません
                          {% endfor %}

辞書のループ               → {% for key, value in dict.items %}
                             {{ key }}: {{ value }}
                          {% endfor %}

逆順ループ                 → {% for item in items reversed %}
                             {{ item }}
                          {% endfor %}
==================================================

forループ変数
==================================================
1から始まるカウンタ        → {{ forloop.counter }}
0から始まるカウンタ        → {{ forloop.counter0 }}
残り回数（1から）          → {{ forloop.revcounter }}
残り回数（0から）          → {{ forloop.revcounter0 }}
最初の要素か               → {% if forloop.first %}
最後の要素か               → {% if forloop.last %}
親ループのカウンタ         → {{ forloop.parentloop.counter }}
==================================================

静的ファイル
==================================================
静的ファイル読み込み       → {% load static %}

CSS読み込み                → <link rel="stylesheet" href="{% static 'css/style.css' %}">
JavaScript読み込み         → <script src="{% static 'js/script.js' %}"></script>
画像読み込み               → <img src="{% static 'images/logo.png' %}" alt="Logo">

メディアファイル           → <img src="{{ user.avatar.url }}" alt="Avatar">
                          → <a href="{{ document.file.url }}">ダウンロード</a>
==================================================

テンプレート継承
==================================================
親テンプレート定義         → {% block content %}
                             デフォルトコンテンツ
                          {% endblock %}
                          
                          {% block title %}サイト名{% endblock %}

子テンプレートで継承       → {% extends "base.html" %}

ブロックの上書き           → {% block content %}
                             新しいコンテンツ
                          {% endblock %}

親のコンテンツを含める     → {% block content %}
                             {{ block.super }}
                             追加コンテンツ
                          {% endblock %}
==================================================

インクルード
==================================================
部分テンプレート読み込み   → {% include "partials/header.html" %}

変数を渡す                 → {% include "partials/card.html" with title="タイトル" %}
                          → {% include "item.html" with item=product %}

コンテキストを制限         → {% include "partial.html" with title="タイトル" only %}
==================================================

CSRF保護
==================================================
フォーム内に必須           → <form method="post">
                             {% csrf_token %}
                             <!-- フォーム要素 -->
                          </form>
==================================================

コメント
==================================================
単一行コメント             → {# これはコメントです #}

複数行コメント             → {% comment %}
                             これは
                             複数行の
                             コメントです
                          {% endcomment %}
==================================================

変数の定義
==================================================
変数の定義                 → {% with total=items|length %}
                             合計: {{ total }}個
                          {% endwith %}

複数変数の定義             → {% with price=100 tax=0.1 %}
                             税込: {{ price|add:price|floatformat:0 }}円
                          {% endwith %}

as句での定義               → {% url 'detail' pk=1 as detail_url %}
                          → {% get_current_language as LANGUAGE_CODE %}
==================================================

よく使うパターン
==================================================
ログイン状態で分岐         → {% if user.is_authenticated %}
                             こんにちは、{{ user.username }}さん
                             <a href="{% url 'logout' %}">ログアウト</a>
                          {% else %}
                             <a href="{% url 'login' %}">ログイン</a>
                          {% endif %}

権限チェック               → {% if perms.app_label.add_model %}
                             <a href="{% url 'model_add' %}">追加</a>
                          {% endif %}

現在のURLをチェック        → {% if request.resolver_match.url_name == 'home' %}
                             class="active"
                          {% endif %}

フラッシュメッセージ       → {% if messages %}
                             {% for message in messages %}
                               <div class="alert alert-{{ message.tags }}">
                                 {{ message }}
                               </div>
                             {% endfor %}
                          {% endif %}

ページネーション           → {% if page_obj.has_previous %}
                             <a href="?page={{ page_obj.previous_page_number }}">前へ</a>
                          {% endif %}
                          
                          ページ {{ page_obj.number }} / {{ page_obj.paginator.num_pages }}
                          
                          {% if page_obj.has_next %}
                             <a href="?page={{ page_obj.next_page_number }}">次へ</a>
                          {% endif %}
==================================================

カスタムタグ・フィルタ
==================================================
読み込み                   → {% load custom_tags %}
                          → {% load humanize %}
                          → {% load i18n %}

複数読み込み               → {% load static custom_tags humanize %}
==================================================

実践例：ブログ詳細ページ
==================================================
{% extends "base.html" %}
{% load static %}

{% block title %}{{ blog.title }} - {{ block.super }}{% endblock %}

{% block content %}
<article>
   <h1>{{ blog.title }}</h1>
   <p class="meta">
       投稿者: {{ blog.author.get_full_name|default:blog.author.username }}
       | {{ blog.created_at|date:"Y年m月d日 H:i" }}
   </p>
   
   {% if blog.thumbnail %}
       <img src="{{ blog.thumbnail.url }}" alt="{{ blog.title }}">
   {% endif %}
   
   <div class="content">
       {{ blog.content|linebreaks }}
   </div>
   
   <div class="tags">
       {% for tag in blog.tags.all %}
           <a href="{% url 'blogs:tag' tag_slug=tag.slug %}">#{{ tag.name }}</a>
           {% if not forloop.last %}, {% endif %}
       {% endfor %}
   </div>
   
   {% if user.is_authenticated and user == blog.author %}
       <div class="actions">
           <a href="{% url 'blogs:edit' blog_id=blog.id %}" class="btn">編集</a>
           <a href="{% url 'blogs:delete' blog_id=blog.id %}" class="btn btn-danger">削除</a>
       </div>
   {% endif %}
</article>
{% endblock %}
==================================================
```

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


#### [テストクライアント](https://docs.djangoproject.com/ja/2.0/intro/tutorial05/#the-django-test-client)

テストクライアントはターミナル上で動作する仮想のブラウザ
テストクライアントを使用して、次のステップを自動的に実行し、結果を確認することができる
1. URLを入力してアクセス
2. HTMLが表示される
3. フォームに入力して送信
4. 結果が表示される 

##### テストクライアントに仕事を頼む準備
```python
python manage.py shell

>>> from django.test.utils import setup_test_environment
>>> setup_test_environment()
>>> from django.test import Client
>>> # create an instance of the client for our use
>>> client = Client()
```

`setup_test_environment()` は、テンプレートのレンダラーをインストール
レンダラー = テンプレートとデータを組み合わせて、最終的なHTMLを生成するエンジン
テスト用レンダラー = 通常のレンダラー + 詳細情報を記録する機能


```
1. 
>>> response = client.get('/')
Not Found: /
>>> response.status_code
404

2. 
>>> from django.urls import reverse
>>> response = client.get(reverse('polls:index'))
>>> response.status_code
200

3. 
>>> response.content
b'\n    <ul>\n    \n        <li><a href="/polls/1/">What&#39;s up?</a></li>\n    \n    </ul>\n\n'

4. 
>>> response.context['latest_question_list']
<QuerySet [<Question: What's up?>]>
```

1. `http://127.0.0.1:8000/`にアクセスした際の挙動を確かめている 
    a. まだルートのテンプレートは作成していないので、404になる
2. reverse('polls:index')は`http://127.0.0.1:8000/polls/`にアクセスし4ている 1. status200はアクセスが成功していることを示す
3. b'...'のbはわからない
4. response.contextは'polls:index'にアクセスした際のレスポンスの内容を示している


```
Question.objects.filter(pub_date__lte=timezone.now())
```
pub_date が timezone.now 以前の Question を含んだクエリセットを返します。
ダブルアンダースコアは「Djangoが作った、フィールドと検索条件を区切るための特別な記号」で__lteは<=を意味する。


##### [テストの実行](https://docs.djangoproject.com/ja/2.0/intro/tutorial05/#running-tests)

```zsh
python manage.py test polls
```






---

<details><summary>履歴</summary>

- [2025-06-27 Fri] 何をどれだけやったらプログラミングがわかるようになるのか、ということを主題に
- [2025-06-27 Fri] Claudにコードを解説してもらって、理解してから次に移る

</details>