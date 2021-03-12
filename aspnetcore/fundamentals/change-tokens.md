---
title: ASP.NET Core で変更トークンを使用して変更を検出する
author: rick-anderson
description: 変更トークンを使用して変更を追跡する方法を説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 10/07/2019
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
uid: fundamentals/change-tokens
ms.openlocfilehash: df2be1b89ad9681ff70dd71cb3026786f59c8b2a
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589388"
---
# <a name="detect-changes-with-change-tokens-in-aspnet-core"></a>ASP.NET Core で変更トークンを使用して変更を検出する

::: moniker range=">= aspnetcore-3.0"

"*変更トークン*" は、状態の変更を追跡するために使用される汎用の低レベル構成ブロックです。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/change-tokens/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="ichangetoken-interface"></a>IChangeToken インターフェイス

<xref:Microsoft.Extensions.Primitives.IChangeToken> により、変更が発生したという通知が伝達されます。 `IChangeToken` は <xref:Microsoft.Extensions.Primitives?displayProperty=fullName> 名前空間内にあります。 [Microsoft.Extensions.Primitives](https://www.nuget.org/packages/Microsoft.Extensions.Primitives/) NuGet パッケージは、ASP.NET Core アプリに暗黙的に提供されます。

`IChangeToken` に次の 2 つのプロパティがあります。

* <xref:Microsoft.Extensions.Primitives.IChangeToken.ActiveChangeCallbacks> は、トークンによってコールバックがプロアクティブに生成されるかどうかを示します。 `ActiveChangedCallbacks` が `false` に設定されている場合、コールバックは呼び出されず、アプリは変更のために `HasChanged` をポーリングする必要があります。 変更がまったく発生しない場合、または基になる変更リスナーが破棄されるか無効になっている場合、トークンがまったくキャンセルされない可能性もあります。
* <xref:Microsoft.Extensions.Primitives.IChangeToken.HasChanged> は、変更が発生したかどうかを示す値を受け取ります。

`IChangeToken` インターフェイスには、[RegisterChangeCallback(Action\<Object>, Object)](xref:Microsoft.Extensions.Primitives.IChangeToken.RegisterChangeCallback*) というメソッドが含まれています。これを使うと、トークンが変更されたときに呼び出されるコールバックを登録できます。 コールバックが呼び出される前に `HasChanged` を設定する必要があります。

## <a name="changetoken-class"></a>ChangeToken クラス

<xref:Microsoft.Extensions.Primitives.ChangeToken> は、変更が発生したことの通知を伝達するために使用される静的クラスです。 `ChangeToken` は <xref:Microsoft.Extensions.Primitives?displayProperty=fullName> 名前空間内にあります。 [Microsoft.Extensions.Primitives](https://www.nuget.org/packages/Microsoft.Extensions.Primitives/) NuGet パッケージは、ASP.NET Core アプリに暗黙的に提供されます。

[ChangeToken.OnChange(Func\<IChangeToken>, Action)](xref:Microsoft.Extensions.Primitives.ChangeToken.OnChange*) メソッドは、トークンが変更されるたびに呼び出される `Action` を登録します。

* `Func<IChangeToken>` はトークンを生成します。
* `Action` は、トークンが変更されたときに呼び出されます。

[ChangeToken.OnChange\<TState>(Func\<IChangeToken>, Action\<TState>, TState)](xref:Microsoft.Extensions.Primitives.ChangeToken.OnChange*) オーバーロードは、トークン コンシューマー `Action` に渡される追加の `TState` パラメーターを取ります。

`OnChange` では <xref:System.IDisposable> が返されます。 <xref:System.IDisposable.Dispose*> を呼び出すと、トークンによるその後の変更のリッスンが停止され、トークンのリソースが解放されます。

## <a name="example-uses-of-change-tokens-in-aspnet-core"></a>ASP.NET Core での変更のトークンの使用例

変更トークンは、オブジェクトの変更を監視するために ASP.NET Core の主要な領域で使用されます。

* ファイルへの変更を監視するために、<xref:Microsoft.Extensions.FileProviders.IFileProvider> の <xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*> メソッドでは、指定された監視対象のファイルまたはフォルダーの `IChangeToken` が作成されます。
* `IChangeToken` トークンをキャッシュ エントリに追加して、変更時にキャッシュの削除をトリガーすることができます。
* `TOptions` の変更のために、<xref:Microsoft.Extensions.Options.OptionsMonitor`1> の既定の実装である <xref:Microsoft.Extensions.Options.IOptionsMonitor`1> には、1 つ以上の <xref:Microsoft.Extensions.Options.IOptionsChangeTokenSource`1> インスタンスを受け取るオーバーロードが含まれています。 各インスタンスは、オプションの変更を追跡するために変更通知のコールバックを登録する `IChangeToken` を返します。

## <a name="monitor-for-configuration-changes"></a>構成の変更の監視

既定では、ASP.NET Core テンプレートは、[JSON 構成ファイル](xref:fundamentals/configuration/index#json-configuration-provider) ( *appsettings.json* 、*appsettings.Development.json*、および *appsettings.Production.json*) を使用して、アプリの構成設定を読み込みます。

これらのファイルは、`reloadOnChange` パラメーターを受け取る、<xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> 上の [AddJsonFile(IConfigurationBuilder, String, Boolean, Boolean)](xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*) 拡張メソッドを使用して構成されます。 `reloadOnChange` は、ファイルの変更時に構成を再読み込みするかどうかを示します。 この設定は、<xref:Microsoft.Extensions.Hosting.Host> の便利なメソッド <xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder*> 内に現れます。

```csharp
config.AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
      .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true, 
          reloadOnChange: true);
```

ファイル ベースの構成は、<xref:Microsoft.Extensions.Configuration.FileConfigurationSource> によって表されます。 `FileConfigurationSource` ではファイルを監視するために <xref:Microsoft.Extensions.FileProviders.IFileProvider> が使用されます。

既定では、`IFileMonitor` は <xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider> によって提供されます。これは、<xref:System.IO.FileSystemWatcher> を使用して、構成ファイルの変更を監視します。

サンプル アプリは、構成の変更を監視するための 2 つの実装を示します。 *appsettings* ファイルのいずれかが変更されると、ファイルを監視する両方の実装によってカスタム コードが実行されます。サンプル アプリによりコンソールにメッセージが書き込まれます。

構成ファイルの `FileSystemWatcher` は、1 つの構成ファイルの変更に対して複数のトークンのコールバックをトリガーできます。 複数のトークンのコールバックがトリガーされたときにカスタム コードが一度だけ実行されるようにするために、サンプルの実装ではファイル ハッシュがチェックされます。 サンプルでは、SHA1 ファイル ハッシュが使用されています。 再試行は、指数バックオフを使用して実装されます。 再実行が提供されるのは、ファイルの新しいハッシュの計算を一時的に妨げるファイル ロックが発生する場合があるためです。

*Utilities/Utilities.cs*:

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Utilities/Utilities.cs?name=snippet1)]

### <a name="simple-startup-change-token"></a>シンプルなスタートアップ変更トークン

構成再読み込みトークンへの変更通知のために、トークン コンシューマー `Action` のコールバックを登録します。

`Startup.Configure`の場合:

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Startup.cs?name=snippet2)]

`config.GetReloadToken()` はトークンを提供します。 コールバックは、`InvokeChanged` メソッドです。

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Startup.cs?name=snippet3)]

コールバックの `state` は、`IWebHostEnvironment` を渡すためのに使われます。これは監視する適切な *appsettings* 構成ファイルを指定するのに便利です (たとえば、開発環境の場合は *appsettings.Development.json*)。 ファイル ハッシュは、構成ファイルが 1 回のみ変更されたときの複数のトークンのコールバックのために `WriteConsole` ステートメントが複数回実行されるのを防ぐために使用されます。

このシステムは、アプリが実行されている限り実行され、ユーザーが無効にすることはできません。

### <a name="monitor-configuration-changes-as-a-service"></a>サービスとしての構成の変更の監視

サンプルは次のものを実装します。

* 基本的なスタートアップ トークンの監視
* サービスとしての監視
* 監視を有効および無効にするメカニズム。

サンプルでは `IConfigurationMonitor` インターフェイスが設定されています。

*Extensions/ConfigurationMonitor.cs*:

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet1)]

実装済みのクラスのコンストラクター `ConfigurationMonitor` は、変更通知のコールバックを登録します。

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet2)]

`config.GetReloadToken()` はトークンを提供します。 `InvokeChanged` はコールバック メソッドです。 このインスタンスの `state` は、監視の状態にアクセスするために使用される `IConfigurationMonitor` インスタンスへの参照です。 次の 2 つのプロパティが使用されます。

* `MonitoringEnabled`:コールバックでそのカスタム コードを実行する必要があるかどうかを示します。
* `CurrentState`:UI で使用するための現在の監視状態を説明します。

`InvokeChanged` メソッドは、次の点を除けば、前の方法に似ています。

* `MonitoringEnabled` が `true` ではない限りコードを実行しません。
* 現在の `state` をその `WriteConsole` 出力に出力します。

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet3)]

インスタンス `ConfigurationMonitor` は、`Startup.ConfigureServices` でサービスとして登録されます。

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Startup.cs?name=snippet1)]

インデックス ページは、ユーザーに構成監視の制御を提供します。 `IConfigurationMonitor` のインスタンスは、`IndexModel` に挿入されます。

*Pages/Index.cshtml.cs*:

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Pages/Index.cshtml.cs?name=snippet1)]

構成監視 (`_monitor`) は、監視を有効または無効にし、UI フィードバックの現在の状態を設定するために使用されます。

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Pages/Index.cshtml.cs?name=snippet2)]

`OnPostStartMonitoring` がトリガーされると、監視が有効になり、現在の状態がクリアされます。 `OnPostStopMonitoring` がトリガーされると、監視が無効になり、監視が発生していないことを反映するように状態が設定されます。

UI のボタンを使って監視を有効および無効にできます。

*Pages/Index.cshtml*:

[!code-cshtml[](change-tokens/samples/3.x/SampleApp/Pages/Index.cshtml?name=snippet_Buttons)]

## <a name="monitor-cached-file-changes"></a>キャッシュされたファイルの変更の監視

<xref:Microsoft.Extensions.Caching.Memory.IMemoryCache> を使用して、ファイルの内容をメモリ内にキャッシュすることができます。 メモリ内キャッシュの説明については、「[メモリ内キャッシュ](xref:performance/caching/memory)」トピックを参照してください。 以下に示す実装など、追加の手順を実行しないと、ソース データが変更された場合に *期限切れの* (古い) データがキャッシュから返されます。

たとえば、[スライド式有効期限](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.SlidingExpiration)期間の更新時にキャッシュされたソース ファイルの状態を考慮しないと、キャッシュされたファイル データが古くなります。 データの各要求が、スライド式有効期限を更新しますが、ファイルがキャッシュに再読み込みされることはありません。 ファイルのキャッシュされた内容を使用するアプリの機能は、古い内容を受け取る可能性があります。

ファイル キャッシュのシナリオで変更トークンを使用すると、キャッシュに古いファイルの内容が残ることを防止できます。 サンプル アプリは、このアプローチの実装を示しています。

このサンプルでは、`GetFileContent` を使用して次の操作を実行します。

* ファイルの内容を返します。
* 指数バックオフを使用する再試行アルゴリズムを実装し、ファイル ロックによりファイルの読み取りが一時的に妨げられるケースに対処します。

*Utilities/Utilities.cs*:

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Utilities/Utilities.cs?name=snippet2)]

`FileService` はキャッシュされたファイルの参照の処理するために作成されます。 サービスの `GetFileContent` メソッドの呼び出しは、メモリ内キャッシュからファイルの内容を取得して、呼び出し元 (*Services/FileService.cs*) に戻そうとします。

キャッシュ キーを使用してキャッシュされた内容が見つからない場合、次の操作が実行されます。

1. `GetFileContent` を使用してファイルの内容が取得されます。
1. 変更トークンは、[IFileProviders.Watch](xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*) を使用してファイル プロバイダーから取得します。 ファイルが変更されたときに、トークンのコールバックがトリガーされます。
1. ファイルの内容は、[スライド式有効期限](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.SlidingExpiration)の期間キャシュされます。 ファイルがキャッシュされている間に変更された場合、キャッシュ エントリを削除するために、[MemoryCacheEntryExtensions.AddExpirationToken](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryExtensions.AddExpirationToken*) を使用して変更トークンがアタッチされます。

次の例では、ファイルはアプリの[コンテンツ ルート](xref:fundamentals/index#content-root)に格納されます。 `IWebHostEnvironment.ContentRootFileProvider` は、アプリの `IWebHostEnvironment.ContentRootPath` を指す <xref:Microsoft.Extensions.FileProviders.IFileProvider> を取得するために使用されます。 [IFileInfo.PhysicalPath](xref:Microsoft.Extensions.FileProviders.IFileInfo.PhysicalPath) を使って `filePath` を取得します。

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Services/FileService.cs?name=snippet1)]

`FileService` はメモリ キャッシュ サービスと共にサービス コンテナーに登録されます。

`Startup.ConfigureServices`の場合:

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Startup.cs?name=snippet4)]

ページ モデルでは、このサービスを使ってファイルの内容が読み込まれます。

Index ページの `OnGet` メソッド内 (*Pages/Index.cshtml.cs*):

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Pages/Index.cshtml.cs?name=snippet3)]

## <a name="compositechangetoken-class"></a>CompositeChangeToken クラス

1 つのオブジェクト内で 1 つまたは複数の `IChangeToken` インスタンスを表すためには、<xref:Microsoft.Extensions.Primitives.CompositeChangeToken> クラスを使用します。

```csharp
var firstCancellationTokenSource = new CancellationTokenSource();
var secondCancellationTokenSource = new CancellationTokenSource();

var firstCancellationToken = firstCancellationTokenSource.Token;
var secondCancellationToken = secondCancellationTokenSource.Token;

var firstCancellationChangeToken = new CancellationChangeToken(firstCancellationToken);
var secondCancellationChangeToken = new CancellationChangeToken(secondCancellationToken);

var compositeChangeToken = 
    new CompositeChangeToken(
        new List<IChangeToken> 
        {
            firstCancellationChangeToken, 
            secondCancellationChangeToken
        });
```

複合トークンの `HasChanged` は、いずれかの表現されているトークン `HasChanged` が `true` である場合に、`true` を報告します。 複合トークンの `ActiveChangeCallbacks` は、いずれかの表現されているトークン `ActiveChangeCallbacks` が `true` である場合に、`true` を報告します。 複数の同時変更イベントが発生すると、複合変更コールバックが 1 回呼び出されます。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

"*変更トークン*" は、状態の変更を追跡するために使用される汎用の低レベル構成ブロックです。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/change-tokens/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="ichangetoken-interface"></a>IChangeToken インターフェイス

<xref:Microsoft.Extensions.Primitives.IChangeToken> により、変更が発生したという通知が伝達されます。 `IChangeToken` は <xref:Microsoft.Extensions.Primitives?displayProperty=fullName> 名前空間内にあります。 [Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)を使用しないアプリの場合は、[Microsoft.Extensions.Primitives](https://www.nuget.org/packages/Microsoft.Extensions.Primitives/) NuGet パッケージのパッケージ参照を作成します。

`IChangeToken` に次の 2 つのプロパティがあります。

* <xref:Microsoft.Extensions.Primitives.IChangeToken.ActiveChangeCallbacks> は、トークンによってコールバックがプロアクティブに生成されるかどうかを示します。 `ActiveChangedCallbacks` が `false` に設定されている場合、コールバックは呼び出されず、アプリは変更のために `HasChanged` をポーリングする必要があります。 変更がまったく発生しない場合、または基になる変更リスナーが破棄されるか無効になっている場合、トークンがまったくキャンセルされない可能性もあります。
* <xref:Microsoft.Extensions.Primitives.IChangeToken.HasChanged> は、変更が発生したかどうかを示す値を受け取ります。

`IChangeToken` インターフェイスには、[RegisterChangeCallback(Action\<Object>, Object)](xref:Microsoft.Extensions.Primitives.IChangeToken.RegisterChangeCallback*) というメソッドが含まれています。これを使うと、トークンが変更されたときに呼び出されるコールバックを登録できます。 コールバックが呼び出される前に `HasChanged` を設定する必要があります。

## <a name="changetoken-class"></a>ChangeToken クラス

<xref:Microsoft.Extensions.Primitives.ChangeToken> は、変更が発生したことの通知を伝達するために使用される静的クラスです。 `ChangeToken` は <xref:Microsoft.Extensions.Primitives?displayProperty=fullName> 名前空間内にあります。 [Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)を使用しないアプリの場合は、[Microsoft.Extensions.Primitives](https://www.nuget.org/packages/Microsoft.Extensions.Primitives/) NuGet パッケージのパッケージ参照を作成します。

[ChangeToken.OnChange(Func\<IChangeToken>, Action)](xref:Microsoft.Extensions.Primitives.ChangeToken.OnChange*) メソッドは、トークンが変更されるたびに呼び出される `Action` を登録します。

* `Func<IChangeToken>` はトークンを生成します。
* `Action` は、トークンが変更されたときに呼び出されます。

[ChangeToken.OnChange\<TState>(Func\<IChangeToken>, Action\<TState>, TState)](xref:Microsoft.Extensions.Primitives.ChangeToken.OnChange*) オーバーロードは、トークン コンシューマー `Action` に渡される追加の `TState` パラメーターを取ります。

`OnChange` では <xref:System.IDisposable> が返されます。 <xref:System.IDisposable.Dispose*> を呼び出すと、トークンによるその後の変更のリッスンが停止され、トークンのリソースが解放されます。

## <a name="example-uses-of-change-tokens-in-aspnet-core"></a>ASP.NET Core での変更のトークンの使用例

変更トークンは、オブジェクトの変更を監視するために ASP.NET Core の主要な領域で使用されます。

* ファイルへの変更を監視するために、<xref:Microsoft.Extensions.FileProviders.IFileProvider> の <xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*> メソッドでは、指定された監視対象のファイルまたはフォルダーの `IChangeToken` が作成されます。
* `IChangeToken` トークンをキャッシュ エントリに追加して、変更時にキャッシュの削除をトリガーすることができます。
* `TOptions` の変更のために、<xref:Microsoft.Extensions.Options.OptionsMonitor`1> の既定の実装である <xref:Microsoft.Extensions.Options.IOptionsMonitor`1> には、1 つ以上の <xref:Microsoft.Extensions.Options.IOptionsChangeTokenSource`1> インスタンスを受け取るオーバーロードが含まれています。 各インスタンスは、オプションの変更を追跡するために変更通知のコールバックを登録する `IChangeToken` を返します。

## <a name="monitor-for-configuration-changes"></a>構成の変更の監視

既定では、ASP.NET Core テンプレートは、[JSON 構成ファイル](xref:fundamentals/configuration/index#json-configuration-provider) ( *appsettings.json* 、*appsettings.Development.json*、および *appsettings.Production.json*) を使用して、アプリの構成設定を読み込みます。

これらのファイルは、`reloadOnChange` パラメーターを受け取る、<xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> 上の [AddJsonFile(IConfigurationBuilder, String, Boolean, Boolean)](xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*) 拡張メソッドを使用して構成されます。 `reloadOnChange` は、ファイルの変更時に構成を再読み込みするかどうかを示します。 この設定は、<xref:Microsoft.AspNetCore.WebHost> の便利なメソッド <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> 内に現れます。

```csharp
config.AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
      .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true, 
          reloadOnChange: true);
```

ファイル ベースの構成は、<xref:Microsoft.Extensions.Configuration.FileConfigurationSource> によって表されます。 `FileConfigurationSource` ではファイルを監視するために <xref:Microsoft.Extensions.FileProviders.IFileProvider> が使用されます。

既定では、`IFileMonitor` は <xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider> によって提供されます。これは、<xref:System.IO.FileSystemWatcher> を使用して、構成ファイルの変更を監視します。

サンプル アプリは、構成の変更を監視するための 2 つの実装を示します。 *appsettings* ファイルのいずれかが変更されると、ファイルを監視する両方の実装によってカスタム コードが実行されます。サンプル アプリによりコンソールにメッセージが書き込まれます。

構成ファイルの `FileSystemWatcher` は、1 つの構成ファイルの変更に対して複数のトークンのコールバックをトリガーできます。 複数のトークンのコールバックがトリガーされたときにカスタム コードが一度だけ実行されるようにするために、サンプルの実装ではファイル ハッシュがチェックされます。 サンプルでは、SHA1 ファイル ハッシュが使用されています。 再試行は、指数バックオフを使用して実装されます。 再実行が提供されるのは、ファイルの新しいハッシュの計算を一時的に妨げるファイル ロックが発生する場合があるためです。

*Utilities/Utilities.cs*:

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Utilities/Utilities.cs?name=snippet1)]

