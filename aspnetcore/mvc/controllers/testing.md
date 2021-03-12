---
title: ASP.NET Core のコントローラーのロジックをテストする
author: ardalis
description: Moq と xUnit を使って ASP.NET Core のコントローラーのロジックをテストする方法を説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 7/22/2020
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
uid: mvc/controllers/testing
ms.openlocfilehash: 0f4fcceefdef946a5f29633c069cbb64ec345a3d
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586957"
---
# <a name="unit-test-controller-logic-in-aspnet-core"></a>ASP.NET Core でコントローラーのロジックの単体テストを行う

作成者: [Steve Smith](https://ardalis.com/)

::: moniker range=">= aspnetcore-3.0"

[単体テスト](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)では、アプリの一部をインフラストラクチャや依存関係から切り離してテストすることが必要とされます。 コントローラー ロジックの単体テストを行うと、単一のアクションの内容のみがテストされ、その依存関係やフレームワーク自体の動作はテストされません。

## <a name="unit-testing-controllers"></a>コントローラーの単体テスト

コントローラーのアクションの単体テストは、コントローラーの動作に注目するように設定します。 コントローラーの単体テストでは、[フィルター](xref:mvc/controllers/filters)、[ルーティング](xref:fundamentals/routing)、[モデル バインド](xref:mvc/models/model-binding)などのシナリオは除外します。 まとまって要求に応答するコンポーネント間のインタラクションをカバーするテストは、*統合テスト* によって処理されます。 統合テストの詳細については、「<xref:test/integration-tests>」を参照してください。

カスタム フィルターやルートを作成している場合は、コントローラーの特定のアクションに対するテストの一部としてではなく、単体テストを切り離して実行します。

コントローラーの単体テストを理解するために、次のサンプル アプリ内のコントローラーを確認してください。 

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/controllers/testing/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

Home コントローラーは、ブレーンストーミング セッションの一覧を表示し、POST 要求で新しいブレーンストーミング セッションを作成できるようにします。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Controllers/HomeController.cs?name=snippet_HomeController&highlight=1,5,10,31-32)]

上記のコントローラーの説明:

* [明示的な依存関係の原則](/dotnet/architecture/modern-web-apps-azure/architectural-principles#explicit-dependencies)に従います。
* [依存関係の注入 (DI)](xref:fundamentals/dependency-injection) を予期して、`IBrainstormSessionRepository` のインスタンスを提供します。
* モック オブジェクト フレームワークを使用するモックされた `IBrainstormSessionRepository` サービスでテストできます ([Moq](https://www.nuget.org/packages/Moq/) サービスなど)。 *モック オブジェクト* は、テストで使用されるプロパティとメソッドの動作が事前定義されている加工オブジェクトです。 詳細については、「[Introduction to integration tests](xref:test/integration-tests#introduction-to-integration-tests)」(統合テストの概要) を参照してください。

`HTTP GET Index` メソッドにはループや分岐はなく、1 つのメソッドを呼び出すだけです。 このアクションに対する単体テストでは、以下を行います。

* `IBrainstormSessionRepository` メソッドを使用する `GetTestSessions` サービスをモックする。 `GetTestSessions` が日付とセッション名を持つ 2 つのモック ブレーンストーミング セッションを作成する。
* `Index` メソッドを実行する。
* メソッドによって返された結果に関するアサーションを行う。
  * <xref:Microsoft.AspNetCore.Mvc.ViewResult> が返される。
  * [ViewDataDictionary.Model](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ViewDataDictionary.Model*) は `StormSessionViewModel`。
  * `ViewDataDictionary.Model`に格納された 2 つのブレーンストーミング セッションがある。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_Index_ReturnsAViewResult_WithAListOfBrainstormSessions&highlight=14-17)]

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_GetTestSessions)]

Home コントローラーの `HTTP POST Index` メソッドのテストでは、以下が検証されます。

