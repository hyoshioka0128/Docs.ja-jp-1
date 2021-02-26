---
title: ASP.NET Core Blazor SignalR ガイダンス
author: guardrex
description: Blazor SignalR の接続を構成および管理する方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/27/2021
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
ms.openlocfilehash: 3198f45819020ca551617aa12a146f2b8a9a9f8e
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100279861"
---
# <a name="aspnet-core-blazor-signalr-guidance"></a>ASP.NET Core Blazor SignalR ガイダンス

ASP.NET Core SignalR の構成の一般的なガイダンスについては、ドキュメントの「<xref:signalr/introduction>」領域のトピックを参照してください。 [ホステッド Blazor WebAssembly ソリューションに追加された](xref:tutorials/signalr-blazor) SignalR を構成する方法については、「<xref:signalr/configuration#configure-server-options>」を参照してください。

## <a name="circuit-handler-options"></a>回線ハンドラーのオプション

*このセクションは Blazor Server に適用されます。*

次の表に示す <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions> を使用して Blazor Server の回線を構成してください。

| オプション | Default | 説明 |
| --- | --- | --- |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DetailedErrors> | `false` | 回線でハンドルされない例外が発生した場合、または JS 相互運用機能を介した .NET メソッドの呼び出しの結果として例外が発生した場合に、詳細な例外メッセージを JavaScript に送信します。 |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DisconnectedCircuitMaxRetained> | 100 | 特定のサーバーが一度にメモリに保持する切断された回線の最大数。 |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DisconnectedCircuitRetentionPeriod> | 3 分 | 切断された回線が破棄されるまでにメモリに保持される最大時間。 |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout> | 1 分 | 非同期の JavaScript 関数呼び出しがタイムアウトするまでにサーバーが待機する最大時間。 |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.MaxBufferedUnacknowledgedRenderBatches> | 10 | 堅牢な再接続をサポートするために、サーバーが一定期間、メモリに保持する回線あたりの未確認のレンダリング バッチの最大数。 制限に達すると、クライアントによって 1 つ以上のバッチが確認されるまで、サーバーは新しいレンダリング バッチの生成を停止します。 |

オプションの <xref:Microsoft.Extensions.DependencyInjection.ComponentServiceCollectionExtensions.AddServerSideBlazor%2A> へのデリゲートを使用して `Startup.ConfigureServices` のオプションを構成します。 次の例では、前の表に示した既定のオプション値を割り当てます。

```csharp
using System;

...

services.AddServerSideBlazor(options =>
{
    options.DetailedErrors = false;
    options.DisconnectedCircuitMaxRetained = 100;
    options.DisconnectedCircuitRetentionPeriod = TimeSpan.FromMinutes(3);
    options.JSInteropDefaultCallTimeout = TimeSpan.FromMinutes(1);
    options.MaxBufferedUnacknowledgedRenderBatches = 10;
});
```

<xref:Microsoft.AspNetCore.SignalR.HubConnectionContext> を構成するには、<xref:Microsoft.AspNetCore.SignalR.HubConnectionContextOptions> と共に <xref:Microsoft.Extensions.DependencyInjection.ServerSideBlazorBuilderExtensions.AddHubOptions%2A> を使用します。 オプションの説明については、「<xref:signalr/configuration#configure-server-options>」を参照してください。 次の例では、既定のオプション値を割り当てます。

```csharp
using System;

...

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

## <a name="signalr-cross-origin-negotiation-for-authentication"></a>認証のための SignalR のクロスオリジンネゴシエーション

*このセクションは Blazor WebAssembly に適用されます。*

cookie や HTTP 認証ヘッダーなどの資格情報を送信するように SignalR の基となるクライアントを構成するには:

* <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestCredentials%2A> を使用して、クロスオリジン [`fetch`](https://developer.mozilla.org/docs/Web/API/Fetch_API/Using_Fetch) 要求に <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.BrowserRequestCredentials.Include> を設定します。

  ```csharp
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

