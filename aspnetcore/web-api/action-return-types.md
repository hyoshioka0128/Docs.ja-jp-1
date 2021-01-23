---
title: ASP.NET Core Web API のコントローラー アクションの戻り値の型
author: scottaddie
description: ASP.NET Core Web API でのさまざまなコントローラー アクション メソッドの戻り値の型の使用について説明します。
ms.author: scaddie
ms.custom: mvc
ms.date: 02/03/2020
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
uid: web-api/action-return-types
ms.openlocfilehash: 62c8227ca770a3a9adbe780685b140bc0e86841e
ms.sourcegitcommit: da5a5bed5718a9f8db59356ef8890b4b60ced6e9
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/22/2021
ms.locfileid: "98710699"
---
# <a name="controller-action-return-types-in-aspnet-core-web-api"></a>ASP.NET Core Web API のコントローラー アクションの戻り値の型

作成者: [Scott Addie](https://github.com/scottaddie)

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/action-return-types/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

ASP.NET Core では、Web API コントローラー アクションの戻り値の型に次のオプションを提供しています。

::: moniker range=">= aspnetcore-2.1"

* [特定の型](#specific-type)
* [IActionResult](#iactionresult-type)
* [ActionResult\<T>](#actionresultt-type)

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

* [特定の型](#specific-type)
* [IActionResult](#iactionresult-type)

::: moniker-end

このドキュメントでは、各戻り値の型を使用するのが最適な場合について説明します。

## <a name="specific-type"></a>特定の型

最もシンプルなアクションでは、プリミティブ データ型または複合データ型が返されます (`string` やカスタム オブジェクトの型など)。 カスタム `Product` オブジェクトのコレクションを返す次のアクションを考えてみましょう。

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_Get)]

アクションの実行中に保護できる既知の条件がない場合は、特定の型を返すだけで十分です。 前のアクションでパラメーターを受け取っていないので、パラメーターの制約の検証は必要ありません。

複数の戻り値の型が可能な場合は、戻り値の型 <xref:Microsoft.AspNetCore.Mvc.ActionResult> とプリミティブまたは複合の戻り値の型を混在させることが一般的です。 この種類のアクションに対応するには、 [Iactionresult](#iactionresult-type)または[ \<T> actionresult](#actionresultt-type)のいずれかが必要です。 このドキュメントでは、複数の戻り値の型のサンプルをいくつか紹介します。

### <a name="return-ienumerablet-or-iasyncenumerablet"></a>IEnumerable \<T> または IAsyncEnumerable を返します\<T>

ASP.NET Core 2.2 以前では、アクションから <xref:System.Collections.Generic.IEnumerable%601> が返されると、シリアライザーによって同期コレクションのイテレーションが行われます。 その結果、呼び出しがブロックされ、スレッド プールが枯渇する可能性があります。 例として、Web API のデータ アクセスのニーズに Entity Framework (EF) コアが使用されているとします。 次のアクションの戻り値の型は、シリアル化中に同期的に列挙されます。

```csharp
public IEnumerable<Product> GetOnSaleProducts() =>
    _context.Products.Where(p => p.IsOnSale);
```

ASP.NET Core 2.2 以前のデータベースで同期的な列挙とブロックの待機を回避するには、`ToListAsync` を呼び出します。

```csharp
public async Task<IEnumerable<Product>> GetOnSaleProducts() =>
    await _context.Products.Where(p => p.IsOnSale).ToListAsync();
```

ASP.NET Core 3.0 以降では、アクションから `IAsyncEnumerable<T>` を返します。

* 同期のイテレーションは行われなくなります。
* <xref:System.Collections.Generic.IEnumerable%601> を返す場合と同様に効率的になります。

ASP.NET Core 3.0 以降では、次のアクションの結果がバッファーに出力されてから、シリアライザーに渡されます。

```csharp
public IEnumerable<Product> GetOnSaleProducts() =>
    _context.Products.Where(p => p.IsOnSale);
```

非同期のイテレーションを保証するために、アクション シグネチャの戻り値の型を `IAsyncEnumerable<T>` と宣言することを検討してください。 最終的に、イテレーション モードは、返される基となる具象型に基づいています。 MVC では、`IAsyncEnumerable<T>` を実装するすべての具象型が自動的にバッファーされます。

販売価格が付けられた製品レコードが `IEnumerable<Product>` として返される次のようなアクションを考えてみます。

[!code-csharp[](../web-api/action-return-types/samples/3x/WebApiSample.Api.31/Controllers/ProductsController.cs?name=snippet_GetOnSaleProducts)]

前の `IAsyncEnumerable<Product>` のアクションに相当するものは次のとおりです。

[!code-csharp[](../web-api/action-return-types/samples/3x/WebApiSample.Api.31/Controllers/ProductsController.cs?name=snippet_GetOnSaleProductsAsync)]

上記のアクションはいずれも ASP.NET Core 3.0 以降ではブロックされません。

## <a name="iactionresult-type"></a>IActionResult 型

戻り値の型 <xref:Microsoft.AspNetCore.Mvc.IActionResult> は、アクションの戻り値の型 `ActionResult` が複数考えられる場合に適しています。 `ActionResult` 型は、さまざまな HTTP 状態コードを表します。 `ActionResult` から派生したすべての非抽象クラスは、有効な戻り値の型として修飾されます。 このカテゴリの一般的な戻り値の型として、<xref:Microsoft.AspNetCore.Mvc.BadRequestResult> (400)、<xref:Microsoft.AspNetCore.Mvc.NotFoundResult> (404)、<xref:Microsoft.AspNetCore.Mvc.OkObjectResult> (200) などがあります。 また、<xref:Microsoft.AspNetCore.Mvc.ControllerBase> クラスの便利なメソッドを使用して、アクションから `ActionResult` 型を返すこともできます。 たとえば、`return BadRequest();` は、`return new BadRequestResult();` の短縮形です。

この種類のアクションには複数の戻り値の型とパスがあるため、属性を自由に使用する [`[ProducesResponseType]`](xref:Microsoft.AspNetCore.Mvc.ProducesResponseTypeAttribute) 必要があります。 この属性は、[Swagger](xref:tutorials/web-api-help-pages-using-swagger) などのツールで生成される Web API ヘルプ ページのよりわかりやすい応答の詳細を生成します。 `[ProducesResponseType]` は、アクションによって返される既知の型と HTTP 状態コードを示します。

### <a name="synchronous-action"></a>同期アクション

2 つの戻り値の型が考えられる、次の同期アクションを考えてみましょう。

::: moniker range=">= aspnetcore-2.1"

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_GetByIdIActionResult&highlight=8,11)]

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.Pre21/Controllers/ProductsController.cs?name=snippet_GetById&highlight=8,11)]

::: moniker-end

前のアクションの内容:

* `id` によって表される製品が、基になるデータ ストアに存在しないと、404 状態コードが返されます。 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*> の便利なメソッドは、`return new NotFoundResult();` の短縮形として呼び出されます。
* 製品が存在する場合は、`Product` オブジェクトと共に 200 状態コードが返されます。 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Ok*> の便利なメソッドは、`return new OkObjectResult(product);` の短縮形として呼び出されます。

### <a name="asynchronous-action"></a>非同期アクション

2 つの戻り値の型が考えられる、次の非同期アクションを考えてみましょう。

::: moniker range=">= aspnetcore-2.1"

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_CreateAsyncIActionResult&highlight=9,14)]

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.Pre21/Controllers/ProductsController.cs?name=snippet_CreateAsync&highlight=9,14)]

