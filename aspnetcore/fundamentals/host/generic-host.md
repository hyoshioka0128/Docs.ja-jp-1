---
title: ASP.NET Core の .NET 汎用ホスト
author: rick-anderson
description: ASP.NET Core アプリで .NET Core 汎用ホストを使用します。  汎用ホストが担当するのは、アプリの起動および有効期間の管理です。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 4/17/2020
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
uid: fundamentals/host/generic-host
ms.openlocfilehash: ceb766cae1f8a1735f5b578a44f0c481927e47a5
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586749"
---
# <a name="net-generic-host-in-aspnet-core"></a>ASP.NET Core の .NET 汎用ホスト

::: moniker range=">= aspnetcore-5.0"

ASP.NET Core テンプレートでは、.NET Core の汎用ホスト (<xref:Microsoft.Extensions.Hosting.HostBuilder>) が作成されます。

このトピックでは、ASP.NET Core での .NET 汎用ホストの使用方法について説明します。 コンソール アプリで .NET 汎用ホストを使用する方法の詳細については、「[.NET 汎用ホスト](/dotnet/core/extensions/generic-host)」を参照してください。

## <a name="host-definition"></a>ホストの定義

"*ホスト*" とは、以下のようなアプリのリソースをカプセル化するオブジェクトです:

* 依存関係の挿入 (DI)
* ログの記録
* 構成
* `IHostedService` の実装

ホストが起動すると、サービス コンテナーのホステッド サービスのコレクションに登録されている <xref:Microsoft.Extensions.Hosting.IHostedService> の各実装で <xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync%2A?displayProperty=nameWithType> が呼び出されます。 Web アプリでは、`IHostedService` 実装の 1 つが [ HTTP サーバー実装](xref:fundamentals/index#servers)を起動する Web サービスとなります。

アプリの相互依存するすべてのリソースを 1 つのオブジェクトに含める主な理由は、アプリの起動と正常なシャットダウンの制御の有効期間の管理のためです。

## <a name="set-up-a-host"></a>ホストを設定する

ホストは通常、`Program` クラス内のコードによって構成、ビルド、および実行されます。 `Main` メソッド:

* `CreateHostBuilder` メソッドを呼び出して、builder オブジェクトを作成および構成します。
* builder オブジェクト上で `Build` メソッドと `Run` メソッドを呼び出します。

ASP.NET Core の web テンプレートでは、ホストを作成するために、次のコードが生成されます。

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

次のコードを実行すると、DI コンテナーに追加された `IHostedService` の実装を使用して、非 HTTP ワークロードが作成されます。

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureServices((hostContext, services) =>
            {
               services.AddHostedService<Worker>();
            });
}
```

HTTP ワークロードの場合、`Main` メソッドは同じですが、`CreateHostBuilder` によって `ConfigureWebHostDefaults` が呼び出されます。

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

Entity Framework Core がアプリで使用されている場合は、`CreateHostBuilder` メソッドの名前またはシグネチャを変更しないでください。 [Entity Framework Core ツール](/ef/core/miscellaneous/cli/) では、アプリを実行することなくホストを構成する `CreateHostBuilder` メソッドを検出することが想定されています。 詳細については、「[デザイン時 DbContext 作成](/ef/core/miscellaneous/cli/dbcontext-creation)」をご覧ください。

## <a name="default-builder-settings"></a>既定の builder 設定

<xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder*> メソッド:

* [コンテンツ ルート](xref:fundamentals/index#content-root)を、<xref:System.IO.Directory.GetCurrentDirectory*> によって返されるパスに設定します。
* 次からホスト構成を読み込みます。
  * プレフィックス `DOTNET_` が付いた環境変数。
  * コマンド ライン引数。
* 次からアプリの構成を読み込みます。
  * *appsettings.json*.
  * *appsettings.{Environment}.json*。
  * `Development` 環境でアプリが実行される場合に使用される[ユーザー シークレット](xref:security/app-secrets)。
  * 環境変数。
  * コマンド ライン引数。
* 次の[ログ](xref:fundamentals/logging/index) プロバイダーを追加します。
  * コンソール
  * デバッグ
  * EventSource
  * イベント ログ (Windows で実行されている場合のみ)
* 環境が [開発] になっている場合は、[スコープの検証](xref:fundamentals/dependency-injection#scope-validation)と[依存関係の検証](xref:Microsoft.Extensions.DependencyInjection.ServiceProviderOptions.ValidateOnBuild)を有効にします。

<xref:Microsoft.Extensions.Hosting.GenericHostBuilderExtensions.ConfigureWebHostDefaults*> メソッド:

* プレフィックス `ASPNETCORE_` が付いた環境変数からホスト構成を読み込みます。
* [Kestrel](xref:fundamentals/servers/kestrel) サーバーを Web サーバーとして設定し、アプリのホスティング構成プロバイダーを使用してそれを構成します。 Kestrel サーバーの既定のオプションについては、<xref:fundamentals/servers/kestrel/options> を参照してください。
* [Host Filtering middleware](xref:fundamentals/servers/kestrel/host-filtering) を追加します。
* `ASPNETCORE_FORWARDEDHEADERS_ENABLED` が `true` の場合、[Forwarded Headers Middleware](xref:host-and-deploy/proxy-load-balancer#forwarded-headers) を追加します。
* IIS 統合を有効にします。 IIS の既定のオプションについては、<xref:host-and-deploy/iis/index#iis-options> を参照してください。

この記事で後述する「[すべての種類のアプリの設定](#settings-for-all-app-types)」および「[Web アプリの設定](#settings-for-web-apps)」セクションに、既定のビルダー設定をオーバーライドする方法を示します。

## <a name="framework-provided-services"></a>フレームワークが提供するサービス

以下のサービスは、自動的に登録されます。

* [IHostApplicationLifetime](#ihostapplicationlifetime)
* [IHostLifetime](#ihostlifetime)
* [IHostEnvironment / IWebHostEnvironment](#ihostenvironment)

フレームワークによって提供されるサービスの詳細については、<xref:fundamentals/dependency-injection#framework-provided-services> を参照してください。

## <a name="ihostapplicationlifetime"></a>IHostApplicationLifetime

起動後タスクとグレースフル シャットダウン タスクを処理するために <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime> (旧称 `IApplicationLifetime`) サービスを任意のクラスに注入します。 インターフェイス上の 3 つのプロパティは、アプリの起動およびアプリの停止のイベント ハンドラー メソッドを登録するために使用されるキャンセル トークンです。 インターフェイスには `StopApplication` メソッドも含まれています。

次の例は、`IHostApplicationLifetime` イベントを登録する `IHostedService` の実装です。

[!code-csharp[](generic-host/samples-snapshot/3.x/LifetimeEventsHostedService.cs?name=snippet_LifetimeEvents)]

## <a name="ihostlifetime"></a>IHostLifetime

<xref:Microsoft.Extensions.Hosting.IHostLifetime> 実装では、ホストを開始および停止するタイミングが制御されます。 登録されている最後の実装が使用されます。

`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` は、既定の `IHostLifetime` 実装です。 `ConsoleLifetime`:

* <kbd>Ctrl</kbd> + <kbd>C</kbd>/SIGINT または SIGTERM をリッスンし、<xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime.StopApplication*> を呼び出して、シャットダウン プロセスを開始します。
* [RunAsync](#runasync) や [WaitForShutdownAsync](#waitforshutdownasync) などの拡張機能のブロックを解除します。

## <a name="ihostenvironment"></a>IHostEnvironment

次の設定に関する情報を取得するため、クラスに <xref:Microsoft.Extensions.Hosting.IHostEnvironment> サービスを注入します。

* [ApplicationName](#applicationname)
* [EnvironmentName](#environmentname)
* [ContentRootPath](#contentroot)

Web アプリで `IWebHostEnvironment` インターフェイスを実装します。これにより、`IHostEnvironment` が継承され、[WebRootPath](#webroot) が追加されます。

## <a name="host-configuration"></a>ホストの構成

ホストの構成は、<xref:Microsoft.Extensions.Hosting.IHostEnvironment> 実装のプロパティで使用されます。

ホストの構成は、<xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> 内の [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration) から使用できます。 `ConfigureAppConfiguration` の後、`HostBuilderContext.Configuration` はアプリの構成に置き換えられます。

ホストの構成を追加するには、`IHostBuilder` 上で <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> を呼び出します。 `ConfigureHostConfiguration` を複数回呼び出して結果を追加できます。 ホストは、指定されたキーで最後に値を設定したオプションを使用します。

プレフィックス `DOTNET_` を持つ環境変数プロバイダーとコマンドライン引数が、`CreateDefaultBuilder` によって組み込まれます。 Web アプリの場合は、プレフィックス `ASPNETCORE_` を持つ環境変数プロバイダーが追加されます。 環境変数が読み取られると、プレフィックスは削除されます。 たとえば、`ASPNETCORE_ENVIRONMENT` の環境変数の値が `environment` キーのホスト構成値になります。

次の例では、ホストの構成を作成します。

[!code-csharp[](generic-host/samples-snapshot/3.x/Program.cs?name=snippet_HostConfig)]

## <a name="app-configuration"></a>アプリの構成

アプリの構成は、`IHostBuilder` 上で <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> を呼び出すことで作成されます。 `ConfigureAppConfiguration` を複数回呼び出して結果を追加できます。 アプリは、指定されたキーで最後に値を設定したオプションを使用します。 

`ConfigureAppConfiguration` によって作成された構成は、[ HostBuilderContext.Configuration ](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration*) で、以降の操作のために、かつ DI からのサービスとして利用できます。 ホストの構成はアプリの構成にも追加されます。

詳細については、「[ASP.NET Core の構成](xref:fundamentals/configuration/index#configureappconfiguration)」を参照してください。

## <a name="settings-for-all-app-types"></a>すべての種類のアプリの設定

このセクションでは、HTTP のワークロードと HTTP 以外のワークロードの両方に適用されるホストの設定を一覧します。 既定では、これらの設定を構成するのに使用する環境変数には、プレフィックスとして `DOTNET_` または `ASPNETCORE_` を付けることができます。 詳細については、「[既定の builder 設定](#default-builder-settings)」を参照してください。

<!-- In the following sections, two spaces at end of line are used to force line breaks in the rendered page. -->

### <a name="applicationname"></a>ApplicationName

[IHostEnvironment.ApplicationName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ApplicationName*) プロパティは、ホストの構築時にホストの構成から設定されます。

**キー**: `applicationName`  
**型**: `string`  
**既定**: アプリのエントリ ポイントを含むアセンブリの名前。  
**環境変数**: `<PREFIX_>APPLICATIONNAME`

この値を設定するには、環境変数を使用します。 

### <a name="contentroot"></a>ContentRoot

[IHostEnvironment.ContentRootPath](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath*) プロパティでは、ホストがコンテンツ ファイルの検索を開始する位置が決定されます。 パスが存在しない場合は、ホストを起動できません。

**キー**: `contentRoot`  
**型**: `string`  
**既定**: アプリ アセンブリが存在するフォルダー。  
**環境変数**: `<PREFIX_>CONTENTROOT`

この値を設定するには、環境変数を使用するか、または `IHostBuilder` 上で `UseContentRoot` を呼び出します。

```csharp
Host.CreateDefaultBuilder(args)
    .UseContentRoot("c:\\content-root")
    //...
