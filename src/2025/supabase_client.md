# Next.jsでSupabaseクライアントを3つ作る理由 - クライアント/サーバー/Middlewareの違い

<!--
date = "2025-11-03"
-->

Supabaseを使用したNext.jsアプリケーションでは、3種類のクライアントを作成します。この記事では、「なぜ3つも必要なのか」「それぞれの違いは何か」を徹底的に解説します。

## 結論: なぜ3つのクライアントが必要なのか

Next.jsには3つの実行環境があり、それぞれ異なる方法でSupabaseにアクセスする必要があります。
```
1. クライアント側（ブラウザ）
   └→ client.ts を使用

2. サーバー側（Next.jsサーバー）
   └→ server.ts を使用

3. Middleware（リクエスト処理）
   └→ middleware.ts を使用
```

それぞれの環境では、認証情報（セッション）の取得方法が異なるため、専用のクライアントが必要です。

## Next.jsの3つの実行環境

### 環境1: クライアント側（ブラウザ）

ユーザーのブラウザで実行されるコードです。
```typescript
'use client'  // このファイルはブラウザで実行される

import { useState } from 'react'

export default function TodoList() {
  const [todos, setTodos] = useState([])
  // ブラウザで実行
}
```

**特徴:**
- ユーザーのブラウザで実行
- Reactの状態管理やイベント処理
- リアルタイムでUIを更新

### 環境2: サーバー側（Next.jsサーバー）

Next.jsサーバーで実行されるコードです。
```typescript
// 'use client' がない = サーバーで実行される

export default async function TodoPage() {
  const todos = await fetchTodos()  // サーバーで実行
  return <div>{todos}</div>
}
```

**特徴:**
- Next.jsサーバーで実行
- データベースから直接データを取得
- HTMLを生成してブラウザに送信

### 環境3: Middleware（リクエスト処理）

全てのリクエストの前に実行されるコードです。
```typescript
// middleware.ts
export async function middleware(request: NextRequest) {
  // 全てのリクエストで実行される
  // 認証チェック、リダイレクトなど
}
```

**特徴:**
- リクエストごとに実行
- ページが表示される前に実行
- 認証チェックやリダイレクトに使用

## なぜ環境ごとに異なるクライアントが必要なのか

### 理由: 認証情報（Cookie）の取得方法が異なる

Supabaseの認証情報はCookieに保存されます。しかし、Cookieの取得方法は実行環境によって異なります。
```
クライアント側（ブラウザ）
├── ブラウザが自動的にCookieを送信
└── document.cookie で取得可能

サーバー側（Next.jsサーバー）
├── Next.jsの cookies() 関数を使用
└── リクエストヘッダーから取得

Middleware（リクエスト処理）
├── NextRequest オブジェクトから取得
└── NextResponse オブジェクトに設定
```

この違いがあるため、3つの異なるクライアントが必要です。

## 1. クライアントサイド用クライアント

### コード
```typescript
import { createClientComponentClient } from '@supabase/auth-helpers-nextjs'
import { Database } from '@/lib/types/database.types'

export const createClient = () => {
  return createClientComponentClient<Database>()
}
```

### 何をしているのか

ブラウザで実行されるReactコンポーネント用のSupabaseクライアントを作成します。

### なぜ必要なのか

ブラウザでは、ユーザーのインタラクション（ボタンクリック、フォーム送信など）に応じてデータベースにアクセスする必要があります。

### 使用例
```typescript
'use client'

import { createClient } from '@/lib/supabase/client'
import { useState } from 'react'

export default function TodoForm() {
  const [title, setTitle] = useState('')
  const supabase = createClient()  // ブラウザで実行

  const handleSubmit = async () => {
    // ブラウザからSupabaseにアクセス
    const { data, error } = await supabase
      .from('todos')
      .insert({ title })
    
    if (!error) {
      alert('Todoを作成しました')
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input 
        value={title} 
        onChange={(e) => setTitle(e.target.value)} 
      />
      <button type="submit">作成</button>
    </form>
  )
}
```

