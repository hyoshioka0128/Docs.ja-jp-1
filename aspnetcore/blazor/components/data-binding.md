---
title: ASP.NET Core Blazor データ バインディング
author: guardrex
description: Blazor アプリのコンポーネントとドキュメント オブジェクト モデル (DOM) 要素のデータ バインディング機能について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/15/2021
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
uid: blazor/components/data-binding
ms.openlocfilehash: 5f1e9963a7b62b90ee492bebe9bfc357a8090caf
ms.sourcegitcommit: b81327f1a62e9857d9e51fb34775f752261a88ae
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/25/2021
ms.locfileid: "105051037"
---
# <a name="aspnet-core-blazor-data-binding"></a>ASP.NET Core Blazor データ バインディング

Razor コンポーネントには、フィールド、プロパティ、または Razor 式の値が含まれる [`@bind`](xref:mvc/views/razor#bind) Razor ディレクティブ属性を使用したデータ バインディング機能が用意されています。

以下の例で行われているバインドは次のとおりです。

* `<input>` 要素の値を C# `inputValue` フィールドに。
* 2 番目の `<input>` 要素の値を C# `InputValue` プロパティに。

`<input>` 要素がフォーカスを失うと、そのバインドされたフィールドまたはプロパティが更新されます。

`Pages/Bind.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/data-binding/Bind.razor?highlight=4,8)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/data-binding/Bind.razor?highlight=4,8)]

::: moniker-end

テキスト ボックスは、フィールドまたはプロパティの値の変更に対する応答としてではなく、コンポーネントがレンダリングされたときにのみ UI が更新されます。 コンポーネントはイベント ハンドラーのコードが実行された後に自身をレンダリングするため、通常は、イベント ハンドラーがトリガーされた直後に、フィールドとプロパティの更新が UI に反映されます。

次の例では、データ バインディングが HTML でどのように構成されるかを示すために、`InputValue` プロパティを 2 番目の `<input>` 要素の `value` および [`onchange`](https://developer.mozilla.org/docs/Web/API/GlobalEventHandlers/onchange) 属性にバインドします。 *次の例の 2 番目の `<input>` 要素は概念実証であって、Razor コンポーネントでデータをバインドする方法を提案するものではありません。*

`Pages/BindTheory.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/data-binding/BindTheory.razor?highlight=12-14)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/data-binding/BindTheory.razor?highlight=12-14)]

::: moniker-end

`BindTheory` コンポーネントがレンダリングされると、HTML 実証の `<input>` 要素の `value` は `InputValue` プロパティから取得されます。 ユーザーがテキスト ボックスに値を入力し、要素のフォーカスを変更すると、`onchange` イベントが発生し、`InputValue` プロパティは変更された値に設定されます。 実際には、[`@bind`](xref:mvc/views/razor#bind) で扱われるのは、型変換が行われるケースであるため、コード生成はより複雑になります。 一般に、[`@bind`](xref:mvc/views/razor#bind) では、式の現在の値を `value` 属性と関連付けてから、登録されたハンドラーを使用して変更を処理します。

`{EVENT}` プレースホルダーに DOM イベントを使用した `@bind:event="{EVENT}"` 属性を含めることにより、他の[ドキュメント オブジェクト モデル (DOM)](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction) イベントのプロパティまたはフィールドをバインドします。 次の例では、`<input>` 要素の [`oninput` イベント](https://developer.mozilla.org/docs/Web/API/GlobalEventHandlers/oninput)がトリガーされたときに、その要素の値に `InputValue` プロパティをバインドします。 要素がフォーカスを失ったときに発生する [`onchange` イベント](https://developer.mozilla.org/docs/Web/API/GlobalEventHandlers/onchange)とは異なり、テキスト ボックスの値が変更されたときに [`oninput`](https://developer.mozilla.org/docs/Web/API/GlobalEventHandlers/oninput) が発生します。

`Page/BindEvent.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/data-binding/BindEvent.razor?highlight=4)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/data-binding/BindEvent.razor?highlight=4)]

::: moniker-end

Razor 属性バインディングでは大文字と小文字が区別されます。

* `@bind` および `@bind:event` が有効です。
* `@Bind`/`@Bind:Event` (大文字の `B` と `E`) または `@BIND`/`@BIND:EVENT` (すべて大文字) は **無効** です。

## <a name="unparsable-values"></a>解析不可能値

ユーザーがデータバインド要素に解析できない値を指定すると、バインド イベントがトリガーされたときに、解析できない値は自動的に前の値に戻されます。

次のコンポーネントについて考えてみます。ここで、`<input>` 要素は、初期値が `123` である `int` 型にバインドされています。

`Pages/UnparsableValues.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/data-binding/UnparsableValues.razor?highlight=4,12)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/data-binding/UnparsableValues.razor?highlight=4,12)]

::: moniker-end

