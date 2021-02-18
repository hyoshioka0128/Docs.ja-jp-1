---
title: ASP.NET Core Blazor の高度なシナリオ
author: guardrex
description: Blazor の高度なシナリオについて説明します。これには、手動の RenderTreeBuilder ロジックをアプリに組み込む方法などが含まれます。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/18/2020
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
uid: blazor/advanced-scenarios
ms.openlocfilehash: ba2bf91f3318225383ec9d164c34be9124aa311b
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280853"
---
# <a name="aspnet-core-blazor-advanced-scenarios"></a>ASP.NET Core Blazor の高度なシナリオ

## <a name="blazor-server-circuit-handler"></a>Blazor Server 回線ハンドラー

Blazor Server を使用すると、コードで "*回線ハンドラー*" を定義できます。これにより、ユーザーの回線の状態の変更時にコードを実行できます。 回線ハンドラーは、`CircuitHandler` から派生させ、そのクラスをアプリのサービス コンテナーに登録することで実装します。 次の回線ハンドラーの例では、開いている SignalR 接続を追跡します。

```csharp
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.Server.Circuits;

public class TrackingCircuitHandler : CircuitHandler
{
    private HashSet<Circuit> circuits = new HashSet<Circuit>();

    public override Task OnConnectionUpAsync(Circuit circuit, 
        CancellationToken cancellationToken)
    {
        circuits.Add(circuit);

        return Task.CompletedTask;
    }

    public override Task OnConnectionDownAsync(Circuit circuit, 
        CancellationToken cancellationToken)
    {
        circuits.Remove(circuit);

        return Task.CompletedTask;
    }

    public int ConnectedCircuits => circuits.Count;
}
```

回線ハンドラーは DI を使用して登録されます。 スコープを持つインスタンスは、回線のインスタンスごとに作成されます。 前の例の `TrackingCircuitHandler` を使用すると、すべての回線の状態を追跡する必要があるため、シングルトン サービスが作成されます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...
    services.AddSingleton<CircuitHandler, TrackingCircuitHandler>();
}
```

カスタム回線ハンドラーのメソッドでハンドルされない例外がスローされる場合は、その例外は Blazor Server 回線にとって致命的です。 ハンドラーのコードまたはメソッドで例外が許容されるようにするには、エラー処理とログを含む 1 つ以上の [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) ステートメントでコードをラップします。

ユーザーが切断し、フレームワークで回線の状態がクリーンアップされていることが原因で回線が終了すると、フレームワークによって回線の DI スコープが破棄されます。 スコープが破棄されると、<xref:System.IDisposable?displayProperty=fullName> を実装するサーキットスコープの DI サービスはすべて破棄されます。 破棄中にいずれかの DI サービスでハンドルされない例外がスローされると、フレームワークによって例外がログに記録されます。

## <a name="manual-rendertreebuilder-logic"></a>手動の RenderTreeBuilder ロジック

<xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> には、コンポーネントと要素を操作するためのメソッドが用意されています。これには、C# コードでコンポーネントを手動で作成することも含まれます。

> [!NOTE]
> コンポーネントの作成に <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> を使用することは、高度なシナリオです。 形式が正しくないコンポーネント (閉じられていないマークアップ タグなど) により、定義されていない動作が発生する可能性があります。

次の `PetDetails` コンポーネントを考えてみましょう。これは手動で別のコンポーネントに組み込むことができます。

```razor
<h2>Pet Details Component</h2>

<p>@PetDetailsQuote</p>

@code
{
    [Parameter]
    public string PetDetailsQuote { get; set; }
}
```

次の例では、`CreateComponent` メソッド内のループによって、3 つの `PetDetails` コンポーネントが生成されます。 シーケンス番号のある <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> メソッドでは、シーケンス番号はソース コードの行番号です。 Blazor の差分アルゴリズムは、個別の呼び出しではなく、個別のコード行に対応するシーケンス番号に依存しています。 <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> メソッドを使用してコンポーネントを作成する場合は、シーケンス番号の引数をハードコードします。 **計算またはカウンターを使用してシーケンス番号を生成すると、パフォーマンスが低下する可能性があります。** 詳細については、「[シーケンス番号は実行順序ではなくコード行番号に関係する](#sequence-numbers-relate-to-code-line-numbers-and-not-execution-order)」セクションを参照してください。

`BuiltContent` コンポーネント:

```razor
@page "/BuiltContent"

<h1>Build a component</h1>

@CustomRender

<button type="button" @onclick="RenderComponent">
    Create three Pet Details components
</button>

