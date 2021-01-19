---
title: ASP.NET Core Blazor コンポーネントの仮想化
author: guardrex
description: ASP.NET Core Blazor アプリでコンポーネントの仮想化を使用する方法について説明します。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/02/2020
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
uid: blazor/components/virtualization
ms.openlocfilehash: afd2da19641b41871f06426934c39348daa54b1f
ms.sourcegitcommit: 2fea9bfe6127bbbdbb438406c82529b2bc331944
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/11/2021
ms.locfileid: "98065533"
---
# <a name="aspnet-core-no-locblazor-component-virtualization"></a>ASP.NET Core Blazor コンポーネントの仮想化

作成者: [Daniel Roth](https://github.com/danroth27)

Blazor フレームワークに組み込まれている仮想化サポートを使用して、コンポーネント レンダリングの認識されるパフォーマンスを向上させます。 仮想化は、UI レンダリングを現在表示されている部分のみに制限するための手法です。 たとえば、仮想化が有用なのは、アプリで項目の長いリストや項目の一部のみをレンダリングする必要があるが、表示する必要があるのはどんなときでも項目のサブセットのみである場合です。 Blazor には、仮想化をアプリのコンポーネントに追加するために使用できる [`Virtualize` コンポーネント](xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601)が用意されています。

`Virtualize` コンポーネントは、次の場合に使用できます。

* ループ内の一連のデータ項目をレンダリングする。
* スクロールが原因でほとんどの項目が表示されない。
* レンダリングされる項目のサイズはまったく同じ。 ユーザーが任意の点にスクロールすると、コンポーネントでは表示する項目を計算できます。

仮想化を使用しない場合は、一般的なリストで、C# [`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) ループを使用してリストの各項目を表示できます。

```razor
<div style="height:500px;overflow-y:scroll">
    @foreach (var flight in allFlights)
    {
        <FlightSummary @key="flight.FlightId" Details="@flight.Summary" />
    }
</div>
```

リストに何千もの項目が含まれている場合は、リストの表示に時間がかかることがあります。 UI の表示が明らかに遅いとユーザーが感じる場合があります。

リスト内の各項目を一度に表示するのではなく、[`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) ループを `Virtualize` コンポーネントに置き換え、<xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Items%2A?displayProperty=nameWithType> を使用して固定の項目ソースを指定します。 現在表示される項目のみがレンダリングされます。

```razor
<div style="height:500px;overflow-y:scroll">
    <Virtualize Items="@allFlights" Context="flight">
        <FlightSummary @key="flight.FlightId" Details="@flight.Summary" />
    </Virtualize>
</div>
```

`Context` を使用してコンポーネントにコンテキストを指定しない場合は、項目コンテンツ テンプレートで `context` 値を使用します。

```razor
<div style="height:500px;overflow-y:scroll">
    <Virtualize Items="@allFlights">
        <FlightSummary @key="context.FlightId" Details="@context.Summary" />
    </Virtualize>
</div>
```

> [!NOTE]
> 要素およびコンポーネントへのモデル オブジェクトのマッピング プロセスは、[`@key`](xref:mvc/views/razor#key) ディレクティブ属性を使用して制御できます。 `@key` により、比較アルゴリズムで、キーの値に基づいて要素またはコンポーネントが確実に保持されます。
>
> 詳細については、次の記事を参照してください。
>
> * <xref:blazor/components/index#use-key-to-control-the-preservation-of-elements-and-components>
> * <xref:mvc/views/razor#key>

`Virtualize` コンポーネント:

* コンテナーの高さとレンダリングする項目のサイズに基づいて、レンダリングする項目の数を計算します。
* ユーザーがスクロールするときに項目を再計算し、再レンダリングします。
* コレクションからすべてのデータをダウンロードするのでなく、現在の表示領域に対応するレコードの一部だけを外部 API からフェッチします。

`Virtualize` コンポーネントの項目コンテンツには次を含めることができます。

* 前の例に含まれていたプレーン HTML および Razor コード。
* 1 つまたは複数の Razor コンポーネント。
* HTML/Razor および Razor コンポーネントの混合。

## <a name="item-provider-delegate"></a>項目プロバイダー デリゲート

すべての項目をメモリに読み込みたいわけではない場合は、要求された項目をオンデマンドで非同期的に取得するコンポーネントの <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemsProvider%2A?displayProperty=nameWithType> パラメーターに項目プロバイダー デリゲート メソッドを指定できます。 次の例では、`LoadEmployees` メソッドによって、項目が `Virtualize` コンポーネントに提供されます。

```razor
<Virtualize Context="employee" ItemsProvider="@LoadEmployees">
    <p>
        @employee.FirstName @employee.LastName has the 
        job title of @employee.JobTitle.
    </p>
</Virtualize>
```

項目プロバイダーは、特定の開始インデックスを開始位置として、必要な項目数を指定する <xref:Microsoft.AspNetCore.Components.Web.Virtualization.ItemsProviderRequest> を受け取ります。 次に、項目プロバイダーにより、要求される項目がデータベースまたは他のサービスから取得され、それらが <xref:Microsoft.AspNetCore.Components.Web.Virtualization.ItemsProviderResult%601> として、合計項目数と共に返されます。 項目プロバイダーでは、項目を要求ごとに取得するか、すぐに使用できるようにキャッシュするかを選択できます。

`Virtualize` コンポーネントでは、そのパラメーターから **1 つの項目ソース** のみを受け入れることができるので、項目プロバイダーを同時に使用して `Items` にコレクションを割り当てることは避けてください。 両方が割り当てられている場合は、コンポーネントのパラメーターが実行時に設定されると <xref:System.InvalidOperationException> がスローされます。

次の `LoadEmployees` メソッドの例では、`EmployeeService` (表示されていません) から従業員が読み込まれます。

```csharp
private async ValueTask<ItemsProviderResult<Employee>> LoadEmployees(
    ItemsProviderRequest request)
{
    var numEmployees = Math.Min(request.Count, totalEmployees - request.StartIndex);
    var employees = await EmployeesService.GetEmployeesAsync(request.StartIndex, 
        numEmployees, request.CancellationToken);

    return new ItemsProviderResult<Employee>(employees, totalEmployees);
}
```

<xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemsProvider%2A> にデータを再要求するように、<xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.RefreshDataAsync%2A?displayProperty=nameWithType> からコンポーネントに指示が出されます。 これは、外部データが変更される場合に便利です。 <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Items%2A> を使用する場合、これを呼び出す必要はありません。

## <a name="placeholder"></a>プレースホルダー

リモート データ ソースに項目を要求には時間がかかる場合があるため、項目のコンテンツを含むプレースホルダーをレンダリングするオプションがあります。

* 項目データが使用可能になるまでコンテンツを表示するには、<xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Placeholder%2A> (`<Placeholder>...</Placeholder>`) を使用します。
* リストの項目テンプレートを設定するには、<xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemContent%2A?displayProperty=nameWithType> を使用します。

```razor
<Virtualize Context="employee" ItemsProvider="@LoadEmployees">
    <ItemContent>
        <p>
            @employee.FirstName @employee.LastName has the 
            job title of @employee.JobTitle.
        </p>
    </ItemContent>
    <Placeholder>
        <p>
            Loading&hellip;
        </p>
    </Placeholder>
</Virtualize>
```

## <a name="item-size"></a>アイテムのサイズ

各項目のサイズは <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A?displayProperty=nameWithType> で設定できます (既定値: 50):

```razor
<Virtualize Context="employee" Items="@employees" ItemSize="25">
    ...
</Virtualize>
```

## <a name="overscan-count"></a>オーバースキャン数

<xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.OverscanCount%2A?displayProperty=nameWithType> を使用して、表示領域の前後にレンダリングされる追加項目の数を指定します。 この設定は、スクロール中のレンダリングの頻度を減らすのに利用できます。 ただし、値を大きくすると、ページにレンダリングされる要素が多くなります (既定値:3)。

```razor
<Virtualize Context="employee" Items="@employees" OverscanCount="4">
    ...
</Virtualize>
```

## <a name="state-changes"></a>状態変更

`Virtualize` コンポーネントによってレンダリングされる項目を変更する場合は、<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出して、強制的にコンポーネントを再評価して再レンダリングします。
