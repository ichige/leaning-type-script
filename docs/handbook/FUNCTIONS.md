# Functions 関数

## はじめに

関数は、JavaScriptのアプリケーションの基本的なブロック構造です。
クラスは、抽象化、クラスの模倣、情報隠蔽、およびモジュールのレイヤーを構築する方法です。
TypeScriptでは、クラス、名前空間、およびモジュールが存在しますが、関数は、動かし方を記述する際に重要な役割を果たします。
TypeScriptはまた、標準のJavaScript関数にいくつかの新しい機能を追加して、使いやすくしています。

## Functions 関数とは

まず、JavaScriptの場合と同様に、名前付き関数または無名関数の両方として型関数を作成することができます。
これにより、APIの関数リストを作成する場合でも、別の関数に引き渡すための一回限りの関数を作成する場合でも、アプリケーションに最適なアプローチを選択することができます。

JavaScriptでこれらの2つのアプローチがどのように見えるかをすばやく説明するには…

```typescript
// Named function
function add(x, y) {
    return x + y;
}

// Anonymous function
let myAdd = function(x, y) { return x + y; };
```

JavaScriptと同様に、関数は関数本体の外部の変数を参照できます。
そういった場合、これらの変数を `capure` すると言います。
この仕組みを理解する上で、このテクニックを使用する際のトレードオフについては、この記事の範囲外であり、JavaScriptとTypeScriptを使用してこの機械がどのように重要な役割を果たしているかを理解しています。

```typescript
let z = 100;

function addToZ(x, y) {
    return x + y + z;
}
```

## Function Types 関数の型

### Typing the function 関数に型を付ける

先ほどの簡単な例に型を追加しましょう。

```typescript
function add(x: number, y: number): number {
    return x + y;
}

let myAdd = function(x: number, y: number): number { return x + y; };
```

各パラメータに型を追加することも、関数自体の戻り値に型を追加することもできます。
TypeScriptはreturn文を見て戻り値の型を調べることができるので、オプションとして多くの場合これを省略することもできます。


### Writing the function type 関数型を書く

関数に型を付けたので、各関数の型を見て関数の完全な型を書き出しましょう。

```typescript
let myAdd: (x: number, y: number) => number =
    function(x: number, y: number): number { return x + y; };
```

関数の型は、引数の型と戻り型の2つの部分が同じです。
関数型全体を書き出すときは、両方の部分が必要です。
私たちはパラメータリストをパラメータリストと同様に書き出し、各パラメータに名前と型を与えます。
この名前は読みやすくするためのものです。 代わりに以下のように書いています。

```typescript
let myAdd: (baseValue: number, increment: number) => number =
    function(x: number, y: number): number { return x + y; };
```

パラメータ型が整列している限り、関数型でパラメータを指定した名前にか関係なく、関数の有効な型とみなされます。

2番目の部分は戻り値の型です。 パラメータと戻り値の型の間に太い矢印（ `=>` ）を使用することで、戻り値の型を明確にします。
前述したように、これは関数型の必須部分です。したがって、関数が値を返さない場合は、それを残す代わりに `void` を使用します。

注目すべきは、パラメータと戻り値の型だけが関数型を構成することです。
キャプチャされた変数は型に反映されません。 実際、キャプチャされた変数は、関数の「隠れた状態」の一部であり、APIを構成しません。

### Inferring the types 型の推論

この例を試してみると、方程式の一方の側に型があり、もう一方に型がない場合は、型コンパイラが型を把握できることがわかります。

```typescript
// myAdd has the full function type
let myAdd = function(x: number, y: number): number { return  x + y; };

// The parameters 'x' and 'y' have the type number
let myAdd: (baseValue: number, increment: number) => number =
    function(x, y) { return x + y; };
```

これは型推論の一種である "文脈型の型付け"と呼ばれています。
これは、プログラムの入力を抑えるための労力を削減するのに役立ちます。


## Optional and Default Parameters  オプションとデフォルトのパラメータ

