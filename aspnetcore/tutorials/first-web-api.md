---
title: 'チュートリアル: ASP.NET Core で Web API を作成する'
author: rick-anderson
description: ASP.NET Core で Web API をビルドする方法を学習します。
ms.author: riande
ms.custom: mvc, devx-track-js
ms.date: 02/04/2021
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
- Models
uid: tutorials/first-web-api
ms.openlocfilehash: 1f7c7db857090ff0a174d37b86e1265bab40b4fd
ms.sourcegitcommit: f77a7467651bab61b24261da9dc5c1dd75fc1fa9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/17/2021
ms.locfileid: "100564094"
---
# <a name="tutorial-create-a-web-api-with-aspnet-core"></a>チュートリアル: ASP.NET Core で Web API を作成する

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)、[Kirk Larkin](https://twitter.com/serpent5)、[Mike Wasson](https://github.com/mikewasson)

このチュートリアルでは、ASP.NET Core で Web API をビルドする方法の基本について説明します。

::: moniker range=">= aspnetcore-5.0"

このチュートリアルでは、次の作業を行う方法について説明します。

> [!div class="checklist"]
> * Web API プロジェクトを作成する。
> * モデル クラスとデータベース コンテキストを追加する。
> * CRUD メソッドを使用してコントローラーのスキャフォールディング。
> * ルーティング、URL パス、戻り値を構成する。
> * Postman で Web API を呼び出す。

最後に、データベースに格納されている "To Do" アイテムを管理できる Web API が作成されます。

## <a name="overview"></a>概要

このチュートリアルでは、次の API を作成します。

|API | 説明 | 要求本文 | 応答本文 |
|--- | ---- | ---- | ---- |
|`GET /api/TodoItems` | すべての To Do アイテムを取得します。 | None | To Do アイテムの配列|
|`GET /api/TodoItems/{id}` | ID でアイテムを取得します。 | None | To Do アイテム|
|`POST /api/TodoItems` | 新しいアイテムを追加します。 | To Do アイテム | To Do アイテム |
|`PUT /api/TodoItems/{id}` | 既存のアイテムを更新します。&nbsp; | To Do アイテム | None |
|`DELETE /api/TodoItems/{id}` &nbsp; &nbsp; | アイテムを削除します &nbsp; &nbsp; | None | None|

次の図は、アプリのデザインを示しています。

![クライアントは、左側のボックスに表されます。 要求を送信し、アプリケーション (右側に描画されたボックス) から応答を受信します。 アプリケーション ボックス内の 3 つのボックスは、コントローラー、モデル、およびデータ アクセス レイヤーを表しています。 要求はアプリケーションのコントローラーに送られ、コントローラーとデータ アクセス レイヤー間で読み取り/書き込み操作が行われます。 モデルはシリアル化され、応答でクライアントに返されます。](first-web-api/_static/architecture.png)

## <a name="prerequisites"></a>必須コンポーネント

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-5.0.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-5.0.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-5.0.md)]

---

## <a name="create-a-web-project"></a>Web プロジェクトの作成

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* **[ファイル]** メニューで、 **[新規作成]** > **[プロジェクト]** の順に選択します。
* **[ASP.NET Core Web アプリケーション]** テンプレートを選択して、 **[次へ]** をクリックします。
* プロジェクトに「*TodoApi*」という名前を付け、 **[作成]** をクリックします。
* **[新しい ASP.NET Core Web アプリケーションを作成する]** ダイアログで、 **[.NET Core]** と **[ASP.NET Core 5.0]** が選択されていることを確認します。 **API** テンプレートを選択し、 **[作成]** をクリックします。

![VS の [新しいプロジェクト] ダイアログ](first-web-api/_static/5/vs.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* [統合ターミナル](https://code.visualstudio.com/docs/editor/integrated-terminal)を開きます。
* ディレクトリ (`cd`) を、プロジェクト フォルダーを格納するフォルダーに変更します。
* 次のコマンドを実行します。

   ```dotnetcli
   dotnet new webapi -o TodoApi
   cd TodoApi
   dotnet add package Microsoft.EntityFrameworkCore.InMemory
   code -r ../TodoApi
   ```

* ダイアログ ボックスで、プロジェクトに必要な資産を追加するかどうかを確認されたら、 **[はい]** を選択します。

  上のコマンドでは以下の操作が行われます。

  * 新しい Web API プロジェクトを作成し、Visual Studio Code で開きます。
  * 次のセクションで必要な NuGet パッケージを追加します。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* **[ファイル]** > **[新しいソリューション]** の順に選択します。

  ![macOS の新しいソリューション](first-web-api-mac/_static/sln.png)

* バージョン 8.6 より前の Visual Studio for Mac では、 **[.NET Core]**  >  **[アプリ]**  >  **[API]**  >  **[次へ]** の順に選択します。 バージョン 8.6 以降では、 **[Web and Console]** \(Web とコンソール\) >  **[アプリ]**  >  **[API]**  >  **[次へ]** の順に選択します。

  ![macOS での API テンプレートの選択](first-web-api-mac/_static/api_template.png)

* **[Configure the new ASP.NET Core Web API]\(新しい ASP.NET Core Web API を構成する\)** ダイアログで、最新の .NET Core 5.x **ターゲット フレームワーク** を選択します。 **[次へ]** を選択します。

* **[プロジェクト名]** に「*TodoApi*」と入力し、 **[作成]** を選択します。

  ![構成ダイアログ](first-web-api-mac/_static/2.png)

[!INCLUDE[](~/includes/mac-terminal-access.md)]

プロジェクト フォルダーでコマンド ターミナルを開き、次のコマンドを実行します。

   ```dotnetcli
   dotnet add package Microsoft.EntityFrameworkCore.InMemory
   ```

---

### <a name="test-the-project"></a>プロジェクトをテストする

プロジェクト テンプレートにより、[Swagger](xref:tutorials/web-api-help-pages-using-swagger) をサポートする `WeatherForecast` API が作成されます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Ctrl + F5 キーを押して、デバッガーなしで実行します。

[!INCLUDE[](~/includes/trustCertVS.md)]

  Visual Studio により、以下が起動されます。

* IIS Express Web サーバー。
* 既定のブラウザーです。`https://localhost:<port>/swagger/index.html` に移動します。`<port>` はランダムに選択されるポート番号です。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/trustCertVSC.md)]

