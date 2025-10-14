# Pythonモデル（SQLAlchemy ORM）完全ガイド
<!--
date = "2025-10-14"
-->

## Pythonモデルとは

Pythonモデルとは、データベースのテーブルをPythonのクラスとして表現したものです。SQLAlchemy ORMを使うことで、データベースをPythonオブジェクトとして扱えるようになります。

### 基本的な対応関係

| データベース | Pythonモデル | 説明 |
|-------------|-------------|------|
| テーブル | クラス | usersテーブル → Userクラス |
| カラム（列） | クラス変数 | nameカラム → User.name |
| レコード（行） | インスタンス | 1人のユーザー → userオブジェクト |

### 簡単な例

データベースのテーブル（SQL）:
```sql
CREATE TABLE users (
    id UUID,
    name VARCHAR(100),
    phone VARCHAR(20)
);
```

Pythonモデル（models.py）:
```python
from sqlalchemy import Column, String
from database import Base

class User(Base):
    __tablename__ = "users"
    
    id = Column(String, primary_key=True)
    name = Column(String(100))
    phone = Column(String(20))
```

実際のデータ（Pythonオブジェクト）:
```python
user = User()
user.id = "uuid-123"
user.name = "田中太郎"
user.phone = "090-1234-5678"

print(user.name)  # "田中太郎"
```

## なぜPythonモデルが必要なのか

### モデルがない場合（直接SQL）

データベースから取得したデータは、タプルやリストで返されます。
```python
# 直接SQLを実行
result = db.execute("SELECT * FROM users WHERE id = '123'")
row = result.fetchone()

# 結果: ('uuid-123', 'U123abc', '田中太郎', '090-1234-5678')

# 使いにくい...
print(row[0])  # uuid-123 ← これが何のデータか分からない
print(row[1])  # U123abc ← これも分からない
print(row[2])  # 田中太郎 ← やっと名前だと分かった

# カラムの順番を覚えておく必要がある
# カラムの順番が変わったらコードも修正が必要
```

#### 問題点

- データが配列やタプルで返される
- どのインデックスが何のデータか分からない
- カラムの順番を覚える必要がある
- タイプミスしてもエラーにならない
- コードが読みにくい

### モデルがある場合（Pythonオブジェクト）

データベースから取得したデータが、Pythonオブジェクトとして返されます。
```python
# Pythonモデルを使って取得
user = db.query(User).filter(User.id == '123').first()

# 使いやすい！
print(user.id)    # uuid-123
print(user.name)  # 田中太郎
print(user.phone) # 090-1234-5678

# IDEの補完が効く
# user.na... と入力すると name が候補に出る
```

#### メリット

- データがオブジェクトとして扱える
- 属性名で直感的にアクセスできる
- IDEの補完機能が使える
- タイプミスをIDEが検出してくれる
- コードが読みやすい

## 具体例で理解する

### データベースのテーブル

Supabaseに以下のようなusersテーブルがあるとします:
```
users テーブル
┌────────────┬──────────────┬──────────┬───────────────┐
│ id         │ line_user_id │ name     │ phone         │
├────────────┼──────────────┼──────────┼───────────────┤
│ uuid-123   │ U123abc      │ 田中太郎  │ 090-1234-5678 │
│ uuid-456   │ U456def      │ 佐藤花子  │ 090-9876-5432 │
│ uuid-789   │ U789ghi      │ 鈴木一郎  │ 090-1111-2222 │
└────────────┴──────────────┴──────────┴───────────────┘
```

### Pythonモデルの定義

このテーブルに対応するPythonモデルを作成します。
```python
# models.py
from sqlalchemy import Column, String, DateTime
from sqlalchemy.sql import func
from database import Base
import uuid

def generate_uuid():
    """UUID生成関数"""
    return str(uuid.uuid4())

class User(Base):
    """ユーザー情報のモデル"""
    
    # テーブル名を指定
    __tablename__ = "users"
    
    # カラムの定義
    id = Column(String, primary_key=True, default=generate_uuid)
    line_user_id = Column(String(100), unique=True, nullable=False, index=True)
    display_name = Column(String(100))
    phone = Column(String(20))
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), server_default=func.now(), onupdate=func.now())
    
    def __repr__(self):
        """オブジェクトを print した時の表示"""
        return f"<User(id={self.id}, name={self.display_name})>"
```

#### コードの説明

##### `__tablename__`

データベースのどのテーブルに対応するか指定します。
```python
__tablename__ = "users"  # ← usersテーブルを指す
```

##### `Column`

テーブルのカラム（列）を定義します。
```python
id = Column(String, primary_key=True, default=generate_uuid)
#    ^^^^^^  ^^^^^^  ^^^^^^^^^^^^       ^^^^^
#    カラム名  型      主キー             デフォルト値
```

主なカラムオプション:

| オプション | 説明 | 例 |
|-----------|------|-----|
| `primary_key=True` | 主キー（一意の識別子） | `id` カラム |
| `unique=True` | 重複不可 | LINE ID |
| `nullable=False` | NULL不可（必須項目） | 名前 |
| `index=True` | インデックスを作成（検索高速化） | LINE ID |
| `default=値` | デフォルト値 | UUID生成 |

##### データ型

主なデータ型:

| SQLAlchemy型 | PostgreSQL型 | Python型 | 用途 |
|-------------|-------------|---------|------|
| `String(100)` | VARCHAR(100) | str | 短い文字列 |
| `Text` | TEXT | str | 長い文字列 |
| `Integer` | INTEGER | int | 整数 |
| `Boolean` | BOOLEAN | bool | True/False |
| `DateTime` | TIMESTAMP | datetime | 日時 |
| `JSONB` | JSONB | dict | JSON |

##### `__repr__`

オブジェクトをprintした時の表示を定義します。
```python
def __repr__(self):
    return f"<User(id={self.id}, name={self.display_name})>"

# 実行例
user = User(id="123", display_name="田中太郎")
print(user)  # <User(id=123, name=田中太郎)>
```

### モデルの使い方

#### 1. 新しいユーザーを作成
```python
from database import SessionLocal
from models import User

# データベースセッションを開始
db = SessionLocal()

# 新しいユーザーオブジェクトを作成
new_user = User(
    line_user_id="U789ghi",
    display_name="鈴木一郎",
    phone="090-1111-2222"
)

# データベースに追加（まだ保存されていない）
db.add(new_user)

# データベースに保存（コミット）
db.commit()

# 最新の状態を取得（IDが自動生成された場合など）
db.refresh(new_user)

print(f"登録完了: {new_user.id}")  # uuid-789
```

SQLに変換されると:
```sql
INSERT INTO users (id, line_user_id, display_name, phone, created_at)
VALUES ('uuid-789', 'U789ghi', '鈴木一郎', '090-1111-2222', NOW());
```

#### 2. ユーザーを検索
```python
# 1件取得
user = db.query(User).filter(
    User.display_name == "田中太郎"
).first()

if user:
    print(f"名前: {user.display_name}")  # 田中太郎
    print(f"電話: {user.phone}")         # 090-1234-5678
else:
    print("ユーザーが見つかりません")
```

SQLに変換されると:
```sql
SELECT * FROM users WHERE display_name = '田中太郎' LIMIT 1;
```

複数条件での検索:
```python
# LINE IDと電話番号で検索
user = db.query(User).filter(
    User.line_user_id == "U123abc",
    User.phone == "090-1234-5678"
).first()
```

SQLに変換されると:
```sql
SELECT * FROM users 
WHERE line_user_id = 'U123abc' AND phone = '090-1234-5678' 
LIMIT 1;
```

#### 3. ユーザーを更新
```python
# ユーザーを取得
user = db.query(User).filter(User.id == "uuid-123").first()

# 電話番号を更新
user.phone = "090-9999-8888"

# データベースに保存
db.commit()

print(f"更新完了: {user.phone}")  # 090-9999-8888
```

SQLに変換されると:
```sql
UPDATE users SET phone = '090-9999-8888' WHERE id = 'uuid-123';
```

#### 4. ユーザーを削除
```python
# ユーザーを取得
user = db.query(User).filter(User.id == "uuid-123").first()

if user:
    # 削除
    db.delete(user)
    db.commit()
    print("削除完了")
```

SQLに変換されると:
```sql
DELETE FROM users WHERE id = 'uuid-123';
```

#### 5. 全ユーザーを取得
```python
# 全ユーザーを取得
all_users = db.query(User).all()

# ループで表示
for user in all_users:
    print(f"{user.display_name}: {user.phone}")

# 出力:
# 田中太郎: 090-1234-5678
# 佐藤花子: 090-9876-5432
# 鈴木一郎: 090-1111-2222
```

SQLに変換されると:
```sql
SELECT * FROM users;
```

#### 6. 条件を満たすユーザーを複数取得
```python
# 電話番号が登録されているユーザーのみ
users_with_phone = db.query(User).filter(
    User.phone != None
).all()

# 名前で部分一致検索（「田中」を含む）
users = db.query(User).filter(
    User.display_name.like("%田中%")
).all()

# 件数を取得
count = db.query(User).filter(
    User.phone != None
).count()
print(f"電話番号登録済み: {count}件")
```

## SQLAlchemy ORMとは

### ORMの意味

ORM = Object-Relational Mapping（オブジェクト関係マッピング）

データベースのテーブルとPythonのクラスを対応付ける技術です。
```
データベース          →  Python
───────────────────────────────
テーブル（users）     →  クラス（User）
カラム（name）        →  属性（user.name）
レコード（1行）       →  インスタンス（userオブジェクト）
```

### SQLAlchemyとは

Pythonで最も人気のあるORMライブラリです。

