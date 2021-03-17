---
title: ASP.NET Core の Web ホスト
author: rick-anderson
description: ASP.NET Core アプリの Web ホスト (アプリの起動と有効期間の管理を担当する) について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
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
uid: fundamentals/host/web-host
ms.openlocfilehash: fa9b1941d6dcda30855a4729dfa1cd78f897d9b6
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102109976"
---
# <a name="aspnet-core-web-host"></a>ASP.NET Core の Web ホスト

ASP.NET Core アプリは *ホスト* を構成して起動します。 ホストはアプリの起動と有効期間の管理を担当します。 少なくとも、ホストはサーバーおよび要求処理パイプラインを構成します。 ホストでは、ログ記録、依存関係挿入、構成を設定することもできます。

::: moniker range=">= aspnetcore-3.0"

この記事では、Web ホストについて説明します。これは、下位互換性のためだけに引き続き使用可能となっています。 ASP.NET Core テンプレートを使用すると、すべての種類のアプリに推奨されている [.NET での汎用ホスト](<xref:fundamentals/host/generic-host>)が作成されます。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

この記事では Web ホストについて説明します。これは Web アプリをホストするためのものです。 その他の種類のアプリでは、[汎用ホスト](xref:fundamentals/host/generic-host)をご使用ください。

::: moniker-end

## <a name="set-up-a-host"></a>ホストを設定する

[IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder) のインスタンスを使用して、ホストを作成します。 通常、これはアプリのエントリ ポイントの `Main` メソッドで実行されます。

