---
title: ASP.NET Core Blazor Server のホストと展開
author: guardrex
description: ASP.NET Core を使用して Blazor Server アプリをホストおよびデプロイする方法について学習します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/26/2020
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
uid: blazor/host-and-deploy/server
ms.openlocfilehash: 75682171a59a610a24364778616774c49257d2ad
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100279844"
---
# <a name="host-and-deploy-blazor-server"></a>Blazor Server のホストと展開

## <a name="host-configuration-values"></a>ホストの構成値

[Blazor Server アプリ](xref:blazor/hosting-models#blazor-server)では、[汎用ホスト構成値](xref:fundamentals/host/generic-host#host-configuration)を受け入れることができます。

## <a name="deployment"></a>配置

[Blazor Server のホスティング モデル](xref:blazor/hosting-models#blazor-server)を使用する場合、Blazor はサーバー上で ASP.NET Core アプリ内から実行されます。 UI の更新、イベント処理、JavaScript の呼び出しは、[SignalR](xref:signalr/introduction) 接続経由で処理されます。

ASP.NET Core アプリをホストできる Web サーバーが必要です。 Visual Studio には **Blazor Server アプリ** プロジェクト テンプレートが含まれています ([`dotnet new`](/dotnet/core/tools/dotnet-new) コマンドを使用する場合は `blazorserverside` テンプレート)。

## <a name="scalability"></a>スケーラビリティ

Blazor Server アプリで使用できるインフラストラクチャを最大限に活用できるようにデプロイを計画します。 Blazor Server アプリのスケーラビリティに対処するには、次のリソースを参照してください。

* [Blazor Server アプリの基礎](xref:blazor/hosting-models#blazor-server)
* <xref:blazor/security/server/threat-mitigation>

### <a name="deployment-server"></a>展開サーバー

1 台のサーバーのスケーラビリティ (スケールアップ) を検討する場合、アプリに使用できるメモリは、ユーザーの要求が増えたときにアプリによって使い果たされる最初のリソースになります。 サーバー上の使用可能なメモリは、以下の影響を受けます。

* サーバーがサポートできるアクティブ回線の数。
* クライアントでの UI の待機時間。

セキュリティで保護されたスケーラブルな Blazor サーバー アプリを構築するためのガイダンスについては、「<xref:blazor/security/server/threat-mitigation>」を参照してください。

各回線では、最小限の *Hello World* スタイルのアプリに約 250 KB のメモリが使用されます。 回線のサイズは、アプリのコードと各コンポーネントに関連付けられている状態の保守要件によって変わります。 アプリとインフラストラクチャの開発時にはリソースのニーズを測定することをお勧めしますが、展開ターゲットを計画する際に、次のベースラインを出発点にすることができます。アプリで 5,000 人の同時ユーザーをサポートすることを想定している場合は、アプリに対して少なくとも 1.3 GB のサーバー メモリ (またはユーザーあたり最大 273 KB) の予算を割り当てること検討してください。

### <a name="signalr-configuration"></a>SignalR 構成

Blazor Server アプリでは、ブラウザーとの通信に ASP.NET Core SignalR が使用されます。 [SignalR のホストとスケーリングの条件](xref:signalr/publish-to-azure-web-app)は、Blazor Server アプリに適用されます。

Blazor は、待ち時間、信頼性、および[セキュリティ](xref:signalr/security)が低いために WebSocket を SignalR トランスポートとして使用する場合に最適です。 WebSocket が使用できない場合や、ロング ポーリングを使用するようにアプリが明示的に構成されている場合は、SignalR によってロング ポーリングが使用されます。 Azure App Service にデプロイする場合は、サービスの Azure portal 設定で WebSocket を使用するようにアプリを構成します。 Azure App Service 用にアプリを構成する方法の詳細については、[SignalR の発行ガイドライン](xref:signalr/publish-to-azure-web-app)を参照してください。

#### <a name="azure-signalr-service"></a>Azure SignalR Service

Blazor Server アプリには [Azure SignalR Service](xref:signalr/scale#azure-signalr-service) を使用することをお勧めします。 このサービスでは、多数の同時 SignalR 接続に対して Blazor Server アプリをスケールアップできます。 さらに、SignalR サービスのグローバル リーチとハイパフォーマンスのデータ センターは、地理的条件による待機時間の短縮に役立ちます。

> [!IMPORTANT]
> [WebSocket](https://wikipedia.org/wiki/WebSocket) が無効になっている場合、Azure App Service によって、リアルタイム接続のシミュレーションが HTTP ロング ポーリングを使用して行われます。 HTTP ロング ポーリングは、WebSocket を有効にして、クライアントとサーバー間の接続のシミュレーションにポーリングを使用せずに実行する場合よりも著しく遅くなります。
>
> Azure App Service にデプロイされた Blazor Server アプリには WebSocket を使用することをお勧めします。 [Azure SignalR Service](xref:signalr/scale#azure-signalr-service) では、既定で WebSocket が使用されます。 アプリで Azure SignalR Service を使用しない場合は、「<xref:signalr/publish-to-azure-web-app#configure-the-app-in-azure-app-service>」を参照してください。
>
> 詳細については、次を参照してください。
>
> * [Azure SignalR Service とは](/azure/azure-signalr/signalr-overview)
> * [Azure SignalR Service のパフォーマンス ガイド](/azure-signalr/signalr-concept-performance#performance-factors)

### <a name="configuration"></a>構成

Azure SignalR サービス用にアプリを構成するには、そのアプリで "*スティッキー セッション*" をサポートする必要があります。それにより、クライアントは [プリレンダリングのときに同じサーバーにリダイレクトされます](xref:blazor/hosting-models#connection-to-the-server)。 `ServerStickyMode` オプションまたは構成値は `Required` に設定されます。 通常、アプリでは次の方法のいずれか **_1 つ_** を使用して構成を作成します。

   * `Startup.ConfigureServices`:
  
     ```csharp
     services.AddSignalR().AddAzureSignalR(options =>
     {
         options.ServerStickyMode = 
             Microsoft.Azure.SignalR.ServerStickyMode.Required;
     });
     ```

   * 構成 (次の方法のいずれか **_1 つ_** を使用):
  
     * `appsettings.json`:

       ```json
       "Azure:SignalR:StickyServerMode": "Required"
       ```

     * Azure portal で App Service の **[構成]**  >  **[アプリケーションの設定]** (**名前**: `Azure__SignalR__StickyServerMode`、**値**: `Required`)。 この方法は、[Azure SignalR サービスをプロビジョニングする](#provision-the-azure-signalr-service)場合に、アプリに自動的に適用されます。

### <a name="provision-the-azure-signalr-service"></a>Azure SignalR サービスをプロビジョニングする

Visual Studio でアプリに合わせて Azure SignalR サービスをプロビジョニングするには、次のようにします。

1. Blazor Server アプリ用に、Visual Studio に Azure アプリ発行プロファイルを作成する。
1. プロファイルに **Azure SignalR Service** の依存関係を追加する。 Azure サブスクリプションに、アプリに割り当てる既存の Azure SignalR Service のインスタンスがない場合は、 **[新しい Azure SignalR Service のインスタンスを作成する]** を選択して新しいサービス インスタンスをプロビジョニングします。
1. Azure にアプリを公開します。

Visual Studio で Azure SignalR サービスを自動的にプロビジョニングすると、["*スティッキー セッション*"](#configuration) が有効になり、SignalR 接続文字列が App Service の構成に追加されます。

#### <a name="iis"></a>IIS

IIS を使用する場合は、次を有効にします。

* [IIS での WebSockets](xref:fundamentals/websockets#enabling-websockets-on-iis)
* [アプリケーション要求ルーティング処理でのスティッキー セッション](/iis/extensions/configuring-application-request-routing-arr/http-load-balancing-using-application-request-routing)

#### <a name="kubernetes"></a>Kubernetes

次の[スティッキー セッションに対する Kubernetes 注釈](https://kubernetes.github.io/ingress-nginx/examples/affinity/cookie/)を使用して、イングレス定義を作成します。

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: <ingress-name>
  annotations:
    nginx.ingress.kubernetes.io/affinity: "cookie"
    nginx.ingress.kubernetes.io/session-cookie-name: "affinity"
    nginx.ingress.kubernetes.io/session-cookie-expires: "14400"
    nginx.ingress.kubernetes.io/session-cookie-max-age: "14400"
```

#### <a name="linux-with-nginx"></a>Nginx を使用した Linux

SignalR WebSocket が正常に機能するには、プロキシの `Upgrade` と `Connection` のヘッダーが次の値に設定されていること、および `$connection_upgrade` が次のどちらかにマップされていることを確認します。

* アップグレード ヘッダーの値 (既定)。
* アップグレード ヘッダーが存在しないか、空である場合は `close`。

```
http {
    map $http_upgrade $connection_upgrade {
        default Upgrade;
        ''      close;
    }

    server {
        listen      80;
        server_name example.com *.example.com
        location / {
            proxy_pass         http://localhost:5000;
            proxy_http_version 1.1;
            proxy_set_header   Upgrade $http_upgrade;
            proxy_set_header   Connection $connection_upgrade;
            proxy_set_header   Host $host;
            proxy_cache_bypass $http_upgrade;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Proto $scheme;
        }
    }
}
```

詳細については、次の記事を参照してください。

* [WebSocket プロキシとしての NGINX](https://www.nginx.com/blog/websocket-nginx/)
* [WebSocket プロキシ](http://nginx.org/docs/http/websocket.html)
* <xref:host-and-deploy/linux-nginx>

## <a name="linux-with-apache"></a>Apache を使用した Linux

Linux 上の Apache の背後に Blazor アプリをホストするには、HTTP および Websocket トラフィック用に `ProxyPass` を構成します。

次に例を示します。

* Kestrel サーバーは、ホスト コンピューター上で実行されています。
* このアプリは、ポート 5000 でトラフィックをリッスンします。

```
ProxyRequests       On
ProxyPreserveHost   On
ProxyPassMatch      ^/_blazor/(.*) http://localhost:5000/_blazor/$1
ProxyPass           /_blazor ws://localhost:5000/_blazor
ProxyPass           / http://localhost:5000/
ProxyPassReverse    / http://localhost:5000/
```

次のモジュールを有効にします。

```
a2enmod   proxy
a2enmod   proxy_wstunnel
```

WebSockets エラーをブラウザ コンソールで確認します。 エラーの例:

* Firefox は ws://the-domain-name.tld/_blazor?id=XXX のサーバーに接続を確立できません。
* エラー :'WebSockets' の転送の開始に失敗しました。エラー :転送のエラーがありました。
* エラー :'LongPolling' の転送の開始に失敗しました。TypeError: this.transport は定義されていません
* エラー :利用可能ないかなる転送を使用しても、サーバーに接続できませんでした。 WebSockets が失敗しました
* エラー :接続が 'Connected' 状態ではない場合、データは送信できません。

詳細については、[Apache のドキュメント](https://httpd.apache.org/docs/current/mod/mod_proxy.html)を参照してください。

### <a name="measure-network-latency"></a>ネットワーク待機時間の測定

次の例に示すように、[JS 相互運用機能](xref:blazor/call-javascript-from-dotnet)を使用してネットワーク待機時間を測定できます。

```razor
@inject IJSRuntime JS

@if (latency is null)
{
    <span>Calculating...</span>
}
else
{
    <span>@(latency.Value.TotalMilliseconds)ms</span>
}

@code {
    private DateTime startTime;
    private TimeSpan? latency;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            startTime = DateTime.UtcNow;
            var _ = await JS.InvokeAsync<string>("toString");
            latency = DateTime.UtcNow - startTime;
            StateHasChanged();
        }
    }
}
```

妥当な UI エクスペリエンスを実現するために、UI の待機時間を 250 ミリ秒以下にすることをお勧めします。
