---
title: ASP.NET Core Blazor のイベント処理
author: guardrex
description: イベント引数の型、イベントのコールバック、既定のブラウザー イベントの管理など、Blazor のイベント処理機能について説明します。
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
uid: blazor/components/event-handling
ms.openlocfilehash: c04ff3c2a8de6e8c9d0fa0653f09da2984dc528e
ms.sourcegitcommit: 4bbc69f51c59bed1a96aa46f9f5dca2f2a2634cb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/25/2021
ms.locfileid: "105555046"
---
# <a name="aspnet-core-blazor-event-handling"></a>ASP.NET Core Blazor のイベント処理

次の [`@on{DOM EVENT}="{DELEGATE}"`](xref:mvc/views/razor#onevent) Razor 構文を使用して、Razor コンポーネント マークアップでデリゲート イベント ハンドラーを指定します。

* `{DOM EVENT}` プレースホルダーは、[ドキュメント オブジェクト モデル (DOM) イベント](https://developer.mozilla.org/docs/Web/Events)です (たとえば、`click`)。
* `{DELEGATE}` プレースホルダーは、C# デリゲート イベント ハンドラーです。

イベント処理の場合:

* <xref:System.Threading.Tasks.Task> を返す非同期デリゲート イベント ハンドラーがサポートされています。
* デリゲート イベント ハンドラーによって UI レンダリングが自動的にトリガーされるため、[StateHasChanged](xref:blazor/components/lifecycle#state-changes-statehaschanged) を手動で呼び出す必要はありません。
* 例外がログされます。

コード例を次に示します。

* UI 内でボタンが選択されたときに `UpdateHeading` メソッドを呼び出します。
* UI 内でチェック ボックスが変更されたときに `CheckChanged` メソッドを呼び出します。

`Pages/EventHandlerExample1.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/event-handling/EventHandlerExample1.razor?highlight=10,17,27-30,32-35)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/event-handling/EventHandlerExample1.razor?highlight=10,17,27-30,32-35)]

::: moniker-end

`UpdateHeading` は、以下の例では次のようになります。

* ボタンが選択されると、非同期に呼び出されます。
* 2 秒間待機してから、見出しを更新します。

`Pages/EventHandlerExample2.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/event-handling/EventHandlerExample2.razor?highlight=10,19-24)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/event-handling/EventHandlerExample2.razor?highlight=10,19-24)]

::: moniker-end

## <a name="event-arguments"></a>イベント引数

::: moniker range=">= aspnetcore-6.0"

### <a name="built-in-event-arguments"></a>組み込みのイベント引数

::: moniker-end

イベント引数の型をサポートするイベントの場合、イベント メソッド定義でイベント パラメーターを指定する必要があるのは、イベントの型がメソッドで使用されている場合のみです。 次の例では、`ReportPointerLocation` メソッドで <xref:Microsoft.AspNetCore.Components.Web.MouseEventArgs> が使用されています。これにより、ユーザーが UI 内でボタンを選択すると、マウスの座標を報告するメッセージ テキストが設定されます。