プロジェクト テンプレートでは、`Main` は *Program.cs* にあります。 一般的なアプリでは、[CreateDefaultBuilder](/dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder) を呼び出してホストの設定を開始します。

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateWebHostBuilder(args).Build().Run();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>();
}
```

`CreateDefaultBuilder` を呼び出すコードは、`CreateWebHostBuilder` という名前のメソッドに含まれ、ビルダー オブジェクトで `Run` を呼び出す `Main` のコードから分離されています。 [Entity Framework Core ツール](/ef/core/miscellaneous/cli/)を使用する場合は、このような分離が必要です。 ツールでは、アプリを実行することなくホストを構成するために、デザイン時に呼び出すことができる `CreateWebHostBuilder` メソッドの検出が想定されます。 別の方法は、`IDesignTimeDbContextFactory` を実装することです。 詳細については、「[デザイン時 DbContext 作成](/ef/core/miscellaneous/cli/dbcontext-creation)」をご覧ください。

`CreateDefaultBuilder` では次のタスクを実行します。

::: moniker range=">= aspnetcore-5.0"
* アプリのホスティング構成プロバイダーを使用して [Kestrel](xref:fundamentals/servers/kestrel) サーバーを Web サーバーとして構成します。 Kestrel サーバーの既定のオプションについては、<xref:fundamentals/servers/kestrel/options> を参照してください。
::: moniker-end
::: moniker range="< aspnetcore-5.0"
* アプリのホスティング構成プロバイダーを使用して [Kestrel](xref:fundamentals/servers/kestrel) サーバーを Web サーバーとして構成します。 Kestrel サーバーの既定のオプションについては、<xref:fundamentals/servers/kestrel#kestrel-options> を参照してください。
::: moniker-end
* [コンテンツ ルート](xref:fundamentals/index#content-root)を、[Directory.GetCurrentDirectory](/dotnet/api/system.io.directory.getcurrentdirectory) によって返されるパスに設定します。
* 次から[ ホスト構成](#host-configuration-values)を読み込みます。
  * `ASPNETCORE_` のプレフィックスが付いた環境変数 (たとえば、`ASPNETCORE_ENVIRONMENT`)。
  * コマンド ライン引数。
* 次の順序でアプリの構成を読み込みます。
  * *appsettings.json*.
  * *appsettings.{Environment}.json*。
  * エントリ アセンブリを使用して `Development` 環境でアプリが実行される場合に使用される[ユーザー シークレット](xref:security/app-secrets)。
  * 環境変数。
  * コマンド ライン引数。
* コンソールとデバッグ出力の[ログ](xref:fundamentals/logging/index)を構成します。 ログには、 *appsettings.json* または *appsettings.{Environment}.json* ファイルのログ構成セクションで指定される [ログ フィルター](xref:fundamentals/logging/index#log-filtering)規則が含まれます。
* [ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)を使用して IIS の背後で実行されている場合は、`CreateDefaultBuilder` によってアプリのベース アドレスとポートが構成される [IIS の統合](xref:host-and-deploy/iis/index)が有効になります。 IIS の統合により、[スタートアップ エラーをキャプチャする](#capture-startup-errors)アプリも構成されます。 IIS の既定のオプションについては、<xref:host-and-deploy/iis/index#iis-options> を参照してください。
* アプリの環境が開発の場合、[ServiceProviderOptions.ValidateScopes](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions.validatescopes) を `true` に設定します。 詳しくは、「[スコープの検証](#scope-validation)」をご覧ください。

`CreateDefaultBuilder` によって定義された構成は、[ConfigureAppConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configureappconfiguration)、[ConfigureLogging](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configurelogging)、そして [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder) のその他のメソッドと拡張メソッドによってオーバーライドされ、拡張されます。 以下に、いくつかの例を示します。

* [ConfigureAppConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configureappconfiguration) はアプリの追加の `IConfiguration` を指定するために使用します。 次の `ConfigureAppConfiguration` の呼び出しによりデリゲートが追加され、*appsettings.xml* ファイルにアプリの構成が含まれます。 `ConfigureAppConfiguration` は複数回呼び出すことができます。 この構成はホスト (たとえば、サーバーの URL や環境など) には適用されないことに注意してください。 「[ホストの構成値](#host-configuration-values)」のセクションを参照してください。

    ```csharp
    WebHost.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration((hostingContext, config) =>
        {
            config.AddXmlFile("appsettings.xml", optional: true, reloadOnChange: true);
        })
        ...
    ```

* 次の `ConfigureLogging` の呼び出しによりデリゲートが追加され、最小ログ記録レベル ([SetMinimumLevel](/dotnet/api/microsoft.extensions.logging.loggingbuilderextensions.setminimumlevel)) が [LogLevel.Warning](/dotnet/api/microsoft.extensions.logging.loglevel) に構成されます。 この設定により、`CreateDefaultBuilder` で構成された *appsettings.Development.json* (`LogLevel.Debug`) および *appsettings.Production.json* (`LogLevel.Error`) の設定がオーバーライドされます。 `ConfigureLogging` は複数回呼び出すことができます。

    ```csharp
    WebHost.CreateDefaultBuilder(args)
        .ConfigureLogging(logging => 
        {
            logging.SetMinimumLevel(LogLevel.Warning);
        })
        ...
    ```

::: moniker range=">= aspnetcore-2.2"

* 次の `ConfigureKestrel` の呼び出しにより、Kestrel が `CreateDefaultBuilder` によって構成されたときに確立された既定の [Limits.MaxRequestBodySize](/dotnet/api/microsoft.aspnetcore.server.kestrel.core.kestrelserverlimits.maxrequestbodysize) サイズである 30,000,000 バイトがオーバーライドされます。

    ```csharp
    WebHost.CreateDefaultBuilder(args)
        .ConfigureKestrel((context, options) =>
        {
            options.Limits.MaxRequestBodySize = 20000000;
        });
    ```

::: moniker-end

::: moniker range="< aspnetcore-2.2"

* 次の [UseKestrel](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderkestrelextensions.usekestrel) の呼び出しにより、Kestrel が `CreateDefaultBuilder` によって構成されたときに確立された既定の [Limits.MaxRequestBodySize](/dotnet/api/microsoft.aspnetcore.server.kestrel.core.kestrelserverlimits.maxrequestbodysize) サイズである 30,000,000 バイトがオーバーライドされます。

    ```csharp
    WebHost.CreateDefaultBuilder(args)
        .UseKestrel(options =>
        {
            options.Limits.MaxRequestBodySize = 20000000;
        });
    ```

::: moniker-end

[コンテンツ ルート](xref:fundamentals/index#content-root)で、ホストが MVC ビュー ファイルなどのコンテンツ ファイルを検索する場所を決定します。 プロジェクトのルート フォルダーからアプリが開始された場合は、そのプロジェクトのルート フォルダーがコンテンツ ルートとして使用されます。 これは、[Visual Studio](https://visualstudio.microsoft.com) と [dotnet の新しいテンプレート](/dotnet/core/tools/dotnet-new)で使用される既定です。

アプリの構成の詳細については、<xref:fundamentals/configuration/index> を参照してください。

> [!NOTE]
> 静的な `CreateDefaultBuilder` メソッドを使用する代わりに、[WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder) からホストを作成する方法が ASP.NET Core 2.x でサポートされています。

ホストの構成時に、[Configure](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configure) および [ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder.configureservices) メソッドを指摘することができます。 `Startup` クラスが指定されている場合は、`Configure` メソッドを定義する必要があります。 詳細については、「<xref:fundamentals/startup>」を参照してください。 `ConfigureServices` の複数回の呼び出しでは、互いに追加されます。 `WebHostBuilder` での `Configure` または `UseStartup` の複数回の呼び出しでは、以前の設定が置き換えられます。

## <a name="host-configuration-values"></a>ホストの構成値

[WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder) では、ホストの構成値を設定する場合に以下の方法に依存します。

* `ASPNETCORE_{configurationKey}` 形式の環境変数を含む、ホスト ビルダー構成。 たとえば、`ASPNETCORE_ENVIRONMENT` のようにします。
* [UseContentRoot](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.usecontentroot) や [UseConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useconfiguration) などの拡張機能 (「[構成をオーバーライドする](#override-configuration)」のセクションを参照)。
* [UseSetting](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder.usesetting) と関連するキー。 `UseSetting` で値を設定すると、値はその型に関係なく、文字列として設定されます。

ホストは、最後に値を設定したオプションを使用します。 詳しくは、次のセクション「[構成をオーバーライドする](#override-configuration)」をご覧ください。

### <a name="application-key-name"></a>アプリケーション キー (名前)

::: moniker range=">= aspnetcore-3.0"

ホストの構築時に [UseStartup](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.usestartup) または [Configure](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configure) を呼び出すと、`IWebHostEnvironment.ApplicationName` プロパティが自動的に設定されます。 この値はアプリのエントリ ポイントを含むアセンブリの名前に設定されます。 値を明示的に設定するには、[WebHostDefaults.ApplicationKey](/dotnet/api/microsoft.aspnetcore.hosting.webhostdefaults.applicationkey) を使用します。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ホストの構築時に [UseStartup](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.usestartup) または [Configure](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configure) が呼び出されると、[IHostingEnvironment.ApplicationName](/dotnet/api/microsoft.extensions.hosting.ihostingenvironment.applicationname) プロパティが自動的に設定されます。 この値はアプリのエントリ ポイントを含むアセンブリの名前に設定されます。 値を明示的に設定するには、[WebHostDefaults.ApplicationKey](/dotnet/api/microsoft.aspnetcore.hosting.webhostdefaults.applicationkey) を使用します。

::: moniker-end

**キー**: applicationName  
**型**: *文字列*  
**既定値**:アプリのエントリ ポイントを含むアセンブリの名前。  
**次を使用して設定**: `UseSetting`  
**環境変数**: `ASPNETCORE_APPLICATIONNAME`

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.ApplicationKey, "CustomApplicationName")
```

