---
title: SignalR運用環境のホスティングとスケーリングの ASP.NET Core
author: bradygaster
description: ASP.NET Core を使用するアプリのパフォーマンスとスケーリングに関する問題を回避する方法について説明 SignalR します。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 01/17/2020
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
uid: signalr/scale
ms.openlocfilehash: e70f3143159a1817e326a95b30e7369a5c9ab025
ms.sourcegitcommit: f77a7467651bab61b24261da9dc5c1dd75fc1fa9
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/17/2021
ms.locfileid: "100564004"
---
# <a name="aspnet-core-signalr-hosting-and-scaling"></a>ASP.NET Core SignalR のホストとスケーリング

( [Andrew Stanton](https://twitter.com/anurse))、 [Brady](https://twitter.com/bradygaster)氏、 [Tom Dykstra](https://github.com/tdykstra)

この記事では、ASP.NET Core を使用する高トラフィックアプリのホストとスケーリングに関する考慮事項について説明 SignalR します。

## <a name="sticky-sessions"></a>固定セッション

SignalR では、特定の接続に対するすべての HTTP 要求を同じサーバープロセスで処理する必要があります。 SignalRサーバーファーム (複数のサーバー) でが実行されている場合は、"固定セッション" を使用する必要があります。 "固定セッション" は、一部のロードバランサーによってセッションアフィニティとも呼ばれます。 Azure App Service は、 [アプリケーション要求ルーティング](/iis/extensions/planning-for-arr/application-request-routing-version-2-overview) 処理 (ARR) を使用して要求をルーティングします。 Azure App Service で "ARR Affinity" 設定を有効にすると、"固定セッション" が有効になります。 固定セッションが不要な状況は次のとおりです。

1. 単一のサーバーでホストする場合、1つのプロセスで実行します。
1. Azure サービスを使用する場合 SignalR 。
1. すべてのクライアントが Websocket **のみ** を使用するように構成され、 [skipnegotiation 設定](xref:signalr/configuration#configure-additional-options)がクライアント構成で有効に **なっている** 場合。

その他のすべての状況 (Redis バックプレーンを使用する場合を含む) では、サーバー環境を固定セッション用に構成する必要があります。

の Azure App Service の構成に関するガイダンスについ SignalR ては、「」を参照してください <xref:signalr/publish-to-azure-web-app> 。

## <a name="tcp-connection-resources"></a>TCP 接続リソース

Web サーバーがサポートできる同時 TCP 接続の数は制限されています。 標準 HTTP クライアントは、 *一時的* な接続を使用します。 これらの接続は、クライアントがアイドル状態になったときに終了し、後で再度開くことができます。 一方、 SignalR 接続は *永続的* です。 SignalR クライアントがアイドル状態になった場合でも、接続は開いたままになります。 多数のクライアントにサービスを提供する高トラフィックアプリでは、これらの永続的な接続により、サーバーが最大接続数に達する可能性があります。

また、固定接続では、各接続を追跡するために、追加のメモリがいくつか消費されます。

によって接続関連のリソースが頻繁に使用 SignalR される場合、同じサーバー上でホストされている他の web アプリに影響を与えることがあります。 SignalRが開いていて、最後に使用可能な TCP 接続が保持されている場合、同じサーバー上の他の web アプリにも使用できる接続がありません。

サーバーの接続が不足している場合は、ランダムソケットエラーと接続リセットエラーが表示されます。 次に例を示します。

```
An attempt was made to access a socket in a way forbidden by its access permissions...
```

他の web SignalR アプリでリソースの使用状況が発生しないようにするには、 SignalR 他の web アプリとは異なるサーバー上でを実行します。

リソースの SignalR 使用状況によってアプリでエラーが発生しないようにするには SignalR 、スケールアウトして、サーバーが処理する接続の数を制限します。

## <a name="scale-out"></a>スケール アウト

を使用するアプリでは、 SignalR すべての接続を追跡する必要があります。これにより、サーバーファームに関する問題が発生します。 サーバーを追加すると、他のサーバーが認識していない新しい接続が取得されます。 たとえば、 SignalR 次の図の各サーバーでは、他のサーバー上の接続が認識されません。 SignalRいずれかのサーバーですべてのクライアントにメッセージを送信する場合、メッセージは、そのサーバーに接続されているクライアントのみに送信されます。

![スケーリング::: なし (SignalR)::: バックプレーンなし](scale/_static/scale-no-backplane.png)

この問題を解決するためのオプションは、 [Azure SignalR サービス](#azure-signalr-service) と [Redis バックプレーン](#redis-backplane)です。

## <a name="azure-signalr-service"></a>Azure SignalR Service

Azure SignalR サービスは、バックプレーンではなくプロキシです。 クライアントがサーバーへの接続を開始するたびに、クライアントはサービスに接続するためにリダイレクトされます。 このプロセスを次の図に示します。

![Azure::: no loc (SignalR)::: Service への接続を確立しています](scale/_static/azure-signalr-service-one-connection.png)

その結果、次の図に示すように、サービスはすべてのクライアント接続を管理しますが、各サーバーにはサービスへの接続数をごくわずかにする必要があります。

![サービスに接続されているクライアント、サービスに接続されているサーバー](scale/_static/azure-signalr-service-multiple-connections.png)

このスケールアウトのアプローチには、Redis バックプレーンよりもいくつかの利点があります。

* クライアント [アフィニティ](/iis/extensions/configuring-application-request-routing-arr/http-load-balancing-using-application-request-routing#step-3---configure-client-affinity)とも呼ばれる固定セッションは、接続時にクライアントが Azure サービスに直ちにリダイレクトされるため、必須ではありません SignalR 。
* アプリは、 SignalR 送信されたメッセージの数に基づいてスケールアウトできます。一方、Azure サービスは、 SignalR 任意の数の接続を処理するようにスケーリングします。 たとえば、クライアントが何千も存在する可能性はありますが、1秒あたり数個のメッセージしか送信されない場合、 SignalR アプリは接続自体を処理するためだけに複数のサーバーにスケールアウトする必要がありません。
* アプリでは SignalR 、を使用しないと、web アプリよりもはるかに多くの接続リソースが使用されません SignalR 。

これらの理由から、 SignalR SignalR App Service、vm、コンテナーなど、azure でホストされているすべての ASP.NET Core アプリに対して azure サービスを使用することをお勧めします。

詳細については、 [Azure SignalR サービスのドキュメント](/azure/azure-signalr/signalr-overview)を参照してください。

## <a name="redis-backplane"></a>Redis バックプレーン

[Redis](https://redis.io/) は、パブリッシュ/サブスクライブモデルを持つメッセージングシステムをサポートするメモリ内キー値ストアです。 SignalRRedis バックプレーンは、pub/sub 機能を使用して、メッセージを他のサーバーに転送します。 クライアントが接続を確立すると、接続情報がバックプレーンに渡されます。 サーバーは、すべてのクライアントにメッセージを送信するときに、バックプレーンに送信します。 バックプレーンは、接続されているすべてのクライアントと、それらがあるサーバーを認識します。 このメッセージは、各サーバーを介してすべてのクライアントに送信されます。 このプロセスを次の図に示します。

![Redis バックプレーン、1つのサーバーからすべてのクライアントに送信されたメッセージ](scale/_static/redis-backplane.png)

Redis バックプレーンは、お客様のインフラストラクチャでホストされているアプリに推奨されるスケールアウトアプローチです。 データセンターと Azure データセンターの間に大きな接続の待機時間がある場合、 SignalR 低待機時間または高いスループットの要件を持つオンプレミスのアプリでは、azure サービスが実用的な選択肢ではない可能性があります。

前述した Azure SignalR サービスの利点は、Redis バックプレーンの欠点です。

* 次の **両方** が当てはまる場合を除き、固定セッション ([クライアントアフィニティ](/iis/extensions/configuring-application-request-routing-arr/http-load-balancing-using-application-request-routing#step-3---configure-client-affinity)とも呼ばれます) が必要です。
  * すべてのクライアントは、Websocket **のみ** を使用するように構成されています。
  * クライアント構成で [Skipnegotiation 設定](xref:signalr/configuration#configure-additional-options) が有効になっています。 
   サーバーで接続が開始されると、接続はそのサーバー上にとどまります。
* アプリは、 SignalR 送信されるメッセージが少ない場合でも、クライアントの数に基づいてスケールアウトする必要があります。
* アプリは、を SignalR 使用しない場合、web アプリよりもはるかに多くの接続リソースを使用 SignalR します。

## <a name="iis-limitations-on-windows-client-os"></a>Windows クライアント OS での IIS の制限事項

Windows 10 と Windows 8.x はクライアントオペレーティングシステムです。 クライアントオペレーティングシステムの IIS では、同時接続数が10個に制限されています。 SignalRの接続は次のとおりです。

* 一時的で、頻繁に再確立されます。
* 使用されなくなった場合、すぐに **は破棄されません**。

上記の条件を満たすと、クライアント OS で10個の接続制限に達する可能性が高くなります。 クライアント OS を開発に使用する場合は、次のことをお勧めします。

* IIS を避けます。
* 配置ターゲットとして Kestrel または IIS Express を使用します。

## <a name="linux-with-nginx"></a>Nginx を使用した Linux

次の表に、の Websocket、Serverのイベント、および LongPolling を有効にするために最低限必要な設定を示し SignalR ます。

```nginx
http {
  map $http_connection $connection_upgrade {
    "~*Upgrade" $http_connection;
    default keep-alive;
}

  server {
    listen 80;
    server_name example.com *.example.com;

    # Configure the SignalR Endpoint
    location /hubroute {
      # App server url
      proxy_pass http://localhost:5000;

      # Configuration for WebSockets
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;
      proxy_cache off;

      # Configuration for ServerSentEvents
      proxy_buffering off;

      # Configuration for LongPolling or if your KeepAliveInterval is longer than 60 seconds
      proxy_read_timeout 100s;

      proxy_set_header Host $host;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
    }
  }
}
```

複数のバックエンドサーバーを使用する場合は、接続時に接続がサーバーを切り替えるのを防ぐために、固定セッションを追加する必要があり SignalR ます。 Nginx では、複数の方法で固定セッションを追加できます。 次に、使用している内容に応じて2つの方法を示します。

前の構成に加えて、次のものも追加されています。 次の例で `backend` は、はサーバーのグループの名前です。

[Nginx オープンソース](https://nginx.org/en/)では、を使用して、 `ip_hash` クライアントの IP アドレスに基づいてサーバーへの接続をルーティングします。

```nginx
http {
  upstream backend {
    # App server 1
    server http://localhost:5000;
    # App server 2
    server http://localhost:5002;

    ip_hash;
  }
}
```

[Nginx Plus](https://www.nginx.com/products/nginx)では、を使用し `sticky` てを cookie 要求に追加し、ユーザーの要求をサーバーにピン留めします。

```nginx
http {
  upstream backend {
    # App server 1
    server http://localhost:5000;
    # App server 2
    server http://localhost:5002;

    sticky cookie srv_id expires=max domain=.example.com path=/ httponly;
  }
}
```

最後に、 `proxy_pass http://localhost:5000` セクションの `server` をに変更し `proxy_pass http://backend` ます。

Websocket over Nginx の詳細については、「 [Nginx as a WebSocket Proxy](https://www.nginx.com/blog/websocket-nginx)」を参照してください。

負荷分散と固定セッションの詳細については、「 [NGINX load balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)」を参照してください。

Nginx の ASP.NET Core の詳細については、次の記事を参照してください。
* <xref:host-and-deploy/linux-nginx>

## <a name="third-party-signalr-backplane-providers"></a>サードパーティ製 SignalR バックプレーンプロバイダー

* [NCache](https://www.alachisoft.com/ncache/asp-net-core-signalr.html)
* [Orleans](https://github.com/OrleansContrib/SignalR.Orleans)
* [Rebus](https://github.com/rebus-org/Rebus.SignalR)

## <a name="next-steps"></a>次のステップ

詳細については、次のリソースを参照してください。

* [Azure SignalR サービスのドキュメント](/azure/azure-signalr/signalr-overview)
* [Redis バックプレーンを設定する](xref:signalr/redis-backplane)
