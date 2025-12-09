# 【完全保存版】週末で完成！バイブコーディングでゼロからWebサービスを作る5ステップ

<!--
date = "2025-12-08"
-->

「プログラミング初心者でも、本当にWebサービスが作れるの？」

答えはYESです。**バイブコーディングを使えば、週末の2日間でプロダクションレベルのWebサービスが完成します。**

この記事では、実際に「タスク管理アプリ」を作りながら、ゼロからデプロイまでの全工程を解説します。読み終わる頃には、**あなたのアイデアを形にする具体的な方法**が完全に理解できているはずです。

## 今回作るもの：「TaskMaster」タスク管理Webアプリ

### 完成イメージ

**機能：**
- ユーザー登録・ログイン（メール認証付き）
- タスクの追加・編集・削除
- 優先度設定（高・中・低）
- 期限設定とリマインダー
- 完了/未完了の切り替え
- タスクの検索・フィルタリング

**技術スタック：**
- フロントエンド: Next.js 14 + TypeScript + Tailwind CSS
- バックエンド: Next.js API Routes
- データベース: Supabase (PostgreSQL)
- 認証: Supabase Auth
- デプロイ: Vercel

**開発時間：** 2日間（土日）
**使用ツール：** Cursor（初心者向け）

## 事前準備（30分）

作業を始める前に、以下を準備しましょう。

### 必要なアカウント作成

