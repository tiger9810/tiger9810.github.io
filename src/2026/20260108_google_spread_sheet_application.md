# GoogleスプレッドシートをDBとして使うNext.jsアプリ構築ハンズオン
<!--
date = "2026-01-08"
-->
## 編集方針

この記事では、以下の内容が：

1. **完全無料で運用できるWebアプリケーション**を構築できる
2. **非エンジニアでもデータ更新が可能**な仕組みを理解できる
3. **Next.jsのServer Components**を活用した実践的なデータ取得パターンを習得できる
4. **よくある落とし穴**（環境変数の設定、日付ソートなど）を事前に回避できる

初心者でも迷わず実装できるよう、各ステップで「なぜそうするのか」を明確に説明し、実際のコードに詳細なコメントを付与します。

---

## 1. 導入：バイブコーディングとスプレッドシートDBのメリット

個人開発者やスタートアップにとって、データベースの運用コストは無視できません。PostgreSQLやMySQLをホスティングするには月額費用がかかり、SupabaseやFirebaseも無料枠には制限があります。

そこで注目したいのが、**Googleスプレッドシートをデータベースとして活用する**アプローチです。

### スプレッドシートDBのメリット

- ✅ **完全無料**：Googleアカウントがあれば利用可能
- ✅ **非エンジニアでも更新可能**：UIが直感的で、CSVエクスポートも簡単
- ✅ **リアルタイム更新**：スプレッドシートを更新すると、次回のリクエストで反映
- ✅ **バージョン管理**：Googleスプレッドシートの変更履歴機能を活用可能
- ✅ **簡単なバックアップ**：Googleドライブに自動保存

### この記事で構築するもの

「Mission Control」という個人開発者向けダッシュボードを題材に、以下の機能を実装します：

- スプレッドシートからKPIデータを取得して表示
- 月次トレンドデータをグラフ化（Recharts）
- プロジェクト一覧の動的表示
- ISR（Incremental Static Regeneration）による効率的なデータ更新

---

## 2. Google Cloudの設定：サービスアカウントのセットアップ

スプレッドシートをプログラムから読み取るには、**サービスアカウント**を使用します。これは、人間のユーザーではなく、アプリケーション専用のアカウントです。

### ステップ1: Google Cloud Consoleでプロジェクトを作成

