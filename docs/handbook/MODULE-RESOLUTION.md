# Module Resolution モジュールの解決

>このセクションは、モジュールに関するいくつかの基本的な知識を前提としています。
詳細については、[モジュール](./MODULES.md)のドキュメントを参照してください。

モジュールの解決は、インポートが何を参照するかをコンパイラが判断するために使用するプロセスです。
`import { a } from "modules";"` のようなインポート文を使って、 `a` の使用をチェックするには、コンパイラがその表現を正確に知る必要があり、その定義 `moduleA` をチェックする必要があります。

この時点で、コンパイラは「 `moduleA` の形状は何ですか？」と尋ねます。
これは簡単ですが、 `moduleA` は独自の `.ts/.tsx` ファイルの1つ、またはコードが依存する `.d.ts` ファイルで定義できます。

まず、コンパイラは、インポートされたモジュールを表すファイルの検索を試みます。
そうするために、コンパイラは、[Classic](#Clasic) または [Node](#Node) という2つの異なる戦略のうちの1つに従います。
これらの戦略は、 `moduleA` を探す場所をコンパイラに指示します。

それがうまく動作せず、モジュール名が非相対（かつ  `"moduleA"` の場合）であれば、コンパイラは[アンビエントモジュール](./MODULES.md)宣言の位置を見つけようと試みます。 
次に、非相対的なインポートについて説明します。

最後に、コンパイラがモジュールを解決できなかった場合は、エラーが記録されます。
この場合、エラーは次のようなものになります。 `error TS2307: Cannot find module 'moduleA'`

## Relative vs. Non-relative module imports 相対モジュールと非相対モジュールのインポート

モジュールのインポートは、モジュール参照が相対的であるか非相対的であるかによって異なる方法で解決されます。

相対インポートは、 `/` 、 `./` または `../` で始まるものです。 いくつかの例があります。

- `import Entry from "./components/Entry";`
- `import { DefaultHeaders } from "../constants/http";`
- `import "/mod";`

その他のインポートは非相対的とみなされます。 いくつかの例があります。

- `import * as $ from "jquery";`
- `import { Component } from "@angular/core";`

相対インポートは、インポートファイルに関連して解決され、アンビエントモジュール宣言に解決できません。
実行時に相対的な位置を維持することが保証されている独自のモジュールの相対インポートを使用する必要があります。

非相対インポートは、 `baseUrl` またはパスマッピングを基準に解決できます。これについては、以下で説明します。
また、アンビエントモジュール宣言にも解決できます。 外部依存関係のいずれかをインポートするときに、非相対パスを使用します。


## Module Resolution Strategies モジュール解決戦略

可能なモジュール解決策には、[Node](#Node) と [Classic](#Classic) の2つがあります。
`--moduleResolution` フラグを使用して、モジュール解決戦略を指定することができます。 
指定されていない場合、デフォルトは `--module AMD | system | ES2015` それ以外の場合は Node。


### Classic

これは、TypeScript のデフォルトの解決策でした。
今日、この戦略は主に下位互換性のために存在します。

相対インポートは、インポートするファイルに関連して解決されます。
したがって、ソースファイル `/root/src/folder/A.ts` 内の `import {b} from "./ moduleB"` は、次の検索を行います。

1. `/root/src/folder/moduleB.ts`
1. `/root/src/folder/moduleB.d.ts`

ただし、非相対モジュールのインポートでは、コンパイラは、インポートファイルを含むディレクトリから、ディレクトリツリーを見つけ出し、一致する定義ファイルを探します。

例えば…

ソースファイル `/root/src/folder/A.ts` の `import {b} from "moduleB"` のような `moduleB` への相対的でないインポートは、 `moduleB` を見つけるために次の場所を試行する結果となります。

1. `/root/src/folder/moduleB.ts`
1. `/root/src/folder/moduleB.d.ts`
1. `/root/src/moduleB.ts`
1. `/root/src/moduleB.d.ts`
1. `/root/moduleB.ts`
1. `/root/moduleB.d.ts`
1. `/moduleB.ts`
1. `/moduleB.d.ts`


### Node

この解決戦略は、実行時に Node.js モジュール解決メカニズムを模倣しようとします。 
完全な Node.js 解決アルゴリズムの概要は、[Node.js モジュールのドキュメント](https://nodejs.org/api/modules.html#modules_all_together)に記載されています。

#### How Node.js resolves modules Node.js によるモジュールの解決方法

TSコンパイラがどのような手順を実行するかを理解するには、Node.js モジュールを理解することが重要です。
従来、Node.js のインポートは、 `require` という名前の関数を呼び出すことによって実行されます。
Node.js の動作は、 `require` に相対パスまたは非相対パスが指定されているかどうかによって異なります。

相対パスはかなり簡単です。 
例として、 `import var x = require ("./ moduleB")` を含む `/root/src/moduleA.js` にあるファイルを考えてみましょう。 
Node.js は、次の順序でインポートを解決します。

1. `/root/src/moduleB.js` という名前のファイルがある場合はそれを尋ねます。
1. `main` モジュールを指定する `package.json` という名前のファイルが含まれている場合は、 `/root/src/moduleB` フォルダに問い合わせてください。
この例では、Node.js が `{"main"： "lib / mainModule.js"}` を含む `/root/src/moduleB/package.json` ファイルを見つけた場合、Node.js は `/root/src/moduleB/lib/mainModule.js` を参照します。
1. `/root/src/moduleB` フォルダーに `index.js` という名前のファイルが含まれているかどうかを確認します。
そのファイルは、そのフォルダの 「`main`」 モジュールと暗黙的に見なされます。

これについて詳しくは、 [ファイルモジュール](https://nodejs.org/api/modules.html#modules_file_modules) と [フォルダモジュール](https://nodejs.org/api/modules.html#modules_folders_as_modules) に関する Node.js のドキュメントを参照してください。

しかし、非相対的なモジュール名の解決方法は異なっています。
Node は、 `node_modules` という名前の特別なフォルダ内のモジュールを探します。
`node_modules` フォルダは、現在のファイルと同じレベルにすることも、ディレクトリ・チェーン内の上位に置くこともできます。
Node は、ロードしようとしたモジュールが見つかるまで、各 `node_modules` を調べて、ディレクトリチェーンを上って行きます。

上記の例を追って、 `/root/src/moduleA.js` が相対パスを使用せず、 `import var x = require("moduleB");` を持っているかどうかを検討してください。
Node は、 `moduleB` を1つが働くまで各場所に解決しようとします。

1. `/root/src/node_modules/moduleB.js`
1. `/root/src/node_modules/moduleB/package.json` ( `"main"` プロパティが指定されている場合)
1. `/root/src/node_modules/moduleB/index.js` 

1. `/root/node_modules/moduleB.js`
1. `/root/node_modules/moduleB/package.json` ( `"main"` プロパティが指定されている場合)
1. `/root/node_modules/moduleB/index.js`

1. `/node_modules/moduleB.js`
1. `/node_modules/moduleB/package.json` ( `"main"` プロパティが指定されている場合)
1. `/node_modules/moduleB/index.js`

Node.js が手順（4）と（7）でディレクトリを上に移動したことに注目してください。

[`node_modules` からのモジュールのロード](https://nodejs.org/api/modules.html#modules_loading_from_node_modules_folders)に関する Node.js のドキュメントのプロセスの詳細を読むことができます。


#### How TypeScript resolves modules  TypeScript によるモジュールの解決方法

TypeScript は、コンパイル時にモジュール用の定義ファイルを見つけるために、Node.js ランタイム解決戦略を模倣します。
これを達成するために、TypeScript は、TypeScript ソースファイル拡張子（ `.ts`、 `.tsx`、および `.d.ts`）をノードの解決ロジックにオーバーレイします。 
TypeScriptは、 `"types"` という名前の `package.json` 内のフィールドを `"main"` の目的を反映するためにも使用します - コンパイラはこれを使用して、参照する `"main"` 定義ファイルを探します。

たとえば、 `/root/src/moduleA.ts` の `import {b} from "./ moduleB"` のような `import` 文は、 `"./moduleB"` を見つけるために次の場所を試行します。

1. `/root/src/moduleB.ts`
1. `/root/src/moduleB.tsx`
1. `/root/src/moduleB.d.ts`
1. `/root/src/moduleB/package.json` ( `"types"` プロパティが指定されている場合)
1. `/root/src/moduleB/index.ts`
1. `/root/src/moduleB/index.tsx`
1. `/root/src/moduleB/index.d.ts`

Node.js が `moduleB.js` という名前のファイルを検索した後、該当する `package.json` を探し、次に `index.js` を探したことを思い出してください。

同様に、非相対的なインポートは Node.js 解決戦略に続き、最初にファイルを検索してから該当するフォルダを検索します。
したがって、ソースファイル `/root/src/moduleA.ts` 内の `import {b} from "moduleB"` は、次の検索を行います。

1. `/root/src/node_modules/moduleB.ts`
1. `/root/src/node_modules/moduleB.tsx`
1. `/root/src/node_modules/moduleB.d.ts`
1. `/root/src/node_modules/moduleB/package.json` ( `"types"` プロパティが指定されている場合)
1. `/root/src/node_modules/moduleB/index.ts`
1. `/root/src/node_modules/moduleB/index.tsx`
1. `/root/src/node_modules/moduleB/index.d.ts` 

1. `/root/node_modules/moduleB.ts`
1. `/root/node_modules/moduleB.tsx`
1. `/root/node_modules/moduleB.d.ts`
1. `/root/node_modules/moduleB/package.json` ( `"types"` プロパティが指定されている場合)
1. `/root/node_modules/moduleB/index.ts`
1. `/root/node_modules/moduleB/index.tsx`
1. `/root/node_modules/moduleB/index.d.ts` 

1. `/node_modules/moduleB.ts`
1. `/node_modules/moduleB.tsx`
1. `/node_modules/moduleB.d.ts`
1. `/node_modules/moduleB/package.json` ( `"types"` プロパティが指定されている場合)
1. `/node_modules/moduleB/index.ts`
1. `/node_modules/moduleB/index.tsx`
1. `/node_modules/moduleB/index.d.ts`

ここでの手順の数に煩わされることはありません。TypeScript は、ステップ（8）と（15）でディレクトリを2回だけ飛ばしています。
これは実際に Node.js 自体がやっていること以上に複雑ではありません。


### Additional module resolution flags 追加のモジュール解決フラグ

プロジェクトのソースレイアウトが出力のレイアウトと一致しないことがあります。
通常、一連のビルドステップで最終的な出力が生成されます。
これには、 `.ts` ファイルを `.js` にコンパイルすること、および異なるソースの場所から単一の出力場所に依存関係をコピーすることが含まれます。
結果として、実行時のモジュールは、定義を含むソースファイルとは異なる名前を持つ可能性があります。
または、コンパイル時に最終出力のモジュールパスが対応するソースファイルパスと一致しないことがあります。

TypeScriptコンパイラには、最終出力を生成するためにソースに発生すると予想される変換をコンパイラに通知するための一連の追加フラグがあります。

コンパイラはこれらの変換を実行しないことに注意することが重要です。
これらの情報を使用してモジュールインポートを定義ファイルに解決するプロセスを指導します。

### Base URL

`baseUrl` の使用は、モジュールが実行時に1つのフォルダに 「デプロイ」 される AMD モジュールローダを使用するアプリケーションで一般的な方法です。
これらのモジュールのソースは別々のディレクトリに置くことができますが、ビルドスクリプトはそれらをまとめて配置します。

`baseUrl` を設定すると、コンパイラにモジュールの検索場所が通知されます。
非相対的な名前を持つすべてのモジュールのインポートは、 `baseUrl` からの相対的なものとみなされます。

`baseUrl` の値は、次のいずれかで決定されます。

- `baseUrl` コマンドライン引数の値（指定されたパスが相対パスの場合、現在のディレクトリに基づいて計算されます）
- `tsconfig.json` の `baseUrl` プロパティの値（指定されたパスが相対パスの場合、 `tsconfig.json` の場所に基づいて計算されます）

相対モジュールのインポートは `baseUrl` の設定によって影響を受けないことに注意してください。
それらはインポートファイルに対して常に解決されるためです。

[RequireJS](http://requirejs.org/docs/api.html#config-baseUrl) および [SystemJS](http://requirejs.org/docs/api.html#config-baseUrl) のドキュメントで、 `baseUrl` に関する詳細なドキュメントを参照できます。


### Path mapping

モジュールが `baseUrl` の下に直接配置されていないことがあります。
たとえば、モジュール `"jquery"` へのインポートは、実行時に `"node_modules/jquery/dist/jquery.slim.min.js"` に変換されます。
ローダは、実行時にモジュール名をファイルにマップするマッピング設定を使用します（ [RequireJs](http://requirejs.org/docs/api.html#config-baseUrl) と [SystemJS](https://github.com/systemjs/systemjs/blob/master/docs/config-api.md#paths) のドキュメントを参照）。

TypeScript コンパイラは、 `tsconfig.json` ファイルの `"paths"` プロパティを使用して、そのようなマッピングの宣言をサポートします。 
`jquery` の `"paths"` プロパティを指定する方法の例を次に示します。

```json
{
  "compilerOptions": {
    "baseUrl": ".", // This must be specified if "paths" is.
    "paths": {
      "jquery": ["node_modules/jquery/dist/jquery"] // This mapping is relative to "baseUrl"
    }
  }
}
```

`"paths"` は `"baseUrl"` に関連して解決されることに注意してください。 
`"baseUrl"` を `"."` 以外の値、すなわち `tsconfig.json` のディレクトリに設定する場合は、それに応じてマッピングを変更する必要があります。
例えば、上記の例では `"baseUrl"： "./src"` を設定すると、jquery は `"../node_modules/jquery/dist/jquery"` にマップされます。

`"paths"` を使用すると、複数のフォールバック位置を含むより洗練されたマッピングが可能になります。
一部のモジュールのみが1つの場所で使用可能で、残りのモジュールが別の場所にあるプロジェクト構成を考えてみましょう。
ビルドのステップは、すべてを1つの場所にまとめます。 プロジェクトのレイアウトは次のようになります。

```bash
projectRoot
├── folder1
│   ├── file1.ts (imports 'folder1/file2' and 'folder2/file3')
│   └── file2.ts
├── generated
│   ├── folder1
│   └── folder2
│       └── file3.ts
└── tsconfig.json
```

対応する `tsconfig.json` は次のようになります。

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "*": [
        "*",
        "generated/*"
      ]
    }
  }
}
```

これは、パターン `"*"` （すべての値）と一致するモジュールのインポートをコンパイラに指示し、2つの場所を調べます。

1. `"*"` ：同じ名前が変更されていないことを意味するので、`<moduleName>` => `<baseUrl>/<moduleName>` 
1. 追加された接頭辞 `"generated"` を持つモジュール名を意味する `"generated/*"` なので、 `<moduleName>` => `<baseUrl>/generated/<moduleName>`

このロジックに従って、コンパイラは2つのインポートを次のように解決しようとします。

- import ‘folder1/file2’
    1. パターン `*` がマッチし、ワイルドカードはモジュール名全体をキャプチャします
    1. 最初の置換を試してください： `*` -> `folder1/file2`
    1. 置換の結果は非相対名です - `baseUrl` -> `projectRoot/folder1/file2.ts` と組み合わせてください。
    1. ファイルが存在しています。 完了しました。
- import ‘folder2/file3’
    1. パターン `*` がマッチし、ワイルドカードはモジュール名全体をキャプチャします
    1. 最初の置換を試してください： `*` -> `folder2/file3`
    1. 置換の結果は非相対名です - `baseUrl` -> `projectRoot/folder2/file3.ts` と組み合わせてください。
    1. ファイルが存在しない、2番目の置換に移動する
    1. 2番目の置換は `generated/*` -> `generated/folder2/file3`
    1. 置換の結果は非相対名です - `baseUrl` -> `projectRoot/generated/folder2/file3.ts` と組み合わせてください。
    1. ファイルが存在しています。 完了しました。


### Virtual Directories with `rootDirs`

コンパイル時に複数のディレクトリからのプロジェクトソースがすべて1つの出力ディレクトリを生成するために結合されることがあります。
これはソースディレクトリのセットとして「仮想」ディレクトリを作成することができます。

`rootDirs` を使用すると、この「仮想」ディレクトリを構成するルートをコンパイラに知らせることができます。
コンパイラは、これらの「仮想」ディレクトリ内の相対モジュールのインポートを、あたかも1つのディレクトリに併合されているかのように解決することができる。

たとえば、このプロジェクトの構造を考えてみましょう。

```bash
src
 └── views
     └── view1.ts (imports './template1')
     └── view2.ts

 generated
 └── templates
         └── views
             └── template1.ts (imports './view2')
```

`src/views` のファイルは、いくつかの UI コントロールのユーザーコードです。
生成/テンプレート内のファイルは、ビルドの一部としてテンプレートジェネレータによって自動的に生成された UI テンプレートバインディングコードです。
ビルドステップでは、 `/src/views` および `/generated/templates/views` 内のファイルを出力内の同じディレクトリにコピーします。
実行時に、ビューはそのテンプレートがその横に存在することを期待できるため、相対名を `"./template"` としてインポートする必要があります。

この関係をコンパイラに指定するには、 `"rootDirs"` を使用します。
`"rootDirs"` は、実行時に内容がマージされると予想されるルートのリストを指定します。
したがって、この例では、 `tsconfig.json` ファイルは次のようになります。

```json
{
  "compilerOptions": {
    "rootDirs": [
      "src/views",
      "generated/templates/views"
    ]
  }
}
```

コンパイラは、 `rootDirs` の1つのサブフォルダに相対モジュールのインポートを見るたびに、 `rootDirs` の各エントリでこのインポートを検索しようとします。

`rootDirs` の柔軟性は、論理的にマージされた物理的なソースディレクトリのリストを指定することに限定されません。
指定された配列には、存在するかどうかにかかわらず、任意の数のアドホックな任意のディレクトリ名を含めることができます。 
これにより、コンパイラは、条件付きインクルードやプロジェクト固有のローダープラグインなどの洗練されたバンドルとランタイム機能をタイプセーフな方法で取得できます。


`./#{locale}/messages` のような相対的なモジュールパスの一部として、特別なパストークン、例えば `＃{locale}` を補間することによって、ビルドツールが自動的にロケール固有のバンドルを生成する国際化シナリオを考えてみましょう。
この仮想的な設定では、ツールはサポートされているロケールを列挙し、抽象パスを `./zh/messages` 、 `./de/messages` などにマッピングします。

これらのモジュールはそれぞれ、文字列の配列をエクスポートするものとします。 たとえば、 `./zh/messages` には次のものが含まれます。

```typescript
export default [
    "您好吗",
    "很高兴认识你"
];
```

`rootDirs` を利用することで、このマッピングをコンパイラに知らせることができ、ディレクトリが決して存在しなくても安全に `./#{locale}/messages` を解決できるようになります。
 たとえば、次の `tsconfig.json` を使用します。
 
 ```json
{
  "compilerOptions": {
    "rootDirs": [
      "src/zh",
      "src/de",
      "src/#{locale}"
    ]
  }
}
```

コンパイラは、`import messages from './#{locale}/messages'` をツール目的で `import messages from './zh/messages'` に解決し、設計時間のサポートを損なうことなくロケールに依存しない方法で開発を可能にします。


### Tracing module resolution トレースモジュールの解決

前に説明したように、コンパイラは、モジュールを解決する際に、現在のフォルダの外にあるファイルを参照することができます。
これは、モジュールが解決されなかった理由を診断するときや、誤った定義に解決されたときには難しい場合があります。
`--traceResolution` を使用してコンパイラモジュールの解決トレースを有効にすると、モジュール解決プロセス中に何が起こったかがわかります。

typescriptモジュールを使用するサンプルアプリケーションがあるとしましょう。
`app.ts` には `import * as ts from "typescript"` のようなインポートがあります。

```bash
│   tsconfig.json
├───node_modules
│   └───typescript
│       └───lib
│               typescript.d.ts
└───src
        app.ts
```

`--traceResolution` でコンパイラを呼び出す

```bash
tsc --traceResolution
```

結果は次のようになります。

```bash
======== Resolving module 'typescript' from 'src/app.ts'. ========
Module resolution kind is not specified, using 'NodeJs'.
Loading module 'typescript' from 'node_modules' folder.
File 'src/node_modules/typescript.ts' does not exist.
File 'src/node_modules/typescript.tsx' does not exist.
File 'src/node_modules/typescript.d.ts' does not exist.
File 'src/node_modules/typescript/package.json' does not exist.
File 'node_modules/typescript.ts' does not exist.
File 'node_modules/typescript.tsx' does not exist.
File 'node_modules/typescript.d.ts' does not exist.
Found 'package.json' at 'node_modules/typescript/package.json'.
'package.json' has 'types' field './lib/typescript.d.ts' that references 'node_modules/typescript/lib/typescript.d.ts'.
File 'node_modules/typescript/lib/typescript.d.ts' exist - use it as a module resolution result.
======== Module name 'typescript' was successfully resolved to 'node_modules/typescript/lib/typescript.d.ts'. ========
```


#### Things to look out for 眺めるべきこと

- インポートの名前と場所
> ======== Resolving module **‘typescript’** from **‘src/app.ts’**. ========

- コンパイラが従う戦略
> Module resolution kind is not specified, using **‘NodeJs’**.

- npm パッケージからのタイプのロード
> ‘package.json’ has **‘types’** field ‘./lib/typescript.d.ts’ that references ‘node_modules/typescript/lib/typescript.d.ts’.

- 最終結果
> ======== Module name ‘typescript’ was **successfully** resolved to ‘node_modules/typescript/lib/typescript.d.ts’. ========


### Using `--noResolve`

通常、コンパイラはコンパイルプロセスを開始する前に、すべてのモジュールのインポートを解決しようとします。
`import` をファイルに正常に解決するたびに、コンパイラが後で処理するファイルのセットにファイルが追加されます。

`--noResolve` コンパイラオプションは、コマンドラインで渡されなかったファイルをコンパイルに 「追加」しないようにコンパイラに指示します。 
それでもモジュールをファイルに解決しようとしますが、ファイルが指定されていない場合は含まれません。

例えば…

**app.ts**
```typescript
import * as A from "moduleA" // OK, 'moduleA' passed on the command-line
import * as B from "moduleB" // Error TS2307: Cannot find module 'moduleB'.
```

```bash
tsc app.ts moduleA.ts --noResolve
```

`--noResolve` を使用して `app.ts` をコンパイルすると、次の結果が得られます。

- コマンドラインで渡された `moduleA` を正しく検索します。
- `moduleB` が渡されなかったため、 `moduleB` が見つからないというエラーが発生しました。


### Common Questions よくある質問

**除外リスト内のモジュールがコンパイラによって引き上げられるのはなぜですか？**

`tsconfig.json` は、フォルダを「プロジェクト」に変換します。 
`"exclude"` または `"files"` エントリを指定しないと、 `tsconfig.json` とそのすべてのサブディレクトリを含むフォルダ内のすべてのファイルがコンパイルに含まれます。
 `"exclude"` を使用するファイルの一部を除外したい場合は、コンパイラがそれらを検索する代わりにすべてのファイルを指定する場合は、`"files"` を使用します。

それは `tsconfig.json` の自動インクルードです。 
これは、上記で議論されたモジュール解決を埋め込むものではありません。
コンパイラーがファイルをモジュールインポートのターゲットとして指定した場合は、前の手順で除外されたかどうかにかかわらず、コンパイルに含められます。

したがって、コンパイルからファイルを除外するには、そのファイルと、 `import` または `/// <reference path = "..." />` ディレクティブを含むすべてのファイルを除外する必要があります。
