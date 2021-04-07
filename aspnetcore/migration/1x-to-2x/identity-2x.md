---
title: 認証と Identity ASP.NET Core 2.0 への移行
author: rick-anderson
description: この記事では ASP.NET Core 1.x 認証と ASP.NET Core 2.0 に移行するための最も一般的な手順について説明し Identity ます。
ms.author: scaddie
ms.date: 06/21/2019
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
uid: migration/1x-to-2x/identity-2x
ms.openlocfilehash: 8fcd2f176082f8b8a418af8aaab80ba33e9c3c2e
ms.sourcegitcommit: 0abfe496fed8e9470037c8128efa8a50069ccd52
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/07/2021
ms.locfileid: "106564191"
---
# <a name="migrate-authentication-and-identity-to-aspnet-core-20"></a>認証と Identity ASP.NET Core 2.0 への移行

[Scott Addie](https://github.com/scottaddie)および[hao Kung](https://github.com/HaoK)

ASP.NET Core 2.0 には、認証のための新しいモデルが用意されており、 [Identity](xref:security/authentication/identity) サービスを使用した構成が簡単になります。 認証を使用するか Identity 、次に示すように、新しいモデルを使用するように更新することができる ASP.NET Core 1.x アプリケーション。

## <a name="update-namespaces"></a>名前空間の更新

1.x では、やなどのクラスは `IdentityRole` `IdentityUser` 名前空間にありました `Microsoft.AspNetCore.Identity.EntityFrameworkCore` 。

2.0 では、 <xref:Microsoft.AspNetCore.Identity> このようなクラスのいくつかの新しいホームに名前空間が追加されました。 既定のコードでは Identity 、影響を受けるクラスにはおよびが含ま `ApplicationUser` `Startup` れます。 ステートメントを調整して `using` 、影響を受ける参照を解決します。

<a name="auth-middleware"></a>

## <a name="authentication-middleware-and-services"></a>認証ミドルウェアとサービス

1.x プロジェクトでは、認証はミドルウェアを介して構成されます。 ミドルウェアメソッドは、サポートする各認証スキームに対して呼び出されます。

次の 1. x の例では、スタートアップで Facebook 認証を構成 Identity *し* ます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddIdentity<ApplicationUser, IdentityRole>()
            .AddEntityFrameworkStores<ApplicationDbContext>();
}

public void Configure(IApplicationBuilder app, ILoggerFactory loggerfactory)
{
    app.UseIdentity();
    app.UseFacebookAuthentication(new FacebookOptions {
        AppId = Configuration["auth:facebook:appid"],
        AppSecret = Configuration["auth:facebook:appsecret"]
    });
}
```

2.0 プロジェクトでは、認証はサービスを介して構成されます。 各認証スキームは、 `ConfigureServices` *スタートアップ* のメソッドに登録されます。 `UseIdentity`メソッドはに置き換えられ `UseAuthentication` ます。

次の2.0 例では、 Identity スタートアップでを使用 *し* て Facebook 認証を構成します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddIdentity<ApplicationUser, IdentityRole>()
            .AddEntityFrameworkStores<ApplicationDbContext>();

    // If you want to tweak Identity cookies, they're no longer part of IdentityOptions.
    services.ConfigureApplicationCookie(options => options.LoginPath = "/Account/LogIn");
    services.AddAuthentication()
            .AddFacebook(options =>
            {
                options.AppId = Configuration["auth:facebook:appid"];
                options.AppSecret = Configuration["auth:facebook:appsecret"];
            });
}

public void Configure(IApplicationBuilder app, ILoggerFactory loggerfactory) {
    app.UseAuthentication();
}
```

メソッドは、 `UseAuthentication` 認証ミドルウェアコンポーネントを1つ追加します。これは、自動認証とリモート認証要求の処理を担当します。 個々のミドルウェアコンポーネントはすべて、単一の共通ミドルウェアコンポーネントに置き換えられます。

次に、各主要な認証スキームの2.0 移行手順を示します。

### <a name="cookie-based-authentication"></a>Cookieベースの認証

次の2つのオプションのいずれかを選択し、スタートアップで必要な変更を行い *ます。*

