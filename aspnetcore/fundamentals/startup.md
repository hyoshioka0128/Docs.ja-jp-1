---
title: ASP.NET Core でのアプリケーションのスタートアップ
author: rick-anderson
description: ASP.NET Core の Startup クラスがサービスとアプリケーションの要求パイプラインをどのように構成しているかを説明します。
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
uid: fundamentals/startup
ms.openlocfilehash: e2825476c5fe36571b365ac5ee3c57ff4db61b87
ms.sourcegitcommit: cc405f20537484744423ddaf87bd1e7d82b6bdf0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/21/2021
ms.locfileid: "98658678"
---
# <a name="app-startup-in-aspnet-core"></a>ASP.NET Core でのアプリケーションのスタートアップ

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)、[Tom Dykstra](https://github.com/tdykstra)、[Steve Smith](https://ardalis.com)

`Startup` クラスはサービスとアプリケーションの要求パイプラインを構成します。

::: moniker range=">= aspnetcore-3.0"

## <a name="the-startup-class"></a>Startup クラス

ASP.NET Core アプリケーションでは `Startup` クラスが使用されています。このクラスは規約に従って `Startup` と名前が付けられています。 `Startup` クラス:

* 必要に応じて <xref:Microsoft.AspNetCore.Hosting.StartupBase.ConfigureServices*> メソッドを含め、アプリの *サービス* を構成することができます。 サービスとは、アプリ機能を提供する再利用可能なコンポーネントです。 サービスは `ConfigureServices` に *登録され*、[依存関係の挿入 (DI)](xref:fundamentals/dependency-injection) または <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.ApplicationServices*> を介してアプリ全体で利用されます。
* アプリの要求処理パイプラインを作成するために <xref:Microsoft.AspNetCore.Hosting.StartupBase.Configure*> メソッドを含めます。

`ConfigureServices` と `Configure` はアプリの起動時に ASP.NET Core ランタイムによって呼び出されます。

[!code-csharp[](startup/3.0_samples/StartupFilterSample/Startup.cs?name=snippet)]

上記のサンプルは [Razor Pages](xref:razor-pages/index) 用です。MVC バージョンは似ています。


アプリの[ホスト](xref:fundamentals/index#host)がビルドされるときに、`Startup` クラスが指定されます。 `Startup` クラスは通常、ホスト ビルダーで [WebHostBuilderExtensions.UseStartup\<TStartup>](xref:Microsoft.AspNetCore.Hosting.WebHostBuilderExtensions.UseStartup*) メソッドを呼び出すことで指定されます。

[!code-csharp[](startup/3.0_samples/Program3.cs?name=snippet_Program&highlight=12)]

ホストには、`Startup` クラス コンストラクターで使用できるサービスが用意されています。 アプリケーションから `ConfigureServices` 経由でサービスが追加されます。 ホスト サービスとアプリ サービスの両方が `Configure` 内とアプリ全体で使用できます。

[汎用ホスト](xref:fundamentals/host/generic-host) (<xref:Microsoft.Extensions.Hosting.IHostBuilder>) を使用すると、次のサービスの種類のみを `Startup` コンストラクターに挿入できます。

* <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment>
* <xref:Microsoft.Extensions.Hosting.IHostEnvironment>
* <xref:Microsoft.Extensions.Configuration.IConfiguration>

[!code-csharp[](startup/3.0_samples/StartupFilterSample/StartUp2.cs?name=snippet)]

ほとんどのサービスは、`Configure` メソッドが呼び出されるまで使用できません。

### <a name="multiple-startup"></a>マルチ スタートアップ

アプリケーションの環境 (たとえば `StartupDevelopment`) ごとに個別の `Startup` クラスが定義されると、実行時に適切な `Startup` クラスが選択されます。 名前のサフィックスが現在の環境と一致するクラスが優先されます。 アプリケーションが Development 環境で実行され、`Startup` クラスと `StartupDevelopment` クラスの両方が含まれている場合は、`StartupDevelopment` クラスが使用されます。 詳細については、「[Use multiple environments](xref:fundamentals/environments#environment-based-startup-class-and-methods)」(複数の環境の使用) を参照してください。

ホストの詳細については、「[ホスト](xref:fundamentals/index#host)」を参照してください。 スタートアップ時のエラー処理については、「[Startup exception handling](xref:fundamentals/error-handling#startup-exception-handling)」(スタートアップ例外処理) を参照してください。

## <a name="the-configureservices-method"></a>ConfigureServices メソッド

<xref:Microsoft.AspNetCore.Hosting.StartupBase.ConfigureServices*> メソッド:

* 任意。
* アプリケーションのサービスを構成する `Configure` メソッドの前にホストによって呼び出されます。
* 規約によって[構成オプション](xref:fundamentals/configuration/index)が設定されます。

ホストでは、`Startup` メソッドが呼び出される前にいくつかのサービスを構成することができます。 詳細については、「[ホスト](xref:fundamentals/index#host)」を参照してください。

多くの設定が必要な機能には、<xref:Microsoft.Extensions.DependencyInjection.IServiceCollection> 上の `Add{Service}` 拡張メソッドがあります。 たとえば、**Add** DbContext、**Add** DefaultIdentity、**Add** EntityFrameworkStores、**Add**RazorPages です。

[!code-csharp[](startup/3.0_samples/StartupFilterSample/StartupIdentity.cs?name=snippet)]

サービス コンテナーにサービスを追加すると、アプリケーション内と `Configure` メソッド内でサービスを利用できるようになります。 サービスは[依存関係の挿入](xref:fundamentals/dependency-injection)または <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.ApplicationServices*> によって解決されます。

## <a name="the-configure-method"></a>Configure メソッド

<xref:Microsoft.AspNetCore.Hosting.StartupBase.Configure*> メソッドは、アプリケーションが HTTP 要求にどのように応答するかを指定するために使用されます。 要求パイプラインは、[ミドルウェア](xref:fundamentals/middleware/index) コンポーネントを <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder> インスタンスに追加することで構成されます。 `IApplicationBuilder` は `Configure` メソッドから使用できますが、サービス コンテナーに登録されていません。 ホスティングによって `IApplicationBuilder` が作成され、`Configure` に直接渡されます。

[ASP.NET Core テンプレート](/dotnet/core/tools/dotnet-new)では、次をサポートするパイプラインが構成されます。

* [開発者例外ページ](xref:fundamentals/error-handling#developer-exception-page)
* [例外ハンドラー](xref:fundamentals/error-handling#exception-handler-page)
* [HTTP Strict Transport Security (HSTS)](xref:security/enforcing-ssl#http-strict-transport-security-protocol-hsts)
* [HTTPS リダイレクト](xref:security/enforcing-ssl)
* [静的ファイル](xref:fundamentals/static-files)
* ASP.NET Core [MVC](xref:mvc/overview) と [Razor ページ](xref:razor-pages/index)


[!code-csharp[](startup/3.0_samples/StartupFilterSample/Startup.cs?name=snippet)]

上記のサンプルは [Razor Pages](xref:razor-pages/index) 用です。MVC バージョンは似ています。

各 `Use` 拡張メソッドによって、要求パイプラインに 1 つまたは複数のミドルウェア コンポーネントが追加されます。 たとえば、<xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles*> では、[静的ファイル](xref:fundamentals/static-files)を提供するように、[ミドルウェア](xref:fundamentals/middleware/index)を構成します。

要求パイプライン内の各ミドルウェア コンポーネントは、パイプラインの次のコンポーネントを呼び出すか、妥当な場合はチェーンをショートさせます。

`IWebHostEnvironment`、`ILoggerFactory`、または `ConfigureServices` で定義された任意のものなどの追加サービスを `Configure` メソッド シグネチャで指定できます。 使用可能な場合、これらのサービスが挿入されます。

`IApplicationBuilder` の使用方法およびミドルウェアの処理の順序については、「<xref:fundamentals/middleware/index>」を参照してください。

<a name="convenience-methods"></a>

## <a name="configure-services-without-startup"></a>スタートアップを使用せずにサービスを構成する

`Startup` クラスを使用せず、サービスと要求処理パイプラインを構成するには、ホスト ビルダーで便利なメソッド、`ConfigureServices` と `Configure` を呼び出します。 `ConfigureServices` の複数回の呼び出しでは、互いに追加されます。 `Configure` メソッドが複数回呼び出された場合、最後の `Configure` 呼び出しが使用されます。

[!code-csharp[](startup/3.0_samples/StartupFilterSample/Program1.cs?name=snippet)]

## <a name="extend-startup-with-startup-filters"></a>スタートアップ フィルターを使用した Startup の拡張

<xref:Microsoft.AspNetCore.Hosting.IStartupFilter> を使用して次のことを行います。

* `Use{Middleware}` を明示的に呼び出さずに、アプリの [Configure](#the-configure-method) ミドルウェア パイプラインの先頭または末尾でミドルウェアを構成します。 `IStartupFilter` は、アプリの作成者が既定のミドルウェアを明示的に登録する必要がないよう、パイプラインの先頭に既定値を追加するために、ASP.NET Core によって使用されます。 `IStartupFilter` により、アプリの作成者に代わって別のコンポーネントが `Use{Middleware}` を呼び出すことを許可します。
* `Configure` メソッドのパイプラインを作成します。 [IStartupFilter.Configure](xref:Microsoft.AspNetCore.Hosting.IStartupFilter.Configure*) では、ライブラリによって追加されたミドルウェアの前後に実行するように、ミドルウェアを設定することができます。

`IStartupFilter` には、`Action<IApplicationBuilder>` を受け取って返す <xref:Microsoft.AspNetCore.Hosting.StartupBase.Configure*> が実装されています。 <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder> で、アプリケーションの要求パイプラインを構成するクラスを定義します。 詳細については、「[Create a middleware pipeline with IApplicationBuilder](xref:fundamentals/middleware/index#create-a-middleware-pipeline-with-iapplicationbuilder)」(IApplicationBuilder を使用したミドルウェア パイプラインの作成) を参照してください。

各 `IStartupFilter` によって、要求パイプラインで 1 つまたは複数のミドルウェアを追加できます。 フィルターは、サービス コンテナーに追加された順に呼び出されます。 フィルターは、コントロールを次のフィルターに渡す前または後にミドルウェアを追加できるため、アプリケーション パイプラインの先頭または末尾に追加されます。

`IStartupFilter` でミドルウェアを登録する方法を次の例に示します。 `RequestSetOptionsMiddleware` ミドルウェアによって、クエリ文字列パラメーターからオプション値が設定されます。

[!code-csharp[](startup/3.0_samples/StartupFilterSample/RequestSetOptionsMiddleware.cs?name=snippet1)]

`RequestSetOptionsMiddleware` は `RequestSetOptionsStartupFilter` クラスで構成されています。

[!code-csharp[](startup/3.0_samples/StartupFilterSample/RequestSetOptionsStartupFilter.cs?name=snippet1&highlight=7)]

`IStartupFilter` は <xref:Microsoft.AspNetCore.Hosting.StartupBase.ConfigureServices*> のサービス コンテナーに登録されています。

[!code-csharp[](startup/3.0_samples/StartupFilterSample/Program.cs?name=snippet&highlight=19-20)]

`option` のクエリ文字列パラメーターを指定すると、ミドルウェアでは ASP.NET Core ミドルウェアによって応答がレンダリングされる前に値の割り当てを処理します。

ミドルウェアの実行順序は、`IStartupFilter` の登録順に設定されています。

* 複数の `IStartupFilter` の実装が、同じオブジェクトとやり取りする可能性があります。 順序が重要な場合は、ミドルウェアの実行順序に合わせて `IStartupFilter` サービスの登録順序を指定してください。
* `IStartupFilter` に登録された他のアプリケーション ミドルウェアの前または後に実行される `IStartupFilter` の実装が 1 つまたは複数あるミドルウェアを、ライブラリで追加することができます。 ライブラリの `IStartupFilter` よって追加されたミドルウェアの前に `IStartupFilter` ミドルウェアを呼び出すには、次のようにします。

  * サービス コンテナーにライブラリを追加する前に、サービス登録を配置する
  * 後で呼び出すために、ライブラリが追加された後にサービス登録を配置する

## <a name="add-configuration-at-startup-from-an-external-assembly"></a>外部アセンブリからの起動時に構成を追加する

<xref:Microsoft.AspNetCore.Hosting.IHostingStartup> の実装により、アプリの `Startup` クラスの外部にある外部アセンブリから、起動時に拡張機能をアプリに追加できるようになります。 詳細については、「<xref:fundamentals/configuration/platform-specific-configuration>」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* [ホスト](xref:fundamentals/index#host)
* <xref:fundamentals/environments>
* <xref:fundamentals/middleware/index>
* <xref:fundamentals/logging/index>
* <xref:fundamentals/configuration/index>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

## <a name="the-startup-class"></a>Startup クラス

ASP.NET Core アプリケーションでは `Startup` クラスが使用されています。このクラスは規約に従って `Startup` と名前が付けられています。 `Startup` クラス:

* 必要に応じて <xref:Microsoft.AspNetCore.Hosting.StartupBase.ConfigureServices*> メソッドを含め、アプリの *サービス* を構成することができます。 サービスとは、アプリ機能を提供する再利用可能なコンポーネントです。 サービスは `ConfigureServices` に *登録され*、[依存関係の挿入 (DI)](xref:fundamentals/dependency-injection) または <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.ApplicationServices*> を介してアプリ全体で利用されます。
* アプリの要求処理パイプラインを作成するために <xref:Microsoft.AspNetCore.Hosting.StartupBase.Configure*> メソッドを含めます。

`ConfigureServices` と `Configure` はアプリの起動時に ASP.NET Core ランタイムによって呼び出されます。

[!code-csharp[](startup/sample_snapshot/Startup1.cs)]

アプリの[ホスト](xref:fundamentals/index#host)がビルドされるときに、`Startup` クラスが指定されます。 `Startup` クラスは通常、ホスト ビルダーで [WebHostBuilderExtensions.UseStartup\<TStartup>](xref:Microsoft.AspNetCore.Hosting.WebHostBuilderExtensions.UseStartup*) メソッドを呼び出すことで指定されます。

[!code-csharp[](startup/sample_snapshot/Program3.cs?name=snippet_Program&highlight=12)]

ホストには、`Startup` クラス コンストラクターで使用できるサービスが用意されています。 アプリケーションから `ConfigureServices` 経由でサービスが追加されます。 これで、ホスト サービスとアプリケーション サービスの両方が `Configure` 内とアプリ全体で使用できるようになります。

`Startup` クラスへの[依存関係挿入](xref:fundamentals/dependency-injection)の一般的な用途は、以下を挿入する場合です。

* <xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment> (環境別にサービスを構成するため)。
* <xref:Microsoft.Extensions.Configuration.IConfiguration> (構成を読み取るため)。
* <xref:Microsoft.Extensions.Logging.ILoggerFactory> (`Startup.ConfigureServices` でロガーを作成するため)。

[!code-csharp[](startup/sample_snapshot/Startup2.cs?highlight=7-8)]

ほとんどのサービスは、`Configure` メソッドが呼び出されるまで使用できません。

### <a name="multiple-startup"></a>マルチ スタートアップ

アプリケーションの環境 (たとえば `StartupDevelopment`) ごとに個別の `Startup` クラスが定義されると、実行時に適切な `Startup` クラスが選択されます。 名前のサフィックスが現在の環境と一致するクラスが優先されます。 アプリケーションが Development 環境で実行され、`Startup` クラスと `StartupDevelopment` クラスの両方が含まれている場合は、`StartupDevelopment` クラスが使用されます。 詳細については、「[Use multiple environments](xref:fundamentals/environments#environment-based-startup-class-and-methods)」(複数の環境の使用) を参照してください。

ホストの詳細については、「[ホスト](xref:fundamentals/index#host)」を参照してください。 スタートアップ時のエラー処理については、「[Startup exception handling](xref:fundamentals/error-handling#startup-exception-handling)」(スタートアップ例外処理) を参照してください。

## <a name="the-configureservices-method"></a>ConfigureServices メソッド

<xref:Microsoft.AspNetCore.Hosting.StartupBase.ConfigureServices*> メソッド:

* 任意。
* アプリケーションのサービスを構成する `Configure` メソッドの前にホストによって呼び出されます。
* 規約によって[構成オプション](xref:fundamentals/configuration/index)が設定されます。

ホストでは、`Startup` メソッドが呼び出される前にいくつかのサービスを構成することができます。 詳細については、「[ホスト](xref:fundamentals/index#host)」を参照してください。

多くの設定が必要な機能には、<xref:Microsoft.Extensions.DependencyInjection.IServiceCollection> 上の `Add{Service}` 拡張メソッドがあります。 たとえば、**Add** DbContext、**Add** DefaultIdentity、**Add** EntityFrameworkStores、**Add**RazorPages です。

[!code-csharp[](startup/sample_snapshot/Startup3.cs)]

サービス コンテナーにサービスを追加すると、アプリケーション内と `Configure` メソッド内でサービスを利用できるようになります。 サービスは[依存関係の挿入](xref:fundamentals/dependency-injection)または <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.ApplicationServices*> によって解決されます。

`SetCompatibilityVersion` について詳しくは、「[SetCompatibilityVersion](xref:mvc/compatibility-version)」をご覧ください。

## <a name="the-configure-method"></a>Configure メソッド

<xref:Microsoft.AspNetCore.Hosting.StartupBase.Configure*> メソッドは、アプリケーションが HTTP 要求にどのように応答するかを指定するために使用されます。 要求パイプラインは、[ミドルウェア](xref:fundamentals/middleware/index) コンポーネントを <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder> インスタンスに追加することで構成されます。 `IApplicationBuilder` は `Configure` メソッドから使用できますが、サービス コンテナーに登録されていません。 ホスティングによって `IApplicationBuilder` が作成され、`Configure` に直接渡されます。

[ASP.NET Core テンプレート](/dotnet/core/tools/dotnet-new)では、次をサポートするパイプラインが構成されます。

* [開発者例外ページ](xref:fundamentals/error-handling#developer-exception-page)
* [例外ハンドラー](xref:fundamentals/error-handling#exception-handler-page)
* [HTTP Strict Transport Security (HSTS)](xref:security/enforcing-ssl#http-strict-transport-security-protocol-hsts)
* [HTTPS リダイレクト](xref:security/enforcing-ssl)
* [静的ファイル](xref:fundamentals/static-files)
* ASP.NET Core [MVC](xref:mvc/overview) と [Razor ページ](xref:razor-pages/index)
* [一般データ保護規制 (GDPR)](xref:security/gdpr)

[!code-csharp[](startup/sample_snapshot/Startup4.cs)]

各 `Use` 拡張メソッドによって、要求パイプラインに 1 つまたは複数のミドルウェア コンポーネントが追加されます。 たとえば、<xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles*> では、[静的ファイル](xref:fundamentals/static-files)を提供するように、[ミドルウェア](xref:fundamentals/middleware/index)を構成します。

要求パイプライン内の各ミドルウェア コンポーネントは、パイプラインの次のコンポーネントを呼び出すか、妥当な場合はチェーンをショートさせます。

`IHostingEnvironment` や `ILoggerFactory`、または `ConfigureServices` で定義された任意のものなどの追加サービスを `Configure` メソッド シグネチャで指定できます。 使用可能な場合、これらのサービスが挿入されます。

`IApplicationBuilder` の使用方法およびミドルウェアの処理の順序については、「<xref:fundamentals/middleware/index>」を参照してください。

<a name="convenience-methods"></a>

## <a name="configure-services-without-startup"></a>スタートアップを使用せずにサービスを構成する

`Startup` クラスを使用せず、サービスと要求処理パイプラインを構成するには、ホスト ビルダーで便利なメソッド、`ConfigureServices` と `Configure` を呼び出します。 `ConfigureServices` の複数回の呼び出しでは、互いに追加されます。 `Configure` メソッドが複数回呼び出された場合、最後の `Configure` 呼び出しが使用されます。

[!code-csharp[](startup/sample_snapshot/Program1.cs?highlight=16,20)]

## <a name="extend-startup-with-startup-filters"></a>スタートアップ フィルターを使用した Startup の拡張

<xref:Microsoft.AspNetCore.Hosting.IStartupFilter> を使用して次のことを行います。

* `Use{Middleware}` を明示的に呼び出さずに、アプリの [Configure](#the-configure-method) ミドルウェア パイプラインの先頭または末尾でミドルウェアを構成します。 `IStartupFilter` は、アプリの作成者が既定のミドルウェアを明示的に登録する必要がないよう、パイプラインの先頭に既定値を追加するために、ASP.NET Core によって使用されます。 `IStartupFilter` は、アプリの作成者に代わって、別のコンポーネント呼び出し `Use{Middleware}` を許可します。
* `Configure` メソッドのパイプラインを作成します。 [IStartupFilter.Configure](xref:Microsoft.AspNetCore.Hosting.IStartupFilter.Configure*) では、ライブラリによって追加されたミドルウェアの前後に実行するように、ミドルウェアを設定することができます。

`IStartupFilter` には、`Action<IApplicationBuilder>` を受け取って返す <xref:Microsoft.AspNetCore.Hosting.StartupBase.Configure*> が実装されています。 <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder> で、アプリケーションの要求パイプラインを構成するクラスを定義します。 詳細については、「[Create a middleware pipeline with IApplicationBuilder](xref:fundamentals/middleware/index#create-a-middleware-pipeline-with-iapplicationbuilder)」(IApplicationBuilder を使用したミドルウェア パイプラインの作成) を参照してください。

各 `IStartupFilter` によって、要求パイプラインで 1 つまたは複数のミドルウェアを追加できます。 フィルターは、サービス コンテナーに追加された順に呼び出されます。 フィルターは、コントロールを次のフィルターに渡す前または後にミドルウェアを追加できるため、アプリケーション パイプラインの先頭または末尾に追加されます。

`IStartupFilter` でミドルウェアを登録する方法を次の例に示します。 `RequestSetOptionsMiddleware` ミドルウェアによって、クエリ文字列パラメーターからオプション値が設定されます。

[!code-csharp[](startup/sample_snapshot/RequestSetOptionsMiddleware.cs?name=snippet1&highlight=21)]

`RequestSetOptionsMiddleware` は `RequestSetOptionsStartupFilter` クラスで構成されています。

[!code-csharp[](startup/sample_snapshot/RequestSetOptionsStartupFilter.cs?name=snippet1&highlight=7)]

`IStartupFilter` は <xref:Microsoft.AspNetCore.Hosting.StartupBase.ConfigureServices*> のサービス コンテナーに登録されています。

[!code-csharp[](startup/sample_snapshot/Program2.cs?name=snippet1&highlight=4-5)]

`option` のクエリ文字列パラメーターを指定すると、ミドルウェアでは ASP.NET Core ミドルウェアによって応答がレンダリングされる前に値の割り当てを処理します。

ミドルウェアの実行順序は、`IStartupFilter` の登録順に設定されています。

* 複数の `IStartupFilter` の実装が、同じオブジェクトとやり取りする可能性があります。 順序が重要な場合は、ミドルウェアの実行順序に合わせて `IStartupFilter` サービスの登録順序を指定してください。
* `IStartupFilter` に登録された他のアプリケーション ミドルウェアの前または後に実行される `IStartupFilter` の実装が 1 つまたは複数あるミドルウェアを、ライブラリで追加することができます。 ライブラリの `IStartupFilter` よって追加されたミドルウェアの前に `IStartupFilter` ミドルウェアを呼び出すには、次のようにします。

  * サービス コンテナーにライブラリを追加する前に、サービス登録を配置する
  * 後で呼び出すために、ライブラリが追加された後にサービス登録を配置する

## <a name="add-configuration-at-startup-from-an-external-assembly"></a>外部アセンブリからの起動時に構成を追加する

<xref:Microsoft.AspNetCore.Hosting.IHostingStartup> の実装により、アプリの `Startup` クラスの外部にある外部アセンブリから、起動時に拡張機能をアプリに追加できるようになります。 詳細については、「<xref:fundamentals/configuration/platform-specific-configuration>」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* [ホスト](xref:fundamentals/index#host)
* <xref:fundamentals/environments>
* <xref:fundamentals/middleware/index>
* <xref:fundamentals/logging/index>
* <xref:fundamentals/configuration/index>

::: moniker-end
