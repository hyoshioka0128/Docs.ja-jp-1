---
title: ASP.NET Core の URL リライト ミドルウェア
author: rick-anderson
description: ASP.NET Core アプリケーションの URL リライト ミドルウェアを使用した URL の書き換えとリダイレクトについて説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/16/2019
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
uid: fundamentals/url-rewriting
ms.openlocfilehash: 44e6570090de9c5f3550d18a30fcde758fb16f85
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588524"
---
# <a name="url-rewriting-middleware-in-aspnet-core"></a>ASP.NET Core の URL リライト ミドルウェア

作成者: [Mikael Mengistu](https://github.com/mikaelm12)

::: moniker range=">= aspnetcore-3.0"

このドキュメントでは、ASP.NET Core アプリの URL リライト ミドルウェアを使用して URL を書き換える手順について説明します。

URL の書き換えは、1 つまたは複数の事前定義された規則に基づいて URL 要求を変更する操作です。 URL 書き換えは、リソースの場所とそれらのアドレスの間の抽象化を作成し、場所とアドレスが緊密にリンクされていないようにします。 URL 書き換えは複数のシナリオで使用できます。

* サーバー リソースを一時的または永続的に移動するか置き換えて、リソースに対する安定したロケーターを維持します。
* 要求処理を異なる複数のアプリまたは 1 つのアプリの複数の区分に分割します。
* 受信した要求の URL セグメントを削除、追加、または再編成します。
* 検索エンジン最適化 (SEO) のためにパブリック URL を最適化します。
* ビジターがリソースの要求から返される内容を予測しやすいように、フレンドリなパブリック URL の使用を許可します。
* セキュリティで保護されていない要求をセキュリティで保護されたエンドポイントにリダイレクトします。
* 外部サイトが別のサイトでホストされている静的資産を独自のコンテンツにリンクすることによってその資産を使用するホットリンクを防止します。

> [!NOTE]
> URL の書き換えによってアプリのパフォーマンスが低下することがあります。 可能であれば、ルールの複雑さと数を制限します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/url-rewriting/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="url-redirect-and-url-rewrite"></a>URL リダイレクトと URL 書き換え

"*URL リダイレクト*" と "*URL 書き換え*" では表現上はあまり違いがないように見えますが、クライアントへのリソースの提供に関しては重要な意味を持っています。 ASP.NET Core の URL リライト ミドルウェアは、両方のニーズを満たすことができます。

"*URL リダイレクト*" にはクライアント側の操作が関係しており、クライアントはもともと要求したものとは異なるアドレスにあるリソースにアクセスするよう指示されます。 これによりサーバーへのラウンドトリップが必要になります。 クライアントが、リソースの新しい要求を実行するときに、クライアントに返されたリダイレクト URL がブラウザーのアドレス バーに表示されます。

`/resource` が `/different-resource` に "*リダイレクトされる*" 場合、サーバーからの応答では、クライアントが `/different-resource` にあるリソースを取得する必要があることと、リダイレクトが一時的または永続的のどちらかを示す状態コードが示されます。

![WebAPI サービス エンドポイントは、サーバー上でバージョン 1 (v1) からバージョン 2 (v2) に一時的に変更されました。 クライアントは、バージョン 1 パス /v1/api のサービスに対して要求を実行します。 サーバーは、バージョン 2/v2/api のサービスの新しい一時的なパスを使用して 302 (検出) 応答を返します。 クライアントは、リダイレクト URL のサービスに対して 2 回目の要求を実行します。 サーバーは、200 (OK) 状態コードで応答します。](url-rewriting/_static/url_redirect.png)

要求を別の URL にリダイレクトする場合は、応答で状態コードを指定することにより、リダイレクトが永続的か一時的かのどちらかを示します。

* "*301 - 完全な移動*" 状態コードは、リソースに新しい永続的な URL があり、リソースに対する将来のすべての要求で新しい URL を使用する必要があることをクライアントに指示したい場合に使用します。 "*301 状態コードを受信すると、クライアントは応答をキャッシュに入れて再利用することができます。* "

* "*302 - 検出*" 状態コードは、リダイレクトが一時的である場合、または一般に変更される可能性がある場合に使用されます。 302 状態コードは、URL を保存して後で再利用しないようクライアントに指示します。

状態コードの詳細については、[RFC 2616: 状態コードの定義](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)に関するページを参照してください。

"*URL 書き換え*" はサーバー側の操作であり、クライアントが要求したものとは異なるリソース アドレスからリソースが提供されます。 URL 書き換えでは、サーバーへのラウンドトリップは必要ありません。 書き換えられた URL は、クライアントに返されず、ブラウザーのアドレス バーに表示されません。

`/resource` が `/different-resource` に "*書き換えられた*" 場合、サーバーは `/different-resource` にあるリソースを "*内部的に*" 取得して返します。

クライアントは、書き換えられた URL にあるリソースを取得できる場合がありますが、クライアントは、その要求を実行して応答を受信したときに、書き換えられた URL にリソースが存在することは通知されません。

![WebAPI サービス エンドポイントは、サーバー上でバージョン 1 (v1) からバージョン 2 (v2) に変更されました。 クライアントは、バージョン 1 パス /v1/api のサービスに対して要求を実行します。 要求 URL は、バージョン 2 パス/v2/api でサービスにアクセスするように再度書き込まれます。 サービスは、200 (OK) 状態コードでクライアントに応答します。](url-rewriting/_static/url_rewrite.png)

## <a name="url-rewriting-sample-app"></a>URL リライト サンプル アプリ

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/url-rewriting/samples/)を使用して、URL リライト ミドルウェアの機能を調べることができます。 そのアプリは、リダイレクトと書き換えのルールを適用し、複数のシナリオについて、リダイレクトされた URL または書き換えられた URL を表示します。

## <a name="when-to-use-url-rewriting-middleware"></a>URL リライト ミドルウェアを使用する状況

次の方法を使用できない場合は、URL リライト ミドルウェアを使用します。

