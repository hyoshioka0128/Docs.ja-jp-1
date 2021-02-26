---
title: ASP.NET Core Blazor Server の脅威の緩和のガイダンス
author: guardrex
description: Blazor Server アプリに対するセキュリティ上の脅威を緩和する方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 11/09/2020
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
uid: blazor/security/server/threat-mitigation
ms.openlocfilehash: 524a1f85838d083ed2f01afd96601bcd5e6ce12b
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280979"
---
# <a name="threat-mitigation-guidance-for-aspnet-core-blazor-server"></a>ASP.NET Core Blazor Server の脅威の緩和のガイダンス

Blazor Server アプリでは、サーバーとクライアントが長期間のリレーションシップを維持する、"*ステートフル*" なデータ処理モデルが採用されています。 永続的な状態は[回線](xref:blazor/state-management)によって維持されます。これは、長期間続く可能性がある接続にまたがることがあります。

ユーザーが Blazor Server サイトにアクセスすると、サーバーによってサーバーのメモリに回線が作成されます。 ユーザーが UI のボタンを選択したときなどに、この回線によって、レンダリングするコンテンツがブラウザーに指示されたり、イベントへの応答が行われたりします。 これらのアクションを実行するために、回線によって、ユーザーのブラウザーで JavaScript 関数が呼び出され、サーバー上で .NET メソッドが呼び出されます。 この JavaScript ベースの双方向の対話は、[JavaScript 相互運用 (JS 相互運用)](xref:blazor/call-javascript-from-dotnet) と呼ばれます。

JS 相互運用はインターネット経由で行われ、クライアントではリモート ブラウザーが使用されるため、Blazor Server アプリではほとんどの Web アプリのセキュリティの問題が共有されています。 このトピックでは、Blazor Server アプリに対する一般的な脅威について説明し、インターネットに接続されたアプリに重点を置いた脅威の緩和のガイダンスを提供します。

企業ネットワーク内やイントラネット内などの制約された環境では、緩和ガイダンスの一部は、次のいずれかになります。

* 制約された環境には適用されません。
* 制約された環境はセキュリティ リスクが低いため、実装する価値がありません。

## <a name="blazor-and-shared-state"></a>Blazor と共有状態

[!INCLUDE[](~/blazor/includes/security/blazor-shared-state.md)]

## <a name="resource-exhaustion"></a>リソースの枯渇

リソースの枯渇は、クライアントがサーバーとやり取りし、サーバーが過剰にリソースを消費する場合に発生する可能性があります。 過剰なリソースの消費が主に影響するのは次のものです。

