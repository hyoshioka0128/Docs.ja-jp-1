---
title: ASP.NET Core Blazor で JavaScript 関数から .NET メソッドを呼び出す
author: guardrex
description: Blazor アプリで JavaScript 関数から .NET メソッドを呼び出す方法について学習します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc, devx-track-js
ms.date: 08/12/2020
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
uid: blazor/call-dotnet-from-javascript
ms.openlocfilehash: 45ddcc9e006df2c5e86a7859efc76882b269a496
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280398"
---
# <a name="call-net-methods-from-javascript-functions-in-aspnet-core-blazor"></a>ASP.NET Core Blazor で JavaScript 関数から .NET メソッドを呼び出す

Blazor アプリでは、.NET メソッドから JavaScript 関数を呼び出すことも、JavaScript 関数から .NET メソッドを呼び出すこともできます。 これらのシナリオは、"*JavaScript 相互運用*" ("*JS 相互運用*") と呼ばれます。

この記事では、JavaScript から .NET メソッドを呼び出す方法について説明します。 .NET から JavaScript 関数を呼び出す方法については、「<xref:blazor/call-javascript-from-dotnet>」を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

> [!NOTE]
> `wwwroot/index.html` ファイル (Blazor WebAssembly) または `Pages/_Host.cshtml` ファイル (Blazor Server) では、終了タグ `</body>` の前に JS ファイル (`<script>` タグ) を追加します。 JS 相互運用メソッドを含む JS ファイルは、Blazor フレームワークの JS ファイルよりも先に含めるようにします。

## <a name="static-net-method-call"></a>静的 .NET メソッドの呼び出し

JavaScript から静的 .NET メソッドを呼び出すには、`DotNet.invokeMethod` 関数または `DotNet.invokeMethodAsync` 関数を使用します。 呼び出す静的メソッドの識別子、関数を含むアセンブリの名前、任意の引数を渡します。 Blazor Server のシナリオをサポートするには、非同期バージョンを使用することをお勧めします。 .NET メソッドはパブリックかつ静的であり、[`[JSInvokable]` 属性](xref:Microsoft.JSInterop.JSInvokableAttribute)を持つ必要があります。 オープン ジェネリック メソッドを呼び出すことは、現在サポートされていません。

サンプル アプリには、`int` 配列を返す C# メソッドが含まれています。 [`[JSInvokable]` 属性](xref:Microsoft.JSInterop.JSInvokableAttribute)がメソッドに適用されます。

`Pages/JsInterop.razor`:

```razor
<button type="button" class="btn btn-primary"
        onclick="exampleJsFunctions.returnArrayAsyncJs()">
    Trigger .NET static method ReturnArrayAsync
</button>

@code {
    [JSInvokable]
    public static Task<int[]> ReturnArrayAsync()
    {
        return Task.FromResult(new int[] { 1, 2, 3 });
    }
}
```

クライアントに提供される JavaScript は、C# .NET メソッドを呼び出します。

`wwwroot/exampleJsInterop.js`:

[!code-javascript[](./common/samples/5.x/BlazorWebAssemblySample/wwwroot/exampleJsInterop.js?highlight=8-14)]

**`Trigger .NET static method ReturnArrayAsync`** ボタンが選択されている場合は、ブラウザーの Web 開発者ツールでコンソール出力を確認します。

コンソール出力は、次のようになります。

```console
Array(4) [ 1, 2, 3, 4 ]
```

4 番目の配列値は、`ReturnArrayAsync` によって返される配列 (`data.push(4);`) にプッシュされます。

既定では、メソッド識別子はメソッド名ですが、[`[JSInvokable]` 属性](xref:Microsoft.JSInterop.JSInvokableAttribute)コンストラクターを使用して別の識別子を指定することもできます。

```csharp
@code {
    [JSInvokable("DifferentMethodName")]
    public static Task<int[]> ReturnArrayAsync()
    {
        return Task.FromResult(new int[] { 1, 2, 3 });
    }
}
```

クライアント側の JavaScript ファイル

```javascript
returnArrayAsyncJs: function () {
  DotNet.invokeMethodAsync('{APP ASSEMBLY}', 'DifferentMethodName')
    .then(data => {
      data.push(4);
      console.log(data);
    });
}
```

プレースホルダー `{APP ASSEMBLY}` は、アプリのアプリ アセンブリ名です (例: `BlazorSample`)。

