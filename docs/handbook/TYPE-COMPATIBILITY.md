# TYPE-COMPATIBILITY 型の互換性

## Introduction はじめに

TypeScriptの型互換性は、構造的なサブタイプに基づいています。
構造型は、メンバだけに基づいて型を関連付ける方法です。
これは公称タイピングとは対照的です。 
次のコードを考えてみましょう。

```typescript
interface Named {
    name: string;
}

class Person {
    name: string;
}

let p: Named;
// OK, because of structural typing
p = new Person();
```

C＃やJavaのような公称型言語では、 `Person` クラスが自身を `Named` インタフェースの実装者として明示的に記述していないため、同等のコードがエラーになります。

TypeScriptの構造型システムは、通常JavaScriptコードの記述方法に基づいて設計されています。
JavaScriptでは関数式やオブジェクトリテラルなどの匿名オブジェクトが広く使用されているため、JavaScriptライブラリにある種類の関係を名目上の構造型システムではなく構造型システムで表現する方が自然です。


### A Note on Soundness 安全性に関する注意

TypeScriptの型システムは、コンパイル時には知り得ない操作を安全にします。
型システムがこのプロパティを持つとき、それは 「安全」ではないと言われます。
TypeScriptが不安定な動作を可能にする場所は慎重に検討されました。
このドキュメントでは、これらの発生場所とその背後にある動機付けのシナリオについて説明します。


## Starting out 始めてみましょう

TypeScriptの構造型システムの基本規則は、 `y` が少なくとも `x` と同じメンバを持つ場合、 `x` は `y` と互換性があるということです。
例えば…

```typescript
interface Named {
    name: string;
}

let x: Named;
// y's inferred type is { name: string; location: string; }
let y = { name: "Alice", location: "Seattle" };
x = y;
```

`y` を `x` に割り当てることができるかどうかをチェックするために、コンパイラは `x` の各プロパティをチェックして、対応する互換性のあるプロパティを見つけます。
この場合、 `y` には `name` という文字列のメンバが必要です。
それで、割り当てが許可されます。

関数呼び出しの引数をチェックするときは、同じ割り当て規則が使用されます。

```typescript
function greet(n: Named) {
    alert("Hello, " + n.name);
}
greet(y); // OK
```

`y` には追加の `location` プロパティがありますが、これはエラーを発行しません。
互換性をチェックするときは、ターゲットタイプのメンバ（この場合は `Named` ）のみが考慮されます。

この比較プロセスは再帰的に進行し、各メンバーとサブメンバーのタイプを調べます。


## Comparing two functions 2つの関数の比較

プリミティブ型とオブジェクト型を比較するのは比較的簡単ですが、どのような種類の関数を互換性があるとみなすべきかという問題はもう少し複雑です。
パラメータリストだけが異なる2つの関数の基本的な例から始めましょう。

```typescript
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // OK
x = y; // Error
```

`x` が `y` に代入可能かどうかを調べるには、まずパラメータリストを調べます。
`x` の各パラメータには、互換性のある型の `y` の対応するパラメータが必要です。
パラメータの名前は考慮されず、その型だけが考慮されることに注意してください。
この場合、 `x` のすべてのパラメータに対応する互換性のあるパラメータがyにあるため、代入が許可されます。

2番目の代入はエラーです。なぜなら、 `x` には `y` に必要な2番目のパラメータがないため、代入が許可されていないからです。

`y = x` の例のように、なぜ「廃棄」パラメータを許可するのか疑問に思うかもしれません。
この割り当てが許可される理由は、余分な関数のパラメータを無視することは、実際にはJavaScriptでかなり一般的です。
たとえば、 `Array＃forEach` は、配列要素、そのインデックス、および配列を含む3つのパラメータをコールバック関数に提供します。
それにもかかわらず、最初のパラメータだけを使用するコールバックを提供することは非常に便利です。

```typescript
let items = [1, 2, 3];

// Don't force these extra parameters
items.forEach((item, index, array) => console.log(item));

// Should be OK!
items.forEach(item => console.log(item));
```

次に、戻り値の型だけが異なる2つの関数を使用して、戻り値の型がどのように扱われるかを見てみましょう。

```typescript
let x = () => ({name: "Alice"});
let y = () => ({name: "Alice", location: "Seattle"});

x = y; // OK
y = x; // Error because x() lacks a location property
```

型システムは、ソース関数の戻り値の型が、対象型の戻り値の型のサブタイプであることを強制します。


### Function Parameter Bivariance 関数パラメータ二項分布

関数パラメーターの型を比較するとき、ソースパラメーターがターゲットパラメーターに割り当てられるか、またはその逆の場合に割り当てが成功します。
これは、呼び出し元がより特殊化された型を取るが、それほど特殊化されていない型を持つ関数を呼び出すことになる可能性があるため、不正です。
実際には、この種のエラーはまれであり、これにより多くの一般的なJavaScriptパターンが可能になります。
簡単な例…

