---
title: ASP.NET Core でのアセンブリの遅延読み込みBlazor WebAssembly
author: guardrex
description: ASP.NET Core Blazor WebAssembly アプリでアセンブリの遅延読み込みを行う方法について説明します。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 09/09/2020
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
uid: blazor/webassembly-lazy-load-assemblies
ms.openlocfilehash: e8589a1e288c39b487673fafc04c59fa07916335
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280673"
---
# <a name="lazy-load-assemblies-in-aspnet-core-blazor-webassembly"></a>ASP.NET Core でのアセンブリの遅延読み込みBlazor WebAssembly

Blazor WebAssembly アプリの起動時のパフォーマンスは、一部のアプリケーション アセンブリの読み込みをそれが必要になるまで延期することで改善できます。これは "*遅延読み込み*" と呼ばれています。 たとえば、1 つのコンポーネントをレンダリングするためにのみ使用されるアセンブリの読み込みを、ユーザーがそのコンポーネントに移動する場合にのみ行うように設定することができます。 読み込みが完了すると、アセンブリはクライアント側にキャッシュされ、今後のすべてのナビゲーションで使用できるようになります。

Blazor の遅延読み込み機能を使用すると、アプリ アセンブリに遅延読み込みのマークを付けることができます。これにより、実行時にユーザーが特定のルートに移動したとき、アセンブリが読み込まれます。 この機能は、プロジェクト ファイルに対する変更と、アプリケーションのルーターに対する変更で構成されています。

> [!NOTE]
> Blazor Server アプリでは、アセンブリの遅延読み込みを行っても効果がありません。これは、Blazor Server アプリのクライアントにアセンブリがダウンロードされないためです。

## <a name="project-file"></a>プロジェクト ファイル

アプリのプロジェクト ファイル (`.csproj`) 内で、`BlazorWebAssemblyLazyLoad` 項目を使用して、遅延読み込みのマークをアセンブリに付けます。 `.dll` 拡張子が含まれるアセンブリ名を使用します。 Blazor フレームワークを使用すると、この項目グループによって指定されたアセンブリはアプリの起動時に読み込まれません。 次の例では、大規模なカスタム アセンブリ (`GrantImaharaRobotControls.dll`) に遅延読み込みのマークを付けいています。 遅延読み込みのマークが付けられているアセンブリに依存関係がある場合、それらにはプロジェクト ファイル内でも遅延読み込みのマークを付ける必要があります。

```xml
<ItemGroup>
  <BlazorWebAssemblyLazyLoad Include="GrantImaharaRobotControls.dll" />
</ItemGroup>
```

## <a name="router-component"></a>`Router` コンポーネント

Blazor がルーティング可能なコンポーネントを求めて探索するアセンブリは、Blazor の `Router` コンポーネントによって指定されます。 `Router` コンポーネントは、ユーザーが移動するルートのコンポーネントをレンダリングする役割も担います。 `Router` コンポーネントでは、遅延読み込みと組み合わせて使用できる `OnNavigateAsync` 機能がサポートされています。

アプリの `Router` コンポーネント (`App.razor`):

* `OnNavigateAsync` コールバックを追加します。 `OnNavigateAsync` ハンドラーは、ユーザーが次のことを行った場合に呼び出されます。
  * 使用しているブラウザーからの直接移動によって、ルートに初めてアクセスする。
  * リンクまたは <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A?displayProperty=nameWithType> 呼び出しを使用して新しいルートに移動する。
* 遅延読み込みされたアセンブリにルーティング可能なコンポーネントが含まれている場合は、コンポーネントに [List](xref:System.Collections.Generic.List%601)\<<xref:System.Reflection.Assembly>> (たとえば、名前付きの `lazyLoadedAssemblies`) を追加します。 アセンブリにルーティング可能なコンポーネントが含まれている場合、そのアセンブリは <xref:Microsoft.AspNetCore.Components.Routing.Router.AdditionalAssemblies> コレクションに戻されます。 フレームワークでは、アセンブリ内でルートが検索され、新しいルートが見つかった場合はルート コレクションが更新されます。