* [ModelState.IsValid](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.IsValid*) が `false` の場合、アクション メソッドは、*400 Bad Request* <xref:Microsoft.AspNetCore.Mvc.ViewResult> と適切なデータを返す。
* `ModelState.IsValid` が `true` の場合:
  * リポジトリの `Add` メソッドが呼び出される。
  * <xref:Microsoft.AspNetCore.Mvc.RedirectToActionResult> と適切な引数が返される。

下の最初のテストに示すように、<xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.AddModelError*> を使用してエラーを追加することで、モデルの無効状態をテストできます。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_ModelState_ValidOrInvalid&highlight=9,16-17,38-41)]

[ModelState](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary) が有効でない場合は、GET 要求と同じ `ViewResult` が返されます。 このテストでは、無効なモデルを渡すことは試していません。 モデル バインドが実行されていないため、無効なモデルを渡すことは効果的なアプローチではありません (ただし、[統合テスト](xref:test/integration-tests)ではモデル バインドが使用されます)。 ここでは、モデル バインドはテストされていません。 これらの単体テストでは、アクション メソッドのコードだけをテストしています。

2 番目のテストでは、`ModelState` が有効な場合の検証を行います。

* 新しい `BrainstormSession` が追加される (リポジトリ経由)。
* メソッドが `RedirectToActionResult` と予期されるプロパティを返す。

呼び出されないモックの呼び出しは通常は無視されますが、Setup 呼び出しの最後で `Verifiable` を呼び出すと、テスト内でのモック検証が許可されます。 これは `mockRepo.Verify` の呼び出しで実行され、予期されるメソッドが呼び出されないとテストは失敗します。

> [!NOTE]
> このサンプルで使われている Moq ライブラリにより、検証可能な ("厳密な") モックと検証不可能なモック ("厳密でない" モックまたはスタブとも呼ばれます) を混在させることができます。 詳しくは、Moq の「[Customizing Mock Behavior](https://github.com/Moq/moq4/wiki/Quickstart#customizing-mock-behavior)」(モックの動作のカスタマイズ) をご覧ください。

サンプル アプリの [SessionController](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/controllers/testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Controllers/SessionController.cs) は、特定のブレーンストーミング セッションに関する情報を表示します。 このコントローラーには、無効な `id` 値を処理するロジックが含まれています (これらのシナリオをカバーするために、次のサンプルには 2 つの `return` シナリオがあります)。 最終的な `return` ステートメントにより、新しい `StormSessionViewModel` がビュー (*Controllers/SessionController.cs*) に返されます。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Controllers/SessionController.cs?name=snippet_SessionController&highlight=12-16,18-22,31)]

Session コントローラーの `Index`アクション内に各 `return` シナリオ用の 1 つのテストを含む単体テスト:

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/SessionControllerTests.cs?name=snippet_SessionControllerTests&highlight=2,11-14,18,31-32,36,50-55)]

Ideas コントローラーに移ると、アプリは、機能を Web API として `api/ideas` ルートで公開します。

* ブレーンストーミング セッションに関連付けられたアイデアの一覧 (`IdeaDTO`) が、`ForSession` メソッドによって返される。
* `Create` メソッドが新しいアイデアをセッションに追加する。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_ForSessionAndCreate&highlight=1-2,21-22)]

API の呼び出しでビジネス ドメイン エンティティを直接返すことは避けてください。 ドメイン エンティティ:

* 多くの場合、クライアントが必要とするもの以上のデータを含みます。
* アプリの内部ドメイン モデルと一般公開されている API が不必要に結合します。

ドメイン エンティティとクライアントに返される型の間のマッピングは、以下のように実行できます。

