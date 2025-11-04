# 【エラー解決】Next.js 16で「nextCookies.get is not a function」が出る理由と対処法

<!--
date = "2025-11-03"
-->

Next.js 16でSupabaseを使用すると、`nextCookies.get is not a function`というエラーが発生します。この記事では、エラーの原因と解決方法を段階的に解説します。

## エラーメッセージを読み解く

### エラーメッセージ全体
```
TypeError: nextCookies.get is not a function

node_modules/@supabase/auth-helpers-nextjs/src/serverComponentClient.ts (27:22)

25 | protected getCookie(name: string): string | null | undefined {
26 | const nextCookies = this.context.cookies();
> 27 | return nextCookies.get(name)?.value;
   |                           ^
28 | }
```

### エラーを読み解く3つのステップ

#### ステップ1: エラーの種類を確認
```
TypeError: nextCookies.get is not a function
```

**何を言っているか:**
- `TypeError`: 型エラー（存在しない関数を呼び出そうとした）
- `nextCookies.get is not a function`: `nextCookies`に`get`という関数が存在しない

#### ステップ2: どこでエラーが起きたか確認
```
node_modules/@supabase/auth-helpers-nextjs/src/serverComponentClient.ts (27:22)
```

**何を言っているか:**
- ファイルパス: `node_modules/@supabase/auth-helpers-nextjs/...`
- 27行目、22文字目でエラー発生
- これは自分のコードではなく、ライブラリ内部のコード

#### ステップ3: 問題のコードを確認
```typescript
26 | const nextCookies = this.context.cookies();
27 | return nextCookies.get(name)?.value;
```

**何をしようとしているか:**
- 26行目: `cookies()`関数を呼び出して結果を`nextCookies`に格納
- 27行目: `nextCookies.get(name)`でCookieを取得しようとしている
- しかし、`get`関数が存在しないためエラー

## なぜこのエラーが起きるのか

### 原因: Next.js 16でCookieのAPIが変更された

Next.jsのバージョンによって、`cookies()`関数の戻り値が異なります。

#### Next.js 14以前
```typescript
const nextCookies = cookies()

// nextCookies.get() が使える
const value = nextCookies.get('session')?.value
```

#### Next.js 15以降
```typescript
const cookieStore = await cookies()

// API が変わった
const value = cookieStore.getAll()
```

**重要な変更点:**

1. `cookies()`が非同期関数になった（`await`が必要）
2. `get()`メソッドではなく`getAll()`メソッドを使う
3. 戻り値の型が変わった

### 互換性の問題
```
Next.js 16
    ↓ cookies() の API が変更
@supabase/auth-helpers-nextjs
    ↓ 古い API を使用しているため動作しない
エラー発生
```

`@supabase/auth-helpers-nextjs`は古いNext.jsのAPIを前提に作られているため、Next.js 16では動作しません。

## 解決方法: 新しいライブラリに切り替える

### 解決策の概要
```
古い方法（動かない）
@supabase/auth-helpers-nextjs を使用
    ↓
新しい方法（動く）
@supabase/ssr を使用
```

Supabaseは、Next.js 15以降に対応した新しいライブラリ`@supabase/ssr`を提供しています。

### なぜ`@supabase/ssr`が必要なのか

#### 理由1: Next.js 16のAPIに対応

`@supabase/ssr`は、Next.js 16の新しい`cookies()` APIに対応しています。
```typescript
// @supabase/ssr は新しい API に対応
const cookieStore = await cookies()  // await が必要
cookieStore.getAll()  // getAll() を使う
```

#### 理由2: Supabase公式の推奨

Supabase公式ドキュメントでも、Next.js 15以降では`@supabase/ssr`を使用することが推奨されています。

## 修正手順

### ステップ1: パッケージの削除

古いライブラリを削除します。
```bash
npm uninstall @supabase/auth-helpers-nextjs
```

### ステップ2: 新しいパッケージのインストール

新しいライブラリをインストールします（既にインストール済みの場合は不要）。
```bash
npm install @supabase/ssr
```

### ステップ3: ファイルの修正

3つのファイルを修正します。

## 修正内容の詳細解説

### 1. `lib/supabase/server.ts` の修正

#### 修正前（動かないコード）
```typescript
import { createServerComponentClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import { Database } from '@/lib/types/database.types'

export const createClient = () => {
  return createServerComponentClient<Database>({ cookies })
}
```

**問題点:**
- `createServerComponentClient`がNext.js 16に対応していない
- `cookies()`に`await`がない

#### 修正後（動くコード）
```typescript
import { createBrowserClient } from '@supabase/ssr'
import { cookies } from 'next/headers'
import type { Database } from '@/lib/types/database.types'

export async function createClient() {
  const cookieStore = await cookies()
  
  return createBrowserClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll()
        },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            )
          } catch {
            // Server Component からは Cookie を設定できない
            // Middleware で更新されるため無視
          }
        },
      },
    }
  )
}
```

**何が変わったか:**

1. **import の変更**
```typescript
// 変更前
import { createServerComponentClient } from '@supabase/auth-helpers-nextjs'

// 変更後
import { createBrowserClient } from '@supabase/ssr'
```

2. **async/await の追加**
```typescript
// 変更前
export const createClient = () => {

// 変更後
export async function createClient() {
  const cookieStore = await cookies()  // await を追加
```

3. **Cookie操作の実装**
```typescript
cookies: {
  getAll() {
    return cookieStore.getAll()  // 新しいAPI
  },
  setAll(cookiesToSet) {
    // Cookie の設定処理
  },
}
```