### メリット

1. **リアルタイム更新**: ブラウザで即座にUIを更新
2. **ユーザーインタラクション**: ボタンクリックなどに即座に反応
3. **Cookieの自動管理**: ブラウザが自動的に認証情報を送信

### 動作の流れ
```
ユーザーがボタンをクリック
    ↓
ブラウザでJavaScriptが実行
    ↓
createClientComponentClient() でクライアント作成
    ↓
ブラウザのCookieから認証情報を取得
    ↓
Supabase APIにリクエスト
    ↓
結果を受け取りUIを更新
```

## 2. サーバーサイド用クライアント

### コード
```typescript
import { createServerComponentClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import { Database } from '@/lib/types/database.types'

export const createClient = () => {
  return createServerComponentClient<Database>({ cookies })
}
```

### 何をしているのか

Next.jsサーバーで実行されるコンポーネント用のSupabaseクライアントを作成します。`cookies`関数を使用してサーバー側でCookieを取得します。

### なぜ必要なのか

サーバー側では、ページを生成する前にデータベースからデータを取得する必要があります。これにより、検索エンジンに最適化されたHTMLを生成できます。

### 使用例
```typescript
// Server Component（'use client' がない）

import { createClient } from '@/lib/supabase/server'

export default async function TodoPage() {
  const supabase = createClient()  // サーバーで実行

  // サーバーでデータを取得
  const { data: todos } = await supabase
    .from('todos')
    .select('*')
  
  // HTMLを生成してブラウザに送信
  return (
    <div>
      <h1>Todo一覧</h1>
      <ul>
        {todos?.map(todo => (
          <li key={todo.id}>{todo.title}</li>
        ))}
      </ul>
    </div>
  )
}
```

### メリット

1. **SEO対策**: 検索エンジンがコンテンツを読み取れる
2. **初期表示の高速化**: サーバーでデータを取得してHTMLを生成
3. **セキュリティ**: サーバー側でのみアクセス可能なデータを取得可能

### クライアント側との違い

| 項目 | クライアント側 | サーバー側 |
|-----|-------------|----------|
| **実行場所** | ブラウザ | Next.jsサーバー |
| **Cookie取得** | 自動 | `cookies()` 関数 |
| **SEO** | ❌ 不可 | ⭕ 可能 |
| **初期表示** | 遅い | 速い |
| **リアルタイム更新** | ⭕ 可能 | ❌ 不可 |

### 動作の流れ
```
ユーザーがページにアクセス
    ↓
Next.jsサーバーでコンポーネント実行
    ↓
createServerComponentClient({ cookies }) でクライアント作成
    ↓
cookies() 関数でCookieを取得
    ↓
Supabaseからデータを取得
    ↓
HTMLを生成
    ↓
ブラウザに送信
```

## 3. Middleware用クライアント

### コード
```typescript
import { createMiddlewareClient } from '@supabase/auth-helpers-nextjs'
import { NextRequest, NextResponse } from 'next/server'
import { Database } from '@/lib/types/database.types'

export const createClient = (req: NextRequest, res: NextResponse) => {
  return createMiddlewareClient<Database>({ req, res })
}
```

### 何をしているのか

Middlewareで実行されるSupabaseクライアントを作成します。`NextRequest`と`NextResponse`オブジェクトを使用してCookieを取得・更新します。

### なぜ必要なのか

Middlewareは、全てのリクエストの前に実行されます。認証チェックやセッションの更新を行うために、専用のクライアントが必要です。

### 使用例
```typescript
// middleware.ts

import { createClient } from '@/lib/supabase/middleware'
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export async function middleware(request: NextRequest) {
  const response = NextResponse.next()
  const supabase = createClient(request, response)

  // セッションを取得（認証チェック）
  const { data: { session } } = await supabase.auth.getSession()

  // 未ログインの場合はログインページにリダイレクト
  if (!session && request.nextUrl.pathname.startsWith('/todos')) {
    return NextResponse.redirect(new URL('/login', request.url))
  }

  return response
}

export const config = {
  matcher: ['/todos/:path*']  // /todos 配下の全てのページで実行
}
```

