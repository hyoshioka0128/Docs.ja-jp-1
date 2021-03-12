---
title: Open Web Interface for .NET (OWIN) と ASP.NET Core
author: ardalis
description: ASP.NET Core が Open Web Interface for .NET (OWIN) をどのようにサポートしているかを確認します。確認することで、Web アプリケーションを Web サーバーから切り離すことができるようになります。
ms.author: riande
ms.custom: H1Hack27Feb2017
ms.date: 2/8/2021
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
uid: fundamentals/owin
ms.openlocfilehash: e476f3f62a250d960c809e5062b3bd8ca3a1940a
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587235"
---
# <a name="open-web-interface-for-net-owin-with-aspnet-core"></a>Open Web Interface for .NET (OWIN) と ASP.NET Core

作成者: [Steve Smith](https://ardalis.com/)、[Rick Anderson](https://twitter.com/RickAndMSFT)

ASP.NET Core:

* Open Web Interface for .NET (OWIN) をサポートします。
* `Microsoft.Owin.*` ([Katana](/aspnet/aspnet/overview/owin-and-katana/)) ライブラリの .NET Core と互換性のある置換があります。

OWIN により、Web アプリを Web サーバーから切り離すことが可能になります。 ミドルウェアをパイプラインで使用し、要求と関連する応答を処理するための標準的な方法を定義します。 ASP.NET Core アプリケーションとミドルウェアは、OWIN ベースのアプリケーション、サーバー、およびミドルウェアと相互運用できます。

OWIN には、さまざまなオブジェクト モデルを使用する 2 つのフレームワークを併用できる分離レイヤーがあります。 `Microsoft.AspNetCore.Owin` パッケージには 2 つのアダプター実装が用意されています。

* ASP.NET Core から OWIN へ 
* OWIN から ASP.NET Core へ

これにより、ASP.NET Core を OWIN 互換のサーバー/ホスト上でホストするか、他の OWIN 互換コンポーネントを ASP.NET Core 上で実行することができます。

> [!NOTE]
> これらのアダプターを使用すると、パフォーマンスが低下します。 ASP.NET Core コンポーネントのみを使用するアプリケーションでは、`Microsoft.AspNetCore.Owin` パッケージまたはアダプターを使用しないでください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/owin/sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="running-owin-middleware-in-the-aspnet-core-pipeline"></a>ASP.NET Core パイプラインで OWIN ミドルウェアを実行する

ASP.NET Core の OWIN のサポートは、`Microsoft.AspNetCore.Owin` パッケージの一部として展開されます。 このパッケージをインストールすることで、OWIN のサポートをプロジェクトにインポートできます。

OWIN ミドルウェアは、`Func<IDictionary<string, object>, Task>` インターフェイスと特定のキー (`owin.ResponseBody` など) の設定を必須とする [OWIN 仕様](https://owin.org/spec/spec/owin-1.0.0.html)に準拠しています。 次の単純な OWIN ミドルウェアを実行すると "Hello World" が表示されます。

```csharp
public Task OwinHello(IDictionary<string, object> environment)
{
    string responseText = "Hello World via OWIN";
    byte[] responseBytes = Encoding.UTF8.GetBytes(responseText);

    // OWIN Environment Keys: https://owin.org/spec/spec/owin-1.0.0.html
    var responseStream = (Stream)environment["owin.ResponseBody"];
    var responseHeaders = (IDictionary<string, string[]>)environment["owin.ResponseHeaders"];

    responseHeaders["Content-Length"] = new string[] { responseBytes.Length.ToString(CultureInfo.InvariantCulture) };
    responseHeaders["Content-Type"] = new string[] { "text/plain" };

    return responseStream.WriteAsync(responseBytes, 0, responseBytes.Length);
}
```

サンプル署名は `Task` を返し、OWIN で必要な場合に `IDictionary<string, object>` を受け取ります。

次のコードは、`UseOwin` 拡張メソッドを使用して ASP.NET Core パイプラインに `OwinHello` ミドルウェア (上の図) を追加する方法を示しています。

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseOwin(pipeline =>
    {
        pipeline(next => OwinHello);
    });
}
```

OWIN パイプライン内で実行する他のアクションを構成できます。

> [!NOTE]
> 応答ヘッダーは、応答ストリームへの最初の書き込み前にのみ変更してください。

> [!NOTE]
> パフォーマンス上の理由から、`UseOwin` を複数回、呼び出すことは避けてください。 OWIN コンポーネントは、グループ化されている場合に最適に動作します。

```csharp
app.UseOwin(pipeline =>
{
    pipeline(next =>
    {
        return async environment =>
        {
            // Do something before.
            await next(environment);
            // Do something after.
        };
    });
});
```

<a name="hosting-on-owin"></a>

## <a name="run-aspnet-core-on-an-owin-based-server-and-use-its-websockets-support"></a>OWIN ベースのサーバー上で ASP.NET Core を実行し、その WebSockets のサポートを利用する

OWIN ベースのサーバーの機能を ASP.NET Core で利用する方法のもう 1 つの例として、WebSockets などの機能へのアクセスが挙げられます。 前の例で使用していた .NET OWIN Web サーバーは、ASP.NET Core アプリケーションから利用できる組み込みの Web Sockets をサポートしています。 Web Sockets をサポートし、WebSockets を介してサーバーに送信されたすべてをエコー バックする単純な Web アプリケーションの例を次に示します。

```csharp
public class Startup
{
    public void Configure(IApplicationBuilder app)
    {
        app.Use(async (context, next) =>
        {
            if (context.WebSockets.IsWebSocketRequest)
            {
                WebSocket webSocket = await context.WebSockets.AcceptWebSocketAsync();
                await EchoWebSocket(webSocket);
            }
            else
            {
                await next();
            }
        });

        app.Run(context =>
        {
            return context.Response.WriteAsync("Hello World");
        });
    }

    private async Task EchoWebSocket(WebSocket webSocket)
    {
        byte[] buffer = new byte[1024];
        WebSocketReceiveResult received = await webSocket.ReceiveAsync(
            new ArraySegment<byte>(buffer), CancellationToken.None);

        while (!webSocket.CloseStatus.HasValue)
        {
            // Echo anything we receive
            await webSocket.SendAsync(new ArraySegment<byte>(buffer, 0, received.Count), 
                received.MessageType, received.EndOfMessage, CancellationToken.None);

            received = await webSocket.ReceiveAsync(new ArraySegment<byte>(buffer), 
                CancellationToken.None);
        }

        await webSocket.CloseAsync(webSocket.CloseStatus.Value, 
            webSocket.CloseStatusDescription, CancellationToken.None);
    }
}
```

## <a name="owin-environment"></a>OWIN 環境

`HttpContext` を使用して OWIN 環境を構築できます。

```csharp

   var environment = new OwinEnvironment(HttpContext);
   var features = new OwinFeatureCollection(environment);
   ```

## <a name="owin-keys"></a>OWIN キー

OWIN は、HTTP 要求/応答の交換を通じて情報を伝達するために `IDictionary<string,object>` オブジェクトに依存しています。 ASP.NET Core は次のキーを実装しています。 [主な仕様、拡張機能](https://owin.org/#spec)に関するセクションと、「[OWIN Key Guidelines and Common Keys](https://owin.org/spec/spec/CommonKeys.html)」(OWIN キーのガイドラインと共通キー) を参照してください。

### <a name="request-data-owin-v100"></a>要求データ (OWIN v1.0.0)

| Key               | 値 (型) | 説明 |
| ----------------- | ------------ | ----------- |
| owin.RequestScheme | `String` |  |
| owin.RequestMethod  | `String` | |    
| owin.RequestPathBase  | `String` | |    
| owin.RequestPath | `String` | |     
| owin.RequestQueryString  | `String` | |    
| owin.RequestProtocol  | `String` | |    
| owin.RequestHeaders | `IDictionary<string,string[]>`  | |
| owin.RequestBody | `Stream`  | |

### <a name="request-data-owin-v110"></a>要求データ (OWIN v1.1.0)

| Key               | 値 (型) | 説明 |
| ----------------- | ------------ | ----------- |
| owin.RequestId | `String` | Optional |

### <a name="response-data-owin-v100"></a>応答データ (OWIN v1.0.0)

| Key               | 値 (型) | 説明 |
| ----------------- | ------------ | ----------- |
| owin.ResponseStatusCode | `int` | Optional |
| owin.ResponseReasonPhrase | `String` | Optional |
| owin.ResponseHeaders | `IDictionary<string,string[]>`  | |
| owin.ResponseBody | `Stream`  | |

### <a name="other-data-owin-v100"></a>その他のデータ (OWIN v1.0.0)

| Key               | 値 (型) | 説明 |
| ----------------- | ------------ | ----------- |
| owin.CallCancelled | `CancellationToken` |  |
| owin.Version  | `String` | |   

### <a name="common-keys"></a>共通キー

| Key               | 値 (型) | 説明 |
| ----------------- | ------------ | ----------- |
| ssl.ClientCertificate | `X509Certificate` |  |
| ssl.LoadClientCertAsync  | `Func<Task>` | |    
| server.RemoteIpAddress  | `String` | |    
| server.RemotePort | `String` | |     
| server.LocalIpAddress  | `String` | |    
| server.LocalPort  | `String` | |    
| server.IsLocal  | `bool` | |    
| server.OnSendingHeaders  | `Action<Action<object>,object>` | |

### <a name="sendfiles-v030"></a>SendFiles v0.3.0

| Key               | 値 (型) | 説明 |
| ----------------- | ------------ | ----------- |
| sendfile.SendAsync | 「[Delegate Signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm)」(デリゲート シグネチャ) を参照してください。 | 要求ごと |

### <a name="opaque-v030"></a>Opaque v0.3.0

| Key               | 値 (型) | 説明 |
| ----------------- | ------------ | ----------- |
| opaque.Version | `String` |  |
| opaque.Upgrade | `OpaqueUpgrade` | 「[Delegate Signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm)」(デリゲート シグネチャ) を参照してください。 |
| opaque.Stream | `Stream` |  |
| opaque.CallCancelled | `CancellationToken` |  |

### <a name="websocket-v030"></a>WebSocket v0.3.0

| Key               | 値 (型) | 説明 |
| ----------------- | ------------ | ----------- |
| websocket.Version | `String` |  |
| websocket.Accept | `WebSocketAccept` | 「[Delegate Signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm)」(デリゲート シグネチャ) を参照してください。 |
| websocket.AcceptAlt |  | 記述なし |
| websocket.SubProtocol | `String` | [RFC6455 のセクション 4.2.2](https://tools.ietf.org/html/rfc6455#section-4.2.2) の手順 5.5 を参照してください。 |
| websocket.SendAsync | `WebSocketSendAsync` | 「[Delegate Signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm)」(デリゲート シグネチャ) を参照してください。  |
| websocket.ReceiveAsync | `WebSocketReceiveAsync` | 「[Delegate Signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm)」(デリゲート シグネチャ) を参照してください。  |
| websocket.CloseAsync | `WebSocketCloseAsync` | 「[Delegate Signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm)」(デリゲート シグネチャ) を参照してください。  |
| websocket.CallCancelled | `CancellationToken` |  |
| websocket.ClientCloseStatus | `int` | Optional |
| websocket.ClientCloseDescription | `String` | Optional |

## <a name="additional-resources"></a>その他の技術情報

* [ミドルウェア](xref:fundamentals/middleware/index)
* [サーバー](xref:fundamentals/servers/index)
