---
title: プロキシ サーバーとロード バランサーを使用するために ASP.NET Core を構成する
author: rick-anderson
description: プロキシ サーバーとロード バランサーの背後にホストされているアプリの構成について説明します。このような構成では、要求の重要な情報がわからなくなることがよくあります。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/07/2020
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
uid: host-and-deploy/proxy-load-balancer
ms.openlocfilehash: e63821743dad565b51d5c2360dcc2fbf0632754f
ms.sourcegitcommit: e311cfb77f26a0a23681019bd334929d1aaeda20
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99530048"
---
# <a name="configure-aspnet-core-to-work-with-proxy-servers-and-load-balancers"></a>プロキシ サーバーとロード バランサーを使用するために ASP.NET Core を構成する

作成者: [Chris Ross](https://github.com/Tratcher)

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core の推奨される構成では、アプリは IIS/ASP.NET Core モジュール、Nginx、または Apache を使ってホストされます。 プロキシ サーバー、ロード バランサー、および他のネットワーク アプライアンスにより、要求に関する情報が、アプリに到達する前にわからなくなることがよくあります。

* HTTPS 要求が HTTP によってプロキシされると、元のスキーム (HTTPS) は失われ、ヘッダーで転送される必要があります。
* アプリは、インターネット上または社内ネットワーク上の本来の送信元ではなく、プロキシから要求を受信するため、送信元クライアントの IP アドレスもヘッダーで転送される必要があります。

この情報は、リダイレクト、認証、リンクの生成、ポリシーの評価、クライアントの位置情報など、要求の処理で重要になる可能性があります。

## <a name="forwarded-headers"></a>転送されるヘッダー

慣例によりは、プロキシは HTTP ヘッダーで情報を転送します。

| Header | 説明 |
| ------ | ----------- |
| X-Forwarded-For | 要求を開始したクライアントと、プロキシ チェーン内の後続のプロキシに関する情報を保持します。 このパラメーターは、IP アドレス (および、必要に応じてポート番号) を含むことがあります。 プロキシ サーバーのチェーンにおいては、最初のパラメーターが、要求を最初に行ったクライアントを示します。 その後に、後続のプロキシの識別子が指定されています。 チェーン内の最後のプロキシは、パラメーターのリストには含まれません。 最後のプロキシの IP アドレスとオプションのポート番号は、トランスポート層のリモート IP アドレスとして入手できます。 |
| X-Forwarded-Proto | 開始スキーム (HTTP/HTTPS) の値です。 要求が複数のプロキシを通過している場合、値はスキームのリストである場合もあります。 |
| X-Forwarded-Host | ホスト ヘッダー フィールドの元の値です。 通常、プロキシはホスト ヘッダーを変更しません。 プロキシにおいてホスト ヘッダーが既知の適切な値であることが検証されない、または既知の適切な値に制限されないシステムに影響を与える、特権の昇格脆弱性については、[マイクロソフト セキュリティ アドバイザリ CVE-2018-0787](https://github.com/aspnet/Announcements/issues/295) をご覧ください。 |

Forwarded Headers Middleware (<xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersMiddleware>) によって、これらのヘッダーが読み取られ、<xref:Microsoft.AspNetCore.Http.HttpContext> 上の関連するフィールドに入力されます。

ミドルウェアの更新:

* [HttpContext.Connection.RemoteIpAddress](xref:Microsoft.AspNetCore.Http.ConnectionInfo.RemoteIpAddress):`X-Forwarded-For` ヘッダー値を使用して設定します。 追加の設定は、ミドルウェアが `RemoteIpAddress` を設定する方法に影響を与えます。 詳しくは、「[Forwarded Headers Middleware のオプション](#forwarded-headers-middleware-options)」をご覧ください。
* [HttpContext.Request.Scheme](xref:Microsoft.AspNetCore.Http.HttpRequest.Scheme):`X-Forwarded-Proto` ヘッダー値を使用して設定します。
* [HttpContext.Request.Host](xref:Microsoft.AspNetCore.Http.HttpRequest.Host):`X-Forwarded-Host` ヘッダー値を使用して設定します。

Forwarded Headers Middleware の[既定の設定](#forwarded-headers-middleware-options)は構成できます。 既定の設定は次のとおりです。

* アプリと要求のソースの間には、"*1 つのプロキシ*" だけが存在します。
* 既知のプロキシと既知のネットワークに対しては、ループバック アドレスのみが構成されています。
* 転送されるヘッダーには `X-Forwarded-For` と `X-Forwarded-Proto` の名前が付けられます。

すべてのネットワーク アプライアンスが追加構成なしで `X-Forwarded-For` および `X-Forwarded-Proto` ヘッダーを追加するわけではありません。 プロキシされた要求がアプリに届いたときにこれらのヘッダーが含まれていない場合は、アプライアンスの製造元のガイダンスを参照してください。 アプライアンスが `X-Forwarded-For` と `X-Forwarded-Proto` 以外の名前を使用している場合は、アプライアンスで使用されるヘッダー名と一致するように <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedForHeaderName> と <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedProtoHeaderName> のオプションを設定してください。 詳細については、「[Forwarded Headers Middleware のオプション](#forwarded-headers-middleware-options)」と「[別のヘッダー名を使用するプロキシの構成](#configuration-for-a-proxy-that-uses-different-header-names)」を参照してください。

## <a name="iisiis-express-and-aspnet-core-module"></a>IIS/IIS Express と ASP.NET Core モジュール

アプリが IIS と ASP.NET Core モジュールの背後で[アウト プロセス](xref:host-and-deploy/iis/index#out-of-process-hosting-model)でホストされている場合、[IIS Integration Middleware](xref:host-and-deploy/iis/index#enable-the-iisintegration-components) によって Forwarded Headers Middleware が既定で有効にされます。 転送されるヘッダーの信頼に関する問題のため (たとえば、[IP スプーフィング](https://www.iplocation.net/ip-spoofing))、Forwarded Headers Middleware はアクティブ化されると最初に、ASP.NET Core モジュールに固有の制限された構成を使って、ミドルウェア パイプライン内で実行します。 ミドルウェアは、`X-Forwarded-For` および `X-Forwarded-Proto` ヘッダーを転送するように構成され、単一の localhost プロキシに制限されます。 追加の構成が必要な場合は、「[Forwarded Headers Middleware のオプション](#forwarded-headers-middleware-options)」をご覧ください。

## <a name="other-proxy-server-and-load-balancer-scenarios"></a>プロキシ サーバーとロード バランサーの他のシナリオ

[アウト プロセス](xref:host-and-deploy/iis/index#out-of-process-hosting-model)でホストされている場合に [IIS Integration](xref:host-and-deploy/iis/index#enable-the-iisintegration-components) が使われていない場合は、Forwarded Headers Middleware は既定で有効になりません。 <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*> を含む転送されたヘッダーをアプリで処理するためには、Forwarded Headers Middleware を有効にする必要があります。 ミドルウェアを有効にした後、ミドルウェアに対して <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> が指定されていない場合の既定の [ForwardedHeadersOptions.ForwardedHeaders](xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHeaders) は [ForwardedHeaders.None](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeaders) です。

<xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> でミドルウェアを構成して、`Startup.ConfigureServices` で `X-Forwarded-For` および `X-Forwarded-Proto` ヘッダーを転送します。

<a name="fhmo"></a>

### <a name="forwarded-headers-middleware-order"></a>Forwarded Headers Middleware の順序

Forwarded Headers Middleware は、他のミドルウェアの前に実行する必要があります。 この順序により、転送されるヘッダー情報に依存するミドルウェアが処理にヘッダー値を使用できます。 Forwarded Headers Middleware は、診断とエラー処理の後に実行できますが、`UseHsts` を呼び出す前に実行する必要があります。

[!code-csharp[](~/host-and-deploy/proxy-load-balancer/3.1samples/Startup.cs?name=snippet&highlight=13-17,25,30)]

または、診断の前に `UseForwardedHeaders` を呼び出します。

[!code-csharp[](~/host-and-deploy/proxy-load-balancer/3.1samples/Startup2.cs?name=snippet)]

> [!NOTE]
> <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> が `Startup.ConfigureServices` において指定されていない場合、または <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*> を使って拡張メソッドに直接渡されない場合、転送される既定のヘッダーは [ForwardedHeaders.None](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeaders) です。 転送するヘッダーで <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHeaders> プロパティが構成されている必要があります。

## <a name="nginx-configuration"></a>Nginx の構成

`X-Forwarded-For` および `X-Forwarded-Proto` ヘッダーを転送する場合は、<xref:host-and-deploy/linux-nginx#configure-nginx> を参照してください。 詳細については、「[NGINX:Using the Forwarded header](https://www.nginx.com/resources/wiki/start/topics/examples/forwarded/)」 (NGINX: 転送されるヘッダーの使用) を参照してください。

## <a name="apache-configuration"></a>Apache の構成

`X-Forwarded-For` は自動的に追加されます (「[Apache Module mod_proxy:Reverse Proxy Request Headers](https://httpd.apache.org/docs/2.4/mod/mod_proxy.html#x-headers)」 (Apache モジュール mod_proxy: リバース プロキシがヘッダーを要求する) を参照)。 `X-Forwarded-Proto` ヘッダーを転送する方法については、<xref:host-and-deploy/linux-apache#configure-apache> を参照してください。

## <a name="forwarded-headers-middleware-options"></a>Forwarded Headers Middleware のオプション

<xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> は、Forwarded Headers Middleware の動作を制御します。 既定の値の変更例を次に示します。

* 転送されるヘッダーのエントリ数を `2` に制限します。
* `127.0.10.1` の既知のプロキシ アドレスを追加します。
* 転送されるヘッダーの名前を既定の `X-Forwarded-For` から `X-Forwarded-For-My-Custom-Header-Name` に変更します。

```csharp
services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardLimit = 2;
    options.KnownProxies.Add(IPAddress.Parse("127.0.10.1"));
    options.ForwardedForHeaderName = "X-Forwarded-For-My-Custom-Header-Name";
});
```

| オプション | 説明 |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.AllowedHosts> | `X-Forwarded-Host` ヘッダーによるホストを指定した値に制限します。<ul><li>値は、大文字と小文字を無視して序数で比較されます。</li><li>ポート番号は除外されている必要があります。</li><li>リストが空の場合、すべてのホストが許可されます。</li><li>最上位のワイルドカード `*` は、すべての空でないホストを許可します。</li><li>サブドメインのワイルドカードは許可されますが、ルート ドメインとは一致しません。 たとえば、`*.contoso.com` はサブドメイン `foo.contoso.com` と一致しますが、ルート ドメイン `contoso.com` とは一致しません。</li><li>Unicode のホスト名は許可されますが、一致のために [Punycode](https://tools.ietf.org/html/rfc3492) に変換されます。</li><li>[IPv6 アドレス](https://tools.ietf.org/html/rfc4291)は境界の角かっこを含む必要があり、[従来の形式](https://tools.ietf.org/html/rfc4291#section-2.2) (例: `[ABCD:EF01:2345:6789:ABCD:EF01:2345:6789]`) になっている必要があります。 IPv6 アドレスは、異なる形式間で論理的な等価性を調べるために特別な大文字小文字にはされず、正規化は行われません。</li><li>許可されるホストの制限に失敗すると、サービスによって生成されたリンクを攻撃者が偽装する可能性があります。</li></ul>既定値は空の `IList<string>` です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedForHeaderName> | [ForwardedHeadersDefaults.XForwardedForHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XForwardedForHeaderName) によって指定されているヘッダーではなく、このプロパティによって指定されているヘッダーを使います。 このオプションは、プロキシ/フォワーダーが `X-Forwarded-For` ヘッダーを使用していないが、情報の転送のためにその他のヘッダーを使用している場合に使用されます。<br><br>既定値は、`X-Forwarded-For` です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHeaders> | 処理する必要があるフォワーダーを識別します。 適用されるフィールドの一覧については、「[ForwardedHeaders Enum](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeaders)」(ForwardedHeaders 列挙型) をご覧ください。 このプロパティに割り当てられる標準値は、`ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto` です。<br><br>既定値は [ForwardedHeaders.None](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeaders) です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHostHeaderName> | [ForwardedHeadersDefaults.XForwardedHostHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XForwardedHostHeaderName) によって指定されているヘッダーではなく、このプロパティによって指定されているヘッダーを使います。 このオプションは、プロキシ/フォワーダーが `X-Forwarded-Host` ヘッダーを使用していないが、情報の転送のためにその他のヘッダーを使用している場合に使用されます。<br><br>既定値は、`X-Forwarded-Host` です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedProtoHeaderName> | [ForwardedHeadersDefaults.XForwardedProtoHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XForwardedProtoHeaderName) によって指定されているヘッダーではなく、このプロパティによって指定されているヘッダーを使います。 このオプションは、プロキシ/フォワーダーが `X-Forwarded-Proto` ヘッダーを使用していないが、情報の転送のためにその他のヘッダーを使用している場合に使用されます。<br><br>既定値は、`X-Forwarded-Proto` です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardLimit> | 処理されるヘッダー内のエントリの数を制限します。 制限を無効にするには `null` に設定しますが、これは `KnownProxies` または `KnownNetworks` が構成されている場合にのみ行う必要があります。 `null` 以外の値を設定することは、正しく構成されていないプロキシや、ネットワーク上のサイドチャネルから届く悪意のある要求から保護するための予防策となります (ただし保証はできません)。<br><br>Forwarded Headers Middleware では、ヘッダーが右から左の逆の順序で処理されます。 規定値 (`1`) を使う場合は、`ForwardLimit` の値を増やさない限りヘッダーの右端にある値のみが処理されます。<br><br>既定値は、`1` です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.KnownNetworks> | 受け付けるヘッダーの転送元である既知のネットワークのアドレス範囲です。 クラスレス ドメイン間ルーティング (CIDR) の表記を使って、IP の範囲を指定します。<br><br>サーバーがデュアル モードのソケットを使用している場合、IPv4 アドレスは IPv6 形式で提供されます (たとえば、IPv4 での `10.0.0.1` は IPv6 で `::ffff:10.0.0.1` と表現されます)。 「[IPAddress.MapToIPv6](xref:System.Net.IPAddress.MapToIPv6*)」をご覧ください。 この形式が必要かどうかを判断するには、「[HttpContext.Connection.RemoteIpAddress](xref:Microsoft.AspNetCore.Http.ConnectionInfo.RemoteIpAddress*)」をご覧ください。<br><br>既定値は、`IPAddress.Loopback` の単一エントリを含む `IList`\<<xref:Microsoft.AspNetCore.HttpOverrides.IPNetwork>> です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.KnownProxies> | 受け付けるヘッダーの転送元である既知のプロキシのアドレスです。 IP アドレスの正確な一致を指定するには、`KnownProxies` を使います。<br><br>サーバーがデュアル モードのソケットを使用している場合、IPv4 アドレスは IPv6 形式で提供されます (たとえば、IPv4 での `10.0.0.1` は IPv6 で `::ffff:10.0.0.1` と表現されます)。 「[IPAddress.MapToIPv6](xref:System.Net.IPAddress.MapToIPv6*)」をご覧ください。 この形式が必要かどうかを判断するには、「[HttpContext.Connection.RemoteIpAddress](xref:Microsoft.AspNetCore.Http.ConnectionInfo.RemoteIpAddress*)」をご覧ください。<br><br>既定値は、`IPAddress.IPv6Loopback` の単一エントリを含む `IList`\<<xref:System.Net.IPAddress>> です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.OriginalForHeaderName> | [ForwardedHeadersDefaults.XOriginalForHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XOriginalForHeaderName) によって指定されているヘッダーではなく、このプロパティによって指定されているヘッダーを使います。<br><br>既定値は、`X-Original-For` です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.OriginalHostHeaderName> | [ForwardedHeadersDefaults.XOriginalHostHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XOriginalHostHeaderName) によって指定されているヘッダーではなく、このプロパティによって指定されているヘッダーを使います。<br><br>既定値は、`X-Original-Host` です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.OriginalProtoHeaderName> | [ForwardedHeadersDefaults.XOriginalProtoHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XOriginalProtoHeaderName) によって指定されているヘッダーではなく、このプロパティによって指定されているヘッダーを使います。<br><br>既定値は、`X-Original-Proto` です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.RequireHeaderSymmetry> | 処理対象の [ForwardedHeadersOptions.ForwardedHeaders](xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHeaders) の間でヘッダー値の数が同期していることを要求します。<br><br>ASP.NET Core 1.x での既定値は `true` です。 ASP.NET Core 2.0 以降での既定値は `false` です。 |

## <a name="scenarios-and-use-cases"></a>シナリオとユース ケース

### <a name="when-it-isnt-possible-to-add-forwarded-headers-and-all-requests-are-secure"></a>転送されるヘッダーを追加することができず、すべての要求が安全な場合

アプリにプロキシされる要求に、転送されるヘッダーを追加できない場合があります。 すべてのパブリック外部要求が HTTPS を使うことをプロキシが強制している場合、すべての種類のミドルウェアを使う前に、`Startup.Configure` でスキームを手動で設定できます。

```csharp
app.Use((context, next) =>
{
    context.Request.Scheme = "https";
    return next();
});
```

このコードは、開発環境またはステージング環境の環境変数または他の構成設定によって、無効にすることができます。

### <a name="deal-with-path-base-and-proxies-that-change-the-request-path"></a>要求のパスを変更するパス ベースとプロキシを処理する

一部のプロキシでは、パスはそのまま渡されますが、ルーティングが正しく機能するためには削除する必要のあるアプリ ベース パスが含まれます。 [UsePathBaseExtensions.UsePathBase](xref:Microsoft.AspNetCore.Builder.UsePathBaseExtensions.UsePathBase*) ミドルウェアは、パスを [HttpRequest.Path](xref:Microsoft.AspNetCore.Http.HttpRequest.Path) と [HttpRequest.PathBase](xref:Microsoft.AspNetCore.Http.HttpRequest.PathBase) へのアプリ ベース パスに分割します。

`/foo` が `/foo/api/1` として渡されるプロキシ パスのアプリ ベース パスである場合、ミドルウェアは次のコマンドで `Request.PathBase` を `/foo`に、`Request.Path` を `/api/1` に設定します。

```csharp
app.UsePathBase("/foo");
```

ミドルウェアが逆方向にもう一度呼び出されると、元のパスとパス ベースが再度適用されます。 ミドルウェアの処理の順序について詳しくは、<xref:fundamentals/middleware/index> を参照してください。

プロキシがパスをトリミングする場合は (たとえば、`/foo/api/1` を `/api/1` に転送)、要求の [PathBase](xref:Microsoft.AspNetCore.Http.HttpRequest.PathBase) プロパティを設定することによって、リダイレクトとリンクを修正します。

```csharp
app.Use((context, next) =>
{
    context.Request.PathBase = new PathString("/foo");
    return next();
});
```

プロキシがパス データを追加している場合は、<xref:Microsoft.AspNetCore.Http.PathString.StartsWithSegments*> を使用して <xref:Microsoft.AspNetCore.Http.HttpRequest.Path> プロパティに割り当てることで、パスの一部を破棄してリダイレクトとリンクを修正します。

```csharp
app.Use((context, next) =>
{
    if (context.Request.Path.StartsWithSegments("/foo", out var remainder))
    {
        context.Request.Path = remainder;
    }

    return next();
});
```

### <a name="configuration-for-a-proxy-that-uses-different-header-names"></a>別のヘッダー名を使用するプロキシの構成

プロキシがプロキシ アドレス/ポートおよび開始スキーム情報の転送に名前が `X-Forwarded-For` と `X-Forwarded-Proto` のヘッダーを使用しない場合、プロキシで使用されるヘッダー名と一致するように、<xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedForHeaderName> と <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedProtoHeaderName> のオプションを設定してください。

```csharp
services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedForHeaderName = "Header_Name_Used_By_Proxy_For_X-Forwarded-For_Header";
    options.ForwardedProtoHeaderName = "Header_Name_Used_By_Proxy_For_X-Forwarded-Proto_Header";
});
```

## <a name="forward-the-scheme-for-linux-and-non-iis-reverse-proxies"></a>Linux および非 IIS のリバース プロキシのスキームを転送する

<xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection*> および <xref:Microsoft.AspNetCore.Builder.HstsBuilderExtensions.UseHsts*> を呼び出すアプリが、Azure Linux App Service、Azure Linux 仮想マシン (VM)、または IIS 以外の他のリバース プロキシの背後に展開されている場合、サイトは無限ループに陥ります。 TLS はリバース プロキシによって終了され、Kestrel では正しい要求スキームが認識されません。 OAuth と OIDC もこの構成では正しくないリダイレクトを生成するため、失敗します。 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration*> は IIS の背後で実行される場合、Forwarded Headers Middleware を追加して構成しますが、Linux 用の対応する自動設定 (Apache または Nginx 統合) はありません。

非 IIS シナリオでプロキシからスキームを転送するには、Forwarded Headers Middleware を追加して構成します。 `Startup.ConfigureServices` で、次のコードを使用します。

```csharp
// using Microsoft.AspNetCore.HttpOverrides;

if (string.Equals(
    Environment.GetEnvironmentVariable("ASPNETCORE_FORWARDEDHEADERS_ENABLED"), 
    "true", StringComparison.OrdinalIgnoreCase))
{
    services.Configure<ForwardedHeadersOptions>(options =>
    {
        options.ForwardedHeaders = ForwardedHeaders.XForwardedFor | 
            ForwardedHeaders.XForwardedProto;
        // Only loopback proxies are allowed by default.
        // Clear that restriction because forwarders are enabled by explicit 
        // configuration.
        options.KnownNetworks.Clear();
        options.KnownProxies.Clear();
    });
}
```

## <a name="certificate-forwarding"></a>証明書の転送 

### <a name="azure"></a>Azure

証明書の転送用に Azure App Service を構成するには、「[Azure App Service に対する TLS 相互認証の構成](/azure/app-service/app-service-web-configure-tls-mutual-auth)」を参照してください。 次のガイダンスは、ASP.NET Core アプリの構成に関するものです。

`Startup.Configure` で、`app.UseAuthentication();` の呼び出しの前に次のコードを追加します。

```csharp
app.UseCertificateForwarding();
```


Azure で使用されるヘッダー名を指定するように、証明書転送ミドルウェアを構成します。 `Startup.ConfigureServices` に次のコードを追加し、ミドルウェアによる証明書作成の基になるヘッダーを構成します。

```csharp
services.AddCertificateForwarding(options =>
    options.CertificateHeader = "X-ARR-ClientCert");
```

### <a name="other-web-proxies"></a>他の Web プロキシ

使用されているプロキシが、IIS でも、Azure App Service のアプリケーション要求ルーティング処理 (ARR) でもない場合は、HTTP ヘッダーで受け取った証明書を転送するように、プロキシを構成します。 `Startup.Configure` で、`app.UseAuthentication();` の呼び出しの前に次のコードを追加します。

```csharp
app.UseCertificateForwarding();
```

ヘッダー名を指定するように証明書の転送ミドルウェアを構成します。 `Startup.ConfigureServices` に次のコードを追加し、ミドルウェアによる証明書作成の基になるヘッダーを構成します。

```csharp
services.AddCertificateForwarding(options =>
    options.CertificateHeader = "YOUR_CERTIFICATE_HEADER_NAME");
```

プロキシで証明書が base64 エンコードではない場合は (Nginx の場合と同様)、`HeaderConverter` オプションを設定します。 `Startup.ConfigureServices` での次の例を検討してください。

```csharp
services.AddCertificateForwarding(options =>
{
    options.CertificateHeader = "YOUR_CUSTOM_HEADER_NAME";
    options.HeaderConverter = (headerValue) => 
    {
        var clientCertificate = 
           /* some conversion logic to create an X509Certificate2 */
        return clientCertificate;
    }
});
```

## <a name="troubleshoot"></a>トラブルシューティング

ヘッダーが意図したとおりに転送されない場合は、[ログ](xref:fundamentals/logging/index) を有効にします。 ログで問題のトラブルシューティングに十分な情報が提供されない場合は、サーバーが受信した要求ヘッダーを列挙します。 インライン ミドルウェアを使用し、アプリ応答に要求ヘッダーを書き込んだり、ヘッダーをログに記録したりします。 

アプリの応答にヘッダーを書き込むには、`Startup.Configure` 内の <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*> の呼び出しの直後に、次の端末のインライン ミドルウェアを配置します。

```csharp
app.Run(async (context) =>
{
    context.Response.ContentType = "text/plain";

    // Request method, scheme, and path
    await context.Response.WriteAsync(
        $"Request Method: {context.Request.Method}{Environment.NewLine}");
    await context.Response.WriteAsync(
        $"Request Scheme: {context.Request.Scheme}{Environment.NewLine}");
    await context.Response.WriteAsync(
        $"Request Path: {context.Request.Path}{Environment.NewLine}");

    // Headers
    await context.Response.WriteAsync($"Request Headers:{Environment.NewLine}");

    foreach (var header in context.Request.Headers)
    {
        await context.Response.WriteAsync($"{header.Key}: " +
            $"{header.Value}{Environment.NewLine}");
    }

    await context.Response.WriteAsync(Environment.NewLine);

    // Connection: RemoteIp
    await context.Response.WriteAsync(
        $"Request RemoteIp: {context.Connection.RemoteIpAddress}");
});
```

応答本文ではなく、ログに書き込むことができます。 ログに書き込むことで、デバッグしている間、サイトは正常に機能できます。

応答本文ではなく、ログに書き込むには:

* 「[Startup でログを作成する](xref:fundamentals/logging/index#create-logs-in-startup)」に説明されているように、`ILogger<Startup>` を `Startup` クラスに挿入します。
* `Startup.Configure` 内で <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*> を呼び出した直後に次のインライン ミドルウェアを配置します。

```csharp
app.Use(async (context, next) =>
{
    // Request method, scheme, and path
    _logger.LogDebug("Request Method: {Method}", context.Request.Method);
    _logger.LogDebug("Request Scheme: {Scheme}", context.Request.Scheme);
    _logger.LogDebug("Request Path: {Path}", context.Request.Path);

    // Headers
    foreach (var header in context.Request.Headers)
    {
        _logger.LogDebug("Header: {Key}: {Value}", header.Key, header.Value);
    }

    // Connection: RemoteIp
    _logger.LogDebug("Request RemoteIp: {RemoteIpAddress}", 
        context.Connection.RemoteIpAddress);

    await next();
});
```

処理時、`X-Forwarded-{For|Proto|Host}` 値は `X-Original-{For|Proto|Host}` に移動されます。 指定されたヘッダーに複数の値がある場合、Forwarded Headers Middleware では右から左の逆の順序でヘッダーが処理されます。 既定の `ForwardLimit` は `1` です。そのため、`ForwardLimit` の値を増やさない限り、ヘッダーの右端にある値のみが処理されます。

Forwarded Headers が処理される前に、要求の元のリモート IP アドレスが `KnownProxies` リストまたは `KnownNetworks` リストのエントリと一致する必要があります。 これにより、信頼されないプロキシからの転送を受け付けないことで、ヘッダーのスプーフィングを制限します。 不明なプロキシが検出されると、ログにプロキシのアドレスが示されます。

```console
September 20th 2018, 15:49:44.168 Unknown proxy: 10.0.0.100:54321
```

前の例では、10.0.0.100 がプロキシ サーバーです。 サーバーが信頼されているプロキシであれば、`Startup.ConfigureServices` で `KnownProxies` にサーバーの IP アドレスを追加します (あるいは、信頼されているネットワークを `KnownNetworks` に追加します)。 詳細については、「[Forwarded Headers Middleware のオプション](#forwarded-headers-middleware-options)」セクションをご覧ください。

```csharp
services.Configure<ForwardedHeadersOptions>(options =>
{
    options.KnownProxies.Add(IPAddress.Parse("10.0.0.100"));
});
```

> [!IMPORTANT]
> 信頼されているプロキシとネットワークにのみ、ヘッダーの転送を許可します。 それ以外に許可すると、[IP なりすまし](https://www.iplocation.net/ip-spoofing)攻撃が可能になります。

## <a name="additional-resources"></a>その他の技術情報

* <xref:host-and-deploy/web-farm>
* [Microsoft セキュリティ アドバイザリ CVE-2018-0787:ASP.NET Core の特権の昇格脆弱性](https://github.com/aspnet/Announcements/issues/295)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core の推奨される構成では、アプリは IIS/ASP.NET Core モジュール、Nginx、または Apache を使ってホストされます。 プロキシ サーバー、ロード バランサー、および他のネットワーク アプライアンスにより、要求に関する情報が、アプリに到達する前にわからなくなることがよくあります。

* HTTPS 要求が HTTP によってプロキシされると、元のスキーム (HTTPS) は失われ、ヘッダーで転送される必要があります。
* アプリは、インターネット上または社内ネットワーク上の本来の送信元ではなく、プロキシから要求を受信するため、送信元クライアントの IP アドレスもヘッダーで転送される必要があります。

この情報は、リダイレクト、認証、リンクの生成、ポリシーの評価、クライアントの位置情報など、要求の処理で重要になる可能性があります。

## <a name="forwarded-headers"></a>転送されるヘッダー

慣例によりは、プロキシは HTTP ヘッダーで情報を転送します。

| Header | 説明 |
| ------ | ----------- |
| X-Forwarded-For | 要求を開始したクライアントと、プロキシ チェーン内の後続のプロキシに関する情報を保持します。 このパラメーターは、IP アドレス (および、必要に応じてポート番号) を含むことがあります。 プロキシ サーバーのチェーンにおいては、最初のパラメーターが、要求を最初に行ったクライアントを示します。 その後に、後続のプロキシの識別子が指定されています。 チェーン内の最後のプロキシは、パラメーターのリストには含まれません。 最後のプロキシの IP アドレスとオプションのポート番号は、トランスポート層のリモート IP アドレスとして入手できます。 |
| X-Forwarded-Proto | 開始スキーム (HTTP/HTTPS) の値です。 要求が複数のプロキシを通過している場合、値はスキームのリストである場合もあります。 |
| X-Forwarded-Host | ホスト ヘッダー フィールドの元の値です。 通常、プロキシはホスト ヘッダーを変更しません。 プロキシにおいてホスト ヘッダーが既知の適切な値であることが検証されない、または既知の適切な値に制限されないシステムに影響を与える、特権の昇格脆弱性については、[マイクロソフト セキュリティ アドバイザリ CVE-2018-0787](https://github.com/aspnet/Announcements/issues/295) をご覧ください。 |

[Microsoft.AspNetCore.HttpOverrides](https://www.nuget.org/packages/Microsoft.AspNetCore.HttpOverrides/) パッケージの Forwarded Headers Middleware は、これらのヘッダーを読み取り、<xref:Microsoft.AspNetCore.Http.HttpContext> の関連するフィールドに入力します。

ミドルウェアの更新:

* [HttpContext.Connection.RemoteIpAddress](xref:Microsoft.AspNetCore.Http.ConnectionInfo.RemoteIpAddress):`X-Forwarded-For` ヘッダー値を使用して設定します。 追加の設定は、ミドルウェアが `RemoteIpAddress` を設定する方法に影響を与えます。 詳しくは、「[Forwarded Headers Middleware のオプション](#forwarded-headers-middleware-options)」をご覧ください。
* [HttpContext.Request.Scheme](xref:Microsoft.AspNetCore.Http.HttpRequest.Scheme):`X-Forwarded-Proto` ヘッダー値を使用して設定します。
* [HttpContext.Request.Host](xref:Microsoft.AspNetCore.Http.HttpRequest.Host):`X-Forwarded-Host` ヘッダー値を使用して設定します。

Forwarded Headers Middleware の[既定の設定](#forwarded-headers-middleware-options)は構成できます。 既定の設定は次のとおりです。

* アプリと要求のソースの間には、"*1 つのプロキシ*" だけが存在します。
* 既知のプロキシと既知のネットワークに対しては、ループバック アドレスのみが構成されています。
* 転送されるヘッダーには `X-Forwarded-For` と `X-Forwarded-Proto` の名前が付けられます。

すべてのネットワーク アプライアンスが追加構成なしで `X-Forwarded-For` および `X-Forwarded-Proto` ヘッダーを追加するわけではありません。 プロキシされた要求がアプリに届いたときにこれらのヘッダーが含まれていない場合は、アプライアンスの製造元のガイダンスを参照してください。 アプライアンスが `X-Forwarded-For` と `X-Forwarded-Proto` 以外の名前を使用している場合は、アプライアンスで使用されるヘッダー名と一致するように <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedForHeaderName> と <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedProtoHeaderName> のオプションを設定してください。 詳細については、「[Forwarded Headers Middleware のオプション](#forwarded-headers-middleware-options)」と「[別のヘッダー名を使用するプロキシの構成](#configuration-for-a-proxy-that-uses-different-header-names)」を参照してください。

## <a name="iisiis-express-and-aspnet-core-module"></a>IIS/IIS Express と ASP.NET Core モジュール

アプリが IIS と ASP.NET Core モジュールの背後で[アウト プロセス](xref:host-and-deploy/iis/index#out-of-process-hosting-model)でホストされている場合、[IIS Integration Middleware](xref:host-and-deploy/iis/index#enable-the-iisintegration-components) によって Forwarded Headers Middleware が既定で有効にされます。 転送されるヘッダーの信頼に関する問題のため (たとえば、[IP スプーフィング](https://www.iplocation.net/ip-spoofing))、Forwarded Headers Middleware はアクティブ化されると最初に、ASP.NET Core モジュールに固有の制限された構成を使って、ミドルウェア パイプライン内で実行します。 ミドルウェアは、`X-Forwarded-For` および `X-Forwarded-Proto` ヘッダーを転送するように構成され、単一の localhost プロキシに制限されます。 追加の構成が必要な場合は、「[Forwarded Headers Middleware のオプション](#forwarded-headers-middleware-options)」をご覧ください。

## <a name="other-proxy-server-and-load-balancer-scenarios"></a>プロキシ サーバーとロード バランサーの他のシナリオ

[アウト プロセス](xref:host-and-deploy/iis/index#out-of-process-hosting-model)でホストされている場合に [IIS Integration](xref:host-and-deploy/iis/index#enable-the-iisintegration-components) が使われていない場合は、Forwarded Headers Middleware は既定で有効になりません。 <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*> を含む転送されたヘッダーをアプリで処理するためには、Forwarded Headers Middleware を有効にする必要があります。 ミドルウェアを有効にした後、ミドルウェアに対して <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> が指定されていない場合の既定の [ForwardedHeadersOptions.ForwardedHeaders](xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHeaders) は [ForwardedHeaders.None](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeaders) です。

<xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> でミドルウェアを構成して、`Startup.ConfigureServices` で `X-Forwarded-For` および `X-Forwarded-Proto` ヘッダーを転送します。 他のミドルウェアを呼び出す前に、`Startup.Configure` 内で <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*> メソッドを呼び出します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();

    services.Configure<ForwardedHeadersOptions>(options =>
    {
        options.ForwardedHeaders = 
            ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto;
    });
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseForwardedHeaders();

    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseExceptionHandler("/Home/Error");
    }

    app.UseStaticFiles();
    // In ASP.NET Core 1.x, replace the following line with: app.UseIdentity();
    app.UseAuthentication();
    app.UseMvc();
}
```

> [!NOTE]
> <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> が `Startup.ConfigureServices` において指定されていない場合、または <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*> を使って拡張メソッドに直接渡されない場合、転送される既定のヘッダーは [ForwardedHeaders.None](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeaders) です。 転送するヘッダーで <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHeaders> プロパティが構成されている必要があります。

## <a name="nginx-configuration"></a>Nginx の構成

`X-Forwarded-For` および `X-Forwarded-Proto` ヘッダーを転送する場合は、<xref:host-and-deploy/linux-nginx#configure-nginx> を参照してください。 詳細については、「[NGINX:Using the Forwarded header](https://www.nginx.com/resources/wiki/start/topics/examples/forwarded/)」 (NGINX: 転送されるヘッダーの使用) を参照してください。

## <a name="apache-configuration"></a>Apache の構成

`X-Forwarded-For` は自動的に追加されます (「[Apache Module mod_proxy:Reverse Proxy Request Headers](https://httpd.apache.org/docs/2.4/mod/mod_proxy.html#x-headers)」 (Apache モジュール mod_proxy: リバース プロキシがヘッダーを要求する) を参照)。 `X-Forwarded-Proto` ヘッダーを転送する方法については、<xref:host-and-deploy/linux-apache#configure-apache> を参照してください。

## <a name="forwarded-headers-middleware-options"></a>Forwarded Headers Middleware のオプション

<xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> は、Forwarded Headers Middleware の動作を制御します。 既定の値の変更例を次に示します。

* 転送されるヘッダーのエントリ数を `2` に制限します。
* `127.0.10.1` の既知のプロキシ アドレスを追加します。
* 転送されるヘッダーの名前を既定の `X-Forwarded-For` から `X-Forwarded-For-My-Custom-Header-Name` に変更します。

```csharp
services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardLimit = 2;
    options.KnownProxies.Add(IPAddress.Parse("127.0.10.1"));
    options.ForwardedForHeaderName = "X-Forwarded-For-My-Custom-Header-Name";
});
```

| オプション | 説明 |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.AllowedHosts> | `X-Forwarded-Host` ヘッダーによるホストを指定した値に制限します。<ul><li>値は、大文字と小文字を無視して序数で比較されます。</li><li>ポート番号は除外されている必要があります。</li><li>リストが空の場合、すべてのホストが許可されます。</li><li>最上位のワイルドカード `*` は、すべての空でないホストを許可します。</li><li>サブドメインのワイルドカードは許可されますが、ルート ドメインとは一致しません。 たとえば、`*.contoso.com` はサブドメイン `foo.contoso.com` と一致しますが、ルート ドメイン `contoso.com` とは一致しません。</li><li>Unicode のホスト名は許可されますが、一致のために [Punycode](https://tools.ietf.org/html/rfc3492) に変換されます。</li><li>[IPv6 アドレス](https://tools.ietf.org/html/rfc4291)は境界の角かっこを含む必要があり、[従来の形式](https://tools.ietf.org/html/rfc4291#section-2.2) (例: `[ABCD:EF01:2345:6789:ABCD:EF01:2345:6789]`) になっている必要があります。 IPv6 アドレスは、異なる形式間で論理的な等価性を調べるために特別な大文字小文字にはされず、正規化は行われません。</li><li>許可されるホストの制限に失敗すると、サービスによって生成されたリンクを攻撃者が偽装する可能性があります。</li></ul>既定値は空の `IList<string>` です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedForHeaderName> | [ForwardedHeadersDefaults.XForwardedForHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XForwardedForHeaderName) によって指定されているヘッダーではなく、このプロパティによって指定されているヘッダーを使います。 このオプションは、プロキシ/フォワーダーが `X-Forwarded-For` ヘッダーを使用していないが、情報の転送のためにその他のヘッダーを使用している場合に使用されます。<br><br>既定値は、`X-Forwarded-For` です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHeaders> | 処理する必要があるフォワーダーを識別します。 適用されるフィールドの一覧については、「[ForwardedHeaders Enum](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeaders)」(ForwardedHeaders 列挙型) をご覧ください。 このプロパティに割り当てられる標準値は、`ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto` です。<br><br>既定値は [ForwardedHeaders.None](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeaders) です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHostHeaderName> | [ForwardedHeadersDefaults.XForwardedHostHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XForwardedHostHeaderName) によって指定されているヘッダーではなく、このプロパティによって指定されているヘッダーを使います。 このオプションは、プロキシ/フォワーダーが `X-Forwarded-Host` ヘッダーを使用していないが、情報の転送のためにその他のヘッダーを使用している場合に使用されます。<br><br>既定値は、`X-Forwarded-Host` です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedProtoHeaderName> | [ForwardedHeadersDefaults.XForwardedProtoHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XForwardedProtoHeaderName) によって指定されているヘッダーではなく、このプロパティによって指定されているヘッダーを使います。 このオプションは、プロキシ/フォワーダーが `X-Forwarded-Proto` ヘッダーを使用していないが、情報の転送のためにその他のヘッダーを使用している場合に使用されます。<br><br>既定値は、`X-Forwarded-Proto` です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardLimit> | 処理されるヘッダー内のエントリの数を制限します。 制限を無効にするには `null` に設定しますが、これは `KnownProxies` または `KnownNetworks` が構成されている場合にのみ行う必要があります。 `null` 以外の値を設定することは、正しく構成されていないプロキシや、ネットワーク上のサイドチャネルから届く悪意のある要求から保護するための予防策となります (ただし保証はできません)。<br><br>Forwarded Headers Middleware では、ヘッダーが右から左の逆の順序で処理されます。 規定値 (`1`) を使う場合は、`ForwardLimit` の値を増やさない限りヘッダーの右端にある値のみが処理されます。<br><br>既定値は、`1` です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.KnownNetworks> | 受け付けるヘッダーの転送元である既知のネットワークのアドレス範囲です。 クラスレス ドメイン間ルーティング (CIDR) の表記を使って、IP の範囲を指定します。<br><br>サーバーがデュアル モードのソケットを使用している場合、IPv4 アドレスは IPv6 形式で提供されます (たとえば、IPv4 での `10.0.0.1` は IPv6 で `::ffff:10.0.0.1` と表現されます)。 「[IPAddress.MapToIPv6](xref:System.Net.IPAddress.MapToIPv6*)」をご覧ください。 この形式が必要かどうかを判断するには、「[HttpContext.Connection.RemoteIpAddress](xref:Microsoft.AspNetCore.Http.ConnectionInfo.RemoteIpAddress*)」をご覧ください。 詳細については、「[IPv6 アドレスとして表される IPv4 アドレスの構成](#configuration-for-an-ipv4-address-represented-as-an-ipv6-address)」セクションをご覧ください。<br><br>既定値は、`IPAddress.Loopback` の単一エントリを含む `IList`\<<xref:Microsoft.AspNetCore.HttpOverrides.IPNetwork>> です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.KnownProxies> | 受け付けるヘッダーの転送元である既知のプロキシのアドレスです。 IP アドレスの正確な一致を指定するには、`KnownProxies` を使います。<br><br>サーバーがデュアル モードのソケットを使用している場合、IPv4 アドレスは IPv6 形式で提供されます (たとえば、IPv4 での `10.0.0.1` は IPv6 で `::ffff:10.0.0.1` と表現されます)。 「[IPAddress.MapToIPv6](xref:System.Net.IPAddress.MapToIPv6*)」をご覧ください。 この形式が必要かどうかを判断するには、「[HttpContext.Connection.RemoteIpAddress](xref:Microsoft.AspNetCore.Http.ConnectionInfo.RemoteIpAddress*)」をご覧ください。 詳細については、「[IPv6 アドレスとして表される IPv4 アドレスの構成](#configuration-for-an-ipv4-address-represented-as-an-ipv6-address)」セクションをご覧ください。<br><br>既定値は、`IPAddress.IPv6Loopback` の単一エントリを含む `IList`\<<xref:System.Net.IPAddress>> です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.OriginalForHeaderName> | [ForwardedHeadersDefaults.XOriginalForHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XOriginalForHeaderName) によって指定されているヘッダーではなく、このプロパティによって指定されているヘッダーを使います。<br><br>既定値は、`X-Original-For` です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.OriginalHostHeaderName> | [ForwardedHeadersDefaults.XOriginalHostHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XOriginalHostHeaderName) によって指定されているヘッダーではなく、このプロパティによって指定されているヘッダーを使います。<br><br>既定値は、`X-Original-Host` です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.OriginalProtoHeaderName> | [ForwardedHeadersDefaults.XOriginalProtoHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XOriginalProtoHeaderName) によって指定されているヘッダーではなく、このプロパティによって指定されているヘッダーを使います。<br><br>既定値は、`X-Original-Proto` です。 |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.RequireHeaderSymmetry> | 処理対象の [ForwardedHeadersOptions.ForwardedHeaders](xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHeaders) の間でヘッダー値の数が同期していることを要求します。<br><br>ASP.NET Core 1.x での既定値は `true` です。 ASP.NET Core 2.0 以降での既定値は `false` です。 |

## <a name="scenarios-and-use-cases"></a>シナリオとユース ケース

### <a name="when-it-isnt-possible-to-add-forwarded-headers-and-all-requests-are-secure"></a>転送されるヘッダーを追加することができず、すべての要求が安全な場合

アプリにプロキシされる要求に、転送されるヘッダーを追加できない場合があります。 すべてのパブリック外部要求が HTTPS を使うことをプロキシが強制している場合、すべての種類のミドルウェアを使う前に、`Startup.Configure` でスキームを手動で設定できます。

```csharp
app.Use((context, next) =>
{
    context.Request.Scheme = "https";
    return next();
});
```

このコードは、開発環境またはステージング環境の環境変数または他の構成設定によって、無効にすることができます。

### <a name="deal-with-path-base-and-proxies-that-change-the-request-path"></a>要求のパスを変更するパス ベースとプロキシを処理する

一部のプロキシでは、パスはそのまま渡されますが、ルーティングが正しく機能するためには削除する必要のあるアプリ ベース パスが含まれます。 [UsePathBaseExtensions.UsePathBase](xref:Microsoft.AspNetCore.Builder.UsePathBaseExtensions.UsePathBase*) ミドルウェアは、パスを [HttpRequest.Path](xref:Microsoft.AspNetCore.Http.HttpRequest.Path) と [HttpRequest.PathBase](xref:Microsoft.AspNetCore.Http.HttpRequest.PathBase) へのアプリ ベース パスに分割します。

`/foo` が `/foo/api/1` として渡されるプロキシ パスのアプリ ベース パスである場合、ミドルウェアは次のコマンドで `Request.PathBase` を `/foo`に、`Request.Path` を `/api/1` に設定します。

```csharp
app.UsePathBase("/foo");
```

ミドルウェアが逆方向にもう一度呼び出されると、元のパスとパス ベースが再度適用されます。 ミドルウェアの処理の順序について詳しくは、<xref:fundamentals/middleware/index> を参照してください。

プロキシがパスをトリミングする場合は (たとえば、`/foo/api/1` を `/api/1` に転送)、要求の [PathBase](xref:Microsoft.AspNetCore.Http.HttpRequest.PathBase) プロパティを設定することによって、リダイレクトとリンクを修正します。

```csharp
app.Use((context, next) =>
{
    context.Request.PathBase = new PathString("/foo");
    return next();
});
```

プロキシがパス データを追加している場合は、<xref:Microsoft.AspNetCore.Http.PathString.StartsWithSegments*> を使用して <xref:Microsoft.AspNetCore.Http.HttpRequest.Path> プロパティに割り当てることで、パスの一部を破棄してリダイレクトとリンクを修正します。

```csharp
app.Use((context, next) =>
{
    if (context.Request.Path.StartsWithSegments("/foo", out var remainder))
    {
        context.Request.Path = remainder;
    }

    return next();
});
```

### <a name="configuration-for-a-proxy-that-uses-different-header-names"></a>別のヘッダー名を使用するプロキシの構成

プロキシがプロキシ アドレス/ポートおよび開始スキーム情報の転送に名前が `X-Forwarded-For` と `X-Forwarded-Proto` のヘッダーを使用しない場合、プロキシで使用されるヘッダー名と一致するように、<xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedForHeaderName> と <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedProtoHeaderName> のオプションを設定してください。

```csharp
services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedForHeaderName = "Header_Name_Used_By_Proxy_For_X-Forwarded-For_Header";
    options.ForwardedProtoHeaderName = "Header_Name_Used_By_Proxy_For_X-Forwarded-Proto_Header";
});
```

### <a name="configuration-for-an-ipv4-address-represented-as-an-ipv6-address"></a>IPv6 アドレスとして表される IPv4 アドレスの構成

サーバーがデュアル モードのソケットを使用している場合、IPv4 アドレスは IPv6 形式で提供されます (たとえば、IPv4 での `10.0.0.1` は IPv6 で `::ffff:10.0.0.1` または `::ffff:a00:1` と表現されます)。 「[IPAddress.MapToIPv6](xref:System.Net.IPAddress.MapToIPv6*)」をご覧ください。 この形式が必要かどうかを判断するには、「[HttpContext.Connection.RemoteIpAddress](xref:Microsoft.AspNetCore.Http.ConnectionInfo.RemoteIpAddress*)」をご覧ください。

次の例では、転送されるヘッダーを提供するネットワーク アドレスを、IPv6 形式の `KnownNetworks` リストに追加します。

IPv4 アドレス: `10.11.12.1/8`

変換された IPv6 アドレス: `::ffff:10.11.12.1`  
変換されたプレフィックス長:104

16 進数形式でアドレスを指定することもできます (`10.11.12.1` は IPv6 で `::ffff:0a0b:0c01` として表されます)。 IPv4 アドレスを IPv6 に変換するときは、CIDR プレフィックス長 (例では `8`) に 96 を足して、追加の `::ffff:` IPv6 プレフィックスを構成します (8 + 96 = 104)。 

```csharp
// To access IPNetwork and IPAddress, add the following namespaces:
// using System.Net;
// using Microsoft.AspNetCore.HttpOverrides;
services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedHeaders =
        ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto;
    options.KnownNetworks.Add(new IPNetwork(
        IPAddress.Parse("::ffff:10.11.12.1"), 104));
});
```

## <a name="forward-the-scheme-for-linux-and-non-iis-reverse-proxies"></a>Linux および非 IIS のリバース プロキシのスキームを転送する

<xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection*> および <xref:Microsoft.AspNetCore.Builder.HstsBuilderExtensions.UseHsts*> を呼び出すアプリが、Azure Linux App Service、Azure Linux 仮想マシン (VM)、または IIS 以外の他のリバース プロキシの背後に展開されている場合、サイトは無限ループに陥ります。 TLS はリバース プロキシによって終了され、Kestrel では正しい要求スキームが認識されません。 OAuth と OIDC もこの構成では正しくないリダイレクトを生成するため、失敗します。 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration*> は IIS の背後で実行される場合、Forwarded Headers Middleware を追加して構成しますが、Linux 用の対応する自動設定 (Apache または Nginx 統合) はありません。

非 IIS シナリオでプロキシからスキームを転送するには、Forwarded Headers Middleware を追加して構成します。 `Startup.ConfigureServices` で、次のコードを使用します。

```csharp
// using Microsoft.AspNetCore.HttpOverrides;

if (string.Equals(
    Environment.GetEnvironmentVariable("ASPNETCORE_FORWARDEDHEADERS_ENABLED"), 
    "true", StringComparison.OrdinalIgnoreCase))
{
    services.Configure<ForwardedHeadersOptions>(options =>
    {
        options.ForwardedHeaders = ForwardedHeaders.XForwardedFor | 
            ForwardedHeaders.XForwardedProto;
        // Only loopback proxies are allowed by default.
        // Clear that restriction because forwarders are enabled by explicit 
        // configuration.
        options.KnownNetworks.Clear();
        options.KnownProxies.Clear();
    });
}
```

## <a name="troubleshoot"></a>トラブルシューティング

ヘッダーが意図したとおりに転送されない場合は、[ログ](xref:fundamentals/logging/index) を有効にします。 ログで問題のトラブルシューティングに十分な情報が提供されない場合は、サーバーが受信した要求ヘッダーを列挙します。 インライン ミドルウェアを使用し、アプリ応答に要求ヘッダーを書き込んだり、ヘッダーをログに記録したりします。 

アプリの応答にヘッダーを書き込むには、`Startup.Configure` 内の <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*> の呼び出しの直後に、次の端末のインライン ミドルウェアを配置します。

```csharp
app.Run(async (context) =>
{
    context.Response.ContentType = "text/plain";

    // Request method, scheme, and path
    await context.Response.WriteAsync(
        $"Request Method: {context.Request.Method}{Environment.NewLine}");
    await context.Response.WriteAsync(
        $"Request Scheme: {context.Request.Scheme}{Environment.NewLine}");
    await context.Response.WriteAsync(
        $"Request Path: {context.Request.Path}{Environment.NewLine}");

    // Headers
    await context.Response.WriteAsync($"Request Headers:{Environment.NewLine}");

    foreach (var header in context.Request.Headers)
    {
        await context.Response.WriteAsync($"{header.Key}: " +
            $"{header.Value}{Environment.NewLine}");
    }

    await context.Response.WriteAsync(Environment.NewLine);

    // Connection: RemoteIp
    await context.Response.WriteAsync(
        $"Request RemoteIp: {context.Connection.RemoteIpAddress}");
});
```

応答本文ではなく、ログに書き込むことができます。 ログに書き込むことで、デバッグしている間、サイトは正常に機能できます。

応答本文ではなく、ログに書き込むには:

* 「[Startup でログを作成する](xref:fundamentals/logging/index#create-logs-in-startup)」に説明されているように、`ILogger<Startup>` を `Startup` クラスに挿入します。
* `Startup.Configure` 内で <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*> を呼び出した直後に次のインライン ミドルウェアを配置します。

```csharp
app.Use(async (context, next) =>
{
    // Request method, scheme, and path
    _logger.LogDebug("Request Method: {Method}", context.Request.Method);
    _logger.LogDebug("Request Scheme: {Scheme}", context.Request.Scheme);
    _logger.LogDebug("Request Path: {Path}", context.Request.Path);

    // Headers
    foreach (var header in context.Request.Headers)
    {
        _logger.LogDebug("Header: {Key}: {Value}", header.Key, header.Value);
    }

    // Connection: RemoteIp
    _logger.LogDebug("Request RemoteIp: {RemoteIpAddress}", 
        context.Connection.RemoteIpAddress);

    await next();
});
```

処理時、`X-Forwarded-{For|Proto|Host}` 値は `X-Original-{For|Proto|Host}` に移動されます。 指定されたヘッダーに複数の値がある場合、Forwarded Headers Middleware では右から左の逆の順序でヘッダーが処理されます。 既定の `ForwardLimit` は `1` です。そのため、`ForwardLimit` の値を増やさない限り、ヘッダーの右端にある値のみが処理されます。

Forwarded Headers が処理される前に、要求の元のリモート IP アドレスが `KnownProxies` リストまたは `KnownNetworks` リストのエントリと一致する必要があります。 これにより、信頼されないプロキシからの転送を受け付けないことで、ヘッダーのスプーフィングを制限します。 不明なプロキシが検出されると、ログにプロキシのアドレスが示されます。

```console
September 20th 2018, 15:49:44.168 Unknown proxy: 10.0.0.100:54321
```

前の例では、10.0.0.100 がプロキシ サーバーです。 サーバーが信頼されているプロキシであれば、`Startup.ConfigureServices` で `KnownProxies` にサーバーの IP アドレスを追加します (あるいは、信頼されているネットワークを `KnownNetworks` に追加します)。 詳細については、「[Forwarded Headers Middleware のオプション](#forwarded-headers-middleware-options)」セクションをご覧ください。

```csharp
services.Configure<ForwardedHeadersOptions>(options =>
{
    options.KnownProxies.Add(IPAddress.Parse("10.0.0.100"));
});
```

> [!IMPORTANT]
> 信頼されているプロキシとネットワークにのみ、ヘッダーの転送を許可します。 それ以外に許可すると、[IP なりすまし](https://www.iplocation.net/ip-spoofing)攻撃が可能になります。

## <a name="additional-resources"></a>その他の技術情報

* <xref:host-and-deploy/web-farm>
* [Microsoft セキュリティ アドバイザリ CVE-2018-0787:ASP.NET Core の特権の昇格脆弱性](https://github.com/aspnet/Announcements/issues/295)

::: moniker-end
