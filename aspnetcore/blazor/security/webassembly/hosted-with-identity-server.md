---
title: ホストされている ASP.NET Core Blazor WebAssembly アプリを Identity Server でセキュリティ保護する
author: guardrex
description: ホストされている ASP.NET Core Blazor WebAssembly アプリを Identity Server でセキュリティ保護する方法について説明します。
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
uid: blazor/security/webassembly/hosted-with-identity-server
ms.openlocfilehash: c74711c10fe399718600f879c3d9151bfb1abd42
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100281008"
---
# <a name="secure-an-aspnet-core-blazor-webassembly-hosted-app-with-identity-server"></a>ASP.NET Core Blazor WebAssembly でホストされているアプリを Identity Server でセキュリティ保護する

この記事では、ユーザーと API 呼び出しの認証に [IdentityServer](https://identityserver.io/) を使用する、[ホステッド Blazor WebAssembly アプリ](xref:blazor/hosting-models#blazor-webassembly)を作成する方法について説明します。

> [!NOTE]
> 既存の外部 Identity サーバー インスタンスを使用するように、スタンドアロンの、またはホストされた Blazor WebAssembly アプリを構成するには、<xref:blazor/security/webassembly/standalone-with-authentication-library> のガイダンスに従ってください。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

認証メカニズムを使用して新しい Blazor WebAssembly プロジェクトを作成するには:

1. **[新しい ASP.NET Core Web アプリケーションを作成する]** ダイアログで **[Blazor WebAssembly アプリ]** テンプレートを選択した後、 **[認証]** の下の **[変更]** を選択します。

1. ASP.NET Core の [Identity](xref:security/authentication/identity) システムを使用してアプリ内にユーザーを格納するには、 **[アプリ内のストア ユーザー アカウント]** オプションを使用して、 **[個人のユーザー アカウント]** を選択します。

1. **[詳細設定]** セクションで、 **[ASP.NET Core hosted]\(ASP.NET Core ホステッド\)** チェック ボックスをオンにします。

# <a name="visual-studio-code--net-core-cli"></a>[Visual Studio Code / .NET Core CLI](#tab/visual-studio-code+netcore-cli)

空のフォルダーに認証メカニズムを使用して新しい Blazor WebAssembly プロジェクトを作成するには、`-au|--auth` オプションを使用して `Individual` 認証メカニズムを指定し、ASP.NET Core の [Identity](xref:security/authentication/identity) システムを使用してアプリ内にユーザーを格納します。

```dotnetcli
dotnet new blazorwasm -au Individual -ho -o {APP NAME}
```

| プレースホルダー  | 例        |
| ------------ | -------------- |
| `{APP NAME}` | `BlazorSample` |

`-o|--output` オプションで指定した出力場所にプロジェクト フォルダーが存在しない場合は作成されて、アプリの名前の一部になります。

詳細については、.NET Core ガイドの [`dotnet new`](/dotnet/core/tools/dotnet-new) コマンドを参照してください。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

認証メカニズムを使用して新しい Blazor WebAssembly プロジェクトを作成するには:

1. **[新しい Blazor WebAssembly アプリの構成]** ステップで、 **[認証]** ドロップダウンから **[Individual Authentication (in-app)]\(個別認証 (アプリ内)\)** を選択します。

1. ASP.NET Core の [Identity](xref:security/authentication/identity) を使用してアプリに格納されている個々のユーザーに対して、アプリが作成されます。

1. **[ASP.NET Core hosted]\(ASP.NET Core ホステッド\)** チェック ボックスをオンにします。

---

## <a name="server-app-configuration"></a>*`Server`* アプリの構成

次のセクションでは、認証のサポートが含まれる場合のプロジェクトへの追加について説明します。

### <a name="startup-class"></a>スタートアップ クラス

`Startup` クラスには次のものが追加されます。

* `Startup.ConfigureServices`の場合:

  * ASP.NET Core Identity:

    ```csharp
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlite(
            Configuration.GetConnectionString("DefaultConnection")));

    services.AddDefaultIdentity<ApplicationUser>(options => 
            options.SignIn.RequireConfirmedAccount = true)
        .AddEntityFrameworkStores<ApplicationDbContext>();
    ```

  * IdentityServer 上に既定の ASP.NET Core 規則を設定する <xref:Microsoft.Extensions.DependencyInjection.IdentityServerBuilderConfigurationExtensions.AddApiAuthorization%2A> ヘルパー メソッドが追加された IdentityServer:

    ```csharp
    services.AddIdentityServer()
        .AddApiAuthorization<ApplicationUser, ApplicationDbContext>();
    ```

  * IdentityServer によって生成された JWT トークンを検証するようにアプリを構成する <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> ヘルパー メソッドが追加された Authentication:

    ```csharp
    services.AddAuthentication()
        .AddIdentityServerJwt();
    ```

* `Startup.Configure`の場合:

  * IdentityServer のミドルウェアによって、OpenID Connect (OIDC) のエンドポイントが公開されます。

    ```csharp
    app.UseIdentityServer();
    ```

  * 認証ミドルウェアでは、要求の資格情報の検証と、要求コンテキストでのユーザーの設定が行われます。

    ```csharp
    app.UseAuthentication();
    ```

  * 承認ミドルウェアにより、承認機能が有効になります。

    ```csharp
    app.UseAuthorization();
    ```

### <a name="azure-app-service-on-linux"></a>Azure App Service on Linux

Azure App Service on Linux にデプロイするときに、発行者を明示的に指定します。 詳細については、「<xref:security/authentication/identity/spa#azure-app-service-on-linux>」を参照してください。

### <a name="addapiauthorization"></a>AddApiAuthorization

<xref:Microsoft.Extensions.DependencyInjection.IdentityServerBuilderConfigurationExtensions.AddApiAuthorization%2A> ヘルパー メソッドでは、ASP.NET Core シナリオ対応に [IdentityServer](https://identityserver.io/) が構成されます。 IdentityServer は、アプリのセキュリティの問題を処理するための強力で拡張可能なフレームワークです。 IdentityServer を使用すると、ほとんどの一般的なシナリオには必要のない複雑さが発生します。 そのため、使用開始時に適切であると考えられる一連の規則と構成オプションが用意されています。 認証のニーズが変わったら、IdentityServer のあらゆる機能を利用し、アプリの要件に合わせて認証をカスタマイズできます。

### <a name="addidentityserverjwt"></a>AddIdentityServerJwt

<xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> ヘルパー メソッドでは、アプリに対するポリシー スキームが既定の認証ハンドラーとして構成されます。 そのポリシーは、Identity の URL 空間 `/Identity` のサブパスにルーティングされたすべての要求を Identity で処理できるように構成されています。 それ以外のすべての要求は、<xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerHandler> で処理されます。 さらに、このメソッドでは次のことが行われます。

* 既定のスコープ `{APPLICATION NAME}API` を使用して、`{APPLICATION NAME}API` API リソースが IdentityServer に登録されます。
* IdentityServer によってアプリに対して発行されたトークンを検証するように、JWT ベアラー トークン ミドルウェアが構成されます。

### <a name="weatherforecastcontroller"></a>WeatherForecastController

`WeatherForecastController` (`Controllers/WeatherForecastController.cs`) では、[`[Authorize]` 属性](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute)がクラスに適用されます。 その属性では、ユーザーはリソースにアクセスするために既定のポリシーに基づいて承認される必要があることが示されています。 既定の承認ポリシーは、<xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> によって設定される既定の認証スキームを使用するように構成されています。 ヘルパー メソッドでは、アプリへの要求に対する既定のハンドラーとして <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerHandler> が構成されます。

### <a name="applicationdbcontext"></a>ApplicationDbContext

`ApplicationDbContext` (`Data/ApplicationDbContext.cs`) では、<xref:Microsoft.EntityFrameworkCore.DbContext> により、IdentityServer 用のスキーマを含むように <xref:Microsoft.AspNetCore.ApiAuthorization.IdentityServer.ApiAuthorizationDbContext%601> が拡張されます。 <xref:Microsoft.AspNetCore.ApiAuthorization.IdentityServer.ApiAuthorizationDbContext%601> は、<xref:Microsoft.AspNetCore.Identity.EntityFrameworkCore.IdentityDbContext> から派生しています。

データベース スキーマを完全に制御するには、使用可能な Identity <xref:Microsoft.EntityFrameworkCore.DbContext> クラスの 1 つを継承し、<xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A> メソッドで `builder.ConfigurePersistedGrantContext(_operationalStoreOptions.Value)` を呼び出すことによって、Identity スキーマを含むようにコンテキストを構成します。

### <a name="oidcconfigurationcontroller"></a>OidcConfigurationController

`OidcConfigurationController` (`Controllers/OidcConfigurationController.cs`) では、OIDC パラメーターを提供するために、クライアント エンドポイントがプロビジョニングされます。

### <a name="app-settings"></a>アプリの設定

プロジェクト ルートにあるアプリ設定ファイル (`appsettings.json`) の `IdentityServer` セクションには、構成されているクライアントの一覧が記述されてます。 次の例には、1 つのクライアントがあります。 クライアント名はアプリケーション名に対応し、規則によって OAuth の `ClientId` パラメーターにマップされます。 構成対象のアプリの種類は、プロファイルによって示されています。 プロファイルは、サーバーの構成プロセスを簡素化する規則を促進するために、内部的に使用されます。 <!-- There are several profiles available, as explained in the [Application profiles](#application-profiles) section. -->

```json
"IdentityServer": {
  "Clients": {
    "{APP ASSEMBLY}.Client": {
      "Profile": "IdentityServerSPA"
    }
  }
}
```

プレースホルダー `{APP ASSEMBLY}` は、アプリのアセンブリ名です (例: `BlazorSample.Client`)。

## <a name="client-app-configuration"></a>*`Client`* アプリの構成

### <a name="authentication-package"></a>認証パッケージ

個人のユーザー アカウント (`Individual`) を使用するようにアプリを作成すると、アプリはそのアプリのプロジェクト ファイル内で [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) パッケージのパッケージ参照を自動的に受け取ります。 このパッケージには、アプリでユーザーを認証し、保護された API を呼び出すためのトークンを取得するのに役立つ一連のプリミティブが用意されています。

アプリに認証を追加する場合は、アプリのプロジェクト ファイルにパッケージを手動で追加します。

```xml
<PackageReference 
  Include="Microsoft.AspNetCore.Components.WebAssembly.Authentication" 
  Version="{VERSION}" />
```

プレースホルダー `{VERSION}` では、[NuGet.org](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) のパッケージの **バージョン履歴** にある、アプリの共有フレームワークのバージョンに一致するパッケージの安定した最新バージョンを確認できます。

### <a name="httpclient-configuration"></a>`HttpClient` 構成

`Program.Main` (`Program.cs`) では、サーバー API への要求を行うときのアクセス トークンが含まれる <xref:System.Net.Http.HttpClient> インスタンスを提供するように、名前付きの <xref:System.Net.Http.HttpClient> (`HostIS.ServerAPI`) が構成されます。

```csharp
builder.Services.AddHttpClient("HostIS.ServerAPI", 
        client => client.BaseAddress = new Uri(builder.HostEnvironment.BaseAddress))
    .AddHttpMessageHandler<BaseAddressAuthorizationMessageHandler>();

builder.Services.AddScoped(sp => sp.GetRequiredService<IHttpClientFactory>()
    .CreateClient("HostIS.ServerAPI"));
```

> [!NOTE]
> ホストされている Blazorソリューションの一部ではない既存の Identity Server インスタンスを使用するように Blazor WebAssembly アプリを構成する場合は、<xref:System.Net.Http.HttpClient> ベース アドレスの登録を <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> (`builder.HostEnvironment.BaseAddress`) からサーバー アプリの API 認証エンドポイント URL に変更します。

### <a name="api-authorization-support"></a>API の承認のサポート

ユーザーの認証に対するサポートは、[`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) パッケージ内で提供される拡張メソッドによって、サービス コンテナーに接続されます。 このメソッドでは、アプリが既存の承認システムとやり取りするために必要なサービスが設定されます。

```csharp
builder.Services.AddApiAuthorization();
```

既定では、アプリの構成は規則により `_configuration/{client-id}` から読み込まれます。 規則により、アプリのアセンブリ名にはクライアント ID が設定されます。 オプションを指定してオーバーロードを呼び出すことにより、別のエンドポイントを指すようにこの URL を変更できます。

### <a name="imports-file"></a>インポート ファイル

[!INCLUDE[](~/blazor/includes/security/imports-file-hosted.md)]

### <a name="index-page"></a>インデックス ページ

[!INCLUDE[](~/blazor/includes/security/index-page-authentication.md)]

### <a name="app-component"></a>アプリ コンポーネント

[!INCLUDE[](~/blazor/includes/security/app-component.md)]

### <a name="redirecttologin-component"></a>RedirectToLogin コンポーネント

[!INCLUDE[](~/blazor/includes/security/redirecttologin-component.md)]

### <a name="logindisplay-component"></a>LoginDisplay コンポーネント

`LoginDisplay` コンポーネント (`Shared/LoginDisplay.razor`) は `MainLayout` コンポーネント (`Shared/MainLayout.razor`) でレンダリングされます。このコンポーネントによって次の動作が管理されます。

* 認証されたユーザーの場合:
  * 現在のユーザー名が表示されます。
  * ASP.NET Core Identity のユーザー プロファイル ページへのリンクが提供されます。
  * アプリからログアウトするためのボタンが用意されます。
* 匿名ユーザーの場合:
  * 登録するオプションが提供されます。
  * ログインするオプションが提供されます。

```razor
@using Microsoft.AspNetCore.Components.Authorization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@inject NavigationManager Navigation
@inject SignOutSessionStateManager SignOutManager

<AuthorizeView>
    <Authorized>
        <a href="authentication/profile">Hello, @context.User.Identity.Name!</a>
        <button class="nav-link btn btn-link" @onclick="BeginSignOut">
            Log out
        </button>
    </Authorized>
    <NotAuthorized>
        <a href="authentication/register">Register</a>
        <a href="authentication/login">Log in</a>
    </NotAuthorized>
</AuthorizeView>

@code {
    private async Task BeginSignOut(MouseEventArgs args)
    {
        await SignOutManager.SetSignOutState();
        Navigation.NavigateTo("authentication/logout");
    }
}
```

### <a name="authentication-component"></a>認証コンポーネント

[!INCLUDE[](~/blazor/includes/security/authentication-component.md)]

### <a name="fetchdata-component"></a>FetchData コンポーネント

[!INCLUDE[](~/blazor/includes/security/fetchdata-component.md)]

## <a name="run-the-app"></a>アプリを実行する

サーバー プロジェクトからアプリを実行します。 Visual Studio を使用しているときは、次のいずれかを行います。

* ツール バーの **[スタートアップ プロジェクト]** ドロップダウン リストを "*サーバー API アプリ*" に設定して、 **[実行]** ボタンを選択します。
* **ソリューション エクスプローラー** でサーバー プロジェクトを選択し、ツール バーの **[実行]** ボタンを選択するか、 **[デバッグ]** メニューからアプリを開始します。

## <a name="name-and-role-claim-with-api-authorization"></a>API 認証での名前とロール要求

### <a name="custom-user-factory"></a>カスタム ユーザー ファクトリ

*`Client`* アプリで、カスタム ユーザー ファクトリを作成します。 Identity Server により、複数のロールが JSON 配列として 1 つの `role` 要求で送信されます。 1 つのロールは、要求内で文字列値として送信されます。 ファクトリにより、ユーザーのロールごとに個別の `role` 要求が作成されます。

`CustomUserFactory.cs`:

```csharp
using System.Linq;
using System.Security.Claims;
using System.Text.Json;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication.Internal;

public class CustomUserFactory
    : AccountClaimsPrincipalFactory<RemoteUserAccount>
{
    public CustomUserFactory(IAccessTokenProviderAccessor accessor)
        : base(accessor)
    {
    }

    public async override ValueTask<ClaimsPrincipal> CreateUserAsync(
        RemoteUserAccount account,
        RemoteAuthenticationUserOptions options)
    {
        var user = await base.CreateUserAsync(account, options);

        if (user.Identity.IsAuthenticated)
        {
            var identity = (ClaimsIdentity)user.Identity;
            var roleClaims = identity.FindAll(identity.RoleClaimType).ToArray();

            if (roleClaims != null && roleClaims.Any())
            {
                foreach (var existingClaim in roleClaims)
                {
                    identity.RemoveClaim(existingClaim);
                }

                var rolesElem = account.AdditionalProperties[identity.RoleClaimType];

                if (rolesElem is JsonElement roles)
                {
                    if (roles.ValueKind == JsonValueKind.Array)
                    {
                        foreach (var role in roles.EnumerateArray())
                        {
                            identity.AddClaim(new Claim(options.RoleClaim, role.GetString()));
                        }
                    }
                    else
                    {
                        identity.AddClaim(new Claim(options.RoleClaim, roles.GetString()));
                    }
                }
            }
        }

        return user;
    }
}
```

*`Client`* アプリでは、`Program.Main` (`Program.cs`) にファクトリを登録します。

```csharp
builder.Services.AddApiAuthorization()
    .AddAccountClaimsPrincipalFactory<CustomUserFactory>();
```

*`Server`* アプリでは、Identity ビルダーで <xref:Microsoft.AspNetCore.Identity.IdentityBuilder.AddRoles*> を呼び出します。これにより、ロールに関連するサービスが追加されます。

```csharp
using Microsoft.AspNetCore.Identity;

...

services.AddDefaultIdentity<ApplicationUser>(options => 
    options.SignIn.RequireConfirmedAccount = true)
    .AddRoles<IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>();
```

### <a name="configure-identity-server"></a>Identity Server を構成する

次の方法の **いずれか** を使用します。

* [API 承認のオプション](#api-authorization-options)
* [プロファイル サービス](#profile-service)

#### <a name="api-authorization-options"></a>API 承認のオプション

*`Server`* アプリで:

* `name` 要求と `role` 要求を ID トークンとアクセス トークンに挿入するように、Identity Server を構成します。
* JWT トークン ハンドラーでロールの既定のマッピングを禁止します。

```csharp
using System.IdentityModel.Tokens.Jwt;
using System.Linq;

...

services.AddIdentityServer()
    .AddApiAuthorization<ApplicationUser, ApplicationDbContext>(options => {
        options.IdentityResources["openid"].UserClaims.Add("name");
        options.ApiResources.Single().UserClaims.Add("name");
        options.IdentityResources["openid"].UserClaims.Add("role");
        options.ApiResources.Single().UserClaims.Add("role");
    });

JwtSecurityTokenHandler.DefaultInboundClaimTypeMap.Remove("role");
```

#### <a name="profile-service"></a>プロファイル サービス

*`Server`* アプリで、`ProfileService` の実装を作成します。

`ProfileService.cs`:

```csharp
using IdentityModel;
using IdentityServer4.Models;
using IdentityServer4.Services;
using System.Threading.Tasks;

public class ProfileService : IProfileService
{
    public ProfileService()
    {
    }

    public async Task GetProfileDataAsync(ProfileDataRequestContext context)
    {
        var nameClaim = context.Subject.FindAll(JwtClaimTypes.Name);
        context.IssuedClaims.AddRange(nameClaim);

        var roleClaims = context.Subject.FindAll(JwtClaimTypes.Role);
        context.IssuedClaims.AddRange(roleClaims);

        await Task.CompletedTask;
    }

    public async Task IsActiveAsync(IsActiveContext context)
    {
        await Task.CompletedTask;
    }
}
```

*`Server`* アプリで、`Startup.ConfigureServices` にプロファイル サービスを登録します。

```csharp
using IdentityServer4.Services;

...

services.AddTransient<IProfileService, ProfileService>();
```

### <a name="use-authorization-mechanisms"></a>承認メカニズムを使用する

*`Client`* アプリでは、この時点でコンポーネントの承認方法が機能しています。 コンポーネント内のすべての承認メカニズムで、ロールを使用してユーザーを承認できます。

* [`AuthorizeView` コンポーネント](xref:blazor/security/index#authorizeview-component) (例: `<AuthorizeView Roles="admin">`)
* [`[Authorize]` 属性ディレクティブ](xref:blazor/security/index#authorize-attribute) (<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute>) (例: `@attribute [Authorize(Roles = "admin")]`)
* [手続き型ロジック](xref:blazor/security/index#procedural-logic) (例: `if (user.IsInRole("admin")) { ... }`)

  複数のロール テストがサポートされています。

  ```csharp
  if (user.IsInRole("admin") && user.IsInRole("developer"))
  {
      ...
  }
  ```

`User.Identity.Name` には、 *`Client`* アプリにおいてユーザーのユーザー名が設定されます。これは通常、サインイン メール アドレスです。

[!INCLUDE[](~/blazor/includes/security/usermanager-signinmanager.md)]

## <a name="host-in-azure-app-service-with-a-custom-domain-and-certificate"></a>カスタム ドメインと証明書を使用した Azure App Service でのホスト

以下のガイダンスでは、次について説明します。

* Identity Server を使用してホストされている Blazor WebAssembly アプリを、カスタム ドメインを使用して [Azure App Service](https://azure.microsoft.com/services/app-service/) にデプロイする方法。
* ブラウザーとの HTTPS プロトコル通信に対して、TLS 証明書を作成して使用する方法。 このガイダンスではカスタム ドメインで証明書を使用する方法に焦点を当てていますが、`contoso.azurewebsites.net` などの既定の Azure アプリのドメインを使用する場合にも同様に当てはまります。

このホスティング シナリオの場合、[Identity Server のトークン署名キー](https://docs.identityserver.io/en/latest/topics/crypto.html#token-signing-and-validation)と、サイトの HTTPS でセキュリティ保護されたブラウザーとの通信に、同じ証明書を **使用しないでください**。

* この 2 つの要件に対して異なる証明書を使用することは、それぞれの目的に対して秘密キーが分離されるため、優れたセキュリティ方法です。
* ブラウザーとの通信に使用される TLS 証明書は、Identity Server のトークン署名に影響を与えることなく、個別に管理されます。
* カスタム ドメイン バインドのために [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) で App Service アプリに証明書を提供すると、Identity Server で Azure Key Vault からトークン署名用に同じ証明書を取得することはできません。 物理パスから同じ TLS 証明書を使用するように Identity Server を構成することはできますが、セキュリティ証明書をソース管理に置くことは **不適切な方法であり、ほとんどのシナリオで避ける必要があります**。

次のガイダンスでは、自己署名証明書は Identity Server のトークン署名のためだけに Azure Key Vault に作成されます。 Identity Server の構成により、アプリの `My` > `CurrentUser` 証明書ストアを介して、キー コンテナー証明書が使用されます。 カスタム ドメインで HTTPS トラフィックに使用される他の証明書は、Identity Server の署名証明書とは別に作成および構成されます。

カスタム ドメインと HTTPS を使用してホストするようにアプリ、Azure App Service、Azure Key Vault を構成するには:

1. プラン レベル `Basic B1` 以上を使用して、[App Service プラン](/azure/app-service/overview-hosting-plans)を作成します。 App Service でカスタム ドメインを使用するには、`Basic B1` 以上のサービス レベルが必要です。
1. 組織で管理するサイトの完全修飾ドメイン名 (FQDN) の共通名を使用して (例: `www.contoso.com`)、サイトのセキュリティで保護されたブラウザー通信 (HTTPS プロトコル) 用の PFX 証明書を作成します。 次のように証明書を作成します。
   * キーで使用
     * デジタル署名の検証 (`digitalSignature`)
     * キーの暗号化 (`keyEncipherment`)
   * 強化および拡張されたキーで使用
     * クライアント認証 (1.3.6.1.5.5.7.3.2)
     * サーバー認証 (1.3.6.1.5.5.7.3.1)

   証明書を作成するには、次のいずれかの方法、または他の適切なツールやオンライン サービスを使用します。

   * [Azure Key Vault](/azure/key-vault/certificates/quick-create-portal#add-a-certificate-to-key-vault)
   * [Windows での MakeCert](/windows/desktop/seccrypto/makecert)
   * [OpenSSL](https://www.openssl.org)

   パスワードを記録しておきます。これは後で Azure Key Vault に証明書をインポートするために使用します。

   Azure Key Vault 証明書の詳細については、[Azure Key Vault のCertificate\(次へ: 証明書\)](/azure/key-vault/certificates/) を選択します。
1. 新しい Azure キー コンテナーを作成するか、Azure サブスクリプションの既存のキー コンテナーを使用します。
1. キー コンテナーの **[証明書]** 領域で、PFX サイト証明書をインポートします。 後でアプリの構成に使用するので、証明書のサムプリントを記録しておきます。
1. Azure Key Vault で、Identity Server のトークン署名用に新しい自己署名証明書を生成します。 証明書の **証明書名** と **サブジェクト** を指定します。 **サブジェクト** は `CN={COMMON NAME}` と指定します。`{COMMON NAME}` プレースホルダーは証明書の共通名です。 共通名には、任意の英数字を使用できます。 たとえば、`CN=IdentityServerSigning` は証明書の有効な **サブジェクト** です。 **[ポリシーの詳細構成]** は既定の設定を使用します。 後でアプリの構成に使用するので、証明書のサムプリントを記録しておきます。
1. Azure portal で Azure App Service に移動し、次の構成を使用して新しいアプリ サービスを作成します。
   * **[発行]** は、`Code` に設定します。
   * **[ランタイム スタック]** は、アプリのランタイムに設定します。
   * **[SKU とサイズ]** については、App Service のレベルが `Basic B1` 以上であることを確認します。  App Service でカスタム ドメインを使用するには、`Basic B1` 以上のサービス レベルが必要です。
1. Azure によって App Service が作成された後、アプリの **[構成]** を開き、前に記録した証明書のサムプリントを指定して新しいアプリケーション設定を追加します。 アプリ設定キーは `WEBSITE_LOAD_CERTIFICATES` です。 次の例に示すように、アプリの設定値で証明書のサムプリントを区切るにはコンマを使用します。
   * キー: `WEBSITE_LOAD_CERTIFICATES`
   * 値: `57443A552A46DB...D55E28D412B943565,29F43A772CB6AF...1D04F0C67F85FB0B1`

   Azure portal で、アプリの設定の保存は 2 段階のプロセスです。`WEBSITE_LOAD_CERTIFICATES` のキーと値の設定を保存した後、ブレードの上部にある **[保存]** ボタンを選択します。
1. アプリの **[TLS/SSL の設定]** を選択します。 **[秘密キー証明書 (.pfx)]** を選択します。 **[Key Vault 証明書のインポート]** プロセスを 2 回使用して、HTTPS 通信用のサイトの証明書と、サイトの自己署名された Identity Server トークン署名証明書の両方をインポートします。
1. **[カスタム ドメイン]** ブレードに移動します。 ドメイン レジストラーの Web サイトで、**IP アドレス** と **カスタム ドメイン検証 ID** を使用して、ドメインを構成します。 一般的なドメイン構成には次のものが含まれます。
   * **ホスト** が `@` で、値が Azure portal の IP アドレスである **A レコード**。
   * **ホスト** が `asuid` で、値が Azure によって生成されて Azure portal によって提供される検証 ID である **TXT レコード**。

   ドメイン レジストラーの Web サイトで変更を正しく保存したことを確認します。 レジストラーの Web サイトによっては、ドメイン レコードを保存するために 2 段階のプロセスが必要な場合があります。1 つ以上のレコードを個別に保存した後、別のボタンを使用してドメインの登録を更新します。
1. Azure portal の **[カスタム ドメイン]** ブレードに戻ります。 **[カスタム ドメインの追加]** を選択します。 **[A レコード]** オプションを選択します。 ドメインを指定し、 **[検証]** を選択します。 ドメイン レコードが正しく、インターネットを通して反映されている場合、ポータルで **[カスタム ドメインの追加]** ボタンを選択できます。

   ドメイン登録の変更がインターネットのドメイン ネーム サーバー (DNS) 全体に反映されるまで、ドメイン レジストラーによって処理されてから数日かかることがあります。 ドメイン レコードが 3 営業日以内に更新されない場合は、レコードが正しく設定されていることをドメイン レジストラーで確認し、カスタマー サポートに問い合わせてください。
1. **[カスタム ドメイン]** ブレードで、ドメインの **[SSL 状態]** は `Not Secure` とマークされます。 **[バインドの追加]** リンクを選択します。 キー コンテナーからカスタム ドメイン バインド用のサイトの HTTPS 証明書を選択します。
1. Visual Studio で、*Server* プロジェクトのアプリ設定ファイル (`appsettings.json` または `appsettings.Production.json`) を開きます。 Identity Server の構成で、次の `Key` セクションを追加します。 `Name` キーに対しては、自己署名証明書の **サブジェクト** を指定します。 次の例の場合、キー コンテナーで割り当てられている証明書の共通名は `IdentityServerSigning` であり、それにより `CN=IdentityServerSigning` という **サブジェクト** が生成されます。

   ```json
   "IdentityServer": {

     ...

     "Key": {
       "Type": "Store",
       "StoreName": "My",
       "StoreLocation": "CurrentUser",
       "Name": "CN=IdentityServerSigning"
     }
   },
   ```

1. Visual Studio で、*Server* プロジェクトに対する Azure App Service の "[発行プロファイル](xref:host-and-deploy/visual-studio-publish-profiles#publish-profiles)" を作成します。 メニュー バーから次のように選択します。 **[ビルド]**  >  **[発行]**  >  **[新規]**  >  **[Azure]**  >  **[Azure App Service]** (Windows または Linux)。 Visual Studio が Azure サブスクリプションに接続されたら、 **[リソースの種類]** で Azure リソースの **[ビュー]** を設定できます。 **[Web アプリ]** の一覧内を移動し、アプリの App Service を見つけて選択します。 **[完了]** を選択します。
1. Visual Studio が **[発行]** ウィンドウに戻ると、キー コンテナーと SQL Server データベース サービスの依存関係が自動的に検出されます。

   Key Vault サービスの既定の設定で構成を変更する必要はありません。

   テストの目的で、既定では Blazor テンプレートによって構成されるアプリのローカル [SQLite](https://www.sqlite.org/index.html) データベースを、追加の構成を行わずにアプリでデプロイできます。 運用環境で Identity Server 用に別のデータベースを構成する方法については、この記事では説明しません。 詳細については、次のドキュメント セットのデータベース リソースを参照してください。
   * [App Service](/azure/app-service/)
   * [Identity サーバー](https://identityserver4.readthedocs.io/en/latest/)

1. ウィンドウの上部にある配置プロファイル名の下にある **[編集]** リンクを選択します。 デプロイ先の URL を、サイトのカスタム ドメイン URL に変更します (例: `https://www.contoso.com`)。 設定を保存します。
1. アプリの発行 Visual Studio によってブラウザー ウィンドウが開かれ、カスタム ドメインのサイトが要求されます。

Azure のドキュメントには、App Service の TLS バインドでの Azure サービスとカスタム ドメインの使用に関する詳細が含まれています。これには、A レコードの代わりに CNAME レコードを使用する方法の情報も含まれます。 詳細については、次のリソースを参照してください。

* [App Service のドキュメント](/azure/app-service/)
* [チュートリアル:既存のカスタム DNS 名を Azure App Service にマップする](/azure/app-service/app-service-web-tutorial-custom-domain)
* [Azure App Service で TLS/SSL バインドを使用してカスタム DNS 名をセキュリティで保護する](/azure/app-service/configure-ssl-bindings)
* [Azure Key Vault](/azure/key-vault/)

Azure portal でアプリ、アプリの構成、または Azure サービスを変更した後は、アプリのテストを実行するたびに、新しいプライベートまたはシークレットのブラウザー ウィンドウを使用することをお勧めします。 サイトの構成が正しい場合でも、前回のテスト実行から残っている cookie により、サイトをテストするときに認証または承認が失敗する可能性があります。 テストを実行するたびに新しいプライベートまたはシークレットのブラウザー ウィンドウを開くように Visual Studio を構成する方法の詳細については、「[Cookie とサイト データ](#cookies-and-site-data)」セクションを参照してください。

Azure portal で App Service の構成を変更すると、通常、更新は短時間で有効になりますが、すぐにではありません。 場合によっては、構成の変更を有効にするために App Service が再起動されるまでしばらく待つ必要があります。

証明書の読み込みに関する問題のトラブルシューティングを行う場合は、Azure portal の [Kudu](https://github.com/projectkudu/kudu/wiki/Accessing-the-kudu-service) PowerShell コマンド シェルで次のコマンドを実行します。 このコマンドにより、アプリで `My` > `CurrentUser` 証明書ストアからアクセスできる証明書の一覧が提供されます。 出力には、アプリをデバッグするときに役立つ証明書のサブジェクトとサムプリントが含まれています。

```powershell
Get-ChildItem -path Cert:\CurrentUser\My -Recurse | Format-List DnsNameList, Subject, Thumbprint, EnhancedKeyUsageList
```

[!INCLUDE[](~/blazor/includes/security/troubleshoot.md)]

## <a name="additional-resources"></a>その他の技術情報

* [Azure App Service にデプロイする](xref:security/authentication/identity/spa#deploy-to-production)
* [Key Vault から証明書をインポートする (Azure ドキュメント)](/azure/app-service/configure-ssl-certificate#import-a-certificate-from-key-vault)
* <xref:blazor/security/webassembly/additional-scenarios>
* [セキュリティで保護された既定のクライアントを使用する、アプリ内の認証または承認されていない Web API 要求](xref:blazor/security/webassembly/additional-scenarios#unauthenticated-or-unauthorized-web-api-requests-in-an-app-with-a-secure-default-client)
* <xref:host-and-deploy/proxy-load-balancer>: 次のガイダンスが含まれます。
  * 転送されたヘッダー ミドルウェアを使用した、プロキシ サーバーと内部ネットワークの間での HTTPS スキーム情報の保持。
  * 追加のシナリオとユース ケース (手動スキーム構成を含む)、正しい要求ルーティングのための要求パスの変更、Linux および IIS 以外のリバース プロキシのための要求スキームの転送。