1. cookieでを使用するIdentity
    - `UseIdentity` `UseAuthentication` メソッド内のをに置き換え `Configure` ます。

        ```csharp
        app.UseAuthentication();
        ```

    - `AddIdentity`メソッドでメソッドを呼び出して `ConfigureServices` 、認証サービスを追加し cookie ます。
    - 必要に応じて、 `ConfigureApplicationCookie` メソッドでメソッドまたはメソッドを呼び出して、設定を調整し `ConfigureExternalCookie` `ConfigureServices` Identity cookie ます。

        ```csharp
        services.AddIdentity<ApplicationUser, IdentityRole>()
                .AddEntityFrameworkStores<ApplicationDbContext>()
                .AddDefaultTokenProviders();

        services.ConfigureApplicationCookie(options => options.LoginPath = "/Account/LogIn");
        ```

2. を cookie 指定せずにを使用する Identity
    - メソッドの `UseCookieAuthentication` メソッド呼び出しを `Configure` 次のように置き換え `UseAuthentication` ます。

        ```csharp
        app.UseAuthentication();
        ```

    - `AddAuthentication`メソッドでメソッドとメソッドを呼び出し `AddCookie` `ConfigureServices` ます。

        ```csharp
        // If you don't want the cookie to be automatically authenticated and assigned to HttpContext.User,
        // remove the CookieAuthenticationDefaults.AuthenticationScheme parameter passed to AddAuthentication.
        services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
                .AddCookie(options =>
                {
                    options.LoginPath = "/Account/LogIn";
                    options.LogoutPath = "/Account/LogOff";
                });
        ```

### <a name="jwt-bearer-authentication"></a>JWT ベアラー認証

*スタートアップ* で次の変更を行います。
- メソッドの `UseJwtBearerAuthentication` メソッド呼び出しを `Configure` 次のように置き換え `UseAuthentication` ます。

    ```csharp
    app.UseAuthentication();
    ```

- `AddJwtBearer`メソッドでメソッドを呼び出し `ConfigureServices` ます。

    ```csharp
    services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
            .AddJwtBearer(options =>
            {
                options.Audience = "http://localhost:5001/";
                options.Authority = "http://localhost:5000/";
            });
    ```

    このコードスニペットはを使用しない Identity ため、既定のスキームはメソッドに渡すことによって設定する必要があり `JwtBearerDefaults.AuthenticationScheme` `AddAuthentication` ます。

### <a name="openid-connect-oidc-authentication"></a>OpenID Connect (OIDC) 認証

*スタートアップ* で次の変更を行います。

- メソッドの `UseOpenIdConnectAuthentication` メソッド呼び出しを `Configure` 次のように置き換え `UseAuthentication` ます。

    ```csharp
    app.UseAuthentication();
    ```

- `AddOpenIdConnect`メソッドでメソッドを呼び出し `ConfigureServices` ます。

    ```csharp
    services.AddAuthentication(options =>
    {
        options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
    })
    .AddCookie()
    .AddOpenIdConnect(options =>
    {
        options.Authority = Configuration["auth:oidc:authority"];
        options.ClientId = Configuration["auth:oidc:clientid"];
    });
    ```

- アクションの `PostLogoutRedirectUri` プロパティを `OpenIdConnectOptions` 次のように置き換え `SignedOutRedirectUri` ます。

    ```csharp
    .AddOpenIdConnect(options =>
    {
        options.SignedOutRedirectUri = "https://contoso.com";
    });
    ```
    
### <a name="facebook-authentication"></a>Facebook での認証

*スタートアップ* で次の変更を行います。
- メソッドの `UseFacebookAuthentication` メソッド呼び出しを `Configure` 次のように置き換え `UseAuthentication` ます。

    ```csharp
    app.UseAuthentication();
    ```

- `AddFacebook`メソッドでメソッドを呼び出し `ConfigureServices` ます。

    ```csharp
    services.AddAuthentication()
            .AddFacebook(options =>
            {
                options.AppId = Configuration["auth:facebook:appid"];
                options.AppSecret = Configuration["auth:facebook:appsecret"];
            });
    ```

### <a name="google-authentication"></a>Google での認証

*スタートアップ* で次の変更を行います。
- メソッドの `UseGoogleAuthentication` メソッド呼び出しを `Configure` 次のように置き換え `UseAuthentication` ます。

    ```csharp
    app.UseAuthentication();
    ```

