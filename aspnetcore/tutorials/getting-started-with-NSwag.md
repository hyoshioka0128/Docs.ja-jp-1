---
title: NSwag と ASP.NET Core の概要
author: zuckerthoben
description: NSwag を使用し、ASP.NET Core Web API のドキュメント ページとヘルプ ページを生成する方法について説明します。
ms.author: scaddie
ms.custom: mvc
ms.date: 12/05/2019
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
uid: tutorials/get-started-with-nswag
ms.openlocfilehash: 882bbe659990cb8ea66110fccb85b8c6197df3f7
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588576"
---
# <a name="get-started-with-nswag-and-aspnet-core"></a>NSwag と ASP.NET Core の概要

作成者: [Christoph Nienaber](https://twitter.com/zuckerthoben)、[Rico Suter](https://rsuter.com)、[Dave Brock](https://twitter.com/daveabrock)

::: moniker range=">= aspnetcore-2.1"

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.NSwag)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.NSwag)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

::: moniker-end

NSwag には次の機能があります。

* Swagger UI と Swagger Generator を利用できます。
* 柔軟なコード生成が可能です。

NSwag では、既存の API は必要ありません&mdash;Swagger が組み込まれているサードパーティ製の API を利用し、クライアント実装を生成することができます。 NSwag を使用すれば、開発サイクルを短縮でき、API の変更にも容易に対応できます。

## <a name="register-the-nswag-middleware"></a>NSwag ミドルウェアの登録

NSwag ミドルウェアの登録でできること:

* 実装済みの Web API に対して Swagger 仕様を生成します。
* Web API を参照し、テストするサービスを Swagger UI に提供します。

[NSwag](https://github.com/RicoSuter/NSwag) ASP.NET Core ミドルウェアを使用するには、[NSwag.AspNetCore](https://www.nuget.org/packages/NSwag.AspNetCore/) NuGet パッケージをインストールします。 このパッケージには、Swagger 仕様、Swagger UI (v2 と v3)、[ReDoc UI](https://github.com/Rebilly/ReDoc) を生成し、それらにサービスを提供するミドルウェアが含まれています。

次のいずれかの方法で NSwag NuGet パッケージをインストールします。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* **[パッケージ マネージャー コンソール]** ウィンドウから:
  * **[ビュー]**  >  **[Other Windows]** \(その他の Windows\) >  **[パッケージ マネージャー コンソール]** に移動します。
  * *TodoApi.csproj* ファイルが存在するディレクトリに移動します。
  * 次のコマンドを実行します。

    ```powershell
    Install-Package NSwag.AspNetCore
    ```

* **[NuGet パッケージの管理]** ダイアログ ボックスから:
  * **[ソリューション エクスプローラー]**  >  **[NuGet パッケージの管理]** でプロジェクトを右クリックします。
  * **パッケージ ソース** を "nuget.org" に設定します。
  * 検索ボックスに「NSwag.AspNetCore」と入力します。
  * **[参照]** タブから "NSwag.AspNetCore" パッケージを選択して、 **[インストール]** をクリックします。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* **[Solution Pad]**  >  **[パッケージを追加]** で [*パッケージ*] フォルダーを右クリックします。
* **[パッケージを追加]** ウィンドウの **[ソース]** ドロップダウンを "nuget.org" に設定します。
* 検索ボックスに「NSwag.AspNetCore」と入力します。
* 結果ウィンドウから NSwag.AspNetCore パッケージを選択し、 **[パッケージを追加]** をクリックします。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

次のコマンドを実行します。

```dotnetcli
dotnet add TodoApi.csproj package NSwag.AspNetCore
```

---

## <a name="add-and-configure-swagger-middleware"></a>Swagger ミドルウェアを追加して構成する

次の手順を実行することで、ASP.NET Core アプリに Swagger を追加して構成します。

* `Startup.ConfigureServices` メソッドで、必須の Swagger サービスを登録します。

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.NSwag/Startup.cs?name=snippet_ConfigureServices&highlight=8)]

* `Startup.Configure` メソッドで、生成された Swagger 仕様と SwaggerUI 対応のミドルウェアを有効にします。

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.NSwag/Startup.cs?name=snippet_Configure&highlight=6-7)]

* アプリを起動します。 次に移動します。
  * `http://localhost:<port>/swagger` (Swagger UI が表示されます)。
  * `http://localhost:<port>/swagger/v1/swagger.json` (Swagger 仕様が表示されます)。

## <a name="code-generation"></a>コード生成

次のオプションのいずれかを選択することで、NSwag のコード生成機能を活用できます。

