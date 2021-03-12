---
title: ASP.NET Core での応答圧縮
author: rick-anderson
description: 応答圧縮と ASP.NET Core アプリで応答圧縮ミドルウェアを使用する方法について説明します。
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
uid: performance/response-compression
ms.openlocfilehash: 239f9e84d068bfd75c84ccf16f0e74cdbbbebfb2
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586320"
---
# <a name="response-compression-in-aspnet-core"></a>ASP.NET Core での応答圧縮

::: moniker range=">= aspnetcore-3.0"

ネットワーク帯域幅はリソースが限られています。 通常、応答のサイズを小さくすると、アプリの応答性が大幅に向上します。 ペイロードサイズを減らす方法の1つは、アプリの応答を圧縮することです。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/response-compression/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="when-to-use-response-compression-middleware"></a>応答圧縮ミドルウェアを使用する場合

IIS、Apache、または Nginx のサーバーベースの応答圧縮テクノロジを使用します。 ミドルウェアのパフォーマンスがサーバーモジュールのパフォーマンスと一致しない場合があります。 [HTTP.sys server](xref:fundamentals/servers/httpsys) Server と [kestrel](xref:fundamentals/servers/kestrel) server では、現在、組み込みの圧縮サポートは提供されていません。

次の場合に応答圧縮ミドルウェアを使用します。