## <a name="instance-method-call"></a>インスタンス メソッドの呼び出し

JavaScript から .NET インスタンス メソッドを呼び出すこともできます。 JavaScript から .NET インスタンス メソッドを呼び出すには

* 参照渡しで .NET インスタンスを JavaScript に渡します。
  * 静的呼び出しを <xref:Microsoft.JSInterop.DotNetObjectReference.Create%2A?displayProperty=nameWithType> にします。
  * インスタンスを <xref:Microsoft.JSInterop.DotNetObjectReference> インスタンスにラップし、<xref:Microsoft.JSInterop.DotNetObjectReference> インスタンスで <xref:Microsoft.JSInterop.DotNetObjectReference.Create%2A> を呼び出します。 <xref:Microsoft.JSInterop.DotNetObjectReference> オブジェクトを破棄します (このセクションの後半で例を示します)。
* `invokeMethod` 関数または `invokeMethodAsync` 関数を使用して、インスタンスで .NET インスタンス メソッドを呼び出します。 .NET インスタンスは、JavaScript から他の .NET メソッドを呼び出すときに引数として渡すこともできます。

> [!NOTE]
> サンプル アプリでは、メッセージがクライアント側のコンソールにログ出力されます。 サンプル アプリで示される以下の例については、ブラウザーの開発者ツールでブラウザーのコンソール出力を確認してください。

**`Trigger .NET instance method HelloHelper.SayHello`** ボタンを選択すると、`ExampleJsInterop.CallHelloHelperSayHello` が呼び出され、メソッドに名前 `Blazor` が渡されます。

`Pages/JsInterop.razor`:

```razor
<button type="button" class="btn btn-primary" @onclick="TriggerNetInstanceMethod">
    Trigger .NET instance method HelloHelper.SayHello
</button>

@code {
    public async Task TriggerNetInstanceMethod()
    {
        var exampleJsInterop = new ExampleJsInterop(JS);
        await exampleJsInterop.CallHelloHelperSayHello("Blazor");
    }
}
```

`CallHelloHelperSayHello` では、`HelloHelper` の新しいインスタンスを使用して JavaScript 関数 `sayHello` を呼び出します。

`JsInteropClasses/ExampleJsInterop.cs`:

[!code-csharp[](./common/samples/5.x/BlazorWebAssemblySample/JsInteropClasses/ExampleJsInterop.cs?name=snippet1&highlight=11-18)]

`wwwroot/exampleJsInterop.js`:

[!code-javascript[](./common/samples/5.x/BlazorWebAssemblySample/wwwroot/exampleJsInterop.js?highlight=15-18)]

名前は `HelloHelper` のコンストラクターに渡されます。これにより、`HelloHelper.Name` プロパティが設定されます。 JavaScript 関数 `sayHello` が実行されると、`HelloHelper.SayHello` によって `Hello, {Name}!` メッセージが返されます。これは、JavaScript 関数によってコンソールに書き込まれます。

`JsInteropClasses/HelloHelper.cs`:

[!code-csharp[](./common/samples/5.x/BlazorWebAssemblySample/JsInteropClasses/HelloHelper.cs?name=snippet1&highlight=5,10-11)]

ブラウザーの Web 開発者ツールでのコンソール出力

```console
Hello, Blazor!
```

メモリ リークを回避し、<xref:Microsoft.JSInterop.DotNetObjectReference> を作成するコンポーネントでガベージ コレクションを許可するには、次のいずれかの方法を採用します。

* <xref:Microsoft.JSInterop.DotNetObjectReference> インスタンスを作成したクラスのオブジェクトを破棄します。

  ```csharp
  public class ExampleJsInterop : IDisposable
  {
      private readonly IJSRuntime js;
      private DotNetObjectReference<HelloHelper> objRef;

      public ExampleJsInterop(IJSRuntime js)
      {
          this.js = js;
      }

      public ValueTask<string> CallHelloHelperSayHello(string name)
      {
          objRef = DotNetObjectReference.Create(new HelloHelper(name));

          return js.InvokeAsync<string>(
              "exampleJsFunctions.sayHello",
              objRef);
      }

      public void Dispose()
      {
          objRef?.Dispose();
      }
  }
  ```

  `ExampleJsInterop` クラスに示されている上記のパターンは、コンポーネントに実装することもできます。

  ```razor
  @page "/JSInteropComponent"
  @using {APP ASSEMBLY}.JsInteropClasses
  @implements IDisposable
  @inject IJSRuntime JS

  <h1>JavaScript Interop</h1>

  <button type="button" class="btn btn-primary" @onclick="TriggerNetInstanceMethod">
      Trigger .NET instance method HelloHelper.SayHello
  </button>

  @code {
      private DotNetObjectReference<HelloHelper> objRef;

      public async Task TriggerNetInstanceMethod()
      {
          objRef = DotNetObjectReference.Create(new HelloHelper("Blazor"));

          await JS.InvokeAsync<string>(
              "exampleJsFunctions.sayHello",
              objRef);
      }

      public void Dispose()
      {
          objRef?.Dispose();
      }
  }
  ```
  
  プレースホルダー `{APP ASSEMBLY}` は、アプリのアプリ アセンブリ名です (例: `BlazorSample`)。

