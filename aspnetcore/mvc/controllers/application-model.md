---
title: ASP.NET Core のアプリケーション モデルの使用
author: ardalis
description: アプリケーションを読み、操作し、ASP.NET Core での MVC 要素の動作を変更する方法について説明します。
ms.author: riande
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
uid: mvc/controllers/application-model
ms.openlocfilehash: cf16536284ee9c88913257607df837ad6e50ea2c
ms.sourcegitcommit: 7e394a8527c9818caebb940f692ae4fcf2f1b277
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/31/2021
ms.locfileid: "99217376"
---
# <a name="work-with-the-application-model-in-aspnet-core"></a>ASP.NET Core のアプリケーション モデルの使用

作成者: [Steve Smith](https://ardalis.com/)

ASP.NET Core MVC では、MVC アプリのコンポーネントを表す *アプリケーション モデル* を定義できます。 このモデルを読み、操作し、MVC 要素の動作を変更することができます。 既定で、MVC には、どのクラスがコントローラーとして考慮されるか、それらのクラスのどのメソッドがアクションであるか、パラメーターおよびルーティングがどのように動作するかについて、特定の規則があります。 この動作をアプリのニーズに合うようにカスタマイズして、独自の規則を作成し、それらをグローバルにまたは属性として適用することができます。

## <a name="models-and-providers"></a>モデルおよびプロバイダー

ASP.NET Core MVC アプリケーションモデルには、MVC アプリケーションを記述する抽象インターフェイスと具象実装クラスの両方が含まれています。 このモデルは、既定の規則に従ってアプリのコントローラー、アクション、アクション パラメーター、ルート、およびフィルターを検出する MVC の結果です。 アプリケーション モデルを使用すると、既定の MVC の動作とは異なる規則を使用するようアプリを変更することができます。 パラメーター、名前、ルート、およびフィルターは、すべてアクションおよびコントローラーの構成データとして使用されます。

ASP.NET Core MVC アプリケーション モデルの構造は、次のとおりです。

* ApplicationModel
  * コントローラー (ControllerModel)
    * アクション (ActionModel)
      * パラメーター (ParameterModel)

このモデルでは、各レベルで、共通の `Properties` コレクションにアクセスできます。下位レベルでは、階層の上位レベルで設定されたプロパティ値にアクセスしたり上書きしたりできます。 このプロパティは、アクションの作成時、`ActionDescriptor.Properties` に保存されます。 そして要求の処理時に、規則によって追加または変更されたすべてのプロパティに、`ActionContext.ActionDescriptor.Properties` を介してアクセスできます。 フィルターやモデル バインダーなどをアクションごとに構成するのに、プロパティを使用するのはよい方法です。

> [!NOTE]
> アプリのスタートアップが完了している場合、`ActionDescriptor.Properties` コレクションは (書き込みに) スレッド セーフではありません。 このコレクションにデータを安全に追加するには、規則が最善の方法です。

### <a name="iapplicationmodelprovider"></a>IApplicationModelProvider

ASP.NET Core MVC は、[IApplicationModelProvider](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iapplicationmodelprovider) インターフェイスによって定義されるプロバイダー パターンを使用して、アプリケーション モデルを読み込みます。 このセクションでは、このプロバイダーがどのように機能するかについての、いくつかの内部実装に関する詳細を説明します。 これは高度なトピックです。アプリケーション モデルを活用するアプリのほとんどでは、規則を使用してアプリケーション モデルを活用する必要があります。

`IApplicationModelProvider` インターフェイスの実装は、その `Order` プロパティに応じて、昇順で `OnProvidersExecuting` を呼び出して互いを "ラップ" します。 次いで、`OnProvidersExecuted` メソッドが逆順で呼び出されます。 このフレームワークでは、次のいくつかのプロバイダーが定義されます。

1 番目 (`Order=-1000`):

* [`DefaultApplicationModelProvider`](/dotnet/api/microsoft.aspnetcore.mvc.internal.defaultapplicationmodelprovider)

次 (`Order=-990`):

* [`AuthorizationApplicationModelProvider`](/dotnet/api/microsoft.aspnetcore.mvc.internal.authorizationapplicationmodelprovider)
* [`CorsApplicationModelProvider`](/dotnet/api/microsoft.aspnetcore.mvc.cors.internal.corsapplicationmodelprovider)

> [!NOTE]
> 2 つのプロバイダーの `Order` の値が同じである場合、順序は定義されていないため、これには依存しないようにする必要があります。

> [!NOTE]
> `IApplicationModelProvider` は、フレームワークの作成者が拡張する高度な概念です。 一般に、規則やフレームワークを使う必要があるアプリは、プロバイダーを使う必要があります。 重要な違いは、プロバイダーは常に規則の前に実行されるということです。

`DefaultApplicationModelProvider` は ASP.NET Core MVC で使用される多数の既定の動作を確立します。 次の役割があります。

* コンテキストにグローバル フィルターを追加する
* コンテキストにコントローラーを追加する
* アクションとしてパブリック コントローラー メソッドを追加する
* コンテキストにアクション メソッド パラメーターを追加する
* ルートおよびその他の属性を適用する

いくつかの組み込みの動作は、`DefaultApplicationModelProvider` によって実装されます。 このプロバイダーは、、、およびの各インスタンスを参照するを構築する役割を担い [`ControllerModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.controllermodel) [`ActionModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.actionmodel) [`PropertyModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.propertymodel) [`ParameterModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.parametermodel) ます。 `DefaultApplicationModelProvider` クラスは、今後変更する可能性がある変更される、内部フレームワークの実装についての詳細です。 

`AuthorizationApplicationModelProvider` は、`AuthorizeFilter` 属性および `AllowAnonymousFilter` 属性に関連付けられた動作を適用します。 [これらの属性については、こちらを参照してください](xref:security/authorization/simple)。

`CorsApplicationModelProvider` は、`IEnableCorsAttribute` および `IDisableCorsAttribute` および `DisableCorsAuthorizationFilter` に関連付けられた動作を実装します。 [CORS の詳細については、こちらを参照してください](xref:security/cors).

## <a name="conventions"></a>規則

このアプリケーション モデルでは、モデルまたはプロバイダー全体をオーバーライドするよりも簡単に、モデルの動作をカスタマイズできる、規則の抽象化を定義できます。 これらの抽象化は、アプリの動作の変更に推奨されます。 規則では、動的にカスタマイズすることが可能なコードを記述することができます。 [フィルター](xref:mvc/controllers/filters)では、フレームワークの動作を変更できるのに対して、カスタマイズではアプリ全体がどのように連携するかを制御できます。

次の規則があります。

* [`IApplicationModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iapplicationmodelconvention)
* [`IControllerModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.icontrollermodelconvention)
* [`IActionModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iactionmodelconvention)
* [`IParameterModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iparametermodelconvention)

規則を適用するには、それらを MVC オプションに追加するか、 `Attribute` を実装してコントローラー、アクション、またはアクションパラメーターに適用します (に似て [`Filters`](xref:mvc/controllers/filters) います)。 フィルターとは異なり、規則は、各要求の一部としてではなく、アプリの起動時にのみ実行されます。

### <a name="sample-modifying-the-applicationmodel"></a>サンプル: ApplicationModel を変更する

次の規則は、アプリケーション モデルにプロパティを追加するために使用します。 

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ApplicationDescription.cs)]

