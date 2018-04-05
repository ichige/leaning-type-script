# MODULES モジュール

> 用語についての注意：TypeScript 1.5では、その命名法が変更されていることに注意することが重要です。
 「インターナル・モジュール」は「名前空間」になりました。
 ECMAScript 2015の用語に合わせて、「エクスターナル・モジュール」は単に「モジュール」になっています（つまり、`module X {` は現在では `namespace X {` と同等です）。
 
 
 ## Introduction はじめに
 
 ECMAScript 2015以降、JavaScript にはモジュールという概念があります。 
 TypeScriptはこの概念を共有しています。
 
 モジュールは、グローバルスコープではなく、独自のスコープ内で実行されます。 
 これは、変数、関数、クラスなどを意味します。
 モジュール内で宣言されているものは、[`export` form](#export-form) の1つを使用して明示的にエクスポートされていない限り、モジュールの外側には表示されません。
 逆に、変数、関数、クラス、インタフェースなどを消費します。
 別のモジュールからエクスポートされた場合、[`import` form](#inport-form)の1つを使用してインポートする必要があります。
 
 モジュールは宣言的です。 モジュール間の関係は、ファイルレベルでのインポートとエクスポートの観点から指定されます。
 
 モジュールは、モジュールローダを使用して相互にインポートします。
 実行時に、モジュールローダは、モジュールを実行する前にモジュールのすべての依存関係を特定し実行する責任があります。
 JavaScriptでよく使われるモジュールローダーは、Node.js の [CommonJS](https://en.wikipedia.org/wiki/CommonJS) モジュールローダーとWebアプリケーションの [require.js](http://requirejs.org/) です。
 
 TypeScript では、ECMAScript 2015 と同様に、トップレベルの `import` または `export` を含むファイルはすべてモジュールと見なされます。
 逆に、トップレベルの `import` または `export` 宣言がないファイルは、その内容がグローバルスコープで（したがってモジュールにも）利用可能なスクリプトとして扱われます。
 

##  Export エクスポート

### Exporting a declaration エクスポートの宣言

`export` キーワードを追加すると、宣言（変数、関数、クラス、エイリアス、またはインタフェースなど）をすべてエクスポートできます。

**Validation.ts**
```typescript
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

**ZipCodeValidator.ts**
```typescript
export const numberRegexp = /^[0-9]+$/;

export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
```


### Export statements エクスポート・ステートメント

エクスポート・ステートメントは、コンシューマ向けにエクスポートの名前を変更する必要がある場合に便利です。
したがって、上記の例は次のように記述できます。

```typescript
class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export { ZipCodeValidator };
export { ZipCodeValidator as mainValidator };
```


### Re-exports 再エクスポート

多くの場合、モジュールは他のモジュールを拡張し、その一部の機能を部分的に公開します。 
再エクスポートでは、ローカルにインポートしたり、ローカル変数をインポートしたりしません。

**ParseIntBasedZipCodeValidator.ts**

```typescript
export class ParseIntBasedZipCodeValidator {
    isAcceptable(s: string) {
        return s.length === 5 && parseInt(s).toString() === s;
    }
}

// Export original validator but rename it
export {ZipCodeValidator as RegExpBasedZipCodeValidator} from "./ZipCodeValidator";
```

必要に応じて、モジュールは1つまたは複数のモジュールをラップし、 `export * from "module"` 構文を使用してすべてのエクスポートを組み合わせることができます。

**AllValidators.ts**
```typescript
export * from "./StringValidator"; // exports interface 'StringValidator'
export * from "./LettersOnlyValidator"; // exports class 'LettersOnlyValidator'
export * from "./ZipCodeValidator";  // exports class 'ZipCodeValidator'
```


## Import インポート

インポートは、モジュールからエクスポートするよりも簡単です。 
エクスポートされた宣言のインポートは、以下の `import` フォームのいずれかを使用して行います。


### Import a single export from a module モジュールから1つのエクスポートをインポートする

```typescript
import { ZipCodeValidator } from "./ZipCodeValidator";

let myValidator = new ZipCodeValidator();
```

インポートの名前を変更することもできます。

```typescript
import { ZipCodeValidator as ZCV } from "./ZipCodeValidator";
let myValidator = new ZCV();
```


### Import the entire module into a single variable, and use it to access the module exports モジュール全体を1つの変数にインポートし、それを使用してモジュールのエクスポートにアクセスする

```typescript
import * as validator from "./ZipCodeValidator";
let myValidator = new validator.ZipCodeValidator();
```


### Import a module for side-effects only 副作用のみのモジュールをインポートする

推奨される方法ではありませんが、一部のモジュールは他のモジュールで使用できるグローバル状態を設定します。 
これらのモジュールにはエクスポートがないか、利用側ではエクスポートに関心がありません。 
これらのモジュールをインポートするには、以下を使用します。

```typescript
import "./my-module.js";
```


## Default exports デフォルトのエクスポート

各モジュールは、オプションで `default` のエクスポートをエクスポートできます。
デフォルトのエクスポートはキーワード `default` でマークされます。 モジュールごとに1つの `default` エクスポートしか存在できません。
`default` のエクスポートは、別のインポート形式を使用してインポートされます。

`default` のエクスポートは本当に便利です。
たとえば、JQuery のようなライブラリでは、デフォルトの `jQuery` または `$` のエクスポートがあります。
`$` や `jQuery` という名前でインポートします。

**JQuery.d.ts**
```typescript
declare let $: JQuery;
export default $;
```

**App.ts**
```typescript
import $ from "JQuery";

$("button.continue").html( "Next Step..." );
```

クラスと関数の宣言は、デフォルトのエクスポートとして直接作成することができます。 
デフォルトのエクスポートクラス名と関数宣言名はオプションです。

**ZipCodeValidator.ts**
```typescript
export default class ZipCodeValidator {
    static numberRegexp = /^[0-9]+$/;
    isAcceptable(s: string) {
        return s.length === 5 && ZipCodeValidator.numberRegexp.test(s);
    }
}
```

**Test.ts**
```typescript
import validator from "./ZipCodeValidator";

let myValidator = new validator();
```

または…

**StaticZipCodeValidator.ts**
```typescript
const numberRegexp = /^[0-9]+$/;

export default function (s: string) {
    return s.length === 5 && numberRegexp.test(s);
}
```

**Test.ts**
```typescript
import validate from "./StaticZipCodeValidator";

let strings = ["Hello", "98052", "101"];

// Use function validate
strings.forEach(s => {
  console.log(`"${s}" ${validate(s) ? " matches" : " does not match"}`);
});
```

`default` のエクスポートは単なる値にすることもできます。

**OneTwoThree.ts**
```typescript
export default "123";
```

**Log.ts**
```typescript
import num from "./OneTwoThree";

console.log(num); // "123"
```


## `export =` and `import = require()` `export =` と `import = require()`

CommonJS と AMD の両方には、一般に、モジュールからのすべてのエクスポートを含む `exports` オブジェクトという概念があります。

また、 `exports` オブジェクトをカスタム単一オブジェクトで置き換えることもサポートしています。
デフォルトのエクスポートは、この動作の代用として機能することを意図しています。 しかし、両者は互換性がありません。
TypeScript は、従来の CommonJS および AMD ワークフローをモデル化するために `export =` をサポートしています。

`export =` 構文は、モジュールからエクスポートされた単一のオブジェクトを指定します。 
これは、クラス、インタフェース、名前空間、関数、または列挙型にすることができます。

`export =` を使用してモジュールをエクスポートする場合、モジュールをインポートするには、TypeScrip t固有の `import module = require("module")` を使用する必要があります。

**ZipCodeValidator.ts**
```typescript
let numberRegexp = /^[0-9]+$/;
class ZipCodeValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export = ZipCodeValidator;
```

**Test.ts**
```typescript
import zip = require("./ZipCodeValidator");

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validator = new zip();

// Show whether each string passed each validator
strings.forEach(s => {
  console.log(`"${ s }" - ${ validator.isAcceptable(s) ? "matches" : "does not match" }`);
});
```


## Code Generation for Modules モジュールのコード生成

コンパイル時に指定されたモジュールターゲットに応じて、コンパイラは Node.js（[CommonJS](http://wiki.commonjs.org/wiki/CommonJS)）、require.js（[AMD](https://github.com/amdjs/amdjs-api/wiki/AMD)）、[UMD](https://github.com/umdjs/umd)、[SystemJS](https://github.com/systemjs/systemjs)、または [ECMAScript 2015ネイティブモジュール（ES6）](http://www.ecma-international.org/ecma-262/6.0/#sec-modules)モジュールロードシステムに適切なコードを生成します。
生成されたコードの `defined`、 `require`、および `register` の詳細については、各モジュールローダーのドキュメントを参照してください。

この単純な例は、インポートおよびエクスポート中に使用された名前が、モジュールをロードするコードにどのように変換されるかを示しています。

**SimpleModule.ts**

```javascript
import m = require("mod");
export let t = m.something + 1;
```

**AMD / RequireJS SimpleModule.js**

```javascript
define(["require", "exports", "./mod"], function (require, exports, mod_1) {
    exports.t = mod_1.something + 1;
});
```

**CommonJS / Node SimpleModule.js**

```javascript
var mod_1 = require("./mod");
exports.t = mod_1.something + 1;
```

**UMD SimpleModule.js**

```javascript
(function (factory) {
    if (typeof module === "object" && typeof module.exports === "object") {
        var v = factory(require, exports); if (v !== undefined) module.exports = v;
    }
    else if (typeof define === "function" && define.amd) {
        define(["require", "exports", "./mod"], factory);
    }
})(function (require, exports) {
    var mod_1 = require("./mod");
    exports.t = mod_1.something + 1;
});
```

**System SimpleModule.js**

```javascript
System.register(["./mod"], function(exports_1) {
    var mod_1;
    var t;
    return {
        setters:[
            function (mod_1_1) {
                mod_1 = mod_1_1;
            }],
        execute: function() {
            exports_1("t", t = mod_1.something + 1);
        }
    }
});
```

**Native ECMAScript 2015 modules SimpleModule.js**

```javascript
import { something } from "./mod";
export var t = something + 1;
```


## Simple Example

以下では、前の例で使用した Validator の実装を統合して、各モジュールから単一の名前付きエクスポートをエクスポートしました。

コンパイルするには、コマンドラインでモジュールターゲットを指定する必要があります。 
Node.js には、 `--module commonjs` を使用します。  require.js には `--module amd` を使います。 例えば…

```bash
tsc --module commonjs Test.ts
```

コンパイルすると、各モジュールは別の `.js` ファイルになります。
参照タグと同様に、コンパイラーは `import` ステートメントに従って従属ファイルをコンパイルします。

**Validation.ts**

```typescript
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

**LettersOnlyValidator.ts**

```typescript
import { StringValidator } from "./Validation";

const lettersRegexp = /^[A-Za-z]+$/;

export class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}
```

**ZipCodeValidator.ts**

```typescript
import { StringValidator } from "./Validation";

const numberRegexp = /^[0-9]+$/;

export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
```

**Test.ts**

```typescript
import { StringValidator } from "./Validation";
import { ZipCodeValidator } from "./ZipCodeValidator";
import { LettersOnlyValidator } from "./LettersOnlyValidator";

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: StringValidator; } = {};
validators["ZIP code"] = new ZipCodeValidator();
validators["Letters only"] = new LettersOnlyValidator();

// Show whether each string passed each validator
strings.forEach(s => {
    for (let name in validators) {
        console.log(`"${ s }" - ${ validators[name].isAcceptable(s) ? "matches" : "does not match" } ${ name }`);
    }
});
```


## Optional Module Loading and Other Advanced Loading Scenarios オプションのモジュール読み込みおよびその他の高度な読み込みシナリオ

場合によっては、いくつかの条件の下でモジュールをロードするだけの場合があります。
TypeScript では、以下に示すパターンを使用して、タイプの安全性を失うことなく、モジュールローダを直接呼び出すための高度なロードシナリオを実装できます。

コンパイラは、各モジュールが発行された JavaScript で使用されているかどうかを検出します。
モジュール識別子がタイプ注釈の一部としてのみ使用され、決して式として使用されない場合、そのモジュールに対して `require` 呼び出しは発行されません。
この未使用参照の削除は、パフォーマンスの最適な最適化であり、これらのモジュールのオプションのロードも可能です。

パターンのコアアイデアは、`import id = require("...")`ステートメントが、モジュールによって公開される型へのアクセスを提供するということです。
以下の `if` ブロックに示すように、モジュールローダーは（ `require` を介して）動的に呼び出されます。
これは、基準溶出最適化を利用して、モジュールが必要なときにのみロードされるようにします。
このパターンを機能させるには、 `import` 経由で定義されたシンボルが型の位置（JavaScriptに送出される位置に決して置かれない）でのみ使用されることが重要です。

型の安全性を維持するために、 `typeof` キーワードを使用できます。
`typeof` キーワードをタイプ位置で使用すると、値のタイプ（この場合はモジュールのタイプ）が生成されます。

**Dynamic Module Loading in Node.js**

```typescript
declare function require(moduleName: string): any;

import { ZipCodeValidator as Zip } from "./ZipCodeValidator";

if (needZipValidation) {
    let ZipCodeValidator: typeof Zip = require("./ZipCodeValidator");
    let validator = new ZipCodeValidator();
    if (validator.isAcceptable("...")) { /* ... */ }
}
```

**Sample: Dynamic Module Loading in require.js**

```typescript
declare function require(moduleNames: string[], onLoad: (...args: any[]) => void): void;

import * as Zip from "./ZipCodeValidator";

if (needZipValidation) {
    require(["./ZipCodeValidator"], (ZipCodeValidator: typeof Zip) => {
        let validator = new ZipCodeValidator.ZipCodeValidator();
        if (validator.isAcceptable("...")) { /* ... */ }
    });
}
```

**Sample: Dynamic Module Loading in System.js**

```typescript
declare const System: any;

import { ZipCodeValidator as Zip } from "./ZipCodeValidator";

if (needZipValidation) {
    System.import("./ZipCodeValidator").then((ZipCodeValidator: typeof Zip) => {
        var x = new ZipCodeValidator();
        if (x.isAcceptable("...")) { /* ... */ }
    });
}
```


## Working with Other JavaScript Libraries 他の JavaScript ライブラリとの連携

TypeScriptで書かれていないライブラリの形状を記述するためには、ライブラリが公開するAPIを宣言する必要があります。

実装を「ambient」と定義しない宣言を呼び出します。
通常、これらは. `d.ts` ファイルで定義されています。
C/C++ に精通している場合は、これらを `.h` ファイルと考えることができます。 いくつかの例を見てみましょう。


### Ambient Modules アンビエントモジュール

Node.js では、ほとんどのタスクは1つまたは複数のモジュールをロードすることによって実行されます。
最上位のエクスポート宣言を持つ独自の `.d.ts` ファイルで各モジュールを定義できますが、大きな `.d.ts` ファイルとして記述する方が便利です。
そうするためには、アンビエントネームスペースに似た構造を使用しますが、後でインポートできるモジュールのキーワードと引用されたモジュールの名前を使用します。 
例えば…

**node.d.ts (simplified excerpt)**

```typescript
declare module "url" {
    export interface Url {
        protocol?: string;
        hostname?: string;
        pathname?: string;
    }

    export function parse(urlStr: string, parseQueryString?, slashesDenoteHost?): Url;
}

declare module "path" {
    export function normalize(p: string): string;
    export function join(...paths: any[]): string;
    export var sep: string;
}
```

これで、 `/// <ref-erence> node.d.ts` を読み込み、 `import url = require("url");` を使用してモジュールをロードできます。 
または `im-port * as URL from "url"` 。

```typescript
/// <reference path="node.d.ts"/>
import * as URL from "url";
let myUrl = URL.parse("http://www.typescriptlang.org");
```


### Shorthand ambient modules 短縮型アンビエントモジュール

新しいモジュールを使用する前に宣言を書く時間を取らないようにするには、簡単な宣言を使って素早く始めることができます。

**declarations.d.ts**

```typescript
declare module "hot-new-module";
```

省略形モジュールからのすべてのインポートは、`any` 型を持ちます。

```typescript
import x, {y} from "hot-new-module";
x(y);
```


### Wildcard module declarations ワイルドカードモジュール宣言

SystemJS や AMD などの一部のモジュールローダーでは、JavaScript 以外のコンテンツをインポートすることができます。
これらは通常、特殊な読み込みセマンティクスを示すために接頭辞または接尾辞を使用します。
ワイルドカードモジュールの宣言は、これらのケースをカバーするために使用できます。

```typescript
declare module "*!text" {
    const content: string;
    export default content;
}
// Some do it the other way around.
declare module "json!*" {
    const value: any;
    export default value;
}
```

これで、 `"*！text"` または `"json！*"` と一致するものをインポートできます。

```typescript
import fileContent from "./xyz.txt!text";
import data from "json!http://example.com/data.json";
console.log(data, fileContent);
```


### UMD modules UMDモジュール

一部のライブラリは、多くのモジュールローダで使用するように設計されているか、モジュールをロードしていない（グローバル変数）ように設計されています。
これらは [UMD](https://github.com/umdjs/umd) モジュールとして知られています。
これらのライブラリには、インポート変数またはグローバル変数のいずれかを使用してアクセスできます。
例えば…

**math-lib.d.ts**

```typescript
export function isPrime(x: number): boolean;
export as namespace mathLib;
```

ライブラリは、モジュール内でのインポートとして使用できます。

```typescript
import { isPrime } from "math-lib";
isPrime(2);
mathLib.isPrime(2); // ERROR: can't use the global definition from inside a module
```

グローバル変数としても使用できますが、スクリプト内でのみ使用できます。 
（スクリプトは、インポートまたはエクスポートがないファイルです）。

```typescript
mathLib.isPrime(2);
```


## Guidance for structuring modules 構造化モジュールのガイダンス

### Export as close to top-level as possible できるだけ最上位に近いレベルでエクスポートする

モジュールの消費者は、輸出したものを使うときは、できるだけ摩擦を少なくするべきです。
あまりにも多くのレベルの入れ子を追加するのは面倒な傾向があるので、物事をどのように構造化したいかを慎重に考えてください。

モジュールからネームスペースをエクスポートすることは、ネストのレイヤーをあまりにも多く追加することの例です。
名前空間にはいつか使用されることがありますが、モジュールを使用するときには余分なレベルの間接参照が追加されます。
これは、ユーザーにとってはすぐに苦労することになり、通常は不要です。

エクスポートされたクラスの静的メソッドにも同様の問題があります。クラス自体は入れ子のレイヤーを追加します。
表現力や意図が明らかに有用な形で増加しない限り、単純にヘルパー関数をエクスポートすることを検討してください。

**1つの `class` または `function` だけをエクスポートする場合は、 `export default` を使用します**

「トップレベル付近でのエクスポート」がモジュールのコンシューマの摩擦を減らすのと同様に、デフォルトのエクスポートを導入します。
モジュールの主な目的が1つの特定のエクスポートを格納することである場合、それをデフォルトのエクスポートとしてエクスポートすることを検討する必要があります。
これにより、インポートと実際にインポートを少しでも簡単に行うことができます。 例えば…

**MyClass.ts**

```typescript
export default class SomeType {
  constructor() { ... }
}
```

**MyFunc.ts**

```typescript
export default function getThing() { return "thing"; }
```

**Consumer.ts**

```typescript
import t from "./MyClass";
import f from "./MyFunc";
let x = new t();
console.log(f());
```

これは消費者にとって最適です。 
彼らはあなたのタイプに名前をつけることができます（この場合は `t` ）、あなたのオブジェクトを見つけるために余分なドットを入れる必要はありません。


### If you’re exporting multiple objects, put them all at top-level 複数のオブジェクトをエクスポートする場合は、それらをすべて最上位に配置します

**MyThings.ts**

```typescript
export class SomeType { /* ... */ }
export function someFunc() { /* ... */ }
```

逆にインポートする場合…


### Explicitly list imported names インポートされた名前を明示的に表示する

**Consumer.ts**

```typescript
import { SomeType, someFunc } from "./MyThings";
let x = new SomeType();
let y = someFunc();
```


### Use the namespace import pattern if you’re importing a large number of things 多数のものをインポートする場合は、名前空間のインポートパターンを使用します

**MyLargeModule.ts**

```typescript
export class Dog { ... }
export class Cat { ... }
export class Tree { ... }
export class Flower { ... }
```

**Consumer.ts**

```typescript
import * as myLargeModule from "./MyLargeModule.ts";
let x = new myLargeModule.Dog();
```
 
### Re-export to extend 再エクスポートして拡張する

多くの場合、モジュール上で機能を拡張する必要があります。
一般的な JS パターンは、JQuery 拡張機能の動作と同様に、元のオブジェクトを拡張することです。
前述したように、モジュールはグローバル名前空間オブジェクトのようにマージしません。
推奨される解決策は、元のオブジェクトを変更するのではなく、新しい機能を提供する新しいエンティティをエクスポートすることです。

`Calculator.ts` モジュールで定義されている簡単な電卓実装を考えてみましょう。 
また、モジュールは、入力文字列のリストを渡して最後に結果を書き込むことによって、電卓機能をテストするヘルパー関数をエクスポートします。

**Calculator.ts**

```typescript
export class Calculator {
    private current = 0;
    private memory = 0;
    private operator: string;

    protected processDigit(digit: string, currentValue: number) {
        if (digit >= "0" && digit <= "9") {
            return currentValue * 10 + (digit.charCodeAt(0) - "0".charCodeAt(0));
        }
    }

    protected processOperator(operator: string) {
        if (["+", "-", "*", "/"].indexOf(operator) >= 0) {
            return operator;
        }
    }

    protected evaluateOperator(operator: string, left: number, right: number): number {
        switch (this.operator) {
            case "+": return left + right;
            case "-": return left - right;
            case "*": return left * right;
            case "/": return left / right;
        }
    }

    private evaluate() {
        if (this.operator) {
            this.memory = this.evaluateOperator(this.operator, this.memory, this.current);
        }
        else {
            this.memory = this.current;
        }
        this.current = 0;
    }

    public handelChar(char: string) {
        if (char === "=") {
            this.evaluate();
            return;
        }
        else {
            let value = this.processDigit(char, this.current);
            if (value !== undefined) {
                this.current = value;
                return;
            }
            else {
                let value = this.processOperator(char);
                if (value !== undefined) {
                    this.evaluate();
                    this.operator = value;
                    return;
                }
            }
        }
        throw new Error(`Unsupported input: '${char}'`);
    }

    public getResult() {
        return this.memory;
    }
}

export function test(c: Calculator, input: string) {
    for (let i = 0; i < input.length; i++) {
        c.handelChar(input[i]);
    }

    console.log(`result of '${input}' is '${c.getResult()}'`);
}
```

露出されたテスト機能を使用した電卓の簡単なテストです。

**TestCalculator.ts**

```typescript
import { Calculator, test } from "./Calculator";


let c = new Calculator();
test(c, "1+2*33/11="); // prints 9
```

これを拡張して、10以外の数字で入力をサポートするようにするには、 `ProgrammerCalculator.ts` を作成しましょう。

**ProgrammerCalculator.ts**

```typescript
import { Calculator } from "./Calculator";

class ProgrammerCalculator extends Calculator {
    static digits = ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "A", "B", "C", "D", "E", "F"];

    constructor(public base: number) {
        super();
        if (base <= 0 || base > ProgrammerCalculator.digits.length) {
            throw new Error("base has to be within 0 to 16 inclusive.");
        }
    }

    protected processDigit(digit: string, currentValue: number) {
        if (ProgrammerCalculator.digits.indexOf(digit) >= 0) {
            return currentValue * this.base + ProgrammerCalculator.digits.indexOf(digit);
        }
    }
}

// Export the new extended calculator as Calculator
export { ProgrammerCalculator as Calculator };

// Also, export the helper function
export { test } from "./Calculator";
```

新しいモジュール `ProgrammerCalculator` は、元の `Calculator` モジュールのAPIシェイプに似たAPIシェイプをエクスポートしますが、元のモジュール内のオブジェクトを増やすことはありません。 
`ProgrammerCalculator` クラスのテストは次のとおりです。

**TestProgrammerCalculator.ts**

```typescript
import { Calculator, test } from "./ProgrammerCalculator";

let c = new Calculator(2);
test(c, "001+010="); // prints 3
```


### Do not use namespaces in modules モジュール内で名前空間を使用しない

モジュールベースの組織に最初に移行するとき、一般的な傾向は、名前空間の追加層にエクスポートをラップすることです。
モジュールは独自のスコープを持ち、エクスポートされた宣言だけがモジュール外部から見えます。
このことを念頭に置いて、ネームスペースは、モジュールを扱う際の価値はほとんどありません。

組織のフロントでは、グローバルスコープ内の論理的に関連するオブジェクトとタイプをグループ化するのに便利です。
たとえば、C＃ では、System.Collections 内のすべてのコレクション型を検索します。
私たちのタイプを階層的な名前空間に編成することにより、我々はそれらのタイプのユーザにとって良い "発見"経験を提供します。
一方、モジュールは、ファイルシステムにはすでに存在しています。
パスとファイル名で解決する必要があるため、私たちが使用する論理的な体系があります。
リストモジュールを持つ /collections/generic/folder を持つことができます。

名前空間は、グローバルスコープでの名前の衝突を避けるために重要です。
たとえば、 `My.Application.Customer.AddForm` と `My.Application.Order.AddForm` があります。
名前は同じで名前空間が異なる2つのタイプがあります。
しかし、これはモジュールの問題ではありません。
モジュール内には、同じ名前の2つのオブジェクトを持つもっともらしい理由はありません。
消費側からは、任意のモジュールの消費者がモジュールを参照するために使用する名前を選択するため、偶発的な名前の競合は不可能です。

>モジュールと名前空間の詳細については、[「名前空間とモジュール」](NAMESPACES-AND-MODULES.md)を参照してください。


### Red Flags レッドフラッグ

以下はモジュール構造化のためのレッドフラッグです。 
ファイルにこれらのいずれかが当てはまる場合は、外部モジュールに名前を付けようとしていないことを再度確認してください。

- トップレベルの宣言だけが `export namespace Foo { ... }`（ `Foo` を削除し、レベル全てを '上に移動'）するファイル
- 1つの `export class` または `export function` を持つファイル（ `export default` を使用することを検討してください）
- 同一の `export namespace {` トップレベルにある複数のファイル（これらが1つの `Foo` に結合されるとは思わない）
