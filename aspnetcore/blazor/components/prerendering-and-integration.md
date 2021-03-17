---
title: ASP.NET Core Razor コンポーネントのプリレンダリングと統合を行う
author: guardrex
description: サーバー上の Razor コンポーネントのプリレンダリングを含む Blazor アプリの Razor コンポーネントの統合シナリオについて説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/29/2020
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
uid: blazor/components/prerendering-and-integration
zone_pivot_groups: blazor-hosting-models
ms.openlocfilehash: a0c5cc0bdc78f2ea70b8c128616ad09328ccf87d
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587386"
---
# <a name="prerender-and-integrate-aspnet-core-razor-components"></a>ASP.NET Core Razor コンポーネントのプリレンダリングと統合を行う

::: zone pivot="webassembly"

::: moniker range=">= aspnetcore-5.0"

ホストされる Blazor WebAssembly ソリューションの Razor Pages と MVC アプリに Razor コンポーネントを統合できます。 ページまたはビューがレンダリングされるときには、コンポーネントを同時に事前レンダリングすることができます。

## <a name="configuration"></a>構成

Blazor WebAssembly アプリのプリレンダリングを設定するには:

1. ASP.NET Core アプリで Blazor WebAssembly をホストします。 スタンドアロンの Blazor WebAssembly アプリは、ASP.NET Core に追加することができます。または、[Blazor WebAssembly プロジェクト テンプレート](xref:blazor/project-structure)から作成された、ホストされている Blazor WebAssembly アプリを使用することもできます。

1. 既定の静的な `wwwroot/index.html` ファイルを Blazor WebAssembly クライアント プロジェクトから削除します。

1. クライアント プロジェクトで `Program.Main` 内の次の行を削除します。

   ```csharp
   builder.RootComponents.Add<App>("#app");
   ```