アプリケーション モデルの規則は、MVC が `Startup` の `ConfigureServices` に追加されるときに、オプションとしてが適用されます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Startup.cs?name=ConfigureServices&highlight=5)]

プロパティは、コントローラー アクション内の `ActionDescriptor` プロパティ コレクションからアクセスできます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/AppModelController.cs?name=AppModelController)]

### <a name="sample-modifying-the-controllermodel-description"></a>サンプル: ControllerModel の説明を変更する

前の例のように、コントローラー モデルを変更して、カスタム プロパティを含めることもできます。 これらは、アプリケーション モデルで指定した同じ名前の既存のプロパティをオーバーライドします。 次の規則属性では、コントローラー レベルで説明が追加されます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ControllerDescriptionAttribute.cs)]

この規則は、コントローラーの属性として適用されます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/DescriptionAttributesController.cs?name=ControllerDescription&highlight=1)]

"description" プロパティは、前の例と同じ方法でアクセスされます。

### <a name="sample-modifying-the-actionmodel-description"></a>サンプル: ActionModel の説明を変更する

既にアプリケーションまたはコントローラー レベルに適用されている動作をオーバーライドして、個別のアクションに別の属性規則を適用することができます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ActionDescriptionAttribute.cs)]

コントローラーのアクションにこれを適用している前の例は、コントローラー レベルで規則をオーバーライドする方法を示しています。

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/DescriptionAttributesController.cs?name=DescriptionAttributesController&highlight=9)]

