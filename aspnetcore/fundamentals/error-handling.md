---
title: ASP.NET Core のエラーを処理する
author: rick-anderson
description: ASP.NET Core アプリでエラーを処理する方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/05/2019
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
uid: fundamentals/error-handling
ms.openlocfilehash: 1687195bc5d80d50289db51a5e8ac6113ca8b05f
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586125"
---
# <a name="handle-errors-in-aspnet-core"></a>ASP.NET Core のエラーを処理する

::: moniker range=">= aspnetcore-5.0"

作成者: [Kirk Larkin](https://twitter.com/serpent5)、[Tom Dykstra](https://github.com/tdykstra/)、[Steve Smith](https://ardalis.com/)

この記事では、ASP.NET Core Web アプリでエラーを処理するための一般的な手法について取り上げます。 Web API については、<xref:web-api/handle-errors> を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples)します。 ([ダウンロード方法](xref:index#how-to-download-a-sample)。)F12 ブラウザー開発者ツールの [ネットワーク] タブは、サンプル アプリをテストする場合に便利です。

## <a name="developer-exception-page"></a>開発者例外ページ

"*開発者例外ページ*" には、要求の例外に関する詳細な情報が表示されます。 ASP.NET Core テンプレートにより、次のコードが生成されます。

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Startup.cs?name=snippet&highlight=3-6)]

前の強調表示されたコードは、アプリが[開発環境](xref:fundamentals/environments)で実行されている場合に、開発者例外ページを有効にします。

このテンプレートでは、後続のミドルウェアでスローされた例外をキャッチできるように、ミドルウェア パイプラインの早い段階で <xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage%2A> が配置されます。

前のコードは、アプリが開発環境で実行されている場合に ***のみ***、開発者例外ページを有効にします。 アプリが運用環境で実行されている場合は、詳細な例外情報を公開しないようにしてください。 環境の構成について詳しくは、「<xref:fundamentals/environments>」をご覧ください。

開発者例外ページには、例外と要求に関する次の情報が含まれています。

* スタック トレース
* クエリ文字列のパラメーター (ある場合)
* Cookie (ある場合)
* ヘッダー

## <a name="exception-handler-page"></a>例外ハンドラー ページ

[運用環境](xref:fundamentals/environments)のカスタム エラー処理ページを構成するには、<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> を呼び出します。 この例外処理ミドルウェアは、次のことを行います。

* 例外をキャッチしてログに記録します。
* 指定されたパスを使用して、別のパイプラインで要求を再実行します。 応答が始まっていた場合、要求は再実行されません。 テンプレートによって生成されたコードは、`/Error` パスを使用して要求を再実行します。

次の例では、<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> により非開発環境に例外処理ミドルウェアが追加されます。

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_DevPageAndHandlerPage&highlight=5-9)]

Razor Pages アプリのテンプレートには、エラー ページ ( *.cshtml*) と <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> クラス (`ErrorModel`) が *Pages* フォルダー内に用意されています。 MVC アプリの場合、プロジェクト テンプレートには、Home コントローラーの `Error` アクション メソッドとエラー ビューが含まれています。

例外処理ミドルウェアでは、"*元の*" HTTP メソッドを使用して要求が再実行されます。 エラー ハンドラーのエンドポイントが特定の HTTP メソッドのセットに制限されている場合は、それらの HTTP メソッドに対してのみ実行されます。 たとえば、`[HttpGet]` 属性を使用する MVC コントローラーのアクションは、GET 要求に対してのみ実行されます。 "*すべての*" 要求がカスタム エラー処理ページに到達するようにするために、それらを特定の HTTP メソッドのセットに制限しないでください。

元の HTTP メソッドに応じて例外を異なる方法で処理するには:

* Razor Pages の場合は、複数のハンドラー メソッドを作成します。 たとえば、GET 例外を処理するために `OnGet` を使用し、POST 例外を処理するために `OnPost` を使用します。
* MVC の場合は、複数のアクションに HTTP 動詞属性を適用します。 たとえば、GET 例外を処理するために `[HttpGet]` を使用し、POST 例外を処理するために `[HttpPost]` を使用します。

