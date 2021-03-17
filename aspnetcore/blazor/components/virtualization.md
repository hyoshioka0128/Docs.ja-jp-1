---
title: ASP.NET Core Blazor コンポーネントの仮想化
author: guardrex
description: ASP.NET Core Blazor アプリでコンポーネントの仮想化を使用する方法について説明します。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 02/26/2021
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
ms.openlocfilehash: c81732c29b262e9134a4ff7dab077a4f31db96af
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102109820"
---
# <a name="aspnet-core-blazor-component-virtualization"></a>ASP.NET Core Blazor コンポーネントの仮想化

[`Virtualize` コンポーネント](xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601) で Blazor フレームワークに組み込まれている仮想化サポートを使用して、コンポーネント レンダリングの体感パフォーマンスを向上させます。 仮想化は、UI レンダリングを現在表示されている部分のみに制限するための手法です。 たとえば、仮想化が有用なのは、アプリで項目の長いリストや項目の一部のみをレンダリングする必要があるが、表示する必要があるのはどんなときでも項目のサブセットのみである場合です。

`Virtualize` コンポーネントを使用する場合:

* ループ内の一連のデータ項目をレンダリングする。
* スクロールが原因でほとんどの項目が表示されない。
* レンダリングされる項目のサイズが同じ。

`Virtualize` コンポーネントの項目リスト内の任意のポイントにユーザーがスクロールすると、コンポーネントによって表示可能な項目が計算されます。 非表示の項目はレンダリングされません。

仮想化を使用しない場合は、一般的なリストで、C# [`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) ループを使用してリストの各項目を表示できます。 次に例を示します。

* `allFlights` は、飛行機のフライトのコレクションです。
* `FlightSummary` コンポーネントには各フライトの詳細が表示されます。
* [`@key` ディレクティブ属性](xref:blazor/components/index#use-key-to-control-the-preservation-of-elements-and-components)によって、フライトの `FlightId` ごとにレンダリングされたフライトに対する各 `FlightSummary` コンポーネントの関係が保持されます。

```razor
<div style="height:500px;overflow-y:scroll">
    @foreach (var flight in allFlights)
    {
        <FlightSummary @key="flight.FlightId" Details="@flight.Summary" />
    }
</div>
```

コレクションに数千のフライトが含まれている場合、フライトのレンダリングに時間がかかり、ユーザーは UI の表示が明らかに遅いと感じます。 ほとんどのフライトは、`<div>` 要素の高さから外れているため、レンダリングされません。

フライトのリスト全体を一度にレンダリングするのではなく、前の例の [`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) ループを `Virtualize` コンポーネントに置き換えます。

* <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Items%2A?displayProperty=nameWithType> の固定項目ソースとして `allFlights` を指定します。 現在表示されているフライトだけが、`Virtualize` コンポーネントによってレンダリングされます。
* `Context` パラメーターを使用して、各フライトのコンテキストを指定します。 次の例では、`flight` がコンテキストとして使用されています。これにより、各フライトのメンバーへのアクセスを提供します。

```razor
<div style="height:500px;overflow-y:scroll">
    <Virtualize Items="@allFlights" Context="flight">
        <FlightSummary @key="flight.FlightId" Details="@flight.Summary" />
    </Virtualize>
</div>
```

コンテキストが `Context` パラメーターを使用して指定されていない場合は、項目コンテンツ テンプレート内の `context` の値を使用して、各フライトのメンバーにアクセスします。

```razor
<div style="height:500px;overflow-y:scroll">
    <Virtualize Items="@allFlights">
        <FlightSummary @key="context.FlightId" Details="@context.Summary" />
    </Virtualize>
</div>
```

`Virtualize` コンポーネント:

* コンテナーの高さとレンダリングする項目のサイズに基づいて、レンダリングする項目の数を計算します。
* ユーザーがスクロールすると、項目が再計算され、再レンダリングされます。
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

<xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemsProvider%2A> にデータを再要求するように、<xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.RefreshDataAsync%2A?displayProperty=nameWithType> からコンポーネントに指示が出されます。 これは、外部データが変更される場合に便利です。 <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Items%2A> を使用する場合、<xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.RefreshDataAsync%2A> を呼び出す必要はありません。

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

各項目の高さ (ピクセル単位) は <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A?displayProperty=nameWithType> で設定できます (既定値: 50)。 次の例では、各項目の高さを既定値の 50 ピクセルから 25 ピクセルに変更します。

```razor
<Virtualize Context="employee" Items="@employees" ItemSize="25">
    ...
</Virtualize>
```

既定では、初期レンダリングが行われた "*後*" に、`Virtualize` コンポーネントによって個々の項目のレンダリング サイズ (高さ) が測定されます。 正確な初期レンダリングのパフォーマンスを支援し、ページを再読み込みするための正しいスクロール位置を確保するために、<xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A> を使用して事前に正確な項目のサイズを提供します。 既定の <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A> によって一部の項目が現在表示されているビューの外側にレンダーされる場合、2 回目の再レンダリングがトリガーされます。 仮想化されたリストでブラウザーのスクロール位置を正しく維持するには、最初のレンダリングが正しいことが必要です。 そうでない場合、間違った項目がユーザーに表示されるおそれがあります。

## <a name="overscan-count"></a>オーバースキャン数

<xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.OverscanCount%2A?displayProperty=nameWithType> を使用して、表示領域の前後にレンダリングされる追加項目の数を指定します。 この設定は、スクロール中のレンダリングの頻度を減らすのに利用できます。 ただし、値を大きくすると、ページにレンダリングされる要素が多くなります (既定値: 3)。 次の例では、オーバースキャン数を既定の 3 項目から 4 項目に変更します。

```razor
<Virtualize Context="employee" Items="@employees" OverscanCount="4">
    ...
</Virtualize>
```

## <a name="state-changes"></a>状態変更

`Virtualize` コンポーネントによってレンダリングされる項目を変更する場合は、<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出して、強制的にコンポーネントを再評価して再レンダリングします。 詳細については、「<xref:blazor/components/rendering>」を参照してください。
