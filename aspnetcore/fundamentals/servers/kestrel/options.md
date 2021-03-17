---
title: ASP.NET Core Kestrel Web サーバーのオプションを構成する
author: rick-anderson
description: ASP.NET Core 用のクロスプラットフォーム Web サーバーである Kestrel のオプションを構成する方法について説明します。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 05/04/2020
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
uid: fundamentals/servers/kestrel/options
ms.openlocfilehash: 48b4af2dfc925c4444c2bd0e43d04f2f0f3ddd17
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587009"
---
# <a name="configure-options-for-the-aspnet-core-kestrel-web-server"></a>ASP.NET Core Kestrel Web サーバーのオプションを構成する

Kestrel Web サーバーには、インターネットに接続する展開で特に有効な制約構成オプションがいくつかあります。

<xref:Microsoft.Extensions.Hosting.GenericHostBuilderExtensions.ConfigureWebHostDefaults%2A> を呼び出した後に追加の構成を指定するには、<xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.ConfigureKestrel%2A> を使用します。

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel(serverOptions =>
            {
                // Set properties and call methods on options
            })
            .UseStartup<Startup>();
        });
```

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> クラスの <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Limits> プロパティで制約を設定します。 `Limits` プロパティは、<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits> クラスのインスタンスを保持します。

<xref:Microsoft.AspNetCore.Server.Kestrel.Core> 名前空間を使用する例を次に示します。

```csharp
using Microsoft.AspNetCore.Server.Kestrel.Core;
```

この記事の後半で示す例では、Kestrel オプションが C# コードで構成されています。 Kestrel オプションは、[構成プロバイダー](xref:fundamentals/configuration/index)を使用して設定することもできます。 たとえば、[ファイル構成プロバイダー](xref:fundamentals/configuration/index#file-configuration-provider)を使用すると、`appsettings.json` ファイルまたは `appsettings.{Environment}.json` ファイルから、Kestrel 構成を読み込むことができます。

```json
{
  "Kestrel": {
    "Limits": {
      "MaxConcurrentConnections": 100,
      "MaxConcurrentUpgradedConnections": 100
    },
    "DisableStringReuse": true
  }
}
```

> [!NOTE]
> <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> および[エンドポイント構成](xref:fundamentals/servers/kestrel/endpoints)は、構成プロバイダーから構成できます。 残りの Kestrel 構成は、C# コードで構成する必要があります。

次の方法の **いずれか** を使用します。

* `Startup.ConfigureServices` で Kestrel を構成する。

  1. `IConfiguration` のインスタンスを `Startup` クラスに挿入します。 以下の例では、挿入された構成が `Configuration` プロパティに割り当てられることを前提としています。
  2. `Startup.ConfigureServices` で、構成の `Kestrel` セクションを Kestrel の構成に読み込みます。

     ```csharp
     using Microsoft.Extensions.Configuration
     
     public class Startup
     {
         public Startup(IConfiguration configuration)
         {
             Configuration = configuration;
         }

         public IConfiguration Configuration { get; }

         public void ConfigureServices(IServiceCollection services)
         {
             services.Configure<KestrelServerOptions>(
                 Configuration.GetSection("Kestrel"));
         }

         public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
         {
             ...
         }
     }
     ```

* ホストのビルド時に Kestrel を構成する。

  `Program.cs` で、構成の `Kestrel` セクションを Kestrel の構成に読み込みます。

  ```csharp
  // using Microsoft.Extensions.DependencyInjection;

  public static IHostBuilder CreateHostBuilder(string[] args) =>
      Host.CreateDefaultBuilder(args)
          .ConfigureServices((context, services) =>
          {
              services.Configure<KestrelServerOptions>(
                  context.Configuration.GetSection("Kestrel"));
          })
          .ConfigureWebHostDefaults(webBuilder =>
          {
              webBuilder.UseStartup<Startup>();
          });
  ```

上記の方法はいずれも、任意の[構成プロバイダー](xref:fundamentals/configuration/index)で使用できます。

## <a name="general-limits"></a>全般的な制限

### <a name="keep-alive-timeout"></a>Keep-Alive タイムアウト

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.KeepAliveTimeout>

[Keep-Alive タイムアウト](https://tools.ietf.org/html/rfc7230#section-6.5)を取得するか、設定します。 既定値は 2 分です。

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=19-20)]

### <a name="maximum-client-connections"></a>クライアントの最大接続数

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentConnections><br>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentUpgradedConnections>

次のコードを使用することで、アプリ全体に対して同時に開かれる TCP 接続の最大数を設定できます。

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=3)]

HTTP または HTTPS から別のプロトコルにアップグレードされた接続については別個の制限があります (WebSockets 要求に関する制限など)。 接続がアップグレードされた後、それは `MaxConcurrentConnections` 制限に対してカウントされません。

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=4)]

接続の最大数は既定では無制限 (null) です。

### <a name="maximum-request-body-size"></a>要求本文の最大サイズ

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxRequestBodySize>

既定の要求本文の最大サイズは、30,000,000 バイトです。これは約 28.6 MB になります。

ASP.NET Core MVC アプリでの制限をオーバーライドする方法としては、アクション メソッドに対して <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> 属性を使用することをお勧めします。

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

次の例では、すべての要求について、アプリの制約を構成する方法を示しています。

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=5)]

ミドルウェア内の特定の要求で設定をオーバーライドします。

[!code-csharp[](samples/3.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=3-4)]

アプリが要求の読み取りを開始した後で、要求に対する制限をアプリで構成すると、例外がスローされます。 `MaxRequestBodySize` プロパティが読み取り専用状態にある (制限を構成するには遅すぎる) かどうかを示す `IsReadOnly` プロパティがあります。

[ASP.NET Core Module](xref:host-and-deploy/aspnet-core-module) の背後で、アプリが[アウト プロセス](xref:host-and-deploy/iis/index#out-of-process-hosting-model)で実行される場合は、Kestrel の要求本文サイズの上限は無効になります。 IIS によって、上限が既に設定されています。

### <a name="minimum-request-body-data-rate"></a>要求本文の最小レート

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinRequestBodyDataRate><br>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinResponseDataRate>

kestrel はデータが指定のレート (バイト数/秒) で到着しているかどうかを毎秒チェックします。 レートが最小値を下回った場合は接続がタイムアウトになります。猶予期間とは、クライアントによって送信レートを最低ラインまで引き上げられるのを、Kestrel が待機する時間のことです。 この期間中、レートはチェックされません。 猶予期間を設定すると、TCP のスロースタートが原因で最初のデータ送信が低速レートで行われる接続の切断を回避することができます。

既定の最小レートは 240 バイト/秒であり、5 秒の猶予時間が設定されています。

最小レートは応答にも適用されます。 要求制限と応答制限を設定するコードは、プロパティ名およびインターフェイス名に `RequestBody` または `Response` が使用されることを除けば同じです。

次の例では、`Program.cs` で最小データ レートを構成する方法を示します。

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=6-11)]

ミドルウェアでは要求ごとに最小レート制限をオーバーライドします。

[!code-csharp[](samples/3.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=6-21)]

前のサンプルで参照した <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinResponseDataRateFeature> は、HTTP/2 要求の <xref:Microsoft.AspNetCore.Http.HttpContext.Features?displayProperty=nameWithType> には存在しません。 要求の多重化がプロトコルでサポートされているため、要求ごとにレート制限を変更することは、一般的に HTTP/2 ではサポートされていません。 ただし、<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinRequestBodyDataRateFeature> は引き続き現在の HTTP/2 要求の `HttpContext.Features` です。これは、HTTP/2 要求に対してであっても、<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinResponseDataRateFeature.MinDataRate?displayProperty=nameWithType> を `null` に設定すれば、読み取りのレート制限を要求ごとに "*すべて無効*" にできるためです。 `IHttpMinRequestBodyDataRateFeature.MinDataRate` を読み取ろうとしたり、`null` 以外の値に設定しようとしたりすると、HTTP/2 要求を指定した <xref:System.NotSupportedException> がスローされます。

<xref:Microsoft.AspNetCore.Server.Kestrel.KestrelServerOptions.Limits?displayProperty=nameWithType> で構成したサーバー全体のレート制限は、引き続き HTTP/1.x と HTTP/2 の両方の接続に適用されます。

### <a name="request-headers-timeout"></a>要求ヘッダー タイムアウト

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.RequestHeadersTimeout>

サーバーで要求ヘッダーの受信にかかる時間の最大値を取得または設定します。 既定値は 30 秒です。

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=21-22)]

## <a name="http2-limits"></a>HTTP/2 制限

このセクションの制限は、<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.Http2?displayProperty=nameWithType> に設定されています。

### <a name="maximum-streams-per-connection"></a>接続ごとの最大ストリーム

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.MaxStreamsPerConnection>

HTTP/2 接続ごとの同時要求ストリームの数を制限します。 余分なストリームは拒否されます。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxStreamsPerConnection = 100;
});
```

