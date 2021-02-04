---
title: SignalRバックグラウンドサービスでのホスト ASP.NET Core
author: bradygaster
description: .NET Core BackgroundService クラスからクライアントにメッセージを送信する方法について説明 SignalR します。
monikerRange: '>= aspnetcore-2.2'
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
uid: signalr/background-services
ms.openlocfilehash: 4b877c64a881fec15ac9e9bd74ffdde0b5fa60f9
ms.sourcegitcommit: e311cfb77f26a0a23681019bd334929d1aaeda20
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99530178"
---
# <a name="host-aspnet-core-signalr-in-background-services"></a>SignalRバックグラウンドサービスでのホスト ASP.NET Core

[Brady](https://twitter.com/bradygaster)による

この記事では、次のガイダンスを提供します。

* SignalRASP.NET Core でホストされたバックグラウンドワーカープロセスを使用してハブをホストする。
* .NET Core [Backgroundservice](xref:Microsoft.Extensions.Hosting.BackgroundService)内から接続されたクライアントにメッセージを送信する。

::: moniker range=">= aspnetcore-3.0"

[サンプル コードを表示またはダウンロードする](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/signalr/background-service/samples/3.x) ([ダウンロード方法](xref:index#how-to-download-a-sample))

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[サンプル コードを表示またはダウンロードする](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/signalr/background-service/samples/2.2) ([ダウンロード方法](xref:index#how-to-download-a-sample))

::: moniker-end

## <a name="enable-signalr-in-startup"></a>スタートアップで有効にする SignalR

::: moniker range=">= aspnetcore-3.0"

SignalRバックグラウンドワーカープロセスのコンテキストで ASP.NET Core ハブをホストすることは、ASP.NET Core web アプリでハブをホストすることと同じです。 メソッドで `Startup.ConfigureServices` を呼び出すと、を `services.AddSignalR` サポートするために必要なサービスが ASP.NET Core 依存関係挿入 (DI) 層に追加され SignalR ます。 では、 `Startup.Configure` `MapHub` メソッドがコールバックで呼び出され、 `UseEndpoints` ASP.NET Core 要求パイプラインのハブエンドポイントに接続します。

[!code-csharp[Startup](background-service/samples/3.x/Server/Startup.cs?name=Startup)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

SignalRバックグラウンドワーカープロセスのコンテキストで ASP.NET Core ハブをホストすることは、ASP.NET Core web アプリでハブをホストすることと同じです。 メソッドで `Startup.ConfigureServices` を呼び出すと、を `services.AddSignalR` サポートするために必要なサービスが ASP.NET Core 依存関係挿入 (DI) 層に追加され SignalR ます。 で `Startup.Configure` は、 `UseSignalR` メソッドを呼び出して、ASP.NET Core 要求パイプラインのハブエンドポイントを接続します。

[!code-csharp[Startup](background-service/samples/2.2/Server/Startup.cs?name=Startup)]

::: moniker-end

前の例では、 `ClockHub` クラスは、 `Hub<T>` 厳密に型指定されたハブを作成するクラスを実装しています。 は、 `ClockHub` `Startup` エンドポイントでの要求に応答するようにクラスで構成されてい `/hubs/clock` ます。

厳密に型指定されたハブの詳細については、「 [ SignalR のハブを使用](xref:signalr/hubs#strongly-typed-hubs)した ASP.NET Core」を参照してください。

> [!NOTE]
> この機能は[ハブ \<T> ](xref:Microsoft.AspNetCore.SignalR.Hub`1)クラスに限定されません。 [Dynamichub](xref:Microsoft.AspNetCore.SignalR.DynamicHub)など、[ハブ](xref:Microsoft.AspNetCore.SignalR.Hub)から継承するクラスはすべて機能します。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[Startup](background-service/samples/3.x/Server/ClockHub.cs?name=ClockHub)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[!code-csharp[Startup](background-service/samples/2.2/Server/ClockHub.cs?name=ClockHub)]

::: moniker-end

厳密に型指定されたによって使用されるインターフェイスは、 `ClockHub` `IClock` インターフェイスです。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[Startup](background-service/samples/3.x/HubServiceInterfaces/IClock.cs?name=IClock)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[!code-csharp[Startup](background-service/samples/2.2/HubServiceInterfaces/IClock.cs?name=IClock)]

::: moniker-end

## <a name="call-a-signalr-hub-from-a-background-service"></a>SignalRバックグラウンドサービスからハブを呼び出す

起動時に、クラスはを使用して `Worker` `BackgroundService` 有効になり `AddHostedService` ます。

```csharp
services.AddHostedService<Worker>();
```

SignalRはフェーズ中にも有効に `Startup` なるため、各ハブは ASP.NET CORE の HTTP 要求パイプライン内の個々のエンドポイントにアタッチされます。各ハブは、サーバー上のによって表され `IHubContext<T>` ます。 ASP.NET Core の DI 機能を使用して、クラス、MVC コントローラークラス、ページモデルなどのホスト層によってインスタンス化された他のクラスは、 `BackgroundService` Razor 構築中にのインスタンスを受け入れることによって、サーバー側ハブへの参照を取得でき `IHubContext<ClockHub, IClock>` ます。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[Startup](background-service/samples/3.x/Server/Worker.cs?name=Worker)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[!code-csharp[Startup](background-service/samples/2.2/Server/Worker.cs?name=Worker)]

::: moniker-end

`ExecuteAsync`バックグラウンドサービスでメソッドが繰り返し呼び出されると、サーバーの現在の日付と時刻が、を使用して接続されたクライアントに送信され `ClockHub` ます。

## <a name="react-to-signalr-events-with-background-services"></a>SignalRバックグラウンドサービスを使用したイベントへの対応

の JavaScript クライアントを使用するシングルページアプリ SignalR や、を使用する .net デスクトップアプリのよう <xref:signalr/dotnet-client> に、またはの実装を使用して `BackgroundService` `IHostedService` ハブに接続し、イベントに応答することもでき SignalR ます。

クラスは、 `ClockHubClient` インターフェイスとインターフェイスの両方を実装し `IClock` `IHostedService` ます。 このようにして、の実行中にを有効に `Startup` し、サーバーからハブイベントに応答することができます。

```csharp
public partial class ClockHubClient : IClock, IHostedService
{
}
```

初期化中に、によっ `ClockHubClient` てのインスタンスが作成され、その `HubConnection` `IClock.ShowTime` メソッドがハブのイベントのハンドラーとして有効になり `ShowTime` ます。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[The ClockHubClient constructor](background-service/samples/3.x/Clients.ConsoleTwo/ClockHubClient.cs?name=ClockHubClientCtor)]

実装では `IHostedService.StartAsync` 、 `HubConnection` が非同期的に開始されます。

[!code-csharp[StartAsync method](background-service/samples/3.x/Clients.ConsoleTwo/ClockHubClient.cs?name=StartAsync)]

メソッドの実行中 `IHostedService.StopAsync` 、は `HubConnection` 非同期的に破棄されます。

[!code-csharp[StopAsync method](background-service/samples/3.x/Clients.ConsoleTwo/ClockHubClient.cs?name=StopAsync)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[!code-csharp[The ClockHubClient constructor](background-service/samples/2.2/Clients.ConsoleTwo/ClockHubClient.cs?name=ClockHubClientCtor)]

実装では `IHostedService.StartAsync` 、 `HubConnection` が非同期的に開始されます。

[!code-csharp[StartAsync method](background-service/samples/2.2/Clients.ConsoleTwo/ClockHubClient.cs?name=StartAsync)]

メソッドの実行中 `IHostedService.StopAsync` 、は `HubConnection` 非同期的に破棄されます。

[!code-csharp[StopAsync method](background-service/samples/2.2/Clients.ConsoleTwo/ClockHubClient.cs?name=StopAsync)]

::: moniker-end

## <a name="additional-resources"></a>その他のリソース

* [開始するには](xref:tutorials/signalr)
* [取って代わり](xref:signalr/hubs)
* [Azure に発行する](xref:signalr/publish-to-azure-web-app)
* [厳密に型指定されたハブ](xref:signalr/hubs#strongly-typed-hubs)
