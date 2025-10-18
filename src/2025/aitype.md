# .cursorrules
<!--
date = "2025-10-22"
-->

## 基本方針
- **日本語対応**: すべてのコメント、変数名、ドキュメントは日本語で記述
- **ステップバイステップ**: 大きな変更は小さなステップに分割し、各ステップで動作確認
- **エラーハンドリング**: すべてのAPI呼び出し、データベース操作にはtry-catch/エラーハンドリングを実装

## 技術スタック

### バックエンド
- **言語**: Python 3.9
- **フレームワーク**: FastAPI
- **ORM**: SQLAlchemy
- **データベース**: PostgreSQL
- **スケジューラー**: APScheduler
- **LINE SDK**: linebot-v3

### フロントエンド
- **言語**: TypeScript
- **フレームワーク**: React 18
- **ルーティング**: React Router v6
- **スタイリング**: Tailwind CSS
- **フォーム**: React Hook Form
- **日付処理**: date-fns
- **HTTP**: axios

## コーディング規約

### Python (バックエンド)
```python
# ファイル構造
routers/       # APIエンドポイント
services/      # ビジネスロジック
models.py      # データモデル
database.py    # DB接続

# 命名規則
- 関数名: snake_case (例: get_reminder_settings)
- クラス名: PascalCase (例: ReminderService)
- 定数: UPPER_SNAKE_CASE (例: DATABASE_URL)
- プライベート: _prefix (例: _internal_function)

# 型ヒント必須
def create_booking(booking: BookingCreate, db: Session) -> Booking:
    ...

# ロギング
logger.info("✅ 成功メッセージ")
logger.error("❌ エラーメッセージ")
logger.warning("⚠️ 警告メッセージ")

# Pydanticモデル
- すべてのAPIリクエスト/レスポンスにPydanticモデルを使用
- BaseModelを継承
```

### TypeScript (フロントエンド)
```typescript
// ファイル構造
pages/         # ページコンポーネント
components/    # 再利用可能なコンポーネント
services/      # API通信
types/         # 型定義
utils/         # ユーティリティ関数

// 命名規則
- コンポーネント: PascalCase (例: CustomerList.tsx)
- 関数: camelCase (例: fetchCustomers)
- 定数: UPPER_SNAKE_CASE (例: API_URL)
- 型/インターフェース: PascalCase (例: Customer)

// コンポーネント構造
const MyComponent: React.FC = () => {
  // 1. useState
  // 2. useEffect
  // 3. カスタムフック
  // 4. イベントハンドラ
  // 5. return JSX
};

// async/await必須（Promiseチェーンは使わない）
const fetchData = async () => {
  try {
    const response = await api.get('/endpoint');
    return response.data;
  } catch (error) {
    console.error('エラー:', error);
    throw error;
  }
};
```

## データベース操作

### SQLAlchemy
```python
# トランザクション管理必須
try:
    db.add(instance)
    db.commit()
    db.refresh(instance)
except Exception as e:
    db.rollback()
    raise

# N+1問題回避
# ✅ Good: joinedloadを使用
bookings = db.query(Booking).options(
    joinedload(Booking.customer),
    joinedload(Booking.staff)
).all()

# ❌ Bad: 個別にクエリ
bookings = db.query(Booking).all()
for booking in bookings:
    customer = booking.customer  # N+1発生
```

### マイグレーション
- Alembicは使用しない
- SQLファイルを直接実行するスクリプトを作成
- `create_*.sql` + `apply_*.py` のペアで管理

## API設計

### エンドポイント命名
```
GET    /api/resources/           # 一覧取得
GET    /api/resources/{id}       # 詳細取得
POST   /api/resources/           # 作成
PUT    /api/resources/{id}       # 更新
DELETE /api/resources/{id}       # 削除
POST   /api/resources/bulk       # 一括操作
```

### レスポンス形式
```python
# 成功
return {
    "id": "uuid",
    "field": "value"
}

# エラー
raise HTTPException(
    status_code=400,
    detail="エラーメッセージ（日本語）"
)
```