既定値は 100 です。

### <a name="header-table-size"></a>ヘッダー テーブルのサイズ

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.HeaderTableSize>

HTTP/2 接続では、HTTP ヘッダーは HPACK デコーダーによって圧縮解除されます。 HPACK デコーダーが使用するヘッダー圧縮テーブルのサイズは `HeaderTableSize` によって制限されます。 値はオクテット単位で指定し、ゼロ (0) より大きくなければなりません。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.HeaderTableSize = 4096;
});
```

既定値は 4096 です。

### <a name="maximum-frame-size"></a>最大フレーム サイズ

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.MaxFrameSize>

サーバーによって受信または送信された HTTP/2 接続フレーム ペイロードの最大許容サイズを示します。 値はオクテット単位で指定し、2^14 (16,384) から 2^24-1 (16,777,215) までの範囲とする必要があります。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxFrameSize = 16384;
});
```

既定値は 2^14 (16,384) です。

### <a name="maximum-request-header-size"></a>最大要求ヘッダー サイズ

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.MaxRequestHeaderFieldSize>

要求ヘッダー値の最大許容サイズをオクテット単位で示します。 この制限は、名前と値の圧縮表示と非圧縮表示の両方に適用されます。 ゼロ (0) より大きい値である必要があります。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxRequestHeaderFieldSize = 8192;
});
```

既定値は 8,192 です。

### <a name="initial-connection-window-size"></a>初期接続ウィンドウ サイズ

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.InitialConnectionWindowSize>

接続ごとの要求 (ストリーム) 全体で、サーバーによって一度にバッファーされる要求本文の最大データ量をバイト単位で示します。 要求は `Http2.InitialStreamWindowSize` による制限も受けます。 この値は 65,535 より大きく、2^31 (2,147,483,648) 未満である必要があります。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.InitialConnectionWindowSize = 131072;
});
```

