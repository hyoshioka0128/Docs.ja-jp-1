---
title: Visual Studio の ASP.NET Core で LibMan を使用する
author: rick-anderson
description: Visual Studio の ASP.NET Core プロジェクトで LibMan を使用する方法について説明します。
ms.author: scaddie
ms.custom: mvc
ms.date: 08/20/2018
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
uid: client-side/libman/libman-vs
ms.openlocfilehash: 8aabdd3fc8b4e9ebeabec482514768d23a152263
ms.sourcegitcommit: 0abfe496fed8e9470037c8128efa8a50069ccd52
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/07/2021
ms.locfileid: "106563877"
---
# <a name="use-libman-with-aspnet-core-in-visual-studio"></a>Visual Studio の ASP.NET Core で LibMan を使用する

作成者: [Scott Addie](https://twitter.com/Scott_Addie)

Visual Studio には、ASP.NET Core プロジェクトでの [LibMan](xref:client-side/libman/index) に対する次のようなサポートが組み込まれています。

* ビルド時に LibMan の復元操作を構成して実行するためのサポート。
* LibMan の復元操作とクリーン操作をトリガーするためのメニュー項目。
* ライブラリを検索し、プロジェクトにファイルを追加するための検索ダイアログ。
* LibMan マニフェスト ファイル &mdash; *libman.json* の編集サポート。

[サンプル コードを表示またはダウンロードします](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/client-side/libman/samples/) [(ダウンロード方法)](xref:index#how-to-download-a-sample)。

## <a name="prerequisites"></a>必須コンポーネント

* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) と **ASP.NET と Web 開発** ワークロード

## <a name="add-library-files"></a>ライブラリ ファイルを追加する

ライブラリ ファイルは、次の 2 つの方法で ASP.NET Core プロジェクトに追加できます。

1. [[クライアント側のライブラリを追加します] ダイアログを使用する](#use-the-add-client-side-library-dialog)
1. [LibMan マニフェスト ファイル エントリを手動で構成する](#manually-configure-libman-manifest-file-entries)

### <a name="use-the-add-client-side-library-dialog"></a>[クライアント側のライブラリを追加します] ダイアログを使用する

クライアント側ライブラリをインストールするには、次の手順に従います。

* **ソリューション エクスプローラー** で、ファイルを追加するプロジェクト フォルダーを右クリックします。 **[追加]**  >  **[クライアント側のライブラリ]** を選択します。 **[クライアント側のライブラリを追加します]** ダイアログが表示されます。

  ![[クライアント側のライブラリを追加します] ダイアログ](_static/add-library-dialog.png)

* **[プロバイダー]** ドロップ ダウンからライブラリ プロバイダーを選択します。 CDNJS が既定のプロバイダーです。
* **[ライブラリ]** テキスト ボックスに、フェッチするライブラリ名を入力します。 IntelliSense により、指定したテキストで始まるライブラリの一覧が表示されます。
* IntelliSense の一覧からライブラリを選択します。 ライブラリ名には、`@` 記号と、選択したプロバイダーによって認識される最新の安定バージョンがサフィックスとして付けられています。
* 含めるファイルを決定します。
  * すべてのライブラリ ファイルを含めるには、 **[Include all library files]\(すべてのライブラリ ファイルを含める\)** ラジオ ボタンを選択します。
  * ライブラリのファイルのサブセットを含めるには、 **[Choose specific files]\(特定のファイルを選択する\)** ラジオ ボタンを選択します。 ラジオ ボタンを選択すると、ファイル セレクター ツリーが有効になります。 ダウンロードするファイルの名前の左側にあるボックスをオンにします。
* **[ターゲットの場所]** テキスト ボックスで、ファイルを格納するプロジェクト フォルダーを指定します。 各ライブラリを別々のフォルダーに保存することをお勧めします。

  **[ターゲットの場所]** として提案されるフォルダーは、ダイアログを起動した場所に基づいています。

  * プロジェクト ルートから起動した場合は、次のようになります。
    * *wwwroot* が存在する場合は、*wwwroot/lib* が使用されます。
    * *wwwroot* が存在しない場合は、*lib* が使用されます。
  * プロジェクト フォルダーから起動した場合は、対応するフォルダー名が使用されます。

  提案されるフォルダーには、ライブラリ名がサフィックスとして付けられます。 次の表は、Razor Pages プロジェクトに jQuery をインストールするときに提案されるフォルダーを示しています。
  
  |起動場所                           |提案されるフォルダー      |
  |------------------------------------------|----------------------|
  |プロジェクト ルート (*wwwroot* が存在する場合)        |*wwwroot/lib/jquery/* |
  |プロジェクト ルート (*wwwroot* が存在しない場合) |*lib/jquery/*         |
  |プロジェクトの *Pages* フォルダー                 |*Pages/jquery/*       |

* **[インストール]** ボタンをクリックして、*libman. json* の構成に従ってファイルをダウンロードします。
* インストールの詳細については、 **[出力]** ウィンドウの **[ライブラリ マネージャー]** フィードを確認してください。 次に例を示します。

  ```console
  Restore operation started...
  Restoring libraries for project LibManSample
  Restoring library jquery@3.3.1... (LibManSample)
  wwwroot/lib/jquery/jquery.min.js written to destination (LibManSample)
  wwwroot/lib/jquery/jquery.js written to destination (LibManSample)
  wwwroot/lib/jquery/jquery.min.map written to destination (LibManSample)
  Restore operation completed
  1 libraries restored in 2.32 seconds
  ```

### <a name="manually-configure-libman-manifest-file-entries"></a>LibMan マニフェスト ファイル エントリを手動で構成する

Visual Studio のすべての LibMan 操作は、プロジェクト ルートの LibMan マニフェスト (*libman. json*) のコンテンツに基づいています。 *libman. json* を手動で編集して、プロジェクトのライブラリ ファイルを構成できます。 *libman. json* が保存されると、Visual Studio ではすべてのライブラリ ファイルが復元されます。

編集のために *libman. json* を開くには、次のオプションがあります。

* **ソリューション エクスプローラー** で *libman. json* ファイルをダブルクリックします。
* **ソリューション エクスプローラー** でプロジェクトを右クリックし、 **[クライアント側のライブラリを管理する]** を選択します。 **&#8224;**
* Visual Studio の **[プロジェクト]** メニューで、 **[クライアント側のライブラリを管理する]** を選択します。 **&#8224;**

**&#8224;** *libman. json* ファイルがプロジェクト ルートにまだ存在しない場合は、既定の項目テンプレート コンテンツを使用して作成されます。

Visual Studio には、色付け、書式設定、IntelliSense、スキーマ検証などの豊富な JSON 編集サポートが用意されています。 LibMan マニフェストの JSON スキーマは、[https://json.schemastore.org/libman](https://json.schemastore.org/libman) にあります。

次のマニフェスト ファイルを使用すると、LibMan では、`libraries` プロパティに定義されている構成に従ってファイルが取得されます。 `libraries` 内で定義されているオブジェクト リテラルの説明を次に示します。

* [jQuery](https://jquery.com/) バージョン 3.3.1 のサブセットは、CDNJS プロバイダーから取得されます。 このサブセットは、`files` プロパティ &mdash; *jquery.min.js*、*jquery.js*、および *jquery.min.map* で定義されます。 これらのファイルは、プロジェクトの *wwwroot/lib/jquery* フォルダーに格納されます。
* [ブートストラップ](https://getbootstrap.com/) バージョン 4.1.3 の全体が取得され、*wwwroot/lib/bootstrap* フォルダーに格納されます。 オブジェクト リテラルの `provider` プロパティによって `defaultProvider` プロパティ値がオーバーライドされます。 LibMan により、unpkg プロバイダーからブートストラップ ファイルが取得されます。
* [Lodash](https://lodash.com/) のサブセットは、組織内の管理機関によって承認されています。 *lodash.js* ファイルと *lodash.min.js* ファイルは、*C:\\temp\\lodash\\* のローカル ファイル システムから取得されます。 これらのファイルは、プロジェクトの *wwwroot/lib/lodash* フォルダーにコピーされます。

[!code-json[](samples/LibManSample/libman.json)]

> [!NOTE]
> LibMan では、各プロバイダーのライブラリごとに 1 つのバージョンのみがサポートされます。 *libman. json* ファイルでは、特定のプロバイダーに対して同じライブラリ名を持つ 2 つのライブラリが含まれている場合、スキーマの検証に失敗します。

## <a name="restore-library-files"></a>ライブラリ ファイルを復元する

Visual Studio 内からライブラリ ファイルを復元するには、プロジェクト ルートに有効な *libman. json* ファイルが存在する必要があります。 復元されたファイルは、各ライブラリに指定された場所にあるプロジェクトに配置されます。

ライブラリ ファイルは、ASP.NET Core プロジェクトで次の 2 つの方法で復元できます。

1. [ビルド時にファイルを復元する](#restore-files-during-build)
1. [ファイルを手動で復元する](#restore-files-manually)

### <a name="restore-files-during-build"></a>ビルド時にファイルを復元する

LibMan では、ビルド プロセスの一環として、定義済みのライブラリ ファイルを復元できます。 既定では、"*ビルド時の復元*" 動作は無効になっています。

ビルド時の復元動作を有効にしてテストするには、次の手順に従います。

* **ソリューション エクスプローラー** で *[libman. json]* を右クリックし、コンテキスト メニューで **[ビルド時にクライアント側のライブラリの復元を有効にする]** をオンにします。
* NuGet パッケージのインストールを求めるメッセージが表示されたら、 **[はい]** ボタンをクリックします。 プロジェクトに [Microsoft.Web.LibraryManager.Build](https://www.nuget.org/packages/Microsoft.Web.LibraryManager.Build/) NuGet パッケージが追加されます。

  [!code-xml[](samples/LibManSample/LibManSample.csproj?name=snippet_RestoreOnBuildPackage)]

* プロジェクトをビルドして、LibMan ファイルの復元が実行されることを確認します。 `Microsoft.Web.LibraryManager.Build` パッケージにより、プロジェクトのビルド操作中に LibMan を実行する MSBuild ターゲットが挿入されます。
* **[出力]** ウィンドウの **[ビルド]** フィードで LibMan アクティビティ ログを確認します。

  ```console
  1>------ Build started: Project: LibManSample, Configuration: Debug Any CPU ------
  1>
  1>Restore operation started...
  1>Restoring library jquery@3.3.1...
  1>Restoring library bootstrap@4.1.3...
  1>
  1>2 libraries restored in 10.66 seconds
  1>LibManSample -> C:\LibManSample\bin\Debug\netcoreapp2.1\LibManSample.dll
  ========== Build: 1 succeeded, 0 failed, 0 up-to-date, 0 skipped ==========
  ```

ビルド時の復元動作が有効になっている場合は、 *[libman. json]* コンテキスト メニューに **[Disable Restore Client-Side Libraries on Build]\(ビルド時にクライアント側のライブラリの復元を無効にする\)** オプションが表示されます。 このオプションを選択すると、プロジェクト ファイルから `Microsoft.Web.LibraryManager.Build` パッケージ参照が削除されます。 その結果、各ビルド時にクライアント側ライブラリは復元されなくなります。

ビルド時の復元設定に関係なく、*libman. json* コンテキスト メニューからいつでも手動で復元を行うことができます。 詳細については、「[ファイルを手動で復元する](#restore-files-manually)」を参照してください。

### <a name="restore-files-manually"></a>ファイルを手動で復元する

ライブラリ ファイルを手動で復元するには、次の手順に従います。

* ソリューション内のすべてのプロジェクトについて:
  * **ソリューション エクスプローラー** でソリューション名を右クリックします。
  * **[クライアント側のライブラリを復元する]** オプションを選択します。
* 特定のプロジェクトについて:
  * **ソリューション エクスプローラー** で *libman. json* ファイルを右クリックします。
  * **[クライアント側のライブラリを復元する]** オプションを選択します。

復元操作の実行時には、次の処理が行われます。

* Visual Studio のステータス バーのタスク ステータス センター (TSC) アイコンがアニメーション化され、 *[復元操作が開始されました]* と表示されます。 このアイコンをクリックすると、既知のバックグラウンド タスクを示すヒントが表示されます。
* ステータス バーと、 **[出力]** ウィンドウの **[ライブラリ マネージャー]** フィードに、メッセージが送信されます。 次に例を示します。

  ```console
  Restore operation started...
  Restoring libraries for project LibManSample
  Restoring library jquery@3.3.1... (LibManSample)
  wwwroot/lib/jquery/jquery.min.js written to destination (LibManSample)
  wwwroot/lib/jquery/jquery.js written to destination (LibManSample)
  wwwroot/lib/jquery/jquery.min.map written to destination (LibManSample)
  Restore operation completed
  1 libraries restored in 2.32 seconds
  ```

## <a name="delete-library-files"></a>ライブラリ ファイルを削除する

Visual Studio で以前に復元されたライブラリ ファイルを削除するために "*クリーン*" 操作を実行するには、次の手順に従います。

* **ソリューション エクスプローラー** で *libman. json* ファイルを右クリックします。
* **[クライアント側のライブラリをクリーンアップする]** オプションを選択します。

ライブラリ ファイル以外が誤って削除されないように、クリーン操作ではディレクトリ全体が削除されることはありません。 以前の復元に含まれていたファイルのみが削除されます。

クリーン操作の実行時には、次の処理が行われます。

* Visual Studio の TSC アイコンがアニメーション化され、 *[ライブラリのクリーン操作が開始されました]* と表示されます。 このアイコンをクリックすると、既知のバックグラウンド タスクを示すヒントが表示されます。
* ステータス バーと、 **[出力]** ウィンドウの **[ライブラリ マネージャー]** フィードに、メッセージが送信されます。 次に例を示します。

```console
Clean libraries operation started...
Clean libraries operation completed
2 libraries were successfully deleted in 1.91 secs
```

クリーン操作では、プロジェクトからファイルのみが削除されます。 ライブラリ ファイルは、今後の復元操作で速やかに取得できるようにキャッシュに残っています。 ローカル コンピューターのキャッシュに格納されているライブラリ ファイルを管理するには、[LibMan CLI](xref:client-side/libman/libman-cli) を使用します。

## <a name="uninstall-library-files"></a>ライブラリ ファイルをアンインストールする

ライブラリ ファイルをアンインストールするには、次の手順に従います。

* *libman. json* を開きます。
* 対応する `libraries` オブジェクト リテラル内にカレットを配置します。
* 左余白に表示される電球アイコンをクリックし、 **[\<library_name>@\<library_version> のアンインストール]** を選択します。

  ![ライブラリのアンインストール コンテキスト メニュー オプション](_static/uninstall-menu-option.png)

または、LibMan マニフェスト (*libman. json*) を手動で編集して保存することもできます。 ファイルが保存されると、[復元操作](#restore-library-files)が実行されます。 *libman. json* で定義されなくなったライブラリ ファイルは、プロジェクトから削除されます。

## <a name="update-library-version"></a>ライブラリ バージョンを更新する

更新されたライブラリ バージョンを確認するには、次の手順に従います。

* *libman. json* を開きます。
* 対応する `libraries` オブジェクト リテラル内にカレットを配置します。
* 左余白に表示される電球アイコンをクリックします。 **[更新プログラムの確認]** にカーソルを合わせます。

LibMan により、インストールされているバージョンよりも新しいライブラリ バージョンがないか確認されます。 次の結果が生じる可能性があります。

* 最新バージョンが既にインストールされている場合は、 **[更新プログラムが見つかりませんでした]** メッセージが表示されます。
* まだインストールされていない場合は、最新の安定バージョンが表示されます。

  ![[更新プログラムの確認] コンテキスト メニュー オプション](_static/update-menu-option.png)

* インストールされているバージョンよりも新しいプレリリースを使用できる場合は、プレリリースが表示されます。

古いライブラリ バージョンにダウングレードするには、*libman. json* ファイルを手動で編集します。 ファイルが保存されると、LibMan [復元操作](#restore-library-files)によって次の処理が行われます。

* 以前のバージョンから冗長なファイルが削除されます。
* 新しいバージョンから新しいファイルと更新済みのファイルが追加されます。

## <a name="additional-resources"></a>その他の技術情報

* <xref:client-side/libman/libman-cli>
* [LibMan の GitHub リポジトリ](https://github.com/aspnet/LibraryManager)