```razor
@using System.Reflection

<Router AppAssembly="@typeof(Program).Assembly" 
    AdditionalAssemblies="@lazyLoadedAssemblies" OnNavigateAsync="@OnNavigateAsync">
    ...
</Router>

@code {
    private List<Assembly> lazyLoadedAssemblies = new List<Assembly>();

    private async Task OnNavigateAsync(NavigationContext args)
    {
    }
}
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

`OnNavigateAsync` コールバックから、ハンドルされない例外がスローされた場合は、[Blazorエラー UI](xref:blazor/fundamentals/handle-errors#detailed-errors-during-development) が呼び出されます。

### <a name="assembly-load-logic-in-onnavigateasync"></a>`OnNavigateAsync` 内のアセンブリ読み込みロジック

`OnNavigateAsync` には `NavigationContext` パラメーターがあります。これによって、ターゲット パス (`Path`) やキャンセル トークン (`CancellationToken`) など、現在の非同期ナビゲーション イベントに関する情報が提供されます。

* `Path` プロパティは、`/robot` など、アプリのベース パスを基準としたユーザーの宛先パスです。
* `CancellationToken` を使用すると、非同期タスクの取り消しを確認することができます。 ユーザーが別のページに移動したときは、現在実行中のナビゲーション タスクが `OnNavigateAsync` によって自動的にキャンセルされます。

`OnNavigateAsync` 内で、読み込むアセンブリを決定するロジックを実装します。 次のオプションがあります。

* `OnNavigateAsync` メソッド内での条件チェック。
* ルートをアセンブリ名にマップするルックアップ テーブル (コンポーネントに挿入されるか、[`@code`](xref:mvc/views/razor#code) ブロック内に実装される)。

`LazyAssemblyLoader` はフレームワークによって提供されるシングルトン サービスであり、アセンブリを読み込むためのものです。 `Router` コンポーネントに `LazyAssemblyLoader` を挿入します。

```razor
...
@using Microsoft.AspNetCore.Components.WebAssembly.Services
@inject LazyAssemblyLoader assemblyLoader

...
```

`LazyAssemblyLoader` には、次のような `LoadAssembliesAsync` メソッドが用意されています。

* JS 相互運用を使用して、ネットワーク呼び出しを介してアセンブリをフェッチします。
* ブラウザー内の WebAssembly で実行されているランタイムにアセンブリを読み込みます。

フレームワークの遅延読み込みの実装では、ホストされた Blazor ソリューションでのプリレンダリングによる遅延読み込みがサポートされます。 プリレンダリング中は、遅延読み込みのマークが付けられたものも含め、すべてのアセンブリが読み込まれると見なされます。 "*サーバー*" プロジェクトの `Startup.ConfigureServices` メソッド (`Startup.cs`) に、手動で `LazyAssemblyLoader` を登録します。

```csharp
services.AddScoped<LazyAssemblyLoader>();
```

### <a name="user-interaction-with-navigating-content"></a>`<Navigating>` コンテンツとのユーザー操作

アセンブリの読み込み中 (数秒かかることがあります)、`Router` コンポーネントからユーザーに、ページの切り替えが行われていることを示すことができます。

* <xref:Microsoft.AspNetCore.Components.Routing?displayProperty=fullName> 名前空間の [`@using`](xref:mvc/views/razor#using) ディレクティブを追加します。
* ページ切り替えイベント中に表示するマークアップを含むコンポーネントに `<Navigating>` タグを追加します。

```razor
...
@using Microsoft.AspNetCore.Components.Routing
...

<Router ...>
    <Navigating>
        <div style="...">
            <p>Loading the requested page&hellip;</p>
        </div>
    </Navigating>
</Router>

...
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

### <a name="handle-cancellations-in-onnavigateasync"></a>`OnNavigateAsync` でキャンセルを処理する

`OnNavigateAsync` コールバックに渡される `NavigationContext` オブジェクトには、新しいナビゲーション イベントが発生したときに設定される `CancellationToken` が含まれています。 このキャンセル トークンが、古いナビゲーションに対して `OnNavigateAsync` コールバックを継続して実行しないように設定されている場合は、`OnNavigateAsync` コールバックをスローする必要があります。