```typescript
enum EventType { Mouse, Keyboard }

interface Event { timestamp: number; }
interface MouseEvent extends Event { x: number; y: number }
interface KeyEvent extends Event { keyCode: number }

function listenEvent(eventType: EventType, handler: (n: Event) => void) {
    /* ... */
}

// Unsound, but useful and common
listenEvent(EventType.Mouse, (e: MouseEvent) => console.log(e.x + "," + e.y));

// Undesirable alternatives in presence of soundness
listenEvent(EventType.Mouse, (e: Event) => console.log((<MouseEvent>e).x + "," + (<MouseEvent>e).y));
listenEvent(EventType.Mouse, <(e: Event) => void>((e: MouseEvent) => console.log(e.x + "," + e.y)));

// Still disallowed (clear error). Type safety enforced for wholly incompatible types
listenEvent(EventType.Mouse, (e: number) => console.log(e));
```


### Optional Parameters and Rest Parameters オプションのパラメータと残りのパラメータ

互換性のために関数を比較する場合、オプションのパラメータと必須のパラメータは同じ意味を持ちます。
ソース・タイプの余分なオプション・パラメーターはエラーではなく、ソース・タイプの対応するパラメーターのないターゲット・タイプのオプション・パラメーターはエラーではありません。

関数が残りのパラメータを持つ場合、それは無限の一連のオプションパラメータであるかのように扱われます。

これは型システムの視点からは分かりませんが、実行時の視点からは、省略可能なパラメータの考え方は一般的に十分に強制されません。なぜなら、その位置の `undefined` を渡すことはほとんどの関数で同等であるからです。

モチベーションの例は、コールバックをとり、予測可能な（プログラマには）知られていない（型システムに対する）数の引数でコールバックする関数の共通パターンです。

```typescript
function invokeLater(args: any[], callback: (...args: any[]) => void) {
    /* ... Invoke callback with 'args' ... */
}

// Unsound - invokeLater "might" provide any number of arguments
invokeLater([1, 2], (x, y) => console.log(x + ", " + y));

// Confusing (x and y are actually required) and undiscoverable
invokeLater([1, 2], (x?, y?) => console.log(x + ", " + y));
```


### Functions with overloads 過負荷のある関数

関数にオーバーロードがある場合、ソース・タイプの各オーバーロードは、ターゲット・タイプの互換シグネチャと一致する必要があります。
これにより、ソース関数と同じ状況でターゲット関数を呼び出すことができます。


## Enums 列挙型

列挙型は数値と互換性があり、数値は列挙型と互換性があります。
異なる列挙型のEnum値は互換性がないとみなされます。
例えば…

```typescript
enum Status { Ready, Waiting };
enum Color { Red, Blue, Green };

let status = Status.Ready;
status = Color.Green; //error
```


## Classes クラス型

クラスは、静的型とインスタンス型の両方を持つ例外を除いて、オブジェクトリテラル型およびインタフェースと同様に機能します。
クラス型の2つのオブジェクトを比較する場合、インスタンスのメンバーだけが比較されます。
静的メンバーとコンストラクタは互換性に影響しません。

```typescript
class Animal {
    feet: number;
    constructor(name: string, numFeet: number) { }
}

class Size {
    feet: number;
    constructor(numFeet: number) { }
}

let a: Animal;
let s: Size;

a = s;  //OK
s = a;  //OK
```


### Private and protected members in classes クラスのプライベートメンバーと保護されたメンバー

クラスのプライベートメンバーと保護されたメンバーは、互換性に影響します。
クラスのインスタンスの互換性がチェックされている場合、ターゲット・タイプにプライベート・メンバーが含まれている場合、ソース・タイプには同じクラスから生成されたプライベート・メンバーも含まれている必要があります。
同様に、保護されたメンバーを持つインスタンスにも同じことが適用されます。
これにより、クラスはそのスーパークラスとの間で互換性がありますが、他の継承階層のクラスとは互換性がありません。


## Generics 汎用型

TypeScriptは構造型システムなので、型パラメータはメンバーの型の一部として消費されたときに結果の型にのみ影響します。
例えば…

```typescript
interface Empty<T> {
}
let x: Empty<number>;
let y: Empty<string>;

x = y;  // okay, y matches structure of x
```

上記の `x` と `y` は、その構造が型引数を区別する方法で使用しないため、互換性があります。
`Empty <T>` にメンバーを追加してこの例を変更すると、これがどのように動作するかが示されます。

```typescript
interface NotEmpty<T> {
    data: T;
}
let x: NotEmpty<number>;
let y: NotEmpty<string>;

x = y;  // error, x and y are not compatible
```

このように、型引数を指定した汎用型は、非汎用型と同様に動作します。

タイプ引数が指定されていない汎用型については、すべての指定されていないタイプ引数の代わりに `any` を指定して互換性をチェックします。
結果の型は互換性がないかのようにチェックされます。

例えば…

```typescript
let identity = function<T>(x: T): T {
    // ...
}

let reverse = function<U>(y: U): U {
    // ...
}

identity = reverse;  // Okay because (x: any)=>any matches (y: any)=>any
```


## Advanced Topics 高度なトピック


### Subtype vs Assignment サブタイプと代入

これまでは、互換性を使用しました。これは言語仕様で定義された用語ではありません。
TypeScriptには、サブタイプと代入という2種類の互換性があります。
これらは、サブタイプとの互換性をルールとの互換性を拡張して、対応する数値で `any` および `enum` との間での代入を許可するという点でのみ異なります。

言語の異なる場所は、状況に応じて2つの互換性メカニズムのいずれかを使用します。
実用的な目的のために、型の互換性は、`implements` および `extends` 節の場合でも割り当ての互換性によって決定されます。
詳細については、[TypeScriptの仕様](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md)を参照してください。
