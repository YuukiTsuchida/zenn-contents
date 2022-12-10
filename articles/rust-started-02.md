---
title: "Rust入門2" # 記事のタイトル
emoji: "👶" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["rust"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# 基本

## プリミティブ型

### ユニット型
空を表す型でサイズも0バイトになる。  
空のタプル型で `()` で表現する。  
値を返さない関数の戻り値もユニット型になる。  

### 真理値
bool型  
値は `true`, `false` を持つ。

### 整数
| 型     | 符号  | ビット幅      |
| ----- | --- | --------- |
| i8    | あり  | 8         |
| u8    | なし  | 8         |
| i16   | あり  | 16        |
| u16   | なし  | 16        |
| i32   | あり  | 32        |
| u32   | なし  | 32        |
| i64   | あり  | 64        |
| u64   | なし  | 64        |
| i128  | あり  | 128       |
| u128  | なし  | 128       |
| isize | あり  | アドレスのビット幅 |
| usize | なし  | アドレスのビット幅 |

usize型は配列のインデックスや長さなどで使われている。  

#### 整数リテラル
整数リテラルはデフォルトでi32になる。  
型を指定する場合 `0u8` このようにサフィックスに型を書いて指定することもできる。  
整数リテラルにはアンダースコアを利用して読みやすい形で書くことができる。  

```
let n1 = 10_000; // i32型
let n2 = 0u8; // u8型
let n3 = -100_isize; // isize型


let n4 = 10;      
let n5 = n3 + n4; // ここで isize型 + n4 の計算をしているので型推論でn4もisize型になる
```

#### 10進数以外の数値
```
let n1 = 0xff; // 16進数
let n2 = 0o744; // 8進数
let n3 = 0b1010; // 2進数
```

#### アスキーコード
```
let n1 = b`A`; // ASCII文字'A'の文字コード65u8になる
```

### 少数
f32型とf64型がある。  
デフォルトはf64型になる。  
整数型と同様にサフィックスに型を書いて指定ができる。  
```
let f1 = 10.0; // f64
let f2 = 1_234.567f32; // f32
let f3 = 578.6E+77; // f64 指数指定
```

### 文字
Unicodeの1文字を持つ。  
charリテラルはシングルクォートで作る。  
1文字を表すのに英数でも4バイトを使う。  

```
let c1 = 'A'; // char型
let c2 = 'a';

assert!(c1.is_uppercase()); // 大文字か検証
let c3 = '0';
assert!(c3.is_digit(10)); // 10進数数字か検証

let c9 = '漢'; // マルチバイト文字も可能
```

### 参照
メモリ安全なポインタ  
`&T`, `&mut T` で定義する。

- &T  
不変参照　readonly
- &mut T
可変参照 read&write可能

```
let c1 = 'A';
let c1_ptr = &c1;       // イミュータブル参照

let mut n1 = 0;
let n1_ptr = &mut n1;  // ミュータブル参照
*n1_ptr = 100; // 参照外しをして値の変更
```

### 生ポインタ
メモリ安全ではないポインタ  
`*const T`, `*mut T` で定義する。  

- *const T  
不変参照　readonly
- *mut T
可変参照 read&write可能

生ポインタは他の言語とアドレスの受け渡しによる連携や所有権システムから外したい場合に利用する。  
参照外し、他のポインタへの型変換は `unsafe` ブロックに書く必要がある。

```
let c1 = 100;
let c1_ptr: *const char = &c1;
unsafe{
    println!(*c1_ptr);
}

let mut n1 = 0;
let n1_ptr: *mut i32 = &mut n1;
unsafe{
    println!(*n1_ptr);
}
```

### 関数ポインタ
関数を示すポインタ　サイズはusizeと同じになる。
`fn(T) -> T` で定義する。  
関数ポインタの場合型注釈は書く必要がある。  
書かない場合関数定義型として扱われる。 サイズは0バイト
関数ごとに異なる型になる。  
関数定義型は匿名型として扱われるが、プログラム中の型名として利用はできない。  

```
fn type_of<T>(_: T) -> String {
    let a = std::any::type_name::<T>();
    return a.to_string();
}

fn double(value: i32) -> i32 {
    value * 2
}

let f1: fn(i32) -> i32 = double;
println!("function pointer: {}", f1(5));

let f2 = double;
println!("function pointer: {}", type_of(f1)); // function pointer: fn(i32) -> i32
println!("function pointer: {}", type_of(f2)); // function pointer: firststep::double
```

### タプル
複数の型のデータを入れることができる。  
`(値, 値)` で定義する。  

#### 各要素のアクセス
**フィールド名**
`変数名.定数`でアクセスすることができる。  
定数はデータの先頭を0から始まる  

```
let t1 = (1, true);
let n = t1.0;

let mut t2 = (1, true);
t2.0 = 2;
```

**パターンマッチ**
`let (x1, x2) = tuple変数` とすることで、xにタプルの内容を束縛してくれる。  
要素を書き換える場合 `ref mut` を変数名の前に付ける。  

```
let t1 = (1, true);
let (n1, b1) = t1;

let t2 = (1, true);
let (n2, _) = t2; // 不要な値にはアンダースコアを使うと無視できる。

// 要素を書き換える場合
let mut t3 = (2, false);
let (ref mut n3, ref mut b3) = t3;
```

### 配列
メモリ上に連続した領域を確保する。  
`[値, 値]` で定義する。  
型指定をする場合 `let x: [型;要素数]` と書く。  
サイズを調べる場合 `x.len()` とすればサイズが返ってくる。  
要素に指定するにはCopyトレイトを実装する必要がある。  

#### 各要素のアクセス
**インデックス**
配列外にアクセスした場合panicになる。  
getメソッドを利用することで安全なアクセスが可能。  

```
let a1 = [1, 2, 3];

let a2: [f32; 3] = [1.1, 2.1, 3.1];
println!("{}", a1[0]);
println!("{}", a1.len());

println!("{}", a2[0]);

// 
let some = a1.get(0); // Option型で返る
println!("{:?}", some);
let none = a1.get(5); // 配列外ならNoneに
println!("{:?}", none);
```

**イテレータ**
配列の全要素に対してアクセスをする場合はイテレータを使う。  
`変数.iter()` で取得できる。 値を変更する場合 `変数.iter_mut()` にする。  

```
let a1 = [1, 2, 3];
for i in a1.iter() {
    print!("{}", *i);
}

let mut a3 = [1, 2, 3];
for i in a3.iter_mut() {
    *i += 1;
}
```

len, iter, iter_mutなどは配列の機能ではなく、slice型に暗黙的な変換が行われ呼び出される。
これ以外にもsliceのメソッドはありそれらも利用可能。  

### スライス
配列要素の範囲に効率よくアクセスするためのビューとして使う。  
連続したメモリ領域に同じ型の要素が並んでいるデータ構造であればどれでも対象にすることができる。  
不変参照(&[型])、可変参照(&mut [型])、Box(Box<型>)を経由してアクセスする。  
配列と違い実体を持たないので、型にサイズは不要  
インデックスアクセス以外にも `[begin..end]` の指定ができる。  
この時endが含まれないことに注意  

```
let a1 = [1, 2, 3];
// let s1 = &a1; 型を見る感じこれだと配列の参照
let s1: &[i32] = &a1; // これでsはスライスになる。

println!("{}", s1[0]);
println!("{:?}", a1.first()); // Some(1)が返る
println!("{:?}", a1.last()); // Some(3)が返る

let s2 = &a1[0..2];
let s3 = &a1[0..];
let s4 = &a1[..3];
println!("{:?}", s2); // [1, 2]
println!("{:?}", s3); // [2, 3]
println!("{:?}", s4); // [1, 2, 3]

// sort() と sort_ unstable() の 違い
// // sort() は 安定 ソート なので 同 順 な データ の ソート 前 の 順序 が ソート 後 も 保存 さ れる
// // soft_ unstable() は 安定 ソート では ない が、 一般的 に sort() より 高速
//

let (s7, s8) = s6.split_at_mut(3);
println!("{:?}", s7); // [1, 3, 3]
println!("{:?}", s8); // [5, 5, 6, 8, 9, 9]
```

### str
Unicodeの文字で構成された文字列。  
strはスライスでアクセスを行うため、文字列スライス型とも呼ばれる。  
型は `&str` になる。
ダブルクォートで作る文字列はstrになり、型は `&'static str` になる。  

lenメソッドを使った場合文字数ではなく、バイト数が返ってくる。
インデックスでのアクセス等に関しても文字ではなくバイトデータに対してアクセスすることになる。  

```
let str1 = "abcd";
println!("{}", str1);

let str2 = "abcd
efg"; // この場合dの後に改行が入る
println!("{}", str2);

let str3 = "abcd\
efg"; // エスケープを指定すれば改行は入らない
println!("{}", str3);

let str4 = r#"abc#d"\na"#; // r#""# でraw文字列リテラルになる
println!("{}", str4);

let str4 = r###"abcd"#\n"###; // r###""### とすれば "# が使える
println!("{}", str4);

let str5 = "あいうえお";
println!("{}", str5.len()); // 15になる

let str6 = "あ,い\nうえお";
let lines = str6.lines().next();

println!("{:?}", lines); // Some("あい")

let mut chars = str6.chars(); // char型のイテレータの取り出し
println!("{:?}", chars.next());

let mut chars_ind = str6.char_indices(); // char_indicesを使うと文字と位置のタプルになる
println!("{:?}", chars_ind.next()); // Some((0, 'あ'))
println!("{:?}", chars_ind.next()); // Some((3, ','))
```

## ユーザー定義型

### Box<T>
Box型はデータをヒープに移動し、その参照を保持する。  
Boxのデータを作るときは `Box::new(変数)` と書く。  
移動させた変数は破棄されて、アクセスをした場合コンパイルエラーになる。  
スコープから外れBox変数が破棄される時にヒープに作られたデータも破棄される。  

```
let t1 = (1, "aaa".to_string()); // (&str).to_string でメモリ確保した文字列になる
let b1 = Box::new(t1);

let (n1, s1) = &(*b1); // (*b1) で実体を取り出してその参照
println!("{}, {}", n1, s1);
println!("{:?}", b1);
```

### ベクタ
可変長配列 配列はスタックに確保するがベクタはデータをヒープに確保する。  
データを作るときは `vec![]` と書く。  
`Vec::with_capacity(数値)` で確保サイズの指定ができる。  

```
// ベクタ
let mut v1 = vec![1, 2, 3, 4];
println!("legth {}, capacity {}", v1.len(), v1.capacity());
v1.push(5);
println!("legth {}, capacity {}", v1.len(), v1.capacity());
v1.pop();
println!("legth {}, capacity {}", v1.len(), v1.capacity());

v1.remove(0);
println!("legth {}, capacity {}", v1.len(), v1.capacity());
for i in v1.iter() {
    println!("{}", *i);
}

v1.insert(0, 5);
println!("legth {}, capacity {}", v1.len(), v1.capacity());
for i in v1.iter() {
    println!("{}", *i);
}

let mut v2 = vec![6, 7, 8];
v1.append(&mut v2); // appendだとv2の中身を移動

let mut v2 = vec![6, 7, 8];
v1.extend_from_slice(&mut v2); // extend_from_sliceだとv2の中身をコピー

println!("v1: legth {}, capacity {}", v1.len(), v1.capacity());
for i in v1.iter() {
    println!("{}", *i);
}
println!("v2: legth {}, capacity {}", v2.len(), v2.capacity());

let mut empty: Vec<i32> = Vec::new(); // 空の配列

let mut v3 = Vec::with_capacity(10); // サイズは0で、メモリは10個分確保
println!("v3: legth {}, capacity {}", v3.len(), v3.capacity());
v3.insert(0, 1);
println!("v3: legth {}, capacity {}", v3.len(), v3.capacity());
for i in v3.iter() {
    println!("{}", *i);
}

// スライスで参照は可能
let s1: &[i32] = &v2[0..];
println!("{}", s1[0]);

// Box<[i32]> に変換サイズ変更ができないため必要最低限に切り詰められる
let b1 = v2.into_boxed_slice();
println!("{}", b1[0]);

// BoxからVecに変換
let v2 = b1.into_vec();
println!("{:?}", v2);
```

### String
str型と似た性質を持つヒープ領域にメモリ確保をした文字列。  
str型と違い文字の追加や削除などの操作が可能。　　

```
let s1 = "abcd".to_string(); // std::string::Stringなる
let mut s2 = format!("efghi");

println!("{}", type_of(&s2));
s2.push_str(&s1); // s2にs1の内容を追加
s2.push('あ'); // char型データを追加
println!("{}", s2);

println!("{}", s2.len());

// str型と同じくchar型とインデックスのタプルを返す
for i in s2.char_indices() {
    println!("{:?}", i);
}

let format_string = format!("{}", 1);
println!("{}", format_string);

let parse_test = "1".parse::<i32>(); // 文字列から数値への変換
if let Ok(v) = parse_test {
    println!("{}", v);
}
```

文字列はこれ以外にも
- CString, CStr  
C言語で使われるヌル終端文字列  
- OsString, OsStr  
OSのネイティブ文字列
- PathBuf, Path
パス文字列 OS間のパス区切りの差異吸収をしてくれる


### Range
スライスを作る時に利用した構文は範囲を利用している。  
`start..end`, `..end`, `start..`, `start..=end`  
と多数の指定がある。  

Rangeを使うことで特定回数ループをするコードも簡単に書ける

```
for i in 0..12 {
    println!("{}", i);
}
```


### オプション型
値があるのか不明な戻り値に使われる。  
C言語などはnullで表現しているがrustではOption型を利用する。  
値がある場合はSomeとなり、ない場合はNoneになる。  

```
let a1 = [1, 2, 3, 4];
let op1 = a1.get(0);

// 安全な値の取り出し
if let Some(value) = op1 {
    println!("{}", value);
}

// unwrapで値の取出し
// Noneに対して行うとpanic
let value2 = op1.unwrap();
println!("{}", value2);

let op2 = None;
let value3 = op2.unwrap_or_else(|| 0); // Noneの時にクロージャを実行
println!("{}", value3);


fn option() -> Option<i32> {
    let a1 = [1, 2, 3, 4];
    let op1 = a1.get(4)?; // ?で値を取り出す、Noneの場合はこの関数の戻り地もNoneになる
    let op2 = a1.get(1)?;

    Some(op1 + op2)
}
```

### リザルト型
呼び出しの結果エラーになる可能性を暗示する型  
OkとErrの２つの値がある。  
オプション型との違いはErrでは失敗した理由の情報を持てる。  

オプションで使った機能はリザルトでも同じように使える。
```
let parse_ok = "1".parse::<i32>(); // 文字列から数値への変換
println!("{:?}", parse_ok);

let parse_err = "a".parse::<i32>(); // 文字列から数値への変換
println!("{:?}", parse_err); // Err(ParseIntError { kind: InvalidDigit })になる
```

## 新しい型の定義

### エイリアス
型に別名をつけて利用できるようにする  
`type エイリアス = 型` で記述する。  


### 構造体
3種類の構造体が存在していて、それぞれ 
- 名前付きフィールド構造体
- タプル構造体
- ユニット構造体

の3つがある。

#### 名前付きフィールド構造体
アトリビュートに `#[derive(Debug)]` を追加すると  
formatやprintで自動出力ができるので便利  

```
struct 型名 {
    変数名: 型,
}
```
で定義する。  

```
#[derive(Debug)]
struct Human {
    age: i32,
    name: std::string::String,
}

let human = Human {
    age: 20,
    name: format!("human"),
};

println!("{}", human.age);
println!("{}", human.name);

// こんな初期化もできる
// 省略形の初期化の場合フィールド名を一致する変数を与える必要がある。
let age = 20;
let name = format!("human_init1");
let human_init1 = Human { name, age };

// この初期化はageは直接していして、nameはhuman_init1を利用する
// ただ、Stringなどムーブされるデータはコピーされないので注意
let human_init2 = Human {
    name: format!("human_init2"),
    ..human_init1
};

println!("{:?}", human_init1);
println!("{:?}", human_init2);

// Defaultアトリビュートを指定すると
// デフォルト値使える
#[derive(Default, Debug)]
struct DefaultStruct {
    param1: i32,
    param2: i32,
}

let default_struct1: DefaultStruct = Default::default();
println!("{:?}", default_struct1);

let default_struct2 = DefaultStruct {
    param1: 20,
    ..Default::default()
};
println!("{:?}", default_struct2);

// デフォルト値を指定したい場合
// Defaultトレイトを実装する
#[derive(Debug)]
struct DefaultStructTrait {
    param1: i32,
    param2: i32,
}

impl Default for DefaultStructTrait {
    fn default() -> Self {
        Self {
            param1: 1,
            param2: 2,
        }
    }
}

let default_struct_trait: DefaultStructTrait = Default::default();
println!("{:?}", default_struct_trait);
```


#### タプル構造体
名前の通りタプル構造体  
フィールドに名前を与えてないので変数へのアクセスはタプルと同じように `.0`の形になる。

タプル構造体の便利な使い方として、newtypeというデザインパターンがある。  
エイリアスだと元の型の別名として扱うだけなので、元の型の変数を代入するなどが可能になっている。  
タプル構造体を利用してコンパイラの型チェックを強化することができる。

```
#[derive(Debug)]
struct TupleStruct(i32, i32);

let tuple_struct = TupleStruct(1, 2);
println!("{}", tuple_struct.0);
println!("{}", tuple_struct.1);

// newtypeで型チェックを強化
type Int32 = i32;

#[derive(Debug)]
struct Struct1 {
    param1: Int32,
    param2: Int32,
}

let param1 = 1;
let param2 = 2;

// Struct1の初期化でi32の変数を使ってもOK
println!("{:?}", Struct1 { param1, param2 });

#[derive(Debug)]
struct NewType(i32);

#[derive(Debug)]
struct Struct2 {
    param1: Int32,
    param2: NewType,
}

// これはエラー
// println!("{:?}", Struct2 { param1, param2 });
println!(
    "{:?}",
    Struct2 {
        param1,
        param2: NewType(2)
    }
);
```


#### ユニット構造体
値を持たない構造体  
フィールドを持たないトレイトの実装をしたい時に使う  

```
struct UnitStruct;
println!("{}", std::mem::size_of::<UnitStruct>()); // サイズは0バイト
```


### 列挙型
rustのenumは定数だけではなく、各定義固有のデータをもたせることができる。  
Optionも列挙型で定義されていて、Someは値を持ちNoneは値を持たな様な差を作ることができる。  
データの持たない列挙型はisize型の整数値が割り当てられる。  

#### 定数のみ

```
// PartialEqを付けると == での比較が可能
#[derive(Debug, PartialEq)]
enum Enum {
    Zero,
    One,
    Two,
}

let enum_value = Enum::Zero;

match enum_value {
    Enum::Zero => println!("zero"),
    Enum::One => println!("one"),
    Enum::Two => println!("two"),
}

if enum_value == Enum::Zero {}

println!("{}", type_of(Enum::Zero));

enum EnumValueSet {
    Zero = 1,
    One = 2,
    Two = 3,
}
```

#### データを持つ列挙型
列挙型は整数値以外にも構造体と同じ構文でフィールドをもたせることができる。  
データを持つEnumはmatch式を利用してフィールドの値を別変数に束縛できる。  

```
#[derive(Debug, PartialEq)]
enum Enum {
    Zero,
    One { param1: i32, param2: i32 },
    Two,
}

let data = [
    Enum::Zero,
    Enum::One {
        param1: 0,
        param2: 1,
    },
    Enum::Two,
];

for i in data.iter() {
    match i {
        Enum::One { param1, param2 } => println!("param1:{}, param2:{}", param1, param2),
        _ => println!("{:?}", i),
    }
}
```

## 型変換

### 型キャスト
asを利用して行う明示的な型変換。  
桁あふれのチェックを行わない。  
as はスカラ型のみサポートしていて、タプルや構造体に対して行うことはできない。(Formトレイトを実装すると可能) 

```
let c1 = 'a';
let us1 = c1 as i8;

println!("{}", us1); // aの文字コード

let n1 = 300;
let us2 = n1 as i8;
println!("{}", us2); // 桁あふれはしないで44になる 300 % 256
```

### Transmute
std::mem::transmuteは明示的な型変換。  
ビット列のサイズが同じデータに対して行うことができる。  
asと違い型の情報のみを変換して、ビット列は同じ状態になるので f -> i32 みたいな変換をするとでかい値になったりする。  

```
let c1 = 'a';
//     let us1: i8 = unsafe { std::mem::transmute(c1) }; // バイトサイズが違うのでエラー
let n1: i32 = unsafe { std::mem::transmute(c1) }; // こっちは可能

println!("{}", n1);
```

## 構文
### use宣言
他のモジュールに定義されている要素を短く書ける。  
例えば型名を取得する `std::any::type_name` をuse宣言を利用することで `any::type_name` と書ける。

```
use std::any;

fn type_of<T>(_: T) -> String {
    let a = any::type_name::<T>();
    return a.to_string();
}
```

### 関数

```
fn 関数名(引数: 型) -> 戻り値の型 {
}
```
戻り値を書かない場合は強制的にunit型になる


### メソッド
構造体に定義された関数  
インスタンスに関連付けられている。
`impl 型` のスコープ内で関数を定義する。  
メソッドの場合必ず引数に `self` が必要でこれがインスタンスになる。  

```
#[derive(Debug)]
struct Vector2 {
    x: f32,
    y: f32,
}

impl Vector2 {
    fn length(&self) -> f32 {
        (self.x * self.x + self.y * self.y).sqrt()
    }

    fn add_x(&mut self, value: f32) {
        self.x += value;
    }
}
```

### 関連関数
これも構造体に定義された関数だが、メソッドと違い型に紐付けされている。  
定義もメソッドと同じだが、`self` を引数に取らなくすることで関連関数になる。

```
#[derive(Debug)]
struct Vector2 {
    x: f32,
    y: f32,
}

impl Vector2 {
    fn length(&self) -> f32 {
        (self.x * self.x + self.y * self.y).sqrt()
    }

    fn add_x(&mut self, value: f32) {
        self.x += value;
    }

    fn new() -> Vector2 {
        Vector2 { x: 0.0, y: 0.0 }
    }
}
```

### 分岐
**if式**

```
if a == 0 {
    println!("{} is zero", a);
} else if a % 2 == 0 {
    println!("{} is even", a);
} else {
    println!("{} is odd", a);
}
```

rustでのifは式なので値を返すこともできる。

```
let a = 12;

let result = if a == 0 { "even" } else { "odd" };
println!("{} is {}", a, result);
```

**if let式**
パターンを1つだけ書くことができる。  
主にResult型やOption型などで使う。

```
let mut s = String::new();
io::stdin().read_line(&mut s).ok();

// parseの戻り値がOkならスコープ内で結果値が利用できる
if let Ok(s_r) = s.trim().parse::<i32>() {
    let type_str = type_of(s_r);
    println!("{}: {}", s_r, type_str);
    a = s_r;
}
```



**match式**
```
let mut a = 12;

let mut s = String::new();
io::stdin().read_line(&mut s).ok();

if let Ok(s_r) = s.trim().parse::<i32>() {
    let type_str = type_of(s_r);
    println!("{}: {}", s_r, type_str);
    a = s_r;
}

match a {
    1 => println!("one"),
    10 => println!("ten"),
    _ => println!("some"),
};
```

matchもif同様に値を返すことができる。

```
let m_result = match a {
    1 => "a",
    10 => "b",
    _ => "z",
};
```

match式ではどのパターンにもマッチしないような結果があるとコンパイルエラーになる。  
どのパターンにもマッチしない場合 `_` を使って処理を行う。  

### 繰り返し

**loop式**  
終了条件のない無限ループとして動作する。  
loopから抜ける場合 `break` を使う。  
loop式から値を返す場合breakキーワードの後に値を記述する。

```
loop {
    break 1;
}
```

**while式**  
条件付きのループ。  
loop式と違い値を返すことができない。  
返す値は常に()となる。  

```
while true {

}
```

**while let式**  
パターンマッチを行い成功した時にループ本体を実行する。

```
let value = Some(10);
while let Some(v) = value {
    println!("{}", v);
}
```

**for式**  
ベクタのコレクションなどに繰り返しができる。  

```
let vector = vec!["one", "two", "three"];
for v in vector.iter() {
    println!("{}", v);
}
```

### クロージャ
Rustにおけるクロージャは無名関数になる。  

```
let f = |引数リスト| {
    // 処理
};
```
これが無名関数の定義になる。

```
let mut base = 1;
let add = |value| value + base;
println!("{}", add(10));

// base = 5; // コンパイルエラー
println!("{}", add(10));
```

このコードでコンパイルエラーになる原因は base を無名関数に渡しているため。  
クロージャーにbase変数を貸している状態なので、baseを使おうとエラーになる。  
貸すのではなくキーワードを使い値をコピーすれば回避ができる。  
moveキーワードを使うと所有権がクロージャに移る。ただ、Copyトレイトを持つ型はコピーをしてくれる。

```
let mut base = 1;
let add = move |value| value + base;
println!("{}", add(10));

base = 5;
println!("{}", add(10));
```

### モジュール
`mod`キーワードを使うことでモジュールごとに分割ができ、外部に公開・非公開ができる。  

```
mod math {
    #[derive(Debug)]
    pub struct Vector {
        pub x: f32,
        pub y: f32,
    }

    impl Vector {
        pub fn length(&self) -> f32 {
            (self.x * self.x + self.y * self.y).sqrt()
        }
    }
}

fn main() {
    let v = math::Vector { x: 5.0, y: 5.0 };
    println!("{:?} {}", v, v.length());
}
```

`mod`の中で`pub`とつけると公開のフィールドとなり、それ以外は非公開になる。  
structのメンバ変数を非公開にするとフィールド名指定での初期化はエラーになる。  
また、`pub`キーワードは`pub(公開先)`とすることで一部の場所に対してだけ公開にすることもできる。  

```
mod hoge {
    pub(crate) fn echo() {
        println!("hoge.echo");
        self::fuga::echo();
        self::fuga::piyo::t();
    }

    mod fuga {
        pub(super) fn echo() {
            println!("hoge.fuga.echo");
        }

        pub(super) mod piyo {
            pub(in crate::hoge) fn t() {
                super::echo();
                println!("hoge.fuga.piyo.echo");
            }
        }
    }
}

fn main() {
    crate::hoge::echo();
}
```

- crate  
ルートのモジュールを指す特別な名前  
`pub(crate)`とすると`crate::`でアクセスできる。
- super  
親のモジュールを指す  
- self
現在のスコープのモジュールを指す  
- in ~~
相対パスで指定して公開先の指摘ができる。  
あまり使わなそうな気もする。  

### ファイル分割
モジュールは別のファイルに切り出すことができる。  
src/new.rs というファイルをつくると`new`モジュールとして扱う。  
ディレクトリを使うことで階層を表現することができる。

**同階層**
```
// src/new.rs
pub fn test() {
    println!("new.test");
}

// src/main.rs
mod new;

fn main() {
    new::test();
}
```
main.rsと同じディレクトリにソースがある場合 `mod ファイル名` で使うことができる。  

**子階層**
サブディレクトリにソースがある場合はちょっと手間がかかる。  
ディレクトリ名と同じ名前のソースファイルを作り、そこにサブディレクトリに配置したモジュールの公開設定を行うひつようがある。  

```
// src/nest/nestfile.rs
pub fn test() {
    println!("nest.test");
}

// src/nest.rs
pub mod nestfile;

// src/main.rs
mod nest;

fn main() {
    nest::nestfile::test();
}
```

また、`use`を使うことで省略できる。  
```
mod nest;
use nest::nestfile;
fn main() {
    nestfile::test();
}
```