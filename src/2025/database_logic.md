# 【図解で理解】なぜデータベースでロジックを定義するのか - インデックス、トリガー、RLSの本質

<!--
date = "2025-11-3"
-->

データベーステーブルを作成するSQL構文には、テーブル定義以外にインデックス、トリガー、Row Level Security（RLS）などの機能が含まれています。これらを「アプリケーション側ではなく、なぜデータベース側で定義するのか」を中心に解説します。

## 結論: データベース側で定義する3つの理由

先に結論を示します。データベース側で機能を定義する理由は以下の3つです。

1. **一貫性の保証**: 全てのアプリケーションで同じロジックが適用される
2. **パフォーマンスの最適化**: データベースエンジンが最適な実行計画を選択
3. **セキュリティの強化**: データベースレベルでデータを保護

## なぜアプリケーション側では不十分なのか

データベースには複数の経路からアクセスします。
```
Webアプリ　　 ─┐
モバイルアプリ ─┤
管理画面　    ─┼→ データベース
バッチ処理    ─┤
外部API      ─┘
```

**アプリケーション側で制御する場合の問題:**
```typescript
// Webアプリのコード
function getTodos(userId: string) {
  return db.query('SELECT * FROM todos WHERE user_id = ?', [userId])
}

// モバイルアプリのコード（バグ）
function getTodos(userId: string) {
  // WHERE句を書き忘れた
  return db.query('SELECT * FROM todos')  // 全ユーザーのTodoが取得される
}
```

1つのアプリで`WHERE user_id = ?`を書き忘れると、他のユーザーのデータにアクセスできてしまいます。

**データベース側で制御する場合:**
```sql
-- 一度定義すれば全てのアクセスに適用される
CREATE POLICY "Users can only access their own todos"
  ON todos
  USING (auth.uid() = user_id);
```

どのアプリからアクセスしても自動的に制限されます。

## インデックス（INDEX）- 検索の高速化

### インデックスとは

インデックスは、**データベースの検索を高速化するための仕組み**です。
```sql
CREATE INDEX idx_todos_user_id ON todos(user_id);
CREATE INDEX idx_todos_created_at ON todos(created_at DESC);
CREATE INDEX idx_todos_completed ON todos(completed);
```

### 動作原理の比較

**インデックスなしの場合:**
```
SELECT * FROM todos WHERE user_id = '123';

全行をスキャン（10,000行）→ 所要時間: 100ms
```

**インデックスありの場合:**
```
SELECT * FROM todos WHERE user_id = '123';

インデックスで直接ジャンプ（5行のみ）→ 所要時間: 2ms
```

50倍高速になります。

### アプリケーション側 vs データベース側

#### アプリケーション側の実装
```typescript
// キャッシュを使う例
const todoCache = new Map<string, Todo[]>()

function getTodos(userId: string) {
  if (todoCache.has(userId)) {
    return todoCache.get(userId)
  }
  
  const todos = db.query('SELECT * FROM todos WHERE user_id = ?', [userId])
  todoCache.set(userId, todos)
  return todos
}
```

**問題点:**

- メモリ消費が大きい
- データ更新時にキャッシュの無効化が必要
- 複数サーバーで同期が複雑
- サーバー再起動で消失

#### データベース側の実装
```sql
CREATE INDEX idx_todos_user_id ON todos(user_id);
```

**メリット:**

- 全てのクエリに自動適用
- データベースが最適なメモリ管理
- サーバー再起動後も有効
- 永続的な最適化

## トリガー（TRIGGER）- 自動処理の実行

### トリガーとは

トリガーは、データ変更時に自動的に実行される関数です。
```sql
-- 関数の定義
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- トリガーの設定
CREATE TRIGGER update_todos_updated_at
  BEFORE UPDATE ON todos
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();
```

### 動作の流れ
```
UPDATE 文を実行
    ↓
トリガーが発動
    ↓
updated_at に現在時刻を自動設定
    ↓
UPDATE が実行される
```

### アプリケーション側 vs データベース側

#### アプリケーション側の実装
```typescript
function updateTodo(id: string, title: string) {
  return db.query(
    'UPDATE todos SET title = ?, updated_at = ? WHERE id = ?',
    [title, new Date(), id]
  )
}
```