TypeScriptでは、すべてのパラメータが関数で必要とされています。
これは `null` または `undefined` を与えることができないことを意味するのではなく、関数が呼び出されるときに、ユーザーが各パラメーターの値を提供したことをコンパイラーがチェックします。
コンパイラは、これらのパラメータが関数に渡される唯一のパラメータであるとも仮定します。
要するに、関数に与えられる引数の数は、関数が期待するパラメータの数と一致しなければなりません。

```typescript
function buildName(firstName: string, lastName: string) {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // error, too few parameters
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");         // ah, just right
```

JavaScriptでは、すべてのパラメータはオプションであり、ユーザーは適切と思われるパラメータをオフのままにすることができます。 
その場合、これらの値はは `undefined` です。
TypeScriptでこの機能を利用できるようにするには、オプションにしたいパラメータの最後に `?` を追加します。
たとえば、上記の姓のパラメータをオプションにしたいとしましょう。

```typescript
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

let result1 = buildName("Bob");                  // works correctly now
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");         // ah, just right
```

オプションのパラメータは、必須のパラメータに従わなければなりません。
姓ではなく名をオプションにする必要がある場合は、関数のパラメータの順序を変更し、名をリストの最後に置きます。

TypeScriptでは、ユーザーがパラメーターを提供しない場合、またはユーザーが `undefined` をその場所に渡す場合に、パラメーターを割り当てる値を設定することもできます。
これらは、デフォルトで初期化されたパラメータと呼ばれます。 前の例を取り、 `"Smith"` の姓をデフォルトにしましょう。

```typescript
function buildName(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // works correctly now, returns "Bob Smith"
let result2 = buildName("Bob", undefined);       // still works, also returns "Bob Smith"
let result3 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result4 = buildName("Bob", "Adams");         // ah, just right
```

すべての必須パラメータの後に来るデフォルト初期化パラメータはオプションで扱われ、オプションのパラメータと同様に、それぞれの関数を呼び出すときに省略することができます。
これはオプションのパラメータを意味し、後続のデフォルトパラメータは型の共通性を共有するため、両方とも…

```typescript
function buildName(firstName: string, lastName?: string) {
    // ...
}
```

そして…

```typescript
function buildName(firstName: string, lastName = "Smith") {
    // ...
}
```

同じタイプ `(firstName: string, lastName?: string) => string` を共有します。
`lastName` のデフォルト値は型で消えますが、パラメータがオプションであるという事実だけが残されます。

プレーンなオプションのパラメータとは異なり、デフォルトで初期化されたパラメータは、必要なパラメータの後に発生する必要はありません。
デフォルトで初期化されたパラメータが必須パラメータの前にある場合、ユーザーは明示的に `undefined` を渡してデフォルトの初期化された値を取得する必要があります。
たとえば、`firstName` にデフォルトの初期化子のみを入れた最後の例を書くことができます。

```typescript
function buildName(firstName = "Will", lastName: string) {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // error, too few parameters
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");         // okay and returns "Bob Adams"
let result4 = buildName(undefined, "Adams");     // okay and returns "Will Adams"
```

## Rest Parameters 残りのパラメータ

必須パラメータ、オプションパラメータ、およびデフォルトパラメータはすべて共通点があります。一度に1つのパラメータについて話します。
場合によっては、複数のパラメータをグループとして扱いたい場合や、関数が最終的に何個のパラメータを取るかを知ることができない場合があります。
JavaScriptでは、すべての関数本体の内部に表示される `arguments` 変数を使用して、引数を直接使用できます。

```typescript
function buildName(firstName: string, ...restOfName: string[]) {
    return firstName + " " + restOfName.join(" ");
}

let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

残りのパラメータは無限の数のオプションパラメータとして扱われます。
残りのパラメータの引数を渡すときは、必要な数だけ使用することができます。 あなたは何も渡すことはできません。
コンパイラは、省略記号（ `...` ）の後に指定された名前で渡された引数の配列を作成し、関数内で使用できるようにします。

省略記号は、残りのパラメータを持つ関数の型でも使用されます。

```typescript
function buildName(firstName: string, ...restOfName: string[]) {
    return firstName + " " + restOfName.join(" ");
}

