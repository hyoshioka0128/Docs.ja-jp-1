---
title: ASP.NET Core SignalR .Net クライアント
author: bradygaster
description: ASP.NET Core .Net クライアントに関する情報 SignalR
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 01/14/2020
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
uid: signalr/dotnet-client
ms.openlocfilehash: db2bfa6d3aa440615c5a9c17ae843dbe22755c97
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589245"
---
# <a name="aspnet-core-signalr-net-client"></a>ASP.NET Core SignalR .Net クライアント

ASP.NET Core SignalR .net クライアントライブラリを使用すると、 SignalR .net アプリからハブと通信できます。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/dotnet-client/sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

この記事のコードサンプルは、ASP.NET Core .Net クライアントを使用する WPF アプリです SignalR 。

## <a name="install-the-signalr-net-client-package"></a>SignalR.Net クライアントパッケージをインストールする

[AspNetCore. SignalR .クライアント](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client)パッケージは、.net クライアントがハブに接続するために必要です SignalR 。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

クライアントライブラリをインストールするには、 **パッケージマネージャーコンソール** ウィンドウで次のコマンドを実行します。

```powershell
Install-Package Microsoft.AspNetCore.SignalR.Client
```

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

クライアントライブラリをインストールするには、コマンドシェルで次のコマンドを実行します。

```dotnetcli
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

---

## <a name="connect-to-a-hub"></a>ハブへの接続

接続を確立するには、を作成し、を `HubConnectionBuilder` 呼び出し `Build` ます。 接続の構築中に、ハブの URL、プロトコル、トランスポートの種類、ログレベル、ヘッダー、およびその他のオプションを構成できます。 任意のメソッドをに挿入して、必要なオプションを構成 `HubConnectionBuilder` `Build` します。 との接続を開始 `StartAsync` します。

[!code-csharp[Build hub connection](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_MainWindowClass&highlight=15-17,39)]

## <a name="handle-lost-connection"></a>失われた接続の処理

::: moniker range=">= aspnetcore-3.0"

### <a name="automatically-reconnect"></a>自動的に再接続する

は、の <xref:Microsoft.AspNetCore.SignalR.Client.HubConnection> メソッドを使用して自動的に再接続するように構成でき `WithAutomaticReconnect` <xref:Microsoft.AspNetCore.SignalR.Client.HubConnectionBuilder> ます。 既定では、自動的に再接続されません。

```csharp
HubConnection connection= new HubConnectionBuilder()
    .WithUrl(new Uri("http://127.0.0.1:5000/chathub"))
    .WithAutomaticReconnect()
    .Build();
