# Iterators and Generators イテレータとジェネレータ

## Iterables イテラブル

オブジェクトは、 `Symbol.iterator` プロパティの実装がある場合、iterable とみなされます。
`Array` 、 `Map` 、 `Set` 、 `String` 、 `Int32Array` 、 `Uint32Array` などの組み込み型の中には、 `Symbol.iterator` プロパティが既に実装されているものがあります。
オブジェクト上の `Symbol.iterator` 関数は反復する値のリストを返す責任があります。

### `for..of`  statements `for ～ of` 構文

オブジェクトの `Symbol.iterator` プロパティを呼び出して、繰り返し可能なオブジェクトに対する `for..of` ループを作成します。 
配列の単純な `for..of` ループを次に示します。

```typescript
let someArray = [1, "string", false];

for (let entry of someArray) {
    console.log(entry); // 1, "string", false
}
```


### `for..of` vs. `for..in` statements `for ～ of` vs `for ～ in` 構文

`for..of` と `for..in` ステートメントの両方がリストを繰り返します。 
反復される値は異なりますが、 `for..in` は反復されるオブジェクトのキーのリストを返しますが、 `for..of` は反復されるオブジェクトの数値プロパティの値のリストを返します。

この区別を示す例を次に示します。

```typescript
let list = [4, 5, 6];

for (let i in list) {
   console.log(i); // "0", "1", "2",
}

for (let i of list) {
   console.log(i); // "4", "5", "6"
}
```

もう一つの違いは `for..in` がどんなオブジェクトでも動作することです。 
このオブジェクトのプロパティを検査する方法として機能します。 
`for..of` は、主に反復可能オブジェクトの値に関心があります。
`Map` や `Set` のようなビルトインオブジェクトは、格納された値にアクセスするための `Symbol.iterator` プロパティを実装しています。

```typescript
let pets = new Set(["Cat", "Dog", "Hamster"]);
pets["species"] = "mammals";

for (let pet in pets) {
   console.log(pet); // "species"
}

for (let pet of pets) {
    console.log(pet); // "Cat", "Dog", "Hamster"
}
```

### Code generation コードの出力

#### Targeting ES5 and ES3

ES5 または ES3 をターゲットにする場合、イテレータは `Array` 型の値に対してのみ使用できます。
これらの非配列値が `Symbol.iterator` プロパティを実装していても、非配列値に対して `for..of` ループを使用するとエラーになります。

コンパイラは、 `for..of` ループの単純な `for` ループを生成します。
たとえば、次のようになります。

```typescript
let numbers = [1, 2, 3];
for (let num of numbers) {
    console.log(num);
}
```

次のように生成されます。

```javascript
var numbers = [1, 2, 3];
for (var _i = 0; _i < numbers.length; _i++) {
    var num = numbers[_i];
    console.log(num);
}
```

#### Targeting ECMAScript 2015 and higher

ECMAScipt 2015 準拠のエンジンをターゲットにすると、コンパイラは `for..of` ループを生成し、エンジンの組み込みイテレータ実装をターゲットにします。

