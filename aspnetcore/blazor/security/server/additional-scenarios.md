---
title: ASP.NET Core Blazor Server のセキュリティに関するその他のシナリオ
author: guardrex
description: セキュリティに関するその他のシナリオ用に Blazor Server を構成する方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/06/2020
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
uid: blazor/security/server/additional-scenarios
ms.openlocfilehash: d47cfba75b640f57cc713049594d4e8acd1fcd0e
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280325"
---
# <a name="aspnet-core-blazor-server-additional-security-scenarios"></a>ASP.NET Core Blazor Server のセキュリティに関するその他のシナリオ

::: moniker range=">= aspnetcore-5.0"

<h2 id="pass-tokens-to-a-blazor-server-app">Blazor Server アプリにトークンを渡す</h2>

Blazor Server アプリの Razor コンポーネントの外部で使用できるトークンは、このセクションで説明する方法を使用してコンポーネントに渡すことができます。

通常の Razor Pages や MVC アプリの場合と同じように、Blazor Server アプリを認証します。 トークンをプロビジョニングし、認証 cookie に保存します。 次に例を示します。

```csharp
using Microsoft.AspNetCore.Authentication.OpenIdConnect;
using Microsoft.IdentityModel.Protocols.OpenIdConnect;

...

services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options =>
{
    options.ResponseType = OpenIdConnectResponseType.Code;
    options.SaveTokens = true;

    options.Scope.Add("offline_access");
});
```

必要があれば、`options.Scope.Add("{SCOPE}");` を使用して、さらにスコープを追加します。ここで、プレースホルダー `{SCOPE}` は追加するスコープです。

[依存関係の挿入 (DI)](xref:blazor/fundamentals/dependency-injection) からトークンを解決するために Blazor アプリ内で使用できる **スコープを持つ** トークン プロバイダー サービスを定義します。

```csharp
public class TokenProvider
{
    public string AccessToken { get; set; }
    public string RefreshToken { get; set; }
}
```

`Startup.ConfigureServices` で、次のサービスを追加します。

* `IHttpClientFactory`
* `TokenProvider`

```csharp
services.AddHttpClient();
services.AddScoped<TokenProvider>();
```

アクセス トークンと更新トークンを使用して最初のアプリの状態を渡すクラスを定義します。

```csharp
public class InitialApplicationState
{
    public string AccessToken { get; set; }
    public string RefreshToken { get; set; }
}
```

`_Host.cshtml` ファイルで、`InitialApplicationState` のインスタンスを作成し、それをパラメーターとしてアプリに渡します。

```cshtml
@using Microsoft.AspNetCore.Authentication

...

@{
    var tokens = new InitialApplicationState
    {
        AccessToken = await HttpContext.GetTokenAsync("access_token"),
        RefreshToken = await HttpContext.GetTokenAsync("refresh_token")
    };
}

<component type="typeof(App)" param-InitialState="tokens" 
    render-mode="ServerPrerendered" />
```

`App` コンポーネント (`App.razor`) で、サービスを解決し、パラメーターからのデータを使用してそれを初期化します。

```razor
@inject TokenProvider TokenProvider

...

@code {
    [Parameter]
    public InitialApplicationState InitialState { get; set; }

    protected override Task OnInitializedAsync()
    {
        TokenProvider.AccessToken = InitialState.AccessToken;
        TokenProvider.RefreshToken = InitialState.RefreshToken;

        return base.OnInitializedAsync();
    }
}
```

[`Microsoft.AspNet.WebApi.Client`](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Client) NuGet パッケージのパッケージ参照をアプリに追加します。

セキュリティで保護された API 要求を行うサービスで、トークン プロバイダーを挿入し、API 要求のトークンを取得します。

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

public class WeatherForecastService
{
    private readonly HttpClient http;
    private readonly TokenProvider tokenProvider;

    public WeatherForecastService(IHttpClientFactory clientFactory, 
        TokenProvider tokenProvider)
    {
        http = clientFactory.CreateClient();
        this.tokenProvider = tokenProvider;
    }