認証されていないユーザーがカスタム エラー処理ページを表示できるようにするには、匿名アクセスがサポートされるようにします。

### <a name="access-the-exception"></a>例外にアクセスする

エラー ハンドラーで例外や元の要求パスにアクセスするには、<xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> を使います。 次のコードは、ASP.NET Core テンプレートによって生成される既定の *Pages/Error.cshtml.cs* に `ExceptionMessage` を追加します。

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Pages/Error.cshtml.cs?name=snippet)]

> [!WARNING]
> 機密性の高いエラー情報をクライアントに提供 **しないでください**。 エラーの提供はセキュリティ上のリスクです。

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x)で例外をテストするには:

* 環境を運用環境に設定します。
* *Program.cs* の `webBuilder.UseStartup<Startup>();` からコメントを削除します。
* ホーム ページで **[Trigger an exception]\(例外をトリガーする\)** を選択します。

## <a name="exception-handler-lambda"></a>例外ハンドラー ラムダ

[カスタム例外ハンドラー ページ](#exception-handler-page)の代わりになるのは、<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> にラムダを提供することです。 ラムダを使うと、応答を返す前にエラーにアクセスできます。

次のコードは、例外処理にラムダを使用しています。

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupLambda.cs?name=snippet)]

<!-- 
In the preceding code, `await context.Response.WriteAsync(new string(' ', 512));` is added so the Internet Explorer browser displays the error message rather than an IE error message. For more information, see [this GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/16144).
-->

> [!WARNING]
> <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerFeature> または <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> からの機密性の高いエラー情報をクライアントに提供 **しないでください**。 エラーの提供はセキュリティ上のリスクです。

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x)で例外処理ラムダをテストするには:

* 環境を運用環境に設定します。
* *Program.cs* の `webBuilder.UseStartup<StartupLambda>();` からコメントを削除します。
* ホーム ページで **[Trigger an exception]\(例外をトリガーする\)** を選択します。

## <a name="usestatuscodepages"></a>UseStatusCodePages

ASP.NET Core アプリでは、既定で、"*404 - 見つかりません*" などの HTTP エラー状態コードの状態コード ページが表示されません。 アプリで、本文のない HTTP 400 から 599 のエラー状態コードが発生すると、状態コードと空の応答本文が返されます。 状態コード ページを提供するには、状態コード ページ ミドルウェアを使用します。 一般的なエラー状態コード用に既定のテキスト専用ハンドラーを有効にするには、`Startup.Configure` メソッドで <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> を呼び出します。

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupUseStatusCodePages.cs?name=snippet&highlight=13)]

要求処理ミドルウェアの前に `UseStatusCodePages` を呼び出します。 たとえば、静的ファイル ミドルウェアとエンドポイント ミドルウェアの前に `UseStatusCodePages` を呼び出します。

`UseStatusCodePages` を使用しない場合、エンドポイントなしで URL に移動すると、エンドポイントが見つからないことを示すブラウザー依存のエラー メッセージが返されます。 たとえば、`Home/Privacy2` に移動します。 `UseStatusCodePages` が呼び出されると、ブラウザーにより次が返されます。

```html
Status Code: 404; Not Found
```

`UseStatusCodePages` は、ユーザーにとって役に立たないメッセージを返すため、通常は、運用環境では使用されません。

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x)で `UseStatusCodePages` をテストするには:

* 環境を運用環境に設定します。
* *Program.cs* の `webBuilder.UseStartup<StartupUseStatusCodePages>();` からコメントを削除します。
* ホーム ページにあるリンクを選択します。

> [!NOTE]
> 状態コード ページのミドルウェアは例外をキャッチ **しません**。 カスタム エラー処理ページを提供するには、[例外ハンドラー ページ](#exception-handler-page)を使用します。

### <a name="usestatuscodepages-with-format-string"></a>書式設定文字列での UseStatusCodePages

応答の内容の種類とテキストをカスタマイズするには、内容の種類と書式文字列を受け取る <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> のオーバーロードを使います。

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupFormat.cs?name=snippet&highlight=13-14)]

