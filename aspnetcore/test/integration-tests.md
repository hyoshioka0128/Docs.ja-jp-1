---
title: ASP.NET Core での統合テスト
author: rick-anderson
description: 統合テストによってデータベース、ファイル システム、ネットワークなどのインフラストラクチャ レベルで、アプリのコンポーネントがどのように正しく機能するようになるかを説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/14/2020
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
uid: test/integration-tests
ms.openlocfilehash: 50c3f04c4add9d3dec032b5f2e95ce503466693e
ms.sourcegitcommit: 07e7ee573fe4e12be93249a385db745d714ff6ae
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/12/2021
ms.locfileid: "103413380"
---
# <a name="integration-tests-in-aspnet-core"></a>ASP.NET Core での統合テスト

作成者: [Javier Calvarro Nelson](https://github.com/javiercn)、[Steve Smith](https://ardalis.com/)、[Jos van der Til](https://jvandertil.nl)

::: moniker range=">= aspnetcore-3.0"

統合テストでは、データベース、ファイル システム、ネットワークなど、アプリのサポート インフラストラクチャを含むレベルで、アプリのコンポーネントが正しく機能していることを確認します。 ASP.NET Core では、単体テスト フレームワークとテスト Web ホストおよびメモリ内テスト サーバーを使用した統合テストがサポートされています。

このトピックを読むには、単体テストの基本的な知識があることが前提となります。 テストの概念を理解していない場合は、「[.NET Core の単体テストと .NET Standard」](/dotnet/core/testing/) のトピックとリンクされたコンテンツを参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/test/integration-tests/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

このサンプル アプリは Razor Pages アプリであり、Razor Pages の基本を理解していることを前提としています。 Razor Pages に慣れていない場合は、次のトピックを参照してください。

* [Razor Pages の概要](xref:razor-pages/index)
* [Razor Pages の使用を開始する](xref:tutorials/razor-pages/razor-pages-start)
* [Razor Pages の単体テスト](xref:test/razor-pages-tests)

> [!NOTE]
> SPA をテストする場合は、ブラウザーを自動化できる [Selenium](https://www.seleniumhq.org/) などのツールを推奨します。

## <a name="introduction-to-integration-tests"></a>統合テストの概要

統合テストでは [単体テスト](/dotnet/core/testing/)よりも広範なレベルでアプリのコンポーネントを評価します。 単体テストは、個々のクラス メソッドなど、分離されたソフトウェア コンポーネントをテストするために使用されます。 統合テストでは、2 つ以上のアプリ コンポーネントが、連携して予想される結果を生成することを確認します。これは、要求を完全に処理するために必要なすべてのコンポーネントを含む可能性があります。

これらの広範なテストは、アプリのインフラストラクチャとフレームワーク全体をテストするために使用されます。多くの場合、次のコンポーネントが含まれます。

* データベース
* ファイル システム
* ネットワーク アプライアンス
* 要求 - 応答パイプライン

単体テストでは、インフラストラクチャ コンポーネントの代わりに、*フェイク* または *モック オブジェクト* と呼ばれる、作成済みのコンポーネントを使用します。

単体テストと比較すると、統合テストは次のようになります。

* アプリが運用環境で使用する実際のコンポーネントを使用します。
* より多くのコードとデータ処理が必要です。
* 実行に時間がかかります。

そのため、統合テストの使用は最も重要なインフラストラクチャ シナリオに限定します。 単体テストと統合テストのどちらを使用しても動作をテストできる場合は、単体テストを選択します。

> [!TIP]
> 統合テストは、データベースとファイル システムを使用するデータおよびファイル アクセスのすべての順列として記述してはいけません。 通常は、アプリ全体でデータベースやファイル システムを操作する場所がいくつあったとしても、的を絞った一連の読み取り、書き込み、更新、削除の統合テストを行うことで、データベースおよびファイル システム コンポーネントを適切にテストすることができます。 これらのコンポーネントと連携するメソッドのロジックのルーチン テストには、単体テストを使用します。 単体テストでは、インフラストラクチャのフェイク/モックを使用することにより、テストの実行時間が短縮されます。

> [!NOTE]
> 統合テストの説明では、テスト対象のプロジェクトをよく *テスト対象システム*、または短縮して "SUT" と呼びます。
>
> *このトピック全体で、テスト対象の ASP.NET Core アプリを指すために "SUT" を使用します。*

## <a name="aspnet-core-integration-tests"></a>ASP.NET Core 統合テスト

ASP.NET Core の統合テストには、次のものが必要です。

* テスト プロジェクトは、テストを格納して実行するために使用します。 テスト プロジェクトは SUT への参照を含みます。
* テスト プロジェクトは、SUT のテスト Web ホストを作成し、テスト サーバー クライアントを使用して SUT との要求と応答を処理します。
* テスト ランナーは、テストを実行し、テスト結果を報告するために使用されます。

統合テストでは、通常の *Arrange (配置)* 、*Act (実行)* 、および *Assert (確認)* のテスト ステップを含む一連のイベントに従います。

1. SUT の Web ホストが構成されます。
1. アプリに要求を送信するためのテスト サーバー クライアントが作成されます。
1. *Arrange (配置)* テスト ステップが実行されます。テスト アプリが要求を準備します。
1. *Act (実行)* テスト ステップが実行されます。クライアントは要求を送信し、応答を受信します。
1. *Assert (確認)* テスト ステップが実行されます。*実際* の応答は、*予測される* 応答に基づき、*成功* または *失敗* として検証されます。
1. このプロセスは、すべてのテストが実行されるまで続行されます。
1. テスト結果が報告されます。

通常、テスト Web ホストは、アプリの通常のテスト用の Web ホストとは異なる方法で構成されています。 たとえば、テスト用に別のデータベースまたは異なるアプリ設定を使用する場合があります。

テスト Web ホストやメモリ内テスト サーバー ([TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver)) などのインフラストラクチャ コンポーネントは、[Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing) パッケージによって提供または管理されます。 このパッケージを使用すると、テストの作成と実行を効率化できます。

`Microsoft.AspNetCore.Mvc.Testing` パッケージは、次のタスクを処理します。

* 依存関係ファイル ( *.deps*) を SUT からテスト プロジェクトの *bin* ディレクトリにコピーします。
* テストを実行したときに、静的なファイルとページ/ビューが検出されるように、[コンテンツ ルート](xref:fundamentals/index#content-root)を SUT のプロジェクト ルートに設定します。
* [WebApplicationFactory](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1) クラスを提供し、`TestServer` を使用して SUT のブートストラップを効率化します。

[単体テストの](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)ドキュメントでは、テスト プロジェクトとテスト ランナーを設定する方法、テストを実行する方法の詳細な手順、テストおよびテスト クラスの命名方法に関する推奨事項について説明します。

> [!NOTE]
> アプリのテスト プロジェクトを作成する場合は、単体テストを統合テストから別のプロジェクトに分離します。 これにより、インフラストラクチャ テスト コンポーネントが誤って単体テストに含まれないようにすることができます。 単体テストと統合テストを分離すると、実行されるテストのセットを制御することもできます。

Razor Pages アプリと MVC アプリのテストの構成には、ほぼ違いがありません。 唯一の違いは、テストの命名方法です。 Razor Pages アプリでは、ページ エンドポイントのテストは通常、ページ モデル クラスにちなんだ名前が付けられます (たとえば、`IndexPageTests` では Index ページのコンポーネントの統合テストが行われます)。 MVC アプリでは、テストは通常、コントローラー クラス別に編成され、テストするコントローラーにちなんだ名前が付けられます (たとえば、`HomeControllerTests` は Home コントローラーのコンポーネントの統合テストを行います)。

## <a name="test-app-prerequisites"></a>テスト アプリの前提条件

テスト プロジェクトは、次の条件を満たす必要があります。

* [Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing) パッケージを参照しています。
* プロジェクト ファイル (`<Project Sdk="Microsoft.NET.Sdk.Web">`) で Web SDK を指定しています。

これらの前提条件は、[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/test/integration-tests/samples/)で確認できます。 *tests/RazorPagesProject.Tests/RazorPagesProject.Tests.csproj* ファイルを調べます。 このサンプル アプリでは、[xUnit](https://xunit.github.io/) テスト フレームワークと [AngleSharp](https://anglesharp.github.io/) パーサー ライブラリを使用するので、サンプル アプリは以下も参照します。

* [xunit](https://www.nuget.org/packages/xunit)
* [xunit.runner.visualstudio](https://www.nuget.org/packages/xunit.runner.visualstudio)
* [AngleSharp](https://www.nuget.org/packages/AngleSharp)

テストでは Entity Framework Core も使用します。 アプリは以下を参照します。

* [Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore)
* [Microsoft.AspNetCore.Identity.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Identity.EntityFrameworkCore)
* [Microsoft.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore)
* [Microsoft.EntityFrameworkCore.InMemory](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.InMemory)
* [Microsoft.EntityFrameworkCore.Tools](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools)

## <a name="sut-environment"></a>SUT 環境

SUT の [環境](xref:fundamentals/environments) が設定されていない場合、環境は既定で開発になります。

## <a name="basic-tests-with-the-default-webapplicationfactory"></a>既定の WebApplicationFactory を使用した基本的なテスト

[WebApplicationFactory\<TEntryPoint>](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1) は、統合テスト用の [TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver) を作成するために使用します。 `TEntryPoint` は SUT のエントリ ポイント クラスであり、通常は `Startup` クラスです。

テスト クラスは、クラスにテストが含まれていることを示すために *クラス フィクスチャ* インターフェイス ([IClassFixture](https://xunit.github.io/docs/shared-context#class-fixture)) を実装し、クラス内テストの共有オブジェクト インスタンスを提供します。

次のテスト クラス `BasicTests` は、`WebApplicationFactory` を使用して SUT をブートストラップし、テスト メソッド `Get_EndpointsReturnSuccessAndCorrectContentType` に [HttpClient](/dotnet/api/system.net.http.httpclient) を提供します。 このメソッドは、複数のアプリ ページで応答状態コードが成功かどうか (200-299 の範囲の状態コード) と、`Content-Type` ヘッダーが `text/html; charset=utf-8` であるかどうかを確認します。

[CreateClient](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.createclient) は `HttpClient` のインスタンスを作成します。このインスタンスは、自動的にリダイレクトに従い、cookie を処理します。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/BasicTests.cs?name=snippet1)]

既定では、[GDPR 同意ポリシー](xref:security/gdpr)が有効になっている場合、要求間で必須でない cookie は保持されません。 TempData プロバイダーで使用されているような必須ではない cookie を保持するには、テストに必須であることをマークします。 cookie を必須としてマークする手順については、[必須 cookie](xref:security/gdpr#essential-cookies) に関する記事をご覧ください。

## <a name="customize-webapplicationfactory"></a>WebApplicationFactory のカスタマイズ

Web ホストの構成は、`WebApplicationFactory` から継承して 1 つ以上のカスタム ファクトリを作成することで、テスト クラスとは別に作成できます。

1. `WebApplicationFactory` から継承し、[ConfigureWebHost](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.configurewebhost) をオーバーライドします。 [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder) では、[ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configureservices) を使用してサービス コレクションを構成できます。

   [!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/CustomWebApplicationFactory.cs?name=snippet1)]

   [サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/test/integration-tests/samples)でのデータベースのシード処理は、`InitializeDbForTests` メソッドによって実行されます。 このメソッドについては、[統合テストのサンプル: テスト アプリの構成](#test-app-organization)に関するセクションをご覧ください。

   SUT のデータベース コンテキストは、`Startup.ConfigureServices` メソッドに登録されます。 テスト アプリの `builder.ConfigureServices` コールバックは、アプリの `Startup.ConfigureServices` コードが実行された *後* に実行されます。 ASP.NET Core 3.0 リリースの[汎用ホスト](xref:fundamentals/host/generic-host)により、実行順序に関する互換性のない変更が行われています。 アプリのデータベースとは異なるデータベースをテストに使用するには、`builder.ConfigureServices` でアプリのデータベース コンテキストを置き換える必要があります。

   まだ [Web ホスト](xref:fundamentals/host/web-host)を使用している SUT の場合、テスト アプリの `builder.ConfigureServices` コールバックは、SUT の `Startup.ConfigureServices` コードの "*前に*" 実行されます。 テスト アプリの `builder.ConfigureTestServices` コールバックは、"*後で*" 実行されます。

   サンプル アプリでは、データベース コンテキストのサービス記述子を検索し、記述子を使用してサービス登録を削除しています。 次に、ファクトリは、テストにメモリ内データベースを使用する新しい `ApplicationDbContext` を追加します。

   メモリ内データベースではないデータベースに接続するには、`UseInMemoryDatabase` 呼び出しを変更して、コンテキストを別のデータベースに接続します。 SQL Server テスト データベースを使用するには、次のようにします。

   * プロジェクト ファイルで [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer/) NuGet パッケージを参照します。
   * データベースへの接続文字列を使用して `UseSqlServer` を呼び出します。

   ```csharp
   services.AddDbContext<ApplicationDbContext>((options, context) => 
   {
       context.UseSqlServer(
           Configuration.GetConnectionString("TestingDbConnectionString"));
   });
   ```

2. テスト クラスでカスタム `CustomWebApplicationFactory` を使用します。 次の例では、`IndexPageTests` クラスでファクトリを使用しています。

   [!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet1)]

   サンプル アプリのクライアントは、`HttpClient` がリダイレクトに従わないように構成されています。 [モック認証](#mock-authentication)のセクションで後述するように、これによってアプリの最初の応答結果を確認するテストが可能になります。 これらのテストでは、多くの場合、最初の応答は `Location` ヘッダーを持つリダイレクトです。

3. 一般的なテストでは、`HttpClient` およびヘルパー メソッドを使用して、要求と応答を処理します。

   [!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet2)]

SUT に対する POST 要求は、アプリの[偽造防止データ保護システム](xref:security/data-protection/introduction)によって自動的に行われる偽造防止チェックを満たす必要があります。 テストで POST 要求を実行するには、テスト アプリで次のことを行う必要があります。

1. ページに対して要求を行います。
1. 応答の偽造防止 cookie と要求検証トークンを解析します。
1. 偽造防止 cookie と要求検証トークンを使用して POST 要求を行います。

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/test/integration-tests/samples/)の `SendAsync` ヘルパー拡張メソッド (*Helpers/HttpClientExtensions.cs*) と `GetDocumentAsync` ヘルパー メソッド (*Helpers/HtmlHelpers.cs*) は、次のメソッドで [AngleSharp](https://anglesharp.github.io/) パーサーを使用して偽造防止チェック処理を行います。

* `GetDocumentAsync`:[HttpResponseMessage](/dotnet/api/system.net.http.httpresponsemessage) を受け取り、`IHtmlDocument` を返します。 `GetDocumentAsync` は、元の `HttpResponseMessage` に基づいて *仮想応答* を準備するファクトリを使用します。 詳しくは、[AngleSharp のドキュメント](https://github.com/AngleSharp/AngleSharp#documentation)をご覧ください。
* `HttpClient` の `SendAsync` 拡張メソッドは、[HttpRequestMessage](/dotnet/api/system.net.http.httprequestmessage) を作成し、[SendAsync(HttpRequestMessage)](/dotnet/api/system.net.http.httpclient.sendasync#System_Net_Http_HttpClient_SendAsync_System_Net_Http_HttpRequestMessage_) を呼び出して、SUT に要求を送信します。 `SendAsync` のオーバーロードは、HTML フォーム (`IHtmlFormElement`) と次のものを受け入れます。
  * フォームの送信ボタン (`IHtmlElement`)
  * フォームの値コレクション (`IEnumerable<KeyValuePair<string, string>>`)
  * 送信ボタン (`IHtmlElement`) とフォームの値 (`IEnumerable<KeyValuePair<string, string>>`)

> [!NOTE]
> [AngleSharp](https://anglesharp.github.io/) は、このトピックとサンプル アプリのデモンストレーションのために使用するサードパーティ製の解析ライブラリです。 ASP.NET Core アプリの統合テストでは、AngleSharp はサポートされていないか、必要ありません。 [Html Agility Pack (HAP)](https://html-agility-pack.net/) などの他のパーサーを使用することもできます。 もう 1 つの方法として、偽造防止システムの要求検証トークンを処理するコードを記述し、偽造防止 cookie を直接処理する方法もあります。

## <a name="customize-the-client-with-withwebhostbuilder"></a>WithWebHostBuilder を使用したクライアントのカスタマイズ

テスト メソッド内で追加の構成が必要な場合は、[WithWebHostBuilder](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.withwebhostbuilder) が構成によってさらにカスタマイズされた [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder) を使用して新しい `WebApplicationFactory` を作成します。

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/test/integration-tests/samples) の `Post_DeleteMessageHandler_ReturnsRedirectToRoot` テスト メソッドは、`WithWebHostBuilder` の使用方法を示しています。 このテストでは、SUT からフォーム送信をトリガーすることによって、データベース内のレコードの削除を実行します。

`IndexPageTests` クラス内の別のテストは、データベース内のすべてのレコードを削除する操作を実行します。この操作が `Post_DeleteMessageHandler_ReturnsRedirectToRoot` メソッドの前に実行される可能性があるため、このテスト メソッド内でデータベースの再シード処理を行い、確実に SUT が削除するレコードが存在するようにしています。 SUT に対する要求では、SUT の `messages` フォームの最初の [削除] ボタンの選択がシミュレートされます。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet3)]

## <a name="client-options"></a>クライアントのオプション

次の表に、`HttpClient` インスタンスを作成するときに使用できる既定の [WebApplicationFactoryClientOptions](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions) を示します。

| オプション | 説明 | Default |
| ------ | ----------- | ------- |
| [AllowAutoRedirect](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.allowautoredirect) | `HttpClient` インスタンスがリダイレクト応答に自動的に従うかどうかを取得または設定します。 | `true` |
| [BaseAddress](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.baseaddress) | `HttpClient` インスタンスのベース アドレスを取得または設定します。 | `http://localhost` |
| [HandleCookies](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.handlecookies) | `HttpClient` インスタンスが cookie を処理する必要があるかどうかを取得または設定します。 | `true` |
| [MaxAutomaticRedirections](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.maxautomaticredirections) | `HttpClient` インスタンスが従う必要があるリダイレクト応答の最大数を取得または設定します。 | 7 |

`WebApplicationFactoryClientOptions` クラスを作成し、それを [CreateClient](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.createclient) メソッドに渡します (既定値については、コード例を参照してください)。

```csharp
// Default client option values are shown
var clientOptions = new WebApplicationFactoryClientOptions();
clientOptions.AllowAutoRedirect = true;
clientOptions.BaseAddress = new Uri("http://localhost");
clientOptions.HandleCookies = true;
clientOptions.MaxAutomaticRedirections = 7;

_client = _factory.CreateClient(clientOptions);
```

## <a name="inject-mock-services"></a>モック サービスの注入

ホスト ビルダーで [ConfigureTestServices](/dotnet/api/microsoft.aspnetcore.testhost.webhostbuilderextensions.configuretestservices) を呼び出すと、テストでサービスをオーバーライドできます。 **モック サービスを注入するには、SUT に `Startup` クラスが存在し、そこに `Startup.ConfigureServices` メソッドが存在している必要があります。**

サンプルの SUT には、引用符を返すスコープ サービスが含まれています。 インデックス ページが要求されると、インデックス ページの非表示フィールドに引用符が埋め込まれます。

*Services/IQuoteService.cs*:

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Services/IQuoteService.cs?name=snippet1)]

*Services/QuoteService.cs*:

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Services/QuoteService.cs?name=snippet1)]

*Startup.cs*:

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Startup.cs?name=snippet2)]

*Pages/Index.cshtml.cs*:

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Pages/Index.cshtml.cs?name=snippet1&highlight=4,9,20,26)]

