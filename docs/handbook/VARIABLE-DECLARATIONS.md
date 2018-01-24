# Variable Declarations (変数の宣言)

## Variable Declarations とは

`let` と `const` はJavaScriptの2つの比較的新しい型の変数宣言です。
以前に述べたように、`let` はいくつかの点で `var` に似ていますが、ユーザーがJavaScriptで実行する共通の「問題」を避けることができます。
`const` は変数への再代入を防ぐという点で `let` の拡張です。

TypeScriptはJavaScriptのスーパーセットであるため、`let` と `const` を自然にサポートします。
ここでは、これらの新しい宣言についてさらに詳しく説明し、なぜ `var` に匹敵するのかを詳しく説明します。

JavaScriptをオフラインで使用したことがある場合は、次のセクションがあなたのメモリをリフレッシュする良い方法かもしれません。
JavaScriptの `var` 宣言のすべての癖に精通している方は、先に飛ばすほうが簡単かもしれません。


## `var` declarations

JavaScriptで変数を宣言することは、伝統的に `var` キーワードで行われてきました。

```javascript
var a = 10;
```

変数 `a` を値 `10` で宣言しました。

関数の中で変数を宣言することもできます：

```javascript
function f() {
    var message = "Hello, world!";

    return message;
}
```

他の関数内で同じ変数にアクセスすることもできます。

```javascript
function f() {
    var a = 10;
    return function g() {
        var b = a + 1;
        return b;
    }
}

var g = f();
g(); // returns '11'
```

この例では、関数 `g` は 関数 `f` で宣言された変数 `a` を取得しました。
`g` 関数が呼び出されると、`a` の値は `f` 関数の中の `a` の値に結び付けられます。
`f` 関数が一度実行されると `g` 関数が呼び出されたとしても、変数 `a` へアクセスして変更する事ができるでしょう。

```javascript
function f() {
    var a = 1;

    a = 2;
    var b = g();
    a = 3;

    return b;

    function g() {
        return a;
    }
}

f(); // returns '2'
```

### Scoping rules 

`var` 宣言には、他の言語に使用されている人にとっては奇妙なスコープ規則があります。 
次の例を考えてみましょう。

```typescript
function f(shouldInitialize: boolean) {
    if (shouldInitialize) {
        var x = 10;
    }

    return x;
}

f(true);  // returns '10'
f(false); // returns 'undefined'
```

いくつかの読者がこの例では二度見をするかもしれません。
変数 `x` は `if` ブロック内で宣言されましたが、そのブロックの外側から変数 `x` にアクセスできました。
なぜなら、`var` 宣言は、それを含む関数、モジュール、名前空間、またはグローバルスコープ内のどこにでもアクセスできるからです。
一部の人々は、この `var` スコーピングまたは 関数スコーピングと呼んでいます。
パラメータも関数スコープです。

これらのスコーピング規則は、いくつかのタイプの間違いを引き起こす可能性があります。
これらが悪化させる1つの問題は、同じ変数を複数回宣言することがエラーではないという事実です。

```typescript
function sumMatrix(matrix: number[][]) {
    var sum = 0;
    for (var i = 0; i < matrix.length; i++) {
        var currentRow = matrix[i];
        for (var i = 0; i < currentRow.length; i++) {
            sum += currentRow[i];
        }
    }

    return sum;
}
```

たぶん、いくつかの点を見つけるのは簡単でしたが、内側の `for` ループは、同じ関数スコープの変数を参照しているので、誤って変数 `i` を上書きしてしまいます。
経験豊富な開発者が今知っているように、同様の種類のバグはコードレビューをすり抜けるものであり、無限の不満の根源になる可能性があります。

### Variable capturing quirks (可変キャプチャの特徴)

すぐに次のスニペットの出力が何であるかを推測してください：

```typescript
for (var i = 0; i < 10; i++) {
    setTimeout(function() { console.log(i); }, 100 * i);
}
```

よく知られていない人のために、`setTimeout` は特定のミリ秒後に関数を実行しようとします（他の何かが実行を止めるのを待っています）。

