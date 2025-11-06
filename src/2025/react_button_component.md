# Reactのボタンコンポーネントを1行ずつ解説する

<!--
date = "2025-11-5"
-->

## この記事で何を解説するのか

実務でよく使われるボタンコンポーネントのコードを、1行ずつ分解して解説します。このコードには、Reactの重要な概念が詰まっています。

- 型とは何か
- propsの拡張とは何か
- デフォルト値の設定
- スプレッド構文
- 条件分岐

読み終わる頃には、このコードがなぜこう書かれているのか、理解できるようになるはずです。

## 前提知識: 「型」とは何か

コードを読む前に、「型」という概念を理解する必要があります。

### 型の基本

型とは、**データの種類を指定するルール**です。
```typescript
let name: string = "田中";  // 文字列型
let age: number = 25;       // 数値型
let isStudent: boolean = true;  // 真偽値型
```

- `string`: 文字列だけ入る
- `number`: 数値だけ入る
- `boolean`: trueかfalseだけ入る

### なぜ型が必要なのか

型があると、**間違いを防げる**からです。
```typescript
let age: number = 25;
age = "二十五才";  // エラー! 数値以外は入れられない
```

TypeScriptは、コードを書いている時点でエラーを教えてくれます。

### 型定義とは

型定義とは、**どんなデータを受け取るかの設計図**です。
```typescript
type UserData = {
  name: string;
  age: number;
};
```

この設計図は「`name`は文字列、`age`は数値を受け取る」という約束です。
```typescript
// OK
const user: UserData = {
  name: "田中",
  age: 25
};

// エラー! ageが文字列になっている
const user2: UserData = {
  name: "佐藤",
  age: "25才"
};
```

この「型定義」を理解すると、これから読むコードが理解できます。

## コード全体
```typescript
import { ButtonHTMLAttributes, ReactNode } from 'react'

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  children: ReactNode
  variant?: 'primary' | 'secondary' | 'danger'
  isLoading?: boolean
}

export function Button({
  children,
  variant = 'primary',
  isLoading = false,
  className = '',
  disabled,
  ...props
}: ButtonProps) {
  const baseClasses = 'px-4 py-2 rounded-lg font-medium transition-colors disabled:opacity-50 disabled:cursor-not-allowed'
  
  const variantClasses = {
    primary: 'bg-blue-500 text-white hover:bg-blue-600',
    secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300',
    danger: 'bg-red-500 text-white hover:bg-red-600',
  }

  return (
    <button
      className={`${baseClasses} ${variantClasses[variant]} ${className}`}
      disabled={disabled || isLoading}
      {...props}
    >
      {isLoading ? '読み込み中...' : children}
    </button>
  )
}
```

## 1行目: import文
```typescript
import { ButtonHTMLAttributes, ReactNode } from 'react'
```

### 何をしているか

Reactから2つの型定義をインポートしています。

### `ButtonHTMLAttributes<HTMLButtonElement>`とは

HTMLの`<button>`タグが持つすべての属性の型定義です。

**具体的には:**
- `onClick`: クリック時の処理を書く関数
- `disabled`: ボタンを無効化するか(true/false)
- `type`: ボタンのタイプ("button", "submit", "reset")
- `className`: CSSのクラス名(文字列)
- `id`, `style`, `aria-*`など、100以上の属性

**つまり、普通のHTMLボタンに書けることすべて**が含まれています。

### `ReactNode`とは

Reactで表示できるすべてのものを表す型です。

**具体例:**
- 文字列: `"クリック"`
- 数値: `123`
- JSX要素: `<span>アイコン</span>`
- 配列: `[<Icon />, "送信"]`
- null, undefined(画面には表示されない)

**要するに、ボタンの中に入れられるものすべて**を表す型です。

### なぜこの2つをインポートするのか

- `ButtonHTMLAttributes`: 通常のボタンが持つ機能を全部使えるようにする
- `ReactNode`: ボタンの中身(`children`)に何でも入れられるようにする

## 3-7行目: ButtonPropsとは何か
```typescript
interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  children: ReactNode
  variant?: 'primary' | 'secondary' | 'danger'
  isLoading?: boolean
}
```

### `ButtonProps`の役割

`ButtonProps`は、**このボタンコンポーネントが受け取れるデータの設計図**です。

まだピンと来ないと思うので、例で説明します。

