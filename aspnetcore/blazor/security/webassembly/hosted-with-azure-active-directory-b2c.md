---
title: ASP.NET Core Blazor WebAssembly でホストされるアプリを Azure Active Directory B2C でセキュリティ保護する
author: guardrex
description: ASP.NET Core Blazor WebAssembly でホストされるアプリを Azure Active Directory B2C でセキュリティ保護する方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/26/2020
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
uid: blazor/security/webassembly/hosted-with-azure-active-directory-b2c
ms.openlocfilehash: c67f8e8d81fbdbd8a1f103dd1bd258212efe014f
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280952"
---
# <a name="secure-an-aspnet-core-blazor-webassembly-hosted-app-with-azure-active-directory-b2c"></a>ASP.NET Core Blazor WebAssembly でホストされるアプリを Azure Active Directory B2C でセキュリティ保護する

この記事では、認証用に [Azure Active Directory (AAD) B2C](/azure/active-directory-b2c/overview) を使用する[ホステッド Blazor WebAssembly アプリ](xref:blazor/hosting-models#blazor-webassembly)を作成する方法について説明します。

## <a name="register-apps-in-aad-b2c-and-create-solution"></a>AAD B2C でアプリを登録してソリューションを作成する

### <a name="create-a-tenant"></a>テナントの作成

「[チュートリアル: Azure Active Directory B2C テナントの作成](/azure/active-directory-b2c/tutorial-create-tenant)」のガイダンスに従って、AAD B2C テナントを作成します。 使用するテナントを作成または特定した直後に、この記事に戻ります。

AAD B2C インスタンスを記録しておきます (例: 末尾にスラッシュが含まれている `https://contoso.b2clogin.com/`)。 インスタンスは、Azure B2C アプリの登録のスキームとホストです。これは、Azure portal で **[アプリの登録]** ページから **[エンドポイント]** ウィンドウを開くことによって見つけることができます。

### <a name="register-a-server-api-app"></a>サーバー API アプリを登録する

"*サーバー API アプリ*" 用の AAD B2C アプリを登録します。

1. **[Azure Active Directory]**  >  **[アプリの登録]** で、 **[新規登録]** を選択します。
1. アプリの **[名前]** を指定します (例: **Blazor Server AAD B2C**)。
1. **[サポートされているアカウントの種類]** で、マルチテナント オプションを選択します: **ID プロバイダーまたは組織ディレクトリのアカウント (ユーザー フローでユーザーを認証するため)**
1. "*サーバー API アプリ*" の場合、このシナリオでは **[リダイレクト URI]** は必要ないので、ドロップダウンは **[Web]** に設定されたままにして、リダイレクト URI は入力しません。
1. **[アクセス許可]**  >  **[openid と offline_access アクセス許可に対して管理者の同意を付与します]** が選択されていることを確認します。
1. **[登録]** を選択します。

次の情報を記録しておきます。

* "*サーバー API アプリ*" のアプリケーション (クライアント) ID (例: `41451fa7-82d9-4673-8fa5-69eff5a761fd`)
* AAD プライマリ、パブリッシャー、テナント ドメイン (`contoso.onmicrosoft.com` など):ドメインは、Azure portal で登録されているアプリに対する **[ブランド]** ブレードの **[パブリッシャー ドメイン]** として使用できます。

**[API の公開]** で:

1. **[Scope の追加]** を選択します。
1. **[Save and continue]\(保存して続行\)** を選択します。
1. **[スコープ名]** を指定します (例: `API.Access`)。
1. **[管理者の同意の表示名]** を指定します (例: `Access API`)。
1. **[管理者の同意の説明]** を指定します (例: `Allows the app to access server app API endpoints.`)。
1. **[状態]** が **[有効]** に設定されていることを確認します。
1. **[スコープの追加]** を選択します。

次の情報を記録しておきます。

* アプリ ID の URI (例: `api://41451fa7-82d9-4673-8fa5-69eff5a761fd`、`https://contoso.onmicrosoft.com/41451fa7-82d9-4673-8fa5-69eff5a761fd`、または指定したカスタム値)
* スコープ名 (例: `API.Access`)

### <a name="register-a-client-app"></a>クライアント アプリを登録する

"*クライアント アプリ*" 用の AAD B2C アプリを登録します。

::: moniker range=">= aspnetcore-5.0"

1. **[Azure Active Directory]** > **[アプリの登録]** で、 **[新規登録]** を選択します。
1. アプリの **[名前]** を指定します (例: **Blazor クライアント AAD B2C**)。
1. **[サポートされているアカウントの種類]** で、マルチテナント オプションを選択します: **ID プロバイダーまたは組織ディレクトリのアカウント (ユーザー フローでユーザーを認証するため)**
1. **[リダイレクト URI]** ドロップダウンを **[シングルページ アプリケーション (SPA)]** に設定し、次のリダイレクト URI を指定します: `https://localhost:{PORT}/authentication/login-callback`。 Kestrel で実行されているアプリの既定のポートは 5001 です。 アプリが別の Kestrel ポートで実行されている場合は、アプリのポートを使用します。 IIS Express の場合、アプリのランダムに生成されたポートは、 **[デバッグ]** パネルの *`Server`* アプリのプロパティで確認できます。 この時点ではアプリは存在せず、IIS Express ポートは不明であるため、アプリが作成された後にこの手順に戻り、リダイレクト URI を更新してください。 「[アプリを作成する](#create-the-app)」セクションの解説には、IIS Express ユーザーに対してリダイレクト URI の更新を促す注意が示されています。
1. **[アクセス許可]** > **[openid と offline_access アクセス許可に対して管理者の同意を付与します]** が選択されていることを確認します。
1. **[登録]** を選択します。

1. アプリケーション (クライアント) ID を記録しておきます (例: `4369008b-21fa-427c-abaa-9b53bf58e538`)。

**[認証]** > **[プラットフォーム構成]** > **[シングルページ アプリケーション (SPA)]** で次の操作を行います。

1. **[リダイレクト URI]** が`https://localhost:{PORT}/authentication/login-callback` であることを確認します。
1. **[暗黙の付与]** で、 **[アクセス トークン]** と **[ID トークン]** のチェック ボックスが **オフ** になっていることを確認します。
1. アプリの残りの既定値は、このエクスペリエンスで使用可能です。
1. **[保存]** ボタンを選択します。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. **[Azure Active Directory]** > **[アプリの登録]** で、 **[新規登録]** を選択します。
1. アプリの **[名前]** を指定します (例: **Blazor クライアント AAD B2C**)。
1. **[サポートされているアカウントの種類]** で、マルチテナント オプションを選択します: **ID プロバイダーまたは組織ディレクトリのアカウント (ユーザー フローでユーザーを認証するため)**
1. **[リダイレクト URI]** ドロップ ダウンの設定を **[Web]** のままとし、次のリダイレクト URI を指定します: `https://localhost:{PORT}/authentication/login-callback`。 Kestrel で実行されているアプリの既定のポートは 5001 です。 アプリが別の Kestrel ポートで実行されている場合は、アプリのポートを使用します。 IIS Express の場合、アプリのランダムに生成されたポートは、 **[デバッグ]** パネルの *`Server`* アプリのプロパティで確認できます。 この時点ではアプリは存在せず、IIS Express ポートは不明であるため、アプリが作成された後にこの手順に戻り、リダイレクト URI を更新してください。 「[アプリを作成する](#create-the-app)」セクションの解説には、IIS Express ユーザーに対してリダイレクト URI の更新を促す注意が示されています。
1. **[アクセス許可]** > **[openid と offline_access アクセス許可に対して管理者の同意を付与します]** が選択されていることを確認します。
1. **[登録]** を選択します。

アプリケーション (クライアント) ID を記録しておきます (例: `4369008b-21fa-427c-abaa-9b53bf58e538`)。

**[認証]** > **[プラットフォーム構成]** > **[Web]** で次の操作を行います。

1. **[リダイレクト URI]** が`https://localhost:{PORT}/authentication/login-callback` であることを確認します。
1. **[暗黙の付与]** で、 **[アクセス トークン]** と **[ID トークン]** のチェック ボックスをオンにします。
1. アプリの残りの既定値は、このエクスペリエンスで使用可能です。
1. **[保存]** ボタンを選択します。

::: moniker-end

**[API のアクセス許可]** で:

1. **[アクセス許可の追加]** を選択し、 **[自分の API]** を選択します。
1. **[名前]** 列で "*サーバー API アプリ*" を選択します (例: **Blazor Server AAD B2C**)。
1. **[API]** の一覧を開きます。
1. API へのアクセスを有効にします (例: `API.Access`)。
1. **[アクセス許可の追加]** を選択します.
1. **[<テナント名> に管理者の同意を与えます]** ボタンを選択します。 **[はい]** を選択して確定します。

**[ホーム]**  >  **[Azure AD B2C]**  >  **[ユーザー フロー]** で、次を行います。

[サインアップとサインイン ユーザー フローを作成する](/azure/active-directory-b2c/tutorial-create-user-flows)

少なくとも、 **[アプリケーション要求]**  >  **[表示名]** ユーザー属性を選択して、`LoginDisplay` コンポーネント (`Shared/LoginDisplay.razor`) に `context.User.Identity.Name` を設定します。

アプリ用に作成されたサインアップおよびサインイン ユーザーフロー名を記録しておきます (例: `B2C_1_signupsignin`)。

### <a name="create-the-app"></a>アプリを作成する

次のコマンドのプレースホルダーを、前に記録した情報に置き換え、コマンド シェルでこのコマンドを実行します。

```dotnetcli
dotnet new blazorwasm -au IndividualB2C --aad-b2c-instance "{AAD B2C INSTANCE}" --api-client-id "{SERVER API APP CLIENT ID}" --app-id-uri "{SERVER API APP ID URI}" --client-id "{CLIENT APP CLIENT ID}" --default-scope "{DEFAULT SCOPE}" --domain "{TENANT DOMAIN}" -ho -o {APP NAME} -ssp "{SIGN UP OR SIGN IN POLICY}"
```

| プレースホルダー                   | Azure portal での名前                                     | 例                                        |
| ----------------------------- | ----------------------------------------------------- | ---------------------------------------------- |
| `{AAD B2C INSTANCE}`          | インスタンス                                              | `https://contoso.b2clogin.com/`                |
| `{APP NAME}`                  | &mdash;                                               | `BlazorSample`                                 |
| `{CLIENT APP CLIENT ID}`      | *`Client`* アプリのアプリケーション (クライアント) ID        | `4369008b-21fa-427c-abaa-9b53bf58e538`         |
| `{DEFAULT SCOPE}`             | スコープ名                                            | `API.Access`                                   |
| `{SERVER API APP CLIENT ID}`  | "*サーバー API アプリ*" のアプリケーション (クライアント) ID      | `41451fa7-82d9-4673-8fa5-69eff5a761fd`         |
| `{SERVER API APP ID URI}`     | アプリケーション ID URI&dagger;                            | `41451fa7-82d9-4673-8fa5-69eff5a761fd`&dagger; |
| `{SIGN UP OR SIGN IN POLICY}` | サインアップまたはサインインのユーザー フロー                             | `B2C_1_signupsignin1`                          |
| `{TENANT DOMAIN}`             | プライマリ、パブリッシャー、テナント ドメイン                       | `contoso.onmicrosoft.com`                      |

&dagger;`dotnet new` コマンドに渡されるアプリ ID URI 引数に、Blazor WebAssembly テンプレートによって `api://` のスキームが自動的に追加されます。 `{SERVER API APP ID URI}` プレースホルダーでアプリ ID URI を指定し、スキームが `api://` の場合は、上記の表の例の値に示すように、引数からスキーム (`api://`) を削除します。 アプリ ID URI がカスタム値の場合、または他の何らかのスキームである場合 (たとえば、`https://contoso.onmicrosoft.com/41451fa7-82d9-4673-8fa5-69eff5a761fd` のような信頼されていない発行元ドメインの `https://`) は、既定のスコープ URI を手動で更新し、テンプレートによって *`Client`* アプリが作成された後で、`api://` スキームを削除する必要があります。 詳細については、「[アクセス トークン スコープ](#access-token-scopes)」セクションの注を参照してください。 Blazor WebAssembly テンプレートは、このようなシナリオに対応するため、ASP.NET Core の今後のリリースで変更される可能性があります。 詳細については、「[Blazor WASM テンプレートでのアプリ ID URI 用の二重スキーム (ホストされた、単一組織) (dotnet/aspnetcore #27417)](https://github.com/dotnet/aspnetcore/issues/27417)」を参照してください。

`-o|--output` オプションで指定した出力場所にプロジェクト フォルダーが存在しない場合は作成されて、アプリの名前の一部になります。

> [!NOTE]
> ホストされている Blazor テンプレートによって設定されるスコープでは、アプリ ID URI ホストが繰り返される場合があります。 `DefaultAccessTokenScopes` コレクションに対して構成されたスコープが *`Client`* アプリの `Program.Main` (`Program.cs`) で正しいことを確認します。

> [!NOTE]
> Azure portal では、 *`Client`* アプリのプラットフォーム構成の **[リダイレクト URI]** は、既定の設定の Kestrel サーバーで実行されるアプリの場合、ポート 5001 に構成されます。
>
> *`Client`* アプリがランダムな IIS Express ポートで実行されている場合、アプリのポートは **[デバッグ]** パネルの "*サーバー API アプリ*" のプロパティで確認できます。
>
> ポートが *`Client`* アプリの既知のポートで事前に構成されていない場合は、Azure portal で *`Client`* アプリの登録に戻り、正しいポートでリダイレクト URI を更新します。

## <a name="server-app-configuration"></a>*`Server`* アプリの構成

"*このセクションは、ソリューションの **`Server`** アプリに関連しています。* "

### <a name="authentication-package"></a>認証パッケージ

::: moniker range=">= aspnetcore-5.0"

Microsoft Identity Platform での ASP.NET Core Web API の呼び出しの認証と承認のサポートは、次のパッケージによって提供されます。

* [`Microsoft.Identity.Web`](https://www.nuget.org/packages/Microsoft.Identity.Web)
* [`Microsoft.Identity.Web.UI`](https://www.nuget.org/packages/Microsoft.Identity.Web.UI)

```xml
<PackageReference Include="Microsoft.Identity.Web" Version="{VERSION}" />
<PackageReference Include="Microsoft.Identity.Web.UI" Version="{VERSION}" />
```

プレースホルダー `{VERSION}` では、NuGet.org のパッケージの **バージョン履歴** にある、アプリの共有フレームワークのバージョンに一致するパッケージの安定した最新バージョンを確認できます。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

ASP.NET Core Web API の呼び出しの認証と承認のサポートは、[`Microsoft.AspNetCore.Authentication.AzureADB2C.UI`](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.AzureADB2C.UI) パッケージによって提供されます。

```xml
<PackageReference Include="Microsoft.AspNetCore.Authentication.AzureADB2C.UI" 
  Version="{VERSION}" />
```

プレースホルダー `{VERSION}` では、[NuGet.org](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.AzureAD.UI) のパッケージの **バージョン履歴** にある、アプリの共有フレームワークのバージョンに一致するパッケージの安定した最新バージョンを確認できます。

::: moniker-end

### <a name="authentication-service-support"></a>認証サービスのサポート

::: moniker range=">= aspnetcore-5.0"

`AddAuthentication` メソッドにより、アプリ内での認証サービスが設定され、JWT ベアラー ハンドラーが既定の認証方法として構成されます。 <xref:Microsoft.Identity.Web.MicrosoftIdentityWebApiAuthenticationBuilderExtensions.AddMicrosoftIdentityWebApi%2A> メソッドによって、Microsoft Identity Platform v2.0 を使用して Web API を保護するようにサービスを構成できます。 このメソッドでは、認証オプションを初期化するために必要な設定で、アプリの構成の `AzureAdB2C` セクションが想定されます。

```csharp
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(Configuration.GetSection("AzureAdB2C"));
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

`AddAuthentication` メソッドにより、アプリ内での認証サービスが設定され、JWT ベアラー ハンドラーが既定の認証方法として構成されます。 <xref:Microsoft.AspNetCore.Authentication.AzureADB2CAuthenticationBuilderExtensions.AddAzureADB2CBearer%2A> メソッドにより、Azure Active Directory B2C によって送信されるトークンを検証するために必要な JWT ベアラー ハンドラーの特定のパラメーターが設定されます。

```csharp
services.AddAuthentication(AzureADB2CDefaults.BearerAuthenticationScheme)
    .AddAzureADB2CBearer(options => Configuration.Bind("AzureAdB2C", options));
```

::: moniker-end

<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A> と <xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization%2A> により、次のようになります。

* アプリにより、受信要求のトークンの解析と検証が試みられます。
* 適切な資格情報なしで保護されたリソースへのアクセスを試みた要求は失敗します。

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

### <a name="useridentityname"></a>User.Identity.Name

既定では、`User.Identity.Name` は設定されません。

`name` 要求の種類から値を受け取るようにアプリを構成するために、以下のようにします。

* `Startup.cs` に <xref:Microsoft.AspNetCore.Authentication.JwtBearer?displayProperty=fullName> の名前空間を追加します。

  ```csharp
  using Microsoft.AspNetCore.Authentication.JwtBearer;
  ```

::: moniker range=">= aspnetcore-5.0"

* `Startup.ConfigureServices` で <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerOptions> の <xref:Microsoft.IdentityModel.Tokens.TokenValidationParameters.NameClaimType?displayProperty=nameWithType> を構成します。

  ```csharp
  services.Configure<JwtBearerOptions>(
      JwtBearerDefaults.AuthenticationScheme, options =>
      {
          options.TokenValidationParameters.NameClaimType = "name";
      });
  ```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* `Startup.ConfigureServices` で <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerOptions> の <xref:Microsoft.IdentityModel.Tokens.TokenValidationParameters.NameClaimType?displayProperty=nameWithType> を構成します。

  ```csharp
  services.Configure<JwtBearerOptions>(
      AzureADB2CDefaults.JwtBearerAuthenticationScheme, options =>
      {
          options.TokenValidationParameters.NameClaimType = "name";
      });
  ```

::: moniker-end

### <a name="app-settings"></a>アプリの設定

`appsettings.json` ファイルには、アクセス トークンの検証に使用される JWT ベアラー ハンドラーを構成するためのオプションが含まれています。

```json
{
  "AzureAdB2C": {
    "Instance": "https://{TENANT}.b2clogin.com/",
    "ClientId": "{SERVER API APP CLIENT ID}",
    "Domain": "{TENANT DOMAIN}",
    "SignUpSignInPolicyId": "{SIGN UP OR SIGN IN POLICY}"
  }
}
```

例:

```json
{
  "AzureAdB2C": {
    "Instance": "https://contoso.b2clogin.com/",
    "ClientId": "41451fa7-82d9-4673-8fa5-69eff5a761fd",
    "Domain": "contoso.onmicrosoft.com",
    "SignUpSignInPolicyId": "B2C_1_signupsignin1",
  }
}
```

### <a name="weatherforecast-controller"></a>WeatherForecast コントローラー

WeatherForecast コントローラー (*Controllers/WeatherForecastController.cs*) を使用すると、[`[Authorize]` 属性](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute)がコントローラーに適用されている状態で、保護された API が公開されます。 次のことを理解しておくことが **重要** です。

* この API コントローラーの [`[Authorize]` 属性](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute)は、この API を不正アクセスから保護する唯一のものです。
* Blazor WebAssembly アプリで使用される [`[Authorize]` 属性](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute)は、アプリが正しく動作するにはユーザーを承認する必要がある、というアプリへのヒントとしてのみ機能します。

```csharp
[Authorize]
[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    [HttpGet]
    public IEnumerable<WeatherForecast> Get()
    {
        ...
    }
}
```

## <a name="client-app-configuration"></a>*`Client`* アプリの構成

"*このセクションは、ソリューションの **`Client`** アプリに関連しています。* "

### <a name="authentication-package"></a>認証パッケージ

個人の B2C アカウント (`IndividualB2C`) を使用するようにアプリを作成すると、アプリは [Microsoft Authentication Library](/azure/active-directory/develop/msal-overview) ([`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal)) のパッケージ参照を自動的に受け取ります。 このパッケージには、アプリでユーザーを認証し、保護された API を呼び出すためのトークンを取得するのに役立つ一連のプリミティブが用意されています。

アプリに認証を追加する場合は、アプリのプロジェクト ファイルにパッケージを手動で追加します。

```xml
<PackageReference Include="Microsoft.Authentication.WebAssembly.Msal" 
  Version="{VERSION}" />
```

プレースホルダー `{VERSION}` では、[NuGet.org](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal) のパッケージの **バージョン履歴** にある、アプリの共有フレームワークのバージョンに一致するパッケージの安定した最新バージョンを確認できます。

[`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal) パッケージによって、[`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) パッケージがアプリに推移的に追加されます。

### <a name="authentication-service-support"></a>認証サービスのサポート

サーバー プロジェクトへの要求を行うときのアクセス トークンが含まれる <xref:System.Net.Http.HttpClient> インスタンスのサポートが追加されます。

`Program.cs`:

```csharp
builder.Services.AddHttpClient("{APP ASSEMBLY}.ServerAPI", client => 
    client.BaseAddress = new Uri(builder.HostEnvironment.BaseAddress))
    .AddHttpMessageHandler<BaseAddressAuthorizationMessageHandler>();

builder.Services.AddScoped(sp => sp.GetRequiredService<IHttpClientFactory>()
    .CreateClient("{APP ASSEMBLY}.ServerAPI"));
```

プレースホルダー `{APP ASSEMBLY}` は、アプリのアセンブリ名です (例: `BlazorSample.ServerAPI`)。

ユーザーの認証に対するサポートは、[`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal) パッケージによって提供される <xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A> 拡張メソッドを使用して、サービス コンテナーに登録されます。 このメソッドでは、アプリが IdentityID プロバイダー (IP) とやり取りするために必要なサービスが設定されます。

`Program.cs`:

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    builder.Configuration.Bind("AzureAdB2C", options.ProviderOptions.Authentication);
    options.ProviderOptions.DefaultAccessTokenScopes.Add("{SCOPE URI}");
});
```

<xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A> メソッドでは、アプリを認証するために必要なパラメーターを構成するためのコールバックを受け入れます。 アプリを構成するために必要な値は、アプリを登録するときに Azure Portal の AAD の構成から取得できます。

構成は `wwwroot/appsettings.json` ファイルによって提供されます。

```json
{
  "AzureAdB2C": {
    "Authority": "{AAD B2C INSTANCE}{TENANT DOMAIN}/{SIGN UP OR SIGN IN POLICY}",
    "ClientId": "{CLIENT APP CLIENT ID}",
    "ValidateAuthority": false
  }
}
```

例:

```json
{
  "AzureAdB2C": {
    "Authority": "https://contoso.b2clogin.com/contoso.onmicrosoft.com/B2C_1_signupsignin1",
    "ClientId": "4369008b-21fa-427c-abaa-9b53bf58e538",
    "ValidateAuthority": false
  }
}
```

### <a name="access-token-scopes"></a>アクセス トークン スコープ

既定のアクセス トークン スコープでは、次のようなアクセス トークン スコープの一覧が表されます。

* サインイン要求に既定で含まれます。
* 認証直後にアクセス トークンをプロビジョニングするために使用されます。

すべてのスコープは、Azure Active Directory の規則に従って同じアプリに属している必要があります。 必要に応じて追加の API アプリ用に追加のスコープを追加できます。

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...
    options.ProviderOptions.DefaultAccessTokenScopes.Add("{SCOPE URI}");
});
```

> [!NOTE]
> `dotnet new` コマンドに渡されるアプリ ID URI 引数に、Blazor WebAssembly テンプレートによって `api://` のスキームが自動的に追加されます。 Blazor プロジェクト テンプレートからアプリを生成するときは、既定のアクセス トークン スコープの値で、Azure portal で指定した正しいカスタム アプリ ID URI の値、または次の **いずれか** の形式の値が使用されていることを確認します。
>
> * ディレクトリの発行元ドメインが **信頼されている** 場合は、通常、既定のアクセス トークンのスコープは次の例のような値になります。`API.Access` は既定のスコープ名です。
>
>   ```csharp
>   options.ProviderOptions.DefaultAccessTokenScopes.Add(
>       "api://41451fa7-82d9-4673-8fa5-69eff5a761fd/API.Access");
>   ```
>
>   二重スキーム (`api://api://...`) の値を調べます。 二重スキームが存在する場合は、最初の `api://` スキームを値から削除します。
>
> * ディレクトリの発行元ドメインが **信頼されていない** 場合は、通常、既定のアクセス トークンのスコープは次の例のような値になります。`API.Access` は既定のスコープ名です。
>
>   ```csharp
>   options.ProviderOptions.DefaultAccessTokenScopes.Add(
>       "https://contoso.onmicrosoft.com/41451fa7-82d9-4673-8fa5-69eff5a761fd/API.Access");
>   ```
>
>   余分な `api://` スキーム (`api://https://contoso.onmicrosoft.com/...`) の値を調べます。 余分な `api://` スキームが存在する場合は、`api://` スキームを値から削除します。
>
> Blazor WebAssembly テンプレートは、このようなシナリオに対応するため、ASP.NET Core の今後のリリースで変更される可能性があります。 詳細については、「[Blazor WASM テンプレートでのアプリ ID URI 用の二重スキーム (ホストされた、単一組織) (dotnet/aspnetcore #27417)](https://github.com/dotnet/aspnetcore/issues/27417)」を参照してください。

`AdditionalScopesToConsent` を使用して追加のスコープを指定します。

```csharp
options.ProviderOptions.AdditionalScopesToConsent.Add("{ADDITIONAL SCOPE URI}");
```

詳細については、"*その他のシナリオ*" に関する記事の次のセクションを参照してください。

* [追加のアクセス トークンを要求する](xref:blazor/security/webassembly/additional-scenarios#request-additional-access-tokens)
* [送信要求にトークンを添付する](xref:blazor/security/webassembly/additional-scenarios#attach-tokens-to-outgoing-requests)

::: moniker range=">= aspnetcore-5.0"

### <a name="login-mode"></a>ログイン モード

[!INCLUDE[](~/blazor/includes/security/msal-login-mode.md)]

::: moniker-end

### <a name="imports-file"></a>インポート ファイル

[!INCLUDE[](~/blazor/includes/security/imports-file-hosted.md)]

### <a name="index-page"></a>インデックス ページ

[!INCLUDE[](~/blazor/includes/security/index-page-msal.md)]

### <a name="app-component"></a>アプリ コンポーネント

[!INCLUDE[](~/blazor/includes/security/app-component.md)]

### <a name="redirecttologin-component"></a>RedirectToLogin コンポーネント

[!INCLUDE[](~/blazor/includes/security/redirecttologin-component.md)]

### <a name="logindisplay-component"></a>LoginDisplay コンポーネント

[!INCLUDE[](~/blazor/includes/security/logindisplay-component.md)]

### <a name="authentication-component"></a>認証コンポーネント

[!INCLUDE[](~/blazor/includes/security/authentication-component.md)]

### <a name="fetchdata-component"></a>FetchData コンポーネント

[!INCLUDE[](~/blazor/includes/security/fetchdata-component.md)]

## <a name="run-the-app"></a>アプリを実行する

サーバー プロジェクトからアプリを実行します。 Visual Studio を使用しているときは、次のいずれかを行います。

* ツール バーの **[スタートアップ プロジェクト]** ドロップダウン リストを "*サーバー API アプリ*" に設定して、 **[実行]** ボタンを選択します。
* **ソリューション エクスプローラー** でサーバー プロジェクトを選択し、ツール バーの **[実行]** ボタンを選択するか、 **[デバッグ]** メニューからアプリを開始します。

<!-- HOLD
[!INCLUDE[](~/blazor/includes/security/usermanager-signinmanager.md)]
-->

[!INCLUDE[](~/blazor/includes/security/wasm-aad-b2c-userflows.md)]

[!INCLUDE[](~/blazor/includes/security/troubleshoot.md)]

## <a name="additional-resources"></a>その他の技術情報

* <xref:blazor/security/webassembly/additional-scenarios>
* [Authentication.MSAL JavaScript ライブラリのカスタム バージョンをビルドする](xref:blazor/security/webassembly/additional-scenarios#build-a-custom-version-of-the-authenticationmsal-javascript-library)
* [セキュリティで保護された既定のクライアントを使用する、アプリ内の認証または承認されていない Web API 要求](xref:blazor/security/webassembly/additional-scenarios#unauthenticated-or-unauthorized-web-api-requests-in-an-app-with-a-secure-default-client)
* <xref:security/authentication/azure-ad-b2c>
* [チュートリアル: Azure Active Directory B2C テナントを作成する](/azure/active-directory-b2c/tutorial-create-tenant)
* [チュートリアル:Azure Active Directory B2C にアプリケーションを登録する](/azure/active-directory-b2c/tutorial-register-applications)
* [Microsoft ID プラットフォームのドキュメント](/azure/active-directory/develop/)