- `AddGoogle`メソッドでメソッドを呼び出し `ConfigureServices` ます。

    ```csharp
    services.AddAuthentication()
            .AddGoogle(options =>
            {
                options.ClientId = Configuration["auth:google:clientid"];
                options.ClientSecret = Configuration["auth:google:clientsecret"];
            });
    ```

### <a name="microsoft-account-authentication"></a>Microsoft アカウント認証

Microsoft アカウント認証の詳細については、 [GitHub の問題](https://github.com/dotnet/AspNetCore.Docs/issues/14455)を参照してください。

*スタートアップ* で次の変更を行います。
- メソッドの `UseMicrosoftAccountAuthentication` メソッド呼び出しを `Configure` 次のように置き換え `UseAuthentication` ます。

    ```csharp
    app.UseAuthentication();
    ```

- `AddMicrosoftAccount`メソッドでメソッドを呼び出し `ConfigureServices` ます。

    ```csharp
    services.AddAuthentication()
            .AddMicrosoftAccount(options =>
            {
                options.ClientId = Configuration["auth:microsoft:clientid"];
                options.ClientSecret = Configuration["auth:microsoft:clientsecret"];
            });
    ```

### <a name="twitter-authentication"></a>Twitter での認証

*スタートアップ* で次の変更を行います。
- メソッドの `UseTwitterAuthentication` メソッド呼び出しを `Configure` 次のように置き換え `UseAuthentication` ます。

    ```csharp
    app.UseAuthentication();
    ```

- `AddTwitter`メソッドでメソッドを呼び出し `ConfigureServices` ます。

    ```csharp
    services.AddAuthentication()
            .AddTwitter(options =>
            {
                options.ConsumerKey = Configuration["auth:twitter:consumerkey"];
                options.ConsumerSecret = Configuration["auth:twitter:consumersecret"];
            });
    ```

### <a name="setting-default-authentication-schemes"></a>既定の認証方式を設定する

1.x では、 `AutomaticAuthenticate` `AutomaticChallenge` [authenticationoptions](/dotnet/api/Microsoft.AspNetCore.Builder.AuthenticationOptions?view=aspnetcore-1.1) 基本クラスのプロパティとプロパティが、1つの認証スキームで設定されることを意図していました。 これを実施するための適切な方法はありませんでした。

2.0 では、これら2つのプロパティは個別のインスタンスのプロパティとして削除されてい `AuthenticationOptions` ます。 これらは、 `AddAuthentication` `ConfigureServices` *スタートアップ* のメソッド内のメソッド呼び出しで構成できます。

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme);
```

前のコードスニペットでは、既定のスキームは `CookieAuthenticationDefaults.AuthenticationScheme` ("s") に設定されて Cookie います。

または、オーバーロードされたバージョンのメソッドを使用して、複数の `AddAuthentication` プロパティを設定します。 次のオーバーロードされたメソッドの例では、既定のスキームはに設定されて `CookieAuthenticationDefaults.AuthenticationScheme` います。 認証方式は、個々の `[Authorize]` 属性または承認ポリシー内で指定することもできます。

```csharp
services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
});
```

次のいずれかの条件に該当する場合は、2.0 で既定のスキームを定義します。
- ユーザーが自動的にサインインするようにするには
- `[Authorize]`スキーマを指定せずに属性または承認ポリシーを使用する

この規則の例外は `AddIdentity` メソッドです。 このメソッドは、を追加 cookie し、既定の認証およびチャレンジのスキームをアプリケーションに設定し cookie `IdentityConstants.ApplicationScheme` ます。 また、既定のサインインスキームが外部に設定され cookie `IdentityConstants.ExternalScheme` ます。

<a name="obsolete-interface"></a>

## <a name="use-httpcontext-authentication-extensions"></a>HttpContext 認証拡張機能の使用

`IAuthenticationManager`インターフェイスは、1. x 認証システムへのメインエントリポイントです。 名前空間の新しい拡張メソッドのセットに置き換えられました `HttpContext` `Microsoft.AspNetCore.Authentication` 。

たとえば、1. x プロジェクトはプロパティを参照します。 `Authentication`

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Controllers/AccountController.cs?name=snippet_AuthenticationProperty)]

2.0 プロジェクトで、 `Microsoft.AspNetCore.Authentication` 名前空間をインポートし、プロパティ参照を削除し `Authentication` ます。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Controllers/AccountController.cs?name=snippet_AuthenticationProperty)]

<a name="windows-auth-changes"></a>

## <a name="windows-authentication-httpsys--iisintegration"></a>Windows 認証 (HTTP.sys/IISIntegration)

Windows 認証には、次の2つのバリエーションがあります。

* ホストは、認証されたユーザーのみを許可します。 このバリエーションは、2.0 の変更の影響を受けません。
* ホストは、匿名ユーザーと認証済みユーザーの両方を許可します。 このバリエーションは、2.0 の変更の影響を受けます。 たとえば、アプリでは、 [IIS](xref:host-and-deploy/iis/index) または [HTTP.sys](xref:fundamentals/servers/httpsys) レイヤーで匿名ユーザーを許可する必要がありますが、コントローラーレベルでユーザーを承認する必要があります。 このシナリオでは、メソッドで既定のスキームを設定し `Startup.ConfigureServices` ます。

  [AspNetCore 統合](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.IISIntegration/)では、既定のスキームを次のように設定します。 `IISDefaults.AuthenticationScheme`

  ```csharp
  using Microsoft.AspNetCore.Server.IISIntegration;

  services.AddAuthentication(IISDefaults.AuthenticationScheme);
  ```

  [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.HttpSys/)の場合は、既定のスキームを次のように設定します。 `HttpSysDefaults.AuthenticationScheme`

  ```csharp
  using Microsoft.AspNetCore.Server.HttpSys;

  services.AddAuthentication(HttpSysDefaults.AuthenticationScheme);
  ```

  既定のスキームを設定しなかった場合は、次の例外で承認 (チャレンジ) 要求を処理できなくなります。

  > `System.InvalidOperationException`: AuthenticationScheme が指定されていません。 DefaultChallengeScheme が見つかりませんでした。

詳細については、「<xref:security/authentication/windowsauth>」を参照してください。

<a name="identity-cookie-options"></a>

## <a name="identitycookieoptions-instances"></a>IdentityCookieオプションのインスタンス

2.0 の変更の副作用は、オプションのインスタンスの代わりに名前付きオプションを使用するように切り替えることです cookie 。 スキーム名をカスタマイズする機能 Identity cookie は削除されます。

たとえば、1.x プロジェクトでは、 [コンストラクターの挿入](xref:mvc/controllers/dependency-injection#constructor-injection) を使用し `IdentityCookieOptions` て、 *accountcontroller* と *ManageController* にパラメーターを渡します。 外部 cookie 認証スキームは、指定されたインスタンスからアクセスされます。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Controllers/AccountController.cs?name=snippet_AccountControllerConstructor&highlight=4,11)]

上記のコンストラクターインジェクションは、2.0 プロジェクトでは不要になり、 `_externalCookieScheme` フィールドを削除することができます。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Controllers/AccountController.cs?name=snippet_AccountControllerConstructor)]

1.x プロジェクトでは、次のようにフィールドが使用されていました。 `_externalCookieScheme`

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Controllers/AccountController.cs?name=snippet_AuthenticationProperty)]

2.0 プロジェクトで、上記のコードを次のコードに置き換えます。 定数は、 `IdentityConstants.ExternalScheme` 直接使用できます。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Controllers/AccountController.cs?name=snippet_AuthenticationProperty)]

次の名前空間をインポートして、新しく追加された呼び出しを解決し `SignOutAsync` ます。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Controllers/AccountController.cs?name=snippet_AuthenticationImport)]

<a name="navigation-properties"></a>

## <a name="add-identityuser-poco-navigation-properties"></a>Identityユーザー POCO ナビゲーションプロパティの追加

基本 POCO (Plain Old CLR Object) の Entity Framework (EF) コアナビゲーションプロパティ `IdentityUser` が削除されました。 1.x プロジェクトでこれらのプロパティを使用していた場合は、手動で2.0 プロジェクトに追加します。

```csharp
/// <summary>
/// Navigation property for the roles this user belongs to.
/// </summary>
public virtual ICollection<IdentityUserRole<int>> Roles { get; } = new List<IdentityUserRole<int>>();

