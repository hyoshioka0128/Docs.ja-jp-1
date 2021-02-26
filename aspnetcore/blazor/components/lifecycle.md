---
title: ASP.NET Core Blazor ライフサイクル
author: guardrex
description: ASP.NET Core Blazor アプリで Razor コンポーネント ライフサイクル メソッドを使用する方法について学習します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 11/06/2020
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
uid: blazor/components/lifecycle
ms.openlocfilehash: 03a49c827a1f70e6b721adf293857bb33475ed36
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/10/2021
ms.locfileid: "100107078"
---
# <a name="aspnet-core-blazor-lifecycle"></a>ASP.NET Core Blazor ライフサイクル

Blazor フレームワークには、同期と非同期のライフサイクル メソッドが含まれています。 コンポーネントの初期化およびレンダリング中にコンポーネントで追加の操作を実行するには、ライフサイクル メソッドをオーバーライドします。

次の図は、Blazor のライフサイクルを示しています。 この記事の後続のセクションで、例を示しながらライフサイクル メソッドを定義します。

コンポーネント ライフサイクル イベント:

1. 要求に対してコンポーネントが初めてレンダリングされる場合は、次のようにします。
   * コンポーネントのインスタンスを作成します。
   * プロパティの挿入を実行します。 [`SetParametersAsync`](#before-parameters-are-set) を実行します。
   * [`OnInitialized{Async}`](#component-initialization-methods) を呼び出します。 <xref:System.Threading.Tasks.Task> が返された場合、<xref:System.Threading.Tasks.Task> が待機され、コンポーネントがレンダリングされます。 <xref:System.Threading.Tasks.Task> が返されない場合は、コンポーネントがレンダリングされます。
1. [`OnParametersSet{Async}`](#after-parameters-are-set) を呼び出し、コンポーネントをレンダリングします。 `OnParametersSetAsync` から <xref:System.Threading.Tasks.Task> が返された場合は、<xref:System.Threading.Tasks.Task> を待機してから、コンポーネントが再レンダリングされます。

![Blazor の Razor コンポーネントのコンポーネント ライフサイクル イベント](lifecycle/_static/lifecycle1.png)

ドキュメント オブジェクト モデル (DOM) イベント処理:

1. イベント ハンドラーが実行されます。
1. <xref:System.Threading.Tasks.Task> が返された場合、<xref:System.Threading.Tasks.Task> を待機してから、コンポーネントがレンダリングされます。 <xref:System.Threading.Tasks.Task> が返されない場合は、コンポーネントがレンダリングされます。

![ドキュメント オブジェクト モデル (DOM) イベント処理](lifecycle/_static/lifecycle2.png)

`Render` のライフサイクル:

1. コンポーネントでそれ以上のレンダリング操作を行わないようにします。
   * 最初のレンダリングの後。
   * [`ShouldRender`](#suppress-ui-refreshing) が `false` の場合。
1. レンダリング ツリーの差分を作成し、コンポーネントをレンダリングします。
1. DOM が更新されるのを待機します。
1. [`OnAfterRender{Async}`](#after-component-render) を呼び出します。

![Render ライフサイクル](lifecycle/_static/lifecycle3.png)

Developer によって [`StateHasChanged`](#state-changes) の呼び出しが行われると、結果としてレンダリングが実行されます。 詳細については、「<xref:blazor/components/rendering>」を参照してください。

## <a name="lifecycle-methods"></a>ライフサイクル メソッド

### <a name="before-parameters-are-set"></a>パラメーターが設定される前

<xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> により、レンダリング ツリーのコンポーネントの親によって、またはルート パラメーターから指定されたパラメーターが設定されます。 開発者のコードでは、このメソッドをオーバーライドすることによって、<xref:Microsoft.AspNetCore.Components.ParameterView> のパラメーターと直接対話できます。

次の例では、`Param` のルート パラメーターの解析が成功した場合、<xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A?displayProperty=nameWithType> によって `Param` パラメーターの値が `value` に代入されます。 `value` が `null` でなければ、コンポーネントによってその値が表示されます。

[ルート パラメーターの照合では大文字と小文字が区別されません](xref:blazor/fundamentals/routing#route-parameters)が、ルート テンプレートでは <xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A> によってパラメーター名が大文字と小文字を区別して照合されます。 次の例では、値を取得するために、`/{param?}` ではなく、`/{Param?}` を使用する必要があります。 このシナリオで `/{param?}` が使用される場合、<xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A> から `false` が返され、`message` はどちらの文字列にも設定されません。

`Pages/SetParametersAsyncExample.razor`:

```razor
@page "/setparametersasync-example/{Param?}"

<h1>SetParametersAsync Example</h1>

<p>@message</p>

@code {
    private string message;

    [Parameter]
    public string Param { get; set; }

    public override async Task SetParametersAsync(ParameterView parameters)
    {
        if (parameters.TryGetValue<string>(nameof(Param), out var value))
        {
            if (value is null)
            {
                message = "The value of 'Param' is null.";
            }
            else
            {
                message = $"The value of 'Param' is {value}.";
            }
        }

        await base.SetParametersAsync(parameters);
    }
}
```

<xref:Microsoft.AspNetCore.Components.ParameterView> には、<xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> が呼び出されるたびに、コンポーネントのパラメーター値のセットが含まれます。

<xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> の既定の実装では、対応する値が <xref:Microsoft.AspNetCore.Components.ParameterView> 内にある [`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute) または [`[CascadingParameter]` 属性](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute)を使用して、各プロパティの値が設定されます。 対応する値が <xref:Microsoft.AspNetCore.Components.ParameterView> 内にないパラメーターは、変更されないままになります。

[`base.SetParametersAsync`](xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A) が呼び出されない場合、カスタム コードでは、必要に応じて受信パラメーター値を解釈できます。 たとえば、受信したパラメーターをクラスのプロパティに割り当てる必要はありません。

イベント ハンドラーが設定されている場合は、破棄時にそれらをアンフックします。 詳細については、「[`IDisposable` を使用したコンポーネントの破棄](#component-disposal-with-idisposable)」セクションを参照してください。

### <a name="component-initialization-methods"></a>コンポーネントの初期化メソッド

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> および <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> は、コンポーネントが、<xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> でその親コンポーネントから初期パラメーターを受け取った後で初期化されるときに呼び出されます。 

コンポーネントが非同期操作を実行し、操作の完了時に更新する必要がある場合は、<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> を使用します。

同期操作の場合は、<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> をオーバーライドします。

```csharp
protected override void OnInitialized()
{
    ...
}
```

非同期操作を実行するには、<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> をオーバーライドし、操作で [`await`](/dotnet/csharp/language-reference/operators/await) 演算子を使用します。

```csharp
protected override async Task OnInitializedAsync()
{
    await ...
}
```

[コンテンツをプリレンダリングする](xref:blazor/fundamentals/signalr#render-mode) Blazor Server アプリによって、<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> が "*2 回*" 呼び出されます。

* コンポーネントが最初にページの一部として静的にレンダリングされるときに 1 回。
* ブラウザーがサーバーへの接続を確立するときに 2 回目。

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> 内で開発者コードが 2 回実行されないようにするには、「[プリレンダリング後のステートフル再接続](#stateful-reconnection-after-prerendering)」セクションを参照してください。

Blazor Server アプリをプリレンダリングしている間、ブラウザーとの接続が確立されていないため、JavaScript への呼び出しなどの特定のアクションは実行できません。 コンポーネントは、プリレンダリング時に異なるレンダリングが必要になる場合があります。 詳細については、「[アプリがプリレンダリングされていることを検出する](#detect-when-the-app-is-prerendering)」セクションを参照してください。

イベント ハンドラーが設定されている場合は、破棄時にそれらをアンフックします。 詳細については、「[`IDisposable` を使用したコンポーネントの破棄](#component-disposal-with-idisposable)」セクションを参照してください。

### <a name="after-parameters-are-set"></a>パラメーターが設定された後

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> または <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSet%2A> が呼び出されます。

* コンポーネントが <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> または <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> で初期化された後。
* 親コンポーネントが再レンダリングし、次のものを提供するとき:
  * 少なくとも 1 つのパラメーターが変更された既知のプリミティブ不変型のみ。
  * 任意の複合型のパラメーター。 フレームワークは、複合型のパラメーターの値が内部で変更されているかどうかを認識できないため、パラメーター セットは変更済みとして扱われます。

```csharp
protected override async Task OnParametersSetAsync()
{
    await ...
}
```

> [!NOTE]
> パラメーターとプロパティ値を適用するときの非同期処理は、<xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> ライフサイクル イベント中に発生する必要があります。

```csharp
protected override void OnParametersSet()
{
    ...
}
```

イベント ハンドラーが設定されている場合は、破棄時にそれらをアンフックします。 詳細については、「[`IDisposable` を使用したコンポーネントの破棄](#component-disposal-with-idisposable)」セクションを参照してください。

### <a name="after-component-render"></a>コンポーネントのレンダリング後

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> および <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> は、コンポーネントのレンダリングが完了した後に呼び出されます。 この時点で、要素およびコンポーネント参照が設定されます。 レンダリングされた DOM 要素を操作するサードパーティ製の JavaScript ライブラリをアクティブ化するなど、レンダリングされたコンテンツを使用して追加の初期化手順を行うには、この段階を使用します。

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> と <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> の `firstRender` パラメーター:

* コンポーネント インスタンスを初めて表示するときに `true` に設定されます。
* 初期化作業が確実に 1 回だけ実行されるように使用できます。

```csharp
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
    {
        await ...
    }
}
```

> [!NOTE]
> <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> ライフサイクル イベント中に、レンダリング直後の非同期作業が発生する必要があります。
>
> <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> から <xref:System.Threading.Tasks.Task> を返した場合でも、フレームワークでは、そのタスクが完了しても、コンポーネントに対してさらにレンダリング サイクルがスケジュールされることはありません。 これは、無限のレンダリング ループを回避するためです。 返されたタスクが完了すると、さらにレンダリング サイクルをスケジュールする他のライフサイクル メソッドとは異なります。

```csharp
protected override void OnAfterRender(bool firstRender)
{
    if (firstRender)
    {
        ...
    }
}
```

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> および <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> " *はサーバーでのプリレンダリング プロセス中には呼び出されません*"。 メソッドは、プリレンダリングが完了した後にコンポーネントが対話形式でレンダリングされるときに呼び出されます。 次の場合に、アプリによりプリレンダリングされます。

1. コンポーネントがサーバー上で実行され、HTTP 応答でいくつかの静的 HTML マークアップが生成される。 このフェーズでは、<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> と <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> は呼び出されません。
1. ブラウザーで `blazor.server.js` または `blazor.webassembly.js` が起動すると、コンポーネントが対話型のレンダリング モードで再起動される。 コンポーネントが再起動されると、アプリはプリレンダリング フェーズでなくなるため、<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> と <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> **が呼び出されます**。

イベント ハンドラーが設定されている場合は、破棄時にそれらをアンフックします。 詳細については、「[`IDisposable` を使用したコンポーネントの破棄](#component-disposal-with-idisposable)」セクションを参照してください。

### <a name="suppress-ui-refreshing"></a>UI 更新の抑制

<xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> をオーバーライドして、UI の更新を抑制します。 実装によって `true` が返された場合は、UI が更新されます。

```csharp
protected override bool ShouldRender()
{
    var renderUI = true;

    return renderUI;
}
```

<xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> は、コンポーネントがレンダリングされるたびに呼び出されます。

<xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> がオーバーライドされる場合でも、コンポーネントは常に最初にレンダリングされます。

詳細については、「<xref:blazor/webassembly-performance-best-practices#avoid-unnecessary-rendering-of-component-subtrees>」を参照してください。

## <a name="state-changes"></a>状態変更

<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> は、状態が変更されたことをコンポーネントに通知します。 必要に応じて、<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出すと、コンポーネントが再レンダリングされます。

<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> は、<xref:Microsoft.AspNetCore.Components.EventCallback> メソッドに対して自動的に呼び出されます。 詳細については、「<xref:blazor/components/event-handling#eventcallback>」を参照してください。

詳細については、「<xref:blazor/components/rendering>」を参照してください。

## <a name="handle-incomplete-async-actions-at-render"></a>レンダリング時の不完全な非同期アクションを処理する

ライフサイクル イベントで実行される非同期アクションは、コンポーネントがレンダリングされる前に完了していない可能性があります。 ライフサイクル メソッドの実行中に、オブジェクトが `null` またはデータが不完全に設定されている可能性があります。 オブジェクトが初期化されていることを確認するレンダリング ロジックを提供します。 オブジェクトが `null` の間、プレースホルダー UI 要素 (読み込みメッセージなど) をレンダリングします。

Blazor テンプレートの `FetchData` コンポーネントでは、予測データ (`forecasts`) を非同期に受信するように、<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> がオーバーライドされます。 `forecasts` が `null` の場合、読み込みメッセージがユーザーに表示されます。 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> によって返された `Task` が完了すると、コンポーネントは更新された状態で再レンダリングされます。

Blazor Server テンプレートの `Pages/FetchData.razor` は以下のようになります。

[!code-razor[](lifecycle/samples_snapshot/FetchData.razor?highlight=9,21,25)]

## <a name="handle-errors"></a>エラーの処理

ライフサイクル メソッド実行中のエラー処理の詳細については、「<xref:blazor/fundamentals/handle-errors#lifecycle-methods>」を参照してください。

## <a name="stateful-reconnection-after-prerendering"></a>プリレンダリング後のステートフル再接続

Blazor Server アプリで <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper.RenderMode> が <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.ServerPrerendered> の場合、コンポーネントは最初にページの一部として静的にレンダリングされます。 ブラウザーがサーバーへの接続を確立すると、コンポーネントが "*再度*" レンダリングされ、コンポーネントがやりとりできるようになります。 コンポーネントを初期化するための [`OnInitialized{Async}`](#component-initialization-methods) ライフサイクル メソッドが存在する場合、メソッドは "*2 回*" 実行されます。

* コンポーネントが静的にプリレンダリングされたとき。
* サーバー接続が確立された後。

これにより、コンポーネントが最終的にレンダリングされるときに、UI に表示されるデータが大幅に変わる可能性があります。

Blazor Server アプリ内の二重レンダリングのシナリオを回避するには、次の手順を行います。

* プリレンダリング中に状態をキャッシュし、アプリの再起動後に状態を取得するために使用できる識別子を渡します。
* 識別子をプリレンダリング中に使用して、コンポーネントの状態を保存します。
* 識別子をプリレンダリング後に使用して、キャッシュされた状態を取得します。

次のコードは、二重レンダリングを回避するテンプレートベースの Blazor Server アプリ内で更新される `WeatherForecastService` を示しています。

```csharp
public class WeatherForecastService
{
    private static readonly string[] summaries = new[]
    {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild",
        "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };
    
    public WeatherForecastService(IMemoryCache memoryCache)
    {
        MemoryCache = memoryCache;
    }
    
    public IMemoryCache MemoryCache { get; }

    public Task<WeatherForecast[]> GetForecastAsync(DateTime startDate)
    {
        return MemoryCache.GetOrCreateAsync(startDate, async e =>
        {
            e.SetOptions(new MemoryCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = 
                    TimeSpan.FromSeconds(30)
            });

            var rng = new Random();

            await Task.Delay(TimeSpan.FromSeconds(10));

            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = startDate.AddDays(index),
                TemperatureC = rng.Next(-20, 55),
                Summary = summaries[rng.Next(summaries.Length)]
            }).ToArray();
        });
    }
}
```

<xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper.RenderMode> の詳細については、「<xref:blazor/fundamentals/signalr#render-mode>」を参照してください。

## <a name="detect-when-the-app-is-prerendering"></a>アプリがプリレンダリングされていることを検出する

[!INCLUDE[](~/blazor/includes/prerendering.md)]

## <a name="component-disposal-with-idisposable"></a>`IDisposable` を使用したコンポーネントの破棄

コンポーネントが <xref:System.IDisposable> を実装している場合、そのコンポーネントが UI から削除されるときにフレームワークで[破棄メソッド](/dotnet/standard/garbage-collection/implementing-dispose)を呼び出し、アンマネージ リソースを解放することができます。 破棄は、[コンポーネントの初期化](#component-initialization-methods)中など、いつでも実行できます。 次のコンポーネントでは、[`@implements`](xref:mvc/views/razor#implements) Razor ディレクティブを使用して <xref:System.IDisposable> が実装されています。

```razor
@using System
@implements IDisposable

...

@code {
    public void Dispose()
    {
        ...
    }
}
```

オブジェクトが破棄を必要とする場合、<xref:System.IDisposable.Dispose%2A?displayProperty=nameWithType> が呼び出されるきに、ラムダを使用してオブジェクトを破棄できます。

`Pages/CounterWithTimerDisposal.razor`:

```razor
@page "/counter-with-timer-disposal"
@using System.Timers
@implements IDisposable

<h1>Counter with <code>Timer</code> disposal</h1>

<p>Current count: @currentCount</p>

@code {
    private int currentCount = 0;
    private Timer timer = new Timer(1000);

    protected override void OnInitialized()
    {
        timer.Elapsed += (sender, eventArgs) => OnTimerCallback();
        timer.Start();
    }

    private void OnTimerCallback()
    {
        _ = InvokeAsync(() =>
        {
            currentCount++;
            StateHasChanged();
        });
    }

    public void IDisposable.Dispose() => timer.Dispose();
}
```

上記の例は、「<xref:blazor/components/rendering#receiving-a-call-from-something-external-to-the-blazor-rendering-and-event-handling-system>」に示されています。

非同期の破棄タスクの場合は、<xref:System.IDisposable.Dispose> の代わりに `DisposeAsync` を使用します。

```csharp
public async ValueTask DisposeAsync()
{
    ...
}
```

> [!NOTE]
> `Dispose` では、<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> の呼び出しはサポートされていません。 <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> は、レンダラーの破棄の一部として呼び出されることがあるため、その時点での UI 更新の要求はサポートされていません。

.NET イベントからイベント ハンドラーのサブスクライブを解除します。 次の [Blazor フォーム](xref:blazor/forms-validation)の例は、`Dispose` メソッドでイベント ハンドラーの登録を解除する方法を示しています。

* プライベート フィールドとラムダのアプローチ

  [!code-razor[](lifecycle/samples_snapshot/event-handler-disposal-1.razor?highlight=23,28)]

* プライベート メソッドのアプローチ

  [!code-razor[](lifecycle/samples_snapshot/event-handler-disposal-2.razor?highlight=16,26)]

[匿名関数](/dotnet/csharp/programming-guide/statements-expressions-operators/anonymous-functions)、メソッド、または式が使用されている場合、<xref:System.IDisposable> の実装やデリゲートの登録解除を行う必要はありません。 しかし、デリゲートの登録解除に失敗することは、**イベントを公開するオブジェクトが、デリゲートを登録するコンポーネントの有効期間を超えている場合** に問題になります。 この場合、登録されたデリゲートによって元のオブジェクトが保持されているため、メモリ リークが発生します。 そのため、イベント デリゲートがすぐに破棄されることがわかっている場合にのみ、次のアプローチを使用してください。 破棄が必要なオブジェクトの有効期間が不明な場合は、前の例で示したように、デリゲート メソッドを登録し、そのデリゲートを適切に破棄します。

* 匿名のラムダ メソッドのアプローチ (明示的な破棄は不要)

  ```csharp
  private void HandleFieldChanged(object sender, FieldChangedEventArgs e)
  {
      formInvalid = !editContext.Validate();
      StateHasChanged();
  }

  protected override void OnInitialized()
  {
      editContext = new EditContext(starship);
      editContext.OnFieldChanged += (s, e) => HandleFieldChanged((editContext)s, e);
  }
  ```

* 匿名ラムダ式のアプローチ (明示的な破棄は不要)

  ```csharp
  private ValidationMessageStore messageStore;

  [CascadingParameter]
  private EditContext CurrentEditContext { get; set; }

  protected override void OnInitialized()
  {
      ...

      messageStore = new ValidationMessageStore(CurrentEditContext);

      CurrentEditContext.OnValidationRequested += (s, e) => messageStore.Clear();
      CurrentEditContext.OnFieldChanged += (s, e) => 
          messageStore.Clear(e.FieldIdentifier);
  }
  ```

  匿名ラムダ式を使用した上記のコードの完全な例は、「<xref:blazor/forms-validation#validator-components>」に示されています。

詳細については、「[アンマネージ リソースのクリーンアップ](/dotnet/standard/garbage-collection/unmanaged)」と、その後に続く `Dispose` および `DisposeAsync` メソッドの実装に関するトピックを参照してください。

## <a name="cancelable-background-work"></a>取り消し可能なバックグラウンド作業

ネットワーク呼び出し (<xref:System.Net.Http.HttpClient>) の実行やデータベースとの対話など、コンポーネントによって実行時間の長いバックグラウンド作業が実行されることがよくあります。 いくつかの状況でシステム リソースを節約するために、バックグラウンド作業を停止することをお勧めします。 たとえば、ユーザーがコンポーネントの操作を止めても、バックグラウンドの非同期操作は自動的に停止しません。

バックグラウンド作業項目の取り消しが必要になるその他の理由には、次のようなものがあります。

* 実行中のバックグラウンド タスクは、不完全な入力データまたは処理パラメーターを使用して開始されました。
* 現在実行中のバックグラウンド作業項目のセットを、新しい作業項目のセットに置き換える必要があります。
* 現在実行中のタスクの優先度を変更する必要があります。
* アプリをサーバーに再展開するには、シャットダウンする必要があります。
* サーバー リソースが制限され、バックグラウンド作業項目の再スケジュールが必要になりました。

コンポーネントに取り消し可能なバックグラウンド作業パターンを実装するには:

* <xref:System.Threading.CancellationTokenSource> と <xref:System.Threading.CancellationToken> を使用します。
* [コンポーネントの破棄](#component-disposal-with-idisposable)時と、任意の時点で手動でトークンを取り消すことで取り消しが望まれた場合は、[`CancellationTokenSource.Cancel`](xref:System.Threading.CancellationTokenSource.Cancel%2A) を呼び出して、バックグラウンド作業を取り消す必要があることを通知します。
* 非同期呼び出しが返された後、トークンに対して <xref:System.Threading.CancellationToken.ThrowIfCancellationRequested%2A> を呼び出します。

次に例を示します。

* `await Task.Delay(5000, cts.Token);` は、実行時間が長い非同期のバックグラウンド作業を表します。
* `BackgroundResourceMethod` は、メソッドが呼び出される前に `Resource` が破棄された場合に開始されない、実行時間が長いバックグラウンド メソッドを表します。

```razor
@implements IDisposable
@using System.Threading

<button @onclick="LongRunningWork">Trigger long running work</button>

@code {
    private Resource resource = new Resource();
    private CancellationTokenSource cts = new CancellationTokenSource();

    protected async Task LongRunningWork()
    {
        await Task.Delay(5000, cts.Token);

        cts.Token.ThrowIfCancellationRequested();
        resource.BackgroundResourceMethod();
    }

    public void Dispose()
    {
        cts.Cancel();
        cts.Dispose();
        resource.Dispose();
    }

    private class Resource : IDisposable
    {
        private bool disposed;

        public void BackgroundResourceMethod()
        {
            if (disposed)
            {
                throw new ObjectDisposedException(nameof(Resource));
            }
            
            ...
        }
        
        public void Dispose()
        {
            disposed = true;
        }
    }
}
```

## <a name="blazor-server-reconnection-events"></a>Blazor Server 再接続イベント

この記事で説明するコンポーネント ライフサイクル イベントは、[Blazor Server の再接続イベント ハンドラー](xref:blazor/fundamentals/signalr#reflect-the-connection-state-in-the-ui)とは別々に動作します。 Blazor Server アプリとクライアントの SignalR 接続が失われた場合には、UI の更新だけが中断されます。 その接続が再確立されると、UI の更新が再開されます。 回線ハンドラーのイベントと構成の詳細については、「<xref:blazor/fundamentals/signalr>」を参照してください。