```

詳細については次を参照してください:

* [基礎: コンテンツ ルート](xref:fundamentals/index#content-root)
* [WebRoot](#webroot)

### <a name="environmentname"></a>EnvironmentName

[IHostEnvironment.EnvironmentName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.EnvironmentName*) プロパティは、任意の値に設定することができます。 フレームワークで定義された値には `Development`、`Staging`、`Production` が含まれます。 値は大文字と小文字が区別されません。

**キー**: `environment`  
**型**: `string`  
**既定値**: `Production`  
**環境変数**: `<PREFIX_>ENVIRONMENT`

この値を設定するには、環境変数を使用するか、または `IHostBuilder` 上で `UseEnvironment` を呼び出します。

```csharp
Host.CreateDefaultBuilder(args)
    .UseEnvironment("Development")
    //...
```

### <a name="shutdowntimeout"></a>ShutdownTimeout

[HostOptions.ShutdownTimeout](xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*) では、<xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> のタイムアウトが設定されます。 既定値は 5 秒です。  タイムアウト期間中、ホストでは次のことが行われます。

* [IHostApplicationLifetime.ApplicationStopping](/dotnet/api/microsoft.extensions.hosting.ihostapplicationlifetime.applicationstopping) をトリガーします。
* ホステッド サービスの停止を試み、停止に失敗したサービスのエラーをログに記録します。

すべてのホステッド サービスが停止する前にタイムアウト時間が切れた場合、残っているアクティブなサービスはアプリのシャットダウン時に停止します。 処理が完了していない場合でも、サービスは停止します。 サービスが停止するまでにさらに時間が必要な場合は、タイムアウト値を増やします。

**キー**: `shutdownTimeoutSeconds`  
**型**: `int`  
**既定**:5 秒  
**環境変数**: `<PREFIX_>SHUTDOWNTIMEOUTSECONDS`

この値を設定するには、環境変数を使用するか、または `HostOptions` を構成します。 次の例では、タイムアウトを 20 秒に設定します。

[!code-csharp[](generic-host/samples-snapshot/3.x/Program.cs?name=snippet_HostOptions)]

### <a name="disable-app-configuration-reload-on-change"></a>変更時にアプリ構成の再度読み込みを無効にする

[既定](xref:fundamentals/configuration/index#default)では、 *appsettings.json* と *appsettings.{Environment}.json* は、ファイルの変更時に再度読み込まれます。 ASP.NET Core 5.0 以降でこの再度読み込み動作を無効にするには、`hostBuilder:reloadConfigOnChange` キーを `false` に設定します。

**キー**: `hostBuilder:reloadConfigOnChange`  
**型**: `bool` (`true` または `1`)  
**既定値**: `true`  
**コマンドライン引数**: `hostBuilder:reloadConfigOnChange`  
**環境変数**: `<PREFIX_>hostBuilder:reloadConfigOnChange`

> [!WARNING]
> コロン (`:`) の区切り記号は、すべてのプラットフォームの環境変数階層キーには対応していません。 詳細については、「[環境変数](xref:fundamentals/configuration/index#environment-variables)」を参照してください。

## <a name="settings-for-web-apps"></a>Web アプリの設定

一部のホスト設定は、HTTP のワークロードにのみに適用されます。 既定では、これらの設定を構成するのに使用する環境変数には、プレフィックスとして `DOTNET_` または `ASPNETCORE_` を付けることができます。

`IWebHostBuilder` 上の拡張メソッドはこれらの設定で使用できます。 次の例に示すように、拡張メソッドを呼び出す方法を示すコード サンプルでは `webBuilder` が `IWebHostBuilder` のインスタンスであると想定しています。

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.CaptureStartupErrors(true);
            webBuilder.UseStartup<Startup>();
        });
```

### <a name="capturestartuperrors"></a>CaptureStartupErrors

`false` の場合、起動時にエラーが発生するとホストが終了します。 `true` の場合、ホストは起動時に例外をキャプチャして、サーバーを起動しようとします。

**キー**: `captureStartupErrors`  
**型**: `bool` (`true` または `1`)  
**既定**:アプリが IIS の背後で Kestrel を使用して実行されている場合 (既定値は `true`) を除き、既定では `false` に設定されます。  
**環境変数**: `<PREFIX_>CAPTURESTARTUPERRORS`

この値を設定するには、構成を使用するか、または `CaptureStartupErrors` を呼び出します。

```csharp
webBuilder.CaptureStartupErrors(true);
```

### <a name="detailederrors"></a>DetailedErrors

有効にされている場合、または環境が `Development` である場合、アプリによって詳細なエラーがキャプチャされます。

**キー**: `detailedErrors`  
**型**: `bool` (`true` または `1`)  
**既定値**: `false`  
**環境変数**: `<PREFIX_>_DETAILEDERRORS`

この値を設定するには、構成を使用するか、または `UseSetting` を呼び出します。

```csharp
webBuilder.UseSetting(WebHostDefaults.DetailedErrorsKey, "true");
```

### <a name="hostingstartupassemblies"></a>HostingStartupAssemblies

起動時に読み込むホスティング スタートアップ アセンブリのセミコロンで区切られた文字列。 構成値は既定で空の文字列に設定されますが、ホスティング スタートアップ アセンブリには常にアプリのアセンブリが含まれます。 ホスティング スタートアップ アセンブリが提供されている場合、アプリが起動中に共通サービスをビルドしたときに読み込むためにアプリのアセンブリに追加されます。

**キー**: `hostingStartupAssemblies`  
**型**: `string`  
**既定**:空の文字列  
**環境変数**: `<PREFIX_>_HOSTINGSTARTUPASSEMBLIES`

この値を設定するには、構成を使用するか、または `UseSetting` を呼び出します。

```csharp
webBuilder.UseSetting(WebHostDefaults.HostingStartupAssembliesKey, "assembly1;assembly2");
```

### <a name="hostingstartupexcludeassemblies"></a>HostingStartupExcludeAssemblies

起動時に除外するホスティング スタートアップ アセンブリのセミコロン区切り文字列。

**キー**: `hostingStartupExcludeAssemblies`  
**型**: `string`  
**既定**:空の文字列  
**環境変数**: `<PREFIX_>_HOSTINGSTARTUPEXCLUDEASSEMBLIES`

