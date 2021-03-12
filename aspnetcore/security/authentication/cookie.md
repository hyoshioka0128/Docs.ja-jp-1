---
title: 認証を使用 cookie しない ASP.NET Core Identity
author: rick-anderson
description: を使用せずに認証を使用する方法について説明 cookie ASP.NET Core Identity します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 02/11/2020
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
uid: security/authentication/cookie
ms.openlocfilehash: 5b1f1bb3de7126c401a81b89b99a45c7e45f8f8d
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586294"
---
# <a name="use-cookie-authentication-without-aspnet-core-identity"></a>認証を使用 cookie しない ASP.NET Core Identity

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core Identity は、ログインを作成および管理するための完全な機能を備えた完全な認証プロバイダーです。 ただし、を cookie 使用しないベースの認証プロバイダーを ASP.NET Core Identity 使用することはできません。 詳細については、「<xref:security/authentication/identity>」を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/cookie/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

サンプルアプリでのデモンストレーションのために、架空のユーザーであるマリア Rodriguez のユーザーアカウントは、アプリにハードコードされています。 **電子メール** アドレスと任意のパスワードを使用して、 `maria.rodriguez@contoso.com` ユーザーにサインインします。 ユーザーは、 `AuthenticateUser` *Pages/Account/Login. cshtml. .cs* ファイルのメソッドで認証されます。 実際の例では、ユーザーはデータベースに対して認証されます。

## <a name="configuration"></a>構成

メソッドで、メソッド `Startup.ConfigureServices` とメソッドを使用して認証ミドルウェアサービスを作成し <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> ます。

[!code-csharp[](cookie/samples/3.x/CookieSample/Startup.cs?name=snippet1)]

<xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme> に渡されたは、 `AddAuthentication` アプリの既定の認証スキームを設定します。 `AuthenticationScheme` は、認証の複数のインスタンスがあり cookie 、 [特定のスキームで承認](xref:security/authorization/limitingidentitybyscheme)する場合に便利です。 `AuthenticationScheme`を[ Cookie authenticationdefaults に設定します。 authenticationdefaults](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme)は、スキームの値として "s" を提供します。 Cookie スキームを区別する任意の文字列値を指定できます。

アプリの認証方式は、アプリの認証スキームとは異なり cookie ます。 cookieに認証スキームが指定されていない場合は <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> 、 `CookieAuthenticationDefaults.AuthenticationScheme` (" Cookie s") が使用されます。

認証 cookie の <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> プロパティは、既定でに設定され `true` ています。 cookieサイトビジターがデータコレクションに同意していない場合は、認証が許可されます。 詳細については、「<xref:security/gdpr#essential-cookies>」を参照してください。

で、とを呼び出して、 `Startup.Configure` `UseAuthentication` プロパティを `UseAuthorization` 設定し、 `HttpContext.User` 要求の承認ミドルウェアを実行します。 `UseAuthentication`を `UseAuthorization` 呼び出す前に、メソッドとメソッドを呼び出し `UseEndpoints` ます。

[!code-csharp[](cookie/samples/3.x/CookieSample/Startup.cs?name=snippet2)]

クラスは、 <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions> 認証プロバイダーオプションを構成するために使用されます。

`CookieAuthenticationOptions`メソッドの認証用にサービス構成でを設定し `Startup.ConfigureServices` ます。

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        ...
    });