既定では、バインドは要素の `onchange` イベントに適用されます。 ユーザーがテキスト ボックスのエントリの値を `123.45` に更新してフォーカスを変更した場合、`onchange`が呼び出されると、要素の値は `123` に戻ります。 値 `123.45` が拒否されて元の値 `123` が設定された場合、ユーザーはその値が受け入れられなかったことを認識します。

`oninput` イベント (`@bind:event="oninput"`) の場合、解析できない値が入力されたキーストロークの後に、値は元に戻されます。 `int` にバインドされた型の `oninput` イベントを対象とする場合、ユーザーがドット (`.`) 文字を入力することはできません。 ドット (`.`) 文字はすぐに削除されるので、ユーザーは整数のみが許可されるというフィードバックをすぐに受け取ることができます。 `oninput` イベントで値を元に戻すのが理想的ではないシナリオもあります。たとえば、ユーザーが解析できない `<input>` 値をクリアできる必要がある場合などです。 代替手段は次のとおりです。

* `oninput` イベントは使用しません。 既定の `onchange` イベントを使用します。この場合、要素がフォーカスを失うまで無効な値は元に戻されません。
* `int?` や `string` などの null 許容型にバインドし、無効なエントリを処理する[カスタム`get`および`set`アクセサーのロジック](#custom-binding-formats)を提供します。
* <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601> や <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601> などの[フォーム検証コンポーネント](xref:blazor/forms-validation)を使用します。 フォーム検証コンポーネントには、無効な入力を管理するためのサポートが組み込まれています。 フォーム検証コンポーネントを使うと、次のことができます。
  * ユーザーの無効な入力を許可し、関連付けられた <xref:Microsoft.AspNetCore.Components.Forms.EditContext> で検証エラーを受信します。
  * ユーザーが追加の Web フォーム データを入力することを妨げることなく、UI に検証エラーを表示します。

## <a name="format-strings"></a>書式指定文字列

データ バインディングでは、`@bind:format="{FORMAT STRING}"` を使用して単一の <xref:System.DateTime> 書式指定文字列を処理できます。ここで、`{FORMAT STRING}` プレースホルダーは書式指定文字列です。 通貨や数値の形式など、その他の書式指定式は現時点では使用できませんが、今後のリリースで追加される可能性があります。

`Pages/DateBinding.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/data-binding/DateBinding.razor?highlight=6)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/data-binding/DateBinding.razor?highlight=6)]

::: moniker-end

上のコードでは、`<input>` 要素のフィールドの種類 (`type` 属性) は既定で `text` に設定されています。

サポートされるのは、Null 許容の <xref:System.DateTime?displayProperty=fullName> および <xref:System.DateTimeOffset?displayProperty=fullName> だけです。

```csharp
private DateTime? date;
private DateTimeOffset? dateOffset;
```

Blazor には日付の書式を設定するためのサポートが組み込まれているため、`date` フィールド型の形式を指定することは推奨されません。 この推奨事項には反しますが、`date` フィールド型で形式を指定する場合は、`yyyy-MM-dd` 日付形式を使用してバインドしたのみ正しく機能します。

```razor
<input type="date" @bind="startDate" @bind:format="yyyy-MM-dd">
```

## <a name="custom-binding-formats"></a>カスタム バインディング形式

次の `DecimalBinding` コンポーネントで示すように、[C# の `get` および `set` アクセサー](/dotnet/csharp/programming-guide/classes-and-structs/using-properties)を使用して、カスタム バインディング形式の動作を作成できます。 このコンポーネントでは、`string` プロパティ (`DecimalValue`) によって `<input>` 要素に、最大 3 桁の小数点以下を含む正または負の 10 進数を疑似バインドします。

`Pages/DecimalBinding.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/data-binding/DecimalBinding.razor?highlight=7,21-31)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/data-binding/DecimalBinding.razor?highlight=7,21-31)]

::: moniker-end

## <a name="binding-with-component-parameters"></a>コンポーネント パラメーターによるバインディング

一般的なシナリオは、子コンポーネントのプロパティをその親コンポーネントのプロパティにバインドすることです。 このシナリオは、複数のレベルのバインドが同時に発生するため、*チェーン バインド* と呼ばれます。

