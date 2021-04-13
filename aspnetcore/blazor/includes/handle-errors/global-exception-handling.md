---
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
ms.openlocfilehash: 0ce606cc4800bcf5855b70a26272fceb3ad8e944
ms.sourcegitcommit: 4bbc69f51c59bed1a96aa46f9f5dca2f2a2634cb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/25/2021
ms.locfileid: "105554884"
---
Blazor は、シングルページ アプリケーション (SPA) のクライアント側フレームワークです。 このブラウザーは、アプリのホストとして機能するため、ナビゲーションと静的アセットの URI 要求に基づいて、個々の Razor コンポーネントの処理パイプラインとして機能します。 ミドルウェア処理パイプラインを使用してサーバー上で実行される ASP.NET Core アプリとは異なり、グローバル エラー処理に利用できる Razor コンポーネントの要求を処理するミドルウェア パイプラインはありません。 ただし、アプリはエラー処理コンポーネントをカスケード値として使用して、一元的な方法でエラーを処理できます。

次の `Error` コンポーネントは、それ自体を [`CascadingValue`](xref:blazor/components/cascading-values-and-parameters#cascadingvalue-component) として子コンポーネントに渡します。 次の例では、単にエラーをログするだけですが、コンポーネントのメソッドは、アプリが必要とする任意の方法で (複数のエラー処理メソッドを使用するなど) エラーを処理できます。 [挿入されたサービス](xref:blazor/fundamentals/dependency-injection)またはカスタム ロガーの実装を使用するよりもコンポーネントを使用する利点は、カスケードされたコンポーネントがコンテンツをレンダリングし、エラーが発生したときに CSS スタイルを適用できることです。

`Shared/Error.razor`:

```razor
@using Microsoft.Extensions.Logging
@inject ILogger<Error> Logger

<CascadingValue Value=this>
    @ChildContent
</CascadingValue>

@code {
    [Parameter]
    public RenderFragment ChildContent { get; set; }

    public void ProcessError(Exception ex)
    {
        Logger.LogError("Error:ProcessError - Type: {Type} Message: {Message}", 
            ex.GetType(), ex.Message);
    }
}
```

`App` コンポーネントで `Router` コンポーネントを `Error` コンポーネントでラップします。 これにより、`Error` コンポーネントは、`Error` コンポーネントが [`CascadingParameter`](xref:blazor/components/cascading-values-and-parameters#cascadingparameter-attribute) として受信されるアプリの任意のコンポーネントにカスケードできるようになります。

`App.razor`:

```razor
<Error>
    <Router ...>
        ...
    </Router>
</Error>
```

コンポーネントのエラーを処理するには:

* `Error` コンポーネントを [`@code`](xref:mvc/views/razor#code) ブロック内の [`CascadingParameter`](xref:blazor/components/cascading-values-and-parameters#cascadingparameter-attribute) として指定します。

  ```razor
  [CascadingParameter]
  public Error Error { get; set; }
  ```

* 適切な例外の種類を使用して、任意の `catch` ブロックでエラー処理メソッドを呼び出します。 この例の `Error` コンポーネントは 1 つの `ProcessError` メソッドのみを提供しますが、エラー処理コンポーネントは、アプリ全体の他のエラー処理要件に対処するために、任意の数のエラー処理メソッドを提供できます。

  ```csharp
  try
  {
      ...
  }
  catch (Exception ex)
  {
      Error.ProcessError(ex);
  }
  ```

前の例の `Error` コンポーネントと `ProcessError` メソッドを使用すると、ブラウザーの開発者ツール コンソールに、トラップされログされた次のエラーが示されます。

> 失敗: BlazorSample.Shared.Error[0] Error:ProcessError - Type: System.NullReferenceException メッセージ: オブジェクト参照がオブジェクトのインスタンスに設定されていません。

カスタム エラー メッセージ バーの表示やレンダリングされた要素の CSS スタイルの変更など、`ProcessError` メソッドがレンダリングに直接関与している場合は、`ProcessErrors` メソッドの最後で [`StateHasChanged`](xref:blazor/components/lifecycle#state-changes-statehaschanged) を呼び出して、UI を再レンダリングします。