* [Windows Server 上の IIS での URL リライト モジュール](https://www.iis.net/downloads/microsoft/url-rewrite)
* [Apache サーバー上の Apache mod_rewrite モジュール](https://httpd.apache.org/docs/2.4/rewrite/)
* [Nginx での URL 書き換え](https://www.nginx.com/blog/creating-nginx-rewrite-rules/)

また、アプリが [HTTP.sys サーバー](xref:fundamentals/servers/httpsys) (旧称 WebListener) でホストされているときも、ミドルウェアを使用します。

IIS、Apache、Nginx でサーバー ベースの URL 書き換えテクノロジが使用される主な理由は次のとおりです。

* ミドルウェアでは、これらのモジュールの完全な機能がサポートされていません。

  IIS 書き換えモジュールの `IsFile` 制約や `IsDirectory` 制約など、サーバー モジュールの一部の機能は、ASP.NET Core プロジェクトでは動作しません。 これらのシナリオでは、代わりにミドルウェアを使用します。
* 通常、ミドルウェアのパフォーマンスはモジュールより劣ります。

  どちらのアプローチの方がパフォーマンスの低下が大きいか、またはパフォーマンスが低下した場合でもごくわずかかどうかを確実に知るには、ベンチマークが唯一の方法です。

## <a name="package"></a>Package

URL リライト ミドルウェアは、[Microsoft.AspNetCore.Rewrite](https://www.nuget.org/packages/Microsoft.AspNetCore.Rewrite) パッケージによって提供されます。このパッケージは ASP.NET Core アプリに暗黙的に含まれています。

## <a name="extension-and-options"></a>拡張機能とオプション

各書き換えルールに対する拡張メソッドを含む [RewriteOptions](xref:Microsoft.AspNetCore.Rewrite.RewriteOptions) クラスのインスタンスを作成することにより、URL 書き換えとリダイレクトのルールを設定します。 処理したい順序で複数のルールを連結します。 <xref:Microsoft.AspNetCore.Builder.RewriteBuilderExtensions.UseRewriter*> によって URL リライト ミドルウェアが要求パイプラインに追加されると、`RewriteOptions` が渡されます。

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1)]

### <a name="redirect-non-www-to-www"></a>非 WWW を WWW にリダイレクトする

アプリで非 `www` の要求を `www` にリダイレクトできる 3 つのオプションがあります。

* <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToWwwPermanent*>:要求が非 `www` の場合、要求を `www` サブドメインに永続的にリダイレクトします。 [Status308PermanentRedirect](xref:Microsoft.AspNetCore.Http.StatusCodes.Status308PermanentRedirect) 状態コードでリダイレクトします。

* <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToWww*>:受信した要求が非 `www` の場合、要求を `www` サブドメインにリダイレクトします。 [Status307TemporaryRedirect](xref:Microsoft.AspNetCore.Http.StatusCodes.Status307TemporaryRedirect) 状態コードでリダイレクトします。 オーバーロードにより、応答に対する状態コードを提供することができます。 状態コードの割り当てには、<xref:Microsoft.AspNetCore.Http.StatusCodes> クラスのフィールドを使用します。

### <a name="url-redirect"></a>URL リダイレクト

要求をリダイレクトするには、<xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirect*> を使用します。 最初のパラメーターには、着信 URL のパスに一致する正規表現が含まれています。 2 番目のパラメーターは、置換文字列です。 3 番目のパラメーター (存在する場合) は、状態コードを指定します。 状態コードを指定しない場合、状態コードは既定で "*302 - 検出*" に設定されます。これは、リソースが一時的に移動されるか置き換えられることを示します。

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=9)]

開発者ツールが有効になっているブラウザーで、パス `/redirect-rule/1234/5678` を指定してサンプル アプリに対する要求を実行します。 正規表現が `redirect-rule/(.*)` の要求のパスに一致し、パスが `/redirected/1234/5678` に置き換えられます。 リダイレクト URL が、"*302 - 検出*" 状態コードでクライアントに返されます。 ブラウザーは、ブラウザーのアドレス バーに表示されるリダイレクト URL に新しい要求を実行します リダイレクト URL にはサンプル アプリに一致するルールがないため、次のようになります。

* 2 番目の要求は、アプリから *200 - OK* 応答を受け取ります。
* 応答の本文では、リダイレクト URL が示されています。

URL が "*リダイレクト*" されるときに、サーバーへのラウンドトリップが実行されます。

> [!WARNING]
> リダイレクト ルールを設定するときは注意してください。 リダイレクト ルールは、リダイレクト後を含めて、アプリへのすべての要求で評価されます。 "*無限リダイレクトのループ*" が誤って作成されることがよくあります。

元の要求: `/redirect-rule/1234/5678`

![要求および応答を追跡する開発者ツールを備えたブラウザー ウィンドウ](url-rewriting/_static/add_redirect.png)

かっこ内に含まれる式の一部は、*キャプチャ グループ* と呼ばれます。 式のドット (`.`) は、*どの文字とも一致する* ことを意味します。 アスタリスク (`*`) は、*直前の文字に 0 回以上一致する* ことを示します。 そのため、URL の最後の 2 つのパス セグメント `1234/5678` は、キャプチャ グループ `(.*)` によってキャプチャされます。 要求 URL で `redirect-rule/` の後に指定した任意の値が、この単一のキャプチャ グループによってキャプチャされます。

置換文字列では、キャプチャされたグループがドル記号 (`$`) を使用して文字列に挿入され、その後にキャプチャのシーケンス番号が続きます。 最初のキャプチャ グループ値は、`$1` で取得され、2 番目は `$2` で取得され、これらは正規表現内のキャプチャ グループの順序で続行されます。 サンプル アプリでは、リダイレクト ルールの正規表現内に 1 つのキャプチャ グループだけがあるので、置換文字列に 1 つの挿入されたグループ (`$1`) だけがあります。 ルールが適用されるときに、URL `/redirected/1234/5678` になります。

### <a name="url-redirect-to-a-secure-endpoint"></a>セキュリティで保護されたエンドポイントへの URL リダイレクト

HTTP 要求を、同じホストとパスに HTTPS プロトコルを使用してリダイレクトするには、<xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToHttps*> を使用します。 状態コードが指定されていない場合、ミドルウェアは既定で "*302 - 検出*" に設定します。 ポートが指定されていない場合は、次のようになります。

* ミドルウェアの既定値 `null` に設定されます。
* スキームは `https` (HTTPS プロトコル) に変更されて、クライアントはポート 443 でリソースにアクセスします。

次の例では、状態コードを "*301 - 完全な移動*" に設定し、ポートを 5001 に変更する方法を示します。

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .AddRedirectToHttps(301, 5001);

    app.UseRewriter(options);
}
```

セキュリティで保護されていない要求を、セキュリティで保護された HTTPS プロトコル (ポート 443) を使用して同じホストとパスにリダイレクトするには、<xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToHttpsPermanent*> を使用します。 ミドルウェアは状態コードを "*301 -完全な移動"* に設定します。

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .AddRedirectToHttpsPermanent();

    app.UseRewriter(options);
}
```

> [!NOTE]
> リダイレクト規則を追加せずにセキュリティで保護されたエンドポイントにリダイレクトするときは、HTTPS リダイレクト ミドルウェアを使用することをお勧めします。 詳細については、「[HTTPS の適用](xref:security/enforcing-ssl#require-https)」のトピックをご覧ください。

サンプル アプリは、`AddRedirectToHttps` または `AddRedirectToHttpsPermanent` の使用方法の例を示すことができます。 拡張メソッドを `RewriteOptions` に追加します。 任意の URL でセキュリティ保護されていない要求をアプリに対して行います。 自己署名証明書が信頼できないことを示すブラウザーのセキュリティ警告を消去するか、証明書を信頼する例外を作成します。

`AddRedirectToHttps(301, 5001)`: `http://localhost:5000/secure` を使用する元の要求

![要求および応答を追跡する開発者ツールを備えたブラウザー ウィンドウ](url-rewriting/_static/add_redirect_to_https.png)

`AddRedirectToHttpsPermanent`: `http://localhost:5000/secure` を使用する元の要求

![要求および応答を追跡する開発者ツールを備えたブラウザー ウィンドウ](url-rewriting/_static/add_redirect_to_https_permanent.png)

### <a name="url-rewrite"></a>URL 書き換え

URL 書き換えのルールを作成するには、<xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRewrite*> を使用します。 最初のパラメーターには、着信 URL のパスに一致する正規表現が含まれています。 2 番目のパラメーターは、置換文字列です。 3 番目のパラメーター `skipRemainingRules: {true|false}` は、現在のルールが適用される場合に追加の書き換えルールをスキップするかどうかをミドルウェアに示します。

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=10-11)]

