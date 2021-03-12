---
title: ASP.NET Core でのセッション
author: rick-anderson
description: 要求間でセッションを維持する方法を説明します。
ms.author: riande
ms.custom: mvc
ms.date: 03/06/2020
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
uid: fundamentals/app-state
ms.openlocfilehash: ab09867a1b1aa73e423babbe295036e292fc733f
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586580"
---
# <a name="session-and-state-management-in-aspnet-core"></a>ASP.NET Core でのセッションと状態の管理

::: moniker range=">= aspnetcore-3.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)、[Kirk Larkin](https://twitter.com/serpent5)、および [Diana LaRose](https://github.com/DianaLaRose)

HTTP はステートレス プロトコルです。 既定で、HTTP 要求は独立したメッセージであり、ユーザーの値は保持されません。 この記事では、要求間でユーザー データを保持するためのいくつかの方法について説明します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/app-state/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="state-management"></a>状態管理

状態は、いくつかの方法で格納することができます。 このトピックではそれぞれの方法について説明します。

| 格納の方法 | 格納のメカニズム |
| ---------------- | ----------------- |
| [Cookie ](#cookies) | HTTP cookie。 サーバー側アプリ コードを使って格納されたデータを含めることができます。 |
| [セッション状態](#session-state) | HTTP cookie とサーバー側アプリ コード |
| [TempData](#tempdata) | HTTP cookie またはセッション状態 |
| [クエリ文字列](#query-strings) | HTTP クエリ文字列 |
| [非表示フィールド](#hidden-fields) | HTTP フォーム フィールド |
| [HttpContext.Items](#httpcontextitems) | サーバー側アプリ コード |
| [キャッシュ](#cache) | サーバー側アプリ コード |

## <a name="cookies"></a>Cookies

Cookie には、要求と要求の間でデータが格納されます。 cookie は要求ごとに送信されるために、そのサイズは最小に抑える必要があります。 理想的には、識別子だけを cookie に格納し、データはアプリで格納します。 ほとんどのブラウザーで cookie のサイズは 4096 バイトに制限されています。 ドメインごとに使用できる cookie の数も制限されています。

cookie は改ざんされる可能性があるため、アプリで検証する必要があります。 Cookie は、ユーザーが削除でき、クライアント上で期限切れになります。 ただし、cookie は一般に、クライアントでデータを永続化するときに最も持続性のある形式です。

Cookie は、多くの場合、パーソナル化に利用されます。既知のユーザーのためにコンテンツをカスタマイズします。 ユーザーは識別されるだけであり、ほとんどの場合は認証されません。 cookie には、ユーザーの名前、アカウント名、または GUID などの一意のユーザー ID を格納できます。 cookie は、好みの Web サイトの背景色など、ユーザーの個人用設定にアクセスするために使用できます。

cookie を発行し、プライバシーの問題を扱うときは、[欧州連合の一般データ保護規則 (GDPR)](https://ec.europa.eu/info/law/law-topic/data-protection) を参照してください。 詳細については、「[General Data Protection Regulation (GDPR) support in ASP.NET Core](xref:security/gdpr)」(ASP.NET Core での一般データ保護規則 (GDPR) のサポート) をご覧ください。

## <a name="session-state"></a>セッション状態

セッション状態は、ユーザーが Web アプリを参照している期間中ユーザー データを格納するための ASP.NET Core のシナリオです。 セッション状態では、アプリで管理されているストアを使用して、クライアントからの要求間でデータを保持します。 セッション データはキャッシュによってバックアップされ、一時的なデータと見なされます。 セッション データがなくても、サイトは機能し続けられる必要があります。 重要なアプリケーションのデータはユーザー データベースに格納し、パフォーマンスの最適化としてのみセッションでキャッシュする必要があります。

[SignalR ハブ](xref:signalr/hubs)は HTTP コンテキストとは独立して実行する可能性があるため、[SignalR](xref:signalr/index) アプリではセッションはサポートされていません。 このようなことは、たとえば、長いポーリング要求が HTTP コンテキストの有効期間を超えてハブによって開かれている場合に発生する可能性があります。

ASP.NET Core により、セッション ID を含む cookie がクライアントに提供されて、セッションの状態が維持されます。 cookie セッション ID:

* 各要求でアプリに送信されます。
* アプリによってセッション データをフェッチするために使用されます。

セッション状態は次の動作を示します。

* セッション cookie は、ブラウザーに固有です。 セッションはブラウザー間で共有されません。
* セッション cookie は、ブラウザー セッションが終了するときに削除されます。
* cookie を受け取り、セッションが期限切れになった場合、同じセッション cookie を使用する新しいセッションが作成されます。
* 空のセッションは保持されません。 要求間でセッションを維持するには、少なくとも 1 つの値をセッションが持っている必要があります。 セッションが保持されないと、新しい要求ごとに新しいセッション ID が生成されます。
* アプリは、最後の要求から限られた時間だけセッションを維持します。 アプリでは、セッション タイムアウトを設定するか、既定値の 20 分を使用します。 セッション状態は、次のユーザー データの格納に最適です。
  * 特定のセッションに固有である。
  * データがセッション間で永続的に保存される必要がない。
* セッション データは、<xref:Microsoft.AspNetCore.Http.ISession.Clear%2A?displayProperty=nameWithType> の実装が呼び出されるか、セッションが期限切れになると、削除されます。
* クライアント ブラウザーが閉じられたこと、またはクライアントでセッション cookie が削除されるか期限切れになったことを、アプリ コードに通知する既定のメカニズムはありません。
* セッション状態の cookie は既定では必須としてマークされていません。 サイトの訪問者が追跡を許可しない限り、セッション状態は機能しません。 詳細については、「<xref:security/gdpr#tempdata-provider-and-session-state-cookies-arent-essential>」を参照してください。

> [!WARNING]
> セッション状態には機密データを保存しないでください。 ユーザーがブラウザーを閉じず、セッション cookie がクリアされない可能性があります。 一部のブラウザーでは、ブラウザーのウィンドウ間で有効なセッションの cookie が維持されます。 セッションが 1 人のユーザーに制限されないことがあります。 次のユーザーが、同じセッション cookie でアプリを閲覧し続けることがあります。

メモリ内キャッシュ プロバイダーは、アプリが存在するサーバーのメモリにセッション データを格納します。 サーバー ファームのシナリオでは次のようになります。

* "*固定セッション*" を使用して、個々のサーバー上の特定のアプリのインスタンスに、各セッションを結び付けます。 [Azure App Service](https://azure.microsoft.com/services/app-service/) は[アプリケーション要求ルーティング処理 (ARR)](/iis/extensions/planning-for-arr/using-the-application-request-routing-module) を使って、既定で固定セッションを強制的に使用します。 ただし、固定セッションは拡張性に影響を与え、Web アプリの更新を複雑にすることがあります。 もっとよい方法は、Redis または SQL Server の分散キャッシュを使用することで、固定セッションを必要としません。 詳細については、「<xref:performance/caching/distributed>」を参照してください。
* セッション cookie は <xref:Microsoft.AspNetCore.DataProtection.IDataProtector> によって暗号化されます。 各コンピューターでセッション cookie が読み取られるようにするには、データ保護を適切に構成する必要があります。 詳細については、<xref:security/data-protection/introduction> および[キー ストレージ プロバイダー](xref:security/data-protection/implementation/key-storage-providers)に関する記事をご覧ください。

### <a name="configure-session-state"></a>セッション状態を構成する

[Microsoft.AspNetCore.Session](https://www.nuget.org/packages/Microsoft.AspNetCore.Session/) パッケージは:

* フレームワークによって暗黙的に含まれます。
* セッション状態を管理するためのミドルウェアを提供します。

セッション ミドルウェアを有効にするには、`Startup` に次が含まれている必要があります。

* いずれかの <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> メモリ キャッシュ。 `IDistributedCache` 実装はセッションのバックアップ ストアとして利用されます。 詳細については、「<xref:performance/caching/distributed>」を参照してください。
* `ConfigureServices` での <xref:Microsoft.Extensions.DependencyInjection.SessionServiceCollectionExtensions.AddSession%2A> の呼び出し。
* `Configure` での <xref:Microsoft.AspNetCore.Builder.SessionMiddlewareExtensions.UseSession%2A> の呼び出し。

次のコードでは、`IDistributedCache` の既定のメモリ内実装でメモリ内セッション プロバイダーを設定する方法を示します。

[!code-csharp[](app-state/samples/3.x/SessionSample/Startup4.cs?name=snippet1&highlight=12-19,45)]

前のコードでは、テストを簡単にするために短いタイムアウトを設定しています。

ミドルウェアの順序が重要です。  `UseSession` を `UseRouting` の後、かつ `UseEndpoints` の前に呼び出します。 [ミドルウェアの順序付け](xref:fundamentals/middleware/index#order)に関するページを参照してください。

[HttpContext.Session](xref:Microsoft.AspNetCore.Http.HttpContext.Session) は、セッション状態を構成した後で使用できます。

`UseSession` を呼び出前に `HttpContext.Session` にアクセスすることはできません。

アプリが応答ストリームへの書き込みを開始した後では、新しいセッション cookie を含む新しいセッションを作成できません。 例外は Web サーバー ログに記録され、ブラウザーには表示されません。

### <a name="load-session-state-asynchronously"></a>セッション状態を非同期的に読み込む

ASP.NET Core の既定のセッション プロバイダーでは、<xref:Microsoft.AspNetCore.Http.ISession.TryGetValue%2A>、<xref:Microsoft.AspNetCore.Http.ISession.Set%2A>、または <xref:Microsoft.AspNetCore.Http.ISession.Remove%2A> メソッドの前に <xref:Microsoft.AspNetCore.Http.ISession.LoadAsync%2A?displayProperty=nameWithType> メソッドが明示的に呼び出された場合にのみ、基になる <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> バッキング ストアから非同期的にセッション レコードを読み込みます。 `LoadAsync` を最初に呼び出さないと、基になっているセッション レコードは同期的に読み込まれ、パフォーマンスが大幅に低下する可能性があります。

アプリにこのパターンを強制させるには、`TryGetValue`、`Set`、または `Remove` の前に `LoadAsync` メソッドが呼び出されない場合に例外をスローするバージョンで、<xref:Microsoft.AspNetCore.Session.DistributedSessionStore> および <xref:Microsoft.AspNetCore.Session.DistributedSession> の実装をラップします。 ラップしたバージョンをサービス コンテナーに登録します。

### <a name="session-options"></a>セッション オプション

セッションの既定値をオーバーライドするには、<xref:Microsoft.AspNetCore.Builder.SessionOptions> を使用します。

| オプション | 説明 |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.Builder.SessionOptions.Cookie> | cookie の作成に使用される設定を決定します。 <xref:Microsoft.AspNetCore.Http.CookieBuilder.Name> の既定値は <xref:Microsoft.AspNetCore.Session.SessionDefaults.CookieName?displayProperty=nameWithType> (`.AspNetCore.Session`) です。 <xref:Microsoft.AspNetCore.Http.CookieBuilder.Path> の既定値は <xref:Microsoft.AspNetCore.Session.SessionDefaults.CookiePath?displayProperty=nameWithType> (`/`) です。 <xref:Microsoft.AspNetCore.Http.CookieBuilder.SameSite> の既定値は <xref:Microsoft.AspNetCore.Http.SameSiteMode.Lax?displayProperty=nameWithType> (`1`) です。 <xref:Microsoft.AspNetCore.Http.CookieBuilder.HttpOnly> では、既定値が `true` に設定されます。 <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> では、既定値が `false` に設定されます。 |
| <xref:Microsoft.AspNetCore.Builder.SessionOptions.IdleTimeout> | `IdleTimeout` は、内容を破棄されることなくセッションがアイドル状態になっていることのできる最大時間を示します。 セッションへのアクセスがあるたびに、タイムアウトはリセットされます。 この設定はセッションの内容にのみ適用され、cookie には適用されません。 既定値は 20 分です。 |
| <xref:Microsoft.AspNetCore.Builder.SessionOptions.IOTimeout> | ストアからのセッションの読み込み、またはストアに戻すコミットに対して許容される最大時間です。 この設定は非同期操作にのみ適用できます。 このタイムアウトは、<xref:System.Threading.Timeout.InfiniteTimeSpan> を使用して無効にすることができます。 既定値は 1 分です。 |

セッションは cookie を利用し、1 つのブラウザーからの要求を追跡し、識別します。 既定では、この cookie は `.AspNetCore.Session` という名前であり、パス `/` が使用されます。 cookie の既定値ではドメインが指定されないため、ページのクライアント側スクリプトには使用できません (<xref:Microsoft.AspNetCore.Http.CookieBuilder.HttpOnly> の既定値が `true` になるため)。

cookie セッションの既定値をオーバーライドするには、<xref:Microsoft.AspNetCore.Builder.SessionOptions> を使用します。

[!code-csharp[](app-state/samples/3.x/SessionSample/Startup2.cs?name=snippet1&highlight=5-10)]

アプリでは、<xref:Microsoft.AspNetCore.Builder.SessionOptions.IdleTimeout> プロパティを使用して、サーバーのキャッシュ内の内容が破棄されるまでのセッションのアイドル時間が決定されます。 このプロパティは cookie の有効期限に依存しません。 要求が[セッション ミドルウェア](xref:Microsoft.AspNetCore.Session.SessionMiddleware)を通過するたびにタイムアウトがリセットされます。

セッション状態は "*ロックなし*" です。 2 つの要求がセッションの内容を同時に変更しようとした場合、最後の要求が最初の要求をオーバーライドします。 `Session` は *一貫性のあるセッション* として実装されます。つまり、コンテンツは全部まとめて保管されます。 2 つの要求が異なるセッション値を変更しようとしたとき、最後の要求が最初の要求によって行われたセッションの変更をオーバーライドすることがあります。

### <a name="set-and-get-session-values"></a>セッション値の設定および取得

セッション状態にアクセスするには、Razor Pages の <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> クラスか、MVC の <xref:Microsoft.AspNetCore.Mvc.Controller> クラスを <xref:Microsoft.AspNetCore.Http.HttpContext.Session?displayProperty=nameWithType> と共に使用します。 このプロパティは <xref:Microsoft.AspNetCore.Http.ISession> の実装です。

`ISession` の実装では、整数値や文字列値を設定および取得するための複数の拡張メソッドが提供されています。 拡張メソッドは <xref:Microsoft.AspNetCore.Http> 名前空間にあります。

`ISession` 拡張メソッド:

* [Get(ISession, String)](xref:Microsoft.AspNetCore.Http.SessionExtensions.Get%2A)
* [GetInt32(ISession, String)](xref:Microsoft.AspNetCore.Http.SessionExtensions.GetInt32%2A)
* [GetString(ISession, String)](xref:Microsoft.AspNetCore.Http.SessionExtensions.GetString%2A)
* [SetInt32(ISession, String, Int32)](xref:Microsoft.AspNetCore.Http.SessionExtensions.SetInt32%2A)
* [SetString(ISession, String, String)](xref:Microsoft.AspNetCore.Http.SessionExtensions.SetString%2A)

次の例では、`IndexModel.SessionKeyName` キー (サンプル アプリ内の `_Name`) のセッション値を Razor Pages ページで取得します。

```csharp
@page
@using Microsoft.AspNetCore.Http
@model IndexModel

...

Name: @HttpContext.Session.GetString(IndexModel.SessionKeyName)
```

次の例では、整数および文字列を設定および取得する方法を示します。

[!code-csharp[](app-state/samples/3.x/SessionSample/Pages/Index.cshtml.cs?name=snippet1&highlight=18-19,22-23)]

分散キャッシュのシナリオを有効にするには、メモリ内キャッシュを使用する場合であっても、すべてのセッション データをシリアル化する必要があります。 文字列と整数のシリアライザーは、<xref:Microsoft.AspNetCore.Http.ISession> の拡張メソッドによって提供されます。 複合型は、JSON などの別のメカニズムを使ってユーザーがシリアル化する必要があります。

次のサンプル コードを使用して、オブジェクトをシリアル化します。

[!code-csharp[](app-state/samples/3.x/SessionSample/Extensions/SessionExtensions.cs?name=snippet1)]

次の例では、シリアル化可能オブジェクトを `SessionExtensions` クラスで設定および取得する方法を示します。

[!code-csharp[](app-state/samples/3.x/SessionSample/Pages/Index.cshtml.cs?name=snippet2)]

## <a name="tempdata"></a>TempData

ASP.NET Core によって、Razor Pages [TempData](xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel.TempData) または Controller <xref:Microsoft.AspNetCore.Mvc.Controller.TempData> が発行されます。 このプロパティには、別の要求で読み取られるまでデータが格納されます。 [Keep (String)](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ITempDataDictionary.Keep*) メソッドと [Peek (String)](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ITempDataDictionary.Peek*) メソッドを使用すると、要求の最後に削除せずにデータを調べることができます。 [Keep](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ITempDataDictionary.Keep*) によって、リテンション期間についてディクショナリ内のすべての項目がマークされます。 `TempData` は次のようになります。

* 複数の要求に対してデータが必要な場合のリダイレクトに役立ちます。
* `TempData` プロバイダーによって cookie またはセッション状態を使用して実装されます。

## <a name="tempdata-samples"></a>TempData のサンプル

顧客を作成する次のページについて考えてみましょう。

[!code-csharp[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet&highlight=15-16,30)]

次のページに `TempData["Message"]` が表示されます。

[!code-cshtml[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/IndexPeek.cshtml?range=1-14)]

前のマークアップでは、`Peek` が使用されているため、要求の最後に `TempData["Message"]` が削除 **されません**。 ページを更新すると `TempData["Message"]` のコンテンツが表示されます。

次のマークアップは前のコードと似ていますが、`Keep` を使用して要求の最後にデータを保存しています。

[!code-cshtml[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/IndexKeep.cshtml?range=1-14)]

*Indexpeek* ページと *IndexKeep* ページ間を移動しても `TempData["Message"]` は削除されません。

次のコードでは `TempData["Message"]` が表示されますが、要求の最後に `TempData["Message"]` が削除されます。

[!code-cshtml[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/Index.cshtml?range=1-14)]

### <a name="tempdata-providers"></a>TempData プロバイダー

cookie に TempData を格納するには、既定では、cookie ベースの TempData プロバイダーが使われます。

cookie データは、<xref:Microsoft.AspNetCore.DataProtection.IDataProtector> を使用して暗号化され、<xref:Microsoft.AspNetCore.WebUtilities.Base64UrlTextEncoder> でエンコードされた後、チャンクされます。 cookie の最大サイズは、暗号化とチャンクのため、[4096](http://www.faqs.org/rfcs/rfc2965.html) バイト未満です。 暗号化されているデータを圧縮すると、[CRIME](https://wikipedia.org/wiki/CRIME_(security_exploit)) 攻撃や [BREACH](https://wikipedia.org/wiki/BREACH_(security_exploit)) 攻撃など、セキュリティ上の問題を起す可能性があるため、cookie データは圧縮されません。 cookie ベース TempData プロバイダーの詳細については、<xref:Microsoft.AspNetCore.Mvc.ViewFeatures.CookieTempDataProvider> に関する記事を参照してください。

### <a name="choose-a-tempdata-provider"></a>TempData プロバイダーを選択する

TempData プロバイダーを選択するときの考慮事項:

* アプリは既にセッション状態を使っているかどうか。 その場合、データのサイズを超えて、セッション状態 TempData プロバイダーがそのアプリにコストを追加することはありません。
* アプリでは、比較的少量のデータに対して (最大 500 バイト) TempData がわずかばかり使用されているか。 該当する場合、cookie TempData プロバイダーによって TempData を送信する要求ごとに少額のコストが追加されます。 該当しない場合、セッション状態 TempData プロバイダーは便利かもしれません。TempData が尽きるまで、要求のたびに大量のデータをラウンドトリップすることが回避されます。
* アプリは複数サーバーのサーバー ファームで実行しているか。 そうである場合は、データ保護の外部で cookie TempData プロバイダーを使用するために、追加の構成は必要ありません (「<xref:security/data-protection/introduction>」および[キー ストレージ プロバイダー](xref:security/data-protection/implementation/key-storage-providers)に関する記事を参照)。

Web ブラウザーなどのほとんどの Web クライアントによって、各 cookie の最大サイズと cookie の合計数に上限が設けられます。 cookie TempData プロバイダーを使用するとき、アプリで[それらの上限](http://www.faqs.org/rfcs/rfc2965.html)が超えないことを確認してください。 データの合計サイズを考慮してください。 暗号化とチャンクによる cookie のサイズの増加を考慮してください。

### <a name="configure-the-tempdata-provider"></a>TempData プロバイダーを構成する

cookie ベース TempData プロバイダーは既定で有効になります。

セッション ベースの TempData プロバイダーを有効にするには、<xref:Microsoft.Extensions.DependencyInjection.MvcViewFeaturesMvcBuilderExtensions.AddSessionStateTempDataProvider%2A> 拡張メソッドを使います。 `AddSessionStateTempDataProvider` の呼び出しは 1 つだけ必要です。

[!code-csharp[](app-state/samples/3.x/SessionSample/Startup3.cs?name=snippet1&highlight=4,6,8,30)]

## <a name="query-strings"></a>クエリ文字列

限られた量のデータを要求間で渡すことができます。新しい要求のクエリ文字列にそれを追加します。 これは、リンクと埋め込まれた状態がメールまたはソーシャル ネットワークを通して共有されるよう、永久的に状態をキャプチャするのに役立ちます。 URL クエリ文字列はパブリックであるため、機密データにはクエリ文字列を使わないでください。

意図しない共有に加えて、クエリ文字列にデータを含めると、アプリが[クロスサイト リクエスト フォージェリ (CSRF)](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)) 攻撃に晒される可能性があります。 保存されたセッション状態を CSRF 攻撃から保護する必要があります。 詳細については、「<xref:security/anti-request-forgery>」を参照してください。

## <a name="hidden-fields"></a>非表示フィールド

データは非表示フォーム フィールドに保存したり、次の要求で転記したりできます。 複数ページ フォームでは、これは一般的です。 クライアントがデータを改ざんする可能性があるため、アプリで非表示フィールドに格納されているデータを常に再検証する必要があります。

## <a name="httpcontextitems"></a>HttpContext.Items

1 つの要求を処理している間にデータを格納するには、<xref:Microsoft.AspNetCore.Http.HttpContext.Items?displayProperty=nameWithType> コレクションを使います。 要求が処理された後、コレクションの内容は破棄されます。 `Items` コレクションは、コンポーネントまたはミドルウェアが要求の間の異なる時点で動作し、パラメーターを受け渡す直接的な方法がない場合に、コンポーネントとミドルウェアが通信できるようにするためによく使われます。

次の例では、[ミドルウェア](xref:fundamentals/middleware/index)により `isVerified` が `Items` コレクションに追加されます。

[!code-csharp[](app-state/samples/3.x/SessionSample/Startup.cs?name=snippet1)]

1 つのアプリでのみ使用されるミドルウェアの場合、固定の `string` キーを使用できます。 アプリ間で共有されるミドルウェアでは、キーの競合を避けるため、一意のオブジェクト キーを使う必要があります。 次の例では、ミドルウェア クラスで定義されている一意のオブジェクト キーを使用する方法を示します。

[!code-csharp[](app-state/samples/3.x/SessionSample/Middleware/HttpContextItemsMiddleware.cs?name=snippet1&highlight=4,13)]

その他のコードは、ミドルウェア クラスで公開されるキーを利用し、`HttpContext.Items` に格納されている値にアクセスできます。

[!code-csharp[](app-state/samples/3.x/SessionSample/Pages/Index.cshtml.cs?name=snippet3)]

この方法には、コード内でキーの文字列を使用する必要がないという利点もあります。

## <a name="cache"></a>キャッシュ

キャッシュは、データを保存し、取得する効率的な方法です。 アプリでは、キャッシュされた項目の有効期間を制御できます。 詳細については、「<xref:performance/caching/response>」を参照してください。

キャッシュされたデータは、特定の要求、ユーザー、またはセッションに関連付けられていません。 **他のユーザーの要求によって取得される可能性があるので、ユーザー固有データをキャッシュしないでください。**

アプリケーション全体のデータをキャッシュするには、「<xref:performance/caching/memory>」を参照してください。

## <a name="common-errors"></a>一般的なエラー

* "'Microsoft.AspNetCore.Session.DistributedSessionStore' を起動しようとしましたが、型 'Microsoft.Extensions.Caching.Distributed.IDistributedCache' のサービスを解決できません。"

  これは通常、少なくとも 1 つの `IDistributedCache` 実装で構成に失敗したことで発生します。 詳細については、次のトピックを参照してください。 <xref:performance/caching/distributed> および <xref:performance/caching/memory>

セッション ミドルウェアがセッションを永続化できない場合:

* ミドルウェアは例外をログに記録し、要求は普通に続行されます。
* これにより、予期しない動作が発生します。

バッキング ストアを利用できない場合、セッション ミドルウェアがセッションを永続化できないことがあります。 たとえば、ユーザーがセッションでショッピング カートを格納します。 ユーザーはアイテムをカートに追加しますが、コミットが失敗します。 アプリはこの失敗を認識しないので、アイテムがカートに追加されたことをユーザーに伝えますが、これは正しくありません。

エラーを確認するための推奨される方法は、アプリがセッションへの書き込みを終了したら、`await feature.Session.CommitAsync` を呼び出すことです。 バッキング ストアが利用できない場合、<xref:Microsoft.AspNetCore.Http.ISession.CommitAsync*> は例外をスローします。 `CommitAsync` が失敗した場合、アプリは例外を処理できます。 <xref:Microsoft.AspNetCore.Http.ISession.LoadAsync*> は、データ ストアが利用できない場合に同じ条件でスローします。
  
## <a name="signalr-and-session-state"></a>SignalR とセッション状態

SignalR アプリでは、セッション状態を使用して情報を格納することはできません。 SignalR アプリは、ハブ内の `Context.Items` に接続ごとの状態を格納できます。 <!-- https://github.com/aspnet/SignalR/issues/2139 -->

## <a name="additional-resources"></a>その他の技術情報

<xref:host-and-deploy/web-farm>
::: moniker-end

::: moniker range="< aspnetcore-3.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)、[Steve Smith](https://ardalis.com/)、[Diana LaRose](https://github.com/DianaLaRose)、[Luke Latham](https://github.com/guardrex)

HTTP はステートレス プロトコルです。 手順を追加しないと、HTTP 要求は独立したメッセージであり、ユーザーの値やアプリの状態は保持されません。 この記事では、要求と要求の間でユーザー データとアプリの状態を保持するための複数の方法について説明します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/app-state/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="state-management"></a>状態管理

状態は、いくつかの方法で格納することができます。 このトピックではそれぞれの方法について説明します。

| 格納の方法 | 格納のメカニズム |
| ---------------- | ----------------- |
| [Cookie ](#cookies) | HTTP cookie (サーバー側アプリ コードを使って格納されたデータを含めることができます) |
| [セッション状態](#session-state) | HTTP cookie とサーバー側アプリ コード |
| [TempData](#tempdata) | HTTP cookie またはセッション状態 |
| [クエリ文字列](#query-strings) | HTTP クエリ文字列 |
| [非表示フィールド](#hidden-fields) | HTTP フォーム フィールド |
| [HttpContext.Items](#httpcontextitems) | サーバー側アプリ コード |
| [キャッシュ](#cache) | サーバー側アプリ コード |
| [依存性の注入](#dependency-injection) | サーバー側アプリ コード |

## <a name="cookies"></a>Cookies

Cookie には、要求と要求の間でデータが格納されます。 cookie は要求ごとに送信されるために、そのサイズは最小に抑える必要があります。 理想的には、識別子だけを cookie に格納し、データはアプリで格納します。 ほとんどのブラウザーで cookie のサイズは 4096 バイトに制限されています。 ドメインごとに使用できる cookie の数も制限されています。

cookie は改ざんされる可能性があるため、アプリで検証する必要があります。 Cookie は、ユーザーが削除でき、クライアント上で期限切れになります。 ただし、cookie は一般に、クライアントでデータを永続化するときに最も持続性のある形式です。

Cookie は、多くの場合、パーソナル化に利用されます。既知のユーザーのためにコンテンツをカスタマイズします。 ユーザーは識別されるだけであり、ほとんどの場合は認証されません。 cookie には、ユーザーの名前、アカウント名、または一意のユーザー ID (GUID など) を格納できます。 その後は、優先される Web サイトの背景色など、ユーザーの個人用設定に cookie を使ってアクセスできます。

cookie を発行し、プライバシーの問題を扱うときは、[欧州連合の一般データ保護規則 (GDPR)](https://ec.europa.eu/info/law/law-topic/data-protection) に留意してください。 詳細については、「[General Data Protection Regulation (GDPR) support in ASP.NET Core](xref:security/gdpr)」(ASP.NET Core での一般データ保護規則 (GDPR) のサポート) をご覧ください。

## <a name="session-state"></a>セッション状態

セッション状態は、ユーザーが Web アプリを参照している期間中ユーザー データを格納するための ASP.NET Core のシナリオです。 セッション状態では、アプリで管理されているストアを使用して、クライアントからの要求間でデータを保持します。 セッション データはキャッシュによってバックアップされ、一時的なデータと見なされます。セッション データがなくても、サイトは機能し続けられる必要があります。 重要なアプリケーションのデータはユーザー データベースに格納し、パフォーマンスの最適化としてのみセッションでキャッシュする必要があります。

> [!NOTE]
> [SignalR ハブ](xref:signalr/hubs)は HTTP コンテキストとは独立して実行する可能性があるため、[SignalR](xref:signalr/index) アプリではセッションはサポートされていません。 このようなことは、たとえば、長いポーリング要求が HTTP コンテキストの有効期間を超えてハブによって開かれている場合に発生する可能性があります。

ASP.NET Core は、セッション ID を含み、要求ごとに各アプリに送信される cookie がクライアントに提供されて、セッションの状態が維持されます。 アプリは、セッション ID を使用してセッション データをフェッチします。

セッション状態は次の動作を示します。

* セッション cookie はブラウザーに固有であるため、セッションはブラウザー間では共有されません。
* セッション cookie は、ブラウザー セッションが終了するときに削除されます。
* cookie を受け取り、セッションが期限切れになった場合、同じセッション cookie を使用する新しいセッションが作成されます。
* 空のセッションは保持されません。要求と要求の間でセッションを維持するには、少なくとも 1 つの値をセッションが持っている必要があります。 セッションが保持されないと、新しい要求ごとに新しいセッション ID が生成されます。
* アプリは、最後の要求から限られた時間だけセッションを維持します。 アプリでは、セッション タイムアウトを設定するか、既定値の 20 分を使用します。 セッション状態は、特定のセッションに固有であるが、セッション間で永続的に保持する必要のないユーザー データの格納に最適です。
* セッション データは、<xref:Microsoft.AspNetCore.Http.ISession.Clear%2A?displayProperty=nameWithType> の実装が呼び出されるか、セッションが期限切れになると、削除されます。
* クライアント ブラウザーが閉じられたこと、またはクライアントでセッション cookie が削除されるか期限切れになったことを、アプリ コードに通知する既定のメカニズムはありません。
* ASP.NET Core MVC と Razor ページのテンプレートには、一般データ保護規制 (GDPR) のサポートが含まれます。 セッション状態の cookie は既定では必須になっていません。このため、サイトの訪問者が追跡を許可しない限り、セッション状態は機能しません。 詳細については、「<xref:security/gdpr#tempdata-provider-and-session-state-cookies-arent-essential>」を参照してください。

> [!WARNING]
> セッション状態には機密データを保存しないでください。 ユーザーがブラウザーを閉じず、セッション cookie がクリアされない可能性があります。 一部のブラウザーでは、ブラウザーのウィンドウ間で有効なセッションの cookie が維持されます。 セッションが 1 人のユーザーに制限されず&mdash;次のユーザーが同じセッション cookie でアプリの閲覧を続けることがあります。

メモリ内キャッシュ プロバイダーは、アプリが存在するサーバーのメモリにセッション データを格納します。 サーバー ファームのシナリオでは次のようになります。

* "*固定セッション*" を使用して、個々のサーバー上の特定のアプリのインスタンスに、各セッションを結び付けます。 [Azure App Service](https://azure.microsoft.com/services/app-service/) は[アプリケーション要求ルーティング処理 (ARR)](/iis/extensions/planning-for-arr/using-the-application-request-routing-module) を使って、既定で固定セッションを強制的に使用します。 ただし、固定セッションは拡張性に影響を与え、Web アプリの更新を複雑にすることがあります。 もっとよい方法は、Redis または SQL Server の分散キャッシュを使用することで、固定セッションを必要としません。 詳細については、「<xref:performance/caching/distributed>」を参照してください。
* セッション cookie は <xref:Microsoft.AspNetCore.DataProtection.IDataProtector> によって暗号化されます。 各コンピューターでセッション cookie が読み取られるようにするには、データ保護を適切に構成する必要があります。 詳細については、<xref:security/data-protection/introduction> および[キー ストレージ プロバイダー](xref:security/data-protection/implementation/key-storage-providers)に関する記事をご覧ください。

### <a name="configure-session-state"></a>セッション状態を構成する

[Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) に含まれる [Microsoft.AspNetCore.Session](https://www.nuget.org/packages/Microsoft.AspNetCore.Session/) パッケージは、セッション状態を管理するためのミドルウェアを提供します。 セッション ミドルウェアを有効にするには、`Startup` に次が含まれている必要があります。

* いずれかの <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> メモリ キャッシュ。 `IDistributedCache` 実装はセッションのバックアップ ストアとして利用されます。 詳細については、「<xref:performance/caching/distributed>」を参照してください。
* `ConfigureServices` での <xref:Microsoft.Extensions.DependencyInjection.SessionServiceCollectionExtensions.AddSession%2A> の呼び出し。
* `Configure` での <xref:Microsoft.AspNetCore.Builder.SessionMiddlewareExtensions.UseSession%2A> の呼び出し。

次のコードでは、`IDistributedCache` の既定のメモリ内実装でメモリ内セッション プロバイダーを設定する方法を示します。

[!code-csharp[](app-state/samples/2.x/SessionSample/Startup.cs?name=snippet1&highlight=5-14,34)]

ミドルウェアの順序が重要です。 上の例では、`UseMvc` の後で `UseSession` を呼び出すと、`InvalidOperationException` 例外が発生します。 詳細については、[ミドルウェアの順序](xref:fundamentals/middleware/index#order)に関するページをご覧ください。

<xref:Microsoft.AspNetCore.Http.HttpContext.Session?displayProperty=nameWithType> は、セッション状態を構成した後に使用できます。

`UseSession` を呼び出前に `HttpContext.Session` にアクセスすることはできません。

アプリが応答ストリームへの書き込みを開始した後では、新しいセッション cookie を含む新しいセッションを作成できません。 例外は Web サーバー ログに記録され、ブラウザーには表示されません。

### <a name="load-session-state-asynchronously"></a>セッション状態を非同期的に読み込む

ASP.NET Core の既定のセッション プロバイダーでは、<xref:Microsoft.AspNetCore.Http.ISession.TryGetValue%2A>、<xref:Microsoft.AspNetCore.Http.ISession.Set%2A>、または <xref:Microsoft.AspNetCore.Http.ISession.Remove%2A> メソッドの前に <xref:Microsoft.AspNetCore.Http.ISession.LoadAsync%2A?displayProperty=nameWithType> メソッドが明示的に呼び出された場合にのみ、基になる <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> バッキング ストアから非同期的にセッション レコードを読み込みます。 `LoadAsync` を最初に呼び出さないと、基になっているセッション レコードは同期的に読み込まれ、パフォーマンスが大幅に低下する可能性があります。

アプリにこのパターンを強制させるには、`TryGetValue`、`Set`、または `Remove` の前に `LoadAsync` メソッドが呼び出されない場合に例外をスローするバージョンで、<xref:Microsoft.AspNetCore.Session.DistributedSessionStore> および <xref:Microsoft.AspNetCore.Session.DistributedSession> の実装をラップします。 ラップしたバージョンをサービス コンテナーに登録します。

### <a name="session-options"></a>セッション オプション

セッションの既定値をオーバーライドするには、<xref:Microsoft.AspNetCore.Builder.SessionOptions> を使用します。

| オプション | 説明 |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.Builder.SessionOptions.Cookie> | cookie の作成に使用される設定を決定します。 <xref:Microsoft.AspNetCore.Http.CookieBuilder.Name> の既定値は <xref:Microsoft.AspNetCore.Session.SessionDefaults.CookieName?displayProperty=nameWithType> (`.AspNetCore.Session`) です。 <xref:Microsoft.AspNetCore.Http.CookieBuilder.Path> の既定値は <xref:Microsoft.AspNetCore.Session.SessionDefaults.CookiePath?displayProperty=nameWithType> (`/`) です。 <xref:Microsoft.AspNetCore.Http.CookieBuilder.SameSite> の既定値は <xref:Microsoft.AspNetCore.Http.SameSiteMode.Lax?displayProperty=nameWithType> (`1`) です。 <xref:Microsoft.AspNetCore.Http.CookieBuilder.HttpOnly> では、既定値が `true` に設定されます。 <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> では、既定値が `false` に設定されます。 |
| <xref:Microsoft.AspNetCore.Builder.SessionOptions.IdleTimeout> | `IdleTimeout` は、内容を破棄されることなくセッションがアイドル状態になっていることのできる最大時間を示します。 セッションへのアクセスがあるたびに、タイムアウトはリセットされます。 この設定はセッションの内容にのみ適用され、cookie には適用されません。 既定値は 20 分です。 |
| <xref:Microsoft.AspNetCore.Builder.SessionOptions.IOTimeout> | ストアからのセッションの読み込み、またはストアに戻すコミットに対して許容される最大時間です。 この設定は非同期操作にのみ適用できます。 このタイムアウトは、<xref:System.Threading.Timeout.InfiniteTimeSpan> を使用して無効にすることができます。 既定値は 1 分です。 |

セッションは cookie を利用し、1 つのブラウザーからの要求を追跡し、識別します。 既定では、この cookie は `.AspNetCore.Session` という名前であり、パス `/` が使用されます。 cookie の既定値ではドメインが指定されないため、ページのクライアント側スクリプトには使用できません (<xref:Microsoft.AspNetCore.Http.CookieBuilder.HttpOnly> の既定値が `true` になるため)。

cookie セッションの既定値をオーバーライドするには、`SessionOptions` を使用します。

[!code-csharp[](app-state/samples_snapshot/2.x/SessionSample/Startup.cs?name=snippet1&highlight=14-19)]

アプリでは、<xref:Microsoft.AspNetCore.Builder.SessionOptions.IdleTimeout> プロパティを使用して、サーバーのキャッシュ内の内容が破棄されるまでのセッションのアイドル時間が決定されます。 このプロパティは cookie の有効期限に依存しません。 要求が[セッション ミドルウェア](xref:Microsoft.AspNetCore.Session.SessionMiddleware)を通過するたびにタイムアウトがリセットされます。

セッション状態は "*ロックなし*" です。 2 つの要求がセッションの内容を同時に変更しようとした場合、最後の要求が最初の要求をオーバーライドします。 `Session` は *一貫性のあるセッション* として実装されます。つまり、コンテンツは全部まとめて保管されます。 2 つの要求が異なるセッション値を変更しようとしたとき、最後の要求が最初の要求によって行われたセッションの変更をオーバーライドすることがあります。

### <a name="set-and-get-session-values"></a>セッション値の設定および取得

セッション状態にアクセスするには、Razor Pages の <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> クラスか、MVC の <xref:Microsoft.AspNetCore.Mvc.Controller> クラスを <xref:Microsoft.AspNetCore.Http.HttpContext.Session?displayProperty=nameWithType> と共に使用します。 このプロパティは <xref:Microsoft.AspNetCore.Http.ISession> の実装です。

`ISession` の実装では、整数値や文字列値を設定および取得するための複数の拡張メソッドが提供されています。 [Microsoft.AspNetCore.Http.Extensions](https://www.nuget.org/packages/Microsoft.AspNetCore.Http.Extensions/) パッケージがプロジェクトによって参照されている場合、拡張メソッドは <xref:Microsoft.AspNetCore.Http> 名前空間内にあります (拡張メソッドにアクセスするには、`using Microsoft.AspNetCore.Http;` ステートメントを追加します)。 どちらのパッケージも、[Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)に含まれます。

`ISession` 拡張メソッド:

* [Get(ISession, String)](xref:Microsoft.AspNetCore.Http.SessionExtensions.Get%2A)
* [GetInt32(ISession, String)](xref:Microsoft.AspNetCore.Http.SessionExtensions.GetInt32%2A)
* [GetString(ISession, String)](xref:Microsoft.AspNetCore.Http.SessionExtensions.GetString%2A)
* [SetInt32(ISession, String, Int32)](xref:Microsoft.AspNetCore.Http.SessionExtensions.SetInt32%2A)
* [SetString(ISession, String, String)](xref:Microsoft.AspNetCore.Http.SessionExtensions.SetString%2A)

次の例では、`IndexModel.SessionKeyName` キー (サンプル アプリ内の `_Name`) のセッション値を Razor Pages ページで取得します。

```csharp
@page
@using Microsoft.AspNetCore.Http
@model IndexModel

...

Name: @HttpContext.Session.GetString(IndexModel.SessionKeyName)
```

次の例では、整数および文字列を設定および取得する方法を示します。

[!code-csharp[](app-state/samples/2.x/SessionSample/Pages/Index.cshtml.cs?name=snippet1&highlight=18-19,22-23)]

分散キャッシュのシナリオを有効にするには、メモリ内キャッシュを使用する場合であっても、すべてのセッション データをシリアル化する必要があります。 文字列と整数のシリアライザーは、<xref:Microsoft.AspNetCore.Http.ISession>) の拡張メソッドによって提供されます。 複合型は、JSON などの別のメカニズムを使ってユーザーがシリアル化する必要があります。

シリアル化可能なオブジェクトを設定および取得するには、次の拡張メソッドを追加します。

[!code-csharp[](app-state/samples/2.x/SessionSample/Extensions/SessionExtensions.cs?name=snippet1)]

次の例では、シリアル化可能オブジェクトを拡張メソッドで設定および取得する方法を示します。

[!code-csharp[](app-state/samples/2.x/SessionSample/Pages/Index.cshtml.cs?name=snippet2)]

## <a name="tempdata"></a>TempData

ASP.NET Core によって、Razor Pages [TempData](xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel.TempData) または Controller <xref:Microsoft.AspNetCore.Mvc.Controller.TempData> が発行されます。 このプロパティには、別の要求で読み取られるまでデータが格納されます。 [Keep (String)](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ITempDataDictionary.Keep*) メソッドと [Peek (String)](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ITempDataDictionary.Peek*) メソッドを使用すると、要求の最後に削除せずにデータを調べることができます。 [Keep()](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ITempDataDictionary.Keep*) によって、リテンション期間についてディクショナリ内のすべての項目がマークされます。 `TempData` は特に、複数の要求にデータが必要な場合のリダイレクトに役立ちます。 `TempData` は、`TempData` プロバイダーによって cookie またはセッション状態を使用して実装されます。

## <a name="tempdata-samples"></a>TempData のサンプル

顧客を作成する次のページについて考えてみましょう。

[!code-csharp[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet&highlight=15-16,30)]

次のページに `TempData["Message"]` が表示されます。

[!code-cshtml[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/IndexPeek.cshtml?range=1-14)]

前のマークアップでは、`Peek` が使用されているため、要求の最後に `TempData["Message"]` が削除 **されません**。 ページを更新すると `TempData["Message"]` が表示されます。

次のマークアップは前のコードと似ていますが、`Keep` を使用して要求の最後にデータを保存しています。

[!code-cshtml[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/IndexKeep.cshtml?range=1-14)]

*Indexpeek* ページと *IndexKeep* ページ間を移動しても `TempData["Message"]` は削除されません。

次のコードでは `TempData["Message"]` が表示されますが、要求の最後に `TempData["Message"]` が削除されます。

[!code-cshtml[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/Index.cshtml?range=1-14)]

### <a name="tempdata-providers"></a>TempData プロバイダー

cookie に TempData を格納するには、既定では、cookie ベースの TempData プロバイダーが使われます。

cookie データは、<xref:Microsoft.AspNetCore.DataProtection.IDataProtector> を使用して暗号化され、<xref:Microsoft.AspNetCore.WebUtilities.Base64UrlTextEncoder> でエンコードされた後、チャンクされます。 cookie はチャンクされるため、ASP.NET Core 1.x の 1 cookie のサイズ上限は適用されません。 暗号化されているデータを圧縮すると、[CRIME](https://wikipedia.org/wiki/CRIME_(security_exploit)) 攻撃や [BREACH](https://wikipedia.org/wiki/BREACH_(security_exploit)) 攻撃など、セキュリティ上の問題を起す可能性があるため、cookie データは圧縮されません。 cookie ベース TempData プロバイダーの詳細については、<xref:Microsoft.AspNetCore.Mvc.ViewFeatures.CookieTempDataProvider> に関する記事を参照してください。

### <a name="choose-a-tempdata-provider"></a>TempData プロバイダーを選択する

TempData プロバイダーを選択するときの考慮事項:

1. アプリは既にセッション状態を使っているかどうか。 使っている場合、(データのサイズを除き) セッション状態 TempData プロバイダーがそのアプリにコストを追加することはありません。
2. アプリでは、比較的少量のデータに対して (最大 500 バイト) TempData がわずかばかり使用されているか。 該当する場合、cookie TempData プロバイダーによって TempData を送信する要求ごとに少額のコストが追加されます。 該当しない場合、セッション状態 TempData プロバイダーは便利かもしれません。TempData が尽きるまで、要求のたびに大量のデータをラウンドトリップすることが回避されます。
3. アプリは複数サーバーのサーバー ファームで実行しているか。 そうである場合は、データ保護の外部で cookie TempData プロバイダーを使用するために、追加の構成は必要ありません (「<xref:security/data-protection/introduction>」および[キー ストレージ プロバイダー](xref:security/data-protection/implementation/key-storage-providers)に関する記事を参照)。

> [!NOTE]
> Web ブラウザーなどのほとんどの Web クライアントによって、各 cookie の最大サイズ、cookie の合計数、またはその両方に上限が設けられます。 cookie TempData プロバイダーを使用するとき、アプリでそれらの上限が超えないことを確認してください。 データの合計サイズを考慮してください。 暗号化とチャンクによる cookie のサイズの増加を考慮してください。

### <a name="configure-the-tempdata-provider"></a>TempData プロバイダーを構成する

cookie ベース TempData プロバイダーは既定で有効になります。

セッション ベースの TempData プロバイダーを有効にするには、<xref:Microsoft.Extensions.DependencyInjection.MvcViewFeaturesMvcBuilderExtensions.AddSessionStateTempDataProvider%2A> 拡張メソッドを使います。

[!code-csharp[](app-state/samples_snapshot_2/2.x/SessionSample/Startup.cs?name=snippet1&highlight=11,13,32)]

ミドルウェアの順序が重要です。 上の例では、`UseMvc` の後で `UseSession` を呼び出すと、`InvalidOperationException` 例外が発生します。 詳細については、[ミドルウェアの順序](xref:fundamentals/middleware/index#order)に関するページをご覧ください。

> [!IMPORTANT]
> .NET Framework が対象で、セッション ベースの TempData プロバイダーを使う場合は、[Microsoft.AspNetCore.Session](https://www.nuget.org/packages/Microsoft.AspNetCore.Session/) パッケージをプロジェクトに追加します。

## <a name="query-strings"></a>クエリ文字列

限られた量のデータを要求間で渡すことができます。新しい要求のクエリ文字列にそれを追加します。 これは、リンクと埋め込まれた状態がメールまたはソーシャル ネットワークを通して共有されるよう、永久的に状態をキャプチャするのに役立ちます。 URL クエリ文字列はパブリックであるため、機密データにはクエリ文字列を使わないでください。

意図しない共有が発生するだけでなく、クエリ文字列にデータを含めると、[クロスサイト リクエスト フォージェリ (CSRF)](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)) 攻撃の機会を与えてしまいます。認証中、ユーザーをだまして悪意のあるサイトに誘導します。 攻撃者はアプリからユーザー データを盗んだり、ユーザーになりすまして悪意のある行為を行ったりできます。 保存されるアプリまたはセッション状態は CSRF 攻撃を防ぐ必要があります。 詳細については、「<xref:security/anti-request-forgery>」を参照してください。

## <a name="hidden-fields"></a>非表示フィールド

データは非表示フォーム フィールドに保存したり、次の要求で転記したりできます。 複数ページ フォームでは、これは一般的です。 クライアントがデータを改ざんする可能性があるため、アプリで非表示フィールドに格納されているデータを常に再検証する必要があります。

## <a name="httpcontextitems"></a>HttpContext.Items

1 つの要求を処理している間にデータを格納するには、<xref:Microsoft.AspNetCore.Http.HttpContext.Items?displayProperty=nameWithType> コレクションを使います。 要求が処理された後、コレクションの内容は破棄されます。 `Items` コレクションは、コンポーネントまたはミドルウェアが要求の間の異なる時点で動作し、パラメーターを受け渡す直接的な方法がない場合に、コンポーネントとミドルウェアが通信できるようにするためによく使われます。

次の例では、[ミドルウェア](xref:fundamentals/middleware/index)により `isVerified` が `Items` コレクションに追加されます。

```csharp
app.Use(async (context, next) =>
{
    // perform some verification
    context.Items["isVerified"] = true;
    await next.Invoke();
});
```

後のパイプラインで、別のミドルウェアが `isVerified` の値にアクセスできます。

```csharp
app.Run(async (context) =>
{
    await context.Response.WriteAsync($"Verified: {context.Items["isVerified"]}");
});
```

1 つのアプリでのみ使用されるミドルウェアの場合、`string` キーが許容されます。 アプリのインスタンス間で共有されるミドルウェアでは、キーの競合を避けるため、一意のオブジェクト キーを使う必要があります。 次の例では、ミドルウェア クラスで定義されている一意のオブジェクト キーを使用する方法を示します。

[!code-csharp[](app-state/samples/2.x/SessionSample/Middleware/HttpContextItemsMiddleware.cs?name=snippet1&highlight=4,13)]

その他のコードは、ミドルウェア クラスで公開されるキーを利用し、`HttpContext.Items` に格納されている値にアクセスできます。

[!code-csharp[](app-state/samples/2.x/SessionSample/Pages/Index.cshtml.cs?name=snippet3)]

この方法には、コード内でキーの文字列を使用する必要がないという利点もあります。

## <a name="cache"></a>キャッシュ

キャッシュは、データを保存し、取得する効率的な方法です。 アプリでは、キャッシュされた項目の有効期間を制御できます。

キャッシュされたデータは、特定の要求、ユーザー、またはセッションに関連付けられていません。 **他のユーザーの要求によって取得される可能性があるので、ユーザー固有データをキャッシュしないように注意してください。**

詳細については、「<xref:performance/caching/response>」を参照してください。

## <a name="dependency-injection"></a>依存関係の挿入

[依存関係の注入](xref:fundamentals/dependency-injection)を利用し、すべてのユーザーがデータを利用できるようにします。

1. データを含むサービスを定義します。 たとえば、`MyAppData` という名前のクラスを定義します。

    ```csharp
    public class MyAppData
    {
        // Declare properties and methods
    }
    ```

2. サービス クラスを `Startup.ConfigureServices` に追加します。

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddSingleton<MyAppData>();
    }
    ```

3. データ サービス クラスを使用します。

    ```csharp
    public class IndexModel : PageModel
    {
        public IndexModel(MyAppData myService)
        {
            // Do something with the service
            //    Examples: Read data, store in a field or property
        }
    }
    ```

## <a name="common-errors"></a>一般的なエラー

* "'Microsoft.AspNetCore.Session.DistributedSessionStore' を起動しようとしましたが、型 'Microsoft.Extensions.Caching.Distributed.IDistributedCache' のサービスを解決できません。"

  これは通常、少なくとも 1 つの `IDistributedCache` 実装で構成に失敗したことで発生します。 詳細については、次のトピックを参照してください。 <xref:performance/caching/distributed> および <xref:performance/caching/memory>

* セッション ミドルウェアがセッションを永続化できなかった場合 (バッキング ストアを利用できない場合など)、ミドルウェアは例外をログに記録し、要求は普通に続行されます。 これにより、予期しない動作が発生します。

  たとえば、ユーザーがセッションでショッピング カートを格納します。 ユーザーはアイテムをカートに追加しますが、コミットが失敗します。 アプリはこの失敗を認識しないので、アイテムがカートに追加されたことをユーザーに伝えますが、これは正しくありません。

  エラーを確認するための推奨される方法は、アプリがセッションへの書き込みを終了したら、アプリ コードから `await feature.Session.CommitAsync();` を呼び出すことです。 バッキング ストアが利用できない場合、`CommitAsync` は例外をスローします。 `CommitAsync` が失敗した場合、アプリは例外を処理できます。 `LoadAsync` は、データ ストアが利用できない場合に同じ条件で例外をスローします。
  
## <a name="signalr-and-session-state"></a>SignalR とセッション状態

SignalR アプリでは、セッション状態を使用して情報を格納することはできません。 SignalR アプリは、ハブ内の `Context.Items` に接続ごとの状態を格納できます。 <!-- https://github.com/aspnet/SignalR/issues/2139 -->

## <a name="additional-resources"></a>その他の技術情報

<xref:host-and-deploy/web-farm>
::: moniker-end