*Pages/Index.cs*:

[!code-cshtml[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Pages/Index.cshtml?name=snippet_Quote)]

次のマークアップは、SUT アプリの実行時に生成されます。

```html
<input id="quote" type="hidden" value="Come on, Sarah. We&#x27;ve an appointment in 
    London, and we&#x27;re already 30,000 years late.">
```

統合テストでサービスと引用符の注入をテストするため、テストは SUT にモック サービスを注入します。 モック サービスは、アプリの `QuoteService` をテスト アプリが提供する `TestQuoteService` と呼ばれるサービスに置き換えます。

*IntegrationTests.IndexPageTests.cs*:

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet4)]

`ConfigureTestServices` が呼び出され、スコープ サービスが登録されます。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet5&highlight=7-10,17,20-21)]

`TestQuoteService` によって指定された引用符テキストがテストの実行中に生成されたマークアップに反映されるため、アサーションは成功します。

```html
<input id="quote" type="hidden" value="Something&#x27;s interfering with time, 
    Mr. Scarman, and time is my business.">
```

## <a name="mock-authentication"></a>モック認証

`AuthTests` クラスのテストは、セキュリティで保護されたエンドポイントであることを確認します。

* 認証されていないユーザーは、アプリのログイン ページにリダイレクトされます。
* 認証されたユーザーには、コンテンツを返します。

