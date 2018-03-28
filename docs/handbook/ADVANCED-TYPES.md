# Advanced Types 高度な型

## Intersection Types 交差型

交差タイプは、複数のタイプを1つに結合します。
これにより、既存のタイプを一緒に追加して、必要なすべての機能を備えた単一のタイプを得ることができます。
たとえば、 `Person & Serializable & Loggable` は `Person` 、 `and Serializable` および `Loggable` です。
つまり、この型のオブジェクトには、3つの型のすべてのメンバーが含まれます。

ミックスインやクラシックオブジェクト指向のモールドには収まらないその他のコンセプトに使用される交差の種類がほとんどです。
（JavaScriptにはたくさんのものがあります！）次に、ミックスインの作成方法を示す簡単な例を示します。

```typescript
function extend<T, U>(first: T, second: U): T & U {
    let result = <T & U>{};
    for (let id in first) {
        (<any>result)[id] = (<any>first)[id];
    }
    for (let id in second) {
        if (!result.hasOwnProperty(id)) {
            (<any>result)[id] = (<any>second)[id];
        }
    }
    return result;
}

class Person {
    constructor(public name: string) { }
}
interface Loggable {
    log(): void;
}
class ConsoleLogger implements Loggable {
    log() {
        // ...
    }
}
var jim = extend(new Person("Jim"), new ConsoleLogger());
var n = jim.name;
jim.log();
```


## Union Types 結合型

結合型は交差型に密接に関連していますが、それらは非常に異なって使用されます。
場合によっては、パラメータが `number` か `string` のいずれかであることを期待するライブラリを実行します。
たとえば、次の関数を使用します。

```typescript
/**
 * Takes a string and adds "padding" to the left.
 * If 'padding' is a string, then 'padding' is appended to the left side.
 * If 'padding' is a number, then that number of spaces is added to the left side.
 */
function padLeft(value: string, padding: any) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}

padLeft("Hello world", 4); // returns "    Hello world"
```

`padLeft` の問題は、 `padding` パラメーターが `any` と入力されていることです。
つまり、数字でも文字列でもない引数で呼び出すことができますが、TypeScriptでも問題ありません。

```typescript
let indentedString = padLeft("Hello world", true); // passes at compile time, fails at runtime.
```

伝統的なオブジェクト指向のコードでは、型の階層を作成して2つの型を抽象化することができます。
これははるかに明示的ですが、やはり少し過度のものです。
`padLeft` のオリジナルバージョンに関する素晴らしい点の1つは、プリミティブを渡すことができたことでした。
つまり、使い方は簡単で簡潔でした。
この新しいアプローチは、すでに他の場所に存在する関数を使用しようとしているだけでは役に立ちません。

`any` の代わりに、 `padding` パラメータに結合型を使用できます。

```typescript
/**
 * Takes a string and adds "padding" to the left.
 * If 'padding' is a string, then 'padding' is appended to the left side.
 * If 'padding' is a number, then that number of spaces is added to the left side.
 */
function padLeft(value: string, padding: string | number) {
    // ...
}

let indentedString = padLeft("Hello world", true); // errors during compilation
```

結合型は、いくつかの型のうちの1つである値を記述します。
縦棒（ `|` ）を使用して各タイプを区切ります。
したがって、 `number | string | boolean` は、 `number` 、 `string` 、または `boolean` のいずれかの値の型です。

結合型の値を持つ場合は、結合型のすべての型に共通のメンバーにのみアクセスできます。

```typescript
interface Bird {
    fly();
    layEggs();
}

interface Fish {
    swim();
    layEggs();
}

function getSmallPet(): Fish | Bird {
    // ...
}

let pet = getSmallPet();
pet.layEggs(); // okay
pet.swim();    // errors
```

結合型はここではちょっと難しいかもしれませんが、慣れるにはちょっとした直感が必要です。
型の値 `A | B` であれば、 `A` と `B` の両方にメンバーがあることがわかっています。
この例では、 `Bird` に `fly` という名前のメンバーがあります。
変数 `Bird | fly` には `fly` メソッドがあります。
実行時に変数が実際に `Fish` である場合、 `pet.fly()` の呼び出しは失敗します。


## Type Guards and Differentiating Types 型ガードと区別型

