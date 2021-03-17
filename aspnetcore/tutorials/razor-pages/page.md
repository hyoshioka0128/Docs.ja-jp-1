---
title: パート 3、スキャフォールディングされた Razor Pages
author: rick-anderson
description: Razor ページのチュートリアル シリーズのパート 3。
ms.author: riande
ms.date: 09/25/2020
ms.custom: contperf-fy21q2
no-loc:
- Index
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
uid: tutorials/razor-pages/page
ms.openlocfilehash: 049e7764766a4d5d535f7d7959a3554b040607c5
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588504"
---
# <a name="part-3-scaffolded-razor-pages-in-aspnet-core"></a>パート 3、ASP.NET Core でスキャフォールディングされた Razor ページ

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

このチュートリアルでは、[前のチュートリアル](xref:tutorials/razor-pages/model)でスキャフォールディングによって作成された Razor ページについて説明します。

::: moniker range=">= aspnetcore-5.0"

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

::: moniker-end

::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

## <a name="the-create-delete-details-and-edit-pages"></a>[作成]、[削除]、[詳細]、および [編集] ページ

*Pages/Movies/Index.cshtml.cs* ページ モデルを確認します。

[!code-csharp[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs)]

Razor ページは `PageModel` から派生します。 慣例により、`PageModel` 派生クラスは `<PageName>Model` と呼ばれます。 コンストラクターは[依存性の注入](xref:fundamentals/dependency-injection)を使用して、`RazorPagesMovieContext` をページに追加します。

[!code-csharp[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet1&highlight=5)]

Entity Framework での非同期プログラミングの詳細については、「[非同期コード](xref:data/ef-rp/intro#asynchronous-code)」を参照してください。

ページに対して要求が行われると、`OnGetAsync` メソッドは Razor ページにムービーのリストを返します。 Razor ページでは、`OnGetAsync` または `OnGet` が呼び出され、ページの状態が初期化されます。 この場合、`OnGetAsync` でムービーのリストを取得し、表示します。

`OnGet` によって `void` が返される場合、または `OnGetAsync` によって `Task` が返される場合は、return ステートメントは使用されません。 たとえば、[プライバシー] ページでは、次のようになります。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Privacy.cshtml.cs?name=snippet)]

戻り値の型が `IActionResult` または `Task<IActionResult>` の場合は、return ステートメントを指定する必要があります。 たとえば、*Pages/Movies/Create.cshtml.cs* `OnPostAsync` メソッドでは、次のようになります。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Create.cshtml.cs?name=snippet)]

<a name="index"></a>次のように、*Pages/Movies/Index.cshtml* Razor ページを確認します。

[!code-cshtml[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Index.cshtml)]

Razor では、HTML から C# または Razor 固有のマークアップに移行できます。 `@` シンボルの後に [Razor で予約済みのキーワード](xref:mvc/views/razor#razor-reserved-keywords)が続いている場合は、Razor 固有のマークアップに移行します。それ以外の場合は、C# に移行します。

### <a name="the-page-directive"></a>@page ディレクティブ

