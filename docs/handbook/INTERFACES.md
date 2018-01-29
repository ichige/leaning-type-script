# Interfaces インタフェース

## Introduction はじめに

TypeScriptの基本原則の1つは、型チェックは値が持つ形に焦点を合わせることです。
これは「ダックタイピング」または「構造的サブタイピング」と呼ばれることがあります。
TypeScriptでは、インタフェースがこれらの型の名前付けの役割を果たすため、コード内のコントラクトやプロジェクト外のコードとのコントラクトを定義する強力な方法です。

## Our First Interface はじめてのインタフェース

インタフェースがどのように機能するかを見る最も簡単な方法は、単純な例から始めることです。

```typescript
function printLabel(labelledObj: { label: string }) {
    console.log(labelledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

型チェッカーは、`printLabel` への呼び出しをチェックします。
`printLabel` 関数には、渡されたオブジェクトにstring型の `label` というプロパティが必要な単一のパラメータがあります。
オブジェクトは実際にこれより多くのプロパティを持っていますが、コンパイラは少なくとも必要な型が存在し、必要な型と一致するかどうかだけチェックします。
TypeScriptが寛大ではない場合があります。これについては少し説明します。

同じ例をもう一度書くことができます。今度はインタフェースを使用して、`label` プロパティを文字列にする必要があることを記述します。

```typescript
interface LabelledValue {
    label: string;
}

