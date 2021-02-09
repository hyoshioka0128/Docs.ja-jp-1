---
title: ASP.NET Core で IHttpClientFactory を使用して HTTP 要求を行う
author: stevejgordon
description: IHttpClientFactory インターフェイスを使用して、ASP.NET Core の論理 HttpClient インスタンスを管理する方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: mvc
ms.date: 1/21/2021
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
uid: fundamentals/http-requests
ms.openlocfilehash: 1cf3029452f87a396847f969f0f3136a75874752
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057331"
---
# <a name="make-http-requests-using-ihttpclientfactory-in-aspnet-core"></a>ASP.NET Core で IHttpClientFactory を使用して HTTP 要求を行う

::: moniker range=">= aspnetcore-3.0"

作成者: [Kirk Larkin](https://github.com/serpent5)、[Steve Gordon](https://github.com/stevejgordon)、[Glenn Condron](https://github.com/glennc)、[Ryan Nowak](https://github.com/rynowak)。

アプリ内で <xref:System.Net.Http.HttpClient> インスタンスを構成して作成するために、<xref:System.Net.Http.IHttpClientFactory> を登録して使用できます。 `IHttpClientFactory` には次のような利点があります。

* 論理 `HttpClient` インスタンスの名前付けと構成を一元化します。 たとえば、[GitHub](https://github.com/) にアクセスするために、*github* という名前のクライアントを登録して構成できます。 一般的なアクセスのために、既定のクライアントを登録できます。
* `HttpClient` でのハンドラーのデリゲートにより、送信ミドルウェアの概念が体系化されます。 `HttpClient` でのハンドラーのデリゲートを利用するために、Polly ベースのミドルウェアに対する拡張機能が提供されます。
* 基になっている `HttpClientMessageHandler` インスタンスのプールと有効期間が管理されます。 自動管理により、`HttpClient` の有効期間を手動で管理するときの一般的な DNS (ドメイン ネーム システム) の問題が発生しなくなります。
* ファクトリによって作成されたクライアントから送信されるすべての要求に対し、(`ILogger` によって) 構成可能なログ エクスペリエンスを追加します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/http-requests/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

このトピックのバージョンのサンプル コードでは、HTTP 応答で返された JSON コンテンツを、<xref:System.Text.Json> を使用して逆シリアル化します。 `Json.NET` および `ReadAsAsync<T>` を使用するサンプルについては、バージョン セレクターを使用して、このトピックの 2.x バージョンを選択してください。

## <a name="consumption-patterns"></a>利用パターン

アプリで `IHttpClientFactory` を使用するには複数の方法があります。

* [基本的な使用方法](#basic-usage)
* [名前付きクライアント](#named-clients)
* [型指定されたクライアント](#typed-clients)
* [生成されたクライアント](#generated-clients)

どの方法が最善かは、アプリの要件によって異なります。

### <a name="basic-usage"></a>基本的な使用方法

`IHttpClientFactory` は、`AddHttpClient` を呼び出すことによって登録できます。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet1&highlight=13)]

`IHttpClientFactory` は、[依存関係の挿入 (DI)](xref:fundamentals/dependency-injection) を使用して要求できます。 次のコードでは、`IHttpClientFactory` を使用して `HttpClient` インスタンスを作成しています。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Pages/BasicUsage.cshtml.cs?name=snippet1&highlight=9-12,21)]

前の例のように `IHttpClientFactory` を使用するのは、既存のアプリをリファクタリングするのに適した方法です。 `HttpClient` の使用方法に対する影響はありません。 既存のアプリで `HttpClient` のインスタンスが作成されている場所を、<xref:System.Net.Http.IHttpClientFactory.CreateClient*> の呼び出しに置き換えます。

### <a name="named-clients"></a>名前付きクライアント

名前付きクライアントは、次の場合に適しています。

* アプリで、多くの異なる `HttpClient` を使用する必要がある。
* 多くの `HttpClient` の構成が異なる。

名前付き `HttpClient` の構成は、`Startup.ConfigureServices` での登録時に指定できます。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet2)]

上記のコードでは、クライアントに次のものが構成されます。

* ベース アドレス `https://api.github.com/`。
* GitHub API を使用するために必要な 2 つのヘッダー。

#### <a name="createclient"></a>CreateClient

<xref:System.Net.Http.IHttpClientFactory.CreateClient*> が呼び出されるたびに、次のことが行われます。

* `HttpClient` の新しいインスタンスが作成されます。
* 構成アクションが呼び出されます。

名前付きクライアントを作成するには、その名前を `CreateClient` に渡します。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Pages/NamedClient.cshtml.cs?name=snippet1&highlight=21)]

上記のコードでは、要求でホスト名を指定する必要はありません。 クライアントに構成されているベース アドレスが使用されるため、コードではパスを渡すだけで済みます。

### <a name="typed-clients"></a>型指定されたクライアント

型指定されたクライアント:

* キーとして文字列を使用する必要なしに、名前付きクライアントと同じ機能を提供します。
* クライアントを使用するときに、IntelliSense とコンパイラのヘルプが提供されます。
* 特定の `HttpClient` を構成してそれと対話する 1 つの場所を提供します。 たとえば、単一の型指定されたクライアントは、次のために使用される場合があります。
  * 単一のバックエンド エンドポイント用。
  * エンドポイントを処理するすべてのロジックをカプセル化するため。
* DI に対応しており、アプリ内の必要な場所に挿入できます。

型指定されたクライアントは、そのコンストラクターで `HttpClient` パラメーターを受け取ります。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/GitHub/GitHubService.cs?name=snippet1&highlight=5)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

上のコードでは以下の操作が行われます。

* 構成が型指定されたクライアントに移動されます。
* `HttpClient` オブジェクトは、パブリック プロパティとして公開されます。

`HttpClient` の機能を公開する API 固有のメソッドを作成できます。 たとえば、`GetAspNetDocsIssues` メソッドでは、未解決の問題を取得するためのコードがカプセル化されています。

次のコードでは、`Startup.ConfigureServices` 内で <xref:Microsoft.Extensions.DependencyInjection.HttpClientFactoryServiceCollectionExtensions.AddHttpClient*> を呼び出して、型指定されたクライアント クラスを登録しています。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet3)]

型指定されたクライアントは、DI で一時的として登録されます。 上記のコードで、`AddHttpClient` は `GitHubService` を一時的なサービスとして登録します。 この登録では、ファクトリ メソッドを使用して次のことを行います。

1. `HttpClient` のインスタンスを作成します。
1. `GitHubService` のインスタンスを作成し、`HttpClient` のインスタンスをそのコンストラクターに渡します。

型指定されたクライアントは、直接挿入して使用できます。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Pages/TypedClient.cshtml.cs?name=snippet1&highlight=11-14,20)]

型指定されたクライアントのコンストラクターではなく、`Startup.ConfigureServices` での登録時に型指定されたクライアントの構成を指定できます。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet4)]

`HttpClient` は、型指定されたクライアント内にカプセル化できます。 プロパティとして公開するのではなく、`HttpClient` インスタンスを内部的に呼び出すパブリック メソッドを定義します。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/GitHub/RepoService.cs?name=snippet1&highlight=4)]

上記のコードでは、`HttpClient` はプライベート フィールドに格納されます。 `HttpClient` へのアクセスは、パブリック `GetRepos` メソッドによって行われます。

### <a name="generated-clients"></a>生成されたクライアント

`IHttpClientFactory` は、[Refit](https://github.com/paulcbetts/refit) などのサードパーティ製ライブラリと組み合わせて使用できます。 Refit は、.NET 用の REST ライブラリです。 REST API をライブ インターフェイスに変換します。 インターフェイスの実装は `RestService` によって動的に生成され、`HttpClient` を使用して外部 HTTP の呼び出しを行います。

インターフェイスと応答は、外部の API とその応答を表すように定義されます。

```csharp
public interface IHelloClient
{
    [Get("/helloworld")]
    Task<Reply> GetMessageAsync();
}

public class Reply
{
    public string Message { get; set; }
}
```

型指定されたクライアントを追加し、Refit を使用して実装を生成できます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddHttpClient("hello", c =>
    {
        c.BaseAddress = new Uri("http://localhost:5000");
    })
    .AddTypedClient(c => Refit.RestService.For<IHelloClient>(c));

    services.AddControllers();
}
```

定義されたインターフェイスは、DI および Refit によって提供される実装で必要に応じて使用できます。

```csharp
[ApiController]
public class ValuesController : ControllerBase
{
    private readonly IHelloClient _client;

    public ValuesController(IHelloClient client)
    {
        _client = client;
    }