結合型は、値が重なり合う可能性のある型で重複している状況をモデル化するのに便利です。
私たちが `Fish` を持っているかどうかを具体的に知る必要があるときはどうなりますか？
2つの可能な値を区別するためのJavaScriptの共通のイディオムは、メンバーの存在をチェックすることです。
前述したように、結合型のすべての構成要素に含まれることが保証されているメンバーにのみアクセスできます。

```typescript
let pet = getSmallPet();

// Each of these property accesses will cause an error
if (pet.swim) {
    pet.swim();
}
else if (pet.fly) {
    pet.fly();
}
```

同じコードが動作するようにするには、型アサーションを使用する必要があります。

```typescript
let pet = getSmallPet();

if ((<Fish>pet).swim) {
    (<Fish>pet).swim();
}
else {
    (<Bird>pet).fly();
}
```


### User-Defined Type Guards ユーザー定義型ガード

型アサーションを何回か使用しなければならないことに注意してください。
一度チェックをしたら、各支店内の `pet` の型を知ることができればもっと良いでしょう。

TypeScriptには型ガードというものがあります。
型ガードは、ある種のスコープの型を保証するランタイムチェックを実行する式です。
型ガードを定義するには、戻り型が型述語である関数を定義するだけです。

```typescript
function isFish(pet: Fish | Bird): pet is Fish {
    return (<Fish>pet).swim !== undefined;
}
```

この例では、`pet is Fish` は型述語です。
  述部は、`parameterName is Type` の形式をとります。ここで、 `parameterName` は、現在の関数シグニチャーのパラメーターの名前でなければなりません。

`isFish` がいくつかの変数で呼び出されると、元の型と互換性がある場合、その変数をその特定の型に絞り込みます。

```typescript
// Both calls to 'swim' and 'fly' are now okay.

if (isFish(pet)) {
    pet.swim();
}
else {
    pet.fly();
}
```

TypeScriptは、`pet` が `if` ブランチの `Fish` であることを知っているだけでなく、 それはまた、他のブランチでは、`Fish` を持っていないことを知っているので、 `Bird` を持っている必要があります。


### `typeof` type guards `typeof` 型ガード

結合型を使用する `padLeft` のバージョンのコードを書き戻してみましょう。
次のように、型述語でそれを書くことができます。

```typescript
function isNumber(x: any): x is number {
    return typeof x === "number";
}

function isString(x: any): x is string {
    return typeof x === "string";
}

function padLeft(value: string, padding: string | number) {
    if (isNumber(padding)) {
        return Array(padding + 1).join(" ") + value;
    }
    if (isString(padding)) {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```

しかし、型がプリミティブであるかどうかを調べる関数を定義しなければならないことは、一種の痛みです。
幸運なことに、TypeScriptがそれ自身の型ガードとして認識するので、`typeof x === "number"` を独自の関数に抽象化する必要はありません。
これは、これらの小切手をインラインで書くことができることを意味します。

```typescript
function padLeft(value: string, padding: string | number) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```

これらの `typeof` 型ガード 、`typeof v === "typename"` と `typeof v！== "typename"` 、ここで  `"typename"` は  `"number"` 、  `"string"` 、  `"boolean"` 、または  `"symbol"` でなければなりません。
TypeScriptは他の文字列との比較を止めませんが、言語はそれらの式を型ガードとして認識しません。


### `instanceof` type guards `instanceof` 型ガード

`typeof` の型ガードについて読んだことがあり、JavaScriptの `instanceof` 演算子に精通している方は、おそらくこのセクションの内容を理解しているはずです。

`instanceof` 型ガードは、コンストラクタ関数を使用して型を絞り込む方法です。
たとえば、先ほど紹介した産業用ストリングパッドの例を借りてみましょう。

```typescript
interface Padder {
    getPaddingString(): string
}

class SpaceRepeatingPadder implements Padder {
    constructor(private numSpaces: number) { }
    getPaddingString() {
        return Array(this.numSpaces + 1).join(" ");
    }
}

class StringPadder implements Padder {
    constructor(private value: string) { }
    getPaddingString() {
        return this.value;
    }
}

function getRandomPadder() {
    return Math.random() < 0.5 ?
        new SpaceRepeatingPadder(4) :
        new StringPadder("  ");
}

// Type is 'SpaceRepeatingPadder | StringPadder'
let padder: Padder = getRandomPadder();

if (padder instanceof SpaceRepeatingPadder) {
    padder; // type narrowed to 'SpaceRepeatingPadder'
}
if (padder instanceof StringPadder) {
    padder; // type narrowed to 'StringPadder'
}
```