SUT の `/SecurePage` ページは、[AuthorizePage](/dotnet/api/microsoft.extensions.dependencyinjection.pageconventioncollectionextensions.authorizepage) 規約を使用してページに [AuthorizeFilter](/dotnet/api/microsoft.aspnetcore.mvc.authorization.authorizefilter) を適用します。 詳細については、[Razor Pages の承認規則](xref:security/authorization/razor-pages-authorization#require-authorization-to-access-a-page)に関するページを参照してください。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Startup.cs?name=snippet1)]

`Get_SecurePageRedirectsAnUnauthenticatedUser` テストでは、[AllowAutoRedirect](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.allowautoredirect) を `false` に設定することで、[WebApplicationFactoryClientOptions](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions) がリダイレクトを許可しないように設定しています。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet2)]

クライアントがリダイレクトに従うことを許可しないことで、次のチェックを行うことができます。

* ログイン ページにリダイレクトした後の最終的な状態コード (これは [HttpStatusCode.OK](/dotnet/api/system.net.httpstatuscode) になります) ではなく、予期される [HttpStatusCode.Redirect](/dotnet/api/system.net.httpstatuscode) の結果を SUT が返すステータス コードと突き合わせて確認できます。
* 最終的なログイン ページ応答 (ここでは、`Location` ヘッダーは存在しません) ではなく、応答ヘッダーの `Location` ヘッダー値が `http://localhost/Identity/Account/Login` で始まることを確認できます。

テスト アプリでは、認証と承認の側面をテストするために [ConfigureTestServices](/dotnet/api/microsoft.aspnetcore.testhost.webhostbuilderextensions.configuretestservices) の <xref:Microsoft.AspNetCore.Authentication.AuthenticationHandler`1> をモックすることができます。 最小のシナリオでは、[AuthenticateResult.Success](xref:Microsoft.AspNetCore.Authentication.AuthenticateResult.Success*) が返されます。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet4&highlight=11-18)]

`TestAuthHandler` は、認証スキームが `Test` に設定されている場合 (この場合、`AddAuthentication` が `ConfigureTestServices` に登録されています) に、ユーザーを認証するために呼び出されます。 アプリから要求されるスキームと `Test` スキームを一致させることが重要です。 そうでない場合は、認証が機能しません。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet3&highlight=7-12)]

`WebApplicationFactoryClientOptions` の詳細については、「[クライアントのオプション](#client-options)」セクションを参照してください。

## <a name="set-the-environment"></a>環境を設定する

既定では、SUT のホストとアプリ環境は、開発環境を使用するように構成されています。 `IHostBuilder` を使用しているときに SUT の環境をオーバーライドするには、次のようにします。

* `ASPNETCORE_ENVIRONMENT` 環境変数 (たとえば、`Staging`、`Production`、または `Testing` などのカスタム値) を設定します。
* テスト アプリで `CreateHostBuilder` をオーバーライドして、`ASPNETCORE` で始まる環境変数を読み取ります。

```csharp
protected override IHostBuilder CreateHostBuilder() =>
    base.CreateHostBuilder()
        .ConfigureHostConfiguration(
            config => config.AddEnvironmentVariables("ASPNETCORE"));