1. **Cursor アカウント**
   - [https://cursor.sh/](https://cursor.sh/) でダウンロード
   - 無料プランでOK

2. **Supabase アカウント**
   - [https://supabase.com/](https://supabase.com/) で無料登録
   - プロジェクトを1つ作成（名前: taskmaster）

3. **Vercel アカウント**
   - [https://vercel.com/](https://vercel.com/) で無料登録
   - GitHubアカウントと連携

### 作業フォルダの作成

デスクトップに `taskmaster` フォルダを作成し、Cursorで開きます。

**準備完了！ここから実際の開発が始まります。**

---

## 【Day 1 - 土曜日】フェーズ1〜3（8時間）

### フェーズ1：プロジェクトの設計と足場固め（1時間）

#### ステップ1-1：AIと一緒にPRDを作る

まず、プロジェクトの「設計図」を作ります。

**Cursorで新規ファイル作成：** `PRD.md`

**あなたの指示（Cmd+K または Ctrl+K）：**
```
あなたはシニアプロダクトマネージャーです。
タスク管理Webアプリ「TaskMaster」のPRD（プロダクト要件定義書）を作成してください。

【アプリ概要】
個人向けのシンプルなタスク管理ツール。
Todoistのような機能を持つが、よりシンプルで使いやすいもの。

【主要機能】
- ユーザー登録・ログイン
- タスクのCRUD操作
- 優先度設定
- 期限設定
- 完了/未完了の切り替え
- タスク検索

【技術スタック】
- Next.js 14 (App Router)
- TypeScript
- Tailwind CSS
- Supabase
- Vercel

以下の形式でPRDを生成してください：
1. プロジェクト概要
2. 主要機能の詳細
3. データモデル
4. 技術スタック
5. ファイル構造
6. 開発フェーズ
```

**AIが生成するPRD（例）：**
```markdown
# TaskMaster プロダクト要件定義書

## 1. プロジェクト概要
TaskMasterは、個人向けのシンプルで直感的なタスク管理Webアプリケーションです。

## 2. 主要機能

### 2.1 認証機能
- メールアドレス + パスワードでのユーザー登録
- ログイン/ログアウト
- メール認証（Supabase Auth）

### 2.2 タスク管理
- タスクの作成（タイトル、説明、優先度、期限）
- タスクの編集
- タスクの削除
- 完了/未完了の切り替え

### 2.3 表示・フィルタリング
- 全タスク一覧
- 優先度でフィルタリング
- 期限でソート
- キーワード検索

## 3. データモデル

### users テーブル (Supabase Auth管理)
- id (UUID)
- email (string)
- created_at (timestamp)

### tasks テーブル
- id (UUID, primary key)
- user_id (UUID, foreign key -> users.id)
- title (string, required)
- description (text, optional)
- priority ('high' | 'medium' | 'low')
- due_date (date, optional)
- is_completed (boolean, default: false)
- created_at (timestamp)
- updated_at (timestamp)

## 4. 技術スタック
- Framework: Next.js 14 (App Router)
- Language: TypeScript
- Styling: Tailwind CSS
- Database: Supabase (PostgreSQL)
- Auth: Supabase Auth
- Deployment: Vercel

## 5. ファイル構造
```
taskmaster/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   └── register/
│   ├── (dashboard)/
│   │   ├── tasks/
│   │   └── layout.tsx
│   ├── api/
│   │   └── tasks/
│   ├── layout.tsx
│   └── page.tsx
├── components/
│   ├── TaskCard.tsx
│   ├── TaskForm.tsx
│   ├── TaskList.tsx
│   └── Header.tsx
├── lib/
│   ├── supabase.ts
│   └── types.ts
└── public/
```

## 6. 開発フェーズ
Phase 1: プロジェクトセットアップ + 認証
Phase 2: データベース設計 + タスクCRUD
Phase 3: UI実装
Phase 4: フィルタリング・検索
Phase 5: デプロイ
```

**所要時間：AIが3分で生成**

このPRDが、今後の開発の「真実の唯一のソース」になります。

#### ステップ1-2：.cursorrulesでルールを確立

**新規ファイル作成：** `.cursorrules`

**あなたの指示：**
```
@PRD.md を参照して、このプロジェクトのコーディング規約を定義した .cursorrules ファイルを作成してください。

含めるべき内容：
- TypeScript strict モードの使用
- Next.js 14 App Router のベストプラクティス
- Tailwind CSS の使用
- コンポーネントは必ず TypeScript で型定義
- エラーハンドリングは必ず含める
- コメントは日本語で記述
```

**AIが生成する .cursorrules（例）：**
```
# TaskMaster コーディング規約

## 技術スタック
- Next.js 14 (App Router) を使用すること
- TypeScript strict モード必須
- スタイリングは Tailwind CSS のみ使用
- データベースは Supabase

## コーディング規則
- すべてのコンポーネントは TypeScript で記述
- Props の型定義は必須（interface または type で定義）
- any 型の使用は絶対禁止
- 関数コンポーネントのみ使用（クラスコンポーネント禁止）
- エラーハンドリングは必ず try-catch で実装
- 環境変数は .env.local に記載し、.gitignore に含める

## コメント
- コメントは日本語で記述
- 複雑なロジックには必ず説明コメントを付ける

## インポート順序
1. React関連
2. Next.js関連
3. 外部ライブラリ
4. 内部コンポーネント
5. 型定義
6. スタイル

## 命名規則
- コンポーネント: PascalCase (例: TaskCard.tsx)
- 関数: camelCase (例: getTasks)
- 定数: UPPER_SNAKE_CASE (例: API_URL)
```

#### ステップ1-3：プロジェクトの初期化

**Cursorのターミナルを開いて、以下を入力：**
```bash
npx create-next-app@latest . --typescript --tailwind --app --eslint
```

質問には全て「Yes」で答えます。

**完了後、あなたの指示（Composer: Cmd+I）：**
```
@PRD.md に従って、必要な依存関係をインストールしてください。

必要なパッケージ：
- @supabase/supabase-js (Supabase クライアント)
- @supabase/auth-helpers-nextjs (Next.js用認証ヘルパー)
- date-fns (日付処理)
- lucide-react (アイコン)

package.json を更新し、npm install を実行してください。
```

**AIの動き：**
```
package.json を更新しました。
npm install を実行中...
完了しました。
```

**フェーズ1完了！所要時間：1時間**

---

### フェーズ2：データベースとSupabase連携（2時間）

#### ステップ2-1：Supabaseプロジェクトのセットアップ

**Supabaseダッシュボード（https://app.supabase.com）で：**

1. 作成したプロジェクトを開く
2. Settings → API から以下をコピー：
   - `Project URL`
   - `anon public key`

**ローカルに環境変数ファイル作成：** `.env.local`

**あなたの指示：**
```
.env.local ファイルを作成し、以下の環境変数を設定してください：

NEXT_PUBLIC_SUPABASE_URL=（あなたのProject URL）
NEXT_PUBLIC_SUPABASE_ANON_KEY=（あなたのanon key）

このファイルは .gitignore に必ず含めてください。
```

#### ステップ2-2：データベーススキーマの作成

**Supabaseダッシュボードで SQL Editor を開く**

**あなたの指示（Cursorで新規ファイル `schema.sql` を作成）：**
```
@PRD.md のデータモデルに基づいて、PostgreSQL の CREATE TABLE 文を生成してください。

要件：
- tasks テーブルを作成
- user_id は auth.users への外部キー
- created_at, updated_at は自動設定
- Row Level Security (RLS) を有効化
- ユーザーは自分のタスクのみ操作可能なポリシーを設定
```

**AIが生成するSQL：**
```sql
-- tasks テーブルの作成
CREATE TABLE tasks (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  title TEXT NOT NULL,
  description TEXT,
  priority TEXT CHECK (priority IN ('high', 'medium', 'low')) DEFAULT 'medium',
  due_date DATE,
  is_completed BOOLEAN DEFAULT false,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc'::text, now()) NOT NULL,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc'::text, now()) NOT NULL
);

-- RLS（Row Level Security）を有効化
ALTER TABLE tasks ENABLE ROW LEVEL SECURITY;

-- ポリシー：ユーザーは自分のタスクのみ閲覧可能
CREATE POLICY "Users can view own tasks"
  ON tasks FOR SELECT
  USING (auth.uid() = user_id);

-- ポリシー：ユーザーは自分のタスクのみ作成可能
CREATE POLICY "Users can create own tasks"
  ON tasks FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- ポリシー：ユーザーは自分のタスクのみ更新可能
CREATE POLICY "Users can update own tasks"
  ON tasks FOR UPDATE
  USING (auth.uid() = user_id);

-- ポリシー：ユーザーは自分のタスクのみ削除可能
CREATE POLICY "Users can delete own tasks"
  ON tasks FOR DELETE
  USING (auth.uid() = user_id);

-- updated_at の自動更新トリガー
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = timezone('utc'::text, now());
  RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_tasks_updated_at BEFORE UPDATE ON tasks
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

**このSQLをSupabase SQL Editorに貼り付けて実行！**

#### ステップ2-3：Supabaseクライアントの設定

**あなたの指示：**
```
lib/supabase.ts ファイルを作成し、Supabaseクライアントを初期化してください。

要件：
- createClientComponentClient と createServerComponentClient の両方をエクスポート
- 環境変数から URL と KEY を読み込む
- TypeScript の型定義を含める
```

**AIが生成するコード：**
```typescript
// lib/supabase.ts
import { createClientComponentClient, createServerComponentClient } from '@supabase/auth-helpers-nextjs';
import { cookies } from 'next/headers';

// クライアントコンポーネント用
export const createClient = () => createClientComponentClient();

// サーバーコンポーネント用
export const createServerClient = () => createServerComponentClient({ cookies });

// 型定義
export type Database = {
  public: {
    Tables: {
      tasks: {
        Row: {
          id: string;
          user_id: string;
          title: string;
          description: string | null;
          priority: 'high' | 'medium' | 'low';
          due_date: string | null;
          is_completed: boolean;
          created_at: string;
          updated_at: string;
        };
        Insert: {
          id?: string;
          user_id: string;
          title: string;
          description?: string | null;
          priority?: 'high' | 'medium' | 'low';
          due_date?: string | null;
          is_completed?: boolean;
        };
        Update: {
          title?: string;
          description?: string | null;
          priority?: 'high' | 'medium' | 'low';
          due_date?: string | null;
          is_completed?: boolean;
        };
      };
    };
  };
};
```

**フェーズ2完了！所要時間：2時間**

---

### フェーズ3：認証機能の実装（2時間）

#### ステップ3-1：ログインページの作成

**あなたの指示（Composer）：**
```
app/(auth)/login/page.tsx を作成してください。

要件：
- メールアドレスとパスワードの入力フォーム
- Supabase Auth でログイン処理
- エラーメッセージの表示
- Tailwind CSS でモダンなデザイン
- ログイン成功後は /tasks にリダイレクト
- 「アカウント作成」リンクを /register に配置

@lib/supabase.ts を参照してください。
```

**AIが生成するコード（例）：**
```typescript
// app/(auth)/login/page.tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { createClient } from '@/lib/supabase';
import Link from 'next/link';

export default function LoginPage() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  const router = useRouter();
  const supabase = createClient();

  const handleLogin = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');
    setLoading(true);

    try {
      const { data, error } = await supabase.auth.signInWithPassword({
        email,
        password,
      });

      if (error) throw error;

      router.push('/tasks');
      router.refresh();
    } catch (error: any) {
      setError(error.message || 'ログインに失敗しました');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <div className="max-w-md w-full space-y-8 p-8 bg-white rounded-lg shadow-md">
        <div>
          <h2 className="text-center text-3xl font-bold text-gray-900">
            TaskMaster
          </h2>
          <p className="mt-2 text-center text-sm text-gray-600">
            ログインしてタスクを管理しましょう
          </p>
        </div>

        <form className="mt-8 space-y-6" onSubmit={handleLogin}>
          {error && (
            <div className="bg-red-50 border border-red-200 text-red-600 px-4 py-3 rounded">
              {error}
            </div>
          )}

          <div className="space-y-4">
            <div>
              <label htmlFor="email" className="block text-sm font-medium text-gray-700">
                メールアドレス
              </label>
              <input
                id="email"
                type="email"
                required
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                className="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500"
              />
            </div>

            <div>
              <label htmlFor="password" className="block text-sm font-medium text-gray-700">
                パスワード
              </label>
              <input
                id="password"
                type="password"
                required
                value={password}
                onChange={(e) => setPassword(e.target.value)}
                className="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500"
              />
            </div>
          </div>

          <button
            type="submit"
            disabled={loading}
            className="w-full flex justify-center py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 disabled:opacity-50"
          >
            {loading ? 'ログイン中...' : 'ログイン'}
          </button>

          <p className="text-center text-sm text-gray-600">
            アカウントをお持ちでないですか？{' '}
            <Link href="/register" className="font-medium text-blue-600 hover:text-blue-500">
              新規登録
            </Link>
          </p>
        </form>
      </div>
    </div>
  );
}
```

#### ステップ3-2：ユーザー登録ページの作成

**あなたの指示：**
```
@(auth)/login/page.tsx を参考に、
app/(auth)/register/page.tsx を作成してください。

要件：
- メールアドレスとパスワード（確認用も含む）の入力
- Supabase Auth で新規登録
- パスワードは8文字以上のバリデーション
- 登録成功後は /tasks にリダイレクト
- 「すでにアカウントをお持ちの方」リンクを /login に配置
```

**AIが自動生成！**

#### ステップ3-3：認証状態の確認

**あなたの指示：**
```
app/(dashboard)/layout.tsx を作成してください。

要件：
- ユーザーの認証状態をチェック
- 未ログインの場合は /login にリダイレクト
- ログイン中の場合は子コンポーネントを表示
- ヘッダーにログアウトボタンを配置
```

**フェーズ3完了！所要時間：2時間**

**Day 1 終了。ここまでで認証システムが完成しました。**

---

## 【Day 2 - 日曜日】フェーズ4〜5（6時間）

### フェーズ4：タスク機能の実装（4時間）

#### ステップ4-1：タスク一覧の表示

**あなたの指示：**
```
app/(dashboard)/tasks/page.tsx を作成してください。

要件：
- Supabase からログインユーザーのタスク一覧を取得
- TaskList コンポーネントで表示
- ローディング状態の表示
- エラーハンドリング
- 「新規タスク追加」ボタン

@lib/supabase.ts と @PRD.md を参照してください。
```

**AIが生成するコード（例）：**
```typescript
// app/(dashboard)/tasks/page.tsx
import { createServerClient } from '@/lib/supabase';
import TaskList from '@/components/TaskList';
import { PlusIcon } from 'lucide-react';
import Link from 'next/link';

export default async function TasksPage() {
  const supabase = createServerClient();

  // ユーザーの認証状態を確認
  const { data: { user } } = await supabase.auth.getUser();

  if (!user) {
    redirect('/login');
  }

  // タスク一覧を取得
  const { data: tasks, error } = await supabase
    .from('tasks')
    .select('*')
    .order('created_at', { ascending: false });

  if (error) {
    console.error('Error fetching tasks:', error);
  }

  return (
    <div className="max-w-4xl mx-auto px-4 py-8">
      <div className="flex justify-between items-center mb-8">
        <h1 className="text-3xl font-bold text-gray-900">
          マイタスク
        </h1>
        <Link
          href="/tasks/new"
          className="inline-flex items-center px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700"
        >
          <PlusIcon className="w-5 h-5 mr-2" />
          新規タスク
        </Link>
      </div>

      <TaskList tasks={tasks || []} />
    </div>
  );
}
```

#### ステップ4-2：TaskListコンポーネントの作成

**あなたの指示：**
```
components/TaskList.tsx を作成してください。

要件：
- タスクの配列を props で受け取る
- 各タスクを TaskCard コンポーネントで表示
- 優先度別に色分け（高: 赤、中: 黄、低: 緑）
- 完了済みタスクは打ち消し線
- タスクがない場合は「タスクがありません」と表示
```

#### ステップ4-3：TaskCardコンポーネントの作成

**あなたの指示：**
```
components/TaskCard.tsx を作成してください。

要件：
- タスクの情報を表示（タイトル、説明、優先度、期限）
- 完了チェックボックス（トグルでis_completedを更新）
- 編集ボタン
- 削除ボタン
- 優先度別に左ボーダーの色を変更
- Tailwind CSS でモダンなカードデザイン

@lib/supabase.ts を使用してデータ更新を実装してください。
```

#### ステップ4-4：タスク作成フォーム

**あなたの指示：**
```
app/(dashboard)/tasks/new/page.tsx を作成してください。

要件：
- タイトル、説明、優先度、期限の入力フォーム
- React Hook Form でフォーム管理（インストール必要なら実行）
- Zod でバリデーション
- Supabase に新規タスクを INSERT
- 作成成功後は /tasks にリダイレクト
- キャンセルボタンで /tasks に戻る
```

**AIが自動的に：**
1. `npm install react-hook-form zod @hookform/resolvers` を実行
2. フォームコンポーネントを生成
3. バリデーションスキーマを定義
4. Submit処理を実装

#### ステップ4-5：タスク編集機能

**あなたの指示：**
```
@tasks/new/page.tsx を参考に、
app/(dashboard)/tasks/[id]/edit/page.tsx を作成してください。

要件：
- URL パラメータから task id を取得
- 既存のタスク情報を取得してフォームに初期表示
- 編集内容を Supabase で UPDATE
- 更新成功後は /tasks にリダイレクト
```

**フェーズ4完了！所要時間：4時間**

---

### フェーズ5：デプロイとテスト（2時間）

#### ステップ5-1：Gitリポジトリの作成

**Cursorターミナルで：**
```bash
git init
git add .
git commit -m "Initial commit: TaskMaster app"
```

**GitHubで新規リポジトリ作成後：**
```bash
git remote add origin https://github.com/あなたのユーザー名/taskmaster.git
git branch -M main
git push -u origin main
```

#### ステップ5-2：Vercelへデプロイ

**Vercelダッシュボード（https://vercel.com）で：**

1. 「Add New Project」をクリック
2. GitHubリポジトリから `taskmaster` を選択
3. Environment Variables に以下を追加：
   - `NEXT_PUBLIC_SUPABASE_URL`
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
4. 「Deploy」ボタンをクリック

**数分後、デプロイ完了！**

あなたのアプリが `https://taskmaster-あなたのユーザー名.vercel.app` で公開されます。

#### ステップ5-3：動作確認

**以下をテストしましょう：**

- [ ] ユーザー登録ができるか
- [ ] ログインができるか
- [ ] タスクの作成ができるか
- [ ] タスクの完了チェックができるか
- [ ] タスクの編集ができるか
- [ ] タスクの削除ができるか
- [ ] ログアウトができるか

**すべてOKなら完成です！🎉**

---

## トラブルシューティング

### よくあるエラーと対処法

#### エラー1：「Supabase connection failed」

**原因：** 環境変数が正しく設定されていない

**対処法：**
```
.env.local ファイルを確認してください。
NEXT_PUBLIC_SUPABASE_URL と NEXT_PUBLIC_SUPABASE_ANON_KEY が正しく設定されているか確認。
Vercel の Environment Variables も同様にチェック。
```

#### エラー2：「Row Level Security policy violation」

**原因：** RLSポリシーが正しく設定されていない

**対処法：**
```
Supabase SQL Editor で以下を実行：

SELECT * FROM tasks;

エラーが出る場合、ポリシーを再度確認。
schema.sql のポリシー設定を再実行してください。
```

#### エラー3：「Module not found」

**原因：** 依存関係がインストールされていない

**対処法：**
```bash
npm install
# または
rm -rf node_modules package-lock.json
npm install
```

---

## 拡張アイデア：さらに機能を追加する

基本機能が完成したら、以下の機能を追加してみましょう。

### 追加機能1：タスクのフィルタリング

**あなたの指示：**
```
tasks/page.tsx にフィルタリング機能を追加してください。

- 優先度でフィルタ（全て・高・中・低）
- 完了状態でフィルタ（全て・未完了・完了済み）
- URLパラメータでフィルタ状態を保持
```

### 追加機能2：検索機能

**あなたの指示：**
```
タスクをタイトルと説明で検索できる機能を実装してください。

- 検索バーを追加
- リアルタイムで検索結果を表示
- Supabase の ilike 演算子を使用
```

### 追加機能3：ダークモード

**あなたの指示：**
```
next-themes を使用してダークモードを実装してください。

- トグルボタンをヘッダーに追加
- ライト/ダークテーマの切り替え
- ローカルストレージに保存
```

---

## まとめ：あなたもWebサービスが作れる時代

**この記事で学んだこと：**

1. **PRD（要件定義書）の作成** - AIと一緒に設計
2. **プロジェクトセットアップ** - 自動化された足場固め
3. **データベース設計** - Supabaseでの実装
4. **認証システム** - ログイン・登録機能
5. **CRUD操作** - タスクの作成・読取・更新・削除
6. **デプロイ** - Vercelで本番公開

**実際の開発時間：**
- Day 1（土曜日）：8時間
- Day 2（日曜日）：6時間
- **合計：14時間**

従来なら数週間〜数ヶ月かかる開発が、**週末で完成**しました。

## 次のステップ

1. **このアプリをカスタマイズ**
   - 自分好みのデザインに変更
   - 新機能を追加
   - 友達に使ってもらう

2. **別のアイデアで新しいアプリ作成**
   - ブログシステム
   - 在庫管理アプリ
   - SNSクローン

3. **より高度なツールに挑戦**
   - Claude Code で大規模プロジェクト
   - Antigravity で複数サービス同時開発

**バイブコーディングの世界へようこそ。あなたのアイデアを、今すぐ形にしましょう！**

---

**📚 関連記事：**
- [プログラミング初心者のためのバイブコーディング入門](#)
- [バイブコーディングツール徹底比較](#)
- [AIが10倍賢くなる！プロンプトの書き方完全ガイド](#)

**🔗 参考Web記事：**
- [Next.js公式ドキュメント](https://nextjs.org/docs)
- [Supabase公式ドキュメント](https://supabase.com/docs)
- [Tailwind CSS公式ドキュメント](https://tailwindcss.com/docs)
- [Vercel デプロイガイド](https://vercel.com/docs)
- [TypeScript公式ハンドブック](https://www.typescriptlang.org/docs/handbook/intro.html)
- [React Hook Form ドキュメント](https://react-hook-form.com/get-started)