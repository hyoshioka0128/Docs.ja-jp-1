---
title: ASP.NET Core を使って Web API を作成する
author: scottaddie
description: ASP.NET Core での Web API の作成の基本について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: mvc
ms.date: 07/20/2020
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
uid: web-api/index
ms.openlocfilehash: fb7dfa53be1fee9904b4539a5c9da0700c2aa884
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102585735"
---
# <a name="create-web-apis-with-aspnet-core"></a>ASP.NET Core を使って Web API を作成する

作成者: [Scott Addie](https://github.com/scottaddie)、[Tom Dykstra](https://github.com/tdykstra)

ASP.NET Core では、C# を使った RESTful サービス (別名: Web API) の作成がサポートされています。 要求を処理するために、Web API ではコントローラーを使用します。 Web API の "*コントローラー*" は `ControllerBase` から派生するクラスです。 この記事では、コントローラーを使って Web API 要求を処理する方法について説明します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/index/samples)します。 ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="controllerbase-class"></a>ControllerBase クラス

Web API は、<xref:Microsoft.AspNetCore.Mvc.ControllerBase> から派生した 1 つ以上のコントローラー クラスで構成されます。 Web API のプロジェクト テンプレートでは、スターター コントローラーが提供されます。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_ControllerSignature&highlight=3)]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Controllers/ValuesController.cs?name=snippet_ControllerSignature&highlight=3)]

::: moniker-end

<xref:Microsoft.AspNetCore.Mvc.Controller> クラスから派生させて Web API のコントローラーを作成しないでください。 `ControllerBase` から派生した `Controller` にはビューのサポートが追加されるため、これは Web API 要求ではなく Web ページを処理するためのものです。 このルールには例外があります。ビューと Web API の両方で同じコントローラーを使うことを計画している場合は、`Controller` から派生させます。

`ControllerBase` クラスには、HTTP 要求の処理に役立つプロパティとメソッドが多数用意されています。 たとえば、`ControllerBase.CreatedAtAction` では状態コード 201 が返されます。

[!code-csharp[](index/samples/2.x/2.2/Controllers/PetsController.cs?name=snippet_400And201&highlight=10)]

次に、`ControllerBase` に用意されているメソッドの例をさらにいくつか示します。

|メソッド   |メモ    |
|---------|---------|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest%2A>| 400 状態コードを返します。|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound%2A>|404 状態コードを返します。|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.PhysicalFile%2A>|ファイルを返します。|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.TryUpdateModelAsync%2A>|[モデル バインド](xref:mvc/models/model-binding)を呼び出します。|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.TryValidateModel%2A>|[モデル検証](xref:mvc/models/validation)を呼び出します。|

使用可能なすべてのメソッドとプロパティの一覧については、「<xref:Microsoft.AspNetCore.Mvc.ControllerBase>」を参照してください。

## <a name="attributes"></a>属性

<xref:Microsoft.AspNetCore.Mvc> 名前空間には、Web API のコントローラーとアクション メソッドの動作の構成に使用できる属性が用意されています。 次の例では、属性を使って、サポート対象の HTTP アクション動詞と、返却される可能性がある既知の HTTP ステータス コードを指定します。

[!code-csharp[](index/samples/2.x/2.2/Controllers/PetsController.cs?name=snippet_400And201&highlight=1-3)]

次に、使用できる属性の例をさらにいくつか示します。

|属性|メモ|
|---------|-----|
|[`[Route]`](<xref:Microsoft.AspNetCore.Mvc.RouteAttribute>)      |コントローラーまたはアクションの URL パターンを指定します。|
|[`[Bind]`](<xref:Microsoft.AspNetCore.Mvc.BindAttribute>)        |モデル バインドのために含めるプレフィックスとプロパティを指定します。|
|[`[HttpGet]`](<xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute>)  |HTTP GET アクション動詞をサポートするアクションを特定します。|
|[`[Consumes]`](<xref:Microsoft.AspNetCore.Mvc.ConsumesAttribute>)|アクションが受け取るデータ型を指定します。|
|[`[Produces]`](<xref:Microsoft.AspNetCore.Mvc.ProducesAttribute>)|アクションによって返すデータ型を指定します。|

