---
title: ASP.NET Core Blazor WebAssembly のセキュリティに関するその他のシナリオ
author: guardrex
description: セキュリティに関するその他のシナリオ用に Blazor WebAssembly を構成する方法について説明します。
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
uid: blazor/security/webassembly/additional-scenarios
ms.openlocfilehash: fb5b6f75959d9933e228b0288e70498ef05efc4a
ms.sourcegitcommit: da5a5bed5718a9f8db59356ef8890b4b60ced6e9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/22/2021
ms.locfileid: "98710634"
---
# <a name="aspnet-core-blazor-webassembly-additional-security-scenarios"></a>ASP.NET Core Blazor WebAssembly のセキュリティに関するその他のシナリオ

作成者: [Javier Calvarro Nelson](https://github.com/javiercn)、[Luke Latham](https://github.com/guardrex)

## <a name="attach-tokens-to-outgoing-requests"></a>送信要求にトークンを添付する

<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> は、送信 <xref:System.Net.Http.HttpResponseMessage> インスタンスにアクセス トークンをアタッチするために使用される <xref:System.Net.Http.DelegatingHandler> です。 トークンは、フレームワークによって登録されている <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.IAccessTokenProvider> サービスを使用して取得されます。 トークンを取得できない場合は、<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenNotAvailableException> がスローされます。 <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenNotAvailableException> には、<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenNotAvailableException.Redirect%2A> メソッドがあります。このメソッドを使用すると、ユーザーを ID プロバイダーに移動して、新しいトークンを取得できます。

便宜上、フレームワークには、アプリのベース アドレスを承認された URL として使用して事前構成が行われた <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.BaseAddressAuthorizationMessageHandler> が用意されています。 **アクセス トークンは、要求 URI がアプリのベース URI 内にある場合にのみ追加されます。** 送信要求 URI がアプリのベース URI 内にない場合は、[カスタム `AuthorizationMessageHandler` クラス (*推奨*)](#custom-authorizationmessagehandler-class) を使用するか、または [`AuthorizationMessageHandler`](#configure-authorizationmessagehandler) を構成します。

> [!NOTE]
> サーバー API アクセス用のクライアント アプリ構成に加えて、クライアントとサーバーが同じベース アドレス内に存在しない場合にクロスオリジン要求 (CORS) がサーバー API によって許可される必要があります。 サーバー側の CORS 構成の詳細については、この記事で後述する「[クロスオリジン リソース共有 (CORS)](#cross-origin-resource-sharing-cors)」セクションを参照してください。

次の例では

* <xref:Microsoft.Extensions.DependencyInjection.HttpClientFactoryServiceCollectionExtensions.AddHttpClient%2A> によって、<xref:System.Net.Http.IHttpClientFactory> および関連サービスがサービス コレクションに追加され、名前付き <xref:System.Net.Http.HttpClient> (`ServerAPI`) が構成されます。 <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> は、要求を送信するときのリソース URI のベース アドレスです。 <xref:System.Net.Http.IHttpClientFactory> は、[`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) NuGet パッケージによって提供されます。
* <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.BaseAddressAuthorizationMessageHandler> は、送信 <xref:System.Net.Http.HttpResponseMessage> インスタンスにアクセス トークンをアタッチするために使用される <xref:System.Net.Http.DelegatingHandler> です。 アクセス トークンは、要求 URI がアプリのベース URI 内にある場合にのみ追加されます。
* <xref:System.Net.Http.IHttpClientFactory.CreateClient%2A?displayProperty=nameWithType> では、名前付き <xref:System.Net.Http.HttpClient> (`ServerAPI`) に対応する構成を使用して、送信要求用に <xref:System.Net.Http.HttpClient> インスタンスが作成および構成されます。

```csharp
using System.Net.Http;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

...

builder.Services.AddHttpClient("ServerAPI", 
        client => client.BaseAddress = new Uri("https://www.example.com/base"))
    .AddHttpMessageHandler<BaseAddressAuthorizationMessageHandler>();

builder.Services.AddScoped(sp => sp.GetRequiredService<IHttpClientFactory>()
    .CreateClient("ServerAPI"));
```

Blazor WebAssembly のホストされたプロジェクト テンプレートに基づく Blazor アプリの場合、要求 URI は既定ではアプリのベース URI 内にあります。 したがって、<xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> (`new Uri(builder.HostEnvironment.BaseAddress)`) は、プロジェクト テンプレートから生成されたアプリ内の <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> に割り当てられます。

構成された <xref:System.Net.Http.HttpClient> を使用し、[`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) パターンを使用して、承認された要求を行います。

```razor
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@inject HttpClient Http

...

protected override async Task OnInitializedAsync()
{
    private ExampleType[] examples;

    try
    {
        examples = 
            await Http.GetFromJsonAsync<ExampleType[]>("ExampleAPIMethod");

        ...
    }
    catch (AccessTokenNotAvailableException exception)
    {
        exception.Redirect();
    }
}
```

### <a name="custom-authorizationmessagehandler-class"></a>カスタム `AuthorizationMessageHandler` クラス

*このセクションのこのガイダンスは、アプリのベース URI 内に存在しない URI に対して送信要求を行うクライアント アプリに推奨されます。*

次の例では、カスタム クラスによって、<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> が、<xref:System.Net.Http.HttpClient> 用の <xref:System.Net.Http.DelegatingHandler> として使用するために拡張されます。 <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler.ConfigureHandler%2A> によって、アクセス トークンを使用して送信 HTTP 要求を承認できるように、このハンドラーが構成されます。 アクセス トークンがアタッチされるのは、承認された URL の少なくとも 1 つが要求 URI (<xref:System.Net.Http.HttpRequestMessage.RequestUri?displayProperty=nameWithType>) のベースである場合に限られます。

```csharp
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class CustomAuthorizationMessageHandler : AuthorizationMessageHandler
{
    public CustomAuthorizationMessageHandler(IAccessTokenProvider provider, 
        NavigationManager navigationManager)
        : base(provider, navigationManager)
    {
        ConfigureHandler(
            authorizedUrls: new[] { "https://www.example.com/base" },
            scopes: new[] { "example.read", "example.write" });
    }
}
```

`Program.Main` (`Program.cs`) では、`CustomAuthorizationMessageHandler` がスコープされたサービスとして登録され、名前付き <xref:System.Net.Http.HttpClient> によって作成された送信 <xref:System.Net.Http.HttpResponseMessage> インスタンスの <xref:System.Net.Http.DelegatingHandler> として構成されます。

```csharp
builder.Services.AddScoped<CustomAuthorizationMessageHandler>();

builder.Services.AddHttpClient("ServerAPI",
        client => client.BaseAddress = new Uri("https://www.example.com/base"))
    .AddHttpMessageHandler<CustomAuthorizationMessageHandler>();
```

Blazor WebAssembly のホストされたプロジェクト テンプレートに基づく Blazor アプリの場合、<xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> (`new Uri(builder.HostEnvironment.BaseAddress)`) は既定では <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> に割り当てられます。

構成が行われた <xref:System.Net.Http.HttpClient> を使用し、[`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) パターンを使用して、承認された要求を行います。 <xref:System.Net.Http.IHttpClientFactory.CreateClient%2A> ([`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) パッケージ) を使用してクライアントが作成される場合にサーバー API への要求を行うと、アクセス トークンが含まれるインスタンスが <xref:System.Net.Http.HttpClient> に提供されます。 要求 URI が次の例 (`ExampleAPIMethod`) のように相対 URI である場合、それは、クライアント アプリから要求があったときに <xref:System.Net.Http.HttpClient.BaseAddress> に結合されます。

```razor
@inject IHttpClientFactory ClientFactory

...

@code {
    private ExampleType[] examples;

    protected override async Task OnInitializedAsync()
    {
        try
        {
            var client = ClientFactory.CreateClient("ServerAPI");

            examples = 
                await client.GetFromJsonAsync<ExampleType[]>("ExampleAPIMethod");
        }
        catch (AccessTokenNotAvailableException exception)
        {
            exception.Redirect();
        }
    }
}
```

### <a name="configure-authorizationmessagehandler"></a>`AuthorizationMessageHandler` を構成する

<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler.ConfigureHandler%2A> メソッドを使用すれば、承認された URL、スコープ、および戻り先 URL で <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> を構成できます。 <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler.ConfigureHandler%2A> によって、アクセス トークンを使用して送信 HTTP 要求を承認できるように、ハンドラーが構成されます。 アクセス トークンがアタッチされるのは、承認された URL の少なくとも 1 つが要求 URI (<xref:System.Net.Http.HttpRequestMessage.RequestUri?displayProperty=nameWithType>) のベースである場合に限られます。 要求 URI が相対 URI である場合、これは <xref:System.Net.Http.HttpClient.BaseAddress> に結合されます。

次の例では、`Program.Main` (`Program.cs`) で、<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> によって <xref:System.Net.Http.HttpClient> が構成されます。

```csharp
using System.Net.Http;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

...

builder.Services.AddScoped(sp => new HttpClient(
    sp.GetRequiredService<AuthorizationMessageHandler>()
    .ConfigureHandler(
        authorizedUrls: new[] { "https://www.example.com/base" },
        scopes: new[] { "example.read", "example.write" }))
    {
        BaseAddress = new Uri("https://www.example.com/base")
    });
```

Blazor WebAssembly のホストされたプロジェクト テンプレートに基づく Blazor アプリの場合、<xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> は既定では以下に割り当てられます。

* <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> (`new Uri(builder.HostEnvironment.BaseAddress)`)。
* `authorizedUrls` 配列の URL。

## <a name="typed-httpclient"></a>型指定された `HttpClient`

単一クラス内のすべての HTTP およびトークンの取得に関する問題を処理する、型指定されたクライアントを定義できます。

`WeatherForecastClient.cs`:

```csharp
using System.Net.Http;
using System.Net.Http.Json;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using static {APP ASSEMBLY}.Data;

public class WeatherForecastClient
{
    private readonly HttpClient http;
 
    public WeatherForecastClient(HttpClient http)
    {
        this.http = http;
    }
 
    public async Task<WeatherForecast[]> GetForecastAsync()
    {
        var forecasts = new WeatherForecast[0];

        try
        {
            forecasts = await http.GetFromJsonAsync<WeatherForecast[]>(
                "WeatherForecast");
        }
        catch (AccessTokenNotAvailableException exception)
        {
            exception.Redirect();
        }

        return forecasts;
    }
}
```

プレースホルダー `{APP ASSEMBLY}` は、アプリのアセンブリ名です (例: `using static BlazorSample.Data;`)。

`Program.Main` (`Program.cs`):

```csharp
using System.Net.Http;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

...

builder.Services.AddHttpClient<WeatherForecastClient>(
        client => client.BaseAddress = new Uri("https://www.example.com/base"))
    .AddHttpMessageHandler<BaseAddressAuthorizationMessageHandler>();
```

Blazor WebAssembly のホストされたプロジェクト テンプレートに基づく Blazor アプリの場合、<xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> (`new Uri(builder.HostEnvironment.BaseAddress)`) は既定では <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> に割り当てられます。

`FetchData` コンポーネント (`Pages/FetchData.razor`):

```razor
@inject WeatherForecastClient Client

...

protected override async Task OnInitializedAsync()
{
    forecasts = await Client.GetForecastAsync();
}
```

## <a name="configure-the-httpclient-handler"></a>`HttpClient` ハンドラーを構成する

ハンドラーは、送信 HTTP 要求の <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler.ConfigureHandler%2A> を使用して、さらに構成を行うことができます。

`Program.Main` (`Program.cs`):

```csharp
builder.Services.AddHttpClient<WeatherForecastClient>(
        client => client.BaseAddress = new Uri("https://www.example.com/base"))
    .AddHttpMessageHandler(sp => sp.GetRequiredService<AuthorizationMessageHandler>()
    .ConfigureHandler(
        authorizedUrls: new [] { "https://www.example.com/base" },
        scopes: new[] { "example.read", "example.write" }));
```

Blazor WebAssembly のホストされたプロジェクト テンプレートに基づく Blazor アプリの場合、<xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> は既定では以下に割り当てられます。

* <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> (`new Uri(builder.HostEnvironment.BaseAddress)`)。
* `authorizedUrls` 配列の URL。

## <a name="unauthenticated-or-unauthorized-web-api-requests-in-an-app-with-a-secure-default-client"></a>セキュリティで保護された既定のクライアントを使用する、アプリ内の認証または承認されていない Web API 要求

通常、Blazor WebAssembly アプリがセキュリティで保護された既定の <xref:System.Net.Http.HttpClient> を使用する場合、アプリでは、名前付きの <xref:System.Net.Http.HttpClient> の構成を行うことで、認証または承認されていない Web API 要求が行われます。

`Program.Main` (`Program.cs`):

```csharp
builder.Services.AddHttpClient("ServerAPI.NoAuthenticationClient", 
    client => client.BaseAddress = new Uri("https://www.example.com/base"));
```

Blazor WebAssembly のホストされたプロジェクト テンプレートに基づく Blazor アプリの場合、<xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> (`new Uri(builder.HostEnvironment.BaseAddress)`) は既定では <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> に割り当てられます。

前述の登録は、セキュリティで保護された既定の <xref:System.Net.Http.HttpClient> 登録に追加されます。

コンポーネントでは、<xref:System.Net.Http.IHttpClientFactory> ([`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) パッケージ) から <xref:System.Net.Http.HttpClient> が作成され、認証または承認されていない要求が行われます。

```razor
@inject IHttpClientFactory ClientFactory

...

@code {
    private WeatherForecast[] forecasts;

    protected override async Task OnInitializedAsync()
    {
        var client = ClientFactory.CreateClient("ServerAPI.NoAuthenticationClient");

        forecasts = await client.GetFromJsonAsync<WeatherForecast[]>(
            "WeatherForecastNoAuthentication");
    }
}
```

> [!NOTE]
> サーバー API のコントローラー (前の例では `WeatherForecastNoAuthenticationController`) では、[`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) 属性を使用してマークされることはありません。

既定の <xref:System.Net.Http.HttpClient> インスタンスとしてセキュリティで保護されたクライアントを使用するか、セキュリティで保護されていないクライアントを使用するかは、開発者が決定します。 この決定を行う方法の 1 つは、アプリが通信する認証済みのエンドポイントと認証されていないエンドポイントの数を考慮することです。 アプリの要求の大部分が API エンドポイントをセキュリティで保護する場合は、認証された <xref:System.Net.Http.HttpClient> インスタンスを既定として使用します。 それ以外の場合は、認証されていない <xref:System.Net.Http.HttpClient> インスタンスを既定として登録します。

<xref:System.Net.Http.IHttpClientFactory> を使用する別の方法として、匿名エンドポイントへの認証されていないアクセス用に、[型指定されたクライアント](#typed-httpclient)を作成することもできます。

## <a name="request-additional-access-tokens"></a>追加のアクセス トークンを要求する

アクセス トークンは、`IAccessTokenProvider.RequestAccessToken` を呼び出して手動で取得できます。 次の例では、既定の <xref:System.Net.Http.HttpClient> に対して、アプリが追加のスコープを必要としています。 Microsoft Authentication Library (MSAL) の例では、`MsalProviderOptions` を使用してスコープが構成されます。

`Program.Main` (`Program.cs`):

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...

    options.ProviderOptions.AdditionalScopesToConsent.Add("{CUSTOM SCOPE 1}");
    options.ProviderOptions.AdditionalScopesToConsent.Add("{CUSTOM SCOPE 2}");
}
```

前の例の `{CUSTOM SCOPE 1}` と `{CUSTOM SCOPE 2}` のプレースホルダーは、カスタム スコープです。

`IAccessTokenProvider.RequestToken` メソッドには、指定されたスコープ セットを使用して、アプリでアクセス トークンをプロビジョニングできるようにするオーバーロードが用意されています。

Razor コンポーネントでは、次のようになります。

```razor
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@inject IAccessTokenProvider TokenProvider

...

var tokenResult = await TokenProvider.RequestAccessToken(
    new AccessTokenRequestOptions
    {
        Scopes = new[] { "{CUSTOM SCOPE 1}", "{CUSTOM SCOPE 2}" }
    });

if (tokenResult.TryGetToken(out var token))
{
    ...
}
```

前の例の `{CUSTOM SCOPE 1}` と `{CUSTOM SCOPE 2}` のプレースホルダーは、カスタム スコープです。

<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenResult.TryGetToken%2A?displayProperty=nameWithType> が次のように返します。

* `token` を使用する場合は `true`。
* トークンが取得されない場合は `false`。

## <a name="cross-origin-resource-sharing-cors"></a>クロスオリジン リソース共有 (CORS)

CORS 要求で資格情報 (承認 cookie またはヘッダー) を送信するときには、CORS ポリシーで `Authorization` ヘッダーが許可されている必要があります。

次のポリシーには、以下についての構成が含まれています。

* 要求元 (`http://localhost:5000`、`https://localhost:5001`)。
* 任意のメソッド (動詞)。
* `Content-Type` ヘッダーと `Authorization` ヘッダー。 カスタム ヘッダー (`x-custom-header`など) を許可するには、<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithHeaders*> の呼び出し時にヘッダーを一覧表示します。
* クライアント側の JavaScript コードによって設定された資格情報 (`credentials` プロパティが `include`に設定されています)。

```csharp
app.UseCors(policy => 
    policy.WithOrigins("http://localhost:5000", "https://localhost:5001")
    .AllowAnyMethod()
    .WithHeaders(HeaderNames.ContentType, HeaderNames.Authorization, "x-custom-header")
    .AllowCredentials());
```

Blazor のホストされたプロジェクト テンプレートに基づくホスト型 Blazor ソリューションでは、クライアントおよびサーバー アプリ用に同じベース アドレスが使用されます。 クライアント アプリの <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> は、既定では `builder.HostEnvironment.BaseAddress` の URI に設定されます。 Blazor のホストされたプロジェクト テンプレートから作成されるホスト型アプリの既定の構成では、CORS 構成は **必須ではありません**。 サーバー プロジェクトでホストされておらず、サーバー アプリのベース アドレスを共有していない追加のクライアント アプリでは、サーバー プロジェクト内の CORS 構成は **必須です**。

詳細については、「<xref:security/cors>」と、サンプル アプリの HTTP 要求テスター コンポーネント (`Components/HTTPRequestTester.razor`) を参照してください。

## <a name="handle-token-request-errors"></a>トークン要求エラーを処理する

シングル ページ アプリケーション (SPA) で OpenID Connect (OIDC) を使用してユーザーが認証されると、ユーザーが資格情報を入力したときに設定されるセッション cookie の形式で、SPA 内および Identity プロバイダー (IP) 内で、認証状態がローカルに維持されます。

通常、IP によってユーザーに出力されるトークンが有効なのは短時間のため (通常は約 1 時間)、クライアント アプリでは定期的に新しいトークンをフェッチする必要があります。 それ以外の場合は、許可されたトークンの有効期限が切れると、ユーザーがログアウトします。 ほとんどの場合、OIDC クライアントでは、ユーザーに対して認証の再要求を行うことなく、新しいトークンをプロビジョニングできます。これは、認証状態または IP 内に保持される "セッション" によるものです。

場合によっては、ユーザーの介入なしに、クライアントでトークンを取得できないことがあります。たとえば、何らかの理由でユーザーが明示的に IP からログアウトした場合などです。 このシナリオは、ユーザーが `https://login.microsoftonline.com` にアクセスしてログアウトした場合に発生します。これらのシナリオでは、ユーザーがログアウトしたことを、アプリはすぐに認識しません。クライアントで保持されるトークンは、有効でなくなった可能性があります。 また、クライアントでは、現在のトークンの有効期限が切れた後に、ユーザーの介入なしに新しいトークンをプロビジョニングすることはできません。

これらのシナリオは、トークンベースの認証に固有のものではありません。 これらは、SPA の性質の一部です。 また、認証 cookie が削除されると、cookie を使用する SPA でサーバー API を呼び出すこともできません。

保護されたリソースに対する API 呼び出しをアプリで実行するときは、次の点に注意する必要があります。

* API を呼び出すための新しいアクセス トークンをプロビジョニングするには、ユーザーの再認証が必要です。
* 有効かもしれないトークンがクライアントにある場合でも、そのトークンはユーザーによって取り消されているため、サーバーへの呼び出しが失敗する可能性があります。

アプリでトークンが要求されている場合は、次の 2 つの結果が考えられます。

* 要求が成功します。アプリには有効なトークンがあります。
* 要求が失敗します。新しいトークンを取得するために、ユーザーの再認証が必要となります。

トークン要求が失敗した場合は、リダイレクトを実行する前に、現在の状態を保存するかどうかを決定する必要があります。 次のようないくつかの方法がありますが、さらに複雑になります。

* 現在のページの状態をセッション ストレージに格納します。 [`OnInitializedAsync` ライフサイクル イベント](xref:blazor/components/lifecycle#component-initialization-methods) (<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>) 中に、続行する前に状態を復元できるかどうかを確認します。
* クエリ文字列パラメーターを追加して、以前に保存した状態を再ハイドレートする必要があることをアプリに通知する方法として使用します。
* 他の項目と競合するリスクなしにセッション ストレージにデータを格納するための一意識別子を持つクエリ文字列パラメーターを追加します。

以下の例では、次のことを行っています。

* ログイン ページにリダイレクトする前に、状態を保持します。
* クエリ文字列パラメーターを使用して、認証後に以前の状態を回復します。

```razor
...
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@inject IAccessTokenProvider TokenProvider
...

<EditForm Model="User" @onsubmit="OnSaveAsync">
    <label>User
        <InputText @bind-Value="User.Name" />
    </label>
    <label>Last name
        <InputText @bind-Value="User.LastName" />
    </label>
</EditForm>

@code {
    public class Profile
    {
        public string Name { get; set; }
        public string LastName { get; set; }
    }

    public Profile User { get; set; } = new Profile();

    protected async override Task OnInitializedAsync()
    {
        var currentQuery = new Uri(Navigation.Uri).Query;

        if (currentQuery.Contains("state=resumeSavingProfile"))
        {
            User = await JS.InvokeAsync<Profile>("sessionStorage.getState", 
                "resumeSavingProfile");
        }
    }

    public async Task OnSaveAsync()
    {
        var http = new HttpClient();
        http.BaseAddress = new Uri(Navigation.BaseUri);

        var resumeUri = Navigation.Uri + $"?state=resumeSavingProfile";

        var tokenResult = await TokenProvider.RequestAccessToken(
            new AccessTokenRequestOptions
            {
                ReturnUrl = resumeUri
            });

        if (tokenResult.TryGetToken(out var token))
        {
            http.DefaultRequestHeaders.Add("Authorization", 
                $"Bearer {token.Value}");
            await http.PostAsJsonAsync("Save", User);
        }
        else
        {
            await JS.InvokeVoidAsync("sessionStorage.setState", 
                "resumeSavingProfile", User);
            Navigation.NavigateTo(tokenResult.RedirectUrl);
        }
    }
}
```

## <a name="save-app-state-before-an-authentication-operation"></a>認証操作の前にアプリの状態を保存する

認証操作中に、ブラウザーが IP にリダイレクトされる前に、アプリの状態を保存することが必要になる場合があります。 状態コンテナーを使用していて、認証が成功した後に状態を復元する場合には、このようなことが起こる可能性があります。 カスタム認証状態オブジェクトを使用して、アプリ固有の状態、またはその参照を保持し、認証操作が正常に完了した後で、その状態を復元することができます。 このアプローチの例を次に示します。

状態コンテナー クラスは、アプリの状態値を保持するプロパティを使用して、アプリ内に作成されます。 次の例では、コンテナーを使用して、既定のプロジェクト テンプレートの `Counter` コンポーネント (`Pages/Counter.razor`) のカウンター値を維持します。 コンテナーをシリアル化および逆シリアル化するためのメソッドは、<xref:System.Text.Json> に基づいています。

```csharp
using System.Text.Json;

public class StateContainer
{
    public int CounterValue { get; set; }

    public string GetStateForLocalStorage()
    {
        return JsonSerializer.Serialize(this);
    }

    public void SetStateFromLocalStorage(string locallyStoredState)
    {
        var deserializedState = 
            JsonSerializer.Deserialize<StateContainer>(locallyStoredState);

        CounterValue = deserializedState.CounterValue;
    }
}
```

`Counter` コンポーネントでは、状態コンテナーを使用して、コンポーネントの外部に `currentCount` 値を維持します。

```razor
@page "/counter"
@inject StateContainer State

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;

    protected override void OnInitialized()
    {
        if (State.CounterValue > 0)
        {
            currentCount = State.CounterValue;
        }
    }

    private void IncrementCount()
    {
        currentCount++;
        State.CounterValue = currentCount;
    }
}
```

<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticationState> から `ApplicationAuthenticationState` を作成します。 ローカルに格納されている状態の識別子として機能する `Id` プロパティを指定します。

`ApplicationAuthenticationState.cs`:

```csharp
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class ApplicationAuthenticationState : RemoteAuthenticationState
{
    public string Id { get; set; }
}
```

`Authentication` コンポーネント (`Pages/Authentication.razor`) では、`StateContainer` のシリアル化と逆シリアル化の方法である `GetStateForLocalStorage` および `SetStateFromLocalStorage` で、ローカル セッション ストレージを使用してアプリの状態を保存および復元します。

```razor
@page "/authentication/{action}"
@inject IJSRuntime JS
@inject StateContainer State
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication

<RemoteAuthenticatorViewCore Action="@Action"
                             TAuthenticationState="ApplicationAuthenticationState"
                             AuthenticationState="AuthenticationState"
                             OnLogInSucceeded="RestoreState"
                             OnLogOutSucceeded="RestoreState" />

@code {
    [Parameter]
    public string Action { get; set; }

    public ApplicationAuthenticationState AuthenticationState { get; set; } =
        new ApplicationAuthenticationState();

    protected async override Task OnInitializedAsync()
    {
        if (RemoteAuthenticationActions.IsAction(RemoteAuthenticationActions.LogIn,
            Action) ||
            RemoteAuthenticationActions.IsAction(RemoteAuthenticationActions.LogOut,
            Action))
        {
            AuthenticationState.Id = Guid.NewGuid().ToString();

            await JS.InvokeVoidAsync("sessionStorage.setItem",
                AuthenticationState.Id, State.GetStateForLocalStorage());
        }
    }

    private async Task RestoreState(ApplicationAuthenticationState state)
    {
        if (state.Id != null)
        {
            var locallyStoredState = await JS.InvokeAsync<string>(
                "sessionStorage.getItem", state.Id);

            if (locallyStoredState != null)
            {
                State.SetStateFromLocalStorage(locallyStoredState);
                await JS.InvokeVoidAsync("sessionStorage.removeItem", state.Id);
            }
        }
    }
}
```

この例では、Azure Active Directory (AAD) を使用して認証を行います。 `Program.Main` (`Program.cs`):

* `ApplicationAuthenticationState` は、Microsoft Authentication Library (MSAL) の `RemoteAuthenticationState` 型として、構成が行われます。
* 状態コンテナーがサービス コンテナーに登録されます。

```csharp
builder.Services.AddMsalAuthentication<ApplicationAuthenticationState>(options =>
{
    builder.Configuration.Bind("AzureAd", options.ProviderOptions.Authentication);
});

builder.Services.AddSingleton<StateContainer>();
```

## <a name="customize-app-routes"></a>アプリ ルートをカスタマイズする

[`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) ライブラリでは既定で、次の表に示すルートを使用して、さまざまな認証状態が表されます。

| ルート                            | 目的 |
| -------------------------------- | ------- |
| `authentication/login`           | サインイン操作をトリガーします。 |
| `authentication/login-callback`  | サインイン操作の結果を処理します。 |
| `authentication/login-failed`    | 何らかの理由でサインイン操作が失敗した場合に、エラー メッセージを表示します。 |
| `authentication/logout`          | サインアウト操作をトリガーします。 |
| `authentication/logout-callback` | サインアウト操作の結果を処理します。 |
| `authentication/logout-failed`   | 何らかの理由でサインアウト操作が失敗した場合に、エラー メッセージを表示します。 |
| `authentication/logged-out`      | ユーザーが正常にログアウトしたことを示します。 |
| `authentication/profile`         | ユーザー プロファイルを編集する操作をトリガーします。 |
| `authentication/register`        | 新しいユーザーを登録する操作をトリガーします。 |

上の表に示すルートは、<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticationOptions%601.AuthenticationPaths%2A?displayProperty=nameWithType> を使用して構成できます。 カスタム ルートを提供するオプションを設定する場合は、アプリに各パスを処理するルートがあることを確認します。

次の例では、すべてのパスが `/security` で始まります。

`Authentication` コンポーネント (`Pages/Authentication.razor`):

```razor
@page "/security/{action}"
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication

<RemoteAuthenticatorView Action="@Action" />

@code{
    [Parameter]
    public string Action { get; set; }
}
```

`Program.Main` (`Program.cs`):

```csharp
builder.Services.AddApiAuthorization(options => { 
    options.AuthenticationPaths.LogInPath = "security/login";
    options.AuthenticationPaths.LogInCallbackPath = "security/login-callback";
    options.AuthenticationPaths.LogInFailedPath = "security/login-failed";
    options.AuthenticationPaths.LogOutPath = "security/logout";
    options.AuthenticationPaths.LogOutCallbackPath = "security/logout-callback";
    options.AuthenticationPaths.LogOutFailedPath = "security/logout-failed";
    options.AuthenticationPaths.LogOutSucceededPath = "security/logged-out";
    options.AuthenticationPaths.ProfilePath = "security/profile";
    options.AuthenticationPaths.RegisterPath = "security/register";
});
```

完全に異なるパスを必要とする場合は、前述のようにルートを設定し、明示的なアクション パラメーターを使用して <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView> をレンダリングします。

```razor
@page "/register"

<RemoteAuthenticatorView Action="@RemoteAuthenticationActions.Register" />
```

UI を別のページに分割することもできます。

## <a name="customize-the-authentication-user-interface"></a>認証ユーザー インターフェイスをカスタマイズする

<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView> には、各認証状態の UI 部分の既定のセットが含まれます。 各状態は、カスタム <xref:Microsoft.AspNetCore.Components.RenderFragment> を渡すことでカスタマイズできます。 最初のログイン プロセス中に表示されるテキストをカスタマイズするには、次のように <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView> を変更します。

`Authentication` コンポーネント (`Pages/Authentication.razor`):

```razor
@page "/security/{action}"
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication

<RemoteAuthenticatorView Action="@Action">
    <LoggingIn>
        You are about to be redirected to https://login.microsoftonline.com.
    </LoggingIn>
</RemoteAuthenticatorView>

@code{
    [Parameter]
    public string Action { get; set; }
}
```

<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView> には、次の表に示す認証ルートごとに使用できるフラグメントが 1 つあります。

| ルート                            | Fragment                |
| -------------------------------- | ----------------------- |
| `authentication/login`           | `<LoggingIn>`           |
| `authentication/login-callback`  | `<CompletingLoggingIn>` |
| `authentication/login-failed`    | `<LogInFailed>`         |
| `authentication/logout`          | `<LogOut>`              |
| `authentication/logout-callback` | `<CompletingLogOut>`    |
| `authentication/logout-failed`   | `<LogOutFailed>`        |
| `authentication/logged-out`      | `<LogOutSucceeded>`     |
| `authentication/profile`         | `<UserProfile>`         |
| `authentication/register`        | `<Registering>`         |

## <a name="customize-the-user"></a>ユーザーをカスタマイズする

アプリにバインドされているユーザーをカスタマイズできます。

### <a name="customize-the-user-with-a-payload-claim"></a>ペイロード要求を使用してユーザーをカスタマイズする

次の例では、アプリの認証されたユーザーが、ユーザーの認証方法ごとに `amr` 要求を受け取ります。 この `amr` 要求は、Microsoft Identity プラットフォーム v1.0 [ペイロード要求](/azure/active-directory/develop/access-tokens#the-amr-claim)でトークンのサブジェクトが認証された方法を示しています。 この例では、<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> に基づくカスタム ユーザー アカウント クラスを使用します。

<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> クラスを拡張するクラスを作成します。 次の例では、`AuthenticationMethod` プロパティを `amr` JSON プロパティ値のユーザー配列に設定します。 ユーザーが認証されると、フレームワークによって `AuthenticationMethod` が自動的に設定されます。

```csharp
using System.Text.Json.Serialization;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class CustomUserAccount : RemoteUserAccount
{
    [JsonPropertyName("amr")]
    public string[] AuthenticationMethod { get; set; }
}
```

<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccountClaimsPrincipalFactory%601> を拡張するファクトリを作成して、`CustomUserAccount.AuthenticationMethod` に格納されているユーザーの認証方法から要求を作成します。

```csharp
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication.Internal;

public class CustomAccountFactory 
    : AccountClaimsPrincipalFactory<CustomUserAccount>
{
    public CustomAccountFactory(NavigationManager navigationManager, 
        IAccessTokenProviderAccessor accessor) : base(accessor)
    {
    }
  
    public async override ValueTask<ClaimsPrincipal> CreateUserAsync(
        CustomUserAccount account, RemoteAuthenticationUserOptions options)
    {
        var initialUser = await base.CreateUserAsync(account, options);

        if (initialUser.Identity.IsAuthenticated)
        {
            foreach (var value in account.AuthenticationMethod)
            {
                ((ClaimsIdentity)initialUser.Identity)
                    .AddClaim(new Claim("amr", value));
            }
        }

        return initialUser;
    }
}
```

使用中の認証プロバイダーの `CustomAccountFactory` を登録します。 次の登録のいずれかが有効です。

* <xref:Microsoft.Extensions.DependencyInjection.WebAssemblyAuthenticationServiceCollectionExtensions.AddOidcAuthentication%2A>:

  ```csharp
  using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

  ...

  builder.Services.AddOidcAuthentication<RemoteAuthenticationState, 
      CustomUserAccount>(options =>
      {
          ...
      })
      .AddAccountClaimsPrincipalFactory<RemoteAuthenticationState, 
          CustomUserAccount, CustomAccountFactory>();
  ```

* <xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A>:

  ```csharp
  using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

  ...

  builder.Services.AddMsalAuthentication<RemoteAuthenticationState, 
      CustomUserAccount>(options =>
      {
          ...
      })
      .AddAccountClaimsPrincipalFactory<RemoteAuthenticationState, 
          CustomUserAccount, CustomAccountFactory>();
  ```
  
* <xref:Microsoft.Extensions.DependencyInjection.WebAssemblyAuthenticationServiceCollectionExtensions.AddApiAuthorization%2A>:

  ```csharp
  using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

  ...

  builder.Services.AddApiAuthorization<RemoteAuthenticationState, 
      CustomUserAccount>(options =>
      {
          ...
      })
      .AddAccountClaimsPrincipalFactory<RemoteAuthenticationState, 
          CustomUserAccount, CustomAccountFactory>();
  ```

### <a name="aad-security-groups-and-roles-with-a-custom-user-account-class"></a>カスタム ユーザー アカウント クラスを持つ AAD セキュリティ グループおよびロール

AAD セキュリティ グループと AAD 管理者ロール、およびカスタム ユーザー アカウント クラスを使用するその他の例については、<xref:blazor/security/webassembly/aad-groups-roles> を参照してください。

## <a name="support-prerendering-with-authentication"></a>認証を使用したプリレンダリングのサポート

ホストされている Blazor WebAssembly アプリのトピックのいずれかのガイダンスを実行した後は、この後の手順に従って次のようなアプリを作成できます。

* 承認が不要なパスをプリレンダリングする。
* 承認が必要なパスをプリレンダリングしない。

*`Client`* アプリの `Program` クラス (`Program.cs`) で、共通のサービスの登録を別のメソッド (たとえば、`ConfigureCommonServices`) に組み入れます。

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var builder = WebAssemblyHostBuilder.CreateDefault(args);
        builder.RootComponents.Add...;

        builder.Services.AddScoped( ... );

        services.Add...;

        ConfigureCommonServices(builder.Services);

        await builder.Build().RunAsync();
    }

    public static void ConfigureCommonServices(IServiceCollection services)
    {
        // Common service registrations
    }
}
```

サーバー アプリの `Startup.ConfigureServices` で、次の追加サービスを登録します。

```csharp
using Microsoft.AspNetCore.Components.Authorization;
using Microsoft.AspNetCore.Components.Server;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public void ConfigureServices(IServiceCollection services)
{
    ...

    services.AddRazorPages();
    services.AddScoped<AuthenticationStateProvider, 
        ServerAuthenticationStateProvider>();
    services.AddScoped<SignOutSessionStateManager>();

    Client.Program.ConfigureCommonServices(services);
}
```

サーバー アプリの `Startup.Configure` メソッドで、[`endpoints.MapFallbackToFile("index.html")`](xref:Microsoft.AspNetCore.Builder.StaticFilesEndpointRouteBuilderExtensions.MapFallbackToFile%2A) を [`endpoints.MapFallbackToPage("/_Host")`](xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapFallbackToPage%2A) に置き換えます。

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers();
    endpoints.MapFallbackToPage("/_Host");
});
```

サーバー アプリで、`Pages` フォルダーが存在しない場合は作成します。 サーバー アプリの `Pages` フォルダー内に `_Host.cshtml` ページを作成します。 *`Client`* アプリの `wwwroot/index.html` ファイルの内容を `Pages/_Host.cshtml` ファイルに貼り付けます。 ファイルの内容を更新します。

::: moniker range=">= aspnetcore-5.0"

* ファイルの先頭に、`@page "_Host"` を追加します。
* `<div id="app">Loading...</div>` タグを次のように置き換えます。

  ```cshtml
  <div id="app">
      @if (!HttpContext.Request.Path.StartsWithSegments("/authentication"))
      {
          <component type="typeof({CLIENT APP ASSEMBLY NAME}.App)" 
              render-mode="Static" />
      }
      else
      {
          <text>Loading...</text>
      }
  </div>
  ```
  
  前の例では、プレースホルダー `{CLIENT APP ASSEMBLY NAME}` はクライアント アプリのアセンブリ名です (たとえば `BlazorSample.Client`)。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* ファイルの先頭に、`@page "_Host"` を追加します。
* `<app>Loading...</app>` タグを次のように置き換えます。

  ```cshtml
  <app>
      @if (!HttpContext.Request.Path.StartsWithSegments("/authentication"))
      {
          <component type="typeof({CLIENT APP ASSEMBLY NAME}.App)" 
              render-mode="Static" />
      }
      else
      {
          <text>Loading...</text>
      }
  </app>
  ```
  
  前の例では、プレースホルダー `{CLIENT APP ASSEMBLY NAME}` はクライアント アプリのアセンブリ名です (たとえば `BlazorSample.Client`)。

::: moniker-end
  
## <a name="options-for-hosted-apps-and-third-party-login-providers"></a>ホストされているアプリおよびサードパーティ ログイン プロバイダーに関するオプション

ホストされている Blazor WebAssembly アプリをサードパーティ プロバイダーで認証および承認する場合、ユーザーの認証にはいくつかのオプションを使用できます。 どれを選択するかは、シナリオによって異なります。

詳細については、「<xref:security/authentication/social/additional-claims>」を参照してください。

### <a name="authenticate-users-to-only-call-protected-third-party-apis"></a>ユーザーを認証して保護されたサードパーティ API のみを呼び出す

サードパーティ API プロバイダーに対してクライアント側の OAuth フローを使用してユーザーを認証します。

 ```csharp
 builder.services.AddOidcAuthentication(options => { ... });
 ```
 
 このシナリオでは:

* アプリをホストしているサーバーは関与しません。
* サーバー上の API を保護することはできません。
* アプリでは、保護されたサードパーティ API のみを呼び出すことができます。

### <a name="authenticate-users-with-a-third-party-provider-and-call-protected-apis-on-the-host-server-and-the-third-party"></a>サードパーティ プロバイダーでユーザーを認証し、ホスト サーバーおよびサード パーティ上で保護された API を呼び出す

サードパーティのログイン プロバイダーを使用して、Identity の構成を行います。 サードパーティ API へのアクセスに必要なトークンを取得し、それを格納します。

ユーザーがログインすると、認証プロセスの一環として、アクセス トークンと更新トークンが IdentityID によって収集されます。 その時点で、サードパーティ API の API 呼び出しを行うために使用できる方法はいくつかあります。

#### <a name="use-a-server-access-token-to-retrieve-the-third-party-access-token"></a>サーバー アクセス トークンを使用してサードパーティのアクセス トークンを取得する

サーバー上で生成されたアクセス トークンを使用して、サーバー API エンドポイントからサードパーティのアクセストークンを取得します。 そこから、サードパーティのアクセス トークンを使用して、クライアント上の IdentityID からサードパーティ API リソースを直接呼び出します。

この方法はお勧めしません。 この方法では、サードパーティのアクセス トークンをパブリック クライアント用に生成されたものとして扱う必要があります。 OAuth 規約では、パブリック アプリにはクライアント シークレットがありません。これはシークレットを安全に格納することが信頼できないためです。アクセス トークンは機密クライアントに対して生成されます。 機密クライアントとは、クライアント シークレットを持っていてシークレットを安全に格納できると想定されるクライアントです。

* サードパーティのアクセス トークンには、サードパーティがより信頼できるクライアントのトークンを生成したという事実に基づいて機密性の高い操作を実行するための追加のスコープが付与される場合があります。
* 同様に、信頼されていないクライアントに更新トークンを発行してはなりません。それを行ってしまうと、他の制限が適用されない限り、クライアントは無制限にアクセスできます。

#### <a name="make-api-calls-from-the-client-to-the-server-api-in-order-to-call-third-party-apis"></a>サードパーティ API を呼び出すために、クライアントからサーバー API への API 呼び出しを行う

クライアントからサーバー API への API 呼び出しを行います。 サーバーから、サードパーティ API リソースのアクセス トークンを取得し、必要な呼び出しはすべて発行します。

この方法では、サードパーティ API を呼び出すためにサーバー経由で追加のネットワーク ホップが必要になりますが、それによって最終的にはより安全なエクスペリエンスが得られます。

* サーバーでは、更新トークンを格納し、アプリからサードパーティ リソースへのアクセスが決して失われないようにすることができます。
* アプリでは、より機密性の高いアクセス許可を含む可能性のあるサーバーからのアクセス トークンをリークさせることはできません。

## <a name="use-openid-connect-oidc-v20-endpoints"></a>OpenID Connect (OIDC) v2.0 エンドポイントを使用する

認証ライブラリと Blazor プロジェクト テンプレートでは、Open ID Connect (OIDC) v1.0 エンドポイントが使用されます。 v2.0 エンドポイントを使用するには、JWT ベアラー <xref:Microsoft.AspNetCore.Builder.JwtBearerOptions.Authority?displayProperty=nameWithType> オプションの構成を行います。 次の例では、<xref:Microsoft.AspNetCore.Builder.JwtBearerOptions.Authority> プロパティに `v2.0` セグメントを追加することで、v2.0 に対して AAD の構成が行われます。

```csharp
builder.Services.Configure<JwtBearerOptions>(
    AzureADDefaults.JwtBearerAuthenticationScheme, 
    options =>
    {
        options.Authority += "/v2.0";
    });
```

または、アプリ設定ファイル (`appsettings.json`) で設定を行うこともできます。

```json
{
  "Local": {
    "Authority": "https://login.microsoftonline.com/common/oauth2/v2.0/",
    ...
  }
}
```

証明機関へのセグメントを追跡することがアプリの OIDC プロバイダー (AAD 以外のプロバイダーなど) にとって適切でない場合は、<xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority> プロパティを直接設定します。 <xref:Microsoft.AspNetCore.Builder.JwtBearerOptions> またはアプリ設定ファイル (`appsettings.json`) で `Authority` キーを使用してプロパティを設定します。

ID トークンの要求のリストは、v2.0 エンドポイントで変更されています。 詳細については、「[Microsoft ID プラットフォーム (v2.0) に更新する理由](/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison)」を参照してください。

## <a name="configure-and-use-grpc-in-components"></a>コンポーネントで gRPC を構成し、使用する

[ASP.NET Core gRPC フレームワーク](xref:grpc/index)を使用するように Blazor WebAssembly アプリを構成するには:

* サーバーで gRPC-Web を有効にします。 詳細については、「<xref:grpc/browser>」を参照してください。
* アプリのメッセージ ハンドラーに gRPC サービスを登録します。 次の例では、gRPC チュートリアル[ (`Program.Main`) から ](xref:tutorials/grpc/grpc-start#create-a-grpc-service)`GreeterClient` サービスを使用するように、アプリの認可メッセージ ハンドラーが構成されます。

```csharp
using System.Net.Http;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Grpc.Net.Client;
using Grpc.Net.Client.Web;
using {APP ASSEMBLY}.Shared;

...

builder.Services.AddScoped(sp =>
{
    var baseAddressMessageHandler = 
        sp.GetRequiredService<BaseAddressAuthorizationMessageHandler>();
    baseAddressMessageHandler.InnerHandler = new HttpClientHandler();
    var grpcWebHandler = 
        new GrpcWebHandler(GrpcWebMode.GrpcWeb, baseAddressMessageHandler);
    var channel = GrpcChannel.ForAddress(builder.HostEnvironment.BaseAddress, 
        new GrpcChannelOptions { HttpHandler = grpcWebHandler });

    return new Greeter.GreeterClient(channel);
});
```

プレースホルダー `{APP ASSEMBLY}` は、アプリのアセンブリ名です (例: `BlazorSample`)。 ホストされている Blazor ソリューションの `Shared` プロジェクトに `.proto` ファイルを置きます。

クライアント アプリのコンポーネントでは、gRPC クライアント (`Pages/Grpc.razor`) を使用し、gRPC 呼び出しを行うことができます。

```razor
@page "/grpc"
@attribute [Authorize]
@using Microsoft.AspNetCore.Authorization
@using {APP ASSEMBLY}.Shared
@inject Greeter.GreeterClient GreeterClient

<h1>Invoke gRPC service</h1>

<p>
    <input @bind="name" placeholder="Type your name" />
    <button @onclick="GetGreeting" class="btn btn-primary">Call gRPC service</button>
</p>

Server response: <strong>@serverResponse</strong>

@code {
    private string name = "Bert";
    private string serverResponse;

    private async Task GetGreeting()
    {
        try
        {
            var request = new HelloRequest { Name = name };
            var reply = await GreeterClient.SayHelloAsync(request);
            serverResponse = reply.Message;
        }
        catch (Grpc.Core.RpcException ex)
            when (ex.Status.DebugException is 
                AccessTokenNotAvailableException tokenEx)
        {
            tokenEx.Redirect();
        }
    }
}
```

プレースホルダー `{APP ASSEMBLY}` は、アプリのアセンブリ名です (例: `BlazorSample`)。 `Status.DebugException` プロパティを使用するには、[Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client) バージョン 2.30.0 以降を使用します。

詳細については、「<xref:grpc/browser>」を参照してください。

## <a name="build-a-custom-version-of-the-authenticationmsal-javascript-library"></a>Authentication.MSAL JavaScript ライブラリのカスタム バージョンをビルドする

アプリでカスタム バージョンの [JavaScript 用 Microsoft Authentication Library (MSAL js)](https://www.npmjs.com/package/@azure/msal-browser) が必要な場合は、次の手順を実行します。

1. システムに最新の開発者用 .NET SDK がインストールされていることを確認するか、次から最新の開発者用 SDK を入手してインストールします: [.NET Core SDK:「インストーラーとバイナリ」](https://github.com/dotnet/installer#installers-and-binaries)。 このシナリオでは、内部 NuGet フィードの構成は必要ありません。
1. 「[ソースから ASP.NET Core をビルドする](https://github.com/dotnet/aspnetcore/blob/main/docs/BuildFromSource.md)」のドキュメントに従って、開発用の `dotnet/aspnetcore` GitHub リポジトリを設定します。 [dotnet/aspnetcore GitHub リポジトリ](https://github.com/dotnet/aspnetcore)をフォークしてクローンするか、ZIP アーカイブをダウンロードします。
1. `src/Components/WebAssembly/Authentication.Msal/src/Interop/package.json` ファイルを開き、必要なバージョンの `@azure/msal-browser` を設定します。 リリース済みバージョンの一覧については、[`@azure/msal-browser` の npm Web サイト](https://www.npmjs.com/package/@azure/msal-browser)にアクセスし、 **[Versions]\(バージョン\)** タブを選択してください。
1. コマンド シェルで `yarn build` コマンドを使用して、`src/Components/WebAssembly/Authentication.Msal/src` フォルダー内に `Authentication.Msal` プロジェクトをビルドします。
1. アプリで[圧縮された資産 (Brotli/Gzip)](xref:blazor/host-and-deploy/webassembly#compression) を使用する場合は、`Interop/dist/Release/AuthenticationService.js` ファイルを圧縮します。
1. `AuthenticationService.js` ファイルと、生成した場合はファイルの圧縮バージョン (`.br`/`.gz`) を、`Interop/dist/Release` フォルダーから、アプリの発行済み資産にあるアプリの `publish/wwwroot/_content/Microsoft.Authentication.WebAssembly.Msal` フォルダーにコピーします。

## <a name="additional-resources"></a>その他の技術情報

* <xref:blazor/security/webassembly/graph-api>
* [`HttpClient` および `HttpRequestMessage` と Fetch API 要求オプション](xref:blazor/call-web-api#httpclient-and-httprequestmessage-with-fetch-api-request-options)