### <a name="simple-startup-change-token"></a>シンプルなスタートアップ変更トークン

構成再読み込みトークンへの変更通知のために、トークン コンシューマー `Action` のコールバックを登録します。

`Startup.Configure`の場合:

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Startup.cs?name=snippet2)]

`config.GetReloadToken()` はトークンを提供します。 コールバックは、`InvokeChanged` メソッドです。

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Startup.cs?name=snippet3)]

コールバックの `state` は、`IHostingEnvironment` を渡すためのに使われます。これは監視する適切な *appsettings* 構成ファイルを指定するのに便利です (たとえば、開発環境の場合は *appsettings.Development.json*)。 ファイル ハッシュは、構成ファイルが 1 回のみ変更されたときの複数のトークンのコールバックのために `WriteConsole` ステートメントが複数回実行されるのを防ぐために使用されます。

このシステムは、アプリが実行されている限り実行され、ユーザーが無効にすることはできません。

### <a name="monitor-configuration-changes-as-a-service"></a>サービスとしての構成の変更の監視

サンプルは次のものを実装します。

* 基本的なスタートアップ トークンの監視
* サービスとしての監視
* 監視を有効および無効にするメカニズム。

サンプルでは `IConfigurationMonitor` インターフェイスが設定されています。

*Extensions/ConfigurationMonitor.cs*:

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet1)]

