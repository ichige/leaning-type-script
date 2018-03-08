# ENUMS 列挙型

## Enums 列挙型とは

列挙型は、名前付き定数のセットを定義することを可能にします。
列挙型を使用すると、意図を文書化するのが簡単になり、別のケースのセットが作成される可能性があります。
TypeScriptは数値と文字列ベースの列挙型の両方を提供します。


### Numeric enums 数値の列挙型

最初に数値列挙型から始めます。これはおそらく他の言語から来ている場合にはもっとよく知られています。
列挙型は、 `enum` キーワードを使用して定義できます。

```typescript
enum Direction {
    Up = 1,
    Down,
    Left,
    Right,
}
```

上記では、`Up` が `1` で初期化されている数値列挙型があります。
次のメンバーはすべて、その時点から自動的にインクリメントされます。
言い換えれば、 `Direction.Up` の値は `1` 、 `Down` の値は `2` 、 `Left` の値は `3` 、 `Right` の値は `4` です。

望むなら、イニシャライザを完全に離れることができます。

```typescript
enum Direction {
    Up,
    Down,
    Left,
    Right,
}
```

ここで、 `Up` は値 `0` を持ち、 `Down` は `1` を持ちます。
この自動インクリメントの動作は、メンバの値自体は気にしないかもしれないが、各値が同じ列挙型の他の値と区別されている場合に便利です。

列挙型の使用は単純です。
列挙型自体のプロパティとして任意のメンバーにアクセスし、列挙型の名前を使用して型を宣言します。

```typescript
enum Response {
    No = 0,
    Yes = 1,
}

function respond(recipient: string, message: Response): void {
    // ...
}

respond("Princess Caroline", Response.Yes)
```