既定値は 128 KB (131,072) です。

### <a name="initial-stream-window-size"></a>初期ストリーム ウィンドウ サイズ

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.InitialStreamWindowSize>

要求 (ストリーム) ごとに、サーバーによって一度にバッファーされる要求本文の最大データ量をバイト単位で示します。 また、要求は、[`InitialConnectionWindowSize`](#initial-connection-window-size) による制限も受けます。 この値は 65,535 より大きく、2^31 (2,147,483,648) 未満である必要があります。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.InitialStreamWindowSize = 98304;
});
```

既定値は 96 KB (98,304) です。

### <a name="http2-keep-alive-ping-configuration"></a>HTTP/2 キープ アライブ ping 構成

Kestrel は、接続されているクライアントに HTTP/2 ping を送信するように構成できます。 HTTP/2 ping は複数の目的で機能します。

* アイドル状態の接続を維持します。 一部のクライアントとプロキシ サーバーでは、アイドル状態の接続が閉じられます。 HTTP/2 ping は接続におけるアクティビティと見なされ、接続がアイドル状態として閉じられるのを防ぎます。
* 異常な接続を閉じます。 構成された時間でキープ アライブ ping にクライアントが応答しない接続はサーバーによって閉じられます。

HTTP/2 キープ アライブ ping に関連する 2 つの構成オプション:

* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.KeepAlivePingDelay> は、ping 間隔を構成する <xref:System.TimeSpan> です。 この時間内にフレームが受信されない場合、サーバーからクライアントにキープ アライブ ping が送信されます。 このオプションが <xref:System.TimeSpan.MaxValue?displayProperty=nameWithType> に設定されているとき、キープ アライブ ping は無効になります。 既定値は <xref:System.TimeSpan.MaxValue?displayProperty=nameWithType> です。
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.KeepAlivePingTimeout> は、ping タイムアウトを構成する <xref:System.TimeSpan> です。 このタイムアウト内でサーバーが応答 ping など、いかなるフレームも受信しない場合、接続は閉じられます。 このオプションが <xref:System.TimeSpan.MaxValue?displayProperty=nameWithType> に設定されているとき、キープ アライブ タイムアウトは無効になります。 既定値は 20 秒です。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.KeepAlivePingDelay = TimeSpan.FromSeconds(30);
    serverOptions.Limits.Http2.KeepAlivePingTimeout = TimeSpan.FromSeconds(60);
});
```

## <a name="other-options"></a>その他のオプション

### <a name="synchronous-io"></a>同期 I/O

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.AllowSynchronousIO> を使うと、要求と応答に対して同期 I/O を許可するかどうかを制御できます。 既定値は `false` です。

> [!WARNING]
> ブロッキング同期 I/O 操作の回数が多いと、スレッド プールの不足を招き、アプリが応答しなくなる可能性があります。 非同期 I/O をサポートしていないライブラリを使用する場合にのみ `AllowSynchronousIO` を有効にしてください。

同期 I/O を有効にする例を次に示します。

[!code-csharp[](samples/5.x/KestrelSample/Program.cs?name=snippet_SyncIO)]

Kestrel のその他のオプションと制限については、以下をご覧ください。

* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>