::: moniker-end

前のアクションの内容:

* 製品の説明に "XYZ Widget" が含まれている場合は、400 状態コードが返されます。 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*> の便利なメソッドは、`return new BadRequestResult();` の短縮形として呼び出されます。
* 201 状態コードは、製品の作成時に <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*> の便利なメソッドによって生成されます。 `CreatedAtAction` を呼び出す代替の方法として `return new CreatedAtActionResult(nameof(GetById), "Products", new { id = product.Id }, product);` があります。 このコード パスでは、`Product` オブジェクトは応答本文で指定されます。 新しく作成された製品の URL を含む `Location` 応答ヘッダーが用意されています。

たとえば、次のモデルでは、要求に `Name` プロパティと `Description` プロパティを含める必要があることを示しています。 要求で `Name` と `Description` を提供するのに失敗すると、モデルの検証が失敗します。

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.DataAccess/Models/Product.cs?name=snippet_ProductClass&highlight=5-6,8-9)]

::: moniker range=">= aspnetcore-2.1"

[`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute)ASP.NET Core 2.1 以降の属性が適用されている場合、モデル検証エラーは400状態コードになります。 詳細については、「[自動的な HTTP 400 応答](xref:web-api/index#automatic-http-400-responses)」を参照してください。

## <a name="actionresultt-type"></a>ActionResult \<T> 型

ASP.NET Core 2.1 では、web API コントローラーアクションの[Actionresult \<T> ](xref:Microsoft.AspNetCore.Mvc.ActionResult`1)戻り値の型が導入されました。 これで、<xref:Microsoft.AspNetCore.Mvc.ActionResult> から派生した型を返したり、[特定の型](#specific-type)を返したりすることができるようになりました。 `ActionResult<T>` により、[IActionResult 型](#iactionresult-type)は次の利点を得られます。

* [`[ProducesResponseType]`](xref:Microsoft.AspNetCore.Mvc.ProducesResponseTypeAttribute)属性の `Type` プロパティは除外できます。 たとえば、`[ProducesResponseType(200, Type = typeof(Product))]` は `[ProducesResponseType(200)]` に簡略化されます。 アクションの予期される戻り値の型は、代わりに `ActionResult<T>` の `T` から推論されます。
* [暗黙的なキャスト演算子](/dotnet/csharp/language-reference/keywords/implicit)は、`T` と `ActionResult` の両方の `ActionResult<T>` への変換をサポートしています。 `T` は <xref:Microsoft.AspNetCore.Mvc.ObjectResult> に変換されます。つまり、`return new ObjectResult(T);` は `return T;` に簡略化されます。

C# はインターフェイス上での暗黙的なキャスト演算子をサポートしていません。 そのため、`ActionResult<T>` を使用するには、インターフェイスを具象型に変換する必要があります。 たとえば、次の例における `IEnumerable` の使用は機能しません。

```csharp
[HttpGet]
public ActionResult<IEnumerable<Product>> Get() =>
    _repository.GetProducts();
```

上記のコードを修正するための選択肢の 1 つは、`_repository.GetProducts().ToList();` を返すことです。

ほとんどのアクションには、特定の戻り値の型があります。 アクションの実行中に予期しない状態が発生する場合、特定の型は返されません。 たとえば、アクションの入力パラメーターがモデルの検証に失敗する場合があります。 このような場合、通常は特定の型ではなく、適切な `ActionResult` 型が返されます。

### <a name="synchronous-action"></a>同期アクション

2 つの戻り値の型が考えられる、同期アクションを考えてみましょう。

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_GetByIdActionResult&highlight=8,11)]

前のアクションの内容:

* 製品がデータベース内に存在しない場合、404 状態コードが返されます。
* 製品が存在する場合、対応する `Product` オブジェクトと共に 200 状態コードが返されます。 ASP.NET Core 2.1 より前のバージョンは、`return product;` 行を `return Ok(product);` にする必要がありました。

### <a name="asynchronous-action"></a>非同期アクション

2 つの戻り値の型が考えられる、非同期アクションを考えてみましょう。

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_CreateAsyncActionResult&highlight=9,14)]

前のアクションの内容:

* 次の場合、ASP.NET Core ランタイムによって 400 状態コード (<xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*>) が返されます。
  * [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute)属性が適用され、モデルの検証が失敗します。
  * 製品の説明に "XYZ Widget" が含まれている。
* 201 状態コードは、製品の作成時に <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*> メソッドによって生成されます。 このコード パスでは、`Product` オブジェクトは応答本文で指定されます。 新しく作成された製品の URL を含む `Location` 応答ヘッダーが用意されています。

::: moniker-end

## <a name="additional-resources"></a>その他のリソース

* <xref:mvc/controllers/actions>
* <xref:mvc/models/validation>
* <xref:tutorials/web-api-help-pages-using-swagger>
