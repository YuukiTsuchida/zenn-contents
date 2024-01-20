---
title: "Docker入門" # 記事のタイトル
emoji: "👶" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["docker"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# Docker入門

dockerの使い方を忘れないようにするための備忘録

## イメージの取得

他の人が作ったイメージは
[docker hub](https://hub.docker.com/)ここから検索してDLすることが可能となっている。  
docker hubをブラウザで開いて検索する以外にも

``` cmd
docker search リポジトリ(検索したいイメージ名)
```

で検索することもできる。

取得したいイメージが決まったら

``` cmd
docker pull リポジトリ[:tag]
```

で取得することができる。
Tagに関しては未指定でも取得が行える。
指定あり、なしの違いとしては下記のようになる。

Tagの指定あり：指定のバージョンのイメージを取得できる。
Tagの指定なし：現行の最新版を取得する。

取得したイメージを確認する場合

```cmd
docker images

# 以下出力
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              16.04               00fd29ccc6f1        10 days ago         111MB
ubuntu              latest              00fd29ccc6f1        10 days ago         111MB
```

## イメージからコンテナを起動する

イメージの取得ができたらそこからコンテナを起動する。

``` cmd
docker run ubuntu(image[:tag])
```

とすることでコンテナを起動することができる。  
ただ、このコマンドでコンテナを起動した場合動いているものがないためプロセスが即座に終了してしまう。  
`docker run [OPTIONS] IMAGE[:TAG] [COMMAND] [ARG...]`  
docker runコマンドはこのような形になっていて[COMMAND]を指定するとそのコマンドを実行してくれる。  

``` cmd
docker run ubuntu echo "hoge" # hoge
```

echo "hoge"を指定すると"hoge"と出力されてプロセスが終了する。  

## コンテナのシェルを起動する

コンテナを作成しても即座にプロセスが終了してしまう。  
そのため、

``` cmd
docker run -it ubuntu
```

とするとプロセスが終了しないで作成したコンテナのシェルを利用することができる。  
この方法でbashを起動してパッケージをインストールしても再度`docker run`を実行すると**別のコンテナとして扱われる。**  

## 作成済みのコンテナを確認する

``` cmd
docker ps -a

# 以下出力
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                         PORTS               NAMES
07725b38bcf5        ubuntu              "bash"              23 minutes ago      Exited (0) 5 seconds ago                           tender_lovelace
ff5321ba5d7a        ubuntu              "echo aaa"          About an hour ago   Exited (0) About an hour ago                       angry_payne
098f7e1231bb        ubuntu              "/bin/bash"         About an hour ago   Exited (0) About an hour ago                       practical_mirzakhani
```

とすることで今まで作ったコンテナを確認することができる。  
※`-a`オプションを指定しなかった場合起動中のプロセスが確認できる。

## 既存のコンテナを起動する

作成済みのコンテナを起動させる場合各コンテナにつけられたIDを指定する必要がある。
`docker ps`コマンドで出力されたCONTAINER IDがこの時指定するものになる。

``` cmd
docker start -i 07725b38bcf5(コンテナID)
```

とすることで先程作成ときにbashを起動したコンテナを起動してbashを実行することができる。  
※コンテナIDはフルネームで指定せずに省略することも可能になっている。最小だと頭の1文字を指定して起動することができる。  
頭の1文字目が被っている場合2文字目までと言った具合にユニークになる長さになれば良い様子