Ctrl キーを押しながら F5 キーを押して、アプリを実行します。 ブラウザーで、次の URL に移動します: [https://localhost:5001/swagger](https://localhost:5001/swagger)

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

**[実行]**  >  **[デバッグの開始]** の順に選択してアプリを起動します。 Visual Studio for Mac でブラウザーが起動し、`https://localhost:<port>` にアクセスします。ここで、`<port>` はランダムに選択されたポート番号になります。 HTTP 404 (Not Found) エラーが返されます。 URL に `/swagger` を追加します (URL を `https://localhost:<port>/swagger` に変更します)。

---

Swagger ページ `/swagger/index.html` が表示されます。 **[取得]**  >  **[試してみる]**  >  **[実行]** を選択します。 ページに以下が表示されます。

* WeatherForecast API をテストするための [Curl](https://curl.haxx.se/) コマンド。
* WeatherForecast API をテストする URL。
* 応答コード、本文、およびヘッダー。
* メディアの種類と、値とスキーマの例を含むドロップ ダウン リスト ボックス。

<!-- Review: Do we care the IE generates several errors. It shows the data, but with  Unrecognized response type; displaying content as text.
-->
Swagger は、Web API の有用なドキュメントやヘルプ ページを生成するために使用されます。 このチュートリアルでは、Web API の作成について説明します。 Swagger の詳細については、<xref:tutorials/web-api-help-pages-using-swagger> を参照してください。

ブラウザーで **要求 URL** をコピーして貼り付けます: `https://localhost:<port>/WeatherForecast`

次のような JSON が返されます。

```json
[
    {
        "date": "2019-07-16T19:04:05.7257911-06:00",
        "temperatureC": 52,
        "temperatureF": 125,
        "summary": "Mild"
    },
    {
        "date": "2019-07-17T19:04:05.7258461-06:00",
        "temperatureC": 36,
        "temperatureF": 96,
        "summary": "Warm"
    },
    {
        "date": "2019-07-18T19:04:05.7258467-06:00",
        "temperatureC": 39,
        "temperatureF": 102,
        "summary": "Cool"
    },
    {
        "date": "2019-07-19T19:04:05.7258471-06:00",
        "temperatureC": 10,
        "temperatureF": 49,
        "summary": "Bracing"
    },
    {
        "date": "2019-07-20T19:04:05.7258474-06:00",
        "temperatureC": -1,
        "temperatureF": 31,
        "summary": "Chilly"
    }
]
```

### <a name="update-the-launchurl"></a>launchUrl を更新する

*Properties\launchSettings.json* で、`launchUrl` を `"swagger"` から `"api/TodoItems"` に更新します。

```json
"launchUrl": "api/TodoItems",
```

Swagger が削除されているため、上記のマークアップにより、開始される URL が、次のセクションで追加されるコントローラーの GET メソッドに変更されます。

## <a name="add-a-model-class"></a>モデル クラスの追加

*モデル* は、アプリが管理するデータを表すクラスのセットです。 このアプリのモデルは、単一の `TodoItem` クラスです。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* **ソリューション エクスプローラー** で、プロジェクトを右クリックします。 **[追加]**  >  **[新しいフォルダー]** の順に選択します。 フォルダーに「 *Models* 」という名前を付けます。

* *Models* フォルダーを右クリックして、 **[追加]**  >  **[クラス]** の順に選択します。 クラスに「*TodoItem*」という名前を付け、 **[追加]** を選択します。

* テンプレート コードを次のコードに置き換えます。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* *Models* という名前のフォルダーを追加します。

* 次のコードを使用して、`TodoItem` クラスを *Models* フォルダーに追加します。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* プロジェクトを右クリックします。 **[追加]**  >  **[新しいフォルダー]** の順に選択します。 フォルダーに「 *Models* 」という名前を付けます。

  ![新しいフォルダー](first-web-api-mac/_static/folder.png)

* *Models* フォルダーを右クリックして、 **[追加]** > **[新しいファイル]** > **[全般]** > **[空のクラス]** の順に選択します。

* クラスに「*TodoItem*」という名前を付け、 **[新規]** をクリックします。

* テンプレート コードを次のコードに置き換えます。

---

  [!code-csharp[](first-web-api/samples/5.x/TodoApi/Models/TodoItem.cs?name=snippet)]

`Id` プロパティは、リレーショナル データベース内の一意のキーとして機能します。

モデル クラスはプロジェクト内のどこでも使用できますが、慣例により *Models* フォルダーが使用されます。

## <a name="add-a-database-context"></a>データベース コンテキストの追加

*データベース コンテキスト* は、データ モデルに対して Entity Framework 機能を調整するメイン クラスです。 このクラスは <xref:Microsoft.EntityFrameworkCore.DbContext?displayProperty=fullName> クラスから派生させて作成します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

### <a name="add-nuget-packages"></a>NuGet パッケージを追加する

* **[ツール]** メニューで **[NuGet パッケージ マネージャー]、[ソリューションの NuGet パッケージの管理]** の順に選択します。
* **[参照]** タブを選択し、検索ボックスに「**Microsoft.EntityFrameworkCore.InMemory**」と入力します。
* 左側のウィンドウで、 **[Microsoft.EntityFrameworkCore.InMemory]** を選択します。
* 右側のウィンドウで **[プロジェクト]** チェックボックスをオンにして、 **[インストール]** を選択します。

![NuGet パッケージ マネージャー](first-web-api/_static/5/vsNuGet.png)

## <a name="add-the-todocontext-database-context"></a>TodoContext データベースコンテキストの追加

* *Models* フォルダーを右クリックして、 **[追加]**  >  **[クラス]** の順に選択します。 クラスに「*TodoContext*」という名前を付け、 **[追加]** をクリックします。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

* *Models* フォルダーに `TodoContext` クラスを追加します。

---

* 次のコードを入力します。

  [!code-csharp[](first-web-api/samples/5.x/TodoApi/Models/TodoContext.cs)]

## <a name="register-the-database-context"></a>データベース コンテキストの登録

ASP.NET Core で、サービス (DB コンテキストなど) を[依存関係の挿入 (DI)](xref:fundamentals/dependency-injection)コンテナーに登録する必要があります。 コンテナーは、コントローラーにサービスを提供します。

次のコードを使用して *Startup.cs* を更新します。

[!code-csharp[](first-web-api/samples/5.x/TodoApi/Startup.cs?highlight=7-8,23-24&name=snippet_all)]

上記のコードでは次の操作が行われます。

* Swagger 呼び出しを削除します。
* 不要な `using` 宣言を削除します。
* DI コンテナーにデータベース コンテキストを追加します。
* データベース コンテキストがメモリ内データベースを使用することを指定します。

## <a name="scaffold-a-controller"></a>コントローラーのスキャフォールディング

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* *Controllers* フォルダーを右クリックします。
* **[追加]** > **[スキャフォールディングされた新しい項目]** を選択します。
* **[Entity Framework を使用したアクションがある API コントローラー]** を選択してから、 **[追加]** を選択します。
* **[Entity Framework を使用したアクションがある API コントローラー]** ダイアログで次を実行します。

  * **モデル クラス** で **TodoItem (TodoApi.Models)** を選択します。
  * **データ コンテキスト クラス** で **TodoContext (TodoApi.Models)** を選択します。
  * **[追加]** を選びます。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

次のコマンドを実行します。

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet tool update -g dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator controller -name TodoItemsController -async -api -m TodoItem -dc TodoContext -outDir Controllers
```

上のコマンドでは以下の操作が行われます。

* スキャフォールディングに必要な NuGet パッケージを追加します。
* スキャフォールディング エンジン (`dotnet-aspnet-codegenerator`) をインストールします。
* `TodoItemsController` をスキャフォールディングします。

---

生成されたコードでは次の操作が行われます。

* クラスを [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) 属性でマークします。 この属性は、コントローラーが Web API 要求に応答することを示します。 属性によって有効化される特定の動作については、「<xref:web-api/index>」 を参照してください。
* DI を使用して、データベース コンテキスト (`TodoContext`) をコントローラーに挿入します。 データベース コンテキストは、コントローラーの各 [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) メソッドで使用されます。

ASP.NET Core テンプレートの対象は次のとおりです。

* ビューを含むコントローラーには、ルート テンプレートの `[action]` が含まれます。
* API コントローラーには、ルート テンプレートの `[action]` が含まれません。

`[action]` トークンがルート テンプレート内にない場合、[アクション](xref:mvc/controllers/routing#action)名はルートから除外されます。 つまり、アクションの関連付けられたメソッド名は一致するルートでは使用されません。

## <a name="update-the-posttodoitem-create-method"></a>PostTodoItem 作成メソッドの更新

[nameof](/dotnet/csharp/language-reference/operators/nameof) 演算子を使用するために、`PostTodoItem` で return ステートメントを更新します。

[!code-csharp[](first-web-api/samples/5.x/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Create)]

[`[HttpPost]`](xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute) 属性が示すように、上記のコードは HTTP POST メソッドです。 このメソッドは、HTTP 要求の本文から To Do アイテムの値を取得します。

詳細については、「[Http[Verb] 属性を使用する属性ルーティング](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes)」を参照してください。

<xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*> メソッド:

* 成功すると、[HTTP 201 状態コード](https://developer.mozilla.org/docs/Web/HTTP/Status/201)が返されます。 HTTP 201 は、サーバーに新しいリソースを作成する HTTP POST メソッドに対する標準の応答です。
* 応答に [Location](https://developer.mozilla.org/docs/Web/HTTP/Headers/Location) ヘッダーが追加されます。 `Location` ヘッダーでは、新しく作成された To Do アイテムの [URI](https://developer.mozilla.org/docs/Glossary/URI) が指定されます。 詳細については、「[10.2.2 201 Created](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)」を参照してください。
* `GetTodoItem` アクションを参照して `Location` ヘッダーの URI を作成します。 C# の `nameof` キーワードを使って、`CreatedAtAction` 呼び出しでアクション名をハードコーディングすることを回避しています。

### <a name="install-postman"></a>Postman のインストール

このチュートリアルでは、Postman を使用して Web API をテストします。

* [Postman](https://www.getpostman.com/downloads/) をインストールします。
* Web アプリを起動します。
* Postman を起動します。
* **[SSL 証明書の確認]** を無効にします。
  * **[ファイル]** > **[設定]** ( **[全般]** タブ) で、 **[SSL 証明書の確認]** を無効にします。
    > [!WARNING]
    > コントローラーをテストした後、SSL 証明書の検証を再度有効にします。

<a name="post"></a>

### <a name="test-posttodoitem-with-postman"></a>Postman を使用した PostTodoItem のテスト

* 新しい要求を作成します。
* HTTP メソッドを `POST` に設定します。
* URI を `https://localhost:<port>/api/TodoItems` に設定します。 たとえば、「 `https://localhost:5001/api/TodoItems` 」のように入力します。
* **[Body]** タブを選択します。
* **[raw]** ラジオ ボタンを選択します。
* 型を **[JSON (application/json)]** に設定します。
* 要求本文に、To Do アイテムの JSON を入力します。

    ```json
    {
      "name":"walk dog",
      "isComplete":true
    }
    ```

* **[Send]** を選択します。

  ![Postman での Create 要求](first-web-api/_static/3/create.png)

### <a name="test-the-location-header-uri"></a>場所ヘッダー URI のテスト

場所ヘッダー URI は、ブラウザーでテストできます。 場所ヘッダー URI をコピーしてブラウザーに貼り付けます。

Postman でテストするには:

* **[Response]** ウィンドウで、 **[Headers]** タブを選択します。
* **[Location]** ヘッダー値をコピーします。

  ![Postman コンソールの [Headers] タブ](first-web-api/_static/3/create.png)

* HTTP メソッドを `GET` に設定します。
* URI を `https://localhost:<port>/api/TodoItems/1` に設定します。 たとえば、「 `https://localhost:5001/api/TodoItems/1` 」のように入力します。
* **[Send]** を選択します。

## <a name="examine-the-get-methods"></a>GET メソッドの確認

2 つの GET エンドポイントが実装されます。

* `GET /api/TodoItems`
* `GET /api/TodoItems/{id}`

ブラウザーまたは Postman から 2 つのエンドポイントを呼び出すことによって、アプリをテストします。 次に例を示します。

* `https://localhost:5001/api/TodoItems`
* `https://localhost:5001/api/TodoItems/1`

`GetTodoItems` への呼び出しによって、次のような応答が生成されます。

```json
[
  {
    "id": 1,
    "name": "Item1",
    "isComplete": false
  }
]
```

### <a name="test-get-with-postman"></a>Postman を使用して Get をテストする

* 新しい要求を作成します。
* HTTP メソッドを **GET** に設定します。
* 要求 URI を `https://localhost:<port>/api/TodoItems` に設定します。 たとえば、`https://localhost:5001/api/TodoItems` のようにします。
* Postman で **[Two pane view]** を設定します。
* **[Send]** を選択します。

このアプリではメモリ内データベースが使用されます。 アプリが停止して開始された場合、上記の GET 要求はデータを返しません。 データが返されない場合は、アプリにデータを [POST](#post) します。

## <a name="routing-and-url-paths"></a>ルーティングと URL パス

[`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) 属性は、HTTP GET 要求に応答するメソッドを表します。 各メソッドの URL パスは次のように構成されます。

* コントローラーの `Route` 属性でテンプレート文字列を使用します。

  [!code-csharp[](first-web-api/samples/5.x/TodoApi/Controllers/TodoItemsController.cs?name=TodoController&highlight=1)]

* `[controller]` をコントローラーの名前 (慣例では "Controller" サフィックスを除くコントローラー クラス名) に置き換えます。 このサンプルでは、コントローラー クラス名は **TodoItems** Controller なので、コントローラー名は "TodoItems" です。 ASP.NET Core の[ルーティング](xref:mvc/controllers/routing)では、大文字と小文字が区別されません。
* `[HttpGet]` 属性にルート テンプレート (たとえば、`[HttpGet("products")]`) がある場合は、それをパスに追加します。 このサンプルではテンプレートを使用しません。 詳細については、「[Http[Verb] 属性を使用する属性ルーティング](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes)」を参照してください。

次の `GetTodoItem` メソッドで、`"{id}"` は To Do アイテムの一意識別子に使用するプレースホルダーの変数です。 `GetTodoItem` が呼び出されると、その `id` パラメーター内のメソッドに URL の `"{id}"` の値が指定されます。

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_GetByID&highlight=1-2)]

## <a name="return-values"></a>戻り値

`GetTodoItems` と `GetTodoItem` メソッドの戻り値の型は、[ActionResult\<T> 型](xref:web-api/action-return-types#actionresultt-type)です。 ASP.NET Core は自動的にオブジェクトを [JSON](https://www.json.org/) にシリアル化して、応答メッセージの本文に JSON を書き込みます。 ハンドルされない例外がないと仮定すると、この戻り値の型の応答コードは [200 OK](https://developer.mozilla.org/docs/Web/HTTP/Status/200) です。 ハンドルされない例外は 5xx エラーに変換されます。

`ActionResult` 戻り値の型は、幅広い範囲の HTTP 状態コードを表すことができます。 たとえば、`GetTodoItem` は、次の 2 つの異なる状態値を返す可能性があります。

* 要求された ID に一致するアイテムがない場合、このメソッドにより [404 ステータス](https://developer.mozilla.org/docs/Web/HTTP/Status/404) <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound%2A> エラー コードが返されます。
* それ以外の場合、メソッドは JSON 応答本文で 200 を返します。 戻り値が `item` の場合、HTTP 200 応答が返されます。

## <a name="the-puttodoitem-method"></a>PutTodoItem メソッド

`PutTodoItem` メソッドを検証します。

[!code-csharp[](first-web-api/samples/5.x/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Update)]

`PutTodoItem` は `PostTodoItem` と似ていますが、HTTP PUT を使用します。 応答は [204 (No Content)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) となります。 HTTP 仕様に従って、PUT 要求では、変更だけでなく、更新されたエンティティ全体を送信するようクライアントに求めます。 部分的な更新をサポートするには、[HTTP PATCH](xref:Microsoft.AspNetCore.Mvc.HttpPatchAttribute) を使用します。

`PutTodoItem` 呼び出しでエラーが発生した場合、`GET` を呼び出してデータベース内にアイテムがあることを確認してください。

### <a name="test-the-puttodoitem-method"></a>PutTodoItem メソッドのテスト

このサンプルでは、アプリを起動するたびに開始することが必要なメモリ内データベースが使われています。 PUT 呼び出しを実行する前に、データベース内にアイテムが存在している必要があります。 GET を呼び出して、PUT 呼び出しを実行する前にデータベース内にアイテムが確実に存在していることを確認します。

Id = 1 の To Do アイテムを更新し、その名前を `"feed fish"` に設定します。

```json
  {
    "Id":1,
    "name":"feed fish",
    "isComplete":true
  }
```

次の図は、Postman の更新を示しています。

![204 (No Content) の応答を示す Postman コンソール](first-web-api/_static/3/pmcput.png)

## <a name="the-deletetodoitem-method"></a>DeleteTodoItem メソッド

`DeleteTodoItem` メソッドを検証します。

[!code-csharp[](first-web-api/samples/5.x/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Delete)]

### <a name="test-the-deletetodoitem-method"></a>DeleteTodoItem メソッドのテスト

Postman を使用して、To Do アイテムを削除します。

* メソッドを `DELETE` に設定します。
* 削除するオブジェクトの URI (たとえば、`https://localhost:5001/api/TodoItems/1`) を設定します。
* **[Send]** を選択します。

<a name="over-post-v5"></a>

## <a name="prevent-over-posting"></a>過剰な投稿を防止する

現在、サンプル アプリでは `TodoItem` オブジェクト全体が公開されています。 通常、運用環境のアプリでは、モデルのサブセットを使用して入力されるデータおよび返されるデータが制限されています。 その背景には複数の理由があり、セキュリティは主なものです。 モデルのサブセットは、通常、データ転送オブジェクト (DTO)、入力モデル、またはビュー モデルと呼ばれます。 この記事では **DTO** を使用しています。

DTO は次の目的で使用できます。

* 過剰な投稿を防止する。
* クライアントが表示しないことになっているプロパティを非表示にする。
* ペイロード サイズを減らすために、いくつかのプロパティを省略する。
* 入れ子になったオブジェクトを含むオブジェクト グラフをフラット化する。 フラット化されたオブジェクト グラフは、クライアントにとってより便利になる可能性があります。

DTO のアプローチを実演するために、`TodoItem` クラスを更新して、シークレット フィールドを含めます。

[!code-csharp[](first-web-api/samples/5.x/TodoApiDTO/Models/TodoItem.cs?name=snippet&highlight=8)]

シークレット フィールドは、このアプリでは非表示にする必要がありますが、管理アプリの場合は公開することを選択できます。

シークレット フィールドを投稿および取得できることを確認します。

次のように DTO モデルを作成します。

[!code-csharp[](first-web-api/samples/5.x/TodoApiDTO/Models/TodoItemDTO.cs?name=snippet)]

`TodoItemDTO` を使用するように `TodoItemsController` を更新します。

[!code-csharp[](first-web-api/samples/5.x/TodoApiDTO/Controllers/TodoItemsController.cs?name=snippet)]

シークレット フィールドを投稿または取得できないことを確認します。

## <a name="call-the-web-api-with-javascript"></a>JavaScript を使用した Web API の呼び出し

「[チュートリアル:JavaScript を使用して ASP.NET Core Web API を呼び出す](xref:tutorials/web-api-javascript)」を参照してください。

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

このチュートリアルでは、次の作業を行う方法について説明します。

> [!div class="checklist"]
> * Web API プロジェクトを作成する。
> * モデル クラスとデータベース コンテキストを追加する。
> * CRUD メソッドを使用してコントローラーのスキャフォールディング。
> * ルーティング、URL パス、戻り値を構成する。
> * Postman で Web API を呼び出す。

最後に、データベースに格納されている "To Do" アイテムを管理できる Web API が作成されます。

## <a name="overview"></a>概要

このチュートリアルでは、次の API を作成します。

|API | 説明 | 要求本文 | 応答本文 |
|--- | ---- | ---- | ---- |
|`GET /api/TodoItems` | すべての To Do アイテムを取得します。 | None | To Do アイテムの配列|
|`GET /api/TodoItems/{id}` | ID でアイテムを取得します。 | None | To Do アイテム|
|`POST /api/TodoItems` | 新しいアイテムを追加します。 | To Do アイテム | To Do アイテム |
|`PUT /api/TodoItems/{id}` | 既存のアイテムを更新します。&nbsp; | To Do アイテム | None |
|`DELETE /api/TodoItems/{id}` &nbsp; &nbsp; | アイテムを削除します &nbsp; &nbsp; | None | None|

次の図は、アプリのデザインを示しています。

![クライアントは、左側のボックスに表されます。 要求を送信し、アプリケーション (右側に描画されたボックス) から応答を受信します。 アプリケーション ボックス内の 3 つのボックスは、コントローラー、モデル、およびデータ アクセス レイヤーを表しています。 要求はアプリケーションのコントローラーに送られ、コントローラーとデータ アクセス レイヤー間で読み取り/書き込み操作が行われます。 モデルはシリアル化され、応答でクライアントに返されます。](first-web-api/_static/architecture.png)

## <a name="prerequisites"></a>必須コンポーネント

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.1.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.1.md)]

---

## <a name="create-a-web-project"></a>Web プロジェクトの作成

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* **[ファイル]** メニューで、 **[新規作成]** > **[プロジェクト]** の順に選択します。
* **[ASP.NET Core Web アプリケーション]** テンプレートを選択して、 **[次へ]** をクリックします。
* プロジェクトに「*TodoApi*」という名前を付け、 **[作成]** をクリックします。
* **[新しい ASP.NET Core Web アプリケーションを作成する]** ダイアログで、 **[.NET Core]** と **[ASP.NET Core 3.1]** が選択されていることを確認します。 **API** テンプレートを選択し、 **[作成]** をクリックします。

![VS の [新しいプロジェクト] ダイアログ](first-web-api/_static/vs3.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* [統合ターミナル](https://code.visualstudio.com/docs/editor/integrated-terminal)を開きます。
* ディレクトリ (`cd`) を、プロジェクト フォルダーを格納するフォルダーに変更します。
* 次のコマンドを実行します。

   ```dotnetcli
   dotnet new webapi -o TodoApi
   cd TodoApi
   dotnet add package Microsoft.EntityFrameworkCore.InMemory
   code -r ../TodoApi
   ```

* ダイアログ ボックスで、プロジェクトに必要な資産を追加するかどうかを確認されたら、 **[はい]** を選択します。

  上のコマンドでは以下の操作が行われます。

  * 新しい Web API プロジェクトを作成し、Visual Studio Code で開きます。
  * 次のセクションで必要な NuGet パッケージを追加します。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* **[ファイル]** > **[新しいソリューション]** の順に選択します。

  ![macOS の新しいソリューション](first-web-api-mac/_static/sln.png)

* バージョン 8.6 より前の Visual Studio for Mac では、 **[.NET Core]**  >  **[アプリ]**  >  **[API]**  >  **[次へ]** の順に選択します。 バージョン 8.6 以降では、 **[Web and Console]** \(Web とコンソール\) >  **[アプリ]**  >  **[API]**  >  **[次へ]** の順に選択します。

  ![macOS での API テンプレートの選択](first-web-api-mac/_static/api_template.png)

* **[Configure the new ASP.NET Core Web API]\(新しい ASP.NET Core Web API を構成する\)** ダイアログで、最新の .NET Core 3.x **ターゲット フレームワーク** を選択します。 **[次へ]** を選択します。

* **[プロジェクト名]** に「*TodoApi*」と入力し、 **[作成]** を選択します。

  ![構成ダイアログ](first-web-api-mac/_static/2.png)

[!INCLUDE[](~/includes/mac-terminal-access.md)]

プロジェクト フォルダーでコマンド ターミナルを開き、次のコマンドを実行します。

   ```dotnetcli
   dotnet add package Microsoft.EntityFrameworkCore.InMemory
   ```

---

### <a name="test-the-api"></a>API のテスト

プロジェクト テンプレートによって `WeatherForecast` API が作成されます。 ブラウザーから `Get` メソッドを呼び出して、アプリをテストします。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Ctrl キーを押しながら F5 キーを押して、アプリを実行します。 Visual Studio でブラウザーが起動し、`https://localhost:<port>/WeatherForecast` にアクセスします。ここで、`<port>` はランダムに選択されたポート番号になります。

IIS Express 証明書を信頼するかどうかを確認するダイアログ ボックスが表示された場合は、 **[はい]** を選択します。 次に表示される **[セキュリティ警告]** ダイアログ ボックスで、 **[はい]** を選択します。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Ctrl キーを押しながら F5 キーを押して、アプリを実行します。 ブラウザーで、次の URL に移動します: `https://localhost:5001/WeatherForecast`。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

**[実行]**  >  **[デバッグの開始]** の順に選択してアプリを起動します。 Visual Studio for Mac でブラウザーが起動し、`https://localhost:<port>` にアクセスします。ここで、`<port>` はランダムに選択されたポート番号になります。 HTTP 404 (Not Found) エラーが返されます。 URL に `/WeatherForecast` を追加します (URL を `https://localhost:<port>/WeatherForecast` に変更します)。

---

次のような JSON が返されます。

```json
[
    {
        "date": "2019-07-16T19:04:05.7257911-06:00",
        "temperatureC": 52,
        "temperatureF": 125,
        "summary": "Mild"
    },
    {
        "date": "2019-07-17T19:04:05.7258461-06:00",
        "temperatureC": 36,
        "temperatureF": 96,
        "summary": "Warm"
    },
    {
        "date": "2019-07-18T19:04:05.7258467-06:00",
        "temperatureC": 39,
        "temperatureF": 102,
        "summary": "Cool"
    },
    {
        "date": "2019-07-19T19:04:05.7258471-06:00",
        "temperatureC": 10,
        "temperatureF": 49,
        "summary": "Bracing"
    },
    {
        "date": "2019-07-20T19:04:05.7258474-06:00",
        "temperatureC": -1,
        "temperatureF": 31,
        "summary": "Chilly"
    }
]
```

## <a name="add-a-model-class"></a>モデル クラスの追加

*モデル* は、アプリが管理するデータを表すクラスのセットです。 このアプリのモデルは、単一の `TodoItem` クラスです。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* **ソリューション エクスプローラー** で、プロジェクトを右クリックします。 **[追加]**  >  **[新しいフォルダー]** の順に選択します。 フォルダーに「 *Models* 」という名前を付けます。

* *Models* フォルダーを右クリックして、 **[追加]**  >  **[クラス]** の順に選択します。 クラスに「*TodoItem*」という名前を付け、 **[追加]** を選択します。

* テンプレート コードを次のコードに置き換えます。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* *Models* という名前のフォルダーを追加します。

* 次のコードを使用して、`TodoItem` クラスを *Models* フォルダーに追加します。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* プロジェクトを右クリックします。 **[追加]**  >  **[新しいフォルダー]** の順に選択します。 フォルダーに「 *Models* 」という名前を付けます。

  ![新しいフォルダー](first-web-api-mac/_static/folder.png)

* *Models* フォルダーを右クリックして、 **[追加]** > **[新しいファイル]** > **[全般]** > **[空のクラス]** の順に選択します。

* クラスに「*TodoItem*」という名前を付け、 **[新規]** をクリックします。

* テンプレート コードを次のコードに置き換えます。

---

  [!code-csharp[](first-web-api/samples/3.0/TodoApi/Models/TodoItem.cs?name=snippet)]

`Id` プロパティは、リレーショナル データベース内の一意のキーとして機能します。

モデル クラスはプロジェクト内のどこでも使用できますが、慣例により *Models* フォルダーが使用されます。

## <a name="add-a-database-context"></a>データベース コンテキストの追加

*データベース コンテキスト* は、データ モデルに対して Entity Framework 機能を調整するメイン クラスです。 このクラスは `Microsoft.EntityFrameworkCore.DbContext` クラスから派生させて作成します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

### <a name="add-nuget-packages"></a>NuGet パッケージを追加する

* **[ツール]** メニューで **[NuGet パッケージ マネージャー]、[ソリューションの NuGet パッケージの管理]** の順に選択します。
* **[参照]** タブを選択し、検索ボックスに「**Microsoft.EntityFrameworkCore.InMemory**」と入力します。
* 左側のウィンドウで、 **[Microsoft.EntityFrameworkCore.InMemory]** を選択します。
* 右側のウィンドウで **[プロジェクト]** チェックボックスをオンにして、 **[インストール]** を選択します。

![NuGet パッケージ マネージャー](first-web-api/_static/vs3NuGet.png)

## <a name="add-the-todocontext-database-context"></a>TodoContext データベースコンテキストの追加

* *Models* フォルダーを右クリックして、 **[追加]**  >  **[クラス]** の順に選択します。 クラスに「*TodoContext*」という名前を付け、 **[追加]** をクリックします。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

* *Models* フォルダーに `TodoContext` クラスを追加します。

---

* 次のコードを入力します。

  [!code-csharp[](first-web-api/samples/3.0/TodoApi/Models/TodoContext.cs)]

## <a name="register-the-database-context"></a>データベース コンテキストの登録

ASP.NET Core で、サービス (DB コンテキストなど) を[依存関係の挿入 (DI)](xref:fundamentals/dependency-injection)コンテナーに登録する必要があります。 コンテナーは、コントローラーにサービスを提供します。

次の強調表示されているコードを使用して、*Startup.cs* を更新します。

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Startup.cs?highlight=7-8,23-24&name=snippet_all)]

上記のコードでは次の操作が行われます。

* 不要な `using` 宣言を削除します。
* DI コンテナーにデータベース コンテキストを追加します。
* データベース コンテキストがメモリ内データベースを使用することを指定します。

## <a name="scaffold-a-controller"></a>コントローラーのスキャフォールディング

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* *Controllers* フォルダーを右クリックします。
* **[追加]** > **[スキャフォールディングされた新しい項目]** を選択します。
* **[Entity Framework を使用したアクションがある API コントローラー]** を選択してから、 **[追加]** を選択します。
* **[Entity Framework を使用したアクションがある API コントローラー]** ダイアログで次を実行します。

  * **モデル クラス** で **TodoItem (TodoApi.Models)** を選択します。
  * **データ コンテキスト クラス** で **TodoContext (TodoApi.Models)** を選択します。
  * **[追加]** を選びます。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

次のコマンドを実行します。

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet tool install --global dotnet-aspnet-codegenerator
dotnet tool update -g Dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator controller -name TodoItemsController -async -api -m TodoItem -dc TodoContext -outDir Controllers
```

上のコマンドでは以下の操作が行われます。

* スキャフォールディングに必要な NuGet パッケージを追加します。
* スキャフォールディング エンジン (`dotnet-aspnet-codegenerator`) をインストールします。
* `TodoItemsController` をスキャフォールディングします。

---

生成されたコードでは次の操作が行われます。

* クラスを [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) 属性でマークします。 この属性は、コントローラーが Web API 要求に応答することを示します。 属性によって有効化される特定の動作については、「<xref:web-api/index>」 を参照してください。
* DI を使用して、データベース コンテキスト (`TodoContext`) をコントローラーに挿入します。 データベース コンテキストは、コントローラーの各 [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) メソッドで使用されます。

ASP.NET Core テンプレートの対象は次のとおりです。

* ビューを含むコントローラーには、ルート テンプレートの `[action]` が含まれます。
* API コントローラーには、ルート テンプレートの `[action]` が含まれません。

`[action]` トークンがルート テンプレート内にない場合、[アクション](xref:mvc/controllers/routing#action)名はルートから除外されます。 つまり、アクションの関連付けられたメソッド名は一致するルートでは使用されません。

## <a name="examine-the-posttodoitem-create-method"></a>PostTodoItem 作成メソッドの確認

[nameof](/dotnet/csharp/language-reference/operators/nameof) 演算子を使用するために、`PostTodoItem` で return ステートメントを置き換えます。

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Create)]

[`[HttpPost]`](xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute) 属性が示すように、上記のコードは HTTP POST メソッドです。 このメソッドは、HTTP 要求の本文から To Do アイテムの値を取得します。

詳細については、「[Http[Verb] 属性を使用する属性ルーティング](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes)」を参照してください。

<xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*> メソッド:

* 成功すると、HTTP 201 状態コードが返されます。 HTTP 201 は、サーバーに新しいリソースを作成する HTTP POST メソッドに対する標準の応答です。
* 応答に [Location](https://developer.mozilla.org/docs/Web/HTTP/Headers/Location) ヘッダーが追加されます。 `Location` ヘッダーでは、新しく作成された To Do アイテムの [URI](https://developer.mozilla.org/docs/Glossary/URI) が指定されます。 詳細については、「[10.2.2 201 Created](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)」を参照してください。
* `GetTodoItem` アクションを参照して `Location` ヘッダーの URI を作成します。 C# の `nameof` キーワードを使って、`CreatedAtAction` 呼び出しでアクション名をハードコーディングすることを回避しています。

### <a name="install-postman"></a>Postman のインストール

このチュートリアルでは、Postman を使用して Web API をテストします。

* [Postman](https://www.getpostman.com/downloads/) をインストールします。
* Web アプリを起動します。
* Postman を起動します。
* **[SSL 証明書の確認]** を無効にします。
  * **[ファイル]** > **[設定]** ( **[全般]** タブ) で、 **[SSL 証明書の確認]** を無効にします。
    > [!WARNING]
    > コントローラーをテストした後、SSL 証明書の検証を再度有効にします。

<a name="post"></a>

### <a name="test-posttodoitem-with-postman"></a>Postman を使用した PostTodoItem のテスト

* 新しい要求を作成します。
* HTTP メソッドを `POST` に設定します。
* URI を `https://localhost:<port>/api/TodoItems` に設定します。 たとえば、「 `https://localhost:5001/api/TodoItems` 」のように入力します。
* **[Body]** タブを選択します。
* **[raw]** ラジオ ボタンを選択します。
* 型を **[JSON (application/json)]** に設定します。
* 要求本文に、To Do アイテムの JSON を入力します。

    ```json
    {
      "name":"walk dog",
      "isComplete":true
    }
    ```

* **[Send]** を選択します。

  ![Postman での Create 要求](first-web-api/_static/3/create.png)

### <a name="test-the-location-header-uri-with-postman"></a>Postman で Location ヘッダーの URI をテストする

* **[Response]** ウィンドウで、 **[Headers]** タブを選択します。
* **[Location]** ヘッダー値をコピーします。

  ![Postman コンソールの [Headers] タブ](first-web-api/_static/3/create.png)

* HTTP メソッドを `GET` に設定します。
* URI を `https://localhost:<port>/api/TodoItems/1` に設定します。 たとえば、「 `https://localhost:5001/api/TodoItems/1` 」のように入力します。
* **[Send]** を選択します。

## <a name="examine-the-get-methods"></a>GET メソッドの確認

これらのメソッドは、次の 2 つの GET エンドポイントを実装します。

* `GET /api/TodoItems`
* `GET /api/TodoItems/{id}`

ブラウザーまたは Postman から 2 つのエンドポイントを呼び出すことによって、アプリをテストします。 次に例を示します。

* `https://localhost:5001/api/TodoItems`
* `https://localhost:5001/api/TodoItems/1`

`GetTodoItems` への呼び出しによって、次のような応答が生成されます。

```json
[
  {
    "id": 1,
    "name": "Item1",
    "isComplete": false
  }
]
```

### <a name="test-get-with-postman"></a>Postman を使用して Get をテストする

* 新しい要求を作成します。
* HTTP メソッドを **GET** に設定します。
* 要求 URI を `https://localhost:<port>/api/TodoItems` に設定します。 たとえば、`https://localhost:5001/api/TodoItems` のようにします。
* Postman で **[Two pane view]** を設定します。
* **[Send]** を選択します。

このアプリではメモリ内データベースが使用されます。 アプリが停止して開始された場合、上記の GET 要求はデータを返しません。 データが返されない場合は、アプリにデータを [POST](#post) します。

## <a name="routing-and-url-paths"></a>ルーティングと URL パス

[`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) 属性は、HTTP GET 要求に応答するメソッドを表します。 各メソッドの URL パスは次のように構成されます。

* コントローラーの `Route` 属性でテンプレート文字列を使用します。

  [!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=TodoController&highlight=1)]

* `[controller]` をコントローラーの名前 (慣例では "Controller" サフィックスを除くコントローラー クラス名) に置き換えます。 このサンプルでは、コントローラー クラス名は **TodoItems** Controller なので、コントローラー名は "TodoItems" です。 ASP.NET Core の[ルーティング](xref:mvc/controllers/routing)では、大文字と小文字が区別されません。
* `[HttpGet]` 属性にルート テンプレート (たとえば、`[HttpGet("products")]`) がある場合は、それをパスに追加します。 このサンプルではテンプレートを使用しません。 詳細については、「[Http[Verb] 属性を使用する属性ルーティング](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes)」を参照してください。

次の `GetTodoItem` メソッドで、`"{id}"` は To Do アイテムの一意識別子に使用するプレースホルダーの変数です。 `GetTodoItem` が呼び出されると、その `id` パラメーター内のメソッドに URL の `"{id}"` の値が指定されます。

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_GetByID&highlight=1-2)]

## <a name="return-values"></a>戻り値 

`GetTodoItems` と `GetTodoItem` メソッドの戻り値の型は、[ActionResult\<T> 型](xref:web-api/action-return-types#actionresultt-type)です。 ASP.NET Core は自動的にオブジェクトを [JSON](https://www.json.org/) にシリアル化して、応答メッセージの本文に JSON を書き込みます。 この戻り値の型の応答コードは 200 で、ハンドルされない例外がないものと想定します。 ハンドルされない例外は 5xx エラーに変換されます。

`ActionResult` 戻り値の型は、幅広い範囲の HTTP 状態コードを表すことができます。 たとえば、`GetTodoItem` は、次の 2 つの異なる状態値を返す可能性があります。

* 要求された ID と一致するアイテムがない場合、メソッドは 404 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound%2A> エラー コードを返します。
* それ以外の場合、メソッドは JSON 応答本文で 200 を返します。 戻り値が `item` の場合、HTTP 200 応答が返されます。

## <a name="the-puttodoitem-method"></a>PutTodoItem メソッド

`PutTodoItem` メソッドを検証します。

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Update)]

`PutTodoItem` は `PostTodoItem` と似ていますが、HTTP PUT を使用します。 応答は [204 (No Content)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) となります。 HTTP 仕様に従って、PUT 要求では、変更だけでなく、更新されたエンティティ全体を送信するようクライアントに求めます。 部分的な更新をサポートするには、[HTTP PATCH](xref:Microsoft.AspNetCore.Mvc.HttpPatchAttribute) を使用します。

`PutTodoItem` 呼び出しでエラーが発生した場合、`GET` を呼び出してデータベース内にアイテムがあることを確認してください。

### <a name="test-the-puttodoitem-method"></a>PutTodoItem メソッドのテスト

このサンプルでは、アプリを起動するたびに開始することが必要なメモリ内データベースが使われています。 PUT 呼び出しを実行する前に、データベース内にアイテムが存在している必要があります。 GET を呼び出して、PUT 呼び出しを実行する前にデータベース内にアイテムが確実に存在していることを確認します。

Id = 1 の To Do アイテムを更新し、その名前を "feed fish" に設定します。

```json
  {
    "id":1,
    "name":"feed fish",
    "isComplete":true
  }
```

次の図は、Postman の更新を示しています。

![204 (No Content) の応答を示す Postman コンソール](first-web-api/_static/3/pmcput.png)

## <a name="the-deletetodoitem-method"></a>DeleteTodoItem メソッド

`DeleteTodoItem` メソッドを検証します。

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Delete)]

### <a name="test-the-deletetodoitem-method"></a>DeleteTodoItem メソッドのテスト

Postman を使用して、To Do アイテムを削除します。

* メソッドを `DELETE` に設定します。
* 削除するオブジェクトの URI (たとえば、`https://localhost:5001/api/TodoItems/1`) を設定します。
* **[Send]** を選択します。

<a name="over-post"></a>
<a name="over-post-v3"></a>

## <a name="prevent-over-posting"></a>過剰な投稿を防止する

現在、サンプル アプリでは `TodoItem` オブジェクト全体が公開されています。 通常、運用環境のアプリでは、モデルのサブセットを使用して入力されるデータおよび返されるデータが制限されています。 その背景には複数の理由があり、セキュリティは主なものです。 モデルのサブセットは、通常、データ転送オブジェクト (DTO)、入力モデル、またはビュー モデルと呼ばれます。 この記事では **DTO** を使用しています。

DTO は次の目的で使用できます。

* 過剰な投稿を防止する。
* クライアントが表示しないことになっているプロパティを非表示にする。
* ペイロード サイズを減らすために、いくつかのプロパティを省略する。
* 入れ子になったオブジェクトを含むオブジェクト グラフをフラット化する。 フラット化されたオブジェクト グラフは、クライアントにとってより便利になる可能性があります。

DTO のアプローチを実演するために、`TodoItem` クラスを更新して、シークレット フィールドを含めます。

[!code-csharp[](first-web-api/samples/3.0/TodoApiDTO/Models/TodoItem.cs?name=snippet&highlight=6)]

シークレット フィールドは、このアプリでは非表示にする必要がありますが、管理アプリの場合は公開することを選択できます。

シークレット フィールドを投稿および取得できることを確認します。

次のように DTO モデルを作成します。

[!code-csharp[](first-web-api/samples/3.0/TodoApiDTO/Models/TodoItemDTO.cs?name=snippet)]

`TodoItemDTO` を使用するように `TodoItemsController` を更新します。

[!code-csharp[](first-web-api/samples/3.0/TodoApiDTO/Controllers/TodoItemsController.cs?name=snippet)]

シークレット フィールドを投稿または取得できないことを確認します。

## <a name="call-the-web-api-with-javascript"></a>JavaScript を使用した Web API の呼び出し

「[チュートリアル:JavaScript を使用して ASP.NET Core Web API を呼び出す](xref:tutorials/web-api-javascript)」を参照してください。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

このチュートリアルでは、次の作業を行う方法について説明します。

> [!div class="checklist"]
> * Web API プロジェクトを作成する。
> * モデル クラスとデータベース コンテキストを追加する。
> * コントローラーを追加する。
> * CRUD メソッドを追加する。
> * ルーティングと URL パスを構成する。
> * 戻り値を指定する。
> * Postman で Web API を呼び出す。
> * JavaScript を使用した Web API の呼び出し。

最後に、リレーショナル データベースに格納されている "To Do" アイテムを管理できる Web API が作成されます。

## <a name="overview-21"></a>概要 2.1

このチュートリアルでは、次の API を作成します。

|API | 説明 | 要求本文 | 応答本文 |
|--- | ---- | ---- | ---- |
|GET /api/TodoItems | すべての To Do アイテムを取得します。 | None | To Do アイテムの配列|
|GET /api/TodoItems/{id} | ID でアイテムを取得します。 | None | To Do アイテム|
|POST /api/TodoItems | 新しいアイテムを追加します。 | To Do アイテム | To Do アイテム |
|PUT /api/TodoItems/{id} | 既存のアイテムを更新します。&nbsp; | To Do アイテム | None |
|DELETE /api/TodoItems/{id} &nbsp; &nbsp; | アイテムを削除します &nbsp; &nbsp; | None | None|

次の図は、アプリのデザインを示しています。

![クライアントは、左側のボックスに表されます。 要求を送信し、アプリケーション (右側に描画されたボックス) から応答を受信します。 アプリケーション ボックス内の 3 つのボックスは、コントローラー、モデル、およびデータ アクセス レイヤーを表しています。 要求はアプリケーションのコントローラーに送られ、コントローラーとデータ アクセス レイヤー間で読み取り/書き込み操作が行われます。 モデルはシリアル化され、応答でクライアントに返されます。](first-web-api/_static/architecture.png)

## <a name="prerequisites-21"></a>前提条件 2.1

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs2019-2.2.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-2.2.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-2.2.md)]

---

## <a name="create-a-web-project-21"></a>Web プロジェクト 2.1 の作成

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* **[ファイル]** メニューで、 **[新規作成]** > **[プロジェクト]** の順に選択します。
* **[ASP.NET Core Web アプリケーション]** テンプレートを選択して、 **[次へ]** をクリックします。
* プロジェクトに「*TodoApi*」という名前を付け、 **[作成]** をクリックします。
* **[新しい ASP.NET Core Web アプリケーションを作成する]** ダイアログで、 **[.NET Core]** と **[ASP.NET Core 2.2]** が選択されていることを確認します。 **API** テンプレートを選択し、 **[作成]** をクリックします。 **[Enable Docker Support]\(Docker サポートを有効にする\)** は **選択しないで** ください。

![VS の [新しいプロジェクト] ダイアログ](first-web-api/_static/vs.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* [統合ターミナル](https://code.visualstudio.com/docs/editor/integrated-terminal)を開きます。
* ディレクトリ (`cd`) を、プロジェクト フォルダーを格納するフォルダーに変更します。
* 次のコマンドを実行します。

   ```dotnetcli
   dotnet new webapi -o TodoApi
   code -r TodoApi
   ```

  これらのコマンドでは、新しい Web API プロジェクトが作成され、新しいプロジェクト フォルダー内に Visual Studio Code の新しいインスタンスが開かれます。

* ダイアログ ボックスで、プロジェクトに必要な資産を追加するかどうかを確認されたら、 **[はい]** を選択します。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* **[ファイル]** > **[新しいソリューション]** の順に選択します。

  ![macOS の新しいソリューション](first-web-api-mac/_static/sln.png)

* バージョン 8.6 より前の Visual Studio for Mac では、 **[.NET Core]**  >  **[アプリ]**  >  **[API]**  >  **[次へ]** の順に選択します。 バージョン 8.6 以降では、 **[Web and Console]** \(Web とコンソール\) >  **[アプリ]**  >  **[API]**  >  **[次へ]** の順に選択します。
  
* **[Configure the new ASP.NET Core Web API]\(新しい ASP.NET Core Web API を構成する\)** ダイアログで、最新の .NET Core 2.x **ターゲット フレームワーク** を選択します。 **[次へ]** を選択します。

* **[プロジェクト名]** に「*TodoApi*」と入力し、 **[作成]** を選択します。

  ![構成ダイアログ](first-web-api-mac/_static/2.png)

---

### <a name="test-the-api-21"></a>API 2.1 のテスト

プロジェクト テンプレートによって `values` API が作成されます。 ブラウザーから `Get` メソッドを呼び出して、アプリをテストします。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Ctrl キーを押しながら F5 キーを押して、アプリを実行します。 Visual Studio でブラウザーが起動し、`https://localhost:<port>/api/values` にアクセスします。ここで、`<port>` はランダムに選択されたポート番号になります。

IIS Express 証明書を信頼するかどうかを確認するダイアログ ボックスが表示された場合は、 **[はい]** を選択します。 次に表示される **[セキュリティ警告]** ダイアログ ボックスで、 **[はい]** を選択します。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Ctrl キーを押しながら F5 キーを押して、アプリを実行します。 ブラウザーで、次の URL に移動します: `https://localhost:5001/api/values`。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

**[実行]**  >  **[デバッグの開始]** の順に選択してアプリを起動します。 Visual Studio for Mac でブラウザーが起動し、`https://localhost:<port>` にアクセスします。ここで、`<port>` はランダムに選択されたポート番号になります。 HTTP 404 (Not Found) エラーが返されます。 URL に `/api/values` を追加します (URL を `https://localhost:<port>/api/values` に変更します)。

---

次の JSON が返されます。

```json
["value1","value2"]
```

## <a name="add-a-model-class-21"></a>モデル クラス 2.1 の追加

*モデル* は、アプリが管理するデータを表すクラスのセットです。 このアプリのモデルは、単一の `TodoItem` クラスです。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* **ソリューション エクスプローラー** で、プロジェクトを右クリックします。 **[追加]**  >  **[新しいフォルダー]** の順に選択します。 フォルダーに「 *Models* 」という名前を付けます。

* *Models* フォルダーを右クリックして、 **[追加]**  >  **[クラス]** の順に選択します。 クラスに「*TodoItem*」という名前を付け、 **[追加]** を選択します。

* テンプレート コードを次のコードに置き換えます。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* *Models* という名前のフォルダーを追加します。

* 次のコードを使用して、`TodoItem` クラスを *Models* フォルダーに追加します。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* プロジェクトを右クリックします。 **[追加]**  >  **[新しいフォルダー]** の順に選択します。 フォルダーに「 *Models* 」という名前を付けます。

  ![新しいフォルダー](first-web-api-mac/_static/folder.png)

* *Models* フォルダーを右クリックして、 **[追加]** > **[新しいファイル]** > **[全般]** > **[空のクラス]** の順に選択します。

* クラスに「*TodoItem*」という名前を付け、 **[新規]** をクリックします。

* テンプレート コードを次のコードに置き換えます。

---

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Models/TodoItem.cs)]

`Id` プロパティは、リレーショナル データベース内の一意のキーとして機能します。

モデル クラスはプロジェクト内のどこでも使用できますが、慣例により *Models* フォルダーが使用されます。

## <a name="add-a-database-context-21"></a>データベース コンテキスト 2.1 の追加

*データベース コンテキスト* は、データ モデルに対して Entity Framework 機能を調整するメイン クラスです。 このクラスは `Microsoft.EntityFrameworkCore.DbContext` クラスから派生させて作成します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* *Models* フォルダーを右クリックして、 **[追加]**  >  **[クラス]** の順に選択します。 クラスに「*TodoContext*」という名前を付け、 **[追加]** をクリックします。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

* *Models* フォルダーに `TodoContext` クラスを追加します。

---

* テンプレート コードを次のコードに置き換えます。

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Models/TodoContext.cs)]