上記のコードでは、`{0}` はエラー コードのプレースホルダーです。

書式指定文字列を含む `UseStatusCodePages` は、ユーザーにとって役に立たないメッセージを返すため、通常は、運用環境では使用されません。

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x)で `UseStatusCodePages` をテストするには、*Program.cs* の `webBuilder.UseStartup<StartupFormat>();` からコメントを削除します。

### <a name="usestatuscodepages-with-lambda"></a>ラムダでの UseStatusCodePages

カスタム エラー処理と応答書き込みコードを指定するには、ラムダ式を受け取る <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> のオーバーロードを使います。

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupStatusLambda.cs?name=snippet&highlight=13-20)]

ラムダを含む `UseStatusCodePages` は、ユーザーにとって役に立たないメッセージを返すため、通常は、運用環境では使用されません。

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x)で `UseStatusCodePages` をテストするには、*Program.cs* の `webBuilder.UseStartup<StartupStatusLambda>();` からコメントを削除します。

### <a name="usestatuscodepageswithredirects"></a>UseStatusCodePagesWithRedirects

<xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithRedirects%2A> 拡張メソッド:

* クライアントに [302 - Found](https://developer.mozilla.org/docs/Web/HTTP/Status/302) 状態コードを送信します。
* URL テンプレートで指定されているエラー処理エンドポイントにクライアントをリダイレクトします。 エラー処理エンドポイントには、通常、エラー情報が表示され、HTTP 200 が返されます。

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupSCredirect.cs?name=snippet&highlight=13)]

前のコードに示されているように、URL テンプレートには状態コード用の `{0}` プレースホルダーを含めることができます。 URL テンプレートが `~` (チルダ) で始まっている場合、`~` はアプリの `PathBase` に置き換えられます。 アプリでエンドポイントを指定するときに、そのエンドポイントの MVC ビューまたは Razor ページを作成します。 Razor Pages の例については、[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x)にある [Pages/MyStatusCode.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x/ErrorHandlingSample/Pages) をご覧ください。

この方法は、次のようなアプリで一般的に使用されます。

* クライアントを別のエンドポイントにリダイレクトする必要がある場合 (通常は、別のアプリがエラーを処理する場合)。 Web アプリの場合は、クライアントのブラウザーのアドレス バーにリダイレクトされたエンドポイントが反映されます。
* 元の状態コードを保持し、初回のリダイレクト応答で返してはいけない場合。

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x)で `UseStatusCodePages` をテストするには、*Program.cs* の `webBuilder.UseStartup<StartupSCredirect>();` からコメントを削除します。

### <a name="usestatuscodepageswithreexecute"></a>UseStatusCodePagesWithReExecute

<xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithReExecute%2A> 拡張メソッド:

* 元の状態コードをクライアントに返します。
* 代替パスを使用して要求パイプラインを再実行することで、応答本文を生成します。

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupSCreX.cs?name=snippet&highlight=13)]

アプリ内でエンドポイントが指定されている場合は、そのエンドポイントの MVC ビューまたは Razor ページを作成します。 `UseRouting` の前に `UseStatusCodePagesWithReExecute` が配置されていることを確認して、要求を状態ページに再ルーティングできるようにします。 Razor Pages の例については、[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x)にある [Pages/MyStatusCode2.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x/ErrorHandlingSample/Pages) をご覧ください。

この方法は、次のようなアプリで一般的に使用されます。

* 別のエンドポイントにリダイレクトすることなく要求を処理する。 Web アプリの場合は、クライアントのブラウザーのアドレス バーに、初めに要求されていたエンドポイントが反映されます。
* 元の状態コードを保持し、応答で返す。

URL とクエリ文字列のテンプレートには、状態コード用のプレースホルダー `{0}` を含めることができます。 URL テンプレートの先頭には、`/` を付ける必要があります。

<!-- Review: removing this. The sample code doesn't use @page "{code?}"
If you want that, it should be @page "{code:int?}"
but that's not required. Original text follows:

When using a placeholder in the path, confirm that the endpoint can process the path segment. For example, a Razor Page for errors should accept the optional path segment value with the `@page` directive:

```cshtml
@page "{code?}"
```
-->

次の例で示すように、エラーを処理するエンドポイントでは、エラーを生成した元の URL を取得できます。

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Pages/MyStatusCode2.cshtml.cs?name=snippet)]

Razor Pages の例については、[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x)にある [Pages/MyStatusCode2.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x/ErrorHandlingSample/Pages) をご覧ください。

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x)で `UseStatusCodePages` をテストするには、*Program.cs* の `webBuilder.UseStartup<StartupSCreX>();` からコメントを削除します。

## <a name="disable-status-code-pages"></a>状態コード ページを無効にする

MVC コントローラーまたはアクション メソッドの状態コード ページを無効にするには、[[SkipStatusCodePages]](xref:Microsoft.AspNetCore.Mvc.SkipStatusCodePagesAttribute) 属性を使用します。

Razor Pages ハンドラー メソッドまたは MVC コントローラーの特定の要求に対して状態コード ページを無効にするには、<xref:Microsoft.AspNetCore.Diagnostics.IStatusCodePagesFeature> を使用します。

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Pages/Privacy.cshtml.cs?name=snippet)]

## <a name="exception-handling-code"></a>例外処理コード

例外処理ページのコードが例外をスローすることもあります。 運用環境のエラー ページは十分にテストし、それ自体から例外がスローされないように特に注意する必要があります。
<!-- Review: original, which is not realistic 
 > It's often a good idea for production error pages to consist of purely static content.

 comments: - after you catch the exception, you need code to log the details and perhaps dynamically create a string with an error message. 
-->

### <a name="response-headers"></a>応答ヘッダー

応答のヘッダーが送信された後は、次のようになります。

* アプリで応答の状態コードを変更できません。
* すべての例外ページやハンドラーを実行できません。 応答は完了している必要があります。あるいは、接続が中止となっている必要があります。

## <a name="server-exception-handling"></a>サーバー例外処理

アプリ内の例外処理ロジックに加えて、[HTTP サーバーの実装](xref:fundamentals/servers/index)でも一部の例外を処理できます。 応答ヘッダーの送信前にサーバーで例外がキャッチされると、サーバーによって "`500 - Internal Server Error`" 応答が応答本文なしで送信されます。 応答ヘッダーの送信後にサーバーで例外がキャッチされた場合、サーバーは接続を閉じます。 アプリで処理されない要求はサーバーで処理されます。 サーバーが要求を処理しているときに発生した例外は、すべてサーバーの例外処理によって処理されます。 この動作は、アプリのカスタム エラー ページ、例外処理ミドルウェア、およびフィルターから影響を受けません。

## <a name="startup-exception-handling"></a>起動時の例外処理

