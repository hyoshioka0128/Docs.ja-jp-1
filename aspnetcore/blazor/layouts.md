---
title: ASP.NET Core Blazor レイアウト
author: guardrex
description: Blazor アプリの再利用可能なレイアウト コンポーネントを作成する方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/02/2021
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
uid: blazor/layouts
ms.openlocfilehash: 9f3400b766a043fdf3872838bffe392eddba4049
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102394949"
---
# <a name="aspnet-core-blazor-layouts"></a>ASP.NET Core Blazor レイアウト

メニュー、著作権メッセージ、会社のロゴなどの一部のアプリ要素は、通常、アプリの全体的なプレゼンテーションの一部です。 これらの要素のマークアップのコピーをアプリのすべてのコンポーネントに配置するのは、効率的ではありません。 これらの要素のいずれかが更新されるたびに、その要素が使用されているすべてのコンポーネントを更新する必要があります。 この方法は維持するのにコストがかかり、更新が行われなかった場合にコンテンツの一貫性が失われるおそれがあります。 "*レイアウト*" を使用することで、これらの問題が解決されます。

Blazor レイアウトとは、それを参照するコンポーネントとマークアップを共有する Razor コンポーネントのことです。 レイアウトでは、[データ バインディング](xref:blazor/components/data-binding)、[依存関係の挿入](xref:blazor/fundamentals/dependency-injection)、およびコンポーネントのその他の機能を使用できます。

## <a name="layout-components"></a>レイアウト コンポーネント

### <a name="create-a-layout-component"></a>レイアウト コンポーネントを作成する

レイアウト コンポーネントを作成するには:

* Razor テンプレートまたは C# コードによって定義された Razor コンポーネントを作成します。 Razor テンプレートが基になっているレイアウト コンポーネントでは、通常の Razor コンポーネントと同じように `.razor` ファイル拡張子が使用されます。 レイアウト コンポーネントはアプリのコンポーネント間で共有されるため、通常はアプリの `Shared` フォルダーに配置されます。 ただし、レイアウトは、それを使用するコンポーネントにアクセスできる任意の場所に配置できます。 たとえば、それを使用するコンポーネントと同じフォルダーに、レイアウトを配置できます。
* コンポーネントを <xref:Microsoft.AspNetCore.Components.LayoutComponentBase> から継承します。 <xref:Microsoft.AspNetCore.Components.LayoutComponentBase> によって、レイアウト内にレンダリングされるコンテンツの <xref:Microsoft.AspNetCore.Components.LayoutComponentBase.Body> プロパティ (<xref:Microsoft.AspNetCore.Components.RenderFragment> 型) が定義されています。
* Razor 構文 `@Body` を使用して、コンテンツがレンダリングされるレイアウト マークアップ内の場所を指定します。

次の `DoctorWhoLayout` コンポーネントには、レイアウト コンポーネント Razor テンプレートが示されています。 レイアウトにより <xref:Microsoft.AspNetCore.Components.LayoutComponentBase> が継承されて、ナビゲーション バー (`<nav>...</nav>`) とフッター (`<footer>...</footer>`) の間に `@Body` が設定されます。

`Shared/DoctorWhoLayout.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/layouts/DoctorWhoLayout.razor?highlight=1,13)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/layouts/DoctorWhoLayout.razor?highlight=1,13)]

::: moniker-end

### <a name="mainlayout-component"></a>`MainLayout` コンポーネント

[Blazor プロジェクト テンプレート](xref:blazor/project-structure)から作成されたアプリでは、`MainLayout` コンポーネントがアプリの[既定のレイアウト](#apply-a-default-layout-to-an-app)です。

`Shared/MainLayout.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/layouts/MainLayout.razor)]

[Blazor の CSS 分離機能](xref:blazor/components/css-isolation)により、分離された CSS スタイルが `MainLayout` コンポーネントに適用されます。 慣例により、スタイルは同じ名前 `Shared/MainLayout.razor.css` の付随するスタイルシートによって提供されます。 スタイルシートの ASP.NET Core フレームワークの実装を、[ASP.NET Core 参照ソース (dotnet/Aspnetcore GitHub リポジトリ)](https://github.com/dotnet/aspnetcore/blob/main/src/ProjectTemplates/Web.ProjectTemplates/content/ComponentsWebAssembly-CSharp/Client/Shared/MainLayout.razor.css) での検査に使用できます。

[!INCLUDE[](~/blazor/includes/aspnetcore-repo-ref-source-links.md)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/layouts/MainLayout.razor)]

::: moniker-end

## <a name="apply-a-layout"></a>レイアウトを適用する

### <a name="apply-a-layout-to-a-component"></a>コンポーネントにレイアウトを適用する

