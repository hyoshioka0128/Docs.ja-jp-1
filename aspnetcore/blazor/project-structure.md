---
title: ASP.NET Core Blazor プロジェクトの構造
author: guardrex
description: ASP.NET Core Blazor アプリ プロジェクトの構造について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/19/2021
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
uid: blazor/project-structure
ms.openlocfilehash: fe42c2d43b79ea959bb0ba8e5b96e6c865b2a416
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/06/2021
ms.locfileid: "102394877"
---
# <a name="aspnet-core-blazor-project-structure"></a>ASP.NET Core Blazor プロジェクトの構造

この記事では、Blazor フレームワークのいずれかのプロジェクト テンプレートから生成された Blazor アプリを構成するファイルとフォルダーについて説明します。 ツールを使用して Blazor プロジェクト テンプレートから Blazor アプリを作成する方法については、「<xref:blazor/tooling>」を参照してください。 Blazor のホスティング モデルである Blazor WebAssembly と Blazor Server の詳細については、「<xref:blazor/hosting-models>」を参照してください。

## Blazor WebAssembly

Blazor WebAssembly プロジェクト テンプレート: `blazorwasm`

Blazor WebAssembly テンプレートによって、Blazor WebAssembly アプリの初期ファイルとディレクトリの構造が作成されます。 このアプリには、静的資産からデータを読み込む `FetchData` コンポーネント、`weather.json`、および `Counter` コンポーネントとのユーザーの対話のためのデモンストレーション コードが設定されます。

