# Declaration Merging - マージの宣言

## Introduction - 導入

TypeScript のユニークな概念のいくつかは、タイプレベルで JavaScript オブジェクトの形状を記述します。 
TypeScript にとって特にユニークな例の1つは、「宣言のマージ」という概念です。 
このコンセプトを理解することで、既存の JavaScript を使用する際に利点が得られます。 
また、より高度な抽象概念への扉を開く。

この記事の目的上、「宣言のマージ」とは、コンパイラが同じ名前で宣言された2つの宣言を1つの定義にマージすることを意味します。 
このマージされた定義には、元の宣言の両方の機能があります。 
任意の数の宣言をマージすることができます。 
これは2つの宣言だけに限定されません。


## Basic Concepts - 基本コンセプト

TypeScript では、宣言によって、ネームスペース、型、または値の3つのグループの少なくとも1つにエンティティが作成されます。 
ネームスペースを作成する宣言は、名前付きスペースを作成します。
ネームスペースには、ドット付き表記を使用してアクセスされる名前が含まれています。 
型作成宣言はそれだけです。
宣言された図形で表示され、指定された名前にバインドされた型を作成します。 
最後に、値を作成する宣言は、出力 JavaScript に表示される値を作成します。

| DeclarationType | Namespace | Type | Value |
| --- | --- | --- | --- |
| Namespace | X | | X |
| Class | |  X | X |
| Enum | | X | X |
| Interface | | X | |
| Type Alias |  | X | | 
| Function | | | X |
| Variable | | | X |

各宣言で何が作成されているか理解することは、宣言のマージを実行するときにマージされるものを理解するのに役立ちます。


## Merging Interfaces - マージ・インタフェース

最も簡単で最も一般的な宣言マージのタイプは、インタフェースマージです。 
最も基本的なレベルでは、マージは両方の宣言のメンバを機械的に同じ名前の単一のインタフェースに結合します。

```typescript
interface Box {
    height: number;
    width: number;
}

interface Box {
    scale: number;
}

let box: Box = {height: 5, width: 6, scale: 10};
```

インタフェースの非関数メンバは一意でなければなりません。 ユニークでない場合は、同じタイプでなければなりません。 
インタフェースが両方とも同じ名前の異なる関数メンバーを宣言していても、異なる型のものである場合、コンパイラはエラーを発行します。

関数メンバーの場合、同じ名前の各関数メンバは、同じ関数のオーバーロードを記述するものとして扱われます。 
注意すべき点は、インタフェース `A` が後のインタフェース `A` とマージする場合、第2のインタフェースは第1のインタフェースよりも高い優先順位を有することである。

つまり、この例では…

```typescript
interface Cloner {
    clone(animal: Animal): Animal;
}

interface Cloner {
    clone(animal: Sheep): Sheep;
}

interface Cloner {
    clone(animal: Dog): Dog;
    clone(animal: Cat): Cat;
}
```

3つのインタフェースがマージされ、1つの宣言が次のように作成されます。

```typescript
interface Cloner {
    clone(animal: Dog): Dog;
    clone(animal: Cat): Cat;
    clone(animal: Sheep): Sheep;
    clone(animal: Animal): Animal;
}
```

各グループの要素は同じ順序を維持しますが、グループ自体は最初に順序付けされた後のオーバーロードセットとマージされます。

このルールの例外の1つは特殊なシグネチャです。 
シグネチャに型が単一の文字列リテラル型（たとえば、文字列リテラルの和集合ではない）のパラメータを持つ場合、それはマージされたオーバーロードリストの先頭にバブリングされます。

たとえば、次のインタフェースが一緒にマージされます。

```typescript
interface Document {
    createElement(tagName: any): Element;
}
interface Document {
    createElement(tagName: "div"): HTMLDivElement;
    createElement(tagName: "span"): HTMLSpanElement;
}
interface Document {
    createElement(tagName: string): HTMLElement;
    createElement(tagName: "canvas"): HTMLCanvasElement;
}
```

その結果、 `Document` のマージされた宣言は次のようになります。

```typescript
interface Document {
    createElement(tagName: "canvas"): HTMLCanvasElement;
    createElement(tagName: "div"): HTMLDivElement;
    createElement(tagName: "span"): HTMLSpanElement;
    createElement(tagName: string): HTMLElement;
    createElement(tagName: any): Element;
}
```


## Merging Namespaces - ネームスペースのマージ

インタフェースと同様に、同じ名前のネームスペースもメンバーをマージします。 
ネームスペースはネームスペースと値の両方を作成するので、両方の方法を理解する必要があります。

ネームスペースをマージするには、各ネームスペースで宣言されたエクスポートされたインターフェイスからの型定義がマージされ、内部にマージされたインターフェイス定義を持つ単一のネームスペースが形成されます。

ネームスペース値をマージするには、各宣言サイトで、指定された名前のネームスペースが既に存在する場合は、既存のネームスペースを取り出し、2番目のネームスペースのエクスポートされたメンバーを最初のネームスペースに追加します。

この例では、 `Animals` の宣言のマージ…

```typescript
namespace Animals {
    export class Zebra { }
}

namespace Animals {
    export interface Legged { numberOfLegs: number; }
    export class Dog { }
}
```

次のものと同等です。

```typescript
namespace Animals {
    export interface Legged { numberOfLegs: number; }

    export class Zebra { }
    export class Dog { }
}
```

