---
title: ASP.NET Core の部分ビュー
author: ardalis
description: 部分ビューを使用して大規模なマークアップ ファイルを分割し、ASP.NET Core アプリの Web ページ間で共通するマークアップの重複を減らす方法について説明します。
ms.author: riande
ms.custom: mvc
ms.date: 06/12/2019
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
uid: mvc/views/partial
ms.openlocfilehash: 0a8e4a4fdecd657840c6c02424ffffa64d4ab473
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586879"
---
# <a name="partial-views-in-aspnet-core"></a>ASP.NET Core の部分ビュー

作成者: [Steve Smith](https://ardalis.com/)、[Maher JENDOUBI](https://twitter.com/maherjend)、[Rick Anderson](https://twitter.com/RickAndMSFT)、[Scott Sauber](https://twitter.com/scottsauber)

部分ビューは、 [Razor](xref:mvc/views/razor)  [`@page`](xref:mvc/views/razor#page) 別のマークアップファイルの表示出力 *内で* HTML 出力をレンダリングするディレクティブのないマークアップファイル (cshtml) です。

::: moniker range=">= aspnetcore-2.1"

*部分ビュー* という用語は、マークアップファイルが *ビュー* と呼ばれる MVC アプリを開発する場合や、 Razor マークアップファイルが *ページ* と呼ばれるページアプリを開発する場合に使用します。 このトピックでは、一般的に MVC ビューと Razor ページページを *マークアップファイル* と呼びます。

::: moniker-end

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/partial/sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="when-to-use-partial-views"></a>部分ビューを使用する状況

部分ビューは、次の場合に効果的な方法です。

* 大規模なマークアップ ファイルをより小さなコンポーネントに分割します。

  複数の論理パーツで構成された大規模かつ複雑なマークアップ ファイルでは、各パーツが部分ビューに分かれた状態で操作すると便利です。 全体のページ構造と部分ビューへの参照がマークアップのみに含まれるため、マークアップ ファイルのコードは管理性に優れています。
* マークアップ ファイル間で共通するマークアップ コンテンツの重複を減らします。

  マークアップ ファイル間で同じマークアップ要素が使用されている場合、1 つの部分ビュー ファイルとなり、部分ビューによってマークアップ コンテンツの重複が除去されます。 部分ビュー内でマークアップが変更された場合、その部分ビューを使用しているマークアップ ファイルの出力表示が更新されます。

共通するレイアウト要素を維持するために、部分ビューを使用しないでください。 共通するレイアウト要素は、[_Layout.cshtml](xref:mvc/views/layout) ファイルで指定する必要があります。

マークアップを表示するために、複雑な表示ロジックやコード実行が必要になる場合は、部分ビューを使用しないでください。 部分ビューの代わりに、[ビュー コンポーネント](xref:mvc/views/view-components)を使用してください。

## <a name="declare-partial-views"></a>部分ビューを宣言する

::: moniker range=">= aspnetcore-2.0"

部分ビューは、  [`@page`](xref:mvc/views/razor#page) *Views* フォルダー (MVC) または *pages* フォルダー ( Razor ページ) 内でディレクティブが保持されていない、cshtml マークアップファイルです。

ASP.NET Core MVC では、コントローラーの <xref:Microsoft.AspNetCore.Mvc.ViewResult> が、ビューまたは部分ビューのどちらかを返すことができます。 Razorページでは、は <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> オブジェクトとして表される部分ビューを返すことができ <xref:Microsoft.AspNetCore.Mvc.PartialViewResult> ます。 部分ビューの参照と表示については、「[部分ビューを参照する](#reference-a-partial-view)」セクションで説明します。

MVC ビューやページ レンダリングとは異なり、部分ビューは *_ViewStart.cshtml* を実行しません。 *_ViewStart.cshtml* の詳細については、<xref:mvc/views/layout> を参照してください。

部分ビューのファイル名は、多くの場合アンダースコア (`_`) から始まります。 この名前付け規則は必須ではありませんが、ビューおよびページと部分ビューを視覚的に区別するのに役立ちます。

::: moniker-end

::: moniker range="< aspnetcore-2.0"

部分ビューは、*Views* フォルダー内で保持される *.cshtml* マークアップ ファイルです。

コントローラーの <xref:Microsoft.AspNetCore.Mvc.ViewResult> は、ビューまたは部分ビューのどちらかを返すことができます。 部分ビューの参照と表示については、「[部分ビューを参照する](#reference-a-partial-view)」セクションで説明します。

MVC ビューのレンダリングとは異なり、部分ビューは *_ViewStart.cshtml* を実行しません。 *_ViewStart.cshtml* の詳細については、<xref:mvc/views/layout> を参照してください。

部分ビューのファイル名は、多くの場合アンダースコア (`_`) から始まります。 この名前付け規則は必須ではありませんが、ビューと部分ビューを視覚的に区別するのに役立ちます。

::: moniker-end

## <a name="reference-a-partial-view"></a>部分ビューを参照する

::: moniker range=">= aspnetcore-2.0"

### <a name="use-a-partial-view-in-a-razor-pages-pagemodel"></a>PageModel ページで部分ビューを使用する Razor

ASP.NET Core 2.0 または2.1 で、次のハンドラーメソッドは、 *\_ authorpartialrp. cshtml* 部分ビューを応答にレンダリングします。

```csharp
public IActionResult OnGetPartial() =>
    new PartialViewResult
    {
        ViewName = "_AuthorPartialRP",
        ViewData = ViewData,
    };
```

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

ASP.NET Core 2.2 以降では、別の方法としてハンドラー メソッドによって、<xref:Microsoft.AspNetCore.Mvc.RazorPages.PageBase.Partial*> メソッドを呼び出して `PartialViewResult` オブジェクトを生成することもできます。

[!code-csharp[](partial/sample/PartialViewsSample/Pages/DiscoveryRP.cshtml.cs?name=snippet_OnGetPartial)]

::: moniker-end

### <a name="use-a-partial-view-in-a-markup-file"></a>マークアップ ファイルで部分ビューを使用する

::: moniker range=">= aspnetcore-2.1"

マークアップ ファイル内で、部分ビューを参照する方法はいくつかあります。 アプリでは、次の非同期レンダリング手法のいずれかを使用することをお勧めします。

* [部分タグ ヘルパー](#partial-tag-helper)
* [非同期の HTML ヘルパー](#asynchronous-html-helper)

::: moniker-end

::: moniker range="< aspnetcore-2.1"

マークアップ ファイル内で、部分ビューを参照するには、次の 2 つの方法があります。

* [非同期の HTML ヘルパー](#asynchronous-html-helper)
* [同期の HTML ヘルパー](#synchronous-html-helper)

アプリでは、[非同期の HTML ヘルパー](#asynchronous-html-helper)を使用することをお勧めします。

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

### <a name="partial-tag-helper"></a>部分タグ ヘルパー

[部分タグ ヘルパー](xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper)では、ASP.NET Core 2.1 以降が必要です。

部分タグ ヘルパーでは、非同期でコンテンツをレンダリングして、HTML のような構文を使用します。

```cshtml
<partial name="_PartialName" />
```

ファイル拡張子がある場合、タグ ヘルパーは、部分ビューを呼び出すマークアップ ファイルと同じフォルダー内に必ず配置されている部分ビューを参照します。

```cshtml
<partial name="_PartialName.cshtml" />
```

次の例では、アプリ ルートから部分ビューを参照しています。 チルダとスラッシュ (`~/`) またはスラッシュ (`/`) から始まるパスは、次のようにアプリ ルートを参照します。

**Razor トピック**

```cshtml
<partial name="~/Pages/Folder/_PartialName.cshtml" />
<partial name="/Pages/Folder/_PartialName.cshtml" />
```

**MVC**

```cshtml
<partial name="~/Views/Folder/_PartialName.cshtml" />
<partial name="/Views/Folder/_PartialName.cshtml" />
```

次の例では、相対パスを使って部分ビューを参照しています。

```cshtml
<partial name="../Account/_PartialName.cshtml" />
```

詳細については、「<xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper>」を参照してください。

::: moniker-end

### <a name="asynchronous-html-helper"></a>非同期の HTML ヘルパー

HTML ヘルパーを使用している場合、ベスト プラクティスは <xref:Microsoft.AspNetCore.Mvc.Rendering.HtmlHelperPartialExtensions.PartialAsync*> を使用することです。 `PartialAsync` は、<xref:System.Threading.Tasks.Task%601> でラップされた <xref:Microsoft.AspNetCore.Html.IHtmlContent> 型を返します。 待機中の呼び出しの前に `@` 文字を付与することで、メソッドが参照されます。

```cshtml
@await Html.PartialAsync("_PartialName")
```

ファイル拡張子がある場合、HTML ヘルパーは、部分ビューを呼び出すマークアップ ファイルと同じフォルダー内に必ず配置されている部分ビューを参照します。

```cshtml
@await Html.PartialAsync("_PartialName.cshtml")
```

次の例では、アプリ ルートから部分ビューを参照しています。 チルダとスラッシュ (`~/`) またはスラッシュ (`/`) から始まるパスは、次のようにアプリ ルートを参照します。

::: moniker range=">= aspnetcore-2.1"

**Razor トピック**

```cshtml
@await Html.PartialAsync("~/Pages/Folder/_PartialName.cshtml")
@await Html.PartialAsync("/Pages/Folder/_PartialName.cshtml")
```

**MVC**

::: moniker-end

```cshtml
@await Html.PartialAsync("~/Views/Folder/_PartialName.cshtml")
@await Html.PartialAsync("/Views/Folder/_PartialName.cshtml")
```

次の例では、相対パスを使って部分ビューを参照しています。

```cshtml
@await Html.PartialAsync("../Account/_LoginPartial.cshtml")
```

代わりに、<xref:Microsoft.AspNetCore.Mvc.Rendering.HtmlHelperPartialExtensions.RenderPartialAsync*> を使って部分ビューをレンダリングすることもできます。 このメソッドは <xref:Microsoft.AspNetCore.Html.IHtmlContent> を返しません。 レンダリングされた出力を直接応答にストリーミングします。 メソッドは結果を返さないため、コードブロック内で呼び出す必要があり Razor ます。

[!code-cshtml[](partial/sample/PartialViewsSample/Views/Home/Discovery.cshtml?name=snippet_RenderPartialAsync)]

`RenderPartialAsync` は表示されるコンテンツをストリーム配信するため、一部のシナリオではより高いパフォーマンスが得られます。 パフォーマンスが重要な場合には、両方の手法を使用してページのベンチマークを実行し、より迅速に応答を生成する手法を採用します。

### <a name="synchronous-html-helper"></a>同期の HTML ヘルパー

<xref:Microsoft.AspNetCore.Mvc.Rendering.HtmlHelperPartialExtensions.Partial*> と <xref:Microsoft.AspNetCore.Mvc.Rendering.HtmlHelperPartialExtensions.RenderPartial*> はそれぞれ、`PartialAsync` と `RenderPartialAsync` の同期に相当します。 デッドロックが発生するシナリオがあるため、同期に相当する機能はお勧めしません。 同期メソッドは、将来のリリースでは削除対象になります。

> [!IMPORTANT]
> コードを実行する必要がある場合は、部分ビューの代わりに[ビュー コンポーネント](xref:mvc/views/view-components)を使用してください。

::: moniker range=">= aspnetcore-2.1"

`Partial` または `RenderPartial` を呼び出すと、Visual Studio アナライザーの警告が発生します。 たとえば、`Partial` を指定すると、次の警告メッセージが生成されます。

> IHtmlHelper.Partial を使用すると、アプリケーションのデッドロックが発生する可能性があります。 &lt;部分&gt;タグ ヘルパーまたは IHtmlHelper.PartialAsync を使用することを検討してください。

`@Html.Partial` の呼び出しを、`@await Html.PartialAsync` または[部分タグ ヘルパー](xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper)に置き換えます。 部分タグ ヘルパーの移行の詳細については、「[HTML ヘルパーから移行する](xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper#migrate-from-an-html-helper)」を参照してください。

::: moniker-end

## <a name="partial-view-discovery"></a>部分ビューの検出

部分ビューがファイル拡張子を指定せずに名前で参照された場合、所定の順序で次の場所が検索されます。

::: moniker range=">= aspnetcore-2.1"

**Razor トピック**

1. 現在実行中のページのフォルダー
1. ページのフォルダーの上にあるディレクトリ グラフ
1. `/Shared`
1. `/Pages/Shared`
1. `/Views/Shared`

**MVC**

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

1. `/Areas/<Area-Name>/Views/<Controller-Name>`
1. `/Areas/<Area-Name>/Views/Shared`
1. `/Views/Shared`
1. `/Pages/Shared`

::: moniker-end

::: moniker range="< aspnetcore-2.0"

1. `/Areas/<Area-Name>/Views/<Controller-Name>`
1. `/Areas/<Area-Name>/Views/Shared`
1. `/Views/Shared`

::: moniker-end

部分ビューの検索には、次の規則が適用されます。

* 部分ビューが異なるフォルダー内にある場合は、同じファイル名の別の部分ビューが許可されます。
* ファイル拡張子を指定せずに部分ビューを名前で参照しており、かつ、部分ビューが呼び出し元のフォルダーと *Shared* フォルダーの両方に存在する場合、呼び出し元のフォルダーにある部分ビューが、部分ビューとして機能します。 部分ビューが呼び出し元のフォルダーに存在しない場合、部分ビューは *Shared* フォルダーから提供されます。 *Shared* フォルダーの部分ビューは、"*共有の部分ビュー*" または "*既定の部分ビュー*" と呼ばれます。
* 部分ビューは、  &mdash; 呼び出しによって循環参照が形成されていない場合に、部分ビューを連結して別の部分ビューを呼び出すことができます。 相対パスは常に、ファイルのルートや親ではなく、現在のファイルを基準とします。

> [!NOTE]
> [Razor](xref:mvc/views/razor) `section` 部分ビューで定義されたは、親マークアップファイルからは見えません。 `section` は定義されている部分ビューにのみ表示されます。

## <a name="access-data-from-partial-views"></a>部分ビューからデータにアクセスする

部分ビューがインスタンス化されると、親の `ViewData` ディクショナリの *コピー* を取得します。 親ビュー内のデータに対して行われた更新は、親ビューでは保持されません。 部分ビューで変更された `ViewData` は、部分ビューから返されるときに失われます。

次の例に、[ViewDataDictionary](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.viewdatadictionary) のインスタンスを部分ビューに渡す方法を示します。

```cshtml
@await Html.PartialAsync("_PartialName", customViewData)
```

部分ビューにモデルを渡すことができます。 モデルは、カスタム オブジェクトでもかまいません。 `PartialAsync` (呼び出し元にコンテンツのブロックを表示する) または `RenderPartialAsync` (コンテンツを出力にストリーム配信する) を使って、モデルを渡すことができます。

```cshtml
@await Html.PartialAsync("_PartialName", model)
```

::: moniker range=">= aspnetcore-2.1"

**Razor トピック**

サンプル アプリの次のマークアップは、*Pages/ArticlesRP/ReadRP.cshtml* ページが元になっています。 ページには、2 つの部分ビューが含まれています。 2 番目の部分ビューは、モデルと `ViewData` を部分ビューに渡します。 `ViewDataDictionary` のコンストラクター オーバーロードは、既存の `ViewData` ディクショナリを維持したまま、新しい `ViewData` ディクショナリを渡すために使用されます。

[!code-cshtml[](partial/sample/PartialViewsSample/Pages/ArticlesRP/ReadRP.cshtml?name=snippet_ReadPartialViewRP&highlight=5,15-20)]

*Pages/Shared/_AuthorPartialRP.cshtml* は、*ReadRP.cshtml* マークアップ ファイルで参照される最初の部分ビューです。

[!code-cshtml[](partial/sample/PartialViewsSample/Pages/Shared/_AuthorPartialRP.cshtml)]

*Pages/ArticlesRP/_ArticleSectionRP.cshtml* は、*ReadRP.cshtml* マークアップ ファイルで参照される 2 番目の部分ビューです。

[!code-cshtml[](partial/sample/PartialViewsSample/Pages/ArticlesRP/_ArticleSectionRP.cshtml)]

**MVC**

::: moniker-end

サンプル アプリの以下のマークアップは、*Views/Articles/Read.cshtml* ビューを示しています。 ビューには、2 つの部分ビューが含まれています。 2 番目の部分ビューは、モデルと `ViewData` を部分ビューに渡します。 `ViewDataDictionary` のコンストラクター オーバーロードは、既存の `ViewData` ディクショナリを維持したまま、新しい `ViewData` ディクショナリを渡すために使用されます。

[!code-cshtml[](partial/sample/PartialViewsSample/Views/Articles/Read.cshtml?name=snippet_ReadPartialView&highlight=5,15-20)]

*Views/Shared/_AuthorPartial.cshtml* は、*Read.cshtml* マークアップ ファイルで参照される最初の部分ビューです。

[!code-cshtml[](partial/sample/PartialViewsSample/Views/Shared/_AuthorPartial.cshtml)]

*Views/Articles/_ArticleSection.cshtml* は、*Read.cshtml* マークアップ ファイルで参照される 2 番目の部分ビューです。

[!code-cshtml[](partial/sample/PartialViewsSample/Views/Articles/_ArticleSection.cshtml)]

この部分は実行時に、親のマークアップファイルの出力表示にレンダリングされます。親ビュー自体は共有されている *_Layout.cshtml* 内にレンダリングされます。 最初の部分ビューは、記事の作成者の名前と発行日を表示します。

> アブラハム リンカーン
>
> この部分ビューは&lt;共有されている部分ビューのファイル パス&gt;が元になっている。
> 1863 年 11 月 19 日 12 時 00 分 00 秒

2 番目の部分ビューは、次に示す記事のセクションを表示します。

> セクション 1 のインデックス: 0
>
> 87 年前...
>
> セクション 2 のインデックス: 1
>
> 大きな内紛の渦中で、試した...
>
> セクション 3 のインデックス: 2
>
> しかし、広義では、我々が献身することはできない...

## <a name="additional-resources"></a>その他のリソース

::: moniker range=">= aspnetcore-2.1"

* [ASP.NET Core の Razor 構文リファレンス](xref:mvc/views/razor)
* <xref:mvc/views/tag-helpers/intro>
* <xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper>
* <xref:mvc/views/view-components>
* <xref:mvc/controllers/areas>

::: moniker-end

::: moniker range="< aspnetcore-2.1"

* [ASP.NET Core の Razor 構文リファレンス](xref:mvc/views/razor)
* <xref:mvc/views/view-components>
* <xref:mvc/controllers/areas>

::: moniker-end