`Pages/EventHandlerExample3.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/event-handling/EventHandlerExample3.razor?highlight=17-20)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/event-handling/EventHandlerExample3.razor?highlight=17-20)]

::: moniker-end

サポートされている <xref:System.EventArgs> を次の表に示します。

::: moniker range=">= aspnetcore-5.0"

| event            | クラス  | [ドキュメント オブジェクト モデル (DOM)](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction) イベントと注記 |
| ---------------- | ------ | --- |
| クリップボードのトピック        | <xref:Microsoft.AspNetCore.Components.Web.ClipboardEventArgs> | `oncut`, `oncopy`, `onpaste` |
| ドラッグ             | <xref:Microsoft.AspNetCore.Components.Web.DragEventArgs> | `ondrag`, `ondragstart`, `ondragenter`, `ondragleave`, `ondragover`, `ondrop`, `ondragend`<br><br><xref:Microsoft.AspNetCore.Components.Web.DataTransfer> および <xref:Microsoft.AspNetCore.Components.Web.DataTransferItem> では、ドラッグされた項目データを保持します。<br><br>[HTML ドラッグ アンド ドロップ API](https://developer.mozilla.org/docs/Web/API/HTML_Drag_and_Drop_API) と共に [JS 相互運用](xref:blazor/call-javascript-from-dotnet)を使用し、Blazor アプリにドラッグ アンド ドロップを実装します。 |
| Error            | <xref:Microsoft.AspNetCore.Components.Web.ErrorEventArgs> | `onerror` |
| event            | <xref:System.EventArgs> | *全般*<br>`onactivate`, `onbeforeactivate`, `onbeforedeactivate`, `ondeactivate`, `onfullscreenchange`, `onfullscreenerror`, `onloadeddata`, `onloadedmetadata`, `onpointerlockchange`, `onpointerlockerror`, `onreadystatechange`, `onscroll`<br><br>*クリップボード*<br>`onbeforecut`, `onbeforecopy`, `onbeforepaste`<br><br>*入力*<br>`oninvalid`, `onreset`, `onselect`, `onselectionchange`, `onselectstart`, `onsubmit`<br><br>*メディア*<br>`oncanplay`, `oncanplaythrough`, `oncuechange`, `ondurationchange`, `onemptied`, `onended`, `onpause`, `onplay`, `onplaying`, `onratechange`, `onseeked`, `onseeking`, `onstalled`, `onstop`, `onsuspend`, `ontimeupdate`, `ontoggle`, `onvolumechange`, `onwaiting`<br><br><xref:Microsoft.AspNetCore.Components.Web.EventHandlers> は、イベント名とイベント引数の型の間のマッピングを構成する属性を保持します。 |
| フォーカス            | <xref:Microsoft.AspNetCore.Components.Web.FocusEventArgs> | `onfocus`, `onblur`, `onfocusin`, `onfocusout`<br><br>`relatedTarget` のサポートは含まれません。 |
| 入力            | <xref:Microsoft.AspNetCore.Components.ChangeEventArgs> | `onchange`, `oninput` |
| キーボード         | <xref:Microsoft.AspNetCore.Components.Web.KeyboardEventArgs> | `onkeydown`, `onkeypress`, `onkeyup` |
| マウス            | <xref:Microsoft.AspNetCore.Components.Web.MouseEventArgs> | `onclick`, `oncontextmenu`, `ondblclick`, `onmousedown`, `onmouseup`, `onmouseover`, `onmousemove`, `onmouseout` |
| マウス ポインター    | <xref:Microsoft.AspNetCore.Components.Web.PointerEventArgs> | `onpointerdown`, `onpointerup`, `onpointercancel`, `onpointermove`, `onpointerover`, `onpointerout`, `onpointerenter`, `onpointerleave`, `ongotpointercapture`, `onlostpointercapture` |
| マウス ホイール      | <xref:Microsoft.AspNetCore.Components.Web.WheelEventArgs> | `onwheel`, `onmousewheel` |
| 進行状況         | <xref:Microsoft.AspNetCore.Components.Web.ProgressEventArgs> | `onabort`, `onload`, `onloadend`, `onloadstart`, `onprogress`, `ontimeout` |
| タッチ            | <xref:Microsoft.AspNetCore.Components.Web.TouchEventArgs> | `ontouchstart`, `ontouchend`, `ontouchmove`, `ontouchenter`, `ontouchleave`, `ontouchcancel`<br><br><xref:Microsoft.AspNetCore.Components.Web.TouchPoint> は、タッチを検知するデバイス上の単一接触点を表します。 |

::: moniker-end

::: moniker range="< aspnetcore-5.0"

| event            | クラス | [ドキュメント オブジェクト モデル (DOM)](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction) イベントと注記 |
| ---------------- | ----- | --- |
| クリップボードのトピック        | <xref:Microsoft.AspNetCore.Components.Web.ClipboardEventArgs> | `oncut`, `oncopy`, `onpaste` |
| ドラッグ             | <xref:Microsoft.AspNetCore.Components.Web.DragEventArgs> | `ondrag`, `ondragstart`, `ondragenter`, `ondragleave`, `ondragover`, `ondrop`, `ondragend`<br><br><xref:Microsoft.AspNetCore.Components.Web.DataTransfer> および <xref:Microsoft.AspNetCore.Components.Web.DataTransferItem> では、ドラッグされた項目データを保持します。<br><br>[HTML ドラッグ アンド ドロップ API](https://developer.mozilla.org/docs/Web/API/HTML_Drag_and_Drop_API) と共に [JS 相互運用](xref:blazor/call-javascript-from-dotnet)を使用し、Blazor アプリにドラッグ アンド ドロップを実装します。 |
| Error            | <xref:Microsoft.AspNetCore.Components.Web.ErrorEventArgs> | `onerror` |
| event            | <xref:System.EventArgs> | *全般*<br>`onactivate`, `onbeforeactivate`, `onbeforedeactivate`, `ondeactivate`, `onfullscreenchange`, `onfullscreenerror`, `onloadeddata`, `onloadedmetadata`, `onpointerlockchange`, `onpointerlockerror`, `onreadystatechange`, `onscroll`<br><br>*クリップボード*<br>`onbeforecut`, `onbeforecopy`, `onbeforepaste`<br><br>*入力*<br>`oninvalid`, `onreset`, `onselect`, `onselectionchange`, `onselectstart`, `onsubmit`<br><br>*メディア*<br>`oncanplay`, `oncanplaythrough`, `oncuechange`, `ondurationchange`, `onemptied`, `onended`, `onpause`, `onplay`, `onplaying`, `onratechange`, `onseeked`, `onseeking`, `onstalled`, `onstop`, `onsuspend`, `ontimeupdate`, `onvolumechange`, `onwaiting`<br><br><xref:Microsoft.AspNetCore.Components.Web.EventHandlers> は、イベント名とイベント引数の型の間のマッピングを構成する属性を保持します。 |
| フォーカス            | <xref:Microsoft.AspNetCore.Components.Web.FocusEventArgs> | `onfocus`, `onblur`, `onfocusin`, `onfocusout`<br><br>`relatedTarget` のサポートは含まれません。 |
| 入力            | <xref:Microsoft.AspNetCore.Components.ChangeEventArgs> | `onchange`, `oninput` |
| キーボード         | <xref:Microsoft.AspNetCore.Components.Web.KeyboardEventArgs> | `onkeydown`, `onkeypress`, `onkeyup` |
| マウス            | <xref:Microsoft.AspNetCore.Components.Web.MouseEventArgs> | `onclick`, `oncontextmenu`, `ondblclick`, `onmousedown`, `onmouseup`, `onmouseover`, `onmousemove`, `onmouseout` |
| マウス ポインター    | <xref:Microsoft.AspNetCore.Components.Web.PointerEventArgs> | `onpointerdown`, `onpointerup`, `onpointercancel`, `onpointermove`, `onpointerover`, `onpointerout`, `onpointerenter`, `onpointerleave`, `ongotpointercapture`, `onlostpointercapture` |
| マウス ホイール      | <xref:Microsoft.AspNetCore.Components.Web.WheelEventArgs> | `onwheel`, `onmousewheel` |
| 進行状況         | <xref:Microsoft.AspNetCore.Components.Web.ProgressEventArgs> | `onabort`, `onload`, `onloadend`, `onloadstart`, `onprogress`, `ontimeout` |
| タッチ            | <xref:Microsoft.AspNetCore.Components.Web.TouchEventArgs> | `ontouchstart`, `ontouchend`, `ontouchmove`, `ontouchenter`, `ontouchleave`, `ontouchcancel`<br><br><xref:Microsoft.AspNetCore.Components.Web.TouchPoint> は、タッチを検知するデバイス上の単一接触点を表します。 |

::: moniker-end

詳細については、次のリソースを参照してください。

* ASP.NET Core 参照ソース内の [`EventArgs` クラス (dotnet/aspnetcore `main` ブランチ)](https://github.com/dotnet/aspnetcore/tree/main/src/Components/Web/src/Web)

  [!INCLUDE[](~/blazor/includes/aspnetcore-repo-ref-source-links.md)]

* [MDN Web ドキュメント:GlobalEventHandlers](https://developer.mozilla.org/docs/Web/API/GlobalEventHandlers):各 DOM イベントをサポートする HTML 要素に関する情報が含まれています。

::: moniker range=">= aspnetcore-6.0"

### <a name="custom-event-arguments"></a>カスタム イベント引数

Blazor ではカスタム イベント引数がサポートされています。このため、カスタム イベントを使用して任意のデータを .NET イベント ハンドラーに渡すことができます。

#### <a name="general-configuration"></a>全般構成

カスタム イベント引数を使用したカスタム イベントは、一般に次の手順によって有効にされます。

1. JavaScript では、ソース イベントからカスタム イベント引数オブジェクトをビルドするための関数を定義します。

   ```javascript
   function eventArgsCreator(event) { 
     return {
       customProperty1: 'any value for property 1',
       customProperty2: event.srcElement.value
     };
   }
   ```

1. Blazor `<script>` の直後にある `wwwroot/index.html` (Blazor WebAssembly) または `Pages/_Host.cshtml` (Blazor Server) で前のハンドラーにカスタム イベントを登録します。

   ```html
   <script>
       Blazor.registerCustomEventType('customevent', {
           createEventArgs: eventArgsCreator;
       });
   </script>
   ```

   > [!NOTE]
   > `registerCustomEventType` の呼び出しは、1 つのイベントにつき 1 回だけスクリプト内で実行されます。

1. イベント引数のクラスは次のように定義します。

   ```csharp
   public class CustomEventArgs : EventArgs
   {
       public string CustomProperty1 {get; set;}
       public string CustomProperty2 {get; set;}
   }
   ```

1. カスタム イベントに関する <xref:Microsoft.AspNetCore.Components.EventHandlerAttribute> 属性注釈を追加することによって、イベント引数を使用したカスタム イベントを接続します。 このクラスにメンバーは必要ありません。

   ```csharp
   [EventHandler("oncustomevent", typeof(CustomEventArgs), enableStopPropagation: true, enablePreventDefault: true)]
   static class EventHandlers
   {
   }
   ```

1. 1 つまたは複数の HTML 要素に対してイベント ハンドラーを登録します。 Javascript の形式でデリゲート ハンドラー メソッドに渡されたデータにアクセスします。

   ```razor
   <button @oncustomevent="HandleCustomEvent">Handle</button>

   @code
   {
       void HandleCustomEvent(CustomEventArgs eventArgs)
       {
           // eventArgs.CustomProperty1
           // eventArgs.CustomProperty2
       }
   }
   ```

DOM 上でカスタム イベントが発生するたびに、Javascript から渡されたデータを使用してイベント ハンドラーが呼び出されます。

カスタム イベントを発生させようとしている場合は、[`bubbles`](https://developer.mozilla.org/docs/Web/API/Event/bubbles) を有効にするために、その値を `true` に設定する必要があります。 それを行わない場合は、イベントが発生しても、処理用の Blazor ハンドラーが C# カスタム <xref:Microsoft.AspNetCore.Components.EventHandlerAttribute> メソッドに到達することはありません。 詳細については、「[MDN Web Docs: イベントのバブリング](https://developer.mozilla.org/docs/Web/Guide/Events/Creating_and_triggering_events#event_bubbling)」を参照してください。

#### <a name="custom-clipboard-paste-event-example"></a>カスタム クリップボードの貼り付けイベントの例

次の例では、貼り付けの時刻とユーザーが貼り付けたテキストを含む、カスタム クリップボードの貼り付けイベントを受け取ります。

イベントのカスタム名 (`oncustompaste`) と、このイベントのイベント引数を保持する .NET クラス (`CustomPasteEventArgs`) を宣言します。

`CustomEvents.cs`:

```csharp
[EventHandler("oncustompaste", typeof(CustomPasteEventArgs), 
    enableStopPropagation: true, enablePreventDefault: true)]
