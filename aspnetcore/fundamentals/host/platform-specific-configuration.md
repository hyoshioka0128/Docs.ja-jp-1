---
title: ASP.NET Core でホスティング スタートアップ アセンブリを使用する
author: rick-anderson
description: IHostingStartup 実装を使用して、外部アセンブリから ASP.NET Core アプリを拡張する方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc, seodec18
ms.date: 09/26/2019
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
uid: fundamentals/configuration/platform-specific-configuration
ms.openlocfilehash: b39215a70f990afeb7d3fe0a62981113b154354e
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588244"
---
# <a name="use-hosting-startup-assemblies-in-aspnet-core"></a>ASP.NET Core でホスティング スタートアップ アセンブリを使用する

作成者: [Pavel Krymets](https://github.com/pakrym)

::: moniker range=">= aspnetcore-3.0"

<xref:Microsoft.AspNetCore.Hosting.IHostingStartup> (ホスティング スタートアップ) の実装を使うと、外部アセンブリからの起動時にアプリに拡張機能を追加できます。 たとえば、外部ライブラリではホスティング スタートアップ実装を使用して、追加の構成プロバイダーまたはサービスをアプリに提供できます。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/platform-specific-configuration/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="hostingstartup-attribute"></a>HostingStartup 属性

[HostingStartup](xref:Microsoft.AspNetCore.Hosting.HostingStartupAttribute) 属性は、実行時にアクティブ化されるホスティング スタートアップ アセンブリが存在することを示しています。

入力アセンブリまたは `Startup` クラスを含むアセンブリは、`HostingStartup` 属性について自動的にスキャンされます。 `HostingStartup` 属性を検索するアセンブリの一覧は、[WebHostDefaults.HostingStartupAssembliesKey](xref:Microsoft.AspNetCore.Hosting.WebHostDefaults.HostingStartupAssembliesKey) 内の構成からランタイム時に読み込まれます。 検出から除外されるアセンブリの一覧は、[WebHostDefaults.HostingStartupExcludeAssembliesKey](xref:Microsoft.AspNetCore.Hosting.WebHostDefaults.HostingStartupExcludeAssembliesKey) から読み込まれます。

次に例では、ホスティング スタートアップ アセンブリの名前空間は `StartupEnhancement` となります。 ホスティング スタートアップ コードを含むクラスは `StartupEnhancementHostingStartup` です。

[!code-csharp[](platform-specific-configuration/samples-snapshot/3.x/StartupEnhancement.cs?name=snippet1)]

`HostingStartup` 属性は、通常、ホスティング スタートアップ アセンブリの `IHostingStartup` 実装クラス ファイル内に置かれます。

## <a name="discover-loaded-hosting-startup-assemblies"></a>読み込まれたホスティング スタートアップ アセンブリを見つける

読み込まれたホスティング スタートアップ アセンブリを検出するには、ログ記録を有効にしてアプリのログを確認します。 アセンブリの読み込み時に発生したエラーがログ記録されます。 読み込まれたホスティング スタートアップ アセンブリは、デバッグ レベルでログ記録され、すべてのエラーが記録されます。

## <a name="disable-automatic-loading-of-hosting-startup-assemblies"></a>ホスティング スタートアップ アセンブリの自動読み込みを無効にする

ホスティング スタートアップ アセンブリの自動読み込みを無効にするには、次の方法のいずれかを使用します。

* すべてのホスティング スタートアップ アセンブリの読み込みを回避するには、次のいずれかを `true` または `1` に設定します。

  * [ホスティング スタートアップを回避する] ホスト構成設定。

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseSetting(
                        WebHostDefaults.PreventHostingStartupKey, "true")
                    .UseStartup<Startup>();
            });
    ```

  * `ASPNETCORE_PREVENTHOSTINGSTARTUP` 環境変数。

* 特定のホスティング スタートアップ アセンブリの読み込みを回避するには、次に示すいずれかを、起動時に除外するホスティング スタートアップ アセンブリを示すセミコロンで区切られた文字列に設定します。

  * [Hosting Startup Exclude Assemblies]\(除外するホスティング スタートアップ アセンブリ\) ホスト構成設定。

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseSetting(
                        WebHostDefaults.HostingStartupExcludeAssembliesKey, 
                        "{ASSEMBLY1;ASSEMBLY2; ...}")
                    .UseStartup<Startup>();
            });
    ```

  * `ASPNETCORE_HOSTINGSTARTUPEXCLUDEASSEMBLIES` 環境変数。

ホスト構成設定と環境変数が両方とも設定されている場合は、ホスト設定によって動作が制御されます。

ホスト設定または環境変数を使用してホスティング スタートアップ アセンブリを無効にすると、アセンブリがグローバルに無効になり、場合によってはアプリのいくつかの属性も無効になります。

## <a name="project"></a>Project

次のいずれかの種類のプロジェクトを使用して、ホスティング スタートアップを作成します。