```

## <a name="cookie-policy-middleware"></a>Cookie ポリシーミドルウェア

ポリシー [ Cookie ミドルウェア](xref:Microsoft.AspNetCore.CookiePolicy.CookiePolicyMiddleware)は cookie 、ポリシー機能を有効にします。 アプリ処理パイプラインへのミドルウェアの追加は順序に依存しており、 &mdash; パイプラインに登録されている下流コンポーネントにのみ影響します。

```csharp
app.UseCookiePolicy(cookiePolicyOptions);
```

<xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions>ポリシーミドルウェアに提供されるを使用して、 Cookie cookie cookie cookie が追加または削除されたときの処理ハンドラーへの処理とフックのグローバル特性を制御します。

既定 <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy> 値は、 `SameSiteMode.Lax` OAuth2 認証を許可することです。 と同じサイトポリシーを厳密に適用するに `SameSiteMode.Strict` は、を設定し `MinimumSameSitePolicy` ます。 この設定は、OAuth2 やその他のクロスオリジン認証スキームを分割しますが、 cookie クロスオリジン要求処理に依存しない他の種類のアプリのセキュリティレベルを昇格させます。

```csharp
var cookiePolicyOptions = new CookiePolicyOptions
{
    MinimumSameSitePolicy = SameSiteMode.Strict,
};
```

Cookieのポリシーミドルウェア設定は、次の表 `MinimumSameSitePolicy` に従って設定のの設定に影響する場合が `Cookie.SameSite` `CookieAuthenticationOptions` あります。

| MinimumSameSitePolicy | Cookie.SameSite | 結果 Cookie 。SameSite の設定 |
| --------------------- | --------------- | --------------------------------- |
| SameSiteMode なし     | SameSiteMode なし<br>SameSiteMode 厳密でない<br>SameSiteMode Strict | SameSiteMode なし<br>SameSiteMode 厳密でない<br>SameSiteMode Strict |
| SameSiteMode 厳密でない      | SameSiteMode なし<br>SameSiteMode 厳密でない<br>SameSiteMode Strict | SameSiteMode 厳密でない<br>SameSiteMode 厳密でない<br>SameSiteMode Strict |
| SameSiteMode Strict   | SameSiteMode なし<br>SameSiteMode 厳密でない<br>SameSiteMode Strict | SameSiteMode Strict<br>SameSiteMode Strict<br>SameSiteMode Strict |

## <a name="create-an-authentication-cookie"></a>認証を作成する cookie

保持しているユーザー情報を作成するに cookie は、を構築 <xref:System.Security.Claims.ClaimsPrincipal> します。 ユーザー情報はにシリアル化され、に格納され cookie ます。 

任意の <xref:System.Security.Claims.ClaimsIdentity> が必要なを使用してを作成 <xref:System.Security.Claims.Claim> し <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*> 、を呼び出してユーザーにサインインします。

[!code-csharp[](cookie/samples/3.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet1)]

[!INCLUDE[request localized comments](~/includes/code-comments-loc.md)]

`SignInAsync` 暗号化されたを作成 cookie し、現在の応答に追加します。 が指定されていない場合は `AuthenticationScheme` 、既定のスキームが使用されます。

<xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.RedirectUri> は、既定ではいくつかの特定のパス (ログインパスやログアウトパスなど) でのみ使用されます。 詳細については、「 [ Cookie authenticationhandler source](https://github.com/dotnet/aspnetcore/blob/f2e6e6ff334176540ef0b3291122e359c2106d1a/src/Security/Authentication/Cookies/src/CookieAuthenticationHandler.cs#L334)」を参照してください。

暗号化には ASP.NET Core の [データ保護](xref:security/data-protection/using-data-protection) システムが使用されます。 複数のコンピューターでホストされているアプリの場合、アプリ間で負荷を分散する場合、または web ファームを使用する場合は、同じキーリングとアプリ識別子を使用するように [データ保護を構成](xref:security/data-protection/configuration/overview) します。

## <a name="sign-out"></a>サインアウト

現在のユーザーをサインアウトして削除するには cookie 、次のように呼び出し <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*> ます。

[!code-csharp[](cookie/samples/3.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet2)]

`CookieAuthenticationDefaults.AuthenticationScheme`(または " Cookie s") がスキームとして使用されていない場合 ("Contoso Cookie " など)、認証プロバイダーを構成するときに使用するスキームを指定します。 それ以外の場合は、既定のスキームが使用されます。

ブラウザーが閉じると、セッションベースのが自動的に削除 cookie され cookie ますが (非持続 s)、個々のタブが閉じられても、 cookie はクリアされません。 サーバーには、タブまたはブラウザーの終了イベントは通知されません。

## <a name="react-to-back-end-changes"></a>バックエンドの変更に対処する

が作成されると、は cookie cookie id の単一のソースになります。 バックエンドシステムでユーザーアカウントが無効になっている場合は、次のようになります。

* アプリの cookie 認証システムは、認証に基づいて要求を処理し続け cookie ます。
* 認証が有効である限り、ユーザーはアプリにサインインしたままに cookie なります。

イベントを使用して <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents.ValidatePrincipal*> 、id の検証をインターセプトおよびオーバーライドでき cookie ます。 すべての要求でを検証すると、 cookie アプリにアクセスするユーザーが失効するリスクが軽減されます。

検証の方法 cookie の1つは、ユーザーデータベースがいつ変更されたかを追跡することです。 ユーザーの発行後にデータベースが変更されていない場合は、 cookie ユーザー cookie がまだ有効である場合は、再認証を行う必要はありません。 サンプルアプリでは、データベースはに実装され、 `IUserRepository` 値を格納し `LastChanged` ます。 データベースでユーザーが更新されると、 `LastChanged` 値は現在の時刻に設定されます。

値に基づいてデータベースが変更されたときにを無効にするには、 cookie データベースの現在の値を `LastChanged` 含むクレームを使用してを作成し cookie `LastChanged` `LastChanged` ます。

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, user.Email),
    new Claim("LastChanged", {Database Value})
};

var claimsIdentity = new ClaimsIdentity(
    claims, 
    CookieAuthenticationDefaults.AuthenticationScheme);

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme, 
    new ClaimsPrincipal(claimsIdentity));
```

