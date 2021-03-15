---
title: ASP.NET Core のミドルウェア
author: rick-anderson
description: ASP.NET Core のミドルウェアと要求パイプラインについて説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/15/2020
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
uid: fundamentals/middleware/index
ms.openlocfilehash: 15d011e88ab291173668a0b6dc5f46e97fdfeff0
ms.sourcegitcommit: acfe51c35497a204f75c2a61125c9408c04493e6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102605712"
---
# <a name="aspnet-core-middleware"></a>ASP.NET Core のミドルウェア

::: moniker range=">= aspnetcore-3.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)、[Steve Smith](https://ardalis.com/)

ミドルウェアとは、要求と応答を処理するために、アプリのパイプラインに組み込まれたソフトウェアです。 各コンポーネントで次の処理を実行します。

* 要求をパイプライン内の次のコンポーネントに渡すかどうかを選択します。
* パイプラインの次のコンポーネントの前または後に作業を実行できます。

要求デリゲートは、要求パイプラインの構築に使用されます。 要求デリゲートは、各 HTTP 要求を処理します。

要求デリゲートは、<xref:Microsoft.AspNetCore.Builder.RunExtensions.Run%2A>、<xref:Microsoft.AspNetCore.Builder.MapExtensions.Map%2A>、<xref:Microsoft.AspNetCore.Builder.UseExtensions.Use%2A> の各拡張メソッドを使って構成されます。 個々の要求デリゲートは、匿名メソッドとしてインラインで指定するか (インライン ミドルウェアと呼ばれます)、または再利用可能なクラスで定義することができます。 これらの再利用可能なクラスとインラインの匿名メソッドは、"*ミドルウェア*" です。"*ミドルウェア コンポーネント*" とも呼ばれます。 要求パイプライン内の各ミドルウェア コンポーネントは、パイプラインの次のコンポーネントを呼び出すか、パイプラインをショートさせます。 ショートサーキットしたミドルウェアは "*ターミナル ミドルウェア*" と呼ばれます。これによってさらなるミドルウェアによる要求の処理が回避されるためです。

<xref:migration/http-modules> では、ASP.NET Core と ASP.NET 4.x の要求パイプラインの違いについて説明し、その他のミドルウェア サンプルが提供されています。

## <a name="create-a-middleware-pipeline-with-iapplicationbuilder"></a>IApplicationBuilder を使用したミドルウェア パイプラインの作成

ASP.NET Core 要求パイプラインは、順番に呼び出される一連の要求デリゲートで構成されています。 次の図は、その概念を示しています。 実行のスレッドは黒い矢印をたどります。

![要求の到着、3 つのミドルウェアによる処理、アプリからの応答の送信を示す要求処理パターン。 各ミドルウェアは、そのロジックを実行し、next() ステートメントで要求を次のミドルウェアに渡します。 3 番目のミドルウェアが要求を処理した後、要求は前の 2 つのミドルウェアを逆の順序で通過して、それぞれの next() ステートメントの後の追加処理が行われた後、クライアントへの応答としてアプリを終了します。](index/_static/request-delegate-pipeline.png)

各デリゲートは、次のデリゲートの前と後に操作を実行できます。 例外処理デリゲートは、パイプラインの後のステージで発生する例外をキャッチできるように、パイプラインの早い段階で呼び出される必要があります。

考えられる最も簡単な ASP.NET Core アプリは、1 つの要求デリゲートを設定してすべての要求を処理するものです。 この場合、実際の要求パイプラインは含まれません。 代わりに、すべての HTTP 要求に対応して単一の匿名関数が呼び出されます。

[!code-csharp[](index/snapshot/Middleware/Startup.cs)]

複数の要求デリゲートを <xref:Microsoft.AspNetCore.Builder.UseExtensions.Use%2A> と一緒にチェーンします。 `next` パラメーターは、パイプラインの次のデリゲートを表します *next* パラメーターを "*呼び出さない*" ことで、パイプラインをショートさせることができます。 次の例で示すように、通常は、次のデリゲートの前と後の両方でアクションを実行できます。

[!code-csharp[](index/snapshot/Chain/Startup.cs?highlight=5-10)]

デリゲートが次のデリゲートに要求を渡さない場合、これは "*要求パイプラインのショートサーキット*" と呼ばれます。 不要な処理を回避するために、ショートさせることが望ましい場合がよくあります。 たとえば、[静的ファイル ミドルウェア](xref:fundamentals/static-files)は、静的ファイルの要求を処理して残りのパイプラインをショートサーキットすることにより、"*ターミナル ミドルウェア*" として動作させることができます。 後続の処理を終了させるミドルウェアの前にパイプラインに追加されたミドルウェアでは、その `next.Invoke` ステートメントの後も引き続きコードが処理されます。 ただし、既に送信された応答に対する書き込みの試行については、次の警告を参照してください。