アプリの起動中に起こる例外はホスティング層だけが処理できます。 [起動時のエラーをキャプチャ](xref:fundamentals/host/web-host#capture-startup-errors)したり、[詳細なエラーをキャプチャ](xref:fundamentals/host/web-host#detailed-errors)したりするように、ホストを構成することができます。

ホスティング レイヤーでは、ホスト アドレス/ポート バインド後にエラーが発生した場合にのみ、キャプチャされた起動時エラーに対するエラー ページを表示できます。 バインドが失敗した場合は、次のようになります。

* ホスティング レイヤーにより重大な例外がログに記録されます。
* dotnet プロセスがクラッシュします。
* HTTP サーバーが [Kestrel](xref:fundamentals/servers/kestrel) のときは、エラー ページは表示されません。

[IIS](/iis) (または Azure App Service) または [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) 上で実行している場合、プロセスを開始できなければ、[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)から "*502.5 - 処理エラー*" が返されます。 詳細については、「<xref:test/troubleshoot-azure-iis>」を参照してください。

## <a name="database-error-page"></a>データベース エラー ページ

データベース開発者ページ例外フィルター `AddDatabaseDeveloperPageExceptionFilter` では、Entity Framework Core の移行を使って解決できるデータベース関連の例外がキャプチャされます。 これらの例外が発生すると、問題が解決する可能性のあるアクションの詳細を含む HTML 応答が生成されます。 このページは、開発環境でのみ有効です。 次のコードは、個々のユーザー アカウントが指定されたときに、ASP.NET Core Razor Pages テンプレートによって生成されたものです。

[!code-csharp[](error-handling/samples/5.x/StartupDBexFilter.cs?name=snippet&highlight=6)]

## <a name="exception-filters"></a>例外フィルター

MVC アプリでは、例外フィルターをグローバルに、またはコントローラーやアクションの単位で構成できます。 Razor Pages アプリでは、グローバルに、またはページ モデルの単位で構成できます。 このようなフィルターは、コントローラー アクションや別のフィルターの実行中に発生する未処理の例外を処理します。 詳細については、「<xref:mvc/controllers/filters#exception-filters>」を参照してください。

例外フィルターは、MVC アクション内で発生する例外をトラップする場合には便利ですが、組み込みの[例外処理ミドルウェア](https://github.com/dotnet/aspnetcore/blob/main/src/Middleware/Diagnostics/src/ExceptionHandler/ExceptionHandlerMiddleware.cs) `UseExceptionHandler` ほど柔軟ではありません。 選択された MVC アクションに応じて異なる方法でエラー処理を実行する必要がある場合を除き、`UseExceptionHandler` を使用することをお勧めします。

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Startup.cs?name=snippet&highlight=9)]

## <a name="model-state-errors"></a>モデル状態エラー

モデル状態エラーを処理する方法については、[モデル バインド](xref:mvc/models/model-binding)および[モデルの検証](xref:mvc/models/validation)に関する記事をご覧ください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:test/troubleshoot-azure-iis>
* <xref:host-and-deploy/azure-iis-errors-reference>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

作成者: [Tom Dykstra](https://github.com/tdykstra/)、[Steve Smith](https://ardalis.com/)

この記事では、ASP.NET Core Web アプリでエラーを処理するための一般的な手法について取り上げます。 Web API については、<xref:web-api/handle-errors> を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples)します。 ([ダウンロード方法](xref:index#how-to-download-a-sample)。)

## <a name="developer-exception-page"></a>開発者例外ページ

"*開発者例外ページ*" には、要求の例外に関する詳細な情報が表示されます。 ASP.NET Core テンプレートにより、次のコードが生成されます。

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_DevPageAndHandlerPage&highlight=1-4)]

前のコードは、アプリが[開発環境](xref:fundamentals/environments)で実行されている場合に、開発者例外ページを有効にします。

このテンプレートでは、後続のミドルウェアで例外をキャッチできるように、ミドルウェアの前に <xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage%2A> が配置されます。

前のコードは、アプリが **開発環境** で実行されている場合にのみ、開発者例外ページを有効にします。 アプリが運用環境で実行されている場合は、詳細な例外情報を公開しないようにしてください。 環境の構成について詳しくは、「<xref:fundamentals/environments>」をご覧ください。

開発者例外ページには、例外と要求に関する次の情報が含まれています。

* スタック トレース
* クエリ文字列のパラメーター (ある場合)
* Cookie (ある場合)
* ヘッダー

## <a name="exception-handler-page"></a>例外ハンドラー ページ

運用環境用のカスタム エラー処理ページを構成するには、例外処理ミドルウェアを使用します。 ミドルウェアでは次を行います。

* 例外をキャッチしてログに記録します。
* ページ用の、またはコントローラーが指定した別のパイプラインで、要求を再実行します。 応答が始まっていた場合、要求は再実行されません。 テンプレートによって生成されたコードは、`/Error` への要求を再実行します。

次の例では、<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> により非開発環境に例外処理ミドルウェアが追加されます。

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_DevPageAndHandlerPage&highlight=5-9)]

Razor Pages アプリのテンプレートには、エラー ページ ( *.cshtml*) と <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> クラス (`ErrorModel`) が *Pages* フォルダー内に用意されています。 MVC アプリの場合、プロジェクト テンプレートには、Home コントローラーのエラー アクション メソッドとエラー ビューが含まれています。

