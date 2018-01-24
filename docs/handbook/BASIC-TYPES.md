# 基本的な型

## はじめに

プログラムの有用性を保つには、最も簡単なデータ単位（数値、文字列、構造体、真偽値など）を扱えるようにする必要があります。
TypeScriptでは、JavaScriptで期待されるのとほぼ同じ型をサポートしています。

## Boolean 型 (真偽値)

最も基本的なデータ型は、JavaScriptとTypeScriptが `boolean` 値として扱える単純な真偽値です。

```typescript
let isDone: boolean = false;
```

## Number 型 (数値)

JavaScriptのように、TypeScriptの数値はすべて浮動小数点値です。
これらの浮動小数点数は `number` 型を取得します。
TypeScriptは、16進リテラルおよび10進リテラルに加えて、ECMAScript 2015で導入されたバイナリおよび8進リテラルもサポートしています。

```typescript
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;
```

## String 型 (文字列)

ウェブページやサーバー用のJavaScriptでプログラムを作成するもう1つの基本的な部分は、テキストデータで動作しています。
他の言語と同様に、これらのテキストデータ型を参照するために `string` 型を使用します。
JavaScriptのように、TypeScriptは二重引用符（`"`）または一重引用符（`'`）も使用して文字列データを囲みます。

```typescript
let color: string = "blue";
color = 'red';
```

テンプレート文字列を使用することもできます。
テンプレート文字列は複数の行にまたがる事もでき、式が埋め込む事もできます。
これらの文字列は backtick/backquot (``) 文字で囲まれ、埋め込み式は `${ exper }` という記法です。

```typescript
let fullName: string = `Bob Bobbington`;
let age: number = 37;
let sentence: string = `Hello, my name is ${ fullName }.

I'll be ${ age + 1 } years old next month.`;
```

これは次のsentenceように宣言するのと同じです：

```typescript
let sentence: string = "Hello, my name is " + fullName + ".\n\n" +
    "I'll be " + (age + 1) + " years old next month.";
```

## Array 型 (配列)

TypeScriptは、JavaScriptと同様、値の配列を扱うことができます。
配列の型は、次の2つの方法のいずれかで記述できます。
最初に、要素の型の後ろに `[]` を使用して、その要素型の配列を示します。

```typescript
let list: number[] = [1, 2, 3];
```

2番目の方法では、汎用配列型を使用します `Array<elemType>`。

```typescript
let list: Array<number> = [1, 2, 3];
```

## Tuple 型 (混合配列)

タプル型では、固定数の要素の型がわかっている配列を表現できますが、同じである必要はありません。
たとえば、`string` と `number` のペアとして値を表現することができます：

```typescript
// Declare a tuple type
let x: [string, number];
// Initialize it
x = ["hello", 10]; // OK
// Initialize it incorrectly
x = [10, "hello"]; // Error
```

既知のインデックスを持つ要素にアクセスすると、正しい型が取得されます。

```typescript
console.log(x[0].substr(1)); // OK
console.log(x[1].substr(1)); // Error, 'number' does not have 'substr'
```

既知のインデックスのセットの外側にある要素にアクセスする場合、代わりにユニオン型が使用されます。

```typescript
x[3] = "world"; // OK, 'string' can be assigned to 'string | number'

console.log(x[5].toString()); // OK, 'string' and 'number' both have 'toString'

x[6] = true; // Error, 'boolean' isn't 'string | number'
```

ユニオン型は、後の章で説明する高度なトピックです。

## Enum 型 (列挙)

JavaScriptからのデータ型の標準セットに役立つ追加情報は、`enum` です。
C＃などの言語と同様に、列挙型は数値の集合に親しみやすい名前を付ける方法です。

```typescript
enum Color {Red, Green, Blue}
let c: Color = Color.Green;
```

デフォルトでは、enumsは、`0` で始まるメンバーの番号付けを開始します。
メンバーの1つの値を手動で設定することでこれを変更できます。
例えば、前の例の `0` の代わりに、`1` から開始させる事ができます。

```typescript
enum Color {Red = 1, Green, Blue}
let c: Color = Color.Green;
```

または、enumのすべての値を手動で設定することもできます。

```typescript
enum Color {Red = 1, Green = 2, Blue = 4}
let c: Color = Color.Green;
```

列挙型の便利な機能は、数値から列挙型の値の名前に移動できることです。
たとえば、値 `2` があったものの、上記の `Color` enumに何がマッピングされているのかわからない場合は、対応する名前を検索できます。

```typescript
enum Color {Red = 1, Green, Blue}
let colorName: string = Color[2];

