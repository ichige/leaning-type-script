# Mixins - ミックスイン

## Introduction - 導入

伝統的な OO 階層と並んで、再利用可能なコンポーネントからクラスを構築するもう1つの一般的な方法は、より単純な部分クラスを組み合わせて構築することです。
Scala のような言語のミックスインや特性の考え方に精通しているかもしれません。
そのパターンも JavaScript コミュニティで人気を博しています。


## Mixin sample - ミックスインのサンプル

下のコードでは、TypeScript でミックスインをモデル化する方法を示します。 
コードの後、我々はそれがどのように動作するかを分解する。

```typescript
// Disposable Mixin
class Disposable {
    isDisposed: boolean;
    dispose() {
        this.isDisposed = true;
    }

}

// Activatable Mixin
class Activatable {
    isActive: boolean;
    activate() {
        this.isActive = true;
    }
    deactivate() {
        this.isActive = false;
    }
}

class SmartObject implements Disposable, Activatable {
    constructor() {
        setInterval(() => console.log(this.isActive + " : " + this.isDisposed), 500);
    }

    interact() {
        this.activate();
    }

    // Disposable
    isDisposed: boolean = false;
    dispose: () => void;
    // Activatable
    isActive: boolean = false;
    activate: () => void;
    deactivate: () => void;
}
applyMixins(SmartObject, [Disposable, Activatable]);

let smartObj = new SmartObject();
setTimeout(() => smartObj.interact(), 1000);

////////////////////////////////////////
// In your runtime library somewhere
////////////////////////////////////////

function applyMixins(derivedCtor: any, baseCtors: any[]) {
    baseCtors.forEach(baseCtor => {
        Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
            derivedCtor.prototype[name] = baseCtor.prototype[name];
        });
    });
}
```

## Understanding the sample - サンプルを理解する

コードサンプルは、ミックスインとして機能する2つのクラスから始まります。 
それぞれが特定の活動や能力に焦点を当てているのが分かります。 
後でそれらを組み合わせて、両方の機能から新しいクラスを作成します。

```typescript
// Disposable Mixin
class Disposable {
    isDisposed: boolean;
    dispose() {
        this.isDisposed = true;
    }

}

// Activatable Mixin
class Activatable {
    isActive: boolean;
    activate() {
        this.isActive = true;
    }
    deactivate() {
        this.isActive = false;
    }
}
```

次に、2つのミックスインの組み合わせを扱うクラスを作成します。 
これをどのように行うかを詳しく見てみましょう。

```typescript
class SmartObject implements Disposable, Activatable {
```

上で気づいた最初のことは、 `extends` を使用する代わりに、`implements` を使用することです。 
これは、クラスをインタフェースとして扱い、実装ではなく、使い捨てとアクティブ可な隠れた型だけを使用します。 
つまり、クラス内で実装を提供する必要があります。 
それ以外はミックスインを使って避けたいものです。

この要件を満たすために、私たちはミックスインから来るメンバーのためのスタンダードプロパティとそのタイプを作成します。 
これにより、コンパイラは実行時にこれらのメンバを使用できるようになります。 
これにより、たとえ何らかの帳簿上のオーバーヘッドがあっても、ミックスインの利益を得ることができます。

```typescript
// Disposable
isDisposed: boolean = false;
dispose: () => void;
// Activatable
isActive: boolean = false;
activate: () => void;
deactivate: () => void;
```

最後に、ミックスインをクラスにミックスし、完全な実装を作成します。

```typescript
applyMixins(SmartObject, [Disposable, Activatable]);
```

最後に、私たちのためにミキシングを行うヘルパー関数を作成します。 
これは、各ミックスインのプロパティを通って実行され、ミックスインのターゲットにコピーされ、それらのインプリメンテーションと共にスタンドインプロパティが埋められます。

```typescript
function applyMixins(derivedCtor: any, baseCtors: any[]) {
    baseCtors.forEach(baseCtor => {
        Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
            derivedCtor.prototype[name] = baseCtor.prototype[name];
        });
    });
}
```
