---
title: ASP.NET Core Blazor WebAssembly をデバッグする
author: guardrex
description: Blazor アプリをデバッグする方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/26/2020
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
uid: blazor/debug
ms.openlocfilehash: 9214fa10a2bf7d53a4cb12263a3fa69bded84b29
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100536234"
---
# <a name="debug-aspnet-core-blazor-webassembly"></a>ASP.NET Core Blazor WebAssembly をデバッグする

Blazor WebAssembly アプリは、Chromium ベースのブラウザー (Edge/Chrome) のブラウザー開発ツールを使用してデバッグできます。 次の統合開発環境 (IDE) を使用して、アプリをデバッグすることもできます。

* Visual Studio
* Visual Studio for Mac
* Visual Studio Code

利用可能なシナリオには以下が含まれます。

* ブレークポイントの設定と削除を行う。
* IDE でデバッグ サポートを使用してアプリを実行する。
* コードを 1 ステップずつ実行する。
* IDE でキーボード ショートカットを使用してコード実行を再開する。
* *[ローカル]* ウィンドウで、ローカル変数の値を観察する。
* JavaScript と .NET の間の呼び出しチェーンなど、呼び出し履歴を確認する。

現時点では、次の操作を行うことは *できません*。

* 未処理の例外の発生時に中断する。
* デバッグ プロキシが実行される前に、アプリの起動中にブレークポイントにヒットします。 これには、`Program.Main` (`Program.cs`) のブレークポイントと、アプリから要求された最初のページによって読み込まれるコンポーネントの [`OnInitialized{Async}` メソッド](xref:blazor/components/lifecycle#component-initialization-methods)のブレークポイントが含まれます。
* ローカル以外のシナリオ (たとえば、[Linux 用 Windows サブシステム (WSL)](/windows/wsl/) や [Visual Studio Codespaces](/visualstudio/codespaces/overview/what-is-vsonline)) でデバッグします。
* [`dotnet watch run`](xref:tutorials/dotnet-watch) を使用してアプリを実行するなどして、デバッグ中にホステッド Blazor ソリューションのバックエンド `*Server*` アプリを自動的に再構築します。

## <a name="prerequisites"></a>必須コンポーネント

デバッグには、次のいずれかのブラウザーが必要です。

* Google Chrome (バージョン 70 以降) (既定値)
* Microsoft Edge (バージョン 80 以降)

ファイアウォールまたはプロキシでデバッグ プロキシ (`NodeJS` プロセス) との通信がブロックされていないことを確認します。 詳細については、「[ファイアウォールの構成](#firewall-configuration)」セクションをご覧ください。

Visual Studio for Mac のバージョン 8.8 (ビルド 1532) 以降が必要です。

1. Visual Studio for Mac の最新リリースをインストールします。それには、 **[Visual Studio for Mac をダウンロードする]** ボタンを選択します ([Microsoft: Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/))。
1. Visual Studio 内から "*プレビュー*" チャネルを選択します。 詳細については、「[Visual Studio for Mac のプレビュー バージョンをインストールする](/visualstudio/mac/install-preview)」を参照してください。

> [!NOTE]
> 現在、macOS での Apple Safari はサポートされていません。

## <a name="enable-debugging"></a>デバッグの有効化

既存の Blazor WebAssembly アプリのデバッグを有効にするには、スタートアップ プロジェクトの `launchSettings.json` ファイルを更新して、各起動プロファイルに次の `inspectUri` プロパティを含めます。

```json
"inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/_framework/debug/ws-proxy?browser={browserInspectUri}"
```

更新されると、`launchSettings.json` ファイルは次の例のようになります。

[!code-json[](debug/launchSettings.json?highlight=14,22)]

`inspectUri` プロパティ:

* アプリが Blazor WebAssembly アプリであることを IDE で検出できるようにします。
* スクリプト デバッグ インフラストラクチャに対して、Blazor のデバッグ プロキシを使用してブラウザーに接続するように指示します。

起動したブラウザー (`browserInspectUri`) の Websocket プロトコル (`wsProtocol`)、ホスト (`url.hostname`)、ポート (`url.port`)、およびインスペクター URI のプレースホルダー値は、フレームワークによって提供されます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Visual Studio で Blazor WebAssembly アプリをデバッグするには、次のようにします。

1. 新しい ASP.NET Core でホストされる Blazor WebAssembly アプリを作成します。
1. <kbd>F5</kbd> キーを押して、デバッガーでアプリを実行します。

   > [!NOTE]
   > **デバッグなしの開始** (<kbd>Ctrl</kbd>+<kbd>F5</kbd>) はサポートされていません。 アプリがデバッグ構成で実行されている場合、デバッグのオーバーヘッドによって常にパフォーマンスがわずかに低下します。

1. "`*Client*`" アプリで、`Pages/Counter.razor` の `currentCount++;` 行にブレークポイントを設定します。
1. ブラウザーで `Counter` ページに移動し、 **[Click me]** ボタンを選択して、ブレークポイントで停止させます。
1. Visual Studio の **[ローカル]** ウィンドウで、`currentCount` フィールドの値を調べます。
1. <kbd>F5</kbd> キーを押して、実行を続行します。

Blazor WebAssembly アプリをデバッグしている間に、サーバー コードをデバッグすることもできます。

1. `Pages/FetchData.razor` ページの <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> にブレークポイントを設定します。
1. `Get` アクション メソッドの `WeatherForecastController` にブレークポイントを設定します。
1. `Fetch Data` ページに移動し、サーバーに HTTP 要求を発行する直前の、`FetchData` コンポーネント内の最初のブレークポイントで停止させます。
1. <kbd>F5</kbd> キーを押して実行を続け、サーバーの `WeatherForecastController` 内のブレークポイントで停止させます。
1. <kbd>F5</kbd> キーをもう一度押して実行を続け、天気予報テーブルがブラウザーにレンダリングされることを確認します。

> [!NOTE]
> デバッグ プロキシが実行状態になる前のアプリの起動中には、ブレークポイントで停止することは **ありません**。 これには、`Program.Main` (`Program.cs`) のブレークポイントと、アプリから要求された最初のページによって読み込まれるコンポーネントの [`OnInitialized{Async}` メソッド](xref:blazor/components/lifecycle#component-initialization-methods)のブレークポイントが含まれます。

アプリが `/` 以外の[アプリ ベース パス](xref:blazor/host-and-deploy/index#app-base-path)でホストされている場合、`Properties/launchSettings.json` で次のプロパティを更新し、アプリのベース パスを反映させます。

* `applicationUrl`:

  ```json
  "iisSettings": {
    ...
    "iisExpress": {
      "applicationUrl": "http://localhost:{INSECURE PORT}/{APP BASE PATH}/",
      "sslPort": {SECURE PORT}
    }
  },
  ```

* 各プロファイルの `inspectUri`:

  ```json
  "profiles": {
    ...
    "{PROFILE 1, 2, ... N}": {
      ...
      "inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/{APP BASE PATH}/_framework/debug/ws-proxy?browser={browserInspectUri}",
      ...
    }
  }
  ```

前の設定のプレースホルダー:

* `{INSECURE PORT}`:セキュリティで保護されていないポート。 既定では、ランダムな値が提供されますが、カスタム ポートが許可されます。
* `{APP BASE PATH}`:アプリのベース パス。
* `{SECURE PORT}`:セキュリティで保護されているポート。 既定では、ランダムな値が提供されますが、カスタム ポートが許可されます。
* `{PROFILE 1, 2, ... N}`:起動設定プロファイル。 通常、既定では、1 つのアプリから複数のプロファイルが指定されます (たとえば、IIS Express 用のプロファイルと、Kestrel サーバーで使用されるプロジェクト プロファイル)。

次の例では、アプリは `/OAT` でホストされており、アプリ ベース パスは `<base href="/OAT/">` として `wwwroot/index.html` に構成されています。

```json
"applicationUrl": "http://localhost:{INSECURE PORT}/OAT/",
```

```json
"inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/OAT/_framework/debug/ws-proxy?browser={browserInspectUri}",
```

Blazor WebAssembly アプリにカスタム アプリ ベース パスを使用する方法については、「<xref:blazor/host-and-deploy/index#app-base-path>」を参照してください。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

<h2 id="vscode">スタンドアロン Blazor WebAssembly のデバッグ</h2>

`.vscode` フォルダー内の VS Code 資産の構成の詳細については、「<xref:blazor/tooling>」の **Linux** オペレーティング システムのガイダンスを参照してください。

1. VS Code でスタンドアロンの Blazor WebAssembly アプリを開きます。

   デバッグを有効にするために追加の設定が必要であることを示す通知が表示される場合があります。

   > Blazor WebAssembly アプリケーションをデバッグするには、追加のセットアップが必要です。

   通知を受信した場合:

   * 最新の [C# for Visual Studio Code 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)がインストールされていることを確認します。 インストールされている拡張機能を確認するには、メニュー バーから **[表示]**  >  **[拡張機能]** を開くか、**アクティビティ** サイド バーにある **拡張機能** アイコンを選択します。
   * JavaScript のプレビュー デバッグが有効になっていることを確認します。 メニュー バーから設定を開きます ( **[ファイル]**  >  **[ユーザー設定]**  >  **[設定]** )。 キーワード「`debug preview`」を使用して検索します。 検索結果で、 **[デバッグ] > [JavaScript:Use Preview]\(JavaScript: 使用プレビュー\)** のチェック ボックスがオンになっていることを確認します。 プレビュー デバッグを有効にするオプションがない場合、VS Code の最新版にアップグレードするか、[JavaScript デバッガー拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-vscode.js-debug-nightly) (VS Code バージョン 1.46 以前) をインストールしてください。
   * ウィンドウを再度読み込みます。

1. <kbd>F5</kbd> キーボード ショートカットまたはメニュー項目を使用してデバッグを開始します。

   > [!NOTE]
   > **デバッグなしの開始** (<kbd>Ctrl</kbd>+<kbd>F5</kbd>) はサポートされていません。 アプリがデバッグ構成で実行されている場合、デバッグのオーバーヘッドによって常にパフォーマンスがわずかに低下します。

1. プロンプトが表示されたら、 **[Blazor WebAssembly Debug]** オプションを選択してデバッグを開始します。

1. スタンドアロン アプリが起動され、デバッグ ブラウザーが開きます。

1. "`*Client*`" アプリで、`Pages/Counter.razor` の `currentCount++;` 行にブレークポイントを設定します。

1. ブラウザーで `Counter` ページに移動し、 **[Click me]** ボタンを選択して、ブレークポイントで停止させます。

> [!NOTE]
> デバッグ プロキシが実行状態になる前のアプリの起動中には、ブレークポイントで停止することは **ありません**。 これには、`Program.Main` (`Program.cs`) のブレークポイントと、アプリから要求された最初のページによって読み込まれるコンポーネントの [`OnInitialized{Async}` メソッド](xref:blazor/components/lifecycle#component-initialization-methods)のブレークポイントが含まれます。

## <a name="debug-hosted-blazor-webassembly"></a>ホストされた Blazor WebAssembly のデバッグ

1. VS Code で、ホストされた Blazor WebAssembly アプリのソリューション フォルダーを開きます。

1. プロジェクトの起動構成が設定されていない場合は、次の通知が表示されます。 **[はい]** を選択します。

   > Required assets to build and debug are missing from '{APPLICATION NAME}'. (ビルドとデバッグに必要な資産が、'<アプリケーション名>' にありません。) 追加しますか?

   `.vscode` フォルダー内の VS Code 資産の構成の詳細については、「<xref:blazor/tooling>」の **Linux** オペレーティング システムのガイダンスを参照してください。

1. ウィンドウの上部にあるコマンド パレットで、ホストされているソリューション内の *Server* プロジェクトを選択します。

`launch.json` ファイルが、デバッガーを起動するための起動構成を使用して生成されます。

## <a name="attach-to-an-existing-debugging-session"></a>既存のデバッグセッションにアタッチする

実行中の Blazor アプリにアタッチするには、次のように構成された `launch.json` ファイルを作成します。

```json
{
  "type": "blazorwasm",
  "request": "attach",
  "name": "Attach to Existing Blazor WebAssembly Application"
}
```

> [!NOTE]
> デバッグ セッションへのアタッチは、スタンドアロン アプリでのみサポートされています。 フルスタック デバッグを使用するには、VS Code からアプリを起動する必要があります。

## <a name="launch-configuration-options"></a>起動構成のオプション

`blazorwasm` デバッグの種類では、次の起動構成オプションがサポートされています (`.vscode/launch.json`)。

| オプション    | 説明 |
| --------- | ----------- |
| `request` | `launch` を使用して、Blazor WebAssembly アプリにデバッグ セッションを起動してアタッチするか、`attach` を使用して既に実行中のアプリにデバッグ セッションをアタッチします。 |
| `url`     | デバッグ時にブラウザーで開く URL。 既定値は `https://localhost:5001` です。 |
| `browser` | デバッグ セッション用に起動するブラウザー。 `edge` または `chrome` に設定します。 既定値は `chrome` です。 |
| `trace`   | JS デバッガーからログを生成するために使用されます。 ログを生成するには `true` に設定します。 |
| `hosted`  | ホストされている Blazor WebAssembly アプリを起動およびデバッグする場合は、`true` に設定する必要があります。 |
| `webRoot` | Web サーバーの絶対パスを指定します。 アプリをサブルートから提供する場合は設定する必要があります。 |
| `timeout` | デバッグ セッションがアタッチされるのを待機するミリ秒単位の時間。 既定値は 30,000 ミリ秒 (30 秒) です。 |
| `program` | ホストされたアプリのサーバーを実行する実行可能ファイルへの参照。 `hosted` が `true` の場合に設定する必要があります。 |
| `cwd`     | アプリを起動する作業ディレクトリ。 `hosted` が `true` の場合に設定する必要があります。 |
| `env`     | 起動されたプロセスに提供する環境変数。 `hosted` が `true`に設定されている場合にのみ適用されます。 |

## <a name="example-launch-configurations"></a>起動構成の例

### <a name="launch-and-debug-a-standalone-blazor-webassembly-app"></a>スタンドアロン Blazor WebAssembly アプリを起動してデバッグする

```json
{
  "type": "blazorwasm",
  "request": "launch",
  "name": "Launch and Debug"
}
```

### <a name="attach-to-a-running-app-at-a-specified-url"></a>指定された URL で実行中のアプリにアタッチする

```json
{
  "type": "blazorwasm",
  "request": "attach",
  "name": "Attach and Debug",
  "url": "http://localhost:5000"
}
```

### <a name="launch-and-debug-a-hosted-blazor-webassembly-app-with-microsoft-edge"></a>Microsoft Edge でホストされている Blazor WebAssembly アプリを起動してデバッグする

ブラウザーの構成の既定値は Google Chrome です。 デバッグに Microsoft Edge を使用する場合は、`browser` を `edge` に設定します。 Google Chrome を使用するには、`browser` オプションを設定しないか、オプションの値を `chrome` に設定します。

```json
{
  "name": "Launch and Debug Hosted Blazor WebAssembly App",
  "type": "blazorwasm",
  "request": "launch",
  "hosted": true,
  "program": "${workspaceFolder}/Server/bin/Debug/netcoreapp3.1/MyHostedApp.Server.dll",
  "cwd": "${workspaceFolder}/Server",
  "browser": "edge"
}
```

前の例では、`MyHostedApp.Server.dll` は *Server* アプリのアセンブリです。 `.vscode` フォルダーは、`Client`、`Server`、および `Shared` フォルダーの隣にあるソリューションのフォルダー内にあります。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

Visual Studio for Mac で Blazor WebAssembly アプリをデバッグするには:

1. 新しい ASP.NET Core でホストされる Blazor WebAssembly アプリを作成します。
1. <kbd>&#8984;</kbd> + <kbd>&#8617;</kbd> キーを押して、デバッガーでアプリを実行します。

   > [!NOTE]
   > **デバッグなしの開始** (<kbd>&#8997;</kbd> + <kbd>&#8984;</kbd> + <kbd>&#8617;</kbd>) はサポートされていません。 アプリがデバッグ構成で実行されている場合、デバッグのオーバーヘッドによって常にパフォーマンスがわずかに低下します。

   > [!IMPORTANT]
   > デバッグ セッション用のブラウザーとしては、Google Chrome または Microsoft Edge を選択する必要があります。

1. "`*Client*`" アプリで、`Pages/Counter.razor` の `currentCount++;` 行にブレークポイントを設定します。
1. ブラウザーで `Counter` ページに移動し、 **[Click me]** ボタンを選択して、ブレークポイントで停止させます。
1. Visual Studio の **[ローカル]** ウィンドウで、`currentCount` フィールドの値を調べます。
1. <kbd>&#8984;</kbd> + <kbd>&#8617;</kbd> キーを押して、実行を続けます。

Blazor WebAssembly アプリをデバッグしている間に、サーバー コードをデバッグすることもできます。

1. `Pages/FetchData.razor` ページの <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> にブレークポイントを設定します。
1. `Get` アクション メソッドの `WeatherForecastController` にブレークポイントを設定します。
1. `Fetch Data` ページに移動し、サーバーに HTTP 要求を発行する直前の、`FetchData` コンポーネント内の最初のブレークポイントで停止させます。
1. <kbd>&#8984;</kbd> + <kbd>&#8617;</kbd> キーを押して実行を続け、サーバーの `WeatherForecastController` 内のブレークポイントで停止させます。
1. <kbd>&#8984;</kbd> + <kbd>&#8617;</kbd> キーをもう一度押して実行を続け、天気予報テーブルがブラウザーにレンダリングされることを確認します。

> [!NOTE]
> デバッグ プロキシが実行状態になる前のアプリの起動中には、ブレークポイントで停止することは **ありません**。 これには、`Program.Main` (`Program.cs`) のブレークポイントと、アプリから要求された最初のページによって読み込まれるコンポーネントの [`OnInitialized{Async}` メソッド](xref:blazor/components/lifecycle#component-initialization-methods)のブレークポイントが含まれます。

詳細については、「[Visual Studio for Mac を使用したデバッグ](/visualstudio/mac/debugging)」を参照してください。

---

## <a name="debug-in-the-browser"></a>ブラウザーでデバッグする

*このセクションのガイダンスは、Windows で実行されている Google Chrome と Microsoft Edge に適用されます。*

1. 開発環境でアプリのデバッグ ビルドを実行します。

1. ブラウザーを起動し、アプリの URL (`https://localhost:5001` など) に移動します。

1. ブラウザーで <kbd>Shift</kbd> + <kbd>Alt</kbd> + <kbd>D</kbd> キーを押すことにより、リモート デバッグの開始を試みます。

   ブラウザーはリモート デバッグが有効で実行されている必要があります。これは既定ではありません。 リモート デバッグが無効になっている場合、"**デバッグ可能なブラウザー タブが見つからない**" というエラー ページと、デバッグ ポートを開いた状態でブラウザーを起動する手順がレンダリングされます。 ブラウザーの指示に従って、新しいブラウザー ウィンドウを開きます。 前のブラウザー ウィンドウを閉じます。

<!-- HOLD 
1. In the browser, attempt to commence remote debugging by pressing:

   * <kbd>Shift</kbd>+<kbd>Alt</kbd>+<kbd>d</kbd> on Windows.
   * <kbd>Shift</kbd>+<kbd>&#8984;</kbd>+<kbd>d</kbd> on macOS.

   The browser must be running with remote debugging enabled, which isn't the default. If remote debugging is disabled, an **Unable to find debuggable browser tab** error page is rendered with instructions for launching the browser with the debugging port open. Follow the instructions for your browser, which opens a new browser window. Close the previous browser window.
-->

1. リモート デバッグが有効な状態でブラウザーが実行されていると、前のステップのデバッグ キーボード ショートカットにより、新しいデバッガー タブが開きます。

1. 少しすると、 **[ソース]** タブに、`file://` ノード内にあるアプリの .NET アセンブリの一覧が表示されます。

1. コンポーネント コード (`.razor` ファイル) と C# コード ファイル (`.cs`) で、設定したブレークポイントがコードの実行時にヒットします。 ブレークポイントにヒットした後、コード全体をステップ実行する (<kbd>F10</kbd>) か、コードの実行を普通に再開 (<kbd>F8</kbd>) します。

Blazor は、[Chrome DevTools プロトコル](https://chromedevtools.github.io/devtools-protocol/)を実装し、.NET 固有の情報によってプロトコルを拡張するデバッグ プロキシを備えています。 デバッグ用のキーボード ショートカットが押されると、Blazor はプロキシで Chrome DevTools を指し示します。 プロキシは、デバッグしようとしているブラウザー ウィンドウに接続します (そのためリモート デバッグを有効にする必要があります)。

## <a name="browser-source-maps"></a>ブラウザー ソース マップ

ブラウザー ソース マップを使用すると、ブラウザーのコンパイルされたファイルを元のソース ファイルにマップし直すことができ、これはクライアント側のデバッグによく使用されます。 ただし Blazor では現在、C# は JavaScript/WASM に直接マップされません。 その代わり、Blazor はブラウザー内で中間言語の解釈を行うため、ソース マップは関係がありません。

## <a name="firewall-configuration"></a>ファイアウォールの構成

ファイアウォールでデバッグ プロキシとの通信がブロックされている場合は、ブラウザーと `NodeJS` プロセス間の通信を許可するファイアウォールの例外規則を作成します。

> [!WARNING]
> セキュリティの脆弱性が発生しないようにするには、ファイアウォールの構成を変更する必要があります。 セキュリティ ガイダンスを慎重に適用し、セキュリティに関するベスト プラクティスに従い、ファイアウォールの製造元から発行された警告を順守します。
>
> `NodeJS` プロセスとのオープンな通信を許可する:
>
> * ファイアウォールの機能と構成に応じて、Node サーバーが開いて接続されます。
> * ネットワークによっては危険な場合があります。
> * **開発者のマシンでのみ行うことをお勧めします。**
>
> 可能な場合は、**信頼されているネットワークまたはプライベート ネットワーク上の** `NodeJS` プロセスとのオープン通信のみを許可します。

[Windows ファイアウォール](/windows/security/threat-protection/windows-firewall/windows-firewall-with-advanced-security)の構成ガイダンスについては、[受信プログラムまたはサービス規則の作成](/windows/security/threat-protection/windows-firewall/create-an-inbound-program-or-service-rule)に関する記事をご覧ください。 詳細については、Windows ファイアウォールのドキュメント セットで[高度なセキュリティを備えた Windows Defender ファイアウォール](/windows/security/threat-protection/windows-firewall/windows-firewall-with-advanced-security)に関連する記事をご覧ください。

## <a name="troubleshoot"></a>トラブルシューティング

エラーが発生している場合は、次のヒントが役立つことがあります。

* **[デバッガー]** タブで、ブラウザー内の開発者ツールを開きます。 コンソールで `localStorage.clear()` を実行して、任意のブレークポイントを削除します。
* ASP.NET Core HTTPS 開発証明書がインストールされ、信頼されていることを確認します。 詳細については、「<xref:security/enforcing-ssl#troubleshoot-certificate-problems>」を参照してください。
* Visual Studio では、 **[ツール]**  >  **[オプション]**  >  **[デバッグ]**  >  **[全般]** で **[Enable JavaScript debugging for ASP.NET (Chrome, Edge and IE)] (ASP.NET (Chrome、Edge、IE) の JavaScript デバッグを有効にする)** オプションをオンにする必要があります。 これは、Visual Studio の既定の設定です。 デバッグが機能していない場合は、オプションがオンになっていることを確認します。
* ご利用の環境で HTTP プロキシを使用している場合は、プロキシ バイパス設定に `localhost` が含まれていることを確認してください。 これを行うには、次のいずれかに `NO_PROXY` 環境変数を設定します。
  * プロジェクトの `launchSettings.json` ファイル。
  * すべてのアプリに適用する場合は、ユーザーまたはシステム環境変数レベルで。 環境変数を使用する場合は、Visual Studio を再起動して変更を有効にしてください。
* ファイアウォールまたはプロキシでデバッグ プロキシ (`NodeJS` プロセス) との通信がブロックされていないことを確認します。 詳細については、「[ファイアウォールの構成](#firewall-configuration)」セクションをご覧ください。

### <a name="breakpoints-in-oninitializedasync-not-hit"></a>`OnInitialized{Async}` 内のブレークポイントにヒットしない

Blazor フレームワークのデバッグ プロキシは、起動に少し時間がかかります。そのため、[`OnInitialized{Async}` ライフサイクル メソッド](xref:blazor/components/lifecycle#component-initialization-methods)内のブレークポイントにヒットしない可能性があります。 デバッグ プロキシが起動してブレークポイントにヒットするまでの時間を確保するために、メソッド本体の開始に対して遅延を追加することをお勧めします。 [`if` コンパイラ ディレクティブ](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-if)に基づいて遅延を含めると、アプリのリリース ビルドに対しては遅延が存在しないようにすることができます。

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A>:

```csharp
protected override void OnInitialized()
{
#if DEBUG
    Thread.Sleep(10000)
#endif

    ...
}
```

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>:

```csharp
protected override async Task OnInitializedAsync()
{
#if DEBUG
    await Task.Delay(10000)
#endif

    ...
}
```

### <a name="visual-studio-windows-timeout"></a>Visual Studio (Windows) のタイムアウト

Visual Studio で、タイムアウトに達したことを示すデバッグ アダプターの起動失敗の例外がスローされた場合、レジストリ設定でタイムアウトを調整できます。

```console
VsRegEdit.exe set "<VSInstallFolder>" HKCU JSDebugger\Options\Debugging "BlazorTimeoutInMilliseconds" dword {TIMEOUT}
```

上記のコマンドの `{TIMEOUT}` プレースホルダーはミリ秒単位です。 たとえば、1 分は `60000` として割り当てられます。
