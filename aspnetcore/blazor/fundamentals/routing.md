---
title: ASP.NET Core Blazor のルーティング
author: guardrex
description: アプリで要求ルーティングを管理する方法と、Blazor アプリでナビゲーション用に NavLink コンポーネントを使用する方法について学習します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/09/2020
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
uid: blazor/fundamentals/routing
ms.openlocfilehash: 74886eae7431045e56de535b5221040bd56cdc2d
ms.sourcegitcommit: 610936e4d3507f7f3d467ed7859ab9354ec158ba
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/25/2021
ms.locfileid: "98751569"
---
# <a name="aspnet-core-blazor-routing"></a>ASP.NET Core Blazor のルーティング

作成者: [Luke Latham](https://github.com/guardrex)

この記事では、要求ルーティングを管理する方法と、<xref:Microsoft.AspNetCore.Components.Routing.NavLink> コンポーネントを使用して Blazor アプリでナビゲーション リンクを作成する方法について学習します。

## <a name="route-templates"></a>ルート テンプレート

<xref:Microsoft.AspNetCore.Components.Routing.Router> コンポーネントを使用すると、Blazor アプリで Razor コンポーネントにルーティングできます。 <xref:Microsoft.AspNetCore.Components.Routing.Router> コンポーネントは Blazor アプリの `App` コンポーネントで使用されます。

`App.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/App1.razor)]

::: moniker-end

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/App1.razor)]

::: moniker-end

[`@page` ディレクティブ](xref:mvc/views/razor#page) を含む Razor コンポーネント (`.razor`) がコンパイルされると、生成されたコンポーネント クラスに、コンポーネントのルート テンプレートを指定する <xref:Microsoft.AspNetCore.Components.RouteAttribute> が指定されます。

アプリが起動すると、Router の `AppAssembly` として指定されたアセンブリがスキャンされ、<xref:Microsoft.AspNetCore.Components.RouteAttribute> を持つアプリのコンポーネントのルート情報が収集されます。

実行時に、<xref:Microsoft.AspNetCore.Components.RouteView> コンポーネントは、

* <xref:Microsoft.AspNetCore.Components.Routing.Router> から、ルート パラメーターと共に <xref:Microsoft.AspNetCore.Components.RouteData> を受け取ります。
* さらに入れ子になったレイアウトを含め、指定されたコンポーネントをその[レイアウト](xref:blazor/layouts)でレンダリングします。

必要に応じて、[`@layout` ディレクティブ](xref:blazor/layouts#specify-a-layout-in-a-component) を使用してレイアウトを指定しないコンポーネントに対して、レイアウト クラスで <xref:Microsoft.AspNetCore.Components.RouteView.DefaultLayout> パラメーターを指定します。 フレームワークの Blazor プロジェクト テンプレートでは、アプリの既定のレイアウトとして `MainLayout` コンポーネント (`Shared/MainLayout.razor`) を指定します。 レイアウトの詳細については、「<xref:blazor/layouts>」を参照してください。

コンポーネントにより、複数の [`@page` ディレクティブ](xref:mvc/views/razor#page) を使用する複数のルート テンプレートがサポートされます。 次のコンポーネントの例では、`/BlazorRoute` と `/DifferentBlazorRoute` に対する要求を読み込みます。

`Pages/BlazorRoute.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/BlazorRoute.razor?highlight=1-2)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/BlazorRoute.razor?highlight=1-2)]

::: moniker-end

> [!IMPORTANT]
> URL が正しく解決されるように、アプリでは、`href` 属性に指定されているアプリのベース パスを使用して、その `wwwroot/index.html` ファイル (Blazor WebAssembly) または `Pages/_Host.cshtml` ファイル (Blazor Server) に `<base>` タグを含める必要があります。 詳細については、「<xref:blazor/host-and-deploy/index#app-base-path>」を参照してください。

## <a name="provide-custom-content-when-content-isnt-found"></a>コンテンツが見つからないときにカスタム コンテンツを提供する

