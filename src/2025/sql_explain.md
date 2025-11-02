# SupabaseのSQL構文解説 - データベース設計を理解する

<!--
date = "2025-11-02"
-->

データベーステーブルを作成するSQL構文には、多くの専門用語と概念が含まれています。この記事では、Todoアプリで使用する2つのテーブル（`profiles` と `todos`）の設計を、1行ずつ解説します。

## Supabaseの認証システムを理解する

SQL構文を読む前に、Supabaseの認証システムを理解する必要があります。

### auth.usersテーブルとは

Supabaseには `auth.users` という特別なテーブルが存在します。このテーブルの特徴は以下の通りです。

- Supabaseが自動的に作成・管理
- ユーザーの認証情報（メール、パスワード等）を保存
- 開発者は直接アクセスできない
- Supabaseの認証APIを通じて操作
```
auth.users (Supabaseが自動管理)
├── id (UUID) ← 全てのユーザー情報の基準となるID
├── email
├── encrypted_password
└── その他の認証情報
```

### なぜ追加のテーブルが必要なのか

`auth.users` は認証情報のみを管理します。ニックネームやアバター画像などの追加情報を保存するには、別のテーブル（`profiles`）が必要です。

## profilesテーブルの解説

### テーブルの役割

`profiles` テーブルは、ユーザーの追加情報を保存します。
```
認証情報: auth.users (Supabase管理)
プロフィール情報: profiles (開発者が管理)
```