## <a name="register-the-database-context-21"></a>データベース コンテキスト 2.1 の登録

ASP.NET Core で、サービス (DB コンテキストなど) を[依存関係の挿入 (DI)](xref:fundamentals/dependency-injection)コンテナーに登録する必要があります。 コンテナーは、コントローラーにサービスを提供します。

次の強調表示されているコードを使用して、*Startup.cs* を更新します。

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Startup1.cs?highlight=5,8,25-26&name=snippet_all)]

上記のコードでは次の操作が行われます。

* 不要な `using` 宣言を削除します。
* DI コンテナーにデータベース コンテキストを追加します。
* データベース コンテキストがメモリ内データベースを使用することを指定します。

## <a name="add-a-controller-21"></a>コントローラー 2.1 の追加

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* *Controllers* フォルダーを右クリックします。
* **[追加]** > **[新しい項目]** の順に選択します。
* **[新しい項目の追加]** ダイアログで、 **[API コントローラー クラス]** テンプレートを選択します。
* クラスに「*TodoController*」という名前を付け、 **[追加]** を選択します。

  ![[新しい項目の追加] ダイアログ。検索ボックスに「controller」と入力されています。Web API コントローラーが選択されています。](first-web-api/_static/new_controller.png)

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

* *Controllers* フォルダーで、「`TodoController`」という名前のクラスを作成します。

