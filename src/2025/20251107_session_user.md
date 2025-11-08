# Supabaseで「認証が危険です」の警告が出たら：getSession()を今すぐ修正すべき理由

<!--
date = "2025-11-07"
-->

## あなたのコードは大丈夫ですか？

Supabaseを使っていると、こんな警告が表示されることがあります。
```
Using the user object as returned from supabase.auth.getSession() 
or from some supabase.auth.onAuthStateChange() events could be insecure!
```

この警告を無視すると、**他人のデータを自由に閲覧・削除できてしまう**可能性があります。

この記事では、なぜこの警告が表示されるのか、どう修正すればよいのかを説明します。

## 認証の基礎：セッションとは何か

まず、認証の基本を押さえておきます。

Webアプリケーションでは、「このユーザーは誰か」を識別する必要があります。たとえば、TodoアプリでAさんがログインしたとき、「このTodoはAさんのもの」と判断するためです。

Supabaseは、この情報を**セッション**という形で保存しています。セッションには、ユーザーIDやメールアドレスなどが含まれます。

### セッション情報はどこにあるのか

セッション情報は、ブラウザの**クッキー**や**ローカルストレージ**に保存されています。

ここで問題があります。クッキーやローカルストレージは、ユーザーが自由に編集できる場所です。つまり、悪意のあるユーザーが書き換えることができます。

## 警告の正体：getSession()の危険性

### getSession()は何をしているのか

`supabase.auth.getSession()`は、ブラウザに保存されているセッション情報を、そのまま取得します。
```typescript
const { data: { session } } = await supabase.auth.getSession()
const userId = session.user.id
```

この`userId`は、ブラウザから取得された情報です。ユーザーが書き換えていたら、別人のIDが入っている可能性があります。

### 実際に何が起こるのか

想像してください。

1. あなたのTodoアプリで、AさんのIDは`user-123`、BさんのIDは`user-456`です
2. Bさんが、ブラウザの開発者ツールでクッキーを開き、自分のIDを`user-123`に書き換えます
3. あなたのアプリは`getSession()`で取得したIDを信じて、「この人はAさんだ」と判断します
4. Bさんは、AさんのすべてのTodoを見たり、削除したりできてしまいます

これが、この警告が示している危険性です。

## 解決策：getUser()を使う

### getUser()の仕組み

`supabase.auth.getUser()`は、ブラウザの情報をそのまま信じるのではなく、**Supabaseのサーバーに問い合わせ**をします。
```typescript
const { data: { user }, error: authError } = await supabase.auth.getUser()
const userId = user.id
```

サーバーは、セッションが本当に有効か、ユーザーIDが正しいかを確認してから、情報を返します。ユーザーがクッキーを書き換えても、サーバーが「この情報は正しくない」と判断します。

### どちらを使うべきか

判断基準はシンプルです。

| 操作の種類 | 使うべきメソッド | 理由 |
|----------|----------------|------|
| データベースへの書き込み | `getUser()` | 他人のデータを書き換えられてしまう |
| データベースからの読み込み | `getUser()` | 他人のデータを閲覧できてしまう |
| ログイン状態の確認のみ | `getSession()` | セッションの有無を見るだけなら安全 |
| 画面の表示切り替え | `getSession()` | 表示を変えるだけなら問題ない |

**重要な操作には`getUser()`、単なる確認には`getSession()`**です。

## 修正の手順

### 修正が必要なファイル

Todoアプリの例で説明します。`user.id`を使ってデータベースを操作している関数を特定します。

- `getTodos()` - Todoの一覧取得
- `createTodo()` - Todo作成
- `updateTodo()` - Todo更新
- `deleteTodo()` - Todo削除

これらはすべて`getUser()`に変更する必要があります。

