---
title: ASP.NET Core のアプリケーション モデルの使用
author: rick-anderson
description: アプリケーションを読み、操作し、ASP.NET Core での MVC 要素の動作を変更する方法について説明します。
ms.author: riande
ms.date: 04/05/2021
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
uid: mvc/controllers/application-model
ms.openlocfilehash: 8772a432f137fd6d0278208963bee03f6e8a715b
ms.sourcegitcommit: 0abfe496fed8e9470037c8128efa8a50069ccd52
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/07/2021
ms.locfileid: "106564045"
---
# <a name="work-with-the-application-model-in-aspnet-core"></a>ASP.NET Core のアプリケーション モデルの使用

作成者: [Steve Smith](https://ardalis.com/)

ASP.NET Core MVC では、MVC アプリのコンポーネントを表す *アプリケーション モデル* を定義できます。 MVC 要素の動作を変更するには、このモデルを読み取り、操作します。 既定では、MVC は特定の規則に従って、コントローラーと見なされるクラス、それらのクラスのメソッドはアクション、およびパラメーターとルーティングの動作を決定します。 カスタム規則を作成してグローバルまたは属性として適用することで、アプリのニーズに合わせてこの動作をカスタマイズします。

## <a name="models-and-providers-iapplicationmodelprovider"></a>モデルとプロバイダー ( `IApplicationModelProvider` )

ASP.NET Core MVC アプリケーションモデルには、MVC アプリケーションを記述する抽象インターフェイスと具象実装クラスの両方が含まれています。 このモデルは、既定の規則に従ってアプリのコントローラー、アクション、アクション パラメーター、ルート、およびフィルターを検出する MVC の結果です。 アプリケーションモデルを使用して、既定の MVC 動作とは異なる規則に従うようにアプリを変更します。 パラメーター、名前、ルート、およびフィルターは、すべてアクションおよびコントローラーの構成データとして使用されます。

ASP.NET Core MVC アプリケーション モデルの構造は、次のとおりです。

* ApplicationModel
  * コントローラー (ControllerModel)
    * アクション (ActionModel)
      * パラメーター (ParameterModel)

このモデルでは、各レベルで、共通の `Properties` コレクションにアクセスできます。下位レベルでは、階層の上位レベルで設定されたプロパティ値にアクセスしたり上書きしたりできます。 このプロパティは、アクションの作成時、<xref:Microsoft.AspNetCore.Mvc.Abstractions.ActionDescriptor.Properties?displayProperty=nameWithType> に保存されます。 そして要求の処理時に、規則によって追加または変更されたすべてのプロパティに、<xref:Microsoft.AspNetCore.Mvc.ActionContext.ActionDescriptor?displayProperty=nameWithType> を介してアクセスできます。 プロパティの使用は、アクションごとにフィルター、モデルバインダー、およびその他のアプリモデルの側面を構成するための優れた方法です。

> [!NOTE]
> コレクションは、 <xref:Microsoft.AspNetCore.Mvc.Abstractions.ActionDescriptor.Properties?displayProperty=nameWithType> アプリの起動後にスレッドセーフ (書き込み用) ではありません。 このコレクションにデータを安全に追加するには、規則が最善の方法です。

MVC ASP.NET Core は、インターフェイスで定義されているプロバイダーパターンを使用して、アプリケーションモデルを読み込み <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IApplicationModelProvider> ます。 このセクションでは、このプロバイダーがどのように機能するかについての、いくつかの内部実装に関する詳細を説明します。 プロバイダーパターンの使用は高度なテーマであり、主にフレームワークで使用されます。 ほとんどのアプリでは、プロバイダーパターンではなく、規約を使用する必要があります。

インターフェイスの実装 <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IApplicationModelProvider> では、各実装が <xref:Microsoft.AspNetCore.Mvc.Abstractions.IActionInvokerProvider.OnProvidersExecuting%2A> そのプロパティに基づいて昇順にを呼び出して、相互にラップし <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IApplicationModelProvider.Order> ます。 次いで、<xref:Microsoft.AspNetCore.Mvc.Abstractions.IActionInvokerProvider.OnProvidersExecuted%2A> メソッドが逆順で呼び出されます。 このフレームワークでは、次のいくつかのプロバイダーが定義されます。

1 番目 (`Order=-1000`):

* `DefaultApplicationModelProvider`

次 (`Order=-990`):

* `AuthorizationApplicationModelProvider`
* `CorsApplicationModelProvider`

> [!NOTE]
> の同じ値を持つ2つのプロバイダーが呼び出される順序は定義されていないため、 `Order` 依存することはできません。

> [!NOTE]
> <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IApplicationModelProvider> は、フレームワークの作成者が拡張する高度な概念です。 一般に、アプリは規約を使用する必要があり、フレームワークではプロバイダーを使用する必要があります。 重要な違いは、プロバイダーは常に規則の前に実行されるということです。

`DefaultApplicationModelProvider` は ASP.NET Core MVC で使用される多数の既定の動作を確立します。 次の役割があります。

* コンテキストにグローバル フィルターを追加する
* コンテキストにコントローラーを追加する
* アクションとしてパブリック コントローラー メソッドを追加する
* コンテキストにアクション メソッド パラメーターを追加する
* ルートおよびその他の属性を適用する

いくつかの組み込みの動作は、`DefaultApplicationModelProvider` によって実装されます。 このプロバイダーは、、、およびの各インスタンスを参照するを構築する役割を担い <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.ControllerModel> <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.ActionModel> <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PropertyModel> <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.ParameterModel> ます。 クラスは、 `DefaultApplicationModelProvider` 今後変更される可能性がある内部フレームワークの実装の詳細です。

`AuthorizationApplicationModelProvider` は、<xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> 属性および <xref:Microsoft.AspNetCore.Mvc.Authorization.AllowAnonymousFilter> 属性に関連付けられた動作を適用します。 詳細については、「<xref:security/authorization/simple>」を参照してください。

は、 `CorsApplicationModelProvider` およびに関連付けられている動作を実装し <xref:Microsoft.AspNetCore.Cors.Infrastructure.IEnableCorsAttribute> <xref:Microsoft.AspNetCore.Cors.Infrastructure.IDisableCorsAttribute> ます。 詳細については、「<xref:security/cors>」を参照してください。

このセクションで説明するフレームワークの内部プロバイダーに関する情報は、 [.NET API ブラウザー](https://docs.microsoft.com/dotnet/api/)では使用できません。 ただし、プロバイダーは [ASP.NET Core 参照ソース (dotnet/Aspnetcore GitHub リポジトリ)](https://github.com/dotnet/aspnetcore)で検査できます。 GitHub 検索を使用して名前でプロバイダーを検索し、[ **ブランチ/タグの切り替え** ] ボックスの一覧でソースのバージョンを選択します。

## <a name="conventions"></a>規約

このアプリケーション モデルでは、モデルまたはプロバイダー全体をオーバーライドするよりも簡単に、モデルの動作をカスタマイズできる、規則の抽象化を定義できます。 これらの抽象化は、アプリの動作を変更するために推奨される方法です。 規則は、カスタマイズを動的に適用するコードを記述する方法を提供します。 [フィルター](xref:mvc/controllers/filters)はフレームワークの動作を変更する手段を提供しますが、カスタマイズによって、アプリ全体の動作を制御できます。

次の規則があります。

* <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IApplicationModelConvention>
* <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IControllerModelConvention>
* <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IActionModelConvention>
* <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IParameterModelConvention>

規則を適用するには、それらを MVC オプションに追加するか、属性を実装してコントローラー、アクション、またはアクションパラメーターに適用します ( [フィルター](xref:mvc/controllers/filters)に似ています)。フィルターとは異なり、規則は、各要求の一部としてではなく、アプリの開始時にのみ実行されます。

> [!NOTE]
> Razorページルートとアプリケーションモデルプロバイダーの規則の詳細については、「」を参照してください <xref:razor-pages/razor-pages-conventions> 。

## <a name="modify-the-applicationmodel"></a>を変更します。 `ApplicationModel`

アプリケーションモデルにプロパティを追加するには、次の規則を使用します。

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ApplicationDescription.cs)]

