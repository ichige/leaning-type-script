# Classes クラス

## Introduction はじめに

伝統的なJavaScriptでは、関数とプロトタイプベースの継承を使用して再利用可能なコンポーネントを構築していますが、これは、クラスが継承するオブジェクト指向アプローチや、クラスからオブジェクトを構築するプログラマーにとっては面倒です。
ECMAScript 2015（ECMAScript 6）から始まって、JavaScriptプログラマはこのオブジェクト指向のクラスベースのアプローチを使用してアプリケーションを構築することができます。
TypeScriptでは、開発者がこれらのテクニックを今すぐ使用し、JavaScriptの次のバージョンを待つことなく、すべての主要なブラウザとプラットフォームで動作するJavaScriptにコンパイルできます。


## Classes クラスとは

クラスベースの簡単な例を見てみましょう。

```typescript
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter = new Greeter("world");
```

以前はC＃またはJavaを使用していた場合、構文はよく分かります。
新しいクラスの `Greeter` を宣言します。
このクラスには、`greeting` という名前のプロパティ、コンストラクタ、および `greet` メソッドという3つのメンバがあります。

クラスのメンバーの1つを参照するとき、`this.` を前に置きます 。
これはメンバーアクセスであることを示しています。

最後の行では、`new` を使用して `Greeter` クラスのインスタンスを作成します。
これは先に定義したコンストラクタを呼び出し、`Greeter` シェイプで新しいオブジェクトを作成し、コンストラクタを実行して初期化します。


## Inheritance 継承

TypeScriptでは、共通のオブジェクト指向のパターンを使用できます。
クラスベースのプログラミングで最も基本的なパターンの1つは、継承を使用して新しいクラスを作成するために既存のクラスを拡張できることです。

例を見てみましょう。

```typescript
class Animal {
    move(distanceInMeters: number = 0) {
        console.log(`Animal moved ${distanceInMeters}m.`);
    }
}

class Dog extends Animal {
    bark() {
        console.log('Woof! Woof!');
    }
}

const dog = new Dog();
dog.bark();
dog.move(10);
dog.bark();
```

この例は、最も基本的な継承機能を示しています。クラスは、基本クラスからプロパティとメソッドを継承します。
ここで、`Dog` は、`extends` キーワードを使用して `Animal` 基本クラスから派生した派生クラスです。
派生クラスはしばしばサブクラスと呼ばれ、基本クラスはスーパークラスと呼ばれることが多い。

`Dog` は `Animal` から機能を拡張しているので、`bark()` と `move()` の両方が可能な `Dog` のインスタンスを作成することができました。

もっと複雑な例を見てみましょう。

```typescript
class Animal {
    name: string;
    constructor(theName: string) { this.name = theName; }
    move(distanceInMeters: number = 0) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}

class Snake extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 5) {
        console.log("Slithering...");
        super.move(distanceInMeters);
    }
}

class Horse extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 45) {
        console.log("Galloping...");
        super.move(distanceInMeters);
    }
}

let sam = new Snake("Sammy the Python");
let tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34);
```

この例では、これまで述べていない他のいくつかの機能について説明します。
ここでも、`Animal:Horse` と `Animal:Snake` の2つの新しいサブクラスを作成するために使用される `extends` キーワードがあります。

先の例との1つの違いは、コンストラクタ関数を含む各派生クラスは、基底クラスのコンストラクタを実行する `super()` を呼び出す必要があることです。
さらに、コンストラクタ本体のプロパティにアクセスする前に、`super()` を呼び出す必要があります。
これはTypeScriptが強制する重要なルールです。

この例では、基本クラスのメソッドを、そのサブクラスに特化したメソッドでオーバーライドする方法も示しています。
ここでは、`Snake` と `Horse` の両方が `Animal` からの `move` をオーバーライドする `move` メソッドを作成し、各クラスに固有の機能を提供します。
`tom` は `Animal` として宣言されていますが、その値は `Horse` であるため、`tom.move(34)` を呼び出すと `Horse` のオーバーライドメソッドが呼び出されます。

```
Slithering...
Sammy the Python moved 5m.
Galloping...
Tommy the Palomino moved 34m.
```


## Public, private, and protected modifiers `public`、`provate`、`protected` 修飾子

### Public by default デフォルトは `public`