### <a name="sample-modifying-the-parametermodel"></a>例: ParameterModel を変更する

次の規則をアクション パラメーターに適用して、`BindingInfo` を変更することができます。 次の規則では、パラメーターはルート パラメーターである必要があります。その他のバインディング ソースとなる可能性のあるものは (クエリ文字列の値など) 無視されます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/MustBeInRouteParameterModelConvention.cs)]

この属性は、任意のアクション パラメーターに適用できます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/ParameterModelController.cs?name=ParameterModelController&highlight=5)]

### <a name="sample-modifying-the-actionmodel-name"></a>Sample: ActionModel 名を変更する

次の規則は、`ActionModel` が適用されるアクションの *名前* を更新してそれを変更します。 この新しい名前は、属性にパラメーターとして提供されます。 この新しい名前はルーティングによって使用されるので、このアクション メソッドに到達するのに使用されるルートに影響します。

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/CustomActionNameAttribute.cs)]

この属性は、`HomeController` のアクション メソッドに適用されます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/HomeController.cs?name=ActionModelConvention&highlight=2)]

メソッド名は `SomeName` ですが、このメソッド名を使用する MVC 規則をこの属性はオーバーライドし、アクション名を `MyCoolAction` に置換します。 したがって、このアクションに到達するのに使用されるルートは、`/Home/MyCoolAction` です。

> [!NOTE]
> この例は、基本的に、組み込みの [ActionName](/dotnet/api/microsoft.aspnetcore.mvc.actionnameattribute) 属性を使用するのと同じです。

### <a name="sample-custom-routing-convention"></a>サンプル: カスタム ルーティング規則

`IApplicationModelConvention` を使用して、ルーティングの動作をカスタマイズできます。 たとえば、次の規則は、名前空間の `.` をルートの `/` に置き換えてコントローラーの名前空間をそのルートに組み込みます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/NamespaceRoutingConvention.cs)]

この規則は、スタートアップのオプションとして追加されています。

[!code-csharp[](./application-model/sample/src/AppModelSample/Startup.cs?name=ConfigureServices&highlight=6)]

> [!TIP]
> `services.Configure<MvcOptions>(c => c.Conventions.Add(YOURCONVENTION));` を使用し、`MvcOptions` にアクセスして、[ミドルウェア](xref:fundamentals/middleware/index)に規則を追加できます。

このサンプルでは、コントローラー名に "Namespace" が含まれる、属性のルーティングを使用していないルートにこの規則は適用されます。 この規則は、次のコントローラーによって使用されます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/NamespaceRoutingController.cs?highlight=7-8)]

## <a name="application-model-usage-in-webapicompatshim"></a>WebApiCompatShim でのアプリケーション モデルの使用法