`@page` Razor ディレクティブを使うと、ファイルが MVC アクションになります。つまり、これで要求を処理できます。 `@page` はページ上で最初の Razor ディレクティブである必要があります。 `@page` および `@model` は、Razor 固有のマークアップへの移行の例です。 詳細については、「[Razor の構文](xref:mvc/views/razor#razor-syntax)」を参照してください。

<a name="md"></a>

### <a name="the-model-directive"></a>@model ディレクティブ

[!code-cshtml[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Index.cshtml?range=1-2&highlight=2)]

`@model` ディレクティブは、Razor ページに渡されるモデルの型を指定します。 前の例の `@model` 行は、Razor ページで `PageModel` 派生クラスを使用できるようにします。 モデルは、ページの `@Html.DisplayNameFor` および `@Html.DisplayFor` [HTML ヘルパーで](/aspnet/mvc/overview/older-versions-1/views/creating-custom-html-helpers-cs#understanding-html-helpers)使用されます。


次の HTML ヘルパーで使用されるラムダ式を確認します。

```cshtml
@Html.DisplayNameFor(model => model.Movie[0].Title)
```

<xref:System.Web.Mvc.Html.DisplayNameExtensions.DisplayNameFor%2A?displayProperty=nameWithType> HTML ヘルパーは、ラムダ式で参照される `Title` プロパティを検査し、表示名を判別します。 ラムダ式は評価されるのではなく検査されます。 これは、`model`、`model.Movie`、または `model.Movie[0]` が `null` または空である場合、アクセス違反がないことを意味します。 ラムダ式が (`@Html.DisplayFor(modelItem => item.Title)` などを使用して) 評価される場合は、モデルのプロパティ値が評価されます。

### <a name="the-layout-page"></a>レイアウト ページ

メニューのリンク ( **[RazorPagesMovie]** 、 **[ホーム]** 、 **[プライバシー]** ) を選択します。 各ページには同じメニューのレイアウトが表示されます。 メニューのレイアウトは、*Pages/Shared/_Layout.cshtml* ファイルに実装されています。

*Pages/Shared/_Layout.cshtml* ファイルを開いて確認します。

[レイアウト](xref:mvc/views/layout) テンプレートを使用すると、HTML コンテナーのレイアウトを次のようにすることができます。

* 1 つの場所で指定される。
* サイトの複数のページに適用される。

`@RenderBody()` という行を見つけます。 `RenderBody` は、ページ固有のビューがすべて表示されるプレースホルダーで、レイアウト ページに "*ラップ*" されます。 たとえば、 **[プライバシー]** リンクを選択すると、`RenderBody` メソッド内で *Pages/Privacy.cshtml* ビューがレンダリングされます。

<a name="vd"></a>

### <a name="viewdata-and-layout"></a>ViewData とレイアウト

*Pages/Movies/Index.cshtml* ファイルの次のマークアップを考えてみます。

[!code-cshtml[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Index.cshtml?range=1-6&highlight=4-999)]

上の強調表示されたマークアップは、Razor の C# への移行例です。 `{` 文字と `}` 文字で C# コードのブロックを囲みます。

`PageModel` 基底クラスには `ViewData` 辞書プロパティが含まれており、これを使用してビューにデータを渡すことができます。 オブジェクトは、****キー値**** のパターンを使用して、`ViewData` 辞書に追加されます。 上のサンプルでは、`Title` プロパティが `ViewData` 辞書に追加されます。

`Title` プロパティは *Pages/Shared/_Layout.cshtml* ファイルで使用されます。 次のマークアップは、 *_Layout.cshtml* ファイルの最初の数行を示しています。

<!-- We need a snapshot copy of layout because we are changing in the next step. -->

[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/NU/_Layout.cshtml?highlight=6)]

`@*Markup removed for brevity.*@` 行は Razor コメントです。 HTML コメント `<!-- -->` とは異なり、Razor コメントはクライアントには送信されません。 詳細については、「[MDN Web ドキュメント: HTML の概要](https://developer.mozilla.org/docs/Learn/HTML/Introduction_to_HTML/Getting_started#HTML_comments)」をご覧ください。

### <a name="update-the-layout"></a>レイアウトの更新

1. **RazorPagesMovie** ではなく **Movie** が表示されるように、*Pages/Shared/_Layout.cshtml* ファイルの `<title>` 要素を変更します。

   [!code-cshtml[](razor-pages-start/sample/RazorPagesMovie30/Pages/Shared/_Layout.cshtml?range=1-6&highlight=6)]

1. *Pages/Shared/_Layout.cshtml* ファイルで次のアンカー要素を見つけます。

   ```cshtml
   <a class="navbar-brand" asp-area="" asp-page="/Index">RazorPagesMovie</a>
   ```

1. 上の要素を次のマークアップに置き換えます。

   ```cshtml
   <a class="navbar-brand" asp-page="/Movies/Index">RpMovie</a>
   ```

   上のアンカー要素は[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)です。 この場合は、[アンカー タグ ヘルパー](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper)です。 `asp-page="/Movies/Index"` タグ ヘルパー属性と値で、`/Movies/Index` Razor ページへのリンクが作成されます。 `asp-area` 属性の値が空なので、リンクではこの区分が使用されていません。 詳細については、[区分](xref:mvc/controllers/areas)に関する記事を参照してください。

1. 変更内容を保存し、**RpMovie** リンクを選択してアプリをテストします。 問題がある場合は、GitHub の [_Layout.cshtml](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Pages/Shared/_Layout.cshtml) ファイルを参照してください。

1. **[Home]\(ホーム\)** 、 **[RpMovie]** 、 **[Create]/(作成/)** 、 **[Edit]\(編集\)** 、 **[Delete]\(削除\)** のリンクをテストします。 各ページで、ブラウザー タブで表示できるタイトルを設定します。ページをブックマークすると、ブックマークでタイトルが使用されます。

> [!NOTE]
> `Price` フィールドに小数点のコンマを入力できない場合があります。 小数点にコンマ (",") を使い、英語 (米国) 以外の日付形式を使う英語以外のロケールの [jQuery 検証](https://jqueryvalidation.org/)をサポートするには、アプリをグローバル化する手順を行う必要があります。 小数点のコンマの追加方法については、[こちらの GitHub issue 4076](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420) を参照してください。

`Layout` プロパティは *Pages/_ViewStart.cshtml* ファイルで設定されています。

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie30/Pages/_ViewStart.cshtml)]

上のマークアップでは、*Pages* フォルダーの下のすべての Razor ファイルに対して、レイアウト ファイルを *Pages/Shared/_Layout.cshtml* に設定します。 詳細については、「[Layout](xref:razor-pages/index#layout)」 (レイアウト) を参照してください。

### <a name="the-create-page-model"></a>Create ページのモデル

*Pages/Movies/Create.cshtml.cs* ページ モデルを確認します。

[!code-csharp[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Create.cshtml.cs?name=snippetALL)]

`OnGet` メソッドは、ページに必要な状態を初期化します。 [作成] ページには初期化する状態はないため、`Page` が返されます。 このチュートリアルの後半では、状態を初期化する `OnGet` の例を示します。 `Page` メソッドでは、*Create.cshtml* ページをレンダリングする `PageResult` オブジェクトが作成されます。

`Movie` プロパティでは [[BindProperty]](xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute) 属性を使用して、[モデル バインド](xref:mvc/models/model-binding)にオプトインします。 [作成] フォームでフォーム値が投稿されると、ASP.NET Core ランタイムが投稿された値を `Movie` モデルにバインドします。

`OnPostAsync` メソッドは、ページでフォーム データが投稿されたときに実行されます。

[!code-csharp[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Create.cshtml.cs?name=snippetPost)]

モデル エラーがある場合は、投稿されたフォーム データと共にフォームが再表示されます。 ほとんどのモデル エラーは、フォームが投稿される前にクライアント側でキャッチできます。 モデル エラーの例では、日付に変換できない日付フィールドの値が投稿されています。 クライアント側の検証とモデルの検証については、チュートリアルの後半で説明します。

モデル エラーがない場合は、次のようになります。

* データが保存されます。
* ブラウザーは [Index] ページにリダイレクトされます。

### <a name="the-create-razor-page"></a>Razor の作成ページ

次のように、*Pages/Movies/Create.cshtml* Razor ページ ファイルを確認します。

[!code-cshtml[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Create.cshtml)]

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Visual Studio に、タグ ヘルパーで使用される独特な太字のフォントで次のタグが表示されます。

* `<form method="post">`
* `<div asp-validation-summary="ModelOnly" class="text-danger"></div>`
* `<label asp-for="Movie.Title" class="control-label"></label>`
* `<input asp-for="Movie.Title" class="form-control" />`
* `<span asp-validation-for="Movie.Title" class="text-danger"></span>`

![Create.cshtml ページの VS17 ビュー](page/_static/th3.png)

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

上記のマークアップには、次のタグ ヘルパーが示されています。

* `<form method="post">`
* `<div asp-validation-summary="ModelOnly" class="text-danger"></div>`
* `<label asp-for="Movie.Title" class="control-label"></label>`
* `<input asp-for="Movie.Title" class="form-control" />`
* `<span asp-validation-for="Movie.Title" class="text-danger"></span>`

---

`<form method="post">` 要素は[フォーム タグ ヘルパー](xref:mvc/views/working-with-forms#the-form-tag-helper)です。 フォーム タグ ヘルパーには自動的に[偽造防止トークン](xref:security/anti-request-forgery)が含まれます。

スキャフォールディング エンジンでは、次のような、(ID を除く) モデルの各フィールドの Razor マークアップが作成されます。

[!code-cshtml[](~/tutorials/razor-pages/razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Create.cshtml?range=15-20)]

[検証タグ ヘルパー](xref:mvc/views/working-with-forms#the-validation-tag-helpers) (`<div asp-validation-summary` と `<span asp-validation-for`) には検証エラーが表示されます。 検証については、後で詳しく説明します。

[ラベル タグ ヘルパー](xref:mvc/views/working-with-forms#the-label-tag-helper) (`<label asp-for="Movie.Title" class="control-label"></label>`) は、`Title` プロパティのラベル キャプションと `[for]` 属性を生成します。

[入力タグ ヘルパー](xref:mvc/views/working-with-forms) (`<input asp-for="Movie.Title" class="form-control">`) は [DataAnnotations](/aspnet/mvc/overview/older-versions/mvc-music-store/mvc-music-store-part-6) 属性を使用し、クライアント側で jQuery 検証に必要な HTML 属性を生成します。

`<form method="post">` などのタグ ヘルパーについては、「[ASP.NET Core のタグ ヘルパー](xref:mvc/views/tag-helpers/intro)」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

> [!div class="step-by-step"]
> [前へ:モデルの追加](xref:tutorials/razor-pages/model)
> [次へ:データベースの操作](xref:tutorials/razor-pages/sql)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

## <a name="the-create-delete-details-and-edit-pages"></a>[作成]、[削除]、[詳細]、および [編集] ページ

*Pages/Movies/Index.cshtml.cs* ページ モデルを確認します。

[!code-csharp[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Index.cshtml.cs)]

Razor ページは `PageModel` から派生します。 慣例により、`PageModel` 派生クラスは `<PageName>Model` と呼ばれます。 コンストラクターは[依存性の注入](xref:fundamentals/dependency-injection)を使用して、`RazorPagesMovieContext` をページに追加します。 スキャフォールディングされたページでは、このパターンに従います。 Entity Framework での非同期プログラミングの詳細については、「[非同期コード](xref:data/ef-rp/intro#asynchronous-code)」を参照してください。

ページに対して要求が行われると、`OnGetAsync` メソッドは Razor ページにムービーのリストを返します。 `OnGetAsync` または `OnGet` が Razor ページで呼び出され、ページの状態が初期化されます。 この場合、`OnGetAsync` でムービーのリストを取得し、表示します。

`OnGet` から `void` が返される場合、または `OnGetAsync` から `Task` が返される場合、return メソッドは使用されません。 戻り値の型が `IActionResult` または `Task<IActionResult>` の場合は、return ステートメントを指定する必要があります。 たとえば、*Pages/Movies/Create.cshtml.cs* `OnPostAsync` メソッドでは、次のようになります。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Create.cshtml.cs?name=snippet)]

<a name="index"></a>次のように、*Pages/Movies/Index.cshtml* Razor ページを確認します。

[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Index.cshtml)]

Razor では、HTML から C# または Razor 固有のマークアップに移行できます。 `@` シンボルの後に [Razor で予約済みのキーワード](xref:mvc/views/razor#razor-reserved-keywords)が続いている場合は、Razor 固有のマークアップに移行します。それ以外の場合は、C# に移行します。

`@page` Razor ディレクティブを使うと、ファイルが MVC アクションになります。つまり、これで要求を処理できます。 `@page` はページ上で最初の Razor ディレクティブである必要があります。 `@page` は、Razor 固有のマークアップへの移行の例です。 詳細については、「[Razor の構文](xref:mvc/views/razor#razor-syntax)」を参照してください。

<a name="md"></a>

### <a name="the-model-directive"></a>@model ディレクティブ

[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Index.cshtml?range=1-2&highlight=2)]

`@model` ディレクティブは、Razor ページに渡されるモデルの型を指定します。 前の例の `@model` 行は、Razor ページで `PageModel` 派生クラスを使用できるようにします。 モデルは、ページの `@Html.DisplayNameFor` および `@Html.DisplayFor` [HTML ヘルパーで](/aspnet/mvc/overview/older-versions-1/views/creating-custom-html-helpers-cs#understanding-html-helpers)使用されます。

### <a name="html-helpers"></a>HTML ヘルパー

次の HTML ヘルパーで使用されるラムダ式を確認します。

```cshtml
@Html.DisplayNameFor(model => model.Movie[0].Title)
```

<xref:System.Web.Mvc.Html.DisplayNameExtensions.DisplayNameFor%2A?displayProperty=nameWithType> HTML ヘルパーは、ラムダ式で参照される `Title` プロパティを検査し、表示名を判別します。 ラムダ式は評価されるのではなく検査されます。 これは、`model`、`model.Movie`、または `model.Movie[0]` が `null` または空である場合、アクセス違反がないことを意味します。 ラムダ式が (`@Html.DisplayFor(modelItem => item.Title)` などを使用して) 評価される場合は、モデルのプロパティ値が評価されます。
### <a name="the-layout-page"></a>レイアウト ページ

メニューのリンク ( **[RazorPagesMovie]** 、 **[ホーム]** 、 **[プライバシー]** ) を選択します。 各ページには同じメニューのレイアウトが表示されます。 メニューのレイアウトは、*Pages/Shared/_Layout.cshtml* ファイルに実装されています。

*Pages/Shared/_Layout.cshtml* ファイルを開いて確認します。

[[レイアウト]](xref:mvc/views/layout) テンプレートでは、1 か所でサイトの HTML コンテナー レイアウトを指定し、それをサイト内の複数のページに適用できます。 `@RenderBody()` という行を見つけます。 `RenderBody` は、作成したページ固有のビューがすべて表示されるプレースホルダーで、レイアウト ページに *ラップ* されます。 たとえば、 **[プライバシー]** リンクを選択した場合、`RenderBody` メソッド内で **Pages/Privacy.cshtml** ビューがレンダリングされます。

<a name="vd"></a>

### <a name="viewdata-and-layout"></a>ViewData とレイアウト

*Pages/Movies/Index.cshtml* ファイルの次のコードを考えてみます。

[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Index.cshtml?range=1-6&highlight=4-999)]

上の強調表示されたコードは、Razor の C# への移行例です。 `{` 文字と `}` 文字で C# コードのブロックを囲みます。

`PageModel` 基本クラスには `ViewData` 辞書プロパティがあり、これを使用して、ビューに渡すデータを追加できます。 キー/値のパターンを使用して、`ViewData` 辞書にオブジェクトを追加します。 上のサンプルでは、`Title` プロパティが `ViewData` 辞書に追加されます。

`Title` プロパティは *Pages/Shared/_Layout.cshtml* ファイルで使用されます。 次のマークアップは、 *_Layout.cshtml* ファイルの最初の数行を示しています。

<!-- We need a snapshot copy of layout because we are changing in the next step. -->

[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/NU/_Layout.cshtml?highlight=6-99)]

`@*Markup removed for brevity.*@` 行は Razor コメントです。 HTML コメント `<!-- -->` とは異なり、Razor コメントはクライアントには送信されません。 詳細については、「[MDN Web ドキュメント: HTML の概要](https://developer.mozilla.org/docs/Learn/HTML/Introduction_to_HTML/Getting_started#HTML_comments)」をご覧ください。

### <a name="update-the-layout"></a>レイアウトの更新

**RazorPagesMovie** ではなく **Movie** が表示されるように、*Pages/Shared/_Layout.cshtml* ファイルの `<title>` 要素を変更します。

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/Shared/_Layout.cshtml?range=1-6&highlight=6)]

*Pages/Shared/_Layout.cshtml* ファイルで次のアンカー要素を見つけます。

```cshtml
<a class="navbar-brand" asp-area="" asp-page="/Index">RazorPagesMovie</a>
```

上の要素を次のマークアップに置き換えます。

```cshtml
<a class="navbar-brand" asp-page="/Movies/Index">RpMovie</a>
```

上のアンカー要素は[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)です。 この場合は、[アンカー タグ ヘルパー](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper)です。 `asp-page="/Movies/Index"` タグ ヘルパー属性と値で、`/Movies/Index` Razor ページへのリンクが作成されます。 `asp-area` 属性の値が空なので、リンクではこの区分が使用されていません。 詳細については、[区分](xref:mvc/controllers/areas)に関する記事を参照してください。

変更内容を保存し、**RpMovie** リンクをクリックしてアプリをテストします。 問題がある場合は、GitHub の [_Layout.cshtml](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Pages/Shared/_Layout.cshtml) ファイルを参照してください。

その他のリンク ( **[Home]** 、 **[RpMovie]** 、 **[Create]** 、 **[Edit]** 、および **[Delete]** ) をテストします。 各ページで、ブラウザー タブで表示できるタイトルを設定します。ページをブックマークすると、ブックマークでタイトルが使用されます。

> [!NOTE]
> `Price` フィールドに小数点のコンマを入力できない場合があります。 小数点にコンマ (",") を使い、英語 (米国) 以外の日付形式を使う英語以外のロケールの [jQuery 検証](https://jqueryvalidation.org/)をサポートするには、アプリをグローバル化する手順を行う必要があります。 小数点のコンマの追加方法については、[こちらの GitHub issue 4076](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420) を参照してください。

`Layout` プロパティは *Pages/_ViewStart.cshtml* ファイルで設定されています。

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/_ViewStart.cshtml)]