数値の列挙型は、[計算されたメンバと定数メンバに混在する](https://www.typescriptlang.org/docs/handbook/enums.html#computed-and-constant-members)ことができます（下記参照）。
簡単な話として、イニシャライザを持たない列挙型は最初に配置する必要があり、数値定数またはその他の定数列挙型メンバーで初期化された数値列挙型の後に来なければならないことです。
言い換えれば、以下は許されない。

```typescript
enum E {
    A = getSomeValue(),
    B, // error! 'A' is not constant-initialized, so 'B' needs an initializer
}
```


### String enums 文字列の列挙型

文字列列挙型も同様の概念ですが、以下に記載されているように微妙な[実行時の違い](https://www.typescriptlang.org/docs/handbook/enums.html#enums-at-runtime)があります。
文字列列挙型では、各メンバを文字列リテラルで、または別の文字列列挙型メンバで定数で初期化する必要があります。

```typescript
enum Direction {
    Up = "UP",
    Down = "DOWN",
    Left = "LEFT",
    Right = "RIGHT",
}
```

文字列列挙型には自動インクリメントの動作はありませんが、文字列列挙型は、文字列列挙型が効果的に「直列化」されます。
言い換えれば、デバッグして数値列挙型の実行時の値を読み取る必要がある場合、その値は不透明であることが多く、それ自体は有用な意味を持ちません（[逆マッピング](https://www.typescriptlang.org/docs/handbook/enums.html#enums-at-runtime)がよく役立ちますが） 列挙型メンバ自体の名前とは無関係に、コードが実行されるときに意味のある読みやすい値を与えることができます。


### Heterogeneous enums 異種の列挙型

技術的に列挙型は文字列と数字のメンバーと混在することがありますが、それがなぜそうしたいのかは分かりません。

```typescript
enum BooleanLikeHeterogeneousEnum {
    No = 0,
    Yes = "YES",
}
```

JavaScriptのランタイム動作を巧みに利用しようとしているのでない限り、これをしないことをお勧めします。


### Computed and constant members 計算された定数

各列挙型メンバには、それに関連付けられた値があり、定数または計算式のいずれかになります。 列挙型メンバは、次の場合に定数と見なされます。

- これは列挙型の最初のメンバーであり、初期化子はありません。その場合、値 `0` が割り当てられます。

```typescript
// E.X is constant:
enum E { X }
```

- それには初期化子がなく、先行する列挙型メンバは数値定数です。この場合、現在の列挙型メンバの値は、前の列挙型メンバの値に1を加えた値になります。

```typescript
// All enum members in 'E1' and 'E2' are constant.

enum E1 { X, Y, Z }

enum E2 {
    A = 1, B, C
}
```

- 列挙型メンバは定数 `enum` 式で初期化されます。定数式 `enum` 式は、コンパイル時に完全に評価できるTypeScript式のサブセットです。次の場合、式は定数列挙式です。

    1. リテラル `enum` 式（基本的に文字列リテラルまたは数値リテラル）
    1. 以前に定義された定数列挙型メンバ（別の `enum` に由来する）への参照です。
    1. カッコで囲まれた定数の `enum` 式
    1. 定数 `enum` 式に適用される `+` 、 `-` 、`~` の単項演算子の1つ
    1. `+` 、 `-`、 `*`、 `/`、 `%`、 `<<`、 `>>`、 `>>>`、 `&`、 `|`、 `^` オペランドとして定数列挙式を持つ2項演算子定数列挙式が `NaN` または `Infinity` に評価されるのはコンパイル時エラーです。


それ以外の場合、列挙型メンバーは計算されたとみなされます。

```typescript
enum FileAccess {
    // constant members
    None,
    Read    = 1 << 1,
    Write   = 1 << 2,
    ReadWrite  = Read | Write,
    // computed member
    G = "123".length
}
```


### Union enums and enum member types 連合列挙型および列挙型メンバー型

計算されていない定数列挙型の特別なサブセットがあります。定数列挙型メンバー。
リテラル列挙型メンバは、初期化された値を持たない定数列挙型メンバ、または…

- 任意の文字列リテラル（ `foo`、`bar`、`baz`など）
- 任意の数値リテラル（例えば `1` , `100`）
- 任意の数値リテラルに適用される単項マイナス（たとえば、`-1` 、`-100` ）

列挙型のすべてのメンバーがリテラル列挙型の値を持つとき、いくつかの特別なセマンティクスが再生されます。

1つ目は、列挙型メンバーも型になるということです！
たとえば、特定のメンバは列挙型メンバの値のみを持つことができます。

```typescript
enum ShapeKind {
    Circle,
    Square,
}

interface Circle {
    kind: ShapeKind.Circle;
    radius: number;
}

interface Square {
    kind: ShapeKind.Square;
    sideLength: number;
}

let c: Circle = {
    kind: ShapeKind.Square,
    //    ~~~~~~~~~~~~~~~~ Error!
    radius: 100,
}
```

もう1つの変更は、列挙型自体が効果的に各列挙型メンバの和集合になるということです。
[ユニオンタイプ](https://www.typescriptlang.org/docs/handbook/advanced-types.html#union-types)についてはまだ議論していませんが、あなたが知る必要があるのは、ユニオン列挙型を持つタイプシステムは、列挙型自体に存在する正確な値のセットを知ることができるということです。
そのため、TypeScriptはバリューを誤って比較しているバグをキャッチすることができます。
例えば…

```typescript
enum E {
    Foo,
    Bar,
}

function f(x: E) {
    if (x !== E.Foo || x !== E.Bar) {
        //             ~~~~~~~~~~~
        // Error! Operator '!==' cannot be applied to types 'E.Foo' and 'E.Bar'.
    }
}
```

この例では、まず `x` が `E.Foo` ではないかどうかを確認しました。
その検査が成功すれば、`||` が短絡し、 `if` の本体が動作します。
しかし、チェックが成功しなかった場合、 `x` は `E.Foo` にしかなりません。
したがって、 `E.Bar` と等しいかどうかを確認することはできません。


### Enums at runtime 実行時の列挙型

列挙型は実行時に存在する実際のオブジェクトです。
たとえば、次の列挙型では…

```typescript
enum E {
    X, Y, Z
}
```

実際に関数に渡すことができます。

```typescript
function f(obj: { X: number }) {
    return obj.X;
}

// Works, since 'E' has a property named 'X' which is a number.
f(E);
```


### Reverse mappings 逆マッピング

数値の列挙型メンバーは、メンバーのプロパティー名を持つオブジェクトを作成するだけでなく、列挙値から列挙名への逆マッピングも取得します。
たとえば、次の例では…

```typescript
enum Enum {
    A
}
let a = Enum.A;
let nameOfA = Enum[a]; // "A"
```

TypeScriptはこれを以下のようなJavaScriptにコンパイルするかもしれません。

```javascript
var Enum;
(function (Enum) {
    Enum[Enum["A"] = 0] = "A";
})(Enum || (Enum = {}));
var a = Enum.A;
var nameOfA = Enum[a]; // "A"
```

この生成コードでは、列挙型は、forward（ `name` -> `value` ）とreverse（ `value` -> `name` ）の両方のマッピングを格納するオブジェクトにコンパイルされます。
他の列挙型メンバへの参照は、常にプロパティへのアクセスとして発行され、インライン化されることはありません。

文字列列挙型メンバーは逆マッピングをまったく生成しないことに注意してください。


### const enums 定数列挙型

ほとんどの場合、列挙型は完全に有効なソリューションです。
しかし時にはより厳しい要求もあります。
列挙型の値にアクセスするときに余分に生成されたコードと追加の間接化のコストを支払うのを避けるために、定数列挙型を使用することができます。
定数列挙型は、列挙型の `const` 修飾子を使って定義されています。

```typescript
const enum Enum {
    A = 1,
    B = A * 2
}
```

定数列挙型は定数の列挙式のみを使用でき、通常の列挙型と異なり、コンパイル時に完全に削除されます。
定数列挙型メンバーは、使用サイトでインライン展開されます。
定数列挙型は計算されたメンバーを持つことができないので、これは可能です。

```typescript
const enum Directions {
    Up,
    Down,
    Left,
    Right
}

let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right]
```

生成されたコードは…

```javascript
var directions = [0 /* Up */, 1 /* Down */, 2 /* Left */, 3 /* Right */];
```


### Ambient enums アンビエント列挙型

アンビエント列挙型は、すでに存在する列挙型の形状を記述するために使用されます。

```typescript
declare enum Enum {
    A = 1,
    B,
    C = 2
}
```

アンビエント環境とアンビエント環境の間の重要な違いの1つは、通常の列挙型では、イニシャライザを持たないメンバは、その前の列挙体メンバが一定であるとみなされると定数と見なされます。
対照的に、イニシャライザを持たないアンビエント（および非定数）列挙メンバは、常に計算されたものとみなされます。
