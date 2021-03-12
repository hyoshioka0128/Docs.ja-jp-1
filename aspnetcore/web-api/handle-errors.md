---
title: ASP.NET Core Web API のエラーを処理する
author: pranavkm
description: ASP.NET Core Web API を使用したエラー処理について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: prkrishn
ms.custom: mvc
ms.date: 1/11/2021
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
uid: web-api/handle-errors
ms.openlocfilehash: f9cc38f444bd3db826595b7de64db05792915a23
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102585722"
---
# <a name="handle-errors-in-aspnet-core-web-apis"></a>ASP.NET Core Web API のエラーを処理する

この記事では、ASP.NET Core Web API を使用したエラーの処理方法とエラー処理のカスタマイズ方法について説明します。

[サンプルコードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/handle-errors/samples)[する (ダウンロードする方法](xref:index#how-to-download-a-sample))

## <a name="developer-exception-page"></a>開発者例外ページ

[開発者例外ページ](xref:fundamentals/error-handling)は、サーバー エラーの詳しいスタック トレースを取得するために役立つツールです。 <xref:Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware> を使用して、HTTP パイプラインから同期および非同期例外をキャプチャし、エラー応答を生成します。 これを説明するために、次のコントローラー アクションがあるとします。

[!code-csharp[](handle-errors/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_GetByCity)]

次の `curl` コマンドを実行して、上記のアクションをテストします。

```bash
curl -i https://localhost:5001/weatherforecast/chicago
```

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core 3.0 以降では、クライアントにより HTML 形式の出力が要求されない場合、開発者例外ページにはテキスト形式の応答が表示されます。 次のような出力が表示されます。

```console
HTTP/1.1 500 Internal Server Error
Transfer-Encoding: chunked
Content-Type: text/plain
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
Date: Fri, 27 Sep 2019 16:13:16 GMT

System.ArgumentException: We don't offer a weather forecast for chicago. (Parameter 'city')
   at WebApiSample.Controllers.WeatherForecastController.Get(String city) in C:\working_folder\aspnet\AspNetCore.Docs\aspnetcore\web-api\handle-errors\samples\3.x\Controllers\WeatherForecastController.cs:line 34
   at lambda_method(Closure , Object , Object[] )
   at Microsoft.Extensions.Internal.ObjectMethodExecutor.Execute(Object target, Object[] parameters)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ActionMethodExecutor.SyncObjectResultExecutor.Execute(IActionResultTypeMapper mapper, ObjectMethodExecutor executor, Object controller, Object[] arguments)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.<InvokeActionMethodAsync>g__Logged|12_1(ControllerActionInvoker invoker)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.<InvokeNextActionFilterAsync>g__Awaited|10_0(ControllerActionInvoker invoker, Task lastTask, State next, Scope scope, Object state, Boolean isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.Rethrow(ActionExecutedContextSealed context)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.Next(State& next, Scope& scope, Object& state, Boolean& isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.InvokeInnerFilterAsync()
--- End of stack trace from previous location where exception was thrown ---
   at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeFilterPipelineAsync>g__Awaited|19_0(ResourceInvoker invoker, Task lastTask, State next, Scope scope, Object state, Boolean isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeAsync>g__Logged|17_1(ResourceInvoker invoker)
   at Microsoft.AspNetCore.Routing.EndpointMiddleware.<Invoke>g__AwaitRequestTask|6_0(Endpoint endpoint, Task requestTask, ILogger logger)
   at Microsoft.AspNetCore.Authorization.AuthorizationMiddleware.Invoke(HttpContext context)
   at Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware.Invoke(HttpContext context)

HEADERS
=======
Accept: */*
Host: localhost:44312
User-Agent: curl/7.55.1
```

代わりに HTML 形式の応答を表示するには、`Accept` HTTP 要求ヘッダーを `text/html` のメディアの種類に設定します。 次に例を示します。

```bash
curl -i -H "Accept: text/html" https://localhost:5001/weatherforecast/chicago
```

HTTP 応答からの次の抜粋を考えてみます。

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

ASP.NET Core 2.2 以前では、開発者例外ページには HTML 形式の応答が表示されます。 たとえば、HTTP 応答からの次の抜粋を考えてみます。

::: moniker-end

```console
HTTP/1.1 500 Internal Server Error
Transfer-Encoding: chunked
Content-Type: text/html; charset=utf-8
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
Date: Fri, 27 Sep 2019 16:55:37 GMT

<!DOCTYPE html>
<html lang="en" xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <meta charset="utf-8" />
        <title>Internal Server Error</title>
        <style>
            body {
    font-family: 'Segoe UI', Tahoma, Arial, Helvetica, sans-serif;
    font-size: .813em;
    color: #222;
    background-color: #fff;
}
```

::: moniker range=">= aspnetcore-3.0"

HTML 形式の応答は、Postman などのツールを使用してテストするときに役立ちます。 次の画面キャプチャは、Postman のテキスト形式と HTML 形式の両方の応答を示しています。

![Postman での開発者例外ページのテスト](handle-errors/_static/developer-exception-page-postman.gif)

::: moniker-end

> [!WARNING]
> **アプリを開発環境で実行するときにのみ**、開発者例外ページを有効にします。 アプリを運用環境で実行するときに、詳細な例外情報を公開しないでください。 環境の構成について詳しくは、「<xref:fundamentals/environments>」をご覧ください。
>
> `HttpGet` などの HTTP メソッド属性を使ってエラー ハンドラー アクション メソッドをマークしないでください。 明示的な動詞を使用すると、要求がアクション メソッドに届かないことがあります。 認証されていないユーザーにエラーが表示される場合は、メソッドへの匿名アクセスを許可します。

## <a name="exception-handler"></a>例外ハンドラー

開発以外の環境では、[例外処理ミドルウェア](xref:fundamentals/error-handling)を使用してエラー ペイロードを生成できます。

1. `Startup.Configure` で、<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> を呼び出してミドルウェアを使用します。

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_UseExceptionHandler&highlight=9)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_UseExceptionHandler&highlight=9)]

    ::: moniker-end