元の要求: `/rewrite-rule/1234/5678`

![要求および応答を追跡する開発者ツールを備えたブラウザー ウィンドウ](url-rewriting/_static/add_rewrite.png)

式の先頭にあるキャレット (`^`) は、URL パスの先頭からマッチングが開始されることを意味します。

前のリダイレクト ルール `redirect-rule/(.*)` の例では、正規表現の先頭にキャレット (`^`) がありません。 したがって、パスの `redirect-rule/` の前にどのような文字があっても一致します。

| パス                               | 一致したもの |
| ---------------------------------- | :---: |
| `/redirect-rule/1234/5678`         | はい   |
| `/my-cool-redirect-rule/1234/5678` | はい   |
| `/anotherredirect-rule/1234/5678`  | はい   |

書き換えルール `^rewrite-rule/(\d+)/(\d+)` は、`rewrite-rule/` で始まる場合のみパスと一致します。 次の表では、一致の違いに注意してください。

| パス                              | 一致したもの |
| --------------------------------- | :---: |
| `/rewrite-rule/1234/5678`         | はい   |
| `/my-cool-rewrite-rule/1234/5678` | いいえ    |
| `/anotherrewrite-rule/1234/5678`  | いいえ    |

式の `^rewrite-rule/` に続く部分に、2 つのキャプチャ グループ `(\d+)/(\d+)` があります。 `\d` は、*1 桁 (数字) に一致する* ことを示します。 プラス記号 (`+`) は、*直前の 1 つ以上の文字に一致する* ことを意味します。 そのため、URL は、数字とその後に続くスラッシュ、さらにその後に続く別の数字を含んでいる必要があります。 これらのキャプチャ グループが `$1` および `$2` として書き換えられた URL に挿入されます。 書き換えルールの置換文字列は、クエリ文字列にキャプチャされたグループを配置します。 `/rewrite-rule/1234/5678` の要求されたパスは、`/rewritten?var1=1234&var2=5678` でリソースを取得するように書き換えられます。 クエリ文字列が元の要求に存在する場合、URL が書き換えられるときに保持されます。

リソースを取得するためのサーバーへのラウンドトリップはありません。 リソースが存在する場合は、取得され、*200 - OK* 状態コードと共にクライアントに返されます。 クライアントはリダイレクトされていないため、ブラウザーのアドレス バーの URL は変更されません。 クライアントは、URL の書き換え操作がサーバーで発生したことを検出できません。

> [!NOTE]
> 式内の照合ルールは計算量が多く、アプリの応答時間が長くなるので、可能な限り `skipRemainingRules: true` を使用します。 最も高速なアプリの応答を実現するには以下のようにします。
>
> * 一致する頻度が最も多いルールから一致頻度が最も少ないルールへの順番で書き換えルールを並べ替えます。
> * 一致が発生し、追加の処理が必要ないときに残りのルールの処理をスキップします。

### <a name="apache-mod_rewrite"></a>Apache mod_rewrite

<xref:Microsoft.AspNetCore.Rewrite.ApacheModRewriteOptionsExtensions.AddApacheModRewrite*> を使用して Apache mod_rewrite ルールを適用します。 ルール ファイルがアプリと共に展開されていることを確認します。 mod_rewrite ルールの情報と例については、「[Apache mod_rewrite](https://httpd.apache.org/docs/2.4/rewrite/)」を参照してください。

*ApacheModRewrite.txt* ルール ファイルからルールを読み取るには、<xref:System.IO.StreamReader> を使用します。

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=3-4,12)]

サンプル アプリは、要求を `/apache-mod-rules-redirect/(.\*)` から `/redirected?id=$1` にリダイレクトします。 応答の状態コードは "*302 -検出*" です。

[!code[](url-rewriting/samples/3.x/SampleApp/ApacheModRewrite.txt)]

元の要求: `/apache-mod-rules-redirect/1234`

![要求および応答を追跡する開発者ツールを備えたブラウザー ウィンドウ](url-rewriting/_static/add_apache_mod_redirect.png)

ミドルウェアは、次の Apache mod_rewrite サーバー変数をサポートします。

* CONN_REMOTE_ADDR
* HTTP_ACCEPT
* HTTP_CONNECTION
* HTTP_COOKIE
* HTTP_FORWARDED
* HTTP_HOST
* HTTP_REFERER
* HTTP_USER_AGENT
* HTTPS
* IPV6
* QUERY_STRING
* REMOTE_ADDR
* REMOTE_PORT
* REQUEST_FILENAME
* REQUEST_METHOD
* REQUEST_SCHEME
* REQUEST_URI
* SCRIPT_FILENAME
* SERVER_ADDR
* SERVER_PORT
* SERVER_PROTOCOL
* TIME
* TIME_DAY
* TIME_HOUR
* TIME_MIN
* TIME_MON
* TIME_SEC
* TIME_WDAY
* TIME_YEAR

### <a name="iis-url-rewrite-module-rules"></a>IIS URL リライト モジュールのルール

IIS URL リライト モジュールに適用される同じルール セットを使用するには、<xref:Microsoft.AspNetCore.Rewrite.IISUrlRewriteOptionsExtensions.AddIISUrlRewrite*> を使用します。 ルール ファイルがアプリと共に展開されていることを確認します。 Windows Server IIS で実行されているときにミドルウェアでアプリの *web.config* ファイルを使用するように指定しないでください。 IIS を使用する場合、IIS リライト モジュールとの競合を避けるため、これらのルールをアプリの *web.config* の外部に保存する必要があります。 IIS URL リライト モジュールのルールの詳細と例については、「[Using Url Rewrite Module 2.0](/iis/extensions/url-rewrite-module/using-url-rewrite-module-20)」(URL リライト モジュール 2.0 の使用 ) と「[URL Rewrite Module Configuration Reference](/iis/extensions/url-rewrite-module/url-rewrite-module-configuration-reference)」(URL リライト モジュール構成リファレンス) を参照してください。

*IISUrlRewrite.xml* ルール ファイルからルールを読み取るには、<xref:System.IO.StreamReader> を使用します。

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=5-6,13)]

サンプル アプリは、要求を `/iis-rules-rewrite/(.*)` から `/rewritten?id=$1` に書き換えます。 応答は、*200 - OK* 状態コードでクライアントに送信されます。

[!code-xml[](url-rewriting/samples/3.x/SampleApp/IISUrlRewrite.xml)]

元の要求: `/iis-rules-rewrite/1234`

![要求および応答を追跡する開発者ツールを備えたブラウザー ウィンドウ](url-rewriting/_static/add_iis_url_rewrite.png)

