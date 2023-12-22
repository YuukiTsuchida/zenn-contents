---
title: "MessagePipeの使い方" # 記事のタイトル
emoji: "👶" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["rust"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# MessagePipe

URL: https://github.com/Cysharp/MessagePipe?tab=readme-ov-file

## 概要

.NetやUnityで使えるハイパフォーマンスのPub/Subパターンの実装を提供するライブラリ

DIと併用することが前提で作られている。

基本の使い方はSignalと同じように inject されるタイミングで型に紐づくpublisher/subscriberを受け取って利用するが、DIに依存しないイベントも行える。

## 使い方

### 基本の使い方

```csharp
public class SampleInstaller : MonoInstaller<SampleInstaller>
{
    public override void InstallBindings()
    {
        var option = Container.BindMessagePipe();

        // 型の登録
        Container.BindMessageBroker<int>(option);

        Container.BindInterfacesAndSelfTo<Publisher>().AsSingle();
        Container.BindInterfacesAndSelfTo<Subscriber>().AsSingle();
    }
}

public class Publisher
{
    private readonly IPublisher<int> publisher;
    public Publisher(IPublisher<int> publisher)
    {
        this.publisher = publisher;
    }

    public void Pub()
    {
        this.publisher.Publish(10);
    }
}

public class Subscriber
{
    private readonly ISubscriber<int> subscriber;

    private IDisposable disposable;
    public Subscriber(ISubscriber<int> subscriber)
    {
        this.subscriber = subscriber;

        this.disposable =  this.subscriber.Subscribe((value) => UnityEngine.Debug.Log(value));
    }

    public void SubDispose()
    {
        // subscribe解除
        this.disposable.Dispose();
    }
}
```

### IDによる振り分け

型だけで振り分けだけではなくIDによるpublisherとsubscriberの振り分けをすることもできる

```csharp
public class SampleInstaller : MonoInstaller<SampleInstaller>
{
    public override void InstallBindings()
    {
        var option = Container.BindMessagePipe();

        Container.BindMessageBroker<Guid, int>(option);

        Container.BindInterfacesAndSelfTo<Publisher>().AsSingle();
        Container.BindInterfacesAndSelfTo<Subscriber>().AsSingle();
    }
}

public class Publisher
{
    private readonly IPublisher<Guid, int> publisher;
    public Publisher(IPublisher<Guid, int> publisher)
    {
        this.publisher = publisher;
    }

    public void Pub(Guid guid, int value)
    {
        this.publisher.Publish(guid, value);
    }
}

public class Subscriber
{
    private readonly ISubscriber<Guid, int> subscriber;

    private IDisposable disposable;
    public Subscriber(ISubscriber<Guid, int> subscriber)
    {
        this.subscriber = subscriber;
    }

    public void SubDispose()
    {
        // subscribe解除
        this.disposable.Dispose();
    }

    public void AddSubscribe(Guid guid)
    {
        this.disposable =  this.subscriber.Subscribe(guid, (value) => UnityEngine.Debug.Log($"{guid} {value}"));
    }
}
```

ID振り分けをする場合型が同じでも別として扱われるので全ての型を別にする必要がなくなる。

変更箇所は型を登録する `BindMessageBroker` と `IPublisher/ISubscriber` のジェネリックパラメータの第一引数にIDの型指定をするだけ。

System.Guidで指定をしたが他の型も利用できる。

### 非同期のメッセージ

async/await を利用したメッセージを送ることが可能。

async/awaitで利用する場合 `IPublisher/ISubscriber` で発行、購読していた型を `IAsyncPublusher/IAsyncSubscriber` に変更する必要がある。

```csharp
public class SampleInstaller : MonoInstaller<SampleInstaller>
{
    public override void InstallBindings()
    {
        var option = Container.BindMessagePipe();

        Container.BindMessageBroker<int>(option);

        Container.BindInterfacesAndSelfTo<Publisher>().AsSingle();
        Container.BindInterfacesAndSelfTo<Subscriber>().AsSingle();
    }
}

public class Publisher
{
    private readonly IAsyncPublisher<int> publisher;
    public Publisher(IAsyncPublisher<int> publisher)
    {
        this.publisher = publisher;
    }

    public async void Pub()
    {
        UnityEngine.Debug.Log($"publisher {DateTime.Now}");
        await this.publisher.PublishAsync(10);
        UnityEngine.Debug.Log($"publisher {DateTime.Now}");
    }
}

public class Subscriber
{
    private readonly IAsyncSubscriber<int> subscriber;

    private IDisposable disposable;
    public Subscriber(IAsyncSubscriber<int> subscriber)
    {
        this.subscriber = subscriber;

        this.disposable = this.subscriber.Subscribe(async (value, cancellationToken) => {
            UnityEngine.Debug.Log($"subscriber {DateTime.Now}");
            await UniTask.Delay(1000);
            UnityEngine.Debug.Log($"subscriber {DateTime.Now}");
        });
    }

    public void SubDispose()
    {
        // subscribe解除
        this.disposable.Dispose();
    }
}
```

型を変えればメッセージの発行をする関数を `PublishAsync` で `Subscribe` の終了を待つことができる。

また、コードは`IAsyncPublusher/IAsyncSubscriber` にしたが `IAsyncPublusher/ISubscriber` といった組み合わせにすることも可能。

この場合受け取る方が非同期になってないのでそのまま同期的に先に進むので特に意味はない。

複数の購読者がいて物によって同期か非同期か選択ができる。

IDに対応した `IAsyncPublusher/IAsyncSubscriber` もありそちらを利用する場合はIDによる振り分けと同じようにジェネリックパラメータの第一引数にIDの型指定をするだけ。

### Buffered

直近で使わなそうな気がするのでひとまずスキップ必要なら追記

### EventFactory

型ではなくインスタンスによるグルーピングを作る

直近で使わなそうな気がするのでひとまずスキップ必要なら追記

### フィルタ

メッセージをSubscribeした時に特定のフィルターを登録することができる。

フィルタを登録しておくと値の変換を行ったり、whereをかけてでSubscribeで登録処理を呼び出さないようにすることができる。

```csharp
public class ChangedValueFilter : MessageHandlerFilter<int>
{
    int value;
    public override void Handle(int message, Action<int> next)
    {
        UnityEngine.Debug.Log($"ChangedValueFilter {message}");
        next(message * 2);
    }
}

public class ConditionFilter : MessageHandlerFilter<int>
{
    int value;
    public override void Handle(int message, Action<int> next)
    {
        UnityEngine.Debug.Log($"ConditionFilter {message}");
        if (message > 20)
        {
            next(message * 2);
        }
    }
}

public class Publisher
{
    private readonly IPublisher<int> publisher;
    public Publisher(IPublisher<int> publisher)
    {
        this.publisher = publisher;
    }

    public void Pub()
    {
        this.publisher.Publish(10);
    }
}

public class Subscriber
{
    private ISubscriber<int> subscriber;

    private IDisposable disposable;
    public Subscriber(ISubscriber<int> subscriber)
    {
        this.subscriber = subscriber;

        this.disposable =  this.subscriber.Subscribe((value) => UnityEngine.Debug.Log(value), new ChangedValueFilter(){Order = 0}, new ConditionFilter(){Order = 1});
    }

    public void SubDispose()
    {
        // subscribe解除
        this.disposable.Dispose();
    }
}
```

`ChangedValueFilter` で値を2倍にして、 `ConditionFilter` では20以下の値が来た場合後続の処理がスキップされるようになっている。

`[Publisher.Pub](http://Publisher.Pub)` を呼び出すとログには

1. ChangedValueFilter 10
2. ConditionFilter 20

だけが表示され `Subscribe` の第一引数の処理は実行されなくなる。

また、フィルタを指定する時に order の指定をなくした場合後に登録したものから順番に実行される。

`this.subscriber.Subscribe((value) => UnityEngine.Debug.Log(value), new ChangedValueFilter(), new ConditionFilter());` 

このように Order を消すと `ConditionFilter` ⇒ `ChangedValueFilter` ⇒ 購読の処理 という流れで実行される。

`Publisher/Subscriber` と同様にフィルターにも非同期型が存在している。

非同期のフィルタを使う場合 `AsyncMessageHandlerFilter` を継承すればよい。（インターフェースも変わるので注意）

### グローバルメッセージ

基本の使い方はDIを併用してメッセージを送る用になっているのでイベントの範囲がInjectできる範囲に限定されるが、この範囲をプロジェクト全体に対してメッセージの発行と購読をすることができる。

```csharp
public class SampleInstaller : MonoInstaller<SampleInstaller>
{
    public override void InstallBindings()
    {
        var option = Container.BindMessagePipe();

        Container.BindMessageBroker<int>(option);

        Container.BindInterfacesAndSelfTo<Publisher>().AsSingle();
        Container.BindInterfacesAndSelfTo<Subscriber>().AsSingle();
        
        GlobalMessagePipe.SetProvider(Container.AsServiceProvider()); // Zenjectの場合これで利用が可能になる
        GlobalMessagePipe.GetSubscriber<int>().Subscribe((value) => UnityEngine.Debug.Log($"global message {value}")); // globalな購読者が追加される
    }
}

public class Publisher
{
    private readonly IPublisher<int> publisher;
    public Publisher(IPublisher<int> publisher)
    {
        this.publisher = publisher;
    }

    public void Pub()
    {
        this.publisher.Publish(10);
    }

    public void Pub2()
    {
        GlobalMessagePipe.GetPublisher<int>().Publish(100); // グローバルなメッセージ発行
    }
}

public class Subscriber
{
    private ISubscriber<int> subscriber;

    private IDisposable disposable;
    public Subscriber(ISubscriber<int> subscriber)
    {
        this.subscriber = subscriber;

        this.disposable =  this.subscriber.Subscribe((value) => UnityEngine.Debug.Log(value));
    }

    public void SubDispose()
    {
        // subscribe解除
        this.disposable.Dispose();
    }
}
```

利用するには `GlobalMessagePipe.SetProvider` を呼び出して初期化をする必要があるが、これでプロジェクト全体に対してのメッセージの発行と購読ができる。

このグローバルなメッセージでもフィルターをかけることができる。

Subscribeする時にやる方法と `AddGlobalMessageHandlerFilter` でフィルタを指定するなど、複数の方法がある。

`AddGlobalMessageHandlerFilter` の場合共通のインタスタンを使い回すことになり、Zenjectの場合はクラスの登録が必要となる。