/// <summary>
/// Navigation property for the claims this user possesses.
/// </summary>
public virtual ICollection<IdentityUserClaim<int>> Claims { get; } = new List<IdentityUserClaim<int>>();

/// <summary>
/// Navigation property for this users login accounts.
/// </summary>
public virtual ICollection<IdentityUserLogin<int>> Logins { get; } = new List<IdentityUserLogin<int>>();
```

EF Core 移行の実行時に外部キーが重複しないようにするには、 `IdentityDbContext` クラスの `OnModelCreating` メソッド (呼び出しの後) に次のを追加し `base.OnModelCreating();` ます。

```csharp
protected override void OnModelCreating(ModelBuilder builder)
{
    base.OnModelCreating(builder);
    // Customize the ASP.NET Core Identity model and override the defaults if needed.
    // For example, you can rename the ASP.NET Core Identity table names and more.
    // Add your customizations after calling base.OnModelCreating(builder);

    builder.Entity<ApplicationUser>()
        .HasMany(e => e.Claims)
        .WithOne()
        .HasForeignKey(e => e.UserId)
        .IsRequired()
        .OnDelete(DeleteBehavior.Cascade);

    builder.Entity<ApplicationUser>()
        .HasMany(e => e.Logins)
        .WithOne()
        .HasForeignKey(e => e.UserId)
        .IsRequired()
        .OnDelete(DeleteBehavior.Cascade);

    builder.Entity<ApplicationUser>()
        .HasMany(e => e.Roles)
        .WithOne()
        .HasForeignKey(e => e.UserId)
        .IsRequired()
        .OnDelete(DeleteBehavior.Cascade);
}
```

<a name="synchronous-method-removal"></a>

## <a name="replace-getexternalauthenticationschemes"></a>GetExternalAuthenticationSchemes の置換

`GetExternalAuthenticationSchemes`非同期バージョンを優先するため、同期メソッドが削除されました。 1.x プロジェクトでは、 *Controllers/ManageController* に次のコードが含まれています。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Controllers/ManageController.cs?name=snippet_GetExternalAuthenticationSchemes)]

このメソッドは *Views/Account/Login. cshtml* にも表示されます。

[!code-cshtml[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Views/Account/Login.cshtml?name=snippet_GetExtAuthNSchemes&highlight=2)]

2.0 プロジェクトでは、メソッドを使用し <xref:Microsoft.AspNetCore.Identity.SignInManager`1.GetExternalAuthenticationSchemesAsync*> ます。 ManageController の変更は、次のコードのように *なり* ます。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Controllers/ManageController.cs?name=snippet_GetExternalAuthenticationSchemesAsync)]

