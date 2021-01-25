---
title: ASP.NET Core のルーティング
author: rick-anderson
description: ASP.NET Core ルーティングによって、HTTP 要求の照合と実行可能なエンドポイントへのディスパッチがどのように行われるかについて説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 4/1/2020
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
uid: fundamentals/routing
ms.openlocfilehash: 1355fdaeae58b6f4e0cf8d41a74b1c28aee0e8fe
ms.sourcegitcommit: 063a06b644d3ade3c15ce00e72a758ec1187dd06
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98253086"
---
# <a name="routing-in-aspnet-core"></a>ASP.NET Core のルーティング

作成者: [Ryan Nowak](https://github.com/rynowak)、[Kirk Larkin](https://twitter.com/serpent5)、[Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

ルーティングの役割は、受信した HTTP 要求を照合し、それらの要求をアプリの実行可能なエンドポイントにディスパッチすることです。 [エンドポイント](#endpoint)は、アプリの実行可能な要求処理コードの単位です。 エンドポイントはアプリで定義され、アプリの起動時に構成されます。 エンドポイントの照合プロセスでは、要求の URL から値を抽出し、それらの値を要求の処理に提供できます。 アプリからのルート情報を使用して、ルーティングでエンドポイントにマップする URL を生成することもできます。

アプリでは、次のものを使用してルーティングを構成できます。

- Controllers
- Razor Pages
- SignalR
- gRPC サービス
- エンドポイント対応の[ミドルウェア](xref:fundamentals/middleware/index) ([正常性チェックなど)](xref:host-and-deploy/health-checks)。
- ルーティングに登録されているデリゲートとラムダ。

このドキュメントでは、ASP.NET Core のルーティングについて詳しく説明します。 ルーティングの構成については、以下を参照してください。

* コントローラーについては、<xref:mvc/controllers/routing> を参照してください。
* Razor Pages の規則については、「<xref:razor-pages/razor-pages-conventions>」を参照してください。

このドキュメントで説明されているエンドポイント ルーティング システムは、ASP.NET Core 3.0 以降に適用されます。 <xref:Microsoft.AspNetCore.Routing.IRouter> に基づく以前のルーティング システムについては、次のいずれかの方法を使用して ASP.NET Core 2.1 バージョンを選択してください。

* 以前のバージョンのバージョン セレクター。
* [ASP.NET Core 2.1 でのルーティング](?view=aspnetcore-2.1)に関する記事を選択します。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/routing/samples/3.x)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

このドキュメントのダウンロード サンプルは、特定の `Startup` クラスによって有効になります。 特定のサンプルを実行するには、目的の `Startup` クラスを呼び出すように *Program.cs* を変更します。

## <a name="routing-basics"></a>ルーティングの基本

すべての ASP.NET Core テンプレートには、生成されたコードでのルーティングが含まれます。 ルーティングは、`Startup.Configure` で[ミドルウェア](xref:fundamentals/middleware/index) パイプラインに登録されます。

次のコードでは、ルーティングの基本的な例を示します。

[!code-csharp[](routing/samples/3.x/RoutingSample/Startup.cs?name=snippet&highlight=8,10)]

ルーティングでは、<xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseRouting*> と <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> によって登録されたミドルウェアのペアを使用します。

* `UseRouting` では、ルートの照合がミドルウェア パイプラインに追加されます。 このミドルウェアによって、アプリで定義されているエンドポイントのセットが調べられ、要求に基づいて[最適な一致](#urlm)が選択されます。
* `UseEndpoints` では、エンドポイントの実行がミドルウェア パイプラインに追加されます。 選択されたエンドポイントに関連付けられているデリゲートが実行されます。

前の例には、[MapGet](xref:Microsoft.AspNetCore.Builder.EndpointRouteBuilderExtensions.MapGet*) メソッドを使用する "*コードへのルーティング*" エンドポイントが 1 つ含まれます。

* HTTP `GET` 要求がルート URL `/` に送信された場合:
  * 示されている要求デリゲートが実行されます。
  * `Hello World!` が HTTP 応答に書き込まれます。 既定では、ルート URL `/` は `https://localhost:5001/` です。
* 要求メソッドが `GET` ではない場合、またはルート URL が `/` ではない場合は、一致するルートはなく、HTTP 404 が返されます。

### <a name="endpoint"></a>エンドポイント

<a name="endpoint"></a>

**エンドポイント** を定義するには、`MapGet` メソッドが使用されます。 エンドポイントとは、次のようなものです。

* URL と HTTP メソッドを一致させることによって選択できます。
* デリゲートを実行することによって実行できます。

アプリによって一致させて実行できるエンドポイントは、`UseEndpoints` で構成します。 たとえば、<xref:Microsoft.AspNetCore.Builder.EndpointRouteBuilderExtensions.MapGet*>、<xref:Microsoft.AspNetCore.Builder.EndpointRouteBuilderExtensions.MapPost*>、および[類似のメソッド](xref:Microsoft.AspNetCore.Builder.EndpointRouteBuilderExtensions)では、要求のデリゲートがルーティング システムに接続されます。
他のメソッドを使用して、ASP.NET Core フレームワークの機能をルーティング システムに接続できます。
- [Razor Pages 用の MapRazorPages](xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapRazorPages*)
- [コントローラー用の MapControllers](xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllers*)
- [SignalR 用の MapHub\<THub>](xref:Microsoft.AspNetCore.SignalR.HubRouteBuilder.MapHub*) 
- [gRPC 用の MapGrpcService\<TService>](xref:grpc/aspnetcore)

次の例では、より高度なルート テンプレートによるルーティングを示します。

[!code-csharp[](routing/samples/3.x/RoutingSample/RouteTemplateStartup.cs?name=snippet)]

文字列 `/hello/{name:alpha}` は、**ルート テンプレート** です。 これは、エンドポイントの一致方法を構成するために使用されます。 この場合、テンプレートは次のものと一致します。

* `/hello/Ryan` のような URL
* `/hello/` で始まり、その後に一連の英字が続く任意の URL パス。  `:alpha` では、英字のみと一致するルート制約が適用されます。 [ルート制約](#route-constraint-reference)については、このドキュメントで後ほど説明します。

URL パスの 2 番目のセグメント `{name:alpha}` は次のようになります。

* `name` パラメーターにバインドされます。
* キャプチャされて [HttpRequest.RouteValues](xref:Microsoft.AspNetCore.Http.HttpRequest.RouteValues*) に格納されます。

このドキュメントで説明されているエンドポイント ルーティング システムは、ASP.NET Core 3.0 での新機能です。 ただし、ASP.NET Core のすべてのバージョンで、同じルート テンプレート機能とルート制約のセットがサポートされます。

次の例では、[正常性チェック](xref:host-and-deploy/health-checks)と承認を使用するルーティングを示します。

[!code-csharp[](routing/samples/3.x/RoutingSample/AuthorizationStartup.cs?name=snippet)]

[!INCLUDE[request localized comments](~/includes/code-comments-loc.md)]

上の例では、次の方法が示されています。

* ルーティングで承認ミドルウェアを使用できます。
* エンドポイントを使用して、承認動作を構成できます。

<xref:Microsoft.AspNetCore.Builder.HealthCheckEndpointRouteBuilderExtensions.MapHealthChecks*> の呼び出しにより、正常性チェック エンドポイントが追加されます。 この呼び出しに <xref:Microsoft.AspNetCore.Builder.AuthorizationEndpointConventionBuilderExtensions.RequireAuthorization*> をチェーンすると、エンドポイントに承認ポリシーがアタッチされます。

<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*> と <xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization*> を呼び出すと、認証ミドルウェアと承認ミドルウェアが追加されます。 これらのミドルウェアは、次のことができるように <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseRouting*> と `UseEndpoints` の間に配置されます。

* `UseRouting` によって選択されたエンドポイントを確認します。
* <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> によってエンドポイントにディスパッチされる前に、承認ポリシーを適用します。

<a name="metadata"></a>

### <a name="endpoint-metadata"></a>エンドポイントのメタデータ

前の例には 2 つのエンドポイントがありますが、承認ポリシーがアタッチされているのは正常性チェック エンドポイントだけです。 要求が正常性チェック エンドポイント `/healthz` と一致した場合、承認チェックが実行されます。 これは、エンドポイントに追加のデータをアタッチできることを示しています。 この追加データは、エンドポイントの **メタデータ** と呼ばれます。

* メタデータは、ルーティング対応ミドルウェアによって処理できます。
* メタデータには、任意の .NET 型を使用できます。

## <a name="routing-concepts"></a>ルーティングの概念

ルーティング システムは、ミドルウェア パイプラインを基にして、強力な **エンドポイント** 概念を追加することにより、構築されています。 エンドポイントは、ルーティング、承認、および任意の数の ASP.NET Core システムに関して相互に独立している、アプリの機能の単位を表します。

<a name="endpoint"></a>

### <a name="aspnet-core-endpoint-definition"></a>ASP.NET Core エンドポイントの定義

ASP.NET Core エンドポイントとは次のようなものです。

* 実行可能: <xref:Microsoft.AspNetCore.Http.Endpoint.RequestDelegate> を持っています。
* 拡張可能: [Metadata](xref:Microsoft.AspNetCore.Http.Endpoint.Metadata*) コレクションを持っています。
* Selectable: 必要に応じて、[ルーティング情報](xref:Microsoft.AspNetCore.Routing.RouteEndpoint.RoutePattern*)を持ちます。
* 列挙可能: エンドポイントのコレクションの一覧は、[DI](xref:fundamentals/dependency-injection) から <xref:Microsoft.AspNetCore.Routing.EndpointDataSource> を取得することによって得られます。

次のコードでは、エンドポイントを取得し、現在の要求と一致するものを検査する方法を示します。

[!code-csharp[](routing/samples/3.x/RoutingSample/EndpointInspectorStartup.cs?name=snippet)]

エンドポイントが選択されている場合は、`HttpContext` から取得できます。 そのプロパティを検査できます。 エンドポイント オブジェクトは不変であり、作成後に変更することはできません。 最も一般的なエンドポイントの型は <xref:Microsoft.AspNetCore.Routing.RouteEndpoint> です。 `RouteEndpoint` には、ルーティング システムによる選択を可能にする情報が含まれています。

上のコードでは、[app.Use](xref:Microsoft.AspNetCore.Builder.UseExtensions.Use*) によってインライン [ミドルウェア](xref:fundamentals/middleware/index)が構成されます。

<a name="mt"></a>

次のコードでは、パイプラインで `app.Use` が呼び出される場所によっては、エンドポイントが存在しない場合があることを示します。

[!code-csharp[](routing/samples/3.x/RoutingSample/MiddlewareFlowStartup.cs?name=snippet)]

前のサンプルでは、エンドポイントが選択されているかどうかを表示する `Console.WriteLine` ステートメントが追加されています。 わかりやすくするため、このサンプルでは、指定された `/` エンドポイントに表示名が割り当てられています。

このコードを `/` の URL で実行すると、次のように表示されます。

```txt
1. Endpoint: (null)
2. Endpoint: Hello
3. Endpoint: Hello
```

このコード他の URL で実行すると、次のように表示されます。

```txt
1. Endpoint: (null)
2. Endpoint: (null)
4. Endpoint: (null)
```

この出力は次のことを示しています。

* `UseRouting` が呼び出される前は、エンドポイントは常に null になっています。
* 一致が見つかった場合、エンドポイントは `UseRouting` と <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> の間で null 以外の値になります。
* 一致が見つかると、`UseEndpoints` ミドルウェアは **ターミナル** です。 [ターミナル ミドルウェア](#tm)については、このドキュメントで後ほど定義します。
* `UseEndpoints` の後のミドルウェアは、一致が検出されなかった場合にのみ実行されます。

`UseRouting` ミドルウェアでは、[SetEndpoint](xref:Microsoft.AspNetCore.Http.EndpointHttpContextExtensions.SetEndpoint*) メソッドを使用して、エンドポイントが現在のコンテキストにアタッチされます。 `UseRouting` ミドルウェアをカスタム ロジックに置き換えることができ、その場合でもエンドポイントを使用する利点を得られます。 エンドポイントはミドルウェアのような低レベルのプリミティブであり、ルーティングの実装には結合されません。 ほとんどのアプリでは、`UseRouting` をカスタム ロジックに置き換える必要はありません。

`UseEndpoints` ミドルウェアは、`UseRouting` ミドルウェアと連携して使用するように設計されています。 エンドポイントを実行するためのコア ロジックは複雑ではありません。 <xref:Microsoft.AspNetCore.Http.EndpointHttpContextExtensions.GetEndpoint*> を使用してエンドポイントを取得し、その <xref:Microsoft.AspNetCore.Http.Endpoint.RequestDelegate> プロパティを呼び出します。

次のコードでは、ミドルウェアがルーティングに与える影響またはルーティングに対応する方法を示します。

[!code-csharp[](routing/samples/3.x/RoutingSample/IntegratedMiddlewareStartup.cs?name=snippet)]

前の例では、2 つの重要な概念が示されています。

* ミドルウェアは、`UseRouting` の前に実行して、ルーティングの動作に使用されるデータを変更できます。
    * 通常、ルーティングの前に出現するミドルウェアでは、<xref:Microsoft.AspNetCore.Builder.RewriteBuilderExtensions.UseRewriter*>、<xref:Microsoft.AspNetCore.Builder.HttpMethodOverrideExtensions.UseHttpMethodOverride*>、<xref:Microsoft.AspNetCore.Builder.UsePathBaseExtensions.UsePathBase*> などの要求のプロパティが変更されます。
* ミドルウェアは、`UseRouting` と <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> の間で実行して、エンドポイントが実行される前にルーティングの結果を処理できます。
    * `UseRouting` と `UseEndpoints` の間で実行されるミドルウェア:
      * 通常、メタデータを検査してエンドポイントを認識します。
      * 多くの場合、`UseAuthorization` や `UseCors` によって実行されるセキュリティに関する決定を行います。
    * ミドルウェアとメタデータを組み合わせることで、エンドポイントごとにポリシーを構成できます。

上のコードでは、エンドポイントごとのポリシーをサポートするカスタム ミドルウェアの例が示されています。 ミドルウェアによって、機密データへのアクセスの "*監査ログ*" がコンソールに書き込まれます。 `AuditPolicyAttribute` メタデータを使用して、エンドポイントを "*監査する*" ようにミドルウェアを構成できます。 このサンプルでは、機密としてマークされているエンドポイントのみが監査される "*オプトイン*" パターンを示します。 このロジックの逆を定義して、たとえば安全としてマークされていないすべてのものを監査することができます。 エンドポイント メタデータ システムは柔軟です。 ユース ケースに適したどのような方法でも、このロジックを設計できます。

前のサンプル コードは、エンドポイントの基本的な概念を示すことが意図されています。 **サンプルは運用環境での使用は意図されていません**。 より完全なバージョンの "*監査ログ*" ミドルウェアでは、次のことが行われます。

* ファイルまたはデータベースにログを記録します。
* ユーザー、IP アドレス、機密性の高いエンドポイントの名前などの詳細情報が追加されます。

コントローラーや SignalR などのクラスベースのフレームワークで簡単に使用できるように、監査ポリシー メタデータ `AuditPolicyAttribute` は `Attribute` と定義されています。 "*コードへのルーティング*" を使用すると、次のようになります。

* メタデータがビルダー API にアタッチされます。
* エンドポイントを作成するとき、クラス ベースのフレームワークに、対応するメソッドとクラスのすべての属性が組み込まれます。

メタデータの型に対するベスト プラクティスは、インターフェイスまたは属性として定義することです。 インターフェイスと属性では、コードを再利用できます。 メタデータ システムは柔軟であり、どのような制限もありません。

<a name="tm"></a>

### <a name="comparing-a-terminal-middleware-and-routing"></a>ターミナル ミドルウェアとルーティングの比較

次のコード サンプルでは、ミドルウェアの使用とルーティングの使用が比較されています。

[!code-csharp[](routing/samples/3.x/RoutingSample/TerminalMiddlewareStartup.cs?name=snippet)]

`Approach 1:` で示されているミドルウェアのスタイルは、**ターミナル ミドルウェア** です。 ターミナル ミドルウェアと呼ばれるのは、照合操作を実行するためです。

* 前のサンプルの照合操作は、ミドルウェアの場合は `Path == "/"`、ルーティングの場合は `Path == "/Movie"` です。
* 照合が成功すると、`next` ミドルウェアを呼び出すのではなく、一部の機能を実行して戻ります。

ターミナル ミドルウェアと呼ばれるのは、検索を終了し、いくつかの機能を実行してから制御を返すためです。

ターミナル ミドルウェアとルーティングの比較:
* どちらの方法でも、処理パイプラインを終了できます。
    * ミドルウェアでは、`next` を呼び出すのではなく、戻ることによってパイプラインが終了されます。
    * エンドポイントは常にターミナルです。
* ターミナル ミドルウェアを使用すると、パイプライン内の任意の場所にミドルウェアを配置できます。
    * エンドポイントは、<xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> の位置で実行されます。
* ターミナル ミドルウェアでは、任意のコードを使用してミドルウェアが一致するかどうかを判定できます。
    * カスタム ルート一致コードは、冗長で、正しく記述するのが困難な場合があります。
    * ルーティングでは、一般的なアプリに対して簡単なソリューションが提供されます。 ほとんどのアプリでは、カスタム ルート一致コードは必要ありません。
* `UseAuthorization` や `UseCors` などのミドルウェアを使用したエンドポイント インターフェイス。
    * `UseAuthorization` または `UseCors` でターミナル ミドルウェアを使用するには、承認システムとの手動インターフェイスが必要です。

[エンドポイント](#endpoint) では、次の両方が定義されます。

* 要求を処理するためのデリゲート。
* 任意のメタデータのコレクション。 メタデータは、各エンドポイントにアタッチされている構成とポリシーに基づいて横断的な関心事を実装するために使用されます。

ターミナル ミドルウェアは効果的なツールになる可能性がありますが、次のものが必要です。

* 大量のコーディングとテスト。
* 必要なレベルの柔軟性を実現するための、他のシステムとの手作業による統合。

ターミナル ミドルウェアを作成する前に、ルーティングとの統合を検討してください。

[Map](xref:fundamentals/middleware/index#branch-the-middleware-pipeline) または <xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen*> と統合されている既存のターミナル ミドルウェアは、通常、ルーティング対応のエンドポイントにすることができます。 [MapHealthChecks](https://github.com/aspnet/AspNetCore/blob/master/src/Middleware/HealthChecks/src/Builder/HealthCheckEndpointRouteBuilderExtensions.cs#L16) では、ルーターウェアのパターンが示されています。
* <xref:Microsoft.AspNetCore.Routing.IEndpointRouteBuilder> で拡張メソッドを作成します。
* <xref:Microsoft.AspNetCore.Routing.IEndpointRouteBuilder.CreateApplicationBuilder*> を使用して、入れ子になったミドルウェア パイプラインを作成します。
* ミドルウェアを新しいパイプラインにアタッチします。 例では、 <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*>が使用されます。
* <xref:Microsoft.AspNetCore.Http.RequestDelegate> にミドルウェア パイプラインを <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.Build*> します。
* `Map` を呼び出し、新しいミドルウェア パイプラインを提供します。
* 拡張メソッドから `Map` によって提供されるビルダー オブジェクトを返します。

次のコードでは、[MapHealthChecks](xref:host-and-deploy/health-checks) の使用方法を示します。

[!code-csharp[](routing/samples/3.x/RoutingSample/AuthorizationStartup.cs?name=snippet)]

前のサンプルでは、ビルダー オブジェクトを返すことが重要である理由が示されています。 ビルダー オブジェクトを返すことで、アプリ開発者はエンドポイントの承認などのポリシーを構成できます。 この例では、正常性チェック ミドルウェアと承認システムは直接統合されていません。

そのメタデータ システムは、ターミナル ミドルウェアを使用する機能拡張作成者によって、発生する問題に対応して作成されました。 各ミドルウェアで承認システムとの独自の統合を実装することには問題があります。

<a name="urlm"></a>

### <a name="url-matching"></a>URL 一致

* ルーティングによって受信要求が[エンドポイント](#endpoint)と照合されるプロセスです。
* URL パスとヘッダーのデータに基づいています。
* 要求内の任意のデータを考慮するように拡張できます。

実行されたルーティング ミドルウェアでは、`Endpoint` が設定され、現在の要求からの <xref:Microsoft.AspNetCore.Http.HttpContext> で[要求機能](xref:fundamentals/request-features)に値がルーティングされます。

* [HttpContext.GetEndpoint](<xref:Microsoft.AspNetCore.Http.EndpointHttpContextExtensions.GetEndpoint*>) を呼び出すと、エンドポイントが取得されます。
* `HttpRequest.RouteValues` では、ルート値のコレクションが取得されます。

ルーティング ミドルウェアの後で実行された[ミドルウェア](xref:fundamentals/middleware/index)では、エンドポイントを調べて、アクションを実行することができます。 たとえば、承認ミドルウェアでは、エンドポイントのメタデータ コレクションに対し、承認ポリシーを問い合わせることができます。 要求処理パイプライン内のすべてのミドルウェアが実行された後、選択したエンドポイントのデリゲートが呼び出されます。

エンドポイント ルーティングのルーティング システムでは、配布に関するすべての決定が行われます。 ミドルウェアでは選択されたエンドポイントに基づいてポリシーが適用されるため、次のことが重要です。

* ディスパッチまたはセキュリティ ポリシーの適用に影響を与える可能性のある決定は、ルーティング システム内で行われます。

> [!WARNING]
> 後方互換性のために、コントローラーまたは Razor Pages エンドポイント デリゲートが実行されると、それまでに実行された要求処理に基づいて、[RouteContext.RouteData](xref:Microsoft.AspNetCore.Routing.RouteContext.RouteData) のプロパティが適切な値に設定されます。
>
> `RouteContext` の種類は、今後のリリースでは古いものとしてマークされます。
>
> * `RouteData.Values` を `HttpRequest.RouteValues` に移行します。
> * `RouteData.DataTokens` を移行して、エンドポイント メタデータから [IDataTokensMetadata](xref:Microsoft.AspNetCore.Routing.IDataTokensMetadata) を取得します。

URL の照合は、構成可能な一連のフェーズで動作します。 各フェーズでの出力は一致のセットとなります。 一致のセットは、次のフェーズでさらに絞り込むことができます。 ルーティングの実装では、一致するエンドポイントの処理順序は保証されません。 一致の可能性のあるものは一度に **すべて** 処理されます。 URL 照合フェーズは、次の順序で発生します。 ASP.NET Core:

1. エンドポイントのセットおよびそれらのルート テンプレートに対して URL パスを処理し、**すべて** の一致を収集します。
1. 前のリストを取得し、ルート制約が適用されると失敗する一致を削除します。
1. 前のリストを取得し、[MatcherPolicy](xref:Microsoft.AspNetCore.Routing.MatcherPolicy) インスタンスのセットを失敗させる一致を削除します。
1. [EndpointSelector](xref:Microsoft.AspNetCore.Routing.Matching.EndpointSelector) を使用して、前のリストから最終的な決定を行います。

エンドポイントのリストは、次の内容に従って優先度付けが行われます。

* [RouteEndpoint.Order](xref:Microsoft.AspNetCore.Routing.RouteEndpoint.Order*)
* [ルート テンプレートの優先順位](#rtp)

<xref:Microsoft.AspNetCore.Routing.Matching.EndpointSelector> に到達するまで、各フェーズで一致するすべてのエンドポイントが処理されます。 `EndpointSelector` は最後のフェーズです。 一致の中から最も優先度の高いエンドポイントが最適な一致として選択されます。 最適な一致と優先度が同じである一致が他にもある場合は、あいまい一致の例外がスローされます。

ルートの優先順位は **より具体的な** ルート テンプレートに、より高い優先度が与えられることに基づいて算出されます。 たとえば、テンプレート `/hello` と `/{message}` を検討してみます。

* どちらも URL パス `/hello` と一致します。
* `/hello` の方がより具体的なので、優先度が高くなります。

一般に、ルートの優先順位は、実際に使用される URL スキームの種類として最適なものを選択するのに適しています。 <xref:Microsoft.AspNetCore.Routing.RouteEndpoint.Order> は、あいまいさを避けるために必要な場合にのみ使用します。

拡張性の種類がルーティングによって指定されるため、あいまいなルートを事前にルーティング システムによって計算することはできません。 ルート テンプレート `/{message:alpha}` や `/{message:int}` などの例を考えてみましょう。

* `alpha` 制約を使用すると、アルファベット文字のみと一致します。
* `int` 制約を使用すると、数値のみと一致します。
* これらのテンプレートのルート優先順位は同じですが、この両方と一致する単一の URL はありません。
* 起動時にルーティング システムからあいまいエラーが報告された場合、それによってこの有効なユース ケースはブロックされます。

> [!WARNING]
>
> <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> 内での処理の順序は、ルーティングの動作には影響しませんが、例外が 1 つあります。 <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute*> および <xref:Microsoft.AspNetCore.Builder.MvcAreaRouteBuilderExtensions.MapAreaRoute*> では、それぞれが呼び出された順序に基づいて、それぞれのエンドポイントに順序値が自動的に割り当てられます。 これにより、ルーティング システムでより古いルーティング実装と同じ保証を提供しなくても、コントローラーの長時間の動作がシミュレートされます。
>
> ルーティングの従来の実装では、ルートの処理順序に依存するルーティング拡張性を実装することができます。 ASP.NET Core 3.0 以降でのエンドポイントのルーティング:
> 
> * ルートの概念がありません。
> * 順序付けが保証されません。 すべてのエンドポイントが一度に処理されます。

<a name="rtp"></a>

### <a name="route-template-precedence-and-endpoint-selection-order"></a>ルート テンプレートの優先順位とエンドポイントの選択順序

[ルート テンプレートの優先順位](https://github.com/dotnet/aspnetcore/blob/master/src/Http/Routing/src/Template/RoutePrecedence.cs#L16)とは、どれほど具体的であるかに基づいて、各ルート テンプレートに値を割り当てるシステムです。 ルート テンプレートの優先順位:

* 一般的なケースでは、エンドポイントの順序を調整する必要はなくなります。
* 一般的に期待されるルーティング動作との一致が試みられます。

たとえば、テンプレート `/Products/List` と `/Products/{id}` について考えてみます。 URL パス `/Products/List` に対しては、`/Products/List` の方が `/Products/{id}` よりも適していると想定するのが妥当です。 このように言えるのは、リテラル セグメント `/List` がパラメーター セグメント `/{id}` よりも優先順位が高いと見なされるためです。

優先順位のしくみの詳細は、ルート テンプレートの定義方法と関連付けられています。

* より多くのセグメントを持つテンプレートは、より具体的なものと見なされます。
* リテラル テキストを含むセグメントは、パラメーター セグメントよりも具体的であると見なされます。
* 制約が含まれるパラメーター セグメントは、それが含まれないものよりも具体的であると見なされます。
* 複雑なセグメントは、制約を含むパラメーター セグメントと同じくらい具体的であると見なされます。
* キャッチオール パラメーターは、まったく具体的ではありません。 キャッチオール ルートに関する重要な情報については、「[ルート テンプレート参照](#rtr)」の **キャッチオール** を参照してください。

実際の値のリファレンスについては、[GitHub 上のソース コード](https://github.com/dotnet/aspnetcore/blob/master/src/Http/Routing/src/Template/RoutePrecedence.cs#L189)を参照してください。

<a name="lg"></a>

### <a name="url-generation-concepts"></a>URL 生成の概念

URL の生成:

* ルーティングにおいて、一連のルート値に基づいて URL パスを作成するプロセスです。
* エンドポイントとそれにアクセスする URL を論理的に分離できます。

エンドポイント ルーティングには、<xref:Microsoft.AspNetCore.Routing.LinkGenerator> API が含まれます。 `LinkGenerator` は [DI](xref:fundamentals/dependency-injection) から使用できるシングルトン サービスです。 `LinkGenerator` API は、実行中の要求のコンテキスト外で使用することができます。 [Mvc.IUrlHelper](xref:Microsoft.AspNetCore.Mvc.IUrlHelper) と、[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)、HTML ヘルパー、[アクション結果](xref:mvc/controllers/actions)など、<xref:Microsoft.AspNetCore.Mvc.IUrlHelper> に依存するシナリオでは `LinkGenerator` API を内部的に使用して、リンク生成機能が提供されます。

リンク ジェネレーターは、**アドレス** と **アドレス スキーム** の概念に基づいています。 アドレス スキームは、リンク生成で考慮すべきエンドポイントを決定する方法です。 たとえば、コントローラーおよび Razor Pages からの、多くのユーザーに馴染みのあるルート名やルート値シナリオは、アドレス スキームとして実装されます。

リンク ジェネレーターでは、次の拡張メソッドを介して、コントローラーおよび Razor Pages にリンクできます。

* <xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetPathByAction*>
* <xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetUriByAction*>
* <xref:Microsoft.AspNetCore.Routing.PageLinkGeneratorExtensions.GetPathByPage*>
* <xref:Microsoft.AspNetCore.Routing.PageLinkGeneratorExtensions.GetUriByPage*>

これらのメソッドのオーバーロードでは、`HttpContext` を含む引数が受け入れられます。 これらのメソッドは [Url.Action](xref:System.Web.Mvc.UrlHelper.Action*) および [Url.Page](xref:Microsoft.AspNetCore.Mvc.UrlHelperExtensions.Page*) と機能的には同等ですが、柔軟性とオプションがさらに提供されます。

`GetPath*` メソッドは、絶対パスを含む URI を生成するという点で `Url.Action` および `Url.Page` に最も似ています。 `GetUri*` メソッドでは常に、スキームとホストを含む絶対 URI が生成されます。 `HttpContext` を受け入れるメソッドでは、実行中の要求のコンテキストで URI が生成されます。 実行中の要求からの[アンビエント](#ambient) ルート値、URL ベース パス、スキーム、およびホストは、オーバーライドされない限り使用されます。

<xref:Microsoft.AspNetCore.Routing.LinkGenerator> はアドレスと共に呼び出されます。 URI の生成は、次の 2 つの手順で行われます。

1. アドレスは、そのアドレスと一致するエンドポイントのリストにバインドされます。
1. 各エンドポイントの <xref:Microsoft.AspNetCore.Routing.RouteEndpoint.RoutePattern> は、指定された値と一致するルート パターンが見つかるまで評価されます。 結果の出力は、リンク ジェネレーターに指定された他の URI 部分と結合され、返されます。

<xref:Microsoft.AspNetCore.Routing.LinkGenerator> によって提供されるメソッドでは、すべての種類のアドレスの標準的なリンク生成機能がサポートされます。 リンク ジェネレーターを使用する最も便利な方法は、特定のアドレスの種類の操作を実行する拡張メソッドを使用することです。

| 拡張メソッド | 説明 |
| ---------------- | ----------- |
| <xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetPathByAddress*> | 指定された値に基づき、絶対パスを含む URI を生成します。 |
| <xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetUriByAddress*> | 指定された値に基づき、絶対 URI を生成します。             |

> [!WARNING]
> <xref:Microsoft.AspNetCore.Routing.LinkGenerator> メソッド呼び出しによる次の影響に注意してください。
>
> * 受信要求の `Host` ヘッダーが確認されないアプリ構成では、`GetUri*` 拡張メソッドは注意して使用してください。 受信要求の `Host` ヘッダーが確認されていない場合、信頼されていない要求入力を、ビューまたはページの URI でクライアントに送り返すことができます。 すべての運用アプリで、`Host` ヘッダーを既知の有効な値と照らし合わせて確認するようにサーバーを構成することをお勧めします。
>
> * ミドルウェアで `Map` または `MapWhen` と組み合わせて、<xref:Microsoft.AspNetCore.Routing.LinkGenerator> を使用する場合は注意してください。 `Map*` では、実行中の要求の基本パスが変更され、リンク生成の出力に影響します。 すべての <xref:Microsoft.AspNetCore.Routing.LinkGenerator> API で基本パスを指定することができます。 リンク生成への `Map*` の影響を元に戻すための空の基本パスを指定してください。

### <a name="middleware-example"></a>ミドルウェアの例

次の例では、ミドルウェアで <xref:Microsoft.AspNetCore.Routing.LinkGenerator> API を使用して、商品をリストするアクション メソッドへのリンクを作成します。 リンク ジェネレーターは、クラスに挿入し、`GenerateLink` を呼び出すことで、アプリのどのクラスでも使用できます。

[!code-csharp[](routing/samples/3.x/RoutingSample/Middleware/ProductsLinkMiddleware.cs?name=snippet)]

<a name="rtr"></a>

## <a name="route-template-reference"></a>ルート テンプレート参照

`{}` 内のトークンでは、ルートが一致した場合にバインドされるルート パラメーターが定義されます。 1 つのルート セグメントに複数のルート パラメーターを定義できますが、各ルート パラメーターをリテラル値で区切る必要があります。 たとえば、`{controller=Home}{action=Index}` は有効なルートになりません。`{controller}` と `{action}` の間にリテラル値がないためです。  ルート パラメーターには名前を付ける必要があります。付加的な属性を指定することもあります。

ルート パラメーター以外のリテラル テキスト (`{id}` など) とパス区切り `/` は URL のテキストに一致する必要があります。 テキスト照合は復号された URL のパスを基盤とし、大文字と小文字が区別されます。 リテラル ルート パラメーターの区切り記号 (`{` または `}`) を照合するには、文字を繰り返して区切り記号をエスケープします。 たとえば、`{{` または `}}` です。

アスタリスク `*` または二重アスタリスク`**`:

* ルート パラメーターのプレフィックスとして使用して、URI の残りの部分にバインドすることができます。
* **キャッチオール** パラメーターと呼ばれています。 `blog/{**slug}` の例を次に示します。
  * `/blog` で始まり、その後に任意の値が続く URI と一致します。
  * `/blog` に続く値は、[slug](https://developer.mozilla.org/docs/Glossary/Slug) ルート値に割り当てられます。

[!INCLUDE[](~/includes/catchall.md)]

キャッチオール パラメーターは空の文字列に一致することもあります。

キャッチオール パラメーターでは、パス区切り `/` 文字を含め、URL の生成にルートが使用されるときに適切な文字がエスケープされます。 たとえば、ルート値が `{ path = "my/path" }` のルート `foo/{*path}` では、`foo/my%2Fpath` が生成されます。 エスケープされたスラッシュに注意してください。 パス区切り文字をラウンドトリップさせるには、`**` ルート パラメーター プレフィックスを使用します。 `{ path = "my/path" }` のルート `foo/{**path}` では、`foo/my/path` が生成されます。

任意のファイル拡張子が付いたファイル名のキャプチャを試行する URL パターンには、追加の考慮事項があります。 たとえば、テンプレート `files/{filename}.{ext?}` について考えてみます。 `filename` と `ext` の両方の値が存在するときに、両方の値が入力されます。 URL に `filename` の値だけが存在する場合、末尾の `.` は任意であるため、このルートは一致となります。 次の URL はこのルートに一致します。

* `/files/myFile.txt`
* `/files/myFile`

ルート パラメーターには、**既定値** が含まれることがあります。パラメーター名の後に既定値を指定し、等号 (`=`) で区切ることで指定されます。 たとえば、`{controller=Home}` では、`controller` の既定値として `Home` が定義されます。 パラメーターの URL に値がない場合、既定値が使用されます。 ルート パラメーターは、パラメーター名の終わりに疑問符 (`?`) を追加することでオプションとして扱われます。 たとえば、`id?` のようにします。 省略可能な値と既定のルート パラメーターの違いは次のとおりです。

* 既定値を持つルート パラメーターでは常に値が生成されます。
* 省略可能なパラメーターには、要求 URL によって値が指定された場合にのみ値が含められます。

ルート パラメーターには、URL からバインドされるルート値に一致しなければならないという制約が含まれることがあります。 `:` と制約名をルート パラメーター名の後に追加すると、ルート パラメーターのインライン制約が指定されます。 その制約で引数が要求される場合、制約名の後にかっこ `(...)` で囲まれます。 複数の "*インライン制約*" を指定するには、別の `:` と制約名を追加します。

制約名と引数が <xref:Microsoft.AspNetCore.Routing.IInlineConstraintResolver> サービスに渡され、URL 処理で使用する <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> のインスタンスが作成されます。 たとえば、ルート テンプレート `blog/{article:minlength(10)}` によって、制約 `minlength` と引数 `10` が指定されます。 ルート制約の詳細とこのフレームワークによって指定される制約のリストについては、「[ルート制約参照](#route-constraint-reference)」セクションを参照してください。

ルート パラメーターには、パラメーター トランスフォーマーを指定することもできます。 パラメーター トランスフォーマーを指定すると、リンクを生成し、アクションおよびページを URL と一致させるときにパラメーターの値が変換されます。 制約と同様に、パラメーター トランスフォーマーをルート パラメーターにインラインで追加することができます。その場合、ルート パラメーター名の後に `:` とトランスフォーマー名を追加します。 たとえば、ルート テンプレート `blog/{article:slugify}` では、`slugify` トランスフォーマーが指定されます。 パラメーター トランスフォーマーの詳細については、「[パラメーター トランスフォーマー参照](#parameter-transformer-reference)」セクションを参照してください。

次の表に、ルート テンプレートの例とその動作を示します。

| ルート テンプレート                           | 一致する URI の例    | 要求 URI&hellip;                                                    |
| ---------------------------------------- | ----------------------- | -------------------------------------------------------------------------- |
| `hello`                                  | `/hello`                | 単一パス `/hello` にのみ一致します。                                     |
| `{Page=Home}`                            | `/`                     | 一致し、`Page` が `Home` に設定されます。                                         |
| `{Page=Home}`                            | `/Contact`              | 一致し、`Page` が `Contact` に設定されます。                                      |
| `{controller}/{action}/{id?}`            | `/Products/List`        | `Products` コントローラーと `List` アクションにマッピングされます。                       |
| `{controller}/{action}/{id?}`            | `/Products/Details/123` | `Products` コントローラーと `Details` アクションにマッピングされ、`id` は 123 に設定されます。 |
| `{controller=Home}/{action=Index}/{id?}` | `/`                     | `Home` コントローラーと `Index` メソッドにマッピングされます。 `id` は無視されます。        |
| `{controller=Home}/{action=Index}/{id?}` | `/Products`         | `Products` コントローラーと `Index` メソッドにマッピングされます。 `id` は無視されます。        |

一般的に、テンプレートの利用が最も簡単なルーティングの手法となります。 ルート テンプレート以外では、制約と既定値も指定できます。

### <a name="complex-segments"></a>複雑なセグメント

複雑なセグメントは、リテラル区切り文字を右から左に[最短一致](#greedy)の方法で照合することによって処理されます。 たとえば、`[Route("/a{b}c{d}")]` は複雑なセグメントです。
複雑なセグメントは、それらを適切に使用する上で理解する必要がある特定の方法で機能します。 このセクションの例では、パラメーター値の中に区切り文字が含まれていない場合にのみ、複雑なセグメントが本当にうまく機能する理由を示します。 より複雑なケースでは、[regex](/dotnet/standard/base-types/regular-expressions) を使用し、値を手動で抽出する必要があります。

[!INCLUDE[](~/includes/regex.md)]

これは、ルーティングがテンプレート `/a{b}c{d}` と URL パス `/abcd` を使用して実行するステップの概要です。 `|` は、アルゴリズムの動作を視覚化するために使用されます。

* 最初のリテラル (右から左へ) は `c` です。 そこで、`/abcd` は右から検索され、`/ab|c|d` となります。
* ここで、右にあるすべてのもの (`d`) がルート パラメーター `{d}` と照合されます。
* 次のリテラル (右から左へ) は `a` です。 そのため `/ab|c|d` は中断したところから検索されて、`a` が見つかり、`/|a|b|c|d` となります。
* ここで、右の値 (`b`) がルート パラメーター `{b}` と照合されます。
* 残りのテキストも残りのルート テンプレートも存在しないため、これは一致となります。

同じテンプレート `/a{b}c{d}` と、URL パス `/aabcd` を使用した場合の否定の例を次に示します。 `|` は、アルゴリズムの動作を視覚化するために使用されます。 このケースは一致ではありませんが、同じアルゴリズムで説明します。
* 最初のリテラル (右から左へ) は `c` です。 そこで、`/aabcd` は右から検索され、`/aab|c|d` となります。
* ここで、右にあるすべてのもの (`d`) がルート パラメーター `{d}` と照合されます。
* 次のリテラル (右から左へ) は `a` です。 そのため `/aab|c|d` は中断したところから検索されて、`a` が見つかり、`/a|a|b|c|d` となります。
* ここで、右の値 (`b`) がルート パラメーター `{b}` と照合されます。
* この時点で、テキスト `a` が残っていますが、アルゴリズムは解析するためのルート テンプレートを使い果たしたので、これは一致とはなりません。

照合アルゴリズムは[最短一致](#greedy)のため、次のようになります。

* 各ステップで可能な限りの最短のテキストに一致します。
* パラメーター値の内部に区切り記号の値が表示されている場合は一致しません。

正規表現を使用すると、一致の動作をより細かく制御できます。

<a name="greedy"></a>

最長一致 ([遅延一致](https://wikipedia.org/wiki/Regular_expression#Lazy_matching) とも呼ばれる) を使用すると、可能な限り長い文字列と一致します。 最短一致は、可能な限り最短の文字列と一致します。

## <a name="route-constraint-reference"></a>ルート制約参照

ルート制約は、受信 URL と一致し、URL パスがルート値にトークン化されたときに実行されます。 ルート制約では、通常、ルート テンプレート経由で関連付けられるルート値を調べ、値が許容できるかどうかを true または false で決定します。 一部のルート制約では、ルート値以外のデータを使用し、要求をルーティングできるかどうかが考慮されます。 たとえば、<xref:Microsoft.AspNetCore.Routing.Constraints.HttpMethodRouteConstraint> はその HTTP Verb に基づいて要求を承認または却下します。 制約は、要求のルーティングとリンクの生成で使用されます。

> [!WARNING]
> 入力の検証には制約を使用しないでください。 入力の検証に制約を使用した場合、入力が無効だと "`404` 見つかりません" が返されます。 無効な入力の場合は、"`400` 要求が無効です" と適切なエラー メッセージが生成されます。 ルート制約は、特定のルートに対する入力の妥当性を検証するためではなく、似たようなルートの違いを明らかにするために使用されます。

次の表では、ルート制約の例とそれに求められる動作をまとめています。

| 制約 | 例 | 一致の例 | メモ |
| ---------- | ------- | --------------- | ----- |
| `int` | `{id:int}` | `123456789`, `-123456789` | あらゆる整数に一致する |
| `bool` | `{active:bool}` | `true`, `FALSE` | `true` または `false` に一致する。 大文字と小文字は区別されない |
| `datetime` | `{dob:datetime}` | `2016-12-31`, `2016-12-31 7:32pm` | インバリアント カルチャの有効な `DateTime` 値に一致します。 前の警告を参照してください。 |
| `decimal` | `{price:decimal}` | `49.99`, `-1,000.01` | インバリアント カルチャの有効な `decimal` 値に一致します。 前の警告を参照してください。|
| `double` | `{weight:double}` | `1.234`, `-1,001.01e8` | インバリアント カルチャの有効な `double` 値に一致します。 前の警告を参照してください。|
| `float` | `{weight:float}` | `1.234`, `-1,001.01e8` | インバリアント カルチャの有効な `float` 値に一致します。 前の警告を参照してください。|
| `guid` | `{id:guid}` | `CD2C1638-1638-72D5-1638-DEADBEEF1638` | 有効な `Guid` 値に一致する |
| `long` | `{ticks:long}` | `123456789`, `-123456789` | 有効な `long` 値に一致する |
| `minlength(value)` | `{username:minlength(4)}` | `Rick` | 4 文字以上の文字列であることが必要 |
| `maxlength(value)` | `{filename:maxlength(8)}` | `MyFile` | 8 文字以内の文字列であることが必要 |
| `length(length)` | `{filename:length(12)}` | `somefile.txt` | 厳密に 12 文字の文字列であることが必要 |
| `length(min,max)` | `{filename:length(8,16)}` | `somefile.txt` | 8 文字以上、16 文字以内の文字列であることが必要 |
| `min(value)` | `{age:min(18)}` | `19` | 18 以上の整数値であることが必要 |
| `max(value)` | `{age:max(120)}` | `91` | 120 以下の整数値であることが必要 |
| `range(min,max)` | `{age:range(18,120)}` | `91` | 18 以上、120 以下の整数値であることが必要 |
| `alpha` | `{name:alpha}` | `Rick` | 文字列は 1 つまたは複数のアルファベット文字で構成されることが必要 (`a`-`z`、大文字と小文字は区別されません)。 |
| `regex(expression)` | `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}` | `123-45-6789` | 文字列は正規表現と一致する必要があります。 正規表現の定義に関するヒントを参照してください。 |
| `required` | `{name:required}` | `Rick` | URL 生成中、非パラメーターが提示されるように強制する |

[!INCLUDE[](~/includes/regex.md)]

1 のパラメーターには、複数の制約をコロンで区切って適用できます。 たとえば、次の制約では、パラメーターが 1 以上の整数値に制限されます。

```csharp
[Route("users/{id:int:min(1)}")]
public User GetUserById(int id) { }
```

> [!WARNING]
> URL の妥当性を検証し、CLR 型に変換するルート制約では、常にインバリアント カルチャが使用されます。 たとえば、`int` または `DateTime` の CLR 型に変換される場合などです。 これらの制約では、URL のローカライズが不可であることが前提です。 フレームワークから提供されるルート制約がルート値に格納されている値を変更することはありません。 URL から解析されたルート値はすべて文字列として格納されます。 たとえば、`float` 制約はルート値を浮動小数に変換しますが、変換された値は、浮動小数に変換できることを検証するためにだけ利用されます。

### <a name="regular-expressions-in-constraints"></a>正規表現の制約

[!INCLUDE[](~/includes/regex.md)]

正規表現は、`regex(...)` ルート制約を使用して、インライン制約として指定できます。 <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute*> ファミリのメソッドでも、制約のオブジェクト リテラルを取ります。 この形式が使用されている場合、文字列値は正規表現として解釈されます。

次のコードでは、インラインで regex 制約が使用されています。

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupRegex.cs?name=snippet)]

次のコードでは、regex 制約の指定にオブジェクト リテラルが使用されています。

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupRegex2.cs?name=snippet)]

ASP.NET Core フレームワークでは、正規表現コンストラクターに `RegexOptions.IgnoreCase | RegexOptions.Compiled | RegexOptions.CultureInvariant` が追加されます。 これらのメンバーの詳細については、「<xref:System.Text.RegularExpressions.RegexOptions>」を参照してください。

正規表現では、ルーティングや C# 言語で使用されるものに似た区切り記号とトークンが使用されます。 正規表現トークンはエスケープする必要があります。 インライン制約で正規表現 `^\d{3}-\d{2}-\d{4}$` を使用するには、次のいずれかを使用します。

* `\` 文字列エスケープ文字をエスケープするには、文字列で指定した `\` 文字を、C# ソース ファイル内の `\\` 文字に置き換えます。
* [逐語的文字列リテラル](/dotnet/csharp/language-reference/keywords/string)。

ルーティング パラメーター区切り記号文字 (`{`、`}`、`[`、`]`) をエスケープするには、表現の文字を二重にします (例: `{{`、`}}`、`[[`、`]]`)。 次の表に、正規表現とそれにエスケープを適用した後のものを示します。

| 正規表現    | エスケープ適用後の正規表現     |
| --------------------- | ------------------------------ |
| `^\d{3}-\d{2}-\d{4}$` | `^\\d{{3}}-\\d{{2}}-\\d{{4}}$` |
| `^[a-z]{2}$`          | `^[[a-z]]{{2}}$`               |

ルーティングで使用される正規表現は、多くの場合、`^` 文字で始まり、これは文字列の開始位置と一致します。 この式は、多くの場合、`$` 文字で終わり、文字列の末尾と一致します。 `^` 文字と `$` 文字により、正規表現がルート パラメーター値全体に一致することが保証されます。 `^` 文字と `$` 文字がなければ、意図に反し、正規表現は文字列内のあらゆる部分文字列に一致してしまいます。 下の表では、一致または不一致の理由を例を示し説明します。

| 式   | String    | 一致したもの | コメント               |
| ------------ | --------- | :---: |  -------------------- |
| `[a-z]{2}`   | hello     | はい   | サブ文字列の一致     |
| `[a-z]{2}`   | 123abc456 | はい   | サブ文字列の一致     |
| `[a-z]{2}`   | mz        | はい   | 一致する表現    |
| `[a-z]{2}`   | MZ        | はい   | 大文字と小文字の使い方が違う    |
| `^[a-z]{2}$` | hello     | いいえ    | 上の `^` と `$` を参照 |
| `^[a-z]{2}$` | 123abc456 | いいえ    | 上の `^` と `$` を参照 |

正規表現構文の詳細については、[.NET Framework 正規表現](/dotnet/standard/base-types/regular-expression-language-quick-reference)に関するページを参照してください。

既知の入力可能値の集まりにパラメーターを制限するには、正規表現を使用します。 たとえば、`{action:regex(^(list|get|create)$)}` の場合、`action` ルート値は `list`、`get`、`create` とのみ照合されます。 制約ディクショナリに渡された場合、文字列 `^(list|get|create)$` で同じものになります。 既知の制約に一致しない、制約ディクショナリに渡された制約も、正規表現として扱われます。 既知の制約に一致しない、テンプレート内で渡される制約は、正規表現としては扱われません。

### <a name="custom-route-constraints"></a>カスタム ルート制約

カスタム ルート制約は、<xref:Microsoft.AspNetCore.Routing.IRouteConstraint> インターフェイスを実装して作成できます。 `IRouteConstraint` インターフェイスには、<xref:System.Web.Routing.IRouteConstraint.Match*> が含まれています。これでは、制約が満たされている場合は `true` を返し、それ以外の場合は `false` を返します。

カスタム ルート制約は通常必要ありません。 カスタム ルート制約を実装する前に、モデル バインドなどの代替手段を検討してください。

ASP.NET Core の [Constraints](https://github.com/dotnet/aspnetcore/tree/master/src/Http/Routing/src/Constraints) フォルダーには、制約を作成するための適切な例が用意されています。 たとえば、[GuidRouteConstraint](https://github.com/dotnet/aspnetcore/blob/master/src/Http/Routing/src/Constraints/GuidRouteConstraint.cs#L18) です。

カスタムの `IRouteConstraint` を使うには、サービス コンテナー内の <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> に、ルート制約の種類が登録されている必要があります。 `ConstraintMap` は、ルート制約キーを、その制約を検証する `IRouteConstraint` の実装にマッピングするディクショナリです。 アプリの `ConstraintMap` は、`Startup.ConfigureServices` で、[services.AddRouting](xref:Microsoft.Extensions.DependencyInjection.RoutingServiceCollectionExtensions.AddRouting*) 呼び出しの一部として、または `services.Configure<RouteOptions>` を使って <xref:Microsoft.AspNetCore.Routing.RouteOptions> を直接構成することで、更新できます。 次に例を示します。

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupConstraint.cs?name=snippet)]

上記の制約は、次のコードに適用されます。

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/TestController.cs?name=snippet&highlight=6,13)]

[!INCLUDE[](~/includes/MyDisplayRouteInfo.md)]

`MyCustomConstraint` を実装することにより、ルート パラメーターに `0` が適用されなくなります。

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupConstraint.cs?name=snippet2)]

[!INCLUDE[](~/includes/regex.md)]

上記のコードでは次の操作が行われます。

* ルートの `{id}` セグメントの `0` を禁止します。
* カスタム制約を実装する基本的な例を示しています。 実稼働しているアプリでは使用しないでください。

次のコードは、`0` を含む `id` が処理されないようにする優れた方法です。

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/TestController.cs?name=snippet2)]

上記のコードには `MyCustomConstraint` アプローチに対し、次の利点があります。

* カスタム制約が必要ありません。
* ルート パラメーターに `0` が含まれている場合は、よりわかりやすいエラーが返されます。

## <a name="parameter-transformer-reference"></a>パラメーター トランスフォーマー参照

パラメーター トランスフォーマー:

* <xref:Microsoft.AspNetCore.Routing.LinkGenerator> を使用したリンクの生成時に実行されます。
* <xref:Microsoft.AspNetCore.Routing.IOutboundParameterTransformer?displayProperty=fullName>を実装します。
* <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> を使用して構成されます。
* パラメーターのルート値を取得し、それを新しい文字列値に変換します。
* 変換された値は生成されたリンクで使用されるようになります。

たとえば、`Url.Action(new { article = "MyTestArticle" })` のルート パターン `blog\{article:slugify}` のカスタム `slugify` パラメーター トランスフォーマーでは、`blog\my-test-article` が生成されます。

`IOutboundParameterTransformer` の次の実装を見てみましょう。

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupConstraint2.cs?name=snippet2)]

ルート パターンでパラメーター トランスフォーマーを使用するには、これを `Startup.ConfigureServices` で <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> を使用して構成します。

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupConstraint2.cs?name=snippet)]

ASP.NET Core フレームワークでは、エンドポイントを解決する URI の変換にパラメーター トランスフォーマーを使用します。 たとえば、パラメーター トランスフォーマーでは、`area`、`controller`、`action`、`page` を照合するために使用されるルート値が変換されます。

```csharp
routes.MapControllerRoute(
    name: "default",
    template: "{controller:slugify=Home}/{action:slugify=Index}/{id?}");
```

上記のルート テンプレートでは、アクション `SubscriptionManagementController.GetAll` は URI `/subscription-management/get-all` と照合されます。 パラメーター トランスフォーマーでは、リンクを生成するために使用されるルート値は変更されません。 たとえば、`Url.Action("GetAll", "SubscriptionManagement")` では `/subscription-management/get-all` が出力されます。

ASP.NET Core には、生成されたルートでパラメーター トランスフォーマーを使用する API 規則があります。

* <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.RouteTokenTransformerConvention?displayProperty=fullName> MVC 規則では、指定されたパラメーター トランスフォーマーをアプリ内のすべての属性ルートに適用します。 パラメーター トランスフォーマーでは、置き換えられる属性ルート トークンが変換されます。 詳細については、「[パラメーター トランスフォーマーを使用してトークンの置換をカスタマイズする](xref:mvc/controllers/routing#use-a-parameter-transformer-to-customize-token-replacement)」をご覧ください。
* Razor Pages には、<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageRouteTransformerConvention> API 規則を使用しています。 この規則では、指定されたパラメーター トランスフォーマーが、自動で検出されたすべての Razor Pages に適用されます。 パラメーター トランスフォーマーでは、Razor Pages ルートのフォルダーとファイル名のセグメントが変換されます。 詳細については、[パラメーター トランスフォーマーを使用したページ ルートのカスタマイズ](xref:razor-pages/razor-pages-conventions#use-a-parameter-transformer-to-customize-page-routes)に関する記事をご覧ください。

<a name="ugr"></a>

## <a name="url-generation-reference"></a>URL 生成参照

このセクションには、URL の生成で実装するアルゴリズムの参照情報が含まれています。 実際には、URL 生成の最も複雑な例で、コントローラーまたは Razor Pages が使用されます。 詳細については、[コントローラーでのルーティング](xref:mvc/controllers/routing)に関するページを参照してください。

URL の生成プロセスは、[LinkGenerator. GetPathByAddress](xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetPathByAddress*)、または類似のメソッドへの呼び出しで開始されます。 このメソッドは、アドレス、一連のルート値、およびオプションで `HttpContext` からの現在の要求に関する情報と共に渡されます。

まずは、アドレスを使用して、アドレスの型に一致する [`IEndpointAddressScheme<TAddress>`](xref:Microsoft.AspNetCore.Routing.IEndpointAddressScheme`1) を使用して、一連の候補のエンドポイントが解決されます。

アドレス スキームによって一連の候補が検出されると、URL の生成操作が成功するまで、エンドポイントは反復的に並べ替えられ、処理されます。 URL が生成される際には、あいまいさの確認は **行われず**、最初に返される結果が最終的な結果になります。

### <a name="troubleshooting-url-generation-with-logging"></a>ログを使用した URL 生成のトラブルシューティング

URL の生成のトラブルシューティングを行う場合、まずは `Microsoft.AspNetCore.Routing` のログ記録レベルを `TRACE` に設定します。 `LinkGenerator` では、問題のトラブルシューティングに役立つ、処理に関する多くの詳細がログに記録されます。

URL 生成の詳細については、「[URL 生成参照](#ugr)」を参照してください。

### <a name="addresses"></a>アドレス

アドレスとは、リンク ジェネレーターへの呼び出しを一連の候補エンドポイントにバインドするために使用する、URL 生成の概念です。

アドレスとは、次の 2 つの実装を既定で備えた拡張可能な概念です。

* アドレスとして "*エンドポイント名*" (`string`) を使用します。
    * MVC のルート名と同様の機能があります。
    * <xref:Microsoft.AspNetCore.Routing.IEndpointNameMetadata> メタデータ型を使用します。
    * 指定された文字列を、登録されているすべてのエンドポイントのメタデータに対して解決します。
    * 複数のエンドポイントが同じ名前を使用している場合は、起動時に例外をスローします。
    * コントローラーと Razor Pages 以外で汎用的に使用する場合にお勧めします。
* *ルート値* (<xref:Microsoft.AspNetCore.Routing.RouteValuesAddress>) をアドレスとして使用すると、次のようになります。
    * コントローラーおよび Razor Pages での従来の URL 生成と同様の機能があります。
    * 拡張およびデバッグする場合に非常に複雑です。
    * `IUrlHelper`、タグ ヘルパー、HTML ヘルパー、アクションの結果などで使用される実装を提供します。

アドレス スキームの役割は、任意の条件によって、アドレスと一致するエンドポイント間の関連付けを作成することです。

* エンドポイント名スキームでは、基本的な辞書検索が実行されます。
* ルート値のスキームには、セット アルゴリズムの複雑な最良のサブセットがあります。

<a name="ambient"></a>

### <a name="ambient-values-and-explicit-values"></a>アンビエント値と明示的な値

ルーティングは、現在の要求から現在の要求 `HttpContext.Request.RouteValues` のルート値にアクセスします。 現在の要求に関連付けられている値は、**アンビエント値** と呼ばれます。 このドキュメントでは、わかりやすくするために、メソッドに渡されるルート値を **明示的な値** と呼びます。

次の例では、アンビエント値と明示的な値を示しています。 これでは、アンビエント値を現在の要求と明示的な値から提供します (`{ id = 17, }`)。

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/WidgetController.cs?name=snippet)]

上記のコードでは次の操作が行われます。

* `/Widget/Index/17` を返します。
* [DI](xref:fundamentals/dependency-injection) を介して <xref:Microsoft.AspNetCore.Routing.LinkGenerator> を取得します。

次のコードでは、アンビエント値と明示的な値は提供されていません (`{ controller = "Home", action = "Subscribe", id = 17, }`)。

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/WidgetController.cs?name=snippet2)]

上記のメソッドでは `/Home/Subscribe/17` が返されます。

`WidgetController` の次のコードでは、`/Widget/Subscribe/17` が返されます。

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/WidgetController.cs?name=snippet3)]

次のコードは、現在の要求と明示的な値のアンビエント値 (`{ action = "Edit", id = 17, }`) からコントローラーを提供します。

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/GadgetController.cs?name=snippet)]

上のコードでは以下の操作が行われます。

* `/Gadget/Edit/17` が返されます。
* <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Url> が <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> を取得します。
* <xref:Microsoft.AspNetCore.Mvc.UrlHelperExtensions.Action*>   
が、アクション メソッドの絶対パスを使用して URL を生成します。 URL には、指定した `action` 名と `route` 値が含まれます。

次のコードでは、現在の要求と明示的な値 (`{ page = "./Edit, id = 17, }`) からアンビエント値を提供します。

[!code-csharp[](routing/samples/3.x/RoutingSample/Pages/Index.cshtml.cs?name=snippet)]

前のコードでは、[Edit Razor Page]\(Razor ページの編集\) に次のページ ディレクティブが含まれている場合に、`url` を `/Edit/17` に設定します。

 `@page "{id:int}"`

[編集] ページに `"{id:int}"` ルート テンプレートが含まれない場合は、`url` は `/Edit?id=17` になります。

MVC の <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> の動作により、ここで説明するルールに加えて、複雑なレイヤーが追加されます。

* `IUrlHelper` は、常に現在の要求からルート値をアンビエント値として提供します。
* [IUrlHelper.Action](xref:Microsoft.AspNetCore.Mvc.UrlHelperExtensions.Action*) は、開発者がオーバーライドしない場合を除き、常に現在の `action` と `controller` ルート値を明示的な値としてコピーします。
* [IUrlHelper.Page](xref:Microsoft.AspNetCore.Mvc.UrlHelperExtensions.Page*) は、オーバーライドされない場合を除き、常に現在の `page` ルート値を明示的な値としてコピーします。 <!--by the user-->
* `IUrlHelper.Page` は、オーバーライドされない場合を除き、常に現在の `handler` ルート値を明示的な値として `null` にオーバーライドします。

ユーザーは、MVC が独自のルールに従っていないように見えるため、アンビエント値の動作の詳細にしばしば驚きます。 これまでの経緯および互換性の理由により、`action`、`controller`、`page`、`handler` などの特定のルート値には、独自の特殊な動作があります。

`LinkGenerator.GetPathByAction` と `LinkGenerator.GetPathByPage` の同等の機能では、互換性のために `IUrlHelper` のこの異常と同じ動作が行われます。

### <a name="url-generation-process"></a>URL の生成処理

一連の候補エンドポイントが見つかると、URL の生成アルゴリズムでは次が実行されます。

* エンドポイントが反復処理されます。
* 最初に成功した結果が返されます。

このプロセスの最初の手順は **ルート値の無効化** と呼ばれます。  ルート値の無効化は、アンビエント値からどのルート値を使用する必要があり、無視する必要があるかをルーティングが決定するプロセスです。 各アンビエント値が検討され、明示的な値と組み合わされるか、または無視されます。

アンビエント値の役割について一番わかりやすい考え方は、一部の一般的なケースでアプリケーション開発者の入力作業が省かれるということです。 従来、アンビエント値が役に立つシナリオは MVC に関連しています。

* 同じコントローラー内の別のアクションにリンクする場合、コントローラー名を指定する必要はありません。
* 同じ領域内の別のコントローラーにリンクする場合、領域名を指定する必要はありません。
* 同じアクション メソッドにリンクする場合は、ルート値を指定する必要はありません。
* アプリの別の部分にリンクする場合は、アプリのその部分には意味のないルート値は引き継ぎません。

`null` を返す `LinkGenerator` または `IUrlHelper` の呼び出しは、通常、ルート値の無効化について理解していないことが原因で発生します。 ルート値の無効化のトラブルシューティングを行うには、さらにルート値を明示的に指定して、これにより問題が解決されるかどうかを確認します。

ルート値の無効化は、アプリの URL スキームが階層的であり、階層が左から右に形成されていることを前提として機能します。 基本的なコントローラー ルート テンプレート `{controller}/{action}/{id?}` について考えてみましょう。これが実際にどのように動作するかを直感的に理解できます。 値に対する **変更** により、右側に表示されるすべてのルート値が **無効化** されます。 これには、階層に関する前提が反映されています。 アプリに `id` のアンビエント値があり、操作によって `controller` に対して異なる値が指定された場合、

* `{controller}` が `{id?}` の左側にあるため、`id` は再利用されません。

この原則を示すいくつかの例を次に示します。

* 明示的な値に `id` の値が含まれている場合、`id` のアンビエント値は無視されます。 `controller` と `action` のアンビエント値を使用できます。
* 明示的な値に `action` の値が含まれている場合、`action` のアンビエント値はすべて無視されます。 `controller` のアンビエント値を使用できます。 `action` の明示的な値が `action` のアンビエント値と異なる場合、`id` 値は使用されません。  `action` の明示的な値が `action` のアンビエント値と同じ場合、`id` 値を使用できます。
* 明示的な値に `controller` の値が含まれている場合、`controller` のアンビエント値はすべて無視されます。 `controller` の明示的な値が `controller` のアンビエント値と異なる場合、`action` と `id` の値は使用されません。 `controller` の明示的な値が `controller` のアンビエント値と同じ場合、`action` と `id` の値を使用できます。

このプロセスは、属性ルートと専用規則ルートが存在することでさらに複雑になります。 `{controller}/{action}/{id?}` などのコントローラーの規則ルートでは、ルート パラメーターを使用して階層が指定されます。 コントローラーと Razor Pages に対する[専用規則ルート](xref:mvc/controllers/routing#dcr)と[属性ルート](xref:mvc/controllers/routing#ar)の場合、

* ルート値の階層があります。
* テンプレートには表示されません。

このような場合は、URL の生成によって **必要な値** の概念が定義されます。 コントローラーおよび Razor Pages によって作成されたエンドポイントには、ルート値の無効化を機能させるために必要な値が指定されています。

ルート値の無効化アルゴリズムの詳細は次のとおりです。

* 必要な値の名前がルート パラメーターと組み合わされ、左から右に処理されます。
* 各パラメーターについて、アンビエント値と明示的な値が比較されます。
    * アンビエント値と明示的な値が同じ場合、プロセスは続行されます。
    * アンビエント値が存在し、明示的な値が存在しない場合は、URL を生成するときにアンビエント値が使用されます。
    * アンビエント値が存在せず、明示的な値が存在する場合は、そのアンビエント値とそれ以降のすべてのアンビエント値が拒否されます。
    * アンビエント値と明示的な値が存在し、2 つの値が異なる場合は、そのアンビエント値とそれ以降のすべてのアンビエント値が拒否されます。

この時点で、URL の生成操作はルート制約を評価する準備ができています。 許容可能な値のセットがパラメーターの既定値と組み合わされ、制約に提供されます。 すべての制約について合格した場合、操作が続行されます。

次に、**許容可能な値** を使用してルート テンプレートを展開できます。 ルート テンプレートは次のように処理されます。

* 左から右。
* 各パラメーターに、許容可能な値が代入されます。
* 次の特殊なケースがあります。
  * 許容可能な値がなく、パラメーターに既定値がある場合は、既定値が使用されます。
  * 許容可能な値がなく、パラメーターが省略可能な場合は、処理が続行されます。
  * 存在しない省略可能なパラメーターの右側にあるルート パラメーターのいずれかに値がある場合、操作は失敗します。
  * <!-- review default-valued parameters optional parameters --> 連続する既定値パラメーターと省略可能なパラメーターは、可能な場合、折りたたまれています。

ルートのセグメントと一致しない明示的に指定された値は、クエリ文字列に追加されます。 次の表は、ルート テンプレート `{controller}/{action}/{id?}` の使用時の結果をまとめたものです。

| アンビエント値                     | 明示的な値                        | 結果                  |
| ---------------------------------- | -------------------------------------- | ----------------------- |
| controller = "Home"                | action = "About"                       | `/Home/About`           |
| controller = "Home"                | controller = "Order", action = "About" | `/Order/About`          |
| controller = "Home", color = "Red" | action = "About"                       | `/Home/About`           |
| controller = "Home"                | action = "About", color = "Red"        | `/Home/About?color=Red` |

### <a name="problems-with-route-value-invalidation"></a>ルート値の無効化に関する問題

ASP.NET Core 3.0 の時点では、以前の ASP.NET Core バージョンで使用されている一部の URL 生成スキームが URL の生成においてうまく機能しません。 ASP.NET Core チームは、今後のリリースでこれらのニーズに対応する機能を追加することを計画しています。 当面は、従来のルーティングを使用することをお勧めします。

次のコードは、ルーティングでサポートされていない URL 生成スキームの例を示しています。

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupUnsupported.cs?name=snippet)]

上記のコードでは、`culture` ルート パラメーターがローカライズに使用されています。 `culture` パラメーターを常にアンビエント値として許容されるようにすることが望まれます。 しかし、必要な値の動作方法が理由で、`culture` パラメーターはアンビエント値として許容されません。

* `"default"` ルート テンプレートでは、`culture` ルート パラメーターは `controller` の左側にあるため、`controller` を変更しても `culture` は無効になりません。
* `"blog"` ルート テンプレートでは、`culture` ルート パラメーターは `controller` の右側にあると見なされ、必要な値に表示されます。

## <a name="configuring-endpoint-metadata"></a>エンドポイント メタデータの構成

次のリンクでは、エンドポイント メタデータの構成に関する情報を提供します。

* [エンドポイント ルーティングで CORS を有効にする](xref:security/cors#enable-cors-with-endpoint-routing)
* カスタム `[MinimumAgeAuthorize]` 属性を使用する [IAuthorizationPolicyProvider のサンプル](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/CustomPolicyProvider)
* [[Authorize] 属性を使用して認証をテストする](xref:security/authentication/identity#test-identity)
* <xref:Microsoft.AspNetCore.Builder.AuthorizationEndpointConventionBuilderExtensions.RequireAuthorization*>
* [[Authorize] 属性を使用してスキームを選択する](xref:security/authorization/limitingidentitybyscheme#selecting-the-scheme-with-the-authorize-attribute)
* [[Authorize] 属性を使用してポリシーを適用する](xref:security/authorization/policies#apply-policies-to-mvc-controllers)
* <xref:security/authorization/roles>

<a name="hostmatch"></a>

## <a name="host-matching-in-routes-with-requirehost"></a>RequireHost とルートが一致するホスト

<xref:Microsoft.AspNetCore.Builder.RoutingEndpointConventionBuilderExtensions.RequireHost*> では、指定したホストが必要であるという制約がルートに適用されます。 `RequireHost` または [[Host]](xref:Microsoft.AspNetCore.Routing.HostAttribute) パラメーターには、次のいずれかを指定できます。

* ホスト: `www.domain.com`。任意のポートの `www.domain.com` と一致します。
* ホストとワイルドカード: `*.domain.com`。任意のポートの `www.domain.com`、`subdomain.domain.com`、または `www.subdomain.domain.com` と一致します。
* ポート: `*:5000`。任意のホストのポート 5000 と一致します。
* ホストとポート: `www.domain.com:5000` または `*.domain.com:5000`。ホストとポートと一致します。

`RequireHost` または `[Host]` を使用して、複数のパラメーターを指定できます。 制約は、いずれかのパラメーターに対して有効なホストと一致します。 たとえば、`[Host("domain.com", "*.domain.com")]` は `domain.com`、`www.domain.com`、および `subdomain.domain.com` と一致します。

次のコードでは、`RequireHost` を使用して、指定したホストをルートに対して要求します。

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupRequireHost.cs?name=snippet)]

次のコードでは、コントローラー上の `[Host]` 属性を使用して、指定したホストのいずれかを要求します。

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/ProductController.cs?name=snippet)]

`[Host]` 属性がコントローラーとアクション メソッドの両方に適用される場合は、次のようになります。

* アクションの属性が使用されます。
* コントローラーの属性は無視されます。

## <a name="performance-guidance-for-routing"></a>ルーティングに関するパフォーマンス ガイダンス

ASP.NET Core 3.0 では、ほとんどのルーティングが更新され、パフォーマンスが向上しました。

アプリにパフォーマンス上の問題がある場合、多くの場合ルーティングが問題として疑われます。 ルーティングが疑われる理由は、コントローラーや Razor Pages などのフレームワークにより、フレームワーク内で費やされた時間がログ メッセージで報告されるためです。 コントローラーによって報告された時間と要求の合計時間の間に大きな違いがある場合、次のようになります。

* 開発者は、問題の発生源としてアプリ コードを排除します。
* ルーティングが原因であると考えるのが一般的です。

ルーティングは、数千のエンドポイントを使用してパフォーマンス テストされています。 一般的なアプリでは、大きすぎるだけでパフォーマンスの問題が発生する可能性はほとんどありません。 ルーティングのパフォーマンス低下の最も一般的な根本原因は、通常、正しく動作していないカスタム ミドルウェアです。

次のコード サンプルは、遅延の原因を絞り込むための基本的な手法を示したものです。

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupDelay.cs?name=snippet)]

ルーティングの時間は次のように計ります。

* 各ミドルウェアを、上記のコードに示されている時間を計るミドルウェアのコピーでインターリーブします。
* 計られた時間データをコードと関連付けるための一意の識別子を追加します。

これは、遅延が `10ms` を超えるなど顕著な場合に絞り込むための基本的な方法です。  `Time 1` から `Time 2` を引くことで、`UseRouting` ミドルウェア内で費やされた時間を報告します。

次のコードでは、前の時間を計るコードに対して、よりコンパクトなアプローチを使用します。

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupSW.cs?name=snippetSW)]

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupSW.cs?name=snippet)]