function printLabel(labelledObj: LabelledValue) {
    console.log(labelledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

`LabelledValue` インタフェースは、前の例の要件を記述するために使用できる名前です。
それでも、文字列型の `label` という単一のプロパティを持つことを表します。
`printLabel` に渡すオブジェクトが他の言語のようにこのインタフェースを実装していることを明示的に述べる必要はありません。
ここで重要なのは形状だけです。
関数に渡すオブジェクトがリストされている要件を満たしている場合は、それが許可されます。

タイプチェッカーでは、これらのプロパティーが何らかの順序で指定されている必要はなく、インタフェースに必要なプロパティーが存在し、必要なタイプしか持たないことを指摘する事に価値があります。

## Optional Properties オプションプロパティ

インタフェースのすべてのプロパティが必要なわけではありません。
特定の条件の下に存在するものもあれば、まったく存在しないものもあります。
これらのオプションのプロパティは、「オプションバッグ」のようなパターンを作成するときによく使用されます。
ここでは、いくつかのプロパティを持つ関数にオブジェクトを渡します。

このパターンの例を次に示します。

```typescript
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
    let newSquare = {color: "white", area: 100};
    if (config.color) {
        newSquare.color = config.color;
    }
    if (config.width) {
        newSquare.area = config.width * config.width;
    }
    return newSquare;
}

let mySquare = createSquare({color: "black"});
```

オプションのプロパティを持つインタフェースは他のインタフェースと同様に記述され、宣言のプロパティー名の最後に `?` と表示されます。

オプションのプロパティの利点は、これら利用可能なプロパティを記述できる一方、インタフェースの一部ではないプロパティの使用を防ぐことができることです。
たとえば、`createSquare` の `color` プロパティの名前を誤って入力した場合、私たちに知らせるエラーメッセージが表示されます。

```typescript
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    let newSquare = {color: "white", area: 100};
    if (config.color) {
        // Error: Property 'clor' does not exist on type 'SquareConfig'
        newSquare.color = config.clor;
    }
    if (config.width) {
        newSquare.area = config.width * config.width;
    }
    return newSquare;
}

let mySquare = createSquare({color: "black"});
```

## Readonly properties 読込専用プロパティ

一部のプロパティは、オブジェクトが最初に作成されたときにのみ変更可能にする必要があります。
これは、プロパティの名前の前に `readonly` を置くことで指定できます。

```typescript
interface Point {
    readonly x: number;
    readonly y: number;
}
```

オブジェクトリテラルを割り当てることで `Point` を構築することができます。
代入後、`x` と `y` は変更できません。

```typescript
let p1: Point = { x: 10, y: 20 };
p1.x = 5; // error!
```

TypeScriptには、すべての変更メソッドが削除された `Array<T>` と同じ `ReadonlyArray<T>` 型が用意されているため、作成後に配列を変更しないようにすることができます。

```typescript
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;
ro[0] = 12; // error!
ro.push(5); // error!
ro.length = 100; // error!
a = ro; // error!
```

スニペットの最後の行では、`ReadonlyArray` 全体を通常の配列に戻すことさえも違法であることがわかります。
ただし、それでもタイプアサーションでオーバーライドすることはできます：

```typescript
a = ro as number[];
```

### `readonly` vs `const`

readonlyまたはconstを使用するかどうかを覚える最も簡単な方法は、変数またはプロパティで使用するかどうかを尋ねることです。
変数は `const` を使用し、プロパティはr `eadonly` を使用します。
 

## Excess Property Checks 余剰プロパティのチェック

インタフェースを使用した最初の例では、TypeScriptを使用すると、`{ label: string; }` を期待しているものに `{ size: number; label: string; }` を渡すことができます。
オプションのプロパティについても学び、いわゆる「オプションバッグ」を記述するときにそれらがどのように役立つかについても学びました。

しかし、2つを巧みに組み合わせることで、JavaScriptの場合と同じように足で自分を撃つことができます。
例えば、`createSquare` を使った最後の例を考えてみましょう。

```typescript
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}

let mySquare = createSquare({ colour: "red", width: 100 });
```

`createSquare` の与えられた引数は、`color` ではなく書き間違えの `colour `であることに注意してください。
単純なJavaScriptでは、この種のことは黙って失敗します。

しかし、TypeScriptは、おそらくこのコードにバグがあると考えています。
オブジェクトリテラルは特別な扱いを受け、それらを他の変数に代入するとき、または引数として渡すときに過剰なプロパティ検査を受けます。
オブジェクトリテラルに "ターゲットタイプ"にないプロパティがある場合、エラーが発生します。

```typescript
// error: 'colour' not expected in type 'SquareConfig'
let mySquare = createSquare({ colour: "red", width: 100 });
```

これらのチェックを回避することは、実際にはとても簡単です。
最も簡単な方法は型アサーションを使うことです。

```typescript
let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
```

ただし、オブジェクトが特別な方法で使用されるいくつかの追加のプロパティを持つことができる場合は、文字列のインデックスシグネチャを追加する方がよいでしょう。
`SquareConfig` が上記のタイプの `color` と `width` のプロパティを持つことができますが、他のプロパティもいくつでも持つことができる場合、SquareConfigsは次のように定義できます。

```typescript
interface SquareConfig {
    color?: string;
    width?: number;
    [propName: string]: any;
}
```

インデックスシグネチャについては少し説明しますが、ここでは `SquareConfig` に任意の数のプロパティを持たせることができます。
また、`color` や `width` でない限り、その型は関係ありません。

これらのチェックを回避する最後の方法は、驚くべきことですが、オブジェクトを別の変数に割り当てることです： `squareOptions` はプロパティのチェックを余儀なくされるので、コンパイラはエラーを返さないでしょう。

```typescript
let squareOptions = { colour: "red", width: 100 };
let mySquare = createSquare(squareOptions);
```

上記のような簡単なコードの場合、これらのチェックを "回避"しようとしないでください。
メソッドと保持状態を持つより複雑なオブジェクトリテラルでは、これらのテクニックを念頭に置く必要があるかもしれませんが、過剰なプロパティエラーの大部分は実際にはバグです。
つまり、オプションバッグのようなものに対して過剰な財産チェックの問題が発生している場合は、型宣言のいくつかを修正する必要があります。
この例では、`color` または `colour` プロパティの両方を持つオブジェクトを `createSquare` に渡すことができれば、それを反映する `SquareConfig` の定義を修正する必要があります。


## Function Types 関数型

インタフェースは、JavaScriptオブジェクトが取ることができる幅広いシェイプを記述することができます。
プロパティを持つオブジェクトを記述することに加えて、インタフェースは関数型を記述することもできます。

インタフェースを持つ関数型を記述するために、インタフェースに呼び出しシグネチャを与えます。
これは、パラメータリストと戻り値の型だけを指定した関数宣言に似ています。
パラメータリストの各パラメータには、名前とタイプの両方が必要です。

```typescript
interface SearchFunc {
    (source: string, subString: string): boolean;
}
```

ひとたび定義されると、他のインタフェースと同じように、この関数型インタフェースを使うことができます。
ここでは、関数型の変数を作成し、同じ型の関数値を割り当てる方法を示します。

```typescript
let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
    let result = source.search(subString);
    return result > -1;
}
```

関数型が正しく型チェックされるためには、パラメータの名前が一致する必要はありません。
たとえば、次のように上記の例を書いたとします。

```typescript
let mySearch: SearchFunc;
mySearch = function(src: string, sub: string): boolean {
    let result = src.search(sub);
    return result > -1;
}
```

関数のパラメータは、一度に1つずつチェックされ、対応する各パラメータ位置のタイプが互いに照合されます。
型をまったく指定したくない場合は、関数型の値が `SearchFunc` 型の変数に直接代入されているため、TypeScriptのコンテキスト型は型の型を推論できます。
ここでもまた、関数式の戻り値の型は、それが返す値（ここでは `false` と `true`）によって暗示されます。
関数式が数値または文字列を返す場合、型チェッカーは、戻り値の型が `SearchFunc` インタフェースで記述された戻り値の型と一致しないことを警告していました。

```typescript
let mySearch: SearchFunc;
mySearch = function(src, sub) {
    let result = src.search(sub);
    return result > -1;
}
```

## Indexable Types インデックス型

関数型を記述するためにインタフェースを使う方法と同様に、`a[10]` や `ageMap["daniel"]` のように "インデックスを付ける"こともできる型を記述することができます。
インデックス付け可能な型には、インデックス付け時に対応する戻り型とともに、オブジェクトに索引付けするために使用できる型を記述する索引シグネチャがあります。
例を見てみましょう。

```typescript
interface StringArray {
    [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```

上では、インデックスシグネチャを持つ `StringArray` インタフェースがあります。
このインデックスシグネチャは、`StringArray` が `number` でインデックス付けされている場合、`string` を返します。

サポートされるインデックスシグネチャには、`string` と `number` の2種類があります。
両方の型のインデクスをサポートすることは可能ですが、数値インデクスから返される型は、文字列インデクサから返される型のサブタイプでなければなりません。
これは、`number` を使用してインデックスを付ける場合、JavaScriptは実際にインデックスをオブジェクトに挿入する前に `string` に変換するためです。
つまり、`100`（1つの `number`）での索引付けは「`100`」（1つの `string`）による索引付けと同じであるため、2つは一貫している必要があります。

```typescript
class Animal {
    name: string;
}
class Dog extends Animal {
    breed: string;
}

// Error: indexing with a 'string' will sometimes get you an Animal!
interface NotOkay {
    [x: number]: Animal;
    [x: string]: Dog;
}
```

文字列インデックスシグネチャは "辞書"パターンを記述する強力な方法ですが、すべてのプロパティが戻り値の型と一致するように強制します。
これは、文字列インデックスが `obj.property` を `obj["property"]` としても使用できることを宣言するためです。
次の例では、`name` の型が文字列型の型と一致せず、型チェッカーでエラーが発生します。

```typescript
interface NumberDictionary {
    [index: string]: number;
    length: number;    // ok, length is a number
    name: string;      // error, the type of 'name' is not a subtype of the indexer
}
```

最後に、インデックスへの割り当てを防ぐために、インデックスシグネチャを読み取り専用にすることができます。

```typescript
interface ReadonlyStringArray {
    readonly [index: number]: string;
}
let myArray: ReadonlyStringArray = ["Alice", "Bob"];
myArray[2] = "Mallory"; // error!
```

インデックスのシグネチャが読み込み専用であるため、`myArray[2]` は設定できません。

## Class Types クラス型

### Implementing an interface インタフェースの実装

C＃やJavaなどの言語でのインタフェースの最も一般的な使用法の1つは、クラスが特定の契約を満たすことを明示的に強制することです。
これはTypeScriptでも可能です。

```typescript
interface ClockInterface {
    currentTime: Date;
}

class Clock implements ClockInterface {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

次の例の `setTime` と同様に、クラス内に実装されているインタフェース内のメソッドを記述することもできます。

```typescript
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

インタフェースは、パブリックとプライベートの両方ではなく、クラスのパブリック側を記述します。
これにより、クラスを使用して、クラスインスタンスのプライベート側にも特定の型があることを確認することができなくなります。

### Difference between the static and instance sides of classes クラスのインスタンスとインスタンスの違い

クラスとインタフェースを操作する場合、クラスには2つのタイプがあることに留意してください。
1つは静的側の型とインスタンス側の型です。
コンストラクトシグネチャでインターフェイスを作成し、このインターフェイスを実装するクラスを作成しようとすると、エラーが発生することがあります。

```typescript
interface ClockConstructor {
    new (hour: number, minute: number);
}

class Clock implements ClockConstructor {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

これは、クラスがインタフェースを実装する場合、そのクラスのインスタンス側のみがチェックされるためです。
コンストラクタは静的な側にあるため、このチェックには含まれません。

代わりに、クラスの静的な側で直接作業する必要があります。
この例では、コンストラクタの `ClockConstructor` とインスタンスメソッドの `ClockInterface` の2つのインタフェースを定義します。
次に、便宜上、渡される型のインスタンスを作成するコンストラクタ関数 `createClock` を定義します。

```typescript
interface ClockConstructor {
    new (hour: number, minute: number): ClockInterface;
}
interface ClockInterface {
    tick();
}

function createClock(ctor: ClockConstructor, hour: number, minute: number): ClockInterface {
    return new ctor(hour, minute);
}

class DigitalClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("beep beep");
    }
}
class AnalogClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("tick tock");
    }
}