    [HttpGet("/")]
    public async Task<ActionResult<Reply>> Index()
    {
        return await _client.GetMessageAsync();
    }
}
```

## <a name="make-post-put-and-delete-requests"></a>POST、PUT、DELETE の要求を行う

前の例では、すべての HTTP 要求で GET HTTP 動詞が使用されています。 `HttpClient` では、次のような他の HTTP 動詞もサポートされています。

* POST
* PUT
* Del
* PATCH

サポートされている HTTP 動詞の一覧については、「<xref:System.Net.Http.HttpMethod>」を参照してください。

次の例は、HTTP POST 要求の方法を示しています。

[!code-csharp[](http-requests/samples/3.x/HttpRequestsSample/Models/TodoClient.cs?name=snippet_POST)]

上記のコードの `CreateItemAsync` メソッドは:

* `System.Text.Json` を使用して `TodoItem` パラメーターを JSON にシリアル化します。 これは、<xref:System.Text.Json.JsonSerializerOptions> のインスタンスを使用して、シリアル化プロセスを構成します。
* HTTP 要求の本文で送信するためにシリアル化された JSON をパッケージ化する <xref:System.Net.Http.StringContent> のインスタンスを作成します。
* <xref:System.Net.Http.HttpClient.PostAsync%2A> を呼び出して、指定した URL に JSON の内容を送信します。 これは [HttpClient.BaseAddress](xref:System.Net.Http.HttpClient.BaseAddress) に追加される相対 URL です。
* 応答状態コードが成功を示していない場合に、<xref:System.Net.Http.HttpResponseMessage.EnsureSuccessStatusCode%2A> を呼び出して例外をスローします。

`HttpClient` は、他の種類のコンテンツもサポートしています。 たとえば、<xref:System.Net.Http.MultipartContent> と<xref:System.Net.Http.StreamContent> です。 サポートされているコンテンツの一覧については、「<xref:System.Net.Http.HttpContent>」を参照してください。

HTTP PUT 要求の例を次に示します。

[!code-csharp[](http-requests/samples/3.x/HttpRequestsSample/Models/TodoClient.cs?name=snippet_PUT)]

上記のコードは、POST の例によく似ています。 `SaveItemAsync` メソッドは `PostAsync` ではなく <xref:System.Net.Http.HttpClient.PutAsync%2A> を呼び出します。

HTTP DELETE 要求の例を次に示します。

[!code-csharp[](http-requests/samples/3.x/HttpRequestsSample/Models/TodoClient.cs?name=snippet_DELETE)]

上記のコードの `DeleteItemAsync` メソッドは <xref:System.Net.Http.HttpClient.DeleteAsync%2A> を呼び出します。 HTTP DELETE 要求には通常、本文が含まれていないため、`DeleteAsync` メソッドは `HttpContent` のインスタンスを受け入れるオーバーロードを提供しません。

`HttpClient` でのさまざまな HTTP 動詞の使用の詳細については、「<xref:System.Net.Http.HttpClient>」を参照してください。

## <a name="outgoing-request-middleware"></a>送信要求ミドルウェア

`HttpClient` は、送信 HTTP 要求用にリンクできるハンドラーのデリゲートの概念を備えています。 `IHttpClientFactory`:

  * 各名前付きクライアントに適用するハンドラーの定義が簡単になります。
  * 送信要求ミドルウェア パイプラインを構築するための複数のハンドラーの登録とチェーン化がサポートされています。 これらの各ハンドラーでは、送信要求の前と後に処理を実行できます。 このパターンは次のようなものです。
  
    * ASP.NET Core での受信ミドルウェア パイプラインに似ています。
    * 次のような HTTP 要求に関する横断的な問題を管理するためのメカニズムが提供されます。
      * キャッシュ
      * エラー処理
      * シリアル化
      * ログ

デリゲート ハンドラーを作成するには、次のようにします。

* <xref:System.Net.Http.DelegatingHandler> から派生します。
* <xref:System.Net.Http.DelegatingHandler.SendAsync*> をオーバーライドします。 パイプライン内の次のハンドラーに要求を渡す前に、コードを実行します。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Handlers/ValidateHeaderHandler.cs?name=snippet1)]

上記のコードでは、`X-API-KEY` ヘッダーが要求に含まれているかどうかが確認されます。 `X-API-KEY` がない場合は、<xref:System.Net.HttpStatusCode.BadRequest> が返されます。

<xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.AddHttpMessageHandler*?displayProperty=fullName> を使用して `HttpClient` の構成に複数のハンドラーを追加できます。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup2.cs?name=snippet1)]

上記のコードでは、`ValidateHeaderHandler` が DI で登録されます。 登録が済むと、<xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.AddHttpMessageHandler*> を呼び出してハンドラーの型を渡すことができます。

実行する順序で、複数のハンドラーを登録することができます。 最後の `HttpClientHandler` が要求を実行するまで、各ハンドラーは次のハンドラーをラップします。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet6)]

### <a name="use-di-in-outgoing-request-middleware"></a>送信要求ミドルウェアで DI を使用する

`IHttpClientFactory` によって新しいデリゲート ハンドラーが作成される際、ハンドラーのコンストラクター パラメーターを満たすために DI が使用されます。 `IHttpClientFactory` により、ハンドラーごとに **個別の** DI スコープが作成されます。これにより、ハンドラーによって "*スコープ付き*" サービスが使用されるときに、予期しない動作が発生する可能性があります。

たとえば、識別子 `OperationId` を持つ操作としてタスクを表す、次のインターフェイスとその実装について考えてみます。

[!code-csharp[](http-requests/samples/3.x/HttpRequestsSample/Models/OperationScoped.cs?name=snippet_Types)]

その名前が示すように、`IOperationScoped` は "*スコープ付き*" 有効期間を使用して DI に登録されます。

[!code-csharp[](http-requests/samples/3.x/HttpRequestsSample/Startup.cs?name=snippet_IOperationScoped&highlight=18,26)]

次のデリゲート ハンドラーでは、`IOperationScoped` を使用して、送信要求用の `X-OPERATION-ID` ヘッダーを設定します。

[!code-csharp[](http-requests/samples/3.x/HttpRequestsSample/Handlers/OperationHandler.cs?name=snippet_Class&highlight=13)]

[`HttpRequestsSample` ダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/http-requests/samples/3.x/HttpRequestsSample)] で、`/Operation` に移動してページを更新します。 要求スコープの値は要求ごとに変更されますが、ハンドラーのスコープの値は 5 秒ごとに変更されるだけです。

ハンドラーは、任意のスコープのサービスに依存することができます。 ハンドラーが依存するサービスは、ハンドラーが破棄されるときに破棄されます。

次の方法のいずれかを使って、要求ごとの状態をメッセージ ハンドラーと共有します。

* [HttpRequestMessage.Properties](xref:System.Net.Http.HttpRequestMessage.Properties) を使用して、ハンドラーにデータを渡します。
* <xref:Microsoft.AspNetCore.Http.IHttpContextAccessor> を使って現在の要求にアクセスします。
* データを渡すカスタムの <xref:System.Threading.AsyncLocal`1> ストレージ オブジェクトを作成します。

## <a name="use-polly-based-handlers"></a>Polly ベースのハンドラーを使用する

`IHttpClientFactory` は、サードパーティのライブラリ [Polly](https://github.com/App-vNext/Polly) と統合します。 Polly は、.NET 用の包括的な回復力および一時的エラー処理ライブラリです。 開発者は、自然でスレッド セーフな方法を使用して、Retry、Circuit Breaker、Timeout、Bulkhead Isolation、Fallback などのポリシーを表現できます。

構成されている `HttpClient` インスタンスで Polly ポリシーを使用できるようにするための、拡張メソッドが提供されています。 Polly の拡張機能では、クライアントへの Polly ベースのハンドラーの追加がサポートされています。 Polly では、[Microsoft.Extensions.Http.Polly](https://www.nuget.org/packages/Microsoft.Extensions.Http.Polly/) NuGet パッケージが必要です。

### <a name="handle-transient-faults"></a>一時的な障害を処理する

通常、外部 HTTP を呼び出すときに発生する障害は、一時的なものです。 <xref:Microsoft.Extensions.DependencyInjection.PollyHttpClientBuilderExtensions.AddTransientHttpErrorPolicy*> では、一時的なエラーを処理するためのポリシーを定義できます。 `AddTransientHttpErrorPolicy` で構成されたポリシーでは、次の応答が処理されます。

* <xref:System.Net.Http.HttpRequestException>
* HTTP 5xx
* HTTP 408

`AddTransientHttpErrorPolicy` では、可能性のある一時的障害を表すエラーを処理するために構成された `PolicyBuilder` オブジェクトへのアクセスが提供されます。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup3.cs?name=snippet1)]

上記のコードでは、`WaitAndRetryAsync` ポリシーが定義されています。 失敗した要求は最大 3 回再試行され、再試行の間には 600 ミリ秒の遅延が設けられます。

### <a name="dynamically-select-policies"></a>ポリシーを動的に選択する

Polly ベースのハンドラーを追加するための拡張メソッドが提供されています (<xref:Microsoft.Extensions.DependencyInjection.PollyHttpClientBuilderExtensions.AddPolicyHandler*> など)。 次の `AddPolicyHandler` のオーバーロードでは、要求が検査されて、適用するポリシーが決定されます。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet8)]