上のマークアップでは、*Pages* フォルダーの下のすべての Razor ファイルに対して、レイアウト ファイルを *Pages/Shared/_Layout.cshtml* に設定します。 詳細については、「[Layout](xref:razor-pages/index#layout)」 (レイアウト) を参照してください。

### <a name="the-create-page-model"></a>Create ページのモデル

*Pages/Movies/Create.cshtml.cs* ページ モデルを確認します。

[!code-csharp[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Create.cshtml.cs?name=snippetALL)]

`OnGet` メソッドは、ページに必要な状態を初期化します。 [作成] ページには初期化する状態はないため、`Page` が返されます。 このチュートリアルで後ほど、`OnGet` メソッドの状態の初期化を確認できます。 `Page` メソッドでは、*Create.cshtml* ページをレンダリングする `PageResult` オブジェクトが作成されます。

`Movie` プロパティでは [BindProperty]<xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute> 属性を使用して、[モデル バインド](xref:mvc/models/model-binding)にオプトインします。 [作成] フォームでフォーム値が投稿されると、ASP.NET Core ランタイムが投稿された値を `Movie` モデルにバインドします。

`OnPostAsync` メソッドは、ページでフォーム データが投稿されたときに実行されます。

[!code-csharp[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Create.cshtml.cs?name=snippetPost)]

モデル エラーがある場合は、投稿されたフォーム データと共にフォームが再表示されます。 ほとんどのモデル エラーは、フォームが投稿される前にクライアント側でキャッチできます。 モデル エラーの例では、日付に変換できない日付フィールドの値が投稿されています。 クライアント側の検証とモデルの検証については、チュートリアルの後半で説明します。

モデル エラーがない場合、データは保存され、ブラウザーは [Index] ページにリダイレクトされます。

### <a name="the-create-razor-page"></a>Razor の作成ページ

次のように、*Pages/Movies/Create.cshtml* Razor ページ ファイルを確認します。

[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Create.cshtml)]

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Visual Studio に、タグ ヘルパーで使用される独特な太字のフォントで `<form method="post">` タグが表示されます。