ユーザーがルート A に移動し、その後すぐにルート B に移動する場合、アプリでルート A の `OnNavigateAsync` コールバックを実行し続けるべきではありません。

```razor
@inject HttpClient Http
@inject ProductCatalog Products

<Router AppAssembly="@typeof(Program).Assembly" 
    OnNavigateAsync="@OnNavigateAsync">
    ...
</Router>

@code {
    private async Task OnNavigateAsync(NavigationContext context)
    {
        if (context.Path == "/about") 
        {
            var stats = new Stats = { Page = "/about" };
            await Http.PostAsJsonAsync("api/visited", stats, context.CancellationToken);
        }
        else if (context.Path == "/store")
        {
            var productIds = [345, 789, 135, 689];

            foreach (var productId in productIds) 
            {
                context.CancellationToken.ThrowIfCancellationRequested();
                Products.Prefetch(productId);
            }
        }
    }
}
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

> [!NOTE]
> `NavigationContext` 内のキャンセル トークンが取り消された場合にスローしないと、前のナビゲーションからのコンポーネントをレンダリングするなど、意図しない動作が発生する可能性があります。

### <a name="onnavigateasync-events-and-renamed-assembly-files"></a>`OnNavigateAsync` イベントと名前が変更されたアセンブリ ファイル

リソース ローダーでは、`blazor.boot.json` ファイルで定義されているアセンブリ名が使用されます。 [アセンブリの名前が変更された](xref:blazor/host-and-deploy/webassembly#change-the-filename-extension-of-dll-files)場合、`OnNavigateAsync` メソッドで使用されるアセンブリ名と `blazor.boot.json` ファイル内のアセンブリ名が同期されなくなります。

これを修正するには:

* 使用するアセンブリ名を決定するときに、アプリが運用環境で実行されているかどうかを確認します。
* 名前を変更したアセンブリ名を別のファイルに格納し、そのファイルから読み取りを行い、`LazyLoadAssemblyService` および `OnNavigateAsync` メソッドで使用するアセンブリ名を決定します。

### <a name="complete-example"></a>コード例全体

次の完全な `Router` コンポーネントによって、ユーザーが `/robot` に移動したときに `GrantImaharaRobotControls.dll` アセンブリを読み込む方法が実演されています。 ページの切り替え中に、スタイル付きのメッセージがユーザーに表示されます。

```razor
@using System.Reflection
@using Microsoft.AspNetCore.Components.Routing
@using Microsoft.AspNetCore.Components.WebAssembly.Services
@inject LazyAssemblyLoader assemblyLoader

<Router AppAssembly="@typeof(Program).Assembly" 
    AdditionalAssemblies="@lazyLoadedAssemblies" OnNavigateAsync="@OnNavigateAsync">
    <Navigating>
        <div style="padding:20px;background-color:blue;color:white">
            <p>Loading the requested page&hellip;</p>
        </div>
    </Navigating>
    <Found Context="routeData">
        <RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
    </Found>
    <NotFound>
        <LayoutView Layout="@typeof(MainLayout)">
            <p>Sorry, there's nothing at this address.</p>
        </LayoutView>
    </NotFound>
</Router>

@code {
    private List<Assembly> lazyLoadedAssemblies = new List<Assembly>();

    private async Task OnNavigateAsync(NavigationContext args)
    {
        try
        {
            if (args.Path.EndsWith("/robot"))
            {
                var assemblies = await assemblyLoader.LoadAssembliesAsync(
                    new List<string>() { "GrantImaharaRobotControls.dll" });
                lazyLoadedAssemblies.AddRange(assemblies);
            }
        }
        catch (Exception ex)
        {
            ...
        }
    }
}
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

## <a name="troubleshoot"></a>トラブルシューティング

* 予期しないレンダリングが発生した場合 (たとえば、前のナビゲーションからのコンポーネントがレンダリングされる) は、キャンセル トークンが設定されている場合にコードがスローされることを確認してください。
* アプリケーションの起動時にアセンブリがまだ読み込まれている場合は、アセンブリがプロジェクト ファイル内で遅延読み込みとしてマークされていることを確認します。

## <a name="additional-resources"></a>その他のリソース

* <xref:blazor/webassembly-performance-best-practices>