上記のコードでは、送信要求が HTTP GET の場合は、10 秒のタイムアウトが適用されます。 他の HTTP メソッドの場合は、30 秒のタイムアウトが使用されます。

### <a name="add-multiple-polly-handlers"></a>複数の Polly ハンドラーを追加する

Polly のポリシーは入れ子にするのが一般的です。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet9)]

前の例の場合:

* 2 つのハンドラーが追加されます。
* 1 つ目のハンドラーでは、<xref:Microsoft.Extensions.DependencyInjection.PollyHttpClientBuilderExtensions.AddTransientHttpErrorPolicy*> を使用して再試行ポリシーが追加されます。 失敗した要求は 3 回まで再試行されます。
* 2 つ目の `AddTransientHttpErrorPolicy` の呼び出しでは、サーキット ブレーカー ポリシーが追加されます。 試行が連続して 5 回失敗した場合、それ以上の外部要求は 30 秒間ブロックされます。 サーキット ブレーカー ポリシーはステートフルです。 このクライアントからのすべての呼び出しは、同じサーキット状態を共有します。

### <a name="add-policies-from-the-polly-registry"></a>Polly レジストリからポリシーを追加する

定期的に使用されるポリシーを管理するには、ポリシーを 1 回定義した後、`PolicyRegistry` でポリシーを登録します。

次のコードの内容は以下のとおりです。

* "regular" ポリシーと "long" ポリシーが追加されます。
* <xref:Microsoft.Extensions.DependencyInjection.PollyHttpClientBuilderExtensions.AddPolicyHandlerFromRegistry*> では、レジストリから "regular" ポリシーと "long" ポリシーが追加されます。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup4.cs?name=snippet1)]

`IHttpClientFactory` と Polly の統合の詳細については、[Polly に関する wiki](https://github.com/App-vNext/Polly/wiki/Polly-and-HttpClientFactory) を参照してください。

## <a name="httpclient-and-lifetime-management"></a>HttpClient と有効期間の管理

`IHttpClientFactory` で `CreateClient` を呼び出すたびに、`HttpClient` の新しいインスタンスが返されます。 <xref:System.Net.Http.HttpMessageHandler> は、名前付きクライアントごとに作成されます。 ファクトリによって `HttpMessageHandler` インスタンスの有効期間が管理されます。

`IHttpClientFactory` は、リソースの消費量を減らすため、ファクトリによって作成された `HttpMessageHandler` のインスタンスをプールします。 新しい `HttpClient` インスタンスを作成するときに、プールの `HttpMessageHandler` インスタンスの有効期間が切れていない場合はそれを再利用する場合があります。

通常、各ハンドラーでは基になる HTTP 接続が独自に管理されるため、ハンドラーはプールすることが望まれます。 必要以上のハンドラーを作成すると、接続に遅延が発生する可能性があります。 また、一部のハンドラーでは接続が無期限に開かれており、DNS (ドメイン ネーム システム) の変更にハンドラーが対応できないことがあります。

ハンドラーの既定の有効期間は 2 分です。 名前付きクライアントごとに、既定値をオーバーライドすることができます。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup5.cs?name=snippet1)]

通常、`HttpClient` インスタンスは、破棄する必要の **ない** .NET オブジェクトとして扱うことができます。 破棄すると送信要求がキャンセルされ、<xref:System.IDisposable.Dispose*> の呼び出し後には指定された `HttpClient` インスタンスを使用できなくなります。 `IHttpClientFactory` は、`HttpClient` インスタンスによって使用されたリソースの追跡と破棄を行います。

`IHttpClientFactory` の登場以前は、1 つの `HttpClient` インスタンスを長い期間使い続けるパターンが一般的に使用されていました。 `IHttpClientFactory` に移行した後は、このパターンは不要になります。

### <a name="alternatives-to-ihttpclientfactory"></a>IHttpClientFactory の代替手段

DI 対応のアプリ内で `IHttpClientFactory` を使用すれば、次のことを回避できます。

* `HttpMessageHandler` インスタンスをプールすることによるリソース枯渇の問題。
* 一定の間隔で `HttpMessageHandler` インスタンスを循環させることによって発生する古くなった DNS の問題。

有効期間の長い <xref:System.Net.Http.SocketsHttpHandler> インスタンスを使用して、上記の問題を解決する別の方法があります。

- アプリの起動時に `SocketsHttpHandler` インスタンスを作成し、アプリの有効期間中、それを使用します。
- DNS の更新時間に基づいて、<xref:System.Net.Http.SocketsHttpHandler.PooledConnectionLifetime> を適切な値に構成します。
- 必要に応じて `new HttpClient(handler, disposeHandler: false)` を使用して `HttpClient` インスタンスを作成します。

上記の方法を使用すると、`IHttpClientFactory` が同様の方法で解決するリソース管理の問題を解決できます。

- `SocketsHttpHandler` を使用すると、`HttpClient` インスタンス間で接続を共有できます。 この共有によってソケットの枯渇が防止されます。
- `SocketsHttpHandler` では、古くなった DNS の問題を回避するために `PooledConnectionLifetime` に従って接続を循環されます。

### <a name="cookies"></a>Cookies

`HttpMessageHandler` インスタンスをプールすると、`CookieContainer` オブジェクトが共有されます。 予期せぬ `CookieContainer` オブジェクト共有があると、多くの場合、コードは不適切なものとなります。 cookie を必要とするアプリの場合は、次のいずれかを検討してください。

 - 自動的な cookie 処理の無効化
 - `IHttpClientFactory` の回避

<xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> を呼び出して、自動的な cookie 処理を無効にします。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet13)]

## <a name="logging"></a>ログの記録

`IHttpClientFactory` によって作成されたクライアントは、すべての要求のログ メッセージを記録します。 既定のログ メッセージを見るには、ログの構成で適切な情報レベルを有効にします。 要求ヘッダーのログなどの追加ログは、トレース レベルでのみ含まれます。

各クライアントに使用されるログのカテゴリには、クライアントの名前が含まれます。 たとえば、*MyNamedClient* という名前のクライアントでは、"System.Net.Http.HttpClient.**MyNamedClient**.LogicalHandler" というカテゴリでメッセージがログに記録されます。 *LogicalHandler* というサフィックスが付いたメッセージが、要求ハンドラーのパイプラインの外部で発生します。 要求では、パイプライン内の他のハンドラーが要求を処理する前に、メッセージがログに記録されます。 応答では、他のパイプライン ハンドラーが応答を受け取った後で、メッセージがログに記録されます。

ログ記録は、要求ハンドラーのパイプラインの内部でも行われます。 *MyNamedClient* の例では、これらのメッセージはログ カテゴリ "System.Net.Http.HttpClient.**MyNamedClient**.ClientHandler" でログに記録されます。 要求では、他のすべてのハンドラーが実行された後、要求が送信される直前に、これが行われます。 応答では、このログ記録には、ハンドラー パイプラインを通じ戻される前の応答の状態が含まれます。

パイプラインの外部と内部でログを有効にすると、他のパイプライン ハンドラーによる変更を検査できます。 これには、要求ヘッダーや応答状態コードへの変更を含めることができます。

ログのカテゴリにクライアントの名前を含めると、特定の名前付きクライアントでログをフィルター処理できます。

## <a name="configure-the-httpmessagehandler"></a>HttpMessageHandler を構成する

クライアントによって使用される内部 `HttpMessageHandler` の構成を制御することが必要な場合があります。

名前付きクライアントまたは型指定されたクライアントを追加すると、`IHttpClientBuilder` が返されます。 <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> 拡張メソッドを使用して、デリゲートを定義することができます。 デリゲートは、そのクライアントによって使用されるプライマリ `HttpMessageHandler` の作成と構成に使用されます。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup6.cs?name=snippet1)]

## <a name="use-ihttpclientfactory-in-a-console-app"></a>コンソール アプリで IHttpClientFactory を使用する

コンソール アプリで、次のパッケージ参照をプロジェクトに追加します。

* [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting)
* [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http)

次に例を示します。

* <xref:System.Net.Http.IHttpClientFactory> は[汎用ホスト](xref:fundamentals/host/generic-host)のサービス コンテナーに登録されます。
* `MyService` により、クライアント ファクトリ インスタンスがサービスから作成され、それが `HttpClient` の作成に使用されます。 `HttpClient` は Web ページを取得する目的で使用されます。
* `Main` により、サービスの `GetPage` メソッドを実行し、Web ページ コンテンツの最初の 500 文字をコンソールに書き込むためのスコープが作成されます。

[!code-csharp[](http-requests/samples/3.x/HttpClientFactoryConsoleSample/Program.cs?highlight=14-15,20,26-27,59-62)]

## <a name="header-propagation-middleware"></a>ヘッダー伝達ミドルウェア

ヘッダー伝達は、受信要求から送信 HTTP クライアント要求に HTTP ヘッダーを伝達するための ASP.NET Core ミドルウェアです。 ヘッダー伝達を使用するには、次を行います。