<xref:Microsoft.AspNetCore.Components.Routing.Router> コンポーネントを使用すると、要求されたルートでコンテンツが見つからない場合に、アプリでカスタム コンテンツを指定できます。

`App` コンポーネントで、<xref:Microsoft.AspNetCore.Components.Routing.Router> コンポーネントの <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> テンプレートにカスタム コンテンツを設定します。

`App.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/App2.razor?highlight=5-8)]

::: moniker-end

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/App2.razor?highlight=5-8)]

::: moniker-end

他の対話型コンポーネントなど、`<NotFound>` タグのコンテンツとして任意の項目がサポートされます。 <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> コンテンツに既定のレイアウトを適用するには、「<xref:blazor/layouts#default-layout>」を参照してください。

## <a name="route-to-components-from-multiple-assemblies"></a>複数のアセンブリからコンポーネントにルーティングする

<xref:Microsoft.AspNetCore.Components.Routing.Router.AdditionalAssemblies> パラメーターを使用して、<xref:Microsoft.AspNetCore.Components.Routing.Router> コンポーネントで、ルーティング可能なコンポーネントを検索するときに考慮する追加のアセンブリを指定します。 `AppAssembly` に指定されたアセンブリに加え、追加のアセンブリがスキャンされます。 次の例では、`Component1` は、参照されている[コンポーネント クラス ライブラリ](xref:blazor/components/class-libraries)に定義されているルーティング可能なコンポーネントです。 次の <xref:Microsoft.AspNetCore.Components.Routing.Router.AdditionalAssemblies> の例は、`Component1` のルーティング サポートの結果を示しています。

`App.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/App3.razor)]

::: moniker-end

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/App3.razor)]

::: moniker-end

## <a name="route-parameters"></a>ルート パラメーター

ルーターでルート パラメーターを使用すれば、同じ名前の対応する[コンポーネント パラメーター](xref:blazor/components/index#component-parameters)を設定できます。 ルート パラメーター名では大文字と小文字は区別されません。 次の例では、`text` パラメーターを使用して、ルート セグメントの値をコンポーネントの `Text` プロパティに割り当てます。 `/RouteParameter/amazing` に対して要求が行われると、`<h1>` タグのコンテンツは `Blazor is amazing!` としてレンダリングされます。

`Pages/RouteParameter.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/RouteParameter1.razor?highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/RouteParameter1.razor?highlight=1)]

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

省略可能なパラメーターがサポートされています。 次の例では、省略可能なパラメーター `text` を使用して、ルート セグメントの値をコンポーネントの `Text` プロパティに割り当てます。 セグメントが存在しない場合、`Text` の値は `fantastic` に設定されます。

`Pages/RouteParameter.razor`:

[!code-razor[](routing/samples_snapshot/5.x/RouteParameter2.razor?highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

省略可能なパラメーターはサポートされていません。 以下の例では、2 つの [`@page` ディレクティブ](xref:mvc/views/razor#page)が適用されています。 1 つ目のディレクティブでは、パラメーターを指定せずにコンポーネントへの移動を許可します。 2 つ目のディレクティブでは、`{text}` ルート パラメーターの値をコンポーネントの `Text` プロパティに割り当てます。

`Pages/RouteParameter.razor`:

[!code-razor[](routing/samples_snapshot/3.x/RouteParameter2.razor?highlight=2)]

::: moniker-end

オプションのパラメーター値が異なる同じコンポーネントへのアプリの移動を許可するには、[`OnInitialized`](xref:blazor/components/lifecycle#component-initialization-methods) ではなく、[`OnParametersSet`](xref:blazor/components/lifecycle#after-parameters-are-set) を使用します。 前の例に基づいて、ユーザーが `/RouteParameter` から `/RouteParameter/amazing` に、または `/RouteParameter/amazing` から `/RouteParameter` に移動できる必要がある場合は、`OnParametersSet` を使用してください。

```csharp
protected override void OnParametersSet()
{
    Text = Text ?? "fantastic";
}
```

## <a name="route-constraints"></a>ルート制約

ルート制約は、コンポーネントへのルート セグメントに型の一致を適用します。

次の例で、`User` コンポーネントへのルートは、次の場合にのみ一致します。

* 要求 URL に `Id` ルート セグメントが存在する。
* `Id` セグメントが整数 (`int`) 型である。

`Pages/User.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/User.razor?highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/User.razor?highlight=1)]