この例では、プログラムを通じて宣言したメンバーに自由にアクセスできました。
あなたが他の言語のクラスに慣れているなら、上記の例ではこれを実現するために `public` という言葉を使用する必要はなかったことに気づいたかもしれません。
たとえば、C＃では、各メンバを明示的に  `public` と指定して可視にする必要があります。
TypeScriptでは、各メンバーはデフォルトで `public` です。

メンバーを明示的に `public` にマークすることができます。
前のセクションの `Animal` クラスを次のように書くことができました。

```typescript
class Animal {
    public name: string;
    public constructor(theName: string) { this.name = theName; }
    public move(distanceInMeters: number) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}
```

### Understanding private `private` を理解する

メンバーが `private` とマークされている場合、そのメンバーを含むクラスの外部からアクセスすることはできません。 
例えば…

```typescript
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

new Animal("Cat").name; // Error: 'name' is private;
```

TypeScriptは構造型システムです。
2つの異なる型を比較すると、どこから来たのかにかかわらず、すべての型の型が互換性があれば、型自体に互換性があると言います。

しかし、`private` メンバーと `protected` メンバーを持つタイプを比較する場合、これらのタイプを異なる方法で扱います。
互換性があると見なされる2つのタイプのうち、一方が `private` メンバーを持つ場合、他方は同じ宣言で作成された `private` メンバーを持っていなければなりません。
`protected` メンバーにも同じことが言えます。

実際にこれがどのように機能するかをよく見てみましょう。

```typescript
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

class Rhino extends Animal {
    constructor() { super("Rhino"); }
}

class Employee {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

let animal = new Animal("Goat");
let rhino = new Rhino();
let employee = new Employee("Bob");

animal = rhino;
animal = employee; // Error: 'Animal' and 'Employee' are not compatible
```

この例では、`Animal` と `Rhino` があり、`Rhino` は `Animal` のサブクラスです。
我々はまた、形状に関して `Animal` と同じように見える新しいクラス `Employee` を持っています。
これらのクラスのいくつかのインスタンスを作成し、次にそれらを相互に割り当てて、何が起こるかを確認します。
`Animal` と `Rhino` は `Animal` の `private name: string` と同じ宣言から自分の形の `private` サイドを共有しているので、互換性があります。
しかし、`Employee` の場合はそうではありません。
`Employee` から `Animal` に割り当てようとすると、これらの型に互換性がないというエラーが発生します。
`Employee` には `name` という名前の `private` メンバーもありますが、これは `Animal` で宣言したメンバーではありません。

### Understanding protected `protected を理解する

`protected` 修飾子は、`private` 修飾子とよく似た働きをします。
ただし、`protected` 宣言されたメンバーには、派生クラスのインスタンスからアクセスすることができます。 
例えば…

```typescript
class Person {
    protected name: string;
    constructor(name: string) { this.name = name; }
}

class Employee extends Person {
    private department: string;

    constructor(name: string, department: string) {
        super(name);
        this.department = department;
    }

    public getElevatorPitch() {
        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
    }
}

let howard = new Employee("Howard", "Sales");
console.log(howard.getElevatorPitch());
console.log(howard.name); // error
```

`Person` の外部からの `name` は使用できませんが、`Employee` が `Person` から派生しているため、`Employee` のインスタンスメソッド内から使用することができます。

コンストラクタには `protected` とマークすることもできます。
これは、そのクラスがそのクラスを包含するクラス外でインスタンス化することはできないが、拡張できることを意味する。 
例えば…

```typescript
class Person {
    protected name: string;
    protected constructor(theName: string) { this.name = theName; }
}

// Employee can extend Person
class Employee extends Person {
    private department: string;

    constructor(name: string, department: string) {
        super(name);
        this.department = department;
    }

    public getElevatorPitch() {
        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
    }
}

let howard = new Employee("Howard", "Sales");
let john = new Person("John"); // Error: The 'Person' constructor is protected
```


## Readonly modifier 読み取り専用 修飾子

`readonly` キーワードを使用すると、プロパティを読み取り専用にすることができます。
読み取り専プロパティは、宣言またはコンストラクタで初期化する必要があります。

```typescript
class Octopus {
    readonly name: string;
    readonly numberOfLegs: number = 8;
    constructor (theName: string) {
        this.name = theName;
    }
}
let dad = new Octopus("Man with the 8 strong legs");
dad.name = "Man with the 3-piece suit"; // error! name is readonly.
```

