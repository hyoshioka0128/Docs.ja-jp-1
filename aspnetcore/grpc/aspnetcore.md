---
title: ASP.NET Core を使用した gRPC サービス
author: juntaoluo
description: ASP.NET Core で gRPC サービスを作成するときの基本的な概念について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: johluo
ms.date: 01/14/2021
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
uid: grpc/aspnetcore
ms.openlocfilehash: 44a6f1d2a25314460fa4bce469f697a2fa4c0825
ms.sourcegitcommit: 063a06b644d3ade3c15ce00e72a758ec1187dd06
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98252852"
---
# <a name="grpc-services-with-aspnet-core"></a>ASP.NET Core を使用した gRPC サービス

このドキュメントでは、ASP.NET Core を使用して gRPC サービスを開始する方法について説明します。

[!INCLUDE[](~/includes/gRPCazure.md)]

## <a name="prerequisites"></a>必須コンポーネント

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.0.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.0.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.0.md)]

---

## <a name="get-started-with-grpc-service-in-aspnet-core"></a>ASP.NET Core の gRPC サービスの概要

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/grpc/grpc-start/sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

gRPC プロジェクトを作成する詳細な手順については、[gRPC サービスの概要](xref:tutorials/grpc/grpc-start)に関するページをご覧ください。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

コマンド ラインから `dotnet new grpc -o GrpcGreeter` を実行します。

---

## <a name="add-grpc-services-to-an-aspnet-core-app"></a>ASP.NET Core アプリに gRPC サービスを追加する

gRPC には [Grpc. AspNetCore](https://www.nuget.org/packages/Grpc.AspNetCore) パッケージが必要です。

### <a name="configure-grpc"></a>gRPC を構成する

*Startup.cs* の場合:

* gRPC を有効にするには、`AddGrpc` メソッドを使用します。
* 各 gRPC サービスをルーティング パイプラインに追加するには、`MapGrpcService` メソッドを使用します。

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeter/Startup.cs?name=snippet&highlight=7,24)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

ルーティング パイプラインは ASP.NET Core のミドルウェアと機能で共有されるため、追加の要求ハンドラーを提供するようにアプリを構成することができます。 MVC コントローラーなどの追加の要求ハンドラーは、構成されている gRPC サービスと並行して動作します。

::: moniker range=">= aspnetcore-5.0"

### <a name="configure-kestrel"></a>Kestrel を構成する

Kestrel gRPC エンドポイントは次のようになります。

