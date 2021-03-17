---
title: ASP.NET Core Blazor で .NET メソッドから JavaScript 関数を呼び出す
author: guardrex
description: Blazor アプリで .NET メソッドから JavaScript 関数を呼び出す方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc, devx-track-js
ms.date: 11/25/2020
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
uid: blazor/call-javascript-from-dotnet
ms.openlocfilehash: ab342f6bdc5da68f2a2ee8d9aee29911a050056d
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586619"
---
# <a name="call-javascript-functions-from-net-methods-in-aspnet-core-blazor"></a>ASP.NET Core Blazor で .NET メソッドから JavaScript 関数を呼び出す

Blazor アプリでは、.NET メソッドから JavaScript 関数を呼び出すことも、JavaScript 関数から .NET メソッドを呼び出すこともできます。 これらのシナリオは、"*JavaScript 相互運用*" ("*JS 相互運用*") と呼ばれます。

この記事では、.NET から JavaScript 関数を呼び出す方法について説明します。 JavaScript から .NET メソッドを呼び出す方法については、「<xref:blazor/call-dotnet-from-javascript>」を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/blazor/common/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

> [!NOTE]
> `wwwroot/index.html` ファイル (Blazor WebAssembly) または `Pages/_Host.cshtml` ファイル (Blazor Server) では、終了タグ `</body>` の前に JS ファイル (`<script>` タグ) を追加します。 JS 相互運用メソッドを含む JS ファイルは、Blazor フレームワークの JS ファイルよりも先に含めるようにします。

.NET から JavaScript を呼び出すには、<xref:Microsoft.JSInterop.IJSRuntime> 抽象化を使用します。 JS 相互運用呼び出しを発行するには、コンポーネントに <xref:Microsoft.JSInterop.IJSRuntime> 抽象化を挿入します。 <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> は、JSON シリアル化可能な任意の数の引数と共に呼び出す JavaScript 関数の識別子を受け取ります。 関数の識別子は、グローバル スコープ (`window`) に関連しています。 `window.someScope.someFunction` を呼び出す場合、識別子は `someScope.someFunction` です。 関数は、呼び出す前に登録する必要はありません。 また、戻り値の型 `T` も JSON シリアル化可能である必要があります。 `T` は、返される JSON 型に最適にマップされる .NET 型と一致する必要があります。

[Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) を返す JavaScript 関数は、<xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> と共に呼び出されます。 `InvokeAsync` は Promise のラップを解除し、Promise によって待機された値を返します。

