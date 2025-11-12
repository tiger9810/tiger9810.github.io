# クリックするたびに数字が増える仕組み - React状態管理の基礎

<!--
date = "2025-11-12"
-->

## ボタンを押すと数字が増える。これを自分で作れますか?

Webサイトでボタンを押すと何かが変わる。例えば、いいねボタンを押すと数字が増える、カートに追加すると商品数が増える。この「変化」はどうやって実現されているのでしょうか。

答えは「状態管理」という仕組みにあります。この記事では、最もシンプルなクリックカウンターを作りながら、Reactの状態管理を理解していきます。

**30分後、あなたは自分でボタンを押すと反応するアプリを作れるようになります。**

## 何を作るのか

画面の中央に数字が表示されています。「+1」ボタンを押すと1ずつ増え、「リセット」ボタンを押すと0に戻ります。
```
表示される数字: 0
[+1ボタン] [リセットボタン]
```

シンプルですが、このカウンターには以下のReactの核心的な技術が含まれています。

- **useState**: 変化する値を管理する
- **onClick**: ユーザーの操作に反応する
- **再レンダリング**: 値が変わると自動的に画面が更新される

これらはTodoリスト、ショッピングカート、SNSのいいね機能など、あらゆるWebアプリケーションの土台です。

---

## プロジェクトを作成する

### ターミナルを開いてコマンドを実行する

まず、プロジェクトの土台を作ります。ターミナル(WindowsならPowerShell、Macならターミナルアプリ)を開いて、以下を入力してください。
```bash
npx create-next-app@latest click-counter
```

**これは何をしているのか?**
- `npx`: Node.jsのパッケージを一時的に実行するコマンド
- `create-next-app`: Next.jsというフレームワークのプロジェクトを作成するツール
- `click-counter`: 作成するプロジェクトの名前

実行すると、必要なファイルが自動的にダウンロードされ、プロジェクトのフォルダ構造が作られます。

### いくつか質問に答える

コマンドを実行すると、設定についての質問が表示されます。以下のように答えてください。
```
✔ Would you like to use TypeScript? › No
✔ Would you like to use ESLint? › Yes  
✔ Would you like to use Tailwind CSS? › Yes
✔ Would you like your code inside a `src/` directory? › No
✔ Would you like to use App Router? › Yes
✔ Would you like to use Turbopack? › No
✔ Would you like to customize the import alias? › No
```

**なぜTypeScriptをNoにするのか?**

TypeScriptは型を厳密にチェックする仕組みで、大規模開発では便利です。しかし、今回は基本を学ぶため、通常のJavaScriptを使います。段階的に学ぶことが重要です。

### プロジェクトフォルダに移動する

作成が完了したら、以下のコマンドでフォルダの中に入ります。
```bash
cd click-counter
```

`cd`は「change directory」の略で、フォルダを移動するコマンドです。

---

## 実際に動かしてみる - 最初の成功体験

### 開発サーバーを起動する

以下のコマンドを実行してください。
```bash
npm run dev
```

**これは何をしているのか?**
- `npm run dev`: 開発用のサーバーを起動する
- コードを変更すると自動的にブラウザの表示が更新される仕組みが動き始める

ターミナルに`Local: http://localhost:3000`のような表示が出れば成功です。

### ブラウザで確認する

ブラウザを開いて、アドレスバーに以下を入力してください。
```
http://localhost:3000
```

**Next.jsのデフォルトページが表示されましたか?**

表示されたなら、あなたは今、自分のパソコンの中でWebサーバーを動かし、そこにブラウザからアクセスしています。これが「ローカル開発環境」です。

`localhost`は「あなたのパソコン自身」を意味し、`3000`は「ポート番号」です。一つのパソコンで複数のサーバーを動かすために、ポート番号で区別します。

---

## コードを書く準備

### VSCodeでプロジェクトを開く

VSCode(Visual Studio Code)というエディタを使います。ターミナルで以下を実行すると開きます。
```bash
code .
```