### 設計図がない場合
```typescript
// どんなデータを渡せばいいか分からない
<Button ???>
```

### 設計図がある場合
```typescript
// ButtonPropsという設計図があるので、何を渡せばいいか分かる
<Button variant="primary" isLoading={false}>
  送信
</Button>
```

`ButtonProps`が「このボタンには`variant`と`isLoading`が渡せます」と教えてくれます。

### 3行目: `interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement>`

#### `interface ButtonProps`

設計図の名前です。「ButtonPropsという名前の設計図を作る」という宣言です。

#### `extends ButtonHTMLAttributes<HTMLButtonElement>`

`extends`は「継承する」という意味です。**既にある設計図を引き継ぐ**ことです。

**図で表すと:**
```
ButtonHTMLAttributes (既存の設計図)
├─ onClick
├─ disabled
├─ className
└─ その他100以上の属性

↓ extends (引き継ぐ)

ButtonProps (新しい設計図)
├─ onClick          ← ButtonHTMLAttributesから
├─ disabled         ← ButtonHTMLAttributesから
├─ className        ← ButtonHTMLAttributesから
├─ children         ← 新しく追加
├─ variant          ← 新しく追加
└─ isLoading        ← 新しく追加
```

**つまり:**
- 既存のボタンの機能(onClick, disabledなど)は全部使える
- さらに独自の機能(variant, isLoadingなど)も追加する

**具体例:**
```typescript
// ButtonHTMLAttributesから使える機能
<Button onClick={() => alert('clicked')}>送信</Button>
<Button disabled={true}>送信</Button>
<Button className="mt-4">送信</Button>

// ButtonPropsで新しく追加した機能
<Button variant="primary">送信</Button>
<Button isLoading={true}>送信</Button>

// 全部組み合わせられる
<Button 
  variant="danger" 
  isLoading={false}
  onClick={() => alert('削除します')}
  className="w-full"
>
  削除
</Button>
```

### 4行目: `children: ReactNode`

ボタンの中身を受け取る型定義です。
```typescript
<Button>送信</Button>  
// children = "送信" (文字列)

<Button><Icon />送信</Button>  
// children = [<Icon />, "送信"] (配列)
```

`ReactNode`型なので、文字列でもJSXでも何でも入ります。

### 5行目: `variant?: 'primary' | 'secondary' | 'danger'`

#### `variant`の意味

variantは「変化形」「バリエーション」という意味です。

このボタンコンポーネントでは、**ボタンの見た目のバリエーション**を指定します。

- `primary`: メインのボタン(青色)
- `secondary`: サブのボタン(グレー)
- `danger`: 危険な操作のボタン(赤色)

#### `?`の意味

省略可能という意味です。`variant`を渡さなくてもエラーになりません。
```typescript
<Button>送信</Button>  // variantなし → OK
<Button variant="primary">送信</Button>  // variantあり → OK
```

#### `'primary' | 'secondary' | 'danger'`の意味

この3つの文字列**だけ**を受け付けます。
```typescript
<Button variant="primary">   // OK
<Button variant="secondary"> // OK
<Button variant="danger">    // OK
<Button variant="warning">   // エラー! 'warning'は設計図にない
<Button variant="成功">      // エラー! 日本語は設計図にない
```

#### なぜこう定義するのか

**ボタンの見た目を3種類に限定するため**です。

もし何でも受け付けると:
```typescript
variant?: string  // 何でもOK

<Button variant="あいうえお">  // 変なデータが入る
```

こうなると、対応する色が定義されていないのでエラーになります。

だから、**使える値を限定する**必要があります。

### 6行目: `isLoading?: boolean`

ローディング中かどうかのフラグです。

- `true`: ローディング中
- `false`: 通常

`?`があるので省略可能です。
```typescript
<Button isLoading={true}>   // ローディング中
<Button isLoading={false}>  // 通常
<Button>                    // 省略(デフォルトでfalseになる)
```

## ButtonPropsとfunction Buttonの違い

ここまで読んで、こう思ったはずです。

「`ButtonProps`と`function Button`は何が違うの?」

### ButtonPropsの役割

**設計図**です。「どんなデータを受け取れるか」を定義します。
```typescript
interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  children: ReactNode
  variant?: 'primary' | 'secondary' | 'danger'
  isLoading?: boolean
}
```

これは「受け取れるデータの種類」を書いただけです。**実際の処理は何も書いていません。**

