---
title: ASP.NET Core での依存関係の挿入
author: rick-anderson
description: ASP.NET Core で依存関係の挿入を実装する方法とそれを使う方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 7/21/2020
no-loc:
- appsettings.json
- ASP.NET Core Identity
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: fundamentals/dependency-injection
ms.openlocfilehash: ce6804a58c5b17b57732713acc3aeca15042da0a
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589713"
---
# <a name="dependency-injection-in-aspnet-core"></a>ASP.NET Core での依存関係の挿入

::: moniker range=">= aspnetcore-3.0"

作成者: [Kirk Larkin](https://twitter.com/serpent5)、[Steve Smith](https://ardalis.com/)、[Scott Addie](https://scottaddie.com)、[Brandon Dahler](https://github.com/brandondahler)

ASP.NET Core では依存関係の挿入 (DI) ソフトウェア設計パターンがサポートされています。これは、クラスとその依存関係の間で[制御の反転 (IoC)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) を実現するための手法です。

MVC コントローラー内部における依存関係の挿入に固有の情報については、<xref:mvc/controllers/dependency-injection> をご覧ください。

Web アプリ以外のアプリケーションで依存関係の挿入を使用する方法の詳細については、「[.NET での依存関係の挿入](/dotnet/core/extensions/dependency-injection)」を参照してください。

オプションとしての依存関係挿入の詳細については、「<xref:fundamentals/configuration/options>」を参照してください。

このトピックでは、ASP.NET Core での依存関係の挿入について説明します。 依存関係の挿入の使用に関する主なドキュメントは、「[.NET での依存関係の挿入](/dotnet/core/extensions/dependency-injection)」に含まれています。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/dependency-injection/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="overview-of-dependency-injection"></a>依存関係の挿入の概要

"*依存関係*" とは、他のオブジェクトが依存するオブジェクトのことです。 他のクラスが依存している、次の `WriteMessage` メソッドを備えた `MyDependency` クラスを調べます。

```csharp
public class MyDependency
{
    public void WriteMessage(string message)
    {
        Console.WriteLine($"MyDependency.WriteMessage called. Message: {message}");
    }
}
```

クラスは、`MyDependency` クラスのインスタンスを作成して、その `WriteMessage` メソッドを使用することができます。 次の例で、`MyDependency` クラスは `IndexModel` クラスの依存関係です。

```csharp
public class IndexModel : PageModel
{
    private readonly MyDependency _dependency = new MyDependency();

    public void OnGet()
    {
        _dependency.WriteMessage("IndexModel.OnGet created this message.");
    }
}
```

このクラスは `MyDependency` クラスを作成し、これに直接依存しています。 コードの依存関係には、前の例のような問題が含まれ、次の理由から回避する必要があります。

* `MyDependency` を別の実装で置き換えるには、`IndexModel` クラスを変更する必要があります。
* `MyDependency` が依存関係を含んでいる場合、これらは `IndexModel` クラスによって構成する必要があります。 複数のクラスが `MyDependency` に依存している大規模なプロジェクトでは、構成コードがアプリ全体に分散するようになります。
* このような実装では、単体テストを行うことが困難です。 アプリはモックまたはスタブの `MyDependency` クラスを使用する必要がありますが、この方法では不可能です。

依存関係の挿入は、次の方法によってこれらの問題に対応します。

* 依存関係の実装を抽象化するための、インターフェイスまたは基底クラスの使用。
* サービス コンテナー内の依存関係の登録。 ASP.NET Core には、組み込みのサービス コンテナー <xref:System.IServiceProvider> が用意されています。 サービスは通常、アプリの `Startup.ConfigureServices` メソッドに登録されています。
* サービスを使用するクラスのコンストラクターへの、サービスの "*挿入*"。 依存関係のインスタンスの作成、およびインスタンスが不要になったときの廃棄の役割を、フレームワークが担当します。

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/dependency-injection/samples)では、`IMyDependency` インターフェイスは、`WriteMessage` メソッドを定義します。

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Interfaces/IMyDependency.cs?name=snippet1)]

このインターフェイスは、具象型 `MyDependency` によって実装されます。

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Services/MyDependency.cs?name=snippet1)]

サンプル アプリにおいて、`IMyDependency` サービスは具象型 `MyDependency` を使用して登録されます。 <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddScoped%2A> メソッドによって、サービスは、1 つの要求の有効期間であるスコープ付き有効期間で登録されます。 [サービスの有効期間](#service-lifetimes)については、このトピックの後半で説明します。

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/StartupMyDependency.cs?name=snippet1)]

サンプル アプリでは、`IMyDependency` サービスが要求され、`WriteMessage` メソッドを呼び出すために使用されます。

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Pages/Index2.cshtml.cs?name=snippet1)]

DI パターンを使用すると、コントローラーは次のようになります。

* 具象型 `MyDependency` は使用されず、実装される `IMyDependency` インターフェイスのみが使用されます。 これにより、コントローラーが使用する実装は、コントローラーを変更することなく、簡単に変更できるようになります。
* `MyDependency` のインスタンスは作成されず、DI コンテナーによって作成されます。

組み込みのログ API を使用すると、`IMyDependency` インターフェイスの実装を向上させることができます。

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Services/MyDependency.cs?name=snippet2)]

更新された `ConfigureServices` メソッドでは、新しい `IMyDependency` の実装が登録されます。

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/StartupMyDependency2.cs?name=snippet1)]

`MyDependency2` は、コンストラクターで要求される <xref:Microsoft.Extensions.Logging.ILogger%601> によって異なります。 `ILogger<TCategoryName>` は、[フレームワークで提供されるサービス](#framework-provided-services)です。

依存関係の挿入をチェーン形式で使用することはよくあります。 次に、要求されたそれぞれの依存関係が、それ自身の依存関係を要求します。 コンテナーによってグラフ内の依存関係が解決され、完全に解決されたサービスが返されます。 解決する必要がある依存関係の集合的なセットは、通常、"*依存関係ツリー*"、"*依存関係グラフ*"、または "*オブジェクト グラフ*" と呼ばれます。

コンテナーでは、[(ジェネリック) オープン型](/dotnet/csharp/language-reference/language-specification/types#open-and-closed-types)を活用し、すべての [(ジェネリック) 構築型](/dotnet/csharp/language-reference/language-specification/types#constructed-types)を登録する必要をなくすことで、`ILogger<TCategoryName>` を解決します。

依存関係の挿入に関する用語では、サービスは次のようになります。

* 通常、他のオブジェクト (`IMyDependency` サービスなど) にサービスを提供するオブジェクトです。
* Web サービスを使用する場合はありますが、サービスは Web サービスに関連付けられていません。

フレームワークは、堅牢な [ログ](xref:fundamentals/logging/index) システムを備えています。 前の例に示されている `IMyDependency` の実装は、ログを実装するのではなく、基本的な DI を実演するために記述されています。 ほとんどのアプリでは、ロガーを記述する必要はありません。 次のコードでは、既定のログを使用する方法が示されます。この場合、`ConfigureServices` にサービスを登録する必要はありません。

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Pages/About.cshtml.cs?name=snippet)]

前のコードを使用すると、[ログ](xref:fundamentals/logging/index) がフレームワークによって提供されるため、`ConfigureServices` を更新する必要はありません。

## <a name="services-injected-into-startup"></a>Startup に挿入されるサービス

サービスは、`Startup` コンストラクターと `Startup.Configure` メソッドに挿入できます。

汎用ホスト (<xref:Microsoft.Extensions.Hosting.IHostBuilder>) を使用すると、次のサービスのみを `Startup` コンストラクターに挿入できます。

* <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment>
* <xref:Microsoft.Extensions.Hosting.IHostEnvironment>
* <xref:Microsoft.Extensions.Configuration.IConfiguration>

DI コンテナーに登録されているすべてのサービスを、`Startup.Configure` メソッドに挿入できます。

```csharp
public void Configure(IApplicationBuilder app, ILogger<Startup> logger)
{
    ...
}
```