let buildNameFun: (fname: string, ...rest: string[]) => string = buildName;
```

## `this`

JavaScriptで `this` を使用する方法を学ぶことは、通例のようなものです。
TypeScriptはJavaScriptのスーパーセットなので、TypeScriptの開発者は `this` の使い方を学ぶ必要があり、 `this` が正しく使用されていないときを見分ける方法も必要です。
幸運なことに、TypeScriptではいくつかのテクニックを使って `this` の間違った使い方をキャッチできます。
しかし、JavaScriptで `this` がどのように動作するのかを知る必要がある場合は、まずYehuda Katzの[JavaScript関数呼び出しの理解と "this"](http://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/)を読んでください。
Yehudaの記事では、 `this` の内部動作について説明していますので、ここでは基本について説明します。

### `this` and arrow functions `this` とアロー関数

JavaScriptでは、`this` は関数が呼び出されたときに設定される変数です。
これは非常に強力で柔軟な機能ですが、関数が実行されている状況を常に知る必要はありません。
これは、特に関数を返すときや引数として関数を渡すときに、混乱を招くことが知られています。

例を見てみましょう。

```typescript
let deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        return function() {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

`createCardPicker ` はそれ自身が関数を返す関数であることに注意してください。 
例を実行しようとすると、予想される警告ボックスの代わりにエラーが発生します。
これは、`createCardPicker ` によって作成された関数で使用されている `this` が、`deck` オブジェクトの代わりに `window` に設定されるためです。
これは、 `cardPicker()` を単独で呼び出すためです。
このようなトップレベルの非メソッド構文呼び出しは、`this` に `window` を使用します。 （注：strictモードでは、`window` ではなく`undefined` になります）。

後で使用する関数を返す前に、関数が正しい `this` に束縛されていることを確認することで、これを修正できます。
この方法では、後でどのように使用されるかにかかわらず、元の `deck` オブジェクトを見ることができます。
これを行うために、関数式を変更してECMAScript 6のアロー構文を使用します。
アロー関数は、呼び出される場所ではなく関数が作成される場所を取得します。

```typescript
let deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        // NOTE: the line below is now an arrow function, allowing us to capture 'this' right here
        return () => {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

さらに、コンパイラに `--noImplicitThis` フラグを渡すと、この誤植をしたときに、TypeScriptから警告が表示されます。 
`this.suits[pickedSuit]` の `this` は `any` 型であることを指摘します。

### `this` parameters `this` のパラメータ

残念ながら、 `this.suits[pickedSuit]` のタイプはまだ `any` です。
なぜなら、`this` はオブジェクトリテラル内の関数式から来ているからです。
これを修正するには、明示的な `this` パラメーターを指定します。
`this` パラメータは、関数のパラメータリストの最初に来る偽のパラメータです。

```typescript
function f(this: void) {
    // make sure `this` is unusable in this standalone function
}
```

上記の例の `Card` と `deck` のインターフェイスをいくつか追加して、タイプを明確にして再利用しやすくしてみましょう。

```typescript
interface Card {
    suit: string;
    card: number;
}
interface Deck {
    suits: string[];
    cards: number[];
    createCardPicker(this: Deck): () => Card;
}
let deck: Deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    // NOTE: The function now explicitly specifies that its callee must be of type Deck
    createCardPicker: function(this: Deck) {
        return () => {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

これでTypeScriptは、 `createCardPicker ` が  `Deck` オブジェクトに対して呼び出されることを期待していることを認識します。
つまり、 `this` は `any` ではなく `Deck` タイプであるため、`--noImplecitThis` でエラーが発生することはありません。


### `this` parameters in callbacks

関数を後で呼び出すライブラリに渡すと、コールバックで `this` でエラーが発生することもあります。
あなたのコールバックを呼び出すライブラリは通常の関数のように呼び出すので、`this` は `undefined` になります。
いくつかの作業で、ホッグパラメータを使用してコールバックによるエラーを防ぐことができます。
まず、ライブラリ作成者はコールバック型に `this` を注釈する必要があります。

```typescript
interface UIElement {
    addClickListener(onclick: (this: void, e: Event) => void): void;
}
```

`this` ： `void` は、 `addClickListener` がこのタイプを必要としない関数であることを `onclick` が期待していることを意味します。 
次に、あなたの呼び出しコードに `this` で注釈を付けます。

```typescript
class Handler {
    info: string;
    onClickBad(this: Handler, e: Event) {
        // oops, used this here. using this callback would crash at runtime
        this.info = e.message;
    }
}
let h = new Handler();
uiElement.addClickListener(h.onClickBad); // error!
```

アノテーションが付いている `this` を使って、ハンドラのインスタンスで `onClickBad` を呼び出さなければならないことを明示します。
次にTypeScriptは `addClickListener` に `this` ： `void` を持つ関数が必要であることを検出します。
エラーを修正するには、 `this` のタイプを変更します。

```typescript
class Handler {
    info: string;
    onClickGood(this: void, e: Event) {
        // can't use this here because it's of type void!
        console.log('clicked!');
    }
}
let h = new Handler();
uiElement.addClickListener(h.onClickGood);
```

`onClickGood` はそのタイプを `void` と指定しているので、`addClickListener` に正しいです。
もちろん、これは `this.info` を使用できないことを意味します。
両方とも必要な場合は、アロー関数を使用する必要があります。

```typescript
class Handler {
    info: string;
    onClickGood = (e: Event) => { this.info = e.message }
}
```

これは、アロー関数は `this` をキャプチャしないので、いつも `this`： `void` を期待するものにそれらを渡すことができます。
欠点は、`Handler` 型のオブジェクトごとに1つの矢印関数が作成されていることです。
一方、メソッドは一度だけ作成され、`Handler` のプロトタイプに添付されます。
`Handler` 型のすべてのオブジェクト間で共有されます。


## Overloads オーバーロード

javaScriptは本質的に非常に動的な言語です。
1つのJavaScript関数が、渡された引数の形状に基づいて異なるタイプのオブジェクトを返すことは珍しいことではありません。

````typescript
let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x): any {
    // Check to see if we're working with an object/array
    // if so, they gave us the deck and we'll pick the card
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Otherwise just let them pick the card
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
````

ここで `pickCard` 関数は、ユーザーが渡した内容に基づいて2つの異なるものを返します。
ユーザーがデッキを表すオブジェクトを渡すと、関数はカードを選択します。
ユーザーがカードを選択すると、選択したカードが表示されます。
では、これを型システムにどのように記述しますか？

答えは、オーバーロードのリストと同じ関数に複数の関数型を指定することです。
このリストは、コンパイラが関数呼び出しを解決するために使用するリストです。
`pickCard` が受け入れるものと返すものを記述するオーバーロードのリストを作成しましょう。

```typescript
let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x): any {
    // Check to see if we're working with an object/array
    // if so, they gave us the deck and we'll pick the card
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Otherwise just let them pick the card
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

この変更により、オーバーロードにより、 `pickCard` 関数への型チェックされた呼び出しが可能になりました。

コンパイラが正しい型チェックを選択するためには、基本的なJavaScriptと同様の処理が行われます。
オーバーロードリストを調べ、最初のオーバーロードを試行して、指定されたパラメータで関数を呼び出そうとします。
一致した場合は、このオーバーロードを正しいオーバーロードとして選択します。
このため、慣習的な注文は、最も具体的なものから最も具体的でないものへと過負荷をかけます。

`function pickCard(x): any` の部分はオーバーロードリストの一部ではないので、オブジェクトを取るオーバーロードと番号を取るオーバーロードという2つのオーバーロードしかありません。 
他のパラメータ型で `pickCard` を呼び出すと、エラーが発生します。