::: moniker-end

次の表に示すルート制約を使用できます。 インバリアント カルチャと一致するルート制約については、表の下の警告で詳細をご確認ください。

| 制約 | 例           | 一致の例                                                                  | インバリアント<br>カルチャ<br>一致 |
| ---------- | ----------------- | -------------------------------------------------------------------------------- | :------------------------------: |
| `bool`     | `{active:bool}`   | `true`, `FALSE`                                                                  | いいえ                               |
| `datetime` | `{dob:datetime}`  | `2016-12-31`, `2016-12-31 7:32pm`                                                | はい                              |
| `decimal`  | `{price:decimal}` | `49.99`, `-1,000.01`                                                             | はい                              |
| `double`   | `{weight:double}` | `1.234`, `-1,001.01e8`                                                           | はい                              |
| `float`    | `{weight:float}`  | `1.234`, `-1,001.01e8`                                                           | はい                              |
| `guid`     | `{id:guid}`       | `CD2C1638-1638-72D5-1638-DEADBEEF1638`, `{CD2C1638-1638-72D5-1638-DEADBEEF1638}` | いいえ                               |
| `int`      | `{id:int}`        | `123456789`, `-123456789`                                                        | はい                              |
| `long`     | `{ticks:long}`    | `123456789`, `-123456789`                                                        | はい                              |

> [!WARNING]
> URL の妥当性を検証し、CLR 型 (`int` や <xref:System.DateTime> など) に変換されるルート制約では、常にインバリアント カルチャが使用されます。 これらの制約では、URL がローカライズ不可であることが前提となります。

## <a name="routing-with-urls-that-contain-dots"></a>ドットを含む URL によるルーティング

