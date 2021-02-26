---
title: ASP.NET Core Blazor アプリのエラーを処理する
author: guardrex
description: ASP.NET Core Blazor で、ハンドルされない例外を Blazor で管理する方法と、エラーを検出して処理するアプリを開発する方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 04/23/2020
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
uid: blazor/fundamentals/handle-errors
ms.openlocfilehash: f7cac477e2c5bca54e24ae3faeadff9b51bdcd0f
ms.sourcegitcommit: 422e8444b9f5cedc373be5efe8032822db54fcaf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/18/2021
ms.locfileid: "101101062"
---
# <a name="handle-errors-in-aspnet-core-blazor-apps"></a>ASP.NET Core Blazor アプリのエラーを処理する

この記事では、ハンドルされない例外を Blazor で管理する方法と、エラーを検出して処理するアプリを開発する方法について説明します。

## <a name="detailed-errors-during-development"></a>開発中の詳細なエラー

開発中に Blazor アプリが正常に機能していない場合、アプリからの詳細なエラー情報を受け取ることで、問題のトラブルシューティングと修正に役立ちます。 エラーが発生すると Blazor アプリによって画面の下部に金色のバーが表示されます。

* 開発中は、金色のバーによってブラウザー コンソールが表示され、そこで例外を確認できます。
* 実稼働環境では、金色のバーによって、エラーが発生したことがユーザーに通知され、ブラウザーの更新が推奨されます。

このエラー処理エクスペリエンスの UI は、Blazor プロジェクト テンプレートの一部です。

Blazor WebAssembly アプリでは、`wwwroot/index.html` ファイルでエクスペリエンスをカスタマイズします。

```html
<div id="blazor-error-ui">
    An unhandled error has occurred.
    <a href="" class="reload">Reload</a>
    <a class="dismiss">🗙</a>
</div>
```

Blazor Server アプリでは、`Pages/_Host.cshtml` ファイルでエクスペリエンスをカスタマイズします。

```cshtml
<div id="blazor-error-ui">
    <environment include="Staging,Production">
        An error has occurred. This application may no longer respond until reloaded.
    </environment>
    <environment include="Development">
        An unhandled exception has occurred. See browser dev tools for details.
    </environment>
    <a href="" class="reload">Reload</a>
    <a class="dismiss">🗙</a>
</div>
```

`blazor-error-ui` 要素は、Blazor テンプレート (`wwwroot/css/app.css` または `wwwroot/css/site.css`) に含まれるスタイルによって非表示にされ、エラーが発生したときに表示されます。

```css
#blazor-error-ui {
    background: lightyellow;
    bottom: 0;
    box-shadow: 0 -1px 2px rgba(0, 0, 0, 0.2);
    display: none;
    left: 0;
    padding: 0.6rem 1.25rem 0.7rem 1.25rem;
    position: fixed;
    width: 100%;
    z-index: 1000;
}

#blazor-error-ui .dismiss {
    cursor: pointer;
    position: absolute;
    right: 0.75rem;
    top: 0.5rem;
}
```

## <a name="blazor-server-detailed-circuit-errors"></a>Blazor Server の詳細な回線エラー

クライアント側のエラーには、呼び出し履歴は含まれず、エラーの原因についての詳細は提供されませんが、サーバー ログにはこのような情報が含まれています。 開発目的で、詳細なエラーを有効にすることによって、機密性の高い回線エラー情報をクライアントが利用できるようにすることができます。

次の方法を使用して Blazor Server の詳細なエラーを有効にします。