### Parameter properties パラメータプロパティ

最後の例では、`Octopus` クラスの `readonly` メンバ名とコンストラクタパラメータ `theName` を宣言し、直ちに `name` に `theName` を設定しなければなりませんでした。
これは非常に一般的な方法であることが判明しました。 パラメータプロパティを使用すると、メンバを1か所に作成して初期化できます。
パラメータプロパティを使用して、以前の `Octopus` クラスをさらに改訂しました。

```typescript
class Octopus {
    readonly numberOfLegs: number = 8;
    constructor(readonly name: string) {
    }
}
```

どのように `theName` を削除したかに注目して、コンストラクタ上の短縮された `readonly name：string` パラメータを使用して、`name` メンバを作成して初期化してください。
宣言と割り当てを1つの場所にまとめました。

パラメータプロパティは、アクセシビリティ修飾子または読み取り専用のコンストラクタパラメータの接頭辞、またはその両方によって宣言されます。
パラメータプロパティに `private` を使用すると、`private` メンバーが宣言され、初期化されます。
同様に、`public`、`protected`、`readonly` でも同じことが行われます。


## Accessors アクセッサ

TypeScriptは、オブジェクトのメンバへのアクセスをインターセプトする方法として、`getters/setters` をサポートしています。
これにより、各オブジェクトでメンバーにアクセスする方法をきめ細かく制御できるようになります。

`get` と `set` を使うための単純なクラスを変換しましょう。
まず、ゲッタやセッタのない例を見てみましょう。

```typescript
class Employee {
    fullName: string;
}

let employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    console.log(employee.fullName);
}
```

`fullName` を無作為に直接設定できるようにするのはかなり便利ですが、人々が気まぐれに名前を変えることができれば、私たちは困ってしまうかもしれません。

このバージョンでは、従業員の変更を許可する前に、ユーザーに秘密のパスコードがあることを確認します。
これを行うには、`fullName` への直接アクセスをパスコードをチェックする `set` に置き換えます。
前の例がシームレスに動作し続けるように、対応する `get` を追加します。

```typescript
let passcode = "secret passcode";

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }

    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            console.log("Error: Unauthorized update of employee!");
        }
    }
}

let employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    console.log(employee.fullName);
}
```

アクセッサがパスコードを確認していることを証明するために、パスコードを変更して、一致しない場合には、従業員を更新するアクセス権がないことを警告するメッセージを表示します。

アクセッサに関する注意点。

まず、アクセサはECMAScript 5以降を出力するようにコンパイラを設定する必要があります。
ECMAScript3へのダウンレベリングはサポートされていません。
第2に、`get` を持ち `set`を持たないアクセッサは自動的に `readonly` であると推測されます。
これは、コードから `.d.ts` ファイルを生成するときに役立ちます。
なぜなら、プロパティのユーザーは変更できないことがわかるからです。

## Static Properties 静的プロパティ

ここまでは、クラスのインスタンスメンバーについてのみ説明しました。
インスタンスメンバーは、インスタンス化されたときにオブジェクトに表示されるものです。
また、インスタンスではなくクラス自体に表示されるクラスの `staic` メンバーを作成することもできます。
この例では、すべてのグリッドの一般的な値であるため、原点に対して `static` を使用します。
各インスタンスは、クラスの名前の前にこの値にアクセスします。
インスタンスアクセスの前に `this.` を前置するのと同様に、ここでは静的アクセスの前に `Grid.` を追加します。

```typescript
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        let xDist = (point.x - Grid.origin.x);
        let yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

let grid1 = new Grid(1.0);  // 1x scale
let grid2 = new Grid(5.0);  // 5x scale

console.log(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
console.log(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```

## Abstract Classes 抽象クラス

抽象クラスは、他のクラスを導出するための基本クラスです。
それらは直接インスタンス化されないかもしれません。
インタフェースとは異なり、抽象クラスにはメンバの実装の詳細が含まれることがあります。
`abstract` キーワードは、抽象クラスと、抽象クラス内の抽象メソッドを定義するために使用されます。

```typescript
abstract class Animal {
    abstract makeSound(): void;
    move(): void {
        console.log("roaming the earth...");
    }
}
```

`abstract` としてマークされた抽象クラス内のメソッドは、実装を含まず、派生クラスで実装する必要があります。
抽象メソッドは、インタフェースメソッドと同様の構文を共有します。
どちらも、メソッド本体を含まずにメソッドのシグネチャを定義します。
ただし、抽象メソッドには `abstract` キーワードを含める必要があり、オプションでアクセス修飾子を含めることができます。

