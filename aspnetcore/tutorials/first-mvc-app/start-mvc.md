---
title: ASP.NET Core MVC の概要
author: rick-anderson
description: ASP.NET Core MVC の概要について説明します。
ms.author: riande
ms.date: 01/20/2021
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
uid: tutorials/first-mvc-app/start-mvc
ms.custom: contperf-fy21q3
ms.openlocfilehash: aaf930eee351ed757be60f648bce88b182d52799
ms.sourcegitcommit: da5a5bed5718a9f8db59356ef8890b4b60ced6e9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/22/2021
ms.locfileid: "98710799"
---
# <a name="get-started-with-aspnet-core-mvc"></a>ASP.NET Core MVC の概要

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-5.0"

[!INCLUDE [consider RP](~/includes/razor.md)]

これは、コントローラーとビューを使用した ASP.NET Core MVC Web 開発について説明するシリーズの最初のチュートリアルです。

シリーズの最後には、映画のデータを管理および表示できるアプリができあがります。 以下の方法について説明します。

> [!div class="checklist"]
> * Web アプリを作成する。
> * モデルを追加してスキャフォールディングする。
> * データベースを使用する。
> * 検索と検証を追加する。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/first-mvc-app/start-mvc/sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="prerequisites"></a>必須コンポーネント

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-5.0.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-5.0.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-5.0.md)]

---

## <a name="create-a-web-app"></a>Web アプリの作成

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Visual Studio を開始し、 **[新しいプロジェクトの作成]** を選択します。
* **[新しいプロジェクトの作成]** ダイアログで、 **[ASP.NET Core Web アプリケーション]** > **[次へ]** の順に選択します。
* **[新しいプロジェクトの構成]** ダイアログで、 **[プロジェクト名]** に「`MvcMovie`」と入力します。 プロジェクトに *MvcMovie* という名前を付けることが重要です。 コードをコピーする際に、大文字と小文字の区別が各 `namespace` と一致する必要があります。
* **［作成］** を選択します
* **[新しい ASP.NET Core Web アプリケーションの作成]** ダイアログで、次のものを選択します。
  * ドロップダウンで **[.NET Core]** と **[ASP.NET Core 5.0]**
  * **ASP.NET Core Web アプリ (Model-View-Controller)**
  * **Create**。

![新しい ASP.NET Core Web アプリケーションを作成する ](start-mvc/_static/mvcVS19v16.9.png)

プロジェクトを作成する別の方法については、「[Visual Studio で新しいプロジェクトを作成する](/visualstudio/ide/create-new-project)」をご覧ください。

Visual Studio により、作成した MVC プロジェクトに既定のプロジェクト テンプレートが使用されました。 作成したプロジェクトは:

* 動作するアプリです。
* 基本的なスターター プロジェクトです。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

