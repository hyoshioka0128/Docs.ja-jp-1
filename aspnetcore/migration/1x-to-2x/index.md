---
title: ASP.NET Core 1.x から 2.0 への移行
author: rick-anderson
description: この記事では、ASP.NET Core 1.x プロジェクトを ASP.NET Core 2.0 に移行する前提条件と最も一般的な手順について説明します。
ms.author: scaddie
ms.custom: mvc
ms.date: 12/05/2019
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
uid: migration/1x-to-2x/index
ms.openlocfilehash: 9a175c8a3cd960404006aad559b2ca65b5a8c1e1
ms.sourcegitcommit: 0abfe496fed8e9470037c8128efa8a50069ccd52
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/07/2021
ms.locfileid: "106564227"
---
# <a name="migrate-from-aspnet-core-1x-to-20"></a>ASP.NET Core 1.x から 2.0 への移行

作成者: [Scott Addie](https://github.com/scottaddie)

この記事では、既存の ASP.NET Core 1.x プロジェクトを ASP.NET Core 2.0 に更新する手順を説明します。 アプリケーションを ASP.NET Core 2.0 に移行すると、[多数の新機能を利用したり、パフォーマンスを向上させたりすることができます](xref:aspnetcore-2.0)。

既存の ASP.NET Core 1.x アプリケーションは、バージョン固有のプロジェクト テンプレートには基づいていません。 ASP.NET Core のフレームワークの進化に伴い、プロジェクト テンプレートと、それに含まれるスタート コードも進化しています。 ASP.NET Core のフレームワークを更新するだけでなく、アプリケーションのコードも更新する必要があります。

<a name="prerequisites"></a>

## <a name="prerequisites"></a>必須コンポーネント

「[ASP.NET Core の概要](xref:getting-started)」を参照してください。

<a name="tfm"></a>

## <a name="update-target-framework-moniker-tfm"></a>ターゲット フレームワーク モニカー (TFM) の更新

.NET Core をターゲットとするプロジェクトでは、.NET Core 2.0 以上のバージョンの [TFM](/dotnet/standard/frameworks) を使用する必要があります。 *.csproj* ファイルで `<TargetFramework>` ノードを探し、その内側のテキストを `netcoreapp2.0` に置き換えます。

[!code-xml[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App.csproj?range=3)]

.NET Framework をターゲットとするプロジェクトでは、.NET Framework 4.6.1 以上の TFM バージョンを使用する必要があります。 *.csproj* ファイルで `<TargetFramework>` ノードを探し、その内側のテキストを `net461` に置き換えます。

[!code-xml[](../1x-to-2x/samples/AspNetCoreDotNetFx2.0App/AspNetCoreDotNetFx2.0App/AspNetCoreDotNetFx2.0App.csproj?range=4)]

> [!NOTE]
> .NET Core 1.x よりセキュリティ上外部からアクセスできるところの多い .NET core 2.0 NET Core 1.x に API がないためだけに .NET Framework をターゲットにする場合、.NET Core 2.0 をターゲットにすると機能します。

プロジェクト ファイルに `<RuntimeFrameworkVersion>1.{sub-version}</RuntimeFrameworkVersion>` が含まれている場合は、[こちらの GitHub の問題](https://github.com/dotnet/AspNetCore/issues/3221#issuecomment-413094268)を参照してください。

<a name="global-json"></a>

## <a name="update-net-core-sdk-version-in-globaljson"></a>global.json での .NET Core SDK バージョンの更新

使用するソリューションが [global.json](/dotnet/core/tools/global-json) ファイルに依存して特定の .NET Core SDK バージョンをターゲットにしている場合、その `version` プロパティを更新して、コンピューターにインストールされている 2.0 バージョンが使用されるようにします。

[!code-json[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/global.json?highlight=3)]

<a name="package-reference"></a>

## <a name="update-package-references"></a>パッケージ参照の更新

1\.x プロジェクト内の *.csproj* ファイルには、プロジェクトによって使用される各 NuGet パッケージの一覧があります。

.NET Core 2.0 をターゲットとする ASP.NET Core 2.0 プロジェクトでは、 *.csproj* ファイル内の 1 つの [メタパッケージ](xref:fundamentals/metapackage)への参照によってパッケージのコレクションが置き換えられます。

[!code-xml[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App.csproj?range=8-10)]

メタパッケージには、ASP.NET Core 2.0 および Entity Framework Core 2.0 のすべての機能が含まれます。

.NET Framework をターゲットとする ASP.NET Core 2.0 プロジェクトは、個々の NuGet パッケージを参照し続ける必要があります。 各 `<PackageReference />` ノードの `Version` 属性を 2.0.0 に更新します。

たとえば、.NET Framework をターゲットとする一般的な ASP.NET Core 2.0 プロジェクトで使用される `<PackageReference />` ノードの一覧を次に示します。

[!code-xml[](../1x-to-2x/samples/AspNetCoreDotNetFx2.0App/AspNetCoreDotNetFx2.0App/AspNetCoreDotNetFx2.0App.csproj?range=9-22)]

<a name="dot-net-cli-tool-reference"></a>

## <a name="update-net-core-cli-tools"></a>.NET Core CLI ツールの更新

*.csproj* ファイルで、`<DotNetCliToolReference />` ノードのそれぞれの `Version` 属性を 2.0.0 に更新します。

たとえば、.NET Core 2.0 をターゲットとする一般的な ASP.NET Core 2.0 プロジェクトで使用される CLI ツールの一覧を次に示します。

[!code-xml[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App.csproj?range=12-16)]

<a name="package-target-fallback"></a>

## <a name="rename-package-target-fallback-property"></a>Package Target Fallback プロパティの名前変更

1\.x プロジェクトの *.csproj* ファイルでは、`PackageTargetFallback` ノードと変数を使用していました。

[!code-xml[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App.csproj?range=5)]

ノードと変数の両方を `AssetTargetFallback` に名前変更します。

[!code-xml[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App.csproj?range=4)]

<a name="program-cs"></a>

## <a name="update-main-method-in-programcs"></a>Program.cs の Main メソッドの更新

1\.x プロジェクトでは、*Program.cs* の `Main` メソッドは次のようでした。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Program.cs?name=snippet_ProgramCs&highlight=8-19)]