### メリット

1. **認証チェック**: ページ表示前に認証状態を確認
2. **セッション更新**: 期限切れ前にセッションを自動更新
3. **リダイレクト**: 未ログインユーザーを自動的にリダイレクト
4. **全体的な保護**: アプリケーション全体のセキュリティを強化

### 他の2つとの違い

| 項目 | Middleware | クライアント/サーバー |
|-----|-----------|-------------------|
| **実行タイミング** | リクエストの前 | ページ表示時 |
| **用途** | 認証チェック | データ取得 |
| **Cookie操作** | 取得・更新 | 取得のみ |
| **リダイレクト** | ⭕ 可能 | 制限あり |

### 動作の流れ
```
ユーザーがページにアクセス
    ↓
Middlewareが実行される（ページ表示前）
    ↓
createMiddlewareClient(req, res) でクライアント作成
    ↓
セッションを取得
    ↓
セッションが期限切れ間近の場合は更新
    ↓
認証状態をチェック
    ↓
未ログインの場合はリダイレクト
    ↓
ログイン済みの場合はページを表示
```

## 型定義ファイル（database.types.ts）

### コード
```typescript
export interface Database {
  public: {
    Tables: {
      todos: {
        Row: {
          id: string
          title: string
          completed: boolean
        }
        Insert: {
          id?: string
          title: string
          completed?: boolean
        }
        Update: {
          id?: string
          title?: string
          completed?: boolean
        }
      }
    }
  }
}
```

### 何をしているのか

データベースのテーブル構造をTypeScriptの型として定義します。

### なぜ必要なのか

TypeScriptの型チェックにより、以下のエラーを防ぎます。
```typescript
// 型定義があると...

// ❌ エラー: 存在しないカラムを指定
const { data } = await supabase
  .from('todos')
  .select('title, completedd')  // completedd は typo
// TypeScriptがエラーを表示: 'completedd' というプロパティは存在しません

// ❌ エラー: 必須カラムの欠落
const { data } = await supabase
  .from('todos')
  .insert({ completed: false })  // title が必須だが欠落
// TypeScriptがエラーを表示: 'title' プロパティが必要です

// ⭕ 正しい使用
const { data } = await supabase
  .from('todos')
  .insert({ title: '買い物', completed: false })
```

### 3つの型の違い

#### Row型: データベースから取得する型
```typescript
Row: {
  id: string
  title: string
  completed: boolean
  created_at: string
}
```

**使用場面:**
```typescript
// SELECT で取得したデータの型
const { data } = await supabase.from('todos').select('*')
// data の型は Todo[] (= Row[])
```

#### Insert型: データベースに挿入する型
```typescript
Insert: {
  id?: string          // 自動生成されるためオプション
  title: string        // 必須
  completed?: boolean  // デフォルト値があるためオプション
  created_at?: string  // 自動設定されるためオプション
}
```

**使用場面:**
```typescript
// INSERT で新規作成する型
const { data } = await supabase
  .from('todos')
  .insert({ title: '買い物' })  // id, completed, created_at は省略可能
```

#### Update型: データベースを更新する型
```typescript
Update: {
  id?: string
  title?: string
  completed?: boolean
  created_at?: string
}
```

全てオプションです。更新したいフィールドのみを指定します。

**使用場面:**
```typescript
// UPDATE で更新する型
const { data } = await supabase
  .from('todos')
  .update({ completed: true })  // title は更新しない
  .eq('id', '123')
```

### メリット

1. **型安全性**: コンパイル時にエラーを検出
2. **自動補完**: エディタが候補を表示
3. **ドキュメント**: 型定義がドキュメントの役割
4. **リファクタリング**: 型を変更すると関連する箇所が自動検出

