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



### 索引
[Djangobrothersのチュートリアル](https://djangobrothers.com/tutorials/blog_app/first_app/)

- プロジェクト作成
```
django-admin startproject project_name
```

- タイムゾーンと言語の変更
- マイグレーション
- スーパーユーザー設定
- 開発用サーバーの起動`python manage.py runserver`
- アプリケーション作成`python manage.py startapp app_name`
- モデルの定義
- プロジェクトにアプリケーションを登録(モデルの有効化)
- モデルの定義
- マイグレーションファイルを作成`python manage.py makemigrations cms`
- データベースに反映`python manage.py migrate cms`
- 管理サイトからデータの追加
- modelをadmin上で編集できるようにする(admin.pyにモデルを追加する)
- [管理サイトの一覧ページをカスタマイズする](https://qiita.com/kaki_k/items/7b178ad39394a031b50d#%E7%AE%A1%E7%90%86%E3%82%B5%E3%82%A4%E3%83%88%E3%81%AE%E4%B8%80%E8%A6%A7%E3%83%9A%E3%83%BC%E3%82%B8%E3%82%92%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%9E%E3%82%A4%E3%82%BA%E3%81%99%E3%82%8B)
- [Bootstrapの導入](https://qiita.com/kaki_k/items/6e17597804437ef170ae#bootstrap%E3%81%AE%E5%B0%8E%E5%85%A5)
- CRUDの作成


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
```python
Django Views チートシート

関数ベースビュー（FBV）
==================================================
基本のビュー               → def index(request):
                              return render(request, 'index.html')

コンテキスト付き           → def post_list(request):
                              posts = Post.objects.all()
                              return render(request, 'posts.html', {'posts': posts})

パラメータ受け取り         → def post_detail(request, pk):
                              post = get_object_or_404(Post, pk=pk)
                              return render(request, 'detail.html', {'post': post})
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
                              context_object_name = 'posts'

詳細表示                   → class PostDetailView(DetailView):
                              model = Post
                              template_name = 'post_detail.html'

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

### urls
urlsファイルでは同じ階層のviews.pyをimportしていて、urlsで対応させたviewsの関数を実行する。urls->viewsの順番。
```python
Django URLs チートシート

URLパターンの基本
==================================================
基本のURL                   → path('', views.index, name='index')
詳細ページ                  → path('<int:pk>/', views.detail, name='detail')
文字列パラメータ            → path('<str:slug>/', views.post, name='post')
複数パラメータ              → path('<int:year>/<int:month>/', views.archive)
任意のパス                  → path('<path:url>/', views.redirect_view)
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

include を使った分割
==================================================
アプリのURL読み込み         → path('blog/', include('blog.urls'))
名前空間付き               → path('blog/', include('blog.urls', namespace='blog'))
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

### modelの定義
```python
Django モデルフィールド チートシート

要件                        → フィールド
==================================================
短いテキスト（〜255文字）     → CharField
長いテキスト                → TextField
整数                       → IntegerField
正の整数                    → PositiveIntegerField
小数（金額など）             → DecimalField
真偽値                      → BooleanField
日付                       → DateField
日時                       → DateTimeField
メール                      → EmailField
URL                        → URLField
画像                       → ImageField
ファイル                    → FileField
他モデルを1つ参照            → ForeignKey
他モデルを複数参照           → ManyToManyField
==================================================

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


##### [APIで遊んでみる](https://docs.djangoproject.com/ja/2.0/intro/tutorial02/#playing-with-the-api)

```
python manage.py shell
```
manage.pyからpythonシェルの起動

データベースの確認

Djangoのmanage.pyからシェルを呼び出して、データベースにデータを追加する



##### Django Admin
Django adminサイトにアクセスできるアクセスできるユーザーの作成
```
Django Admin チートシート

基本の登録
==================================================
シンプルな登録             → admin.site.register(Model)

カスタマイズして登録       → @admin.register(Model)
                          class ModelAdmin(admin.ModelAdmin):
                              pass

複数モデル登録             → admin.site.register([Model1, Model2])
==================================================

リスト表示のカスタマイズ
==================================================
表示フィールド             → list_display = ['name', 'created_at', 'status']
リンク付きフィールド       → list_display_links = ['name', 'created_at']
フィルター                 → list_filter = ['status', 'created_at', 'category']
検索                       → search_fields = ['name', 'description']
日付階層                   → date_hierarchy = 'created_at'
ページあたりの表示数       → list_per_page = 50
編集可能フィールド         → list_editable = ['status', 'price']
デフォルトの並び順         → ordering = ['-created_at']
==================================================

リスト表示の高度な設定
==================================================
カスタムカラム             → def colored_status(self, obj):
                              if obj.status == 'published':
                                  return format_html('<span style="color: green;">公開</span>')
                              return obj.status
                          colored_status.short_description = 'ステータス'
                          list_display = ['name', 'colored_status']

関連モデルの表示           → list_display = ['name', 'category__name']
                          list_select_related = ['category']  # N+1問題対策

アクション                 → actions = ['make_published', 'make_draft']
                          def make_published(self, request, queryset):
                              queryset.update(status='published')
                          make_published.short_description = '選択したアイテムを公開'
==================================================

詳細画面のカスタマイズ
==================================================
フィールドセット           → fieldsets = [
                              (None, {
                                  'fields': ['name', 'slug']
                              }),
                              ('詳細情報', {
                                  'fields': ['description', 'price'],
                                  'classes': ['collapse']
                              }),
                          ]

表示フィールド指定         → fields = ['name', 'description', 'price']
除外フィールド             → exclude = ['created_by']
読み取り専用               → readonly_fields = ['created_at', 'updated_at']
==================================================

インライン編集
==================================================
タブ形式                   → class ItemInline(admin.TabularInline):
                              model = Item
                              extra = 1

スタック形式               → class ItemInline(admin.StackedInline):
                              model = Item
                              extra = 0
                              
インラインの追加           → inlines = [ItemInline]
==================================================

フォームのカスタマイズ
==================================================
カスタムフォーム           → form = MyModelForm

フォームフィールド上書き   → formfield_overrides = {
                              models.TextField: {'widget': forms.Textarea(attrs={'rows': 4})},
                          }

動的なフィールド           → def get_form(self, request, obj=None, **kwargs):
                              form = super().get_form(request, obj, **kwargs)
                              if not request.user.is_superuser:
                                  form.base_fields['status'].disabled = True
                              return form
==================================================

権限とフィルタリング
==================================================
閲覧権限                   → def has_view_permission(self, request, obj=None):
                              return request.user.is_staff

追加権限                   → def has_add_permission(self, request):
                              return request.user.is_superuser

変更権限                   → def has_change_permission(self, request, obj=None):
                              if obj and obj.created_by != request.user:
                                  return False
                              return True

削除権限                   → def has_delete_permission(self, request, obj=None):
                              return False  # 削除を禁止

クエリセット制限           → def get_queryset(self, request):
                              qs = super().get_queryset(request)
                              if request.user.is_superuser:
                                  return qs
                              return qs.filter(created_by=request.user)
==================================================

保存時の処理
==================================================
保存前の処理               → def save_model(self, request, obj, form, change):
                              if not change:  # 新規作成時
                                  obj.created_by = request.user
                              obj.updated_by = request.user
                              super().save_model(request, obj, form, change)

フォーム保存時             → def save_form(self, request, form, change):
                              obj = form.save(commit=False)
                              # カスタム処理
                              return obj
==================================================

表示のカスタマイズ
==================================================
管理画面のタイトル         → admin.site.site_header = 'My Admin'
                          admin.site.site_title = 'My Admin Portal'
                          admin.site.index_title = 'Welcome to My Admin'

モデル名の変更             → class Meta:
                              verbose_name = '記事'
                              verbose_name_plural = '記事一覧'

空の値の表示               → empty_value_display = '-'
==================================================

よく使うパターン（完全な例）
==================================================
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
   # リスト表示
   list_display = ['title', 'author', 'status', 'created_at']
   list_filter = ['status', 'created_at', 'author']
   search_fields = ['title', 'content']
   date_hierarchy = 'created_at'
   ordering = ['-created_at']
   
   # 詳細画面
   fieldsets = [
       (None, {
           'fields': ['title', 'slug', 'author']
       }),
       ('コンテンツ', {
           'fields': ['content', 'excerpt']
       }),
       ('公開設定', {
           'fields': ['status', 'published_at'],
           'classes': ['collapse']
       }),
   ]
   
   # 自動入力
   prepopulated_fields = {'slug': ('title',)}
   
   # 読み取り専用
   readonly_fields = ['created_at', 'updated_at']
   
   # インライン
   inlines = [CommentInline, AttachmentInline]
   
   # アクション
   actions = ['make_published', 'make_draft']
   
   def make_published(self, request, queryset):
       count = queryset.update(status='published')
       self.message_user(request, f'{count}件の記事を公開しました。')
   make_published.short_description = '選択した記事を公開'
==================================================

高度な機能
==================================================
カスタムビュー追加         → def get_urls(self):
                              urls = super().get_urls()
                              custom_urls = [
                                  path('stats/', self.stats_view, name='post_stats'),
                              ]
                              return custom_urls + urls

一括フィールド変更         → def changelist_view(self, request, extra_context=None):
                              extra_context = extra_context or {}
                              extra_context['custom_var'] = 'value'
                              return super().changelist_view(request, extra_context)

メディアファイル追加       → class Media:
                              css = {
                                  'all': ('admin/css/custom.css',)
                              }
                              js = ('admin/js/custom.js',)
==================================================
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
Django Template チートシート

基本構文
==================================================
変数                       → {{ variable }}
変数のプロパティ           → {{ object.property }}
リストのインデックス       → {{ list.0 }}
辞書のキー                 → {{ dict.key }}
タグ                       → {% tag %}
コメント                   → {# コメント #}
複数行コメント             → {% comment %} ... {% endcomment %}
==================================================

テンプレート継承
==================================================
ベーステンプレート定義     → {% block content %}
                          {% endblock %}

テンプレート継承           → {% extends "base.html" %}

ブロックの上書き           → {% block content %}
                             新しい内容
                          {% endblock %}

親のブロックを含める       → {% block content %}
                             {{ block.super }}
                             追加内容
                          {% endblock %}
==================================================

変数の表示
==================================================
デフォルト値               → {{ variable|default:"デフォルト" }}
空の場合のデフォルト       → {{ variable|default_if_none:"なし" }}
HTMLエスケープなし         → {{ variable|safe }}
改行をbrタグに             → {{ text|linebreaks }}
URLを自動リンク            → {{ text|urlize }}
==================================================

条件分岐
==================================================
if文                       → {% if condition %}
                          {% elif other_condition %}
                          {% else %}
                          {% endif %}

存在チェック               → {% if variable %}
比較                       → {% if value > 10 %}
論理演算                   → {% if user.is_authenticated and perms.app.add_model %}
否定                       → {% if not variable %}
==================================================

ループ
==================================================
基本のループ               → {% for item in items %}
                             {{ item }}
                          {% endfor %}

空の場合                   → {% for item in items %}
                             {{ item }}
                          {% empty %}
                             アイテムがありません
                          {% endfor %}

ループカウンタ             → {{ forloop.counter }}      # 1から
                          → {{ forloop.counter0 }}     # 0から
                          → {{ forloop.first }}        # 最初
                          → {{ forloop.last }}         # 最後
==================================================

URL生成
==================================================
名前付きURL                → {% url 'view_name' %}
パラメータ付き             → {% url 'view_name' pk=object.pk %}
名前空間付き               → {% url 'app:view_name' %}
変数として保存             → {% url 'view_name' as the_url %}
==================================================

静的ファイル
==================================================
静的ファイル読み込み       → {% load static %}
                          <link href="{% static 'css/style.css' %}" rel="stylesheet">
                          <script src="{% static 'js/script.js' %}"></script>
                          <img src="{% static 'images/logo.png' %}">

メディアファイル           → <img src="{{ object.image.url }}">
==================================================

フォーム
==================================================
CSRFトークン               → {% csrf_token %}

フォーム全体               → {{ form }}
                          {{ form.as_p }}
                          {{ form.as_table }}
                          {{ form.as_ul }}

個別フィールド             → {{ form.field_name }}
                          {{ form.field_name.label }}
                          {{ form.field_name.errors }}
                          {{ form.field_name.help_text }}

フォームエラー             → {{ form.non_field_errors }}
==================================================

フィルター（よく使うもの）
==================================================
文字数制限                 → {{ text|truncatechars:20 }}
単語数制限                 → {{ text|truncatewords:10 }}
小文字/大文字              → {{ text|lower }} / {{ text|upper }}
最初を大文字               → {{ text|title }}
日付フォーマット           → {{ date|date:"Y年m月d日" }}
時刻フォーマット           → {{ time|time:"H:i" }}
数値カンマ区切り           → {{ number|floatformat:2 }}
ファイルサイズ             → {{ bytes|filesizeformat }}
リストの長さ               → {{ list|length }}
最初/最後の要素            → {{ list|first }} / {{ list|last }}
結合                       → {{ list|join:", " }}
==================================================

インクルード
==================================================
他のテンプレート読み込み   → {% include "partial.html" %}
変数付き                   → {% include "partial.html" with variable=value %}
コンテキスト制限           → {% include "partial.html" only %}
==================================================

便利なタグ
==================================================
現在時刻                   → {% now "Y年m月d日 H:i" %}
Lorem ipsum                → {% lorem %}
デバッグ情報               → {% debug %}
改行を無視                 → {% spaceless %} ... {% endspaceless %}
自動エスケープ無効         → {% autoescape off %} ... {% endautoescape %}
==================================================

カスタムタグ・フィルター読み込み
==================================================
カスタムタグ読み込み       → {% load my_tags %}
複数読み込み               → {% load my_tags my_filters %}
==================================================

テンプレート内での変数定義
==================================================
変数の定義                 → {% with total=items|length %}
                             合計: {{ total }}
                          {% endwith %}

複数変数                   → {% with a=1 b=2 c=3 %}
                             {{ a }} + {{ b }} + {{ c }}
                          {% endwith %}
==================================================

よく使うパターン
==================================================
ページネーション           → {% if page.has_previous %}
                             <a href="?page={{ page.previous_page_number }}">前へ</a>
                          {% endif %}
                          
                          ページ {{ page.number }} / {{ page.paginator.num_pages }}
                          
                          {% if page.has_next %}
                             <a href="?page={{ page.next_page_number }}">次へ</a>
                          {% endif %}

フラッシュメッセージ       → {% if messages %}
                             {% for message in messages %}
                               <div class="alert alert-{{ message.tags }}">
                                 {{ message }}
                               </div>
                             {% endfor %}
                          {% endif %}
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