* サンプル アプリで使用しているように、LINQ `Select` を使用して手動で実行します。 詳細については、「[LINQ (統合言語クエリ)](/dotnet/standard/using-linq)」を参照してください。
* [AutoMapper](https://github.com/AutoMapper/AutoMapper) などのライブラリを使用して自動的に実行します。

次のサンプル アプリは、Ideas コントローラーの `Create` と `ForSession` API メソッドの単体テストを示します。

このサンプル アプリには、2 つの `ForSession` テストが含まれています。 最初のテストでは、無効なセッションに対して `ForSession` が <xref:Microsoft.AspNetCore.Mvc.NotFoundObjectResult> (HTTP Not Found) を返すかどうかを判断します。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests4&highlight=5,7-8,15-16)]

2 番目の `ForSession` テストでは、有効なセッションに対して `ForSession` がセッションのアイデアの一覧 (`<List<IdeaDTO>>`) を返すかどうかを判断します。 これらのチェックでは、最初のアイデアを調べて、その `Name` プロパティが正しいことも確認します。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests5&highlight=5,7-8,15-18)]

`ModelState` が無効の場合の `Create` メソッドの動作をテストするため、サンプル アプリでは、テストの一部としてコントローラーにモデル エラーを追加します。 単体テストでは、モデルの検証またはモデル バインドのテストを試さないでください。無効な `ModelState` に遭遇したときのアクション メソッドの動作だけをテストしてください。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests1&highlight=7,13)]

`Create` の 2 番目のテストでは、リポジトリが `null` を返すことに依存しているため、`null` を返すようにモック リポジトリが構成されます。 テスト データベース (メモリ内またはそれ以外の場所) を作成し、この結果を返すクエリを作成する必要はありません。 サンプル コードに示すように、このテストは単一のステートメントで実行できます。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests2&highlight=7-8,15)]

3 番目の `Create` テスト (`Create_ReturnsNewlyCreatedIdeaForSession`) では、リポジトリの `UpdateAsync` メソッドが呼び出されることを検証します。 モックが `Verifiable` で呼び出され、モック リポジトリの `Verify` メソッドが呼び出されて、検証可能なメソッドが実行されることを確認します。 `UpdateAsync` メソッドがデータを保存したことを確認するのは、単体テストの役割ではありません。それは統合テストで実行できます。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests3&highlight=20-22,28-33)]

## <a name="test-actionresultt"></a>テスト ActionResult\<T>

ASP.NET Core 2.1 以降では、 [actionresult \<T> ](xref:web-api/action-return-types#actionresultt-type) () を使用して、 <xref:Microsoft.AspNetCore.Mvc.ActionResult%601> から派生した型を返す `ActionResult` か、特定の型を返すことができます。

サンプル アプリには、特定のセッション `id` に対して `List<IdeaDTO>` を返すメソッドが含まれています。 セッションに `id` が存在しない場合、コントローラーは <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*> を返します。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_ForSessionActionResult&highlight=10,21)]

`ApiIdeasControllerTests`には、`ForSessionActionResult` コントローラーの 2 つのテストが含まれています。

最初のテストでは、コントローラーは `ActionResult` を返すが、存在しないセッション `id` の存在しないアイデアの一覧は返さないことを確認します。