アプリケーションモデルの規則は、MVC がに追加されたときのオプションとして適用され `Startup.ConfigureServices` ます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Startup.cs?name=ConfigureServices&highlight=5)]

プロパティは、 <xref:Microsoft.AspNetCore.Mvc.Abstractions.ActionDescriptor.Properties?displayProperty=nameWithType> コントローラーアクション内のコレクションからアクセスできます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/AppModelController.cs?name=AppModelController)]

## <a name="modify-the-controllermodel-description"></a>説明を変更する `ControllerModel`

コントローラーモデルには、カスタムプロパティを含めることもできます。 カスタムプロパティは、アプリケーションモデルで指定されている名前と同じ名前を持つ既存のプロパティをオーバーライドします。 次の規則属性では、コントローラー レベルで説明が追加されます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ControllerDescriptionAttribute.cs)]

この規則は、コントローラーの属性として適用されます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/DescriptionAttributesController.cs?name=ControllerDescription&highlight=1)]

## <a name="modify-the-actionmodel-description"></a>説明を変更する `ActionModel`

個別のアクションには、アプリケーションレベルまたはコントローラーレベルで既に適用されている動作をオーバーライドすることによって、個別の属性規則を適用できます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ActionDescriptionAttribute.cs)]

コントローラー内のアクションにこれを適用すると、コントローラーレベルの規則がどのようにオーバーライドされるかが示されます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/DescriptionAttributesController.cs?name=DescriptionAttributesController&highlight=9)]