イベントのオーバーライドを実装するには `ValidatePrincipal` 、から派生するクラスに次のシグネチャを持つメソッドを記述し <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents> ます。

```csharp
ValidatePrincipal(CookieValidatePrincipalContext)
```

の実装例を次に示し `CookieAuthenticationEvents` ます。

```csharp
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;

public class CustomCookieAuthenticationEvents : CookieAuthenticationEvents
{
    private readonly IUserRepository _userRepository;

    public CustomCookieAuthenticationEvents(IUserRepository userRepository)
    {
        // Get the database from registered DI services.
        _userRepository = userRepository;
    }

    public override async Task ValidatePrincipal(CookieValidatePrincipalContext context)
    {
        var userPrincipal = context.Principal;

        // Look for the LastChanged claim.
        var lastChanged = (from c in userPrincipal.Claims
                           where c.Type == "LastChanged"
                           select c.Value).FirstOrDefault();

        if (string.IsNullOrEmpty(lastChanged) ||
            !_userRepository.ValidateLastChanged(lastChanged))
        {
            context.RejectPrincipal();

            await context.HttpContext.SignOutAsync(
                CookieAuthenticationDefaults.AuthenticationScheme);
        }
    }
}
```

サービス登録時にイベントインスタンスを cookie メソッドに登録 `Startup.ConfigureServices` します。 クラスの [スコープ付きサービス登録](xref:fundamentals/dependency-injection#service-lifetimes) を指定し `CustomCookieAuthenticationEvents` ます。

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.EventsType = typeof(CustomCookieAuthenticationEvents);
    });

services.AddScoped<CustomCookieAuthenticationEvents>();
```

ユーザーの名前が、 &mdash; セキュリティに影響しない決定をどのような方法でも更新している場合を考えてみます。 ユーザープリンシパルを非破壊更新する場合は、を呼び出し、 `context.ReplacePrincipal` `context.ShouldRenew` プロパティをに設定し `true` ます。

> [!WARNING]
> ここで説明する方法は、すべての要求でトリガーされます。 cookieすべての要求に対して認証を検証すると、アプリケーションのパフォーマンスが大幅に低下する可能性があります。

## <a name="persistent-cookies"></a>永続的な cookie s

cookieをブラウザーセッション間で永続化することをお勧めします。 この永続化を有効にする必要があるのは、サインイン時に [忘れずに保存] チェックボックスをオンにして、明示的なユーザーの同意を有効にすることだけです。 

次のコードスニペットは、 cookie ブラウザーのクロージャを通過する id と対応するを作成します。 以前に構成したスライド式有効期限の設定はすべて受け入れられます。 ブラウザーを閉じている間にが cookie 期限切れになると、ブラウザーは再起動後にをクリアし cookie ます。

<xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.IsPersistent>をに設定し `true` <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties> ます。

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true
    });
```

## <a name="absolute-cookie-expiration"></a>絶対 cookie 有効期限

絶対有効期限は、で設定でき <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.ExpiresUtc> ます。 永続的なを作成するには cookie 、 `IsPersistent` も設定する必要があります。 それ以外の場合、は cookie セッションベースの有効期間で作成され、保持する認証チケットの前または後に期限切れになる可能性があります。 を設定すると、 `ExpiresUtc` のオプションの値がオーバーライドされ <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions.ExpireTimeSpan> <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions> ます (設定されている場合)。

