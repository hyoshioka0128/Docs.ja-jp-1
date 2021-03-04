---
title: ASP.NET Core で外部プロバイダーからの追加の要求とトークンを保持する
author: rick-anderson
description: 外部プロバイダーから追加の要求とトークンを確立する方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/18/2021
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
uid: security/authentication/social/additional-claims
ms.openlocfilehash: 9c04ca466566e956b5e6dfec8131096c3995bc14
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102110145"
---
# <a name="persist-additional-claims-and-tokens-from-external-providers-in-aspnet-core"></a>ASP.NET Core で外部プロバイダーからの追加の要求とトークンを保持する

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core アプリは、Facebook、Google、Microsoft、Twitter などの外部認証プロバイダーからの追加の要求とトークンを確立できます。 各プロバイダーは、プラットフォーム上のユーザーに関するさまざまな情報を明らかにしますが、ユーザーデータを受け取って追加の要求に変換するパターンは同じです。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/social/additional-claims/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="prerequisites"></a>必須コンポーネント

アプリでサポートする外部認証プロバイダーを決定します。 各プロバイダーについて、アプリを登録し、クライアント ID とクライアントシークレットを取得します。 詳細については、「<xref:security/authentication/social/index>」を参照してください。 このサンプルアプリでは、 [Google 認証プロバイダー](xref:security/authentication/google-logins)を使用します。

## <a name="set-the-client-id-and-client-secret"></a>クライアント ID とクライアントシークレットを設定する

OAuth 認証プロバイダーは、クライアント ID とクライアントシークレットを使用して、アプリとの信頼関係を確立します。 アプリがプロバイダーに登録されるときに、外部認証プロバイダーによってアプリのクライアント ID とクライアントシークレットの値が作成されます。 アプリが使用する各外部プロバイダーは、プロバイダーのクライアント ID とクライアントシークレットとは別に構成する必要があります。 詳細については、シナリオに当てはまる外部認証プロバイダーのトピックを参照してください。

* [Facebook での認証](xref:security/authentication/facebook-logins)
* [Google での認証](xref:security/authentication/google-logins)
* [Microsoft での認証](xref:security/authentication/microsoft-logins)
* [Twitter での認証](xref:security/authentication/twitter-logins)
* [他の認証プロバイダー](xref:security/authentication/otherlogins)
* [OpenIdConnect](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2)

このサンプルアプリでは、google によって提供されるクライアント ID とクライアントシークレットを使用して Google 認証プロバイダーを構成します。

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=4,9)]

## <a name="establish-the-authentication-scope"></a>認証スコープを確立する

を指定して、プロバイダーから取得するアクセス許可の一覧を指定し <xref:Microsoft.AspNetCore.Authentication.OAuth.OAuthOptions.Scope*> ます。 共通外部プロバイダーの認証スコープは、次の表に表示されます。

| プロバイダー  | Scope                                                            |
| --------- | ---------------------------------------------------------------- |
| Facebook  | `https://www.facebook.com/dialog/oauth`                          |
| Google    | `profile`, `email`, `openid`                                     |
| Microsoft | `https://login.microsoftonline.com/common/oauth2/v2.0/authorize` |
| Twitter   | `https://api.twitter.com/oauth/authenticate`                     |

サンプルアプリで `profile` は、がで呼び出されると、Google の、、の各 `email` `openid` スコープがフレームワークによって自動的に追加され <xref:Microsoft.Extensions.DependencyInjection.GoogleExtensions.AddGoogle%2A> <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilder> ます。 アプリに追加のスコープが必要な場合は、それらをオプションに追加します。 次の例で `https://www.googleapis.com/auth/user.birthday.read` は、ユーザーの誕生日を取得するために Google スコープが追加されています。

```csharp
options.Scope.Add("https://www.googleapis.com/auth/user.birthday.read");
```

## <a name="map-user-data-keys-and-create-claims"></a>ユーザーデータキーをマップして要求を作成する

プロバイダーのオプションで、 <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonKey*> <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonSubKey*> サインイン時に読み取るアプリ id の外部プロバイダーの JSON ユーザーデータのキー/サブキーごとに、またはを指定します。 要求の種類の詳細については、「」を参照してください <xref:System.Security.Claims.ClaimTypes> 。