## フロントエンド設計

### 状態管理
- useState: コンポーネント内部の状態
- useEffect: 副作用（API呼び出し、イベントリスナー）
- グローバル状態管理は不使用（必要に応じて検討）

### UIパターン
```typescript
// ローディング状態
const [loading, setLoading] = useState(false);

// エラー状態
const [error, setError] = useState<string | null>(null);

// Tailwind CSS
// - レスポンシブ: sm: md: lg: xl:
// - カラー: blue-600, gray-100, red-500
// - スペーシング: p-4, m-2, space-y-4
```

## テスト・デバッグ

### バックエンド
```bash
# API動作確認
curl -X POST http://localhost:8000/api/endpoint \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}' | python3 -m json.tool

# データベース確認
psql $DATABASE_URL -c "SELECT * FROM table_name;"
```

### フロントエンド
```typescript
// console.logパターン
console.log('🔍 [ComponentName] メッセージ:', data);
console.error('❌ [ComponentName] エラー:', error);

// デバッグ情報は開発中のみ残す
// 本番環境へのデプロイ前に削除
```

## Git コミットメッセージ

```
feat: 新機能追加
fix: バグ修正
refactor: リファクタリング
docs: ドキュメント更新
style: コードスタイル修正
test: テスト追加・修正
chore: ビルド、設定ファイル等の変更
```

## 禁止事項

### やってはいけないこと
❌ `--force` を使ったgit push
❌ `.env` ファイルのコミット
❌ ハードコードされたAPIキー、パスワード
❌ `any` 型の多用（TypeScript）
❌ エラーハンドリングなしのAPI呼び出し
❌ console.logの大量使用（本番環境）
❌ 同期的なブロッキング処理

### 避けるべきパターン
⚠️ グローバル変数の多用
⚠️ 複雑なネストされたif文（Early returnを推奨）
⚠️ 200行を超える関数・コンポーネント
⚠️ マジックナンバー（定数化推奨）

## パフォーマンス

### バックエンド
- データベースクエリは最小限に
- N+1問題を回避
- 適切なインデックスを設定

### フロントエンド
- React.memoでメモ化（必要な場合のみ）
- useMemo/useCallbackで最適化
- 画像の遅延読み込み

## セキュリティ

### バックエンド
- SQLインジェクション対策（SQLAlchemyのパラメータバインディング）
- CORS設定を適切に
- 環境変数で機密情報管理

### フロントエンド
- XSS対策（Reactのデフォルトエスケープを活用）
- CSRFトークンの実装（必要に応じて）
- 機密情報をローカルストレージに保存しない

## プロジェクト固有ルール

### リマインド機能
- 前日リマインド: app_settingsのグローバル設定を使用
- 1時間前リマインド: 個別予約の設定を使用
- LINE User IDは必須

### シフト管理
- ShiftTypeはEnum（MORNING, DAY, EVENING, CUSTOM）
- 1スタッフ1日1シフトまで（ユニーク制約）
- プリセットは論理削除（is_active）

### タイムゾーン
- データベース: UTC with timezone
- Python: pytz.timezone('Asia/Tokyo')
- JavaScript: date-fns-tz

## 開発フロー

1. **要件確認**: SOWを作成
2. **設計**: データモデル、API設計
3. **実装**: バックエンド → フロントエンド
4. **テスト**: 各ステップで動作確認
5. **デバッグ**: ログ確認、エラー修正
6. **リファクタリング**: コードの整理

## AIへの指示

- **日本語で応答**: すべての説明は日本語で
- **段階的実装**: 一度に多くの変更をしない
- **動作確認**: 各ステップで確認コマンドを実行
- **エラー対応**: エラーが出たら原因を分析し、解決策を提示
- **コード品質**: 読みやすく、保守しやすいコードを書く
- **ベストプラクティス**: 各技術のベストプラクティスに従う

---

このルールはプロジェクトの成長に合わせて更新してください。