ネームスペースのマージのこのモデルは有益な出発点ですが、エクスポートされていないメンバーがどうなるかを理解する必要もあります。
エクスポートされていないメンバーは、元の（結合されていない）ネームスペースでのみ表示されます。 
これは、マージ後、他の宣言から来たマージされたメンバーは、エクスポートされていないメンバーを見ることができないことを意味します。

この例では、これをより明確に見ることができます。

```typescript
namespace Animal {
    let haveMuscles = true;

    export function animalsHaveMuscles() {
        return haveMuscles;
    }
}

namespace Animal {
    export function doAnimalsHaveMuscles() {
        return haveMuscles;  // <-- error, haveMuscles is not visible here
    }
}
```

`haveMuscles` はエクスポートされないため、同じマージされていないネームスペースを共有する `animalsHaveMuscles` 関数のみがシンボルを参照できます。 
`doAnimalsHaveMuscles` 関数は、マージされた `Animal` ネームスペースの一部であっても、この未エクスポートのメンバーを見ることはできません。


## Merging Namespaces with Classes, Functions, and Enums - クラス、関数型、列挙型のネームスペースのマージ

ネームスペースは、他のタイプの宣言とマージするのにも十分柔軟です。 
そのためには、ネームスペース宣言は、それがマージする宣言の後になければなりません。 
結果の宣言には、両方の宣言型のプロパティがあります。 
TypeScript は、この機能を使用して、JavaScript や他のプログラミング言語の一部のパターンをモデル化します。

### Merging Namespaces with Classes - クラスのネームスペースのマージ

これにより、ユーザーは内部クラスを記述することができます。

```typescript
class Album {
    label: Album.AlbumLabel;
}
namespace Album {
    export class AlbumLabel { }
}
```

マージされたメンバーの表示ルールは、「ネームスペースのマージ」セクションで説明したものと同じです。
したがって、マージされたクラスの `AlbumLabel` クラスをエクスポートして表示する必要があります。 
最終結果は、別のクラスの内部で管理されるクラスです。 
また、既存のクラスに静的メンバーを追加するためにネームスペースを使用することもできます。

内部クラスのパターンに加えて、関数を作成し、その関数にプロパティを追加することでさらに機能を拡張する JavaScript の習慣に精通しているかもしれません。 
TypeScript は、宣言のマージを使用して、このような定義を型安全に構築します。

```typescript
function buildLabel(name: string): string {
    return buildLabel.prefix + name + buildLabel.suffix;
}

namespace buildLabel {
    export let suffix = "";
    export let prefix = "Hello, ";
}

alert(buildLabel("Sam Smith"));
```

同様に、静的メンバーで列挙型を拡張するためにネームスペースを使用することもできます。

```typescript
enum Color {
    red = 1,
    green = 2,
    blue = 4
}

namespace Color {
    export function mixColor(colorName: string) {
        if (colorName == "yellow") {
            return Color.red + Color.green;
        }
        else if (colorName == "white") {
            return Color.red + Color.green + Color.blue;
        }
        else if (colorName == "magenta") {
            return Color.red + Color.blue;
        }
        else if (colorName == "cyan") {
            return Color.green + Color.blue;
        }
    }
}
```


## Disallowed Merges - 許可されないマージ

TypeScript ではすべてのマージが許可されるわけではありません。 
現在、クラスは他のクラスまたは変数とマージできません。 
クラスのマージを模倣する方法については、[MixScript の TypeScript](./MIXIN.md) を参照してください。


## Module Augmentation - モジュールの拡張

JavaScript モジュールはマージをサポートしていませんが、既存のオブジェクトをインポートして更新することで、既存のオブジェクトにパッチを適用できます。 
観察可能なおもちゃの例を見てみましょう…。

```typescript
// observable.js
export class Observable<T> {
    // ... implementation left as an exercise for the reader ...
}

// map.js
import { Observable } from "./observable";
Observable.prototype.map = function (f) {
    // ... another exercise for the reader
}
```

これは TypeScript でもうまくいきますが、コンパイラは `Observable.prototype.map` について知らない。 
モジュールの拡張を使用して、コンパイラにそのことを伝えることができます…

```typescript
// observable.ts stays the same
// map.ts
import { Observable } from "./observable";
declare module "./observable" {
    interface Observable<T> {
        map<U>(f: (x: T) => U): Observable<U>;
    }
}
Observable.prototype.map = function (f) {
    // ... another exercise for the reader
}


// consumer.ts
import { Observable } from "./observable";
import "./map";
let o: Observable<number>;
o.map(x => x.toFixed());
```

モジュール名は、`import/export` のモジュール指定子と同じ方法で解決されます。 
詳細については、[モジュール](./MODULES.md)を参照してください。 
拡張の宣言は、元のファイルと同じファイルに宣言されているかのようにマージされます。 
ただし、既存の宣言に加えて、新しい上位レベルの宣言を追加で宣言することはできません。


### Global augmentation - グローバルの拡張

モジュール内からグローバルスコープに宣言を追加することもできます。

```typescript
// observable.ts
export class Observable<T> {
    // ... still no implementation ...
}

declare global {
    interface Array<T> {
        toObservable(): Observable<T>;
    }
}

Array.prototype.toObservable = function () {
    // ...
}
```

グローバルの拡張は、モジュールの拡張と同じ動作と制限を持ちます。