public static class EventHandlers
{
}

public class CustomPasteEventArgs : EventArgs
{
    public DateTime EventTimestamp { get; set; }
    public string PastedData { get; set; }
}
```

<xref:System.EventArgs> サブクラスにデータを提供する JavaScript コードを追加します。 `wwwroot/index.html` または `Pages/_Host.cshtml` ファイル内で、Blazor スクリプトの直後に次の `<script>` タグとコンテンツを追加します。 次の例では、テキストの貼り付けのみを処理しますが、任意の JavaScript API を使用して、ユーザーによる他の種類のデータ (画像など) の貼り付けを処理することもできます。

Blazor スクリプトの直後の `wwwroot/index.html` (Blazor WebAssembly) または `Pages/_Host.cshtml` (Blazor Server)

```html
<script>
    Blazor.registerCustomEventType('custompaste', {
        browserEventName: 'paste',
        createEventArgs: event => {
            return {
                eventTimestamp: new Date(),
                pastedData: event.clipboardData.getData('text')
            };
        }
    });
</script>
```

上記のコードでは、ネイティブの [`paste`](https://developer.mozilla.org/docs/Web/API/Element/paste_event) イベントが発生したときに、次のことをブラウザーに指示します。

* `custompaste` イベントを発生させる。
* 次に示したカスタムロジックを使用して、イベント引数データを指定する。
  * `eventTimestamp` の場合は、新しい日付を作成します。
  * `pastedData` の場合は、クリップボード データをテキストとして取得します。 詳細については、「[MDN Web Docs: ClipboardEvent.clipboardData](https://developer.mozilla.org/docs/Web/API/ClipboardEvent/clipboardData)」を参照してください。

イベント名の規則は、.NET と JavaScript で異なります。

* .NET では、イベント名の先頭に "`on`" が付きます。
* JavaScript では、イベント名にプレフィックスは付きません。

Razor コンポーネントでは、カスタム ハンドラーを要素にアタッチします。

`Pages/CustomPasteArguments.razor`:

```razor
@page "/custom-paste-arguments"