ホストされている Blazor WebAssembly および Blazor Server アプリの場合は、サーバー側の既定のルート テンプレートで、ファイルが要求されているドット (`.`) が要求 URL の最後のセグメントに含まれていると想定されます。 たとえば、URL `https://localhost.com:5001/example/some.thing` は、ルーターによって `some.thing` という名前のファイルに対する要求として解釈されます。 追加の構成がないと、`some.thing` が [`@page` ディレクティブ](xref:mvc/views/razor#page) を含むコンポーネントにルーティングすることを意図しており、`some.thing` がルート パラメーター値である場合に、アプリによって "*404 - 見つかりません*" という応答が返されます。 ドットが含まれる 1 つまたは複数のパラメーターを指定してルートを使用するには、アプリでカスタム テンプレートを使ってルートを構成する必要があります。

URL の最後のセグメントからルート パラメーターを受け取ることができる次の `Example` コンポーネントについて考えてみます。

`Pages/Example.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/Example.razor?highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/Example.razor?highlight=2)]

::: moniker-end

ホストされている Blazor WebAssembly ソリューションの *`Server`* アプリで、`param` ルート パラメーターのドットを使って要求をルーティングできるようにするには、`Startup.Configure` で省略可能なパラメーターを指定してフォールバック ファイル ルート テンプレートを追加します。

`Startup.cs`:

```csharp
endpoints.MapFallbackToFile("/example/{param?}", "index.html");
```

`param` ルート パラメーターのドットを使って要求をルーティングするように Blazor Server アプリを構成するには、`Startup.Configure` で省略可能なパラメーターを指定してフォールバック ページ ルート テンプレートを追加します。

`Startup.cs`:

```csharp
endpoints.MapFallbackToPage("/example/{param?}", "/_Host");
```

詳細については、「<xref:fundamentals/routing>」を参照してください。

## <a name="catch-all-route-parameters"></a>キャッチオールのルート パラメーター

::: moniker range=">= aspnetcore-5.0"

複数のフォルダー境界にまたがるパスをキャプチャするキャッチオール ルート パラメーターが、コンポーネントでサポートされます。

キャッチオールのルート パラメーターは次のとおりです。

* ルート セグメント名と一致する名前が付けられている。 名前付けで大文字と小文字は区別されない。
* `string` 型。 フレームワークによって自動キャストは提供されません。
* URL の末尾。

`Pages/CatchAll.razor`:

[!code-razor[](routing/samples_snapshot/5.x/CatchAll.razor)]

ルート テンプレートが `/catch-all/{*pageRoute}` の URL `/catch-all/this/is/a/test` の場合、`PageRoute` の値は `this/is/a/test` に設定されます。

キャプチャされたパスのスラッシュとセグメントはデコードされます。 `/catch-all/{*pageRoute}` のルート テンプレートでは、URL `/catch-all/this/is/a%2Ftest%2A` から `this/is/a/test*` が生成されます。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

キャッチオール ルート パラメーターは、ASP.NET Core 5.0 以降でサポートされます。 詳細については、この記事の 5.0 バージョンを選択してください。

::: moniker-end

## <a name="uri-and-navigation-state-helpers"></a>URI およびナビゲーション状態ヘルパー

C# コード内の URI とナビゲーションを管理するには、<xref:Microsoft.AspNetCore.Components.NavigationManager> を使用します。 <xref:Microsoft.AspNetCore.Components.NavigationManager> には、次の表に示すイベントとメソッドがあります。

| メンバー | 説明 |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.Uri> | 現在の絶対 URI を取得します。 |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.BaseUri> | 絶対 URI を生成するために、相対 URI パスの前に付加できるベース URI (末尾のスラッシュを含む) を取得します。 通常、<xref:Microsoft.AspNetCore.Components.NavigationManager.BaseUri> は `wwwroot/index.html` (Blazor WebAssembly) または `Pages/_Host.cshtml` (Blazor Server) 内のドキュメントの `<base>` 要素の `href` 属性に対応します。 |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A> | 指定された URI に移動します。 `forceLoad` が `true` の場合:<ul><li>クライアント側のルーティングはバイパスされます。</li><li>URI が通常クライアント側ルーターによって処理されるかどうかにかかわらず、ブラウザーでは、強制的にサーバーから新しいページが読み込まれます。</li></ul> |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.LocationChanged> | ナビゲーションの場所が変更されたときに発生するイベントです。 |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.ToAbsoluteUri%2A> | 相対 URI を絶対 URI に変換します。 |
| <span style="word-break:normal;word-wrap:normal"><xref:Microsoft.AspNetCore.Components.NavigationManager.ToBaseRelativePath%2A></span> | ベース URI (たとえば、<xref:Microsoft.AspNetCore.Components.NavigationManager.BaseUri> によって以前に返された URI) が与えられると、絶対 URI を、ベース URI プレフィックスに相対的な URI に変換します。 |

<xref:Microsoft.AspNetCore.Components.NavigationManager.LocationChanged> イベントの場合、<xref:Microsoft.AspNetCore.Components.Routing.LocationChangedEventArgs> でナビゲーション イベントに関する次の情報が提供されます。

* <xref:Microsoft.AspNetCore.Components.Routing.LocationChangedEventArgs.Location>:新しい場所の URL。
* <xref:Microsoft.AspNetCore.Components.Routing.LocationChangedEventArgs.IsNavigationIntercepted>:`true` の場合、Blazor によってブラウザーからナビゲーションがインターセプトされました。 `false` の場合、<xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A?displayProperty=nameWithType> によってナビゲーションが発生しました。

次のコンポーネント:

* <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A> を使用してボタンが選択されたときに、アプリの `Counter` コンポーネント (`Pages/Counter.razor`) に移動します。
* <xref:Microsoft.AspNetCore.Components.NavigationManager.LocationChanged?displayProperty=nameWithType> をサブスクライブすることで、場所の変更イベントを処理します。
  * `HandleLocationChanged` メソッドは、`Dispose` がフレームワークによって呼び出されると、アンフックになります。 このメソッドをアンフックすることで、コンポーネントのガベージ コレクションが許可されます。
  * ロガーの実装では、ボタンが選択されたときに次の情報をログに記録します。

    > `BlazorSample.Pages.Navigate: Information: URL of new location: https://localhost:5001/counter`

`Pages/Navigate.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/Navigate.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/Navigate.razor)]

::: moniker-end

コンポーネントの破棄の詳細については、「<xref:blazor/components/lifecycle#component-disposal-with-idisposable>」を参照してください。

## <a name="query-string-and-parse-parameters"></a>クエリ文字列とパラメーターの解析

要求のクエリ文字列は、<xref:Microsoft.AspNetCore.Components.NavigationManager.Uri?displayProperty=nameWithType> プロパティから取得されます。

```razor
@inject NavigationManager NavigationManager

...

var query = new Uri(NavigationManager.Uri).Query;
```

クエリ文字列のパラメーターを解析するには:

* アプリでは <xref:Microsoft.AspNetCore.WebUtilities> API を使用できます。 アプリで API を使用できない場合は、[Microsoft.AspNetCore.WebUtilities](https://www.nuget.org/packages/Microsoft.AspNetCore.WebUtilities) のアプリのプロジェクト ファイルにパッケージ参照を追加します。
* <xref:Microsoft.AspNetCore.WebUtilities.QueryHelpers.ParseQuery%2A?displayProperty=nameWithType> を使用してクエリ文字列を解析した後に値を取得します。

次の `ParseQueryString` コンポーネントの例では、`ship` という名前のクエリ文字列パラメーターのキーを解析します。 たとえば、URL クエリ文字列のキーと値のペアの `?ship=Tardis` の場合、`queryValue` の `Tardis` 値が取り込まれます。 次の例では、`https://localhost:5001/parse-query-string?ship=Tardis` という URL を使用してアプリに移動します。

`Pages/ParseQueryString.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/ParseQueryString.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/ParseQueryString.razor)]

::: moniker-end

## <a name="navlink-and-navmenu-components"></a>`NavLink` および `NavMenu` コンポーネント

ナビゲーション リンクを作成するときは、HTML ハイパーリンク要素 (`<a>`) の代わりに <xref:Microsoft.AspNetCore.Components.Routing.NavLink> コンポーネントを使用します。 <xref:Microsoft.AspNetCore.Components.Routing.NavLink> コンポーネントは `<a>` 要素のように動作しますが、`href` が現在の URL と一致するかどうかに基づいて `active` CSS クラスを切り替える点が異なります。 `active` クラスは、表示されているナビゲーション リンクの中でどのページがアクティブ ページであるかをユーザーが理解するのに役立ちます。 必要に応じて、CSS クラス名を <xref:Microsoft.AspNetCore.Components.Routing.NavLink.ActiveClass?displayProperty=nameWithType> に割り当てて、現在のルートが `href` と一致したときに、レンダリングされるリンクにカスタム CSS クラスを適用します。

次の `NavMenu` コンポーネントでは、<xref:Microsoft.AspNetCore.Components.Routing.NavLink> コンポーネントの使用方法を示す [`Bootstrap`](https://getbootstrap.com/docs/) ナビゲーション バーを作成しています。

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/NavMenu.razor?highlight=4,9)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/NavMenu.razor?highlight=4,9)]

::: moniker-end

> [!NOTE]
> `NavMenu` コンポーネント (`NavMenu.razor`) は、Blazor プロジェクト テンプレートから生成されたアプリの `Shared` フォルダーにあります。

`<NavLink>` 要素の `Match` 属性に割り当てられる 2 つの <xref:Microsoft.AspNetCore.Components.Routing.NavLinkMatch> オプションがあります。

* <xref:Microsoft.AspNetCore.Components.Routing.NavLinkMatch.All?displayProperty=nameWithType>:<xref:Microsoft.AspNetCore.Components.Routing.NavLink> は、現在の URL 全体に一致する場合にアクティブになります。
* <xref:Microsoft.AspNetCore.Components.Routing.NavLinkMatch.Prefix?displayProperty=nameWithType> (*既定値*):<xref:Microsoft.AspNetCore.Components.Routing.NavLink> は、現在の URL の任意のプレフィックスに一致する場合にアクティブになります。

前の例では、ホーム <xref:Microsoft.AspNetCore.Components.Routing.NavLink> `href=""` はホーム URL と一致し、アプリの既定のベース パス URL (`https://localhost:5001/` など) でのみ `active` CSS クラスを受け取ります。 2 番目の <xref:Microsoft.AspNetCore.Components.Routing.NavLink> は、ユーザーが `component` プレフィックスを含む任意の URL (`https://localhost:5001/component` や `https://localhost:5001/component/another-segment` など) にアクセスしたときに、`active` クラスを受け取ります。

追加の <xref:Microsoft.AspNetCore.Components.Routing.NavLink> コンポーネント属性は、レンダリングされるアンカー タグに渡されます。 次の例では、<xref:Microsoft.AspNetCore.Components.Routing.NavLink> コンポーネントに `target` 属性が含まれています。

```razor
<NavLink href="example-page" target="_blank">Example page</NavLink>
```

次の HTML マークアップがレンダリングされます。

```html
<a href="example-page" target="_blank">Example page</a>
```

> [!WARNING]
> Blazor による子コンテンツのレンダリング方法により、`for` ループ内の `NavLink` コンポーネントのレンダリングでは、インクリメントするループ変数が `NavLink` (子) コンポーネントのコンテンツ内で使用されている場合、ローカル インデックス変数が必要になります。
>
> ```razor
> @for (int c = 0; c < 10; c++)
> {
>     var current = c;
>     <li ...>
>         <NavLink ... href="@c">
>             <span ...></span> @current
>         </NavLink>
>     </li>
> }
> ```
>
> このシナリオでのインデックス変数の使用は、`NavLink` コンポーネントだけでなく、[子コンテンツ](xref:blazor/components/index#child-content)でループ変数を使用する **すべての** 子コンポーネントで必須です。
>
> または、<xref:System.Linq.Enumerable.Range%2A?displayProperty=nameWithType> と共に `foreach` ループを使用します。
>
> ```razor
> @foreach(var c in Enumerable.Range(0,10))
> {
>     <li ...>
>         <NavLink ... href="@c">
>             <span ...></span> @c
>         </NavLink>
>     </li>
> }
> ```

## <a name="aspnet-core-endpoint-routing-integration"></a>ASP.NET Core エンドポイントのルーティングの統合

"*このセクションは Blazor Server アプリにのみ適用されます。* "

Blazor Server は [ASP.NET Core エンドポイントのルーティング](xref:fundamentals/routing)に統合されています。 ASP.NET Core アプリは、`Startup.Configure` で <xref:Microsoft.AspNetCore.Builder.ComponentEndpointRouteBuilderExtensions.MapBlazorHub%2A> を使用して、対話型コンポーネントの着信接続を受け入れるように構成されています。

`Startup.cs`:

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](routing/samples_snapshot/5.x/Startup.cs?highlight=5)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-csharp[](routing/samples_snapshot/3.x/Startup.cs?highlight=5)]

::: moniker-end

一般的な構成は、すべての要求を Razor ページにルーティングすることです。これは、Blazor Server アプリのサーバー側部分のホストとして機能します。 慣例により、"*ホスト*" のページは通常、アプリの `Pages` フォルダーでは `_Host.cshtml` という名前になります。

ホスト ファイルに指定されるルートは、ルート照合で低い優先順位で動作するため、*フォールバック ルート* と呼ばれます。 フォールバック ルートは、他のルートが一致しない場合に使用されます。 これにより、Blazor Server アプリのコンポーネント ルーティングに干渉することなく、他のコントローラーやページをアプリで使用できるようになります。

ルート以外の URL のサーバー ホスト用に <xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapFallbackToPage%2A> を構成する方法の詳細については、<xref:blazor/host-and-deploy/index#app-base-path> をご覧ください。
