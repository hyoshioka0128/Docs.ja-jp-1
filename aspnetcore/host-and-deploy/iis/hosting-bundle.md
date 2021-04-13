---
title: ホスティング バンドル
author: rick-anderson
description: .NET Core ホスティング バンドルを構成する方法について説明します。
monikerRange: '>= aspnetcore-5.0'
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
uid: host-and-deploy/iis/hosting-bundle
ms.openlocfilehash: 044b7ef98610e504eb4887dae88c8636c7c9a8fe
ms.sourcegitcommit: fafcf015d64aa2388bacee16ba38799daf06a4f0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/30/2021
ms.locfileid: "105957666"
---
# <a name="the-net-core-hosting-bundle"></a>.NET Core ホスティング バンドル

.NET Core ホスティング バンドルは、.NET Core ランタイムおよび [ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)用のインストーラーです。 バンドルを使用すると、ASP.NET Core アプリを IIS で実行できます。

## <a name="install-the-net-core-hosting-bundle"></a>.NET Core ホスティング バンドルのインストール

> [!IMPORTANT]
> ホスティング バンドルが IIS の前にインストールされている場合、バンドルのインストールを修復する必要があります。 IIS をインストールした後に、ホスティング バンドル インストーラーをもう一度実行します。
>
> .NET Core の 64 ビット (x64) バージョンをインストールした後、ホスティング バンドルをインストールした場合は、SDK が表示されない可能性があります ([.NET Core SDK が検出されない](xref:test/troubleshoot#no-net-core-sdks-were-detected))。 この問題を解決するには、<xref:test/troubleshoot#missing-sdk-after-installing-the-net-core-hosting-bundle> を参照してください。

[!INCLUDE[](~/includes/announcements.md)]

## <a name="direct-download-current-version"></a>直接ダウンロード (現在のバージョン)

次のリンクを使用してインストーラーをダウンロードします。

[現在の .NET Core ホスティング バンドルのインストーラー (直接ダウンロード)](https://dotnet.microsoft.com/permalink/dotnetcore-current-windows-runtime-bundle-installer)

## <a name="visual-c-redistributable-requirement"></a>Visual C++ 再頒布可能パッケージの要件

Windows Server 2012 R2 など、古いバージョンの Windows では、Visual Studio C++ 2015、2017、2019 の再頒布可能パッケージをインストールします。 そうしないと、`The data is the error.` という混乱を招くエラー メッセージが Windows イベント ログで報告されます。

[現在の x64 VS C++ 再頒布可能パッケージ](https://aka.ms/vs/16/release/vc_redist.x64.exe)
[現在の x86 VS C++ 再頒布可能パッケージ](https://aka.ms/vs/16/release/vc_redist.x86.exe)

## <a name="earlier-versions-of-the-installer"></a>以前のバージョンのインストーラー

以前のバージョンのインストーラーを入手するには:

1. 「[.NET Core のダウンロード](https://dotnet.microsoft.com/download/dotnet-core)」ページに移動します。
1. 目的の .NET Core のバージョンを選択します。
1. **[Run apps - Runtime]\(アプリの実行 - ランタイム\)** 列から、目的の .NET Core ランタイム バージョンの行を探します。
1. **[Hosting Bundle]\(ホスティング バンドル\)** のリンクを使用してインストーラーをダウンロードします。

> [!WARNING]
> 一部のインストーラーには、有効期限 (EOL) に達して Microsoft によるサポートが終了した、リリース バージョンが含まれています。 詳細については、[サポート ポリシー](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)をご覧ください。

## <a name="options"></a>オプション

1. 管理者のコマンド シェルからインストーラーを実行する場合、次のパラメーターを使用できます。

   * `OPT_NO_ANCM=1`:ASP.NET Core モジュールのインストールをスキップします。
   * `OPT_NO_RUNTIME=1`:.NET Core ランタイムのインストールをスキップします。 サーバーが[自己完結型の展開 (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd) のみをホストする場合に使用します。
   * `OPT_NO_SHAREDFX=1`:ASP.NET Shared Framework (ASP.NET ランタイム) のインストールをスキップします。 サーバーが[自己完結型の展開 (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd) のみをホストする場合に使用します。
   * `OPT_NO_X86=1`:x86 ランタイムのインストールをスキップします。 32 ビット アプリをホストしない場合は、このパラメーターを使用します。 今後、32 ビットと 64 ビットのアプリ両方をホストする可能性がある場合は、このパラメーターを使用せずに、両方のランタイムをインストールします。
   * `OPT_NO_SHARED_CONFIG_CHECK=1`:共有構成 (`applicationHost.config`) が IIS のインストールと同じマシン上にある場合、IIS 共有構成を使用するためのチェックを無効にします。 "*ASP.NET Core 2.2 以降の Hosting Bundler インストーラーに対してのみ使用できます。* " 詳細については、「<xref:host-and-deploy/aspnet-core-module#aspnet-core-module-with-an-iis-shared-configuration>」を参照してください。

> [!NOTE]
> IIS 共有構成の詳細については、「[ASP.NET Core Module with IIS Shared Configuration](xref:host-and-deploy/aspnet-core-module#aspnet-core-module-with-an-iis-shared-configuration)」 (IIS 共有構成の ASP.NET Core モジュール) を参照してください。

## <a name="restart-iis"></a>IIS を再開する

ホスティング バンドルをインストールした後は、手動で IIS を再起動することが必要な場合があります。 たとえば、IIS ワーカー プロセスを実行するための PATH に、`dotnet` CLI ツール (コマンド) が存在しない場合があります。

IIS を手動で停止して開始するには、管理者特権のコマンド シェルで次のコマンドを実行します。

```console
net stop was /y
net start w3svc
```

## <a name="module-version-and-hosting-bundle-installer-logs"></a>モジュールのバージョンとホスティング バンドル インストーラーのログ

インストールされている ASP.NET Core モジュールのバージョンを確認するには:

1. ホスティング システム上で、 `%PROGRAMFILES%\IIS\Asp.Net Core Module\V2` に移動します。
1. `aspnetcorev2.dll` ファイルを見つけます。
1. ファイルを右クリックし、コンテキスト メニューの **[プロパティ]** を選びます。
1. **[詳細]** タブを選びます。 **[ファイル バージョン]** と **[製品バージョン]** が、インストールされているモジュールのバージョンを表します。

モジュールに関するホスティング バンドル インストーラーのログは、`C:\Users\%UserName%\AppData\Local\Temp` にあります。 このファイルには `dd_DotNetCoreWinSvrHosting__{TIMESTAMP}_000_AspNetCoreModule_x64.log` という名前が付けられます。ここで、プレースホルダー `{TIMESTAMP}` はファイルのタイムスタンプです。