* [Microsoft.AspNetCore.HeaderPropagation](https://www.nuget.org/packages/Microsoft.AspNetCore.HeaderPropagation) パッケージを参照します。
* `Startup` でミドルウェアと `HttpClient` を構成します。

  [!code-csharp[](http-requests/samples/3.x/Startup.cs?highlight=5-9,21&name=snippet)]

* クライアントで、構成されたヘッダーが送信要求に含まれます。

  ```csharp
  var client = clientFactory.CreateClient("MyForwardingClient");
  var response = client.GetAsync(...);
  ```

## <a name="additional-resources"></a>その他の技術情報

* [HttpClientFactory を使用して回復力の高い HTTP 要求を実装する](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)
* [HttpClientFactory ポリシーと Polly ポリシーで指数バックオフを含む HTTP 呼び出しの再試行を実装する](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)
* [サーキット ブレーカー パターンを実装する](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-circuit-breaker-pattern)
* [.NET で JSON のシリアル化と逆シリアル化を行う方法](/dotnet/standard/serialization/system-text-json-how-to)

::: moniker-end

::: moniker range="= aspnetcore-2.2"

寄稿者: [Glenn Condron](https://github.com/glennc)、[Ryan Nowak](https://github.com/rynowak)、[Steve Gordon](https://github.com/stevejgordon)

アプリ内で <xref:System.Net.Http.HttpClient> インスタンスを構成して作成するために、<xref:System.Net.Http.IHttpClientFactory> を登録して使用できます。 次のような利点があります。

* 論理 `HttpClient` インスタンスの名前付けと構成を一元化します。 たとえば、*github* クライアントを登録して、[GitHub](https://github.com/) にアクセスするように構成できます。 既定のクライアントは、他の目的に登録できます。
* `HttpClient` でのハンドラーのデリゲートにより送信ミドルウェアの概念を体系化し、Polly ベースのミドルウェアでそれを利用するための拡張機能を提供します。
* 基になっている `HttpClientMessageHandler` インスタンスのプールと有効期間を管理し、`HttpClient` の有効期間を手動で管理するときに発生する一般的な DNS の問題を防ぎます。
* ファクトリによって作成されたクライアントから送信されるすべての要求に対し、(`ILogger` によって) 構成可能なログ エクスペリエンスを追加します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/http-requests/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="consumption-patterns"></a>利用パターン

アプリで `IHttpClientFactory` を使用するには複数の方法があります。

* [基本的な使用方法](#basic-usage)
* [名前付きクライアント](#named-clients)
* [型指定されたクライアント](#typed-clients)
* [生成されたクライアント](#generated-clients)

これらの間に厳密な優劣はありません。 どの方法が最善かは、アプリの制約に依存します。

### <a name="basic-usage"></a>基本的な使用方法

`IHttpClientFactory` は、`Startup.ConfigureServices` メソッドの内部で `IServiceCollection` の `AddHttpClient` 拡張メソッドを呼び出すことによって登録できます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet1)]

登録が済むと、コードは、[依存関係の挿入 (DI)](xref:fundamentals/dependency-injection) を使用してサービスを挿入できる任意の場所で、`IHttpClientFactory` を受け取ることができます。 `IHttpClientFactory` を使用して、`HttpClient` インスタンスを作成できます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/BasicUsage.cshtml.cs?name=snippet1&highlight=9-12,21)]

このように `IHttpClientFactory` を使用するのは、既存のアプリをリファクタリングする優れた方法です。 `HttpClient` の使用方法に影響はありません。 現在 `HttpClient` インスタンスが作成されている場所で、それを <xref:System.Net.Http.IHttpClientFactory.CreateClient*> の呼び出しに置き換えます。

### <a name="named-clients"></a>名前付きクライアント

それぞれ構成が異なる多数の `HttpClient` をアプリで個別に使用する必要がある場合は、**名前付きクライアント** を使用することができます。 名前付き `HttpClient` の構成は、`Startup.ConfigureServices` での登録時に指定できます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet2)]

上記のコードでは、*github* という名前を指定して `AddHttpClient` を呼び出しています。 このクライアントには既定の構成がいくつか適用されています。つまり、GitHub API を使用するために必要なベース アドレスと 2 つのヘッダーです。

`CreateClient` を呼び出すたびに、`HttpClient` の新しいインスタンスが作成されて、構成アクションが呼び出されます。

名前付きクライアントを使用するには、文字列パラメーターを `CreateClient` に渡すことができます。 作成するクライアントの名前を指定します。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/NamedClient.cshtml.cs?name=snippet1&highlight=21)]

上記のコードでは、要求でホスト名を指定する必要はありません。 クライアントに構成されているベース アドレスが使用されるため、パスだけを渡すことができます。

### <a name="typed-clients"></a>型指定されたクライアント

型指定されたクライアント:

* キーとして文字列を使用する必要なしに、名前付きクライアントと同じ機能を提供します。
* クライアントを使用するときに、IntelliSense とコンパイラのヘルプが提供されます。
* 特定の `HttpClient` を構成してそれと対話する 1 つの場所を提供します。 たとえば、単一の型指定されたクライアントを単一のバックエンド エンドポイントに対して使用し、そのエンドポイントを操作するすべてのロジックをカプセル化できます。
* DI に対応しており、アプリ内の必要な場所に挿入できます。

型指定されたクライアントは、そのコンストラクターで `HttpClient` パラメーターを受け取ります。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/GitHub/GitHubService.cs?name=snippet1&highlight=5)]

上記のコードでは、構成が型指定されたクライアントに移動されています。 `HttpClient` オブジェクトは、パブリック プロパティとして公開されます。 `HttpClient` 機能を公開する API 固有のメソッドを定義することができます。 `GetAspNetDocsIssues` メソッドは、GitHub リポジトリで最新の未解決の問題をクエリして解析するために必要なコードをカプセル化します。

型指定されたクライアントを登録するには、ジェネリック <xref:Microsoft.Extensions.DependencyInjection.HttpClientFactoryServiceCollectionExtensions.AddHttpClient*> 拡張メソッドを `Startup.ConfigureServices` 内で使用して、型指定されたクライアントのクラスを指定します。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet3)]

型指定されたクライアントは、DI で一時的として登録されます。 型指定されたクライアントは、直接挿入して使用できます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/TypedClient.cshtml.cs?name=snippet1&highlight=11-14,20)]

好みに応じて、型指定されたクライアントのコンストラクターではなく、`Startup.ConfigureServices` での登録時に型指定されたクライアントの構成を指定できます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet4)]

型指定されたクライアントの内部に `HttpClient` を完全にカプセル化することができます。 プロパティとして公開するのではなく、`HttpClient` インスタンスを内部的に呼び出すパブリック メソッドとして提供することができます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/GitHub/RepoService.cs?name=snippet1&highlight=4)]

上記のコードでは、`HttpClient` はプライベート フィールドとして格納されます。 外部呼び出しを行うすべてのアクセスは、`GetRepos` メソッドを通して行われます。

### <a name="generated-clients"></a>生成されたクライアント

`IHttpClientFactory` は、[Refit](https://github.com/paulcbetts/refit) などの他のサードパーティ製ライブラリと組み合わせて使用できます。 Refit は、.NET 用の REST ライブラリです。 REST API をライブ インターフェイスに変換します。 インターフェイスの実装は `RestService` によって動的に生成され、`HttpClient` を使用して外部 HTTP の呼び出しを行います。

インターフェイスと応答は、外部の API とその応答を表すように定義されます。

```csharp
public interface IHelloClient
{
    [Get("/helloworld")]
    Task<Reply> GetMessageAsync();
}

public class Reply
{
    public string Message { get; set; }
}
```

型指定されたクライアントを追加し、Refit を使用して実装を生成できます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddHttpClient("hello", c =>
    {
        c.BaseAddress = new Uri("https://localhost:5001");
    })
    .AddTypedClient(c => Refit.RestService.For<IHelloClient>(c));

    services.AddMvc();
}
```

定義されたインターフェイスは、DI および Refit によって提供される実装で必要に応じて使用できます。

```csharp
[ApiController]
public class ValuesController : ControllerBase
{
    private readonly IHelloClient _client;

    public ValuesController(IHelloClient client)
    {
        _client = client;
    }