### <a name="capture-startup-errors"></a>スタートアップ エラーをキャプチャする

この設定では、スタートアップ エラーのキャプチャを制御します。

**キー**: captureStartupErrors  
**型**: *ブール* (`true` または `1`)  
**既定**:アプリが IIS の背後で Kestrel を使用して実行されている場合 (既定値は `true`) を除き、既定では `false` に設定されます。  
**次を使用して設定**: `CaptureStartupErrors`  
**環境変数**: `ASPNETCORE_CAPTURESTARTUPERRORS`

`false` の場合、起動時にエラーが発生するとホストが終了します。 `true` の場合、ホストは起動時に例外をキャプチャして、サーバーを起動しようとします。

```csharp
WebHost.CreateDefaultBuilder(args)
    .CaptureStartupErrors(true)
```

### <a name="content-root"></a>コンテンツ ルート

この設定により、ASP.NET Core でコンテンツ ファイルの検索が開始される場所が決まります。

**キー**: contentRoot  
**型**: *文字列*  
**既定**:既定でアプリ アセンブリが存在するフォルダーに設定されます。  
**次を使用して設定**: `UseContentRoot`  
**環境変数**: `ASPNETCORE_CONTENTROOT`

コンテンツ ルートは、[Web ルート](xref:fundamentals/index#web-root)の基本パスとしても使用されます。 コンテンツ ルート パスが存在しない場合は、ホストを起動できません。

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseContentRoot("c:\\<content-root>")
```

詳細については次を参照してください:

* [基礎: コンテンツ ルート](xref:fundamentals/index#content-root)
* [Web ルート](#web-root)

### <a name="detailed-errors"></a>詳細なエラー

詳細なエラーをキャプチャするかどうかを判断します。

**キー**: detailedErrors  
**型**: *ブール* (`true` または `1`)  
**既定値**: false  
**次を使用して設定**: `UseSetting`  
**環境変数**: `ASPNETCORE_DETAILEDERRORS`

有効な場合 (または<a href="#environment">環境</a>が `Development` に設定されている場合)、アプリは詳細な例外をキャプチャします。

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.DetailedErrorsKey, "true")
```

### <a name="environment"></a>環境

アプリの環境を設定します。

**キー**: 環境  
**型**: *文字列*  
**既定**:実稼働  
**次を使用して設定**: `UseEnvironment`  
**環境変数**: `ASPNETCORE_ENVIRONMENT`

環境は任意の値に設定することができます。 フレームワークで定義された値には `Development`、`Staging`、`Production` が含まれます。 値は大文字と小文字が区別されません。 既定では、*環境* は `ASPNETCORE_ENVIRONMENT` 環境変数から読み取られます。 [Visual Studio](https://visualstudio.microsoft.com) を使用する場合、環境変数は *launchSettings.json* ファイルで設定できます。 詳細については、「<xref:fundamentals/environments>」を参照してください。

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseEnvironment(EnvironmentName.Development)
```

### <a name="hosting-startup-assemblies"></a>ホスティング スタートアップ アセンブリ

アプリのホスティング スタートアップ アセンブリを設定します。

**キー**: hostingStartupAssemblies  
**型**: *文字列*  
**既定**:空の文字列  
**次を使用して設定**: `UseSetting`  
**環境変数**: `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`

起動時に読み込むホスティング スタートアップ アセンブリのセミコロンで区切られた文字列。

構成値は既定で空の文字列に設定されますが、ホスティング スタートアップ アセンブリには常にアプリのアセンブリが含まれます。 ホスティング スタートアップ アセンブリが提供されている場合、アプリが起動中に共通サービスをビルドしたときに読み込むためにアプリのアセンブリに追加されます。

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.HostingStartupAssembliesKey, "assembly1;assembly2")
```

### <a name="https-port"></a>HTTPS Port

HTTPS リダイレクト ポートを設定します。 [HTTPS の適用](xref:security/enforcing-ssl)に使用されます。

**キー**: https_port **型**: *文字列*
**既定値**:既定値は設定されていません。
**次を使用して設定**:`UseSetting`
**環境変数**: `ASPNETCORE_HTTPS_PORT`

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting("https_port", "8080")
```

### <a name="hosting-startup-exclude-assemblies"></a>除外するホスティング スタートアップ アセンブリ

起動時に除外するホスティング スタートアップ アセンブリのセミコロン区切り文字列。

**キー**: hostingStartupExcludeAssemblies  
**型**: *文字列*  
**既定**:空の文字列  
**次を使用して設定**: `UseSetting`  
**環境変数**: `ASPNETCORE_HOSTINGSTARTUPEXCLUDEASSEMBLIES`

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.HostingStartupExcludeAssembliesKey, "assembly1;assembly2")
```

### <a name="prefer-hosting-urls"></a>ホスティング URL を優先する

`IServer` 実装で構成されているものではなく、`WebHostBuilder` で構成されている URL でホストがリッスンするかどうかを示します。

**キー**: preferHostingUrls  
**型**: *ブール* (`true` または `1`)  
**既定値**: true  
**次を使用して設定**: `PreferHostingUrls`  
**環境変数**: `ASPNETCORE_PREFERHOSTINGURLS`

```csharp
WebHost.CreateDefaultBuilder(args)
    .PreferHostingUrls(false)
```

### <a name="prevent-hosting-startup"></a>ホスティング スタートアップを回避する

アプリのアセンブリで構成されているホスティング スタートアップ アセンブリを含む、ホスティング スタートアップ アセンブリの自動読み込みを回避します。 詳細については、「<xref:fundamentals/configuration/platform-specific-configuration>」を参照してください。

**キー**: preventHostingStartup  
**型**: *ブール* (`true` または `1`)  
**既定値**: false  
**次を使用して設定**: `UseSetting`  
**環境変数**: `ASPNETCORE_PREVENTHOSTINGSTARTUP`

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.PreventHostingStartupKey, "true")
```

### <a name="server-urls"></a>サーバーの URL

サーバーが要求をリッスンする必要があるポートとプロトコルを使用して、IP アドレスまたはホスト アドレスを示します。

**キー**: urls  
**型**: *文字列*  
**既定値**: http://localhost:5000  
**次を使用して設定**: `UseUrls`  
**環境変数**: `ASPNETCORE_URLS`

サーバーが応答する必要がある URL プレフィックスのセミコロン (;) で区切られたリストに設定します。 たとえば、`http://localhost:123` のようにします。 "\*" を使用し、サーバーが指定されたポートとプロトコル (`http://*:5000` など) を使用して IP アドレスまたはホスト名に関する要求をリッスンする必要があることを示します。 プロトコル (`http://` または `https://`) は各 URL に含める必要があります。 サポートされている形式はサーバー間で異なります。

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002")
```

::: moniker range=">= aspnetcore-5.0"
Kestrel には独自のエンドポイント構成 API があります。 詳細については、「<xref:fundamentals/servers/kestrel/endpoints>」を参照してください。
::: moniker-end
::: moniker range="< aspnetcore-5.0"
Kestrel には独自のエンドポイント構成 API があります。 詳細については、「<xref:fundamentals/servers/kestrel#endpoint-configuration>」を参照してください。
::: moniker-end

### <a name="shutdown-timeout"></a>シャットダウン タイムアウト

Web ホストがシャットダウンするまで待機する時間を指定します。

**キー**: shutdownTimeoutSeconds  
**型**: *int*  
**既定**:5  
**次を使用して設定**: `UseShutdownTimeout`  
**環境変数**: `ASPNETCORE_SHUTDOWNTIMEOUTSECONDS`

キーは `UseSetting` で *int* を受け取りますが (`.UseSetting(WebHostDefaults.ShutdownTimeoutKey, "10")` など)、[UseShutdownTimeout](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useshutdowntimeout) 拡張メソッドは [TimeSpan](/dotnet/api/system.timespan) を受け取ります。

タイムアウト期間中、ホスティングは次のことを行います。

* [IApplicationLifetime.ApplicationStopping](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime.applicationstopping) をトリガーします。
* ホステッド サービスの停止を試み、停止に失敗したサービスのエラーをログに記録します。

すべてのホステッド サービスが停止する前にタイムアウト時間が切れた場合、残っているアクティブなサービスはアプリのシャットダウン時に停止します。 処理が完了していない場合でも、サービスは停止します。 サービスが停止するまでにさらに時間が必要な場合は、タイムアウト値を増やします。

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseShutdownTimeout(TimeSpan.FromSeconds(10))
```

### <a name="startup-assembly"></a>スタートアップ アセンブリ

`Startup` クラスを検索するアセンブリを決定します。

**キー**: startupAssembly  
**型**: *文字列*  
**既定**:アプリのアセンブリ  
**次を使用して設定**: `UseStartup`  
**環境変数**: `ASPNETCORE_STARTUPASSEMBLY`

名前 (`string`) または型 (`TStartup`) 別のアセンブリを参照できます。 複数の `UseStartup` メソッドが呼び出された場合は、最後のメソッドが優先されます。

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseStartup("StartupAssemblyName")
```

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseStartup<TStartup>()
```

### <a name="web-root"></a>Web ルート

アプリの静的資産への相対パスを設定します。

**キー**: webroot  
**型**: *文字列*  
**既定**:既定値は、`wwwroot` です。 *{content root}/wwwroot* へのパスが存在する必要があります。 パスが存在しない場合は、no-op ファイル プロバイダーが使用されます。  
**次を使用して設定**: `UseWebRoot`  
**環境変数**: `ASPNETCORE_WEBROOT`

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseWebRoot("public")
```

詳細については次を参照してください:

* [基礎: Web ルート](xref:fundamentals/index#web-root)
* [コンテンツ ルート](#content-root)

## <a name="override-configuration"></a>構成をオーバーライドする

[Configuration](xref:fundamentals/configuration/index) を使用して、Web ホストを構成します。 次の例では、ホスト構成はオプションで *hostsettings.json* ファイルに指定されます。 *hostsettings.json* ファイルから読み込まれた構成は、コマンド ライン引数でオーバーライドされる場合があります。 (`config` の) ビルド構成は、[UseConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useconfiguration) でホストを構成する場合に使用されます。 `IWebHostBuilder` 構成はアプリの構成に追加されますが、その逆は当てはまりません&mdash;`ConfigureAppConfiguration` は `IWebHostBuilder` の構成に影響しません。

次のように、`UseUrls` で指定された構成をオーバーライドします (最初の構成は *hostsettings.json*、2 番目の構成はコマンドライン引数です)。

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateWebHostBuilder(args).Build().Run();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args)
    {
        var config = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("hostsettings.json", optional: true)
            .AddCommandLine(args)
            .Build();

        return WebHost.CreateDefaultBuilder(args)
            .UseUrls("http://*:5000")
            .UseConfiguration(config)
            .Configure(app =>
            {
                app.Run(context => 
                    context.Response.WriteAsync("Hello, World!"));
            });
    }
}
```

*hostsettings.json*:

```json
{
    urls: "http://*:5005"
}
```

> [!NOTE]
> [UseConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useconfiguration) は、指定された `IConfiguration` からホスト ビルダーの構成へのキーのみをコピーします。 そのため、JSON、INI、XML 設定のファイルに `reloadOnChange: true` を設定しても影響はありません。

特定の URL でホストを実行するように指定する場合、[dotnet run](/dotnet/core/tools/dotnet-run) の実行時にコマンド プロンプトから必要な値を渡すことができます。 コマンドライン引数は *hostsettings.json* ファイルからの `urls` 値をオーバーライドし、サーバーはポート 8080 でリッスンします。

```dotnetcli
dotnet run --urls "http://*:8080"
```

## <a name="manage-the-host"></a>ホストを管理する

**実行**

`Run` メソッドは Web アプリを起動し、ホストがシャットダウンするまで呼び出し元のスレッドをブロックします。

```csharp
host.Run();
```

**[開始]**

`Start` メソッドを呼び出して、ブロックせずにホストを実行します。

```csharp
using (host)
{
    host.Start();
    Console.ReadLine();
}
```

URL のリストが `Start` メソッドに渡された場合、指定された URL でリッスンします。

```csharp
var urls = new List<string>()
{
    "http://*:5000",
    "http://localhost:5001"
};

var host = new WebHostBuilder()
    .UseKestrel()
    .UseStartup<Startup>()
    .Start(urls.ToArray());

using (host)
{
    Console.ReadLine();
}
```

アプリは、便利な静的メソッドを使用し、`CreateDefaultBuilder` の構成済みの既定値を使用して、新しいホストを初期化して起動できます。 これらのメソッドは、コンソール出力なしでサーバーを起動します。その場合、[WaitForShutdown](/dotnet/api/microsoft.aspnetcore.hosting.webhostextensions.waitforshutdown) を指定し、中断される (Ctrl-C/SIGINT または SIGTERM) まで待機します。

**Start(RequestDelegate app)**

次のように `RequestDelegate` で始まります。

```csharp
using (var host = WebHost.Start(app => app.Response.WriteAsync("Hello, World!")))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

"Hello World!" という応答を受け取るには、ブラウザで `http://localhost:5000` に対する要求を行います。 `WaitForShutdown` は、中断される (Ctrl-C/SIGINT または SIGTERM) までブロックします。 アプリは `Console.WriteLine` メッセージを表示し、キー入力が終わるまで待機します。

**Start(string url, RequestDelegate app)**

次のように、URL と `RequestDelegate` で始まります。

```csharp
using (var host = WebHost.Start("http://localhost:8080", app => app.Response.WriteAsync("Hello, World!")))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

アプリが `http://localhost:8080` で応答する場合を除き、**Start(RequestDelegate app)** と同じ結果が生成されます。

**Start(Action\<IRouteBuilder> routeBuilder)**

次のように、`IRouteBuilder` ([Microsoft.AspNetCore.Routing](https://www.nuget.org/packages/Microsoft.AspNetCore.Routing/)) のインスタンスを使用してルーティング ミドルウェアを使用します。

```csharp
using (var host = WebHost.Start(router => router
    .MapGet("hello/{name}", (req, res, data) => 
        res.WriteAsync($"Hello, {data.Values["name"]}!"))
    .MapGet("buenosdias/{name}", (req, res, data) => 
        res.WriteAsync($"Buenos dias, {data.Values["name"]}!"))
    .MapGet("throw/{message?}", (req, res, data) => 
        throw new Exception((string)data.Values["message"] ?? "Uh oh!"))
    .MapGet("{greeting}/{name}", (req, res, data) => 
        res.WriteAsync($"{data.Values["greeting"]}, {data.Values["name"]}!"))
    .MapGet("", (req, res, data) => res.WriteAsync("Hello, World!"))))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

この例では、以下のブラウザー要求を使用します。

| 要求                                    | 応答                                 |
| ------------------------------------------ | ---------------------------------------- |
| `http://localhost:5000/hello/Martin`       | Hello, Martin!                           |
| `http://localhost:5000/buenosdias/Catrina` | Buenos dias, Catrina!                    |
| `http://localhost:5000/throw/ooops!`       | "ooops!" という文字列がある場合は例外をスローします。 |
| `http://localhost:5000/throw`              | "Uh oh!" という文字列がある場合は例外をスローします。 |
| `http://localhost:5000/Sante/Kevin`        | Sante, Kevin!                            |
| `http://localhost:5000`                    | Hello World!                             |

`WaitForShutdown` は、中断される (Ctrl-C/SIGINT または SIGTERM) までブロックします。 アプリは `Console.WriteLine` メッセージを表示し、キー入力が終わるまで待機します。

**Start(string url, Action\<IRouteBuilder> routeBuilder)**

次のように、URL と `IRouteBuilder` のインスタンスを使用します。

```csharp
using (var host = WebHost.Start("http://localhost:8080", router => router
    .MapGet("hello/{name}", (req, res, data) => 
        res.WriteAsync($"Hello, {data.Values["name"]}!"))
    .MapGet("buenosdias/{name}", (req, res, data) => 
        res.WriteAsync($"Buenos dias, {data.Values["name"]}!"))
    .MapGet("throw/{message?}", (req, res, data) => 
        throw new Exception((string)data.Values["message"] ?? "Uh oh!"))
    .MapGet("{greeting}/{name}", (req, res, data) => 
        res.WriteAsync($"{data.Values["greeting"]}, {data.Values["name"]}!"))
    .MapGet("", (req, res, data) => res.WriteAsync("Hello, World!"))))
{
    Console.WriteLine("Use Ctrl-C to shut down the host...");
    host.WaitForShutdown();
}
```

アプリが `http://localhost:8080` で応答する場合を除き、**Start(Action\<IRouteBuilder> routeBuilder)** と同じ結果が生成されます。

**StartWith(Action\<IApplicationBuilder> app)**

次のように、デリゲートを指定して `IApplicationBuilder` を構成します。

```csharp
using (var host = WebHost.StartWith(app => 
    app.Use(next => 
    {
        return async context => 
        {
            await context.Response.WriteAsync("Hello World!");
        };
    })))
{
    Console.WriteLine("Use Ctrl-C to shut down the host...");
    host.WaitForShutdown();
}
```

"Hello World!" という応答を受け取るには、ブラウザで `http://localhost:5000` に対する要求を行います。 `WaitForShutdown` は、中断される (Ctrl-C/SIGINT または SIGTERM) までブロックします。 アプリは `Console.WriteLine` メッセージを表示し、キー入力が終わるまで待機します。

**StartWith(string url, Action\<IApplicationBuilder> app)**

次のように、URL とデリゲートを指定して `IApplicationBuilder` を構成します。

```csharp
using (var host = WebHost.StartWith("http://localhost:8080", app => 
    app.Use(next => 
    {
        return async context => 
        {
            await context.Response.WriteAsync("Hello World!");
        };
    })))
{
    Console.WriteLine("Use Ctrl-C to shut down the host...");
    host.WaitForShutdown();
}
```

アプリが `http://localhost:8080` で応答する場合を除き、**StartWith(Action\<IApplicationBuilder> app)** と同じ結果が生成されます。

::: moniker range=">= aspnetcore-3.0"

## <a name="iwebhostenvironment-interface"></a>IWebHostEnvironment インターフェイス

`IWebHostEnvironment` インターフェイスでは、アプリの Web ホスティング環境に関する情報が提供されます。 プロパティと拡張メソッドを使用するには、[コンストラクターの挿入](xref:fundamentals/dependency-injection)を使用して `IWebHostEnvironment` を取得します。

```csharp
public class CustomFileReader
{
    private readonly IWebHostEnvironment _env;

    public CustomFileReader(IWebHostEnvironment env)
    {
        _env = env;
    }

    public string ReadFile(string filePath)
    {
        var fileProvider = _env.WebRootFileProvider;
        // Process the file here
    }
}
```

[規則ベースのアプローチ](xref:fundamentals/environments#environment-based-startup-class-and-methods)を使用して、環境に基づいて起動時にアプリを構成することができます。 あるいは、次のように `ConfigureServices` で使用するために `IWebHostEnvironment` を `Startup` コンストラクターに挿入します。

```csharp
public class Startup
{
    public Startup(IWebHostEnvironment env)
    {
        HostingEnvironment = env;
    }

    public IWebHostEnvironment HostingEnvironment { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        if (HostingEnvironment.IsDevelopment())
        {
            // Development configuration
        }
        else
        {
            // Staging/Production configuration
        }

        var contentRootPath = HostingEnvironment.ContentRootPath;
    }
}
```

> [!NOTE]
> `IsDevelopment` 拡張メソッドに加え、`IWebHostEnvironment` は `IsStaging`、`IsProduction`、`IsEnvironment(string environmentName)` メソッドを提供します。 詳細については、「<xref:fundamentals/environments>」を参照してください。

処理パイプラインを設定するために、次のように `IWebHostEnvironment` サービスを `Configure` メソッドに直接挿入することもできます。

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        // In Development, use the Developer Exception Page
        app.UseDeveloperExceptionPage();
    }
    else
    {
        // In Staging/Production, route exceptions to /error
        app.UseExceptionHandler("/error");
    }

    var contentRootPath = env.ContentRootPath;
}
```

カスタムの[ミドルウェア](xref:fundamentals/middleware/write)を作成する際に、次のように `IWebHostEnvironment` を `Invoke` メソッドに挿入することができます。

```csharp
public async Task Invoke(HttpContext context, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        // Configure middleware for Development
    }
    else
    {
        // Configure middleware for Staging/Production
    }

    var contentRootPath = env.ContentRootPath;
}
```

::: moniker-end

::: moniker range="< aspnetcore-3.0"

## <a name="ihostingenvironment-interface"></a>IHostingEnvironment インターフェイス

[IHostingEnvironment インターフェイス](/dotnet/api/microsoft.aspnetcore.hosting.ihostingenvironment)では、アプリの Web ホスティング環境に関する情報を提供します。 プロパティと拡張メソッドを使用するには、[コンストラクターの挿入](xref:fundamentals/dependency-injection)を使用して `IHostingEnvironment` を取得します。

```csharp
public class CustomFileReader
{
    private readonly IHostingEnvironment _env;

    public CustomFileReader(IHostingEnvironment env)
    {
        _env = env;
    }

    public string ReadFile(string filePath)
    {
        var fileProvider = _env.WebRootFileProvider;
        // Process the file here
    }
}
```

[規則ベースのアプローチ](xref:fundamentals/environments#environment-based-startup-class-and-methods)を使用して、環境に基づいて起動時にアプリを構成することができます。 あるいは、次のように `ConfigureServices` で使用するために `IHostingEnvironment` を `Startup` コンストラクターに挿入します。

```csharp
public class Startup
{
    public Startup(IHostingEnvironment env)
    {
        HostingEnvironment = env;
    }

    public IHostingEnvironment HostingEnvironment { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        if (HostingEnvironment.IsDevelopment())
        {
            // Development configuration
        }
        else
        {
            // Staging/Production configuration
        }

        var contentRootPath = HostingEnvironment.ContentRootPath;
    }
}
```

> [!NOTE]
> `IsDevelopment` 拡張メソッドに加え、`IHostingEnvironment` は `IsStaging`、`IsProduction`、`IsEnvironment(string environmentName)` メソッドを提供します。 詳細については、「<xref:fundamentals/environments>」を参照してください。

処理パイプラインを設定するために、次のように `IHostingEnvironment` サービスを `Configure` メソッドに直接挿入することもできます。

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        // In Development, use the Developer Exception Page
        app.UseDeveloperExceptionPage();
    }
    else
    {
        // In Staging/Production, route exceptions to /error
        app.UseExceptionHandler("/error");
    }

    var contentRootPath = env.ContentRootPath;
}
```

カスタムの[ミドルウェア](xref:fundamentals/middleware/write)を作成する際に、次のように `IHostingEnvironment` を `Invoke` メソッドに挿入することができます。

```csharp
public async Task Invoke(HttpContext context, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        // Configure middleware for Development
    }
    else
    {
        // Configure middleware for Staging/Production
    }

    var contentRootPath = env.ContentRootPath;
}
```

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

## <a name="ihostapplicationlifetime-interface"></a>IHostApplicationLifetime インターフェイス

`IHostApplicationLifetime` では、起動後とシャットダウンのアクティビティを実行できます。 インターフェイスの 3 つのプロパティは、起動とシャットダウン イベントを定義する `Action` メソッドを登録するために使用されるキャンセル トークンです。

| キャンセル トークン    | トリガーのタイミング |
| --------------------- | --------------------- |
| `ApplicationStarted`  | ホストが完全に起動されたとき。 |
| `ApplicationStopped`  | ホストが正常なシャットダウンを完了しているとき。 すべての要求を処理する必要があります。 このイベントが完了するまで、シャットダウンはブロックされます。 |
| `ApplicationStopping` | ホストが正常なシャットダウンを行っているとき。 要求がまだ処理されている可能性がある場合。 このイベントが完了するまで、シャットダウンはブロックされます。 |

```csharp
public class Startup
{
    public void Configure(IApplicationBuilder app, IHostApplicationLifetime appLifetime)
    {
        appLifetime.ApplicationStarted.Register(OnStarted);
        appLifetime.ApplicationStopping.Register(OnStopping);
        appLifetime.ApplicationStopped.Register(OnStopped);

        Console.CancelKeyPress += (sender, eventArgs) =>
        {
            appLifetime.StopApplication();
            // Don't terminate the process immediately, wait for the Main thread to exit gracefully.
            eventArgs.Cancel = true;
        };
    }

    private void OnStarted()
    {
        // Perform post-startup activities here
    }

    private void OnStopping()
    {
        // Perform on-stopping activities here
    }

    private void OnStopped()
    {
        // Perform post-stopped activities here
    }
}
```

`StopApplication` は、アプリの終了を要求します。 以下のクラスは、クラスの `Shutdown` メソッドが呼び出されると、`StopApplication` を使ってアプリを正常にシャットダウンします。

```csharp
public class MyClass
{
    private readonly IHostApplicationLifetime _appLifetime;

    public MyClass(IHostApplicationLifetime appLifetime)
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

::: moniker range="< aspnetcore-3.0"

## <a name="iapplicationlifetime-interface"></a>IApplicationLifetime インターフェイス

[IApplicationLifetime](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime) は、起動後とシャットダウンのアクティビティに適用できます。 インターフェイスの 3 つのプロパティは、起動とシャットダウン イベントを定義する `Action` メソッドを登録するために使用されるキャンセル トークンです。

| キャンセル トークン    | トリガーのタイミング |
| --------------------- | --------------------- |
| [ApplicationStarted](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.applicationstarted) | ホストが完全に起動されたとき。 |
| [ApplicationStopped](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.applicationstopped) | ホストが正常なシャットダウンを完了しているとき。 すべての要求を処理する必要があります。 このイベントが完了するまで、シャットダウンはブロックされます。 |
| [ApplicationStopping](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.applicationstopping) | ホストが正常なシャットダウンを行っているとき。 要求がまだ処理されている可能性がある場合。 このイベントが完了するまで、シャットダウンはブロックされます。 |

```csharp
public class Startup
{
    public void Configure(IApplicationBuilder app, IApplicationLifetime appLifetime)
    {
        appLifetime.ApplicationStarted.Register(OnStarted);
        appLifetime.ApplicationStopping.Register(OnStopping);
        appLifetime.ApplicationStopped.Register(OnStopped);

        Console.CancelKeyPress += (sender, eventArgs) =>
        {
            appLifetime.StopApplication();
            // Don't terminate the process immediately, wait for the Main thread to exit gracefully.
            eventArgs.Cancel = true;
        };
    }

    private void OnStarted()
    {
        // Perform post-startup activities here
    }

    private void OnStopping()
    {
        // Perform on-stopping activities here
    }

    private void OnStopped()
    {
        // Perform post-stopped activities here
    }
}
```

[StopApplication](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime.stopapplication) は、アプリの終了を要求します。 以下のクラスは、クラスの `Shutdown` メソッドが呼び出されると、`StopApplication` を使ってアプリを正常にシャットダウンします。

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

## <a name="scope-validation"></a>スコープの検証

アプリの環境が開発の場合、[CreateDefaultBuilder](/dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder) は [ServiceProviderOptions.ValidateScopes](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions.validatescopes) を `true` に設定します。

`ValidateScopes` が `true` に設定されていると、既定のサービス プロバイダーはチェックを実行して次のことを確認します。

* スコープ サービスが、ルート サービス プロバイダーによって直接的または間接的に解決されない。
* スコープ サービスが、シングルトンに直接または間接に挿入されない。

[BuildServiceProvider](/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectioncontainerbuilderextensions.buildserviceprovider) が呼び出されると、ルート サービス プロバイダーが作成されます。 ルート サービス プロバイダーの有効期間は、プロバイダーがアプリで開始されるとアプリとサーバーの有効期間に対応し、アプリのシャットダウンには破棄されます。

スコープ サービスは、それを作成したコンテナーによって破棄されます。 ルート コンテナーにスコープ サービスが作成されると、サービスはアプリ/サーバーのシャットダウン時に、ルート コンテナーによってのみ破棄されるため、サービスの有効期間は実質的にシングルトンに昇格されます。 `BuildServiceProvider` が呼び出されると、サービス スコープの検証がこれらの状況をキャッチします。

運用環境も含めて、常にスコープを検証するには、ホスト ビルダー上の [UseDefaultServiceProvider](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.usedefaultserviceprovider) で [ServiceProviderOptions](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions) を構成します。

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseDefaultServiceProvider((context, options) => {
        options.ValidateScopes = true;
    })
```

## <a name="additional-resources"></a>その他の技術情報

* <xref:host-and-deploy/iis/index>
* <xref:host-and-deploy/linux-nginx>
* <xref:host-and-deploy/linux-apache>
* <xref:host-and-deploy/windows-service>