```

SUT が Web ホスト (`IWebHostBuilder`) を使用している場合は、`CreateWebHostBuilder` をオーバーライドします。

```csharp
protected override IWebHostBuilder CreateWebHostBuilder() =>
    base.CreateWebHostBuilder().UseEnvironment("Testing");
```

## <a name="how-the-test-infrastructure-infers-the-app-content-root-path"></a>テスト インフラストラクチャがアプリ コンテンツのルート パスを推測する方法

`WebApplicationFactory` コンストラクターは、`TEntryPoint` アセンブリの `System.Reflection.Assembly.FullName` と同じキーを持つ統合テストを含むアセンブリで [WebApplicationFactoryContentRootAttribute](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactorycontentrootattribute) を検索することによって、アプリの[コンテンツ ルート](xref:fundamentals/index#content-root) パスを推測します。 正しいキーを持つ属性が見つからない場合、`WebApplicationFactory` はフォールバックしてソリューション ファイル ( *.sln*) を検索し、`TEntryPoint` アセンブリ名をソリューション ディレクトリに追加します。 アプリのルート ディレクトリ (コンテンツ ルート パス) は、ビューやコンテンツのファイルを検出するために使用されます。

## <a name="disable-shadow-copying"></a>シャドウ コピーの無効化

シャドウ コピーを行うと、テストが出力ディレクトリとは異なるディレクトリで実行されます。 テストが適切に機能するように、シャドウ コピーを無効化する必要があります。 [サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/test/integration-tests/samples)では、xUnit を使用して正しい構成設定の *xunit.runner.json* ファイルを含めることにより、xUnit のシャドウ コピーを無効化しています。 詳細については、[xUnit のドキュメント](https://xunit.net/docs/configuration-files)を参照してください。

次の内容を使用して、テスト プロジェクトのルートに *xunit.runner.json* ファイルを追加します。

```json
{
  "shadowCopy": false
}
```

## <a name="disposal-of-objects"></a>オブジェクトの破棄

`IClassFixture` 実装のテストを実行した後、xUnit が [WebApplicationFactory](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1) を破棄すると、[TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver) と [HttpClient](/dotnet/api/system.net.http.httpclient) が破棄されます。 開発者がインスタンス化したオブジェクトを破棄する必要がある場合は、`IClassFixture` の実装で破棄します。 詳細については、「[Dispose メソッドの実装](/dotnet/standard/garbage-collection/implementing-dispose)」を参照してください。

## <a name="integration-tests-sample"></a>統合テストのサンプル

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/test/integration-tests/samples)は、次の 2 つのアプリで構成されています。

| アプリ | プロジェクト ディレクトリ | 説明 |
| --- | ----------------- | ----------- |
| メッセージ アプリ (SUT) | *src/RazorPagesProject* | ユーザーは、メッセージの追加、1 つ削除、すべて削除、および分析を行うことができます。 |
| アプリをテストする | *tests/RazorPagesProject.Tests* | SUT の統合テストに使用されます。 |

テストは、[Visual Studio](https://visualstudio.microsoft.com) などの IDE に組み込まれているテスト機能を使用して実行できます。 [Visual Studio Code](https://code.visualstudio.com/) またはコマンド ラインを使用している場合は、コマンド プロンプトで *tests/RazorPagesProject.Tests* ディレクトリを開き、次のコマンドを実行します。

```console
dotnet test
```

### <a name="message-app-sut-organization"></a>メッセージ アプリ (SUT) の構成

SUT は、次の特性を持つ Razor Pages メッセージ システムです。

* アプリのインデックス ページ (*Pages/Index.cshtml* と *Pages/Index.cshtml.cs*) には、メッセージの追加、削除、および分析 (メッセージあたりの平均単語数) を制御する UI およびページ モデル メソッドが用意されています。
* メッセージは、`Id` (キー) と `Text` (メッセージ) の 2 つのプロパティを持つ `Message` クラス (*Data/Message.cs*) によって記述されます。 `Text` プロパティは必須であり、200 文字までに制限されています。
* メッセージは [Entity Framework のメモリ内データベース](/ef/core/providers/in-memory/)&#8224; を使用して格納されます。
* アプリのデータベース コンテキスト クラスである `AppDbContext` (*Data/AppDbContext.cs*) には、データアクセス層 (DAL) が含まれています。
* アプリの起動時にデータベースが空の場合、メッセージ ストアが 3 つのメッセージで初期化されます。
* アプリには、認証されたユーザーのみがアクセスできる `/SecurePage` が含まれています。

&#8224; EF トピック「[InMemory を使用したテスト](/ef/core/miscellaneous/testing/in-memory)」では、MSTest を使用したテストでメモリ内データベースを使用する方法について説明します。 このトピックでは、[xUnit](https://xunit.github.io/) テスト フレームワークを使用します。 テストの概念とテストの実装は、異なるテスト フレームワークでも類似していますが、同一ではありません。

アプリはリポジトリ パターンを使用しておらず、[Unit of Work (UoW) パターン](https://martinfowler.com/eaaCatalog/unitOfWork.html)の有効な例ではありませんが、Razor Pages ではこれらの開発パターンがサポートされています。 詳細については、[インフラストラクチャの永続化レイヤーの設計](/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)および[コントローラー ロジックのテスト](../mvc/controllers/testing.md)に関する記事を参照してください (このサンプルはリポジトリ パターンを実装しています)。

### <a name="test-app-organization"></a>テスト アプリの構成

テスト アプリは、*tests/RazorPagesProject.Tests* ディレクトリにあるコンソール アプリです。

| テスト アプリのディレクトリ | 説明 |
| ------------------ | ----------- |
| *AuthTests* | 次のテスト メソッドを含みます。<ul><li>認証されていないユーザーがセキュリティで保護されたページにアクセスする。</li><li>モック <xref:Microsoft.AspNetCore.Authentication.AuthenticationHandler`1> で認証されたユーザーがセキュリティで保護されたページにアクセスする。</li><li>GitHub ユーザー プロファイルを取得し、プロファイルのユーザー ログインを確認する。</li></ul> |
| *BasicTests* | ルーティングおよびコンテンツ タイプのテスト メソッドを含みます。 |
| *IntegrationTests* | カスタム `WebApplicationFactory` クラスを使用したインデックス ページの統合テストを含みます。 |
| *Helpers/Utilities* | <ul><li>*Utilities.cs* には、データベースにテスト データをシードする `InitializeDbForTests` メソッドが含まれています。</li><li>*HtmlHelpers.cs* には、テスト メソッドで使用する AngleSharp `IHtmlDocument` を返すメソッドが用意されています。</li><li>*HttpClientExtensions.cs* は、SUT に要求を送信する `SendAsync` のオーバーロードを提供します。</li></ul> |

テスト フレームワークは、[xUnit](https://xunit.github.io/) です。 統合テストは、[TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver) を含む [Microsoft.AspNetCore.TestHost](/dotnet/api/microsoft.aspnetcore.testhost) を使用して実行されます。 [Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing) パッケージはテスト ホストとテスト サーバーを構成するために使用されるため、テスト アプリのプロジェクト ファイルまたはテスト アプリの開発者構成で直接 `TestHost` および `TestServer` パッケージを参照する必要はありません。

**テスト用のデータベースのシード処理**

統合テストでは、通常、テストを実行する前に、データベース内に小さなデータセットが必要です。 たとえば、削除テストでは、データベース レコードの削除を呼び出します。そのため、削除要求を成功させるには、データベースに少なくとも 1 つのレコードが必要です。

このサンプル アプリでは、*Utilities.cs* で 3 つのメッセージを使用してデータベースをシードします。このメッセージは、テストを実行する際に使用することができます。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/Helpers/Utilities.cs?name=snippet1)]

