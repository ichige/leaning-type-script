# 5分で学ぶ TypeScript

TypeScriptを使って簡単なWebアプリケーションを構築しましょう。

## TypeScriptのインストール

TypeScriptのツールを入手する主な方法は2つあります。

- npm経由で（Node.jsパッケージマネージャ）
- TypeScriptのVisual Studioプラグインをインストールする

Visual Studio 2017およびVisual Studio 2015 Update 3には、デフォルトでTypeScriptが含まれています。
Visual StudioでTypeScriptをインストールしていない場合でも、それを[ダウンロード](https://www.typescriptlang.org/#download-links)できます。

npm利用の場合

```bash
> npm install -g typescript
```

## 最初のTypeScriptファイルの作成

エディタで、次のJavaScriptコードを `greeter.ts` ファイルに入力します。

```typescript
function greeter(person) {
    return "Hello, " + person;
}

let user = "Jane User";

document.body.innerHTML = greeter(user);
```

## コードのコンパイル

`.ts` 拡張子を使用しましたが、このコードはJavaScriptにすぎません。
既存のJavaScriptアプリケーションをそのままコピー/ペーストすることができます。

コマンドラインで、TypeScriptコンパイラを実行します。

```bash
> tsc greeter.ts
```

結果は、入力したものと同じJavaScriptを含むファイル `greeter.js` になります。
JavaScriptアプリケーションでTypeScriptを使用して動かしているのです!

これで、TypeScriptが提供する新しいツールのいくつかを利用できるようになりました。
次の `: string` ように、 'person'関数の引数に型アノテーションを追加します。

```typescript
function greeter(person: string) {
    return "Hello, " + person;
}

let user = "Jane User";

document.body.innerHTML = greeter(user);
```

## 型アノテーション

TypeScriptの型アノテーションは、関数または変数の意図したコントラクトを記録するための軽量な方法です。
この場合、単一の文字列パラメータで `greeter` 関数を呼び出すことを意図しています。
代わりに配列を渡すようにコール・グリーティングを変更してみます：

```typescript
function greeter(person: string) {
    return "Hello, " + person;
}

let user = [0, 1, 2];

document.body.innerHTML = greeter(user);
```

再コンパイルすると、エラーが表示されます：

```bash
> tsc greeter-error-ts
greeter-error.ts(7,35): error TS2345: Argument of type 'number[]' is not assignable to parameter of type 'string'.
```

同様に、 `greeter` 呼び出しの引数をすべて削除してみてください。
TypeScriptは、予期せぬ数のパラメータでこの関数を呼び出したことを知らせます。
どちらの場合でも、TypeScriptは、コードの構造と提供する型注釈の両方に基づいて静的分析を提供できます。

エラーがあっても `greeter-error.js` ファイルは作成されていることに注意してください。
コードにエラーがあっても、TypeScriptを使用できます。
しかし、この場合、TypeScriptはコードが期待どおりに実行されない可能性があることを警告します。

## インタフェース

サンプルをさらに開発しましょう。
ここでは、`firstName` フィールドと `lastName` フィールドを持つオブジェクトを記述するインタフェースを使用します。
TypeScriptでは、内部構造に互換性がある場合、2つのタイプが互換性があります。
これにより、明示的な `implements` 句を使用することなく、インタフェースが必要とするシェイプを持つだけでインタフェースを実装することができます。

```typescript
interface Person {
    firstName: string;
    lastName: string;
}

function greeter(person: Person) {
    return "Hello, " + person.firstName + " " + person.lastName;
}

let user = { firstName: "Jane", lastName: "User" };

document.body.innerHTML = greeter(user);
```

## クラス

最後に、前回の例をクラスで拡張しましょう。
TypeScriptは、クラスベースのオブジェクト指向プログラミングのサポートなど、JavaScriptの新機能をサポートしています。

ここでは、コンストラクタといくつかのパブリックフィールドを持つ  `Student` クラスを作成します。
クラスとインタフェースがうまく連携し、プログラマが適切な抽象レベルを決定できるようになります。

また、コンストラクタに対する `public` 付きの引数の使用は、その名前のプロパティを自動的に作成することを可能にする省略表現です。

```typescript
class Student {
    fullName: string;
    constructor(public firstName: string, public middleInitial: string, public lastName: string) {
        this.fullName = firstName + " " + middleInitial + " " + lastName;
    }
}

interface Person {
    firstName: string;
    lastName: string;
}

function greeter(person : Person) {
    return "Hello, " + person.firstName + " " + person.lastName;
}

let user = new Student("Jane", "M.", "User");

document.body.innerHTML = greeter(user);
```

再実行する `tsc classes.ts`と、生成されたJavaScriptが以前のコードと同じであることがわかります。
TypeScriptのクラスは、JavaScriptで頻繁に使用されるものと同じプロトタイプベースのオブジェクト指向の略語です。

## TypeScript Webアプリケーションの実行

ここで次のように `greeter.html` に入力します：

```html
<!DOCTYPE html>
<html>
    <head><title>TypeScript Greeter</title></head>
    <body>
        <script src="classes.js"></script>
    </body>
</html>
```

最初のシンプルなTypeScript Webアプリケーションを実行するには、`greeter.html` をブラウザで開きます！

