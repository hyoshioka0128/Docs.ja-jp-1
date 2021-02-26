---
title: ASP.NET Core Blazor の値とパラメーターのカスケード
author: guardrex
description: 先祖のコンポーネントから子孫のコンポーネントにデータをフローさせる方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/02/2021
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
uid: blazor/components/cascading-values-and-parameters
ms.openlocfilehash: 1fb9d75ca1613a7098840efd3ecb86ee90f4064c
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280243"
---
# <a name="aspnet-core-blazor-cascading-values-and-parameters"></a>ASP.NET Core Blazor の値とパラメーターのカスケード

"*カスケード値とパラメーター*" の使用は、コンポーネント階層で先祖コンポーネントから下位の任意の数の子孫コンポーネントにデータをフローさせる便利な方法です。 カスケード値およびパラメーターでは、[コンポーネント パラメーター](xref:blazor/components/index#component-parameters)とは異なり、データが使用される各子孫コンポーネントに属性を割り当てる必要がありません。 また、カスケード値とパラメーターを使用すると、コンポーネント階層全体でコンポーネントを相互連携させることができます。

## <a name="cascadingvalue-component"></a>`CascadingValue` コンポーネント

先祖コンポーネントは、コンポーネント階層のサブツリーをラップし、そのサブツリー内のすべてのコンポーネントに単一の値を提供する Blazor フレームワークの [`CascadingValue`](xref:Microsoft.AspNetCore.Components.CascadingValue%601) コンポーネントを使用して、カスケード値を提供します。

次の例では、子コンポーネントのボタンに CSS 形式のクラスを提供する、レイアウト コンポーネントのコンポーネント階層におけるテーマ情報のフローを示しています。

次の `ThemeInfo` C# クラスは、`UIThemeClasses` という名前のフォルダーに配置され、テーマ情報を指定します。

> [!NOTE]
> このセクションの例では、アプリの名前空間は `BlazorSample` です。 自分独自のサンプル アプリでコードを試す場合は、アプリの名前空間をお使いのサンプル アプリの名前空間に変更します。

`UIThemeClasses/ThemeInfo.cs`:

```csharp
namespace BlazorSample.UIThemeClasses
{
    public class ThemeInfo
    {
        public string ButtonClass { get; set; }
    }
}
```

次の[レイアウト コンポーネント](xref:blazor/layouts)は、<xref:Microsoft.AspNetCore.Components.LayoutComponentBase.Body> プロパティのレイアウト本体を構成するすべてのコンポーネントに、テーマ情報 (`ThemeInfo`) をカスケード値として指定しています。 `ButtonClass` には、Bootstrap ボタン形式の [`btn-success`](https://getbootstrap.com/docs/5.0/components/buttons/) 値が割り当てられています。 `ButtonClass` プロパティは、`ThemeInfo` カスケード値を介し、コンポーネント階層内のすべての子孫コンポーネントで使用できます。

`Shared/MainLayout.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/MainLayout.razor?highlight=2,10-14,19)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/MainLayout.razor?highlight=2,9-13,17)]

::: moniker-end

## <a name="cascadingparameter-attribute"></a>`[CascadingParameter]` 属性

子孫コンポーネントでは、[`[CascadingParameter]` 属性](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute)を使用してカスケード型パラメーターを宣言し、カスケード値を使用します。 カスケード値は、**型で** カスケード型パラメーターにバインドされます。 同じ型の複数の値のカスケードについては、後でこの記事の「[複数の値のカスケード](#cascade-multiple-values)」セクションで説明します。

次のコンポーネントは、オプションで同じ `ThemeInfo` 名を使用してカスケード型パラメーターに `ThemeInfo` カスケード値をバインドします。 このパラメーターは、 **`Increment Counter (Themed)`** ボタンの CSS クラスを設定するのに使用されます。

`Pages/ThemedCounter.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/ThemedCounter.razor?highlight=2,15-17,23-24)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/ThemedCounter.razor?highlight=2,15-17,23-24)]

::: moniker-end

## <a name="cascade-multiple-values"></a>複数の値のカスケード

同じサブツリー内で同じ型の値を複数カスケードするには、各 [`CascadingValue`](xref:Microsoft.AspNetCore.Components.CascadingValue%601) コンポーネントとそれに対応する [`[CascadingParameter]` 属性](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute)に一意の <xref:Microsoft.AspNetCore.Components.CascadingValue%601.Name%2A> 文字列を指定します。

次の例では、2 つの [`CascadingValue`](xref:Microsoft.AspNetCore.Components.CascadingValue%601) コンポーネントが、`CascadingType` の異なるインスタンスをカスケードしています。

```razor
<CascadingValue Value="@parentCascadeParameter1" Name="CascadeParam1">
    <CascadingValue Value="@ParentCascadeParameter2" Name="CascadeParam2">
        ...
    </CascadingValue>
</CascadingValue>

@code {
    private CascadingType parentCascadeParameter1;

    [Parameter]
    public CascadingType ParentCascadeParameter2 { get; set; }

    ...
}
```

