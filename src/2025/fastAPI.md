# FastAPIでCRUD APIを作る完全ガイド - スタッフ管理機能編
<!--
date = "2025-10-09"
-->

## はじめに

「APIを作る」と聞くと難しそうに感じますが、実は料理のレシピを作るのと似ています。材料（データ）があって、調理方法（処理）があって、完成した料理（レスポンス）を提供する。この記事では、プログラミング初心者でも理解できるよう、**なぜそうするのか**を徹底的に解説します。

この記事を読み終える頃には、あなたも「APIってこういうことか！」とアハ体験できるはずです。

## 今日作るもの

スタッフ情報を管理するAPIを作ります。具体的には以下の5つの機能です。

| 機能 | URL | 例え |
|------|-----|------|
| 一覧取得 | GET /api/staff | 名簿を見る |
| 新規作成 | POST /api/staff | 名簿に追加 |
| 詳細取得 | GET /api/staff/{id} | 特定の人を探す |
| 更新 | PUT /api/staff/{id} | 情報を修正 |
| 削除 | DELETE /api/staff/{id} | 名簿から消す |

これを**CRUD**（クラッド）と呼びます。Create（作成）、Read（読取）、Update（更新）、Delete（削除）の頭文字です。

## なぜルーターが必要なのか

### 問題：main.pyが巨大化する

すべてのAPIをmain.pyに書くと、こうなります：
```python
# main.py
@app.get("/api/staff")
def get_staff():
    # スタッフ一覧

@app.post("/api/staff")
def create_staff():
    # スタッフ作成

@app.get("/api/customers")
def get_customers():
    # 顧客一覧

@app.post("/api/bookings")
def create_booking():
    # 予約作成

# ... 100行、200行と増えていく
```

**問題点**：
- どこに何があるか分からない
- 修正が大変
- チーム開発で衝突しやすい

### 解決策：ルーターで分割

料理のレシピを「前菜」「メイン」「デザート」に分けるように、APIも機能ごとに分けます。
```
routers/
├── staff.py      ← スタッフ関連のAPI
├── customers.py  ← 顧客関連のAPI
└── bookings.py   ← 予約関連のAPI
```

**メリット**：
- 見つけやすい
- 修正しやすい
- チームで分担できる

これが**ルーター**の役割です。

## ステップ1: フォルダ構成を作る

### なぜこの構成なのか
```
line-booking-bot/
├── main.py              ← 司令塔（全体をまとめる）
├── database.py          ← データベース接続
├── models.py            ← データの設計図
├── routers/             ← API置き場
│   ├── __init__.py      ← 「ここはPythonパッケージだよ」の印
│   └── staff.py         ← スタッフAPI
```

#### なぜ`__init__.py`が必要？

Pythonは`__init__.py`があるフォルダを「パッケージ（まとまり）」として認識します。これがないと、以下のようなimportができません：
```python
from routers import staff  # ← これができない
```

空のファイルでOKです。**「ここは特別なフォルダだよ」とPythonに教えるための目印**です。

### 実際に作る
```bash
# routersフォルダ作成
mkdir routers

# __init__.pyを作成（空でOK）
touch routers/__init__.py

# staff.pyを作成
touch routers/staff.py
```

## ステップ2: Pydanticモデルを理解する

### なぜPydanticが必要なのか

APIを作る時、**「どんなデータを受け取るか」「どんなデータを返すか」を明確にする必要があります**。

#### 例：レストランの注文

お客さん（クライアント）がウェイター（API）に注文します。

**注文票がない場合**：
```
お客さん: 「何か食べたいです」
ウェイター: 「...何を？」
お客さん: 「美味しいやつ」
ウェイター: 「...具体的には？」
```

**注文票がある場合**：
```
【注文票】
- 料理名: カレーライス
- 辛さ: 中辛
- サイズ: 大盛り

→ 明確！スムーズ！
```

Pydanticは、この**注文票**の役割を果たします。

### 3つのPydanticモデル
```python
# 1. スタッフ作成時（POST）に受け取るデータ
class StaffCreate(BaseModel):
    name: str               # 名前（必須）
    display_order: int = 0  # 表示順（省略可、デフォルト0）

# 2. スタッフ更新時（PUT）に受け取るデータ
class StaffUpdate(BaseModel):
    name: Optional[str] = None         # 名前（省略可）
    is_active: Optional[bool] = None   # 有効/無効（省略可）
    display_order: Optional[int] = None # 表示順（省略可）

# 3. スタッフ情報を返す時（レスポンス）のデータ
class StaffResponse(BaseModel):
    id: str
    name: str
    is_active: bool
    display_order: int
```

