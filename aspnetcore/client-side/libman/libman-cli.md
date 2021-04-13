---
title: ASP.NET Core で LibMan CLI を使用する
author: rick-anderson
description: ASP.NET Core プロジェクトで LibMan CLI を使用する方法について説明します。
ms.author: scaddie
ms.custom: mvc
ms.date: 11/12/2019
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
uid: client-side/libman/libman-cli
ms.openlocfilehash: 966455ea67a880e369658c34253335521aa5c321
ms.sourcegitcommit: 0abfe496fed8e9470037c8128efa8a50069ccd52
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/07/2021
ms.locfileid: "106563655"
---
# <a name="use-the-libman-cli-with-aspnet-core"></a>ASP.NET Core で LibMan CLI を使用する

作成者: [Scott Addie](https://twitter.com/Scott_Addie)

[LibMan](xref:client-side/libman/index) CLI は、.NET Core がサポートされているすべての場所でサポートされているクロスプラットフォーム ツールです。

## <a name="prerequisites"></a>必須コンポーネント

* [!INCLUDE [2.1-SDK](../../includes/2.1-SDK.md)]

## <a name="installation"></a>インストール

LibMan CLI をインストールするには、次のようにします。

```dotnetcli
dotnet tool install -g Microsoft.Web.LibraryManager.Cli
```

[.NET Core グローバル ツール](/dotnet/core/tools/global-tools#install-a-global-tool)は、[Microsoft.Web.LibraryManager.Cli](https://www.nuget.org/packages/Microsoft.Web.LibraryManager.Cli/) NuGet パッケージからインストールされます。

特定の NuGet パッケージ ソースから LibMan CLI をインストールするには、次のようにします。

```dotnetcli
dotnet tool install -g Microsoft.Web.LibraryManager.Cli --version 1.0.94-g606058a278 --add-source C:\Temp\
```

この例では、ローカル Windows コンピューターの *C:\Temp\Microsoft.Web.LibraryManager.Cli.1.0.94-g606058a278.nupkg* ファイルから .NET Core グローバル ツールがインストールされます。

## <a name="usage"></a>使用方法

CLI のインストールが正常に完了したら、次のコマンドを使用できます。

```console
libman
```

インストールされている CLI のバージョンを表示するには、次のようにします。

```console
libman --version
```

使用可能な CLI コマンドを表示するには、次のようにします。

```console
libman --help
```

上のコマンドを実行すると、次のような出力が表示されます。

```console
 1.0.163+g45474d37ed

Usage: libman [options] [command]

Options:
  --help|-h  Show help information
  --version  Show version information

Commands:
  cache      List or clean libman cache contents
  clean      Deletes all library files defined in libman.json from the project
  init       Create a new libman.json
  install    Add a library definition to the libman.json file, and download the 
             library to the specified location
  restore    Downloads all files from provider and saves them to specified 
             destination
  uninstall  Deletes all files for the specified library from their specified 
             destination, then removes the specified library definition from 
             libman.json
  update     Updates the specified library

Use "libman [command] --help" for more information about a command.
```

次のセクションでは、使用可能な CLI コマンドの概要を示します。

## <a name="initialize-libman-in-the-project"></a>プロジェクト内の LibMan の初期化

`libman init` コマンドを実行すると、*libman.json* ファイルが存在しない場合は作成されます。 このファイルは、既定の項目テンプレート コンテンツを使用して作成されます。

### <a name="synopsis"></a>構文

```console
libman init [-d|--default-destination] [-p|--default-provider] [--verbosity]
libman init [-h|--help]
```

### <a name="options"></a>オプション

`libman init` コマンドには以下のオプションを使用できます。

* `-d|--default-destination <PATH>`

  現在のフォルダーを基準とした相対パス。 *libman.json* のライブラリに `destination` プロパティが定義されていない場合、ライブラリ ファイルはこの場所にインストールされます。 `<PATH>` 値は *libman.json* の `defaultDestination` プロパティに書き込まれます。

* `-p|--default-provider <PROVIDER>`

  指定されたライブラリでプロバイダーが定義されていない場合に使用するプロバイダー。 `<PROVIDER>` 値は *libman.json* の `defaultProvider` プロパティに書き込まれます。 `<PROVIDER>` を次のいずれかの値に置き換えます。

  [!INCLUDE [LibMan provider names](../../includes/libman-cli/provider-names.md)]

[!INCLUDE [standard-cli-options](../../includes/libman-cli/standard-cli-options.md)]

### <a name="examples"></a>使用例

ASP.NET Core プロジェクトで *libman.json* ファイルを作成するには、次のようにします。

* プロジェクトのルートに移動します。
* 次のコマンドを実行します。

  ```console
  libman init
  ```

* 既定のプロバイダーの名前を入力するか、`Enter` を押して既定の CDNJS プロバイダーを使用します。 有効な値を次に示します。

  [!INCLUDE [LibMan provider names](../../includes/libman-cli/provider-names.md)]

  ![libman init コマンド - 既定のプロバイダー](_static/libman-init-provider.png)

次の内容の *libman.json* ファイルがプロジェクト ルートに追加されます。

```json
{
  "version": "1.0",
  "defaultProvider": "cdnjs",
  "libraries": []
}
```

## <a name="add-library-files"></a>ライブラリ ファイルの追加

`libman install` コマンドを実行すると、ライブラリ ファイルがダウンロードされ、プロジェクトにインストールされます。 *libman.json* ファイルが存在しない場合は追加されます。 *libman.json* ファイルは、ライブラリ ファイルの構成の詳細を格納するように変更されます。

### <a name="synopsis"></a>構文

```console
libman install <LIBRARY> [-d|--destination] [--files] [-p|--provider] [--verbosity]
libman install [-h|--help]
```

### <a name="arguments"></a>引数

`LIBRARY`

インストールするライブラリの名前。 この名前には、バージョン番号の表記 (`@1.2.0` など) を含めることができます。

### <a name="options"></a>オプション

`libman install` コマンドには以下のオプションを使用できます。

* `-d|--destination <PATH>`

  ライブラリをインストールする場所。 指定しない場合は、既定の場所が使用されます。 *libman.json* で `defaultDestination` プロパティが指定されていない場合、このオプションは必須です。

* `--files <FILE>`

  ライブラリからインストールするファイルの名前を指定します。 指定しない場合、ライブラリのすべてのファイルがインストールされます。 インストールするファイルごとに 1 つの `--files` オプションを指定します。 相対パスもサポートされています。 たとえば、`--files dist/browser/signalr.js` のように指定します。

* `-p|--provider <PROVIDER>`

  ライブラリの取得に使用するプロバイダーの名前。 `<PROVIDER>` を次のいずれかの値に置き換えます。
  
  [!INCLUDE [LibMan provider names](../../includes/libman-cli/provider-names.md)]

  指定しない場合、*libman.json* の `defaultProvider` プロパティが使用されます。 *libman.json* で `defaultProvider` プロパティが指定されていない場合、このオプションは必須です。

[!INCLUDE [standard-cli-options](../../includes/libman-cli/standard-cli-options.md)]

### <a name="examples"></a>使用例

次の *libman.json* ファイルについて考えてみます。

```json
{
  "version": "1.0",
  "defaultProvider": "cdnjs",
  "libraries": []
}
```

CDNJS プロバイダーを使用して JQuery バージョン 3.2.1 の *jquery.min.js* ファイルを *wwwroot/scripts/jquery* フォルダーにインストールするには、次のようにします。

```console
libman install jquery@3.2.1 --provider cdnjs --destination wwwroot/scripts/jquery --files jquery.min.js
```

*libman.json* ファイルは次のようになります。

```json
{
  "version": "1.0",
  "defaultProvider": "cdnjs",
  "libraries": [
    {
      "library": "jquery@3.2.1",
      "destination": "wwwroot/scripts/jquery",
      "files": [
        "jquery.min.js"
      ]
    }
  ]
}
```

ファイル システム プロバイダーを使用して *C:\\temp\\contosoCalendar\\* から *calendar.js* ファイルと *calendar.css* ファイルをインストールするには、次のようにします。

  ```console
  libman install C:\temp\contosoCalendar\ --provider filesystem --files calendar.js --files calendar.css
  ```

次のプロンプトは、以下の 2 つの理由で表示されます。

* *libman.json* ファイルに `defaultDestination` プロパティが含まれていません。
* `libman install` コマンドに `-d|--destination` オプションが含まれていません。

![libman install コマンド - インストール先](_static/libman-install-destination.png)

既定の保存先を受け入れると、*libman.json* ファイルは次のようになります。

```json
{
  "version": "1.0",
  "defaultProvider": "cdnjs",
  "libraries": [
    {
      "library": "jquery@3.2.1",
      "destination": "wwwroot/scripts/jquery",
      "files": [
        "jquery.min.js"
      ]
    },
    {
      "library": "C:\\temp\\contosoCalendar\\",
      "provider": "filesystem",
      "destination": "wwwroot/lib/contosoCalendar",
      "files": [
        "calendar.js",
        "calendar.css"
      ]
    }
  ]
}
```

## <a name="restore-library-files"></a>ライブラリ ファイルの復元

`libman restore` コマンドは、*libman.json* で定義されているライブラリ ファイルをインストールします。 次の規則が適用されます。

* プロジェクト ルートに *libman.json* ファイルが存在しない場合は、エラーが返されます。
* ライブラリがプロバイダーを指定している場合、*libman.json* の `defaultProvider` プロパティは無視されます。
* ライブラリがインストール先を指定している場合、*libman.json* の `defaultDestination` プロパティは無視されます。

### <a name="synopsis"></a>構文

```console
libman restore [--verbosity]
libman restore [-h|--help]
```

### <a name="options"></a>オプション

`libman restore` コマンドには以下のオプションを使用できます。

[!INCLUDE [standard-cli-options](../../includes/libman-cli/standard-cli-options.md)]

### <a name="examples"></a>使用例

*libman.json* で定義されているライブラリ ファイルを復元するには、次のようにします。

```console
libman restore
```

## <a name="delete-library-files"></a>ライブラリ ファイルの削除

`libman clean` コマンドは、LibMan を使用して以前に復元されたライブラリ ファイルを削除します。 この操作の後、空になったフォルダーがあれば削除されます。 *libman.json* の `libraries` プロパティでライブラリ ファイルに関連付けられている構成は削除されません。

### <a name="synopsis"></a>構文

```console
libman clean [--verbosity]
libman clean [-h|--help]
```

### <a name="options"></a>オプション

`libman clean` コマンドには以下のオプションを使用できます。

[!INCLUDE [standard-cli-options](../../includes/libman-cli/standard-cli-options.md)]

### <a name="examples"></a>使用例

LibMan を使用してインストールされたライブラリ ファイルを削除するには、次のようにします。

```console
libman clean
```

## <a name="uninstall-library-files"></a>ライブラリ ファイルのアンインストール

`libman uninstall` コマンドは、次のことを行います。

* 指定したライブラリに関連付けられているすべてのファイルを *libman.json* のインストール先から削除します。
* 関連付けられたライブラリ構成を *libman.json* から削除します。

次の場合、エラーが発生します。

* プロジェクト ルートに *libman.json* ファイルが存在しません。
* 指定されたライブラリが存在しません。

同じ名前のライブラリが複数インストールされている場合は、1 つを選択するように求められます。

### <a name="synopsis"></a>構文

```console
libman uninstall <LIBRARY> [--verbosity]
libman uninstall [-h|--help]
```

### <a name="arguments"></a>引数

`LIBRARY`

アンインストールするライブラリの名前。 この名前には、バージョン番号の表記 (`@1.2.0` など) を含めることができます。

### <a name="options"></a>オプション

`libman uninstall` コマンドには以下のオプションを使用できます。

[!INCLUDE [standard-cli-options](../../includes/libman-cli/standard-cli-options.md)]

### <a name="examples"></a>使用例

次の *libman.json* ファイルについて考えてみます。

[!code-json[](samples/LibManSample/libman.json)]

* jQuery をアンインストールするには、次のいずれかのコマンドを正常に実行します。

  ```console
  libman uninstall jquery
  ```

  ```console
  libman uninstall jquery@3.3.1
  ```

* `filesystem` プロバイダーを使用してインストールされた Lodash ファイルをアンインストールするには、次のようにします。

  ```console
  libman uninstall C:\temp\lodash\
  ```

## <a name="update-library-version"></a>ライブラリ バージョンの更新

`libman update` コマンドは、LibMan を使用してインストールされたライブラリを指定されたバージョンに更新します。

次の場合、エラーが発生します。

* プロジェクト ルートに *libman.json* ファイルが存在しません。
* 指定されたライブラリが存在しません。

同じ名前のライブラリが複数インストールされている場合は、1 つを選択するように求められます。

### <a name="synopsis"></a>構文

```console
libman update <LIBRARY> [-pre] [--to] [--verbosity]
libman update [-h|--help]
```

### <a name="arguments"></a>引数

`LIBRARY`

更新するライブラリの名前。

### <a name="options"></a>オプション

`libman update` コマンドには以下のオプションを使用できます。

* `-pre`

  ライブラリの最新のプレリリース バージョンを取得します。

* `--to <VERSION>`

  特定のバージョンのライブラリを取得します。

[!INCLUDE [standard-cli-options](../../includes/libman-cli/standard-cli-options.md)]

### <a name="examples"></a>使用例

* jQuery を最新バージョンに更新するには、次のようにします。

  ```console
  libman update jquery
  ```

* jQuery をバージョン 3.3.1 に更新するには、次のようにします。

  ```console
  libman update jquery --to 3.3.1
  ```

* jQuery を最新のプレリリース バージョンに更新するには、次のようにします。

  ```console
  libman update jquery -pre
  ```

## <a name="manage-library-cache"></a>ライブラリ キャッシュの管理

`libman cache` コマンドは、LibMan ライブラリ キャッシュを管理します。 `filesystem` プロバイダーはライブラリ キャッシュを使用しません。

### <a name="synopsis"></a>構文

```console
libman cache clean [<PROVIDER>] [--verbosity]
libman cache list [--files] [--libraries] [--verbosity]
libman cache [-h|--help]
```

### <a name="arguments"></a>引数

`PROVIDER`

`clean` コマンドでのみ使用されます。 消去するプロバイダー キャッシュを指定します。 有効な値を次に示します。

[!INCLUDE [LibMan provider names](../../includes/libman-cli/provider-names.md)]

### <a name="options"></a>オプション

`libman cache` コマンドには以下のオプションを使用できます。

* `--files`

  キャッシュされているファイルの名前を一覧表示します。

* `--libraries`

  キャッシュされているライブラリの名前を一覧表示します。

[!INCLUDE [standard-cli-options](../../includes/libman-cli/standard-cli-options.md)]

### <a name="examples"></a>使用例

* キャッシュされたライブラリの名前をプロバイダーごとに表示するには、次のコマンドのいずれかを使用します。

  ```console
  libman cache list
  ```

  ```console
  libman cache list --libraries
  ```

  次のような出力が表示されます。

  ```console
  Cache contents:
  ---------------
  unpkg:
      knockout
      react
      vue
  cdnjs:
      font-awesome
      jquery
      knockout
      lodash.js
      react
  ```

* キャッシュされたライブラリ ファイルの名前をプロバイダーごとに表示するには、次のようにします。

  ```console
  libman cache list --files
  ```

  次のような出力が表示されます。

  ```console
  Cache contents:
  ---------------
  unpkg:
      knockout:
          <list omitted for brevity>
      react:
          <list omitted for brevity>
      vue:
          <list omitted for brevity>
  cdnjs:
      font-awesome
          metadata.json
      jquery
          metadata.json
          3.2.1\core.js
          3.2.1\jquery.js
          3.2.1\jquery.min.js
          3.2.1\jquery.min.map
          3.2.1\jquery.slim.js
          3.2.1\jquery.slim.min.js
          3.2.1\jquery.slim.min.map
          3.3.1\core.js
          3.3.1\jquery.js
          3.3.1\jquery.min.js
          3.3.1\jquery.min.map
          3.3.1\jquery.slim.js
          3.3.1\jquery.slim.min.js
          3.3.1\jquery.slim.min.map
      knockout
          metadata.json
          3.4.2\knockout-debug.js
          3.4.2\knockout-min.js
      lodash.js
          metadata.json
          4.17.10\lodash.js
          4.17.10\lodash.min.js
      react
          metadata.json
  ```

  前の出力から、jQuery バージョン 3.2.1 と 3.3.1 が CDNJS プロバイダーでキャッシュされていることがわかります。

* CDNJS プロバイダーのライブラリ キャッシュを空にするには、次のようにします。

  ```console
  libman cache clean cdnjs
  ```

  CDNJS プロバイダーのキャッシュが空になると、`libman cache list` コマンドは次のように表示します。

  ```console
  Cache contents:
  ---------------
  unpkg:
      knockout
      react
      vue
  cdnjs:
      (empty)
  ```

* サポートされているすべてのプロバイダーのキャッシュを空にするには、次のようにします。

  ```console
  libman cache clean
  ```

  すべてのプロバイダーのキャッシュが空になると、`libman cache list` コマンドは次のように表示します。

  ```console
  Cache contents:
  ---------------
  unpkg:
      (empty)
  cdnjs:
      (empty)
  ```

## <a name="additional-resources"></a>その他の技術情報

* [グローバル ツールのインストール](/dotnet/core/tools/global-tools#install-a-global-tool)
* <xref:client-side/libman/libman-vs>
* [LibMan の GitHub リポジトリ](https://github.com/aspnet/LibraryManager)
