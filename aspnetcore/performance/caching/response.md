---
title: ASP.NET Core での応答のキャッシュ
author: rick-anderson
description: 応答キャッシュを使用し、帯域幅要件を下げ、ASP.NET Core アプリのパフォーマンスを上げる方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 11/04/2019
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
uid: performance/caching/response
ms.openlocfilehash: 539ddb118279adb3a53394cdb0c2e5169092ebc0
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589232"
---
# <a name="response-caching-in-aspnet-core"></a>ASP.NET Core での応答のキャッシュ

By [John Luo](https://github.com/JunTaoLuo)、 [Rick Anderson](https://twitter.com/RickAndMSFT)、および [上田 Smith](https://ardalis.com/)

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/caching/response/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

応答キャッシュを使用すると、クライアントまたはプロキシが web サーバーに対して行う要求の数を減らすことができます。 応答キャッシュを使用すると、web サーバーが応答を生成するために実行する作業量も少なくなります。 応答のキャッシュは、クライアント、プロキシ、およびミドルウェアが応答をキャッシュする方法を指定するヘッダーによって制御されます。

[ResponseCache 属性](#responsecache-attribute)は、応答キャッシュヘッダーの設定に関与します。 クライアントと中間プロキシは、 [HTTP 1.1 キャッシュ仕様](https://tools.ietf.org/html/rfc7234)での応答をキャッシュするためのヘッダーを優先する必要があります。

HTTP 1.1 キャッシュ仕様に従ったサーバー側キャッシュの場合は、 [応答キャッシュミドルウェア](xref:performance/caching/middleware)を使用します。 ミドルウェアは、プロパティを使用して、 <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> サーバー側のキャッシュ動作に影響を与えることができます。

## <a name="http-based-response-caching"></a>HTTP ベースの応答のキャッシュ

[HTTP 1.1 キャッシュ仕様](https://tools.ietf.org/html/rfc7234)では、インターネットキャッシュの動作方法について説明します。 キャッシュに使用されるプライマリ HTTP ヘッダーは [cache-control](https://tools.ietf.org/html/rfc7234#section-5.2)で、キャッシュ *ディレクティブ* を指定するために使用されます。 ディレクティブは、要求に応じてキャッシュ動作を制御し、応答としてサーバーからクライアントへの応答を行います。 要求と応答はプロキシサーバーを経由して移動し、プロキシサーバーも HTTP 1.1 キャッシュ仕様に準拠している必要があります。

共通の `Cache-Control` ディレクティブを次の表に示します。

| ディレクティブ                                                       | アクション |
| --------------------------------------------------------------- | ------ |
| [public](https://tools.ietf.org/html/rfc7234#section-5.2.2.5)   | キャッシュは応答を格納できます。 |
| [private](https://tools.ietf.org/html/rfc7234#section-5.2.2.6)  | 応答は、共有キャッシュによって格納されていない必要があります。 プライベートキャッシュは、応答を格納して再利用できます。 |
| [最長有効期間](https://tools.ietf.org/html/rfc7234#section-5.2.1.1)  | クライアントは、指定された秒数よりも有効期間が長い応答を受け入れません。 例: `max-age=60` (60 秒)、 `max-age=2592000` (1 か月) |
| [キャッシュなし](https://tools.ietf.org/html/rfc7234#section-5.2.1.4) | **要求時**: キャッシュは、要求を満たすために格納された応答を使用することはできません。 配信元サーバーはクライアントの応答を再生成し、ミドルウェアはキャッシュに格納されている応答を更新します。<br><br>**応答時**: 配信元サーバーで検証を行わずに、後続の要求に応答を使用することはできません。 |
| [ストアなし](https://tools.ietf.org/html/rfc7234#section-5.2.1.5) | **要求時**: キャッシュは要求を格納できません。<br><br>**応答** の場合: キャッシュは、応答の一部を格納することはできません。 |

キャッシュでロールを果たすその他のキャッシュヘッダーを次の表に示します。

| ヘッダー                                                     | 機能 |
| ---------------------------------------------------------- | -------- |
| [Age](https://tools.ietf.org/html/rfc7234#section-5.1)     | 配信元サーバーで応答が生成または正常に検証されてからの、秒単位の推定時間。 |
| [経過](https://tools.ietf.org/html/rfc7234#section-5.3) | 応答が古くなったと見なされるまでの時間。 |
| [Unmanaged](https://tools.ietf.org/html/rfc7234#section-5.4)  | 動作を設定するために HTTP/1.0 キャッシュとの下位互換性を維持するために存在し `no-cache` ます。 `Cache-Control`ヘッダーが存在する場合、 `Pragma` ヘッダーは無視されます。 |
| [要因](https://tools.ietf.org/html/rfc7231#section-7.1.4)  | キャッシュされた `Vary` 応答の元の要求と新しい要求の両方ですべてのヘッダーフィールドが一致する場合を除き、キャッシュされた応答を送信しないように指定します。 |

## <a name="http-based-caching-respects-request-cache-control-directives"></a>HTTP ベースのキャッシュは、要求 Cache-Control ディレクティブを尊重します

[Cache-Control ヘッダーの HTTP 1.1 キャッシュ仕様](https://tools.ietf.org/html/rfc7234#section-5.2)では、 `Cache-Control` クライアントによって送信された有効なヘッダーを受け入れるためにキャッシュが必要です。 クライアントは、ヘッダー値を使用して要求を行い、 `no-cache` すべての要求に対して新しい応答を強制的に生成するようにサーバーに要求できます。

`Cache-Control`HTTP キャッシュの目的を検討する場合は、常にクライアント要求ヘッダーを使用することをお勧めします。 公式仕様では、キャッシュは、クライアント、プロキシ、およびサーバーのネットワーク経由で要求を満たすことの待機時間とネットワークオーバーヘッドを削減することを目的としています。 配信元サーバーの負荷を制御する方法であるとは限りません。

ミドルウェアが公式のキャッシュ仕様に準拠しているため、 [応答キャッシュミドルウェア](xref:performance/caching/middleware) を使用する場合、開発者はこのキャッシュ動作を制御することはできません。 [ミドルウェアの計画](https://github.com/dotnet/AspNetCore/issues/2612) された拡張機能は、キャッシュされ `Cache-Control` た応答を提供することを決定するときに、要求のヘッダーを無視するようにミドルウェアを構成する機会です。 計画された拡張機能により、サーバーの負荷をより適切に制御できるようになります。

## <a name="other-caching-technology-in-aspnet-core"></a>ASP.NET Core のその他のキャッシュテクノロジ

### <a name="in-memory-caching"></a>メモリ内キャッシュ

インメモリキャッシュは、キャッシュされたデータを格納するためにサーバーメモリを使用します。 この種のキャッシュは、1台のサーバーまたは *固定セッション* を使用している複数のサーバーに適しています。 固定セッションとは、クライアントからの要求が常に同じサーバーにルーティングされて処理されることを意味します。

詳細については、「<xref:performance/caching/memory>」を参照してください。

### <a name="distributed-cache"></a>分散キャッシュ

アプリがクラウドまたはサーバーファームでホストされている場合は、分散キャッシュを使用してデータをメモリに格納します。 キャッシュは、要求を処理するサーバー間で共有されます。 クライアントのキャッシュデータが使用可能な場合、クライアントは、グループ内の任意のサーバーによって処理される要求を送信できます。 ASP.NET Core は、SQL Server、 [Redis](https://www.nuget.org/packages/Microsoft.Extensions.Caching.StackExchangeRedis)、および [ncache](https://www.nuget.org/packages/Alachisoft.NCache.OpenSource.SDK/) 分散キャッシュと連動します。

詳細については、「<xref:performance/caching/distributed>」を参照してください。

### <a name="cache-tag-helper"></a>キャッシュ タグ ヘルパー

キャッシュタグヘルパーを使用して、MVC ビューまたはページからコンテンツをキャッシュし Razor ます。 キャッシュタグヘルパーは、メモリ内キャッシュを使用してデータを格納します。

詳細については、「<xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>」を参照してください。

### <a name="distributed-cache-tag-helper"></a>分散キャッシュ タグ ヘルパー

分散 Razor キャッシュタグヘルパーを使用して、分散型クラウドまたは web ファームのシナリオで、MVC ビューまたはページからコンテンツをキャッシュします。 分散キャッシュタグヘルパーは、SQL Server、 [Redis](https://www.nuget.org/packages/Microsoft.Extensions.Caching.StackExchangeRedis)、または [ncache](https://www.nuget.org/packages/Alachisoft.NCache.OpenSource.SDK/) を使用してデータを格納します。

詳細については、「<xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>」を参照してください。

## <a name="responsecache-attribute"></a>ResponseCache 属性

は、 <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> 応答キャッシュに適切なヘッダーを設定するために必要なパラメーターを指定します。

> [!WARNING]
> 認証されたクライアントの情報が含まれているコンテンツのキャッシュを無効にします。 キャッシュは、ユーザーの id またはユーザーがサインインしているかどうかによって変更されないコンテンツに対してのみ有効にする必要があります。

<xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByQueryKeys> 指定されたクエリキーのリストの値によって、格納されている応答を変化させる。 1つの値を指定した場合 `*` 、ミドルウェアはすべての要求クエリ文字列パラメーターによって応答を変化させることができます。

プロパティを設定するには、[応答キャッシュミドルウェア](xref:performance/caching/middleware)を有効にする必要があり <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByQueryKeys> ます。 それ以外の場合は、ランタイム例外がスローされます。 プロパティに対応する HTTP ヘッダーがありません <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByQueryKeys> 。 プロパティは、応答キャッシュミドルウェアによって処理される HTTP 機能です。 ミドルウェアがキャッシュされた応答を提供するには、クエリ文字列とクエリ文字列の値が以前の要求と一致している必要があります。 たとえば、次の表に示すような一連の要求と結果を考えてみましょう。

| Request                          | 結果                    |
| -------------------------------- | ------------------------- |
| `http://example.com?key1=value1` | サーバーから返されます。 |
| `http://example.com?key1=value1` | ミドルウェアから返されます。 |
| `http://example.com?key1=value2` | サーバーから返されます。 |

最初の要求はサーバーによって返され、ミドルウェアにキャッシュされます。 2番目の要求は、クエリ文字列が前の要求と一致するため、ミドルウェアによって返されます。 クエリ文字列値が以前の要求と一致しないので、3番目の要求がミドルウェアキャッシュにありません。

は、 <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> を使用してを構成および作成するために使用され <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> `Microsoft.AspNetCore.Mvc.Internal.ResponseCacheFilter` ます。 は、 `ResponseCacheFilter` 適切な HTTP ヘッダーと応答の機能を更新する作業を実行します。 フィルター:

* 、、およびの既存のヘッダーを削除 `Vary` `Cache-Control` `Pragma` します。
* で設定されたプロパティに基づいて、適切なヘッダーを書き込み <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> ます。
* が設定されている場合は、応答キャッシュ HTTP 機能を更新し <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByQueryKeys> ます。

### <a name="vary"></a>要因

このヘッダーは、プロパティが設定されている場合にのみ書き込まれ <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByHeader> ます。 プロパティの値に設定されたプロパティ `Vary` 。 次の例では、プロパティを使用し <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByHeader> ます。

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Pages/Cache1.cshtml.cs?name=snippet)]

サンプルアプリを使用して、ブラウザーのネットワークツールで応答ヘッダーを表示します。 次の応答ヘッダーは、Cache1 ページの応答と共に送信されます。

```
Cache-Control: public,max-age=30
Vary: User-Agent
```

### <a name="nostore-and-locationnone"></a>NoStore と Location。なし

<xref:Microsoft.AspNetCore.Mvc.CacheProfile.NoStore> その他のほとんどのプロパティをオーバーライドします。 このプロパティがに設定されている場合 `true` 、 `Cache-Control` ヘッダーはに設定され `no-store` ます。 <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location>がに設定されている場合 `None` :

* `Cache-Control` が `no-store,no-cache` に設定されます。
* `Pragma` が `no-cache` に設定されます。

がで、がである場合、 <xref:Microsoft.AspNetCore.Mvc.CacheProfile.NoStore> `false` およびは <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> `None` `Cache-Control` `Pragma` に設定され `no-cache` ます。

<xref:Microsoft.AspNetCore.Mvc.CacheProfile.NoStore> は、通常、 `true` エラーページに対してに設定されます。 サンプルアプリの Cache2 ページには、応答を格納しないようにクライアントに指示する応答ヘッダーが生成されます。

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Pages/Cache2.cshtml.cs?name=snippet)]

このサンプルアプリでは、次のヘッダーを含む Cache2 ページが返されます。

```
Cache-Control: no-store,no-cache
Pragma: no-cache
```

### <a name="location-and-duration"></a>場所と期間

キャッシュを有効にするには、を <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Duration> 正の値に設定し、 <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> (既定値) またはのいずれかにする必要があり `Any` `Client` ます。 フレームワークによって、ヘッダーは location 値に設定され、その `Cache-Control` 後に `max-age` 応答のが続きます。

<xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location>との各オプションは、 `Any` `Client` `Cache-Control` それぞれおよびのヘッダー値に変換さ `public` `private` れます。 [Nostore と Location. None](#nostore-and-locationnone)セクションに記載されているよう <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> に、をに設定する `None` と、との両方の `Cache-Control` `Pragma` ヘッダーがに設定さ `no-cache` れます。

`Location.Any` ( `Cache-Control` に設定 `public` ) *は、クライアントまたは任意の中間プロキシ* が値 ( [応答キャッシュミドルウェア](xref:performance/caching/middleware)を含む) をキャッシュする可能性があることを示します。

`Location.Client` ( `Cache-Control` に設定 `private` ) は *、クライアントのみ* が値をキャッシュできることを示します。 [応答キャッシュミドルウェア](xref:performance/caching/middleware)を含め、中間キャッシュで値をキャッシュすることはできません。

キャッシュ制御ヘッダーは、応答をキャッシュするタイミングと方法について、クライアントと仲介プロキシに関するガイダンスを提供するだけです。 クライアントとプロキシが [HTTP 1.1 のキャッシュ仕様](https://tools.ietf.org/html/rfc7234)に従うという保証はありません。 [応答キャッシュミドルウェア](xref:performance/caching/middleware) は、常に仕様によって配置されたキャッシュ規則に従います。

次の例では、サンプルアプリの Cache3 ページモデルと、を設定し、 <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Duration> 既定値のままにして生成されたヘッダーを示し <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> ます。

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Pages/Cache3.cshtml.cs?name=snippet)]

このサンプルアプリでは、次のヘッダーを含む Cache3 ページが返されます。

```
Cache-Control: public,max-age=10
```

### <a name="cache-profiles"></a>キャッシュプロファイル

多くのコントローラーアクション属性に対して応答キャッシュ設定を複製するのではなく、で MVC/ページを設定するときに、キャッシュプロファイルをオプションとして構成でき Razor `Startup.ConfigureServices` ます。 参照キャッシュプロファイルで見つかった値は、によって既定値として使用され、 <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> 属性で指定されたプロパティによって上書きされます。

キャッシュプロファイルを設定します。 次の例は、サンプルアプリの30秒のキャッシュプロファイルを示してい `Startup.ConfigureServices` ます。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](response/samples/3.x/Startup.cs?name=snippet1)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Startup.cs?name=snippet1)]

::: moniker-end

サンプルアプリの Cache4 ページモデルは、キャッシュプロファイルを参照し `Default30` ます。

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Pages/Cache4.cshtml.cs?name=snippet)]

は、 <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> 次のように適用できます。

* Razor ページ: ハンドラーメソッドに属性を適用することはできません。
* MVC コントローラー。
* MVC アクションメソッド: メソッドレベルの属性は、クラスレベルの属性で指定された設定をオーバーライドします。

結果のヘッダーは、キャッシュプロファイルによって Cache4 ページの応答に適用され `Default30` ます。

```
Cache-Control: public,max-age=30
```

## <a name="additional-resources"></a>その他のリソース

* [キャッシュへの応答の格納](https://tools.ietf.org/html/rfc7234#section-3)
* [Cache-Control](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9)
* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>