### <a name="potentially-expensive-routing-features"></a>潜在的にコストが高いルーティング機能

次の一覧に、基本的なルート テンプレートと比べて比較的コストが高いルーティング機能についての洞察を示します。

* 正規表現: 複雑な正規表現を作成すること、つまり少量の入力で長い実行時間を実現することができます。

* 複雑なセグメント (`{x}-{y}-{z}`): 
  * 通常の URL パス セグメントを解析するよりもかなりコストがかかります。
  * より多くの部分文字列が割り当てられることになります。
  * ASP.NET Core 3.0 ルーティング パフォーマンスの更新では、複雑なセグメントのロジックは更新されませんでした。

* 同期データ アクセス: 多くの複雑なアプリでは、ルーティングの一部としてデータベースにアクセスします。 ASP.NET Core 2.2 以前のルーティングでは、データベース アクセス ルーティングをサポートするための適切な拡張ポイントが提供されない場合があります。 たとえば、<xref:Microsoft.AspNetCore.Routing.IRouteConstraint> と <xref:Microsoft.AspNetCore.Mvc.ActionConstraints.IActionConstraint> は同期的です。 <xref:Microsoft.AspNetCore.Routing.MatcherPolicy> や <xref:Microsoft.AspNetCore.Routing.EndpointSelectorContext> などの拡張ポイントは非同期的です。