このサンプルアプリでは、 `urn:google:locale` `urn:google:picture` `locale` Google ユーザーデータのキーとキーから、ロケール () と画像 () の要求を作成し `picture` ます。

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=13-14)]

では、 `Microsoft.AspNetCore.Identity.UI.Pages.Account.Internal.ExternalLoginModel.OnPostConfirmationAsync` <xref:Microsoft.AspNetCore.Identity.IdentityUser> ( `ApplicationUser` ) がを使用してアプリにサインイン <xref:Microsoft.AspNetCore.Identity.SignInManager%601.SignInAsync*> します。 サインインプロセス中に、で <xref:Microsoft.AspNetCore.Identity.UserManager%601> `ApplicationUser` 使用可能なユーザーデータの要求を格納でき <xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.Principal*> ます。

サンプルアプリでは、 `OnPostConfirmationAsync` (*Account/externallogin. cshtml*) によって、 `urn:google:locale` `urn:google:picture` `ApplicationUser` 次の要求を含む、サインインしたのロケール () と画像 () の要求が確立 <xref:System.Security.Claims.ClaimTypes.GivenName> されます。

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=35-51)]

既定では、ユーザーの要求は認証に格納され cookie ます。 認証 cookie が大きすぎると、次の理由により、アプリでエラーが発生する可能性があります。

* ブラウザーは、 cookie ヘッダーが長すぎることを検出します。
* 要求の全体的なサイズが大きすぎます。

ユーザーの要求を処理するために大量のユーザーデータが必要な場合は、次のようにします。

* 要求処理のユーザー要求の数とサイズは、アプリで必要なもののみに制限します。
* 認証ミドルウェアのカスタムを使用し <xref:Microsoft.AspNetCore.Authentication.Cookies.ITicketStore> て、 Cookie <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions.SessionStore> 要求間で id を格納します。 小さいセッション識別子キーをクライアントに送信するだけで、サーバー上の大量の id 情報を保持します。

## <a name="save-the-access-token"></a>アクセストークンを保存する

<xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.SaveTokens*> 承認が成功した後に、アクセストークンと更新トークンをに格納するかどうかを定義し <xref:Microsoft.AspNetCore.Http.Authentication.AuthenticationProperties> ます。 `SaveTokens` は `false` 、最終的な認証のサイズを小さくするために、既定でに設定され cookie ます。

サンプルアプリでは、の値をに設定し `SaveTokens` `true` <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions> ます。

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=15)]

を実行するときに、 `OnPostConfirmationAsync` の外部プロバイダーからアクセストークン ([Externallogininfo. authenticationtokens](xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.AuthenticationTokens*)) をに格納 `ApplicationUser` `AuthenticationProperties` します。

サンプルアプリでは、アクセストークン `OnPostConfirmationAsync` (新しいユーザー登録) と `OnGetCallbackAsync` (以前に登録されたユーザー) を *Account/externallogin. cshtml. cs* に保存します。

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=54-56)]

> [!NOTE]
> アプリのコンポーネントにトークンを渡す方法の詳細につい Razor Blazor Server ては、「」を参照してください <xref:blazor/security/server/additional-scenarios#pass-tokens-to-a-blazor-server-app> 。

## <a name="how-to-add-additional-custom-tokens"></a>カスタムトークンを追加する方法

の一部として格納されているカスタムトークンを追加する方法を示すために、 `SaveTokens` サンプルアプリでは、の AuthenticationToken.Name の現在のを使用してを追加し <xref:Microsoft.AspNetCore.Authentication.AuthenticationToken> <xref:System.DateTime> [](xref:Microsoft.AspNetCore.Authentication.AuthenticationToken.Name*) `TicketCreated` ます。

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=17-30)]

## <a name="creating-and-adding-claims"></a>要求の作成と追加

フレームワークには、要求を作成してコレクションに追加するための一般的なアクションと拡張メソッドが用意されています。 詳細については、「 <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions> 」および「 <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionUniqueExtensions>」を参照してください。

ユーザーは、から派生し、抽象メソッドを実装することによって、カスタムアクションを定義でき <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction> <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.Run*> ます。

詳細については、「<xref:Microsoft.AspNetCore.Authentication.OAuth.Claims>」を参照してください。

