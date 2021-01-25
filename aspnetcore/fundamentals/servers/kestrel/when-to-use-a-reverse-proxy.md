---
title: ASP.NET Core Kestrel Web サーバーでリバース プロキシを使用するタイミング
author: rick-anderson
description: ASP.NET Core 向けのクロスプラットフォーム Web サーバーである Kestrel の前で、リバース プロキシを使用するタイミングについて学習します。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 01/14/2021
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
uid: fundamentals/servers/kestrel/when-to-use-a-reverse-proxy
ms.openlocfilehash: fc89a9f841403bbccedff0a9c0720a08c11abdd6
ms.sourcegitcommit: 063a06b644d3ade3c15ce00e72a758ec1187dd06
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98253840"
---
# <a name="when-to-use-kestrel-with-a-reverse-proxy"></a>Kestrel とリバース プロキシを使用するタイミング

Kestrel を単独で使用することも、[インターネット インフォメーション サービス (IIS)](https://www.iis.net/)、[Nginx](https://nginx.org)、[Apache](https://httpd.apache.org/) などの *リバース プロキシ サーバー* と併用することもできます。 リバース プロキシ サーバーはネットワークから HTTP 要求を受け取り、これを Kestrel に転送します。

エッジ (インターネットに接続する) Web サーバーとして使用される Kestrel:

![リバース プロキシ サーバーなしでインターネットと直接通信する Kestrel](_static/kestrel-to-internet2.png)

リバース プロキシ構成で使用される Kestrel:

![IIS、Nginx、または Apache などのリバース プロキシ サーバーを介してインターネットと間接的に通信する Kestrel](_static/kestrel-to-internet.png)

いずれの構成でも、リバース プロキシ サーバーの有無に関わらず、ホスティング構成がサポートされています。

リバース プロキシ サーバーのないエッジ サーバーとして Kestrel を使用する場合、複数のプロセス間で同じ IP アドレスとポートを共有することはサポートされません。 あるポートをリッスンするように Kestrel を構成する場合は、要求の `Host` ヘッダーに関係なく、Kestrel によってそのポートに対するすべてのトラフィックが処理されます。 ポートを共有できるリバース プロキシを使用すると、一意の IP とポート上で Kestrel に要求を転送することができます。

リバース プロキシ サーバーが必要ない場合であっても、リバース プロキシ サーバーを使用すると次のような利点があります。

リバース プロキシ:

* ホストするアプリの公開されるパブリック サーフェス領域を制限することができます。
* 構成および防御の層を追加します。
* 既存のインフラストラクチャとより適切に統合できる場合があります。
* 負荷分散とセキュリティで保護された通信 (HTTPS) の構成を簡略化します。 リバース プロキシ サーバーのみで X.509 証明書が必要です。このサーバーでは、プレーンな HTTP を使用して内部ネットワーク上のアプリのサーバーと通信することができます。

> [!WARNING]
> リバース プロキシ構成でのホストには[ホストのフィルター処理](xref:fundamentals/servers/kestrel/host-filtering)が必要です。

## <a name="additional-resources"></a>その他のリソース

<xref:host-and-deploy/proxy-load-balancer>

