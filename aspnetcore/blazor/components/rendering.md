---
title: ASP.NET Core Blazor コンポーネントのレンダリング
author: guardrex
description: StateHasChanged を呼び出すタイミングなど、ASP.NET Core Blazor アプリでの Razor コンポーネントのレンダリングについて説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/16/2021
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
ms.openlocfilehash: 1d244434cd3aa7e1a49839cc0c6cecb61abbcdff
ms.sourcegitcommit: 1f35de0ca9ba13ea63186c4dc387db4fb8e541e0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2021
ms.locfileid: "104711335"
---
# <a name="aspnet-core-blazor-component-rendering"></a>ASP.NET Core Blazor コンポーネントのレンダリング

コンポーネントは、親コンポーネントによってコンポーネント階層に最初に追加されるときにレンダリングされる "*必要*" があります。 コンポーネントのレンダリングが必要なのは、このときだけです。 コンポーネントでは、その独自のロジックと規則に従って、他のタイミングでレンダリングすることが "*できます*"。

## <a name="rendering-conventions-for-componentbase"></a>`ComponentBase` のレンダリング規則

既定では、Razor コンポーネントは <xref:Microsoft.AspNetCore.Components.ComponentBase> 基底クラスから継承されます。これには次のタイミングで再レンダリングをトリガーするロジックが含まれています。

