# Symbols シンボル

## はじめに

ECMAScript 2015から始まる `symbol` は、`number` と `string` のような基本的なデータ型です。

`symbol` 値は `Symbol` コンストラクタを呼び出して作成されます。

```typescript
let sym1 = Symbol();

let sym2 = Symbol("key"); // optional string key
```

シンボルは不変でユニークです。

```typescript
let sym2 = Symbol("key");
let sym3 = Symbol("key");

sym2 === sym3; // false, symbols are unique
```

文字列と同様に、シンボルはオブジェクトプロパティのキーとして使用できます。

```typescript
let sym = Symbol();

let obj = {
    [sym]: "value"
};

console.log(obj[sym]); // "value"
```

シンボルを計算プロパティ宣言と組み合わせて、オブジェクトプロパティとクラスメンバを宣言することもできます。

```typescript
const getClassNameSymbol = Symbol();

class C {
    [getClassNameSymbol](){
       return "C";
    }
}

let c = new C();
let className = c[getClassNameSymbol](); // "C"
```

## Well-known Symbols 良く知られているシンボル

ユーザ定義シンボルに加えて、よく知られている組み込みシンボルがあります。
組み込みのシンボルは、内部言語の動作を表すために使用されます。

よく知られているシンボルのリストを以下に示します。

### `Symbol.hasInstance`

コンストラクタオブジェクトがコンストラクタのインスタンスの1つとしてオブジェクトを認識するかどうかを判断するメソッド。
`instanceof` 演算子のセマンティクスによって呼び出されます。

### `Symbol.isConcatSpreadable`

`Array.prototype.concat` によってオブジェクトを配列要素にフラット化する必要があることを示すブール値。

### `Symbol.iterator`

オブジェクトのデフォルトイテレータを返すメソッドです。 
`for-of` ステートメントのセマンティクスによって呼び出されます。

### `Symbol.match`

A regular expression method that matches the regular expression against a string. Called by the String.prototype.match method.

### `Symbol.replace`

正規表現と文字列を照合する正規表現メソッド。 
`String.prototype.match` メソッドによって呼び出されます。

### `Symbol.search`

正規表現に一致する文字列内のインデックスを返す正規表現メソッド。 
`String.prototype.search` メソッドによって呼び出されます。

### `Symbol.species`

派生オブジェクトを作成するために使用されるコンストラクタ関数である、関数値のプロパティ。

### `Symbol.split`

正規表現と一致するインデックスで文字列を分割する正規表現メソッド。 
`String.prototype.split` メソッドによって呼び出されます。

### `Symbol.toPrimitive`

オブジェクトを対応するプリミティブ値に変換するメソッド。 
`ToPrimitive` 抽象オペレーションによって呼び出されます。

### `Symbol.toStringTag`

オブジェクトのデフォルトの文字列記述の作成に使用される `String` 値。 
組み込みメソッド `Object.prototype.toString` によって呼び出されます。

### `Symbol.unscopables`

自身のプロパティ名が、関連付けられたオブジェクトの「with」環境バインディングから除外されたプロパティ名であるオブジェクト。