詳細については、「<xref:fundamentals/startup>」および「[起動時の構成へのアクセス](xref:fundamentals/configuration/index#access-configuration-in-startup)」を参照してください。

## <a name="register-groups-of-services-with-extension-methods"></a>拡張メソッドを使用したサービスのグループを登録する

ASP.NET Core フレームワークは、関連するサービスのグループを登録するための規則を使用します。 規則は、単一の `Add{GROUP_NAME}` 拡張メソッドを使用して、フレームワーク機能に必要なすべてのサービスを登録するというものです。 たとえば、<xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddControllers%2A> 拡張メソッドには、MVC コントローラーに必要なサービスが登録されます。

次のコードは、個々のユーザー アカウントを使用して Razor ページ テンプレートに基づいて生成されており、拡張メソッド <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> および <xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionUIExtensions.AddDefaultIdentity%2A> を使用してコンテナーにさらにサービスを追加する方法を示しています。

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/StartupEF.cs?name=snippet)]

[!INCLUDE[](~/includes/combine-di.md)]

## <a name="service-lifetimes"></a>サービスの有効期間

「[.NET での依存関係の挿入](/dotnet/core/extensions/dependency-injection)」の「[サービスの有効期間](/dotnet/core/extensions/dependency-injection#service-lifetimes)」を参照してください

ミドルウェアでスコープ付きサービスを使用するには、次のいずれかの方法を使用します。

* ミドルウェアの `Invoke` または `InvokeAsync` メソッドにサービスを挿入します。 [コンストラクターの挿入](xref:mvc/controllers/dependency-injection#constructor-injection)を使用すると、スコープ付きサービスがシングルトンのように動作するように強制されるため、実行時の例外がスローされます。 「[有効期間と登録のオプション](#lifetime-and-registration-options)」セクションにあるサンプルは、`InvokeAsync` による方法を示しています。
* [ファクトリ ベースのミドルウェア](xref:fundamentals/middleware/extensibility)を使用します。 この方法を使用して登録されたミドルウェアは、クライアント要求 (接続) ごとにアクティブ化されます。これにより、スコープ付きサービスをミドルウェアの `InvokeAsync` メソッドに挿入できるようになります。

詳細については、「<xref:fundamentals/middleware/write#per-request-middleware-dependencies>」を参照してください。

## <a name="service-registration-methods"></a>サービス登録メソッド

「[.NET での依存関係の挿入](/dotnet/core/extensions/dependency-injection)」の「[サービス登録メソッド](/dotnet/core/extensions/dependency-injection#service-registration-methods)」を参照してください

 [テスト用に型のモックを作成](xref:test/integration-tests#inject-mock-services)する場合に、複数の実装を使用することは一般的です。

実装型のみでサービスを登録することは、同じ実装とサービスの型でそのサービスを登録することと同じです。 明示的なサービス型を使用しないメソッドを使用してサービスの複数の実装を登録できないのは、このためです。 これらのメソッドでは、サービスの複数の "*インスタンス*" を登録できますが、すべて同じ "*実装*" 型になります。

上記のサービス登録メソッドのずれかを使用して、同じサービス型の複数のサービス インスタンスを登録できます。 次の例では、`IMyDependency` をサービス型として使用して、`AddSingleton` を 2 回呼び出します。 2 回目の `AddSingleton` の呼び出しにより、`IMyDependency` として解決された場合は前のものがオーバーライドされ、`IEnumerable<IMyDependency>` を介して複数のサービスが解決された場合は前のものに追加されます。 `IEnumerable<{SERVICE}>` を介して解決された場合、サービスは登録された順に表示されます。

```csharp
services.AddSingleton<IMyDependency, MyDependency>();
services.AddSingleton<IMyDependency, DifferentDependency>();

public class MyService
{
    public MyService(IMyDependency myDependency, 
       IEnumerable<IMyDependency> myDependencies)
    {
        Trace.Assert(myDependency is DifferentDependency);

        var dependencyArray = myDependencies.ToArray();
        Trace.Assert(dependencyArray[0] is MyDependency);
        Trace.Assert(dependencyArray[1] is DifferentDependency);
    }
}
```

## <a name="constructor-injection-behavior"></a>コンストラクターの挿入の動作

「[.NET での依存関係の挿入](/dotnet/core/extensions/dependency-injection)」の「[コンストラクターの挿入の動作](/dotnet/core/extensions/dependency-injection#constructor-injection-behavior)」を参照してください

## <a name="entity-framework-contexts"></a>Entity Framework コンテキスト

既定の Entity Framework コンテキストは、[スコープ付き有効期間](#service-lifetimes)を使用してサービス コンテナーに追加されます。これは、Web アプリ データベース操作は通常、そのスコープがクライアント要求に設定されるためです。 異なる有効期間を使用するには、<xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> のオーバーロードを使用して有効期間を指定します。 有効期間が与えられたサービスの場合、そのサービスより有効期間が短いデータベース コンテキストを使用できません。

## <a name="lifetime-and-registration-options"></a>有効期間と登録のオプション

サービスの有効期間とその登録のオプションの違いを示すために、タスクを識別子 `OperationId` を備えた操作として表す、次のインターフェイスについて考えます。 次のインターフェイスに対して操作のサービスの有効期間がどのように構成されているかに応じて、コンテナーからは、クラスによって要求されたときに、サービスの同じインスタンスか別のインスタンスが提供されます。

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Interfaces/IOperation.cs?name=snippet1)]

次の `Operation` クラスでは、上記のすべてのインターフェイスが実装されます。 `Operation` コンストラクターによって GUID が生成され、`OperationId` プロパティの最後の 4 文字が格納されます。

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Models/Operation.cs?name=snippet1)]

`Startup.ConfigureServices` メソッドでは、指定された有効期間に従って、`Operation` クラスの複数の登録が作成されます。

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Startup2.cs?name=snippet1)]

サンプル アプリでは、それぞれの要求内、およびそれぞれの要求間におけるオブジェクトの有効期間が示されます。 `IndexModel` とミドルウェアでは、すべての種類の `IOperation` 型が要求され、それぞれの `OperationId` がログに記録されます。

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Pages/Index.cshtml.cs?name=snippet1)]

ミドルウェアは `IndexModel` に類似していて、同じサービスを解決します。

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Middleware/MyMiddleware.cs?name=snippet)]

スコープ付きのサービスは `InvokeAsync` メソッドで解決する必要があります。

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Middleware/MyMiddleware.cs?name=snippet2&highlight=2)]

ロガーの出力は次のようになります。

* "*一時的な*" オブジェクトは常に異なります。 一時的な `OperationId` 値は、`IndexModel` とミドルウェアでは異なります。
* "*スコープ付きの*" オブジェクトは、それぞれの要求内では同じですが、それぞれの要求間では異なります。
* "*シングルトン*" オブジェクトは、すべての要求において同じです。

ログ出力を減らすには、*appsettings.Development.json* ファイル内で "Logging:LogLevel:Microsoft:Error" を設定します。

[!code-json[](dependency-injection/samples/3.x/DependencyInjectionSample/appsettings.Development.json?highlight=7)]

## <a name="call-services-from-main"></a>main からサービスを呼び出す

[IServiceScopeFactory.CreateScope](xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory.CreateScope%2A) を使用して <xref:Microsoft.Extensions.DependencyInjection.IServiceScope> を作成し、アプリのスコープ内のスコープ サービスを解決します。 この方法は、起動時に初期化タスクを実行するために、スコープ サービスにアクセスするのに便利です。

次の例は、スコープ付きの `IMyDependency` サービスにアクセスし、`Program.Main` でその `WriteMessage` メソッドを呼び出す方法を示します。

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Program.cs?name=snippet)]

<a name="sv"></a>

## <a name="scope-validation"></a>スコープの検証

