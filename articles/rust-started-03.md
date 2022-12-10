---
title: "Rust入門3" # 記事のタイトル
emoji: "👶" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["rust"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---


Rustは1つのリソースに対して単一の所有者を持たせ、必要な時に借用（一時的な貸与）が基本となっている。  
ただ、構成によっては対応しきれないことがある。  
そういった場合に活用できるデータ構造の紹介  


# 共同所有者
標準ライブラリに複数の所有者をもたせることができるポインタ`Rc<T>`と`Rrc<T>`がある。

## Rc<T>
Rcポインタを作ると対象のリソースはヒープ領域に格納され、参照カウンタによる管理が行われる。  

```
{
    let rc = std::rc::Rc::new(1);
    let rc2 = rc.clone();

    println!(
        "rc count{} rc2 count{}",
        std::rc::Rc::strong_count(&rc),
        std::rc::Rc::strong_count(&rc2)
    );
}

{
    let mut rc = std::rc::Rc::new(1);
    if let Some(v) = std::rc::Rc::get_mut(&mut rc) {
        *v = 5;
    }
    println!("{}", rc);

    {
        let rc2 = rc.clone();
        // 参照カウントが2以上の場合の場合Noneが返って来て値の変更ができない
        println!("{:?} {}", std::rc::Rc::get_mut(&mut rc), rc2);
    }

    // weakポインタだと参照カウントは増えない
    let weak = std::rc::Rc::downgrade(&rc);
    println!("rc count{}", std::rc::Rc::strong_count(&rc),);

    // weakポインタで共有している場合も値の変更ができない
    if let Some(v) = std::rc::Rc::get_mut(&mut rc) {
        *v = 10;
        println!("{}", rc);
    }

    {
        // 参照カウントを増やして共有する
        let rc2 = weak.upgrade().unwrap();
        println!("rc2 count{}", std::rc::Rc::strong_count(&rc2));
    }

    std::mem::drop(rc);

    // Rcが解放しているのでNoneになる
    println!("{:?}", weak.upgrade());
}
```

## Arc<T>
Rcと似ているがこちらはSyncトレイトを実装しているため複数のスレッドで共有ができる。  
ただ、オーバーヘッドが増え速度は遅い。  
モジュールはRcと違い `std::sync::Arc` になる。  

# 内側のミュータビリティ
コンパイル時の借用チェックを迂回してデータを可変する仕組み。  
Rcを使ってcloneした変数がに2つ以上存在する場合は値を変更できないが、この仕組を利用すると変更できるようになる。  

```
let rc = std::rc::Rc::new(std::cell::RefCell::new(1));
// RefCellで包まれているの
println!("{:?}", rc);

// 値を取り出す場合
// borrowで不変の値を取り出す
println!("{:?}", rc.borrow());

// 値を変更する場合
// borrow_mut 可変の参照を取り出す
*rc.borrow_mut() = 5;
println!("{:?}", rc.borrow());

let rc2 = rc.clone();
*rc2.borrow_mut() = 10;
println!("{:?}", rc.borrow());
```

この様にすることでRcやArcで参照カウントが2以上でも変更できるようになる。  
ただ、borrow_mutを行ったとき、他の参照が有効の場合クラッシュする。  
クラッシュしないようにする場合 `try_borrow_mut` を使う。  
これを使うとResult型の値を返し、成否のチェックが行える。  

```
if let Ok(v) = rc2.try_borrow_mut() {
    println!("ref OK {}", v);
}
```

内側のミュータビリティもRc, Arcのようにシングルスレッド、マルチスレッド向けのものがある。

## シングルスレッド
- UnsafeCell   
安全装置がない。getでポインタを取得して内部の値を書き換える
- Cell  
UnsafeCellをラップして作られていて、オーバーヘッドがない。set関数を使って内部で値を書き換える。
- RefCell  
参照を得ることができる。ただ、借用ルールの検証があるためオーバーヘッドがある。

## マルチスレッド
- Mutex
- RxLock
- AtomicUsize
- AtomicI32

など