子孫コンポーネントで、カスケードされたパラメーターはそれらのカスケードされた値を、次のように <xref:Microsoft.AspNetCore.Components.CascadingValue%601.Name%2A> を使用して、先祖コンポーネントから受け取ります。

```razor
...

@code {
    [CascadingParameter(Name = "CascadeParam1")]
    protected CascadingType ChildCascadeParameter1 { get; set; }
    
    [CascadingParameter(Name = "CascadeParam2")]
    protected CascadingType ChildCascadeParameter2 { get; set; }
}
```

## <a name="pass-data-across-a-component-hierarchy"></a>コンポーネント階層に渡ってデータを渡す

カスケード型パラメーターにより、コンポーネントがコンポーネント階層間でデータを渡せるようにすることもできます。 タブ セット コンポーネントによって一連の個別タブが維持される、次の UI タブ セットの例を考えてみてください。

> [!NOTE]
> このセクションの例では、アプリの名前空間は `BlazorSample` です。 自分独自のサンプル アプリでコードを試す場合は、名前空間をお使いのサンプル アプリの名前空間に変更します。

`UIInterfaces` という名前のフォルダーに、タブが実装する `ITab` インターフェイスを作成します。

`UIInterfaces/ITab.cs`:

```csharp
using Microsoft.AspNetCore.Components;

namespace BlazorSample.UIInterfaces
{
    public interface ITab
    {
        RenderFragment ChildContent { get; }
    }
}
```

一連のタブは、次の `TabSet` コンポーネントによって維持されます。 リスト (`<ul>...</ul>`) のリスト項目 (`<li>...</li>`) は、このセクションで後で作成するタブ セットの `Tab` コンポーネントによって提供されます。

子 `Tab` コンポーネントは、`TabSet` にパラメーターとして明示的に渡されません。 代わりに、子 `Tab` コンポーネントは、`TabSet` の子コンテンツに含まれます。 ただし、ヘッダーとアクティブなタブをレンダリングできるように、`TabSet` は、各 `Tab` コンポーネントをまだ参照する必要があります。追加のコードを必要とせずにこの調整を可能にするために、`TabSet` コンポーネントでは、*それ自体をカスケード値として指定し*、その後に子孫 `Tab` コンポーネントによって取得できるようにします。

`Shared/TabSet.razor`:

```razor
@using BlazorSample.UIInterfaces

<!-- Display the tab headers -->

<CascadingValue Value=this>
    <ul class="nav nav-tabs">
        @ChildContent
    </ul>
</CascadingValue>

<!-- Display body for only the active tab -->

<div class="nav-tabs-body p-4">
    @ActiveTab?.ChildContent
</div>

@code {
    [Parameter]
    public RenderFragment ChildContent { get; set; }

    public ITab ActiveTab { get; private set; }

    public void AddTab(ITab tab)
    {
        if (ActiveTab == null)
        {
            SetActiveTab(tab);
        }
    }

    public void SetActiveTab(ITab tab)
    {
        if (ActiveTab != tab)
        {
            ActiveTab = tab;
            StateHasChanged();
        }
    }
}
```

子孫 `Tab` コンポーネントは、カスケード型パラメーターとして含まれる `TabSet` を取得します。 `Tab` コンポーネントは、アクティブなタブの設定のために自身を `TabSet` と座標に追加します。

`Shared/Tab.razor`:

```razor
@using BlazorSample.UIInterfaces
@implements ITab

<li>
    <a @onclick="ActivateTab" class="nav-link @TitleCssClass" role="button">
        @Title
    </a>
</li>

@code {
    [CascadingParameter]
    public TabSet ContainerTabSet { get; set; }

    [Parameter]
    public string Title { get; set; }

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    private string TitleCssClass => 
        ContainerTabSet.ActiveTab == this ? "active" : null;

    protected override void OnInitialized()
    {
        ContainerTabSet.AddTab(this);
    }

    private void ActivateTab()
    {
        ContainerTabSet.SetActiveTab(this);
    }
}
```

次の `ExampleTabSet` コンポーネントは、3 つの `Tab` コンポーネントを含む `TabSet` コンポーネントを使用しています。

`Pages/ExampleTabSet.razor`:

```razor
@page "/example-tab-set"

<TabSet>
    <Tab Title="First tab">
        <h4>Greetings from the first tab!</h4>

        <label>
            <input type="checkbox" @bind="showThirdTab" />
            Toggle third tab
        </label>
    </Tab>

    <Tab Title="Second tab">
        <h4>Hello from the second tab!</h4>
    </Tab>

    @if (showThirdTab)
    {
        <Tab Title="Third tab">
            <h4>Welcome to the disappearing third tab!</h4>
            <p>Toggle this tab from the first tab.</p>
        </Tab>
    }
</TabSet>

@code {
    private bool showThirdTab;
}
```