---

* テンプレート コードを次のコードに置き換えます。

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController2.cs?name=snippet_todo1)]

上記のコードでは次の操作が行われます。

* メソッドを使用せず、API コントローラー クラスを定義します。
* クラスを [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) 属性でマークします。 この属性は、コントローラーが Web API 要求に応答することを示します。 属性によって有効化される特定の動作については、「<xref:web-api/index>」 を参照してください。
* DI を使用して、データベース コンテキスト (`TodoContext`) をコントローラーに挿入します。 データベース コンテキストは、コントローラーの各 [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) メソッドで使用されます。
* 追加データベースが空の場合、`Item1` という名前のアイテムをデータベースにします。 このコードはコンストラクター内にあるので、新しい HTTP 要求が行われるたびに実行されます。 すべてのアイテムを削除した場合、コンストラクターは、次回に API メソッドが呼び出されたときに `Item1` をもう一度作成します。 そのため、削除が実際には機能していても、機能しなかったように見える場合があります。

## <a name="add-get-methods-21"></a>Get メソッド 2.1 の追加

To Do アイテムを取得する API を指定するには、`TodoController` クラスに次のメソッドを追加します。

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_GetAll)]

これらのメソッドは、次の 2 つの GET エンドポイントを実装します。

* `GET /api/todo`
* `GET /api/todo/{id}`