* `ActionResult` の型は `ActionResult<List<IdeaDTO>>`。
* <xref:Microsoft.AspNetCore.Mvc.ActionResult`1.Result*> は <xref:Microsoft.AspNetCore.Mvc.NotFoundObjectResult>。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ForSessionActionResult_ReturnsNotFoundObjectResultForNonexistentSession&highlight=7,10,13-14)]

有効なセッション `id` に対する 2 番目のテストでは、メソッドが以下を返すことを確認します。

* `List<IdeaDTO>` 型の `ActionResult`。
* [Actionresult \<T> 。値](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Value*)は `List<IdeaDTO>` 型です。
* 一覧の最初の項目は、モック セッションに格納されているアイデアと一致する有効なアイデア (`GetTestSession` の呼び出しによって取得)。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ForSessionActionResult_ReturnsIdeasForSession&highlight=7-8,15-18)]

このサンプル アプリには、特定のセッションで新しい `Idea` を作成するメソッドも含まれています。 コントローラーは以下を返します。

* 無効なモデルに対する <xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*>。
* セッションが存在しない場合は <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*>。
* セッションが新しいアイデアで更新された場合は <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*>。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_CreateActionResult&highlight=9,16,29)]

`ApiIdeasControllerTests` には、`CreateActionResult` の 3 つのテストが含まれます。

最初のテストでは、無効なモデルに対して <xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*> が返されることを確認します。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsBadRequest_GivenInvalidModel&highlight=7,13-14)]

2 番目のテストでは、セッションが存在しない場合は <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*> が返されることを確認します。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsNotFoundObjectResultForNonexistentSession&highlight=5,15,22-23)]

有効なセッション `id` に対しては、最後のテストで以下を確認します。

* メソッドが `BrainstormSession` 型の `ActionResult` を返す。
* [Actionresult \<T> 。結果](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Result*)は <xref:Microsoft.AspNetCore.Mvc.CreatedAtActionResult> です。 `CreatedAtActionResult` は *201 Created* 応答に類似した `Location` ヘッダー付きの応答である。
* [Actionresult \<T> 。値](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Value*)は `BrainstormSession` 型です。
* セッション `UpdateAsync(testSession)` を更新するモック 呼び出しが呼び出された。 `Verifiable` メソッド呼び出しは、アサーション内で `mockRepo.Verify()` を実行することでチェックされます。
* セッションに対して 2 つの `Idea` オブジェクトが返された。
* 最後の項目 (`UpdateAsync` へのモック呼び出しによって追加された `Idea`) が、テスト中にセッションに追加された `newIdea` と一致する。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsNewlyCreatedIdeaForSession&highlight=20-22,28-34)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[コントローラー](xref:mvc/controllers/actions)は、すべての ASP.NET Core MVC アプリで中心的な役割を担います。 そのため、コントローラーが意図するとおりに動作するという信頼が必要です。 自動テストによって、アプリが運用環境にデプロイされる前にエラーを検出できます。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/controllers/testing/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="unit-tests-of-controller-logic"></a>コントローラー ロジックの単体テスト

[単体テスト](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)では、アプリの一部をインフラストラクチャや依存関係から切り離してテストすることが必要とされます。 コントローラー ロジックの単体テストを行うと、単一のアクションの内容のみがテストされ、その依存関係やフレームワーク自体の動作はテストされません。

コントローラーのアクションの単体テストは、コントローラーの動作に注目するように設定します。 コントローラーの単体テストでは、[フィルター](xref:mvc/controllers/filters)、[ルーティング](xref:fundamentals/routing)、[モデル バインド](xref:mvc/models/model-binding)などのシナリオは除外します。 まとまって要求に応答するコンポーネント間のインタラクションをカバーするテストは、*統合テスト* によって処理されます。 統合テストの詳細については、「<xref:test/integration-tests>」を参照してください。

カスタム フィルターやルートを作成している場合は、コントローラーの特定のアクションに対するテストの一部としてではなく、単体テストを切り離して実行します。

コントローラーの単体テストを理解するために、次のサンプル アプリ内のコントローラーを確認してください。 Home コントローラーは、ブレーンストーミング セッションの一覧を表示し、POST 要求で新しいブレーンストーミング セッションを作成できるようにします。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Controllers/HomeController.cs?name=snippet_HomeController&highlight=1,5,10,31-32)]

上記のコントローラーの説明:

* [明示的な依存関係の原則](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies)に従います。
* [依存関係の注入 (DI)](xref:fundamentals/dependency-injection) を予期して、`IBrainstormSessionRepository` のインスタンスを提供します。
* モック オブジェクト フレームワークを使用するモックされた `IBrainstormSessionRepository` サービスでテストできます ([Moq](https://www.nuget.org/packages/Moq/) サービスなど)。 *モック オブジェクト* は、テストで使用されるプロパティとメソッドの動作が事前定義されている加工オブジェクトです。 詳細については、「[Introduction to integration tests](xref:test/integration-tests#introduction-to-integration-tests)」(統合テストの概要) を参照してください。

`HTTP GET Index` メソッドにはループや分岐はなく、1 つのメソッドを呼び出すだけです。 このアクションに対する単体テストでは、以下を行います。

* `IBrainstormSessionRepository` メソッドを使用する `GetTestSessions` サービスをモックする。 `GetTestSessions` が日付とセッション名を持つ 2 つのモック ブレーンストーミング セッションを作成する。
* `Index` メソッドを実行する。
* メソッドによって返された結果に関するアサーションを行う。
  * <xref:Microsoft.AspNetCore.Mvc.ViewResult> が返される。
  * [ViewDataDictionary.Model](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ViewDataDictionary.Model*) は `StormSessionViewModel`。
  * `ViewDataDictionary.Model`に格納された 2 つのブレーンストーミング セッションがある。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_Index_ReturnsAViewResult_WithAListOfBrainstormSessions&highlight=14-17)]

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_GetTestSessions)]

Home コントローラーの `HTTP POST Index` メソッドのテストでは、以下が検証されます。

* [ModelState.IsValid](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.IsValid*) が `false` の場合、アクション メソッドは、*400 Bad Request* <xref:Microsoft.AspNetCore.Mvc.ViewResult> と適切なデータを返す。
* `ModelState.IsValid` が `true` の場合:
  * リポジトリの `Add` メソッドが呼び出される。
  * <xref:Microsoft.AspNetCore.Mvc.RedirectToActionResult> と適切な引数が返される。

下の最初のテストに示すように、<xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.AddModelError*> を使用してエラーを追加することで、モデルの無効状態をテストできます。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_ModelState_ValidOrInvalid&highlight=9,16-17,38-41)]

[ModelState](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary) が有効でない場合は、GET 要求と同じ `ViewResult` が返されます。 このテストでは、無効なモデルを渡すことは試していません。 モデル バインドが実行されていないため、無効なモデルを渡すことは効果的なアプローチではありません (ただし、[統合テスト](xref:test/integration-tests)ではモデル バインドが使用されます)。 ここでは、モデル バインドはテストされていません。 これらの単体テストでは、アクション メソッドのコードだけをテストしています。

2 番目のテストでは、`ModelState` が有効な場合の検証を行います。

* 新しい `BrainstormSession` が追加される (リポジトリ経由)。
* メソッドが `RedirectToActionResult` と予期されるプロパティを返す。

呼び出されないモックの呼び出しは通常は無視されますが、Setup 呼び出しの最後で `Verifiable` を呼び出すと、テスト内でのモック検証が許可されます。 これは `mockRepo.Verify` の呼び出しで実行され、予期されるメソッドが呼び出されないとテストは失敗します。

> [!NOTE]
> このサンプルで使われている Moq ライブラリにより、検証可能な ("厳密な") モックと検証不可能なモック ("厳密でない" モックまたはスタブとも呼ばれます) を混在させることができます。 詳しくは、Moq の「[Customizing Mock Behavior](https://github.com/Moq/moq4/wiki/Quickstart#customizing-mock-behavior)」(モックの動作のカスタマイズ) をご覧ください。

サンプル アプリの [SessionController](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/controllers/testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Controllers/SessionController.cs) は、特定のブレーンストーミング セッションに関する情報を表示します。 このコントローラーには、無効な `id` 値を処理するロジックが含まれています (これらのシナリオをカバーするために、次のサンプルには 2 つの `return` シナリオがあります)。 最終的な `return` ステートメントにより、新しい `StormSessionViewModel` がビュー (*Controllers/SessionController.cs*) に返されます。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Controllers/SessionController.cs?name=snippet_SessionController&highlight=12-16,18-22,31)]

Session コントローラーの `Index`アクション内に各 `return` シナリオ用の 1 つのテストを含む単体テスト:

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/SessionControllerTests.cs?name=snippet_SessionControllerTests&highlight=2,11-14,18,31-32,36,50-55)]

Ideas コントローラーに移ると、アプリは、機能を Web API として `api/ideas` ルートで公開します。

* ブレーンストーミング セッションに関連付けられたアイデアの一覧 (`IdeaDTO`) が、`ForSession` メソッドによって返される。
* `Create` メソッドが新しいアイデアをセッションに追加する。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_ForSessionAndCreate&highlight=1-2,21-22)]

API の呼び出しでビジネス ドメイン エンティティを直接返すことは避けてください。 ドメイン エンティティ:

* 多くの場合、クライアントが必要とするもの以上のデータを含みます。
* アプリの内部ドメイン モデルと一般公開されている API が不必要に結合します。

ドメイン エンティティとクライアントに返される型の間のマッピングは、以下のように実行できます。

* サンプル アプリで使用しているように、LINQ `Select` を使用して手動で実行します。 詳細については、「[LINQ (統合言語クエリ)](/dotnet/standard/using-linq)」を参照してください。
* [AutoMapper](https://github.com/AutoMapper/AutoMapper) などのライブラリを使用して自動的に実行します。

次のサンプル アプリは、Ideas コントローラーの `Create` と `ForSession` API メソッドの単体テストを示します。

このサンプル アプリには、2 つの `ForSession` テストが含まれています。 最初のテストでは、無効なセッションに対して `ForSession` が <xref:Microsoft.AspNetCore.Mvc.NotFoundObjectResult> (HTTP Not Found) を返すかどうかを判断します。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests4&highlight=5,7-8,15-16)]

2 番目の `ForSession` テストでは、有効なセッションに対して `ForSession` がセッションのアイデアの一覧 (`<List<IdeaDTO>>`) を返すかどうかを判断します。 これらのチェックでは、最初のアイデアを調べて、その `Name` プロパティが正しいことも確認します。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests5&highlight=5,7-8,15-18)]

`ModelState` が無効の場合の `Create` メソッドの動作をテストするため、サンプル アプリでは、テストの一部としてコントローラーにモデル エラーを追加します。 単体テストでは、モデルの検証またはモデル バインドのテストを試さないでください。無効な `ModelState` に遭遇したときのアクション メソッドの動作だけをテストしてください。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests1&highlight=7,13)]

`Create` の 2 番目のテストでは、リポジトリが `null` を返すことに依存しているため、`null` を返すようにモック リポジトリが構成されます。 テスト データベース (メモリ内またはそれ以外の場所) を作成し、この結果を返すクエリを作成する必要はありません。 サンプル コードに示すように、このテストは単一のステートメントで実行できます。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests2&highlight=7-8,15)]

3 番目の `Create` テスト (`Create_ReturnsNewlyCreatedIdeaForSession`) では、リポジトリの `UpdateAsync` メソッドが呼び出されることを検証します。 モックが `Verifiable` で呼び出され、モック リポジトリの `Verify` メソッドが呼び出されて、検証可能なメソッドが実行されることを確認します。 `UpdateAsync` メソッドがデータを保存したことを確認するのは、単体テストの役割ではありません。それは統合テストで実行できます。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests3&highlight=20-22,28-33)]

## <a name="test-actionresultt"></a>テスト ActionResult\<T>

ASP.NET Core 2.1 以降では、 [actionresult \<T> ](xref:web-api/action-return-types#actionresultt-type) () を使用して、 <xref:Microsoft.AspNetCore.Mvc.ActionResult%601> から派生した型を返す `ActionResult` か、特定の型を返すことができます。

サンプル アプリには、特定のセッション `id` に対して `List<IdeaDTO>` を返すメソッドが含まれています。 セッションに `id` が存在しない場合、コントローラーは <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*> を返します。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_ForSessionActionResult&highlight=10,21)]

`ApiIdeasControllerTests`には、`ForSessionActionResult` コントローラーの 2 つのテストが含まれています。

最初のテストでは、コントローラーは `ActionResult` を返すが、存在しないセッション `id` の存在しないアイデアの一覧は返さないことを確認します。

* `ActionResult` の型は `ActionResult<List<IdeaDTO>>`。
* <xref:Microsoft.AspNetCore.Mvc.ActionResult`1.Result*> は <xref:Microsoft.AspNetCore.Mvc.NotFoundObjectResult>。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ForSessionActionResult_ReturnsNotFoundObjectResultForNonexistentSession&highlight=7,10,13-14)]