この値を設定するには、構成を使用するか、または `UseSetting` を呼び出します。

```csharp
webBuilder.UseSetting(WebHostDefaults.HostingStartupExcludeAssembliesKey, "assembly1;assembly2");
```

### <a name="https_port"></a>HTTPS_Port

HTTPS リダイレクト ポート。 [HTTPS の適用](xref:security/enforcing-ssl)に使用されます。

**キー**: `https_port`  
**型**: `string`  
**既定**:既定値は設定されていません。  
**環境変数**: `<PREFIX_>HTTPS_PORT`

この値を設定するには、構成を使用するか、または `UseSetting` を呼び出します。

```csharp
webBuilder.UseSetting("https_port", "8080");
```

### <a name="preferhostingurls"></a>PreferHostingUrls

`IServer` の実装で構成されている URL ではなく、`IWebHostBuilder` で構成されている URL でホストがリッスンするかどうかを示します。

**キー**: `preferHostingUrls`  
**型**: `bool` (`true` または `1`)  
**既定値**: `true`  
**環境変数**: `<PREFIX_>_PREFERHOSTINGURLS`

この値を設定するには、環境変数を使用するか、または `PreferHostingUrls` を呼び出します。

```csharp
webBuilder.PreferHostingUrls(false);
```

### <a name="preventhostingstartup"></a>PreventHostingStartup

アプリのアセンブリで構成されているホスティング スタートアップ アセンブリを含む、ホスティング スタートアップ アセンブリの自動読み込みを回避します。 詳細については、「<xref:fundamentals/configuration/platform-specific-configuration>」を参照してください。

**キー**: `preventHostingStartup`  
**型**: `bool` (`true` または `1`)  
**既定値**: `false`  
**環境変数**: `<PREFIX_>_PREVENTHOSTINGSTARTUP`

この値を設定するには、環境変数を使用するか、または `UseSetting` を呼び出します。

```csharp
webBuilder.UseSetting(WebHostDefaults.PreventHostingStartupKey, "true");
```

### <a name="startupassembly"></a>StartupAssembly

`Startup` クラスを検索するアセンブリ。

**キー**: `startupAssembly`  
**型**: `string`  
**既定**:アプリのアセンブリ  
**環境変数**: `<PREFIX_>STARTUPASSEMBLY`

この値を設定するには、環境変数を使用するか、または `UseStartup` を呼び出します。 `UseStartup` は、アセンブリ名 (`string`) または型 (`TStartup`) を取ることができます。 複数の `UseStartup` メソッドが呼び出された場合は、最後のメソッドが優先されます。

```csharp
webBuilder.UseStartup("StartupAssemblyName");
```

```csharp
webBuilder.UseStartup<Startup>();
```

### <a name="urls"></a>URL

サーバーが要求をリッスンする必要があるポートとプロトコルを含む IP アドレスまたはホスト アドレスを示すセミコロンで区切られたリスト。 たとえば、`http://localhost:123` のようにします。 "\*" を使用し、サーバーが指定されたポートとプロトコル (`http://*:5000` など) を使用して IP アドレスまたはホスト名に関する要求をリッスンする必要があることを示します。 プロトコル (`http://` または `https://`) は各 URL に含める必要があります。 サポートされている形式はサーバー間で異なります。

**キー**: `urls`  
**型**: `string`  
**既定値**: `http://localhost:5000` および `https://localhost:5001`  
**環境変数**: `<PREFIX_>URLS`

この値を設定するには、環境変数を使用するか、または `UseUrls` を呼び出します。

```csharp
webBuilder.UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002");
```

Kestrel には独自のエンドポイント構成 API があります。 詳細については、「<xref:fundamentals/servers/kestrel/endpoints>」を参照してください。

### <a name="webroot"></a>WebRoot

[IWebHostEnvironment.WebRootPath](xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment.WebRootPath) プロパティでは、アプリの静的アセットへの相対パスが決定されます。 パスが存在しない場合は、no-op ファイル プロバイダーが使用されます。  

**キー**: `webroot`  
**型**: `string`  
**既定**:既定値は、`wwwroot` です。 *{content root}/wwwroot* へのパスが存在する必要があります。  
**環境変数**: `<PREFIX_>WEBROOT`

この値を設定するには、環境変数を使用するか、または `IWebHostBuilder` 上で `UseWebRoot` を呼び出します。

```csharp
webBuilder.UseWebRoot("public");
```

詳細については次を参照してください:

* [基礎: Web ルート](xref:fundamentals/index#web-root)
* [ContentRoot](#contentroot)

## <a name="manage-the-host-lifetime"></a>ホストの有効期間を管理する

組み込みの <xref:Microsoft.Extensions.Hosting.IHost> 実装でメソッドを呼び出して、アプリの開始および停止を行います。 これらのメソッドは、サービス コンテナーに登録されているすべての <xref:Microsoft.Extensions.Hosting.IHostedService> 実装に影響を与えます。

### <a name="run"></a>実行

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Run*> はアプリを実行し、ホストがシャットダウンされるまで呼び出し元のスレッドをブロックします。

### <a name="runasync"></a>RunAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync*> はアプリを実行し、キャンセル トークンまたはシャットダウンがトリガーされると完了する <xref:System.Threading.Tasks.Task> を返します。

### <a name="runconsoleasync"></a>RunConsoleAsync

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.RunConsoleAsync*> は、コンソールのサポートを有効にし、ホストをビルドして開始した後、<kbd>Ctrl</kbd> + <kbd>C</kbd>/SIGINT または SIGTERM がシャットダウンするのを待機します。

### <a name="start"></a>[開始]

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Start*> は、ホストを同期的に開始します。

### <a name="startasync"></a>StartAsync

<xref:Microsoft.Extensions.Hosting.IHost.StartAsync*> ではホストが開始され、キャンセル トークンまたはシャットダウンがトリガーされると完了する <xref:System.Threading.Tasks.Task> が返されます。 

<xref:Microsoft.Extensions.Hosting.IHostLifetime.WaitForStartAsync*> は `StartAsync` の開始時に呼び出され、これが完了するまで待機してから続行します。 これを使って、外部イベントによって通知されるまで開始を遅らせることができます。

### <a name="stopasync"></a>StopAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.StopAsync*> は、指定されたタイムアウト内でホストの停止を試みます。

### <a name="waitforshutdown"></a>WaitForShutdown

<kbd>Ctrl</kbd> + <kbd>C</kbd>/SIGINT や SIGTERM を介するなどして IHostLifetime によってシャットダウンがトリガーされるまで、<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*> では呼び出し側スレッドがブロックされます。

### <a name="waitforshutdownasync"></a>WaitForShutdownAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdownAsync*> が返す <xref:System.Threading.Tasks.Task> は、提供されたトークンによってシャットダウンがトリガーされると完了し、<xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> を呼び出します。

### <a name="external-control"></a>外部コントロール

ホストの有効期間の直接コントロールは、外部から呼び出すことができるメソッドを使って実現できます。

```csharp
public class Program
{
    private IHost _host;

    public Program()
    {
        _host = new HostBuilder()
            .Build();
    }

    public async Task StartAsync()
    {
        _host.StartAsync();
    }

    public async Task StopAsync()
    {
        using (_host)
        {
            await _host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

ASP.NET Core テンプレートでは、.NET Core の汎用ホスト (<xref:Microsoft.Extensions.Hosting.HostBuilder>) が作成されます。

このトピックでは、ASP.NET Core での .NET 汎用ホストの使用方法について説明します。 コンソール アプリで .NET 汎用ホストを使用する方法の詳細については、「[.NET 汎用ホスト](/dotnet/core/extensions/generic-host)」を参照してください。

## <a name="host-definition"></a>ホストの定義

"*ホスト*" とは、以下のようなアプリのリソースをカプセル化するオブジェクトです:

* 依存関係の挿入 (DI)
* ログの記録
* 構成
* `IHostedService` の実装

ホストが起動すると、サービス コンテナーのホステッド サービスのコレクションに登録されている <xref:Microsoft.Extensions.Hosting.IHostedService> の各実装で <xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync%2A?displayProperty=nameWithType> が呼び出されます。 Web アプリでは、`IHostedService` 実装の 1 つが [ HTTP サーバー実装](xref:fundamentals/index#servers)を起動する Web サービスとなります。

アプリの相互依存するすべてのリソースを 1 つのオブジェクトに含める主な理由は、アプリの起動と正常なシャットダウンの制御の有効期間の管理のためです。

## <a name="set-up-a-host"></a>ホストを設定する

ホストは通常、`Program` クラス内のコードによって構成、ビルド、および実行されます。 `Main` メソッド:

* `CreateHostBuilder` メソッドを呼び出して、builder オブジェクトを作成および構成します。
* builder オブジェクト上で `Build` メソッドと `Run` メソッドを呼び出します。

ASP.NET Core の web テンプレートでは、汎用ホストを作成するために次のコードが生成されます。

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

次のコードでは、HTTP 以外のワークロードを使用して、汎用ホストを作成します。 次のコードでは、`IHostedService` の実装が DI コンテナーに追加されます。

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureServices((hostContext, services) =>
            {
               services.AddHostedService<Worker>();
            });
}
```

HTTP ワークロードの場合、`Main` メソッドは同じですが、`CreateHostBuilder` によって `ConfigureWebHostDefaults` が呼び出されます。

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

上記のコードは、ASP.NET Core テンプレートによって生成されます。

Entity Framework Core がアプリで使用されている場合は、`CreateHostBuilder` メソッドの名前またはシグネチャを変更しないでください。 [Entity Framework Core ツール](/ef/core/miscellaneous/cli/) では、アプリを実行することなくホストを構成する `CreateHostBuilder` メソッドを検出することが想定されています。 詳細については、「[デザイン時 DbContext 作成](/ef/core/miscellaneous/cli/dbcontext-creation)」をご覧ください。

## <a name="default-builder-settings"></a>既定の builder 設定

<xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder%2A> メソッド:

* [コンテンツ ルート](xref:fundamentals/index#content-root)を、<xref:System.IO.Directory.GetCurrentDirectory%2A> によって返されるパスに設定します。
* 次からホスト構成を読み込みます。
  * プレフィックス `DOTNET_` が付いた環境変数。
  * コマンド ライン引数。
* 次からアプリの構成を読み込みます。
  * *appsettings.json*.
  * *appsettings.{Environment}.json*。
  * `Development` 環境でアプリが実行される場合に使用される[ユーザー シークレット](xref:security/app-secrets)。
  * 環境変数。
  * コマンド ライン引数。
* 次の[ログ](xref:fundamentals/logging/index) プロバイダーを追加します。
  * コンソール
  * デバッグ
  * EventSource
  * イベント ログ (Windows で実行されている場合のみ)
* 環境が [開発] になっている場合は、[スコープの検証](xref:fundamentals/dependency-injection#scope-validation)と[依存関係の検証](xref:Microsoft.Extensions.DependencyInjection.ServiceProviderOptions.ValidateOnBuild)を有効にします。

`ConfigureWebHostDefaults` メソッド:

* プレフィックス `ASPNETCORE_` が付いた環境変数からホスト構成を読み込みます。
* [Kestrel](xref:fundamentals/servers/kestrel) サーバーを Web サーバーとして設定し、アプリのホスティング構成プロバイダーを使用してそれを構成します。 Kestrel サーバーの既定のオプションについては、<xref:fundamentals/servers/kestrel#kestrel-options> を参照してください。
* [Host Filtering middleware](xref:fundamentals/servers/kestrel#host-filtering) を追加します。
* `ASPNETCORE_FORWARDEDHEADERS_ENABLED` が `true` の場合、[Forwarded Headers Middleware](xref:host-and-deploy/proxy-load-balancer#forwarded-headers) を追加します。
* IIS 統合を有効にします。 IIS の既定のオプションについては、<xref:host-and-deploy/iis/index#iis-options> を参照してください。

この記事で後述する「[すべての種類のアプリの設定](#settings-for-all-app-types)」および「[Web アプリの設定](#settings-for-web-apps)」セクションに、既定のビルダー設定をオーバーライドする方法を示します。

## <a name="framework-provided-services"></a>フレームワークが提供するサービス

以下のサービスは、自動的に登録されます。

* [IHostApplicationLifetime](#ihostapplicationlifetime)
* [IHostLifetime](#ihostlifetime)
* [IHostEnvironment / IWebHostEnvironment](#ihostenvironment)

フレームワークによって提供されるサービスの詳細については、<xref:fundamentals/dependency-injection#framework-provided-services> を参照してください。

## <a name="ihostapplicationlifetime"></a>IHostApplicationLifetime

起動後タスクとグレースフル シャットダウン タスクを処理するために <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime> (旧称 `IApplicationLifetime`) サービスを任意のクラスに注入します。 インターフェイス上の 3 つのプロパティは、アプリの起動およびアプリの停止のイベント ハンドラー メソッドを登録するために使用されるキャンセル トークンです。 インターフェイスには `StopApplication` メソッドも含まれています。

次の例は、`IHostApplicationLifetime` イベントを登録する `IHostedService` の実装です。

[!code-csharp[](generic-host/samples-snapshot/3.x/LifetimeEventsHostedService.cs?name=snippet_LifetimeEvents)]

## <a name="ihostlifetime"></a>IHostLifetime

<xref:Microsoft.Extensions.Hosting.IHostLifetime> 実装では、ホストを開始および停止するタイミングが制御されます。 登録されている最後の実装が使用されます。

`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` は、既定の `IHostLifetime` 実装です。 `ConsoleLifetime`:

* <kbd>Ctrl</kbd> + <kbd>C</kbd>/SIGINT または SIGTERM をリッスンし、<xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime.StopApplication%2A> を呼び出して、シャットダウン プロセスを開始します。
* [RunAsync](#runasync) や [WaitForShutdownAsync](#waitforshutdownasync) などの拡張機能のブロックを解除します。

## <a name="ihostenvironment"></a>IHostEnvironment

次の設定に関する情報を取得するため、クラスに <xref:Microsoft.Extensions.Hosting.IHostEnvironment> サービスを注入します。

* [ApplicationName](#applicationname)
* [EnvironmentName](#environmentname)
* [ContentRootPath](#contentroot)

Web アプリで `IWebHostEnvironment` インターフェイスを実装します。これにより、`IHostEnvironment` が継承され、[WebRootPath](#webroot) が追加されます。

## <a name="host-configuration"></a>ホストの構成

ホストの構成は、<xref:Microsoft.Extensions.Hosting.IHostEnvironment> 実装のプロパティで使用されます。

ホストの構成は、<xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration%2A> 内の [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration) から使用できます。 `ConfigureAppConfiguration` の後、`HostBuilderContext.Configuration` はアプリの構成に置き換えられます。

ホストの構成を追加するには、`IHostBuilder` 上で <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration%2A> を呼び出します。 `ConfigureHostConfiguration` を複数回呼び出して結果を追加できます。 ホストは、指定されたキーで最後に値を設定したオプションを使用します。

プレフィックス `DOTNET_` を持つ環境変数プロバイダーとコマンドライン引数が、`CreateDefaultBuilder` によって組み込まれます。 Web アプリの場合は、プレフィックス `ASPNETCORE_` を持つ環境変数プロバイダーが追加されます。 環境変数が読み取られると、プレフィックスは削除されます。 たとえば、`ASPNETCORE_ENVIRONMENT` の環境変数の値が `environment` キーのホスト構成値になります。

次の例では、ホストの構成を作成します。

[!code-csharp[](generic-host/samples-snapshot/3.x/Program.cs?name=snippet_HostConfig)]

## <a name="app-configuration"></a>アプリの構成

アプリの構成は、`IHostBuilder` 上で <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> を呼び出すことで作成されます。 `ConfigureAppConfiguration` を複数回呼び出して結果を追加できます。 アプリは、指定されたキーで最後に値を設定したオプションを使用します。 

`ConfigureAppConfiguration` によって作成された構成は、[ HostBuilderContext.Configuration ](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration*) で、以降の操作のために、かつ DI からのサービスとして利用できます。 ホストの構成はアプリの構成にも追加されます。

詳細については、「[ASP.NET Core の構成](xref:fundamentals/configuration/index#configureappconfiguration)」を参照してください。

## <a name="settings-for-all-app-types"></a>すべての種類のアプリの設定

このセクションでは、HTTP のワークロードと HTTP 以外のワークロードの両方に適用されるホストの設定を一覧します。 既定では、これらの設定を構成するのに使用する環境変数には、プレフィックスとして `DOTNET_` または `ASPNETCORE_` を付けることができます。

<!-- In the following sections, two spaces at end of line are used to force line breaks in the rendered page. -->

### <a name="applicationname"></a>ApplicationName

[IHostEnvironment.ApplicationName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ApplicationName*) プロパティは、ホストの構築時にホストの構成から設定されます。

**キー**: `applicationName`  
**型**: `string`  
**既定**: アプリのエントリ ポイントを含むアセンブリの名前。  
**環境変数**: `<PREFIX_>APPLICATIONNAME`

この値を設定するには、環境変数を使用します。 

### <a name="contentroot"></a>ContentRoot

[IHostEnvironment.ContentRootPath](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath*) プロパティでは、ホストがコンテンツ ファイルの検索を開始する位置が決定されます。 パスが存在しない場合は、ホストを起動できません。

**キー**: `contentRoot`  
**型**: `string`  
**既定**: アプリ アセンブリが存在するフォルダー。  
**環境変数**: `<PREFIX_>CONTENTROOT`

この値を設定するには、環境変数を使用するか、または `IHostBuilder` 上で `UseContentRoot` を呼び出します。

```csharp
Host.CreateDefaultBuilder(args)
    .UseContentRoot("c:\\content-root")
    //...
```

詳細については次を参照してください:

* [基礎: コンテンツ ルート](xref:fundamentals/index#content-root)
* [WebRoot](#webroot)

### <a name="environmentname"></a>EnvironmentName

[IHostEnvironment.EnvironmentName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.EnvironmentName*) プロパティは、任意の値に設定することができます。 フレームワークで定義された値には `Development`、`Staging`、`Production` が含まれます。 値は大文字と小文字が区別されません。

**キー**: `environment`  
**型**: `string`  
**既定値**: `Production`  
**環境変数**: `<PREFIX_>ENVIRONMENT`

この値を設定するには、環境変数を使用するか、または `IHostBuilder` 上で `UseEnvironment` を呼び出します。

```csharp
Host.CreateDefaultBuilder(args)
    .UseEnvironment("Development")
    //...
```

### <a name="shutdowntimeout"></a>ShutdownTimeout

[HostOptions.ShutdownTimeout](xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*) では、<xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> のタイムアウトが設定されます。 既定値は 5 秒です。  タイムアウト期間中、ホストでは次のことが行われます。

* [IHostApplicationLifetime.ApplicationStopping](/dotnet/api/microsoft.extensions.hosting.ihostapplicationlifetime.applicationstopping) をトリガーします。
* ホステッド サービスの停止を試み、停止に失敗したサービスのエラーをログに記録します。

すべてのホステッド サービスが停止する前にタイムアウト時間が切れた場合、残っているアクティブなサービスはアプリのシャットダウン時に停止します。 処理が完了していない場合でも、サービスは停止します。 サービスが停止するまでにさらに時間が必要な場合は、タイムアウト値を増やします。

**キー**: `shutdownTimeoutSeconds`  
**型**: `int`  
**既定**:5 秒  
**環境変数**: `<PREFIX_>SHUTDOWNTIMEOUTSECONDS`

この値を設定するには、環境変数を使用するか、または `HostOptions` を構成します。 次の例では、タイムアウトを 20 秒に設定します。

[!code-csharp[](generic-host/samples-snapshot/3.x/Program.cs?name=snippet_HostOptions)]

## <a name="settings-for-web-apps"></a>Web アプリの設定

一部のホスト設定は、HTTP のワークロードにのみに適用されます。 既定では、これらの設定を構成するのに使用する環境変数には、プレフィックスとして `DOTNET_` または `ASPNETCORE_` を付けることができます。

`IWebHostBuilder` 上の拡張メソッドはこれらの設定で使用できます。 次の例に示すように、拡張メソッドを呼び出す方法を示すコード サンプルでは `webBuilder` が `IWebHostBuilder` のインスタンスであると想定しています。

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.CaptureStartupErrors(true);
            webBuilder.UseStartup<Startup>();
        });
```

### <a name="capturestartuperrors"></a>CaptureStartupErrors

`false` の場合、起動時にエラーが発生するとホストが終了します。 `true` の場合、ホストは起動時に例外をキャプチャして、サーバーを起動しようとします。

**キー**: `captureStartupErrors`  
**型**: `bool` (`true` または `1`)  
**既定**:アプリが IIS の背後で Kestrel を使用して実行されている場合 (既定値は `true`) を除き、既定では `false` に設定されます。  
**環境変数**: `<PREFIX_>CAPTURESTARTUPERRORS`

この値を設定するには、構成を使用するか、または `CaptureStartupErrors` を呼び出します。

```csharp
webBuilder.CaptureStartupErrors(true);
```

### <a name="detailederrors"></a>DetailedErrors

有効にされている場合、または環境が `Development` である場合、アプリによって詳細なエラーがキャプチャされます。

**キー**: `detailedErrors`  
**型**: `bool` (`true` または `1`)  
**既定値**: `false`  
**環境変数**: `<PREFIX_>_DETAILEDERRORS`

この値を設定するには、構成を使用するか、または `UseSetting` を呼び出します。

```csharp
webBuilder.UseSetting(WebHostDefaults.DetailedErrorsKey, "true");
```

### <a name="hostingstartupassemblies"></a>HostingStartupAssemblies

起動時に読み込むホスティング スタートアップ アセンブリのセミコロンで区切られた文字列。 構成値は既定で空の文字列に設定されますが、ホスティング スタートアップ アセンブリには常にアプリのアセンブリが含まれます。 ホスティング スタートアップ アセンブリが提供されている場合、アプリが起動中に共通サービスをビルドしたときに読み込むためにアプリのアセンブリに追加されます。

**キー**: `hostingStartupAssemblies`  
**型**: `string`  
**既定**:空の文字列  
**環境変数**: `<PREFIX_>_HOSTINGSTARTUPASSEMBLIES`

この値を設定するには、構成を使用するか、または `UseSetting` を呼び出します。

```csharp
webBuilder.UseSetting(WebHostDefaults.HostingStartupAssembliesKey, "assembly1;assembly2");
```

### <a name="hostingstartupexcludeassemblies"></a>HostingStartupExcludeAssemblies

起動時に除外するホスティング スタートアップ アセンブリのセミコロン区切り文字列。

**キー**: `hostingStartupExcludeAssemblies`  
**型**: `string`  
**既定**:空の文字列  
**環境変数**: `<PREFIX_>_HOSTINGSTARTUPEXCLUDEASSEMBLIES`

この値を設定するには、構成を使用するか、または `UseSetting` を呼び出します。

```csharp
webBuilder.UseSetting(WebHostDefaults.HostingStartupExcludeAssembliesKey, "assembly1;assembly2");
```

### <a name="https_port"></a>HTTPS_Port

HTTPS リダイレクト ポート。 [HTTPS の適用](xref:security/enforcing-ssl)に使用されます。

**キー**: `https_port`  
**型**: `string`  
**既定**:既定値は設定されていません。  
**環境変数**: `<PREFIX_>HTTPS_PORT`

この値を設定するには、構成を使用するか、または `UseSetting` を呼び出します。

```csharp
webBuilder.UseSetting("https_port", "8080");
```

### <a name="preferhostingurls"></a>PreferHostingUrls

`IServer` の実装で構成されている URL ではなく、`IWebHostBuilder` で構成されている URL でホストがリッスンするかどうかを示します。

**キー**: `preferHostingUrls`  
**型**: `bool` (`true` または `1`)  
**既定値**: `true`  
**環境変数**: `<PREFIX_>_PREFERHOSTINGURLS`

この値を設定するには、環境変数を使用するか、または `PreferHostingUrls` を呼び出します。

```csharp
webBuilder.PreferHostingUrls(false);
```

### <a name="preventhostingstartup"></a>PreventHostingStartup

アプリのアセンブリで構成されているホスティング スタートアップ アセンブリを含む、ホスティング スタートアップ アセンブリの自動読み込みを回避します。 詳細については、「<xref:fundamentals/configuration/platform-specific-configuration>」を参照してください。

**キー**: `preventHostingStartup`  
**型**: `bool` (`true` または `1`)  
**既定値**: `false`  
**環境変数**: `<PREFIX_>_PREVENTHOSTINGSTARTUP`

この値を設定するには、環境変数を使用するか、または `UseSetting` を呼び出します。

```csharp
webBuilder.UseSetting(WebHostDefaults.PreventHostingStartupKey, "true");
```

### <a name="startupassembly"></a>StartupAssembly

`Startup` クラスを検索するアセンブリ。

**キー**: `startupAssembly`  
**型**: `string`  
**既定**:アプリのアセンブリ  
**環境変数**: `<PREFIX_>STARTUPASSEMBLY`

この値を設定するには、環境変数を使用するか、または `UseStartup` を呼び出します。 `UseStartup` は、アセンブリ名 (`string`) または型 (`TStartup`) を取ることができます。 複数の `UseStartup` メソッドが呼び出された場合は、最後のメソッドが優先されます。

```csharp
webBuilder.UseStartup("StartupAssemblyName");
```

```csharp
webBuilder.UseStartup<Startup>();
```

### <a name="urls"></a>URL

サーバーが要求をリッスンする必要があるポートとプロトコルを含む IP アドレスまたはホスト アドレスを示すセミコロンで区切られたリスト。 たとえば、`http://localhost:123` のようにします。 "\*" を使用し、サーバーが指定されたポートとプロトコル (`http://*:5000` など) を使用して IP アドレスまたはホスト名に関する要求をリッスンする必要があることを示します。 プロトコル (`http://` または `https://`) は各 URL に含める必要があります。 サポートされている形式はサーバー間で異なります。

**キー**: `urls`  
**型**: `string`  
**既定値**: `http://localhost:5000` および `https://localhost:5001`  
**環境変数**: `<PREFIX_>URLS`

この値を設定するには、環境変数を使用するか、または `UseUrls` を呼び出します。

```csharp
webBuilder.UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002");
```

Kestrel には独自のエンドポイント構成 API があります。 詳細については、「<xref:fundamentals/servers/kestrel#endpoint-configuration>」を参照してください。

### <a name="webroot"></a>WebRoot

[IWebHostEnvironment.WebRootPath](xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment.WebRootPath) プロパティでは、アプリの静的アセットへの相対パスが決定されます。 パスが存在しない場合は、no-op ファイル プロバイダーが使用されます。  

**キー**: `webroot`  
**型**: `string`  
**既定**:既定値は、`wwwroot` です。 *{content root}/wwwroot* へのパスが存在する必要があります。  
**環境変数**: `<PREFIX_>WEBROOT`

この値を設定するには、環境変数を使用するか、または `IWebHostBuilder` 上で `UseWebRoot` を呼び出します。

```csharp
webBuilder.UseWebRoot("public");
```

詳細については次を参照してください:

* [基礎: Web ルート](xref:fundamentals/index#web-root)
* [ContentRoot](#contentroot)

## <a name="manage-the-host-lifetime"></a>ホストの有効期間を管理する

組み込みの <xref:Microsoft.Extensions.Hosting.IHost> 実装でメソッドを呼び出して、アプリの開始および停止を行います。 これらのメソッドは、サービス コンテナーに登録されているすべての <xref:Microsoft.Extensions.Hosting.IHostedService> 実装に影響を与えます。

### <a name="run"></a>実行

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Run*> はアプリを実行し、ホストがシャットダウンされるまで呼び出し元のスレッドをブロックします。

### <a name="runasync"></a>RunAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync*> はアプリを実行し、キャンセル トークンまたはシャットダウンがトリガーされると完了する <xref:System.Threading.Tasks.Task> を返します。

### <a name="runconsoleasync"></a>RunConsoleAsync

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.RunConsoleAsync*> は、コンソールのサポートを有効にし、ホストをビルドして開始した後、<kbd>Ctrl</kbd> + <kbd>C</kbd>/SIGINT または SIGTERM がシャットダウンするのを待機します。

### <a name="start"></a>[開始]

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Start*> は、ホストを同期的に開始します。

### <a name="startasync"></a>StartAsync

<xref:Microsoft.Extensions.Hosting.IHost.StartAsync*> ではホストが開始され、キャンセル トークンまたはシャットダウンがトリガーされると完了する <xref:System.Threading.Tasks.Task> が返されます。 

<xref:Microsoft.Extensions.Hosting.IHostLifetime.WaitForStartAsync*> は `StartAsync` の開始時に呼び出され、これが完了するまで待機してから続行します。 これを使って、外部イベントによって通知されるまで開始を遅らせることができます。

### <a name="stopasync"></a>StopAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.StopAsync*> は、指定されたタイムアウト内でホストの停止を試みます。

### <a name="waitforshutdown"></a>WaitForShutdown

<kbd>Ctrl</kbd> + <kbd>C</kbd>/SIGINT や SIGTERM を介するなどして IHostLifetime によってシャットダウンがトリガーされるまで、<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*> では呼び出し側スレッドがブロックされます。

### <a name="waitforshutdownasync"></a>WaitForShutdownAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdownAsync*> が返す <xref:System.Threading.Tasks.Task> は、提供されたトークンによってシャットダウンがトリガーされると完了し、<xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> を呼び出します。

### <a name="external-control"></a>外部コントロール

ホストの有効期間の直接コントロールは、外部から呼び出すことができるメソッドを使って実現できます。

```csharp
public class Program
{
    private IHost _host;

    public Program()
    {
        _host = new HostBuilder()
            .Build();
    }

    public async Task StartAsync()
    {
        _host.StartAsync();
    }

    public async Task StopAsync()
    {
        using (_host)
        {
            await _host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core アプリはホストを構成して起動します。 ホストはアプリの起動と有効期間の管理を担当します。

この記事では、HTTP 要求を処理しないアプリに使用される、ASP.NET Core の汎用ホスト (<xref:Microsoft.Extensions.Hosting.HostBuilder>) について説明します。

汎用ホストの目的は、Web ホスト API から HTTP パイプラインを切り離して、多様なホスト シナリオを有効にすることです。 メッセージング、バックグラウンド タスク、汎用ホストに基づくその他の HTTP ワークロードに対して、構成、依存関係の挿入 (DI)、ログなどの横断的機能によるメリットがあります。

汎用ホストは ASP.NET Core 2.1 の新機能であり、Web ホスティングのシナリオには適していません。 Web ホスティングのシナリオの場合は、[Web ホスト](xref:fundamentals/host/web-host)を使ってください。 汎用ホストは将来のリリースで Web ホストに代わるものであり、HTTP と非 HTTP 両方のシナリオのプライマリ ホスト API として機能するようになります。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/generic-host/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

[Visual Studio Code](https://code.visualstudio.com/) でサンプル アプリを実行するときは、"*外部ターミナルまたは統合ターミナル*" を使います。 `internalConsole` ではサンプルを実行しないでください。

Visual Studio Code でコンソールを設定するには:

1. *.vscode/launch.json* ファイルを開きます。
1. **.NET Core Launch (console)** の構成で、**console** エントリを探します。 値を `externalTerminal` または `integratedTerminal` に設定します。

## <a name="introduction"></a>はじめに

汎用ホスト ライブラリは、<xref:Microsoft.Extensions.Hosting> 名前空間で使用でき、[Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting/) パッケージで提供されます。 `Microsoft.Extensions.Hosting` パッケージは、[Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)に含まれます (ASP.NET Core 2.1 以降)。

<xref:Microsoft.Extensions.Hosting.IHostedService> がコード実行のエントリ ポイントです。 <xref:Microsoft.Extensions.Hosting.IHostedService> の各実装は、[ConfigureServices でのサービス登録](#configureservices)の順序で実行されます。 ホストが開始するときは <xref:Microsoft.Extensions.Hosting.IHostedService> ごとに <xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync*> が呼び出され、ホストが正常に終了するときは登録と逆の順序で <xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*> が呼び出されます。

## <a name="set-up-a-host"></a>ホストを設定する

<xref:Microsoft.Extensions.Hosting.IHostBuilder> は、ライブラリとアプリがホストの初期化、ビルド、実行に使用するメイン コンポーネントです。

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_HostBuilder)]

## <a name="options"></a>オプション

<xref:Microsoft.Extensions.Hosting.IHost> の <xref:Microsoft.Extensions.Hosting.HostOptions> 構成オプションです。

### <a name="shutdown-timeout"></a>シャットダウン タイムアウト

<xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*> によって <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> のタイムアウトが設定されます。 既定値は 5 秒です。

次に示す `Program.Main` のオプション構成では、既定の 5 秒のシャットダウン タイムアウトが 20 秒に延長されます。

```csharp
var host = new HostBuilder()
    .ConfigureServices((hostContext, services) =>
    {
        services.Configure<HostOptions>(option =>
        {
            option.ShutdownTimeout = System.TimeSpan.FromSeconds(20);
        });
    })
    .Build();
```

## <a name="default-services"></a>既定のサービス

次のサービスは、ホストの初期化中に登録されます。

* [環境](xref:fundamentals/environments) (<xref:Microsoft.Extensions.Hosting.IHostingEnvironment>)
* <xref:Microsoft.Extensions.Hosting.HostBuilderContext>
* [構成](xref:fundamentals/configuration/index) (<xref:Microsoft.Extensions.Configuration.IConfiguration>)
* <xref:Microsoft.Extensions.Hosting.IApplicationLifetime> (`Microsoft.Extensions.Hosting.Internal.ApplicationLifetime`)
* <xref:Microsoft.Extensions.Hosting.IHostLifetime> (`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime`)
* <xref:Microsoft.Extensions.Hosting.IHost>
* [オプション](xref:fundamentals/configuration/options) (<xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.AddOptions*>)
* [ログ](xref:fundamentals/logging/index) (<xref:Microsoft.Extensions.DependencyInjection.LoggingServiceCollectionExtensions.AddLogging*>)

## <a name="host-configuration"></a>ホストの構成

ホストの構成は、次のようにして作成されます。

* <xref:Microsoft.Extensions.Hosting.IHostBuilder> で拡張メソッドを呼び出して、[コンテンツ ルート](#content-root)と[環境](#environment)を設定する。
* <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> の構成プロバイダーから構成を読み取る。

### <a name="extension-methods"></a>拡張メソッド

### <a name="application-key-name"></a>アプリケーション キー (名前)

[IHostingEnvironment.ApplicationName](xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ApplicationName*) プロパティは、ホストの構築時にホストの構成から設定されます。 値を明示的に設定するには、[HostDefaults.ApplicationKey](xref:Microsoft.Extensions.Hosting.HostDefaults.ApplicationKey) を使用します。

**キー**: `applicationName`  
**型**: `string`  
**既定**:アプリのエントリ ポイントを含むアセンブリの名前。  
**次を使用して設定**: `HostBuilderContext.HostingEnvironment.ApplicationName`  
**環境変数**: `<PREFIX_>APPLICATIONNAME` (`<PREFIX_>` は [オプションであり、ユーザー定義です](#configurehostconfiguration))

### <a name="content-root"></a>コンテンツ ルート

この設定では、ホストがコンテンツ ファイルの検索を開始する場所を決定します。

**キー**: `contentRoot`  
**型**: `string`  
**既定**:既定でアプリ アセンブリが存在するフォルダーに設定されます。  
**次を使用して設定**: `UseContentRoot`  
**環境変数**: `<PREFIX_>CONTENTROOT` (`<PREFIX_>` は [オプションであり、ユーザー定義です](#configurehostconfiguration))

パスが存在しない場合は、ホストを起動できません。

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_UseContentRoot)]

詳細については、[基礎: コンテンツ ルート](xref:fundamentals/index#content-root)に関する記事を参照してください。

### <a name="environment"></a>環境

アプリの[環境](xref:fundamentals/environments)を設定します。

**キー**: `environment`  
**型**: `string`  
**既定値**: `Production`  
**次を使用して設定**: `UseEnvironment`  
**環境変数**: `<PREFIX_>ENVIRONMENT` (`<PREFIX_>` は [オプションであり、ユーザー定義です](#configurehostconfiguration))

環境は任意の値に設定することができます。 フレームワークで定義された値には `Development`、`Staging`、`Production` が含まれます。 値は大文字と小文字が区別されません。

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_UseEnvironment)]

### <a name="configurehostconfiguration"></a>ConfigureHostConfiguration

<xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> は <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> を使用して、ホストの <xref:Microsoft.Extensions.Configuration.IConfiguration> を作成します。 ホストの構成は、<xref:Microsoft.Extensions.Hosting.IHostingEnvironment> をアプリのビルド プロセスで使用するための初期化に使用されます。

<xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> を複数回呼び出して結果を追加できます。 ホストは、指定されたキーで最後に値を設定したオプションを使用します。

既定ではプロバイダーが含まれていません。 次のような、アプリが <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> で必要とする構成プロバイダーを明示的に指定する必要があります。

* ファイルの構成 (*hostsettings.json* ファイルからなど)。
* 環境変数の構成。
* コマンドライン引数の構成。
* その他に必要なすべての構成プロバイダー。

ホストのファイル構成は、いずれかの[ファイル構成プロバイダー](xref:fundamentals/configuration/index#file-configuration-provider)に対する呼び出しの前に `SetBasePath` のアプリのベース パスを指定することで有効になります。 サンプル アプリは、JSON ファイル (*hostsettings.json*) を使用し、<xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*> を呼び出して、ファイルのホスト構成設定を使用します。

ホストの[環境変数の構成](xref:fundamentals/configuration/index#environment-variables-configuration-provider)を追加するには、ホスト ビルダーで <xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*> を呼び出します。 `AddEnvironmentVariables` は、オプションのユーザー定義プレフィックスを受け入れます。 サンプル アプリは、`PREFIX_` のプレフィックスを使用します。 環境変数が読み取られると、プレフィックスは削除されます。 サンプル アプリのホストが構成されると、`PREFIX_ENVIRONMENT` の環境変数の値が `environment` キーのホスト構成値になります。

開発中に [Visual Studio](https://visualstudio.microsoft.com) を使用している、または `dotnet run` を使用してアプリを実行している場合は、環境変数を *Properties/launchSettings.json* ファイルに設定することができます。 [Visual Studio Code](https://code.visualstudio.com/) では、開発中に環境変数を *.vscode/launch.json* ファイルに設定することができます。 詳細については、「<xref:fundamentals/environments>」を参照してください。

[コマンドラインの構成](xref:fundamentals/configuration/index#command-line-configuration-provider)は、<xref:Microsoft.Extensions.Configuration.CommandLineConfigurationExtensions.AddCommandLine*>を呼び出すことで追加されます。 コマンドラインの構成は、コマンドライン引数を許可して以前の構成プロバイダーから提供された構成をオーバーライドするために最後に追加されます。

*hostsettings.json*:

[!code-json[](generic-host/samples/2.x/GenericHostSample/hostsettings.json)]

追加の構成は、[applicationName](#application-key-name) と [contentRoot](#content-root) キーで指定することができます。

<xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> を使う `HostBuilder` の構成の例:

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureHostConfiguration)]

## <a name="configureappconfiguration"></a>ConfigureAppConfiguration

アプリの構成は、<xref:Microsoft.Extensions.Hosting.IHostBuilder> の実装で <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> を呼び出すことで作成されます。 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> は <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> を使用して、アプリの <xref:Microsoft.Extensions.Configuration.IConfiguration> を作成します。 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> を複数回呼び出して結果を追加できます。 アプリは、指定されたキーで最後に値を設定したオプションを使用します。 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> によって作成された構成は、以降の操作に対する [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration*) において、および <xref:Microsoft.Extensions.Hosting.IHost.Services*>サービス内で使用できます。

アプリの構成は、[ConfigureHostConfiguration](#configurehostconfiguration) によって提供されたホストの構成を自動的に受信します。

<xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> を使うアプリ構成の例:

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureAppConfiguration)]

*appsettings.json*:

[!code-json[](generic-host/samples/2.x/GenericHostSample/appsettings.json)]

*appsettings.Development.json*:

[!code-json[](generic-host/samples/2.x/GenericHostSample/appsettings.Development.json)]

*appsettings.Production.json*:

[!code-json[](generic-host/samples/2.x/GenericHostSample/appsettings.Production.json)]

設定ファイルを出力ディレクトリに移動するには、プロジェクト ファイルの設定ファイルを [MSBuild プロジェクト項目](/visualstudio/msbuild/common-msbuild-project-items) として指定します。 サンプル アプリはその JSON アプリ設定ファイルと、次の `<Content>` 項目を含む *hostsettings.json* を移動します。

```xml
<ItemGroup>
  <Content Include="**\*.json" Exclude="bin\**\*;obj\**\*" 
      CopyToOutputDirectory="PreserveNewest" />
</ItemGroup>
```

> [!NOTE]
> <xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*> や <xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*> などの構成拡張メソッドでは、[Microsoft.Extensions.Configuration.Json](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json) や [Microsoft.Extensions.Configuration.EnvironmentVariables](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.EnvironmentVariables) など、追加の NuGet パッケージを必要とします。 アプリが [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) を使用しない限り、中心となる [Microsoft.Extensions.Configuration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration) パッケージに加えて、これらのパッケージがプロジェクトに追加されます。 詳細については、「<xref:fundamentals/configuration/index>」を参照してください。

## <a name="configureservices"></a>ConfigureServices

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureServices*> は、アプリの[依存関係の挿入](xref:fundamentals/dependency-injection)コンテナーにサービスを追加します。 <xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureServices*> を複数回呼び出して結果を追加できます。

ホストされるサービスは、<xref:Microsoft.Extensions.Hosting.IHostedService> インターフェイスを実装するバックグラウンド タスク ロジックを持つクラスです。 詳細については、「<xref:fundamentals/host/hosted-services>」を参照してください。

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/generic-host/samples/)では、`AddHostedService` 拡張メソッドを使って、有効期間イベント `LifetimeEventsHostedService` と時刻指定付きバックグラウンド タスク `TimedHostedService` のサービスをアプリに追加します。

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureServices)]

## <a name="configurelogging"></a>ConfigureLogging

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureLogging*> は、指定された <xref:Microsoft.Extensions.Logging.ILoggingBuilder> を構成するためのデリゲートを追加します。 <xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureLogging*> を複数回呼び出して結果を追加できます。

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureLogging)]

### <a name="useconsolelifetime"></a>UseConsoleLifetime

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.UseConsoleLifetime*> では、<kbd>Ctrl</kbd> + <kbd>C</kbd>/SIGINT または SIGTERM をリッスンし、<xref:Microsoft.Extensions.Hosting.IApplicationLifetime.StopApplication*> を呼び出して、シャットダウン プロセスを開始します。 <xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.UseConsoleLifetime*> は、[RunAsync](#runasync) や [WaitForShutdownAsync](#waitforshutdownasync) などの拡張機能のブロックを解除します。 `Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` は、既定の有効期間の実装として事前に登録されています。 最後に登録された有効期間が使用されます。

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_UseConsoleLifetime)]

## <a name="container-configuration"></a>コンテナーの構成

他のコンテナーの接続をサポートするため、ホストは <xref:Microsoft.Extensions.DependencyInjection.IServiceProviderFactory%601> を受け入れることができます。 ファクトリの提供は、DI コンテナーの登録の一部ではなく、具象 DI コンテナーの作成に使用されるホストの組み込みです。 [UseServiceProviderFactory(IServiceProviderFactory&lt;TContainerBuilder&gt;)](xref:Microsoft.Extensions.Hosting.HostBuilder.UseServiceProviderFactory*) は、アプリのサービス プロバイダーの作成に使われる既定のファクトリをオーバーライドします。

カスタム コンテナーの構成は、<xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureContainer*> メソッドによって管理されます。 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureContainer*> は、基盤のホスト API を基にしてコンテナーを構成するための、厳密に型指定されたエクスペリエンスを提供します。 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureContainer*> を複数回呼び出して結果を追加できます。

アプリのサービス コンテナーを作成します。

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/ServiceContainer.cs)]

サービス コンテナーのファクトリを提供します。

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/ServiceContainerFactory.cs)]

ファクトリを使って、アプリのカスタム サービス コンテナーを構成します。

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ContainerConfiguration)]

## <a name="extensibility"></a>機能拡張

ホスト拡張機能は、<xref:Microsoft.Extensions.Hosting.IHostBuilder> 上の拡張メソッドによって実行されます。 次の例では、拡張メソッドが <xref:fundamentals/host/hosted-services> で示される [TimedHostedService](xref:fundamentals/host/hosted-services#timed-background-tasks) の例を使って <xref:Microsoft.Extensions.Hosting.IHostBuilder> の実装を拡張する方法を示しています。

```csharp
var host = new HostBuilder()
    .UseHostedService<TimedHostedService>()
    .Build();

await host.StartAsync();
```

アプリは `UseHostedService` 拡張メソッドを確率して `T` に渡されたホステッド サービスを登録します。

```csharp
using System;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

public static class Extensions
{
    public static IHostBuilder UseHostedService<T>(this IHostBuilder hostBuilder)
        where T : class, IHostedService, IDisposable
    {
        return hostBuilder.ConfigureServices(services =>
            services.AddHostedService<T>());
    }
}
```

## <a name="manage-the-host"></a>ホストを管理する

<xref:Microsoft.Extensions.Hosting.IHost> の実装は、サービス コンテナーに登録されている <xref:Microsoft.Extensions.Hosting.IHostedService> の実装の開始と停止を行います。

### <a name="run"></a>実行

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Run*> はアプリを実行し、ホストがシャットダウンされるまで呼び出し元のスレッドをブロックします。

```csharp
public class Program
{
    public void Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        host.Run();
    }
}
```

### <a name="runasync"></a>RunAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync*> はアプリを実行し、キャンセル トークンまたはシャットダウンがトリガーされると完了する <xref:System.Threading.Tasks.Task> を返します。

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        await host.RunAsync();
    }
}
```

### <a name="runconsoleasync"></a>RunConsoleAsync

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.RunConsoleAsync*> は、コンソールのサポートを有効にし、ホストをビルドして開始した後、<kbd>Ctrl</kbd> + <kbd>C</kbd>/SIGINT または SIGTERM がシャットダウンするのを待機します。

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var hostBuilder = new HostBuilder();

        await hostBuilder.RunConsoleAsync();
    }
}
```

### <a name="start-and-stopasync"></a>Start と StopAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Start*> は、ホストを同期的に開始します。

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.StopAsync*> は、指定されたタイムアウト内でホストの停止を試みます。

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            host.Start();

            await host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

### <a name="startasync-and-stopasync"></a>StartAsync と StopAsync

<xref:Microsoft.Extensions.Hosting.IHost.StartAsync*> がアプリを起動します。

<xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> がアプリを停止します。

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            await host.StartAsync();

            await host.StopAsync();
        }
    }
}
```

### <a name="waitforshutdown"></a>WaitForShutdown

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*> は、`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` (<kbd>Ctrl</kbd> + <kbd>C</kbd>/SIGINT または SIGTERM をリッスンする) などの <xref:Microsoft.Extensions.Hosting.IHostLifetime> を介してトリガーされます。 <xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*> は <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> を呼び出します。

```csharp
public class Program
{
    public void Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            host.Start();

            host.WaitForShutdown();
        }
    }
}
```

### <a name="waitforshutdownasync"></a>WaitForShutdownAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdownAsync*> が返す <xref:System.Threading.Tasks.Task> は、提供されたトークンによってシャットダウンがトリガーされると完了し、<xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> を呼び出します。

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            await host.StartAsync();

            await host.WaitForShutdownAsync();
        }

    }
}
```

### <a name="external-control"></a>外部コントロール

ホストの外部コントロールは、外部から呼び出すことができるメソッドを使って実現できます。

```csharp
public class Program
{
    private IHost _host;

    public Program()
    {
        _host = new HostBuilder()
            .Build();
    }

    public async Task StartAsync()
    {
        _host.StartAsync();
    }

    public async Task StopAsync()
    {
        using (_host)
        {
            await _host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

<xref:Microsoft.Extensions.Hosting.IHostLifetime.WaitForStartAsync*> は <xref:Microsoft.Extensions.Hosting.IHost.StartAsync*> の開始時に呼び出され、これが完了するまで待機してから続行します。 これを使って、外部イベントによって通知されるまで開始を遅らせることができます。

## <a name="ihostingenvironment-interface"></a>IHostingEnvironment インターフェイス

<xref:Microsoft.Extensions.Hosting.IHostingEnvironment> は、アプリのホスティング環境に関する情報を提供します。 プロパティと拡張メソッドを使用するには、[コンストラクターの挿入](xref:fundamentals/dependency-injection)を使用して <xref:Microsoft.Extensions.Hosting.IHostingEnvironment> を取得します。

```csharp
public class MyClass
{
    private readonly IHostingEnvironment _env;

    public MyClass(IHostingEnvironment env)
    {
        _env = env;
    }

    public void DoSomething()
    {
        var environmentName = _env.EnvironmentName;
    }
}
```

詳細については、「<xref:fundamentals/environments>」を参照してください。

## <a name="iapplicationlifetime-interface"></a>IApplicationLifetime インターフェイス

<xref:Microsoft.Extensions.Hosting.IApplicationLifetime> は、正常なシャットダウンの要求など、起動後とシャットダウンのアクティビティに対応します。 インターフェイスの 3 つのプロパティは、起動とシャットダウン イベントを定義する <xref:System.Action> メソッドを登録するために使用されるキャンセル トークンです。

| キャンセル トークン | トリガーのタイミング |
| ------------------ | --------------------- |
| <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.ApplicationStarted*> | ホストが完全に起動されたとき。 |
| <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.ApplicationStopped*> | ホストが正常なシャットダウンを完了しているとき。 すべての要求を処理する必要があります。 このイベントが完了するまで、シャットダウンはブロックされます。 |
| <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.ApplicationStopping*> | ホストが正常なシャットダウンを行っているとき。 要求がまだ処理されている可能性がある場合。 このイベントが完了するまで、シャットダウンはブロックされます。 |

コンストラクターは任意のクラスに <xref:Microsoft.Extensions.Hosting.IApplicationLifetime> サービスを挿入します。 [サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/generic-host/samples/)は、`LifetimeEventsHostedService` クラス (<xref:Microsoft.Extensions.Hosting.IHostedService> の実装) へのコンストラクターの挿入を使って、イベントを登録します。

*LifetimeEventsHostedService.cs*:

[!code-csharp[](generic-host/samples/2.x/GenericHostSample/LifetimeEventsHostedService.cs?name=snippet1)]

<xref:Microsoft.Extensions.Hosting.IApplicationLifetime.StopApplication*> は、アプリの終了を要求します。 以下のクラスは、クラスの `Shutdown` メソッドが呼び出されると、<xref:Microsoft.Extensions.Hosting.IApplicationLifetime.StopApplication*> を使ってアプリを正常にシャットダウンします。

```csharp
public class MyClass
{
    private readonly IApplicationLifetime _appLifetime;

    public MyClass(IApplicationLifetime appLifetime)
    {
        _appLifetime = appLifetime;
    }

    public void Shutdown()
    {
        _appLifetime.StopApplication();
    }
}
```

::: moniker-end

## <a name="additional-resources"></a>その他の技術情報

* <xref:fundamentals/host/hosted-services>