## <a name="add-and-update-user-claims"></a>ユーザー要求の追加と更新

要求は、サインイン時ではなく、最初の登録時に外部プロバイダーからユーザーデータベースにコピーされます。 ユーザーがアプリを使用するように登録した後、アプリで追加の要求が有効になっている場合は、ユーザーの [RefreshSignInAsync](xref:Microsoft.AspNetCore.Identity.SignInManager%601) を呼び出して、新しい認証を強制的に生成 cookie します。

テストユーザーアカウントを使用する開発環境では、単にユーザーアカウントを削除して再作成することができます。 実稼働システムでは、アプリに追加された新しい要求をユーザーアカウントにバックすることができます。 で [ `ExternalLogin` ページ](xref:security/authentication/scaffold-identity) をアプリにスキャフォールディングした後 `Areas/Pages/Identity/Account/Manage` 、ファイル内のに次のコードを追加し `ExternalLoginModel` `ExternalLogin.cshtml.cs` ます。

追加された要求の辞書を追加します。 ディクショナリキーを使用して要求の種類を保持し、値を使用して既定値を保持します。 クラスの先頭に次の行を追加します。 次の例では、既定値として一般的なヘッドショットイメージを使用して、ユーザーの Google picture に1つの要求が追加されていることを前提としています。

```csharp
private readonly IReadOnlyDictionary<string, string> _claimsToSync = 
    new Dictionary<string, string>()
    {
        { "urn:google:picture", "https://localhost:5001/headshot.png" },
    };
```

メソッドの既定のコードを `OnGetCallbackAsync` 次のコードに置き換えます。 このコードは、要求ディクショナリをループ処理します。 要求は、ユーザーごとに追加 (バックフィル) または更新されます。 要求が追加または更新されると、を使用してユーザーのサインインが更新され、 <xref:Microsoft.AspNetCore.Identity.SignInManager%601> 既存の認証プロパティ () が保持され `AuthenticationProperties` ます。

```csharp
public async Task<IActionResult> OnGetCallbackAsync(
    string returnUrl = null, string remoteError = null)
{
    returnUrl = returnUrl ?? Url.Content("~/");

    if (remoteError != null)
    {
        ErrorMessage = $"Error from external provider: {remoteError}";

        return RedirectToPage("./Login", new {ReturnUrl = returnUrl });
    }

    var info = await _signInManager.GetExternalLoginInfoAsync();

    if (info == null)
    {
        ErrorMessage = "Error loading external login information.";
        return RedirectToPage("./Login", new { ReturnUrl = returnUrl });
    }

    // Sign in the user with this external login provider if the user already has a 
    // login.
    var result = await _signInManager.ExternalLoginSignInAsync(info.LoginProvider, 
        info.ProviderKey, isPersistent: false, bypassTwoFactor : true);

    if (result.Succeeded)
    {
        _logger.LogInformation("{Name} logged in with {LoginProvider} provider.", 
            info.Principal.Identity.Name, info.LoginProvider);

        if (_claimsToSync.Count > 0)
        {
            var user = await _userManager.FindByLoginAsync(info.LoginProvider, 
                info.ProviderKey);
            var userClaims = await _userManager.GetClaimsAsync(user);
            bool refreshSignIn = false;

            foreach (var addedClaim in _claimsToSync)
            {
                var userClaim = userClaims
                    .FirstOrDefault(c => c.Type == addedClaim.Key);

                if (info.Principal.HasClaim(c => c.Type == addedClaim.Key))
                {
                    var externalClaim = info.Principal.FindFirst(addedClaim.Key);

                    if (userClaim == null)
                    {
                        await _userManager.AddClaimAsync(user, 
                            new Claim(addedClaim.Key, externalClaim.Value));
                        refreshSignIn = true;
                    }
                    else if (userClaim.Value != externalClaim.Value)
                    {
                        await _userManager
                            .ReplaceClaimAsync(user, userClaim, externalClaim);
                        refreshSignIn = true;
                    }
                }
                else if (userClaim == null)
                {
                    // Fill with a default value
                    await _userManager.AddClaimAsync(user, new Claim(addedClaim.Key, 
                        addedClaim.Value));
                    refreshSignIn = true;
                }
            }

            if (refreshSignIn)
            {
                await _signInManager.RefreshSignInAsync(user);
            }
        }

        return LocalRedirect(returnUrl);
    }

    if (result.IsLockedOut)
    {
        return RedirectToPage("./Lockout");
    }
    else
    {
        // If the user does not have an account, then ask the user to create an 
        // account.
        ReturnUrl = returnUrl;
        ProviderDisplayName = info.ProviderDisplayName;

        if (info.Principal.HasClaim(c => c.Type == ClaimTypes.Email))
        {
            Input = new InputModel
            {
                Email = info.Principal.FindFirstValue(ClaimTypes.Email)
            };
        }

        return Page();
    }
}
```