```typescript
abstract class Department {

    constructor(public name: string) {
    }

    printName(): void {
        console.log("Department name: " + this.name);
    }

    abstract printMeeting(): void; // must be implemented in derived classes
}

class AccountingDepartment extends Department {

    constructor() {
        super("Accounting and Auditing"); // constructors in derived classes must call super()
    }

    printMeeting(): void {
        console.log("The Accounting Department meets each Monday at 10am.");
    }

    generateReports(): void {
        console.log("Generating accounting reports...");
    }
}

let department: Department; // ok to create a reference to an abstract type
department = new Department(); // error: cannot create an instance of an abstract class
department = new AccountingDepartment(); // ok to create and assign a non-abstract subclass
department.printName();
department.printMeeting();
department.generateReports(); // error: method doesn't exist on declared abstract type
```

## Advanced Techniques 高度なテクニック

### Constructor functions

TypeScriptでクラスを宣言すると、同時に複数の宣言が実際に作成されます。
最初のクラスは、クラスのインスタンスの型です。

```typescript
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter: Greeter;
greeter = new Greeter("world");
console.log(greeter.greet());
```

ここで、`let greeter: Greeter` と言うとき、クラスターのインスタンスの型として `Greeter` を使用しています。
これは、他のオブジェクト指向言語のプログラマーにとってはほとんど第2の性質です。

また、コンストラクタ関数と呼ばれる別の値も作成しています。
これは、クラスのインスタンスを新規に作成するときに呼び出される関数です。
実際にどのように見えるかを確認するには、上の例で作成したJavaScriptを見てみましょう。

```typescript
let Greeter = (function () {
    function Greeter(message) {
        this.greeting = message;
    }
    Greeter.prototype.greet = function () {
        return "Hello, " + this.greeting;
    };
    return Greeter;
})();

let greeter;
greeter = new Greeter("world");
console.log(greeter.greet());
```

ここでは、`Greeter` にコンストラクタ関数を割り当てることにします。
`new` を呼び出してこの関数を実行すると、そのクラスのインスタンスが取得されます。
コンストラクタ関数には、クラスのすべての静的メンバーも含まれます。
各クラスを考えるもう一つの方法は、インスタンス側と静的側があることです。

例を少し変更してこの違いを見てみましょう。

```typescript
class Greeter {
    static standardGreeting = "Hello, there";
    greeting: string;
    greet() {
        if (this.greeting) {
            return "Hello, " + this.greeting;
        }
        else {
            return Greeter.standardGreeting;
        }
    }
}

let greeter1: Greeter;
greeter1 = new Greeter();
console.log(greeter1.greet());

let greeterMaker: typeof Greeter = Greeter;
greeterMaker.standardGreeting = "Hey there!";

let greeter2: Greeter = new greeterMaker();
console.log(greeter2.greet());
```

この例では、`greeter1` は以前と同様に動作します。
`Greeter` クラスをインスタンス化し、このオブジェクトを使用します。 これまでに見たことがあります。

次に、クラスを直接使用します。 ここでは、`greeterMaker` という新しい変数を作成します。
この変数は、クラス自体を保持するか、別の方法でそのコンストラクタ関数を保持します。
ここでは `typeof Greeter` を使用します。つまり、インスタンスタイプではなく、「Greeterクラスそのもののタイプを教えてください」ということです。
または、より正確には、「`Greeter` というシンボルのタイプを与えてください。これはコンストラクタ関数の型です。
このタイプには、`Greeter` クラスのインスタンスを作成するコンストラクタとともに、`Greeter` のすべての静的メンバーが含まれます。
これは、`greeterMaker` の `new` を使用して、これを示します。
`Greeter` の新しいインスタンスを作成し、以前と同じように呼び出します。

### Using a class as an interface クラスをインタフェースとして使用する

前のセクションで述べたように、クラス宣言は、クラスのインスタンスを表す型とコンストラクタ関数の2つを作成します。
クラスは型を作成するので、インタフェースを使用できる同じ場所で型を使用できます。

```typescript
class Point {
    x: number;
    y: number;
}

interface Point3d extends Point {
    z: number;
}

let point3d: Point3d = {x: 1, y: 2, z: 3};
```