* <xref:System.Net.Http.HttpMessageHandler> を <xref:Microsoft.AspNetCore.Http.Connections.Client.HttpConnectionOptions.HttpMessageHandlerFactory> オプションに割り当てます。

  ```csharp
  var connection = new HubConnectionBuilder()
      .WithUrl(new Uri("http://signalr.example.com"), options =>
      {
          options.HttpMessageHandlerFactory = innerHandler => 
              new IncludeRequestCredentialsMessageHandler { InnerHandler = innerHandler };
      }).Build();
  ```

詳細については、「<xref:signalr/configuration#configure-additional-options>」を参照してください。

## <a name="reflect-the-connection-state-in-the-ui"></a>UI に接続状態を反映する

*このセクションは Blazor Server に適用されます。*

接続が失われたことがクライアントで検出されると、クライアントによって再接続が試行される間、ユーザーに対して既定の UI が表示されます。 再接続に失敗した場合、ユーザーには再試行のオプションが表示されます。

UI をカスタマイズするには、`_Host.cshtml` Razor ページの `<body>` に、`components-reconnect-modal` の `id` を持つ要素を定義します。

```cshtml
<div id="components-reconnect-modal">
    ...
</div>
```

アプリのスタイルシート (`wwwroot/css/app.css` または `wwwroot/css/site.css`) に以下を追加します。

```css
#components-reconnect-modal {
    display: none;
}

#components-reconnect-modal.components-reconnect-show {
    display: block;
}
```

次の表では、`components-reconnect-modal` 要素に適用される CSS クラスについて説明します。

| CSS クラス                       | 示す内容&hellip; |
| ------------------------------- | ----------------- |
| `components-reconnect-show`     | 接続が失われました。 クライアントによって再接続が試行されています。 モーダルを表示します。 |
| `components-reconnect-hide`     | サーバーへのアクティブな接続が再確立されます。 モーダルを非表示にします。 |
| `components-reconnect-failed`   | 再接続に失敗しました。ネットワーク障害が原因である可能性があります。 再接続を試みるには、`window.Blazor.reconnect()` を呼び出します。 |
| `components-reconnect-rejected` | 再接続が拒否されました。 サーバーに到達したが接続が拒否されたため、サーバー上のユーザーの状態が失われました。 アプリを再度読み込むには、`location.reload()` を呼び出します。 この接続状態は、次の場合に発生する可能性があります。<ul><li>サーバー側回線でクラッシュが発生した場合。</li><li>クライアントが長時間切断されているため、サーバーでユーザーの状態が削除された場合。 ユーザーが対話しているコンポーネントのインスタンスは破棄されます。</li><li>サーバーが再起動されたか、アプリのワーカー プロセスがリサイクルされた場合。</li></ul> |

## <a name="render-mode"></a>表示モード

::: moniker range=">= aspnetcore-5.0"

*このセクションは、ホストされている Blazor WebAssembly および Blazor Server に適用されます。*

Blazor アプリは既定では、サーバー上の UI をプリレンダリングするように設定されます。 詳細については、「<xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>」を参照してください。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

*このセクションは Blazor Server に適用されます。*

Blazor Server アプリは、サーバーへのクライアント接続が確立される前に、サーバー上の UI をプリレンダリングするよう既定で設定されます。 詳細については、「<xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>」を参照してください。

::: moniker-end

## <a name="initialize-the-blazor-circuit"></a>Blazor 回線を初期化する

*このセクションは Blazor Server に適用されます。*