<label>
    Try pasting into the following text box:
    <input @oncustompaste="HandleCustomPaste" />
</label>

<p>
    @message
</p>

@code {
    private string message;

    private void HandleCustomPaste(CustomPasteEventArgs eventArgs)
    {
        message = $"At {eventArgs.EventTimestamp.ToShortTimeString()}, " +
            $"you pasted: {eventArgs.PastedData}";
    }
}
```

::: moniker-end

## <a name="lambda-expressions"></a>ラムダ式

[ラムダ式](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions) は、デリゲート イベント ハンドラーとしてサポートされています。

`Pages/EventHandlerExample4.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/event-handling/EventHandlerExample4.razor?highlight=6)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/event-handling/EventHandlerExample4.razor?highlight=6)]

::: moniker-end

要素のセットを反復処理するときなど、C# メソッド パラメーターを使用して追加の値に集中すると便利な場合がよくあります。 次の例では、3 つのボタンを作成します。それぞれを押すと、`UpdateHeading` が呼び出され、次のデータが渡されます。

* `e` に対してイベント引数 (<xref:Microsoft.AspNetCore.Components.Web.MouseEventArgs>)。
* `buttonNumber` に対してボタン番号。

`Pages/EventHandlerExample5.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/event-handling/EventHandlerExample5.razor?highlight=10,19)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/event-handling/EventHandlerExample5.razor?highlight=10,19)]

::: moniker-end

> [!NOTE]
> 前の `for` ループの例の `i` などのラムダ式内で、ループ変数を直接使用し **ない** でください。 そうしないと、すべてのラムダ式で同じ変数が使用され、すべてのラムダで同じ値が使用されることになります。 常にローカル変数の変数値をキャプチャしてから使用してください。 前の例の場合:
>
> * ループ変数 `i` は `buttonNumber` に割り当てられます。
> * `buttonNumber` はラムダ式で使用されます。

## <a name="eventcallback"></a>EventCallback

入れ子になったコンポーネントがある一般的なシナリオでは、子コンポーネントのイベントが発生したときに親コンポーネントのメソッドを実行します。 子コンポーネントで発生する `onclick` イベントが、一般的なユース ケースです。 コンポーネント間にわたってイベントを公開するには、<xref:Microsoft.AspNetCore.Components.EventCallback> を使用します。 親コンポーネントでは、コールバック メソッドを子コンポーネントの <xref:Microsoft.AspNetCore.Components.EventCallback> に割り当てることができます。

次の `Child` は、ボタンの `onclick` ハンドラーがどのように、サンプルの `ParentComponent` から <xref:Microsoft.AspNetCore.Components.EventCallback> デリゲートを受け取るように設定されているかを示しています。 <xref:Microsoft.AspNetCore.Components.EventCallback> は `MouseEventArgs` によって型指定されます。これは、周辺機器の `onclick` イベントに適しています。

`Shared/Child.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/event-handling/Child.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/event-handling/Child.razor)]

