---
title: ASP.NET Core Blazor コンポーネントのレンダリング
author: guardrex
description: StateHasChanged を呼び出すタイミングなど、ASP.NET Core Blazor アプリでの Razor コンポーネントのレンダリングについて説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/13/2021
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
uid: blazor/components/rendering
ms.openlocfilehash: 1a4d4116b8a6d9266bbacbbdd8f20dc49b4e1db0
ms.sourcegitcommit: 063a06b644d3ade3c15ce00e72a758ec1187dd06
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98253860"
---
# <a name="aspnet-core-no-locblazor-component-rendering"></a>ASP.NET Core Blazor コンポーネントのレンダリング

作成者: [Steve Sanderson](https://github.com/SteveSandersonMS)

コンポーネントは、その親コンポーネントによってコンポーネント階層に最初に追加されるときにレンダリングされる "*必要*" があります。 コンポーネントのレンダリングが厳密に必要なのは、このときだけです。

コンポーネントでは、その独自のロジックと規則に従って、他の任意のタイミングでレンダリングすることを選択 "*できます*"。

## <a name="conventions-for-componentbase"></a>`ComponentBase` の規則

既定では、Razor コンポーネント (`.razor`) は <xref:Microsoft.AspNetCore.Components.ComponentBase> 基底クラスから継承されます。これには次のタイミングで再レンダリングをトリガーするロジックが含まれています。

* 更新された一連のパラメーターを親コンポーネントから適用した後。
* カスケード パラメーターの更新された値を適用した後。
* イベントが通知され、その独自のイベント ハンドラーの 1 つを呼び出した後。
* その独自の <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> メソッドの呼び出しの後。

次のいずれかに該当する場合、<xref:Microsoft.AspNetCore.Components.ComponentBase> から継承されるコンポーネントの再レンダリングは、パラメーターの更新が理由でスキップされます。

* パラメーター値はすべて、既知の変更できないプリミティブ型 (`int`、`string`、`DateTime` など) であり、前にパラメーター セットが設定されてから変更されていない。
* コンポーネントの <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> メソッドから `false` が返される。

<xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> の詳細については、「<xref:blazor/webassembly-performance-best-practices#use-of-shouldrender>」を参照してください。

## <a name="control-the-rendering-flow"></a>レンダリング フローを制御する

ほとんどの場合、イベントの発生後は <xref:Microsoft.AspNetCore.Components.ComponentBase> 規則に従って適切な一部のコンポーネント再レンダリングが行われます。 通常、開発者は、再レンダリングするコンポーネントと、それらを再レンダリングするタイミングをフレームワークに指示する手動のロジックを用意する必要はありません。 フレームワークの規則の全体的な効果は、イベントを受信したコンポーネントが自動的に再レンダリングされることにあります。これにより、パラメーター値が変更された可能性のある子孫コンポーネントの再レンダリングが再帰的にトリガーされます。

フレームワークの規則がパフォーマンスに及ぼす影響と、アプリのコンポーネント階層を最適化する方法の詳細については、「<xref:blazor/webassembly-performance-best-practices#optimize-rendering-speed>」を参照してください。

## <a name="when-to-call-statehaschanged"></a>`StateHasChanged` を呼び出すタイミング

<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A?displayProperty=nameWithType> メソッドを使用すると、いつでもレンダリングをトリガーできます。 ただし、よくある間違いですが、<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を不必要に呼び出さないように注意してください。これは不要なレンダリング コストがかかるためです。

次の場合に <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出す必要は "*ありません*"。

* <xref:Microsoft.AspNetCore.Components.ComponentBase> によってほとんどのルーチン イベント ハンドラーに対するレンダリングがトリガーされるため、同期的または非同期的にかかわらず、イベントを定期的に処理する。
* <xref:Microsoft.AspNetCore.Components.ComponentBase> によって典型的なライフサイクル イベントに対するレンダリングがトリガーされるため、同期的または非同期的にかかわらず、[`OnInitialized`](xref:blazor/components/lifecycle#component-initialization-methods) や [`OnParametersSetAsync`](xref:blazor/components/lifecycle#after-parameters-are-set) などの典型的なライフサイクル ロジックを実装する。

ただし、次のセクションで説明するケースでは、<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出すことが理にかなっている場合があります。

* [非同期ハンドラーに複数の非同期フェーズが含まれる](#an-asynchronous-handler-involves-multiple-asynchronous-phases)
* [Blazor 再レンダリングおよびイベント処理システムの外部の何かからの呼び出しを受信する](#receiving-a-call-from-something-external-to-the-blazor-rendering-and-event-handling-system)
* [特定のイベントによって再レンダリングされるサブツリーの外部でコンポーネントをレンダリングするには](#to-render-a-component-outside-the-subtree-thats-rerendered-by-a-particular-event)

### <a name="an-asynchronous-handler-involves-multiple-asynchronous-phases"></a>非同期ハンドラーに複数の非同期フェーズが含まれる

クリックのたびにカウントを 4 回更新する以下の `Counter` コンポーネントについて考えてみましょう。

`Pages/Counter.razor`:

```razor
@page "/counter"

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;

    private async Task IncrementCount()
    {
        currentCount++;
        // Renders here automatically

        await Task.Delay(1000);
        currentCount++;
        StateHasChanged();

        await Task.Delay(1000);
        currentCount++;
        StateHasChanged();

        await Task.Delay(1000);
        currentCount++;
        // Renders here automatically
    }
}
```

.NET でのタスクの定義方法が理由で、<xref:System.Threading.Tasks.Task> の受信側で観察できるのは、中間の非同期状態ではなく、最終的な完了だけとなっています。 したがって、<xref:Microsoft.AspNetCore.Components.ComponentBase> で再レンダリングをトリガーできるのは、<xref:System.Threading.Tasks.Task> が最初に返されたときと、<xref:System.Threading.Tasks.Task> が最終的に完了したときに限られます。 他の中間点での再レンダリングに対する認識はありません。 中間点で再レンダリングを行いたい場合は、<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を使用します。

### <a name="receiving-a-call-from-something-external-to-the-no-locblazor-rendering-and-event-handling-system"></a>Blazor 再レンダリングおよびイベント処理システムの外部の何かからの呼び出しを受信する

<xref:Microsoft.AspNetCore.Components.ComponentBase> によって認識されるのは、その独自のライフサイクル メソッドと Blazor でトリガーされるイベントのみです。 ご利用のコード内で発生する可能性がある他のイベントについては、<xref:Microsoft.AspNetCore.Components.ComponentBase> によって認識されません。 たとえば、C# カスタム データ ストアによって発生したイベントは、Blazor によって認識されません。 そのようなイベントで再レンダリングをトリガーして、更新された値を UI に表示するためには、<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を使用します。

<xref:System.Timers.Timer?displayProperty=fullName> を使用してカウントを一定の間隔で更新し、<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出して UI を更新する次の `Counter` コンポーネントについて、別のユース ケースで考えてみましょう。

`Pages/Counter.razor`:

```razor
@page "/counter"
@using System.Timers
@implements IDisposable

<p>Current count: @currentCount</p>

@code {
    private int currentCount = 0;
    private Timer timer = new Timer(1000);

    protected override void OnInitialized()
    {
        timer.Elapsed += (sender, eventArgs) => OnTimerCallback();
        timer.Start();
    }

    void OnTimerCallback()
    {
        _ = InvokeAsync(() =>
        {
            currentCount++;
            StateHasChanged();
        });
    }

    void IDisposable.Dispose() => timer.Dispose();
}
```

前の例において、コールバックで `currentCount` への変更が Blazor によって認識されることはないため、`OnTimerCallback` では <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出す必要があります。 `OnTimerCallback` は、Blazor で管理される再レンダリング フローまたはイベント通知の外部で実行されます。

同様に、コールバックは Blazor の同期コンテキストの外部で呼び出されるため、<xref:Microsoft.AspNetCore.Components.ComponentBase.InvokeAsync%2A?displayProperty=nameWithType> 内のロジックをラップして、それをレンダラーの同期コンテキストに移動することが必要です。 <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出すことができるのは、レンダラーの同期コンテキストからのみであり、それ以外の場合は例外がスローされます。 これは、他の UI フレームワーク内の UI スレッドへのマーシャリングに相当します。

### <a name="to-render-a-component-outside-the-subtree-thats-rerendered-by-a-particular-event"></a>特定のイベントによって再レンダリングされるサブツリーの外部でコンポーネントをレンダリングするには

ご利用の UI には、1 つのコンポーネントへのイベントのディスパッチ、何らかの状態の変更、およびイベントを受け取るコンポーネントの子孫ではないまったく異なるコンポーネントの再レンダリング、が必要になる場合があります。

このシナリオに対処する方法の 1 つとして、"*状態管理*" クラス (たとえば、DI サービス) を複数のコンポーネントに挿入することが挙げられます。 状態マネージャー上で 1 つのコンポーネントによってメソッドが呼び出されると、別個のコンポーネントによって受信される C# イベントがその状態マネージャーによって引き起こされる可能性があります。

これらの C# イベントは Blazor レンダリング パイプラインの外部にあるため、状態マネージャーのイベントに応答してレンダリングしたい他のコンポーネント上で <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出してください。

これは、[前のセクション](#receiving-a-call-from-something-external-to-the-blazor-rendering-and-event-handling-system)での <xref:System.Timers.Timer?displayProperty=fullName> に関する前のケースと似ています。 実行コール スタックは一般にレンダラーの同期コンテキスト上に残っているため、<xref:Microsoft.AspNetCore.Components.ComponentBase.InvokeAsync%2A> は通常必要ありません。 <xref:Microsoft.AspNetCore.Components.ComponentBase.InvokeAsync%2A> は、ロジックによって同期コンテキストがエスケープされる場合にのみ必要です (<xref:System.Threading.Tasks.Task> 上で <xref:System.Threading.Tasks.Task.ContinueWith%2A> が呼び出される場合や、[`ConfigureAwait(false)`](xref:System.Threading.Tasks.Task.ConfigureAwait%2A) を使用して <xref:System.Threading.Tasks.Task> が待機される場合など)。