望ましくない方法でアプリに影響するように構成されたサーバー レベル ルールを使用するアクティブな IIS リライト モジュールがある場合は、アプリに対して IIS リライト モジュールを無効にできます。 詳細については、「[Disabling IIS modules](xref:host-and-deploy/iis/modules#disabling-iis-modules)」 (IIS モジュールの無効化) を参照してください。

#### <a name="unsupported-features"></a>サポートされていない機能

ミドルウェアでは、次の IIS URL リライト モジュールの機能がサポートされていません。

* 送信ルール
* カスタム サーバー変数
* ワイルドカード
* LogRewrittenUrl

#### <a name="supported-server-variables"></a>サポートされるサーバー変数

ミドルウェアは、次の IIS URL リライト モジュール サーバー変数をサポートします。

* CONTENT_LENGTH
* CONTENT_TYPE
* HTTP_ACCEPT
* HTTP_CONNECTION
* HTTP_COOKIE
* HTTP_HOST
* HTTP_REFERER
* HTTP_URL
* HTTP_USER_AGENT
* HTTPS
* LOCAL_ADDR
* QUERY_STRING
* REMOTE_ADDR
* REMOTE_PORT
* REQUEST_FILENAME
* REQUEST_URI

> [!NOTE]
> <xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider> を介して <xref:Microsoft.Extensions.FileProviders.IFileProvider> を取得することもできます。 このアプローチは、書き換えルール ファイルの場所に関する大きな柔軟性を提供する場合があります。 書き換えルール ファイルを指定したパスでサーバーに導入されていることを確認してください。
>
> ```csharp
> PhysicalFileProvider fileProvider = new PhysicalFileProvider(Directory.GetCurrentDirectory());
> ```

### <a name="method-based-rule"></a>メソッド ベースのルール

メソッド内で独自のルール ロジックを実装するには、<xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.Add*> を使用します。 `Add` は <xref:Microsoft.AspNetCore.Rewrite.RewriteContext> を公開し、メソッドで <xref:Microsoft.AspNetCore.Http.HttpContext> を使用できるようにします。 [RewriteContext.Result](xref:Microsoft.AspNetCore.Rewrite.RewriteContext.Result*) は、追加のパイプライン処理の実行方法を決定します。 次の表で説明する <xref:Microsoft.AspNetCore.Rewrite.RuleResult> フィールドのいずれかに値を設定します。

| コンテキストの書き換えの結果               | アクション                                                           |
| ------------------------------------ | ---------------------------------------------------------------- |
| `RuleResult.ContinueRules` (既定値) | ルールの適用を続けます。                                         |
| `RuleResult.EndResponse`             | ルールの適用を停止し、応答を送信します。                       |
| `RuleResult.SkipRemainingRules`      | ルールの適用を停止し、次のミドルウェアにコンテキストを送信します。 |

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=14)]

サンプル アプリは、 *.xml* で終了するパスの要求をリダイレクトするメソッドの例を示します。 `/file.xml` に対して要求が行われた場合、要求は `/xmlfiles/file.xml` にリダイレクトされます。 状態コードは "*301 - 完全な移動*" に設定されます。 ブラウザーが */xmlfiles/file.xml* に対する新しい要求を行うと、静的ファイル ミドルウェアはクライアントに *wwwroot/xmlfiles* フォルダーからファイルを提供します。 リダイレクトの場合は、応答の状態コードを明示的に設定します。 それ以外の場合は、*200 - OK* 状態コードが返され、クライアントでのリダイレクトは発生しません。

*RewriteRules.cs*:

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/RewriteRules.cs?name=snippet_RedirectXmlFileRequests&highlight=14-18)]

このアプローチでは、要求を書き換えることもできます。 サンプル アプリでは、*wwwroot* フォルダーから *file.txt* テキスト ファイルを提供するためにテキスト ファイル要求のパスを書き換える処理が示されています。 静的ファイル ミドルウェアでは、更新された要求パスに基づいてファイルが提供されます。

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=15,22)]

*RewriteRules.cs*:

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/RewriteRules.cs?name=snippet_RewriteTextFileRequests&highlight=7-8)]

### <a name="irule-based-rule"></a>IRule ベースのルール

<xref:Microsoft.AspNetCore.Rewrite.IRule> インターフェイスを実装するクラス内のルール ロジックを使用するには、<xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.Add*> を使用します。 `IRule` では、メソッド ベースのルール アプローチを使用する場合より高い柔軟性が提供されます。 実装クラスには、<xref:Microsoft.AspNetCore.Rewrite.IRule.ApplyRule*> メソッドに対してパラメーターを渡すことができるコンストラクターを含めることができます。

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=16-17)]

サンプル アプリの `extension` と `newPath` のパラメーターの値は、いくつかの条件を満たすかチェックされます。 `extension` は、値を含んでいる必要があり、値は *.png*、 *.jpg*、または *.gif* でなければなりません。 `newPath` が有効ではない場合、<xref:System.ArgumentException> がスローされます。 *image.png* に対して行われた要求は、`/png-images/image.png` にリダイレクトされます。 *image.jpg* に対して行われた要求は、`/jpg-images/image.jpg` にリダイレクトされます。 状態コードは "*301 - 完全な移動*" に設定され、`context.Result` はルールの処理を停止して応答を送信するように設定されます。

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/RewriteRules.cs?name=snippet_RedirectImageRequests)]

元の要求: `/image.png`

![image.png の要求および応答を追跡する開発者ツールを備えたブラウザー ウィンドウ](url-rewriting/_static/add_redirect_png_requests.png)

元の要求: `/image.jpg`

![image.jpg の要求および応答を追跡する開発者ツールを備えたブラウザー ウィンドウ](url-rewriting/_static/add_redirect_jpg_requests.png)

## <a name="regex-examples"></a>正規表現の例

| Goal | 正規表現文字列と<br>一致の例 | 置換文字列と<br>出力の例 |
| ---- | ------------------------------- | -------------------------------------- |
| クエリ文字列内のパスを書き換える | `^path/(.*)/(.*)`<br>`/path/abc/123` | `path?var1=$1&var2=$2`<br>`/path?var1=abc&var2=123` |
| 末尾のスラッシュを除去する | `(.*)/$`<br>`/path/` | `$1`<br>`/path` |
| 末尾のスラッシュを強制する | `(.*[^/])$`<br>`/path` | `$1/`<br>`/path/` |
| 特定の要求を書き換えを回避する | `^(.*)(?<!\.axd)$` または `^(?!.*\.axd$)(.*)$`<br>はい: `/resource.htm`<br>いいえ: `/resource.axd` | `rewritten/$1`<br>`/rewritten/resource.htm`<br>`/resource.axd` |
| URL セグメントを再配置する | `path/(.*)/(.*)/(.*)`<br>`path/1/2/3` | `path/$3/$2/$1`<br>`path/3/2/1` |
| URL セグメントを置き換える | `^(.*)/segment2/(.*)`<br>`/segment1/segment2/segment3` | `$1/replaced/$2`<br>`/segment1/replaced/segment3` |

::: moniker-end

::: moniker range="< aspnetcore-3.0"

このドキュメントでは、ASP.NET Core アプリの URL リライト ミドルウェアを使用して URL を書き換える手順について説明します。

URL の書き換えは、1 つまたは複数の事前定義された規則に基づいて URL 要求を変更する操作です。 URL 書き換えは、リソースの場所とそれらのアドレスの間の抽象化を作成し、場所とアドレスが緊密にリンクされていないようにします。 URL 書き換えは複数のシナリオで使用できます。