* [クラス ライブラリ](#class-library)
* [エントリ ポイントのないコンソール アプリ](#console-app-without-an-entry-point)

### <a name="class-library"></a>クラス ライブラリ

クラス ライブラリでは、ホスティング スタートアップの拡張機能を提供できます。 このライブラリには `HostingStartup` 属性が含まれています。

[サンプル コード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/platform-specific-configuration/samples/)には、Razor Pages アプリ、*HostingStartupApp*、クラス ライブラリ、*HostingStartupLibrary* が含まれています。 クラス ライブラリには次のものが含まれています。

* `IHostingStartup` を実装するホスティング スタートアップ クラス `ServiceKeyInjection`。 `ServiceKeyInjection` では、メモリ内の構成プロバイダー ([AddInMemoryCollection](xref:Microsoft.Extensions.Configuration.MemoryConfigurationBuilderExtensions.AddInMemoryCollection*)) を使用して、サービスの文字列のペアがアプリの構成に追加されます。
* ホスティング スタートアップの名前空間とクラスを識別する `HostingStartup` 属性。

`ServiceKeyInjection` クラスの <xref:Microsoft.AspNetCore.Hosting.IHostingStartup.Configure*> メソッドでは、<xref:Microsoft.AspNetCore.Hosting.IWebHostBuilder> を使ってアプリに拡張機能を追加します。

*HostingStartupLibrary/ServiceKeyInjection.cs*:

[!code-csharp[](platform-specific-configuration/samples/3.x/HostingStartupLibrary/ServiceKeyInjection.cs?name=snippet1)]

アプリのインデックス ページでは、クラス ライブラリのホスティング スタートアップ アセンブリによって設定された 2 つのキーの構成値が読み取られて表示されます。

*HostingStartupApp/Pages/Index.cshtml.cs*:

[!code-csharp[](platform-specific-configuration/samples/3.x/HostingStartupApp/Pages/Index.cshtml.cs?name=snippet1&highlight=5-6,11-12)]

[サンプル コード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/platform-specific-configuration/samples/) には、別のホスティング スタートアップ *HostingStartupPackage* を提供する NuGet パッケージ プロジェクトも含まれています。 パッケージには、前に説明したクラス ライブラリと同じ特性があります。 パッケージには次のものが含まれます。

* `IHostingStartup` を実装するホスティング スタートアップ クラス `ServiceKeyInjection`。 `ServiceKeyInjection` では、アプリの構成にサービス文字列のペアが追加されます。
* `HostingStartup`属性。

*HostingStartupPackage/ServiceKeyInjection.cs*:

[!code-csharp[](platform-specific-configuration/samples/3.x/HostingStartupPackage/ServiceKeyInjection.cs?name=snippet1)]

アプリのインデックス ページでは、パッケージのホスティング スタートアップ アセンブリによって設定された 2 つのキーの構成値が読み取られて表示されます。

*HostingStartupApp/Pages/Index.cshtml.cs*:

[!code-csharp[](platform-specific-configuration/samples/3.x/HostingStartupApp/Pages/Index.cshtml.cs?name=snippet1&highlight=7-8,13-14)]

### <a name="console-app-without-an-entry-point"></a>エントリ ポイントのないコンソール アプリ

*このアプローチは、.NET Core アプリでのみ利用でき、.NET Framework では利用できません。*

`HostingStartup` 属性を含むエントリ ポイントがないコンソール アプリ内では、アクティブ化に関するコンパイル時参照を必要としない動的ホスティング スタートアップ拡張機能を指定できます。 コンソール アプリを発行すると、ランタイム ストアから使用できるホスティング スタートアップ アセンブリが生成されます。

このプロセスにおいてエントリ ポイントのないコンソール アプリが使用される理由は、次のとおりです。

* ホスティング スタートアップ アセンブリ内のホスティング スタートアップを使用するには、依存関係ファイルが必要です。 依存関係ファイルは、ライブラリではなくアプリを発行することによって生成される、実行可能なアプリ資産です。
* [ランタイム パッケージ ストア](/dotnet/core/deploying/runtime-store) にライブラリを直接追加することはできません。それには、共有ランタイムを対象とする実行可能なプロジェクトが必要です。

動的ホスティング スタートアップを作成する場合:

* 次のようなエントリ ポイントがないコンソール アプリからホスティング スタートアップ アセンブリが作成されます。
  * `IHostingStartup` の実装を含むクラスを含んでいる。
  * `IHostingStartup` 実装クラスを識別するための [HostingStartup](xref:Microsoft.AspNetCore.Hosting.HostingStartupAttribute) 属性を含んでいる。
* ホスティング スタートアップの依存関係を取得するには、コンソール アプリを発行します。 コンソール アプリを発行すると、その結果として、依存関係ファイルから未使用の依存関係が切り捨てられます。
* ホスティング スタートアップ アセンブリのランタイムの場所を設定するために、依存関係ファイルが変更されます。
* ホスティング スタートアップ アセンブリとその依存関係ファイルは、ランタイム パッケージ ストアに配置されます。 ホスティング スタートアップ アセンブリとその依存関係ファイルを検出する場合、それらは環境変数のペアに記載されています。

コンソール アセンブリでは、[Microsoft.AspNetCore.Hosting.Abstractions](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.Abstractions/) パッケージが参照されます。

[!code-xml[](platform-specific-configuration/samples-snapshot/3.x/StartupEnhancement.csproj)]

[HostingStartup](xref:Microsoft.AspNetCore.Hosting.HostingStartupAttribute) 属性を使うと、<xref:Microsoft.AspNetCore.Hosting.IWebHost> のビルド時に、読み込みと実行のための `IHostingStartup` の実装としてクラスが識別されます。 次の例では、名前空間は `StartupEnhancement`、クラスは `StartupEnhancementHostingStartup` です。

[!code-csharp[](platform-specific-configuration/samples-snapshot/3.x/StartupEnhancement.cs?name=snippet1)]

クラスは `IHostingStartup` を実装しています。 このクラスの <xref:Microsoft.AspNetCore.Hosting.IHostingStartup.Configure*> メソッドでは、<xref:Microsoft.AspNetCore.Hosting.IWebHostBuilder> を使ってアプリに拡張機能を追加します。 ホスティング スタートアップ アセンブリ内の `IHostingStartup.Configure` は、ユーザー コード内の `Startup.Configure` よりも前にランタイムによって呼び出されます。このため、ホスティング スタートアップ アセンブリによって提供されるすべての構成をユーザー コードで上書きすることができます。

[!code-csharp[](platform-specific-configuration/samples-snapshot/3.x/StartupEnhancement.cs?name=snippet2&highlight=3,5)]

`IHostingStartup` プロジェクトのビルド時に、依存関係ファイル ( *.deps.json*) によってアセンブリの `runtime` の場所が *bin* フォルダーに設定されます。

[!code-json[](platform-specific-configuration/samples-snapshot/3.x/StartupEnhancement1.deps.json?range=2-13&highlight=8)]

ファイルの一部のみが示されています。 例のアセンブリ名は `StartupEnhancement` です。

## <a name="configuration-provided-by-the-hosting-startup"></a>ホスティング スタートアップによって指定される構成

ホスティング スタートアップの構成、アプリの構成のどちらを優先させるかによって、2 つの異なる方法で構成を処理できます。

1. アプリの <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureAppConfiguration*> デリゲートを実行した後で <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureAppConfiguration*> を使って構成を読み込み、アプリに構成を指定します。 この方法を使うと、ホスティング スタートアップの構成の方がアプリの構成よりも優先されます。
1. アプリの <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureAppConfiguration*> デリゲートを実行する前に <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseConfiguration*> を使って構成を読み込み、アプリに構成を指定します。 この方法を使うと、アプリの構成値の方がホスティング スタートアップにより指定されるものよりも優先されます。

```csharp
public class ConfigurationInjection : IHostingStartup
{
    public void Configure(IWebHostBuilder builder)
    {
        Dictionary<string, string> dict;

        builder.ConfigureAppConfiguration(config =>
        {
            dict = new Dictionary<string, string>
            {
                {"ConfigurationKey1", 
                    "From IHostingStartup: Higher priority " +
                    "than the app's configuration."},
            };

            config.AddInMemoryCollection(dict);
        });

        dict = new Dictionary<string, string>
        {
            {"ConfigurationKey2", 
                "From IHostingStartup: Lower priority " +
                "than the app's configuration."},
        };

        var builtConfig = new ConfigurationBuilder()
            .AddInMemoryCollection(dict)
            .Build();

        builder.UseConfiguration(builtConfig);
    }
}
```

## <a name="specify-the-hosting-startup-assembly"></a>ホスティング スタートアップ アセンブリを指定する

クラス ライブラリまたはコンソール アプリのいずれかで提供されるホスティング スタートアップの場合は、`ASPNETCORE_HOSTINGSTARTUPASSEMBLIES` 環境変数内にホスティング スタートアップ アセンブリの名前を指定します。 環境変数はアセンブリのセミコロン区切りのリストです。

ホスティング スタートアップ アセンブリのみが、`HostingStartup` 属性に対してスキャンされます。 サンプル アプリ *HostingStartupApp* では、前述したホスティング スタートアップを検出するために、環境変数が次の値に設定されています。

```
HostingStartupLibrary;HostingStartupPackage;StartupDiagnostics
```

また、ホスティング スタートアップ アセンブリは、[ホスティング スタートアップ アセンブリ] のホスト構成設定を使用して設定することもできます。

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseSetting(
                    WebHostDefaults.HostingStartupAssembliesKey, 
                    "{ASSEMBLY1;ASSEMBLY2; ...}")
                .UseStartup<Startup>();
        });