有効なセッション `id` に対する 2 番目のテストでは、メソッドが以下を返すことを確認します。

* `List<IdeaDTO>` 型の `ActionResult`。
* [Actionresult \<T> 。値](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Value*)は `List<IdeaDTO>` 型です。
* 一覧の最初の項目は、モック セッションに格納されているアイデアと一致する有効なアイデア (`GetTestSession` の呼び出しによって取得)。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ForSessionActionResult_ReturnsIdeasForSession&highlight=7-8,15-18)]

このサンプル アプリには、特定のセッションで新しい `Idea` を作成するメソッドも含まれています。 コントローラーは以下を返します。

* 無効なモデルに対する <xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*>。
* セッションが存在しない場合は <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*>。
* セッションが新しいアイデアで更新された場合は <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*>。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_CreateActionResult&highlight=9,16,29)]

`ApiIdeasControllerTests` には、`CreateActionResult` の 3 つのテストが含まれます。

最初のテストでは、無効なモデルに対して <xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*> が返されることを確認します。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsBadRequest_GivenInvalidModel&highlight=7,13-14)]

2 番目のテストでは、セッションが存在しない場合は <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*> が返されることを確認します。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsNotFoundObjectResultForNonexistentSession&highlight=5,15,22-23)]

有効なセッション `id` に対しては、最後のテストで以下を確認します。