* `Pages` フォルダー:Blazor アプリを構成するルーティング可能なコンポーネントまたはページ (`.razor`) が含まれています。 各ページのルートは、[`@page`](xref:mvc/views/razor#page) ディレクティブを使用して指定します。 テンプレートには、以下のコンポーネントが含まれています。
  * `Counter` コンポーネント (`Counter.razor`): カウンター ページを実装します。
  * `FetchData` コンポーネント (`FetchData.razor`): フェッチ データ ページを実装します。
  * `Index` コンポーネント (`Index.razor`): ホーム ページを実装します。
  
* `Properties/launchSettings.json`:[開発環境の構成](xref:fundamentals/environments#development-and-launchsettingsjson)を保持します。

::: moniker range=">= aspnetcore-5.0"

* `Shared` フォルダー:次の共有コンポーネントおよびスタイルシートが含まれています。
  * `MainLayout` コンポーネント (`MainLayout.razor`): アプリの[レイアウト コンポーネント](xref:blazor/layouts)。
  * `MainLayout.razor.css`: アプリのメイン レイアウト用のスタイルシート。
  * `NavMenu` コンポーネント (`NavMenu.razor`): サイドバー ナビゲーションを実装します。 ナビゲーション リンクを他の Razor コンポーネントに表示する [`NavLink` コンポーネント](xref:blazor/fundamentals/routing#navlink-and-navmenu-components) (<xref:Microsoft.AspNetCore.Components.Routing.NavLink>) が含まれます。 <xref:Microsoft.AspNetCore.Components.Routing.NavLink> コンポーネントは、そのコンポーネントが読み込まれると、自動的に選択された状態を示します。これは、ユーザーが現在どのコンポーネントが表示されているかを理解するために役立ちます。
  * `NavMenu.razor.css`: アプリのナビゲーション メニュー用のスタイルシート。
  * `SurveyPrompt` コンポーネント (`SurveyPrompt.razor`): Blazor のアンケート用コンポーネント。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* `Shared` フォルダー:次の共有コンポーネントが含まれています。
  * `MainLayout` コンポーネント (`MainLayout.razor`): アプリの[レイアウト コンポーネント](xref:blazor/layouts)。
  * `NavMenu` コンポーネント (`NavMenu.razor`): サイドバー ナビゲーションを実装します。 ナビゲーション リンクを他の Razor コンポーネントに表示する [`NavLink` コンポーネント](xref:blazor/fundamentals/routing#navlink-and-navmenu-components) (<xref:Microsoft.AspNetCore.Components.Routing.NavLink>) が含まれます。 <xref:Microsoft.AspNetCore.Components.Routing.NavLink> コンポーネントは、そのコンポーネントが読み込まれると、自動的に選択された状態を示します。これは、ユーザーが現在どのコンポーネントが表示されているかを理解するために役立ちます。
  * `SurveyPrompt` コンポーネント (`SurveyPrompt.razor`): Blazor のアンケート用コンポーネント。
  
::: moniker-end

::: moniker range=">= aspnetcore-5.0"

* `wwwroot`: アプリのパブリックな静的資産が含まれる、アプリの [Web ルート](xref:fundamentals/index#web-root) フォルダー。[構成設定](xref:blazor/fundamentals/configuration)のための `appsettings.json` と環境アプリ設定ファイルなどが含まれます。 `index.html` Web ページは、HTML ページとして実装されるアプリのルート ページです。
  * アプリのいずれかのページが最初に要求されると、このページが表示されて応答として返されます。
  * このページは、ルート `App` コンポーネントを表示する場所を指定します。 コンポーネントは、`app` の `id` (`<div id="app">Loading...</div>`) を持つ `div` DOM 要素の位置にレンダリングされます。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* `wwwroot`: アプリのパブリックな静的資産が含まれる、アプリの [Web ルート](xref:fundamentals/index#web-root) フォルダー。[構成設定](xref:blazor/fundamentals/configuration)のための `appsettings.json` と環境アプリ設定ファイルなどが含まれます。 `index.html` Web ページは、HTML ページとして実装されるアプリのルート ページです。
  * アプリのいずれかのページが最初に要求されると、このページが表示されて応答として返されます。
  * このページは、ルート `App` コンポーネントを表示する場所を指定します。 コンポーネントは `app` DOM 要素 (`<app>Loading...</app>`) の位置に表示されます。

::: moniker-end

> [!NOTE]
> `wwwroot/index.html` ファイルに追加する JavaScript (JS) ファイルは、終了タグ `</body>` の前に配置する必要があります。 JS ファイルからカスタム JS コードが読み込まれる順序は、一部のシナリオで重要となります。 たとえば、相互運用メソッドを含む JS ファイルは、Blazor フレームワークの JS ファイルよりも先に含めるようにします。

* `_Imports.razor`:名前空間の [`@using`](xref:mvc/views/razor#using) ディレクティブなど、アプリのコンポーネント (`.razor`) に含める一般的な Razor ディレクティブが含まれます。

* `App.razor`:<xref:Microsoft.AspNetCore.Components.Routing.Router> コンポーネントを使用してクライアント側のルーティングを設定するアプリのルート コンポーネント。 <xref:Microsoft.AspNetCore.Components.Routing.Router> コンポーネントは、ブラウザーのナビゲーションをインターセプトし、要求されたアドレスに一致するページをレンダリングします。

::: moniker range=">= aspnetcore-5.0"

* `Program.cs`: WebAssembly ホストを設定するアプリのエントリ ポイントです。
  
  * `App` コンポーネントは、アプリのルート コンポーネントです。 `App` コンポーネントは、ルート コンポーネント コレクション (`builder.RootComponents.Add<App>("#app")`) に対する `app` の `id` (`wwwroot/index.html` の `<div id="app">Loading...</div>`) を持つ `div` DOM 要素として指定されます。
  * [サービス](xref:blazor/fundamentals/dependency-injection)が追加され、構成されます (例: `builder.Services.AddSingleton<IMyDependency, MyDependency>()`)。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* `Program.cs`: WebAssembly ホストを設定するアプリのエントリ ポイントです。

  * `App` コンポーネントは、アプリのルート コンポーネントです。 `App` コンポーネントは、ルート コンポーネント コレクション (`builder.RootComponents.Add<App>("app")`) に対する `app` DOM 要素 (`wwwroot/index.html` の `<app>Loading...</app>`) として指定されます。
  * [サービス](xref:blazor/fundamentals/dependency-injection)が追加され、構成されます (例: `builder.Services.AddSingleton<IMyDependency, MyDependency>()`)。

::: moniker-end

## Blazor Server

Blazor Server プロジェクト テンプレート: `blazorserver`

Blazor Server テンプレートによって、Blazor Server アプリの初期ファイルとディレクトリの構造が作成されます。 このアプリには、登録済みサービスからデータを読み込む `FetchData` コンポーネント、`WeatherForecastService`、および `Counter` コンポーネントとのユーザーの対話のためのデモンストレーション コードが設定されます。

* `Data` フォルダー:アプリの `FetchData` コンポーネントに気象データの例を提供する、`WeatherForecast` クラスと `WeatherForecastService` の実装が含まれます。

* `Pages` フォルダー:Blazor アプリと Blazor Server アプリのルート Razor ページを構成するルーティング可能なコンポーネントまたはページ (`.razor`) が含まれています。 各ページのルートは、[`@page`](xref:mvc/views/razor#page) ディレクティブを使用して指定します。 テンプレートには以下が含まれています。
  * `_Host.cshtml`: Razor ページとして実装されるアプリのルート ページ。
    * アプリのいずれかのページが最初に要求されると、このページが表示されて応答として返されます。
    * [ホスト] ページは、ルート `App` コンポーネント (`App.razor`) を表示する場所を指定します。
  * `Counter` コンポーネント (`Counter.razor`): カウンター ページを実装します。
  * `Error` コンポーネント (`Error.razor`): アプリでハンドルされない例外が発生したときに表示されます。
  * `FetchData` コンポーネント (`FetchData.razor`): フェッチ データ ページを実装します。
  * `Index` コンポーネント (`Index.razor`): ホーム ページを実装します。

> [!NOTE]
> `Pages/_Host.cshtml` ファイルに追加する JavaScript (JS) ファイルは、終了タグ `</body>` の前に配置する必要があります。 JS ファイルからカスタム JS コードが読み込まれる順序は、一部のシナリオで重要となります。 たとえば、相互運用メソッドを含む JS ファイルは、Blazor フレームワークの JS ファイルよりも先に含めるようにします。

* `Properties/launchSettings.json`:[開発環境の構成](xref:fundamentals/environments#development-and-launchsettingsjson)を保持します。

::: moniker range=">= aspnetcore-5.0"

* `Shared` フォルダー:次の共有コンポーネントおよびスタイルシートが含まれています。
  * `MainLayout` コンポーネント (`MainLayout.razor`): アプリの[レイアウト コンポーネント](xref:blazor/layouts)。
  * `MainLayout.razor.css`: アプリのメイン レイアウト用のスタイルシート。
  * `NavMenu` コンポーネント (`NavMenu.razor`): サイドバー ナビゲーションを実装します。 ナビゲーション リンクを他の Razor コンポーネントに表示する [`NavLink` コンポーネント](xref:blazor/fundamentals/routing#navlink-and-navmenu-components) (<xref:Microsoft.AspNetCore.Components.Routing.NavLink>) が含まれます。 <xref:Microsoft.AspNetCore.Components.Routing.NavLink> コンポーネントは、そのコンポーネントが読み込まれると、自動的に選択された状態を示します。これは、ユーザーが現在どのコンポーネントが表示されているかを理解するために役立ちます。
  * `NavMenu.razor.css`: アプリのナビゲーション メニュー用のスタイルシート。
  * `SurveyPrompt` コンポーネント (`SurveyPrompt.razor`): Blazor のアンケート用コンポーネント。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* `Shared` フォルダー:次の共有コンポーネントが含まれています。
  * `MainLayout` コンポーネント (`MainLayout.razor`): アプリの[レイアウト コンポーネント](xref:blazor/layouts)。
  * `NavMenu` コンポーネント (`NavMenu.razor`): サイドバー ナビゲーションを実装します。 ナビゲーション リンクを他の Razor コンポーネントに表示する [`NavLink` コンポーネント](xref:blazor/fundamentals/routing#navlink-and-navmenu-components) (<xref:Microsoft.AspNetCore.Components.Routing.NavLink>) が含まれます。 <xref:Microsoft.AspNetCore.Components.Routing.NavLink> コンポーネントは、そのコンポーネントが読み込まれると、自動的に選択された状態を示します。これは、ユーザーが現在どのコンポーネントが表示されているかを理解するために役立ちます。
  * `SurveyPrompt` コンポーネント (`SurveyPrompt.razor`): Blazor のアンケート用コンポーネント。
  
::: moniker-end

* `wwwroot`:アプリのパブリックな静的アセットを含むアプリの [Web ルート](xref:fundamentals/index#web-root) フォルダー。

* `_Imports.razor`:名前空間の [`@using`](xref:mvc/views/razor#using) ディレクティブなど、アプリのコンポーネント (`.razor`) に含める一般的な Razor ディレクティブが含まれます。

* `App.razor`:<xref:Microsoft.AspNetCore.Components.Routing.Router> コンポーネントを使用してクライアント側のルーティングを設定するアプリのルート コンポーネント。 <xref:Microsoft.AspNetCore.Components.Routing.Router> コンポーネントは、ブラウザーのナビゲーションをインターセプトし、要求されたアドレスに一致するページをレンダリングします。

* `appsettings.json` および環境アプリ設定ファイル:アプリの[構成設定](xref:blazor/fundamentals/configuration)を指定します。

* `Program.cs`: ASP.NET Core [ホスト](xref:fundamentals/host/generic-host)を設定するアプリのエントリ ポイント。

* `Startup.cs`: アプリのスタートアップ ロジックを含みます。 `Startup` クラスには、次の 2 つのメソッドがあります。

  * `ConfigureServices`:アプリの[依存関係の挿入 (DI)](xref:fundamentals/dependency-injection)サービスを構成します。 <xref:Microsoft.Extensions.DependencyInjection.ComponentServiceCollectionExtensions.AddServerSideBlazor%2A> を呼び出すことによってサービスが追加されます。`WeatherForecastService` は、サンプルの `FetchData` コンポーネントで使用するためにサービス コンテナーに追加されます。
  * `Configure`:アプリの要求処理パイプラインを構成します。
    * <xref:Microsoft.AspNetCore.Builder.ComponentEndpointRouteBuilderExtensions.MapBlazorHub%2A> は、ブラウザーとのリアルタイム接続用のエンドポイントを設定するために呼び出されます。 この接続は [SignalR](xref:signalr/introduction) で作成されます。これは、アプリにリアルタイム Web 機能を追加するためのフレームワークです。
    * [`MapFallbackToPage("/_Host")`](xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapFallbackToPage*) は、アプリ (`Pages/_Host.cshtml`) のルート ページを設定し、ナビゲーションを有効にするために呼び出されます。

## <a name="additional-resources"></a>その他のリソース

* <xref:blazor/tooling>
* <xref:blazor/hosting-models>