[`@page`](xref:mvc/views/razor#page) ディレクティブが使用されているルーティング可能な Razor コンポーネントにレイアウトを適用するには、[`@layout`](xref:mvc/views/razor#layout) Razor ディレクティブを使用します。 コンパイラにより、`@layout` が <xref:Microsoft.AspNetCore.Components.LayoutAttribute> に変換され、その属性がコンポーネント クラスに適用されます。

次の `Episodes` コンポーネントの内容が、`@Body` の位置にある `DoctorWhoLayout` に挿入されます。

`Pages/Episodes.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/layouts/Episodes.razor?highlight=2)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/layouts/Episodes.razor?highlight=2)]

::: moniker-end

レンダリングされた次の HTML マークアップが、前の `DoctorWhoLayout` および `Episodes` コンポーネントによって生成されます。 関連する 2 つのコンポーネントによって提供されるコンテンツに注目するため、余分なマークアップは示されていません。

* ヘッダー (`<header>...</header>`) の **Doctor Who&trade; Episode Database** という見出し (`<h1>...</h1>`)、ナビゲーション バー (`<nav>...</nav>`)、フッター (`<footer>...</footer>`) の商標情報要素 (`<div>...</div>`) は、`DoctorWhoLayout` コンポーネントによって生成されたものです。
* **Episodes** という見出し (`<h2>...</h2>`) とエピソードの一覧 (`<ul>...</ul>`) は、`Episodes` コンポーネントによって生成されたものです。

```html
<body>
    <div id="app">
        <header>
            <h1>Doctor Who&trade; Episode Database</h1>
        </header>

        <nav>
            <a href="masterlist">Master Episode List</a>
            <a href="search">Search</a>
            <a href="new">Add Episode</a>
        </nav>

        <h2>Episodes</h2>

        <ul>
            <li>...</li>
            <li>...</li>
            <li>...</li>
        </ul>

        <footer>
            Doctor Who is a registered trademark of the BBC. 
            https://www.doctorwho.tv/
        </footer>
    </div>
</body>
```

コンポーネントでレイアウトを直接指定すると、"*既定のレイアウト*" がオーバーライドされます。

* `_Imports` コンポーネント (`_Imports.razor`) からインポートされた `@layout` ディレクティブによって設定されます。次の「[レイアウトをコンポーネントのフォルダーに適用する](#apply-a-layout-to-a-folder-of-components)」セクションの説明を参照してください。
* アプリの既定のレイアウトとして設定します。後の「[アプリに既定のレイアウトを適用する](#apply-a-default-layout-to-an-app)」セクションの説明を参照してください。

### <a name="apply-a-layout-to-a-folder-of-components"></a>レイアウトをコンポーネントのフォルダーに適用する

アプリのすべてのフォルダーには、必要に応じて、`_Imports.razor` という名前のテンプレート ファイルを格納できます。 コンパイラにより、インポート ファイルに指定されたディレクティブが、同じフォルダー内とそのすべてのサブフォルダー内で再帰的にすべての Razor テンプレートに含まれます。 そのため、`@layout DoctorWhoLayout` が含まれる `_Imports.razor` ファイルにより、フォルダー内のすべてのコンポーネントで `DoctorWhoLayout` コンポーネントが確実に使用されます。 フォルダーとサブフォルダー内のすべての Razor コンポーネント (`.razor`) に、`@layout DoctorWhoLayout` を繰り返し追加する必要はありません。

`_Imports.razor`:

```razor
@layout DoctorWhoLayout
...
```

`_Imports.razor` ファイルは、Razor ビューおよびページに対する [_ViewImports.cshtml ファイル](xref:mvc/views/layout#importing-shared-directives)に似ていますが、Razor コンポーネント ファイルに限定して適用されます。

`_Imports.razor` でレイアウトを指定すると、ルーターの[既定のアプリ レイアウト](#apply-a-default-layout-to-an-app)として指定されているレイアウトがオーバーライドされます。これについては、次のセクションで説明します。

> [!WARNING]
> Razor `@layout` ディレクティブをルート `_Imports.razor` ファイルに追加 **しない** でください。レイアウトが無限ループになります。 既定のアプリ レイアウトを制御するには、`Router` コンポーネントでレイアウトを指定します。 詳細については、次の「[アプリに既定のレイアウトを適用する](#apply-a-default-layout-to-an-app)」セクションを参照してください。

> [!NOTE]
> [`@layout`](xref:mvc/views/razor#layout) Razor ディレクティブによってレイアウトが適用されるのは、[`@page`](xref:mvc/views/razor#page) ディレクティブが使用されているルーティング可能な Razor コンポーネントのみです。

### <a name="apply-a-default-layout-to-an-app"></a>アプリに既定のレイアウトを適用する

`App` コンポーネントの <xref:Microsoft.AspNetCore.Components.Routing.Router> コンポーネントで、既定のアプリ レイアウトを指定します。 [Blazor プロジェクト テンプレート](xref:blazor/project-structure)に基づくアプリからの次の例では、既定のレイアウトが `MainLayout` コンポーネントに設定されています。

`App.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/layouts/App1.razor?highlight=3)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/layouts/App1.razor?highlight=3)]

::: moniker-end

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

<xref:Microsoft.AspNetCore.Components.Routing.Router> コンポーネントの詳細については、「<xref:blazor/fundamentals/routing>」を参照してください。