#### なぜ3つも必要？

**作成時**：IDは自動生成されるので不要。名前だけ必須。
**更新時**：変更したい項目だけ送りたい。全部Noneでもいい。
**レスポンス**：IDや有効/無効フラグなど、すべての情報を返す。

**用途が違うから、型も違う**のです。

### `Optional[str]`とは？

「この値はNone（空）でもいいよ」という意味です。
```python
from typing import Optional

name: Optional[str] = None
# ↑ nameはstrかNone、デフォルトはNone
```

Python 3.10以降では新しい書き方もできます：
```python
name: str | None = None  # Python 3.10+の新しい構文
```

**どちらも同じ意味**ですが、Python 3.9では`Optional`を使う必要があります。

### `class Config`とは？

Pydanticに追加の設定を伝えるためのクラスです。
```python
class Config:
    from_attributes = True  # SQLAlchemyのオブジェクトから変換OK
    json_schema_extra = {   # Swagger UIに表示する例
        "example": {
            "name": "田中先生",
            "display_order": 1
        }
    }
```

**なぜ`from_attributes = True`が必要？**

データベースから取得したデータ（SQLAlchemyオブジェクト）を、Pydanticモデルにそのまま変換するために必要です。
```python
# データベースから取得
staff = db.query(Staff).first()  # SQLAlchemyオブジェクト

# Pydanticモデルに変換
return staff  # from_attributes = True がないとエラー
```

## ステップ3: 5つのAPIエンドポイントを作る

### 完全なコード（routers/staff.py）
```python
# routers/staff.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from typing import List, Optional  # ← Optional を追加
from pydantic import BaseModel
from database import get_db
from models import Staff

# ルーター作成
router = APIRouter(
    prefix="/api/staff",
    tags=["staff"]
)

# Pydanticモデル（リクエスト・レスポンスの形式）
class StaffCreate(BaseModel):
    """スタッフ作成時のリクエストボディ"""
    name: str
    display_order: int = 0
    
    class Config:
        json_schema_extra = {
            "example": {
                "name": "田中先生",
                "display_order": 1
            }
        }

class StaffUpdate(BaseModel):
    """スタッフ更新時のリクエストボディ"""
    name: Optional[str] = None
    is_active: Optional[bool] = None
    display_order: Optional[int] = None
    
    class Config:
        json_schema_extra = {
            "example": {
                "name": "田中太郎先生",
                "is_active": True,
                "display_order": 2
            }
        }

class StaffResponse(BaseModel):
    """スタッフのレスポンス"""
    id: str
    name: str
    is_active: bool
    display_order: int
    
    class Config:
        from_attributes = True
        json_schema_extra = {
            "example": {
                "id": "123e4567-e89b-12d3-a456-426614174000",
                "name": "田中先生",
                "is_active": True,
                "display_order": 1
            }
        }


# ========================================
# API エンドポイント
# ========================================

@router.get("/", response_model=List[StaffResponse])
def get_staff_list(db: Session = Depends(get_db)):
    """
    スタッフ一覧取得
    
    - display_orderの昇順でソート
    - 有効・無効に関わらず全て取得
    """
    staff_list = db.query(Staff).order_by(Staff.display_order).all()
    return staff_list


@router.post("/", response_model=StaffResponse, status_code=201)
def create_staff(staff: StaffCreate, db: Session = Depends(get_db)):
    """
    スタッフ作成
    
    - 同じ名前でも作成可能
    - IDは自動生成
    """
    new_staff = Staff(
        name=staff.name,
        display_order=staff.display_order
    )
    db.add(new_staff)
    db.commit()
    db.refresh(new_staff)
    return new_staff


@router.get("/{staff_id}", response_model=StaffResponse)
def get_staff(staff_id: str, db: Session = Depends(get_db)):
    """
    スタッフ詳細取得
    
    - staff_idで1件取得
    - 存在しない場合は404エラー
    """
    staff = db.query(Staff).filter(Staff.id == staff_id).first()
    if not staff:
        raise HTTPException(status_code=404, detail="Staff not found")
    return staff


@router.put("/{staff_id}", response_model=StaffResponse)
def update_staff(
    staff_id: str,
    staff_update: StaffUpdate,
    db: Session = Depends(get_db)
):
    """
    スタッフ更新
    
    - 指定したフィールドのみ更新
    - Noneの場合は更新しない
    """
    staff = db.query(Staff).filter(Staff.id == staff_id).first()
    if not staff:
        raise HTTPException(status_code=404, detail="Staff not found")
    
    # 更新処理
    if staff_update.name is not None:
        staff.name = staff_update.name
    if staff_update.is_active is not None:
        staff.is_active = staff_update.is_active
    if staff_update.display_order is not None:
        staff.display_order = staff_update.display_order
    
    db.commit()
    db.refresh(staff)
    return staff


@router.delete("/{staff_id}")
def delete_staff(staff_id: str, db: Session = Depends(get_db)):
    """
    スタッフ削除
    
    - 物理削除（データベースから完全削除）
    - 予約データがある場合の処理は未実装（後で対応）
    """
    staff = db.query(Staff).filter(Staff.id == staff_id).first()
    if not staff:
        raise HTTPException(status_code=404, detail="Staff not found")
    
    db.delete(staff)
    db.commit()
    return {"message": "Staff deleted successfully", "id": staff_id}
```