SUT のデータベース コンテキストは、`Startup.ConfigureServices` メソッドに登録されます。 テスト アプリの `builder.ConfigureServices` コールバックは、アプリの `Startup.ConfigureServices` コードが実行された *後* に実行されます。 テストに異なるデータベースを使用するには、アプリのデータベース コンテキストを `builder.ConfigureServices`で置き換える必要があります。 詳細については、「[WebApplicationFactory のカスタマイズ](#customize-webapplicationfactory)」セクションをご覧ください。

まだ [Web ホスト](xref:fundamentals/host/web-host)を使用している SUT の場合、テスト アプリの `builder.ConfigureServices` コールバックは、SUT の `Startup.ConfigureServices` コードの "*前に*" 実行されます。 テスト アプリの `builder.ConfigureTestServices` コールバックは、"*後で*" 実行されます。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

統合テストでは、データベース、ファイル システム、ネットワークなど、アプリのサポート インフラストラクチャを含むレベルで、アプリのコンポーネントが正しく機能していることを確認します。 ASP.NET Core では、単体テスト フレームワークとテスト Web ホストおよびメモリ内テスト サーバーを使用した統合テストがサポートされています。

このトピックを読むには、単体テストの基本的な知識があることが前提となります。 テストの概念を理解していない場合は、「[.NET Core の単体テストと .NET Standard」](/dotnet/core/testing/) のトピックとリンクされたコンテンツを参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/test/integration-tests/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

このサンプル アプリは Razor Pages アプリであり、Razor Pages の基本を理解していることを前提としています。 Razor Pages に慣れていない場合は、次のトピックを参照してください。

* [Razor Pages の概要](xref:razor-pages/index)
* [Razor Pages の使用を開始する](xref:tutorials/razor-pages/razor-pages-start)
* [Razor Pages の単体テスト](xref:test/razor-pages-tests)

> [!NOTE]
> SPA をテストする場合は、ブラウザーを自動化できる [Selenium](https://www.seleniumhq.org/) などのツールを推奨します。

## <a name="introduction-to-integration-tests"></a>統合テストの概要

統合テストでは [単体テスト](/dotnet/core/testing/)よりも広範なレベルでアプリのコンポーネントを評価します。 単体テストは、個々のクラス メソッドなど、分離されたソフトウェア コンポーネントをテストするために使用されます。 統合テストでは、2 つ以上のアプリ コンポーネントが、連携して予想される結果を生成することを確認します。これは、要求を完全に処理するために必要なすべてのコンポーネントを含む可能性があります。

これらの広範なテストは、アプリのインフラストラクチャとフレームワーク全体をテストするために使用されます。多くの場合、次のコンポーネントが含まれます。

* データベース
* ファイル システム
* ネットワーク アプライアンス
* 要求 - 応答パイプライン

単体テストでは、インフラストラクチャ コンポーネントの代わりに、*フェイク* または *モック オブジェクト* と呼ばれる、作成済みのコンポーネントを使用します。

単体テストと比較すると、統合テストは次のようになります。

* アプリが運用環境で使用する実際のコンポーネントを使用します。
* より多くのコードとデータ処理が必要です。
* 実行に時間がかかります。

そのため、統合テストの使用は最も重要なインフラストラクチャ シナリオに限定します。 単体テストと統合テストのどちらを使用しても動作をテストできる場合は、単体テストを選択します。

> [!TIP]
> 統合テストは、データベースとファイル システムを使用するデータおよびファイル アクセスのすべての順列として記述してはいけません。 通常は、アプリ全体でデータベースやファイル システムを操作する場所がいくつあったとしても、的を絞った一連の読み取り、書き込み、更新、削除の統合テストを行うことで、データベースおよびファイル システム コンポーネントを適切にテストすることができます。 これらのコンポーネントと連携するメソッドのロジックのルーチン テストには、単体テストを使用します。 単体テストでは、インフラストラクチャのフェイク/モックを使用することにより、テストの実行時間が短縮されます。

> [!NOTE]
> 統合テストの説明では、テスト対象のプロジェクトをよく *テスト対象システム*、または短縮して "SUT" と呼びます。
>
> *このトピック全体で、テスト対象の ASP.NET Core アプリを指すために "SUT" を使用します。*

## <a name="aspnet-core-integration-tests"></a>ASP.NET Core 統合テスト

ASP.NET Core の統合テストには、次のものが必要です。

* テスト プロジェクトは、テストを格納して実行するために使用します。 テスト プロジェクトは SUT への参照を含みます。
* テスト プロジェクトは、SUT のテスト Web ホストを作成し、テスト サーバー クライアントを使用して SUT との要求と応答を処理します。
* テスト ランナーは、テストを実行し、テスト結果を報告するために使用されます。

統合テストでは、通常の *Arrange (配置)* 、*Act (実行)* 、および *Assert (確認)* のテスト ステップを含む一連のイベントに従います。

1. SUT の Web ホストが構成されます。
1. アプリに要求を送信するためのテスト サーバー クライアントが作成されます。
1. *Arrange (配置)* テスト ステップが実行されます。テスト アプリが要求を準備します。
1. *Act (実行)* テスト ステップが実行されます。クライアントは要求を送信し、応答を受信します。
1. *Assert (確認)* テスト ステップが実行されます。*実際* の応答は、*予測される* 応答に基づき、*成功* または *失敗* として検証されます。
1. このプロセスは、すべてのテストが実行されるまで続行されます。
1. テスト結果が報告されます。

通常、テスト Web ホストは、アプリの通常のテスト用の Web ホストとは異なる方法で構成されています。 たとえば、テスト用に別のデータベースまたは異なるアプリ設定を使用する場合があります。

テスト Web ホストやメモリ内テスト サーバー ([TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver)) などのインフラストラクチャ コンポーネントは、[Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing) パッケージによって提供または管理されます。 このパッケージを使用すると、テストの作成と実行を効率化できます。

`Microsoft.AspNetCore.Mvc.Testing` パッケージは、次のタスクを処理します。

* 依存関係ファイル ( *.deps*) を SUT からテスト プロジェクトの *bin* ディレクトリにコピーします。
* テストを実行したときに、静的なファイルとページ/ビューが検出されるように、[コンテンツ ルート](xref:fundamentals/index#content-root)を SUT のプロジェクト ルートに設定します。
* [WebApplicationFactory](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1) クラスを提供し、`TestServer` を使用して SUT のブートストラップを効率化します。

[単体テストの](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)ドキュメントでは、テスト プロジェクトとテスト ランナーを設定する方法、テストを実行する方法の詳細な手順、テストおよびテスト クラスの命名方法に関する推奨事項について説明します。

> [!NOTE]
> アプリのテスト プロジェクトを作成する場合は、単体テストを統合テストから別のプロジェクトに分離します。 これにより、インフラストラクチャ テスト コンポーネントが誤って単体テストに含まれないようにすることができます。 単体テストと統合テストを分離すると、実行されるテストのセットを制御することもできます。

Razor Pages アプリと MVC アプリのテストの構成には、ほぼ違いがありません。 唯一の違いは、テストの命名方法です。 Razor Pages アプリでは、ページ エンドポイントのテストは通常、ページ モデル クラスにちなんだ名前が付けられます (たとえば、`IndexPageTests` では Index ページのコンポーネントの統合テストが行われます)。 MVC アプリでは、テストは通常、コントローラー クラス別に編成され、テストするコントローラーにちなんだ名前が付けられます (たとえば、`HomeControllerTests` は Home コントローラーのコンポーネントの統合テストを行います)。

## <a name="test-app-prerequisites"></a>テスト アプリの前提条件

テスト プロジェクトは、次の条件を満たす必要があります。

* 次のパッケージを参照します。
  * [Microsoft.AspNetCore.App](https://www.nuget.org/packages/Microsoft.AspNetCore.App/)
  * [Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing/)
* プロジェクト ファイル (`<Project Sdk="Microsoft.NET.Sdk.Web">`) で Web SDK を指定しています。 [Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)を参照する場合は、Web SDK が必要です。

これらの前提条件は、[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/test/integration-tests/samples/)で確認できます。 *tests/RazorPagesProject.Tests/RazorPagesProject.Tests.csproj* ファイルを調べます。 このサンプル アプリでは、[xUnit](https://xunit.github.io/) テスト フレームワークと [AngleSharp](https://anglesharp.github.io/) パーサー ライブラリを使用するので、サンプル アプリは以下も参照します。

* [xunit](https://www.nuget.org/packages/xunit/)
* [xunit.runner.visualstudio](https://www.nuget.org/packages/xunit.runner.visualstudio/)
* [AngleSharp](https://www.nuget.org/packages/AngleSharp/)

## <a name="sut-environment"></a>SUT 環境

SUT の [環境](xref:fundamentals/environments) が設定されていない場合、環境は既定で開発になります。

## <a name="basic-tests-with-the-default-webapplicationfactory"></a>既定の WebApplicationFactory を使用した基本的なテスト

[WebApplicationFactory\<TEntryPoint>](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1) は、統合テスト用の [TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver) を作成するために使用します。 `TEntryPoint` は SUT のエントリ ポイント クラスであり、通常は `Startup` クラスです。

テスト クラスは、クラスにテストが含まれていることを示すために *クラス フィクスチャ* インターフェイス ([IClassFixture](https://xunit.github.io/docs/shared-context#class-fixture)) を実装し、クラス内テストの共有オブジェクト インスタンスを提供します。

次のテスト クラス `BasicTests` は、`WebApplicationFactory` を使用して SUT をブートストラップし、テスト メソッド `Get_EndpointsReturnSuccessAndCorrectContentType` に [HttpClient](/dotnet/api/system.net.http.httpclient) を提供します。 このメソッドは、複数のアプリ ページで応答状態コードが成功かどうか (200-299 の範囲の状態コード) と、`Content-Type` ヘッダーが `text/html; charset=utf-8` であるかどうかを確認します。

[CreateClient](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.createclient) は `HttpClient` のインスタンスを作成します。このインスタンスは、自動的にリダイレクトに従い、cookie を処理します。

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/BasicTests.cs?name=snippet1)]

既定では、[GDPR 同意ポリシー](xref:security/gdpr)が有効になっている場合、要求間で必須でない cookie は保持されません。 TempData プロバイダーで使用されているような必須ではない cookie を保持するには、テストに必須であることをマークします。 cookie を必須としてマークする手順については、[必須 cookie](xref:security/gdpr#essential-cookies) に関する記事をご覧ください。

## <a name="customize-webapplicationfactory"></a>WebApplicationFactory のカスタマイズ

Web ホストの構成は、`WebApplicationFactory` から継承して 1 つ以上のカスタム ファクトリを作成することで、テスト クラスとは別に作成できます。

1. `WebApplicationFactory` から継承し、[ConfigureWebHost](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.configurewebhost) をオーバーライドします。 [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder) では、[ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configureservices) を使用してサービス コレクションを構成できます。これはアプリの `Startup.ConfigureServices` の前に実行されます。 [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder) では、[ConfigureTestServices](/dotnet/api/microsoft.aspnetcore.testhost.webhostbuilderextensions.configuretestservices) を使用して、サービス コレクション内の登録済みサービスをオーバーライドしたり変更したりできます。

   [!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/CustomWebApplicationFactory.cs?name=snippet1)]

   [サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/test/integration-tests/samples)でのデータベースのシード処理は、`InitializeDbForTests` メソッドによって実行されます。 このメソッドについては、[統合テストのサンプル: テスト アプリの構成](#test-app-organization)に関するセクションをご覧ください。

2. テスト クラスでカスタム `CustomWebApplicationFactory` を使用します。 次の例では、`IndexPageTests` クラスでファクトリを使用しています。

   [!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet1)]

   サンプル アプリのクライアントは、`HttpClient` がリダイレクトに従わないように構成されています。 [モック認証](#mock-authentication)のセクションで後述するように、これによってアプリの最初の応答結果を確認するテストが可能になります。 これらのテストでは、多くの場合、最初の応答は `Location` ヘッダーを持つリダイレクトです。

3. 一般的なテストでは、`HttpClient` およびヘルパー メソッドを使用して、要求と応答を処理します。

   [!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet2)]

SUT に対する POST 要求は、アプリの[偽造防止データ保護システム](xref:security/data-protection/introduction)によって自動的に行われる偽造防止チェックを満たす必要があります。 テストで POST 要求を実行するには、テスト アプリで次のことを行う必要があります。

1. ページに対して要求を行います。
1. 応答の偽造防止 cookie と要求検証トークンを解析します。
1. 偽造防止 cookie と要求検証トークンを使用して POST 要求を行います。

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/test/integration-tests/samples/)の `SendAsync` ヘルパー拡張メソッド (*Helpers/HttpClientExtensions.cs*) と `GetDocumentAsync` ヘルパー メソッド (*Helpers/HtmlHelpers.cs*) は、次のメソッドで [AngleSharp](https://anglesharp.github.io/) パーサーを使用して偽造防止チェック処理を行います。

* `GetDocumentAsync`:[HttpResponseMessage](/dotnet/api/system.net.http.httpresponsemessage) を受け取り、`IHtmlDocument` を返します。 `GetDocumentAsync` は、元の `HttpResponseMessage` に基づいて *仮想応答* を準備するファクトリを使用します。 詳しくは、[AngleSharp のドキュメント](https://github.com/AngleSharp/AngleSharp#documentation)をご覧ください。
* `HttpClient` の `SendAsync` 拡張メソッドは、[HttpRequestMessage](/dotnet/api/system.net.http.httprequestmessage) を作成し、[SendAsync(HttpRequestMessage)](/dotnet/api/system.net.http.httpclient.sendasync#System_Net_Http_HttpClient_SendAsync_System_Net_Http_HttpRequestMessage_) を呼び出して、SUT に要求を送信します。 `SendAsync` のオーバーロードは、HTML フォーム (`IHtmlFormElement`) と次のものを受け入れます。
  * フォームの送信ボタン (`IHtmlElement`)
  * フォームの値コレクション (`IEnumerable<KeyValuePair<string, string>>`)
  * 送信ボタン (`IHtmlElement`) とフォームの値 (`IEnumerable<KeyValuePair<string, string>>`)

> [!NOTE]
> [AngleSharp](https://anglesharp.github.io/) は、このトピックとサンプル アプリのデモンストレーションのために使用するサードパーティ製の解析ライブラリです。 ASP.NET Core アプリの統合テストでは、AngleSharp はサポートされていないか、必要ありません。 [Html Agility Pack (HAP)](https://html-agility-pack.net/) などの他のパーサーを使用することもできます。 もう 1 つの方法として、偽造防止システムの要求検証トークンを処理するコードを記述し、偽造防止 cookie を直接処理する方法もあります。

## <a name="customize-the-client-with-withwebhostbuilder"></a>WithWebHostBuilder を使用したクライアントのカスタマイズ

テスト メソッド内で追加の構成が必要な場合は、[WithWebHostBuilder](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.withwebhostbuilder) が構成によってさらにカスタマイズされた [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder) を使用して新しい `WebApplicationFactory` を作成します。

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/test/integration-tests/samples) の `Post_DeleteMessageHandler_ReturnsRedirectToRoot` テスト メソッドは、`WithWebHostBuilder` の使用方法を示しています。 このテストでは、SUT からフォーム送信をトリガーすることによって、データベース内のレコードの削除を実行します。

`IndexPageTests` クラス内の別のテストは、データベース内のすべてのレコードを削除する操作を実行します。この操作が `Post_DeleteMessageHandler_ReturnsRedirectToRoot` メソッドの前に実行される可能性があるため、このテスト メソッド内でデータベースの再シード処理を行い、確実に SUT が削除するレコードが存在するようにしています。 SUT に対する要求では、SUT の `messages` フォームの最初の [削除] ボタンの選択がシミュレートされます。

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet3)]

## <a name="client-options"></a>クライアントのオプション

次の表に、`HttpClient` インスタンスを作成するときに使用できる既定の [WebApplicationFactoryClientOptions](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions) を示します。

| オプション | 説明 | Default |
| ------ | ----------- | ------- |
| [AllowAutoRedirect](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.allowautoredirect) | `HttpClient` インスタンスがリダイレクト応答に自動的に従うかどうかを取得または設定します。 | `true` |
| [BaseAddress](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.baseaddress) | `HttpClient` インスタンスのベース アドレスを取得または設定します。 | `http://localhost` |
| [HandleCookies](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.handlecookies) | `HttpClient` インスタンスが cookie を処理する必要があるかどうかを取得または設定します。 | `true` |
| [MaxAutomaticRedirections](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.maxautomaticredirections) | `HttpClient` インスタンスが従う必要があるリダイレクト応答の最大数を取得または設定します。 | 7 |

`WebApplicationFactoryClientOptions` クラスを作成し、それを [CreateClient](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.createclient) メソッドに渡します (既定値については、コード例を参照してください)。

```csharp
// Default client option values are shown
var clientOptions = new WebApplicationFactoryClientOptions();
clientOptions.AllowAutoRedirect = true;
clientOptions.BaseAddress = new Uri("http://localhost");
clientOptions.HandleCookies = true;
clientOptions.MaxAutomaticRedirections = 7;

_client = _factory.CreateClient(clientOptions);
```

## <a name="inject-mock-services"></a>モック サービスの注入

ホスト ビルダーで [ConfigureTestServices](/dotnet/api/microsoft.aspnetcore.testhost.webhostbuilderextensions.configuretestservices) を呼び出すと、テストでサービスをオーバーライドできます。 **モック サービスを注入するには、SUT に `Startup` クラスが存在し、そこに `Startup.ConfigureServices` メソッドが存在している必要があります。**

サンプルの SUT には、引用符を返すスコープ サービスが含まれています。 インデックス ページが要求されると、インデックス ページの非表示フィールドに引用符が埋め込まれます。

*Services/IQuoteService.cs*:

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Services/IQuoteService.cs?name=snippet1)]

*Services/QuoteService.cs*:

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Services/QuoteService.cs?name=snippet1)]

*Startup.cs*:

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Startup.cs?name=snippet2)]

*Pages/Index.cshtml.cs*:

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Pages/Index.cshtml.cs?name=snippet1&highlight=4,9,20,26)]

