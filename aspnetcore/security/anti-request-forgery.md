---
title: ASP.NET Core でのクロスサイト要求偽造 (XSRF/CSRF) 攻撃を防ぐ
author: steve-smith
description: 悪意のある web サイトがクライアントブラウザーとアプリの間の対話に影響を与える可能性がある web アプリに対する攻撃を防ぐ方法について説明します。
ms.author: riande
ms.custom: mvc, devx-track-js
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
uid: security/anti-request-forgery
ms.openlocfilehash: 5d6f2915dd9b27142ac7d8ac55e68c6a26e41f81
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102585787"
---
# <a name="prevent-cross-site-request-forgery-xsrfcsrf-attacks-in-aspnet-core"></a>ASP.NET Core でのクロスサイト要求偽造 (XSRF/CSRF) 攻撃を防ぐ

[Rick Anderson](https://twitter.com/RickAndMSFT)、 [Fiyaz Hasan](https://twitter.com/FiyazBinHasan)、[上田 Smith](https://ardalis.com/)

クロスサイト要求偽造 (XSRF または CSRF とも呼ばれます) は、web ホストアプリに対する攻撃であり、悪意のある web アプリがクライアントブラウザーとそのブラウザーを信頼する web アプリとの間の対話に影響を与える可能性があります。 これらの攻撃は、web ブラウザーがすべての要求を web サイトに自動的に送信するために発生する可能性があります。 この形式の悪用は、 *ワンクリック攻撃* または *セッション* によっても知られています。これは、攻撃がユーザーの以前に認証されたセッションを利用しているためです。

CSRF 攻撃の例を次に示します。

1. ユーザーは、 `www.good-banking-site.com` フォーム認証を使用してサインインします。 サーバーはユーザーを認証し、認証を含む応答を発行し cookie ます。 サイトは、有効な認証を使用して受信したすべての要求を信頼しているため、攻撃に対して脆弱です cookie 。
1. ユーザーが悪意のあるサイト () にアクセスして `www.bad-crook-site.com` います。

   悪意のあるサイトには、 `www.bad-crook-site.com` 次のような HTML フォームが含まれています。

   ```html
   <h1>Congratulations! You're a Winner!</h1>
   <form action="http://good-banking-site.com/api/account" method="post">
       <input type="hidden" name="Transaction" value="withdraw">
       <input type="hidden" name="Amount" value="1000000">
       <input type="submit" value="Click to collect your prize!">
   </form>
   ```

   悪意のあるサイトではなく、脆弱なサイトにフォームが投稿されていることに注意して `action` ください。 これは CSRF の "クロスサイト" 部分です。

1. ユーザーは [送信] ボタンを選択します。 ブラウザーは要求を行い、要求されたドメインの認証を自動的に含め cookie `www.good-banking-site.com` ます。
1. 要求は、 `www.good-banking-site.com` ユーザーの認証コンテキストを使用してサーバー上で実行され、認証されたユーザーが実行を許可されている任意のアクションを実行できます。

フォームを送信するためにユーザーがボタンを選択するシナリオに加えて、悪意のあるサイトも考えられます。

* フォームを自動的に送信するスクリプトを実行します。
* フォーム送信を AJAX 要求として送信します。
* CSS を使用してフォームを非表示にします。

これらの代替シナリオでは、最初に悪意のあるサイトにアクセスしていないユーザーからの操作や入力は必要ありません。

HTTPS を使用しても CSRF 攻撃を防ぐことはできません。 悪意のあるサイトは、 `https://www.good-banking-site.com/` セキュリティで保護されていない要求を送信するのと同じように簡単に要求を送信できます。

一部の攻撃では、GET 要求に応答するエンドポイントを対象にしています。この場合、イメージタグを使用してアクションを実行できます。 この形式の攻撃は、イメージを許可し、JavaScript をブロックするフォーラムサイトで共通です。 GET 要求の状態を変更するアプリ (変数またはリソースが変更された場合) は、悪意のある攻撃に対して脆弱です。 **状態を変更する GET 要求は安全ではありません。GET 要求の状態を変更しないことをお勧めします。**

cookie次の理由により、認証にを使用する web アプリに対して CSRF 攻撃が行われる可能性があります。

* cookieWeb アプリによって発行されたのブラウザーストア。
* 保存された cookie には、認証されたユーザーのセッションが含まれ cookie ます。
* cookieブラウザーでは、アプリへの要求がブラウザー内でどのように生成されたかに関係なく、ドメインに関連付けられているすべてのを要求ごとに web アプリに送信します。

ただし、CSRF 攻撃は、を利用することに限定されません cookie 。 たとえば、基本認証とダイジェスト認証も脆弱です。 ユーザーが基本認証またはダイジェスト認証を使用してサインインすると、セッションが終了するまで、ブラウザーは資格情報を自動的に送信し &dagger; ます。

&dagger;このコンテキストでは、 *セッション* とは、ユーザーが認証されるクライアント側のセッションを指します。 サーバー側セッションや [ASP.NET Core セッションミドルウェア](xref:fundamentals/app-state)とは無関係です。

ユーザーは、次のように予防措置を講じて CSRF 脆弱性を防ぐことができます。

* Web apps の使用が完了したら、サインオフします。
* ブラウザー cookie を定期的にクリアします。

ただし、CSRF 脆弱性は、エンドユーザーではなく、根本的に web アプリに問題があります。

## <a name="authentication-fundamentals"></a>認証の基礎

Cookieベースの認証は、一般的な認証形式です。 特にシングルページアプリケーション (spa) では、トークンベースの認証システムの普及が高まっています。

### <a name="cookie-based-authentication"></a>Cookieベースの認証

ユーザー名とパスワードを使用して認証を行うと、認証と承認に使用できる認証チケットを含むトークンが発行されます。 トークンは、 cookie クライアントが行うすべての要求に付随するとして格納されます。 これ cookie は、認証ミドルウェアによって生成および検証され Cookie ます。 [ミドルウェア](xref:fundamentals/middleware/index)は、ユーザープリンシパルを暗号化されたにシリアル化し cookie ます。 後続の要求では、ミドルウェアはを検証し、 cookie プリンシパルを再作成して、そのプリンシパルを[HttpContext](/dotnet/api/microsoft.aspnetcore.http.httpcontext)の[User](/dotnet/api/microsoft.aspnetcore.http.httpcontext.user)プロパティに割り当てます。

### <a name="token-based-authentication"></a>トークンベースの認証

ユーザーが認証されると、トークンが発行されます (偽造防止トークンではありません)。 このトークンには、 [要求](/dotnet/framework/security/claims-based-identity-model) の形式でユーザー情報が含まれているか、アプリがアプリで保持されているユーザー状態を示す参照トークンが含まれています。 ユーザーが認証を必要とするリソースにアクセスしようとすると、トークンは、ベアラートークンの形式で追加の authorization ヘッダーを使用してアプリに送信されます。 これにより、アプリはステートレスになります。 後続の各要求では、サーバー側の検証要求でトークンが渡されます。 このトークンは *暗号化さ* れていません。これは *エンコード* されます。 サーバーでは、トークンをデコードして情報にアクセスします。 後続の要求でトークンを送信するには、ブラウザーのローカルストレージにトークンを格納します。 トークンがブラウザーのローカルストレージに格納されている場合、CSRF の脆弱性について心配しないでください。 CSRF は、トークンがに格納されている場合に問題になり cookie ます。 詳細については、GitHub の issue [SPA のコードサンプルを cookie 参照して](https://github.com/dotnet/AspNetCore.Docs/issues/13369)ください。

### <a name="multiple-apps-hosted-at-one-domain"></a>1つのドメインでホストされている複数のアプリ

共有ホスティング環境は、セッションハイジャック、ログイン CSRF、およびその他の攻撃に対して脆弱です。

`example1.contoso.net`と `example2.contoso.net` は異なるホストですが、ドメイン内のホスト間には暗黙的な信頼関係があり `*.contoso.net` ます。 この暗黙的な信頼関係により、信頼できない可能性のあるホストが互いのに影響を与えることがあり cookie ます (AJAX 要求を管理する同じオリジンポリシーは必ずしも HTTP s に適用されるわけではありません cookie )。

同じドメインでホストされているアプリ間で信頼されたを利用する攻撃は cookie 、ドメインを共有しないことで回避できます。 各アプリが独自のドメインでホストされている場合、 cookie 悪用に対する暗黙の信頼関係はありません。

## <a name="aspnet-core-antiforgery-configuration"></a>ASP.NET Core 偽造防止の構成

> [!WARNING]
> ASP.NET Core は [ASP.NET Core データ保護](xref:security/data-protection/introduction)を使用してアンチ偽造を実装します。 データ保護スタックは、サーバーファームで動作するように構成されている必要があります。 詳細については、「 [データ保護の構成](xref:security/data-protection/configuration/overview) 」を参照してください。

::: moniker range=">= aspnetcore-3.0"

次のいずれかの Api がで呼び出されると、アンチ偽造ミドルウェアが [依存関係の注入](xref:fundamentals/dependency-injection) コンテナーに追加され `Startup.ConfigureServices` ます。

* <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddMvc*>
* <xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapRazorPages*>
* <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute*>
* <xref:Microsoft.AspNetCore.Builder.ComponentEndpointRouteBuilderExtensions.MapBlazorHub*>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

が呼び出されると、アンチ偽造ミドルウェアが[依存関係挿入](xref:fundamentals/dependency-injection)コンテナーに追加されます。 <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddMvc*>`Startup.ConfigureServices`

::: moniker-end

ASP.NET Core 2.0 以降では、 [Formtaghelper](xref:mvc/views/working-with-forms#the-form-tag-helper) は、アンチ偽造トークンを HTML フォーム要素に挿入します。 ファイル内の次のマークアップは、 Razor アンチ偽造トークンを自動的に生成します。

```cshtml
<form method="post">
    ...
</form>
```

同様に、フォームのメソッドが GET でない場合、 [Ihtmlhelper. BeginForm](/dotnet/api/microsoft.aspnetcore.mvc.rendering.ihtmlhelper.beginform) は既定で偽造防止トークンを生成します。

HTML フォーム要素のアンチ偽造トークンの自動生成は、タグに `<form>` 属性が含まれて `method="post"` おり、次のいずれかが当てはまる場合に発生します。

* Action 属性が空です ( `action=""` )。
* Action 属性が指定されていません ( `<form method="post">` )。

HTML フォーム要素のアンチ偽造トークンの自動生成は無効にすることができます。

* 属性を使用してアンチ偽造トークンを明示的に無効にし `asp-antiforgery` ます。

  ```cshtml
  <form method="post" asp-antiforgery="false">
      ...
  </form>
  ```

* Form 要素は、タグヘルパー [! オプトアウトシンボル](xref:mvc/views/tag-helpers/intro#opt-out)を使用してタグヘルパーをオプトアウトします。

  ```cshtml
  <!form method="post">
      ...
  </!form>
  ```

* ビューからを削除し `FormTagHelper` ます。 ビュー `FormTagHelper` に次のディレクティブを追加することで、ビューからを削除でき Razor ます。

  ```cshtml
  @removeTagHelper Microsoft.AspNetCore.Mvc.TagHelpers.FormTagHelper, Microsoft.AspNetCore.Mvc.TagHelpers
  ```

> [!NOTE]
> [ Razor ページ](xref:razor-pages/index)は XSRF/csrf から自動的に保護されます。 詳細については、「 [XSRF/CSRF と Razor Pages](xref:razor-pages/index#xsrf)」を参照してください。

CSRF 攻撃を防ぐための最も一般的な方法は、 *シンクロナイザートークンパターン* (STP) を使用することです。 STP は、ユーザーがフォームデータを使用してページを要求したときに使用されます。

1. サーバーは、現在のユーザーの id に関連付けられているトークンをクライアントに送信します。
1. クライアントは、トークンを検証のためにサーバーに送り返します。
1. サーバーが、認証されたユーザーの id と一致しないトークンを受け取った場合、要求は拒否されます。

トークンは一意で、予測できません。 このトークンは、一連の要求を適切にシーケンス処理するためにも使用できます (たとえば、ページ 1 > ページ 2 > ページ 3)。 ASP.NET Core MVC およびページテンプレートのすべてのフォームで、 Razor 偽造防止トークンが生成されます。 次の2つのビュー例では、偽造偽造トークンが生成されます。

```cshtml
<form asp-controller="Manage" asp-action="ChangePassword" method="post">
    ...
</form>

@using (Html.BeginForm("ChangePassword", "Manage"))
{
    ...
}
```

`<form>`タグヘルパーと HTML ヘルパーを使用せずに、次のように、アンチ偽造トークンを要素に明示的に追加し [`@Html.AntiForgeryToken`](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.htmlhelper.antiforgerytoken) ます。

```cshtml
<form action="/" method="post">
    @Html.AntiForgeryToken()
</form>
```

これらの各ケースでは、次のような非表示のフォームフィールドが ASP.NET Core によって追加されます。

```cshtml
<input name="__RequestVerificationToken" type="hidden" value="CfDJ8NrAkS ... s2-m9Yw">
```

ASP.NET Core には、偽造防止トークンを操作するための3つの [フィルター](xref:mvc/controllers/filters) が含まれています。

* [ValidateAntiForgeryToken](/dotnet/api/microsoft.aspnetcore.mvc.validateantiforgerytokenattribute)
* [Autovalidateアンチ Forgerytoken](/dotnet/api/microsoft.aspnetcore.mvc.autovalidateantiforgerytokenattribute)
* [Ignoreアンチ Forgerytoken](/dotnet/api/microsoft.aspnetcore.mvc.ignoreantiforgerytokenattribute)

## <a name="antiforgery-options"></a>偽造防止オプション

次の方法で [偽造防止オプション](/dotnet/api/Microsoft.AspNetCore.Antiforgery.AntiforgeryOptions) をカスタマイズし `Startup.ConfigureServices` ます。

::: moniker range=">= aspnetcore-2.0"

```csharp
services.AddAntiforgery(options => 
{
    // Set Cookie properties using CookieBuilder properties†.
    options.FormFieldName = "AntiforgeryFieldname";
    options.HeaderName = "X-CSRF-TOKEN-HEADERNAME";
    options.SuppressXFrameOptionsHeader = false;
});
```

&dagger;`Cookie` [ Cookie ビルダー](/dotnet/api/microsoft.aspnetcore.http.cookiebuilder)クラスのプロパティを使用して、アンチ偽造プロパティを設定します。

| オプション | 説明 |
| ------ | ----------- |
| [Cookie](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.cookie) | アンチ偽造を作成するために使用する設定を決定し cookie ます。 |
| [FormFieldName](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.formfieldname) | アンチ偽造システムがビューで偽造防止トークンをレンダリングするために使用する非表示フォームフィールドの名前。 |
| [HeaderName](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.headername) | アンチ偽造システムによって使用されるヘッダーの名前。 の場合 `null` 、システムはフォームデータのみを考慮します。 |
| [SuppressXFrameOptionsHeader](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.suppressxframeoptionsheader) | ヘッダーの生成を抑制するかどうかを指定し `X-Frame-Options` ます。 既定では、ヘッダーは値 "SAMEORIGIN" を使用して生成されます。 既定値は `false` です。 |

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
services.AddAntiforgery(options => 
{
    options.CookieDomain = "contoso.com";
    options.CookieName = "X-CSRF-TOKEN-COOKIENAME";
    options.CookiePath = "Path";
    options.FormFieldName = "AntiforgeryFieldname";
    options.HeaderName = "X-CSRF-TOKEN-HEADERNAME";
    options.RequireSsl = false;
    options.SuppressXFrameOptionsHeader = false;
});
```

| オプション | 説明 |
| ------ | ----------- |
| [Cookie](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.cookie) | アンチ偽造を作成するために使用する設定を決定し cookie ます。 |
| [CookieDomain](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.cookiedomain) | のドメイン cookie 。 既定値は `null` です。 このプロパティは互換性のために残されていますが、今後のバージョンでは削除される予定です。 別の方法をお勧めし Cookie ます。領域. |
| [Cookie名前](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.cookiename) | cookie の名前。 設定しない場合、システムは [既定の Cookie プレフィックス](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.defaultcookieprefix) ("を使用して、一意の名前を生成します。AspNetCore。 ")。 このプロパティは互換性のために残されていますが、今後のバージョンでは削除される予定です。 別の方法をお勧めし Cookie ます。指定. |
| [Cookieパス](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.cookiepath) | に設定されたパス cookie 。 このプロパティは互換性のために残されていますが、今後のバージョンでは削除される予定です。 別の方法をお勧めし Cookie ます。道. |
| [FormFieldName](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.formfieldname) | アンチ偽造システムがビューで偽造防止トークンをレンダリングするために使用する非表示フォームフィールドの名前。 |
| [HeaderName](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.headername) | アンチ偽造システムによって使用されるヘッダーの名前。 の場合 `null` 、システムはフォームデータのみを考慮します。 |
| [RequireSsl](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.requiressl) | 偽造防止システムによって HTTPS が要求されるかどうかを指定します。 `true`の場合、HTTPS 以外の要求は失敗します。 既定値は `false` です。 このプロパティは互換性のために残されていますが、今後のバージョンでは削除される予定です。 別の方法として、を設定することをお勧めし Cookie ます。SecurePolicy。 |
| [SuppressXFrameOptionsHeader](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.suppressxframeoptionsheader) | ヘッダーの生成を抑制するかどうかを指定し `X-Frame-Options` ます。 既定では、ヘッダーは値 "SAMEORIGIN" を使用して生成されます。 既定値は `false` です。 |

::: moniker-end

詳細については、「 [ Cookie authenticationoptions](/dotnet/api/Microsoft.AspNetCore.Builder.CookieAuthenticationOptions)」を参照してください。

## <a name="configure-antiforgery-features-with-iantiforgery"></a>偽造防止機能の構成 (Iアンチ偽造)

[Iアンチ偽造](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgery) は、偽造防止機能を構成するための API を提供します。 `IAntiforgery` は `Configure` 、クラスのメソッドで要求でき `Startup` ます。 次の例では、アプリのホームページからミドルウェアを使用して、アンチ偽造トークンを生成し、として応答に送信し cookie ます (このトピックで後述する既定の角度の名前付け規則を使用します)。

```csharp
public void Configure(IApplicationBuilder app, IAntiforgery antiforgery)
{
    app.Use(next => context =>
    {
        string path = context.Request.Path.Value;

        if (
            string.Equals(path, "/", StringComparison.OrdinalIgnoreCase) ||
            string.Equals(path, "/index.html", StringComparison.OrdinalIgnoreCase))
        {
            // The request token can be sent as a JavaScript-readable cookie, 
            // and Angular uses it by default.
            var tokens = antiforgery.GetAndStoreTokens(context);
            context.Response.Cookies.Append("XSRF-TOKEN", tokens.RequestToken, 
                new CookieOptions() { HttpOnly = false });
        }

        return next(context);
    });
}
```

### <a name="require-antiforgery-validation"></a>偽造防止の検証を必要とする

[Validateアンチ Forgerytoken](/dotnet/api/microsoft.aspnetcore.mvc.validateantiforgerytokenattribute) は、個々のアクション、コントローラー、またはグローバルに適用できるアクションフィルターです。 このフィルターが適用されているアクションに対して行われた要求は、要求に有効な偽造偽造トークンが含まれていない限り、ブロックされます。

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> RemoveLogin(RemoveLoginViewModel account)
{
    ManageMessageId? message = ManageMessageId.Error;
    var user = await GetCurrentUserAsync();

    if (user != null)
    {
        var result = 
            await _userManager.RemoveLoginAsync(
                user, account.LoginProvider, account.ProviderKey);

        if (result.Succeeded)
        {
            await _signInManager.SignInAsync(user, isPersistent: false);
            message = ManageMessageId.RemoveLoginSuccess;
        }
    }

    return RedirectToAction(nameof(ManageLogins), new { Message = message });
}
```

属性には、 `ValidateAntiForgeryToken` HTTP GET 要求を含む、マークするアクションメソッドへの要求にトークンが必要です。 `ValidateAntiForgeryToken`属性がアプリのコントローラー全体に適用される場合は、属性でオーバーライドでき `IgnoreAntiforgeryToken` ます。

> [!NOTE]
> ASP.NET Core は、要求を自動的に取得するための偽造防止トークンの追加をサポートしていません。

### <a name="automatically-validate-antiforgery-tokens-for-unsafe-http-methods-only"></a>安全でない HTTP メソッドに対してのみ偽造防止トークンを自動的に検証する

ASP.NET Core アプリは、安全な HTTP メソッド (GET、HEAD、OPTIONS、および TRACE) に対して偽造防止トークンを生成しません。 属性を広範に適用して属性でオーバーライドするのではなく `ValidateAntiForgeryToken` `IgnoreAntiforgeryToken` 、 [Autovalidateアンチ Forgerytoken](/dotnet/api/microsoft.aspnetcore.mvc.autovalidateantiforgerytokenattribute) 属性を使用できます。 この属性は、属性と同じように動作し `ValidateAntiForgeryToken` ますが、次の HTTP メソッドを使用して行われた要求に対してトークンを必要としない点が異なります。

* GET
* HEAD
* OPTIONS
* TRACE

非 API のシナリオでは、広範なを使用することをお勧め `AutoValidateAntiforgeryToken` します。 これにより、POST アクションが既定で保護されるようになります。 代替手段は、 `ValidateAntiForgeryToken` が個々のアクションメソッドに適用されない限り、既定で偽造防止トークンを無視することです。 このシナリオでは、POST アクションメソッドが誤って保護されないままになり、アプリが CSRF 攻撃に対して脆弱になる可能性が高くなります。 すべての投稿は、アンチ偽造トークンを送信する必要があります。

Api には、トークンの一部を送信するための自動メカニズムがありません cookie 。 実装は、クライアントコードの実装によって異なる場合があります。 いくつかの例を次に示します。

クラスレベルの例:

```csharp
[Authorize]
[AutoValidateAntiforgeryToken]
public class ManageController : Controller
{
```

グローバルな例:

::: moniker range="< aspnetcore-3.0"

サーヴィス.AddMvc (オプション => オプション。Filters。 Add (新しい Autovalidateアンチ Forgerytokenattribute ());

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

```csharp
services.AddControllersWithViews(options =>
    options.Filters.Add(new AutoValidateAntiforgeryTokenAttribute()));
```

::: moniker-end

### <a name="override-global-or-controller-antiforgery-attributes"></a>グローバルまたはコントローラーのアンチ偽造属性を上書きする

[Ignoreアンチ Forgerytoken](/dotnet/api/microsoft.aspnetcore.mvc.ignoreantiforgerytokenattribute)フィルターは、特定のアクション (またはコントローラー) に対して偽造防止トークンが不要になるようにするために使用されます。 このフィルターを適用すると `ValidateAntiForgeryToken` 、 `AutoValidateAntiforgeryToken` より高いレベルで指定されたフィルター (グローバルまたはコントローラー上) が上書きされます。

```csharp
[Authorize]
[AutoValidateAntiforgeryToken]
public class ManageController : Controller
{
    [HttpPost]
    [IgnoreAntiforgeryToken]
    public async Task<IActionResult> DoSomethingSafe(SomeViewModel model)
    {
        // no antiforgery token required
    }
}
```

## <a name="refresh-tokens-after-authentication"></a>認証後のトークンの更新

ユーザーを認証した後で、ユーザーをビューページまたはページページにリダイレクトすることによって、トークンを更新する必要があり Razor ます。

## <a name="javascript-ajax-and-spas"></a>JavaScript、AJAX、および SPAs

従来の HTML ベースのアプリでは、非表示のフォームフィールドを使用して、偽造防止トークンがサーバーに渡されます。 最新の JavaScript ベースのアプリと SPAs では、多くの要求がプログラムによって行われます。 これらの AJAX 要求では、他の手法 (要求ヘッダー cookie 、s など) を使用してトークンを送信できます。

cookieが認証トークンの格納に使用され、サーバーで API 要求を認証するために、CSRF が発生する可能性があります。 ローカルストレージを使用してトークンを格納している場合は、ローカルストレージからの値がすべての要求でサーバーに自動的に送信されないため、CSRF 脆弱性が軽減される可能性があります。 したがって、ローカルストレージを使用して、クライアントにアンチ偽造トークンを格納し、要求ヘッダーとしてトークンを送信することをお勧めします。

### <a name="javascript"></a>JavaScript

ビューで JavaScript を使用すると、ビュー内からサービスを使用してトークンを作成できます。 ビューに [AspNetCore の偽造](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgery) サービスを挿入し、 [Getandstoretokens](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgery.getandstoretokens)を呼び出します。

[!code-cshtml[](anti-request-forgery/sample/MvcSample/Views/Home/Ajax.cshtml?highlight=4-10,12-13,35-36)]

この方法を使用すると、サーバーからのの設定やクライアントからの読み取りを直接処理する必要がなくなり cookie ます。

前の例では、JavaScript を使用して、AJAX POST ヘッダーの隠しフィールド値を読み取ります。

また、JavaScript は、のトークンにアクセス cookie し、そのコンテンツを使用して、 cookie トークンの値を持つヘッダーを作成することもできます。

```csharp
context.Response.Cookies.Append("CSRF-TOKEN", tokens.RequestToken, 
    new Microsoft.AspNetCore.Http.CookieOptions { HttpOnly = false });
```

スクリプトがというヘッダー内のトークンを送信するように要求した `X-CSRF-TOKEN` 場合、そのヘッダーを検索するようにアンチ偽造サービスを構成し `X-CSRF-TOKEN` ます。

```csharp
services.AddAntiforgery(options => options.HeaderName = "X-CSRF-TOKEN");
```

次の例では、JavaScript を使用して、適切なヘッダーを含む AJAX 要求を作成します。

```javascript
function getCookie(cname) {
    var name = cname + "=";
    var decodedCookie = decodeURIComponent(document.cookie);
    var ca = decodedCookie.split(';');
    for(var i = 0; i <ca.length; i++) {
        var c = ca[i];
        while (c.charAt(0) == ' ') {
            c = c.substring(1);
        }
        if (c.indexOf(name) == 0) {
            return c.substring(name.length, c.length);
        }
    }
    return "";
}

var csrfToken = getCookie("CSRF-TOKEN");

var xhttp = new XMLHttpRequest();
xhttp.onreadystatechange = function() {
    if (xhttp.readyState == XMLHttpRequest.DONE) {
        if (xhttp.status == 200) {
            alert(xhttp.responseText);
        } else {
            alert('There was an error processing the AJAX request.');
        }
    }
};
xhttp.open('POST', '/api/password/changepassword', true);
xhttp.setRequestHeader("Content-type", "application/json");
xhttp.setRequestHeader("X-CSRF-TOKEN", csrfToken);
xhttp.send(JSON.stringify({ "newPassword": "ReallySecurePassword999$$$" }));
```

### <a name="angularjs"></a>AngularJS

AngularJS は、規約を使用して CSRF に対処します。 サーバーがという名前のを送信した場合 cookie `XSRF-TOKEN` 、AngularJS サービスは、 `$http` cookie サーバーに要求を送信するときに、ヘッダーに値を追加します。 このプロセスは自動的に実行されます。 ヘッダーは、クライアントで明示的に設定する必要はありません。 ヘッダー名が `X-XSRF-TOKEN` です。 サーバーは、このヘッダーを検出し、その内容を検証する必要があります。

アプリケーションの起動時に、ASP.NET Core API がこの規則を使用するようにするには、次の操作を行います。

* という名前のでトークンを提供するようにアプリを構成 cookie `XSRF-TOKEN` します。
* という名前のヘッダーを検索するようにアンチ偽造サービスを構成 `X-XSRF-TOKEN` します。

```csharp
public void Configure(IApplicationBuilder app, IAntiforgery antiforgery)
{
    app.Use(next => context =>
    {
        string path = context.Request.Path.Value;

        if (
            string.Equals(path, "/", StringComparison.OrdinalIgnoreCase) ||
            string.Equals(path, "/index.html", StringComparison.OrdinalIgnoreCase))
        {
            // The request token can be sent as a JavaScript-readable cookie, 
            // and Angular uses it by default.
            var tokens = antiforgery.GetAndStoreTokens(context);
            context.Response.Cookies.Append("XSRF-TOKEN", tokens.RequestToken, 
                new CookieOptions() { HttpOnly = false });
        }

        return next(context);
    });
}

public void ConfigureServices(IServiceCollection services)
{
    // Angular's default header name for sending the XSRF token.
    services.AddAntiforgery(options => options.HeaderName = "X-XSRF-TOKEN");
}
```

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/anti-request-forgery/sample/AngularSample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="windows-authentication-and-antiforgery-cookies"></a>Windows 認証 cookie と偽造防止

Windows 認証を使用する場合、アプリケーションエンドポイントは、の場合と同じように CSRF 攻撃に対して保護する必要があり cookie ます。  ブラウザーは認証コンテキストをサーバーに暗黙的に送信するため、エンドポイントは CSRF 攻撃に対して保護する必要があります。

## <a name="extend-antiforgery"></a>アンチ偽造の延長

[Iアンチ Forgeryadditionaldataprovider](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgeryadditionaldataprovider)型を使用すると、開発者は各トークンの追加データをラウンドトリップさせることで、csrf システムの動作を拡張できます。 [Getadditionaldata](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgeryadditionaldataprovider.getadditionaldata)メソッドは、フィールドトークンが生成されるたびに呼び出され、戻り値は生成されたトークン内に埋め込まれます。 実装者は、タイムスタンプ、nonce、またはその他の値を返し、 [Validateadditionaldata](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgeryadditionaldataprovider.validateadditionaldata) を呼び出して、トークンが検証されたときにこのデータを検証することができます。 クライアントのユーザー名は、生成されたトークンに既に埋め込まれているため、この情報を含める必要はありません。 トークンに追加データが含まれていても構成されていない場合 `IAntiForgeryAdditionalDataProvider` 、補足データは検証されません。

## <a name="additional-resources"></a>その他のリソース

* [Open Web Application Security Project](https://www.owasp.org/index.php/Main_Page) (owasp) での[csrf](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)) 。
* <xref:host-and-deploy/web-farm>
