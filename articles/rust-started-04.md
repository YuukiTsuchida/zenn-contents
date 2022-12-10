---
title: "Rust入門4" # 記事のタイトル
emoji: "👶" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["rust"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# トレイト

## 基本的な使い方
型に対して実装するべきメソッドの定義をしたもの。  
`trait 名前 {定義}` で定義をする。  
トレイト内でメソッドは実装を書かないで `fn 名前(引数) -> 戻り値型;` で終わらせる。  
各型で実装するときは `impl トレイト名 for 型名 {実装}` で実装する。  

```
trait Output {
    fn print(&self);
    fn get_str() -> String;
}

struct Vector2 {
    x: f32,
    y: f32,
}

impl Output for Vector2 {
    fn print(&self) {
        println!("x: {}, y: {}", self.x, self.y);
    }

    fn get_str() -> String {
        "Vector2".to_string()
    }
}

fn main() {
    let vec2 = Vector2 { x: 1.0, y: 1.0 };

    vec2.print();
    println!("{}", Vector2::get_str());
}

```

## トレイト境界
```
fn call_print<V>(v: &V) {
    v.print();
}
```
ジェネリックを使い、こんな関数を定義した場合 V にはprint関数が存在しない場合もあるためコンパイルエラーになる。  
そういった場合トレイト境界を使い V に`トレイトを実装している型`といった制約を設定することで回避することができる。  
トレイト境界を設定する場合3種類の書き方がある。  

### ジェネリックのパラメータで指定
ジェネリック型を定義するときに`<ジェネリック型: トレイト名>`と定義する。

```
fn call_print<V: Output>(v: &V) {
    v.print();
}
```

### where
関数の型の後に`where ジェネリック型: トレイト名`を続ける。
```
fn call_print<V>(v: &V)
where
    V: Output,
{
    v.print();
}
```

### impl Trait構文
関数の引数の型を`impl トレイト名`とする。
```
fn call_print(v: &impl Output) {
    v.print();
}
```

### 複数の境界
境界を複数したい場合ジェネリックのパラメータで指定に+で複数していできる。  
```
fn call_print<V: Output + Output2>(v: &V) {
    v.print();
}
```

## トレイトの継承
トレイトは継承することもできる。  
継承先のトレイトを実装する場合継承するトレイトを全て実装する必要がある。(継承先で関数を実装するのはNG)  

```
trait Output {
    fn print(&self);
    fn get_str() -> String;
}

trait Output2: Output {
    fn execute(&self);
}

struct Test {
    x: f32,
    y: f32,
}

impl Output for Test {
    fn print(&self) {
        println!("x: {}, y: {}", self.x, self.y);
    }

    fn get_str() -> String {
        "Vector2".to_string()
    }
}

impl Output2 for Test {
    fn execute(&self) {}
}
```

## デフォルト実装
トレイトのメソッドはデフォルトの実装を持つことができる。  

```
struct Test {
    x: f32,
    y: f32,
}

trait Output {
    fn print(&self) {
        println!("test");
    }

    fn get_str() -> String {
        "".to_string()
    }
}

impl Output for Test {}
```

## 実装のルール
トレイトには実装場合、トレイトまたは型のどちらが一方の定義があるクレートで実装しないといけないというルールがある。
トレイトの定義をファイル分割して別クレートにした場合。  

1. クレートAでトレイト1を定義  
2. クレートBでトレイト1をStringに実装  
3. クレートCでトレイト1をStringに実装  

このような実装を許可した場合
Stringに関数を追加したことになるので、Stringの変数を用意すればトレイトの関数を呼び出せる。  
ただ、2,3で共にStringに対してトレイト1を実装していることになるのでどちらの関数が実行されるのか判断をすることができない。  

そのため、トレイトまたは型のどちらが一方の定義があるクレートにて実装をして曖昧の解消をルールとしている。  

## 自動導入
`#[derive(***)]` アトリビュートを使うといくつかのトレイトは自動実装ができる。  
標準ライブラリで導入可能なのは
- Clone  
`.clone()` でコピーをする
- Copy  
`A = B` でムーブをしないでコピーをする
- Debug  
ユーザー定義型のデバッグ出力を可能にする
- Default  
`Default::default()` 初期化用の関数を作らないで変数の初期化ができる
- Hash  
hash計算関数を追加
- PartialEq  
等価であることの比較を可能にする
- Eq  
等価比較が可能なことの保証をする。  
f32はNANがある関係で完全な等価比較ができない
- PartialOrd  
大小関係の比較を可能にする
- Ord  
大小関係の比較が可能なことの保証をする。  
f32はNANがある関係で完全な比較ができない

これらのderiveは可能な限りつけるのが良いとされる。


## トレイトのジェネリック
トレイトもジェネリックを持つことができる  

```
trait Output<T> {
    fn print(&self, t: T) -> &Self;
}

struct Test<T> {
    value: T,
}

impl<T> Output<T> for Test<T> {
    fn print(&self, t: T) -> &Self {
        self
    }
}
```

## ジェネリックの型パラメータの特殊化
実装で型を指定することで型毎に処理を分けることができる。 

```
trait Output<T> {
    fn print(&self, t: T) -> &Self;
}

struct Test<T> {
    value: T,
}

impl<T> Output<T> for Test<T> {
    fn print(&self, t: T) -> &Self {
        self
    }
}
```

## impl Trait
トレイト境界でimpl Traitに関して書いたが、戻り値に利用することもできる。  
引数のimpl Traitは全称impl Traitとも呼ばれる。  
戻り値のimpl Traitは存在impl Traitとも呼ばれる。  

存在impl Traitは戻り値の型が実装しているトレイトを書いて境界の指定をする。  
```
fn to_n(n: i32) -> impl Iterator {
    0..n
}
```

戻り値の型が複雑になるときに短くかけて嬉しい機能だが、デメリットもある。  
存在impl Traitが戻り値になっている関数を使用側では元の型情報が失われる。  

```
use std::fmt;

fn value() -> impl fmt::Display {
    1
}

fn main() {
    let t = value();

    //  binary assignment operation `+=` cannot be applied to type `impl std::fmt::Display` [Error]    
    t += 1;
}
```

value関数の戻り値はi32だが、`+=` を行うとエラーになる。  
また、関数の戻り値はトレイトを実装していればよいが、条件によって返す型が違うコードもエラーになる。  