このチュートリアルは VS Code の知識があることを前提としています。 詳細については、[VS Code の概要](https://code.visualstudio.com/docs)に関するページと「[Visual Studio Code ヘルプ](#visual-studio-code-help)」を参照してください。

* [統合ターミナル](https://code.visualstudio.com/docs/editor/integrated-terminal)を開きます。
* プロジェクトを格納するディレクトリに移動します (`cd`)。
* 次のコマンドを実行します。

   ```dotnetcli
   dotnet new mvc -o MvcMovie
   code -r MvcMovie
   ```

  * "**ビルドとデバッグに必要な資産が 'MvcMovie' にありません。追加しますか?** " というダイアログ ボックスが表示されたら、 **[はい]** を選択します

  * `dotnet new mvc -o MvcMovie`: *MvcMovie* フォルダー内に新しい ASP.NET Core MVC プロジェクトを作成します。
  * `code -r MvcMovie`:Visual Studio Code で *MvcMovie.csproj* プロジェクト ファイルを読み込みます。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* **[ファイル]** > **[新しいソリューション]** の順に選択します。

  ![macOS の新しいソリューション](start-mvc/_static/new_project_vsmac.png)

* バージョン 8.6 より前の Visual Studio for Mac では、 **[.NET Core]** 、 **[アプリ]** 、 **[Web アプリケーション (Model-View-Controller)]** 、 **[次へ]** の順に選択します。 バージョン 8.6 以降では、 **[Web and Console]\(Web とコンソール\)** 、 **[アプリ]** 、 **[Web アプリケーション (Model-View-Controller)]** 、 **[次へ]** の順に選択します。

  ![macOS Web アプリ テンプレートの選択](start-mvc/_static/web_app_template_vsmac.png)

* **[Configure your new Web Application]\(新しい Web アプリケーションを構成する\)** ダイアログで、次の操作を行います。

  * **[認証]** に **[認証なし]** が設定されていることを確認します。
  * **[ターゲット フレームワーク]** を選択するオプションが表示されている場合は、最新の 5.x バージョンを選択します。
  * **[次へ]** を選択します。

* プロジェクトに **MvcMovie** という名前を付けて、 **[作成]** を選択します。

  ![macOS でプロジェクトに名前を付ける](start-mvc/_static/MvcMovie.png)

---

### <a name="run-the-app"></a>アプリを実行する

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Ctrl + F5 キーを選択して、デバッガーなしでアプリを実行します。

  [!INCLUDE[](~/includes/trustCertVS.md)]

  Visual Studio:

  * [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) を起動します。
  * アプリを実行します。

  アドレス バーには、`example.com` などではなく、`localhost:port#` が表示されます。 ローカル コンピューターの標準的なホスト名は `localhost` です。 Visual Studio で Web プロジェクトが作成されるとき、Web サーバーにはランダムなポートが使用されます。

Ctrl + F5 キーを選択してデバッグなしでアプリを起動すると、次の操作を行えます。

* コードを変更します。
* ファイルを保存します。
* ブラウザーをすぐに更新して、コードの変更を確認します。

**[デバッグ]** メニュー項目から、デバッグ モードまたは非デバッグ モードでアプリを起動できます。

![[デバッグ] メニュー](start-mvc/_static/debug_menu50.png)

**[IIS Express]** ボタンを選択することで、アプリをデバッグできます。

![IIS Express](start-mvc/_static/iis_express50.png)

次の図はアプリを示しています。

![ホームまたはインデックス ページ](start-mvc/_static/home50-vs.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Ctrl + F5 キーを選択して、デバッガーなしで実行します。

  [!INCLUDE[](~/includes/trustCertVSC.md)]

  Visual Studio Code:

  * [Kestrel](xref:fundamentals/servers/kestrel) を起動します
  * ブラウザーを起動します。
  * `https://localhost:5001` に移動します。

  アドレス バーには、`example.com` などではなく、`localhost:port:5001` が表示されます。 ローカル コンピューターの標準的なホスト名は `localhost` です。 localhost では、ローカル コンピューターからの Web 要求のみが処理されます。

Ctrl + F5 キーを選択してデバッグなしでアプリを起動すると、次の操作を行えます。

* コードを変更します。
* ファイルを保存します。
* ブラウザーをすぐに更新して、コードの変更を確認します。

  ![ホームまたはインデックス ページ](start-mvc/_static/home50-port5001.png)

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* **[実行]** > **[デバッグなしで開始]** の順に選択してアプリを起動します。

  Visual Studio for Mac:

  * [Kestrel](xref:fundamentals/servers/index#kestrel) サーバーを起動します。
  * ブラウザーを起動します。
  * `http://localhost:port` に移動します。*port* はランダムに選択されるポート番号です。

  [!INCLUDE[](~/includes/trustCertMac.md)]

  アドレス バーには、`example.com` などではなく、`localhost:port#` が表示されます。 ローカル コンピューターの標準的なホスト名は `localhost` です。 Visual Studio で Web プロジェクトが作成されるとき、Web サーバーにはランダムなポートが使用されます。

**[実行]** メニューから、デバッグ モードまたは非デバッグ モードでアプリを起動できます。

次の図はアプリを示しています。

![ホームまたはインデックス ページ](./start-mvc/_static/output_macos.png)

---

[!INCLUDE[](~/includes/vs-vsc-vsmac-help.md)]

このチュートリアルの次のパートでは、MVC について説明し、コードの作成を開始します。

> [!div class="step-by-step"]
> [次へ: コントローラーの追加](adding-controller.md)

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!INCLUDE [consider RP](~/includes/razor.md)]

これは、コントローラーとビューを使用した ASP.NET Core MVC Web 開発について説明するシリーズの最初のチュートリアルです。

シリーズの最後には、映画のデータを管理および表示できるアプリができあがります。 以下の方法について説明します。

> [!div class="checklist"]
> * Web アプリを作成する。
> * モデルを追加してスキャフォールディングする。
> * データベースを使用する。
> * 検索と検証を追加する。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/first-mvc-app/start-mvc/sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="prerequisites"></a>必須コンポーネント

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.1.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.1.md)]

---

## <a name="create-a-web-app"></a>Web アプリの作成

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Visual Studio から **[新しいプロジェクトの作成]** を選択します。

* **[ASP.NET Core Web アプリケーション]** > **[次へ]** の順に選択します。

  ![新しい ASP.NET Core Web アプリケーション プロジェクトを作成します](start-mvc/_static/np_2.1.png)

* プロジェクトに **MvcMovie** という名前を付けて、 **[作成]** を選択します。 コードをコピーするときに名前空間が一致するように、プロジェクトに **MvcMovie** と名前を付けることが重要です。

  ![新しいプロジェクトを構成する](start-mvc/_static/config.png)

* **[Web アプリケーション (モデル ビュー コントローラー)]** を選択します。 ドロップダウン ボックスから **[.NET Core]** と **[ASP.NET Core 3.1]** を選択した後、 **[作成]** を選択します。

  ![[新しいプロジェクト] ダイアログ、左ウィンドウの .NET Core、ASP.NET Core Web ](start-mvc/_static/new_project30.png)

Visual Studio により、作成した MVC プロジェクトに既定のプロジェクト テンプレートが使用されました。 作成したプロジェクトは:

* 動作するアプリです。
* 基本的なスターター プロジェクトです。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

このチュートリアルは VS Code の知識があることを前提としています。 詳細については、[VS Code の概要](https://code.visualstudio.com/docs)に関するページと「[Visual Studio Code ヘルプ](#visual-studio-code-help)」を参照してください。

* [統合ターミナル](https://code.visualstudio.com/docs/editor/integrated-terminal)を開きます。
* ディレクトリを、プロジェクトを格納するフォルダーに変更します (`cd`)。
* 次のコマンドを実行します。

   ```dotnetcli
   dotnet new mvc -o MvcMovie
   code -r MvcMovie
   ```

  * "**ビルドとデバッグに必要な資産が 'MvcMovie' にありません。追加しますか?** " というダイアログ ボックスが表示されたら、 **[はい]** を選択します。

  * `dotnet new mvc -o MvcMovie`: *MvcMovie* フォルダー内に新しい ASP.NET Core MVC プロジェクトを作成します。
  * `code -r MvcMovie`:Visual Studio Code で *MvcMovie.csproj* プロジェクト ファイルを読み込みます。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* **[ファイル]** > **[新しいソリューション]** の順に選択します。

  ![macOS の新しいソリューション](start-mvc/_static/new_project_vsmac.png)

* バージョン 8.6 より前の Visual Studio for Mac では、 **[.NET Core]** 、 **[アプリ]** 、 **[Web アプリケーション (Model-View-Controller)]** 、 **[次へ]** の順に選択します。 バージョン 8.6 以降では、 **[Web and Console]\(Web とコンソール\)** 、 **[アプリ]** 、 **[Web アプリケーション (Model-View-Controller)]** 、 **[次へ]** の順に選択します。

  ![macOS Web アプリ テンプレートの選択](start-mvc/_static/web_app_template_vsmac.png)

* **[Configure your new Web Application]\(新しい Web アプリケーションを構成する\)** ダイアログで、次の操作を行います。

  * **[認証]** に **[認証なし]** が設定されていることを確認します。
  * **[ターゲット フレームワーク]** を選択するオプションが表示されている場合は、最新の 3.x バージョンを選択します。
  * **[次へ]** を選択します。

* プロジェクトに **MvcMovie** という名前を付けて、 **[作成]** を選択します。

  ![macOS でプロジェクトに名前を付ける](start-mvc/_static/MvcMovie.png)

---

### <a name="run-the-app"></a>アプリを実行する

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Ctrl + F5 キーを選択して、デバッグなしでアプリを実行します。

  [!INCLUDE[](~/includes/trustCertVS.md)]

  Visual Studio:

  * [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) を起動します。
  * アプリを実行します。

  アドレス バーには、`example.com` などではなく、`localhost:port#` が表示されます。 ローカル コンピューターの標準的なホスト名は `localhost` です。 Visual Studio で Web プロジェクトが作成されるとき、Web サーバーにはランダムなポートが使用されます。

Ctrl + F5 キーを選択してデバッグなしでアプリを起動すると、次の操作を行えます。

* コードを変更します。
* ファイルを保存します。
* ブラウザーをすぐに更新して、コードの変更を確認します。

**[デバッグ]** メニュー項目から、デバッグ モードまたは非デバッグ モードでアプリを起動できます。

![[デバッグ] メニュー](start-mvc/_static/debug_menu.png)

**[IIS Express]** ボタンを選択することで、アプリをデバッグできます。

![IIS Express](start-mvc/_static/iis_express.png)

次の図はアプリを示しています。

![ホームまたはインデックス ページ](start-mvc/_static/home2.2.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Ctrl + F5 キーを選択して、デバッグなしでアプリを実行します。

  [!INCLUDE[](~/includes/trustCertVSC.md)]

  Visual Studio Code:

  * [Kestrel](xref:fundamentals/servers/kestrel) を起動します
  * ブラウザーを起動します。
  * `https://localhost:5001` に移動します。

  アドレス バーには、`example.com` などではなく、`localhost:port:5001` が表示されます。 ローカル コンピューターの標準的なホスト名は `localhost` です。 localhost では、ローカル コンピューターからの Web 要求のみが処理されます。

Ctrl + F5 キーを選択してデバッグなしでアプリを起動すると、次の操作を行えます。

* コードを変更します。
* ファイルを保存します。
* ブラウザーをすぐに更新して、コードの変更を確認します。

  ![ホームまたはインデックス ページ](start-mvc/_static/home2.2.png)

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* **[実行]** > **[デバッグなしで開始]** の順に選択してアプリを起動します。

  Visual Studio for Mac: [Kestrel](xref:fundamentals/servers/index#kestrel) サーバーが開始され、ブラウザーが起動して `http://localhost:port` にアクセスします。*port* はランダムに選択されたポート番号になります。

[!INCLUDE[](~/includes/trustCertMac.md)]

アドレス バーには、`example.com` などではなく、`localhost:port#` が表示されます。 ローカル コンピューターの標準的なホスト名は `localhost` です。 Visual Studio が Web プロジェクトを作成する場合は、Web サーバーにランダム ポートが使用されます。 アプリを実行する際には、別のポート番号が表示されます。

**[実行]** メニューから、デバッグ モードまたは非デバッグ モードでアプリを起動できます。

次の図はアプリを示しています。

![ホームまたはインデックス ページ](./start-mvc/_static/output_macos.png)

---

[!INCLUDE[](~/includes/vs-vsc-vsmac-help.md)]

このチュートリアルの次のパートでは、MVC について説明し、コードの作成を開始します。

> [!div class="step-by-step"]
> [次へ](adding-controller.md)

::: moniker-end