実装済みのクラスのコンストラクター `ConfigurationMonitor` は、変更通知のコールバックを登録します。

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet2)]

`config.GetReloadToken()` はトークンを提供します。 `InvokeChanged` はコールバック メソッドです。 このインスタンスの `state` は、監視の状態にアクセスするために使用される `IConfigurationMonitor` インスタンスへの参照です。 次の 2 つのプロパティが使用されます。

* `MonitoringEnabled`:コールバックでそのカスタム コードを実行する必要があるかどうかを示します。
* `CurrentState`:UI で使用するための現在の監視状態を説明します。

`InvokeChanged` メソッドは、次の点を除けば、前の方法に似ています。

* `MonitoringEnabled` が `true` ではない限りコードを実行しません。
* 現在の `state` をその `WriteConsole` 出力に出力します。

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet3)]

インスタンス `ConfigurationMonitor` は、`Startup.ConfigureServices` でサービスとして登録されます。

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Startup.cs?name=snippet1)]

インデックス ページは、ユーザーに構成監視の制御を提供します。 `IConfigurationMonitor` のインスタンスは、`IndexModel` に挿入されます。

*Pages/Index.cshtml.cs*:

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Pages/Index.cshtml.cs?name=snippet1)]

構成監視 (`_monitor`) は、監視を有効または無効にし、UI フィードバックの現在の状態を設定するために使用されます。

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Pages/Index.cshtml.cs?name=snippet2)]

