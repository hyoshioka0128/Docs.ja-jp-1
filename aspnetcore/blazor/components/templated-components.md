---
title: ASP.NET Core Blazor テンプレート コンポーネント
author: guardrex
description: テンプレート コンポーネントで 1 つまたは複数の UI テンプレートをパラメーターとして受け取る方法について学習します。これは、コンポーネントのレンダリング ロジックの一部として使用できます。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/04/2021
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
uid: blazor/components/templated-components
ms.openlocfilehash: 6c94218f3808baca18f23a53688bafdd6354e760
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589479"
---
# <a name="aspnet-core-blazor-templated-components"></a>ASP.NET Core Blazor テンプレート コンポーネント

テンプレート コンポーネントは、1 つまたは複数の UI テンプレートをパラメーターとして受け取るコンポーネントです。その後、コンポーネントのレンダリング ロジックの一部として使用できます。 テンプレート コンポーネントを使用すると、通常のコンポーネントよりも再利用しやすい上位レベルのコンポーネントを作成できます。 いくつかの例を次に示します。

* テーブルのヘッダー、行、フッターのテンプレートをユーザーが指定できるようにするテーブル コンポーネント
* リスト内の項目をレンダリングするためのテンプレートをユーザーが指定できるようにするリスト コンポーネント

<xref:Microsoft.AspNetCore.Components.RenderFragment> または <xref:Microsoft.AspNetCore.Components.RenderFragment%601> の型の 1 つまたは複数のコンポーネント パラメーターを指定することで、テンプレート コンポーネントが定義されます。 レンダー フラグメントは、レンダリングする UI のセグメントを表します。 <xref:Microsoft.AspNetCore.Components.RenderFragment%601> は、レンダー フラグメントが呼び出されたときに指定できる型パラメーターを受け取ります。

次の `TableTemplate` コンポーネントが示すように、多くの場合、テンプレート コンポーネントはジェネリック型です。 この例で、ジェネリック型 `<T>` は、`IReadOnlyList<T>` 値のレンダリングに使用されます。ここでは、pet の表を表示するコンポーネント内の一連の pet 行です。

`Shared/TableTemplate.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/templated-components/TableTemplate.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/templated-components/TableTemplate.razor)]

::: moniker-end

テンプレート コンポーネントを使用する場合、テンプレート パラメーターは、パラメーターの名前と一致する子要素を使用して指定することができます。 次の例では、`<TableHeader>...</TableHeader>` と `<RowTemplate>...<RowTemplate>` を使用して、`TableTemplate` コンポーネントの `TableHeader` と `RowTemplate` に対して、<xref:Microsoft.AspNetCore.Components.RenderFragment%601> を指定します。

暗黙的な子コンテンツ (ラップする子要素を持たない) のコンテンツ パラメーター名を指定する場合は、コンポーネント要素に `Context` 属性を指定します。 次の例では、`Context` 属性が `TableTemplate` 要素に表示され、すべての <xref:Microsoft.AspNetCore.Components.RenderFragment%601> テンプレート パラメーターに適用されます。

`Pages/Pets.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets1.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets1.razor)]

::: moniker-end

または、<xref:Microsoft.AspNetCore.Components.RenderFragment%601> 子要素で `Context` 属性を使用して、パラメーター名を変更することもできます。 次の例では、`Context` が、`TableTemplate` ではなく、`RowTemplate` に設定されています。

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets2.razor?name=snippet&highlight=6)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets2.razor?name=snippet&highlight=6)]

::: moniker-end

<xref:Microsoft.AspNetCore.Components.RenderFragment%601> 型のコンポーネント引数には、`context` という名前の暗黙的なパラメーターがあり、これを使用することができます。 次の例では、`Context` は設定されていません。 `@context.{PROPERTY}` によって、テンプレートに pet 値が指定されます。このテンプレートでは、`{PROPERTY}` は `Pet` プロパティです。

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets3.razor?name=snippet&highlight=7-8)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets3.razor?name=snippet&highlight=7-8)]

::: moniker-end

ジェネリック型のコンポーネントを使用するときは、可能な場合に型パラメーターが推論されます。 ただし、前の例の `TItem` のように、型パラメーターに一致する名前を持つ属性を使用して、明示的に型を指定することができます。

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets4.razor?name=snippet&highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets4.razor?name=snippet&highlight=1)]

::: moniker-end

## <a name="generic-type-constraints"></a>ジェネリック型の制約

> [!NOTE]
> ジェネリック型の制約は、将来のリリースでサポートされる予定です。 詳細については、[ジェネリック型の制約の許可 (dotnet/aspnetcore #8433)](https://github.com/dotnet/aspnetcore/issues/8433) を参照してください。

## <a name="additional-resources"></a>その他のリソース

* <xref:blazor/webassembly-performance-best-practices#define-reusable-renderfragments-in-code>