### 全体像
```python
router = APIRouter(prefix="/api/staff", tags=["staff"])

@router.get("/")              # 一覧取得
@router.post("/")             # 新規作成
@router.get("/{staff_id}")    # 詳細取得
@router.put("/{staff_id}")    # 更新
@router.delete("/{staff_id}") # 削除
```

### なぜ`APIRouter`を使うのか

main.pyで直接`@app.get()`と書くこともできます。でも、ルーターを使うと：
```python
# main.pyには短く書ける
app.include_router(staff.router)  # これだけ！

# routers/staff.pyで詳しく書く
router = APIRouter(prefix="/api/staff")
@router.get("/")
def get_staff_list():
    ...
```

**分離すると管理が楽**になります。

### 1. 一覧取得（GET /api/staff）
```python
@router.get("/", response_model=List[StaffResponse])
def get_staff_list(db: Session = Depends(get_db)):
    """スタッフ一覧取得"""
    staff_list = db.query(Staff).order_by(Staff.display_order).all()
    return staff_list
```

#### なぜ`response_model=List[StaffResponse]`？

FastAPIに「このAPIは`StaffResponse`のリスト（配列）を返すよ」と教えます。すると：

1. 自動でバリデーション（検証）
2. Swagger UIに型情報が表示される
3. 不要なデータは自動で除外される

#### なぜ`Depends(get_db)`？

**依存性注入**（Dependency Injection）という仕組みです。

難しく聞こえますが、要するに**「必要なものを自動で渡してもらう」**仕組みです。
```python
# Depends(get_db) がないと
def get_staff_list():
    db = SessionLocal()  # ← 毎回手動で接続
    try:
        staff_list = db.query(Staff).all()
        return staff_list
    finally:
        db.close()  # ← 毎回手動でクローズ

# Depends(get_db) があると
def get_staff_list(db: Session = Depends(get_db)):
    staff_list = db.query(Staff).all()
    return staff_list  # ← 自動でクローズされる
```

**メリット**：
- コードが短くなる
- クローズ忘れがない
- テストしやすい

#### なぜ`order_by(Staff.display_order)`？

データベースから取得する順番を指定します。
```python
# 指定なし
staff_list = db.query(Staff).all()
# → ランダムな順番（データベース任せ）

# display_orderで昇順
staff_list = db.query(Staff).order_by(Staff.display_order).all()
# → 1, 2, 3... の順番で取得
```

スタッフを表示する順番を制御したいので、`display_order`でソートします。

### 2. 新規作成（POST /api/staff）
```python
@router.post("/", response_model=StaffResponse, status_code=201)
def create_staff(staff: StaffCreate, db: Session = Depends(get_db)):
    """スタッフ作成"""
    new_staff = Staff(
        name=staff.name,
        display_order=staff.display_order
    )
    db.add(new_staff)
    db.commit()
    db.refresh(new_staff)
    return new_staff
```

#### なぜ`status_code=201`？

HTTPステータスコードには意味があります。

| コード | 意味 | 使い所 |
|--------|------|--------|
| 200 | 成功 | データ取得 |
| 201 | 作成成功 | データ作成 |
| 400 | リクエストエラー | 入力が間違い |
| 404 | 見つからない | データが存在しない |
| 500 | サーバーエラー | バグ |

**201は「新しいデータを作りました！」という意味**です。RESTful APIの慣習として、作成成功時は201を返します。

#### なぜ`db.refresh(new_staff)`？
```python
new_staff = Staff(name="田中先生")  # idはまだない
db.add(new_staff)
db.commit()  # ← ここでデータベースに保存され、idが自動生成される

print(new_staff.id)  # None ← まだPythonオブジェクトには反映されていない

db.refresh(new_staff)  # ← データベースから最新情報を取得
print(new_staff.id)  # uuid-123 ← idが反映された！
```

