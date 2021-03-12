---
title: ASP.NET Core でのファクトリ ベースのミドルウェアのアクティブ化
author: rick-anderson
description: ASP.NET Core で、ファクトリ ベースの厳密に型指定されたミドルウェアをアクティブ化する方法を説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 09/22/2019
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
uid: fundamentals/middleware/extensibility
ms.openlocfilehash: c4835d0d9b0cc20a1176bbb5b0aedf3c5b7d51af
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587190"
---
# <a name="factory-based-middleware-activation-in-aspnet-core"></a>ASP.NET Core でのファクトリ ベースのミドルウェアのアクティブ化

::: moniker range=">= aspnetcore-3.0"

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>/<xref:Microsoft.AspNetCore.Http.IMiddleware> は、[ミドルウェア](xref:fundamentals/middleware/index)のアクティブ化の拡張ポイントです。

<xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*> 拡張メソッドでは、ミドルウェアの登録済みの型で <xref:Microsoft.AspNetCore.Http.IMiddleware> が実装されているかが確認されます。 実装されている場合、規則に基づくミドルウェアのライセンス認証ロジックを使用する代わりに、コンテナーに登録されている <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> インスタンスが <xref:Microsoft.AspNetCore.Http.IMiddleware> の実装の解決に使用されます。 ミドルウェアは、アプリのサービス コンテナーで、[スコープ サービスまたは一時的サービス](xref:fundamentals/dependency-injection#service-lifetimes)として登録されています。

利点:

* クライアント要求ごとにアクティブ化 (スコープ サービスの挿入)
* ミドルウェアの厳密な型指定

<xref:Microsoft.AspNetCore.Http.IMiddleware> はクライアント要求 (接続) ごとにアクティブ化されているので、スコープ サービスをミドルウェアのコンストラクターに挿入することができます。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/middleware/extensibility/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="imiddleware"></a>IMiddleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> では、アプリの要求パイプライン用にミドルウェアが定義されます。 要求は、[InvokeAsync (HttpContext, RequestDelegate)](xref:Microsoft.AspNetCore.Http.IMiddleware.InvokeAsync*) メソッドによって処理され、ミドルウェアの実行を表す <xref:System.Threading.Tasks.Task> が返されます。

規則でアクティブ化されるミドルウェア:

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Middleware/ConventionalMiddleware.cs?name=snippet1)]

<xref:Microsoft.AspNetCore.Http.MiddlewareFactory> でアクティブ化されるミドルウェア:

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Middleware/FactoryActivatedMiddleware.cs?name=snippet1)]

ミドルウェア用に拡張機能が作成されます。

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Middleware/MiddlewareExtensions.cs?name=snippet1)]

<xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*> では、ファクトリでアクティブ化されたミドルウェアにオブジェクトを渡すことはできません。

```csharp
public static IApplicationBuilder UseFactoryActivatedMiddleware(
    this IApplicationBuilder builder, bool option)
{
    // Passing 'option' as an argument throws a NotSupportedException at runtime.
    return builder.UseMiddleware<FactoryActivatedMiddleware>(option);
}
```

ファクトリでアクティブ化されたミドルウェアは、`Startup.ConfigureServices` の組み込みのコンテナーに追加されます。

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet1&highlight=6)]

この 2 つのミドルウェアは、要求を処理する `Startup.Configure` のパイプラインに登録されます。

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet2&highlight=12-13)]

## <a name="imiddlewarefactory"></a>IMiddlewareFactory

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> では、ミドルウェアを作成するメソッドが提供されます。 ミドルウェア ファクトリの実装は、スコープ化されたサービスとして、コンテナーに登録されます。

既定の <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> の実装である <xref:Microsoft.AspNetCore.Http.MiddlewareFactory> は、[Microsoft.AspNetCore.Http](https://www.nuget.org/packages/Microsoft.AspNetCore.Http/) パッケージにあります。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>/<xref:Microsoft.AspNetCore.Http.IMiddleware> は、[ミドルウェア](xref:fundamentals/middleware/index)のアクティブ化の拡張ポイントです。

<xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*> 拡張メソッドでは、ミドルウェアの登録済みの型で <xref:Microsoft.AspNetCore.Http.IMiddleware> が実装されているかが確認されます。 実装されている場合、規則に基づくミドルウェアのライセンス認証ロジックを使用する代わりに、コンテナーに登録されている <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> インスタンスが <xref:Microsoft.AspNetCore.Http.IMiddleware> の実装の解決に使用されます。 ミドルウェアは、アプリのサービス コンテナーで、[スコープ サービスまたは一時的サービス](xref:fundamentals/dependency-injection#service-lifetimes)として登録されています。

利点:

* クライアント要求ごとにアクティブ化 (スコープ サービスの挿入)
* ミドルウェアの厳密な型指定

<xref:Microsoft.AspNetCore.Http.IMiddleware> はクライアント要求 (接続) ごとにアクティブ化されているので、スコープ サービスをミドルウェアのコンストラクターに挿入することができます。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/middleware/extensibility/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="imiddleware"></a>IMiddleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> では、アプリの要求パイプライン用にミドルウェアが定義されます。 要求は、[InvokeAsync (HttpContext, RequestDelegate)](xref:Microsoft.AspNetCore.Http.IMiddleware.InvokeAsync*) メソッドによって処理され、ミドルウェアの実行を表す <xref:System.Threading.Tasks.Task> が返されます。

規則でアクティブ化されるミドルウェア:

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/ConventionalMiddleware.cs?name=snippet1)]

<xref:Microsoft.AspNetCore.Http.MiddlewareFactory> でアクティブ化されるミドルウェア:

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/FactoryActivatedMiddleware.cs?name=snippet1)]

ミドルウェア用に拡張機能が作成されます。

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/MiddlewareExtensions.cs?name=snippet1)]

<xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*> では、ファクトリでアクティブ化されたミドルウェアにオブジェクトを渡すことはできません。

```csharp
public static IApplicationBuilder UseFactoryActivatedMiddleware(
    this IApplicationBuilder builder, bool option)
{
    // Passing 'option' as an argument throws a NotSupportedException at runtime.
    return builder.UseMiddleware<FactoryActivatedMiddleware>(option);
}
```

ファクトリでアクティブ化されたミドルウェアは、`Startup.ConfigureServices` の組み込みのコンテナーに追加されます。

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet1&highlight=6)]

この 2 つのミドルウェアは、要求を処理する `Startup.Configure` のパイプラインに登録されます。

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet2&highlight=13-14)]

## <a name="imiddlewarefactory"></a>IMiddlewareFactory

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> では、ミドルウェアを作成するメソッドが提供されます。 ミドルウェア ファクトリの実装は、スコープ化されたサービスとして、コンテナーに登録されます。

既定の <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> の実装である <xref:Microsoft.AspNetCore.Http.MiddlewareFactory> は、[Microsoft.AspNetCore.Http](https://www.nuget.org/packages/Microsoft.AspNetCore.Http/) パッケージにあります。

::: moniker-end

## <a name="additional-resources"></a>その他の技術情報

* <xref:fundamentals/middleware/index>
* <xref:fundamentals/middleware/extensibility-third-party-container>
