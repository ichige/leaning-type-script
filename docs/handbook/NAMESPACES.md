# Namespaces ネームスペース

>用語についての注意：TypeScript 1.5では、その命名法が変更されていることに注意することが重要です。
「内部モジュール」は 「ネームスペース」になりました。
ECMAScript 2015の用語に合わせて、「外部モジュール」は単に「モジュール」になっています（つまり、`module X {` は現在好ましい `namespace X {` と同等です）。


## Introduction 導入

この記事では、TypeScript の名前空間（以前は「内部モジュール」）を使用してコードを整理するさまざまな方法の概要を説明しました。
私たちが用語について書き留めたように、「内部モジュール」は現在「ネームスペース」と呼ばれています。
さらに、内部モジュールを宣言するときに `module` キーワードが使用された場所であれば、代わりに `namespace` キーワードを使用できます。
これにより、新しいユーザーを同様の名前の用語でオーバーロードすることで混乱を避けることができます。


## First steps はじめに

このページ全体を通して例として使用するプログラムから始めましょう。 
ウェブページのフォーム上のユーザーの入力をチェックしたり、外部から提供されたデータファイルの形式をチェックしたりするために、単純な文字列バリデータを作成しました。

### Validators in a single file

```typescript
interface StringValidator {
    isAcceptable(s: string): boolean;
}

let lettersRegexp = /^[A-Za-z]+$/;
let numberRegexp = /^[0-9]+$/;

class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}

class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: StringValidator; } = {};
validators["ZIP code"] = new ZipCodeValidator();
validators["Letters only"] = new LettersOnlyValidator();

// Show whether each string passed each validator
for (let s of strings) {
    for (let name in validators) {
        let isMatch = validators[name].isAcceptable(s);
        console.log(`'${ s }' ${ isMatch ? "matches" : "does not match" } '${ name }'.`);
    }
}
```

## Namespacing ネームスペース

より多くのバリデーターを追加するには、いくつかの種類の組織体系が必要になりますので、他のオブジェクトとの名前の衝突について心配することなく、型を追跡することができます。
グローバルな名前空間にたくさんの異なる名前を入れるのではなく、オブジェクトを名前空間にまとめましょう。

この例では、バリデーター関連のエンティティーをすべて `validation` と呼ばれるネームスペースに移動します。
私たちはここでインタフェースとクラスをネームスペースの外に見えるようにしたいので、それらのインターフェイスの前に `export` をします。
逆に、 `lettersRegexp` と `numberRegexp` という変数は実装の詳細なので、それらはアンエクスポートされており、ネームスペースの外側のコードには表示されません。
ファイルの一番下にあるテストコードでは、ネームスペース外で使用するときに型の名前を修飾する必要があります（例： `Validation.LettersOnlyValidator` ）。


### Namespaced Validators

```typescript
namespace Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }

    const lettersRegexp = /^[A-Za-z]+$/;
    const numberRegexp = /^[0-9]+$/;

    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }

    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: Validation.StringValidator; } = {};
validators["ZIP code"] = new Validation.ZipCodeValidator();
validators["Letters only"] = new Validation.LettersOnlyValidator();

// Show whether each string passed each validator
for (let s of strings) {
    for (let name in validators) {
        console.log(`"${ s }" - ${ validators[name].isAcceptable(s) ? "matches" : "does not match" } ${ name }`);
    }
}
```


## Splitting Across Files ファイル間の分割

アプリケーションの規模が拡大するにつれ、複数のファイルにコードを分割して管理しやすくする必要があります。

ここでは、 `Validation` のネームスペースを多数のファイルに分割します。
ファイルは別々のファイルでも、それぞれが同じネームスペースに寄与し、あたかも1つの場所にすべて定義されているかのように消費することができます。
ファイル間には依存関係があるため、ファイル間の関係についてコンパイラに指示する参照タグを追加します。
私たちのテストコードはそれ以外は変更されていません。

**Validation.ts**
```typescript
namespace Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }
}
```

**LettersOnlyValidator.ts**
```typescript
/// <reference path="Validation.ts" />
namespace Validation {
    const lettersRegexp = /^[A-Za-z]+$/;
    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }
}
```

**ZipCodeValidator.ts**
```typescript
/// <reference path="Validation.ts" />
namespace Validation {
    const numberRegexp = /^[0-9]+$/;
    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}
```

