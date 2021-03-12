---
title: ASP.NET Core での Razor ページのルートとアプリの規則
author: rick-anderson
description: ルートとアプリ モデル プロバイダーの規則が、ページのルーティング、検出、および処理の制御にどのように役立つかについて確認します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/07/2020
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
uid: razor-pages/razor-pages-conventions
ms.openlocfilehash: c75e2f1de522f80f4d8e13cf3f60c99cc6b7196a
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589414"
---
# <a name="razor-pages-route-and-app-conventions-in-aspnet-core"></a>ASP.NET Core での Razor ページのルートとアプリの規則

::: moniker range=">= aspnetcore-3.0"

[ページ ルートとアプリ モデル プロバイダーの規則](xref:mvc/controllers/application-model#conventions)を使用して、Razor ページ アプリでページのルーティング、検出、および処理を制御する方法について説明します。

個別のページにカスタムのページ ルートを構成する必要がある場合、このトピックで後述される「[AddPageRoute convention](#configure-a-page-route)」で、ページへのルーティングを構成します。

ページ ルートの指定、ルート セグメントの追加、ルートへのパラメーターの追加を行うには、ページの `@page` ディレクティブを使用します。 詳しくは、「[カスタム ルート](xref:razor-pages/index#custom-routes)」をご覧ください。

ルート セグメントやパラメーター名として使用できない予約語がいくつかあります。 詳しくは、[ルーティング: ルーティングの予約名](xref:mvc/controllers/routing#reserved-routing-names)に関するページをご覧ください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/razor-pages-conventions/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

| シナリオ | このサンプルでは、次のデモを実行します。 |
| -------- | --------------------------- |
| [モデルの規則](#model-conventions)<br><br>Conventions.Add<ul><li>IPageRouteModelConvention</li><li>IPageApplicationModelConvention</li><li>IPageHandlerModelConvention</li></ul> | ルート テンプレートとヘッダーをアプリのページに追加します。 |
| [ページ ルート アクション規則](#page-route-action-conventions)<ul><li>AddFolderRouteModelConvention</li><li>AddPageRouteModelConvention</li><li>AddPageRoute</li></ul> | ルート テンプレートをフォルダー内のページおよび単一ページに追加します。 |
| [ページ モデル アクション規則](#page-model-action-conventions)<ul><li>AddFolderApplicationModelConvention</li><li>AddPageApplicationModelConvention</li><li>ConfigureFilter (フィルター クラス、ラムダ式、またはフィルター ファクトリ)</li></ul> | ヘッダーをフォルダー内のページに追加し、ヘッダーを単一ページに追加し、ヘッダーをアプリのページに追加するように[フィルター ファクトリ](xref:mvc/controllers/filters#ifilterfactory)を構成します。 |

Razor Pages の規則は、`Startup.ConfigureServices` で <xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions> を構成する <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddRazorPages%2A> のオーバーロードを使用して構成されます。 次の規則の例は、このトピックで後述されます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddRazorPages(options =>
    {
        options.Conventions.Add( ... );
        options.Conventions.AddFolderRouteModelConvention(
            "/OtherPages", model => { ... });
        options.Conventions.AddPageRouteModelConvention(
            "/About", model => { ... });
        options.Conventions.AddPageRoute(
            "/Contact", "TheContactPage/{text?}");
        options.Conventions.AddFolderApplicationModelConvention(
            "/OtherPages", model => { ... });
        options.Conventions.AddPageApplicationModelConvention(
            "/About", model => { ... });
        options.Conventions.ConfigureFilter(model => { ... });
        options.Conventions.ConfigureFilter( ... );
    });
}
```

## <a name="route-order"></a>ルートの順番

ルートは、処理 (ルートの照合) に使われる順番 (<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel.Order*>) を指定します。

| 順番            | 動作 |
| :--------------: | -------- |
| -1               | ルートは、他のルートが処理される前に処理されます。 |
| 0                | 順番が指定されていません (既定値)。 `Order` を割り当てない (`Order = null`) 場合、処理に使われるルートの `Order` は既定で 0 (ゼロ) になります。 |
| 1、2、&hellip; n | ルートの処理順序を指定します。 |

ルートの処理は、次の規則で定められています。

* ルートは並んだ順番に処理されます (-1、0、1、2、&hellip; n)。
* ルートの `Order` が同じ場合は、最初に最も明確なルートが照合され、続けて次に明確なルートが照合されます。
* `Order` とパラメーター数が同じルートが 1 つの要求 URL と一致した場合、ルートは <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection> に追加された順番で処理されます。

可能であれば、定められたルートの処理順序に依存しないようにしてください。 通常は、ルーティングによって、URL が一致する正しいルートが選択されます。 要求が正しくルーティングされるようにルートの `Order` プロパティを設定する必要がある場合、アプリのルーティング方式がクライアントにとって紛らわしいものとなり、保守が脆弱になる可能性があります。 アプリのルーティング方式を簡略化するよう努めてください。 サンプル アプリでは、1 つのアプリで複数のルーティング シナリオを示すために、ルートの明示的な処理順序が必要です。 ただし、運用環境のアプリでルートの `Order` を設定するやり方は避けるようにしてください。

Razor Pages ルーティングと MVC コントローラー ルーティングは、実装を共有します。 MVC のトピックに記載されているルートの順番については、[コントローラー アクションへのルーティング: 属性ルートの順序の指定](xref:mvc/controllers/routing#ordering-attribute-routes)に関するページをご覧ください。

## <a name="model-conventions"></a>モデルの規則

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageConvention> の委任を追加すると、Razor Pages に適用される[モデルの規則](xref:mvc/controllers/application-model#conventions)を追加できます。

### <a name="add-a-route-model-convention-to-all-pages"></a>すべてのページにルート モデル規則を追加する

<xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions.Conventions> を使用すると、<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageRouteModelConvention> を作成して、ページ ルート モデルの構築中に適用される <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageConvention> インスタンスのコレクションに追加できます。

サンプル アプリでは、`{globalTemplate?}` ルート テンプレートをアプリ内のすべてのページに追加します。

[!code-csharp[](razor-pages-conventions/samples/3.x/SampleApp/Conventions/GlobalTemplatePageRouteModelConvention.cs?name=snippet1)]

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel> の <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel.Order*> プロパティに `1` が設定されます。 これにより、サンプル アプリで次のルートの照合動作が保証されます。

* `TheContactPage/{text?}` のルート テンプレートは、このトピックの後半で追加されます。 連絡先ページのルートには既定の順番 `null` (`Order = 0`) が設定されているため、`{globalTemplate?}` ルート テンプレートの前に一致します。
* `{aboutTemplate?}` ルート テンプレートは、このトピックの後半で追加されます。 `{aboutTemplate?}` テンプレートの `Order` には `2` が指定されます。 [About] ページが `/About/RouteDataValue` で要求されると、"RouteDataValue" は `RouteData.Values["globalTemplate"]` (`Order = 1`) に読み込まれ、`Order` プロパティが設定されるため `RouteData.Values["aboutTemplate"]` (`Order = 2`) には読み込まれません。
* `{otherPagesTemplate?}` ルート テンプレートは、このトピックの後半で追加されます。 `{otherPagesTemplate?}` テンプレートの `Order` には `2` が指定されます。 ルート パラメーター (`/OtherPages/Page1/RouteDataValue` など) を使用して *Pages/OtherPages* フォルダー内のいずれかのページが要求されると、`Order` プロパティが設定されているため、"RouteDataValue" が `RouteData.Values["otherPagesTemplate"]` (`Order = 2`) ではなく `RouteData.Values["globalTemplate"]` (`Order = 1`) に読み込まれます。

可能な限り、`Order` を設定しないでください。これにより、`Order = 0` が生じます。 正しいルートの選択には、ルーティングを使用してください。

Razor Pages のオプション (<xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions.Conventions> の追加など) は、Razor Pages が `Startup.ConfigureServices` のサービス コレクションに追加されたときに追加されます。 例については、[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/razor-pages-conventions/samples/)を参照してください。

[!code-csharp[](razor-pages-conventions/samples/3.x/SampleApp/Startup.cs?name=snippet1)]

`localhost:5000/About/GlobalRouteValue` でサンプルの [About] ページを要求し、その結果を調べます。

![[About] ページは、GlobalRouteValue のルート セグメントで要求されます。 レンダリングされたページは、ルート データの値がページの OnGet メソッドでキャプチャされたことを示しています。](razor-pages-conventions/_static/about-page-global-template.png)

### <a name="add-an-app-model-convention-to-all-pages"></a>すべてのページにアプリ モデル規則を追加する

<xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions.Conventions> を使用すると、<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageApplicationModelConvention> を作成して、ページ アプリ モデルの構築中に適用される <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageConvention> インスタンスのコレクションに追加できます。

この規則やこのトピックで後述されるその他の規則のデモを実行するには、サンプル アプリに `AddHeaderAttribute` クラスを含めます。 クラス コンストラクターは、`name` 文字列と `values` 文字列配列を受け入れます。 これらの値は、応答ヘッダーを設定するために、その `OnResultExecuting` メソッド内で使用されます。 完全クラスは、このトピックで後述される「[ページ モデル アクション規則](#page-model-action-conventions)」セクションで示されます。

サンプル アプリでは、`AddHeaderAttribute` クラスを使用して、ヘッダー (`GlobalHeader`) をアプリ内のすべてのページに追加します。

[!code-csharp[](razor-pages-conventions/samples/3.x/SampleApp/Conventions/GlobalHeaderPageApplicationModelConvention.cs?name=snippet1)]

*Startup.cs*:

[!code-csharp[](razor-pages-conventions/samples/3.x/SampleApp/Startup.cs?name=snippet2)]

`localhost:5000/About` でサンプルの [About] ページを要求し、そのヘッダーを調べて結果を確認します。

![[About] ページの応答ヘッダーは、GlobalHeader が追加されたことを示しています。](razor-pages-conventions/_static/about-page-global-header.png)

### <a name="add-a-handler-model-convention-to-all-pages"></a>すべてのページにハンドラー モデル規則を追加する

<xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions.Conventions> を使用すると、<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageHandlerModelConvention> を作成して、ページ ハンドラー モデルの構築中に適用される <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageConvention> インスタンスのコレクションに追加できます。

[!code-csharp[](razor-pages-conventions/samples/3.x/SampleApp/Conventions/GlobalPageHandlerModelConvention.cs?name=snippet1)]

*Startup.cs*:

[!code-csharp[](razor-pages-conventions/samples/3.x/SampleApp/Startup.cs?name=snippet10)]

## <a name="page-route-action-conventions"></a>ページ ルート アクション規則

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageRouteModelProvider> から派生する既定のルート モデル プロバイダーは、ページ ルートを構成するための拡張ポイントを提供するようにデザインされた規則を呼び出します。

### <a name="folder-route-model-convention"></a>フォルダー ルート モデル規則

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection.AddFolderRouteModelConvention*> を使用すると、指定したフォルダーの下にあるすべてのページを対象に <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageRouteModel> へのアクションを呼び出す <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageRouteModelConvention> を作成して追加できます。

サンプル アプリでは <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection.AddFolderRouteModelConvention*> を使用して、`{otherPagesTemplate?}` ルート テンプレートを *OtherPages* フォルダーのページに追加します。

[!code-csharp[](razor-pages-conventions/samples/3.x/SampleApp/Startup.cs?name=snippet3)]

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel> の <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel.Order*> プロパティに `2` が設定されます。 この設定により、1 つのルート値が指定されたときに、(このトピックの前半で `1` に設定した) `{globalTemplate?}` のテンプレートが優先的に最初のルート データ値の位置に指定されます。 ルート パラメーター値 (`/OtherPages/Page1/RouteDataValue` など) を使用して *Pages/OtherPages* フォルダー内のページが要求されると、`Order` プロパティが設定されているため、"RouteDataValue" が `RouteData.Values["otherPagesTemplate"]` (`Order = 2`) ではなく `RouteData.Values["globalTemplate"]` (`Order = 1`) に読み込まれます。

可能な限り、`Order` を設定しないでください。これにより、`Order = 0` が生じます。 正しいルートの選択には、ルーティングを使用してください。

`localhost:5000/OtherPages/Page1/GlobalRouteValue/OtherPagesRouteValue` でサンプルの Page1 ページを要求し、その結果を調べます。

![OtherPages フォルダーの Page1 は、GlobalRouteValue と OtherPagesRouteValue のルート セグメントで要求されます。 レンダリングされたページは、ルート データの値がページの OnGet メソッドでキャプチャされたことを示しています。](razor-pages-conventions/_static/otherpages-page1-global-and-otherpages-templates.png)

### <a name="page-route-model-convention"></a>ページ ルート モデル規則

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection.AddPageRouteModelConvention*> を使用すると、指定した名前のページを対象に <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageRouteModel> へのアクションを呼び出す <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageRouteModelConvention> を作成して追加できます。

サンプル アプリでは `AddPageRouteModelConvention` を使用して、`{aboutTemplate?}` ルート テンプレートを [About] ページに追加します。

[!code-csharp[](razor-pages-conventions/samples/3.x/SampleApp/Startup.cs?name=snippet4)]

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel> の <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel.Order*> プロパティに `2` が設定されます。 この設定により、1 つのルート値が指定されたときに、(このトピックの前半で `1` に設定した) `{globalTemplate?}` のテンプレートが優先的に最初のルート データ値の位置に指定されます。 [About] ページが `/About/RouteDataValue` にあるルート パラメーター値で要求されると、`Order` プロパティが設定されているため、"RouteDataValue" は `RouteData.Values["aboutTemplate"]` (`Order = 2`) ではなく `RouteData.Values["globalTemplate"]` (`Order = 1`) に読み込まれます。

可能な限り、`Order` を設定しないでください。これにより、`Order = 0` が生じます。 正しいルートの選択には、ルーティングを使用してください。

`localhost:5000/About/GlobalRouteValue/AboutRouteValue` でサンプルの [About] ページを要求し、その結果を調べます。

![[About] ページは、GlobalRouteValue と AboutRouteValue のルート セグメントで要求されます。 レンダリングされたページは、ルート データの値がページの OnGet メソッドでキャプチャされたことを示しています。](razor-pages-conventions/_static/about-page-global-and-about-templates.png)

## <a name="use-a-parameter-transformer-to-customize-page-routes"></a>パラメーター トランスフォーマーを使用してページ ルートをカスタマイズする

ASP.NET Core によって生成されたページ ルートは、パラメーター トランスフォーマーを使用してカスタマイズできます。 パラメーター トランスフォーマーは `IOutboundParameterTransformer` を実装し、パラメーターの値を変換します。 たとえば、`SlugifyParameterTransformer` パラメーター トランスフォーマーでは、`SubscriptionManagement` のルート値が `subscription-management` に変更されます。

`PageRouteTransformerConvention` ページ ルート モデル規則では、アプリで自動的に生成されたページ ルートのフォルダー名とファイル名のセグメントにパラメーター トランスフォーマーを適用します。 たとえば、 */Pages/SubscriptionManagement/ViewAll.cshtml* にある Razor ページ ファイルのルートは `/SubscriptionManagement/ViewAll` から `/subscription-management/view-all` に書き換えられます。

`PageRouteTransformerConvention` では、Razor ページのフォルダー名とファイル名に由来する、自動的に生成されたページ ルートのセグメントのみを変換します。 `@page` ディレクティブを使用して追加したルート セグメントは変換されません。 この規則では、<xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AddPageRoute*> で追加したルートも変換されません。

`PageRouteTransformerConvention` は、`Startup.ConfigureServices` でオプションとして登録されます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddRazorPages(options =>
    {
        options.Conventions.Add(
            new PageRouteTransformerConvention(
                new SlugifyParameterTransformer()));
    });
}
```

[!code-csharp[](~/mvc/controllers/routing/samples/3.x/main/StartupSlugifyParamTransformer.cs?name=snippet2)]

[!INCLUDE[](~/includes/regex.md)]

## <a name="configure-a-page-route"></a>ページ ルートの構成

<xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AddPageRoute*> を使用すると、指定したページ パスにあるページへのルートを構成できます。 そのページに対して生成されたリンクでは、指定したルートを使用します。 `AddPageRoute` では、`AddPageRouteModelConvention` を使用してルートを確立します。

サンプル アプリでは、*Contact.cshtml* の `/TheContactPage` へのルートを作成します。

[!code-csharp[](razor-pages-conventions/samples/3.x/SampleApp/Startup.cs?name=snippet5)]

[Contact] ページには、既定のルート経由の `/Contact` でアクセスすることもできます。

サンプル アプリの [Contact] ページに対するカスタム ルートでは、省略可能な `text` ルート セグメント (`{text?}`) を許可します。 また、訪問者が `/Contact` ルートでページにアクセスする場合、ページの `@page` ディレクティブにはこの省略可能なセグメントも含まれます。

[!code-cshtml[](razor-pages-conventions/samples/3.x/SampleApp/Pages/Contact.cshtml?highlight=1)]

レンダリングされたページの **Contact** リンク用に生成された URL には、更新されたルートが反映されることに注意してください。

![ナビゲーション バーのサンプル アプリの [Contact] リンク](razor-pages-conventions/_static/contact-link.png)

![レンダリングされた HTML の [Contact] リンクを調べると、href に '/TheContactPage' が設定されています](razor-pages-conventions/_static/contact-link-source.png)

通常のルート (`/Contact`) またはカスタム ルート (`/TheContactPage`) のいずれかで、[Contact] ページにアクセスします。 追加の `text` ルート セグメントを指定した場合、ページには指定した HTML エンコードのセグメントが示されます。

![URL の 'TextValue' の省略可能な 'text' ルート セグメントを適用する Microsoft Edge ブラウザーの例。 レンダリングされたページには、'text' セグメントの値が示されています。](razor-pages-conventions/_static/route-segment-with-custom-route.png)

## <a name="page-model-action-conventions"></a>ページ モデル アクション規則

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageApplicationModelProvider> を実装する既定のページ モデル プロバイダーは、ページ モデルを構成するための拡張ポイントを提供するようにデザインされた規則を呼び出します。 これらの規則は、ページ検出をビルドおよび変更したり、シナリオを処理したりするときに便利です。

このセクションの例の場合、サンプル アプリでは、応答ヘッダーを適用する <xref:Microsoft.AspNetCore.Mvc.Filters.ResultFilterAttribute> である、`AddHeaderAttribute` クラスを使用します。

[!code-csharp[](razor-pages-conventions/samples/3.x/SampleApp/Filters/AddHeader.cs?name=snippet1)]

規則を使用して、このサンプルでは、フォルダー内のすべてのページおよび単一ページに属性を適用する方法のデモを実行します。

**フォルダー アプリ モデル規則**

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection.AddFolderApplicationModelConvention*> を使用すると、指定したフォルダーの下にあるすべてのページを対象に <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageApplicationModel> インスタンスへのアクションを呼び出す <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageApplicationModelConvention> を作成して追加できます。

このサンプルでは、ヘッダー (`OtherPagesHeader`) をアプリの *OtherPages* フォルダー内にあるページに追加して、`AddFolderApplicationModelConvention` を使用するデモを実行します。

[!code-csharp[](razor-pages-conventions/samples/3.x/SampleApp/Startup.cs?name=snippet6)]

`localhost:5000/OtherPages/Page1` でサンプルの Page1 ページを要求し、そのヘッダーを調べて結果を確認します。

![OtherPages/Page1 ページの応答ヘッダーは、OtherPagesHeader が追加されていることを示しています。](razor-pages-conventions/_static/page1-otherpages-header.png)

**ページ アプリ モデル規則**

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection.AddPageApplicationModelConvention*> を使用すると、指定した名前のページを対象に <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageApplicationModel> へのアクションを呼び出す <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageApplicationModelConvention> を作成して追加できます。

サンプルでは、ヘッダー (`AboutHeader`) を [About] ページに追加して、`AddPageApplicationModelConvention` を使用するデモを実行します。

[!code-csharp[](razor-pages-conventions/samples/3.x/SampleApp/Startup.cs?name=snippet7)]

`localhost:5000/About` でサンプルの [About] ページを要求し、そのヘッダーを調べて結果を確認します。

![[About] ページの応答ヘッダーは、AboutHeader が追加されたことを示しています。](razor-pages-conventions/_static/about-page-about-header.png)

**フィルターの構成**

<xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.ConfigureFilter*> では、指定したフィルターを適用するように構成します。 フィルター クラスを実装できますが、サンプル アプリでは、ラムダ式でフィルターを実装する方法を示しています。これは、フィルターを返すファクトリとしてバックグラウンドで実装されます。

[!code-csharp[](razor-pages-conventions/samples/3.x/SampleApp/Startup.cs?name=snippet8)]

ページ アプリ モデルは、*OtherPages* フォルダーの Page2 ページにつながるセグメントの相対パスを確認するために使用されます。 条件を満たすと、ヘッダーが追加されます。 満たさない場合は、`EmptyFilter` が適用されます。

`EmptyFilter` は[アクション フィルター](xref:mvc/controllers/filters#action-filters)です。 アクション フィルターは Razor Pages によって無視されるため、パスに `OtherPages/Page2` が含まれない場合、`EmptyFilter` には意図されたような効果はありません。

`localhost:5000/OtherPages/Page2` でサンプルの Page2 ページを要求し、そのヘッダーを調べて結果を確認します。

![OtherPagesPage2Header は、Page2 への応答に追加されます。](razor-pages-conventions/_static/page2-filter-header.png)

**フィルター ファクトリの構成**

<xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.ConfigureFilter*> では、すべての Razor Pages に[フィルター](xref:mvc/controllers/filters)を適用するように、指定したファクトリを構成します。

サンプル アプリでは、アプリのページに対する 2 つの値と共にヘッダー (`FilterFactoryHeader`) を追加して、[フィルター ファクトリ](xref:mvc/controllers/filters#ifilterfactory)を使用する例を提供します。

[!code-csharp[](razor-pages-conventions/samples/3.x/SampleApp/Startup.cs?name=snippet9)]

*AddHeaderWithFactory.cs*:

[!code-csharp[](razor-pages-conventions/samples/3.x/SampleApp/Factories/AddHeaderWithFactory.cs?name=snippet1)]

`localhost:5000/About` でサンプルの [About] ページを要求し、そのヘッダーを調べて結果を確認します。

![[About] ページの応答ヘッダーは、FilterFactoryHeader ヘッダーが 2 つ追加されたことを示しています。](razor-pages-conventions/_static/about-page-filter-factory-header.png)

## <a name="mvc-filters-and-the-page-filter-ipagefilter"></a>MVC フィルターとページ フィルター (IPageFilter)

Razor Pages はハンドラー メソッドを使用するため、MVC [アクション フィルター](xref:mvc/controllers/filters#action-filters)は Razor Pages によって無視されます。 使用できるその他の種類の MVC フィルターは次のとおりです。[承認](xref:mvc/controllers/filters#authorization-filters)、[例外](xref:mvc/controllers/filters#exception-filters)、[リソース](xref:mvc/controllers/filters#resource-filters)、および[結果](xref:mvc/controllers/filters#result-filters)。 詳細については、「[フィルター](xref:mvc/controllers/filters)」トピックを参照してください。

ページ フィルター (<xref:Microsoft.AspNetCore.Mvc.Filters.IPageFilter>) は、Razor Pages に適用されるフィルターの 1 つです。 詳細については、[Razor Pages のフィルター メソッド](xref:razor-pages/filter)に関するページを参照してください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:security/authorization/razor-pages-authorization>
* <xref:mvc/controllers/areas#areas-with-razor-pages>

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[ページ ルートとアプリ モデル プロバイダーの規則](xref:mvc/controllers/application-model#conventions)を使用して、Razor ページ アプリでページのルーティング、検出、および処理を制御する方法について説明します。

個別のページにカスタムのページ ルートを構成する必要がある場合、このトピックで後述される「[AddPageRoute convention](#configure-a-page-route)」で、ページへのルーティングを構成します。

ページ ルートの指定、ルート セグメントの追加、ルートへのパラメーターの追加を行うには、ページの `@page` ディレクティブを使用します。 詳しくは、「[カスタム ルート](xref:razor-pages/index#custom-routes)」をご覧ください。

ルート セグメントやパラメーター名として使用できない予約語がいくつかあります。 詳しくは、[ルーティング: ルーティングの予約名](xref:fundamentals/routing#reserved-routing-names)に関するページをご覧ください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/razor-pages-conventions/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

| シナリオ | このサンプルでは、次のデモを実行します。 |
| -------- | --------------------------- |
| [モデルの規則](#model-conventions)<br><br>Conventions.Add<ul><li>IPageRouteModelConvention</li><li>IPageApplicationModelConvention</li><li>IPageHandlerModelConvention</li></ul> | ルート テンプレートとヘッダーをアプリのページに追加します。 |
| [ページ ルート アクション規則](#page-route-action-conventions)<ul><li>AddFolderRouteModelConvention</li><li>AddPageRouteModelConvention</li><li>AddPageRoute</li></ul> | ルート テンプレートをフォルダー内のページおよび単一ページに追加します。 |
| [ページ モデル アクション規則](#page-model-action-conventions)<ul><li>AddFolderApplicationModelConvention</li><li>AddPageApplicationModelConvention</li><li>ConfigureFilter (フィルター クラス、ラムダ式、またはフィルター ファクトリ)</li></ul> | ヘッダーをフォルダー内のページに追加し、ヘッダーを単一ページに追加し、ヘッダーをアプリのページに追加するように[フィルター ファクトリ](xref:mvc/controllers/filters#ifilterfactory)を構成します。 |

Razor ページの規則を追加および構成するには、`Startup` クラス内のサービス コレクションの <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddMvc*> に <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.AddRazorPagesOptions*> 拡張メソッドを使用します。 次の規則の例は、このトピックで後述されます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddRazorPagesOptions(options =>
        {
            options.Conventions.Add( ... );
            options.Conventions.AddFolderRouteModelConvention(
                "/OtherPages", model => { ... });
            options.Conventions.AddPageRouteModelConvention(
                "/About", model => { ... });
            options.Conventions.AddPageRoute(
                "/Contact", "TheContactPage/{text?}");
            options.Conventions.AddFolderApplicationModelConvention(
                "/OtherPages", model => { ... });
            options.Conventions.AddPageApplicationModelConvention(
                "/About", model => { ... });
            options.Conventions.ConfigureFilter(model => { ... });
            options.Conventions.ConfigureFilter( ... );
        });
}
```

## <a name="route-order"></a>ルートの順番

ルートは、処理 (ルートの照合) に使われる順番 (<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel.Order*>) を指定します。

| 順番            | 動作 |
| :--------------: | -------- |
| -1               | ルートは、他のルートが処理される前に処理されます。 |
| 0                | 順番が指定されていません (既定値)。 `Order` を割り当てない (`Order = null`) 場合、処理に使われるルートの `Order` は既定で 0 (ゼロ) になります。 |
| 1、2、&hellip; n | ルートの処理順序を指定します。 |

ルートの処理は、次の規則で定められています。

* ルートは並んだ順番に処理されます (-1、0、1、2、&hellip; n)。
* ルートの `Order` が同じ場合は、最初に最も明確なルートが照合され、続けて次に明確なルートが照合されます。
* `Order` とパラメーター数が同じルートが 1 つの要求 URL と一致した場合、ルートは <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection> に追加された順番で処理されます。

可能であれば、定められたルートの処理順序に依存しないようにしてください。 通常は、ルーティングによって、URL が一致する正しいルートが選択されます。 要求が正しくルーティングされるようにルートの `Order` プロパティを設定する必要がある場合、アプリのルーティング方式がクライアントにとって紛らわしいものとなり、保守が脆弱になる可能性があります。 アプリのルーティング方式を簡略化するよう努めてください。 サンプル アプリでは、1 つのアプリで複数のルーティング シナリオを示すために、ルートの明示的な処理順序が必要です。 ただし、運用環境のアプリでルートの `Order` を設定するやり方は避けるようにしてください。

Razor Pages ルーティングと MVC コントローラー ルーティングは、実装を共有します。 MVC のトピックに記載されているルートの順番については、[コントローラー アクションへのルーティング: 属性ルートの順序の指定](xref:mvc/controllers/routing#ordering-attribute-routes)に関するページをご覧ください。

## <a name="model-conventions"></a>モデルの規則

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageConvention> の委任を追加すると、Razor Pages に適用される[モデルの規則](xref:mvc/controllers/application-model#conventions)を追加できます。

### <a name="add-a-route-model-convention-to-all-pages"></a>すべてのページにルート モデル規則を追加する

<xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions.Conventions> を使用すると、<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageRouteModelConvention> を作成して、ページ ルート モデルの構築中に適用される <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageConvention> インスタンスのコレクションに追加できます。

サンプル アプリでは、`{globalTemplate?}` ルート テンプレートをアプリ内のすべてのページに追加します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Conventions/GlobalTemplatePageRouteModelConvention.cs?name=snippet1)]

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel> の <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel.Order*> プロパティに `1` が設定されます。 これにより、サンプル アプリで次のルートの照合動作が保証されます。

* `TheContactPage/{text?}` のルート テンプレートは、このトピックの後半で追加されます。 連絡先ページのルートには既定の順番 `null` (`Order = 0`) が設定されているため、`{globalTemplate?}` ルート テンプレートの前に一致します。
* `{aboutTemplate?}` ルート テンプレートは、このトピックの後半で追加されます。 `{aboutTemplate?}` テンプレートの `Order` には `2` が指定されます。 [About] ページが `/About/RouteDataValue` で要求されると、"RouteDataValue" は `RouteData.Values["globalTemplate"]` (`Order = 1`) に読み込まれ、`Order` プロパティが設定されるため `RouteData.Values["aboutTemplate"]` (`Order = 2`) には読み込まれません。
* `{otherPagesTemplate?}` ルート テンプレートは、このトピックの後半で追加されます。 `{otherPagesTemplate?}` テンプレートの `Order` には `2` が指定されます。 ルート パラメーター (`/OtherPages/Page1/RouteDataValue` など) を使用して *Pages/OtherPages* フォルダー内のいずれかのページが要求されると、`Order` プロパティが設定されているため、"RouteDataValue" が `RouteData.Values["otherPagesTemplate"]` (`Order = 2`) ではなく `RouteData.Values["globalTemplate"]` (`Order = 1`) に読み込まれます。

可能な限り、`Order` を設定しないでください。これにより、`Order = 0` が生じます。 正しいルートの選択には、ルーティングを使用してください。

Razor Pages のオプション (<xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions.Conventions> の追加など) は、MVC が `Startup.ConfigureServices` のサービス コレクションに追加されたときに追加されます。 例については、[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/razor-pages-conventions/samples/)を参照してください。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet1)]

`localhost:5000/About/GlobalRouteValue` でサンプルの [About] ページを要求し、その結果を調べます。

![[About] ページは、GlobalRouteValue のルート セグメントで要求されます。 レンダリングされたページは、ルート データの値がページの OnGet メソッドでキャプチャされたことを示しています。](razor-pages-conventions/_static/about-page-global-template.png)

### <a name="add-an-app-model-convention-to-all-pages"></a>すべてのページにアプリ モデル規則を追加する

<xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions.Conventions> を使用すると、<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageApplicationModelConvention> を作成して、ページ アプリ モデルの構築中に適用される <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageConvention> インスタンスのコレクションに追加できます。

この規則やこのトピックで後述されるその他の規則のデモを実行するには、サンプル アプリに `AddHeaderAttribute` クラスを含めます。 クラス コンストラクターは、`name` 文字列と `values` 文字列配列を受け入れます。 これらの値は、応答ヘッダーを設定するために、その `OnResultExecuting` メソッド内で使用されます。 完全クラスは、このトピックで後述される「[ページ モデル アクション規則](#page-model-action-conventions)」セクションで示されます。

サンプル アプリでは、`AddHeaderAttribute` クラスを使用して、ヘッダー (`GlobalHeader`) をアプリ内のすべてのページに追加します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Conventions/GlobalHeaderPageApplicationModelConvention.cs?name=snippet1)]

*Startup.cs*:

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet2)]

`localhost:5000/About` でサンプルの [About] ページを要求し、そのヘッダーを調べて結果を確認します。

![[About] ページの応答ヘッダーは、GlobalHeader が追加されたことを示しています。](razor-pages-conventions/_static/about-page-global-header.png)

### <a name="add-a-handler-model-convention-to-all-pages"></a>すべてのページにハンドラー モデル規則を追加する

<xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions.Conventions> を使用すると、<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageHandlerModelConvention> を作成して、ページ ハンドラー モデルの構築中に適用される <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageConvention> インスタンスのコレクションに追加できます。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Conventions/GlobalPageHandlerModelConvention.cs?name=snippet1)]

*Startup.cs*:

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet10)]

## <a name="page-route-action-conventions"></a>ページ ルート アクション規則

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageRouteModelProvider> から派生する既定のルート モデル プロバイダーは、ページ ルートを構成するための拡張ポイントを提供するようにデザインされた規則を呼び出します。

### <a name="folder-route-model-convention"></a>フォルダー ルート モデル規則

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection.AddFolderRouteModelConvention*> を使用すると、指定したフォルダーの下にあるすべてのページを対象に <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageRouteModel> へのアクションを呼び出す <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageRouteModelConvention> を作成して追加できます。

サンプル アプリでは <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection.AddFolderRouteModelConvention*> を使用して、`{otherPagesTemplate?}` ルート テンプレートを *OtherPages* フォルダーのページに追加します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet3)]

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel> の <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel.Order*> プロパティに `2` が設定されます。 この設定により、1 つのルート値が指定されたときに、(このトピックの前半で `1` に設定した) `{globalTemplate?}` のテンプレートが優先的に最初のルート データ値の位置に指定されます。 ルート パラメーター値 (`/OtherPages/Page1/RouteDataValue` など) を使用して *Pages/OtherPages* フォルダー内のページが要求されると、`Order` プロパティが設定されているため、"RouteDataValue" が `RouteData.Values["otherPagesTemplate"]` (`Order = 2`) ではなく `RouteData.Values["globalTemplate"]` (`Order = 1`) に読み込まれます。

可能な限り、`Order` を設定しないでください。これにより、`Order = 0` が生じます。 正しいルートの選択には、ルーティングを使用してください。

`localhost:5000/OtherPages/Page1/GlobalRouteValue/OtherPagesRouteValue` でサンプルの Page1 ページを要求し、その結果を調べます。

![OtherPages フォルダーの Page1 は、GlobalRouteValue と OtherPagesRouteValue のルート セグメントで要求されます。 レンダリングされたページは、ルート データの値がページの OnGet メソッドでキャプチャされたことを示しています。](razor-pages-conventions/_static/otherpages-page1-global-and-otherpages-templates.png)

### <a name="page-route-model-convention"></a>ページ ルート モデル規則

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection.AddPageRouteModelConvention*> を使用すると、指定した名前のページを対象に <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageRouteModel> へのアクションを呼び出す <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageRouteModelConvention> を作成して追加できます。

サンプル アプリでは `AddPageRouteModelConvention` を使用して、`{aboutTemplate?}` ルート テンプレートを [About] ページに追加します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet4)]

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel> の <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel.Order*> プロパティに `2` が設定されます。 この設定により、1 つのルート値が指定されたときに、(このトピックの前半で `1` に設定した) `{globalTemplate?}` のテンプレートが優先的に最初のルート データ値の位置に指定されます。 [About] ページが `/About/RouteDataValue` にあるルート パラメーター値で要求されると、`Order` プロパティが設定されているため、"RouteDataValue" は `RouteData.Values["aboutTemplate"]` (`Order = 2`) ではなく `RouteData.Values["globalTemplate"]` (`Order = 1`) に読み込まれます。

可能な限り、`Order` を設定しないでください。これにより、`Order = 0` が生じます。 正しいルートの選択には、ルーティングを使用してください。

`localhost:5000/About/GlobalRouteValue/AboutRouteValue` でサンプルの [About] ページを要求し、その結果を調べます。

![[About] ページは、GlobalRouteValue と AboutRouteValue のルート セグメントで要求されます。 レンダリングされたページは、ルート データの値がページの OnGet メソッドでキャプチャされたことを示しています。](razor-pages-conventions/_static/about-page-global-and-about-templates.png)

## <a name="use-a-parameter-transformer-to-customize-page-routes"></a>パラメーター トランスフォーマーを使用してページ ルートをカスタマイズする

ASP.NET Core によって生成されたページ ルートは、パラメーター トランスフォーマーを使用してカスタマイズできます。 パラメーター トランスフォーマーは `IOutboundParameterTransformer` を実装し、パラメーターの値を変換します。 たとえば、`SlugifyParameterTransformer` パラメーター トランスフォーマーでは、`SubscriptionManagement` のルート値が `subscription-management` に変更されます。

`PageRouteTransformerConvention` ページ ルート モデル規則では、アプリで自動的に生成されたページ ルートのフォルダー名とファイル名のセグメントにパラメーター トランスフォーマーを適用します。 たとえば、 */Pages/SubscriptionManagement/ViewAll.cshtml* にある Razor ページ ファイルのルートは `/SubscriptionManagement/ViewAll` から `/subscription-management/view-all` に書き換えられます。

`PageRouteTransformerConvention` では、Razor ページのフォルダー名とファイル名に由来する、自動的に生成されたページ ルートのセグメントのみを変換します。 `@page` ディレクティブを使用して追加したルート セグメントは変換されません。 この規則では、<xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AddPageRoute*> で追加したルートも変換されません。

`PageRouteTransformerConvention` は、`Startup.ConfigureServices` でオプションとして登録されます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddRazorPagesOptions(options =>
        {
            options.Conventions.Add(
                new PageRouteTransformerConvention(
                    new SlugifyParameterTransformer()));
        });
}

public class SlugifyParameterTransformer : IOutboundParameterTransformer
{
    public string TransformOutbound(object value)
    {
        if (value == null) { return null; }

        // Slugify value
        return Regex.Replace(value.ToString(), "([a-z])([A-Z])", "$1-$2").ToLower();
    }
}
```

## <a name="configure-a-page-route"></a>ページ ルートの構成

<xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AddPageRoute*> を使用すると、指定したページ パスにあるページへのルートを構成できます。 そのページに対して生成されたリンクでは、指定したルートを使用します。 `AddPageRoute` では、`AddPageRouteModelConvention` を使用してルートを確立します。

サンプル アプリでは、*Contact.cshtml* の `/TheContactPage` へのルートを作成します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet5)]

[Contact] ページには、既定のルート経由の `/Contact` でアクセスすることもできます。

サンプル アプリの [Contact] ページに対するカスタム ルートでは、省略可能な `text` ルート セグメント (`{text?}`) を許可します。 また、訪問者が `/Contact` ルートでページにアクセスする場合、ページの `@page` ディレクティブにはこの省略可能なセグメントも含まれます。

[!code-cshtml[](razor-pages-conventions/samples/2.x/SampleApp/Pages/Contact.cshtml?highlight=1)]

レンダリングされたページの **Contact** リンク用に生成された URL には、更新されたルートが反映されることに注意してください。

![ナビゲーション バーのサンプル アプリの [Contact] リンク](razor-pages-conventions/_static/contact-link.png)

![レンダリングされた HTML の [Contact] リンクを調べると、href に '/TheContactPage' が設定されています](razor-pages-conventions/_static/contact-link-source.png)

通常のルート (`/Contact`) またはカスタム ルート (`/TheContactPage`) のいずれかで、[Contact] ページにアクセスします。 追加の `text` ルート セグメントを指定した場合、ページには指定した HTML エンコードのセグメントが示されます。

![URL の 'TextValue' の省略可能な 'text' ルート セグメントを適用する Microsoft Edge ブラウザーの例。 レンダリングされたページには、'text' セグメントの値が示されています。](razor-pages-conventions/_static/route-segment-with-custom-route.png)

## <a name="page-model-action-conventions"></a>ページ モデル アクション規則

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageApplicationModelProvider> を実装する既定のページ モデル プロバイダーは、ページ モデルを構成するための拡張ポイントを提供するようにデザインされた規則を呼び出します。 これらの規則は、ページ検出をビルドおよび変更したり、シナリオを処理したりするときに便利です。

このセクションの例の場合、サンプル アプリでは、応答ヘッダーを適用する <xref:Microsoft.AspNetCore.Mvc.Filters.ResultFilterAttribute> である、`AddHeaderAttribute` クラスを使用します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Filters/AddHeader.cs?name=snippet1)]

規則を使用して、このサンプルでは、フォルダー内のすべてのページおよび単一ページに属性を適用する方法のデモを実行します。

**フォルダー アプリ モデル規則**

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection.AddFolderApplicationModelConvention*> を使用すると、指定したフォルダーの下にあるすべてのページを対象に <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageApplicationModel> インスタンスへのアクションを呼び出す <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageApplicationModelConvention> を作成して追加できます。

このサンプルでは、ヘッダー (`OtherPagesHeader`) をアプリの *OtherPages* フォルダー内にあるページに追加して、`AddFolderApplicationModelConvention` を使用するデモを実行します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet6)]

`localhost:5000/OtherPages/Page1` でサンプルの Page1 ページを要求し、そのヘッダーを調べて結果を確認します。

![OtherPages/Page1 ページの応答ヘッダーは、OtherPagesHeader が追加されていることを示しています。](razor-pages-conventions/_static/page1-otherpages-header.png)

**ページ アプリ モデル規則**

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection.AddPageApplicationModelConvention*> を使用すると、指定した名前のページを対象に <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageApplicationModel> へのアクションを呼び出す <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageApplicationModelConvention> を作成して追加できます。

サンプルでは、ヘッダー (`AboutHeader`) を [About] ページに追加して、`AddPageApplicationModelConvention` を使用するデモを実行します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet7)]

`localhost:5000/About` でサンプルの [About] ページを要求し、そのヘッダーを調べて結果を確認します。

![[About] ページの応答ヘッダーは、AboutHeader が追加されたことを示しています。](razor-pages-conventions/_static/about-page-about-header.png)

**フィルターの構成**

<xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.ConfigureFilter*> では、指定したフィルターを適用するように構成します。 フィルター クラスを実装できますが、サンプル アプリでは、ラムダ式でフィルターを実装する方法を示しています。これは、フィルターを返すファクトリとしてバックグラウンドで実装されます。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet8)]

ページ アプリ モデルは、*OtherPages* フォルダーの Page2 ページにつながるセグメントの相対パスを確認するために使用されます。 条件を満たすと、ヘッダーが追加されます。 満たさない場合は、`EmptyFilter` が適用されます。

`EmptyFilter` は[アクション フィルター](xref:mvc/controllers/filters#action-filters)です。 アクション フィルターは Razor Pages によって無視されるため、パスに `OtherPages/Page2` が含まれない場合、`EmptyFilter` には意図されたような効果はありません。

`localhost:5000/OtherPages/Page2` でサンプルの Page2 ページを要求し、そのヘッダーを調べて結果を確認します。

![OtherPagesPage2Header は、Page2 への応答に追加されます。](razor-pages-conventions/_static/page2-filter-header.png)

**フィルター ファクトリの構成**

<xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.ConfigureFilter*> では、すべての Razor Pages に[フィルター](xref:mvc/controllers/filters)を適用するように、指定したファクトリを構成します。

サンプル アプリでは、アプリのページに対する 2 つの値と共にヘッダー (`FilterFactoryHeader`) を追加して、[フィルター ファクトリ](xref:mvc/controllers/filters#ifilterfactory)を使用する例を提供します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet9)]

*AddHeaderWithFactory.cs*:

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Factories/AddHeaderWithFactory.cs?name=snippet1)]

`localhost:5000/About` でサンプルの [About] ページを要求し、そのヘッダーを調べて結果を確認します。

![[About] ページの応答ヘッダーは、FilterFactoryHeader ヘッダーが 2 つ追加されたことを示しています。](razor-pages-conventions/_static/about-page-filter-factory-header.png)

## <a name="mvc-filters-and-the-page-filter-ipagefilter"></a>MVC フィルターとページ フィルター (IPageFilter)

Razor Pages はハンドラー メソッドを使用するため、MVC [アクション フィルター](xref:mvc/controllers/filters#action-filters)は Razor Pages によって無視されます。 使用できるその他の種類の MVC フィルターは次のとおりです。[承認](xref:mvc/controllers/filters#authorization-filters)、[例外](xref:mvc/controllers/filters#exception-filters)、[リソース](xref:mvc/controllers/filters#resource-filters)、および[結果](xref:mvc/controllers/filters#result-filters)。 詳細については、「[フィルター](xref:mvc/controllers/filters)」トピックを参照してください。

ページ フィルター (<xref:Microsoft.AspNetCore.Mvc.Filters.IPageFilter>) は、Razor Pages に適用されるフィルターの 1 つです。 詳細については、[Razor Pages のフィルター メソッド](xref:razor-pages/filter)に関するページを参照してください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:security/authorization/razor-pages-authorization>
* <xref:mvc/controllers/areas#areas-with-razor-pages>

::: moniker-end

::: moniker range="< aspnetcore-2.2"

[ページ ルートとアプリ モデル プロバイダーの規則](xref:mvc/controllers/application-model#conventions)を使用して、Razor ページ アプリでページのルーティング、検出、および処理を制御する方法について説明します。

個別のページにカスタムのページ ルートを構成する必要がある場合、このトピックで後述される「[AddPageRoute convention](#configure-a-page-route)」で、ページへのルーティングを構成します。

ページ ルートの指定、ルート セグメントの追加、ルートへのパラメーターの追加を行うには、ページの `@page` ディレクティブを使用します。 詳しくは、「[カスタム ルート](xref:razor-pages/index#custom-routes)」をご覧ください。

ルート セグメントやパラメーター名として使用できない予約語がいくつかあります。 詳しくは、[ルーティング: ルーティングの予約名](xref:fundamentals/routing#reserved-routing-names)に関するページをご覧ください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/razor-pages-conventions/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

| シナリオ | このサンプルでは、次のデモを実行します。 |
| -------- | --------------------------- |
| [モデルの規則](#model-conventions)<br><br>Conventions.Add<ul><li>IPageRouteModelConvention</li><li>IPageApplicationModelConvention</li><li>IPageHandlerModelConvention</li></ul> | ルート テンプレートとヘッダーをアプリのページに追加します。 |
| [ページ ルート アクション規則](#page-route-action-conventions)<ul><li>AddFolderRouteModelConvention</li><li>AddPageRouteModelConvention</li><li>AddPageRoute</li></ul> | ルート テンプレートをフォルダー内のページおよび単一ページに追加します。 |
| [ページ モデル アクション規則](#page-model-action-conventions)<ul><li>AddFolderApplicationModelConvention</li><li>AddPageApplicationModelConvention</li><li>ConfigureFilter (フィルター クラス、ラムダ式、またはフィルター ファクトリ)</li></ul> | ヘッダーをフォルダー内のページに追加し、ヘッダーを単一ページに追加し、ヘッダーをアプリのページに追加するように[フィルター ファクトリ](xref:mvc/controllers/filters#ifilterfactory)を構成します。 |

Razor ページの規則を追加および構成するには、`Startup` クラス内のサービス コレクションの <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddMvc*> に <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.AddRazorPagesOptions*> 拡張メソッドを使用します。 次の規則の例は、このトピックで後述されます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddRazorPagesOptions(options =>
        {
            options.Conventions.Add( ... );
            options.Conventions.AddFolderRouteModelConvention(
                "/OtherPages", model => { ... });
            options.Conventions.AddPageRouteModelConvention(
                "/About", model => { ... });
            options.Conventions.AddPageRoute(
                "/Contact", "TheContactPage/{text?}");
            options.Conventions.AddFolderApplicationModelConvention(
                "/OtherPages", model => { ... });
            options.Conventions.AddPageApplicationModelConvention(
                "/About", model => { ... });
            options.Conventions.ConfigureFilter(model => { ... });
            options.Conventions.ConfigureFilter( ... );
        });
}
```

## <a name="route-order"></a>ルートの順番

ルートは、処理 (ルートの照合) に使われる順番 (<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel.Order*>) を指定します。

| 順番            | 動作 |
| :--------------: | -------- |
| -1               | ルートは、他のルートが処理される前に処理されます。 |
| 0                | 順番が指定されていません (既定値)。 `Order` を割り当てない (`Order = null`) 場合、処理に使われるルートの `Order` は既定で 0 (ゼロ) になります。 |
| 1、2、&hellip; n | ルートの処理順序を指定します。 |

ルートの処理は、次の規則で定められています。

* ルートは並んだ順番に処理されます (-1、0、1、2、&hellip; n)。
* ルートの `Order` が同じ場合は、最初に最も明確なルートが照合され、続けて次に明確なルートが照合されます。
* `Order` とパラメーター数が同じルートが 1 つの要求 URL と一致した場合、ルートは <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection> に追加された順番で処理されます。

可能であれば、定められたルートの処理順序に依存しないようにしてください。 通常は、ルーティングによって、URL が一致する正しいルートが選択されます。 要求が正しくルーティングされるようにルートの `Order` プロパティを設定する必要がある場合、アプリのルーティング方式がクライアントにとって紛らわしいものとなり、保守が脆弱になる可能性があります。 アプリのルーティング方式を簡略化するよう努めてください。 サンプル アプリでは、1 つのアプリで複数のルーティング シナリオを示すために、ルートの明示的な処理順序が必要です。 ただし、運用環境のアプリでルートの `Order` を設定するやり方は避けるようにしてください。

Razor Pages ルーティングと MVC コントローラー ルーティングは、実装を共有します。 MVC のトピックに記載されているルートの順番については、[コントローラー アクションへのルーティング: 属性ルートの順序の指定](xref:mvc/controllers/routing#ordering-attribute-routes)に関するページをご覧ください。

## <a name="model-conventions"></a>モデルの規則

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageConvention> の委任を追加すると、Razor Pages に適用される[モデルの規則](xref:mvc/controllers/application-model#conventions)を追加できます。

### <a name="add-a-route-model-convention-to-all-pages"></a>すべてのページにルート モデル規則を追加する

<xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions.Conventions> を使用すると、<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageRouteModelConvention> を作成して、ページ ルート モデルの構築中に適用される <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageConvention> インスタンスのコレクションに追加できます。

サンプル アプリでは、`{globalTemplate?}` ルート テンプレートをアプリ内のすべてのページに追加します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Conventions/GlobalTemplatePageRouteModelConvention.cs?name=snippet1)]

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel> の <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel.Order*> プロパティに `1` が設定されます。 これにより、サンプル アプリで次のルートの照合動作が保証されます。

* `TheContactPage/{text?}` のルート テンプレートは、このトピックの後半で追加されます。 連絡先ページのルートには既定の順番 `null` (`Order = 0`) が設定されているため、`{globalTemplate?}` ルート テンプレートの前に一致します。
* `{aboutTemplate?}` ルート テンプレートは、このトピックの後半で追加されます。 `{aboutTemplate?}` テンプレートの `Order` には `2` が指定されます。 [About] ページが `/About/RouteDataValue` で要求されると、"RouteDataValue" は `RouteData.Values["globalTemplate"]` (`Order = 1`) に読み込まれ、`Order` プロパティが設定されるため `RouteData.Values["aboutTemplate"]` (`Order = 2`) には読み込まれません。
* `{otherPagesTemplate?}` ルート テンプレートは、このトピックの後半で追加されます。 `{otherPagesTemplate?}` テンプレートの `Order` には `2` が指定されます。 ルート パラメーター (`/OtherPages/Page1/RouteDataValue` など) を使用して *Pages/OtherPages* フォルダー内のいずれかのページが要求されると、`Order` プロパティが設定されているため、"RouteDataValue" が `RouteData.Values["otherPagesTemplate"]` (`Order = 2`) ではなく `RouteData.Values["globalTemplate"]` (`Order = 1`) に読み込まれます。

可能な限り、`Order` を設定しないでください。これにより、`Order = 0` が生じます。 正しいルートの選択には、ルーティングを使用してください。

Razor Pages のオプション (<xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions.Conventions> の追加など) は、MVC が `Startup.ConfigureServices` のサービス コレクションに追加されたときに追加されます。 例については、[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/razor-pages-conventions/samples/)を参照してください。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet1)]

`localhost:5000/About/GlobalRouteValue` でサンプルの [About] ページを要求し、その結果を調べます。

![[About] ページは、GlobalRouteValue のルート セグメントで要求されます。 レンダリングされたページは、ルート データの値がページの OnGet メソッドでキャプチャされたことを示しています。](razor-pages-conventions/_static/about-page-global-template.png)

### <a name="add-an-app-model-convention-to-all-pages"></a>すべてのページにアプリ モデル規則を追加する

<xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions.Conventions> を使用すると、<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageApplicationModelConvention> を作成して、ページ アプリ モデルの構築中に適用される <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageConvention> インスタンスのコレクションに追加できます。

この規則やこのトピックで後述されるその他の規則のデモを実行するには、サンプル アプリに `AddHeaderAttribute` クラスを含めます。 クラス コンストラクターは、`name` 文字列と `values` 文字列配列を受け入れます。 これらの値は、応答ヘッダーを設定するために、その `OnResultExecuting` メソッド内で使用されます。 完全クラスは、このトピックで後述される「[ページ モデル アクション規則](#page-model-action-conventions)」セクションで示されます。

サンプル アプリでは、`AddHeaderAttribute` クラスを使用して、ヘッダー (`GlobalHeader`) をアプリ内のすべてのページに追加します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Conventions/GlobalHeaderPageApplicationModelConvention.cs?name=snippet1)]

*Startup.cs*:

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet2)]

`localhost:5000/About` でサンプルの [About] ページを要求し、そのヘッダーを調べて結果を確認します。

![[About] ページの応答ヘッダーは、GlobalHeader が追加されたことを示しています。](razor-pages-conventions/_static/about-page-global-header.png)

### <a name="add-a-handler-model-convention-to-all-pages"></a>すべてのページにハンドラー モデル規則を追加する

<xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions.Conventions> を使用すると、<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageHandlerModelConvention> を作成して、ページ ハンドラー モデルの構築中に適用される <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageConvention> インスタンスのコレクションに追加できます。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Conventions/GlobalPageHandlerModelConvention.cs?name=snippet1)]

*Startup.cs*:

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet10)]

## <a name="page-route-action-conventions"></a>ページ ルート アクション規則

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageRouteModelProvider> から派生する既定のルート モデル プロバイダーは、ページ ルートを構成するための拡張ポイントを提供するようにデザインされた規則を呼び出します。

### <a name="folder-route-model-convention"></a>フォルダー ルート モデル規則

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection.AddFolderRouteModelConvention*> を使用すると、指定したフォルダーの下にあるすべてのページを対象に <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageRouteModel> へのアクションを呼び出す <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageRouteModelConvention> を作成して追加できます。

サンプル アプリでは <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection.AddFolderRouteModelConvention*> を使用して、`{otherPagesTemplate?}` ルート テンプレートを *OtherPages* フォルダーのページに追加します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet3)]

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel> の <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel.Order*> プロパティに `2` が設定されます。 この設定により、1 つのルート値が指定されたときに、(このトピックの前半で `1` に設定した) `{globalTemplate?}` のテンプレートが優先的に最初のルート データ値の位置に指定されます。 ルート パラメーター値 (`/OtherPages/Page1/RouteDataValue` など) を使用して *Pages/OtherPages* フォルダー内のページが要求されると、`Order` プロパティが設定されているため、"RouteDataValue" が `RouteData.Values["otherPagesTemplate"]` (`Order = 2`) ではなく `RouteData.Values["globalTemplate"]` (`Order = 1`) に読み込まれます。

可能な限り、`Order` を設定しないでください。これにより、`Order = 0` が生じます。 正しいルートの選択には、ルーティングを使用してください。

`localhost:5000/OtherPages/Page1/GlobalRouteValue/OtherPagesRouteValue` でサンプルの Page1 ページを要求し、その結果を調べます。

![OtherPages フォルダーの Page1 は、GlobalRouteValue と OtherPagesRouteValue のルート セグメントで要求されます。 レンダリングされたページは、ルート データの値がページの OnGet メソッドでキャプチャされたことを示しています。](razor-pages-conventions/_static/otherpages-page1-global-and-otherpages-templates.png)

### <a name="page-route-model-convention"></a>ページ ルート モデル規則

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection.AddPageRouteModelConvention*> を使用すると、指定した名前のページを対象に <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageRouteModel> へのアクションを呼び出す <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageRouteModelConvention> を作成して追加できます。

サンプル アプリでは `AddPageRouteModelConvention` を使用して、`{aboutTemplate?}` ルート テンプレートを [About] ページに追加します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet4)]

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel> の <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.AttributeRouteModel.Order*> プロパティに `2` が設定されます。 この設定により、1 つのルート値が指定されたときに、(このトピックの前半で `1` に設定した) `{globalTemplate?}` のテンプレートが優先的に最初のルート データ値の位置に指定されます。 [About] ページが `/About/RouteDataValue` にあるルート パラメーター値で要求されると、`Order` プロパティが設定されているため、"RouteDataValue" は `RouteData.Values["aboutTemplate"]` (`Order = 2`) ではなく `RouteData.Values["globalTemplate"]` (`Order = 1`) に読み込まれます。

可能な限り、`Order` を設定しないでください。これにより、`Order = 0` が生じます。 正しいルートの選択には、ルーティングを使用してください。

`localhost:5000/About/GlobalRouteValue/AboutRouteValue` でサンプルの [About] ページを要求し、その結果を調べます。

![[About] ページは、GlobalRouteValue と AboutRouteValue のルート セグメントで要求されます。 レンダリングされたページは、ルート データの値がページの OnGet メソッドでキャプチャされたことを示しています。](razor-pages-conventions/_static/about-page-global-and-about-templates.png)

## <a name="configure-a-page-route"></a>ページ ルートの構成

<xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AddPageRoute*> を使用すると、指定したページ パスにあるページへのルートを構成できます。 そのページに対して生成されたリンクでは、指定したルートを使用します。 `AddPageRoute` では、`AddPageRouteModelConvention` を使用してルートを確立します。

サンプル アプリでは、*Contact.cshtml* の `/TheContactPage` へのルートを作成します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet5)]

[Contact] ページには、既定のルート経由の `/Contact` でアクセスすることもできます。

サンプル アプリの [Contact] ページに対するカスタム ルートでは、省略可能な `text` ルート セグメント (`{text?}`) を許可します。 また、訪問者が `/Contact` ルートでページにアクセスする場合、ページの `@page` ディレクティブにはこの省略可能なセグメントも含まれます。

[!code-cshtml[](razor-pages-conventions/samples/2.x/SampleApp/Pages/Contact.cshtml?highlight=1)]

レンダリングされたページの **Contact** リンク用に生成された URL には、更新されたルートが反映されることに注意してください。

![ナビゲーション バーのサンプル アプリの [Contact] リンク](razor-pages-conventions/_static/contact-link.png)

![レンダリングされた HTML の [Contact] リンクを調べると、href に '/TheContactPage' が設定されています](razor-pages-conventions/_static/contact-link-source.png)

通常のルート (`/Contact`) またはカスタム ルート (`/TheContactPage`) のいずれかで、[Contact] ページにアクセスします。 追加の `text` ルート セグメントを指定した場合、ページには指定した HTML エンコードのセグメントが示されます。

![URL の 'TextValue' の省略可能な 'text' ルート セグメントを適用する Microsoft Edge ブラウザーの例。 レンダリングされたページには、'text' セグメントの値が示されています。](razor-pages-conventions/_static/route-segment-with-custom-route.png)

## <a name="page-model-action-conventions"></a>ページ モデル アクション規則

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageApplicationModelProvider> を実装する既定のページ モデル プロバイダーは、ページ モデルを構成するための拡張ポイントを提供するようにデザインされた規則を呼び出します。 これらの規則は、ページ検出をビルドおよび変更したり、シナリオを処理したりするときに便利です。

このセクションの例の場合、サンプル アプリでは、応答ヘッダーを適用する <xref:Microsoft.AspNetCore.Mvc.Filters.ResultFilterAttribute> である、`AddHeaderAttribute` クラスを使用します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Filters/AddHeader.cs?name=snippet1)]

規則を使用して、このサンプルでは、フォルダー内のすべてのページおよび単一ページに属性を適用する方法のデモを実行します。

**フォルダー アプリ モデル規則**

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection.AddFolderApplicationModelConvention*> を使用すると、指定したフォルダーの下にあるすべてのページを対象に <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageApplicationModel> インスタンスへのアクションを呼び出す <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageApplicationModelConvention> を作成して追加できます。

このサンプルでは、ヘッダー (`OtherPagesHeader`) をアプリの *OtherPages* フォルダー内にあるページに追加して、`AddFolderApplicationModelConvention` を使用するデモを実行します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet6)]

`localhost:5000/OtherPages/Page1` でサンプルの Page1 ページを要求し、そのヘッダーを調べて結果を確認します。

![OtherPages/Page1 ページの応答ヘッダーは、OtherPagesHeader が追加されていることを示しています。](razor-pages-conventions/_static/page1-otherpages-header.png)

**ページ アプリ モデル規則**

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection.AddPageApplicationModelConvention*> を使用すると、指定した名前のページを対象に <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageApplicationModel> へのアクションを呼び出す <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IPageApplicationModelConvention> を作成して追加できます。

サンプルでは、ヘッダー (`AboutHeader`) を [About] ページに追加して、`AddPageApplicationModelConvention` を使用するデモを実行します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet7)]

`localhost:5000/About` でサンプルの [About] ページを要求し、そのヘッダーを調べて結果を確認します。

![[About] ページの応答ヘッダーは、AboutHeader が追加されたことを示しています。](razor-pages-conventions/_static/about-page-about-header.png)

**フィルターの構成**

<xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.ConfigureFilter*> では、指定したフィルターを適用するように構成します。 フィルター クラスを実装できますが、サンプル アプリでは、ラムダ式でフィルターを実装する方法を示しています。これは、フィルターを返すファクトリとしてバックグラウンドで実装されます。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet8)]

ページ アプリ モデルは、*OtherPages* フォルダーの Page2 ページにつながるセグメントの相対パスを確認するために使用されます。 条件を満たすと、ヘッダーが追加されます。 満たさない場合は、`EmptyFilter` が適用されます。

`EmptyFilter` は[アクション フィルター](xref:mvc/controllers/filters#action-filters)です。 アクション フィルターは Razor Pages によって無視されるため、パスに `OtherPages/Page2` が含まれない場合、`EmptyFilter` には意図されたような効果はありません。

`localhost:5000/OtherPages/Page2` でサンプルの Page2 ページを要求し、そのヘッダーを調べて結果を確認します。

![OtherPagesPage2Header は、Page2 への応答に追加されます。](razor-pages-conventions/_static/page2-filter-header.png)

**フィルター ファクトリの構成**

<xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.ConfigureFilter*> では、すべての Razor Pages に[フィルター](xref:mvc/controllers/filters)を適用するように、指定したファクトリを構成します。

サンプル アプリでは、アプリのページに対する 2 つの値と共にヘッダー (`FilterFactoryHeader`) を追加して、[フィルター ファクトリ](xref:mvc/controllers/filters#ifilterfactory)を使用する例を提供します。

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Startup.cs?name=snippet9)]

*AddHeaderWithFactory.cs*:

[!code-csharp[](razor-pages-conventions/samples/2.x/SampleApp/Factories/AddHeaderWithFactory.cs?name=snippet1)]

`localhost:5000/About` でサンプルの [About] ページを要求し、そのヘッダーを調べて結果を確認します。

![[About] ページの応答ヘッダーは、FilterFactoryHeader ヘッダーが 2 つ追加されたことを示しています。](razor-pages-conventions/_static/about-page-filter-factory-header.png)

## <a name="mvc-filters-and-the-page-filter-ipagefilter"></a>MVC フィルターとページ フィルター (IPageFilter)

Razor Pages はハンドラー メソッドを使用するため、MVC [アクション フィルター](xref:mvc/controllers/filters#action-filters)は Razor Pages によって無視されます。 使用できるその他の種類の MVC フィルターは次のとおりです。[承認](xref:mvc/controllers/filters#authorization-filters)、[例外](xref:mvc/controllers/filters#exception-filters)、[リソース](xref:mvc/controllers/filters#resource-filters)、および[結果](xref:mvc/controllers/filters#result-filters)。 詳細については、「[フィルター](xref:mvc/controllers/filters)」トピックを参照してください。

ページ フィルター (<xref:Microsoft.AspNetCore.Mvc.Filters.IPageFilter>) は、Razor Pages に適用されるフィルターの 1 つです。 詳細については、[Razor Pages のフィルター メソッド](xref:razor-pages/filter)に関するページを参照してください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:security/authorization/razor-pages-authorization>
* <xref:mvc/controllers/areas#areas-with-razor-pages>

::: moniker-end