* メソッドが `BrainstormSession` 型の `ActionResult` を返す。
* [Actionresult \<T> 。結果](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Result*)は <xref:Microsoft.AspNetCore.Mvc.CreatedAtActionResult> です。 `CreatedAtActionResult` は *201 Created* 応答に類似した `Location` ヘッダー付きの応答である。
* [Actionresult \<T> 。値](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Value*)は `BrainstormSession` 型です。
* セッション `UpdateAsync(testSession)` を更新するモック 呼び出しが呼び出された。 `Verifiable` メソッド呼び出しは、アサーション内で `mockRepo.Verify()` を実行することでチェックされます。
* セッションに対して 2 つの `Idea` オブジェクトが返された。
* 最後の項目 (`UpdateAsync` へのモック呼び出しによって追加された `Idea`) が、テスト中にセッションに追加された `newIdea` と一致する。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsNewlyCreatedIdeaForSession&highlight=20-22,28-34)]

::: moniker-end

## <a name="additional-resources"></a>その他のリソース

* <xref:test/integration-tests>
* [Visual Studio で単体テストを作成して実行する](/visualstudio/test/unit-test-your-code)
* [ASP.NET CORE mvc 用の AspNetCore-Fluent テストライブラリ](https://github.com/ivaylokenov/MyTested.AspNetCore.Mvc): 厳密に型指定された単体テストライブラリ。 mvc および web API アプリをテストするための fluent インターフェイスを提供します。 ("*Microsoft では保守管理もサポートも行っていません。* ")
* [JustMockLite](https://github.com/telerik/JustMockLite):.NET 開発者向けのモック フレームワーク。 ("*Microsoft では保守管理もサポートも行っていません。* ")