* [NSwagStudio](https://github.com/RicoSuter/NSwag/wiki/NSwagStudio):API のクライアント コードを C# または TypeScript で生成するための Windows デスクトップ アプリ。
* [NSwag.CodeGeneration.CSharp](https://www.nuget.org/packages/NSwag.CodeGeneration.CSharp/) または [NSwag.CodeGeneration.TypeScript](https://www.nuget.org/packages/NSwag.CodeGeneration.TypeScript/) NuGet パッケージ。これは、プロジェクト内でコードを生成するために使用します。
* [コマンド ライン](https://github.com/RicoSuter/NSwag/wiki/CommandLine)からの NSwag。
* [NSwag.MSBuild](https://github.com/RicoSuter/NSwag/wiki/NSwag.MSBuild) NuGet パッケージ。
* [Unchase OpenAPI (Swagger) Connected Service](https://marketplace.visualstudio.com/items?itemName=Unchase.unchaseopenapiconnectedservice):C# または TypeScript で API クライアント コードを生成するための Visual Studio の接続済みサービス。 NSwag を使用した OpenAPI サービス用の C# コントローラーも生成されます。

### <a name="generate-code-with-nswagstudio"></a>NSwagStudio を使用したコード生成

* [NSwagStudio GitHub リポジトリ](https://github.com/RicoSuter/NSwag/wiki/NSwagStudio)の手順に従って、NSwagStudio をインストールします。 NSwag リリース ページで、インストールや管理者特権なしで起動できる xcopy バージョンをダウンロードできます。
* NSwagStudio を起動し、 **[Swagger Specification URL]\(Swagger 仕様 URL\)** テキスト ボックスに *swagger.json* ファイルの URL を入力します。 たとえば、 *http://localhost:44354/swagger/v1/swagger.json* です。
* **[Create local Copy]\(ローカル コピーの作成\)** をクリックして、Swagger 仕様の JSON 表現を生成します。

  ![Swagger 仕様のローカル コピーの作成](web-api-help-pages-using-swagger/_static/CreateLocalCopy-NSwagStudio.PNG)

* **[Outputs]\(出力\)** 領域で、 **[CSharp Client]\(CSharp クライアント\)** チェック ボックスをオンにします。 ご自身のプロジェクトに応じて、 **[TypeScript Client]\(TypeScript クライアント\)** または **[CSharp Web API Controller]\(CSharp Web API コントローラー\)** を選択することもできます。 **[CSharp Web API Controller]\(CSharp Web API コントローラー\)** を選択した場合は、逆方向の生成が行われ、サービスの仕様からサービスが再構築されます。
* **[Generate Outputs]\(出力の生成\)** をクリックすると、*TodoApi.NSwag* プロジェクトの完全な C# クライアントの実装が作成されます。 生成されたクライアント コードを表示するには、 **[CSharp Client]\(CSharp クライアント\)** タブをクリックします。

```csharp
//----------------------
// <auto-generated>
//     Generated using the NSwag toolchain v12.0.9.0 (NJsonSchema v9.13.10.0 (Newtonsoft.Json v11.0.0.0)) (http://NSwag.org)
// </auto-generated>
//----------------------

namespace MyNamespace
{
    #pragma warning disable

    [System.CodeDom.Compiler.GeneratedCode("NSwag", "12.0.9.0 (NJsonSchema v9.13.10.0 (Newtonsoft.Json v11.0.0.0))")]
    public partial class TodoClient
    {
        private string _baseUrl = "https://localhost:44354";
        private System.Net.Http.HttpClient _httpClient;
        private System.Lazy<Newtonsoft.Json.JsonSerializerSettings> _settings;

        public TodoClient(System.Net.Http.HttpClient httpClient)
        {
            _httpClient = httpClient;
            _settings = new System.Lazy<Newtonsoft.Json.JsonSerializerSettings>(() =>
            {
                var settings = new Newtonsoft.Json.JsonSerializerSettings();
                UpdateJsonSerializerSettings(settings);
                return settings;
            });
        }

        public string BaseUrl
        {
            get { return _baseUrl; }
            set { _baseUrl = value; }
        }

        // code omitted for brevity
```

> [!TIP]
> **[Settings]\(設定\)** タブでの選択に基づいて、C# クライアント コードが生成されます。この設定を変更して、既定の名前空間の名前変更や同期メソッドの生成などのタスクを実行します。

* 生成された C# コードを、API を消費するクライアント プロジェクト内のファイルにコピーします。
* Web API の使用を開始します。

```csharp
 var todoClient = new TodoClient();

// Gets all to-dos from the API
 var allTodos = await todoClient.GetAllAsync();

 // Create a new TodoItem, and save it via the API.
var createdTodo = await todoClient.CreateAsync(new TodoItem());

// Get a single to-do by ID
var foundTodo = await todoClient.GetByIdAsync(1);
```

## <a name="customize-api-documentation"></a>API ドキュメントのカスタマイズ

Swagger には、Web API の消費を容易にする、オブジェクト モデルをドキュメント化するオプションがあります。

### <a name="api-info-and-description"></a>API 情報と説明

`Startup.ConfigureServices`メソッドでは、`AddSwaggerDocument` メソッドに渡された構成アクションによって、作成者、ライセンス、説明などの情報が追加されます。

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.NSwag/Startup2.cs?name=snippet_AddSwaggerDocument)]

Swagger UI には、バージョンの情報が表示されます。

![バージョン情報を含む Swagger UI](web-api-help-pages-using-swagger/_static/custom-info-nswag.png)

### <a name="xml-comments"></a>XML コメント

XML コメントを有効にするには、次の手順を実行します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

::: moniker range=">= aspnetcore-2.0"

* **ソリューション エクスプローラー** でプロジェクトを右クリックし、 **[<project_name>.csproj の編集]** を選択します。
* 強調表示された行を手動で *.csproj* ファイルに追加します。

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.NSwag/TodoApi.csproj?name=snippet_DocumentationFileElement&highlight=1-2,4)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

* **ソリューション エクスプローラー** で、プロジェクトを右クリックし、 **[プロパティ]** を選択します。
* **[ビルド]** タブの **[出力]** セクションの下にある **[XML ドキュメント ファイル]** チェック ボックスをオンにします。

::: moniker-end

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

::: moniker range=">= aspnetcore-2.0"

* *Solution Pad* から **Ctrl** キーを押しながらプロジェクト名をクリックします。 **[ツール]**  >  **[ファイルの編集]** に移動します。
* 強調表示された行を手動で *.csproj* ファイルに追加します。

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.NSwag/TodoApi.csproj?name=snippet_DocumentationFileElement&highlight=1-2,4)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

* **[プロジェクト オプション]** ダイアログ > **[ビルド]** > **[コンパイラ]** を開きます。
* **[全般オプション]** セクションの下にある **[XML ドキュメントを生成する]** チェック ボックスをオンにします。

::: moniker-end

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

強調表示された行を手動で *.csproj* ファイルに追加します。

::: moniker range=">= aspnetcore-2.0"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.NSwag/TodoApi.csproj?name=snippet_DocumentationFileElement&highlight=1-2,4)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.NSwag/TodoApi.csproj?name=snippet_DocumentationFileElement&highlight=1-2,4)]