* [CPU](#cpu)
* [メモリ](#memory)
* [クライアント接続](#client-connections)

サービス拒否 (DoS) 攻撃は、通常、アプリまたはサーバーのリソースを枯渇させようとします。 しかし、リソースの枯渇が必ずしもシステムへの攻撃の結果であるとは限りません。 たとえば、ユーザーの要求が多いことで、有限のリソースが使い果たされる可能性があります。 DoS については、「[サービス拒否 (DoS) 攻撃](#denial-of-service-dos-attacks)」のセクションで詳しく説明しています。

データベースやファイル ハンドル (ファイルの読み取りと書き込みに使用される) など、Blazor フレームワークの外部のリソースでも、リソースの枯渇が発生する可能性があります。 詳細については、「<xref:performance/performance-best-practices>」を参照してください。

### <a name="cpu"></a>CPU

CPU の枯渇は、1 つ以上のクライアントが大量の CPU 処理を実行するようサーバーに強制した場合に発生する可能性があります。

例として、"*フィボナッチ数*" を計算する Blazor Server アプリを考えてみましょう。 フィボナッチ数は、フィボナッチ数列から生成されます。この数列の各数は、先行する 2 つの数の合計です。 答えに達するまでに必要な作業量は、数列の長さと初期値のサイズによって異なります。 アプリでクライアントの要求に制限を設けていない場合、CPU 負荷の高い計算によって CPU の時間が占有され、他のタスクのパフォーマンスが低下する可能性があります。 過剰なリソース消費は、可用性に影響を与えるセキュリティ上の懸念事項です。

CPU の枯渇は、すべての公開アプリに関する懸念事項です。 通常の Web アプリでは、セーフガードとして要求と接続がタイムアウトになりますが、Blazor Server アプリには同じセーフガードが用意されていません。 CPU を集中的に使用する作業を実行する前に、Blazor Server アプリに適切なチェックと制限を含める必要があります。

### <a name="memory"></a>メモリ

メモリの枯渇は、1 つ以上のクライアントが大量のメモリを消費するようサーバーに強制した場合に発生する可能性があります。

例として、項目のリストを受け入れて表示するコンポーネントがある Blazor サーバー側アプリについて考えてみましょう。 Blazor アプリで許可される項目の数またはクライアントにレンダリングされる項目の数が制限されていない場合、メモリを集中的に使用する処理とレンダリングで、サーバーのメモリが占有され、パフォーマンスが低下する可能性があります。 サーバーがクラッシュしたり、クラッシュしたかのように遅くなったりする場合があります。

サーバー上のメモリの枯渇のシナリオに関連する項目のリストを保持および表示するために、次のシナリオを検討してください。

* `List<MyItem>` プロパティまたはフィールドの項目は、サーバーのメモリを使用します。 アプリで項目のリストを無制限に拡張することが許可されていると、サーバーでメモリが不足する危険性があります。 メモリが不足すると、現在のセッションが終了 (クラッシュ) し、そのサーバー インスタンス内のすべての同時セッションでメモリ不足の例外が発生します。 このシナリオが発生しないようにするには、同時ユーザーに項目の制限を課すデータ構造をアプリで使用する必要があります。
* レンダリングにページング スキームが使用されていない場合、サーバーにより、UI に表示されないオブジェクトに対して追加のメモリが使用されます。 項目の数に制限がないと、メモリの需要によって使用可能なサーバー メモリが枯渇する可能性があります。 このシナリオを回避するには、次のいずれかの方法を使用します。
  * レンダリング時にページ分割されたリストを使用します。
  * 最初の 100 から 1,000 項目のみを表示し、表示された項目以外の項目を検索するには、検索条件を入力するようユーザーに求めます。
  * より高度なレンダリング シナリオでは、"*仮想化*" をサポートするリストまたはグリッドを実装します。 仮想化を使用すると、リストには、現在ユーザーに表示されている項目のサブセットのみがレンダリングされます。 ユーザーが UI のスクロールバーを操作すると、コンポーネントでは表示に必要な項目だけがレンダリングされます。 現時点で、表示に必要ない項目は、セカンダリ ストレージに保持できます。これが理想的な方法です。 表示されていない項目をメモリに保持することもできますが、これはあまり理想的ではありません。

Blazor Server アプリには、WPF、Windows フォーム、Blazor WebAssembly など、ステートフル アプリ用の他の UI フレームワークと同様のプログラミング モデルが用意されています。 主な違いは、いくつかの UI フレームワークでは、アプリによって消費されるメモリがクライアントに属し、その個々のクライアントのみに影響することです。 たとえば、Blazor WebAssembly アプリは、完全にクライアント上で実行され、クライアントのメモリ リソースのみが使用されます。 Blazor Server のシナリオでは、アプリによって消費されるメモリは、サーバーに属し、サーバー インスタンス上のクライアント間で共有されます。

サーバー側のメモリ要求は、すべての Blazor Server アプリにとっての考慮事項です。 ただし、ほとんどの Web アプリはステートレスであり、要求の処理中に使用されたメモリは、応答が返されると解放されます。 一般的な推奨事項として、クライアント接続を保持する他のサーバー側アプリと同じように、クライアントが解放されているメモリ容量を割り当てられないようにしてください。 Blazor Server アプリによって消費されるメモリは、1 つの要求よりも長い時間保持されます。

> [!NOTE]
> 開発中は、プロファイラーを使用したり、トレースをキャプチャしてクライアントのメモリ要求を評価したりすることができます。 プロファイラーまたはトレースでは、特定のクライアントに割り当てられたメモリはキャプチャされません。 開発時に特定のクライアントのメモリ使用量をキャプチャするには、ダンプをキャプチャし、ユーザーの回線をルートとするすべてのオブジェクトのメモリ要求を調べます。

### <a name="client-connections"></a>クライアント接続

接続の枯渇は、1 つ以上のクライアントがサーバーへのコンカレント接続を多数開いていて、他のクライアントが新しい接続を確立できない場合に発生する可能性があります。

Blazor クライアントは、セッションごとに 1 つの接続を確立し、ブラウザー ウィンドウが開いている間は接続を開いたままにします。 すべての接続を維持するサーバーに対する要求は、Blazor アプリに固有のものではありません。 接続の永続的な性質と Blazor Server アプリのステートフルな性質を考えると、接続の枯渇はアプリの可用性に対するリスクが高くなります。

既定では、Blazor Server アプリのユーザーあたりの接続数に制限はありません。 アプリで接続制限が必要な場合は、次の方法の 1 つまたは複数を実行します。

* 認証を要求します。これにより、承認されていないユーザーがアプリに接続する能力が自然に制限されます。 このシナリオを有効にするには、ユーザーが新しいユーザーを自由にプロビジョニングできないようにする必要があります。
* ユーザーあたりの接続数を制限します。 接続の制限は、次の方法で実現できます。 正当なユーザーにアプリへのアクセスを許可するようにご注意ください (たとえば、クライアントの IP アドレスに基づいて接続の制限が確立されている場合)。
  * アプリ レベル:
    * エンドポイント ルーティングの拡張性。
    * アプリに接続し、ユーザーごとにアクティブなセッションを追跡するには、認証を要求する。
    * 制限に達したら新しいセッションを拒否する。
    * プロキシを使用したアプリへのプロキシ WebSocket 接続 (クライアントからアプリへの接続を多重化する [Azure SignalR Service](/azure/azure-signalr/signalr-overview) など)。 これにより、1 つのクライアントで確立できるよりも多くの接続容量がアプリに提供されるため、クライアントがサーバーへの接続を使い果たすことを防ぐことができます。
  * サーバー レベル: アプリの前にプロキシ/ゲートウェイを使用します。 たとえば [Azure Front Door](/azure/frontdoor/front-door-overview) を使用すると、アプリへの Web トラフィックのグローバルなルーティングを定義、管理、監視することができ、Blazor Server アプリがロング ポーリングを使用するように構成されている場合に機能します。
  
    > [!NOTE]
    > ロング ポーリングは Blazor Server アプリ用にサポートされていますが、[WebSocket が推奨されるトランスポート プロトコル](xref:blazor/host-and-deploy/server#azure-signalr-service)です。 現時点では、[Azure Front Door](/azure/frontdoor/front-door-overview) では WebSocket はサポートされていませんが、WebSocket のサポートは今後のサービスのリリースに向けて検討中です。

## <a name="denial-of-service-dos-attacks"></a>サービス拒否 (DoS) 攻撃

サービス拒否 (DoS) 攻撃では、クライアントがサーバーにそのリソースを 1 つ以上使い果たさせることで、アプリを利用できなくします。 Blazor Server アプリには、既定の制限があり、DoS 攻撃から保護するために <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions> で設定される他の ASP.NET Core や SignalR の制限に依存しています。

* <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DisconnectedCircuitMaxRetained?displayProperty=nameWithType>
* <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DisconnectedCircuitRetentionPeriod?displayProperty=nameWithType>
* <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout?displayProperty=nameWithType>
* <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.MaxBufferedUnacknowledgedRenderBatches?displayProperty=nameWithType>
* <xref:Microsoft.AspNetCore.SignalR.HubConnectionContextOptions.MaximumReceiveMessageSize?displayProperty=nameWithType>

詳細および構成コーディングの例については、次の記事を参照してください。

* <xref:blazor/fundamentals/signalr>
* <xref:signalr/configuration>

## <a name="interactions-with-the-browser-client"></a>ブラウザー (クライアント) との対話

クライアントは、JS 相互運用イベントのディスパッチとレンダリングの完了を通じてサーバーと対話します。 JS 相互運用通信は、JavaScript と .NET の間で双方向に行われます。

* ブラウザー イベントは、非同期方式でクライアントからサーバーにディスパッチされます。
* サーバーは、必要に応じて UI を非同期に再レンダリングして応答します。

### <a name="javascript-functions-invoked-from-net"></a>.NET から呼び出される JavaScript 関数

.NET メソッドから JavaScript への呼び出しの場合:

* すべての呼び出しには、構成可能なタイムアウトがあり、失敗すると、呼び出し元に <xref:System.OperationCanceledException> が返されます。
  * 呼び出し (<xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout?displayProperty=nameWithType>) の既定のタイムアウトは 1 分です。 この制限を構成するには、<xref:blazor/call-javascript-from-dotnet#harden-js-interop-calls> に関するセクションを参照してください。
  * キャンセル トークンを指定して、呼び出しごとに取り消しを制御できます。 可能な限り既定の呼び出しタイムアウトを使用し、キャンセル トークンが指定されている場合は、クライアントへの呼び出しに時間制限を設けます。
* JavaScript 呼び出しの結果は信頼できません。 ブラウザーで実行されている Blazor アプリ クライアントは、呼び出す JavaScript 関数を検索します。 関数が呼び出され、結果またはエラーのいずれかが生成されます。 悪意のあるクライアントは次のことを試みる可能性があります。
  * JavaScript 関数からエラーを返すことによって、アプリで問題を発生させる。
  * JavaScript 関数から予期しない結果を返すことによって、サーバーで意図しない動作を誘発させる。

上記のシナリオを防止するために、次の予防措置を講じてください。

* 呼び出し中に発生する可能性のあるエラーを考慮するために、[`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) ステートメント内に JS 相互運用呼び出しをラップします。 詳細については、「<xref:blazor/fundamentals/handle-errors#javascript-interop>」を参照してください。
* アクションを実行する前に、JS 相互運用の呼び出しから返されたデータ (エラー メッセージを含む) を検証します。

### <a name="net-methods-invoked-from-the-browser"></a>ブラウザーから呼び出される .NET メソッド

JavaScript から .NET メソッドへの呼び出しを信頼しないでください。 .NET メソッドが JavaScript に公開されている場合は、.NET メソッドの呼び出し方法を検討してください。

* JavaScript に公開されているすべての .NET メソッドは、アプリのパブリック エンドポイントと同様に扱います。
  * 入力を検証します。
    * 値が想定される範囲内であることを確認します。
    * ユーザーが要求された操作を実行するアクセス許可を持っていることを確認します。
  * .NET メソッドの呼び出しの一部として、過剰な量のリソースを割り当てないでください。 たとえば、チェックを実行し、CPU とメモリの使用量に制限を設けます。
  * 静的メソッドとインスタンス メソッドは JavaScript クライアントに公開できることを考慮してください。 設計で適切な制約を使用して状態の共有を行う必要がある場合を除き、セッション間で状態を共有しないでください。
    * 依存関係の挿入 (DI) によって最初に作成された `DotNetReference` オブジェクトを介して公開されるインスタンス メソッドの場合、オブジェクトはスコープ付きオブジェクトとして登録する必要があります。 これは、Blazor Server アプリが使用するすべての DI サービスに適用されます。
    * 静的メソッドの場合は、アプリが明示的にサーバー インスタンス上のすべてのユーザー間で状態を明示的に共有している場合を除き、クライアントにスコープを設定できない状態を確立しないようにしてください。
  * ユーザーが指定したデータをパラメーターで JavaScript の呼び出しに渡さないでください。 どうしてもパラメーターでデータを渡す必要がある場合は、[クロスサイト スクリプティング (XSS)](#cross-site-scripting-xss) の脆弱性を取り込むことなく、JavaScript コードでデータの引き渡しを処理できるようにします。 たとえば、要素の `innerHTML` プロパティを設定することによって、ユーザー指定のデータをドキュメント オブジェクト モデル (DOM) に書き込まないようにしてください。 [コンテンツ セキュリティ ポリシー (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) を使用して、`eval` およびその他の安全でない JavaScript プリミティブを無効にすることを検討してください。
* フレームワークのディスパッチ実装の上に .NET 呼び出しのカスタム ディスパッチを実装しないでください。 ブラウザーに .NET メソッドを公開することは高度なシナリオであるため、一般的な Blazor 開発にはお勧めしません。

### <a name="events"></a>イベント

イベントは、Blazor Server アプリへのエントリ ポイントを提供します。 Web アプリでエンドポイントを保護する場合と同じ規則が、Blazor Server アプリのイベント処理に適用されます。 悪意のあるクライアントが、イベントのペイロードとして送信したい任意のデータを送信する可能性があります。

次に例を示します。

* `<select>` の変更イベントでは、アプリがクライアントに提示したオプションに含まれていない値が送信されることがあります。
* `<input>` は、クライアント側の検証をバイパスして、テキスト データをサーバーに送信することがあります。

アプリが処理する任意のイベントのデータをアプリで検証する必要があります。 Blazor フレームワークの[フォーム コンポーネント](xref:blazor/forms-validation)では、基本的な検証が実行されます。 アプリでカスタム フォーム コンポーネントを使用する場合は、必要に応じてカスタム コードを記述してイベント データを検証する必要があります。

Blazor Server イベントは非同期であるため、アプリが新しいレンダリングを生成することによって反応するまでに、複数のイベントをサーバーにディスパッチすることができます。 これには、考慮すべきセキュリティへの影響がいくつかあります。 アプリでのクライアント アクションの制限は、イベント ハンドラー内で実行する必要があり、現在レンダリングされているビュー状態に依存してはいけません。

ユーザーがカウンターを最大 3 回インクリメントできるようにするカウンター コンポーネントを考えてみましょう。 カウンターをインクリメントするボタンは、条件付きで `count` の値に基づいています。

```razor
<p>Count: @count<p>

@if (count < 3)
{
    <button @onclick="IncrementCount" value="Increment count" />
}

@code 
{
    private int count = 0;

    private void IncrementCount()
    {
        count++;
    }
}
```

クライアントは、フレームワークがこのコンポーネントの新しいレンダリングを生成する前に、1 つ以上のインクリメント イベントをディスパッチできます。 その結果、UI によってすぐにボタンが削除されないため、ユーザーは `count` を "*3 回以上*" インクリメントできます。 3 回の `count` のインクリメントの制限を実現するための正しい方法を次の例で示します。

```razor
<p>Count: @count<p>

@if (count < 3)
{
    <button @onclick="IncrementCount" value="Increment count" />
}

@code 
{
    private int count = 0;

    private void IncrementCount()
    {
        if (count < 3)
        {
            count++;
        }
    }
}
```

ハンドラー内に `if (count < 3) { ... }` チェックを追加することによって、`count` をインクリメントするかどうかは、現在のアプリの状態に基づいて決定されます。 この決定は、前の例とは異なり、一時的に古くなっている可能性がある UI の状態に基づいていません。

### <a name="guard-against-multiple-dispatches"></a>複数のディスパッチに対するガード

イベント コールバックによって、長時間実行される操作 (外部サービスまたはデータベースからのデータのフェッチなど) が非同期に呼び出される場合は、ガードの使用を検討してください。 ガードを使用すると、視覚的なフィードバックを使用して、操作の進行中に、ユーザーが複数の操作をキューに入れることを防ぐことができます。 次のコンポーネント コードでは、`GetForecastAsync` がサーバーからデータを取得する間に、`isLoading` を `true` に設定します。 `isLoading` が `true` の間は、このボタンは UI では無効になります。

```razor
@page "/fetchdata"
@using BlazorServerSample.Data
@inject WeatherForecastService ForecastService

<button disabled="@isLoading" @onclick="UpdateForecasts">Update</button>

@code {
    private bool isLoading;
    private WeatherForecast[] forecasts;

    private async Task UpdateForecasts()
    {
        if (!isLoading)
        {
            isLoading = true;
            forecasts = await ForecastService.GetForecastAsync(DateTime.Now);
            isLoading = false;
        }
    }
}
```

前の例で示したガード パターンは、バックグラウンド操作が `async`-`await` パターンで非同期に実行される場合に機能します。

### <a name="cancel-early-and-avoid-use-after-dispose"></a>早期にキャンセルして、破棄後の使用を回避する

コンポーネントが破棄される場合は、「[複数のディスパッチに対するガード](#guard-against-multiple-dispatches)」セクションで説明したガードの使用に加えて、<xref:System.Threading.CancellationToken> を使用して長時間実行される操作を取り消すことを検討してください。 このアプローチには、コンポーネントで "*破棄後の使用*" を回避するという追加の利点があります。

```razor
@implements IDisposable

...

@code {
    private readonly CancellationTokenSource TokenSource = 
        new CancellationTokenSource();

    private async Task UpdateForecasts()
    {
        ...

        forecasts = await ForecastService.GetForecastAsync(DateTime.Now, 
            TokenSource.Token);

        if (TokenSource.Token.IsCancellationRequested)
        {
           return;
        }

        ...
    }

    public void Dispose()
    {
        TokenSource.Cancel();
    }
}
```

### <a name="avoid-events-that-produce-large-amounts-of-data"></a>大量のデータを生成するイベントを回避する

`oninput` や `onscroll` などの一部の DOM イベントでは、大量のデータが生成される可能性があります。 Blazor サーバー アプリでは、これらのイベントを使用しないようにしてください。

## <a name="additional-security-guidance"></a>セキュリティに関するその他のガイダンス

ASP.NET Core アプリをセキュリティで保護するためのガイダンスが、Blazor Server アプリに適用されます。詳細については、以降のセクションで説明します。

* [ログ記録と機密データ](#logging-and-sensitive-data)
* [HTTPS を使用した転送中の情報の保護](#protect-information-in-transit-with-https)
* [クロスサイト スクリプティング (XSS)](#cross-site-scripting-xss)
* [クロスオリジン保護](#cross-origin-protection)
* [クリック ジャッキング](#click-jacking)
* [オープン リダイレクト](#open-redirects)

### <a name="logging-and-sensitive-data"></a>ログ記録と機密データ

クライアントとサーバー間の JS 相互運用操作は、<xref:Microsoft.Extensions.Logging.ILogger> インスタンスと共にサーバーのログに記録されます。 Blazor では、実際のイベントや JS 相互運用の入力や出力などの機密情報のログ記録が回避されます。

サーバーでエラーが発生すると、フレームワークによってクライアントに通知され、セッションが破棄されます。 既定では、クライアントは、ブラウザーの開発者ツールで見られる一般的なエラー メッセージを受け取ります。

クライアント側のエラーには、呼び出し履歴は含まれず、エラーの原因についての詳細は提供されませんが、サーバー ログにはこのような情報が含まれています。 開発目的で、[詳細なエラーを有効にする](xref:blazor/fundamentals/handle-errors#blazor-server-detailed-circuit-errors)ことによって、機密性の高いエラー情報をクライアントが利用できるようにすることができます。

> [!WARNING]
> インターネット上でクライアントにエラー情報を公開することは、常に回避すべきセキュリティ リスクです。

### <a name="protect-information-in-transit-with-https"></a>HTTPS を使用した転送中の情報の保護

Blazor Server では、クライアントとサーバー間の通信に SignalR が使用されます。 Blazor Server では通常、SignalR がネゴシエートするトランスポート (通常は WebSocket) が使用されます。

Blazor Server では、サーバーとクライアントの間で送信されるデータの整合性と機密性は保証されません。 常に HTTPS を使用します。

### <a name="cross-site-scripting-xss"></a>クロスサイト スクリプティング (XSS)

クロスサイトス クリプティング (XSS) を使用すると、承認されていないパーティが、ブラウザーのコンテキストで任意のロジックを実行することができます。 侵害されたアプリが、クライアントで任意のコードを実行する可能性があります。 脆弱性を利用して、サーバーに対して多くの悪意のある操作が実行される可能性があります。

* 偽のイベントや無効なイベントをサーバーにディスパッチする。
* 失敗または無効なレンダリング完了をディスパッチする。
* レンダリング完了のディスパッチを回避する。
* JavaScript から相互運用呼び出しを .NET にディスパッチする。
* .NET から JavaScript への相互運用呼び出しの応答を変更する。
* JS 相互運用の結果に .NET をディスパッチすることを回避する。

Blazor Server フレームワークでは、前述のいくつかの脅威から保護するためのステップが実行されます。

* クライアントがレンダリング バッチを確認していない場合は、新しい UI 更新の生成を停止します。 <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.MaxBufferedUnacknowledgedRenderBatches?displayProperty=nameWithType> で構成されます。
* クライアントからの応答を受信せずに、1 分後に .NET から JavaScript への呼び出しをタイムアウトにします。 <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout?displayProperty=nameWithType> で構成されます。
* JS 相互運用中にブラウザーからのすべての入力に対して、次の基本的な検証を実行します。
  * .NET 参照は有効であり、.NET メソッドで予期される型であること。
  * データが不正な形式ではないこと。
  * メソッドの正しい数の引数がペイロードに存在していること。
  * メソッドを呼び出す前に、引数または結果が正しく逆シリアル化できること。
* 次に示すディスパッチされたイベントから、ブラウザーからのすべての入力に対して、次の基本的な検証を実行します。
  * イベントに有効な型があること。
  * イベントのデータを逆シリアル化できること。
  * イベントに関連付けられているイベント ハンドラーがあること。

脅威から保護し、適切なアクションを実行するには、フレームワークが実装するセーフガードに加えて、アプリが開発者によってコーディングされている必要があります。

* イベントを処理するときは常にデータを検証します。
* 無効なデータの受信時に適切なアクションを実行します。
  * データを無視して戻ります。 これにより、アプリは要求の処理を続行できます。
  * アプリは、入力が不正であり、正当なクライアントによって生成されたものではないと判断すると、例外をスローします。 例外をスローすると、回線が破棄され、セッションが終了します。
* ログに含まれるレンダリング バッチの完了によって提供されるエラー メッセージは信頼しないでください。 このエラーはクライアントによって提供され、クライアントが侵害されている可能性があるため、通常は信頼できません。
* JavaScript と .NET メソッドの間では、どちらの方向でも JS 相互運用呼び出しの入力を信頼しないでください。
* 引数または結果が正しく逆シリアル化された場合でも、引数と結果の内容が有効であることを検証するのはアプリの役割です。

XSS の脆弱性が存在するには、アプリがレンダリングされたページにユーザー入力を組み込む必要があります。 Blazor Server コンポーネントでは、`.razor` ファイルのマークアップが手続き型 C# ロジックに変換されるコンパイル時のステップが実行されます。 実行時に、C# ロジックによって、要素、テキスト、および子コンポーネントを記述する "*レンダリング ツリー*" が構築されます。 これは、JavaScript 命令のシーケンスを通じてブラウザーの DOM に適用されます (または、プリレンダリングの場合は HTML にシリアル化されます)。

* 通常の Razor 構文 (`@someStringValue` など) を使用してレンダリングされたユーザー入力では、XSS 脆弱性は公開されません。これは、Razor 構文が、テキストのみを書き込むことができるコマンドを使用して DOM に追加されるためです。 値に HTML マークアップが含まれている場合でも、値は静的なテキストとして表示されます。 プリレンダリング時に、出力は HTML エンコードされ、コンテンツも静的テキストとして表示されます。
* スクリプト タグは許可されていないため、アプリのコンポーネント レンダリング ツリーに含めることはできません。 コンポーネントのマークアップにスクリプト タグが含まれていると、コンパイル時のエラーが生成されます。
* コンポーネントの作成者は、Razor を使用せずに C# でコンポーネントを作成できます。 コンポーネントの作成者は、出力の生成時に適切な API を使用する必要があります。 たとえば、`builder.AddContent(0, someUserSuppliedString)` を使用します。`builder.AddMarkupContent(0, someUserSuppliedString)` は使用 "*しないでください*"。後者では XSS 脆弱性が生成される可能性があります。

XSS 攻撃からの保護の一環として、[コンテンツ セキュリティ ポリシー (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) などの XSS 軽減策を実装することを検討してください。

詳細については、「<xref:security/cross-site-scripting>」を参照してください。

### <a name="cross-origin-protection"></a>クロスオリジン保護

クロスオリジン攻撃では、サーバーに対してアクションを実行する別のオリジンのクライアントが関与します。 悪意のあるアクションは通常、GET 要求またはフォーム POST (クロスサイト リクエスト フォージェリ (CSRF)) ですが、悪意のある WebSocket を開くことも可能です。 Blazor Server アプリでは、[ハブ プロトコルを使用する他の SignalR アプリが提供するのと同じ保証](xref:signalr/security)を提供します。

* クロスオリジンを防ぐ追加の手段を講じない限り、Blazor Server アプリにアクセスされる可能性があります。 クロスオリジンのアクセスを無効にするには、CORS ミドルウェアをパイプラインに追加し、<xref:Microsoft.AspNetCore.Cors.DisableCorsAttribute> を Blazor エンドポイント メタデータに追加することによって、エンドポイントで CORS を無効にするか、[クロスオリジン リソース共有に SignalR を構成する](xref:signalr/security#cross-origin-resource-sharing)ことによって、許可されるオリジンのセットを制限します。
* CORS が有効になっている場合、CORS の構成によっては、アプリを保護するために追加のステップが必要になることがあります。 CORS がグローバルに有効になっている場合、Blazor Server ハブに対する CORS を無効にするには、エンドポイント ルート ビルダーで <xref:Microsoft.AspNetCore.Builder.ComponentEndpointRouteBuilderExtensions.MapBlazorHub%2A> を呼び出した後、エンドポイントのメタデータに <xref:Microsoft.AspNetCore.Cors.DisableCorsAttribute> メタデータを追加します。

詳細については、「<xref:security/anti-request-forgery>」を参照してください。

### <a name="click-jacking"></a>クリック ジャッキング

クリック ジャッキングでは、ユーザーをだまして攻撃を受けているサイトでアクションを実行させるために、サイトを異なるオリジンのサイト内の `<iframe>` としてレンダリングします。

`<iframe>` 内でのレンダリングからアプリを保護するには、[コンテンツ セキュリティ ポリシー (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) と `X-Frame-Options` ヘッダーを使用します。 詳細については、[MDN Web ドキュメント:X-Frame-Options](https://developer.mozilla.org/docs/Web/HTTP/Headers/X-Frame-Options) を参照してください。

### <a name="open-redirects"></a>オープン リダイレクト

Blazor Server アプリ セッションが開始されると、セッションの開始時に送信される URL の基本的な検証がサーバーによって実行されます。 フレームワークでは、回線を確立する前に、ベース URL が現在の URL の親であることが確認されます。 フレームワークでは、追加のチェックは実行されません。

クライアントでユーザーがリンクを選択すると、リンクの URL がサーバーに送信され、これにより実行するアクションが決まります。 たとえば、アプリがクライアント側のナビゲーションを実行したり、新しい場所に移動するようにブラウザーに指示したりすることができます。

コンポーネントでも、<xref:Microsoft.AspNetCore.Components.NavigationManager> を使用して、プログラムによってナビゲーション要求をトリガーすることができます。 このようなシナリオでは、アプリがクライアント側のナビゲーションを実行したり、新しい場所に移動するようにブラウザーに指示したりすることができます。

コンポーネントでは、次のことを行う必要があります。

* ナビゲーション呼び出しの引数の一部としてのユーザー入力の使用を避ける。
* 引数を検証して、ターゲットがアプリで許可されていることを保証する。

そうしないと、悪意のあるユーザーが、ブラウザーを攻撃者が制御するサイトに強制的に移動させる可能性があります。 このシナリオでは、攻撃者はアプリをだまして <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A?displayProperty=nameWithType> メソッドの呼び出しの一部としてユーザー入力を使用させます。

このアドバイスは、アプリの一部としてリンクをレンダリングする場合にも当てはまります。

* 可能であれば、相対リンクを使用します。
* 絶対リンクをページに含める前に、リンク先が有効であることを検証します。

詳細については、「<xref:security/preventing-open-redirects>」を参照してください。

## <a name="security-checklist"></a>セキュリティ チェックリスト

次のセキュリティの考慮事項のリストは、包括的なものではありません。

* イベントからの引数を検証する。
* JS 相互運用呼び出しからの入力と結果を検証する。
* .NET から JS への相互運用呼び出しに対してユーザー入力を使用しないようにする (または事前に検証する)。
* クライアントが、解放されたメモリ容量を割り当てないようにする。
  * コンポーネント内のデータ。
  * クライアントに返された `DotNetObject` 参照。
* 複数のディスパッチに対するガード。
* コンポーネントが破棄されるときに、実行時間の長い操作をキャンセルする。
* 大量のデータを生成するイベントを回避する。
* <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A?displayProperty=nameWithType> の呼び出しの一部としてユーザー入力を使用することは避け、避けられない場合は、最初に許可されたオリジンのセットに対して URL のユーザー入力を検証する。
* 承認は、UI の状態に基づいてではなく、コンポーネントの状態からのみ判断する。
* XSS 攻撃から保護するために[コンテンツ セキュリティ ポリシー (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) の使用を検討する。
* クリック ジャッキングから保護するため、CSP と [X-Frame-Options](https://developer.mozilla.org/docs/Web/HTTP/Headers/X-Frame-Options) の使用を検討する。
* CORS を有効にする場合や Blazor アプリの CORS を明示的に無効にする場合は、CORS 設定が適切であることを確認する。
* Blazor アプリのサーバー側の制限が、許容できないレベルのリスクなしに、許容できるユーザー エクスペリエンスを提供することを保証するためにテストする。