    public async Task<WeatherForecast[]> GetForecastAsync()
    {
        var token = tokenProvider.AccessToken;
        var request = new HttpRequestMessage(HttpMethod.Get, 
            "https://localhost:5003/WeatherForecast");
        request.Headers.Add("Authorization", $"Bearer {token}");
        var response = await http.SendAsync(request);
        response.EnsureSuccessStatusCode();

        return await response.Content.ReadAsAsync<WeatherForecast[]>();
    }
}
```

<h2 id="set-the-authentication-scheme">認証スキームを設定する</h2>

複数の認証ミドルウェアを使用していることにより、複数の認証スキームがあるアプリの場合、Blazor で使用されるスキームを、`Startup.Configure` のエンドポイント構成で明示的に設定できます。 次の例では、Azure Active Directory スキームを設定します。

```csharp
endpoints.MapBlazorHub().RequireAuthorization(
    new AuthorizeAttribute 
    {
        AuthenticationSchemes = AzureADDefaults.AuthenticationScheme
    });
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<h2 id="pass-tokens-to-a-blazor-server-app">Blazor Server アプリにトークンを渡す</h2>

Blazor Server アプリの Razor コンポーネントの外部で使用できるトークンは、このセクションで説明する方法を使用してコンポーネントに渡すことができます。

通常の Razor Pages や MVC アプリの場合と同じように、Blazor Server アプリを認証します。 トークンをプロビジョニングし、認証 cookie に保存します。 次に例を示します。

```csharp
using Microsoft.AspNetCore.Authentication.OpenIdConnect;
using Microsoft.IdentityModel.Protocols.OpenIdConnect;

...

services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options =>
{
    options.ResponseType = OpenIdConnectResponseType.Code;
    options.SaveTokens = true;

    options.Scope.Add("offline_access");
});
```

必要があれば、`options.Scope.Add("{SCOPE}");` を使用して、さらにスコープを追加します。ここで、プレースホルダー `{SCOPE}` は追加するスコープです。

必要があれば、`options.Resource = "{RESOURCE}";` を使用してリソースを指定します。ここで、プレースホルダー `{RESOURCE}` はリソースです。 次に例を示します。

```csharp
options.Resource = "https://graph.microsoft.com";
```

アクセス トークンと更新トークンを使用して最初のアプリの状態を渡すクラスを定義します。

```csharp
public class InitialApplicationState
{
    public string AccessToken { get; set; }
    public string RefreshToken { get; set; }
}
```

[依存関係の挿入 (DI)](xref:blazor/fundamentals/dependency-injection) からトークンを解決するために Blazor アプリ内で使用できる **スコープを持つ** トークン プロバイダー サービスを定義します。

```csharp
public class TokenProvider
{
    public string AccessToken { get; set; }
    public string RefreshToken { get; set; }
}
```

`Startup.ConfigureServices` で、次のサービスを追加します。

* `IHttpClientFactory`
* `TokenProvider`

```csharp
services.AddHttpClient();
services.AddScoped<TokenProvider>();
```

`_Host.cshtml` ファイルで、`InitialApplicationState` のインスタンスを作成し、それをパラメーターとしてアプリに渡します。

```cshtml
@using Microsoft.AspNetCore.Authentication

...

@{
    var tokens = new InitialApplicationState
    {
        AccessToken = await HttpContext.GetTokenAsync("access_token"),
        RefreshToken = await HttpContext.GetTokenAsync("refresh_token")
    };
}

<app>
    <component type="typeof(App)" param-InitialState="tokens" 
        render-mode="ServerPrerendered" />
</app>
```

`App` コンポーネント (`App.razor`) で、サービスを解決し、パラメーターからのデータを使用してそれを初期化します。

```razor
@inject TokenProvider TokenProvider

...

@code {
    [Parameter]
    public InitialApplicationState InitialState { get; set; }

    protected override Task OnInitializedAsync()
    {
        TokenProvider.AccessToken = InitialState.AccessToken;
        TokenProvider.RefreshToken = InitialState.RefreshToken;

        return base.OnInitializedAsync();
    }
}
```

[`Microsoft.AspNet.WebApi.Client`](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Client) NuGet パッケージのパッケージ参照をアプリに追加します。