::: moniker-end

---

### <a name="data-annotations"></a>データの注釈

::: moniker range="<= aspnetcore-2.0"

NSwag では [Reflection](/dotnet/csharp/programming-guide/concepts/reflection) が使用されます。また、Web API アクションに推奨される戻り値の型は [IActionResult](xref:Microsoft.AspNetCore.Mvc.IActionResult) です。そのため、アクションの内容や戻り値の型を推測することはできません。

次に例を示します。

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.NSwag/Controllers/TodoController.cs?name=snippet_CreateAction)]

先行するアクションによって `IActionResult` が返されますが、このアクションの内部では [CreatedAtRoute](xref:System.Web.Http.ApiController.CreatedAtRoute*) または [BadRequest](xref:System.Web.Http.ApiController.BadRequest*) を返しています。 データ注釈を使用して、このアクションによって返される既知の HTTP 状態コードをクライアントに通知します。 次の属性でアクションをマークします。

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.NSwag/Controllers/TodoController.cs?name=snippet_CreateActionAttributes)]

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

NSwag では [Reflection](/dotnet/csharp/programming-guide/concepts/reflection) が使用されます。また、Web API アクションに推奨される戻り値の型は [ActionResult\<T>](xref:Microsoft.AspNetCore.Mvc.ActionResult%601) です。そのため、`T` によって定義される戻り値の型のみを推測できます。 考えられる他の戻り値の型を自動的に推測することはできません。

次に例を示します。

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.NSwag/Controllers/TodoController.cs?name=snippet_CreateAction)]

前のアクションでは、`ActionResult<T>` を返します。 アクションの内部では、[CreatedAtRoute](xref:System.Web.Http.ApiController.CreatedAtRoute*) を返します。 このコントローラーには [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) 属性が指定されているため、[BadRequest](xref:System.Web.Http.ApiController.BadRequest*) 応答が発生する場合もあります。 詳細については、「[自動的な HTTP 400 応答](xref:web-api/index#automatic-http-400-responses)」を参照してください。 データ注釈を使用して、このアクションによって返される既知の HTTP 状態コードをクライアントに通知します。 次の属性でアクションをマークします。

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.NSwag/Controllers/TodoController.cs?name=snippet_CreateActionAttributes)]

ASP.NET Core 2.2 以降では、明示的に個別のアクションを `[ProducesResponseType]` で修飾する代わりに、規約を使用できます。 詳細については、「<xref:web-api/advanced/conventions>」を参照してください。

::: moniker-end

Swagger ジェネレーターでは、このアクションを正確に表現できるようになりました。生成されたクライアントでは、エンドポイントの呼び出し時に受け取るものが認識されます。 すべてのアクションをこれらの属性でマークすることをお勧めします。

API アクションで返される HTTP 応答のガイドラインについては、[RFC 7231 仕様](https://tools.ietf.org/html/rfc7231#section-4.3)をご覧ください。
