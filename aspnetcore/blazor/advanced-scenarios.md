---
title: ASP.NET Core Blazor の高度なシナリオ
author: guardrex
description: Blazor の高度なシナリオについて説明します。これには、手動の RenderTreeBuilder ロジックをアプリに組み込む方法などが含まれます。
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
uid: blazor/advanced-scenarios
ms.openlocfilehash: bbefdf7272cc09d09baa2e5f2a42d04f91eca1bd
ms.sourcegitcommit: 1f35de0ca9ba13ea63186c4dc387db4fb8e541e0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2021
ms.locfileid: "104711205"
---
# <a name="aspnet-core-blazor-advanced-scenarios"></a>ASP.NET Core Blazor の高度なシナリオ

## <a name="manual-rendertreebuilder-logic"></a>手動の RenderTreeBuilder ロジック

<xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> には、コンポーネントと要素を操作するためのメソッドが用意されています。これには、C# コードでコンポーネントを手動で作成することも含まれます。

> [!WARNING]
> コンポーネントの作成に <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> を使用することは、"*高度なシナリオ*" です。 形式が正しくないコンポーネント (閉じられていないマークアップ タグなど) により、定義されていない動作が発生する可能性があります。 未定義の動作には、コンテンツのレンダリングの中断、アプリの機能の損失、**_セキュリティ侵害_** などが含まれます。

次の `PetDetails` コンポーネントを考えてみましょう。これは手動で別のコンポーネントにレンダリングすることができます。

`Shared/PetDetails.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/advanced-scenarios/PetDetails.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/advanced-scenarios/PetDetails.razor)]

::: moniker-end

次の `BuiltContent` コンポーネントでは、`CreateComponent` メソッド内のループによって、3 つの `PetDetails` コンポーネントが生成されます。

シーケンス番号のある <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> メソッドでは、シーケンス番号はソース コードの行番号です。 Blazor の差分アルゴリズムは、個別の呼び出しではなく、個別のコード行に対応するシーケンス番号に依存しています。 <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> メソッドを使用してコンポーネントを作成する場合は、シーケンス番号の引数をハードコードします。 **計算またはカウンターを使用してシーケンス番号を生成すると、パフォーマンスが低下する可能性があります。** 詳細については、「[シーケンス番号は実行順序ではなくコード行番号に関係する](#sequence-numbers-relate-to-code-line-numbers-and-not-execution-order)」セクションを参照してください。

`Pages/BuiltContent.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/advanced-scenarios/BuiltContent.razor?highlight=6,16-24,28)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/advanced-scenarios/BuiltContent.razor?highlight=6,16-24,28)]

::: moniker-end

> [!WARNING]
> <xref:Microsoft.AspNetCore.Components.RenderTree> の型により、レンダリング操作の "*結果*" の処理が許可されます。 これらは、Blazor フレームワーク実装の内部的な詳細です。 これらの型は "*不安定*" と考えるべきで、今後のリリースで変更される可能性があります。

### <a name="sequence-numbers-relate-to-code-line-numbers-and-not-execution-order"></a>シーケンス番号は実行順序ではなくコード行番号に関係する

Razor コンポーネント ファイル (`.razor`) は常にコンパイルされます。 コンパイル済みコードを実行する方が、コードを解釈するよりも潜在的な利点があります。コンパイル済みコードを生成するコンパイル ステップを使用して、実行時のアプリのパフォーマンスを向上させる情報を挿入できるからです。

これらの機能強化の主な例として、"*シーケンス番号*" があります。 シーケンス番号は、出力がコードの個別の順序付けられたどの行からのものかをランタイムに示します。 ランタイムでは、この情報を使用して、効率的なツリーの差分を線形時間で生成します。これは、一般的なツリーの差分アルゴリズムで通常にできるよりもかなり高速です。

次の Razor コンポーネント ファイル (`.razor`) について考えてみましょう。

```razor
@if (someFlag)
{
    <text>First</text>
}

Second
```

上記の Razor マークアップとテキスト コンテンツは、次のような C# コードにコンパイルされます。

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

ランタイムで差分を実行すると、シーケンス `0` の項目が削除されたことが認識されるため、1 つのステップで次のような単純な "*編集スクリプト*" が生成されます。

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

今度は、差分アルゴリズムによって、"*2 つ*" の変更が発生したことが認識されます。 アルゴリズムによって次の編集スクリプトが生成されます。

* 最初のテキスト ノードの値を `Second` に変更します。
* 2 番目のテキスト ノードを削除します。

シーケンス番号を生成すると、`if/else` 分岐とループが元のコードのどこにあったかに関する有用な情報がすべて失われます。 これにより、差分が以前の **2 倍の長さ** なります。

これは簡単な例です。 複雑で深く入れ子になった構造体で、特にループがある、より現実的なケースでは、通常、パフォーマンス コストが高くなります。 どのループ ブロックまたは分岐が挿入または削除されたかを即座に特定する代わりに、差分アルゴリズムではレンダリング ツリー内を深く再帰処理する必要があります。 これにより、古い構造と新しい構造体が相互にどのように関連しているかについて差分アルゴリズムが誤って通知されるため、通常、より長い編集スクリプトを作成することになります。

### <a name="guidance-and-conclusions"></a>ガイダンスと結論

* シーケンス番号が動的に生成される場合、アプリのパフォーマンスが低下します。
* コンパイル時にキャプチャされない限り、必要な情報が存在しないため、実行時にフレームワークで独自のシーケンス番号を自動的に作成することはできません。
* 手動で実装された <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> ロジックの長いブロックは記述しないでください。 `.razor` ファイルを優先し、コンパイラがシーケンス番号を処理できるようにします。 手動の <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> ロジックを回避できない場合は、長いブロックのコードを <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.OpenRegion%2A>/<xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.CloseRegion%2A> 呼び出しでラップされたより小さな部分に分割します。 各リージョンには独自のシーケンス番号の個別のスペースがあるため、各リージョン内でゼロ (または他の任意の数) から再開できます。
* シーケンス番号がハードコードされている場合、差分アルゴリズムでは、シーケンス番号の値が増えることだけが要求されます。 初期値とギャップは関係ありません。 合理的な選択肢の 1 つは、コード行番号をシーケンス番号として使用するか、ゼロから開始し、1 つずつまたは 100 ずつ (または任意の間隔で) 増やすことです。
* Blazor ではシーケンス番号が使用されていますが、他のツリー差分 UI フレームワークでは使用されていません。 シーケンス番号を使用すると、差分がはるかに高速になります。また、Blazor には、`.razor` ファイルを作成する開発者に対して、シーケンス番号を自動的に処理するコンパイル ステップの利点があります。