`.`(ドット)は「現在いるフォルダ」を意味します。つまり、「現在のフォルダをVSCodeで開く」という命令です。

### フォルダの構造を理解する

VSCodeの左側を見ると、以下のような構造が表示されています。
```
click-counter/
├── app/
│   ├── page.js          ← 今回編集するファイル
│   ├── layout.js        ← ページ全体のレイアウト
│   └── globals.css      ← スタイル設定
├── public/              ← 画像などを置く場所
├── package.json         ← プロジェクトの設定ファイル
└── その他...
```

**今回編集するのは`app/page.js`だけです。** このファイルが、ブラウザに表示される内容を決定しています。

Next.jsでは、`app`フォルダの中のファイルが自動的にページとして認識されます。`page.js`という名前のファイルは、そのフォルダのメインページになります。

---

## カウンターを作る - ここが本番

### page.jsを開いて全て削除する

VSCodeで`app/page.js`を開いてください。デフォルトで書かれているコードがありますが、**全て選択して削除**してください。

まっさらな状態から始めることで、各行の意味が明確になります。

### カウンターのコードを書く

削除した後、以下のコードを貼り付けてください。
```javascript
'use client'

import { useState } from 'react'

export default function Home() {
  const [count, setCount] = useState(0)

  const increment = () => {
    setCount(count + 1)
  }

  const reset = () => {
    setCount(0)
  }

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-100">
      <div className="bg-white p-8 rounded-lg shadow-md">
        <h1 className="text-4xl font-bold text-center mb-8">
          カウンター
        </h1>
        
        <div className="text-6xl font-bold text-center mb-8 text-blue-600">
          {count}
        </div>
        
        <div className="flex gap-4">
          <button
            onClick={increment}
            className="bg-blue-500 text-white px-6 py-3 rounded-lg hover:bg-blue-600 transition"
          >
            +1
          </button>
          
          <button
            onClick={reset}
            className="bg-gray-500 text-white px-6 py-3 rounded-lg hover:bg-gray-600 transition"
          >
            リセット
          </button>
        </div>
      </div>
    </div>
  )
}
```

### 保存して確認する

WindowsならCtrl+S、MacならCmd+Sで保存してください。

**ブラウザを見てください。** 自動的にページが更新され、カウンターが表示されているはずです。

**ボタンを押してみてください。** 数字が増えましたか? これがReactの状態管理です。

---

## なぜ動くのか - コードの仕組みを理解する

### 'use client' - ブラウザで実行する宣言
```javascript
'use client'
```

ファイルの一番上にこの行があります。

**これは何のため?**

Next.js 13以降では、デフォルトでコンポーネントは「サーバー」で実行されます。サーバーで実行されるコンポーネントは、最初にHTMLを生成して送信します。しかし、ボタンを押すなどのインタラクティブな機能は、ブラウザ(クライアント)で実行される必要があります。

`'use client'`と書くことで、「このコンポーネントはブラウザで実行してください」とNext.jsに伝えています。

**この行がないとどうなる?**

`useState`などのReactのインタラクティブな機能を使おうとした時点で、エラーが発生します。エラーメッセージには「Client Component」などの言葉が含まれます。

### useState - 変化する値を管理する魔法
```javascript
const [count, setCount] = useState(0)
```

この1行が、Reactの状態管理の核心です。

**分解して理解する:**

- `useState`: Reactが提供する「状態を管理する」ための関数
- `useState(0)`: 初期値を0に設定。最初は`count`が0になる
- `count`: 現在の値を保持する変数
- `setCount`: 値を変更するための専用関数

**なぜsetCountが必要なのか?**

通常のJavaScriptでは、変数の値を変えても画面は更新されません。
```javascript
let count = 0
count = count + 1  // これだけでは画面は変わらない
```

しかし、`setCount`を使うと、値が変わった時にReactが「この値が変わった!画面を更新しなければ!」と気づき、自動的に画面を再描画します。これを**再レンダリング**と呼びます。

### increment - 値を増やす関数
```javascript
const increment = () => {
  setCount(count + 1)
}
```