1. [Google Cloud Console](https://console.cloud.google.com/)にアクセス
2. 新しいプロジェクトを作成（例: `mission-control-dashboard`）
3. プロジェクトを選択

### ステップ2: Google Sheets APIを有効化

1. 「APIとサービス」→「ライブラリ」に移動
2. 「Google Sheets API」を検索して有効化

### ステップ3: サービスアカウントを作成

1. 「IAMと管理」→「サービスアカウント」に移動
2. 「サービスアカウントを作成」をクリック
3. 名前を入力（例: `sheets-reader`）
4. 「作成して続行」をクリック
5. ロールは設定せず、「完了」をクリック

### ステップ4: 認証情報（JSONキー）をダウンロード

1. 作成したサービスアカウントをクリック
2. 「キー」タブに移動
3. 「キーを追加」→「新しいキーを作成」
4. キーのタイプ: **JSON**を選択
5. JSONファイルがダウンロードされます（**このファイルは絶対に公開しないでください**）

### ステップ5: スプレッドシートを共有

1. 使用するGoogleスプレッドシートを開く
2. 「共有」ボタンをクリック
3. サービスアカウントのメールアドレス（JSONファイル内の`client_email`）を入力
4. 権限は「閲覧者」でOK（読み取り専用）

### ステップ6: スプレッドシートIDを取得

スプレッドシートのURLからIDを抽出します：

```
https://docs.google.com/spreadsheets/d/[ここがスプレッドシートID]/edit
```

例: `https://docs.google.com/spreadsheets/d/1a2b3c4d5e6f7g8h9i0j/edit`
→ スプレッドシートIDは `1a2b3c4d5e6f7g8h9i0j`

---

## 3. 環境構築とライブラリ導入

### プロジェクトのセットアップ

```bash
# Next.jsプロジェクトを作成（TypeScript、App Router使用）
npx create-next-app@latest mission-control-dashboard --typescript --app

# 必要なライブラリをインストール
npm install google-spreadsheet google-auth-library
```

### 環境変数の設定

プロジェクトルートに `.env.local` ファイルを作成します：

```env
# サービスアカウントのメールアドレス（JSONファイル内のclient_email）
GOOGLE_SERVICE_ACCOUNT_EMAIL=sheets-reader@your-project.iam.gserviceaccount.com

# サービスアカウントの秘密鍵（JSONファイル内のprivate_key）
# 注意: \nを\\nにエスケープする必要があります
GOOGLE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nMIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQC...\n-----END PRIVATE KEY-----\n"

# スプレッドシートID（URLから抽出）
GOOGLE_SHEET_ID=1a2b3c4d5e6f7g8h9i0j
```

**⚠️ 重要な注意点：**

- `.env.local`は`.gitignore`に含まれていることを確認してください
- `GOOGLE_PRIVATE_KEY`の改行文字（`\n`）は、JSONファイルからコピーする際に`\\n`にエスケープする必要があります
- 本番環境（Vercel等）では、環境変数を設定画面から追加してください

### 環境変数のバリデーション

環境変数が正しく設定されているか確認するためのユーティリティを作成します：

```typescript:src/lib/env.ts
/**
 * 環境変数の型定義とバリデーション
 */

interface Env {
  GOOGLE_SERVICE_ACCOUNT_EMAIL: string;
  GOOGLE_PRIVATE_KEY: string;
  GOOGLE_SHEET_ID: string;
}

/**
 * 環境変数を取得し、必須項目が設定されているか確認する
 * @returns 環境変数のオブジェクト
 * @throws 必須環境変数が設定されていない場合
 */
export function getEnv(): Env {
  const email = process.env.GOOGLE_SERVICE_ACCOUNT_EMAIL;
  const privateKey = process.env.GOOGLE_PRIVATE_KEY;
  const sheetId = process.env.GOOGLE_SHEET_ID;

  // 必須環境変数が設定されていない場合はエラーを投げる
  if (!email) {
    throw new Error("GOOGLE_SERVICE_ACCOUNT_EMAIL is not set");
  }
  if (!privateKey) {
    throw new Error("GOOGLE_PRIVATE_KEY is not set");
  }
  if (!sheetId) {
    throw new Error("GOOGLE_SHEET_ID is not set");
  }

  return {
    GOOGLE_SERVICE_ACCOUNT_EMAIL: email,
    // 環境変数内の\\nを実際の改行文字に変換
    // これがないと、秘密鍵が正しくパースされません
    GOOGLE_PRIVATE_KEY: privateKey.replace(/\\n/g, "\n"),
    GOOGLE_SHEET_ID: sheetId,
  };
}
```

---

## 4. 実装：スプレッドシートからのデータ取得（Server Components）

Next.js 14のApp Routerでは、**Server Components**がデフォルトです。これにより、サーバー側でデータを取得し、クライアントに送信する前に処理できます。

### Google Sheetsクライアントの実装

```typescript:src/lib/google-sheets.ts
import { GoogleSpreadsheet } from "google-spreadsheet";
import { JWT } from "google-auth-library";
import { getEnv } from "@/src/lib/env";

/**
 * Google Sheetsからデータを取得するためのクライアント
 */
class GoogleSheetsClient {
  private doc: GoogleSpreadsheet;
  private jwt: JWT;
  private docLoaded: boolean = false;

  constructor() {
    const env = getEnv();
    
    // JWT（JSON Web Token）認証を設定
    // サービスアカウントの認証情報を使用してGoogle APIにアクセス
    this.jwt = new JWT({
      email: env.GOOGLE_SERVICE_ACCOUNT_EMAIL,
      key: env.GOOGLE_PRIVATE_KEY,
      scopes: ["https://www.googleapis.com/auth/spreadsheets.readonly"], // 読み取り専用スコープ
    });

    // スプレッドシートのインスタンスを作成
    this.doc = new GoogleSpreadsheet(env.GOOGLE_SHEET_ID, this.jwt);
  }

  /**
   * スプレッドシートの情報を読み込む（一度だけ実行）
   * パフォーマンス向上のため、複数回のAPI呼び出しを避ける
   */
  private async ensureDocLoaded(): Promise<void> {
    if (!this.docLoaded) {
      await this.doc.loadInfo(); // シート一覧などのメタ情報を取得
      this.docLoaded = true;
    }
  }

  /**
   * 「projects」シートからデータを取得する
   * @returns ProjectSheetRow配列
   */
  async getProjectsData(): Promise<ProjectSheetRow[]> {
    try {
      await this.ensureDocLoaded();
      
      // シート名でシートを取得（タブ名が「projects」のシート）
      const sheet = this.doc.sheetsByTitle["projects"];

      if (!sheet) {
        throw new Error('Sheet "projects" not found');
      }

      // すべての行を取得（1行目はヘッダーとして自動的に認識される）
      const rows = await sheet.getRows();
      
      // 各行をオブジェクトに変換
      return rows.map((row) => ({
        id: String(row.get("id") || ""), // 列名「id」の値を取得
        name: String(row.get("name") || ""),
        status: String(row.get("status") || ""),
        users: parseNumber(row.get("users")), // 数値に変換（空文字列の場合は0）
        revenue: parseNumber(row.get("revenue")),
        release_date: String(row.get("release_date") || ""),
      }));
    } catch (error) {
      throw new Error(
        `Failed to fetch projects data: ${error instanceof Error ? error.message : String(error)}`
      );
    }
  }
}

/**
 * ダッシュボード全体のデータを一括取得する
 * Promise.allを使用して並列取得することで、パフォーマンスを向上
 */
export async function getDashboardData(): Promise<DashboardData> {
  const client = new GoogleSheetsClient();

  // 3つのシートから並列でデータを取得
  const [summary, projects, trends] = await Promise.all([
    client.getSummaryData(),
    client.getProjectsData(),
    client.getTrendsData(),
  ]);

  return {
    summary,
    projects,
    trends,
  };
}
```

### Server Componentでのデータ取得

```typescript:app/page.tsx
import { getDashboardPageData } from "@/src/lib/dashboard";

/**
 * データ再検証の間隔（3時間 = 10800秒）
 * ISR（Incremental Static Regeneration）を使用して、
 * 定期的にデータを再取得し、静的ページを更新
 */
export const revalidate = 10800;

/**
 * ダッシュボードページ（Server Component）
 * サーバー側でデータを取得し、クライアントに送信
 */
export default async function DashboardPage() {
  // Server Componentでは、async/awaitを直接使用できる
  const { kpiData, monthlyData, projects } = await getDashboardPageData();

  return (
    <div className="flex min-h-screen flex-col">
      <Header />
      <div className="flex-1 p-8">
        <div className="mx-auto max-w-7xl">
          <DashboardContent
            kpiData={kpiData}
            monthlyData={monthlyData}
            projects={projects}
          />
        </div>
      </div>
    </div>
  );
}
```

**ポイント：**

- `export const revalidate = 10800`により、3時間ごとにデータを再取得します
- Server Componentでは、`async`関数を直接使用できます
- データ取得はサーバー側で実行されるため、APIキーなどの機密情報がクライアントに露出しません

---

## 5. 実装：データのソートとグラフ表示（Recharts連動のコツ）

スプレッドシートから取得したデータは、必ずしも正しい順序で並んでいるとは限りません。特に日付データは、文字列として比較すると「2025/10」が「2025/5」より前に来てしまう問題があります。

### 日付ソートの実装

```typescript:src/lib/utils/date.ts
/**
 * 月次データ（YYYY-MM形式）をDateオブジェクトに変換する
 * @param monthStr 月次文字列（例: "2025-05"）
 * @returns Dateオブジェクト（その月の1日）
 */
export function parseMonthToDate(monthStr: string): Date {
  // "2025-05" -> "2025-05-01" に変換してDateオブジェクトを作成
  const [year, month] = monthStr.split("-");
  // Dateコンストラクタの月は0始まりなので、-1する
  return new Date(parseInt(year, 10), parseInt(month, 10) - 1, 1);
}

/**
 * 月次データ配列を時系列順にソートする
 * 文字列比較ではなく、Dateオブジェクトに変換して比較することで、
 * 「2025/10」が「2025/5」より前に来る問題を解決
 */
export function sortByMonth<T>(
  data: T[],
  getMonth: (item: T) => string
): T[] {
  return [...data].sort((a, b) => {
    const dateA = parseMonthToDate(getMonth(a));
    const dateB = parseMonthToDate(getMonth(b));
    // Dateオブジェクトのタイムスタンプで比較
    return dateA.getTime() - dateB.getTime();
  });
}
```

### データ取得時のソート適用

```typescript:src/lib/google-sheets.ts
async getTrendsData(): Promise<TrendRow[]> {
  try {
    await this.ensureDocLoaded();
    const sheet = this.doc.sheetsByTitle["trends"];

    if (!sheet) {
      throw new Error('Sheet "trends" not found');
    }

    const rows = await sheet.getRows();
    const data: TrendRow[] = rows.map((row) => ({
      month: String(row.get("month") || ""),
      mau: Number(parseNumber(row.get("mau"))),
      pv: Number(parseNumber(row.get("pv"))),
      mr: Number(parseNumber(row.get("mr"))),
    }));

    // ⚠️ 重要: 時系列順にソート
    // これがないと、グラフのX軸が正しい順序で表示されません
    return sortByMonth(data, (item) => item.month);
  } catch (error) {
    throw new Error(
      `Failed to fetch trends data: ${error instanceof Error ? error.message : String(error)}`
    );
  }
}
```

### Rechartsでのグラフ表示

```typescript:components/GrowthLog.tsx
"use client";

import { LineChart, Line, XAxis, YAxis, Tooltip, Legend, ResponsiveContainer } from "recharts";
import { useMemo } from "react";
import { sortByMonth } from "@/src/lib/utils/date";

interface GrowthLogProps {
  data: MonthlyDataPoint[];
  selectedMetric: "mau" | "pv" | "mr";
}

export default function GrowthLog({ data, selectedMetric }: GrowthLogProps) {
  // useMemoでソート処理をメモ化（パフォーマンス向上）
  const sortedData = useMemo(() => {
    return sortByMonth(data, (item) => item.month);
  }, [data]);

  return (
    <ResponsiveContainer width="100%" height={400}>
      <LineChart data={sortedData}>
        <XAxis 
          dataKey="month" 
          // X軸の表示を「2025-05」から「2025/05」に変換
          tickFormatter={(value) => value.replace("-", "/")}
        />
        <YAxis />
        <Tooltip />
        <Legend />
        <Line 
          type="monotone" 
          dataKey={selectedMetric} 
          stroke="#2563eb" 
          strokeWidth={2}
        />
      </LineChart>
    </ResponsiveContainer>
  );
}
```

**よくある落とし穴：**

1. **文字列比較によるソート失敗**
   - `["2025/10", "2025/5"]`をソートすると、文字列比較では`"2025/10" < "2025/5"`となり、順序が逆になります
   - 解決策: Dateオブジェクトに変換して比較

2. **データ取得時の型変換忘れ**
   - スプレッドシートから取得した値は文字列なので、数値として使用する場合は`Number()`で変換が必要

3. **環境変数の改行文字エスケープ**
   - JSONファイルから秘密鍵をコピーする際、`\n`を`\\n`にエスケープする必要があります

---

## 6. まとめ：完全に無料で運用する戦略

この構成により、以下のコストでWebアプリケーションを運用できます：

### 運用コスト

- ✅ **Googleスプレッドシート**: 無料（15GBまで）
- ✅ **Next.js（Vercel）**: 無料（Hobbyプラン）
- ✅ **ドメイン**: 任意（無料の`.vercel.app`ドメインも使用可能）

### スケーラビリティの考慮

- **読み取り頻度**: Google Sheets APIの無料枠は1日あたり500リクエストまで
- **データ量**: スプレッドシートは最大500万セルまで
- **同時アクセス**: Vercelの無料プランでも十分なパフォーマンス

### 次のステップ

1. **エラーハンドリングの強化**: スプレッドシートが存在しない場合のフォールバック
2. **キャッシュ戦略**: Redis等を使用したデータキャッシュ（オプション）
3. **データ検証**: Zod等を使用したスキーマバリデーション
4. **監視**: Vercel AnalyticsやSentryでエラー監視

### まとめ

GoogleスプレッドシートをDBとして活用することで、**完全無料で運用可能なWebアプリケーション**を構築できます。非エンジニアでもデータ更新が可能なため、個人開発者やスタートアップに最適な構成です。

Next.jsのServer ComponentsとISRを組み合わせることで、パフォーマンスとコストのバランスが取れたアプリケーションを実現できます。

---

## 参考リソース

- [Google Sheets API ドキュメント](https://developers.google.com/sheets/api)
- [Next.js Server Components](https://nextjs.org/docs/app/building-your-application/rendering/server-components)
- [google-spreadsheet ライブラリ](https://theoephraim.github.io/node-google-spreadsheet/)
- [Vercel デプロイガイド](https://vercel.com/docs)

---

**著者**: Mission Control開発チーム  
**公開日**: 2025年1月  
**ライセンス**: MIT