次のコードスニペットでは、 cookie 20 分間継続する id と対応する id を作成します。 これにより、以前に構成したスライド式有効期限の設定は無視されます。

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true,
        ExpiresUtc = DateTime.UtcNow.AddMinutes(20)
    });
```

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core Identity は、ログインを作成および管理するための完全な機能を備えた完全な認証プロバイダーです。 ただし、を cookie 使用しないベースの認証プロバイダーを ASP.NET Core Identity 使用することはできません。 詳細については、「<xref:security/authentication/identity>」を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/cookie/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

サンプルアプリでのデモンストレーションのために、架空のユーザーであるマリア Rodriguez のユーザーアカウントは、アプリにハードコードされています。 **電子メール** アドレスと任意のパスワードを使用して、 `maria.rodriguez@contoso.com` ユーザーにサインインします。 ユーザーは、 `AuthenticateUser` *Pages/Account/Login. cshtml. .cs* ファイルのメソッドで認証されます。 実際の例では、ユーザーはデータベースに対して認証されます。

## <a name="configuration"></a>構成

アプリで [AspNetCore メタパッケージ](xref:fundamentals/metapackage-app)が使用されていない場合は、 [AspNetCore Cookie のプロジェクトファイルにパッケージ参照を作成します。s](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Cookies/) パッケージ。

メソッドで、メソッド `Startup.ConfigureServices` とメソッドを使用して認証ミドルウェアサービスを作成し <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> ます。

[!code-csharp[](cookie/samples/2.x/CookieSample/Startup.cs?name=snippet1)]

<xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme> に渡されたは、 `AddAuthentication` アプリの既定の認証スキームを設定します。 `AuthenticationScheme` は、認証の複数のインスタンスがあり cookie 、 [特定のスキームで承認](xref:security/authorization/limitingidentitybyscheme)する場合に便利です。 `AuthenticationScheme`を[ Cookie authenticationdefaults に設定します。 authenticationdefaults](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme)は、スキームの値として "s" を提供します。 Cookie スキームを区別する任意の文字列値を指定できます。

アプリの認証方式は、アプリの認証スキームとは異なり cookie ます。 cookieに認証スキームが指定されていない場合は <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> 、 `CookieAuthenticationDefaults.AuthenticationScheme` (" Cookie s") が使用されます。

認証 cookie の <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> プロパティは、既定でに設定され `true` ています。 cookieサイトビジターがデータコレクションに同意していない場合は、認証が許可されます。 詳細については、「<xref:security/gdpr#essential-cookies>」を参照してください。

メソッドで、 `Startup.Configure` メソッドを呼び出し `UseAuthentication` て、プロパティを設定する認証ミドルウェアを呼び出し `HttpContext.User` ます。 `UseAuthentication`またはを呼び出す前に、メソッドを呼び出し `UseMvcWithDefaultRoute` `UseMvc` ます。

[!code-csharp[](cookie/samples/2.x/CookieSample/Startup.cs?name=snippet2)]

クラスは、 <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions> 認証プロバイダーオプションを構成するために使用されます。

`CookieAuthenticationOptions`メソッドの認証用にサービス構成でを設定し `Startup.ConfigureServices` ます。

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        ...
    });
```

## <a name="cookie-policy-middleware"></a>Cookie ポリシーミドルウェア

ポリシー [ Cookie ミドルウェア](xref:Microsoft.AspNetCore.CookiePolicy.CookiePolicyMiddleware)は cookie 、ポリシー機能を有効にします。 アプリ処理パイプラインへのミドルウェアの追加は順序に依存しており、 &mdash; パイプラインに登録されている下流コンポーネントにのみ影響します。

```csharp
app.UseCookiePolicy(cookiePolicyOptions);
```

<xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions>ポリシーミドルウェアに提供されるを使用して、 Cookie cookie cookie cookie が追加または削除されたときの処理ハンドラーへの処理とフックのグローバル特性を制御します。