1. `/error` ルートに応答するようにコントローラー アクションを構成します。

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Controllers/ErrorController.cs?name=snippet_ErrorController)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Controllers/ErrorController.cs?name=snippet_ErrorController)]

    ::: moniker-end

前の `Error` アクションは、[RFC 7807](https://tools.ietf.org/html/rfc7807) 準拠のペイロードをクライアントに送信します。

ローカル開発環境では、例外処理ミドルウェアによって、さらに詳しいコンテンツ ネゴシエーション結果も提供されます。 次の手順に従い、開発環境と運用環境で一貫性のあるペイロード形式を生成します。

1. `Startup.Configure` で、環境固有の例外処理ミドルウェア インスタンスを登録します。

    ::: moniker range=">= aspnetcore-3.0"

    ```csharp
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseExceptionHandler("/error-local-development");
        }
        else
        {
            app.UseExceptionHandler("/error");
        }
    }
    ```

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    ```csharp
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseExceptionHandler("/error-local-development");
        }
        else
        {
            app.UseExceptionHandler("/error");
        }
    }
    ```

    ::: moniker-end

    前のコードでは、ミドルウェアは次のように登録されます。

    * 開発環境では `/error-local-development` のルート。
    * 開発以外の環境では `/error` のルート。
    
1. 属性ルーティングをコントローラー アクションに適用します。

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Controllers/ErrorController.cs?name=snippet_ErrorControllerEnvironmentSpecific)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Controllers/ErrorController.cs?name=snippet_ErrorControllerEnvironmentSpecific)]

    ::: moniker-end

    前のコードでは、[コントローラー](xref:Microsoft.AspNetCore.Mvc.ControllerBase.Problem%2A)を呼び出して、応答を作成しています。 <xref:Microsoft.AspNetCore.Mvc.ProblemDetails>

## <a name="use-exceptions-to-modify-the-response"></a>例外を使用して応答を変更する

応答の内容は、コントローラーの外部で変更できます。 ASP.NET 4.x Web API の場合、これを行う方法の 1 つが <xref:System.Web.Http.HttpResponseException> 型の使用でした。 ASP.NET Core には同等の型が含まれていません。 `HttpResponseException` のサポートは以下の手順で追加することができます。

1. `HttpResponseException` という名前の一般的な例外の種類を作成します。

    [!code-csharp[](handle-errors/samples/3.x/Exceptions/HttpResponseException.cs?name=snippet_HttpResponseException)]