準備はできましたか？ 見てみましょう：

```typescript
10
10
10
10
10
10
10
10
10
10
```

多くのJavaScript開発者はこの動作に精通していますが、驚いた場合は、確かに一人ではありません。
ほとんどの人は出力が…

```typescript
0
1
2
3
4
5
6
7
8
9
```

変数キャプチャについて前述したことを覚えていますか？
`setTimeout` に渡すすべての関数式は、実際には同じスコープの同じ `i` を参照します。

それが何を意味するかを考えてみましょう。
`setTimeout` は、数ミリ秒後に関数を実行しますが、`for` ループの実行が停止した後にのみ実行されます。
`for` ループの実行が停止するまでに、`i` の値は10になります。
だから、与えられた関数が呼び出されるたびに、それは `10` を出力します！

一般的な回避策は、即時呼び出し関数式であるIIFEを使用して、各繰り返しで `i` を取得することです。

```typescript
for (var i = 0; i < 10; i++) {
    // capture the current state of 'i'
    // by invoking a function with its current value
    (function(i) {
        setTimeout(function() { console.log(i); }, 100 * i);
    })(i);
}
```

この奇妙なパターンは実際にはかなり一般的です。
パラメータリストの `i` は実際に `for` ループで宣言された `i` をシャドウしますが、それらを同じ名前にしたので、ループボディをあまり変更する必要はありませんでした。


## `let` declarations

ここまでで、`var` にいくつかの問題があることが分かりました。これは、なぜ `let` 文が導入されたのかということです。
使用されるキーワードとは別に、`let` ステートメントは `var` ステートメントと同じ方法で記述されます。

```typescript
let hello = "Hello!";
```

重要な違いは構文にではなく、意味論であり、ここではこれから説明します。

### Block-scoping

変数が `let` を使って宣言されると、それは何らかの呼び出しのレキシカルスコープまたはブロックスコープを使用します。
スコープが含まれている関数にリークする `var` で宣言された変数とは異なり、ブロックスコープ変数は最も近い格納ブロックまたは `for` ループの外側には表示されません。

```typescript
function f(input: boolean) {
    let a = 100;

    if (input) {
        // Still okay to reference 'a'
        let b = a + 1;
        return b;
    }

    // Error: 'b' doesn't exist here
    return b;
}
```

ここでは、`a` と `b` という2つのローカル変数があります。
`a` のスコープは `f` 関数の本文に限定され、`b` のスコープは `if` 文のブロックを含むものに限定されます。

`catch` 節で宣言された変数も同様のスコープ規則を持ちます。

```typescript
try {
    throw "oh no!";
}
catch (e) {
    console.log("Oh well.");
}

// Error: 'e' doesn't exist here
console.log(e);
```

ブロックスコープ変数のもう1つの特性は、実際に宣言される前に読み書きできないことです。
これらの変数は、その範囲全体にわたって「存在」していますが、宣言が一時的なデッドゾーンの一部になるまで、すべての点を指します。
これは、`let` 文の前にアクセスできないという洗練された方法です。
幸いなことに、TypeScriptはそれをあなたに知らせます。

```typescript
a++; // illegal to use 'a' before it's declared;
let a;
```

注意すべき点は、宣言される前にブロックスコープの変数を取り込むことができることです。
唯一のキャッチは、宣言の前にその関数を呼び出すことは違法だということです。
ES2015を対象とすると、モダンなランタイムはエラーをスローします。
しかし、今はTypeScriptが許可されており、これをエラーとして報告しません。

```typescript
function foo() {
    // okay to capture 'a'
    return a;
}

// illegal call 'foo' before 'a' is declared
// runtimes should throw an error here
foo();

let a;
```

