---
title: ASP.NET Core でのサードパーティ コンテナーによるミドルウェアのアクティブ化
author: rick-anderson
description: ASP.NET Core で、ファクトリベースのアクティブ化とサードパーティ コンテナーによる厳密に型指定されたミドルウェアを使用する方法を説明します。
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
uid: fundamentals/middleware/extensibility-third-party-container
ms.openlocfilehash: bbbfe0e9fba33eba9d69101e5615e931ea917bc1
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587373"
---
# <a name="middleware-activation-with-a-third-party-container-in-aspnet-core"></a>ASP.NET Core でのサードパーティ コンテナーによるミドルウェアのアクティブ化

::: moniker range=">= aspnetcore-3.0"

この記事では、<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> と <xref:Microsoft.AspNetCore.Http.IMiddleware> を、サードパーティ コンテナーによる[ミドルウェア](xref:fundamentals/middleware/index)のアクティブ化の拡張ポイントとして使用する方法について説明します。 `IMiddlewareFactory` と `IMiddleware` の概要については、「<xref:fundamentals/middleware/extensibility>」をご覧ください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/middleware/extensibility-third-party-container/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

このサンプル アプリでは、`IMiddlewareFactory` の実装である `SimpleInjectorMiddlewareFactory` によるミドルウェアのアクティブ化を示します。 このサンプルでは、[Simple Injector](https://simpleinjector.org) 依存関係の挿入 (DI) コンテナーを使用しています。

サンプルのミドルウェアの実装は、クエリ文字列パラメーターで提供された値を記録します (`key`)。 ミドルウェアは、メモリ内データベースにクエリ文字列値を記録するのに、挿入されたデータベース コンテキスト (スコープ化されたサービス) を使用します。

> [!NOTE]
> このサンプル アプリでは、デモンストレーション目的でのみ [Simple Injector](https://github.com/simpleinjector/SimpleInjector) を使用しています。 Simple Injector の使用を推奨するものではありません。 Simple Injector のドキュメントと GitHub Issues に記載されているミドルウェアのアクティブ化方法は、Simple Injector の保守担当者たちから推奨されています。 詳細については、[Simple Injector のドキュメント](https://simpleinjector.readthedocs.io/en/latest/index.html)と [Simple Injector の GitHub リポジトリ](https://github.com/simpleinjector/SimpleInjector)を参照してください。

## <a name="imiddlewarefactory"></a>IMiddlewareFactory

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> では、ミドルウェアを作成するメソッドが提供されます。

サンプル アプリでは、`SimpleInjectorActivatedMiddleware` インスタンスを作成するミドルウェア ファクトリが実装されています。 このミドルウェア ファクトリでは、Simple Injector コンテナーを使用してミドルウェアを解決しています。

[!code-csharp[](extensibility-third-party-container/samples/3.x/SampleApp/Middleware/SimpleInjectorMiddlewareFactory.cs?name=snippet1&highlight=5-8,12)]

## <a name="imiddleware"></a>IMiddleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> では、アプリの要求パイプライン用にミドルウェアが定義されます。

`IMiddlewareFactory` の実装によってアクティブ化されるミドルウェア (*Middleware/SimpleInjectorActivatedMiddleware.cs*):

[!code-csharp[](extensibility-third-party-container/samples/3.x/SampleApp/Middleware/SimpleInjectorActivatedMiddleware.cs?name=snippet1)]

ミドルウェア (*Middleware/MiddlewareExtensions.cs*) の拡張機能が作成されます。

[!code-csharp[](extensibility-third-party-container/samples/3.x/SampleApp/Middleware/MiddlewareExtensions.cs?name=snippet1)]

`Startup.ConfigureServices` はいくつかのタスクを実行する必要があります。

* Simple Injector コンテナーを設定します。
* ファクトリとミドルウェアを登録します。
* Simple Injector コンテナーからアプリのデータベース コンテキストを利用できるようにします。

[!code-csharp[](extensibility-third-party-container/samples/3.x/SampleApp/Startup.cs?name=snippet1)]

ミドルウェアは、要求を処理する `Startup.Configure` のパイプラインに登録されます。

[!code-csharp[](extensibility-third-party-container/samples/3.x/SampleApp/Startup.cs?name=snippet2&highlight=12)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

この記事では、<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> と <xref:Microsoft.AspNetCore.Http.IMiddleware> を、サードパーティ コンテナーによる[ミドルウェア](xref:fundamentals/middleware/index)のアクティブ化の拡張ポイントとして使用する方法について説明します。 `IMiddlewareFactory` と `IMiddleware` の概要については、「<xref:fundamentals/middleware/extensibility>」をご覧ください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/middleware/extensibility-third-party-container/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

このサンプル アプリでは、`IMiddlewareFactory` の実装である `SimpleInjectorMiddlewareFactory` によるミドルウェアのアクティブ化を示します。 このサンプルでは、[Simple Injector](https://simpleinjector.org) 依存関係の挿入 (DI) コンテナーを使用しています。

サンプルのミドルウェアの実装は、クエリ文字列パラメーターで提供された値を記録します (`key`)。 ミドルウェアは、メモリ内データベースにクエリ文字列値を記録するのに、挿入されたデータベース コンテキスト (スコープ化されたサービス) を使用します。

> [!NOTE]
> このサンプル アプリでは、デモンストレーション目的でのみ [Simple Injector](https://github.com/simpleinjector/SimpleInjector) を使用しています。 Simple Injector の使用を推奨するものではありません。 Simple Injector のドキュメントと GitHub Issues に記載されているミドルウェアのアクティブ化方法は、Simple Injector の保守担当者たちから推奨されています。 詳細については、[Simple Injector のドキュメント](https://simpleinjector.readthedocs.io/en/latest/index.html)と [Simple Injector の GitHub リポジトリ](https://github.com/simpleinjector/SimpleInjector)を参照してください。

## <a name="imiddlewarefactory"></a>IMiddlewareFactory

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> では、ミドルウェアを作成するメソッドが提供されます。

サンプル アプリでは、`SimpleInjectorActivatedMiddleware` インスタンスを作成するミドルウェア ファクトリが実装されています。 このミドルウェア ファクトリでは、Simple Injector コンテナーを使用してミドルウェアを解決しています。

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Middleware/SimpleInjectorMiddlewareFactory.cs?name=snippet1&highlight=5-8,12)]

## <a name="imiddleware"></a>IMiddleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> では、アプリの要求パイプライン用にミドルウェアが定義されます。

`IMiddlewareFactory` の実装によってアクティブ化されるミドルウェア (*Middleware/SimpleInjectorActivatedMiddleware.cs*):

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Middleware/SimpleInjectorActivatedMiddleware.cs?name=snippet1)]

ミドルウェア (*Middleware/MiddlewareExtensions.cs*) の拡張機能が作成されます。

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Middleware/MiddlewareExtensions.cs?name=snippet1)]

`Startup.ConfigureServices` はいくつかのタスクを実行する必要があります。

* Simple Injector コンテナーを設定します。
* ファクトリとミドルウェアを登録します。
* Simple Injector コンテナーからアプリのデータベース コンテキストを利用できるようにします。

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Startup.cs?name=snippet1)]

ミドルウェアは、要求を処理する `Startup.Configure` のパイプラインに登録されます。

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Startup.cs?name=snippet2&highlight=12)]

::: moniker-end

## <a name="additional-resources"></a>その他の技術情報

* [ミドルウェア](xref:fundamentals/middleware/index)
* [ファクトリ ベースのミドルウェアのアクティブ化](xref:fundamentals/middleware/extensibility)
* [Simple Injector の GitHub リポジトリ](https://github.com/simpleinjector/SimpleInjector)
* [Simple Injector のドキュメント](https://simpleinjector.readthedocs.io/en/latest/index.html)