> [!WARNING]
> 応答がクライアントに送信された後に、`next.Invoke` を呼び出さないでください。 応答が開始した後で <xref:Microsoft.AspNetCore.Http.HttpResponse> を変更すると、例外がスローされます。 たとえば、[ヘッダーや状態コードを設定すると、例外がスローされます](xref:performance/performance-best-practices#do-not-modify-the-status-code-or-headers-after-the-response-body-has-started)。 `next` を呼び出した後で応答本文に書き込むと、次のようになります。
>
> * プロトコル違反が発生する可能性があります。 たとえば、示されている `Content-Length` より多くを書き込んだ場合。
> * 本文の形式が破損する可能性があります。 たとえば、CSS ファイルに HTML フッターを書き込んだ場合。
>
> <xref:Microsoft.AspNetCore.Http.HttpResponse.HasStarted%2A> は、ヘッダーが送信されたかどうかや本文が書き込まれたかどうかを示すために役立つヒントです。

<xref:Microsoft.AspNetCore.Builder.RunExtensions.Run%2A> デリゲートでは、`next` パラメーターは受け取られません。 最初の `Run` デリゲートが常に終点となり、パイプラインが終了されます。 `Run` は規則です。 一部のミドルウェア コンポーネントでは、パイプラインの最後に実行される `Run[Middleware]` メソッドが公開されることがあります。

[!code-csharp[](index/snapshot/Chain/Startup.cs?highlight=12-15)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

前の例では、`Run` デリゲートによって応答に `"Hello from 2nd delegate."` が書き込まれ、その後、パイプラインが終了となります。 別の `Use` または `Run` デリゲートが `Run` デリゲートの後に追加される場合、そのデリゲートは呼び出されません。

<a name="order"></a>

## <a name="middleware-order"></a>ミドルウェアの順序

次の図は、ASP.NET Core MVC と Razor Pages アプリの完全な要求処理パイプラインを示しています。 一般的なアプリでどのように既存のミドルウェアが順序付けされ、どこにカスタム ミドルウェアが追加されるかを確認できます。 シナリオでの必要性に応じて、既存のミドルウェアの順序を変更したり、新しいカスタム ミドルウェアを挿入したりする方法については、完全に制御できます。

![ASP.NET Core のミドルウェア パイプライン](index/_static/middleware-pipeline.svg)

前の図の **エンドポイント** ミドルウェアでは、対応するアプリの種類 (MVC または Razor Pages) のフィルター パイプラインが実行されます。

![ASP.NET Core のフィルター パイプライン](index/_static/mvc-endpoint.svg)

`Startup.Configure` メソッドでミドルウェア コンポーネントを追加する順序は、要求でミドルウェア コンポーネントが呼び出される順序および応答での逆の順序を定義します。 この順序は、セキュリティ、パフォーマンス、および機能にとって **重要** です。

次の `Startup.Configure` メソッドでは、セキュリティ関連のミドルウェア コンポーネントが、一般的な推奨される順序で追加されています。

[!code-csharp[](index/snapshot/StartupAll3.cs?name=snippet)]

上のコードでは以下の操作が行われます。

* [個人のユーザー アカウント](xref:security/authentication/identity)を使用して新しい Web アプリを作成するときに追加されないミドルウェアは、コメント アウトされています。
* すべてのミドルウェアを厳密にこの順序で追加する必要があるわけではありませんが、多くはそうする必要があります。 次に例を示します。
  * `UseCors`、`UseAuthentication`、`UseAuthorization` は、示されている順序で追加する必要があります。
  * 現時点では、[こちらのバグ](https://github.com/dotnet/aspnetcore/issues/23218)のため、`UseResponseCaching` の前に `UseCors` を追加する必要があります。

一部のシナリオでは、ミドルウェアの順序が異なる場合があります。 たとえば、キャッシュと圧縮の順序はシナリオ固有のものであり、複数の有効な順序があります。 次に例を示します。

```csharp
app.UseResponseCaching();
app.UseResponseCompression();
```

上記のコードでは、圧縮された応答をキャッシュすることによって CPU を節約できますが、gzip や brotli などの異なる圧縮アルゴリズムを使用してリソースの複数の表現をキャッシュすることができます。

次の順序では、圧縮された静的ファイルをキャッシュできるように、静的ファイルが結合されます。

```csharp
app.UseResponseCaching();
app.UseResponseCompression();
app.UseStaticFiles();
```

次の `Startup.Configure` メソッドにより、共通アプリ シナリオのためのミドルウェア コンポーネントが追加されます。

1. 例外/エラー処理
   * 開発環境でアプリを実行する場合:
     * 開発者例外ページ ミドルウェア (<xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage%2A>) によりアプリの実行時エラーが報告されます。
     * データベース エラー ページ ミドルウェアによりデータベースの実行時エラーが報告されます。
   * 運用環境でアプリを実行する場合:
     * 例外ハンドラー ミドルウェア (<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A>) によって、後続のミドルウェアによってスローされた例外がキャッチされます。
     * HTTP Strict Transport Security プロトコル (HSTS) ミドルウェア (<xref:Microsoft.AspNetCore.Builder.HstsBuilderExtensions.UseHsts%2A>) により `Strict-Transport-Security` ヘッダーが追加されます。
1. HTTPS リダイレクト ミドルウェア (<xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection%2A>) により、HTTP 要求が HTTPS にリダイレクトされます。
1. 静的ファイル ミドルウェア (<xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A>) によって静的ファイルが返され、さらなる要求の処理がスキップされます。
1. Cookie ポリシー ミドルウェア (<xref:Microsoft.AspNetCore.Builder.CookiePolicyAppBuilderExtensions.UseCookiePolicy%2A>) により、アプリを EU の一般データ保護規制 (GDPR) に準拠させます。
1. ルーティング ミドルウェア (<xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseRouting%2A>) により、要求がルーティングされます。
1. 認証ミドルウェア (<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A>) により、ユーザーがセキュリティで保護されたリソースにアクセスする前に、ユーザーの認証が試行されます。
1. 承認ミドルウェア (<xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization%2A>) により、ユーザーがセキュリティで保護されたリソースにアクセスすることが承認されます。
1. セッション ミドルウェア (<xref:Microsoft.AspNetCore.Builder.SessionMiddlewareExtensions.UseSession%2A>) により、セッション状態が確立され保持されます。 アプリでセッション状態が使用されている場合は、Cookie ポリシー ミドルウェアの後、MVC ミドルウェアの前に、セッション ミドルウェアを呼び出します。
1. エンドポイント ルーティング ミドルウェア (<xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapRazorPages%2A> を含む <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints%2A>) により、Razor Pages エンドポイントが要求パイプラインに追加されます。

<!--

FUTURE UPDATE

On the next topic overhaul/release update, add API crosslink to "Database Error Page Middleware" in Item 1 of the list ...

Microsoft.AspNetCore.Builder.DatabaseErrorPageExtensions.UseDatabaseErrorPage*

... when available via the API docs.

-->

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
        app.UseDatabaseErrorPage();
    }
    else
    {
        app.UseExceptionHandler("/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseCookiePolicy();
    app.UseRouting();
    app.UseAuthentication();
    app.UseAuthorization();
    app.UseSession();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapRazorPages();
    });
}
```

前のコード例では、各ミドルウェアの拡張メソッドが <xref:Microsoft.AspNetCore.Builder?displayProperty=fullName> 名前空間を通じて <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder> で公開されています。

パイプラインに追加された最初のミドルウェア コンポーネントは <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> です。 そのため、例外ハンドラー ミドルウェアでは、以降の呼び出しで発生するすべての例外がキャッチされます。

静的ファイル ミドルウェアはパイプラインの早い段階で呼び出されるので、要求を処理し、残りのコンポーネントを通過せずにショートさせることができます。 静的ファイル ミドルウェアでは、承認チェックは提供 **されません**。 *wwwroot* の下にあるものも含め、この静的ファイル ミドルウェアによって提供されるすべてのファイルは、一般に公開されます。 静的ファイルを保護する方法については、「<xref:fundamentals/static-files>」を参照してください。

要求が静的ファイル ミドルウェアによって処理されない場合、要求は認証を実行する認証ミドルウェア (<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A>) に渡されます。 認証は、認証されない要求をショートさせません。 認証ミドルウェアは要求を認証しますが、承認 (および却下) は、MVC が特定の Razor ページまたは MVC コントローラーとアクションを選んだ後でのみ行われます。

次の例は、静的ファイルの要求が応答圧縮ミドルウェアの前に静的ファイル ミドルウェアによって処理される、ミドルウェアの順序を示します。 静的ファイルは、このミドルウェアの順序では圧縮されません。 Razor Pages の応答は圧縮できます。

```csharp
public void Configure(IApplicationBuilder app)
{
    // Static files aren't compressed by Static File Middleware.
    app.UseStaticFiles();

    app.UseRouting();

    app.UseResponseCompression();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapRazorPages();
    });
}
```

シングルページ アプリケーション (SPA) の場合、通常はミドルウェア パイプラインの最後に SPA ミドルウェア <xref:Microsoft.Extensions.DependencyInjection.SpaStaticFilesExtensions.UseSpaStaticFiles%2A> を配置します。 SPA ミドルウェアは、次のために最後になります。

* 対応する要求に最初にその他のミドルウェアが応答するため。
* クライアント側ルーティングを使用する SPA がサーバー アプリに認識されないすべてのルートを実行するため。

SPA の詳細については、[React](xref:spa/react) と [Angular](xref:spa/angular) プロジェクト テンプレートのガイドを参照してください。

### <a name="forwarded-headers-middleware-order"></a>Forwarded Headers Middleware の順序

[!INCLUDE[](~/includes/ForwardedHeaders.md)]

## <a name="branch-the-middleware-pipeline"></a>ミドルウェア パイプラインを分岐する

<xref:Microsoft.AspNetCore.Builder.MapExtensions.Map%2A> 拡張メソッドは、パイプラインを分岐する規則として使われます。 `Map` は、指定された要求パスの一致に基づいて、要求パイプラインを分岐します。 要求パスが指定されたパスで開始する場合、分岐が実行されます。

[!code-csharp[](index/snapshot/Chain/StartupMap.cs)]

次の表は、前のコードを使用した `http://localhost:1234` からの要求および応答を示しています。