* HTTP/2 が必要です。
* [トランスポート層セキュリティ (TLS)](https://tools.ietf.org/html/rfc5246) で保護されている必要があります。

#### <a name="http2"></a>HTTP/2

gRPC には HTTP/2 が必要です。 ASP.NET Core 用 gRPC は、[HttpRequest](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol%2A) が `HTTP/2` であるかを検証します。

Kestrel は、最新のオペレーティング システムで [HTTP/2 をサポート](xref:fundamentals/servers/kestrel/http2)しています。 Kestrel エンドポイントは、既定で HTTP/1.1 接続と HTTP/2 接続をサポートするように構成されています。

#### <a name="tls"></a>TLS

gRPC に使用される Kestrel エンドポイントは、TLS で保護されている必要があります。 開発環境では、ASP.NET Core 開発証明書が存在する場合に、TLS で保護されたエンドポイントが `https://localhost:5001` に自動的に作成されます。 構成は必要ありません。 `https` プレフィックスによって、Kestrel エンドポイントが TLS を使用していることが確認されます。

運用環境では、TLS を明示的に構成する必要があります。 次の *appsettings.json* の例では、TLS で保護された HTTP/2 エンドポイントが指定されています。

[!code-json[](~/grpc/aspnetcore/sample/appsettings.json?highlight=4)]

または、*Program.cs* 内に Kestrel エンドポイントを構成することもできます。

[!code-csharp[](~/grpc/aspnetcore/sample/Program.cs?highlight=7&name=snippet)]

#### <a name="protocol-negotiation"></a>プロトコル ネゴシエーション

TLS は、通信のセキュリティ保護以外にも使用されます。 エンドポイントで複数のプロトコルがサポートされている場合、クライアントとサーバー間の接続プロトコルをネゴシエートするために TLS [ALPN (Application-Layer Protocol Negotiation)](https://tools.ietf.org/html/rfc7301#section-3) ハンドシェイクが使用されます。 このネゴシエーションでは、HTTP/1.1 または HTTP/2 のどちらが接続に使用されるかを判断します。

HTTP/2 エンドポイントが TLS を使用せずに構成されている場合は、エンドポイントの [ListenOptions.Protocols](xref:fundamentals/servers/kestrel/endpoints#listenoptionsprotocols) を `HttpProtocols.Http2` に設定する必要があります。 複数のプロトコル (`HttpProtocols.Http1AndHttp2` など) を持つエンドポイントは、ネゴシエーションがないため、TLS なしで使用することはできません。 セキュリティで保護されていないエンドポイントへの接続はすべて、既定で HTTP/1.1 に設定されるため、gRPC の呼び出しは失敗します。

Kestrel で HTTP/2 および TLS を有効にする方法について詳しくは、[Kestrel のエンドポイント構成](xref:fundamentals/servers/kestrel/endpoints)に関するページをご覧ください。

> [!NOTE]
> macOS の場合、ASP.NET Core gRPC と TLS の組み合わせに対応していません。 macOS で gRPC サービスを正常に実行するには、追加の構成が必要です。 詳細については、[macOS で ASP.NET Core gRPC アプリを起動できない](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos)場合に関するページを参照してください。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

### <a name="configure-kestrel"></a>Kestrel を構成する

Kestrel gRPC エンドポイントは次のようになります。

* HTTP/2 が必要です。
* [トランスポート層セキュリティ (TLS)](https://tools.ietf.org/html/rfc5246) で保護されている必要があります。

#### <a name="http2"></a>HTTP/2

gRPC には HTTP/2 が必要です。 ASP.NET Core 用 gRPC は、[HttpRequest](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol%2A) が `HTTP/2` であるかを検証します。

Kestrel は、最新のオペレーティング システムで [HTTP/2 をサポート](xref:fundamentals/servers/kestrel#http2-support)しています。 Kestrel エンドポイントは、既定で HTTP/1.1 接続と HTTP/2 接続をサポートするように構成されています。

#### <a name="tls"></a>TLS

gRPC に使用される Kestrel エンドポイントは、TLS で保護されている必要があります。 開発環境では、ASP.NET Core 開発証明書が存在する場合に、TLS で保護されたエンドポイントが `https://localhost:5001` に自動的に作成されます。 構成は必要ありません。 `https` プレフィックスによって、Kestrel エンドポイントが TLS を使用していることが確認されます。

運用環境では、TLS を明示的に構成する必要があります。 次の *appsettings.json* の例では、TLS で保護された HTTP/2 エンドポイントが指定されています。

[!code-json[](~/grpc/aspnetcore/sample/appsettings.json?highlight=4)]

または、*Program.cs* 内に Kestrel エンドポイントを構成することもできます。

[!code-csharp[](~/grpc/aspnetcore/sample/Program.cs?highlight=7&name=snippet)]

#### <a name="protocol-negotiation"></a>プロトコル ネゴシエーション

TLS は、通信のセキュリティ保護以外にも使用されます。 エンドポイントで複数のプロトコルがサポートされている場合、クライアントとサーバー間の接続プロトコルをネゴシエートするために TLS [ALPN (Application-Layer Protocol Negotiation)](https://tools.ietf.org/html/rfc7301#section-3) ハンドシェイクが使用されます。 このネゴシエーションでは、HTTP/1.1 または HTTP/2 のどちらが接続に使用されるかを判断します。

HTTP/2 エンドポイントが TLS を使用せずに構成されている場合は、エンドポイントの [ListenOptions.Protocols](xref:fundamentals/servers/kestrel#listenoptionsprotocols) を `HttpProtocols.Http2` に設定する必要があります。 複数のプロトコル (`HttpProtocols.Http1AndHttp2` など) を持つエンドポイントは、ネゴシエーションがないため、TLS なしで使用することはできません。 セキュリティで保護されていないエンドポイントへの接続はすべて、既定で HTTP/1.1 に設定されるため、gRPC の呼び出しは失敗します。

Kestrel で HTTP/2 および TLS を有効にする方法について詳しくは、[Kestrel のエンドポイント構成](xref:fundamentals/servers/kestrel#endpoint-configuration)に関するページをご覧ください。

> [!NOTE]
> macOS の場合、ASP.NET Core gRPC と TLS の組み合わせに対応していません。 macOS で gRPC サービスを正常に実行するには、追加の構成が必要です。 詳細については、[macOS で ASP.NET Core gRPC アプリを起動できない](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos)場合に関するページを参照してください。

::: moniker-end

## <a name="integration-with-aspnet-core-apis"></a>ASP.NET Core API との統合

gRPC サービスには、[依存関係の挿入](xref:fundamentals/dependency-injection) (DI) や[ログ記録](xref:fundamentals/logging/index)などの ASP.NET Core 機能へのフル アクセスがあります。 たとえば、サービスの実装では、コンストラクターを使用して DI コンテナーから logger サービスを解決できます。

```csharp
public class GreeterService : Greeter.GreeterBase
{
    public GreeterService(ILogger<GreeterService> logger)
    {
    }
}
```

既定では、gRPC サービスの実装では、任意の有効期間 (シングルトン、スコープ、または一時的) を持つ他の DI サービスを解決できます。

### <a name="resolve-httpcontext-in-grpc-methods"></a>gRPC メソッドで HttpContext を解決する

gRPC API は、メソッド、ホスト、ヘッダー、トレーラーなど、一部の HTTP/2 メッセージ データへのアクセスを提供します。 アクセスには、各 gRPC メソッドに渡される `ServerCallContext` 引数が使用されます。

[!code-csharp[](~/grpc/aspnetcore/sample/GrcpService/GreeterService.cs?highlight=3-4&name=snippet)]

`ServerCallContext` を使用しても、すべての ASP.NET API で `HttpContext` へのフル アクセスが提供されるわけではありません。 `GetHttpContext` 拡張メソッドを使用すると、ASP.NET API で基となる HTTP/2 メッセージを表す `HttpContext` にフル アクセスできます。

[!code-csharp[](~/grpc/aspnetcore/sample/GrcpService/GreeterService2.cs?highlight=6-7&name=snippet)]


## <a name="additional-resources"></a>その他の技術情報

* <xref:tutorials/grpc/grpc-start>
* <xref:grpc/index>
* <xref:grpc/basics>
* <xref:fundamentals/servers/kestrel>