@code {
    private RenderFragment CustomRender { get; set; }
    
    private RenderFragment CreateComponent() => builder =>
    {
        for (var i = 0; i < 3; i++) 
        {
            builder.OpenComponent(0, typeof(PetDetails));
            builder.AddAttribute(1, "PetDetailsQuote", "Someone's best friend!");
            builder.CloseComponent();
        }
    };    
    
    private void RenderComponent()
    {
        CustomRender = CreateComponent();
    }
}
```

> [!WARNING]
> <xref:Microsoft.AspNetCore.Components.RenderTree> の型により、レンダリング操作の "*結果*" の処理が許可されます。 これらは、Blazor フレームワーク実装の内部的な詳細です。 これらの型は "*不安定*" と考えるべきで、今後のリリースで変更される可能性があります。

### <a name="sequence-numbers-relate-to-code-line-numbers-and-not-execution-order"></a>シーケンス番号は実行順序ではなくコード行番号に関係する

Razor コンポーネント ファイル (`.razor`) は常にコンパイルされます。 コンパイル ステップは、実行時にアプリのパフォーマンスを向上させる情報を挿入するために使用できるため、コンパイルの方がコードを解釈するよりも潜在的な利点があります。

これらの機能強化の主な例として、"*シーケンス番号*" があります。 シーケンス番号は、出力がコードの個別の順序付けられたどの行からのものかをランタイムに示します。 ランタイムでは、この情報を使用して、効率的なツリーの差分を線形時間で生成します。これは、一般的なツリーの差分アルゴリズムで通常にできるよりもかなり高速です。

次の Razor コンポーネント (`.razor`) ファイルについて考えてみましょう。

```razor
@if (someFlag)
{
    <text>First</text>
}

Second
```

上記のコードは、次のようにコンパイルされます。

```csharp
if (someFlag)
{
    builder.AddContent(0, "First");
}

builder.AddContent(1, "Second");
```

コードを初めて実行するときに、`someFlag` が `true` の場合、ビルダーは以下を受け取ります。

| シーケンス | 種類      | データ   |
| :------: | --------- | :----: |
| 0        | テキスト ノード | First  |
| 1        | テキスト ノード | Second |

`someFlag` が `false` になり、マークアップが再びレンダリングされるとします。 今度は、ビルダーは以下を受け取ります。

| シーケンス | 種類       | データ   |
| :------: | ---------- | :----: |
| 1        | テキスト ノード  | Second |

ランタイムで差分を実行すると、シーケンス `0` の項目が削除されたことが認識されるため、次のような単純な "*編集スクリプト*" が生成されます。

* Remove the first text node. (最初のテキスト ノードを削除します。)

### <a name="the-problem-with-generating-sequence-numbers-programmatically"></a>プログラムによってシーケンス番号を生成する場合の問題

代わりに、次のレンダリング ツリー ビルダー ロジックを記述したとします。

```csharp
var seq = 0;

if (someFlag)
{
    builder.AddContent(seq++, "First");
}

builder.AddContent(seq++, "Second");
```

最初の出力は次のようになります。

| シーケンス | 種類      | データ   |
| :------: | --------- | :----: |
| 0        | テキスト ノード | First  |
| 1        | テキスト ノード | Second |

この結果は前のケースと同じであるため、否定的な問題は存在しません。 `someFlag` は 2 番目のレンダリングでは `false` で、出力は次のようになります。

| シーケンス | 種類      | データ   |
| :------: | --------- | ------ |
| 0        | テキスト ノード | Second |

今度は、差分アルゴリズムによって、"*2 つ*" の変更が発生したことが認識され、アルゴリズムによって次の編集スクリプトが生成されます。

* 最初のテキスト ノードの値を `Second` に変更します。
* 2 番目のテキスト ノードを削除します。

シーケンス番号を生成すると、`if/else` 分岐とループが元のコードのどこにあったかに関する有用な情報がすべて失われます。 これにより、差分が以前の **2 倍の長さ** なります。

これは簡単な例です。 複雑で深く入れ子になった構造体で、特にループがある、より現実的なケースでは、通常、パフォーマンス コストが高くなります。 どのループ ブロックまたは分岐が挿入または削除されたかを即座に特定する代わりに、差分アルゴリズムではレンダリング ツリー内を深く再帰処理する必要があります。 これにより、古い構造と新しい構造体が相互にどのように関連しているかについて差分アルゴリズムが誤って通知されるため、通常、より長い編集スクリプトを作成することが必要になります。

### <a name="guidance-and-conclusions"></a>ガイダンスと結論

* シーケンス番号が動的に生成される場合、アプリのパフォーマンスが低下します。
* コンパイル時にキャプチャされない限り、必要な情報が存在しないため、実行時にフレームワークで独自のシーケンス番号を自動的に作成することはできません。
* 手動で実装された <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> ロジックの長いブロックは記述しないでください。 `.razor` ファイルを優先し、コンパイラがシーケンス番号を処理できるようにします。 手動の <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> ロジックを回避できない場合は、長いブロックのコードを <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.OpenRegion%2A>/<xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.CloseRegion%2A> 呼び出しでラップされたより小さな部分に分割します。 各リージョンには独自のシーケンス番号の個別のスペースがあるため、各リージョン内でゼロ (または他の任意の数) から再開できます。
* シーケンス番号がハードコードされている場合、差分アルゴリズムでは、シーケンス番号の値が増えることだけが要求されます。 初期値とギャップは関係ありません。 合理的な選択肢の 1 つは、コード行番号をシーケンス番号として使用するか、ゼロから開始し、1 つずつまたは 100 ずつ (または任意の間隔で) 増やすことです。 
* Blazor ではシーケンス番号が使用されていますが、他のツリー差分 UI フレームワークでは使用されていません。 シーケンス番号を使用すると、差分がはるかに高速になります。また、Blazor には、`.razor` ファイルを作成する開発者に対して、シーケンス番号を自動的に処理するコンパイル ステップの利点があります。