![Create.cshtml ページの VS17 ビュー](page/_static/th.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

`<form method="post">` などのタグ ヘルパーについては、「[ASP.NET Core のタグ ヘルパー](xref:mvc/views/tag-helpers/intro)」を参照してください。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

Visual Studio for Mac に、タグ ヘルパーで使用される独特な太字のフォントで `<form method="post">` タグが表示されます。

---

`<form method="post">` 要素は[フォーム タグ ヘルパー](xref:mvc/views/working-with-forms#the-form-tag-helper)です。 フォーム タグ ヘルパーには自動的に[偽造防止トークン](xref:security/anti-request-forgery)が含まれます。

スキャフォールディング エンジンでは、次のような、(ID を除く) モデルの各フィールドの Razor マークアップが作成されます。

[!code-cshtml[](~/tutorials/razor-pages/razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Create.cshtml?range=15-20)]

[検証タグ ヘルパー](xref:mvc/views/working-with-forms#the-validation-tag-helpers) (`<div asp-validation-summary` と `<span asp-validation-for`) には検証エラーが表示されます。 検証については、後で詳しく説明します。

[ラベル タグ ヘルパー](xref:mvc/views/working-with-forms#the-label-tag-helper) (`<label asp-for="Movie.Title" class="control-label"></label>`) は、`Title` プロパティのラベル キャプションと `[for]` 属性を生成します。

[入力タグ ヘルパー](xref:mvc/views/working-with-forms) (`<input asp-for="Movie.Title" class="form-control">`) は [DataAnnotations](/aspnet/mvc/overview/older-versions/mvc-music-store/mvc-music-store-part-6) 属性を使用し、クライアント側で jQuery 検証に必要な HTML 属性を生成します。

## <a name="additional-resources"></a>その他の技術情報

* [このチュートリアルの YouTube バージョン](https://youtu.be/zxgKjPYnOMM)

> [!div class="step-by-step"]
> [前へ:モデルの追加](xref:tutorials/razor-pages/model)
> [次へ:データベースの操作](xref:tutorials/razor-pages/sql)

::: moniker-end