*Login. cshtml* で、 `AuthenticationScheme` ループでアクセスされるプロパティが `foreach` 次のように変更されます。 `Name`

[!code-cshtml[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Views/Account/Login.cshtml?name=snippet_GetExtAuthNSchemesAsync&highlight=2,19)]

<a name="property-change"></a>

## <a name="manageloginsviewmodel-property-change"></a>ManageLoginsViewModel プロパティの変更

`ManageLoginsViewModel`オブジェクトは、 `ManageLogins` *ManageController* のアクションで使用されます。 1.x プロジェクトでは、オブジェクトのプロパティの `OtherLogins` 戻り値の型は `IList<AuthenticationDescription>` です。 この戻り値の型をインポートするには、次のものが必要です `Microsoft.AspNetCore.Http.Authentication` 。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Models/ManageViewModels/ManageLoginsViewModel.cs?name=snippet_ManageLoginsViewModel&highlight=2,11)]

2.0 プロジェクトでは、戻り値の型はに変更され `IList<AuthenticationScheme>` ます。 この新しい戻り値の型では、インポートをインポートに置き換える必要があり `Microsoft.AspNetCore.Http.Authentication` `Microsoft.AspNetCore.Authentication` ます。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Models/ManageViewModels/ManageLoginsViewModel.cs?name=snippet_ManageLoginsViewModel&highlight=2,11)]

<a name="additional-resources"></a>

## <a name="additional-resources"></a>その他の技術情報

詳細については、GitHub で [の Auth 2.0 の問題の説明](https://github.com/aspnet/Security/issues/1338) を参照してください。