let digital = createClock(DigitalClock, 12, 17);
let analog = createClock(AnalogClock, 7, 32);
```

## Extending Interfaces インタフェースの拡張

クラスと同様に、インタフェースは互いに拡張することができます。
これにより、あるインタフェースのメンバーを別のインタフェースにコピーすることができます。
これにより、インタフェースを再利用可能なコンポーネントに分ける方法をより柔軟にすることができます。

```typescript
interface Shape {
    color: string;
}

interface Square extends Shape {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
```

インタフェースは複数のインタフェースを拡張して、すべてのインタフェースの組み合わせを作成することができます。

```typescript
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

## Hybrid Types ハイブリッド型

前述のように、インタフェースは現実のJavaScriptに存在する豊富な種類を記述することができます。
JavaScriptの動的で柔軟な性質のため、時には上記のいくつかのタイプの組み合わせとして機能するオブジェクトに遭遇することがあります。

このような例の1つは、関数とオブジェクトの両方として機能し、追加のプロパティを持つオブジェクトです。

```typescript
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = <Counter>function (start: number) { };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```

サードパーティのJavaScriptとやりとりするときは、型の形を完全に記述するために上記のようなパターンを使用する必要があります。


## Interfaces Extending Classes インタフェースを用いたクラスの拡張

インタフェース型がクラス型を拡張する場合、クラスのメンバを継承しますが、実装は継承しません。
これは、インタフェースが実装を提供せずにクラスのすべてのメンバーを宣言したかのようです。
インタフェースは、基本クラスのprivateメンバーとprotectedメンバーを継承します。
つまり、privateメンバーまたはprotectedメンバーを持つクラスを拡張するインタフェースを作成すると、そのインタフェースタイプはそのクラスまたはそのサブクラスによってのみ実装できます。

これは、継承階層が大きく、特定のプロパティを持つサブクラスのみでコードが機能するように指定する場合に便利です。
サブクラスは基本クラスから継承する以外に関連する必要はありません。 例えば…

```typescript
class Control {
    private state: any;
}

interface SelectableControl extends Control {
    select(): void;
}

class Button extends Control implements SelectableControl {
    select() { }
}

class TextBox extends Control {

}

// Error: Property 'state' is missing in type 'Image'.
class Image implements SelectableControl {
    select() { }
}

class Location {

}
```

上記の例では、`SelectableControl` には、private `state` プロパティを含む `Control` のすべてのメンバーが含まれています。
`state` はprivateメンバーなので、`Control` の子孫は `SelectableControl` を実装することしかできません。
これは、`Control` の子孫だけが同じ宣言で始まる`state` privateメンバーを持つためです。
これは、priavteメンバーが互換性があるための要件です。

`Control` クラス内では、`SelectableControl` のインスタンスを通じて状態プライベートメンバにアクセスできます。
効果的に、`SelectableControl` は `select` メソッドを持つことが知られている `Control` のように動作します。
`Button` クラスと `TextBox` クラスは、`SelectableControl` のサブタイプです（これらは両方とも `Control` から継承され、`select` メソッドを持っているため）が、`Image` クラスと `Location` クラスはそうではありません。