* コンポーネントまたはクラスによって <xref:Microsoft.JSInterop.DotNetObjectReference> が破棄されない場合は、`.dispose()` を呼び出すことによって、クライアント上のオブジェクトを破棄します。

  ```javascript
  window.myFunction = (dotnetHelper) => {
    dotnetHelper.invokeMethodAsync('{APP ASSEMBLY}', 'MyMethod');
    dotnetHelper.dispose();
  }
  ```

## <a name="component-instance-method-call"></a>コンポーネント インスタンス メソッドの呼び出し

コンポーネントの .NET メソッドを呼び出すには、次の手順を行います。

* コンポーネントに対して静的メソッド呼び出しを行うには、`invokeMethod` または `invokeMethodAsync` 関数を使用します。
* コンポーネントの静的メソッドにより、そのインスタンス メソッドへの呼び出しが、呼び出された <xref:System.Action> としてラップされます。

> [!NOTE]
> 複数のユーザーが同じコンポーネントを同時に使用している可能性がある Blazor Server アプリの場合、ヘルパー クラスを使用してインスタンス メソッドを呼び出します。
>
> 詳細については、「[コンポーネント インスタンス メソッド ヘルパー クラス](#component-instance-method-helper-class)」セクションを参照してください。

クライアント側の JavaScript:

```javascript
function updateMessageCallerJS() {
  DotNet.invokeMethodAsync('{APP ASSEMBLY}', 'UpdateMessageCaller');
}
```

プレースホルダー `{APP ASSEMBLY}` は、アプリのアプリ アセンブリ名です (例: `BlazorSample`)。

`Pages/JSInteropComponent.razor`:

```razor
@page "/JSInteropComponent"

<p>
    Message: @message
</p>

<p>
    <button onclick="updateMessageCallerJS()">Call JS Method</button>
</p>

@code {
    private static Action action;
    private string message = "Select the button.";

    protected override void OnInitialized()
    {
        action = UpdateMessage;
    }

    private void UpdateMessage()
    {
        message = "UpdateMessage Called!";
        StateHasChanged();
    }

    [JSInvokable]
    public static void UpdateMessageCaller()
    {
        action.Invoke();
    }
}
```

インスタンス メソッドに引数を渡すには:

* JS メソッドの呼び出しにパラメーターを追加します。 次の例では、名前がメソッドに渡されます。 必要に応じて、追加のパラメーターを一覧に追加できます。

  ```javascript
  function updateMessageCallerJS(name) {
    DotNet.invokeMethodAsync('{APP ASSEMBLY}', 'UpdateMessageCaller', name);
  }
  ```
  
  プレースホルダー `{APP ASSEMBLY}` は、アプリのアプリ アセンブリ名です (例: `BlazorSample`)。

* パラメーターの <xref:System.Action> に適切な型を指定します。 C# メソッドにパラメーター一覧を指定します。 <xref:System.Action> (`UpdateMessage`) をパラメーター (`action.Invoke(name)`) を使用して呼び出します。

  `Pages/JSInteropComponent.razor`:

  ```razor
  @page "/JSInteropComponent"

  <p>
      Message: @message
  </p>

  <p>
      <button onclick="updateMessageCallerJS('Sarah Jane')">
          Call JS Method
      </button>
  </p>

  @code {
      private static Action<string> action;
      private string message = "Select the button.";

      protected override void OnInitialized()
      {
          action = UpdateMessage;
      }

      private void UpdateMessage(string name)
      {
          message = $"{name}, UpdateMessage Called!";
          StateHasChanged();
      }

      [JSInvokable]
      public static void UpdateMessageCaller(string name)
      {
          action.Invoke(name);
      }
  }
  ```

  **[Call JS Method]\(JS メソッドの呼び出し\)** ボタンを選択したときに `message` を出力します。

  ```
  Sarah Jane, UpdateMessage Called!
  ```

## <a name="component-instance-method-helper-class"></a>コンポーネント インスタンス メソッド ヘルパー クラス

ヘルパー クラスは、<xref:System.Action> としてインスタンス メソッドを呼び出すために使用されます。 ヘルパー クラスは、次の場合に役立ちます。

* 同じ種類の複数のコンポーネントが同じページにレンダリングされる。
* Blazor Server アプリが使用され、複数のユーザーがコンポーネントを同時に使用している可能性があります。

次に例を示します。

* `JSInteropExample` コンポーネントには、複数の `ListItem` コンポーネントが含まれています。
* 各 `ListItem` コンポーネントは、メッセージとボタンで構成されます。
* `ListItem` コンポーネント ボタンが選択されると、その `ListItem` の `UpdateMessage` メソッドによってリスト項目のテキストが変更され、ボタンが非表示になります。

`MessageUpdateInvokeHelper.cs`:

```csharp
using System;
using Microsoft.JSInterop;

public class MessageUpdateInvokeHelper
{
    private Action action;

    public MessageUpdateInvokeHelper(Action action)
    {
        this.action = action;
    }

    [JSInvokable("{APP ASSEMBLY}")]
    public void UpdateMessageCaller()
    {
        action.Invoke();
    }
}
```

プレースホルダー `{APP ASSEMBLY}` は、アプリのアプリ アセンブリ名です (例: `BlazorSample`)。

クライアント側の JavaScript:

```javascript
window.updateMessageCallerJS = (dotnetHelper) => {
    dotnetHelper.invokeMethodAsync('{APP ASSEMBLY}', 'UpdateMessageCaller');
    dotnetHelper.dispose();
}
```

プレースホルダー `{APP ASSEMBLY}` は、アプリのアプリ アセンブリ名です (例: `BlazorSample`)。

`Shared/ListItem.razor`:

```razor
@inject IJSRuntime JS

<li>
    @message
    <button @onclick="InteropCall" style="display:@display">InteropCall</button>
</li>

@code {
    private string message = "Select one of these list item buttons.";
    private string display = "inline-block";
    private MessageUpdateInvokeHelper messageUpdateInvokeHelper;

    protected override void OnInitialized()
    {
        messageUpdateInvokeHelper = new MessageUpdateInvokeHelper(UpdateMessage);
    }

    protected async Task InteropCall()
    {
        await JS.InvokeVoidAsync("updateMessageCallerJS",
            DotNetObjectReference.Create(messageUpdateInvokeHelper));
    }

    private void UpdateMessage()
    {
        message = "UpdateMessage Called!";
        display = "none";
        StateHasChanged();
    }
}
```

`Pages/JSInteropExample.razor`:

```razor
@page "/JSInteropExample"

<h1>List of components</h1>

<ul>
    <ListItem />
    <ListItem />
    <ListItem />
    <ListItem />
</ul>
```

[!INCLUDE[](~/blazor/includes/share-interop-code.md)]

## <a name="avoid-circular-object-references"></a>循環オブジェクト参照の回避

循環参照を含むオブジェクトは、次のいずれに対しても、クライアントでシリアル化することはできません。

* .NET メソッドの呼び出し。
* 戻り値の型に循環参照がある場合の、C# からの JavaScript メソッドの呼び出し。

詳細については、次のイシューを参照してください。

* [Circular references are not supported, take two (dotnet/aspnetcore #20525)](https://github.com/dotnet/aspnetcore/issues/20525) (循環参照はサポートされていません、テイク 2 (dotnet/aspnetcore #20525))
* [Proposal: Add mechanism to handle circular references when serializing (dotnet/runtime #30820)](https://github.com/dotnet/runtime/issues/30820) (提案: シリアル化するときに循環参照を処理するメカニズムを追加する (dotnet/runtime #30820))

## <a name="size-limits-on-js-interop-calls"></a>JS 相互運用呼び出しのサイズ制限

Blazor WebAssembly では、フレームワークによって JS 相互運用の入力と出力のサイズが制限されることはありません。

Blazor Server では、ハブ メソッドで許可される SignalR 受信メッセージの最大サイズによって、JS 相互運用呼び出しのサイズが制限されます。これは、<xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize?displayProperty=nameWithType> によって適用されます (既定値: 32 KB)。 JS から .NET への SignalR メッセージが <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize> より大きい場合は、エラーがスローされます。 このフレームワークでは、ハブからクライアントへの SignalR メッセージのサイズが制限されることはありません。

SignalR のログが[Debug](xref:Microsoft.Extensions.Logging.LogLevel) または [Trace](xref:Microsoft.Extensions.Logging.LogLevel) に設定されていない場合、メッセージ サイズのエラーはブラウザーの開発者ツール コンソールにのみ表示されます。

> エラー :次のエラーで接続が切断されました。"エラー: サーバーが終了時にエラーを返しました:接続はエラーで終了しました。"

[SignalR サーバー側のログ ](xref:signalr/diagnostics#server-side-logging) が [Debug](xref:Microsoft.Extensions.Logging.LogLevel) または [Trace](xref:Microsoft.Extensions.Logging.LogLevel) に設定されている場合、サーバー側のログには、メッセージ サイズ エラーの <xref:System.IO.InvalidDataException> が表示されます。

`appsettings.Development.json`:

```json
{
  "DetailedErrors": true,
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information",
      "Microsoft.AspNetCore.SignalR": "Debug"
    }
  }
}
```

> System.IO.InvalidDataException:メッセージの最大サイズ 32,768 B を超えました。 メッセージのサイズは、AddHubOptions で構成できます。

制限値を増やすには、`Startup.ConfigureServices` で <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize> を設定します。 次の例では、受信メッセージの最大サイズを 64 KB (64 * 1024) に設定します。

```csharp
services.AddServerSideBlazor()
   .AddHubOptions(options => options.MaximumReceiveMessageSize = 64 * 1024);
```

SignalR 受信メッセージ サイズの制限値を増やすと、より多くのサーバー リソースが必要になり、悪意のあるユーザーからのより大きなリスクにサーバーがさらされます。 また、大量のコンテンツを文字列またはバイト配列としてメモリに読み取ると、ガベージ コレクターがうまく機能しない割り当てが発生する可能性もあり、その結果、パフォーマンスがさらに低下します。

大きなペイロードを読み取るための 1 つの選択肢は、小さいチャンクでコンテンツを送信し、ペイロードを <xref:System.IO.Stream> として処理することです。 これは、大量の JSON ペイロードを読み取る場合、またはデータを JavaScript で生バイトとして利用できる場合に使用できます。 Blazor Server で大規模なバイナリ ペイロードを送信する、`InputFile` コンポーネントに似た手法を使用する方法の例については、[Binary Submit サンプル アプリ](https://github.com/aspnet/samples/tree/master/samples/aspnetcore/blazor/BinarySubmit)を参照してください。

JavaScript と Blazor の間で大量のデータを転送するコードを開発するときは、次のガイダンスを考慮してください。

* データをより小さな部分にスライスし、すべてのデータがサーバーによって受信されるまでデータ セグメントを順番に送信します。
* JavaScript および C# コードで大きなオブジェクトを割り当てないでください。
* データを送受信するときに、メイン UI スレッドを長時間ブロックしないでください。
* プロセスの完了時またはキャンセル時に、消費していたメモリを解放します。
* セキュリティ上の理由から、次の追加要件を適用します。
  * 渡すことのできるファイルまたはデータの最大サイズを宣言します。
  * クライアントからサーバーへの最小アップロード レートを宣言します。
* データがサーバーによって受信されたら、データは:
  * すべてのセグメントが収集されるまで、一時的にメモリ バッファーに格納できます。
  * 直ちに消費できます。 たとえば、データは、データベースに直ちに格納することも、セグメントを受信するたびにディスクに書き込むこともできます。

## <a name="js-modules"></a>JS モジュール

JS の分離では、JS 相互運用は、ブラウザーで [EcmaScript モジュール (ESM)](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules) ([ECMAScript 仕様](https://tc39.es/ecma262/#sec-modules)) が既定でサポートされている場合に機能します。

## <a name="additional-resources"></a>その他の技術情報

* <xref:blazor/call-javascript-from-dotnet>
* [`InteropComponent.razor` の例 (dotnet/AspNetCore GitHub リポジトリ、3.1 リリース ブランチ)](https://github.com/dotnet/AspNetCore/blob/release/3.1/src/Components/test/testassets/BasicTestApp/InteropComponent.razor)
