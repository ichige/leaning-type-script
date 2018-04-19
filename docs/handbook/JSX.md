# JSX

## Introduction - 導入

[JSX](https://facebook.github.io/jsx/) は埋め込み可能なXMLのような構文です。 
これは、有効なJavaScriptに変換されることを意味しますが、その変換のセマンティクスは実装固有です。 
JSXは [React](https://reactjs.org/) フレームワークで人気を博しましたが、他のアプリケーションも見てきました。 
TypeScript は埋め込み、型チェック、および JSX を JavaScript に直接コンパイルすることをサポートしています。


## Basic usage - 基本的な使い方

JSX を使用するには、2つのことを行う必要があります。

1. 拡張子 `.tsx` のファイルに名前を付ける
1. `jsx` オプションを有効にする

TypeScript には、保存、反応、ネイティブ反応の3つのJSXモードが用意されています。 
これらのモードはエミットステージにのみ影響します。
タイプのチェックは影響を受けません。 
保存モードは、別の変換ステップ（例えば、Babel）によってさらに消費される出力の一部としてJSXを保持する。 
さらに、出力には `.jsx` ファイル拡張子が付きます。 
反応モードでは `React.createElement` が発行され、使用前に JSX 変換を行う必要はなく、出力には `.js` ファイル拡張子が付きます。 
`react-native` モードはすべての JSX を保持するという点で `preserve` と同じですが、代わりに `.js` ファイル拡張子が付きます。

| Mode | Input | Output | Output File Extension |
| --- | --- | --- | --- |
| `preserve` | `<div />` | `<div />` | `.jsx` |
| `react` | `<div />` | `React.createElement("div")` | `.js` |
| `react-native` | `<div />` | `<div />` | `.js` |

このモードは、 `--jsx` コマンドラインフラグまたは [tsconfig.json](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) ファイルの対応するオプションを使用して指定できます。

> 注：識別子Reactはハードコードされているため、 `React` を大文字のRで使用可能にする必要があります。


## The `as` operator - `as` 演算子

型アサーションを書く方法を思い出してください。

```typescript
var foo = <foo>bar;
```

ここでは変数 `bar` の型が `foo` であることを宣言しています。 
TypeScript では型アサーションに山括弧も使用されるため、JSX の構文では構文解析の難しさが生じます。 
結果として、TypeScript は `.tsx` ファイル内にアングルブラケットタイプのアサーションを許可しません。

`.tsx` ファイルのこの機能の損失を補うために、新しいタイプのアサーション演算子が追加されました。 
上の例は、 `as` 演算子で簡単に書き直すことができます。

```typescript
var foo = bar as foo;
```

`as` 演算子は、 `.ts` ファイルと `.tsx` ファイルの両方で使用でき、動作が他の型のアサーション・スタイルと同じです。


## Type Checking - 型のチェック

JSX で型チェックを理解するためには、まず内在要素と価値に基づく要素の違いを理解しなければなりません。 
JSX式 `<expr />` を指定すると、`expr` は環境固有のもの（DOM環境の `div` や `span` など）または作成したカスタムコンポーネントを参照することがあります。 
これは2つの理由から重要です。

1. Reactでは、組み込み要素は文字列（ `React.createElement("div")` ）として生成されますが、作成したコンポーネントは `React.createElement(MyComponent)` ではありません。
1. JSX 要素で渡される属性のタイプは、別の方法で検索する必要があります。 内在要素の属性は本質的に分かっているべきですが、コンポーネントはおそらく独自の属性セットを指定する必要があります。

TypeScript は、[React がこれらを区別するのと同じ規則](http://facebook.github.io/react/docs/jsx-in-depth.html#html-tags-vs.-react-components)を使用します。 
内在要素は常に小文字で始まり、値に基づく要素は常に大文字で始まります。


### Intrinsic elements - 本質的な要素

固有の要素は `JSX.IntrinsicElements` という特別なインタフェースで参照されます。 
デフォルトでは、このインターフェイスが指定されていない場合、何かが実行され、組み込み要素は型チェックされません。 
ただし、このインタフェースが存在する場合、組み込み要素の名前は `JSX.IntrinsicElements` インタフェースのプロパティとして参照されます。 
例えば…

```typescript
declare namespace JSX {
    interface IntrinsicElements {
        foo: any
    }
}

<foo />; // ok
<bar />; // error
```

上の例では、 `<foo />` は正常に動作しますが、 `<bar />` は `JSX.IntrinsicElements` で指定されていないためエラーになります。

> 注意：次のように、 `JSX.IntrinsicElements` でキャッチオール文字列インデクサを指定することもできます。
>```typescript
>declare namespace JSX {
>   interface IntrinsicElements {
>       [elemName: string]: any;
>   }
>}
>```


### Value-based elements - 値ベースの要素

値ベースの要素は、スコープ内にある識別子によって単純に参照されます。

```typescript
import MyComponent from "./myComponent";

<MyComponent />; // ok
<SomeOtherComponent />; // error
```

値ベースの要素を定義するには、2つの方法があります。

1. ステートレス関数コンポーネント（SFC）
1. クラスコンポーネント

これらの2つのタイプの値ベースの要素は JSX 式で互いに区別できないため、最初にオーバーロード解決を使用してステートレス関数コンポーネントとして式を解決しようとします。 
プロセスが成功すると、式をその宣言に解決します。 
SFCとして解決できない場合は、クラスコンポーネントとして解決しようとします。 
それが失敗した場合は、エラーを報告します。


### Stateless Functional Component - ステートレス関数コンポーネント

名前が示すように、コンポーネントは JavaScript 関数として定義され、最初の引数は `props` オブジェクトです。 
戻り型を `JSX.Element` に割り当てる必要があることを強制します

```typescript
interface FooProp {
  name: string;
  X: number;
  Y: number;
}

declare function AnotherComponent(prop: {name: string});
function ComponentFoo(prop: FooProp) {
  return <AnotherComponent name=prop.name />;
}

const Button = (prop: {value: string}, context: { color: string }) => <button>
```

SFC は単なる JavaScript 関数なので、ここでも関数のオーバーロードを利用できます。

```typescript
interface ClickableProps {
  children: JSX.Element[] | JSX.Element
}

interface HomeProps extends ClickableProps {
  home: JSX.Element;
}

interface SideProps extends ClickableProps {
  side: JSX.Element | string;
}

function MainButton(prop: HomeProps): JSX.Element;
function MainButton(prop: SideProps): JSX.Element {
  ...
}
```


### Class Component - クラスコンポーネント

クラスコンポーネントの型を制限することは可能です。 
しかし、このためには、要素クラス型と要素インスタンス型の2つの新しい用語を導入する必要があります。

`<Expr />` とすると、要素クラス型は `Expr` の型です。 
したがって、上記の例では、 `MyComponent` が ES6 クラスの場合、クラス型はそのクラスになります。 
`MyComponent` がファクトリ関数の場合、クラス型はその関数になります。

クラス型が確立されると、インスタンス型は、クラス型のコール・シグニチャーの戻りタイプと構造シグニチャーの共用体によって決定されます。 
したがって、ES6 クラスの場合、インスタンス型はそのクラスのインスタンスの型になり、ファクトリ関数の場合は、関数から返される値の型になります。

```typescript
class MyComponent {
  render() {}
}

// use a construct signature
var myComponent = new MyComponent();

// element class type => MyComponent
// element instance type => { render: () => void }

function MyFactoryFunction() {
  return {
    render: () => {
    }
  }
}

// use a call signature
var myComponent = MyFactoryFunction();

// element class type => FactoryFunction
// element instance type => { render: () => void }
```

要素インスタンスの型は、 `JSX.ElementClass` に代入可能でなければならないので面白いです。
そうしないと、エラーが発生します。 
デフォルトで `JSX.ElementClass` は `{}` ですが、JSX の使用を適切なインタフェースに準拠する型に限定するように拡張することができます。

```typescript
declare namespace JSX {
  interface ElementClass {
    render: any;
  }
}

class MyComponent {
  render() {}
}
function MyFactoryFunction() {
  return { render: () => {} }
}

<MyComponent />; // ok
<MyFactoryFunction />; // ok

class NotAValidComponent {}
function NotAValidFactoryFunction() {
  return {};
}

<NotAValidComponent />; // error
<NotAValidFactoryFunction />; // error
```


### Attribute type checking - 属性型のチェック

属性の型チェックをする最初のステップは、要素の属性型を決定することです。 
これは、組み込み要素と値ベース要素の間でわずかに異なります。

組み込み要素の場合は、 `JSX.IntrinsicElements` のプロパティの型です。

```typescript
declare namespace JSX {
  interface IntrinsicElements {
    foo: { bar?: boolean }
  }
}

// element attributes type for 'foo' is '{bar?: boolean}'
<foo bar />;
```

値ベースの要素については、もう少し複雑です。 
これは、以前に決定された要素インスタンスタイプのプロパティの型によって決定されます。 
使用するプロパティは `JSX.ElementAttributesProperty` によって決まります。 
それは単一のプロパティで宣言する必要があります。 
そのプロパティの名前が使用されます。

```typescript
declare namespace JSX {
  interface ElementAttributesProperty {
    props; // specify the property name to use
  }
}

class MyComponent {
  // specify the property on the element instance type
  props: {
    foo?: string;
  }
}

// element attributes type for 'MyComponent' is '{foo?: string}'
<MyComponent foo="bar" />
```

要素属性の型は、JSX の属性を型チェックするために使用されます。 
オプションおよび必須プロパティーがサポートされています。


```typescript
declare namespace JSX {
  interface IntrinsicElements {
    foo: { requiredProp: string; optionalProp?: number }
  }
}

<foo requiredProp="bar" />; // ok
<foo requiredProp="bar" optionalProp={0} />; // ok
<foo />; // error, requiredProp is missing
<foo requiredProp={0} />; // error, requiredProp should be a string
<foo requiredProp="bar" unknownProp />; // error, unknownProp does not exist
<foo requiredProp="bar" some-unknown-prop />; // ok, because 'some-unknown-prop' is not a valid identifier
```

> 注意：属性名が有効なJS識別子（ `data-*` 属性など）でない場合、要素属性タイプに属性名が見つからない場合はエラーとみなされません。

スプレッド演算子も動作します。

```typescript
var props = { requiredProp: "bar" };
<foo {...props} />; // ok

var badProps = {};
<foo {...badProps} />; // error
```

### Children Type Checking - 子要素の型チェック

2.3 では、子要素のタイプチェックを紹介します。 
子要素は、属性の型チェックから決定した要素属性型のプロパティです。 
`JSX.ElementAttributesProperty` を使用して小道具の名前を判別する方法と同様に、 `JSX.ElementChildrenAttribute` を使用して子の名前を判別します。 
`JSX.ElementChildrenAttribute` は単一のプロパティで宣言する必要があります。

```typescript
declare namespace JSX {
  interface ElementChildrenAttribute {
    children: {};  // specify children name to use
  }
}
```

子要素の型を明示的に指定することなく、 [React typings](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/react) のデフォルトタイプを使用します。

```typescript
<div>
  <h1>Hello</h1>
</div>;

<div>
  <h1>Hello</h1>
  World
</div>;

const CustomComp = (props) => <div>props.children</div>
<CustomComp>
  <div>Hello World</div>
  {"This is just a JS expression..." + 1000}
</CustomComp>
```

他の属性と同様に、子要素の型を指定することができます。 
これは、 [React typings](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/react) のデフォルトタイプを上書きします。

```typescript
interface PropsType {
  children: JSX.Element
  name: string
}

class Component extends React.Component<PropsType, {}> {
  render() {
    return (
      <h2>
        this.props.children
      </h2>
    )
  }
}

// OK
<Component>
  <h1>Hello World</h1>
</Component>

// Error: children is of type JSX.Element not array of JSX.Element
<Component>
  <h1>Hello World</h1>
  <h2>Hello World</h2>
</Component>

// Error: children is of type JSX.Element not array of JSX.Element or string.
<Component>
  <h1>Hello</h1>
  World
</Component>
```

## The JSX result type - JSX の結果型

デフォルトでは、JSX 式の結果は `any` と入力されます。 `JSX.Element` インタフェースを指定することによって、タイプをカスタマイズできます。 
ただし、このインタフェースから JSX の要素、属性、または子に関する型情報を取得することはできません。 
それはブラックボックスです。


## Embedding Expressions - 式の埋め込み

JSX では、式を中括弧（ `{}` ）で囲んでタグ間に式を埋め込むことができます。

```typescript
var a = <div>
  {["foo", "bar"].map(i => <span>{i / 2}</span>)}
</div>
```

文字列を数値で区切ることはできないので、上記のコードはエラーになります。
`preserve` オプションを使用した場合の出力は次のようになります。

```typescript
var a = <div>
  {["foo", "bar"].map(function (i) { return <span>{i / 2}</span>; })}
</div>
```


## React integration - React の統合

Reac tでJSX を使用するには、 [React typings](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/react) を使用する必要があります。 
これらの型指定は、React で使用するために JSX ネームスペースを適切に定義します。

```typescript
/// <reference path="react.d.ts" />

interface Props {
  foo: string;
}

class MyComponent extends React.Component<Props, {}> {
  render() {
    return <span>{this.props.foo}</span>
  }
}

<MyComponent foo="bar" />; // ok
<MyComponent foo={0} />; // error
```