### 修正前
```typescript
export async function getTodos() {
  const supabase = await createClient()
  
  const { data: { session } } = await supabase.auth.getSession()
  if (!session) {
    return { error: '認証が必要です' }
  }
  
  const { data, error } = await supabase
    .from('todos')
    .select('*')
    .eq('user_id', session.user.id)  // 危険：改ざんの可能性
    
  return { data, error }
}
```

### 修正後
```typescript
export async function getTodos() {
  const supabase = await createClient()
  
  const { data: { user }, error: authError } = await supabase.auth.getUser()
  if (authError || !user) {
    return { error: '認証が必要です' }
  }
  
  const { data, error } = await supabase
    .from('todos')
    .select('*')
    .eq('user_id', user.id)  // 安全：サーバーで確認済み
    
  return { data, error }
}
```

変更点は3つです。

1. `getSession()`を`getUser()`に変更
2. `session`を`user`に変更
3. エラーハンドリングで`authError`を追加

### エラーハンドリングの注意点

`getUser()`は、サーバーとの通信が発生します。そのため、通信エラーの可能性があります。
```typescript
const { data: { user }, error: authError } = await supabase.auth.getUser()
if (authError || !user) {
  return { error: '認証が必要です' }
}
```

`authError`と`user`の両方を確認してください。

## 修正しなくてよい場所

### layout.tsxやmiddleware.ts

これらのファイルで`getSession()`を使っている場合、ログイン状態を確認しているだけなら、修正は不要です。
```typescript
const { data: { session } } = await supabase.auth.getSession()
if (!session) {
  redirect('/login')
}
```

セッションがあるかないかを見ているだけで、`user.id`を使って重要な操作をしているわけではないためです。

### login/page.tsxやsignup/page.tsx

ログインページでも同様です。ログイン済みかどうかを確認して、リダイレクトする処理では、`getSession()`のままで問題ありません。

## 修正後の確認

修正が完了したら、以下を確認してください。

1. ✅ 警告が表示されなくなった
2. ✅ ログイン、サインアップが正常に動作する
3. ✅ Todo の作成、更新、削除が正常に動作する
4. ✅ ブラウザでクッキーを書き換えても、他人のデータにアクセスできない

特に4番目が重要です。開発者ツールでクッキーを編集し、意図的に不正なアクセスを試みてください。正しく修正されていれば、エラーが返されるはずです。

## よくある質問

### Q: 全部getUser()に変更してもいいですか？

A: 可能ですが、パフォーマンスに影響があります。`getUser()`はサーバーとの通信が発生するため、`getSession()`よりも遅くなります。単なる確認であれば、`getSession()`を使う方が効率的です。

### Q: getSession()は使ってはいけないのですか？

A: そうではありません。ログイン状態の確認や画面の表示切り替えなど、セキュリティ上重要でない場面では、`getSession()`で問題ありません。

### Q: この修正をしないとどうなりますか？

A: 最悪の場合、ユーザーが他人のデータを閲覧、編集、削除できてしまいます。個人情報の漏洩やデータ破壊につながる可能性があります。

## まとめ

- `getSession()`はブラウザの情報を直接取得するため、改ざんのリスクがある
- `getUser()`はサーバーで確認するため、安全性が高い
- データベース操作など重要な処理では`getUser()`を使う
- 単なるログイン確認では`getSession()`でも問題ない
- 修正後は必ず動作確認を行う

この修正により、Supabaseの警告が解消され、安全なアプリケーションになります。

---

## 参考記事

この記事の作成にあたり、以下の公式ドキュメントと記事を参考にしました。

- [Supabase公式ドキュメント：Auth Helpers](https://supabase.com/docs/guides/auth/auth-helpers)
- [Supabase公式ドキュメント：Server-Side Auth](https://supabase.com/docs/guides/auth/server-side)
- [Stack Overflow：Supabase getSession vs getUser](https://stackoverflow.com/questions/tagged/supabase)

より詳しい情報や最新のベストプラクティスについては、Supabase公式ドキュメントをご確認ください。