`Pages/_Host.cshtml` ファイル内にある Blazor Server アプリの [SignalR 回線](xref:blazor/hosting-models#circuits)の手動での起動を構成します。

* `blazor.server.js` スクリプトの `<script>` タグに `autostart="false"` 属性を追加します。
* `Blazor.start` を呼び出すスクリプトを、`blazor.server.js` スクリプトのタグの後の終了 `</body>` タグ内に配置します。

`autostart` が無効になっている場合、回線に依存しないアプリのすべての側面が正常に動作します。 たとえば、クライアント側のルーティングは動作します。 ただし、回線に依存する側面はすべて、`Blazor.start` が呼び出されるまで動作しません。 回線が確立されていなければ、アプリの動作は予測不可能です。 たとえば、回線が切断されている間、コンポーネント メソッドは実行できません。

### <a name="initialize-blazor-when-the-document-is-ready"></a>ドキュメントの準備完了時に Blazor を初期化する

ドキュメントの準備完了時に Blazor アプリを初期化するには、次のようにします。

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

JS 相互運用機能の初期化など、追加のタスクを実行するには、`then` を使用して、手動で Blazor アプリを起動した結果として得た `Promise` に連結します。

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

### <a name="configure-the-signalr-client"></a>SignalR クライアントを構成する

#### <a name="logging"></a>ログの記録

SignalR クライアント ログを構成するには、クライアント ビルダーでログ レベルを指定して `configureLogging` を呼び出す構成オブジェクト (`configureSignalR`) を渡します。

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

`onConnectionDown` と `onConnectionUp` の **両方** を指定する必要があります。

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

モーダル要素に対して、アプリの CSS (`wwwroot/css/site.css`) の `transition-delay` プロパティを設定して、再接続表示が表示されるまでの遅延時間をカスタマイズします。 次の例では、移行遅延時間を 500 ミリ秒 (既定値) から 1,000 ミリ秒 (1 秒) に設定しています。

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

<!-- HOLD for reactivation at 5x

THIS WILL BE MOVED TO ANOTHER TOPIC WHEN RE-ACTIVATED.

## Influence HTML `<head>` tag elements

*This section applies to the upcoming ASP.NET Core 5.0 release of Blazor WebAssembly and Blazor Server.*

When rendered, the `Title`, `Link`, and `Meta` components add or update data in the HTML `<head>` tag elements:

```razor
@using Microsoft.AspNetCore.Components.Web.Extensions.Head

<Title Value="{TITLE}" />
<Link href="{URL}" rel="stylesheet" />
<Meta content="{DESCRIPTION}" name="description" />
```

In the preceding example, placeholders for `{TITLE}`, `{URL}`, and `{DESCRIPTION}` are string values, Razor variables, or Razor expressions.

The following characteristics apply:

* Server-side prerendering is supported.
* The `Value` parameter is the only valid parameter for the `Title` component.
* HTML attributes provided to the `Meta` and `Link` components are captured in [additional attributes](xref:blazor/components/index#attribute-splatting-and-arbitrary-parameters) and passed through to the rendered HTML tag.
* For multiple `Title` components, the title of the page reflects the `Value` of the last `Title` component rendered.
* If multiple `Meta` or `Link` components are included with identical attributes, there's exactly one HTML tag rendered per `Meta` or `Link` component. Two `Meta` or `Link` components can't refer to the same rendered HTML tag.
* Changes to the parameters of existing `Meta` or `Link` components are reflected in their rendered HTML tags.
* When the `Link` or `Meta` components are no longer rendered and thus disposed by the framework, their rendered HTML tags are removed.

When one of the framework components is used in a child component, the rendered HTML tag influences any other child component of the parent component as long as the child component containing the framework component is rendered. The distinction between using the one of these framework components in a child component and placing a an HTML tag in `wwwroot/index.html` or `Pages/_Host.cshtml` is that a framework component's rendered HTML tag:

* Can be modified by application state. A hard-coded HTML tag can't be modified by application state.
* Is removed from the HTML `<head>` when the parent component is no longer rendered.

-->

::: moniker-end

## <a name="additional-resources"></a>その他のリソース

* <xref:signalr/introduction>
* <xref:signalr/configuration>
* <xref:blazor/security/server/threat-mitigation>
* [Blazor Server 再接続イベントとコンポーネント ライフサイクル イベント](xref:blazor/components/lifecycle#blazor-server-reconnection-events)
