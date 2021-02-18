---
title: ASP.NET Core Blazor WebAssembly パフォーマンスに関するベスト プラクティス
author: pranavkm
description: ASP.NET Core Blazor WebAssembly アプリのパフォーマンスの向上、および一般的なパフォーマンスの問題回避のためのヒント。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc, devx-track-js
ms.date: 10/09/2020
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
uid: blazor/webassembly-performance-best-practices
ms.openlocfilehash: 1860a36ba4122fb39ca92797da9a44b282afa793
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280657"
---
# <a name="aspnet-core-blazor-webassembly-performance-best-practices"></a>ASP.NET Core Blazor WebAssembly パフォーマンスに関するベスト プラクティス

Blazor WebAssembly は、最も現実的なアプリケーションの UI シナリオでハイ パフォーマンスを実現できるように、慎重に設計および最適化されています。 ただし、最適な結果を得るには、開発者が適切なパターンと機能を使用する必要があります。 次の点を考慮してください。

* **ランタイムのスループット**:.NET コードは WebAssembly ランタイム内のインタープリター上で実行されるため、CPU スループットが制限されます。 要求の厳しいシナリオでは、[レンダリング速度の最適化](#optimize-rendering-speed)によってアプリが恩恵を受けます。
* **起動時間**:アプリによって .NET ランタイムがブラウザーに転送されるため、[アプリケーションのダウンロード サイズを最小化する](#minimize-app-download-size)機能を使用することが重要です。

## <a name="optimize-rendering-speed"></a>レンダリング速度を最適化する

以下のセクションでは、レンダリング ワークロードを最小化し、UI の応答性を向上させるための推奨事項を示します。 このアドバイスに従うことで、UI のレンダリング速度を容易に "*10 倍以上改善*" できる可能性があります。

### <a name="avoid-unnecessary-rendering-of-component-subtrees"></a>コンポーネントのサブツリーの不要なレンダリングを避ける

実行時に、コンポーネントは階層として存在します。 ルート コンポーネントには子コンポーネントがあります。 さらに、ルートの子にはそれぞれの子コンポーネントがあり、同様に続きます。 ユーザーがボタンを選択するなどのイベントが発生すると、Blazor によってどのコンポーネントがレンダリングされるかが、次のように決定されます。

1. イベント自体は、イベントのハンドラーをレンダリングしたコンポーネントにディスパッチされます。 イベント ハンドラーの実行後、そのコンポーネントは再レンダリングされます。
1. いずれかのコンポーネントが再レンダリングされるたびに、その各子コンポーネントに対してパラメーター値の新しいコピーが提供されます。
1. 各コンポーネントでは、新しいパラメーター値のセットを受け取ったときに、レンダリングするかどうかが選択されます。 既定では、パラメーター値が変更されている可能性がある場合 (たとえば、変更可能なオブジェクトの場合など)、コンポーネントはレンダリングされます。

このシーケンスの最後の 2 つの手順は、コンポーネント階層を下って再帰的に繰り返されます。 多くの場合、サブツリー全体が再レンダリングされます。 これは、上位レベルのコンポーネントを対象とするイベントによって、コストがかかる再レンダリング プロセスが発生する可能性があることを意味します。それより下位のすべてを再レンダリングする必要があるためです。

このプロセスを中断し、特定のサブツリーへの再帰的なレンダリングを防止するには、次のいずれかの操作を行います。

* 特定のコンポーネントに対する全パラメーターを、変更不可のプリミティブ型 (たとえば `string`、`int`、`bool`、`DateTime` など) にします。 変更を検出するための組み込みロジックでは、これらのパラメーター値がどれも変更されていない場合、再レンダリングが自動的にスキップされます。 `<Customer CustomerId="@item.CustomerId" />` (`CustomerId` は `int` 値) を持つ子コンポーネントをレンダリングする場合、`item.CustomerId` が変更された場合を除き、再レンダリングされることはありません。
* カスタム モデル型、イベントのコールバック、または <xref:Microsoft.AspNetCore.Components.RenderFragment> 値など、非プリミティブ型のパラメーター値を受け入れる必要がある場合は、<xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> をオーバーライドしてレンダリングするかどうかの決定を制御できます。これについては、「[`ShouldRender` の使用](#use-of-shouldrender)」セクションで説明します。

サブツリー全体の再レンダリングをスキップすることで、イベントが発生したときのレンダリング コストの大半を取り除くことができる場合があります。

子コンポーネントを具体的に排除して、UI のその部分の再レンダリングをスキップできるようにしたい場合があります。 これは、親コンポーネントのレンダリング コストを削減するための有効な方法です。

#### <a name="use-of-shouldrender"></a>`ShouldRender` の使用

(パラメーター値に関係なく) 初回のレンダリング後に変更されることがない UI のみのコンポーネントを作成する場合は、`false` を返すように <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> を構成します。

```razor
@code {
    protected override bool ShouldRender() => false;
}
```

コンポーネントの再レンダリングが、そのパラメーター値が特定の方法で変更された場合にのみ必要な場合は、プライベート フィールドを使用して変更を検出するために必要な情報を追跡できます。 次の例の場合、`shouldRender` は、再レンダリングを発生させる必要がある任意の変更のチェックに基づいています。 `prevOutboundFlightId` と `prevInboundFlightId` によって、次に発生する可能性のある更新についての情報を追跡します。

```razor
@code {
    [Parameter]
    public FlightInfo OutboundFlight { get; set; }
    
    [Parameter]
    public FlightInfo InboundFlight { get; set; }

    private int prevOutboundFlightId;
    private int prevInboundFlightId;
    private bool shouldRender;

    protected override void OnParametersSet()
    {
        shouldRender = OutboundFlight.FlightId != prevOutboundFlightId
            || InboundFlight.FlightId != prevInboundFlightId;

        prevOutboundFlightId = OutboundFlight.FlightId;
        prevInboundFlightId = InboundFlight.FlightId;
    }

    protected override bool ShouldRender() => shouldRender;

    // Note that 
}
```

上のコードでは、イベント ハンドラーで `shouldRender` を `true` に設定して、イベント後にコンポーネントが再レンダリングされるようにすることもできます。

ほとんどのコンポーネントでは、このレベルの手動制御は必要ありません。 サブツリーのレンダリングに特にコストがかかり、UI の遅延の原因となっている場合にのみ、それらのサブツリーのレンダリングをスキップすることを検討してください。

詳細については、<xref:blazor/components/lifecycle> を参照してください。

::: moniker range=">= aspnetcore-5.0"

### <a name="virtualization"></a>仮想化

何千ものエントリが含まれたリストやグリッドなど、ループ内で大量の UI をレンダリングする場合、膨大な量のレンダリング操作によって UI のレンダリングに遅延が発生し、ユーザー エクスペリエンスが低下する可能性があります。 ユーザーがスクロールなしで一度に少数の要素しか表示できない場合、現在表示されていない要素のレンダリングに多くの時間を費やすことは無駄のように見えます。

この問題に対処するために、Blazor には `Virtualize` コンポーネントが用意されています。これを使用すると、任意のサイズを持つリストの外観とスクロール動作が作成されますが、現在のスクロール ビューポート内のリスト項目のみがレンダリングされます。 これはたとえば、アプリに 100,000 個のエントリを持つリストを含めて、一度に表示される 20 項目のレンダリング コストのみを負担することが可能であることを意味します。 `Virtualize` コンポーネントを使用すると、UI パフォーマンスを大幅にスケールアップできます。

詳細については、<xref:blazor/components/virtualization> を参照してください。

::: moniker-end

### <a name="create-lightweight-optimized-components"></a>軽量で最適化されたコンポーネントを作成する

ほとんどの Blazor コンポーネントでは、積極的な最適化を行う必要はありません。 これは、ほとんどのコンポーネントは UI で繰り返し表示されず、高頻度で再レンダリングされることがないためです。 たとえば、`@page` コンポーネントや、ダイアログやフォームなどの高レベルの UI 部分を表すコンポーネントは、通常は一度に 1 つだけ表示され、ユーザー ジェスチャへの応答としてのみ再レンダリングされます。 これらのコンポーネントによってレンダリングの高ワークロードが作成されることはないため、レンダリングのパフォーマンスについてあまり心配することなく、必要なフレームワーク機能を自由に組み合わせて使用できます。

ただし、大規模に繰り返す必要のあるコンポーネントを構築する一般的なシナリオもあります。 次に例を示します。

* 入れ子になった大規模なフォームには、何百もの個別の入力、ラベル、その他の要素が含まれる場合があります。
* グリッドには何千ものセルが含まれる場合があります。
* 散布図には何百万ものデータ ポイントが含まれる場合があります。

各ユニットを個別のコンポーネント インスタンスとしてモデリングする場合は、その数が多くなり、レンダリング パフォーマンスが重要になります。 このセクションでは、UI の速度と応答性を維持できるように、これらのコンポーネントを軽量にするためのアドバイスを提供します。

#### <a name="avoid-thousands-of-component-instances"></a>何千ものコンポーネント インスタンスを避ける

各コンポーネントは、その親と子とは無関係にレンダリングできる独立した島です。 UI をコンポーネント階層に分割する方法を選択することで、UI レンダリングの粒度を制御できます。 これはパフォーマンスにとって良い場合も悪い場合もあります。

* UI をより多くのコンポーネントに分割することで、イベントの発生時に再レンダリングする UI の部分を小さくすることができます。 たとえば、ユーザーがテーブルの行のボタンをクリックしたときに、ページまたはテーブル全体ではなく、その 1 つの行だけを再レンダリングすることができます。
* ただし、それぞれの追加コンポーネントには、その独立した状態とレンダリング ライフサイクルを処理するための、追加のメモリと CPU オーバーヘッドが伴います。

.NET 5 で Blazor WebAssembly のパフォーマンスを調整する場合、コンポーネント インスタンスあたり約 0.06 ミリ秒のレンダリング オーバーヘッドが測定されました。 これは、一般的なノート PC 上で実行される、3 つのパラメーターを受け取るシンプルなコンポーネントに基づいています。 内部的には、オーバーヘッドの大部分が、ディクショナリからのコンポーネントごとの状態の取得と、パラメーターの受け渡しに起因します。 乗算により、2,000 個のコンポーネント インスタンスを新たに追加するとレンダリング時間が 0.12 秒長くなり、ユーザーが UI を低速に感じ始めることがわかります。

コンポーネントをより軽量にしてその数を増やすこともできますが、多くの場合、より強力な手法はコンポーネントの数を増やすことではありません。 以下のセクションでは、2 つの方法について説明します。

##### <a name="inline-child-components-into-their-parents"></a>子コンポーネントをその親にインラインで挿入する

子コンポーネントのシーケンスをレンダリングする次のコンポーネントについて考えてみます。

```razor
<div class="chat">
    @foreach (var message in messages)
    {
        <ChatMessageDisplay Message="@message" />
    }
</div>
```

上のコード例では、`<ChatMessageDisplay>` コンポーネントは次のようなファイル `ChatMessageDisplay.razor` で定義されています。

```razor
<div class="chat-message">
    <span class="author">@Message.Author</span>
    <span class="text">@Message.Text</span>
</div>

@code {
    [Parameter]
    public ChatMessage Message { get; set; }
}
```

何千ものメッセージが一度に表示されない限り、上の例は正常に動作し、パフォーマンスも良好です。 何千ものメッセージを一度に表示するには、個別の `ChatMessageDisplay` コンポーネントを "*取り出さない*" ことを検討してください。 代わりに、レンダリングを直接親にインラインで挿入します。

```razor
<div class="chat">
    @foreach (var message in messages)
    {
        <div class="chat-message">
            <span class="author">@message.Author</span>
            <span class="text">@message.Text</span>
        </div>
    }
</div>
```

これにより、個別にレンダリングすることができなくなる代わりに、コンポーネントごとに多くの子コンポーネントをレンダリングするオーバーヘッドを回避できます。

##### <a name="define-reusable-renderfragments-in-code"></a>コード内で再利用可能な `RenderFragments` を定義する

レンダリング ロジックを再利用するための方法として、子コンポーネントを純粋に取り出すことができます。 その場合でも、実際のコンポーネントを宣言することなくレンダリング ロジックを再利用できます。 コンポーネントの `@code` ブロックでは、UI を出力し、どこからでも呼び出すことができる <xref:Microsoft.AspNetCore.Components.RenderFragment> を定義できます。

```razor
<h1>Hello, world!</h1>

@RenderWelcomeInfo

@code {
    private RenderFragment RenderWelcomeInfo = __builder =>
    {
        <div>
            <p>Welcome to your new app!</p>

            <SurveyPrompt Title="How is Blazor working for you?" />
        </div>
    };
}
```

上の例で示されているように、コンポーネントでは `@code` ブロック内およびその外部のコードからマークアップを出力できます。 この方法では、コンポーネントの通常のレンダリング出力内で、必要に応じて複数の場所でレンダリングできる <xref:Microsoft.AspNetCore.Components.RenderFragment> デリゲートを定義します。 デリゲートでは、<xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> 型の `__builder` というパラメーターを受け取り、Razor コンパイラがレンダリング指示を生成できるようにする必要があります。

これを複数のコンポーネント間で再利用できるようにするには、`public static` メンバーとして宣言することを検討してください。

```razor
public static RenderFragment SayHello = __builder =>
{
    <h1>Hello!</h1>
};
```

これで、関連付けられていないコンポーネントから呼び出すことができるようになりました。 この手法は、コンポーネントごとのオーバーヘッドなしでレンダリングされる、再利用可能なマークアップ スニペットのライブラリを構築する場合に便利です。

<xref:Microsoft.AspNetCore.Components.RenderFragment> デリゲートでは、パラメーターを受け取ることもできます。 前の例の `ChatMessageDisplay` コンポーネントと同等のものを作成するには、次のようにします。

```razor
<div class="chat">
    @foreach (var message in messages)
    {
        @ChatMessageDisplay(message)
    }
</div>

@code {
    private RenderFragment<ChatMessage> ChatMessageDisplay = message => __builder =>
    {
        <div class="chat-message">
            <span class="author">@message.Author</span>
            <span class="text">@message.Text</span>
        </div>
    };
}
```

この方法には、コンポーネントごとのオーバーヘッドなしでレンダリング ロジックを再利用できるという利点があります。 ただし、その UI のサブツリーを個別に更新することはできません。また、コンポーネント境界がないため、親がレンダリングされるときに UI のそのサブツリーのレンダリングをスキップすることもできません。

次の例の `TitleTemplate` など、フィールド初期化子で参照できない非静的フィールド、メソッド、またはプロパティについては、<xref:Microsoft.AspNetCore.Components.RenderFragment> のフィールドの代わりにプロパティを使用します。

```csharp
protected RenderFragment DisplayTitle => __builder =>
{
    <div>
        @TitleTemplate
    </div>   
};
```

#### <a name="dont-receive-too-many-parameters"></a>受け取るパラメーターの数が多すぎないようにする

1 つのコンポーネントが非常に頻繁に繰り返される場合 (数百回、数千回など) は、各パラメーターを受け渡しするオーバーヘッドが増大していくことに注意してください。

多すぎるパラメーターによってパフォーマンスが著しく制限されることはまれですが、1 つの要因になる可能性があります。 グリッド内で 1,000 回レンダリングされる `<TableCell>` コンポーネントの場合、渡される追加パラメーターごとに、全体のレンダリング コストが約 15 ミリ秒増大する可能性があります。 各セルが 10 個のパラメーターを受け取る場合、コンポーネントのレンダリングごとにパラメーターの受け渡しに 150 ミリ秒かかります。したがっておそらく 150,000 ミリ秒 (150 秒) かかり、それ自体が遅い UI の原因になります。

この負荷を軽減するために、カスタム クラスを使用して複数のパラメーターをバンドルすることができます。 たとえば、`<TableCell>` コンポーネントで次を受け取るようにします。

```razor
@typeparam TItem

...

@code {
    [Parameter]
    public TItem Data { get; set; }
    
    [Parameter]
    public GridOptions Options { get; set; }
}
```

上の例では、`Data` はすべてのセルで異なりますが、`Options` はすべてのセルで共通です。 もちろん、`<TableCell>` コンポーネントを使用せず、代わりにそのロジックを親コンポーネントにインラインで挿入した方が改善される場合もあります。

#### <a name="ensure-cascading-parameters-are-fixed"></a>カスケード型パラメーターが固定されていることを確認する

`<CascadingValue>` コンポーネントには、`IsFixed` という省略可能なパラメーターがあります。

* `IsFixed` の値が `false` (既定値) の場合、カスケードされた値のすべての受信者は、変更通知を受信するためのサブスクリプションを設定します。 この場合、サブスクリプションの追跡により、各 `[CascadingParameter]` は通常の `[Parameter]` よりも **大幅にコストが高くなります**。
* `IsFixed` の値が `true` (`<CascadingValue Value="@someValue" IsFixed="true">` など) の場合、受信者は初期値を受け取りますが、更新を受信するためのサブスクリプションを設定 "*しません*"。 この場合、各 `[CascadingParameter]` は軽量であり、通常の `[Parameter]` よりも **コストが高くなることはありません**。

したがって、可能な限り、カスケードされた値には `IsFixed="true"` を使用する必要があります。 指定される値が時間の経過と共に変化しない場合は、常にこれを行うことができます。 コンポーネントによって `this` がカスケードされた値として渡される一般的なパターンでは、`IsFixed="true"` を使用する必要があります。

```razor
<CascadingValue Value="this" IsFixed="true">
    <SomeOtherComponents>
</CascadingValue>
```

これにより、カスケードされた値を受け取る他のコンポーネントが多数ある場合に大きな違いが生じます。 詳細については、<xref:blazor/components/cascading-values-and-parameters> を参照してください。

#### <a name="avoid-attribute-splatting-with-captureunmatchedvalues"></a>`CaptureUnmatchedValues` で属性スプラッティングを避ける

コンポーネントでは、<xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> フラグを使用して、"一致しない" パラメーター値を受け取ることができます。

```razor
<div @attributes="OtherAttributes">...</div>

@code {
    [Parameter(CaptureUnmatchedValues = true)]
    public IDictionary<string, object> OtherAttributes { get; set; }
}
```

この方法では、任意の追加属性を要素に渡すことができます。 ただし、レンダラーで次を行う必要があるため、非常にコストがかかります。

* 指定されたすべてのパラメーターを既知のパラメーターのセットと照合して、ディクショナリを構築する。
* 同じ属性の複数のコピーが相互に上書きされているかどうかを追跡する。

頻繁に繰り返されないコンポーネントなど、パフォーマンスが重要でないコンポーネントでは <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> を自由に使用できます。 ただし、大規模なリスト内の項目やグリッド内のセルなど、大規模にレンダリングされるコンポーネントの場合は、属性スプラッティングを避けるようにしてください。

詳細については、<xref:blazor/components/index#attribute-splatting-and-arbitrary-parameters> を参照してください。

#### <a name="implement-setparametersasync-manually"></a>手動で `SetParametersAsync` を実装する

コンポーネントごとのレンダリングにかかるオーバーヘッドの主な側面の 1 つは、入力パラメーター値を `[Parameter]` プロパティに書き込むことです。 レンダラーでは、リフレクションを使用してこれを行う必要があります。 これは多少最適化されていますが、WebAssembly ランタイムに対する JIT サポートの不在により限界があります。

極端なケースでは、リフレクションを使用せずに、独自のパラメーター設定ロジックを手動で実装したい場合があります。 これは、次の場合に該当します。

* 非常に頻繁にレンダリングされるコンポーネントがある (たとえば、UI 内にそのコピーが何百または何千個ある)。
* それが多くのパラメーターを受け取る。
* パラメーターを受け取るオーバーヘッドが UI の応答性に目に見える影響を与えていることがわかる。

このような場合は、コンポーネントの仮想 <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> メソッドをオーバーライドし、独自のコンポーネント固有ロジックを実装できます。 次の例では、すべてのディクショナリ参照を意図的に回避しています。

```razor
@code {
    [Parameter]
    public int MessageId { get; set; }

    [Parameter]
    public string Text { get; set; }

    [Parameter]
    public EventCallback<string> TextChanged { get; set; }

    [Parameter]
    public Theme CurrentTheme { get; set; }

    public override Task SetParametersAsync(ParameterView parameters)
    {
        foreach (var parameter in parameters)
        {
            switch (parameter.Name)
            {
                case nameof(MessageId):
                    MessageId = (int)parameter.Value;
                    break;
                case nameof(Text):
                    Text = (string)parameter.Value;
                    break;
                case nameof(TextChanged):
                    TextChanged = (EventCallback<string>)parameter.Value;
                    break;
                case nameof(CurrentTheme):
                    CurrentTheme = (Theme)parameter.Value;
                    break;
                default:
                    throw new ArgumentException($"Unknown parameter: {parameter.Name}");
            }
        }

        return base.SetParametersAsync(ParameterView.Empty);
    }
}
```

上のコードでは、基底クラス <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> を返すと、パラメーターを再度割り当てずに通常のライフサイクル メソッドが実行されます。

上のコードでわかるように、<xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> のオーバーライドとカスタム ロジックの提供は複雑で手間がかかるため、この方法は一般にお勧めしません。 極端なケースでは、これによってレンダリング パフォーマンスを 20-25% 向上させることができますが、この方法は前述のシナリオでのみ検討してください。

### <a name="dont-trigger-events-too-rapidly"></a>イベントをすぐにトリガーしない

一部のブラウザー イベントは非常に頻繁に発生します。たとえば `onmousemove` や `onscroll` などは、1 秒間に数十回または数百回発生する可能性があります。 ほとんどの場合、これほど頻繁に UI 更新を実行する必要はありません。 これを行おうとすると、UI の応答性が損なわれたり、過剰な CPU 時間が消費されたりする可能性があります。

ネイティブの `@onmousemove` や `@onscroll` イベントを使用するのではなく、JS 相互運用を使用して、発生する頻度の低いコールバックを登録することをお勧めします。 たとえば、次のコンポーネント (`MyComponent.razor`) にはマウスの位置が表示されますが、500 ミリ秒ごとに最大で 1 回しか更新されません。

```razor
@inject IJSRuntime JS
@implements IDisposable

<h1>@message</h1>

<div @ref="myMouseMoveElement" style="border:1px dashed red;height:200px;">
    Move mouse here
</div>

@code {
    ElementReference myMouseMoveElement;
    DotNetObjectReference<MyComponent> selfReference;
    private string message = "Move the mouse in the box";

    [JSInvokable]
    public void HandleMouseMove(int x, int y)
    {
        message = $"Mouse move at {x}, {y}";
        StateHasChanged();
    }

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            selfReference = DotNetObjectReference.Create(this);
            var minInterval = 500; // Only notify every 500 ms
            await JS.InvokeVoidAsync("onThrottledMouseMove", 
                myMouseMoveElement, selfReference, minInterval);
        }
    }

    public void Dispose() => selfReference?.Dispose();
}
```

対応する JavaScript コード (`index.html` ページに配置したり、ES6 モジュールとして読み込んだりできます) によって、実際の DOM イベント リスナーが登録されます。 この例の場合、イベント リスナーでは [Lodash の `throttle` 関数](https://lodash.com/docs/4.17.15#throttle)を使用して、呼び出し率を制限しています。

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.20/lodash.min.js"></script>
<script>
  function onThrottledMouseMove(elem, component, interval) {
    elem.addEventListener('mousemove', _.throttle(e => {
      component.invokeMethodAsync('HandleMouseMove', e.offsetX, e.offsetY);
    }, interval));
  }
</script>
```

この手法は Blazor Server ではさらに重要になる可能性があります。各イベントの呼び出しに、ネットワーク経由でのメッセージの配信が伴うためです。 これが Blazor WebAssembly において価値を持つのは、レンダリング作業の量を大幅に削減できるためです。

## <a name="optimize-javascript-interop-speed"></a>JavaScript 相互運用の速度を最適化する

.NET と JavaScript の間の呼び出しには、次の理由から追加のオーバーヘッドが伴います。

* 既定では、呼び出しは非同期です。
* 既定では、パラメーターと戻り値は JSON でシリアル化されます。 これは .NET と JavaScript の型の間にわかりやすい変換メカニズムを提供するためです。

さらに Blazor Server では、これらの呼び出しはネットワーク経由で渡されます。

### <a name="avoid-excessively-fine-grained-calls"></a>過度に細かい呼び出しを避ける

各呼び出しにはある程度のオーバーヘッドが伴うため、呼び出しの回数を減らすことが有益な場合があります。 ブラウザーの `localStorage` ストアに項目のコレクションを格納する次のコードについて考えてみます。

```csharp
private async Task StoreAllInLocalStorage(IEnumerable<TodoItem> items)
{
    foreach (var item in items)
    {
        await JS.InvokeVoidAsync("localStorage.setItem", item.Id, 
            JsonSerializer.Serialize(item));
    }
}
```

上の例では、各項目に対して個別の JS 相互運用呼び出しを行います。 代わりに、次の方法では JS 相互運用を 1 回の呼び出しに減らしています。

```csharp
private async Task StoreAllInLocalStorage(IEnumerable<TodoItem> items)
{
    await JS.InvokeVoidAsync("storeAllInLocalStorage", items);
}
```

対応する JavaScript 関数は次のように定義されます。

```javascript
function storeAllInLocalStorage(items) {
  items.forEach(item => {
    localStorage.setItem(item.id, JSON.stringify(item));
  });
}
```

Blazor WebAssembly の場合、これは通常、多数の JS 相互運用呼び出しを行う場合にのみ重要になります。

### <a name="consider-making-synchronous-calls"></a>同期呼び出しを行うことを検討する

JavaScript 相互運用呼び出しは、呼び出されているコードが同期であるか非同期であるかに関係なく、既定で非同期です。 これは、コンポーネントが Blazor WebAssembly と Blazor Server の両方と互換性を持つようにするためです。 Blazor Server では、すべての JavaScript 相互運用呼び出しはネットワーク接続を介して送信されるため、非同期である必要があります。

アプリが Blazor WebAssembly でのみ実行されることが確実にわかっている場合は、同期 JavaScript 相互運用呼び出しの実行を選択できます。 これにより、非同期呼び出しを行う場合よりもオーバーヘッドがわずかに減少し、レンダリング サイクルが少なくなる可能性があります。結果を待機する間の中間状態が存在しないためです。

.NET から JavaScript への同期呼び出しを行うには、<xref:Microsoft.JSInterop.IJSRuntime> を <xref:Microsoft.JSInterop.IJSInProcessRuntime> にキャストします。

```razor
@inject IJSRuntime JS

...

@code {
    protected override void HandleSomeEvent()
    {
        var jsInProcess = (IJSInProcessRuntime)JS;
        var value = jsInProcess.Invoke<string>("javascriptFunctionIdentifier");
    }
}
```

::: moniker range=">= aspnetcore-5.0"

`IJSObjectReference` を使用する場合は、`IJSInProcessObjectReference` にキャストすることによって同期呼び出しを行うことができます。

::: moniker-end

JavaScript から .NET への同期呼び出しを行うには、`DotNet.invokeMethodAsync` ではなく `DotNet.invokeMethod` を使用します。

同期呼び出しは、次の場合に機能します。

* アプリが Blazor Server ではなく Blazor WebAssembly で実行されている。
* 呼び出された関数から同期的に値が返される (`async` メソッドではなく、.NET <xref:System.Threading.Tasks.Task> や JavaScript `Promise` が返されない)。

詳細については、<xref:blazor/call-javascript-from-dotnet> を参照してください。

::: moniker range=">= aspnetcore-5.0"
 
### <a name="consider-making-unmarshalled-calls"></a>マーシャリング解除された呼び出しを行うことを検討する

Blazor WebAssembly で実行する場合、.NET から JavaScript へのマーシャリング解除された呼び出しを行うことができます。 これらは、引数または戻り値の JSON シリアル化を実行しない同期呼び出しです。 .NET と JavaScript の表現の間のメモリ管理と翻訳のすべての側面は、開発者に任されます。

> [!WARNING]
> `IJSUnmarshalledRuntime` を使用すると JS 相互運用アプローチのオーバーヘッドが最小限になりますが、これらの API とのやり取りに必要な JavaScript API は現在ドキュメントに記載されていません。また、今後のリリースでの破壊的変更の対象となることがあります。

```javascript
function jsInteropCall() {
    return BINDING.js_to_mono_obj("Hello world");
}
```

```razor
@inject IJSRuntime JS

@code {
    protected override void OnInitialized()
    {
        var unmarshalledJs = (IJSUnmarshalledRuntime)JS;
        var value = unmarshalledJs.InvokeUnmarshalled<string>("jsInteropCall");
    }
}
```

::: moniker-end

## <a name="minimize-app-download-size"></a>アプリのダウンロード サイズを最小限にする

::: moniker range=">= aspnetcore-5.0"

### <a name="intermediate-language-il-trimming"></a>中間言語 (IL) のトリミング

使用されていないアセンブリを Blazor WebAssembly アプリからトリミングすると、アプリのバイナリで使用されていないコードを削除して、アプリのサイズを縮小することができます。 詳細については、「<xref:blazor/host-and-deploy/configure-trimmer>」を参照してください。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

### <a name="intermediate-language-il-linking"></a>中間言語 (IL) のリンク

[Blazor WebAssembly アプリをリンク](xref:blazor/host-and-deploy/configure-linker)すると、アプリのバイナリで使用されていないコードをトリミングすることで、アプリのサイズが縮小されます。 既定では、中間言語 (IL) リンカーは、`Release` 構成でビルドする場合にのみ有効になります。 これを活用するには、[-c|--configuration](/dotnet/core/tools/dotnet-publish#options) オプションを `Release` に設定した状態で [`dotnet publish`](/dotnet/core/tools/dotnet-publish) コマンドを使用して、展開用にアプリを発行します。

```dotnetcli
dotnet publish -c Release
```

::: moniker-end

### <a name="use-systemtextjson"></a>System.Text.Json を使用する

Blazor の JS 相互運用の実装は <xref:System.Text.Json> に依存します。これは、メモリ割り当てが少ない高パフォーマンスの JSON シリアル化ライブラリです。 <xref:System.Text.Json> を使用しても、1 つ以上の代替 JSON ライブラリを追加したときにアプリのペイロードのサイズは増加しません。

移行のガイダンスについては、「[`Newtonsoft.Json` から `System.Text.Json` に移行する方法](/dotnet/standard/serialization/system-text-json-migrate-from-newtonsoft-how-to)」を参照してください。

### <a name="lazy-load-assemblies"></a>アセンブリの遅延読み込み

アセンブリがルートによって要求されたときに、実行時にアセンブリを読み込みます。 詳細については、「<xref:blazor/webassembly-lazy-load-assemblies>」を参照してください。

### <a name="compression"></a>[圧縮]

Blazor WebAssembly アプリが公開されると、公開中に出力が静的に圧縮されてアプリのサイズが縮小され、実行時の圧縮に必要なオーバーヘッドがなくなります。 Blazor は、コンテンツ ネゴシエーションの実行および静的に圧縮されたファイルの提供に関して、サーバーに依存します。

アプリが展開されたら、アプリが圧縮ファイルを提供していることを確認します。 ブラウザーの開発者ツールの [ネットワーク] タブを調べ、ファイルが `Content-Encoding: br` または `Content-Encoding: gz` で提供されていることを確認します。 ホストが圧縮ファイルを提供していない場合は、「<xref:blazor/host-and-deploy/webassembly#compression>」に記載されている手順に従ってください。

### <a name="disable-unused-features"></a>未使用の機能を無効にする

Blazor WebAssembly のランタイムには、次の .NET 機能が含まれています。これらの機能はアプリで必要がない場合、ペイロード サイズを小さくするために無効にすることができます。

* データ ファイルは、タイムゾーン情報を正しく保つために含まれています。 アプリでこの機能を必要としない場合は、アプリのプロジェクト ファイル内の `BlazorEnableTimeZoneSupport` MSBuild プロパティを `false` に設定して無効にすることを検討してください。

  ```xml
  <PropertyGroup>
    <BlazorEnableTimeZoneSupport>false</BlazorEnableTimeZoneSupport>
  </PropertyGroup>
  ```

::: moniker range=">= aspnetcore-5.0"

* 既定では、Blazor WebAssembly には、ユーザーのカルチャで日付や通貨などの値を表示するために必要なグローバリゼーション リソースがあります。 アプリがローカライズを必要としない場合は、`en-US` カルチャに基づく[インバリアント カルチャをサポートするようにアプリを構成する](xref:blazor/globalization-localization)ことができます。

  ```xml
  <PropertyGroup>
    <InvariantGlobalization>true</InvariantGlobalization>
  </PropertyGroup>
  ```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* <xref:System.StringComparison.InvariantCultureIgnoreCase?displayProperty=nameWithType> などの API を正常に動作させるために、照合順序情報が含まれています。 アプリで照合順序情報を必要としないことが確実な場合は、アプリのプロジェクト ファイル内の `BlazorWebAssemblyPreserveCollationData` MSBuild プロパティを `false` に設定して無効にすることを検討してください。

  ```xml
  <PropertyGroup>
    <BlazorWebAssemblyPreserveCollationData>false</BlazorWebAssemblyPreserveCollationData>
  </PropertyGroup>
  ```

::: moniker-end
