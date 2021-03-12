---
title: ASP.NET Core のための gRPC での認証と承認
author: jamesnk
description: ASP.NET Core のために gRPC で認証と承認を使用する方法について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 05/26/2020
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
uid: grpc/authn-and-authz
ms.openlocfilehash: 5120964459a81ef2d668877bb08ecde512c2389d
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587997"
---
# <a name="authentication-and-authorization-in-grpc-for-aspnet-core"></a>ASP.NET Core のための gRPC での認証と承認

作成者: [James Newton-King](https://twitter.com/jamesnk)

[サンプル コードを表示またはダウンロードする](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/grpc/authn-and-authz/sample/) ([ダウンロード方法](xref:index#how-to-download-a-sample))

## <a name="authenticate-users-calling-a-grpc-service"></a>gRPC サービスを呼び出しているユーザーを認証する

gRPC を [ASP.NET Core 認証](xref:security/authentication/identity)と共に使用して、ユーザーを各呼び出しに関連付けることができます。

以下に、gRPC と ASP.NET Core 認証を使用する `Startup.Configure` の例を示します。

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseRouting();
    
    app.UseAuthentication();
    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapGrpcService<GreeterService>();
    });
}
```

> [!NOTE]
> ASP.NET Core 認証ミドルウェアを登録する順序が重要です。 必ず `UseRouting` の後に `UseAuthentication` と `UseAuthorization` を呼び出し、その後に `UseEndpoints` を呼び出します。

呼び出し時にアプリによって使用される認証メカニズムは、構成する必要があります。 認証の構成は、`Startup.ConfigureServices` に追加され、アプリで使用される認証メカニズムによって異なるものとなります。 ASP.NET Core アプリをセキュリティで保護する方法の例については、[認証サンプル](xref:security/authentication/samples)に関するページを参照してください。

認証が設定されると、`ServerCallContext` を介して gRPC サービス メソッドでユーザーにアクセスできます。

```csharp
public override Task<BuyTicketsResponse> BuyTickets(
    BuyTicketsRequest request, ServerCallContext context)
{
    var user = context.GetHttpContext().User;

    // ... access data from ClaimsPrincipal ...
}