Blazor Server アプリでプリレンダリングが有効になっている場合、最初のプリレンダリング中に JavaScript を呼び出すことはできません。 JavaScript 相互運用呼び出しは、ブラウザーとの接続が確立されるまで遅延させる必要があります。 詳細については、「[Blazor Server アプリがプリレンダリングされていることを検出する](#detect-when-a-blazor-server-app-is-prerendering)」セクションを参照してください。

次の例は、JavaScript ベースのデコーダーである [`TextDecoder`](https://developer.mozilla.org/docs/Web/API/TextDecoder) に基づいています。 この例では、開発者コードから既存の JavaScript API に要件をオフロードする C# メソッドから JavaScript 関数を呼び出す方法を示します。 JavaScript 関数は、C# メソッドからバイト配列を受け取り、配列をデコードし、テキストをコンポーネントに返して表示できるようにします。

`wwwroot/index.html` (Blazor WebAssembly) または `Pages/_Host.cshtml` (Blazor Server) の `<head>` 要素内で、`TextDecoder` を使用して、渡された配列をデコードし、デコードした値を返す JavaScript 関数を提供します。

[!code-html[](call-javascript-from-dotnet/samples_snapshot/index-script-convertarray.html)]

JavaScript コードでは、前の例で示したコードのように、スクリプト ファイルへの参照を使用して JavaScript ファイル (`.js`) から読み込むこともできます。

```html
<script src="exampleJsInterop.js"></script>
```

次のコンポーネント:

* コンポーネント ボタン ( **`Convert Array`** ) が選択された場合、`JS` を使用して `convertArray` JavaScript 関数を呼び出します。
* JavaScript 関数が呼び出されると、渡された配列が文字列に変換されます。 文字列は、表示できるようにコンポーネントに返されます。

[!code-razor[](call-javascript-from-dotnet/samples_snapshot/call-js-example.razor?highlight=2,34-35)]

## <a name="ijsruntime"></a>IJSRuntime

<xref:Microsoft.JSInterop.IJSRuntime> 抽象化を使用するには、次のいずれかの方法を採用します。

* Razor コンポーネント (`.razor`) に <xref:Microsoft.JSInterop.IJSRuntime> 抽象化を挿入します。

  [!code-razor[](call-javascript-from-dotnet/samples_snapshot/inject-abstraction.razor?highlight=1)]

  `wwwroot/index.html` (Blazor WebAssembly) または `Pages/_Host.cshtml` (Blazor Server) の `<head>` 要素内で、`handleTickerChanged` JavaScript 関数を指定します。 関数は <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType> を指定して呼び出され、値を返しません。

  [!code-html[](call-javascript-from-dotnet/samples_snapshot/index-script-handleTickerChanged1.html)]

* <xref:Microsoft.JSInterop.IJSRuntime> 抽象化をクラス (`.cs`) に挿入します。

  [!code-csharp[](call-javascript-from-dotnet/samples_snapshot/inject-abstraction-class.cs?highlight=5)]

  `wwwroot/index.html` (Blazor WebAssembly) または `Pages/_Host.cshtml` (Blazor Server) の `<head>` 要素内で、`handleTickerChanged` JavaScript 関数を指定します。 関数は `JS.InvokeAsync` を指定して呼び出され、次の値を返します。

  [!code-html[](call-javascript-from-dotnet/samples_snapshot/index-script-handleTickerChanged2.html)]

* [BuildRenderTree](xref:blazor/advanced-scenarios#manual-rendertreebuilder-logic) でコンテンツを動的に生成するには、`[Inject]` 属性を使用します。

  ```razor
  [Inject]
  IJSRuntime JS { get; set; }
  ```

このトピックに添付されているクライアント側のサンプル アプリでは、ユーザー入力を受け取り、ウェルカム メッセージを表示するために、DOM とやりとりする 2 つの JavaScript 関数をアプリで使用できます。

* `showPrompt`:ユーザー入力 (ユーザーの名前) を受け入れるプロンプトを生成し、名前を呼び出し元に返します。
* `displayWelcome`:`welcome` の `id` を持つ DOM オブジェクトに、呼び出し元からのウェルカム メッセージを割り当てます。

`wwwroot/exampleJsInterop.js`:

[!code-javascript[](~/blazor/common/samples/5.x/BlazorWebAssemblySample/wwwroot/exampleJsInterop.js?highlight=2-7)]

JavaScript ファイルを参照する `<script>` タグを `wwwroot/index.html` ファイル (Blazor WebAssembly) または `Pages/_Host.cshtml` ファイル (Blazor Server) に配置します。

`wwwroot/index.html` (Blazor WebAssembly):

[!code-html[](~/blazor/common/samples/5.x/BlazorWebAssemblySample/wwwroot/index.html?highlight=22)]

`Pages/_Host.cshtml` (Blazor Server):

[!code-cshtml[](~/blazor/common/samples/5.x/BlazorServerSample/Pages/_Host.cshtml?highlight=33)]

`<script>` タグを動的に更新できないため、`<script>` タグをコンポーネント ファイル内に配置しないでください。

.NET メソッドは、<xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType> を呼び出して、`exampleJsInterop.js` ファイル内の JavaScript 関数と相互運用します。

<xref:Microsoft.JSInterop.IJSRuntime> 抽象化は、Blazor Server のシナリオを可能にするために非同期です。 アプリが Blazor WebAssembly アプリであり、JavaScript 関数を同期的に呼び出す必要がある場合は、<xref:Microsoft.JSInterop.IJSInProcessRuntime> にダウンキャストし、代わりに <xref:Microsoft.JSInterop.IJSInProcessRuntime.Invoke%2A> を呼び出します。 ほとんどの JS 相互運用ライブラリでは、確実にすべてのシナリオでライブラリを使用できるように、非同期 API を使用することをお勧めします。

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> 標準 [JavaScript モジュール](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules)で JavaScript の分離を有効にするには、「[Blazor JavaScript の分離とオブジェクト参照](#blazor-javascript-isolation-and-object-references)」セクションを参照してください。

::: moniker-end

サンプル アプリには、JS 相互運用を示すコンポーネントが含まれています。 コンポーネント:

* JavaScript プロンプトを介してユーザー入力を受け取ります。
* テキストをコンポーネントに返して処理します。
* DOM とやりとりしてウェルカム メッセージを表示する 2 番目の JavaScript 関数を呼び出します。

`Pages/JsInterop.razor`:

```razor
@page "/JSInterop"
@using {APP ASSEMBLY}.JsInteropClasses
@inject IJSRuntime JS

<h1>JavaScript Interop</h1>

<h2>Invoke JavaScript functions from .NET methods</h2>

<button type="button" class="btn btn-primary" @onclick="TriggerJsPrompt">
    Trigger JavaScript Prompt
</button>

<h3 id="welcome" style="color:green;font-style:italic"></h3>

@code {
    public async Task TriggerJsPrompt()
    {
        var name = await JS.InvokeAsync<string>(
                "exampleJsFunctions.showPrompt",
                "What's your name?");

        await JS.InvokeVoidAsync(
                "exampleJsFunctions.displayWelcome",
                $"Hello {name}! Welcome to Blazor!");
    }
}
```

プレースホルダー `{APP ASSEMBLY}` は、アプリのアプリ アセンブリ名です (例: `BlazorSample`)。

1. コンポーネントの **`Trigger JavaScript Prompt`** ボタンを選択して `TriggerJsPrompt` を実行すると、`wwwroot/exampleJsInterop.js` ファイル内に指定した JavaScript `showPrompt` 関数が呼び出されます。
1. `showPrompt` 関数は、ユーザー入力 (ユーザーの名前) を受け取ります。これは、HTML エンコードされ、コンポーネントに返されます。 コンポーネントにより、ユーザーの名前がローカル変数 `name` に格納されます。
1. `name` に格納された文字列は、ウェルカム メッセージに組み込まれます。このメッセージが JavaScript 関数 `displayWelcome` に渡され、ウェルカム メッセージが見出しタグにレンダリングされます。

## <a name="call-a-void-javascript-function"></a>void JavaScript 関数を呼び出す

次の場合に <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType> を使用します。

* [void(0)/void 0](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/void) または [undefined](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/undefined) を返す JavaScript 関数。
* JavaScript 呼び出しの結果を読み取るために .NET が不要な場合。

## <a name="detect-when-a-blazor-server-app-is-prerendering"></a>Blazor Server アプリがプリレンダリングされていることを検出する
 
[!INCLUDE[](~/blazor/includes/prerendering.md)]

## <a name="capture-references-to-elements"></a>要素への参照をキャプチャする

一部の JS 相互運用シナリオでは、HTML 要素への参照が必要です。 たとえば、UI ライブラリで初期化のための要素参照が必要な場合、`focus` や `play` などの要素でコマンドのような API の呼び出し必要になる可能性があります。

次の方法を使用して、コンポーネント内の HTML 要素への参照をキャプチャします。

* `@ref` 属性を HTML 要素に追加します。
* 名前が `@ref` 属性の値に一致する <xref:Microsoft.AspNetCore.Components.ElementReference> 型のフィールドを定義します。

次の例は、`username` `<input>` 要素への参照をキャプチャする方法を示しています。

```razor
<input @ref="username" ... />

@code {
    ElementReference username;
}
```

> [!WARNING]
> Blazor とやりとりしない空の要素のコンテンツを変化させるには、要素参照のみを使用します。 このシナリオは、サードパーティの API から要素にコンテンツが提供される場合に便利です。 Blazor は要素とやりとりしないため、Blazor の要素表現と DOM との間に競合が発生する可能性がありません。
>
> 次の例では、Blazor が DOM とやりとりしてこの要素のリスト項目 (`<li>`) を設定するため、順序なしリスト (`ul`) のコンテンツを変化させるのは "*危険*" です。
>
> ```razor
> <ul ref="MyList">
>     @foreach (var item in Todos)
>     {
>         <li>@item.Text</li>
>     }
> </ul>
> ```
>
> JS 相互運用により要素 `MyList` のコンテンツが変更され、Blazor でその要素に差分を適用しようとした場合、差分は DOM と一致しません。

<xref:Microsoft.AspNetCore.Components.ElementReference> は JS Interop 経由で JavaScript コードに渡されます。 JavaScript コードが `HTMLElement` インスタンスを受け取り、通常の DOM API で使用できます。 たとえば、次のコードでは、要素にマウス クリックを送信できるようにする .NET 拡張メソッドを定義しています。

`exampleJsInterop.js`:

```javascript
window.interopFunctions = {
  clickElement : function (element) {
    element.click();
  }
}
```

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> C# コードで [`FocusAsync`](xref:blazor/components/event-handling#focus-an-element) を使用し、Blazor フレームワークに組み込まれ、要素参照と連動する要素にフォーカスを合わせます。

::: moniker-end

値を返さない JavaScript 関数を呼び出すには、<xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType> を使用します。 次のコードは、キャプチャされた <xref:Microsoft.AspNetCore.Components.ElementReference> で前述の JavaScript 関数を呼び出し、クライアント側の `Click` イベントをトリガーします。

[!code-razor[](call-javascript-from-dotnet/samples_snapshot/component1.razor?highlight=14-15)]

拡張メソッドを使用するには、<xref:Microsoft.JSInterop.IJSRuntime> インスタンスを受け取る静的拡張メソッドを作成します。

```csharp
public static async Task TriggerClickEvent(this ElementReference elementRef, 
    IJSRuntime js)
{
    await js.InvokeVoidAsync("interopFunctions.clickElement", elementRef);
}
```

`clickElement` メソッドは、オブジェクトで直接呼び出されます。 次の例では、`TriggerClickEvent` メソッドが `JsInteropClasses` 名前空間から使用できることを前提としています。

[!code-razor[](call-javascript-from-dotnet/samples_snapshot/component2.razor?highlight=15)]

> [!IMPORTANT]
> `exampleButton` 変数は、コンポーネントがレンダリングされた後にのみ設定されます。 未入力の <xref:Microsoft.AspNetCore.Components.ElementReference> が JavaScript コードに渡された場合、JavaScript コードは `null` の値を受け取ります。 コンポーネントのレンダリングが完了した後に要素参照を操作するには、[`OnAfterRenderAsync` または `OnAfterRender` コンポーネント ライフサイクル メソッド](xref:blazor/components/lifecycle#after-component-render)を使用します。

ジェネリック型を操作して値を返す場合は、<xref:System.Threading.Tasks.ValueTask%601> を使用します。

```csharp
public static ValueTask<T> GenericMethod<T>(this ElementReference elementRef, 
    IJSRuntime js)
{
    return js.InvokeAsync<T>("exampleJsFunctions.doSomethingGeneric", elementRef);
}
```

`GenericMethod` は、型を持つオブジェクトで直接呼び出されます。 次の例では、`GenericMethod` が `JsInteropClasses` 名前空間から使用できることを前提としています。

[!code-razor[](call-javascript-from-dotnet/samples_snapshot/component3.razor?highlight=17)]

## <a name="reference-elements-across-components"></a>コンポーネント間で要素を参照する

<xref:Microsoft.AspNetCore.Components.ElementReference> は次の理由からコンポーネントで渡すことができません。

* インスタンスはコンポーネントのレンダリング後にのみ存在することが保証されます。それはコンポーネントの <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A>/<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> メソッドの実行中か実行後になります。
* <xref:Microsoft.AspNetCore.Components.ElementReference> は [`struct`](/csharp/language-reference/builtin-types/struct) であり、[コンポーネント パラメーター](xref:blazor/components/index#component-parameters)として渡すことはできません。

親コンポーネントが要素参照を他のコンポーネントで使用できるようにするために、親コンポーネントは次のことを実行できます。

* 子コンポーネントがコールバックを登録できるようにします。
* 渡された要素参照を使用して、登録されたコールバックを<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> イベント中に呼び出します。 間接的には、この方法により、子コンポーネントが親の要素参照とやりとりできるようになります。

次の Blazor WebAssembly の例でこのアプローチを示します。

`wwwroot/index.html` の `<head>` では、次のことが行われます。

```html
<style>
    .red { color: red }
</style>
```

`wwwroot/index.html` の `<body>` では、次のことが行われます。

```html
<script>
    function setElementClass(element, className) {
        /** @type {HTMLElement} **/
        var myElement = element;
        myElement.classList.add(className);
    }
</script>
```

`Pages/Index.razor` (親コンポーネント):

```razor
@page "/"

<h1 @ref="title">Hello, world!</h1>

Welcome to your new app.

<SurveyPrompt Parent="this" Title="How is Blazor working for you?" />
```

`Pages/Index.razor.cs`:

```csharp
using System;
using System.Collections.Generic;
using Microsoft.AspNetCore.Components;

namespace {APP ASSEMBLY}.Pages
{
    public partial class Index : 
        ComponentBase, IObservable<ElementReference>, IDisposable
    {
        private bool disposing;
        private IList<IObserver<ElementReference>> subscriptions = 
            new List<IObserver<ElementReference>>();
        private ElementReference title;

        protected override void OnAfterRender(bool firstRender)
        {
            base.OnAfterRender(firstRender);

            foreach (var subscription in subscriptions)
            {
                try
                {
                    subscription.OnNext(title);
                }
                catch (Exception)
                {
                    throw;
                }
            }
        }

        public void Dispose()
        {
            disposing = true;

            foreach (var subscription in subscriptions)
            {
                try
                {
                    subscription.OnCompleted();
                }
                catch (Exception)
                {
                }
            }

            subscriptions.Clear();
        }

        public IDisposable Subscribe(IObserver<ElementReference> observer)
        {
            if (disposing)
            {
                throw new InvalidOperationException("Parent being disposed");
            }

            subscriptions.Add(observer);

            return new Subscription(observer, this);
        }

        private class Subscription : IDisposable
        {
            public Subscription(IObserver<ElementReference> observer, Index self)
            {
                Observer = observer;
                Self = self;
            }

            public IObserver<ElementReference> Observer { get; }
            public Index Self { get; }

            public void Dispose()
            {
                Self.subscriptions.Remove(Observer);
            }
        }
    }
}
```

プレースホルダー `{APP ASSEMBLY}` は、アプリのアプリ アセンブリ名です (例: `BlazorSample`)。

`Shared/SurveyPrompt.razor` (子コンポーネント):

```razor
@inject IJSRuntime JS

<div class="alert alert-secondary mt-4" role="alert">
    <span class="oi oi-pencil mr-2" aria-hidden="true"></span>
    <strong>@Title</strong>

    <span class="text-nowrap">
        Please take our
        <a target="_blank" class="font-weight-bold" 
            href="https://go.microsoft.com/fwlink/?linkid=2109206">brief survey</a>
    </span>
    and tell us what you think.
</div>

@code {
    [Parameter]
    public string Title { get; set; }
}
```

`Shared/SurveyPrompt.razor.cs`:

```csharp
using System;
using Microsoft.AspNetCore.Components;

namespace {APP ASSEMBLY}.Shared
{
    public partial class SurveyPrompt : 
        ComponentBase, IObserver<ElementReference>, IDisposable
    {
        private IDisposable subscription = null;

        [Parameter]
        public IObservable<ElementReference> Parent { get; set; }

        protected override void OnParametersSet()
        {
            base.OnParametersSet();

            if (subscription != null)
            {
                subscription.Dispose();
            }

            subscription = Parent.Subscribe(this);
        }

        public void OnCompleted()
        {
            subscription = null;
        }

        public void OnError(Exception error)
        {
            subscription = null;
        }

        public void OnNext(ElementReference value)
        {
            JS.InvokeAsync<object>(
                "setElementClass", new object[] { value, "red" });
        }

        public void Dispose()
        {
            subscription?.Dispose();
        }
    }
}
```

プレースホルダー `{APP ASSEMBLY}` は、アプリのアプリ アセンブリ名です (例: `BlazorSample`)。

## <a name="harden-js-interop-calls"></a>JS 相互運用呼び出しの強化

JS 相互運用は、ネットワーク エラーにより失敗する可能性があるため、信頼性の低いものとして扱う必要があります。 既定では、Blazor Server アプリでは、サーバー上の JS 相互運用の呼び出しは 1 分後にタイムアウトします。 アプリでより積極的なタイムアウトが許容される場合は、次のいずれかの方法を使用してタイムアウトを設定します。

* `Startup.ConfigureServices` でグローバルに、タイムアウトを指定します。

  ```csharp
  services.AddServerSideBlazor(
      options => options.JSInteropDefaultCallTimeout = TimeSpan.FromSeconds({SECONDS}));
  ```

* コンポーネント コードでの呼び出しごとに、1 回の呼び出しでタイムアウトを指定できます。

  ```csharp
  var result = await JS.InvokeAsync<string>("MyJSOperation", 
      TimeSpan.FromSeconds({SECONDS}), new[] { "Arg1" });
  ```

リソース枯渇の詳細については、「<xref:blazor/security/server/threat-mitigation>」を参照してください。

[!INCLUDE[](~/blazor/includes/share-interop-code.md)]

## <a name="avoid-circular-object-references"></a>循環オブジェクト参照の回避

循環参照を含むオブジェクトは、次のいずれに対しても、クライアントでシリアル化することはできません。

* .NET メソッドの呼び出し。
* 戻り値の型に循環参照がある場合の、C# からの JavaScript メソッドの呼び出し。

詳細については、[「循環参照はサポートされていません、テイク 2」(dotnet/aspnetcore #20525)](https://github.com/dotnet/aspnetcore/issues/20525) を参照してください。

::: moniker range=">= aspnetcore-5.0"

## <a name="blazor-javascript-isolation-and-object-references"></a>Blazor JavaScript の分離とオブジェクト参照

Blazor により、標準 [JavaScript モジュール](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules)で JavaScript の分離が有効にされます。 JavaScript の分離には、次のような利点があります。

* インポートされる JavaScript によって、グローバル名前空間が汚染されなくなります。
* ライブラリおよびコンポーネントのコンシューマーは、関連する JavaScript をインポートする必要がありません。

たとえば、次の JavaScript モジュールにより、ブラウザー プロンプトを表示する JavaScript 関数がエクスポートされます。

```javascript
export function showPrompt(message) {
  return prompt(message, 'Type anything here');
}
```

前の JavaScript モジュールを静的 Web アセット (`wwwroot/exampleJsInterop.js`) として .NET ライブラリに追加し、<xref:Microsoft.JSInterop.IJSRuntime> サービスの <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> を呼び出して、そのモジュールを .NET コードにインポートします。 サービスは、次の例では `js` (表示はなし) として挿入されます。

```csharp
var module = await js.InvokeAsync<IJSObjectReference>(
    "import", "./_content/MyComponents/exampleJsInterop.js");
```

前の例の `import` 識別子は、JavaScript モジュールをインポートするために特別に使用される特殊な識別子です。 安定した静的な Web アセット パスを使用してモジュールを指定します: `./_content/{LIBRARY NAME}/{PATH UNDER WWWROOT}`。 JavaScript ファイルへの正しい静的アセット パスを作成するためには、現在のディレクトリ (`./`) に対するパス セグメントが必要です。 モジュールを動的にインポートするにはネットワーク要求が必要であるため、これは <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> を呼び出すことによってのみ、非同期的に実現できます。 `{LIBRARY NAME}` プレースホルダーは、ライブラリの名前です。 `{PATH UNDER WWWROOT}` プレースホルダーは、`wwwroot` の下にあるスクリプトへのパスです。

<xref:Microsoft.JSInterop.IJSRuntime> により、モジュールが `IJSObjectReference` としてインポートされます。これは、.NET コードから JavaScript オブジェクトへの参照を表します。 モジュールからエクスポートされた JavaScript 関数を呼び出すには、`IJSObjectReference` を使用します。

```csharp
public async ValueTask<string> Prompt(string message)
{
    return await module.InvokeAsync<string>("showPrompt", message);
}
```

`IJSInProcessObjectReference` は、関数を同期的に呼び出すことができる JavaScript オブジェクトへの参照を表します。

## <a name="use-of-javascript-libraries-that-render-ui-dom-elements"></a>UI をレンダリングする JavaScript ライブラリの使用 (DOM 要素)

ブラウザー DOM 内に表示可能なユーザー インターフェイス要素を生成する JavaScript ライブラリを使用することが必要になる場合があります。 Blazor の差分システムは、DOM 要素のツリーに対する制御に依存しており、外部コードによって DOM ツリーが変更されて、差分を適用するためのメカニズムが無効になるとエラーが発生するため、一見すると、これは困難に思えるかもしれません。 これは、Blazor に固有の制限ではありません。 差分ベースの UI フレームワークでは同じ課題が発生します。

幸い、外部で生成された UI を Blazor コンポーネントの UI に確実に埋め込むのは簡単です。 推奨される方法は、コンポーネントのコード (`.razor` ファイル) で空の要素を生成することです。 Blazor の差分システムに関する限り、要素は常に空であるため、レンダラーによって要素は再帰されず、代わりにその内容はそのままの状態になります。 これにより、外部で管理されている任意の内容を要素に設定しても安全になります。

次の例はこの概念を示したものです。 `if` ステートメント内で、`firstRender` が `true` の場合は、`myElement` に対する何らかの処理を行います。 たとえば、外部の JavaScript ライブラリを呼び出してそれを設定します。 このコンポーネント自体が削除されるまで、要素の内容が Blazor によって操作されることはありません。 コンポーネントが削除されると、コンポーネントの DOM サブツリー全体も削除されます。

```razor
<h1>Hello! This is a Blazor component rendered at @DateTime.Now</h1>

<div @ref="myElement"></div>

@code {
    HtmlElement myElement;
    
    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            ...
        }
    }
}
```

さらに詳細な例として、[オープンソースの Mapbox API](https://www.mapbox.com/) を使用して対話型マップをレンダリングする次のようなコンポーネントについて考えてみます。

```razor
@inject IJSRuntime JS
@implements IAsyncDisposable

<div @ref="mapElement" style='width: 400px; height: 300px;'></div>

<button @onclick="() => ShowAsync(51.454514, -2.587910)">Show Bristol, UK</button>
<button @onclick="() => ShowAsync(35.6762, 139.6503)">Show Tokyo, Japan</button>

@code
{
    ElementReference mapElement;
    IJSObjectReference mapModule;
    IJSObjectReference mapInstance;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            mapModule = await JS.InvokeAsync<IJSObjectReference>(
                "import", "./mapComponent.js");
            mapInstance = await mapModule.InvokeAsync<IJSObjectReference>(
                "addMapToElement", mapElement);
        }
    }

    Task ShowAsync(double latitude, double longitude)
        => mapModule.InvokeVoidAsync("setMapCenter", mapInstance, latitude, 
            longitude).AsTask();

    private async ValueTask IAsyncDisposable.DisposeAsync()
    {
        await mapInstance.DisposeAsync();
        await mapModule.DisposeAsync();
    }
}
```

`wwwroot/mapComponent.js` に配置する必要のある、対応する JavaScript モジュールは、次のとおりです。

```javascript
import 'https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.js';

// TO MAKE THE MAP APPEAR YOU MUST ADD YOUR ACCESS TOKEN FROM 
// https://account.mapbox.com
mapboxgl.accessToken = '{ACCESS TOKEN}';

export function addMapToElement(element) {
  return new mapboxgl.Map({
    container: element,
    style: 'mapbox://styles/mapbox/streets-v11',
    center: [-74.5, 40],
    zoom: 9
  });
}

export function setMapCenter(map, latitude, longitude) {
  map.setCenter([longitude, latitude]);
}
```

前の例で、文字列 `{ACCESS TOKEN}` は、 https://account.mapbox.com から取得できる有効なアクセス トークンに置き換えます。

正しいスタイルを生成するには、ホストの HTML ページ (`index.html` または `_Host.cshtml`) に次のスタイル シート タグを追加します。

```html
<link rel="stylesheet" href="https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.css" />
```

前の例で生成される対話型のマップ UI で、ユーザーは次のことができます。

* ドラッグしてスクロールまたはズームできます。
* ボタンをクリックして、あらかじめ定義されている場所に移動します。

![Mapbox による東京の市街地図。英国のブリストルと日本の東京を選択するためのボタンがあります](https://user-images.githubusercontent.com/1101362/94939821-92ef6700-04ca-11eb-858e-fff6df0053ae.png)

理解しておくべき重要な点は次のとおりです。

 * `@ref="mapElement"` が含まれる `<div>` は、Blazor に関する限り空のままになります。 したがって、やがて `mapbox-gl.js` によって設定されたり、内容が変更されたりしても安全です。 この手法は、UI をレンダリングする任意の JavaScript ライブラリで使用できます。 ページの他の部分に手を伸ばして変更しようとしない限り、サードパーティの JavaScript SPA フレームワークのコンポーネントを Blazor コンポーネントの内部に埋め込むことさえできます。 Blazor によって空と見なされない要素を、外部の JavaScript コードで変更することは、安全では "*ありません*"。
 * このアプローチを使用する場合は、Blazor によって DOM 要素が保持または破棄される方法に関する規則に留意してください。 前の例で、既定では DOM 要素が可能な限り保持されるため、コンポーネントにより安全にボタン クリック イベントが処理され、既存のマップ インスタンスが更新されます。 `@foreach` ループの内側からマップ要素のリストをレンダリングしていた場合は、`@key` を使用して、コンポーネントのインスタンスを確実に保持する必要があります。 そうしないと、リスト データを変更した場合、コンポーネントのインスタンスによって前のインスタンスの状態が望ましくない状態で保持される可能性があります。 詳細については、[@key を使用した要素とコンポーネントの保持](xref:blazor/components/index#use-key-to-control-the-preservation-of-elements-and-components)に関する記事を参照してください。

また、前の例では、JavaScript のロジックと依存関係を ES6 モジュール内にカプセル化し、`import` 識別子を使用して動的に読み込むことができる方法が示されています。 詳細については、[JavaScript の分離とオブジェクト参照](#blazor-javascript-isolation-and-object-references)に関する記事を参照してください。

::: moniker-end

## <a name="size-limits-on-js-interop-calls"></a>JS 相互運用呼び出しのサイズ制限

Blazor WebAssembly では、フレームワークによって JS 相互運用の入力と出力のサイズが制限されることはありません。

Blazor Server では、ハブ メソッドで許可される SignalR 受信メッセージの最大サイズによって、JS 相互運用呼び出しのサイズが制限されます。これは、<xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize?displayProperty=nameWithType> によって適用されます (既定値: 32 KB)。 JS から .NET への SignalR メッセージが <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize> より大きい場合は、エラーがスローされます。 このフレームワークでは、ハブからクライアントへの SignalR メッセージのサイズが制限されることはありません。 詳細については、「<xref:blazor/call-dotnet-from-javascript#size-limits-on-js-interop-calls>」を参照してください。
  
## <a name="js-modules"></a>JS モジュール

JS の分離では、JS 相互運用は、ブラウザーで [EcmaScript モジュール (ESM)](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules) ([ECMAScript 仕様](https://tc39.es/ecma262/#sec-modules)) が既定でサポートされている場合に機能します。

## <a name="unmarshalled-js-interop"></a>JS 相互運用のマーシャリングを解除する

.NET オブジェクトが JS 相互運用のためにシリアル化され、次のいずれかに該当する場合、Blazor WebAssembly コンポーネントのパフォーマンスが低下する可能性があります。

* 大量の .NET オブジェクトが短時間でシリアル化される。 例:JS 相互運用の呼び出しは、マウス ホイールの回転など、入力デバイスの動きに基づいて行われます。
* 大きな .NET オブジェクトまたは多数の .NET オブジェクトを JS 相互運用のためにシリアル化する必要がある。 例:JS 相互運用の呼び出しでは、数十個のファイルをシリアル化する必要があります。

<xref:Microsoft.JSInterop.IJSUnmarshalledObjectReference> は、.NET データをシリアル化するオーバーヘッドなしで関数を呼び出すことができる JavaScript オブジェクトへの参照を表します。

次に例を示します。

* 文字列と整数を含む [struct](/dotnet/csharp/language-reference/builtin-types/struct) は、シリアル化されずに JavaScript に渡されます。
* JavaScript 関数によってデータが処理され、ブール値または文字列が呼び出し元に返されます。
* JavaScript 文字列を .NET `string` オブジェクトに直接変換することはできません。 `unmarshalledFunctionReturnString` 関数によって `BINDING.js_string_to_mono_string` が呼び出され、Javascript 文字列の変換が管理されます。

> [!NOTE]
> 次の例は、JavaScript に渡された [struct](/dotnet/csharp/language-reference/builtin-types/struct) によってコンポーネントのパフォーマンスは低下しないので、このシナリオの一般的な使用例ではありません。 この例では、シリアル化されていない .NET データを渡すための概念を示すためだけに、小さなオブジェクトを使用しています。

`wwwroot/index.html` の `<script>` ブロックまたは `wwwroot/index.html` から参照される外部 Javascript ファイルの内容:

```javascript
window.returnJSObjectReference = () => {
    return {
        unmarshalledFunctionReturnBoolean: function (fields) {
            const name = Blazor.platform.readStringField(fields, 0);
            const year = Blazor.platform.readInt32Field(fields, 8);

            return name === "Brigadier Alistair Gordon Lethbridge-Stewart" &&
                year === 1968;
        },
        unmarshalledFunctionReturnString: function (fields) {
            const name = Blazor.platform.readStringField(fields, 0);
            const year = Blazor.platform.readInt32Field(fields, 8);

            return BINDING.js_string_to_mono_string(`Hello, ${name} (${year})!`);
        }
    };
}
```

> [!WARNING]
> `js_string_to_mono_string` 関数の名前、動作、および存在は、.NET の今後のリリースで変更される可能性があります。 次に例を示します。
>
> * 関数の名前が変更される可能性があります。
> * フレームワークによる文字列の自動変換を優先して、関数自体が削除される可能性があります。

`Pages/UnmarshalledJSInterop.razor` (URL: `/unmarshalled-js-interop`):

```razor
@page "/unmarshalled-js-interop"
@using System.Runtime.InteropServices
@using Microsoft.JSInterop
@inject IJSRuntime JS

<h1>Unmarshalled JS interop</h1>

@if (callResultForBoolean)
{
    <p>JS interop was successful!</p>
}

@if (!string.IsNullOrEmpty(callResultForString))
{
    <p>@callResultForString</p>
}

<p>
    <button @onclick="CallJSUnmarshalledForBoolean">
        Call Unmarshalled JS & Return Boolean
    </button>
    <button @onclick="CallJSUnmarshalledForString">
        Call Unmarshalled JS & Return String
    </button>
</p>

<p>
    <a href="https://www.doctorwho.tv">Doctor Who</a>
    is a registered trademark of the <a href="https://www.bbc.com/">BBC</a>.
</p>

@code {
    private bool callResultForBoolean;
    private string callResultForString;

    private void CallJSUnmarshalledForBoolean()
    {
        var unmarshalledRuntime = (IJSUnmarshalledRuntime)JS;

        var jsUnmarshalledReference = unmarshalledRuntime
            .InvokeUnmarshalled<IJSUnmarshalledObjectReference>(
                "returnJSObjectReference");

        callResultForBoolean = 
            jsUnmarshalledReference.InvokeUnmarshalled<InteropStruct, bool>(
                "unmarshalledFunctionReturnBoolean", GetStruct());
    }

    private void CallJSUnmarshalledForString()
    {
        var unmarshalledRuntime = (IJSUnmarshalledRuntime)JS;

        var jsUnmarshalledReference = unmarshalledRuntime
            .InvokeUnmarshalled<IJSUnmarshalledObjectReference>(
                "returnJSObjectReference");

        callResultForString = 
            jsUnmarshalledReference.InvokeUnmarshalled<InteropStruct, string>(
                "unmarshalledFunctionReturnString", GetStruct());
    }

    private InteropStruct GetStruct()
    {
        return new InteropStruct
        {
            Name = "Brigadier Alistair Gordon Lethbridge-Stewart",
            Year = 1968,
        };
    }

    [StructLayout(LayoutKind.Explicit)]
    public struct InteropStruct
    {
        [FieldOffset(0)]
        public string Name;

        [FieldOffset(8)]
        public int Year;
    }
}
```

`IJSUnmarshalledObjectReference` インスタンスが C# コードで破棄されない場合は、JavaScript で破棄することができます。 次の `dispose` 関数を使用すると、JavaScript から呼び出されたときにオブジェクト参照が破棄されます。

```javascript
window.exampleJSObjectReferenceNotDisposedInCSharp = () => {
    return {
        dispose: function () {
            DotNet.disposeJSObjectReference(this);
        },

        ...
    };
}
```

配列型は、`js_typed_array_to_array` を使用して JavaScript オブジェクトから .NET オブジェクトに変換できますが、JavaScript 配列は型指定された配列である必要があります。 JavaScript の配列は、C# コードで .NET オブジェクト配列 (`object[]`) として読み取ることができます。

文字列配列などの他のデータ型は変換できますが、新しい Mono 配列オブジェクト (`mono_obj_array_new`) を作成し、その値 (`mono_obj_array_set`) を設定する必要があります。

> [!WARNING]
> `js_typed_array_to_array`、`mono_obj_array_new`、`mono_obj_array_set` など、Blazor フレームワークによって提供される JavaScript 関数は、.NET の今後のリリースで、名前の変更、動作の変更、または削除の対象となる可能性があります。

## <a name="additional-resources"></a>その他の技術情報

* <xref:blazor/call-dotnet-from-javascript>
* [`InteropComponent.razor` の例 (dotnet/AspNetCore GitHub リポジトリの `main` ブランチ)](https://github.com/dotnet/AspNetCore/blob/main/src/Components/test/testassets/BasicTestApp/InteropComponent.razor): `main` ブランチは、ASP.NET Core の製品単位の現在の開発を表します。 別のリリース (`release/5.0`) のブランチを選択するには、 **[Switch branches or tags]\(ブランチまたはタグの切り替え\)** ドロップダウン リストを使用して、そのブランチを選択します。
