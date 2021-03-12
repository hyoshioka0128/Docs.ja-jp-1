---
title: ASP.NET Core の Razor Pages のフィルター メソッド
author: Rick-Anderson
description: ASP.NET Core の Razor Pages のフィルター メソッドを作成する方法を学習します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 2/18/2020
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
uid: razor-pages/filter
ms.openlocfilehash: 178e6348d2d50dae34feea6a0ed261de01037136
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586138"
---
# <a name="filter-methods-for-razor-pages-in-aspnet-core"></a>ASP.NET Core の Razor Pages のフィルター メソッド

::: moniker range=">= aspnetcore-3.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

Razor ページのフィルターである [IPageFilter](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter?view=aspnetcore-2.0) および [IAsyncPageFilter](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter?view=aspnetcore-2.0) を使用すると、Razor ページ ハンドラーの実行の前後に Razor Pages でコードを実行できます。 Razor ページ フィルターは、個々のページ ハンドラー メソッドに適用できないことを除き、[ASP.NET Core MVC アクション フィルター](xref:mvc/controllers/filters#action-filters)と類似しています。

Razor ページ フィルター

* モデルのバインドが行われる前の、ハンドラー メソッドが選択された後にコードを実行します。
* モデルのバインドの完了後の、ハンドラー メソッドの実行前にコードを実行します。
* ハンドラー メソッドの実行後にコードを実行します。
* ページまたはグローバルに実装できます。
* 特定のページ ハンドラー メソッドには適用できません。
* コンストラクターの依存関係は、[依存関係の挿入](xref:fundamentals/dependency-injection) (DI) によって入力されるようにできます。 詳細については、「[ServiceFilterAttribute](../mvc/controllers/filters.md#servicefilterattribute)」と「[TypeFilterAttribute](../mvc/controllers/filters.md#typefilterattribute)」を参照してください。

ページ コンストラクターとミドルウェアにより、ハンドラー メソッドが実行される前にカスタム コードの実行が可能になりますが、<xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel.HttpContext> とページへのアクセスを可能にするのは Razor ページ フィルターのみです。 ミドルウェアは `HttpContext` にアクセスできますが、"ページ コンテキスト" にはアクセスできません。 フィルターには、`HttpContext` へのアクセスを提供する派生型のパラメーター <xref:Microsoft.AspNetCore.Mvc.Filters.FilterContext> があります。 ページ フィルターの例を次に示します。応答にヘッダーが追加する[フィルター属性を実装する](#ifa)。これは、コンストラクターやミドルウェアでは実行できません。 ページとそのモデルのインスタンスへのアクセスを含む、ページ コンテキストへのアクセスは、フィルター、ハンドラー、Razor ページの本文を実行するときにのみ利用できます。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/filter/3.1sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

Razor ページ フィルターには、グローバルまたはページ レベルで適用できる次のメソッドがあります。

* 同期メソッド:

  * [OnPageHandlerSelected](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerselected?view=aspnetcore-2.0):ハンドラー メソッドが選択された後の、モデル バインドが行われる前に呼び出されます。
  * [OnPageHandlerExecuting](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerexecuting?view=aspnetcore-2.0):ハンドラー メソッドが実行される前の、モデル バインドが完了した後に呼び出されます。
  * [OnPageHandlerExecuted](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerexecuted?view=aspnetcore-2.0):ハンドラー メソッドが実行された後の、アクションの結果の前に呼び出されます。

* 非同期メソッド:

  * [OnPageHandlerSelectionAsync](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter.onpagehandlerselectionasync?view=aspnetcore-2.0):ハンドラー メソッドが選択された後の、モデル バインドが行われる前に非同期で呼び出されます。
  * [OnPageHandlerExecutionAsync](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter.onpagehandlerexecutionasync?view=aspnetcore-2.0):ハンドラー メソッドが呼び出される前の、モデル バインドの完了後に非同期で呼び出されます。

フィルター インターフェイスの同期と非同期バージョンの **両方ではなく**、**いずれか** を実装します。 フレームワークは、最初にフィルターが非同期インターフェイスを実装しているかどうかをチェックして、している場合はそれを呼び出します。 していない場合は、同期インターフェイスのメソッドを呼び出します。 両方のインターフェイスを実装した場合、非同期メソッドのみが呼び出されます。 ページのオーバーライドでもこの規則は同じです。オーバーライドの同期バージョンまたは非同期バージョンを実装でき、両方はできません。

## <a name="implement-razor-page-filters-globally"></a>Razor ページにフィルターをグローバルに実装する

`IAsyncPageFilter` は、次のコードによって実装されます。

[!code-csharp[Main](filter/3.1sample/PageFilter/Filters/SampleAsyncPageFilter.cs?name=snippet1)]

上記のコードでは、`ProcessUserAgent.Write` が、ユーザー エージェント文字列を使用して機能するユーザー指定のコードです。

次のコードは、`Startup` クラスで `SampleAsyncPageFilter` を有効にします。

[!code-csharp[Main](filter/3.1sample/PageFilter/Startup.cs?name=snippet2)]

次のコードでは、<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection.AddFolderApplicationModelConvention*> を呼び出して、 */Movies* 内のページにのみ `SampleAsyncPageFilter` を適用しています。

[!code-csharp[Main](filter/3.1sample/PageFilter/Startup2.cs?name=snippet2)]

次のコードは、同期 `IPageFilter` を実装します。

[!code-csharp[Main](filter/3.1sample/PageFilter/Filters/SamplePageFilter.cs?name=snippet1)]

次のコードは、`SamplePageFilter` を有効にします。

[!code-csharp[Main](filter/3.1sample/PageFilter/StartupSync.cs?name=snippet2)]

## <a name="implement-razor-page-filters-by-overriding-filter-methods"></a>フィルター メソッドをオーバーライドして Razor ページにフィルターを実装する

次のコードでは、非同期 Razor ページ フィルターをオーバーライドしています。

[!code-csharp[Main](filter/3.1sample/PageFilter/Pages/Index.cshtml.cs?name=snippet)]

<a name="ifa"></a>

## <a name="implement-a-filter-attribute"></a>フィルター属性を実装する

組み込みの属性ベースのフィルターである <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter.OnResultExecutionAsync*> フィルターは、サブクラス化できます。 次のフィルターは、応答にヘッダーを追加します。

[!code-csharp[Main](filter/3.1sample/PageFilter/Filters/AddHeaderAttribute.cs)]

次のコードは、`AddHeader` 属性を追加します。

[!code-csharp[Main](filter/3.1sample/PageFilter/Pages/Movies/Test.cshtml.cs)]

ブラウザー開発者ツールなどのツールを使用して、ヘッダーを調べます。 **[応答ヘッダー]** の下に `author: Rick` が表示されます。

順序をオーバーライドする手順については、「[既定の順序のオーバーライド](xref:mvc/controllers/filters#overriding-the-default-order)」を参照してください。

フィルターからフィルター パイプラインをショート サーキットする手順については、「[キャンセルとショート サーキット](xref:mvc/controllers/filters#cancellation-and-short-circuiting)」を参照してください。

<a name="auth"></a>

## <a name="authorize-filter-attribute"></a>Authorize フィルター属性

`PageModel` に [Authorize](/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute?view=aspnetcore-2.0) 属性を適用できます。

[!code-csharp[Main](filter/sample/PageFilter/Pages/ModelWithAuthFilter.cshtml.cs?highlight=7)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

Razor ページのフィルターである [IPageFilter](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter?view=aspnetcore-2.0) および [IAsyncPageFilter](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter?view=aspnetcore-2.0) を使用すると、Razor ページ ハンドラーの実行の前後に Razor Pages でコードを実行できます。 Razor ページ フィルターは、個々のページ ハンドラー メソッドに適用できないことを除き、[ASP.NET Core MVC アクション フィルター](xref:mvc/controllers/filters#action-filters)と類似しています。

Razor ページ フィルター

* モデルのバインドが行われる前の、ハンドラー メソッドが選択された後にコードを実行します。
* モデルのバインドの完了後の、ハンドラー メソッドの実行前にコードを実行します。
* ハンドラー メソッドの実行後にコードを実行します。
* ページまたはグローバルに実装できます。
* 特定のページ ハンドラー メソッドには適用できません。

コードは、ページ コンストラクターまたはミドルウェアを使用してハンドラー メソッドの実行前に実行できますが、[HttpContext](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel.httpcontext?view=aspnetcore-2.0#Microsoft_AspNetCore_Mvc_RazorPages_PageModel_HttpContext) にアクセスできるのは Razor ページ フィルターのみです。 フィルターには、`HttpContext` へのアクセスを提供する [FilterContext](/dotnet/api/microsoft.aspnetcore.mvc.filters.filtercontext?view=aspnetcore-2.0) 派生のパラメーターがあります。 たとえば、「[フィルター属性を実装する](#ifa)」のサンプルでは、応答にヘッダーが追加されます。これは、コンストラクターやミドルウェアでは実行できません。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/filter/sample/PageFilter)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

Razor ページ フィルターには、グローバルまたはページ レベルで適用できる次のメソッドがあります。

* 同期メソッド:

  * [OnPageHandlerSelected](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerselected?view=aspnetcore-2.0):ハンドラー メソッドが選択された後の、モデル バインドが行われる前に呼び出されます。
  * [OnPageHandlerExecuting](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerexecuting?view=aspnetcore-2.0):ハンドラー メソッドが実行される前の、モデル バインドが完了した後に呼び出されます。
  * [OnPageHandlerExecuted](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerexecuted?view=aspnetcore-2.0):ハンドラー メソッドが実行された後の、アクションの結果の前に呼び出されます。

* 非同期メソッド:

  * [OnPageHandlerSelectionAsync](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter.onpagehandlerselectionasync?view=aspnetcore-2.0):ハンドラー メソッドが選択された後の、モデル バインドが行われる前に非同期で呼び出されます。
  * [OnPageHandlerExecutionAsync](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter.onpagehandlerexecutionasync?view=aspnetcore-2.0):ハンドラー メソッドが呼び出される前の、モデル バインドの完了後に非同期で呼び出されます。

> [!NOTE]
> フィルター インターフェイスの同期と非同期バージョンの両方ではなく、**いずれか** を実装します。 フレームワークは、最初にフィルターが非同期インターフェイスを実装しているかどうかをチェックして、している場合はそれを呼び出します。 していない場合は、同期インターフェイスのメソッドを呼び出します。 両方のインターフェイスを実装した場合、非同期メソッドのみが呼び出されます。 ページのオーバーライドでもこの規則は同じです。オーバーライドの同期バージョンまたは非同期バージョンを実装でき、両方はできません。

## <a name="implement-razor-page-filters-globally"></a>Razor ページにフィルターをグローバルに実装する

`IAsyncPageFilter` は、次のコードによって実装されます。

[!code-csharp[Main](filter/sample/PageFilter/Filters/SampleAsyncPageFilter.cs?name=snippet1)]

前述のコードでは、[ILogger](/dotnet/api/microsoft.extensions.logging.ilogger?view=aspnetcore-2.0) は不要です。 これは、アプリケーションのトレース情報を提供するためにサンプルで使用されています。

次のコードは、`Startup` クラスで `SampleAsyncPageFilter` を有効にします。

[!code-csharp[Main](filter/sample/PageFilter/Startup.cs?name=snippet2&highlight=11)]

次は、完全な `Startup` クラスのコードです。

[!code-csharp[Main](filter/sample/PageFilter/Startup.cs?name=snippet1)]

次のコードは、`AddFolderApplicationModelConvention` を呼び出し、 */subFolder* のページにのみ `SampleAsyncPageFilter` を適用します。

[!code-csharp[Main](filter/sample/PageFilter/Startup2.cs?name=snippet2)]

次のコードは、同期 `IPageFilter` を実装します。

[!code-csharp[Main](filter/sample/PageFilter/Filters/SamplePageFilter.cs?name=snippet1)]

次のコードは、`SamplePageFilter` を有効にします。

[!code-csharp[Main](filter/sample/PageFilter/StartupSync.cs?name=snippet2&highlight=11)]

## <a name="implement-razor-page-filters-by-overriding-filter-methods"></a>フィルター メソッドをオーバーライドして Razor ページにフィルターを実装する

次のコードでは、同期 Razor ページ フィルターをオーバーライドしています。

[!code-csharp[Main](filter/sample/PageFilter/Pages/Index.cshtml.cs)]

<a name="ifa"></a>

## <a name="implement-a-filter-attribute"></a>フィルター属性を実装する

組み込みの属性ベースのフィルターである [OnResultExecutionAsync](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncresultfilter.onresultexecutionasync?view=aspnetcore-2.0#Microsoft_AspNetCore_Mvc_Filters_IAsyncResultFilter_OnResultExecutionAsync_Microsoft_AspNetCore_Mvc_Filters_ResultExecutingContext_Microsoft_AspNetCore_Mvc_Filters_ResultExecutionDelegate_) フィルターはサブクラス化することができます。 次のフィルターは、応答にヘッダーを追加します。

[!code-csharp[Main](filter/sample/PageFilter/Filters/AddHeaderAttribute.cs)]

次のコードは、`AddHeader` 属性を追加します。

[!code-csharp[Main](filter/sample/PageFilter/Pages/Contact.cshtml.cs?name=snippet1)]

順序をオーバーライドする手順については、「[既定の順序のオーバーライド](xref:mvc/controllers/filters#overriding-the-default-order)」を参照してください。

フィルターからフィルター パイプラインをショート サーキットする手順については、「[キャンセルとショート サーキット](xref:mvc/controllers/filters#cancellation-and-short-circuiting)」を参照してください。 

<a name="auth"></a>

## <a name="authorize-filter-attribute"></a>Authorize フィルター属性

`PageModel` に [Authorize](/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute?view=aspnetcore-2.0) 属性を適用できます。

[!code-csharp[Main](filter/sample/PageFilter/Pages/ModelWithAuthFilter.cshtml.cs?highlight=7)]

::: moniker-end