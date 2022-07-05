---
title: "TypeScript で DesignPattern を学んでみる〜SingletonPattern編〜"
emoji: "🛠"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typescript", "designpattern", "singleton"]
published: false
---

# はじめに

「業務で TypeScript 覚えなあかんくなったわ〜」
「せっかくなら、TypeScript 以外でも使えるようなスキルも学びたいわ〜」
「せや、DesignPattern も一緒に勉強するか。一石二鳥やん」

そんな軽いノリで執筆していきたいと思います〜〜

# Singleton Pattern とは？

オブジェクト指向の文脈で**このクラスのインスタンスはただ 1 つだけ存在してほしい**、そんなシーンがあるそうです。

そんな願いを叶えるために、パターンとして抽象したものが Singleton Pattern です。

# 実装

[こちら](https://github.com/RikiyaOta/DesignPatternByTypeScript/tree/main/src/singletonPattern)に置いてます〜

## Singleton Class

```typescript:singleton.ts
export class Singleton {
  private static singleton: Singleton = new Singleton();

  private constructor() {
    console.log("Construct a instance!");
  }

  public static getInstance() {
    return Singleton.singleton;
  }
}
```

ポイントは**constructor を private にしていること**です。

こうすることで、誤ってプログラマーが複数のインスタンスを生成することを防ぐことができます。
Singleton クラスのインスタンスが欲しい場合は、`Singleton.getInstance()`という唯一のインターフェースを介してインスタンスを取得します。

`Singleton.getInstance()`は、常に static property にて生成されたインスタンスを常に返すため、Singleton インスタンスは 1 つだけ存在するという要求を実現しています。

## 動作確認

`main.ts`で実際に`Singleton.getInstance()`を複数回呼び出して、それらが同じインスタンスを返すことを確認してみましょう〜

```typescript:main.ts
import { Singleton } from "./singleton";

const singleton1 = Singleton.getInstance();
const singleton2 = Singleton.getInstance();

console.log("Start!");

if (singleton1 === singleton2) {
  console.log("singleton1 is the same instance with singleton2 !");
} else {
  console.log("singleton1 is the NOT same instance with singleton2...");
}

console.log("End!");
```

```shell
$ npx tsc
$ node src/singletonPattern/main.js
```

```
Construct a instance!
Start!
singleton1 is the same instance with singleton2 !
End!
```

お〜〜、同じインスタンスを返すことが確認できました！上手くできてるな〜〜。

試しに、`Singleton`クラスを以下のように変更してみましょうか。ただのしょうもない実験です。

```diff typescript
@@ -6,9 +6,9 @@ export class Singleton {
   }

   public static getInstance() {
-    return Singleton.singleton;
+    return new Singleton();
   }
 }
```

static property をガン無視して、コンストラクターを呼び出してます。最悪ですね。
これでは、実質コンストラクターが public になっていると言えると思います。

再度`main.ts`をコンパイルして実行してみると？

```
Construct a instance!
Construct a instance!
Construct a instance!
Start!
singleton1 is the NOT same instance with singleton2...
End!
```

たしかに、インスタンスが複数回生成されてしまっていますね〜〜。やはり、コンストラクターを private に保っておくのは重要ですね〜〜。

# Singleton Pattern の応用例

## Logger

アプリケーションにおけるログ出力を担うインスタンスを Singleton として実装するのがひとつ有力な例だそう。

たしかに、ファイルにログを出力することを考えると、複数の Logger instance が同時に同じファイルにアクセスして書き込みをするのは排他制御のこととか考え始めて面倒ですね〜

## Configuration file

アプリケーションにおいて、設定ファイルを何度も読みにいくのはパフォーマンス的にあまり嬉しくないですね。ディスクアクセスなので。

設定ファイルをメモリに読み込んで保持しておくためのクラスを Singleton で実装しておくと、

- アプリはこのインスタンスに対して、設定値の取得を要求する
- Singleton はインスタンス生成時にファイルから情報をメモリ上に読み込んでいる
- メモリ上のデータを呼び出しもとに返すだけなので、パフォーマンス上のメリットがある

という嬉しさがありますね〜

同じ要領で cache を Singleton で実装するパターンもあるみたいです。ふむふむ。

# 所感

実装面では、**何を公開しない(public にしない)か**をきちんと設計することの重要性を学べました。

利用例では、排他制御など、複数スレッド・プロセスを用いるときの実装の際に、考え方の引き出しとして持っておくと良さそうだなということを学べました〜

一方で、Singleton クラスを実際に使うと、グローバル変数的な扱いになりそうだな〜という点や、インスタンス内で状態を持つことで、Unit Test がやりにくくなる可能性があるといった点がデメリットとしてあるみたいですね。使う際にはその辺りもしっかり検討した上で実装するようにせなあきまへんな。

# 参考

- [増補改訂版 Java 言語で学ぶデザインパターン入門](https://www.amazon.co.jp/%E5%A2%97%E8%A3%9C%E6%94%B9%E8%A8%82%E7%89%88Java%E8%A8%80%E8%AA%9E%E3%81%A7%E5%AD%A6%E3%81%B6%E3%83%87%E3%82%B6%E3%82%A4%E3%83%B3%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3%E5%85%A5%E9%96%80-%E7%B5%90%E5%9F%8E-%E6%B5%A9/dp/4797327030)
- [GeeksforGeeks | Singleton Design Pattern](https://www.geeksforgeeks.org/singleton-design-pattern-introduction/)