アプリがまだ実行中の場合は、停止します。 次に、それを再度実行して、最新の変更を含めます。

ブラウザーからこれらの 2 つのエンドポイントを呼び出すことによって、アプリをテストします。 次に例を示します。

* `https://localhost:<port>/api/todo`
* `https://localhost:<port>/api/todo/1`

`GetTodoItems` への呼び出しによって、次の HTTP 応答が生成されます。

```json
[
  {
    "id": 1,
    "name": "Item1",
    "isComplete": false
  }
]
```

## <a name="routing-and-url-paths-21"></a>ルーティングと URL パス 2.1

[`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) 属性は、HTTP GET 要求に応答するメソッドを表します。 各メソッドの URL パスは次のように構成されます。

* コントローラーの `Route` 属性でテンプレート文字列を使用します。

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=TodoController&highlight=3)]

* `[controller]` をコントローラーの名前 (慣例では "Controller" サフィックスを除くコントローラー クラス名) に置き換えます。 このサンプルでは、コントローラー クラス名は **Todo** Controller なので、コントローラー名は "todo" です。 ASP.NET Core の[ルーティング](xref:mvc/controllers/routing)では、大文字と小文字が区別されません。
* `[HttpGet]` 属性にルート テンプレート (たとえば、`[HttpGet("products")]`) がある場合は、それをパスに追加します。 このサンプルではテンプレートを使用しません。 詳細については、「[Http[Verb] 属性を使用する属性ルーティング](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes)」を参照してください。

次の `GetTodoItem` メソッドで、`"{id}"` は To Do アイテムの一意識別子に使用するプレースホルダーの変数です。 `GetTodoItem` が呼び出されると、その `id` パラメーター内のメソッドに URL の `"{id}"` の値が指定されます。

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_GetByID&highlight=1-2)]

## <a name="return-values-21"></a>戻り値 2.1

`GetTodoItems` と `GetTodoItem` メソッドの戻り値の型は、[ActionResult\<T> 型](xref:web-api/action-return-types#actionresultt-type)です。 ASP.NET Core は自動的にオブジェクトを [JSON](https://www.json.org/) にシリアル化して、応答メッセージの本文に JSON を書き込みます。 この戻り値の型の応答コードは 200 で、ハンドルされない例外がないものと想定します。 ハンドルされない例外は 5xx エラーに変換されます。

`ActionResult` 戻り値の型は、幅広い範囲の HTTP 状態コードを表すことができます。 たとえば、`GetTodoItem` は、次の 2 つの異なる状態値を返す可能性があります。

* 要求された ID と一致するアイテムがない場合、メソッドは 404 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound%2A> エラー コードを返します。
* それ以外の場合、メソッドは JSON 応答本文で 200 を返します。 戻り値が `item` の場合、HTTP 200 応答が返されます。

## <a name="test-the-gettodoitems-method-21"></a>GetTodoItems メソッド 2.1 のテスト

このチュートリアルでは、Postman を使用して Web API をテストします。

* [Postman](https://www.getpostman.com/downloads/) をインストールします。
* Web アプリを起動します。
* Postman を起動します。
* **[SSL 証明書の確認]** を無効にします。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* **[ファイル]** > **[設定]** ( **[全般]** タブ) で、 **[SSL 証明書の確認]** を無効にします。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

* **[Postman]**  >  **[ユーザー設定]** ( **[全般]** タブ) で、**SSL 証明書の検証** を無効にします。 あるいは、レンチを選択し、 **[設定]** を選択して、SSL 証明書の検証を無効にします。

---
  
> [!WARNING]
> コントローラーをテストした後、SSL 証明書の検証を再度有効にします。

* 新しい要求を作成します。
  * HTTP メソッドを **GET** に設定します。
  * 要求 URI を `https://localhost:<port>/api/todo` に設定します。 たとえば、`https://localhost:5001/api/todo` のようにします。
* Postman で **[Two pane view]** を設定します。
* **[Send]** を選択します。

![Postman での Get 要求](first-web-api/_static/2pv.png)

## <a name="add-a-create-method-21"></a>Create メソッド 2.1 の追加

*Controllers/TodoController.cs* 内に次の `PostTodoItem` メソッドを追加します。 

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_Create)]

