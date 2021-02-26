---
title: ASP.NET Core Blazor 用のツール
author: guardrex
description: Blazor アプリのビルドに使用できるツールについて説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/11/2021
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
uid: blazor/tooling
zone_pivot_groups: operating-systems
ms.openlocfilehash: 6b61d9a4645d273b0c78fae0388d569771c43a2d
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100536247"
---
# <a name="tooling-for-aspnet-core-blazor"></a>ASP.NET Core Blazor 用のツール

::: zone pivot="windows"

1. **ASP.NET および Web 開発** ワークロードと共に、最新バージョンの [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) をインストールする

1. 新しいプロジェクトを作成します。

1. **[Blazor アプリ]** を選択します。 **[次へ]** を選択します。

1. **[プロジェクト名]** フィールドにプロジェクト名を入力するか、既定のプロジェクト名をそのまま使用します。 **[場所]** エントリが正しいことを確認します。または、プロジェクトの場所を指定します。 **[作成]** を選択します。

1. Blazor WebAssembly エクスペリエンスの場合は、 **Blazor WebAssembly アプリ** テンプレートを選択します。 Blazor Server エクスペリエンスの場合は、 **Blazor Server アプリ** テンプレートを選択します。 **[作成]** を選択します。

   ホステッド Blazor WebAssembly エクスペリエンスの場合は、 **[ASP.NET Core hosted]\(ASP.NET Core ホステッド\)** チェック ボックスをオンにします。

   2 つの Blazor ホスティング モデル、 *Blazor WebAssembly* (スタンドアロンとホステッド) と *Blazor Server* については、「<xref:blazor/hosting-models>」を参照してください。

1. <kbd>Ctrl</kbd>+<kbd>F5</kbd> キーを押してアプリを実行します。

ASP.NET Core HTTPS 開発証明書の信頼の詳細については、「<xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos>」を参照してください。

ホストされている Blazor WebAssembly アプリを実行する場合は、ソリューションの **`Server`** プロジェクトから、そのアプリを実行します。

::: zone-end

::: zone pivot="linux"

