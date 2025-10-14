# SupabaseでLINEBot用のデータベースを構築する手順
<!--
date = "2025-10-14"
-->
## 概要

LINE Botで予約データを保存するために、Supabaseを使ってデータベースを構築します。Supabaseは無料で使えるPostgreSQLデータベースで、クラウド上で簡単にデータベースを作成できます。

- 所要時間: 1-2時間
- 前提条件: Github(or Google)アカウント（サインアップ用）
- 目的: LINE Botで使うデータベースを作成し、FastAPIから接続する

## Supabaseとは

Supabaseは「オープンソースのFirebase代替」として人気のサービスです。

### 主な特徴

- 完全無料で始められる（クレジットカード不要）
- PostgreSQLデータベースが使える
- 管理画面が使いやすい
- 自動でバックアップされる
- 世界中のデータセンターから選べる

### 無料プランの制限

- データベース容量: 500MB
- 月間転送量: 1GB
- APIリクエスト: 無制限
- 同時接続: 60

個人開発や小規模なLINE Botには十分なスペックです。

## 1. Supabaseプロジェクト作成

### アカウント作成

1. https://supabase.com/ にアクセス
2. 「Start your project」をクリック
3. GitHubアカウントでサインアップ（推奨）

GitHubアカウントを持っていない場合は、メールアドレスでもサインアップできます。

### プロジェクト作成

サインアップ後、新規プロジェクトを作成します。

1. 「New project」をクリック
2. 以下の項目を入力:

| 項目 | 設定内容 | 説明 |
|------|----------|------|
| Name | line-booking-bot | プロジェクト名（任意） |
| Database Password | 強力なパスワード | **必ずメモする** |
| Region | Northeast Asia (Tokyo) | 日本に最も近いサーバー |
| Pricing Plan | Free | 無料プラン |

3. 「Create new project」をクリック

プロジェクトの作成には2-3分かかります。コーヒーを飲んで待ちましょう。

### Database URL取得

目的: FastAPIからデータベースに接続するためのURLを取得する。

1. プロジェクト作成完了後、上のタブの「Connect」を開く
2. 「Connection string」セクションで「URI」を選択
3. 表示されたURLをコピー

URLの形式:
```
postgresql://postgres.xxxxx:[YOUR-PASSWORD]@xxx.supabase.co:5432/postgres
```

`[YOUR-PASSWORD]`の部分を、先ほど設定したパスワードに置き換えます。

例:
```
postgresql://postgres.abcdef:MySecurePassword123@db.abcdefghij.supabase.co:5432/postgres
```

このURLを .env ファイルに保存します:
```
DATABASE_URL=postgresql://postgres.abcdef:MySecurePassword123@db.abcdefghij.supabase.co:5432/postgres
```

## 2. テーブル設計

データベースには3つのテーブルを作成します。

### テーブル構成

| テーブル名 | 用途 | 主なカラム |
|-----------|------|-----------|
| users | LINE利用者の情報 | LINE ID、名前、電話番号 |
| staff | スタッフ情報 | 名前、有効/無効 |
| bookings | 予約情報 | 日時、ステータス、メモ |

### ER図（関係図）
```
users (顧客)
  ↓ 1対多
bookings (予約)
  ↓ 多対1
staff (スタッフ)
```

1人の顧客は複数の予約ができ、1人のスタッフも複数の予約を受け持てます。

## 3. テーブル作成

### usersテーブル

目的: LINE Botを使う顧客の情報を保存する。

1. Supabase Dashboard → 左メニュー「SQL Editor」
2. 「New query」をクリック
3. 以下のSQLを貼り付け:
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    line_user_id VARCHAR(100) UNIQUE NOT NULL,
    display_name VARCHAR(100),
    phone VARCHAR(20),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_users_line_user_id ON users(line_user_id);