::: moniker-end

`Parent` コンポーネントでは、子の <xref:Microsoft.AspNetCore.Components.EventCallback%601> (`OnClickCallback`) を `ShowMessage` メソッドに設定しています。

`Pages/Parent.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/event-handling/Parent.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/event-handling/Parent.razor)]

::: moniker-end

`ChildComponent` でボタンが選択されると:

* `Parent` コンポーネントの `ShowMessage` メソッドが呼び出されます。 `message` が更新されて、`Parent` コンポーネントに表示されます。
* コールバックのメソッド (`ShowMessage`) 内に、[`StateHasChanged`](xref:blazor/components/lifecycle#state-changes-statehaschanged) の呼び出しは必要ありません。 <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> は、子イベントが子の中で実行されるイベント ハンドラーでコンポーネントのレンダリングをトリガーするのと同様に、`Parent` コンポーネントを再レンダリングするために自動的に呼び出されます。 詳細については、「<xref:blazor/components/rendering>」を参照してください。

<xref:Microsoft.AspNetCore.Components.EventCallback> と <xref:Microsoft.AspNetCore.Components.EventCallback%601> では非同期デリゲートを使用できます。 <xref:Microsoft.AspNetCore.Components.EventCallback> は弱く型指定されており、`InvokeAsync(Object)` では任意の型の引数を渡すことができます。 <xref:Microsoft.AspNetCore.Components.EventCallback%601> は厳密に型指定されており、`InvokeAsync(T)` では `TValue` に代入可能な `T` 引数を渡す必要があります。

```razor
<ChildComponent 
    OnClickCallback="@(async () => { await Task.Yield(); messageText = "Blaze It!"; })" />