`HttpGet` などの HTTP メソッド属性を使ってエラー ハンドラー アクション メソッドをマークしないでください。 明示的な動詞を使用すると、要求がメソッドに届かないことがあります。 認証されていないユーザーにエラー ビューが表示される場合は、メソッドへの匿名アクセスを許可します。

### <a name="access-the-exception"></a>例外にアクセスする

エラー ハンドラー コントローラーまたはページ内で例外や元の要求パスにアクセスするには、<xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> を使います。

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Pages/MyFolder/Error.cshtml.cs?name=snippet_ExceptionHandlerPathFeature&3,7)]

> [!WARNING]
> 機密性の高いエラー情報をクライアントに提供 **しないでください**。 エラーの提供はセキュリティ上のリスクです。

上記の例外処理ページをトリガーするには、環境を運用に設定し、例外を強制します。

## <a name="exception-handler-lambda"></a>例外ハンドラー ラムダ

[カスタム例外ハンドラー ページ](#exception-handler-page)の代わりになるのは、<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> にラムダを提供することです。 ラムダを使うと、応答を返す前にエラーにアクセスできます。

例外処理にラムダを使う例を次に示します。

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_HandlerPageLambda)]

上記のコードでは `await context.Response.WriteAsync(new string(' ', 512));` が追加されるため、Internet Explorer ブラウザーには IE のエラー メッセージではなく、このエラー メッセージが表示されます。 詳細については、次を参照してください。[この GitHub の問題](https://github.com/dotnet/AspNetCore.Docs/issues/16144)します。

> [!WARNING]
> <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerFeature> または <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> からの機密性の高いエラー情報をクライアントに提供 **しないでください**。 エラーの提供はセキュリティ上のリスクです。

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples)で例外処理ラムダの結果を表示するには、`ProdEnvironment` および `ErrorHandlerLambda` プリプロセッサ ディレクティブを使用して、ホーム ページで **[Trigger an exception]\(例外をトリガーする\)** を選択します。

## <a name="usestatuscodepages"></a>UseStatusCodePages

ASP.NET Core アプリでは、既定で、"*404 - 見つかりません*" などの HTTP 状態コードの状態コード ページが表示されません。 アプリでは、状態コードと空の応答本文が返されます。 状態コード ページを提供するには、状態コード ページ ミドルウェアを使用します。

そのミドルウェアは、[Microsoft.AspNetCore.Diagnostics](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics/) パッケージによって使用可能になります。

一般的なエラー状態コード用に既定のテキスト専用ハンドラーを有効にするには、`Startup.Configure` メソッドで <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> を呼び出します。

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePages)]

要求処理ミドルウェア (たとえば、静的ファイル ミドルウェアや MVC ミドルウェア) の前に `UseStatusCodePages` を呼び出します。

`UseStatusCodePages` を使用しない場合、エンドポイントなしで URL に移動すると、エンドポイントが見つからないことを示すブラウザー依存のエラー メッセージが返されます。 たとえば、`Home/Privacy2` に移動します。 `UseStatusCodePages` が呼び出されると、ブラウザーにより次が返されます。

```
Status Code: 404; Not Found
```

## <a name="usestatuscodepages-with-format-string"></a>書式設定文字列での UseStatusCodePages

応答の内容の種類とテキストをカスタマイズするには、内容の種類と書式文字列を受け取る <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> のオーバーロードを使います。

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePagesFormatString)]

## <a name="usestatuscodepages-with-lambda"></a>ラムダでの UseStatusCodePages

カスタム エラー処理と応答書き込みコードを指定するには、ラムダ式を受け取る <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> のオーバーロードを使います。

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePagesLambda)]

## <a name="usestatuscodepageswithredirects"></a>UseStatusCodePagesWithRedirects

<xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithRedirects%2A> 拡張メソッド:

* クライアントに *302 - Found* 状態コードを送信します。
* URL テンプレートで指定された場所にクライアントをリダイレクトします。

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePagesWithRedirect)]