## <a name="guidance-for-library-authors"></a>ライブラリ作成者向けのガイダンス

このセクションでは、ルーティングを基盤とするライブラリ作成者向けのガイダンスを示します。 これらの詳細情報は、アプリの開発者が、ルーティングを拡張するライブラリとフレームワークを使用して優れたエクスペリエンスを実現できるようにすることを目的としています。

### <a name="define-endpoints"></a>エンドポイントを定義する

URL 照合にルーティングを使用するフレームワークを作成するには、まず <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> の上に構築されるユーザー エクスペリエンスを定義します。

<xref:Microsoft.AspNetCore.Routing.IEndpointRouteBuilder> の上に **構築します**。 これにより、ユーザーは、他の ASP.NET Core 機能と混同せずにフレームワークを構成できます。 すべての ASP.NET Core テンプレートには、ルーティングが含まれます。 ルーティングが存在し、ユーザーになじみのあるものとします。

```csharp
app.UseEndpoints(endpoints =>
{
    // Your framework
    endpoints.MapMyFramework(...);

    endpoints.MapHealthChecks("/healthz");
});
```

<xref:Microsoft.AspNetCore.Builder.IEndpointConventionBuilder> を実装する `MapMyFramework(...)` の呼び出しから、シールドの具象型を **返します**。 ほとんどのフレームワーク `Map...` メソッドは、このパターンに従います。 `IEndpointConventionBuilder` インターフェイスは、

