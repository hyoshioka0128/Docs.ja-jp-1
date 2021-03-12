---
title: ASP.NET Core でハブを使用する SignalR
author: bradygaster
description: ASP.NET Core でハブを使用する方法について説明 SignalR します。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 01/16/2020
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
uid: signalr/hubs
ms.openlocfilehash: 872b88cc3c87137365de8c50a37bf5dd5fd9fe10
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587867"
---
# <a name="use-hubs-in-signalr-for-aspnet-core"></a>のハブを SignalR ASP.NET Core に使用する

[Rachel appel](https://twitter.com/rachelappel)および[加山 Griffin](https://twitter.com/1kevgriff)

[サンプル コードを表示またはダウンロードする](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/hubs/sample/ ) ([ダウンロード方法](xref:index#how-to-download-a-sample))

## <a name="what-is-a-signalr-hub"></a>ハブとは SignalR

SignalRハブ API を使用すると、接続されているクライアント上のメソッドをサーバーから呼び出すことができます。 サーバーコードでは、クライアントによって呼び出されるメソッドを定義します。 クライアントコードでは、サーバーから呼び出されるメソッドを定義します。 SignalR は、クライアントとサーバー間のリアルタイム通信とサーバー間の通信を可能にする、バックグラウンドの背後にあるすべての処理を行います。

## <a name="configure-signalr-hubs"></a>ハブの構成 SignalR

ミドルウェアには SignalR 、を呼び出すことによって構成されるいくつかのサービスが必要です `services.AddSignalR` 。

[!code-csharp[Configure service](hubs/sample/startup.cs?range=38)]

::: moniker range=">= aspnetcore-3.0"

SignalRASP.NET Core アプリに機能を追加する場合、 SignalR `endpoint.MapHub` `Startup.Configure` メソッドのコールバックでを呼び出すことによってルートをセットアップし `app.UseEndpoints` ます。

```csharp
app.UseRouting();
app.UseEndpoints(endpoints =>
{
    endpoints.MapHub<ChatHub>("/chathub");
});
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

ASP.NET Core アプリに機能を追加するときに SignalR 、 SignalR メソッドでを呼び出すことによってルートをセットアップし `app.UseSignalR` `Startup.Configure` ます。

[!code-csharp[Configure routes to hubs](hubs/sample/startup.cs?range=57-60)]

::: moniker-end

## <a name="create-and-use-hubs"></a>ハブの作成と使用

から継承するクラスを宣言してハブを作成 `Hub` し、パブリックメソッドを追加します。 クライアントは、として定義されているメソッドを呼び出すことができ `public` ます。

```csharp
public class ChatHub : Hub
{
    public Task SendMessage(string user, string message)
    {
        return Clients.All.SendAsync("ReceiveMessage", user, message);
    }
}
```

C# のメソッドの場合と同様に、戻り値の型とパラメーター (複合型や配列を含む) を指定できます。 SignalR パラメーターと戻り値の複合オブジェクトおよび配列のシリアル化と逆シリアル化を処理します。

> [!NOTE]
> ハブは一時的なものです。
>
> * ハブクラスのプロパティに状態を格納しないでください。 すべてのハブメソッド呼び出しは、新しいハブインスタンスで実行されます。
> * `await`ハブをキープアライブに依存する非同期メソッドを呼び出すときに使用します。 たとえば、を `Clients.All.SendAsync(...)` 指定せずに呼び出され、 `await` ハブメソッドが終了する前に完了した場合、などのメソッドは失敗する可能性が `SendAsync` あります。

## <a name="the-context-object"></a>コンテキストオブジェクト

`Hub`クラスには、 `Context` 接続に関する情報を含む次のプロパティを含むプロパティがあります。

| プロパティ | 説明 |
| ------ | ----------- |
| `ConnectionId` | によって割り当てられる、接続の一意の ID を取得し SignalR ます。 接続ごとに1つの接続 ID があります。|
| `UserIdentifier` | [ユーザー識別子](xref:signalr/groups)を取得します。 既定では、は、 SignalR `ClaimTypes.NameIdentifier` `ClaimsPrincipal` 接続に関連付けられているのをユーザー識別子として使用します。 |
| `User` | 現在の `ClaimsPrincipal` ユーザーに関連付けられているを取得します。 |
| `Items` | この接続のスコープ内でデータを共有するために使用できるキー/値のコレクションを取得します。 このコレクションにデータを格納することができ、さまざまなハブメソッド呼び出し間の接続が維持されます。 |
| `Features` | 接続で使用できる機能のコレクションを取得します。 現時点では、ほとんどのシナリオでこのコレクションは必要ないため、詳細には記載されていません。 |
| `ConnectionAborted` | `CancellationToken`接続が中止されたときに通知するを取得します。 |

`Hub.Context` には、次のメソッドも含まれています。

| メソッド | 説明 |
| ------ | ----------- |
| `GetHttpContext` | `HttpContext`接続のを返し `null` ます。接続が HTTP 要求に関連付けられていない場合はを返します。 HTTP 接続の場合は、このメソッドを使用して、HTTP ヘッダーやクエリ文字列などの情報を取得できます。 |
| `Abort` | 接続を中止します。 |

## <a name="the-clients-object"></a>クライアントオブジェクト

`Hub`クラスには、 `Clients` サーバーとクライアント間の通信に関する次のプロパティを含むプロパティがあります。

| プロパティ | 説明 |
| ------ | ----------- |
| `All` | 接続されているすべてのクライアントでメソッドを呼び出します |
| `Caller` | ハブメソッドを呼び出したクライアントでメソッドを呼び出します。 |
| `Others` | メソッドを呼び出したクライアントを除く、接続されているすべてのクライアントでメソッドを呼び出します。 |

`Hub.Clients` には、次のメソッドも含まれています。

| メソッド | 説明 |
| ------ | ----------- |
| `AllExcept` | 指定された接続を除く、接続されているすべてのクライアントでメソッドを呼び出します |
| `Client` | 特定の接続されたクライアントでメソッドを呼び出します |
| `Clients` | 特定の接続されたクライアントでメソッドを呼び出します |
| `Group` | 指定されたグループ内のすべての接続でメソッドを呼び出します  |
| `GroupExcept` | 指定された接続を除く、指定されたグループ内のすべての接続でメソッドを呼び出します。 |
| `Groups` | 複数の接続グループに対してメソッドを呼び出します。  |
| `OthersInGroup` | ハブメソッドを呼び出したクライアントを除く、接続のグループでメソッドを呼び出します。  |
| `User` | 特定のユーザーに関連付けられているすべての接続でメソッドを呼び出します |
| `Users` | 指定されたユーザーに関連付けられているすべての接続でメソッドを呼び出します |

前の表の各プロパティまたはメソッドは、メソッドを使用してオブジェクトを返し `SendAsync` ます。 メソッドを使用すると、 `SendAsync` クライアントメソッドの名前とパラメーターを指定して呼び出すことができます。

## <a name="send-messages-to-clients"></a>クライアントへのメッセージの送信

特定のクライアントに対する呼び出しを行うには、オブジェクトのプロパティを使用し `Clients` ます。 次の例には、3つのハブメソッドがあります。

* `SendMessage` を使用して、接続されているすべてのクライアントにメッセージを送信し `Clients.All` ます。
* `SendMessageToCaller` を使用して、メッセージを呼び出し元に返信し `Clients.Caller` ます。
* `SendMessageToGroups` グループ内のすべてのクライアントにメッセージを送信し `SignalR Users` ます。

[!code-csharp[Send messages](hubs/sample/hubs/chathub.cs?name=HubMethods)]

## <a name="strongly-typed-hubs"></a>厳密に型指定されたハブ

を使用する場合の欠点 `SendAsync` は、呼び出されるクライアントメソッドを指定するマジック文字列に依存することです。 これにより、メソッド名のスペルが間違っている場合、またはクライアントに存在しない場合、コードは実行時エラーになります。

を使用する代わりに、を使用してを `SendAsync` 厳密に型にする方法も `Hub` <xref:Microsoft.AspNetCore.SignalR.Hub%601> あります。 次の例では、 `ChatHub` クライアントメソッドがというインターフェイスに抽出されてい `IChatClient` ます。

[!code-csharp[Interface for IChatClient](hubs/sample/hubs/ichatclient.cs?name=snippet_IChatClient)]

このインターフェイスは、前の例をリファクターするために使用でき `ChatHub` ます。

[!code-csharp[Strongly typed ChatHub](hubs/sample/hubs/StronglyTypedChatHub.cs?range=8-18,36)]

を使用すると `Hub<IChatClient>` 、クライアントメソッドのコンパイル時チェックが有効になります。 これにより、 `Hub<T>` は、インターフェイスで定義されたメソッドへのアクセスのみを提供できるため、マジック文字列を使用した場合に発生する問題を回避できます。

厳密に型指定されたを使用 `Hub<T>` すると、を使用することができなくなり `SendAsync` ます。 インターフェイスで定義されているメソッドは、引き続き非同期として定義できます。 実際、これらの各メソッドはを返す必要があり `Task` ます。 これはインターフェイスであるため、キーワードを使用しないで `async` ください。 次に例を示します。

```csharp
public interface IClient
{
    Task ClientMethod();
}
```

> [!NOTE]
> この `Async` サフィックスは、メソッド名から削除されません。 クライアントメソッドがで定義されていない場合は `.on('MyMethodAsync')` 、を `MyMethodAsync` 名前として使用しないでください。

## <a name="change-the-name-of-a-hub-method"></a>ハブメソッドの名前を変更する

既定では、サーバーハブのメソッド名は .NET メソッドの名前です。 ただし、 [HubMethodName](xref:Microsoft.AspNetCore.SignalR.HubMethodNameAttribute) 属性を使用してこの既定値を変更し、メソッドの名前を手動で指定することができます。 クライアントは、メソッドを呼び出すときに、.NET メソッド名の代わりにこの名前を使用する必要があります。

[!code-csharp[HubMethodName attribute](hubs/sample/hubs/chathub.cs?name=HubMethodName&highlight=1)]

## <a name="handle-events-for-a-connection"></a>接続のイベントを処理する

SignalRHUB API は、 `OnConnectedAsync` 接続の `OnDisconnectedAsync` 管理と追跡のためのおよび仮想メソッドを提供します。 `OnConnectedAsync`仮想メソッドを上書きして、クライアントがハブに接続したときにアクションを実行します (グループへの追加など)。

[!code-csharp[Handle connection](hubs/sample/hubs/chathub.cs?name=OnConnectedAsync)]

`OnDisconnectedAsync`クライアントの接続が切断されたときにアクションを実行するには、仮想メソッドをオーバーライドします。 クライアントが意図的に (たとえば、を呼び出すことによって) 切断された場合、パラメーターはになり `connection.stop()` `exception` `null` ます。 ただし、エラー (ネットワークエラーなど) が原因でクライアントが切断された場合、パラメーターにはエラーを `exception` 説明する例外が含まれます。

[!code-csharp[Handle disconnection](hubs/sample/hubs/chathub.cs?name=OnDisconnectedAsync)]

[!INCLUDE[](~/includes/connectionid-signalr.md)]

## <a name="handle-errors"></a>エラーの処理

ハブメソッドでスローされた例外は、メソッドを呼び出したクライアントに送信されます。 JavaScript クライアントでは、 `invoke` メソッドは javascript の [約束](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Using_promises)を返します。 クライアントは、を使用して promise にアタッチされたハンドラーでエラーを受信すると `catch` 、それを呼び出し、JavaScript オブジェクトとして渡され `Error` ます。

[!code-javascript[Error](hubs/sample/wwwroot/js/chat.js?range=23)]

ハブが例外をスローした場合、接続は閉じられません。 既定では、は、 SignalR 一般的なエラーメッセージをクライアントに返します。 次に例を示します。

```
Microsoft.AspNetCore.SignalR.HubException: An unexpected error occurred invoking 'MethodName' on the server.
```

予期しない例外には、データベース接続が失敗したときにトリガーされる例外のデータベースサーバーの名前など、重要な情報が含まれていることがよくあります。 SignalR では、これらの詳細なエラーメッセージは既定でセキュリティ対策として公開されません。 例外の詳細が抑制される理由の詳細については、「 [セキュリティに関する考慮事項](xref:signalr/security#exceptions) 」を参照してください。

クライアントに伝達する必要のある *例外的な条件* がある場合は、クラスを使用でき `HubException` ます。 ハブメソッドからをスローすると `HubException` 、 SignalR **は** 、メッセージ全体を未変更のままクライアントに送信します。

[!code-csharp[ThrowHubException](hubs/sample/hubs/chathub.cs?name=ThrowHubException&highlight=3)]

> [!NOTE]
> SignalR では、 `Message` 例外のプロパティのみがクライアントに送信されます。 例外のスタックトレースおよびその他のプロパティは、クライアントでは使用できません。

## <a name="related-resources"></a>関連リソース

* [ASP.NET Core の概要 SignalR](xref:signalr/introduction)
* [JavaScript クライアント](xref:signalr/javascript-client)
* [Azure に発行する](xref:signalr/publish-to-azure-web-app)