**Test.ts**
```typescript
/// <reference path="Validation.ts" />
/// <reference path="LettersOnlyValidator.ts" />
/// <reference path="ZipCodeValidator.ts" />

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: Validation.StringValidator; } = {};
validators["ZIP code"] = new Validation.ZipCodeValidator();
validators["Letters only"] = new Validation.LettersOnlyValidator();

// Show whether each string passed each validator
for (let s of strings) {
    for (let name in validators) {
        console.log(`"${ s }" - ${ validators[name].isAcceptable(s) ? "matches" : "does not match" } ${ name }`);
    }
}
```

複数のファイルが含まれると、コンパイルされたコードがすべて読み込まれるようにする必要があります。 
これを行うには2つの方法があります。

まず、 `--outFile` フラグを使用して連結出力を使用して、すべての入力ファイルを1つの JavaScript 出力ファイルにコンパイルします。

```bash
tsc --outFile sample.js Test.ts
```

コンパイラは、ファイルに存在する参照タグに基づいて出力ファイルを自動的に並べ替えます。 
各ファイルを個別に指定することもできます。

```bash
tsc --outFile sample.js Validation.ts LettersOnlyValidator.ts ZipCodeValidator.ts Test.ts
```

あるいは、ファイルごとのコンパイル（デフォルト）を使用して、入力ファイルごとに1つの JavaScript ファイルを出力することもできます。
複数のJSファイルが生成された場合は、Webページで `<script>` タグを使用して、出力された各ファイルを適切な順序でロードする必要があります。

**MyTestPage.html (excerpt)**
```html
    <script src="Validation.js" type="text/javascript" />
    <script src="LettersOnlyValidator.js" type="text/javascript" />
    <script src="ZipCodeValidator.js" type="text/javascript" />
    <script src="Test.js" type="text/javascript" />
```


## Aliases エイリアス

名前空間の操作を簡単にするもう1つの方法は、 `import q = x.y.z` を使用して、一般的に使用されているオブジェクトの短い名前を作成することです。
モジュールをロードするために使用される `import x = require("name")` 構文と混同しないでください。
この構文は、指定されたシンボルのエイリアスを作成するだけです。
これらの種類のインポート（通常はエイリアスと呼ばれます）を、モジュールのインポートから作成されたオブジェクトを含むあらゆる種類の識別子に使用できます。

```typescript
namespace Shapes {
    export namespace Polygons {
        export class Triangle { }
        export class Square { }
    }
}

import polygons = Shapes.Polygons;
let sq = new polygons.Square(); // Same as 'new Shapes.Polygons.Square()'
```

`require` キーワードは使用しないことに注意してください。 
代わりに、インポートするシンボルの修飾名から直接割り当てます。
これは `var` の使用に似ていますが、インポートされたシンボルの型と名前空間の意味にも作用します。 
重要なことに、値の場合、インポートは元のシンボルとは別の参照であるため、エイリアス化された変数への変更は元の変数に反映されません。


## Working with Other JavaScript Libraries 他のJavaScriptライブラリとの連携

TypeScriptで書かれていないライブラリの形状を記述するためには、ライブラリが公開するAPIを宣言する必要があります。
ほとんどのJavaScriptライブラリはトップレベルのオブジェクトをいくつか公開しているだけなので、名前空間はそれらを表現する良い方法です。

実装を「アンビエント」と定義しない宣言を呼び出します。
通常これらは `.d.ts` ファイルで定義されています。 C/C++ に精通している場合は、これらを `.h` ファイルと考えることができます。
いくつかの例を見てみましょう。


### Ambient Namespaces アンビエントネームスペース

一般的なライブラリ D3 は、その機能を `d3` というグローバルオブジェクトに定義しています。
このライブラリは（モジュールローダーではなく） `<script>` タグによってロードされるため、その宣言は名前空間を使用してその形状を定義します。
TypeScript コンパイラがこの形状を見るためには、アンビエント名前空間宣言を使用します。
たとえば、次のように記述することができます。

**D3.d.ts (simplified excerpt)**
```typescript
declare namespace D3 {
    export interface Selectors {
        select: {
            (selector: string): Selection;
            (element: EventTarget): Selection;
        };
    }

    export interface Event {
        x: number;
        y: number;
    }

    export interface Base extends Selectors {
        event: Event;
    }
}

declare var d3: D3.Base;
```