```

4. 「RUN」をクリック

#### カラム説明

- `id`: 一意の識別子（UUID形式で自動生成）
- `line_user_id`: LINEのユーザーID（重複不可）
- `display_name`: 表示名
- `phone`: 電話番号
- `created_at`: 作成日時（自動設定）
- `updated_at`: 更新日時（自動更新）

インデックスを作成することで、LINE IDでの検索が高速になります。

### staffテーブル

目的: 予約を受け付けるスタッフの情報を保存する。

SQL Editorで新しいクエリを作成し、以下を実行:
```sql
CREATE TABLE staff (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    is_active BOOLEAN DEFAULT true,
    display_order INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_staff_active ON staff(is_active);
```

#### カラム説明

- `id`: 一意の識別子
- `name`: スタッフ名
- `is_active`: 有効/無効フラグ（退職したスタッフを無効化）
- `display_order`: 表示順（小さい数字が先に表示される）
- `created_at`: 作成日時
- `updated_at`: 更新日時

### bookingsテーブル

目的: 予約情報を保存する。
```sql
CREATE TABLE bookings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    staff_id UUID REFERENCES staff(id) ON DELETE SET NULL,
    booking_date TIMESTAMP WITH TIME ZONE NOT NULL,
    status VARCHAR(20) DEFAULT 'confirmed',
    service_name VARCHAR(100),
    notes TEXT,
    reminder_settings JSONB DEFAULT '{"day_before": true, "hour_before": false}'::jsonb,
    reminder_sent JSONB DEFAULT '{"day_before": false, "hour_before": false}'::jsonb,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_bookings_user_id ON bookings(user_id);
CREATE INDEX idx_bookings_staff_id ON bookings(staff_id);
CREATE INDEX idx_bookings_date ON bookings(booking_date);
CREATE INDEX idx_bookings_status ON bookings(status);
```

#### カラム説明

- `id`: 一意の識別子
- `user_id`: 予約した顧客のID（usersテーブルへの参照）
- `staff_id`: 担当スタッフのID（staffテーブルへの参照）
- `booking_date`: 予約日時
- `status`: ステータス（confirmed: 確定、cancelled: キャンセル、completed: 完了）
- `service_name`: サービス名（例: 鍼治療、マッサージ）
- `notes`: メモ欄
- `reminder_settings`: リマインダー設定（JSON形式）
- `reminder_sent`: リマインダー送信履歴（JSON形式）
- `created_at`: 作成日時
- `updated_at`: 更新日時

#### 外部キー制約の説明

- `ON DELETE CASCADE`: 顧客が削除されたら、その顧客の予約も全て削除
- `ON DELETE SET NULL`: スタッフが削除されても、予約は残る（staff_idがNULLになる）

## 4. テーブル確認

左メニューの「Table Editor」をクリックすると、作成したテーブルが表示されます。

各テーブルをクリックして、カラムが正しく作成されているか確認してください。

## 5. FastAPIとの接続設定

### database.py更新

目的: FastAPIからSupabaseデータベースに接続する設定を行う。

ローカルのプロジェクトフォルダで database.py を以下の内容に更新:
```python
"""
データベース接続設定（Supabase PostgreSQL）
"""
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
from dotenv import load_dotenv

# 環境変数読み込み
load_dotenv()

# Supabase Database URL
DATABASE_URL = os.getenv("DATABASE_URL")

if not DATABASE_URL:
    raise ValueError("DATABASE_URL が設定されていません。.env ファイルを確認してください。")

# エンジン作成
engine = create_engine(
    DATABASE_URL,
    echo=True,  # SQLログを表示（開発時のみ）
    pool_pre_ping=True  # 接続チェック
)

# セッション作成
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# ベースクラス
Base = declarative_base()

# データベース接続を取得する関数（FastAPIで使用）
def get_db():
    """
    データベースセッションを取得
    使用後は自動的にクローズされる
    """
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### .env更新

.env ファイルに DATABASE_URL を追加:
```
LINE_CHANNEL_ACCESS_TOKEN=your_token_here
LINE_CHANNEL_SECRET=your_secret_here
DATABASE_URL=postgresql://postgres.xxxxx:YOUR_PASSWORD@xxx.supabase.co:5432/postgres
```

**重要**: .gitignore に .env が含まれていることを必ず確認してください。

## 6. 接続テスト

目的: FastAPIからSupabaseに正しく接続できるか確認する。

ターミナルで以下を実行:
```bash
python -c "from database import engine; engine.connect(); print('✅ データベース接続成功')"
```

成功すると以下のように表示されます:
```
✅ データベース接続成功
```

### エラーが出た場合

#### パスワードが間違っている
```
could not connect to server: FATAL: password authentication failed
```

対処法: .env ファイルの DATABASE_URL のパスワード部分を確認。

#### DATABASE_URLが設定されていない
```
ValueError: DATABASE_URL が設定されていません
```

対処法: .env ファイルが存在し、DATABASE_URL が記載されているか確認。

#### ネットワークエラー
```
could not connect to server: Connection refused
```

対処法:

- インターネット接続を確認
- ファイアウォール設定を確認
- SupabaseのプロジェクトがPausedになっていないか確認

## 7. Renderへのデプロイ

目的: 本番環境でもSupabaseデータベースを使えるようにする。

### 環境変数の設定

1. Render Dashboard → line-booking-bot サービスを選択
2. 左メニュー「Environment」をクリック
3. 「Add Environment Variable」をクリック
4. 以下を追加:

| Key | Value |
|-----|-------|
| DATABASE_URL | SupabaseのDatabase URL |

5. 「Save Changes」をクリック

Renderが自動で再デプロイを開始します。

### デプロイ確認

デプロイログで以下を確認:
```
==> Successfully installed ...
==> Your service is live 🎉
```

ブラウザで以下にアクセス:
```
https://line-booking-bot-xxxx.onrender.com/health
```

`{"status":"healthy"}` が表示されれば成功。

## 8. データの確認方法

Supabaseの管理画面でデータを直接確認できます。

### Table Editorでの確認

1. Supabase Dashboard → Table Editor
2. テーブル名（users、staff、bookings）を選択
3. データが表示される

### 手動でデータ追加

テスト用のデータを手動で追加できます。

1. Table Editor → staff テーブルを選択
2. 「Insert」→「Insert row」
3. 以下を入力:

| カラム | 値 |
|--------|-----|
| name | 田中先生 |
| is_active | true |
| display_order | 1 |

4. 「Save」

これでスタッフデータが1件追加されました。

## トラブルシューティング

### テーブル作成でエラーが出る

症状: SQL実行時に構文エラー

対処法:

- SQLをコピペする際に余分な空白や改行が入っていないか確認
- 既に同じ名前のテーブルが存在していないか確認（Table Editorで確認）

既存テーブルを削除する場合:
```sql
DROP TABLE IF EXISTS bookings CASCADE;
DROP TABLE IF EXISTS staff CASCADE;
DROP TABLE IF EXISTS users CASCADE;
```

その後、再度テーブル作成SQLを実行。

### 接続テストが失敗する

症状: データベースに接続できない

確認項目:

- .env ファイルにDATABASE_URLが記載されているか
- パスワードが正しいか（Supabaseで再確認）
- DATABASE_URLのコピペミスがないか（前後に空白が入っていないか）

### Renderデプロイ後にデータベースエラー

症状: 本番環境でデータベース接続エラー

確認項目:

- Render の Environment に DATABASE_URL が設定されているか
- DATABASE_URL の値が正しいか
- Renderのログでエラー内容を確認

## まとめ

これでSupabaseデータベースの構築が完了しました。

### 完了したこと

- Supabaseプロジェクト作成
- 3つのテーブル作成（users、staff、bookings）
- FastAPIからの接続設定
- ローカル環境での接続確認
- Renderへのデプロイ

### 次のステップ

次はFastAPIでCRUD APIを実装します。これにより、プログラムからデータベースを操作できるようになります。

## 参考情報

- Supabase Documentation: https://supabase.com/docs
- PostgreSQL公式ドキュメント: https://www.postgresql.org/docs/
- SQLAlchemy Documentation: https://docs.sqlalchemy.org/