    [HttpGet("/")]
    public async Task<ActionResult<Reply>> Index()
    {
        return await _client.GetMessageAsync();
    }
}
```

## <a name="outgoing-request-middleware"></a>送信要求ミドルウェア

`HttpClient` は既に、送信 HTTP 要求用にリンクできるハンドラーのデリゲートの概念を備えています。 `IHttpClientFactory` を使用すると、各名前付きクライアントに適用するハンドラーを簡単に定義できます。 複数のハンドラーを登録してチェーン化し、送信要求ミドルウェア パイプラインを構築できます。 各ハンドラーは、送信要求の前と後に処理を実行できます。 このパターンは、ASP.NET Core での受信ミドルウェア パイプラインに似ています。 このパターンは、キャッシュ、エラー処理、シリアル化、ログ記録など、HTTP 要求に関する横断的関心事を管理するためのメカニズムを提供します。

ハンドラーを作成するには、<xref:System.Net.Http.DelegatingHandler> の派生クラスを定義します。 パイプライン内の次のハンドラーに要求を渡す前にコードを実行するように、`SendAsync` メソッドをオーバーライドします。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Handlers/ValidateHeaderHandler.cs?name=snippet1)]

上記のコードでは、基本的なハンドラーが定義されています。 このコードは、`X-API-KEY` ヘッダーが要求に含まれていたかどうかを確認します。 ヘッダーがない場合、HTTP 呼び出しを行わずに適切な応答を返すことができます。

登録時には、1 つまたは複数のハンドラーを `HttpClient` の構成に追加することができます。 このタスクは、<xref:Microsoft.Extensions.DependencyInjection.IHttpClientBuilder> の拡張メソッドを使用して実行されます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet5)]

上記のコードでは、`ValidateHeaderHandler` が DI で登録されます。 `IHttpClientFactory` では、ハンドラーごとに個別の DI スコープが作成されます。 ハンドラーは、任意のスコープのサービスに自由に依存することができます。 ハンドラーが依存するサービスは、ハンドラーが破棄されるときに破棄されます。

登録が済むと、<xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.AddHttpMessageHandler*> を呼び出してハンドラーの型を渡すことができます。

実行する順序で、複数のハンドラーを登録することができます。 最後の `HttpClientHandler` が要求を実行するまで、各ハンドラーは次のハンドラーをラップします。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet6)]

次の方法のいずれかを使って、要求ごとの状態をメッセージ ハンドラーと共有します。

* `HttpRequestMessage.Properties` を使ってデータをハンドラーに渡します。
* `IHttpContextAccessor` を使って現在の要求にアクセスします。
* データを渡すカスタムの `AsyncLocal` ストレージ オブジェクトを作成します。

## <a name="use-polly-based-handlers"></a>Polly ベースのハンドラーを使用する

`IHttpClientFactory` は、[Polly](https://github.com/App-vNext/Polly) という名前の人気のあるサードパーティ製ライブラリと統合します。 Polly は、.NET 用の包括的な回復力および一時的エラー処理ライブラリです。 開発者は、自然でスレッド セーフな方法を使用して、Retry、Circuit Breaker、Timeout、Bulkhead Isolation、Fallback などのポリシーを表現できます。

構成されている `HttpClient` インスタンスで Polly ポリシーを使用できるようにするための、拡張メソッドが提供されています。 Polly の拡張機能は、次のようになっています。

* クライアントへの Polly ベースのハンドラーの追加がサポートされます。
* [Microsoft.Extensions.Http.Polly](https://www.nuget.org/packages/Microsoft.Extensions.Http.Polly/) NuGet パッケージのインストール後に使用できます。 このパッケージは、ASP.NET Core 共有フレームワークに含まれていません。

### <a name="handle-transient-faults"></a>一時的な障害を処理する

外部 HTTP を呼び出す際に発生する一般的な障害のほとんどは、一時的なものです。 `AddTransientHttpErrorPolicy` という便利な拡張メソッドが含まれており、一時的なエラーを処理するためのポリシーを定義できます。 この拡張メソッドで構成されたポリシーは、`HttpRequestException`、HTTP 5xx 応答、および HTTP 408 応答を処理します。

`AddTransientHttpErrorPolicy` 拡張メソッドは、`Startup.ConfigureServices` 内で使用できます。 この拡張メソッドは、可能性のある一時的障害を表すエラーを処理するように構成された `PolicyBuilder` オブジェクトへのアクセスを提供します。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet7)]

上記のコードでは、`WaitAndRetryAsync` ポリシーが定義されています。 失敗した要求は最大 3 回再試行され、再試行の間には 600 ミリ秒の遅延が設けられます。

### <a name="dynamically-select-policies"></a>ポリシーを動的に選択する

Polly ベースのハンドラーを追加するために使用できる追加の拡張メソッドが存在します。 そのような拡張メソッドの 1 つは `AddPolicyHandler` であり、複数のオーバーロードを持ちます。 1 つのオーバーロードでは、適用するポリシーを定義するときに要求を検査できます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet8)]

上記のコードでは、送信要求が HTTP GET の場合は、10 秒のタイムアウトが適用されます。 他の HTTP メソッドの場合は、30 秒のタイムアウトが使用されます。

### <a name="add-multiple-polly-handlers"></a>複数の Polly ハンドラーを追加する

Polly ポリシーを入れ子にして拡張機能を提供するのが一般的です。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet9)]

前の例では、2 つのハンドラーが追加されています。 最初のハンドラーは、`AddTransientHttpErrorPolicy` 拡張メソッドを使用して再試行ポリシーを追加します。 失敗した要求は 3 回まで再試行されます。 `AddTransientHttpErrorPolicy` の 2 番目の呼び出しでは、サーキット ブレーカー ポリシーが追加されています。 試行が連続して 5 回失敗した場合、それ以上の外部要求は 30 秒間ブロックされます。 サーキット ブレーカー ポリシーはステートフルです。 このクライアントからのすべての呼び出しは、同じサーキット状態を共有します。

### <a name="add-policies-from-the-polly-registry"></a>Polly レジストリからポリシーを追加する

定期的に使用されるポリシーを管理するには、ポリシーを 1 回定義した後、`PolicyRegistry` でポリシーを登録します。 提供されている拡張メソッドを使用することで、レジストリからポリシーを使用してハンドラーを追加できます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet10)]

上記のコードでは、`PolicyRegistry` が `ServiceCollection` に追加されるときに 2 つのポリシーが登録されています。 レジストリからポリシーを使用するには、適用するポリシーの名前を渡して `AddPolicyHandlerFromRegistry` メソッドを使用します。

`IHttpClientFactory` および Polly の統合について詳しくは、[Polly の Wiki](https://github.com/App-vNext/Polly/wiki/Polly-and-HttpClientFactory) をご覧ください。

## <a name="httpclient-and-lifetime-management"></a>HttpClient と有効期間の管理

`IHttpClientFactory` で `CreateClient` を呼び出すたびに、`HttpClient` の新しいインスタンスが返されます。 名前付きクライアントごとに <xref:System.Net.Http.HttpMessageHandler> が存在します。 ファクトリによって `HttpMessageHandler` インスタンスの有効期間が管理されます。

`IHttpClientFactory` は、リソースの消費量を減らすため、ファクトリによって作成された `HttpMessageHandler` のインスタンスをプールします。 新しい `HttpClient` インスタンスを作成するときに、プールの `HttpMessageHandler` インスタンスの有効期間が切れていない場合はそれを再利用する場合があります。

通常、各ハンドラーでは基になる HTTP 接続が独自に管理されるため、ハンドラーはプールすることが望まれます。 必要以上のハンドラーを作成すると、接続に遅延が発生する可能性があります。 また、一部のハンドラーは接続を無期限に開いており、DNS の変更にハンドラーが対応できないことがあります。

ハンドラーの既定の有効期間は 2 分です。 名前付きクライアントごとに、既定値をオーバーライドすることができます。 オーバーライドするには、クライアント作成時に返された `IHttpClientBuilder` で <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.SetHandlerLifetime*> を呼び出します。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet11)]

クライアントを破棄する必要はありません。 破棄すると送信要求がキャンセルされ、<xref:System.IDisposable.Dispose*> の呼び出し後には指定された `HttpClient` インスタンスを使用できなくなります。 `IHttpClientFactory` は、`HttpClient` インスタンスによって使用されたリソースの追跡と破棄を行います。 通常、`HttpClient` インスタンスは、破棄を必要としない .NET オブジェクトとして扱うことができます。

`IHttpClientFactory` の登場以前は、1 つの `HttpClient` インスタンスを長い期間使い続けるパターンが一般的に使用されていました。 `IHttpClientFactory` に移行した後は、このパターンは不要になります。

### <a name="alternatives-to-ihttpclientfactory"></a>IHttpClientFactory の代替手段

DI 対応のアプリ内で `IHttpClientFactory` を使用すれば、次のことを回避できます。

* `HttpMessageHandler` インスタンスをプールすることによるリソース枯渇の問題。
* 一定の間隔で `HttpMessageHandler` インスタンスを循環させることによって発生する古くなった DNS の問題。

有効期間の長い <xref:System.Net.Http.SocketsHttpHandler> インスタンスを使用して、上記の問題を解決する別の方法があります。

- アプリの起動時に `SocketsHttpHandler` インスタンスを作成し、アプリの有効期間中、それを使用します。
- DNS の更新時間に基づいて、<xref:System.Net.Http.SocketsHttpHandler.PooledConnectionLifetime> を適切な値に構成します。
- 必要に応じて `new HttpClient(handler, disposeHandler: false)` を使用して `HttpClient` インスタンスを作成します。

上記の方法を使用すると、`IHttpClientFactory` が同様の方法で解決するリソース管理の問題を解決できます。

- `SocketsHttpHandler` を使用すると、`HttpClient` インスタンス間で接続を共有できます。 この共有によってソケットの枯渇が防止されます。
- `SocketsHttpHandler` では、古くなった DNS の問題を回避するために `PooledConnectionLifetime` に従って接続を循環されます。

### <a name="cookies"></a>Cookies

`HttpMessageHandler` インスタンスをプールすると、`CookieContainer` オブジェクトが共有されます。 予期せぬ `CookieContainer` オブジェクト共有があると、多くの場合、コードは不適切なものとなります。 cookie を必要とするアプリの場合は、次のいずれかを検討してください。

 - 自動的な cookie 処理の無効化
 - `IHttpClientFactory` の回避

<xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> を呼び出して、自動的な cookie 処理を無効にします。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet13)]

## <a name="logging"></a>ログの記録

`IHttpClientFactory` によって作成されたクライアントは、すべての要求のログ メッセージを記録します。 既定のログ メッセージを見るには、ログの構成で適切な情報レベルを有効にします。 要求ヘッダーのログなどの追加ログは、トレース レベルでのみ含まれます。

各クライアントに使用されるログのカテゴリには、クライアントの名前が含まれます。 たとえば、*MyNamedClient* という名前のクライアントは、カテゴリ `System.Net.Http.HttpClient.MyNamedClient.LogicalHandler` でメッセージをログに記録します。 *LogicalHandler* というサフィックスが付いたメッセージが、要求ハンドラーのパイプラインの外部で発生します。 要求では、パイプライン内の他のハンドラーが要求を処理する前に、メッセージがログに記録されます。 応答では、他のパイプライン ハンドラーが応答を受け取った後で、メッセージがログに記録されます。

ログ記録は、要求ハンドラーのパイプラインの内部でも行われます。 *MyNamedClient* の例では、それらのメッセージはログ カテゴリ `System.Net.Http.HttpClient.MyNamedClient.ClientHandler` に対して記録されます。 要求では、他のすべてのハンドラーが実行した後、要求がネットワークに送信される直前に、これが行われます。 応答では、このログ記録には、ハンドラー パイプラインを通じ戻される前の応答の状態が含まれます。

パイプラインの外部と内部でログを有効にすると、他のパイプライン ハンドラーによる変更を検査できます。 これには、たとえば、要求ヘッダーや応答状態コードへの変更を含めることができます。

ログのカテゴリにクライアントの名前を含めると、必要に応じて、特定の名前付きクライアントでログをフィルター処理できます。

## <a name="configure-the-httpmessagehandler"></a>HttpMessageHandler を構成する

クライアントによって使用される内部 `HttpMessageHandler` の構成を制御することが必要な場合があります。

名前付きクライアントまたは型指定されたクライアントを追加すると、`IHttpClientBuilder` が返されます。 <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> 拡張メソッドを使用して、デリゲートを定義することができます。 デリゲートは、そのクライアントによって使用されるプライマリ `HttpMessageHandler` の作成と構成に使用されます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet12)]

## <a name="use-ihttpclientfactory-in-a-console-app"></a>コンソール アプリで IHttpClientFactory を使用する

コンソール アプリで、次のパッケージ参照をプロジェクトに追加します。

* [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting)
* [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http)

次に例を示します。

* <xref:System.Net.Http.IHttpClientFactory> は[汎用ホスト](xref:fundamentals/host/generic-host)のサービス コンテナーに登録されます。
* `MyService` により、クライアント ファクトリ インスタンスがサービスから作成され、それが `HttpClient` の作成に使用されます。 `HttpClient` は Web ページを取得する目的で使用されます。
* `Main` により、サービスの `GetPage` メソッドを実行し、Web ページ コンテンツの最初の 500 文字をコンソールに書き込むためのスコープが作成されます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactoryConsoleSample/Program.cs?highlight=14-15,20,26-27,59-62)]

## <a name="additional-resources"></a>その他の技術情報

* [HttpClientFactory を使用して回復力の高い HTTP 要求を実装する](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)
* [HttpClientFactory ポリシーと Polly ポリシーで指数バックオフを含む HTTP 呼び出しの再試行を実装する](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)
* [サーキット ブレーカー パターンを実装する](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-circuit-breaker-pattern)

::: moniker-end

::: moniker range="= aspnetcore-2.1"

寄稿者: [Glenn Condron](https://github.com/glennc)、[Ryan Nowak](https://github.com/rynowak)、[Steve Gordon](https://github.com/stevejgordon)

アプリ内で <xref:System.Net.Http.HttpClient> インスタンスを構成して作成するために、<xref:System.Net.Http.IHttpClientFactory> を登録して使用できます。 次のような利点があります。

* 論理 `HttpClient` インスタンスの名前付けと構成を一元化します。 たとえば、*github* クライアントを登録して、[GitHub](https://github.com/) にアクセスするように構成できます。 既定のクライアントは、他の目的に登録できます。
* `HttpClient` でのハンドラーのデリゲートにより送信ミドルウェアの概念を体系化し、Polly ベースのミドルウェアでそれを利用するための拡張機能を提供します。
* 基になっている `HttpClientMessageHandler` インスタンスのプールと有効期間を管理し、`HttpClient` の有効期間を手動で管理するときに発生する一般的な DNS の問題を防ぎます。
* ファクトリによって作成されたクライアントから送信されるすべての要求に対し、(`ILogger` によって) 構成可能なログ エクスペリエンスを追加します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/http-requests/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="prerequisites"></a>必須コンポーネント

.NET Framework をターゲットとするプロジェクトでは、[Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http/) NuGet パッケージのインストールが必要です。 .NET Core をターゲットとし、[Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)を参照するプロジェクトには、既に `Microsoft.Extensions.Http` パッケージが含まれています。

## <a name="consumption-patterns"></a>利用パターン

アプリで `IHttpClientFactory` を使用するには複数の方法があります。

* [基本的な使用方法](#basic-usage)
* [名前付きクライアント](#named-clients)
* [型指定されたクライアント](#typed-clients)
* [生成されたクライアント](#generated-clients)

これらの間に厳密な優劣はありません。 どの方法が最善かは、アプリの制約に依存します。

### <a name="basic-usage"></a>基本的な使用方法

`IHttpClientFactory` は、`Startup.ConfigureServices` メソッドの内部で `IServiceCollection` の `AddHttpClient` 拡張メソッドを呼び出すことによって登録できます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet1)]

登録が済むと、コードは、[依存関係の挿入 (DI)](xref:fundamentals/dependency-injection) を使用してサービスを挿入できる任意の場所で、`IHttpClientFactory` を受け取ることができます。 `IHttpClientFactory` を使用して、`HttpClient` インスタンスを作成できます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/BasicUsage.cshtml.cs?name=snippet1&highlight=9-12,21)]

このように `IHttpClientFactory` を使用するのは、既存のアプリをリファクタリングする優れた方法です。 `HttpClient` の使用方法に影響はありません。 現在 `HttpClient` インスタンスが作成されている場所で、それを <xref:System.Net.Http.IHttpClientFactory.CreateClient*> の呼び出しに置き換えます。

### <a name="named-clients"></a>名前付きクライアント

それぞれ構成が異なる多数の `HttpClient` をアプリで個別に使用する必要がある場合は、**名前付きクライアント** を使用することができます。 名前付き `HttpClient` の構成は、`Startup.ConfigureServices` での登録時に指定できます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet2)]

上記のコードでは、*github* という名前を指定して `AddHttpClient` を呼び出しています。 このクライアントには既定の構成がいくつか適用されています。つまり、GitHub API を使用するために必要なベース アドレスと 2 つのヘッダーです。

`CreateClient` を呼び出すたびに、`HttpClient` の新しいインスタンスが作成されて、構成アクションが呼び出されます。

名前付きクライアントを使用するには、文字列パラメーターを `CreateClient` に渡すことができます。 作成するクライアントの名前を指定します。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/NamedClient.cshtml.cs?name=snippet1&highlight=21)]

上記のコードでは、要求でホスト名を指定する必要はありません。 クライアントに構成されているベース アドレスが使用されるため、パスだけを渡すことができます。

### <a name="typed-clients"></a>型指定されたクライアント

型指定されたクライアント:

* キーとして文字列を使用する必要なしに、名前付きクライアントと同じ機能を提供します。
* クライアントを使用するときに、IntelliSense とコンパイラのヘルプが提供されます。
* 特定の `HttpClient` を構成してそれと対話する 1 つの場所を提供します。 たとえば、単一の型指定されたクライアントを単一のバックエンド エンドポイントに対して使用し、そのエンドポイントを操作するすべてのロジックをカプセル化できます。
* DI に対応しており、アプリ内の必要な場所に挿入できます。

型指定されたクライアントは、そのコンストラクターで `HttpClient` パラメーターを受け取ります。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/GitHub/GitHubService.cs?name=snippet1&highlight=5)]

上記のコードでは、構成が型指定されたクライアントに移動されています。 `HttpClient` オブジェクトは、パブリック プロパティとして公開されます。 `HttpClient` 機能を公開する API 固有のメソッドを定義することができます。 `GetAspNetDocsIssues` メソッドは、GitHub リポジトリで最新の未解決の問題をクエリして解析するために必要なコードをカプセル化します。

型指定されたクライアントを登録するには、ジェネリック <xref:Microsoft.Extensions.DependencyInjection.HttpClientFactoryServiceCollectionExtensions.AddHttpClient*> 拡張メソッドを `Startup.ConfigureServices` 内で使用して、型指定されたクライアントのクラスを指定します。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet3)]

型指定されたクライアントは、DI で一時的として登録されます。 型指定されたクライアントは、直接挿入して使用できます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/TypedClient.cshtml.cs?name=snippet1&highlight=11-14,20)]

好みに応じて、型指定されたクライアントのコンストラクターではなく、`Startup.ConfigureServices` での登録時に型指定されたクライアントの構成を指定できます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet4)]

型指定されたクライアントの内部に `HttpClient` を完全にカプセル化することができます。 プロパティとして公開するのではなく、`HttpClient` インスタンスを内部的に呼び出すパブリック メソッドとして提供することができます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/GitHub/RepoService.cs?name=snippet1&highlight=4)]

上記のコードでは、`HttpClient` はプライベート フィールドとして格納されます。 外部呼び出しを行うすべてのアクセスは、`GetRepos` メソッドを通して行われます。

### <a name="generated-clients"></a>生成されたクライアント

`IHttpClientFactory` は、[Refit](https://github.com/paulcbetts/refit) などの他のサードパーティ製ライブラリと組み合わせて使用できます。 Refit は、.NET 用の REST ライブラリです。 REST API をライブ インターフェイスに変換します。 インターフェイスの実装は `RestService` によって動的に生成され、`HttpClient` を使用して外部 HTTP の呼び出しを行います。

インターフェイスと応答は、外部の API とその応答を表すように定義されます。

```csharp
public interface IHelloClient
{
    [Get("/helloworld")]
    Task<Reply> GetMessageAsync();
}