[コンポーネント パラメーター](xref:blazor/components/index#component-parameters)を使用すると、`@bind-{PROPERTY}` 構文で親コンポーネントのプロパティをバインドすることができます。ここで、`{PROPERTY}` プレースホルダーはバインドするプロパティです。

子コンポーネントには、[`@bind`](xref:mvc/views/razor#bind) 構文を使用してチェーン バインドは実装することはできません。 子コンポーネントから親のプロパティを更新するには、イベント ハンドラーと値を別々に指定する必要があります。

子コンポーネントによるデータバインディングを設定するため、親コンポーネントによって引き続き [`@bind`](xref:mvc/views/razor#bind) 構文が活用されます。

次の `ChildBind` コンポーネントには、`Year` コンポーネント パラメーターと <xref:Microsoft.AspNetCore.Components.EventCallback%601> があります。 慣例により、パラメーターの <xref:Microsoft.AspNetCore.Components.EventCallback%601> には、コンポーネント パラメーター名としてサフィックス "`Changed`" を使用して名前を付ける必要があります。 名前付け構文は `{PARAMETER NAME}Changed` です。ここで、`{PARAMETER NAME}` プレースホルダーはパラメーター名です。 次の例では、<xref:Microsoft.AspNetCore.Components.EventCallback%601> に `YearChanged` という名前が付けられています。

<xref:Microsoft.AspNetCore.Components.EventCallback.InvokeAsync%2A?displayProperty=nameWithType> によって、バインディングに関連付けられているデリゲートが指定の引数で呼び出され、変更されたプロパティのイベント通知が送信されます。

`Shared/ChildBind.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/data-binding/ChildBind.razor?highlight=14-15,17-18,22)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/data-binding/ChildBind.razor?highlight=14-15,17-18,22)]

::: moniker-end

イベントと <xref:Microsoft.AspNetCore.Components.EventCallback%601> の詳細については、記事「<xref:blazor/components/event-handling#eventcallback>」の「*EventCallback*」セクションを参照してください。

次の `Parent` コンポーネントでは、`year` フィールドが子コンポーネントの `Year` パラメーターにバインドされています。 `Year` パラメーターには、`Year` パラメーターの型と一致するコンパニオン `YearChanged` イベントがあるため、バインド可能です。

`Pages/Parent.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/data-binding/Parent1.razor?highlight=9)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/data-binding/Parent1.razor?highlight=9)]

::: moniker-end

慣例により、ハンドラーに割り当てられた `@bind-{PROPERTY}:event` 属性を含めることで、プロパティを対応するイベント ハンドラーにバインドできます。ここで、`{PROPERTY}` プレースホルダーはプロパティです。 `<ChildBind @bind-Year="year" />` は次のように書く場合と同じです。

```razor
<ChildBind @bind-Year="year" @bind-Year:event="YearChanged" />
```

もっと洗練された実世界の例では、次の `Password` コンポーネントにより、次のことが行われます。

* `password` フィールドに `<input>` 要素の値を設定します。
* 子の `password` フィールドの現在の値を引数として渡す [`EventCallback`](xref:blazor/components/event-handling#eventcallback) を使用して、`Password` プロパティの変更を親コンポーネントに公開します。
* `onclick` イベントを使用して、`ToggleShowPassword` メソッドをトリガーします。 詳細については、「<xref:blazor/components/event-handling>」を参照してください。

`Shared/PasswordEntry.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/data-binding/PasswordEntry.razor?highlight=7-10,13,23-24,26-27,36-39)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/data-binding/PasswordEntry.razor?highlight=7-10,13,23-24,26-27,36-39)]

::: moniker-end

`PasswordEntry` コンポーネントは、次の `PasswordBinding` コンポーネントの例のように、別のコンポーネントで使用されます。

`Pages/PasswordBinding.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/data-binding/PasswordBinding.razor?highlight=5)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/data-binding/PasswordBinding.razor?highlight=5)]

::: moniker-end

バインドのデリゲートを呼び出すメソッドで、チェックを実行またはエラーをトラップします。 次の改訂された `PasswordEntry` コンポーネントでは、パスワードの値にスペースが使用されている場合、すぐにユーザーにフィードバックが返されます。

`Shared/PasswordEntry.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/data-binding/PasswordEntry2.razor?highlight=35-46)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/data-binding/PasswordEntry2.razor?highlight=35-46)]

::: moniker-end

## <a name="bind-across-more-than-two-components"></a>3 つ以上のコンポーネント間でバインドする

入れ子になった任意の数のコンポーネントを介してパラメーターをバインドできますが、次のような一方向のデータ フローを考慮する必要があります。

* 変更通知は "*階層をフローアップ*" します。
* 新しいパラメーター値は "*階層をフローダウン*" します。

次の例に示すように、一般的な推奨される方法は、基になるデータを親コンポーネントに格納のみすることであり、これにより、更新する必要がある状態に関する混乱を避けることができます。

`Pages/Parent.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/data-binding/Parent2.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/data-binding/Parent2.razor)]

::: moniker-end

`Shared/NestedChild.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/data-binding/NestedChild.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/data-binding/NestedChild.razor)]

::: moniker-end

`Shared/NestedGrandchild.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/data-binding/NestedGrandchild.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/data-binding/NestedGrandchild.razor)]

::: moniker-end

メモリ内で、および必ずしも入れ子にはなっていないコンポーネント間でデータを共有するのに適した別の方法については、「<xref:blazor/state-management>」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:blazor/forms-validation>
* [フォーム内のラジオ ボタンへのバインド](xref:blazor/forms-validation#radio-buttons)
* [フォーム内の C# オブジェクトの `null` 値への `<select>` 要素オプションのバインド](xref:blazor/forms-validation#binding-select-element-options-to-c-object-null-values)
* [ASP.NET Core Blazor のイベント処理: `EventCallback` セクション](xref:blazor/components/event-handling#eventcallback)