2\.0 プロジェクトでは、*Program.cs* の `Main` メソッドは次のように簡素化されました。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Program.cs?highlight=8-11)]

この新しい 2.0 のパターンの導入は強く推奨され、これらの動作には、[Entity Framework (EF) Core Migrations](xref:data/ef-mvc/migrations) のような製品機能が必要です。 たとえば、パッケージ マネージャーのコンソール ウィンドウから `Update-Database` を実行したり、(ASP.NET Core 2.0 に変換されたプロジェクトで) コマンドラインから `dotnet ef database update` を実行したりすると、次のエラーが発生します。

```
Unable to create an object of type '<Context>'. Add an implementation of 'IDesignTimeDbContextFactory<Context>' to the project, or see https://go.microsoft.com/fwlink/?linkid=851728 for additional patterns supported at design time.
```

<a name="add-modify-configuration"></a>

## <a name="add-configuration-providers"></a>構成プロバイダーの追加

1\.x プロジェクトでは、アプリへの構成プロバイダーの追加は `Startup` コンストラクターを使用して実行しました。 この手順には `ConfigurationBuilder` のインスタンスの作成、適用可能なプロバイダー (環境変数、アプリの設定など) の読み込み、`IConfigurationRoot` のメンバーの初期化などが伴いました。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Startup.cs?name=snippet_1xStartup)]

前の例では、 *appsettings.json* と、`IHostingEnvironment.EnvironmentName` プロパティに一致する *appsettings.\<EnvironmentName\>.json* ファイルとから、構成設定を含む `Configuration` メンバーを読み込んでいます。 これらのファイルの場所は *Startup.cs* と同じパスです。

2\.0 プロジェクトでは、1.x プロジェクトに固有の定型句による構成コードがバックグラウンドで実行されていました。 たとえば、環境変数とアプリの設定は起動時に読み込まれます。 同等の *Startup.cs* コードは、挿入されたインスタンスによって `IConfiguration` の初期化に削減されます。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetFx2.0App/AspNetCoreDotNetFx2.0App/Startup.cs?name=snippet_2xStartup)]

`WebHostBuilder.CreateDefaultBuilder` によって追加された既定のプロバイダーを削除するには、`ConfigureAppConfiguration` の内部の `IConfigurationBuilder.Sources` プロパティで `Clear` メソッドを呼び出します。 プロバイダーを戻すには、*Program.cs* の `ConfigureAppConfiguration` メソッドを利用します。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetFx2.0App/AspNetCoreDotNetFx2.0App/Program.cs?name=snippet_ProgramMainConfigProviders&highlight=9-14)]