### function Buttonの役割

**実際の処理**を書く場所です。受け取ったデータを使って、ボタンを作ります。
```typescript
export function Button({ children, variant, ... }: ButtonProps) {
  // 受け取ったデータを使って、実際にボタンを作る処理
  return <button>...</button>
}
```

### 図で表すと
```
ButtonProps (設計図)
↓
「variantは'primary' | 'secondary' | 'danger'だけ受け取れます」
「isLoadingはboolean型です」

↓ 設計図を使って

function Button (実際の処理)
↓
「受け取ったvariantに応じて、色を変えます」
「isLoadingがtrueなら、ボタンを無効化します」
```

### 例え話

**ButtonProps = 料理のレシピ**
- 材料: 鶏肉、玉ねぎ、カレールー
- 分量: 鶏肉300g、玉ねぎ1個

**function Button = 実際の調理**
- 材料を切る
- 炒める
- 煮込む
- 完成

レシピがあっても、調理しないと料理はできません。
同様に、`ButtonProps`があっても、`function Button`がないとボタンは作れません。

## 9-16行目: 関数定義とpropsの受け取り
```typescript
export function Button({
  children,
  variant = 'primary',
  isLoading = false,
  className = '',
  disabled,
  ...props
}: ButtonProps) {
```

### 9行目: `export function Button(`

`Button`という名前の関数(コンポーネント)を作り、外部から使えるようにします。

### 10-16行目: データの受け取り方
```typescript
{
  children,
  variant = 'primary',
  isLoading = false,
  className = '',
  disabled,
  ...props
}: ButtonProps
```

#### 末尾の`: ButtonProps`

「この関数は`ButtonProps`という設計図に従ってデータを受け取る」という宣言です。

つまり、`ButtonProps`で定義した型以外は受け取れません。
```typescript
// OK: ButtonPropsに定義されている
<Button variant="primary">送信</Button>

// エラー: ButtonPropsに定義されていない
<Button color="red">送信</Button>
```

#### `{ children, variant, ... }`の意味(分割代入)

**propsオブジェクトから、必要な値を取り出す**書き方です。

**分割代入を使わない場合:**
```typescript
function Button(props: ButtonProps) {
  const children = props.children
  const variant = props.variant
  const isLoading = props.isLoading
  // 毎回props.○○と書く必要がある
}
```

**分割代入を使う場合:**
```typescript
function Button({ children, variant, isLoading }: ButtonProps) {
  // そのまま使える
  console.log(children)
  console.log(variant)
}
```

#### デフォルト値の設定

`variant = 'primary'`は、`variant`が渡されなかった時の初期値です。
```typescript
<Button>送信</Button>
// variantを渡していない
// → 自動的に variant = 'primary' になる

<Button variant="danger">削除</Button>
// variantを渡している
// → variant = 'danger' が使われる
```

同様に:
- `isLoading = false`: 省略時はfalse
- `className = ''`: 省略時は空文字

#### `disabled`にデフォルト値がない理由

`disabled`は元々のHTML仕様で省略可能だからです。
```html
<!-- disabled属性がない = 有効 -->
<button>クリック</button>

<!-- disabled属性がある = 無効 -->
<button disabled>クリック</button>
```

だから、デフォルト値を設定する必要がありません。

#### `...props`の意味(スプレッド構文)

**残りのpropsをすべて受け取る**という意味です。

**具体例:**
```typescript
<Button
  variant="primary"
  onClick={() => alert('click')}
  id="submit"
  aria-label="送信ボタン"
>
  送信
</Button>
```

この場合、データは以下のように分かれます:
```typescript
// 個別に受け取るもの
variant = "primary"

// ...propsに入るもの
props = {
  onClick: () => alert('click'),
  id: "submit",
  "aria-label": "送信ボタン"
}
```

**なぜこうするのか?**

すべてのpropsを個別に書くと大変だからです。
```typescript
// これは大変
function Button({
  children,
  variant,
  isLoading,
  className,
  disabled,
  onClick,
  onMouseEnter,
  onMouseLeave,
  id,
  style,
  // ... 100個以上書く必要がある
}: ButtonProps) {
```

`...props`を使えば、残りは一括で受け取れます。

## 17行目: 基本スタイルの定義
```typescript
const baseClasses = 'px-4 py-2 rounded-lg font-medium transition-colors disabled:opacity-50 disabled:cursor-not-allowed'
```

