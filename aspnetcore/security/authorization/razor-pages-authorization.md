---
title: Razor ASP.NET Core でのページ承認規則
author: rick-anderson
description: ユーザーを承認し、匿名ユーザーがページまたはページのフォルダーにアクセスすることを許可する規則を使用して、ページへのアクセスを制御する方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/12/2019
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
uid: security/authorization/razor-pages-authorization
ms.openlocfilehash: ffcb16b626773da69c45b8ab5dbd7c3cdc84bb5f
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589115"
---
# <a name="razor-pages-authorization-conventions-in-aspnet-core"></a>Razor ASP.NET Core でのページ承認規則

::: moniker range=">= aspnetcore-3.0"

ページアプリでアクセスを制御する方法の1つ Razor は、起動時に承認規則を使用することです。 これらの規則を使用すると、ユーザーを承認し、匿名ユーザーが個々のページやページのフォルダーにアクセスすることを許可できます。 このトピックで説明する規則は、アクセスを制御するための [承認フィルター](xref:mvc/controllers/filters#authorization-filters) を自動的に適用します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/razor-pages-authorization/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

サンプルアプリでは、を使用[ cookie せず ASP.NET Core Identity に認証](xref:security/authentication/cookie)を使用します。 このトピックで示す概念と例は、を使用するアプリにも同様に適用され ASP.NET Core Identity ます。 を使用するには ASP.NET Core Identity 、「」のガイダンスに従って <xref:security/authentication/identity> ください。

## <a name="require-authorization-to-access-a-page"></a>ページへのアクセスに承認を要求する

<xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizePage*> <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> 指定されたパスのページにを追加するには、規則を使用します。

[!code-csharp[](razor-pages-authorization/samples/3.x/AuthorizationSample/Startup.cs?name=snippet1&highlight=3)]

指定されたパスは、ビューエンジンのパスです。これは、 Razor 拡張子のないページルート相対パスで、スラッシュだけを含みます。

[承認ポリシー](xref:security/authorization/policies)を指定するには、 [authorizepage オーバーロード](xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizePage*)を使用します。

```csharp
options.Conventions.AuthorizePage("/Contact", "AtLeast21");
```

> [!NOTE]
> は、 <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> フィルター属性を使用してページモデルクラスに適用でき `[Authorize]` ます。 詳細については、「 [承認フィルター属性](xref:razor-pages/filter#authorize-filter-attribute)」を参照してください。

## <a name="require-authorization-to-access-a-folder-of-pages"></a>ページのフォルダーにアクセスするには承認が必要です

指定し <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeFolder*> <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> たパスにあるフォルダー内のすべてのページにを追加するには、規則を使用します。

[!code-csharp[](razor-pages-authorization/samples/3.x/AuthorizationSample/Startup.cs?name=snippet1&highlight=4)]

指定されたパスは、ページルートの相対パスであるビューエンジンのパスです Razor 。

[承認ポリシー](xref:security/authorization/policies)を指定するには、 [authorizefolder オーバーロード](xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeFolder*)を使用します。

```csharp
options.Conventions.AuthorizeFolder("/Private", "AtLeast21");
```

## <a name="require-authorization-to-access-an-area-page"></a>区分ページへのアクセスに承認を要求する

指定し <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeAreaPage*> <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> たパスの領域ページにを追加するには、規則を使用します。

```csharp
options.Conventions.AuthorizeAreaPage("Identity", "/Manage/Accounts");
```

ページ名は、指定された領域のページルートディレクトリを基準とした拡張子のないファイルのパスです。 たとえば、ファイル *領域/ Identity /Pages/Manage/Accounts.cshtml* のページ名は、/ *manage/Accounts* です。

[承認ポリシー](xref:security/authorization/policies)を指定するには、次のように、 [Authorizeareapage オーバーロード](xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeAreaPage*)を使用します。

```csharp
options.Conventions.AuthorizeAreaPage("Identity", "/Manage/Accounts", "AtLeast21");
```

## <a name="require-authorization-to-access-a-folder-of-areas"></a>領域のフォルダーにアクセスするための承認が必要

指定し <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeAreaFolder*> <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> たパスにあるフォルダー内のすべての領域にを追加するには、規則を使用します。

```csharp
options.Conventions.AuthorizeAreaFolder("Identity", "/Manage");
```

フォルダーパスは、指定された領域のページルートディレクトリを基準としたフォルダーのパスです。 たとえば、[区分]、[/]、 *Identity* [ *管理*] の下にあるファイルのフォルダーパスを使用します。

[承認ポリシー](xref:security/authorization/policies)を指定するには、 [AuthorizeAreaFolder オーバーロード](xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeAreaFolder*)を使用します。

```csharp
options.Conventions.AuthorizeAreaFolder("Identity", "/Manage", "AtLeast21");
```

## <a name="allow-anonymous-access-to-a-page"></a>ページへの匿名アクセスを許可する

指定し <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AllowAnonymousToPage*> <xref:Microsoft.AspNetCore.Mvc.Authorization.AllowAnonymousFilter> たパスにあるページにを追加するには、規則を使用します。

[!code-csharp[](razor-pages-authorization/samples/3.x/AuthorizationSample/Startup.cs?name=snippet1&highlight=5)]

指定されたパスは、ビューエンジンのパスです。これは、 Razor 拡張子のないページルート相対パスで、スラッシュだけを含みます。

## <a name="allow-anonymous-access-to-a-folder-of-pages"></a>ページのフォルダーへの匿名アクセスを許可する

指定し <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AllowAnonymousToFolder*> <xref:Microsoft.AspNetCore.Mvc.Authorization.AllowAnonymousFilter> たパスにあるフォルダー内のすべてのページにを追加するには、規則を使用します。

[!code-csharp[](razor-pages-authorization/samples/3.x/AuthorizationSample/Startup.cs?name=snippet1&highlight=6)]

指定されたパスは、ページルートの相対パスであるビューエンジンのパスです Razor 。

## <a name="note-on-combining-authorized-and-anonymous-access"></a>承認済みアクセスと匿名アクセスの組み合わせに関する注意事項

ページのフォルダーで承認が必要であることを指定し、そのフォルダー内のページで匿名アクセスが許可されるように指定することが有効です。

```csharp
// This works.
.AuthorizeFolder("/Private").AllowAnonymousToPage("/Private/Public")
```

ただし、逆は無効です。 匿名アクセス用のページのフォルダーを宣言して、承認が必要なフォルダー内のページを指定することはできません。

```csharp
// This doesn't work!
.AllowAnonymousToFolder("/Public").AuthorizePage("/Public/Private")
```

プライベートページで承認を要求すると失敗します。 との両方 <xref:Microsoft.AspNetCore.Mvc.Authorization.AllowAnonymousFilter> がページに適用されると、が優先され、 <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> アクセスが制御さ <xref:Microsoft.AspNetCore.Mvc.Authorization.AllowAnonymousFilter> れます。

## <a name="additional-resources"></a>その他のリソース

* <xref:razor-pages/razor-pages-conventions>
* <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ページアプリでアクセスを制御する方法の1つ Razor は、起動時に承認規則を使用することです。 これらの規則を使用すると、ユーザーを承認し、匿名ユーザーが個々のページやページのフォルダーにアクセスすることを許可できます。 このトピックで説明する規則は、アクセスを制御するための [承認フィルター](xref:mvc/controllers/filters#authorization-filters) を自動的に適用します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/razor-pages-authorization/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

サンプルアプリでは、を使用[ cookie せず ASP.NET Core Identity に認証](xref:security/authentication/cookie)を使用します。 このトピックで示す概念と例は、を使用するアプリにも同様に適用され ASP.NET Core Identity ます。 を使用するには ASP.NET Core Identity 、「」のガイダンスに従って <xref:security/authentication/identity> ください。

## <a name="require-authorization-to-access-a-page"></a>ページへのアクセスに承認を要求する

を使用して、 <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizePage*> <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.AddRazorPagesOptions*> 指定し <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> たパスのページにを追加します。

[!code-csharp[](razor-pages-authorization/samples/2.x/AuthorizationSample/Startup.cs?name=snippet1&highlight=2,4)]

指定されたパスは、ビューエンジンのパスです。これは、 Razor 拡張子のないページルート相対パスで、スラッシュだけを含みます。

[承認ポリシー](xref:security/authorization/policies)を指定するには、 [authorizepage オーバーロード](xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizePage*)を使用します。

```csharp
options.Conventions.AuthorizePage("/Contact", "AtLeast21");
```

> [!NOTE]
> は、 <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> フィルター属性を使用してページモデルクラスに適用でき `[Authorize]` ます。 詳細については、「 [承認フィルター属性](xref:razor-pages/filter#authorize-filter-attribute)」を参照してください。

## <a name="require-authorization-to-access-a-folder-of-pages"></a>ページのフォルダーにアクセスするには承認が必要です

を使用して、 <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeFolder*> <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.AddRazorPagesOptions*> 指定し <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> たパスにあるフォルダー内のすべてのページにを追加します。

[!code-csharp[](razor-pages-authorization/samples/2.x/AuthorizationSample/Startup.cs?name=snippet1&highlight=2,5)]

指定されたパスは、ページルートの相対パスであるビューエンジンのパスです Razor 。

[承認ポリシー](xref:security/authorization/policies)を指定するには、 [authorizefolder オーバーロード](xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeFolder*)を使用します。

```csharp
options.Conventions.AuthorizeFolder("/Private", "AtLeast21");
```

## <a name="require-authorization-to-access-an-area-page"></a>区分ページへのアクセスに承認を要求する

を使用して <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeAreaPage*> <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.AddRazorPagesOptions*> 、指定し <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> たパスの領域ページにを追加します。

```csharp
options.Conventions.AuthorizeAreaPage("Identity", "/Manage/Accounts");
```

ページ名は、指定された領域のページルートディレクトリを基準とした拡張子のないファイルのパスです。 たとえば、ファイル *領域/ Identity /Pages/Manage/Accounts.cshtml* のページ名は、/ *manage/Accounts* です。

[承認ポリシー](xref:security/authorization/policies)を指定するには、次のように、 [Authorizeareapage オーバーロード](xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeAreaPage*)を使用します。

```csharp
options.Conventions.AuthorizeAreaPage("Identity", "/Manage/Accounts", "AtLeast21");
```

## <a name="require-authorization-to-access-a-folder-of-areas"></a>領域のフォルダーにアクセスするための承認が必要

を使用して、 <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeAreaFolder*> <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.AddRazorPagesOptions*> 指定し <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> たパスにあるフォルダー内のすべての領域にを追加します。

```csharp
options.Conventions.AuthorizeAreaFolder("Identity", "/Manage");
```

フォルダーパスは、指定された領域のページルートディレクトリを基準としたフォルダーのパスです。 たとえば、[区分]、[/]、 *Identity* [ *管理*] の下にあるファイルのフォルダーパスを使用します。

[承認ポリシー](xref:security/authorization/policies)を指定するには、 [AuthorizeAreaFolder オーバーロード](xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeAreaFolder*)を使用します。

```csharp
options.Conventions.AuthorizeAreaFolder("Identity", "/Manage", "AtLeast21");
```

## <a name="allow-anonymous-access-to-a-page"></a>ページへの匿名アクセスを許可する

を使用して <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AllowAnonymousToPage*> <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.AddRazorPagesOptions*> 、指定し <xref:Microsoft.AspNetCore.Mvc.Authorization.AllowAnonymousFilter> たパスにあるページにを追加します。

[!code-csharp[](razor-pages-authorization/samples/2.x/AuthorizationSample/Startup.cs?name=snippet1&highlight=2,6)]

指定されたパスは、ビューエンジンのパスです。これは、 Razor 拡張子のないページルート相対パスで、スラッシュだけを含みます。

## <a name="allow-anonymous-access-to-a-folder-of-pages"></a>ページのフォルダーへの匿名アクセスを許可する

を使用して、 <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AllowAnonymousToFolder*> <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.AddRazorPagesOptions*> 指定し <xref:Microsoft.AspNetCore.Mvc.Authorization.AllowAnonymousFilter> たパスにあるフォルダー内のすべてのページにを追加します。

[!code-csharp[](razor-pages-authorization/samples/2.x/AuthorizationSample/Startup.cs?name=snippet1&highlight=2,7)]

指定されたパスは、ページルートの相対パスであるビューエンジンのパスです Razor 。

## <a name="note-on-combining-authorized-and-anonymous-access"></a>承認済みアクセスと匿名アクセスの組み合わせに関する注意事項

承認を必要とするページのフォルダーを指定し、そのフォルダー内のページで匿名アクセスが許可されるように指定することが有効です。

```csharp
// This works.
.AuthorizeFolder("/Private").AllowAnonymousToPage("/Private/Public")
```

ただし、逆は無効です。 匿名アクセス用のページのフォルダーを宣言して、承認が必要なフォルダー内のページを指定することはできません。

```csharp
// This doesn't work!
.AllowAnonymousToFolder("/Public").AuthorizePage("/Public/Private")
```

プライベートページで承認を要求すると失敗します。 との両方 <xref:Microsoft.AspNetCore.Mvc.Authorization.AllowAnonymousFilter> がページに適用されると、が優先され、 <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> アクセスが制御さ <xref:Microsoft.AspNetCore.Mvc.Authorization.AllowAnonymousFilter> れます。

## <a name="additional-resources"></a>その他のリソース

* <xref:razor-pages/razor-pages-conventions>
* <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection>

::: moniker-end
