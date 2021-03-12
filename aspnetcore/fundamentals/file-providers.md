---
title: ASP.NET Core でのファイル プロバイダー
author: rick-anderson
description: ASP.NET Core がファイル プロバイダーを使用してファイル システムへのアクセスを抽象化する方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 04/06/2020
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
uid: fundamentals/file-providers
ms.openlocfilehash: c66c35e93991333229e367e9f371b125d8067131
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588218"
---
# <a name="file-providers-in-aspnet-core"></a>ASP.NET Core でのファイル プロバイダー

作成者: [Steve Smith](https://ardalis.com/)

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core は、ファイル プロバイダーを使用してファイル システムへのアクセスを抽象化します。 ファイル プロバイダーは、ASP.NET Core フレームワーク全体で使用されます。 次に例を示します。

* <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> では、アプリの[コンテンツ ルート](xref:fundamentals/index#content-root)と [Web ルート](xref:fundamentals/index#web-root)が `IFileProvider` 型として公開されます。
* [静的ファイル ミドルウェア](xref:fundamentals/static-files)では、ファイル プロバイダーを使用して静的なファイルを見つけます。
* [Razor](xref:mvc/views/razor) では、ファイル プロバイダーを使用してページとビューを見つけます。
* .NET Core Tooling では、ファイル プロバイダーと glob パターンを使用して、公開するファイルを指定します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/file-providers/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="file-provider-interfaces"></a>ファイル プロバイダーのインターフェイス

プライマリ インターフェイスは <xref:Microsoft.Extensions.FileProviders.IFileProvider> です。 `IFileProvider` では次のためのメソッドが公開されます。

* ファイルの情報を取得します (<xref:Microsoft.Extensions.FileProviders.IFileInfo>)。
* ディレクトリの情報を取得します (<xref:Microsoft.Extensions.FileProviders.IDirectoryContents>)。
* 変更通知を設定します (<xref:Microsoft.Extensions.Primitives.IChangeToken> を使用)。

`IFileInfo` ではファイルを操作するためのメソッドとプロパティが提供されます。

* <xref:Microsoft.Extensions.FileProviders.IFileInfo.Exists>
* <xref:Microsoft.Extensions.FileProviders.IFileInfo.IsDirectory>
* <xref:Microsoft.Extensions.FileProviders.IFileInfo.Name>
* <xref:Microsoft.Extensions.FileProviders.IFileInfo.Length> (バイト単位)
* <xref:Microsoft.Extensions.FileProviders.IFileInfo.LastModified> の日付

<xref:Microsoft.Extensions.FileProviders.IFileInfo.CreateReadStream*?displayProperty=nameWithType> メソッドを使用して、ファイルから情報を読み取ることができます。

*FileProviderSample* サンプル アプリでは、[依存関係の挿入](xref:fundamentals/dependency-injection)を介してアプリ全体で使用するために、`Startup.ConfigureServices` でファイル プロバイダーを構成する方法を示します。

## <a name="file-provider-implementations"></a>ファイル プロバイダーの実装

次の表に、`IFileProvider` の実装の一覧を示します。

| 実装 | 説明 |
| -------------- | ----------- |
| [CompositeFileProvider](#compositefileprovider) | その他の 1 つまたは複数のプロバイダーからのファイルおよびディレクトリへのアクセスを結合するために使用します。 |
| [ManifestEmbeddedFileProvider](#manifestembeddedfileprovider) | アセンブリに埋め込まれているファイルにアクセスする場合に使用します。 |
| [PhysicalFileProvider](#physicalfileprovider) | システムの物理ファイルにアクセスするために使用します。 |

### <a name="physicalfileprovider"></a>PhysicalFileProvider

<xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider> は、物理ファイル システムへのアクセス許可を提供します。 `PhysicalFileProvider` では、<xref:System.IO.File?displayProperty=fullName> 型が使用され (物理プロバイダーの場合)、すべてのパスのスコープが 1 つのディレクトリとその子ディレクトリに設定されます。 このスコープ設定により、指定されたディレクトリとその子ディレクトリを除くファイル システムにアクセスできなくなります。 `PhysicalFileProvider` を作成して使用する最も一般的なシナリオは、[依存関係の挿入](xref:fundamentals/dependency-injection)を通してコンストラクターで `IFileProvider` を要求する場合です。

このプロバイダーを直接インスタンス化するときは、絶対ディレクトリ パスが要求され、そのプロバイダーを使用して行われるすべての要求のベース パスとして機能します。 glob パターンはディレクトリ パスではサポートされていません。

次のコードは、`PhysicalFileProvider` を使用してディレクトリの内容とファイルの情報を取得する方法を示しています。

```csharp
var provider = new PhysicalFileProvider(applicationRoot);
var contents = provider.GetDirectoryContents(string.Empty);
var filePath = Path.Combine("wwwroot", "js", "site.js");
var fileInfo = provider.GetFileInfo(filePath);
```

前の例の型は次のとおりです。

* `provider` は `IFileProvider` です。
* `contents` は `IDirectoryContents` です。
* `fileInfo` は `IFileInfo` です。

ファイル プロバイダーを使用して、`applicationRoot` で指定したディレクトリ全体を反復処理したり、`GetFileInfo` を呼び出してファイル情報を取得したりできます。 glob パターンを `GetFileInfo` メソッドに渡すことはできません。 ファイル プロバイダーは、`applicationRoot` ディレクトリの外部にはアクセスできません。

*FileProviderSample* サンプル アプリでは、<xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ContentRootFileProvider?displayProperty=nameWithType> を使用して `Startup.ConfigureServices` メソッドにプロバイダーが作成されます。

```csharp
var physicalProvider = _env.ContentRootFileProvider;
```

### <a name="manifestembeddedfileprovider"></a>ManifestEmbeddedFileProvider

<xref:Microsoft.Extensions.FileProviders.ManifestEmbeddedFileProvider> は、アセンブリに埋め込まれたファイルにアクセスするために使用されます。 `ManifestEmbeddedFileProvider` では、アセンブリにコンパイルされたマニフェストを使用して、埋め込まれたファイルの元のパスを再構築します。

埋め込みファイルのマニフェストを生成するには、次のようにします。

1. [Microsoft.Extensions.FileProviders.Embedded](https://www.nuget.org/packages/Microsoft.Extensions.FileProviders.Embedded) NuGet パッケージをプロジェクトに追加します。
1. `<GenerateEmbeddedFilesManifest>` プロパティを `true`に設定します。 [\<EmbeddedResource>](/dotnet/core/tools/csproj#default-compilation-includes-in-net-core-projects) を使用して列挙するファイルを指定します:

    [!code-xml[](file-providers/samples/3.x/FileProviderSample/FileProviderSample.csproj?highlight=5,13)]

[glob パターン](#glob-patterns)を使用して、アセンブリに埋め込むファイルを 1 つまたは複数指定します。

*FileProviderSample* サンプル アプリでは `ManifestEmbeddedFileProvider` が作成され、現在実行しているアセンブリがそのコンストラクターに渡されます。

*Startup.cs*:

```csharp
var manifestEmbeddedProvider = 
    new ManifestEmbeddedFileProvider(typeof(Program).Assembly);
```

追加のオーバーロードを使用すると、次のことが可能になります。

* 相対ファイル パスを指定します。
* ファイルのスコープを最終変更日に設定します。
* 埋め込みファイルのマニフェストを含む埋め込みリソースに名前を付けます。

| オーバーロード | 説明 |
| -------- | ----------- |
| `ManifestEmbeddedFileProvider(Assembly, String)` | 必要に応じて相対パスのパラメーター `root` を指定できます。 `root` を指定して、<xref:Microsoft.Extensions.FileProviders.IFileProvider.GetDirectoryContents*> の呼び出しのスコープを指定したパス以下のリソースに設定します。 |
| `ManifestEmbeddedFileProvider(Assembly, String, DateTimeOffset)` | 必要に応じて、相対パス パラメーター `root` および日付パラメーター `lastModified` (<xref:System.DateTimeOffset>) を指定できます。 `lastModified` の日付では、<xref:Microsoft.Extensions.FileProviders.IFileProvider> によって返される <xref:Microsoft.Extensions.FileProviders.IFileInfo> インスタンスの最終更新日のスコープを設定します。 |
| `ManifestEmbeddedFileProvider(Assembly, String, String, DateTimeOffset)` | 必要に応じて、相対パス `root`、日付 `lastModified`、`manifestName` パラメーターを指定できます。 `manifestName` は、マニフェストを含む埋め込みリソースの名前を表します。 |

### <a name="compositefileprovider"></a>CompositeFileProvider

<xref:Microsoft.Extensions.FileProviders.CompositeFileProvider> は、`IFileProvider` インスタンスを結合し、複数のプロバイダーからのファイルを操作するための 1 つのインターフェイスを公開します。 `CompositeFileProvider` を作成する場合、1 つまたは複数の `IFileProvider` インスタンスをそのコンストラクターに渡します。

*FileProviderSample* サンプル アプリでは、`PhysicalFileProvider` と `ManifestEmbeddedFileProvider` により、アプリのサービス コンテナーに登録されている `CompositeFileProvider` にファイルが提供されます。 次のコードは、プロジェクトの `Startup.ConfigureServices` メソッドにあります。

[!code-csharp[](file-providers/samples/3.x/FileProviderSample/Startup.cs?name=snippet1)]

## <a name="watch-for-changes"></a>変更の監視

<xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*?displayProperty=nameWithType> メソッドでは、変更がないかどうか 1 つ以上のファイルまたはディレクトリを監視するシナリオを提供します。 `Watch` メソッド:

* ファイル パス文字列を指定できます。これにより、[glob パターン](#glob-patterns)を使用して複数のファイルを指定できます。
* <xref:Microsoft.Extensions.Primitives.IChangeToken> を返します。

生成される変更トークンでは次のものが公開されます。

* <xref:Microsoft.Extensions.Primitives.IChangeToken.HasChanged>:このプロパティを調べることで、変更があったかどうかを判断できます。
* <xref:Microsoft.Extensions.Primitives.IChangeToken.RegisterChangeCallback*>:指定したパス文字列に対して変更が検出されたときに呼び出されます。 各変更トークンは、1 つの変更への応答として、関連付けられたコールバックを呼び出すのみです。 定数の監視を有効にするには、<xref:System.Threading.Tasks.TaskCompletionSource`1> を使用するか (以下を参照)、変更への応答として `IChangeToken` インスタンスを再作成します。

*WatchConsole* サンプル アプリでは、*TextFiles* ディレクトリの *.txt* ファイルが変更されるたびに、メッセージが書き込まれます。

[!code-csharp[](file-providers/samples/3.x/WatchConsole/Program.cs?name=snippet1)]

Docker コンテナーやネットワーク共有など、一部のファイル システムは、変更通知を確実に送信しない可能性があります。 `DOTNET_USE_POLLING_FILE_WATCHER` 環境変数を `1` または `true` に設定して、変更がないかどうか、4 秒ごとにファイル システムをポーリングして確認します (構成不可)。

### <a name="glob-patterns"></a>glob パターン

ファイル システム パスは、"*glob (または globbing) パターン*" と呼ばれるワイルドカード パターンを使用します。 これらのパターンを使用して、ファイルのグループを指定します。 2 つのワイルドカード文字は、`*` と `**` です。

**`*`**  
現在のフォルダー レベルにある任意の要素、任意のファイル名、または任意のファイル拡張子を照合します。 照合はファイル パス内の `/` 文字および `.` 文字によって終了します。

**`**`**  
複数のディレクトリ レベルにわたって任意の要素を照合します。 ディレクトリ階層内の多数のファイルを再帰的に照合する場合に使用できます。

次の表は、glob パターンの一般的な例を示しています。

|パターン  |説明  |
|---------|---------|
|`directory/file.txt`|特定のディレクトリ内の特定のファイルを照合します。|
|`directory/*.txt`|特定のディレクトリ内の *.txt* 拡張子を持つすべてのファイルを照合します。|
|`directory/*/appsettings.json`|*directory* フォルダーのちょうど 1 つ下のレベルにあるディレクトリ内のすべての *appsettings.json* ファイルを照合します。|
|`directory/**/*.txt`|*directory* フォルダーの下の任意の場所にある、 *.txt* 拡張子を持つすべてのファイルを照合します。|

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core は、ファイル プロバイダーを使用してファイル システムへのアクセスを抽象化します。 ファイル プロバイダーは、ASP.NET Core フレームワークの全体で使用されます。

* <xref:Microsoft.Extensions.Hosting.IHostingEnvironment> では、アプリの[コンテンツ ルート](xref:fundamentals/index#content-root)と [Web ルート](xref:fundamentals/index#web-root)が `IFileProvider` 型として公開されます。
* [静的ファイル ミドルウェア](xref:fundamentals/static-files)では、ファイル プロバイダーを使用して静的なファイルを見つけます。
* [Razor](xref:mvc/views/razor) では、ファイル プロバイダーを使用してページとビューを見つけます。
* .NET Core Tooling では、ファイル プロバイダーと glob パターンを使用して、公開するファイルを指定します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/file-providers/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="file-provider-interfaces"></a>ファイル プロバイダーのインターフェイス

プライマリ インターフェイスは <xref:Microsoft.Extensions.FileProviders.IFileProvider> です。 `IFileProvider` では次のためのメソッドが公開されます。

* ファイルの情報を取得します (<xref:Microsoft.Extensions.FileProviders.IFileInfo>)。
* ディレクトリの情報を取得します (<xref:Microsoft.Extensions.FileProviders.IDirectoryContents>)。
* 変更通知を設定します (<xref:Microsoft.Extensions.Primitives.IChangeToken> を使用)。

`IFileInfo` ではファイルを操作するためのメソッドとプロパティが提供されます。

* <xref:Microsoft.Extensions.FileProviders.IFileInfo.Exists>
* <xref:Microsoft.Extensions.FileProviders.IFileInfo.IsDirectory>
* <xref:Microsoft.Extensions.FileProviders.IFileInfo.Name>
* <xref:Microsoft.Extensions.FileProviders.IFileInfo.Length> (バイト単位)
* <xref:Microsoft.Extensions.FileProviders.IFileInfo.LastModified> の日付

[IFileInfo.CreateReadStream](xref:Microsoft.Extensions.FileProviders.IFileInfo.CreateReadStream*) メソッドを使用してファイルから読み取ることができます。

サンプル アプリでは、[依存関係の挿入](xref:fundamentals/dependency-injection)を介してアプリ全体で使用するために、`Startup.ConfigureServices` でファイル プロバイダーを構成する方法を示します。

## <a name="file-provider-implementations"></a>ファイル プロバイダーの実装

利用できる `IFileProvider` の実装は 3 つあります。

| 実装 | 説明 |
| -------------- | ----------- |
| [PhysicalFileProvider](#physicalfileprovider) | システムの物理ファイルにアクセスするために、物理プロバイダーが使用されます。 |
| [ManifestEmbeddedFileProvider](#manifestembeddedfileprovider) | アセンブリに埋め込まれているファイルにアクセスするために、マニフェストが埋め込まれたプロバイダーが使用されます。 |
| [CompositeFileProvider](#compositefileprovider) | コンポジット プロパイダーは、その他の 1 つまたは複数のプロバイダーからのファイルおよびディレクトリに対するアクセスを結合する場合に使用されます。 |

### <a name="physicalfileprovider"></a>PhysicalFileProvider

<xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider> は、物理ファイル システムへのアクセス許可を提供します。 `PhysicalFileProvider` では、<xref:System.IO.File?displayProperty=fullName> 型が使用され (物理プロバイダーの場合)、すべてのパスのスコープが 1 つのディレクトリとその子ディレクトリに設定されます。 このスコープ設定により、指定されたディレクトリとその子ディレクトリを除くファイル システムにアクセスできなくなります。 `PhysicalFileProvider` を作成して使用する最も一般的なシナリオは、[依存関係の挿入](xref:fundamentals/dependency-injection)を通してコンストラクターで `IFileProvider` を要求する場合です。

このプロバイダーを直接インスタンス化するときは、ディレクトリ パスが要求され、そのプロバイダーを使用して行われるすべての要求のベース パスとして機能します。

次のコードでは、`PhysicalFileProvider` の作成方法と、これを使ってディレクトリのコンテンツとファイル情報を取得する方法が示されます。

```csharp
var provider = new PhysicalFileProvider(applicationRoot);
var contents = provider.GetDirectoryContents(string.Empty);
var fileInfo = provider.GetFileInfo("wwwroot/js/site.js");
```

前の例の型は次のとおりです。

* `provider` は `IFileProvider` です。
* `contents` は `IDirectoryContents` です。
* `fileInfo` は `IFileInfo` です。

ファイル プロバイダーを使用して、`applicationRoot` で指定したディレクトリ全体を反復処理したり、`GetFileInfo` を呼び出してファイル情報を取得したりできます。 ファイル プロバイダーは、`applicationRoot` ディレクトリの外部にはアクセスできません。

サンプル アプリの `Startup.ConfigureServices` クラスでは、[IHostingEnvironment.ContentRootFileProvider](xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ContentRootFileProvider) を使用してプロバイダーを作成しています。

```csharp
var physicalProvider = _env.ContentRootFileProvider;
```

### <a name="manifestembeddedfileprovider"></a>ManifestEmbeddedFileProvider

<xref:Microsoft.Extensions.FileProviders.ManifestEmbeddedFileProvider> は、アセンブリに埋め込まれたファイルにアクセスするために使用されます。 `ManifestEmbeddedFileProvider` では、アセンブリにコンパイルされたマニフェストを使用して、埋め込まれたファイルの元のパスを再構築します。

埋め込みファイルのマニフェストを生成するには、`<GenerateEmbeddedFilesManifest>` プロパティを `true` に設定します。 [&lt;EmbeddedResource&gt;](/dotnet/core/tools/csproj#default-compilation-includes-in-net-core-projects) を使用して埋め込むファイルを指定します。

[!code-xml[](file-providers/samples/2.x/FileProviderSample/FileProviderSample.csproj?highlight=6,14)]

[glob パターン](#glob-patterns)を使用して、アセンブリに埋め込むファイルを 1 つまたは複数指定します。

サンプル アプリでは `ManifestEmbeddedFileProvider` を作成して、現在実行しているアセンブリをそのコンストラクターに渡します。

*Startup.cs*:

```csharp
var manifestEmbeddedProvider = 
    new ManifestEmbeddedFileProvider(typeof(Program).Assembly);
```

追加のオーバーロードを使用すると、次のことが可能になります。

* 相対ファイル パスを指定します。
* ファイルのスコープを最終変更日に設定します。
* 埋め込みファイルのマニフェストを含む埋め込みリソースに名前を付けます。

| オーバーロード | 説明 |
| -------- | ----------- |
| `ManifestEmbeddedFileProvider(Assembly, String)` | 必要に応じて相対パスのパラメーター `root` を指定できます。 `root` を指定して、<xref:Microsoft.Extensions.FileProviders.IFileProvider.GetDirectoryContents*> の呼び出しのスコープを指定したパス以下のリソースに設定します。 |
| `ManifestEmbeddedFileProvider(Assembly, String, DateTimeOffset)` | 必要に応じて、相対パス パラメーター `root` および日付パラメーター `lastModified` (<xref:System.DateTimeOffset>) を指定できます。 `lastModified` の日付では、<xref:Microsoft.Extensions.FileProviders.IFileProvider> によって返される <xref:Microsoft.Extensions.FileProviders.IFileInfo> インスタンスの最終更新日のスコープを設定します。 |
| `ManifestEmbeddedFileProvider(Assembly, String, String, DateTimeOffset)` | 必要に応じて、相対パス `root`、日付 `lastModified`、`manifestName` パラメーターを指定できます。 `manifestName` は、マニフェストを含む埋め込みリソースの名前を表します。 |

### <a name="compositefileprovider"></a>CompositeFileProvider

<xref:Microsoft.Extensions.FileProviders.CompositeFileProvider> は、`IFileProvider` インスタンスを結合し、複数のプロバイダーからのファイルを操作するための 1 つのインターフェイスを公開します。 `CompositeFileProvider` を作成する場合、1 つまたは複数の `IFileProvider` インスタンスをそのコンストラクターに渡します。

サンプル アプリでは、`PhysicalFileProvider` と `ManifestEmbeddedFileProvider` が、アプリのサービス コンテナーに登録されている `CompositeFileProvider` にファイルを提供します。

[!code-csharp[](file-providers/samples/2.x/FileProviderSample/Startup.cs?name=snippet1)]

## <a name="watch-for-changes"></a>変更の監視

[IFileProvider.Watch](xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*) メソッドによって、1 つまたは複数のファイルやディレクトリに変更がないかどうか監視するシナリオが提供されます。 `Watch` にはパス文字列を指定できます。ここでは、[glob パターン](#glob-patterns)を使用して複数のファイルを指定できます。 `Watch` では <xref:Microsoft.Extensions.Primitives.IChangeToken> が返されます。 変更トークンでは次のものが公開されます。

* <xref:Microsoft.Extensions.Primitives.IChangeToken.HasChanged>:このプロパティを調べることで、変更があったかどうかを判断できます。
* <xref:Microsoft.Extensions.Primitives.IChangeToken.RegisterChangeCallback*>:指定したパス文字列に対して変更が検出されたときに呼び出されます。 各変更トークンは、1 つの変更への応答として、関連付けられたコールバックを呼び出すのみです。 定数の監視を有効にするには、<xref:System.Threading.Tasks.TaskCompletionSource`1> を使用するか (以下を参照)、変更への応答として `IChangeToken` インスタンスを再作成します。

サンプル アプリでは、*WatchConsole* コンソール アプリは、テキスト ファイルが変更されるたびにメッセージを表示するように構成されています。

[!code-csharp[](file-providers/samples/2.x/WatchConsole/Program.cs?name=snippet1&highlight=1-2,16,19-20)]

Docker コンテナーやネットワーク共有など、一部のファイル システムは、変更通知を確実に送信しない可能性があります。 `DOTNET_USE_POLLING_FILE_WATCHER` 環境変数を `1` または `true` に設定して、変更がないかどうか、4 秒ごとにファイル システムをポーリングして確認します (構成不可)。

## <a name="glob-patterns"></a>glob パターン

ファイル システム パスは、"*glob (または globbing) パターン*" と呼ばれるワイルドカード パターンを使用します。 これらのパターンを使用して、ファイルのグループを指定します。 2 つのワイルドカード文字は、`*` と `**` です。

**`*`**  
現在のフォルダー レベルにある任意の要素、任意のファイル名、または任意のファイル拡張子を照合します。 照合はファイル パス内の `/` 文字および `.` 文字によって終了します。

**`**`**  
複数のディレクトリ レベルにわたって任意の要素を照合します。 ディレクトリ階層内の多数のファイルを再帰的に照合する場合に使用できます。

**glob パターンの例**

**`directory/file.txt`**  
特定のディレクトリ内の特定のファイルを照合します。

**`directory/*.txt`**  
特定のディレクトリ内の *.txt* 拡張子を持つすべてのファイルを照合します。

**`directory/*/appsettings.json`**  
*directory* フォルダーのちょうど 1 つ下のレベルにあるディレクトリ内のすべての `appsettings.json` ファイルを照合します。

**`directory/**/*.txt`**  
*directory* フォルダーの下の任意の場所にある、 *.txt* 拡張子を持つすべてのファイルを照合します。

::: moniker-end
