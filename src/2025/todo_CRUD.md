# Phase 3: Todo機能実装ガイド - 初心者がつまずく5つの罠と回避法

<!--
date = "2025-11-06"
-->

## なぜTodoアプリの実装で9割の初心者が挫折するのか

Todo Listアプリは「簡単そう」に見えます。しかし、実際に実装すると、多くの初心者が以下の壁にぶつかります。

- データベースとの接続でエラーが出る
- 作成したTodoが表示されない
- 他人のTodoが見えてしまう
- 編集ボタンを押しても何も起きない

この記事では、これらの問題を回避しながら、Todo機能を実装する方法を解説します。

## この記事で実装する機能

以下の4つの基本操作（CRUD）を実装します。

- **Create**: Todoを作成する
- **Read**: Todoを表示する
- **Update**: Todoを編集する
- **Delete**: Todoを削除する

これらは、どのWebアプリケーションでも必要になる基本機能です。

---

## 罠1: 型定義を理解せずに進めると、後で地獄を見る

### 型とは何か

TypeScriptでは、データに「型」を付けます。型は、データの形を決めるルールです。

例えば、Todoには以下の情報があります。

- タイトル（文字列）
- 完了状態（true/false）
- 期限（日付）

型を定義しないと、間違ったデータが入り込み、アプリが壊れます。

### 実装: 型エイリアスの追加

`lib/types/database.types.ts`の最後に以下を追加します。
```typescript
export type Todo = Database['public']['Tables']['todos']['Row']
export type TodoInsert = Database['public']['Tables']['todos']['Insert']
export type TodoUpdate = Database['public']['Tables']['todos']['Update']
```

**重要**: これをやらないと、後で「型が合わない」エラーに悩まされます。

---

## 罠2: Server Actionsの仕組みを知らないと、なぜ動くのか理解できない

### Server Actionsとは

Server Actionsは、サーバー側で実行される関数です。データベースへのアクセスは、必ずサーバー側で行います。

理由は2つです。

- セキュリティ（接続情報を隠す）
- データの整合性（誰でも勝手に変更できないようにする）

### 実装: getTodos関数

`app/actions/todos.ts`を作成し、以下を記述します。
```typescript
'use server'

export async function getTodos(filter?: 'all' | 'active' | 'completed') {
  const supabase = await createClient()
  const { data: { session } } = await supabase.auth.getSession()
  
  if (!session) {
    throw new Error('認証が必要です')
  }
  
  let query = supabase
    .from('todos')
    .select('*')
    .eq('user_id', session.user.id)
```

**重要**: `eq('user_id', session.user.id)`がないと、他人のTodoが見えてしまいます。

---

## 罠3: revalidatePathを忘れると、画面が更新されない

### なぜ画面が更新されないのか

Next.jsは、一度取得したデータをキャッシュします。Todoを作成しても、キャッシュが残っているため、画面に反映されません。

### 解決策: revalidatePathを使う
```typescript
export async function createTodo(formData: FormData) {
  // ... データベースに挿入
  
  revalidatePath('/todos')  // ← これが必須
  return { success: true }
}
```

**revalidatePath**: Next.jsに「このページのデータが変わったから、もう一度取得し直してね」と伝える仕組みです。

---

## 罠4: 権限チェックを忘れると、セキュリティホールになる

### 他人のTodoを編集できてしまう問題

Server Actionsで更新処理を書くだけでは不十分です。「誰のTodoか」を確認しないと、他人のTodoを勝手に編集できてしまいます。

### 実装: 権限チェック
```typescript
export async function updateTodo(id: string, updates: Partial<TodoUpdate>) {
  const supabase = await createClient()
  const { data: { session } } = await supabase.auth.getSession()
  
  // 自分のTodoか確認
  const { data: existingTodo } = await supabase
    .from('todos')
    .select('user_id')
    .eq('id', id)
    .single()
  
  if (existingTodo.user_id !== session.user.id) {
    return { error: '権限がありません' }
  }
  
  // 更新処理
}
```

**重要**: この確認を省略すると、重大なセキュリティ問題になります。

---

## 罠5: UIコンポーネントの役割を理解しないと、コードが複雑になる

### コンポーネントとは

Reactでは、画面をパーツに分けます。各パーツを「コンポーネント」と呼びます。

例えば、Todo項目を表示する部分を`TodoItem`コンポーネントとして分離します。

### 実装: TodoItemコンポーネント

`components/todos/TodoItem.tsx`を作成します。
```typescript
export function TodoItem({ todo }: TodoItemProps) {
  async function handleToggleComplete() {
    await toggleTodoComplete(todo.id, !todo.completed)
  }
  
  return (
    <div className={todo.completed ? 'opacity-75' : ''}>
      <Checkbox checked={todo.completed} onChange={handleToggleComplete} />
      <h3>{todo.title}</h3>
    </div>
  )
}
```

**コンポーネント化のメリット**: コードが読みやすくなり、再利用できます。

---

## 実装の全体像

ここまでの内容を踏まえ、以下の順番で実装します。

1. 型定義（database.types.ts）
2. Server Actions（app/actions/todos.ts）
3. UIコンポーネント（Checkbox、Modal）
4. Todoコンポーネント（TodoItem、TodoForm、TodoList）
5. ページ（app/(dashboard)/todos/page.tsx）

各ファイルの詳細なコードは、元のガイドを参照してください。

---

## 動作確認: 5つのチェックポイント

実装後、以下を確認します。

### 1. Todoが作成できるか

1. タイトルを入力
2. 「作成」をクリック
3. 一覧に表示される

### 2. Todoが編集できるか

1. 「編集」ボタンをクリック
2. 内容を変更
3. 変更が反映される

### 3. Todoが削除できるか

1. 「削除」ボタンをクリック
2. 確認ダイアログが表示
3. Todoが消える

### 4. フィルターが動作するか

1. 「すべて」「未完了」「完了」を切り替え
2. 表示内容が変わる

### 5. データが分離されているか

1. 別アカウントでログイン
2. 他人のTodoが表示されない

---

## よくあるエラーと解決法

### エラー: "認証が必要です"

**原因**: セッションが切れている

**解決法**: 再度ログインする

### エラー: Todoが表示されない

**原因**: フィルターが「完了」になっている

**解決法**: 「すべて」を選択する

### エラー: 作成しても画面に反映されない

**原因**: revalidatePathを忘れている

**解決法**: createTodo関数に`revalidatePath('/todos')`を追加する

---

## まとめ: 5つの罠を回避すれば、Todo機能は完成する

この記事では、初心者がつまずきやすい5つの罠と、その回避法を解説しました。

1. 型定義を最初に設定する
2. Server Actionsでデータベースにアクセスする
3. revalidatePathで画面を更新する
4. 権限チェックでセキュリティを守る
5. コンポーネントで画面を分割する

これらを理解すれば、Todo機能の実装は難しくありません。

次のPhase 4では、UI/UXの仕上げを行います。

---

## 参考記事

- [Next.js Server Actionsの公式ドキュメント](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations)
- [Supabaseの認証ガイド](https://supabase.com/docs/guides/auth)
- [TypeScript型定義の基礎](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)
- [Reactコンポーネント設計のベストプラクティス](https://react.dev/learn/thinking-in-react)