1. `HttpResponseExceptionFilter` という名前のアクション フィルターを作成します。

    [!code-csharp[](handle-errors/samples/3.x/Filters/HttpResponseExceptionFilter.cs?name=snippet_HttpResponseExceptionFilter)]

    前のフィルターでは、 `Order` 最大の整数値から10を引いたを指定しています。 これにより、パイプラインの最後で他のフィルターを実行できます。

1. `Startup.ConfigureServices` に、フィルター コレクションに対するアクション フィルターを追加します。

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

    ::: moniker range="= aspnetcore-2.2"
    
    [!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

    ::: moniker range="= aspnetcore-2.1"

    [!code-csharp[](handle-errors/samples/2.x/2.1/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

## <a name="validation-failure-error-response"></a>検証失敗のエラー応答

Web API コントローラーでは、モデルの検証が失敗すると、MVC が <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> という応答の種類で応答します。 MVC は <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.InvalidModelStateResponseFactory> の結果を使用して、検証失敗に対するエラー応答を作成します。 次の例では、`Startup.ConfigureServices` で、ファクトリを使用して応答の既定の種類を <xref:Microsoft.AspNetCore.Mvc.SerializableError> に変更します。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=4-13)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=5-14)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](handle-errors/samples/2.x/2.1/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=6-15)]

::: moniker-end

## <a name="client-error-response"></a>クライアントのエラー応答

"*エラー結果*" は、HTTP 状態コードが 400 以上の結果として定義されます。 Web API コントローラーの場合、MVC によってエラー結果が <xref:Microsoft.AspNetCore.Mvc.ProblemDetails> を含む結果に変換されます。

::: moniker range="= aspnetcore-2.1"

> [!IMPORTANT]
> ASP.NET Core 2.1 では、RFC 7807 にほぼ準拠した、問題の詳しい応答が生成されます。 100% のコンプライアンスが重要な場合は、プロジェクトを ASP.NET Core 2.2 以降にアップグレードしてください。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

エラー応答は、次のいずれかの方法で構成できます。

1. [ProblemDetailsFactory の実装](#implement-problemdetailsfactory)
1. [ApiBehaviorOptions.ClientErrorMapping の使用](#use-apibehavioroptionsclienterrormapping)

### <a name="implement-problemdetailsfactory"></a>`ProblemDetailsFactory` を実装する

MVC は、<xref:Microsoft.AspNetCore.Mvc.Infrastructure.ProblemDetailsFactory?displayProperty=fullName> を使用して、<xref:Microsoft.AspNetCore.Mvc.ProblemDetails> と <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> のすべてのインスタンスを生成します。 これには、クライアント エラー応答および検証失敗エラー応答と、<xref:Microsoft.AspNetCore.Mvc.ControllerBase.Problem%2A?displayProperty=nameWithType> および <xref:Microsoft.AspNetCore.Mvc.ControllerBase.ValidationProblem%2A?displayProperty=nameWithType> ヘルパー メソッドが含まれます。

問題の詳しい応答をカスタマイズするには、`Startup.ConfigureServices`で <xref:Microsoft.AspNetCore.Mvc.Infrastructure.ProblemDetailsFactory> のカスタム実装を登録します。

```csharp
public void ConfigureServices(IServiceCollection serviceCollection)
{
    services.AddControllers();
    services.AddTransient<ProblemDetailsFactory, CustomProblemDetailsFactory>();
}
```

::: moniker-end

::: moniker range="= aspnetcore-2.2"

エラー応答は、「[ApiBehaviorOptions.ClientErrorMapping の使用](#use-apibehavioroptionsclienterrormapping)」セクションの説明に従って構成できます。

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

### <a name="use-apibehavioroptionsclienterrormapping"></a>ApiBehaviorOptions.ClientErrorMapping の使用

`ProblemDetails` の応答の内容を構成するには、<xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.ClientErrorMapping%2A> プロパティを使用します。 たとえば、`Startup.ConfigureServices`の次のコードにより、404 応答の `type` プロパティが更新されます。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=8-9)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=9-10)]

::: moniker-end

## <a name="custom-middleware-to-handle-exceptions"></a>例外を処理するカスタムミドルウェア

例外処理ミドルウェアの既定値は、ほとんどのアプリに適しています。 特殊な例外処理を必要とするアプリの場合は、 [例外処理ミドルウェアをカスタマイズ](xref:fundamentals/error-handling#exception-handler-lambda)することを検討してください。