`Router` コンポーネントで既定のレイアウトとしてレイアウトを指定することは、この記事のこれまでのセクションで説明したように、コンポーネントごとまたはフォルダーごとにレイアウトをオーバーライドできるため、便利な方法です。 レイアウトを使用する最も一般的で柔軟な方法であるため、`Router` コンポーネントを使用してアプリの既定のレイアウトを設定することをお勧めします。

### <a name="apply-a-layout-to-arbitrary-content-layoutview-component"></a>任意のコンテンツにレイアウトを適用する (`LayoutView` コンポーネント)

任意の Razor テンプレート コンテンツにレイアウトを設定するには、<xref:Microsoft.AspNetCore.Components.LayoutView> コンポーネントでレイアウトを指定します。 <xref:Microsoft.AspNetCore.Components.LayoutView> は、任意の Razor コンポーネントで使用できます。 次の例では、`MainLayout` コンポーネントの <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> テンプレート (`<NotFound>...</NotFound>`) に `ErrorLayout` という名前のレイアウト コンポーネントを設定しています。

`App.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/layouts/App2.razor?name=snippet&highlight=6,9)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/layouts/App2.razor?name=snippet&highlight=6,9)]

::: moniker-end

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

## <a name="nested-layouts"></a>入れ子になったレイアウト

コンポーネントであるレイアウトを参照し、そこからさらに別のレイアウトを参照することができます。 たとえば、複数レベルのメニュー構造を作成するために、入れ子になったレイアウトを使用します。

次の例に、入れ子になったレイアウトの使用方法を示しています。 「[コンポーネントにレイアウトを適用する](#apply-a-layout-to-a-component)」セクションで示されている `Episodes` コンポーネントは、表示するコンポーネントです。 そのコンポーネントで、`DoctorWhoLayout` コンポーネントが参照されています。

次の `DoctorWhoLayout` コンポーネントは、この記事の前の方で示した例を変更したバージョンです。 ヘッダー要素とフッター要素が削除され、レイアウトで別のレイアウト `ProductionsLayout` が参照されています。 `Episodes` コンポーネントは、`DoctorWhoLayout` 内の `@Body` が出現する場所にレンダリングされます。

`Shared/DoctorWhoLayout.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/layouts/DoctorWhoLayout2.razor?highlight=2,12)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/layouts/DoctorWhoLayout2.razor?highlight=2,12)]

::: moniker-end

`ProductionsLayout` コンポーネントには最上位レベルのレイアウト要素が含まれ、現在はそこにヘッダー要素 (`<header>...</header>`) とフッター要素 (`<footer>...</footer>`) が存在します。 `Episodes` コンポーネントが含まれる `DoctorWhoLayout` は、`@Body` が出現する場所にレンダリングされます。

`Shared/ProductionsLayout.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/layouts/ProductionsLayout.razor?highlight=13)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/layouts/ProductionsLayout.razor?highlight=13)]

::: moniker-end

レンダリングされた次の HTML マークアップが、前の入れ子になったレイアウトによって生成されます。 関連する 3 つのコンポーネントによって提供される入れ子になったコンテンツに注目するため、余分なマークアップは示されていません。

* ヘッダー (`<header>...</header>`)、プロダクション ナビゲーション バー (`<nav>...</nav>`)、フッター (`<footer>...</footer>`) の各要素とその内容は、`ProductionsLayout` コンポーネントから生成されます。
* **Doctor Who&trade; Episode Database** という見出し (`<h1>...</h1>`)、エピソード ナビゲーション バー (`<nav>...</nav>`)、商標情報要素 (`<div>...</div>`) は、`DoctorWhoLayout` コンポーネントから生成されます。
* **Episodes** という見出し (`<h2>...</h2>`) とエピソードの一覧 (`<ul>...</ul>`) は、`Episodes` コンポーネントによって生成されたものです。

```html
<body>
    <div id="app">
        <header>
            <h1>Productions</h1>
        </header>

        <nav>
            <a href="master-production-list">Master Production List</a>
            <a href="production-search">Search</a>
            <a href="new-production">Add Production</a>
        </nav>

        <h1>Doctor Who&trade; Episode Database</h1>

        <nav>
            <a href="episode-masterlist">Master Episode List</a>
            <a href="episode-search">Search</a>
            <a href="new-episode">Add Episode</a>
        </nav>

        <h2>Episodes</h2>

        <ul>
            <li>...</li>
            <li>...</li>
            <li>...</li>
        </ul>

        <div>
            Doctor Who is a registered trademark of the BBC. 
            https://www.doctorwho.tv/
        </div>

        <footer>
            Footer of Productions Layout
        </footer>
    </div>
</body>
```

## <a name="share-a-razor-pages-layout-with-integrated-components"></a>統合コンポーネントと Razor Pages レイアウトを共有する

ルーティング可能なコンポーネントが Razor Pages アプリに統合されている場合、コンポーネントでアプリの共有レイアウトを使用できます。 詳細については、「<xref:blazor/components/prerendering-and-integration>」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:mvc/views/layout>