### SQL構文
```sql
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  display_name TEXT,
  avatar_url TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 1行目: テーブル作成コマンド
```sql
CREATE TABLE profiles (
```

PostgreSQLに対して、`profiles` という名前の新しいテーブルを作成するよう指示します。

### 2行目: id列の定義
```sql
id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
```

この1行には5つの重要な要素が含まれています。

#### ① id UUID

- **意味**: id列のデータ型はUUID
- **UUID**: 一意の識別子（例: `123e4567-e89b-12d3-a456-426614174000`）

#### ② PRIMARY KEY

- **意味**: この列が主キー
- **主キー**: テーブル内で各レコードを一意に識別する列

#### ③ REFERENCES auth.users(id)

- **意味**: この列は `auth.users` テーブルの `id` 列を参照
- **外部キー制約**: 2つのテーブル間の関係を定義
```
auth.users
├── id: "123e4567-..."
├── email: "user@example.com"
└── ...
     ↓ 参照
profiles
├── id: "123e4567-..." ← 同じ値
├── display_name: "山田太郎"
└── ...
```

#### ④ ON DELETE CASCADE

- **意味**: 参照先が削除されたとき、このレコードも自動削除
- **動作**: `auth.users` のユーザーが削除されると、`profiles` のレコードも削除される

**具体例:**
```
ステップ1: ユーザー削除
auth.users からユーザー "123e4567-..." を削除

ステップ2: 自動削除（CASCADE）
profiles からレコード "123e4567-..." も自動削除
```

これにより、孤立したデータを防ぎます。

### 3-4行目: プロフィール情報
```sql
display_name TEXT,
avatar_url TEXT,
```

- **display_name**: ユーザーの表示名（例: "山田太郎"）
- **avatar_url**: アバター画像のURL（例: "https://example.com/avatar.jpg"）
- **TEXT型**: 文字列を保存するデータ型

### 5-6行目: タイムスタンプ
```sql
created_at TIMESTAMPTZ DEFAULT NOW(),
updated_at TIMESTAMPTZ DEFAULT NOW()
```

- **created_at**: レコードの作成日時
- **updated_at**: レコードの最終更新日時
- **TIMESTAMPTZ**: タイムゾーン付きタイムスタンプ型
- **DEFAULT NOW()**: デフォルト値として現在時刻を自動設定

## todosテーブルの解説

### テーブルの役割

`todos` テーブルは、各ユーザーのTodoアイテムを保存します。

### SQL構文
```sql
CREATE TABLE todos (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  description TEXT,
  completed BOOLEAN DEFAULT FALSE,
  due_date TIMESTAMPTZ,
  priority TEXT CHECK (priority IN ('low', 'medium', 'high')),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 2行目: id列の定義
```sql
id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
```

- **id UUID**: id列はUUID型
- **PRIMARY KEY**: 主キー
- **DEFAULT gen_random_uuid()**: PostgreSQLが自動的にUUIDを生成

`profiles` テーブルとの違いは、UUIDを自動生成する点です。

### 3行目: user_id列の定義（最重要）
```sql
user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
```

この列が、Todoとユーザーを紐付ける最も重要な列です。

#### ① user_id UUID

- **意味**: ユーザーのIDを保存する列

#### ② NOT NULL

- **意味**: この列は必須。NULL値を許可しない
- **理由**: 全てのTodoには必ず作成者が存在する必要がある

#### ③ REFERENCES auth.users(id)

- **意味**: `auth.users` テーブルの `id` 列を参照
- **効果**: Todoの作成者を特定

#### ④ ON DELETE CASCADE

- **意味**: ユーザー削除時に、そのユーザーのTodoも自動削除

**具体例:**
```
auth.users
└── id: "123e4567-..."
     ↓ 参照
todos
├── id: "a1b2c3d4-..."
├── user_id: "123e4567-..." ← 作成者を識別
├── title: "買い物"
└── ...
```

ユーザー `123e4567-...` が削除されると、`user_id` が `123e4567-...` の全てのTodoが自動削除されます。

### 4-5行目: Todoの内容
```sql
title TEXT NOT NULL,
description TEXT,
```

- **title**: Todoのタイトル（必須）
- **description**: Todoの詳細説明（省略可能）

`NOT NULL` の有無に注目します。titleは必須、descriptionは省略可能です。

### 6行目: 完了状態
```sql
completed BOOLEAN DEFAULT FALSE,
```

- **completed**: Todoが完了したかどうか
- **BOOLEAN型**: trueまたはfalseの値のみ
- **DEFAULT FALSE**: 新規作成時は未完了

### 7行目: 期限
```sql
due_date TIMESTAMPTZ,
```

Todoの期限を保存します。省略可能です（`NOT NULL` が付いていない）。

### 8行目: 優先度
```sql
priority TEXT CHECK (priority IN ('low', 'medium', 'high')),
```

- **priority**: 優先度（低、中、高）
- **CHECK制約**: 'low'、'medium'、'high' のいずれかのみを許可
- **効果**: 不正な値の挿入を防ぐ

**CHECK制約の例:**
```sql
-- 成功: 許可された値
INSERT INTO todos (priority) VALUES ('low');

-- 失敗: 許可されていない値
INSERT INTO todos (priority) VALUES ('urgent'); -- エラー
```

## データの関係性を図解する

3つのテーブルの関係をまとめます。
```
auth.users (Supabaseが管理)
  ├── id: "user-001" ← 基準となるID
  ├── email: "user@example.com"
  └── encrypted_password: "..."
       ↓ 参照（REFERENCES）
profiles (開発者が管理)
  ├── id: "user-001" ← auth.users.idと同じ
  ├── display_name: "山田太郎"
  └── avatar_url: "https://..."
       ↓
todos (開発者が管理)
  ├── id: "todo-001"
  ├── user_id: "user-001" ← auth.users.idを参照
  ├── title: "買い物"
  ├── completed: false
  └── ...
  
  ├── id: "todo-002"
  ├── user_id: "user-001" ← 同じユーザー
  ├── title: "掃除"
  └── ...
```

### 削除時の動作フロー

ユーザー削除時の動作を時系列で確認します。
```
ステップ1: ユーザー削除コマンド実行
DELETE FROM auth.users WHERE id = 'user-001';

ステップ2: CASCADE動作（自動）
profiles のレコード削除（id = 'user-001'）
todos のレコード削除（user_id = 'user-001' の全て）

結果: 関連データが全て削除され、孤立データが残らない
```

## データ操作の実例

実際にデータを操作する例を示します。

### 新規ユーザー登録時
```sql
-- 1. Supabaseが自動的に auth.users にレコードを作成
-- id: "123e4567-..."
-- email: "user@example.com"

-- 2. トリガーが自動的に profiles を作成（後のガイドで実装）
INSERT INTO profiles (id, display_name, avatar_url)
VALUES (
  '123e4567-e89b-12d3-a456-426614174000',
  'ユーザー名',
  NULL
);
```

### Todo作成時
```sql
INSERT INTO todos (user_id, title, description, completed, due_date, priority)
VALUES (
  '123e4567-e89b-12d3-a456-426614174000',  -- ログイン中のユーザーID
  '買い物',
  '牛乳を買う',
  false,
  '2025-11-02 18:00:00+09',
  'medium'
);
```

`id` 列は指定していません。`gen_random_uuid()` が自動的にUUIDを生成するためです。

### Todo検索時
```sql
-- 特定ユーザーのTodoを全て取得
SELECT * FROM todos 
WHERE user_id = '123e4567-e89b-12d3-a456-426614174000';

-- 未完了のTodoのみ取得
SELECT * FROM todos 
WHERE user_id = '123e4567-e89b-12d3-a456-426614174000'
  AND completed = false;
```

## 用語集

この記事で使用した専門用語をまとめます。

| 用語 | 意味 | 例 |
|-----|------|---|
| **UUID** | 一意の識別子 | `123e4567-e89b-12d3-a456-426614174000` |
| **PRIMARY KEY** | 主キー。レコードを一意に識別 | `id UUID PRIMARY KEY` |
| **REFERENCES** | 外部キー制約。他のテーブルを参照 | `REFERENCES auth.users(id)` |
| **ON DELETE CASCADE** | 親削除時に子も自動削除 | ユーザー削除時にTodoも削除 |
| **NOT NULL** | NULL値を許可しない | `title TEXT NOT NULL` |
| **DEFAULT** | デフォルト値の設定 | `DEFAULT FALSE` |
| **CHECK** | 値の制約 | `CHECK (priority IN ('low', 'medium', 'high'))` |
| **TIMESTAMPTZ** | タイムゾーン付きタイムスタンプ | `2025-11-02 18:00:00+09` |

## まとめ

この記事で解説した内容をまとめます。

**テーブルの役割:**
- `auth.users`: 認証情報（Supabase管理）
- `profiles`: プロフィール情報（開発者管理）
- `todos`: Todoアイテム（開発者管理）

**外部キー制約の仕組み:**
- `REFERENCES` でテーブル間の関係を定義
- `ON DELETE CASCADE` で親削除時に子も自動削除
- 孤立データを防止

**データ型と制約:**
- `UUID`: 一意の識別子
- `TEXT`: 文字列
- `BOOLEAN`: true/false
- `TIMESTAMPTZ`: タイムゾーン付き日時
- `NOT NULL`: 必須項目
- `DEFAULT`: デフォルト値
- `CHECK`: 値の制限

**参考リソース:**
- [PostgreSQL公式ドキュメント - CREATE TABLE](https://www.postgresql.org/docs/current/sql-createtable.html)
- [PostgreSQL公式ドキュメント - 外部キー](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-FK)
- [Supabase公式ドキュメント - データベース](https://supabase.com/docs/guides/database/overview)

以上がデータベーステーブル設計の解説です。次のステップでは、これらのテーブルを使用したアプリケーション開発を進めます。