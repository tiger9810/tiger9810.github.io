# Reactのpropsとchildrenの違いについて

<!--
date = "2025-11-5"
-->

## propsとchildrenは何が違うのか

ReactでTypeScriptを書いていると、必ず出会う2つの言葉。`props`と`children`。

「どっちも同じようなものでしょ?」

違います。この2つを混同すると、コンポーネント設計で確実に詰まります。

この記事では、実際のコードを見ながら、2つの本質的な違いを解説します。読み終わる頃には、どちらを使うべきか瞬時に判断できるようになります。

## まず、propsとは何か

propsは「properties」の略。コンポーネントに渡すデータです。
```typescript
type Props = {
  name: string;
  age: number;
};
```

この型定義は「`name`という文字列と、`age`という数値を受け取る」という意味です。

### コンポーネントの中身
```typescript
const MyComponent: React.FC<Props> = ({ name, age }) => {
  return (
    <h1>
      {name}さん({age}才)
    </h1>
  );
};
```

`name`と`age`を受け取って、画面に表示します。シンプルです。

### 呼び出し方
```typescript
<MyComponent name="田中" age={25} />
```

結果: 「田中さん(25才)」と表示されます。

ここまでは理解できるはずです。問題はここからです。

## childrenは何が違うのか

childrenも、実はpropsの一種です。しかし、特別な性質を持っています。
```typescript
type Props = {
  title: string;
  children: React.ReactNode;
};
```

`children`は型が`React.ReactNode`です。この型は「何でも入る」という意味です。

### コンポーネントの中身
```typescript
const MyComponent: React.FC<Props> = ({ title, children }) => {
  return (
    <>
      <h1>{title}</h1>
      {children}
    </>
  );
};
```

`title`を見出しにして、その下に`children`を表示します。

ここで重要なのは、**コンポーネント側は`children`の中身を知らない**という点です。

### 呼び出し方が全く違う
```typescript
<MyComponent title="お知らせ">
  <p>本日は晴れです。</p>
  <p>気温は25度です。</p>
</MyComponent>
```

注目してください。`children`という名前は、どこにも書いていません。

タグの間に書いた内容が、自動的に`children`として渡されます。

結果:
```
お知らせ
本日は晴れです。
気温は25度です。
```

## 2つの決定的な違い

### 違い1: 渡し方

**props:**
```typescript
<Component name="田中" age={25} />
```
名前を明示して渡します。

**children:**
```typescript
<Component>
  ここに書いた内容が自動的にchildrenになる
</Component>
```
タグの間に書けば、自動的に渡されます。

### 違い2: 型の明確さ

**props:**
```typescript
type Props = {
  name: string;  // 文字列のみ
  age: number;   // 数値のみ
};
```
型が厳密に決まっています。

**children:**
```typescript
children: React.ReactNode;  // 何でもOK
```
テキスト、HTMLタグ、コンポーネント、配列、すべて入ります。

### 違い3: 用途

**props:** コンポーネントの「設定値」を渡す
- 例: 名前、色、サイズ、表示/非表示フラグ

**children:** コンポーネントの「中身」を渡す
- 例: カード内に表示する内容、モーダルの本文、リストの項目

## 実践: どちらを使うべきか

### ケース1: ユーザー情報を表示
```typescript
// propsを使う
<UserProfile name="田中" age={25} job="エンジニア" />
```

理由: 渡すデータが明確で、型も決まっているから。

### ケース2: カードコンポーネント
```typescript
// childrenを使う
<Card title="プロフィール">
  <p>名前: 田中</p>
  <p>年齢: 25才</p>
  <Image src="photo.jpg" />
</Card>
```

理由: カードの中身は毎回変わるから。外側の枠だけを提供し、中身は呼び出し側が決める。

### ケース3: 両方使う
```typescript
<Section title="ニュース" bgColor="blue">
  <Article>記事1</Article>
  <Article>記事2</Article>
</Section>
```

- `title`と`bgColor`はprops(設定値)
- `<Article>`タグはchildren(中身)

この使い分けが、Reactコンポーネント設計の核心です。

## よくある誤解

### 誤解1: 「childrenは省略できる」

できません。`children: React.ReactNode`を型定義したら、必ず何か渡す必要があります。

省略可能にするには:
```typescript
children?: React.ReactNode;
```

### 誤解2: 「childrenは1つだけ」

違います。複数渡せます。
```typescript
<Component>
  <div>要素1</div>
  <div>要素2</div>
  <div>要素3</div>
</Component>
```

すべてがchildrenとして扱われます。

### 誤解3: 「propsの方が優れている」

用途次第です。childrenがないと、再利用可能なコンポーネントは作れません。

## 実務での使い分けルール

### propsを使う場合
- データが明確に決まっている
- 型で制限したい
- 設定値を渡す

### childrenを使う場合
- 中身が毎回変わる
- 柔軟性が必要
- レイアウトや枠だけを提供する

### 両方使う場合
- 設定値と中身の両方が必要
- 実務では最も多いパターン

## まとめ

propsとchildrenの違いは、次の一文に集約されます。

**propsは「名前をつけて渡す設定値」、childrenは「タグの間に書く中身」。**

この違いを理解すれば、Reactのコンポーネント設計で迷うことはなくなります。

次にコンポーネントを作る時、「これはpropsか? childrenか?」と自問してください。その判断が、コードの品質を決めます。

---

## 参考資料

- [React公式ドキュメント - Children を JSX として渡す](https://ja.react.dev/learn/passing-props-to-a-component#passing-jsx-as-children)
- [TypeScript公式 - React.ReactNodeの型定義](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts)
- [コンポーネント設計のベストプラクティス](https://zenn.dev/brachio_takumi/articles/3ba40e80dc35da)