`OnPostStartMonitoring` がトリガーされると、監視が有効になり、現在の状態がクリアされます。 `OnPostStopMonitoring` がトリガーされると、監視が無効になり、監視が発生していないことを反映するように状態が設定されます。

UI のボタンを使って監視を有効および無効にできます。

*Pages/Index.cshtml*:

[!code-cshtml[](change-tokens/samples/2.x/SampleApp/Pages/Index.cshtml?name=snippet_Buttons)]

## <a name="monitor-cached-file-changes"></a>キャッシュされたファイルの変更の監視

<xref:Microsoft.Extensions.Caching.Memory.IMemoryCache> を使用して、ファイルの内容をメモリ内にキャッシュすることができます。 メモリ内キャッシュの説明については、「[メモリ内キャッシュ](xref:performance/caching/memory)」トピックを参照してください。 以下に示す実装など、追加の手順を実行しないと、ソース データが変更された場合に *期限切れの* (古い) データがキャッシュから返されます。

たとえば、[スライド式有効期限](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.SlidingExpiration)期間の更新時にキャッシュされたソース ファイルの状態を考慮しないと、キャッシュされたファイル データが古くなります。 データの各要求が、スライド式有効期限を更新しますが、ファイルがキャッシュに再読み込みされることはありません。 ファイルのキャッシュされた内容を使用するアプリの機能は、古い内容を受け取る可能性があります。

