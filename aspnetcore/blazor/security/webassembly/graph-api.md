---
title: ASP.NET Core Blazor WebAssembly で Graph API を使用する
author: guardrex
description: Blazor WebAssemlby アプリで Graph API を使用する方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/27/2020
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
uid: blazor/security/webassembly/graph-api
ms.openlocfilehash: b9e95afc2ccff95c6acd215a70c8b0f92fb1b8c0
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/06/2021
ms.locfileid: "102394734"
---
# <a name="use-graph-api-with-aspnet-core-blazor-webassembly"></a>ASP.NET Core Blazor WebAssembly で Graph API を使用する

::: moniker range=">= aspnetcore-5.0"

[Microsoft Graph API](/graph/use-the-api) は、Blazor および他の .NET Framework アプリが Microsoft Cloud サービス リソースにアクセスできるようにする RESTful Web API です。

## <a name="graph-sdk"></a>Graph SDK

[Microsoft Graph SDK](/graph/sdks/sdks-overview) は、Microsoft Graph にアクセスする高品質で効率的であり、回復性があるアプリケーションのビルドを簡素化するように設計されています。

このセクションの例では、スタンドアロンのプロジェクト ファイルまたは **`Client`** アプリのプロジェクト ファイルに次のパッケージのパッケージ参照が必要です。

* [`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http)
* [`Microsoft.Graph`](https://www.nuget.org/packages/Microsoft.Graph)

次のユーティリティ クラスおよび構成は、この記事の次の各サブセクションで使用されます。

* [Graph SDK を使用してコンポーネントから Graph API を呼び出す](#call-graph-api-from-a-component-using-the-graph-sdk)
* [Graph SDK を使用してユーザー要求をカスタマイズする](#customize-user-claims-with-the-graph-sdk)

Azure portal の AAD 領域で Microsoft Graph API スコープを追加した後:

* ホストされている Blazor ソリューションのスタンドアロン アプリまたは **`Client`** アプリに次の `GraphClientExtensions.cs` クラスを追加します。
* `AuthenticateRequestAsync` メソッド内の <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenRequestOptions> の <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenRequestOptions.Scopes> プロパティに必要なスコープを提供します。 次の例では、この記事の後述のセクションにある例と一致するように `User.Read` スコープを指定しています。

```csharp
using System;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.Authentication.WebAssembly.Msal.Models;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Graph;

internal static class GraphClientExtensions
{
    public static IServiceCollection AddGraphClient(
        this IServiceCollection services, params string[] scopes)
    {
        services.Configure<RemoteAuthenticationOptions<MsalProviderOptions>>(
            options =>
            {
                foreach (var scope in scopes)
                {
                    options.ProviderOptions.AdditionalScopesToConsent.Add(scope);
                }
            });

        services.AddScoped<IAuthenticationProvider, 
            NoOpGraphAuthenticationProvider>();
        services.AddScoped<IHttpProvider, HttpClientHttpProvider>(sp => 
            new HttpClientHttpProvider(new HttpClient()));
        services.AddScoped(sp =>
        {
            return new GraphServiceClient(
                sp.GetRequiredService<IAuthenticationProvider>(),
                sp.GetRequiredService<IHttpProvider>());
        });

        return services;
    }

    private class NoOpGraphAuthenticationProvider : IAuthenticationProvider
    {
        public NoOpGraphAuthenticationProvider(IAccessTokenProvider tokenProvider)
        {
            TokenProvider = tokenProvider;
        }

        public IAccessTokenProvider TokenProvider { get; }

        public async Task AuthenticateRequestAsync(HttpRequestMessage request)
        {
            var result = await TokenProvider.RequestAccessToken(
                new AccessTokenRequestOptions()
                {
                    Scopes = new[] { "{SCOPE 1}", "{SCOPE 2}", ... "{SCOPE X}" }
                });

            if (result.TryGetToken(out var token))
            {
                request.Headers.Authorization ??= new AuthenticationHeaderValue(
                    "Bearer", token.Value);
            }
        }
    }

    private class HttpClientHttpProvider : IHttpProvider
    {
        private readonly HttpClient http;

        public HttpClientHttpProvider(HttpClient http)
        {
            this.http = http;
        }

        public ISerializer Serializer { get; } = new Serializer();

        public TimeSpan OverallTimeout { get; set; } = TimeSpan.FromSeconds(300);

        public void Dispose()
        {
        }

        public Task<HttpResponseMessage> SendAsync(HttpRequestMessage request)
        {
            return http.SendAsync(request);
        }

        public Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, 
            HttpCompletionOption completionOption, 
            CancellationToken cancellationToken)
        {
            return http.SendAsync(request, completionOption, cancellationToken);
        }
    }
}
```

前のコードのスコープのプレースホルダー `"{SCOPE 1}", "{SCOPE 2}", ... "{SCOPE X}"` は、1 つ以上の許可されたスコープを表します。 たとえば、`Scopes` を、この記事の後のセクションにある例の `User.Read` の 1 つのスコープの文字列配列に設定します。

```csharp
Scopes = new[] { "https://graph.microsoft.com/User.Read" }
```

`Program.Main` (`Program.cs`) で、`AddGraphClient` 拡張メソッドを使用して Graph クライアント サービスと構成を追加します。

```csharp
builder.Services.AddGraphClient("{SCOPE 1}", "{SCOPE 2}", ... "{SCOPE X}");
```

前のコードのスコープのプレースホルダー `"{SCOPE 1}", "{SCOPE 2}", ... "{SCOPE X}"` は、1 つ以上の許可されたスコープを表します。 たとえば、`User.Read` を、この記事の次のセクションにある例の `AddGraphClient` スコープに渡します。

```csharp
builder.Services.AddGraphClient("https://graph.microsoft.com/User.Read");
```

### <a name="call-graph-api-from-a-component-using-the-graph-sdk"></a>Graph SDK を使用してコンポーネントから Graph API を呼び出す

このセクションでは、この記事で既に説明した [ユーティリティ クラス (`GraphClientExtensions.cs`)](#graph-sdk) を使用します。 次の `GraphExample` コンポーネントでは、挿入された `GraphServiceClient` を使用してユーザーの AAD プロファイル データを取得し、その携帯電話番号を表示します。

```razor
@page "/GraphExample"
@using Microsoft.AspNetCore.Authorization
@using Microsoft.Graph
@attribute [Authorize]
@inject GraphServiceClient GraphClient

