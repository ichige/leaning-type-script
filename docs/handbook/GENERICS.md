# GENERICS 汎用型

## はじめに

ソフトウェアエンジニアリングの大部分は、明確かつ一貫したAPIを持つだけでなく、再利用可能なコンポーネントを構築することです。
明日のデータだけでなく、今日のデータを処理できるコンポーネントは、大規模なソフトウェアシステムを構築するための最も柔軟な機能を提供します。

C＃やJavaなどの言語では、再利用可能なコンポーネントを作成するためのツールボックスの主なツールの1つはジェネリクスであり、1つではなくさまざまなタイプで動作するコンポーネントを作成できます。
これにより、ユーザーはこれらのコンポーネントを使用して独自のタイプを使用できます。


## Hello World of Generics ジェネリクスの Hallo World.

まず、ジェネリクスの「Hello World.」をやってみましょう。アイデンティティ関数です。 
アイデンティティ関数は、渡されたものを返す関数です。これは、`echo` コマンドと同様に考えることができます。

ジェネリクスがなければ、アイデンティティ関数に特定の型を与える必要があります。

```typescript
function identity(arg: number): number {
    return arg;
}
```

あるいは、`any` 型を使ってアイデンティティ関数を記述することができます。

```typescript
function identity(arg: any): any {
    return arg;
}
```

`any` を使うのは確かに一般的ですが、関数が `arg` の型の任意の型とすべての型を受け入れるようにするため、関数が返ったときの型に関する情報は実際には失われています。
番号を渡した場合、唯一の情報はすべての型が返されるということです。

代わりに、返されているものを示すために引数の型を取り込む方法が必要です。
ここでは、タイプ変数、つまり値ではなくタイプで動作する特殊な変数を使用します。

```typescript
function identity<T>(arg: T): T {
    return arg;
}
```

ここで、型変数 `T` を `identity` 関数に追加しました。
この `T` は、ユーザーが提供するタイプ（ `number` など）を取得できるため、後でその情報を使用できます。
ここでは、戻り値の型として `T` を再度使用します。
検査の結果、引数と戻り値の型に同じ型が使用されていることがわかりました。
これにより、関数の一方の側でその型情報をトラフィックすることができます。

`identity` 関数のこのバージョンは汎用的なものであり、さまざまなタイプの関数で動作します。
`any` を使用するのとは異なり、引数と戻り値の型を使用した最初の `identity` 関数としても正確です（つまり、情報を失うことはありません）。

汎用型 `identity` 関数を書いたら、次の2つの方法のいずれかでコールすることができます。
最初の方法は、型引数を含むすべての引数を関数に渡すことです。

```typescript
let output = identity<string>("myString");  // type of output will be 'string'
```

ここでは、関数呼び出しの引数の1つとして `T` を `string` に明示的に設定します。これは、`()` の代わりに `<>` を使用して表します。

第2の方法はおそらく最も一般的です。
ここでは、型引数の推論を使用します。つまり、渡す引数の型に基づいて自動的に `T` の値を設定します。

```typescript
let output = identity("myString");  // type of output will be 'string'
```

角カッコ（ `<>` ）で型を明示的に渡す必要はなかったことに注意してください。コンパイラは値  `"myString"` を見て、`T` をその型に設定しました。
型引数の推論は、コードをより短く読みやすくするための有益なツールですが、より複雑な例で起こり得るように、コンパイラが型を推測しないときに前の例で行ったように、型引数を明示的に渡す必要があります 。


### Working with Generic Type Variables 汎用型変数の操作

汎用型の使用を開始すると、`identity` のような汎用型関数を作成すると、関数の本体に一般的に型付けされたパラメータを正しく使用することがコンパイラによって強制されます。
つまり、これらのパラメータは実際には、すべての型であるかのように扱われます。

先ほどのアイデンティティ関数を使ってみましょう。

```typescript
function identity<T>(arg: T): T {
    return arg;
}
```

それぞれの呼び出しで引数 `arg` の長さをコンソールに記録したい場合はどうすればよいでしょうか？ 
これを書くように誘惑されるかもしれません。

