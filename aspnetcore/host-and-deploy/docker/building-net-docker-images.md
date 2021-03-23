---
title: ASP.NET Core 向けの Docker イメージ
author: rick-anderson
description: 公開されている ASP.NET Core Docker イメージを Docker レジストリから使用する方法について説明します。 独自のイメージをプルしてビルドします。
ms.author: riande
ms.custom: mvc
ms.date: 01/04/2021
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
uid: host-and-deploy/docker/building-net-docker-images
ms.openlocfilehash: 32e721035df8bd9e746ad4db6bb2753c358f3dac
ms.sourcegitcommit: 07e7ee573fe4e12be93249a385db745d714ff6ae
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/12/2021
ms.locfileid: "103413497"
---
# <a name="docker-images-for-aspnet-core"></a>ASP.NET Core 向けの Docker イメージ

このチュートリアルでは、Docker コンテナー内で ASP.NET Core アプリを実行する方法を示します。

このチュートリアルでは、次の作業を行いました。
> [!div class="checklist"]
> * ASP.NET Core Docker イメージについて学習する
> * ASP.NET Core サンプル アプリをダウンロードする
> * サンプル アプリをローカルで実行する
> * Linux コンテナー内でサンプル アプリを実行する
> * Windows コンテナー内でサンプル アプリを実行する
> * 手動でビルドしてデプロイする

## <a name="aspnet-core-docker-images"></a>ASP.NET Core の Docker イメージ

このチュートリアルでは、ASP.NET Core サンプル アプリをダウンロードして、Docker コンテナー内で実行します。 このサンプルは Linux コンテナーと Windows コンテナーのどちらでも動作します。

