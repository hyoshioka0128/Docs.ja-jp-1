---
title: TypeScript と Webpack で ASP.NET Core SignalR を使用する
author: ssougnez
description: このチュートリアルでは、クライアントが TypeScript で記述された ASP.NET Core SignalR Web アプリをバンドルおよびビルドするために Webpack を構成します。
ms.author: bradyg
ms.custom: mvc
ms.date: 02/10/2020
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
uid: tutorials/signalr-typescript-webpack
ms.openlocfilehash: 9d52b72c669a9345cc7c5386876db22af55a97c7
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589752"
---
# <a name="use-aspnet-core-signalr-with-typescript-and-webpack"></a>TypeScript と Webpack で ASP.NET Core SignalR を使用する

作成者: [Sébastien Sougnez](https://twitter.com/ssougnez)、[Scott Addie](https://twitter.com/Scott_Addie)

[Webpack](https://webpack.js.org/) を使用すると、開発者は Web アプリのクライアント側のリソースをバンドルおよびビルドすることができます。 このチュートリアルでは、クライアントが [TypeScript](https://www.typescriptlang.org/) で記述された ASP.NET Core SignalR Web アプリでの Webpack の使用法を示します。

このチュートリアルでは、次の作業を行う方法について説明します。

> [!div class="checklist"]
> * スターター ASP.NET Core SignalR アプリをスキャフォールディングする
> * SignalR TypeScript クライアントを構成する
> * Webpack を使用してビルド パイプラインを構成する
> * SignalR サーバーを構成する
> * クライアントとサーバー間の通信を有効にする

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/signalr-typescript-webpack/sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

::: moniker range=">= aspnetcore-3.0"

## <a name="prerequisites"></a>必須コンポーネント

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) と **ASP.NET と Web 開発** ワークロード
* [.NET Core SDK 3.0 以降](https://dotnet.microsoft.com/download/dotnet-core)
* [Node.js](https://nodejs.org/) ([npm](https://www.npmjs.com/) 使用)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* [Visual Studio Code](https://code.visualstudio.com/download)
* [.NET Core SDK 3.0 以降](https://dotnet.microsoft.com/download/dotnet-core)
* [C# for Visual Studio Code バージョン 1.17.1 またはそれ以降](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* [Node.js](https://nodejs.org/) ([npm](https://www.npmjs.com/) 使用)

---

## <a name="create-the-aspnet-core-web-app"></a>ASP.NET Core Web アプリを作成する

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

*PATH* 環境変数で npm を検索するように Visual Studio を構成します。 既定では、Visual Studio は、そのインストール ディレクトリ内で見つかった npm のバージョンを使用します。 Visual Studio のこれらの説明に従ってください。

1. Visual Studio を起動します。 スタート ウィンドウで、 **[コードなしで続行]** を選択します。
1. **[ツール]** > **[オプション]** > **[プロジェクトとソリューション]** > **[Web パッケージ管理]** > **[外部 Web ツール]** に移動します。
1. リストから *[$(PATH)]* エントリを選択します。 上向き矢印をクリックして、エントリをリストの 2 番目の位置に移動し、 **[OK]** を選択します。

    ![Visual Studio の構成](signalr-typescript-webpack/_static/signalr-configure-path-visual-studio.png)

Visual Studio の構成が完了しました。

1. **[ファイル]**  >  **[新規]**  >  **[プロジェクト]** メニュー オプションを使用して、 **[ASP.NET Core Web アプリケーション]** テンプレートを選択します。 **[次へ]** を選択します。
1. プロジェクトに *SignalRWebPack* という名前を付け、 **[作成]** を選択します。
1. ターゲット フレームワークのドロップダウンから *[.NET Core]* を選択し、フレームワーク セレクターのドロップダウンから *[ASP.NET Core 3.1]* を選択します。 **空** のテンプレートを選択して、 **[作成]** を選択します。

`Microsoft.TypeScript.MSBuild` パッケージをプロジェクトに追加します。

1. **ソリューション エクスプローラー** (右ペイン) でプロジェクト ノードを右クリックし、 **[NuGet パッケージの管理]** を選択します。 **[参照]** タブで `Microsoft.TypeScript.MSBuild` を検索し、右側にある **[インストール]** をクリックしてパッケージをインストールします。

Visual Studio によって **ソリューション エクスプローラー** の **[依存関係]** ノードの下に NuGet パッケージが追加され、プロジェクトで TypeScript のコンパイルができるようになります。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

**統合ターミナル** で次のコマンドを実行します。

```dotnetcli
dotnet new web -o SignalRWebPack
code -r SignalRWebPack
```

* `dotnet new` コマンドによって、空の ASP.NET Core Web アプリが *SignalRWebPack* ディレクトリ内に作成されます。
* `code` コマンドは、Visual Studio Code の現在のインスタンス内で *SignalRWebPack* フォルダーを開きます。

**統合ターミナル** で次の .NET Core CLI コマンドを実行します。

```dotnetcli
dotnet add package Microsoft.TypeScript.MSBuild
```

上記のコマンドにより、[Microsoft.TypeScript.MSBuild](https://www.nuget.org/packages/Microsoft.TypeScript.MSBuild/) パッケージが追加され、プロジェクトでの TypeScript コンパイルが可能になります。

---

## <a name="configure-webpack-and-typescript"></a>WebPack および TypeScript の構成

次の手順では、TypeScript の JavaScript への変換と、クライアント側のリソースのバンドルを構成します。

1. プロジェクト ルートで次のコマンドを実行して、"*package.json*" ファイルを作成します。

    ```console
    npm init -y
    ```

1. 強調表示されているプロパティを "*package.json*" ファイルに追加し、ファイルの変更を保存します。

    [!code-json[package.json](signalr-typescript-webpack/sample/3.x/snippets/package1.json?highlight=4)]

    `private` プロパティを `true` に設定して、次の手順でパッケージのインストールの警告が表示されないようにします。

1. 必要な npm パッケージをインストールします。 プロジェクト ルートで、次のコマンドを実行します。

    ```console
    npm i -D -E clean-webpack-plugin@3.0.0 css-loader@3.4.2 html-webpack-plugin@3.2.0 mini-css-extract-plugin@0.9.0 ts-loader@6.2.1 typescript@3.7.5 webpack@4.41.5 webpack-cli@3.3.10
    ```

    注目するべきコマンドの詳細:

    * バージョン番号は、各パッケージ名の `@` 記号の後に続きます。 npm によって、これらの特定のパッケージ バージョンがインストールされます。
    * `-E` オプションは、[セマンティック バージョニング](https://semver.org/)範囲演算子を *package.json* に書き込む npm の既定の動作を無効にします。 たとえば、`"webpack": "4.41.5"` が `"webpack": "^4.41.5"` の代わりに使用されています。 このオプションにより、新しいパッケージ バージョンへの予期しないアップグレードが防止されます。

    詳細については、[npm-install](https://docs.npmjs.com/cli/install) のドキュメントを参照してください。

1. "*package.json*" ファイルの `scripts` プロパティを次のコードで置き換えます。

    ```json
    "scripts": {
      "build": "webpack --mode=development --watch",
      "release": "webpack --mode=production",
      "publish": "npm run release && dotnet publish -c Release"
    },
    ```

    スクリプトの説明 (一部):

    * `build`:開発モードでクライアント側のリソースをバンドルし、ファイルの変更を監視します。 ファイル監視により、プロジェクト ファイルを変更するたびにバンドルが再生成されます。 `mode` オプションは、ツリー シェイキングや縮小などの運用環境の最適化を無効にします。 `build` は開発でのみ使用します。
    * `release`:運用モードでクライアント側のリソースをバンドルします。
    * `publish`:`release` スクリプトを実行して、運用モードでクライアント側のリソースをバンドルします。 .NET Core CLI の [publish](/dotnet/core/tools/dotnet-publish) コマンドを呼び出してアプリを公開します。

1. プロジェクト ルートに、次のコードを含む "*webpack.config.js*" という名前のファイルを作成します。

    [!code-javascript[webpack.config.js](signalr-typescript-webpack/sample/3.x/webpack.config.js)]

    上記のファイルは、Webpack コンパイルを構成します。 注目するべき構成の詳細 (一部):

    * `output` プロパティにより、*dist* の既定値がオーバーライドされます。 代わりにバンドルが *wwwroot* ディレクトリ内に生成されます。
    * `resolve.extensions` 配列には、SignalR クライアント JavaScript をインポートするための *.js* が含まれています。

1. プロジェクトのクライアント側アセットを格納するために、プロジェクト ルートに新しい "*src*" ディレクトリを作成します。

1. 次のマークアップを含む "*src/index.html*" を作成します。

    [!code-html[index.html](signalr-typescript-webpack/sample/3.x/src/index.html)]

    上記の HTML では、ホームページの定型マークアップが定義されます。

1. 新しい *src/css* ディレクトリを作成します。 その目的は、プロジェクトの *.css* ファイルを格納することです。

1. 次の CSS を含む "*src/css/main.css*" を作成します。

    [!code-css[main.css](signalr-typescript-webpack/sample/3.x/src/css/main.css)]

    上記の *main.css* ファイルは、アプリをスタイル設定します。

1. 次の JSON を含む "*src/tsconfig.json*" を作成します。

    [!code-json[tsconfig.json](signalr-typescript-webpack/sample/3.x/src/tsconfig.json)]

    上記のコードは、[ECMAScript](https://wikipedia.org/wiki/ECMAScript) 5 対応の JavaScript を生成するように TypeScript コンパイラを構成します。

1. 次のコードを含む "*src/index.ts*" を作成します。

    [!code-typescript[index.ts](signalr-typescript-webpack/sample/3.x/snippets/index1.ts?name=snippet_IndexTsPhase1File)]

    上記の TypeScript は、DOM 要素への参照を取得し、次の 2 つのイベント ハンドラーをアタッチします。

    * `keyup`:ユーザーが `tbMessage` テキスト ボックスに入力すると、このイベントが発生します。 ユーザーが **Enter** キーを押すと、`send` 関数が呼び出されます。
    * `click`: ユーザーが **[送信]** ボタンをクリックすると、このイベントが発生します。 `send` 関数が呼び出されます。

## <a name="configure-the-app"></a>Configure the app

1. `Startup.Configure` で、[UseDefaultFiles](/dotnet/api/microsoft.aspnetcore.builder.defaultfilesextensions.usedefaultfiles#Microsoft_AspNetCore_Builder_DefaultFilesExtensions_UseDefaultFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_) および [UseStaticFiles](/dotnet/api/microsoft.aspnetcore.builder.staticfileextensions.usestaticfiles#Microsoft_AspNetCore_Builder_StaticFileExtensions_UseStaticFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_) の呼び出しを追加します。

   [!code-csharp[Startup](signalr-typescript-webpack/sample/3.x/Startup.cs?name=snippet_UseStaticDefaultFiles&highlight=9-10)]

   上記のコードを使用すると、サーバーで "*index.html*" ファイルを検索して提供できるようになります。  ファイルは、ユーザーが Web アプリの完全な URL またはルート URL を入力した場合に提供されます。

1. `Startup.Configure` の最後で、" */hub*" ルートを `ChatHub` ハブにマップします。 *Hello World!* を表示するコードを 次の行に置き換えます。 

   [!code-csharp[Startup](signalr-typescript-webpack/sample/3.x/Startup.cs?name=snippet_UseSignalR&highlight=3)]

1. `Startup.ConfigureServices` で、[AddSignalR](/dotnet/api/microsoft.extensions.dependencyinjection.signalrdependencyinjectionextensions.addsignalr#Microsoft_Extensions_DependencyInjection_SignalRDependencyInjectionExtensions_AddSignalR_Microsoft_Extensions_DependencyInjection_IServiceCollection_) を呼び出します。

   [!code-csharp[Startup](signalr-typescript-webpack/sample/3.x/Startup.cs?name=snippet_AddSignalR)]

1. SignalR ハブを格納するために、プロジェクト ルート *SignalRWebPack/* に *Hubs* という名前の新しいディレクトリを作成します。

1. 次のコードを使用して、*Hubs/ChatHub.cs* を作成します。

    [!code-csharp[ChatHub](signalr-typescript-webpack/sample/3.x/snippets/ChatHub.cs?name=snippet_ChatHubStubClass)]

1. `ChatHub` 参照を解決するには、次の `using` ステートメントを "*Startup.cs*" ファイルの先頭に追加します。

    [!code-csharp[Startup](signalr-typescript-webpack/sample/3.x/Startup.cs?name=snippet_HubsNamespace)]

## <a name="enable-client-and-server-communication"></a>クライアントとサーバーの通信を有効にする

現在、アプリにはメッセージを送信するための基本フォームが表示されていますが、まだ機能していません。 サーバーは特定のルートをリッスンしていますが、メッセージの送信については何も行いません。

1. プロジェクト ルートで、次のコマンドを実行します。

    ```console
    npm i @microsoft/signalr @types/node
    ```

    上記のコマンドにより、次がインストールされます。

     * [SignalR TypeScript クライアント](https://www.npmjs.com/package/@microsoft/signalr)。クライアントがサーバーにメッセージを送信できるようになります。
     * Node.js の TypeScript 型定義。Node.js 型のコンパイル時チェックが有効になります。

1. 強調表示されたコードを *src/index.ts* ファイルに追加します。

    [!code-typescript[index.ts](signalr-typescript-webpack/sample/3.x/snippets/index2.ts?name=snippet_IndexTsPhase2File&highlight=2,9-23)]

    上記のコードは、サーバーからのメッセージの受信をサポートします。 `HubConnectionBuilder` クラスは、サーバー接続を構成するための新しいビルダーを作成します。 `withUrl` 関数は、ハブ URL を構成します。

    SignalR により、クライアントとサーバー間でのメッセージのやり取りが可能になります。 各メッセージには特定の名前があります。 たとえば、`messageReceived` という名前のメッセージは、メッセージ ゾーンに新しいメッセージを表示するためのロジックを実行できます。 特定のメッセージをリッスンするには、`on` 関数を使用します。 任意の数のメッセージ名をリッスンできます。 作成者の名前や受信したメッセージの内容など、パラメーターをメッセージに渡すこともできます。 クライアントがメッセージを受信すると、`innerHTML` 属性に作成者の名前とメッセージ コンテンツを持つ新しい `div` 要素が作成されます。 これはメッセージを表示する主要な `div` 要素に追加されます。

1. これでクライアントがメッセージを受信できるようになったので、メッセージを送信するように構成します。 強調表示されたコードを *src/index.ts* ファイルに追加します。

    [!code-typescript[index.ts](signalr-typescript-webpack/sample/3.x/src/index.ts?highlight=34-35)]

    WebSocket 接続を介してメッセージを送信するには、`send` メソッドを呼び出す必要があります。 メソッドの最初のパラメーターは、メッセージ名です。 メッセージ データは、他のパラメーターに存在しています。 この例では、`newMessage` として識別されたメッセージがサーバーに送信されます。 メッセージは、ユーザー名と、テキスト ボックスへのユーザー入力で構成されます。 送信が機能していると、テキスト ボックスの値はクリアされます。

1. `NewMessage` メソッドを `ChatHub` クラスに追加します。

    [!code-csharp[ChatHub](signalr-typescript-webpack/sample/3.x/Hubs/ChatHub.cs?highlight=8-11)]

    上記のコードは、サーバーがメッセージを受信すると、受信したメッセージをすべての接続されているユーザーにブロードキャストします。 すべてのメッセージを受信するのに、ジェネリック `on` メソッドは必要ありません。 メソッドはメッセージ名のサフィックスに基づいて名前が付けられています。

    この例では、TypeScript クライアントが `newMessage` として識別されるメッセージを送信します。 C# `NewMessage` メソッドは、データがクライアントから送信されることを想定しています。 [Clients.All](/dotnet/api/microsoft.aspnetcore.signalr.ihubclients-1.all) で [SendAsync](/dotnet/api/microsoft.aspnetcore.signalr.clientproxyextensions.sendasync) が呼び出されます。 受信したメッセージは、ハブに接続されているすべてのクライアントに送信されます。

## <a name="test-the-app"></a>アプリのテスト

次の手順で、アプリの動作を確認します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. *リリース* モードで Webpack を実行します。 **パッケージ マネージャー コンソール** ウィンドウを使用して、プロジェクト ルートで次のコマンドを実行します。 プロジェクト ルートにいない場合は、コマンドを入力する前に `cd SignalRWebPack` と入力します。

    [!INCLUDE [npm-run-release](../includes/signalr-typescript-webpack/npm-run-release.md)]

1. **[デバッグ]**  >  **[デバッグなしで開始]** の順に選択して、デバッガーをアタッチせずに、ブラウザーでアプリを起動します。 *wwroot/index.html* ファイルは `http://localhost:<port_number>` で提供されます。

   コンパイル エラーが発生した場合は、ソリューションを閉じてから再度開いてみてください。 

1. 別のブラウザー インスタンスを開きます (任意のブラウザー)。 アドレス バーに URL を貼り付けます。

1. いずれかのブラウザーを選択し、 **[メッセージ]** テキスト ボックスにメッセージを入力し、 **[送信]** ボタンをクリックします。 次の瞬間、両方のページに一意のユーザー名とメッセージが表示されます。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

1. プロジェクト ルートで次のコマンドを実行して、*リリース* モードで Webpack を実行します。

    [!INCLUDE [npm-run-release](../includes/signalr-typescript-webpack/npm-run-release.md)]

1. プロジェクト ルートで次のコマンドを実行して、アプリをビルドして実行します。

    ```dotnetcli
    dotnet run
    ```

    Web サーバーは、アプリを起動して、localhost 上で使用できるようにします。

1. ブラウザーを開いて `http://localhost:<port_number>` にアクセスします。 *wwwroot/index.html* ファイルが提供されます。 アドレス バーから URL をコピーします。

1. 別のブラウザー インスタンスを開きます (任意のブラウザー)。 アドレス バーに URL を貼り付けます。

1. いずれかのブラウザーを選択し、 **[メッセージ]** テキスト ボックスにメッセージを入力し、 **[送信]** ボタンをクリックします。 次の瞬間、両方のページに一意のユーザー名とメッセージが表示されます。

---

![両方のブラウザー ウィンドウに表示されるメッセージ](signalr-typescript-webpack/_static/browsers-message-broadcast.png)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

## <a name="prerequisites"></a>必須コンポーネント

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) と **ASP.NET と Web 開発** ワークロード
* [.NET Core SDK 2.2 以降](https://dotnet.microsoft.com/download/dotnet-core)
* [Node.js](https://nodejs.org/) ([npm](https://www.npmjs.com/) 使用)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* [Visual Studio Code](https://code.visualstudio.com/download)
* [.NET Core SDK 2.2 以降](https://dotnet.microsoft.com/download/dotnet-core)
* [C# for Visual Studio Code バージョン 1.17.1 またはそれ以降](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* [Node.js](https://nodejs.org/) ([npm](https://www.npmjs.com/) 使用)

---

## <a name="create-the-aspnet-core-web-app"></a>ASP.NET Core Web アプリを作成する

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

*PATH* 環境変数で npm を検索するように Visual Studio を構成します。 既定では、Visual Studio は、そのインストール ディレクトリ内で見つかった npm のバージョンを使用します。 Visual Studio のこれらの説明に従ってください。

1. **[ツール]** > **[オプション]** > **[プロジェクトとソリューション]** > **[Web パッケージ管理]** > **[外部 Web ツール]** に移動します。
1. リストから *[$(PATH)]* エントリを選択します。 上向き矢印をクリックして、エントリをリストの 2 番目の位置に移動します。

    ![Visual Studio の構成](signalr-typescript-webpack/_static/signalr-configure-path-visual-studio.png)

Visual Studio の構成が完了しました。 次はプロジェクトを作成します。

1. **[ファイル]** > **[新規]** > **[プロジェクト]** メニュー オプションを使用して、 **[ASP.NET Core Web アプリケーション]** テンプレートを選択します。
1. プロジェクトに *SignalRWebPack* という名前を付け、 **[作成]** を選択します。
1. ターゲット フレームワークのドロップダウンから、 *[.NET Core]* を選択し、フレームワーク セレクターのドロップダウンから *[ASP.NET Core 2.2]* を選択します。 **空** のテンプレートを選択して、 **[作成]** を選択します。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

**統合ターミナル** で次のコマンドを実行します。

```dotnetcli
dotnet new web -o SignalRWebPack
```

.NET Core をターゲットとする、空の ASP.NET Core Web アプリが *SignalRWebPack* ディレクトリ内に作成されます。

---

## <a name="configure-webpack-and-typescript"></a>WebPack および TypeScript の構成

次の手順では、TypeScript の JavaScript への変換と、クライアント側のリソースのバンドルを構成します。

1. プロジェクト ルートで次のコマンドを実行して、"*package.json*" ファイルを作成します。

    ```console
    npm init -y
    ```

1. 強調表示されているプロパティを *package.json* ファイルに追加します。

    [!code-json[package.json](signalr-typescript-webpack/sample/2.x/snippets/package1.json?highlight=4)]

    `private` プロパティを `true` に設定して、次の手順でパッケージのインストールの警告が表示されないようにします。

1. 必要な npm パッケージをインストールします。 プロジェクト ルートで、次のコマンドを実行します。

    ```console
    npm install -D -E clean-webpack-plugin@1.0.1 css-loader@2.1.0 html-webpack-plugin@4.0.0-beta.5 mini-css-extract-plugin@0.5.0 ts-loader@5.3.3 typescript@3.3.3 webpack@4.29.3 webpack-cli@3.2.3
    ```

    注目するべきコマンドの詳細:

    * バージョン番号は、各パッケージ名の `@` 記号の後に続きます。 npm によって、これらの特定のパッケージ バージョンがインストールされます。
    * `-E` オプションは、[セマンティック バージョニング](https://semver.org/)範囲演算子を *package.json* に書き込む npm の既定の動作を無効にします。 たとえば、`"webpack": "4.29.3"` が `"webpack": "^4.29.3"` の代わりに使用されています。 このオプションにより、新しいパッケージ バージョンへの予期しないアップグレードが防止されます。

    詳細については、[npm-install](https://docs.npmjs.com/cli/install) のドキュメントを参照してください。

1. "*package.json*" ファイルの `scripts` プロパティを次のコードで置き換えます。

    ```json
    "scripts": {
      "build": "webpack --mode=development --watch",
      "release": "webpack --mode=production",
      "publish": "npm run release && dotnet publish -c Release"
    },
    ```

    スクリプトの説明 (一部):

    * `build`:開発モードでクライアント側のリソースをバンドルし、ファイルの変更を監視します。 ファイル監視により、プロジェクト ファイルを変更するたびにバンドルが再生成されます。 `mode` オプションは、ツリー シェイキングや縮小などの運用環境の最適化を無効にします。 `build` は開発でのみ使用します。
    * `release`:運用モードでクライアント側のリソースをバンドルします。
    * `publish`:`release` スクリプトを実行して、運用モードでクライアント側のリソースをバンドルします。 .NET Core CLI の [publish](/dotnet/core/tools/dotnet-publish) コマンドを呼び出してアプリを公開します。

1. プロジェクト ルートに、次のコードを含む "*webpack.config.js*" という名前のファイルを作成します。

    [!code-javascript[webpack.config.js](signalr-typescript-webpack/sample/2.x/webpack.config.js)]

    上記のファイルは、Webpack コンパイルを構成します。 注目するべき構成の詳細 (一部):

    * `output` プロパティにより、*dist* の既定値がオーバーライドされます。 代わりにバンドルが *wwwroot* ディレクトリ内に生成されます。
    * `resolve.extensions` 配列には、SignalR クライアント JavaScript をインポートするための *.js* が含まれています。

1. プロジェクトのクライアント側アセットを格納するために、プロジェクト ルートに新しい "*src*" ディレクトリを作成します。

1. 次のマークアップを含む "*src/index.html*" を作成します。

    [!code-html[index.html](signalr-typescript-webpack/sample/2.x/src/index.html)]

    上記の HTML では、ホームページの定型マークアップが定義されます。

1. 新しい *src/css* ディレクトリを作成します。 その目的は、プロジェクトの *.css* ファイルを格納することです。

1. 次のマークアップを含む "*src/css/main.css*" を作成します。

    [!code-css[main.css](signalr-typescript-webpack/sample/2.x/src/css/main.css)]

    上記の *main.css* ファイルは、アプリをスタイル設定します。

1. 次の JSON を含む "*src/tsconfig.json*" を作成します。

    [!code-json[tsconfig.json](signalr-typescript-webpack/sample/2.x/src/tsconfig.json)]

    上記のコードは、[ECMAScript](https://wikipedia.org/wiki/ECMAScript) 5 対応の JavaScript を生成するように TypeScript コンパイラを構成します。

1. 次のコードを含む "*src/index.ts*" を作成します。

    [!code-typescript[index.ts](signalr-typescript-webpack/sample/2.x/snippets/index1.ts?name=snippet_IndexTsPhase1File)]

    上記の TypeScript は、DOM 要素への参照を取得し、次の 2 つのイベント ハンドラーをアタッチします。

    * `keyup`:ユーザーが `tbMessage` テキスト ボックスに入力すると、このイベントが発生します。 ユーザーが **Enter** キーを押すと、`send` 関数が呼び出されます。
    * `click`: ユーザーが **[送信]** ボタンをクリックすると、このイベントが発生します。 `send` 関数が呼び出されます。

## <a name="configure-the-aspnet-core-app"></a>ASP.NET Core アプリを構成する

1. `Startup.Configure` メソッドで提供されたコードにより、*Hello World!* が表示されます。 `app.Run` メソッド呼び出しを、[UseDefaultFiles](/dotnet/api/microsoft.aspnetcore.builder.defaultfilesextensions.usedefaultfiles#Microsoft_AspNetCore_Builder_DefaultFilesExtensions_UseDefaultFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_) と [UseStaticFiles](/dotnet/api/microsoft.aspnetcore.builder.staticfileextensions.usestaticfiles#Microsoft_AspNetCore_Builder_StaticFileExtensions_UseStaticFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_) の呼び出しで置き換えます。

    [!code-csharp[Startup](signalr-typescript-webpack/sample/2.x/Startup.cs?name=snippet_UseStaticDefaultFiles)]

    上記のコードにより、サーバーが *index.html* ファイルを見つけて提供することができます。ユーザーがファイルの完全な URL または Web アプリのルート URL を入力するかどうかは関係ありません。

1. `Startup.ConfigureServices` で [AddSignalR](/dotnet/api/microsoft.extensions.dependencyinjection.signalrdependencyinjectionextensions.addsignalr#Microsoft_Extensions_DependencyInjection_SignalRDependencyInjectionExtensions_AddSignalR_Microsoft_Extensions_DependencyInjection_IServiceCollection_) を呼び出します。 これにより SignalR サービスがプロジェクトに追加されます。

    [!code-csharp[Startup](signalr-typescript-webpack/sample/2.x/Startup.cs?name=snippet_AddSignalR)]

1. */hub* ルートを `ChatHub` ハブにマップします。 `Startup.Configure` の末尾に次の行を追加します。

    [!code-csharp[Startup](signalr-typescript-webpack/sample/2.x/Startup.cs?name=snippet_UseSignalR)]

1. プロジェクト ルートに *Hubs* という新しいディレクトリを作成します。 その目的は、次の手順で作成される SignalR ハブを格納することです。

1. 次のコードを使用して、*Hubs/ChatHub.cs* を作成します。

    [!code-csharp[ChatHub](signalr-typescript-webpack/sample/2.x/snippets/ChatHub.cs?name=snippet_ChatHubStubClass)]

1. `ChatHub` 参照を解決するには、次のコードを *Startup.cs* ファイルの先頭に追加します。

    [!code-csharp[Startup](signalr-typescript-webpack/sample/2.x/Startup.cs?name=snippet_HubsNamespace)]

## <a name="enable-client-and-server-communication"></a>クライアントとサーバーの通信を有効にする

アプリには現在、メッセージを送信するための単純なフォームが表示されています。 送信しようとしても、何も起こりません。 サーバーは特定のルートをリッスンしていますが、メッセージの送信については何も行いません。

1. プロジェクト ルートで、次のコマンドを実行します。

    ```console
    npm install @aspnet/signalr
    ```

    上記のコマンドにより [SignalR TypeScript クライアント](https://www.npmjs.com/package/@microsoft/signalr) がインストールされ、クライアントがサーバーにメッセージを送信できるようになります。

1. 強調表示されたコードを *src/index.ts* ファイルに追加します。

    [!code-typescript[index.ts](signalr-typescript-webpack/sample/2.x/snippets/index2.ts?name=snippet_IndexTsPhase2File&highlight=2,9-23)]

    上記のコードは、サーバーからのメッセージの受信をサポートします。 `HubConnectionBuilder` クラスは、サーバー接続を構成するための新しいビルダーを作成します。 `withUrl` 関数は、ハブ URL を構成します。

    SignalR により、クライアントとサーバー間でのメッセージのやり取りが可能になります。 各メッセージには特定の名前があります。 たとえば、`messageReceived` という名前のメッセージは、メッセージ ゾーンに新しいメッセージを表示するためのロジックを実行できます。 特定のメッセージをリッスンするには、`on` 関数を使用します。 任意の数のメッセージ名をリッスンできます。 作成者の名前や受信したメッセージの内容など、パラメーターをメッセージに渡すこともできます。 クライアントがメッセージを受信すると、`innerHTML` 属性に作成者の名前とメッセージ コンテンツを持つ新しい `div` 要素が作成されます。 新しいメッセージが、メッセージを表示する主要な `div` 要素に追加されます。

1. これでクライアントがメッセージを受信できるようになったので、メッセージを送信するように構成します。 強調表示されたコードを *src/index.ts* ファイルに追加します。

    [!code-typescript[index.ts](signalr-typescript-webpack/sample/2.x/src/index.ts?highlight=34-35)]

    WebSocket 接続を介してメッセージを送信するには、`send` メソッドを呼び出す必要があります。 メソッドの最初のパラメーターは、メッセージ名です。 メッセージ データは、他のパラメーターに存在しています。 この例では、`newMessage` として識別されたメッセージがサーバーに送信されます。 メッセージは、ユーザー名と、テキスト ボックスへのユーザー入力で構成されます。 送信が機能していると、テキスト ボックスの値はクリアされます。

1. `NewMessage` メソッドを `ChatHub` クラスに追加します。

    [!code-csharp[ChatHub](signalr-typescript-webpack/sample/2.x/Hubs/ChatHub.cs?highlight=8-11)]

    上記のコードは、サーバーがメッセージを受信すると、受信したメッセージをすべての接続されているユーザーにブロードキャストします。 すべてのメッセージを受信するのに、ジェネリック `on` メソッドは必要ありません。 メソッドはメッセージ名のサフィックスに基づいて名前が付けられています。

    この例では、TypeScript クライアントが `newMessage` として識別されるメッセージを送信します。 C# `NewMessage` メソッドは、データがクライアントから送信されることを想定しています。 [Clients.All](/dotnet/api/microsoft.aspnetcore.signalr.ihubclients-1.all) で [SendAsync](/dotnet/api/microsoft.aspnetcore.signalr.clientproxyextensions.sendasync) が呼び出されます。 受信したメッセージは、ハブに接続されているすべてのクライアントに送信されます。

## <a name="test-the-app"></a>アプリのテスト

次の手順で、アプリの動作を確認します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. *リリース* モードで Webpack を実行します。 **パッケージ マネージャー コンソール** ウィンドウを使用して、プロジェクト ルートで次のコマンドを実行します。 プロジェクト ルートにいない場合は、コマンドを入力する前に `cd SignalRWebPack` と入力します。

    [!INCLUDE [npm-run-release](../includes/signalr-typescript-webpack/npm-run-release.md)]

1. **[デバッグ]**  >  **[デバッグなしで開始]** の順に選択して、デバッガーをアタッチせずに、ブラウザーでアプリを起動します。 *wwroot/index.html* ファイルは `http://localhost:<port_number>` で提供されます。

1. 別のブラウザー インスタンスを開きます (任意のブラウザー)。 アドレス バーに URL を貼り付けます。

1. いずれかのブラウザーを選択し、 **[メッセージ]** テキスト ボックスにメッセージを入力し、 **[送信]** ボタンをクリックします。 次の瞬間、両方のページに一意のユーザー名とメッセージが表示されます。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

1. プロジェクト ルートで次のコマンドを実行して、*リリース* モードで Webpack を実行します。

    [!INCLUDE [npm-run-release](../includes/signalr-typescript-webpack/npm-run-release.md)]

1. プロジェクト ルートで次のコマンドを実行して、アプリをビルドして実行します。

    ```dotnetcli
    dotnet run
    ```

    Web サーバーは、アプリを起動して、localhost 上で使用できるようにします。

1. ブラウザーを開いて `http://localhost:<port_number>` にアクセスします。 *wwwroot/index.html* ファイルが提供されます。 アドレス バーから URL をコピーします。

1. 別のブラウザー インスタンスを開きます (任意のブラウザー)。 アドレス バーに URL を貼り付けます。

1. いずれかのブラウザーを選択し、 **[メッセージ]** テキスト ボックスにメッセージを入力し、 **[送信]** ボタンをクリックします。 次の瞬間、両方のページに一意のユーザー名とメッセージが表示されます。

---

![両方のブラウザー ウィンドウに表示されるメッセージ](signalr-typescript-webpack/_static/browsers-message-broadcast.png)

::: moniker-end

## <a name="additional-resources"></a>その他の技術情報

* <xref:signalr/javascript-client>
* <xref:signalr/hubs>