```

パラメーターを指定しない場合、は、 `WithAutomaticReconnect()` 再接続を試行する前に、それぞれ0、2、10、および30秒間待機するようにクライアントを構成します。これにより、4回の試行が失敗すると停止します。

再接続の試行を開始する前に、は `HubConnection` 状態に遷移し、 `HubConnectionState.Reconnecting` イベントを発生させ `Reconnecting` ます。  これにより、接続が失われたことをユーザーに警告し、UI 要素を無効にすることができます。 非対話型アプリでは、メッセージのキューまたは削除を開始できます。

```csharp
connection.Reconnecting += error =>
{
    Debug.Assert(connection.State == HubConnectionState.Reconnecting);

    // Notify users the connection was lost and the client is reconnecting.
    // Start queuing or dropping messages.

    return Task.CompletedTask;
};
```

最初の4回の試行でクライアントが正常に再接続した場合、は `HubConnection` 状態に戻り、 `Connected` イベントを発生させ `Reconnected` ます。 これにより、接続が再確立されたことをユーザーに通知し、キューに置かれたすべてのメッセージをデキューすることができます。

接続はサーバーにまったく新しいものであるため、 `ConnectionId` イベントハンドラーに新しいが提供され `Reconnected` ます。

> [!WARNING]
> が `Reconnected` `connectionId` `HubConnection` [ネゴシエーションをスキップ](xref:signalr/configuration#configure-client-options)するように構成されている場合、イベントハンドラーのパラメーターは null になります。

```csharp
connection.Reconnected += connectionId =>
{
    Debug.Assert(connection.State == HubConnectionState.Connected);

    // Notify users the connection was reestablished.
    // Start dequeuing messages queued while reconnecting if any.

    return Task.CompletedTask;
};
```

`WithAutomaticReconnect()` 最初の開始エラーを再試行するようにを構成しません `HubConnection` 。そのため、開始エラーは手動で処理する必要があります。

```csharp
public static async Task<bool> ConnectWithRetryAsync(HubConnection connection, CancellationToken token)
{
    // Keep trying to until we can start or the token is canceled.
    while (true)
    {
        try
        {
            await connection.StartAsync(token);
            Debug.Assert(connection.State == HubConnectionState.Connected);
            return true;
        }
        catch when (token.IsCancellationRequested)
        {
            return false;
        }
        catch
        {
            // Failed to connect, trying again in 5000 ms.
            Debug.Assert(connection.State == HubConnectionState.Disconnected);
            await Task.Delay(5000);
        }
    }
}
```

最初の4回の試行でクライアントが正常に再接続されない場合、は `HubConnection` 状態に遷移し、 `Disconnected` イベントを発生させ <xref:Microsoft.AspNetCore.SignalR.Client.HubConnection.Closed> ます。 これにより、接続を手動で再起動したり、接続が完全に失われたことをユーザーに通知したりすることができます。

```csharp
connection.Closed += error =>
{
    Debug.Assert(connection.State == HubConnectionState.Disconnected);

    // Notify users the connection has been closed or manually try to restart the connection.

    return Task.CompletedTask;
};
```

再接続のタイミングを切断または変更する前に、カスタムの再接続試行回数を構成するために、では、 `WithAutomaticReconnect` 各再接続の試行を開始するまでの待機時間 (ミリ秒) を表す数値の配列を受け取ります。

```csharp
HubConnection connection= new HubConnectionBuilder()
    .WithUrl(new Uri("http://127.0.0.1:5000/chathub"))
    .WithAutomaticReconnect(new[] { TimeSpan.Zero, TimeSpan.Zero, TimeSpan.FromSeconds(10) })
    .Build();

    // .WithAutomaticReconnect(new[] { TimeSpan.Zero, TimeSpan.FromSeconds(2), TimeSpan.FromSeconds(10), TimeSpan.FromSeconds(30) }) yields the default behavior.
```

前の例では、 `HubConnection` 接続が失われた直後に再接続を開始するようにを構成しています。 これは、既定の構成にも当てはまります。

最初の再接続の試行が失敗した場合、2回目の再接続試行も、既定の構成のように2秒間待機するのではなく、直ちに開始されます。

2回目の再接続の試行が失敗した場合、3回目の再接続は10秒後に開始され、既定の構成のようになります。

その後、3回目の再接続の試行が失敗した後に停止することで、カスタム動作が既定の動作から再び逸脱します。 既定の構成では、もう1回は30秒後に再接続が試行されます。

タイミングと自動再接続試行回数をさらに細かく制御する場合は、 `WithAutomaticReconnect` インターフェイスを実装するオブジェクトを受け取り `IRetryPolicy` ます。このインターフェイスには、という名前の1つのメソッドがあり `NextRetryDelay` ます。

`NextRetryDelay` 型の1つの引数を受け取り `RetryContext` ます。 には、、、およびの `RetryContext` 3 つのプロパティがあり `PreviousRetryCount` `ElapsedTime` `RetryReason` `long` `TimeSpan` `Exception` ます。 最初の再接続を試行する前に、 `PreviousRetryCount` と `ElapsedTime` の両方がゼロになり、は接続が失われる `RetryReason` 原因となった例外になります。 再試行が再試行されるたびに、が1ずつインクリメントされ、これ `PreviousRetryCount` `ElapsedTime` までに再接続に費やされた時間が反映されます。また、は、 `RetryReason` 最後の再接続の試行が失敗した原因となった例外になります。

`NextRetryDelay` 次の再接続が試行される前に待機する時間を表す TimeSpan を返すか、またはの再接続を停止する必要があり `null` `HubConnection` ます。

```csharp
public class RandomRetryPolicy : IRetryPolicy
{
    private readonly Random _random = new Random();

    public TimeSpan? NextRetryDelay(RetryContext retryContext)
    {
        // If we've been reconnecting for less than 60 seconds so far,
        // wait between 0 and 10 seconds before the next reconnect attempt.
        if (retryContext.ElapsedTime < TimeSpan.FromSeconds(60))
        {
            return TimeSpan.FromSeconds(_random.NextDouble() * 10);
        }
        else
        {
            // If we've been reconnecting for more than 60 seconds so far, stop reconnecting.
            return null;
        }
    }
}
```

```csharp
HubConnection connection = new HubConnectionBuilder()
    .WithUrl(new Uri("http://127.0.0.1:5000/chathub"))
    .WithAutomaticReconnect(new RandomRetryPolicy())
    .Build();
