---
title: gRPC を使用したパフォーマンスのベスト プラクティス
author: jamesnk
description: 高パフォーマンスの gRPC サービスを構築するためのベスト プラクティスについて説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 08/23/2020
no-loc:
- appsettings.json
- ASP.NET Core Identity
- cookie
- Cookie
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: grpc/performance
ms.openlocfilehash: 5d19ace2e844f2159c1ba0e8bc92960bcf00d54e
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586996"
---
# <a name="performance-best-practices-with-grpc"></a>gRPC を使用したパフォーマンスのベスト プラクティス

作成者: [James Newton-King](https://twitter.com/jamesnk)

gRPC は、高パフォーマンス サービス向けに設計されています。 このドキュメントでは、gRPC から最大限のパフォーマンスを得る方法について説明します。

## <a name="reuse-grpc-channels"></a>gRPC チャネルを再利用する

gRPC の呼び出しを行う場合は、gRPC チャネルを再利用する必要があります。 チャネルを再利用すると、既存の HTTP/2 接続を介して、呼び出しを多重化できます。

gRPC 呼び出しごとに新しいチャネルが作成された場合、完了までにかかる時間が大幅に増加する可能性があります。 呼び出しのたびに新しい HTTP/2 接続が作成されるため、クライアントとサーバーの間に次のような複数のネットワーク ラウンドトリップが必要になります。

1. ソケットを開く
2. TCP 接続を確立する
3. TLS ネゴシエーションを行う
4. HTTP/2 接続を開始する
5. gRPC 呼び出しを行う

チャネルは、gRPC 呼び出しの間で安全に共有し、再利用できます。

* gRPC クライアントはチャネルを使用して作成されます。 gRPC クライアントは軽量なオブジェクトであり、キャッシュしたり再利用したりする必要はありません。
* 異なる種類のクライアントを含め、1 つチャネルから複数の gRPC クライアントを作成できます。
* チャネルおよびそのチャネルから作成されたクライアントは、複数のスレッドで安全に使用できます。
* チャネルから作成されたクライアントは、複数の同時呼び出しを行えます。

gRPC クライアント ファクトリには、チャネルを構成するための一元的な方法が用意されています。 基になるチャネルが自動的に再利用されます。 詳細については、「<xref:grpc/clientfactory>」を参照してください。

## <a name="connection-concurrency"></a>接続の同時実行

HTTP/2 接続では、通常、1 つの接続で[同時に実行できるストリームの最大数 (アクティブな HTTP 要求数)](https://httpwg.github.io/specs/rfc7540.html#rfc.section.5.1.2) に制限があります。 既定では、ほとんどのサーバーの同時に実行できるストリームの制限数は 100 に設定されます。

gRPC チャネルは 1 つの HTTP/2 接続を使用し、その接続で同時呼び出しが多重化されます。 アクティブな呼び出しの数が接続ストリームの制限に達すると、追加の呼び出しがクライアントのキューに入れられます。 キューに入れられた呼び出しは、アクティブな呼び出しが完了するのを待ってから送信されます。 高負荷のアプリケーションや、長時間実行されるストリーミング gRPC 呼び出しでは、この制限により、呼び出しキューが原因でパフォーマンスの問題が発生する可能性があります。

::: moniker range=">= aspnetcore-5.0"

.NET 5 により、`SocketsHttpHandler.EnableMultipleHttp2Connections` プロパティが導入されています。 `true` に設定すると、同時実行ストリームの制限に達したときに、チャネルによって追加の HTTP/2 接続が作成されます。 `GrpcChannel` が作成されると、その内部の `SocketsHttpHandler` が、追加の HTTP/2 接続を作成するように自動的に構成されます。 アプリによって独自のハンドラーが構成される場合は、`EnableMultipleHttp2Connections` を `true` に設定することを検討してください。

```csharp
var channel = GrpcChannel.ForAddress("https://localhost", new GrpcChannelOptions
{
    HttpHandler = new SocketsHttpHandler
    {
        EnableMultipleHttp2Connections = true,

        // ...configure other handler settings
    }
});
```

::: moniker-end

.NET Core 3.1 アプリには、2 つの回避策があります。

* 高負荷のアプリの領域に対して、別々の gRPC チャネルを作成します。 たとえば、`Logger` gRPC サービスに高い負荷がかかることがあります。 別個のチャネルを使用して、このアプリに `LoggerClient` を作成します。
* gRPC チャネルのプールを使用します。たとえば、gRPC チャネルの一覧を作成します。 `Random` は、gRPC チャネルが必要になるたびに一覧からチャネルを選択するために使用されます。 `Random` を使用すると、複数の接続で呼び出しがランダムに分散されます。

> [!IMPORTANT]
> この問題を解決するもう 1 つの方法は、サーバーの最大同時実行ストリーム制限を増やすことです。 Kestrel では、これは <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.MaxStreamsPerConnection> で構成されます。
>
> 最大同時実行ストリーム制限を増やすことはお勧めしません。 1 つの HTTP/2 接続でストリームが多すぎると、パフォーマンスの新たな問題が発生します。
>
> * ストリーム間のスレッド競合により、接続への書き込みが試行されます。
> * 接続パケットが失われると、TCP 層ですべての呼び出しがブロックされます。

## <a name="load-balancing"></a>負荷分散

gRPC では、一部のロード バランサーが効率よく動作しません。 L4 (トランスポート) ロードバランサーは、TCP 接続をエンドポイント間で分散することで、接続レベルで動作します。 この方法は、HTTP/1.1 を使用して行われた API 呼び出しを負荷分散する場合に適しています。 HTTP/1.1 で行われる同時呼び出しは異なる接続で送信されるため、エンドポイント間で負荷を分散することができます。

L4 ロード バランサーは接続レベルで動作するため、gRPC ではうまく機能しません。 gRPC は HTTP/2 を使用します。これにより、1 つの TCP 接続での複数の呼び出しが多重化されます。 その接続でのすべての gRPC 呼び出しが、1 つのエンドポイントに送られます。

gRPC を効果的に負荷分散するには、次の 2 つのオプションがあります。

* クライアント側の負荷分散
* L7 (アプリケーション) のプロキシ負荷分散

> [!NOTE]
> gRPC 呼び出しのみが、エンドポイント間で負荷分散できます。 ストリーミング gRPC 呼び出しが確立されると、ストリームを介して送信されるすべてのメッセージが 1 つのエンドポイントに送られます。

### <a name="client-side-load-balancing"></a>クライアント側の負荷分散

クライアント側の負荷分散では、クライアントがエンドポイントを認識します。 各 gRPC 呼び出しの場合は、呼び出しの送信先として、別のエンドポイントが選択されます。 クライアント側の負荷分散は、待機時間が重要な場合に適しています。 クライアントとサービスの間にプロキシが存在しないため、呼び出しはサービスに直接送信されます。 クライアント側の負荷分散には、使用可能なエンドポイントを各クライアントが追跡し続ける必要があるという欠点があります。

ルックアサイド クライアントの負荷分散は、負荷分散の状態を中央の場所に格納する手法です。 クライアントは、負荷分散の決定を行うときに使用する情報を中央の場所に定期的にクエリします。

`Grpc.Net.Client` は現在、クライアント側の負荷分散をサポートしていません。 [Grpc.Core](https://www.nuget.org/packages/Grpc.Core) は .NET でクライアント側の負荷分散が必要な場合に適しています。

### <a name="proxy-load-balancing"></a>プロキシの負荷分散

L7 (アプリケーション) プロキシは、L4 (トランスポート) プロキシよりも高いレベルで動作します。 L7 プロキシは HTTP/2 を認識し、複数のエンドポイントにわたる 1 つの HTTP/2 接続で、gRPC 呼び出しをプロキシに多重化することができます。 プロキシの使用は、クライアント側の負荷分散よりも簡単ですが、gRPC 呼び出しに待機時間を追加することができます。

使用できる L7 プロキシは多数あります。 次のようなオプションがあります。

* [Envoy](https://www.envoyproxy.io/) - 広く普及しているオープン ソース プロキシ。
* [Linkerd](https://linkerd.io/) - Kubernetes のサービス メッシュ。
* [YARP: リバース プロキシ](https://microsoft.github.io/reverse-proxy/) - .NET で記述されたプレビューのオープン ソース プロキシ。

::: moniker range=">= aspnetcore-5.0"

## <a name="inter-process-communication"></a>プロセス間通信

クライアントとサービス間の gRPC 呼び出しは、通常、TCP ソケットを介して送信されます。 TCP はネットワーク経由の通信に適していますが、クライアントとサービスが同じコンピューター上にある場合は、[プロセス間通信 (IPC)](https://wikipedia.org/wiki/Inter-process_communication) の方が効率的です。

同じマシン上のプロセス間の gRPC 呼び出しには、UNIX ドメイン ソケットや名前付きパイプなどのトランスポートを使用することを検討してください。 詳細については、「<xref:grpc/interprocess>」を参照してください。

## <a name="keep-alive-pings"></a>キープ アライブ ping

キープ アライブ ping を使用して、非アクティブな状態の間 HTTP/2 接続を維持することができます。 アプリでアクティビティが再開されるときに既存の HTTP/2 接続が準備できていると、接続の再確立による遅延が発生することなく、初期の gRPC 呼び出しをすばやく行うことができます。

キープ アライブ ping は <xref:System.Net.Http.SocketsHttpHandler> に構成されます。

```csharp
var handler = new SocketsHttpHandler
{
    PooledConnectionIdleTimeout = Timeout.InfiniteTimeSpan,
    KeepAlivePingDelay = TimeSpan.FromSeconds(60),
    KeepAlivePingTimeout = TimeSpan.FromSeconds(30),
    EnableMultipleHttp2Connections = true
};

var channel = GrpcChannel.ForAddress("https://localhost:5001", new GrpcChannelOptions
{
    HttpHandler = handler
});
```

上記のコードにより、非アクティブな状態の間 60 秒ごとにキープ アライブ ping をサーバーに送信するチャネルが構成されます。 ping により、サーバーと使用中のすべてのプロキシが、非アクティブが理由で接続が閉じられることがないように保証されます。

::: moniker-end

## <a name="streaming"></a>ストリーム

gRPC 双方向ストリーミングを使用して、高パフォーマンスのシナリオで単項 gRPC 呼び出しを置き換えることができます。 双方向ストリームが開始されると、メッセージのやり取りが、複数の単項 gRPC 呼び出しでメッセージを送信するよりも高速になります。 ストリーム メッセージは、既存の HTTP/2 要求にデータとして送信され、各単項呼び出しに対して新しい HTTP/2 要求を作成するオーバーヘッドがなくなります。

サービスの例:

```csharp
public override async Task SayHello(IAsyncStreamReader<HelloRequest> requestStream,
    IServerStreamWriter<HelloReply> responseStream, ServerCallContext context)
{
    await foreach (var request in requestStream.ReadAllAsync())
    {
        var helloReply = new HelloReply { Message = "Hello " + request.Name };

        await responseStream.WriteAsync(helloReply);
    }
}
```

クライアントの例:

```csharp
var client = new Greet.GreeterClient(channel);
using var call = client.SayHello();

Console.WriteLine("Type a name then press enter.");
while (true)
{
    var text = Console.ReadLine();

    // Send and receive messages over the stream
    await call.RequestStream.WriteAsync(new HelloRequest { Name = text });
    await call.ResponseStream.MoveNext();

    Console.WriteLine($"Greeting: {call.ResponseStream.Current.Message}");
}
```

パフォーマンス上の理由で単項呼び出しを双方向ストリーミングに置き換えることは高度な手法であり、多くの状況では適切ではありません。

ストリーミング呼び出しの使用は、次の場合に適しています。

1. 高スループットまたは短い待機時間が必要とされている。
2. gRPC と HTTP/2 が、パフォーマンスのボトルネックとして特定された。
3. クライアントのワーカーが、gRPC サービスを使用して通常のメッセージを送受信している。

単項ではなく、ストリーミング呼び出しを使用する場合は、さらに複雑になり、制限事項が追加されることに注意してください。

1. ストリームは、サービスまたは接続エラーによって中断される可能性があります。 エラーが発生した場合にストリームを再開するためのロジックが必要です。
2. `RequestStream.WriteAsync` は、マルチスレッドでは安全ではありません。 一度に 1 つのストリームに書き込むことができるメッセージは 1 つだけです。 1 つのストリームで複数のスレッドからメッセージを送信するには、メッセージをマーシャリングするための <xref:System.Threading.Channels.Channel%601> のようなプロデューサーまたはコンシューマー キューが必要です。
3. gRPC ストリーミング方式は、1 種類のメッセージの受信と 1 種類のメッセージの送信に制限されます。 たとえば、`rpc StreamingCall(stream RequestMessage) returns (stream ResponseMessage)` により、`RequestMessage` が受信され、`ResponseMessage` が送信されます。 `Any` と `oneof` を使用した不明なメッセージまたは条件付きメッセージに対する Protobuf のサポートにより、この制限を回避できます。

## <a name="send-binary-payloads"></a>バイナリ ペイロードの送信

バイナリ ペイロードは、`bytes` スカラー値型の Protobuf でサポートされています。 C# で生成されたプロパティにより、プロパティの型として `ByteString` が使用されます。

```protobuf
syntax = "proto3";

message PayloadResponse {
    bytes data = 1;
}  
```

`ByteString` インスタンスは、`ByteString.CopyFrom(byte[] data)` を使用して作成されます。 このメソッドでは、新しい `ByteString` と新しい `byte[]` が割り当てられます。 データは、新しいバイト配列にコピーされます。

`UnsafeByteOperations.UnsafeWrap(ReadOnlyMemory<byte> bytes)` を使用して `ByteString` インスタンスを作成すると、追加の割り当てとコピーを回避できます。

```csharp
var data = await File.ReadAllBytesAsync(path);

var payload = new PayloadResponse();
payload.Data = UnsafeByteOperations.UnsafeWrap(data);
```

`ByteString` の使用中に変更が行われないように、`UnsafeByteOperations.UnsafeWrap` を使用してもバイトはコピーされません。

`UnsafeByteOperations.UnsafeWrap` には、[Google.Protobuf](https://www.nuget.org/packages/Google.Protobuf/) バージョン 3.15.0 以降が必要です。