public class Reply
{
    public string Message { get; set; }
}
```

型指定されたクライアントを追加し、Refit を使用して実装を生成できます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddHttpClient("hello", c =>
    {
        c.BaseAddress = new Uri("http://localhost:5000");
    })
    .AddTypedClient(c => Refit.RestService.For<IHelloClient>(c));

    services.AddMvc();
}
```

定義されたインターフェイスは、DI および Refit によって提供される実装で必要に応じて使用できます。

```csharp
[ApiController]
public class ValuesController : ControllerBase
{
    private readonly IHelloClient _client;

    public ValuesController(IHelloClient client)
    {
        _client = client;
    }

    [HttpGet("/")]
    public async Task<ActionResult<Reply>> Index()
    {
        return await _client.GetMessageAsync();
    }
}
```

## <a name="outgoing-request-middleware"></a>送信要求ミドルウェア

`HttpClient` は既に、送信 HTTP 要求用にリンクできるハンドラーのデリゲートの概念を備えています。 `IHttpClientFactory` を使用すると、各名前付きクライアントに適用するハンドラーを簡単に定義できます。 複数のハンドラーを登録してチェーン化し、送信要求ミドルウェア パイプラインを構築できます。 各ハンドラーは、送信要求の前と後に処理を実行できます。 このパターンは、ASP.NET Core での受信ミドルウェア パイプラインに似ています。 このパターンは、キャッシュ、エラー処理、シリアル化、ログ記録など、HTTP 要求に関する横断的関心事を管理するためのメカニズムを提供します。