```

複数のホスティング スタートアップ アセンブリが存在する場合、それらの <xref:Microsoft.AspNetCore.Hosting.IHostingStartup.Configure*> メソッドは、アセンブリが一覧表示されている順序で実行されます。

## <a name="activation"></a>アクティベーション

ホスティング スタートアップのアクティブ化のオプションは次のとおりです。

* [ランタイム ストア](#runtime-store):アクティブ化では、アクティブ化に関するコンパイル時参照を必要としません。 サンプル アプリでは、ホスティング スタートアップ アセンブリおよび依存関係のファイルが *deployment* フォルダーに配置されています。これにより、複数のコンピューターから成る環境でのホスティング スタートアップの配置が容易になります。 *deployment* フォルダーには、ホスティングスタートアップが有効になるように配置システム上で環境変数を作成および変更する PowerShell スクリプトも含まれています。
* アクティブ化で必須のコンパイル時参照
  * [NuGet パッケージ](#nuget-package)
  * [プロジェクトの bin フォルダー](#project-bin-folder)

### <a name="runtime-store"></a>ランタイム ストア

ホスティング スタートアップ実装は、[ランタイム ストア](/dotnet/core/deploying/runtime-store)に置かれます。 アセンブリへのコンパイル時参照は、機能強化されたアプリで必須ではありません。

ホスティング スタートアップをビルドした後、プロジェクトのマニフェスト ファイルと [dotnet store](/dotnet/core/tools/dotnet-store) コマンドの使用によってランタイム ストアが生成されます。

```dotnetcli
dotnet store --manifest {MANIFEST FILE} --runtime {RUNTIME IDENTIFIER} --output {OUTPUT LOCATION} --skip-optimization
```

サンプル アプリ (*RuntimeStore* プロジェクト) では、次のコマンドを使用します。

```dotnetcli
dotnet store --manifest store.manifest.csproj --runtime win7-x64 --output ./deployment/store --skip-optimization
```

ランタイムでランタイム ストアを検出できるように、ランタイム ストアの場所を環境変数 `DOTNET_SHARED_STORE` に追加します。

**ホスティング スタートアップの依存関係ファイルを変更して配置する**

拡張機能へのパッケージ参照なしで拡張機能をアクティブ化するには、`additionalDeps` を使用してランタイムに追加の依存関係を指定します。 `additionalDeps` を使用すると、次のことを実行できます。

* スタートアップ時にアプリの独自の *.deps.json* ファイルとマージさせる追加の *.deps.json* ファイルのセットを指定することで、アプリのライブラリのグラフを拡張します。
* ホスティング スタートアップ アセンブリを検出可能および読み込み可能にします。

追加の依存関係ファイルを生成するために推奨される方法は次のとおりです。

 1. 前のセクションで参照したランタイム ストアのマニフェスト ファイルに対して `dotnet publish` を実行します。
 1. ライブラリと、結果として得られる *deps.json* ファイルの `runtime` セクションから、マニフェスト参照を削除します。

プロジェクトの例では、`targets` と `libraries` セクションから `store.manifest/1.0.0` プロパティが削除されます。

```json
{
  "runtimeTarget": {
    "name": ".NETCoreApp,Version=v3.0",
    "signature": ""
  },
  "compilationOptions": {},
  "targets": {
    ".NETCoreApp,Version=v3.0": {
      "store.manifest/1.0.0": {
        "dependencies": {
          "StartupDiagnostics": "1.0.0"
        },
        "runtime": {
          "store.manifest.dll": {}
        }
      },
      "StartupDiagnostics/1.0.0": {
        "runtime": {
          "lib/netcoreapp3.0/StartupDiagnostics.dll": {
            "assemblyVersion": "1.0.0.0",
            "fileVersion": "1.0.0.0"
          }
        }
      }
    }
  },
  "libraries": {
    "store.manifest/1.0.0": {
      "type": "project",
      "serviceable": false,
      "sha512": ""
    },
    "StartupDiagnostics/1.0.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-xrhzuNSyM5/f4ZswhooJ9dmIYLP64wMnqUJSyTKVDKDVj5T+qtzypl8JmM/aFJLLpYrf0FYpVWvGujd7/FfMEw==",
      "path": "startupdiagnostics/1.0.0",
      "hashPath": "startupdiagnostics.1.0.0.nupkg.sha512"
    }
  }
}
```

次の場所に *.deps.json* ファイルを配置します。

```
{ADDITIONAL DEPENDENCIES PATH}/shared/{SHARED FRAMEWORK NAME}/{SHARED FRAMEWORK VERSION}/{ENHANCEMENT ASSEMBLY NAME}.deps.json
```

* `{ADDITIONAL DEPENDENCIES PATH}`:`DOTNET_ADDITIONAL_DEPS` 環境変数に追加される場所。
* `{SHARED FRAMEWORK NAME}`:この追加の依存関係ファイルに必要な共有フレームワーク。
* `{SHARED FRAMEWORK VERSION}`:共有フレームワークの最小バージョン。
* `{ENHANCEMENT ASSEMBLY NAME}`:拡張機能のアセンブリ名。

サンプル アプリ (*RuntimeStore* プロジェクト) では、追加の依存関係ファイルは以下の場所に配置されます。

```
deployment/additionalDeps/shared/Microsoft.AspNetCore.App/3.0.0/StartupDiagnostics.deps.json
```

ランタイムでランタイム ストアの場所を検出できるように、追加の依存関係ファイルの場所が環境変数 `DOTNET_ADDITIONAL_DEPS` に追加されます。

サンプル アプリ (*RuntimeStore* プロジェクト) では、[PowerShell](/powershell/scripting/powershell-scripting) スクリプトを使用してランタイム ストアのビルドと追加の依存関係ファイルの生成を行います。

さまざまなオペレーティング システムの環境変数を設定する方法の例については、[複数の環境の使用](xref:fundamentals/environments)に関するページを参照してください。

**配置**

サンプル アプリでは、複数のコンピューターから成る環境へのホスティング スタートアップの配置を容易にするために、発行された出力内に、次を含む *deployment* フォルダーが作成されます。

* ホスティング スタートアップのランタイム ストア。
* ホスティング スタートアップの依存関係ファイル。
* ホスティング スタートアップのアクティブ化をサポートできるように、`ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`、`DOTNET_SHARED_STORE`、`DOTNET_ADDITIONAL_DEPS` を作成および変更する PowerShell スクリプト。 このスクリプトは、配置システム上の PowerShell 管理用コマンド プロンプトから実行します。

### <a name="nuget-package"></a>NuGet パッケージ

NuGet パッケージでは、ホスティング スタートアップの拡張機能を提供できます。 このパッケージには `HostingStartup` 属性があります。 このパッケージで提供される種類のホスティング スタートアップは、次のアプローチのいずれかを使用してアプリで利用できるようになります。

* 機能強化されたアプリのプロジェクト ファイルでは、アプリのプロジェクト ファイル内のホスティング スタートアップに対するパッケージ参照 (コンパイル時参照) が作成されます。 コンパイル時参照を適用すると、ホスティング スタートアップ アセンブリとその依存関係のすべてが、アプリの依存関係ファイル ( *.deps.json*) に組み込まれます。 このアプローチは、[nuget.org](https://www.nuget.org/) に発行されるホスティング スタートアップ アセンブリ パッケージに適用されます。
* ホスティング スタートアップの依存関係ファイルは、「[ランタイム ストア](#runtime-store)」のセクションで説明にしたように (コンパイル参照がない場合)、機能強化されたアプリで利用できるようになります。

NuGet パッケージとランタイム ストアの詳細については、次のトピックを参照してください。

* [クロスプラットフォーム ツールを使用して NuGet パッケージを作成する方法](/dotnet/core/deploying/creating-nuget-packages)
* [パッケージを公開する](/nuget/create-packages/publish-a-package)
* [ランタイム パッケージ ストア](/dotnet/core/deploying/runtime-store)

### <a name="project-bin-folder"></a>プロジェクトの bin フォルダー

ホスティング スタートアップの拡張機能は、機能強化されたアプリ内の *bin* 配置アセンブリによって提供できます。 このアセンブリで提供される種類のホスティング スタートアップは、次のアプローチのいずれかを使用してアプリで利用できるようになります。

* 機能強化されたアプリのプロジェクト ファイルでは、ホスティング スタートアップへのアセンブリ参照 (コンパイル時参照) が作成されます。 コンパイル時参照を適用すると、ホスティング スタートアップ アセンブリとその依存関係のすべてが、アプリの依存関係ファイル ( *.deps.json*) に組み込まれます。 この手法は、展開シナリオによって、ホスティング スタートアップのアセンブリ ( *.dll* ファイル) に対してコンパイル時参照を作成し、次のいずれかにそのアセンブリを移動させることが要請される場合に適用されます。
  * 使用するプロジェクト。
  * 使用するプロジェクトでアクセスできる場所。
* ホスティング スタートアップの依存関係ファイルは、「[ランタイム ストア](#runtime-store)」のセクションで説明にしたように (コンパイル参照がない場合)、機能強化されたアプリで利用できるようになります。
* .NET Framework を対象にするとき、アセンブリは既定の読み込みコンテキストで読み込み可能です。これは、.NET Framework の場合、アセンブリが次のいずれかの場所にあることを意味します。
  * アプリケーション ベース パス:アプリの実行可能ファイル ( *.exe*) が置かれている *bin* フォルダー。
  * グローバル アセンブリ キャッシュ (GAC):GAC には、複数の .NET Framework アプリで共有されるアセンブリが格納されています。 詳細については、「[方法: アセンブリをグローバル アセンブリ キャッシュにインストールする](/dotnet/framework/app-domains/how-to-install-an-assembly-into-the-gac)」を参照してください。これは .NET Framework ドキュメントにあります。

## <a name="sample-code"></a>サンプル コード

[サンプル コード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/platform-specific-configuration/samples/) ([ダウンロードする方法](xref:index#how-to-download-a-sample)) では、ホスティング スタートアップ実装のシナリオを示します。

* 2 つのホスティング スタートアップ アセンブリ (クラス ライブラリ) ではそれぞれ、メモリ内の構成のキーと値のペアの組が設定されます。
  * NuGet パッケージ (*HostingStartupPackage*)
  * クラス ライブラリ (*HostingStartupLibrary*)
* ホスティング スタートアップは、ランタイム ストア配置アセンブリ (*StartupDiagnostics*) からアクティブ化されます。 このアセンブリによってスタートアップ時に 2 つのミドルウェアがアプリに追加され、これにより診断情報が提供されます。
  * 登録サービス
  * アドレス (スキーム、ホスト、パス ベース、パス、クエリ文字列)
  * 接続 (リモート IP、リモート ポート、ローカル IP、ローカル ポート、クライアント証明書)
  * 要求ヘッダー
  * 環境変数

サンプルを実行するには

**NuGet パッケージからのアクティブ化**

1. [dotnet pack](/dotnet/core/tools/dotnet-pack) コマンドを使用して *HostingStartupPackage* パッケージをコンパイルします。
1. *HostingStartupPackage* のパッケージのアセンブリ名を `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES` 環境変数に追加します。
1. アプリをコンパイルして実行します。 機能拡張されたアプリにはパッケージ参照 (コンパイル時参照) が存在します。 アプリのプロジェクト ファイル内の `<PropertyGroup>` では、パッケージ プロジェクトの出力 ( *../HostingStartupPackage/Bin/debug*) がパッケージ ソースとして指定されます。 これにより、[nuget.org](https://www.nuget.org/) にパッケージをアップロードすることなく、アプリによりパッケージが使用されるようになります。詳細については、HostingStartupApp のプロジェクト ファイル内の注を参照してください。

   ```xml
   <PropertyGroup>
     <RestoreSources>$(RestoreSources);https://api.nuget.org/v3/index.json;../HostingStartupPackage/bin/Debug</RestoreSources>
   </PropertyGroup>
   ```

1. インデックス ページで表示されるサービス構成キー値が、パッケージの `ServiceKeyInjection.Configure` メソッドによって設定された値と一致することを確認します。

*HostingStartupPackage* プロジェクトに変更を加えてそれをコンパイルする場合は、ローカル キャッシュから、古いパッケージではなく更新されたパッケージが、*HostingStartupApp* によって確実に受信されるように、ローカルの NuGet パッケージ キャッシュをクリアします。 ローカルの NuGet キャッシュをクリアするには、次の [dotnet nuget locals](/dotnet/core/tools/dotnet-nuget-locals) コマンドを実行します。

```dotnetcli
dotnet nuget locals all --clear
```

**クラス ライブラリからのアクティブ化**

1. [dotnet build](/dotnet/core/tools/dotnet-build) コマンドを使用して *HostingStartupLibrary* クラス ライブラリをコンパイルします。
1. *HostingStartupLibrary* のクラス ライブラリのアセンブリ名を `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES` 環境変数に追加します。
1. クラス ライブラリのアセンブリをアプリに *bin* 配置するには、クラス ライブラリのコンパイルされた出力からアプリの *bin/Debug* フォルダーに *HostingStartupLibrary.dll* ファイルをコピーします。
1. アプリをコンパイルして実行します。 アプリのプロジェクト ファイル内の `<ItemGroup>` により、クラス ライブラリのアセンブリ ( *.\bin\Debug\netcoreapp3.0\HostingStartupLibrary.dll*) が参照されます (コンパイル時参照)。 詳細については、HostingStartupApp のプロジェクト ファイル内の注を参照してください。

   ```xml
   <ItemGroup>
     <Reference Include=".\\bin\\Debug\\netcoreapp3.0\\HostingStartupLibrary.dll">
       <HintPath>.\bin\Debug\netcoreapp3.0\HostingStartupLibrary.dll</HintPath>
       <SpecificVersion>False</SpecificVersion> 
     </Reference>
   </ItemGroup>
   ```

1. インデックス ページで表示されるサービス構成キー値が、クラス ライブラリの `ServiceKeyInjection.Configure` メソッドによって設定された値と一致することを確認します。

**ランタイム ストア配置アセンブリからのアクティブ化**

1. *StartupDiagnostics* プロジェクトでは、[PowerShell](/powershell/scripting/powershell-scripting) を使用して、その *StartupDiagnostics.deps.json* ファイルを変更します。 既定では、PowerShell は Windows 7 SP1 と Windows Server 2008 R2 SP1 以降の Windows 上にインストールされます。 他のプラットフォームで PowerShell を取得するには、「[PowerShell のさまざまなバージョンのインストール](/powershell/scripting/install/installing-powershell)」を参照してください。
1. *RuntimeStore* フォルダーにある *build.ps1* スクリプトを実行します。 スクリプトでは次のことが行われます。
   * *obj\packages* フォルダーに `StartupDiagnostics` パッケージが生成されます。
   * *store* フォルダー内に `StartupDiagnostics` 用のランタイム ストアが生成されます。 スクリプト内の `dotnet store` コマンドでは、Windows に展開されているホスティング スタートアップの `win7-x64`[ランタイム識別子 (RID)](/dotnet/core/rid-catalog) が使用されます。 別のランタイムに対してホスティング スタートアップを指定する場合は、スクリプトの 37 行目を適切な RID に置き換えます。 `StartupDiagnostics` のランタイム ストアは、後で、アセンブリが使用されるコンピューター上のユーザーまたはシステムのランタイム ストアに移動されます。 `StartupDiagnostics` アセンブリのユーザー ランタイム ストアのインストール場所は、 *.dotnet/store/x64/netcoreapp3.0/startupdiagnostics/1.0.0/lib/netcoreapp3.0/StartupDiagnostics.dll* です。
   * *additionalDeps* フォルダーに `StartupDiagnostics` の `additionalDeps` が生成されます。 追加の依存関係は、後でユーザーまたはシステムの追加の依存関係に移動されます。 ユーザーの `StartupDiagnostics` に関する追加の依存関係のインストール場所は、 *.dotnet/x64/additionalDeps/StartupDiagnostics/shared/Microsoft.NETCore.App/3.0.0/StartupDiagnostics.deps.json* です。
   * *deploy.ps1* ファイルが *deployment* フォルダーに配置されます。
1. *deployment* フォルダーにある *deploy.ps1* スクリプトを実行します。 スクリプトでは以下のものが追加されます。
   * `StartupDiagnostics` が `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES` 環境変数に。
   * (RuntimeStore プロジェクトの *deployment*  フォルダーにある) ホスティング スタートアップの依存関係のパスが、`DOTNET_ADDITIONAL_DEPS` 環境変数に。
   * (RuntimeStore プロジェクトの *deployment*  フォルダーにある) ランタイム ストアのパスが、`DOTNET_SHARED_STORE` 環境変数に。
1. サンプル アプリを実行します。
1. `/services` エンドポイントを要求して、アプリの登録サービスを参照します。 `/diag` エンドポイントを要求して、診断情報を参照します。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<xref:Microsoft.AspNetCore.Hosting.IHostingStartup> (ホスティング スタートアップ) の実装を使うと、外部アセンブリからの起動時にアプリに拡張機能を追加できます。 たとえば、外部ライブラリではホスティング スタートアップ実装を使用して、追加の構成プロバイダーまたはサービスをアプリに提供できます。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/platform-specific-configuration/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="hostingstartup-attribute"></a>HostingStartup 属性

[HostingStartup](xref:Microsoft.AspNetCore.Hosting.HostingStartupAttribute) 属性は、実行時にアクティブ化されるホスティング スタートアップ アセンブリが存在することを示しています。

入力アセンブリまたは `Startup` クラスを含むアセンブリは、`HostingStartup` 属性について自動的にスキャンされます。 `HostingStartup` 属性を検索するアセンブリの一覧は、[WebHostDefaults.HostingStartupAssembliesKey](xref:Microsoft.AspNetCore.Hosting.WebHostDefaults.HostingStartupAssembliesKey) 内の構成からランタイム時に読み込まれます。 検出から除外されるアセンブリの一覧は、[WebHostDefaults.HostingStartupExcludeAssembliesKey](xref:Microsoft.AspNetCore.Hosting.WebHostDefaults.HostingStartupExcludeAssembliesKey) から読み込まれます。 詳細については、「[Web ホスト: ホスティング スタートアップ アセンブリ](xref:fundamentals/host/web-host#hosting-startup-assemblies)と[Web ホスト: 除外するホスティング スタートアップ アセンブリ](xref:fundamentals/host/web-host#hosting-startup-exclude-assemblies)

次に例では、ホスティング スタートアップ アセンブリの名前空間は `StartupEnhancement` となります。 ホスティング スタートアップ コードを含むクラスは `StartupEnhancementHostingStartup` です。

[!code-csharp[](platform-specific-configuration/samples-snapshot/2.x/StartupEnhancement.cs?name=snippet1)]

`HostingStartup` 属性は、通常、ホスティング スタートアップ アセンブリの `IHostingStartup` 実装クラス ファイル内に置かれます。

## <a name="discover-loaded-hosting-startup-assemblies"></a>読み込まれたホスティング スタートアップ アセンブリを見つける

読み込まれたホスティング スタートアップ アセンブリを検出するには、ログ記録を有効にしてアプリのログを確認します。 アセンブリの読み込み時に発生したエラーがログ記録されます。 読み込まれたホスティング スタートアップ アセンブリは、デバッグ レベルでログ記録され、すべてのエラーが記録されます。

## <a name="disable-automatic-loading-of-hosting-startup-assemblies"></a>ホスティング スタートアップ アセンブリの自動読み込みを無効にする

ホスティング スタートアップ アセンブリの自動読み込みを無効にするには、次の方法のいずれかを使用します。

* すべてのホスティング スタートアップ アセンブリの読み込みを回避するには、次のいずれかを `true` または `1` に設定します。
  * [[ホスティング スタートアップを回避する]](xref:fundamentals/host/web-host#prevent-hosting-startup) ホスト構成設定。
  * `ASPNETCORE_PREVENTHOSTINGSTARTUP` 環境変数。
* 特定のホスティング スタートアップ アセンブリの読み込みを回避するには、次に示すいずれかを、起動時に除外するホスティング スタートアップ アセンブリを示すセミコロンで区切られた文字列に設定します。
  * [[Hosting Startup Exclude Assemblies]\(除外するホスティング スタートアップ アセンブリ\)](xref:fundamentals/host/web-host#hosting-startup-exclude-assemblies) ホスト構成設定。
  * `ASPNETCORE_HOSTINGSTARTUPEXCLUDEASSEMBLIES` 環境変数。

ホスト構成設定と環境変数が両方とも設定されている場合は、ホスト設定によって動作が制御されます。

ホスト設定または環境変数を使用してホスティング スタートアップ アセンブリを無効にすると、アセンブリがグローバルに無効になり、場合によってはアプリのいくつかの属性も無効になります。

## <a name="project"></a>Project

次のいずれかの種類のプロジェクトを使用して、ホスティング スタートアップを作成します。

* [クラス ライブラリ](#class-library)
* [エントリ ポイントのないコンソール アプリ](#console-app-without-an-entry-point)

### <a name="class-library"></a>クラス ライブラリ

クラス ライブラリでは、ホスティング スタートアップの拡張機能を提供できます。 このライブラリには `HostingStartup` 属性が含まれています。

[サンプル コード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/platform-specific-configuration/samples/)には、Razor Pages アプリ、*HostingStartupApp*、クラス ライブラリ、*HostingStartupLibrary* が含まれています。 クラス ライブラリには次のものが含まれています。

* `IHostingStartup` を実装するホスティング スタートアップ クラス `ServiceKeyInjection`。 `ServiceKeyInjection` では、メモリ内の構成プロバイダー ([AddInMemoryCollection](xref:Microsoft.Extensions.Configuration.MemoryConfigurationBuilderExtensions.AddInMemoryCollection*)) を使用して、サービスの文字列のペアがアプリの構成に追加されます。
* ホスティング スタートアップの名前空間とクラスを識別する `HostingStartup` 属性。

`ServiceKeyInjection` クラスの <xref:Microsoft.AspNetCore.Hosting.IHostingStartup.Configure*> メソッドでは、<xref:Microsoft.AspNetCore.Hosting.IWebHostBuilder> を使ってアプリに拡張機能を追加します。

*HostingStartupLibrary/ServiceKeyInjection.cs*:

[!code-csharp[](platform-specific-configuration/samples/2.x/HostingStartupLibrary/ServiceKeyInjection.cs?name=snippet1)]

アプリのインデックス ページでは、クラス ライブラリのホスティング スタートアップ アセンブリによって設定された 2 つのキーの構成値が読み取られて表示されます。

*HostingStartupApp/Pages/Index.cshtml.cs*:

[!code-csharp[](platform-specific-configuration/samples/2.x/HostingStartupApp/Pages/Index.cshtml.cs?name=snippet1&highlight=5-6,11-12)]

[サンプル コード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/platform-specific-configuration/samples/) には、別のホスティング スタートアップ *HostingStartupPackage* を提供する NuGet パッケージ プロジェクトも含まれています。 パッケージには、前に説明したクラス ライブラリと同じ特性があります。 パッケージには次のものが含まれます。

* `IHostingStartup` を実装するホスティング スタートアップ クラス `ServiceKeyInjection`。 `ServiceKeyInjection` では、アプリの構成にサービス文字列のペアが追加されます。
* `HostingStartup`属性。

*HostingStartupPackage/ServiceKeyInjection.cs*:

[!code-csharp[](platform-specific-configuration/samples/2.x/HostingStartupPackage/ServiceKeyInjection.cs?name=snippet1)]

アプリのインデックス ページでは、パッケージのホスティング スタートアップ アセンブリによって設定された 2 つのキーの構成値が読み取られて表示されます。

*HostingStartupApp/Pages/Index.cshtml.cs*:

[!code-csharp[](platform-specific-configuration/samples/2.x/HostingStartupApp/Pages/Index.cshtml.cs?name=snippet1&highlight=7-8,13-14)]

### <a name="console-app-without-an-entry-point"></a>エントリ ポイントのないコンソール アプリ

*このアプローチは、.NET Core アプリでのみ利用でき、.NET Framework では利用できません。*

`HostingStartup` 属性を含むエントリ ポイントがないコンソール アプリ内では、アクティブ化に関するコンパイル時参照を必要としない動的ホスティング スタートアップ拡張機能を指定できます。 コンソール アプリを発行すると、ランタイム ストアから使用できるホスティング スタートアップ アセンブリが生成されます。

このプロセスにおいてエントリ ポイントのないコンソール アプリが使用される理由は、次のとおりです。

* ホスティング スタートアップ アセンブリ内のホスティング スタートアップを使用するには、依存関係ファイルが必要です。 依存関係ファイルは、ライブラリではなくアプリを発行することによって生成される、実行可能なアプリ資産です。
* [ランタイム パッケージ ストア](/dotnet/core/deploying/runtime-store) にライブラリを直接追加することはできません。それには、共有ランタイムを対象とする実行可能なプロジェクトが必要です。

動的ホスティング スタートアップを作成する場合:

* 次のようなエントリ ポイントがないコンソール アプリからホスティング スタートアップ アセンブリが作成されます。
  * `IHostingStartup` の実装を含むクラスを含んでいる。
  * `IHostingStartup` 実装クラスを識別するための [HostingStartup](xref:Microsoft.AspNetCore.Hosting.HostingStartupAttribute) 属性を含んでいる。
* ホスティング スタートアップの依存関係を取得するには、コンソール アプリを発行します。 コンソール アプリを発行すると、その結果として、依存関係ファイルから未使用の依存関係が切り捨てられます。
* ホスティング スタートアップ アセンブリのランタイムの場所を設定するために、依存関係ファイルが変更されます。
* ホスティング スタートアップ アセンブリとその依存関係ファイルは、ランタイム パッケージ ストアに配置されます。 ホスティング スタートアップ アセンブリとその依存関係ファイルを検出する場合、それらは環境変数のペアに記載されています。

コンソール アセンブリでは、[Microsoft.AspNetCore.Hosting.Abstractions](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.Abstractions/) パッケージが参照されます。

[!code-xml[](platform-specific-configuration/samples-snapshot/2.x/StartupEnhancement.csproj)]

[HostingStartup](xref:Microsoft.AspNetCore.Hosting.HostingStartupAttribute) 属性を使うと、<xref:Microsoft.AspNetCore.Hosting.IWebHost> のビルド時に、読み込みと実行のための `IHostingStartup` の実装としてクラスが識別されます。 次の例では、名前空間は `StartupEnhancement`、クラスは `StartupEnhancementHostingStartup` です。

[!code-csharp[](platform-specific-configuration/samples-snapshot/2.x/StartupEnhancement.cs?name=snippet1)]

クラスは `IHostingStartup` を実装しています。 このクラスの <xref:Microsoft.AspNetCore.Hosting.IHostingStartup.Configure*> メソッドでは、<xref:Microsoft.AspNetCore.Hosting.IWebHostBuilder> を使ってアプリに拡張機能を追加します。 ホスティング スタートアップ アセンブリ内の `IHostingStartup.Configure` は、ユーザー コード内の `Startup.Configure` よりも前にランタイムによって呼び出されます。このため、ホスティング スタートアップ アセンブリによって提供されるすべての構成をユーザー コードで上書きすることができます。

[!code-csharp[](platform-specific-configuration/samples-snapshot/2.x/StartupEnhancement.cs?name=snippet2&highlight=3,5)]

`IHostingStartup` プロジェクトのビルド時に、依存関係ファイル ( *.deps.json*) によってアセンブリの `runtime` の場所が *bin* フォルダーに設定されます。

[!code-json[](platform-specific-configuration/samples-snapshot/2.x/StartupEnhancement1.deps.json?range=2-13&highlight=8)]

ファイルの一部のみが示されています。 例のアセンブリ名は `StartupEnhancement` です。

## <a name="configuration-provided-by-the-hosting-startup"></a>ホスティング スタートアップによって指定される構成

ホスティング スタートアップの構成、アプリの構成のどちらを優先させるかによって、2 つの異なる方法で構成を処理できます。

1. アプリの <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureAppConfiguration*> デリゲートを実行した後で <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureAppConfiguration*> を使って構成を読み込み、アプリに構成を指定します。 この方法を使うと、ホスティング スタートアップの構成の方がアプリの構成よりも優先されます。
1. アプリの <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureAppConfiguration*> デリゲートを実行する前に <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseConfiguration*> を使って構成を読み込み、アプリに構成を指定します。 この方法を使うと、アプリの構成値の方がホスティング スタートアップにより指定されるものよりも優先されます。

```csharp
public class ConfigurationInjection : IHostingStartup
{
    public void Configure(IWebHostBuilder builder)
    {
        Dictionary<string, string> dict;

        builder.ConfigureAppConfiguration(config =>
        {
            dict = new Dictionary<string, string>
            {
                {"ConfigurationKey1", 
                    "From IHostingStartup: Higher priority " +
                    "than the app's configuration."},
            };

            config.AddInMemoryCollection(dict);
        });

        dict = new Dictionary<string, string>
        {
            {"ConfigurationKey2", 
                "From IHostingStartup: Lower priority " +
                "than the app's configuration."},
        };

        var builtConfig = new ConfigurationBuilder()
            .AddInMemoryCollection(dict)
            .Build();

        builder.UseConfiguration(builtConfig);
    }
}
```

## <a name="specify-the-hosting-startup-assembly"></a>ホスティング スタートアップ アセンブリを指定する

クラス ライブラリまたはコンソール アプリのいずれかで提供されるホスティング スタートアップの場合は、`ASPNETCORE_HOSTINGSTARTUPASSEMBLIES` 環境変数内にホスティング スタートアップ アセンブリの名前を指定します。 環境変数はアセンブリのセミコロン区切りのリストです。

ホスティング スタートアップ アセンブリのみが、`HostingStartup` 属性に対してスキャンされます。 サンプル アプリ *HostingStartupApp* では、前述したホスティング スタートアップを検出するために、環境変数が次の値に設定されています。

```
HostingStartupLibrary;HostingStartupPackage;StartupDiagnostics
```

ホスティング スタートアップ アセンブリはまた、[[ホスティング スタートアップ アセンブリ]](xref:fundamentals/host/web-host#hosting-startup-assemblies) ホスト構成設定を使用して設定することもできます。

複数のホスティング スタートアップ アセンブリが存在する場合、それらの <xref:Microsoft.AspNetCore.Hosting.IHostingStartup.Configure*> メソッドは、アセンブリが一覧表示されている順序で実行されます。

## <a name="activation"></a>アクティベーション

ホスティング スタートアップのアクティブ化のオプションは次のとおりです。

* [ランタイム ストア](#runtime-store):アクティブ化では、アクティブ化に関するコンパイル時参照を必要としません。 サンプル アプリでは、ホスティング スタートアップ アセンブリおよび依存関係のファイルが *deployment* フォルダーに配置されています。これにより、複数のコンピューターから成る環境でのホスティング スタートアップの配置が容易になります。 *deployment* フォルダーには、ホスティングスタートアップが有効になるように配置システム上で環境変数を作成および変更する PowerShell スクリプトも含まれています。
* アクティブ化で必須のコンパイル時参照
  * [NuGet パッケージ](#nuget-package)
  * [プロジェクトの bin フォルダー](#project-bin-folder)

### <a name="runtime-store"></a>ランタイム ストア

ホスティング スタートアップ実装は、[ランタイム ストア](/dotnet/core/deploying/runtime-store)に置かれます。 アセンブリへのコンパイル時参照は、機能強化されたアプリで必須ではありません。

ホスティング スタートアップをビルドした後、プロジェクトのマニフェスト ファイルと [dotnet store](/dotnet/core/tools/dotnet-store) コマンドの使用によってランタイム ストアが生成されます。

```dotnetcli
dotnet store --manifest {MANIFEST FILE} --runtime {RUNTIME IDENTIFIER} --output {OUTPUT LOCATION} --skip-optimization
```

サンプル アプリ (*RuntimeStore* プロジェクト) では、次のコマンドを使用します。

```dotnetcli
dotnet store --manifest store.manifest.csproj --runtime win7-x64 --output ./deployment/store --skip-optimization
```

ランタイムでランタイム ストアを検出できるように、ランタイム ストアの場所を環境変数 `DOTNET_SHARED_STORE` に追加します。

**ホスティング スタートアップの依存関係ファイルを変更して配置する**

拡張機能へのパッケージ参照なしで拡張機能をアクティブ化するには、`additionalDeps` を使用してランタイムに追加の依存関係を指定します。 `additionalDeps` を使用すると、次のことを実行できます。

* スタートアップ時にアプリの独自の *.deps.json* ファイルとマージさせる追加の *.deps.json* ファイルのセットを指定することで、アプリのライブラリのグラフを拡張します。
* ホスティング スタートアップ アセンブリを検出可能および読み込み可能にします。

追加の依存関係ファイルを生成するために推奨される方法は次のとおりです。

 1. 前のセクションで参照したランタイム ストアのマニフェスト ファイルに対して `dotnet publish` を実行します。
 1. ライブラリと、結果として得られる *deps.json* ファイルの `runtime` セクションから、マニフェスト参照を削除します。

プロジェクトの例では、`targets` と `libraries` セクションから `store.manifest/1.0.0` プロパティが削除されます。

```json
{
  "runtimeTarget": {
    "name": ".NETCoreApp,Version=v2.1",
    "signature": "4ea77c7b75ad1895ae1ea65e6ba2399010514f99"
  },
  "compilationOptions": {},
  "targets": {
    ".NETCoreApp,Version=v2.1": {
      "store.manifest/1.0.0": {
        "dependencies": {
          "StartupDiagnostics": "1.0.0"
        },
        "runtime": {
          "store.manifest.dll": {}
        }
      },
      "StartupDiagnostics/1.0.0": {
        "runtime": {
          "lib/netcoreapp2.1/StartupDiagnostics.dll": {
            "assemblyVersion": "1.0.0.0",
            "fileVersion": "1.0.0.0"
          }
        }
      }
    }
  },
  "libraries": {
    "store.manifest/1.0.0": {
      "type": "project",
      "serviceable": false,
      "sha512": ""
    },
    "StartupDiagnostics/1.0.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-oiQr60vBQW7+nBTmgKLSldj06WNLRTdhOZpAdEbCuapoZ+M2DJH2uQbRLvFT8EGAAv4TAKzNtcztpx5YOgBXQQ==",
      "path": "startupdiagnostics/1.0.0",
      "hashPath": "startupdiagnostics.1.0.0.nupkg.sha512"
    }
  }
}
```

次の場所に *.deps.json* ファイルを配置します。

```
{ADDITIONAL DEPENDENCIES PATH}/shared/{SHARED FRAMEWORK NAME}/{SHARED FRAMEWORK VERSION}/{ENHANCEMENT ASSEMBLY NAME}.deps.json
```

* `{ADDITIONAL DEPENDENCIES PATH}`:`DOTNET_ADDITIONAL_DEPS` 環境変数に追加される場所。
* `{SHARED FRAMEWORK NAME}`:この追加の依存関係ファイルに必要な共有フレームワーク。
* `{SHARED FRAMEWORK VERSION}`:共有フレームワークの最小バージョン。
* `{ENHANCEMENT ASSEMBLY NAME}`:拡張機能のアセンブリ名。

サンプル アプリ (*RuntimeStore* プロジェクト) では、追加の依存関係ファイルは以下の場所に配置されます。

```
deployment/additionalDeps/shared/Microsoft.AspNetCore.App/2.1.0/StartupDiagnostics.deps.json
```

ランタイムでランタイム ストアの場所を検出できるように、追加の依存関係ファイルの場所が環境変数 `DOTNET_ADDITIONAL_DEPS` に追加されます。

サンプル アプリ (*RuntimeStore* プロジェクト) では、[PowerShell](/powershell/scripting/powershell-scripting) スクリプトを使用してランタイム ストアのビルドと追加の依存関係ファイルの生成を行います。

さまざまなオペレーティング システムの環境変数を設定する方法の例については、[複数の環境の使用](xref:fundamentals/environments)に関するページを参照してください。

**配置**

サンプル アプリでは、複数のコンピューターから成る環境へのホスティング スタートアップの配置を容易にするために、発行された出力内に、次を含む *deployment* フォルダーが作成されます。

* ホスティング スタートアップのランタイム ストア。
* ホスティング スタートアップの依存関係ファイル。
* ホスティング スタートアップのアクティブ化をサポートできるように、`ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`、`DOTNET_SHARED_STORE`、`DOTNET_ADDITIONAL_DEPS` を作成および変更する PowerShell スクリプト。 このスクリプトは、配置システム上の PowerShell 管理用コマンド プロンプトから実行します。

### <a name="nuget-package"></a>NuGet パッケージ

NuGet パッケージでは、ホスティング スタートアップの拡張機能を提供できます。 このパッケージには `HostingStartup` 属性があります。 このパッケージで提供される種類のホスティング スタートアップは、次のアプローチのいずれかを使用してアプリで利用できるようになります。

* 機能強化されたアプリのプロジェクト ファイルでは、アプリのプロジェクト ファイル内のホスティング スタートアップに対するパッケージ参照 (コンパイル時参照) が作成されます。 コンパイル時参照を適用すると、ホスティング スタートアップ アセンブリとその依存関係のすべてが、アプリの依存関係ファイル ( *.deps.json*) に組み込まれます。 このアプローチは、[nuget.org](https://www.nuget.org/) に発行されるホスティング スタートアップ アセンブリ パッケージに適用されます。
* ホスティング スタートアップの依存関係ファイルは、「[ランタイム ストア](#runtime-store)」のセクションで説明にしたように (コンパイル参照がない場合)、機能強化されたアプリで利用できるようになります。

NuGet パッケージとランタイム ストアの詳細については、次のトピックを参照してください。

* [クロスプラットフォーム ツールを使用して NuGet パッケージを作成する方法](/dotnet/core/deploying/creating-nuget-packages)
* [パッケージを公開する](/nuget/create-packages/publish-a-package)
* [ランタイム パッケージ ストア](/dotnet/core/deploying/runtime-store)

### <a name="project-bin-folder"></a>プロジェクトの bin フォルダー

ホスティング スタートアップの拡張機能は、機能強化されたアプリ内の *bin* 配置アセンブリによって提供できます。 このアセンブリで提供される種類のホスティング スタートアップは、次のアプローチのいずれかを使用してアプリで利用できるようになります。

* 機能強化されたアプリのプロジェクト ファイルでは、ホスティング スタートアップへのアセンブリ参照 (コンパイル時参照) が作成されます。 コンパイル時参照を適用すると、ホスティング スタートアップ アセンブリとその依存関係のすべてが、アプリの依存関係ファイル ( *.deps.json*) に組み込まれます。 この手法は、展開シナリオによって、ホスティング スタートアップのアセンブリ ( *.dll* ファイル) に対してコンパイル時参照を作成し、次のいずれかにそのアセンブリを移動させることが要請される場合に適用されます。
  * 使用するプロジェクト。
  * 使用するプロジェクトでアクセスできる場所。
* ホスティング スタートアップの依存関係ファイルは、「[ランタイム ストア](#runtime-store)」のセクションで説明にしたように (コンパイル参照がない場合)、機能強化されたアプリで利用できるようになります。
* .NET Framework を対象にするとき、アセンブリは既定の読み込みコンテキストで読み込み可能です。これは、.NET Framework の場合、アセンブリが次のいずれかの場所にあることを意味します。
  * アプリケーション ベース パス:アプリの実行可能ファイル ( *.exe*) が置かれている *bin* フォルダー。
  * グローバル アセンブリ キャッシュ (GAC):GAC には、複数の .NET Framework アプリで共有されるアセンブリが格納されています。 詳細については、「[方法: アセンブリをグローバル アセンブリ キャッシュにインストールする](/dotnet/framework/app-domains/how-to-install-an-assembly-into-the-gac)」を参照してください。これは .NET Framework ドキュメントにあります。

## <a name="sample-code"></a>サンプル コード

[サンプル コード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/platform-specific-configuration/samples/) ([ダウンロードする方法](xref:index#how-to-download-a-sample)) では、ホスティング スタートアップ実装のシナリオを示します。

* 2 つのホスティング スタートアップ アセンブリ (クラス ライブラリ) ではそれぞれ、メモリ内の構成のキーと値のペアの組が設定されます。
  * NuGet パッケージ (*HostingStartupPackage*)
  * クラス ライブラリ (*HostingStartupLibrary*)
* ホスティング スタートアップは、ランタイム ストア配置アセンブリ (*StartupDiagnostics*) からアクティブ化されます。 このアセンブリによってスタートアップ時に 2 つのミドルウェアがアプリに追加され、これにより診断情報が提供されます。
  * 登録サービス
  * アドレス (スキーム、ホスト、パス ベース、パス、クエリ文字列)
  * 接続 (リモート IP、リモート ポート、ローカル IP、ローカル ポート、クライアント証明書)
  * 要求ヘッダー
  * 環境変数

サンプルを実行するには

**NuGet パッケージからのアクティブ化**

1. [dotnet pack](/dotnet/core/tools/dotnet-pack) コマンドを使用して *HostingStartupPackage* パッケージをコンパイルします。
1. *HostingStartupPackage* のパッケージのアセンブリ名を `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES` 環境変数に追加します。
1. アプリをコンパイルして実行します。 機能拡張されたアプリにはパッケージ参照 (コンパイル時参照) が存在します。 アプリのプロジェクト ファイル内の `<PropertyGroup>` では、パッケージ プロジェクトの出力 ( *../HostingStartupPackage/Bin/debug*) がパッケージ ソースとして指定されます。 これにより、[nuget.org](https://www.nuget.org/) にパッケージをアップロードすることなく、アプリによりパッケージが使用されるようになります。詳細については、HostingStartupApp のプロジェクト ファイル内の注を参照してください。

   ```xml
   <PropertyGroup>
     <RestoreSources>$(RestoreSources);https://api.nuget.org/v3/index.json;../HostingStartupPackage/bin/Debug</RestoreSources>
   </PropertyGroup>
   ```

1. インデックス ページで表示されるサービス構成キー値が、パッケージの `ServiceKeyInjection.Configure` メソッドによって設定された値と一致することを確認します。

*HostingStartupPackage* プロジェクトに変更を加えてそれをコンパイルする場合は、ローカル キャッシュから、古いパッケージではなく更新されたパッケージが、*HostingStartupApp* によって確実に受信されるように、ローカルの NuGet パッケージ キャッシュをクリアします。 ローカルの NuGet キャッシュをクリアするには、次の [dotnet nuget locals](/dotnet/core/tools/dotnet-nuget-locals) コマンドを実行します。

```dotnetcli
dotnet nuget locals all --clear
```

**クラス ライブラリからのアクティブ化**

1. [dotnet build](/dotnet/core/tools/dotnet-build) コマンドを使用して *HostingStartupLibrary* クラス ライブラリをコンパイルします。
1. *HostingStartupLibrary* のクラス ライブラリのアセンブリ名を `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES` 環境変数に追加します。
1. クラス ライブラリのアセンブリをアプリに *bin* 配置するには、クラス ライブラリのコンパイルされた出力からアプリの *bin/Debug* フォルダーに *HostingStartupLibrary.dll* ファイルをコピーします。
1. アプリをコンパイルして実行します。 アプリのプロジェクト ファイル内の `<ItemGroup>` は、クラス ライブラリのアセンブリ ( *.\bin\Debug\netcoreapp2.1\HostingStartupLibrary.dll*) を参照します (コンパイル時参照)。 詳細については、HostingStartupApp のプロジェクト ファイル内の注を参照してください。

   ```xml
   <ItemGroup>
     <Reference Include=".\\bin\\Debug\\netcoreapp2.1\\HostingStartupLibrary.dll">
       <HintPath>.\bin\Debug\netcoreapp2.1\HostingStartupLibrary.dll</HintPath>
       <SpecificVersion>False</SpecificVersion>
     </Reference>
   </ItemGroup>
   ```

1. インデックス ページで表示されるサービス構成キー値が、クラス ライブラリの `ServiceKeyInjection.Configure` メソッドによって設定された値と一致することを確認します。

**ランタイム ストア配置アセンブリからのアクティブ化**

1. *StartupDiagnostics* プロジェクトでは、[PowerShell](/powershell/scripting/powershell-scripting) を使用して、その *StartupDiagnostics.deps.json* ファイルを変更します。 既定では、PowerShell は Windows 7 SP1 と Windows Server 2008 R2 SP1 以降の Windows 上にインストールされます。 他のプラットフォームで PowerShell を取得するには、「[PowerShell のさまざまなバージョンのインストール](/powershell/scripting/install/installing-powershell)」を参照してください。
1. *RuntimeStore* フォルダーにある *build.ps1* スクリプトを実行します。 スクリプトでは次のことが行われます。
   * *obj\packages* フォルダーに `StartupDiagnostics` パッケージが生成されます。
   * *store* フォルダー内に `StartupDiagnostics` 用のランタイム ストアが生成されます。 スクリプト内の `dotnet store` コマンドでは、Windows に展開されているホスティング スタートアップの `win7-x64`[ランタイム識別子 (RID)](/dotnet/core/rid-catalog) が使用されます。 別のランタイムに対してホスティング スタートアップを指定する場合は、スクリプトの 37 行目を適切な RID に置き換えます。 `StartupDiagnostics` のランタイム ストアは、後で、アセンブリが使用されるコンピューター上のユーザーまたはシステムのランタイム ストアに移動されます。 `StartupDiagnostics` アセンブリのユーザー ランタイム ストアのインストール場所は、 *.dotnet/store/x64/netcoreapp2.2/startupdiagnostics/1.0.0/lib/netcoreapp2.2/StartupDiagnostics.dll* です。
   * *additionalDeps* フォルダーに `StartupDiagnostics` の `additionalDeps` が生成されます。 追加の依存関係は、後でユーザーまたはシステムの追加の依存関係に移動されます。 ユーザーの `StartupDiagnostics` に関する追加の依存関係のインストール場所は、 *.dotnet/x64/additionalDeps/StartupDiagnostics/shared/Microsoft.NETCore.App/2.2.0/StartupDiagnostics.deps.json* です。
   * *deploy.ps1* ファイルが *deployment* フォルダーに配置されます。
1. *deployment* フォルダーにある *deploy.ps1* スクリプトを実行します。 スクリプトでは以下のものが追加されます。
   * `StartupDiagnostics` が `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES` 環境変数に。
   * (RuntimeStore プロジェクトの *deployment*  フォルダーにある) ホスティング スタートアップの依存関係のパスが、`DOTNET_ADDITIONAL_DEPS` 環境変数に。
   * (RuntimeStore プロジェクトの *deployment*  フォルダーにある) ランタイム ストアのパスが、`DOTNET_SHARED_STORE` 環境変数に。
1. サンプル アプリを実行します。
1. `/services` エンドポイントを要求して、アプリの登録サービスを参照します。 `/diag` エンドポイントを要求して、診断情報を参照します。

::: moniker-end
