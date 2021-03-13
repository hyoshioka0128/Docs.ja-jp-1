---
title: ASP.NET Core でのクロスオリジン要求 (CORS) を有効にする
author: rick-anderson
description: ASP.NET Core アプリでのクロスオリジン要求を許可または拒否するための標準として CORS を使用する方法について説明します。
ms.author: riande
ms.custom: mvc
ms.date: 04/17/2020
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
uid: security/cors
ms.openlocfilehash: b057e5e08b8a4d0f9bcd68f92102cad309655acc
ms.sourcegitcommit: 07e7ee573fe4e12be93249a385db745d714ff6ae
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/12/2021
ms.locfileid: "103413510"
---
# <a name="enable-cross-origin-requests-cors-in-aspnet-core"></a>ASP.NET Core でのクロスオリジン要求 (CORS) を有効にする

::: moniker range=">= aspnetcore-3.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT) および [Kirk Larkin](https://twitter.com/serpent5)

この記事では、ASP.NET Core アプリで CORS を有効にする方法について説明します。

ブラウザーのセキュリティにより、Web ページを提供したドメインと異なるドメインに対して、Web ページが要求を行うことはできません。 この制限は、*同一オリジン ポリシー* と呼ばれます。 同一オリジン ポリシーにより、悪意のあるサイトが別のサイトから機密データを読み取れないようになっています。 場合によっては、他のサイトがアプリに対してクロスオリジン要求を行うことを許可する必要があります。 詳細については、「 [MOZILLA CORS](https://developer.mozilla.org/docs/Web/HTTP/CORS)」を参照してください。

[クロスオリジンリソース共有](https://www.w3.org/TR/cors/) (CORS):

* は、サーバーが同じオリジンポリシーを緩めることを可能にする W3C 標準です。
* は、CORS 緩和され security のセキュリティ機能では **ありません** 。 CORS を許可すると、API の安全性が向上しません。 詳細については、「 [CORS のしくみ](#how-cors)」を参照してください。
* サーバーが他のユーザーを拒否している間に、一部のクロスオリジン要求を明示的に許可することを許可します。
* は、 [JSONP](/dotnet/framework/wcf/samples/jsonp)など、以前の手法よりも安全で柔軟性に優れています。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/cors/3.1sample/Cors/WebAPI)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="same-origin"></a>同じオリジン

同じスキーム、ホスト、およびポート ([RFC 6454](https://tools.ietf.org/html/rfc6454)) がある場合、2つの url のオリジンは同じになります。

この2つの Url のオリジンは同じです。

* `https://example.com/foo.html`
* `https://example.com/bar.html`

これらの Url には、前の2つの Url とは異なるオリジンがあります。

* `https://example.net`: 異なるドメイン
* `https://www.example.com/foo.html`: 異なるサブドメイン
* `http://example.com/foo.html`: 異なるスキーム
* `https://example.com:9000/foo.html`: 別のポート

## <a name="enable-cors"></a>CORS を有効にする

CORS を有効にするには、次の3つの方法があります。

* [名前付きポリシー](#np)または既定の[ポリシー](#dp)を使用するミドルウェア。
* [エンドポイントルーティング](#ecors)を使用する。
* [[Enablecors]](#attr)属性を使用します。

名前付きポリシーで [[Enablecors]](#attr) 属性を使用すると、CORS をサポートするエンドポイントを制限するための最も細かい制御が可能になります。

> [!WARNING]
> <xref:Microsoft.AspNetCore.Builder.CorsMiddlewareExtensions.UseCors%2A> は正しい順序で呼び出す必要があります。 詳細については、「 [ミドルウェアの順序](xref:fundamentals/middleware/index#middleware-order)」を参照してください。 たとえば、を使用する場合は、の `UseCors` 前にを呼び出す必要があり <xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching%2A> `UseResponseCaching` ます。

各方法の詳細については、次のセクションで説明します。

<a name="np"></a>

## <a name="cors-with-named-policy-and-middleware"></a>名前付きポリシーとミドルウェアを使用した CORS

CORS ミドルウェアは、クロスオリジン要求を処理します。 次のコードは、指定されたオリジンを持つすべてのアプリのエンドポイントに CORS ポリシーを適用します。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Startup.cs?name=snippet&highlight=3,9,32)]

上記のコードでは、次のことが行われます。

* ポリシー名をに設定 `_myAllowSpecificOrigins` します。 ポリシー名は任意です。
* 拡張メソッドを呼び出し、 <xref:Microsoft.AspNetCore.Builder.CorsMiddlewareExtensions.UseCors*> CORS ポリシーを指定し  `_myAllowSpecificOrigins` ます。 `UseCors` CORS ミドルウェアを追加します。 への呼び出しは `UseCors` `UseRouting` 、の後、の前に配置する必要があり `UseAuthorization` ます。 詳細については、「 [ミドルウェアの順序](xref:fundamentals/middleware/index#middleware-order)」を参照してください。
* <xref:Microsoft.Extensions.DependencyInjection.CorsServiceCollectionExtensions.AddCors*>[ラムダ式](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions)を使用してを呼び出します。 ラムダはオブジェクトを受け取り <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder> ます。 などの[構成オプション](#cors-policy-options)に `WithOrigins` ついては、この記事の後半で説明します。
* `_myAllowSpecificOrigins`すべてのコントローラーエンドポイントに対して CORS ポリシーを有効にします。 特定のエンドポイントに CORS ポリシーを適用するには、「 [エンドポイントルーティング](#ecors) 」を参照してください。
* [応答キャッシュミドルウェア](xref:performance/caching/middleware)を使用する場合は、の <xref:Microsoft.AspNetCore.Builder.CorsMiddlewareExtensions.UseCors%2A> 前にを呼び出し <xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching%2A> ます。

エンドポイントルーティングでは、CORS ミドルウェアをとの呼び出しの間で実行するように構成する **必要があり** `UseRouting` `UseEndpoints` ます。

前のコードのようなコードをテストする方法については、「 [テスト CORS](#testc) 」を参照してください。

メソッド呼び出しによって、 <xref:Microsoft.Extensions.DependencyInjection.MvcCorsMvcCoreBuilderExtensions.AddCors*> アプリのサービスコンテナーに CORS サービスが追加されます。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Startup.cs?name=snippet2)]

詳細については、このドキュメントの「 [CORS ポリシーオプション](#cpo) 」を参照してください。

メソッドは、 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder> 次のコードに示すように、連結できます。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Startup2.cs?name=snippet)]

注: 指定された URL に末尾にスラッシュ () を含めることは **できません** `/` 。 URL がで終了した場合 `/` 、比較はを返し、 `false` ヘッダーは返されません。

<a name="dp"></a>

### <a name="cors-with-default-policy-and-middleware"></a>既定のポリシーとミドルウェアを使用した CORS

次の強調表示されたコードは、既定の CORS ポリシーを有効にします。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupDefaultPolicy.cs?name=snippet2&highlight=7,29)]

上記のコードは、すべてのコントローラーエンドポイントに既定の CORS ポリシーを適用します。

<a name="ecors"></a>

## <a name="enable-cors-with-endpoint-routing"></a>エンドポイント ルーティングで CORS を有効にする

現在、を使用して、エンドポイントごとに CORS を有効にする `RequireCors` ことは、[自動プレフライト要求](#apf)をサポートして **いません**。 詳細については、こちらの [GitHub の問題](https://github.com/dotnet/aspnetcore/issues/20709) を参照してください。また、 [エンドポイントルーティングと [HttpOptions] を使用](#tcer)して CORS をテストしてください。

エンドポイントルーティングを使用すると、一連の拡張メソッドを使用して、エンドポイントごとに CORS を有効にすることができ <xref:Microsoft.AspNetCore.Builder.CorsEndpointConventionBuilderExtensions.RequireCors*> ます。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupEndPt.cs?name=snippet2&highlight=3,7-15,32,40,43)]

上のコードでは以下の操作が行われます。

* `app.UseCors` CORS ミドルウェアを有効にします。 既定のポリシーが構成されていないため、 `app.UseCors()` 単独で CORS を有効にすることはできません。
* `/echo`およびコントローラーエンドポイントは、指定されたポリシーを使用して、クロスオリジン要求を許可します。
* `/echo2`既定のポリシーが指定されていないため、と Razor Pages エンドポイントはクロスオリジン要求を許可し **ません**。

[[Disablecors]](#dc)属性では、を使用してエンドポイントルーティングによって有効にされた CORS は無効になり **ません** `RequireCors` 。

前述のようなコードをテストする方法については、「 [エンドポイントルーティングを使用した CORS のテスト」および「[HttpOptions]](#tcer) 」を参照してください。

<a name="attr"></a>

## <a name="enable-cors-with-attributes"></a>属性を使用して CORS を有効にする

[[Enablecors]](xref:Microsoft.AspNetCore.Cors.EnableCorsAttribute)属性を使用して cors を有効にし、cors を必要とするエンドポイントのみに名前付きポリシーを適用することで、最も細かい制御が可能になります。

[[Enablecors]](xref:Microsoft.AspNetCore.Cors.EnableCorsAttribute)属性は、CORS をグローバルに適用するための代替手段を提供します。 属性は、 `[EnableCors]` すべてのエンドポイントではなく、選択したエンドポイントに対して CORS を有効にします。

* `[EnableCors]` 既定のポリシーを指定します。
* `[EnableCors("{Policy String}")]` 名前付きポリシーを指定します。

`[EnableCors]`属性は次のものに適用できます。

* Razor 改 `PageModel`
* コントローラー
* コントローラーアクションメソッド

属性を使用して、さまざまなポリシーをコントローラー、ページモデル、またはアクションメソッドに適用でき `[EnableCors]` ます。 `[EnableCors]`属性がコントローラー、ページモデル、またはアクションメソッドに適用され、CORS がミドルウェアで有効になっている場合、**両方** のポリシーが適用されます。 **ポリシーを組み合わせることはお勧めしません。** `[EnableCors]` **同じアプリ内ではなく、属性またはミドルウェアを使用します。**

次のコードは、各メソッドに異なるポリシーを適用します。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/WidgetController.cs?name=snippet&highlight=6,14)]

次のコードでは、2つの CORS ポリシーを作成します。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Startup3.cs?name=snippet&highlight=12-28,44)]

CORS 要求を制限する最も細かい制御:

* `[EnableCors("MyPolicy")]`名前付きポリシーでを使用します。
* 既定のポリシーは定義しないでください。
* [エンドポイントルーティング](#ecors)を使用しないでください。

次のセクションのコードは、上記のリストを満たしています。

前のコードのようなコードをテストする方法については、「 [テスト CORS](#testc) 」を参照してください。

<a name="dc"></a>

### <a name="disable-cors"></a>CORS を無効にする

[[Disablecors]](xref:Microsoft.AspNetCore.Cors.DisableCorsAttribute)属性は、[エンドポイントルーティング](#ecors)によって有効にされた CORS を無効にし **ません**。

次のコードでは、CORS ポリシーを定義してい `"MyPolicy"` ます。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupTestMyPolicy.cs?name=snippet)]

次のコードは、アクションの CORS を無効にし `GetValues2` ます。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/ValuesController.cs?name=snippet&highlight=1,23)]

上記のコードでは、次のことが行われます。

* [エンドポイントルーティング](#ecors)で CORS を有効にしません。
* [既定の CORS ポリシー](#dp)は定義されていません。
* では、 [[Enablecors ("MyPolicy")]](#attr) を使用して `"MyPolicy"` 、コントローラーの CORS ポリシーを有効にします。
* メソッドの CORS を無効に `GetValues2` します。

前のコードをテストする手順については、「 [テスト CORS](#testc) 」を参照してください。

<a name="cpo"></a>

## <a name="cors-policy-options"></a>CORS ポリシーのオプション

ここでは、CORS ポリシーで設定できるさまざまなオプションについて説明します。

* [許可されるオリジンの設定](#set-the-allowed-origins)
* [許可される HTTP メソッドを設定する](#set-the-allowed-http-methods)
* [許可された要求ヘッダーを設定する](#set-the-allowed-request-headers)
* [公開された応答ヘッダーを設定する](#set-the-exposed-response-headers)
* [クロスオリジン要求の資格情報](#credentials-in-cross-origin-requests)
* [プレフライトの有効期限を設定する](#set-the-preflight-expiration-time)

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsOptions.AddPolicy*> はで呼び出され `Startup.ConfigureServices` ます。 いくつかのオプションについては、まず「 [CORS のしくみ](#how-cors) 」セクションを参照してください。

## <a name="set-the-allowed-origins"></a>許可されるオリジンの設定

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyOrigin*>: 任意のスキーム (または) を持つすべてのオリジンからの CORS 要求を許可 `http` `https` します。 `AllowAnyOrigin`*web サイト* はアプリに対してクロスオリジン要求を行うことができるため、安全ではありません。

> [!NOTE]
> とを指定すると、 `AllowAnyOrigin` `AllowCredentials` 安全ではない構成で、クロスサイト要求の偽造が発生する可能性があります。 アプリが両方の方法で構成されている場合、CORS サービスは無効な CORS 応答を返します。

`AllowAnyOrigin` プレフライト要求とヘッダーに影響し `Access-Control-Allow-Origin` ます。 詳細については、「 [プレフライト要求](#preflight-requests) 」セクションを参照してください。

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.SetIsOriginAllowedToAllowWildcardSubdomains*>: <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicy.IsOriginAllowed*> オリジンが許可されているかどうかを評価するときに、構成されたワイルドカードドメインとオリジンが一致するようにするための関数として、ポリシーのプロパティを設定します。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet)]

### <a name="set-the-allowed-http-methods"></a>許可される HTTP メソッドを設定する

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyMethod*>:

* すべての HTTP メソッドを許可します。
* プレフライト要求とヘッダーに影響し `Access-Control-Allow-Methods` ます。 詳細については、「 [プレフライト要求](#preflight-requests) 」セクションを参照してください。

### <a name="set-the-allowed-request-headers"></a>許可された要求ヘッダーを設定する

CORS 要求で特定のヘッダーを送信できるようにするには、 [author 要求ヘッダー](https://xhr.spec.whatwg.org/#request)と呼ばれるを呼び出し、 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithHeaders*> 許可されているヘッダーを指定します。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet2)]

すべての [作成者要求ヘッダー](https://www.w3.org/TR/cors/#author-request-headers)を許可するには、次のように呼び出し <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyHeader*> ます。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet3)]

`AllowAnyHeader` プレフライト要求および [アクセス制御-要求](https://developer.mozilla.org/docs/Web/HTTP/Headers/Access-Control-Request-Method) ヘッダーヘッダーに影響します。 詳細については、「 [プレフライト要求](#preflight-requests) 」セクションを参照してください。

によって指定された特定のヘッダーに対して CORS ミドルウェアポリシーが一致するのは、ヘッダーがで記述されているヘッダーと `WithHeaders` 完全に一致する場合のみです `Access-Control-Request-Headers` `WithHeaders` 。

たとえば、次のように構成されたアプリを考えてみます。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet4)]

CORS ミドルウェアは、次の要求ヘッダーを使用してプレフライト要求を拒否し `Content-Language` ます。 ([headernames](xref:Microsoft.Net.Http.Headers.HeaderNames.ContentLanguage)) は、には記載されていないため `WithHeaders` です。

```
Access-Control-Request-Headers: Cache-Control, Content-Language
```

アプリは *200 OK* 応答を返しますが、CORS ヘッダーを返信しません。 そのため、ブラウザーはクロスオリジン要求を試行しません。

### <a name="set-the-exposed-response-headers"></a>公開された応答ヘッダーを設定する

既定では、ブラウザーはすべての応答ヘッダーをアプリに公開しません。 詳細については、「 [W3C クロスオリジンリソース共有 (用語): 単純な応答ヘッダー](https://www.w3.org/TR/cors/#simple-response-header)」を参照してください。

既定で使用できる応答ヘッダーは次のとおりです。

* `Cache-Control`
* `Content-Language`
* `Content-Type`
* `Expires`
* `Last-Modified`
* `Pragma`

CORS 仕様は、これらのヘッダーの *単純な応答ヘッダー* を呼び出します。 アプリで他のヘッダーを使用できるようにするには、次のように呼び出し <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithExposedHeaders*> ます。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet5)]
### <a name="credentials-in-cross-origin-requests"></a>クロスオリジン要求の資格情報

資格情報では、CORS 要求で特別な処理を行う必要があります。 既定では、ブラウザーはクロスオリジン要求で資格情報を送信しません。 資格情報には cookie 、および HTTP 認証スキームが含まれます。 クロスオリジン要求で資格情報を送信するには、クライアントがに設定されている必要があり `XMLHttpRequest.withCredentials` `true` ます。

直接の使用 `XMLHttpRequest` :

```javascript
var xhr = new XMLHttpRequest();
xhr.open('get', 'https://www.example.com/api/test');
xhr.withCredentials = true;
```

JQuery の使用:

```javascript
$.ajax({
  type: 'get',
  url: 'https://www.example.com/api/test',
  xhrFields: {
    withCredentials: true
  }
});
```

[FETCH API](https://developer.mozilla.org/docs/Web/API/Fetch_API)の使用:

```javascript
fetch('https://www.example.com/api/test', {
    credentials: 'include'
});
```

サーバーは資格情報を許可する必要があります。 クロスオリジンの資格情報を許可するには、次のように呼び出し <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowCredentials*> ます。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet6)]

HTTP 応答には、 `Access-Control-Allow-Credentials` サーバーがクロスオリジン要求に対して資格情報を許可することをブラウザーに示すヘッダーが含まれています。

ブラウザーが資格情報を送信しても、応答に有効なヘッダーが含まれていない場合 `Access-Control-Allow-Credentials` 、ブラウザーはアプリへの応答を公開せず、クロスオリジン要求は失敗します。

クロスオリジンの資格情報を許可することは、セキュリティ上のリスクです。 別のドメインの web サイトは、ユーザーの知らないうちに、サインインしているユーザーの資格情報をユーザーの代わりにアプリに送信できます。 <!-- TODO Review: When using `AllowCredentials`, all CORS enabled domains must be trusted.
I don't like "all CORS enabled domains must be trusted", because it implies that if you're not using  `AllowCredentials`, domains don't need to be trusted. -->

また、CORS 仕様では、ヘッダーが存在する場合、[オリジン] を [(すべてのオリジン)] に設定することもでき `"*"` `Access-Control-Allow-Credentials` ます。

<a name="pref"></a>

## <a name="preflight-requests"></a>プレフライト要求

一部の CORS 要求については、ブラウザーは、実際の要求を行う前に、追加の [オプション](https://developer.mozilla.org/docs/Web/HTTP/Methods/OPTIONS) 要求を送信します。 この要求は [プレフライト要求](https://developer.mozilla.org/docs/Glossary/Preflight_request)と呼ばれます。 次の **すべて** の条件に該当する場合、ブラウザーはプレフライト要求をスキップできます。

* 要求メソッドは GET、HEAD、または POST です。
* アプリで `Accept` は、、、 `Accept-Language` `Content-Language` 、 `Content-Type` 、または `Last-Event-ID` 以外の要求ヘッダーは設定されません。
* ヘッダーには、設定されて `Content-Type` いる場合、次のいずれかの値が含まれます。
  * `application/x-www-form-urlencoded`
  * `multipart/form-data`
  * `text/plain`

クライアント要求に対して設定された要求ヘッダーに適用される規則は、オブジェクトに対してを呼び出すことによって、アプリが設定するヘッダーに適用され `setRequestHeader` `XMLHttpRequest` ます。 CORS 仕様は、これらのヘッダー [作成者要求ヘッダー](https://www.w3.org/TR/cors/#author-request-headers)を呼び出します。 このルールは、ブラウザーが設定できるヘッダー (、、など) には適用されません `User-Agent` `Host` `Content-Length` 。

次に示すのは、このドキュメントの「[テスト CORS](#testc) 」セクションの **[Put test]** ボタンから行ったプレフライト要求に似た応答の例です。

```
General:
Request URL: https://cors3.azurewebsites.net/api/values/5
Request Method: OPTIONS
Status Code: 204 No Content

Response Headers:
Access-Control-Allow-Methods: PUT,DELETE,GET
Access-Control-Allow-Origin: https://cors1.azurewebsites.net
Server: Microsoft-IIS/10.0
Set-Cookie: ARRAffinity=8f8...8;Path=/;HttpOnly;Domain=cors1.azurewebsites.net
Vary: Origin

Request Headers:
Accept: */*
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Access-Control-Request-Method: PUT
Connection: keep-alive
Host: cors3.azurewebsites.net
Origin: https://cors1.azurewebsites.net
Referer: https://cors1.azurewebsites.net/
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: cross-site
User-Agent: Mozilla/5.0
```

プレフライト要求では、 [HTTP OPTIONS](https://developer.mozilla.org/docs/Web/HTTP/Methods/OPTIONS) メソッドを使用します。 次のヘッダーが含まれている場合があります。

* [アクセス制御要求メソッド](https://developer.mozilla.org/docs/Web/HTTP/Headers/Access-Control-Request-Method): 実際の要求に使用される HTTP メソッド。
* [アクセス制御要求ヘッダー](https://developer.mozilla.org/docs/Web/HTTP/Headers/Access-Control-Allow-Headers): アプリが実際の要求に対して設定する要求ヘッダーのリスト。 前述のように、これにはブラウザーが設定するヘッダー (など) は含まれません `User-Agent` 。
* [アクセス制御-許可-メソッド](https://developer.mozilla.org/docs/Web/HTTP/Headers/Access-Control-Allow-Methods)

プレフライト要求が拒否された場合、アプリは応答を返し `200 OK` ますが、CORS ヘッダーは設定しません。 そのため、ブラウザーはクロスオリジン要求を試行しません。 拒否されたプレフライト要求の例については、このドキュメントの「 [テスト CORS](#testc) 」セクションを参照してください。

F12 ツールを使用すると、コンソールアプリには、ブラウザーに応じて次のいずれかのようなエラーが表示されます。

* Firefox: クロスオリジン要求がブロックされています: 同じオリジンポリシーで、でのリモートリソースの読み取りが許可されて `https://cors1.azurewebsites.net/api/TodoItems1/MyDelete2/5` いません。 (理由: CORS 要求が失敗しました)。 [詳細情報](https://developer.mozilla.org/docs/Web/HTTP/CORS/Errors/CORSDidNotSucceed)
* Chromium ベース: オリジン ' ' からの ' ' でのフェッチへのアクセス https://cors1.azurewebsites.net/api/TodoItems1/MyDelete2/5 https://cors3.azurewebsites.net が CORS ポリシーによってブロックされました。プレフライト要求への応答がアクセス制御チェックに合格しません: 要求されたリソースに ' アクセス制御-許可-オリジン ' ヘッダーが存在しません。 非透過の応答が要求に対応している場合は、要求のモードを 'no-cors' に設定し、CORS を無効にしてリソースをフェッチしてください。

特定のヘッダーを許可するには、次のように呼び出し <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithHeaders*> ます。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet2)]

すべての [作成者要求ヘッダー](https://www.w3.org/TR/cors/#author-request-headers)を許可するには、次のように呼び出し <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyHeader*> ます。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet3)]

ブラウザーの設定方法が一貫していません `Access-Control-Request-Headers` 。 次のいずれかの場合:

* ヘッダーは、 `"*"`
* <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicy.AllowAnyHeader*> が呼び出されます: 少なくとも、、、 `Accept` `Content-Type` `Origin` 、およびサポートするカスタムヘッダーを含めます。

<a name="apf"></a>

### <a name="automatic-preflight-request-code"></a>自動プレフライト要求コード

CORS ポリシーを適用する場合は、次のいずれかの方法を実行します。

* でを呼び出すことにより、グローバルに `app.UseCors` `Startup.Configure` 。
* 属性を使用し `[EnableCors]` ます。

ASP.NET Core は、プレフライトオプション要求に応答します。

現在、を使用して、エンドポイントごとに CORS を有効にする `RequireCors` ことは、自動プレフライト要求をサポートして **いません** 。

このドキュメントの「 [テスト CORS](#testc) 」セクションでは、この動作について説明します。

<a name="pro"></a>

### <a name="httpoptions-attribute-for-preflight-requests"></a>プレフライト要求の [HttpOptions] 属性

CORS が適切なポリシーで有効になっている場合、ASP.NET Core は通常、CORS プレフライト要求に自動的に応答します。 シナリオによっては、これが当てはまりません。 たとえば、CORS を [エンドポイントルーティングと共](#ecors)に使用します。

次のコードでは、 [[HttpOptions]](xref:Microsoft.AspNetCore.Mvc.HttpOptionsAttribute) 属性を使用して、OPTIONS 要求のエンドポイントを作成します。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/TodoItems2Controller.cs?name=snippet&highlight=5-17)]

前のコードをテストする手順については、「 [エンドポイントルーティングを使用した CORS のテスト」および「[HttpOptions]](#tcer) 」を参照してください。

### <a name="set-the-preflight-expiration-time"></a>プレフライトの有効期限を設定する

ヘッダーは、 `Access-Control-Max-Age` プレフライト要求への応答をキャッシュできる期間を指定します。 このヘッダーを設定するには、次のように呼び出し <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.SetPreflightMaxAge*> ます。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet7)]
<a name="how-cors"></a>

## <a name="how-cors-works"></a>CORS のしくみ

このセクションでは、HTTP メッセージレベルでの [CORS](https://developer.mozilla.org/docs/Web/HTTP/CORS) 要求の動作について説明します。

* CORS はセキュリティ機能では **ありません** 。 CORS は、サーバーが同じオリジンポリシーを緩めることを可能にする W3C 標準です。
  * たとえば、悪意のあるアクターがサイトに対して [クロスサイトスクリプティング (XSS)](xref:security/cross-site-scripting) を使用して、CORS が有効になっているサイトに対してクロスサイト要求を実行し、情報を盗むことができます。
* CORS を許可することで、API の安全性が向上します。
  * CORS を適用するには、クライアント (ブラウザー) が必要です。 サーバーは要求を実行し、応答を返します。これは、エラーを返し、応答をブロックするクライアントです。 たとえば、次のツールを使用すると、サーバーの応答が表示されます。
    * [Fiddler](https://www.telerik.com/fiddler)
    * [Postman](https://www.getpostman.com/)
    * [.NET HttpClient](/dotnet/csharp/tutorials/console-webapiclient)
    * アドレスバーに URL を入力して、web ブラウザーを表示します。
* これは、ブラウザーがクロスオリジン [Xhr](https://developer.mozilla.org/docs/Web/API/XMLHttpRequest) または [Fetch API](https://developer.mozilla.org/docs/Web/API/Fetch_API) 要求を実行して、それ以外は禁止されるようにする方法です。
  * CORS のないブラウザーは、クロスオリジン要求を行うことができません。 CORS の前に、この制限を回避するために [JSONP](https://www.w3schools.com/js/js_json_jsonp.asp) が使用されていました。 JSONP は XHR を使用しないので、タグを使用して `<script>` 応答を受信します。 スクリプトをクロスオリジンで読み込むことができます。

[CORS 仕様](https://www.w3.org/TR/cors/)では、クロスオリジン要求を有効にする新しい HTTP ヘッダーがいくつか導入されました。 ブラウザーが CORS をサポートしている場合、クロスオリジン要求に対してこれらのヘッダーが自動的に設定されます。 CORS を有効にするためにカスタム JavaScript コードは必要ありません。

配置された[サンプル](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/cors/3.1sample/Cors/WebAPI)の [[テストの配置] ボタン](https://cors3.azurewebsites.net/test)

次に、[ [値](https://cors3.azurewebsites.net/) のテスト] ボタンからへのクロスオリジン要求の例を示し `https://cors1.azurewebsites.net/api/values` ます。 `Origin`ヘッダー:

* 要求を行っているサイトのドメインを提供します。
* は必須であり、ホストと異なる必要があります。

**一般的なヘッダー**

```
Request URL: https://cors1.azurewebsites.net/api/values
Request Method: GET
Status Code: 200 OK
```

**応答ヘッダー**

```
Content-Encoding: gzip
Content-Type: text/plain; charset=utf-8
Server: Microsoft-IIS/10.0
Set-Cookie: ARRAffinity=8f...;Path=/;HttpOnly;Domain=cors1.azurewebsites.net
Transfer-Encoding: chunked
Vary: Accept-Encoding
X-Powered-By: ASP.NET
```

**要求ヘッダー**

```
Accept: */*
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Connection: keep-alive
Host: cors1.azurewebsites.net
Origin: https://cors3.azurewebsites.net
Referer: https://cors3.azurewebsites.net/
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: cross-site
User-Agent: Mozilla/5.0 ...
```

要求では、 `OPTIONS` サーバーは応答のヘッダーヘッダーを設定し `Access-Control-Allow-Origin: {allowed origin}` ます。 たとえば、デプロイされた [サンプル](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/cors/3.1sample/Cors/WebAPI)の [Delete [enablecors]](https://cors1.azurewebsites.net/test?number=2) button 要求には、 `OPTIONS` 次のヘッダーが含まれています。

**一般的なヘッダー**

```
Request URL: https://cors3.azurewebsites.net/api/TodoItems2/MyDelete2/5
Request Method: OPTIONS
Status Code: 204 No Content
```

**応答ヘッダー**

```
Access-Control-Allow-Headers: Content-Type,x-custom-header
Access-Control-Allow-Methods: PUT,DELETE,GET,OPTIONS
Access-Control-Allow-Origin: https://cors1.azurewebsites.net
Server: Microsoft-IIS/10.0
Set-Cookie: ARRAffinity=8f...;Path=/;HttpOnly;Domain=cors3.azurewebsites.net
Vary: Origin
X-Powered-By: ASP.NET
```

**要求ヘッダー**

```
Accept: */*
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Access-Control-Request-Headers: content-type
Access-Control-Request-Method: DELETE
Connection: keep-alive
Host: cors3.azurewebsites.net
Origin: https://cors1.azurewebsites.net
Referer: https://cors1.azurewebsites.net/test?number=2
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: cross-site
User-Agent: Mozilla/5.0
```

上記の **応答ヘッダー** では、サーバーは応答で [アクセス制御-許可](https://developer.mozilla.org/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) ヘッダーを設定します。 `https://cors1.azurewebsites.net`このヘッダーの値は、要求のヘッダーと一致し `Origin` ます。

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyOrigin*>が呼び出されると、 `Access-Control-Allow-Origin: *` ワイルドカード値が返されます。 `AllowAnyOrigin` 任意の発信元を許可します。

応答にヘッダーが含まれていない場合 `Access-Control-Allow-Origin` 、クロスオリジン要求は失敗します。 具体的には、ブラウザーは要求を許可しません。 サーバーが応答を正常に返す場合でも、ブラウザーはクライアントアプリで応答を使用できません。

<a name="options"></a>

### <a name="display-options-requests"></a>表示オプションの要求

既定では、Chrome および Edge ブラウザーでは、F12 ツールの [ネットワーク] タブの [要求] オプションが表示されません。 これらのブラウザーでオプションの要求を表示するには:

* `chrome://flags/#out-of-blink-cors` または `edge://flags/#out-of-blink-cors`
* フラグを無効にします。
* [再起動] をタップします。

既定では、Firefox によってオプションの要求が表示されます。

## <a name="cors-in-iis"></a>IIS での CORS

IIS に展開する場合、サーバーが匿名アクセスを許可するように構成されていない場合、CORS は Windows 認証の前に実行する必要があります。 このシナリオをサポートするには、アプリ用に [IIS CORS モジュール](https://www.iis.net/downloads/microsoft/iis-cors-module) をインストールして構成する必要があります。

<a name="testc"></a>

## <a name="test-cors"></a>CORS のテスト

[サンプルダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/cors/3.1sample/Cors/WebAPI)には、CORS をテストするコードが含まれています。 [ダウンロード方法](xref:index#how-to-download-a-sample)に関するページを参照してください。 このサンプルは、ページが追加された API プロジェクトです Razor 。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupTest2.cs?name=snippet2)]

  > [!WARNING]
  > `WithOrigins("https://localhost:<port>");` サンプルアプリのテストにのみ使用してください [サンプルコードをダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/cors/3.1sample/Cors)します。

`ValuesController`テスト用のエンドポイントは次のとおりです。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/ValuesController.cs?name=snippet)]

[Mydisplayrouteinfo](https://github.com/Rick-Anderson/RouteInfo/blob/master/Microsoft.Docs.Samples.RouteInfo/ControllerContextExtensions.cs) は、Rick.Doc. [ routeinfo](https://www.nuget.org/packages/Rick.Docs.Samples.RouteInfo) NuGet パッケージによって提供され、ルート情報を表示します。

次のいずれかの方法を使用して、上記のサンプルコードをテストします。

* でデプロイされたサンプルアプリを使用し [https://cors3.azurewebsites.net/](https://cors3.azurewebsites.net/) ます。 サンプルをダウンロードする必要はありません。
* `dotnet run`の既定の URL を使用して、サンプルを実行し `https://localhost:5001` ます。
* URL に対してポートを44398に設定して、Visual Studio からサンプルを実行し `https://localhost:44398` ます。

F12 ツールでブラウザーを使用すると、次のようになります。

* [ **値** ] をクリックし、[ **ネットワーク** ] タブでヘッダーを確認します。
* [ **テストの配置** ] ボタンを選択します。 OPTIONS 要求を表示する方法については、「 [表示オプションの要求](#options) 」を参照してください。 **Put テスト** では、2つの要求、オプションのプレフライト要求、および put 要求が作成されます。
* 失敗し **`GetValues2 [DisableCors]`** た CORS 要求をトリガーするには、このボタンを選択します。 ドキュメントに記載されているように、応答は200成功を返しますが、CORS 要求は行われません。 [ **コンソール** ] タブを選択して、CORS エラーを確認します。 ブラウザーによっては、次のようなエラーが表示されます。

     `'https://cors1.azurewebsites.net/api/values/GetValues2'`送信元からのフェッチへのアクセス `'https://cors3.azurewebsites.net'` が CORS ポリシーによってブロックされました。要求されたリソースに ' アクセス制御許可-発信元 ' ヘッダーが存在しません。 非透過の応答が要求に対応している場合は、要求のモードを 'no-cors' に設定し、CORS を無効にしてリソースをフェッチしてください。
     
CORS が有効なエンドポイントは、 [curl](https://curl.haxx.se/)、 [Fiddler](https://www.telerik.com/fiddler)、 [Postman](https://www.getpostman.com/)などのツールを使用してテストできます。 ツールを使用する場合、ヘッダーによって指定された要求の配信元は、要求を受信している `Origin` ホストと異なる必要があります。 ヘッダーの値に基づいて要求が *クロスオリジン* でない場合は、 `Origin` 次のようになります。

* CORS ミドルウェアが要求を処理する必要はありません。
* CORS ヘッダーが応答で返されません。

次のコマンドは、を使用し `curl` て、情報を含む OPTIONS 要求を発行します。

```bash
curl -X OPTIONS https://cors3.azurewebsites.net/api/TodoItems2/5 -i
```

<!--
curl come with Git. Add to path variable
C:\Program Files\Git\mingw64\bin\
-->

<a name="tcer"></a>

### <a name="test-cors-with-endpoint-routing-and-httpoptions"></a>エンドポイントルーティングを使用した CORS のテスト [HttpOptions]

現在、を使用して、エンドポイントごとに CORS を有効にする `RequireCors` ことは、[自動プレフライト要求](#apf)をサポートして **いません**。 [エンドポイントルーティングを使用して CORS を有効に](#ecors)する次のコードについて考えてみます。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupEndPointBugTest.cs?name=snippet2)]

`TodoItems1Controller`テスト用のエンドポイントは次のとおりです。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/TodoItems1Controller.cs?name=snippet2)]

配置された[サンプル](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/cors/3.1sample/Cors/WebAPI)の[テストページ](https://cors1.azurewebsites.net/test?number=1)から、前のコードをテストします。

エンドポイントがプレフライト要求に応答して応答するため、 **Delete [EnableCors** **] ボタンは** 正常に終了し `[EnableCors]` ます。 他のエンドポイントは失敗します。 [JavaScript](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/cors/3.1sample/Cors/WebAPI/wwwroot/js/MyJS.js)によってが送信されるため、 **GET** ボタンは失敗します。

```javascript
 headers: {
      "Content-Type": "x-custom-header"
 },
```

次の例は `TodoItems2Controller` 同様のエンドポイントを提供しますが、オプションの要求に応答するための明示的なコードが含まれています。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/TodoItems2Controller.cs?name=snippet2)]

配置されたサンプルの [テストページ](https://cors1.azurewebsites.net/test?number=2) から、前のコードをテストします。 [ **コントローラー** ] ドロップダウンリストで、[ **プレフライト** ] を選択し、[ **コントローラーの設定**] をクリックします。 エンドポイントに対するすべての CORS 呼び出しが `TodoItems2Controller` 成功します。

## <a name="additional-resources"></a>その他のリソース

* [クロスオリジンリソース共有 (CORS)](https://developer.mozilla.org/docs/Web/HTTP/CORS)
* [IIS CORS モジュールの概要](https://blogs.iis.net/iisteam/getting-started-with-the-iis-cors-module)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

この記事では、ASP.NET Core アプリで CORS を有効にする方法について説明します。

ブラウザーのセキュリティにより、Web ページを提供したドメインと異なるドメインに対して、Web ページが要求を行うことはできません。 この制限は、*同一オリジン ポリシー* と呼ばれます。 同一オリジン ポリシーにより、悪意のあるサイトが別のサイトから機密データを読み取れないようになっています。 場合によっては、他のサイトがアプリに対してクロスオリジン要求を行うことを許可する必要があります。 詳細については、「 [MOZILLA CORS](https://developer.mozilla.org/docs/Web/HTTP/CORS)」を参照してください。

[クロスオリジンリソース共有](https://www.w3.org/TR/cors/) (CORS):

* は、サーバーが同じオリジンポリシーを緩めることを可能にする W3C 標準です。
* は、CORS 緩和され security のセキュリティ機能では **ありません** 。 CORS を許可すると、API の安全性が向上しません。 詳細については、「 [CORS のしくみ](#how-cors)」を参照してください。
* サーバーが他のユーザーを拒否している間に、一部のクロスオリジン要求を明示的に許可することを許可します。
* は、 [JSONP](/dotnet/framework/wcf/samples/jsonp)など、以前の手法よりも安全で柔軟性に優れています。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/cors/sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="same-origin"></a>同じオリジン

同じスキーム、ホスト、およびポート ([RFC 6454](https://tools.ietf.org/html/rfc6454)) がある場合、2つの url のオリジンは同じになります。

この2つの Url のオリジンは同じです。

* `https://example.com/foo.html`
* `https://example.com/bar.html`

これらの Url には、前の2つの Url とは異なるオリジンがあります。

* `https://example.net`: 異なるドメイン
* `https://www.example.com/foo.html`: 異なるサブドメイン
* `http://example.com/foo.html`: 異なるスキーム
* `https://example.com:9000/foo.html`: 別のポート

Internet Explorer は、オリジンを比較するときにポートを考慮しません。

## <a name="cors-with-named-policy-and-middleware"></a>名前付きポリシーとミドルウェアを使用した CORS

CORS ミドルウェアは、クロスオリジン要求を処理します。 次のコードでは、指定したオリジンのアプリ全体に対して CORS を有効にします。

[!code-csharp[](cors/sample/Cors/WebAPI/Startup.cs?name=snippet&highlight=8,14-23,38)]

上記のコードでは、次のことが行われます。

* ポリシー名を " \_ myallow固有のオリジン" に設定します。 ポリシー名は任意です。
* <xref:Microsoft.AspNetCore.Builder.CorsMiddlewareExtensions.UseCors*>CORS を有効にする拡張メソッドを呼び出します。
* <xref:Microsoft.Extensions.DependencyInjection.CorsServiceCollectionExtensions.AddCors*>[ラムダ式](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions)を使用してを呼び出します。 ラムダはオブジェクトを受け取り <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder> ます。 などの[構成オプション](#cors-policy-options)に `WithOrigins` ついては、この記事の後半で説明します。

メソッド呼び出しによって、 <xref:Microsoft.Extensions.DependencyInjection.MvcCorsMvcCoreBuilderExtensions.AddCors*> アプリのサービスコンテナーに CORS サービスが追加されます。

[!code-csharp[](cors/sample/Cors/WebAPI/Startup.cs?name=snippet2)]

詳細については、このドキュメントの「 [CORS ポリシーオプション](#cpo) 」を参照してください。

メソッドは、 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder> 次のコードに示すようにメソッドを連結できます。

[!code-csharp[](cors/sample/Cors/WebAPI/Startup2.cs?name=snippet2)]

注: URL の末尾にスラッシュ () を含めることは **できません** `/` 。 URL がで終了した場合 `/` 、比較はを返し、 `false` ヘッダーは返されません。

次のコードは、CORS ミドルウェアを使用してすべてのアプリのエンドポイントに CORS ポリシーを適用します。
```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseHsts();
    }

    app.UseCors();

    app.UseHttpsRedirection();
    app.UseMvc();
}
```
メモ: `UseCors` の前にを呼び出す必要があり `UseMvc` ます。

ページ/コントローラー/アクションレベルで CORS ポリシーを適用するには [ Razor 、「ページ、コントローラー、およびアクションメソッドで Cors を有効](#ecors) にする」を参照してください。

前のコードのようなコードをテストする方法については、「 [テスト CORS](#test) 」を参照してください。

## <a name="enable-cors-with-attributes"></a>属性を使用して CORS を有効にする

[ &lbrack; Enablecors &rbrack; ](xref:Microsoft.AspNetCore.Cors.EnableCorsAttribute)属性は、CORS をグローバルに適用するための代替手段を提供します。 属性を指定すると `[EnableCors]` 、すべてのエンドポイントではなく、選択したエンドポイントに対して CORS が有効になります。

を使用し `[EnableCors]` て、既定のポリシーを指定し、 `[EnableCors("{Policy String}")]` ポリシーを指定します。

`[EnableCors]`属性は次のものに適用できます。

* Razor 改 `PageModel`
* コントローラー
* コントローラーアクションメソッド

属性を使用して、コントローラー/ページモデル/アクションに異なるポリシーを適用でき  `[EnableCors]` ます。 `[EnableCors]`属性がコントローラー/ページモデル/アクションメソッドに適用され、CORS がミドルウェアで有効になっている場合、**両方** のポリシーが適用されます。 ポリシーを組み合わせることはお勧めし **ません** 。 `[EnableCors]`両方では **なく**、属性またはミドルウェアを使用します。 を使用する場合は `[EnableCors]` 、既定のポリシーを定義 **しないで** ください。

次のコードは、各メソッドに異なるポリシーを適用します。

[!code-csharp[](cors/sample/Cors/WebAPI/Controllers/WidgetController.cs?name=snippet&highlight=6,14)]

次のコードでは、CORS の既定のポリシーとという名前のポリシーを作成し `"AnotherPolicy"` ます。

[!code-csharp[](cors/sample/Cors/WebAPI/StartupMultiPolicy.cs?name=snippet&highlight=12-28)]

### <a name="disable-cors"></a>CORS を無効にする

[ &lbrack; Disablecors &rbrack; ](xref:Microsoft.AspNetCore.Cors.DisableCorsAttribute)属性は、コントローラー/ページモデル/アクションの CORS を無効にします。

<a name="cpo"></a>

## <a name="cors-policy-options"></a>CORS ポリシーのオプション

ここでは、CORS ポリシーで設定できるさまざまなオプションについて説明します。

* [許可されるオリジンの設定](#set-the-allowed-origins)
* [許可される HTTP メソッドを設定する](#set-the-allowed-http-methods)
* [許可された要求ヘッダーを設定する](#set-the-allowed-request-headers)
* [公開された応答ヘッダーを設定する](#set-the-exposed-response-headers)
* [クロスオリジン要求の資格情報](#credentials-in-cross-origin-requests)
* [プレフライトの有効期限を設定する](#set-the-preflight-expiration-time)

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsOptions.AddPolicy*> はで呼び出され `Startup.ConfigureServices` ます。 いくつかのオプションについては、まず「 [CORS のしくみ](#how-cors) 」セクションを参照してください。

## <a name="set-the-allowed-origins"></a>許可されるオリジンの設定

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyOrigin*>: 任意のスキーム (または) を持つすべてのオリジンからの CORS 要求を許可 `http` `https` します。 `AllowAnyOrigin`*web サイト* はアプリに対してクロスオリジン要求を行うことができるため、安全ではありません。

> [!NOTE]
> とを指定すると、 `AllowAnyOrigin` `AllowCredentials` 安全ではない構成で、クロスサイト要求の偽造が発生する可能性があります。 セキュリティで保護されたアプリの場合は、クライアントがサーバーリソースへのアクセスを承認する必要がある場合は、オリジンの正確な一覧を指定します。

`AllowAnyOrigin` プレフライト要求とヘッダーに影響し `Access-Control-Allow-Origin` ます。 詳細については、「 [プレフライト要求](#preflight-requests) 」セクションを参照してください。

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.SetIsOriginAllowedToAllowWildcardSubdomains*>: <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicy.IsOriginAllowed*> オリジンが許可されているかどうかを評価するときに、構成されたワイルドカードドメインとオリジンが一致するようにするための関数として、ポリシーのプロパティを設定します。

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=100-105&highlight=4-5)]

### <a name="set-the-allowed-http-methods"></a>許可される HTTP メソッドを設定する

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyMethod*>:

* すべての HTTP メソッドを許可します。
* プレフライト要求とヘッダーに影響し `Access-Control-Allow-Methods` ます。 詳細については、「 [プレフライト要求](#preflight-requests) 」セクションを参照してください。

### <a name="set-the-allowed-request-headers"></a>許可された要求ヘッダーを設定する

CORS 要求で特定のヘッダーを送信できるようにするには、 *author 要求ヘッダー* と呼ばれるを呼び出し、 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithHeaders*> 許可されているヘッダーを指定します。

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=55-60&highlight=5)]

すべての作成者要求ヘッダーを許可するには、次のように呼び出し <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyHeader*> ます。

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=64-69&highlight=5)]

この設定は、プレフライト要求とヘッダーに影響し `Access-Control-Request-Headers` ます。 詳細については、「 [プレフライト要求](#preflight-requests) 」セクションを参照してください。

CORS ミドルウェアでは、 `Access-Control-Request-Headers` CorsPolicy で構成されている値に関係なく、常にの4つのヘッダーを送信できます。 このヘッダーの一覧には次のものが含まれます。

* `Accept`
* `Accept-Language`
* `Content-Language`
* `Origin`

たとえば、次のように構成されたアプリを考えてみます。

```csharp
app.UseCors(policy => policy.WithHeaders(HeaderNames.CacheControl));
```

は常に許可されているため、CORS ミドルウェアは次の要求ヘッダーを使用してプレフライト要求に正常に応答し `Content-Language` ます。

```
Access-Control-Request-Headers: Cache-Control, Content-Language
```

### <a name="set-the-exposed-response-headers"></a>公開された応答ヘッダーを設定する

既定では、ブラウザーはすべての応答ヘッダーをアプリに公開しません。 詳細については、「 [W3C クロスオリジンリソース共有 (用語): 単純な応答ヘッダー](https://www.w3.org/TR/cors/#simple-response-header)」を参照してください。

既定で使用できる応答ヘッダーは次のとおりです。

* `Cache-Control`
* `Content-Language`
* `Content-Type`
* `Expires`
* `Last-Modified`
* `Pragma`

CORS 仕様は、これらのヘッダーの *単純な応答ヘッダー* を呼び出します。 アプリで他のヘッダーを使用できるようにするには、次のように呼び出し <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithExposedHeaders*> ます。

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=73-78&highlight=5)]

### <a name="credentials-in-cross-origin-requests"></a>クロスオリジン要求の資格情報

資格情報では、CORS 要求で特別な処理を行う必要があります。 既定では、ブラウザーはクロスオリジン要求で資格情報を送信しません。 資格情報には cookie 、および HTTP 認証スキームが含まれます。 クロスオリジン要求で資格情報を送信するには、クライアントがに設定されている必要があり `XMLHttpRequest.withCredentials` `true` ます。

直接の使用 `XMLHttpRequest` :

```javascript
var xhr = new XMLHttpRequest();
xhr.open('get', 'https://www.example.com/api/test');
xhr.withCredentials = true;
```

JQuery の使用:

```javascript
$.ajax({
  type: 'get',
  url: 'https://www.example.com/api/test',
  xhrFields: {
    withCredentials: true
  }
});
```

[FETCH API](https://developer.mozilla.org/docs/Web/API/Fetch_API)の使用:

```javascript
fetch('https://www.example.com/api/test', {
    credentials: 'include'
});
```

サーバーは資格情報を許可する必要があります。 クロスオリジンの資格情報を許可するには、次のように呼び出し <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowCredentials*> ます。

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=82-87&highlight=5)]

HTTP 応答には、 `Access-Control-Allow-Credentials` サーバーがクロスオリジン要求に対して資格情報を許可することをブラウザーに示すヘッダーが含まれています。

ブラウザーが資格情報を送信しても、応答に有効なヘッダーが含まれていない場合 `Access-Control-Allow-Credentials` 、ブラウザーはアプリへの応答を公開せず、クロスオリジン要求は失敗します。

クロスオリジンの資格情報を許可することは、セキュリティ上のリスクです。 別のドメインの web サイトは、ユーザーの知らないうちに、サインインしているユーザーの資格情報をユーザーの代わりにアプリに送信できます。 <!-- TODO Review: When using `AllowCredentials`, all CORS enabled domains must be trusted.
I don't like "all CORS enabled domains must be trusted", because it implies that if you're not using  `AllowCredentials`, domains don't need to be trusted. -->

また、CORS 仕様では、ヘッダーが存在する場合、[オリジン] を [(すべてのオリジン)] に設定することもでき `"*"` `Access-Control-Allow-Credentials` ます。

### <a name="preflight-requests"></a>プレフライト要求

一部の CORS 要求については、ブラウザーは実際の要求を行う前に追加の要求を送信します。 この要求は *プレフライト要求* と呼ばれます。 次の条件に該当する場合、ブラウザーはプレフライト要求をスキップできます。

* 要求メソッドは GET、HEAD、または POST です。
* アプリで `Accept` は、、、 `Accept-Language` `Content-Language` 、 `Content-Type` 、または `Last-Event-ID` 以外の要求ヘッダーは設定されません。
* ヘッダーには、設定されて `Content-Type` いる場合、次のいずれかの値が含まれます。
  * `application/x-www-form-urlencoded`
  * `multipart/form-data`
  * `text/plain`

クライアント要求に対して設定された要求ヘッダーに適用される規則は、オブジェクトに対してを呼び出すことによって、アプリが設定するヘッダーに適用され `setRequestHeader` `XMLHttpRequest` ます。 CORS 仕様は、これらのヘッダー *作成者要求ヘッダー* を呼び出します。 このルールは、ブラウザーが設定できるヘッダー (、、など) には適用されません `User-Agent` `Host` `Content-Length` 。

プレフライト要求の例を次に示します。

```
OPTIONS https://myservice.azurewebsites.net/api/test HTTP/1.1
Accept: */*
Origin: https://myclient.azurewebsites.net
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: accept, x-my-custom-header
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/6.0)
Host: myservice.azurewebsites.net
Content-Length: 0
```

事前フライト要求では、HTTP OPTIONS メソッドを使用します。 次の2つの特殊なヘッダーが含まれます。

* `Access-Control-Request-Method`: 実際の要求に使用される HTTP メソッド。
* `Access-Control-Request-Headers`: アプリが実際の要求に対して設定する要求ヘッダーのリスト。 前述のように、これにはブラウザーが設定するヘッダー (など) は含まれません `User-Agent` 。

<!-- I think this needs to be removed, was put here accidently -->

CORS が適切なポリシーで有効になっている場合、ASP.NET Core は通常、CORS プレフライト要求に自動的に応答します。 [プレフライト要求の場合は、[HttpOptions] 属性を](#pro)参照してください。

CORS のプレフライト要求には、 `Access-Control-Request-Headers` 実際の要求と共に送信されるヘッダーをサーバーに示すヘッダーを含めることができます。

特定のヘッダーを許可するには、次のように呼び出し <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithHeaders*> ます。

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=55-60&highlight=5)]

すべての作成者要求ヘッダーを許可するには、次のように呼び出し <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyHeader*> ます。

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=64-69&highlight=5)]

ブラウザーの設定方法が完全に一致しているわけではありません `Access-Control-Request-Headers` 。 ヘッダーを以外の任意の値 (またはを使用) に設定する場合は、少なくとも、、、 `"*"` <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicy.AllowAnyHeader*> `Accept` `Content-Type` `Origin` 、およびサポートするカスタムヘッダーを含める必要があります。

プレフライト要求に対する応答の例を次に示します (サーバーが要求を許可していることを前提としています)。

```
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Length: 0
Access-Control-Allow-Origin: https://myclient.azurewebsites.net
Access-Control-Allow-Headers: x-my-custom-header
Access-Control-Allow-Methods: PUT
Date: Wed, 20 May 2015 06:33:22 GMT
```

応答には、 `Access-Control-Allow-Methods` 許可されているメソッドと、必要に応じてヘッダーを一覧表示するヘッダーが含まれてい `Access-Control-Allow-Headers` ます。 プレフライト要求が成功した場合、ブラウザーは実際の要求を送信します。

プレフライト要求が拒否された場合、アプリは *200 OK* 応答を返しますが、CORS ヘッダーを返信しません。 そのため、ブラウザーはクロスオリジン要求を試行しません。

### <a name="set-the-preflight-expiration-time"></a>プレフライトの有効期限を設定する

ヘッダーは、 `Access-Control-Max-Age` プレフライト要求への応答をキャッシュできる期間を指定します。 このヘッダーを設定するには、次のように呼び出し <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.SetPreflightMaxAge*> ます。

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=91-96&highlight=5)]

<a name="how-cors"></a>

## <a name="how-cors-works"></a>CORS のしくみ

このセクションでは、HTTP メッセージレベルでの [CORS](https://developer.mozilla.org/docs/Web/HTTP/CORS) 要求の動作について説明します。

* CORS はセキュリティ機能では **ありません** 。 CORS は、サーバーが同じオリジンポリシーを緩めることを可能にする W3C 標準です。
  * たとえば、悪意のあるアクターがを使用してサイトに対して [クロスサイトスクリプティング (XSS) を防止](xref:security/cross-site-scripting) し、CORS が有効になっているサイトに対してクロスサイト要求を実行して情報を盗むことができます。
* CORS を許可することで、API の安全性が向上します。
  * CORS を適用するには、クライアント (ブラウザー) が必要です。 サーバーは要求を実行し、応答を返します。これは、エラーを返し、応答をブロックするクライアントです。 たとえば、次のツールを使用すると、サーバーの応答が表示されます。
    * [Fiddler](https://www.telerik.com/fiddler)
    * [Postman](https://www.getpostman.com/)
    * [.NET HttpClient](/dotnet/csharp/tutorials/console-webapiclient)
    * アドレスバーに URL を入力して、web ブラウザーを表示します。
* これは、ブラウザーがクロスオリジン [Xhr](https://developer.mozilla.org/docs/Web/API/XMLHttpRequest) または [Fetch API](https://developer.mozilla.org/docs/Web/API/Fetch_API) 要求を実行して、それ以外は禁止されるようにする方法です。
  * (CORS を使用しない) ブラウザーは、クロスオリジン要求を行うことができません。 CORS の前に、この制限を回避するために [JSONP](https://www.w3schools.com/js/js_json_jsonp.asp) が使用されていました。 JSONP は XHR を使用しないので、タグを使用して `<script>` 応答を受信します。 スクリプトをクロスオリジンで読み込むことができます。

[CORS 仕様](https://www.w3.org/TR/cors/)では、クロスオリジン要求を有効にする新しい HTTP ヘッダーがいくつか導入されました。 ブラウザーが CORS をサポートしている場合、クロスオリジン要求に対してこれらのヘッダーが自動的に設定されます。 CORS を有効にするためにカスタム JavaScript コードは必要ありません。

次に、クロスオリジン要求の例を示します。 ヘッダーは、 `Origin` 要求を行っているサイトのドメインを提供します。 `Origin`ヘッダーは必須であり、ホストとは異なる必要があります。

```
GET https://myservice.azurewebsites.net/api/test HTTP/1.1
Referer: https://myclient.azurewebsites.net/
Accept: */*
Accept-Language: en-US
Origin: https://myclient.azurewebsites.net
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/6.0)
Host: myservice.azurewebsites.net
```

サーバーが要求を許可している場合は、応答でヘッダーが設定され `Access-Control-Allow-Origin` ます。 このヘッダーの値は、要求のヘッダーと一致するか、 `Origin` またはワイルドカード値です `"*"` 。つまり、すべての配信元が許可されます。

```
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Type: text/plain; charset=utf-8
Access-Control-Allow-Origin: https://myclient.azurewebsites.net
Date: Wed, 20 May 2015 06:27:30 GMT
Content-Length: 12

Test message
```

応答にヘッダーが含まれていない場合 `Access-Control-Allow-Origin` 、クロスオリジン要求は失敗します。 具体的には、ブラウザーは要求を許可しません。 サーバーが応答を正常に返す場合でも、ブラウザーはクライアントアプリで応答を使用できません。

<a name="test"></a>

## <a name="test-cors"></a>CORS のテスト

CORS をテストするには:

1. [API プロジェクトを作成](xref:tutorials/first-web-api)します。 または、 [サンプルをダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/cors/sample/Cors)することもできます。
1. このドキュメントのいずれかの方法を使用して CORS を有効にします。 次に例を示します。

  [!code-csharp[](cors/sample/Cors/WebAPI/StartupTest.cs?name=snippet2&highlight=13-18)]

  > [!WARNING]
  > `WithOrigins("https://localhost:<port>");` サンプルアプリのテストにのみ使用してください [サンプルコードをダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/cors/sample/Cors)します。

1. Web アプリプロジェクトを作成 Razor します (ページまたは MVC)。 このサンプルでは、 Razor ページを使用します。 API プロジェクトと同じソリューションに web アプリを作成できます。
1. 次の強調表示されたコードを *インデックスの cshtml* ファイルに追加します。

  [!code-cshtml[](cors/sample/Cors/ClientApp/Pages/Index2.cshtml?highlight=7-99)]

1. 前のコードで、をデプロイされた `url: 'https://<web app>.azurewebsites.net/api/values/1',` アプリの URL に置き換えます。
1. API プロジェクトをデプロイします。 たとえば、 [Azure にデプロイ](xref:host-and-deploy/azure-apps/index)します。
1. Razorデスクトップからページまたは MVC アプリを実行し、[**テスト**] ボタンをクリックします。 F12 ツールを使用して、エラーメッセージを確認します。
1. から localhost の配信元を削除 `WithOrigins` し、アプリをデプロイします。 または、別のポートを使用してクライアントアプリを実行します。 たとえば、Visual Studio からを実行します。
1. クライアントアプリでテストします。 CORS エラーはエラーを返しますが、エラーメッセージは JavaScript では使用できません。 F12 ツールの [コンソール] タブを使用して、エラーを確認します。 ブラウザーによっては、次のようなエラー (F12 ツールコンソール) が表示されます。

   * Microsoft Edge を使用する:

     **SEC7120: [CORS] オリジンは、次の `https://localhost:44375` `https://localhost:44375` 場所にあるクロスオリジンリソースのアクセス制御-許可-オリジン応答ヘッダーで見つかりませんでした `https://webapi.azurewebsites.net/api/values/1`**

   * Chrome を使用する:

     **`https://webapi.azurewebsites.net/api/values/1`オリジンからの XMLHttpRequest へのアクセス `https://localhost:44375` が CORS ポリシーによってブロックされています。要求されたリソースに ' アクセス制御許可-発信元 ' ヘッダーが存在しません。**
     
CORS が有効なエンドポイントは、 [Fiddler](https://www.telerik.com/fiddler) や [Postman](https://www.getpostman.com/)などのツールを使用してテストできます。 ツールを使用する場合、ヘッダーによって指定された要求の配信元は、要求を受信している `Origin` ホストと異なる必要があります。 ヘッダーの値に基づいて要求が *クロスオリジン* でない場合は、 `Origin` 次のようになります。

* CORS ミドルウェアが要求を処理する必要はありません。
* CORS ヘッダーが応答で返されません。

## <a name="cors-in-iis"></a>IIS での CORS

IIS に展開する場合、サーバーが匿名アクセスを許可するように構成されていない場合、CORS は Windows 認証の前に実行する必要があります。 このシナリオをサポートするには、アプリ用に [IIS CORS モジュール](https://www.iis.net/downloads/microsoft/iis-cors-module) をインストールして構成する必要があります。

## <a name="additional-resources"></a>その他のリソース

* [クロスオリジンリソース共有 (CORS)](https://developer.mozilla.org/docs/Web/HTTP/CORS)
* [IIS CORS モジュールの概要](https://blogs.iis.net/iisteam/getting-started-with-the-iis-cors-module)

::: moniker-end