## 3つのクライアントの使い分けまとめ

### 使い分けのフローチャート
```
何をしたいか？
│
├─ ユーザーのボタンクリックに反応したい
│  └→ client.ts（クライアントサイド）
│
├─ ページを表示する前にデータを取得したい
│  └→ server.ts（サーバーサイド）
│
└─ ページ表示前に認証チェックしたい
   └→ middleware.ts（Middleware）
```

### 実際の使用例
```typescript
// 1. Middleware（認証チェック）
// middleware.ts
export async function middleware(request: NextRequest) {
  const supabase = createClient(request, response)
  const { data: { session } } = await supabase.auth.getSession()
  
  if (!session) {
    return NextResponse.redirect(new URL('/login', request.url))
  }
  
  return response
}

// 2. サーバー側（初期データ取得）
// app/todos/page.tsx
export default async function TodosPage() {
  const supabase = createClient()  // server.ts
  const { data: todos } = await supabase.from('todos').select('*')
  
  return <TodoList initialTodos={todos} />
}

// 3. クライアント側（ユーザーインタラクション）
// components/TodoList.tsx
'use client'

export default function TodoList({ initialTodos }) {
  const supabase = createClient()  // client.ts
  
  const handleToggle = async (id: string) => {
    await supabase
      .from('todos')
      .update({ completed: true })
      .eq('id', id)
  }
  
  return <div>...</div>
}
```

## なぜ1つのクライアントではダメなのか

### 問題1: Cookieの取得方法が異なる
```typescript
// ❌ これは動作しない

// ブラウザで実行
const supabase = createServerComponentClient({ cookies })
// エラー: cookies() はサーバーでのみ使用可能

// サーバーで実行
const supabase = createClientComponentClient()
// エラー: ブラウザのCookieにアクセスできない
```

### 問題2: 実行環境の違い
```typescript
// Middleware
export async function middleware(request: NextRequest) {
  // ❌ これは動作しない
  const supabase = createClientComponentClient()
  // Middlewareには NextRequest/NextResponse が必要
}
```

## まとめ

### 3つのクライアントが必要な理由

1. **実行環境の違い**: ブラウザ、サーバー、Middlewareで実行環境が異なる
2. **Cookie取得方法の違い**: 各環境でCookieの取得方法が異なる
3. **用途の違い**: データ取得、認証チェック、リアルタイム更新で用途が異なる

### それぞれの役割

| クライアント | 実行場所 | 用途 | Cookie取得 |
|------------|---------|------|-----------|
| **client.ts** | ブラウザ | ユーザーインタラクション | 自動 |
| **server.ts** | サーバー | 初期データ取得 | `cookies()` |
| **middleware.ts** | Middleware | 認証チェック | `req/res` |

### 型定義の役割

- **Row型**: データベースから取得
- **Insert型**: データベースに挿入
- **Update型**: データベースを更新

### ベストプラクティス

1. **Middleware**: 認証チェックとセッション更新
2. **サーバー側**: 初期データ取得とSEO対策
3. **クライアント側**: ユーザーインタラクションとリアルタイム更新
4. **型定義**: 全てのクライアントで使用して型安全性を確保

これにより、安全で高速なアプリケーションを構築できます。

## 参考リソース

- [Next.js公式ドキュメント - Server Components](https://nextjs.org/docs/app/building-your-application/rendering/server-components)
- [Next.js公式ドキュメント - Client Components](https://nextjs.org/docs/app/building-your-application/rendering/client-components)
- [Next.js公式ドキュメント - Middleware](https://nextjs.org/docs/app/building-your-application/routing/middleware)
- [Supabase公式ドキュメント - Next.js認証](https://supabase.com/docs/guides/auth/server-side/nextjs)

以上が、3つのSupabaseクライアントとそれぞれの役割の完全解説です。