| 要求             | 応答                     |
| ------------------- | ---------------------------- |
| localhost:1234      | Hello from non-Map delegate. |
| localhost:1234/map1 | Map Test 1                   |
| localhost:1234/map2 | Map Test 2                   |
| localhost:1234/map3 | Hello from non-Map delegate. |

`Map` を使用すると、一致したパス セグメントが `HttpRequest.Path` から削除され、要求ごとに `HttpRequest.PathBase` に追加されます。

`Map` は入れ子をサポートします。次にその例を示します。

```csharp
app.Map("/level1", level1App => {
    level1App.Map("/level2a", level2AApp => {
        // "/level1/level2a" processing
    });
    level1App.Map("/level2b", level2BApp => {
        // "/level1/level2b" processing
    });
});
```

`Map` では、次のように一度に複数のセグメントを照合できます。

[!code-csharp[](index/snapshot/Chain/StartupMultiSeg.cs?highlight=13)]

<xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen%2A> は、指定された述語の結果に基づいて、要求パイプラインを分岐します。 `Func<HttpContext, bool>` という型の任意の述語を使って、要求をパイプラインの新しい分岐にマップできます。 次の例では、クエリ文字列変数 `branch` の存在を検出するために術後が使用されます。

[!code-csharp[](index/snapshot/Chain/StartupMapWhen.cs?highlight=14-15)]