既定 <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy> 値は、 `SameSiteMode.Lax` OAuth2 認証を許可することです。 と同じサイトポリシーを厳密に適用するに `SameSiteMode.Strict` は、を設定し `MinimumSameSitePolicy` ます。 この設定は、OAuth2 やその他のクロスオリジン認証スキームを分割しますが、 cookie クロスオリジン要求処理に依存しない他の種類のアプリのセキュリティレベルを昇格させます。

```csharp
var cookiePolicyOptions = new CookiePolicyOptions
{
    MinimumSameSitePolicy = SameSiteMode.Strict,
};
```

Cookieのポリシーミドルウェア設定は、次の表 `MinimumSameSitePolicy` に従って設定のの設定に影響する場合が `Cookie.SameSite` `CookieAuthenticationOptions` あります。

| MinimumSameSitePolicy | Cookie.SameSite | 結果 Cookie 。SameSite の設定 |
| --------------------- | --------------- | --------------------------------- |
| SameSiteMode なし     | SameSiteMode なし<br>SameSiteMode 厳密でない<br>SameSiteMode Strict | SameSiteMode なし<br>SameSiteMode 厳密でない<br>SameSiteMode Strict |
| SameSiteMode 厳密でない      | SameSiteMode なし<br>SameSiteMode 厳密でない<br>SameSiteMode Strict | SameSiteMode 厳密でない<br>SameSiteMode 厳密でない<br>SameSiteMode Strict |
| SameSiteMode Strict   | SameSiteMode なし<br>SameSiteMode 厳密でない<br>SameSiteMode Strict | SameSiteMode Strict<br>SameSiteMode Strict<br>SameSiteMode Strict |

## <a name="create-an-authentication-cookie"></a>認証を作成する cookie

保持しているユーザー情報を作成するに cookie は、を構築 <xref:System.Security.Claims.ClaimsPrincipal> します。 ユーザー情報はにシリアル化され、に格納され cookie ます。 

任意の <xref:System.Security.Claims.ClaimsIdentity> が必要なを使用してを作成 <xref:System.Security.Claims.Claim> し <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*> 、を呼び出してユーザーにサインインします。

[!code-csharp[](cookie/samples/2.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet1)]

`SignInAsync` 暗号化されたを作成 cookie し、現在の応答に追加します。 が指定されていない場合は `AuthenticationScheme` 、既定のスキームが使用されます。

暗号化には ASP.NET Core の [データ保護](xref:security/data-protection/using-data-protection) システムが使用されます。 複数のコンピューターでホストされているアプリの場合、アプリ間で負荷を分散する場合、または web ファームを使用する場合は、同じキーリングとアプリ識別子を使用するように [データ保護を構成](xref:security/data-protection/configuration/overview) します。

## <a name="sign-out"></a>サインアウト

現在のユーザーをサインアウトして削除するには cookie 、次のように呼び出し <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*> ます。

[!code-csharp[](cookie/samples/2.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet2)]

`CookieAuthenticationDefaults.AuthenticationScheme`(または " Cookie s") がスキームとして使用されていない場合 ("Contoso Cookie " など)、認証プロバイダーを構成するときに使用するスキームを指定します。 それ以外の場合は、既定のスキームが使用されます。

## <a name="react-to-back-end-changes"></a>バックエンドの変更に対処する

が作成されると、は cookie cookie id の単一のソースになります。 バックエンドシステムでユーザーアカウントが無効になっている場合は、次のようになります。

* アプリの cookie 認証システムは、認証に基づいて要求を処理し続け cookie ます。
* 認証が有効である限り、ユーザーはアプリにサインインしたままに cookie なります。

イベントを使用して <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents.ValidatePrincipal*> 、id の検証をインターセプトおよびオーバーライドでき cookie ます。 すべての要求でを検証すると、 cookie アプリにアクセスするユーザーが失効するリスクが軽減されます。

検証の方法 cookie の1つは、ユーザーデータベースがいつ変更されたかを追跡することです。 ユーザーの発行後にデータベースが変更されていない場合は、 cookie ユーザー cookie がまだ有効である場合は、再認証を行う必要はありません。 サンプルアプリでは、データベースはに実装され、 `IUserRepository` 値を格納し `LastChanged` ます。 データベースでユーザーが更新されると、 `LastChanged` 値は現在の時刻に設定されます。

値に基づいてデータベースが変更されたときにを無効にするには、 cookie データベースの現在の値を `LastChanged` 含むクレームを使用してを作成し cookie `LastChanged` `LastChanged` ます。

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, user.Email),
    new Claim("LastChanged", {Database Value})
};