**`db.commit()`でデータベースに保存されるが、Pythonオブジェクトには自動で反映されない**ため、`db.refresh()`で最新情報を取得します。

### 3. 詳細取得（GET /api/staff/{staff_id}）
```python
@router.get("/{staff_id}", response_model=StaffResponse)
def get_staff(staff_id: str, db: Session = Depends(get_db)):
    """スタッフ詳細取得"""
    staff = db.query(Staff).filter(Staff.id == staff_id).first()
    if not staff:
        raise HTTPException(status_code=404, detail="Staff not found")
    return staff
```

#### なぜ`{staff_id}`？

**パスパラメータ**と呼ばれる仕組みです。URLの一部を変数として受け取ります。
```
GET /api/staff/123  ← staff_id = "123"
GET /api/staff/456  ← staff_id = "456"
```

関数の引数として受け取れます：
```python
def get_staff(staff_id: str, ...):
    print(staff_id)  # "123" または "456"
```

#### なぜ`.first()`？
```python
# .first() がないと
staff = db.query(Staff).filter(Staff.id == staff_id)
# → クエリオブジェクト（まだ実行されていない）

# .first() があると
staff = db.query(Staff).filter(Staff.id == staff_id).first()
# → 実際にデータベースに問い合わせて、1件目を取得
```

`.first()`は「最初の1件を取得、なければNone」という意味です。

#### なぜ`HTTPException`？

エラーを返す時は`HTTPException`を使います。
```python
if not staff:
    raise HTTPException(status_code=404, detail="Staff not found")
```

これをすると、クライアントに以下のようなレスポンスが返ります：
```json
{
  "detail": "Staff not found"
}
```

**Pythonの通常の例外（Exception）とは違い、HTTPレスポンスとして返される**ため、クライアントが適切に処理できます。

### 4. 更新（PUT /api/staff/{staff_id}）
```python
@router.put("/{staff_id}", response_model=StaffResponse)
def update_staff(
    staff_id: str,
    staff_update: StaffUpdate,
    db: Session = Depends(get_db)
):
    """スタッフ更新"""
    staff = db.query(Staff).filter(Staff.id == staff_id).first()
    if not staff:
        raise HTTPException(status_code=404, detail="Staff not found")
    
    # 更新処理
    if staff_update.name is not None:
        staff.name = staff_update.name
    if staff_update.is_active is not None:
        staff.is_active = staff_update.is_active
    if staff_update.display_order is not None:
        staff.display_order = staff_update.display_order
    
    db.commit()
    db.refresh(staff)
    return staff
```

#### なぜ`is not None`でチェック？

`StaffUpdate`では全項目が`None`でもOKです。**送られた項目だけを更新したい**ので、チェックが必要です。
```python
# リクエスト: 名前だけ更新
{
  "name": "田中太郎先生"
}

# staff_updateの中身
staff_update.name = "田中太郎先生"
staff_update.is_active = None  # 送られていない
staff_update.display_order = None  # 送られていない
```
```python
# if を使わないと
staff.name = staff_update.name  # OK
staff.is_active = staff_update.is_active  # Noneで上書きされてしまう！

# if を使うと
if staff_update.name is not None:
    staff.name = staff_update.name  # OK
if staff_update.is_active is not None:
    staff.is_active = staff_update.is_active  # スキップ
```

**送られていない項目は更新しない**ために、Noneチェックが必要です。

### 5. 削除（DELETE /api/staff/{staff_id}）
```python
@router.delete("/{staff_id}")
def delete_staff(staff_id: str, db: Session = Depends(get_db)):
    """スタッフ削除"""
    staff = db.query(Staff).filter(Staff.id == staff_id).first()
    if not staff:
        raise HTTPException(status_code=404, detail="Staff not found")
    
    db.delete(staff)
    db.commit()
    return {"message": "Staff deleted successfully", "id": staff_id}
```

#### なぜ辞書を返す？

削除APIは通常、削除されたIDやメッセージを返します。
```python
return {"message": "Staff deleted successfully", "id": staff_id}
```

クライアントに「ちゃんと削除されたよ！」と伝えるためです。
```json
{
  "message": "Staff deleted successfully",
  "id": "uuid-123"
}
```

## ステップ4: main.pyにルーターを登録する

### なぜ登録が必要？

routers/staff.pyでAPIを作っても、**main.pyに登録しないとFastAPIが認識しません**。
```python
# main.py
from routers import staff  # ← インポート

app.include_router(staff.router)  # ← 登録
```

これで、`/api/staff`以下のすべてのエンドポイントが有効になります。

