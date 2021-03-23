---
title: パート 6、検索の追加
author: rick-anderson
description: Razor ページのチュートリアル シリーズのパート 6。
ms.author: riande
ms.date: 12/05/2019
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
uid: tutorials/razor-pages/search
ms.openlocfilehash: 19e20923d5f4cd2bf8671df7558995d34da2cdf6
ms.sourcegitcommit: d5fa39765959738eed4bcf5ee0b207cefddb4873
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/14/2021
ms.locfileid: "103460431"
---
# <a name="part-6-add-search-to-aspnet-core-razor-pages"></a>パート 6、ASP.NET Core Razor ページへの検索の追加

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-5.0"

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

::: moniker-end

::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

次のセクションでは、*ジャンル* または *名前* による映画検索が追加されます。

強調表示されている次の using ステートメントとプロパティを *Pages/Movies/Index.cshtml.cs* に追加します。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_newProps&highlight=3,23,24,25,26,27)]

上のコードでは、次のようになります。

* `SearchString`: ユーザーが検索テキスト ボックスに入力したテキストが含まれる。 `SearchString` には、[`[BindProperty]`](xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute) 属性があります。 `[BindProperty]` により、プロパティと同じ名前に基づきフォーム値とクエリ文字列がバインドされます。 HTTP GET 要求でのバインドには `[BindProperty(SupportsGet = true)]` が必要です。
* `Genres`: ジャンル一覧が含まれる。 `Genres` により、ユーザーは一覧からジャンルを選択できます。 `SelectList` には `using Microsoft.AspNetCore.Mvc.Rendering;` が必要です。
* `MovieGenre`: ユーザーが選択する特定のジャンルが含まれる。 たとえば、"Western (西部劇)" です。
* `Genres` と `MovieGenre` は、このチュートリアルで後述します。

[!INCLUDE[](~/includes/bind-get.md)]

Index ページの `OnGetAsync` メソッドを次のコードで更新します。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_1stSearch)]

`OnGetAsync` メソッドの最初の行により、ムービーを選択する [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) クエリが作成されます。

```csharp
// using System.Linq;
var movies = from m in _context.Movie
             select m;
```

このクエリは、この時点では "**定義される**" のみであり、データベースに対して実行されているわけでは "*_ありません_*"。

`SearchString` プロパティが null でも空でもない場合、検索文字列で絞り込むようにムービークエリが変更されます。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_SearchNull)]

`s => s.Title.Contains()` コードは[ラムダ式](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions)です。 ラムダは、メソッド ベースの [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) クエリで、[Where](/dotnet/csharp/programming-guide/concepts/linq/query-syntax-and-method-syntax-in-linq) メソッドや `Contains` など、標準クエリ演算子メソッドの引数として使用されます。 LINQ クエリは、`Where`、`Contains`、`OrderBy` などのメソッドの呼び出しで定義または変更されたときには実行されません。 クエリ実行は先送りされます。 その具体値が繰り返されるか、`ToListAsync` メソッドが呼び出されるまで、式の評価が延ばされます。 詳細については、「[クエリ実行](/dotnet/framework/data/adonet/ef/language-reference/query-execution)」を参照してください。

> [!NOTE]
> [Contains](/dotnet/api/system.data.objects.dataclasses.entitycollection-1.contains) メソッドは C# コードではなく、データベースで実行されます。 クエリの大文字と小文字の区別は、データベースや照合順序に依存します。 SQL Server では、`Contains` は大文字/小文字の区別がない [SQL LIKE](/sql/t-sql/language-elements/like-transact-sql) にマッピングされます。 SQLite では、既定の照合順序で、大文字と小文字が区別されます。

ムービー ページに移動し、`?searchString=Ghost` のようなクエリ文字列を URL に追加します。 たとえば、「 `https://localhost:5001/Movies?searchString=Ghost` 」のように入力します。 フィルターされたムービーが表示されます。

![Indexビュー](search/_static/ghost.png)

次のルート テンプレートが Index ページに追加される場合、検索文字列を URL セグメントとして渡すことができます。 たとえば、「 `https://localhost:5001/Movies/Ghost` 」のように入力します。

```cshtml
@page "{searchString?}"
```

先のルート制約では、クエリ文字列値の代わりに、ルート データ (URL セグメント) として題名を検索できます。  `"{searchString?}"` の `?` は、これが任意のルート パラメーターであることを意味します。

![ghost という単語が URL に追加されたIndexビュー。Ghostbusters と Ghostbusters 2 という 2 本のムービーからなるムービー リストが返されています。](search/_static/g2.png)

