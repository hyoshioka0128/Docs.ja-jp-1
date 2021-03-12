---
title: ASP.NET Core での認証と承認 SignalR
author: bradygaster
description: ASP.NET Core で認証と承認を使用する方法について説明 SignalR します。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 12/05/2019
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
uid: signalr/authn-and-authz
ms.openlocfilehash: 9a3102e4451bbc5cd9ff15e88bebd4e4f2c115f4
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588101"
---
# <a name="authentication-and-authorization-in-aspnet-core-signalr"></a>ASP.NET Core での認証と承認 SignalR

By [Andrew Stanton-看護師](https://twitter.com/anurse)

[サンプル コードを表示またはダウンロードする](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/authn-and-authz/sample/) ([ダウンロード方法](xref:index#how-to-download-a-sample))

## <a name="authenticate-users-connecting-to-a-signalr-hub"></a>ハブに接続しているユーザーを認証する SignalR

SignalR[ASP.NET Core 認証](xref:security/authentication/identity)と共に使用して、ユーザーを各接続に関連付けることができます。 ハブでは、 [HubConnectionContext](/dotnet/api/microsoft.aspnetcore.signalr.hubconnectioncontext.user) プロパティから認証データにアクセスできます。 認証を使用すると、ユーザーに関連付けられているすべての接続で、ハブがメソッドを呼び出すことができます。 詳細については、「 [」の SignalR 「ユーザーとグループの管理](xref:signalr/groups)」を参照してください。 複数の接続を1人のユーザーに関連付けることができます。

`Startup.Configure` SignalR 認証を使用して ASP.NET Core 認証を使用する例を次に示します。

::: moniker range=">= aspnetcore-3.0"

```csharp
public void Configure(IApplicationBuilder app)
{
    ...

    app.UseStaticFiles();

    app.UseRouting();

    app.UseAuthentication();
    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHub<ChatHub>("/chat");
        endpoints.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");
    });
}
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

```csharp
public void Configure(IApplicationBuilder app)
{
    ...

    app.UseStaticFiles();

    app.UseAuthentication();

    app.UseSignalR(hubs =>
    {
        hubs.MapHub<ChatHub>("/chat");
    });

    app.UseMvc(routes =>
    {
        routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
    });
}
```

> [!NOTE]
> および ASP.NET Core 認証ミドルウェアを登録する順序は SignalR 重要です。 が `UseAuthentication` `UseSignalR` SignalR のユーザーを持つようにするには、常にを呼び出し `HttpContext` ます。

::: moniker-end

### <a name="cookie-authentication"></a>Cookie 認証

ブラウザーベースのアプリでは、 cookie 認証によって既存のユーザーの資格情報を自動的に接続に渡すことができ SignalR ます。 ブラウザークライアントを使用する場合、追加の構成は必要ありません。 ユーザーがアプリにログインしている場合、 SignalR 接続は自動的にこの認証を継承します。

Cookieは、アクセストークンを送信するためのブラウザー固有の方法ですが、ブラウザー以外のクライアントはそれらを送信できます。 [.Net クライアント](xref:signalr/dotnet-client)を使用する場合は、 `Cookies` `.WithUrl` を指定するための呼び出しでプロパティを構成でき cookie ます。 ただし、 cookie .net クライアントから認証を使用するには、アプリケーションがの認証データを交換するための API を提供する必要があり cookie ます。

### <a name="bearer-token-authentication"></a>ベアラー トークン認証

クライアントは、を使用する代わりに、アクセストークンを提供でき cookie ます。 サーバーはトークンを検証し、それを使用してユーザーを識別します。 この検証は、接続が確立された場合にのみ実行されます。 接続の有効期間中は、トークンの失効を確認するためにサーバーが自動的に再検証されることはありません。

JavaScript クライアントでは、 [accessTokenFactory](xref:signalr/configuration#configure-bearer-authentication) オプションを使用してトークンを指定できます。

[!code-typescript[Configure Access Token](authn-and-authz/sample/wwwroot/js/chat.ts?range=52-55)]

.NET クライアントには、トークンの構成に使用できる同様の [れる場合](xref:signalr/configuration#configure-bearer-authentication) プロパティがあります。

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options =>
    { 
        options.AccessTokenProvider = () => Task.FromResult(_myAccessToken);
    })
    .Build();
```

> [!NOTE]
> 提供するアクセストークン関数は、によって行われる **すべて** の HTTP 要求の前に呼び出され SignalR ます。 接続をアクティブな状態に保つために、トークンを更新する必要がある場合 (接続中に期限切れになる可能性があるため)、この関数内からトークンを取得し、更新されたトークンを返します。

標準の web Api では、ベアラートークンは HTTP ヘッダーで送信されます。 ただし、で SignalR は、一部のトランスポートを使用するときに、これらのヘッダーをブラウザーで設定することはできません。 Websocket と Server-Sent イベントを使用する場合、トークンはクエリ文字列パラメーターとして送信されます。 

#### <a name="built-in-jwt-authentication"></a>組み込み JWT 認証

サーバーでは、ベアラートークン認証は [JWT ベアラーミドルウェア](xref:Microsoft.Extensions.DependencyInjection.JwtBearerExtensions.AddJwtBearer%2A)を使用して構成されます。

[!code-csharp[Configure Server to accept access token from Query String](authn-and-authz/sample/Startup.cs?name=snippet)]

[!INCLUDE[request localized comments](~/includes/code-comments-loc.md)]

> [!NOTE]
> クエリ文字列は、ブラウザー API の制限により、Websocket と Server-Sent イベントに接続するときにブラウザーで使用されます。 HTTPS を使用する場合、クエリ文字列の値は TLS 接続によって保護されます。 ただし、多くのサーバーはクエリ文字列の値をログに記録します。 詳細については、「 [ASP.NET Core SignalR のセキュリティに関する考慮事項](xref:signalr/security)」を参照してください。 SignalR はヘッダーを使用して、トークンをサポートする環境 (.NET や Java クライアントなど) でトークンを送信します。

#### <a name="identity-server-jwt-authentication"></a>Identity サーバー JWT 認証

サーバーを使用する場合は Identity 、 <xref:Microsoft.Extensions.Options.PostConfigureOptions%601> 次のようにサービスをプロジェクトに追加します。

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.Extensions.Options;
public class ConfigureJwtBearerOptions : IPostConfigureOptions<JwtBearerOptions>
{
    public void PostConfigure(string name, JwtBearerOptions options)
    {
        var originalOnMessageReceived = options.Events.OnMessageReceived;
        options.Events.OnMessageReceived = async context =>
        {
            await originalOnMessageReceived(context);
                
            if (string.IsNullOrEmpty(context.Token))
            {
                var accessToken = context.Request.Query["access_token"];
                var path = context.HttpContext.Request.Path;
                
                if (!string.IsNullOrEmpty(accessToken) && 
                    path.StartsWithSegments("/hubs"))
                {
                    context.Token = accessToken;
                }
            }
        };
    }
}
```

`Startup.ConfigureServices`認証用のサービス ( <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication%2A> ) とサーバーの認証ハンドラー () を追加した後で、にサービスを登録し Identity <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> ます。

```csharp
services.AddAuthentication()
    .AddIdentityServerJwt();
services.TryAddEnumerable(
    ServiceDescriptor.Singleton<IPostConfigureOptions<JwtBearerOptions>, 
        ConfigureJwtBearerOptions>());
```

### <a name="cookies-vs-bearer-tokens"></a>Cookies およびベアラートークン 

Cookieは、ブラウザーに固有のものです。 他の種類のクライアントから送信すると、ベアラートークンの送信と比較して複雑さが増します。 その cookie ため、アプリがブラウザークライアントからのユーザーの認証のみを必要とする場合を除き、認証は推奨されません。 ベアラートークン認証は、ブラウザークライアント以外のクライアントを使用する場合に推奨される方法です。

### <a name="windows-authentication"></a>Windows 認証

アプリで [Windows 認証](xref:security/authentication/windowsauth) が構成されている場合、は SignalR その id を使用してハブをセキュリティで保護することができます。 ただし、個々のユーザーにメッセージを送信するには、カスタムユーザー ID プロバイダーを追加する必要があります。 Windows 認証システムは、"名前識別子" 要求を提供しません。 SignalR は、要求を使用してユーザー名を決定します。

を実装 `IUserIdProvider` し、識別子として使用する要求の1つをユーザーから取得する新しいクラスを追加します。 たとえば、"Name" 要求 (フォームの Windows ユーザー名) を使用するには、 `[Domain]\[Username]` 次のクラスを作成します。

[!code-csharp[Name based provider](authn-and-authz/sample/nameuseridprovider.cs?name=NameUserIdProvider)]

ではなく、 `ClaimTypes.Name` の任意の値 `User` (Windows SID 識別子など) を使用できます。

> [!NOTE]
> 選択する値は、システム内のすべてのユーザーの間で一意である必要があります。 そうしないと、1人のユーザーを対象としたメッセージが別のユーザーに送信される可能性があります。

メソッドにこのコンポーネントを登録 `Startup.ConfigureServices` します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ... other services ...

    services.AddSignalR();
    services.AddSingleton<IUserIdProvider, NameUserIdProvider>();
}
```

.NET クライアントでは、 [UseDefaultCredentials](/dotnet/api/microsoft.aspnetcore.http.connections.client.httpconnectionoptions.usedefaultcredentials) プロパティを設定することによって Windows 認証を有効にする必要があります。

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options =>
    {
        options.UseDefaultCredentials = true;
    })
    .Build();
```

Windows 認証は Internet Explorer と Microsoft Edge ではサポートされていますが、すべてのブラウザーではサポートされていません。 たとえば、Chrome および Safari で Windows 認証と Websocket を使用しようとすると失敗します。 Windows 認証が失敗した場合、クライアントは、動作する可能性がある他のトランスポートへのフォールバックを試みます。

### <a name="use-claims-to-customize-identity-handling"></a>要求を使用して id 処理をカスタマイズする

ユーザーを認証するアプリは、ユーザー要求からユーザー Id を派生させることができ SignalR ます。 がユーザー Id を作成する方法を指定するには SignalR 、実装を実装し `IUserIdProvider` て登録します。

このサンプルコードでは、要求を使用して、識別プロパティとしてユーザーの電子メールアドレスを選択する方法を示します。 

> [!NOTE]
> 選択する値は、システム内のすべてのユーザーの間で一意である必要があります。 そうしないと、1人のユーザーを対象としたメッセージが別のユーザーに送信される可能性があります。

[!code-csharp[Email provider](authn-and-authz/sample/EmailBasedUserIdProvider.cs?name=EmailBasedUserIdProvider)]

アカウント登録によって、種類がの要求が `ClaimsTypes.Email` ASP.NET identity データベースに追加されます。

[!code-csharp[Adding the email to the ASP.NET identity claims](authn-and-authz/sample/pages/account/Register.cshtml.cs?name=AddEmailClaim)]

このコンポーネントをに登録 `Startup.ConfigureServices` します。

```csharp
services.AddSingleton<IUserIdProvider, EmailBasedUserIdProvider>();
```

## <a name="authorize-users-to-access-hubs-and-hub-methods"></a>ハブおよびハブのメソッドへのアクセスをユーザーに承認する

既定では、ハブ内のすべてのメソッドを認証されていないユーザーが呼び出すことができます。 認証を要求するには、 [承認](/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute) 属性をハブに適用します。

[!code-csharp[Restrict a hub to only authorized users](authn-and-authz/sample/Hubs/ChatHub.cs?range=8-10,32)]

コンストラクターの引数と `[Authorize]` 属性のプロパティを使用して、特定の[承認ポリシー](xref:security/authorization/policies)に一致するユーザーのみにアクセスを制限できます。 たとえば、というカスタム承認ポリシーがある場合は、 `MyAuthorizationPolicy` 次のコードを使用して、そのポリシーに一致するユーザーだけがハブにアクセスできるようにすることができます。

```csharp
[Authorize("MyAuthorizationPolicy")]
public class ChatHub : Hub
{
}
```

個々のハブメソッドに `[Authorize]` も属性を適用できます。 現在のユーザーがメソッドに適用されているポリシーと一致しない場合は、呼び出し元にエラーが返されます。

```csharp
[Authorize]
public class ChatHub : Hub
{
    public async Task Send(string message)
    {
        // ... send a message to all users ...
    }

    [Authorize("Administrators")]
    public void BanUser(string userName)
    {
        // ... ban a user from the chat room (something only Administrators can do) ...
    }
}
```

::: moniker range=">= aspnetcore-3.0"

### <a name="use-authorization-handlers-to-customize-hub-method-authorization"></a>承認ハンドラーを使用してハブメソッドの承認をカスタマイズする

SignalR ハブメソッドが承認を必要とする場合に、カスタムリソースを承認ハンドラーに提供します。 リソースは `HubInvocationContext` のインスタンスです。 には、、 `HubInvocationContext` `HubCallerContext` 呼び出されるハブメソッドの名前、およびハブメソッドへの引数が含まれます。

Azure Active Directory による複数の組織でのサインインを可能にするチャットルームの例を考えてみましょう。 Microsoft アカウントを持つユーザーはだれでもチャットにサインインできますが、所有している組織のメンバーだけがユーザーの許可を禁止したり、ユーザーのチャット履歴を表示したりできるようにする必要があります。 さらに、特定のユーザーの特定の機能を制限することもできます。 ASP.NET Core 3.0 の更新された機能を使用すると、これは完全に可能です。 が `DomainRestrictedRequirement` カスタムとして機能することに注意して `IAuthorizationRequirement` ください。 `HubInvocationContext`リソースパラメーターが渡されたので、内部ロジックはハブが呼び出されているコンテキストを検査し、ユーザーが個々のハブメソッドを実行できるようにすることを決定します。

```csharp
[Authorize]
public class ChatHub : Hub
{
    public void SendMessage(string message)
    {
    }

    [Authorize("DomainRestricted")]
    public void BanUser(string username)
    {
    }

    [Authorize("DomainRestricted")]
    public void ViewUserHistory(string username)
    {
    }
}

public class DomainRestrictedRequirement : 
    AuthorizationHandler<DomainRestrictedRequirement, HubInvocationContext>, 
    IAuthorizationRequirement
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context,
        DomainRestrictedRequirement requirement, 
        HubInvocationContext resource)
    {
        if (IsUserAllowedToDoThis(resource.HubMethodName, context.User.Identity.Name) && 
            context.User.Identity.Name.EndsWith("@microsoft.com"))
        {
            context.Succeed(requirement);
        }
        return Task.CompletedTask;
    }

    private bool IsUserAllowedToDoThis(string hubMethodName,
        string currentUsername)
    {
        return !(currentUsername.Equals("asdf42@microsoft.com") && 
            hubMethodName.Equals("banUser", StringComparison.OrdinalIgnoreCase));
    }
}
```

で、 `Startup.ConfigureServices` 新しいポリシーを追加し、ポリシーを `DomainRestrictedRequirement` 作成するためのパラメーターとしてカスタム要件を指定し `DomainRestricted` ます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ... other services ...

    services
        .AddAuthorization(options =>
        {
            options.AddPolicy("DomainRestricted", policy =>
            {
                policy.Requirements.Add(new DomainRestrictedRequirement());
            });
        });
}
```

前の例では、クラスはとの両方で、その `DomainRestrictedRequirement` `IAuthorizationRequirement` `AuthorizationHandler` 要件に対応しています。 この2つのコンポーネントを別々のクラスに分割して、懸念事項を分けることができます。 この例のアプローチの利点は、 `AuthorizationHandler` 要件とハンドラーが同じであるため、起動時にを挿入する必要がないことです。

::: moniker-end

## <a name="additional-resources"></a>その他のリソース

* [ASP.NET Core でのベアラートークン認証](https://blogs.msdn.microsoft.com/webdev/2016/10/27/bearer-token-authentication-in-asp-net-core/)
* [リソースベースの承認](xref:security/authorization/resourcebased)
