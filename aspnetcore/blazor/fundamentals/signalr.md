---
title: ASP.NET Core Blazor SignalR ガイダンス
author: guardrex
description: Blazor SignalR の接続を構成および管理する方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/12/2021
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
uid: blazor/fundamentals/signalr
zone_pivot_groups: blazor-hosting-models
ms.openlocfilehash: ecac21c1f6f7cea0a221e1a78161b915ee2c755f
ms.sourcegitcommit: 1f35de0ca9ba13ea63186c4dc387db4fb8e541e0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2021
ms.locfileid: "104711062"
---
# <a name="aspnet-core-blazor-signalr-guidance"></a>ASP.NET Core Blazor SignalR ガイダンス

::: zone pivot="webassembly"

この記事では、Blazor アプリで SignalR 接続を構成および管理する方法について説明します。

ASP.NET Core SignalR の構成の一般的なガイダンスについては、ドキュメントの「<xref:signalr/introduction>」領域のトピックを参照してください。 [ホステッド Blazor WebAssembly ソリューションに追加された](xref:tutorials/signalr-blazor) SignalR を構成する方法については、「<xref:signalr/configuration#configure-server-options>」を参照してください。

## <a name="signalr-cross-origin-negotiation-for-authentication"></a>認証のための SignalR のクロスオリジンネゴシエーション

cookie や HTTP 認証ヘッダーなどの資格情報を送信するように SignalR の基となるクライアントを構成するには:

* <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestCredentials%2A> を使用して、クロスオリジン [`fetch`](https://developer.mozilla.org/docs/Web/API/Fetch_API/Using_Fetch) 要求に <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.BrowserRequestCredentials.Include> を設定します。

  `IncludeRequestCredentialsMessageHandler.cs`:

  ```csharp
  using System.Net.Http;
  using System.Threading;
  using System.Threading.Tasks;
  using Microsoft.AspNetCore.Components.WebAssembly.Http;

  public class IncludeRequestCredentialsMessageHandler : DelegatingHandler
  {
      protected override Task<HttpResponseMessage> SendAsync(
          HttpRequestMessage request, CancellationToken cancellationToken)
      {
          request.SetBrowserRequestCredentials(BrowserRequestCredentials.Include);
          return base.SendAsync(request, cancellationToken);
      }
  }
  ```

* ハブ接続が構築されている場合は、<xref:System.Net.Http.HttpMessageHandler> を <xref:Microsoft.AspNetCore.Http.Connections.Client.HttpConnectionOptions.HttpMessageHandlerFactory> オプションに割り当てます。

  ```csharp
  HubConnectionBuilder hubConnecton;

  ...

  hubConnecton = new HubConnectionBuilder()
      .WithUrl(new Uri(NavigationManager.ToAbsoluteUri("/chathub")), options =>
      {
          options.HttpMessageHandlerFactory = innerHandler => 
              new IncludeRequestCredentialsMessageHandler { InnerHandler = innerHandler };
      }).Build();
  ```

  前の例では、ハブ接続 URL を `/chathub` の絶対 URI アドレスに構成しています。これは、`Index` コンポーネント (`Pages/Index.razor`) の [Blazor と SignalR のチュートリアル](xref:tutorials/signalr-blazor)で使用されている URL です。 URI は、文字列 (`https://signalr.example.com` など) または[構成](xref:blazor/fundamentals/configuration)を使用して設定することもできます。

詳細については、「<xref:signalr/configuration#configure-additional-options>」を参照してください。

::: moniker range=">= aspnetcore-5.0"

## <a name="render-mode"></a>表示モード

プリレンダリングするために SignalR を使用する Blazor WebAssembly アプリがサーバーに構成されている場合、サーバーへのクライアント接続が確立される前に、プリレンダリングが行われます。 詳細については、次の記事を参照してください。

* <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>
* <xref:blazor/components/prerendering-and-integration>

::: moniker-end

## <a name="additional-resources"></a>その他のリソース

* <xref:signalr/introduction>
* <xref:signalr/configuration>

::: zone-end

::: zone pivot="server"

この記事では、Blazor アプリで SignalR 接続を構成および管理する方法について説明します。

ASP.NET Core SignalR の構成の一般的なガイダンスについては、ドキュメントの「<xref:signalr/introduction>」領域のトピックを参照してください。 [ホステッド Blazor WebAssembly ソリューションに追加された](xref:tutorials/signalr-blazor) SignalR を構成する方法については、「<xref:signalr/configuration#configure-server-options>」を参照してください。

## <a name="circuit-handler-options"></a>回線ハンドラーのオプション

次の表に示す <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions> を使用して Blazor Server の回線を構成してください。

| オプション | Default | 説明 |
| --- | --- | --- |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DetailedErrors> | `false` | 回線でハンドルされない例外が発生した場合、または JS 相互運用機能を介した .NET メソッドの呼び出しの結果として例外が発生した場合に、詳細な例外メッセージを JavaScript に送信します。 |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DisconnectedCircuitMaxRetained> | 100 | サーバーが一度にメモリに保持する切断された回線の最大数。 |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DisconnectedCircuitRetentionPeriod> | 3 分 | 切断された回線が破棄されるまでにメモリに保持される最大時間。 |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout> | 1 分 | 非同期の JavaScript 関数呼び出しがタイムアウトするまでにサーバーが待機する最大時間。 |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.MaxBufferedUnacknowledgedRenderBatches> | 10 | 堅牢な再接続をサポートするために、サーバーが一定期間、メモリに保持する回線あたりの未確認のレンダリング バッチの最大数。 制限に達すると、クライアントによって 1 つ以上のバッチが確認されるまで、サーバーにより新しいレンダリング バッチの生成が停止されます。 |

オプションの <xref:Microsoft.Extensions.DependencyInjection.ComponentServiceCollectionExtensions.AddServerSideBlazor%2A> へのデリゲートを使用して `Startup.ConfigureServices` のオプションを構成します。 次の例では、前の表に示した既定のオプション値を割り当てます。 `Startup.cs` で <xref:System> 名前空間が使用されていることを確認します (`using System;`)。

`Startup.ConfigureServices`:

```csharp
services.AddServerSideBlazor(options =>
{
    options.DetailedErrors = false;
    options.DisconnectedCircuitMaxRetained = 100;
    options.DisconnectedCircuitRetentionPeriod = TimeSpan.FromMinutes(3);
    options.JSInteropDefaultCallTimeout = TimeSpan.FromMinutes(1);
    options.MaxBufferedUnacknowledgedRenderBatches = 10;
});
```

<xref:Microsoft.AspNetCore.SignalR.HubConnectionContext> を構成するには、<xref:Microsoft.AspNetCore.SignalR.HubConnectionContextOptions> と共に <xref:Microsoft.Extensions.DependencyInjection.ServerSideBlazorBuilderExtensions.AddHubOptions%2A> を使用します。 オプションの説明については、「<xref:signalr/configuration#configure-server-options>」を参照してください。 次の例では、既定のオプション値を割り当てます。 `Startup.cs` で <xref:System> 名前空間が使用されていることを確認します (`using System;`)。

`Startup.ConfigureServices`:

```csharp
services.AddServerSideBlazor()
    .AddHubOptions(options =>
    {
        options.ClientTimeoutInterval = TimeSpan.FromSeconds(30);
        options.EnableDetailedErrors = false;
        options.HandshakeTimeout = TimeSpan.FromSeconds(15);
        options.KeepAliveInterval = TimeSpan.FromSeconds(15);
        options.MaximumParallelInvocationsPerClient = 1;
        options.MaximumReceiveMessageSize = 32 * 1024;
        options.StreamBufferCapacity = 10;
    });
```

## <a name="reflect-the-connection-state-in-the-ui"></a>UI に接続状態を反映する

接続が失われたことがクライアントで検出されると、クライアントによって再接続が試行される間、ユーザーに対して既定の UI が表示されます。 再接続に失敗した場合、ユーザーには再試行のオプションが表示されます。

UI をカスタマイズするには、`_Host.cshtml` Razor ページの `<body>` に、`components-reconnect-modal` の `id` を持つ要素を定義します。

`Pages/_Host.cshtml`:

```cshtml
<div id="components-reconnect-modal">
    ...
</div>
```

次の CSS スタイルをサイトのスタイルシートに追加します。

`wwwroot/css/site.css`:

```css
#components-reconnect-modal {
    display: none;
}

#components-reconnect-modal.components-reconnect-show {
    display: block;
}
```

次の表で、Blazor フレームワークによって`components-reconnect-modal` 要素に適用される CSS クラスについて説明します。

| CSS クラス                       | 示す内容&hellip; |
| ------------------------------- | ----------------- |
| `components-reconnect-show`     | 接続が失われました。 クライアントによって再接続が試行されています。 モーダルを表示します。 |
| `components-reconnect-hide`     | サーバーへのアクティブな接続が再確立されます。 モーダルを非表示にします。 |
| `components-reconnect-failed`   | 再接続に失敗しました。ネットワーク障害が原因である可能性があります。 再接続を試みるには、JavaScript で `window.Blazor.reconnect()` を呼び出します。 |
| `components-reconnect-rejected` | 再接続が拒否されました。 サーバーに到達したが接続が拒否されたため、サーバー上のユーザーの状態が失われました。 アプリを再度読み込むには、JavaScript で `location.reload()` を呼び出します。 この接続状態は、次の場合に発生する可能性があります。<ul><li>サーバー側回線でクラッシュが発生した場合。</li><li>クライアントが長時間切断されているため、サーバーでユーザーの状態が削除された場合。 ユーザーのコンポーネントのインスタンスは破棄されます。</li><li>サーバーが再起動されたか、アプリのワーカー プロセスがリサイクルされた場合。</li></ul> |

## <a name="render-mode"></a>表示モード

既定では、サーバーへのクライアント接続が確立される前に、Blazor Server アプリによってサーバー上の UI がプリレンダリングされます。 詳細については、「<xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>」を参照してください。

## <a name="initialize-the-blazor-circuit"></a>Blazor 回線を初期化する

`Pages/_Host.cshtml` ファイル内にある Blazor Server アプリの [SignalR 回線](xref:blazor/hosting-models#circuits)の手動での起動を構成します。

* `blazor.server.js` スクリプトの `<script>` タグに `autostart="false"` 属性を追加します。
* `Blazor.start` を呼び出すスクリプトを、`blazor.server.js` スクリプトのタグの後の終了 `</body>` タグ内に配置します。

`autostart` が無効になっている場合、回線に依存しないアプリのすべての側面が正常に動作します。 たとえば、クライアント側のルーティングは動作します。 ただし、回線に依存する側面はすべて、`Blazor.start` が呼び出されるまで動作しません。 回線が確立されていなければ、アプリの動作は予測不可能です。 たとえば、回線が切断されている間、コンポーネント メソッドは実行できません。

### <a name="initialize-blazor-when-the-document-is-ready"></a>ドキュメントの準備完了時に Blazor を初期化する

`Pages/_Host.cshtml`:

```cshtml
<body>
    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      document.addEventListener("DOMContentLoaded", function() {
        Blazor.start();
      });
    </script>
</body>
```

### <a name="chain-to-the-promise-that-results-from-a-manual-start"></a>手動で起動した結果として得た `Promise` に連結する

JS 相互運用機能の初期化など、追加のタスクを実行するには、[`then`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) を使用して、手動で Blazor アプリを起動した結果として得た [`Promise`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) に連結します。

`Pages/_Host.cshtml`:

```cshtml
<body>
    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      Blazor.start().then(function () {
        ...
      });
    </script>
</body>
```

### <a name="configure-signalr-client-logging"></a>SignalR クライアント ログの構成

クライアント ビルダーで、ログ レベルを指定して `configureLogging` を呼び出す `configureSignalR` 構成オブジェクトを渡します。

`Pages/_Host.cshtml`:

```cshtml
<body>
    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      Blazor.start({
        configureSignalR: function (builder) {
          builder.configureLogging("information");
        }
      });
    </script>
</body>
```

前の例で、`information` はログ レベル <xref:Microsoft.Extensions.Logging.LogLevel.Information?displayProperty=nameWithType> と同じです。

### <a name="modify-the-reconnection-handler"></a>再接続ハンドラーを変更する

再接続ハンドラーの回線接続イベントは、次のようなカスタム動作を行うように変更できます。

* 接続が切断された場合にユーザーに通知する。
* 回線が接続されているときに (クライアントから) ログ記録を実行する。

接続イベントを変更するには、次の接続の変更に対してコールバックを登録します。

* 切断された接続では、`onConnectionDown` が使用されます。
* 確立または再確立された接続では、`onConnectionUp` が使用されます。

**`onConnectionDown` と `onConnectionUp` の両方を指定する必要があります。**

`Pages/_Host.cshtml`:

```cshtml
<body>
    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      Blazor.start({
        reconnectionHandler: {
          onConnectionDown: (options, error) => console.error(error),
          onConnectionUp: () => console.log("Up, up, and away!")
        }
      });
    </script>
</body>
```

### <a name="adjust-the-reconnection-retry-count-and-interval"></a>再接続の再試行回数と間隔を調整する

再接続の再試行の回数と間隔を調整するには、再試行の回数 (`maxRetries`) と、各再試行で許可されるミリ秒単位の期間 (`retryIntervalMilliseconds`) を設定します。

`Pages/_Host.cshtml`:

```cshtml
<body>
    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      Blazor.start({
        reconnectionOptions: {
          maxRetries: 3,
          retryIntervalMilliseconds: 2000
        }
      });
    </script>
</body>
```

## <a name="hide-or-replace-the-reconnection-display"></a>再接続の表示を非表示にする、または置き換える

再接続の表示を非表示にするには、再接続ハンドラーの `_reconnectionDisplay` を空のオブジェクト (`{}` または `new Object()`) に設定します。

`Pages/_Host.cshtml`:

```cshtml
<body>
    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      window.addEventListener('beforeunload', function () {
        Blazor.defaultReconnectionHandler._reconnectionDisplay = {};
      });

      Blazor.start();
    </script>
</body>
```

再接続の表示を置き換えるには、前の例の `_reconnectionDisplay` を表示する要素に設定します。

```javascript
Blazor.defaultReconnectionHandler._reconnectionDisplay = 
  document.getElementById("{ELEMENT ID}");
```

プレースホルダー `{ELEMENT ID}` は、表示する HTML 要素の ID です。

::: moniker range=">= aspnetcore-5.0"

モーダル要素に対して、サイトの CSS で `transition-delay` プロパティを設定して、再接続表示が表示されるまでの遅延時間をカスタマイズします。 次の例では、移行遅延時間を 500 ms (既定値) から 1,000 ms (1 秒) に設定しています。

`wwwroot/css/site.css`:

```css
#components-reconnect-modal {
    transition: visibility 0s linear 1000ms;
}
```

## <a name="disconnect-the-blazor-circuit-from-the-client"></a>クライアントから Blazor 回線を切断する

既定では、[`unload` ページ イベント](https://developer.mozilla.org/docs/Web/API/Window/unload_event)がトリガーされると、Blazor 回線が切断されます。 クライアント上の他のシナリオで回線を切断するには、適切なイベント ハンドラーで `Blazor.disconnect` を呼び出します。 次の例では、ページが非表示になると、回線が切断されます ([`pagehide` イベント](https://developer.mozilla.org/docs/Web/API/Window/pagehide_event))。

```javascript
window.addEventListener('pagehide', () => {
  Blazor.disconnect();
});
```

::: moniker-end

## <a name="blazor-server-circuit-handler"></a>Blazor Server 回線ハンドラー

Blazor Server を使用すると、コードで "*回線ハンドラー*" を定義できます。これにより、ユーザーの回線の状態の変更時にコードを実行できます。 回線ハンドラーは、<xref:Microsoft.AspNetCore.Components.Server.Circuits.CircuitHandler> から派生させ、そのクラスをアプリのサービス コンテナーに登録することで実装します。 次の回線ハンドラーの例では、開いている SignalR 接続を追跡します。

`TrackingCircuitHandler.cs`:

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](~/blazor/common/samples/5.x/BlazorSample_Server/TrackingCircuitHandler.cs)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-csharp[](~/blazor/common/samples/3.x/BlazorSample_Server/TrackingCircuitHandler.cs)]

::: moniker-end

回線ハンドラーは DI を使用して登録されます。 スコープを持つインスタンスは、回線のインスタンスごとに作成されます。 前の例の `TrackingCircuitHandler` を使用すると、すべての回線の状態を追跡する必要があるため、シングルトン サービスが作成されます。

`Startup.cs`:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...
    services.AddSingleton<CircuitHandler, TrackingCircuitHandler>();
}
```

カスタム回線ハンドラーのメソッドでハンドルされない例外がスローされる場合は、その例外は Blazor Server 回線にとって致命的です。 ハンドラーのコードまたはメソッドで例外が許容されるようにするには、エラー処理とログを含む 1 つ以上の [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) ステートメントでコードをラップします。

ユーザーが切断し、フレームワークで回線の状態がクリーンアップされていることが原因で回線が終了すると、フレームワークによって回線の DI スコープが破棄されます。 スコープが破棄されると、<xref:System.IDisposable?displayProperty=fullName> を実装するサーキットスコープの DI サービスはすべて破棄されます。 破棄中にいずれかの DI サービスでハンドルされない例外がスローされると、フレームワークによって例外がログに記録されます。

## <a name="additional-resources"></a>その他のリソース

* <xref:signalr/introduction>
* <xref:signalr/configuration>
* <xref:blazor/security/server/threat-mitigation>
* [Blazor Server 再接続イベントとコンポーネント ライフサイクル イベント](xref:blazor/components/lifecycle#blazor-server-reconnection-events)

::: zone-end
