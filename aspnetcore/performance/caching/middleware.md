---
title: ASP.NET Core での応答キャッシュミドルウェア
author: rick-anderson
description: ASP.NET Core で応答キャッシュ ミドルウェアを構成し、使用する方法について説明します。
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
uid: performance/caching/middleware
ms.openlocfilehash: 11473bbf8b4e2d67d15798a5d87ee01761682f9a
ms.sourcegitcommit: 07e7ee573fe4e12be93249a385db745d714ff6ae
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/12/2021
ms.locfileid: "103413471"
---
# <a name="response-caching-middleware-in-aspnet-core"></a>ASP.NET Core での応答キャッシュミドルウェア

作成者: [John Luo](https://github.com/JunTaoLuo)

::: moniker range=">= aspnetcore-3.0"

この記事では、ASP.NET Core アプリで応答キャッシュミドルウェアを構成する方法について説明します。 ミドルウェアは、応答をキャッシュ可能にするタイミングを決定し、応答を格納して、キャッシュからの応答を提供します。 HTTP キャッシュと属性の概要につい [`[ResponseCache]`](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute) ては、「 [応答キャッシュ](xref:performance/caching/response)」を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/caching/middleware/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="configuration"></a>構成

応答キャッシュミドルウェアは、共有フレームワークを介して ASP.NET Core アプリで暗黙的に使用できます。

で `Startup.ConfigureServices` 、応答キャッシュミドルウェアをサービスコレクションに追加します。

[!code-csharp[](middleware/samples/3.x/ResponseCachingMiddleware/Startup.cs?name=snippet1&highlight=3)]

拡張メソッドと共にミドルウェアを使用するようにアプリを構成し <xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching*> ます。これにより、の要求処理パイプラインにミドルウェアが追加され `Startup.Configure` ます。

[!code-csharp[](middleware/samples/3.x/ResponseCachingMiddleware/Startup.cs?name=snippet2&highlight=17)]

> [!WARNING]
> <xref:Microsoft.AspNetCore.Builder.CorsMiddlewareExtensions.UseCors%2A><xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching%2A> [CORS ミドルウェア](xref:security/cors)を使用する場合は、前にを呼び出す必要があります。

サンプルアプリでは、後続の要求でキャッシュを制御するためのヘッダーを追加します。

* [Cache-control](https://tools.ietf.org/html/rfc7234#section-5.2): キャッシュ可能な応答を最大10秒間キャッシュします。
* [Vary](https://tools.ietf.org/html/rfc7231#section-7.1.4): 後続の要求の [エンコーディング](https://tools.ietf.org/html/rfc7231#section-5.3.4) ヘッダーが元の要求と一致する場合にのみ、キャッシュされた応答を提供するようにミドルウェアを構成します。

[!code-csharp[](middleware/samples_snippets/3.x/AddHeaders.cs)]

前のヘッダーは応答に書き込まれず、コントローラー、アクション、またはページでオーバーライドされ Razor ます。

* には [[ResponseCache]](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute) 属性があります。 これは、プロパティが設定されていない場合でも適用されます。 たとえば、 [VaryByHeader](./response.md#vary) プロパティを省略すると、対応するヘッダーが応答から削除されます。

応答キャッシュミドルウェアは、200 (OK) 状態コードを生成するサーバー応答のみをキャッシュします。 [エラーページ](xref:fundamentals/error-handling)を含むその他の応答は、ミドルウェアによって無視されます。

> [!WARNING]
> 認証されたクライアントのコンテンツを含む応答は、ミドルウェアがそれらの応答を格納してサービスを提供しないように、キャッシュ不可能としてマークする必要があります。 応答がキャッシュ可能かどうかをミドルウェアが決定する方法の詳細については、「 [キャッシュの条件](#conditions-for-caching) 」を参照してください。

## <a name="options"></a>Options

応答キャッシュのオプションを次の表に示します。

| オプション | 説明 |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.MaximumBodySize> | 応答本文の最大キャッシュ可能サイズ (バイト単位)。 既定値は `64 * 1024 * 1024` (64 MB) です。 |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.SizeLimit> | 応答キャッシュミドルウェアのサイズ制限 (バイト単位)。 既定値は `100 * 1024 * 1024` (100 MB) です。 |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.UseCaseSensitivePaths> | 大文字と小文字が区別されるパスに応答をキャッシュするかどうかを決定します。 既定値は `false` です。 |

次の例では、ミドルウェアをに構成します。

* 本文のサイズが1024バイト以下の応答をキャッシュします。
* 大文字と小文字を区別するパスで応答を格納します。 たとえば、 `/page1` と `/Page1` は別々に格納されます。

```csharp
services.AddResponseCaching(options =>
{
    options.MaximumBodySize = 1024;
    options.UseCaseSensitivePaths = true;
});
```

## <a name="varybyquerykeys"></a>VaryByQueryKeys

MVC/web API コントローラーまたは Razor ページのページモデルを使用する場合、属性は、 [`[ResponseCache]`](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute) 応答のキャッシュに適切なヘッダーを設定するために必要なパラメーターを指定します。 ミドルウェアを厳密に必要とする属性の唯一のパラメーター `[ResponseCache]` は <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute.VaryByQueryKeys> 、実際の HTTP ヘッダーに対応していません。 詳細については、「<xref:performance/caching/response#responsecache-attribute>」を参照してください。

属性を使用しない場合は、を使用して `[ResponseCache]` 応答のキャッシュを変化させることができ `VaryByQueryKeys` ます。 を <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingFeature> HttpContext から直接使用し[ます。](xref:Microsoft.AspNetCore.Http.HttpContext.Features)

```csharp
var responseCachingFeature = context.HttpContext.Features.Get<IResponseCachingFeature>();

if (responseCachingFeature != null)
{
    responseCachingFeature.VaryByQueryKeys = new[] { "MyKey" };
}
```

で1つの値を使用すると `*` `VaryByQueryKeys` 、すべての要求クエリパラメーターによってキャッシュが異なります。

## <a name="http-headers-used-by-response-caching-middleware"></a>応答キャッシュミドルウェアによって使用される HTTP ヘッダー

次の表は、応答のキャッシュに影響を与える HTTP ヘッダーに関する情報を示しています。

| ヘッダー | 詳細情報 |
| ------ | ------- |
| `Authorization` | ヘッダーが存在する場合、応答はキャッシュされません。 |
| `Cache-Control` | ミドルウェアは、cache ディレクティブでマークされたキャッシュ応答のみを考慮し `public` ます。 次のパラメーターを使用してキャッシュを制御します。<ul><li>最長有効期間</li><li>最大-古い&#8224;</li><li>最小-新規</li><li>must-revalidate</li><li>no-cache</li><li>ストアなし</li><li>-if-キャッシュ済み</li><li>プライベート</li><li>public</li><li>s-maxage</li><li>プロキシ再検証&#8225;</li></ul>&#8224;に制限が指定されていない場合 `max-stale` 、ミドルウェアは何も実行しません。<br>&#8225;`proxy-revalidate` はと同じ効果があり `must-revalidate` ます。<br><br>詳細については、「 [RFC 7231: Request Cache-Control ディレクティブ](https://tools.ietf.org/html/rfc7234#section-5.2.1)」を参照してください。 |
| `Pragma` | 要求のヘッダーでは、 `Pragma: no-cache` と同じ効果が得られ `Cache-Control: no-cache` ます。 このヘッダーは、ヘッダー内の関連するディレクティブによってオーバーライドされ `Cache-Control` ます (存在する場合)。 HTTP/1.0 との下位互換性のために考慮されます。 |
| `Set-Cookie` | ヘッダーが存在する場合、応答はキャッシュされません。 1つ以上のを設定する要求処理パイプライン内のすべてのミドルウェア cookie は、応答キャッシュミドルウェアが応答をキャッシュしないようにします (たとえば、 [ cookie ベースの tempdata プロバイダー](xref:fundamentals/app-state#tempdata))。  |
| `Vary` | `Vary`ヘッダーは、キャッシュされた応答を別のヘッダーによって変更するために使用されます。 たとえば、ヘッダーを含めることによって、エンコードによって応答をキャッシュし `Vary: Accept-Encoding` ます。ヘッダーとは別に要求の応答をキャッシュし `Accept-Encoding: gzip` `Accept-Encoding: text/plain` ます。 ヘッダー値がの応答 `*` は格納されません。 |
| `Expires` | このヘッダーによって古いと見なされる応答は、他のヘッダーでオーバーライドされない限り、格納または取得されません `Cache-Control` 。 |
| `If-None-Match` | 値がではなく、 `*` 応答のが指定された値と一致しない場合は、完全な応答がキャッシュから提供され `ETag` ます。 それ以外の場合は、304 (変更なし) の応答が処理されます。 |
| `If-Modified-Since` | ヘッダーが存在しない場合、キャッシュされた `If-None-Match` 応答の日付が指定した値より新しい場合は、完全な応答がキャッシュから提供されます。 それ以外の場合は、 *304-変更されていない* 応答が提供されます。 |
| `Date` | キャッシュからサービスを提供している場合、 `Date` ヘッダーはミドルウェアによって設定されます (元の応答で指定されていない場合)。 |
| `Content-Length` | キャッシュからサービスを提供している場合、 `Content-Length` ヘッダーはミドルウェアによって設定されます (元の応答で指定されていない場合)。 |
| `Age` | `Age`元の応答で送信されたヘッダーは無視されます。 ミドルウェアは、キャッシュされた応答を提供するときに新しい値を計算します。 |

## <a name="caching-respects-request-cache-control-directives"></a>キャッシュは要求 Cache-Control ディレクティブを尊重します

ミドルウェアは、 [HTTP 1.1 キャッシュ仕様](https://tools.ietf.org/html/rfc7234#section-5.2)の規則を尊重します。 規則では、 `Cache-Control` クライアントによって送信された有効なヘッダーを受け入れるためにキャッシュが必要です。 この仕様では、クライアントはヘッダー値を使用して要求を行い、 `no-cache` すべての要求に対してサーバーに新しい応答を強制的に生成させることができます。 現時点では、ミドルウェアが公式のキャッシュ仕様に準拠しているため、ミドルウェアを使用する場合、このキャッシュ動作を開発者が制御することはありません。

キャッシュの動作を詳細に制御するには、ASP.NET Core の他のキャッシュ機能を調べます。 次のトピックを参照してください。

* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>

## <a name="troubleshooting"></a>トラブルシューティング

キャッシュの動作が想定どおりでない場合は、応答がキャッシュ可能であり、キャッシュから提供できることを確認します。 要求の受信ヘッダーと応答の送信ヘッダーを確認します。 デバッグに役立つように [ログ記録](xref:fundamentals/logging/index) を有効にします。

キャッシュ動作のテストとトラブルシューティングを行う場合、ブラウザーでは、望ましくない方法でキャッシュに影響を与える要求ヘッダーを設定できます。 たとえば、ページを更新するときに、ブラウザーで `Cache-Control` ヘッダーをまたはに設定でき `no-cache` `max-age=0` ます。 次のツールでは、要求ヘッダーを明示的に設定でき、キャッシュのテストに適しています。

* [Fiddler](https://www.telerik.com/fiddler)
* [Postman](https://www.getpostman.com/)

### <a name="conditions-for-caching"></a>キャッシュの条件

* 要求は、200 (OK) 状態コードでサーバー応答を生成する必要があります。
* 要求メソッドは GET または HEAD である必要があります。
* では `Startup.Configure` 、応答キャッシュミドルウェアは、キャッシュを必要とするミドルウェアの前に配置する必要があります。 詳細については、「<xref:fundamentals/middleware/index>」を参照してください。
* ヘッダーを指定することはでき `Authorization` ません。
* `Cache-Control` ヘッダーパラメーターは有効である必要があり、応答はとマークされている必要があり `public` `private` ます。
* ヘッダーが存在する場合、ヘッダーはヘッダーをオーバーライドするので、ヘッダーが存在しない場合はヘッダーが `Pragma: no-cache` 存在しない必要があり `Cache-Control` `Cache-Control` `Pragma` ます。
* ヘッダーを指定することはでき `Set-Cookie` ません。
* `Vary` ヘッダーパラメーターはと同じである必要があり `*` ます。
* `Content-Length`ヘッダー値 (設定されている場合) は、応答本文のサイズと一致する必要があります。
* は <xref:Microsoft.AspNetCore.Http.Features.IHttpSendFileFeature> 使用されません。
* `Expires`ヘッダー、および、およびの各 `max-age` キャッシュディレクティブで指定されているとおり、応答を古いものにすることはできません `s-maxage` 。
* 応答バッファリングを成功させる必要があります。 応答のサイズは、構成済みまたは既定値よりも小さくする必要があり <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.SizeLimit> ます。 応答の本文のサイズは、構成済みまたは既定値よりも小さくする必要があり <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.MaximumBodySize> ます。
* 応答は、 [RFC 7234](https://tools.ietf.org/html/rfc7234) 仕様に従ってキャッシュ可能である必要があります。 たとえば、ディレクティブは、 `no-store` 要求または応答のヘッダーフィールドに存在することはできません。 詳細については、 *「セクション 3:* [RFC 7234](https://tools.ietf.org/html/rfc7234) のキャッシュに応答を格納する」を参照してください。

> [!NOTE]
> クロスサイト要求偽造 (CSRF) 攻撃を防ぐために、セキュリティで保護されたトークンを生成するための偽造防止システムは、 `Cache-Control` `Pragma` 応答がキャッシュされないようにヘッダーとヘッダーをに設定し `no-cache` ます。 HTML フォーム要素の偽造防止トークンを無効にする方法については、「」を参照してください <xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration> 。

## <a name="additional-resources"></a>その他のリソース

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

この記事では、ASP.NET Core アプリで応答キャッシュミドルウェアを構成する方法について説明します。 ミドルウェアは、応答をキャッシュ可能にするタイミングを決定し、応答を格納して、キャッシュからの応答を提供します。 HTTP キャッシュと属性の概要につい [`[ResponseCache]`](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute) ては、「 [応答キャッシュ](xref:performance/caching/response)」を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/caching/middleware/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="configuration"></a>構成

[AspNetCore メタパッケージ](xref:fundamentals/metapackage-app)を使用するか、 [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCaching/)パッケージへのパッケージ参照を追加します。

で `Startup.ConfigureServices` 、応答キャッシュミドルウェアをサービスコレクションに追加します。

[!code-csharp[](middleware/samples/2.x/ResponseCachingMiddleware/Startup.cs?name=snippet1&highlight=3)]

拡張メソッドと共にミドルウェアを使用するようにアプリを構成し <xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching*> ます。これにより、の要求処理パイプラインにミドルウェアが追加され `Startup.Configure` ます。

[!code-csharp[](middleware/samples/2.x/ResponseCachingMiddleware/Startup.cs?name=snippet2&highlight=14)]

サンプルアプリでは、後続の要求でキャッシュを制御するためのヘッダーを追加します。

* [Cache-control](https://tools.ietf.org/html/rfc7234#section-5.2): キャッシュ可能な応答を最大10秒間キャッシュします。
* [Vary](https://tools.ietf.org/html/rfc7231#section-7.1.4): 後続の要求の [エンコーディング](https://tools.ietf.org/html/rfc7231#section-5.3.4) ヘッダーが元の要求と一致する場合にのみ、キャッシュされた応答を提供するようにミドルウェアを構成します。

[!code-csharp[](middleware/samples_snippets/2.x/AddHeaders.cs)]

前のヘッダーは応答に書き込まれず、コントローラー、アクション、またはページでオーバーライドされ Razor ます。

* には [[ResponseCache]](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute) 属性があります。 これは、プロパティが設定されていない場合でも適用されます。 たとえば、 [VaryByHeader](./response.md#vary) プロパティを省略すると、対応するヘッダーが応答から削除されます。

応答キャッシュミドルウェアは、200 (OK) 状態コードを生成するサーバー応答のみをキャッシュします。 [エラーページ](xref:fundamentals/error-handling)を含むその他の応答は、ミドルウェアによって無視されます。

> [!WARNING]
> 認証されたクライアントのコンテンツを含む応答は、ミドルウェアがそれらの応答を格納してサービスを提供しないように、キャッシュ不可能としてマークする必要があります。 応答がキャッシュ可能かどうかをミドルウェアが決定する方法の詳細については、「 [キャッシュの条件](#conditions-for-caching) 」を参照してください。

## <a name="options"></a>Options

応答キャッシュのオプションを次の表に示します。

| オプション | 説明 |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.MaximumBodySize> | 応答本文の最大キャッシュ可能サイズ (バイト単位)。 既定値は `64 * 1024 * 1024` (64 MB) です。 |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.SizeLimit> | 応答キャッシュミドルウェアのサイズ制限 (バイト単位)。 既定値は `100 * 1024 * 1024` (100 MB) です。 |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.UseCaseSensitivePaths> | 大文字と小文字が区別されるパスに応答をキャッシュするかどうかを決定します。 既定値は `false` です。 |

次の例では、ミドルウェアをに構成します。

* 本文のサイズが1024バイト以下の応答をキャッシュします。
* 大文字と小文字を区別するパスで応答を格納します。 たとえば、 `/page1` と `/Page1` は別々に格納されます。

```csharp
services.AddResponseCaching(options =>
{
    options.MaximumBodySize = 1024;
    options.UseCaseSensitivePaths = true;
});
```

## <a name="varybyquerykeys"></a>VaryByQueryKeys

MVC/web API コントローラーまたは Razor ページのページモデルを使用する場合、属性は、 [`[ResponseCache]`](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute) 応答のキャッシュに適切なヘッダーを設定するために必要なパラメーターを指定します。 ミドルウェアを厳密に必要とする属性の唯一のパラメーター `[ResponseCache]` は <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute.VaryByQueryKeys> 、実際の HTTP ヘッダーに対応していません。 詳細については、「<xref:performance/caching/response#responsecache-attribute>」を参照してください。

属性を使用しない場合は、を使用して `[ResponseCache]` 応答のキャッシュを変化させることができ `VaryByQueryKeys` ます。 を <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingFeature> HttpContext から直接使用し[ます。](xref:Microsoft.AspNetCore.Http.HttpContext.Features)

```csharp
var responseCachingFeature = context.HttpContext.Features.Get<IResponseCachingFeature>();

if (responseCachingFeature != null)
{
    responseCachingFeature.VaryByQueryKeys = new[] { "MyKey" };
}
```

で1つの値を使用すると `*` `VaryByQueryKeys` 、すべての要求クエリパラメーターによってキャッシュが異なります。

## <a name="http-headers-used-by-response-caching-middleware"></a>応答キャッシュミドルウェアによって使用される HTTP ヘッダー

次の表は、応答のキャッシュに影響を与える HTTP ヘッダーに関する情報を示しています。

| ヘッダー | 詳細情報 |
| ------ | ------- |
| `Authorization` | ヘッダーが存在する場合、応答はキャッシュされません。 |
| `Cache-Control` | ミドルウェアは、cache ディレクティブでマークされたキャッシュ応答のみを考慮し `public` ます。 次のパラメーターを使用してキャッシュを制御します。<ul><li>最長有効期間</li><li>最大-古い&#8224;</li><li>最小-新規</li><li>must-revalidate</li><li>no-cache</li><li>ストアなし</li><li>-if-キャッシュ済み</li><li>プライベート</li><li>public</li><li>s-maxage</li><li>プロキシ再検証&#8225;</li></ul>&#8224;に制限が指定されていない場合 `max-stale` 、ミドルウェアは何も実行しません。<br>&#8225;`proxy-revalidate` はと同じ効果があり `must-revalidate` ます。<br><br>詳細については、「 [RFC 7231: Request Cache-Control ディレクティブ](https://tools.ietf.org/html/rfc7234#section-5.2.1)」を参照してください。 |
| `Pragma` | 要求のヘッダーでは、 `Pragma: no-cache` と同じ効果が得られ `Cache-Control: no-cache` ます。 このヘッダーは、ヘッダー内の関連するディレクティブによってオーバーライドされ `Cache-Control` ます (存在する場合)。 HTTP/1.0 との下位互換性のために考慮されます。 |
| `Set-Cookie` | ヘッダーが存在する場合、応答はキャッシュされません。 1つ以上のを設定する要求処理パイプライン内のすべてのミドルウェア cookie は、応答キャッシュミドルウェアが応答をキャッシュしないようにします (たとえば、 [ cookie ベースの tempdata プロバイダー](xref:fundamentals/app-state#tempdata))。  |
| `Vary` | `Vary`ヘッダーは、キャッシュされた応答を別のヘッダーによって変更するために使用されます。 たとえば、ヘッダーを含めることによって、エンコードによって応答をキャッシュし `Vary: Accept-Encoding` ます。ヘッダーとは別に要求の応答をキャッシュし `Accept-Encoding: gzip` `Accept-Encoding: text/plain` ます。 ヘッダー値がの応答 `*` は格納されません。 |
| `Expires` | このヘッダーによって古いと見なされる応答は、他のヘッダーでオーバーライドされない限り、格納または取得されません `Cache-Control` 。 |
| `If-None-Match` | 値がではなく、 `*` 応答のが指定された値と一致しない場合は、完全な応答がキャッシュから提供され `ETag` ます。 それ以外の場合は、304 (変更なし) の応答が処理されます。 |
| `If-Modified-Since` | ヘッダーが存在しない場合、キャッシュされた `If-None-Match` 応答の日付が指定した値より新しい場合は、完全な応答がキャッシュから提供されます。 それ以外の場合は、 *304-変更されていない* 応答が提供されます。 |
| `Date` | キャッシュからサービスを提供している場合、 `Date` ヘッダーはミドルウェアによって設定されます (元の応答で指定されていない場合)。 |
| `Content-Length` | キャッシュからサービスを提供している場合、 `Content-Length` ヘッダーはミドルウェアによって設定されます (元の応答で指定されていない場合)。 |
| `Age` | `Age`元の応答で送信されたヘッダーは無視されます。 ミドルウェアは、キャッシュされた応答を提供するときに新しい値を計算します。 |

## <a name="caching-respects-request-cache-control-directives"></a>キャッシュは要求 Cache-Control ディレクティブを尊重します

ミドルウェアは、 [HTTP 1.1 キャッシュ仕様](https://tools.ietf.org/html/rfc7234#section-5.2)の規則を尊重します。 規則では、 `Cache-Control` クライアントによって送信された有効なヘッダーを受け入れるためにキャッシュが必要です。 この仕様では、クライアントはヘッダー値を使用して要求を行い、 `no-cache` すべての要求に対してサーバーに新しい応答を強制的に生成させることができます。 現時点では、ミドルウェアが公式のキャッシュ仕様に準拠しているため、ミドルウェアを使用する場合、このキャッシュ動作を開発者が制御することはありません。

キャッシュの動作を詳細に制御するには、ASP.NET Core の他のキャッシュ機能を調べます。 次のトピックを参照してください。

* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>

## <a name="troubleshooting"></a>トラブルシューティング

キャッシュの動作が想定どおりでない場合は、応答がキャッシュ可能であり、キャッシュから提供できることを確認します。 要求の受信ヘッダーと応答の送信ヘッダーを確認します。 デバッグに役立つように [ログ記録](xref:fundamentals/logging/index) を有効にします。

キャッシュ動作のテストとトラブルシューティングを行う場合、ブラウザーでは、望ましくない方法でキャッシュに影響を与える要求ヘッダーを設定できます。 たとえば、ページを更新するときに、ブラウザーで `Cache-Control` ヘッダーをまたはに設定でき `no-cache` `max-age=0` ます。 次のツールでは、要求ヘッダーを明示的に設定でき、キャッシュのテストに適しています。

* [Fiddler](https://www.telerik.com/fiddler)
* [Postman](https://www.getpostman.com/)

### <a name="conditions-for-caching"></a>キャッシュの条件

* 要求は、200 (OK) 状態コードでサーバー応答を生成する必要があります。
* 要求メソッドは GET または HEAD である必要があります。
* では `Startup.Configure` 、応答キャッシュミドルウェアは、キャッシュを必要とするミドルウェアの前に配置する必要があります。 詳細については、「<xref:fundamentals/middleware/index>」を参照してください。
* ヘッダーを指定することはでき `Authorization` ません。
* `Cache-Control` ヘッダーパラメーターは有効である必要があり、応答はとマークされている必要があり `public` `private` ます。
* ヘッダーが存在する場合、ヘッダーはヘッダーをオーバーライドするので、ヘッダーが存在しない場合はヘッダーが `Pragma: no-cache` 存在しない必要があり `Cache-Control` `Cache-Control` `Pragma` ます。
* ヘッダーを指定することはでき `Set-Cookie` ません。
* `Vary` ヘッダーパラメーターはと同じである必要があり `*` ます。
* `Content-Length`ヘッダー値 (設定されている場合) は、応答本文のサイズと一致する必要があります。
* は <xref:Microsoft.AspNetCore.Http.Features.IHttpSendFileFeature> 使用されません。
* `Expires`ヘッダー、および、およびの各 `max-age` キャッシュディレクティブで指定されているとおり、応答を古いものにすることはできません `s-maxage` 。
* 応答バッファリングを成功させる必要があります。 応答のサイズは、構成済みまたは既定値よりも小さくする必要があり <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.SizeLimit> ます。 応答の本文のサイズは、構成済みまたは既定値よりも小さくする必要があり <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.MaximumBodySize> ます。
* 応答は、 [RFC 7234](https://tools.ietf.org/html/rfc7234) 仕様に従ってキャッシュ可能である必要があります。 たとえば、ディレクティブは、 `no-store` 要求または応答のヘッダーフィールドに存在することはできません。 詳細については、 *「セクション 3:* [RFC 7234](https://tools.ietf.org/html/rfc7234) のキャッシュに応答を格納する」を参照してください。

> [!NOTE]
> クロスサイト要求偽造 (CSRF) 攻撃を防ぐために、セキュリティで保護されたトークンを生成するための偽造防止システムは、 `Cache-Control` `Pragma` 応答がキャッシュされないようにヘッダーとヘッダーをに設定し `no-cache` ます。 HTML フォーム要素の偽造防止トークンを無効にする方法については、「」を参照してください <xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration> 。

## <a name="additional-resources"></a>その他のリソース

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>

::: moniker-end