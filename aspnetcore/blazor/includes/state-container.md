---
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
ms.openlocfilehash: 342cbccc5f9a61c0695d501ea2376403bfd459ca
ms.sourcegitcommit: 1f35de0ca9ba13ea63186c4dc387db4fb8e541e0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2021
ms.locfileid: "104719956"
---
入れ子になったコンポーネントは通常、「ASP.NET Core Blazor データ バインディング<xref:blazor/components/data-binding>」で説明されているように、"*チェーン バインド*" を使用してデータをバインドします。 入れ子になったコンポーネントと入れ子になっていないコンポーネントは、登録済みのメモリ内状態コンテナーを使用してデータへのアクセスを共有できます。 カスタムの状態コンテナー クラスでは、割り当て可能な <xref:System.Action> を使用して、状態変更のアプリのさまざまな部分でコンポーネントに通知できます。 次に例を示します。

* コンポーネントのペアでは、状態コンテナーを使用してプロパティを追跡します。
* この例のコンポーネントは入れ子になっていますが、この方法を使用する際に入れ子は必要ありません。

`StateContainer.cs`:

```csharp
public class StateContainer
{
    public string Property { get; set; } = "Initial value from StateContainer";

    public event Action OnChange;

    public void SetProperty(string value)
    {
        Property = value;
        NotifyStateChanged();
    }

    private void NotifyStateChanged() => OnChange?.Invoke();
}
```

`Program.Main` (Blazor WebAssembly):

```csharp
builder.Services.AddSingleton<StateContainer>();
```

`Startup.ConfigureServices` (Blazor Server):

```csharp
services.AddScoped<StateContainer>();
```

`Pages/Component1.razor`:

```razor
@page "/Component1"
@inject StateContainer StateContainer
@implements IDisposable

<h1>Component 1</h1>

<p>Component 1 Property: <b>@StateContainer.Property</b></p>

<p>
    <button @onclick="ChangePropertyValue">Change Property from Component 1</button>
</p>

<Component2 />

@code {
    protected override void OnInitialized()
    {
        StateContainer.OnChange += StateHasChanged;
    }

    private void ChangePropertyValue()
    {
        StateContainer.SetProperty($"New value set in Component 1: {DateTime.Now}");
    }

    public void Dispose()
    {
        StateContainer.OnChange -= StateHasChanged;
    }
}
```

`Shared/Component2.razor`:

```razor
@inject StateContainer StateContainer
@implements IDisposable

<h2>Component 2</h2>

<p>Component 2 Property: <b>@StateContainer.Property</b></p>

<p>
    <button @onclick="ChangePropertyValue">Change Property from Component 2</button>
</p>

@code {
    protected override void OnInitialized()
    {
        StateContainer.OnChange += StateHasChanged;
    }

    private void ChangePropertyValue()
    {
        StateContainer.SetProperty($"New value set in Component 2: {DateTime.Now}");
    }

    public void Dispose()
    {
        StateContainer.OnChange -= StateHasChanged;
    }
}
```

前のコンポーネントによって <xref:System.IDisposable> が実装され、`Dispose` メソッドで `OnChange` デリゲートがサブスクライブ解除されます。このメソッドは、コンポーネントが破棄されるときにフレームワークによって呼び出されます。 詳細については、「<xref:blazor/components/lifecycle#component-disposal-with-idisposable>」を参照してください。