`instanceof` の右側はコンストラクタ関数である必要があり、TypeScriptは次のように以下の順序で絞り込みます。

1. 型が `any` でない場合は、関数 `prototype` プロパティの型
1. その型の構造体シグネチャによって返される結合型


## Nullable types NULL許容型

TypeScriptには、ヌルと未定義の2つの特別な型があり、値はそれぞれ `null` と `undefined` です。 
これらを [基本タイプセクション](../handbook/BASIC-TYPES.md) で簡単に説明しました。 
デフォルトでは、型チェッカーは `null` と `undefined` を何かに代入可能とみなします。 
事実上、 `null` と `undefined` はすべての型の有効な値です。 
つまり、たとえそれを防止したいとしても、どのタイプにも割り当てられないようにすることはできません。 
ヌルの発明者、Tony Hoareは、これを「[十億ドルの間違い](https://en.wikipedia.org/wiki/Null_pointer#History)」と呼んでいます。

`--strictNullChecks` フラグはこれを修正します。
変数を宣言すると、自動的に `null` または `undefined` は含まれません。 
結合型を使用して明示的にインクルードすることができます。

```typescript
let s = "foo";
s = null; // error, 'null' is not assignable to 'string'
let sn: string | null = "bar";
sn = null; // ok

sn = undefined; // error, 'undefined' is not assignable to 'string | null'
```

TypeScriptは、JavaScriptのセマンティクスにマッチするために、 `null` と `undefined` を別々に扱うことに注意してください。 
`string | null` は `string | undefined` および `string | undefined | null` とは異なるタイプです。


### Optional parameters and properties オプションのパラメータとプロパティ

`--strictNullChecks` を使用すると、オプションのパラメータが自動的に `| undefined` を追加します。

```typescript
function f(x: number, y?: number) {
    return x + (y || 0);
}
f(1, 2);
f(1);
f(1, undefined);
f(1, null); // error, 'null' is not assignable to 'number | undefined'
```

オプションのプロパティについても同様です。

```typescript
class C {
    a: number;
    b?: number;
}
let c = new C();
c.a = 12;
c.a = undefined; // error, 'undefined' is not assignable to 'number'
c.b = 13;
c.b = undefined; // ok
c.b = null; // error, 'null' is not assignable to 'number | undefined'
```


### Type guards and type assertions ガードと型アサーションを入力

ヌル許容型は結合型で実装されているため、 `null` を取り除くために型ガードを使用する必要があります。 
幸いにも、これは JavaScript で記述するのと同じコードです。

```typescript
function f(sn: string | null): string {
    if (sn == null) {
        return "default";
    }
    else {
        return sn;
    }
}
```

ここでは `null` の排除がはっきりしていますが、 `||` 演算子も使用できます。

```typescript
function f(sn: string | null): string {
    return sn || "default";
}
```

コンパイラが `null` または `undefined` を取り除くことができない場合は、型アサーション演算子を使用してそれらを手動で削除できます。
構文は postfix です。`identifier!` は、`identifier` の型から `null` と `undefined` を削除します。

```typescript
function broken(name: string | null): string {
  function postfix(epithet: string) {
    return name.charAt(0) + '.  the ' + epithet; // error, 'name' is possibly null
  }
  name = name || "Bob";
  return postfix("great");
}

function fixed(name: string | null): string {
  function postfix(epithet: string) {
    return name!.charAt(0) + '.  the ' + epithet; // ok
  }
  name = name || "Bob";
  return postfix("great");
}
```

この例ではネストした関数を使用しています。
これは、コンパイラーがネストされた関数（直ちに呼び出される関数式を除く）内のNULLを除去できないためです。 
これは、ネストされた関数へのすべての呼び出しを追跡できないためです。
特に、外部関数から戻した場合は特にそうです。 
関数が呼び出される場所がわからなければ、本体の実行時にどのような名前の型になるのかを知ることはできません。


## Type Aliases 型エイリアス

型エイリアスは、型の新しい名前を作成します。 
型エイリアスはインタフェースと似ていることもありますが、そうでなければ手作業で書く必要があるプリミティブ、結合型、タプルなどの名前を付けることができます。

```typescript
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
    if (typeof n === "string") {
        return n;
    }
    else {
        return n();
    }
}
```

エイリアシングは実際には新しい型を作成しません - その型を参照する新しい名前を作成します。 
プリミティブのエイリアシングは非常に便利ではありませんが、ドキュメントの形式として使用できます。

インタフェースと同様に、型エイリアスも汎用的です。型パラメータを追加してエイリアス宣言の右側で使用するだけです。

```typescript
type Container<T> = { value: T };
```

また、プロパティ内でタイプエイリアスを参照することもできます。

```typescript
type Tree<T> = {
    value: T;
    left: Tree<T>;
    right: Tree<T>;
}
```

交差のタイプと一緒に、私たちはいくつかの心の良い曲げ型を作ることができます。

```typescript
type LinkedList<T> = T & { next: LinkedList<T> };

interface Person {
    name: string;
}

var people: LinkedList<Person>;
var s = people.name;
var s = people.next.name;
var s = people.next.next.name;
var s = people.next.next.next.name;
```

ただし、宣言の右側の他の場所には、型エイリアスは表示されません。

```typescript
type Yikes = Array<Yikes>; // error
```


### Interfaces vs. Type Aliases インタフェース vs 型エイリアス

前述したように、型エイリアスは、同じようなインタフェースのように動作できます。 しかし、微妙な違いがあります。

1つの違いは、インタフェースがどこでも使用される新しい名前を作成することです。 
タイプエイリアスは新しい名前を作成しません。
たとえば、エラーメッセージはエイリアス名を使用しません。 
以下のコードでは、エディタで `interfaced` をホバーすると、`Interface` が返されますが、 `aliased` がオブジェクトのリテラル型を返すことが示されます。

```typescript
type Alias = { num: number }
interface Interface {
    num: number;
}
declare function aliased(arg: Alias): Alias;
declare function interfaced(arg: Interface): Interface;
```

もう1つ重要な違いは、型のエイリアスを拡張したり実装したり（他の型を拡張/実装することはできない）ことができないことです。 
[ソフトウェアの理想的な特性は拡張](https://en.wikipedia.org/wiki/Open/closed_principle)のために開かれているので、可能であれば、常に型エイリアス上のインタフェースを使用する必要があります。

一方、インタフェースでいくつかの形を表現できず、共用体またはタプル型を使用する必要がある場合は、通常はエイリアスが使用されます。


## String Literal Types 文字列リテラル型

文字列リテラル型では、文字列に必要な正確な値を指定できます。 
実際には、文字列リテラル型は、結合型、型ガード、型エイリアスとうまく組み合わせます。 
これらの機能を一緒に使用すると、文字列で列挙型の動作を得ることができます。

```typescript
type Easing = "ease-in" | "ease-out" | "ease-in-out";
class UIElement {
    animate(dx: number, dy: number, easing: Easing) {
        if (easing === "ease-in") {
            // ...
        }
        else if (easing === "ease-out") {
        }
        else if (easing === "ease-in-out") {
        }
        else {
            // error! should not pass null or undefined.
        }
    }
}

let button = new UIElement();
button.animate(0, 0, "ease-in");
button.animate(0, 0, "uneasy"); // error: "uneasy" is not allowed here
```

許可された3つの文字列のいずれかを渡すことができますが、他の文字列はエラーを返します。

```
Argument of type '"uneasy"' is not assignable to parameter of type '"ease-in" | "ease-out" | "ease-in-out"'
```

文字列リテラル型は、オーバーロードを区別するために同じ方法で使用できます。

```typescript
function createElement(tagName: "img"): HTMLImageElement;
function createElement(tagName: "input"): HTMLInputElement;
// ... more overloads ...
function createElement(tagName: string): Element {
    // ... code goes here ...
}
```


## Numeric Literal Types 数値リテラル型

TypeScriptには数字のリテラルタイプもあります。

```typescript
function rollDie(): 1 | 2 | 3 | 4 | 5 | 6 {
    // ...
}
```

これらは明示的に書かれていることはめったにありません。
狭くするとバグを捕まえることができます。

```typescript
function foo(x: number) {
    if (x !== 1 || x !== 2) {
        //         ~~~~~~~
        // Operator '!==' cannot be applied to types '1' and '2'.
    }
}
```

言い換えれば、 `x` が `2` と比較されるとき `x` は `1` でなければならない。
つまり、上記のチェックが無効な比較をしていることを意味する。


## Enum Member Types 列挙型メンバー

[列挙型のセクション](ENUMS.md)で説明したように、列挙型メンバーは、すべてのメンバーがリテラルで初期化されるときに型を持ちます。

多くのユーザーが「シングルトン型」と「リテラル型」を同じ意味で使用していますが、「シングルトン型」については、多くの場合、列挙型メンバと数値型/文字列型を参照しています。


## Discriminated Unions 差別化された結合型

シングルトン型、ユニオン型、型ガード、型エイリアスを組み合わせて、区別されたユニオン（タグ付きユニオンまたは代数データタイプとも呼ばれます）と呼ばれる高度なパターンを構築することができます。
差別化されたユニオンは関数型プログラミングに役立ちます。
いくつかの言語は自動的にあなたのためにユニオンを差別します。
TypeScript は今日のように JavaScript パターンをベースにしています。
以下3つの成分があります。

1. 判別式である共通のシングルトンタイプのプロパティを持つタイプ。
1. これらの型の和集合、つまりユニオンをとる型エイリアス。
1. 共通のプロパティのガードを入力します。

```typescript
interface Square {
    kind: "square";
    size: number;
}
interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
interface Circle {
    kind: "circle";
    radius: number;
}
```

最初に、結合するインタフェースを宣言します。 各インターフェイスには、異なる文字列リテラルタイプの `kind` プロパティがあります。 
`kind` プロパティは discriminant または tag と呼ばれます。 その他のプロパティは、各インタフェースに固有です。 
インタフェースは現在無関係であることに注意してください。 ユニオンに入れましょう。

```typescript
type Shape = Square | Rectangle | Circle;
```

差別化されたユニオンを使用しましょう。

```typescript
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
}
```

### Exhaustiveness checking 徹底的なチェック

差別化されたユニオンのすべての変種を網羅していないときにコンパイラに教えてほしい。 
たとえば、`Triangle` を `Shape` に追加すると、`area` も更新する必要があります。

```typescript
type Shape = Square | Rectangle | Circle | Triangle;
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
    // should error here - we didn't handle case "triangle"
}
```

これを行うには2つの方法があります。 最初に `--strictNullChecks` をオンにして戻り値の型を指定します。

```typescript
function area(s: Shape): number { // error: returns number | undefined
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
}
```

`switch` はもはや網羅的ではないため、関数が `undefined` を返すことがあることを TypeScript は認識しています。
明示的な戻り値の型番号がある場合、戻り値の型は実際には `number | undefined` であるというエラーが発生します。
しかし、このメソッドは非常に微妙で、しかも `--strictNullChecks` は古いコードでは必ずしも機能しません。

2番目のメソッドは、コンパイラが徹底的にチェックするために使用する `never` 型を使用します。

```typescript
function assertNever(x: never): never {
    throw new Error("Unexpected object: " + x);
}
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
        default: return assertNever(s); // error here if there are missing cases
    }
}
```

ここで、 `assertNever` は、 `s` が `never` 型であることをチェックします。
これは、他のすべてのケースが削除された後に残される型です。 
大文字小文字を忘れた場合、 `s` は実数型を持ち、型エラーが発生します。 
このメソッドでは、余分な関数を定義する必要がありますが、それを忘れるとはるかに明白です。


## Polymorphic `this` types  色々な `this` 型

色々な `this` 型は、包含するクラスまたはインタフェースのサブタイプである型を表します。 
これをF境界型多型と呼びます。 
これにより、例えば、階層的な流暢なインタフェースをより簡単に表現することができます。 
各操作の後にこれを返す簡単な電卓を用意しましょう。

```typescript
class BasicCalculator {
    public constructor(protected value: number = 0) { }
    public currentValue(): number {
        return this.value;
    }
    public add(operand: number): this {
        this.value += operand;
        return this;
    }
    public multiply(operand: number): this {
        this.value *= operand;
        return this;
    }
    // ... other operations go here ...
}

let v = new BasicCalculator(2)
            .multiply(5)
            .add(1)
            .currentValue();
```

クラスは `this` 型を使用しているため、拡張することができ、新しいクラスは変更を加えずに古いメソッドを使用できます。

```typescript
class ScientificCalculator extends BasicCalculator {
    public constructor(value = 0) {
        super(value);
    }
    public sin() {
        this.value = Math.sin(this.value);
        return this;
    }
    // ... other operations go here ...
}

let v = new ScientificCalculator(2)
        .multiply(5)
        .sin()
        .add(1)
        .currentValue();
```

`this` 型がなければ、 `ScientificCalculator` は `BasicCalculator` を拡張して流暢なインタフェースを維持することができませんでした。
`sin` 関数を持たない `BasicCalculator` を返すことになります。 しかし、`this` 型では、`this` が返されます。
ここでは `ScientificCalculator` です。


## Index types インデックス型

インデックス型を使用すると、動的プロパティ名を使用するコードをコンパイラにチェックさせることができます。 
たとえば、共通のJavascriptパターンは、オブジェクトからプロパティのサブセットを選択することです。

```typescript
function pluck(o, names) {
    return names.map(n => o[n]);
}
```

ここでは、インデックス型のクエリとインデックス付きアクセス演算子を使用して、この関数をTypeScriptで記述して使用する方法を示します。

```typescript
function pluck<T, K extends keyof T>(o: T, names: K[]): T[K][] {
  return names.map(n => o[n]);
}

interface Person {
    name: string;
    age: number;
}
let person: Person = {
    name: 'Jarid',
    age: 35
};
let strings: string[] = pluck(person, ['name']); // ok, string[]
```

コンパイラは、その `name` が実際に `Person` のプロパティであることを確認します。
この例では、新しい型の演算子をいくつか紹介します。 
まず、インデックス型のクエリ演算子である `T` の `keyof` があります。
任意のタイプ `T` について、 `T` の `keyof` は、 `T` の既知の公開プロパティ名の和集合である。
例えば…。

```typescript
let personProps: keyof Person; // 'name' | 'age'
```

`keyof Person` は  `'name' | 'age'` と完全に互換性があります。
違いは `Person` に別の `address：string` というプロパティを追加すると、`keyof Person` は自動的に `'name' | 'age' | 'address'` に更新されます。 「年齢」| '住所'。
また、 `keyof` を `pluck` のような汎用コンテキストで使用することもできます。
ここでは、事前にプロパティ名を知ることができません。
つまり、コンパイラはプロパティ名の正しいセットを渡して `pluck` することをチェックします。

```typescript
pluck(person, ['age', 'unknown']); // error, 'unknown' is not in 'name' | 'age'
```

第2の演算子は、索引付きアクセス演算子である `T[K]` です。
ここで、型構文は式の構文を反映しています。
つまり、 `person['name']` には `Person['name']` という型があります。この例では `string` だけです。
しかし、インデックス型のクエリと同様に、 `T[K]` を一般的なコンテキストで使用することができます。これは、その真の力が生まれる場所です。
型変数 `K extends keyof TT`が正しいことを確認するだけです。
`getProperty` という名前の関数を持つ別の例を次に示します。

```typescript
function getProperty<T, K extends keyof T>(o: T, name: K): T[K] {
    return o[name]; // o[name] is of type T[K]
}
```

`getProperty` では、 `o：T` と `name：K` となるので、`o[name]：T[K]` を意味します。 
`T[K]` の結果を返すと、コンパイラはキーの実際の型をインスタンス化します。
したがって、 `getProperty` の戻り型は、要求するプロパティによって異なります。

```typescript
let name: string = getProperty(person, 'name');
let age: number = getProperty(person, 'age');
let unknown = getProperty(person, 'unknown'); // error, 'unknown' is not in 'name' | 'age'
```

### Index types and string index signatures インデックス型と文字列インデックスのシグネチャ

`keyof` と `T[K]` は文字列インデックスシグネチャと対話する。
文字列のインデックスシグネチャを持つ型がある場合、 `keyof T` は文字列になります。
そして、 `T [string]` はインデックスシグネチャの単なる型です。

```typescript
interface Map<T> {
    [key: string]: T;
}
let keys: keyof Map<number>; // string
let value: Map<number>['foo']; // number
```

## Mapped types マップされた型

一般的なタスクは、既存の型を取り、それぞれのプロパティをオプションにすることです。

```typescript
interface PersonPartial {
    name?: string;
    age?: number;
}
```

あるいは、読み込み専用のバージョンが必要な場合もあります。

```typescript
interface PersonReadonly {
    readonly name: string;
    readonly age: number;
}
```

これは Javascript でよく起こります。
これは、TypeScript が古い型のマップされた型に基づいて新しい型を作成する方法を提供するということです。
マップされた型では、新しい型は同じ方法で古い型の各プロパティを変換します。
たとえば、型のすべてのプロパティを読み取り専用またはオプションにすることができます。
ここにいくつかの例があります。

```typescript
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
}
type Partial<T> = {
    [P in keyof T]?: T[P];
}
```

それを使うには…

```typescript
type PersonPartial = Partial<Person>;
type ReadonlyPerson = Readonly<Person>;
```

最も単純なマップされた型とその部分を見てみましょう。

```typescript
type Keys = 'option1' | 'option2';
type Flags = { [K in Keys]: boolean };
```

この構文は、 `for..in` が内部にあるインデックスシグネチャの構文に似ています。 
3つの部分があります。

1. 型変数Kは、順番に各プロパティにバインドされます。
1. 繰り返し処理するプロパティの名前を含む文字列リテラルのユニオンキーです。
1. 結果のプロパティの型。

この単純な例では、 `Keys` はプロパティ名のハードコードされたリストであり、プロパティの型は常に `boolean` です。
したがって、このマップされた型は次のようになります。

```typescript
type Flags = {
    option1: boolean;
    option2: boolean;
}
```

しかし、実際のアプリケーションは上記の `Readonly` または `Partial` のように見えます。
既存のタイプに基づいており、プロパティを何らかの方法で変換します。
それが `keyof` とインデックス付きのアクセスタイプが入っている場所です。

```typescript
type NullablePerson = { [P in keyof Person]: Person[P] | null }
type PartialPerson = { [P in keyof Person]?: Person[P] }
```

しかし、それは一般的なバージョンを持つと便利です。

```typescript
type Nullable<T> = { [P in keyof T]: T[P] | null }
type Partial<T> = { [P in keyof T]?: T[P] }
```

これらの例では、プロパティリストは `keyof T` であり、結果の型は `T[P]` の変形です。
これは、マップされた型の一般的な使用に適したテンプレートです。
これは、この種の変換が準同型であるためです。つまり、マッピングは `T` のプロパティのみに適用され、他のものは適用されません。
コンパイラは、新しいプロパティを追加する前に、既存のすべてのプロパティ修飾子をコピーできることを認識します。
たとえば、 `Person.name` が読み取り専用であった場合、 `Partial<Person>.name` は読み取り専用でオプションです。

`T[P]` が `Proxy<T>` クラスでラップされているもう1つの例があります。

```typescript
type Proxy<T> = {
    get(): T;
    set(value: T): void;
}
type Proxify<T> = {
    [P in keyof T]: Proxy<T[P]>;
}
function proxify<T>(o: T): Proxify<T> {
   // ... wrap proxies ...
}
let proxyProps = proxify(props);
```

`Readonly<T>` と `Partial<T>` は非常に便利なので、 `PickScript` と `Record` と共に TypeScript の標準ライブラリに含まれています。

```typescript
type ThreeStringProps = Record<'prop1' | 'prop2' | 'prop3', string>
```

非準同型型は基本的に新しいプロパティを作成しているので、どこからでもプロパティ修飾子をコピーすることはできません。


### Inference from mapped types マップされた型からの推論

ある型のプロパティをラップする方法を知ったので、次に行うべきことはそれらのラップを外すことです。 
幸いにも、それはかなり簡単です。

```typescript
function unproxify<T>(t: Proxify<T>): T {
    let result = {} as T;
    for (const k in t) {
        result[k] = t[k].get();
    }
    return result;
}

let originalProps = unproxify(proxyProps);
```

このアンラッピング推論は、準同型のマップされた型に対してのみ機能することに注意してください。 
マッピングされた型が準同型でない場合は、アンラッピング関数に明示的な型パラメータを与える必要があります。