セキュリティで保護された API 要求を行うサービスで、トークン プロバイダーを挿入し、API 要求のトークンを取得します。

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

public class WeatherForecastService
{
    private readonly HttpClient http;
    private readonly TokenProvider tokenProvider;

    public WeatherForecastService(IHttpClientFactory clientFactory, 
        TokenProvider tokenProvider)
    {
        http = clientFactory.CreateClient();
        this.tokenProvider = tokenProvider;
    }

    public async Task<WeatherForecast[]> GetForecastAsync()
    {
        var token = tokenProvider.AccessToken;
        var request = new HttpRequestMessage(HttpMethod.Get, 
            "https://localhost:5003/WeatherForecast");
        request.Headers.Add("Authorization", $"Bearer {token}");
        var response = await http.SendAsync(request);
        response.EnsureSuccessStatusCode();

        return await response.Content.ReadAsAsync<WeatherForecast[]>();
    }
}
```

<h2 id="set-the-authentication-scheme">認証スキームを設定する</h2>

複数の認証ミドルウェアを使用していることにより、複数の認証スキームがあるアプリの場合、Blazor で使用されるスキームを、`Startup.Configure` のエンドポイント構成で明示的に設定できます。 次の例では、Azure Active Directory スキームを設定します。

```csharp
endpoints.MapBlazorHub().RequireAuthorization(
    new AuthorizeAttribute 
    {
        AuthenticationSchemes = AzureADDefaults.AuthenticationScheme
    });
```

## <a name="use-openid-connect-oidc-v20-endpoints"></a>OpenID Connect (OIDC) v2.0 エンドポイントを使用する

5\.0 より前のバージョンの ASP.NET Core では、認証ライブラリと Blazor テンプレートに OpenID Connect (OIDC) v1.0 エンドポイントが使用されます。 5\.0 より前のバージョンの ASP.NET Core で v2.0 エンドポイントを使用するには、<xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions> に <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority?displayProperty=nameWithType> オプションを構成します。

```csharp
services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, 
    options =>
    {
        options.Authority += "/v2.0";
    }
```

または、アプリ設定ファイル (`appsettings.json`) で設定を行うこともできます。

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/common/oauth2/v2.0/",
    ...
  }
}
```

証明機関へのセグメントを追跡することがアプリの OIDC プロバイダー (AAD 以外のプロバイダーなど) にとって適切でない場合は、<xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority> プロパティを直接設定します。 <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions> またはアプリ設定ファイルで <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority> キーを使用してプロパティを設定します。

### <a name="code-changes"></a>コード変更

* ID トークンの要求のリストは、v2.0 エンドポイントで変更されています。 詳細については、「[Microsoft ID プラットフォーム (v2.0) に更新する理由](/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison)」を Azure のドキュメントで参照してください。
* リソースは v2.0 エンドポイントのスコープ URI で指定されているため、<xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions> の <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Resource?displayProperty=nameWithType> プロパティ設定を削除します。

  ```csharp
  services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options => 
      {
          ...
          options.Resource = "...";    // REMOVE THIS LINE
          ...
      }
  ```

  詳細については、Azure のドキュメントの「[リソースではなくスコープ](/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison#scopes-not-resources)」を参照してください。

### <a name="app-id-uri"></a>アプリ ID URI

* v2.0 エンドポイントを使用するとき、API により *`App ID URI`* が定義されます。これは API の一意の識別子を表すものです。
* すべてのスコープにはプレフィックスとしてアプリ ID URI が含まれています。v2.0 エンドポイントからはアプリ ID URI を対象ユーザーとするアクセス トークンが発行されます。
* v2.0 エンドポイントを使用するとき、Server API で構成されたクライアント ID は API アプリケーション ID (クライアント ID) からアプリ ID URI に変更されます。

`appsettings.json`:

```json
{
  "AzureAd": {
    ...
    "ClientId": "https://{TENANT}.onmicrosoft.com/{APP NAME}"
    ...
  }
}
```

使用するアプリ ID URI は、OIDC プロバイダーのアプリ登録の説明で見つけることができます。

::: moniker-end