```

または、 [手動で再接続](#manually-reconnect)する方法で示すように、手動でクライアントを再接続するコードを記述することもできます。

::: moniker-end

### <a name="manually-reconnect"></a>手動で再接続する

::: moniker range="< aspnetcore-3.0"

> [!WARNING]
> 3.0 より前では、の .NET クライアントは SignalR 自動的に再接続しません。 クライアントを手動で再接続するコードを記述する必要があります。

::: moniker-end

<xref:Microsoft.AspNetCore.SignalR.Client.HubConnection.Closed>失われた接続に応答するには、イベントを使用します。 たとえば、再接続を自動化することができます。

イベントには、を `Closed` 返すデリゲートが必要です `Task` 。これにより、を使用せずに非同期コードを実行でき `async void` ます。 同期的に実行されるイベントハンドラーでデリゲートシグネチャを満たすには `Closed` 、次を返し `Task.CompletedTask` ます。

```csharp
connection.Closed += (error) => {
    // Do your close logic.
    return Task.CompletedTask;
};
```

非同期サポートの主な理由は、接続を再起動できるようにするためです。 接続の開始は、非同期アクションです。

接続を `Closed` 再起動するハンドラーで、次の例に示すように、サーバーの過負荷を防ぐために、ランダムな遅延を待機することを検討してください。

[!code-csharp[Use Closed event handler to automate reconnection](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_ClosedRestart)]

## <a name="call-hub-methods-from-client"></a>クライアントからのハブメソッドの呼び出し

`InvokeAsync` ハブでメソッドを呼び出します。 ハブメソッドの名前と、ハブメソッドで定義されているすべての引数をに渡し `InvokeAsync` ます。 SignalR は非同期であるため `async` 、 `await` 呼び出しを行うときにとを使用します。

[!code-csharp[InvokeAsync method](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_InvokeAsync)]

この `InvokeAsync` メソッドは、 `Task` サーバーメソッドから制御が戻ったときに完了するを返します。 戻り値 (存在する場合) は、の結果として提供され `Task` ます。 サーバー上のメソッドによってスローされた例外が発生すると、エラーが発生 `Task` します。 構文を使用して `await` 、サーバーメソッドが完了するのを待機し、構文を使用して `try...catch` エラーを処理します。

メソッドは、 `SendAsync` `Task` メッセージがサーバーに送信されたときに完了するを返します。 `Task`サーバーメソッドが完了するまで待機しないため、戻り値は提供されません。 メッセージの送信中にクライアントでスローされた例外は、エラーを生成し `Task` ます。 および構文を使用し `await` て、 `try...catch` 送信エラーを処理します。

> [!NOTE]
> クライアントからのハブメソッドの呼び出しは、Azure SignalR サービスを *既定* のモードで使用している場合にのみサポートされます。 詳細については、「 [よく寄せられる質問 (azure Signalr GitHub リポジトリ)](https://github.com/Azure/azure-signalr/blob/dev/docs/faq.md#what-is-the-meaning-of-service-mode-defaultserverlessclassic-how-can-i-choose)」を参照してください。

## <a name="call-client-methods-from-hub"></a>ハブからクライアントメソッドを呼び出す

ハブが `connection.On` ビルド後、接続を開始する前にを使用して呼び出すメソッドを定義します。

[!code-csharp[Define client methods](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_ConnectionOn)]

の前のコードは、 `connection.On` サーバー側のコードがメソッドを使用して呼び出したときに実行され `SendAsync` ます。

[!code-csharp[Call client method](dotnet-client/sample/signalrchat/hubs/chathub.cs?name=snippet_SendMessage)]

## <a name="error-handling-and-logging"></a>エラー処理とログ記録

Try-catch ステートメントを使用してエラーを処理します。 オブジェクトを調べて、 `Exception` エラーが発生した後に実行する適切なアクションを決定します。

[!code-csharp[Logging](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_ErrorHandling)]

## <a name="additional-resources"></a>その他のリソース

* [ハブ](xref:signalr/hubs)
* [JavaScript クライアント](xref:signalr/javascript-client)
* [Azure に発行する](xref:signalr/publish-to-azure-web-app)
* [Azure SignalR サービスのサーバーレスドキュメント](/azure/azure-signalr/signalr-concept-serverless-development-config)
