---
title: RazorASP.NET Core のアプリケーションパーツを使用して、コントローラー、ビュー、ページなどを共有する
author: rick-anderson
description: RazorASP.NET Core のアプリケーションパーツを使用して、コントローラー、ビュー、ページなどを共有する
ms.author: riande
ms.date: 11/11/2019
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
uid: mvc/extensibility/app-parts
ms.openlocfilehash: 23bc1db6a184e7babe87e2d311a8ac4a59e78dd0
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588361"
---
# <a name="share-controllers-views-razor-pages-and-more-with-application-parts"></a>Razorアプリケーションパーツを使用して、コントローラー、ビュー、ページなどを共有する

::: moniker range=">= aspnetcore-3.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/app-parts)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

"*アプリケーション パーツ*" は、アプリのリソースを抽象化したものです。 アプリケーションパーツを使用すると、コントローラーの探索、コンポーネントの表示、ヘルパー、 Razor ページ、razor コンパイルソースなどの ASP.NET Core を行うことができます。 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.AssemblyPart> はアプリケーション パーツです。 `AssemblyPart` はアセンブリ参照をカプセル化して、型とコンパイル参照を公開します。

"[機能プロバイダー](#fp)" は、アプリケーション パーツと連携し、ASP.NET Core アプリの機能を組み込みます。 アプリケーション パーツの主なユース ケースは、アセンブリから ASP.NET Core 機能を検出 (または読み込みを回避) するようにアプリを構成することです。 たとえば、複数のアプリで共通の機能を共有したいとします。 アプリケーションパーツを使用すると、複数のアプリを使用して、コントローラー、ビュー、 Razor ページ、razor コンパイルソース、タグヘルパーなどを含むアセンブリ (DLL) を共有できます。 複数のプロジェクトでコードを複製するよりもアセンブリの共有をお勧めします。

ASP.NET Core アプリは <xref:System.Web.WebPages.ApplicationPart> から機能を読み込みます。 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.AssemblyPart> クラスは、アセンブリでバックアップされるアプリケーション パーツを表します。

## <a name="load-aspnet-core-features"></a>ASP.NET Core 機能の読み込み

<xref:Microsoft.AspNetCore.Mvc.ApplicationParts> および <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.AssemblyPart> クラスを使用し、ASP.NET Core 機能 (コントローラー、ビュー コンポーネントなど) を検出して読み込みます。 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.ApplicationPartManager> は、使用可能なアプリケーション パーツと機能プロバイダーを追跡します。 `ApplicationPartManager` は `Startup.ConfigureServices` で構成されます。

[!code-csharp[](./app-parts/3.0sample1/WebAppParts/Startup.cs?name=snippet)]

次のコードは、`AssemblyPart` を使用して `ApplicationPartManager` を構成する別の方法を示しています。

[!code-csharp[](./app-parts/3.0sample1/WebAppParts/Startup2.cs?name=snippet)]

上記 2 つのコード サンプルでは、アセンブリから `SharedController` が読み込まれます。 `SharedController` は、アプリのプロジェクト内にありません。 [WebAppParts ソリューション](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/app-parts/3.0sample1/WebAppParts)のサンプル ダウンロードを参照してください。

### <a name="include-views"></a>ビューを含める

[ Razor クラスライブラリ](xref:razor-pages/ui-class)を使用して、アセンブリにビューを含めます。

### <a name="prevent-loading-resources"></a>リソースの読み込みを防ぐ

アプリケーション パーツを使用して、特定のアセンブリまたは場所にあるソースの読み込みを "*回避*" することができます。 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts> コレクションのメンバーを追加または削除して、リソースを非表示にしたり使用可能にしたりします。 `ApplicationParts` コレクションでのエントリの順序は重要ではありません。 `ApplicationPartManager` を構成してから、コンテナーでサービスを構成するために使用します。 たとえば、`AddControllersAsServices` を呼び出す前に `ApplicationPartManager` を構成します。 リソースを削除するには、`ApplicationParts` コレクションに対して `Remove` を呼び出します。

`ApplicationPartManager` には次のパーツが含まれています。

* アプリのアセンブリおよび依存アセンブリ。
* `Microsoft.AspNetCore.Mvc.ApplicationParts.CompiledRazorAssemblyPart`
* `Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation`
* `Microsoft.AspNetCore.Mvc.TagHelpers`.
* `Microsoft.AspNetCore.Mvc.Razor`.

<a name="fp"></a>

## <a name="feature-providers"></a>機能プロバイダー

アプリケーション機能プロバイダーはアプリケーション パーツを調べ、これらのパーツの機能を提供します。 次の ASP.NET Core 機能には組み込みの機能プロバイダーがあります。

