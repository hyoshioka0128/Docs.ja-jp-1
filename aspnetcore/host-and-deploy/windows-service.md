---
title: Windows サービスでの ASP.NET Core のホスト
author: rick-anderson
description: Windows サービスで ASP.NET Core アプリケーションをホストする方法を説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/07/2020
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
uid: host-and-deploy/windows-service
ms.openlocfilehash: 63267bf938c6d16b8a1b13940a4b3f8a02d1a1e4
ms.sourcegitcommit: 063a06b644d3ade3c15ce00e72a758ec1187dd06
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98252748"
---
# <a name="host-aspnet-core-in-a-windows-service"></a>Windows サービスでの ASP.NET Core のホスト

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core アプリは、IIS を 使用せずに、[Windows サービス](/dotnet/framework/windows-services/introduction-to-windows-service-applications)として Windows にホストできます。 Windows サービスとしてホストされている場合、サーバーの再起動後にアプリが自動的に起動します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/host-and-deploy/windows-service/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="prerequisites"></a>必須コンポーネント

* [ASP.NET Core SDK 2.1 以降](https://dotnet.microsoft.com/download)
* [PowerShell 6.2 以降](https://github.com/PowerShell/PowerShell)

## <a name="worker-service-template"></a>ワーカー サービス テンプレート

ASP.NET Core ワーカー サービス テンプレートは、実行時間が長いサービス アプリを作成する場合の出発点として利用できます。 テンプレートを Windows サービス アプリの基礎として使用するには:

1. .NET Core テンプレートからワーカー サービス アプリを作成します。
1. 「[アプリの構成](#app-configuration)」セクションのガイダンスに従って、Windows サービスとして実行できるようにワーカー サービス アプリを更新します。

[!INCLUDE[](~/includes/worker-template-instructions.md)]

## <a name="app-configuration"></a>アプリの構成

アプリには、[Microsoft.Extensions.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.Extensions.Hosting.WindowsServices)のパッケージ参照が必要です。

ホストのビルド時に `IHostBuilder.UseWindowsService` が呼び出されます。 アプリが Windows サービスとして実行している場合、メソッドは

* ホストの有効期間を `WindowsServiceLifetime` に設定します。
* [コンテンツ ルート](xref:fundamentals/index#content-root)を [AppContext.BaseDirectory](xref:System.AppContext.BaseDirectory) に設定します。 詳しくは、「[現在のディレクトリとコンテンツのルート](#current-directory-and-content-root)」セクションをご覧ください。
* イベント ログへのログ記録を有効にします。
  * アプリケーション名が既定のソース名として使用されます。
  * 既定のログ レベルは、ホストを構築するために `CreateDefaultBuilder` が呼び出される ASP.NET Core テンプレートに基づくアプリに対して "*警告*" 以上です。
  * 既定のログ レベルを、 *appsettings.json* /*appsettings.{Environment}.json* の `Logging:EventLog:LogLevel:Default` キーまたは他の構成プロバイダーでオーバーライドします。
  * 管理者のみが新しいイベント ソースを作成できます。 アプリケーション名を使用して、イベント ソースを作成できない場合、警告が *アプリケーション* ソースに記録され、イベント ログが無効になります。

*Program.cs* の `CreateHostBuilder` の場合:

```csharp
Host.CreateDefaultBuilder(args)
    .UseWindowsService()
    ...
```

このトピックには、次のサンプル アプリが付属しています。

* バックグラウンド ワーカー サービスのサンプル: バックグラウンド タスク用に[ホステッド サービス](xref:fundamentals/host/hosted-services)を使用する[ワーカー サービス テンプレート](#worker-service-template)に基づく、非 Web アプリのサンプルです。
* Web アプリ サービスのサンプル: バックグラウンド タスク用の[ホステッド サービス](xref:fundamentals/host/hosted-services)を使用する Windows サービスとして実行される Razor Pages Web アプリのサンプルです。

MVC のガイダンスについては、「<xref:mvc/overview>」と「<xref:migration/22-to-30>」にある記事を参照してください。

## <a name="deployment-type"></a>配置の種類

展開のシナリオに関する情報および注意事項については、「[.NET Core アプリケーションの展開](/dotnet/core/deploying/)」をご覧ください。

### <a name="sdk"></a>SDK

Razor Pages または MVC フレームワークを使用する Web アプリ ベースのサービスでは、プロジェクト ファイルに Web SDK を指定します。

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

サービスでバックグラウンド タスク ([ホステッド サービス](xref:fundamentals/host/hosted-services)など) のみを実行する場合は、プロジェクト ファイルにワーカー SDK を指定します。

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

### <a name="framework-dependent-deployment-fdd"></a>フレームワーク依存型展開 (FDD)

フレームワーク依存型展開 (FDD) は、ターゲット システムに .NET Core のシステム全体の共有バージョンが存在することに依存します。 この記事のガイダンスに従って、FDD シナリオを採用すると、*フレームワーク依存型実行可能ファイル* と呼ばれる実行可能ファイル ( *.exe*) が SDK によって生成されます。

[Web SDK](#sdk) を使用している場合、*web.config* ファイル (通常 ASP.NET Core アプリを発行する際に生成されます) は、Windows サービス アプリに対しては必要ありません。 *web.config* ファイルの作成を無効にするには、`true` に設定した `<IsTransformWebConfigDisabled>` プロパティを追加します。

```xml
<PropertyGroup>
  <TargetFramework>netcoreapp3.0</TargetFramework>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

### <a name="self-contained-deployment-scd"></a>自己完結型の展開 (SCD)

自己完結型の展開 (SCD) は、ホスト システムに共有フレームワークが存在することに依存しません。 ランタイムとアプリの依存関係が、アプリと共に展開されます。

Windows [ランタイム識別子 (RID)](/dotnet/core/rid-catalog) は、ターゲット フレームワークを格納する `<PropertyGroup>` に含まれます。

```xml
<RuntimeIdentifier>win7-x64</RuntimeIdentifier>
```

複数の RID を発行するには、次の処理を実行します。

* セミコロンで区切られたリストの形式で RID を指定します。
* プロパティ名 [\<RuntimeIdentifiers>](/dotnet/core/tools/csproj#runtimeidentifiers) (複数形) を使用します。

詳細については、「[.NET Core の RID カタログ](/dotnet/core/rid-catalog)」を参照してください。

## <a name="service-user-account"></a>サービス ユーザー アカウント

サービス用のユーザー アカウントを作成するには、PowerShell 6 の管理コマンド シェルから [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) コマンドレットを使用します。

Windows 10 October 2018 Update (バージョン 1809/ビルド 10.0.17763) 以降:

```powershell
New-LocalUser -Name {SERVICE NAME}
```

Windows 10 October 2018 Update (バージョン 1809/ビルド 10.0.17763) 以前の Windows OS:

```console
powershell -Command "New-LocalUser -Name {SERVICE NAME}"
```

入力を求められたら、[強力なパスワード](/windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements)を指定します。

[New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) コマンドレットに <xref:System.DateTime> という有効期限で `-AccountExpires` パラメーターを指定しない場合、アカウントは期限切れになりません。

詳しくは、「[Microsoft.PowerShell.LocalAccounts](/powershell/module/microsoft.powershell.localaccounts/)」および「[Service User Accounts (サービス ユーザー アカウント)](/windows/desktop/services/service-user-accounts)」をご覧ください。

Active Directory を使う場合、ユーザーを管理するための別の方法は、マネージド サービス アカウントを使うことです。 詳細については、「[Group Managed Service Accounts Overview (グループ マネージド サービス アカウントの概要)](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)」をご覧ください。

## <a name="log-on-as-a-service-rights"></a>サービスとしてログオン権利

サービス ユーザー アカウントに *サービスとしてログオン* 権利を確立するには、次の処理を実行します。

1. *secpol.msc* を実行して、ローカル セキュリティ ポリシー エディターを開きます。
1. **[ローカル ポリシー]** ノードを展開し、 **[ユーザー権利の割り当て]** を選択します。
1. **[サービスとしてログオン]** ポリシーを開きます。
1. **[ユーザーまたはグループの追加]** を選択します。
1. 次のいずれかの方法を使用して、オブジェクト名 (ユーザー アカウント) を指定します。
   1. オブジェクト名フィールドにユーザー アカウント (`{DOMAIN OR COMPUTER NAME\USER}`) を入力し、 **[OK]** を選択して、ポリシーにユーザーを追加します。
   1. **[詳細]** を選択します。 **[検索開始]** を選択します。 一覧からユーザー アカウントを選択します。 **[OK]** を選択します。 再度 **[OK]** を選択して、ポリシーにユーザーを追加します。
1. **[OK]** または **[適用]** を選択して、変更を受け入れます。

## <a name="create-and-manage-the-windows-service"></a>Windows サービスを作成して管理する

### <a name="create-a-service"></a>サービスを作成する

PowerShell コマンドを使用して、サービスを登録します。 PowerShell 6 の管理コマンド シェルから次のコマンドを実行します。

```powershell
$acl = Get-Acl "{EXE PATH}"
$aclRuleArgs = "{DOMAIN OR COMPUTER NAME\USER}", "Read,Write,ReadAndExecute", "ContainerInherit,ObjectInherit", "None", "Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($aclRuleArgs)
$acl.SetAccessRule($accessRule)
$acl | Set-Acl "{EXE PATH}"

New-Service -Name {SERVICE NAME} -BinaryPathName "{EXE FILE PATH}" -Credential "{DOMAIN OR COMPUTER NAME\USER}" -Description "{DESCRIPTION}" -DisplayName "{DISPLAY NAME}" -StartupType Automatic
```

* `{EXE PATH}`:ホスト上のアプリのフォルダーへのパス (`d:\myservice` など)。 このパスに、アプリの実行可能ファイルは含めないでください。 末尾のスラッシュは、必要ありません。
* `{DOMAIN OR COMPUTER NAME\USER}`:サービスのユーザー アカウント (`Contoso\ServiceUser` など)。
* `{SERVICE NAME}`:サービス名 (`MyService` など)。
* `{EXE FILE PATH}`:アプリの実行可能ファイルのパス (`d:\myservice\myservice.exe` など)。 拡張子付きの実行可能ファイルのファイル名を含めます。
* `{DESCRIPTION}`:サービスの説明 (`My sample service` など)。
* `{DISPLAY NAME}`:サービスの表示名 (`My Service` など)。

### <a name="start-a-service"></a>サービスを開始する

次の PowerShell 6 コマンドでサービスを開始します。

```powershell
Start-Service -Name {SERVICE NAME}
```

このコマンドでサービスを開始するには数秒かかります。

### <a name="determine-a-services-status"></a>サービスの状態を確認する

サービスの状態を確認するには、次の PowerShell 6 コマンドを使います。

```powershell
Get-Service -Name {SERVICE NAME}
```

この状態は、次のいずれかの値として報告されます。

* `Starting`
* `Running`
* `Stopping`
* `Stopped`

### <a name="stop-a-service"></a>サービスを停止する

次の PowerShell 6 コマンドでサービスを停止します。

```powershell
Stop-Service -Name {SERVICE NAME}
```

### <a name="remove-a-service"></a>サービスを削除する

サービスを停止した後、少ししてから、次の Powershell 6 コマンドを使ってサービスを削除します。

```powershell
Remove-Service -Name {SERVICE NAME}
```

## <a name="proxy-server-and-load-balancer-scenarios"></a>プロキシ サーバーとロード バランサーのシナリオ

インターネットまたは企業ネットワークからの要求とやり取りするサービスやプロキシまたはロード バランサーの背後にあるサービスでは、追加の構成が必要になる場合があります。 詳細については、「<xref:host-and-deploy/proxy-load-balancer>」を参照してください。

## <a name="configure-endpoints"></a>エンドポイントを構成する

既定では、ASP.NET Core は `http://localhost:5000` にバインドされます。 `ASPNETCORE_URLS` 環境変数を設定して、URL とポートを構成します。

追加の URL とポートの構成方法については、関連するサーバーの記事を参照してください。

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

* <xref:fundamentals/servers/kestrel/endpoints>

::: moniker-end

::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"

* <xref:fundamentals/servers/kestrel#endpoint-configuration>

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

* <xref:fundamentals/servers/httpsys#configure-windows-server>

上記のガイダンスでは、HTTPS エンドポイントのサポートについて説明しています。 たとえば、Windows サービスで認証が使用されている場合は、アプリを HTTPS 用に構成します。

> [!NOTE]
> サービス エンドポイントをセキュリティで保護するために ASP.NET Core の HTTPS 開発証明書を使用することはできません。

## <a name="current-directory-and-content-root"></a>現在のディレクトリとコンテンツのルート

Windows サービスに対して <xref:System.IO.Directory.GetCurrentDirectory%2A> を呼び出して返される現在の作業ディレクトリは *C:\\WINDOWS\\system32* フォルダーです。 *system32* フォルダーは、サービスのファイル (設定ファイルなど) を保存するために適した場所ではありません。 次のいずれかの方法を使用して、サービスのアセットと設定ファイルを管理し、アクセスします。

### <a name="use-contentrootpath-or-contentrootfileprovider"></a>ContentRootPath または ContentRootFileProvider を使用する

[IHostEnvironment.ContentRootPath](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath) または <xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootFileProvider> を使用して、アプリのリソースを見つけます。

アプリがサービスとして実行されると、<xref:Microsoft.Extensions.Hosting.WindowsServiceLifetimeHostBuilderExtensions.UseWindowsService%2A> によって <xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath> が [AppContext.BaseDirectory](xref:System.AppContext.BaseDirectory) に設定されます。

アプリの既定設定ファイルである *appsettings.json* と *appsettings.{Environment}.json* は、[ホストの構築中に CreateDefaultBuilder](xref:fundamentals/host/generic-host#set-up-a-host) を呼び出すことで、アプリのコンテンツ ルートから読み込まれます。

<xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration%2A> の開発者コードによって読み込まれた他の設定ファイルについては、<xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath%2A> を呼び出す必要はありません。 次の例では、*custom_settings.json* ファイルはアプリのコンテンツ ルートに存在しており、明示的にベース パスを設定しなくても読み込まれます。

[!code-csharp[](windows-service/samples_snapshot/CustomSettingsExample.cs?highlight=13)]

Windows サービス アプリからは *C:\\WINDOWS\\system32* フォルダーが現在のディレクトリとして返されるため、<xref:System.IO.Directory.GetCurrentDirectory%2A> を使用してリソース パスを取得しないでください。

### <a name="store-a-services-files-in-a-suitable-location-on-disk"></a>ディスク上の適切な場所にサービスのファイルを格納する

ファイルを含むフォルダーに対して <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> を使用するときは、<xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath%2A> を使用して絶対パスを指定します。

## <a name="troubleshoot"></a>トラブルシューティング

Windows サービス アプリのトラブルシューティングについては、「<xref:test/troubleshoot>」を参照してください。

### <a name="common-errors"></a>一般的なエラー

* PowerShell の以前のバージョンまたはプレリリース バージョンが使用されています。
* 登録されたサービスに、[dotnet publish](/dotnet/core/tools/dotnet-publish) コマンドからのアプリの **発行済み** 出力が使用されません。 [dotnet build](/dotnet/core/tools/dotnet-build) コマンドの出力が、アプリの展開でサポートされていません。 発行された資産は、展開の種類に応じて次のいずれかのフォルダーにあります。
  * *bin/Release/{TARGET FRAMEWORK}/publish* (FDD)
  * *bin/Release/{TARGET FRAMEWORK}/{RUNTIME IDENTIFIER}/publish* (SCD)
* サービスが実行中の状態ではありません。
* アプリに使用されるリソース (証明書など) のパスが正しくありません。 Windows サービスのベース パスは *c:\\Windows\\System32* です。
* "*サービスとしてログオンする*" アクセス権がユーザーにありません。
* `New-Service` PowerShell コマンドの実行時に、ユーザーのパスワードの有効期限が切れていたか、正しく渡されていません。
* アプリに ASP.NET Core 認証が必要ですが、セキュリティで保護された接続 (HTTPS) 用に構成されていません。
* 要求 URL ポートが正しくないか、アプリで正しく構成されていません。

### <a name="system-and-application-event-logs"></a>システム イベント ログとアプリケーション イベント ログ

システム イベント ログとアプリケーション イベント ログにアクセスします。

1. [スタート] メニューを開き、*イベント ビューアー* を検索し、**イベント ビューアー** アプリを選択します。
1. **イベント ビューアー** で **[Windows ログ]** ノードを開きます。
1. **[システム]** を選択してシステム イベント ログを開きます。 **[Application]** を選択して、アプリケーション イベント ログを開きます。
1. 失敗したアプリに関連するエラーを検索します。

### <a name="run-the-app-at-a-command-prompt"></a>コマンド プロンプトでアプリを実行する

多くのスタートアップ エラーでは、イベント ログに有用な情報が生成されません。 ホスティング システムのコマンド プロンプトでアプリを実行すると、エラーの原因がわかることがあります。 アプリからさらに詳細情報をログに記録するには、[ログ レベル](xref:fundamentals/logging/index#log-level)を低くするか、[開発環境](xref:fundamentals/environments)でアプリを実行します。

### <a name="clear-package-caches"></a>パッケージ キャッシュをクリアする

開発マシンで .NET Core SDK をアップグレードしたり、アプリ内のパッケージ バージョンを変更したりした直後に、機能しているアプリが失敗することがあります。 場合によっては、パッケージに統一性がないと、メジャー アップグレード実行時にアプリが破壊されることがあります。 これらの問題のほとんどは、次の手順で解決できます。

1. *bin* フォルダーと *obj* フォルダーを削除します。
1. コマンド シェルから [dotnet nuget locals all --clear](/dotnet/core/tools/dotnet-nuget-locals) を実行して、パッケージ キャッシュをクリアします。

   パッケージ キャッシュをクリアするには、[nuget.exe](https://www.nuget.org/downloads) ツールを使用して `nuget locals all -clear` コマンドを実行する方法もあります。 *nuget.exe* は、Windows デスクトップ オペレーティング システムにバンドルされているインストールではなく、[NuGet Web サイト](https://www.nuget.org/downloads)から別に入手する必要があります。

1. プロジェクトを復元してリビルドします。
1. アプリを再展開する前に、サーバー上の展開フォルダー内のすべてのファイルを削除します。

### <a name="slow-or-hanging-app"></a>アプリの速度低下またはハング

"*クラッシュ ダンプ*" とはシステムのメモリのスナップショットであり、アプリのクラッシュ、起動の失敗、遅いアプリの原因を突き止めるのに役立ちます。

#### <a name="app-crashes-or-encounters-an-exception"></a>アプリのクラッシュまたは例外の発生

[Windows エラー報告 (WER)](/windows/desktop/wer/windows-error-reporting) からダンプを取得して分析します。

1. `c:\dumps` にクラッシュ ダンプ ファイルを保持するフォルダーを作成します。
1. 次のアプリケーションの実行可能ファイル名で [EnableDumps PowerShell スクリプト](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/host-and-deploy/windows-service/samples/scripts/EnableDumps.ps1)を実行します。

   ```powershell
   .\EnableDumps {APPLICATION EXE} c:\dumps
   ```

1. クラッシュが発生する条件の下でアプリを実行します。
1. クラッシュが発生した後、[DisableDumps PowerShell スクリプト](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/host-and-deploy/windows-service/samples/scripts/DisableDumps.ps1)を実行します。

   ```powershell
   .\DisableDumps {APPLICATION EXE}
   ```

アプリがクラッシュし、ダンプの収集が完了したら、アプリを普通に終了してかまいません。 PowerShell スクリプトにより、アプリごとに最大 5 つのダンプを収集する WER が構成されます。

> [!WARNING]
> クラッシュ ダンプでは、大量のディスク領域 (1 つにつき最大、数ギガバイト) が使用される場合があります。

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>アプリが起動時または正常な実行中にハングまたは失敗する

アプリが起動時または正常な実行中に "*ハング*" (応答を停止するがクラッシュしない) または失敗するときは、「[User-Mode Dump Files:Choosing the Best Tool](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool)」(ユーザー モード ダンプ ファイル: 最適なツールの選択) を参照し、適切なツールを選択してダンプを生成します。

#### <a name="analyze-the-dump"></a>ダンプを分析する

ダンプはいくつかの方法で分析できます。 詳細については、「[Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file)」(ユーザー モード ダンプ ファイルの分析) を参照してください。

## <a name="additional-resources"></a>その他の技術情報

::: moniker-end

::: moniker range=">= aspnetcore-5.0"
* [Kestrel エンドポイント構成](xref:fundamentals/servers/kestrel/endpoints) (HTTPS 構成と SNI サポートを含む)
::: moniker-end
::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"
* [Kestrel エンドポイント構成](xref:fundamentals/servers/kestrel#endpoint-configuration) (HTTPS 構成と SNI サポートを含む)
::: moniker-end

::: moniker range=">= aspnetcore-3.0"
* <xref:fundamentals/host/generic-host>
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="= aspnetcore-2.2"

ASP.NET Core アプリは、IIS を 使用せずに、[Windows サービス](/dotnet/framework/windows-services/introduction-to-windows-service-applications)として Windows にホストできます。 Windows サービスとしてホストされている場合、サーバーの再起動後にアプリが自動的に起動します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/host-and-deploy/windows-service/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="prerequisites"></a>必須コンポーネント

* [ASP.NET Core SDK 2.1 以降](https://dotnet.microsoft.com/download)
* [PowerShell 6.2 以降](https://github.com/PowerShell/PowerShell)

## <a name="app-configuration"></a>アプリの構成

アプリでは、[Microsoft.AspNetCore.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.WindowsServices) と [Microsoft.Extensions.Logging.EventLog](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventLog) へのパッケージ参照が必要です。

サービスの外部で実行しているときにテストとデバッグを行うには、アプリがサービスとして実行しているかコンソール アプリとして実行しているかを判別するコードを追加します。 デバッガーがアタッチされているか、`--console` スイッチが存在するかを検査します。 いずれかの条件が満たされる場合 (アプリがサービスとして実行していない場合)、<xref:Microsoft.AspNetCore.Hosting.WebHostExtensions.Run*> を呼び出します。 条件が満たされない場合 (アプリがサービスとして実行している場合):

* <xref:System.IO.Directory.SetCurrentDirectory*> を呼び出し、アプリの発行場所のパスを使用します。 パスを取得するために <xref:System.IO.Directory.GetCurrentDirectory*> を呼び出さないでください。<xref:System.IO.Directory.GetCurrentDirectory*> が呼び出されると、Windows サービス アプリは *C:\\WINDOWS\\system32* フォルダーを戻すためです。 詳しくは、「[現在のディレクトリとコンテンツのルート](#current-directory-and-content-root)」セクションをご覧ください。 `CreateWebHostBuilder` でアプリを構成する前に、この手順に従います。
* <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> を呼び出して、アプリをサービスとして実行します。

[コマンドライン構成プロバイダー](xref:fundamentals/configuration/index#command-line-configuration-provider)では、コマンドライン引数に名前と値の組が必要であるため、<xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> が引数を受け取る前に `--console` スイッチは引数から削除されます。

Windows イベント ログに書き込むには、EventLog プロバイダーを <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureLogging*> に追加します。 *appsettings.Production.json* ファイルで `Logging:LogLevel:Default` キーを使用してログ レベルを設定します。

サンプル アプリの次の例では、アプリ内で有効期間イベントを処理するために、<xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> の代わりに `RunAsCustomService` を呼び出しています。 詳しくは、「[イベントの開始と停止を扱う](#handle-starting-and-stopping-events)」セクションをご覧ください。

[!code-csharp[](windows-service/samples/2.x/AspNetCoreService/Program.cs?name=snippet_Program)]

## <a name="deployment-type"></a>配置の種類

展開のシナリオに関する情報および注意事項については、「[.NET Core アプリケーションの展開](/dotnet/core/deploying/)」をご覧ください。

### <a name="sdk"></a>SDK

Razor Pages または MVC フレームワークを使用する Web アプリ ベースのサービスでは、プロジェクト ファイルに Web SDK を指定します。

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

サービスでバックグラウンド タスク ([ホステッド サービス](xref:fundamentals/host/hosted-services)など) のみを実行する場合は、プロジェクト ファイルにワーカー SDK を指定します。

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

### <a name="framework-dependent-deployment-fdd"></a>フレームワーク依存型展開 (FDD)

フレームワーク依存型展開 (FDD) は、ターゲット システムに .NET Core のシステム全体の共有バージョンが存在することに依存します。 この記事のガイダンスに従って、FDD シナリオを採用すると、*フレームワーク依存型実行可能ファイル* と呼ばれる実行可能ファイル ( *.exe*) が SDK によって生成されます。

Windows [ランタイム識別子 (RID)](/dotnet/core/rid-catalog) ([\<RuntimeIdentifier>](/dotnet/core/tools/csproj#runtimeidentifier)) にはターゲット フレームワークが含まれます。 次の例では、RID が `win7-x64` に設定されています。 `<SelfContained>` プロパティが `false` に設定されている。 これらのプロパティによって、Windows 用の実行可能ファイル ( *.exe*) と共有 .NET Core フレームワークに依存するアプリを生成するよう SDK に指示します。

*web.config* ファイル (通常 ASP.NET Core アプリを発行する際に生成されます) は、Windows サービス アプリに対しては必要ありません。 *web.config* ファイルの作成を無効にするには、`true` に設定した `<IsTransformWebConfigDisabled>` プロパティを追加します。

```xml
<PropertyGroup>
  <TargetFramework>netcoreapp2.2</TargetFramework>
  <RuntimeIdentifier>win7-x64</RuntimeIdentifier>
  <SelfContained>false</SelfContained>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

### <a name="self-contained-deployment-scd"></a>自己完結型の展開 (SCD)

自己完結型の展開 (SCD) は、ホスト システムに共有フレームワークが存在することに依存しません。 ランタイムとアプリの依存関係が、アプリと共に展開されます。

Windows [ランタイム識別子 (RID)](/dotnet/core/rid-catalog) は、ターゲット フレームワークを格納する `<PropertyGroup>` に含まれます。

```xml
<RuntimeIdentifier>win7-x64</RuntimeIdentifier>
```

複数の RID を発行するには、次の処理を実行します。

* セミコロンで区切られたリストの形式で RID を指定します。
* プロパティ名 [\<RuntimeIdentifiers>](/dotnet/core/tools/csproj#runtimeidentifiers) (複数形) を使用します。

詳細については、「[.NET Core の RID カタログ](/dotnet/core/rid-catalog)」を参照してください。

`<SelfContained>` プロパティが `true` に設定されています。

```xml
<SelfContained>true</SelfContained>
```

## <a name="service-user-account"></a>サービス ユーザー アカウント

サービス用のユーザー アカウントを作成するには、PowerShell 6 の管理コマンド シェルから [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) コマンドレットを使用します。

Windows 10 October 2018 Update (バージョン 1809/ビルド 10.0.17763) 以降:

```powershell
New-LocalUser -Name {SERVICE NAME}
```

Windows 10 October 2018 Update (バージョン 1809/ビルド 10.0.17763) 以前の Windows OS:

```console
powershell -Command "New-LocalUser -Name {SERVICE NAME}"
```

入力を求められたら、[強力なパスワード](/windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements)を指定します。

[New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) コマンドレットに <xref:System.DateTime> という有効期限で `-AccountExpires` パラメーターを指定しない場合、アカウントは期限切れになりません。

詳しくは、「[Microsoft.PowerShell.LocalAccounts](/powershell/module/microsoft.powershell.localaccounts/)」および「[Service User Accounts (サービス ユーザー アカウント)](/windows/desktop/services/service-user-accounts)」をご覧ください。

Active Directory を使う場合、ユーザーを管理するための別の方法は、マネージド サービス アカウントを使うことです。 詳細については、「[Group Managed Service Accounts Overview (グループ マネージド サービス アカウントの概要)](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)」をご覧ください。

## <a name="log-on-as-a-service-rights"></a>サービスとしてログオン権利

サービス ユーザー アカウントに *サービスとしてログオン* 権利を確立するには、次の処理を実行します。

1. *secpol.msc* を実行して、ローカル セキュリティ ポリシー エディターを開きます。
1. **[ローカル ポリシー]** ノードを展開し、 **[ユーザー権利の割り当て]** を選択します。
1. **[サービスとしてログオン]** ポリシーを開きます。
1. **[ユーザーまたはグループの追加]** を選択します。
1. 次のいずれかの方法を使用して、オブジェクト名 (ユーザー アカウント) を指定します。
   1. オブジェクト名フィールドにユーザー アカウント (`{DOMAIN OR COMPUTER NAME\USER}`) を入力し、 **[OK]** を選択して、ポリシーにユーザーを追加します。
   1. **[詳細]** を選択します。 **[検索開始]** を選択します。 一覧からユーザー アカウントを選択します。 **[OK]** を選択します。 再度 **[OK]** を選択して、ポリシーにユーザーを追加します。
1. **[OK]** または **[適用]** を選択して、変更を受け入れます。

## <a name="create-and-manage-the-windows-service"></a>Windows サービスを作成して管理する

### <a name="create-a-service"></a>サービスを作成する

PowerShell コマンドを使用して、サービスを登録します。 PowerShell 6 の管理コマンド シェルから次のコマンドを実行します。

```powershell
$acl = Get-Acl "{EXE PATH}"
$aclRuleArgs = {DOMAIN OR COMPUTER NAME\USER}, "Read,Write,ReadAndExecute", "ContainerInherit,ObjectInherit", "None", "Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($aclRuleArgs)
$acl.SetAccessRule($accessRule)
$acl | Set-Acl "{EXE PATH}"

New-Service -Name {SERVICE NAME} -BinaryPathName {EXE FILE PATH} -Credential {DOMAIN OR COMPUTER NAME\USER} -Description "{DESCRIPTION}" -DisplayName "{DISPLAY NAME}" -StartupType Automatic
```

* `{EXE PATH}`:ホスト上のアプリのフォルダーへのパス (`d:\myservice` など)。 このパスに、アプリの実行可能ファイルは含めないでください。 末尾のスラッシュは、必要ありません。
* `{DOMAIN OR COMPUTER NAME\USER}`:サービスのユーザー アカウント (`Contoso\ServiceUser` など)。
* `{SERVICE NAME}`:サービス名 (`MyService` など)。
* `{EXE FILE PATH}`:アプリの実行可能ファイルのパス (`d:\myservice\myservice.exe` など)。 拡張子付きの実行可能ファイルのファイル名を含めます。
* `{DESCRIPTION}`:サービスの説明 (`My sample service` など)。
* `{DISPLAY NAME}`:サービスの表示名 (`My Service` など)。

### <a name="start-a-service"></a>サービスを開始する

次の PowerShell 6 コマンドでサービスを開始します。

```powershell
Start-Service -Name {SERVICE NAME}
```

このコマンドでサービスを開始するには数秒かかります。

### <a name="determine-a-services-status"></a>サービスの状態を確認する

サービスの状態を確認するには、次の PowerShell 6 コマンドを使います。

```powershell
Get-Service -Name {SERVICE NAME}
```

この状態は、次のいずれかの値として報告されます。

* `Starting`
* `Running`
* `Stopping`
* `Stopped`

### <a name="stop-a-service"></a>サービスを停止する

次の PowerShell 6 コマンドでサービスを停止します。

```powershell
Stop-Service -Name {SERVICE NAME}
```

### <a name="remove-a-service"></a>サービスを削除する

サービスを停止した後、少ししてから、次の Powershell 6 コマンドを使ってサービスを削除します。

```powershell
Remove-Service -Name {SERVICE NAME}
```

## <a name="handle-starting-and-stopping-events"></a>イベントの開始と停止を扱う

<xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarting*>、<xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarted*>、および <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStopping*> イベントを処理するには、次の処理を実行します。

1. `OnStarting`、`OnStarted`、および `OnStopping` メソッドを使用して、<xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService> から派生するクラスを作成します。

   [!code-csharp[](windows-service/samples/2.x/AspNetCoreService/CustomWebHostService.cs?name=snippet_CustomWebHostService)]

2. `CustomWebHostService` を <xref:System.ServiceProcess.ServiceBase.Run*> に渡す <xref:Microsoft.AspNetCore.Hosting.IWebHost> の拡張メソッドを作成します。

   [!code-csharp[](windows-service/samples/2.x/AspNetCoreService/WebHostServiceExtensions.cs?name=ExtensionsClass)]

3. `Program.Main` で、<xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> ではなく、拡張メソッド `RunAsCustomService` を呼び出します。

   ```csharp
   host.RunAsCustomService();
   ```

   `Program.Main` 内の <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> の場所を確認するには、「[展開の種類](#deployment-type)」セクションに示されているコード サンプルを参照してください。

## <a name="proxy-server-and-load-balancer-scenarios"></a>プロキシ サーバーとロード バランサーのシナリオ

インターネットまたは企業ネットワークからの要求とやり取りするサービスやプロキシまたはロード バランサーの背後にあるサービスでは、追加の構成が必要になる場合があります。 詳細については、「<xref:host-and-deploy/proxy-load-balancer>」を参照してください。

## <a name="configure-endpoints"></a>エンドポイントを構成する

既定では、ASP.NET Core は `http://localhost:5000` にバインドされます。 `ASPNETCORE_URLS` 環境変数を設定して、URL とポートを構成します。

追加の URL とポートの構成方法については、関連するサーバーの記事を参照してください。

* <xref:fundamentals/servers/kestrel#endpoint-configuration>
* <xref:fundamentals/servers/httpsys#configure-windows-server>

上記のガイダンスでは、HTTPS エンドポイントのサポートについて説明しています。 たとえば、Windows サービスで認証が使用されている場合は、アプリを HTTPS 用に構成します。

> [!NOTE]
> サービス エンドポイントをセキュリティで保護するために ASP.NET Core の HTTPS 開発証明書を使用することはできません。

## <a name="current-directory-and-content-root"></a>現在のディレクトリとコンテンツのルート

Windows サービスに対して <xref:System.IO.Directory.GetCurrentDirectory*> を呼び出して返される現在の作業ディレクトリは *C:\\WINDOWS\\system32* フォルダーです。 *system32* フォルダーは、サービスのファイル (設定ファイルなど) を保存するために適した場所ではありません。 次のいずれかの方法を使用して、サービスのアセットと設定ファイルを管理し、アクセスします。

### <a name="set-the-content-root-path-to-the-apps-folder"></a>アプリのフォルダーにコンテンツ ルート パスを設定する

<xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ContentRootPath*> は、サービスの作成時に `binPath` 引数に指定されたものと同じパスです。 `GetCurrentDirectory` を呼び出して設定ファイルへのパスを作成する代わりに、アプリの[コンテンツ ルート](xref:fundamentals/index#content-root)へのパスを指定して <xref:System.IO.Directory.SetCurrentDirectory*> を呼び出します。

`Program.Main` で、サービスの実行可能ファイルがあるフォルダーへのパスを判別し、そのパスを使用してアプリのコンテンツ ルートを確立します。

```csharp
var pathToExe = Process.GetCurrentProcess().MainModule.FileName;
var pathToContentRoot = Path.GetDirectoryName(pathToExe);
Directory.SetCurrentDirectory(pathToContentRoot);

CreateWebHostBuilder(args)
    .Build()
    .RunAsService();
```

### <a name="store-a-services-files-in-a-suitable-location-on-disk"></a>ディスク上の適切な場所にサービスのファイルを格納する

ファイルを含むフォルダーに対して <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> を使用するときは、<xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath*> を使用して絶対パスを指定します。

## <a name="troubleshoot"></a>トラブルシューティング

Windows サービス アプリのトラブルシューティングについては、「<xref:test/troubleshoot>」を参照してください。

### <a name="common-errors"></a>一般的なエラー

* PowerShell の以前のバージョンまたはプレリリース バージョンが使用されています。
* 登録されたサービスに、[dotnet publish](/dotnet/core/tools/dotnet-publish) コマンドからのアプリの **発行済み** 出力が使用されません。 [dotnet build](/dotnet/core/tools/dotnet-build) コマンドの出力が、アプリの展開でサポートされていません。 発行された資産は、展開の種類に応じて次のいずれかのフォルダーにあります。
  * *bin/Release/{TARGET FRAMEWORK}/publish* (FDD)
  * *bin/Release/{TARGET FRAMEWORK}/{RUNTIME IDENTIFIER}/publish* (SCD)
* サービスが実行中の状態ではありません。
* アプリに使用されるリソース (証明書など) のパスが正しくありません。 Windows サービスのベース パスは *c:\\Windows\\System32* です。
* "*サービスとしてログオンする*" アクセス権がユーザーにありません。
* `New-Service` PowerShell コマンドの実行時に、ユーザーのパスワードの有効期限が切れていたか、正しく渡されていません。
* アプリに ASP.NET Core 認証が必要ですが、セキュリティで保護された接続 (HTTPS) 用に構成されていません。
* 要求 URL ポートが正しくないか、アプリで正しく構成されていません。

### <a name="system-and-application-event-logs"></a>システム イベント ログとアプリケーション イベント ログ

システム イベント ログとアプリケーション イベント ログにアクセスします。

1. [スタート] メニューを開き、*イベント ビューアー* を検索し、**イベント ビューアー** アプリを選択します。
1. **イベント ビューアー** で **[Windows ログ]** ノードを開きます。
1. **[システム]** を選択してシステム イベント ログを開きます。 **[Application]** を選択して、アプリケーション イベント ログを開きます。
1. 失敗したアプリに関連するエラーを検索します。

### <a name="run-the-app-at-a-command-prompt"></a>コマンド プロンプトでアプリを実行する

多くのスタートアップ エラーでは、イベント ログに有用な情報が生成されません。 ホスティング システムのコマンド プロンプトでアプリを実行すると、エラーの原因がわかることがあります。 アプリからさらに詳細情報をログに記録するには、[ログ レベル](xref:fundamentals/logging/index#log-level)を低くするか、[開発環境](xref:fundamentals/environments)でアプリを実行します。

### <a name="clear-package-caches"></a>パッケージ キャッシュをクリアする

開発マシンで .NET Core SDK をアップグレードしたり、アプリ内のパッケージ バージョンを変更したりした直後に、機能しているアプリが失敗することがあります。 場合によっては、パッケージに統一性がないと、メジャー アップグレード実行時にアプリが破壊されることがあります。 これらの問題のほとんどは、次の手順で解決できます。

1. *bin* フォルダーと *obj* フォルダーを削除します。
1. コマンド シェルから [dotnet nuget locals all --clear](/dotnet/core/tools/dotnet-nuget-locals) を実行して、パッケージ キャッシュをクリアします。

   パッケージ キャッシュをクリアするには、[nuget.exe](https://www.nuget.org/downloads) ツールを使用して `nuget locals all -clear` コマンドを実行する方法もあります。 *nuget.exe* は、Windows デスクトップ オペレーティング システムにバンドルされているインストールではなく、[NuGet Web サイト](https://www.nuget.org/downloads)から別に入手する必要があります。

1. プロジェクトを復元してリビルドします。
1. アプリを再展開する前に、サーバー上の展開フォルダー内のすべてのファイルを削除します。

### <a name="slow-or-hanging-app"></a>アプリの速度低下またはハング

"*クラッシュ ダンプ*" とはシステムのメモリのスナップショットであり、アプリのクラッシュ、起動の失敗、遅いアプリの原因を突き止めるのに役立ちます。

#### <a name="app-crashes-or-encounters-an-exception"></a>アプリのクラッシュまたは例外の発生

[Windows エラー報告 (WER)](/windows/desktop/wer/windows-error-reporting) からダンプを取得して分析します。

1. `c:\dumps` にクラッシュ ダンプ ファイルを保持するフォルダーを作成します。
1. 次のアプリケーションの実行可能ファイル名で [EnableDumps PowerShell スクリプト](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/host-and-deploy/windows-service/scripts/EnableDumps.ps1)を実行します。

   ```console
   .\EnableDumps {APPLICATION EXE} c:\dumps
   ```

1. クラッシュが発生する条件の下でアプリを実行します。
1. クラッシュが発生した後、[DisableDumps PowerShell スクリプト](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/host-and-deploy/windows-service/scripts/DisableDumps.ps1)を実行します。

   ```console
   .\DisableDumps {APPLICATION EXE}
   ```

アプリがクラッシュし、ダンプの収集が完了したら、アプリを普通に終了してかまいません。 PowerShell スクリプトにより、アプリごとに最大 5 つのダンプを収集する WER が構成されます。

> [!WARNING]
> クラッシュ ダンプでは、大量のディスク領域 (1 つにつき最大、数ギガバイト) が使用される場合があります。

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>アプリが起動時または正常な実行中にハングまたは失敗する

アプリが起動時または正常な実行中に "*ハング*" (応答を停止するがクラッシュしない) または失敗するときは、「[User-Mode Dump Files:Choosing the Best Tool](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool)」(ユーザー モード ダンプ ファイル: 最適なツールの選択) を参照し、適切なツールを選択してダンプを生成します。

#### <a name="analyze-the-dump"></a>ダンプを分析する

ダンプはいくつかの方法で分析できます。 詳細については、「[Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file)」(ユーザー モード ダンプ ファイルの分析) を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* [Kestrel エンドポイント構成](xref:fundamentals/servers/kestrel#endpoint-configuration) (HTTPS 構成と SNI サポートを含む)
* <xref:fundamentals/host/web-host>
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="< aspnetcore-2.2"

ASP.NET Core アプリは、IIS を 使用せずに、[Windows サービス](/dotnet/framework/windows-services/introduction-to-windows-service-applications)として Windows にホストできます。 Windows サービスとしてホストされている場合、サーバーの再起動後にアプリが自動的に起動します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/host-and-deploy/windows-service/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="prerequisites"></a>必須コンポーネント

* [ASP.NET Core SDK 2.1 以降](https://dotnet.microsoft.com/download)
* [PowerShell 6.2 以降](https://github.com/PowerShell/PowerShell)

## <a name="app-configuration"></a>アプリの構成

アプリでは、[Microsoft.AspNetCore.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.WindowsServices) と [Microsoft.Extensions.Logging.EventLog](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventLog) へのパッケージ参照が必要です。

サービスの外部で実行しているときにテストとデバッグを行うには、アプリがサービスとして実行しているかコンソール アプリとして実行しているかを判別するコードを追加します。 デバッガーがアタッチされているか、`--console` スイッチが存在するかを検査します。 いずれかの条件が満たされる場合 (アプリがサービスとして実行していない場合)、<xref:Microsoft.AspNetCore.Hosting.WebHostExtensions.Run*> を呼び出します。 条件が満たされない場合 (アプリがサービスとして実行している場合):

* <xref:System.IO.Directory.SetCurrentDirectory*> を呼び出し、アプリの発行場所のパスを使用します。 パスを取得するために <xref:System.IO.Directory.GetCurrentDirectory*> を呼び出さないでください。<xref:System.IO.Directory.GetCurrentDirectory*> が呼び出されると、Windows サービス アプリは *C:\\WINDOWS\\system32* フォルダーを戻すためです。 詳しくは、「[現在のディレクトリとコンテンツのルート](#current-directory-and-content-root)」セクションをご覧ください。 `CreateWebHostBuilder` でアプリを構成する前に、この手順に従います。
* <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> を呼び出して、アプリをサービスとして実行します。

[コマンドライン構成プロバイダー](xref:fundamentals/configuration/index#command-line-configuration-provider)では、コマンドライン引数に名前と値の組が必要であるため、<xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> が引数を受け取る前に `--console` スイッチは引数から削除されます。

Windows イベント ログに書き込むには、EventLog プロバイダーを <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureLogging*> に追加します。 *appsettings.Production.json* ファイルで `Logging:LogLevel:Default` キーを使用してログ レベルを設定します。

サンプル アプリの次の例では、アプリ内で有効期間イベントを処理するために、<xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> の代わりに `RunAsCustomService` を呼び出しています。 詳しくは、「[イベントの開始と停止を扱う](#handle-starting-and-stopping-events)」セクションをご覧ください。

[!code-csharp[](windows-service/samples/2.x/AspNetCoreService/Program.cs?name=snippet_Program)]

## <a name="deployment-type"></a>配置の種類

展開のシナリオに関する情報および注意事項については、「[.NET Core アプリケーションの展開](/dotnet/core/deploying/)」をご覧ください。

### <a name="sdk"></a>SDK

Razor Pages または MVC フレームワークを使用する Web アプリ ベースのサービスでは、プロジェクト ファイルに Web SDK を指定します。

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

サービスでバックグラウンド タスク ([ホステッド サービス](xref:fundamentals/host/hosted-services)など) のみを実行する場合は、プロジェクト ファイルにワーカー SDK を指定します。

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

### <a name="framework-dependent-deployment-fdd"></a>フレームワーク依存型展開 (FDD)

フレームワーク依存型展開 (FDD) は、ターゲット システムに .NET Core のシステム全体の共有バージョンが存在することに依存します。 この記事のガイダンスに従って、FDD シナリオを採用すると、*フレームワーク依存型実行可能ファイル* と呼ばれる実行可能ファイル ( *.exe*) が SDK によって生成されます。

Windows [ランタイム識別子 (RID)](/dotnet/core/rid-catalog) ([\<RuntimeIdentifier>](/dotnet/core/tools/csproj#runtimeidentifier)) にはターゲット フレームワークが含まれます。 次の例では、RID が `win7-x64` に設定されています。 `<SelfContained>` プロパティが `false` に設定されている。 これらのプロパティによって、Windows 用の実行可能ファイル ( *.exe*) と共有 .NET Core フレームワークに依存するアプリを生成するよう SDK に指示します。

`<UseAppHost>` プロパティが `true` に設定されている。 このプロパティによって、サービスに FDD のアクティブ化パス (実行可能ファイル、 *.exe*) を指定します。

*web.config* ファイル (通常 ASP.NET Core アプリを発行する際に生成されます) は、Windows サービス アプリに対しては必要ありません。 *web.config* ファイルの作成を無効にするには、`true` に設定した `<IsTransformWebConfigDisabled>` プロパティを追加します。

```xml
<PropertyGroup>
  <TargetFramework>netcoreapp2.2</TargetFramework>
  <RuntimeIdentifier>win7-x64</RuntimeIdentifier>
  <UseAppHost>true</UseAppHost>
  <SelfContained>false</SelfContained>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

### <a name="self-contained-deployment-scd"></a>自己完結型の展開 (SCD)

自己完結型の展開 (SCD) は、ホスト システムに共有フレームワークが存在することに依存しません。 ランタイムとアプリの依存関係が、アプリと共に展開されます。

Windows [ランタイム識別子 (RID)](/dotnet/core/rid-catalog) は、ターゲット フレームワークを格納する `<PropertyGroup>` に含まれます。

```xml
<RuntimeIdentifier>win7-x64</RuntimeIdentifier>
```

複数の RID を発行するには、次の処理を実行します。

* セミコロンで区切られたリストの形式で RID を指定します。
* プロパティ名 [\<RuntimeIdentifiers>](/dotnet/core/tools/csproj#runtimeidentifiers) (複数形) を使用します。

詳細については、「[.NET Core の RID カタログ](/dotnet/core/rid-catalog)」を参照してください。

`<SelfContained>` プロパティが `true` に設定されています。

```xml
<SelfContained>true</SelfContained>
```

## <a name="service-user-account"></a>サービス ユーザー アカウント

サービス用のユーザー アカウントを作成するには、PowerShell 6 の管理コマンド シェルから [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) コマンドレットを使用します。

Windows 10 October 2018 Update (バージョン 1809/ビルド 10.0.17763) 以降:

```powershell
New-LocalUser -Name {SERVICE NAME}
```

Windows 10 October 2018 Update (バージョン 1809/ビルド 10.0.17763) 以前の Windows OS:

```console
powershell -Command "New-LocalUser -Name {SERVICE NAME}"
```

入力を求められたら、[強力なパスワード](/windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements)を指定します。

[New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) コマンドレットに <xref:System.DateTime> という有効期限で `-AccountExpires` パラメーターを指定しない場合、アカウントは期限切れになりません。

詳しくは、「[Microsoft.PowerShell.LocalAccounts](/powershell/module/microsoft.powershell.localaccounts/)」および「[Service User Accounts (サービス ユーザー アカウント)](/windows/desktop/services/service-user-accounts)」をご覧ください。

Active Directory を使う場合、ユーザーを管理するための別の方法は、マネージド サービス アカウントを使うことです。 詳細については、「[Group Managed Service Accounts Overview (グループ マネージド サービス アカウントの概要)](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)」をご覧ください。

## <a name="log-on-as-a-service-rights"></a>サービスとしてログオン権利

サービス ユーザー アカウントに *サービスとしてログオン* 権利を確立するには、次の処理を実行します。

1. *secpol.msc* を実行して、ローカル セキュリティ ポリシー エディターを開きます。
1. **[ローカル ポリシー]** ノードを展開し、 **[ユーザー権利の割り当て]** を選択します。
1. **[サービスとしてログオン]** ポリシーを開きます。
1. **[ユーザーまたはグループの追加]** を選択します。
1. 次のいずれかの方法を使用して、オブジェクト名 (ユーザー アカウント) を指定します。
   1. オブジェクト名フィールドにユーザー アカウント (`{DOMAIN OR COMPUTER NAME\USER}`) を入力し、 **[OK]** を選択して、ポリシーにユーザーを追加します。
   1. **[詳細]** を選択します。 **[検索開始]** を選択します。 一覧からユーザー アカウントを選択します。 **[OK]** を選択します。 再度 **[OK]** を選択して、ポリシーにユーザーを追加します。
1. **[OK]** または **[適用]** を選択して、変更を受け入れます。

## <a name="create-and-manage-the-windows-service"></a>Windows サービスを作成して管理する

### <a name="create-a-service"></a>サービスを作成する

PowerShell コマンドを使用して、サービスを登録します。 PowerShell 6 の管理コマンド シェルから次のコマンドを実行します。

```powershell
$acl = Get-Acl "{EXE PATH}"
$aclRuleArgs = {DOMAIN OR COMPUTER NAME\USER}, "Read,Write,ReadAndExecute", "ContainerInherit,ObjectInherit", "None", "Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($aclRuleArgs)
$acl.SetAccessRule($accessRule)
$acl | Set-Acl "{EXE PATH}"

New-Service -Name {SERVICE NAME} -BinaryPathName {EXE FILE PATH} -Credential {DOMAIN OR COMPUTER NAME\USER} -Description "{DESCRIPTION}" -DisplayName "{DISPLAY NAME}" -StartupType Automatic
```

* `{EXE PATH}`:ホスト上のアプリのフォルダーへのパス (`d:\myservice` など)。 このパスに、アプリの実行可能ファイルは含めないでください。 末尾のスラッシュは、必要ありません。
* `{DOMAIN OR COMPUTER NAME\USER}`:サービスのユーザー アカウント (`Contoso\ServiceUser` など)。
* `{SERVICE NAME}`:サービス名 (`MyService` など)。
* `{EXE FILE PATH}`:アプリの実行可能ファイルのパス (`d:\myservice\myservice.exe` など)。 拡張子付きの実行可能ファイルのファイル名を含めます。
* `{DESCRIPTION}`:サービスの説明 (`My sample service` など)。
* `{DISPLAY NAME}`:サービスの表示名 (`My Service` など)。

### <a name="start-a-service"></a>サービスを開始する

次の PowerShell 6 コマンドでサービスを開始します。

```powershell
Start-Service -Name {SERVICE NAME}
```

このコマンドでサービスを開始するには数秒かかります。

### <a name="determine-a-services-status"></a>サービスの状態を確認する

サービスの状態を確認するには、次の PowerShell 6 コマンドを使います。

```powershell
Get-Service -Name {SERVICE NAME}
```

この状態は、次のいずれかの値として報告されます。

* `Starting`
* `Running`
* `Stopping`
* `Stopped`

### <a name="stop-a-service"></a>サービスを停止する

次の PowerShell 6 コマンドでサービスを停止します。

```powershell
Stop-Service -Name {SERVICE NAME}
```

### <a name="remove-a-service"></a>サービスを削除する

サービスを停止した後、少ししてから、次の Powershell 6 コマンドを使ってサービスを削除します。

```powershell
Remove-Service -Name {SERVICE NAME}
```

## <a name="handle-starting-and-stopping-events"></a>イベントの開始と停止を扱う

<xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarting*>、<xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarted*>、および <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStopping*> イベントを処理するには、次の処理を実行します。

1. `OnStarting`、`OnStarted`、および `OnStopping` メソッドを使用して、<xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService> から派生するクラスを作成します。

   [!code-csharp[](windows-service/samples/2.x/AspNetCoreService/CustomWebHostService.cs?name=snippet_CustomWebHostService)]

2. `CustomWebHostService` を <xref:System.ServiceProcess.ServiceBase.Run*> に渡す <xref:Microsoft.AspNetCore.Hosting.IWebHost> の拡張メソッドを作成します。

   [!code-csharp[](windows-service/samples/2.x/AspNetCoreService/WebHostServiceExtensions.cs?name=ExtensionsClass)]

3. `Program.Main` で、<xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> ではなく、拡張メソッド `RunAsCustomService` を呼び出します。

   ```csharp
   host.RunAsCustomService();
   ```

   `Program.Main` 内の <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> の場所を確認するには、「[展開の種類](#deployment-type)」セクションに示されているコード サンプルを参照してください。

## <a name="proxy-server-and-load-balancer-scenarios"></a>プロキシ サーバーとロード バランサーのシナリオ

インターネットまたは企業ネットワークからの要求とやり取りするサービスやプロキシまたはロード バランサーの背後にあるサービスでは、追加の構成が必要になる場合があります。 詳細については、「<xref:host-and-deploy/proxy-load-balancer>」を参照してください。

## <a name="configure-endpoints"></a>エンドポイントを構成する

既定では、ASP.NET Core は `http://localhost:5000` にバインドされます。 `ASPNETCORE_URLS` 環境変数を設定して、URL とポートを構成します。

追加の URL とポートの構成方法については、関連するサーバーの記事を参照してください。

* <xref:fundamentals/servers/kestrel#endpoint-configuration>
* <xref:fundamentals/servers/httpsys#configure-windows-server>

上記のガイダンスでは、HTTPS エンドポイントのサポートについて説明しています。 たとえば、Windows サービスで認証が使用されている場合は、アプリを HTTPS 用に構成します。

> [!NOTE]
> サービス エンドポイントをセキュリティで保護するために ASP.NET Core の HTTPS 開発証明書を使用することはできません。

## <a name="current-directory-and-content-root"></a>現在のディレクトリとコンテンツのルート

Windows サービスに対して <xref:System.IO.Directory.GetCurrentDirectory*> を呼び出して返される現在の作業ディレクトリは *C:\\WINDOWS\\system32* フォルダーです。 *system32* フォルダーは、サービスのファイル (設定ファイルなど) を保存するために適した場所ではありません。 次のいずれかの方法を使用して、サービスのアセットと設定ファイルを管理し、アクセスします。

### <a name="set-the-content-root-path-to-the-apps-folder"></a>アプリのフォルダーにコンテンツ ルート パスを設定する

<xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ContentRootPath*> は、サービスの作成時に `binPath` 引数に指定されたものと同じパスです。 `GetCurrentDirectory` を呼び出して設定ファイルへのパスを作成する代わりに、アプリの[コンテンツ ルート](xref:fundamentals/index#content-root)へのパスを指定して <xref:System.IO.Directory.SetCurrentDirectory*> を呼び出します。

`Program.Main` で、サービスの実行可能ファイルがあるフォルダーへのパスを判別し、そのパスを使用してアプリのコンテンツ ルートを確立します。

```csharp
var pathToExe = Process.GetCurrentProcess().MainModule.FileName;
var pathToContentRoot = Path.GetDirectoryName(pathToExe);
Directory.SetCurrentDirectory(pathToContentRoot);

CreateWebHostBuilder(args)
    .Build()
    .RunAsService();
```

### <a name="store-a-services-files-in-a-suitable-location-on-disk"></a>ディスク上の適切な場所にサービスのファイルを格納する

ファイルを含むフォルダーに対して <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> を使用するときは、<xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath*> を使用して絶対パスを指定します。

## <a name="troubleshoot"></a>トラブルシューティング

Windows サービス アプリのトラブルシューティングについては、「<xref:test/troubleshoot>」を参照してください。

### <a name="common-errors"></a>一般的なエラー

* PowerShell の以前のバージョンまたはプレリリース バージョンが使用されています。
* 登録されたサービスに、[dotnet publish](/dotnet/core/tools/dotnet-publish) コマンドからのアプリの **発行済み** 出力が使用されません。 [dotnet build](/dotnet/core/tools/dotnet-build) コマンドの出力が、アプリの展開でサポートされていません。 発行された資産は、展開の種類に応じて次のいずれかのフォルダーにあります。
  * *bin/Release/{TARGET FRAMEWORK}/publish* (FDD)
  * *bin/Release/{TARGET FRAMEWORK}/{RUNTIME IDENTIFIER}/publish* (SCD)
* サービスが実行中の状態ではありません。
* アプリに使用されるリソース (証明書など) のパスが正しくありません。 Windows サービスのベース パスは *c:\\Windows\\System32* です。
* "*サービスとしてログオンする*" アクセス権がユーザーにありません。
* `New-Service` PowerShell コマンドの実行時に、ユーザーのパスワードの有効期限が切れていたか、正しく渡されていません。
* アプリに ASP.NET Core 認証が必要ですが、セキュリティで保護された接続 (HTTPS) 用に構成されていません。
* 要求 URL ポートが正しくないか、アプリで正しく構成されていません。

### <a name="system-and-application-event-logs"></a>システム イベント ログとアプリケーション イベント ログ

システム イベント ログとアプリケーション イベント ログにアクセスします。

1. [スタート] メニューを開き、*イベント ビューアー* を検索し、**イベント ビューアー** アプリを選択します。
1. **イベント ビューアー** で **[Windows ログ]** ノードを開きます。
1. **[システム]** を選択してシステム イベント ログを開きます。 **[Application]** を選択して、アプリケーション イベント ログを開きます。
1. 失敗したアプリに関連するエラーを検索します。

### <a name="run-the-app-at-a-command-prompt"></a>コマンド プロンプトでアプリを実行する

多くのスタートアップ エラーでは、イベント ログに有用な情報が生成されません。 ホスティング システムのコマンド プロンプトでアプリを実行すると、エラーの原因がわかることがあります。 アプリからさらに詳細情報をログに記録するには、[ログ レベル](xref:fundamentals/logging/index#log-level)を低くするか、[開発環境](xref:fundamentals/environments)でアプリを実行します。

### <a name="clear-package-caches"></a>パッケージ キャッシュをクリアする

開発マシンで .NET Core SDK をアップグレードしたり、アプリ内のパッケージ バージョンを変更したりした直後に、機能しているアプリが失敗することがあります。 場合によっては、パッケージに統一性がないと、メジャー アップグレード実行時にアプリが破壊されることがあります。 これらの問題のほとんどは、次の手順で解決できます。

1. *bin* フォルダーと *obj* フォルダーを削除します。
1. コマンド シェルから [dotnet nuget locals all --clear](/dotnet/core/tools/dotnet-nuget-locals) を実行して、パッケージ キャッシュをクリアします。

   パッケージ キャッシュをクリアするには、[nuget.exe](https://www.nuget.org/downloads) ツールを使用して `nuget locals all -clear` コマンドを実行する方法もあります。 *nuget.exe* は、Windows デスクトップ オペレーティング システムにバンドルされているインストールではなく、[NuGet Web サイト](https://www.nuget.org/downloads)から別に入手する必要があります。

1. プロジェクトを復元してリビルドします。
1. アプリを再展開する前に、サーバー上の展開フォルダー内のすべてのファイルを削除します。

### <a name="slow-or-hanging-app"></a>アプリの速度低下またはハング

"*クラッシュ ダンプ*" とはシステムのメモリのスナップショットであり、アプリのクラッシュ、起動の失敗、遅いアプリの原因を突き止めるのに役立ちます。

#### <a name="app-crashes-or-encounters-an-exception"></a>アプリのクラッシュまたは例外の発生

[Windows エラー報告 (WER)](/windows/desktop/wer/windows-error-reporting) からダンプを取得して分析します。

1. `c:\dumps` にクラッシュ ダンプ ファイルを保持するフォルダーを作成します。
1. 次のアプリケーションの実行可能ファイル名で [EnableDumps PowerShell スクリプト](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/host-and-deploy/windows-service/scripts/EnableDumps.ps1)を実行します。

   ```console
   .\EnableDumps {APPLICATION EXE} c:\dumps
   ```

1. クラッシュが発生する条件の下でアプリを実行します。
1. クラッシュが発生した後、[DisableDumps PowerShell スクリプト](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/host-and-deploy/windows-service/scripts/DisableDumps.ps1)を実行します。

   ```console
   .\DisableDumps {APPLICATION EXE}
   ```

アプリがクラッシュし、ダンプの収集が完了したら、アプリを普通に終了してかまいません。 PowerShell スクリプトにより、アプリごとに最大 5 つのダンプを収集する WER が構成されます。

> [!WARNING]
> クラッシュ ダンプでは、大量のディスク領域 (1 つにつき最大、数ギガバイト) が使用される場合があります。

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>アプリが起動時または正常な実行中にハングまたは失敗する

アプリが起動時または正常な実行中に "*ハング*" (応答を停止するがクラッシュしない) または失敗するときは、「[User-Mode Dump Files:Choosing the Best Tool](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool)」(ユーザー モード ダンプ ファイル: 最適なツールの選択) を参照し、適切なツールを選択してダンプを生成します。

#### <a name="analyze-the-dump"></a>ダンプを分析する

ダンプはいくつかの方法で分析できます。 詳細については、「[Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file)」(ユーザー モード ダンプ ファイルの分析) を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* [Kestrel エンドポイント構成](xref:fundamentals/servers/kestrel#endpoint-configuration) (HTTPS 構成と SNI サポートを含む)
* <xref:fundamentals/host/web-host>
* <xref:test/troubleshoot>

::: moniker-end
