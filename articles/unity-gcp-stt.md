---
title: "Unity + gRPCでCloud Speech-to-Textを利用する" # 記事のタイトル
emoji: "👶" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["csharp", "Unity"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# Cloud Speech-to-Textストリーミング入力をUnityで実現

## 概要

Cloud Speech-to-Textのストリーミング入力を利用するための行ったことの説明

## 問題点

- 1.Grpc.Core  
    Unityでも動くらしいがサポートが切れている。
- 2.grpc-dotnet  
    サポートの切れたGrpc.Coreの替わりになるピュアC#の実装になっている。
    C#でgRPCを実現するならこれでよいのだがUnity がサポートしている .NET のバージョンでは HTTP/2 の標準実装が利用できない。

## 解決策

CysharpさんがYetAnotherHttpHandlerというUnityで使えるHttp/2の通信実装を公開してくれている。（ありがとうございます。）  
この、パッケージは `HttpClient` の通信レイヤーを差し替えられる様に作られている。
そのため`grpc-dotnet`を組み合わせてgRPCを使ったGCPと通信ができるようにする。

## 必要なパッケージの追加

Cloud Speech-to-Textの利用するのに必要なパッケージを追加する

### 1. NuGet for Unityの追加

Google.Cloud.SpeechのパッケージなどNuGetで提供されているパッケージが必要になる。  
NuGet for Unityを導入する。

UPMで `https://github.com/GlitchEnzo/NuGetForUnity.git?path=/src/NuGetForUnity` を追加すればプロジェクトにNuGet for Unityが追加される。

### 2. NuGetから必要なパッケージを追加

- Grpc.Net.Client
- Grpc.Tools
- System.IO.Pipelines
- Google.Cloud.Speech.V2

`grpc-dotnet` + `YetAnotherHttpHandler` でCloud Speech-to-Textを実現するために必要となるこれらのパッケージプロジェクトに追加する。

### 3. YetAnotherHttpHandlerの追加

UPMで `https://github.com/Cysharp/YetAnotherHttpHandler.git?path=src/YetAnotherHttpHandler` を追加すればプロジェクトにYetAnotherHttpHandlerが追加される。  


## サンプルプロジェクト

https://github.com/YuukiTsuchida/gcp-stt-on-unity


### 注意点

Unityでない環境であれば `Google.Cloud.Speech.V2` が提供するデフォルトの使い方をすれば良い。
ただ、Unityで利用する場合は通信のレイヤーを差し替えてやる必要があるため以下の実装が必要になる。  

#### 通信レイヤーの差し替え

https://github.com/YuukiTsuchida/gcp-stt-on-unity/blob/main/Assets/Scripts/Sample.cs#L132-L151

デフォルトであれば `SpeechClientBuilder` のインスタンスを作成して `builder.Build` とするだけで良いが、通信レイヤーの差し替えるために認証のインスタンスとHttpClientのインスタンスを作成してこれらをセットにしたチャネルをSpeechClientBuilderに差し替える。  
これで `Google.Cloud.Speech.V2` は内部で行う通信を `YetAnotherHttpHandler`に依存してくれるのでUnity上でgRPCができるようになる。  