### `include_router`の仕組み
```python
# routers/staff.py
router = APIRouter(prefix="/api/staff")

@router.get("/")       # /api/staff
@router.get("/{id}")   # /api/staff/{id}

# main.py
app.include_router(staff.router)
# ↑ staff.router内のすべてのルートがappに追加される
```

**まとめて登録できる**ので便利です。

## ステップ5: Swagger UIでテストする

### なぜSwagger UIが便利？

FastAPIは自動でAPIドキュメントを生成します。`http://localhost:8000/docs`にアクセスすると：

- すべてのエンドポイントが一覧表示
- ブラウザから直接APIをテストできる
- リクエスト・レスポンスの例が表示される

**PostmanやcURLを使わなくても、ブラウザだけでテストできます！**

### 実際のテスト手順

#### 1. スタッフ作成（POST）

1. 「POST /api/staff」を開く
2. 「Try it out」をクリック
3. Request bodyを編集：
```json
{
  "name": "田中先生",
  "display_order": 1
}
```

4. 「Execute」をクリック

レスポンス：
```json
{
  "id": "生成されたUUID",
  "name": "田中先生",
  "is_active": true,
  "display_order": 1
}
```

**IDが自動生成された！**これをコピーしておきます。

#### 2. 一覧取得（GET）

1. 「GET /api/staff」を開く
2. 「Try it out」→「Execute」

レスポンス：
```json
[
  {
    "id": "uuid-123",
    "name": "田中先生",
    "is_active": true,
    "display_order": 1
  }
]
```

**作成したスタッフが表示された！**

#### 3. 更新（PUT）

1. 「PUT /api/staff/{staff_id}」を開く
2. `staff_id`にコピーしたIDを貼り付け
3. Request body：
```json
{
  "name": "田中太郎先生"
}
```

4. 「Execute」

レスポンス：
```json
{
  "id": "uuid-123",
  "name": "田中太郎先生",  ← 更新された
  "is_active": true,
  "display_order": 1
}
```

**名前だけ更新された！**

#### 4. 削除（DELETE）

1. 「DELETE /api/staff/{staff_id}」を開く
2. `staff_id`を入力
3. 「Execute」

レスポンス：
```json
{
  "message": "Staff deleted successfully",
  "id": "uuid-123"
}
```

再度一覧取得すると、削除されているのが確認できます。

## トラブルシューティング

### エラー: `TypeError: unsupported operand type(s) for |: 'type' and 'NoneType'`

#### 症状
```
File "routers/staff.py", line 31, in StaffUpdate
    name: str | None = None
TypeError: unsupported operand type(s) for |: 'type' and 'NoneType'
```

#### 原因

Python 3.9では`str | None`の構文がサポートされていません。これはPython 3.10以降の新しい構文です。

#### 解決策：`typing.Optional`を使う

routers/staff.py の修正：

**修正前**：
```python
class StaffUpdate(BaseModel):
    name: str | None = None
    is_active: bool | None = None
    display_order: int | None = None
```

**修正後**：
```python
from typing import List, Optional  # ← Optional を追加

class StaffUpdate(BaseModel):
    name: Optional[str] = None
    is_active: Optional[bool] = None
    display_order: Optional[int] = None
```

#### Python 3.9 vs 3.10+の違い

| Pythonバージョン | 書き方 |
|-----------------|--------|
| Python 3.9以前 | `Optional[str]` |
| Python 3.10以降 | `str \| None` または `Optional[str]` |

どちらも同じ意味ですが、**Python 3.9では`Optional`を使う必要があります**。

#### 対処法のまとめ

1. **推奨**: コードを修正する（5分）
   - `from typing import Optional`を追加
   - `str | None`を`Optional[str]`に変更

2. Pythonバージョンをアップグレード（30分〜1時間）
   - 今回は不要、Python 3.9で問題なく動作

## なぜこの設計が優れているのか

### 1. 責任の分離
```
main.py        → 全体の調整役
database.py    → データベース接続専門
models.py      → データ構造の定義
routers/staff.py → スタッフAPI専門
```

**それぞれが専門の仕事をしている**ので、どこを修正すればいいか明確です。

### 2. スケーラビリティ
```
routers/
├── staff.py      ← 100行
├── customers.py  ← 150行
├── bookings.py   ← 200行
└── reports.py    ← 80行
```

機能が増えても、ファイルを追加するだけ。**main.pyは肥大化しません**。

### 3. テストしやすい

各ルーターは独立しているので、個別にテストできます。
```python
# tests/test_staff.py
def test_create_staff():
    response = client.post("/api/staff", json={"name": "田中先生"})
    assert response.status_code == 201
```

### 4. チ