* <xref:Microsoft.AspNetCore.Mvc.Controllers.ControllerFeatureProvider>
* <xref:Microsoft.AspNetCore.Mvc.Razor.TagHelpers.TagHelperFeatureProvider>
* <xref:Microsoft.AspNetCore.Mvc.Razor.Compilation.MetadataReferenceFeatureProvider>
* <xref:Microsoft.AspNetCore.Mvc.Razor.Compilation.ViewsFeatureProvider>
* `internal class`[ Razor CompiledItemFeatureProvider](https://github.com/dotnet/AspNetCore/blob/main/src/Mvc/Mvc.Razor/src/ApplicationParts/RazorCompiledItemFeatureProvider.cs#L14)

機能プロバイダーは <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.IApplicationFeatureProvider`1> から継承されます。ここで `T` は機能の種類です。 機能プロバイダーは、前述の機能の種類のいずれについても実装できます。 `ApplicationPartManager.FeatureProviders` の機能プロバイダーの順序が実行時の動作に影響することがあります。 後から追加されたプロバイダーは、前に追加されたプロバイダーによって行われたアクションに対応できます。

### <a name="display-available-features"></a> 使用可能な機能の表示

アプリで使用できる機能を列挙するには、[依存関係の挿入](../../fundamentals/dependency-injection.md)によって `ApplicationPartManager` を要求します。

[!code-csharp[](./app-parts/sample2/AppPartsSample/Controllers/FeaturesController.cs?highlight=16,25-27)]

[ダウンロード サンプル](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/app-parts/sample2)では、前のコードを使用してアプリの機能を表示します。

```text
Controllers:
    - FeaturesController
    - HomeController
    - HelloController
    - GenericController`1
    - GenericController`1
Tag Helpers:
    - PrerenderTagHelper
    - AnchorTagHelper
    - CacheTagHelper
    - DistributedCacheTagHelper
    - EnvironmentTagHelper
    - Additional Tag Helpers omitted for brevity.
View Components:
    - SampleViewComponent
```

## <a name="discovery-in-application-parts"></a>アプリケーション パーツでの検出

アプリケーション パーツを使用した開発では、HTTP 404 エラーが発生することは珍しくありません。 通常、これらのエラーは、アプリケーション パーツの検出に必要な要件が満たされていないことが原因で発生します。 アプリが HTTP 404 エラーを返す場合、次の要件が満たされていることを確認します。

* `applicationName` 設定は、検出に使用されるルート アセンブリに設定されている必要がある。 検出に使用されるルート アセンブリは、通常、エントリ ポイント アセンブリです。
* ルート アセンブリに、検出に使用されるパーツへの参照が設定されている必要がある。 この参照は、直接的な参照でも推移的な参照でも構いません。
* ルート アセンブリは、Web SDK を参照する必要がある。 このフレームワークには、検出に使用されるルート アセンブリに属性のスタンプを付けるロジックがあります。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/app-parts)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

"*アプリケーション パーツ*" は、アプリのリソースを抽象化したものです。 アプリケーションパーツを使用すると、コントローラーの探索、コンポーネントの表示、ヘルパー、 Razor ページ、razor コンパイルソースなどの ASP.NET Core を行うことができます。 [AssemblyPart](/dotnet/api/microsoft.aspnetcore.mvc.applicationparts.assemblypart#Microsoft_AspNetCore_Mvc_ApplicationParts_AssemblyPart) はアプリケーション パーツです。 `AssemblyPart` はアセンブリ参照をカプセル化して、型とコンパイル参照を公開します。

"*機能プロバイダー*" は、アプリケーション パーツと連携し、ASP.NET Core アプリの機能を組み込みます。 アプリケーション パーツの主なユース ケースは、アセンブリから ASP.NET Core 機能を検出 (または読み込みを回避) するようにアプリを構成することです。 たとえば、複数のアプリで共通の機能を共有したいとします。 アプリケーションパーツを使用すると、複数のアプリを使用して、コントローラー、ビュー、 Razor ページ、razor コンパイルソース、タグヘルパーなどを含むアセンブリ (DLL) を共有できます。 複数のプロジェクトでコードを複製するよりもアセンブリの共有をお勧めします。

ASP.NET Core アプリは <xref:System.Web.WebPages.ApplicationPart> から機能を読み込みます。 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.AssemblyPart> クラスは、アセンブリでバックアップされるアプリケーション パーツを表します。

## <a name="load-aspnet-core-features"></a>ASP.NET Core 機能の読み込み

`ApplicationPart` および `AssemblyPart` クラスを使用し、ASP.NET Core 機能 (コントローラー、ビュー コンポーネントなど) を検出して読み込みます。 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.ApplicationPartManager> は、使用可能なアプリケーション パーツと機能プロバイダーを追跡します。 `ApplicationPartManager` は `Startup.ConfigureServices` で構成されます。

[!code-csharp[](./app-parts/sample1/WebAppParts/Startup.cs?name=snippet)]

次のコードは、`AssemblyPart` を使用して `ApplicationPartManager` を構成する別の方法を示しています。

[!code-csharp[](./app-parts/sample1/WebAppParts/Startup2.cs?name=snippet)]

上記 2 つのコード サンプルでは、アセンブリから `SharedController` が読み込まれます。 `SharedController` は、アプリケーションのプロジェクト内にありません。 [WebAppParts ソリューション](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/app-parts/sample1/WebAppParts)のサンプル ダウンロードを参照してください。

### <a name="include-views"></a>ビューを含める

[ Razor クラスライブラリ](xref:razor-pages/ui-class)を使用して、アセンブリにビューを含めます。

### <a name="prevent-loading-resources"></a>リソースの読み込みを防ぐ

アプリケーション パーツを使用して、特定のアセンブリまたは場所にあるソースの読み込みを "*回避*" することができます。 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts> コレクションのメンバーを追加または削除して、リソースを非表示にしたり使用可能にしたりします。 `ApplicationParts` コレクションでのエントリの順序は重要ではありません。 `ApplicationPartManager` を構成してから、コンテナーでサービスを構成するために使用します。 たとえば、`AddControllersAsServices` を呼び出す前に `ApplicationPartManager` を構成します。 リソースを削除するには、`ApplicationParts` コレクションに対して `Remove` を呼び出します。

次のコードでは、<xref:Microsoft.AspNetCore.Mvc.ApplicationParts> を使用してアプリから `MyDependentLibrary` を削除します。[!code-csharp[](./app-parts/sample1/WebAppParts/StartupRm.cs?name=snippet)]

`ApplicationPartManager` には次のパーツが含まれています。

* アプリのアセンブリおよび依存アセンブリ。
* `Microsoft.AspNetCore.Mvc.TagHelpers`.
* `Microsoft.AspNetCore.Mvc.Razor`.

## <a name="application-feature-providers"></a>アプリケーション機能プロバイダー

アプリケーション機能プロバイダーはアプリケーション パーツを調べ、これらのパーツの機能を提供します。 次の ASP.NET Core 機能には組み込みの機能プロバイダーがあります。

* [コントローラー](/dotnet/api/microsoft.aspnetcore.mvc.controllers.controllerfeatureprovider)
* [タグ ヘルパー](/dotnet/api/microsoft.aspnetcore.mvc.razor.taghelpers.taghelperfeatureprovider)
* [コンポーネントの表示](/dotnet/api/microsoft.aspnetcore.mvc.viewcomponents.viewcomponentfeatureprovider)

機能プロバイダーは <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.IApplicationFeatureProvider`1> から継承されます。ここで `T` は機能の種類です。 機能プロバイダーは、前述の機能の種類のいずれについても実装できます。 `ApplicationPartManager.FeatureProviders` の機能プロバイダーの順序が実行時の動作に影響することがあります。 後から追加されたプロバイダーは、前に追加されたプロバイダーによって行われたアクションに対応できます。

### <a name="display-available-features"></a> 使用可能な機能の表示

アプリで使用できる機能を列挙するには、[依存関係の挿入](../../fundamentals/dependency-injection.md)によって `ApplicationPartManager` を要求します。

[!code-csharp[](./app-parts/sample2/AppPartsSample/Controllers/FeaturesController.cs?highlight=16,25-27)]

[ダウンロード サンプル](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/app-parts/sample2)では、前のコードを使用してアプリの機能を表示します。

```text
Controllers:
    - FeaturesController
    - HomeController
    - HelloController
    - GenericController`1
    - GenericController`1
Tag Helpers:
    - PrerenderTagHelper
    - AnchorTagHelper
    - CacheTagHelper
    - DistributedCacheTagHelper
    - EnvironmentTagHelper
    - Additional Tag Helpers omitted for brevity.
View Components:
    - SampleViewComponent
```

## <a name="discovery-in-application-parts"></a>アプリケーション パーツでの検出

アプリケーション パーツを使用した開発では、HTTP 404 エラーが発生することは珍しくありません。 通常、これらのエラーは、アプリケーション パーツの検出に必要な要件が満たされていないことが原因で発生します。 アプリが HTTP 404 エラーを返す場合、次の要件が満たされていることを確認します。

* `applicationName` 設定は、検出に使用されるルート アセンブリに設定されている必要がある。 検出に使用されるルート アセンブリは、通常、エントリ ポイント アセンブリです。
* ルート アセンブリに、検出に使用されるパーツへの参照が設定されている必要がある。 この参照は、直接的な参照でも推移的な参照でも構いません。
* ルート アセンブリは、Web SDK を参照する必要がある。
  * この ASP.NET Core フレームワークには、検出に使用されるルート アセンブリに属性のスタンプを付けるカスタム ビルド ロジックがあります。

::: moniker-end