[`[HttpPost]`](xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute) 属性が示すように、上記のコードは HTTP POST メソッドです。 このメソッドは、HTTP 要求の本文から To Do アイテムの値を取得します。

`CreatedAtAction` メソッド:

* 成功すると、HTTP 201 状態コードが返されます。 HTTP 201 は、サーバーに新しいリソースを作成する HTTP POST メソッドに対する標準の応答です。
* `Location` ヘッダーを応答に追加します。 `Location` ヘッダーでは、新しく作成された To Do アイテムの URI を指定します。 詳細については、「[10.2.2 201 Created](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)」を参照してください。
* `GetTodoItem` アクションを参照して `Location` ヘッダーの URI を作成します。 C# の `nameof` キーワードを使って、`CreatedAtAction` 呼び出しでアクション名をハードコーディングすることを回避しています。

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_GetByID&highlight=1-2)]

### <a name="test-the-posttodoitem-method-21"></a>PostTodoItem メソッド 2.1 のテスト

* プロジェクトをビルドします。
* Postman で、HTTP メソッド名を `POST` に設定します。
* URI を `https://localhost:<port>/api/Todo` に設定します。 たとえば、「 `https://localhost:5001/api/Todo` 」のように入力します。
* **[Body]** タブを選択します。
* **[raw]** ラジオ ボタンを選択します。
* 型を **[JSON (application/json)]** に設定します。
* 要求本文に、To Do アイテムの JSON を入力します。

    ```json
    {
      "name":"walk dog",
      "isComplete":true
    }
    ```