**問題点:**
```typescript
// Web管理画面
function updateTodo(id: string, data: any) {
  // updated_at を設定している
  return db.query('UPDATE todos SET title = ?, updated_at = ? ...', [...])
}

// バッチ処理
function batchUpdateTodos(ids: string[]) {
  // updated_at の設定を忘れた
  return db.query('UPDATE todos SET completed = true WHERE id IN (?)', [ids])
}
```

- 全ての更新処理で設定が必要
- 書き忘れのリスク
- コードの重複
- 一貫性の欠如

#### データベース側の実装
```sql
CREATE TRIGGER update_todos_updated_at
  BEFORE UPDATE ON todos
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();
```

**メリット:**

- 更新時に自動実行
- 書き忘れ防止
- 1箇所の定義で全体に適用
- 保守性の向上

### トリガーの実用例

#### プロファイルの自動作成
```sql
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.profiles (id, display_name, avatar_url)
  VALUES (
    NEW.id,
    COALESCE(NEW.raw_user_meta_data->>'full_name', NEW.email),
    NEW.raw_user_meta_data->>'avatar_url'
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW
  EXECUTE FUNCTION public.handle_new_user();
```

ユーザー登録時に自動的にプロファイルが作成されます。
```
ユーザーサインアップ
    ↓
auth.users にレコード挿入
    ↓
トリガー発動
    ↓
profiles にレコード自動作成
```

## Row Level Security（RLS）- セキュリティの強化

### RLSとは

Row Level Securityは、データベースの行ごとにアクセス制御を行う技術です。これにより、ユーザーは自分がアクセス権を持つデータのみを閲覧・操作できます。 
```sql
ALTER TABLE todos ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can only access their own todos"
  ON todos
  FOR ALL
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);
```

### 動作原理

RLSは、全てのクエリに暗黙的なWHERE句を追加します。
```sql
-- ユーザーが実行
SELECT * FROM todos;

-- データベース内部で実行
SELECT * FROM todos WHERE auth.uid() = user_id;
```

### アプリケーション側 vs データベース側

#### アプリケーション側の実装
```typescript
function getTodos(userId: string) {
  return db.query('SELECT * FROM todos WHERE user_id = ?', [userId])
}

function updateTodo(userId: string, todoId: string, data: any) {
  return db.query(
    'UPDATE todos SET title = ? WHERE id = ? AND user_id = ?',
    [data.title, todoId, userId]
  )
}
```

**問題点:**

アプリケーション側で万が一コーダーがWHERE user_id = ?を書くのを忘れたらどうなるでしょうか。コードレビューも通過してしまいプロダクトにリリースされてしまいました。結果は、他の企業のデータにアクセスできてしまいます。 
```typescript
// バグのある実装
function deleteTodo(todoId: string) {
  // user_id のチェックを忘れた
  return db.query('DELETE FROM todos WHERE id = ?', [todoId])
  // 他のユーザーのTodoも削除できてしまう
}
```

#### データベース側の実装
```sql
CREATE POLICY "Users can only access their own todos"
  ON todos
  FOR ALL
  USING (auth.uid() = user_id);
```

**メリット:**

RLSを使うとこれを制御することができます。アプリケーションレベルではなくデータベースミドルウェアレベルで企業を跨いだ操作違反を防ぐことが可能になります。 

- WHERE句を書き忘れても安全
- データベースレベルで保護
- アプリケーションコードがシンプル
- 全てのアクセス経路で同じセキュリティ

### RLSの動作例
```typescript
// アプリケーション側（シンプル）
function getTodos() {
  // user_id のフィルタ不要
  return db.query('SELECT * FROM todos')
}

function deleteTodo(todoId: string) {
  // user_id のチェック不要
  return db.query('DELETE FROM todos WHERE id = ?', [todoId])
}
```

データベース側で自動的にフィルタされます。
```sql
-- 実行: SELECT * FROM todos
-- 実際: SELECT * FROM todos WHERE user_id = '現在のユーザーID'

-- 実行: DELETE FROM todos WHERE id = '456'
-- 実際: DELETE FROM todos WHERE id = '456' AND user_id = '現在のユーザーID'
-- 他のユーザーのTodoは削除されない
```

