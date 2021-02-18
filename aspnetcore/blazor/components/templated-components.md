---
title: ASP.NET Core Blazor テンプレート コンポーネント
author: guardrex
description: テンプレート コンポーネントで 1 つまたは複数の UI テンプレートをパラメーターとして受け取る方法について学習します。これは、コンポーネントのレンダリング ロジックの一部として使用できます。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/18/2020
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
ms.openlocfilehash: 579cabd9e6b7141ec6af4c6e221b805272a2fe40
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280030"
---
# <a name="aspnet-core-blazor-templated-components"></a>ASP.NET Core Blazor テンプレート コンポーネント

テンプレート コンポーネントは、1 つまたは複数の UI テンプレートをパラメーターとして受け取るコンポーネントです。その後、コンポーネントのレンダリング ロジックの一部として使用できます。 テンプレート コンポーネントを使用すると、通常のコンポーネントよりも再利用しやすい上位レベルのコンポーネントを作成できます。 いくつかの例を次に示します。

* テーブルのヘッダー、行、フッターのテンプレートをユーザーが指定できるようにするテーブル コンポーネント
* リスト内の項目をレンダリングするためのテンプレートをユーザーが指定できるようにするリスト コンポーネント

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="template-parameters"></a>テンプレート パラメーター

<xref:Microsoft.AspNetCore.Components.RenderFragment> または <xref:Microsoft.AspNetCore.Components.RenderFragment%601> の型の 1 つまたは複数のコンポーネント パラメーターを指定することで、テンプレート コンポーネントが定義されます。 レンダー フラグメントは、レンダリングする UI のセグメントを表します。 <xref:Microsoft.AspNetCore.Components.RenderFragment%601> は、レンダー フラグメントが呼び出されたときに指定できる型パラメーターを受け取ります。

`TableTemplate` コンポーネント (`TableTemplate.razor`):

[!code-razor[](../common/samples/5.x/BlazorWebAssemblySample/Components/TableTemplate.razor)]

テンプレート コンポーネントを使用する場合、テンプレート パラメーターは、パラメーターの名前と一致する子要素を使用して指定できます (次の例では `TableHeader` と `RowTemplate`)。

```razor
<TableTemplate Items="pets">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate>
        <td>@context.PetId</td>
        <td>@context.Name</td>
    </RowTemplate>
</TableTemplate>

@code {
    private List<Pet> pets = new List<Pet>
    {
        new Pet { PetId = 2, Name = "Mr. Bigglesworth" },
        new Pet { PetId = 4, Name = "Salem Saberhagen" },
        new Pet { PetId = 7, Name = "K-9" }
    };

    private class Pet
    {
        public int PetId { get; set; }
        public string Name { get; set; }
    }
}
```

> [!NOTE]
> ジェネリック型の制約は、将来のリリースでサポートされる予定です。 詳細については、[ジェネリック型の制約の許可 (dotnet/aspnetcore #8433)](https://github.com/dotnet/aspnetcore/issues/8433) を参照してください。

## <a name="template-context-parameters"></a>テンプレート コンテキスト パラメーター

要素として渡される型 <xref:Microsoft.AspNetCore.Components.RenderFragment%601> のコンポーネント引数には、`context` という名前の暗黙的なパラメーターがありますが (たとえば、上記のコード サンプルの `@context.PetId`)、子要素の `Context` 属性を使用してパラメーター名を変更できます。 次の例では、`RowTemplate` 要素の `Context` 属性で `pet` パラメーターを指定しています。

```razor
<TableTemplate Items="pets">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate Context="pet">
        <td>@pet.PetId</td>
        <td>@pet.Name</td>
    </RowTemplate>
</TableTemplate>

@code {
    ...
}
```

または、コンポーネント要素の `Context` 属性を指定することもできます。 指定された `Context` 属性は、指定されたすべてのテンプレート パラメーターに適用されます。 これは、(ラップする子要素を持たない) 暗黙的な子コンテンツのコンテンツ パラメーター名を指定する場合に便利です。 次の例では、`Context` 属性が `TableTemplate` 要素に表示され、すべてのテンプレート パラメーターに適用されます。

```razor
<TableTemplate Items="pets" Context="pet">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate>
        <td>@pet.PetId</td>
        <td>@pet.Name</td>
    </RowTemplate>
</TableTemplate>

@code {
    ...
}
```

## <a name="generic-typed-components"></a>ジェネリック型のコンポーネント

多くの場合、テンプレート コンポーネントはジェネリック型です。 たとえば、ジェエリックの `ListViewTemplate` コンポーネント (`ListViewTemplate.razor`) を使用して、`IEnumerable<T>` 値をレンダリングできます。 ジェネリック コンポーネントを定義するには、[`@typeparam`](xref:mvc/views/razor#typeparam) ディレクティブを使用して、型パラメーターを指定します。

[!code-razor[](../common/samples/5.x/BlazorWebAssemblySample/Components/ListViewTemplate.razor)]

ジェネリック型のコンポーネントを使用する場合、可能であれば型パラメーターは推論されます。

```razor
<ListViewTemplate Items="pets">
    <ItemTemplate Context="pet">
        <li>@pet.Name</li>
    </ItemTemplate>
</ListViewTemplate>

@code {
    private List<Pet> pets = new List<Pet>
    {
        new Pet { Name = "Mr. Bigglesworth" },
        new Pet { Name = "Salem Saberhagen" },
        new Pet { Name = "K-9" }
    };

    private class Pet
    {
        public string Name { get; set; }
    }
}
```

それ以外の場合は、型パラメーターの名前と一致する属性を使用して、型パラメーターを明示的に指定する必要があります。 次の例では、`TItem="Pet"` によって次の型が指定されます。

```razor
<ListViewTemplate Items="pets" TItem="Pet">
    <ItemTemplate Context="pet">
        <li>@pet.Name</li>
    </ItemTemplate>
</ListViewTemplate>

@code {
    ...
}
```

## <a name="additional-resources"></a>その他のリソース

* <xref:blazor/webassembly-performance-best-practices#define-reusable-renderfragments-in-code>