* メタデータの構成可能性を許可します。
* さまざまな拡張メソッドの対象とされています。

独自の型を宣言すると、独自のフレームワーク固有の機能をビルダーに追加できます。 フレームワークで宣言されたビルダーをラップし、呼び出しを転送するのは問題ありません。

```csharp
app.UseEndpoints(endpoints =>
{
    // Your framework
    endpoints.MapMyFramework(...).RequireAuthorization()
                                 .WithMyFrameworkFeature(awesome: true);

    endpoints.MapHealthChecks("/healthz");
});
```

独自の <xref:Microsoft.AspNetCore.Routing.EndpointDataSource> を作成することを **検討します**。 `EndpointDataSource` は、エンドポイントのコレクションを宣言および更新するための低レベルのプリミティブです。 `EndpointDataSource` は、コントローラーと Razor Pages によって使用される強力な API です。

ルーティング テストには、更新されていないデータ ソースの[基本的な例](https://github.com/aspnet/AspNetCore/blob/master/src/Http/Routing/test/testassets/RoutingSandbox/Framework/FrameworkEndpointDataSource.cs#L17)があります。

既定では、`EndpointDataSource` の登録を **試行しないでください**。 <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> にフレームワークを登録するようユーザーに要求してください。 ルーティングの原理では、既定では何も含まれておらず、`UseEndpoints` がエンドポイントを登録する場所です。

### <a name="creating-routing-integrated-middleware"></a>ルーティング統合ミドルウェアを作成する

メタデータ型をインターフェイスとして定義することを **検討します**。

クラスおよびメソッドの属性としてメタデータ型を **使用できるようにします**。

[!code-csharp[](routing/samples/3.x/RoutingSample/ICoolMetadata.cs?name=snippet2)]

コントローラーや Razor Pages などのフレームワークでは、型およびメソッドへのメタデータ属性の適用がサポートされています。 メタデータ型を宣言する場合:

* [属性](/dotnet/csharp/programming-guide/concepts/attributes/)としてアクセスできるようにします。
* ほとんどのユーザーが属性の適用に精通しています。

メタデータ型をインターフェイスとして宣言すると、柔軟性の高いレイヤーがさらに追加されます。

* インターフェイスが構成可能です。
* 開発者は、複数のポリシーを結合する独自の型を宣言できます。

次の例に示すように、メタデータを **オーバーライドできるようにします**。

[!code-csharp[](routing/samples/3.x/RoutingSample/ICoolMetadata.cs?name=snippet)]

これらのガイドラインに従う最善の方法は、**マーカーのメタデータ** を定義しないようにすることです。

* メタデータ型が存在するかどうかを確認するだけで終わらせません。
* メタデータのプロパティを定義し、プロパティを確認します。

メタデータ コレクションが順序付けされ、優先順位によるオーバーライドがサポートされます。 コントローラーの場合、アクション メソッドのメタデータが最も限定的です。

ルーティングを使用する場合もしない場合もミドルウェアが **役立つようにします**。

```csharp
app.UseRouting();

app.UseAuthorization(new AuthorizationPolicy() { ... });

app.UseEndpoints(endpoints =>
{
    // Your framework
    endpoints.MapMyFramework(...).RequireAuthorization();
});
```

このガイドラインの例として、`UseAuthorization` ミドルウェアを考えてみましょう。 この承認ミドルウェアを使用すると、フォールバック ポリシーを渡すことができます。 <!-- shown where?  (shown here) --> フォールバック ポリシーは、指定されている場合、次の両方に適用されます。

* 指定されたポリシーのないエンドポイント。
* エンドポイントに一致しない要求。

これにより、承認ミドルウェアはルーティングのコンテキストの外部で役に立ちます。 承認ミドルウェアは、従来のミドルウェア プログラミングに使用できます。

[!INCLUDE[](~/includes/dbg-route.md)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

ルーティングでは、要求 URI をエンドポイントにマッピングし、受信要求をそれらのエンドポイントに配布します。 ルートはアプリに定義され、アプリの起動時に構成されます。 ルートは、要求に含まれている URL から値を任意で抽出できます。その値を要求処理に利用できます。 ルーティングでは、アプリからのルート情報を利用し、エンドポイントにマッピングする URL を生成することもできます。

ASP.NET Core 2.2 で最新のルーティング シナリオを使用するには、`Startup.ConfigureServices` で[互換性バージョン](xref:mvc/compatibility-version)を MVC サービス登録に指定します。

```csharp
services.AddMvc()
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
```

<xref:Microsoft.AspNetCore.Mvc.MvcOptions.EnableEndpointRouting> オプションでは、ルーティングで内部的にエンドポイント ベースのロジックを使用するか、ASP.NET Core 2.1 以前の <xref:Microsoft.AspNetCore.Routing.IRouter> ベースのロジックを使用する必要があるかどうかを決定します。 互換性バージョンが 2.2 以降に設定されている場合、既定値は `true` となります。 以前のルーティング ロジックを使用する場合は、値を `false` に設定します。

```csharp
// Use the routing logic of ASP.NET Core 2.1 or earlier:
services.AddMvc(options => options.EnableEndpointRouting = false)
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
```

<xref:Microsoft.AspNetCore.Routing.IRouter> ベースのルーティングの詳細については、[このトピックの ASP.NET Core 2.1 バージョン](?view=aspnetcore-2.1)を参照してください。

> [!IMPORTANT]
> 本文では、ASP.NET Core ルーティングについて詳しく取り上げます。 ASP.NET Core MVC ルーティングの詳細については、「<xref:mvc/controllers/routing>」を参照してください。 Razor Pages のルーティング規則については、「<xref:razor-pages/razor-pages-conventions>」を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/routing/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="routing-basics"></a>ルーティングの基本

ほとんどのアプリでは、URL を読みやすくてわかりやすいものにするために、基本的なでわかりやすいルーティング スキームを選択する必要があります。 既定の規則ルート `{controller=Home}/{action=Index}/{id?}`:

* 基本的でわかりやすいルーティング スキームをサポートしています。
* UI ベースのアプリの便利な開始点となります。

開発者は一般的に、特殊な状況でのアプリのトラフィックが多い領域に対しては、[属性ルーティング](xref:mvc/controllers/routing#attribute-routing)または専用規則ルートを使用して簡易ルートをさらに追加します。 特殊な状況の例として、ブログや eコマース エンドポイントがあります。

Web API では、属性ルーティングを使用して、HTTP 動詞で操作を表現するリソースのセットとしてアプリの機能をモデル化する必要があります。 つまり、同じ論理リソース上の多くの操作 (たとえば GET や POST) で、同じ URL が使用されます。 属性ルーティングでは、API のパブリック エンドポイント レイアウトを慎重に設計するために必要となるコントロールのレベルが提供されます。

Razor Pages アプリでは、既定の規則ルーティングを使用して、アプリの *Pages* フォルダーの名前付きリソースを提供します。 Razor Pages のルーティング動作をカスタマイズできる追加の規則を使用できます。 詳細については、次のトピックを参照してください。 <xref:razor-pages/index> および <xref:razor-pages/razor-pages-conventions>

URL 生成サポートを使用すると、アプリを相互にリンクする URL をハード コーディングすることなくアプリを開発できます。 このサポートにより、基本的なルーティング構成を使用して作業を開始し、アプリのリソース レイアウトが決まった後でルートを変更することができます。

ルーティングでは *エンドポイント* (`Endpoint`) を使用して、アプリの論理エンドポイントを表します。

エンドポイントでは、要求と、任意のメタデータのコレクションを処理するデリゲートが定義されます。 メタデータは、各エンドポイントにアタッチされている構成とポリシーに基づいて横断的な関心事を実装するために使用されます。

ルーティング システムには次の特性があります。

* ルート テンプレート構文は、トークン化されたルート パラメーターでルートを定義するために使用されます。
* 規則スタイルおよび属性スタイルのエンドポイント構成が許可されます。
* <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> は、URL パラメーターに特定のエンドポイント制約で有効な値が含まれているかどうかを確認するために使用されます。
* MVC/Razor Pages などのアプリ モデルでは、ルーティング シナリオの予測可能な実装を持つ、すべてのエンドポイントを登録します。
* ルーティングの実装では、必要に応じて、ミドルウェア パイプラインでのルーティングに関する決定を行います。
* ルーティング ミドルウェアの後に表示されるミドルウェアでは、特定の要求 URI に関するルーティング ミドルウェアのエンドポイント決定の結果を検査できます。
* ミドルウェア パイプラインの任意の場所でアプリのすべてのエンドポイントを列挙することができます。
* アプリではルーティングを使用し、エンドポイント情報に基づいて URL (リダイレクトやリンクなど) を生成できます。そのため、URL をハードコーディングする必要がなく、保守管理が容易になります。
* URL の生成は、任意の拡張機能をサポートする、アドレスに基づきます。

  * リンク ジェネレーター API (<xref:Microsoft.AspNetCore.Routing.LinkGenerator>) は、URI を生成するために[依存関係挿入 (DI)](xref:fundamentals/dependency-injection) を使用して任意の場所で解決できます。
  * リンク ジェネレーター API を DI で使用できない場合、<xref:Microsoft.AspNetCore.Mvc.IUrlHelper> で URL を構築するためのメソッドが提供されます。

> [!NOTE]
> ASP.NET Core 2.2 のエンドポイント ルーティングのリリースでは、エンドポイント リンクは、MVC/Razor Pages アクションおよびページに制限されます。 エンドポイント リンク機能の拡張は、今後のリリースで予定されています。

ルーティングは <xref:Microsoft.AspNetCore.Builder.RouterMiddleware> クラスにより[ミドルウェア](xref:fundamentals/middleware/index) パイプラインに接続されます。 [ASP.NET Core MVC](xref:mvc/overview) では、ミドルウェア パイプラインにその構成の一部としてルーティングが追加され、MVC および Razor Pages アプリでのルーティングが処理されます。 スタンドアロン コンポーネントとしてルーティングを利用する方法については、「[ルーティング ミドルウェアの使用](#use-routing-middleware)」セクションを参照してください。

### <a name="url-matching"></a>URL 一致

URL 一致というプロセスでは、ルーティングによって、受信要求が *エンドポイント* に配布されます。 このプロセスは URL パスのデータに基づきますが、要求内のあらゆるデータを考慮することもできます。 アプリの規模を大きくしたり、より複雑にしたりするとき、要求を個別のハンドラーに配布する機能が鍵となります。

エンドポイント ルーティングのルーティング システムでは、配布に関するすべての決定が行われます。 ミドルウェアでは選択されたエンドポイントに基づいてポリシーが適用されるため、セキュリティ ポリシーの適用や配布に影響する可能性のある決定は、ルーティング システム内で行うことが重要です。

エンドポイントのデリゲートが実行されると、それまでに実行された要求処理に基づいて、[RouteContext.RouteData](xref:Microsoft.AspNetCore.Routing.RouteContext.RouteData) のプロパティが適切な値に設定されます。

[RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values*) は、ルートから生成された *ルート値* のディクショナリです。 この値は通常、URL のトークン化で決定されます。この値を利用してユーザー入力を受け取ったり、アプリ内で決定をさらに配布したりできます。

[RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) は、一致したルートに関連する追加データのプロパティ バッグです。 一致したルートに基づいてアプリで決定を行えるように、<xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*> が提供されます。これは状態データと各ルートの関連付けをサポートするためのものです。 この値は開発者が定義するものです。ルーティングの動作に影響を与えることは **ありません**。 また、[RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) に一時退避される値はどのような型でも構いません。一方、[RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values) の場合は、文字列間で変換できる必要があります。

[RouteData.Routers](xref:Microsoft.AspNetCore.Routing.RouteData.Routers) は、過去に要求に一致したルートの一覧です。 ルートはルートの中に入れ子にすることができます。 <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> プロパティは、結果的に一致をもたらしたルートの論理ツリーを通るパスを表します。 一般的に、<xref:Microsoft.AspNetCore.Routing.RouteData.Routers> の最初の項目はルート コレクションであり、これは URL 生成に使用するものです。 <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> の最後の項目は、一致したルート ハンドラーです。

<a name="lg"></a>

### <a name="url-generation-with-linkgenerator"></a>LinkGenerator による URL の生成

URL 生成は、ルーティングにおいて、一連のルート値に基づいて URL パスを作成するプロセスです。 これにより、エンドポイントとそれにアクセスする URL を論理的に分離できます。

エンドポイント ルーティングには、リンク ジェネレーター API (<xref:Microsoft.AspNetCore.Routing.LinkGenerator>) が含まれています。 <xref:Microsoft.AspNetCore.Routing.LinkGenerator> は、[DI](xref:fundamentals/dependency-injection) から取得できるシングルトン サービスです。 API は、実行中の要求のコンテキスト外で使用することができます。 MVC の <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> と、[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)、HTML ヘルパー、[アクション結果](xref:mvc/controllers/actions)など、<xref:Microsoft.AspNetCore.Mvc.IUrlHelper> に依存するシナリオではリンク ジェネレーターを使用して、リンク生成機能を提供します。

リンク ジェネレーターは、*アドレス* と *アドレス スキーム* の概念に基づいています。 アドレス スキームは、リンク生成で考慮すべきエンドポイントを決定する方法です。 たとえば、MVC/Razor Pages からの、多くのユーザーに馴染みのあるルート名やルート値シナリオは、アドレス スキームとして実装されます。

リンク ジェネレーターでは、次の拡張メソッドを使用して、MVC/Razor Pages アクションおよびページにリンクできます。

* <xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetPathByAction*>
* <xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetUriByAction*>
* <xref:Microsoft.AspNetCore.Routing.PageLinkGeneratorExtensions.GetPathByPage*>
* <xref:Microsoft.AspNetCore.Routing.PageLinkGeneratorExtensions.GetUriByPage*>

これらのメソッドのオーバーロードでは、`HttpContext` を含む引数が受け入れられます。 これらのメソッドは `Url.Action` および `Url.Page` と機能的には同等ですが、柔軟性とオプションがさらに提供されます。

`GetPath*` メソッドは、絶対パスを含む URI を生成するという点で `Url.Action` および `Url.Page` に最も似ています。 `GetUri*` メソッドでは常に、スキームとホストを含む絶対 URI が生成されます。 `HttpContext` を受け入れるメソッドでは、実行中の要求のコンテキストで URI が生成されます。 実行中の要求からのアンビエント ルート値、URL ベース パス、スキーム、およびホストは、オーバーライドされない限り使用されます。

<xref:Microsoft.AspNetCore.Routing.LinkGenerator> はアドレスと共に呼び出されます。 URI の生成は、次の 2 つの手順で行われます。

1. アドレスは、そのアドレスと一致するエンドポイントのリストにバインドされます。
1. 各エンドポイントの `RoutePattern` は、指定された値と一致するルート パターンが見つかるまで評価されます。 結果の出力は、リンク ジェネレーターに指定された他の URI 部分と結合され、返されます。

<xref:Microsoft.AspNetCore.Routing.LinkGenerator> によって提供されるメソッドでは、すべての種類のアドレスの標準的なリンク生成機能がサポートされます。 リンク ジェネレーターを使用する最も便利な方法は、特定のアドレスの種類の操作を実行する拡張メソッドを使用することです。

| 拡張メソッド   | 説明                                                         |
| ------------------ | ------------------------------------------------------------------- |
| <xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetPathByAddress*> | 指定された値に基づき、絶対パスを含む URI を生成します。 |
| <xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetUriByAddress*> | 指定された値に基づき、絶対 URI を生成します。             |

> [!WARNING]
> <xref:Microsoft.AspNetCore.Routing.LinkGenerator> メソッド呼び出しによる次の影響に注意してください。
>
> * 受信要求の `Host` ヘッダーが確認されないアプリ構成では、`GetUri*` 拡張メソッドは注意して使用してください。 受信要求の `Host` ヘッダーが確認されていない場合、信頼されていない要求入力を、ビュー/ページの URI でクライアントに送り返すことができます。 すべての運用アプリで、`Host` ヘッダーを既知の有効な値と照らし合わせて確認するようにサーバーを構成することをお勧めします。
>
> * ミドルウェアで `Map` または `MapWhen` と組み合わせて、<xref:Microsoft.AspNetCore.Routing.LinkGenerator> を使用する場合は注意してください。 `Map*` では、実行中の要求の基本パスが変更され、リンク生成の出力に影響します。 すべての <xref:Microsoft.AspNetCore.Routing.LinkGenerator> API で基本パスを指定することができます。 リンク生成への `Map*` の影響を元に戻すための空の基本パスを必ず指定してください。

## <a name="differences-from-earlier-versions-of-routing"></a>以前のバージョンのルーティングとの相違点

ASP.NET Core 2.2 以降でのエンドポイント ルーティングと、ASP.NET Core での以前のバージョンのルーティングには、次のようないくつかの違いがあります。

* エンドポイント ルーティング システムでは、<xref:Microsoft.AspNetCore.Routing.Route> からの継承を含む、<xref:Microsoft.AspNetCore.Routing.IRouter> ベースの拡張機能がサポートされません。

* エンドポイント ルーティングでは [WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim) がサポートされません。 互換性 shim を引き続き使用するには、2.1 の[互換性バージョン](xref:mvc/compatibility-version) (`.SetCompatibilityVersion(CompatibilityVersion.Version_2_1)`) を使用します。

* エンドポイント ルーティングでは、規則ルートを使用する場合に生成される URI の大文字と小文字の指定の動作が異なります。

  次の既定のルート テンプレートについて考えてみます。

  ```csharp
  app.UseMvc(routes =>
  {
      routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
  });
  ```

  次のルートを使用して、アクションへのリンクを生成するとします。

  ```csharp
  var link = Url.Action("ReadPost", "blog", new { id = 17, });
  ```

  <xref:Microsoft.AspNetCore.Routing.IRouter> ベースのルーティングでは、このコードで `/blog/ReadPost/17` の URI が生成され、指定されたルート値の大文字と小文字の指定が優先されます。 ASP.NET Core 2.2 以降のエンドポイント ルーティングでは、`/Blog/ReadPost/17` ("Blog" の先頭は大文字) が生成されます。 エンドポイント ルーティングでは `IOutboundParameterTransformer` インターフェイスが提供されます。それを使用して、この動作をグローバルにカスタマイズしたり、URL のマッピングで異なる規則を適用したりすることができます。

  詳細については、「[パラメーター トランスフォーマー参照](#parameter-transformer-reference)」セクションを参照してください。

* 存在しないコントローラー/アクションまたはページへのリンクを試みる場合、規則ルートがある MVC/Razor Pages によって使用されるリンク生成の動作は異なります。

  次の既定のルート テンプレートについて考えてみます。

  ```csharp
  app.UseMvc(routes =>
  {
      routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
  });
  ```

  次のように既定のテンプレートを使用して、アクションへのリンクを生成するとします。

  ```csharp
  var link = Url.Action("ReadPost", "Blog", new { id = 17, });
  ```

  `IRouter` ベースのルーティングでは、`BlogController` が存在しない場合や `ReadPost` アクション メソッドがない場合でも、結果は常に `/Blog/ReadPost/17` となります。 予想どおり、アクション メソッドが存在する場合は、ASP.NET Core 2.2 以降のエンドポイント ルーティングで `/Blog/ReadPost/17` が生成されます。 *しかし、アクションが存在しない場合は、エンドポイント ルーティングで空の文字列が生成されます。* 概念的には、エンドポイント ルーティングでは、アクションが存在しない場合はエンドポイントが存在するとは見なされません。

* エンドポイント ルーティングで使用する場合は、リンク生成の *アンビエント値の無効化アルゴリズム* の動作が異なります。

  *アンビエント値の無効化* は、現在実行中の要求からのルート値 (アンビエント値) のうち、どちらがリンク生成操作で使用できのかを判断するアルゴリズムです。 規則ルーティングでは、異なるアクションへのリンク時に余分なルート値が常に無効化されました。 属性ルーティングは、ASP.NET Core 2.2 のリリースより前ではこのように動作しませんでした。 以前のバージョンの ASP.NET Core では、同じルート パラメーター名を使用する別のアクションへのリンクでリンク生成エラーが発生しました。 ASP.NET Core 2.2 以降では、両方の形式のルーティングで、別のアクションへのリンク時に値が無効化されます。

  ASP.NET Core 2.1 以前での次の例について考えてみます。 別のアクション (または別のページ) にリンクする場合、好ましくない方法でルート値が再利用される可能性があります。

  */Pages/Store/Product.cshtml* の場合:

  ```cshtml
  @page "{id}"
  @Url.Page("/Login")
  ```

  */Pages/Login.cshtml* の場合:

  ```cshtml
  @page "{id?}"
  ```

  ASP.NET Core 2.1 以前で URI が `/Store/Product/18` である場合、`@Url.Page("/Login")` によって Store/Info ページに生成されるリンクは `/Login/18` となります。 リンク先が完全にアプリの異なる部分であっても、18 という `id` の値が再利用されます。 `/Login` ページのコンテキスト内の `id` ルート値は、商品 ID 値ではなく、ユーザー ID 値であると考えられます。

  ASP.NET Core 2.2 以降でのエンドポイント ルーティングでは、結果は `/Login` となります。 リンク先が異なるアクションやページである場合、アンビエント値は再利用されません。

* ラウンド トリップ ルート パラメーターの構文: 二重アスタリスク (`**`) キャッチオール パラメーター構文を使用する場合、スラッシュはエンコードされません。

  リンクの生成中に、ルーティング システムでは、スラッシュを除く二重アスタリスク (`**`) キャッチオール パラメーター (`{**myparametername}` など) でキャプチャされた値をエンコードします。 二重アスタリスク キャッチオールは、ASP.NET Core 2.2 以降の `IRouter` ベース ルーティングでサポートされます。

  以前のバージョンの ASP.NET Core での単一のアスタリスク キャッチオール パラメーター構文 (`{*myparametername}`) は引き続きサポートされ、スラッシュはエンコードされます。

  | ルート              | 以下で生成されるリンク<br>`Url.Action(new { category = "admin/products" })`&hellip; |
  | ------------------ | --------------------------------------------------------------------- |
  | `/search/{*page}`  | `/search/admin%2Fproducts` (スラッシュはエンコードされます)             |
  | `/search/{**page}` | `/search/admin/products`                                              |

### <a name="middleware-example"></a>ミドルウェアの例

次の例では、ミドルウェアで <xref:Microsoft.AspNetCore.Routing.LinkGenerator> API を使用して、商品をリストするアクション メソッドへのリンクを作成します。 リンク ジェネレーターは、クラスに挿入し、`GenerateLink` を呼び出すことで、アプリのどのクラスでも使用できます。

```csharp
using Microsoft.AspNetCore.Routing;

public class ProductsLinkMiddleware
{
    private readonly LinkGenerator _linkGenerator;

    public ProductsLinkMiddleware(RequestDelegate next, LinkGenerator linkGenerator)
    {
        _linkGenerator = linkGenerator;
    }

    public async Task InvokeAsync(HttpContext httpContext)
    {
        var url = _linkGenerator.GetPathByAction("ListProducts", "Store");

        httpContext.Response.ContentType = "text/plain";

        await httpContext.Response.WriteAsync($"Go to {url} to see our products.");
    }
}
```

### <a name="create-routes"></a>ルートを作成する

ほとんどのアプリは、<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> を呼び出すか、<xref:Microsoft.AspNetCore.Routing.IRouteBuilder> で定義されている同様の拡張メソッドの 1 つを呼び出してルートを作成します。 いずれの <xref:Microsoft.AspNetCore.Routing.IRouteBuilder> 拡張メソッドでも、<xref:Microsoft.AspNetCore.Routing.Route> のインスタンスが作成され、それがルート コレクションに追加されます。

<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> では、ルート ハンドラー パラメーターは受け入れられません。 <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> は、<xref:Microsoft.AspNetCore.Routing.RouteBuilder.DefaultHandler*> によって処理されるルートを追加するだけです。 MVC でのルーティングの詳細については、「<xref:mvc/controllers/routing>」を参照してください。

次のコード例は、一般的な ASP.NET Core MVC ルート定義によって使用される <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 呼び出しの例です。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

このテンプレートでは URL パスが照合され、ルート値が抽出されます。 たとえば、`/Products/Details/17` というパスでは、`{ controller = Products, action = Details, id = 17 }` というルート値が生成されます。

ルート値は、URL パスをセグメントに分割し、各セグメントと、ルート テンプレートの *ルート パラメーター* 名を照合することで決定されます。 ルート パラメーターが指定されます。 パラメーターは、中かっこ `{ ... }` でパラメーター名を囲むことで定義されます。

上のテンプレートでは URL パス `/` の照合も行い、`{ controller = Home, action = Index }` という値を生成します。 これは、ルート パラメーターの `{controller}` と `{action}` に既定値が与えられ、`id` ルート パラメーターが任意であるためです。 ルート パラメーター名の後の等号 (`=`) とそれに続く値により、パラメーターの既定値が定義されます。 ルート パラメーター名の後の疑問符 (`?`) により、オプションのパラメーターが定義されます。

既定値のルート パラメーターはルートが一致するとルート値を *常に* 生成します。 省略可能なパラメーターでは、対応する URL パス セグメントがない場合、ルート値は生成されません。 ルート テンプレートのシナリオと構文の詳しい説明については、「[ルート テンプレート参照](#route-template-reference)」セクションを参照してください。

次の例では、ルート パラメーター定義 `{id:int}` により、`id` ルート パラメーターの[ルート制約](#route-constraint-reference)が定義されます。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id:int}");
```

このテンプレートは `/Products/Details/Apples` ではなく、`/Products/Details/17` のような URL パスを照合します。 ルート制約は <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> を実装し、ルート値を調べて正しいことを確認します。 この例では、ルート値 `id` は整数に変換できなければなりません。 フレームワークによって提供されるルート制約については、「[ルート制約参照](#route-constraint-reference)」を参照してください。

<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> の追加オーバーロードは、`constraints`、`dataTokens`、`defaults` の値を受け取ります。 このようなパラメーターは一般的に、匿名型のオブジェクトを渡し、匿名型のプロパティ名がルート パラメーター名と一致するというような使われ方をします。

次の <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 例では、同等のルートを作成します。

```csharp
routes.MapRoute(
    name: "default_route",
    template: "{controller}/{action}/{id?}",
    defaults: new { controller = "Home", action = "Index" });

routes.MapRoute(
    name: "default_route",
    template: "{controller=Home}/{action=Index}/{id?}");
```

> [!TIP]
> 制約と既定値を定義するインライン構文は単純なルートの場合に便利です。 しかし、インライン構文でサポートされていない、データ トークンなどのシナリオがあります。

次の例では、その他のいくつかのシナリオを示します。

```csharp
routes.MapRoute(
    name: "blog",
    template: "Blog/{**article}",
    defaults: new { controller = "Blog", action = "ReadArticle" });
```

上記のテンプレートでは `/Blog/All-About-Routing/Introduction` のような URL パスを照合し、値 `{ controller = Blog, action = ReadArticle, article = All-About-Routing/Introduction }` を抽出します。 `controller` と `action` の既定のルート値は、テンプレートに対応するルート パラメーターがなくても、ルートにより生成されます。 既定値はルート テンプレートで指定できます。 `article` ルート パラメーターは、ルート パラメーター名の前に二重アスタリスク (`**`) があるときに、*キャッチオール* として定義されます。 キャッチオール ルート パラメーターは、URL パスの残りの部分をキャプチャします。空の文字列も照合できます。

次の例では、ルート制約とデータ トークンを追加します。

```csharp
routes.MapRoute(
    name: "us_english_products",
    template: "en-US/Products/{id}",
    defaults: new { controller = "Products", action = "Details" },
    constraints: new { id = new IntRouteConstraint() },
    dataTokens: new { locale = "en-US" });
```

上記のテンプレートでは `/en-US/Products/5` のような URL パスを照合し、値 `{ controller = Products, action = Details, id = 5 }` とデータ トークン `{ locale = en-US }` を抽出します。

![ローカル変数ウィンドウ トークン](routing/_static/tokens.png)

### <a name="route-class-url-generation"></a>ルート クラス URL の生成

<xref:Microsoft.AspNetCore.Routing.Route> クラスは、一連のルート値をそのルート テンプレートと組み合わせる方法でも URL を生成できます。 論理的には、URL パスの照合とは逆のプロセスになります。

> [!TIP]
> URL 生成を理解する方法として、どのような URL を生成したいのかを考えてください。そして、ルート テンプレートがその URL にどのように照合されるのかを考えてください。 どのような値が生成されるでしょうか。 <xref:Microsoft.AspNetCore.Routing.Route> クラスにおける URL 生成のしくみと大体同じになります。

次の例では、一般的な ASP.NET Core MVC の既定のルートを使用します。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

ルート値が `{ controller = Products, action = List }` の場合、URL `/Products/List` が生成されます。 ルート値が対応ルート パラメーターの代替となり、URL パスが作られます。 `id` は任意のルート パラメーターであるため、URL は `id` の値なしで正常に生成されます。

ルート値が `{ controller = Home, action = Index }` の場合、URL `/` が生成されます。 指定されたルート値は既定値と一致し、その既定値に対応するセグメントを省略しても問題は発生しません。

生成された両方の URL では、次のルート定義 (`/Home/Index` と `/`) でラウンドトリップされ、URL の生成に使用された同じルート値が生成されます。

> [!NOTE]
> アプリで ASP.NET Core MVC が使用される場合、ルーティングを直接呼び出す代わりに、<xref:Microsoft.AspNetCore.Mvc.Routing.UrlHelper> を使用して URL を生成する必要があります。

URL 生成の詳細については、「[URL 生成参照](#url-generation-reference)」セクションを参照してください。

## <a name="use-routing-middleware"></a>ルーティング ミドルウェアの使用

アプリのプロジェクト ファイルの [Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)を参照します。

`Startup.ConfigureServices` でサービス コンテナーにルーティングを追加した例:

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_ConfigureServices&highlight=3)]

ルートは `Startup.Configure` メソッドで設定する必要があります。 サンプル アプリでは次の API を使用します。

* <xref:Microsoft.AspNetCore.Routing.RouteBuilder>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*>:HTTP GET 要求のみを照合します。
* <xref:Microsoft.AspNetCore.Builder.RoutingBuilderExtensions.UseRouter*>

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_RouteHandler)]

下の表は、応答と所与の URI をまとめたものです。

| URI                    | 応答                                          |
| ---------------------- | ------------------------------------------------- |
| `/package/create/3`    | Hello! Route values: [operation, create], [id, 3] |
| `/package/track/-3`    | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/-3/`   | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/`      | 要求はフォール スルーし、一致するものはありません。              |
| `GET /hello/Joe`       | Hi, Joe!                                          |
| `POST /hello/Joe`      | 要求はフォール スルーし、HTTP GET のみが一致します。 |
| `GET /hello/Joe/Smith` | 要求はフォール スルーし、一致するものはありません。              |

このフレームワークでは、ルートを作成するために拡張メソッド セット (<xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions>) が提供されます。

* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareVerb*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>

`Map[Verb]` メソッドは制約を利用し、メソッド名の HTTP Verb にルートを制限します。 たとえば、 <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*> や <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>を参照してください。

## <a name="route-template-reference"></a>ルート テンプレート参照

中括弧 (`{ ... }`) 内のトークンは *ルート パラメーター* を定義します。ルートが一致した場合、これがバインドされます。 1 つのルート セグメントで複数のルート パラメーターを定義できますが、リテラル値で区切る必要があります。 たとえば、`{controller=Home}{action=Index}` は有効なルートになりません。`{controller}` と `{action}` の間にリテラル値がないためです。 このようなルート パラメーターには名前を付ける必要があります。付加的な属性を指定することもあります。

ルート パラメーター以外のリテラル テキスト (`{id}` など) とパス区切り `/` は URL のテキストに一致する必要があります。 テキスト照合は復号された URL パスを基盤とし、大文字と小文字が区別されます。 リテラル ルート パラメーターの区切り記号 (`{` または `}`) を照合するには、文字を繰り返して区切り記号をエスケープします (`{{` または `}}`)。

任意のファイル拡張子が付いたファイル名のキャプチャを試行する URL パターンには、追加の考慮事項があります。 たとえば、テンプレート `files/{filename}.{ext?}` について考えてみます。 `filename` と `ext` の両方の値が存在するときに、両方の値が入力されます。 URL に `filename` の値だけが存在する場合、末尾のピリオド (`.`) は任意であるため、このルートは一致となります。 次の URL はこのルートに一致します。

* `/files/myFile.txt`
* `/files/myFile`

ルート パラメーターのプレフィックスとしてアスタリスク (`*`) または二重アスタリスク (`**`) を使用し、URI の残りの部分にバインドできます。 これらは、*キャッチオール* パラメーターと呼ばれています。 たとえば、`blog/{**slug}` は、`/blog` で始まり、その後に (`slug` ルート値に割り当てられる) 任意の値が続くあらゆる URI に一致します。 キャッチオール パラメーターは空の文字列に一致することもあります。

キャッチオール パラメーターでは、パス区切り (`/`) 文字を含め、URL の生成にルートが使用されるときに適切な文字がエスケープされます。 たとえば、ルート値が `{ path = "my/path" }` のルート `foo/{*path}` では、`foo/my%2Fpath` が生成されます。 エスケープされたスラッシュに注意してください。 パス区切り文字をラウンドトリップさせるには、`**` ルート パラメーター プレフィックスを使用します。 `{ path = "my/path" }` のルート `foo/{**path}` では、`foo/my/path` が生成されます。

ルート パラメーターには、*既定値* が含まれることがあります。パラメーター名の後に既定値を指定し、等号 (`=`) で区切ることで指定されます。 たとえば、`{controller=Home}` では、`controller` の既定値として `Home` が定義されます。 パラメーターの URL に値がない場合、既定値が使用されます。 ルート パラメーターは、`id?` のように、パラメーター名の終わりに疑問符 (`?`) を追加することでオプションとして扱われます。 オプション値と既定ルート パラメーターの違いは、既定値を含むルート パラメーターは常に値を生成するのに対し、オプションのパラメーターには、要求 URL によって値が指定されたときにのみ値が与えられることにあります。

ルート パラメーターには、URL からバインドされるルート値に一致しなければならないという制約が含まれることがあります。 コロン (`:`) と制約名をルート パラメーター名の後に追加すると、ルート パラメーターの *インライン制約* が指定されます。 その制約で引数が要求される場合、制約名の後にかっこ (`(...)`) で囲まれます。 別のコロン (`:`) と制約名を追加することで、複数のインライン制約を指定できます。

制約名と引数が <xref:Microsoft.AspNetCore.Routing.IInlineConstraintResolver> サービスに渡され、URL 処理で使用する <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> のインスタンスが作成されます。 たとえば、ルート テンプレート `blog/{article:minlength(10)}` によって、制約 `minlength` と引数 `10` が指定されます。 ルート制約の詳細とこのフレームワークによって指定される制約のリストについては、「[ルート制約参照](#route-constraint-reference)」セクションを参照してください。

ルート パラメーターには、リンクを生成し、ページおよびアクションを URL と一致させるときにパラメーターの値を変換する、パラメーター トランスフォーマーが含まれている場合もあります。 制約と同様に、パラメーター トランスフォーマーをルート パラメーターにインラインで追加することができます。その場合、ルート パラメーター名の後にコロン (`:`) とトランスフォーマー名を追加します。 たとえば、ルート テンプレート `blog/{article:slugify}` では、`slugify` トランスフォーマーが指定されます。 パラメーター トランスフォーマーの詳細については、「[パラメーター トランスフォーマー参照](#parameter-transformer-reference)」セクションを参照してください。

次の表は、ルート テンプレートの例とその動作をまとめたものです。

| ルート テンプレート                           | 一致する URI の例    | 要求 URI&hellip;                                                    |
| ---------------------------------------- | ----------------------- | -------------------------------------------------------------------------- |
| `hello`                                  | `/hello`                | 単一パス `/hello` にのみ一致します。                                     |
| `{Page=Home}`                            | `/`                     | 一致し、`Page` が `Home` に設定されます。                                         |
| `{Page=Home}`                            | `/Contact`              | 一致し、`Page` が `Contact` に設定されます。                                      |
| `{controller}/{action}/{id?}`            | `/Products/List`        | `Products` コントローラーと `List` アクションにマッピングされます。                       |
| `{controller}/{action}/{id?}`            | `/Products/Details/123` | `Products` コントローラーと `Details` アクションにマッピングされます (`id` は 123 に設定されます)。 |
| `{controller=Home}/{action=Index}/{id?}` | `/`                     | `Home` コントローラーと `Index` メソッドにマッピングされます (`id` は無視されます)。        |

一般的に、テンプレートの利用が最も簡単なルーティングの手法となります。 ルート テンプレート以外では、制約と既定値も指定できます。

> [!TIP]
> [ログ](xref:fundamentals/logging/index)を有効にすると、<xref:Microsoft.AspNetCore.Routing.Route> など、組み込みのルーティング実装で要求を照合するしくみを確認できます。

## <a name="reserved-routing-names"></a>ルーティングの予約名

次のキーワードは予約名であり、ルート名やパラメーターとして使用できません。

* `action`
* `area`
* `controller`
* `handler`
* `page`

## <a name="route-constraint-reference"></a>ルート制約参照

ルート制約は、受信 URL と一致し、URL パスがルート値にトークン化されたときに実行されます。 ルート制約では、通常、ルート テンプレート経由で関連付けられるルート値を調べ、値が許容できるかどうかをはい/いいえで決定します。 一部のルート制約では、ルート値以外のデータを使用し、要求をルーティングできるかどうかが考慮されます。 たとえば、<xref:Microsoft.AspNetCore.Routing.Constraints.HttpMethodRouteConstraint> はその HTTP Verb に基づいて要求を承認または却下します。 制約は、要求のルーティングとリンクの生成で使用されます。

> [!WARNING]
> **入力の検証** には制約を使用しないでください。 **入力の検証** に制約が使用されると、無効な入力の結果、*400 - Bad Request* と適切なエラー メッセージではなく、*404 - Not Found* が表示されます。 ルート制約は、特定のルートに対する入力の妥当性を検証するためではなく、似たようなルートの **違いを明らかにする** ために使用されます。

次の表は、ルート制約の例とそれに求められる動作をまとめたものです。

| 制約 | 例 | 一致の例 | メモ |
|------------|---------|-----------------|-------|
| `int` | `{id:int}` | `123456789`, `-123456789` | 任意の整数と一致します。|
| `bool` | `{active:bool}` | `true`, `FALSE` | `true` または `false` に一致する。 大文字と小文字は区別されません|
| `datetime` | `{dob:datetime}` | `2016-12-31`, `2016-12-31 7:32pm` | インバリアント カルチャの有効な `DateTime` 値に一致します。 前の警告を参照してください。|
| `decimal` | `{price:decimal}` | `49.99`, `-1,000.01` | インバリアント カルチャの有効な `decimal` 値に一致します。 前の警告を参照してください。|
| `double` | `{weight:double}` | `1.234`, `-1,001.01e8` | インバリアント カルチャの有効な `double` 値に一致します。 前の警告を参照してください。|
| `float` | `{weight:float}` | `1.234`, `-1,001.01e8` | インバリアント カルチャの有効な `float` 値に一致します。 前の警告を参照してください。|
| `guid` | `{id:guid}` | `CD2C1638-1638-72D5-1638-DEADBEEF1638`, `{CD2C1638-1638-72D5-1638-DEADBEEF1638}` | 有効な `Guid` 値に一致します。|
| `long` | `{ticks:long}` | `123456789`, `-123456789` | 有効な `long` 値に一致します。|
| `minlength(value)` | `{username:minlength(4)}` | `Rick` | 4 文字以上の文字列であることが必要です。|
| `maxlength(value)` | `{filename:maxlength(8)}` | `MyFile` | 文字列の長さは最大 8 文字です。|
| `length(length)` | `{filename:length(12)}` | `somefile.txt` | 厳密に 12 文字の文字列であることが必要です。|
| `length(min,max)` | `{filename:length(8,16)}` | `somefile.txt` | 文字列の長さは 8 文字以上、最大で 16 文字とする必要があります。|
| `min(value)` | `{age:min(18)}` | `19` | 18 以上の整数値であることが必要です。|
| `max(value)` | `{age:max(120)}` | `91` | 最大値が 120 の整数値。|
| `range(min,max)` | `{age:range(18,120)}` | `91` | 整数値は 18 以上、最大で 120 とする必要があります。|
| `alpha` | `{name:alpha}` | `Rick` | 文字列は、1 つまたは複数のアルファベット文字 (`a`-`z`) で構成する必要があります。 大文字と小文字は区別されません|
| `regex(expression)` | `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}` | `123-45-6789` | 文字列は正規表現と一致する必要があります。 正規表現の定義に関するヒントを参照してください。|
| `required` | `{name:required}` | `Rick` | URL 生成中、非パラメーターが提示されるように強制するために使用されます。|

コロンで区切られた複数の制約を単一のパラメーターに適用することができます。 たとえば、次の制約では、パラメーターが 1 以上の整数値に制限されます。

```csharp
[Route("users/{id:int:min(1)}")]
public User GetUserById(int id) { }
```

> [!WARNING]
> URL の妥当性を検証し、CLR 型 (`int` や `DateTime` など) に変換されるルート制約では、常にインバリアント カルチャが使用されます。 これらの制約では、URL がローカライズ不可であることが前提となります。 フレームワークから提供されるルート制約がルート値に格納されている値を変更することはありません。 URL から解析されたルート値はすべて文字列として格納されます。 たとえば、`float` 制約はルート値を浮動小数に変換しますが、変換された値は、浮動小数に変換できることを検証するためにだけ利用されます。

## <a name="regular-expressions"></a>正規表現

ASP.NET Core フレームワークでは、正規表現コンストラクターに `RegexOptions.IgnoreCase | RegexOptions.Compiled | RegexOptions.CultureInvariant` が追加されます。 これらのメンバーの詳細については、「<xref:System.Text.RegularExpressions.RegexOptions>」を参照してください。

正規表現では、ルーティングや C# 言語で使用されるものに似た区切り記号とトークンが使用されます。 正規表現トークンはエスケープする必要があります。 ルーティングで正規表現 `^\d{3}-\d{2}-\d{4}$` を使用するには、次のようにします。

* 表現の場合、文字列内に指定された単一の円記号 `\` 文字を、ソース コード内で二重円記号 `\\` 文字とする必要があります。
* 正規表現では、文字列エスケープ文字 `\` をエスケープするためには `\\` を使用する必要があります。
* 正規表現では、[逐語的文字列リテラル](/dotnet/csharp/language-reference/keywords/string)を使用する場合、`\\` を必要としません。

ルーティング パラメーター区切り記号文字である `{`、`}`、`[`、`]` をエスケープするには、`{{`、`}`、`[[`、`]]` のように表現の文字を二重にします。 次の表に、正規表現とエスケープ適用後のものを示します。

| 正規表現    | エスケープ適用後の正規表現     |
| --------------------- | ------------------------------ |
| `^\d{3}-\d{2}-\d{4}$` | `^\\d{{3}}-\\d{{2}}-\\d{{4}}$` |
| `^[a-z]{2}$`          | `^[[a-z]]{{2}}$`               |

ルーティングで使用される正規表現は、多くの場合、キャレット文字 `^` で始まり、文字列の開始位置と一致します。 表現は、多くの場合、ドル記号 `$` で終わり、文字列の末尾と一致します。 `^` 文字と `$` 文字により、正規表現はルート パラメーター値全体に一致します。 `^` 文字と `$` 文字がなければ、正規表現は、文字列内のあらゆる部分文字列に一致します。それは多くの場合、意図に反することです。 下の表では例を示し、それらが一致する、または一致しない理由について説明します。

| 式   | String    | 一致したもの | コメント               |
| ------------ | --------- | :---: |  -------------------- |
| `[a-z]{2}`   | hello     | はい   | サブ文字列の一致     |
| `[a-z]{2}`   | 123abc456 | はい   | サブ文字列の一致     |
| `[a-z]{2}`   | mz        | はい   | 一致する表現    |
| `[a-z]{2}`   | MZ        | はい   | 大文字と小文字の使い方が違う    |
| `^[a-z]{2}$` | hello     | いいえ    | 上の `^` と `$` を参照 |
| `^[a-z]{2}$` | 123abc456 | いいえ    | 上の `^` と `$` を参照 |

正規表現構文の詳細については、[.NET Framework 正規表現](/dotnet/standard/base-types/regular-expression-language-quick-reference)に関するページを参照してください。

既知の入力可能値の集まりにパラメーターを制限するには、正規表現を使用します。 たとえば、`{action:regex(^(list|get|create)$)}` の場合、`action` ルート値は `list`、`get`、`create` とのみ照合されます。 制約ディクショナリに渡された場合、文字列 `^(list|get|create)$` で同じものになります。 (テンプレート内でインラインではなく) 制約ディクショナリに渡された制約が既知の制約に一致しない場合も、正規表現として扱われます。

## <a name="custom-route-constraints"></a>カスタム ルート制約

組み込みのルート制約だけでなく、<xref:Microsoft.AspNetCore.Routing.IRouteConstraint> インターフェイスを実装してカスタム ルート制約を作成することができます。 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> インターフェイスには、1 つのメソッド `Match` が含まれています。これは、制約が満たされている場合は `true` を、それ以外の場合は `false` を返します。

カスタムの <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> を使うには、アプリのサービス コンテナー内にあるアプリの <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> に、ルート制約の種類を登録する必要があります。 <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> は、ルート制約キーを、その制約を検証する <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> の実装にマッピングするディクショナリです。 アプリの <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> は、`Startup.ConfigureServices` で、[services.AddRouting](xref:Microsoft.Extensions.DependencyInjection.RoutingServiceCollectionExtensions.AddRouting*) 呼び出しの一部として、または `services.Configure<RouteOptions>` を使って <xref:Microsoft.AspNetCore.Routing.RouteOptions> を直接構成することで、更新できます。 次に例を示します。

```csharp
services.AddRouting(options =>
{
    options.ConstraintMap.Add("customName", typeof(MyCustomConstraint));
});
```

これで、制約の種類を登録するときに指定した名前を使って、通常の方法でルートに制約を適用できます。 次に例を示します。

```csharp
[HttpGet("{id:customName}")]
public ActionResult<string> Get(string id)
```

## <a name="parameter-transformer-reference"></a>パラメーター トランスフォーマー参照

パラメーター トランスフォーマー:

* <xref:Microsoft.AspNetCore.Routing.Route> のリンクの生成時に実行されます。
* `Microsoft.AspNetCore.Routing.IOutboundParameterTransformer`を実装します。
* <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> を使用して構成されます。
* パラメーターのルート値を取得し、それを新しい文字列値に変換します。
* 変換された値は生成されたリンクで使用されるようになります。

たとえば、`Url.Action(new { article = "MyTestArticle" })` のルート パターン `blog\{article:slugify}` のカスタム `slugify` パラメーター トランスフォーマーでは、`blog\my-test-article` が生成されます。

ルート パターンでパラメーター トランスフォーマーを使用するには、まず `Startup.ConfigureServices` で <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> を使用してこれを構成します。

```csharp
services.AddRouting(options =>
{
    // Replace the type and the name used to refer to it with your own
    // IOutboundParameterTransformer implementation
    options.ConstraintMap["slugify"] = typeof(SlugifyParameterTransformer);
});
```

パラメーター トランスフォーマーは、エンドポイントが解決される URI を変換するためにフレームワークで使用されます。 たとえば、ASP.NET Core MVC ではパラメーター トランスフォーマーを使用して、`area`、`controller`、`action`、`page` を照合するために使用されるルート値を変換します。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller:slugify=Home}/{action:slugify=Index}/{id?}");
```

上記のルートでは、アクション `SubscriptionManagementController.GetAll` は URI `/subscription-management/get-all` と一致します。 パラメーター トランスフォーマーでは、リンクを生成するために使用されるルート値は変更されません。 たとえば、`Url.Action("GetAll", "SubscriptionManagement")` では `/subscription-management/get-all` が出力されます。

ASP.NET Core では、生成されたルートと共にパラメーター トランスフォーマーを使用するための API 規則が提供されます。

* ASP.NET Core MVC には、`Microsoft.AspNetCore.Mvc.ApplicationModels.RouteTokenTransformerConvention` API 規則が備わっています。 この規則では、指定されたパラメーター トランスフォーマーがアプリ内のすべての属性ルートに適用されます。 パラメーター トランスフォーマーでは、置き換えられる属性ルート トークンが変換されます。 詳細については、「[パラメーター トランスフォーマーを使用してトークンの置換をカスタマイズする](xref:mvc/controllers/routing#use-a-parameter-transformer-to-customize-token-replacement)」をご覧ください。
* Razor Pages には、`Microsoft.AspNetCore.Mvc.ApplicationModels.PageRouteTransformerConvention` API 規則があります。 この規則では、指定されたパラメーター トランスフォーマーが、自動で検出されたすべての Razor Pages に適用されます。 パラメーター トランスフォーマーでは、Razor Pages ルートのフォルダーとファイル名のセグメントが変換されます。 詳細については、[パラメーター トランスフォーマーを使用したページ ルートのカスタマイズ](xref:razor-pages/razor-pages-conventions#use-a-parameter-transformer-to-customize-page-routes)に関する記事をご覧ください。

## <a name="url-generation-reference"></a>URL 生成参照

次の例では、ルートのリンクを生成する方法を確認できます。ルート値のディクショナリと <xref:Microsoft.AspNetCore.Routing.RouteCollection> が指定されています。

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_Dictionary)]

上のサンプルの終わりで生成された <xref:Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath> は `/package/create/123` です。 ディクショナリにより、"Track Package Route" テンプレート `package/{operation}/{id}` のルート値 `operation` と `id` が提供されます。 詳細については、「[ルーティング ミドルウェアの使用](#use-routing-middleware)」セクションのサンプル コードまたは[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/routing/samples)を参照してください。

<xref:Microsoft.AspNetCore.Routing.VirtualPathContext> コンストラクターの 2 番目のパラメーターは *アンビエント値* の集合です。 アンビエント値は、開発者が要求コンテキスト内で指定する必要がある値の数が制限されるため、使用すると便利です。 現在の要求の現在のルート値は、リンク生成の場合、アンビエント値として見なされます。 ASP.NET Core MVC アプリの `HomeController` の `About` アクションでは、コントローラー ルート値を指定し、`Index` アクションにリンクする必要はありません。`Home` のアンビエント値が使用されます。

パラメーターに一致しないアンビエント値は無視されます。 アンビエント値は、明示的に指定された値でアンビエント値がオーバーライドされる場合にも無視されます。 URL では左から右に照合されます。

明示的に指定されているが、ルートのセグメントと一致しない値は、クエリ文字列に追加されます。 次の表は、ルート テンプレート `{controller}/{action}/{id?}` の使用時の結果をまとめたものです。

| アンビエント値                     | 明示的な値                        | 結果                  |
| ---------------------------------- | -------------------------------------- | ----------------------- |
| controller = "Home"                | action = "About"                       | `/Home/About`           |
| controller = "Home"                | controller = "Order", action = "About" | `/Order/About`          |
| controller = "Home", color = "Red" | action = "About"                       | `/Home/About`           |
| controller = "Home"                | action = "About", color = "Red"        | `/Home/About?color=Red` |

パラメーターに対応しない既定値がルートにあり、その値が明示的に指定される場合、それは既定値に一致する必要があります。

```csharp
routes.MapRoute("blog_route", "blog/{*slug}",
    defaults: new { controller = "Blog", action = "ReadPost" });
```

リンク生成では、`controller` と `action` について一致する値が指定されるときにのみ、このルートのリンクが生成されます。

## <a name="complex-segments"></a>複雑なセグメント

複雑なセグメント (例: `[Route("/x{token}y")]`) は、リテラルを右から左に最短一致の方法で照合することによって処理されます。 複雑なセグメントの一致方法に関する詳しい説明については、[こちらのコード](https://github.com/dotnet/aspnetcore/blob/v2.2.13/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293)をご覧ください。 この[コード サンプル](https://github.com/dotnet/aspnetcore/blob/v2.2.13/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293)は ASP.NET Core では使われていませんが、複雑なセグメントに関する優れた説明が提供されています。
<!-- While that code is no longer used by ASP.NET Core for complex segment matching, it provides a good match to the current algorithm. The [current code](https://github.com/dotnet/AspNetCore/blob/91514c9af7e0f4c44029b51f05a01c6fe4c96e4c/src/Http/Routing/src/Matching/DfaMatcherBuilder.cs#L227-L244) is too abstracted from matching to be useful for understanding complex segment matching.
-->

::: moniker-end

::: moniker range="< aspnetcore-2.2"

ルーティングでは要求 URI をルート ハンドラーにマッピングし、受信要求を配布します。 ルートはアプリに定義され、アプリの起動時に構成されます。 ルートは、要求に含まれている URL から値を任意で抽出できます。その値を要求処理に利用できます。 ルーティングでは、アプリからの構成済みルートを使用して、ルート ハンドラーにマッピングする URL を生成できます。

ASP.NET Core 2.1 で最新のルーティング シナリオを使用するには、`Startup.ConfigureServices` で[互換性バージョン](xref:mvc/compatibility-version)を MVC サービス登録に指定します。

```csharp
services.AddMvc()
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
```

> [!IMPORTANT]
> 本文では、ASP.NET Core ルーティングについて詳しく取り上げます。 ASP.NET Core MVC ルーティングの詳細については、「<xref:mvc/controllers/routing>」を参照してください。 Razor Pages のルーティング規則については、「<xref:razor-pages/razor-pages-conventions>」を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/routing/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="routing-basics"></a>ルーティングの基本

ほとんどのアプリでは、URL を読みやすくてわかりやすいものにするために、基本的なでわかりやすいルーティング スキームを選択する必要があります。 既定の規則ルート `{controller=Home}/{action=Index}/{id?}`:

* 基本的でわかりやすいルーティング スキームをサポートしています。
* UI ベースのアプリの便利な開始点となります。

開発者は一般的に、特殊な状況でのアプリのトラフィックが多い領域 (ブログや E コマース エンドポイントなど) に対しては、[属性ルーティング](xref:mvc/controllers/routing#attribute-routing)または専用規則ルートを使用して簡易ルートをさらに追加します。

Web API では、属性ルーティングを使用して、HTTP 動詞で操作を表現するリソースのセットとしてアプリの機能をモデル化する必要があります。 つまり、同じ論理リソース上の多くの操作 (たとえば GET や POST) で、同じ URL を使用します。 属性ルーティングでは、API のパブリック エンドポイント レイアウトを慎重に設計するために必要となるコントロールのレベルが提供されます。

Razor Pages アプリでは、既定の規則ルーティングを使用して、アプリの *Pages* フォルダーの名前付きリソースを提供します。 Razor Pages のルーティング動作をカスタマイズできる追加の規則を使用できます。 詳細については、次のトピックを参照してください。 <xref:razor-pages/index> および <xref:razor-pages/razor-pages-conventions>

URL 生成サポートを使用すると、アプリを相互にリンクする URL をハード コーディングすることなくアプリを開発できます。 このサポートにより、基本的なルーティング構成を使用して作業を開始し、アプリのリソース レイアウトが決まった後でルートを変更することができます。

ルーティングでは <xref:Microsoft.AspNetCore.Routing.IRouter> のルートの実装が次の目的で利用されます。

* 受信要求を *ルート ハンドラー* にマッピングする。
* 応答で使用される URL を生成する。

既定では、アプリにはルート コレクションが 1 つあります。 要求が到着すると、コレクション内のルートは、コレクションに存在する順序で処理されます。 フレームワークでは、コレクション内の各ルートに対して <xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> メソッドを呼び出し、受信要求 URL とコレクション内のルートとの照合を試みます。 応答ではルーティングを使用し、ルート情報に基づいて URL (リダイレクトやリンクなど) を生成できます。そのため、URL をハードコーディングする必要がなく、保守管理が容易になります。

ルーティング システムには次の特性があります。

* ルート テンプレート構文は、トークン化されたルート パラメーターでルートを定義するために使用されます。
* 規則スタイルおよび属性スタイルのエンドポイント構成が許可されます。
* <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> は、URL パラメーターに特定のエンドポイント制約で有効な値が含まれているかどうかを確認するために使用されます。
* MVC/Razor Pages などのアプリ モデルでは、ルーティング シナリオの予測可能な実装を持つ、すべてのルートを登録します。
* 応答ではルーティングを使用し、ルート情報に基づいて URL (リダイレクトやリンクなど) を生成できます。そのため、URL をハードコーディングする必要がなく、保守管理が容易になります。
* URL の生成は、任意の拡張機能をサポートする、ルートに基づきます。 <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> では、URL を構築するためのメソッドが提供されます。
<!-- fix [middleware](xref:fundamentals/middleware/index) -->
ルーティングは <xref:Microsoft.AspNetCore.Builder.RouterMiddleware> クラスにより[ミドルウェア](xref:fundamentals/middleware/index) パイプラインに接続されます。 [ASP.NET Core MVC](xref:mvc/overview) では、ミドルウェア パイプラインにその構成の一部としてルーティングが追加され、MVC および Razor Pages アプリでのルーティングが処理されます。 スタンドアロン コンポーネントとしてルーティングを利用する方法については、「[ルーティング ミドルウェアの使用](#use-routing-middleware)」セクションを参照してください。

### <a name="url-matching"></a>URL 一致

URL 一致というプロセスでは、ルーティングによって、受信要求が *ハンドラー* に配布されます。 このプロセスは URL パスのデータに基づきますが、要求内のあらゆるデータを考慮することもできます。 アプリの規模を大きくしたり、より複雑にしたりするとき、要求を個別のハンドラーに配布する機能が鍵となります。

受信要求は <xref:Microsoft.AspNetCore.Builder.RouterMiddleware> に入ります。これが各ルートで順に <xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> メソッドを呼び出します。 <xref:Microsoft.AspNetCore.Routing.IRouter> インスタンスでは、[RouteContext.Handler](xref:Microsoft.AspNetCore.Routing.RouteContext.Handler*) を非 null の <xref:Microsoft.AspNetCore.Http.RequestDelegate> に設定することで、要求を *処理する* かどうかを選択します。 要求に適したハンドラーがルートに見つかると、ルート処理が停止します。ハンドラーが呼び出され、要求を処理します。 要求を処理するためのルート ハンドラーが見つからない場合、ミドルウェアによって、要求が要求パイプラインの次のミドルウェアに渡されます。

<xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> に最初に入力されるのが、現在の要求に関連付けられている [RouteContext.HttpContext](xref:Microsoft.AspNetCore.Routing.RouteContext.HttpContext*) です。 [RouteContext.Handler](xref:Microsoft.AspNetCore.Routing.RouteContext.Handler) と [RouteContext.RouteData](xref:Microsoft.AspNetCore.Routing.RouteContext.RouteData*) は、ルートが一致した後に設定される出力です。

また、<xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> を呼び出す一致が見つかると、それまでに実行された要求処理に基づいて、[RouteContext.RouteData](xref:Microsoft.AspNetCore.Routing.RouteContext.RouteData) のプロパティが適切な値に設定されます。

[RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values*) は、ルートから生成された *ルート値* のディクショナリです。 この値は通常、URL のトークン化で決定されます。この値を利用してユーザー入力を受け取ったり、アプリ内で決定をさらに配布したりできます。

[RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) は、一致したルートに関連する追加データのプロパティ バッグです。 一致したルートに基づいてアプリで決定を行えるように、<xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*> が提供されます。これは状態データと各ルートの関連付けをサポートするためのものです。 この値は開発者が定義するものです。ルーティングの動作に影響を与えることは **ありません**。 また、[RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) に一時退避される値はどのような型でも構いません。一方、[RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values) の場合は、文字列間で変換できる必要があります。

[RouteData.Routers](xref:Microsoft.AspNetCore.Routing.RouteData.Routers) は、過去に要求に一致したルートの一覧です。 ルートはルートの中に入れ子にすることができます。 <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> プロパティは、結果的に一致をもたらしたルートの論理ツリーを通るパスを表します。 一般的に、<xref:Microsoft.AspNetCore.Routing.RouteData.Routers> の最初の項目はルート コレクションであり、これは URL 生成に使用するものです。 <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> の最後の項目は、一致したルート ハンドラーです。

<a name="lg"></a>

### <a name="url-generation"></a>URL 生成

URL 生成は、ルーティングにおいて、一連のルート値に基づいて URL パスを作成するプロセスです。 これにより、ルート ハンドラーとそれにアクセスする URL を論理的に分離できます。

URL 生成は同様の繰り返しプロセスに従いますが、最初にユーザーまたはフレームワーク コードでルート コレクションの <xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> メソッドを呼び出します。 非 null の <xref:Microsoft.AspNetCore.Routing.VirtualPathData> が返されるまで、各 *ルート* で順番に <xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> メソッドが呼び出されます。

<xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> の第一入力:

* [VirtualPathContext.HttpContext](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.HttpContext)
* [VirtualPathContext.Values](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.Values)
* [VirtualPathContext.AmbientValues](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues)

ルートでは <xref:Microsoft.AspNetCore.Routing.VirtualPathContext.Values> と <xref:Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues> によって指定されるルート値を主に使用し、URL を生成できるかどうかと、どの値を含めるかを判断します。 <xref:Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues> は、現在の要求を照合することで生成された一連のルート値です。 対照的に、<xref:Microsoft.AspNetCore.Routing.VirtualPathContext.Values> は、現在の操作に適した URL の生成方法を指定するルート値です。 ルートで現在のコンテキストに関連付けられているサービスまたは追加データを取得する必要がある場合、<xref:Microsoft.AspNetCore.Routing.VirtualPathContext.HttpContext> が指定されます。

> [!TIP]
> [VirtualPathContext.Values](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.Values*) は [VirtualPathContext.AmbientValues](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues*) のオーバーライド セットであると考えてください。 URL 生成では、同じルートまたはルート値を利用することでリンクの URL 生成を生成するために、現在の要求からのルート値の再利用が試行されます。

<xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> の出力は <xref:Microsoft.AspNetCore.Routing.VirtualPathData> です。 <xref:Microsoft.AspNetCore.Routing.VirtualPathData> は <xref:Microsoft.AspNetCore.Routing.RouteData> の並列です。 <xref:Microsoft.AspNetCore.Routing.VirtualPathData> には出力 URL として <xref:Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath> が含まれ、ルートで設定する追加プロパティがいくつか含まれます。

[VirtualPathData.VirtualPath](xref:Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath*) プロパティには、ルートによって生成された *仮想パス* が含まれます。 必要に応じて、パスをさらに処理する必要がある場合があります。 生成された URL を HTML で表示するには、アプリの基礎パスを先頭に追加する必要があります。

[VirtualPathData.Router](xref:Microsoft.AspNetCore.Routing.VirtualPathData.Router*) は URL を正常に生成したルートの参照です。

[VirtualPathData.DataTokens](xref:Microsoft.AspNetCore.Routing.VirtualPathData.DataTokens*) プロパティは、URL を生成したルートに関連する追加データのディクショナリです。 これは [RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) の並列です。

### <a name="create-routes"></a>ルートを作成する

ルーティングにより、<xref:Microsoft.AspNetCore.Routing.IRouter> の標準実装として <xref:Microsoft.AspNetCore.Routing.Route> クラスが与えられます。 <xref:Microsoft.AspNetCore.Routing.Route> では *ルート テンプレート* 構文を利用して、<xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> の呼び出し時に URL パスに対して照合するパターンを定義します。 <xref:Microsoft.AspNetCore.Routing.Route> は同じルート テンプレートを利用し、<xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> の呼び出し時に URL を生成します。

ほとんどのアプリは、<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> を呼び出すか、<xref:Microsoft.AspNetCore.Routing.IRouteBuilder> で定義されている同様の拡張メソッドの 1 つを呼び出してルートを作成します。 いずれの <xref:Microsoft.AspNetCore.Routing.IRouteBuilder> 拡張メソッドでも、<xref:Microsoft.AspNetCore.Routing.Route> のインスタンスが作成され、それがルート コレクションに追加されます。

<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> では、ルート ハンドラー パラメーターは受け入れられません。 <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> は、<xref:Microsoft.AspNetCore.Routing.RouteBuilder.DefaultHandler*> によって処理されるルートを追加するだけです。 既定のハンドラーは `IRouter` であり、そのハンドラーで要求が処理されない可能性があります。 たとえば、ASP.NET Core MVC は通常、利用できるコントローラーやアクションに一致する要求のみを処理する既定のハンドラーとして構成されます。 MVC でのルーティングの詳細については、「<xref:mvc/controllers/routing>」を参照してください。

次のコード例は、一般的な ASP.NET Core MVC ルート定義によって使用される <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 呼び出しの例です。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

このテンプレートでは URL パスが照合され、ルート値が抽出されます。 たとえば、`/Products/Details/17` というパスでは、`{ controller = Products, action = Details, id = 17 }` というルート値が生成されます。

ルート値は、URL パスをセグメントに分割し、各セグメントと、ルート テンプレートの *ルート パラメーター* 名を照合することで決定されます。 ルート パラメーターが指定されます。 パラメーターは、中かっこ `{ ... }` でパラメーター名を囲むことで定義されます。

上のテンプレートでは URL パス `/` の照合も行い、`{ controller = Home, action = Index }` という値を生成します。 これは、ルート パラメーターの `{controller}` と `{action}` に既定値が与えられ、`id` ルート パラメーターが任意であるためです。 ルート パラメーター名の後の等号 (`=`) とそれに続く値により、パラメーターの既定値が定義されます。 ルート パラメーター名の後の疑問符 (`?`) により、オプションのパラメーターが定義されます。

既定値のルート パラメーターはルートが一致するとルート値を *常に* 生成します。 省略可能なパラメーターでは、対応する URL パス セグメントがない場合、ルート値は生成されません。 ルート テンプレートのシナリオと構文の詳しい説明については、「[ルート テンプレート参照](#route-template-reference)」セクションを参照してください。

次の例では、ルート パラメーター定義 `{id:int}` により、`id` ルート パラメーターの[ルート制約](#route-constraint-reference)が定義されます。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id:int}");
```

このテンプレートは `/Products/Details/Apples` ではなく、`/Products/Details/17` のような URL パスを照合します。 ルート制約は <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> を実装し、ルート値を調べて正しいことを確認します。 この例では、ルート値 `id` は整数に変換できなければなりません。 フレームワークによって提供されるルート制約については、「[ルート制約参照](#route-constraint-reference)」を参照してください。

<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> の追加オーバーロードは、`constraints`、`dataTokens`、`defaults` の値を受け取ります。 このようなパラメーターは一般的に、匿名型のオブジェクトを渡し、匿名型のプロパティ名がルート パラメーター名と一致するというような使われ方をします。

次の <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 例では、同等のルートを作成します。

```csharp
routes.MapRoute(
    name: "default_route",
    template: "{controller}/{action}/{id?}",
    defaults: new { controller = "Home", action = "Index" });

routes.MapRoute(
    name: "default_route",
    template: "{controller=Home}/{action=Index}/{id?}");
```

> [!TIP]
> 制約と既定値を定義するインライン構文は単純なルートの場合に便利です。 しかし、インライン構文でサポートされていない、データ トークンなどのシナリオがあります。

次の例では、その他のいくつかのシナリオを示します。

```csharp
routes.MapRoute(
    name: "blog",
    template: "Blog/{*article}",
    defaults: new { controller = "Blog", action = "ReadArticle" });
```

上記のテンプレートでは `/Blog/All-About-Routing/Introduction` のような URL パスを照合し、値 `{ controller = Blog, action = ReadArticle, article = All-About-Routing/Introduction }` を抽出します。 `controller` と `action` の既定のルート値は、テンプレートに対応するルート パラメーターがなくても、ルートにより生成されます。 既定値はルート テンプレートで指定できます。 `article` ルート パラメーターは、ルート パラメーター名の前にアスタリスク (`*`) があるときに、*キャッチオール* として定義されます。 キャッチオール ルート パラメーターは、URL パスの残りの部分をキャプチャします。空の文字列も照合できます。

次の例では、ルート制約とデータ トークンを追加します。

```csharp
routes.MapRoute(
    name: "us_english_products",
    template: "en-US/Products/{id}",
    defaults: new { controller = "Products", action = "Details" },
    constraints: new { id = new IntRouteConstraint() },
    dataTokens: new { locale = "en-US" });
```

上記のテンプレートでは `/en-US/Products/5` のような URL パスを照合し、値 `{ controller = Products, action = Details, id = 5 }` とデータ トークン `{ locale = en-US }` を抽出します。

![ローカル変数ウィンドウ トークン](routing/_static/tokens.png)

### <a name="route-class-url-generation"></a>ルート クラス URL の生成

<xref:Microsoft.AspNetCore.Routing.Route> クラスは、一連のルート値をそのルート テンプレートと組み合わせる方法でも URL を生成できます。 論理的には、URL パスの照合とは逆のプロセスになります。

> [!TIP]
> URL 生成を理解する方法として、どのような URL を生成したいのかを考えてください。そして、ルート テンプレートがその URL にどのように照合されるのかを考えてください。 どのような値が生成されるでしょうか。 <xref:Microsoft.AspNetCore.Routing.Route> クラスにおける URL 生成のしくみと大体同じになります。

次の例では、一般的な ASP.NET Core MVC の既定のルートを使用します。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

ルート値が `{ controller = Products, action = List }` の場合、URL `/Products/List` が生成されます。 ルート値が対応ルート パラメーターの代替となり、URL パスが作られます。 `id` は任意のルート パラメーターであるため、URL は `id` の値なしで正常に生成されます。

ルート値が `{ controller = Home, action = Index }` の場合、URL `/` が生成されます。 指定されたルート値は既定値と一致し、その既定値に対応するセグメントを省略しても問題は発生しません。

生成された両方の URL では、次のルート定義 (`/Home/Index` と `/`) でラウンドトリップされ、URL の生成に使用された同じルート値が生成されます。

> [!NOTE]
> アプリで ASP.NET Core MVC が使用される場合、ルーティングを直接呼び出す代わりに、<xref:Microsoft.AspNetCore.Mvc.Routing.UrlHelper> を使用して URL を生成する必要があります。

URL 生成の詳細については、「[URL 生成参照](#url-generation-reference)」セクションを参照してください。

## <a name="use-routing-middleware"></a>ルーティング ミドルウェアの使用

アプリのプロジェクト ファイルの [Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)を参照します。

`Startup.ConfigureServices` でサービス コンテナーにルーティングを追加した例:

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_ConfigureServices&highlight=3)]

ルートは `Startup.Configure` メソッドで設定する必要があります。 サンプル アプリでは次の API を使用します。

* <xref:Microsoft.AspNetCore.Routing.RouteBuilder>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*>:HTTP GET 要求のみを照合します。
* <xref:Microsoft.AspNetCore.Builder.RoutingBuilderExtensions.UseRouter*>

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_RouteHandler)]

下の表は、応答と所与の URI をまとめたものです。

| URI                    | 応答                                          |
| ---------------------- | ------------------------------------------------- |
| `/package/create/3`    | Hello! Route values: [operation, create], [id, 3] |
| `/package/track/-3`    | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/-3/`   | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/`      | 要求はフォール スルーし、一致するものはありません。              |
| `GET /hello/Joe`       | Hi, Joe!                                          |
| `POST /hello/Joe`      | 要求はフォール スルーし、HTTP GET のみが一致します。 |
| `GET /hello/Joe/Smith` | 要求はフォール スルーし、一致するものはありません。              |

ルートを 1 つ構成する場合、<xref:Microsoft.AspNetCore.Builder.RoutingBuilderExtensions.UseRouter*> を呼び出し、`IRouter` インスタンスを渡します。 <xref:Microsoft.AspNetCore.Routing.RouteBuilder> を使用する必要はありません。

このフレームワークでは、ルートを作成するために拡張メソッド セット (<xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions>) が提供されます。

* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareVerb*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>

<xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*> など、リストされているメソッドの一部では、<xref:Microsoft.AspNetCore.Http.RequestDelegate> が必要です。 <xref:Microsoft.AspNetCore.Http.RequestDelegate> は、ルートが一致したとき、*ルート ハンドラー* として使用されます。 この仲間の他のメソッドでは、ルート ハンドラーとして使用されるミドルウェア パイプラインを構成できます。 `Map*` メソッドで、<xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapRoute*> などのハンドラーを受け入れない場合、<xref:Microsoft.AspNetCore.Routing.RouteBuilder.DefaultHandler*> が使用されます。

`Map[Verb]` メソッドは制約を利用し、メソッド名の HTTP Verb にルートを制限します。 たとえば、 <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*> や <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>を参照してください。

## <a name="route-template-reference"></a>ルート テンプレート参照

中括弧 (`{ ... }`) 内のトークンは *ルート パラメーター* を定義します。ルートが一致した場合、これがバインドされます。 1 つのルート セグメントで複数のルート パラメーターを定義できますが、リテラル値で区切る必要があります。 たとえば、`{controller=Home}{action=Index}` は有効なルートになりません。`{controller}` と `{action}` の間にリテラル値がないためです。 このようなルート パラメーターには名前を付ける必要があります。付加的な属性を指定することもあります。

ルート パラメーター以外のリテラル テキスト (`{id}` など) とパス区切り `/` は URL のテキストに一致する必要があります。 テキスト照合は復号された URL パスを基盤とし、大文字と小文字が区別されます。 リテラル ルート パラメーターの区切り記号 (`{` または `}`) を照合するには、文字を繰り返して区切り記号をエスケープします (`{{` または `}}`)。

任意のファイル拡張子が付いたファイル名のキャプチャを試行する URL パターンには、追加の考慮事項があります。 たとえば、テンプレート `files/{filename}.{ext?}` について考えてみます。 `filename` と `ext` の両方の値が存在するときに、両方の値が入力されます。 URL に `filename` の値だけが存在する場合、末尾のピリオド (`.`) は任意であるため、このルートは一致となります。 次の URL はこのルートに一致します。

* `/files/myFile.txt`
* `/files/myFile`

ルート パラメーターのプレフィックスとしてアスタリスク (`*`) を使用し、URI の残りの部分にバインドできます。 これは *キャッチオール* パラメーターと呼ばれています。 たとえば、`blog/{*slug}` は、`/blog` で始まり、その後に (`slug` ルート値に割り当てられる) 任意の値が続くあらゆる URI に一致します。 キャッチオール パラメーターは空の文字列に一致することもあります。

キャッチオール パラメーターでは、パス区切り (`/`) 文字を含め、URL の生成にルートが使用されるときに適切な文字がエスケープされます。 たとえば、ルート値が `{ path = "my/path" }` のルート `foo/{*path}` では、`foo/my%2Fpath` が生成されます。 エスケープされたスラッシュに注意してください。

ルート パラメーターには、*既定値* が含まれることがあります。パラメーター名の後に既定値を指定し、等号 (`=`) で区切ることで指定されます。 たとえば、`{controller=Home}` では、`controller` の既定値として `Home` が定義されます。 パラメーターの URL に値がない場合、既定値が使用されます。 ルート パラメーターは、`id?` のように、パラメーター名の終わりに疑問符 (`?`) を追加することでオプションとして扱われます。 オプション値と既定ルート パラメーターの違いは、既定値を含むルート パラメーターは常に値を生成するのに対し、オプションのパラメーターには、要求 URL によって値が指定されたときにのみ値が与えられることにあります。

ルート パラメーターには、URL からバインドされるルート値に一致しなければならないという制約が含まれることがあります。 コロン (`:`) と制約名をルート パラメーター名の後に追加すると、ルート パラメーターの *インライン制約* が指定されます。 その制約で引数が要求される場合、制約名の後にかっこ (`(...)`) で囲まれます。 別のコロン (`:`) と制約名を追加することで、複数のインライン制約を指定できます。

制約名と引数が <xref:Microsoft.AspNetCore.Routing.IInlineConstraintResolver> サービスに渡され、URL 処理で使用する <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> のインスタンスが作成されます。 たとえば、ルート テンプレート `blog/{article:minlength(10)}` によって、制約 `minlength` と引数 `10` が指定されます。 ルート制約の詳細とこのフレームワークによって指定される制約のリストについては、「[ルート制約参照](#route-constraint-reference)」セクションを参照してください。

次の表は、ルート テンプレートの例とその動作をまとめたものです。

| ルート テンプレート                           | 一致する URI の例    | 要求 URI&hellip;                                                    |
| ---------------------------------------- | ----------------------- | -------------------------------------------------------------------------- |
| `hello`                                  | `/hello`                | 単一パス `/hello` にのみ一致します。                                     |
| `{Page=Home}`                            | `/`                     | 一致し、`Page` が `Home` に設定されます。                                         |
| `{Page=Home}`                            | `/Contact`              | 一致し、`Page` が `Contact` に設定されます。                                      |
| `{controller}/{action}/{id?}`            | `/Products/List`        | `Products` コントローラーと `List` アクションにマッピングされます。                       |
| `{controller}/{action}/{id?}`            | `/Products/Details/123` | `Products` コントローラーと `Details` アクションにマッピングされます (`id` は 123 に設定されます)。 |
| `{controller=Home}/{action=Index}/{id?}` | `/`                     | `Home` コントローラーと `Index` メソッドにマッピングされます (`id` は無視されます)。        |

一般的に、テンプレートの利用が最も簡単なルーティングの手法となります。 ルート テンプレート以外では、制約と既定値も指定できます。

> [!TIP]
> [ログ](xref:fundamentals/logging/index)を有効にすると、<xref:Microsoft.AspNetCore.Routing.Route> など、組み込みのルーティング実装で要求を照合するしくみを確認できます。

## <a name="route-constraint-reference"></a>ルート制約参照

ルート制約は、受信 URL と一致し、URL パスがルート値にトークン化されたときに実行されます。 ルート制約では、通常、ルート テンプレート経由で関連付けられるルート値を調べ、値が許容できるかどうかをはい/いいえで決定します。 一部のルート制約では、ルート値以外のデータを使用し、要求をルーティングできるかどうかが考慮されます。 たとえば、<xref:Microsoft.AspNetCore.Routing.Constraints.HttpMethodRouteConstraint> はその HTTP Verb に基づいて要求を承認または却下します。 制約は、要求のルーティングとリンクの生成で使用されます。

> [!WARNING]
> **入力の検証** には制約を使用しないでください。 **入力の検証** に制約が使用されると、無効な入力の結果、*400 - Bad Request* と適切なエラー メッセージではなく、*404 - Not Found* が表示されます。 ルート制約は、特定のルートに対する入力の妥当性を検証するためではなく、似たようなルートの **違いを明らかにする** ために使用されます。

次の表は、ルート制約の例とそれに求められる動作をまとめたものです。

| 制約 | 例 | 一致の例 | メモ |
| ---------- | ------- | --------------- | ----- |
| `int` | `{id:int}` | `123456789`, `-123456789` | あらゆる整数に一致する |
| `bool` | `{active:bool}` | `true`, `FALSE` | `true` または `false` に一致する (大文字と小文字を区別しません) |
| `datetime` | `{dob:datetime}` | `2016-12-31`, `2016-12-31 7:32pm` | インバリアント カルチャの有効な `DateTime` 値に一致します。 前の警告を参照してください。|
| `decimal` | `{price:decimal}` | `49.99`, `-1,000.01` | インバリアント カルチャの有効な `decimal` 値に一致します。 前の警告を参照してください。|
| `double` | `{weight:double}` | `1.234`, `-1,001.01e8` | インバリアント カルチャの有効な `double` 値に一致します。 前の警告を参照してください。|
| `float` | `{weight:float}` | `1.234`, `-1,001.01e8` | インバリアント カルチャの有効な `float` 値に一致します。 前の警告を参照してください。|
| `guid` | `{id:guid}` | `CD2C1638-1638-72D5-1638-DEADBEEF1638`, `{CD2C1638-1638-72D5-1638-DEADBEEF1638}` | 有効な `Guid` 値に一致する |
| `long` | `{ticks:long}` | `123456789`, `-123456789` | 有効な `long` 値に一致する |
| `minlength(value)` | `{username:minlength(4)}` | `Rick` | 4 文字以上の文字列であることが必要 |
| `maxlength(value)` | `{filename:maxlength(8)}` | `Richard` | 8 文字以内の文字列であることが必要 |
| `length(length)` | `{filename:length(12)}` | `somefile.txt` | 厳密に 12 文字の文字列であることが必要 |
| `length(min,max)` | `{filename:length(8,16)}` | `somefile.txt` | 8 文字以上、16 文字以内の文字列であることが必要 |
| `min(value)` | `{age:min(18)}` | `19` | 18 以上の整数値であることが必要 |
| `max(value)` | `{age:max(120)}` | `91` | 120 以下の整数値であることが必要 |
| `range(min,max)` | `{age:range(18,120)}` | `91` | 18 以上、120 以下の整数値であることが必要 |
| `alpha` | `{name:alpha}` | `Rick` | 1 つまたは複数のアルファベット文字で構成される文字列が必要 (`a`-`z`、大文字と小文字を区別) |
| `regex(expression)` | `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}` | `123-45-6789` | 正規表現に一致する文字列が必要 (正規表現の定義についてはヒントを参照) |
| `required` | `{name:required}` | `Rick` | URL 生成中、非パラメーターが提示されるように強制する |

コロンで区切られた複数の制約を単一のパラメーターに適用することができます。 たとえば、次の制約では、パラメーターが 1 以上の整数値に制限されます。

```csharp
[Route("users/{id:int:min(1)}")]
public User GetUserById(int id) { }
```

> [!WARNING]
> URL の妥当性を検証し、CLR 型 (`int` や `DateTime` など) に変換されるルート制約では、常にインバリアント カルチャが使用されます。 これらの制約では、URL がローカライズ不可であることが前提となります。 フレームワークから提供されるルート制約がルート値に格納されている値を変更することはありません。 URL から解析されたルート値はすべて文字列として格納されます。 たとえば、`float` 制約はルート値を浮動小数に変換しますが、変換された値は、浮動小数に変換できることを検証するためにだけ利用されます。

## <a name="regular-expressions"></a>正規表現

ASP.NET Core フレームワークでは、正規表現コンストラクターに `RegexOptions.IgnoreCase | RegexOptions.Compiled | RegexOptions.CultureInvariant` が追加されます。 これらのメンバーの詳細については、「<xref:System.Text.RegularExpressions.RegexOptions>」を参照してください。

正規表現では、ルーティングや C# 言語で使用されるものに似た区切り記号とトークンが使用されます。 正規表現トークンはエスケープする必要があります。 ルーティングで正規表現 `^\d{3}-\d{2}-\d{4}$` を使用するには、C# ソース ファイルで `\\` (二重円記号) 文字として文字列に `\` (単一の円記号) 文字を指定し、文字列エスケープ文字である `\` をエスケープする必要があります ([逐語的な文字列リテラル](/dotnet/csharp/language-reference/keywords/string)を使用しない限り)。 ルーティング パラメーター区切り記号文字 (`{`、`}`、`[`、`]`) をエスケープするには、表現の文字を二重にします (`{{`、`}`、`[[`、`]]`)。 次の表に、正規表現とエスケープ適用後のものを示します。

| 正規表現    | エスケープ適用後の正規表現     |
| --------------------- | ------------------------------ |
| `^\d{3}-\d{2}-\d{4}$` | `^\\d{{3}}-\\d{{2}}-\\d{{4}}$` |
| `^[a-z]{2}$`          | `^[[a-z]]{{2}}$`               |

ルーティングで使用される正規表現は、多くの場合、キャレット (`^`) 文字で始まり、文字列の開始位置と一致します。 表現は、多くの場合、ドル記号 (`$`) で終わり、文字列の末尾と一致します。 `^` 文字と `$` 文字により、正規表現はルート パラメーター値全体に一致します。 `^` 文字と `$` 文字がなければ、正規表現は、文字列内のあらゆる部分文字列に一致します。それは多くの場合、意図に反することです。 下の表では例を示し、それらが一致する、または一致しない理由について説明します。

| 式   | String    | 一致したもの | コメント               |
| ------------ | --------- | :---: |  -------------------- |
| `[a-z]{2}`   | hello     | はい   | サブ文字列の一致     |
| `[a-z]{2}`   | 123abc456 | はい   | サブ文字列の一致     |
| `[a-z]{2}`   | mz        | はい   | 一致する表現    |
| `[a-z]{2}`   | MZ        | はい   | 大文字と小文字の使い方が違う    |
| `^[a-z]{2}$` | hello     | いいえ    | 上の `^` と `$` を参照 |
| `^[a-z]{2}$` | 123abc456 | いいえ    | 上の `^` と `$` を参照 |

正規表現構文の詳細については、[.NET Framework 正規表現](/dotnet/standard/base-types/regular-expression-language-quick-reference)に関するページを参照してください。

既知の入力可能値の集まりにパラメーターを制限するには、正規表現を使用します。 たとえば、`{action:regex(^(list|get|create)$)}` の場合、`action` ルート値は `list`、`get`、`create` とのみ照合されます。 制約ディクショナリに渡された場合、文字列 `^(list|get|create)$` で同じものになります。 (テンプレート内でインラインではなく) 制約ディクショナリに渡された制約が既知の制約に一致しない場合も、正規表現として扱われます。

## <a name="custom-route-constraints"></a>カスタム ルート制約

組み込みのルート制約だけでなく、<xref:Microsoft.AspNetCore.Routing.IRouteConstraint> インターフェイスを実装してカスタム ルート制約を作成することができます。 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> インターフェイスには、1 つのメソッド `Match` が含まれています。これは、制約が満たされている場合は `true` を、それ以外の場合は `false` を返します。

カスタムの <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> を使うには、アプリのサービス コンテナー内にあるアプリの <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> に、ルート制約の種類を登録する必要があります。 <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> は、ルート制約キーを、その制約を検証する <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> の実装にマッピングするディクショナリです。 アプリの <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> は、`Startup.ConfigureServices` で、[services.AddRouting](xref:Microsoft.Extensions.DependencyInjection.RoutingServiceCollectionExtensions.AddRouting*) 呼び出しの一部として、または `services.Configure<RouteOptions>` を使って <xref:Microsoft.AspNetCore.Routing.RouteOptions> を直接構成することで、更新できます。 次に例を示します。

```csharp
services.AddRouting(options =>
{
    options.ConstraintMap.Add("customName", typeof(MyCustomConstraint));
});
```

これで、制約の種類を登録するときに指定した名前を使って、通常の方法でルートに制約を適用できます。 次に例を示します。

```csharp
[HttpGet("{id:customName}")]
public ActionResult<string> Get(string id)
```

## <a name="url-generation-reference"></a>URL 生成参照

次の例では、ルートのリンクを生成する方法を確認できます。ルート値のディクショナリと <xref:Microsoft.AspNetCore.Routing.RouteCollection> が指定されています。

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_Dictionary)]

上のサンプルの終わりで生成された <xref:Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath> は `/package/create/123` です。 ディクショナリにより、"Track Package Route" テンプレート `package/{operation}/{id}` のルート値 `operation` と `id` が提供されます。 詳細については、「[ルーティング ミドルウェアの使用](#use-routing-middleware)」セクションのサンプル コードまたは[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/routing/samples)を参照してください。

<xref:Microsoft.AspNetCore.Routing.VirtualPathContext> コンストラクターの 2 番目のパラメーターは *アンビエント値* の集合です。 アンビエント値は、開発者が要求コンテキスト内で指定する必要がある値の数が制限されるため、使用すると便利です。 現在の要求の現在のルート値は、リンク生成の場合、アンビエント値として見なされます。 ASP.NET Core MVC アプリの `HomeController` の `About` アクションでは、コントローラー ルート値を指定し、`Index` アクションにリンクする必要はありません。`Home` のアンビエント値が使用されます。

パラメーターに一致しないアンビエント値は無視されます。 アンビエント値は、明示的に指定された値でアンビエント値がオーバーライドされる場合にも無視されます。 URL では左から右に照合されます。

明示的に指定されているが、ルートのセグメントと一致しない値は、クエリ文字列に追加されます。 次の表は、ルート テンプレート `{controller}/{action}/{id?}` の使用時の結果をまとめたものです。

| アンビエント値                     | 明示的な値                        | 結果                  |
| ---------------------------------- | -------------------------------------- | ----------------------- |
| controller = "Home"                | action = "About"                       | `/Home/About`           |
| controller = "Home"                | controller = "Order", action = "About" | `/Order/About`          |
| controller = "Home", color = "Red" | action = "About"                       | `/Home/About`           |
| controller = "Home"                | action = "About", color = "Red"        | `/Home/About?color=Red` |

パラメーターに対応しない既定値がルートにあり、その値が明示的に指定される場合、それは既定値に一致する必要があります。

```csharp
routes.MapRoute("blog_route", "blog/{*slug}",
    defaults: new { controller = "Blog", action = "ReadPost" });
```

リンク生成では、`controller` と `action` について一致する値が指定されるときにのみ、このルートのリンクが生成されます。

## <a name="complex-segments"></a>複雑なセグメント

複雑なセグメント (例: `[Route("/x{token}y")]`) は、リテラルを右から左に最短一致の方法で照合することによって処理されます。 複雑なセグメントの一致方法に関する詳しい説明については、[こちらのコード](https://github.com/dotnet/aspnetcore/blob/v2.2.13/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293)をご覧ください。 この[コード サンプル](https://github.com/dotnet/aspnetcore/blob/v2.2.13/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293)は ASP.NET Core では使われていませんが、複雑なセグメントに関する優れた説明が提供されています。

::: moniker-end
