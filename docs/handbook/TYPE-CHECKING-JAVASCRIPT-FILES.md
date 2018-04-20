# Type Checking JavaScript Files - Javascriptファイルの型チェック

TypeScript 2.3 以降では、 `-checkcks` を使用して、 `.js` ファイルの型チェックとエラー報告のモードをサポートしています。

いくつかのファイルをチェックするのをスキップするには、 `//@ts-nocheck` コメントを追加します。 
逆に、いくつかの `.js` ファイルだけをチェックするには、 `-checkJs` を設定せずに `//@ts-check` コメントを追加することができます。 
また、前の行に `//@ts-ignore` を追加することで、特定の行のエラーを無視することもできます。

`.ts` ファイルの `.js` ファイルでの作業の確認方法には、次のような違いがあります。


## Using types in JSDoc - JSDoc の型を使用する

`.js` ファイルでは、型は `.ts` ファイルの場合と同じように推論できます。 
同様に、型を推論できない場合、型の注釈が `.ts` ファイル内で行うのと同じ方法で JSDoc を使用して型を指定することができます。

宣言を飾る JSDoc アノテーションは、その宣言のタイプを設定するために使用されます。 
例えば…

```javascript
/** @type {number} */
var x;

x = 0;      // OK
x = false;  // Error: boolean is not assignable to number
```

サポートされているJSDocパターンの完全なリストは、[JavaScriptドキュメントの JSDoc サポート](https://github.com/Microsoft/TypeScript/wiki/JSDoc-support-in-JavaScript)で見つけることができます。


## Property declaration inferred from assignments in class bodies - クラス本体の割り当てから推測されるプロパティ宣言

ES2015/ES6 には、クラスのプロパティを宣言する手段がありません。 
プロパティは、オブジェクトリテラルの場合と同様に、動的に割り当てられます。

`.js` ファイルでは、プロパティ宣言はクラス本体内のプロパティへの割り当てから推測されます。 
プロパティのタイプは、これらの割り当て内のすべての右辺値のタイプの和集合です。 
コンストラクタで定義されたプロパティは、常に存在するとみなされます。
メソッド、ゲッタ、またはセッタで定義されているものはオプションとみなされます。

必要に応じてプロパティの型を指定するために JSDoc でプロパティの割り当てを飾ります。 
例えば…

```javascript
class C {
    constructor() {
        /** @type {number | undefined} */
        this.prop = undefined;
    }
}


let c = new C();
c.prop = 0;         // OK
c.prop = "string";  // Error: string is not assignable to number|undefined
```

プロパティがクラス本体に決して設定されていない場合は、未知のものとみなされます。 
クラスに読み込み専用のプロパティがある場合は、コンストラクタの初期化を `undefined` に追加することを検討してください。 
例えば `this.prop = undefined;`


## CommonJS module input support - CommonJS モジュールインプットのサポート

`.js` ファイルでは、CommonJS モジュール形式は入力モジュール形式として使用できます。 
`exports` への割り当て、および `module.exports` は、エクスポート宣言として認識されます。 
同様に、 `require` 関数呼び出しはモジュールのインポートとして認識されます。 
例えば…

```javascript
// import module "fs"
const fs = require("fs");


// export function readFile
module.exports.readFile = function(f) {
    return fs.readFileSync(f);
}
```


## Object literals are open-ended - オブジェクト・リテラルはオープンエンド

デフォルトでは、変数宣言のオブジェクトリテラルは、宣言の型を提供します。
元の初期化で指定されていない新規メンバーは追加できません。 
このルールは `.js` ファイルで緩和されています。 
オブジェクトリテラルは、オープンエンド型であり、元々定義されていなかったプロパティの追加と参照が可能です。 
例えば…

```javascript
var obj = { a: 1 };
obj.b = 2;  // Allowed
```

オブジェクトリテラルは、デフォルトのインデックスシグネチャ `[x：string]: any` を取得します。
これは、閉じたオブジェクトではなく開いたマップとして扱うことを許可します。

他の特殊な JS チェック動作と同様に、この動作は変数に JSDoc タイプを指定することで変更できます。 
例えば…

```javascript
/** @type  */
var obj = { a: 1 };
obj.b = 2;  // Error, type {a: number} does not have property b
```


## Function parameters are optional by default - 関数のパラメータはデフォルトでオプション

JS 内のパラメータにオプションを指定する方法はありません（デフォルト値を指定せずに） `.js` ファイル内のすべての関数パラメータはオプションと見なされます。 
引数の数が少ないコールが許可されます。

引数が多すぎる関数を呼び出すのはエラーであることに注意することが重要です。
例えば…

```javascript
function bar(a, b){
    console.log(a + " " + b);
}

bar(1);       // OK, second argument considered optional
bar(1, 2);
bar(1, 2, 3); // Error, too many arguments
```

JSDoc 注釈付き関数はこのルールから除外されます。 
オプション性を表現するには、JSDoc オプションのパラメータ構文を使用します。 
例えば…

```javascript
/**
 * @param {string} [somebody] - Somebody's name.
 */
function sayHello(somebody) {
    if (!somebody) {
        somebody = 'John Doe';
    }
    alert('Hello ' + somebody);
}

sayHello();
```

## Var-args parameter declaration inferred from use of `arguments` - `arguments` の使用から推測された Var-args パラメータ宣言

本体が引数参照を参照する関数は、暗黙的に var-arg パラメータ（つまり `(...arg: any[]) => any)` を持つとみなされます。 
JSDoc var-arg 構文を使用して、引数のタイプを指定します。


## Unspecified type parameters default to any - 指定されていない型パラメータのデフォルトはanyです

指定されていないジェネリック型パラメータのデフォルトは `any` です。 
これが起こる場所はほとんどありません。

### In extends clause - extends 句

例えば、 `React.Component` は、2つのジェネリック型のパラメータ、 `Props` と `State` を持つように定義されています。 
`.js` ファイルでは、 `extend` 句でこれらを指定する正当な方法はありません。 
デフォルトでは、型引数は `any` です。

```javascript
import { Component } from "react";

class MyComponent extends Component {
    render() {
       this.props.b; // Allowed, since this.props is of type any
    }
}
```

タイプを明示的に指定するには、JSDoc `@augments` を使用します。 
例えば…

```javascript
import { Component } from "react";

/**
 * @augments {Component<{a: number}, State>}
 */
class MyComponent extends Component {
    render() {
        this.props.b; // Error: b does not exist on {a:number}
    }
}
```


### In JSDoc references - JSDoc 参照

JSDoc の指定されていないジェネリック型引数のデフォルトは `any` です。

```javascript
/** @type{Array} */
var x = [];

x.push(1);        // OK
x.push("string"); // OK, x is of type Array<any>


/** @type{Array.<number>} */
var y = [];

y.push(1);        // OK
y.push("string"); // Error, string is not assignable to number
```


### In function calls - 関数呼び出し

ジェネリック関数の呼び出しは、ジェネリック型のパラメータを推論するために引数を使用します。 
時には、このプロセスは、主に推論の源がないために、どのような型も推測することができません。 
これらの場合、ジェネリック型パラメータはデフォルトで `any` になります。 
例えば…

```javascript
var p = new Promise((resolve, reject) => { reject() });

p; // Promise<any>;
```