ハンドラーを作成するには、<xref:System.Net.Http.DelegatingHandler> の派生クラスを定義します。 パイプライン内の次のハンドラーに要求を渡す前にコードを実行するように、`SendAsync` メソッドをオーバーライドします。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Handlers/ValidateHeaderHandler.cs?name=snippet1)]

上記のコードでは、基本的なハンドラーが定義されています。 このコードは、`X-API-KEY` ヘッダーが要求に含まれていたかどうかを確認します。 ヘッダーがない場合、HTTP 呼び出しを行わずに適切な応答を返すことができます。

登録時には、1 つまたは複数のハンドラーを `HttpClient` の構成に追加することができます。 このタスクは、<xref:Microsoft.Extensions.DependencyInjection.IHttpClientBuilder> の拡張メソッドを使用して実行されます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet5)]

上記のコードでは、`ValidateHeaderHandler` が DI で登録されます。 ハンドラーは、スコープではなく、一時的なサービスとして DI で登録する **必要があります**。 ハンドラーがスコープ付きサービスとして登録されており、ハンドラーが依存するサービスを破棄できる場合:

* ハンドラーのサービスは、ハンドラーがスコープ外に出る前に破棄されることがあります。
* 破棄されたハンドラー サービスが原因でハンドラーが失敗します。

登録が済むと、<xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.AddHttpMessageHandler*> を呼び出してハンドラーの型を渡すことができます。

実行する順序で、複数のハンドラーを登録することができます。 最後の `HttpClientHandler` が要求を実行するまで、各ハンドラーは次のハンドラーをラップします。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet6)]

次の方法のいずれかを使って、要求ごとの状態をメッセージ ハンドラーと共有します。

* `HttpRequestMessage.Properties` を使ってデータをハンドラーに渡します。
* `IHttpContextAccessor` を使って現在の要求にアクセスします。
* データを渡すカスタムの `AsyncLocal` ストレージ オブジェクトを作成します。

## <a name="use-polly-based-handlers"></a>Polly ベースのハンドラーを使用する

`IHttpClientFactory` は、[Polly](https://github.com/App-vNext/Polly) という名前の人気のあるサードパーティ製ライブラリと統合します。 Polly は、.NET 用の包括的な回復力および一時的エラー処理ライブラリです。 開発者は、自然でスレッド セーフな方法を使用して、Retry、Circuit Breaker、Timeout、Bulkhead Isolation、Fallback などのポリシーを表現できます。

構成されている `HttpClient` インスタンスで Polly ポリシーを使用できるようにするための、拡張メソッドが提供されています。 Polly の拡張機能は、次のようになっています。

* クライアントへの Polly ベースのハンドラーの追加がサポートされます。
* [Microsoft.Extensions.Http.Polly](https://www.nuget.org/packages/Microsoft.Extensions.Http.Polly/) NuGet パッケージのインストール後に使用できます。 このパッケージは、ASP.NET Core 共有フレームワークに含まれていません。

### <a name="handle-transient-faults"></a>一時的な障害を処理する

外部 HTTP を呼び出す際に発生する一般的な障害のほとんどは、一時的なものです。 `AddTransientHttpErrorPolicy` という便利な拡張メソッドが含まれており、一時的なエラーを処理するためのポリシーを定義できます。 この拡張メソッドで構成されたポリシーは、`HttpRequestException`、HTTP 5xx 応答、および HTTP 408 応答を処理します。

`AddTransientHttpErrorPolicy` 拡張メソッドは、`Startup.ConfigureServices` 内で使用できます。 この拡張メソッドは、可能性のある一時的障害を表すエラーを処理するように構成された `PolicyBuilder` オブジェクトへのアクセスを提供します。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet7)]

上記のコードでは、`WaitAndRetryAsync` ポリシーが定義されています。 失敗した要求は最大 3 回再試行され、再試行の間には 600 ミリ秒の遅延が設けられます。

### <a name="dynamically-select-policies"></a>ポリシーを動的に選択する

Polly ベースのハンドラーを追加するために使用できる追加の拡張メソッドが存在します。 そのような拡張メソッドの 1 つは `AddPolicyHandler` であり、複数のオーバーロードを持ちます。 1 つのオーバーロードでは、適用するポリシーを定義するときに要求を検査できます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet8)]

上記のコードでは、送信要求が HTTP GET の場合は、10 秒のタイムアウトが適用されます。 他の HTTP メソッドの場合は、30 秒のタイムアウトが使用されます。

### <a name="add-multiple-polly-handlers"></a>複数の Polly ハンドラーを追加する

Polly ポリシーを入れ子にして拡張機能を提供するのが一般的です。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet9)]

前の例では、2 つのハンドラーが追加されています。 最初のハンドラーは、`AddTransientHttpErrorPolicy` 拡張メソッドを使用して再試行ポリシーを追加します。 失敗した要求は 3 回まで再試行されます。 `AddTransientHttpErrorPolicy` の 2 番目の呼び出しでは、サーキット ブレーカー ポリシーが追加されています。 試行が連続して 5 回失敗した場合、それ以上の外部要求は 30 秒間ブロックされます。 サーキット ブレーカー ポリシーはステートフルです。 このクライアントからのすべての呼び出しは、同じサーキット状態を共有します。

### <a name="add-policies-from-the-polly-registry"></a>Polly レジストリからポリシーを追加する