#### 主な機能

- データベース接続管理
- テーブル定義（モデル）
- CRUD操作（作成・読取・更新・削除）
- リレーション（テーブル間の関連）
- マイグレーション（Alembicと連携）

### ORMを使うメリット

#### 1. Pythonのコードだけでデータベース操作

SQLを直接書く必要がありません。
```python
# ❌ 生のSQL
db.execute("INSERT INTO users (name, phone) VALUES ('田中', '090-1234-5678')")

# ✅ ORM
user = User(display_name="田中", phone="090-1234-5678")
db.add(user)
db.commit()
```

#### 2. タイプミスが減る

IDEの補完機能が使えるため、タイプミスを防げます。
```python
# IDEで user. と入力すると、以下が候補に表示される:
# - user.id
# - user.display_name
# - user.phone
# - user.line_user_id

# タイプミスをIDEが検出
user.nam  # ← IDEが警告: 'nam' は存在しません
```

#### 3. データベースの種類を気にしなくていい

PostgreSQL、MySQL、SQLiteなど、データベースを切り替えてもコードはほぼ同じです。
```python
# PostgreSQL用の設定
DATABASE_URL = "postgresql://user:pass@host/db"

# MySQL用の設定に変えても、モデルのコードは同じ
DATABASE_URL = "mysql://user:pass@host/db"
```

#### 4. コードが読みやすい

SQLよりも直感的で、Pythonらしいコードになります。
```python
# ❌ SQL: どこまでがテーブル名？カラム名？
SELECT u.name, u.phone FROM users u WHERE u.id = '123' AND u.phone IS NOT NULL

# ✅ ORM: 読みやすい
user = db.query(User).filter(
    User.id == '123',
    User.phone != None
).first()
```

#### 5. SQLインジェクション攻撃を防げる

ORMは自動的にパラメータをエスケープするため、セキュリティが向上します。
```python
# ❌ 危険: SQLインジェクションの可能性
name = request.get("name")  # ユーザー入力
db.execute(f"SELECT * FROM users WHERE name = '{name}'")

# ✅ 安全: ORMが自動でエスケープ
user = db.query(User).filter(User.display_name == name).first()
```

## 実践例: FastAPIでの使い方

### FastAPIのエンドポイントで使う
```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from database import get_db
from models import User

router = APIRouter()

@router.get("/users/{user_id}")
def get_user(user_id: str, db: Session = Depends(get_db)):
    """ユーザー詳細取得"""
    user = db.query(User).filter(User.id == user_id).first()
    
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    
    return {
        "id": user.id,
        "name": user.display_name,
        "phone": user.phone
    }

@router.post("/users")
def create_user(line_user_id: str, name: str, db: Session = Depends(get_db)):
    """ユーザー作成"""
    # 既存チェック
    existing = db.query(User).filter(
        User.line_user_id == line_user_id
    ).first()
    
    if existing:
        raise HTTPException(status_code=400, detail="User already exists")
    
    # 新規作成
    new_user = User(
        line_user_id=line_user_id,
        display_name=name
    )
    db.add(new_user)
    db.commit()
    db.refresh(new_user)
    
    return {"id": new_user.id, "message": "User created"}
```

## よくある質問

### Q1: モデルとテーブルの違いは？

- **テーブル**: データベース上の実際のデータ構造
- **モデル**: テーブルをPythonで表現したクラス

モデルはテーブルの「設計図」のようなものです。

### Q2: `db.commit()` を忘れるとどうなる？

データが保存されません。commitしないと、変更は一時的なものとして扱われます。
```python
user = User(name="田中")
db.add(user)
# db.commit()  ← これを忘れると保存されない

# データベースを確認しても、田中さんは存在しない
```

### Q3: `primary_key` とは？

テーブル内で各レコードを一意に識別するためのカラムです。通常は `id` カラムを使います。
```python
id = Column(String, primary_key=True)  # ← 重複しない一意の値
```

### Q4: `nullable=False` とは？

NULLを許可しない（必須項目）という意味です。
```python
name = Column(String(100), nullable=False)  # ← 必須
phone = Column(String(20))  # ← 任意（NULLでもOK）
```

## まとめ

Pythonモデル（SQLAlchemy ORM）を使うことで:

- データベースをPythonオブジェクトとして扱える
- SQLを直接書かなくていい
- コードが読みやすく、保守しやすい
- IDEの補完が効いて開発効率UP
- タイプミスが減る
- セキュリティが向上

FastAPIとSupabaseを使ったLINE Bot開発では、ORMが必須の技術です。

## 参考情報

- SQLAlchemy公式ドキュメント: https://docs.sqlalchemy.org/
- FastAPI + SQLAlchemy: https://fastapi.tiangolo.com/tutorial/sql-databases/
- SQLAlchemy ORM チュートリアル: https://docs.sqlalchemy.org/en/20/orm/quickstart.html