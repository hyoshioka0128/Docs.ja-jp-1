---
title: ローカライズの拡張性
author: hishamco
description: ASP.NET Core アプリでローカライズ API を拡張する方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/03/2019
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
uid: fundamentals/localization-extensibility
ms.openlocfilehash: c41ebc0506a565666963d9b4c70ed73df389c826
ms.sourcegitcommit: 610936e4d3507f7f3d467ed7859ab9354ec158ba
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/25/2021
ms.locfileid: "98751665"
---
# <a name="localization-extensibility"></a>ローカライズの拡張性

作成者: [Hisham Bin Ateya](https://github.com/hishamco)

この記事の内容は次のとおりです。

* ローカライズ API の拡張性ポイントを挙げます。
* ASP.NET Core アプリのローカライズを拡張する手順を紹介します。

## <a name="extensible-points-in-localization-apis"></a>ローカライズ API の拡張性ポイント

ASP.NET Core ローカライズ API は拡張性を持つように作られています。 拡張性により、開発者はそのニーズに合わせてローカライズをカスタマイズできます。 たとえば、[OrchardCore](https://github.com/orchardCMS/OrchardCore/) には `POStringLocalizer` があります。 `POStringLocalizer` では、[Portable Object ローカライズ](xref:fundamentals/portable-object-localization)で `PO` ファイルを使用し、ローカライズ リソースを格納することが詳しく説明されます。

この記事では、ローカライズ API によって与えられる 2 つの主要な拡張性ポイントを挙げます。 

* <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider>
* <xref:Microsoft.Extensions.Localization.IStringLocalizer>

## <a name="localization-culture-providers"></a>ローカライズ カルチャ プロバイダー

ASP.NET Core ローカライズ API には、実行中の要求の現在のカルチャを決定できる既定のプロバイダーが 4 つあります。

* <xref:Microsoft.AspNetCore.Localization.QueryStringRequestCultureProvider>
* <xref:Microsoft.AspNetCore.Localization.CookieRequestCultureProvider>
* <xref:Microsoft.AspNetCore.Localization.AcceptLanguageHeaderRequestCultureProvider>
* <xref:Microsoft.AspNetCore.Localization.CustomRequestCultureProvider>

前のプロバイダーの詳しい説明は、[ローカライズ ミドルウェア](xref:fundamentals/localization) ドキュメントにあります。 既定のプロバイダーでニーズが満たされない場合、次のいずれかの手法でカスタム プロバイダーを構築します。

### <a name="use-customrequestcultureprovider"></a>CustomRequestCultureProvider を使用する

<xref:Microsoft.AspNetCore.Localization.CustomRequestCultureProvider> からは、単純な委任を利用して現在のローカライズ カルチャを決定するカスタム <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider> が与えられます。

::: moniker range="< aspnetcore-3.0"
```csharp
options.RequestCultureProviders.Insert(0, new CustomRequestCultureProvider(async context =>
{
    var currentCulture = "en";
    var segments = context.Request.Path.Value.Split(new char[] { '/' }, 
        StringSplitOptions.RemoveEmptyEntries);

    if (segments.Length > 1 && segments[0].Length == 2)
    {
        currentCulture = segments[0];
    }

    var requestCulture = new ProviderCultureResult(currentCulture);
    
    return Task.FromResult(requestCulture);
}));
```

::: moniker-end

::: moniker range=">= aspnetcore-3.0"
```csharp
options.AddInitialRequestCultureProvider(new CustomRequestCultureProvider(async context =>
{
    var currentCulture = "en";
    var segments = context.Request.Path.Value.Split(new char[] { '/' }, 
        StringSplitOptions.RemoveEmptyEntries);

    if (segments.Length > 1 && segments[0].Length == 2)
    {
        currentCulture = segments[0];
    }

    var requestCulture = new ProviderCultureResult(currentCulture);
    
    return Task.FromResult(requestCulture);
}));
```

::: moniker-end

### <a name="use-a-new-implemetation-of-requestcultureprovider"></a>RequestCultureProvider の新しい実装を使用する

カスタム ソースから要求カルチャ情報を決定する <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider> の新しい実装を作成できます。 たとえば、構成ファイルまたはデータベースをカスタム ソースにすることができます。

次の例では、`AppSettingsRequestCultureProvider` を確認できます。これによって <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider> が拡張され、 *appsettings.json* からの要求カルチャ情報が決定されます。

```csharp
public class AppSettingsRequestCultureProvider : RequestCultureProvider
{
    public string CultureKey { get; set; } = "culture";

    public string UICultureKey { get; set; } = "ui-culture";

    public override Task<ProviderCultureResult> DetermineProviderCultureResult(HttpContext httpContext)
    {
        if (httpContext == null)
        {
            throw new ArgumentNullException();
        }

        var configuration = httpContext.RequestServices.GetService<IConfigurationRoot>();
        var culture = configuration[CultureKey];
        var uiCulture = configuration[UICultureKey];

        if (culture == null && uiCulture == null)
        {
            return Task.FromResult((ProviderCultureResult)null);
        }

        if (culture != null && uiCulture == null)
        {
            uiCulture = culture;
        }

        if (culture == null && uiCulture != null)
        {
            culture = uiCulture;
        }
        
        var providerResultCulture = new ProviderCultureResult(culture, uiCulture);

        return Task.FromResult(providerResultCulture);
    }
}
```

## <a name="localization-resources"></a>ローカライズ リソース

ASP.NET Core ローカライズからは <xref:Microsoft.Extensions.Localization.ResourceManagerStringLocalizer> が与えられます。 <xref:Microsoft.Extensions.Localization.ResourceManagerStringLocalizer> は、`resx` を使用してローカライズ リソースを格納する <xref:Microsoft.Extensions.Localization.IStringLocalizer> の実装です。

`resx` ファイルの使用に限定されることはありません。 `IStringLocalizer` を実装することで、あらゆるデータ ソースを使用できます。

次のサンプル プロジェクトでは <xref:Microsoft.Extensions.Localization.IStringLocalizer> が実装されます。 

* [EFStringLocalizer](https://github.com/aspnet/Entropy/tree/master/samples/Localization.EntityFramework)
* [JsonStringLocalizer](https://github.com/hishamco/My.Extensions.Localization.Json)
* [SqlLocalizer](https://github.com/damienbod/AspNetCoreLocalization)