* **[Send]** を選択します。

  ![Postman での Create 要求](first-web-api/_static/create.png)

  405 (Method Not Allowed) エラーが発生した場合は、`PostTodoItem` メソッドの追加後にプロジェクトをコンパイルしていないことが原因である可能性があります。

### <a name="test-the-location-header-uri-21"></a>場所ヘッダー URI 2.1 のテスト

* **[Response]** ウィンドウで、 **[Headers]** タブを選択します。
* **[Location]** ヘッダー値をコピーします。

  ![Postman コンソールの [Headers] タブ](first-web-api/_static/pmc2.png)

* メソッドを GET に設定します。
* URI を `https://localhost:<port>/api/TodoItems/2` に設定します。 たとえば、「 `https://localhost:5001/api/TodoItems/2` 」のように入力します。
* **[Send]** を選択します。

## <a name="add-a-puttodoitem-method-21"></a>PutTodoItem メソッド 2.1 の追加

次の `PutTodoItem` メソッドを追加します。

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_Update)]

`PutTodoItem` は `PostTodoItem` と似ていますが、HTTP PUT を使用します。 応答は [204 (No Content)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) となります。 HTTP 仕様に従って、PUT 要求では、変更だけでなく、更新されたエンティティ全体を送信するようクライアントに求めます。 部分的な更新をサポートするには、[HTTP PATCH](xref:Microsoft.AspNetCore.Mvc.HttpPatchAttribute) を使用します。