*Pages/Index.cs*:

[!code-cshtml[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Pages/Index.cshtml?name=snippet_Quote)]

次のマークアップは、SUT アプリの実行時に生成されます。

```html
<input id="quote" type="hidden" value="Come on, Sarah. We&#x27;ve an appointment in 
    London, and we&#x27;re already 30,000 years late.">
```

統合テストでサービスと引用符の注入をテストするため、テストは SUT にモック サービスを注入します。 モック サービスは、アプリの `QuoteService` をテスト アプリが提供する `TestQuoteService` と呼ばれるサービスに置き換えます。

*IntegrationTests.IndexPageTests.cs*:

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet4)]

`ConfigureTestServices` が呼び出され、スコープ サービスが登録されます。

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet5&highlight=7-10,17,20-21)]

`TestQuoteService` によって指定された引用符テキストがテストの実行中に生成されたマークアップに反映されるため、アサーションは成功します。

```html
<input id="quote" type="hidden" value="Something&#x27;s interfering with time, 
    Mr. Scarman, and time is my business.">
```

## <a name="mock-authentication"></a>モック認証

`AuthTests` クラスのテストは、セキュリティで保護されたエンドポイントであることを確認します。

* 認証されていないユーザーは、アプリのログイン ページにリダイレクトされます。
* 認証されたユーザーには、コンテンツを返します。