`increment`は「増やす」という意味です。

**この関数は何をしているのか?**

1. 現在の`count`の値を取得する
2. それに1を足す
3. `setCount`を使って新しい値を設定する
4. Reactが変化を検知し、画面を再レンダリングする

結果として、ボタンを押すたびに画面の数字が1ずつ増えます。

### reset - 値を0に戻す関数
```javascript
const reset = () => {
  setCount(0)
}
```

`reset`は「リセットする」という意味です。

現在の値が何であろうと、`setCount(0)`によって強制的に0に設定されます。

### 画面に値を表示する - 中括弧の役割
```javascript
<div className="text-6xl font-bold text-center mb-8 text-blue-600">
  {count}
</div>
```

HTMLのように見えますが、これは**JSX**と呼ばれるReact独自の記法です。

**中括弧{}の意味:**

JSXの中で中括弧`{}`を使うと、その中にJavaScriptの式を書くことができます。`{count}`と書くことで、現在の`count`の値が画面に表示されます。

`count`の値が変わると、この部分だけが自動的に更新されます。ページ全体を再読み込みする必要はありません。

### onClick - ボタンが押された時の処理
```javascript
<button
  onClick={increment}
  className="bg-blue-500 text-white px-6 py-3 rounded-lg hover:bg-blue-600 transition"
>
  +1
</button>
```

`onClick={increment}`という部分で、「このボタンがクリックされたら`increment`関数を実行する」と指定しています。

**流れを整理すると:**

1. ユーザーがボタンをクリック
2. `onClick`に指定された`increment`関数が実行される
3. `increment`の中で`setCount(count + 1)`が実行される
4. Reactが変化を検知
5. 画面が再レンダリングされる
6. 新しい`count`の値が表示される

この一連の流れが、一瞬のうちに完了します。

---

## 実験して理解を深める

### 実験1: 初期値を変える

`useState(0)`の`0`を`10`に変えてみてください。
```javascript
const [count, setCount] = useState(10)
```

**予想してください:** 何が起こりますか?

ファイルを保存してブラウザを確認すると、最初から10が表示されるはずです。

**なぜ?**

`useState(10)`の`10`は「初期値」です。ページが最初に読み込まれた時、`count`は10からスタートします。

### 実験2: 増加量を変える

`count + 1`を`count + 5`に変えてみてください。
```javascript
const increment = () => {
  setCount(count + 5)
}
```

**予想してください:** ボタンを押すとどうなりますか?

5ずつ増えるようになります。

**重要な気づき:**

`setCount`に渡す値は自由に決められます。`count + 5`でも`count * 2`でも`100`でも構いません。あなたがロジックを決められます。

### 実験3: 減らすボタンを追加する

新しい機能を追加してみましょう。数字を減らすボタンを作ります。

**ステップ1: 減らす関数を作る**

`reset`関数の下に、以下を追加してください。
```javascript
const decrement = () => {
  setCount(count - 1)
}
```

`decrement`は「減らす」という意味です。

**ステップ2: ボタンを追加する**

ボタンのセクションを以下のように変更してください。
```javascript
<div className="flex gap-4">
  <button
    onClick={increment}
    className="bg-blue-500 text-white px-6 py-3 rounded-lg hover:bg-blue-600 transition"
  >
    +1
  </button>
  
  <button
    onClick={decrement}
    className="bg-red-500 text-white px-6 py-3 rounded-lg hover:bg-red-600 transition"
  >
    -1
  </button>
  
  <button
    onClick={reset}
    className="bg-gray-500 text-white px-6 py-3 rounded-lg hover:bg-gray-600 transition"
  >
    リセット
  </button>
</div>
```

保存してブラウザを確認してください。3つのボタンが表示され、それぞれが異なる動作をするはずです。

**あなたは今、自分で新しい機能を追加しました。** これがプログラミングです。

---

## エラーが出たらどうする?

### useState is not defined

このエラーが表示される場合、ファイルの一番上に`'use client'`を書き忘れています。
```javascript
'use client'  // 必ず一番上に

import { useState } from 'react'
```