* サーバー リソースを一時的または永続的に移動するか置き換えて、リソースに対する安定したロケーターを維持します。
* 要求処理を異なる複数のアプリまたは 1 つのアプリの複数の区分に分割します。
* 受信した要求の URL セグメントを削除、追加、または再編成します。
* 検索エンジン最適化 (SEO) のためにパブリック URL を最適化します。
* ビジターがリソースの要求から返される内容を予測しやすいように、フレンドリなパブリック URL の使用を許可します。
* セキュリティで保護されていない要求をセキュリティで保護されたエンドポイントにリダイレクトします。
* 外部サイトが別のサイトでホストされている静的資産を独自のコンテンツにリンクすることによってその資産を使用するホットリンクを防止します。

> [!NOTE]
> URL の書き換えによってアプリのパフォーマンスが低下することがあります。 可能であれば、ルールの複雑さと数を制限します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/url-rewriting/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="url-redirect-and-url-rewrite"></a>URL リダイレクトと URL 書き換え

"*URL リダイレクト*" と "*URL 書き換え*" では表現上はあまり違いがないように見えますが、クライアントへのリソースの提供に関しては重要な意味を持っています。 ASP.NET Core の URL リライト ミドルウェアは、両方のニーズを満たすことができます。

"*URL リダイレクト*" にはクライアント側の操作が関係しており、クライアントはもともと要求したものとは異なるアドレスにあるリソースにアクセスするよう指示されます。 これによりサーバーへのラウンドトリップが必要になります。 クライアントが、リソースの新しい要求を実行するときに、クライアントに返されたリダイレクト URL がブラウザーのアドレス バーに表示されます。

`/resource` が `/different-resource` に "*リダイレクトされる*" 場合、サーバーからの応答では、クライアントが `/different-resource` にあるリソースを取得する必要があることと、リダイレクトが一時的または永続的のどちらかを示す状態コードが示されます。

![WebAPI サービス エンドポイントは、サーバー上でバージョン 1 (v1) からバージョン 2 (v2) に一時的に変更されました。 クライアントは、バージョン 1 パス /v1/api のサービスに対して要求を実行します。 サーバーは、バージョン 2/v2/api のサービスの新しい一時的なパスを使用して 302 (検出) 応答を返します。 クライアントは、リダイレクト URL のサービスに対して 2 回目の要求を実行します。 サーバーは、200 (OK) 状態コードで応答します。](url-rewriting/_static/url_redirect.png)

要求を別の URL にリダイレクトする場合は、応答で状態コードを指定することにより、リダイレクトが永続的か一時的かのどちらかを示します。

* "*301 - 完全な移動*" 状態コードは、リソースに新しい永続的な URL があり、リソースに対する将来のすべての要求で新しい URL を使用する必要があることをクライアントに指示したい場合に使用します。 "*301 状態コードを受信すると、クライアントは応答をキャッシュに入れて再利用することができます。* "

* "*302 - 検出*" 状態コードは、リダイレクトが一時的である場合、または一般に変更される可能性がある場合に使用されます。 302 状態コードは、URL を保存して後で再利用しないようクライアントに指示します。

状態コードの詳細については、[RFC 2616: 状態コードの定義](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)に関するページを参照してください。

"*URL 書き換え*" はサーバー側の操作であり、クライアントが要求したものとは異なるリソース アドレスからリソースが提供されます。 URL 書き換えでは、サーバーへのラウンドトリップは必要ありません。 書き換えられた URL は、クライアントに返されず、ブラウザーのアドレス バーに表示されません。

`/resource` が `/different-resource` に "*書き換えられた*" 場合、サーバーは `/different-resource` にあるリソースを "*内部的に*" 取得して返します。

クライアントは、書き換えられた URL にあるリソースを取得できる場合がありますが、クライアントは、その要求を実行して応答を受信したときに、書き換えられた URL にリソースが存在することは通知されません。

![WebAPI サービス エンドポイントは、サーバー上でバージョン 1 (v1) からバージョン 2 (v2) に変更されました。 クライアントは、バージョン 1 パス /v1/api のサービスに対して要求を実行します。 要求 URL は、バージョン 2 パス/v2/api でサービスにアクセスするように再度書き込まれます。 サービスは、200 (OK) 状態コードでクライアントに応答します。](url-rewriting/_static/url_rewrite.png)

## <a name="url-rewriting-sample-app"></a>URL リライト サンプル アプリ

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/url-rewriting/samples/)を使用して、URL リライト ミドルウェアの機能を調べることができます。 そのアプリは、リダイレクトと書き換えのルールを適用し、複数のシナリオについて、リダイレクトされた URL または書き換えられた URL を表示します。

## <a name="when-to-use-url-rewriting-middleware"></a>URL リライト ミドルウェアを使用する状況

次の方法を使用できない場合は、URL リライト ミドルウェアを使用します。

* [Windows Server 上の IIS での URL リライト モジュール](https://www.iis.net/downloads/microsoft/url-rewrite)
* [Apache サーバー上の Apache mod_rewrite モジュール](https://httpd.apache.org/docs/2.4/rewrite/)
* [Nginx での URL 書き換え](https://www.nginx.com/blog/creating-nginx-rewrite-rules/)

また、アプリが [HTTP.sys サーバー](xref:fundamentals/servers/httpsys) (旧称 WebListener) でホストされているときも、ミドルウェアを使用します。

IIS、Apache、Nginx でサーバー ベースの URL 書き換えテクノロジが使用される主な理由は次のとおりです。

* ミドルウェアでは、これらのモジュールの完全な機能がサポートされていません。

  IIS 書き換えモジュールの `IsFile` 制約や `IsDirectory` 制約など、サーバー モジュールの一部の機能は、ASP.NET Core プロジェクトでは動作しません。 これらのシナリオでは、代わりにミドルウェアを使用します。
* 通常、ミドルウェアのパフォーマンスはモジュールより劣ります。

  どちらのアプローチの方がパフォーマンスの低下が大きいか、またはパフォーマンスが低下した場合でもごくわずかかどうかを確実に知るには、ベンチマークが唯一の方法です。

## <a name="package"></a>Package

ミドルウェアをプロジェクトに組み込むには、[Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)へのパッケージ参照をプロジェクト ファイルに追加します。これには、[Microsoft.AspNetCore.Rewrite](https://www.nuget.org/packages/Microsoft.AspNetCore.Rewrite) パッケージが含まれます。

`Microsoft.AspNetCore.App` メタパッケージを使用しない場合は、`Microsoft.AspNetCore.Rewrite` パッケージへのプロジェクト参照を追加します。

## <a name="extension-and-options"></a>拡張機能とオプション

各書き換えルールに対する拡張メソッドを含む [RewriteOptions](xref:Microsoft.AspNetCore.Rewrite.RewriteOptions) クラスのインスタンスを作成することにより、URL 書き換えとリダイレクトのルールを設定します。 処理したい順序で複数のルールを連結します。 <xref:Microsoft.AspNetCore.Builder.RewriteBuilderExtensions.UseRewriter*> によって URL リライト ミドルウェアが要求パイプラインに追加されると、`RewriteOptions` が渡されます。

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1)]

### <a name="redirect-non-www-to-www"></a>非 WWW を WWW にリダイレクトする

アプリで非 `www` の要求を `www` にリダイレクトできる 3 つのオプションがあります。

* <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToWwwPermanent*>:要求が非 `www` の場合、要求を `www` サブドメインに永続的にリダイレクトします。 [Status308PermanentRedirect](xref:Microsoft.AspNetCore.Http.StatusCodes.Status308PermanentRedirect) 状態コードでリダイレクトします。

* <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToWww*>:受信した要求が非 `www` の場合、要求を `www` サブドメインにリダイレクトします。 [Status307TemporaryRedirect](xref:Microsoft.AspNetCore.Http.StatusCodes.Status307TemporaryRedirect) 状態コードでリダイレクトします。 オーバーロードにより、応答に対する状態コードを提供することができます。 状態コードの割り当てには、<xref:Microsoft.AspNetCore.Http.StatusCodes> クラスのフィールドを使用します。

### <a name="url-redirect"></a>URL リダイレクト

要求をリダイレクトするには、<xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirect*> を使用します。 最初のパラメーターには、着信 URL のパスに一致する正規表現が含まれています。 2 番目のパラメーターは、置換文字列です。 3 番目のパラメーター (存在する場合) は、状態コードを指定します。 状態コードを指定しない場合、状態コードは既定で "*302 - 検出*" に設定されます。これは、リソースが一時的に移動されるか置き換えられることを示します。

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=9)]

