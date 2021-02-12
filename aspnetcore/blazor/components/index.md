---
title: ASP.NET Core Razor コンポーネントの作成と使用
author: guardrex
description: データへのバインド、イベントの処理、コンポーネント ライフ サイクルの管理の方法など、Razor コンポーネントを作成および使用する方法について学習します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
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
uid: blazor/components/index
ms.openlocfilehash: 111512916cb7f0a4fc1f17648e2f9c69e366dff3
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/10/2021
ms.locfileid: "100107052"
---
# <a name="create-and-use-aspnet-core-razor-components"></a>ASP.NET Core Razor コンポーネントの作成と使用

作成者: [Luke Latham](https://github.com/guardrex)、[Daniel Roth](https://github.com/danroth27)、[Scott Addie](https://github.com/scottaddie)、[Tobias Bartsch](https://www.aveo-solutions.com/)

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

Blazor アプリは *コンポーネント* を使用してビルドします。 コンポーネントは、ページ、ダイアログ、フォームなどのユーザー インターフェイス (UI) の自己完結型のチャンクです。 コンポーネントには、データの挿入や UI イベントへの応答に必要な HTML マークアップと、処理ロジックが含まれます。 コンポーネントは、柔軟性があり、軽量です。 それらを入れ子にしたり、再利用したり、プロジェクト間で共有したりできます。

## <a name="component-classes"></a>コンポーネント クラス

コンポーネントは、C# と HTML マークアップの組み合わせを使用して、[Razor](xref:mvc/views/razor) コンポーネント ファイル (`.razor`) で実装します。 Blazor のコンポーネントは、正式には *Razor コンポーネント* と呼ばれます。

### <a name="razor-syntax"></a>Razor の構文

Blazor アプリの Razor コンポーネントでは、Razor 構文が多用されます。 Razor マークアップ言語に慣れていない場合は、続行する前に、「[ASP.NET Core の Razor 構文のリファレンス](xref:mvc/views/razor)」を読むことをお勧めします。

Razor 構文でコンテンツにアクセスする場合は、次のセクションに特にご注意ください。

* [ディレクティブ](xref:mvc/views/razor#directives): 通常はコンポーネント マークアップの解析方法や機能を変更する、`@` プレフィックス付きの予約キーワード。
* [ディレクティブ属性](xref:mvc/views/razor#directive-attributes): 通常はコンポーネント要素の解析方法や機能を変更する、`@` プレフィックス付きの予約キーワード。

### <a name="names"></a>名前

コンポーネントの名前は、大文字で始める必要があります。 たとえば、`MyCoolComponent.razor` は有効で、`myCoolComponent.razor` は無効です。

### <a name="routing"></a>ルーティング

Blazor でのルーティングは、アプリ内のアクセス可能な各コンポーネントへのルート テンプレートを提供することで実現します。 [`@page`][9] ディレクティブを含む Razor ファイルがコンパイルされると、生成されたクラスに、ルート テンプレートを指定する <xref:Microsoft.AspNetCore.Mvc.RouteAttribute> が指定されます。 実行時に、ルーターによって <xref:Microsoft.AspNetCore.Mvc.RouteAttribute> を持つコンポーネント クラスが検索され、要求された URL に一致するルート テンプレートを使用するコンポーネントがレンダリングされます。 詳細については、「<xref:blazor/fundamentals/routing>」を参照してください。

```razor
@page "/ParentComponent"

...
```

### <a name="markup"></a>マークアップ

コンポーネントの UI は、HTML を使用して定義します。 動的なレンダリング ロジック (たとえばループ、条件、式) が、 *Razor* と呼ばれる埋め込みの C# 構文を使って追加されています。 アプリがコンパイルされると、HTML マークアップと C# のレンダリング ロジックはコンポーネント クラスに変換されます。 生成されたクラスの名前は、ファイルの名前と一致します。

コンポーネント クラスのメンバーは、[`@code`][1] ブロック内で定義されています。 [`@code`][1] ブロックには、イベント処理のメソッド、またはその他のコンポーネント ロジックを定義するためのメソッドによって、コンポーネントの状態 (プロパティ、フィールド) を指定します。 複数の [`@code`][1] ブロックが許容されます。

コンポーネント メンバーは、`@` で始まる C# 式を使用して、コンポーネントのレンダリング ロジックの一部として使用できます。 たとえば、フィールド名の前に `@` を付けることによって、C# フィールドがレンダリングされます。 次の例では、以下のように評価され、レンダリングされます。

* `headingFontStyle` が `font-style` の CSS プロパティ値に。
* `headingText` が `<h1>` 要素のコンテンツに。

```razor
<h1 style="font-style:@headingFontStyle">@headingText</h1>

@code {
    private string headingFontStyle = "italic";
    private string headingText = "Put on your new Blazor!";
}
```

コンポーネントが最初にレンダリングされた後に、コンポーネントがイベントに応答して、レンダリング ツリーを再生成します。 Blazor によって新旧のレンダリング ツリーが比較され、ブラウザーのドキュメント オブジェクト モデル (DOM) に変更が適用されます。 詳細については、「<xref:blazor/components/rendering>」をご覧ください。

コンポーネントは通常の C# クラスであり、プロジェクト内の任意の場所に配置できます。 Web ページを生成するコンポーネントは、通常、`Pages` フォルダーに存在します。 ページ以外のコンポーネントは、多くの場合、`Shared` フォルダー、またはプロジェクトに追加されたカスタム フォルダーに配置されます。

### <a name="namespaces"></a>名前空間

一般に、コンポーネントの名前空間は、アプリのルート名前空間と、アプリ内のコンポーネントの場所 (フォルダー) から派生します。 アプリのルート名前空間が `BlazorSample` で、`Counter` コンポーネントが `Pages` フォルダーに存在する場合:

* `Counter` コンポーネントの名前空間は `BlazorSample.Pages` になります。
* コンポーネントの完全修飾型名は `BlazorSample.Pages.Counter` になります。

コンポーネントを保持するカスタム フォルダーの場合は、[`@using`][2] ディレクティブを親コンポーネントまたはアプリの `_Imports.razor` ファイルに追加します。 次の例では、`Components` フォルダー内のコンポーネントを使用できるようにします。

```razor
@using BlazorSample.Components
```

コンポーネントは、完全修飾名を使用して参照することもできます。この場合、[`@using`][2] ディレクティブは必要ありません。

```razor
<BlazorSample.Components.MyComponent />
```

Razor で作成されるコンポーネントの名前空間は、次に基づきます (優先順)。

* Razor ファイル (`.razor`) マークアップ内の [`@namespace`][8] の指定 (`@namespace BlazorSample.MyNamespace`)。
* プロジェクト ファイル内のプロジェクトの `RootNamespace` (`<RootNamespace>BlazorSample</RootNamespace>`)。
* プロジェクト ファイルのファイル名 (`.csproj`) から取得されたプロジェクト名、およびプロジェクト ルートからコンポーネントへのパス。 たとえば、フレームワークでは `{PROJECT ROOT}/Pages/Index.razor` (`BlazorSample.csproj`) が名前空間 `BlazorSample.Pages` に解決されます。 コンポーネントは C# の名前のバインド規則に従います。 この例の `Index` コンポーネントの場合、スコープ内のコンポーネントは、次のすべてのコンポーネントです。
  * 同じフォルダー (`Pages`) に含まれるもの。
  * 別の名前空間を明示的に指定しない、プロジェクトのルート内のコンポーネント。

> [!NOTE]
> `global::` 修飾はサポートされていません。
>
> 別名が付けられた [`using`](/dotnet/csharp/language-reference/keywords/using-statement) ステートメント (`@using Foo = Bar` など) によるコンポーネントのインポートはサポートされていません。
>
> 部分修飾名はサポートされていません。 たとえば、`<Shared.NavMenu></Shared.NavMenu>` による `@using BlazorSample` の追加と `NavMenu` コンポーネント (`NavMenu.razor`) の参照はサポートされていません。

### <a name="partial-class-support"></a>部分クラスのサポート

Razor コンポーネントは、部分クラスとして生成されます。 Razor コンポーネントは、次のいずれかの方法を使用して作成します。

* C# コードは、1 つのファイルに HTML マークアップと Razor コードを含む [`@code`][1] ブロックで定義します。 Blazor テンプレートでは、この方法を使用して Razor コンポーネントを定義します。
* C# コードは、部分クラスとして定義されている分離コード ファイルに配置されます。

次の例は、Blazor テンプレートから生成されたアプリ内の [`@code`][1] ブロックを含む既定の `Counter` コンポーネントを示しています。 HTML マークアップ、Razor コード、C# コードは、同じファイル内にあります。

`Pages/Counter.razor`:

```razor
@page "/counter"

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;

    void IncrementCount()
    {
        currentCount++;
    }
}
```

`Counter` コンポーネントは、部分クラスを含む分離コード ファイルを使用して作成することもできます。

`Pages/Counter.razor`:

```razor
@page "/counter"

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>
```

`Counter.razor.cs`:

```csharp
namespace BlazorSample.Pages
{
    public partial class Counter
    {
        private int currentCount = 0;

        void IncrementCount()
        {
            currentCount++;
        }
    }
}
```

必要に応じて、部分クラスファイルに必要な名前空間を追加します。 Razor コンポーネントで使用される一般的な名前空間には次のものが含まれます。

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.Authorization;
using Microsoft.AspNetCore.Components.Forms;
using Microsoft.AspNetCore.Components.Routing;
using Microsoft.AspNetCore.Components.Web;
```

> [!IMPORTANT]
> `_Imports.razor` ファイルの [`@using`][2] ディレクティブは、C# ファイル (`.cs`) ではなく Razor ファイル (`.razor`) にのみ適用されます。

### <a name="specify-a-base-class"></a>基本クラスの指定

[`@inherits`][6] ディレクティブを使用して、コンポーネントの基本クラスを指定できます。 次の例は、コンポーネントが基本クラス `BlazorRocksBase` を継承して、コンポーネントのプロパティとメソッドを提供する方法を示しています。 基本クラスは <xref:Microsoft.AspNetCore.Components.ComponentBase> から派生する必要があります。

`Pages/BlazorRocks.razor`:

```razor
@page "/BlazorRocks"
@inherits BlazorRocksBase

<h1>@BlazorRocksText</h1>
```

`BlazorRocksBase.cs`:

```csharp
using Microsoft.AspNetCore.Components;

namespace BlazorSample
{
    public class BlazorRocksBase : ComponentBase
    {
        public string BlazorRocksText { get; set; } = 
            "Blazor rocks the browser!";
    }
}
```

### <a name="use-components"></a>コンポーネントを使う

コンポーネントには、HTML 要素構文を使用して宣言することで、他のコンポーネントを含めることができます。 コンポーネントを使うためのマークアップは、そのコンポーネントの種類をタグ名とする HTML タグのようになります。

`Pages/Index.razor` の次のマークアップでは、`HeadingComponent` インスタンスがレンダリングされます。

```razor
<HeadingComponent />
```

`Components/HeadingComponent.razor`:

[!code-razor[](index/samples_snapshot/HeadingComponent.razor)]

コンポーネント名と一致しない最初の文字が大文字の HTML 要素がコンポーネントに含まれている場合、要素に予期しない名前が付いていることを示す警告が出力されます。 コンポーネントの名前空間に [`@using`][2] ディレクティブを追加すると、コンポーネントを使用できるようになり、警告が解決されます。

## <a name="parameters"></a>パラメーター

### <a name="route-parameters"></a>ルート パラメーター

コンポーネントでは、[`@page`][9] ディレクティブに指定されたルート テンプレートからルート パラメーターを受け取ることができます。 ルーターでは、ルート パラメーターを使用して、対応するコンポーネント パラメーターが設定されます。

::: moniker range=">= aspnetcore-5.0"

省略可能なパラメーターがサポートされています。 次の例では、省略可能なパラメーター `text` を使用して、ルート セグメントの値をコンポーネントの `Text` プロパティに割り当てます。 セグメントが存在しない場合、`Text` の値は `fantastic` に設定されます。

`Pages/RouteParameter.razor`:

[!code-razor[](index/samples_snapshot/RouteParameter-5x.razor?highlight=1,6-7)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

`Pages/RouteParameter.razor`:

[!code-razor[](index/samples_snapshot/RouteParameter-3x.razor?highlight=2,7-8)]

オプションのパラメーターはサポートされていないため、前の例では 2 つの [`@page`][9] ディレクティブが適用されます。 1 つ目は、パラメーターを指定せずにコンポーネントへの移動を許可します。 2 番目の [`@page`][9] ディレクティブでは、`{text}` ルート パラメーターを受け取り、その値を `Text` プロパティに割り当てます。

::: moniker-end

複数のフォルダーにわたりパスをキャプチャするキャッチオール ルート パラメーター (`{*pageRoute}`) の詳細については、「<xref:blazor/fundamentals/routing#catch-all-route-parameters>」を参照してください。

### <a name="component-parameters"></a>コンポーネントのパラメーター

コンポーネントには、"*コンポーネント パラメーター*" を指定できます。これは、[`[Parameter]` 属性](xref:Microsoft.AspNetCore.Components.ParameterAttribute)を指定したコンポーネント クラス上で、単純な、または複雑なパブリック プロパティを使用して定義します。 マークアップ内でコンポーネントの引数を指定するには、属性を使います。

`Components/ChildComponent.razor`:

[!code-razor[](../common/samples/5.x/BlazorWebAssemblySample/Components/ChildComponent.razor?highlight=2,11-12)]

コンポーネント パラメーターには、既定値を割り当てることができます。

```csharp
[Parameter]
public string Title { get; set; } = "Panel Title from Child";
```

サンプル アプリの次の例では、`ParentComponent` によって `ChildComponent` の `Title` プロパティの値を設定しています。

`Pages/ParentComponent.razor`:

[!code-razor[](index/samples_snapshot/ParentComponent.razor?highlight=5-6)]

慣例により、C# コードで構成される属性値は、[Razor の予約済み `@` シンボル](xref:mvc/views/razor#razor-syntax)を使用してパラメーターに割り当てられます。

* 親フィールドまたはプロパティ: `Title="@{FIELD OR PROPERTY}`。プレースホルダー `{FIELD OR PROPERTY}` は親コンポーネントの C# フィールドまたはプロパティです。
* メソッドの結果: `Title="@{METHOD}"`。プレースホルダー `{METHOD}` は親コンポーネントの C# メソッドです。
* [暗黙的または明示的な式](xref:mvc/views/razor#implicit-razor-expressions): `Title="@({EXPRESSION})"`。プレースホルダー `{EXPRESSION}` は C# 式です。
  
詳細については、「[ASP.NET Coreの Razor 構文リファレンス](xref:mvc/views/razor)」を参照してください。

> [!WARNING]
> 独自の "*コンポーネント パラメーター*" を書き込み先とするコンポーネントを作成する代わりに、プライベート フィールドを使用してください。 詳細については、「[上書きされたパラメーター](#overwritten-parameters)」セクションをご覧ください。

## <a name="child-content"></a>子コンテンツ

コンポーネントでは、別のコンポーネントのコンテンツを設定できます。 割り当てコンポーネントでは、受信コンポーネントを指定するタグ間にコンテンツを指定します。

次の例では、`ChildComponent` に、レンダリングする UI のセグメントを表す <xref:Microsoft.AspNetCore.Components.RenderFragment> を表す `ChildContent` プロパティがあります。 コンテンツをレンダリングする必要があるコンポーネントのマークアップに、`ChildContent` の値を配置します。 `ChildContent` の値は、親コンポーネントから受け取られ、ブートストラップ パネルの `panel-body` 内にレンダリングされます。

`Components/ChildComponent.razor`:

[!code-razor[](../common/samples/5.x/BlazorWebAssemblySample/Components/ChildComponent.razor?highlight=3,14-15)]

> [!NOTE]
> <xref:Microsoft.AspNetCore.Components.RenderFragment> コンテンツを受け取るプロパティは、規則によって `ChildContent` という名前にする必要があります。

サンプル アプリの `ParentComponent` では、コンテンツを `<ChildComponent>` タグ内に配置することによって、`ChildComponent` をレンダリングするためのコンテンツを提供できます。

`Pages/ParentComponent.razor`:

[!code-razor[](index/samples_snapshot/ParentComponent.razor?highlight=7-8)]

Blazor による子コンテンツのレンダリング方法により、`for` ループ内のコンポーネントのレンダリングでは、インクリメントするループ変数が子コンポーネントのコンテンツ内で使用されている場合、ローカル インデックス変数が必要になります。
>
> ```razor
> @for (int c = 0; c < 10; c++)
> {
>     var current = c;
>     <ChildComponent Title="@c">
>         Child Content: Count: @current
>     </ChildComponent>
> }
> ```
>
> または、<xref:System.Linq.Enumerable.Range%2A?displayProperty=nameWithType> と共に `foreach` ループを使用します。
>
> ```razor
> @foreach(var c in Enumerable.Range(0,10))
> {
>     <ChildComponent Title="@c">
>         Child Content: Count: @c
>     </ChildComponent>
> }
> ```

<xref:Microsoft.AspNetCore.Components.RenderFragment> を Razor コンポーネント UI のテンプレートとして使用する方法については、次の記事を参照してください。

* <xref:blazor/components/templated-components>
* <xref:blazor/webassembly-performance-best-practices#define-reusable-renderfragments-in-code>

## <a name="attribute-splatting-and-arbitrary-parameters"></a>属性スプラッティングと任意のパラメーター

コンポーネントでは、コンポーネントの宣言されたパラメーターに加えて、追加の属性をキャプチャしてレンダリングできます。 追加の属性は、ディクショナリにキャプチャし、[`@attributes`][3] Razor ディレクティブを使用して、コンポーネントがレンダリングされるときに、要素に "*スプラッティング*" できます。 このシナリオは、さまざまなカスタマイズをサポートするマークアップ要素を生成するコンポーネントを定義する場合に便利です。 たとえば、多くのパラメーターをサポートする `<input>` に対して、属性を個別に定義するのは面倒な場合があります。

次の例で、最初の `<input>` 要素 (`id="useIndividualParams"`) では、個々のコンポーネント パラメーターを使用していますが、2 番目の `<input>` 要素 (`id="useAttributesDict"`) では、属性スプラッティングを使用しています。

```razor
<input id="useIndividualParams"
       maxlength="@maxlength"
       placeholder="@placeholder"
       required="@required"
       size="@size" />

<input id="useAttributesDict"
       @attributes="InputAttributes" />

@code {
    public string maxlength = "10";
    public string placeholder = "Input placeholder text";
    public string required = "required";
    public string size = "50";

    public Dictionary<string, object> InputAttributes { get; set; } =
        new Dictionary<string, object>()
        {
            { "maxlength", "10" },
            { "placeholder", "Input placeholder text" },
            { "required", "required" },
            { "size", "50" }
        };
}
```

パラメーターの型は、文字列キーで `IEnumerable<KeyValuePair<string, object>>` または `IReadOnlyDictionary<string, object>` を実装する必要があります。

両方の方法を使用してレンダリングされる `<input>` 要素は同じです。

```html
<input id="useIndividualParams"
       maxlength="10"
       placeholder="Input placeholder text"
       required="required"
       size="50">

<input id="useAttributesDict"
       maxlength="10"
       placeholder="Input placeholder text"
       required="required"
       size="50">
```

任意の属性を受け入れるには、<xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> プロパティを `true` に設定した [`[Parameter]` 属性](xref:Microsoft.AspNetCore.Components.ParameterAttribute)を使用して、コンポーネント パラメーターを定義します。

```razor
@code {
    [Parameter(CaptureUnmatchedValues = true)]
    public Dictionary<string, object> InputAttributes { get; set; }
}
```

[`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute) の <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> プロパティにより、パラメーターを他のパラメーターと一致しないすべての属性と一致させることができます。 1 つのコンポーネントで、<xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> を持つパラメーターは 1 つだけ定義できます。 <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> で使用されるプロパティの型は、文字列キーを使用して `Dictionary<string, object>` から割り当て可能である必要があります。 このシナリオでは、`IEnumerable<KeyValuePair<string, object>>` または `IReadOnlyDictionary<string, object>` も使用できます。

要素属性の位置を基準とした [`@attributes`][3] の位置は重要です。 [`@attributes`][3] が要素にスプラッティングされると、属性は右から左 (最後から最初) に処理されます。 `Child` コンポーネントを使用する次のコンポーネントの例を考えます。

`ParentComponent.razor`:

```razor
<ChildComponent extra="10" />
```

`ChildComponent.razor`:

```razor
<div @attributes="AdditionalAttributes" extra="5" />

[Parameter(CaptureUnmatchedValues = true)]
public IDictionary<string, object> AdditionalAttributes { get; set; }
```

`Child` コンポーネントの `extra` 属性が [`@attributes`][3] の右側に設定されています。 属性は右から左 (最後から最初) に処理されるため、追加の属性によって渡された場合に、`Parent` コンポーネントのレンダリングされる `<div>` に、`extra="5"` が含まれます。

```html
<div extra="5" />
```

次の例では、`Child` コンポーネントの `<div>` で、`extra` と [`@attributes`][3] の順序が逆になります。

`ParentComponent.razor`:

```razor
<ChildComponent extra="10" />
```

`ChildComponent.razor`:

```razor
<div extra="5" @attributes="AdditionalAttributes" />

[Parameter(CaptureUnmatchedValues = true)]
public IDictionary<string, object> AdditionalAttributes { get; set; }
```

追加の属性によって渡された場合に、`Parent` コンポーネント内のレンダリングされる `<div>` には、`extra="10"` が含まれます。

```html
<div extra="10" />
```

## <a name="capture-references-to-components"></a>コンポーネントへの参照をキャプチャする

コンポーネント参照によって、コンポーネント インスタンスを参照する方法が得られるため、そのインスタンスに `Show` や `Reset` などのコマンドを発行できます。 コンポーネント参照をキャプチャするには:

* 子コンポーネントに [`@ref`][4] 属性を追加します。
* 子コンポーネントと同じ型のフィールドを定義します。

```razor
<CustomLoginDialog @ref="loginDialog" ... />

@code {
    private CustomLoginDialog loginDialog;

    private void OnSomething()
    {
        loginDialog.Show();
    }
}
```

コンポーネントがレンダリングされると、`loginDialog` フィールドに `CustomLoginDialog` 子コンポーネント インスタンスが設定されます。 これにより、コンポーネント インスタンスに対し、.NET メソッドを呼び出すことができます。

> [!IMPORTANT]
> `loginDialog` 変数は、コンポーネントがレンダリングされた後にのみ設定され、その出力には `MyLoginDialog` 要素が含まれます。 コンポーネントがレンダリングされるまで、参照するものはありません。
>
> コンポーネントのレンダリングが完了した後にコンポーネント参照を操作するには、[`OnAfterRenderAsync`メソッドまたは `OnAfterRender` メソッド](xref:blazor/components/lifecycle#after-component-render)を使用します。
>
> イベント ハンドラーで参照変数を使用するには、ラムダ式を使用するか、[`OnAfterRenderAsync` または `OnAfterRender` メソッド](xref:blazor/components/lifecycle#after-component-render)でイベント ハンドラー デリゲートを割り当てます。 これにより、イベント ハンドラーが割り当てられる前に参照変数が確実に割り当てられます。
>
> ```razor
> <button type="button" 
>     @onclick="@(() => loginDialog.DoSomething())">Do Something</button>
>
> <MyLoginDialog @ref="loginDialog" ... />
>
> @code {
>     private MyLoginDialog loginDialog;
> }
> ```

ループ内のコンポーネントを参照するには、「[Capture references to multiple similar child-components](https://github.com/dotnet/aspnetcore/issues/13358)」(複数の類似した子コンポーネントへの参照をキャプチャする) (dotnet/aspnetcore #13358) を参照してください。

コンポーネント参照のキャプチャでは、[要素参照のキャプチャ](xref:blazor/call-javascript-from-dotnet#capture-references-to-elements)と類似の構文を使用しますが、それは JavaScript 相互運用機能ではありません。 コンポーネント参照は、JavaScript コードに渡されません。 コンポーネント参照は、.NET コードでのみ使用されます。

> [!NOTE]
> 子コンポーネントの状態を変えるためにコンポーネント参照を使用 **しない** でください。 代わりに、通常の宣言型パラメーターを使用して、子コンポーネントにデータを渡します。 通常の宣言型パラメーターを使用すると、子コンポーネントが正しいタイミングで自動的にレンダリングされます。

## <a name="synchronization-context"></a>同期コンテキスト

Blazor では、同期コンテキスト (<xref:System.Threading.SynchronizationContext>) を使用して、1 つの実行の論理スレッドを強制します。 コンポーネントの[ライフサイクル メソッド](xref:blazor/components/lifecycle)と、Blazor によって発生するすべてのイベント コールバックは、同期コンテキストで実行されます。

Blazor Server の同期コンテキストでは、ブラウザーの WebAssembly モデル (シングル スレッド) と厳密に一致するように、シングルスレッド環境のエミュレートが試行されます。 どの時点でも、作業は 1 つのスレッドでのみ実行され、1 つの論理スレッドであるという印象になります。 2 つの操作が同時に実行されることはありません。

### <a name="avoid-thread-blocking-calls"></a>スレッドをブロックする呼び出しを避ける

一般に、次のメソッドは呼び出さないでください。 次のメソッドでは、スレッドがブロックされます。そのため、基になる <xref:System.Threading.Tasks.Task> が完了するまで、アプリの動作が再開されなくなります。

* <xref:System.Threading.Tasks.Task%601.Result%2A>
* <xref:System.Threading.Tasks.Task.Wait%2A>
* <xref:System.Threading.Tasks.Task.WaitAny%2A>
* <xref:System.Threading.Tasks.Task.WaitAll%2A>
* <xref:System.Threading.Thread.Sleep%2A>
* <xref:System.Runtime.CompilerServices.TaskAwaiter.GetResult%2A>

### <a name="invoke-component-methods-externally-to-update-state"></a>状態を更新するために外部でコンポーネント メソッドを呼び出す

タイマーやその他の通知などの外部のイベントに基づいてコンポーネントを更新する必要がある場合は、`InvokeAsync` メソッドを使用します。これにより、Blazor の同期コンテキストにディスパッチされます。 たとえば、リッスンしているコンポーネントに、更新状態を通知できる *通知サービス* を考えてみます。

```csharp
public class NotifierService
{
    // Can be called from anywhere
    public async Task Update(string key, int value)
    {
        if (Notify != null)
        {
            await Notify.Invoke(key, value);
        }
    }

    public event Func<string, int, Task> Notify;
}
```

`NotifierService` を登録します。

* Blazor WebAssembly で、`Program.Main` のシングルトンとしてサービスを登録します。

  ```csharp
  builder.Services.AddSingleton<NotifierService>();
  ```

* Blazor Server で、`Startup.ConfigureServices` のスコープとしてサービスを登録します。

  ```csharp
  services.AddScoped<NotifierService>();
  ```

`NotifierService` を使用して、コンポーネントを更新します。

```razor
@page "/"
@inject NotifierService Notifier
@implements IDisposable

<p>Last update: @lastNotification.key = @lastNotification.value</p>

@code {
    private (string key, int value) lastNotification;

    protected override void OnInitialized()
    {
        Notifier.Notify += OnNotify;
    }

    public async Task OnNotify(string key, int value)
    {
        await InvokeAsync(() =>
        {
            lastNotification = (key, value);
            StateHasChanged();
        });
    }

    public void Dispose()
    {
        Notifier.Notify -= OnNotify;
    }
}
```

前の例の場合:

* Blazor の同期コンテキスト外で `NotifierService` からコンポーネントの `OnNotify` メソッドが呼び出されます。 `InvokeAsync` を使用して、正しいコンテキストに切り替え、レンダリングをキューに登録します。 詳細については、「<xref:blazor/components/rendering>」を参照してください。
* コンポーネントは <xref:System.IDisposable> を実装し、`OnNotify` デリゲートは `Dispose` メソッドでサブスクライブを解除します。このメソッドは、コンポーネントが破棄されたときにフレームワークによって呼び出されます。 詳細については、「<xref:blazor/components/lifecycle#component-disposal-with-idisposable>」を参照してください。

## <a name="use-key-to-control-the-preservation-of-elements-and-components"></a>\@ キーを使用して要素とコンポーネントの保存を制御する

要素またはコンポーネントのリストをレンダリングし、その後に要素またはコンポーネントが変更された場合、Blazor の比較アルゴリズムでは、前のどの要素やコンポーネントを保持できるか、およびモデル オブジェクトをそれらにどのようにマップするかを決定する必要があります。 通常、このプロセスは自動で、無視できますが、プロセスの制御が必要になる場合があります。

次に例を示します。

```csharp
@foreach (var person in People)
{
    <DetailsEditor Details="@person.Details" />
}

@code {
    [Parameter]
    public IEnumerable<Person> People { get; set; }
}
```

`People` コレクションのコンテンツは、挿入、削除、または順序変更されたエントリによって変更される可能性があります。 コンポーネントのレンダリング時に、`<DetailsEditor>` コンポーネントが変更され、異なる `Details` パラメーター値を受け取ることがあります。 これにより、予期したものよりも複雑な再レンダリングが発生する可能性があります。 場合によっては、再レンダリングによって、要素のフォーカスの喪失などの表示動作の違いが発生する可能性があります。

マッピング プロセスは、[`@key`][5] ディレクティブ属性を使用して制御できます。 [`@key`][5] により、比較アルゴリズムで、キーの値に基づいて要素やコンポーネントが確実に保持されます。

```csharp
@foreach (var person in People)
{
    <DetailsEditor @key="person" Details="@person.Details" />
}

@code {
    [Parameter]
    public IEnumerable<Person> People { get; set; }
}
```

`People` コレクションが変更されても、比較アルゴリズムによって、`<DetailsEditor>` インスタンスと `person` インスタンス間の関連付けが保持されます。

* `Person` が `People` リストから削除された場合、対応する `<DetailsEditor>` インスタンスだけが UI から削除されます。 他のインスタンスは変更されません。
* リスト内の特定の位置に `Person` が挿入されると、その対応する位置に、1 つの新しい `<DetailsEditor>` インスタンスが挿入されます。 他のインスタンスは変更されません。
* `Person` エントリの順序が変更された場合、対応する `<DetailsEditor>` インスタンスは UI で保持され、順序が変更されます。

シナリオによっては、[`@key`][5] を使用すると、レンダリングの複雑さが最小限に抑えられ、フォーカス位置など、DOM 変更のステートフルな部分の潜在的な問題を回避できます。

> [!IMPORTANT]
> キーは、各コンテナー要素やコンポーネントに対してローカルです。 キーはドキュメント全体でグローバルに比較されません。

### <a name="when-to-use-key"></a>\@ キーを使用する場面

一般に、リストがレンダリングされ (たとえば、[foreach](/dotnet/csharp/language-reference/keywords/foreach-in) ブロックで)、[`@key`][5] を定義するための適切な値が存在する場合は常に、[`@key`][5] を使用することは意味があります。

また、[`@key`][5] を使用して、オブジェクトが変更されたときに Blazor が要素やコンポーネントのサブツリーを保持しないようにすることもできます。

```razor
<div @key="currentPerson">
    ... content that depends on currentPerson ...
</div>
```

`@currentPerson` が変更された場合、[`@key`][5] 属性ディレクティブによって、Blazor に、`<div>` とその子孫全体を破棄させ、新しい要素とコンポーネントで UI 内のサブツリーをリビルドさせます。 これは、`@currentPerson` が変更されたときに、UI の状態が確実に保持されないようにする必要がある場合に役立つ可能性があります。

### <a name="when-not-to-use-key"></a>\@ キーを使用しない場面

[`@key`][5] で比較すると、パフォーマンスが低下します。 パフォーマンスの低下は大きくありませんが、要素やコンポーネントの保存規則を制御することによって、アプリにメリットがある場合にのみ [`@key`][5] を指定してください。

[`@key`][5] を使用しない場合でも、Blazor では可能な限り、子要素とコンポーネント インスタンスが保持されます。 [`@key`][5] を使用する唯一の利点は、マッピングを選択する比較アルゴリズムではなく、保持されているコンポーネント インスタンスにモデル インスタンスをマップする "*方法*" を制御することです。

### <a name="what-values-to-use-for-key"></a>\@ キーに使用する値

一般に、[`@key`][5] には、次のいずれかの種類の値を指定するのが適切です。

* モデル オブジェクトインスタンス (たとえば、前の例のように、`Person` インスタンス)。 これにより、オブジェクト参照の等価性に基づいて保持されます。
* 一意の識別子 (たとえば、`int` 型、`string` 型、`Guid` 型の主キー値)。

[`@key`][5] に使用される値は確実に競合しないようにしてください。 同じ親要素内で競合する値が検出された場合、Blazor では、古い要素やコンポーネントを新しい要素やコンポーネントに確定的にマップできないため、例外がスローされます。 個別の値 (オブジェクト インスタンスや主キー値など) のみを使用してください。

## <a name="overwritten-parameters"></a>上書きされたパラメーター

Blazor フレームワークでは、一般に安全な親から子へのパラメーターの割り当てが行われます。

* パラメーターが予期せずに上書きされることはありません。
* 副作用は最小限に抑えられます。 たとえば、追加のレンダリングは、無限のレンダリング ループが作成される可能性があるため、回避されます。

子コンポーネントは、親コンポーネントのレンダリング時に既存の値を上書きする可能性がある新しいパラメーター値を受け取ります。 子コンポーネントでパラメーター値が誤って上書きされることは、1 つまたは複数のデータバインド パラメーターを使用してコンポーネントを開発しており、開発者が子のパラメーターに直接書き込む場合に多く発生します。

* 子コンポーネントは、親コンポーネントの 1 つまたは複数のパラメーター値を使用してレンダリングされます。
* 子によって、パラメーターの値が直接書き込まれます。
* 親コンポーネントがレンダリングされ、子のパラメーターの値が上書きされます。

パラメーター値の上書きの可能性は、子コンポーネントのプロパティ セッターにも及びます。

**一般的なガイダンスとして、独自のパラメーターに直接書き込むコンポーネントを作成しないでください。**

次が実行される、問題のある `Expander` コンポーネントについて考えてみましょう。

* 子コンテンツのレンダリング。
* コンポーネント パラメーター (`Expanded`) を使用した、子コンテンツの表示の切り替え。
* コンポーネントによって、`Expanded` パラメーターに直接書き込まれます。これは上書きされるパラメーターの問題を示しているため、回避する必要があります。

```razor
<div @onclick="Toggle" class="card bg-light mb-3" style="width:30rem">
    <div class="card-body">
        <h2 class="card-title">Toggle (<code>Expanded</code> = @Expanded)</h2>

        @if (Expanded)
        {
            <p class="card-text">@ChildContent</p>
        }
    </div>
</div>

@code {
    [Parameter]
    public bool Expanded { get; set; }

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    private void Toggle()
    {
        Expanded = !Expanded;
    }
}
```

`Expander` コンポーネントは、<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> を呼び出す可能性のある親コンポーネントに追加されます。

```razor
@page "/expander"

<Expander Expanded="true">
    Expander 1 content
</Expander>

<Expander Expanded="true" />

<button @onclick="StateHasChanged">
    Call StateHasChanged
</button>
```

初期状態では、`Expanded` プロパティが切り替えられると、`Expander` コンポーネントはそれぞれ独立して動作します。 子コンポーネントの状態は、想定どおりのままです。 親で <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> が呼び出されると、最初の子コンポーネントの `Expanded` パラメーターが初期値 (`true`) にリセットされます。 2 つめの `Expander` コンポーネントの `Expanded` 値はリセットされません。これは、2 つめのコンポーネントでは子コンテンツがレンダリングされないためです。

前のシナリオでの状態を維持するには、`Expander` コンポーネントで "*プライベート フィールド*" を使用して、切り替え状態を維持します。

次の変更された `Expander` コンポーネント:

* 親から `Expanded` コンポーネント パラメーター値を受け入れます。
* コンポーネント パラメーター値を、[OnInitialized イベント](xref:blazor/components/lifecycle#component-initialization-methods) の "*プライベート フィールド*" (`expanded`) に割り当てます。
* プライベート フィールドを使用して、その内部のトグル状態を維持します。これは、パラメーターに直接書き込まれないようにする方法を示しています。

```razor
<div @onclick="Toggle" class="card bg-light mb-3" style="width:30rem">
    <div class="card-body">
        <h2 class="card-title">Toggle (<code>expanded</code> = @expanded)</h2>

        @if (expanded)
        {
            <p class="card-text">@ChildContent</p>
        }
    </div>
</div>

@code {
    private bool expanded;

    [Parameter]
    public bool Expanded { get; set; }

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    protected override void OnInitialized()
    {
        expanded = Expanded;
    }

    private void Toggle()
    {
        expanded = !expanded;
    }
}
```

詳細については、[Blazor両方向のバインド エラー (dotnet/aspnetcore #24599)](https://github.com/dotnet/aspnetcore/issues/24599) に関するページを参照してください。 

## <a name="apply-an-attribute"></a>属性を適用する

属性は、[`@attribute`][7] ディレクティブを使用して Razor コンポーネントに適用できます。 次の例では、[`[Authorize]` 属性](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute)をコンポーネント クラスに適用しています。

```razor
@page "/"
@attribute [Authorize]
```

## <a name="conditional-html-element-attributes"></a>条件付き HTML 要素属性

HTML 要素属性は、.NET 値に基づいて条件付きでレンダリングされます。 値が `false` または `null` の場合、属性はレンダリングされません。 値が `true` の場合、属性が最小化されてレンダリングされます。

次の例では、`IsCompleted` によって `checked` が要素のマークアップにレンダリングされるかどうかを決定します。

```razor
<input type="checkbox" checked="@IsCompleted" />

@code {
    [Parameter]
    public bool IsCompleted { get; set; }
}
```

`IsCompleted` が `true` の場合、このチェック ボックスは次のようにレンダリングされます。

```html
<input type="checkbox" checked />
```

`IsCompleted` が `false` の場合、このチェック ボックスは次のようにレンダリングされます。

```html
<input type="checkbox" />
```

詳細については、「[ASP.NET Coreの Razor 構文リファレンス](xref:mvc/views/razor)」を参照してください。

> [!WARNING]
> .NET 型が `bool` の場合、[`aria-pressed`](https://developer.mozilla.org/docs/Web/Accessibility/ARIA/Roles/button_role#Toggle_buttons) などの一部の HTML 属性が正しく機能しません。 そのような場合は、`bool` ではなく `string` 型を使用します。

## <a name="raw-html"></a>生 HTML

通常、文字列は DOM テキスト ノードを使用してレンダリングされます。つまり、それらに含まれている可能性のあるすべてのマークアップが無視され、リテラル テキストとして扱われます。 生 HTML をレンダリングするには、HTML コンテンツを `MarkupString` 値にラップします。 値は HTML または SVG として解析され、DOM に挿入されます。

> [!WARNING]
> 信頼されていないソースから構築された生 HTML をレンダリングすることは、**セキュリティ リスク** であるため、避ける必要があります。

次の例では、`MarkupString` 型を使用して、コンポーネントのレンダリングされた出力に静的 HTML コンテンツのブロックを追加しています。

```html
@((MarkupString)myMarkup)

@code {
    private string myMarkup = 
        "<p class='markup'>This is a <em>markup string</em>.</p>";
}
```

## <a name="razor-templates"></a>Razor テンプレート

レンダリング フラグメントは、Razor テンプレート構文を使用して定義できます。 Razor テンプレートは、UI スニペットを定義する 1 つの方法であり、次の形式を想定しています。

```razor
@<{HTML tag}>...</{HTML tag}>
```

次の例では、<xref:Microsoft.AspNetCore.Components.RenderFragment> と <xref:Microsoft.AspNetCore.Components.RenderFragment%601> の値を指定し、コンポーネント内にテンプレートを直接レンダリングする方法を示しています。 レンダリング フラグメントは、引数として[テンプレート コンポーネント](xref:blazor/components/templated-components)に渡すこともできます。

```razor
@timeTemplate

@petTemplate(new Pet { Name = "Rex" })

@code {
    private RenderFragment timeTemplate = @<p>The time is @DateTime.Now.</p>;
    private RenderFragment<Pet> petTemplate = (pet) => @<p>Pet: @pet.Name</p>;

    private class Pet
    {
        public string Name { get; set; }
    }
}
```

前のコードのレンダリングされた結果:

```html
<p>The time is 10/04/2018 01:26:52.</p>

<p>Pet: Rex</p>
```

## <a name="static-assets"></a>静的な資産

Blazor は、プロジェクトの [`web root (wwwroot)` フォルダー](xref:fundamentals/index#web-root)に静的アセットを配置する ASP.NET Core アプリの規則に従います。

静的アセットの Web ルートを参照するには、ベース相対パス (`/`) を使用します。 次の例では、`logo.png` が物理的に `{PROJECT ROOT}/wwwroot/images` フォルダーに配置されています。

```razor
<img alt="Company logo" src="/images/logo.png" />
```

Razor コンポーネントでは、チルダ スラッシュ表記 (`~/`) はサポートされて **いません**。

アプリのベース パスの設定の詳細については、「<xref:blazor/host-and-deploy/index#app-base-path>」を参照してください。

## <a name="tag-helpers-arent-supported-in-components"></a>タグ ヘルパーはコンポーネントでサポートされない

[`Tag Helpers`](xref:mvc/views/tag-helpers/intro) は、Razor コンポーネント (`.razor` ファイル) ではサポートされていません。 Blazor にタグ ヘルパーのような機能を提供するには、タグ ヘルパーと同じ機能を持つコンポーネントを作成し、代わりにそのコンポーネントを使用します。

## <a name="scalable-vector-graphics-svg-images"></a>スケーラブル ベクター グラフィックス (SVG) イメージ

Blazor では HTML がレンダリングされるため、スケーラブル ベクター グラフィックス (SVG) 画像 (`.svg`) などのブラウザーでサポートされている画像は、`<img>` タグを介してサポートされます。

```html
<img alt="Example image" src="some-image.svg" />
```

同様に、SVG 画像は、スタイルシート ファイル (`.css`) の CSS 規則でサポートされています。

```css
.my-element {
    background-image: url("some-image.svg");
}
```

ただし、インライン SVG マークアップは、すべてのシナリオでサポートされているわけではありません。 `<svg>` タグをコンポーネント ファイル (`.razor`) に直接配置した場合、基本的な画像レンダリングはサポートされますが、多くの高度なシナリオはまだサポートされていません。 たとえば、`<use>` タグは現在考慮されないため、一部の SVG タグで [`@bind`][10] を使用できません。 詳細については、[Blazor の SVG サポート (dotnet/aspnetcore #18271)](https://github.com/dotnet/aspnetcore/issues/18271)に関する記事を参照してください。

## <a name="whitespace-rendering-behavior"></a>空白文字のレンダリング動作

::: moniker range=">= aspnetcore-5.0"

[`@preservewhitespace`](xref:mvc/views/razor#preservewhitespace) ディレクティブが `true`の値と共に使用されている場合を除き、既定では、次の場合に余分な空白が削除されます。

* 要素内の先頭または末尾。
* `RenderFragment` パラメーター内の先頭または末尾。 別のコンポーネントに渡される子コンテンツなどです。
* `@if` または `@foreach` のような、C# コード ブロックの前か後にある。

空白文字の削除は、`white-space: pre` などの CSS ルールを使用するときに、レンダリングされた出力に影響を与えることがあります。 このパフォーマンスの最適化を無効にして、空白を保持するには、次のいずれかの操作を実行します。

* 特定のコンポーネントに設定を適用するには、 `.razor` ファイルの先頭に `@preservewhitespace true` ディレクティブを追加する。
* サブディレクトリ全体またはプロジェクト全体に設定を適用するには、 `_Imports.razor` ファイル内に `@preservewhitespace true` ディレクティブを追加する。

ほとんどの場合、アプリでは一般的に通常の動作が続行されるため (ただし、速くなります)、何の措置も必要ありません。 空白文字の削除で特定のコンポーネントに問題が発生する場合、そのコンポーネントで `@preservewhitespace true` を使用し、この最適化を無効にします。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

空白は、コンポーネントのソース コードに保持されます。 空白文字のみのテキストは、視覚効果がないときでも、ブラウザーのドキュメント オブジェクト モデル (DOM) にレンダリングされます。

次の Razor コンポーネント コードについて考えてみましょう。

```razor
<ul>
    @foreach (var item in Items)
    {
        <li>
            @item.Text
        </li>
    }
</ul>
```

前の例では、次の不要な空白文字がレンダリングされます。

* `@foreach` コード ブロックの外側。
* `<li>` 要素の前後。
* `@item.Text` 出力の前後。

100 項目を含むリストでは、402 の空白領域になり、余分な空白が、レンダリングされる出力に視覚的に影響を及ぼすことはありません。

コンポーネントの静的 HTML をレンダリングする場合、タグ内の空白文字は保持されません。 たとえば、レンダリングされた出力の次のコンポーネントのソースをご覧ください。

```razor
<img     alt="My image"   src="img.png"     />
```

前の Razor マークアップからの、空白は保持されません。

```razor
<img alt="My image" src="img.png" />
```

::: moniker-end

## <a name="additional-resources"></a>その他のリソース

* <xref:blazor/security/server/threat-mitigation>:リソース不足に対処する必要がある Blazor Server アプリの構築に関するガイダンスが含まれています。

<!--Reference links in article-->
[1]: <xref:mvc/views/razor#code> "ASP.NET Core の Razor 構文リファレンス"
[2]: <xref:mvc/views/razor#using> "ASP.NET Core の Razor 構文リファレンス"
[3]: <xref:mvc/views/razor#attributes> "ASP.NET Core の Razor 構文リファレンス"
[4]: <xref:mvc/views/razor#ref> "ASP.NET Core の Razor 構文リファレンス"
[5]: <xref:mvc/views/razor#key> "ASP.NET Core の Razor 構文リファレンス"
[6]: <xref:mvc/views/razor#inherits> "ASP.NET Core の Razor 構文リファレンス"
[7]: <xref:mvc/views/razor#attribute> "ASP.NET Core の Razor 構文リファレンス"
[8]: <xref:mvc/views/razor#namespace> "ASP.NET Core の Razor 構文リファレンス"
[9]: <xref:mvc/views/razor#page> "ASP.NET Core の Razor 構文リファレンス"
[10]: <xref:mvc/views/razor#bind> "ASP.NET Core の Razor 構文リファレンス"