次の表は、前のコードを使用した `http://localhost:1234` からの要求および応答を示しています。

| 要求                       | 応答                     |
| ----------------------------- | ---------------------------- |
| localhost:1234                | Hello from non-Map delegate. |
| localhost:1234/?branch=main | Branch used = main         |

また <xref:Microsoft.AspNetCore.Builder.UseWhenExtensions.UseWhen%2A> では、指定された述語の結果に基づいて、要求パイプラインが分岐されます。 `MapWhen` とは異なり、この分岐は、ショートしたり、ターミナル ミドルウェアが含まれたりしなければ、メイン パイプラインに再参加します。

[!code-csharp[](index/snapshot/Chain/StartupUseWhen.cs?highlight=18-19)]

前の例では、"Hello from main pipeline." の応答が すべての要求に対して書き込まれます。 要求にクエリ文字列変数 `branch` が含まれる場合、メイン パイプラインの再参加前にその値がログに記録されます。

## <a name="built-in-middleware"></a>組み込みミドルウェア

ASP.NET Core には、次のミドルウェア コンポーネントが付属しています。 "*順番*" 列には、要求を処理するパイプライン内のミドルウェアの配置と、ミドルウェアが要求処理を終了する条件に関するメモが記載されています。 ミドルウェアが要求を処理するパイプラインをショートサーキットし、下流のさらなるミドルウェアによる要求の処理を回避する場合、これは "*ターミナル ミドルウェア*" と呼ばれます。 ショートサーキットについて詳しくは、「[IApplicationBuilder を使用したミドルウェア パイプラインの作成](#create-a-middleware-pipeline-with-iapplicationbuilder)」セクションをご覧ください。

| ミドルウェア | 説明 | 順番 |
| ---------- | ----------- | ----- |
| [認証](xref:security/authentication/identity) | 認証のサポートを提供します。 | `HttpContext.User` が必要な場所の前。 OAuth コールバックの終端。 |
| [承認](xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization%2A) | 承認のサポートを提供します。 | 認証ミドルウェアの直後。 |
| [Cookie ポリシー](xref:security/gdpr) | 個人情報の保存に関してユーザーからの同意を追跡し、`secure` や `SameSite` など、cookie フィールドの最小要件を適用します。 | cookie を発行するミドルウェアの前。 次に例を示します。 認証、セッション、MVC (TempData) |
| [CORS](xref:security/cors) | クロス オリジン リソース共有を構成します。 | CORS を使うコンポーネントの前。 現時点では、[こちらのバグ](https://github.com/dotnet/aspnetcore/issues/23218)のため、`UseResponseCaching` の前に `UseCors` を追加する必要があります。|
| [診断](xref:fundamentals/error-handling) | 開発者の例外ページ、例外処理、状態コード ページ、および新しいアプリの既定の Web ページを提供する複数の独立したミドルウェア。 | エラーを生成するコンポーネントの前。 例外または新しいアプリ用の既定の Web ページの提供の終端。 |
| [転送されるヘッダー](xref:host-and-deploy/proxy-load-balancer) | プロキシされたヘッダーを現在の要求に転送します。 | 更新されたフィールドを使用するコンポーネントの前。 例: スキーム、ホスト、クライアント IP、メソッド。 |
| [正常性チェック](xref:host-and-deploy/health-checks) | ASP.NET Core アプリとその依存関係の正常性を、データベースの可用性などで確認します。 | 要求が正常性チェックのエンドポイントと一致した場合の終端。 |
| [ヘッダーの伝達](xref:fundamentals/http-requests#header-propagation-middleware) | HTTP ヘッダーを受信要求から送信 HTTP クライアント要求に伝達します。 |
| [HTTP メソッドのオーバーライド](xref:Microsoft.AspNetCore.Builder.HttpMethodOverrideExtensions) | メソッドをオーバーライドする受信 POST 要求を許可します。 | 更新されたメソッドを使うコンポーネントの前。 |
| [HTTPS リダイレクト](xref:security/enforcing-ssl#require-https) | すべての HTTP 要求を HTTPS にリダイレクトします。 | URL を使うコンポーネントの前。 |
| [HTTP Strict Transport Security (HSTS)](xref:security/enforcing-ssl#http-strict-transport-security-protocol-hsts) | 特殊な応答ヘッダーを追加するセキュリティ拡張機能のミドルウェア。 | 応答が送信される前と要求を変更するコンポーネントの後。 次に例を示します。 転送されるヘッダー、URL リライト。 |
| [MVC](xref:mvc/overview) | MVC または Razor Pages で要求を処理します。 | 要求がルートと一致した場合の終端。 |
| [OWIN](xref:fundamentals/owin) | OWIN ベースのアプリ、サーバー、およびミドルウェアと相互運用します。 | OWIN ミドルウェアが要求を完全に処理した場合の終端。 |
| [応答キャッシュ](xref:performance/caching/middleware) | 応答のキャッシュのサポートを提供します。 | キャッシュが必要なコンポーネントの前。 `UseResponseCaching` の前に `UseCORS` を追加する必要があります。|
| [応答圧縮](xref:performance/response-compression) | 応答の圧縮のサポートを提供します。 | 圧縮が必要なコンポーネントの前。 |
| [要求のローカライズ](xref:fundamentals/localization) | ローカライズのサポートを提供します。 | ローカリゼーションが重要なコンポーネントの前。 |
| [エンドポイント ルーティング](xref:fundamentals/routing) | 要求のルートを定義および制約します。 | 一致するルートの終端。 |
| [SPA](xref:Microsoft.AspNetCore.Builder.SpaApplicationBuilderExtensions.UseSpa%2A) | シングルページ アプリケーション (SPA) の既定のページを返し、ミドルウェア チェーン内のこの時点以降のすべての要求を処理します。 | チェーンの終わりで、静的ファイルや MVC アクションなどにサービスを提供する他のミドルウェアが優先されるようにするためです。|
| [セッション](xref:fundamentals/app-state) | ユーザー セッションの管理のサポートを提供します。 | セッションが必要なコンポーネントの前。 | 
| [静的ファイル](xref:fundamentals/static-files) | 静的ファイルとディレクトリ参照に対応するサポートを提供します。 | 要求がファイルと一致した場合の終端。 |
| [URL 書き換え](xref:fundamentals/url-rewriting) | URL の書き換えと要求のリダイレクトのサポートを提供します。 | URL を使うコンポーネントの前。 |
| [WebSocket](xref:fundamentals/websockets) | WebSocket プロトコルを有効にします。 | WebSocket 要求を受け入れる必要があるコンポーネントの前。 |

## <a name="additional-resources"></a>その他の技術情報

* [有効期間と登録のオプション](xref:fundamentals/dependency-injection#lifetime-and-registration-options)には、*スコープ*、*一時*、*シングルトン* 有効期間サービスを含むミドルウェアの完全なサンプルが含まれています。
* <xref:fundamentals/middleware/write>
* <xref:test/middleware>
* <xref:migration/http-modules>
* <xref:fundamentals/startup>
* <xref:fundamentals/request-features>
* <xref:fundamentals/middleware/extensibility>
* <xref:fundamentals/middleware/extensibility-third-party-container>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)、[Steve Smith](https://ardalis.com/)

ミドルウェアとは、要求と応答を処理するために、アプリのパイプラインに組み込まれたソフトウェアです。 各コンポーネントで次の処理を実行します。

* 要求をパイプライン内の次のコンポーネントに渡すかどうかを選択します。
* パイプラインの次のコンポーネントの前または後に作業を実行できます。

要求デリゲートは、要求パイプラインの構築に使用されます。 要求デリゲートは、各 HTTP 要求を処理します。

要求デリゲートは、<xref:Microsoft.AspNetCore.Builder.RunExtensions.Run%2A>、<xref:Microsoft.AspNetCore.Builder.MapExtensions.Map%2A>、<xref:Microsoft.AspNetCore.Builder.UseExtensions.Use%2A> の各拡張メソッドを使って構成されます。 個々の要求デリゲートは、匿名メソッドとしてインラインで指定するか (インライン ミドルウェアと呼ばれます)、または再利用可能なクラスで定義することができます。 これらの再利用可能なクラスとインラインの匿名メソッドは、"*ミドルウェア*" です。"*ミドルウェア コンポーネント*" とも呼ばれます。 要求パイプライン内の各ミドルウェア コンポーネントは、パイプラインの次のコンポーネントを呼び出すか、パイプラインをショートさせます。 ショートサーキットしたミドルウェアは "*ターミナル ミドルウェア*" と呼ばれます。これによってさらなるミドルウェアによる要求の処理が回避されるためです。

<xref:migration/http-modules> では、ASP.NET Core と ASP.NET 4.x の要求パイプラインの違いについて説明し、その他のミドルウェア サンプルが提供されています。

## <a name="create-a-middleware-pipeline-with-iapplicationbuilder"></a>IApplicationBuilder を使用したミドルウェア パイプラインの作成

ASP.NET Core 要求パイプラインは、順番に呼び出される一連の要求デリゲートで構成されています。 次の図は、その概念を示しています。 実行のスレッドは黒い矢印をたどります。

![要求の到着、3 つのミドルウェアによる処理、アプリからの応答の送信を示す要求処理パターン。 各ミドルウェアは、そのロジックを実行し、next() ステートメントで要求を次のミドルウェアに渡します。 3 番目のミドルウェアが要求を処理した後、要求は前の 2 つのミドルウェアを逆の順序で通過して、それぞれの next() ステートメントの後の追加処理が行われた後、クライアントへの応答としてアプリを終了します。](index/_static/request-delegate-pipeline.png)

各デリゲートは、次のデリゲートの前と後に操作を実行できます。 例外処理デリゲートは、パイプラインの後のステージで発生する例外をキャッチできるように、パイプラインの早い段階で呼び出される必要があります。

考えられる最も簡単な ASP.NET Core アプリは、1 つの要求デリゲートを設定してすべての要求を処理するものです。 この場合、実際の要求パイプラインは含まれません。 代わりに、すべての HTTP 要求に対応して単一の匿名関数が呼び出されます。

[!code-csharp[](index/snapshot/Middleware/Startup.cs)]

最初の <xref:Microsoft.AspNetCore.Builder.RunExtensions.Run%2A> デリゲートが、パイプラインを終了します。

複数の要求デリゲートを <xref:Microsoft.AspNetCore.Builder.UseExtensions.Use%2A> と一緒にチェーンします。 `next` パラメーターは、パイプラインの次のデリゲートを表します *next* パラメーターを "*呼び出さない*" ことで、パイプラインをショートさせることができます。 次の例で示すように、通常は、次のデリゲートの前と後の両方でアクションを実行できます。

[!code-csharp[](index/snapshot/Chain/Startup.cs)]

デリゲートが次のデリゲートに要求を渡さない場合、これは "*要求パイプラインのショートサーキット*" と呼ばれます。 不要な処理を回避するために、ショートさせることが望ましい場合がよくあります。 たとえば、[静的ファイル ミドルウェア](xref:fundamentals/static-files)は、静的ファイルの要求を処理して残りのパイプラインをショートサーキットすることにより、"*ターミナル ミドルウェア*" として動作させることができます。 後続の処理を終了させるミドルウェアの前にパイプラインに追加されたミドルウェアでは、その `next.Invoke` ステートメントの後も引き続きコードが処理されます。 ただし、既に送信された応答に対する書き込みの試行については、次の警告を参照してください。

> [!WARNING]
> 応答がクライアントに送信された後に、`next.Invoke` を呼び出さないでください。 応答が開始した後で <xref:Microsoft.AspNetCore.Http.HttpResponse> を変更すると、例外がスローされます。 たとえば、ヘッダーや状態コードの設定などの変更があると、例外がスローされます。 `next` を呼び出した後で応答本文に書き込むと、次のようになります。
>
> * プロトコル違反が発生する可能性があります。 たとえば、示されている `Content-Length` より多くを書き込んだ場合。
> * 本文の形式が破損する可能性があります。 たとえば、CSS ファイルに HTML フッターを書き込んだ場合。
>
> <xref:Microsoft.AspNetCore.Http.HttpResponse.HasStarted%2A> は、ヘッダーが送信されたかどうかや本文が書き込まれたかどうかを示すために役立つヒントです。

<a name="order"></a>

## <a name="middleware-order"></a>ミドルウェアの順序

`Startup.Configure` メソッドでミドルウェア コンポーネントを追加する順序は、要求でミドルウェア コンポーネントが呼び出される順序および応答での逆の順序を定義します。 この順序は、セキュリティ、パフォーマンス、および機能にとって **重要** です。

次の `Startup.Configure` メソッドでは、セキュリティ関連のミドルウェア コンポーネントが推奨される順序で追加されています。

[!code-csharp[](index/snapshot/Startup22.cs?name=snippet)]

上のコードでは以下の操作が行われます。

* [個人のユーザー アカウント](xref:security/authentication/identity)を使用して新しい Web アプリを作成するときに追加されないミドルウェアは、コメント アウトされています。
* すべてのミドルウェアを厳密にこの順序で追加する必要があるわけではありませんが、多くはそうする必要があります。 たとえば、`UseCors` と `UseAuthentication` は、示されている順序で追加する必要があります。

次の `Startup.Configure` メソッドにより、共通アプリ シナリオのためのミドルウェア コンポーネントが追加されます。

1. 例外/エラー処理
   * 開発環境でアプリを実行する場合:
     * 開発者例外ページ ミドルウェア (<xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage%2A>) によりアプリの実行時エラーが報告されます。
     * データベース エラー ページ ミドルウェア (`Microsoft.AspNetCore.Builder.DatabaseErrorPageExtensions.UseDatabaseErrorPage`) によりデータベースの実行時エラーが報告されます。
   * 運用環境でアプリを実行する場合:
     * 例外ハンドラー ミドルウェア (<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A>) によって、後続のミドルウェアによってスローされた例外がキャッチされます。
     * HTTP Strict Transport Security プロトコル (HSTS) ミドルウェア (<xref:Microsoft.AspNetCore.Builder.HstsBuilderExtensions.UseHsts%2A>) により `Strict-Transport-Security` ヘッダーが追加されます。
1. HTTPS リダイレクト ミドルウェア (<xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection%2A>) により、HTTP 要求が HTTPS にリダイレクトされます。
1. 静的ファイル ミドルウェア (<xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A>) によって静的ファイルが返され、さらなる要求の処理がスキップされます。
1. Cookie ポリシー ミドルウェア (<xref:Microsoft.AspNetCore.Builder.CookiePolicyAppBuilderExtensions.UseCookiePolicy%2A>) により、アプリを EU の一般データ保護規制 (GDPR) に準拠させます。
1. 認証ミドルウェア (<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A>) により、ユーザーがセキュリティで保護されたリソースにアクセスする前に、ユーザーの認証が試行されます。
1. セッション ミドルウェア (<xref:Microsoft.AspNetCore.Builder.SessionMiddlewareExtensions.UseSession%2A>) により、セッション状態が確立され保持されます。 アプリでセッション状態が使用されている場合は、Cookie ポリシー ミドルウェアの後、MVC ミドルウェアの前に、セッション ミドルウェアを呼び出します。
1. MVC (<xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc%2A>) を使い、要求パイプラインに MVC を追加します。

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
        app.UseDatabaseErrorPage();
    }
    else
    {
        app.UseExceptionHandler("/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseCookiePolicy();
    app.UseAuthentication();
    app.UseSession();
    app.UseMvc();
}
```

前のコード例では、各ミドルウェアの拡張メソッドが <xref:Microsoft.AspNetCore.Builder?displayProperty=fullName> 名前空間を通じて <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder> で公開されています。

パイプラインに追加された最初のミドルウェア コンポーネントは <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> です。 そのため、例外ハンドラー ミドルウェアでは、以降の呼び出しで発生するすべての例外がキャッチされます。

静的ファイル ミドルウェアはパイプラインの早い段階で呼び出されるので、要求を処理し、残りのコンポーネントを通過せずにショートさせることができます。 静的ファイル ミドルウェアでは、承認チェックは提供 **されません**。 *wwwroot* の下にあるものも含め、この静的ファイル ミドルウェアによって提供されるすべてのファイルは、一般に公開されます。 静的ファイルを保護する方法については、「<xref:fundamentals/static-files>」を参照してください。

要求が静的ファイル ミドルウェアによって処理されない場合、要求は認証を実行する認証ミドルウェア (<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A>) に渡されます。 認証は、認証されない要求をショートさせません。 認証ミドルウェアは要求を認証しますが、承認 (および却下) は、MVC が特定の Razor ページまたは MVC コントローラーとアクションを選んだ後でのみ行われます。

次の例は、静的ファイルの要求が応答圧縮ミドルウェアの前に静的ファイル ミドルウェアによって処理される、ミドルウェアの順序を示します。 静的ファイルは、このミドルウェアの順序では圧縮されません。 <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvcWithDefaultRoute%2A> からの MVC 応答を圧縮することができます。

```csharp
public void Configure(IApplicationBuilder app)
{
    // Static files aren't compressed by Static File Middleware.
    app.UseStaticFiles();

    app.UseResponseCompression();

    app.UseMvcWithDefaultRoute();
}
```

## <a name="use-run-and-map"></a>Use、Run、および Map

HTTP パイプラインを構成するには、<xref:Microsoft.AspNetCore.Builder.UseExtensions.Use%2A>、<xref:Microsoft.AspNetCore.Builder.RunExtensions.Run%2A>、<xref:Microsoft.AspNetCore.Builder.MapExtensions.Map%2A> を使います。 `Use` メソッドは、パイプラインをショートさせることができます (つまり `next` 要求デリゲートを呼び出さない場合)。 `Run` は規則であり、一部のミドルウェア コンポーネントは、パイプラインの最後に実行される `Run[Middleware]` メソッドを公開することがあります。

<xref:Microsoft.AspNetCore.Builder.MapExtensions.Map%2A> 拡張メソッドは、パイプラインを分岐する規則として使われます。 `Map` は、指定された要求パスの一致に基づいて、要求パイプラインを分岐します。 要求パスが指定されたパスで開始する場合、分岐が実行されます。

[!code-csharp[](index/snapshot/Chain/StartupMap.cs)]

次の表は、前のコードを使用した `http://localhost:1234` からの要求および応答を示しています。

| 要求             | 応答                     |
| ------------------- | ---------------------------- |
| localhost:1234      | Hello from non-Map delegate. |
| localhost:1234/map1 | Map Test 1                   |
| localhost:1234/map2 | Map Test 2                   |
| localhost:1234/map3 | Hello from non-Map delegate. |

`Map` を使用すると、一致したパス セグメントが `HttpRequest.Path` から削除され、要求ごとに `HttpRequest.PathBase` に追加されます。

<xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen%2A> は、指定された述語の結果に基づいて、要求パイプラインを分岐します。 `Func<HttpContext, bool>` という型の任意の述語を使って、要求をパイプラインの新しい分岐にマップできます。 次の例では、クエリ文字列変数 `branch` の存在を検出するために術後が使用されます。

[!code-csharp[](index/snapshot/Chain/StartupMapWhen.cs)]

次の表は、前のコードを使用した `http://localhost:1234` からの要求および応答を示しています。

| 要求                       | 応答                     |
| ----------------------------- | ---------------------------- |
| localhost:1234                | Hello from non-Map delegate. |
| localhost:1234/?branch=main | Branch used = main         |

`Map` は入れ子をサポートします。次にその例を示します。

```csharp
app.Map("/level1", level1App => {
    level1App.Map("/level2a", level2AApp => {
        // "/level1/level2a" processing
    });
    level1App.Map("/level2b", level2BApp => {
        // "/level1/level2b" processing
    });
});
```

`Map` では、次のように一度に複数のセグメントを照合できます。

[!code-csharp[](index/snapshot/Chain/StartupMultiSeg.cs?highlight=13)]

## <a name="built-in-middleware"></a>組み込みミドルウェア

ASP.NET Core には、次のミドルウェア コンポーネントが付属しています。 "*順番*" 列には、要求を処理するパイプライン内のミドルウェアの配置と、ミドルウェアが要求処理を終了する条件に関するメモが記載されています。 ミドルウェアが要求を処理するパイプラインをショートサーキットし、下流のさらなるミドルウェアによる要求の処理を回避する場合、これは "*ターミナル ミドルウェア*" と呼ばれます。 ショートサーキットについて詳しくは、「[IApplicationBuilder を使用したミドルウェア パイプラインの作成](#create-a-middleware-pipeline-with-iapplicationbuilder)」セクションをご覧ください。

| ミドルウェア | 説明 | 順番 |
| ---------- | ----------- | ----- |
| [認証](xref:security/authentication/identity) | 認証のサポートを提供します。 | `HttpContext.User` が必要な場所の前。 OAuth コールバックの終端。 |
| [Cookie ポリシー](xref:security/gdpr) | 個人情報の保存に関してユーザーからの同意を追跡し、`secure` や `SameSite` など、cookie フィールドの最小要件を適用します。 | cookie を発行するミドルウェアの前。 次に例を示します。 認証、セッション、MVC (TempData) |
| [CORS](xref:security/cors) | クロス オリジン リソース共有を構成します。 | CORS を使うコンポーネントの前。 |
| [診断](xref:fundamentals/error-handling) | 開発者の例外ページ、例外処理、状態コード ページ、および新しいアプリの既定の Web ページを提供する複数の独立したミドルウェア。 | エラーを生成するコンポーネントの前。 例外または新しいアプリ用の既定の Web ページの提供の終端。 |
| [転送されるヘッダー](xref:host-and-deploy/proxy-load-balancer) | プロキシされたヘッダーを現在の要求に転送します。 | 更新されたフィールドを使用するコンポーネントの前。 例: スキーム、ホスト、クライアント IP、メソッド。 |
| [正常性チェック](xref:host-and-deploy/health-checks) | ASP.NET Core アプリとその依存関係の正常性を、データベースの可用性などで確認します。 | 要求が正常性チェックのエンドポイントと一致した場合の終端。 |
| [HTTP メソッドのオーバーライド](xref:Microsoft.AspNetCore.Builder.HttpMethodOverrideExtensions) | メソッドをオーバーライドする受信 POST 要求を許可します。 | 更新されたメソッドを使うコンポーネントの前。 |
| [HTTPS リダイレクト](xref:security/enforcing-ssl#require-https) | すべての HTTP 要求を HTTPS にリダイレクトします。 | URL を使うコンポーネントの前。 |
| [HTTP Strict Transport Security (HSTS)](xref:security/enforcing-ssl#http-strict-transport-security-protocol-hsts) | 特殊な応答ヘッダーを追加するセキュリティ拡張機能のミドルウェア。 | 応答が送信される前と要求を変更するコンポーネントの後。 次に例を示します。 転送されるヘッダー、URL リライト。 |
| [MVC](xref:mvc/overview) | MVC または Razor Pages で要求を処理します。 | 要求がルートと一致した場合の終端。 |
| [OWIN](xref:fundamentals/owin) | OWIN ベースのアプリ、サーバー、およびミドルウェアと相互運用します。 | OWIN ミドルウェアが要求を完全に処理した場合の終端。 |
| [応答キャッシュ](xref:performance/caching/middleware) | 応答のキャッシュのサポートを提供します。 | キャッシュが必要なコンポーネントの前。 |
| [応答圧縮](xref:performance/response-compression) | 応答の圧縮のサポートを提供します。 | 圧縮が必要なコンポーネントの前。 |
| [要求のローカライズ](xref:fundamentals/localization) | ローカライズのサポートを提供します。 | ローカリゼーションが重要なコンポーネントの前。 |
| [エンドポイント ルーティング](xref:fundamentals/routing) | 要求のルートを定義および制約します。 | 一致するルートの終端。 |
| [セッション](xref:fundamentals/app-state) | ユーザー セッションの管理のサポートを提供します。 | セッションが必要なコンポーネントの前。 |
| [静的ファイル](xref:fundamentals/static-files) | 静的ファイルとディレクトリ参照に対応するサポートを提供します。 | 要求がファイルと一致した場合の終端。 |
| [URL 書き換え](xref:fundamentals/url-rewriting) | URL の書き換えと要求のリダイレクトのサポートを提供します。 | URL を使うコンポーネントの前。 |
| [WebSocket](xref:fundamentals/websockets) | WebSocket プロトコルを有効にします。 | WebSocket 要求を受け入れる必要があるコンポーネントの前。 |

## <a name="additional-resources"></a>その他の技術情報

* <xref:fundamentals/middleware/write>
* <xref:test/middleware>
* <xref:migration/http-modules>
* <xref:fundamentals/startup>
* <xref:fundamentals/request-features>
* <xref:fundamentals/middleware/extensibility>
* <xref:fundamentals/middleware/extensibility-third-party-container>

::: moniker-end