### ボタンを押しても何も起こらない

**確認すること:**

1. ブラウザのコンソールを開く(F12キーを押す)
2. エラーメッセージが表示されていないか確認
3. `onClick={increment}`のように書いているか確認

**よくある間違い:**
```javascript
onClick={increment()}  // ❌ 括弧をつけると即座に実行される
onClick={increment}    // ✅ 正しい。関数そのものを渡す
```

括弧をつけると、「クリックされた時に実行する」のではなく、「今すぐ実行する」という意味になってしまいます。

### ページが表示されない

ターミナルで`npm run dev`が実行されているか確認してください。

実行中の場合、ターミナルに`Local: http://localhost:3000`のような表示があるはずです。

### スタイルが反映されない

プロジェクト作成時に「Would you like to use Tailwind CSS?」に対して「Yes」を選んだか確認してください。

もしNoを選んでいた場合は、Tailwind CSSがインストールされていないため、`className`に指定したスタイルが適用されません。

---

## このカウンターで何を学んだのか

### 学んだ技術要素

- **useState**: 変化する値を管理する仕組み
- **イベントハンドラ**: ユーザーの操作に反応する方法
- **再レンダリング**: 状態が変わると自動的に画面が更新される仕組み
- **JSX**: HTMLとJavaScriptを組み合わせた記法

### なぜこれが重要なのか

このシンプルなカウンターには、あらゆるWebアプリケーションの基礎が含まれています。

**例えば:**

- SNSの「いいね」ボタン → カウンターと同じ仕組み
- ショッピングカートの商品数 → カウンターと同じ仕組み
- Todoリストのチェック → 状態管理の応用
- フォームの入力 → `useState`で文字列を管理

つまり、**このカウンターを理解すれば、複雑なアプリケーションへの道が開けます。**

---

## 次のステップ

### 推奨: テキスト入力エコーを作る

カウンターができるようになったら、次は「テキスト入力エコー」に進むことをお勧めします。

**何を作るのか:**

入力欄にテキストを入力すると、リアルタイムでその内容が下に表示されるアプリケーションです。

**何が学べるのか:**

- `useState`で文字列を扱う方法
- `onChange`イベントの使い方
- フォーム処理の基礎

カウンターで学んだ知識がそのまま活用できますが、扱うデータの種類が変わります。この違いを理解することで、Reactの状態管理について更に深く理解できます。

### 参考記事: より深く学ぶために

以下の記事で、より詳細な説明や応用例を学べます。

**公式ドキュメント:**
- [React公式 - useState](https://react.dev/reference/react/useState)
  - useStateの詳細な仕様と使い方
- [Next.js公式 - Client Components](https://nextjs.org/docs/app/building-your-application/rendering/client-components)
  - 'use client'の詳細と使い分け

**初心者向けチュートリアル:**
- [React入門 - 状態管理を理解する](https://ja.react.dev/learn/state-a-components-memory)
  - 状態がコンポーネントの「記憶」である理由
- [Next.js App Router入門](https://nextjs.org/docs/app/getting-started/project-structure)
  - プロジェクト構造の全体像

**実践的な例:**
- [React Examples - Counter Variations](https://react.dev/learn/tutorial-tic-tac-toe)
  - 三目並べを作りながら学ぶ、より実践的なチュートリアル

---

## まとめ

ボタンを押すと数字が増える。この単純な動作の裏には、Reactの核心的な仕組みが隠れています。

**重要な3つの概念:**

1. **useState**: 変化する値を管理し、変化を検知する
2. **イベントハンドラ**: ユーザーの操作に応答する
3. **再レンダリング**: 状態の変化を画面に反映する

この3つの概念が理解できれば、あなたは「動的なWebアプリケーション」の土台を手に入れたことになります。

**次は何をしますか?**

カウンターを改造してみるのもいいでしょう。テキスト入力に進むのもいいでしょう。

重要なのは、実際に手を動かして、動くものを作ることです。知識は実践を通じて定着します。