ASP.NET Core ランタイムでは[モデル バインド](xref:mvc/models/model-binding)を使用し、クエリ文字列 (`?searchString=Ghost`) またはルート データ (`https://localhost:5001/Movies/Ghost`) から `SearchString` プロパティの値が設定されます。 モデル バインドでは、大文字と小文字は区別 "***されません***"。

ただし、URL を変更してムービーを検索することをユーザーに求めることはできません。 この手順では、ムービーを絞り込むための UI を追加します。 ルート制約 `"{searchString?}"` を追加した場合、それを削除します。

*Pages/Movies/Index.cshtml* ファイルを開き、次のコードで強調表示されているマークアップを追加します。

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie30/SnapShots/Index2.cshtml?highlight=14-19&range=1-22)]

HTML `<form>` タグでは、次の[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)が使用されます。

* [フォーム タグ ヘルパー](xref:mvc/views/working-with-forms#the-form-tag-helper) フォームが提出されると、フィルター文字列がクエリ文字列経由で *ページ/ムービー/Index* ページに送信されます。
* [入力タグ ヘルパー](xref:mvc/views/working-with-forms#the-input-tag-helper)

変更を保存し、フィルターをテストします。

![タイトル フィルター テキストボックスに ghost という単語が入力されたIndexビュー](search/_static/filter.png)

## <a name="search-by-genre"></a>ジャンルで検索する

Index ページの `OnGetAsync` メソッドを次のコードで更新します。

   [!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_SearchGenre)]

次のコードは、データベースからすべてのジャンルを取得する LINQ クエリです。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_LINQ)]

ジャンルの `SelectList` は、別個のジャンルを推定することで作成されます。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_SelectList)]

### <a name="add-search-by-genre-to-the-razor-page"></a>ジャンル検索を Razor ページに追加する

1. 次のマークアップで強調表示されているように、 *Index.cshtml* [`<form>`要素](https://developer.mozilla.org/docs/Web/HTML/Element/form)を更新します。

   [!code-cshtml[](razor-pages-start/sample/RazorPagesMovie30/SnapShots/IndexFormGenreNoRating.cshtml?highlight=16-18&range=1-26)]

1. ジャンルまたはムービーのタイトル、あるいはその両方で検索して、アプリをテストします。

## <a name="additional-resources"></a>その他の技術情報

> [!div class="step-by-step"]
> [前へ:ページの更新](xref:tutorials/razor-pages/da1)
> [次へ新しいフィールドの追加](xref:tutorials/razor-pages/new-field)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

次のセクションでは、*ジャンル* または *名前* による映画検索が追加されます。

強調表示されている次のプロパティを *Pages/Movies/Index.cshtml.cs* に追加します。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_newProps&highlight=11-999)]

* `SearchString`: ユーザーが検索テキスト ボックスに入力したテキストが含まれる。 `SearchString` には、[`[BindProperty]`](xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute) 属性があります。 `[BindProperty]` により、プロパティと同じ名前に基づきフォーム値とクエリ文字列がバインドされます。 HTTP GET 要求でのバインドには `[BindProperty(SupportsGet = true)]` が必要です。
* `Genres`: ジャンル一覧が含まれる。 `Genres` により、ユーザーは一覧からジャンルを選択できます。 `SelectList` には `using Microsoft.AspNetCore.Mvc.Rendering;` が必要です。
* `MovieGenre`: ユーザーが選択する特定のジャンルが含まれる。 たとえば、"Western (西部劇)" です。
* `Genres` と `MovieGenre` は、このチュートリアルで後述します。

[!INCLUDE[](~/includes/bind-get.md)]

Index ページの `OnGetAsync` メソッドを次のコードで更新します。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_1stSearch)]

`OnGetAsync` メソッドの最初の行により、ムービーを選択する [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) クエリが作成されます。

```csharp
// using System.Linq;
var movies = from m in _context.Movie
             select m;
```

このクエリはこの時点では定義される *だけ* で、データベースに対して **実行されていません**。

`SearchString` プロパティが null でも空でもない場合、検索文字列で絞り込むようにムービークエリが変更されます。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_SearchNull)]