開発者ツールが有効になっているブラウザーで、パス `/redirect-rule/1234/5678` を指定してサンプル アプリに対する要求を実行します。 正規表現が `redirect-rule/(.*)` の要求のパスに一致し、パスが `/redirected/1234/5678` に置き換えられます。 リダイレクト URL が、"*302 - 検出*" 状態コードでクライアントに返されます。 ブラウザーは、ブラウザーのアドレス バーに表示されるリダイレクト URL に新しい要求を実行します リダイレクト URL にはサンプル アプリに一致するルールがないため、次のようになります。

* 2 番目の要求は、アプリから *200 - OK* 応答を受け取ります。
* 応答の本文では、リダイレクト URL が示されています。

URL が "*リダイレクト*" されるときに、サーバーへのラウンドトリップが実行されます。

> [!WARNING]
> リダイレクト ルールを設定するときは注意してください。 リダイレクト ルールは、リダイレクト後を含めて、アプリへのすべての要求で評価されます。 "*無限リダイレクトのループ*" が誤って作成されることがよくあります。

元の要求: `/redirect-rule/1234/5678`

![要求および応答を追跡する開発者ツールを備えたブラウザー ウィンドウ](url-rewriting/_static/add_redirect.png)

かっこ内に含まれる式の一部は、*キャプチャ グループ* と呼ばれます。 式のドット (`.`) は、*どの文字とも一致する* ことを意味します。 アスタリスク (`*`) は、*直前の文字に 0 回以上一致する* ことを示します。 そのため、URL の最後の 2 つのパス セグメント `1234/5678` は、キャプチャ グループ `(.*)` によってキャプチャされます。 要求 URL で `redirect-rule/` の後に指定した任意の値が、この単一のキャプチャ グループによってキャプチャされます。

置換文字列では、キャプチャされたグループがドル記号 (`$`) を使用して文字列に挿入され、その後にキャプチャのシーケンス番号が続きます。 最初のキャプチャ グループ値は、`$1` で取得され、2 番目は `$2` で取得され、これらは正規表現内のキャプチャ グループの順序で続行されます。 サンプル アプリでは、リダイレクト ルールの正規表現内に 1 つのキャプチャ グループだけがあるので、置換文字列に 1 つの挿入されたグループ (`$1`) だけがあります。 ルールが適用されるときに、URL `/redirected/1234/5678` になります。

### <a name="url-redirect-to-a-secure-endpoint"></a>セキュリティで保護されたエンドポイントへの URL リダイレクト

HTTP 要求を、同じホストとパスに HTTPS プロトコルを使用してリダイレクトするには、<xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToHttps*> を使用します。 状態コードが指定されていない場合、ミドルウェアは既定で "*302 - 検出*" に設定します。 ポートが指定されていない場合は、次のようになります。

* ミドルウェアの既定値 `null` に設定されます。
* スキームは `https` (HTTPS プロトコル) に変更されて、クライアントはポート 443 でリソースにアクセスします。

次の例では、状態コードを "*301 - 完全な移動*" に設定し、ポートを 5001 に変更する方法を示します。

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .AddRedirectToHttps(301, 5001);

    app.UseRewriter(options);
}
```

セキュリティで保護されていない要求を、セキュリティで保護された HTTPS プロトコル (ポート 443) を使用して同じホストとパスにリダイレクトするには、<xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToHttpsPermanent*> を使用します。 ミドルウェアは状態コードを "*301 -完全な移動"* に設定します。

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .AddRedirectToHttpsPermanent();

    app.UseRewriter(options);
}
```

> [!NOTE]
> リダイレクト規則を追加せずにセキュリティで保護されたエンドポイントにリダイレクトするときは、HTTPS リダイレクト ミドルウェアを使用することをお勧めします。 詳細については、「[HTTPS の適用](xref:security/enforcing-ssl#require-https)」のトピックをご覧ください。

サンプル アプリは、`AddRedirectToHttps` または `AddRedirectToHttpsPermanent` の使用方法の例を示すことができます。 拡張メソッドを `RewriteOptions` に追加します。 任意の URL でセキュリティ保護されていない要求をアプリに対して行います。 自己署名証明書が信頼できないことを示すブラウザーのセキュリティ警告を消去するか、証明書を信頼する例外を作成します。

`AddRedirectToHttps(301, 5001)`: `http://localhost:5000/secure` を使用する元の要求

![要求および応答を追跡する開発者ツールを備えたブラウザー ウィンドウ](url-rewriting/_static/add_redirect_to_https.png)

`AddRedirectToHttpsPermanent`: `http://localhost:5000/secure` を使用する元の要求

![要求および応答を追跡する開発者ツールを備えたブラウザー ウィンドウ](url-rewriting/_static/add_redirect_to_https_permanent.png)

### <a name="url-rewrite"></a>URL 書き換え

URL 書き換えのルールを作成するには、<xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRewrite*> を使用します。 最初のパラメーターには、着信 URL のパスに一致する正規表現が含まれています。 2 番目のパラメーターは、置換文字列です。 3 番目のパラメーター `skipRemainingRules: {true|false}` は、現在のルールが適用される場合に追加の書き換えルールをスキップするかどうかをミドルウェアに示します。

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=10-11)]

元の要求: `/rewrite-rule/1234/5678`

![要求および応答を追跡する開発者ツールを備えたブラウザー ウィンドウ](url-rewriting/_static/add_rewrite.png)

式の先頭にあるキャレット (`^`) は、URL パスの先頭からマッチングが開始されることを意味します。

前のリダイレクト ルール `redirect-rule/(.*)` の例では、正規表現の先頭にキャレット (`^`) がありません。 したがって、パスの `redirect-rule/` の前にどのような文字があっても一致します。

