---
title: ASP.NET Core Kestrel Web サーバーでのホストのフィルター処理
author: rick-anderson
description: ASP.NET Core 用のクロスプラットフォーム Web サーバーである Kestrel でのホストのフィルター処理の使用について説明します。
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
uid: fundamentals/servers/kestrel/host-filtering
ms.openlocfilehash: d55c211f05a77f6acabedef2ff62a621d9a1844e
ms.sourcegitcommit: 063a06b644d3ade3c15ce00e72a758ec1187dd06
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98253877"
---
# <a name="host-filtering-with-aspnet-core-kestrel-web-server"></a>ASP.NET Core Kestrel Web サーバーでのホストのフィルター処理

Kestrel は `http://example.com:5000` などのプレフィックスに基づく構成をサポートしますが、Kestrel はほとんどのホスト名を無視します。 ホスト `localhost` は、ループバック アドレスへのバインドに使用される特殊なケースです。 明示的な IP アドレス以外のすべてのホストは、すべてのパブリック IP アドレスにバインドします。 `Host` ヘッダーは検証されません。

これを解決するには、Host Filtering Middleware を使用します。 Host Filtering Middleware は、ASP.NET Core アプリに暗黙的に含まれる、[Microsoft.AspNetCore.HostFiltering](https://www.nuget.org/packages/Microsoft.AspNetCore.HostFiltering) パッケージによって提供されています。 ミドルウェアは <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder%2A> によって追加され、そこでは <xref:Microsoft.AspNetCore.Builder.HostFilteringServicesExtensions.AddHostFiltering%2A> が呼び出されます。

[!code-csharp[](samples-snapshot/2.x/KestrelSample/Program.cs?name=snippet_Program&highlight=9)]

Host Filtering Middleware は既定では無効です。 このミドルウェアを有効にするには、 *appsettings.json* /*appsettings.\<EnvironmentName>.json* で `AllowedHosts` キーを定義します。 この値は、ポート番号を含まないホスト名のセミコロン区切りリストです。

*appsettings.json*:

```json
{
  "AllowedHosts": "example.com;localhost"
}
```

> [!NOTE]
> [Forwarded Headers Middleware](xref:host-and-deploy/proxy-load-balancer) にも <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.AllowedHosts> オプションがあります。 Forwarded Headers Middleware および Host Filtering Middleware には、異なるシナリオ用に類似した機能があります。 リバース プロキシ サーバーまたはロード バランサーを使用して要求を転送するとき、`Host` ヘッダーが保存されていない場合、Forwarded Headers Middleware に `AllowedHosts` を設定するのが適切です。 Kestrel が一般向けエッジ サーバーとして使用されていたり、`Host` ヘッダーが直接転送されたりしている場合、Host Filtering Middleware に `AllowedHosts` を設定するのが適切です。
>
> Forwarded Headers Middleware の詳細については、<xref:host-and-deploy/proxy-load-balancer> を参照してください。
