---
title: .NET Core での gRPC のトラブルシューティング
author: jamesnk
description: .NET Core で gRPC を使用している場合のエラーのトラブルシューティングを行います。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.custom: mvc
ms.date: 07/09/2020
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
uid: grpc/troubleshoot
ms.openlocfilehash: 1fd89059183300993c7fa78aa8dab1bda247a530
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586983"
---
# <a name="troubleshoot-grpc-on-net-core"></a>.NET Core での gRPC のトラブルシューティング

作成者: [James Newton-King](https://twitter.com/jamesnk)

このドキュメントでは、.NET で gRPC アプリを開発する際によく発生する問題について説明します。

## <a name="mismatch-between-client-and-service-ssltls-configuration"></a>クライアントとサービスの SSL/TLS 構成が一致しない

GRPC テンプレートとサンプルでは、[トランスポート層セキュリティ (TLS)](https://tools.ietf.org/html/rfc5246) を使用して、既定で gRPC サービスをセキュリティ保護しています。 gRPC クライアントは、セキュリティ保護された gRPC サービスを正常に呼び出すために、セキュリティ保護された接続を使用する必要があります。

アプリの起動時に書き込まれたログで、ASP.NET Core gRPC サービスが TLS を使用していることを確認できます。 サービスは、HTTPS エンドポイントでリッスンします。

```
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: https://localhost:5001
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
```

.NET Core クライアントは、サーバー アドレスで `https` を使用して、セキュリティ保護された接続で呼び出しを行う必要があります。

```csharp
static async Task Main(string[] args)
{
    // The port number(5001) must match the port of the gRPC server.
    var channel = GrpcChannel.ForAddress("https://localhost:5001");
    var client = new Greet.GreeterClient(channel);
}
```

すべての gRPC クライアント実装で TLS をサポートしています。 他の言語の gRPC クライアントでは、通常、`SslCredentials` によって構成されたチャネルが必要です。 `SslCredentials` は、クライアントで使用する証明書を指定し、安全でない資格情報の代わりにそれが使用される必要があります。 さまざまな gRPC クライアント実装で TLS を使用するように構成する例については、[gRPC 認証に関するページ](https://www.grpc.io/docs/guides/auth/)を参照してください。

## <a name="call-a-grpc-service-with-an-untrustedinvalid-certificate"></a>信頼されていないか無効な証明書で gRPC サービスを呼び出す

.NET gRPC クライアントでは、サービスに信頼された証明書が必要です。 信頼された証明書を使用せずに gRPC サービスを呼び出すと、次のエラーメッセージが返されます。

> ハンドルされていない例外です。 System.Net.Http.HttpRequestException:SSL 接続を確立できませんでした。内部例外を参照してください。
> ---> System.Security.Authentication.AuthenticationException:検証プロシージャによると、リモート証明書は無効です。

このエラーは、アプリをローカルでテストしていて、ASP.NET Core HTTPS 開発証明書が信頼されていない場合に表示されることがあります。 この問題を解決する手順については、「[Windows と macOS で ASP.NET Core HTTPS 開発証明書を信頼します](xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos)」を参照してください。

別のコンピューターで gRPC サービスを呼び出しており、その証明書を信頼できない場合、gRPC クライアントが無効な証明書を無視するように構成できます。 次のコードでは [HttpClientHandler.ServerCertificateCustomValidationCallback](/dotnet/api/system.net.http.httpclienthandler.servercertificatecustomvalidationcallback) を使用して、信頼された証明書を使用しない呼び出しを許可しています。

```csharp
var httpHandler = new HttpClientHandler();
// Return `true` to allow certificates that are untrusted/invalid
httpHandler.ServerCertificateCustomValidationCallback = 
    HttpClientHandler.DangerousAcceptAnyServerCertificateValidator;

var channel = GrpcChannel.ForAddress("https://localhost:5001",
    new GrpcChannelOptions { HttpHandler = httpHandler });
var client = new Greet.GreeterClient(channel);
```

> [!WARNING]
> 信頼されていない証明書は、アプリの開発時にのみ使用してください。 実稼働アプリでは、常に有効な証明書を使用する必要があります。

## <a name="call-insecure-grpc-services-with-net-core-client"></a>.NET Core クライアントで安全でない gRPC サービスを呼び出す

.NET gRPC クライアントを使用する場合、サーバー アドレスで `http` を指定すると、セキュリティで保護されていない gRPC サービスを呼び出すことができます。 たとえば、「 `GrpcChannel.ForAddress("http://localhost:5000")` 」のように入力します。

アプリで使用されている .NET バージョンによっては、セキュリティで保護されていない gRPC サービスの呼び出しには、さらに他の要件もいくつかあります。

* .NET 5 以降では、[Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client) バージョン 2.32.0 以降が必要です。
* .NET Core 3.x の場合は、追加の構成が必要です。 アプリでは、`System.Net.Http.SocketsHttpHandler.Http2UnencryptedSupport` スイッチを `true` に設定する必要があります。
    
    ```csharp
    // This switch must be set before creating the GrpcChannel/HttpClient.
    AppContext.SetSwitch(
        "System.Net.Http.SocketsHttpHandler.Http2UnencryptedSupport", true);
    
    // The port number(5000) must match the port of the gRPC server.
    var channel = GrpcChannel.ForAddress("http://localhost:5000");
    var client = new Greet.GreeterClient(channel);
    ```

`System.Net.Http.SocketsHttpHandler.Http2UnencryptedSupport` スイッチは、.NET Core 3.x. でのみ必要です。 .NET 5 では、何も実行されないため、必要ではありません。

## <a name="unable-to-start-aspnet-core-grpc-app-on-macos"></a>MacOS で ASP.NET Core gRPC アプリを起動できない

Kestrel では、macOS や Windows 7 などの古い Windows バージョンでの TLS による HTTP/2 がサポートされていません。 ASP.NET Core gRPC テンプレートとサンプルでは、既定で TLS を使用しています。 gRPC サーバーを起動しようとすると、次のエラー メッセージが表示されます。

> Unable to bind to https://localhost:5001 on the IPv4 loopback interface:'HTTP/2 over TLS is not supported on macOS due to missing ALPN support.'. (IPv4 ループバック インターフェイスで https://localhost:5001 にバインドできません。'HTTP/2 over TLS は ALPN サポートがないため、macOS でサポートされていません。')

この問題を回避するには、TLS を *使用せずに* HTTP/2 を使用するように Kestrel と gRPC クライアントを構成します。 これは開発時にのみ実行してください。 TLS を使用しないと、gRPC メッセージが暗号化されずに送信されます。

Kestrel では、*Program.cs* で TLS を使用せずに HTTP/2 エンドポイントを構成する必要があります。

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel(options =>
            {
                // Setup a HTTP/2 endpoint without TLS.
                options.ListenLocalhost(5000, o => o.Protocols = 
                    HttpProtocols.Http2);
            });
            webBuilder.UseStartup<Startup>();
        });
```

::: moniker range=">= aspnetcore-5.0"
HTTP/2 エンドポイントが TLS を使用せずに構成されている場合、エンドポイントの [ListenOptions.Protocol](xref:fundamentals/servers/kestrel/endpoints#listenoptionsprotocols) を `HttpProtocols.Http2` に設定する必要があります。 HTTP/2 のネゴシエートに TLS が必要であるため、`HttpProtocols.Http1AndHttp2` は使用できません。 TLS を使用しない場合、エンドポイントへのすべての接続が既定で HTTP/1.1 に設定され、gRPC の呼び出しが失敗します。
::: moniker-end

::: moniker range="< aspnetcore-5.0"
HTTP/2 エンドポイントが TLS を使用せずに構成されている場合、エンドポイントの [ListenOptions.Protocol](xref:fundamentals/servers/kestrel#listenoptionsprotocols) を `HttpProtocols.Http2` に設定する必要があります。 HTTP/2 のネゴシエートに TLS が必要であるため、`HttpProtocols.Http1AndHttp2` は使用できません。 TLS を使用しない場合、エンドポイントへのすべての接続が既定で HTTP/1.1 に設定され、gRPC の呼び出しが失敗します。
::: moniker-end

GRPC クライアントでも、TLS を使用しないように構成する必要があります。 詳細については、「[.NET Core クライアントで安全でない gRPC サービスを呼び出す](#call-insecure-grpc-services-with-net-core-client)」を参照してください。

> [!WARNING]
> TLS を使用しない HTTP/2 は、アプリの開発時にのみ使用してください。 実稼働アプリでは、常にトランスポート セキュリティを使用する必要があります。 詳細については、[ gRPC for ASP.NET Core のセキュリティの考慮事項 ](xref:grpc/security#transport-security)に関するページを参照してください。

## <a name="grpc-c-assets-are-not-code-generated-from-proto-files"></a>gRPC C# アセットが .proto ファイルから生成されたコードでない

具象クライアントとサービス基本クラスの gRPC コード生成には、protobuf ファイルとツールをプロジェクトから参照する必要があります。 次のものを含める必要があります。

* `<Protobuf>` 項目グループで使用する *.proto* ファイル。 [インポートされた *.proto* ファイル](https://developers.google.com/protocol-buffers/docs/proto3#importing-definitions) をプロジェクトで参照する必要があります。
* gRPC ツール パッケージ [Grpc.Tools](https://www.nuget.org/packages/Grpc.Tools/) へのパッケージ参照。

gRPC C# アセットの生成の詳細については、「<xref:grpc/basics>」を参照してください。

gRPC サービスをホストしている ASP.NET Core Web アプリには、生成されたサービス基本クラスのみが必要です。

```xml
<ItemGroup>
  <Protobuf Include="Protos\greet.proto" GrpcServices="Server" />
</ItemGroup>
```

gRPC 呼び出しを行う gRPC クライアント アプリでは、生成された具象クライアントのみが必要です。

```xml
<ItemGroup>
  <Protobuf Include="Protos\greet.proto" GrpcServices="Client" />
</ItemGroup>
```

## <a name="wpf-projects-unable-to-generate-grpc-c-assets-from-proto-files"></a>WPF プロジェクトで .proto ファイルから gRPC C# アセットを生成できない

WPF プロジェクトには、gRPC コードの生成が正常に機能しなくなる[既知の問題](https://github.com/dotnet/wpf/issues/810)があります。 WPF プロジェクトで `Grpc.Tools` と *.proto* ファイルを参照して生成された gRPC 型は、使用すると、コンパイル エラーが発生します。

> エラー CS0246:型または名前空間の名前 'tMyGrpcServices' が見つかりませんでした (using ディレクティブまたはアセンブリ参照が指定されていることを確認してください)

この問題は次の方法で回避できます。

1. 新しい .NET Core クラス ライブラリ プロジェクトを作成します。
2. 新しいプロジェクトで、参照を追加して、[ *\*.proto* ファイルからの C# コード生成](xref:grpc/basics#generated-c-assets)を有効にします。
    * [Grpc.Tools](https://www.nuget.org/packages/Grpc.Tools/) パッケージにパッケージ参照を追加します。
    * `<Protobuf>` 項目グループに *\*.proto* ファイルを追加します。
3. WPF アプリケーションで、新しいプロジェクトに参照を追加します。

WPF アプリケーションでは、新しいクラス ライブラリ プロジェクトから、gRPC によって生成された型を使用できます。

[!INCLUDE[](~/includes/gRPCazure.md)]
