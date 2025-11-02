# 【ガイド】Next.jsとSupabaseでTodoアプリを作る - 環境構築編
<!--
date = "2025-11-1"
-->
このガイドでは、Next.jsとSupabaseを組み合わせたTodoアプリケーションの開発環境を構築します。手順通りに進めることで、約30分でデータベース接続まで完了した状態になります。

## 何を作るのか

この記事では以下の技術スタックを使用します。

- **Next.js**: Reactベースのフルスタックフレームワーク
- **Supabase**: PostgreSQLデータベースと認証機能を提供するBaaS
- **TypeScript**: 型安全な開発のための言語
- **Tailwind CSS**: ユーティリティファーストのCSSフレームワーク

完成時には、ユーザー認証とデータベース連携が動作する状態になります。

## 前提条件の確認

開始前に以下のソフトウェアがインストールされているか確認します。

- **Node.js**: v18.17以上
```bash
  node -v
```
- **npm**: v9以上
```bash
  npm -v
```
- **Git**: バージョン管理用
```bash
  git -v
```

未インストールの場合は公式サイトからダウンロードします。
- Node.js: https://nodejs.org/
- Git: https://git-scm.com/

## ステップ1: Next.jsプロジェクトの作成

ターミナルを開き、以下のコマンドを実行します。
```bash
npx create-next-app@latest todo-app --typescript --tailwind --app
```

プロンプトが表示されたら、以下のように選択します。
```
✔ Would you like to use ESLint? … Yes
✔ Would you like to use Turbopack for next dev? … No
✔ Would you like to use `src/` directory? … No
✔ Would you like to use App Router? (recommended) … Yes
✔ Would you like to customize the import alias (@/* by default)? … No
```

プロジェクトディレクトリに移動します。
```bash
cd todo-app
```

Supabaseとの連携に必要なパッケージをインストールします。
```bash
npm install @supabase/supabase-js @supabase/auth-helpers-nextjs @supabase/ssr
```

インストールしたパッケージの役割は以下の通りです。
- `@supabase/supabase-js`: Supabaseのコアライブラリ
- `@supabase/auth-helpers-nextjs`: Next.js用の認証ヘルパー
- `@supabase/ssr`: SSR対応のSupabaseクライアント

開発サーバーを起動してテストします。
```bash
npm run dev
```

ブラウザで http://localhost:3000 にアクセスし、Next.jsのデフォルトページが表示されることを確認します。確認できたら、`Ctrl+C` でサーバーを停止します。

## ステップ2: Supabaseプロジェクトの作成

Supabaseのアカウントを作成します。

1. https://supabase.com にアクセス
2. 「Start your project」をクリック
3. GitHubアカウントでサインアップ

新規プロジェクトを作成します。データベースのパスワードは必ず保存しておきます。

1. ダッシュボードで「New project」をクリック
2. 以下の情報を入力します。
   - **Name**: `todo-app`
   - **Database Password**: 強力なパスワードを生成して保存
   - **Region**: `Northeast Asia (Tokyo)`
   - **Pricing Plan**: Free
3. 「Create new project」をクリック

プロジェクトの作成には1-2分かかります。作成が完了したら、接続情報を取得します。

1. 左サイドバーの「Settings」→「API」をクリック
2. 以下の情報をコピーしてメモ帳に保存します。
   - **Project URL**: `https://xxxxxxxxxxxxx.supabase.co`
   - **anon public key**: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

## ステップ3: 環境変数の設定

プロジェクトのルートディレクトリで `.env.local` ファイルを作成します。
```bash
touch .env.local
```

`.env.local` をエディタで開き、以下を記入します。`xxxxxxxxxxxxx` の部分をステップ2で取得した実際の値に置き換えます。
```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://xxxxxxxxxxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# アプリケーションURL
NEXT_PUBLIC_URL=http://localhost:3000
```

`NEXT_PUBLIC_` プレフィックスは必須です。これはクライアント側で使用するための設定です。

`.gitignore` ファイルを開き、`.env.local` が含まれているか確認します。
```
# local env files
.env*.local
```

## ステップ4: データベーススキーマの作成

