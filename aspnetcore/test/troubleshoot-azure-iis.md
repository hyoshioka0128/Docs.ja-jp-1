---
title: Azure App Service および IIS での ASP.NET Core のトラブルシューティング
author: rick-anderson
description: ASP.NET Core アプリの Azure App Service およびインターネット インフォメーション サービス (IIS) のデプロイに関する問題を診断する方法について学習します。
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
uid: test/troubleshoot-azure-iis
ms.openlocfilehash: a8bb659a72e8370c31cc9c213689399e74f11b5c
ms.sourcegitcommit: f77a7467651bab61b24261da9dc5c1dd75fc1fa9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/17/2021
ms.locfileid: "100564061"
---
# <a name="troubleshoot-aspnet-core-on-azure-app-service-and-iis"></a>Azure App Service および IIS での ASP.NET Core のトラブルシューティング

作成者: [Justin Kotalik](https://github.com/jkotalik)

::: moniker range=">= aspnetcore-3.0"

この記事では、アプリの起動時に発生する一般的なエラーに関する情報と、アプリが Azure App Service または IIS にデプロイされるときのエラーを診断する手順を提供します。

[アプリ起動時のエラー](#app-startup-errors)  
一般的な起動時の HTTP 状態コードのシナリオについて説明します。

[Azure App Service でのトラブルシューティング](#troubleshoot-on-azure-app-service)  
Azure App Service にデプロイされたアプリに関するトラブルシューティングのアドバイスを提供します。

[IIS でのトラブルシューティング](#troubleshoot-on-iis)  
IIS にデプロイされているアプリまたは IIS Express でローカルに実行されているアプリに関するトラブルシューティングのアドバイスを提供します。 このガイダンスは、Windows Server と Windows デスクトップの両方の配置に適用されます。

[パッケージ キャッシュをクリアする](#clear-package-caches)  
統一性のないパッケージにより、メジャー アップグレードの実行時またはパッケージ バージョンの変更時に、アプリが中断したときの対処方法について説明します。

[その他のリソース](#additional-resources)  
その他のトラブルシューティングのトピックを一覧表示します。

## <a name="app-startup-errors"></a>アプリ起動時のエラー

Visual Studio では、ASP.NET Core プロジェクトのデバッグ時に [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) のホスティングが既定の設定です。 このトピックのアドバイスを使用すると、ローカルでのデバッグ時に発生する "*502.5 - 処理エラー*" または "*500.30 - 開始エラー*" を診断できます。

### <a name="40314-forbidden"></a>403.14 禁止

アプリの開始に失敗しました。 次のエラーがログに記録されます。

```
The Web server is configured to not list the contents of this directory.
```

このエラーが発生するのは、通常、ホスト システム上の配置が壊れている場合です。これには、次のようなシナリオが含まれます。

* アプリがホスト システム上の不適切なフォルダーに配置されています。
* 配置プロセスで、アプリのすべてのファイルとフォルダーを、ホスティング システムの配置フォルダーに移動できませんでした。
* 配置に *web.config* ファイルがありません。または、*web.config* ファイルの内容の形式が正しくありません。

次の手順に従います。

1. ホスト システム上の配置フォルダーからすべてのファイルとフォルダーを削除します。
1. Visual Studio、PowerShell、手動配置など、通常の配置方法を使用して、アプリの "*公開*" フォルダーの内容をホスティング システムに再配置します。
   * *web.config* ファイルがそのデプロイに存在し、その内容が正しいことを確認します。
   * Azure App Service でホストする場合は、アプリが `D:\home\site\wwwroot` フォルダーにデプロイされていることを確認します。
   * アプリが IIS によってホストされている場合は、アプリが **IIS マネージャー** の **[基本設定]** に表示されている IIS の **物理パス** に配置されていることを確認します。
1. ホスティング システムの配置をプロジェクトの "*公開*" フォルダーの内容と比較して、アプリのすべてのファイルとフォルダーが配置されていることを確認します。

公開された ASP.NET Core アプリのレイアウトの詳細については、「<xref:host-and-deploy/directory-structure>」を参照してください。 *web.config* ファイルの詳細については、「<xref:host-and-deploy/aspnet-core-module#configuration-with-webconfig>」を参照してください。

### <a name="500-internal-server-error"></a>500 内部サーバー エラー

アプリは起動しますが、エラーのためにサーバーは要求を実行できません。

このエラーは、起動時または応答の作成中に、アプリのコード内で発生します。 応答にコンテンツが含まれていないか、またはブラウザーに "*500 内部サーバー エラー*" という応答が表示される可能性があります。 通常、アプリケーション イベント ログではアプリが正常に起動したことが示されます。 サーバーから見るとそれは正しいことです。 アプリは起動しましたが、有効な応答を生成できません。 問題を解決するには、サーバー上のコマンド プロンプトでアプリを実行するか、ASP.NET Core モジュールの stdout ログを有効にします。

### <a name="5000-in-process-handler-load-failure"></a>500.0 インプロセス ハンドラーの読み込みエラー

ワーカー プロセスが失敗します。 アプリは起動しません。

[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module) コンポーネントの読み込み中に不明なエラーが発生しました。 次のいずれかのアクションを実行します。

* [Microsoft サポート](https://support.microsoft.com/oas/default.aspx?prid=15832)に問い合わせます ( **[開発者ツール]** 、 **[ASP.NET Core]** の順に選択します)。
* Stack Overflow について質問します。
* [GitHub リポジトリ](https://github.com/dotnet/AspNetCore)で問題を報告します。

### <a name="50030-in-process-startup-failure"></a>500.30 インプロセス起動エラー

ワーカー プロセスが失敗します。 アプリは起動しません。

[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)は .NET Core CLR の開始をインプロセスで試みますが、開始に失敗します。 プロセス起動時のエラーの原因は、通常、アプリケーション イベント ログと ASP.NET Core モジュールの stdout ログのエントリから判断できます。

一般的なエラー条件:

* 存在しないバージョンの ASP.NET Core 共有フレームワークが対象にされていて、アプリが正しく構成されていません。 対象のコンピューターにどのバージョンの ASP.NET Core 共有フレームワークがインストールされているかを確認します。
* Azure Key Vault を使用している場合、Key Vault へのアクセス許可がありません。 対象の Key Vault のアクセス ポリシーを確認して、正しいアクセス許可が付与されていることを確実にします。

### <a name="50031-ancm-failed-to-find-native-dependencies"></a>500.31 ANCM Failed to Find Native Dependencies (500.31 ANCM ネイティブの依存関係を見つけられませんでした)

ワーカー プロセスが失敗します。 アプリは起動しません。

[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)で .NET Core ランタイムの開始がインプロセスで試行されますが、開始に失敗します。 このスタートアップ エラーの最も一般的な原因は、`Microsoft.NETCore.App` または `Microsoft.AspNetCore.App` ランタイムがインストールされていない場合です。 アプリが ASP.NET Core 3.0 をターゲットとして展開されていて、そのバージョンがコンピューターに存在しない場合、このエラーが発生します。 エラー メッセージの例は次のとおりです。

```
The specified framework 'Microsoft.NETCore.App', version '3.0.0' was not found.
  - The following frameworks were found:
      2.2.1 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
      3.0.0-preview5-27626-15 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
      3.0.0-preview6-27713-13 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
      3.0.0-preview6-27714-15 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
      3.0.0-preview6-27723-08 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
```

エラー メッセージには、インストールされているすべての .NET Core のバージョンとアプリに必要なバージョンが一覧表示されます。 このエラーを修正するには、次のいずれかを実行します。

* 適切なバージョンの .NET Core をマシンにインストールします。
* マシンに存在する .NET Core のバージョンをターゲットにするようにアプリを変更します。
* アプリを[自己完結型の展開](/dotnet/core/deploying/#self-contained-deployments-scd)として発行します。

開発環境で実行している場合 (`ASPNETCORE_ENVIRONMENT` 環境変数が `Development` に設定されている場合)、特定のエラーが HTTP 応答に書き込まれます。 プロセスのスタートアップ エラーの原因は、アプリケーション イベント ログでも見つかります。

### <a name="50032-ancm-failed-to-load-dll"></a>500.32 ANCM Failed to Load dll (500.32 ANCM DLL を読み込めませんでした)

ワーカー プロセスが失敗します。 アプリは起動しません。

このエラーの最も一般的な原因は、アプリが互換性のないプロセッサ アーキテクチャ用に発行されていることです。 ワーカー プロセスが 32 ビットアプリとして実行されていて、そのアプリが 64 ビットをターゲットとして発行されている場合、このエラーが発生します。

このエラーを修正するには、次のいずれかを実行します。

* ワーカー プロセスと同じプロセッサ アーキテクチャ用にアプリを再発行します。
* アプリを[フレームワークに依存する展開](/dotnet/core/deploying/#framework-dependent-executables-fde)として発行します。

### <a name="50033-ancm-request-handler-load-failure"></a>500.33 ANCM Request Handler Load Failure (500.33 ANCM 要求ハンドラーの読み込みエラー)

ワーカー プロセスが失敗します。 アプリは起動しません。

アプリは `Microsoft.AspNetCore.App` フレームワークを参照していませんでした。 [ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)でホストできるのは、`Microsoft.AspNetCore.App` フレームワークをターゲットとしているアプリのみです。

このエラーを修正するには、アプリが `Microsoft.AspNetCore.App` フレームワークをターゲットにしていることを確認します。 アプリがターゲットとしているフレームワークを確認するには、`.runtimeconfig.json` を確認します。

### <a name="50034-ancm-mixed-hosting-models-not-supported"></a>500.34 ANCM Mixed Hosting Models Not Supported (500.34 ANCM 混在ホスティング モデルはサポートされません)

ワーカー プロセスでは、インプロセス アプリとアウト プロセス アプリの両方を同じプロセスで実行できません。

このエラーを修正するには、別の IIS アプリケーション プールでアプリを実行します。

### <a name="50035-ancm-multiple-in-process-applications-in-same-process"></a>500.35 ANCM Multiple In-Process Applications in same Process (500.35 ANCM 同一プロセス内の複数のインプロセス アプリケーション)

ワーカー プロセスによって、同じプロセスで複数のインプロセス アプリを実行することはできません。

このエラーを修正するには、別の IIS アプリケーション プールでアプリを実行します。

### <a name="50036-ancm-out-of-process-handler-load-failure"></a>500.36 ANCM Out-Of-Process Handler Load Failure (500.36 ANCM アウト プロセス ハンドラーの読み込みエラー)

アウト プロセス要求ハンドラーの *aspnetcorev2_outofprocess.dll* が *aspnetcorev2.dll* ファイルの次にありません。 これは、[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)のインストールが破損していることを示しています。

このエラーを修正するには、[.NET Core Hosting Bundle](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle) (IIS 用) または Visual Studio (IIS Express 用) のインストールを修復します。

### <a name="50037-ancm-failed-to-start-within-startup-time-limit"></a>500.37 ANCM Failed to Start Within Startup Time Limit (500.37 ANCM スタートアップ時間の制限内に起動できませんでした)

指定されたスタートアップ時間の制限内に ANCM が起動に失敗しました。 既定では、タイムアウトは 120 秒です。

このエラーは、同じマシン上で多数のアプリを起動したときに発生する可能性があります。 スタートアップ中のサーバー上の CPU/メモリ使用量の急上昇を確認します。 必要に応じて、複数のアプリのスタートアップ プロセスをずらします。

### <a name="50038-ancm-application-dll-not-found"></a>500.38 ANCM アプリケーション DLL が見つかりません

ANCM は、実行可能ファイルの横にあるアプリケーション DLL を見つけることができませんでした。

このエラーは、インプロセス ホスティング モデルを使用して[単一ファイルの実行可能ファイル](/dotnet/core/whats-new/dotnet-core-3-0#single-file-executables)としてパッケージ化されたアプリをホストするときに発生します。 インプロセス モデルでは、ANCM によって既存の IIS プロセスに .NET Core アプリが読み込まれることが必要です。 このシナリオは、単一ファイル展開モデルではサポートされていません。 このエラーを修正するには、アプリのプロジェクト ファイルで、次のうち **1 つ** を使用します。

1. `PublishSingleFile` MSBuild プロパティを `false` に設定して、単一ファイルの公開を無効にします。
1. `AspNetCoreHostingModel` MSBuild プロパティを `OutOfProcess` に設定して、アウトプロセス ホスティング モデルに切り替えます。

### <a name="5025-process-failure"></a>502.5 処理エラー

ワーカー プロセスが失敗します。 アプリは起動しません。

[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)はワーカー プロセスの開始を試みますが、開始に失敗します。 プロセス起動時のエラーの原因は、通常、アプリケーション イベント ログと ASP.NET Core モジュールの stdout ログのエントリから判断できます。

一般的なエラー条件は、存在しないバージョンの ASP.NET Core 共有フレームワークが対象にされていて、アプリが正しく構成されていないことです。 対象のコンピューターにどのバージョンの ASP.NET Core 共有フレームワークがインストールされているかを確認します。 *共有フレームワーク* は、コンピューター上にインストールされたアセンブリ ( *.dll* ファイル) のセットであり、`Microsoft.AspNetCore.App` などのメタパッケージによって参照されます。 メタパッケージの参照には、必要な最低限のバージョンを指定できます。 詳しくは、[共有フレームワーク](https://natemcmaster.com/blog/2018/08/29/netcore-primitives-2/)に関するページをご覧ください。

正しく構成されていないホスティングやアプリが原因でワーカー プロセスが失敗する場合、"*502.5 処理エラー*" のエラー ページが返されます。

### <a name="failed-to-start-application-errorcode-0x800700c1"></a>アプリケーションの起動に失敗しました (ErrorCode '0x800700c1')

```
EventID: 1010
Source: IIS AspNetCore Module V2
Failed to start application '/LM/W3SVC/6/ROOT/', ErrorCode '0x800700c1'.
```

アプリのアセンブリ ( *.dll*) を読み込めなかったため、アプリの起動に失敗しました。

このエラーは、公開されたアプリと w3wp/iisexpress プロセスの間でビットの不一致がある場合に発生します。

アプリ プールの 32 ビット設定が正しいことを確認してください。

1. IIS マネージャーの **[アプリケーション プール]** でそのアプリ プールを選択します。
1. **[アクション]** パネルの **[アプリケーション プールの編集]** で **[詳細設定]** を選択します。
1. **[32 ビット アプリケーションの有効化]** を設定します。
   * 32 ビット (x86) アプリを展開する場合は、この値を `True` に設定します。
   * 64 ビット (x64) アプリを展開する場合は、この値を `False` に設定します。

プロジェクト ファイルの `<Platform>` MSBuild プロパティと公開されたアプリのビット数との間で競合が発生していないことを確認します。

### <a name="connection-reset"></a>接続のリセット

ヘッダー送信後にエラーが発生した場合、サーバーが **500 内部サーバー エラー** を送信するには遅すぎます。 このような状況は、応答に対する複雑なオブジェクトのシリアル化中にエラーが起きたときによく発生します。 この種のエラーは、クライアントでは "*接続リセット*" エラーとして表示されます。 この種のエラーのトラブルシューティングには、[アプリケーション ログ](xref:fundamentals/logging/index)が役に立つことがあります。

### <a name="default-startup-limits"></a>既定の起動制限

[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)の *startupTimeLimit* は、既定では 120 秒に構成されます。 既定値のままにした場合、モジュールで処理エラーが記録されるまでに、アプリは最大で 2 分を起動にかけることができます。 モジュールの構成の詳細については、「[AspNetCore 要素の属性](xref:host-and-deploy/aspnet-core-module#attributes-of-the-aspnetcore-element)」を参照してください。

## <a name="troubleshoot-on-azure-app-service"></a>Azure App Service でのトラブルシューティング

[!INCLUDE [Azure App Service Preview Notice](~/includes/azure-apps-preview-notice.md)]

### <a name="application-event-log-azure-app-service"></a>アプリケーション イベント ログ (Azure App Service)

アプリケーション イベント ログにアクセスするには、Azure portal の **[問題の診断と解決]** ブレードを使います。

1. Azure portal の **[App Services]** でアプリを開きます。
1. **[問題の診断と解決]** を選択します。
1. **[診断ツール]** という見出しを選択します。
1. **[サポート ツール]** で **[アプリケーション イベント]** ボタンを選択します。
1. **[Source]\(ソース\)** 列で、*IIS AspNetCoreModule* または *IIS AspNetCoreModule V2* によって提供された最新のエラーを調べます。

**[問題の診断と解決]** ブレードを使う代わりに、[Kudu](https://github.com/projectkudu/kudu/wiki) を使ってアプリケーション イベント ログ ファイルを直接調べることもできます。

1. **[開発ツール]** 領域で **[高度なツール]** を開きます。 **[Go&rarr;]** ボタンを選びます。 新しいブラウザー タブまたはウィンドウで Kudu コンソールが開きます。
1. ページの上部にあるナビゲーション バーを使って **[デバッグ コンソール]** を開き、 **[CMD]** を選びます。
1. **LogFiles** フォルダーを開きます。
1. *eventlog.xml* ファイルの横にある鉛筆アイコンを選びます。
1. ログを調べます。 ログの末尾までスクロールし、最新のイベントを確認します。

### <a name="run-the-app-in-the-kudu-console"></a>Kudu コンソールでアプリを実行します。

多くの起動時エラーでは、アプリケーション イベント ログに役に立つ情報が生成されません。 [Kudu](https://github.com/projectkudu/kudu/wiki) のリモート実行コンソールでアプリを実行すると、エラーを検出することができます。

1. **[開発ツール]** 領域で **[高度なツール]** を開きます。 **[Go&rarr;]** ボタンを選びます。 新しいブラウザー タブまたはウィンドウで Kudu コンソールが開きます。
1. ページの上部にあるナビゲーション バーを使って **[デバッグ コンソール]** を開き、 **[CMD]** を選びます。

#### <a name="test-a-32-bit-x86-app"></a>32 ビット (x86) アプリをテストする

**現在のリリース**

1. `cd d:\home\site\wwwroot`
1. 次のようにアプリを実行します。
   * アプリが[フレームワークに依存する展開](/dotnet/core/deploying/#framework-dependent-deployments-fdd)の場合:

     ```dotnetcli
     dotnet .\{ASSEMBLY NAME}.dll
     ```

   * アプリが[自己完結型の展開](/dotnet/core/deploying/#self-contained-deployments-scd)の場合:

     ```console
     {ASSEMBLY NAME}.exe
     ```

エラーを示すアプリからのコンソール出力はすべて、Kudu コンソールにパイプされます。

**プレビュー リリース上で実行されているフレームワークに依存するデプロイ**

"*ASP.NET Core {バージョン} (x86) ランタイムのサイト拡張機能をインストールする必要があります。* "

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x32` (`{X.Y}` はランタイム バージョンです)
1. アプリを実行します: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

エラーを示すアプリからのコンソール出力はすべて、Kudu コンソールにパイプされます。

#### <a name="test-a-64-bit-x64-app"></a>64 ビット (x64) アプリをテストする

**現在のリリース**

* アプリが 64 ビット (x64) の[フレームワークに依存する展開](/dotnet/core/deploying/#framework-dependent-deployments-fdd)の場合:
  1. `cd D:\Program Files\dotnet`
  1. アプリを実行します: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`
* アプリが[自己完結型の展開](/dotnet/core/deploying/#self-contained-deployments-scd)の場合:
  1. `cd D:\home\site\wwwroot`
  1. アプリを実行します: `{ASSEMBLY NAME}.exe`

エラーを示すアプリからのコンソール出力はすべて、Kudu コンソールにパイプされます。

**プレビュー リリース上で実行されているフレームワークに依存するデプロイ**

"*ASP.NET Core {バージョン} (x64) ランタイムのサイト拡張機能をインストールする必要があります。* "

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x64` (`{X.Y}` はランタイム バージョンです)
1. アプリを実行します: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

エラーを示すアプリからのコンソール出力はすべて、Kudu コンソールにパイプされます。

### <a name="aspnet-core-module-stdout-log-azure-app-service"></a>ASP.NET Core モジュールの stdout ログ (Azure App Service)

> [!WARNING]
> stdout ログを無効にしないと、アプリまたはサーバーで障害が発生する可能性があります。 ログ ファイルのサイズまたは作成されるログ ファイルの数に制限はありません。 stdout ログは、アプリ起動時の問題のトラブルシューティングにのみ使ってください。
>
> 起動後の ASP.NET Core アプリでの一般的なログの場合は、ログ ファイルのサイズを制限し、ログをローテーションするログ ライブラリを使います。 詳細については、「[サードパーティ製のログ プロバイダー](xref:fundamentals/logging/index#third-party-logging-providers)」を参照してください。

ASP.NET Core モジュールの stdout には、アプリケーション イベント ログでは見つからない有用なエラー メッセージが記録されることがよくあります。 stdout ログを有効にして表示するには:

1. Azure portal で Web アプリに移動します。
1. **[App Service]** ブレードで、検索ボックスに「**kudu**」と入力します。
1. **[高度なツール]** 、 **[Go]** の順に選択します。
1. **[デバッグ コンソール]、[CMD]** の順に選択します。
1. *site/wwwroot* に移動します
1. 鉛筆アイコンを選択し、*web.config* ファイルを編集します。
1. `<aspNetCore />` 要素で `stdoutLogEnabled="true"` を設定し、 **[保存]** を選択します。

トラブルシューティングが完了したら、`stdoutLogEnabled="false"` を設定して stdout ログを無効にします。

詳細については、「<xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>」を参照してください。

<a name="enhanced-diagnostic-logs"></a>

### <a name="aspnet-core-module-debug-log-azure-app-service"></a>ASP.NET Core モジュールのデバッグ ログ (Azure App Service)

ASP.NET Core モジュール デバッグ ログでは、ASP.NET Core モジュールのさらに詳しいログが提供されます。 stdout ログを有効にして表示するには:

1. 強化された診断ログを有効にするには、次のいずれかを実行します。
   * 「[強化された診断ログ](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs)」の指示に従い、強化された診断ログをアプリに対して設定します。 アプリを再デプロイします。
   * Kudu コンソールを利用し、「`<handlerSettings>`強化された診断ログ [」にある](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs) をライブ アプリの *web.config* ファイルに追加します。
     1. **[開発ツール]** 領域で **[高度なツール]** を開きます。 **[Go&rarr;]** ボタンを選びます。 新しいブラウザー タブまたはウィンドウで Kudu コンソールが開きます。
     1. ページの上部にあるナビゲーション バーを使って **[デバッグ コンソール]** を開き、 **[CMD]** を選びます。
     1. パス **site** > **wwwroot** へのフォルダーを開きます。 鉛筆アイコンを選択し、*web.config* ファイルを編集します。 「[強化された診断ログ](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs)」にある `<handlerSettings>` セクションを追加します。 **[保存]** ボタンを選択します。
1. **[開発ツール]** 領域で **[高度なツール]** を開きます。 **[Go&rarr;]** ボタンを選びます。 新しいブラウザー タブまたはウィンドウで Kudu コンソールが開きます。
1. ページの上部にあるナビゲーション バーを使って **[デバッグ コンソール]** を開き、 **[CMD]** を選びます。
1. パス **site** > **wwwroot** へのフォルダーを開きます。 *aspnetcore-debug.log* ファイルにパスを指定しなかった場合、ファイルが一覧に表示されます。 パスを指定した場合、ログ ファイルの場所に移動します。
1. ファイル名の隣にある鉛筆アイコンでログ ファイルを開きます。

トラブルシューティングが完了したら、debug ログを無効にします。

強化されたデバッグ ログを無効にするには、次のいずれかを実行します。

* ローカルの *web.config* ファイルから `<handlerSettings>` を削除し、アプリを再デプロイします。
* Kudu コンソールを使用して *web.config* ファイルを編集し、`<handlerSettings>` セクションを削除します。 ファイルを保存します。

詳細については、「<xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs>」を参照してください。

> [!WARNING]
> debug ログを無効にしないと、アプリまたはサーバーで障害が発生する可能性があります。 ログ ファイルのサイズに制限はありません。 debug ログは、アプリ起動時の問題のトラブルシューティングにのみ使ってください。
>
> 起動後の ASP.NET Core アプリでの一般的なログの場合は、ログ ファイルのサイズを制限し、ログをローテーションするログ ライブラリを使います。 詳細については、「[サードパーティ製のログ プロバイダー](xref:fundamentals/logging/index#third-party-logging-providers)」を参照してください。

### <a name="slow-or-hanging-app-azure-app-service"></a>遅いアプリまたはハングしているアプリ (Azure App Service)

要求に対してアプリの反応が遅い場合、またはハングする場合は、次の記事を参照してください。

* [Azure App Service での Web アプリのパフォーマンス低下に関する問題のトラブルシューティング](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [Use Crash Diagnoser Site Extension to Capture Dump for Intermittent Exception issues or performance issues on Azure Web App (クラッシュ診断サイト拡張機能を使用して Azure Web App での断続的な例外の問題またはパフォーマンスの問題用のダンプをキャプチャする)](https://blogs.msdn.microsoft.com/asiatech/2015/12/28/use-crash-diagnoser-site-extension-to-capture-dump-for-intermittent-exception-issues-or-performance-issues-on-azure-web-app/)

### <a name="monitoring-blades"></a>監視ブレード

監視ブレードでは、このトピックでこれまでに説明した方法に代わるトラブルシューティング エクスペリエンスが提供されます。 これらのブレードは、500 番台のエラーの診断に使用できます。

ASP.NET Core 拡張機能がインストールされていることを確認してください。 拡張機能がインストールされていない場合は、手動でインストールします。

1. **[開発ツール]** ブレード セクションで、 **[拡張機能]** ブレードを選びます。
1. **[ASP.NET Core Extensions]\(ASP.NET Core 拡張機能\)** が一覧に表示されます。
1. 拡張機能がインストールされていない場合は、 **[追加]** ボタンを選びます。
1. 一覧から **[ASP.NET Core Extensions]\(ASP.NET Core 拡張機能\)** を選びます。
1. **[OK]** を選んで法的条項に同意します。
1. **[拡張機能の追加]** ブレードで **[OK]** を選びます。
1. 拡張機能が正常にインストールされると、情報ポップアップ メッセージで通知されます。

stdout ログが有効になっていない場合は、次の手順のようにします。

1. Azure portal の **[開発ツール]** 領域で **[高度なツール]** ブレードを選びます。 **[Go&rarr;]** ボタンを選びます。 新しいブラウザー タブまたはウィンドウで Kudu コンソールが開きます。
1. ページの上部にあるナビゲーション バーを使って **[デバッグ コンソール]** を開き、 **[CMD]** を選びます。
1. パス **site** > **wwwroot** へのフォルダーを開き、下にスクロールして、一覧の最後にある *web.config* ファイルを表示します。
1. *web.config* ファイルの隣の鉛筆アイコンをクリックします。
1. **stdoutLogEnabled** を `true` に設定し、**stdoutLogFile** パスを `\\?\%home%\LogFiles\stdout` に変更します。
1. **[保存]** を選んで、更新した *web.config* ファイルを保存します。

続けて診断ログをアクティブにします。

1. Azure portal で **[診断ログ]** ブレードを選びます。
1. **[アプリケーション ログ (ファイル システム)]** および **[詳細なエラー メッセージ]** の **[オン]** スイッチを選びます。 ブレードの上部にある **[保存]** ボタンを選びます。
1. 失敗した要求イベントのバッファ処理 (FREB) とも呼ばれる失敗した要求のトレースを含めるには、 **[失敗した要求のトレース]** で **[オン]** スイッチを選びます。
1. ポータルで **[診断ログ]** ブレードのすぐ下にある **[ログ ストリーム]** ブレードを選びます。
1. アプリに対して要求します。
1. ログ ストリーム データ内で、エラーの原因が示されます。

トラブルシューティングが完了したら、stdout ログを無効にしてください。

失敗した要求のトレース ログ (FREB ログ) を見るには:

1. Azure portal で **[問題の診断と解決]** ブレードに移動します。
1. サイド バーの **[SUPPORT TOOLS]\(サポート ツール\)** 領域で、 **[Failed Request Tracing Logs]\(失敗した要求のトレース ログ\)** を選びます。

詳しくは、[「Azure App Service の Web アプリの診断ログの有効化」トピックの「失敗した要求トレース」セクション](/azure/app-service/web-sites-enable-diagnostic-log#failed-request-traces)および[「Azure での Web アプリのアプリケーション パフォーマンスに関するよくあるご質問」の「失敗した要求トレースをオンにするにはどうすればよいですか?」](/azure/app-service/app-service-web-availability-performance-application-issues-faq#how-do-i-turn-on-failed-request-tracing)をご覧ください。

詳しくは、「[Azure App Service の Web アプリの診断ログの有効化](/azure/app-service/web-sites-enable-diagnostic-log)」をご覧ください。

> [!WARNING]
> stdout ログを無効にしないと、アプリまたはサーバーで障害が発生する可能性があります。 ログ ファイルのサイズまたは作成されるログ ファイルの数に制限はありません。
>
> ASP.NET Core アプリでのルーチン ログの場合は、ログ ファイルのサイズを制限し、ログをローテーションするログ ライブラリを使います。 詳細については、「[サードパーティ製のログ プロバイダー](xref:fundamentals/logging/index#third-party-logging-providers)」を参照してください。

## <a name="troubleshoot-on-iis"></a>IIS でのトラブルシューティング

### <a name="application-event-log-iis"></a>アプリケーション イベント ログ (IIS)

アプリケーション イベント ログにアクセスします。

1. [スタート] メニューを開き、*イベント ビューアー* を検索し、**イベント ビューアー** アプリを選択します。
1. **イベント ビューアー** で **[Windows ログ]** ノードを開きます。
1. **[Application]** を選択して、アプリケーション イベント ログを開きます。
1. 失敗したアプリに関連するエラーを検索します。 エラーの *[ソース]* 列には *IIS AspNetCore Module* または *IIS Express AspNetCore Module* の値が表示されます。

### <a name="run-the-app-at-a-command-prompt"></a>コマンド プロンプトでアプリを実行する

多くの起動時エラーでは、アプリケーション イベント ログに役に立つ情報が生成されません。 ホスティング システムのコマンド プロンプトでアプリを実行すると、エラーの原因がわかることがあります。

#### <a name="framework-dependent-deployment"></a>フレームワークに依存する展開

アプリが[フレームワークに依存する展開](/dotnet/core/deploying/#framework-dependent-deployments-fdd)の場合:

1. コマンド プロンプトで展開フォルダーに移動し、*dotnet.exe* 使用してアプリのアセンブリを実行して、アプリを実行します。 次のコマンドで、\<assembly_name> をアプリのアセンブリの名前に置き換えます。`dotnet .\<assembly_name>.dll`。
1. エラーを示すアプリからのコンソール出力は、すべてコンソール ウィンドウに書き込まれます。
1. アプリへの要求時にエラーが発生した場合は、Kestrel がリッスンしているホストとポートに要求が送信されます。 既定のホストと post を使用して `http://localhost:5000/` に要求を行います。 アプリが Kestrel のエンドポイント アドレスで正常に応答する場合、問題はホスティングの構成に関連している可能性が高く、アプリ内が原因の可能性は低くなります。

#### <a name="self-contained-deployment"></a>自己完結型の展開

アプリが[自己完結型の展開](/dotnet/core/deploying/#self-contained-deployments-scd)の場合:

1. コマンド プロンプトで、展開フォルダーに移動し、アプリの実行可能ファイルを実行します。 次のコマンドで、\<assembly_name> をアプリのアセンブリの名前に置き換えます。`<assembly_name>.exe`。
1. エラーを示すアプリからのコンソール出力は、すべてコンソール ウィンドウに書き込まれます。
1. アプリへの要求時にエラーが発生した場合は、Kestrel がリッスンしているホストとポートに要求が送信されます。 既定のホストと post を使用して `http://localhost:5000/` に要求を行います。 アプリが Kestrel のエンドポイント アドレスで正常に応答する場合、問題はホスティングの構成に関連している可能性が高く、アプリ内が原因の可能性は低くなります。

### <a name="aspnet-core-module-stdout-log-iis"></a>ASP.NET Core モジュールの stdout ログ (IIS)

stdout ログを有効にして表示するには:

1. ホスティング システム上のサイトの展開フォルダーに移動します。
1. *logs* フォルダーが存在しない場合は、フォルダーを作成します。 MSBuild で展開フォルダーに *logs* フォルダーを自動的に作成されるようにする手順については、「[ディレクトリ構造](xref:host-and-deploy/directory-structure)」のトピックを参照してください。
1. *web.config* ファイルを編集します。 **stdoutLogEnabled** を `true` に設定し、**stdoutLogFile** のパスを *logs* フォルダー (たとえば `.\logs\stdout`) を指すように変更します。 パスの `stdout` はログ ファイル名のプレフィックスです。 ログ ファイルの作成時には、タイムスタンプ、プロセス ID、およびファイルの拡張子が自動的に追加されます。 ファイル名のプレフィックスとして `stdout` を使用すると、一般的なログ ファイルの名前は *stdout_20180205184032_5412.log* になります。
1. アプリケーション プールの ID に *logs* フォルダーへの書き込みアクセス許可があることを確認します。
1. 更新した *web.config* ファイルを保存します。
1. アプリに対して要求します。
1. *logs* フォルダーに移動します。 最新の stdout ログを探して開きます。
1. ログのエラーを調べます。

トラブルシューティングが完了したら、stdout ログを無効にします。

1. *web.config* ファイルを編集します。
1. **stdoutLogEnabled** を `false` に設定します。
1. ファイルを保存します。

詳細については、「<xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>」を参照してください。

> [!WARNING]
> stdout ログを無効にしないと、アプリまたはサーバーで障害が発生する可能性があります。 ログ ファイルのサイズまたは作成されるログ ファイルの数に制限はありません。
>
> ASP.NET Core アプリでのルーチン ログの場合は、ログ ファイルのサイズを制限し、ログをローテーションするログ ライブラリを使います。 詳細については、「[サードパーティ製のログ プロバイダー](xref:fundamentals/logging/index#third-party-logging-providers)」を参照してください。

### <a name="aspnet-core-module-debug-log-iis"></a>ASP.NET Core モジュールのデバッグ ログ (IIS)

ASP.NET Core モジュールのデバッグ ログを有効にするには、次のハンドラー設定をアプリの *web.config* ファイルに追加します。

```xml
<aspNetCore ...>
  <handlerSettings>
    <handlerSetting name="debugLevel" value="file" />
    <handlerSetting name="debugFile" value="c:\temp\ancm.log" />
  </handlerSettings>
</aspNetCore>
```

ログに指定されたパスが存在することと、アプリ プールの ID にその場所への書き込みアクセス許可があることを確認します。

詳細については、「<xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs>」を参照してください。

### <a name="enable-the-developer-exception-page"></a>開発者例外ページを有効にする

開発環境でアプリを実行するには、`ASPNETCORE_ENVIRONMENT` [環境変数を web.config に追加することができます](xref:host-and-deploy/aspnet-core-module#setting-environment-variables)。 アプリの起動時にホスト ビルダー上で `UseEnvironment` によって環境がオーバーライドされない限り、環境変数を設定すると、アプリの実行時に[開発者例外ページ](xref:fundamentals/error-handling)が表示されます。

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout"
      hostingModel="InProcess">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
  </environmentVariables>
</aspNetCore>
```

`ASPNETCORE_ENVIRONMENT` の環境変数を設定する方法は、インターネットに公開されないステージング サーバーやテスト用サーバーの場合にのみお勧めされます。 トラブルシューティング後は、必ず *web.config* ファイルからこの環境変数を削除してください。 *web.config* で環境変数を設定する方法の詳細については、[aspNetCore の environmentVariables 子要素](xref:host-and-deploy/aspnet-core-module#setting-environment-variables)を参照してください。

### <a name="obtain-data-from-an-app"></a>アプリからデータを取得する

アプリが要求に応答できる場合は、ターミナル インライン ミドルウェアを使用して、要求、接続、その他のデータをアプリから取得します。 詳細およびサンプル コードについては、「<xref:test/troubleshoot#obtain-data-from-an-app>」を参照してください。

### <a name="slow-or-hanging-app-iis"></a>遅いアプリまたはハングしているアプリ (IIS)

"*クラッシュ ダンプ*" とはシステムのメモリのスナップショットであり、アプリのクラッシュ、起動の失敗、遅いアプリの原因を突き止めるのに役立ちます。

#### <a name="app-crashes-or-encounters-an-exception"></a>アプリのクラッシュまたは例外の発生

[Windows エラー報告 (WER)](/windows/desktop/wer/windows-error-reporting) からダンプを取得して分析します。

1. `c:\dumps` にクラッシュ ダンプ ファイルを保持するフォルダーを作成します。 アプリ プールには、そのフォルダーへの書き込みアクセス権が必要です。
1. [EnableDumps PowerShell スクリプト](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/test/troubleshoot-azure-iis/scripts/EnableDumps.ps1) を実行します。
   * アプリで [インプロセス ホスティング モデル](xref:host-and-deploy/iis/index#in-process-hosting-model)が使われている場合は、*w3wp.exe* に対するスクリプトを実行します。

     ```console
     .\EnableDumps w3wp.exe c:\dumps
     ```

   * アプリで [アウト プロセス ホスティング モデル](xref:host-and-deploy/iis/index#out-of-process-hosting-model)が使われている場合は、*dotnet.exe* に対するスクリプトを実行します。

     ```console
     .\EnableDumps dotnet.exe c:\dumps
     ```

1. クラッシュが発生する条件の下でアプリを実行します。
1. クラッシュが発生した後、[DisableDumps PowerShell スクリプト](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/test/troubleshoot-azure-iis/scripts/DisableDumps.ps1)を実行します。
   * アプリで [インプロセス ホスティング モデル](xref:host-and-deploy/iis/index#in-process-hosting-model)が使われている場合は、*w3wp.exe* に対するスクリプトを実行します。

     ```console
     .\DisableDumps w3wp.exe
     ```

   * アプリで [アウト プロセス ホスティング モデル](xref:host-and-deploy/iis/index#out-of-process-hosting-model)が使われている場合は、*dotnet.exe* に対するスクリプトを実行します。

     ```console
     .\DisableDumps dotnet.exe
     ```

アプリがクラッシュし、ダンプの収集が完了したら、アプリを普通に終了してかまいません。 PowerShell スクリプトにより、アプリごとに最大 5 つのダンプを収集する WER が構成されます。

> [!WARNING]
> クラッシュ ダンプでは、大量のディスク領域 (1 つにつき最大、数ギガバイト) が使用される場合があります。

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>アプリが起動時または正常な実行中にハングまたは失敗する

アプリが起動時または正常な実行中に "*ハング*" (応答を停止するがクラッシュしない) または失敗するときは、「[User-Mode Dump Files:Choosing the Best Tool](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool)」(ユーザー モード ダンプ ファイル: 最適なツールの選択) を参照し、適切なツールを選択してダンプを生成します。

#### <a name="analyze-the-dump"></a>ダンプを分析する

ダンプはいくつかの方法で分析できます。 詳細については、「[Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file)」(ユーザー モード ダンプ ファイルの分析) を参照してください。

## <a name="clear-package-caches"></a>パッケージ キャッシュをクリアする

開発マシンで .NET Core SDK をアップグレードしたり、アプリ内のパッケージ バージョンを変更したりした直後に、機能しているアプリが失敗することがあります。 場合によっては、パッケージに統一性がないと、メジャー アップグレード実行時にアプリが破壊されることがあります。 これらの問題のほとんどは、次の手順で解決できます。

1. *bin* フォルダーと *obj* フォルダーを削除します。
1. コマンド シェルから [dotnet nuget locals all --clear](/dotnet/core/tools/dotnet-nuget-locals) を実行して、パッケージ キャッシュをクリアします。

   パッケージ キャッシュをクリアするには、[nuget.exe](https://www.nuget.org/downloads) ツールを使用して `nuget locals all -clear` コマンドを実行する方法もあります。 *nuget.exe* は、Windows デスクトップ オペレーティング システムにバンドルされているインストールではなく、[NuGet Web サイト](https://www.nuget.org/downloads)から別に入手する必要があります。

1. プロジェクトを復元してリビルドします。
1. アプリを再展開する前に、サーバー上の展開フォルダー内のすべてのファイルを削除します。

## <a name="additional-resources"></a>その他の技術情報

* <xref:test/troubleshoot>
* <xref:host-and-deploy/azure-iis-errors-reference>
* <xref:fundamentals/error-handling>
* <xref:host-and-deploy/aspnet-core-module>

### <a name="azure-documentation"></a>Azure のドキュメント

* [ASP.NET Core 用 Application Insights](/azure/application-insights/app-insights-asp-net-core)
* [Visual Studio を使用した Azure App Service の Web アプリのトラブルシューティングの Web アプリのリモート デバッグ セクション](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio#remotedebug)
* [Azure App Service の診断の概要](/azure/app-service/app-service-diagnostics)
* [方法: Azure App Service でアプリを監視する](/azure/app-service/web-sites-monitor)
* [Visual Studio を使用した Azure App Service での Web アプリのトラブルシューティング](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio)
* [Azure Web Apps での "502 bad gateway" および "503 service unavailable" の HTTP エラーのトラブルシューティング](/azure/app-service/app-service-web-troubleshoot-http-502-http-503)
* [Azure App Service での Web アプリのパフォーマンス低下に関する問題のトラブルシューティング](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [Azure での Web アプリのアプリケーションパフォーマンスに関するよくあるご質問](/azure/app-service/app-service-web-availability-performance-application-issues-faq)
* [Azure Web アプリのサンドボックス (App Service ランタイムの実行の制限)](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox)
* [Azure Friday: Azure App Service の診断とトラブルシューティング (12 分間のビデオ)](https://channel9.msdn.com/Shows/Azure-Friday/Azure-App-Service-Diagnostic-and-Troubleshooting-Experience)

### <a name="visual-studio-documentation"></a>Visual Studio ドキュメント

* [Visual Studio 2017 における Azure の IIS での ASP.NET Core のリモート デバッグ](/visualstudio/debugger/remote-debugging-azure)
* [Visual Studio 2017 におけるリモートの IIS コンピューターでの ASP.NET Core のリモート デバッグ](/visualstudio/debugger/remote-debugging-aspnet-on-a-remote-iis-computer)
* [Visual Studio を使用したデバッグについて理解する](/visualstudio/debugger/getting-started-with-the-debugger)

### <a name="visual-studio-code-documentation"></a>Visual Studio Code ドキュメント

* [Visual Studio Code でのデバッグ](https://code.visualstudio.com/docs/editor/debugging)

::: moniker-end

::: moniker range="= aspnetcore-2.2"

この記事では、アプリの起動時に発生する一般的なエラーに関する情報と、アプリが Azure App Service または IIS にデプロイされるときのエラーを診断する手順を提供します。

[アプリ起動時のエラー](#app-startup-errors)  
一般的な起動時の HTTP 状態コードのシナリオについて説明します。

[Azure App Service でのトラブルシューティング](#troubleshoot-on-azure-app-service)  
Azure App Service にデプロイされたアプリに関するトラブルシューティングのアドバイスを提供します。

[IIS でのトラブルシューティング](#troubleshoot-on-iis)  
IIS にデプロイされているアプリまたは IIS Express でローカルに実行されているアプリに関するトラブルシューティングのアドバイスを提供します。 このガイダンスは、Windows Server と Windows デスクトップの両方の配置に適用されます。

[パッケージ キャッシュをクリアする](#clear-package-caches)  
統一性のないパッケージにより、メジャー アップグレードの実行時またはパッケージ バージョンの変更時に、アプリが中断したときの対処方法について説明します。

[その他のリソース](#additional-resources)  
その他のトラブルシューティングのトピックを一覧表示します。

## <a name="app-startup-errors"></a>アプリ起動時のエラー

Visual Studio では、ASP.NET Core プロジェクトのデバッグ時に [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) のホスティングが既定の設定です。 このトピックのアドバイスを使用すると、ローカルでのデバッグ時に発生する "*502.5 - 処理エラー*" または "*500.30 - 開始エラー*" を診断できます。

### <a name="40314-forbidden"></a>403.14 禁止

アプリの開始に失敗しました。 次のエラーがログに記録されます。

```
The Web server is configured to not list the contents of this directory.
```

このエラーが発生するのは、通常、ホスト システム上の配置が壊れている場合です。これには、次のようなシナリオが含まれます。

* アプリがホスト システム上の不適切なフォルダーに配置されています。
* 配置プロセスで、アプリのすべてのファイルとフォルダーを、ホスティング システムの配置フォルダーに移動できませんでした。
* 配置に *web.config* ファイルがありません。または、*web.config* ファイルの内容の形式が正しくありません。

次の手順に従います。

1. ホスト システム上の配置フォルダーからすべてのファイルとフォルダーを削除します。
1. Visual Studio、PowerShell、手動配置など、通常の配置方法を使用して、アプリの "*公開*" フォルダーの内容をホスティング システムに再配置します。
   * *web.config* ファイルがそのデプロイに存在し、その内容が正しいことを確認します。
   * Azure App Service でホストする場合は、アプリが `D:\home\site\wwwroot` フォルダーにデプロイされていることを確認します。
   * アプリが IIS によってホストされている場合は、アプリが **IIS マネージャー** の **[基本設定]** に表示されている IIS の **物理パス** に配置されていることを確認します。
1. ホスティング システムの配置をプロジェクトの "*公開*" フォルダーの内容と比較して、アプリのすべてのファイルとフォルダーが配置されていることを確認します。

公開された ASP.NET Core アプリのレイアウトの詳細については、「<xref:host-and-deploy/directory-structure>」を参照してください。 *web.config* ファイルの詳細については、「<xref:host-and-deploy/aspnet-core-module#configuration-with-webconfig>」を参照してください。

### <a name="500-internal-server-error"></a>500 内部サーバー エラー

アプリは起動しますが、エラーのためにサーバーは要求を実行できません。

このエラーは、起動時または応答の作成中に、アプリのコード内で発生します。 応答にコンテンツが含まれていないか、またはブラウザーに "*500 内部サーバー エラー*" という応答が表示される可能性があります。 通常、アプリケーション イベント ログではアプリが正常に起動したことが示されます。 サーバーから見るとそれは正しいことです。 アプリは起動しましたが、有効な応答を生成できません。 問題を解決するには、サーバー上のコマンド プロンプトでアプリを実行するか、ASP.NET Core モジュールの stdout ログを有効にします。

### <a name="5000-in-process-handler-load-failure"></a>500.0 インプロセス ハンドラーの読み込みエラー

ワーカー プロセスが失敗します。 アプリは起動しません。

[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)は .NET Core CLR の検出に失敗し、インプロセス要求ハンドラー (*aspnetcorev2_inprocess.dll*) を検出します。 次の点をご確認ください。

* アプリが [Microsoft.AspNetCore.Server.IIS](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.IIS) NuGet パッケージまたは [Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)を対象としている。
* アプリが対象としているバージョンの ASP.NET Core 共有フレームワークが対象のコンピューターにインストールされている。

### <a name="5000-out-of-process-handler-load-failure"></a>500.0 アウト プロセス ハンドラーの読み込みエラー

ワーカー プロセスが失敗します。 アプリは起動しません。

[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)は、アウト プロセスのホスティング要求ハンドラーの検索に失敗します。 *aspnetcorev2_outofprocess.dll* が *aspnetcorev2.dll* の隣のサブフォルダーにあることを確認してください。

### <a name="5025-process-failure"></a>502.5 処理エラー

ワーカー プロセスが失敗します。 アプリは起動しません。

[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)はワーカー プロセスの開始を試みますが、開始に失敗します。 プロセス起動時のエラーの原因は、通常、アプリケーション イベント ログと ASP.NET Core モジュールの stdout ログのエントリから判断できます。

一般的なエラー条件は、存在しないバージョンの ASP.NET Core 共有フレームワークが対象にされていて、アプリが正しく構成されていないことです。 対象のコンピューターにどのバージョンの ASP.NET Core 共有フレームワークがインストールされているかを確認します。 *共有フレームワーク* は、コンピューター上にインストールされたアセンブリ ( *.dll* ファイル) のセットであり、`Microsoft.AspNetCore.App` などのメタパッケージによって参照されます。 メタパッケージの参照には、必要な最低限のバージョンを指定できます。 詳しくは、[共有フレームワーク](https://natemcmaster.com/blog/2018/08/29/netcore-primitives-2/)に関するページをご覧ください。

正しく構成されていないホスティングやアプリが原因でワーカー プロセスが失敗する場合、"*502.5 処理エラー*" のエラー ページが返されます。

### <a name="failed-to-start-application-errorcode-0x800700c1"></a>アプリケーションの起動に失敗しました (ErrorCode '0x800700c1')

```
EventID: 1010
Source: IIS AspNetCore Module V2
Failed to start application '/LM/W3SVC/6/ROOT/', ErrorCode '0x800700c1'.
```

アプリのアセンブリ ( *.dll*) を読み込めなかったため、アプリの起動に失敗しました。

このエラーは、公開されたアプリと w3wp/iisexpress プロセスの間でビットの不一致がある場合に発生します。

アプリ プールの 32 ビット設定が正しいことを確認してください。

1. IIS マネージャーの **[アプリケーション プール]** でそのアプリ プールを選択します。
1. **[アクション]** パネルの **[アプリケーション プールの編集]** で **[詳細設定]** を選択します。
1. **[32 ビット アプリケーションの有効化]** を設定します。
   * 32 ビット (x86) アプリを展開する場合は、この値を `True` に設定します。
   * 64 ビット (x64) アプリを展開する場合は、この値を `False` に設定します。

プロジェクト ファイルの `<Platform>` MSBuild プロパティと公開されたアプリのビット数との間で競合が発生していないことを確認します。

### <a name="connection-reset"></a>接続のリセット

ヘッダー送信後にエラーが発生した場合、サーバーが **500 内部サーバー エラー** を送信するには遅すぎます。 このような状況は、応答に対する複雑なオブジェクトのシリアル化中にエラーが起きたときによく発生します。 この種のエラーは、クライアントでは "*接続リセット*" エラーとして表示されます。 この種のエラーのトラブルシューティングには、[アプリケーション ログ](xref:fundamentals/logging/index)が役に立つことがあります。

### <a name="default-startup-limits"></a>既定の起動制限

[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)の *startupTimeLimit* は、既定では 120 秒に構成されます。 既定値のままにした場合、モジュールで処理エラーが記録されるまでに、アプリは最大で 2 分を起動にかけることができます。 モジュールの構成の詳細については、「[AspNetCore 要素の属性](xref:host-and-deploy/aspnet-core-module#attributes-of-the-aspnetcore-element)」を参照してください。

## <a name="troubleshoot-on-azure-app-service"></a>Azure App Service でのトラブルシューティング

[!INCLUDE [Azure App Service Preview Notice](~/includes/azure-apps-preview-notice.md)]

### <a name="application-event-log-azure-app-service"></a>アプリケーション イベント ログ (Azure App Service)

アプリケーション イベント ログにアクセスするには、Azure portal の **[問題の診断と解決]** ブレードを使います。

1. Azure portal の **[App Services]** でアプリを開きます。
1. **[問題の診断と解決]** を選択します。
1. **[診断ツール]** という見出しを選択します。
1. **[サポート ツール]** で **[アプリケーション イベント]** ボタンを選択します。
1. **[Source]\(ソース\)** 列で、*IIS AspNetCoreModule* または *IIS AspNetCoreModule V2* によって提供された最新のエラーを調べます。

**[問題の診断と解決]** ブレードを使う代わりに、[Kudu](https://github.com/projectkudu/kudu/wiki) を使ってアプリケーション イベント ログ ファイルを直接調べることもできます。

1. **[開発ツール]** 領域で **[高度なツール]** を開きます。 **[Go&rarr;]** ボタンを選びます。 新しいブラウザー タブまたはウィンドウで Kudu コンソールが開きます。
1. ページの上部にあるナビゲーション バーを使って **[デバッグ コンソール]** を開き、 **[CMD]** を選びます。
1. **LogFiles** フォルダーを開きます。
1. *eventlog.xml* ファイルの横にある鉛筆アイコンを選びます。
1. ログを調べます。 ログの末尾までスクロールし、最新のイベントを確認します。

### <a name="run-the-app-in-the-kudu-console"></a>Kudu コンソールでアプリを実行します。

多くの起動時エラーでは、アプリケーション イベント ログに役に立つ情報が生成されません。 [Kudu](https://github.com/projectkudu/kudu/wiki) のリモート実行コンソールでアプリを実行すると、エラーを検出することができます。

1. **[開発ツール]** 領域で **[高度なツール]** を開きます。 **[Go&rarr;]** ボタンを選びます。 新しいブラウザー タブまたはウィンドウで Kudu コンソールが開きます。
1. ページの上部にあるナビゲーション バーを使って **[デバッグ コンソール]** を開き、 **[CMD]** を選びます。

#### <a name="test-a-32-bit-x86-app"></a>32 ビット (x86) アプリをテストする

**現在のリリース**

1. `cd d:\home\site\wwwroot`
1. 次のようにアプリを実行します。
   * アプリが[フレームワークに依存する展開](/dotnet/core/deploying/#framework-dependent-deployments-fdd)の場合:

     ```dotnetcli
     dotnet .\{ASSEMBLY NAME}.dll
     ```

   * アプリが[自己完結型の展開](/dotnet/core/deploying/#self-contained-deployments-scd)の場合:

     ```console
     {ASSEMBLY NAME}.exe
     ```

エラーを示すアプリからのコンソール出力はすべて、Kudu コンソールにパイプされます。

**プレビュー リリース上で実行されているフレームワークに依存するデプロイ**

"*ASP.NET Core {バージョン} (x86) ランタイムのサイト拡張機能をインストールする必要があります。* "

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x32` (`{X.Y}` はランタイム バージョンです)
1. アプリを実行します: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

エラーを示すアプリからのコンソール出力はすべて、Kudu コンソールにパイプされます。

#### <a name="test-a-64-bit-x64-app"></a>64 ビット (x64) アプリをテストする

**現在のリリース**

* アプリが 64 ビット (x64) の[フレームワークに依存する展開](/dotnet/core/deploying/#framework-dependent-deployments-fdd)の場合:
  1. `cd D:\Program Files\dotnet`
  1. アプリを実行します: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`
* アプリが[自己完結型の展開](/dotnet/core/deploying/#self-contained-deployments-scd)の場合:
  1. `cd D:\home\site\wwwroot`
  1. アプリを実行します: `{ASSEMBLY NAME}.exe`

エラーを示すアプリからのコンソール出力はすべて、Kudu コンソールにパイプされます。

**プレビュー リリース上で実行されているフレームワークに依存するデプロイ**

"*ASP.NET Core {バージョン} (x64) ランタイムのサイト拡張機能をインストールする必要があります。* "

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x64` (`{X.Y}` はランタイム バージョンです)
1. アプリを実行します: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

エラーを示すアプリからのコンソール出力はすべて、Kudu コンソールにパイプされます。

### <a name="aspnet-core-module-stdout-log-azure-app-service"></a>ASP.NET Core モジュールの stdout ログ (Azure App Service)

ASP.NET Core モジュールの stdout には、アプリケーション イベント ログでは見つからない有用なエラー メッセージが記録されることがよくあります。 stdout ログを有効にして表示するには:

1. Azure portal で **[問題の診断と解決]** ブレードに移動します。
1. **[SELECT PROBLEM CATEGORY]\(問題カテゴリの選択\)** で、 **[Web App Down]\(Web アプリのダウン\)** ボタンを選びます。
1. **[推奨されている解決方法]** > **[Enable Stdout Log Redirection]\(Stdout ログのリダイレクトを有効にする\)** で、 **[Open Kudu Console to edit Web.Config]\(Kudu コンソールを開いて Web.Config を編集する\)** ボタンを選びます。
1. Kudu の **診断コンソール** で、パス **site** > **wwwroot** へのフォルダーを開きます。 下にスクロールして、一覧の最後にある *web.config* ファイルを表示します。
1. *web.config* ファイルの隣の鉛筆アイコンをクリックします。
1. **stdoutLogEnabled** を `true` に設定し、**stdoutLogFile** パスを `\\?\%home%\LogFiles\stdout` に変更します。
1. **[保存]** を選んで、更新した *web.config* ファイルを保存します。
1. アプリに対して要求します。
1. Azure portal に戻ります。 **[開発ツール]** 領域で **[高度なツール]** ブレードを選びます。 **[Go&rarr;]** ボタンを選びます。 新しいブラウザー タブまたはウィンドウで Kudu コンソールが開きます。
1. ページの上部にあるナビゲーション バーを使って **[デバッグ コンソール]** を開き、 **[CMD]** を選びます。
1. **LogFiles** フォルダーを選びます。
1. **[Modified]\(変更日\)** 列を調べて、変更日が最新の stdout ログの鉛筆アイコンを選んで編集します。
1. ログ ファイルが開くと、エラーが表示されます。

トラブルシューティングが完了したら、stdout ログを無効にします。

1. Kudu の **診断コンソール** で、パス **site** > **wwwroot** に戻り、*web.config* ファイルを表示します。 鉛筆アイコンを選んで **web.config** ファイルを再び開きます。
1. **stdoutLogEnabled** を `false` に設定します。
1. **[保存]** を選んでファイルを保存します。

詳細については、「<xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>」を参照してください。

> [!WARNING]
> stdout ログを無効にしないと、アプリまたはサーバーで障害が発生する可能性があります。 ログ ファイルのサイズまたは作成されるログ ファイルの数に制限はありません。 stdout ログは、アプリ起動時の問題のトラブルシューティングにのみ使ってください。
>
> 起動後の ASP.NET Core アプリでの一般的なログの場合は、ログ ファイルのサイズを制限し、ログをローテーションするログ ライブラリを使います。 詳細については、「[サードパーティ製のログ プロバイダー](xref:fundamentals/logging/index#third-party-logging-providers)」を参照してください。

### <a name="aspnet-core-module-debug-log-azure-app-service"></a>ASP.NET Core モジュールのデバッグ ログ (Azure App Service)

ASP.NET Core モジュール デバッグ ログでは、ASP.NET Core モジュールのさらに詳しいログが提供されます。 stdout ログを有効にして表示するには:

1. 強化された診断ログを有効にするには、次のいずれかを実行します。
   * 「[強化された診断ログ](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs)」の指示に従い、強化された診断ログをアプリに対して設定します。 アプリを再デプロイします。
   * Kudu コンソールを利用し、「`<handlerSettings>`強化された診断ログ [」にある](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs) をライブ アプリの *web.config* ファイルに追加します。
     1. **[開発ツール]** 領域で **[高度なツール]** を開きます。 **[Go&rarr;]** ボタンを選びます。 新しいブラウザー タブまたはウィンドウで Kudu コンソールが開きます。
     1. ページの上部にあるナビゲーション バーを使って **[デバッグ コンソール]** を開き、 **[CMD]** を選びます。
     1. パス **site** > **wwwroot** へのフォルダーを開きます。 鉛筆アイコンを選択し、*web.config* ファイルを編集します。 「[強化された診断ログ](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs)」にある `<handlerSettings>` セクションを追加します。 **[保存]** ボタンを選択します。
1. **[開発ツール]** 領域で **[高度なツール]** を開きます。 **[Go&rarr;]** ボタンを選びます。 新しいブラウザー タブまたはウィンドウで Kudu コンソールが開きます。
1. ページの上部にあるナビゲーション バーを使って **[デバッグ コンソール]** を開き、 **[CMD]** を選びます。
1. パス **site** > **wwwroot** へのフォルダーを開きます。 *aspnetcore-debug.log* ファイルにパスを指定しなかった場合、ファイルが一覧に表示されます。 パスを指定した場合、ログ ファイルの場所に移動します。
1. ファイル名の隣にある鉛筆アイコンでログ ファイルを開きます。

トラブルシューティングが完了したら、debug ログを無効にします。

強化されたデバッグ ログを無効にするには、次のいずれかを実行します。

* ローカルの *web.config* ファイルから `<handlerSettings>` を削除し、アプリを再デプロイします。
* Kudu コンソールを使用して *web.config* ファイルを編集し、`<handlerSettings>` セクションを削除します。 ファイルを保存します。

詳細については、「<xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs>」を参照してください。

> [!WARNING]
> debug ログを無効にしないと、アプリまたはサーバーで障害が発生する可能性があります。 ログ ファイルのサイズに制限はありません。 debug ログは、アプリ起動時の問題のトラブルシューティングにのみ使ってください。
>
> 起動後の ASP.NET Core アプリでの一般的なログの場合は、ログ ファイルのサイズを制限し、ログをローテーションするログ ライブラリを使います。 詳細については、「[サードパーティ製のログ プロバイダー](xref:fundamentals/logging/index#third-party-logging-providers)」を参照してください。

### <a name="slow-or-hanging-app-azure-app-service"></a>遅いアプリまたはハングしているアプリ (Azure App Service)

要求に対してアプリの反応が遅い場合、またはハングする場合は、次の記事を参照してください。

* [Azure App Service での Web アプリのパフォーマンス低下に関する問題のトラブルシューティング](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [Use Crash Diagnoser Site Extension to Capture Dump for Intermittent Exception issues or performance issues on Azure Web App (クラッシュ診断サイト拡張機能を使用して Azure Web App での断続的な例外の問題またはパフォーマンスの問題用のダンプをキャプチャする)](https://blogs.msdn.microsoft.com/asiatech/2015/12/28/use-crash-diagnoser-site-extension-to-capture-dump-for-intermittent-exception-issues-or-performance-issues-on-azure-web-app/)

### <a name="monitoring-blades"></a>監視ブレード

監視ブレードでは、このトピックでこれまでに説明した方法に代わるトラブルシューティング エクスペリエンスが提供されます。 これらのブレードは、500 番台のエラーの診断に使用できます。

ASP.NET Core 拡張機能がインストールされていることを確認してください。 拡張機能がインストールされていない場合は、手動でインストールします。

1. **[開発ツール]** ブレード セクションで、 **[拡張機能]** ブレードを選びます。
1. **[ASP.NET Core Extensions]\(ASP.NET Core 拡張機能\)** が一覧に表示されます。
1. 拡張機能がインストールされていない場合は、 **[追加]** ボタンを選びます。
1. 一覧から **[ASP.NET Core Extensions]\(ASP.NET Core 拡張機能\)** を選びます。
1. **[OK]** を選んで法的条項に同意します。
1. **[拡張機能の追加]** ブレードで **[OK]** を選びます。
1. 拡張機能が正常にインストールされると、情報ポップアップ メッセージで通知されます。

stdout ログが有効になっていない場合は、次の手順のようにします。

1. Azure portal の **[開発ツール]** 領域で **[高度なツール]** ブレードを選びます。 **[Go&rarr;]** ボタンを選びます。 新しいブラウザー タブまたはウィンドウで Kudu コンソールが開きます。
1. ページの上部にあるナビゲーション バーを使って **[デバッグ コンソール]** を開き、 **[CMD]** を選びます。
1. パス **site** > **wwwroot** へのフォルダーを開き、下にスクロールして、一覧の最後にある *web.config* ファイルを表示します。
1. *web.config* ファイルの隣の鉛筆アイコンをクリックします。
1. **stdoutLogEnabled** を `true` に設定し、**stdoutLogFile** パスを `\\?\%home%\LogFiles\stdout` に変更します。
1. **[保存]** を選んで、更新した *web.config* ファイルを保存します。

続けて診断ログをアクティブにします。

1. Azure portal で **[診断ログ]** ブレードを選びます。
1. **[アプリケーション ログ (ファイル システム)]** および **[詳細なエラー メッセージ]** の **[オン]** スイッチを選びます。 ブレードの上部にある **[保存]** ボタンを選びます。
1. 失敗した要求イベントのバッファ処理 (FREB) とも呼ばれる失敗した要求のトレースを含めるには、 **[失敗した要求のトレース]** で **[オン]** スイッチを選びます。
1. ポータルで **[診断ログ]** ブレードのすぐ下にある **[ログ ストリーム]** ブレードを選びます。
1. アプリに対して要求します。
1. ログ ストリーム データ内で、エラーの原因が示されます。

トラブルシューティングが完了したら、stdout ログを無効にしてください。

失敗した要求のトレース ログ (FREB ログ) を見るには:

1. Azure portal で **[問題の診断と解決]** ブレードに移動します。
1. サイド バーの **[SUPPORT TOOLS]\(サポート ツール\)** 領域で、 **[Failed Request Tracing Logs]\(失敗した要求のトレース ログ\)** を選びます。

詳しくは、[「Azure App Service の Web アプリの診断ログの有効化」トピックの「失敗した要求トレース」セクション](/azure/app-service/web-sites-enable-diagnostic-log#failed-request-traces)および[「Azure での Web アプリのアプリケーション パフォーマンスに関するよくあるご質問」の「失敗した要求トレースをオンにするにはどうすればよいですか?」](/azure/app-service/app-service-web-availability-performance-application-issues-faq#how-do-i-turn-on-failed-request-tracing)をご覧ください。

詳しくは、「[Azure App Service の Web アプリの診断ログの有効化](/azure/app-service/web-sites-enable-diagnostic-log)」をご覧ください。

> [!WARNING]
> stdout ログを無効にしないと、アプリまたはサーバーで障害が発生する可能性があります。 ログ ファイルのサイズまたは作成されるログ ファイルの数に制限はありません。
>
> ASP.NET Core アプリでのルーチン ログの場合は、ログ ファイルのサイズを制限し、ログをローテーションするログ ライブラリを使います。 詳細については、「[サードパーティ製のログ プロバイダー](xref:fundamentals/logging/index#third-party-logging-providers)」を参照してください。

## <a name="troubleshoot-on-iis"></a>IIS でのトラブルシューティング

### <a name="application-event-log-iis"></a>アプリケーション イベント ログ (IIS)

アプリケーション イベント ログにアクセスします。

1. [スタート] メニューを開き、*イベント ビューアー* を検索し、**イベント ビューアー** アプリを選択します。
1. **イベント ビューアー** で **[Windows ログ]** ノードを開きます。
1. **[Application]** を選択して、アプリケーション イベント ログを開きます。
1. 失敗したアプリに関連するエラーを検索します。 エラーの *[ソース]* 列には *IIS AspNetCore Module* または *IIS Express AspNetCore Module* の値が表示されます。

### <a name="run-the-app-at-a-command-prompt"></a>コマンド プロンプトでアプリを実行する

多くの起動時エラーでは、アプリケーション イベント ログに役に立つ情報が生成されません。 ホスティング システムのコマンド プロンプトでアプリを実行すると、エラーの原因がわかることがあります。

#### <a name="framework-dependent-deployment"></a>フレームワークに依存する展開

アプリが[フレームワークに依存する展開](/dotnet/core/deploying/#framework-dependent-deployments-fdd)の場合:

1. コマンド プロンプトで展開フォルダーに移動し、*dotnet.exe* 使用してアプリのアセンブリを実行して、アプリを実行します。 次のコマンドで、\<assembly_name> をアプリのアセンブリの名前に置き換えます。`dotnet .\<assembly_name>.dll`。
1. エラーを示すアプリからのコンソール出力は、すべてコンソール ウィンドウに書き込まれます。
1. アプリへの要求時にエラーが発生した場合は、Kestrel がリッスンしているホストとポートに要求が送信されます。 既定のホストと post を使用して `http://localhost:5000/` に要求を行います。 アプリが Kestrel のエンドポイント アドレスで正常に応答する場合、問題はホスティングの構成に関連している可能性が高く、アプリ内が原因の可能性は低くなります。

#### <a name="self-contained-deployment"></a>自己完結型の展開

アプリが[自己完結型の展開](/dotnet/core/deploying/#self-contained-deployments-scd)の場合:

1. コマンド プロンプトで、展開フォルダーに移動し、アプリの実行可能ファイルを実行します。 次のコマンドで、\<assembly_name> をアプリのアセンブリの名前に置き換えます。`<assembly_name>.exe`。
1. エラーを示すアプリからのコンソール出力は、すべてコンソール ウィンドウに書き込まれます。
1. アプリへの要求時にエラーが発生した場合は、Kestrel がリッスンしているホストとポートに要求が送信されます。 既定のホストと post を使用して `http://localhost:5000/` に要求を行います。 アプリが Kestrel のエンドポイント アドレスで正常に応答する場合、問題はホスティングの構成に関連している可能性が高く、アプリ内が原因の可能性は低くなります。

### <a name="aspnet-core-module-stdout-log-iis"></a>ASP.NET Core モジュールの stdout ログ (IIS)

stdout ログを有効にして表示するには:

1. ホスティング システム上のサイトの展開フォルダーに移動します。
1. *logs* フォルダーが存在しない場合は、フォルダーを作成します。 MSBuild で展開フォルダーに *logs* フォルダーを自動的に作成されるようにする手順については、「[ディレクトリ構造](xref:host-and-deploy/directory-structure)」のトピックを参照してください。
1. *web.config* ファイルを編集します。 **stdoutLogEnabled** を `true` に設定し、**stdoutLogFile** のパスを *logs* フォルダー (たとえば `.\logs\stdout`) を指すように変更します。 パスの `stdout` はログ ファイル名のプレフィックスです。 ログ ファイルの作成時には、タイムスタンプ、プロセス ID、およびファイルの拡張子が自動的に追加されます。 ファイル名のプレフィックスとして `stdout` を使用すると、一般的なログ ファイルの名前は *stdout_20180205184032_5412.log* になります。
1. アプリケーション プールの ID に *logs* フォルダーへの書き込みアクセス許可があることを確認します。
1. 更新した *web.config* ファイルを保存します。
1. アプリに対して要求します。
1. *logs* フォルダーに移動します。 最新の stdout ログを探して開きます。
1. ログのエラーを調べます。

トラブルシューティングが完了したら、stdout ログを無効にします。

1. *web.config* ファイルを編集します。
1. **stdoutLogEnabled** を `false` に設定します。
1. ファイルを保存します。

詳細については、「<xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>」を参照してください。

> [!WARNING]
> stdout ログを無効にしないと、アプリまたはサーバーで障害が発生する可能性があります。 ログ ファイルのサイズまたは作成されるログ ファイルの数に制限はありません。
>
> ASP.NET Core アプリでのルーチン ログの場合は、ログ ファイルのサイズを制限し、ログをローテーションするログ ライブラリを使います。 詳細については、「[サードパーティ製のログ プロバイダー](xref:fundamentals/logging/index#third-party-logging-providers)」を参照してください。

### <a name="aspnet-core-module-debug-log-iis"></a>ASP.NET Core モジュールのデバッグ ログ (IIS)

ASP.NET Core モジュールのデバッグ ログを有効にするには、次のハンドラー設定をアプリの *web.config* ファイルに追加します。

```xml
<aspNetCore ...>
  <handlerSettings>
    <handlerSetting name="debugLevel" value="file" />
    <handlerSetting name="debugFile" value="c:\temp\ancm.log" />
  </handlerSettings>
</aspNetCore>
```

ログに指定されたパスが存在することと、アプリ プールの ID にその場所への書き込みアクセス許可があることを確認します。

詳細については、「<xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs>」を参照してください。

### <a name="enable-the-developer-exception-page"></a>開発者例外ページを有効にする

開発環境でアプリを実行するには、`ASPNETCORE_ENVIRONMENT` [環境変数を web.config に追加することができます](xref:host-and-deploy/aspnet-core-module#setting-environment-variables)。 アプリの起動時にホスト ビルダー上で `UseEnvironment` によって環境がオーバーライドされない限り、環境変数を設定すると、アプリの実行時に[開発者例外ページ](xref:fundamentals/error-handling)が表示されます。

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout"
      hostingModel="InProcess">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
  </environmentVariables>
</aspNetCore>
```

`ASPNETCORE_ENVIRONMENT` の環境変数を設定する方法は、インターネットに公開されないステージング サーバーやテスト用サーバーの場合にのみお勧めされます。 トラブルシューティング後は、必ず *web.config* ファイルからこの環境変数を削除してください。 *web.config* で環境変数を設定する方法の詳細については、[aspNetCore の environmentVariables 子要素](xref:host-and-deploy/aspnet-core-module#setting-environment-variables)を参照してください。

### <a name="obtain-data-from-an-app"></a>アプリからデータを取得する

アプリが要求に応答できる場合は、ターミナル インライン ミドルウェアを使用して、要求、接続、その他のデータをアプリから取得します。 詳細およびサンプル コードについては、「<xref:test/troubleshoot#obtain-data-from-an-app>」を参照してください。

### <a name="slow-or-hanging-app-iis"></a>遅いアプリまたはハングしているアプリ (IIS)

"*クラッシュ ダンプ*" とはシステムのメモリのスナップショットであり、アプリのクラッシュ、起動の失敗、遅いアプリの原因を突き止めるのに役立ちます。

#### <a name="app-crashes-or-encounters-an-exception"></a>アプリのクラッシュまたは例外の発生

[Windows エラー報告 (WER)](/windows/desktop/wer/windows-error-reporting) からダンプを取得して分析します。

1. `c:\dumps` にクラッシュ ダンプ ファイルを保持するフォルダーを作成します。 アプリ プールには、そのフォルダーへの書き込みアクセス権が必要です。
1. [EnableDumps PowerShell スクリプト](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/test/troubleshoot-azure-iis/scripts/EnableDumps.ps1) を実行します。
   * アプリで [インプロセス ホスティング モデル](xref:host-and-deploy/iis/index#in-process-hosting-model)が使われている場合は、*w3wp.exe* に対するスクリプトを実行します。

     ```console
     .\EnableDumps w3wp.exe c:\dumps
     ```

   * アプリで [アウト プロセス ホスティング モデル](xref:host-and-deploy/iis/index#out-of-process-hosting-model)が使われている場合は、*dotnet.exe* に対するスクリプトを実行します。

     ```console
     .\EnableDumps dotnet.exe c:\dumps
     ```

1. クラッシュが発生する条件の下でアプリを実行します。
1. クラッシュが発生した後、[DisableDumps PowerShell スクリプト](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/test/troubleshoot-azure-iis/scripts/DisableDumps.ps1)を実行します。
   * アプリで [インプロセス ホスティング モデル](xref:host-and-deploy/iis/index#in-process-hosting-model)が使われている場合は、*w3wp.exe* に対するスクリプトを実行します。

     ```console
     .\DisableDumps w3wp.exe
     ```

   * アプリで [アウト プロセス ホスティング モデル](xref:host-and-deploy/iis/index#out-of-process-hosting-model)が使われている場合は、*dotnet.exe* に対するスクリプトを実行します。

     ```console
     .\DisableDumps dotnet.exe
     ```

アプリがクラッシュし、ダンプの収集が完了したら、アプリを普通に終了してかまいません。 PowerShell スクリプトにより、アプリごとに最大 5 つのダンプを収集する WER が構成されます。

> [!WARNING]
> クラッシュ ダンプでは、大量のディスク領域 (1 つにつき最大、数ギガバイト) が使用される場合があります。

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>アプリが起動時または正常な実行中にハングまたは失敗する

アプリが起動時または正常な実行中に "*ハング*" (応答を停止するがクラッシュしない) または失敗するときは、「[User-Mode Dump Files:Choosing the Best Tool](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool)」(ユーザー モード ダンプ ファイル: 最適なツールの選択) を参照し、適切なツールを選択してダンプを生成します。

#### <a name="analyze-the-dump"></a>ダンプを分析する

ダンプはいくつかの方法で分析できます。 詳細については、「[Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file)」(ユーザー モード ダンプ ファイルの分析) を参照してください。

## <a name="clear-package-caches"></a>パッケージ キャッシュをクリアする

開発マシンで .NET Core SDK をアップグレードしたり、アプリ内のパッケージ バージョンを変更したりした直後に、機能しているアプリが失敗することがあります。 場合によっては、パッケージに統一性がないと、メジャー アップグレード実行時にアプリが破壊されることがあります。 これらの問題のほとんどは、次の手順で解決できます。

1. *bin* フォルダーと *obj* フォルダーを削除します。
1. コマンド シェルから [dotnet nuget locals all --clear](/dotnet/core/tools/dotnet-nuget-locals) を実行して、パッケージ キャッシュをクリアします。

   パッケージ キャッシュをクリアするには、[nuget.exe](https://www.nuget.org/downloads) ツールを使用して `nuget locals all -clear` コマンドを実行する方法もあります。 *nuget.exe* は、Windows デスクトップ オペレーティング システムにバンドルされているインストールではなく、[NuGet Web サイト](https://www.nuget.org/downloads)から別に入手する必要があります。

1. プロジェクトを復元してリビルドします。
1. アプリを再展開する前に、サーバー上の展開フォルダー内のすべてのファイルを削除します。

## <a name="additional-resources"></a>その他の技術情報

* <xref:test/troubleshoot>
* <xref:host-and-deploy/azure-iis-errors-reference>
* <xref:fundamentals/error-handling>
* <xref:host-and-deploy/aspnet-core-module>

### <a name="azure-documentation"></a>Azure のドキュメント

* [ASP.NET Core 用 Application Insights](/azure/application-insights/app-insights-asp-net-core)
* [Visual Studio を使用した Azure App Service の Web アプリのトラブルシューティングの Web アプリのリモート デバッグ セクション](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio#remotedebug)
* [Azure App Service の診断の概要](/azure/app-service/app-service-diagnostics)
* [方法: Azure App Service でアプリを監視する](/azure/app-service/web-sites-monitor)
* [Visual Studio を使用した Azure App Service での Web アプリのトラブルシューティング](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio)
* [Azure Web Apps での "502 bad gateway" および "503 service unavailable" の HTTP エラーのトラブルシューティング](/azure/app-service/app-service-web-troubleshoot-http-502-http-503)
* [Azure App Service での Web アプリのパフォーマンス低下に関する問題のトラブルシューティング](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [Azure での Web アプリのアプリケーションパフォーマンスに関するよくあるご質問](/azure/app-service/app-service-web-availability-performance-application-issues-faq)
* [Azure Web アプリのサンドボックス (App Service ランタイムの実行の制限)](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox)
* [Azure Friday: Azure App Service の診断とトラブルシューティング (12 分間のビデオ)](https://channel9.msdn.com/Shows/Azure-Friday/Azure-App-Service-Diagnostic-and-Troubleshooting-Experience)

### <a name="visual-studio-documentation"></a>Visual Studio ドキュメント

* [Visual Studio 2017 における Azure の IIS での ASP.NET Core のリモート デバッグ](/visualstudio/debugger/remote-debugging-azure)
* [Visual Studio 2017 におけるリモートの IIS コンピューターでの ASP.NET Core のリモート デバッグ](/visualstudio/debugger/remote-debugging-aspnet-on-a-remote-iis-computer)
* [Visual Studio を使用したデバッグについて理解する](/visualstudio/debugger/getting-started-with-the-debugger)

### <a name="visual-studio-code-documentation"></a>Visual Studio Code ドキュメント

* [Visual Studio Code でのデバッグ](https://code.visualstudio.com/docs/editor/debugging)

::: moniker-end

::: moniker range="< aspnetcore-2.2"

この記事では、アプリの起動時に発生する一般的なエラーに関する情報と、アプリが Azure App Service または IIS にデプロイされるときのエラーを診断する手順を提供します。

[アプリ起動時のエラー](#app-startup-errors)  
一般的な起動時の HTTP 状態コードのシナリオについて説明します。

[Azure App Service でのトラブルシューティング](#troubleshoot-on-azure-app-service)  
Azure App Service にデプロイされたアプリに関するトラブルシューティングのアドバイスを提供します。

[IIS でのトラブルシューティング](#troubleshoot-on-iis)  
IIS にデプロイされているアプリまたは IIS Express でローカルに実行されているアプリに関するトラブルシューティングのアドバイスを提供します。 このガイダンスは、Windows Server と Windows デスクトップの両方の配置に適用されます。

[パッケージ キャッシュをクリアする](#clear-package-caches)  
統一性のないパッケージにより、メジャー アップグレードの実行時またはパッケージ バージョンの変更時に、アプリが中断したときの対処方法について説明します。

[その他のリソース](#additional-resources)  
その他のトラブルシューティングのトピックを一覧表示します。

## <a name="app-startup-errors"></a>アプリ起動時のエラー

Visual Studio では、ASP.NET Core プロジェクトのデバッグ時に [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) のホスティングが既定の設定です。 このトピックのアドバイスを使用すると、ローカルでのデバッグ時に発生する "*502.5 処理エラー*" を診断できます。

### <a name="40314-forbidden"></a>403.14 禁止

アプリの開始に失敗しました。 次のエラーがログに記録されます。

```
The Web server is configured to not list the contents of this directory.
```

このエラーが発生するのは、通常、ホスト システム上の配置が壊れている場合です。これには、次のようなシナリオが含まれます。

* アプリがホスト システム上の不適切なフォルダーに配置されています。
* 配置プロセスで、アプリのすべてのファイルとフォルダーを、ホスティング システムの配置フォルダーに移動できませんでした。
* 配置に *web.config* ファイルがありません。または、*web.config* ファイルの内容の形式が正しくありません。

次の手順に従います。

1. ホスト システム上の配置フォルダーからすべてのファイルとフォルダーを削除します。
1. Visual Studio、PowerShell、手動配置など、通常の配置方法を使用して、アプリの "*公開*" フォルダーの内容をホスティング システムに再配置します。
   * *web.config* ファイルがそのデプロイに存在し、その内容が正しいことを確認します。
   * Azure App Service でホストする場合は、アプリが `D:\home\site\wwwroot` フォルダーにデプロイされていることを確認します。
   * アプリが IIS によってホストされている場合は、アプリが **IIS マネージャー** の **[基本設定]** に表示されている IIS の **物理パス** に配置されていることを確認します。
1. ホスティング システムの配置をプロジェクトの "*公開*" フォルダーの内容と比較して、アプリのすべてのファイルとフォルダーが配置されていることを確認します。

公開された ASP.NET Core アプリのレイアウトの詳細については、「<xref:host-and-deploy/directory-structure>」を参照してください。 *web.config* ファイルの詳細については、「<xref:host-and-deploy/aspnet-core-module#configuration-with-webconfig>」を参照してください。

### <a name="500-internal-server-error"></a>500 内部サーバー エラー

アプリは起動しますが、エラーのためにサーバーは要求を実行できません。

このエラーは、起動時または応答の作成中に、アプリのコード内で発生します。 応答にコンテンツが含まれていないか、またはブラウザーに "*500 内部サーバー エラー*" という応答が表示される可能性があります。 通常、アプリケーション イベント ログではアプリが正常に起動したことが示されます。 サーバーから見るとそれは正しいことです。 アプリは起動しましたが、有効な応答を生成できません。 問題を解決するには、サーバー上のコマンド プロンプトでアプリを実行するか、ASP.NET Core モジュールの stdout ログを有効にします。

### <a name="5025-process-failure"></a>502.5 処理エラー

ワーカー プロセスが失敗します。 アプリは起動しません。

[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)はワーカー プロセスの開始を試みますが、開始に失敗します。 プロセス起動時のエラーの原因は、通常、アプリケーション イベント ログと ASP.NET Core モジュールの stdout ログのエントリから判断できます。

一般的なエラー条件は、存在しないバージョンの ASP.NET Core 共有フレームワークが対象にされていて、アプリが正しく構成されていないことです。 対象のコンピューターにどのバージョンの ASP.NET Core 共有フレームワークがインストールされているかを確認します。 *共有フレームワーク* は、コンピューター上にインストールされたアセンブリ ( *.dll* ファイル) のセットであり、`Microsoft.AspNetCore.App` などのメタパッケージによって参照されます。 メタパッケージの参照には、必要な最低限のバージョンを指定できます。 詳しくは、[共有フレームワーク](https://natemcmaster.com/blog/2018/08/29/netcore-primitives-2/)に関するページをご覧ください。

正しく構成されていないホスティングやアプリが原因でワーカー プロセスが失敗する場合、"*502.5 処理エラー*" のエラー ページが返されます。

### <a name="failed-to-start-application-errorcode-0x800700c1"></a>アプリケーションの起動に失敗しました (ErrorCode '0x800700c1')

```
EventID: 1010
Source: IIS AspNetCore Module V2
Failed to start application '/LM/W3SVC/6/ROOT/', ErrorCode '0x800700c1'.
```

アプリのアセンブリ ( *.dll*) を読み込めなかったため、アプリの起動に失敗しました。

このエラーは、公開されたアプリと w3wp/iisexpress プロセスの間でビットの不一致がある場合に発生します。

アプリ プールの 32 ビット設定が正しいことを確認してください。

1. IIS マネージャーの **[アプリケーション プール]** でそのアプリ プールを選択します。
1. **[アクション]** パネルの **[アプリケーション プールの編集]** で **[詳細設定]** を選択します。
1. **[32 ビット アプリケーションの有効化]** を設定します。
   * 32 ビット (x86) アプリを展開する場合は、この値を `True` に設定します。
   * 64 ビット (x64) アプリを展開する場合は、この値を `False` に設定します。

プロジェクト ファイルの `<Platform>` MSBuild プロパティと公開されたアプリのビット数との間で競合が発生していないことを確認します。

### <a name="connection-reset"></a>接続のリセット

ヘッダー送信後にエラーが発生した場合、サーバーが **500 内部サーバー エラー** を送信するには遅すぎます。 このような状況は、応答に対する複雑なオブジェクトのシリアル化中にエラーが起きたときによく発生します。 この種のエラーは、クライアントでは "*接続リセット*" エラーとして表示されます。 この種のエラーのトラブルシューティングには、[アプリケーション ログ](xref:fundamentals/logging/index)が役に立つことがあります。

### <a name="default-startup-limits"></a>既定の起動制限

[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)の *startupTimeLimit* は、既定では 120 秒に構成されます。 既定値のままにした場合、モジュールで処理エラーが記録されるまでに、アプリは最大で 2 分を起動にかけることができます。 モジュールの構成の詳細については、「[AspNetCore 要素の属性](xref:host-and-deploy/aspnet-core-module#attributes-of-the-aspnetcore-element)」を参照してください。

## <a name="troubleshoot-on-azure-app-service"></a>Azure App Service でのトラブルシューティング

[!INCLUDE [Azure App Service Preview Notice](~/includes/azure-apps-preview-notice.md)]

### <a name="application-event-log-azure-app-service"></a>アプリケーション イベント ログ (Azure App Service)

アプリケーション イベント ログにアクセスするには、Azure portal の **[問題の診断と解決]** ブレードを使います。

1. Azure portal の **[App Services]** でアプリを開きます。
1. **[問題の診断と解決]** を選択します。
1. **[診断ツール]** という見出しを選択します。
1. **[サポート ツール]** で **[アプリケーション イベント]** ボタンを選択します。
1. **[Source]\(ソース\)** 列で、*IIS AspNetCoreModule* または *IIS AspNetCoreModule V2* によって提供された最新のエラーを調べます。

**[問題の診断と解決]** ブレードを使う代わりに、[Kudu](https://github.com/projectkudu/kudu/wiki) を使ってアプリケーション イベント ログ ファイルを直接調べることもできます。

1. **[開発ツール]** 領域で **[高度なツール]** を開きます。 **[Go&rarr;]** ボタンを選びます。 新しいブラウザー タブまたはウィンドウで Kudu コンソールが開きます。
1. ページの上部にあるナビゲーション バーを使って **[デバッグ コンソール]** を開き、 **[CMD]** を選びます。
1. **LogFiles** フォルダーを開きます。
1. *eventlog.xml* ファイルの横にある鉛筆アイコンを選びます。
1. ログを調べます。 ログの末尾までスクロールし、最新のイベントを確認します。

### <a name="run-the-app-in-the-kudu-console"></a>Kudu コンソールでアプリを実行します。

多くの起動時エラーでは、アプリケーション イベント ログに役に立つ情報が生成されません。 [Kudu](https://github.com/projectkudu/kudu/wiki) のリモート実行コンソールでアプリを実行すると、エラーを検出することができます。

1. **[開発ツール]** 領域で **[高度なツール]** を開きます。 **[Go&rarr;]** ボタンを選びます。 新しいブラウザー タブまたはウィンドウで Kudu コンソールが開きます。
1. ページの上部にあるナビゲーション バーを使って **[デバッグ コンソール]** を開き、 **[CMD]** を選びます。

#### <a name="test-a-32-bit-x86-app"></a>32 ビット (x86) アプリをテストする

**現在のリリース**

1. `cd d:\home\site\wwwroot`
1. 次のようにアプリを実行します。
   * アプリが[フレームワークに依存する展開](/dotnet/core/deploying/#framework-dependent-deployments-fdd)の場合:

     ```dotnetcli
     dotnet .\{ASSEMBLY NAME}.dll
     ```

   * アプリが[自己完結型の展開](/dotnet/core/deploying/#self-contained-deployments-scd)の場合:

     ```console
     {ASSEMBLY NAME}.exe
     ```

エラーを示すアプリからのコンソール出力はすべて、Kudu コンソールにパイプされます。

**プレビュー リリース上で実行されているフレームワークに依存するデプロイ**

"*ASP.NET Core {バージョン} (x86) ランタイムのサイト拡張機能をインストールする必要があります。* "

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x32` (`{X.Y}` はランタイム バージョンです)
1. アプリを実行します: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

エラーを示すアプリからのコンソール出力はすべて、Kudu コンソールにパイプされます。

#### <a name="test-a-64-bit-x64-app"></a>64 ビット (x64) アプリをテストする

**現在のリリース**

* アプリが 64 ビット (x64) の[フレームワークに依存する展開](/dotnet/core/deploying/#framework-dependent-deployments-fdd)の場合:
  1. `cd D:\Program Files\dotnet`
  1. アプリを実行します: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`
* アプリが[自己完結型の展開](/dotnet/core/deploying/#self-contained-deployments-scd)の場合:
  1. `cd D:\home\site\wwwroot`
  1. アプリを実行します: `{ASSEMBLY NAME}.exe`

エラーを示すアプリからのコンソール出力はすべて、Kudu コンソールにパイプされます。

**プレビュー リリース上で実行されているフレームワークに依存するデプロイ**

"*ASP.NET Core {バージョン} (x64) ランタイムのサイト拡張機能をインストールする必要があります。* "

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x64` (`{X.Y}` はランタイム バージョンです)
1. アプリを実行します: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

エラーを示すアプリからのコンソール出力はすべて、Kudu コンソールにパイプされます。

### <a name="aspnet-core-module-stdout-log-azure-app-service"></a>ASP.NET Core モジュールの stdout ログ (Azure App Service)

ASP.NET Core モジュールの stdout には、アプリケーション イベント ログでは見つからない有用なエラー メッセージが記録されることがよくあります。 stdout ログを有効にして表示するには:

1. Azure portal で **[問題の診断と解決]** ブレードに移動します。
1. **[SELECT PROBLEM CATEGORY]\(問題カテゴリの選択\)** で、 **[Web App Down]\(Web アプリのダウン\)** ボタンを選びます。
1. **[推奨されている解決方法]** > **[Enable Stdout Log Redirection]\(Stdout ログのリダイレクトを有効にする\)** で、 **[Open Kudu Console to edit Web.Config]\(Kudu コンソールを開いて Web.Config を編集する\)** ボタンを選びます。
1. Kudu の **診断コンソール** で、パス **site** > **wwwroot** へのフォルダーを開きます。 下にスクロールして、一覧の最後にある *web.config* ファイルを表示します。
1. *web.config* ファイルの隣の鉛筆アイコンをクリックします。
1. **stdoutLogEnabled** を `true` に設定し、**stdoutLogFile** パスを `\\?\%home%\LogFiles\stdout` に変更します。
1. **[保存]** を選んで、更新した *web.config* ファイルを保存します。
1. アプリに対して要求します。
1. Azure portal に戻ります。 **[開発ツール]** 領域で **[高度なツール]** ブレードを選びます。 **[Go&rarr;]** ボタンを選びます。 新しいブラウザー タブまたはウィンドウで Kudu コンソールが開きます。
1. ページの上部にあるナビゲーション バーを使って **[デバッグ コンソール]** を開き、 **[CMD]** を選びます。
1. **LogFiles** フォルダーを選びます。
1. **[Modified]\(変更日\)** 列を調べて、変更日が最新の stdout ログの鉛筆アイコンを選んで編集します。
1. ログ ファイルが開くと、エラーが表示されます。

トラブルシューティングが完了したら、stdout ログを無効にします。

1. Kudu の **診断コンソール** で、パス **site** > **wwwroot** に戻り、*web.config* ファイルを表示します。 鉛筆アイコンを選んで **web.config** ファイルを再び開きます。
1. **stdoutLogEnabled** を `false` に設定します。
1. **[保存]** を選んでファイルを保存します。

詳細については、「<xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>」を参照してください。

> [!WARNING]
> stdout ログを無効にしないと、アプリまたはサーバーで障害が発生する可能性があります。 ログ ファイルのサイズまたは作成されるログ ファイルの数に制限はありません。 stdout ログは、アプリ起動時の問題のトラブルシューティングにのみ使ってください。
>
> 起動後の ASP.NET Core アプリでの一般的なログの場合は、ログ ファイルのサイズを制限し、ログをローテーションするログ ライブラリを使います。 詳細については、「[サードパーティ製のログ プロバイダー](xref:fundamentals/logging/index#third-party-logging-providers)」を参照してください。

### <a name="slow-or-hanging-app-azure-app-service"></a>遅いアプリまたはハングしているアプリ (Azure App Service)

要求に対してアプリの反応が遅い場合、またはハングする場合は、次の記事を参照してください。

* [Azure App Service での Web アプリのパフォーマンス低下に関する問題のトラブルシューティング](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [Use Crash Diagnoser Site Extension to Capture Dump for Intermittent Exception issues or performance issues on Azure Web App (クラッシュ診断サイト拡張機能を使用して Azure Web App での断続的な例外の問題またはパフォーマンスの問題用のダンプをキャプチャする)](https://blogs.msdn.microsoft.com/asiatech/2015/12/28/use-crash-diagnoser-site-extension-to-capture-dump-for-intermittent-exception-issues-or-performance-issues-on-azure-web-app/)

### <a name="monitoring-blades"></a>監視ブレード

監視ブレードでは、このトピックでこれまでに説明した方法に代わるトラブルシューティング エクスペリエンスが提供されます。 これらのブレードは、500 番台のエラーの診断に使用できます。

ASP.NET Core 拡張機能がインストールされていることを確認してください。 拡張機能がインストールされていない場合は、手動でインストールします。

1. **[開発ツール]** ブレード セクションで、 **[拡張機能]** ブレードを選びます。
1. **[ASP.NET Core Extensions]\(ASP.NET Core 拡張機能\)** が一覧に表示されます。
1. 拡張機能がインストールされていない場合は、 **[追加]** ボタンを選びます。
1. 一覧から **[ASP.NET Core Extensions]\(ASP.NET Core 拡張機能\)** を選びます。
1. **[OK]** を選んで法的条項に同意します。
1. **[拡張機能の追加]** ブレードで **[OK]** を選びます。
1. 拡張機能が正常にインストールされると、情報ポップアップ メッセージで通知されます。

stdout ログが有効になっていない場合は、次の手順のようにします。

1. Azure portal の **[開発ツール]** 領域で **[高度なツール]** ブレードを選びます。 **[Go&rarr;]** ボタンを選びます。 新しいブラウザー タブまたはウィンドウで Kudu コンソールが開きます。
1. ページの上部にあるナビゲーション バーを使って **[デバッグ コンソール]** を開き、 **[CMD]** を選びます。
1. パス **site** > **wwwroot** へのフォルダーを開き、下にスクロールして、一覧の最後にある *web.config* ファイルを表示します。
1. *web.config* ファイルの隣の鉛筆アイコンをクリックします。
1. **stdoutLogEnabled** を `true` に設定し、**stdoutLogFile** パスを `\\?\%home%\LogFiles\stdout` に変更します。
1. **[保存]** を選んで、更新した *web.config* ファイルを保存します。

続けて診断ログをアクティブにします。

1. Azure portal で **[診断ログ]** ブレードを選びます。
1. **[アプリケーション ログ (ファイル システム)]** および **[詳細なエラー メッセージ]** の **[オン]** スイッチを選びます。 ブレードの上部にある **[保存]** ボタンを選びます。
1. 失敗した要求イベントのバッファ処理 (FREB) とも呼ばれる失敗した要求のトレースを含めるには、 **[失敗した要求のトレース]** で **[オン]** スイッチを選びます。
1. ポータルで **[診断ログ]** ブレードのすぐ下にある **[ログ ストリーム]** ブレードを選びます。
1. アプリに対して要求します。
1. ログ ストリーム データ内で、エラーの原因が示されます。

トラブルシューティングが完了したら、stdout ログを無効にしてください。

失敗した要求のトレース ログ (FREB ログ) を見るには:

1. Azure portal で **[問題の診断と解決]** ブレードに移動します。
1. サイド バーの **[SUPPORT TOOLS]\(サポート ツール\)** 領域で、 **[Failed Request Tracing Logs]\(失敗した要求のトレース ログ\)** を選びます。

詳しくは、[「Azure App Service の Web アプリの診断ログの有効化」トピックの「失敗した要求トレース」セクション](/azure/app-service/web-sites-enable-diagnostic-log#failed-request-traces)および[「Azure での Web アプリのアプリケーション パフォーマンスに関するよくあるご質問」の「失敗した要求トレースをオンにするにはどうすればよいですか?」](/azure/app-service/app-service-web-availability-performance-application-issues-faq#how-do-i-turn-on-failed-request-tracing)をご覧ください。

詳しくは、「[Azure App Service の Web アプリの診断ログの有効化](/azure/app-service/web-sites-enable-diagnostic-log)」をご覧ください。

> [!WARNING]
> stdout ログを無効にしないと、アプリまたはサーバーで障害が発生する可能性があります。 ログ ファイルのサイズまたは作成されるログ ファイルの数に制限はありません。
>
> ASP.NET Core アプリでのルーチン ログの場合は、ログ ファイルのサイズを制限し、ログをローテーションするログ ライブラリを使います。 詳細については、「[サードパーティ製のログ プロバイダー](xref:fundamentals/logging/index#third-party-logging-providers)」を参照してください。

## <a name="troubleshoot-on-iis"></a>IIS でのトラブルシューティング

### <a name="application-event-log-iis"></a>アプリケーション イベント ログ (IIS)

アプリケーション イベント ログにアクセスします。

1. [スタート] メニューを開き、*イベント ビューアー* を検索し、**イベント ビューアー** アプリを選択します。
1. **イベント ビューアー** で **[Windows ログ]** ノードを開きます。
1. **[Application]** を選択して、アプリケーション イベント ログを開きます。
1. 失敗したアプリに関連するエラーを検索します。 エラーの *[ソース]* 列には *IIS AspNetCore Module* または *IIS Express AspNetCore Module* の値が表示されます。

### <a name="run-the-app-at-a-command-prompt"></a>コマンド プロンプトでアプリを実行する

多くの起動時エラーでは、アプリケーション イベント ログに役に立つ情報が生成されません。 ホスティング システムのコマンド プロンプトでアプリを実行すると、エラーの原因がわかることがあります。

#### <a name="framework-dependent-deployment"></a>フレームワークに依存する展開

アプリが[フレームワークに依存する展開](/dotnet/core/deploying/#framework-dependent-deployments-fdd)の場合:

1. コマンド プロンプトで展開フォルダーに移動し、*dotnet.exe* 使用してアプリのアセンブリを実行して、アプリを実行します。 次のコマンドで、\<assembly_name> をアプリのアセンブリの名前に置き換えます。`dotnet .\<assembly_name>.dll`。
1. エラーを示すアプリからのコンソール出力は、すべてコンソール ウィンドウに書き込まれます。
1. アプリへの要求時にエラーが発生した場合は、Kestrel がリッスンしているホストとポートに要求が送信されます。 既定のホストと post を使用して `http://localhost:5000/` に要求を行います。 アプリが Kestrel のエンドポイント アドレスで正常に応答する場合、問題はホスティングの構成に関連している可能性が高く、アプリ内が原因の可能性は低くなります。

#### <a name="self-contained-deployment"></a>自己完結型の展開

アプリが[自己完結型の展開](/dotnet/core/deploying/#self-contained-deployments-scd)の場合:

1. コマンド プロンプトで、展開フォルダーに移動し、アプリの実行可能ファイルを実行します。 次のコマンドで、\<assembly_name> をアプリのアセンブリの名前に置き換えます。`<assembly_name>.exe`。
1. エラーを示すアプリからのコンソール出力は、すべてコンソール ウィンドウに書き込まれます。
1. アプリへの要求時にエラーが発生した場合は、Kestrel がリッスンしているホストとポートに要求が送信されます。 既定のホストと post を使用して `http://localhost:5000/` に要求を行います。 アプリが Kestrel のエンドポイント アドレスで正常に応答する場合、問題はホスティングの構成に関連している可能性が高く、アプリ内が原因の可能性は低くなります。

### <a name="aspnet-core-module-stdout-log-iis"></a>ASP.NET Core モジュールの stdout ログ (IIS)

stdout ログを有効にして表示するには:

1. ホスティング システム上のサイトの展開フォルダーに移動します。
1. *logs* フォルダーが存在しない場合は、フォルダーを作成します。 MSBuild で展開フォルダーに *logs* フォルダーを自動的に作成されるようにする手順については、「[ディレクトリ構造](xref:host-and-deploy/directory-structure)」のトピックを参照してください。
1. *web.config* ファイルを編集します。 **stdoutLogEnabled** を `true` に設定し、**stdoutLogFile** のパスを *logs* フォルダー (たとえば `.\logs\stdout`) を指すように変更します。 パスの `stdout` はログ ファイル名のプレフィックスです。 ログ ファイルの作成時には、タイムスタンプ、プロセス ID、およびファイルの拡張子が自動的に追加されます。 ファイル名のプレフィックスとして `stdout` を使用すると、一般的なログ ファイルの名前は *stdout_20180205184032_5412.log* になります。
1. アプリケーション プールの ID に *logs* フォルダーへの書き込みアクセス許可があることを確認します。
1. 更新した *web.config* ファイルを保存します。
1. アプリに対して要求します。
1. *logs* フォルダーに移動します。 最新の stdout ログを探して開きます。
1. ログのエラーを調べます。

トラブルシューティングが完了したら、stdout ログを無効にします。

1. *web.config* ファイルを編集します。
1. **stdoutLogEnabled** を `false` に設定します。
1. ファイルを保存します。

詳細については、「<xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>」を参照してください。

> [!WARNING]
> stdout ログを無効にしないと、アプリまたはサーバーで障害が発生する可能性があります。 ログ ファイルのサイズまたは作成されるログ ファイルの数に制限はありません。
>
> ASP.NET Core アプリでのルーチン ログの場合は、ログ ファイルのサイズを制限し、ログをローテーションするログ ライブラリを使います。 詳細については、「[サードパーティ製のログ プロバイダー](xref:fundamentals/logging/index#third-party-logging-providers)」を参照してください。

### <a name="enable-the-developer-exception-page"></a>開発者例外ページを有効にする

開発環境でアプリを実行するには、`ASPNETCORE_ENVIRONMENT` [環境変数を web.config に追加することができます](xref:host-and-deploy/aspnet-core-module#setting-environment-variables)。 アプリの起動時にホスト ビルダー上で `UseEnvironment` によって環境がオーバーライドされない限り、環境変数を設定すると、アプリの実行時に[開発者例外ページ](xref:fundamentals/error-handling)が表示されます。

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
  </environmentVariables>
</aspNetCore>
```

`ASPNETCORE_ENVIRONMENT` の環境変数を設定する方法は、インターネットに公開されないステージング サーバーやテスト用サーバーの場合にのみお勧めされます。 トラブルシューティング後は、必ず *web.config* ファイルからこの環境変数を削除してください。 *web.config* で環境変数を設定する方法の詳細については、[aspNetCore の environmentVariables 子要素](xref:host-and-deploy/aspnet-core-module#setting-environment-variables)を参照してください。

### <a name="obtain-data-from-an-app"></a>アプリからデータを取得する

アプリが要求に応答できる場合は、ターミナル インライン ミドルウェアを使用して、要求、接続、その他のデータをアプリから取得します。 詳細およびサンプル コードについては、「<xref:test/troubleshoot#obtain-data-from-an-app>」を参照してください。

### <a name="slow-or-hanging-app-iis"></a>遅いアプリまたはハングしているアプリ (IIS)

"*クラッシュ ダンプ*" とはシステムのメモリのスナップショットであり、アプリのクラッシュ、起動の失敗、遅いアプリの原因を突き止めるのに役立ちます。

#### <a name="app-crashes-or-encounters-an-exception"></a>アプリのクラッシュまたは例外の発生

[Windows エラー報告 (WER)](/windows/desktop/wer/windows-error-reporting) からダンプを取得して分析します。

1. `c:\dumps` にクラッシュ ダンプ ファイルを保持するフォルダーを作成します。 アプリ プールには、そのフォルダーへの書き込みアクセス権が必要です。
1. [EnableDumps PowerShell スクリプト](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/test/troubleshoot-azure-iis/scripts/EnableDumps.ps1) を実行します。
   * アプリで [インプロセス ホスティング モデル](xref:host-and-deploy/iis/index#in-process-hosting-model)が使われている場合は、*w3wp.exe* に対するスクリプトを実行します。

     ```console
     .\EnableDumps w3wp.exe c:\dumps
     ```

   * アプリで [アウト プロセス ホスティング モデル](xref:host-and-deploy/iis/index#out-of-process-hosting-model)が使われている場合は、*dotnet.exe* に対するスクリプトを実行します。

     ```console
     .\EnableDumps dotnet.exe c:\dumps
     ```

1. クラッシュが発生する条件の下でアプリを実行します。
1. クラッシュが発生した後、[DisableDumps PowerShell スクリプト](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/test/troubleshoot-azure-iis/scripts/DisableDumps.ps1)を実行します。
   * アプリで [インプロセス ホスティング モデル](xref:host-and-deploy/iis/index#in-process-hosting-model)が使われている場合は、*w3wp.exe* に対するスクリプトを実行します。

     ```console
     .\DisableDumps w3wp.exe
     ```

   * アプリで [アウト プロセス ホスティング モデル](xref:host-and-deploy/iis/index#out-of-process-hosting-model)が使われている場合は、*dotnet.exe* に対するスクリプトを実行します。

     ```console
     .\DisableDumps dotnet.exe
     ```

アプリがクラッシュし、ダンプの収集が完了したら、アプリを普通に終了してかまいません。 PowerShell スクリプトにより、アプリごとに最大 5 つのダンプを収集する WER が構成されます。

> [!WARNING]
> クラッシュ ダンプでは、大量のディスク領域 (1 つにつき最大、数ギガバイト) が使用される場合があります。

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>アプリが起動時または正常な実行中にハングまたは失敗する

アプリが起動時または正常な実行中に "*ハング*" (応答を停止するがクラッシュしない) または失敗するときは、「[User-Mode Dump Files:Choosing the Best Tool](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool)」(ユーザー モード ダンプ ファイル: 最適なツールの選択) を参照し、適切なツールを選択してダンプを生成します。

#### <a name="analyze-the-dump"></a>ダンプを分析する

ダンプはいくつかの方法で分析できます。 詳細については、「[Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file)」(ユーザー モード ダンプ ファイルの分析) を参照してください。

## <a name="clear-package-caches"></a>パッケージ キャッシュをクリアする

開発マシンで .NET Core SDK をアップグレードしたり、アプリ内のパッケージ バージョンを変更したりした直後に、機能しているアプリが失敗することがあります。 場合によっては、パッケージに統一性がないと、メジャー アップグレード実行時にアプリが破壊されることがあります。 これらの問題のほとんどは、次の手順で解決できます。

1. *bin* フォルダーと *obj* フォルダーを削除します。
1. コマンド シェルから [dotnet nuget locals all --clear](/dotnet/core/tools/dotnet-nuget-locals) を実行して、パッケージ キャッシュをクリアします。

   パッケージ キャッシュをクリアするには、[nuget.exe](https://www.nuget.org/downloads) ツールを使用して `nuget locals all -clear` コマンドを実行する方法もあります。 *nuget.exe* は、Windows デスクトップ オペレーティング システムにバンドルされているインストールではなく、[NuGet Web サイト](https://www.nuget.org/downloads)から別に入手する必要があります。

1. プロジェクトを復元してリビルドします。
1. アプリを再展開する前に、サーバー上の展開フォルダー内のすべてのファイルを削除します。

## <a name="additional-resources"></a>その他の技術情報

* <xref:test/troubleshoot>
* <xref:host-and-deploy/azure-iis-errors-reference>
* <xref:fundamentals/error-handling>
* <xref:host-and-deploy/aspnet-core-module>

### <a name="azure-documentation"></a>Azure のドキュメント

* [ASP.NET Core 用 Application Insights](/azure/application-insights/app-insights-asp-net-core)
* [Visual Studio を使用した Azure App Service の Web アプリのトラブルシューティングの Web アプリのリモート デバッグ セクション](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio#remotedebug)
* [Azure App Service の診断の概要](/azure/app-service/app-service-diagnostics)
* [方法: Azure App Service でアプリを監視する](/azure/app-service/web-sites-monitor)
* [Visual Studio を使用した Azure App Service での Web アプリのトラブルシューティング](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio)
* [Azure Web Apps での "502 bad gateway" および "503 service unavailable" の HTTP エラーのトラブルシューティング](/azure/app-service/app-service-web-troubleshoot-http-502-http-503)
* [Azure App Service での Web アプリのパフォーマンス低下に関する問題のトラブルシューティング](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [Azure での Web アプリのアプリケーションパフォーマンスに関するよくあるご質問](/azure/app-service/app-service-web-availability-performance-application-issues-faq)
* [Azure Web アプリのサンドボックス (App Service ランタイムの実行の制限)](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox)
* [Azure Friday: Azure App Service の診断とトラブルシューティング (12 分間のビデオ)](https://channel9.msdn.com/Shows/Azure-Friday/Azure-App-Service-Diagnostic-and-Troubleshooting-Experience)

### <a name="visual-studio-documentation"></a>Visual Studio ドキュメント

* [Visual Studio 2017 における Azure の IIS での ASP.NET Core のリモート デバッグ](/visualstudio/debugger/remote-debugging-azure)
* [Visual Studio 2017 におけるリモートの IIS コンピューターでの ASP.NET Core のリモート デバッグ](/visualstudio/debugger/remote-debugging-aspnet-on-a-remote-iis-computer)
* [Visual Studio を使用したデバッグについて理解する](/visualstudio/debugger/getting-started-with-the-debugger)

### <a name="visual-studio-code-documentation"></a>Visual Studio Code ドキュメント

* [Visual Studio Code でのデバッグ](https://code.visualstudio.com/docs/editor/debugging)

::: moniker-end