すべてのボタンに共通するCSSクラス(見た目の設定)です。

- `px-4 py-2`: 内側の余白(左右4、上下2)
- `rounded-lg`: 角を丸くする
- `font-medium`: 文字の太さを中間に
- `transition-colors`: 色が変わる時にアニメーション
- `disabled:opacity-50`: 無効化時に半透明にする
- `disabled:cursor-not-allowed`: 無効化時にカーソルを禁止マークに

これは`variant`に関係なく、すべてのボタンに適用されます。

## 19-23行目: バリエーション別スタイル
```typescript
const variantClasses = {
  primary: 'bg-blue-500 text-white hover:bg-blue-600',
  secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300',
  danger: 'bg-red-500 text-white hover:bg-red-600',
}
```

### これは何をしているのか

**渡された`variant`によって、ボタンの色を変える**設定です。

- `primary`: 青いボタン
- `secondary`: グレーのボタン
- `danger`: 赤いボタン

### オブジェクト形式で定義する理由
```typescript
variantClasses[variant]
```

で、動的に取り出せるからです。

**動作例:**
```typescript
variant = 'primary'
variantClasses[variant]  
// → 'bg-blue-500 text-white hover:bg-blue-600'

variant = 'danger'
variantClasses[variant]  
// → 'bg-red-500 text-white hover:bg-red-600'
```

### 各クラスの意味

**primary(青いボタン):**
- `bg-blue-500`: 背景色を青に
- `text-white`: 文字色を白に
- `hover:bg-blue-600`: マウスを乗せたら濃い青に

**secondary(グレーのボタン):**
- `bg-gray-200`: 背景色を薄いグレーに
- `text-gray-800`: 文字色を濃いグレーに
- `hover:bg-gray-300`: マウスを乗せたら少し濃いグレーに

**danger(赤いボタン):**
- `bg-red-500`: 背景色を赤に
- `text-white`: 文字色を白に
- `hover:bg-red-600`: マウスを乗せたら濃い赤に

### まとめると

このコードは、**propsで渡された`variant`の値に応じて、ボタンの見た目を変えている**ということです。
```typescript
<Button variant="primary">送信</Button>   // 青いボタン
<Button variant="secondary">キャンセル</Button>  // グレーのボタン
<Button variant="danger">削除</Button>     // 赤いボタン
```

## 25-33行目: JSXの返却
```typescript
return (
  <button
    className={`${baseClasses} ${variantClasses[variant]} ${className}`}
    disabled={disabled || isLoading}
    {...props}
  >
    {isLoading ? '読み込み中...' : children}
  </button>
)
```

### 27行目: classNameの結合
```typescript
className={`${baseClasses} ${variantClasses[variant]} ${className}`}
```

#### テンプレートリテラル(バッククォート)

`` `文字列 ${変数} 文字列` ``の形式です。変数を文字列の中に埋め込めます。

**展開例:**
```typescript
variant = 'primary'
className = 'mt-4'

// 結果
"px-4 py-2 rounded-lg ... bg-blue-500 text-white hover:bg-blue-600 mt-4"
```

3つのクラスが結合されます:
1. `baseClasses`: 共通スタイル
2. `variantClasses[variant]`: バリエーション別スタイル
3. `className`: 外部から渡されたカスタムスタイル

### 28行目: disabledの条件
```typescript
disabled={disabled || isLoading}
```

#### `||`の意味

「または」という意味です。

- `disabled`がtrue **または** `isLoading`がtrue → ボタンを無効化
- 両方false → ボタンは有効

**具体例:**
```typescript
disabled = true, isLoading = false
→ true || false = true (無効化)

disabled = false, isLoading = true
→ false || true = true (無効化)

disabled = false, isLoading = false
→ false || false = false (有効)

disabled = true, isLoading = true
→ true || true = true (無効化)
```

**つまり、どちらか一方でもtrueなら無効化する**という意味です。

### 29行目: propsの展開
```typescript
{...props}
```

残りのpropsをすべてボタンに渡します。

**展開例:**
```typescript
props = {
  onClick: () => alert('click'),
  id: "submit",
  type: "button"
}

// 展開後
<button
  onClick={() => alert('click')}
  id="submit"
  type="button"
>
```

### 31行目: 条件分岐
```typescript
{isLoading ? '読み込み中...' : children}
```

#### 三項演算子

`条件 ? 真の時 : 偽の時`の形式です。