Supabase ダッシュボードで左サイドバーの「SQL Editor」をクリックし、「New query」をクリックします。以下のSQLを貼り付けて「Run」をクリックします。このSQLは、プロファイルとTodoを管理するテーブルを作成し、セキュリティポリシーを設定します。
```sql
-- profiles テーブル作成
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  display_name TEXT,
  avatar_url TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- todos テーブル作成
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

-- インデックス作成
CREATE INDEX idx_todos_user_id ON todos(user_id);
CREATE INDEX idx_todos_created_at ON todos(created_at DESC);
CREATE INDEX idx_todos_completed ON todos(completed);

-- 更新日時自動更新関数
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- トリガー設定
CREATE TRIGGER update_todos_updated_at
  BEFORE UPDATE ON todos
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_profiles_updated_at
  BEFORE UPDATE ON profiles
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();

-- Row Level Security (RLS) 有効化
ALTER TABLE todos ENABLE ROW LEVEL SECURITY;
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

-- RLSポリシー: ユーザーは自分のTodoのみアクセス可能
CREATE POLICY "Users can only access their own todos"
  ON todos
  FOR ALL
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can only access their own profile"
  ON profiles
  FOR ALL
  USING (auth.uid() = id)
  WITH CHECK (auth.uid() = id);

-- プロファイル自動作成トリガー
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

実行が成功すると「Success. No rows returned」と表示されます。左サイドバーの「Table Editor」をクリックし、`profiles` と `todos` テーブルが作成されていることを確認します。

## ステップ5: ディレクトリ構成の作成

プロジェクトルートで以下のコマンドを実行します。これにより、アプリケーションに必要なディレクトリ構造が作成されます。
```bash
# app配下のディレクトリ
mkdir -p app/\(auth\)/login
mkdir -p app/\(auth\)/signup
mkdir -p app/\(auth\)/reset-password
mkdir -p app/auth/callback
mkdir -p app/\(dashboard\)/todos
mkdir -p app/actions

# components配下のディレクトリ
mkdir -p components/auth
mkdir -p components/todos
mkdir -p components/ui
mkdir -p components/layout

# lib配下のディレクトリ
mkdir -p lib/supabase
mkdir -p lib/types
mkdir -p lib/utils
```

作成後の構成は以下のようになります。
```
todo-app/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   ├── signup/
│   │   └── reset-password/
│   ├── auth/
│   │   └── callback/
│   ├── (dashboard)/
│   │   └── todos/
│   └── actions/
├── components/
│   ├── auth/
│   ├── todos/
│   ├── ui/
│   └── layout/
├── lib/
│   ├── supabase/
│   ├── types/
│   └── utils/
├── public/
├── .env.local
└── package.json
```

## ステップ6: Supabaseクライアントの設定

Next.jsでは、クライアント側とサーバー側で異なるSupabaseクライアントを使用します。`lib/supabase/client.ts` を作成します。
```typescript
import { createClientComponentClient } from '@supabase/auth-helpers-nextjs'
import { Database } from '@/lib/types/database.types'