前のコード スニペットの `CreateDefaultBuilder` メソッドで使用される構成は、[こちら](https://github.com/aspnet/MetaPackages/blob/rel/2.0.0/src/Microsoft.AspNetCore/WebHost.cs#L152)で確認できます。

詳細については、「[ASP.NET Core の構成](xref:fundamentals/configuration/index)」を参照してください。

<a name="db-init-code"></a>

## <a name="move-database-initialization-code"></a>データベース初期化コードの移動

EF Core 1.x を利用する 1.x プロジェクトで、`dotnet ef migrations add` のようなコマンドは以下を行います。

1. `Startup` インスタンスをインスタンス化する
1. `ConfigureServices` メソッドを呼び出し、依存性の注入にすべてのサービスを登録する (`DbContext` タイプを含める)
1. その必要なタスクを実行する

EF Core 2.0 を使用する 2.0 プロジェクトでは、`Program.BuildWebHost` が呼び出され、アプリケーション サービスを取得します。 1\.x とは異なり、`Startup.Configure` を呼び出すという副作用が加わります。 1\.x アプリがその `Configure` メソッドでデータベース初期化コードを呼び出すと、予想外の問題が発生することがあります。 たとえば、データベースがまだ存在しない場合、EF Core 移行コマンド実行前にシード処理コードが実行されます。 この問題が原因で、データベースがまだ存在しない場合に `dotnet ef migrations list` コマンドが失敗します。

*Startup.cs* の `Configure` メソッドで次の 1.x シード処理初期化コードを検討してください。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Startup.cs?name=snippet_ConfigureSeedData&highlight=8)]

2\.0 プロジェクトでは、*Program.cs* の `Main` メソッドに `SeedData.Initialize` 呼び出しを移動します。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Program2.cs?name=snippet_Main2Code&highlight=10)]

2\.0 以降、`BuildWebHost` で Web ホストのビルドと構成以外を行うことは正しくない使用例となります。 アプリケーションの実行に関するあらゆることは `BuildWebHost` &mdash; の外側で、一般的には *Program.cs* の `Main` メソッドで処理する必要があります。

<a name="view-compilation"></a>

## <a name="review-razor-view-compilation-setting"></a>Razor ビュー コンパイル設定の確認

ユーザーに最も重要なことは、アプリケーションを高速に起動することとパブリッシュされたバンドル数を少なくすることです。 これらの理由から、ASP.NET Core 2.0 では [Razor ビュー コンパイル](xref:mvc/views/view-compilation)が既定で有効になっています。

`MvcRazorCompileOnPublish` プロパティを true に設定する必要がなくなりました。 ビューのコンパイルを無効にしている場合を除き、 *.csproj* ファイルからこのプロパティが削除されている場合があります。

.NET Framework をターゲットとする場合は、継続して *.csproj* ファイルの [Microsoft.AspNetCore.Mvc.Razor.ViewCompilation](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Razor.ViewCompilation) NuGet パッケージを明示的に参照する必要があります。

[!code-xml[](../1x-to-2x/samples/AspNetCoreDotNetFx2.0App/AspNetCoreDotNetFx2.0App/AspNetCoreDotNetFx2.0App.csproj?range=15)]

<a name="app-insights"></a>

## <a name="rely-on-application-insights-light-up-features"></a>Application Insights の "Light-Up" 機能の利用

アプリケーション パフォーマンスのインストルメンテーションは、楽に設定できることが重要です。 Visual Studio 2017 のツールの新しい [Application Insights](/azure/application-insights/app-insights-overview) "light-up" 機能が利用できるようになりました。

Visual Studio 2017 で作成された ASP.NET Core 1.1 プロジェクトには、既定で Application Insights が追加されています。 *Program.cs* と *Startup.cs* 外で、Application Insights SDK を直接使用していない場合、次の手順に従います。

1. .NET Core をターゲットにする場合、 *.csproj* ファイルから次の `<PackageReference />` ノードを削除します。

    [!code-xml[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App.csproj?range=10)]

2. .NET Core をターゲットにする場合、*Program.cs* から `UseApplicationInsights` 拡張メソッド呼び出しを削除します。

    [!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Program.cs?name=snippet_ProgramCsMain&highlight=8)]

3. *_Layout.cshtml* から Application Insights のクライアント側 API 呼び出しを削除します。 これによって、次の 2 つのコード行が作成されます。

    [!code-cshtml[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Views/Shared/_Layout.cshtml?range=1,19&dedent=4)]

Application Insights SDK を直接使用している場合は、それを継続してください。 2\.0 の[メタパッケージ](xref:fundamentals/metapackage)には、Application Insights の最新バージョンが含まれているので、以前のバージョンを参照している場合、パッケージのダウングレード エラーが表示されます。

<a name="auth-and-identity"></a>

## <a name="adopt-authenticationidentity-improvements"></a>認証、Identity の機能強化の採用

ASP.NET Core 2.0 には、新しい認証モデルと ASP.NET Core Identity に対する大幅な変更が含まれています。 個々のユーザー アカウントを有効にしてプロジェクトを作成した場合や認証または Identity を手動で追加した場合、「IdentityASP.NET Core 2.0 への認証と [ の移行](xref:migration/1x-to-2x/identity-2x)」 を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* [ASP.NET Core 2.0 での互換性に影響する変更点](https://github.com/aspnet/announcements/issues?page=1&q=is%3Aissue+is%3Aopen+label%3A2.0.0+label%3A%22Breaking+change%22&utf8=%E2%9C%93)
