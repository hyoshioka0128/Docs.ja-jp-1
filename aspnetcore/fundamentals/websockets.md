---
title: ASP.NET Core での Websocket のサポート
author: rick-anderson
description: ASP.NET Core で Websocket の使用を開始する方法を説明します。
monikerRange: '>= aspnetcore-1.1'
ms.author: tdykstra
ms.custom: mvc
ms.date: 11/06/2018
uid: fundamentals/websockets
ms.openlocfilehash: 3a649f88699d61636d9aa7fbfe4468ca67b3b018
ms.sourcegitcommit: fc7eb4243188950ae1f1b52669edc007e9d0798d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/07/2018
ms.locfileid: "51225409"
---
# <a name="websockets-support-in-aspnet-core"></a>ASP.NET Core での Websocket のサポート

作成者: [Tom Dykstra](https://github.com/tdykstra) および [Andrew Stanton-Nurse](https://github.com/anurse)

この記事では、ASP.NET Core で Websocket の使用を開始する方法について説明します。 [WebSocket](https://wikipedia.org/wiki/WebSocket) ([RFC 6455](https://tools.ietf.org/html/rfc6455)) は、TCP 接続を使用した双方向の永続的通信チャネルを有効にするプロトコルです。 このプロトコルは、チャット、ダッシュボード、ゲーム アプリなど、高速かつリアルタイムのコミュニケーションを活用するアプリで使用されます。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/websockets/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。 詳細については、「[次の手順](#next-steps)」のセクションを参照してください。

## <a name="prerequisites"></a>必須コンポーネント

* ASP.NET Core 1.1 以降
* ASP.NET Core をサポートする任意の OS:
  
  * Windows 7 / Windows Server 2008 以降
  * Linux
  * macOS
  
* アプリが IIS を含む Windows 上で実行されている場合:

  * Windows 8 / Windows Server 2012 以降
  * IIS 8 / IIS 8 Express
  * WebSockets を有効にする必要があります (「[IIS/IIS Express のサポート](#iisiis-express-support)」セクションを参照してください)。
  
* アプリが [HTTP.sys](xref:fundamentals/servers/httpsys) で実行されている場合:

  * Windows 8 / Windows Server 2012 以降

* サポートされているブラウザーについては、 https://caniuse.com/#feat=websockets を参照してください。

## <a name="when-to-use-websockets"></a>WebSockets を使用する場合

Websocket を使用して、ソケット接続を直接使用します。 たとえば、リアルタイムのゲームで最高のパフォーマンスが必要な場合に WebSockets を使用します。

[ASP.NET Core SignalR](xref:signalr/introduction) は、アプリへのリアルタイム Web 機能の追加を簡単にするライブラリです。 可能なかぎり、WebSocket が使用されます。

## <a name="how-to-use-websockets"></a>WebSocket の使用方法

* [Microsoft.AspNetCore.WebSockets](https://www.nuget.org/packages/Microsoft.AspNetCore.WebSockets/) パッケージをインストールします。
* ミドルウェアを構成します。
* WebSocket の要求を受け入れます。
* メッセージを送受信します。

### <a name="configure-the-middleware"></a>ミドルウェアの構成

`Startup` クラスの `Configure` メソッドに、Websocket ミドルウェアを追加します。

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=UseWebSockets)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

[!code-csharp[](websockets/samples/1.x/WebSocketsSample/Startup.cs?name=UseWebSockets)]

::: moniker-end

::: moniker range="< aspnetcore-2.2"

次の設定を構成できます。

* `KeepAliveInterval`: プロキシの接続の維持を保証する、クライアントに "ping" フレームを送信する頻度。 既定値は 2 分です。
* `ReceiveBufferSize`: データの受信に使用されるバッファーのサイズ。 これは、上級ユーザーが、データのサイズに応じたパフォーマンス調整のために変更する必要がある場合があります。 既定値は 4 KB です。

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

次の設定を構成できます。

* `KeepAliveInterval`: プロキシの接続の維持を保証する、クライアントに "ping" フレームを送信する頻度。 既定値は 2 分です。
* `ReceiveBufferSize`: データの受信に使用されるバッファーのサイズ。 これは、上級ユーザーが、データのサイズに応じたパフォーマンス調整のために変更する必要がある場合があります。 既定値は 4 KB です。
* `AllowedOrigins` - WebSocket 要求で許可される配信元ヘッダー値の一覧。 既定では、すべての配信元が許可されています。 詳細については、下記の "WebSocket の配信元の制限" を参照してください。

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=UseWebSocketsOptions)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

[!code-csharp[](websockets/samples/1.x/WebSocketsSample/Startup.cs?name=UseWebSocketsOptions)]

::: moniker-end

### <a name="accept-websocket-requests"></a>WebSocket の要求の受け入れ

以降の要求ライフサイクルのどこかで (たとえば、以降の `Configure` メソッドまたは MVC アクションで)、それが WebSocket 要求であるかを確認し、WebSocket 要求を受け入れます。

次の例は、以降の `Configure` メソッドから抜粋したものです。

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=AcceptWebSocket&highlight=7)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

[!code-csharp[](websockets/samples/1.x/WebSocketsSample/Startup.cs?name=AcceptWebSocket&highlight=7)]

::: moniker-end

WebSocket 要求はどの URL からも受け取る場合がありますが、このサンプル コードでは `/ws` の要求のみを受け取ります。

### <a name="send-and-receive-messages"></a>メッセージの送受信

`AcceptWebSocketAsync` メソッドは、TCP 接続を WebSocket 接続にアップグレードし、[WebSocket](/dotnet/core/api/system.net.websockets.websocket) オブジェクトを提供します。 メッセージの送受信に、`WebSocket` オブジェクトを使用します。

前に示した、WebSocket 要求を受け入れるコードが、`WebSocket` オブジェクトを `Echo` メソッドに渡します。 このコードは、メッセージを受信し、同じメッセージをすぐに送信します。 クライアントが接続を閉じるまで、メッセージがループで送受信されます。

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=Echo)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

[!code-csharp[](websockets/samples/1.x/WebSocketsSample/Startup.cs?name=Echo)]

::: moniker-end

このループを開始する前に、WebSocket 接続を受け入れた場合、ミドルウェア パイプラインは終了します。 ソケットを閉じると、パイプラインはアンワインドされます。 つまり、WebSocket を受け入れると、要求はパイプラインでの先への移動を中止します。 ループを終了し、ソケットを閉じた場合、要求はパイプラインのバックアップを続けます。

::: moniker range=">= aspnetcore-2.2"

### <a name="websocket-origin-restriction"></a>WebSocket の配信元の制限

CORS で提供される保護は、WebSocket には適用されません。 ブラウザーでは以下を実行**しません**。

* CORS の事前要求を実行する。
* WebSocket 要求を行うときに `Access-Control` ヘッダーに指定された制限を考慮する。

ただし、WebSocket 要求を発行するときにはブラウザーから `Origin` ヘッダーが送信されます。 予期した配信元からの WebSocket のみが許可されるように、アプリケーションでこれらのヘッダーが検証されるように構成する必要があります。

"https://server.com" でサーバーを、"https://client.com" でクライアントをホスティングしている場合は、検証のために "https://client.com" を WebSocket の `AllowedOrigins` 一覧に追加します。

```csharp
app.UseWebSockets(new WebSocketOptions()
{
    AllowedOrigins.Add("https://client.com");
    AllowedOrigins.Add("https://www.client.com");
});
```

> [!NOTE]
> `Origin` ヘッダーは、クライアントによって制御され、`Referer` のように偽装することができます。 これらのヘッダーを認証メカニズムとして使用**しないでください**。

::: moniker-end

## <a name="iisiis-express-support"></a>IIS/IIS Express のサポート

IIS/IIS Express 8 以降を含む、Windows Server 2012 以降および Windows 8 以降では、WebSocket プロトコルをサポートします。

> [!NOTE]
> IIS Express を使用する場合、WebSockets は常に有効になります。

### <a name="enabling-websockets-on-iis"></a>IIS での WebSockets の有効化

Windows Server 2012 以降で WebSocket プロトコルのサポートを有効にするには

> [!NOTE]
> これらの手順は、IIS Express を使用する場合は必要ありません

1. **[管理]** メニューから**役割と機能の追加**ウィザードを使用するか、**サーバー マネージャー**にあるリンクを使用します。
1. **[役割ベースまたは機能ベースのインストール]** を選択します。 **[次へ]** を選択します。
1. 適切なサーバーを選択します (既定では、ローカル サーバーが選択されます)。 **[次へ]** を選択します。
1. **[役割]** ツリーで **[Web サーバー (IIS)]** を展開し、**[Web サーバー]**、**[アプリケーション開発]** の順に展開します。
1. **[WebSocket プロトコル]** を選択します。 **[次へ]** を選択します。
1. 追加機能が不要な場合は、**[次へ]** を選択します。
1. **[インストール]** を選択します。
1. インストールが完了したら、**[閉じる]** を選択してウィザードを終了します。

Windows 8 以降で WebSocket プロトコルのサポートを有効にするには

> [!NOTE]
> これらの手順は、IIS Express を使用する場合は必要ありません

1. **[コントロール パネル]** > **[プログラム]** > **[プログラムと機能]** > **[Windows の機能の有効化または無効化]** (画面の左側) に移動します。
1. 次のノード: **[インターネット インフォメーション サービス]** > **[World Wide Web サービス]** > **[アプリケーション開発機能]** を開きます。
1. **[WebSocket プロトコル]** 機能を選択します。 **[OK]** を選択します。

### <a name="disable-websocket-when-using-socketio-on-nodejs"></a>Node.js で socket.io を使用する場合に WebSocket を無効にする

[Node.js](https://nodejs.org/) の [socket.io](https://socket.io/) で WebSocket サポートを使用する場合、`webSocket` 要素を使用する既定の WebSocket モジュールを *web.config* または *applicationHost.config* で無効にします。この手順が実行されない場合、IIS WebSocket モジュールは Node.js とアプリ以外の WebSocket コミュニケーションを処理しようとします。

```xml
<system.webServer>
  <webSocket enabled="false" />
</system.webServer>
```

## <a name="next-steps"></a>次の手順

この記事に添えられている[サンプル アプリ](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/websockets/samples)は、エコー アプリです。 これには、WebSocket 接続を作成する Web ページがあり、サーバーが受け取るすべてのメッセージをクライアントに再送信します。 コマンド プロンプトからアプリを実行し (IIS Express を使用した Visual Studio からは実行するように設定されていません)、 http://localhost:5000 に移動します。 Web ページの左上に、接続の状態が示されます。

![Web ページの初期状態](websockets/_static/start.png)

**[接続]** を選択し、表示されている URL に WebSocket 要求を送信します。 テスト メッセージを入力し、**[送信]** を選択します。 完了したら、**[Close Socket]\(ソケットを閉じる\)** を選択します。 **[Communication Log]\(コミュニケーション ログ\)** セクションに、発生した各オープン、送信、クローズのアクションが表示されます。

![Web ページの初期状態](websockets/_static/end.png)