さまざまなコンテナー内でビルドして実行するために、サンプルの Dockerfile では [Docker のマルチステージ ビルド機能](https://docs.docker.com/engine/userguide/eng-image/multistage-build/)を使用しています。 ビルドと実行のコンテナーは、マイクロソフトが Docker Hub に提供しているイメージから作成されます。

::: moniker range=">= aspnetcore-5.0"

* `dotnet/sdk`

  サンプルでは、アプリをビルドするためにこのイメージを使用します。 イメージには、コマンド ライン ツール (CLI) が組み込まれた .NET SDK が含まれています。 イメージはローカル開発、デバッグ、および単体テスト用に最適化されています。 開発とコンパイルのためにツールがインストールされているため、比較的大きなイメージになっています。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* `dotnet/core/sdk`

  サンプルでは、アプリをビルドするためにこのイメージを使用します。 イメージには、コマンド ライン ツール (CLI) が組み込まれた .NET Core SDK が含まれています。 イメージはローカル開発、デバッグ、および単体テスト用に最適化されています。 開発とコンパイルのためにツールがインストールされているため、比較的大きなイメージになっています。

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

* `dotnet/aspnet`

   サンプルでは、アプリを実行するためにこのイメージを使用します。 イメージには ASP.NET Core ランタイムとライブラリが含まれており、実稼働環境でアプリを実行するために最適化されています。 デプロイとアプリ起動の速度に対応した設計になっており、Docker レジストリから Docker ホストへのネットワーク パフォーマンスが最適化されていることから、イメージは比較的小さいです。 アプリの実行に必要なバイナリとコンテンツのみが、コンテナーにコピーされます。 コンテンツは実行できる状態になっており、`docker run` からアプリの起動までを最速で行うことができます。 動的コード コンパイルは Docker モデルで必要ありません。
   
::: moniker-end

::: moniker range="< aspnetcore-5.0"

* `dotnet/core/aspnet`

   サンプルでは、アプリを実行するためにこのイメージを使用します。 イメージには ASP.NET Core ランタイムとライブラリが含まれており、実稼働環境でアプリを実行するために最適化されています。 デプロイとアプリ起動の速度に対応した設計になっており、Docker レジストリから Docker ホストへのネットワーク パフォーマンスが最適化されていることから、イメージは比較的小さいです。 アプリの実行に必要なバイナリとコンテンツのみが、コンテナーにコピーされます。 コンテンツは実行できる状態になっており、`docker run` からアプリの起動までを最速で行うことができます。 動的コード コンパイルは Docker モデルで必要ありません。
   
::: moniker-end

## <a name="prerequisites"></a>必須コンポーネント

::: moniker range=">= aspnetcore-5.0"

* [.NET SDK 5.0](https://dotnet.microsoft.com/download)

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

* [.NET Core SDK 3.1](https://dotnet.microsoft.com/download)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

* [.NET Core 2.2 SDK](https://dotnet.microsoft.com/download/dotnet-core)

::: moniker-end

* Docker クライアント 18.03 以降

  * Linux ディストリビューション
    * [CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
    * [Debian](https://docs.docker.com/install/linux/docker-ce/debian/)
    * [Fedora](https://docs.docker.com/install/linux/docker-ce/fedora/)
    * [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
  * [macOS](https://docs.docker.com/docker-for-mac/install/)
  * [Windows](https://docs.docker.com/docker-for-windows/install/)

* [Git](https://git-scm.com/download)

## <a name="download-the-sample-app"></a>サンプル アプリ をダウンロードする

* [.NET Docker リポジトリ](https://github.com/dotnet/dotnet-docker)を複製して、サンプルをダウンロードします。 

  ```console
  git clone https://github.com/dotnet/dotnet-docker
  ```

## <a name="run-the-app-locally"></a>アプリをローカルで実行する

* *dotnet-docker/samples/aspnetapp/aspnetapp* にあるプロジェクト フォルダーに移動します。

* 次のコマンドを実行し、アプリをビルドしてローカルで実行します。

  ```dotnetcli
  dotnet run
  ```

* アプリをテストするには、ブラウザーで `http://localhost:5000` に移動します。

* コマンド プロンプト上で Ctrl +C キーを押して、アプリを停止します。

## <a name="run-in-a-linux-container"></a>Linux コンテナーでの実行

* Docker クライアント上で、[Linux コンテナーに切り替えます](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers)。

* *dotnet-docker/samples/aspnetapp* にある Dockerfile フォルダーに移動します。

* 次のコマンドを実行して、Docker 内でサンプルをビルドして実行します。

  ```console
  docker build -t aspnetapp .
  docker run -it --rm -p 5000:80 --name aspnetcore_sample aspnetapp
  ```

  `build` コマンドの引数:
  * イメージに aspnetapp という名前を付けます。
  * 現在のフォルダー内にある Dockerfile を探します (末尾にピリオド)。

  実行コマンドの引数:
  * 擬似端末を割り当てて、接続されていない場合でも開いた状態を保持します。 (`--interactive --tty` と効果は同じです。)
  * コンテナーが存在する場合は、自動的に削除します。
  * ローカル コンピューター上のポート 5000 をコンテナー内のポート 80 にマップします。
  * コンテナーに aspnetcore_sample という名前を付けます。
  * aspnetapp イメージを指定します。

* アプリをテストするには、ブラウザーで `http://localhost:5000` に移動します。

## <a name="run-in-a-windows-container"></a>Windows コンテナーでの実行

* Docker クライアント上で、[Windows コンテナーに切り替えます](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers)。

`dotnet-docker/samples/aspnetapp` にある Docker ファイルのフォルダーに移動します。

* 次のコマンドを実行して、Docker 内でサンプルをビルドして実行します。

  ```console
  docker build -t aspnetapp .
  docker run -it --rm --name aspnetcore_sample aspnetapp
  ```

* Windows コンテナーの場合、コンテナーの IP アドレスが必要です (`http://localhost:5000` の参照は機能しません)。
  * 別のコマンド プロンプトを開きます。
  * `docker ps` を実行して、実行中のコンテナーを表示します。 "aspnetcore_sample" コンテナーがそこにあることを確認します。
  * `docker exec aspnetcore_sample ipconfig` を実行して、コンテナーの IP アドレスを表示します。 コマンドからの出力は、この例のようになります。

    ```console
    Ethernet adapter Ethernet:

       Connection-specific DNS Suffix  . : contoso.com
       Link-local IPv6 Address . . . . . : fe80::1967:6598:124:cfa3%4
       IPv4 Address. . . . . . . . . . . : 172.29.245.43
       Subnet Mask . . . . . . . . . . . : 255.255.240.0
       Default Gateway . . . . . . . . . : 172.29.240.1
    ```

* コンテナーの IPv4 アドレス (たとえば、172.29.245.43) をコピーして、ブラウザーのアドレス バーに貼り付けてアプリをテストします。

## <a name="build-and-deploy-manually"></a>手動でビルドしてデプロイする

一部のシナリオでは、実行時に必要なアプリのアセットをコピーすることで、アプリをコンテナーにデプロイする方がよい場合があります。 このセクションでは、手動によるデプロイの方法を示します。

* *dotnet-docker/samples/aspnetapp/aspnetapp* にあるプロジェクト フォルダーに移動します。

* [dotnet publish](/dotnet/core/tools/dotnet-publish) コマンドを実行します。

  ```dotnetcli
  dotnet publish -c Release -o published
  ```

  コマンドの引数:
  * リリース モードでアプリをビルドします (既定はデバッグ モードです)。
  * *published* フォルダーにアセットを作成します。

* アプリを実行します。

  * Windows の場合:

    ```dotnetcli
    dotnet published\aspnetapp.dll
    ```

  * Linux の場合:

    ```dotnetcli
    dotnet published/aspnetapp.dll
    ```

* `http://localhost:5000` を参照してホーム ページを確認します。

Docker コンテナー内で手動で発行されたアプリを使用するには、新しい *Dockerfile* を作成し、`docker build .` コマンドを使用してイメージをビルドします。

::: moniker range=">= aspnetcore-5.0"

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS runtime
WORKDIR /app
COPY published/aspnetapp.dll ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

新しいイメージを確認するには、`docker images` コマンドを使用します。

### <a name="the-dockerfile"></a>Dockerfile

ここに示すのは、先ほど実行した `docker build` コマンドで使用された *Dockerfile* です。  このセクションで実行したときと同じ方法で `dotnet publish` を使用して、ビルドとデプロイを行います。  

```dockerfile
# https://hub.docker.com/_/microsoft-dotnet
FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /source

# copy csproj and restore as distinct layers
COPY *.sln .
COPY aspnetapp/*.csproj ./aspnetapp/
RUN dotnet restore

# copy everything else and build app
COPY aspnetapp/. ./aspnetapp/
WORKDIR /source/aspnetapp
RUN dotnet publish -c release -o /app --no-restore

# final stage/image
FROM mcr.microsoft.com/dotnet/aspnet:5.0
WORKDIR /app
COPY --from=build /app ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

前の *Dockerfile* では、`*.csproj` ファイルは別個の "*レイヤー*" としてコピーおよび復元されます。 `docker build` コマンドを使用してイメージをビルドすると、組み込みのキャッシュが使用されます。 `docker build` コマンドが最後に実行されてから `*.csproj` ファイルが変更されていない場合、`dotnet restore` コマンドを再度実行する必要はありません。 代わりに、対応する `dotnet restore` レイヤーの組み込みキャッシュが再利用されます。 詳細については、「[Dockerfile を記述するためのベスト プラクティス](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache)」を参照してください。

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

```dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:3.0 AS runtime
WORKDIR /app
COPY published/aspnetapp.dll ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

### <a name="the-dockerfile"></a>Dockerfile

ここに示すのは、先ほど実行した `docker build` コマンドで使用された *Dockerfile* です。  このセクションで実行したときと同じ方法で `dotnet publish` を使用して、ビルドとデプロイを行います。  

```dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:3.0 AS build
WORKDIR /app

# copy csproj and restore as distinct layers
COPY *.sln .
COPY aspnetapp/*.csproj ./aspnetapp/
RUN dotnet restore

# copy everything else and build app
COPY aspnetapp/. ./aspnetapp/
WORKDIR /app/aspnetapp
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/core/aspnet:3.0 AS runtime
WORKDIR /app
COPY --from=build /app/aspnetapp/out ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

前の Dockerfile で示されているように、`*.csproj` ファイルは別個の "*レイヤー*" としてコピーおよび復元されます。 `docker build` コマンドを使用してイメージをビルドすると、組み込みのキャッシュが使用されます。 `docker build` コマンドが最後に実行されてから `*.csproj` ファイルが変更されていない場合、`dotnet restore` コマンドを再度実行する必要はありません。 代わりに、対応する `dotnet restore` レイヤーの組み込みキャッシュが再利用されます。 詳細については、「[Dockerfile を記述するためのベスト プラクティス](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache)」を参照してください。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

```dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.2 AS runtime
WORKDIR /app
COPY published/aspnetapp.dll ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

### <a name="the-dockerfile"></a>Dockerfile

ここに示すのは、先ほど実行した `docker build` コマンドで使用された *Dockerfile* です。 このセクションで実行したときと同じ方法で `dotnet publish` を使用して、ビルドとデプロイを行います。  

```dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.2 AS build
WORKDIR /app

# copy csproj and restore as distinct layers
COPY *.sln .
COPY aspnetapp/*.csproj ./aspnetapp/
RUN dotnet restore

# copy everything else and build app
COPY aspnetapp/. ./aspnetapp/
WORKDIR /app/aspnetapp
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/core/aspnet:2.2 AS runtime
WORKDIR /app
COPY --from=build /app/aspnetapp/out ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

::: moniker-end

## <a name="additional-resources"></a>その他の技術情報

* [Docker の build コマンド](https://docs.docker.com/engine/reference/commandline/build)
* [Docker の run コマンド](https://docs.docker.com/engine/reference/commandline/run)
* [ASP.NET Core の Docker サンプル](https://github.com/dotnet/dotnet-docker) (このチュートリアルで使用されたものです。)
* [プロキシ サーバーとロード バランサーを使用するために ASP.NET Core を構成する](../proxy-load-balancer.md)
* [Visual Studio Docker ツールの使用](./visual-studio-tools-for-docker.md)
* [Visual Studio Code でのデバッグ](https://code.visualstudio.com/docs/nodejs/debugging-recipes#_debug-nodejs-in-docker-containers)
* [ドッカーと小さなコンテナを使用した GC](xref:performance/memory#sc)

## <a name="next-steps"></a>次の手順

同じアプリを格納している Git リポジトリにも、ドキュメントが用意されています。 リポジトリ内にある利用可能なリソースの概要については、[README ファイル](https://github.com/dotnet/dotnet-docker/blob/main/samples/aspnetapp/README.md)をご覧ください。 特に、HTTPS を実装する方法について確認してください。

> [!div class="nextstepaction"]
> 「[Developing ASP.NET Core Applications with Docker over HTTPS (Docker を使用して HTTPS による ASP.NET Core アプリケーションを開発する)](https://github.com/dotnet/dotnet-docker/blob/main/samples/run-aspnetcore-https-development.md)」
