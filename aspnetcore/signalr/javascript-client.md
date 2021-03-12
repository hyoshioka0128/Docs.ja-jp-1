---
title: ASP.NET Core SignalR JavaScript クライアント
author: bradygaster
description: ASP.NET Core JavaScript クライアントの概要について説明 SignalR します。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc, devx-track-js
ms.date: 04/08/2020
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
uid: signalr/javascript-client
ms.openlocfilehash: 1b5c89c1beaf36912766b3e08b9f16c4d33945f8
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588062"
---
# <a name="aspnet-core-signalr-javascript-client"></a>ASP.NET Core SignalR JavaScript クライアント

::: moniker range=">= aspnetcore-3.0"

作成者: [Rachel Appel](https://twitter.com/rachelappel)

ASP.NET Core SignalR JavaScript クライアントライブラリを使用すると、開発者はサーバー側のハブコードを呼び出すことができます。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/javascript-client/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="install-the-signalr-client-package"></a>クライアントパッケージをインストールする SignalR

SignalRJavaScript クライアントライブラリは[npm](https://www.npmjs.com/)パッケージとして配信されます。 次のセクションでは、さまざまな方法でクライアントライブラリをインストールする方法について説明します。

### <a name="install-with-npm"></a>Npm を使用してインストールする

Visual Studio の場合は、ルートフォルダーで、 **パッケージマネージャーコンソール** から次のコマンドを実行します。 Visual Studio Code には、 **統合ターミナル** から次のコマンドを実行します。

```bash
npm init -y
npm install @microsoft/signalr
```

npm は、 *node_modules \\ @microsoft\signalr\dist\browser* フォルダーにパッケージの内容をインストールします。 *Wwwroot \\ lib* フォルダーの下に *signalr* という名前の新しいフォルダーを作成します。 *signalr.js* ファイルを *wwwroot\lib\signalr* フォルダーにコピーします。

SignalR要素で JavaScript クライアントを参照し `<script>` ます。 次に例を示します。

```html
<script src="~/lib/signalr/signalr.js"></script>
```

### <a name="use-a-content-delivery-network-cdn"></a>Content Delivery Network を使用する (CDN)

Npm の前提条件なしでクライアントライブラリを使用するには、CDN でホストされるクライアントライブラリのコピーを参照します。 次に例を示します。

[!code-html[](javascript-client/samples/3.x/SignalRChat/Pages/Index.cshtml?name=snippet_CDN)]

クライアントライブラリは、次の CDNs で利用できます。

* [cdnjs](https://cdnjs.com/libraries/microsoft-signalr)
* [jsDelivr](https://www.jsdelivr.com/package/npm/@microsoft/signalr)
* [unpkg](https://unpkg.com/@microsoft/signalr@next/dist/browser/signalr.min.js)

### <a name="install-with-libman"></a>LibMan でのインストール

[Libman](xref:client-side/libman/index) を使用すると、CDN でホストされているクライアントライブラリから特定のクライアントライブラリファイルをインストールできます。 たとえば、縮小版の JavaScript ファイルのみをプロジェクトに追加します。 その方法の詳細については、「 [ SignalR クライアントライブラリの追加](xref:tutorials/signalr#add-the-signalr-client-library)」を参照してください。

## <a name="connect-to-a-hub"></a>ハブへの接続

次のコードでは、接続を作成して開始します。 ハブの名前では大文字と小文字が区別されません。

[!code-javascript[](javascript-client/samples/3.x/SignalRChat/wwwroot/chat.js?range=3-6,29-43)]

### <a name="cross-origin-connections"></a>クロスオリジン接続

通常、ブラウザーは、要求されたページと同じドメインからの接続を読み込みます。 ただし、別のドメインへの接続が必要になる場合もあります。

> [!IMPORTANT]
> クライアントコードでは、相対 URL ではなく絶対 URL を使用する必要があります。 `.withUrl("/chathub")` を `.withUrl("https://myappurl/chathub")` に変更します。

悪意のあるサイトが別のサイトから機密データを読み取れないようにするため、既定では、 [クロスオリジン接続](xref:security/cors) は無効になっています。 クロスオリジン要求を許可するには、クラスで有効にし `Startup` ます。

[!code-csharp[](javascript-client/samples/3.x/SignalRChat/Startup.cs?highlight=16-23,40)]

## <a name="call-hub-methods-from-the-client"></a>クライアントからのハブメソッドの呼び出し

JavaScript クライアントは、 [HubConnection](/javascript/api/%40microsoft/signalr/hubconnection)の[invoke](/javascript/api/%40microsoft/signalr/hubconnection#invoke-string--any---)メソッドを使用して、ハブでパブリックメソッドを呼び出します。 `invoke`メソッドは次のものを受け入れます。

* ハブメソッドの名前。
* ハブメソッドで定義されているすべての引数。

次の例では、ハブのメソッド名は `SendMessage` です。 `invoke`ハブメソッドの `user` 引数および引数にマップするために渡される2番目と3番目の引数 `message` 。

[!code-javascript[](javascript-client/samples/3.x/SignalRChat/wwwroot/chat.js?name=snippet_Invoke&highlight=2)]

> [!NOTE]
> クライアントからのハブメソッドの呼び出しは、Azure SignalR サービスを *既定* のモードで使用している場合にのみサポートされます。 詳細については、「 [よく寄せられる質問 (azure Signalr GitHub リポジトリ)](https://github.com/Azure/azure-signalr/blob/dev/docs/faq.md#what-is-the-meaning-of-service-mode-defaultserverlessclassic-how-can-i-choose)」を参照してください。

メソッドは、 `invoke` JavaScript [Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise)を返します。 は、 `Promise` サーバーのメソッドがを返すときに戻り値 (存在する場合) を使用して解決されます。 サーバー上のメソッドがエラーをスローした場合、は `Promise` エラーメッセージと共に拒否されます。 `async` `await` `Promise` `then` `catch` これらのケースを処理するには、およびまたはのメソッドとメソッドを使用します。

JavaScript クライアントは、の [send](/javascript/api/%40microsoft/signalr/hubconnection#send-string--any---) メソッドを使用して、ハブでパブリックメソッドを呼び出すこともでき `HubConnection` ます。 メソッドとは異なり `invoke` 、 `send` メソッドはサーバーからの応答を待機しません。 メソッドは、 `send` JavaScript を返し `Promise` ます。 は、 `Promise` メッセージがサーバーに送信されたときに解決されます。 メッセージの送信中にエラーが発生した場合、は `Promise` エラーメッセージと共に拒否されます。 `async` `await` `Promise` `then` `catch` これらのケースを処理するには、およびまたはのメソッドとメソッドを使用します。

> [!NOTE]
> を使用する `send` と、サーバーがメッセージを受信するまで待機しません。 このため、サーバーからデータまたはエラーを返すことはできません。

## <a name="call-client-methods-from-the-hub"></a>ハブからクライアントメソッドを呼び出す

ハブからメッセージを受信するには、の [on](/javascript/api/%40microsoft/signalr/hubconnection#on-string---args--any-------void-) メソッドを使用してメソッドを定義し `HubConnection` ます。

* JavaScript クライアントメソッドの名前。
* ハブがメソッドに渡す引数。

次の例では、メソッド名は `ReceiveMessage` です。 引数の名前は `user` と `message` です。

[!code-javascript[](javascript-client/samples/3.x/SignalRChat/wwwroot/chat.js?name=snippet_ReceiveMessage)]

の前のコードは、 `connection.on` サーバー側のコードがメソッドを使用して呼び出すときに実行され <xref:Microsoft.AspNetCore.SignalR.ClientProxyExtensions.SendAsync%2A> ます。

[!code-csharp[Call client-side](javascript-client/samples/3.x/SignalRChat/Hubs/ChatHub.cs?name=snippet_SendMessage)]

SignalR とで定義されたメソッド名と引数を照合することによって、どのクライアントメソッドを呼び出すかを決定し `SendAsync` `connection.on` ます。

> [!NOTE]
> ベストプラクティスとして、の後にの [start](/javascript/api/%40aspnet/signalr/hubconnection#start) メソッドを呼び出し `HubConnection` `on` ます。 これにより、メッセージを受信する前にハンドラーが確実に登録されます。

## <a name="error-handling-and-logging"></a>エラー処理とログ記録

と `try` を `catch` `async` and またはのメソッドと共に使用して、 `await` `Promise` `catch` クライアント側のエラーを処理します。 を使用し `console.error` て、ブラウザーのコンソールにエラーを出力します。

[!code-javascript[](javascript-client/samples/3.x/SignalRChat/wwwroot/chat.js?name=snippet_Invoke&highlight=1,3-5)]

接続が確立されたときにログに記録する logger と種類のイベントを渡すことによって、クライアント側のログトレースを設定します。 メッセージは、指定されたログレベル以上でログに記録されます。 使用可能なログレベルは次のとおりです。

* `signalR.LogLevel.Error`: エラーメッセージ。 `Error`メッセージのみをログに記録します。
* `signalR.LogLevel.Warning`: 潜在的なエラーに関する警告メッセージ。 ログ `Warning` と `Error` メッセージ。
* `signalR.LogLevel.Information`: エラーのないステータスメッセージ。 、、およびの各メッセージをログ `Information` に記録 `Warning` `Error` します。
* `signalR.LogLevel.Trace`: トレースメッセージ。 ハブとクライアント間で転送されたデータを含む、すべてをログに記録します。

ログレベルを構成するには、 [HubConnectionBuilder](/javascript/api/%40aspnet/signalr/hubconnectionbuilder)で[configureLogging](/javascript/api/%40aspnet/signalr/hubconnectionbuilder#configurelogging)メソッドを使用します。 メッセージは、ブラウザーコンソールにログ記録されます。

[!code-javascript[](javascript-client/samples/3.x/SignalRChat/wwwroot/chat.js?name=snippet_Connection&highlight=3)]

## <a name="reconnect-clients"></a>クライアントの再接続

### <a name="automatically-reconnect"></a>自動的に再接続する

の JavaScript クライアントは、 SignalR HubConnectionBuilder のメソッドを使用して自動的に再接続するように構成でき `withAutomaticReconnect` ます。 [](/javascript/api/%40aspnet/signalr/hubconnectionbuilder) 既定では、自動的に再接続されません。

```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .withAutomaticReconnect()
    .build();
```

パラメーターを指定しない場合、は、 `withAutomaticReconnect()` 再接続を試行する前に、それぞれ0、2、10、および30秒間待機するようにクライアントを構成します。これにより、4回の試行が失敗すると停止します。

再接続の試行を開始する前に、は状態に遷移し、 `HubConnection` コールバックを起動します。この状態に遷移し、 `HubConnectionState.Reconnecting` `onreconnecting` `Disconnected` `onclose` 自動再接続が構成されていないなどのコールバックをトリガーし `HubConnection` ます。 これにより、接続が失われたことをユーザーに警告し、UI 要素を無効にすることができます。

```javascript
connection.onreconnecting(error => {
    console.assert(connection.state === signalR.HubConnectionState.Reconnecting);

    document.getElementById("messageInput").disabled = true;

    const li = document.createElement("li");
    li.textContent = `Connection lost due to error "${error}". Reconnecting.`;
    document.getElementById("messagesList").appendChild(li);
});
```

最初の4回の試行でクライアントが正常に再接続した場合、は `HubConnection` 状態に戻り、 `Connected` コールバックを起動し `onreconnected` ます。 これにより、接続が再確立されたことをユーザーに通知することができます。

接続はサーバーにまったく新しいものであるため、 `connectionId` コールバックに新しいが提供され `onreconnected` ます。

> [!WARNING]
> が `onreconnected` `connectionId` `HubConnection` [ネゴシエーションをスキップ](xref:signalr/configuration#configure-client-options)するように構成されている場合、コールバックのパラメーターは未定義になります。

```javascript
connection.onreconnected(connectionId => {
    console.assert(connection.state === signalR.HubConnectionState.Connected);

    document.getElementById("messageInput").disabled = false;

    const li = document.createElement("li");
    li.textContent = `Connection reestablished. Connected with connectionId "${connectionId}".`;
    document.getElementById("messagesList").appendChild(li);
});
```

`withAutomaticReconnect()` 最初の開始エラーを再試行するようにを構成しません `HubConnection` 。そのため、開始エラーは手動で処理する必要があります。

```javascript
async function start() {
    try {
        await connection.start();
        console.assert(connection.state === signalR.HubConnectionState.Connected);
        console.log("SignalR Connected.");
    } catch (err) {
        console.assert(connection.state === signalR.HubConnectionState.Disconnected);
        console.log(err);
        setTimeout(() => start(), 5000);
    }
};
```

最初の4回の試行でクライアントが正常に再接続されない場合、は `HubConnection` 状態に遷移し、 `Disconnected` その [閉じる](/javascript/api/%40aspnet/signalr/hubconnection#onclose) コールバックを起動します。 これにより、接続が完全に失われたことをユーザーに通知し、ページを最新の情報に更新することをお勧めします。

```javascript
connection.onclose(error => {
    console.assert(connection.state === signalR.HubConnectionState.Disconnected);

    document.getElementById("messageInput").disabled = true;

    const li = document.createElement("li");
    li.textContent = `Connection closed due to error "${error}". Try refreshing this page to restart the connection.`;
    document.getElementById("messagesList").appendChild(li);
});
```

再接続のタイミングを切断または変更する前に、カスタムの再接続試行回数を構成するために、では、 `withAutomaticReconnect` 各再接続の試行を開始するまでの待機時間 (ミリ秒) を表す数値の配列を受け取ります。

```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .withAutomaticReconnect([0, 0, 10000])
    .build();

    // .withAutomaticReconnect([0, 2000, 10000, 30000]) yields the default behavior
```

前の例では、 `HubConnection` 接続が失われた直後に再接続を開始するようにを構成しています。 これは、既定の構成にも当てはまります。

最初の再接続の試行が失敗した場合、2回目の再接続試行も、既定の構成のように2秒間待機するのではなく、直ちに開始されます。

2回目の再接続の試行が失敗した場合、3回目の再接続は10秒後に開始され、既定の構成のようになります。

このカスタム動作は、既定の構成のように、別の30秒でもう一度再接続を試行するのではなく、3回目の再接続試行の失敗後に停止することで、既定の動作から再び逸脱します。

タイミングと自動再接続試行回数をさらに細かく制御する場合は、 `withAutomaticReconnect` インターフェイスを実装するオブジェクトを受け取り `IRetryPolicy` ます。このインターフェイスには、という名前の1つのメソッドがあり `nextRetryDelayInMilliseconds` ます。

`nextRetryDelayInMilliseconds` 型の1つの引数を受け取り `RetryContext` ます。 には、、、およびの `RetryContext` 3 つのプロパティがあります。 `previousRetryCount` `elapsedMilliseconds` `retryReason` `number` `number` `Error` 最初の再接続を試行する前に、 `previousRetryCount` と `elapsedMilliseconds` の両方がゼロになり、は接続が失われる `retryReason` 原因となったエラーになります。 再試行に失敗するたびに、 `previousRetryCount` が1ずつインクリメントされ `elapsedMilliseconds` ます。これまでの再接続にかかった時間 (ミリ秒単位) を反映して更新され `retryReason` ます。このエラーは、最後の再接続の試行が失敗した原因となったエラーになります。

`nextRetryDelayInMilliseconds` は、次回の再接続が試行される前に待機するミリ秒数を表す数値を返すか、再接続を停止する必要がある場合はを返し `null` `HubConnection` ます。

```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .withAutomaticReconnect({
        nextRetryDelayInMilliseconds: retryContext => {
            if (retryContext.elapsedMilliseconds < 60000) {
                // If we've been reconnecting for less than 60 seconds so far,
                // wait between 0 and 10 seconds before the next reconnect attempt.
                return Math.random() * 10000;
            } else {
                // If we've been reconnecting for more than 60 seconds so far, stop reconnecting.
                return null;
            }
        }
    })
    .build();
```

または、 [手動で再接続](#manually-reconnect)する方法で示すように、手動でクライアントを再接続するコードを記述することもできます。

### <a name="manually-reconnect"></a>手動で再接続する

次のコードは、標準的な手動再接続アプローチを示しています。

1. 接続を開始する関数 (この場合は `start` 関数) が作成されます。
1. `start`接続のイベントハンドラーで関数を呼び出し `onclose` ます。

[!code-javascript[](javascript-client/samples/3.x/SignalRChat/wwwroot/chat.js?range=30-40)]

実際の実装では、指数バックオフを使用するか、指定された回数の再試行を行います。

## <a name="additional-resources"></a>その他のリソース

* [JavaScript API リファレンス](/javascript/api/?view=signalr-js-latest&preserve-view=true )
* [JavaScript のチュートリアル](xref:tutorials/signalr)
* [WebPack と TypeScript のチュートリアル](xref:tutorials/signalr-typescript-webpack)
* [ハブ](xref:signalr/hubs)
* [.NET クライアント](xref:signalr/dotnet-client)
* [Azure に発行する](xref:signalr/publish-to-azure-web-app)
* [クロスオリジン要求 (CORS)](xref:security/cors)
* [Azure SignalR サービスのサーバーレスドキュメント](/azure/azure-signalr/signalr-concept-serverless-development-config)
* [接続エラーのトラブルシューティング](xref:signalr/troubleshoot)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作成者: [Rachel Appel](https://twitter.com/rachelappel)

ASP.NET Core SignalR JavaScript クライアントライブラリを使用すると、開発者はサーバー側のハブコードを呼び出すことができます。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/javascript-client/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="install-the-signalr-client-package"></a>クライアントパッケージをインストールする SignalR

SignalRJavaScript クライアントライブラリは[npm](https://www.npmjs.com/)パッケージとして配信されます。 次のセクションでは、さまざまな方法でクライアントライブラリをインストールする方法について説明します。

### <a name="install-with-npm"></a>Npm を使用してインストールする

Visual Studio を使用している場合は、ルートフォルダーで、 **パッケージマネージャーコンソール** から次のコマンドを実行します。 Visual Studio Code には、 **統合ターミナル** から次のコマンドを実行します。

```bash
npm init -y
npm install @aspnet/signalr
```

npm は、 *node_modules \\ @aspnet\signalr\dist\browser* フォルダーにパッケージの内容をインストールします。 *Wwwroot \\ lib* フォルダーの下に *signalr* という名前の新しいフォルダーを作成します。 *signalr.js* ファイルを *wwwroot\lib\signalr* フォルダーにコピーします。

SignalR要素で JavaScript クライアントを参照し `<script>` ます。 次に例を示します。

```html
<script src="~/lib/signalr/signalr.js"></script>
```

### <a name="use-a-content-delivery-network-cdn"></a>Content Delivery Network を使用する (CDN)

Npm の前提条件なしでクライアントライブラリを使用するには、CDN でホストされるクライアントライブラリのコピーを参照します。 次に例を示します。

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/microsoft-signalr/3.1.3/signalr.min.js"></script>
```

クライアントライブラリは、次の CDNs で利用できます。

* [cdnjs](https://cdnjs.com/libraries/aspnet-signalr)
* [jsDelivr](https://www.jsdelivr.com/package/npm/@aspnet/signalr)
* [unpkg](https://unpkg.com/@aspnet/signalr@next/dist/browser/signalr.min.js)

### <a name="install-with-libman"></a>LibMan でのインストール

[Libman](xref:client-side/libman/index) を使用すると、CDN でホストされているクライアントライブラリから特定のクライアントライブラリファイルをインストールできます。 たとえば、縮小版の JavaScript ファイルのみをプロジェクトに追加します。 その方法の詳細については、「 [ SignalR クライアントライブラリの追加](xref:tutorials/signalr#add-the-signalr-client-library)」を参照してください。

## <a name="connect-to-a-hub"></a>ハブへの接続

次のコードでは、接続を作成して開始します。 ハブの名前は大文字と小文字が区別されません。

[!code-javascript[Call hub methods](javascript-client/samples/2.x/SignalRChat/wwwroot/js/chat.js?range=9-13,28-51)]

### <a name="cross-origin-connections"></a>クロスオリジン接続

通常、ブラウザーは、要求されたページと同じドメインからの接続を読み込みます。 ただし、別のドメインへの接続が必要になる場合もあります。

悪意のあるサイトが別のサイトから機密データを読み取れないようにするため、既定では、 [クロスオリジン接続](xref:security/cors) は無効になっています。 クロスオリジン要求を許可するには、クラスで有効にし `Startup` ます。

[!code-csharp[Cross-origin connections](javascript-client/samples/2.x/SignalRChat/Startup.cs?highlight=29-35,56)]

## <a name="call-hub-methods-from-client"></a>クライアントからのハブメソッドの呼び出し

JavaScript クライアントは、 [HubConnection](/javascript/api/%40aspnet/signalr/hubconnection)の[invoke](/javascript/api/%40aspnet/signalr/hubconnection#invoke)メソッドを使用して、ハブでパブリックメソッドを呼び出します。 メソッドは、 `invoke` 次の2つの引数を受け取ります。

* ハブメソッドの名前。 次の例では、ハブのメソッド名は `SendMessage` です。
* ハブメソッドで定義されているすべての引数。 次の例では、引数名は `message` です。 このコード例では、Internet Explorer を除くすべての主要なブラウザーの現在のバージョンでサポートされている、矢印関数の構文を使用します。

  [!code-javascript[Call hub methods](javascript-client/samples/2.x/SignalRChat/wwwroot/js/chat.js?range=24)]

> [!NOTE]
> クライアントからのハブメソッドの呼び出しは、Azure SignalR サービスを *既定* のモードで使用している場合にのみサポートされます。 詳細については、「 [よく寄せられる質問 (azure Signalr GitHub リポジトリ)](https://github.com/Azure/azure-signalr/blob/dev/docs/faq.md#what-is-the-meaning-of-service-mode-defaultserverlessclassic-how-can-i-choose)」を参照してください。

メソッドは、 `invoke` JavaScript [Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise)を返します。 は、 `Promise` サーバーのメソッドがを返すときに戻り値 (存在する場合) を使用して解決されます。 サーバー上のメソッドがエラーをスローした場合、は `Promise` エラーメッセージと共に拒否されます。 これらの `then` `catch` `Promise` ケース (または構文) を処理するには、自身でメソッドとメソッドを使用し `await` ます。

メソッドは、 `send` JavaScript を返し `Promise` ます。 は、 `Promise` メッセージがサーバーに送信されたときに解決されます。 メッセージの送信中にエラーが発生した場合、は `Promise` エラーメッセージと共に拒否されます。 これらの `then` `catch` `Promise` ケース (または構文) を処理するには、自身でメソッドとメソッドを使用し `await` ます。

> [!NOTE]
> を使用する `send` と、サーバーがメッセージを受信するまで待機しません。 このため、サーバーからデータまたはエラーを返すことはできません。

## <a name="call-client-methods-from-hub"></a>ハブからクライアントメソッドを呼び出す

ハブからメッセージを受信するには、の [on](/javascript/api/%40aspnet/signalr/hubconnection#on) メソッドを使用してメソッドを定義し `HubConnection` ます。

* JavaScript クライアントメソッドの名前。 次の例では、メソッド名は `ReceiveMessage` です。
* ハブがメソッドに渡す引数。 次の例では、引数の値は `message` です。

[!code-javascript[Receive calls from hub](javascript-client/samples/2.x/SignalRChat/wwwroot/js/chat.js?range=14-19)]

の前のコードは、 `connection.on` サーバー側のコードがメソッドを使用して呼び出したときに実行され <xref:Microsoft.AspNetCore.SignalR.ClientProxyExtensions.SendAsync%2A> ます。

[!code-csharp[Call client-side](javascript-client/samples/2.x/SignalRChat/hubs/chathub.cs?range=8-11)]

SignalR とで定義されたメソッド名と引数を照合することによって、どのクライアントメソッドを呼び出すかを決定し `SendAsync` `connection.on` ます。

> [!NOTE]
> ベストプラクティスとして、の後にの [start](/javascript/api/%40aspnet/signalr/hubconnection#start) メソッドを呼び出し `HubConnection` `on` ます。 これにより、メッセージを受信する前にハンドラーが確実に登録されます。

## <a name="error-handling-and-logging"></a>エラー処理とログ記録

メソッドを `catch` メソッドの最後にチェーンして、 `start` クライアント側のエラーを処理します。 `console.error`ブラウザーのコンソールにエラーを出力するには、を使用します。

[!code-javascript[Error handling](javascript-client/samples/2.x/SignalRChat/wwwroot/js/chat.js?range=50)]

接続が確立されたときにログに記録する logger と種類のイベントを渡すことによって、クライアント側のログトレースを設定します。 メッセージは、指定されたログレベル以上でログに記録されます。 使用可能なログレベルは次のとおりです。

* `signalR.LogLevel.Error`: エラーメッセージ。 `Error`メッセージのみをログに記録します。
* `signalR.LogLevel.Warning`: 潜在的なエラーに関する警告メッセージ。 ログ `Warning` と `Error` メッセージ。
* `signalR.LogLevel.Information`: エラーのないステータスメッセージ。 、、およびの各メッセージをログ `Information` に記録 `Warning` `Error` します。
* `signalR.LogLevel.Trace`: トレースメッセージ。 ハブとクライアント間で転送されたデータを含む、すべてをログに記録します。

ログレベルを構成するには、 [HubConnectionBuilder](/javascript/api/%40aspnet/signalr/hubconnectionbuilder)で[configureLogging](/javascript/api/%40aspnet/signalr/hubconnectionbuilder#configurelogging)メソッドを使用します。 メッセージは、ブラウザーコンソールに記録されます。

[!code-javascript[Logging levels](javascript-client/samples/2.x/SignalRChat/wwwroot/js/chat.js?range=9-12)]

## <a name="reconnect-clients"></a>クライアントの再接続

### <a name="manually-reconnect"></a>手動で再接続する

> [!WARNING]
> 3.0 より前では、の JavaScript クライアントは SignalR 自動的に再接続しません。 クライアントを手動で再接続するコードを記述する必要があります。

次のコードは、標準的な手動再接続アプローチを示しています。

1. 接続を開始する関数 (この場合は `start` 関数) が作成されます。
1. `start`接続のイベントハンドラーで関数を呼び出し `onclose` ます。

[!code-javascript[Reconnect the JavaScript client](javascript-client/samples/2.x/SignalRChat/wwwroot/js/chat.js?range=28-40)]

実際の実装では、指数バックオフを使用するか、指定された回数の再試行を行います。

## <a name="additional-resources"></a>その他のリソース

* [JavaScript API リファレンス](/javascript/api/?view=signalr-js-latest)
* [JavaScript のチュートリアル](xref:tutorials/signalr)
* [WebPack と TypeScript のチュートリアル](xref:tutorials/signalr-typescript-webpack)
* [ハブ](xref:signalr/hubs)
* [.NET クライアント](xref:signalr/dotnet-client)
* [Azure に発行する](xref:signalr/publish-to-azure-web-app)
* [クロスオリジン要求 (CORS)](xref:security/cors)
* [Azure SignalR サービスのサーバーレスドキュメント](/azure/azure-signalr/signalr-concept-serverless-development-config)

::: moniker-end
