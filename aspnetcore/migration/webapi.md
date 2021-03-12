---
title: ASP.NET Web API から ASP.NET Core への移行
author: ardalis
description: Web API の実装を ASP.NET 4.x Web API から ASP.NET Core MVC に移行する方法について説明します。
ms.author: scaddie
ms.custom: mvc
ms.date: 05/26/2020
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
uid: migration/webapi
ms.openlocfilehash: 3bd34f677271ddfc00e6e65ddf3a5e3c10eec863
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588784"
---
# <a name="migrate-from-aspnet-web-api-to-aspnet-core"></a>ASP.NET Web API から ASP.NET Core への移行

[Scott Addie](https://twitter.com/scott_addie)と[上田 Smith](https://ardalis.com/)

ASP.NET 4.x Web API は、ブラウザーやモバイルデバイスを含む広範なクライアントに到達する HTTP サービスです。 ASP.NET Core は、ASP.NET 4.x の MVC と Web API アプリのモデルを、ASP.NET Core MVC と呼ばれる単一のプログラミングモデルに統合します。 この記事では、ASP.NET 4.x Web API から ASP.NET Core MVC に移行するために必要な手順について説明します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/migration/webapi/sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

::: moniker range=">= aspnetcore-3.0"

## <a name="prerequisites"></a>必須コンポーネント

[!INCLUDE [prerequisites](../includes/net-core-prereqs-vs-3.1.md)]

## <a name="review-aspnet-4x-web-api-project"></a>ASP.NET 4.x Web API プロジェクトを確認する

この記事では、 [ASP.NET Web API 2 ではじめに](/aspnet/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api)で作成された製品 *アプリ* プロジェクトを使用します。 そのプロジェクトでは、基本的な ASP.NET 4.x Web API プロジェクトは次のように構成されます。

*Global.asax.cs* では、次の呼び出しが行われ `WebApiConfig.Register` ます。

[!code-csharp[](webapi/sample/3.x/ProductsApp/Global.asax.cs?highlight=14)]

`WebApiConfig`クラスは *App_Start* フォルダーにあり、静的メソッドがあり `Register` ます。

[!code-csharp[](webapi/sample/3.x/ProductsApp/App_Start/WebApiConfig.cs)]

前のクラスは次のとおりです。

* [属性ルーティング](/aspnet/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2)を構成します。ただし、実際には使用されません。
* ルーティングテーブルを構成します。
このサンプルコードでは、Url が形式と一致する必要 `/api/{controller}/{id}` が `{id}` あり、省略可能です。

以下のセクションでは、ASP.NET Core MVC に Web API プロジェクトを移行する方法について説明します。

## <a name="create-the-destination-project"></a>変換先プロジェクトを作成する

Visual Studio で新しい空のソリューションを作成し、移行する ASP.NET 4.x Web API プロジェクトを追加します。

1. **[ファイル]** メニューで、 **[新規作成]** > **[プロジェクト]** の順に選択します。
1. 空の **ソリューション** テンプレートを選択し、[ **次へ**] を選択します。
1. ソリューションに *WebAPIMigration* という名前を指定します。 **［作成］** を選択します
1. 既存の製品 *アプリ* プロジェクトをソリューションに追加します。

移行先の新しい API プロジェクトを追加します。

1. 新しい **ASP.NET Core Web アプリケーション** プロジェクトをソリューションに追加します。
1. [ **新しいプロジェクトの構成** ] ダイアログで、プロジェクトに *Productscore* という名前を付け、[ **作成**] を選択します。
1. **[新しい ASP.NET Core Web アプリケーションを作成する]** ダイアログで、 **[.NET Core]** と **[ASP.NET Core 3.1]** が選択されていることを確認します。 **[API]** プロジェクト テンプレートを選択し、 **[作成]** を選択します。
1. 新しい *Productscore* プロジェクトから *WeatherForecast.cs* と *Controllers/WeatherForecastController* サンプルファイルを削除します。

ソリューションに2つのプロジェクトが含まれるようになりました。 次のセクションでは、 *Productscore* プロジェクトの内容を *productscore* プロジェクトに移行する方法について説明します。

## <a name="migrate-configuration"></a>構成の移行

ASP.NET Core では、 *App_Start* フォルダーや *global.asax* ファイルは使用しません。 また、 *web.config* ファイルは発行時に追加されます。

`Startup` クラス:

* *Global.asax* を置き換えます。
* すべてのアプリスタートアップタスクを処理します。

詳細については、「<xref:fundamentals/startup>」を参照してください。

## <a name="migrate-models-and-controllers"></a>モデルとコントローラーの移行

次のコードは、 `ProductsController` ASP.NET Core に対して更新されるを示しています。

[!code-csharp[](webapi/sample/3.x/ProductsApp/Controllers/ProductsController.cs)]

`ProductsController`ASP.NET Core のを更新します。

1. *コントローラー/製品コントローラー .cs* と *モデル* フォルダーを元のプロジェクトから新しいプロジェクトにコピーします。
1. コピーしたファイルのルート名前空間をに変更し `ProductsCore` ます。
1. `using ProductsApp.Models;`ステートメントをに更新 `using ProductsCore.Models;` します。

ASP.NET Core には、次のコンポーネントは存在しません。

* `ApiController` クラス
* `System.Web.Http` 名前空間
* `IHttpActionResult` インターフェイス

次の変更を行います。

1. `ApiController` を <xref:Microsoft.AspNetCore.Mvc.ControllerBase> に変更します。 を追加し `using Microsoft.AspNetCore.Mvc;` て `ControllerBase` 参照を解決します。
1. `using System.Web.Http;`を削除します。
1. `GetProduct`アクションの戻り値の型をから `IHttpActionResult` に変更 `ActionResult<Product>` します。
1. `GetProduct`アクションの `return` ステートメントを次のように簡略化します。

    ```csharp
    return product;
    ```

## <a name="configure-routing"></a>ルーティングを構成する

ASP.NET Core *API* プロジェクトテンプレートには、生成されたコードにエンドポイントルーティング構成が含まれています。

次の <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseRouting%2A> とを <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints%2A> 呼び出します。

* [ミドルウェア](xref:fundamentals/middleware/index)パイプラインでルート照合とエンドポイント実行を登録します。
* 製品 *アプリ* プロジェクトの *App_Start/Webapiconfig.cs* ファイルを置き換えます。

[!code-csharp[](webapi/sample/3.x/ProductsCore/Startup.cs?name=snippet_Configure&highlight=10,14)]

次のようにルーティングを構成します。

1. 次の `ProductsController` 属性を使用してクラスをマークします。

    ```csharp
    [Route("api/[controller]")]
    [ApiController]
    ```

    上記の [`[Route]`](xref:Microsoft.AspNetCore.Mvc.RouteAttribute) 属性は、コントローラーの属性ルーティングパターンを構成します。 [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute)属性を使用すると、このコントローラーのすべてのアクションに対して属性のルーティングが必要になります。

    属性ルーティングでは、やなどのトークンがサポートさ `[controller]` `[action]` れます。 実行時には、各トークンは、属性が適用されたコントローラーまたはアクションの名前にそれぞれ置き換えられます。 トークン:
    * プロジェクト内のマジック文字列の数を減らします。
    * 自動名前変更リファクタリングが適用されている場合は、対応するコントローラーおよびアクションとの同期が維持されていることを確認します。
1. アクションに対する HTTP Get 要求を有効にし `ProductController` ます。
    * [`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute)アクションに属性を適用し `GetAllProducts` ます。
    * `[HttpGet("{id}")]`アクションに属性を適用し `GetProduct` ます。

移行したプロジェクトを実行し、に移動し `/api/products` ます。 3つの製品の完全な一覧が表示されます。 `/api/products/1` を参照します。 最初の製品が表示されます。

## <a name="additional-resources"></a>その他のリソース

* <xref:web-api/index>
* <xref:web-api/action-return-types>
* <xref:mvc/compatibility-version>

::: moniker-end

::: moniker range="<= aspnetcore-2.2"
## <a name="prerequisites"></a>前提条件

[!INCLUDE [prerequisites](../includes/net-core-prereqs-vs2019-2.2.md)]

## <a name="review-aspnet-4x-web-api-project"></a>ASP.NET 4.x Web API プロジェクトを確認する

この記事では、 [ASP.NET Web API 2 ではじめに](/aspnet/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api)で作成された製品 *アプリ* プロジェクトを使用します。 そのプロジェクトでは、基本的な ASP.NET 4.x Web API プロジェクトは次のように構成されます。

*Global.asax.cs* では、次の呼び出しが行われ `WebApiConfig.Register` ます。

[!code-csharp[](webapi/sample/2.x/ProductsApp/Global.asax.cs?highlight=14)]

`WebApiConfig`クラスは *App_Start* フォルダーにあり、静的メソッドがあり `Register` ます。

[!code-csharp[](webapi/sample/2.x/ProductsApp/App_Start/WebApiConfig.cs)]

このクラスは、実際にはプロジェクトで使用されていませんが、 [属性ルーティング](/aspnet/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2)を構成します。 また、ASP.NET Web API によって使用されるルーティングテーブルも構成します。 この場合、ASP.NET 4.x Web API では、Url が形式と一致する必要 `/api/{controller}/{id}` が `{id}` あり、省略可能です。

以下のセクションでは、ASP.NET Core MVC に Web API プロジェクトを移行する方法について説明します。

## <a name="create-the-destination-project"></a>変換先プロジェクトを作成する

Visual Studio で次の手順を実行します。

* [**ファイル**] [  >  **新しい**  >  **プロジェクト**] [  >  **その他のプロジェクト**] [  >  **Visual Studio ソリューション**] にアクセスします。 [ **空のソリューション**] を選択し、ソリューションに *WebAPIMigration* という名前を指定します。 **[OK]** をクリックします。
* 既存の製品 *アプリ* プロジェクトをソリューションに追加します。
* 新しい **ASP.NET Core Web アプリケーション** プロジェクトをソリューションに追加します。 ドロップダウンから **.Net Core** ターゲットフレームワークを選択し、[ **API** プロジェクト] テンプレートを選択します。 プロジェクトに *Productscore* という名前を付け、[ **OK** ] ボタンをクリックします。

ソリューションに2つのプロジェクトが含まれるようになりました。 次のセクションでは、 *Productscore* プロジェクトの内容を *productscore* プロジェクトに移行する方法について説明します。

## <a name="migrate-configuration"></a>構成の移行

ASP.NET Core では使用しません。

* *App_Start* フォルダーまたは *global.asax* ファイル
* *web.config* ファイルは発行時に追加されます。

`Startup` クラス:

* *Global.asax* を置き換えます。
* すべてのアプリスタートアップタスクを処理します。

詳細については、「<xref:fundamentals/startup>」を参照してください。

ASP.NET Core MVC では、がで呼び出されるときに、既定で属性ルーティングが組み込まれてい <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc*> `Startup.Configure` ます。 次の呼び出しを実行すると、製品 `UseMvc` *アプリ* プロジェクトの *App_Start* となります。

[!code-csharp[](webapi/sample/2.x/ProductsCore/Startup.cs?name=snippet_Configure&highlight=13)]

## <a name="migrate-models-and-controllers"></a>モデルとコントローラーの移行

次のコードは、 `ProductsController` ASP.NET Core の更新プログラムを示しています。 [!code-csharp[](webapi/sample/2.x/ProductsApp/Controllers/ProductsController.cs)]

`ProductsController`ASP.NET Core のを更新します。

1. 元のプロジェクトから新しいコントローラー */製品コントローラー .cs* をコピーします。
1. [ *モデル* ] フォルダーを元のプロジェクトから新しいプロジェクトにコピーします。
1. コピーしたファイルのルート名前空間をに変更し `ProductsCore` ます。
1. `using ProductsApp.Models;`ステートメントをに更新 `using ProductsCore.Models;` します。

ASP.NET Core には、次のコンポーネントは存在しません。

* `ApiController` クラス
* `System.Web.Http` 名前空間
* `IHttpActionResult` インターフェイス

次の変更を行います。

1. `ApiController` を <xref:Microsoft.AspNetCore.Mvc.ControllerBase> に変更します。 を追加し `using Microsoft.AspNetCore.Mvc;` て `ControllerBase` 参照を解決します。
1. `using System.Web.Http;`を削除します。
1. `GetProduct`アクションの戻り値の型をから `IHttpActionResult` に変更 `ActionResult<Product>` します。
1. `GetProduct`アクションの `return` ステートメントを次のように簡略化します。

    ```csharp
    return product;
    ```

## <a name="configure-routing"></a>ルーティングを構成する

次のようにルーティングを構成します。

1. 次の `ProductsController` 属性を使用してクラスをマークします。

    ```csharp
    [Route("api/[controller]")]
    [ApiController]
    ```

    上記の [`[Route]`](xref:Microsoft.AspNetCore.Mvc.RouteAttribute) 属性は、コントローラーの属性ルーティングパターンを構成します。 [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute)属性を使用すると、このコントローラーのすべてのアクションに対して属性のルーティングが必要になります。

    属性ルーティングでは、やなどのトークンがサポートさ `[controller]` `[action]` れます。 実行時には、各トークンは、属性が適用されたコントローラーまたはアクションの名前にそれぞれ置き換えられます。 トークンにより、プロジェクト内のマジック文字列の数が減少します。 また、トークンは、自動名前変更リファクタリングが適用された場合に、対応するコントローラーとアクションとの同期を維持することも保証します。
1. プロジェクトの互換モードを ASP.NET Core 2.2 に設定します。

    [!code-csharp[](webapi/sample/2.x/ProductsCore/Startup.cs?name=snippet_ConfigureServices&highlight=4)]

    上記の変更:

    * は、 `[ApiController]` コントローラーレベルで属性を使用するために必要です。
    * ASP.NET Core 2.2 で導入された動作が中断する可能性があります。
1. アクションに対する HTTP Get 要求を有効にし `ProductController` ます。
    * [`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute)アクションに属性を適用し `GetAllProducts` ます。
    * `[HttpGet("{id}")]`アクションに属性を適用し `GetProduct` ます。

移行したプロジェクトを実行し、に移動し `/api/products` ます。 3つの製品の完全な一覧が表示されます。 `/api/products/1` を参照します。 最初の製品が表示されます。

## <a name="compatibility-shim"></a>互換性 shim

[AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim)ライブラリは、ASP.NET 4.X Web API プロジェクトを ASP.NET Core に移動する互換性 shim を提供します。 互換性 shim は、ASP.NET 4.x Web API 2 のさまざまな規則をサポートするために ASP.NET Core を拡張します。 このドキュメントで以前に移植したサンプルは、互換性 shim が不要であるという基本的なものです。 大規模なプロジェクトでは、互換性 shim を使用すると、ASP.NET Core と ASP.NET 4.x Web API 2 間の API ギャップを一時的にブリッジするのに役立ちます。

Web API 互換性 shim は、大規模な ASP.NET 4.x Web API プロジェクトの ASP.NET Core への移行をサポートするための一時的な手段として使用することを意図しています。 時間の経過と共に、互換性 shim に依存するのではなく、ASP.NET Core パターンを使用するようにプロジェクトを更新する必要があります。

に含まれる互換性機能 `Microsoft.AspNetCore.Mvc.WebApiCompatShim` は次のとおりです。

* `ApiController`コントローラーの基本型を更新する必要がないように、型を追加します。
* Web API スタイルモデルのバインドを有効にします。 MVC モデルバインドは、既定では ASP.NET 4.x 5 と同様に機能します。 ASP.NET Core 互換性 shim は、モデルバインディングを ASP.NET 4.x Web API 2 モデルバインディング規則に似たものに変更します。 たとえば、複合型は要求本文から自動的にバインドされます。
* コントローラーアクションが型のパラメーターを受け取ることができるように、モデルバインディングを拡張 `HttpRequestMessage` します。
* アクションが型の結果を返すことを可能にするメッセージフォーマッタを追加し `HttpResponseMessage` ます。
* Web API 2 のアクションが応答の処理に使用する可能性がある応答メソッドを追加します。
  * `HttpResponseMessage` 機
    * `CreateResponse<T>`
    * `CreateErrorResponse`
  * アクションの結果メソッド:
    * `BadRequestErrorMessageResult`
    * `ExceptionResult`
    * `InternalServerErrorResult`
    * `InvalidModelStateResult`
    * `NegotiatedContentResult`
    * `ResponseMessageResult`
* のインスタンスを `IContentNegotiator` アプリの依存関係挿入 (DI) コンテナーに追加し、 [WebApi](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Client/)のコンテンツネゴシエーションに関連する型を使用できるようにします。 このような型の例としては、やなどがあり `DefaultContentNegotiator` `MediaTypeFormatter` ます。

互換性 shim を使用するには、次のようにします。

1. [AspNetCore WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim) NuGet パッケージをインストールします。
1. でを呼び出して、互換性 shim のサービスをアプリの DI コンテナーに登録 `services.AddMvc().AddWebApiConventions()` `Startup.ConfigureServices` します。
1. アプリの呼び出しのでを使用して、web API 固有のルートを定義し `MapWebApiRoute` `IRouteBuilder` `IApplicationBuilder.UseMvc` ます。

## <a name="additional-resources"></a>その他のリソース

* <xref:web-api/index>
* <xref:web-api/action-return-types>
* <xref:mvc/compatibility-version>
::: moniker-end