1. サーバー プロジェクトに `Pages/_Host.cshtml` ファイルを追加します。 コマンド シェルの `dotnet new blazorserver -o BlazorServer` コマンドを使用して、Blazor Server テンプレートから作成されたアプリから `_Host.cshtml` ファイルを取得できます。 ホストされる Blazor WebAssembly ソリューションのサーバー アプリに `Pages/_Host.cshtml` ファイルを配置した後、ファイルに次の変更を加えます。

   * サーバー アプリの `Pages` フォルダーに名前空間を設定します (例: `@namespace BlazorHosted.Server.Pages`)。
   * クライアント プロジェクト用の [`@using`](xref:mvc/views/razor#using) ディレクティブを設定します (例: `@using BlazorHosted.Client`)。
   * スタイルシートのリンクを、WebAssembly アプリのスタイルシートをポイントするように更新します。 次の例では、クライアント アプリの名前空間は `BlazorHosted.Client`、になります。

     ```cshtml
     <link href="css/app.css" rel="stylesheet" />
     <link href="BlazorHosted.Client.styles.css" rel="stylesheet" />
     ```

   * [コンポーネント タグ ヘルパー](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper)の `render-mode` を、ルートの `App` コンポーネントをプリレンダリングするように更新します。

     ```cshtml
     <component type="typeof(App)" render-mode="WebAssemblyPrerendered" />
     ```

   * Blazor スクリプトのソースを、クライアント側の Blazor WebAssembly スクリプトを使用するように更新します。

     ```cshtml
     <script src="_framework/blazor.webassembly.js"></script>
     ```

1. サーバー プロジェクトの `Startup.Configure` (`Startup.cs`) で:

   * 開発環境で、アプリ ビルダーで `UseDeveloperExceptionPage` を呼び出します。
   * アプリ ビルダーで `UseBlazorFrameworkFiles` を呼び出します。
   * フォールバックを `index.html` ファイル (`endpoints.MapFallbackToFile("index.html");`) から `_Host.cshtml` ページ (`endpoints.MapFallbackToPage("/_Host");`) に変更します。

   ```csharp
   public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
   {
       if (env.IsDevelopment())
       {
           app.UseDeveloperExceptionPage();
           app.UseWebAssemblyDebugging();
       }
       else
       {
           app.UseExceptionHandler("/Error");
           app.UseHsts();
       }

       app.UseHttpsRedirection();
       app.UseBlazorFrameworkFiles();
       app.UseStaticFiles();

       app.UseRouting();

       app.UseEndpoints(endpoints =>
       {
           endpoints.MapRazorPages();
           endpoints.MapControllers();
           endpoints.MapFallbackToPage("/_Host");
       });
   }
   ```

## <a name="render-components-in-a-page-or-view-with-the-component-tag-helper"></a>コンポーネント タグ ヘルパーを使用してページまたはビューのコンポーネントをレンダリングする

コンポーネント タグ ヘルパーでは、ページまたはビューに Blazor WebAssembly アプリからコンポーネントをレンダリングするための 2 つのレンダリング モードがサポートされています。

* `WebAssembly`: ブラウザーに読み込まれるときに対話型コンポーネントを含めるために使用される Blazor WebAssembly アプリ用のマーカーをレンダリングします。 コンポーネントはプリレンダリングされません。 このオプションを使用すると、異なる Blazor WebAssembly コンポーネントを異なるページにレンダリングしやすくなります。
* `WebAssemblyPrerendered`: コンポーネントを静的 HTML にプリレンダリングし、ブラウザーに読み込まれるときにコンポーネントを対話型にするために後で使用される Blazor WebAssembly アプリ用のマーカーを含めます。

次の Razor Pages の例では、`Counter` コンポーネントがページにレンダリングされます。 コンポーネントを対話形式にするために、ページの[レンダリング セクション](xref:mvc/views/layout#sections)に Blazor WebAssembly スクリプトが含まれます。 コンポーネント タグ ヘルパー (`{APP ASSEMBLY}.Pages.Counter`) による `Counter` コンポーネントの完全な名前空間の使用を避けるために、クライアント アプリの `Pages` 名前空間に [`@using`](xref:mvc/views/razor#using) ディレクティブを追加します。 次の例では、クライアント アプリの名前空間は `BlazorHosted.Client`、になります。

```cshtml
...
@using BlazorHosted.Client.Pages

<h1>@ViewData["Title"]</h1>

<component type="typeof(Counter)" render-mode="WebAssemblyPrerendered" />

@section Scripts {
    <script src="_framework/blazor.webassembly.js"></script>
}
```

<xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> によって、コンポーネントに対して以下の構成が行われます。

* ページに事前レンダリングするかどうか。
* ページに静的 HTML としてレンダリングするかどうか。または、ユーザー エージェントから Blazor アプリをブートストラップするために必要な情報が含まれているかどうか。

パラメーターの渡し方や <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> の構成などのコンポーネント タグ ヘルパーの詳細については、「<xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>」を参照してください。

前の例では、サーバー アプリのレイアウト (`_Layout.cshtml`) の `</body>` 終了タグの中に、スクリプトの[レンダリング セクション](xref:mvc/views/layout#sections) (<xref:Microsoft.AspNetCore.Mvc.Razor.RazorPage.RenderSection%2A>) が含まれている必要があります。

```cshtml
    ...

    @RenderSection("Scripts", required: false)
</body>
```

`_Layout.cshtml` ファイルは、Razor Pages アプリの `Pages/Shared` フォルダーまたは MVC アプリの `Views/Shared` フォルダーにあります。

アプリで Blazor WebAssembly アプリのスタイルを使用してコンポーネントのスタイルを指定する必要がある場合は、アプリのスタイルを `_Layout.cshtml` ファイルに含めます。 次の例では、クライアント アプリの名前空間は `BlazorHosted.Client`、になります。

```cshtml
<head>
    ...

    <link href="css/app.css" rel="stylesheet" />
    <link href="BlazorHosted.Client.styles.css" rel="stylesheet" />
</head>
```

## <a name="render-components-in-a-page-or-view-with-a-css-selector"></a>CSS セレクターを使用してページまたはビューのコンポーネントをレンダリングする

ルートコンポーネントを `Program.Main` (`Program.cs`) の "*クライアント*" プロジェクトに追加します。 次の例では、`my-counter` と一致する `id` を持つ要素を選択する CSS セレクターを使用して、ルート コンポーネントとして `Counter` コンポーネントが宣言されています。 次の例では、クライアント アプリの名前空間は `BlazorHosted.Client`、になります。

```csharp
using BlazorHosted.Client.Pages;

...

builder.RootComponents.Add<Counter>("#my-counter");
```

次の Razor Pages の例では、`Counter` コンポーネントがページにレンダリングされます。 コンポーネントを対話形式にするために、ページの[レンダリング セクション](xref:mvc/views/layout#sections)に Blazor WebAssembly スクリプトが含まれています。

```cshtml
...

<h1>@ViewData["Title"]</h1>

<div id="my-counter">Loading...</div>

@section Scripts {
    <script src="_framework/blazor.webassembly.js"></script>
}
```

前の例では、サーバー アプリのレイアウト (`_Layout.cshtml`) の `</body>` 終了タグの中に、スクリプトの[レンダリング セクション](xref:mvc/views/layout#sections) (<xref:Microsoft.AspNetCore.Mvc.Razor.RazorPage.RenderSection%2A>) が含まれている必要があります。

```cshtml
    ...

    @RenderSection("Scripts", required: false)
</body>
```

`_Layout.cshtml` ファイルは、Razor Pages アプリの `Pages/Shared` フォルダーまたは MVC アプリの `Views/Shared` フォルダーにあります。

アプリで Blazor WebAssembly アプリのスタイルを使用してコンポーネントのスタイルを指定する必要がある場合は、アプリのスタイルを `_Layout.cshtml` ファイルに含めます。 次の例では、クライアント アプリの名前空間は `BlazorHosted.Client`、になります。

```cshtml
<head>
    ...

    <link href="css/app.css" rel="stylesheet" />
    <link href="BlazorHosted.Client.styles.css" rel="stylesheet" />
</head>
```

## <a name="additional-resources"></a>その他のリソース

* [認証を使用したプリレンダリングのサポート](xref:blazor/security/webassembly/additional-scenarios#support-prerendering-with-authentication)

::: moniker-end

::: moniker range="< aspnetcore-5.0"

ホストされる Blazor WebAssembly ソリューションの Razor Pages と MVC アプリへの Razor コンポーネントの統合は、.NET 5 以降の ASP.NET Core でサポートされています。 この記事の .NET 5 以降のバージョンを選択してください。

::: moniker-end

::: zone-end

::: zone pivot="server"

Razor コンポーネントは、Blazor Server アプリ内の Razor Pages と MVC アプリに統合できます。 ページまたはビューがレンダリングされるときには、コンポーネントを同時に事前レンダリングすることができます。

[アプリを構成](#configuration)した後、アプリの要件に応じて、次のセクションのガイダンスを使用します。

* ルーティング可能なコンポーネント:ユーザー要求から直接ルーティング可能なコンポーネント。 訪問者が [`@page`](xref:mvc/views/razor#page) ディレクティブを含むコンポーネントのブラウザーで HTTP 要求を行うことができる必要がある場合は、このガイダンスに従ってください。
  * [Razor Pages アプリでルーティング可能なコンポーネントを使用する](#use-routable-components-in-a-razor-pages-app)
  * [MVC アプリでルーティング可能なコンポーネントを使用する](#use-routable-components-in-an-mvc-app)
* [ページまたはビューからコンポーネントをレンダリングする](#render-components-from-a-page-or-view):ユーザー要求から直接ルーティングできないコンポーネント。 アプリによって[コンポーネント タグ ヘルパー](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper)を含む既存のページやビューにコンポーネントが埋め込まれる場合は、このガイダンスに従ってください。

## <a name="configuration"></a>構成

既存の Razor Pages や MVC アプリでは、Razor コンポーネントをページとビューに統合できます。

1. アプリのレイアウト ファイル (`_Layout.cshtml`) で:

   * 次の `<base>` タグを `<head>` 要素に追加します。

     ```html
     <base href="~/" />
     ```

     前の例の `href` 値 (*アプリ ベースのパス*) は、アプリがルート URL パス (`/`) に置かれていることを前提としています。 アプリがサブアプリケーションになっている場合は、記事 <xref:blazor/host-and-deploy/index#app-base-path> の「*アプリのベース パス*」セクションのガイダンスに従ってください。

     `_Layout.cshtml` ファイルは、Razor Pages アプリの `Pages/Shared` フォルダーまたは MVC アプリの `Views/Shared` フォルダーにあります。

   * `blazor.server.js` スクリプトの `<script>` タグを `Scripts` レンダリング セクションの直前に追加します。

     ```html
         ...
         <script src="_framework/blazor.server.js"></script>

         @await RenderSectionAsync("Scripts", required: false)
     </body>
     ```

     フレームワークによって `blazor.server.js` スクリプトがアプリに追加されます。 手動でアプリにスクリプトを追加する必要はありません。

1. 次の内容を含む `_Imports.razor` ファイルをプロジェクトのルート フォルダーに追加します (最後の名前空間 `MyAppNamespace` をアプリの名前空間に変更します)。

   ```razor
   @using System.Net.Http
   @using Microsoft.AspNetCore.Authorization
   @using Microsoft.AspNetCore.Components.Authorization
   @using Microsoft.AspNetCore.Components.Forms
   @using Microsoft.AspNetCore.Components.Routing
   @using Microsoft.AspNetCore.Components.Web
   @using Microsoft.JSInterop
   @using MyAppNamespace
   ```

1. `Startup.ConfigureServices` で、Blazor Server サービスを登録します。

   ```csharp
   services.AddServerSideBlazor();
   ```

1. `Startup.Configure` で、Blazor Hub エンドポイントを `app.UseEndpoints` に追加します。

   ```csharp
   endpoints.MapBlazorHub();
   ```

1. コンポーネントを任意のページまたはビューに統合します。 詳細については、「[ページまたはビューからコンポーネントをレンダリングする](#render-components-from-a-page-or-view)」セクションを参照してください。

## <a name="use-routable-components-in-a-razor-pages-app"></a>Razor Pages アプリでルーティング可能なコンポーネントを使用する

*ここは、ユーザー要求から直接ルーティング可能なコンポーネントを追加することに関係のあるセクションです。*

Razor Pages アプリでルーティング可能な Razor コンポーネントをサポートするには、次のようにします。

1. 「[構成](#configuration)」セクションのガイダンスに従います。

1. 次の内容の `App.razor` ファイルをプロジェクト ルートに追加します。

   ```razor
   @using Microsoft.AspNetCore.Components.Routing

   <Router AppAssembly="@typeof(Program).Assembly">
       <Found Context="routeData">
           <RouteView RouteData="routeData" />
       </Found>
       <NotFound>
           <h1>Page not found</h1>
           <p>Sorry, but there's nothing here!</p>
       </NotFound>
   </Router>
   ```

   [!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

1. 次の内容の `_Host.cshtml` ファイルを `Pages` フォルダーに追加します。

   ```cshtml
   @page "/blazor"
   @{
       Layout = "_Layout";
   }

   <app>
       <component type="typeof(App)" render-mode="ServerPrerendered" />
   </app>
   ```

   コンポーネントでは、そのレイアウトで共有される `_Layout.cshtml` ファイルが使用されます。

   <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> によって、`App` コンポーネントに対して以下の構成が行われます。

   * ページに事前レンダリングするかどうか。
   * ページに静的 HTML としてレンダリングするかどうか。または、ユーザー エージェントから Blazor アプリをブートストラップするために必要な情報が含まれているかどうか。

   パラメーターの渡し方や <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> の構成などのコンポーネント タグ ヘルパーの詳細については、「<xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>」を参照してください。

1. `_Host.cshtml` ページの優先度が低いルートを、`Startup.Configure` 内のエンドポイント構成に追加します。

   ```csharp
   app.UseEndpoints(endpoints =>
   {
       ...

       endpoints.MapFallbackToPage("/_Host");
   });
   ```

1. ルーティング可能なコンポーネントをアプリに追加します。 次に例を示します。

   ```razor
   @page "/counter"

   <h1>Counter</h1>

   ...
   ```

名前空間の詳細については、「[コンポーネントの名前空間](#component-namespaces)」セクションを参照してください。

## <a name="use-routable-components-in-an-mvc-app"></a>MVC アプリでルーティング可能なコンポーネントを使用する

*ここは、ユーザー要求から直接ルーティング可能なコンポーネントを追加することに関係のあるセクションです。*

MVC アプリでルーティング可能な Razor コンポーネントをサポートするには、次のようにします。

1. 「[構成](#configuration)」セクションのガイダンスに従います。

1. 次の内容の `App.razor` ファイルを、プロジェクトのルートに追加します。

   ```razor
   @using Microsoft.AspNetCore.Components.Routing

   <Router AppAssembly="@typeof(Program).Assembly">
       <Found Context="routeData">
           <RouteView RouteData="routeData" />
       </Found>
       <NotFound>
           <h1>Page not found</h1>
           <p>Sorry, but there's nothing here!</p>
       </NotFound>
   </Router>
   ```

   [!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

1. 次の内容の `_Host.cshtml` ファイルを `Views/Home` フォルダーに追加します。

   ```cshtml
   @{
       Layout = "_Layout";
   }

   <app>
       <component type="typeof(App)" render-mode="ServerPrerendered" />
   </app>
   ```

   コンポーネントでは、そのレイアウトで共有される `_Layout.cshtml` ファイルが使用されます。

   <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> によって、`App` コンポーネントに対して以下の構成が行われます。

   * ページに事前レンダリングするかどうか。
   * ページに静的 HTML としてレンダリングするかどうか。または、ユーザー エージェントから Blazor アプリをブートストラップするために必要な情報が含まれているかどうか。

   パラメーターの渡し方や <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> の構成などのコンポーネント タグ ヘルパーの詳細については、「<xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>」を参照してください。

1. Home コントローラーにアクションを追加します。

   ```csharp
   public IActionResult Blazor()
   {
      return View("_Host");
   }
   ```

1. `Startup.Configure` 内のエンドポイント構成に `_Host.cshtml` ビューを返すコントローラー アクションのために、優先度が低いルートを追加します。

   ```csharp
   app.UseEndpoints(endpoints =>
   {
       ...

       endpoints.MapFallbackToController("Blazor", "Home");
   });
   ```

1. `Pages` フォルダーを作成し、アプリにルーティング可能なコンポーネントを追加します。 次に例を示します。

   ```razor
   @page "/counter"

   <h1>Counter</h1>

   ...
   ```

名前空間の詳細については、「[コンポーネントの名前空間](#component-namespaces)」セクションを参照してください。

## <a name="render-components-from-a-page-or-view"></a>ページまたはビューからコンポーネントをレンダリングする

*これは、コンポーネントをユーザー要求から直接ルーティングできないページまたはビューにコンポーネントを追加することに関係するセクションです。*

ページまたはビューからコンポーネントをレンダリングするには、[コンポーネント タグ ヘルパー](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper)を使用します。

### <a name="render-stateful-interactive-components"></a>ステートフル対話型コンポーネントをレンダリングする

Razor ページまたはビューには、ステートフル対話型コンポーネントを追加できます。

ページまたはビューがレンダリングされると、次の処理が行われます。

* ページまたはビューと共にコンポーネントがプリレンダリングされます。
* プリレンダリングに使用された初期のコンポーネント状態は失われます。
* SignalR 接続が確立されると、新しいコンポーネント状態が作成されます。

次の Razor ページには、`Counter` コンポーネントがレンダリングされます。

```cshtml
<h1>My Razor Page</h1>

<component type="typeof(Counter)" render-mode="ServerPrerendered" 
    param-InitialValue="InitialValue" />

@functions {
    [BindProperty(SupportsGet=true)]
    public int InitialValue { get; set; }
}
```

詳細については、「<xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>」を参照してください。

### <a name="render-noninteractive-components"></a>非対話型コンポーネントをレンダリングする

次の Razor ページには、フォームを使用して指定された初期値を使用して、`Counter` コンポーネントが静的にレンダリングされます。 コンポーネントは静的にレンダリングされるため、コンポーネントは対話型ではありません。

```cshtml
<h1>My Razor Page</h1>

<form>
    <input type="number" asp-for="InitialValue" />
    <button type="submit">Set initial value</button>
</form>

<component type="typeof(Counter)" render-mode="Static" 
    param-InitialValue="InitialValue" />

@functions {
    [BindProperty(SupportsGet=true)]
    public int InitialValue { get; set; }
}
```

詳細については、「<xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>」を参照してください。

## <a name="component-namespaces"></a>コンポーネントの名前空間

カスタム フォルダーを使用してアプリのコンポーネントを保持する場合は、フォルダーを表す名前空間を、ページまたはビューのいずれかに追加するか、`_ViewImports.cshtml` ファイルに追加します。 次に例を示します。

* `MyAppNamespace` をアプリの名前空間に変更します。
* コンポーネントを保持するために `Components` という名前のフォルダーを使用していない場合は、`Components` を、コンポーネントが置かれているフォルダーに変更します。

```cshtml
@using MyAppNamespace.Components
```

`_ViewImports.cshtml` ファイルは、Razor Pages アプリの `Pages` フォルダーまたは MVC アプリの `Views` フォルダーにあります。

詳細については、「<xref:blazor/components/index#namespaces>」を参照してください。

::: zone-end