同様のアプローチは、ユーザーがサインインしている間に要求が変更されても、バックフィルステップが不要な場合にも行われます。 ユーザーの要求を更新するには、ユーザーに対して次のように呼び出します。

* Usermanager。 id データベースに格納されているクレームに対して、ユーザーの[Eclaimasync](xref:Microsoft.AspNetCore.Identity.UserManager%601)を置き換えます。
* 新しい認証を強制的に生成するために、ユーザーの[RefreshSignInAsync を SignInManager します](xref:Microsoft.AspNetCore.Identity.SignInManager%601) cookie 。

## <a name="removal-of-claim-actions-and-claims"></a>要求アクションと要求の削除

[Claimactioncollection。 Remove (String)](xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimActionCollection.Remove*) は、指定されたのすべての要求アクションを <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> コレクションから削除します。 [DeleteClaim (ClaimActionCollection, String)](xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*) は、指定されたの要求を <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> id から削除します。 <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*> は、主に [OpenID connect (OIDC)](/azure/active-directory/develop/v2-protocols-oidc) と共に使用して、プロトコルによって生成された要求を削除します。

## <a name="sample-app-output"></a>サンプルアプリの出力

```
User Claims

http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier
    9b342344f-7aab-43c2-1ac1-ba75912ca999
http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name
    someone@gmail.com
AspNet.Identity.SecurityStamp
    7D4312MOWRYYBFI1KXRPHGOSTBVWSFDE
http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
    Judy
urn:google:locale
    en
urn:google:picture
    https://lh4.googleusercontent.com/-XXXXXX/XXXXXX/XXXXXX/XXXXXX/photo.jpg

Authentication Properties

.Token.access_token
    yc23.AlvoZqz56...1lxltXV7D-ZWP9
.Token.token_type
    Bearer
.Token.expires_at
    2019-04-11T22:14:51.0000000+00:00
.Token.TicketCreated
    4/11/2019 9:14:52 PM
.TokenNames
    access_token;token_type;expires_at;TicketCreated
.persistent
.issued
    Thu, 11 Apr 2019 20:51:06 GMT
.expires
    Thu, 25 Apr 2019 20:51:06 GMT

```

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core アプリは、Facebook、Google、Microsoft、Twitter などの外部認証プロバイダーからの追加の要求とトークンを確立できます。 各プロバイダーは、プラットフォーム上のユーザーに関するさまざまな情報を明らかにしますが、ユーザーデータを受け取って追加の要求に変換するパターンは同じです。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/social/additional-claims/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="prerequisites"></a>必須コンポーネント

アプリでサポートする外部認証プロバイダーを決定します。 各プロバイダーについて、アプリを登録し、クライアント ID とクライアントシークレットを取得します。 詳細については、「<xref:security/authentication/social/index>」を参照してください。 このサンプルアプリでは、 [Google 認証プロバイダー](xref:security/authentication/google-logins)を使用します。

## <a name="set-the-client-id-and-client-secret"></a>クライアント ID とクライアントシークレットを設定する

OAuth 認証プロバイダーは、クライアント ID とクライアントシークレットを使用して、アプリとの信頼関係を確立します。 アプリがプロバイダーに登録されるときに、外部認証プロバイダーによってアプリのクライアント ID とクライアントシークレットの値が作成されます。 アプリが使用する各外部プロバイダーは、プロバイダーのクライアント ID とクライアントシークレットとは別に構成する必要があります。 詳細については、シナリオに当てはまる外部認証プロバイダーのトピックを参照してください。

