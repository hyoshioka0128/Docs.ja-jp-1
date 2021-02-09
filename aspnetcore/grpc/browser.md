---
title: ブラウザー アプリでの gRPC の使用
author: jamesnk
description: ASP.NET Core で、GRPC-Web を使用してブラウザー アプリから呼び出しできるように、gRPC サービスを構成する方法について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 06/30/2020
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
uid: grpc/browser
ms.openlocfilehash: 0967a70b498156d9c4ea8818ee1c80b37d9f2d87
ms.sourcegitcommit: 7e394a8527c9818caebb940f692ae4fcf2f1b277
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/31/2021
ms.locfileid: "99217480"
---
# <a name="use-grpc-in-browser-apps"></a>ブラウザー アプリでの gRPC の使用

作成者: [James Newton-King](https://twitter.com/jamesnk)

 既存の ASP.NET Core の gRPC サービスを、[gRPC-Web](https://github.com/grpc/grpc/blob/2a388793792cc80944334535b7c729494d209a7e/doc/PROTOCOL-WEB.md) プロトコルを使用してブラウザー アプリから呼び出せるように構成する方法について説明します。 gRPC-Web を使用すると、ブラウザーの JavaScript および Blazor アプリで gRPC サービスを呼び出せます。 ブラウザーベースのアプリから HTTP/2 gRPC サービスを呼び出すことはできません。 ASP.NET Core でホストされている gRPC サービスは、HTTP/2 gRPC と共に gRPC-Web をサポートするように構成できます。


既存の ASP.NET Core アプリに gRPC サービスを追加する手順については、「[ASP.NET Core アプリに gRPC サービスを追加する](xref:grpc/aspnetcore#add-grpc-services-to-an-aspnet-core-app)」を参照してください。

gRPC プロジェクトを作成する手順については、「<xref:tutorials/grpc/grpc-start>」を参照してください。

## <a name="grpc-web-in-aspnet-core-vs-envoy"></a>ASP.NET Core の gRPC-Web と Envoy

gRPC-Web を ASP.NET Core アプリに追加する方法には、次の 2 つの選択肢があります。

* ASP.NET Core の gRPC HTTP/2 と共に、gRPC-Web をサポートします。 このオプションでは、`Grpc.AspNetCore.Web` パッケージによって提供されるミドルウェアを使用します。
* [Envoy プロキシ](https://www.envoyproxy.io/)の gRPC-Web サポートを使用して、gRPC-Web を gRPC HTTP/2 に変換します。 変換された呼び出しが、ASP.NET Core アプリに転送されます。

それぞれの方法には、長所と短所があります。 アプリの環境で既にプロキシとして Envoy を使用している場合は、gRPC-Web サポートを提供するために Envoy を使用するのも、妥当かもしれません。 ASP.NET Core のみを必要とする gRPC-Web の基本的なソリューションについては、`Grpc.AspNetCore.Web` を選択することをお勧めします。

## <a name="configure-grpc-web-in-aspnet-core"></a>ASP.NET Core での gRPC-Web の構成

ASP.NET Core でホストされている gRPC サービスは、HTTP/2 gRPC と共に gRPC-Web をサポートするように構成できます。 gRPC-Web では、サービスを変更する必要はありません。 唯一の変更点はスタートアップ構成です。

ASP.NET Core gRPC サービスで gRPC-Web を有効にするには:

* [Grpc.AspNetCore.Web](https://www.nuget.org/packages/Grpc.AspNetCore.Web) パッケージへの参照を追加します。
* アプリで gRPC-Web を使用するように構成するには、`UseGrpcWeb` と `EnableGrpcWeb` を *Startup.cs* に追加します。

[!code-csharp[](~/grpc/browser/sample/Startup.cs?name=snippet_1&highlight=10,14)]

上記のコードでは次の操作が行われます。

* GRPC-Web ミドルウェアの `UseGrpcWeb` を、ルーティングの後かつエンドポイントの前に追加します。
* `endpoints.MapGrpcService<GreeterService>()` メソッドで、`EnableGrpcWeb` を使用して gRPC-Web をサポートすることを指定します。 

または、gRPC-Web ミドルウェアを構成して、すべてのサービスで既定で gRPC-Web がサポートされ、`EnableGrpcWeb` が不要になるようにすることもできます。 ミドルウェアを追加するときに `new GrpcWebOptions { DefaultEnabled = true }` を指定します。

[!code-csharp[](~/grpc/browser/sample/AllServicesSupportExample_Startup.cs?name=snippet_1&highlight=12)]

> [!NOTE]
> .NET Core 3.x で [HTTP.sys によりホストされている](xref:fundamentals/servers/httpsys)場合は、gRPC-Web が失敗する原因となる既知の問題があります。
>
> HTTP.sys で gRPC-Web を機能させるには、[こちら](https://github.com/grpc/grpc-dotnet/issues/853#issuecomment-610078202)で回避策を参照してください。

### <a name="grpc-web-and-cors"></a>gRPC-Web と CORS

ブラウザーのセキュリティにより、Web ページを提供したドメインと異なるドメインに対して、Web ページが要求を行うことはできません。 この制限は、ブラウザー アプリで gRPC-Web 呼び出しを行う場合に適用されます。 たとえば、`https://www.contoso.com` によって提供されるブラウザー アプリでは、`https://services.contoso.com` でホストされている gRPC-Web サービスの呼び出しがブロックされます。 この制限を緩和するには、クロス オリジン リソース共有 (CORS) を使用できます。

ブラウザー アプリでクロス オリジン gRPC-Web 呼び出しを行えるようにするには、[ASP.NET Core で CORS](xref:security/cors) を設定します。 組み込みの CORS サポートを使用し、<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithExposedHeaders%2A> で gRPC 固有のヘッダーを公開します。

[!code-csharp[](~/grpc/browser/sample/CORS_Startup.cs?name=snippet_1&highlight=5-11,19,24)]

上記のコードでは次の操作が行われます。

* `AddCors` を呼び出して CORS サービスを追加し、gRPC 固有のヘッダーを公開する CORS ポリシーを構成します。
* `UseCors` を呼び出して、ルーティングの後かつエンドポイントの前に CORS ミドルウェアを追加します。
* `endpoints.MapGrpcService<GreeterService>()` メソッドが `RequiresCors` で CORS をサポートすることを指定します。

### <a name="grpc-web-and-streaming"></a>gRPC-Web とストリーミング

従来の HTTP/2 による gRPC では、すべての方向でストリーミングがサポートされます。 gRPC-Web では、ストリーミングのサポートが制限されています。

* gRPC-Web ブラウザー クライアントでは、クライアント ストリーミング メソッドと双方向ストリーミング メソッドの呼び出しはサポートされていません。
* Azure App Service および IIS でホストされている ASP.NET Core gRPC サービスでは、双方向ストリーミングはサポートされていません。

gRPC-Web を使用するときは、単項メソッドとサーバー ストリーミング メソッドのみを使用することをお勧めします。

## <a name="call-grpc-web-from-the-browser"></a>ブラウザーから gRPC-Web を呼び出す

ブラウザー アプリでは gRPC-Web を使用して gRPC サービスを呼び出すことができます。 ブラウザーから gRPC-Web を使用して gRPC サービスを呼び出す場合、いくつかの要件と制限があります。

* サーバーは、gRPC-Web をサポートするように構成されている必要があります。
* クライアント ストリーミングと双方向ストリーミングの呼び出しはサポートされていません。 サーバー ストリーミングはサポートされています。
* 別のドメインで gRPC サービスを呼び出すには、サーバーで [CORS](xref:security/cors) を構成する必要があります。

### <a name="javascript-grpc-web-client"></a>JavaScript gRPC-Web クライアント

JavaScript gRPC-Web クライアントがあります。 JavaScript から gRPC-Web を使用する方法については、[gRPC-Web を使用した JavaScript クライアント コードの作成](https://github.com/grpc/grpc-web/tree/master/net/grpc/gateway/examples/helloworld#write-client-code)に関するページを参照してください。

### <a name="configure-grpc-web-with-the-net-grpc-client"></a>.NET gRPC クライアントを使用して gRPC-Web を構成する

gRPC-Web 呼び出しを行うように .NET gRPC クライアントを構成できます。 これは、ブラウザーでホストされ、JavaScript コードの同じ HTTP 制限がある [Blazor WebAssembly](xref:blazor/index#blazor-webassembly) アプリに役立ちます。 .NET クライアントによる gRPC-Web の呼び出しは、[HTTP/2 gRPC と同じ](xref:grpc/client)です。 唯一の変更点は、チャネルの作成方法です。

gRPC-Web を使用するには:

* [Grpc.Net.Client.Web](https://www.nuget.org/packages/Grpc.Net.Client.Web) パッケージへの参照を追加します。
* [Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client) パッケージへの参照が確実に 2.29.0 以上であるようにします。
* `GrpcWebHandler` を使用するようにチャネルを構成します。

[!code-csharp[](~/grpc/browser/sample/Handler.cs?name=snippet_1)]

上記のコードでは次の操作が行われます。

* gRPC-Web を使用するようにチャネルを構成します。
* クライアントを作成し、チャネルを使用して呼び出しを行います。

`GrpcWebHandler` には次の構成オプションがあります。

* **InnerHandler**:gRPC HTTP 要求を行う基になる <xref:System.Net.Http.HttpMessageHandler> (`HttpClientHandler` など)。
* **GrpcWebMode**:gRPC HTTP 要求の `Content-Type` が `application/grpc-web` または `application/grpc-web-text` であるかどうかを指定する列挙型。
    * `GrpcWebMode.GrpcWeb` は、コンテンツをエンコードせずに送信するように構成します。 既定値です。
    * `GrpcWebMode.GrpcWebText` は、コンテンツを base64 でエンコードするように構成します。 ブラウザーでのサーバー ストリーム呼び出しに必要です。
* **HttpVersion**:基になる gRPC HTTP 要求で、[HttpRequestMessage](xref:System.Net.Http.HttpRequestMessage.Version) を設定するために使用される HTTP プロトコル `Version`。 gRPC-Web では特定のバージョンを必要とせず、指定しない限り、既定値はオーバーライドされません。

> [!IMPORTANT]
> 生成された gRPC クライアントには、単項メソッドを呼び出すための同期メソッドと非同期メソッドがあります。 たとえば、`SayHello` は同期であり、`SayHelloAsync` は非同期です。 Blazor WebAssembly アプリで同期メソッドを呼び出すと、アプリが応答しなくなります。 Blazor WebAssembly では、常に非同期メソッドを使用する必要があります。

### <a name="use-grpc-client-factory-with-grpc-web"></a>gRPC-Web で gRPC クライアント ファクトリを使用する

gRPC-Web と互換性のある .NET クライアントは、gRPC と [HttpClientFactory](xref:System.Net.Http.IHttpClientFactory) の統合を使用して作成できます。

gRPC-Web とクライアント ファクトリを使用するには:

* 次のパッケージのプロジェクト ファイルにパッケージ参照を追加します。
  * [Grpc.Net.Client.Web](https://www.nuget.org/packages/Grpc.Net.Client.Web)
  * [Grpc.Net.ClientFactory](https://www.nuget.org/packages/Grpc.Net.ClientFactory)
* 汎用の `AddGrpcClient` 拡張メソッドを使用して、依存関係の注入 (DI) に gRPC クライアントを登録します。 Blazor WebAssembly アプリでは、サービスは `Program.cs` で DI に登録されます。
* <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler%2A> 拡張メソッドを使用して `GrpcWebHandler` を構成します。

```csharp
builder.Services
    .AddGrpcClient<Greet.GreeterClient>((services, options) =>
    {
        options.Address = new Uri("https://localhost:5001");
    })
    .ConfigurePrimaryHttpMessageHandler(
        () => new GrpcWebHandler(GrpcWebMode.GrpcWebText, new HttpClientHandler()));
```

詳細については、「<xref:grpc/clientfactory>」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* [Web クライアント用 gRPC の GitHub プロジェクト](https://github.com/grpc/grpc-web)
* <xref:security/cors>
* <xref:grpc/httpapi>