var claimsIdentity = new ClaimsIdentity(
    claims, 
    CookieAuthenticationDefaults.AuthenticationScheme);

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme, 
    new ClaimsPrincipal(claimsIdentity));
```

イベントのオーバーライドを実装するには `ValidatePrincipal` 、から派生するクラスに次のシグネチャを持つメソッドを記述し <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents> ます。

```csharp
ValidatePrincipal(CookieValidatePrincipalContext)
```

の実装例を次に示し `CookieAuthenticationEvents` ます。

```csharp
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;

public class CustomCookieAuthenticationEvents : CookieAuthenticationEvents
{
    private readonly IUserRepository _userRepository;

    public CustomCookieAuthenticationEvents(IUserRepository userRepository)
    {
        // Get the database from registered DI services.
        _userRepository = userRepository;
    }

    public override async Task ValidatePrincipal(CookieValidatePrincipalContext context)
    {
        var userPrincipal = context.Principal;

        // Look for the LastChanged claim.
        var lastChanged = (from c in userPrincipal.Claims
                           where c.Type == "LastChanged"
                           select c.Value).FirstOrDefault();

        if (string.IsNullOrEmpty(lastChanged) ||
            !_userRepository.ValidateLastChanged(lastChanged))
        {
            context.RejectPrincipal();

            await context.HttpContext.SignOutAsync(
                CookieAuthenticationDefaults.AuthenticationScheme);
        }
    }
}
```

サービス登録時にイベントインスタンスを cookie メソッドに登録 `Startup.ConfigureServices` します。 クラスの [スコープ付きサービス登録](xref:fundamentals/dependency-injection#service-lifetimes) を指定し `CustomCookieAuthenticationEvents` ます。

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.EventsType = typeof(CustomCookieAuthenticationEvents);
    });

services.AddScoped<CustomCookieAuthenticationEvents>();
```

ユーザーの名前が、 &mdash; セキュリティに影響しない決定をどのような方法でも更新している場合を考えてみます。 ユーザープリンシパルを非破壊更新する場合は、を呼び出し、 `context.ReplacePrincipal` `context.ShouldRenew` プロパティをに設定し `true` ます。

> [!WARNING]
> ここで説明する方法は、すべての要求でトリガーされます。 cookieすべての要求に対して認証を検証すると、アプリケーションのパフォーマンスが大幅に低下する可能性があります。

## <a name="persistent-cookies"></a>永続的な cookie s

cookieをブラウザーセッション間で永続化することをお勧めします。 この永続化を有効にする必要があるのは、サインイン時に [忘れずに保存] チェックボックスをオンにして、明示的なユーザーの同意を有効にすることだけです。 

次のコードスニペットは、 cookie ブラウザーのクロージャを通過する id と対応するを作成します。 以前に構成したスライド式有効期限の設定はすべて受け入れられます。 ブラウザーを閉じている間にが cookie 期限切れになると、ブラウザーは再起動後にをクリアし cookie ます。

<xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.IsPersistent>をに設定し `true` <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties> ます。

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true
    });
```

## <a name="absolute-cookie-expiration"></a>絶対 cookie 有効期限

絶対有効期限は、で設定でき <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.ExpiresUtc> ます。 永続的なを作成するには cookie 、 `IsPersistent` も設定する必要があります。 それ以外の場合、は cookie セッションベースの有効期間で作成され、保持する認証チケットの前または後に期限切れになる可能性があります。 を設定すると、 `ExpiresUtc` のオプションの値がオーバーライドされ <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions.ExpireTimeSpan> <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions> ます (設定されている場合)。

次のコードスニペットでは、 cookie 20 分間継続する id と対応する id を作成します。 これにより、以前に構成したスライド式有効期限の設定は無視されます。

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true,
        ExpiresUtc = DateTime.UtcNow.AddMinutes(20)
    });
```

::: moniker-end

## <a name="additional-resources"></a>その他のリソース

* <xref:security/authorization/limitingidentitybyscheme>
* <xref:security/authorization/claims>
* [ポリシーベースのロールチェック](xref:security/authorization/roles#policy-based-role-checks)
* <xref:host-and-deploy/web-farm>
