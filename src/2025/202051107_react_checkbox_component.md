# React + TypeScriptでチェックボックスコンポーネントを作る

<!--
date = "2025-11-07"
-->

## フォームを作ると必ず出てくる問題

Reactでフォームを作っていると、こんな経験はありませんか。

「チェックボックスをあちこちで使うのに、毎回HTMLを書くのは面倒だ」
「React Hook Formと連携させたいけど、refの渡し方がわからない」
「ラベルをクリックしてもチェックが入らない」

本記事では、これらの問題をすべて解決する、再利用可能なチェックボックスコンポーネントを作ります。コードは30行以下です。

## 先に完成形を見る

まず、最終的に作るコンポーネントがどう動くかを見ます。
```typescriptreact
// 使い方
<Checkbox label="利用規約に同意する" />
<Checkbox label="通知を受け取る" disabled />
<Checkbox label="購読する" onChange={(e) => console.log(e.target.checked)} />
```

たった1行で、スタイル付き・アクセシビリティ対応・イベント処理可能なチェックボックスが使えます。

実装コードは次の通りです。
```typescriptreact
import { InputHTMLAttributes, forwardRef } from 'react'

interface CheckboxProps extends InputHTMLAttributes<HTMLInputElement> {
  label?: string
}

export const Checkbox = forwardRef<HTMLInputElement, CheckboxProps>(
  ({ label, className = '', ...props }, ref) => {
    return (
      <label className="flex items-center gap-2 cursor-pointer">
        <input
          ref={ref}
          type="checkbox"
          className={`w-5 h-5 text-blue-500 rounded focus:ring-2 focus:ring-blue-500 ${className}`}
          {...props}
        />
        {label && <span className="text-sm text-gray-700">{label}</span>}
      </label>
    )
  }
)

Checkbox.displayName = 'Checkbox'
```

このコードには、4つの重要な技術が含まれています。順番に解説します。

## 技術1: InputHTMLAttributesで標準属性を全部受け取る
```typescript
interface CheckboxProps extends InputHTMLAttributes<HTMLInputElement> {
  label?: string
}
```

`InputHTMLAttributes<HTMLInputElement>`は、HTML標準のinput要素が持つ属性の型定義です。これを継承することで、次のような属性がすべて使えます。

- `disabled`: 無効化
- `checked`: チェック状態
- `onChange`: 変更イベント
- `name`: フォーム送信時の名前
- `value`: 値

これにより、「あれ、この属性使えないの?」という問題が起きません。標準のinput要素でできることは、すべて自動的に使えます。

`label?: string`は、チェックボックスの横に表示するテキストです。`?`は「省略可能」を意味します。

## 技術2: forwardRefでDOM要素への参照を渡す
```typescript
export const Checkbox = forwardRef<HTMLInputElement, CheckboxProps>(
  ({ label, className = '', ...props }, ref) => {
    // 中身
  }
)
```

`forwardRef`は、親コンポーネントから子コンポーネントのDOM要素に直接アクセスできるようにする機能です。

なぜこれが必要なのか。React Hook Formなどのフォームライブラリを使う場合、次のようなコードを書きます。
```typescriptreact
const { register } = useForm()
<Checkbox {...register('agreeToTerms')} />
```

この`register`が、内部でDOM要素への参照（ref）を必要とします。`forwardRef`を使わないと、このコードはエラーになります。

`forwardRef<参照先の型, Propsの型>`という形で、2つの型を指定します。

- 第一引数: `HTMLInputElement`（input要素の型）
- 第二引数: `CheckboxProps`（コンポーネントのPropsの型）

関数の引数として、`(props, ref)`を受け取ります。`props`は通常のPropsで、`ref`は親から渡された参照です。

`{ label, className = '', ...props }`は、分割代入です。`label`と`className`を取り出し、残りを`props`としてまとめます。

## 技術3: labelでクリック範囲を広げる
```typescriptreact
<label className="flex items-center gap-2 cursor-pointer">
  <input ... />
  {label && <span>{label}</span>}
</label>
```