ファイル キャッシュのシナリオで変更トークンを使用すると、キャッシュに古いファイルの内容が残ることを防止できます。 サンプル アプリは、このアプローチの実装を示しています。

このサンプルでは、`GetFileContent` を使用して次の操作を実行します。

* ファイルの内容を返します。
* 指数バックオフを使用する再試行アルゴリズムを実装し、ファイル ロックによりファイルの読み取りが一時的に妨げられるケースに対処します。

*Utilities/Utilities.cs*:

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Utilities/Utilities.cs?name=snippet2)]

`FileService` はキャッシュされたファイルの参照の処理するために作成されます。 サービスの `GetFileContent` メソッドの呼び出しは、メモリ内キャッシュからファイルの内容を取得して、呼び出し元 (*Services/FileService.cs*) に戻そうとします。

キャッシュ キーを使用してキャッシュされた内容が見つからない場合、次の操作が実行されます。

1. `GetFileContent` を使用してファイルの内容が取得されます。
1. 変更トークンは、[IFileProviders.Watch](xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*) を使用してファイル プロバイダーから取得します。 ファイルが変更されたときに、トークンのコールバックがトリガーされます。
1. ファイルの内容は、[スライド式有効期限](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.SlidingExpiration)の期間キャシュされます。 ファイルがキャッシュされている間に変更された場合、キャッシュ エントリを削除するために、[MemoryCacheEntryExtensions.AddExpirationToken](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryExtensions.AddExpirationToken*) を使用して変更トークンがアタッチされます。