SUT の `/SecurePage` ページは、[AuthorizePage](/dotnet/api/microsoft.extensions.dependencyinjection.pageconventioncollectionextensions.authorizepage) 規約を使用してページに [AuthorizeFilter](/dotnet/api/microsoft.aspnetcore.mvc.authorization.authorizefilter) を適用します。 詳細については、[Razor Pages の承認規則](xref:security/authorization/razor-pages-authorization#require-authorization-to-access-a-page)に関するページを参照してください。

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Startup.cs?name=snippet1)]

`Get_SecurePageRedirectsAnUnauthenticatedUser` テストでは、[AllowAutoRedirect](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.allowautoredirect) を `false` に設定することで、[WebApplicationFactoryClientOptions](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions) がリダイレクトを許可しないように設定しています。

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet2)]

クライアントがリダイレクトに従うことを許可しないことで、次のチェックを行うことができます。

* ログイン ページにリダイレクトした後の最終的な状態コード (これは [HttpStatusCode.OK](/dotnet/api/system.net.httpstatuscode) になります) ではなく、予期される [HttpStatusCode.Redirect](/dotnet/api/system.net.httpstatuscode) の結果を SUT が返すステータス コードと突き合わせて確認できます。
* 最終的なログイン ページ応答 (ここでは、`Location` ヘッダーは存在しません) ではなく、応答ヘッダーの `Location` ヘッダー値が `http://localhost/Identity/Account/Login` で始まることを確認できます。