次の例で示すように、URL テンプレートには状態コード用の `{0}` プレースホルダーを含めることができます。 URL テンプレートが `~` (チルダ) で始まっている場合、`~` はアプリの `PathBase` に置き換えられます。 アプリ内でエンドポイントを指し示す場合は、そのエンドポイントの MVC ビューまたは Razor ページを作成します。 Razor Pages の例については、[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples)にある *Pages/StatusCode.cshtml* をご覧ください。

この方法は、次のようなアプリで一般的に使用されます。

* クライアントを別のエンドポイントにリダイレクトする必要がある場合 (通常は、別のアプリがエラーを処理する場合)。 Web アプリの場合は、クライアントのブラウザーのアドレス バーにリダイレクトされたエンドポイントが反映されます。
* 元の状態コードを保持し、初回のリダイレクト応答で返してはいけない場合。

## <a name="usestatuscodepageswithreexecute"></a>UseStatusCodePagesWithReExecute

<xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithReExecute%2A> 拡張メソッド:

* 元の状態コードをクライアントに返します。
* 代替パスを使用して要求パイプラインを再実行することで、応答本文を生成します。

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePagesWithReExecute)]

アプリ内でエンドポイントを指し示す場合は、そのエンドポイントの MVC ビューまたは Razor ページを作成します。 `UseRouting` の前に `UseStatusCodePagesWithReExecute` が配置されていることを確認して、要求を状態ページに再ルーティングできるようにします。 Razor Pages の例については、[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples)にある *Pages/StatusCode.cshtml* をご覧ください。

この方法は、次のようなアプリで一般的に使用されます。

* 別のエンドポイントにリダイレクトすることなく要求を処理する。 Web アプリの場合は、クライアントのブラウザーのアドレス バーに、初めに要求されていたエンドポイントが反映されます。
* 元の状態コードを保持し、応答で返す。

URL とクエリ文字列のテンプレートには、状態コード用のプレースホルダー (`{0}`) を含めることができます。 URL テンプレートの先頭には、スラッシュ (`/`) を付ける必要があります。 パスでプレースホルダーを使う場合は、エンドポイント (ページまたはコントローラー) でパスのセグメントを処理できることを確認します。 たとえば、エラー用の Razor ページでは、`@page` ディレクティブの付いた省略可能なパスのセグメント値を受け入れる必要があります。

```cshtml
@page "{code?}"
```

次の例で示すように、エラーを処理するエンドポイントでは、エラーを生成した元の URL を取得できます。

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Pages/StatusCode.cshtml.cs?name=snippet_StatusCodeReExecute)]

`HttpGet` などの HTTP メソッド属性を使ってエラー ハンドラー アクション メソッドをマークしないでください。 明示的な動詞を使用すると、要求がメソッドに届かないことがあります。 認証されていないユーザーにエラー ビューが表示される場合は、メソッドへの匿名アクセスを許可します。

## <a name="disable-status-code-pages"></a>状態コード ページを無効にする

MVC コントローラーまたはアクション メソッドの状態コード ページを無効にするには、[`[SkipStatusCodePages]`](xref:Microsoft.AspNetCore.Mvc.SkipStatusCodePagesAttribute) 属性を使用します。

Razor Pages ハンドラー メソッドまたは MVC コントローラーの特定の要求に対して状態コード ページを無効にするには、<xref:Microsoft.AspNetCore.Diagnostics.IStatusCodePagesFeature> を使用します。

```csharp
var statusCodePagesFeature = HttpContext.Features.Get<IStatusCodePagesFeature>();

if (statusCodePagesFeature != null)
{
    statusCodePagesFeature.Enabled = false;
}
```

## <a name="exception-handling-code"></a>例外処理コード

例外処理ページのコードは例外をスローすることがあります。 実稼働のエラー ページは純粋に静的なコンテンツで構成することをお勧めします。

### <a name="response-headers"></a>応答ヘッダー

応答のヘッダーが送信された後は、次のようになります。

* アプリで応答の状態コードを変更できません。
* すべての例外ページやハンドラーを実行できません。 応答は完了している必要があります。あるいは、接続が中止となっている必要があります。