**なぜこのように変更するのか:**

- **getAll()**: Next.js 16の新しいAPIを使用
- **setAll()**: Cookieを更新する処理（Server Componentでは実行されない）
- **try-catch**: Server Componentから実行された場合はエラーを無視

### 2. `lib/supabase/client.ts` の修正

#### 修正前（動かないコード）
```typescript
import { createClientComponentClient } from '@supabase/auth-helpers-nextjs'
import { Database } from '@/lib/types/database.types'

export const createClient = () => {
  return createClientComponentClient<Database>()
}
```

#### 修正後（動くコード）
```typescript
import { createBrowserClient } from '@supabase/ssr'
import type { Database } from '@/lib/types/database.types'

export function createClient() {
  return createBrowserClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

**何が変わったか:**

1. **import の変更**
```typescript
// 変更前
import { createClientComponentClient } from '@supabase/auth-helpers-nextjs'

// 変更後
import { createBrowserClient } from '@supabase/ssr'
```

2. **環境変数の明示的な指定**
```typescript
// 変更前
return createClientComponentClient<Database>()

// 変更後
return createBrowserClient<Database>(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
)
```

**なぜこのように変更するのか:**

- クライアント側（ブラウザ）では、環境変数を明示的に渡す必要がある
- `createBrowserClient`はブラウザ環境用の関数

### 3. `lib/supabase/middleware.ts` の修正

#### 修正前（動かないコード）
```typescript
import { createMiddlewareClient } from '@supabase/auth-helpers-nextjs'
import { NextRequest, NextResponse } from 'next/server'
import { Database } from '@/lib/types/database.types'

export const createClient = (req: NextRequest, res: NextResponse) => {
  return createMiddlewareClient<Database>({ req, res })
}
```

#### 修正後（動くコード）
```typescript
import { createServerClient } from '@supabase/ssr'
import { NextRequest, NextResponse } from 'next/server'
import type { Database } from '@/lib/types/database.types'

export function createClient(
  request: NextRequest,
  response: NextResponse
) {
  return createServerClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return request.cookies.getAll()
        },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value }) => 
            request.cookies.set(name, value)
          )
          response = NextResponse.next({
            request: {
              headers: request.headers,
            },
          })
          cookiesToSet.forEach(({ name, value, options }) =>
            response.cookies.set(name, value, options)
          )
        },
      },
    }
  )
}
```

**何が変わったか:**

1. **import の変更**
```typescript
// 変更前
import { createMiddlewareClient } from '@supabase/auth-helpers-nextjs'

// 変更後
import { createServerClient } from '@supabase/ssr'
```

2. **Cookie操作の実装**
```typescript
cookies: {
  getAll() {
    return request.cookies.getAll()  // リクエストからCookieを取得
  },
  setAll(cookiesToSet) {
    // リクエストとレスポンスの両方にCookieを設定
  },
}
```

**なぜこのように変更するのか:**

- Middlewareでは、リクエストとレスポンスの両方のCookieを操作する必要がある
- セッションの更新をMiddlewareで行うため

## 修正後の動作確認

### ステップ1: 開発サーバーを再起動
```bash
# Ctrl+C でサーバーを停止
npm run dev
```

### ステップ2: ブラウザで確認

http://localhost:3000 にアクセスして、エラーが消えていることを確認します。

### ステップ3: 正常に動作する場合
```
✓ Compiled successfully
✓ Supabase接続テスト: ✓ 接続成功
```

## トラブルシューティング

### エラー1: `Cannot find module '@supabase/ssr'`

**原因:**
`@supabase/ssr`がインストールされていない

**解決方法:**
```bash
npm install @supabase/ssr
```

### エラー2: `Module not found: Can't resolve '@supabase/auth-helpers-nextjs'`

**原因:**
古いimport文が残っている

**解決方法:**
プロジェクト内で`@supabase/auth-helpers-nextjs`を検索し、全て`@supabase/ssr`に置き換える
```bash
# ファイル内を検索
grep -r "@supabase/auth-helpers-nextjs" .
```

### エラー3: 型エラーが発生する

**原因:**
TypeScriptのキャッシュが残っている

**解決方法:**
```bash
# .next フォルダを削除
rm -rf .next

# サーバーを再起動
npm run dev
```

## まとめ

### エラーの原因

1. Next.js 16で`cookies()` APIが変更された
2. `@supabase/auth-helpers-nextjs`が古いAPIを使用している
3. 互換性がないためエラーが発生

### 解決方法

1. `@supabase/auth-helpers-nextjs`を削除
2. `@supabase/ssr`を使用
3. 3つのファイルを修正

### 変更の要点

| ファイル | 変更点 |
|---------|--------|
| **server.ts** | `createBrowserClient` + `await cookies()` |
| **client.ts** | `createBrowserClient` + 環境変数明示 |
| **middleware.ts** | `createServerClient` + Cookie操作実装 |

### 今後の注意点

- Next.jsのバージョンアップ時は互換性を確認
- Supabaseの公式ドキュメントを参照
- `@supabase/ssr`が現在の推奨ライブラリ

## 参考リソース

- [Supabase公式ドキュメント - Next.js SSR](https://supabase.com/docs/guides/auth/server-side/nextjs)
- [Next.js公式ドキュメント - Cookies](https://nextjs.org/docs/app/api-reference/functions/cookies)
- [Supabase GitHub - @supabase/ssr](https://github.com/supabase/auth-helpers)

以上が、Next.js 16でSupabaseを使用する際のエラー解決方法の完全解説です。