export const createClient = () => {
  return createClientComponentClient<Database>()
}
```

`lib/supabase/server.ts` を作成します。
```typescript
import { createServerComponentClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import { Database } from '@/lib/types/database.types'

export const createClient = () => {
  return createServerComponentClient<Database>({ cookies })
}
```

`lib/supabase/middleware.ts` を作成します。
```typescript
import { createMiddlewareClient } from '@supabase/auth-helpers-nextjs'
import { NextRequest, NextResponse } from 'next/server'
import { Database } from '@/lib/types/database.types'

export const createClient = (req: NextRequest, res: NextResponse) => {
  return createMiddlewareClient<Database>({ req, res })
}
```

`lib/types/database.types.ts` を作成します。このファイルは、データベースのテーブル構造をTypeScriptの型として定義します。
```typescript
export type Json =
  | string
  | number
  | boolean
  | null
  | { [key: string]: Json | undefined }
  | Json[]

export interface Database {
  public: {
    Tables: {
      profiles: {
        Row: {
          id: string
          display_name: string | null
          avatar_url: string | null
          created_at: string
          updated_at: string
        }
        Insert: {
          id: string
          display_name?: string | null
          avatar_url?: string | null
          created_at?: string
          updated_at?: string
        }
        Update: {
          id?: string
          display_name?: string | null
          avatar_url?: string | null
          created_at?: string
          updated_at?: string
        }
      }
      todos: {
        Row: {
          id: string
          user_id: string
          title: string
          description: string | null
          completed: boolean
          due_date: string | null
          priority: 'low' | 'medium' | 'high' | null
          created_at: string
          updated_at: string
        }
        Insert: {
          id?: string
          user_id: string
          title: string
          description?: string | null
          completed?: boolean
          due_date?: string | null
          priority?: 'low' | 'medium' | 'high' | null
          created_at?: string
          updated_at?: string
        }
        Update: {
          id?: string
          user_id?: string
          title?: string
          description?: string | null
          completed?: boolean
          due_date?: string | null
          priority?: 'low' | 'medium' | 'high' | null
          created_at?: string
          updated_at?: string
        }
      }
    }
  }
}

// 便利な型エイリアス
export type Profile = Database['public']['Tables']['profiles']['Row']
export type Todo = Database['public']['Tables']['todos']['Row']
export type TodoInsert = Database['public']['Tables']['todos']['Insert']
export type TodoUpdate = Database['public']['Tables']['todos']['Update']
```

## ステップ7: 接続テスト

`app/page.tsx` を以下の内容に置き換えます。このページは、Supabaseへの接続が正常に機能しているかをテストします。
```typescript
import { createClient } from '@/lib/supabase/server'

export default async function Home() {
  const supabase = createClient()
  
  // Supabase接続テスト
  const { data, error } = await supabase.from('todos').select('count')
  
  return (
    <main className="flex min-h-screen flex-col items-center justify-center p-24">
      <div className="text-center">
        <h1 className="text-4xl font-bold mb-4">Todo List App</h1>
        <p className="text-lg mb-8">環境構築完了！</p>
        
        <div className="bg-gray-100 p-4 rounded-lg">
          <p className="font-semibold">Supabase接続テスト:</p>
          {error ? (
            <p className="text-red-500">エラー: {error.message}</p>
          ) : (
            <p className="text-green-500">✓ 接続成功</p>
          )}
        </div>
        
        <div className="mt-8">
          <a 
            href="/login" 
            className="bg-blue-500 text-white px-6 py-2 rounded-lg hover:bg-blue-600"
          >
            ログインページへ（次のPhaseで作成）
          </a>
        </div>
      </div>
    </main>
  )
}
```

開発サーバーを起動します。
```bash
npm run dev
```

ブラウザで http://localhost:3000 にアクセスし、以下を確認します。

- ページが表示される
- 「Supabase接続テスト: ✓ 接続成功」と表示される

エラーが出る場合は以下を確認します。
- `.env.local` の環境変数が正しいか
- Supabaseプロジェクトが起動しているか
- サーバーを再起動（`Ctrl+C` → `npm run dev`）

## ステップ8: Gitリポジトリの初期化

以下のコマンドでGitリポジトリを初期化します。
```bash
git init
git add .
git commit -m "Initial commit: Phase 1 完了"
```

GitHubリポジトリを作成する場合は以下の手順を実行します。

1. https://github.com にアクセス
2. 「New repository」をクリック
3. リポジトリ名: `todo-app`
4. 「Create repository」をクリック
5. 表示されるコマンドを実行します。
```bash
git remote add origin https://github.com/your-username/todo-app.git
git branch -M main
git push -u origin main
```

## 完了チェックリスト

以下の項目を確認します。

- Next.jsプロジェクトが作成された
- Supabaseプロジェクトが作成された
- `.env.local` に環境変数が設定された
- データベーススキーマが作成された
- ディレクトリ構成が作成された
- Supabaseクライアントが設定された
- 型定義ファイルが作成された
- http://localhost:3000 でページが表示される
- Supabase接続テストが成功する

## トラブルシューティング

### Module not found

パッケージがインストールされていない場合に発生します。
```bash
npm install
```

### Invalid environment variables

`.env.local` の設定が間違っている場合に発生します。以下を確認します。
- `.env.local` のURLとキーを確認
- `NEXT_PUBLIC_` プレフィックスがあるか確認
- サーバーを再起動

### relation "todos" does not exist

データベーススキーマが作成されていない場合に発生します。ステップ4のSQLを再度実行し、Supabase Dashboard の Table Editor でテーブルを確認します。

### ポート3000が使用中

別のポートで起動します。
```bash
PORT=3001 npm run dev
```

## 参考リソース

- [Next.js公式ドキュメント](https://nextjs.org/docs)
- [Supabase公式ドキュメント](https://supabase.com/docs)
- [Next.js + Supabase チュートリアル](https://supabase.com/docs/guides/getting-started/tutorials/with-nextjs)
- [Next.jsでのサーバーサイド認証設定](https://supabase.com/docs/guides/auth/server-side/nextjs)
- [Supabase認証クイックスタート](https://supabase.com/docs/guides/auth/quickstarts/nextjs)

以上で環境構築は完了です。次のステップでは認証機能を実装します。