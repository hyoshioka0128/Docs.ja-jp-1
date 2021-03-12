---
title: Visual Studio コンテナー ツールと ASP.NET Core
author: spboyer
description: Visual Studio ツールと Docker for Windows を使用して、ASP.NET Core アプリをコンテナー化する方法を説明します。
ms.author: scaddie
ms.custom: mvc
ms.date: 09/12/2018
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
uid: host-and-deploy/docker/visual-studio-tools-for-docker
ms.openlocfilehash: c11261fef84234a9caaf872ae9871932d02617fa
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102585995"
---
# <a name="visual-studio-container-tools-with-aspnet-core"></a>Visual Studio コンテナー ツールと ASP.NET Core

Visual Studio 2017 以降のバージョンでは、.NET Core をターゲットとするコンテナー化された ASP.NET Core アプリのビルド、デバッグ、実行がサポートされています。 Windows と Linux の両方のコンテナーがサポートされます。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/docker/visual-studio-tools-for-docker/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="prerequisites"></a>必須コンポーネント

* [Docker for Windows](https://docs.docker.com/docker-for-windows/install/)
* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) と **.NET Core クロスプラットフォームの開発** ワークロード

## <a name="installation-and-setup"></a>インストールとセットアップ

Docker をインストールするには、まず、「[Docker for Windows:What to know before you install](https://docs.docker.com/docker-for-windows/install/#what-to-know-before-you-install)」 (Docker Desktop for Windows: インストール前に知っておくべきこと) の情報を確認します。 次に、[Docker for Windows](https://docs.docker.com/docker-for-windows/install/) をインストールします。

ボリュームのマッピングとデバッグをサポートするように、Docker for Windows で **[共有ドライブ](https://docs.docker.com/docker-for-windows/#shared-drives)**  を構成する必要があります。 システム トレイの Docker アイコンを右クリックし、 **[設定]** 、 **[Shared Drives]\(共有ドライブ\)** の順に選択します。 Docker がファイルを保存するドライブを選択します。 **[適用]** をクリックします。

![コンテナーで共有するローカル C ドライブを選択するためのダイアログ](visual-studio-tools-for-docker/_static/settings-shared-drives-win.png)

> [!TIP]
> Visual Studio 2017 バージョン 15.6 以降では、**共有ドライブ** が構成されていない場合にプロンプトが表示されます。

## <a name="add-a-project-to-a-docker-container"></a>Docker コンテナーにプロジェクトを追加する

ASP.NET Core プロジェクトをコンテナー化するには、プロジェクトで .NET Core をターゲットにする必要があります。 Linux と Windows の両方のコンテナーがサポートされています。

プロジェクトに Docker サポートを追加する場合は、Windows または Linux のいずれかのコンテナーを選択します。 Docker ホストが同じコンテナーの種類を実行している必要があります。 実行中の Docker インスタンスでコンテナーの種類を変更するには、システム トレイの Docker アイコンを右クリックして、 **[Switch to Windows containers...]\(Windows コンテナーに切り替える...\)** または **[Switch to Linux containers...]\(Linux コンテナーに切り替える...\)** を選択します。

### <a name="new-app"></a>新しいアプリ

**ASP.NET Core Web アプリケーション** プロジェクト テンプレートを使用して新しいアプリを作成する場合は、次のように **[Enable Docker Support]\(Docker サポートを有効にする\)** チェック ボックスをオンにします。

![[Enable Docker Support]\(Docker サポートを有効にする\) チェック ボックス](visual-studio-tools-for-docker/_static/enable-docker-support-check-box.png)

ターゲット フレームワークが .NET Core の場合、 **[OS]** ドロップダウンでコンテナーの種類を選択できます。

### <a name="existing-app"></a>既存のアプリ

.NET Core をターゲットとする ASP.NET Core プロジェクトの場合、ツールを使用して Docker サポートを追加するための 2 つのオプションがあります。 Visual Studio でプロジェクトを開き、次のオプションのいずれかを選択します。

* **[プロジェクト]** メニューから **[Docker サポート]** を選択します。
* **ソリューション エクスプローラー** でプロジェクトを右クリックして、 **[追加]**  >  **[Docker サポート]** の順に選択します。

Visual Studio コンテナー ツールでは、.NET Framework をターゲットとする既存の ASP.NET Core プロジェクトへの Docker の追加はサポートされません。

## <a name="dockerfile-overview"></a>Dockerfile の概要

*Dockerfile* (Docker の最終イメージを作成するためのレシピ) は、プロジェクトのルートに追加されます。 その中に含まれるコマンドの詳細については、「[Dockerfile reference](https://docs.docker.com/engine/reference/builder/)」 (Dockerfile リファレンス) を参照してください。 この特定の *Dockerfile* では、次のような、4 つの異なる名前付きのビルド ステージを含む、[multi-stage build](https://docs.docker.com/engine/userguide/eng-image/multistage-build/) を使用します。

::: moniker range=">= aspnetcore-2.1"

[!code-dockerfile[](visual-studio-tools-for-docker/samples/2.1/HelloDockerTools/Dockerfile.original?highlight=1,6,14,17)]

上記の *Dockerfile* は、[microsoft/dotnet](https://hub.docker.com/r/microsoft/dotnet/) イメージに基づいています。 この基本イメージには、ASP.NET Core ランタイムと NuGet パッケージが含まれます。 パッケージは、起動時のパフォーマンスを向上させるために JIT (Just-In-Time) コンパイルされます。

新しいプロジェクト ダイアログの **[Configure for HTTPS]\(HTTPS 用に構成する\)** チェック ボックスがオンになっている場合、*Dockerfile* は 2 つのポートを公開します。 1 つのポートは HTTP トラフィック用、もう 1 つのポートは HTTPS 用に使用されます。 チェック ボックスがオンになっていない場合は、HTTP トラフィック用に単一のポート (80) が公開されます。

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

[!code-dockerfile[](visual-studio-tools-for-docker/samples/2.0/HelloDockerTools/Dockerfile?highlight=1,5,13,16)]

上記の *Dockerfile* は、[microsoft/aspnetcore](https://hub.docker.com/r/microsoft/aspnetcore/) イメージに基づいています。 この基本イメージには、起動時のパフォーマンスを向上させるために JIT (Just-In-Time) コンパイルされる、ASP.NET Core NuGet パッケージが含まれます。

::: moniker-end

## <a name="add-container-orchestrator-support-to-an-app"></a>アプリにコンテナー オーケストレーター サポートを追加する

Visual Studio 2017 バージョン 15.7 以前では、唯一のコンテナー オーケストレーション ソリューションとして、[Docker Compose](https://docs.docker.com/compose/overview/) がサポートされています。 Docker Compose の成果物は、 **[追加]**  >  **[Docker サポート]** を使用して追加されます。

Visual Studio 2017 バージョン 15.8 以降では、指示された場合にのみ、オーケストレーション ソリューションが追加されます。 **ソリューション エクスプローラー** でプロジェクトを右クリックして、 **[追加]**  >  **[Container Orchestrator Support]\(コンテナー オーケストレーター サポート)** の順に選択します。 次の選択肢があります。 

* [Docker Compose](#docker-compose)
* [Service Fabric](#service-fabric)
* [Kubernetes/Helm ](https://helm.sh/)

### <a name="docker-compose"></a>Docker Compose

Visual Studio コンテナー ツールでは、ソリューションに *docker-compose* プロジェクトを追加します。これには以下のファイルが含まれます。

* *docker-compose.dcproj*:プロジェクトを表すファイル。 使用する OS を指定する `<DockerTargetOS>` 要素が含まれます。
* *.dockerignore*:ビルド コンテキストを生成するときに除外するファイルとディレクトリのパターンが一覧表示されます。
* *docker-compose.yml*:`docker-compose build` および `docker-compose run` を使用して、それぞれビルドおよび実行されるイメージのコレクションを定義するために使用される、基本の [Docker Compose](https://docs.docker.com/compose/overview/) ファイル。
* *docker-compose.override.yml*:サービスの構成オーバーライドを含む、Docker Compose によって読み取られるオプション ファイル。 Visual Studio は `docker-compose -f "docker-compose.yml" -f "docker-compose.override.yml"` を実行してこれらのファイルをマージします。

*docker-compose.yml* ファイルでは、プロジェクトの実行時に作成されたイメージの名前を参照します。

[!code-yaml[](visual-studio-tools-for-docker/samples/2.0/docker-compose.yml?highlight=5)]

上の例では、`image: hellodockertools` によって、アプリが **デバッグ** モードで実行されるときに、イメージ `hellodockertools:dev` が生成されます。 `hellodockertools:latest` イメージは、アプリが **リリース** モードで実行されるときに生成されます。

イメージがレジストリにプッシュされる場合は、イメージ名の前に [Docker Hub](https://hub.docker.com/) のユーザー名を付けます (例: `dockerhubusername/hellodockertools`)。 または、構成に応じて、プライベート レジストリ URL を含めるようにイメージ名を変更します (例: `privateregistry.domain.com/hellodockertools`)。

ビルド構成に基づいて動作を別にしたい場合 (デバッグやリリースなど)、構成固有の *docker-compose* ファイルを追加します。 ビルド構成に基づいて (たとえば、*docker-compose.vs.debug.yml* および *docker-compose.vs.release.yml*) ファイル名を指定して、*docker-compose-override.yml* ファイルと同じ場所にそのファイルを配置する必要があります。 

構成固有の上書きファイルを使用して、デバッグおよびリリース ビルド構成に対して、さまざまな構成設定 (環境変数やエンドポイントなど) を指定できます。

Docker Compose で Visual Studio で実行するオプションを表示するには、Docker プロジェクトがスタートアップ プロジェクトである必要があります。

### <a name="service-fabric"></a>Service Fabric

基本的な[前提条件](#prerequisites)に加え、[Service Fabric](/azure/service-fabric/) オーケストレーション ソリューションでは次の前提条件が求められます。

* [Microsoft Azure Service Fabric SDK](https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-CoreSDK) バージョン 2.6 以降
* Visual Studio の **Azure Development** ワークロード

Service Fabric では、Windows 上のローカル開発クラスターでの Linux コンテナーの実行はサポートされません。 プロジェクトで既に Linux コンテナーが使用されている場合は、Visual Studio で Windows コンテナーに切り替えるよう求められます。

Visual Studio コンテナーツールでは、次のタスクを実行します。

* ソリューションに、 *&lt;プロジェクト名&gt;Application* **Service Fabric アプリケーション** プロジェクトが追加されます。
* *Dockerfile* と *.dockerignore* ファイルを ASP.NET Core プロジェクトに追加します。 *Dockerfile* が既に ASP.NET Core プロジェクトに存在する場合は、名前が *Dockerfile.original* に変更されます。 次のような、新しい *Dockerfile* が作成されます。

    [!code-dockerfile[](visual-studio-tools-for-docker/samples/2.1/HelloDockerTools/Dockerfile)]

* `<IsServiceFabricServiceProject>` 要素を ASP.NET Core プロジェクトの *.csproj* ファイルに追加します。

    [!code-xml[](visual-studio-tools-for-docker/samples/2.1/HelloDockerTools/HelloDockerTools.csproj?name=snippet_IsServiceFabricServiceProject)]

* *PackageRoot* フォルダーを ASP.NET Core プロジェクトに追加します。 フォルダーには、新しいサービスのサービス マニフェストと設定が含まれます。

詳細については、「[Windows コンテナー内の .NET アプリケーションを Azure Service Fabric にデプロイする](/azure/service-fabric/service-fabric-host-app-in-a-container)」を参照してください。

## <a name="debug"></a>デバッグ

ツールバーのデバッグ ドロップダウンから **[Docker]** を選択し、アプリのデバッグを開始します。 **[出力]** ウィンドウの **[Docker]** ビューには、行われている次のアクションが表示されます。

::: moniker range=">= aspnetcore-2.1"

* *microsoft/dotnet* ランタイム イメージの *2.1-aspnetcore-runtime* タグが取得されます (キャッシュにまだ存在しない場合)。 イメージでは、ASP.NET Core と .NET Core ランタイムおよび関連付けられているライブラリがインストールされます。 運用環境で ASP.NET Core アプリを実行するために最適化されます。
* `ASPNETCORE_ENVIRONMENT` 環境変数は、コンテナー内で `Development` に設定されます。
* HTTP と HTTPS 用に 1 つずつ、2 つの動的に割り当てられたポートが公開されます。 localhost に割り当てられたポートは、`docker ps` コマンドを使用してクエリを実行できます。
* アプリがコンテナーにコピーされます。
* 動的に割り当てられたポートを使用して、デバッガーがコンテナーにアタッチされ、既定のブラウザーが起動します。

アプリの結果の Docker イメージは、*dev* としてタグ付けされます。 イメージは、*microsoft/dotnet* 基本イメージの *2.1-aspnetcore-runtime* タグに基づいています。 **パッケージ マネージャー コンソール** (PMC) ウィンドウで `docker images` コマンドを実行します。 コンピューター上のイメージが表示されます。

```console
REPOSITORY        TAG                     IMAGE ID      CREATED         SIZE
hellodockertools  dev                     d72ce0f1dfe7  30 seconds ago  255MB
microsoft/dotnet  2.1-aspnetcore-runtime  fcc3887985bb  6 days ago      255MB
```

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

* *microsoft/aspnetcore* ランタイム イメージが取得されます (まだキャッシュにない場合)。
* `ASPNETCORE_ENVIRONMENT` 環境変数は、コンテナー内で `Development` に設定されます。
* ポート 80 が公開され、localhost 用に動的に割り当てられているポートにマップされます。 このポートは、Docker ホストによって決定され、`docker ps` コマンドでクエリを実行することができます。
* アプリがコンテナーにコピーされます。
* 動的に割り当てられたポートを使用して、デバッガーがコンテナーにアタッチされ、既定のブラウザーが起動します。

アプリの結果の Docker イメージは、*dev* としてタグ付けされます。 イメージは、*microsoft/aspnetcore* 基本イメージに基づいています。 **パッケージ マネージャー コンソール** (PMC) ウィンドウで `docker images` コマンドを実行します。 コンピューター上のイメージが表示されます。

```console
REPOSITORY            TAG  IMAGE ID      CREATED        SIZE
hellodockertools      dev  5fafe5d1ad5b  4 minutes ago  347MB
microsoft/aspnetcore  2.0  c69d39472da9  13 days ago    347MB
```

::: moniker-end

> [!NOTE]
> **デバッグ** 構成では、反復にボリューム マウントを使用するため、*dev* イメージにはアプリのコンテンツはありません。 イメージをプッシュするには、**リリース** 構成を使用します。

PMC で `docker ps` コマンドを実行します。 アプリがコンテナーを使用して実行されていることがわかります。

```console
CONTAINER ID        IMAGE                  COMMAND                   CREATED             STATUS              PORTS                   NAMES
baf9a678c88d        hellodockertools:dev   "C:\\remote_debugge..."   21 seconds ago      Up 19 seconds       0.0.0.0:37630->80/tcp   dockercompose4642749010770307127_hellodockertools_1
```

## <a name="edit-and-continue"></a>エディット コンティニュ

静的なファイルや Razor ビューに対する変更は、コンパイルの手順を行う必要はなく、自動的に更新されます。 変更を行って保存し、ブラウザーを更新し、更新を確認します。

コード ファイルを変更する場合、コンテナー内で Kestrel をコンパイルして再起動する必要があります。 変更したら、`CTRL+F5` キーを使用して、コンテナー内でプロセスを実行し、アプリを起動します。 Docker コンテナーが再構築されたり、停止されたりすることはありません。 PMC で `docker ps` コマンドを実行します。 元のコンテナーが 10 分前と同じ状態で実行されていることを確認できます。

```console
CONTAINER ID        IMAGE                  COMMAND                   CREATED             STATUS              PORTS                   NAMES
baf9a678c88d        hellodockertools:dev   "C:\\remote_debugge..."   10 minutes ago      Up 10 minutes       0.0.0.0:37630->80/tcp   dockercompose4642749010770307127_hellodockertools_1
```

## <a name="publish-docker-images"></a>Docker イメージの発行

アプリの開発とデバッグのサイクルが完了したら、Visual Studio コンテナーツールでアプリの実稼働イメージを作成できます。 構成ドロップダウンを **[リリース]** に変更し、アプリを構築します。 ツールは、Docker Hub からコンパイル/発行イメージを取得します (まだキャッシュに存在しない場合)。 イメージは、プライベート レジストリまたは Docker Hub にプッシュできる *latest* タグ付きで生成されます。

PMC で `docker images` コマンドを実行して、イメージの一覧を表示します。 次のような出力が表示されます。

::: moniker range=">= aspnetcore-2.1"

```console
REPOSITORY        TAG                     IMAGE ID      CREATED             SIZE
hellodockertools  latest                  e3984a64230c  About a minute ago  258MB
hellodockertools  dev                     d72ce0f1dfe7  4 minutes ago       255MB
microsoft/dotnet  2.1-sdk                 9e243db15f91  6 days ago          1.7GB
microsoft/dotnet  2.1-aspnetcore-runtime  fcc3887985bb  6 days ago          255MB
```

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

```console
REPOSITORY                  TAG     IMAGE ID      CREATED         SIZE
hellodockertools            latest  cd28f0d4abbd  12 seconds ago  349MB
hellodockertools            dev     5fafe5d1ad5b  23 minutes ago  347MB
microsoft/aspnetcore-build  2.0     7fed40fbb647  13 days ago     2.02GB
microsoft/aspnetcore        2.0     c69d39472da9  13 days ago     347MB
```

上記の出力に一覧表示されている `microsoft/aspnetcore-build` および `microsoft/aspnetcore` イメージは、.NET Core 2.1 の時点で `microsoft/dotnet` イメージに置き換えられます。 詳細については、[Docker リポジトリの移行に関するお知らせ](https://github.com/aspnet/Announcements/issues/298)を参照してください。

::: moniker-end

> [!NOTE]
> `docker images` コマンドは、 *\<none>* として識別されるリポジトリ名とタグを持つ中間イメージを返します (上にはリストされていません)。 これらの名前のないイメージは、[multi-stage build](https://docs.docker.com/engine/userguide/eng-image/multistage-build/) *Dockerfile* によって生成されます。 これにより、最終イメージの構築効率が向上します&mdash;変更時には必要なレイヤーのみが再構築されます。 中間イメージが不要になった場合は、[docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/) コマンドを使用して削除します。

*dev* イメージと比較した場合、実稼働またはリリース イメージはサイズが小さいと思うかもしれません。 ボリューム マッピングにより、デバッガーとアプリは、コンテナー内ではなく、ローカル コンピューターから実行されています。 *latest* イメージには、ホスト コンピューターでアプリを実行するために必要なアプリ コードがパッケージ化されています。 そのため、デルタはアプリ コードのサイズです。

## <a name="additional-resources"></a>その他の技術情報

* [Visual Studio によるコンテナー開発](/visualstudio/containers)
* [Azure Service Fabric:開発環境を準備する](/azure/service-fabric/service-fabric-get-started)
* [Windows コンテナー内の .NET アプリケーションを Azure Service Fabric にデプロイする](/azure/service-fabric/service-fabric-host-app-in-a-container)
* [Docker を使用した Visual Studio 開発のトラブルシューティング](/azure/vs-azure-tools-docker-troubleshooting-docker-errors)
* [Visual Studio コンテナー ツールの GitHub リポジトリ](https://github.com/Microsoft/DockerTools)
* [ドッカーと小さなコンテナを使用した GC](xref:performance/memory#sc)
