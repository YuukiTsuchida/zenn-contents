---
title: "Rust入門1" # 記事のタイトル
emoji: "👶" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["rust"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## インストール
[公式](https://www.rust-lang.org/ja)を見るのが良さそう  
インストールをするとホームディレクトリ配下に `.rustup` と `.cargo` というディレクトリができる。  
- .rustup  
ツールチェインの本体
- .cargo  
rustc, cargo, rustupなどのコマンドが格納されている

**※windowsの場合先にVisual StudioのC++開発ツールのインストールをしてください**

### リンカーのインストール
- Linux  
gccのインストール
- mac  
コマンドライン・デベロッパ・ツールのインストール

をそれぞれ行う。  
windowsはVisual Studioをインストールをすればついて来るので特別やることはない


### インストールをして使えるようになるコマンドのバージョンを確認
```
rustup --version

rustc --version

cargo --version
```

---

## 更新
`rustup update` で本体の更新ができる。

## パッケージ作成
`cargo new` コマンドを作ってパッケージを作ることができる。  
パッケージの種類はバイナリとライブラリを作る2種類ある。  
`cargo new` コマンドを実行するときに指定することで、必要な設定をしてくれる。

### バイナリとライブラリの指定
- バイナリ  
ビルドをして、実行ファイルの出力をしてくれる。  
`cargo new --bin <パッケージ名>` この様なコマンドになる  
- ライブラリ  
ライブラリファイルを出力してくれる。  
`cargo new --lib <パッケージ名>` この様なコマンドになる  

例えばhelloworldとコンソールに出力するプロジェクトを作る場合

```
cargo new --bin helloworld
```
となる

### ビルド
`cargo build` でビルドができる。  
ビルドをすると `<package root>/target/debug` に成果物が出力される。  
`cargo build --release` とするとリリースビルドになり、出力先は`<package root>/target/release` になる。  

### その他コマンド
- 実行  
`cargo run` で実行ができる。ビルドをしていなくても実行ができる
- クリーン  
`cargo clean` で`<package root>` 配下にあるtargetディレクトリを削除する  

## 開発環境
neovimならcocの拡張で[coc-rust-analyzer](https://github.com/fannheyward/coc-rust-analyzer)があるのでこれを使うのがよさそう。  
vscodeでも同様にrust-analyzerを利用するパッケージがある。(試してない)  
共にrust-analyzerを利用してのコード補完をしてくれる。

### rust-analyzerのインストール
coc-rust-analyzerをインストール後にneovimを立ち上げると `rust-analyzer` が存在しないのでインストールをするかと聞かれるがインストールしてくれなかったので手動のインストール方法も書いておく  

[参照](https://rust-analyzer.github.io/manual.html#rust-analyzer-language-server-binary)
```
$ curl -L https://github.com/rust-analyzer/rust-analyzer/releases/latest/download/rust-analyzer-linux -o ~/.local/bin/ 
$ chmod +x ~/.local/bin/rust-analyzer
```
以上でrust-analyzerをインストールができる。  
nvimの場合`:CocConfig` で設定ファイルを開いてから `rust-analyzer.serverPath` の指定を行う。  

### デバッガ
#### vscode

#### ターミナル
LLDB, GDBを利用する。  
これらをインストールした状態でrustツールチェインに含まれる。rust-lldb, rust-gdbを使いデバッガを起動する。

**※v1.46.0での注意**
v1.46.0でrust-lldbを実行したら `file specified in --source (-s) option doesn't exist:` というエラーがでた。  
[解決方](https://github.com/rust-lang/rust/issues/76006#issuecomment-682255665) の手順で起動に成功。
パッケージを作るたびに
```
$ ln -s ~/repos/rust/src/etc/lldb_commands ./
$ ln -s ~/repos/rust/src/etc/lldb_providers.py ./
$ ln -s ~/repos/rust/src/etc/lldb_lookup.py ./
$ ln -s ~/repos/rust/src/etc/lldb_batchmode.py ./
```
これを行う。

## 命名規則
- 小文字スネークケース  
関数、変数、モジュール
- 大文字スネークケース  
定数、グローバル変数
- キャメルケース  
ユーザー定義型、ジェネリックス、トレイト

この命名規則に違反をしている場合コンパイルをした時に警告が表示される。  
無視をしたい場合アトリビュートを利用して警告の抑制ができる。  

- #[allow(non_snake_case)]  
関数、変数、モジュール
- #[allow(non_upper_came_globals)]  
定数、グローバル変数
- #[allow(non_camel_case_types)]  
ユーザー定義型、ジェネリックス、トレイト

## コーディング規約
[Rust Style Guide](https://github.com/rust-dev-tools/fmt-rfcs/blob/master/guide/guide.md)を参照  
rustには `rustfmt` というフォーマットの整形をしてくれるツールがあるのでこれを動かせば良い。  