「[.NET での依存関係の挿入](/dotnet/core/extensions/dependency-injection)」の「[コンストラクターの挿入の動作](/dotnet/core/extensions/dependency-injection#constructor-injection-behavior)」を参照してください

詳しくは、「[スコープの検証](xref:fundamentals/host/web-host#scope-validation)」をご覧ください。

## <a name="request-services"></a>要求サービス

ASP.NET Core 要求内で使用可能なサービスは、[HttpContext.RequestServices](xref:Microsoft.AspNetCore.Http.HttpContext.RequestServices) コレクションを通じて公開されます。 要求内からサービスが要求されると、サービスとその依存関係は `RequestServices` コレクションから解決されます。

フレームワークでは要求ごとにスコープが作成され、`RequestServices` によってスコープ付きサービス プロバイダーが公開されます。 すべてのスコープ サービスは、要求がアクティブである限り有効です。

> [!NOTE]
> `RequestServices` コレクションからサービスを解決するよりも、コンストラクターのパラメーターとして依存関係を要求するようにします。 これにより、テストしやすいクラスが生成されます。

## <a name="design-services-for-dependency-injection"></a>依存関係の挿入のためのサービスの設計

依存関係の挿入のためのサービスの設計時には:

* ステートフル、静的クラス、およびメンバーは避けてください。 代わりにシングルトン サービスを使用するようにアプリを設計し、グローバルな状態を作成しないようにします。
* サービス内部で依存関係のあるクラスを直接インスタンス化することを回避します。 直接のインスタンス化は、コードの固有の実装につながります。
* サービスを、小さく、十分に要素に分割された、テストしやすいものにします。

クラスに含まれる挿入される依存関係が多すぎる場合、それは、クラスの責任が多すぎて、[単一責任の原則 (SRP)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#single-responsibility) に違反しているサインである可能性があります。 責任の一部を新しいクラスに移動することにより、クラスのリファクタリングを試みます。 Razor Pages のページ モデル クラスと MVC コントローラー クラスは、UI の問題に集中する必要があることに留意します。

### <a name="disposal-of-services"></a>サービスの破棄

コンテナーは、作成する <xref:System.IDisposable> 型の <xref:System.IDisposable.Dispose%2A> を呼び出します。 コンテナーから解決されたサービスが、開発者によって破棄されることはありません。 型またはファクトリがシングルトンとして登録されている場合、コンテナーによってシングルトンが自動的に破棄されます。

次の例では、サービスがサービス コンテナーによって作成され、自動的に破棄されます。

[!code-csharp[](dependency-injection/samples/3.x/DIsample2/DIsample2/Services/Service1.cs?name=snippet)]

[!code-csharp[](dependency-injection/samples/3.x/DIsample2/DIsample2/Startup.cs?name=snippet)]

[!code-csharp[](dependency-injection/samples/3.x/DIsample2/DIsample2/Pages/Index.cshtml.cs?name=snippet)]

デバッグ コンソールでは、インデックス ページを更新するたびに次の出力が表示されます。

```console
Service1: IndexModel.OnGet
Service2: IndexModel.OnGet
Service3: IndexModel.OnGet
Service1.Dispose
```

### <a name="services-not-created-by-the-service-container"></a>サービス コンテナーによって作成されていないサービス

次のコードがあるとします。

[!code-csharp[](dependency-injection/samples/3.x/DIsample2/DIsample2/Startup2.cs?name=snippet)]

上のコードでは以下の操作が行われます。

* サービス インスタンスは、サービス コンテナーによって作成されるわけではありません。
* フレームワークでは、サービスが自動的に破棄されることはありません。
* サービスを破棄する責任は開発者にあります。

### <a name="idisposable-guidance-for-transient-and-shared-instances"></a>一時的なインスタンスと共有インスタンスのための IDisposable ガイダンス

「[.NET での依存関係の挿入](/dotnet/core/extensions/dependency-injection)」の「[一時的なインスタンスと共有インスタンスのための IDisposable ガイダンス](/dotnet/core/extensions/dependency-injection-guidelines#idisposable-guidance-for-transient-and-shared-instances)」を参照してください

## <a name="default-service-container-replacement"></a>既定のサービス コンテナーの置換

「[.NET での依存関係の挿入](/dotnet/core/extensions/dependency-injection)」の「[既定のサービス コンテナーの置換](/dotnet/core/extensions/dependency-injection-guidelines#default-service-container-replacement)」を参照してください

## <a name="recommendations"></a>Recommendations

「[.NET での依存関係の挿入](/dotnet/core/extensions/dependency-injection)」の「[推奨事項](/dotnet/core/extensions/dependency-injection-guidelines#recommendations)」を参照してください

* *サービス ロケーター パターン* の使用は避けてください。 たとえば、サービス インスタンスを取得する場合、DI を使用できるときに、<xref:System.IServiceProvider.GetService%2A> を呼び出さないでください。

  **正しくない:**

    ![正しくないコード](dependency-injection/_static/bad.png)

  **正しい**:

  ```csharp
  public class MyClass
  {
      private readonly IOptionsMonitor<MyOptions> _optionsMonitor;

      public MyClass(IOptionsMonitor<MyOptions> optionsMonitor)
      {
          _optionsMonitor = optionsMonitor;
      }

      public void MyMethod()
      {
          var option = _optionsMonitor.CurrentValue.Option;

          ...
      }
  }
  ```
* 回避すべき別のサービス ロケーターのバリエーションは、実行時に依存関係を解決するファクトリを挿入することです。 この両方のプラクティスによって、複数の[制御の反転](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion)方式が組み合わせられます。
* `HttpContext` への静的なアクセスを回避します (たとえば [IHttpContextAccessor.HttpContext](xref:Microsoft.AspNetCore.Http.IHttpContextAccessor.HttpContext))。

<a name="ASP0000"></a>
* `ConfigureServices` で <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionContainerBuilderExtensions.BuildServiceProvider%2A> を呼び出すことは避けてください。 `BuildServiceProvider` の呼び出しは、通常、開発者が `ConfigureServices` でサービスを解決する必要がある場合に発生します。 たとえば、構成から `LoginPath` が読み込まれる場合を考えてみます。 次の方法は避けてください。

  ![BuildServiceProvider を呼び出すコードが正しくありません](~/fundamentals/dependency-injection/_static/badcodeX.png)

  上の図では、`services.BuildServiceProvider` の下の緑の波線を選択すると、次の ASP0000 警告が表示されます。

  > アプリケーション コードから ASP0000 呼び出し 'BuildServiceProvider' を行うと、シングルトン サービスの追加のコピーが作成されます。 'Configure' のパラメーターとして依存関係挿入サービスなどの代替手段を検討してください。

  `BuildServiceProvider` を呼び出すと、2 つ目のコンテナーが作成されます。これを使用すれば、破損したシングルトンが作成され、複数のコンテナーにまたがるオブジェクト グラフへの参照を生じさせることができます。

  `LoginPath` を取得する正しい方法は、DI のオプション パターンの組み込みサポートを使用することです。

  [!code-csharp[](dependency-injection/samples/3.x/AntiPattern3/Startup.cs?name=snippet)]

* 破棄可能な一時的なサービスは、破棄のためにコンテナーによってキャプチャされます。 これにより、最上位のコンテナーから解決された場合、メモリ リークが発生する可能性があります。
* スコープの検証を有効にすることで、アプリにスコープ付きサービスをキャプチャするシングルトンがないようにします。 詳しくは、「[スコープの検証](#scope-validation)」をご覧ください。

どのような推奨事項であっても、それを無視する必要がある状況が発生する可能性があります。 例外はまれです。ほとんどがフレームワーク自体の内の特殊なケースです。

DI は静的/グローバル オブジェクト アクセス パターンの *代替機能* です。 静的オブジェクト アクセスと併用した場合、DI のメリットを実現することはできません。

## <a name="recommended-patterns-for-multi-tenancy-in-di"></a>DI でのマルチテナントの推奨パターン

[Orchard Core](https://github.com/OrchardCMS/OrchardCore) は、ASP.NET Core でモジュール型のマルチテナント アプリケーションを構築するためのアプリケーション フレームワークです。 詳細については、[Orchard Core のドキュメント](https://docs.orchardcore.net/en/dev/)を参照してください。

CMS 固有の機能を使用せずに Orchard Core Framework のみを使用してモジュール型のマルチテナント アプリを構築する方法の例については、[Orchard Core のサンプル](https://github.com/OrchardCMS/OrchardCore.Samples)を参照してください。

## <a name="framework-provided-services"></a>フレームワークが提供するサービス

`Startup.ConfigureServices` メソッドでは、Entity Framework Core や ASP.NET Core MVC のようなプラットフォーム機能など、アプリが使うサービスが登録されます。 最初に、`ConfigureServices` に提供される `IServiceCollection` には、フレームワークによって定義されたサービスがあります ([ホストの構成方法](xref:fundamentals/index#host)によって異なります)。 ASP.NET Core テンプレートに基づくアプリでは、フレームワークによって 250 を超えるサービスが登録されます。 

次の表に、フレームワークによって登録されるサービスのごく一部を示します。

| サービスの種類                                                                                    | 有効期間  |
|-------------------------------------------------------------------------------------------------|-----------|
| <xref:Microsoft.AspNetCore.Hosting.Builder.IApplicationBuilderFactory?displayProperty=fullName> | 一時的 |
| <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime>                                    | シングルトン |
| <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment>                                         | シングルトン |
| <xref:Microsoft.AspNetCore.Hosting.IStartup?displayProperty=fullName>                           | シングルトン |
| <xref:Microsoft.AspNetCore.Hosting.IStartupFilter?displayProperty=fullName>                     | 一時的 |
| <xref:Microsoft.AspNetCore.Hosting.Server.IServer?displayProperty=fullName>                     | シングルトン |
| <xref:Microsoft.AspNetCore.Http.IHttpContextFactory?displayProperty=fullName>                   | 一時的 |
| <xref:Microsoft.Extensions.Logging.ILogger%601?displayProperty=fullName>                        | シングルトン |
| <xref:Microsoft.Extensions.Logging.ILoggerFactory?displayProperty=fullName>                     | シングルトン |
| <xref:Microsoft.Extensions.ObjectPool.ObjectPoolProvider?displayProperty=fullName>              | シングルトン |
| <xref:Microsoft.Extensions.Options.IConfigureOptions%601?displayProperty=fullName>              | 一時的 |
| <xref:Microsoft.Extensions.Options.IOptions%601?displayProperty=fullName>                       | シングルトン |
| <xref:System.Diagnostics.DiagnosticSource?displayProperty=fullName>                             | シングルトン |
| <xref:System.Diagnostics.DiagnosticListener?displayProperty=fullName>                           | シングルトン |

## <a name="additional-resources"></a>その他の技術情報

* <xref:mvc/views/dependency-injection>
* <xref:mvc/controllers/dependency-injection>
* <xref:security/authorization/dependencyinjection>
* <xref:blazor/fundamentals/dependency-injection>
* [NDC カンファレンス DI アプリ開発のパターン](https://www.youtube.com/watch?v=x-C-CNBVTaY)
* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/extensibility>
* [ASP.NET Core で IDisposables を破棄する 4 つの方法](https://andrewlock.net/four-ways-to-dispose-idisposables-in-asp-net-core/)
* [依存関係の挿入による ASP.NET Core でのクリーンなコードの作成 (MSDN)](/archive/msdn-magazine/2016/may/asp-net-writing-clean-code-in-asp-net-core-with-dependency-injection)
* [明示的な依存関係の原則](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies)
* [制御の反転コンテナーと依存関係の挿入パターン (Martin Fowler)](https://www.martinfowler.com/articles/injection.html)
* [ASP.NET Core DI 内の複数のインターフェイスにサービスを登録する方法](https://andrewlock.net/how-to-register-a-service-with-multiple-interfaces-for-in-asp-net-core-di/)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作成者: [Steve Smith](https://ardalis.com/)、[Scott Addie](https://scottaddie.com)、[Brandon Dahler](https://github.com/brandondahler)

ASP.NET Core では依存関係の挿入 (DI) ソフトウェア設計パターンがサポートされています。これは、クラスとその依存関係の間で[制御の反転 (IoC)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) を実現するための手法です。

MVC コントローラー内部における依存関係の挿入に固有の情報については、<xref:mvc/controllers/dependency-injection> をご覧ください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/dependency-injection/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="overview-of-dependency-injection"></a>依存関係の挿入の概要

"*依存関係*" とは、他のオブジェクトが必要とする任意のオブジェクトのことです。 アプリ内の他のクラスが依存している、次の `WriteMessage` メソッドを備えた `MyDependency` クラスを調べます。

```csharp
public class MyDependency
{
    public MyDependency()
    {
    }

    public Task WriteMessage(string message)
    {
        Console.WriteLine(
            $"MyDependency.WriteMessage called. Message: {message}");

        return Task.FromResult(0);
    }
}
```

クラスで `WriteMessage` メソッドを使用できるようにするために、`MyDependency` クラスのインスタンスを作成することができます。 `MyDependency` クラスは `IndexModel` クラスの依存関係です。

```csharp
public class IndexModel : PageModel
{
    MyDependency _dependency = new MyDependency();

    public async Task OnGetAsync()
    {
        await _dependency.WriteMessage(
            "IndexModel.OnGetAsync created this message.");
    }
}
```

このクラスは `MyDependency` のインスタンスを作成し、これに直接依存しています。 コードの依存関係 (前の例など) には問題が含まれ、次の理由から回避する必要があります。

* `MyDependency` を別の実装で置き換えるには、クラスを変更する必要があります。
* `MyDependency` が依存関係を含んでいる場合、これらはクラスによって構成する必要があります。 複数のクラスが `MyDependency` に依存している大規模なプロジェクトでは、構成コードがアプリ全体に分散するようになります。
* このような実装では、単体テストを行うことが困難です。 アプリはモックまたはスタブの `MyDependency` クラスを使用する必要がありますが、この方法では不可能です。

依存関係の挿入は、次の方法によってこれらの問題に対応します。

* 依存関係の実装を抽象化するための、インターフェイスまたは基底クラスの使用。
* サービス コンテナー内の依存関係の登録。 ASP.NET Core には、組み込みのサービス コンテナー <xref:System.IServiceProvider> が用意されています。 サービスはアプリの `Startup.ConfigureServices` メソッドに登録されています。
* サービスを使用するクラスのコンストラクターへの、サービスの "*挿入*"。 依存関係のインスタンスの作成、およびインスタンスが不要になったときの廃棄の役割を、フレームワークが担当します。

[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/dependency-injection/samples)では、サービスがアプリに提供するメソッドが `IMyDependency` インターフェイスによって定義されます。

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Interfaces/IMyDependency.cs?name=snippet1)]

このインターフェイスは、具象型 `MyDependency` によって実装されます。

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Services/MyDependency.cs?name=snippet1)]

`MyDependency` では、コンストラクター内で <xref:Microsoft.Extensions.Logging.ILogger`1> が要求されます。 依存関係の挿入をチェーン形式で使用することはよくあります。 次に、要求されたそれぞれの依存関係が、それ自身の依存関係を要求します。 コンテナーによってグラフ内の依存関係が解決され、完全に解決されたサービスが返されます。 解決する必要がある依存関係の集合的なセットは、通常、"*依存関係ツリー*"、"*依存関係グラフ*"、または "*オブジェクト グラフ*" と呼ばれます。

`IMyDependency` と `ILogger<TCategoryName>` をサービス コンテナーに登録する必要があります。 `IMyDependency` は `Startup.ConfigureServices` に登録されます。 `ILogger<TCategoryName>` はログ記録の抽象化インフラストラクチャによって登録されます。したがって、これは、フレームワークによって既定で登録される[フレームワークが提供するサービス](#framework-provided-services)です。

コンテナーでは、[(ジェネリック) オープン型](/dotnet/csharp/language-reference/language-specification/types#open-and-closed-types)を活用し、すべての [(ジェネリック) 構築型](/dotnet/csharp/language-reference/language-specification/types#constructed-types)を登録する必要をなくすことで、`ILogger<TCategoryName>` を解決します。

```csharp
services.AddSingleton(typeof(ILogger<>), typeof(Logger<>));
```

サンプル アプリにおいて、`IMyDependency` サービスは `MyDependency` 具象型を使用して登録されます。 登録によって、サービスの有効期間が 1 つの要求の有効期間に範囲設定されます。 [サービスの有効期間](#service-lifetimes)については、このトピックの後半で説明します。

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Startup.cs?name=snippet1&highlight=5)]

> [!NOTE]
> 各 `services.Add{SERVICE_NAME}` 拡張メソッドによって、サービスが追加、場合によっては構成されます。 たとえば、`services.AddControllersWithViews`、`services.AddRazorPages`、`services.AddControllers` を使用すると、ASP.NET Core アプリに必要なサービスが追加されます。 アプリをこの規則に従わせることをお勧めします。 拡張メソッドを <xref:Microsoft.Extensions.DependencyInjection?displayProperty=fullName> 名前空間に配置して、サービス登録のグループをカプセル化します。 DI 拡張メソッド用の名前空間部分として `Microsoft.Extensions.DependencyInjection` を含めると、次のようになります。
>
> * `using` ブロックを追加しなくても、それらを [IntelliSense](/visualstudio/ide/using-intellisense) で表示することができます。
> * それらの拡張メソッドの通常の呼び出し元である `Startup` クラス内で `using` ステートメントが過剰になることはありません。

サービスのコンストラクターで[ビルトイン型](/dotnet/csharp/language-reference/keywords/built-in-types-table) (`string` など) が必要な場合は、[構成](xref:fundamentals/configuration/index)や[オプション パターン](xref:fundamentals/configuration/options)を使って型を挿入することができます。

```csharp
public class MyDependency : IMyDependency
{
    public MyDependency(IConfiguration config)
    {
        var myStringValue = config["MyStringKey"];

        // Use myStringValue
    }

    ...
}
```

クラスのコンストラクター経由でサービスのインスタンスが要求されます。サービスはプライベート フィールドに割り当てられて使用されます。 このフィールドは、クラス全体で必要に応じてサービスにアクセスするために使用されます。

サンプル アプリでは、`IMyDependency` インスタンスが要求され、サービスの `WriteMessage` メソッドを呼び出すために使用されます。

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Pages/Index.cshtml.cs?name=snippet1&highlight=3,6,13,29-30)]

## <a name="services-injected-into-startup"></a>Startup に挿入されるサービス

汎用ホスト (<xref:Microsoft.Extensions.Hosting.IHostBuilder>) を使用すると、次のサービスの種類のみを `Startup` コンストラクターに挿入できます。

* `IWebHostEnvironment`
* <xref:Microsoft.Extensions.Hosting.IHostEnvironment>
* <xref:Microsoft.Extensions.Configuration.IConfiguration>

サービスは `Startup.Configure` に挿入できます。

```csharp
public void Configure(IApplicationBuilder app, IOptions<MyOptions> options)
{
    ...
}
```

詳細については、「<xref:fundamentals/startup>」を参照してください。

## <a name="framework-provided-services"></a>フレームワークが提供するサービス

`Startup.ConfigureServices` メソッドでは、Entity Framework Core や ASP.NET Core MVC のようなプラットフォーム機能など、アプリが使うサービスを定義する必要があります。 最初に、`ConfigureServices` に提供される `IServiceCollection` には、フレームワークによって定義されたサービスがあります ([ホストの構成方法](xref:fundamentals/index#host)によって異なります)。 ASP.NET Core テンプレートに基づくアプリが、フレームワークによって登録された何百ものサービスを持つことも珍しくありません。 次の表に、フレームワークによって登録されたサービスのごく一部のサンプルを示します。

| サービスの種類 | 有効期間 |
| ------------ | -------- |
| <xref:Microsoft.AspNetCore.Hosting.Builder.IApplicationBuilderFactory?displayProperty=fullName> | 一時的 |
| <xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime?displayProperty=fullName> | シングルトン |
| <xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment?displayProperty=fullName> | シングルトン |
| <xref:Microsoft.AspNetCore.Hosting.IStartup?displayProperty=fullName> | シングルトン |
| <xref:Microsoft.AspNetCore.Hosting.IStartupFilter?displayProperty=fullName> | 一時的 |
| <xref:Microsoft.AspNetCore.Hosting.Server.IServer?displayProperty=fullName> | シングルトン |
| <xref:Microsoft.AspNetCore.Http.IHttpContextFactory?displayProperty=fullName> | 一時的 |
| <xref:Microsoft.Extensions.Logging.ILogger`1?displayProperty=fullName> | シングルトン |
| <xref:Microsoft.Extensions.Logging.ILoggerFactory?displayProperty=fullName> | シングルトン |
| <xref:Microsoft.Extensions.ObjectPool.ObjectPoolProvider?displayProperty=fullName> | シングルトン |
| <xref:Microsoft.Extensions.Options.IConfigureOptions`1?displayProperty=fullName> | 一時的 |
| <xref:Microsoft.Extensions.Options.IOptions`1?displayProperty=fullName> | シングルトン |
| <xref:System.Diagnostics.DiagnosticSource?displayProperty=fullName> | シングルトン |
| <xref:System.Diagnostics.DiagnosticListener?displayProperty=fullName> | シングルトン |

## <a name="register-additional-services-with-extension-methods"></a>拡張メソッドを使用した追加サービスの登録する

サービス (および必要であればサービスが依存するサービス) を登録するためにサービス コレクションの拡張メソッドを使用できる場合は、1 つの `Add{SERVICE_NAME}` 拡張メソッドを使用してそのサービスが必要とするすべてのサービスを登録することが規則です。 次のコードは、拡張メソッド [AddDbContext\<TContext>](/dotnet/api/microsoft.extensions.dependencyinjection.entityframeworkservicecollectionextensions.adddbcontext) と <xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionExtensions.AddIdentityCore*> を使用して、コンテナーに追加のサービスを追加する方法の例です。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...

    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

    services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

    ...
}
```

詳細については、API ドキュメントにある <xref:Microsoft.Extensions.DependencyInjection.ServiceCollection> クラスを参照してください。

## <a name="service-lifetimes"></a>サービスの有効期間

登録される各サービスの適切な有効期間を選択します。 ASP.NET Core サービスは、次の有効期間で構成できます。

### <a name="transient"></a>一時的

有効期間が一時的なサービス (<xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddTransient*>) は、サービス コンテナーから要求されるたびに作成されます。 この有効期間は、軽量でステートレスのサービスに最適です。

要求を処理するアプリでは、一時的なサービスが要求の最後に破棄されます。

### <a name="scoped"></a>スコープ

有効期間がスコープのサービス (<xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddScoped*>) は、クライアント要求 (接続) ごとに 1 回作成されます。

要求を処理するアプリでは、スコープ付きサービスは要求の最後で破棄されます。

> [!WARNING]
> ミドルウェアでスコープ サービスを使用している場合、サービスを `Invoke` または `InvokeAsync` メソッドに追加します。 コンストラクターを使用して挿入すると、サービスがシングルトンのように動作するよう強制されるので、[コンストラクターの挿入](xref:mvc/controllers/dependency-injection#constructor-injection)を使用した挿入は行わないでください。 詳細については、「<xref:fundamentals/middleware/write#per-request-middleware-dependencies>」を参照してください。

### <a name="singleton"></a>シングルトン

有効期間がシングルトンのサービス (<xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddSingleton*>) は、最初に要求されたときに作成されます (または、`Startup.ConfigureServices` が実行されて、サービス登録でインスタンスが指定された場合)。 以降の要求は、すべて同じインスタンスを使用します。 アプリをシングルトンで動作させる必要がある場合は、サービス コンテナーによるサービスの有効期間の管理を許可することをお勧めします。 クラス内のオブジェクトの有効期間を管理するために、シングルトンの設計パターンを実装してユーザー コードを提供しないでください。

要求を処理するアプリでは、<xref:Microsoft.Extensions.DependencyInjection.ServiceProvider> がアプリのシャットダウン時に破棄されたとき、シングルトン サービスが破棄されます。

> [!WARNING]
> シングルトンからスコープ サービスを解決するのは危険です。 後続の要求を処理する際に、サービスが正しくない状態になる可能性があります。

## <a name="service-registration-methods"></a>サービス登録メソッド

サービス登録拡張メソッドでは、特定のシナリオで役立つオーバーロードが提供されます。

| メソッド | 自動<br>object<br>破棄 | 複数<br>実装 | 引数を渡す |
| ------ | :-----------------------------: | :-------------------------: | :-------: |
| `Add{LIFETIME}<{SERVICE}, {IMPLEMENTATION}>()`<br>例:<br>`services.AddSingleton<IMyDep, MyDep>();` | はい | はい | いいえ |
| `Add{LIFETIME}<{SERVICE}>(sp => new {IMPLEMENTATION})`<br>次に例を示します。<br>`services.AddSingleton<IMyDep>(sp => new MyDep());`<br>`services.AddSingleton<IMyDep>(sp => new MyDep("A string!"));` | はい | はい | はい |
| `Add{LIFETIME}<{IMPLEMENTATION}>()`<br>例:<br>`services.AddSingleton<MyDep>();` | はい | いいえ | いいえ |
| `AddSingleton<{SERVICE}>(new {IMPLEMENTATION})`<br>次に例を示します。<br>`services.AddSingleton<IMyDep>(new MyDep());`<br>`services.AddSingleton<IMyDep>(new MyDep("A string!"));` | いいえ | はい | はい |
| `AddSingleton(new {IMPLEMENTATION})`<br>次に例を示します。<br>`services.AddSingleton(new MyDep());`<br>`services.AddSingleton(new MyDep("A string!"));` | いいえ | いいえ | はい |

型の廃棄の詳細については、「[サービスの破棄](#disposal-of-services)」を参照してください。 実装が複数の場合の一般的なシナリオとしては、[テスト用に型のモックを作成](xref:test/integration-tests#inject-mock-services)します。

実装型のみでサービスを登録することは、同じ実装とサービスの型でそのサービスを登録することと同じです。 明示的なサービス型を使用しないメソッドを使用してサービスの複数の実装を登録できないのは、このためです。 これらのメソッドでは、サービスの複数の "*インスタンス*" を登録できますが、すべて同じ "*実装*" 型になります。

上記のサービス登録メソッドのずれかを使用して、同じサービス型の複数のサービス インスタンスを登録できます。 次の例では、`IMyDependency` をサービス型として使用して、`AddSingleton` を 2 回呼び出します。 2 回目の `AddSingleton` の呼び出しにより、`IMyDependency` として解決された場合は前のものがオーバーライドされ、`IEnumerable<IMyDependency>` を介して複数のサービスが解決された場合は前のものに追加されます。 `IEnumerable<{SERVICE}>` を介して解決された場合、サービスは登録された順に表示されます。

```csharp
services.AddSingleton<IMyDependency, MyDependency>();
services.AddSingleton<IMyDependency, DifferentDependency>();

public class MyService
{
    public MyService(IMyDependency myDependency, 
       IEnumerable<IMyDependency> myDependencies)
    {
        Trace.Assert(myDependency is DifferentDependency);

        var dependencyArray = myDependencies.ToArray();
        Trace.Assert(dependencyArray[0] is MyDependency);
        Trace.Assert(dependencyArray[1] is DifferentDependency);
    }
}
```

フレームワークには `TryAdd{LIFETIME}` 拡張メソッドも用意されており、実装がまだ登録されていない場合にのみ、サービスが登録されます。

次の例では、`AddSingleton` の呼び出しによって、`IMyDependency` の実装として `MyDependency` が登録されます。 `IMyDependency` には登録された実装が既に含まれているため、`TryAddSingleton` の呼び出しでは何も行われません。

```csharp
services.AddSingleton<IMyDependency, MyDependency>();
// The following line has no effect:
services.TryAddSingleton<IMyDependency, DifferentDependency>();

public class MyService
{
    public MyService(IMyDependency myDependency, 
        IEnumerable<IMyDependency> myDependencies)
    {
        Trace.Assert(myDependency is MyDependency);
        Trace.Assert(myDependencies.Single() is MyDependency);
    }
}
```

詳細については次を参照してください:

* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAdd*>
* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddTransient*>
* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddScoped*>
* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddSingleton*>

[TryAddEnumerable(ServiceDescriptor)](xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddEnumerable*) メソッドでは、"*同じ型*" の実装がまだ存在しない場合にのみサービスが登録されます。 複数のサービスは、`IEnumerable<{SERVICE}>` によって解決されます。 サービスを登録するとき、開発者は同じ型のものがまだ追加されていない場合にのみインスタンスを追加します。 一般に、このメソッドは、インスタンスのコピーが 2 つコンテナーに登録されるのを回避するために、ライブラリ作成者によって使用されます。

次の例の最初の行では、`IMyDep1` に `MyDep` を登録します。 2 行目では `IMyDep2` に `MyDep` を登録します。 3 行目では何も行われません。`IMyDep1` には `MyDep` の登録済みの実装が既に含まれているからです。

```csharp
public interface IMyDep1 {}
public interface IMyDep2 {}

public class MyDep : IMyDep1, IMyDep2 {}

services.TryAddEnumerable(ServiceDescriptor.Singleton<IMyDep1, MyDep>());
services.TryAddEnumerable(ServiceDescriptor.Singleton<IMyDep2, MyDep>());
// Two registrations of MyDep for IMyDep1 is avoided by the following line:
services.TryAddEnumerable(ServiceDescriptor.Singleton<IMyDep1, MyDep>());
```

### <a name="constructor-injection-behavior"></a>コンストラクターの挿入の動作

サービスは 2 つのメカニズムによって解決できます。

* <xref:System.IServiceProvider>
* <xref:Microsoft.Extensions.DependencyInjection.ActivatorUtilities>:依存関係の挿入コンテナーにサービスを登録することなくオブジェクトの作成を許可します。 `ActivatorUtilities` はユーザー側の抽象化 (タグ ヘルパー、MVC コントローラー、モデル バインダーなど) で使用されます。

コンストラクターは、依存関係の挿入によって提供されない引数を受け取ることができますが、引数は既定値を割り当てる必要があります。

`IServiceProvider` または `ActivatorUtilities` によってサービスを解決する場合、[コンストラクターの挿入](xref:mvc/controllers/dependency-injection#constructor-injection)には、"*パブリック*" コンストラクターが必要です。

`ActivatorUtilities` によってサービスを解決する場合、[コンストラクターの挿入](xref:mvc/controllers/dependency-injection#constructor-injection)には、該当するコンストラクターが 1 つだけ存在することが必要です。 コンストラクターのオーバーロードはサポートされていますが、依存関係の挿入によってすべての引数を設定できるオーバーロードは 1 つしか存在できません。

## <a name="entity-framework-contexts"></a>Entity Framework コンテキスト

Entity Framework コンテキストでは通常、[範囲が指定された有効期間](#service-lifetimes)が利用され、サービス コンテナーに追加されます。これは、Web アプリ データベース操作は通常、その範囲がクライアント要求に設定されるためです。 データベース コンテキストの登録時、[AddDbContext\<TContext>](/dotnet/api/microsoft.extensions.dependencyinjection.entityframeworkservicecollectionextensions.adddbcontext) オーバーロードによって有効期間が指定されなかった場合、既定の有効期間が範囲となります。 有効期間が与えられたサービスの場合、サービスより有効期間が短いデータベース コンテキストを使用できません。

## <a name="lifetime-and-registration-options"></a>有効期間と登録のオプション

有効期間と登録のオプションの違いを示すために、タスクを一意の識別子 `OperationId` を備えた操作として表す、次のインターフェイスについて考えます。 次のインターフェイスに対して操作のサービスの有効期間がどのように構成されているかに応じて、コンテナーは、クラスに要求されたときに、サービスの同じインスタンスか別のインスタンスを提供します。

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Interfaces/IOperation.cs?name=snippet1)]

インターフェイスは `Operation` クラス内で実装されます。 指定されない場合、`Operation` コンストラクターは GUID を生成します。

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Models/Operation.cs?name=snippet1)]

`OperationService` が登録されます。これは、その他の `Operation` 型のそれぞれに依存します。 依存関係の挿入を経由して `OperationService` が要求される場合、これは各サービスの新しいインスタンスか、依存関係のあるサービスの有効期間に基づく既存のインスタンスを受信します。

* コンテナーからの要求時に一時的なサービスが作成されるとき、`IOperationTransient` サービスの `OperationId` と `OperationService` の `OperationId` は異なります。 `OperationService` は `IOperationTransient` クラスの新しいインスタンスを受け取ります。 新しいインスタンスは異なる `OperationId` を生成します。
* クライアント要求ごとにスコープ サービスが作成されるとき、`IOperationScoped` サービスの `OperationId` は、クライアント要求内の `OperationService` のものと同じになります。 クライアント要求間で、両方のサービスは異なる `OperationId` 値を共有します。
* シングルトンとシングルトン インスタンスのサービスが 1 回作成され、すべてのクライアント要求とすべてのサービス間で使用されるとき、`OperationId` はすべてのサービス要求の間で一定です。

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Services/OperationService.cs?name=snippet1)]

`Startup.ConfigureServices` では、各型が名前で指定されている有効期間に従ってコンテナーに追加されます。

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Startup.cs?name=snippet1&highlight=6-9,12)]

`IOperationSingletonInstance` サービスは、`Guid.Empty` の既知の ID を持った特定のインスタンスを使用しています。 この型が使用されるタイミングは明らかです (その GUID はすべてゼロです)。

サンプル アプリでは、個々の要求内、および個々の要求間におけるオブジェクトの有効期間が示されます。 サンプル アプリの `IndexModel` には、各種類の `IOperation` 型と `OperationService` が必要です。 次に、ページ モデル クラスとサービスのすべての `OperationId` 値が、プロパティの割り当てを通じてページに表示されます。

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Pages/Index.cshtml.cs?name=snippet1&highlight=7-11,14-18,21-25)]

2 つの要求の結果を、以下の 2 つの出力に示します。

**最初の要求:**

コントローラーの操作:

一時的: d233e165-f417-469b-a866-1cf1935d2518  
スコープ:5d997e2d-55f5-4a64-8388-51c4e3a1ad19  
シングルトン:01271bc1-9e31-48e7-8f7c-7261b040ded9  
インスタンス:00000000-0000-0000-0000-000000000000

`OperationService` の操作:

一時的: c6b049eb-1318-4e31-90f1-eb2dd849ff64  
スコープ:5d997e2d-55f5-4a64-8388-51c4e3a1ad19  
シングルトン:01271bc1-9e31-48e7-8f7c-7261b040ded9  
インスタンス:00000000-0000-0000-0000-000000000000

**2 番目の要求:**

コントローラーの操作:

一時的: b63bd538-0a37-4ff1-90ba-081c5138dda0  
スコープ:31e820c5-4834-4d22-83fc-a60118acb9f4  
シングルトン:01271bc1-9e31-48e7-8f7c-7261b040ded9  
インスタンス:00000000-0000-0000-0000-000000000000

`OperationService` の操作:

一時的: c4cbacb8-36a2-436d-81c8-8c1b78808aaf  
スコープ:31e820c5-4834-4d22-83fc-a60118acb9f4  
シングルトン:01271bc1-9e31-48e7-8f7c-7261b040ded9  
インスタンス:00000000-0000-0000-0000-000000000000

要求内および要求間で、どの `OperationId` 値が変化しているかを確認してください。

* "*一時的な*" オブジェクトは常に異なります。 1 番目と 2 番目のクライアント要求の一時的な `OperationId` 値は、`OperationService` 操作とクライアント要求間の両方に対して異なります。 個々のサービス要求とクライアント要求に対して、新しいインスタンスが提供されます。
* *Scoped* オブジェクトは、1 つのクライアント要求内では同じですが、複数のクライアント要求間では異なります。
* "*シングルトン*" オブジェクトは、`Operation` インスタンスが `Startup.ConfigureServices` で提供されるかどうかに関係なく、すべてのオブジェクトとすべての要求について同じです。

## <a name="call-services-from-main"></a>main からサービスを呼び出す

[IServiceScopeFactory.CreateScope](xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory.CreateScope*) を使用して <xref:Microsoft.Extensions.DependencyInjection.IServiceScope> を作成し、アプリのスコープ内のスコープ サービスを解決します。 この方法は、起動時に初期化タスクを実行するために、スコープ サービスにアクセスするのに便利です。 次の例では、`Program.Main` で `MyScopedService` のコンテキストを取得する方法を示します。

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;

public class Program
{
    public static async Task Main(string[] args)
    {
        var host = CreateWebHostBuilder(args).Build();

        using (var serviceScope = host.Services.CreateScope())
        {
            var services = serviceScope.ServiceProvider;

            try
            {
                var serviceContext = services.GetRequiredService<MyScopedService>();
                // Use the context here
            }
            catch (Exception ex)
            {
                var logger = services.GetRequiredService<ILogger<Program>>();
                logger.LogError(ex, "An error occurred.");
            }
        }
    
        await host.RunAsync();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>();
}
```

## <a name="scope-validation"></a>スコープの検証

アプリが開発環境で実行されている場合、既定のサービス プロバイダーは次を確認するためのチェックを実行します。

* スコープ サービスが、ルート サービス プロバイダーによって直接的または間接的に解決されない。
* スコープ サービスが、シングルトンに直接または間接に挿入されない。

<xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionContainerBuilderExtensions.BuildServiceProvider*> が呼び出されると、ルート サービス プロバイダーが作成されます。 ルート サービス プロバイダーの有効期間は、プロバイダーがアプリで開始されるとアプリとサーバーの有効期間に対応し、アプリのシャットダウンには破棄されます。

スコープ サービスは、それを作成したコンテナーによって破棄されます。 ルート コンテナーにスコープ サービスが作成されると、サービスはアプリ/サーバーのシャットダウン時に、ルート コンテナーによってのみ破棄されるため、サービスの有効期間は実質的にシングルトンに昇格されます。 `BuildServiceProvider` が呼び出されると、サービス スコープの検証がこれらの状況をキャッチします。

詳細については、「<xref:fundamentals/host/web-host#scope-validation>」を参照してください。   

## <a name="request-services"></a>要求サービス

`HttpContext` からの ASP.NET Core 要求内で使用可能なサービスは、[HttpContext.RequestServices](xref:Microsoft.AspNetCore.Http.HttpContext.RequestServices) コレクションを通じて公開されます。

要求サービスは、アプリの一部として構成および要求されるサービスを表します。 オブジェクトで依存関係を指定すると、これらは `ApplicationServices` ではなく `RequestServices` で検出された型で満たされます。

一般に、アプリから直接これらのプロパティを使用しないでください。 代わりに、クラスのコンストラクターを介してクラスに必要な型を要求し、フレームワークに依存関係を挿入させます。 これにより、テストしやすいクラスが生成されます。

> [!NOTE]
> コンストラクターのパラメーターとして依存関係を要求し、`RequestServices` コレクションにアクセスするようにします。

## <a name="design-services-for-dependency-injection"></a>依存関係の挿入のためのサービスの設計

ベスト プラクティスは次のとおりです。

* 各依存関係を取得するために依存関係の挿入を使用するようサービスを設計します。
* ステートフル、静的クラス、およびメンバーは避けてください。 代わりにシングルトン サービスを使用するようにアプリを設計し、グローバルな状態を作成しないようにします。
* サービス内部で依存関係のあるクラスを直接インスタンス化することを回避します。 直接のインスタンス化は、コードの固有の実装につながります。
* アプリのクラスを、小さく、十分に要素に分割された、テストしやすいものにします。

クラスに含まれる挿入される依存関係が多すぎるように見える場合、これは通常、クラスが担当する役割が多すぎて、[単一責任の原則 (SRP)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#single-responsibility) に違反していることのサインです。 責任の一部を新しいクラスに移動することにより、クラスのリファクタリングを試みます。 Razor Pages のページ モデル クラスと MVC コントローラー クラスは、UI の問題に集中する必要があることに留意します。 ビジネス ルールとデータ アクセスの実装の詳細は、これらの[個別の問題](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns)に適したクラスに分離する必要があります。

### <a name="disposal-of-services"></a>サービスの破棄

コンテナーは、作成する <xref:System.IDisposable> 型の <xref:System.IDisposable.Dispose*> を呼び出します。 ユーザー コードによってコンテナーに追加されたインスタンスは、自動的に破棄されません。

次の例では、サービスがサービス コンテナーによって作成され、自動的に破棄されます。

```csharp
public class Service1 : IDisposable {}
public class Service2 : IDisposable {}

public interface IService3 {}
public class Service3 : IService3, IDisposable {}

public void ConfigureServices(IServiceCollection services)
{
    services.AddScoped<Service1>();
    services.AddSingleton<Service2>();
    services.AddSingleton<IService3>(sp => new Service3());
}
```

次に例を示します。

* サービス インスタンスは、サービス コンテナーによって作成されるわけではありません。
* サービスの有効期間は、フレームワークによって認識されません。
* フレームワークでは、サービスが自動的に破棄されることはありません。
* サービスが開発者コードで明示的に破棄されていない場合は、アプリがシャットダウンするまで保持されます。

```csharp
public class Service1 : IDisposable {}
public class Service2 : IDisposable {}

public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<Service1>(new Service1());
    services.AddSingleton(new Service2());
}
```

### <a name="idisposable-guidance-for-transient-and-shared-instances"></a>一時的なインスタンスと共有インスタンスのための IDisposable ガイダンス

#### <a name="transient-limited-lifetime"></a>一時的で限定的な有効期間

**シナリオ**

アプリには、次のいずれかのシナリオに対して一時的な有効期間を持つ <xref:System.IDisposable> インスタンスが必要です。

* インスタンスは、ルート スコープで解決されます。
* インスタンスは、スコープが終了する前に破棄する必要があります。

**解決方法**

ファクトリ パターンを使用して、親スコープの外部にインスタンスを作成します。 このような状況では、通常、アプリケーションには、最終的な型のコンストラクターを直接呼び出す `Create` メソッドがあります。 最終的な型に他の依存関係がある場合、ファクトリは次のことができます。

* そのコンストラクターで <xref:System.IServiceProvider> を受け取る。
* <xref:Microsoft.Extensions.DependencyInjection.ActivatorUtilities.CreateInstance%2A?displayProperty=nameWithType> を使用してコンテナー外部でインスタンスをインスタンス化し、同時にコンテナーを依存関係のために使用する。

#### <a name="shared-instance-limited-lifetime"></a>共有インスタンス、限定的な有効期間

**シナリオ**

アプリでは、複数のサービスにわたって共有 <xref:System.IDisposable> インスタンスが必要ですが、<xref:System.IDisposable> の有効期間は限られています。

**解決方法**

インスタンスをスコープ付きの有効期間に登録します。 <xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory.CreateScope%2A?displayProperty=nameWithType> を使用してを開始し、新しい <xref:Microsoft.Extensions.DependencyInjection.IServiceScope> を作成します。 スコープの <xref:System.IServiceProvider> を使用して必要なサービスを取得します。 有効期間が終了するときにスコープを破棄します。

#### <a name="general-guidelines"></a>一般的なガイドライン

* <xref:System.IDisposable> インスタンスは、一時的なスコープを使用して登録しないでください。 代わりに、ファクトリ パターンを使用します。
* 一時的またはスコープ付きの <xref:System.IDisposable> インスタンスをルート スコープ内で解決しないでください。 唯一の一般的な例外は、アプリが <xref:System.IServiceProvider> を作成/再作成し、破棄する場合です。これは理想的なパターンではありません。
* DI を使用して <xref:System.IDisposable> 依存関係を受け取る場合、受信側が <xref:System.IDisposable> 自体を実装する必要はありません。 <xref:System.IDisposable> 依存関係の受信側は、その依存関係で <xref:System.IDisposable.Dispose%2A> を呼び出してはなりません。
* サービスの有効期間を制御するには、スコープを使用する必要があります。 スコープは階層構造ではなく、スコープ間に特別な接続はありません。

## <a name="default-service-container-replacement"></a>既定のサービス コンテナーの置換

組み込みのサービス コンテナーは、フレームワークと、ほとんどのコンシューマー アプリのニーズに対応することを目的としたものです。 組み込みコンテナーでサポートされない、以下のような特定の機能が必要な場合を除き、組み込みコンテナーを使用することをお勧めします。

* プロパティの挿入
* 名前に基づく挿入
* 子コンテナー
* 有効期間のカスタム管理
* 遅延初期化に対する `Func<T>` のサポート
* 規則に基づく登録

次のサードパーティ コンテナーは ASP.NET Core アプリで使用できます。

* [Autofac](https://autofac.readthedocs.io/en/latest/integration/aspnetcore.html)
* [DryIoc](https://www.nuget.org/packages/DryIoc.Microsoft.DependencyInjection)
* [Grace](https://www.nuget.org/packages/Grace.DependencyInjection.Extensions)
* [LightInject](https://github.com/seesharper/LightInject.Microsoft.DependencyInjection)
* [Lamar](https://jasperfx.github.io/lamar/)
* [Stashbox](https://github.com/z4kn4fein/stashbox-extensions-dependencyinjection)
* [Unity](https://www.nuget.org/packages/Unity.Microsoft.DependencyInjection)

### <a name="thread-safety"></a>スレッド セーフ

スレッド セーフのシングルトン サービスを作成します。 シングルトン サービスに一時的サービスへの依存関係がある場合、シングルトンによる一時的サービスの使い方によっては、一時的サービスもスレッド セーフであることが必要な場合があります。

1 つのサービスのファクトリ メソッド (例: [AddSingleton\<TService>(IServiceCollection, Func\<IServiceProvider,TService>)](xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddSingleton*) に対する 2 番目の引数) をスレッド セーフにする必要はありません。 型 (`static`) のコンストラクターのように、1 つのスレッドによって 1 回呼び出されることが保証されます。

## <a name="recommendations"></a>推奨事項

* `async/await` および `Task` ベースのサービスの解決はサポートされていません。 C# では非同期コンストラクターがサポートされていないため、推奨パターンは、サービスを同期的に解決した後に、非同期メソッドを使用することです。

* データと構成をサービス コンテナーに直接格納しないようにします。 たとえば、通常、ユーザーのショッピング カートはサービス コンテナーに追加しません。 構成では、[オプション パターン](xref:fundamentals/configuration/options)を使う必要があります。 同様に、他のオブジェクトへのアクセスを許可するためだけに存在する "データ ホルダー" オブジェクトは避ける必要があります。 実際のアイテムを DI 経由で要求することをお勧めします。
* サービスへの静的なアクセスを行わないようにします。 たとえば、他の場所で使用するために [IApplicationBuilder.ApplicationServices](xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.ApplicationServices) の静的な型指定を行わないようにします。

* *サービス ロケーター パターン* は、[制御の反転](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion)戦略が混在しているので使用しないでください。
  * サービス インスタンスを取得する場合、DI を使用できるときに、<xref:System.IServiceProvider.GetService*> を呼び出さないでください。

    **正しくない:**

    ```csharp
    public class MyClass()
   
      public void MyMethod()
      {
          var optionsMonitor = 
              _services.GetService<IOptionsMonitor<MyOptions>>();
          var option = optionsMonitor.CurrentValue.Option;
   
          ...
      }
      ```
   
    **正しい**:

    ```csharp
    public class MyClass
    {
        private readonly IOptionsMonitor<MyOptions> _optionsMonitor;

        public MyClass(IOptionsMonitor<MyOptions> optionsMonitor)
        {
            _optionsMonitor = optionsMonitor;
        }

        public void MyMethod()
        {
            var option = _optionsMonitor.CurrentValue.Option;

            ...
        }
    }
    ```

* <xref:System.IServiceProvider.GetService*> を使用して、実行時に依存関係を解決するファクトリを挿入しないでください。
* `HttpContext` への静的なアクセスを回避します (たとえば [IHttpContextAccessor.HttpContext](xref:Microsoft.AspNetCore.Http.IHttpContextAccessor.HttpContext))。

どのような推奨事項であっても、それを無視する必要がある状況が発生する可能性があります。 例外はまれです。ほとんどがフレームワーク自体の内の特殊なケースです。

DI は静的/グローバル オブジェクト アクセス パターンの *代替機能* です。 静的オブジェクト アクセスと併用した場合、DI のメリットを実現することはできません。

## <a name="additional-resources"></a>その他の技術情報

* <xref:mvc/views/dependency-injection>
* <xref:mvc/controllers/dependency-injection>
* <xref:security/authorization/dependencyinjection>
* <xref:blazor/fundamentals/dependency-injection>
* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/extensibility>
* [ASP.NET Core で IDisposables を破棄する 4 つの方法](https://andrewlock.net/four-ways-to-dispose-idisposables-in-asp-net-core/)
* [依存関係の挿入による ASP.NET Core でのクリーンなコードの作成 (MSDN)](/archive/msdn-magazine/2016/may/asp-net-writing-clean-code-in-asp-net-core-with-dependency-injection)
* [明示的な依存関係の原則](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies)
* [制御の反転コンテナーと依存関係の挿入パターン (Martin Fowler)](https://www.martinfowler.com/articles/injection.html)
* [ASP.NET Core DI 内の複数のインターフェイスにサービスを登録する方法](https://andrewlock.net/how-to-register-a-service-with-multiple-interfaces-for-in-asp-net-core-di/)

::: moniker-end
