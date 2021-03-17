---
title: ASP.NET Core Blazor 状態管理
author: guardrex
description: Blazor Server アプリで状態を維持する方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/29/2020
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
uid: blazor/state-management
zone_pivot_groups: blazor-hosting-models
ms.openlocfilehash: fee4473aede3fd213314c51a18284b4e4ad95b2b
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/06/2021
ms.locfileid: "102394669"
---
# <a name="aspnet-core-blazor-state-management"></a>ASP.NET Core Blazor 状態管理

::: zone pivot="webassembly"

Blazor WebAssembly アプリで作成されたユーザー状態は、ブラウザーのメモリに保持されます。

ブラウザーのメモリに保持されるユーザー状態の例としては、次のようなものがあります。

* レンダリングされた UI での、コンポーネント インスタンスの階層と最新のレンダリング出力。
* コンポーネント インスタンスに含まれるフィールドとプロパティの値。
* [依存関係の挿入 (DI)](xref:fundamentals/dependency-injection) サービス インスタンスに保持されているデータ。
* [JavaScript 相互運用](xref:blazor/call-javascript-from-dotnet)の呼び出しによって設定された値。

ユーザーがブラウザーを閉じて開き直したり、ページを再度読み込んだりすると、ブラウザーのメモリに保持されているユーザー状態は失われます。