* 更新された一連の[パラメーター](xref:blazor/components/data-binding#binding-with-component-parameters)を親コンポーネントから適用した後。
* [カスケード パラメーター](xref:blazor/components/cascading-values-and-parameters)の更新された値を適用した後。
* イベントが通知され、その独自の[イベント ハンドラー](xref:blazor/components/event-handling)の 1 つを呼び出した後。
* 独自の <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> メソッドを呼び出した後 (「[Blazor ライフサイクル: 状態変更](xref:blazor/components/lifecycle#state-changes)」を参照)。

次のいずれかに該当する場合、<xref:Microsoft.AspNetCore.Components.ComponentBase> から継承されるコンポーネントの再レンダリングは、パラメーターの更新が理由でスキップされます。

* パラメーター値はすべて、既知の変更できないプリミティブ型 (`int`、`string`、`DateTime` など) であり、前にパラメーター セットが設定されてから変更されていない。
* コンポーネントの <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> メソッドから `false` が返される。

<xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> の詳細については、「<xref:blazor/webassembly-performance-best-practices#use-of-shouldrender>」を参照してください。

## <a name="control-the-rendering-flow"></a>レンダリング フローを制御する

ほとんどの場合、イベントの発生後は <xref:Microsoft.AspNetCore.Components.ComponentBase> 規則に従って適切な一部のコンポーネント再レンダリングが行われます。 通常、開発者は、再レンダリングするコンポーネントと、それらを再レンダリングするタイミングをフレームワークに指示する手動のロジックを用意する必要はありません。 フレームワークの規則の全体的な効果は、イベントを受信したコンポーネントが自動的に再レンダリングされることにあります。これにより、パラメーター値が変更された可能性のある子孫コンポーネントの再レンダリングが再帰的にトリガーされます。

フレームワークの規則がパフォーマンスに及ぼす影響と、レンダリングのためにアプリのコンポーネント階層を最適化する方法の詳細については、「<xref:blazor/webassembly-performance-best-practices#optimize-rendering-speed>」を参照してください。

## <a name="when-to-call-statehaschanged"></a>`StateHasChanged` を呼び出すタイミング

<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出すと、いつでもレンダリングをトリガーできます。 ただし、<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を不必要に呼び出さないように注意してください。これはよくある間違いで、不必要なレンダリング コストがかかる原因になります。

次の場合は、コードで <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出す必要はありません。

* <xref:Microsoft.AspNetCore.Components.ComponentBase> によってほとんどのルーチン イベント ハンドラーに対するレンダリングがトリガーされるため、同期的または非同期的にかかわらず、イベントを定期的に処理する。
* <xref:Microsoft.AspNetCore.Components.ComponentBase> によって典型的なライフサイクル イベントに対するレンダリングがトリガーされるため、同期的または非同期的にかかわらず、[`OnInitialized`](xref:blazor/components/lifecycle#component-initialization-methods) や [`OnParametersSetAsync`](xref:blazor/components/lifecycle#after-parameters-are-set) などの典型的なライフサイクル ロジックを実装する。

ただし、この記事の次のセクションで説明するケースでは、<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出すことが理にかなっている場合があります。

* [非同期ハンドラーに複数の非同期フェーズが含まれる](#an-asynchronous-handler-involves-multiple-asynchronous-phases)
* [Blazor 再レンダリングおよびイベント処理システムの外部の何かからの呼び出しを受信する](#receiving-a-call-from-something-external-to-the-blazor-rendering-and-event-handling-system)
* [特定のイベントによって再レンダリングされるサブツリーの外部でコンポーネントをレンダリングするには](#to-render-a-component-outside-the-subtree-thats-rerendered-by-a-particular-event)

### <a name="an-asynchronous-handler-involves-multiple-asynchronous-phases"></a>非同期ハンドラーに複数の非同期フェーズが含まれる

.NET でのタスクの定義方法が理由で、<xref:System.Threading.Tasks.Task> の受信側で観察できるのは、中間の非同期状態ではなく、最終的な完了だけとなっています。 したがって、<xref:Microsoft.AspNetCore.Components.ComponentBase> で再レンダリングをトリガーできるのは、<xref:System.Threading.Tasks.Task> が最初に返されたときと、<xref:System.Threading.Tasks.Task> が最終的に完了したときに限られます。 フレームワークには、他の中間ポイントでのコンポーネントの再レンダリングに対する認識はありません。 中間ポイントで再レンダリングを行いたい場合は、それらのポイントで <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出します。

クリックのたびにカウントを 4 回更新する以下の `CounterState1` コンポーネントについて考えてみましょう。

* 自動レンダリングは、`currentCount` の最初と最後のインクリメントの後に行われます。
* 手動レンダリングは、`currentCount` がインクリメントされる中間処理ポイントでフレームワークによって再レンダリングが自動的にトリガーされない場合に、<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> の呼び出しによってトリガーされます。

`Pages/CounterState1.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/rendering/CounterState1.razor?highlight=17,21,25,29)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/rendering/CounterState1.razor?highlight=17,21,25,29)]

::: moniker-end

### <a name="receiving-a-call-from-something-external-to-the-blazor-rendering-and-event-handling-system"></a>Blazor 再レンダリングおよびイベント処理システムの外部の何かからの呼び出しを受信する

<xref:Microsoft.AspNetCore.Components.ComponentBase> によって認識されるのは、その独自のライフサイクル メソッドと Blazor でトリガーされるイベントのみです。 コード内で発生する可能性がある他のイベントについては、<xref:Microsoft.AspNetCore.Components.ComponentBase> によって認識されません。 たとえば、C# カスタム データ ストアによって発生したイベントは、Blazor によって認識されません。 そのようなイベントで再レンダリングをトリガーして、更新された値を UI に表示するためには、<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出します。

<xref:System.Timers.Timer?displayProperty=fullName> を使用してカウントを一定の間隔で更新し、<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出して UI を更新する次の `CounterState2` コンポーネントについて考えてみましょう。

* `OnTimerCallback` は、Blazor で管理される再レンダリング フローまたはイベント通知の外部で実行されます。 したがって、コールバックでの `currentCount` への変更は Blazor によって認識されないため、`OnTimerCallback` で <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出す必要があります。
* コンポーネントによって <xref:System.IDisposable> が実装されます。この場合、フレームワークによって `Dispose` メソッドが呼び出されると、<xref:System.Timers.Timer> が破棄されます。 詳細については、「<xref:blazor/components/lifecycle#component-disposal-with-idisposable>」を参照してください。

コールバックは Blazor の同期コンテキストの外部で呼び出されるため、コンポーネントでは <xref:Microsoft.AspNetCore.Components.ComponentBase.InvokeAsync%2A?displayProperty=nameWithType> 内の `OnTimerCallback` のロジックをラップして、それをレンダラーの同期コンテキストに移動することが必要です。 <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出すことができるのは、レンダラーの同期コンテキストからのみであり、それ以外の場合は例外がスローされます。 これは、他の UI フレームワーク内の UI スレッドへのマーシャリングに相当します。

`Pages/CounterState2.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/rendering/CounterState2.razor?highlight=26)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/rendering/CounterState2.razor?highlight=26)]

::: moniker-end

### <a name="to-render-a-component-outside-the-subtree-thats-rerendered-by-a-particular-event"></a>特定のイベントによって再レンダリングされるサブツリーの外部でコンポーネントをレンダリングするには

UI では次のことが必要な場合があります。

1. 1 つのコンポーネントにイベントをディスパッチする。
1. 一部の状態を変更する。
1. イベントを受け取るコンポーネントの子孫ではないまったく別のコンポーネントを再レンダリングする。

このシナリオに対処する方法の 1 つは、複数のコンポーネントに挿入される "*状態管理*" クラスを、多くの場合は依存関係の挿入 (DI) サービスとして提供することです。 状態マネージャー上で 1 つのコンポーネントによってメソッドが呼び出されると、別個のコンポーネントによって受信される C# イベントがその状態マネージャーによって引き起こされます。

これらの C# イベントは Blazor レンダリング パイプラインの外部にあるため、状態マネージャーのイベントに応答してレンダリングしたい他のコンポーネント上で <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出してください。

これは、[前のセクション](#receiving-a-call-from-something-external-to-the-blazor-rendering-and-event-handling-system)での <xref:System.Timers.Timer?displayProperty=fullName> に関する前のケースと似ています。 実行コール スタックは一般にレンダラーの同期コンテキスト上に残っているため、<xref:Microsoft.AspNetCore.Components.ComponentBase.InvokeAsync%2A> の呼び出しは通常必要ありません。 <xref:Microsoft.AspNetCore.Components.ComponentBase.InvokeAsync%2A> の呼び出しは、ロジックによって同期コンテキストがエスケープされる場合にのみ必要です (<xref:System.Threading.Tasks.Task> 上で <xref:System.Threading.Tasks.Task.ContinueWith%2A> が呼び出される場合や、[`ConfigureAwait(false)`](xref:System.Threading.Tasks.Task.ConfigureAwait%2A) を使用して <xref:System.Threading.Tasks.Task> が待機される場合など)。