* 次のサーバーベースの圧縮テクノロジは使用できません。
  * [IIS 動的圧縮モジュール](https://www.iis.net/overview/reliability/dynamiccachingandcompression)
  * [Apache mod_deflate モジュール](https://httpd.apache.org/docs/current/mod/mod_deflate.html)
  * [Nginx 圧縮と圧縮解除](https://www.nginx.com/resources/admin-guide/compression-and-decompression/)
* 直接ホストする:
  * [HTTP.sys サーバー](xref:fundamentals/servers/httpsys) (旧称 WebListener)
  * [Kestrel サーバー](xref:fundamentals/servers/kestrel)

## <a name="response-compression"></a>応答圧縮

通常、ネイティブに圧縮されていない応答は、応答圧縮の恩恵を受けることができます。 ネイティブに圧縮されていない応答には、通常、CSS、JavaScript、HTML、XML、JSON が含まれます。 PNG ファイルなど、ネイティブに圧縮されたアセットを圧縮することはできません。 ネイティブ圧縮応答をさらに圧縮しようとすると、サイズが小さく、転送時間が短縮されると、圧縮の処理に要する時間が大きくなります。 ファイルの内容や圧縮の効率によっては、約150-1000 バイトを下回るファイルを圧縮しないようにしてください。 小さいファイルを圧縮すると、圧縮されていないファイルよりも大きな圧縮ファイルが生成される可能性があります。

クライアントは、圧縮されたコンテンツを処理できる場合、 `Accept-Encoding` 要求と共にヘッダーを送信することによって、その機能をサーバーに通知する必要があります。 サーバーは、圧縮されたコンテンツを送信するときに、 `Content-Encoding` 圧縮された応答のエンコード方法に関する情報をヘッダーに含める必要があります。 次の表に、ミドルウェアでサポートされているコンテンツエンコードの表記を示します。

| `Accept-Encoding` ヘッダー値 | サポートされているミドルウェア | 説明 |
| ------------------------------- | :------------------: | ----------- |
| `br`                            | あり (既定)        | [Brotli 圧縮データ形式](https://tools.ietf.org/html/rfc7932) |
| `deflate`                       | いいえ                   | [圧縮データ形式の DEFLATE](https://tools.ietf.org/html/rfc1951) |
| `exi`                           | いいえ                   | [W3C の効率的な XML インターチェンジ](https://tools.ietf.org/id/draft-varga-netconf-exi-capability-00.html) |
| `gzip`                          | はい                  | [Gzip ファイル形式](https://tools.ietf.org/html/rfc1952) |
| `identity`                      | はい                  | "エンコードなし" 識別子: 応答をエンコードすることはできません。 |
| `pack200-gzip`                  | いいえ                   | [Java アーカイブのネットワーク転送形式](https://jcp.org/aboutJava/communityprocess/review/jsr200/index.html) |
| `*`                             | はい                  | 明示的に要求されていない利用可能なコンテンツエンコーディング |

詳細については、「 [IANA 公式コンテンツコーディングリスト](https://www.iana.org/assignments/http-parameters/http-parameters.xml#http-content-coding-registry)」を参照してください。

ミドルウェアを使用すると、カスタムヘッダー値の圧縮プロバイダーを追加でき `Accept-Encoding` ます。 詳細については、以下の「 [カスタムプロバイダー](#custom-providers) 」を参照してください。

ミドルウェアは、 `q` 圧縮スキームの優先順位を決定するためにクライアントから送信されるときに、品質の値 (qvalue,) の重み付けに対応できます。 詳細については、「 [RFC 7231: Accept-Encoding](https://tools.ietf.org/html/rfc7231#section-5.3.4)」を参照してください。

圧縮アルゴリズムは、圧縮速度と圧縮の効果のトレードオフの影響を受けます。 このコンテキストの *効果* は、圧縮後の出力のサイズを示します。 最小サイズは *最適* な圧縮によって実現されます。

次の表では、圧縮されたコンテンツの要求、送信、キャッシュ、および受信に関連するヘッダーについて説明します。

| ヘッダー             | ロール |
| ------------------ | ---- |
| `Accept-Encoding`  | クライアントからサーバーに送信され、クライアントが使用できるコンテンツエンコーディングスキームを示します。 |
| `Content-Encoding` | ペイロード内のコンテンツのエンコードを示すために、サーバーからクライアントに送信されます。 |
| `Content-Length`   | 圧縮が発生すると、 `Content-Length` 応答が圧縮されるときに本文の内容が変更されるため、ヘッダーは削除されます。 |
| `Content-MD5`      | 圧縮が行われると、 `Content-MD5` 本文の内容が変更され、ハッシュが有効でなくなったため、ヘッダーは削除されます。 |
| `Content-Type`     | コンテンツの MIME の種類を指定します。 すべての応答で、を指定する必要があり `Content-Type` ます。 ミドルウェアは、この値をチェックして、応答を圧縮する必要があるかどうかを判断します。 ミドルウェアはエンコードできる既定の [mime の種類](#mime-types) のセットを指定しますが、mime の種類を置き換えたり追加したりすることができます。 |
| `Vary`             | の値がであるサーバーからクライアントとプロキシに送信された場合 `Accept-Encoding` 、ヘッダーは、 `Vary` 要求のヘッダーの値に基づいて応答をキャッシュ (変更) する必要があることをクライアントまたはプロキシに示し `Accept-Encoding` ます。 ヘッダーを使用してコンテンツを返すと、 `Vary: Accept-Encoding` 圧縮された応答と圧縮されていない応答の両方が個別にキャッシュされることになります。 |

[サンプルアプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/response-compression/samples)を使用して、応答圧縮ミドルウェアの機能を検証します。 このサンプルでは、次のことを示します。

* Gzip およびカスタム圧縮プロバイダーを使用したアプリ応答の圧縮。
* 圧縮する mime の種類の既定の一覧に MIME の種類を追加する方法について説明します。

## <a name="package"></a>パッケージ

応答圧縮ミドルウェアは、AspNetCore アプリ ASP.NET Core に暗黙的に含まれる [ResponseCompression](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCompression/) パッケージによって提供されます。

## <a name="configuration"></a>構成

次のコードは、既定の MIME の種類と圧縮プロバイダー ([Brotli](#brotli-compression-provider) と [Gzip](#gzip-compression-provider)) に対して応答圧縮ミドルウェアを有効にする方法を示しています。

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddResponseCompression();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        app.UseResponseCompression();
    }
}
```

メモ:

* `app.UseResponseCompression` 応答を圧縮するミドルウェアの前にを呼び出す必要があります。 詳細については、「<xref:fundamentals/middleware/index#middleware-order>」を参照してください。
* [Fiddler](https://www.telerik.com/fiddler)、[消火バグ](https://getfirebug.com/)、 [Postman](https://www.getpostman.com/)などのツールを使用して要求ヘッダーを設定 `Accept-Encoding` し、応答ヘッダー、サイズ、および本文を調査します。

ヘッダーを使用せずにサンプルアプリに要求を送信 `Accept-Encoding` し、応答が圧縮されていないことを確認します。 `Content-Encoding` `Vary` 応答にヘッダーとヘッダーが存在しません。

![Accept-Encoding ヘッダーのない要求の結果を示す Fiddler ウィンドウ。 応答が圧縮されていません。](response-compression/_static/request-uncompressed.png)

ヘッダー (Brotli compression) を使用してサンプルアプリに要求を送信 `Accept-Encoding: br` し、応答が圧縮されていることを確認します。 `Content-Encoding` `Vary` 応答には、ヘッダーとヘッダーが存在します。

![Accept-Encoding ヘッダーと br の値を含む要求の結果を示す Fiddler ウィンドウ。 Vary ヘッダーと Content-type ヘッダーが応答に追加されます。 応答は圧縮されます。](response-compression/_static/request-compressed-br.png)

## <a name="providers"></a>プロバイダー

### <a name="brotli-compression-provider"></a>Brotli 圧縮プロバイダー

<xref:Microsoft.AspNetCore.ResponseCompression.BrotliCompressionProvider> [Brotli 圧縮データ形式](https://tools.ietf.org/html/rfc7932)で応答を圧縮するには、を使用します。

圧縮プロバイダーがに明示的に追加されていない場合は、次のように <xref:Microsoft.AspNetCore.ResponseCompression.CompressionProviderCollection> なります。

* 既定では、Brotli 圧縮プロバイダーは、 [Gzip 圧縮プロバイダー](#gzip-compression-provider)と共に、圧縮プロバイダーの配列に追加されます。
* Brotli 圧縮データ形式がクライアントでサポートされている場合、圧縮は既定で Brotli 圧縮になります。 Brotli がクライアントでサポートされていない場合、クライアントで Gzip 圧縮がサポートされていると、圧縮の既定値は Gzip になります。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();
}
```

圧縮プロバイダーが明示的に追加されている場合は、Brotli 圧縮プロバイダーを追加する必要があります。

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=5)]

圧縮レベルをに設定 <xref:Microsoft.AspNetCore.ResponseCompression.BrotliCompressionProviderOptions> します。 Brotli 圧縮プロバイダーは、既定で最も高速な圧縮レベル ([CompressionLevel](xref:System.IO.Compression.CompressionLevel)) に設定されています。これにより、圧縮率が最も高くなる可能性があります。 最も効率的な圧縮が必要な場合は、最適な圧縮のためにミドルウェアを構成します。

| Compression Level | 説明 |
| ----------------- | ----------- |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮は、結果の出力が最適に圧縮されない場合でも、できるだけ早く完了する必要があります。 |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮は実行されません。 |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮が完了するまでに時間がかかる場合でも、応答は最適に圧縮される必要があります。 |

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();

    services.Configure<BrotliCompressionProviderOptions>(options => 
    {
        options.Level = CompressionLevel.Fastest;
    });
}
```

### <a name="gzip-compression-provider"></a>Gzip 圧縮プロバイダー

を使用して、 <xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProvider> [Gzip ファイル形式](https://tools.ietf.org/html/rfc1952)で応答を圧縮します。

圧縮プロバイダーがに明示的に追加されていない場合は、次のように <xref:Microsoft.AspNetCore.ResponseCompression.CompressionProviderCollection> なります。

* Gzip 圧縮プロバイダーは、既定では、 [Brotli 圧縮プロバイダー](#brotli-compression-provider)と共に、圧縮プロバイダーの配列に追加されます。
* Brotli 圧縮データ形式がクライアントでサポートされている場合、圧縮は既定で Brotli 圧縮になります。 Brotli がクライアントでサポートされていない場合、クライアントで Gzip 圧縮がサポートされていると、圧縮の既定値は Gzip になります。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();
}
```

圧縮プロバイダーが明示的に追加されている場合は、Gzip 圧縮プロバイダーを追加する必要があります。

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=6)]

圧縮レベルをに設定 <xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProviderOptions> します。 Gzip 圧縮プロバイダーは、既定で最も高速な圧縮レベル ([CompressionLevel](xref:System.IO.Compression.CompressionLevel)) に設定されています。これにより、圧縮率が最も高くなる可能性があります。 最も効率的な圧縮が必要な場合は、最適な圧縮のためにミドルウェアを構成します。

| Compression Level | 説明 |
| ----------------- | ----------- |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮は、結果の出力が最適に圧縮されない場合でも、できるだけ早く完了する必要があります。 |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮は実行されません。 |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮が完了するまでに時間がかかる場合でも、応答は最適に圧縮される必要があります。 |

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();

    services.Configure<GzipCompressionProviderOptions>(options => 
    {
        options.Level = CompressionLevel.Fastest;
    });
}
```

### <a name="custom-providers"></a>カスタム プロバイダー

を使用して、カスタム圧縮実装を作成 <xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider> します。 は、 <xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider.EncodingName*> このによって生成されるコンテンツエンコーディングを表し `ICompressionProvider` ます。 ミドルウェアは、この情報を使用して、要求のヘッダーに指定されている一覧に基づいてプロバイダーを選択し `Accept-Encoding` ます。

サンプルアプリを使用して、クライアントはヘッダーを使用して要求を送信し `Accept-Encoding: mycustomcompression` ます。 ミドルウェアはカスタム圧縮実装を使用して、応答をヘッダーと共に返し `Content-Encoding: mycustomcompression` ます。 カスタム圧縮実装を機能させるには、クライアントがカスタムエンコードを圧縮解除できる必要があります。

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=7)]

[!code-csharp[](response-compression/samples/3.x/SampleApp/CustomCompressionProvider.cs?name=snippet1)]


ヘッダーを使用してサンプルアプリに要求を送信し、応答ヘッダーを確認し `Accept-Encoding: mycustomcompression` ます。 `Vary` `Content-Encoding` 応答には、ヘッダーとヘッダーが存在します。 このサンプルでは、応答本文 (表示されません) は圧縮されません。 サンプルのクラスには圧縮実装がありません `CustomCompressionProvider` 。 ただし、このサンプルでは、このような圧縮アルゴリズムを実装する場所を示しています。

![Accept-Encoding ヘッダーと mycustomcompression の値を含む要求の結果を示す Fiddler ウィンドウ。 Vary ヘッダーと Content-type ヘッダーが応答に追加されます。](response-compression/_static/request-custom-compression.png)

## <a name="mime-types"></a>MIME タイプ

ミドルウェアは、圧縮のために既定の MIME の種類のセットを指定します。

* `application/javascript`
* `application/json`
* `application/xml`
* `text/css`
* `text/html`
* `text/json`
* `text/plain`
* `text/xml`

応答の圧縮ミドルウェアオプションを使用して、MIME の種類を置換または追加します。 などのワイルドカードの MIME 型はサポートされていないことに注意して `text/*` ください。 このサンプルアプリでは、の MIME の種類を追加 `image/svg+xml` し、ASP.NET Core バナーイメージ (*横断幕*) を圧縮して提供します。

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=8-10)]

## <a name="compression-with-secure-protocol"></a>セキュリティで保護されたプロトコルを使用した圧縮

セキュリティで保護された接続での圧縮された応答は、 `EnableForHttps` 既定では無効になっているオプションを使用して制御できます。 動的に生成されたページで圧縮を使用すると、 [犯罪](https://wikipedia.org/wiki/CRIME_(security_exploit)) や [侵害](https://wikipedia.org/wiki/BREACH_(security_exploit)) 攻撃などのセキュリティ上の問題が発生する可能性があります。

## <a name="adding-the-vary-header"></a>Vary ヘッダーを追加する

ヘッダーに基づいて応答を圧縮する場合 `Accept-Encoding` 、応答の複数の圧縮バージョンと圧縮されていないバージョンが存在する可能性があります。 複数のバージョンが存在し、格納する必要があることをクライアントキャッシュとプロキシキャッシュに指示するために、 `Vary` ヘッダーに値を追加し `Accept-Encoding` ます。 ASP.NET Core 2.0 以降では、 `Vary` 応答が圧縮されるときに、ミドルウェアによってヘッダーが自動的に追加されます。

## <a name="middleware-issue-when-behind-an-nginx-reverse-proxy"></a>Nginx リバースプロキシの背後にあるミドルウェアの問題

要求が Nginx によってプロキシされている場合、 `Accept-Encoding` ヘッダーは削除されます。 ヘッダーを削除すると、 `Accept-Encoding` ミドルウェアが応答を圧縮できなくなります。 詳細については、「 [NGINX: Compression And 伸張](https://www.nginx.com/resources/admin-guide/compression-and-decompression/)」を参照してください。 この問題は、 [Nginx (aspnet/BasicMiddleware #123) のパススルー圧縮](https://github.com/aspnet/BasicMiddleware/issues/123)によって追跡されます。

## <a name="working-with-iis-dynamic-compression"></a>IIS 動的圧縮の使用

アプリケーションに対して無効にする、アクティブな IIS 動的圧縮モジュールがサーバーレベルで構成されている場合は、 *web.config* ファイルを追加してモジュールを無効にします。 詳細については、「[Disabling IIS modules](xref:host-and-deploy/iis/modules#disabling-iis-modules)」 (IIS モジュールの無効化) を参照してください。

## <a name="troubleshooting"></a>トラブルシューティング

[Fiddler](https://www.telerik.com/fiddler)、[消火バグ](https://getfirebug.com/)、 [Postman](https://www.getpostman.com/)などのツールを使用します。これにより、 `Accept-Encoding` 要求ヘッダーを設定し、応答ヘッダー、サイズ、本文を調べることができます。 既定では、応答圧縮ミドルウェアは、次の条件を満たす応答を圧縮します。

* `Accept-Encoding`ヘッダーには `br` 、設定した `gzip` `*` カスタム圧縮プロバイダーに一致する値、、、またはカスタムエンコーディングが含まれています。 値は、 `identity` または品質値 (qvalue, `q` ) の 0 (ゼロ) に設定することはできません。
* MIME の種類 ( `Content-Type` ) を設定し、で構成されている mime の種類と一致させる必要があり <xref:Microsoft.AspNetCore.ResponseCompression.ResponseCompressionOptions> ます。
* 要求にヘッダーを含めることはできません `Content-Range` 。
* 応答圧縮ミドルウェアのオプションで secure protocol (https) が構成されている場合を除き、要求では安全でないプロトコル (http) を使用する必要があります。 *セキュリティで保護されたコンテンツの圧縮を有効にする場合は、 [前述](#compression-with-secure-protocol) の危険に注意してください。*

## <a name="additional-resources"></a>その他のリソース

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* [Mozilla Developer Network: Accept-Encoding](https://developer.mozilla.org/docs/Web/HTTP/Headers/Accept-Encoding)
* [RFC 7231 セクション 3.1.2.1: Content Codings](https://tools.ietf.org/html/rfc7231#section-3.1.2.1)
* [RFC 7230 セクション 4.2.3: Gzip コーディング](https://tools.ietf.org/html/rfc7230#section-4.2.3)
* [GZIP ファイル形式仕様バージョン4.3](https://www.ietf.org/rfc/rfc1952.txt)

::: moniker-end

::: moniker range="= aspnetcore-2.2"

ネットワーク帯域幅はリソースが限られています。 通常、応答のサイズを小さくすると、アプリの応答性が大幅に向上します。 ペイロードサイズを減らす方法の1つは、アプリの応答を圧縮することです。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/response-compression/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="when-to-use-response-compression-middleware"></a>応答圧縮ミドルウェアを使用する場合

IIS、Apache、または Nginx のサーバーベースの応答圧縮テクノロジを使用します。 ミドルウェアのパフォーマンスがサーバーモジュールのパフォーマンスと一致しない場合があります。 [HTTP.sys server](xref:fundamentals/servers/httpsys) Server と [kestrel](xref:fundamentals/servers/kestrel) server では、現在、組み込みの圧縮サポートは提供されていません。

次の場合に応答圧縮ミドルウェアを使用します。

* 次のサーバーベースの圧縮テクノロジは使用できません。
  * [IIS 動的圧縮モジュール](https://www.iis.net/overview/reliability/dynamiccachingandcompression)
  * [Apache mod_deflate モジュール](https://httpd.apache.org/docs/current/mod/mod_deflate.html)
  * [Nginx 圧縮と圧縮解除](https://www.nginx.com/resources/admin-guide/compression-and-decompression/)
* 直接ホストする:
  * [HTTP.sys サーバー](xref:fundamentals/servers/httpsys) (旧称 WebListener)
  * [Kestrel サーバー](xref:fundamentals/servers/kestrel)

## <a name="response-compression"></a>応答圧縮

通常、ネイティブに圧縮されていない応答は、応答圧縮の恩恵を受けることができます。 ネイティブに圧縮されていない応答には、通常、CSS、JavaScript、HTML、XML、JSON が含まれます。 PNG ファイルなど、ネイティブに圧縮されたアセットを圧縮することはできません。 ネイティブ圧縮応答をさらに圧縮しようとすると、サイズが小さく、転送時間が短縮されると、圧縮の処理に要する時間が大きくなります。 ファイルの内容や圧縮の効率によっては、約150-1000 バイトを下回るファイルを圧縮しないようにしてください。 小さいファイルを圧縮すると、圧縮されていないファイルよりも大きな圧縮ファイルが生成される可能性があります。

クライアントは、圧縮されたコンテンツを処理できる場合、 `Accept-Encoding` 要求と共にヘッダーを送信することによって、その機能をサーバーに通知する必要があります。 サーバーは、圧縮されたコンテンツを送信するときに、 `Content-Encoding` 圧縮された応答のエンコード方法に関する情報をヘッダーに含める必要があります。 次の表に、ミドルウェアでサポートされているコンテンツエンコードの表記を示します。

| `Accept-Encoding` ヘッダー値 | サポートされているミドルウェア | 説明 |
| ------------------------------- | :------------------: | ----------- |
| `br`                            | あり (既定)        | [Brotli 圧縮データ形式](https://tools.ietf.org/html/rfc7932) |
| `deflate`                       | いいえ                   | [圧縮データ形式の DEFLATE](https://tools.ietf.org/html/rfc1951) |
| `exi`                           | いいえ                   | [W3C の効率的な XML インターチェンジ](https://tools.ietf.org/id/draft-varga-netconf-exi-capability-00.html) |
| `gzip`                          | はい                  | [Gzip ファイル形式](https://tools.ietf.org/html/rfc1952) |
| `identity`                      | はい                  | "エンコードなし" 識別子: 応答をエンコードすることはできません。 |
| `pack200-gzip`                  | いいえ                   | [Java アーカイブのネットワーク転送形式](https://jcp.org/aboutJava/communityprocess/review/jsr200/index.html) |
| `*`                             | はい                  | 明示的に要求されていない利用可能なコンテンツエンコーディング |

詳細については、「 [IANA 公式コンテンツコーディングリスト](https://www.iana.org/assignments/http-parameters/http-parameters.xml#http-content-coding-registry)」を参照してください。

ミドルウェアを使用すると、カスタムヘッダー値の圧縮プロバイダーを追加でき `Accept-Encoding` ます。 詳細については、以下の「 [カスタムプロバイダー](#custom-providers) 」を参照してください。

ミドルウェアは、 `q` 圧縮スキームの優先順位を決定するためにクライアントから送信されるときに、品質の値 (qvalue,) の重み付けに対応できます。 詳細については、「 [RFC 7231: Accept-Encoding](https://tools.ietf.org/html/rfc7231#section-5.3.4)」を参照してください。

圧縮アルゴリズムは、圧縮速度と圧縮の効果のトレードオフの影響を受けます。 このコンテキストの *効果* は、圧縮後の出力のサイズを示します。 最小サイズは *最適* な圧縮によって実現されます。

次の表では、圧縮されたコンテンツの要求、送信、キャッシュ、および受信に関連するヘッダーについて説明します。

| ヘッダー             | ロール |
| ------------------ | ---- |
| `Accept-Encoding`  | クライアントからサーバーに送信され、クライアントが使用できるコンテンツエンコーディングスキームを示します。 |
| `Content-Encoding` | ペイロード内のコンテンツのエンコードを示すために、サーバーからクライアントに送信されます。 |
| `Content-Length`   | 圧縮が発生すると、 `Content-Length` 応答が圧縮されるときに本文の内容が変更されるため、ヘッダーは削除されます。 |
| `Content-MD5`      | 圧縮が行われると、 `Content-MD5` 本文の内容が変更され、ハッシュが有効でなくなったため、ヘッダーは削除されます。 |
| `Content-Type`     | コンテンツの MIME の種類を指定します。 すべての応答で、を指定する必要があり `Content-Type` ます。 ミドルウェアは、この値をチェックして、応答を圧縮する必要があるかどうかを判断します。 ミドルウェアはエンコードできる既定の [mime の種類](#mime-types) のセットを指定しますが、mime の種類を置き換えたり追加したりすることができます。 |
| `Vary`             | の値がであるサーバーからクライアントとプロキシに送信された場合 `Accept-Encoding` 、ヘッダーは、 `Vary` 要求のヘッダーの値に基づいて応答をキャッシュ (変更) する必要があることをクライアントまたはプロキシに示し `Accept-Encoding` ます。 ヘッダーを使用してコンテンツを返すと、 `Vary: Accept-Encoding` 圧縮された応答と圧縮されていない応答の両方が個別にキャッシュされることになります。 |

[サンプルアプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/response-compression/samples)を使用して、応答圧縮ミドルウェアの機能を検証します。 このサンプルでは、次のことを示します。

* Gzip およびカスタム圧縮プロバイダーを使用したアプリ応答の圧縮。
* 圧縮する mime の種類の既定の一覧に MIME の種類を追加する方法について説明します。

## <a name="package"></a>パッケージ

ミドルウェアをプロジェクトに含めるには、 [AspNetCore メタパッケージ](xref:fundamentals/metapackage-app)への参照を追加します。これには、 [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCompression/) パッケージが含まれています。

## <a name="configuration"></a>構成

次のコードは、既定の MIME の種類と圧縮プロバイダー ([Brotli](#brotli-compression-provider) と [Gzip](#gzip-compression-provider)) に対して応答圧縮ミドルウェアを有効にする方法を示しています。

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddResponseCompression();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        app.UseResponseCompression();
    }
}
```

メモ:

* `app.UseResponseCompression` 応答を圧縮するミドルウェアの前にを呼び出す必要があります。 詳細については、「<xref:fundamentals/middleware/index#middleware-order>」を参照してください。
* [Fiddler](https://www.telerik.com/fiddler)、[消火バグ](https://getfirebug.com/)、 [Postman](https://www.getpostman.com/)などのツールを使用して要求ヘッダーを設定 `Accept-Encoding` し、応答ヘッダー、サイズ、および本文を調査します。

ヘッダーを使用せずにサンプルアプリに要求を送信 `Accept-Encoding` し、応答が圧縮されていないことを確認します。 `Content-Encoding` `Vary` 応答にヘッダーとヘッダーが存在しません。

![Accept-Encoding ヘッダーのない要求の結果を示す Fiddler ウィンドウ。 応答が圧縮されていません。](response-compression/_static/request-uncompressed.png)

ヘッダー (Brotli compression) を使用してサンプルアプリに要求を送信 `Accept-Encoding: br` し、応答が圧縮されていることを確認します。 `Content-Encoding` `Vary` 応答には、ヘッダーとヘッダーが存在します。

![Accept-Encoding ヘッダーと br の値を含む要求の結果を示す Fiddler ウィンドウ。 Vary ヘッダーと Content-type ヘッダーが応答に追加されます。 応答は圧縮されます。](response-compression/_static/request-compressed-br.png)

## <a name="providers"></a>プロバイダー

### <a name="brotli-compression-provider"></a>Brotli 圧縮プロバイダー

<xref:Microsoft.AspNetCore.ResponseCompression.BrotliCompressionProvider> [Brotli 圧縮データ形式](https://tools.ietf.org/html/rfc7932)で応答を圧縮するには、を使用します。

圧縮プロバイダーがに明示的に追加されていない場合は、次のように <xref:Microsoft.AspNetCore.ResponseCompression.CompressionProviderCollection> なります。

* 既定では、Brotli 圧縮プロバイダーは、 [Gzip 圧縮プロバイダー](#gzip-compression-provider)と共に、圧縮プロバイダーの配列に追加されます。
* Brotli 圧縮データ形式がクライアントでサポートされている場合、圧縮は既定で Brotli 圧縮になります。 Brotli がクライアントでサポートされていない場合、クライアントで Gzip 圧縮がサポートされていると、圧縮の既定値は Gzip になります。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();
}
```

圧縮プロバイダーが明示的に追加されている場合は、Brotli 圧縮プロバイダーを追加する必要があります。

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=5)]

圧縮レベルをに設定 <xref:Microsoft.AspNetCore.ResponseCompression.BrotliCompressionProviderOptions> します。 Brotli 圧縮プロバイダーは、既定で最も高速な圧縮レベル ([CompressionLevel](xref:System.IO.Compression.CompressionLevel)) に設定されています。これにより、圧縮率が最も高くなる可能性があります。 最も効率的な圧縮が必要な場合は、最適な圧縮のためにミドルウェアを構成します。

| Compression Level | 説明 |
| ----------------- | ----------- |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮は、結果の出力が最適に圧縮されない場合でも、できるだけ早く完了する必要があります。 |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮は実行されません。 |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮が完了するまでに時間がかかる場合でも、応答は最適に圧縮される必要があります。 |

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();

    services.Configure<BrotliCompressionProviderOptions>(options => 
    {
        options.Level = CompressionLevel.Fastest;
    });
}
```

### <a name="gzip-compression-provider"></a>Gzip 圧縮プロバイダー

を使用して、 <xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProvider> [Gzip ファイル形式](https://tools.ietf.org/html/rfc1952)で応答を圧縮します。

圧縮プロバイダーがに明示的に追加されていない場合は、次のように <xref:Microsoft.AspNetCore.ResponseCompression.CompressionProviderCollection> なります。

* Gzip 圧縮プロバイダーは、既定では、 [Brotli 圧縮プロバイダー](#brotli-compression-provider)と共に、圧縮プロバイダーの配列に追加されます。
* Brotli 圧縮データ形式がクライアントでサポートされている場合、圧縮は既定で Brotli 圧縮になります。 Brotli がクライアントでサポートされていない場合、クライアントで Gzip 圧縮がサポートされていると、圧縮の既定値は Gzip になります。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();
}
```

圧縮プロバイダーが明示的に追加されている場合は、Gzip 圧縮プロバイダーを追加する必要があります。

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=6)]

圧縮レベルをに設定 <xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProviderOptions> します。 Gzip 圧縮プロバイダーは、既定で最も高速な圧縮レベル ([CompressionLevel](xref:System.IO.Compression.CompressionLevel)) に設定されています。これにより、圧縮率が最も高くなる可能性があります。 最も効率的な圧縮が必要な場合は、最適な圧縮のためにミドルウェアを構成します。

| Compression Level | 説明 |
| ----------------- | ----------- |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮は、結果の出力が最適に圧縮されない場合でも、できるだけ早く完了する必要があります。 |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮は実行されません。 |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮が完了するまでに時間がかかる場合でも、応答は最適に圧縮される必要があります。 |

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();

    services.Configure<GzipCompressionProviderOptions>(options => 
    {
        options.Level = CompressionLevel.Fastest;
    });
}
```

### <a name="custom-providers"></a>カスタム プロバイダー

を使用して、カスタム圧縮実装を作成 <xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider> します。 は、 <xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider.EncodingName*> このによって生成されるコンテンツエンコーディングを表し `ICompressionProvider` ます。 ミドルウェアは、この情報を使用して、要求のヘッダーに指定されている一覧に基づいてプロバイダーを選択し `Accept-Encoding` ます。

サンプルアプリを使用して、クライアントはヘッダーを使用して要求を送信し `Accept-Encoding: mycustomcompression` ます。 ミドルウェアはカスタム圧縮実装を使用して、応答をヘッダーと共に返し `Content-Encoding: mycustomcompression` ます。 カスタム圧縮実装を機能させるには、クライアントがカスタムエンコードを圧縮解除できる必要があります。

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=7)]

[!code-csharp[](response-compression/samples/2.x/SampleApp/CustomCompressionProvider.cs?name=snippet1)]

ヘッダーを使用してサンプルアプリに要求を送信し、応答ヘッダーを確認し `Accept-Encoding: mycustomcompression` ます。 `Vary` `Content-Encoding` 応答には、ヘッダーとヘッダーが存在します。 このサンプルでは、応答本文 (表示されません) は圧縮されません。 サンプルのクラスには圧縮実装がありません `CustomCompressionProvider` 。 ただし、このサンプルでは、このような圧縮アルゴリズムを実装する場所を示しています。

![Accept-Encoding ヘッダーと mycustomcompression の値を含む要求の結果を示す Fiddler ウィンドウ。 Vary ヘッダーと Content-type ヘッダーが応答に追加されます。](response-compression/_static/request-custom-compression.png)

## <a name="mime-types"></a>MIME タイプ

ミドルウェアは、圧縮のために既定の MIME の種類のセットを指定します。

* `application/javascript`
* `application/json`
* `application/xml`
* `text/css`
* `text/html`
* `text/json`
* `text/plain`
* `text/xml`

応答の圧縮ミドルウェアオプションを使用して、MIME の種類を置換または追加します。 などのワイルドカードの MIME 型はサポートされていないことに注意して `text/*` ください。 このサンプルアプリでは、の MIME の種類を追加 `image/svg+xml` し、ASP.NET Core バナーイメージ (*横断幕*) を圧縮して提供します。

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=8-10)]

## <a name="compression-with-secure-protocol"></a>セキュリティで保護されたプロトコルを使用した圧縮

セキュリティで保護された接続での圧縮された応答は、 `EnableForHttps` 既定では無効になっているオプションを使用して制御できます。 動的に生成されたページで圧縮を使用すると、 [犯罪](https://wikipedia.org/wiki/CRIME_(security_exploit)) や [侵害](https://wikipedia.org/wiki/BREACH_(security_exploit)) 攻撃などのセキュリティ上の問題が発生する可能性があります。

## <a name="adding-the-vary-header"></a>Vary ヘッダーを追加する

ヘッダーに基づいて応答を圧縮する場合 `Accept-Encoding` 、応答の複数の圧縮バージョンと圧縮されていないバージョンが存在する可能性があります。 複数のバージョンが存在し、格納する必要があることをクライアントキャッシュとプロキシキャッシュに指示するために、 `Vary` ヘッダーに値を追加し `Accept-Encoding` ます。 ASP.NET Core 2.0 以降では、 `Vary` 応答が圧縮されるときに、ミドルウェアによってヘッダーが自動的に追加されます。

## <a name="middleware-issue-when-behind-an-nginx-reverse-proxy"></a>Nginx リバースプロキシの背後にあるミドルウェアの問題

要求が Nginx によってプロキシされている場合、 `Accept-Encoding` ヘッダーは削除されます。 ヘッダーを削除すると、 `Accept-Encoding` ミドルウェアが応答を圧縮できなくなります。 詳細については、「 [NGINX: Compression And 伸張](https://www.nginx.com/resources/admin-guide/compression-and-decompression/)」を参照してください。 この問題は、 [Nginx (aspnet/BasicMiddleware #123) のパススルー圧縮](https://github.com/aspnet/BasicMiddleware/issues/123)によって追跡されます。

## <a name="working-with-iis-dynamic-compression"></a>IIS 動的圧縮の使用

アプリケーションに対して無効にする、アクティブな IIS 動的圧縮モジュールがサーバーレベルで構成されている場合は、 *web.config* ファイルを追加してモジュールを無効にします。 詳細については、「[Disabling IIS modules](xref:host-and-deploy/iis/modules#disabling-iis-modules)」 (IIS モジュールの無効化) を参照してください。

## <a name="troubleshooting"></a>トラブルシューティング

[Fiddler](https://www.telerik.com/fiddler)、[消火バグ](https://getfirebug.com/)、 [Postman](https://www.getpostman.com/)などのツールを使用します。これにより、 `Accept-Encoding` 要求ヘッダーを設定し、応答ヘッダー、サイズ、本文を調べることができます。 既定では、応答圧縮ミドルウェアは、次の条件を満たす応答を圧縮します。

* `Accept-Encoding`ヘッダーには `br` 、設定した `gzip` `*` カスタム圧縮プロバイダーに一致する値、、、またはカスタムエンコーディングが含まれています。 値は、 `identity` または品質値 (qvalue, `q` ) の 0 (ゼロ) に設定することはできません。
* MIME の種類 ( `Content-Type` ) を設定し、で構成されている mime の種類と一致させる必要があり <xref:Microsoft.AspNetCore.ResponseCompression.ResponseCompressionOptions> ます。
* 要求にヘッダーを含めることはできません `Content-Range` 。
* 応答圧縮ミドルウェアのオプションで secure protocol (https) が構成されている場合を除き、要求では安全でないプロトコル (http) を使用する必要があります。 *セキュリティで保護されたコンテンツの圧縮を有効にする場合は、 [前述](#compression-with-secure-protocol) の危険に注意してください。*

## <a name="additional-resources"></a>その他のリソース

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* [Mozilla Developer Network: Accept-Encoding](https://developer.mozilla.org/docs/Web/HTTP/Headers/Accept-Encoding)
* [RFC 7231 セクション 3.1.2.1: Content Codings](https://tools.ietf.org/html/rfc7231#section-3.1.2.1)
* [RFC 7230 セクション 4.2.3: Gzip コーディング](https://tools.ietf.org/html/rfc7230#section-4.2.3)
* [GZIP ファイル形式仕様バージョン4.3](https://www.ietf.org/rfc/rfc1952.txt)

::: moniker-end

::: moniker range="< aspnetcore-2.2"

ネットワーク帯域幅はリソースが限られています。 通常、応答のサイズを小さくすると、アプリの応答性が大幅に向上します。 ペイロードサイズを減らす方法の1つは、アプリの応答を圧縮することです。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/response-compression/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="when-to-use-response-compression-middleware"></a>応答圧縮ミドルウェアを使用する場合

IIS、Apache、または Nginx のサーバーベースの応答圧縮テクノロジを使用します。 ミドルウェアのパフォーマンスがサーバーモジュールのパフォーマンスと一致しない場合があります。 [HTTP.sys server](xref:fundamentals/servers/httpsys) Server と [kestrel](xref:fundamentals/servers/kestrel) server では、現在、組み込みの圧縮サポートは提供されていません。

次の場合に応答圧縮ミドルウェアを使用します。

* 次のサーバーベースの圧縮テクノロジは使用できません。
  * [IIS 動的圧縮モジュール](https://www.iis.net/overview/reliability/dynamiccachingandcompression)
  * [Apache mod_deflate モジュール](https://httpd.apache.org/docs/current/mod/mod_deflate.html)
  * [Nginx 圧縮と圧縮解除](https://www.nginx.com/resources/admin-guide/compression-and-decompression/)
* 直接ホストする:
  * [HTTP.sys サーバー](xref:fundamentals/servers/httpsys) (旧称 WebListener)
  * [Kestrel サーバー](xref:fundamentals/servers/kestrel)

## <a name="response-compression"></a>応答圧縮

通常、ネイティブに圧縮されていない応答は、応答圧縮の恩恵を受けることができます。 ネイティブに圧縮されていない応答には、通常、CSS、JavaScript、HTML、XML、JSON が含まれます。 PNG ファイルなど、ネイティブに圧縮されたアセットを圧縮することはできません。 ネイティブ圧縮応答をさらに圧縮しようとすると、サイズが小さく、転送時間が短縮されると、圧縮の処理に要する時間が大きくなります。 ファイルの内容や圧縮の効率によっては、約150-1000 バイトを下回るファイルを圧縮しないようにしてください。 小さいファイルを圧縮すると、圧縮されていないファイルよりも大きな圧縮ファイルが生成される可能性があります。

クライアントは、圧縮されたコンテンツを処理できる場合、 `Accept-Encoding` 要求と共にヘッダーを送信することによって、その機能をサーバーに通知する必要があります。 サーバーは、圧縮されたコンテンツを送信するときに、 `Content-Encoding` 圧縮された応答のエンコード方法に関する情報をヘッダーに含める必要があります。 次の表に、ミドルウェアでサポートされているコンテンツエンコードの表記を示します。

| `Accept-Encoding` ヘッダー値 | サポートされているミドルウェア | 説明 |
| ------------------------------- | :------------------: | ----------- |
| `br`                            | いいえ                   | [Brotli 圧縮データ形式](https://tools.ietf.org/html/rfc7932) |
| `deflate`                       | いいえ                   | [圧縮データ形式の DEFLATE](https://tools.ietf.org/html/rfc1951) |
| `exi`                           | いいえ                   | [W3C の効率的な XML インターチェンジ](https://tools.ietf.org/id/draft-varga-netconf-exi-capability-00.html) |
| `gzip`                          | あり (既定)        | [Gzip ファイル形式](https://tools.ietf.org/html/rfc1952) |
| `identity`                      | はい                  | "エンコードなし" 識別子: 応答をエンコードすることはできません。 |
| `pack200-gzip`                  | いいえ                   | [Java アーカイブのネットワーク転送形式](https://jcp.org/aboutJava/communityprocess/review/jsr200/index.html) |
| `*`                             | はい                  | 明示的に要求されていない利用可能なコンテンツエンコーディング |

詳細については、「 [IANA 公式コンテンツコーディングリスト](https://www.iana.org/assignments/http-parameters/http-parameters.xml#http-content-coding-registry)」を参照してください。

ミドルウェアを使用すると、カスタムヘッダー値の圧縮プロバイダーを追加でき `Accept-Encoding` ます。 詳細については、以下の「 [カスタムプロバイダー](#custom-providers) 」を参照してください。

ミドルウェアは、 `q` 圧縮スキームの優先順位を決定するためにクライアントから送信されるときに、品質の値 (qvalue,) の重み付けに対応できます。 詳細については、「 [RFC 7231: Accept-Encoding](https://tools.ietf.org/html/rfc7231#section-5.3.4)」を参照してください。

圧縮アルゴリズムは、圧縮速度と圧縮の効果のトレードオフの影響を受けます。 このコンテキストの *効果* は、圧縮後の出力のサイズを示します。 最小サイズは *最適* な圧縮によって実現されます。

次の表では、圧縮されたコンテンツの要求、送信、キャッシュ、および受信に関連するヘッダーについて説明します。

| ヘッダー             | ロール |
| ------------------ | ---- |
| `Accept-Encoding`  | クライアントからサーバーに送信され、クライアントが使用できるコンテンツエンコーディングスキームを示します。 |
| `Content-Encoding` | ペイロード内のコンテンツのエンコードを示すために、サーバーからクライアントに送信されます。 |
| `Content-Length`   | 圧縮が発生すると、 `Content-Length` 応答が圧縮されるときに本文の内容が変更されるため、ヘッダーは削除されます。 |
| `Content-MD5`      | 圧縮が行われると、 `Content-MD5` 本文の内容が変更され、ハッシュが有効でなくなったため、ヘッダーは削除されます。 |
| `Content-Type`     | コンテンツの MIME の種類を指定します。 すべての応答で、を指定する必要があり `Content-Type` ます。 ミドルウェアは、この値をチェックして、応答を圧縮する必要があるかどうかを判断します。 ミドルウェアはエンコードできる既定の [mime の種類](#mime-types) のセットを指定しますが、mime の種類を置き換えたり追加したりすることができます。 |
| `Vary`             | の値がであるサーバーからクライアントとプロキシに送信された場合 `Accept-Encoding` 、ヘッダーは、 `Vary` 要求のヘッダーの値に基づいて応答をキャッシュ (変更) する必要があることをクライアントまたはプロキシに示し `Accept-Encoding` ます。 ヘッダーを使用してコンテンツを返すと、 `Vary: Accept-Encoding` 圧縮された応答と圧縮されていない応答の両方が個別にキャッシュされることになります。 |

[サンプルアプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/response-compression/samples)を使用して、応答圧縮ミドルウェアの機能を検証します。 このサンプルでは、次のことを示します。

* Gzip およびカスタム圧縮プロバイダーを使用したアプリ応答の圧縮。
* 圧縮する mime の種類の既定の一覧に MIME の種類を追加する方法について説明します。

## <a name="package"></a>パッケージ

ミドルウェアをプロジェクトに含めるには、 [AspNetCore メタパッケージ](xref:fundamentals/metapackage-app)への参照を追加します。これには、 [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCompression/) パッケージが含まれています。

## <a name="configuration"></a>構成

次のコードは、既定の MIME の種類と [Gzip 圧縮プロバイダー](#gzip-compression-provider)の応答圧縮ミドルウェアを有効にする方法を示しています。

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddResponseCompression();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        app.UseResponseCompression();
    }
}
```

メモ:

* `app.UseResponseCompression` 応答を圧縮するミドルウェアの前にを呼び出す必要があります。 詳細については、「<xref:fundamentals/middleware/index#middleware-order>」を参照してください。
* [Fiddler](https://www.telerik.com/fiddler)、[消火バグ](https://getfirebug.com/)、 [Postman](https://www.getpostman.com/)などのツールを使用して要求ヘッダーを設定 `Accept-Encoding` し、応答ヘッダー、サイズ、および本文を調査します。

ヘッダーを使用せずにサンプルアプリに要求を送信 `Accept-Encoding` し、応答が圧縮されていないことを確認します。 `Content-Encoding` `Vary` 応答にヘッダーとヘッダーが存在しません。

![Accept-Encoding ヘッダーのない要求の結果を示す Fiddler ウィンドウ。 応答が圧縮されていません。](response-compression/_static/request-uncompressed.png)

ヘッダーを使用してサンプルアプリに要求を送信 `Accept-Encoding: gzip` し、応答が圧縮されていることを確認します。 `Content-Encoding` `Vary` 応答には、ヘッダーとヘッダーが存在します。

![Accept-Encoding ヘッダーと gzip の値を持つ要求の結果を示す Fiddler ウィンドウ。 Vary ヘッダーと Content-type ヘッダーが応答に追加されます。 応答は圧縮されます。](response-compression/_static/request-compressed.png)

## <a name="providers"></a>プロバイダー

### <a name="gzip-compression-provider"></a>Gzip 圧縮プロバイダー

を使用して、 <xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProvider> [Gzip ファイル形式](https://tools.ietf.org/html/rfc1952)で応答を圧縮します。

圧縮プロバイダーがに明示的に追加されていない場合は、次のように <xref:Microsoft.AspNetCore.ResponseCompression.CompressionProviderCollection> なります。

* Gzip 圧縮プロバイダーは、既定で圧縮プロバイダーの配列に追加されます。
* クライアントで Gzip 圧縮がサポートされている場合、圧縮の既定値は Gzip です。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();
}
```

圧縮プロバイダーが明示的に追加されている場合は、Gzip 圧縮プロバイダーを追加する必要があります。

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=6)]

圧縮レベルをに設定 <xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProviderOptions> します。 Gzip 圧縮プロバイダーは、既定で最も高速な圧縮レベル ([CompressionLevel](xref:System.IO.Compression.CompressionLevel)) に設定されています。これにより、圧縮率が最も高くなる可能性があります。 最も効率的な圧縮が必要な場合は、最適な圧縮のためにミドルウェアを構成します。

| Compression Level | 説明 |
| ----------------- | ----------- |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮は、結果の出力が最適に圧縮されない場合でも、できるだけ早く完了する必要があります。 |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮は実行されません。 |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮が完了するまでに時間がかかる場合でも、応答は最適に圧縮される必要があります。 |

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();

    services.Configure<GzipCompressionProviderOptions>(options => 
    {
        options.Level = CompressionLevel.Fastest;
    });
}
```

### <a name="custom-providers"></a>カスタム プロバイダー

を使用して、カスタム圧縮実装を作成 <xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider> します。 は、 <xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider.EncodingName*> このによって生成されるコンテンツエンコーディングを表し `ICompressionProvider` ます。 ミドルウェアは、この情報を使用して、要求のヘッダーに指定されている一覧に基づいてプロバイダーを選択し `Accept-Encoding` ます。

サンプルアプリを使用して、クライアントはヘッダーを使用して要求を送信し `Accept-Encoding: mycustomcompression` ます。 ミドルウェアはカスタム圧縮実装を使用して、応答をヘッダーと共に返し `Content-Encoding: mycustomcompression` ます。 カスタム圧縮実装を機能させるには、クライアントがカスタムエンコードを圧縮解除できる必要があります。

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=7)]

[!code-csharp[](response-compression/samples/2.x/SampleApp/CustomCompressionProvider.cs?name=snippet1)]

ヘッダーを使用してサンプルアプリに要求を送信し、応答ヘッダーを確認し `Accept-Encoding: mycustomcompression` ます。 `Vary` `Content-Encoding` 応答には、ヘッダーとヘッダーが存在します。 このサンプルでは、応答本文 (表示されません) は圧縮されません。 サンプルのクラスには圧縮実装がありません `CustomCompressionProvider` 。 ただし、このサンプルでは、このような圧縮アルゴリズムを実装する場所を示しています。

![Accept-Encoding ヘッダーと mycustomcompression の値を含む要求の結果を示す Fiddler ウィンドウ。 Vary ヘッダーと Content-type ヘッダーが応答に追加されます。](response-compression/_static/request-custom-compression.png)

## <a name="mime-types"></a>MIME タイプ

ミドルウェアは、圧縮のために既定の MIME の種類のセットを指定します。

* `application/javascript`
* `application/json`
* `application/xml`
* `text/css`
* `text/html`
* `text/json`
* `text/plain`
* `text/xml`

応答の圧縮ミドルウェアオプションを使用して、MIME の種類を置換または追加します。 などのワイルドカードの MIME 型はサポートされていないことに注意して `text/*` ください。 このサンプルアプリでは、の MIME の種類を追加 `image/svg+xml` し、ASP.NET Core バナーイメージ (*横断幕*) を圧縮して提供します。

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=8-10)]

## <a name="compression-with-secure-protocol"></a>セキュリティで保護されたプロトコルを使用した圧縮

セキュリティで保護された接続での圧縮された応答は、 `EnableForHttps` 既定では無効になっているオプションを使用して制御できます。 動的に生成されたページで圧縮を使用すると、 [犯罪](https://wikipedia.org/wiki/CRIME_(security_exploit)) や [侵害](https://wikipedia.org/wiki/BREACH_(security_exploit)) 攻撃などのセキュリティ上の問題が発生する可能性があります。

## <a name="adding-the-vary-header"></a>Vary ヘッダーを追加する

ヘッダーに基づいて応答を圧縮する場合 `Accept-Encoding` 、応答の複数の圧縮バージョンと圧縮されていないバージョンが存在する可能性があります。 複数のバージョンが存在し、格納する必要があることをクライアントキャッシュとプロキシキャッシュに指示するために、 `Vary` ヘッダーに値を追加し `Accept-Encoding` ます。 ASP.NET Core 2.0 以降では、 `Vary` 応答が圧縮されるときに、ミドルウェアによってヘッダーが自動的に追加されます。

## <a name="middleware-issue-when-behind-an-nginx-reverse-proxy"></a>Nginx リバースプロキシの背後にあるミドルウェアの問題

要求が Nginx によってプロキシされている場合、 `Accept-Encoding` ヘッダーは削除されます。 ヘッダーを削除すると、 `Accept-Encoding` ミドルウェアが応答を圧縮できなくなります。 詳細については、「 [NGINX: Compression And 伸張](https://www.nginx.com/resources/admin-guide/compression-and-decompression/)」を参照してください。 この問題は、 [Nginx (aspnet/BasicMiddleware #123) のパススルー圧縮](https://github.com/aspnet/BasicMiddleware/issues/123)によって追跡されます。

## <a name="working-with-iis-dynamic-compression"></a>IIS 動的圧縮の使用

アプリケーションに対して無効にする、アクティブな IIS 動的圧縮モジュールがサーバーレベルで構成されている場合は、 *web.config* ファイルを追加してモジュールを無効にします。 詳細については、「[Disabling IIS modules](xref:host-and-deploy/iis/modules#disabling-iis-modules)」 (IIS モジュールの無効化) を参照してください。

## <a name="troubleshooting"></a>トラブルシューティング

[Fiddler](https://www.telerik.com/fiddler)、[消火バグ](https://getfirebug.com/)、 [Postman](https://www.getpostman.com/)などのツールを使用します。これにより、 `Accept-Encoding` 要求ヘッダーを設定し、応答ヘッダー、サイズ、本文を調べることができます。 既定では、応答圧縮ミドルウェアは、次の条件を満たす応答を圧縮します。

* `Accept-Encoding`ヘッダーには `gzip` 、設定した `*` カスタム圧縮プロバイダーに一致する値、、またはカスタムエンコーディングが含まれています。 値は、 `identity` または品質値 (qvalue, `q` ) の 0 (ゼロ) に設定することはできません。
* MIME の種類 ( `Content-Type` ) を設定し、で構成されている mime の種類と一致させる必要があり <xref:Microsoft.AspNetCore.ResponseCompression.ResponseCompressionOptions> ます。
* 要求にヘッダーを含めることはできません `Content-Range` 。
* 応答圧縮ミドルウェアのオプションで secure protocol (https) が構成されている場合を除き、要求では安全でないプロトコル (http) を使用する必要があります。 *セキュリティで保護されたコンテンツの圧縮を有効にする場合は、 [前述](#compression-with-secure-protocol) の危険に注意してください。*

## <a name="additional-resources"></a>その他のリソース

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* [Mozilla Developer Network: Accept-Encoding](https://developer.mozilla.org/docs/Web/HTTP/Headers/Accept-Encoding)
* [RFC 7231 セクション 3.1.2.1: Content Codings](https://tools.ietf.org/html/rfc7231#section-3.1.2.1)
* [RFC 7230 セクション 4.2.3: Gzip コーディング](https://tools.ietf.org/html/rfc7230#section-4.2.3)
* [GZIP ファイル形式仕様バージョン4.3](https://www.ietf.org/rfc/rfc1952.txt)

::: moniker-end
