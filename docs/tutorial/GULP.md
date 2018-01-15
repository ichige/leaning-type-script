# Gulpで自動化

このクイックスタートガイドでは、[gulp](https://gulpjs.com/)を利用して、どのようにしてTypeScriptを構築することをお教えします。
その後、[Browserify](http://browserify.org/)、[uglify](http://lisperator.net/uglifyjs/)または[Watchify](https://github.com/substack/watchify)をgulpに繋げます。
このガイドでは、[Babelify](https://github.com/babel/babelify)を使用して[Babel](https://babeljs.io/)機能の機能も追加しています。

既に[npm](https://www.npmjs.com/)と[Node.js](https://nodejs.org/)を使用していることを前提としています。

## 最小限のプロジェクト

新しいディレクトリから始めましょう。
今は `proj` という名前を付けておきますが、希望するものに変更することができます。

```bash
> mkdir proj
> cd proj
```

まず、プロジェクトを次のように構成します。

```
proj/
   ├─ src/
   └─ dist/
```

TypeScriptファイルは `src` フォルダで起動し、TypeScriptコンパイラを実行して `dist` に収めます。

これを足場にしましょう：

```bash
> mkdir src
> mkdir dist
```

### プロジェクトを初期化する

今度はこのフォルダをnpmパッケージにします。

```bash
> npm init
```
※ `yarn init` でも可。

一連のプロンプトが表示されます。
エントリポイント以外のデフォルト値を使用できます。
エントリポイントには、 `./dist/main.js` を使用します。
生成された `package.json` ファイルでいつでも元通りに変更することができます。

### 依存パッケージをインストールする

これで `npm install` でパッケージをインストールすることができます。
最初に `gulp-cli` をグローバルにインストールしてください（Unixシステムを使用している場合は、このガイドのコマンド `npm install` の前に接頭語 `sudo` を付ける必要があります）。

```bash
> npm install -g gulp-cli
```

その後、`typescript` 、`gulp` および `gulp-typescript` をプロジェクトのdev依存パッケージにインストールします。
[Gulp-typescript](https://www.npmjs.com/package/gulp-typescript)はTypescriptのための `gulp` プラグインです。

```bash
> npm install --save-dev typescript gulp gulp-typescript
```
※ `yarn add --dev` でも可。

### 簡単な例を書く

Hello Worldプログラムを作成しましょう。
`src` に、`main.ts` ファイルを作成します。

```typescript
function hello(compiler: string) {
    console.log(`Hello from ${compiler}`);
}
hello("TypeScript");
```

プロジェクトルート `proj` で、次のファイル `tsconfig.json` を作成します。

```json
{
    "files": [
        "src/main.ts"
    ],
    "compilerOptions": {
        "noImplicitAny": true,
        "target": "es5"
    }
}
```

###`gulpfile.js` を作成する

プロジェクトルートに `gulpfile.js` ファイルを作成します。

```javascript
var gulp = require("gulp");
var ts = require("gulp-typescript");
var tsProject = ts.createProject("tsconfig.json");

gulp.task("default", function () {
    return tsProject.src()
        .pipe(tsProject())
        .js.pipe(gulp.dest("dist"));
});
```

### 実行結果を確認する

```bash
> gulp
> node dist/main.js
```

プログラムは "Hello from TypeScript！"を出力するはずです。

## コードにモジュールを追加する

Browserifyを使う前に、コードをビルドし、モジュールをミックスに追加しましょう。
これは、実際のアプリで使用する可能性が高い構造です。

以下のファイル `src/greet.ts` を作成します。

```typescript
export function sayHello(name: string) {
    return `Hello from ${name}`;
}
```

`greet.ts` から `sayHello` をインポートするように、今のコードを変更します：

```typescript
import { sayHello } from "./greet";

console.log(sayHello("TypeScript"));
```

最後に、`tsconfig.json` へ `src/greet.ts` を追加します：

```json
{
    "files": [
        "src/index.ts",
        "src/greet.ts"
    ],
    "compilerOptions": {
        "noImplicitAny": true,
        "target": "es5"
    }
}
```

Nodeで `gulp` を実行してテストして、モジュールが正しく動作することを確認します。

```bash
> gulp
> node dist/main.js
```

ES2015モジュールの構文を使用しても、TypeScriptはNodeが使用するCommonJSモジュールを発行しました。
このチュートリアルではCommonJSを使用しますが、これを変更するにはオプションオブジェクトの `module` に設定することができます

## Browserify

今度はこのプロジェクトをNodeからブラウザに移してみましょう。
これを行うには、すべてのモジュールを1つのJavaScriptファイルにバンドルする必要があります。
幸いにも、それはまさに Browserify がすることです。
さらに、Node によって使用される CommonJS モジュールシステムを使用することができます。
これは、デフォルトのTypeScriptエミットです。
つまり、TypeScriptとNodeのセットアップは、基本的に変更されずにブラウザに転送されます。

まず、browserify、[tsify](https://www.npmjs.com/package/tsify)、およびvinyl-source-streamをインストールします。
tsifyは、gulp-typescriptのように、TypeScriptコンパイラへのアクセスを提供するBrowserifyプラグインです。
vinyl-source-streamは、Browserifyのファイル出力を、gulpが[vinyl](https://github.com/gulpjs/vinyl)と理解している形式に戻します。

```bash
> npm install --save-dev browserify tsify vinyl-source-stream
```

### ページを作成

`src` に `index.html` という名前のファイルを作成する：

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>Hello World!</title>
    </head>
    <body>
        <p id="greeting">Loading ...</p>
        <script src="bundle.js"></script>
    </body>
</html>
```

`index.ts` を変更してページを更新してください：

```typescript
import { sayHello } from "./greet";

function showHello(divName: string, name: string) {
    const elt = document.getElementById(divName);
    elt.innerText = sayHello(name);
}

showHello("greeting", "TypeScript");
```

`showHello` を呼び出すろ、段落のテキストを変更する `sayHello` 呼び出します。
gulpfileを次のように変更してください：

```javascript
var gulp = require("gulp");
var browserify = require("browserify");
var source = require('vinyl-source-stream');
var tsify = require("tsify");
var paths = {
    pages: ['src/*.html']
};

gulp.task("copy-html", function () {
    return gulp.src(paths.pages)
        .pipe(gulp.dest("dist"));
});

gulp.task("default", ["copy-html"], function () {
    return browserify({
        basedir: '.',
        debug: true,
        entries: ['src/index.ts'],
        cache: {},
        packageCache: {}
    })
    .plugin(tsify)
    .bundle()
    .pipe(source('bundle.js'))
    .pipe(gulp.dest("dist"));
});
```

これにより、`copy-html` タスクが追加され、`default` タスクの依存関係として追加されます。
つまり、いつ `default` が実行されても、`copy-html` が最初に実行されます。
また `default`、 `gulp-typescript` の代わりに `tsify` プラグインを使用してBrowserifyを呼び出すように変更しました。
好都合なことに、両方とも、TypeScriptコンパイラに同じオプションオブジェクトを渡すことができます。

`bundle` 呼び出し後、`bundle.js` という名前で出力するめに `source`（vinyl-source-streamのエイリアス）を使用します。

gulpを実行し、`dist/index.html` をブラウザで開き、ページをテストします。
このページで「Hello from TypeScript」が表示されます。

Browserify に `debug: true` 指定したことに注目してください。
これにより、tsifyは、バンドルされたJavaScriptファイル内でソースマップを出力します。
ソースマップを使用すると、バンドルされたJavaScriptではなく、元のTypeScriptコードをブラウザでデバッグできます。
ブラウザのデバッガを開き、ブレークポイントを `index.ts`  の内部に入れることで、ソースマップが動作しているかどうかをテストできます。
ページを更新するとき、ブレークポイントはページを一時停止して `greet.ts` をデバッグします。
 
## Watchify, Babel, と Uglify
 
Browserifyとtsifyでコードをバンドルするので、browserifyプラグインを使用してさまざまな機能をビルドに追加できます。
 
- Watchifyはgulpを開始し、ファイルを保存するたびに段階的にコンパイルして実行し続けます。これにより、ブラウザで編集 - 保存 - 更新サイクルを継続することができます。
- Babelは、ES2015以降をES5とES3に変換する非常に柔軟なコンパイラです。これにより、TypeScriptがサポートしていない大規模でカスタマイズされた変換を追加できます。 
- Uglifyはコードを圧縮してダウンロードする時間を短縮します。

### Watchify

バックグラウンドでコンパイルを提供するために、Watchifyから始めます。

```bash
> npm install --save-dev watchify gulp-util
```

gulpfileを次のように変更してください：

```javascript
var gulp = require("gulp");
var browserify = require("browserify");
var source = require('vinyl-source-stream');
var watchify = require("watchify");
var tsify = require("tsify");
var gutil = require("gulp-util");
var paths = {
    pages: ['src/*.html']
};

var watchedBrowserify = watchify(browserify({
    basedir: '.',
    debug: true,
    entries: ['src/index.ts'],
    cache: {},
    packageCache: {}
}).plugin(tsify));

gulp.task("copy-html", function () {
    return gulp.src(paths.pages)
        .pipe(gulp.dest("dist"));
});

function bundle() {
    return watchedBrowserify
        .bundle()
        .pipe(source('bundle.js'))
        .pipe(gulp.dest("dist"));
}

gulp.task("default", ["copy-html"], bundle);
watchedBrowserify.on("update", bundle);
watchedBrowserify.on("log", gutil.log);
```

ここでは基本的に3つの変更がありますが、コードを少しリファクタリングする必要があります。

1. `browserify` インスタンスを呼び出して `watchify` でラップし、その結果を保持します。
1. `watchedBrowserify.on("update", bundle);` を呼びだし、TypeScriptを変更するBrowserifyが実行されるようにします。
1. `watchedBrowserify.on("log", gutil.log);` を呼び出し、コンソールにログを出力します。
(1)と(2)は、`default` タスク外から `browserify` を呼び出すことを意味します。
またWatchifyとGulpの両方が `default` を呼び出す必要があるため、名前の関数を与える必要があります。
(3)でログ出力を追加することはオプションですが、ビルドの設定のデバッグには非常に便利です。

Gulpを実行すると、起動しつつ監視状態に入ります。
`main.ts` のコードの `showHello` を変更して保存してみてください。
次のような出力が表示されます。

```bash
proj$ gulp
[10:34:20] Using gulpfile ~/src/proj/gulpfile.js
[10:34:20] Starting 'copy-html'...
[10:34:20] Finished 'copy-html' after 26 ms
[10:34:20] Starting 'default'...
[10:34:21] 2824 bytes written (0.13 seconds)
[10:34:21] Finished 'default' after 1.36 s
[10:35:22] 2261 bytes written (0.02 seconds)
[10:35:24] 2808 bytes written (0.05 seconds)
```

### Uglify

最初にUglifyをインストールします。
Uglifyのポイントはあなたのコードを変更することですので、ソースマップを動作させるために `vinyl-buffer` と `gulp-sourcemaps` もインストールする必要があります。

```bash
> npm install --save-dev gulp-uglify vinyl-buffer gulp-sourcemaps
```

gulpfileを次のように変更してください。

```javascript
var gulp = require("gulp");
var browserify = require("browserify");
var source = require('vinyl-source-stream');
var tsify = require("tsify");
var uglify = require('gulp-uglify');
var sourcemaps = require('gulp-sourcemaps');
var buffer = require('vinyl-buffer');
var paths = {
    pages: ['src/*.html']
};

gulp.task("copy-html", function () {
    return gulp.src(paths.pages)
        .pipe(gulp.dest("dist"));
});

gulp.task("default", ["copy-html"], function () {
    return browserify({
        basedir: '.',
        debug: true,
        entries: ['src/index.ts'],
        cache: {},
        packageCache: {}
    })
    .plugin(tsify)
    .bundle()
    .pipe(source('bundle.js'))
    .pipe(buffer())
    .pipe(sourcemaps.init({loadMaps: true}))
    .pipe(uglify())
    .pipe(sourcemaps.write('./'))
    .pipe(gulp.dest("dist"));
});
```

`uglify` 自体を一度だけ呼び出していることに注意してください… `buffer` と `sourcemaps` の呼び出しは、sourcemaps が働き続けることを確認するために存在します。
これらの呼び出しは、以前のようなインライン sourcemaps を使用する代わりに、別の sourcemaps ファイルを提供します。
今度はGulpを実行して `bundle.js` が判読不能なものに縮小されていることを確認できます。

### Babel

まず BabelifyとES2015用のBabelプリセットをインストールします。
Uglifyのように、Babelifyはコードを改ざんするので、 `vinyl-buffer` と `gulp-sourcemaps` が必要になります。
デフォルトでは Babelifyだけの `.js`、`.es`、`.es6` と `.jsx` 拡張子を持つファイルを処理し、Babelifyのオプションとして`.ts` 拡張子を追加する必要があります。

```bash
npm install --save-dev babelify babel-preset-es2015 vinyl-buffer gulp-sourcemaps
```

gulpfileを次のように変更してください。

```javascript
var gulp = require('gulp');
var browserify = require('browserify');
var source = require('vinyl-source-stream');
var tsify = require('tsify');
var sourcemaps = require('gulp-sourcemaps');
var buffer = require('vinyl-buffer');
var paths = {
    pages: ['src/*.html']
};

gulp.task('copyHtml', function () {
    return gulp.src(paths.pages)
        .pipe(gulp.dest('dist'));
});

gulp.task('default', ['copyHtml'], function () {
    return browserify({
        basedir: '.',
        debug: true,
        entries: ['src/index.ts'],
        cache: {},
        packageCache: {}
    })
    .plugin(tsify)
    .transform('babelify', {
        presets: ['es2015'],
        extensions: ['.ts']
    })
    .bundle()
    .pipe(source('bundle.js'))
    .pipe(buffer())
    .pipe(sourcemaps.init({loadMaps: true}))
    .pipe(sourcemaps.write('./'))
    .pipe(gulp.dest('dist'));
});
```

TypeScriptターゲットES2015も必要です。
Babelは、TypeScriptが発行するES2015コードからES5を生成します。
`tsconfig.json` を変更しましょう。

```json
{
    "files": [
        "src/index.ts"
    ],
    "compilerOptions": {
        "noImplicitAny": true,
        "target": "es2015"
    }
}
```

BabelのES5出力は、そのような単純なスクリプトのTypeScriptの出力と非常によく似ているはずです。
