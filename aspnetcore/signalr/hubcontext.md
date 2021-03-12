---
title: SignalR HubContext
author: bradygaster
description: ASP.NET Core HubContext サービスを使用し SignalR て、ハブの外部からクライアントに通知を送信する方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 11/12/2019
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
- IHubContext
uid: signalr/hubcontext
ms.openlocfilehash: 2b2939a7692a195c6dc1b8421433a723310b4bd6
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589206"
---
# <a name="send-messages-from-outside-a-hub"></a>ハブの外部からメッセージを送信する

作成者: [Mikael Mengistu](https://twitter.com/MikaelM_12)

SignalRハブは、サーバーに接続されたクライアントにメッセージを送信するための中核的な抽象化です SignalR 。 また、サービスを使用して、アプリ内の他の場所からメッセージを送信することもでき `IHubContext` ます。 この記事では、にアクセスして、 SignalR `IHubContext` ハブの外部からクライアントに通知を送信する方法について説明します。

[サンプル コードを表示またはダウンロードする](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/hubcontext/sample/) ([ダウンロード方法](xref:index#how-to-download-a-sample))

## <a name="get-an-instance-of-ihubcontext"></a>のインスタンスを取得する IHubContext

ASP.NET Core では SignalR 、依存関係の挿入によってのインスタンスにアクセスでき `IHubContext` ます。 のインスタンスは、 `IHubContext` コントローラー、ミドルウェア、またはその他の DI サービスに挿入できます。 インスタンスを使用して、クライアントにメッセージを送信します。

> [!NOTE]
> これ SignalR は、GlobalHost を使用してへのアクセスを提供する ASP.NET 4.x とは異なり `IHubContext` ます。 ASP.NET Core には、このグローバルシングルトンの必要性を排除する依存関係挿入フレームワークがあります。

### <a name="inject-an-instance-of-ihubcontext-in-a-controller"></a>コントローラーにのインスタンスを挿入する IHubContext

のインスタンス `IHubContext` を、コンストラクターに追加することによって、コントローラーに挿入できます。

[!code-csharp[IHubContext](hubcontext/sample/Controllers/HomeController.cs?range=12-19,57)]

では、のインスタンスにアクセスできるようになったので、ハブ `IHubContext` 自体と同じようにハブメソッドを呼び出すことができます。

[!code-csharp[IHubContext](hubcontext/sample/Controllers/HomeController.cs?range=21-25)]

### <a name="get-an-instance-of-ihubcontext-in-middleware"></a>ミドルウェア内ののインスタンスを取得する IHubContext

次のよう `IHubContext` にミドルウェアパイプライン内のにアクセスします。

```csharp
app.Use(async (context, next) =>
{
    var hubContext = context.RequestServices
                            .GetRequiredService<IHubContext<ChatHub>>();
    //...
    
    if (next != null)
    {
        await next.Invoke();
    }
});
```

> [!NOTE]
> ハブメソッドがクラスの外部から呼び出された場合 `Hub` 、呼び出しに関連付けられている呼び出し元はありません。 そのため、、、およびの各プロパティにアクセスすることはできません `ConnectionId` `Caller` `Others` 。

### <a name="get-an-instance-of-ihubcontext-from-ihost"></a>IHost からインスタンスを取得する IHubContext

`IHubContext`Web ホストからへのアクセスは、サードパーティの依存関係挿入フレームワークを使用するなど、ASP.NET Core 外部の領域と統合する場合に役立ちます。

```csharp
    public class Program
    {
        public static void Main(string[] args)
        {
            var host = CreateHostBuilder(args).Build();
            var hubContext = host.Services.GetService(typeof(IHubContext<ChatHub>));
            host.Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder => {
                    webBuilder.UseStartup<Startup>();
                });
    }
```

### <a name="inject-a-strongly-typed-hubcontext"></a>厳密に型指定された HubContext を挿入する

厳密に型指定された HubContext を挿入するには、ハブがから継承されていることを確認し `Hub<T>` ます。 ではなく、インターフェイスを使用して挿入 `IHubContext<THub, T>` `IHubContext<THub>` します。

```csharp
public class ChatController : Controller
{
    public IHubContext<ChatHub, IChatClient> _strongChatHubContext { get; }

    public ChatController(IHubContext<ChatHub, IChatClient> chatHubContext)
    {
        _strongChatHubContext = chatHubContext;
    }

    public async Task SendMessage(string user, string message)
    {
        await _strongChatHubContext.Clients.All.ReceiveMessage(user, message);
    }
}
```

詳細については、「 [厳密に型指定](xref:signalr/hubs#strongly-typed-hubs) されたハブ」をご覧ください。

## <a name="related-resources"></a>関連リソース

* [開始するには](xref:tutorials/signalr)
* [ハブ](xref:signalr/hubs)
* [Azure に発行する](xref:signalr/publish-to-azure-web-app)