<h3>Graph Client Example</h3>

@if (user != null)
{
    <p>Mobile Phone: @user.MobilePhone</p>
}

@code {
    private User user;

    protected async override Task OnInitializedAsync()
    {
        var request = GraphClient.Me.Request();
        user = await request.GetAsync();
    }
}
```

### <a name="customize-user-claims-with-the-graph-sdk"></a>Graph SDK を使用してユーザー要求をカスタマイズする

このセクションでは、この記事で既に説明した [ユーティリティ クラス (`GraphClientExtensions.cs`)](#graph-sdk) を使用します。

次の例のアプリでは、AAD ユーザー プロファイルの携帯電話番号からユーザーの携帯電話番号要求を作成します。 このアプリには、AAD で構成された `User.Read` Graph API スコープが必要です。

次のカスタム ユーザー アカウント ファクトリでは、フレームワークの <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> はユーザーのアカウントを表します。 <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> を拡張するカスタム ユーザー アカウント クラスがアプリで必要な場合は、そのカスタム ユーザー アカウント クラスを次のコード内の <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> と入れ替えます。

`CustomAccountFactory.cs`:

```csharp
using System;
using System.Net.Http;
using System.Net.Http.Json;
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication.Internal;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.Graph;

public class CustomAccountFactory
    : AccountClaimsPrincipalFactory<RemoteUserAccount>
{
    private readonly ILogger<CustomAccountFactory> logger;
    private readonly IServiceProvider serviceProvider;

    public CustomAccountFactory(IAccessTokenProviderAccessor accessor, 
        IServiceProvider serviceProvider,
        ILogger<CustomAccountFactory> logger)
        : base(accessor)
    {
        this.serviceProvider = serviceProvider;
        this.logger = logger;
    }

    public async override ValueTask<ClaimsPrincipal> CreateUserAsync(
        RemoteUserAccount account,
        RemoteAuthenticationUserOptions options)
    {
        var initialUser = await base.CreateUserAsync(account, options);

        if (initialUser.Identity.IsAuthenticated)
        {
            var userIdentity = (ClaimsIdentity)initialUser.Identity;

            try
            {
                var graphClient = ActivatorUtilities
                    .CreateInstance<GraphServiceClient>(serviceProvider);
                var request = graphClient.Me.Request();
                var user = await request.GetAsync();

                if (user != null)
                {
                    userIdentity.AddClaim(new Claim("mobilephone", 
                        user.MobilePhone));
                }
            }
            catch (ServiceException exception)
            {
                logger.LogError("Graph API service failure: {Message}",
                    exception.Message);
            }
        }

        return initialUser;
    }
}
```

`Program.Main` (`Program.cs`) で、カスタム ユーザー アカウント ファクトリを使用するように MSAL 認証を構成します。<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> を拡張するカスタム ユーザー アカウント クラスがアプリで使用されている場合は、そのカスタム ユーザー アカウント クラスを次のコード内の <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> と入れ替えます。

```csharp
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.Extensions.Configuration;