| パス                               | 一致したもの |
| ---------------------------------- | :---: |
| `/redirect-rule/1234/5678`         | はい   |
| `/my-cool-redirect-rule/1234/5678` | はい   |
| `/anotherredirect-rule/1234/5678`  | はい   |

書き換えルール `^rewrite-rule/(\d+)/(\d+)` は、`rewrite-rule/` で始まる場合のみパスと一致します。 次の表では、一致の違いに注意してください。

| パス                              | 一致したもの |
| --------------------------------- | :---: |
| `/rewrite-rule/1234/5678`         | はい   |
| `/my-cool-rewrite-rule/1234/5678` | いいえ    |
| `/anotherrewrite-rule/1234/5678`  | いいえ    |

式の `^rewrite-rule/` に続く部分に、2 つのキャプチャ グループ `(\d+)/(\d+)` があります。 `\d` は、*1 桁 (数字) に一致する* ことを示します。 プラス記号 (`+`) は、*直前の 1 つ以上の文字に一致する* ことを意味します。 そのため、URL は、数字とその後に続くスラッシュ、さらにその後に続く別の数字を含んでいる必要があります。 これらのキャプチャ グループが `$1` および `$2` として書き換えられた URL に挿入されます。 書き換えルールの置換文字列は、クエリ文字列にキャプチャされたグループを配置します。 `/rewrite-rule/1234/5678` の要求されたパスは、`/rewritten?var1=1234&var2=5678` でリソースを取得するように書き換えられます。 クエリ文字列が元の要求に存在する場合、URL が書き換えられるときに保持されます。

リソースを取得するためのサーバーへのラウンドトリップはありません。 リソースが存在する場合は、取得され、*200 - OK* 状態コードと共にクライアントに返されます。 クライアントはリダイレクトされていないため、ブラウザーのアドレス バーの URL は変更されません。 クライアントは、URL の書き換え操作がサーバーで発生したことを検出できません。

> [!NOTE]
> 式内の照合ルールは計算量が多く、アプリの応答時間が長くなるので、可能な限り `skipRemainingRules: true` を使用します。 最も高速なアプリの応答を実現するには以下のようにします。
>
> * 一致する頻度が最も多いルールから一致頻度が最も少ないルールへの順番で書き換えルールを並べ替えます。
> * 一致が発生し、追加の処理が必要ないときに残りのルールの処理をスキップします。

### <a name="apache-mod_rewrite"></a>Apache mod_rewrite

<xref:Microsoft.AspNetCore.Rewrite.ApacheModRewriteOptionsExtensions.AddApacheModRewrite*> を使用して Apache mod_rewrite ルールを適用します。 ルール ファイルがアプリと共に展開されていることを確認します。 mod_rewrite ルールの情報と例については、「[Apache mod_rewrite](https://httpd.apache.org/docs/2.4/rewrite/)」を参照してください。

*ApacheModRewrite.txt* ルール ファイルからルールを読み取るには、<xref:System.IO.StreamReader> を使用します。

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=3-4,12)]

サンプル アプリは、要求を `/apache-mod-rules-redirect/(.\*)` から `/redirected?id=$1` にリダイレクトします。 応答の状態コードは "*302 -検出*" です。

[!code[](url-rewriting/samples/2.x/SampleApp/ApacheModRewrite.txt)]

元の要求: `/apache-mod-rules-redirect/1234`

![要求および応答を追跡する開発者ツールを備えたブラウザー ウィンドウ](url-rewriting/_static/add_apache_mod_redirect.png)

ミドルウェアは、次の Apache mod_rewrite サーバー変数をサポートします。

* CONN_REMOTE_ADDR
* HTTP_ACCEPT
* HTTP_CONNECTION
* HTTP_COOKIE
* HTTP_FORWARDED
* HTTP_HOST
* HTTP_REFERER
* HTTP_USER_AGENT
* HTTPS
* IPV6
* QUERY_STRING
* REMOTE_ADDR
* REMOTE_PORT
* REQUEST_FILENAME
* REQUEST_METHOD
* REQUEST_SCHEME
* REQUEST_URI
* SCRIPT_FILENAME
* SERVER_ADDR
* SERVER_PORT
* SERVER_PROTOCOL
* TIME
* TIME_DAY
* TIME_HOUR
* TIME_MIN
* TIME_MON
* TIME_SEC
* TIME_WDAY
* TIME_YEAR

### <a name="iis-url-rewrite-module-rules"></a>IIS URL リライト モジュールのルール

IIS URL リライト モジュールに適用される同じルール セットを使用するには、<xref:Microsoft.AspNetCore.Rewrite.IISUrlRewriteOptionsExtensions.AddIISUrlRewrite*> を使用します。 ルール ファイルがアプリと共に展開されていることを確認します。 Windows Server IIS で実行されているときにミドルウェアでアプリの *web.config* ファイルを使用するように指定しないでください。 IIS を使用する場合、IIS リライト モジュールとの競合を避けるため、これらのルールをアプリの *web.config* の外部に保存する必要があります。 IIS URL リライト モジュールのルールの詳細と例については、「[Using Url Rewrite Module 2.0](/iis/extensions/url-rewrite-module/using-url-rewrite-module-20)」(URL リライト モジュール 2.0 の使用 ) と「[URL Rewrite Module Configuration Reference](/iis/extensions/url-rewrite-module/url-rewrite-module-configuration-reference)」(URL リライト モジュール構成リファレンス) を参照してください。

*IISUrlRewrite.xml* ルール ファイルからルールを読み取るには、<xref:System.IO.StreamReader> を使用します。

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=5-6,13)]

サンプル アプリは、要求を `/iis-rules-rewrite/(.*)` から `/rewritten?id=$1` に書き換えます。 応答は、*200 - OK* 状態コードでクライアントに送信されます。

[!code-xml[](url-rewriting/samples/2.x/SampleApp/IISUrlRewrite.xml)]

元の要求: `/iis-rules-rewrite/1234`

![要求および応答を追跡する開発者ツールを備えたブラウザー ウィンドウ](url-rewriting/_static/add_iis_url_rewrite.png)