input要素を`label`で囲むと、テキスト部分をクリックしてもチェックボックスが反応します。これは、アクセシビリティの基本です。

`className="flex items-center gap-2 cursor-pointer"`は、Tailwind CSSのクラスです。

- `flex`: 横並び配置
- `items-center`: 縦方向の中央揃え
- `gap-2`: 要素間の余白
- `cursor-pointer`: マウスカーソルを手の形にする

`{label && <span>{label}</span>}`は、条件付きレンダリングです。`label`が渡されている場合のみ、span要素を表示します。JavaScriptの`&&`演算子は、左側が真なら右側を評価します。

## 技術4: スプレッド構文で属性を展開する
```typescriptreact
<input
  ref={ref}
  type="checkbox"
  className={`w-5 h-5 text-blue-500 rounded focus:ring-2 focus:ring-blue-500 ${className}`}
  {...props}
/>
```

`{...props}`は、残りのすべてのPropsをinput要素に展開します。これにより、次のような使い方がすべて動作します。
```typescriptreact
<Checkbox disabled />
<Checkbox checked onChange={handleChange} />
<Checkbox name="newsletter" value="subscribe" />
```

`className`では、テンプレートリテラルを使って、デフォルトのスタイルと外部から渡されたスタイルを結合しています。
```typescript
`デフォルトのクラス ${追加のクラス}`
```

これにより、基本的な見た目を維持しながら、必要に応じてカスタマイズできます。

## displayNameでデバッグしやすくする
```typescript
Checkbox.displayName = 'Checkbox'
```

`forwardRef`を使うと、React DevToolsで「ForwardRef」と表示されます。`displayName`を設定することで、「Checkbox」と表示されるようになります。

開発中にコンポーネントを特定しやすくなるため、必ず設定してください。

## 実際に使ってみる
```typescriptreact
import { useState } from 'react'
import { Checkbox } from './Checkbox'

function App() {
  const [agreed, setAgreed] = useState(false)
  
  return (
    <div>
      <Checkbox 
        label="利用規約に同意する" 
        checked={agreed}
        onChange={(e) => setAgreed(e.target.checked)}
      />
      <button disabled={!agreed}>登録する</button>
    </div>
  )
}
```

このコードでは、チェックボックスの状態を`useState`で管理し、同意しないとボタンが押せないようにしています。

React Hook Formと連携させる場合は、次のようになります。
```typescriptreact
import { useForm } from 'react-hook-form'
import { Checkbox } from './Checkbox'

function App() {
  const { register, handleSubmit } = useForm()
  
  const onSubmit = (data) => {
    console.log(data) // { agreeToTerms: true }
  }
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Checkbox 
        label="利用規約に同意する" 
        {...register('agreeToTerms')}
      />
      <button type="submit">登録する</button>
    </form>
  )
}
```

`{...register('agreeToTerms')}`が、自動的に`ref`や`onChange`などを設定します。`forwardRef`を使っているため、これが正常に動作します。

## まとめ

本記事では、4つの技術を使ってチェックボックスコンポーネントを作成しました。

1. `InputHTMLAttributes`で標準属性をすべて使えるようにする
2. `forwardRef`で外部からDOM要素にアクセスできるようにする
3. `label`要素でクリック範囲を広げる
4. スプレッド構文で属性を柔軟に渡せるようにする

このコンポーネントは、フォームライブラリとの連携やカスタムスタイルの追加など、様々な場面で使用できます。

## 参考記事

- [React公式ドキュメント - forwardRef](https://react.dev/reference/react/forwardRef)
- [TypeScript公式ドキュメント - Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)
- [MDN - label要素](https://developer.mozilla.org/ja/docs/Web/HTML/Element/label)
- [React Hook Form - register API](https://react-hook-form.com/docs/useform/register)
- [Tailwind CSS - Form Elements](https://tailwindcss.com/docs/forms)