builder.Services.AddMsalAuthentication<RemoteAuthenticationState, 
    RemoteUserAccount>(options =>
    {
        builder.Configuration.Bind("AzureAd", 
            options.ProviderOptions.Authentication);
        options.ProviderOptions.DefaultAccessTokenScopes.Add(
            "https://graph.microsoft.com/User.Read");
    })
    .AddAccountClaimsPrincipalFactory<RemoteAuthenticationState, RemoteUserAccount, 
        CustomAccountFactory>();
```

::: moniker-end

## <a name="named-client-with-graph-api"></a>名前付きクライアントと Graph API

このセクションの例では、Graph API の名前付き <xref:System.Net.Http.HttpClient> を使用して、通話を処理するユーザーの携帯電話番号を取得します。

このセクションの例では、スタンドアロンのプロジェクト ファイルまたは **`Client`** アプリのプロジェクト ファイルに [`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) のパッケージ参照が必要です。

Graph API を使用するには、次のクラスとプロジェクト構成を作成します。 次のクラスおよび構成は、この記事の次の各サブセクションで使用されます。

* [コンポーネントから Graph API を呼び出す](#call-graph-api-from-a-component)
* [Graph API と名前付きクライアントを使用してユーザー要求をカスタマイズする](#customize-user-claims-with-graph-api-and-a-named-client)

Azure portal の AAD 領域で Microsoft Graph API スコープを追加した後、アプリの構成済み Graph API 用ハンドラーに必要なスコープを提供します。 次の例では、`User.Read` スコープのハンドラーを構成します。 さらにスコープを追加できます。

`GraphAuthorizationMessageHandler.cs`:

```csharp
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class GraphAPIAuthorizationMessageHandler : AuthorizationMessageHandler
{
    public GraphAPIAuthorizationMessageHandler(IAccessTokenProvider provider,
        NavigationManager navigationManager)
        : base(provider, navigationManager)
    {
        ConfigureHandler(
            authorizedUrls: new[] { "https://graph.microsoft.com" },
            scopes: new[] { "https://graph.microsoft.com/User.Read" });
    }
}
```

`Program.Main` (`Program.cs`) では、Graph API の名前付き <xref:System.Net.Http.HttpClient> を構成します。

```csharp
builder.Services.AddScoped<GraphAPIAuthorizationMessageHandler>();

builder.Services.AddHttpClient("GraphAPI",
        client => client.BaseAddress = new Uri("https://graph.microsoft.com"))
    .AddHttpMessageHandler<GraphAPIAuthorizationMessageHandler>();
```

### <a name="call-graph-api-from-a-component"></a>コンポーネントから Graph API を呼び出す

このセクションでは、この記事で既に説明した [Graph 認可メッセージ ハンドラー (`GraphAuthorizationMessageHandler.cs`) とアプリへの `Program.Main` の追加](#named-client-with-graph-api)を使用します。これにより、Graph API の名前付き <xref:System.Net.Http.HttpClient> が提供されます。

Razor コンポーネントでは、次のようになります。

* Graph API の <xref:System.Net.Http.HttpClient> を作成し、ユーザーのプロファイル データの要求を発行します。
* `UserInfo.cs` クラスにより、必要なユーザー プロファイル プロパティを、<xref:System.Text.Json.Serialization.JsonPropertyNameAttribute> 属性と、AAD によってそれらのプロパティ用に使用される JSON 名を使用して指定します。

`Pages/CallUser.razor`:

```razor
@page "/CallUser"
@using System.ComponentModel.DataAnnotations
@using System.Text.Json.Serialization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@using Microsoft.Extensions.Logging
@inject IAccessTokenProvider TokenProvider
@inject IHttpClientFactory ClientFactory
@inject ILogger<CallUser> Logger

<h3>Call User</h3>

<EditForm Model="@callInfo" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <p>
        <label>
            Message:
            <InputTextArea @bind-Value="callInfo.Message" />
        </label>
    </p>

    <button type="submit">Place call</button>

    <p>
        @formStatus
    </p>
</EditForm>

@code {
    private string formStatus;
    private CallInfo callInfo = new CallInfo();

    private async Task HandleValidSubmit()
    {
        var tokenResult = await TokenProvider.RequestAccessToken(
            new AccessTokenRequestOptions
            {
                Scopes = new[] { "https://graph.microsoft.com/User.Read" }
            });

        if (tokenResult.TryGetToken(out var token))
        {
            var client = ClientFactory.CreateClient("GraphAPI");

            var userInfo = await client.GetFromJsonAsync<UserInfo>("v1.0/me");

            if (userInfo != null)
            {
                // Use userInfo.MobilePhone and callInfo.Message to make a call

                formStatus = "Form successfully processed.";
                Logger.LogInformation(
                    $"Form successfully processed at {DateTime.UtcNow}. " +
                    $"Mobile Phone: {userInfo.MobilePhone}");
            }
        }
        else
        {
            formStatus = "There was a problem processing the form.";
            Logger.LogError("Token failure");
        }
    }

    private class CallInfo
    {
        [Required]
        [StringLength(1000, ErrorMessage = "Message too long (1,000 char limit)")]
        public string Message { get; set; }
    }

    private class UserInfo
    {
        [JsonPropertyName("mobilePhone")]
        public string MobilePhone { get; set; }
    }
}
```

### <a name="customize-user-claims-with-graph-api-and-a-named-client"></a>Graph API と名前付きクライアントを使用してユーザー要求をカスタマイズする

このセクションでは、この記事で既に説明した [Graph 認可メッセージ ハンドラー (`GraphAuthorizationMessageHandler.cs`) とアプリへの `Program.Main` の追加](#named-client-with-graph-api)を使用します。これにより、Graph API の名前付き <xref:System.Net.Http.HttpClient> が提供されます。

次の例のアプリでは、AAD ユーザー プロファイルの携帯電話番号からユーザーの携帯電話番号要求を作成します。 このアプリには、AAD で構成された `User.Read` Graph API スコープが必要です。

`UserInfo.cs` クラスをアプリに追加し、必要なユーザー プロファイル プロパティを、<xref:System.Text.Json.Serialization.JsonPropertyNameAttribute> 属性と、AAD によってそれらのプロパティ用に使用される JSON 名を使用して指定します。

```csharp
using System.Text.Json.Serialization;

public class UserInfo
{
    [JsonPropertyName("mobilePhone")]
    public string MobilePhone { get; set; }
}
```

次のカスタム ユーザー アカウント ファクトリでは、フレームワークの <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> はユーザーのアカウントを表します。 <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> を拡張するカスタム ユーザー アカウント クラスがアプリで必要な場合は、そのカスタム ユーザー アカウント クラスを次のコード内の <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> と入れ替えます。

`CustomAccountFactory.cs`:

```csharp
using System.Net.Http;
using System.Net.Http.Json;
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication.Internal;
using Microsoft.Extensions.Logging;

public class CustomAccountFactory
    : AccountClaimsPrincipalFactory<RemoteUserAccount>
{
    private readonly ILogger<CustomAccountFactory> logger;
    private readonly IHttpClientFactory clientFactory;

    public CustomAccountFactory(IAccessTokenProviderAccessor accessor, 
        IHttpClientFactory clientFactory, 
        ILogger<CustomAccountFactory> logger)
        : base(accessor)
    {
        this.clientFactory = clientFactory;
        this.logger = logger;
    }

    public async override ValueTask<ClaimsPrincipal> CreateUserAsync(
        RemoteUserAccount account,
        RemoteAuthenticationUserOptions options)
    {
        var initialUser = await base.CreateUserAsync(account, options);

        if (initialUser.Identity.IsAuthenticated)
        {
            var userIdentity = (ClaimsIdentity)initialUser.Identity;

            try
            {
                var client = clientFactory.CreateClient("GraphAPI");

                var userInfo = await client.GetFromJsonAsync<UserInfo>("v1.0/me");

                if (userInfo != null)
                {
                    userIdentity.AddClaim(new Claim("mobilephone", 
                        userInfo.MobilePhone));
                }
            }
            catch (AccessTokenNotAvailableException exception)
            {
                logger.LogError("Graph API access token failure: {Message}",
                    exception.Message);
            }
        }

        return initialUser;
    }
}
```

`Program.Main` (`Program.cs`) で、カスタム ユーザー アカウント ファクトリを使用するように MSAL 認証を構成します。<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> を拡張するカスタム ユーザー アカウント クラスがアプリで使用されている場合は、そのカスタム ユーザー アカウント クラスを次のコード内の <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> と入れ替えます。

```csharp
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

...

builder.Services.AddMsalAuthentication<RemoteAuthenticationState, 
    RemoteUserAccount>(options =>
    {
        builder.Configuration.Bind("AzureAd", 
            options.ProviderOptions.Authentication);
    })
    .AddAccountClaimsPrincipalFactory<RemoteAuthenticationState, RemoteUserAccount, 
        CustomAccountFactory>();
```

前の例は、MSAL で AAD 認証が使用されるアプリの場合です。 OIDC と API 認証にも、同様のパターンがあります。 詳細については、「[ペイロード要求を使用してユーザーをカスタマイズする](xref:blazor/security/webassembly/additional-scenarios#customize-the-user-with-a-payload-claim)」セクションの例を参照してください。