```typescript
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

コンパイラは、`arg` の `.length` メンバを使用しているというエラーを返しますが、 `arg` にはこのメンバがありません。
これらの型変数は、すべての型のために用意されていることを覚えておいてください。この関数を使用する場合は、代わりに `.length` メンバを持たない `number` を渡すことができます。

実際には、この関数が `T` の代わりに `T` の配列を直接操作することを意図したとしましょう。
配列を扱っているので、 `.length` メンバーが利用できるはずです。 他の型の配列を作成するのと同じように記述できます。

```typescript
function loggingIdentity<T>(arg: T[]): T[] {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

「汎用型関数 `loggingIdentity` は型パラメータ `T` をとり、引数 `arg` は `T` の配列であり、 `T` の配列を返します」というように、 `loggingIdentity` の型を読み取ることができます。
数字の配列を渡すと、 `T` が `number` に束縛されるので、数字の配列が返されます。
これにより、タイプ全体ではなく、作業しているタイプの一部として汎用型の変数 `T` を使用できるようになり、柔軟性が向上します。

代わりに、実例のサンプルを次のように書くこともできます。

```typescript
function loggingIdentity<T>(arg: Array<T>): Array<T> {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

他の言語のこの型のスタイルにはすでに慣れているかもしれません。 
次のセクションでは、 `Array<T>` のような独自の汎用型を作成する方法について説明します。


## Generic Types 汎用型

前のセクションでは、さまざまな種類の関数を扱う汎用の `identity` 関数を作成しました。
このセクションでは、関数そのものの型と汎用型インタフェースの作成方法について説明します。

汎用型関数の型は、汎用型宣言と同様に、型パラメータが最初にリストされている非汎用型関数の型に似ています。

```typescript
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: <T>(arg: T) => T = identity;
```

また、型変数の数と型変数の使用方法が一致していれば、その型の汎用型パラメータに別の名前を使用することもできます。

```typescript
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: <U>(arg: U) => U = identity;
```

汎用型をオブジェクトリテラル型の呼び出しシグネチャとして書くこともできます。

```typescript
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: {<T>(arg: T): T} = identity;
```

これは最初の汎用型インタフェースの作成につながります。 
前の例のオブジェクトリテラルを取り出し、それをインタフェースに移動しましょう。

```typescript
interface GenericIdentityFn {
    <T>(arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn = identity;
```

同様の例では、汎用型パラメータをインタフェース全体のパラメータにすることができます。
これにより、どのタイプがは汎用的であるかがわかります（たとえば、 `Dictionary` ではな `Dictionary<string>` ）。
これにより、インタフェースの他のすべてのメンバが型パラメータを参照できるようになります。

```typescript
interface GenericIdentityFn<T> {
    (arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

私たちの例は若干異なるものに変更されています。 汎用型関数を記述する代わりに、汎用型の一部である非汎用型関数シグネチャを使用できるようになりました。
`GenericIdentityFn` を使用する場合、対応する型引数（ここでは `number` ）を指定して、基本的な呼び出しシグネチャが使用するものを効果的にロックする必要があります。
型シグネチャをコールシグネチャに直接配置するタイミングと、それをインタフェース自体に配置するタイミングを理解することは、タイプのどの側面が汎用的であるかを説明するのに役立ちます。

汎用型インタフェースに加えて、汎用クラスも作成できます。 汎用的な列挙型と名前空間を作成することはできません。


## Generic Classes 汎用型クラス

汎用型クラスは汎用型インタフェースと似た形をしています。
汎用型クラスは、クラス名の後ろに山形括弧（ `<>` ）で囲まれた汎用型パラメータリストを持っています。

```typescript
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

これは `GenericNumber` クラスをかなりリテラルに使用していますが、数値型のみを使用するように制限するものは何もないことに気づいたことがあります。
代わりに文字列やさらに複雑なオブジェクトを使用することもできました。

```typescript
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function(x, y) { return x + y; };

alert(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

インタフェースと同様に、型パラメータをクラス自体に置くことで、クラスのすべてのプロパティが同じ型で動作していることを確認できます。

[クラスについてのセクション](https://www.typescriptlang.org/docs/handbook/classes.html)で説明したように、クラスにはそのタイプの2つの側面、つまり静的な側面とインスタンスの側面があります。
汎用クラスは静的な側ではなく、インスタンス側でのみ汎用的です。したがって、クラスを操作する場合、静的メンバーはクラスの型パラメータを使用できません。


## Generic Constraints 汎用型の制約

前の例から覚えていると、タイプのセットがどのような機能を持つかについてある程度知っているタイプのセットで動作する汎用型関数を記述したいことがあります。
`loggingIdentity` の例では、 `arg` の `.length` プロパティにアクセスできるようにしたかったのですが、コンパイラはすべての型に `.length` プロパティがあることを証明できなかったので、この仮定をすることはできないと警告しています。

```typescript
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

任意の型とすべての型で作業するのではなく、 `.length` プロパティも持つすべての型でこの関数を動作させるようにしたいと考えています。
型がこのメンバを持っている限り、それを許可しますが、少なくともこのメンバを持つ必要があります。
そうするためには、`T` ができるものに制約として要件を列挙しなければなりません。

これを行うために、私たちの制約を説明するインタフェースを作成します。
ここでは、単一の `.length` プロパティを持つインタフェースを作成し、このインターフェイスと `extends` キーワードを使用して制約を示します。

```typescript
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}
```

汎用型関数は現在制限されているため、すべての型で動作しなくなります。

```typescript
loggingIdentity(3);  // Error, number doesn't have a .length property
```

代わりに、型に必要なすべてのプロパティがある値を渡す必要があります。

```typescript
loggingIdentity({length: 10, value: 3});
```


### Using Type Parameters in Generic Constraints 汎用型制約での型パラメータの使用

別の型パラメータによって制約される型パラメータを宣言できます。
たとえば、ここでは名前からオブジェクトのプロパティを取得したいとします。
`obj` に存在しないプロパティを誤って取得していないことを確認したいので、2つのタイプの間に制約を設けます。

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K) {
    return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a"); // okay
getProperty(x, "m"); // error: Argument of type 'm' isn't assignable to 'a' | 'b' | 'c' | 'd'.
```


### Using Class Types in Generics 汎用型でのクラス型の使用

汎用型を使用してTypeScriptでファクトリを作成する場合、そのクラスの型をそのコンストラクタ関数で参照する必要があります。
例えば…

```typescript
function create<T>(c: {new(): T; }): T {
    return new c();
}
```

より高度な例では、プロトタイプ・プロパティを使用して、コンストラクタ関数とクラス・タイプのインスタンス側の関係を推測し、制約します。

```typescript
class BeeKeeper {
    hasMask: boolean;
}

class ZooKeeper {
    nametag: string;
}

class Animal {
    numLegs: number;
}

class Bee extends Animal {
    keeper: BeeKeeper;
}

class Lion extends Animal {
    keeper: ZooKeeper;
}

function createInstance<A extends Animal>(c: new () => A): A {
    return new c();
}

createInstance(Lion).keeper.nametag;  // typechecks!
createInstance(Bee).keeper.hasMask;   // typechecks!
```