```

<xref:Microsoft.AspNetCore.Components.EventCallback.InvokeAsync%2A> を使用して <xref:Microsoft.AspNetCore.Components.EventCallback> または <xref:Microsoft.AspNetCore.Components.EventCallback%601> を呼び出して、<xref:System.Threading.Tasks.Task> を待機します。

```csharp
await OnClickCallback.InvokeAsync(arg);
```

イベント処理とバインド コンポーネントのパラメーターには、<xref:Microsoft.AspNetCore.Components.EventCallback> と <xref:Microsoft.AspNetCore.Components.EventCallback%601> を使用します。

厳密に型指定された <xref:Microsoft.AspNetCore.Components.EventCallback%601> を <xref:Microsoft.AspNetCore.Components.EventCallback> よりも優先します。 <xref:Microsoft.AspNetCore.Components.EventCallback%601> からは、強化されたエラー フィードバックがコンポーネントのユーザーに提供されます。 他の UI イベント ハンドラーと同様に、このイベント パラメーターの指定は省略可能です。 コールバックに渡される値がない場合は、<xref:Microsoft.AspNetCore.Components.EventCallback> を使用します。

## <a name="prevent-default-actions"></a>既定のアクションを止める

イベントの既定のアクションを防止するには、[`@on{DOM EVENT}:preventDefault`](xref:mvc/views/razor#oneventpreventdefault) を使用します。ここで、`{DOM EVENT}` プレースホルダーは、[ドキュメント オブジェクト モデル (DOM) イベント](https://developer.mozilla.org/docs/Web/Events)です。

入力デバイスでキーが選択され、要素のフォーカスがテキスト ボックス上にあるときは、通常、ブラウザーによってテキスト ボックスにキーの文字が表示されます。 次の例では、`@onkeydown:preventDefault` ディレクティブ属性を指定することで、既定の動作が止められています。 `<input>` 要素にフォーカスがある場合、カウンターはキー シーケンス <kbd>Shift</kbd>+<kbd>+</kbd> が押されるとインクリメントします。 `+` 文字は、`<input>` 要素の値に割り当てられていません。 `keydown` の詳細については、「[`MDN Web Docs: Document: keydown` イベント](https://developer.mozilla.org/docs/Web/API/Document/keydown_event)」を参照してください。

`Pages/EventHandlerExample6.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/event-handling/EventHandlerExample6.razor?highlight=4)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/event-handling/EventHandlerExample6.razor?highlight=4)]

