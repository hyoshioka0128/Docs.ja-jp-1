---
title: ASP.NET Core Kestrel Web サーバーで HTTP/2 を使用する
author: rick-anderson
description: ASP.NET Core 用のクロスプラットフォーム Web サーバーである Kestrel での HTTP/2 の使用について説明します。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 05/04/2020
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
uid: fundamentals/servers/kestrel/http2
ms.openlocfilehash: 431459bb6ece1d054146558ef865e44845a22686
ms.sourcegitcommit: 063a06b644d3ade3c15ce00e72a758ec1187dd06
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98253872"
---
# <a name="use-http2-with-the-aspnet-core-kestrel-web-server"></a>ASP.NET Core Kestrel Web サーバーで HTTP/2 を使用する

[Http/2](https://httpwg.org/specs/rfc7540.html) は、次の基本要件が満たされている場合に、ASP.NET Core アプリで使用できます。

* オペレーティング システム&dagger;
  * Windows Server 2016/Windows 10 以降&Dagger;
  * OpenSSL 1.0.2 以降を使用した Linux (Ubuntu 16.04 以降など)
* ターゲット フレームワーク: .NET Core 2.2 以降
* [アプリケーション レイヤー プロトコル ネゴシエーション (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) 接続
* TLS 1.2 以降の接続

&dagger;将来のリリースでは HTTP/2 が macOS 上でサポートされるようになります。
&Dagger;Kestrel では、Windows Server 2012 R2 および Windows 8.1 上での HTTP/2 のサポートは制限されています。 サポートが制限されている理由は、これらのオペレーティング システムで使用できる TLS 暗号のスイートのリストが制限されているためです。 TLS 接続をセキュリティで保護するためには、楕円曲線デジタル署名アルゴリズム (ECDSA) を使用して生成した証明書が必要になる場合があります。

Http/2 接続が確立されると、[HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol%2A) が `HTTP/2` を報告します。

.NET Core 3.0 以降では、HTTP/2 は既定で有効になっています。 構成の詳細については、[Kestrel HTTP/2 の制限](xref:fundamentals/servers/kestrel/options#http2-limits)に関するセクションと、「[ListenOptions.Protocols](xref:fundamentals/servers/kestrel/endpoints#listenoptionsprotocols)」セクションを参照してください。

## <a name="advanced-http2-features"></a>高度な HTTP/2 機能

Kestrel に追加された HTTP/2 機能により、gRPC がサポートされています。これには、応答トレーラーのサポートや、リセット フレームの送信のサポートが含まれます。

### <a name="trailers"></a>予告編

[!INCLUDE[](~/includes/trailers.md)]

### <a name="reset"></a>Reset

[!INCLUDE[](~/includes/reset.md)]