### RLSによる制御例
```sql
-- 他のユーザーのTodoを自分のものにしようとする
UPDATE todos SET user_id = '他のユーザーのID' WHERE id = '123';
-- エラー: new row violates row-level security policy

-- 他のユーザーのTodoを作成しようとする
INSERT INTO todos (user_id, title) VALUES ('他のユーザーのID', 'Todo');
-- エラー: new row violates row-level security policy
```

## パフォーマンスへの影響

RLSを使うことで多少パフォーマンスに影響がでる可能性があります。 適切なインデックスが必要です。
```sql
-- RLS ポリシー
CREATE POLICY "Users can only access their own todos"
  ON todos
  USING (auth.uid() = user_id);

-- user_id にインデックスを作成（重要）
CREATE INDEX idx_todos_user_id ON todos(user_id);
```

インデックスがない場合、RLSによってフィルタが暗黙的に実行されるため、多数のレコードがスキャンされます。しかし、適切なインデックスを追加することで、効率的にレコードが取得できます。 

## 3つの機能の比較

| 機能 | 目的 | アプリ側の問題 | DB側のメリット |
|-----|------|--------------|---------------|
| **インデックス** | 検索高速化 | メモリ消費、同期複雑 | 自動最適化、永続性 |
| **トリガー** | 自動処理 | 書き忘れ、重複 | 自動実行、一貫性 |
| **RLS** | セキュリティ | 書き忘れリスク | DB保護、安全性 |

## 実装の全体像
```
アプリケーション層
├── Webアプリ
├── モバイルアプリ
└── 管理画面
     ↓
データベース層
├── RLS: 自動フィルタ
├── インデックス: 高速化
└── トリガー: 自動処理
     ↓
データ（保護された状態）
```

## SQLの詳細解説

### インデックス
```sql
CREATE INDEX idx_todos_user_id ON todos(user_id);
CREATE INDEX idx_todos_created_at ON todos(created_at DESC);
CREATE INDEX idx_todos_completed ON todos(completed);
```

- **idx_todos_user_id**: ユーザーIDでの検索を高速化
- **idx_todos_created_at DESC**: 作成日時の降順ソートを高速化
- **idx_todos_completed**: 完了状態での絞り込みを高速化

### トリガー
```sql
CREATE TRIGGER update_todos_updated_at
  BEFORE UPDATE ON todos
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();
```

- **BEFORE UPDATE**: 更新前に実行
- **FOR EACH ROW**: 各行ごとに実行
- **EXECUTE FUNCTION**: 実行する関数を指定

### RLS
```sql
CREATE POLICY "Users can only access their own todos"
  ON todos
  FOR ALL
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);
```

- **FOR ALL**: 全操作に適用
- **USING**: SELECT時のフィルタ
- **WITH CHECK**: INSERT/UPDATE時のチェック

## まとめ

データベース側で機能を定義する理由をまとめます。

**一貫性の保証:**
- 全てのアプリケーションで同じロジック
- 実装の書き忘れを防止

**パフォーマンスの最適化:**
- データベースエンジンが最適化
- 永続的な効果

**セキュリティの強化:**
- データベースレベルで保護
- バグによる漏洩を防止

**保守性の向上:**
- 1箇所の修正で全体に適用
- ビジネスロジックの集中管理

## 参考リソース

- [PostgreSQL公式ドキュメント - Row Level Security](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [PostgreSQL公式ドキュメント - インデックス](https://www.postgresql.org/docs/current/indexes.html)
- [PostgreSQL公式ドキュメント - トリガー](https://www.postgresql.org/docs/current/triggers.html)
- [Supabase公式ドキュメント - Row Level Security](https://supabase.com/docs/guides/database/postgres/row-level-security)
- [AWS - マルチテナントデータ分離とPostgreSQL RLS](https://aws.amazon.com/jp/blogs/news/multi-tenant-data-isolation-with-postgresql-row-level-security/)

以上が、データベース側でロジックを定義する理由の解説です。