* [Facebook での認証](xref:security/authentication/facebook-logins)
* [Google での認証](xref:security/authentication/google-logins)
* [Microsoft での認証](xref:security/authentication/microsoft-logins)
* [Twitter での認証](xref:security/authentication/twitter-logins)
* [他の認証プロバイダー](xref:security/authentication/otherlogins)
* [OpenIdConnect](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2)

このサンプルアプリでは、google によって提供されるクライアント ID とクライアントシークレットを使用して Google 認証プロバイダーを構成します。

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=4,9)]

## <a name="establish-the-authentication-scope"></a>認証スコープを確立する

を指定して、プロバイダーから取得するアクセス許可の一覧を指定し <xref:Microsoft.AspNetCore.Authentication.OAuth.OAuthOptions.Scope*> ます。 共通外部プロバイダーの認証スコープは、次の表に表示されます。

| プロバイダー  | Scope                                                            |
| --------- | ---------------------------------------------------------------- |
| Facebook  | `https://www.facebook.com/dialog/oauth`                          |
| Google    | `https://www.googleapis.com/auth/userinfo.profile`               |
| Microsoft | `https://login.microsoftonline.com/common/oauth2/v2.0/authorize` |
| Twitter   | `https://api.twitter.com/oauth/authenticate`                     |

サンプルアプリで `userinfo.profile` <xref:Microsoft.Extensions.DependencyInjection.GoogleExtensions.AddGoogle*> は、がで呼び出されたときに、Google のスコープがフレームワークによって自動的に追加され <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilder> ます。 アプリに追加のスコープが必要な場合は、それらをオプションに追加します。 次の例で `https://www.googleapis.com/auth/user.birthday.read` は、ユーザーの誕生日を取得するために Google スコープが追加されています。

```csharp
options.Scope.Add("https://www.googleapis.com/auth/user.birthday.read");
```

## <a name="map-user-data-keys-and-create-claims"></a>ユーザーデータキーをマップして要求を作成する

プロバイダーのオプションで、 <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonKey*> <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonSubKey*> サインイン時に読み取るアプリ id の外部プロバイダーの JSON ユーザーデータのキー/サブキーごとに、またはを指定します。 要求の種類の詳細については、「」を参照してください <xref:System.Security.Claims.ClaimTypes> 。

このサンプルアプリでは、 `urn:google:locale` `urn:google:picture` `locale` Google ユーザーデータのキーとキーから、ロケール () と画像 () の要求を作成し `picture` ます。

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=13-14)]

では、 `Microsoft.AspNetCore.Identity.UI.Pages.Account.Internal.ExternalLoginModel.OnPostConfirmationAsync` <xref:Microsoft.AspNetCore.Identity.IdentityUser> ( `ApplicationUser` ) がを使用してアプリにサインイン <xref:Microsoft.AspNetCore.Identity.SignInManager%601.SignInAsync*> します。 サインインプロセス中に、で <xref:Microsoft.AspNetCore.Identity.UserManager%601> `ApplicationUser` 使用可能なユーザーデータの要求を格納でき <xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.Principal*> ます。

サンプルアプリでは、 `OnPostConfirmationAsync` (*Account/externallogin. cshtml*) によって、 `urn:google:locale` `urn:google:picture` `ApplicationUser` 次の要求を含む、サインインしたのロケール () と画像 () の要求が確立 <xref:System.Security.Claims.ClaimTypes.GivenName> されます。

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=35-51)]

既定では、ユーザーの要求は認証に格納され cookie ます。 認証 cookie が大きすぎると、次の理由により、アプリでエラーが発生する可能性があります。

* ブラウザーは、 cookie ヘッダーが長すぎることを検出します。
* 要求の全体的なサイズが大きすぎます。

ユーザーの要求を処理するために大量のユーザーデータが必要な場合は、次のようにします。

* 要求処理のユーザー要求の数とサイズは、アプリで必要なもののみに制限します。
* 認証ミドルウェアのカスタムを使用し <xref:Microsoft.AspNetCore.Authentication.Cookies.ITicketStore> て、 Cookie <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions.SessionStore> 要求間で id を格納します。 小さいセッション識別子キーをクライアントに送信するだけで、サーバー上の大量の id 情報を保持します。

## <a name="save-the-access-token"></a>アクセストークンを保存する

<xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.SaveTokens*> 承認が成功した後に、アクセストークンと更新トークンをに格納するかどうかを定義し <xref:Microsoft.AspNetCore.Http.Authentication.AuthenticationProperties> ます。 `SaveTokens` は `false` 、最終的な認証のサイズを小さくするために、既定でに設定され cookie ます。

サンプルアプリでは、の値をに設定し `SaveTokens` `true` <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions> ます。

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=15)]

を実行するときに、 `OnPostConfirmationAsync` の外部プロバイダーからアクセストークン ([Externallogininfo. authenticationtokens](xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.AuthenticationTokens*)) をに格納 `ApplicationUser` `AuthenticationProperties` します。

サンプルアプリでは、アクセストークン `OnPostConfirmationAsync` (新しいユーザー登録) と `OnGetCallbackAsync` (以前に登録されたユーザー) を *Account/externallogin. cshtml. cs* に保存します。

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=54-56)]

## <a name="how-to-add-additional-custom-tokens"></a>カスタムトークンを追加する方法

の一部として格納されているカスタムトークンを追加する方法を示すために、 `SaveTokens` サンプルアプリでは、の AuthenticationToken.Name の現在のを使用してを追加し <xref:Microsoft.AspNetCore.Authentication.AuthenticationToken> <xref:System.DateTime> [](xref:Microsoft.AspNetCore.Authentication.AuthenticationToken.Name*) `TicketCreated` ます。

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=17-30)]

## <a name="creating-and-adding-claims"></a>要求の作成と追加

フレームワークには、要求を作成してコレクションに追加するための一般的なアクションと拡張メソッドが用意されています。 詳細については、「 <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions> 」および「 <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionUniqueExtensions>」を参照してください。

ユーザーは、から派生し、抽象メソッドを実装することによって、カスタムアクションを定義でき <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction> <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.Run*> ます。

詳細については、「<xref:Microsoft.AspNetCore.Authentication.OAuth.Claims>」を参照してください。

## <a name="removal-of-claim-actions-and-claims"></a>要求アクションと要求の削除

[Claimactioncollection。 Remove (String)](xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimActionCollection.Remove*) は、指定されたのすべての要求アクションを <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> コレクションから削除します。 [DeleteClaim (ClaimActionCollection, String)](xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*) は、指定されたの要求を <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> id から削除します。 <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*> は、主に [OpenID connect (OIDC)](/azure/active-directory/develop/v2-protocols-oidc) と共に使用して、プロトコルによって生成された要求を削除します。

## <a name="sample-app-output"></a>サンプルアプリの出力

```
User Claims

http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier
    9b342344f-7aab-43c2-1ac1-ba75912ca999
http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name
    someone@gmail.com
AspNet.Identity.SecurityStamp
    7D4312MOWRYYBFI1KXRPHGOSTBVWSFDE
http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
    Judy
urn:google:locale
    en
urn:google:picture
    https://lh4.googleusercontent.com/-XXXXXX/XXXXXX/XXXXXX/XXXXXX/photo.jpg

Authentication Properties

.Token.access_token
    yc23.AlvoZqz56...1lxltXV7D-ZWP9
.Token.token_type
    Bearer
.Token.expires_at
    2019-04-11T22:14:51.0000000+00:00
.Token.TicketCreated
    4/11/2019 9:14:52 PM
.TokenNames
    access_token;token_type;expires_at;TicketCreated
.persistent
.issued
    Thu, 11 Apr 2019 20:51:06 GMT
.expires
    Thu, 25 Apr 2019 20:51:06 GMT

```

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

::: moniker-end

## <a name="additional-resources"></a>その他のリソース

* [dotnet/AspNetCore engineering の社会 Alsample アプリ](https://github.com/dotnet/AspNetCore/tree/master/src/Security/Authentication/samples/SocialSample): リンクされたサンプルアプリは、 [Dotnet/AspNetCore GitHub リポジトリの](https://github.com/dotnet/AspNetCore) `master` エンジニアリングブランチにあります。 ブランチには、 `master` ASP.NET Core の次のリリースでアクティブな開発のコードが含まれています。 リリースされたバージョンの ASP.NET Core 用のサンプルアプリのバージョンを表示するには、[ **ブランチ** ] ドロップダウンリストを使用してリリースブランチを選択します (たとえば、 `release/{X.Y}` )。