- `isLoading`がtrue → 「読み込み中...」と表示
- `isLoading`がfalse → `children`を表示

**具体例:**
```typescript
// ローディング中
<Button isLoading={true}>送信</Button>
// 表示: 「読み込み中...」

// 通常時
<Button isLoading={false}>送信</Button>
// 表示: 「送信」
```

## 全体の流れ

### 1. ボタンを呼び出す
```typescript
<Button variant="danger" isLoading={false} onClick={() => {}}>
  削除
</Button>
```

### 2. propsが渡される
```typescript
children = "削除"
variant = "danger"
isLoading = false
props = { onClick: () => {} }
```

### 3. スタイルが組み立てられる
```typescript
baseClasses = "px-4 py-2 ..."
variantClasses[variant] = "bg-red-500 text-white hover:bg-red-600"
className = ""

// 結合
className = "px-4 py-2 ... bg-red-500 text-white hover:bg-red-600"
```

### 4. disabledが判定される
```typescript
disabled = false || false = false
// ボタンは有効
```

### 5. 表示内容が決まる
```typescript
isLoading = false
// childrenを表示 → "削除"
```

### 6. 最終的なHTML
```html
<button
  class="px-4 py-2 rounded-lg ... bg-red-500 text-white hover:bg-red-600"
  onClick={() => {}}
>
  削除
</button>
```

## このコンポーネントの使い方

### 基本的な使い方
```typescript
<Button>送信</Button>
```

結果:
- 青いボタン(variant="primary"がデフォルト)
- テキストは「送信」
- クリック可能

### バリエーションを変える
```typescript
<Button variant="secondary">キャンセル</Button>
<Button variant="danger">削除</Button>
```

### ローディング状態
```typescript
<Button isLoading={true}>送信中</Button>
```

結果:
- ボタンは無効化される
- 表示は「読み込み中...」になる
- 元の「送信中」というテキストは表示されない

### カスタムスタイルを追加
```typescript
<Button className="w-full mt-4">送信</Button>
```

結果:
- 基本スタイル + バリエーションスタイル + `w-full mt-4`
- 幅100%、上部に余白

### HTML属性を渡す
```typescript
<Button
  onClick={() => alert('clicked')}
  type="submit"
  id="submit-btn"
>
  送信
</Button>
```

すべて`{...props}`によってボタンに渡されます。

## よくある疑問

### 疑問1: なぜ`extends`を使うのか

`ButtonHTMLAttributes`には100以上の属性があります。これを手動で全部書くのは不可能です。

`extends`を使えば、自動的にすべて使えるようになります。

### 疑問2: `...props`は何のためか

予想できないpropsを受け取るためです。
```typescript
<Button data-testid="submit-button">
```

`data-testid`は事前に型定義していませんが、`...props`で受け取れます。

### 疑問3: なぜ`variant`にデフォルト値が必要か

省略時の動作を明確にするためです。

デフォルト値がないと:
```typescript
variant: 'primary' | 'secondary' | 'danger' | undefined
```

undefinedの時の処理が必要になります。

デフォルト値があれば:
```typescript
variant = 'primary'  // 必ず値が入っている
```

## まとめ

このボタンコンポーネントは、以下の技術を組み合わせています。

1. **型の継承**: `extends ButtonHTMLAttributes`で既存の型を引き継ぐ
2. **デフォルト値**: `variant = 'primary'`で省略時の値を設定
3. **スプレッド構文**: `{...props}`で残りのpropsを一括受け取り
4. **条件分岐**: `isLoading ? '読み込み中...' : children`で表示を切り替え
5. **動的スタイル**: `variantClasses[variant]`でpropsに応じてスタイルを変更

これらを理解すれば、実務レベルのコンポーネントが書けるようになります。

次にコンポーネントを作る時は、このパターンを思い出してください。

---

## 参考資料

- [React公式 - コンポーネントへのpropsの渡し方](https://ja.react.dev/learn/passing-props-to-a-component)
- [TypeScript公式 - Interface継承](https://www.typescriptlang.org/docs/handbook/2/objects.html#extending-types)
- [MDN - スプレッド構文](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Spread_syntax)
- [Tailwind CSS - ユーティリティクラス](https://tailwindcss.com/docs/utility-first)
- [TypeScript Deep Dive - 型の基礎](https://typescript-jp.gitbook.io/deep-dive/type-system)