# Decorators - デコレータ

## Introduction - 導入

TypeScript と ES6 のクラスの導入により、クラスやクラスメンバの注釈付けや修正をサポートするための追加機能が必要なシナリオが存在するようになりました。
デコレータは、クラス宣言とメンバのアノテーションとメタプログラミング構文の両方を追加する方法を提供します。 
デコレータは JavaScript の[第2ステージの提案](https://github.com/tc39/proposal-decorators)であり、TypeScript の実験的な機能として利用できます。

> 注: デコレータは、将来のリリースで変更される実験的な機能です。

実験的なデコレータのサポートを有効にするには、コマンドラインまたは `tsconfig.json` の `experimentalDecorators` コンパイラオプションを有効にする必要があります。

**Command Line:**
```bash
tsc --target ES5 --experimentalDecorators
```

**tsconfig.json:**
```json
{
    "compilerOptions": {
        "target": "ES5",
        "experimentalDecorators": true
    }
}
```

## Decorators - デコレータとは

デコレータは、[クラス宣言](#class-decorators)、 [メソッド](#method-decorators)、[アクセサ](#accessor-decorators)、[プロパティ](#property-decorators)、または[パラメータ](#parameter-decorators)にアタッチできる特別な種類の宣言です。 
デコレータは、 `@expression` という形式を使用します。
ここで、 `expression` は、デコレーションされた宣言に関する情報とともに実行時に呼び出される関数に評価されなければなりません。

たとえば、デコレータ `@sealed` を指定すると、次のように `sealed` 関数を記述することができます。

```typescript
function sealed(target) {
    // do something with 'target' ...
}
```

> 注: 下の[クラスデコレータ](#class-decorators)で、デコレータのより詳細な例を見ることができます。


### Decorator Factories - デコレータ・ファクトリ

デコレータが宣言にどのように適用されるかをカスタマイズしたい場合は、デコレータファクトリを記述できます。 
デコレータファクトリは、実行時にデコレータによって呼び出される式を返す単なる関数です。

次のような方法でデコレータファクトリを書くことができます。

```typescript
function color(value: string) { // this is the decorator factory
    return function (target) { // this is the decorator
        // do something with 'target' and 'value'...
    }
}
```

> 注: デコレータファクトリのより詳細な例は、以下の[メソッドデコレータ](#method-decorators)で見ることができます。


### Decorator Composition - デコレータの構成

次の例のように、複数のデコレータを宣言に適用できます。

- 1行の場合

```typescript
@f @g x
```

- 複数行の場合

```typescript
@f
@g
x
```

複数のデコレータが1つの宣言に適用される場合、それらの評価は数学の関数構成と似ています。 
このモデルでは、関数 `f` と `g` を合成すると、得られた複合体 `(f ∘ g)(x) ` は `f(g(x))` と等価である。

そのため、TypeScript の1つの宣言で複数のデコレータを評価する場合、以下の手順が実行されます。

1. 各デコレータの式は上から下まで評価されます。
1. その結果は、下から上への関数として呼び出されます。

[デコレータファクトリ](#decorator-factories)を使用する場合は、次の例でこの評価オーダーを確認できます。

```typescript
function f() {
    console.log("f(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("f(): called");
    }
}

function g() {
    console.log("g(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("g(): called");
    }
}

class C {
    @f()
    @g()
    method() {}
}
```

この出力はコンソールに出力されます。

```
f(): evaluated
g(): evaluated
g(): called
f(): called
```


### Decorator Evaluation - デコレータの評価

クラス内のさまざまな宣言に適用されたデコレータがどのように適用されるかは、明確に定義されています。

1. パラメータ・デコレータ、メソッド、アクセサ、またはプロパティ・デコレータは、各インスタンス・メンバに適用されます。
1. パラメータ・デコレータ、メソッド、アクセサ、またはプロパティ・デコレータは、各スタティック・メンバに適用されます。
1. パラメータデコレータがコンストラクタに適用されます。
1. クラスのデコレータがクラスに適用されます。


### Class Decorators - クラス・デコレータ

クラスデコレータは、クラス宣言の直前に宣言されます。 
クラスデコレータは、クラスのコンストラクタに適用され、クラス定義の観察、変更、または置換に使用できます。 
クラスデコレータは、宣言ファイルや他の環境コンテキスト（ `declare` クラスなど）で使用することはできません。

クラスデコレータの式は実行時に関数として呼び出され、デコレートされたクラスのコンストラクタが唯一の引数として呼び出されます。

クラスデコレータが値を返す場合、クラス宣言は指定されたコンストラクタ関数に置き換えられます。

> 注意: 新しいコンストラクタ関数を返すことを選択した場合は、元のプロトタイプを維持するように注意する必要があります。 
実行時にデコレータを適用するロジックは、これを行うことはありません。

以下は、 `Greeter` クラスに適用されたクラスデコレータ（ `@sealed` ）の例です。

```typescript
@sealed
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}
```

`@sealed` デコレータは、次の関数宣言を使用して定義できます。

```typescript
function sealed(constructor: Function) {
    Object.seal(constructor);
    Object.seal(constructor.prototype);
}
```

`@sealed` を実行すると、コンストラクタとそのプロトタイプの両方がシールされます。

次に、コンストラクタをオーバーライドする方法の例を示します。

```typescript
function classDecorator<T extends {new(...args:any[]):{}}>(constructor:T) {
    return class extends constructor {
        newProperty = "new property";
        hello = "override";
    }
}

@classDecorator
class Greeter {
    property = "property";
    hello: string;
    constructor(m: string) {
        this.hello = m;
    }
}

console.log(new Greeter("world"));
```

### Method Decorators - メソッド・デコレータ

メソッドデコレータは、メソッド宣言の直前に宣言されます。 
デコレータは、メソッドのプロパティ記述子に適用され、メソッド定義を観察、変更、または置換するために使用できます。 
メソッドデコレータは、宣言ファイル、オーバーロード、またはその他の環境コンテキスト（ `declare` クラスなど）で使用することはできません。

メソッドデコレータの式は、実行時に次の3つの引数を指定して関数として呼び出されます。

1. 静的メンバーのクラスのコンストラクター関数、またはインスタンスメンバーのクラスのプロトタイプのいずれかです。
1. メンバーの名前。
1. メンバのプロパティ記述子。

> 注意: スクリプトのターゲットが `ES5` 未満の場合、プロパティ記述子は `undefined` です。

メソッドデコレータが値を返す場合は、メソッドのプロパティ記述子として使用されます。

> 注意: スクリプトのターゲットが `ES5` 未満の場合、戻り値は無視されます。

以下は、 `Greeter` クラスのメソッドに適用されるメソッドデコレータ（ `@enumerable` ）の例です。

```typescript
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }

    @enumerable(false)
    greet() {
        return "Hello, " + this.greeting;
    }
}
```

次の関数宣言を使用して `@enumerable` デコレータを定義できます。

```typescript
function enumerable(value: boolean) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        descriptor.enumerable = value;
    };
}
```

`@enumerable(false)` デコレータは[デコレータファクトリ](#decorator-factories)です。 
`@enumerable(false)` デコレータが呼び出されると、プロパティ記述子の `enumerable` プロパティが変更されます。


### Accessor Decorators - アクセッサ・デコレータ

アクセサーデコレータはアクセサー宣言の直前に宣言されます。 
アクセサーデコレータは、アクセサのプロパティ記述子に適用され、アクセサの定義を観察、変更、または置換するために使用できます。 
アクセサーデコレータは、宣言ファイルや他の環境コンテキスト（ `declare` クラスなど）で使用することはできません。

> 注記: TypeScript では、単一メンバーの `get` アクセサと `set` アクセサの両方を飾りません。 
代わりに、メンバーのすべてのデコレータをドキュメント順で指定された最初のアクセサに適用する必要があります。 
これは、デコレータがプロパティ記述子に適用されるためです。
これは、それぞれの宣言を個別に取得するのではなく、 `get` アクセサと `set` アクセサの両方を結合します。

アクセサーデコレータの式は、実行時に次の3つの引数を指定して関数として呼び出されます。

1. 静的メンバーのクラスのコンストラクター関数、またはインスタンスメンバーのクラスのプロトタイプのいずれかです。
1. メンバーの名前。
1. メンバのプロパティ記述子。

> 注意: スクリプトのターゲットが `ES5` 未満の場合、プロパティ記述子は未定義です。

アクセサーデコレータが値を返すと、それはメンバーのプロパティ記述子として使用されます。

> 注意: スクリプトのターゲットが `ES5` 未満の場合、戻り値は無視されます。

以下は、 `Point` クラスのメンバに適用されたアクセサーデコレータ（ `@configurable` ）の例です。

```typescript
class Point {
    private _x: number;
    private _y: number;
    constructor(x: number, y: number) {
        this._x = x;
        this._y = y;
    }

    @configurable(false)
    get x() { return this._x; }

    @configurable(false)
    get y() { return this._y; }
}
```

`@configurable` デコレータは、次の関数宣言を使用して定義できます。

```typescript
function configurable(value: boolean) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        descriptor.configurable = value;
    };
}
```


### Property Decorators - プロパティ・デコレータ

プロパティデコレータは、プロパティ宣言の直前に宣言されます。 
プロパティデコレータは、宣言ファイルや他の環境コンテキスト（ `declare` クラスなど）で使用することはできません。

プロパティデコレータの式は、実行時に次の2つの引数を持つ関数として呼び出されます。

1. 静的メンバーのクラスのコンストラクター関数、またはインスタンスメンバーのクラスのプロトタイプのいずれかです。
1. メンバーの名前。

> 注: プロパティデコレータは TypeScript でどのように初期化されるかによって、プロパティデコレータの引数として提供されません。 
これは現在、プロトタイプのメンバを定義するときにインスタンスプロパティを記述するメカニズムがなく、プロパティの初期化子を観察または変更する方法がないためです。 
戻り値も無視されます。 そのため、プロパティデコレータは、特定の名前のプロパティがクラスに対して宣言されたことを観察するためにのみ使用できます。

次の例のように、この情報を使用してプロパティに関するメタデータを記録できます。

```typescript
class Greeter {
    @format("Hello, %s")
    greeting: string;

    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        let formatString = getFormat(this, "greeting");
        return formatString.replace("%s", this.greeting);
    }
}
```

次の関数宣言を使用して、 `@format` デコレータ関数と `getFormat` 関数を定義できます。

```typescript
import "reflect-metadata";

const formatMetadataKey = Symbol("format");

function format(formatString: string) {
    return Reflect.metadata(formatMetadataKey, formatString);
}

function getFormat(target: any, propertyKey: string) {
    return Reflect.getMetadata(formatMetadataKey, target, propertyKey);
}
```

`@format("Hello、%s")` デコレータはデコレータファクトリです。 
`@format()"Hello、%s")` が呼び出されると、 `reflect-metadata` ライブラリの `Reflect.metadata` 関数を使用して、プロパティのメタデータエントリが追加されます。 
`getFormat` が呼び出されると、そのフォーマットのメタデータ値が読み込まれます。

> 注: この例では、 `reflect-metadata` ライブラリが必要です。 
`reflect-metadata` ライブラリの詳細については、[メタデータ](#metadata)を参照してください。


### Parameter Decorators - パラメータ・デコレータ

パラメータデコレータは、パラメータ宣言の直前に宣言されます。 
パラメータデコレータは、クラスコンストラクタまたはメソッド宣言の関数に適用されます。 
パラメータデコレータは、宣言ファイル、オーバーロード、またはその他の環境コンテキスト（ `declare` クラスなど）では使用できません。

パラメータデコレータの式は、実行時に次の3つの引数を指定して関数として呼び出されます。

1. 静的メンバーのクラスのコンストラクター関数、またはインスタンスメンバーのクラスのプロトタイプのいずれかです。
1. メンバーの名前。
1. 関数のパラメータリスト内のパラメータの順序インデックス。

> 注記: パラメータデコレータは、メソッド上でパラメータが宣言されたことを観察するためにのみ使用できます。

パラメータデコレータの戻り値は無視されます。

以下は、 `Greeter` クラスのメンバのパラメータに適用されるパラメータデコレータ（ `@required` ）の例です。

```typescript
class Greeter {
    greeting: string;

    constructor(message: string) {
        this.greeting = message;
    }

    @validate
    greet(@required name: string) {
        return "Hello " + name + ", " + this.greeting;
    }
}
```

次に、以下の関数宣言を使用して `@required` デコレータと `@validate` デコレータを定義できます。

```typescript
import "reflect-metadata";

const requiredMetadataKey = Symbol("required");

function required(target: Object, propertyKey: string | symbol, parameterIndex: number) {
    let existingRequiredParameters: number[] = Reflect.getOwnMetadata(requiredMetadataKey, target, propertyKey) || [];
    existingRequiredParameters.push(parameterIndex);
    Reflect.defineMetadata(requiredMetadataKey, existingRequiredParameters, target, propertyKey);
}

function validate(target: any, propertyName: string, descriptor: TypedPropertyDescriptor<Function>) {
    let method = descriptor.value;
    descriptor.value = function () {
        let requiredParameters: number[] = Reflect.getOwnMetadata(requiredMetadataKey, target, propertyName);
        if (requiredParameters) {
            for (let parameterIndex of requiredParameters) {
                if (parameterIndex >= arguments.length || arguments[parameterIndex] === undefined) {
                    throw new Error("Missing required argument.");
                }
            }
        }

        return method.apply(this, arguments);
    }
}
```

`@required` デコレータは、必要に応じてパラメータをマークするメタデータエントリを追加します。 
`@validate` デコレータは、元のメソッドを呼び出す前に引数を検証する関数に既存の `greet` メソッドをラップします。

> 注: この例では、 `reflect-metadata` ライブラリが必要です。 `reflect-metadata` ライブラリの詳細については、[メタデータ](#metadata)を参照してください。


### Metadata - メタデータ

いくつかの例では、[実験的メタデータAPI](https://github.com/rbuckton/ReflectDecorators)に polyfill を追加する `reflect-metadata` ライブラリを使用しています。 
このライブラリは、ECMAScript（JavaScript）標準の一部ではありません。 
しかし、いったんデコレータが ECMAScript 標準の一部として公式に採用されると、これらの拡張は採用のために提案されるでしょう。

このライブラリは npm でインストールできます。

```bash
npm i reflect-metadata --save
```

TypeScript には、デコレータを持つ宣言に対して特定のタイプのメタデータを出力する実験的なサポートが含まれています。 
この実験的サポートを有効にするには、 `emitDecoratorMetadata` コンパイラオプションをコマンドラインまたは `tsconfig.json` のいずれかに設定する必要があります。

**Command Line:**
```bash
tsc --target ES5 --experimentalDecorators --emitDecoratorMetadata
```

**tsconfig.json:**
```json
{
    "compilerOptions": {
        "target": "ES5",
        "experimentalDecorators": true,
        "emitDecoratorMetadata": true
    }
}
```

有効にすると、`reflect-metadata` ライブラリがインポートされている限り、実行時に追加のデザイン時型情報が公開されます。

次の例で実際にこれを見ることができます。

```typescript
import "reflect-metadata";

class Point {
    x: number;
    y: number;
}

class Line {
    private _p0: Point;
    private _p1: Point;

    @validate
    set p0(value: Point) { this._p0 = value; }
    get p0() { return this._p0; }

    @validate
    set p1(value: Point) { this._p1 = value; }
    get p1() { return this._p1; }
}

function validate<T>(target: any, propertyKey: string, descriptor: TypedPropertyDescriptor<T>) {
    let set = descriptor.set;
    descriptor.set = function (value: T) {
        let type = Reflect.getMetadata("design:type", target, propertyKey);
        if (!(value instanceof type)) {
            throw new TypeError("Invalid type.");
        }
        set(value);
    }
}
```

TypeScript コンパイラは、 `@Reflect.metadata` デコレータを使用してデザイン時型情報を注入します。 
あなたはそれを次の TypeScript に相当すると考えることができます。

```typescript
class Line {
    private _p0: Point;
    private _p1: Point;

    @validate
    @Reflect.metadata("design:type", Point)
    set p0(value: Point) { this._p0 = value; }
    get p0() { return this._p0; }

    @validate
    @Reflect.metadata("design:type", Point)
    set p1(value: Point) { this._p1 = value; }
    get p1() { return this._p1; }
}
```

> 注: デコレータのメタデータは実験的な機能であり、将来のリリースで大きな変更を導入する可能性があります。