ASP.NET Core MVC と ASP.NET Web API 2 とでは使用する規則のセットが異なります。 カスタム規則を使用すると、Web API アプリの動作と一致するように、ASP.NET Core MVC アプリの動作を変更できます。 Microsoft では、この目的専用に [WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim/) を出荷しています。

> [!NOTE]
> [ASP.NET Web API からの移行](xref:migration/webapi)に関するページをご覧ください。

Web API Compatibility Shim を使用するには、プロジェクトにパッケージを追加し、`Startup` の `AddWebApiConventions` を呼び出して MVC に規則を追加する必要があります。

```csharp
services.AddMvc().AddWebApiConventions();
```

Shim が提供するこの規則は、それに特定の属性が適用されたアプリの一部にのみ適用されます。 次の 4 つの属性は、Shim の規則でどのコントローラーが規則を変更する必要があるかを制御するために使用されます。

* [UseWebApiActionConventionsAttribute](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapiactionconventionsattribute)
* [UseWebApiOverloadingAttribute](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapioverloadingattribute)
* [UseWebApiParameterConventionsAttribute](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapiparameterconventionsattribute)
* [UseWebApiRoutesAttribute](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapiroutesattribute)

### <a name="action-conventions"></a>アクション規則

`UseWebApiActionConventionsAttribute` は、名に基づいてアクションに HTTP メソッドをマップするために使用されます (たとえば、`Get` は `HttpGet` にマップされます)。 これは、属性のルーティングを使用しないアクションにのみ適用されます。

### <a name="overloading"></a>オーバーロード

`UseWebApiOverloadingAttribute` は、`WebApiOverloadingApplicationModelConvention` 規則を適用するために使用されます。 この規則は、アクションの選択プロセスに `OverloadActionConstraint` を追加します。これによって、候補のアクションは、要求で省略可能でないすべてのパラメーターが満たされるものに制限されます。

### <a name="parameter-conventions"></a>パラメーター規則

`UseWebApiParameterConventionsAttribute` は、`WebApiParameterConventionsApplicationModelConvention` アクション規則を適用するために使用されます。 この規則では、アクション パラメーターとして使用される単純な型は、要求本文からバインドされる複雑な型に対し、既定で URI からバインドされることが指定されます。

### <a name="routes"></a>ルート

`UseWebApiRoutesAttribute` は、`WebApiApplicationModelConvention` コントローラー規則が適用されるかどうかを制御します。 これが有効な場合、この規則はルートに[領域](xref:mvc/controllers/areas)のサポートを追加するために使用されます。

互換パッケージには、規則のセットに加え、Web API によって提供されるものの代わりとなる `System.Web.Http.ApiController` 基底クラスが含まれます。 これにより、Web API 用に記述され、その `ApiController` から継承されたコントローラーが、ASP.NET Core MVC での実行中に、設計どおりに動作するようにすることを可能にします。 前述のすべての `UseWebApi*` 属性は、基本コントローラー クラスに適用されます。 `ApiController` では、Web API にあるものと互換性のあるプロパティ、メソッド、および結果の型が公開されます。

## <a name="using-apiexplorer-to-document-your-app"></a>アプリをドキュメント化するための ApiExplorer の使用

アプリケーションモデルは、 [`ApiExplorer`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.apiexplorermodel) アプリの構造を走査するために使用できる各レベルのプロパティを公開します。 これは、[Swagger などのツールを使用して、Web API のヘルプ ページを生成する](xref:tutorials/web-api-help-pages-using-swagger)ために使用できます。 `ApiExplorer` プロパティは、アプリのモデルのどの部分を公開するか指定するために設定できる `IsVisible` プロパティを公開します。 この設定は、次の規則を使用して構成できます。

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/EnableApiExplorerApplicationConvention.cs)]

このアプローチを使用すると (必要に応じて追加の規則が必要)、アプリ内で任意のレベルで API の可視性を有効または無効にできます。 