## <a name="modify-the-parametermodel"></a>を変更します。 `ParameterModel`

次の規則をアクション パラメーターに適用して、<xref:Microsoft.AspNetCore.Mvc.ModelBinding.BindingInfo> を変更することができます。 次の規則では、パラメーターがルートパラメーターである必要があります。 クエリ文字列値など、その他の潜在的なバインドソースは無視されます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/MustBeInRouteParameterModelConvention.cs)]

この属性は、任意のアクション パラメーターに適用できます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/ParameterModelController.cs?name=ParameterModelController&highlight=5)]

すべてのアクションパラメーターに規則を適用するには、の `MustBeInRouteParameterModelConvention` をに追加し <xref:Microsoft.AspNetCore.Mvc.MvcOptions> `Startup.ConfigureServices` ます。

```csharp
options.Conventions.Add(new MustBeInRouteParameterModelConvention());
```

## <a name="modify-the-actionmodel-name"></a>名前を変更する `ActionModel`

次の規則は、<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.ActionModel> が適用されるアクションの *名前* を更新してそれを変更します。 この新しい名前は、属性にパラメーターとして提供されます。 この新しい名前はルーティングによって使用されるため、このアクションメソッドに到達するために使用されるルートに影響します。

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/CustomActionNameAttribute.cs)]

この属性は、`HomeController` のアクション メソッドに適用されます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/HomeController.cs?name=ActionModelConvention&highlight=2)]

メソッド名は `SomeName` ですが、このメソッド名を使用する MVC 規則をこの属性はオーバーライドし、アクション名を `MyCoolAction` に置換します。 したがって、このアクションに到達するのに使用されるルートは、`/Home/MyCoolAction` です。

> [!NOTE]
> このセクションのこの例は、基本的に組み込みを使用する場合と同じです <xref:Microsoft.AspNetCore.Mvc.ActionNameAttribute> 。

## <a name="custom-routing-convention"></a>カスタムルーティング規則

を使用して、 <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.IApplicationModelConvention> ルーティングの動作をカスタマイズします。 たとえば、次の規則では、コントローラーの名前空間をルートに組み込み、 `.` 名前空間内のを `/` ルートに置き換えます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/NamespaceRoutingConvention.cs)]

規則は、のオプションとして追加され `Startup.ConfigureServices` ます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Startup.cs?name=ConfigureServices&highlight=6)]

> [!TIP]
> [](xref:fundamentals/middleware/index) <xref:Microsoft.AspNetCore.Mvc.MvcOptions> 次の方法を使用して、ミドルウェアに規則を追加します。 `{CONVENTION}`プレースホルダーは、次のように追加する規則です。
>
> ```csharp
> services.Configure<MvcOptions>(c => c.Conventions.Add({CONVENTION}));
> ```

次の例では、コントローラーの名前に属性ルーティングを使用していないルートに規則を適用し  `Namespace` ます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/NamespaceRoutingController.cs?highlight=7-8)]

::: moniker range="<= aspnetcore-2.2"

## <a name="application-model-usage-in-webapicompatshim"></a>でのアプリケーションモデルの使用 `WebApiCompatShim`

ASP.NET Core MVC と ASP.NET Web API 2 とでは使用する規則のセットが異なります。 カスタム規則を使用すると、web API アプリの動作と一貫性を持つように ASP.NET Core MVC アプリの動作を変更できます。 Microsoft は、この目的のために特別な[ `WebApiCompatShim` NuGet パッケージ](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim)を提供しています。

> [!NOTE]
> ASP.NET Web API からの移行の詳細については、「」を参照してください <xref:migration/webapi> 。

Web API 互換性 Shim を使用するには、次のようにします。