> [!NOTE]
> [保護されたブラウザー ストレージ](xref:blazor/state-management?pivots=server#aspnet-core-protected-browser-storage) (<xref:Microsoft.AspNetCore.Components.Server.ProtectedBrowserStorage?displayProperty=fullName> 名前空間) は、ASP.NET Core のデータ保護に依存しており、Blazor Server アプリでのみサポートされます。

## <a name="persist-state-across-browser-sessions"></a>ブラウザー セッション間で状態を保持する

一般に、ユーザーが既存データを単に読み取るのではなく、活発にデータを作成している場合は、ブラウザー セッション間で状態を保持します。

ブラウザー セッション間で状態を維持するには、アプリで、ブラウザーのメモリとは異なる他の保存場所に、データを保持する必要があります。 状態は自動的に保存されません。 ステートフルなデータ保存を実装するアプリを開発するとき、措置を講じる必要があります。

データの保持は、通常、作成するためにユーザーが大きな労力を費やした、価値の高い状態の場合にのみ必要です。 次の例では、状態を維持することで、時間が節約され、商業活動の支援になります。

* 複数ステップの Web フォーム: 複数ステップの Web フォームのいくつかのステップを完了した後で、状態が失われた場合、ユーザーがデータを再入力するのでは時間がかかります。 このシナリオでは、フォームからユーザーが離れて後で戻ってきた場合、状態が失われます。
* ショッピング カート: 収益につながる可能性がある、アプリの中のビジネス上重要なコンポーネントを維持できます。 ユーザーが自分の状態を失い、そのため、自分のショッピング カートが消えると、後でサイトに戻ってきたとき、製品やサービスの購入数が減ることがあります。

アプリでは、"*アプリの状態*" のみが維持されます。 コンポーネント インスタンスやそのレンダー ツリーなど、UI は維持されません。 コンポーネントとレンダー ツリーは一般的にシリアル化されません。 アプリで、ツリー ビュー コントロールの展開されたノードなど、UI の状態を維持するには、カスタム コードを使用して、UI 状態の動作をシリアル化できるアプリ状態としてモデル化する必要があります。

## <a name="where-to-persist-state"></a>状態を維持する場所

一般に、状態を維持するために、次のような場所があります。

* [サーバー側ストレージ](#server-side-storage-wasm)
* [URL](#url-wasm)
* [ブラウザー ストレージ](#browser-storage-wasm)
* [メモリ内状態コンテナー サービス](#in-memory-state-container-service-wasm)

<h2 id="server-side-storage-wasm">サーバー側ストレージ</h2>

複数のユーザーとデバイスにまたがって永続的にデータを保持する場合は、Web API を使用してアクセスされる独立したサーバー側ストレージをアプリで使用できます。 次のオプションがあります。

* BLOB ストレージ
* キーと値のストレージ
* リレーショナル データベース
* テーブル ストレージ

データが保存された後は、ユーザーの状態は保持されていて、新しいブラウザー セッションで使用できます。

Blazor WebAssembly アプリはユーザーのブラウザー内で完全に実行されるため、ストレージ サービスやデータベースなど、セキュリティで保護された外部システムにアクセスするには、追加の手段が必要です。 Blazor WebAssembly アプリは、シングル ページ アプリケーション (SPA) と同じ方法でセキュリティ保護されます。 通常、アプリでは、[OAuth](https://oauth.net) や [OpenID Connect (OIDC)](https://openid.net/connect/) を使用してユーザーの認証を行った後、サーバー側アプリへの Web API 呼び出しを使用して、ストレージ サービスやデータベースとやり取りします。 Blazor WebAssembly アプリとストレージ サービスまたはデータベースの間のデータ転送は、サーバー側アプリによって仲介されます。 Blazor WebAssembly アプリではサーバー側アプリへの一時的な接続を維持し、サーバー側アプリではストレージへの永続的な接続が保持されます。

詳細については、次のリソースを参照してください。

* <xref:blazor/call-web-api>
* <xref:blazor/security/webassembly/index>
* Blazor の "*セキュリティと Identity* " に関する記事

Azure のデータ ストレージ オプションの詳細については、以下を参照してください。

* [Azure のデータベース](https://azure.microsoft.com/product-categories/databases/)
* [Azure Storage のドキュメント](/azure/storage/)

<h2 id="url-wasm">URL</h2>

ナビゲーションの状態を表わす一時的なデータについては、URL の一部としてデータをモデル化します。 たとえば、次のようなユーザー状態が URL でモデル化されます。

* 表示されるエンティティの ID。
* ページ付きグリッドでの現在のページ番号。

ユーザーが手動でページを再度読み込むと、ブラウザーのアドレス バーの内容が保持されます。

[`@page`](xref:mvc/views/razor#page) ディレクティブで URL パターンを定義する方法については、「<xref:blazor/fundamentals/routing>」を参照してください。

<h2 id="browser-storage-wasm">ブラウザー ストレージ</h2>

ユーザーが頻繁に作成する一時的なデータの場合、一般的に使用されるストレージの場所は、ブラウザーの [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) コレクションと [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage) コレクションです。

* `localStorage` の対象範囲はブラウザーのウィンドウです。 ユーザーがページを再読み込みするか、ブラウザーを閉じて再び開くと、状態は維持されます。 ユーザーが複数のブラウザー タブを開くと、状態はすべてのタブで同じになります。 データは直接消去されるまで `localStorage` に残ります。
* `sessionStorage` の対象範囲はブラウザーのタブです。ユーザーがタブを再読み込みすると、状態は維持されます。 ユーザーがタブかブラウザーを閉じると、状態は失われます。 ユーザーが複数のブラウザー タブを開くと、それぞれのタブには、他に依存しないそのタブだけのバージョンのデータが保持されます。

> [!NOTE]
> `localStorage` と `sessionStorage` は Blazor WebAssembly アプリで使用できますが、カスタム コードを記述するか、サードパーティのパッケージを使用する必要があります。

一般に、`sessionStorage` を使用しておけば安全です。 `sessionStorage` の場合、ユーザーが複数のタブを開き、以下に遭遇するリスクが回避されます。

* タブ間の状態保存に含まれるバグ。
* タブで他のタブの状態が上書きされるときの紛らわしい動作。

ブラウザーを閉じて再び開いても状態を維持することがアプリに求められる場合、`localStorage` が最善の選択肢です。

> [!WARNING]
> `localStorage` および `sessionStorage` に格納されているデータは、ユーザーによって表示または改ざんされる可能性があります。

<h2 id="in-memory-state-container-service-wasm">メモリ内状態コンテナー サービス</h2>

[!INCLUDE[](~/blazor/includes/state-container.md)]

## <a name="additional-resources"></a>その他のリソース

* [認証操作の前にアプリの状態を保存する](xref:blazor/security/webassembly/additional-scenarios#save-app-state-before-an-authentication-operation)
* <xref:blazor/call-web-api>
* <xref:blazor/security/webassembly/index>

::: zone-end

::: zone pivot="server"

Blazor Server はステートフル アプリ フレームワークです。 アプリでは、ほとんど常に、サーバーとの接続が維持されています。 ユーザーの状態は、"*回線*" の中のサーバーのメモリに保持されます。 

たとえば、回線には次のようなユーザー状態が保持されます。

* レンダリングされた UI での、コンポーネント インスタンスの階層と最新のレンダリング出力。
* コンポーネント インスタンスに含まれるフィールドとプロパティの値。
* 回線に範囲が設定されている[依存関係の挿入 (DI)](xref:fundamentals/dependency-injection) サービス インスタンスに保存されているデータ。

ユーザー状態は、[JavaScript 相互運用](xref:blazor/call-javascript-from-dotnet)の呼び出しによって設定される、ブラウザーのメモリ内の JavaScript 変数にも存在する場合があります。

ユーザーが一時的にネットワークに接続できなくなった場合は、Blazor により、ユーザーを元の状態で元の回線に再接続することが試みられます。 ただし、サーバーのメモリにある元の回線にいつでもユーザーを再接続できるわけではありません。

* サーバーでは、切断された回線を永久に保持することはできません。 サーバーでは、タイムアウト後、あるいはサーバーがメモリ不足になったとき、切断された回線を解放しなければなりません。
* 複数のサーバーが存在する負荷分散された展開環境では、個々のサーバーは、それがなくても全体的な要求量を処理できるようになると、作動しなくなったり、自動的に削除されたりすることがあります。 ユーザーが再接続しようとしたときに、ユーザーの要求を処理していた元のサーバーを使用できなくなることがあります。
* ユーザーはブラウザーを閉じて再び起動するか、ページを再読み込みできます。それでブラウザーのメモリに保存されている状態が削除されます。 たとえば、JavaScript 相互運用の呼び出しによって設定された JavaScript 変数の値は失われます。

ユーザーが元の回線に再接続できないとき、そのユーザーには、状態が空の新しい回線が与えられます。 これはデスクトップ アプリを終了してもう一度起動することと同じです。

## <a name="persist-state-across-circuits"></a>回線間で状態を維持する

一般に、ユーザーが既存データを単に読み取るのではなく、活発にデータを作成している場合は、回線間で状態を保持します。

回線間で状態を維持するには、アプリで、サーバーのメモリとは異なる他の保存場所に、データを保持する必要があります。 状態は自動的に保存されません。 ステートフルなデータ保存を実装するアプリを開発するとき、措置を講じる必要があります。

データの保持は、通常、作成するためにユーザーが大きな労力を費やした、価値の高い状態の場合にのみ必要です。 次の例では、状態を維持することで、時間が節約され、商業活動の支援になります。

* 複数ステップの Web フォーム: 複数ステップの Web フォームのいくつかのステップを完了した後で、状態が失われた場合、ユーザーがデータを再入力するのでは時間がかかります。 このシナリオでは、フォームからユーザーが離れて後で戻ってきた場合、状態が失われます。
* ショッピング カート: 収益につながる可能性がある、アプリの中のビジネス上重要なコンポーネントを維持できます。 ユーザーが自分の状態を失い、そのため、自分のショッピング カートが消えると、後でサイトに戻ってきたとき、製品やサービスの購入数が減ることがあります。

アプリでは、"*アプリの状態*" のみが維持されます。 コンポーネント インスタンスやそのレンダー ツリーなど、UI は維持されません。 コンポーネントとレンダー ツリーは一般的にシリアル化されません。 アプリで、ツリー ビュー コントロールの展開されたノードなど、UI の状態を維持するには、カスタム コードを使用して、UI 状態の動作をシリアル化できるアプリ状態としてモデル化する必要があります。

## <a name="where-to-persist-state"></a>状態を維持する場所

一般に、状態を維持するために、次のような場所があります。

* [サーバー側ストレージ](#server-side-storage-server)
* [URL](#url-server)
* [ブラウザー ストレージ](#browser-storage-server)
* [メモリ内状態コンテナー サービス](#in-memory-state-container-service-server)

<h2 id="server-side-storage-server">サーバー側ストレージ</h2>

複数のユーザーとデバイスにまたがって永続的にデータを保持する場合は、サーバー側ストレージをアプリで使用できます。 次のオプションがあります。

* BLOB ストレージ
* キーと値のストレージ
* リレーショナル データベース
* テーブル ストレージ

データが保存された後は、ユーザーの状態は保持されていて、新しい回線で使用できます。

Azure のデータ ストレージ オプションの詳細については、以下を参照してください。

* [Azure のデータベース](https://azure.microsoft.com/product-categories/databases/)
* [Azure Storage のドキュメント](/azure/storage/)

<h2 id="url-server">URL</h2>

ナビゲーションの状態を表わす一時的なデータについては、URL の一部としてデータをモデル化します。 たとえば、次のようなユーザー状態が URL でモデル化されます。

* 表示されるエンティティの ID。
* ページ付きグリッドでの現在のページ番号。

次の場合、ブラウザーのアドレス バーのコンテンツが保持されます。

* ユーザーがページを手動で再読み込みした。
* Web サーバーが利用できなくなり、別のサーバーに接続する目的で、ページの再読み込みがユーザーに強制される。

[`@page`](xref:mvc/views/razor#page) ディレクティブで URL パターンを定義する方法については、「<xref:blazor/fundamentals/routing>」を参照してください。

<h2 id="browser-storage-server">ブラウザー ストレージ</h2>

ユーザーが頻繁に作成する一時的なデータの場合、一般的に使用されるストレージの場所は、ブラウザーの [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) コレクションと [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage) コレクションです。

* `localStorage` の対象範囲はブラウザーのウィンドウです。 ユーザーがページを再読み込みするか、ブラウザーを閉じて再び開くと、状態は維持されます。 ユーザーが複数のブラウザー タブを開くと、状態はすべてのタブで同じになります。 データは直接消去されるまで `localStorage` に残ります。
* `sessionStorage` の対象範囲はブラウザーのタブです。ユーザーがタブを再読み込みすると、状態は維持されます。 ユーザーがタブかブラウザーを閉じると、状態は失われます。 ユーザーが複数のブラウザー タブを開くと、それぞれのタブには、他に依存しないそのタブだけのバージョンのデータが保持されます。

一般に、`sessionStorage` を使用しておけば安全です。 `sessionStorage` の場合、ユーザーが複数のタブを開き、以下に遭遇するリスクが回避されます。

* タブ間の状態保存に含まれるバグ。
* タブで他のタブの状態が上書きされるときの紛らわしい動作。

ブラウザーを閉じて再び開いても状態を維持することがアプリに求められる場合、`localStorage` が最善の選択肢です。

ブラウザー ストレージ使用時の注意事項:

* サーバー側データベースの使用に似ていますが、データの読み込みと保存は非同期です。
* サーバー側データベースとは異なり、プリレンダリング中はストレージを利用できません。プリレンダリング中は、要求されたページがブラウザーに存在しないためです。
* Blazor Server アプリの場合、数キロバイトのデータをストレージに保持するのが妥当です。 数キロバイトを超えると、パフォーマンスに影響が出ることを考慮する必要があります。ネットワーク中でデータが読み込まれ、保存されるためです。
* ユーザーはデータを見たり、改ざんしたりするかもしれません。 [ASP.NET Core のデータ保護](xref:security/data-protection/introduction)で、このリスクを軽減できます。 たとえば、[ASP.NET Core で保護されたブラウザー ストレージ](#aspnet-core-protected-browser-storage)では、ASP.NET Core のデータ保護が使用されます。

サードパーティ製 NuGet パッケージからは、`localStorage` と `sessionStorage` を使用するための API が与えられます。 [ASP.NET Core のデータ保護](xref:security/data-protection/introduction)を透過的に使用するパッケージを選択してみることもお勧めします。 データ保護を使用すると、保存データが暗号化され、保存データが改ざんされる潜在的リスクが減ります。 JSON でシリアル化されたデータがプレーンテキストで保存されている場合、ユーザーはブラウザー開発者ツールでデータを表示できます。また、保存データを変更できます。 データが性質上、取るに足りないものであれば、データ セキュリティを問題にする必要はないかもしれません。 たとえば、UI 要素に保存されている色を読み取られたり、変更されたりしたところで、それはユーザーや組織にとって大きなセキュリティ リスクではありません。 "*取り扱いに慎重を要するデータ*" を見たり、改ざんしたりすることをユーザーに禁止します。

::: moniker range=">= aspnetcore-5.0"

## <a name="aspnet-core-protected-browser-storage"></a>ASP.NET Core で保護されたブラウザー ストレージ

ASP.NET Core で保護されたブラウザー ストレージでは、[`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) と [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage) に対して [ASP.NET Core のデータ保護](xref:security/data-protection/introduction)が使用されます。

> [!NOTE]
> 保護されたブラウザー ストレージは、ASP.NET Core のデータ保護に依存しており、Blazor Server アプリでのみサポートされます。

### <a name="save-and-load-data-within-a-component"></a>コンポーネント内でデータを保存し、読み込む

ブラウザー ストレージのデータの読み込みまたは保存が必要なすべてのコンポーネントで、[`@inject`](xref:mvc/views/razor#inject) ディレクティブを使用して、次のいずれかのインスタンスを挿入します。

* `ProtectedLocalStorage`
* `ProtectedSessionStorage`

どれを選択するかは、使用するブラウザー ストレージの場所によって異なります。 次の例では、`sessionStorage` が使用されています。

```razor
@using Microsoft.AspNetCore.Components.Server.ProtectedBrowserStorage
@inject ProtectedSessionStorage ProtectedSessionStore
```

`@using` ディレクティブは、コンポーネントの代わりに、アプリの `_Imports.razor` ファイルに配置できます。 `_Imports.razor` ファイルを使用すると、アプリの中の大きなセグメントで、あるいはアプリ全体で名前空間を利用できます。

[Blazor Server プロジェクト テンプレート](xref:blazor/project-structure)に基づいてアプリの `Counter` コンポーネントに `currentCount` の値を保持するには、`ProtectedSessionStore.SetAsync` を使用するように `IncrementCount` メソッドを変更します。

```csharp
private async Task IncrementCount()
{
    currentCount++;
    await ProtectedSessionStore.SetAsync("count", currentCount);
}
```

もっと大規模で現実に即したアプリの場合、個々のフィールドを保管するというのはありそうにないシナリオです。 アプリでは多くの場合、状態が複雑なモデル オブジェクト全体を保存します。 `ProtectedSessionStore` では、複雑な状態オブジェクトを格納するため、JSON データが自動的にシリアル化および逆シリアル化されます。

前のコード例では、`currentCount` データは、ユーザーのブラウザーに `sessionStorage['count']` として保存されます。 データはプレーンテキストに保存されず、ASP.NET Core のデータ保護を使用して保護されます。 ブラウザーの開発者コンソールで `sessionStorage['count']` が評価される場合、暗号化されたデータを調べることができます。

ユーザーが後で `Counter` コンポーネントに戻ったときに `currentCount` データを回復するには (ユーザーが新しい回線にいる場合も含め)、`ProtectedSessionStore.GetAsync` を使用します。

```csharp
protected override async Task OnInitializedAsync()
{
    var result = await ProtectedSessionStore.GetAsync<int>("count");
    currentCount = result.Success ? result.Value : 0;
}
```

コンポーネントのパラメーターにナビゲーションの状態が含まれている場合は、`ProtectedSessionStore.GetAsync` を呼び出して、<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> ではなく、<xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> に `null` 以外の結果を割り当てます。 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> は、コンポーネントが最初にインスタンス化されたときに 1 回だけ呼び出されます。 後で、その同じページにいるとき、ユーザーが別の URL に移動しても <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> が再び呼び出されることはありません。 詳細については、「<xref:blazor/components/lifecycle>」を参照してください。

> [!WARNING]
> このセクションの例は、サーバーでプリレンダリングが有効になっていない場合に機能します。 プリレンダリングが有効になっていると、コンポーネントがプリレンダリングされているために JavaScript 相互運用の呼び出しを発行できないことを示すエラーが生成されます。
>
> プリレンダリングを無効にするか、プリレンダリングで使用するコードを追加します。 プリレンダリングと連動するコードを記述する方法の詳細については、「[プリレンダリングを処理する](#handle-prerendering)」を参照してください。

### <a name="handle-the-loading-state"></a>読み込み状態を処理する

ブラウザー ストレージはネットワーク接続経由で非同期にアクセスされるため、データが読み込まれ、コンポーネントで利用できるようになるまでに、常に一定の時間があります。 最良の結果を得るには、空のデータや既定のデータを表示するのではなく、読み込みが進行中のとき、読み込み状態メッセージをレンダリングします。

1 つの手法は、データが `null` かどうかを追跡することです。これは、まだ読み込み中であることを意味します。 既定の `Counter` コンポーネントでは、カウントは `int` に保持されます。 型 (`int`) に疑問符 (`?`) を追加することで、[`currentCount` を Null 許容にします](/dotnet/csharp/language-reference/builtin-types/nullable-value-types)。

```csharp
private int? currentCount;
```

カウントや **`Increment`** ボタンを無条件で表示するのではなく、<xref:System.Nullable%601.HasValue%2A> を調べることで、データが読み込まれている場合にのみこれらの要素を表示します。

```razor
@if (currentCount.HasValue)
{
    <p>Current count: <strong>@currentCount</strong></p>
    <button @onclick="IncrementCount">Increment</button>
}
else
{
    <p>Loading...</p>
}
```

### <a name="handle-prerendering"></a>プリレンダリングを処理する

プリレンダリング中:

* ユーザーのブラウザーに対話式で接続することはありません。
* ブラウザーには、JavaScript コードを実行できるページがまだありません。

`localStorage` または `sessionStorage` は、プリレンダリング中に使用できません。 コンポーネントがストレージとのやり取りを試みている場合、コンポーネントがプリレンダリングされているために JavaScript 相互運用の呼び出しを発行できないことを示すエラーが生成されます。

このエラーを解決する方法の 1 つは、プリレンダリングを無効にすることです。 これは通常、ブラウザーベースのストレージがアプリで頻繁に使用される場合、最良の選択肢となります。 プリレンダリングによってさらに複雑になり、アプリにとっては良いことがありません。アプリでは `localStorage` または `sessionStorage` が利用できなければ、役に立つコンテンツをプリレンダリングできないからです。

プリレンダリングを無効にするには、`Pages/_Host.cshtml` ファイルを開き、[コンポーネント タグ ヘルパー](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper)の `render-mode` 属性を、<xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.Server> に変更します。

```cshtml
<component type="typeof(App)" render-mode="Server" />
```

プリレンダリングは、`localStorage` や `sessionStorage` を使用しない他のページでは役に立つかもしれません。 プリレンダリングを保持するには、ブラウザーが回線に接続されるまで読み込み操作を延期します。 次はカウンター値を格納する例です。

```razor
@using Microsoft.AspNetCore.Components.Server.ProtectedBrowserStorage
@inject ProtectedLocalStorage ProtectedLocalStore

@if (isConnected)
{
    <p>Current count: <strong>@currentCount</strong></p>
    <button @onclick="IncrementCount">Increment</button>
}
else
{
    <p>Loading...</p>
}

@code {
    private int currentCount;
    private bool isConnected;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            isConnected = true;
            await LoadStateAsync();
            StateHasChanged();
        }
    }

    private async Task LoadStateAsync()
    {
        var result = await ProtectedLocalStore.GetAsync<int>("count");
        currentCount = result.Success ? result.Value : 0;
    }

    private async Task IncrementCount()
    {
        currentCount++;
        await ProtectedLocalStore.SetAsync("count", currentCount);
    }
}
```

### <a name="factor-out-the-state-preservation-to-a-common-location"></a>状態保存を取り出し、共通の場所に入れる

さまざまなコンポーネントがブラウザーベースのストレージに依存しているとき、状態プロバイダー コードを何回も再実装すると、コードが重複します。 コードの重複を回避する選択肢の 1 つは、状態プロバイダー ロジックをカプセル化する "*状態プロバイダーの親コンポーネント*" を作成することです。 状態保存メカニズムに関係なく、子コンポーネントは永続保存データとやりとりできます。

`CounterStateProvider` コンポーネントの次の例では、カウンター データは `sessionStorage` に保持されます。

```razor
@using Microsoft.AspNetCore.Components.Server.ProtectedBrowserStorage
@inject ProtectedSessionStorage ProtectedSessionStore

@if (isLoaded)
{
    <CascadingValue Value="@this">
        @ChildContent
    </CascadingValue>
}
else
{
    <p>Loading...</p>
}

@code {
    private bool isLoaded;

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    public int CurrentCount { get; set; }

    protected override async Task OnInitializedAsync()
    {
        var result = await ProtectedSessionStore.GetAsync<int>("count");
        currentCount = result.Success ? result.Value : 0;
        isLoaded = true;
    }

    public async Task SaveChangesAsync()
    {
        await ProtectedSessionStore.SetAsync("count", CurrentCount);
    }
}
```

`CounterStateProvider` コンポーネントによって読み込み段階が処理されます。読み込みが完了するまで、その子コンテンツがレンダリングされることはありません。

`CounterStateProvider` コンポーネントを使用するには、カウンター状態にアクセスする必要がある他のコンポーネントをコンポーネントのインスタンスでラップします。 アプリに含まれるすべてのコンポーネントが状態にアクセスできるようにするには、`App` コンポーネント (`App.razor`) で <xref:Microsoft.AspNetCore.Components.Routing.Router> を `CounterStateProvider` コンポーネントでラップします。

```razor
<CounterStateProvider>
    <Router AppAssembly="@typeof(Program).Assembly">
        ...
    </Router>
</CounterStateProvider>
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

ラップされたコンポーネントの元に永続化されたカウンター状態が届くので、それを変更できます。 次の `Counter` コンポーネントはパターンが実装されています。

```razor
@page "/counter"

<p>Current count: <strong>@CounterStateProvider.CurrentCount</strong></p>
<button @onclick="IncrementCount">Increment</button>

@code {
    [CascadingParameter]
    private CounterStateProvider CounterStateProvider { get; set; }

    private async Task IncrementCount()
    {
        CounterStateProvider.CurrentCount++;
        await CounterStateProvider.SaveChangesAsync();
    }
}
```

このコンポーネントは `ProtectedBrowserStorage` とやりとりするために必要ありませんし、"読み込み" 段階にも関係ありません。

前に説明したようにプリレンダリングを扱うには、カウンター データを利用するすべてのコンポーネントが自動的にプリレンダリングを使用するよう、`CounterStateProvider` を変更できます。 詳細については、「[プリレンダリングを処理する](#handle-prerendering)」セクションを参照してください。

一般に、次の場合、"*状態プロバイダーの親コンポーネント*" パターンが推奨されます。

* 多くのコンポーネントで状態を使用する。
* 最上位の状態オブジェクトを 1 つだけ保持する場合。

さまざまな状態オブジェクトを保持し、さまざまな場所でさまざまなオブジェクト サブセットを使用するには、状態をグローバルに保持しないことをお勧めします。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

## <a name="protected-browser-storage-experimental-nuget-package"></a>Protected Browser Storage 試験用 NuGet パッケージ

ASP.NET Core で保護されたブラウザー ストレージでは、[`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) と [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage) に対して [ASP.NET Core のデータ保護](xref:security/data-protection/introduction)が使用されます。

> [!WARNING]
> `Microsoft.AspNetCore.ProtectedBrowserStorage` はサポートのない試験用パッケージであり、運用環境での使用に適していません。
>
> パッケージは、ASP.NET Core 3.1 の Blazor Server アプリでのみ使用できます。

### <a name="configuration"></a>構成

1. [`Microsoft.AspNetCore.ProtectedBrowserStorage`](https://www.nuget.org/packages/Microsoft.AspNetCore.ProtectedBrowserStorage) へのパッケージ参照を追加します。
1. `Pages/_Host.cshtml` ファイルで、終了 `</body>` タグの内部に、次のスクリプトを追加します。

   ```cshtml
   <script src="_content/Microsoft.AspNetCore.ProtectedBrowserStorage/protectedBrowserStorage.js"></script>
   ```

1. `Startup.ConfigureServices` で `AddProtectedBrowserStorage` を呼び出し、`localStorage` サービスと `sessionStorage` サービスをサービス コレクションに追加します。

   ```csharp
   services.AddProtectedBrowserStorage();
   ```

### <a name="save-and-load-data-within-a-component"></a>コンポーネント内でデータを保存し、読み込む

ブラウザー ストレージのデータの読み込みまたは保存が必要なすべてのコンポーネントで、[`@inject`](xref:mvc/views/razor#inject) ディレクティブを使用して、次のいずれかのインスタンスを挿入します。

* `ProtectedLocalStorage`
* `ProtectedSessionStorage`

どれを選択するかは、使用するブラウザー ストレージの場所によって異なります。 次の例では、`sessionStorage` が使用されています。

```razor
@using Microsoft.AspNetCore.ProtectedBrowserStorage
@inject ProtectedSessionStorage ProtectedSessionStore
```

`@using` ステートメントは、コンポーネントの代わりに、`_Imports.razor` ファイルに入れることができます。 `_Imports.razor` ファイルを使用すると、アプリの中の大きなセグメントで、あるいはアプリ全体で名前空間を利用できます。

[Blazor Server プロジェクト テンプレート](xref:blazor/project-structure)に基づいてアプリの `Counter` コンポーネントに `currentCount` の値を保持するには、`ProtectedSessionStore.SetAsync` を使用するように `IncrementCount` メソッドを変更します。

```csharp
private async Task IncrementCount()
{
    currentCount++;
    await ProtectedSessionStore.SetAsync("count", currentCount);
}
```

もっと大規模で現実に即したアプリの場合、個々のフィールドを保管するというのはありそうにないシナリオです。 アプリでは多くの場合、状態が複雑なモデル オブジェクト全体を保存します。 `ProtectedSessionStore` では、JSON データが自動的にシリアル化され、逆シリアル化されます。

前のコード例では、`currentCount` データは、ユーザーのブラウザーに `sessionStorage['count']` として保存されます。 データはプレーンテキストに保存されず、ASP.NET Core のデータ保護を使用して保護されます。 ブラウザーの開発者コンソールで `sessionStorage['count']` が評価される場合、暗号化されたデータを調べることができます。

ユーザーが後で `Counter` コンポーネントに戻ったときに `currentCount` データを回復するには (まったく新しい回線にいる場合も含め)、`ProtectedSessionStore.GetAsync` を使用します。

```csharp
protected override async Task OnInitializedAsync()
{
    currentCount = await ProtectedSessionStore.GetAsync<int>("count");
}
```

コンポーネントのパラメーターにナビゲーションの状態が含まれている場合、`ProtectedSessionStore.GetAsync` を呼び出し、<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> ではなく、<xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> で結果を割り当てます。 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> は、コンポーネントが最初にインスタンス化されたときに 1 回だけ呼び出されます。 後で、その同じページにいるとき、ユーザーが別の URL に移動しても <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> が再び呼び出されることはありません。 詳細については、「<xref:blazor/components/lifecycle>」を参照してください。

> [!WARNING]
> このセクションの例は、サーバーでプリレンダリングが有効になっていない場合に機能します。 プリレンダリングが有効になっていると、コンポーネントがプリレンダリングされているために JavaScript 相互運用の呼び出しを発行できないことを示すエラーが生成されます。
>
> プリレンダリングを無効にするか、プリレンダリングで使用するコードを追加します。 プリレンダリングと連動するコードを記述する方法の詳細については、「[プリレンダリングを処理する](#handle-prerendering)」を参照してください。

### <a name="handle-the-loading-state"></a>読み込み状態を処理する

ブラウザー ストレージはネットワーク接続経由で非同期にアクセスされるため、データが読み込まれ、コンポーネントで利用できるようになるまでに、常に一定の時間があります。 最良の結果を得るには、空のデータや既定のデータを表示するのではなく、読み込みが進行中のとき、読み込み状態メッセージをレンダリングします。

1 つの手法は、データが `null` かどうかを追跡することです。これは、まだ読み込み中であることを意味します。 既定の `Counter` コンポーネントでは、カウントは `int` に保持されます。 型 (`int`) に疑問符 (`?`) を追加することで、[`currentCount` を Null 許容にします](/dotnet/csharp/language-reference/builtin-types/nullable-value-types)。

```csharp
private int? currentCount;
```

カウントや **`Increment`** ボタンを無条件で表示するのではなく、データが読み込まれる場合にのみこれらの要素を表示するように選択します。

```razor
@if (currentCount.HasValue)
{
    <p>Current count: <strong>@currentCount</strong></p>
    <button @onclick="IncrementCount">Increment</button>
}
else
{
    <p>Loading...</p>
}
```

### <a name="handle-prerendering"></a>プリレンダリングを処理する

プリレンダリング中:

* ユーザーのブラウザーに対話式で接続することはありません。
* ブラウザーには、JavaScript コードを実行できるページがまだありません。

`localStorage` または `sessionStorage` は、プリレンダリング中に使用できません。 コンポーネントがストレージとのやり取りを試みている場合、コンポーネントがプリレンダリングされているために JavaScript 相互運用の呼び出しを発行できないことを示すエラーが生成されます。

このエラーを解決する方法の 1 つは、プリレンダリングを無効にすることです。 これは通常、ブラウザーベースのストレージがアプリで頻繁に使用される場合、最良の選択肢となります。 プリレンダリングによってさらに複雑になり、アプリにとっては良いことがありません。アプリでは `localStorage` または `sessionStorage` が利用できなければ、役に立つコンテンツをプリレンダリングできないからです。

プリレンダリングを無効にするには、`Pages/_Host.cshtml` ファイルを開き、[コンポーネント タグ ヘルパー](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper)の `render-mode` 属性を、<xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.Server> に変更します。

```cshtml
<component type="typeof(App)" render-mode="Server" />
```

プリレンダリングは、`localStorage` や `sessionStorage` を使用しない他のページでは役に立つかもしれません。 プリレンダリングを保持するには、ブラウザーが回線に接続されるまで読み込み操作を延期します。 次はカウンター値を格納する例です。

```razor
@using Microsoft.AspNetCore.ProtectedBrowserStorage
@inject ProtectedLocalStorage ProtectedLocalStore

@if (isConnected)
{
    <p>Current count: <strong>@currentCount</strong></p>
    <button @onclick="IncrementCount">Increment</button>
}
else
{
    <p>Loading...</p>
}

@code {
    private int? currentCount;
    private bool isConnected = false;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            isConnected = true;
            await LoadStateAsync();
            StateHasChanged();
        }
    }

    private async Task LoadStateAsync()
    {
        currentCount = await ProtectedLocalStore.GetAsync<int>("count");
    }

    private async Task IncrementCount()
    {
        currentCount++;
        await ProtectedLocalStore.SetAsync("count", currentCount);
    }
}
```

### <a name="factor-out-the-state-preservation-to-a-common-location"></a>状態保存を取り出し、共通の場所に入れる

さまざまなコンポーネントがブラウザーベースのストレージに依存しているとき、状態プロバイダー コードを何回も再実装すると、コードが重複します。 コードの重複を回避する選択肢の 1 つは、状態プロバイダー ロジックをカプセル化する "*状態プロバイダーの親コンポーネント*" を作成することです。 状態保存メカニズムに関係なく、子コンポーネントは永続保存データとやりとりできます。

`CounterStateProvider` コンポーネントの次の例では、カウンター データは `sessionStorage` に保持されます。

```razor
@using Microsoft.AspNetCore.ProtectedBrowserStorage
@inject ProtectedSessionStorage ProtectedSessionStore

@if (isLoaded)
{
    <CascadingValue Value="@this">
        @ChildContent
    </CascadingValue>
}
else
{
    <p>Loading...</p>
}

@code {
    private bool isLoaded;

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    public int CurrentCount { get; set; }

    protected override async Task OnInitializedAsync()
    {
        CurrentCount = await ProtectedSessionStore.GetAsync<int>("count");
        isLoaded = true;
    }

    public async Task SaveChangesAsync()
    {
        await ProtectedSessionStore.SetAsync("count", CurrentCount);
    }
}
```

`CounterStateProvider` コンポーネントによって読み込み段階が処理されます。読み込みが完了するまで、その子コンテンツがレンダリングされることはありません。

`CounterStateProvider` コンポーネントを使用するには、カウンター状態にアクセスする必要がある他のコンポーネントをコンポーネントのインスタンスでラップします。 アプリに含まれるすべてのコンポーネントが状態にアクセスできるようにするには、`App` コンポーネント (`App.razor`) で <xref:Microsoft.AspNetCore.Components.Routing.Router> を `CounterStateProvider` コンポーネントでラップします。

```razor
<CounterStateProvider>
    <Router AppAssembly="@typeof(Program).Assembly">
        ...
    </Router>
</CounterStateProvider>
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

ラップされたコンポーネントの元に永続化されたカウンター状態が届くので、それを変更できます。 次の `Counter` コンポーネントはパターンが実装されています。

```razor
@page "/counter"

<p>Current count: <strong>@CounterStateProvider.CurrentCount</strong></p>
<button @onclick="IncrementCount">Increment</button>

@code {
    [CascadingParameter]
    private CounterStateProvider CounterStateProvider { get; set; }

    private async Task IncrementCount()
    {
        CounterStateProvider.CurrentCount++;
        await CounterStateProvider.SaveChangesAsync();
    }
}
```

このコンポーネントは `ProtectedBrowserStorage` とやりとりするために必要ありませんし、"読み込み" 段階にも関係ありません。

前に説明したようにプリレンダリングを扱うには、カウンター データを利用するすべてのコンポーネントが自動的にプリレンダリングを使用するよう、`CounterStateProvider` を変更できます。 詳細については、「[プリレンダリングを処理する](#handle-prerendering)」セクションを参照してください。

一般的に、次の場合、"*状態プロバイダーの親コンポーネント*" パターンが推奨されます。

* 多くのコンポーネントで状態を使用する。
* 最上位の状態オブジェクトを 1 つだけ保持する場合。

さまざまな状態オブジェクトを保持し、さまざまな場所でさまざまなオブジェクト サブセットを使用するには、状態をグローバルに保持しないことをお勧めします。

::: moniker-end

<h2 id="in-memory-state-container-service-server">メモリ内状態コンテナー サービス</h2>

[!INCLUDE[](~/blazor/includes/state-container.md)]

::: zone-end