`s => s.Title.Contains()` コードは[ラムダ式](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions)です。 ラムダは、メソッド ベースの [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) クエリで、[Where](/dotnet/csharp/programming-guide/concepts/linq/query-syntax-and-method-syntax-in-linq) メソッドや `Contains` など、標準クエリ演算子メソッドの引数として使用されます。 LINQ クエリは、`Where`、`Contains`、`OrderBy` などのメソッドの呼び出しで定義または変更されたときには実行されません。 クエリ実行は先送りされます。 その具体値が繰り返されるか、`ToListAsync` メソッドが呼び出されるまで、式の評価が延ばされます。 詳細については、「[クエリ実行](/dotnet/framework/data/adonet/ef/language-reference/query-execution)」を参照してください。

**注:** [Contains](/dotnet/api/system.data.objects.dataclasses.entitycollection-1.contains) メソッドは C# コードではなく、データベースで実行されます。 クエリの大文字と小文字の区別は、データベースや照合順序に依存します。 SQL Server では、`Contains` は大文字/小文字の区別がない [SQL LIKE](/sql/t-sql/language-elements/like-transact-sql) にマッピングされます。 SQLite では、既定の照合順序で、大文字と小文字が区別されます。

ムービー ページに移動し、`?searchString=Ghost` のようなクエリ文字列を URL に追加します。 たとえば、「 `https://localhost:5001/Movies?searchString=Ghost` 」のように入力します。 フィルターされたムービーが表示されます。

![Indexビュー](search/_static/ghost.png)

次のルート テンプレートが Index ページに追加される場合、検索文字列を URL セグメントとして渡すことができます。 たとえば、「 `https://localhost:5001/Movies/Ghost` 」のように入力します。

```cshtml
@page "{searchString?}"
```

先のルート制約では、クエリ文字列値の代わりに、ルート データ (URL セグメント) として題名を検索できます。  `"{searchString?}"` の `?` は、これが任意のルート パラメーターであることを意味します。

![ghost という単語が URL に追加されたIndexビュー。Ghostbusters と Ghostbusters 2 という 2 本のムービーからなるムービー リストが返されています。](search/_static/g2.png)

ASP.NET Core ランタイムでは[モデル バインド](xref:mvc/models/model-binding)を使用し、クエリ文字列 (`?searchString=Ghost`) またはルート データ (`https://localhost:5001/Movies/Ghost`) から `SearchString` プロパティの値が設定されます。 モデル バインドでは、大文字と小文字は区別 "***されません***"。

ただし、URL を変更してムービーを検索することをユーザーに求めることはできません。 この手順では、ムービーを絞り込むための UI を追加します。 ルート制約 `"{searchString?}"` を追加した場合、それを削除します。

*Pages/Movies/Index.cshtml* ファイルを開き、次のコードで強調表示されている `<form>` マークアップを追加します。

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index2.cshtml?highlight=14-19&range=1-22)]

HTML `<form>` タグでは、次の[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)が使用されます。

* [フォーム タグ ヘルパー](xref:mvc/views/working-with-forms#the-form-tag-helper) フォームが提出されると、フィルター文字列がクエリ文字列経由で *ページ/ムービー/Index* ページに送信されます。
* [入力タグ ヘルパー](xref:mvc/views/working-with-forms#the-input-tag-helper)

変更を保存し、フィルターをテストします。

![タイトル フィルター テキストボックスに ghost という単語が入力されたIndexビュー](search/_static/filter.png)

## <a name="search-by-genre"></a>ジャンルで検索する

`OnGetAsync` メソッドを次のコードで更新します。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_SearchGenre)]

次のコードは、データベースからすべてのジャンルを取得する LINQ クエリです。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_LINQ)]

ジャンルの `SelectList` は、別個のジャンルを推定することで作成されます。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_SelectList)]

### <a name="add-search-by-genre-to-the-razor-page"></a>ジャンル検索を Razor ページに追加する

次のマークアップで強調表示されているように、 *Index.cshtml* [`<form>` 要素] (https://developer.mozilla.org/docs/Web/HTML/Element/form) ) を更新します。

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/IndexFormGenreNoRating.cshtml?highlight=16-18&range=1-26)]

ジャンルまたはムービーのタイトル、あるいはその両方で検索して、アプリをテストします。
前のコードでは、[選択タグ ヘルパー](xref:mvc/views/working-with-forms#the-select-tag-helper)とオプション タグ ヘルパーが使用されています。

## <a name="additional-resources"></a>その他の技術情報

* [このチュートリアルの YouTube バージョン](https://youtu.be/4B6pHtdyo08)

> [!div class="step-by-step"]
> [前へ:ページの更新](xref:tutorials/razor-pages/da1)
> [次へ新しいフィールドの追加](xref:tutorials/razor-pages/new-field)

::: moniker-end