1. 最新バージョンの [.NET Core SDK](https://dotnet.microsoft.com/download) をインストールします。 以前に SDK をインストールした場合、コマンド シェルでコマンドを実行し、インストールされているバージョンを判断できます。

   ```dotnetcli
   dotnet --version
   ```

1. [Visual Studio Code](https://code.visualstudio.com) の最新版をインストールします。

1. 最新の [C# for Visual Studio Code 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)をインストールします。

1. Blazor WebAssembly エクスペリエンスの場合は、コマンド シェルで次のコマンドを実行します。

   ```dotnetcli
   dotnet new blazorwasm -o WebApplication1
   ```

   ホステッド Blazor WebAssembly エクスペリエンスの場合は、ホステッド オプション (`-ho` または `--hosted`) をコマンドに追加します。

   ```dotnetcli
   dotnet new blazorwasm -o WebApplication1 -ho
   ```

   Blazor Server エクスペリエンスの場合は、コマンド シェルで次のコマンドを実行します。

   ```dotnetcli
   dotnet new blazorserver -o WebApplication1
   ```

   2 つの Blazor ホスティング モデル、 *Blazor WebAssembly* (スタンドアロンとホステッド) と *Blazor Server* については、「<xref:blazor/hosting-models>」を参照してください。

1. Visual Studio Code で `WebApplication1` フォルダーを開きます。

1. IDE によって、プロジェクトをビルドおよびデバッグするためにアセットを追加するよう要求されます。 **[はい]** を選択します。

   **ホストされている Blazor WebAssembly の起動とタスクの構成**

   ホストされている Blazor WebAssembly ソリューションについては、`launch.json` ファイルと `tasks.json` ファイルを含む `.vscode` フォルダーを追加 (または移動) します。このフォルダーは、`Client`、`Server`、`Shared` という、通常のプロジェクト フォルダー名を含むものです。 **`Server`** プロジェクトでホストされている Blazor WebAssembly アプリが `launch.json` ファイルと `tasks.json` ファイルの構成によって実行されるように更新または確認します。

   **`.vscode/launch.json`** (`launch` の構成):

   ```json
   ...
   "cwd": "${workspaceFolder}/{SERVER APP FOLDER}",
   ...
   ```

   現在の作業ディレクトリの前の構成 (`cwd`) では、`{SERVER APP FOLDER}` プレースホルダーは **`Server`** プロジェクトのフォルダーです (通常は "`Server`")。

   システム上で Microsoft Edge が使用されており、Google Chrome がインストールされていない場合は、`"browser": "edge"` の追加のプロパティを構成に追加します。

   `Server` のプロジェクト フォルダー、およびそれによって、既定のブラウザーである Google Chrome ではなく、Microsoft Edge がデバッグ実行用のブラウザーとして生成される例を次に示します。

   ```json
   ...
   "cwd": "${workspaceFolder}/Server",
   "browser": "edge"
   ...
   ```

   **`.vscode/tasks.json`** ([`dotnet` コマンド](/dotnet/core/tools/dotnet)引数):

   ```json
   ...
   "${workspaceFolder}/{SERVER APP FOLDER}/{PROJECT NAME}.csproj",
   ...
   ```

   前の引数は次のとおりです。

   * `{SERVER APP FOLDER}` プレースホルダーは **`Server`** プロジェクトのフォルダーです (通常は "`Server`")。
   * `{PROJECT NAME}` プレースホルダーはアプリの名前です。通常、この名前は、Blazor プロジェクト テンプレートから生成されたアプリ内のソリューションの名前とそれに続く "`.Server`" から付けられます。

   [Blazor WebAssembly アプリでの SignalR の使用に関するチュートリアル](xref:tutorials/signalr-blazor)の次の例では、`Server` のプロジェクト フォルダー名と `BlazorWebAssemblySignalRApp.Server` のプロジェクト名を使用します。

   ```json
   ...
   "args": [
     "build",
       "${workspaceFolder}/Server/BlazorWebAssemblySignalRApp.Server.csproj",
       "/property:GenerateFullPaths=true",
       "/consoleloggerparameters:NoSummary"
   ],
   ...
   ```

1. <kbd>Ctrl</kbd>+<kbd>F5</kbd> キーを押してアプリを実行します。

## <a name="trust-a-development-certificate"></a>開発証明書を信頼する

Linux で証明書を信頼するための一元的な方法はありません。 通常、次のいずれかの方法が採用されます。

* ブラウザーの除外リストでアプリの URL を除外します。
* `localhost` に対するすべての自己署名証明書を信頼します。
* ブラウザーの信頼された証明書の一覧に証明書を追加します。

詳細については、ご利用のブラウザーの製造元および Linux ディストリビューションによって提供されているガイダンスを参照してください。

::: zone-end

::: zone pivot="macos"

1. [Visual Studio for Mac をインストールします](https://visualstudio.microsoft.com/vs/mac/)。

1. **[ファイル]**  >  **[新しいソリューション]** を選択するか、 **[スタート ウィンドウ]** から **新しい** プロジェクトを作成します。

1. サイドバーで、 **[Web and Console]\(Web とコンソール\)**  >  **[アプリ]** の順に選択します。

   Blazor WebAssembly エクスペリエンスの場合は、 **Blazor WebAssembly アプリ** テンプレートを選択します。 Blazor Server エクスペリエンスの場合は、 **Blazor Server アプリ** テンプレートを選択します。 **[次へ]** を選択します。

   2 つの Blazor ホスティング モデル、 *Blazor WebAssembly* (スタンドアロンとホステッド) と *Blazor Server* については、「<xref:blazor/hosting-models>」を参照してください。

1. **[認証]** に **[認証なし]** が設定されていることを確認します。 **[次へ]** を選択します。

1. ホステッド Blazor WebAssembly エクスペリエンスの場合は、 **[ASP.NET Core hosted]\(ASP.NET Core ホステッド\)** チェック ボックスをオンにします。

1. **[プロジェクト名]** フィールドで、アプリに `WebApplication1` という名前を付けます。 **[作成]** を選択します。

1. *デバッガーを使用せずに* アプリを実行するには、 **[実行]**  >  **[デバッグなしで開始]** の順に選択します。 **[実行]**  > 、 **[デバッグの開始]** か [実行] (&#9654;) ボタンでアプリを実行すると、"*デバッガーなしで*" アプリが実行されます。

開発証明書を信頼することを求めるメッセージが表示されたら、証明書を信頼して続行します。 証明書を信頼するには、ユーザーとキーチェーンのパスワードが必要です。 ASP.NET Core HTTPS 開発証明書の信頼の詳細については、「<xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos>」を参照してください。

ホストされている Blazor WebAssembly アプリを実行する場合は、ソリューションの **`Server`** プロジェクトから、そのアプリを実行します。

::: zone-end

## <a name="use-visual-studio-code-for-cross-platform-blazor-development"></a>クロスプラットフォームの Blazor 開発に Visual Studio Code を使用する

[Visual Studio Code](https://code.visualstudio.com/) は、Blazor アプリの開発に使用できるオープン ソースのクロスプラットフォーム統合開発環境 (IDE) です。 .NET CLI を使用して、Visual Studio Code で開発用の新しい Blazor アプリを作成します。 詳細については、[この記事の Linux バージョン](?pivots=linux)を参照してください。

## <a name="blazor-template-options"></a>Blazor テンプレート オプション

Blazor フレームワークには、Blazor ホスティング モデルのそれぞれに対して新しいアプリを作成するためのテンプレートが用意されています。 テンプレートは、Blazor 開発用に選択したツール (Visual Studio、Visual Studio for Mac、Visual Studio Code、または .NET CLI) に関係なく、新しい Blazor プロジェクトとソリューションを作成するために使用されます。

* Blazor WebAssembly プロジェクト テンプレート: `blazorwasm`
* Blazor Server プロジェクト テンプレート: `blazorserver`

Blazor のホスティング モデルの詳細については、「<xref:blazor/hosting-models>」を参照してください。

テンプレート オプションを使用するには、コマンド シェルで [`dotnet new`](/dotnet/core/tools/dotnet-new) CLI コマンドにヘルプ オプション (`-h` または `--help`) を渡します。

```dotnetcli
dotnet new blazorwasm -h
dotnet new blazorserver -h
```