テスト アプリでは、認証と承認の側面をテストするために [ConfigureTestServices](/dotnet/api/microsoft.aspnetcore.testhost.webhostbuilderextensions.configuretestservices) の <xref:Microsoft.AspNetCore.Authentication.AuthenticationHandler`1> をモックすることができます。 最小のシナリオでは、[AuthenticateResult.Success](xref:Microsoft.AspNetCore.Authentication.AuthenticateResult.Success*) が返されます。

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet4&highlight=11-18)]

`TestAuthHandler` は、認証スキームが `Test` に設定されている場合 (この場合、`AddAuthentication` が `ConfigureTestServices` に登録されています) に、ユーザーを認証するために呼び出されます。

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet3&highlight=7-12)]

`WebApplicationFactoryClientOptions` の詳細については、「[クライアントのオプション](#client-options)」セクションを参照してください。

## <a name="set-the-environment"></a>環境を設定する

既定では、SUT のホストとアプリ環境は、開発環境を使用するように構成されています。 SUT の環境をオーバーライドするには、次のようにします。

* `ASPNETCORE_ENVIRONMENT` 環境変数 (たとえば、`Staging`、`Production`、または `Testing` などのカスタム値) を設定します。
* テスト アプリで `CreateWebHostBuilder` をオーバーライドして、`ASPNETCORE_ENVIRONMENT` 環境変数を読み取ります。

```csharp
public class CustomWebApplicationFactory<TStartup> 
    : WebApplicationFactory<TStartup> where TStartup: class
{
    protected override IWebHostBuilder CreateWebHostBuilder()
    {
        return base.CreateWebHostBuilder()
            .UseEnvironment(
                Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT"));
    }

    ...
}
```

環境は、カスタムの <xref:Microsoft.AspNetCore.Mvc.Testing.WebApplicationFactory%601> のホスト ビルダーで直接設定することもできます。

```csharp
public class CustomWebApplicationFactory<TStartup> 
    : WebApplicationFactory<TStartup> where TStartup: class
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.UseEnvironment(
            Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT"));
    
        ...
    }

    ...
```

## <a name="how-the-test-infrastructure-infers-the-app-content-root-path"></a>テスト インフラストラクチャがアプリ コンテンツのルート パスを推測する方法

`WebApplicationFactory` コンストラクターは、`TEntryPoint` アセンブリの `System.Reflection.Assembly.FullName` と同じキーを持つ統合テストを含むアセンブリで [WebApplicationFactoryContentRootAttribute](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactorycontentrootattribute) を検索することによって、アプリの[コンテンツ ルート](xref:fundamentals/index#content-root) パスを推測します。 正しいキーを持つ属性が見つからない場合、`WebApplicationFactory` はフォールバックしてソリューション ファイル ( *.sln*) を検索し、`TEntryPoint` アセンブリ名をソリューション ディレクトリに追加します。 アプリのルート ディレクトリ (コンテンツ ルート パス) は、ビューやコンテンツのファイルを検出するために使用されます。

## <a name="disable-shadow-copying"></a>シャドウ コピーの無効化

シャドウ コピーを行うと、テストが出力ディレクトリとは異なるディレクトリで実行されます。 テストが適切に機能するように、シャドウ コピーを無効化する必要があります。 [サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/test/integration-tests/samples)では、xUnit を使用して正しい構成設定の *xunit.runner.json* ファイルを含めることにより、xUnit のシャドウ コピーを無効化しています。 詳しくは、[JSON を使用した xUnit の構成](https://xunit.github.io/docs/configuring-with-json.html)に関するページをご覧ください。

次の内容を使用して、テスト プロジェクトのルートに *xunit.runner.json* ファイルを追加します。

```json
{
  "shadowCopy": false
}
```

Visual Studio を使用する場合は、ファイルの **[出力ディレクトリにコピー]** プロパティを **[常にコピーする]** に設定します。 Visual Studio を使用しない場合は、テスト アプリのプロジェクト ファイルに `Content` ターゲットを追加します。

```xml
<ItemGroup>
  <Content Update="xunit.runner.json">
    <CopyToOutputDirectory>Always</CopyToOutputDirectory>
  </Content>
</ItemGroup>
```

## <a name="disposal-of-objects"></a>オブジェクトの破棄

`IClassFixture` 実装のテストを実行した後、xUnit が [WebApplicationFactory](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1) を破棄すると、[TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver) と [HttpClient](/dotnet/api/system.net.http.httpclient) が破棄されます。 開発者がインスタンス化したオブジェクトを破棄する必要がある場合は、`IClassFixture` の実装で破棄します。 詳細については、「[Dispose メソッドの実装](/dotnet/standard/garbage-collection/implementing-dispose)」を参照してください。

## <a name="integration-tests-sample"></a>統合テストのサンプル

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/test/integration-tests/samples)は、次の 2 つのアプリで構成されています。

| アプリ | プロジェクト ディレクトリ | 説明 |
| --- | ----------------- | ----------- |
| メッセージ アプリ (SUT) | *src/RazorPagesProject* | ユーザーは、メッセージの追加、1 つ削除、すべて削除、および分析を行うことができます。 |
| アプリをテストする | *tests/RazorPagesProject.Tests* | SUT の統合テストに使用されます。 |

テストは、[Visual Studio](https://visualstudio.microsoft.com) などの IDE に組み込まれているテスト機能を使用して実行できます。 [Visual Studio Code](https://code.visualstudio.com/) またはコマンド ラインを使用している場合は、コマンド プロンプトで *tests/RazorPagesProject.Tests* ディレクトリを開き、次のコマンドを実行します。

```dotnetcli
dotnet test
```

### <a name="message-app-sut-organization"></a>メッセージ アプリ (SUT) の構成

SUT は、次の特性を持つ Razor Pages メッセージ システムです。

* アプリのインデックス ページ (*Pages/Index.cshtml* と *Pages/Index.cshtml.cs*) には、メッセージの追加、削除、および分析 (メッセージあたりの平均単語数) を制御する UI およびページ モデル メソッドが用意されています。
* メッセージは、`Id` (キー) と `Text` (メッセージ) の 2 つのプロパティを持つ `Message` クラス (*Data/Message.cs*) によって記述されます。 `Text` プロパティは必須であり、200 文字までに制限されています。
* メッセージは [Entity Framework のメモリ内データベース](/ef/core/providers/in-memory/)&#8224; を使用して格納されます。
* アプリのデータベース コンテキスト クラスである `AppDbContext` (*Data/AppDbContext.cs*) には、データアクセス層 (DAL) が含まれています。
* アプリの起動時にデータベースが空の場合、メッセージ ストアが 3 つのメッセージで初期化されます。
* アプリには、認証されたユーザーのみがアクセスできる `/SecurePage` が含まれています。

&#8224; EF トピック「[InMemory を使用したテスト](/ef/core/miscellaneous/testing/in-memory)」では、MSTest を使用したテストでメモリ内データベースを使用する方法について説明します。 このトピックでは、[xUnit](https://xunit.github.io/) テスト フレームワークを使用します。 テストの概念とテストの実装は、異なるテスト フレームワークでも類似していますが、同一ではありません。

アプリはリポジトリ パターンを使用しておらず、[Unit of Work (UoW) パターン](https://martinfowler.com/eaaCatalog/unitOfWork.html)の有効な例ではありませんが、Razor Pages ではこれらの開発パターンがサポートされています。 詳細については、[インフラストラクチャの永続化レイヤーの設計](/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)および[コントローラー ロジックのテスト](../mvc/controllers/testing.md)に関する記事を参照してください (このサンプルはリポジトリ パターンを実装しています)。

### <a name="test-app-organization"></a>テスト アプリの構成

テスト アプリは、*tests/RazorPagesProject.Tests* ディレクトリにあるコンソール アプリです。

| テスト アプリのディレクトリ | 説明 |
| ------------------ | ----------- |
| *AuthTests* | 次のテスト メソッドを含みます。<ul><li>認証されていないユーザーがセキュリティで保護されたページにアクセスする。</li><li>モック <xref:Microsoft.AspNetCore.Authentication.AuthenticationHandler`1> で認証されたユーザーがセキュリティで保護されたページにアクセスする。</li><li>GitHub ユーザー プロファイルを取得し、プロファイルのユーザー ログインを確認する。</li></ul> |
| *BasicTests* | ルーティングおよびコンテンツ タイプのテスト メソッドを含みます。 |
| *IntegrationTests* | カスタム `WebApplicationFactory` クラスを使用したインデックス ページの統合テストを含みます。 |
| *Helpers/Utilities* | <ul><li>*Utilities.cs* には、データベースにテスト データをシードする `InitializeDbForTests` メソッドが含まれています。</li><li>*HtmlHelpers.cs* には、テスト メソッドで使用する AngleSharp `IHtmlDocument` を返すメソッドが用意されています。</li><li>*HttpClientExtensions.cs* は、SUT に要求を送信する `SendAsync` のオーバーロードを提供します。</li></ul> |

テスト フレームワークは、[xUnit](https://xunit.github.io/) です。 統合テストは、[TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver) を含む [Microsoft.AspNetCore.TestHost](/dotnet/api/microsoft.aspnetcore.testhost) を使用して実行されます。 [Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing) パッケージはテスト ホストとテスト サーバーを構成するために使用されるため、テスト アプリのプロジェクト ファイルまたはテスト アプリの開発者構成で直接 `TestHost` および `TestServer` パッケージを参照する必要はありません。

**テスト用のデータベースのシード処理**

統合テストでは、通常、テストを実行する前に、データベース内に小さなデータセットが必要です。 たとえば、削除テストでは、データベース レコードの削除を呼び出します。そのため、削除要求を成功させるには、データベースに少なくとも 1 つのレコードが必要です。

このサンプル アプリでは、*Utilities.cs* で 3 つのメッセージを使用してデータベースをシードします。このメッセージは、テストを実行する際に使用することができます。

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/Helpers/Utilities.cs?name=snippet1)]

::: moniker-end

## <a name="additional-resources"></a>その他の技術情報

* [単体テスト](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)
* <xref:test/razor-pages-tests>
* <xref:fundamentals/middleware/index>
* <xref:mvc/controllers/testing>