::: moniker-end

値なしで `@on{DOM EVENT}:preventDefault` 属性を指定することは、`@on{DOM EVENT}:preventDefault="true"` と同じことになります。

式は、許可されている属性値でもあります。 次の例では、`shouldPreventDefault` は `true` または `false` のいずれかに設定される `bool` フィールドです。

```razor
<input @onkeydown:preventDefault="shouldPreventDefault" />

...

@code {
    private bool shouldPreventDefault = true;
}
```

## <a name="stop-event-propagation"></a>イベント伝達を停止する

イベントの伝達を停止するには、[`@on{DOM EVENT}:stopPropagation`](xref:mvc/views/razor#oneventstoppropagation) ディレクティブ属性を使用します。ここで、`{DOM EVENT}` プレースホルダーは[ドキュメント オブジェクト モデル (DOM) イベント](https://developer.mozilla.org/docs/Web/Events)です。

次の例では、チェック ボックスをオンにすると、2 番目の子 `<div>` からのクリック イベントが親の `<div>` に伝達されなくなります。 クリック イベントが伝達されると、通常、`OnSelectParentDiv` メソッドが起動されるので、2 番目の子 `<div>` を選択すると、チェック ボックスがオンになっていない限り、親の div メッセージが表示されます。

`Pages/EventHandlerExample7.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/event-handling/EventHandlerExample7.razor?highlight=4,15-16)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/event-handling/EventHandlerExample7.razor?highlight=4,15-16)]

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

## <a name="focus-an-element"></a>要素にフォーカスを合わせる

コード内の要素にフォーカスを合わせるには、[要素参照](xref:blazor/call-javascript-from-dotnet#capture-references-to-elements)で <xref:Microsoft.AspNetCore.Components.ElementReferenceExtensions.FocusAsync%2A> を呼び出します。 次の例では、ボタンを選択して `<input>` 要素にフォーカスを移動します。

`Pages/EventHandlerExample8.razor`:

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/event-handling/EventHandlerExample8.razor?highlight=16)]

::: moniker-end
