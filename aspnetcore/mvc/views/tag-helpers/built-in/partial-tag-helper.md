---
title: ASP.NET Core の部分タグ ヘルパー
author: rick-anderson
description: ASP.NET Core 部分タグ ヘルパーと、その各属性が部分ビューのレンダリングにおいて果たす役割について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: mvc
ms.date: 04/06/2019
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
uid: mvc/views/tag-helpers/builtin-th/partial-tag-helper
ms.openlocfilehash: 4d1c2e2759168b4b1cbccd4ca430fe52dcd028b0
ms.sourcegitcommit: 0abfe496fed8e9470037c8128efa8a50069ccd52
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/07/2021
ms.locfileid: "106564207"
---
# <a name="partial-tag-helper-in-aspnet-core"></a>ASP.NET Core の部分タグ ヘルパー

作成者: [Scott Addie](https://github.com/scottaddie)

タグ ヘルパーの概要については、「<xref:mvc/views/tag-helpers/intro>」をご覧ください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/tag-helpers/built-in/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="overview"></a>概要

部分タグヘルパーは、ページおよび MVC アプリで [部分ビュー](xref:mvc/views/partial) を表示するために使用され Razor ます。 考慮事項:

* ASP.NET Core 2.1 以降を必要とします。
* [HTML ヘルパー構文](xref:mvc/views/partial#reference-a-partial-view)の代替です。
* 部分ビューを非同期でレンダリングします。

部分ビューをレンダリングするための HTML ヘルパー オプション:

* [`@await Html.PartialAsync`](/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.partialasync)
* [`@await Html.RenderPartialAsync`](/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.renderpartialasync)
* [`@Html.Partial`](/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.partial)
* [`@Html.RenderPartial`](/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.renderpartial)

*Product* モデルがこのドキュメント全体でサンプルとして使用されます。

[!code-csharp[](samples/TagHelpersBuiltIn/Models/Product.cs)]

部分タグ ヘルパー属性のインベントリが続きます。

## <a name="name"></a>name

`name` 属性は必須です。 レンダリングする部分ビューの名前またはパスを示します。 部分ビュー名が指定されると、[ビューの検出](xref:mvc/views/overview#view-discovery)プロセスが開始します。 明示的なパスが指定されているとき、このプロセスは回避されます。 許容されるすべての `name` 値については、「[部分ビューの検出](xref:mvc/views/partial#partial-view-discovery)」を参照してください。

次のマークアップでは明示的なパスが使用されており、*_ProductPartial.cshtml* が *Shared* フォルダーから読み込まれることを示しています。 [for](#for) 属性を使用し、バインディングのために部分ビューにモデルが渡されます。

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Product.cshtml?name=snippet_Name)]

## <a name="for"></a>対象

`for` 属性によって、現在のモデルに対して評価する [ModelExpression](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.modelexpression) が割り当てられます。 `ModelExpression` によって `@Model.` 構文が推論されます。 たとえば、`for="Product"` の代わりに `for="@Model.Product"` を使用できます。 この既定の推論動作は、`@` シンボルを使用してインライン式を定義することでオーバーライドされます。

次のマークアップでは、*_ProductPartial.cshtml* が読み込まれます。

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Product.cshtml?name=snippet_For)]

部分ビューは、関連ページ モデルの `Product` プロパティにバインドされています。

[!code-csharp[](samples/TagHelpersBuiltIn/Pages/Product.cshtml.cs?highlight=8)]

## <a name="model"></a>model

`model` 属性によって、部分ビューに渡すモデル インスタンスが割り当てられます。 `model` 属性は [for](#for) 属性と共に使用できません。

次のマークアップでは、新しい `Product` オブジェクトがインスタンス化され、バインディングのために `model` 属性に渡されます。

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Product.cshtml?name=snippet_Model)]

## <a name="view-data"></a>view-data

`view-data` 属性によって、部分ビューに渡す [ViewDataDictionary](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.viewdatadictionary) が割り当てられます。 次のマークアップでは、ViewData コレクション全体が部分ビューで使えるようになります。

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Product.cshtml?name=snippet_ViewData&highlight=5-)]

前述のコードでは、`IsNumberReadOnly` キーの値が `true` に設定されており、ViewData コレクションに追加されます。 その結果、次の部分ビュー内で `ViewData["IsNumberReadOnly"]` を利用できます。

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Shared/_ProductViewDataPartial.cshtml?highlight=5)]

この例では、`ViewData["IsNumberReadOnly"]` の値によって、*Number* フィールドを読み取り専用として表示するかどうかが決定されます。

## <a name="migrate-from-an-html-helper"></a>HTML ヘルパーから移行する

次のような非同期の HTML ヘルパーの例を考えてみてください。 製品のコレクションが反復処理され、表示されます。 `PartialAsync` メソッドの最初のパラメーターごとに、*_ProductPartial.cshtml* の部分ビューが読み込まれます。 `Product` モデルのインスタンスが、バインディングのために部分ビューに渡されます。

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Products.cshtml?name=snippet_HtmlHelper&highlight=3)]

次の部分タグ ヘルパーは、`PartialAsync` HTML ヘルパーと同じ非同期レンダリング動作を実現します。 部分ビューのバインディングのため、`model` 属性に `Product` モデルのインスタンスが割り当てられます。

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Products.cshtml?name=snippet_TagHelper&highlight=3)]

## <a name="additional-resources"></a>その他の技術情報

* <xref:mvc/views/partial>
* <xref:mvc/views/overview#weakly-typed-data-viewdata-viewdata-attribute-and-viewbag>