```

### <a name="bearer-token-authentication"></a>ベアラー トークン認証

クライアントは認証のためのアクセス トークンを提供できます。 サーバーはトークンを検証し、それを使用してユーザーを識別します。

サーバーでは、[JWT ベアラー ミドルウェア](/dotnet/api/microsoft.extensions.dependencyinjection.jwtbearerextensions.addjwtbearer)を使用してベアラー トークン認証が構成されます。

.NET gRPC クライアントでは、`Metadata` コレクションを使用すれば、呼び出しでトークンを送信することができます。 `Metadata` コレクション内のエントリは、gRPC 呼び出しで、HTTP ヘッダーとして送信されます。

```csharp
public bool DoAuthenticatedCall(
    Ticketer.TicketerClient client, string token)
{
    var headers = new Metadata();
    headers.Add("Authorization", $"Bearer {token}");

    var request = new BuyTicketsRequest { Count = 1 };
    var response = await client.BuyTicketsAsync(request, headers);

    return response.Success;
}
```

チャネルで `ChannelCredentials` を構成することは、gRPC 呼び出しを使用してトークンをサービスに送信するもう 1 つの方法です。 `ChannelCredentials` には `CallCredentials` を含めることができます。これにより、`Metadata` を自動的に設定することができます。

gRPC 呼び出しが行われるたびに `CallCredentials` が実行されるので、コードを複数の場所に記述してユーザー自身でトークンを渡す必要がなくなります。 `CallCredentials` はチャネルが TLS でセキュリティ保護されている場合にのみ適用されることに注意してください。 セキュリティで保護されていない非 TLS チャネルに、`CallCredentials` は適用されません。

次の例の資格情報では、すべての gRPC 呼び出しでトークンを送信するようにチャネルを構成しています。

```csharp
private static GrpcChannel CreateAuthenticatedChannel(string address)
{
    var credentials = CallCredentials.FromInterceptor((context, metadata) =>
    {
        if (!string.IsNullOrEmpty(_token))
        {
            metadata.Add("Authorization", $"Bearer {_token}");
        }
        return Task.CompletedTask;
    });

    // SslCredentials is used here because this channel is using TLS.
    // CallCredentials can't be used with ChannelCredentials.Insecure on non-TLS channels.
    var channel = GrpcChannel.ForAddress(address, new GrpcChannelOptions
    {
        Credentials = ChannelCredentials.Create(new SslCredentials(), credentials)
    });
    return channel;
}
```

### <a name="client-certificate-authentication"></a>クライアント証明書認証

別の方法として、クライアントから認証のためにクライアント証明書を提供することもできます。 [証明書の認証](https://tools.ietf.org/html/rfc5246#section-7.4.4)は、ASP.NET Core に到達するずっと前に TLS レベルで実行されます。 要求が ASP.NET Core に到達すると、[クライアント証明書の認証パッケージ](xref:security/authentication/certauth)によって、証明書を `ClaimsPrincipal` に解決できるようになります。

> [!NOTE]
> クライアント証明書を受け入れるようにサーバーを構成します。 Kestrel、IIS、および Azure でのクライアント証明書の受け入れについては、「<xref:security/authentication/certauth#configure-your-server-to-require-certificates>」を参照してください。

.NET gRPC クライアントでは、クライアント証明書は `HttpClientHandler` に追加されます。これがその後、gRPC クライアントの作成に使用されます。

```csharp
public Ticketer.TicketerClient CreateClientWithCert(
    string baseAddress,
    X509Certificate2 certificate)
{
    // Add client cert to the handler
    var handler = new HttpClientHandler();
    handler.ClientCertificates.Add(certificate);

    // Create the gRPC channel
    var channel = GrpcChannel.ForAddress(baseAddress, new GrpcChannelOptions
    {
        HttpHandler = handler
    });

    return new Ticketer.TicketerClient(channel);
}
```

### <a name="other-authentication-mechanisms"></a>その他の認証メカニズム

ASP.NET Core でサポートされている多くの認証メカニズムを、gRPC で使用できます。

* Azure Active Directory
* クライアント証明書
* Identityサーバー
* JWT トークン
* OAuth 2.0
* OpenID Connect
* WS-Federation

サーバーで認証を構成することの詳細については、[ASP.NET Core 認証](xref:security/authentication/identity)に関するページを参照してください。

認証を使用するための gRPC クライアントの構成は、使おうとしている認証メカニズムによって異なります。 前述したベアラー トークンとクライアント証明書の例では、gRPC 呼び出しで認証メタデータを送信するように gRPC クライアントを構成できるいくつかの方法を示しています。

* 厳密に型指定された gRPC クライアントは内部的に `HttpClient` を使用します。 認証は、[HttpClientHandler](/dotnet/api/system.net.http.httpclienthandler) で構成することも、カスタム [HttpMessageHandler](/dotnet/api/system.net.http.httpmessagehandler) インスタンスを `HttpClient` に追加して構成することもできます。
* それぞれの gRPC 呼び出しに、省略可能な `CallOptions` 引数があります。 オプションのヘッダー コレクションを使用して、カスタム ヘッダーを送信できます。

> [!NOTE]
> gRPC で Windows 認証 (NTLM/Kerberos/ネゴシエート) を使用することはできません。 gRPC には HTTP/2 が必要で、HTTP/2 では Windows 認証がサポートされていません。

## <a name="authorize-users-to-access-services-and-service-methods"></a>サービスやサービス メソッドにアクセスするユーザーを承認する

既定では、認証されていないユーザーが、サービスのすべてのメソッドを呼び出すことができます。 認証を要求するには、サービスに [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) 属性を適用します。

```csharp
[Authorize]
public class TicketerService : Ticketer.TicketerBase
{
}
```

コンストラクターの引数と `[Authorize]` 属性のプロパティを使用して、特定の[承認ポリシー](xref:security/authorization/policies)に一致するユーザーのみにアクセスを制限できます。 たとえば、`MyAuthorizationPolicy` というカスタム承認ポリシーがある場合は、次のコードを使用して、そのポリシーに一致するユーザーだけがサービスにアクセスできるようにします。

```csharp
[Authorize("MyAuthorizationPolicy")]
public class TicketerService : Ticketer.TicketerBase
{
}
```

個々のサービス メソッドに `[Authorize]` 属性を適用することもできます。 現在のユーザーが、メソッドとクラスの **両方** に適用されているポリシーと一致しない場合は、呼び出し元にエラーが返されます。

```csharp
[Authorize]
public class TicketerService : Ticketer.TicketerBase
{
    public override Task<AvailableTicketsResponse> GetAvailableTickets(
        Empty request, ServerCallContext context)
    {
        // ... buy tickets for the current user ...
    }

    [Authorize("Administrators")]
    public override Task<BuyTicketsResponse> RefundTickets(
        BuyTicketsRequest request, ServerCallContext context)
    {
        // ... refund tickets (something only Administrators can do) ..
    }
}
```

## <a name="additional-resources"></a>その他の技術情報

* [ASP.NET Core でのベアラー トークン認証](https://blogs.msdn.microsoft.com/webdev/2016/10/27/bearer-token-authentication-in-asp-net-core/)
* [ASP.NET Core でクライアント証明書の認証を構成する](xref:security/authentication/certauth)