alert(colorName); // Displays 'Green' as it's value is 2 above
```

## Any 型(何か)

アプリケーションを書くときにわからない変数のタイプを記述する必要があるかもしれません。
これらの値は、動的コンテンツ（例えば、ユーザまたはサードパーティ製のライブラリ）からのものであってもよい。
このような場合には、型チェックをオプトアウトし、値をコンパイル時のチェックに渡す必要があります。
これを行うには、次のような `any` タイプのラベルを付けます。

```typescript
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean
```

any型は、既存のJavaScriptを操作する強力な方法です。
コンパイル時に、タイプチェックのオプトインとオプトアウトが可能です。
他の言語と同様に、`Object` が同様の役割を果たすことが期待されるかもしれません。
しかし、`Object` 型の変数は、それらに任意の値を割り当てることしかできません。
実際に存在するものであっても、それらに対して任意のメソッドを呼び出すことはできません。

```typescript
let notSure: any = 4;
notSure.ifItExists(); // okay, ifItExists might exist at runtime
notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)

let prettySure: Object = 4;
prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'.
```

`any` 型は、型の一部を知っていると便利ですが、すべてではないかもしれません。
たとえば、配列がありますが、配列にはさまざまな型が混在しています。

```typescript
let list: any[] = [1, true, "free"];

list[1] = 100;
```

## Void 型 (無)

`void` は、`any` 型のものも全く存在しないことの反対のようなものです。
一般的には、これを値を返さない関数の戻り値の型として見ることができます：

```typescript
function warnUser(): void {
    alert("This is my warning message");
}
```

`void` 型の変数を宣言することは、`undefined` または `null` のみを割り当てることができるため、有用ではありません。

```typescript
let unusable: void = undefined;
```

## Null and Undefined

TypeScriptでは、`undefined` と `null` の両方に、それぞれ実際には `undefined` と `null` という独自の型があります。
`void` のように、彼らはまったく便利ではありません：

```typescript
// Not much else we can assign to these variables!
let u: undefined = undefined;
let n: null = null;
```

デフォルトでは、`null` と `undefined` は他のすべての型のサブタイプです。
これは、あなたが `null` のようなものに `number` のようなものを割り当てることができることを意味します。

ただし、`--strictNullChecks` フラグを使用する場合、`null` と `undefined` は `void` とそれぞれの型にのみ割り当てられます。
これにより、多くの一般的なエラーを回避できます。
`string` または `null` または `undefined` のいずれかを渡す場合は、ユニオンタイプの `string | null | undefined` を使うことができます。
もう一度、後でより多くのユニオンタイプについて学ぶ。

> 注意：可能であれば、`--strictNullChecks` の使用を推奨しますが、このハンドブックでは、電源がオフになっていると見なします。

## Never 型

`never` 型は決して出現しない値の型を表す。
たとえば、`never` は、関数式の戻り値の型、または常に例外をスローするアロー関数式、または決して返されない関数式です。
変数はまた、決して真実ではない型ガードによって絞り込まれるとき、`never` 型を獲得する。

`never` 型は、すべてのタイプのサブタイプであり、割り当て可能です。
ただし、`never` のサブタイプ（`never` 自体は除く）はありません。
誰でも `never` に割り当てることはできません。

`never` を返す関数のいくつかの例：

```typescript
// Function returning never must have unreachable end point
function error(message: string): never {
    throw new Error(message);
}

// Inferred return type is never
function fail() {
    return error("Something failed");
}

// Function returning never must have unreachable end point
function infiniteLoop(): never {
    while (true) {
    }
}
```

## Type assertions (型アサーション)

時々、TypeScriptよりも価値があることはわかってくる状況が訪れるでしょう。
通常これは、あるエンティティのタイプが現在のタイプよりも、具体的である可能性があることが分かったときに発生します。

型アサーションは、コンパイラに "信頼してください、何を為すべきかは理解していいます。"と言う方法です。
型アサーションは他の言語の型キャストに似ていますが、特別なチェックやデータの再構築は実行されません。
実行時の影響はなく、純粋にコンパイラによって使用されます。
TypeScriptは、プログラマが必要とする特別なチェックを実行したことを前提としています。

型アサーションには2つの形式があります。 1つは「角かっこ」の構文です：

```typescript
let someValue: any = "this is a string";

let strLength: number = (<string>someValue).length;
```

もう1つは、`as` 構文です：

```typescript
let someValue: any = "this is a string";

let strLength: number = (someValue as string).length;
```

2つのサンプルは同等です。 もう一方を使用することは、主に好みの選択です。
ただし、JSXでTypeScriptを使用する場合、`as` スタイルのアサーションのみが許可されます。

## A note about `let` (`let` についての注意)

これまでのところ、JavaScriptの `var` キーワードの代わりに `let` キーワードを使用していたことにお気づきでしょう。
`let` キーワードは、実際にはTypeScriptが利用できる新しいJavaScript構文です。
詳細については後で説明しますが、`let` の使用によってJavaScriptの多くの一般的な問題が緩和されるため、可能な限り `var` の代わりに使用する必要があります。