`PutTodoItem` 呼び出しでエラーが発生した場合、`GET` を呼び出してデータベース内にアイテムがあることを確認してください。

### <a name="test-the-puttodoitem-method-21"></a>PutTodoItem メソッド 2.1 のテスト

このサンプルでは、アプリを起動するたびに開始することが必要なメモリ内データベースが使われています。 PUT 呼び出しを実行する前に、データベース内にアイテムが存在している必要があります。 GET を呼び出して、PUT 呼び出しを実行する前にデータベース内にアイテムが確実に存在していることを確認します。

Id = 1 の To Do アイテムを更新し、その名前を "feed fish" に設定します。

```json
  {
    "id":1,
    "name":"feed fish",
    "isComplete":true
  }
```

次の図は、Postman の更新を示しています。

![204 (No Content) の応答を示す Postman コンソール](first-web-api/_static/pmcput.png)

## <a name="add-a-deletetodoitem-method-21"></a>DeleteTodoItem メソッド 2.1 の追加

次の `DeleteTodoItem` メソッドを追加します。

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_Delete)]

`DeleteTodoItem` の応答は [204 (No Content)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) となります。

### <a name="test-the-deletetodoitem-method-21"></a>DeleteTodoItem メソッド 2.1 のテスト

Postman を使用して、To Do アイテムを削除します。

* メソッドを `DELETE` に設定します。
* 削除するオブジェクトの URI (たとえば、`https://localhost:5001/api/todo/1`) を設定します。
* **[Send]** を選択します。

サンプル アプリではすべてのアイテムを削除することができます。 ただし、最後のアイテムが削除されると、次回 API が呼び出されたときに、モデル クラス コンストラクターによって新しいアイテムが作成されます。

## <a name="call-the-web-api-with-javascript-21"></a>JavaScript 2.1 を使用した Web API の呼び出し

このセクションでは、JavaScript を使用して Web API を呼び出す HTML ページを追加します。 jQuery によって要求が開始されます。 JavaScript により、Web API の応答からの詳細を使ってページが更新されます。

*Startup.cs* を次の強調表示されたコードで更新して、[静的ファイルを提供](xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A)し、[既定のファイル マッピングを有効にする](xref:Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles%2A)ためのアプリを構成します。

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Startup.cs?highlight=14-15&name=snippet_configure)]

プロジェクト ディレクトリで *wwwroot* フォルダーを作成します。

*index.html* という名前の HTML ファイルを *wwwroot* ディレクトリに追加します。 その内容を次のマークアップに置き換えます。

[!code-html[](first-web-api/samples/2.2/TodoApi/wwwroot/index.html)]

*site.js* という名前の JavaScript ファイルを *wwwroot* ディレクトリに追加します。 その内容を次のコードに置き換えます。

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_SiteJs)]

ローカルで HTML ページをテストするために、ASP.NET Core プロジェクトの起動設定への変更が要求される場合があります。

* *Properties\launchSettings.json* を開きます。
* アプリが強制的に *index.html*&mdash;プロジェクトの既定ファイルで開くようにするには、`launchUrl` プロパティを削除します。

このサンプルでは、Web API のすべての CRUD メソッドを呼び出します。 次に、API の呼び出しについて説明します。

### <a name="get-a-list-of-to-do-items-21"></a>To Do アイテム 2.1 のリストの取得

jQuery により HTTP GET 要求が Web API に送信され、API からは To Do アイテムの配列を表す JSON が返されます。 要求が成功した場合、`success` コールバック関数が呼び出されます。 コールバックでは、DOM は To Do 情報で更新されます。

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_GetData)]

### <a name="add-a-to-do-item-21"></a>To Do アイテム 2.1 の追加

jQuery により、要求本文に To Do アイテムが含まれる HTTP POST 要求が送信されます。 `accepts` オプションと `contentType` オプションは `application/json` に設定されて、送受信されるメディアの種類を指定します。 To Do アイテムは、[JSON.stringify](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) を使用して JSON に変換されます。 API で正常な状況コードが返された場合、`getData` 関数が呼び出され、HTML テーブルを更新します。

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_AddItem)]

### <a name="update-a-to-do-item-21"></a>To Do アイテム 2.1 の更新

To Do アイテムの更新は、追加操作に似ています。 アイテムの一意の識別子を追加するように `url` が変更され、`type` は `PUT` となります。

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_AjaxPut)]

### <a name="delete-a-to-do-item-21"></a>To Do アイテム 2.1 の削除

To Do アイテムを削除するには、`DELETE` への AJAX 呼び出しで `type` を設定して、URL でアイテムの一意の識別子を指定します。

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_AjaxDelete)]

::: moniker-end

<a name="auth"></a>

## <a name="add-authentication-support-to-a-web-api-21"></a>Web API 2.1 への認証サポートの追加

[!INCLUDE[](~/includes/IdentityServer4.md)]

## <a name="additional-resources-21"></a>その他のリソース 2.1

[このチュートリアルのサンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/first-web-api/samples)します。 [ダウンロード方法](xref:index#how-to-download-a-sample)に関するページを参照してください。

詳細については、次のリソースを参照してください。

* <xref:web-api/index>
* <xref:tutorials/web-api-help-pages-using-swagger>
* <xref:data/ef-rp/intro>
* <xref:mvc/controllers/routing>
* <xref:web-api/action-return-types>
* <xref:host-and-deploy/azure-apps/index>
* <xref:host-and-deploy/index>
* [このチュートリアルの YouTube バージョン](https://www.youtube.com/watch?v=TTkhEyGBfAk)