* `Microsoft.AspNetCore.Mvc.WebApiCompatShim`パッケージをプロジェクトに追加します。
* でを呼び出して、MVC に規則を追加し <xref:Microsoft.Extensions.DependencyInjection.WebApiCompatShimMvcBuilderExtensions.AddWebApiConventions%2A> `Startup.ConfigureServices` ます。

```csharp
services.AddMvc().AddWebApiConventions();
```

Shim が提供するこの規則は、それに特定の属性が適用されたアプリの一部にのみ適用されます。 次の 4 つの属性は、Shim の規則でどのコントローラーが規則を変更する必要があるかを制御するために使用されます。

* <xref:Microsoft.AspNetCore.Mvc.WebApiCompatShim.UseWebApiActionConventionsAttribute>
* <xref:Microsoft.AspNetCore.Mvc.WebApiCompatShim.UseWebApiOverloadingAttribute>
* <xref:Microsoft.AspNetCore.Mvc.WebApiCompatShim.UseWebApiParameterConventionsAttribute>
* <xref:Microsoft.AspNetCore.Mvc.WebApiCompatShim.UseWebApiRoutesAttribute>

### <a name="action-conventions"></a>アクション規則

<xref:Microsoft.AspNetCore.Mvc.WebApiCompatShim.UseWebApiActionConventionsAttribute> は、HTTP メソッドを名前に基づいてアクションにマップするために使用されます (たとえば、は `Get` にマップされ `HttpGet` ます)。 これは、属性のルーティングを使用しないアクションにのみ適用されます。

### <a name="overloading"></a>オーバーロード

<xref:Microsoft.AspNetCore.Mvc.WebApiCompatShim.UseWebApiOverloadingAttribute> は、規則を適用するために使用され <xref:Microsoft.AspNetCore.Mvc.WebApiCompatShim.WebApiOverloadingApplicationModelConvention> ます。 この規則は、アクションの選択プロセスに <xref:Microsoft.AspNetCore.Mvc.WebApiCompatShim.OverloadActionConstraint> を追加します。これによって、候補のアクションは、要求で省略可能でないすべてのパラメーターが満たされるものに制限されます。

### <a name="parameter-conventions"></a>パラメーター規則

<xref:Microsoft.AspNetCore.Mvc.WebApiCompatShim.UseWebApiParameterConventionsAttribute> は、アクション規則を適用するために使用され <xref:Microsoft.AspNetCore.Mvc.WebApiCompatShim.WebApiParameterConventionsApplicationModelConvention> ます。 この規則では、アクション パラメーターとして使用される単純な型は、要求本文からバインドされる複雑な型に対し、既定で URI からバインドされることが指定されます。

### <a name="routes"></a>ルート

<xref:Microsoft.AspNetCore.Mvc.WebApiCompatShim.UseWebApiRoutesAttribute> コントローラー規則を適用するかどうか `WebApiApplicationModelConvention` を制御します。 有効にすると、この規則は、 [区分](xref:mvc/controllers/areas) のサポートをルートに追加するために使用され、その領域にコントローラーがあることを示し `api` ます。

一連の規則に加えて、互換性パッケージには、 <xref:System.Web.Http.ApiController?displayProperty=fullName> WEB API によって提供されるものを置き換える基本クラスが含まれています。 これにより、web API 用に記述された web API コントローラーを `ApiController` ASP.NET CORE MVC で実行しながら、それを継承することができます。 [`UseWebApi*`](xref:Microsoft.AspNetCore.Mvc.WebApiCompatShim)前に示したすべての属性は、基本コントローラークラスに適用されます。 は、 `ApiController` WEB API に含まれるものと互換性のあるプロパティ、メソッド、および結果型を公開します。

::: moniker-end

## <a name="use-apiexplorer-to-document-an-app"></a>`ApiExplorer`アプリをドキュメント化するために使用する

このアプリケーション モデルは、アプリの構造をスキャンするために使用できる <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.ApiExplorerModel> プロパティを各レベルで公開します。 これは、 [Swagger などのツールを使用して Web api のヘルプページを生成](xref:tutorials/web-api-help-pages-using-swagger)するために使用できます。 プロパティは、 `ApiExplorer` <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.ApiExplorerModel.IsVisible> アプリケーションのモデルのどの部分を公開するかを指定するために設定できるプロパティを公開します。 規則を使用して、この設定を構成します。

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/EnableApiExplorerApplicationConvention.cs)]

この方法 (および必要に応じて追加の規則) を使用すると、アプリ内の任意のレベルで API の可視性が有効または無効になります。