使用可能な属性を含む一覧については、<xref:Microsoft.AspNetCore.Mvc> 名前空間をご覧ください。

## <a name="apicontroller-attribute"></a>ApiController 属性

[`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) 属性をコントローラー クラスに適用して、次の厳格な、API に固有の動作を有効にできます。

::: moniker range=">= aspnetcore-2.2"

* [属性ルーティング要件](#attribute-routing-requirement)
* [自動的な HTTP 400 応答](#automatic-http-400-responses)
* [バインディング ソース パラメーター推論](#binding-source-parameter-inference)
* [マルチパート/フォーム データ要求の推論](#multipartform-data-request-inference)
* [エラー状態コードに関する問題の詳細](#problem-details-for-error-status-codes)

"*エラー状態コードに関する問題の詳細*" 機能には、[互換性バージョン](xref:mvc/compatibility-version) 2.2 以降が必要です。 その他の機能では、互換性バージョン 2.1 以降が必要です。

::: moniker-end

* [属性ルーティング要件](#attribute-routing-requirement)
* [自動的な HTTP 400 応答](#automatic-http-400-responses)
* [バインディング ソース パラメーター推論](#binding-source-parameter-inference)
* [マルチパート/フォーム データ要求の推論](#multipartform-data-request-inference)

これらの機能では[互換性バージョン](xref:mvc/compatibility-version) 2.1 以降が必要です。

### <a name="attribute-on-specific-controllers"></a>特定のコントローラーにおける属性

プロジェクト テンプレートからの次の例のように、`[ApiController]` 属性は特定のコントローラーに適用できます。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_ControllerSignature&highlight=2])]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Controllers/ValuesController.cs?name=snippet_ControllerSignature&highlight=2)]

::: moniker-end

### <a name="attribute-on-multiple-controllers"></a>複数のコントローラーにおける属性

複数のコントローラーでこの属性を使う方法の 1 つは、`[ApiController]` 属性で注釈を付けたカスタム基本コントローラー クラスを作成することです。 次は、カスタム基底クラスとそこから派生したコントローラーを示す例です。

[!code-csharp[](index/samples/2.x/2.2/Controllers/MyControllerBase.cs?name=snippet_MyControllerBase)]

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Controllers/PetsController.cs?name=snippet_Inherit)]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Controllers/PetsController.cs?name=snippet_Inherit)]

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

### <a name="attribute-on-an-assembly"></a>アセンブリにおける属性

[互換性バージョン](xref:mvc/compatibility-version)が 2.2 以降に設定されている場合は、`[ApiController]` 属性をアセンブリに適用できます。 この方法での注釈では、アセンブリ内のすべてのコントローラーに Web API の動作が適用されます。 個別のコントローラーを除外する方法はありません。 アセンブリ レベルの属性を `Startup` クラスを囲む名前空間宣言に適用します。

```csharp
[assembly: ApiController]
namespace WebApiSample
{
    public class Startup
    {
        ...
    }
}
```

::: moniker-end

## <a name="attribute-routing-requirement"></a>属性ルーティング要件

`[ApiController]` 属性では、属性ルーティング要件が作成されます。 次に例を示します。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_ControllerSignature&highlight=2)]

`Startup.Configure` の `UseEndpoints`、<xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc%2A>、または <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvcWithDefaultRoute%2A> で定義された[規則ルート経由](xref:mvc/controllers/routing#conventional-routing)でアクションにアクセスすることはできません。

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Controllers/ValuesController.cs?name=snippet_ControllerSignature&highlight=1)]

`Startup.Configure` の <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc%2A> または <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvcWithDefaultRoute%2A> で定義された[規則ルート](xref:mvc/controllers/routing#conventional-routing)経由でアクションにアクセスすることはできません。

::: moniker-end

## <a name="automatic-http-400-responses"></a>自動的な HTTP 400 応答

`[ApiController]` 属性により、モデル検証エラーが発生すると HTTP 400 応答が自動的にトリガーされます。 その結果、アクション メソッド内の次のコードは不要になります。

```csharp
if (!ModelState.IsValid)
{
    return BadRequest(ModelState);
}
```

ASP.NET Core MVC では、<xref:Microsoft.AspNetCore.Mvc.Infrastructure.ModelStateInvalidFilter> アクション フィルターの使用により前のチェックが行われます。

### <a name="default-badrequest-response"></a>既定の BadRequest 応答

2\.1 の互換性バージョンを使う場合、HTTP 400 応答に対する既定の応答の種類は <xref:Microsoft.AspNetCore.Mvc.SerializableError> です。 次の要求本文は、シリアル化された型の例です。

```json
{
  "": [
    "A non-empty request body is required."
  ]
}
```

::: moniker range=">= aspnetcore-2.2"

2\.2 以降の互換性バージョンを使う場合、HTTP 400 応答に対する既定の応答の種類は <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> です。 次の要求本文は、シリアル化された型の例です。

```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "traceId": "|7fb5e16a-4c8f23bbfc974667.",
  "errors": {
    "": [
      "A non-empty request body is required."
    ]
  }
}
```

`ValidationProblemDetails` 型:

* Web API 応答でエラーを指定するための、コンピューターが判読できる形式を提供します。
* [RFC 7807 仕様](https://tools.ietf.org/html/rfc7807)に準拠しています。

::: moniker-end

自動とカスタムの応答を一貫させるには、<xref:System.Web.Http.ApiController.BadRequest%2A> ではなく <xref:Microsoft.AspNetCore.Mvc.ControllerBase.ValidationProblem%2A> メソッドを呼び出します。 `ValidationProblem` は <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> オブジェクトと、自動的な応答を返します。

### <a name="log-automatic-400-responses"></a>自動的な 400 応答を記録する

「[How to log automatic 400 responses on model validation error (dotnet/AspNetCore.Docs #12157)](https://github.com/dotnet/AspNetCore.Docs/issues/12157)」(モデル検証エラー時に自動的な 400 応答を記録する方法) を参照してください。

### <a name="disable-automatic-400-response"></a>自動的な 400 応答を無効にする

自動的な 400 の動作を無効にするには、<xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressModelStateInvalidFilter> プロパティを `true` に設定します。 `Startup.ConfigureServices` で次の強調表示されたコードを追加します。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=2,6)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=3,7)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](index/samples/2.x/2.1/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=1,5)]

::: moniker-end

## <a name="binding-source-parameter-inference"></a>バインディング ソース パラメーター推論

バインディング ソース属性では、アクション パラメーターの値が存在する場所が定義されます。 次のバインディング ソース属性が存在します。

|属性|バインド ソース |
|---------|---------|
|[`[FromBody]`](xref:Microsoft.AspNetCore.Mvc.FromBodyAttribute)     | 要求本文 |
|[`[FromForm]`](xref:Microsoft.AspNetCore.Mvc.FromFormAttribute)     | 要求本文内のフォーム データ |
|[`[FromHeader]`](xref:Microsoft.AspNetCore.Mvc.FromHeaderAttribute) | 要求ヘッダー |
|[`[FromQuery]`](xref:Microsoft.AspNetCore.Mvc.FromQueryAttribute)   | 要求のクエリ文字列パラメーター |
|[`[FromRoute]`](xref:Microsoft.AspNetCore.Mvc.FromRouteAttribute)   | 現在の要求からのルート データ |
|[`[FromServices]`](xref:mvc/controllers/dependency-injection#action-injection-with-fromservices) | アクション パラメーターとして挿入される要求サービス |

> [!WARNING]
> 値に `%2f` (つまり、`/`) が含まれる可能性がある場合は、`[FromRoute]` を使用しないでください。 `%2f` は `/` にエスケープ解除されません。 値に `%2f` が含まれる可能性がある場合は、`[FromQuery]` を使用してください。

`[ApiController]` 属性や `[FromQuery]` などのバインディング ソース属性がない場合は、ASP.NET Core ランタイムにより複合オブジェクト モデル バインダーの使用が試行されます。 複合オブジェクト モデル バインダーでは、値プロバイダーから定義された順序でデータを取得します。

次の例では、`discontinuedOnly` パラメーター値が要求 URL のクエリ文字列に指定されていることが `[FromQuery]` によって示されています。

[!code-csharp[](index/samples/2.x/2.2/Controllers/ProductsController.cs?name=snippet_BindingSourceAttributes&highlight=3)]

`[ApiController]` 属性により、アクション パラメーターの既定のデータ ソースに対する推論規則が適用されます。 これらの規則により、アクション パラメーターに属性を適用することで手動でバインディング ソースを特定する必要がなくなります。 バインディング ソースの推論規則は、次のように動作します。

* `[FromBody]` は複合型パラメーターに対して推論されます。 `[FromBody]` 推論規則に対する例外は、<xref:Microsoft.AspNetCore.Http.IFormCollection> や <xref:System.Threading.CancellationToken> など、特殊な意味を持つ組み込みの複合型です。 バインディング ソース推論コードでは、そのような特殊な型は無視されます。
* `[FromForm]` は <xref:Microsoft.AspNetCore.Http.IFormFile> および <xref:Microsoft.AspNetCore.Http.IFormFileCollection> 型のアクション パラメーターに対して推論されます。 簡易型またはユーザー定義型に対しては推論されません。
* `[FromRoute]` は、ルート テンプレート内のパラメーターと一致する任意のアクション パラメーター名に対して推論されます。 複数のルートがアクション パラメーターと一致する場合、ルート値はいずれも `[FromRoute]` と見なされます。
* `[FromQuery]` は他の任意のアクション パラメーターに対して推論されます。

### <a name="frombody-inference-notes"></a>FromBody 推論に関するメモ

`[FromBody]` は、`string` や `int` などの単純型に対しては推論されません。 そのため、その機能が必要な場合、単純型に対しては `[FromBody]` 属性を使用する必要があります。

要求本文からバインドされるパラメーターがアクションに複数ある場合は、例外がスローされます。 たとえば、次のアクション メソッドのシグネチャはすべて例外の原因となります。

* 複合型であるため、両方に対して `[FromBody]` が推論されます。

  ```csharp
  [HttpPost]
  public IActionResult Action1(Product product, Order order)
  ```

* 1 つには `[FromBody]` 属性が使われ、もう 1 つは複合型なので推論されます。

  ```csharp
  [HttpPost]
  public IActionResult Action2(Product product, [FromBody] Order order)
  ```

* 両方で `[FromBody]` 属性が使われます。

  ```csharp
  [HttpPost]
  public IActionResult Action3([FromBody] Product product, [FromBody] Order order)
  ```

::: moniker range="= aspnetcore-2.1"

> [!NOTE]
> ASP.NET Core 2.1 では、リストや配列などのコレクション型パラメーターが誤って `[FromQuery]` と推論されます。 これらのパラメーターを要求本文からバインドする場合は、`[FromBody]` 属性を使用する必要があります。 この動作は ASP.NET Core 2.2 以降で修正されており、既定でコレクション型パラメーターが本文からバインドされることが推論されます。

::: moniker-end

### <a name="disable-inference-rules"></a>推論規則を無効にする

バインディング ソースの推論を無効にするには、<xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressInferBindingSourcesForParameters> を `true` に設定します。 `Startup.ConfigureServices` に次のコードを追加します。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=2,5)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=3,6)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](index/samples/2.x/2.1/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=1,4)]

::: moniker-end

## <a name="multipartform-data-request-inference"></a>マルチパート/フォーム データ要求の推論

[`[FromForm]`](xref:Microsoft.AspNetCore.Mvc.FromFormAttribute) 属性を使用してアクション パラメーターに注釈を付けた場合、`[ApiController]` 属性が推論規則に適用されます。 `multipart/form-data` 要求コンテンツ型が推論されます。

既定の動作を無効にするには、`Startup.ConfigureServices` で <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressConsumesConstraintForFormFileParameters> プロパティを `true` に設定します。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=2,4)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=3,5)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](index/samples/2.x/2.1/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=1,3)]

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

## <a name="problem-details-for-error-status-codes"></a>エラー状態コードに関する問題の詳細

互換性バージョンが 2.2 以上である場合、MVC によってエラー結果 (状態コードが 400 以上の結果) が <xref:Microsoft.AspNetCore.Mvc.ProblemDetails> を含む結果に変換されます。 `ProblemDetails` 型は、HTTP 応答でコンピューターが判読できるエラーの詳細を提供するための [RFC 7807 仕様](https://tools.ietf.org/html/rfc7807)に基づきます。

コントローラー アクションで次のコードがあるとします。

[!code-csharp[](index/samples/2.x/2.2/Controllers/PetsController.cs?name=snippet_ProblemDetailsStatusCode)]

`NotFound` メソッドにより、`ProblemDetails` 本文を使用して HTTP 404 ステータス コードが生成されます。 次に例を示します。

```json
{
  type: "https://tools.ietf.org/html/rfc7231#section-6.5.4",
  title: "Not Found",
  status: 404,
  traceId: "0HLHLV31KRN83:00000001"
}
```

### <a name="disable-problemdetails-response"></a>ProblemDetails 応答を無効にする

<xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressMapClientErrors%2A> プロパティが `true` に設定されている場合、エラー状態コード用の `ProblemDetails` の自動作成は無効になります。 `Startup.ConfigureServices` に次のコードを追加します。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=2,7)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=3,8)]

::: moniker-end

<a name="consumes"></a>

## <a name="define-supported-request-content-types-with-the-consumes-attribute"></a>[Consumes] 属性を使ってサポートされる要求のコンテンツの種類を定義する

既定では、アクションでは使用可能な要求のコンテンツの種類がすべてサポートされます。 たとえば、アプリが JSON と XML の[入力フォーマッタ](xref:mvc/models/model-binding#input-formatters)を両方サポートするように構成されている場合、アクションでは、`application/json` と `application/xml` を含む複数のコンテンツの種類がサポートされます。

[[Consumes]](<xref:Microsoft.AspNetCore.Mvc.ConsumesAttribute>) 属性を使用すると、アクションでサポートされる要求のコンテンツの種類を制限できます。 1 つ以上のコンテンツの種類を指定して、アクションまたはコントローラーに `[Consumes]` 属性を適用します。

```csharp
[HttpPost]
[Consumes("application/xml")]
public IActionResult CreateProduct(Product product)
```

上記のコードでは、`CreateProduct` アクションでコンテンツの種類 `application/xml` が指定されています。 このアクションにルーティングされる要求では、`Content-Type` ヘッダーに `application/xml` を指定する必要があります。 `Content-Type` ヘッダーに `application/xml` を指定していない要求では、[415 Unsupported Media Type](https://developer.mozilla.org/docs/Web/HTTP/Status/415) 応答が発生します。

また、`[Consumes]` 属性を使用すると、種類の制約を適用することによって、受信要求のコンテンツの種類に基づいてアクションの選択に影響を与えることができます。 次に例を示します。

[!code-csharp[](index/samples/3.x/Controllers/ConsumesController.cs?name=snippet_Class)]

上記のコードでは、`ConsumesController` は `https://localhost:5001/api/Consumes` の URL に送信された要求を処理するように構成されています。 コントローラーの両方のアクション `PostJson` と `PostForm` では、同じ URL の POST 要求が処理されます。 種類の制約を適用する `[Consumes]` 属性がなかった場合は、あいまい一致の例外がスローされます。

`[Consumes]` 属性は両方のアクションに適用されます。 `PostJson` アクションでは、`Content-Type` ヘッダーに `application/json` を指定して送信された要求が処理されます。 `PostForm` アクションでは、`Content-Type` ヘッダーに `application/x-www-form-urlencoded` を指定して送信された要求が処理されます。 

## <a name="additional-resources"></a>その他の技術情報

* <xref:web-api/action-return-types>
* <xref:web-api/handle-errors>
* <xref:web-api/advanced/custom-formatters>
* <xref:web-api/advanced/formatting>
* <xref:tutorials/web-api-help-pages-using-swagger>
* <xref:mvc/controllers/routing>
* [Microsoft Learn:ASP.NET Core で Web API を作成する](/learn/modules/build-web-api-aspnet-core/)