## <a name="server-exception-handling"></a>サーバー例外処理

アプリ内の例外処理ロジックに加えて、[HTTP サーバーの実装](xref:fundamentals/servers/index)でも一部の例外を処理できます。 応答ヘッダーの送信前にサーバーで例外がキャッチされると、サーバーによって "*500 - 内部サーバー エラーです*" 応答が応答本文なしで送信されます。 応答ヘッダーの送信後にサーバーで例外がキャッチされた場合、サーバーは接続を閉じます。 アプリで処理されない要求はサーバーで処理されます。 サーバーが要求を処理しているときに発生した例外は、すべてサーバーの例外処理によって処理されます。 この動作は、アプリのカスタム エラー ページ、例外処理ミドルウェア、およびフィルターから影響を受けません。

## <a name="startup-exception-handling"></a>起動時の例外処理

アプリの起動中に起こる例外はホスティング層だけが処理できます。 [起動時のエラーをキャプチャ](xref:fundamentals/host/web-host#capture-startup-errors)したり、[詳細なエラーをキャプチャ](xref:fundamentals/host/web-host#detailed-errors)したりするように、ホストを構成することができます。

ホスティング レイヤーでは、ホスト アドレス/ポート バインド後にエラーが発生した場合にのみ、キャプチャされた起動時エラーに対するエラー ページを表示できます。 バインドが失敗した場合は、次のようになります。

* ホスティング レイヤーにより重大な例外がログに記録されます。
* dotnet プロセスがクラッシュします。
* HTTP サーバーが [Kestrel](xref:fundamentals/servers/kestrel) のときは、エラー ページは表示されません。

[IIS](/iis) (または Azure App Service) または [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) 上で実行している場合、プロセスを開始できなければ、[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)から "*502.5 - 処理エラー*" が返されます。 詳細については、「<xref:test/troubleshoot-azure-iis>」を参照してください。

## <a name="database-error-page"></a>データベース エラー ページ

データベース エラー ページ ミドルウェアでは、Entity Framework の移行を使って解決できるデータベース関連の例外がキャプチャされます。 これらの例外が発生すると、問題が解決する可能性のあるアクションの詳細を含む HTML 応答が生成されます。 このページは、開発環境でのみ有効にする必要があります。 ページを有効にするには、コードを `Startup.Configure` に追加します。

```csharp
if (env.IsDevelopment())
{
    app.UseDatabaseErrorPage();
}
```

<xref:Microsoft.AspNetCore.Builder.DatabaseErrorPageExtensions.UseDatabaseErrorPage%2A> には、[Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore/) NuGet パッケージが必要です。

<!-- FUTURE UPDATE: On the next topic overhaul/release update, add API crosslink to this section for xref:Microsoft.AspNetCore.Builder.DatabaseErrorPageExtensions.UseDatabaseErrorPage* when available via the API docs. -->

## <a name="exception-filters"></a>例外フィルター

MVC アプリでは、例外フィルターをグローバルに、またはコントローラーやアクションの単位で構成できます。 Razor Pages アプリでは、グローバルに、またはページ モデルの単位で構成できます。 このようなフィルターはコントローラー アクションや別のフィルターの実行中に発生する未処理の例外を処理します。 詳細については、「<xref:mvc/controllers/filters#exception-filters>」を参照してください。

> [!TIP]
> 例外フィルターは、MVC アクション内で発生した例外をトラップする場合は便利ですが、例外処理ミドルウェアほど柔軟ではありません。 ミドルウェアの使用をお勧めします。 選択された MVC アクションに応じて異なる方法でエラー処理を実行する必要がある場合にのみ、フィルターを使用します。

## <a name="model-state-errors"></a>モデル状態エラー

モデル状態エラーを処理する方法については、[モデル バインド](xref:mvc/models/model-binding)および[モデルの検証](xref:mvc/models/validation)に関する記事をご覧ください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:test/troubleshoot-azure-iis>
* <xref:host-and-deploy/azure-iis-errors-reference>

::: moniker-end