望ましくない方法でアプリに影響するように構成されたサーバー レベル ルールを使用するアクティブな IIS リライト モジュールがある場合は、アプリに対して IIS リライト モジュールを無効にできます。 詳細については、「[Disabling IIS modules](xref:host-and-deploy/iis/modules#disabling-iis-modules)」 (IIS モジュールの無効化) を参照してください。

#### <a name="unsupported-features"></a>サポートされていない機能

ASP.NET Core 2.x でリリースされたミドルウェアは、次の IIS URL リライト モジュールの機能をサポートしていません。

* 送信ルール
* カスタム サーバー変数
* ワイルドカード
* LogRewrittenUrl

#### <a name="supported-server-variables"></a>サポートされるサーバー変数

ミドルウェアは、次の IIS URL リライト モジュール サーバー変数をサポートします。

* CONTENT_LENGTH
* CONTENT_TYPE
* HTTP_ACCEPT
* HTTP_CONNECTION
* HTTP_COOKIE
* HTTP_HOST
* HTTP_REFERER
* HTTP_URL
* HTTP_USER_AGENT
* HTTPS
* LOCAL_ADDR
* QUERY_STRING
* REMOTE_ADDR
* REMOTE_PORT
* REQUEST_FILENAME
* REQUEST_URI

> [!NOTE]
> <xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider> を介して <xref:Microsoft.Extensions.FileProviders.IFileProvider> を取得することもできます。 このアプローチは、書き換えルール ファイルの場所に関する大きな柔軟性を提供する場合があります。 書き換えルール ファイルを指定したパスでサーバーに導入されていることを確認してください。
>
> ```csharp
> PhysicalFileProvider fileProvider = new PhysicalFileProvider(Directory.GetCurrentDirectory());
> ```

### <a name="method-based-rule"></a>メソッド ベースのルール

メソッド内で独自のルール ロジックを実装するには、<xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.Add*> を使用します。 `Add` は <xref:Microsoft.AspNetCore.Rewrite.RewriteContext> を公開し、メソッドで <xref:Microsoft.AspNetCore.Http.HttpContext> を使用できるようにします。 [RewriteContext.Result](xref:Microsoft.AspNetCore.Rewrite.RewriteContext.Result*) は、追加のパイプライン処理の実行方法を決定します。 次の表で説明する <xref:Microsoft.AspNetCore.Rewrite.RuleResult> フィールドのいずれかに値を設定します。

| コンテキストの書き換えの結果               | アクション                                                           |
| ------------------------------------ | ---------------------------------------------------------------- |
| `RuleResult.ContinueRules` (既定値) | ルールの適用を続けます。                                         |
| `RuleResult.EndResponse`             | ルールの適用を停止し、応答を送信します。                       |
| `RuleResult.SkipRemainingRules`      | ルールの適用を停止し、次のミドルウェアにコンテキストを送信します。 |

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=14)]

サンプル アプリは、 *.xml* で終了するパスの要求をリダイレクトするメソッドの例を示します。 `/file.xml` に対して要求が行われた場合、要求は `/xmlfiles/file.xml` にリダイレクトされます。 状態コードは "*301 - 完全な移動*" に設定されます。 ブラウザーが */xmlfiles/file.xml* に対する新しい要求を行うと、静的ファイル ミドルウェアはクライアントに *wwwroot/xmlfiles* フォルダーからファイルを提供します。 リダイレクトの場合は、応答の状態コードを明示的に設定します。 それ以外の場合は、*200 - OK* 状態コードが返され、クライアントでのリダイレクトは発生しません。

*RewriteRules.cs*:

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/RewriteRules.cs?name=snippet_RedirectXmlFileRequests&highlight=14-18)]

このアプローチでは、要求を書き換えることもできます。 サンプル アプリでは、*wwwroot* フォルダーから *file.txt* テキスト ファイルを提供するためにテキスト ファイル要求のパスを書き換える処理が示されています。 静的ファイル ミドルウェアでは、更新された要求パスに基づいてファイルが提供されます。

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=15,22)]

*RewriteRules.cs*:

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/RewriteRules.cs?name=snippet_RewriteTextFileRequests&highlight=7-8)]

### <a name="irule-based-rule"></a>IRule ベースのルール

<xref:Microsoft.AspNetCore.Rewrite.IRule> インターフェイスを実装するクラス内のルール ロジックを使用するには、<xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.Add*> を使用します。 `IRule` では、メソッド ベースのルール アプローチを使用する場合より高い柔軟性が提供されます。 実装クラスには、<xref:Microsoft.AspNetCore.Rewrite.IRule.ApplyRule*> メソッドに対してパラメーターを渡すことができるコンストラクターを含めることができます。

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=16-17)]

サンプル アプリの `extension` と `newPath` のパラメーターの値は、いくつかの条件を満たすかチェックされます。 `extension` は、値を含んでいる必要があり、値は *.png*、 *.jpg*、または *.gif* でなければなりません。 `newPath` が有効ではない場合、<xref:System.ArgumentException> がスローされます。 *image.png* に対して行われた要求は、`/png-images/image.png` にリダイレクトされます。 *image.jpg* に対して行われた要求は、`/jpg-images/image.jpg` にリダイレクトされます。 状態コードは "*301 - 完全な移動*" に設定され、`context.Result` はルールの処理を停止して応答を送信するように設定されます。

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/RewriteRules.cs?name=snippet_RedirectImageRequests)]

元の要求: `/image.png`

![image.png の要求および応答を追跡する開発者ツールを備えたブラウザー ウィンドウ](url-rewriting/_static/add_redirect_png_requests.png)

元の要求: `/image.jpg`

![image.jpg の要求および応答を追跡する開発者ツールを備えたブラウザー ウィンドウ](url-rewriting/_static/add_redirect_jpg_requests.png)

## <a name="regex-examples"></a>正規表現の例

| Goal | 正規表現文字列と<br>一致の例 | 置換文字列と<br>出力の例 |
| ---- | ------------------------------- | -------------------------------------- |
| クエリ文字列内のパスを書き換える | `^path/(.*)/(.*)`<br>`/path/abc/123` | `path?var1=$1&var2=$2`<br>`/path?var1=abc&var2=123` |
| 末尾のスラッシュを除去する | `(.*)/$`<br>`/path/` | `$1`<br>`/path` |
| 末尾のスラッシュを強制する | `(.*[^/])$`<br>`/path` | `$1/`<br>`/path/` |
| 特定の要求を書き換えを回避する | `^(.*)(?<!\.axd)$` または `^(?!.*\.axd$)(.*)$`<br>はい: `/resource.htm`<br>いいえ: `/resource.axd` | `rewritten/$1`<br>`/rewritten/resource.htm`<br>`/resource.axd` |
| URL セグメントを再配置する | `path/(.*)/(.*)/(.*)`<br>`path/1/2/3` | `path/$3/$2/$1`<br>`path/3/2/1` |
| URL セグメントを置き換える | `^(.*)/segment2/(.*)`<br>`/segment1/segment2/segment3` | `$1/replaced/$2`<br>`/segment1/replaced/segment3` |

::: moniker-end

## <a name="additional-resources"></a>その他の技術情報

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* [.NET の正規表現](/dotnet/articles/standard/base-types/regular-expressions)
* [正規表現言語 - クイック リファレンス](/dotnet/articles/standard/base-types/quick-ref)
* [Apache mod_rewrite](https://httpd.apache.org/docs/2.4/rewrite/)
* [URL リライト モジュール 2.0 (IIS 用) の使用](/iis/extensions/url-rewrite-module/using-url-rewrite-module-20)
* [URL リライト モジュール構成リファレンス](/iis/extensions/url-rewrite-module/url-rewrite-module-configuration-reference)
* [IIS URL リライト モジュール フォーラム](https://forums.iis.net/1152.aspx)
* [単純な URL 構造を保持する](https://support.google.com/webmasters/answer/76329?hl=en)
* [URL 書き換えの 10 のヒントとテクニック](https://ruslany.net/2009/04/10-url-rewriting-tips-and-tricks/)
* [スラッシュを使用すべきかどうか](https://webmasters.googleblog.com/2010/04/to-slash-or-not-to-slash.html)
