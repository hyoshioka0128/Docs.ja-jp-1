---
title: ASP.NET Core で特定のスキームを使用して承認する
author: rick-anderson
description: この記事では、複数の認証方法を使用する場合に、特定のスキームに id を制限する方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 11/08/2019
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
uid: security/authorization/limitingidentitybyscheme
ms.openlocfilehash: c4cbec1b829fb8fd47f7b6924b6870bd5dd7097d
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057305"
---
# <a name="authorize-with-a-specific-scheme-in-aspnet-core"></a>ASP.NET Core で特定のスキームを使用して承認する

ASP.NET Core での認証スキームの概要については、「 [認証スキーム](xref:security/authentication/index#authentication-scheme)」を参照してください。

シングルページアプリケーション (spa) などの一部のシナリオでは、複数の認証方法を使用するのが一般的です。 たとえば、アプリはベース cookie 認証を使用して、JavaScript 要求のログインと JWT ベアラー認証を使用する場合があります。 場合によっては、アプリに認証ハンドラーのインスタンスが複数存在することがあります。 たとえば、1つのに基本 id が格納されている2つの cookie ハンドラーと、multi-factor authentication (MFA) がトリガーされたときに作成されるハンドラーがあります。 ユーザーが追加のセキュリティを必要とする操作を要求したため、MFA がトリガーされる可能性があります。 ユーザーが MFA を必要とするリソースを要求したときに MFA を適用する方法の詳細については、「MFA を使用した GitHub の問題の [保護」セクション](https://github.com/dotnet/AspNetCore.Docs/issues/15791#issuecomment-580464195)を参照してください。

認証時に認証サービスが構成されると、認証スキームに名前が付けられます。 次に例を示します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Code omitted for brevity

    services.AddAuthentication()
        .AddCookie(options => {
            options.LoginPath = "/Account/Unauthorized/";
            options.AccessDeniedPath = "/Account/Forbidden/";
        })
        .AddJwtBearer(options => {
            options.Audience = "http://localhost:5001/";
            options.Authority = "http://localhost:5000/";
        });
```

上記のコードでは、2つの認証ハンドラーが追加されています。1つは cookie s、もう1つはベアラー用です。

>[!NOTE]
>既定のスキームを指定すると、 `HttpContext.User` プロパティはその id に設定されます。 この動作が望ましくない場合は、のパラメーターなしの形式を呼び出して無効に `AddAuthentication` します。

## <a name="selecting-the-scheme-with-the-authorize-attribute"></a>認証属性を使用したスキームの選択

承認の時点で、アプリは使用するハンドラーを示します。 認証スキームのコンマ区切りの一覧をに渡して、アプリが承認するハンドラーを選択し `[Authorize]` ます。 属性は、 `[Authorize]` 既定値が構成されているかどうかに関係なく、使用する認証スキームを指定します。 次に例を示します。

```csharp
[Authorize(AuthenticationSchemes = AuthSchemes)]
public class MixedController : Controller
    // Requires the following imports:
    // using Microsoft.AspNetCore.Authentication.Cookies;
    // using Microsoft.AspNetCore.Authentication.JwtBearer;
    private const string AuthSchemes =
        CookieAuthenticationDefaults.AuthenticationScheme + "," +
        JwtBearerDefaults.AuthenticationScheme;
```

前の例では、 cookie およびベアラーハンドラーの両方が実行され、現在のユーザーの id を作成して追加する機会があります。 1つのスキームのみを指定すると、対応するハンドラーが実行されます。

```csharp
[Authorize(AuthenticationSchemes = 
    JwtBearerDefaults.AuthenticationScheme)]
public class MixedController : Controller
```

前のコードでは、"ベアラー" スキームを持つハンドラーだけが実行されます。 cookieベースの id は無視されます。

## <a name="selecting-the-scheme-with-policies"></a>ポリシーを使用したスキームの選択

[ポリシー](xref:security/authorization/policies)で目的のスキームを指定する場合は、ポリシーを追加するときにコレクションを設定でき `AuthenticationSchemes` ます。

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy("Over18", policy =>
    {
        policy.AuthenticationSchemes.Add(JwtBearerDefaults.AuthenticationScheme);
        policy.RequireAuthenticatedUser();
        policy.Requirements.Add(new MinimumAgeRequirement());
    });
});
```

前の例では、"Over18" ポリシーは "ベアラー" ハンドラーによって作成された id に対してのみ実行されます。 属性のプロパティを設定して、ポリシーを使用し `[Authorize]` `Policy` ます。

```csharp
[Authorize(Policy = "Over18")]
public class RegistrationController : Controller
```

::: moniker range=">= aspnetcore-2.0"

## <a name="use-multiple-authentication-schemes"></a>複数の認証方式を使用する

アプリによっては、複数の種類の認証をサポートする必要がある場合があります。 たとえば、アプリは Azure Active Directory とユーザーデータベースからユーザーを認証する場合があります。 もう1つの例として、Active Directory フェデレーションサービス (AD FS) と Azure Active Directory B2C の両方からユーザーを認証するアプリがあります。 この場合、アプリはいくつかの発行者から JWT ベアラートークンを受け入れる必要があります。

同意するすべての認証スキームを追加します。 たとえば、の次のコードは、 `Startup.ConfigureServices` 発行者が異なる2つの JWT ベアラー認証スキームを追加します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Code omitted for brevity

    services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
        .AddJwtBearer(options =>
        {
            options.Audience = "https://localhost:5000/";
            options.Authority = "https://localhost:5000/identity/";
        })
        .AddJwtBearer("AzureAD", options =>
        {
            options.Audience = "https://localhost:5000/";
            options.Authority = "https://login.microsoftonline.com/eb971100-6f99-4bdc-8611-1bc8edd7f436/";
        });
}
```

> [!NOTE]
> 既定の認証スキームでは、JWT ベアラー認証が1つだけ登録され `JwtBearerDefaults.AuthenticationScheme` ます。 追加の認証は、一意の認証スキームを使用して登録する必要があります。

次の手順では、両方の認証方式を受け入れるように既定の承認ポリシーを更新します。 次に例を示します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Code omitted for brevity

    services.AddAuthorization(options =>
    {
        var defaultAuthorizationPolicyBuilder = new AuthorizationPolicyBuilder(
            JwtBearerDefaults.AuthenticationScheme,
            "AzureAD");
        defaultAuthorizationPolicyBuilder = 
            defaultAuthorizationPolicyBuilder.RequireAuthenticatedUser();
        options.DefaultPolicy = defaultAuthorizationPolicyBuilder.Build();
    });
}
```

既定の承認ポリシーがオーバーライドされると、コントローラーで属性を使用できるようになり `[Authorize]` ます。 コントローラーは、最初または2番目の発行者によって発行された JWT の要求を受け入れます。

::: moniker-end

複数の認証方式の使用については、 [この GitHub の問題](https://github.com/dotnet/aspnetcore/issues/26002) を参照してください。