次の例では、ファイルはアプリの[コンテンツ ルート](xref:fundamentals/index#content-root)に格納されます。 アプリの <xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment.ContentRootPath> を指す <xref:Microsoft.Extensions.FileProviders.IFileProvider> を取得するために、[IHostingEnvironment.ContentRootFileProvider](xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment.ContentRootFileProvider) が使われます。 [IFileInfo.PhysicalPath](xref:Microsoft.Extensions.FileProviders.IFileInfo.PhysicalPath) を使って `filePath` を取得します。

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Services/FileService.cs?name=snippet1)]

`FileService` はメモリ キャッシュ サービスと共にサービス コンテナーに登録されます。

`Startup.ConfigureServices`の場合:

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Startup.cs?name=snippet4)]

ページ モデルでは、このサービスを使ってファイルの内容が読み込まれます。

Index ページの `OnGet` メソッド内 (*Pages/Index.cshtml.cs*):

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Pages/Index.cshtml.cs?name=snippet3)]

## <a name="compositechangetoken-class"></a>CompositeChangeToken クラス

1 つのオブジェクト内で 1 つまたは複数の `IChangeToken` インスタンスを表すためには、<xref:Microsoft.Extensions.Primitives.CompositeChangeToken> クラスを使用します。

```csharp
var firstCancellationTokenSource = new CancellationTokenSource();
var secondCancellationTokenSource = new CancellationTokenSource();

var firstCancellationToken = firstCancellationTokenSource.Token;
var secondCancellationToken = secondCancellationTokenSource.Token;

var firstCancellationChangeToken = new CancellationChangeToken(firstCancellationToken);
var secondCancellationChangeToken = new CancellationChangeToken(secondCancellationToken);

var compositeChangeToken = 
    new CompositeChangeToken(
        new List<IChangeToken> 
        {
            firstCancellationChangeToken, 
            secondCancellationChangeToken
        });
```

複合トークンの `HasChanged` は、いずれかの表現されているトークン `HasChanged` が `true` である場合に、`true` を報告します。 複合トークンの `ActiveChangeCallbacks` は、いずれかの表現されているトークン `ActiveChangeCallbacks` が `true` である場合に、`true` を報告します。 複数の同時変更イベントが発生すると、複合変更コールバックが 1 回呼び出されます。

::: moniker-end

## <a name="additional-resources"></a>その他の技術情報

* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:performance/caching/response>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>
