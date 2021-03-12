---
title: ASP.NET Core のアンカー タグ ヘルパー
author: pkellner
description: ASP.NET Core アンカー タグ ヘルパーの属性と、HTML アンカー タグの動作拡張時の各属性の役割を示します。
ms.author: scaddie
ms.custom: mvc
ms.date: 10/13/2019
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
uid: mvc/views/tag-helpers/builtin-th/anchor-tag-helper
ms.openlocfilehash: 2e49c545b0d343475ce44a636a6ae66324f9d3bf
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587204"
---
# <a name="anchor-tag-helper-in-aspnet-core"></a>ASP.NET Core のアンカー タグ ヘルパー

作成者: [Peter Kellner](https://peterkellner.net)、[Scott Addie](https://github.com/scottaddie)

[アンカー タグ ヘルパー](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper)は、新しい属性を追加することで標準の HTML アンカー (`<a ... ></a>`) タグを拡張します。 規則では、属性名の前に `asp-` が付きます。 レンダリングされたアンカー要素の `href` 属性値は、`asp-` 属性の値によって決まります。

タグ ヘルパーの概要については、「<xref:mvc/views/tag-helpers/intro>」をご覧ください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/tag-helpers/built-in/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

*SpeakerController* は、このドキュメント全体にわたってサンプルとして使用されます。

[!code-csharp[](samples/TagHelpersBuiltIn/Controllers/SpeakerController.cs?name=snippet_SpeakerController)]

## <a name="anchor-tag-helper-attributes"></a>アンカー タグ ヘルパーの属性

### <a name="asp-controller"></a>asp-controller

[asp-controller](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Controller*) 属性は、URL の生成に使用するコント ローラーを割り当てます。 次のマークアップでは、すべてのスピーカーが一覧表示されます。

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspController)]

生成される HTML:

```html
<a href="/Speaker">All Speakers</a>
```

`asp-controller` 属性が指定されていて、`asp-action` が指定されていない場合は、既定値 `asp-action` が現在実行中のビューに関連付けられたコント ローラー アクションです。 `asp-action` が前のマークアップから省略されていて、アンカー タグ ヘルパーが *HomeController* の *Index* ビュー (*/Home*) で使用されている場合、次の HTML が生成されます。

```html
<a href="/Home">All Speakers</a>
```

### <a name="asp-action"></a>asp-action

[asp-action](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Action*) 属性値は、生成された `href` 属性に含まれるコントローラー アクション名を表します。 次のマークアップは、生成された `href` 属性値をスピーカー評価ページに設定します。

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspAction)]

生成される HTML:

```html
<a href="/Speaker/Evaluations">Speaker Evaluations</a>
```

`asp-controller` 属性が指定されていない場合、現在のビューを実行しているビューを呼び出す既定のコントローラーが使用されます。

属性値 `asp-action` が `Index` で、URL にアクションが何も追加されていない場合、既定の `Index` の操作が呼び出されます。 指定された (または既定の) 操作が、`asp-controller` で参照されるコントローラーに存在する必要があります。

### <a name="asp-route-value"></a>asp-route-{value}

[asp-route-{value}](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.RouteValues*) 属性は、ワイルドカード ルート プレフィックスを有効にします。 `{value}` プレースホルダーに入っている値はすべて、潜在的なルートのパラメーターとして解釈されます。 既定のルートが見つからない場合は、このルート プレフィックスが生成された `href` に要求パラメーターおよび値として追加されます。 見つかった場合は、そのルートがルート テンプレートで置き換えられます。

次のコントローラー アクションがあるとします。

[!code-csharp[](samples/TagHelpersBuiltIn/Controllers/BuiltInTagController.cs?name=snippet_AnchorTagHelperAction)]

既定のルート テンプレートは *Startup.Configure* に定義されています。

[!code-csharp[](samples/TagHelpersBuiltIn/Startup.cs?name=snippet_UseMvc&highlight=8-10)]

MVC ビューは、次のように、アクションによって提供されるモデルを使用します。

```cshtml
@model Speaker
<!DOCTYPE html>
<html>
<body>
    <a asp-controller="Speaker"
       asp-action="Detail" 
       asp-route-id="@Model.SpeakerId">SpeakerId: @Model.SpeakerId</a>
</body>
</html>
```

既定のルートの `{id?}` プレースホルダーが一致しました。 生成される HTML:

```html
<a href="/Speaker/Detail/12">SpeakerId: 12</a>
```

次の MVC ビューのように、ルート プレフィックスが一致するルーティング テンプレートに含まれないものとします。

```cshtml
@model Speaker
<!DOCTYPE html>
<html>
<body>
    <a asp-controller="Speaker"
       asp-action="Detail"
       asp-route-speakerid="@Model.SpeakerId">SpeakerId: @Model.SpeakerId</a>
<body>
</html>
```

`speakerid` が一致するルートに見つからなかったため、次の HTML が生成されます。

```html
<a href="/Speaker/Detail?speakerid=12">SpeakerId: 12</a>
```

`asp-controller` または `asp-action` のどちらかが指定されていない場合は、`asp-route` 属性の場合と同じ既定の処理に従います。

### <a name="asp-route"></a>asp-route

[asp-route](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Route*) 属性は、名前付きのルートに直接 URL リンクを作成するために使用します。 [ルーティング属性](xref:mvc/controllers/routing#attribute-routing)を使用すると、`SpeakerController` に示されているようにルートに名前を付け、その `Evaluations` アクションで使用することができます。

[!code-csharp[](samples/TagHelpersBuiltIn/Controllers/SpeakerController.cs?range=22-24)]

次のマークアップでは、`asp-route` 属性が名前付きのルートを参照します。

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspRoute)]

アンカー タグ ヘルパーは URL */Speaker/Evaluations* を使用してそのコントローラー アクションに直接ルートを生成します。 生成される HTML:

```html
<a href="/Speaker/Evaluations">Speaker Evaluations</a>
```

`asp-controller` または `asp-action` が `asp-route` の他に指定されていると、想定と異なるルートが生成される場合があります。 ルートの競合を防ぐには、`asp-route` を属性 `asp-controller` および `asp-action` とともに使用してはいけません。

### <a name="asp-all-route-data"></a>asp-all-route-data

[asp-all-route-data](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.RouteValues*) 属性は、キー/値ペアのディクショナリの作成をサポートします。 キーはパラメーターの名前で、値はパラメーターの値です。

次の例では、ディクショナリが初期化され、ビューに渡され Razor ます。 データがモデルによって渡される場合もあります。

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspAllRouteData)]

上のコードを実行すると、次の HTML が生成されます。

```html
<a href="/Speaker/EvaluationsCurrent?speakerId=11&currentYear=true">Speaker Evaluations</a>
```

`asp-all-route-data` ディクショナリは、オーバー ロードされた `Evaluations` アクションの要件を満たすクエリ文字列を生成するためにフラット化されます。

[!code-csharp[](samples/TagHelpersBuiltIn/Controllers/SpeakerController.cs?range=26-30)]

ディクショナリ内のいずれかのキーがルート パラメーターと一致する場合は、その値がルートで適宜置き換えられます。 その他の一致しない値は要求パラメーターとして生成されます。

### <a name="asp-fragment"></a>asp-fragment

[asp-fragment](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Fragment*) 属性では URL に追加される URL フラグメントが定義されます。 アンカー タグ ヘルパーにより、ハッシュ文字 (#) が追加されます。 次のマークアップがあるとします。

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspFragment)]

生成される HTML:

```html
<a href="/Speaker/Evaluations#SpeakerEvaluations">Speaker Evaluations</a>
```

ハッシュ タグはクライアント側アプリを構築するときに便利です。 たとえば、JavaScript でのマーク付けや検索を簡単にするために使用できます。

### <a name="asp-area"></a>asp-area

[asp-area](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Area*) 属性は、適切なルートの設定に使用する領域名を設定します。 `asp-area` 属性によってどのようにルートの再マップが行われるかの例を以下に示します。

#### <a name="usage-in-razor-pages"></a>使用 ( Razor ページで)

Razor ページ領域は ASP.NET Core 2.1 以降でサポートされています。

次のディレクトリ階層があるとします。

* **{プロジェクト名}**
  * **wwwroot**
  * **領域**
    * **セッション**
      * **ページ**
        * *\_ViewStart. cshtml*
        * *Index.cshtml*
        * *Index.cshtml.cs*
  * **ページ**

[ *セッション* 領域の *インデックス*] ページを参照するマークアップ Razor は次のとおりです。

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspAreaRazorPages)]

生成される HTML:

```html
<a href="/Sessions">View Sessions</a>
```

> [!TIP]
> Pages アプリの領域をサポートするには Razor 、で次のいずれかの操作を行い `Startup.ConfigureServices` ます。
>
> * [互換性バージョン](xref:mvc/compatibility-version)に 2.1 以降を設定します。
> * 次のように、[ [ Razor allowareas]](xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions.AllowAreas*)プロパティをに設定します。 `true`
>
>   [!code-csharp[](samples/TagHelpersBuiltIn/Startup.cs?name=snippet_AllowAreas)]

#### <a name="usage-in-mvc"></a>MVC の使用法

次のディレクトリ階層があるとします。

* **{プロジェクト名}**
  * **wwwroot**
  * **領域**
    * **ブログ**
      * **コントローラー**
        * *HomeController.cs*
      * **ビュー**
        * **Home**
          * *AboutBlog.cshtml*
          * *Index.cshtml*
        * *\_ViewStart. cshtml*
  * **コントローラー**

`asp-area` を [ブログ] に設定すると、このアンカー タグの関連付けられているコントローラーとビューのルートに、ディレクトリ *Areas/Blogs* のプレフィックスが付けられます。 *AboutBlog* ビューを参照するマークアップは次のとおりです。

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspArea)]

生成される HTML:

```html
<a href="/Blogs/Home/AboutBlog">About Blog</a>
```

> [!TIP]
> MVC アプリで領域をサポートするには、ルート テンプレートに領域への参照 (存在する場合) が含まれている必要があります。 そのテンプレートは、*Startup.Configure* の `routes.MapRoute` メソッド呼び出しの 2 番目のパラメーターで表されます
>
> [!code-csharp[](samples/TagHelpersBuiltIn/Startup.cs?name=snippet_UseMvc&highlight=5)]

### <a name="asp-protocol"></a>asp-protocol

[asp-protocol](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Protocol*) 属性は URL に (`https` などの) プロトコルを指定するためにあります。 次に例を示します。

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspProtocol)]

生成される HTML:

```html
<a href="https://localhost/Home/About">About</a>
```

例ではホスト名が localhost です。 アンカー タグ ヘルパーは、URL を生成するときに Web サイトのパブリック ドメインを使用します。

### <a name="asp-host"></a>asp-host

[asp-host](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Host*) 属性は URL のホスト名を指定するためにあります。 次に例を示します。

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspHost)]

生成される HTML:

```html
<a href="https://microsoft.com/Home/About">About</a>
```

### <a name="asp-page"></a>asp-page

[Asp ページ](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Page*)属性は、ページで使用され Razor ます。 アンカー タグの `href` 属性の値を特定のページに設定するために使用します。 ページ名の前にスラッシュ "/" を付けると URL が作成されます。

次の例では、出席者ページを参照してい Razor ます。

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspPage)]

生成される HTML:

```html
<a href="/Attendee">All Attendees</a>
```

`asp-page` 属性は `asp-route`、`asp-controller`、`asp-action` の各属性と相互に排他的です。 ただし、`asp-page` は、次のマークアップのサンプルで示されているように、`asp-route-{value}` とともに使用してルーティングを制御できます。

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspPageAspRouteId)]

生成される HTML:

```html
<a href="/Attendee?attendeeid=10">View Attendee</a>
```

### <a name="asp-page-handler"></a>asp-page-handler

[Asp ページハンドラー](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.PageHandler*)属性は、ページで使用され Razor ます。 特定のページ ハンドラーへのリンクが目的です。

次のページ ハンドラーがあるとします。

[!code-csharp[](samples/TagHelpersBuiltIn/Pages/Attendee.cshtml.cs?name=snippet_OnGetProfileHandler)]

ページ モデルの関連するマークアップが `OnGetProfile` ページ ハンドラーにリンクしています。 ページ ハンドラーのメソッド名の `On<Verb>` プレフィックスは `asp-page-handler` 属性値では省略されることに注意してください。 メソッドが非同期の場合は、`Async` サフィックスも省略されます。

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspPageHandler)]

生成される HTML:

```html
<a href="/Attendee?attendeeid=12&handler=Profile">Attendee Profile</a>
```

## <a name="additional-resources"></a>その他のリソース

* <xref:mvc/controllers/areas>
* <xref:razor-pages/index>
* <xref:mvc/compatibility-version>