一時的なデッドゾーンの詳細については、[Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#Temporal_dead_zone_and_errors_with_let)の関連コンテンツを参照してください。

### Re-declarations and Shadowing

`var` 宣言では、変数を宣言した回数は問題ではないと言いました。
ただ唯一の値となるだけです。

```typescript
function f(x) {
    var x;
    var x;

    if (true) {
        var x;
    }
}
```

上記の例では、`x` の宣言はすべて実際に同じ `x` を参照していますが、これは完全に有効です。
これはしばしばバグの原因になります。
ありがたいことに、`let` 宣言では許可していません。

```typescript
let x = 10;
let x = 20; // error: can't re-declare 'x' in the same scope
```

問題があることを教えてくれる変数は必ずしもTypeScriptのブロックスコープである必要はありません。

```typescript
function f(x) {
    let x = 100; // error: interferes with parameter declaration
}

function g() {
    let x = 100;
    var x = 100; // error: can't have both declarations of 'x'
}
```

ブロックスコープの変数を関数スコープの変数で決して宣言することはできません。
ブロックスコープ変数は、明確に異なるブロック内で宣言する必要があります。

```typescript
function f(condition, x) {
    if (condition) {
        let x = 100;
        return x;
    }

    return x;
}

f(false, 0); // returns '0'
f(true, 0);  // returns '100'
```

よりネストされたスコープに新しい名前を導入することをシャドーイングといいます。
偶発的なシャドーイングが発生した場合には、特定のバグを防ぐことができるという点で、両刃の剣です。
例えば、`let` 変数を使って以前の `sumMatrix` 関数を書いたとします。

```typescript
function sumMatrix(matrix: number[][]) {
    let sum = 0;
    for (let i = 0; i < matrix.length; i++) {
        var currentRow = matrix[i];
        for (let i = 0; i < currentRow.length; i++) {
            sum += currentRow[i];
        }
    }

    return sum;
}
```

ループのこのバージョンは、実際には、内側ループの `i` が外側ループから `i` をシャドーしているため、合計を正しく実行します。

より明瞭なコードを書くために、シャドウイングは避けるべきです。
それを利用するのに適しているシナリオがいくつかありますが、あなたは最良の判断を下すべきです。


### Block-scoped variable capturing

`var` 宣言で変数をキャプチャするというアイデアに初めて触れたとき、一度キャプチャされた変数がどのように動作するかを簡単に説明しました。
スコープを実行するたびに、これをより直感的に理解するために、変数の「環境」を作成します。
スコープ内のすべての実行が完了した後でも、その環境とそのキャプチャされた変数が存在する可能性があります。

```typescript
function theCityThatAlwaysSleeps() {
    let getCity;

    if (true) {
        let city = "Seattle";
        getCity = function() {
            return city;
        }
    }

    return getCity();
}
```

環境内から `city` を獲得したので、`if` ブロックが実行を終了したにもかかわらず、まだアクセスできます。

以前の `setTimeout` の例では、`for` ループの各繰り返しで変数の状態を取得するためにIIFEを使用する必要があったことを思い出してください。
実際、私たちが行っていたことは、取り込まれた変数のための新しい変数環境を作り出すことでした。
それは少し痛みでしたが、幸運にも、TypeScriptではこれをやり直す必要はありません。

ループの一部として宣言されたときに、`let` 宣言の動作が大きく異なるようにします。
これらの宣言は、ループ自体に新しい環境を導入するのではなく、反復ごとに新しいスコープを作成します。
とにかくこれは私たちがIIFEでやっていたものなので、`let` の宣言を使うだけの古い `setTimeout` の例を変更することができます。

```typescript
for (let i = 0; i < 10 ; i++) {
    setTimeout(function() { console.log(i); }, 100 * i);
}
```

期待どおりに印刷されます

```typescript
0
1
2
3
4
5
6
7
8
9
```


## `const` declarations

`const` 宣言は、変数を宣言する別の方法です。

```typescript
const numLivesForCat = 9;
```

彼らは `let` 宣言に似ていますが、その名前が示すように、一度バインドされるとその値は変更できません。
言い換えれば、`let` と同じスコープ規則を持っていますが、それらに再割り当てすることはできません。

これは、それらが参照する値が不変であるという考えと混同すべきではありません。

```typescript
const numLivesForCat = 9;
const kitty = {
    name: "Aurora",
    numLives: numLivesForCat,
}

// Error
kitty = {
    name: "Danielle",
    numLives: numLivesForCat
};

// all "okay"
kitty.name = "Rory";
kitty.name = "Kitty";
kitty.name = "Cat";
kitty.numLives--;
```

それを避けるための具体的な措置をとらない限り、`const` 変数の内部状態は変更可能です。
幸いにも、TypeScriptでは、オブジェクトのメンバーが `readonly` であることを指定できます。
[インタフェースの章](https://www.typescriptlang.org/docs/handbook/interfaces.html)には詳細があります。


## `let` vs. `const`

同様のスコープセマンティクスを持つ2種類の宣言があることを考えれば、どちらを使うべきかを尋ねるのは当然です。
ほとんどの幅広い質問と同様に、答えは「それは依存しています。」

[最小特権の原則](https://en.wikipedia.org/wiki/Principle_of_least_privilege)を適用すると、変更する予定のもの以外の宣言は `const` を使用する必要があります。
その理由は、変数が書き込まれる必要がない場合、同じコードベースで作業している他の人がオブジェクトに自動的に書き込むことができず、実際に変数に代入する必要があるかどうかを検討する必要があるということです。
`const` を使用すると、データの流れについて推論するときにコードをより予測可能にします。

一方、`let` は `var` よりも書き出すことが不要になり、多くのユーザーはその簡潔さを好むでしょう。
このハンドブックの大部分は、その関心事で `let` 宣言を使用しています。

最善の判断を使用し、そして決断する場合はチームの他の人と相談してください。


## Destructuring

TypeScriptが持つもう1つのECMAScript 2015機能は、非構造化です。
詳しくは、[Mozilla Developer Networkの記事](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)を参照してください。 このセクションでは、簡単な概要を説明します。

### Array destructuring

最も単純な形式の非構造化は、配列の構造化の割り当てです。

```typescript
let input = [1, 2];
let [first, second] = input;
console.log(first); // outputs 1
console.log(second); // outputs 2
```

これにより、`first` と `second` という名前の2つの新しい変数が作成されます。
これはインデックスを使用するのと同じですが、はるかに便利です

```typescript
first = input[0];
second = input[1];
```

非構造化は既に宣言された変数でも機能します：

```typescript
// swap variables
[first, second] = [second, first];
```

関数へのパラメータを使用すると次のようになります。

```typescript
function f([first, second]: [number, number]) {
    console.log(first);
    console.log(second);
}
f([1, 2]);
```

`...` の構文を使用して、リスト内の残りの項目の変数を作成することができます。

```typescript
let [first, ...rest] = [1, 2, 3, 4];
console.log(first); // outputs 1
console.log(rest); // outputs [ 2, 3, 4 ]
```

もちろん、これはJavaScriptなので、気にしない末尾の要素は無視できます：

```typescript
let [first] = [1, 2, 3, 4];
console.log(first); // outputs 1
```

または他の要素：

```typescript
let [, second, , fourth] = [1, 2, 3, 4];
```

### Object destructuring

オブジェクトを非構造化することもできます：

```typescript
let o = {
    a: "foo",
    b: 12,
    c: "bar"
};
let { a, b } = o;
```

これは、`o.a` と `o.b` から新しい変数 `a` と `b` を作成します。
必要がない場合は、`c` をスキップできることに注意してください。

配列の非構造化と同様に、宣言なしで代入を行うことができます：

```typescript
({ a, b } = { a: "baz", b: 101 });
```

このステートメントをかっこで囲む必要があることに注意してください。
JavaScriptは通常、ブロックの先頭として `{` を解析します。

`...` の構文を使用して、オブジェクト内の残りの項目の変数を作成することができます。

```typescript
let { a, ...passthrough } = o;
let total = passthrough.b + passthrough.c.length;
```

#### Property renaming

プロパティに異なる名前を付けることもできます。

```typescript
let { a: newName1, b: newName2 } = o;
```

ここで構文が混乱し始めます。
`a:newName1` を 「 `a as newName1` 」として読むことができます。
あなたが書いたかのように、左から右の方向です：

```typescript
let newName1 = o.a;
let newName2 = o.b;
```

紛らわしいことに、ここのコロンはタイプを示していません。
型を指定した場合、型全体が非構造化された後でも型を記述する必要があります。

```typescript
let { a, b }: { a: string, b: number } = o;
```

#### Default values

デフォルト値では、プロパティが未定義の場合にデフォルト値を指定できます。

```typescript
function keepWholeObject(wholeObject: { a: string, b?: number }) {
    let { a, b = 1001 } = wholeObject;
}
```

`keepWholeObject` には、`b` が未定義であっても、プロパティ `a` と `b` だけでなく、`wholeObject` の変数が追加されました。

### Function declarations

非構造化は関数宣言でも機能します。
単純なケースの場合、これは簡単です。

```typescript
type C = { a: string, b?: number }
function f({ a, b }: C): void {
    // ...
}
```

しかし、デフォルトを指定する方がパラメータの方が一般的です。
また、非構造化を使用してデフォルトを取得するのは難しい場合があります。
まず、デフォルト値の前にパターンを置くことを覚えておく必要があります。

```typescript
function f({ a, b } = { a: "", b: 0 }): void {
    // ...
}
f(); // ok, default to { a: "", b: 0 }
```

> 上のスニペットは、型推論の例です。ハンドブックの後半で説明します。

次に、メインイニシャライザではなく、非構造化プロパティのオプションプロパティのデフォルトを指定することを忘れないでください。
`C` は `b` オプションで定義されたことに注意してください。

```typescript
function f({ a, b = 0 } = { a: "" }): void {
    // ...
}
f({ a: "yes" }); // ok, default b = 0
f(); // ok, default to { a: "" }, which then defaults b = 0
f({}); // error, 'a' is required if you supply an argument
```

非構造化は注意して使用してください。
前の例が示すように、最も単純な構造化式以外のものは混乱します。
深くネストされた非構造化の場合は特にそうです。名前の変更、デフォルト値、型の注釈を重ねることなく、実際には理解しづらいことがあります。
非構造化表現を小さく、シンプルに保つようにしてください。
あなたは、自分自身が生成することになる割り当てをいつでも書くことができます。


### Spread

スプレッド演算子は、非構造化の逆です。
配列を別の配列に、またはオブジェクトを別のオブジェクトに広げることができます。
例えば：

```typescript
let first = [1, 2];
let second = [3, 4];
let bothPlus = [0, ...first, ...second, 5];
```

これは、bothPlusに値 `[0,1,2,3,4,5]` を与える。
スプレッドは1番目と2番目の浅いコピーを作成します。
これらはスプレッドによって変わらない。

オブジェクトをスプレッドすることもできます：

```typescript
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { ...defaults, food: "rich" };
```

`search` は `{food： "rich"、price： "$$"、ambiance： "noisy"}` です。
オブジェクトのスプレッドは、配列のスプレッドより複雑です。
配列のスプレッドと同様に、左から右に進みますが、結果はまだオブジェクトです。
つまり、スプレッドオブジェクトの後に来るプロパティは、先に来たプロパティを上書きします。
したがって、前の例を最後に変更すると、次のようになります。

```typescript
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { food: "rich", ...defaults };
```

そして、`defaults` の `food`プロパティで、 `food: "rich"` を上書きします。
これは、このケースでは必要ではありません。

オブジェクトスプレッドには他にも驚くべき制限がいくつかあります。
まず、オブジェクトの[独自の列挙可能なプロパティ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Enumerability_and_ownership_of_properties)のみが含まれています。
基本的には、オブジェクトのインスタンスを広げるときにメソッドを失うということです。

```typescript
class C {
  p = 12;
  m() {
  }
}
let c = new C();
let clone = { ...c };
clone.p; // ok
clone.m(); // error!
```

第2に、Typescriptコンパイラは、汎用関数の型パラメータのスプレッドを許可しません。
その機能は、将来のバージョンの言語で期待されています。
