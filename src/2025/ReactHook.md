# React Hookフォームで空欄項目を正しくAPIに送信する方法
<!--
date = "2025-10-18"
-->

## 問題の概要

予約画面から新規顧客を登録しようとすると「顧客登録に失敗しました」というエラーが発生していました。

## エラーの原因

主な原因は**空文字列の扱い**でした。

フォームでオプショナル項目(メールアドレス、LINE User IDなど)を空欄のままにすると、空文字列(`""`)として送信されます。しかし、バックエンドのAPIは以下のいずれかを期待していました:

- 値が入力されている場合: 実際の値(例: `"tanaka@example.com"`)
- 値が入力されていない場合: `null`または`undefined`

空文字列(`""`)を送ると、バリデーションエラーが発生していたのです。

## 解決方法

フォーム送信時に**空文字列を`undefined`に変換**することで解決しました。

### 修正前のコード
```typescript
const handleFormSubmit = (data: CustomerCreate) => {
  onSubmit(data); // そのまま送信
};
```

### 修正後のコード
```typescript
const handleFormSubmit = (data: CustomerCreate) => {
  const cleanedData: CustomerCreate = {
    display_name: data.display_name,
    phone: data.phone,
    // 空文字列をundefinedに変換
    line_user_id: data.line_user_id?.trim() || undefined,
    email: data.email?.trim() || undefined,
    notes: data.notes?.trim() || undefined,
  };
  onSubmit(cleanedData);
};
```

### 仕組みの説明

`data.email?.trim() || undefined`は以下のように動作します:

1. `data.email?.trim()` - 値があれば前後の空白を削除
2. `|| undefined` - 値が空文字列なら`undefined`に変換

## その他の改善点

### バリデーションの追加

より詳細なエラーメッセージを表示できるようにしました。
```typescript
register('phone', {
  required: '電話番号は必須です',
  pattern: {
    value: /^[0-9\-+()]{10,20}$/,
    message: '正しい電話番号を入力してください',
  },
})
```

### デフォルト値の明示的な設定

新規登録時のフォームを安定させるため、すべてのフィールドに初期値を設定しました。
```typescript
defaultValues: customer ? { ... } : {
  line_user_id: '',
  display_name: '',
  phone: '',
  email: '',
  notes: '',
}
```

## 学んだこと

1. **空文字列とnull/undefinedは違う** - APIが期待する形式を確認する
2. **フォームデータは送信前に整形する** - バリデーションだけでなくデータ変換も重要
3. **エラーログを確認する** - ブラウザのコンソールとネットワークタブが問題解決の鍵

## まとめ

オプショナルなフォーム項目を扱う際は、空文字列を適切に処理することが重要です。`trim()`で空白を削除し、空の場合は`undefined`に変換することで、バックエンドとの連携がスムーズになります。