* <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DetailedErrors?displayProperty=nameWithType>.
* `DetailedErrors` 構成キーを `true` に設定します。これはアプリの Development 設定ファイル (`appsettings.Development.json`) で設定できます。 このキーは、値を `true` にした `ASPNETCORE_DETAILEDERRORS` 環境変数を使用することで設定することもできます。
* [SignalR のサーバー側ログ記録](xref:signalr/diagnostics#server-side-logging) (`Microsoft.AspNetCore.SignalR`) を [Debug](xref:Microsoft.Extensions.Logging.LogLevel) または [Trace](xref:Microsoft.Extensions.Logging.LogLevel) に設定して、SignalR の詳細なログを記録することができます。

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

> [!WARNING]
> インターネット上でクライアントにエラー情報を公開することは、常に回避すべきセキュリティ リスクです。

## <a name="how-a-blazor-server-app-reacts-to-unhandled-exceptions"></a>ハンドルされない例外に対して Blazor Server アプリがどのように反応するか

Blazor Server はステートフルなフレームワークです。 ユーザーはアプリを操作するときに、*回線* と呼ばれる、サーバーへの接続を維持します。 回線では、アクティブなコンポーネント インスタンスに加えて、次のような状態の他の多くの側面が保持されます。

* コンポーネントの表示される最新の出力。
* クライアント側のイベントによってトリガーされる可能性がある、イベント処理デリゲートの現在のセット。

ユーザーが複数のブラウザー タブでアプリを開いた場合は、独立した回線が複数あります。

Blazor は、ハンドルされない例外が発生した回線に対して、ほとんどの例外を致命的として処理します。 ハンドルされない例外のために回線が終了された場合、ユーザーはページを再読み込みして新しい回線を作成するだけで、アプリの操作を続行できます。 他のユーザーや他のブラウザー タブの回線である、終了された回線以外の回線は影響を受けません。 このシナリオは、クラッシュするデスクトップ アプリに似ています。 クラッシュしたアプリを再起動する必要がありますが、他のアプリは影響を受けません。

回線は、以下の理由でハンドルされない例外が発生したときに終了されます。

* ハンドルされない例外によって、回線が未定義の状態のままになることがよくあります。
* ハンドルされない例外の後は、アプリの通常動作を保証できません。
* 回線が存続している場合は、アプリにセキュリティの脆弱性が表れることがあります。

## <a name="manage-unhandled-exceptions-in-developer-code"></a>ハンドルされない例外を開発者コードで管理する

エラー後にアプリを続行するには、アプリがエラー処理ロジックを備えている必要があります。 この記事の後のセクションでは、ハンドルされない例外の潜在的原因について説明します。

運用環境では、フレームワークの例外メッセージやスタック トレースを UI に表示しないでください。 例外メッセージやスタック トレースを表示すると以下の可能性があります。

* エンド ユーザーに機密情報が開示される。
* 悪意のあるユーザーが、アプリ、サーバー、またはネットワークのセキュリティを侵害する可能性のある脆弱性をアプリの中で発見する助けになる。

## <a name="log-errors-with-a-persistent-provider"></a>永続的プロバイダーを使用してエラーをログに記録する

ハンドルされない例外が発生した場合、例外は、サービス コンテナー内に構成されている <xref:Microsoft.Extensions.Logging.ILogger> インスタンスにログ記録されます。 既定では、Blazor アプリは、コンソール ログ プロバイダーを使用してコンソール出力にログを記録します。 ログ サイズやログのローテーションを管理するプロバイダーを使用して、より永続的な場所にログを記録することを検討してください。 詳細については、「<xref:fundamentals/logging/index>」を参照してください。

開発中、Blazor は通常、デバッグの助けになるよう、ブラウザーのコンソールに例外の完全な詳細情報を送信します。 運用環境では、ブラウザー コンソールの詳細なエラーは既定で無効になっています。つまり、エラーはクライアントに送信されませんが、例外の完全な詳細はサーバー側でログに記録され続けます。 詳細については、「<xref:fundamentals/error-handling>」を参照してください。

ログに記録するインシデントと、ログに記録されるインシデントの重大度レベルを決定する必要があります。 悪意のあるユーザーが、意図的にエラーをトリガーできる可能性もあります。 たとえば、製品の詳細を表示するコンポーネントの URL に不明な `ProductId` が指定されているエラーのインシデントは、ログに記録しないようにします。 ログ記録では、すべてのエラーを重大度の高いインシデントとして扱わないようにしてください。

詳細については、「<xref:blazor/fundamentals/logging>」を参照してください。

## <a name="places-where-errors-may-occur"></a>エラーが発生する可能性のある場所

ハンドルされない例外は、以下のいずれかの場所で、フレームワークとアプリ コードによってトリガーされることがあります。

* [コンポーネントのインスタンス化](#component-instantiation)
* [ライフサイクル メソッド](#lifecycle-methods)
* [レンダリング ロジック](#rendering-logic)
* [イベント ハンドラー](#event-handlers)
* [コンポーネントの廃棄](#component-disposal)
* [JavaScript 相互運用](#javascript-interop)
* [Blazor Server の再レンダリング](#blazor-server-prerendering)

上記のハンドルされない例外については、この記事の後続のセクションで説明しています。

### <a name="component-instantiation"></a>コンポーネントのインスタンス化

Blazor によってコンポーネントのインスタンスが作成されるとき:

* コンポーネントのコンストラクターが呼び出されます。
* [`@inject`](xref:mvc/views/razor#inject) ディレクティブ、または [`[Inject]` 属性](xref:blazor/fundamentals/dependency-injection#request-a-service-in-a-component)を介して、コンポーネントのコンストラクターに渡されるシングルトン以外の DI サービスのコンストラクターが呼び出されます。

実行されたいずれかのコンストラクターまたは任意の `[Inject]` プロパティのセッターがハンドルされない例外をスローすると、Blazor Server の回線が失敗します。 フレームワークではコンポーネントをインスタンス化できないため、この例外は致命的です。 コンストラクターのロジックによって例外がスローされる可能性がある場合、アプリでは、エラー処理とログ記録を含む [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) ステートメントを使用して、例外をトラップする必要があります。

### <a name="lifecycle-methods"></a>ライフサイクル メソッド

コンポーネントのライフサイクルの間、Blazor によって以下の[ライフサイクル メソッド](xref:blazor/components/lifecycle)が呼び出されます。

* <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> / <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>
* <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSet%2A> / <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A>
* <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A>
* <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> / <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A>

いずれかのライフサイクル メソッドが同期的または非同期的に例外をスローした場合、例外は Blazor Server 回線にとって致命的です。 コンポーネントでライフサイクル メソッドのエラーに対処するには、エラー処理ロジックを追加します。

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> によって製品を取得するメソッドを呼び出す次の例では:

* `ProductRepository.GetProductByIdAsync` メソッドでスローされた例外は [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) ステートメントによって処理されます。
* `catch` ブロックの実行時には:
  * `loadFailed` が `true` に設定されます。これがユーザーにエラー メッセージを表示するために使われます。
  * エラーがログに記録されます。

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/handle-errors/ProductDetails.razor?name=snippet&highlight=11,27-39)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/handle-errors/ProductDetails.razor?name=snippet&highlight=11,27-39)]

::: moniker-end

### <a name="rendering-logic"></a>レンダリング ロジック

`.razor` コンポーネント ファイル内の宣言マークアップは、<xref:Microsoft.AspNetCore.Components.ComponentBase.BuildRenderTree%2A> と呼ばれる C# メソッドにコンパイルされます。 コンポーネントがレンダリングされるときには、<xref:Microsoft.AspNetCore.Components.ComponentBase.BuildRenderTree%2A> が実行されて、レンダリングされたコンポーネントの要素、テキスト、および子コンポーネントを記述するデータ構造が構築されます。

レンダリング ロジックは例外をスローすることがあります。 このシナリオの例は、`@someObject.PropertyName` が評価されても `@someObject` が `null` であるときに発生しています。 レンダリング ロジックによってスローされるハンドルされない例外は、Blazor Server 回線にとって致命的です。

レンダリング ロジックで null 参照例外が発生しないようにするには、そのメンバーにアクセスする前に `null` オブジェクトかどうかを調べます。 次の例では、`person.Address` が `null` の場合は `person.Address` プロパティにアクセスしません。

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/handle-errors/PersonExample.razor?name=snippet&highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/handle-errors/PersonExample.razor?name=snippet&highlight=1)]

::: moniker-end

上記のコードは、`person` が `null` でないことを前提としています。 多くの場合、コードの構造によって、コンポーネントがレンダリングされる時点でオブジェクトの存在が保証されます。 そのような場合は、レンダリング ロジックで `null` かどうかを調べる必要はありません。 前の例では、`person` が存在することを保証できるのは、コンポーネントがインスタンス化されるときに `person` が作成されるためです。

### <a name="event-handlers"></a>イベント ハンドラー

クライアント側のコードでは、以下を使用して、イベント ハンドラーが作成されるときに C# コードの呼び出しをトリガーします。

* `@onclick`
* `@onchange`
* その他の `@on...` 属性
* `@bind`

これらのシナリオでは、イベント ハンドラー コードによって、ハンドルされない例外がスローされることがあります。

イベント ハンドラーがハンドルされない例外をスローした場合 (たとえば、データベース クエリが失敗した)、例外は Blazor Server 回線にとって致命的です。 アプリが外部の理由で失敗する可能性のあるコードを呼び出す場合は、エラー処理とログ記録を含む [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) ステートメントを使用して、例外をトラップします。

ユーザー コードで例外のトラップと処理が行われない場合は、フレームワークによって例外がログに記録され、回線が終了されます。

### <a name="component-disposal"></a>コンポーネントの廃棄

たとえばユーザーが別のページに移動したため、コンポーネントが UI から削除されることがあります。 <xref:System.IDisposable?displayProperty=fullName> を実装しているコンポーネントが UI から削除されると、フレームワークにより、コンポーネントの <xref:System.IDisposable.Dispose%2A> メソッドが呼び出されます。

コンポーネントの `Dispose` メソッドがハンドルされない例外をスローした場合、例外は Blazor Server 回線にとって致命的です。 破棄のロジックによって例外がスローされる可能性がある場合、アプリでは、エラー処理とログ記録を含む [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) ステートメントを使用して、例外をトラップする必要があります。

コンポーネントの破棄の詳細については、「<xref:blazor/components/lifecycle#component-disposal-with-idisposable>」を参照してください。

### <a name="javascript-interop"></a>JavaScript 相互運用

<xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType> を使用すると、.NET コードによって、ユーザーのブラウザーで JavaScript ランタイムの非同期呼び出しを行えます。

<xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> を使用するエラー処理には、以下の条件が適用されます。

* <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> の呼び出しが同期的に失敗した場合は、.NET 例外が発生します。 たとえば、指定された引数をシリアル化できないため、<xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> の呼び出しが失敗することがあります。 開発者コードで例外をキャッチする必要があります。 イベント ハンドラーまたはコンポーネントのライフサイクル メソッドのアプリ コードで例外が処理されない場合、結果の例外は Blazor Server 回線にとって致命的です。
* <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> の呼び出しが非同期に失敗した場合、.NET <xref:System.Threading.Tasks.Task> が失敗します。 たとえば、JavaScript 側のコードが例外をスローしたり、`rejected` として完了した `Promise` を返したりするために、<xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> の呼び出しが失敗することがあります。 開発者コードで例外をキャッチする必要があります。 [`await`](/dotnet/csharp/language-reference/keywords/await) 演算子を使用する場合は、エラー処理とログ記録を含む [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) ステートメントでメソッド呼び出しをラップすることを検討してください。 そうしないと、失敗したコードにより、Blazor Server 回線にとって致命的なハンドルされない例外が発生する結果となります。
* 既定では、<xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> の呼び出しは一定の期間内に完了する必要があります。そうでないと呼び出しがタイムアウトになります。既定のタイムアウト期間は 1 分です。 タイムアウトにより、完了メッセージを送り返さないネットワーク接続や JavaScript コードでの損失からコードを保護します。 呼び出しがタイムアウトになった場合、結果の <xref:System.Threading.Tasks> は <xref:System.OperationCanceledException> で失敗します。 ログ記録を使用して例外をトラップし、処理します。

同様に、JavaScript コードを使用して、[`[JSInvokable]` 属性](xref:blazor/call-dotnet-from-javascript)によって示される .NET メソッドの呼び出しを開始できます。 ハンドルされない例外が、これらの .NET メソッドでスローされた場合:

* 例外は Blazor Server 回線にとって致命的なものとして扱われません。
* JavaScript 側の `Promise` が拒否されます。

.NET 側か、メソッド呼び出しの JavaScript 側か、どちらでエラー処理コードを使用するかを選択できます。

詳細については、次の記事を参照してください。

* <xref:blazor/call-javascript-from-dotnet>
* <xref:blazor/call-dotnet-from-javascript>

### <a name="blazor-server-prerendering"></a>Blazor Server のプリレンダリング

レンダリングされた HTML マークアップがユーザーの初期 HTTP 要求の一部として返されるように、[コンポーネント タグ ヘルパー](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper)を使用して Blazor コンポーネントを事前レンダリングすることができます。 これは以下によって機能します。

* 同じページに含まれるすべての事前レンダリング コンポーネントに対する新しい回線を作成する。
* 初期 HTML を生成する。
* ユーザーのブラウザーが同じサーバーに戻る SignalR 接続を確立するまで、回線を `disconnected` として扱う。 接続が確立されると、回線でのインタラクティビティが再開され、コンポーネントの HTML マークアップが更新されます。

ライフサイクル メソッドやレンダリング ロジックの実行中など、事前レンダリングでいずれかのコンポーネントからハンドルされない例外がスローされた場合:

* 例外は回線にとって致命的です。
* その例外は、<xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper> タグ ヘルパーの呼び出し履歴から破棄されます。 そのため、開発者コードによって例外が明示的にキャッチされない限り、HTTP 要求全体が失敗します。

事前レンダリングが失敗する通常の状況では、コンポーネントのビルドとレンダリングを続行しても意味がありません。これは、動作中のコンポーネントはレンダリングできないためです。

事前レンダリング中に発生する可能性のあるエラーに耐えるには、例外をスローする可能性のあるコンポーネント内にエラー処理ロジックを配置する必要があります。 エラー処理とログ記録を含む [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) ステートメントを使用してください。 [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) ステートメント内に <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper> タグ ヘルパーをラップするのではなく、<xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper> タグ ヘルパーによってレンダリングされるコンポーネント内にエラー処理ロジックを配置します。

## <a name="advanced-scenarios"></a>高度なシナリオ

### <a name="recursive-rendering"></a>再帰的レンダリング

コンポーネントは、再帰的に入れ子にすることができます。 これは、再帰的なデータ構造を表現する場合に役立ちます。 たとえば `TreeNode` コンポーネントで、ノードの子ごとにより多くの `TreeNode` コンポーネントをレンダリングできます。

再帰的にレンダリングする場合は、無限の再帰となるようなコーディング パターンは回避します。

* 循環が含まれるデータ構造は再帰的にレンダリングしないでください。 たとえば、子にそれ自体が含まれるツリー ノードはレンダリングしないでください。
* 循環を含むひと続きのレイアウトは作成しないでください。 たとえば、レイアウトがそれ自体であるレイアウトは作成しないようにします。
* エンドユーザーが、悪意のあるデータ入力や JavaScript の相互運用呼び出しを通して、再帰による不変 (ルール) を犯さないようにします。

レンダリング中の無限ループ:

* レンダリング プロセスが永久に続行されるようになります。
* これは終了しないループを作成するのと同じです。

これらのシナリオでは、影響を受けた Blazor Server 回線が失敗し、スレッドでは通常、以下のことが試みられます。

* オペレーティング システムで許されている限りの CPU 時間を無期限に消費します。
* サーバー メモリの量を無制限に消費します。 メモリを無制限に使用することは、すべての反復処理で、終了しないループによってコレクションにエントリが追加されるシナリオと同じです。

無限の再帰パターンを回避するには、再帰的なレンダリング コードに適切な停止条件が含まれるようにします。

### <a name="custom-render-tree-logic"></a>カスタム レンダリング ツリーのロジック

ほとんどの Blazor コンポーネントは `.razor` ファイルとして実装され、出力をレンダリングするために <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> 上で動作するロジックを生成するためにコンパイルされます。 開発者は、手続き型の C# コードを使用して <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> ロジックを手動で実装できます。 詳細については、「<xref:blazor/advanced-scenarios#manual-rendertreebuilder-logic>」を参照してください。

> [!WARNING]
> 手動のレンダー ツリー ビルダー ロジックの使用は、高度で安全ではないシナリオと考えられています。一般のコンポーネント開発には推奨されません。

<xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> コードが記述される場合は、開発者がコードの正確性を保証する必要があります。 たとえば、開発者は以下のことを確認する必要があります。

* <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.OpenElement%2A> と <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.CloseElement%2A> の呼び出しが正しく調整されている。
* 正しい場所にのみ属性が追加される。

手動レンダー ツリー ビルダーのロジックが正しくないと、クラッシュ、サーバーのハング、セキュリティ脆弱性など、不特定の未定義の動作が発生する可能性があります。

手動のレンダリング ツリー ビルダー ロジックは、アセンブリ コードや MSIL 命令を手動で記述するのと同じレベルの複雑さと、同じレベルの *危険性* を伴うことを考慮に入れてください。
