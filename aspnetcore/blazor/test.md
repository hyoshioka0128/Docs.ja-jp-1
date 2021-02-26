---
title: ASP.NET Core Blazor のコンポーネントをテストする
author: guardrex
description: Blazor アプリのコンポーネントをテストする方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/10/2020
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
uid: blazor/test
ms.openlocfilehash: 67ebfcd322ae08acf2fddae9bd6101f13fa77e7e
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280706"
---
# <a name="test-components-in-aspnet-core-blazor"></a>ASP.NET Core Blazor のコンポーネントをテストする

作成者: [Egil Hansen](https://egilhansen.com/)

テストは、安定し、保守性に優れたソフトウェアを構築する上で重要な作業です。

Blazor コンポーネントをテストするために、"*テスト対象のコンポーネント*" (CUT) に対して次のことが行われます。

* テスト用の関連する入力でレンダリングされます。
* 実行されるテストの種類によっては、操作または変更される可能性があります。 たとえば、ボタンの `onclick` イベントなどのイベント ハンドラーがトリガーされる可能性があります。
* 予想される値について検査されます。

## <a name="test-approaches"></a>テスト アプローチ

Blazor のコンポーネントをテストするための一般的な 2 つのアプローチは、エンド ツー エンド (E2E) テストと単体テストです。

* **単体テスト**: [単体テスト](/dotnet/core/testing/)は、次の機能を提供する単体テスト ライブラリを使用して記述されます。
  * コンポーネントのレンダリング。
  * コンポーネントの出力と状態の検査。
  * イベント ハンドラーとライフ サイクル メソッドのトリガー。
  * コンポーネントの動作が正しいことのアサーション。

  [bUnit](https://github.com/egil/bUnit) は、Razor コンポーネントの単体テストを実現するライブラリの一例です。

* **E2E テスト**: テスト ランナーでは、CUT を含む Blazor アプリを実行し、ブラウザー インスタンスを自動化します。 テスト ツールでは、ブラウザーを介して CUT を検査および操作します。 [Selenium](https://github.com/SeleniumHQ/selenium) は、Blazor アプリで使用できる E2E テスト フレームワークの一例です。

単体テストでは、Blazor コンポーネント (Razor/C#) のみが関係します。 サービスや JS 相互運用などの外部の依存関係は、複製する必要があります。 E2E テストでは、Blazor コンポーネントとそのすべての補助インフラストラクチャ (CSS、JS、DOM、ブラウザー API など) がテストに含まれます。

"*テスト スコープ*" では、テストの範囲を説明します。 通常、テスト スコープは、テストの速度に影響します。 単体テストは、アプリのサブシステムのサブセットに対して実行され、通常、ミリ秒単位で実行されます。 E2E テストでは、アプリのサブシステムの広範なグループがテストされ、完了するまで数秒かかることがあります。 

単体テストでは、CUT のインスタンスにもアクセスでき、コンポーネントの内部状態の検査と検証を行うことができます。 通常、これは、E2E テストでは行うことができません。

コンポーネントの環境に関しては、E2E テストでは、検証を開始する前に、予想される環境状態に達していることを確認する必要があります。 そうしなければ、結果は予測できません。 単体テストでは、CUT のレンダリングとテストのライフ サイクルはより統合されるため、テストの安定性が向上します。

E2E テストでは、複数のプロセス、ネットワークとディスクの I/O、その他のサブシステムのアクティビティを起動する必要があり、多くの場合、これによってテストの信頼性が低下します。 通常、単体テストは、このような種類の問題とは無関係です。

次の表は、この 2 つのテスト アプローチの違いをまとめたものです。

| 機能                       | 単体テスト                     | E2E テスト                             |
| -------------------------------- | -------------------------------- | --------------------------------------- |
| テスト スコープ                       | Blazor component (Razor/C#) のみ | Blazor component (Razor/C#) と CSS/JS |
| テストの実行時間              | ミリ秒                     | Seconds                                 |
| コンポーネント インスタンスへのアクセス | はい                              | いいえ                                      |
| 環境依存     | いいえ                               | はい                                     |
| [信頼性]                      | 信頼性が高い                    | 信頼性が低い                           |

## <a name="choose-the-most-appropriate-test-approach"></a>最適なテスト アプローチを選択する

実行するテストの種類を選択する際には、シナリオを考慮します。 次の表では、いくつかのシナリオについて説明します。

| シナリオ | 推奨されるアプローチ | 解説 |
| -------- | ------------------ | ------- |
| JS 相互運用ロジックのないコンポーネント | 単体テスト | Blazor コンポーネント内に JS 相互運用への依存関係がない場合、JS または DOM API にアクセスせずにコンポーネントをテストできます。 このシナリオでは、単体テストを選択することにデメリットはありません。 |
| 単純な JS 相互運用ロジックがあるコンポーネント | 単体テスト | コンポーネントでは、JS 相互運用を使用して、DOM に対してクエリを実行したり、アニメーションをトリガーしたりするのが一般的です。 <xref:Microsoft.JSInterop.IJSRuntime> インターフェイスを使用して JS 相互運用を簡単に複製できるので、このシナリオでは、通常、単体テストをお勧めします。 |
| 複雑な JS コードに依存するコンポーネント | 単体テストと個別の JS テスト | コンポーネントで、JS 相互運用を使用して大きなまたは複雑な JS ライブラリを呼び出すが、Blazor コンポーネントと JS ライブラリ間の対話は単純な場合、最適なアプローチは、コンポーネントと JS ライブラリまたはコードを 2 つの個別のパーツと見なして、それぞれを個別にテストすることです。 単体テスト ライブラリを使用して Blazor コンポーネントをテストし、JS テスト ライブラリを使用して JS をテストします。 |
| ブラウザー DOM の JS 操作に依存するロジックのあるコンポーネント | E2E テスト | コンポーネントの機能が JS および DOM の JS の操作に依存する場合、1 回の E2E テストで、JS と Blazor コードの両方をまとめて検証します。 これは、Blazor フレームワーク開発者が、Blazorのブラウザー レンダリング ロジックで採用しているアプローチで、C# と JS コードが密結合されます。 ブラウザーで Blazor コンポーネントをレンダリングするには、C# と JS コードが連携する必要があります。
| 複製が困難な依存関係を持つサードパーティ製コンポーネント ライブラリに依存するコンポーネント | E2E テスト | コンポーネントの機能が、JS 相互運用など、複製が困難な依存関係を持つサードパーティ製コンポーネント ライブラリに依存している場合、E2E テストは、そのコンポーネントをテストするための唯一のオプションになる可能性があります。 |

## <a name="test-components-with-bunit"></a>bUnit でコンポーネントをテストする

Blazor用の公式の Microsoft テスト フレームワークはありませんが、コミュニティ主導のプロジェクト [bUnit](https://github.com/egil/bUnit) では、Blazor コンポーネントの単体テストを行うための便利な方法を提供します。

> [!NOTE]
> bUnit はサードパーティ製のテスト ライブラリであり、Microsoft がサポートおよび保守するものではありません。

bUnit は、[MSTest](/dotnet/core/testing/unit-testing-with-mstest)、[NUnit](https://nunit.org/)、[xUnit](https://xunit.github.io/) などの汎用テスト フレームワークで動作します。 これらのテスト フレームワークでは、bUnit テストは通常の単体テストと同様の外観になります。 汎用のテスト フレームワークと統合された bUnit テストは、通常、次の機能を使用して実行されます。

* [Visual Studio テスト エクスプローラー](/visualstudio/test/run-unit-tests-with-test-explorer)。
* コマンド シェル内の [`dotnet test`](/dotnet/core/tools/dotnet-test) CLI コマンド。
* 自動化された DevOps テスト パイプライン。

> [!NOTE]
> テストの概念とテストの実装は、異なるテスト フレームワークでも類似していますが、同一ではありません。 ガイダンスについては、テスト フレームワークのドキュメントを参照してください。

次の例は、Blazor プロジェクト テンプレートに基づくアプリ内の `Counter` コンポーネントに対する bUnit テストの構造を示します。 `Counter` コンポーネントでは、ユーザーがページで選択するボタンに基づいてカウンターを表示し、増分します。

```razor
@page "/counter"

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;

    private void IncrementCount()
    {
        currentCount++;
    }
}
```

次の bUnit テストでは、ボタンが選択されたときに CUT のカウンターが正しく増分されるかどうかを検証します。

```csharp
[Fact]
public void CounterShouldIncrementWhenSelected()
{
    // Arrange
    using var ctx = new TestContext();
    var cut = ctx.RenderComponent<Counter>();
    var paraElm = cut.Find("p");

    // Act
    cut.Find("button").Click();
    var paraElmText = paraElm.TextContent;

    // Assert
    paraElmText.MarkupMatches("Current count: 1");
}
```

テストの各ステップでは、次のアクションが実行されます。

* *Arrange*: bUnit の `TestContext` を使用して、`Counter` コンポーネントがレンダリングされます。 CUT の段落要素 (`<p>`) が検出され、`paraElm` に割り当てられます。

* *Act*: ボタンの要素 (`<button>`) が配置され、`Click` を呼び出して選択されます。これにより、カウンターが増分され、段落タグ (`<p>`) の内容が更新されます。 段落要素のテキスト コンテンツは、`TextContent` を呼び出すことによって取得されます。

* *Assert*: テキスト コンテンツで `MarkupMatches` が呼び出され、予想される文字列 `Current count: 1` と一致するかどうかが検証されます。

> [!NOTE]
> `MarkupMatches` Assert メソッドは、通常の文字列比較アサーション (たとえば、`Assert.Equal("Current count: 1", paraElmText);`) とは異なり、`MarkupMatches` は、入力と予想される HTML マークアップのセマンティック比較を実行します。 セマンティック比較では、HTML セマンティクスが認識されます。つまり、意味のない空白のようなものは無視されます。 これにより、テストの安定性が向上します。 詳細については、「[Customizing the Semantic HTML Comparison](https://bunit.egilhansen.com/docs/verification/semantic-html-comparison)」 (HTML のセマンティック比較のカスタマイズ) を参照してください。

## <a name="additional-resources"></a>その他のリソース

* [Getting Started with bUnit](https://bunit.egilhansen.com/docs/getting-started/) (bUnit の概要): bUnit の説明には、テスト プロジェクトの作成、テスト フレームワーク パッケージの参照、テストの構築と実行に関するガイダンスが含まれています。
