---
title: HTTPS を使用した docker 構成を使用したコンテナー内の ASP.NET Core イメージのホスト
author: ravipal
description: HTTPS 経由で Docker Compose を使用して ASP.NET Core イメージをホストする方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: ravipal
ms.custom: mvc
ms.date: 03/28/2020
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
uid: security/docker-compose-https
ms.openlocfilehash: 69c449527a64ec928d436245e9e971ec49303489
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588758"
---
# <a name="hosting-aspnet-core-images-with-docker-compose-over-https"></a>HTTPS 経由の Docker Compose を使用した ASP.NET Core イメージのホスト


[既定では、](./enforcing-ssl.md)ASP.NET Core で HTTPS が使用されます。 [HTTPS](https://en.wikipedia.org/wiki/HTTPS) は、信頼、id、および暗号化のための [証明書](https://en.wikipedia.org/wiki/Public_key_certificate) に依存します。

このドキュメントでは、HTTPS で事前に構築されたコンテナーイメージを実行する方法について説明します。

開発シナリオについては、「 [HTTPS 経由の Docker を使用した ASP.NET Core アプリケーションの開発](https://github.com/dotnet/dotnet-docker/blob/main/samples/run-aspnetcore-https-development.md) 」を参照してください。

このサンプルには、[Docker 17.06](https://docs.docker.com/release-notes/docker-ce) 以降の [Docker クライアント](https://www.docker.com/products/docker)が必要です。

## <a name="prerequisites"></a>前提条件

このドキュメントの一部の手順では、 [.Net Core 2.2 SDK](https://dotnet.microsoft.com/download) 以降が必要です。

## <a name="certificates"></a>証明書

ドメインの[運用ホスト](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/)には、[証明機関](https://wikipedia.org/wiki/Certificate_authority)からの証明書が必要です。 [Let's Encrypt](https://letsencrypt.org/) は、無料の証明書を提供する証明機関です。

このドキュメントでは、事前に構築されたイメージをホストするために [自己署名の開発証明書](https://wikipedia.org/wiki/Self-signed_certificate) を使用 `localhost` します。 手順は、実稼働証明書の使用に似ています。

実稼働証明書の場合:

* `dotnet dev-certs`ツールは必要ありません。
* 手順で使用した場所に証明書を保存する必要はありません。 サイトディレクトリの外部にある任意の場所に証明書を格納します。

次のセクションに記載されている手順では、 `volumes` *docker-compose.yml* のプロパティを使用して、証明書をコンテナーにマウントします。 Dockerfile でコマンドを使用してコンテナーイメージに証明書を追加することもでき `COPY` ますが、この方法はお勧めしません。  証明書をイメージにコピーすることは、次の理由から推奨されません。

* これにより、開発者証明書でのテストに同じイメージを使用することが難しくなります。
* これにより、運用証明書でのホストに同じイメージを使用することが難しくなります。
* 証明書の公開には大きなリスクがあります。

## <a name="starting-a-container-with-https-support-using-docker-compose"></a>Docker 構成を使用して https をサポートするコンテナーを開始する

オペレーティングシステムの構成については、次の手順に従います。

### <a name="windows-using-linux-containers"></a>Linux コンテナーを使用した Windows

証明書を生成してローカルコンピューターを構成する:

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

上記のコマンドで、を `{ password here }` パスワードに置き換えます。

次の内容を含む _docker-compose.yml_ ファイルを作成します。

```yaml
version: '3.4'

services:
  webapp:
    image: mcr.microsoft.com/dotnet/core/samples:aspnetapp
    ports:
      - 80
      - 443
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Password=password
      - ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx
    volumes:
      - ~/.aspnet/https:/https:ro
```
Docker 構成ファイルで指定されたパスワードは、証明書に使用されるパスワードと一致する必要があります。

HTTPS 用に構成された ASP.NET Core でコンテナーを起動します。

```console
docker-compose -f "docker-compose.debug.yml" up -d
```

### <a name="macos-or-linux"></a>macOS または Linux

証明書を生成してローカルコンピューターを構成する:

```dotnetcli
dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

`dotnet dev-certs https --trust` は、macOS と Windows でのみサポートされています。 ディストリビューションでサポートされている方法で Linux 上の証明書を信頼する必要があります。 ブラウザーで証明書を信頼する必要があると考えられます。

上記のコマンドで、を `{ password here }` パスワードに置き換えます。

次の内容を含む _docker-compose.yml_ ファイルを作成します。

```yaml
version: '3.4'

services:
  webapp:
    image: mcr.microsoft.com/dotnet/core/samples:aspnetapp
    ports:
      - 80
      - 443
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Password=password
      - ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx
    volumes:
      - ~/.aspnet/https:/https:ro
```
Docker 構成ファイルで指定されたパスワードは、証明書に使用されるパスワードと一致する必要があります。

HTTPS 用に構成された ASP.NET Core でコンテナーを起動します。

```console
docker-compose -f "docker-compose.debug.yml" up -d
```

### <a name="windows-using-windows-containers"></a>Windows コンテナーを使用した windows

証明書を生成してローカルコンピューターを構成する:

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

上記のコマンドで、を `{ password here }` パスワードに置き換えます。

次の内容を含む _docker-compose.yml_ ファイルを作成します。

```yaml
version: '3.4'

services:
  webapp:
    image: mcr.microsoft.com/dotnet/core/samples:aspnetapp
    ports:
      - 80
      - 443
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Password=password
      - ASPNETCORE_Kestrel__Certificates__Default__Path=C:\https\aspnetapp.pfx
    volumes:
      - ${USERPROFILE}\.aspnet\https:C:\https:ro
```
Docker 構成ファイルで指定されたパスワードは、証明書に使用されるパスワードと一致する必要があります。

HTTPS 用に構成された ASP.NET Core でコンテナーを起動します。

```console
docker-compose -f "docker-compose.debug.yml" up -d
```