定期的に使用されるポリシーを管理するには、ポリシーを 1 回定義した後、`PolicyRegistry` でポリシーを登録します。 提供されている拡張メソッドを使用することで、レジストリからポリシーを使用してハンドラーを追加できます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet10)]

上記のコードでは、`PolicyRegistry` が `ServiceCollection` に追加されるときに 2 つのポリシーが登録されています。 レジストリからポリシーを使用するには、適用するポリシーの名前を渡して `AddPolicyHandlerFromRegistry` メソッドを使用します。

`IHttpClientFactory` および Polly の統合について詳しくは、[Polly の Wiki](https://github.com/App-vNext/Polly/wiki/Polly-and-HttpClientFactory) をご覧ください。

## <a name="httpclient-and-lifetime-management"></a>HttpClient と有効期間の管理

`IHttpClientFactory` で `CreateClient` を呼び出すたびに、`HttpClient` の新しいインスタンスが返されます。 名前付きクライアントごとに <xref:System.Net.Http.HttpMessageHandler> が存在します。 ファクトリによって `HttpMessageHandler` インスタンスの有効期間が管理されます。

`IHttpClientFactory` は、リソースの消費量を減らすため、ファクトリによって作成された `HttpMessageHandler` のインスタンスをプールします。 新しい `HttpClient` インスタンスを作成するときに、プールの `HttpMessageHandler` インスタンスの有効期間が切れていない場合はそれを再利用する場合があります。

通常、各ハンドラーでは基になる HTTP 接続が独自に管理されるため、ハンドラーはプールすることが望まれます。 必要以上のハンドラーを作成すると、接続に遅延が発生する可能性があります。 また、一部のハンドラーは接続を無期限に開いており、DNS の変更にハンドラーが対応できないことがあります。

ハンドラーの既定の有効期間は 2 分です。 名前付きクライアントごとに、既定値をオーバーライドすることができます。 オーバーライドするには、クライアント作成時に返された `IHttpClientBuilder` で <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.SetHandlerLifetime*> を呼び出します。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet11)]

クライアントを破棄する必要はありません。 破棄すると送信要求がキャンセルされ、<xref:System.IDisposable.Dispose*> の呼び出し後には指定された `HttpClient` インスタンスを使用できなくなります。 `IHttpClientFactory` は、`HttpClient` インスタンスによって使用されたリソースの追跡と破棄を行います。 通常、`HttpClient` インスタンスは、破棄を必要としない .NET オブジェクトとして扱うことができます。

`IHttpClientFactory` の登場以前は、1 つの `HttpClient` インスタンスを長い期間使い続けるパターンが一般的に使用されていました。 `IHttpClientFactory` に移行した後は、このパターンは不要になります。

### <a name="alternatives-to-ihttpclientfactory"></a>IHttpClientFactory の代替手段

DI 対応のアプリ内で `IHttpClientFactory` を使用すれば、次のことを回避できます。

* `HttpMessageHandler` インスタンスをプールすることによるリソース枯渇の問題。
* 一定の間隔で `HttpMessageHandler` インスタンスを循環させることによって発生する古くなった DNS の問題。

有効期間の長い <xref:System.Net.Http.SocketsHttpHandler> インスタンスを使用して、上記の問題を解決する別の方法があります。

- アプリの起動時に `SocketsHttpHandler` インスタンスを作成し、アプリの有効期間中、それを使用します。
- DNS の更新時間に基づいて、<xref:System.Net.Http.SocketsHttpHandler.PooledConnectionLifetime> を適切な値に構成します。
- 必要に応じて `new HttpClient(handler, disposeHandler: false)` を使用して `HttpClient` インスタンスを作成します。

上記の方法を使用すると、`IHttpClientFactory` が同様の方法で解決するリソース管理の問題を解決できます。

- `SocketsHttpHandler` を使用すると、`HttpClient` インスタンス間で接続を共有できます。 この共有によってソケットの枯渇が防止されます。
- `SocketsHttpHandler` では、古くなった DNS の問題を回避するために `PooledConnectionLifetime` に従って接続を循環されます。

### <a name="cookies"></a>Cookies

`HttpMessageHandler` インスタンスをプールすると、`CookieContainer` オブジェクトが共有されます。 予期せぬ `CookieContainer` オブジェクト共有があると、多くの場合、コードは不適切なものとなります。 cookie を必要とするアプリの場合は、次のいずれかを検討してください。

 - 自動的な cookie 処理の無効化
 - `IHttpClientFactory` の回避

<xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> を呼び出して、自動的な cookie 処理を無効にします。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet13)]

## <a name="logging"></a>ログの記録

`IHttpClientFactory` によって作成されたクライアントは、すべての要求のログ メッセージを記録します。 既定のログ メッセージを見るには、ログの構成で適切な情報レベルを有効にします。 要求ヘッダーのログなどの追加ログは、トレース レベルでのみ含まれます。

各クライアントに使用されるログのカテゴリには、クライアントの名前が含まれます。 たとえば、*MyNamedClient* という名前のクライアントは、カテゴリ `System.Net.Http.HttpClient.MyNamedClient.LogicalHandler` でメッセージをログに記録します。 *LogicalHandler* というサフィックスが付いたメッセージが、要求ハンドラーのパイプラインの外部で発生します。 要求では、パイプライン内の他のハンドラーが要求を処理する前に、メッセージがログに記録されます。 応答では、他のパイプライン ハンドラーが応答を受け取った後で、メッセージがログに記録されます。

ログ記録は、要求ハンドラーのパイプラインの内部でも行われます。 *MyNamedClient* の例では、それらのメッセージはログ カテゴリ `System.Net.Http.HttpClient.MyNamedClient.ClientHandler` に対して記録されます。 要求では、他のすべてのハンドラーが実行した後、要求がネットワークに送信される直前に、これが行われます。 応答では、このログ記録には、ハンドラー パイプラインを通じ戻される前の応答の状態が含まれます。

パイプラインの外部と内部でログを有効にすると、他のパイプライン ハンドラーによる変更を検査できます。 これには、たとえば、要求ヘッダーや応答状態コードへの変更を含めることができます。

ログのカテゴリにクライアントの名前を含めると、必要に応じて、特定の名前付きクライアントでログをフィルター処理できます。

## <a name="configure-the-httpmessagehandler"></a>HttpMessageHandler を構成する

クライアントによって使用される内部 `HttpMessageHandler` の構成を制御することが必要な場合があります。

名前付きクライアントまたは型指定されたクライアントを追加すると、`IHttpClientBuilder` が返されます。 <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> 拡張メソッドを使用して、デリゲートを定義することができます。 デリゲートは、そのクライアントによって使用されるプライマリ `HttpMessageHandler` の作成と構成に使用されます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet12)]

## <a name="use-ihttpclientfactory-in-a-console-app"></a>コンソール アプリで IHttpClientFactory を使用する

コンソール アプリで、次のパッケージ参照をプロジェクトに追加します。

* [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting)
* [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http)

次に例を示します。

* <xref:System.Net.Http.IHttpClientFactory> は[汎用ホスト](xref:fundamentals/host/generic-host)のサービス コンテナーに登録されます。
* `MyService` により、クライアント ファクトリ インスタンスがサービスから作成され、それが `HttpClient` の作成に使用されます。 `HttpClient` は Web ページを取得する目的で使用されます。
* `Main` により、サービスの `GetPage` メソッドを実行し、Web ページ コンテンツの最初の 500 文字をコンソールに書き込むためのスコープが作成されます。

[!code-csharp[](http-requests/samples/2.x/HttpClientFactoryConsoleSample/Program.cs?highlight=14-15,20,26-27,59-62)]

## <a name="header-propagation-middleware"></a>ヘッダー伝達ミドルウェア

ヘッダー伝達は、受信要求から送信 HTTP クライアント要求に HTTP ヘッダーを伝達するためのコミュニティでサポートされたミドルウェアです。 ヘッダー伝達を使用するには、次を行います。

* パッケージ [HeaderPropagation](https://www.nuget.org/packages/HeaderPropagation) のコミュニティでサポートされているポートを参照します。 ASP.NET Core 3.1 以降では、[Microsoft.AspNetCore.HeaderPropagation](https://www.nuget.org/packages/Microsoft.AspNetCore.HeaderPropagation) がサポートされています。

* `Startup` でミドルウェアと `HttpClient` を構成します。

  [!code-csharp[](http-requests/samples/2.x/Startup21.cs?highlight=5-9,25&name=snippet)]

* クライアントで、構成されたヘッダーが送信要求に含まれます。

  ```csharp
  var client = clientFactory.CreateClient("MyForwardingClient");
  var response = client.GetAsync(...);
  ```

## <a name="additional-resources"></a>その他の技術情報

* [HttpClientFactory を使用して回復力の高い HTTP 要求を実装する](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)
* [HttpClientFactory ポリシーと Polly ポリシーで指数バックオフを含む HTTP 呼び出しの再試行を実装する](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)
* [サーキット ブレーカー パターンを実装する](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-circuit-breaker-pattern)

::: moniker-end
