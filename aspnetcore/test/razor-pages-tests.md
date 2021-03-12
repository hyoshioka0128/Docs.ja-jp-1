---
title: ASP.NET Core での Razor Pages の単体テスト
author: rick-anderson
description: Razor Pages アプリの単体テストを作成する方法を学習します。
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
uid: test/razor-pages-tests
ms.openlocfilehash: a67f51561d0a165db04418f2040ffe68e3ebe8ce
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588491"
---
# <a name="razor-pages-unit-tests-in-aspnet-core"></a>ASP.NET Core での Razor Pages の単体テスト

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core では Razor Pages アプリの単体テストをサポートしています。 データ アクセス層 (DAL) とページ モデルのテストによって、次のことを保証できます。

* Razor Pages アプリの一部は、アプリの構築時に独立して、または 1 つの単位として連携して機能します。
* クラスとメソッドの担当のスコープは限定されています。
* アプリの動作に関する追加のドキュメントがあります。
* コードの更新によって発生したエラーである回帰は、自動ビルドおよびデプロイ時に検出されます。

このトピックでは、Razor Pages アプリと単体テストの基本的な知識があることを前提としています。 Razor Pages アプリまたはテストの概念になじみがない場合は、次のトピックを参照してください。

* <xref:razor-pages/index>
* <xref:tutorials/razor-pages/razor-pages-start>
* [dotnet テストと xUnit を使用した .NET Core での単体テスト C#](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/test/razor-pages-tests/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

サンプル プロジェクトは、次の 2 つのアプリで構成されています。

| アプリ         | プロジェクト フォルダー                     | 説明 |
| ----------- | ---------------------------------- | ----------- |
| メッセージ アプリ | *src/RazorPagesTestSample*         | ユーザーがメッセージの追加、1 つのメッセージの削除、すべてのメッセージの削除、およびメッセージの分析 (メッセージあたりの平均単語数の検出) をできるようにします。 |
| アプリをテストする    | *tests/RazorPagesTestSample.Tests* | メッセージ アプリの DAL およびインデックス ページ モデルの単体テストに使用されます。 |

IDE の組み込みテスト機能 ([Visual Studio](/visualstudio/test/unit-test-your-code) や [Visual Studio for Mac](/dotnet/core/tutorials/using-on-mac-vs-full-solution) など) を使用して、テストを実行できます。 [Visual Studio Code](https://code.visualstudio.com/) またはコマンド ラインを使用する場合は、コマンドプロンプトで *tests/RazorPagesTestSample.Tests* フォルダー内の次のコマンドを実行します。

```dotnetcli
dotnet test
```

## <a name="message-app-organization"></a>メッセージ アプリの構成

メッセージ アプリは、次の特性を持つ Razor Pages メッセージ システムです。

* アプリのインデックス ページ (*Pages/Index. cshtml* と *Pages/Index. cshtml*) には、メッセージの追加、削除、および分析 (メッセージあたりの平均単語数の検出) を制御する UI およびページ モデル メソッドが用意されています。
* メッセージは、`Id` (キー) と `Text` (メッセージ) の 2 つのプロパティを持つ `Message` クラス (*Data/Message.cs*) によって記述されます。 `Text` プロパティは必須であり、200 文字までに制限されています。
* メッセージは [Entity Framework のメモリ内データベース](/ef/core/providers/in-memory/)&#8224; を使用して格納されます。
* アプリでは、データベース コンテキスト クラス `AppDbContext` (*Data/AppDbContext.cs*) 内に DAL を格納します。 DAL メソッドは `virtual` とマークされ、これにより、テストで使用するためのメソッドのモックを作成できます。
* アプリの起動時にデータベースが空の場合、メッセージ ストアが 3 つのメッセージで初期化されます。 これらの *シードされたメッセージ* も、テストで使用されます。

&#8224; EF トピック「[InMemory を使用したテスト](/ef/core/miscellaneous/testing/in-memory)」では、MSTest を使用したテストでメモリ内データベースを使用する方法について説明します。 このトピックでは、[xUnit](https://xunit.github.io/) テスト フレームワークを使用します。 テストの概念とテストの実装は、異なるテスト フレームワークでも類似していますが、同一ではありません。

サンプル アプリではリポジトリ パターンを使用しておらず、[Unit of Work (UoW) パターン](https://martinfowler.com/eaaCatalog/unitOfWork.html)の有効な例ではありませんが、Razor Pages ではこれらの開発パターンをサポートしています。 詳細については、[インフラストラクチャの永続レイヤーの設計](/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)に関するページと「<xref:mvc/controllers/testing>」を参照してください (このサンプルでは、リポジトリ パターンを実装しています)。

## <a name="test-app-organization"></a>テスト アプリの構成

テスト アプリは、*tests/RazorPagesTestSample.Tests* フォルダー内のコンソール アプリです。

| テスト アプリ フォルダー | 説明 |
| --------------- | ----------- |
| *UnitTests*     | <ul><li>*DataAccessLayerTest.cs* には、DAL の単体テストが含まれています。</li><li>*IndexPageTests.cs* には、インデックス ページ モデルの単体テストが含まれています。</li></ul> |
| *Utilities*     | データベースが各テストのベースライン条件にリセットされるように、各 DAL 単体テストの新しいデータベース コンテキスト オプションを作成するために使用する `TestDbContextOptions` メソッドが含まれています。 |

テスト フレームワークは、[xUnit](https://xunit.github.io/) です。 オブジェクトのモック フレームワークは [Moq](https://github.com/moq/moq4) です。

## <a name="unit-tests-of-the-data-access-layer-dal"></a>データアクセス層 (DAL) の単体テスト

メッセージ アプリには、`AppDbContext` クラス (*src/RazorPagesTestSample/Data/AppDbContext.cs*) に含まれる 4 つのメソッドを持つ DAL があります。 テスト アプリで、各メソッドには 1 つか 2 つの単体テストがあります。

| DAL メソッド               | 関数                                                                   |
| ------------------------ | -------------------------------------------------------------------------- |
| `GetMessagesAsync`       | データベースから `Text` プロパティで並べ替えられた `List<Message>` を取得します。 |
| `AddMessageAsync`        | `Message` をデータベースに追加します。                                          |
| `DeleteAllMessagesAsync` | データベースからすべての `Message` エントリを削除します。                           |
| `DeleteMessageAsync`     | `Id` によって、データベースから 1 つの `Message` を削除します。                      |

DAL の単体テストでは、各テストの新しい `AppDbContext` を作成する際に、<xref:Microsoft.EntityFrameworkCore.DbContextOptions> が必要です。 各テストの `DbContextOptions` を作成する 1 つの方法として、<xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder> を使用します。

```csharp
var optionsBuilder = new DbContextOptionsBuilder<AppDbContext>()
    .UseInMemoryDatabase("InMemoryDb");

using (var db = new AppDbContext(optionsBuilder.Options))
{
    // Use the db here in the unit test.
}
```

この方法の問題は、各テストでは、前のテストでデータベースがどのような状態になっていても、それを受け取るということです。 相互に干渉しないアトミック単体テストを作成しようとする場合に、これが問題になる可能性があります。 `AppDbContext` でテストごとに新しいデータベース コンテキストを強制的に使用させるには、新しいサービス プロバイダーに基づいた `DbContextOptions` インスタンスを指定します。 テスト アプリでは、その `Utilities` クラス メソッド `TestDbContextOptions` (*tests/RazorPagesTestSample.Tests/Utilities/Utilities.cs*) を使用してこれを行う方法を示しています。

[!code-csharp[](razor-pages-tests/samples/3.x/tests/RazorPagesTestSample.Tests/Utilities/Utilities.cs?name=snippet1)]

DAL 単体テストで `DbContextOptions` を使用すると、新しいデータベース インスタンスを使用して、各テストをアトミックに実行できます。

```csharp
using (var db = new AppDbContext(Utilities.TestDbContextOptions()))
{
    // Use the db here in the unit test.
}
```

`DataAccessLayerTest` クラスの各テスト メソッド (*UnitTests/DataAccessLayerTest.cs*) は、同様の Arrange-Act-Assert パターンに従います。

1. Arrange (環境構築):データベースがテスト用に構成され、期待される結果が定義されています。
1. Act (実行):テストが実行されます。
1. Assert (動作確認):アサーションによって、テスト結果が成功であるかどうかが判断されます。

たとえば、`DeleteMessageAsync` メソッドは、`Id` によって識別された 1 つのメッセージの削除を担当します (*src/RazorPagesTestSample/Data/AppDbContext.cs*)。

[!code-csharp[](razor-pages-tests/samples/3.x/src/RazorPagesTestSample/Data/AppDbContext.cs?name=snippet4)]

このメソッドには 2 つのテストがあります。 1 つのテストでは、メッセージがデータベースに存在する場合に、このメソッドによってメッセージが削除されます。 他方のメソッドでは、削除対象のメッセージ `Id` が存在しない場合に、データベースが変更されないことをテストします。 `DeleteMessageAsync_MessageIsDeleted_WhenMessageIsFound` メソッドを次に示します。

[!code-csharp[](razor-pages-tests/samples_snapshot/3.x/tests/RazorPagesTestSample.Tests/UnitTests/DataAccessLayerTest.cs?name=snippet1)]

まず、このメソッドでは、Arrange ステップを実行し、Act ステップの準備を行います。 シード処理メッセージが取得され、`seedMessages` で保持されます。 シード処理メッセージはデータベースに保存されます。 `Id` が `1` のメッセージは、削除対象として設定されます。 `DeleteMessageAsync` メソッドが実行されると、予期されるメッセージには、`Id` が `1` のメッセージを除くすべてのメッセージが含まれるはずです。 `expectedMessages` 変数は、この予期される結果を表します。

[!code-csharp[](razor-pages-tests/samples/3.x/tests/RazorPagesTestSample.Tests/UnitTests/DataAccessLayerTest.cs?name=snippet1)]

メソッドは次のように動作します。`DeleteMessageAsync` メソッドは `1`の `recId` を渡して実行されます。

[!code-csharp[](razor-pages-tests/samples/3.x/tests/RazorPagesTestSample.Tests/UnitTests/DataAccessLayerTest.cs?name=snippet2)]

最後に、メソッドはコンテキストから `Messages` を取得し、`expectedMessages` と比較して、2 つが等しいことをアサートします。

[!code-csharp[](razor-pages-tests/samples/3.x/tests/RazorPagesTestSample.Tests/UnitTests/DataAccessLayerTest.cs?name=snippet3)]

2 つの `List<Message>` が同じであることを比較するために:

* メッセージは `Id` 順に並べ替えられます。
* メッセージのペアは、`Text` プロパティで比較されます。

類似のテスト メソッド `DeleteMessageAsync_NoMessageIsDeleted_WhenMessageIsNotFound` では、存在しないメッセージを削除しようとした結果を確認します。 この場合、データベース内の予期されるメッセージは、`DeleteMessageAsync` メソッドが実行された後の実際のメッセージと等しくなるはずです。 データベースのコンテンツへの変更はないはずです。

[!code-csharp[](razor-pages-tests/samples/3.x/tests/RazorPagesTestSample.Tests/UnitTests/DataAccessLayerTest.cs?name=snippet4)]

## <a name="unit-tests-of-the-page-model-methods"></a>ページ モデル メソッドの単体テスト

別の単体テスト セットは、ページ モデル メソッドのテストを担当します。 メッセージ アプリで、インデックス ページ モデルは *src/RazorPagesTestSample/Pages/Index.cshtml.cs* の `IndexModel` クラスにあります。

| ページ モデル メソッド | 関数 |
| ----------------- | -------- |
| `OnGetAsync` | `GetMessagesAsync` メソッドを使用して、UI の DAL からメッセージを取得します。 |
| `OnPostAddMessageAsync` | [ModelState](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary) が有効な場合、`AddMessageAsync` を呼び出して、データベースにメッセージを追加します。 |
| `OnPostDeleteAllMessagesAsync` | データベース内のすべてのメッセージを削除するには、`DeleteAllMessagesAsync` を呼び出します。 |
| `OnPostDeleteMessageAsync` | `Id` を指定してメッセージを削除するには、`DeleteMessageAsync` を実行します。 |
| `OnPostAnalyzeMessagesAsync` | データベースに 1 つ以上のメッセージが含まれている場合、メッセージあたりの平均単語数を計算します。 |

ページ モデル メソッドは、`IndexPageTests` クラス (*tests/RazorPagesTestSample.Tests/UnitTests/IndexPageTests.cs*) の 7 つのテストを使用してテストされます。 テストでは、おなじみの Arrange-Assert-Act パターンを使用します。 これらのテストでは次に焦点を合わせています。

* [ModelState](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary) が無効な場合に、メソッドが正しい動作に従うかどうかを判断します。
* メソッドによって正しい <xref:Microsoft.AspNetCore.Mvc.IActionResult> が生成されることを確認します。
* プロパティ値の割り当てが正しく行われていることを確認します。

この一連のテストでは、多くの場合に、ページ モデル メソッドが実行される Act ステップ用に予期されるデータを生成するために、DAL のメソッドのモックを作成します。 たとえば、`AppDbContext` の `GetMessagesAsync` メソッドは、出力を生成するためにモックが作成されます。 ページ モデル メソッドでこのメソッドを実行すると、モックによって結果が返されます。 データはデータベースから取得されません。 これにより、ページ モデル テストで DAL を使用するための、予測可能で信頼性の高いテスト条件が作成されます。

`OnGetAsync_PopulatesThePageModel_WithAListOfMessages` テストでは、ページ モデルに対して `GetMessagesAsync` メソッドのモックを作成する方法を示しています。

[!code-csharp[](razor-pages-tests/samples/3.x/tests/RazorPagesTestSample.Tests/UnitTests/IndexPageTests.cs?name=snippet1&highlight=3-4)]

Act ステップで `OnGetAsync` メソッドが実行されると、ページ モデルの `GetMessagesAsync` メソッドが呼び出されます。

単体テストの Act ステップ (*tests/RazorPagesTestSample.Tests/UnitTests/IndexPageTests.cs*):

[!code-csharp[](razor-pages-tests/samples/3.x/tests/RazorPagesTestSample.Tests/UnitTests/IndexPageTests.cs?name=snippet2)]

`IndexPage` ページ モデルの `OnGetAsync` メソッド (*src/RazorPagesTestSample/Pages/Index.cshtml.cs*):

[!code-csharp[](razor-pages-tests/samples/3.x/src/RazorPagesTestSample/Pages/Index.cshtml.cs?name=snippet1&highlight=3)]

DAL の `GetMessagesAsync` メソッドでは、このメソッド呼び出しの結果を返しません。 メソッドのモック バージョンで、結果を返します。

`Assert` ステップでは、実際のメッセージ (`actualMessages`) がページ モデルの `Messages` プロパティから割り当てられます。 メッセージが割り当てられるときに、型チェックも実行されます。 予期されるメッセージと実際のメッセージが、それらの `Text` プロパティによって比較されます。 テストでは、2 つの `List<Message>` インスタンスに同じメッセージが含まれていることをアサートします。

[!code-csharp[](razor-pages-tests/samples/3.x/tests/RazorPagesTestSample.Tests/UnitTests/IndexPageTests.cs?name=snippet3)]

このグループの他のテストでは、<xref:Microsoft.AspNetCore.Http.DefaultHttpContext>、<xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary>、`PageContext` を確立するための <xref:Microsoft.AspNetCore.Mvc.ActionContext>、`ViewDataDictionary`、および `PageContext` を含むページ モデル オブジェクトが作成されます。 これらは、テストの実施に役立ちます。 たとえば、メッセージ アプリでは、`OnPostAddMessageAsync` の実行時に有効な <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageResult> が返されることを確認するために、<xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.AddModelError*> を使用して `ModelState` エラーを作成します。

[!code-csharp[](razor-pages-tests/samples/3.x/tests/RazorPagesTestSample.Tests/UnitTests/IndexPageTests.cs?name=snippet4&highlight=11,26,29,32)]

## <a name="additional-resources"></a>その他の技術情報

* [dotnet テストと xUnit を使用した .NET Core での単体テスト C#](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)
* <xref:mvc/controllers/testing>
* [コードの単体テスト](/visualstudio/test/unit-test-your-code) (Visual Studio)
* <xref:test/integration-tests>
* [xUnit.net](https://xunit.github.io/)
* [Visual Studio for Mac を使用した macOS での完全な .NET Core ソリューションの構築](/dotnet/core/tutorials/using-on-mac-vs-full-solution)
* [XUnit.net の概要:.NET SDK コマンド ラインでの .Net Core の使用](https://xunit.github.io/docs/getting-started-dotnet-core)
* [Moq](https://github.com/moq/moq4)
* [Moq クイック スタート](https://github.com/Moq/moq4/wiki/Quickstart)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core では Razor Pages アプリの単体テストをサポートしています。 データ アクセス層 (DAL) とページ モデルのテストによって、次のことを保証できます。

* Razor Pages アプリの一部は、アプリの構築時に独立して、または 1 つの単位として連携して機能します。
* クラスとメソッドの担当のスコープは限定されています。
* アプリの動作に関する追加のドキュメントがあります。
* コードの更新によって発生したエラーである回帰は、自動ビルドおよびデプロイ時に検出されます。

このトピックでは、Razor Pages アプリと単体テストの基本的な知識があることを前提としています。 Razor Pages アプリまたはテストの概念になじみがない場合は、次のトピックを参照してください。

* <xref:razor-pages/index>
* <xref:tutorials/razor-pages/razor-pages-start>
* [dotnet テストと xUnit を使用した .NET Core での単体テスト C#](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/test/razor-pages-tests/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

サンプル プロジェクトは、次の 2 つのアプリで構成されています。

| アプリ         | プロジェクト フォルダー                     | 説明 |
| ----------- | ---------------------------------- | ----------- |
| メッセージ アプリ | *src/RazorPagesTestSample*         | ユーザーがメッセージの追加、1 つのメッセージの削除、すべてのメッセージの削除、およびメッセージの分析 (メッセージあたりの平均単語数の検出) をできるようにします。 |
| アプリをテストする    | *tests/RazorPagesTestSample.Tests* | メッセージ アプリの DAL およびインデックス ページ モデルの単体テストに使用されます。 |

IDE の組み込みテスト機能 ([Visual Studio](/visualstudio/test/unit-test-your-code) や [Visual Studio for Mac](/dotnet/core/tutorials/using-on-mac-vs-full-solution) など) を使用して、テストを実行できます。 [Visual Studio Code](https://code.visualstudio.com/) またはコマンド ラインを使用する場合は、コマンドプロンプトで *tests/RazorPagesTestSample.Tests* フォルダー内の次のコマンドを実行します。

```dotnetcli
dotnet test
```

## <a name="message-app-organization"></a>メッセージ アプリの構成

メッセージ アプリは、次の特性を持つ Razor Pages メッセージ システムです。

* アプリのインデックス ページ (*Pages/Index. cshtml* と *Pages/Index. cshtml*) には、メッセージの追加、削除、および分析 (メッセージあたりの平均単語数の検出) を制御する UI およびページ モデル メソッドが用意されています。
* メッセージは、`Id` (キー) と `Text` (メッセージ) の 2 つのプロパティを持つ `Message` クラス (*Data/Message.cs*) によって記述されます。 `Text` プロパティは必須であり、200 文字までに制限されています。
* メッセージは [Entity Framework のメモリ内データベース](/ef/core/providers/in-memory/)&#8224; を使用して格納されます。
* アプリでは、データベース コンテキスト クラス `AppDbContext` (*Data/AppDbContext.cs*) 内に DAL を格納します。 DAL メソッドは `virtual` とマークされ、これにより、テストで使用するためのメソッドのモックを作成できます。
* アプリの起動時にデータベースが空の場合、メッセージ ストアが 3 つのメッセージで初期化されます。 これらの *シードされたメッセージ* も、テストで使用されます。

&#8224; EF トピック「[InMemory を使用したテスト](/ef/core/miscellaneous/testing/in-memory)」では、MSTest を使用したテストでメモリ内データベースを使用する方法について説明します。 このトピックでは、[xUnit](https://xunit.github.io/) テスト フレームワークを使用します。 テストの概念とテストの実装は、異なるテスト フレームワークでも類似していますが、同一ではありません。

サンプル アプリではリポジトリ パターンを使用しておらず、[Unit of Work (UoW) パターン](https://martinfowler.com/eaaCatalog/unitOfWork.html)の有効な例ではありませんが、Razor Pages ではこれらの開発パターンをサポートしています。 詳細については、[インフラストラクチャの永続レイヤーの設計](/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)に関するページと「<xref:mvc/controllers/testing>」を参照してください (このサンプルでは、リポジトリ パターンを実装しています)。

## <a name="test-app-organization"></a>テスト アプリの構成

テスト アプリは、*tests/RazorPagesTestSample.Tests* フォルダー内のコンソール アプリです。

| テスト アプリ フォルダー | 説明 |
| --------------- | ----------- |
| *UnitTests*     | <ul><li>*DataAccessLayerTest.cs* には、DAL の単体テストが含まれています。</li><li>*IndexPageTests.cs* には、インデックス ページ モデルの単体テストが含まれています。</li></ul> |
| *Utilities*     | データベースが各テストのベースライン条件にリセットされるように、各 DAL 単体テストの新しいデータベース コンテキスト オプションを作成するために使用する `TestDbContextOptions` メソッドが含まれています。 |

テスト フレームワークは、[xUnit](https://xunit.github.io/) です。 オブジェクトのモック フレームワークは [Moq](https://github.com/moq/moq4) です。

## <a name="unit-tests-of-the-data-access-layer-dal"></a>データアクセス層 (DAL) の単体テスト

メッセージ アプリには、`AppDbContext` クラス (*src/RazorPagesTestSample/Data/AppDbContext.cs*) に含まれる 4 つのメソッドを持つ DAL があります。 テスト アプリで、各メソッドには 1 つか 2 つの単体テストがあります。

| DAL メソッド               | 関数                                                                   |
| ------------------------ | -------------------------------------------------------------------------- |
| `GetMessagesAsync`       | データベースから `Text` プロパティで並べ替えられた `List<Message>` を取得します。 |
| `AddMessageAsync`        | `Message` をデータベースに追加します。                                          |
| `DeleteAllMessagesAsync` | データベースからすべての `Message` エントリを削除します。                           |
| `DeleteMessageAsync`     | `Id` によって、データベースから 1 つの `Message` を削除します。                      |

DAL の単体テストでは、各テストの新しい `AppDbContext` を作成する際に、<xref:Microsoft.EntityFrameworkCore.DbContextOptions> が必要です。 各テストの `DbContextOptions` を作成する 1 つの方法として、<xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder> を使用します。

```csharp
var optionsBuilder = new DbContextOptionsBuilder<AppDbContext>()
    .UseInMemoryDatabase("InMemoryDb");

using (var db = new AppDbContext(optionsBuilder.Options))
{
    // Use the db here in the unit test.
}
```

この方法の問題は、各テストでは、前のテストでデータベースがどのような状態になっていても、それを受け取るということです。 相互に干渉しないアトミック単体テストを作成しようとする場合に、これが問題になる可能性があります。 `AppDbContext` でテストごとに新しいデータベース コンテキストを強制的に使用させるには、新しいサービス プロバイダーに基づいた `DbContextOptions` インスタンスを指定します。 テスト アプリでは、その `Utilities` クラス メソッド `TestDbContextOptions` (*tests/RazorPagesTestSample.Tests/Utilities/Utilities.cs*) を使用してこれを行う方法を示しています。

[!code-csharp[](razor-pages-tests/samples/2.x/tests/RazorPagesTestSample.Tests/Utilities/Utilities.cs?name=snippet1)]

DAL 単体テストで `DbContextOptions` を使用すると、新しいデータベース インスタンスを使用して、各テストをアトミックに実行できます。

```csharp
using (var db = new AppDbContext(Utilities.TestDbContextOptions()))
{
    // Use the db here in the unit test.
}
```

`DataAccessLayerTest` クラスの各テスト メソッド (*UnitTests/DataAccessLayerTest.cs*) は、同様の Arrange-Act-Assert パターンに従います。

1. Arrange (環境構築):データベースがテスト用に構成され、期待される結果が定義されています。
1. Act (実行):テストが実行されます。
1. Assert (動作確認):アサーションによって、テスト結果が成功であるかどうかが判断されます。

たとえば、`DeleteMessageAsync` メソッドは、`Id` によって識別された 1 つのメッセージの削除を担当します (*src/RazorPagesTestSample/Data/AppDbContext.cs*)。

[!code-csharp[](razor-pages-tests/samples/2.x/src/RazorPagesTestSample/Data/AppDbContext.cs?name=snippet4)]

このメソッドには 2 つのテストがあります。 1 つのテストでは、メッセージがデータベースに存在する場合に、このメソッドによってメッセージが削除されます。 他方のメソッドでは、削除対象のメッセージ `Id` が存在しない場合に、データベースが変更されないことをテストします。 `DeleteMessageAsync_MessageIsDeleted_WhenMessageIsFound` メソッドを次に示します。

[!code-csharp[](razor-pages-tests/samples_snapshot/2.x/tests/RazorPagesTestSample.Tests/UnitTests/DataAccessLayerTest.cs?name=snippet1)]

まず、このメソッドでは、Arrange ステップを実行し、Act ステップの準備を行います。 シード処理メッセージが取得され、`seedMessages` で保持されます。 シード処理メッセージはデータベースに保存されます。 `Id` が `1` のメッセージは、削除対象として設定されます。 `DeleteMessageAsync` メソッドが実行されると、予期されるメッセージには、`Id` が `1` のメッセージを除くすべてのメッセージが含まれるはずです。 `expectedMessages` 変数は、この予期される結果を表します。

[!code-csharp[](razor-pages-tests/samples/2.x/tests/RazorPagesTestSample.Tests/UnitTests/DataAccessLayerTest.cs?name=snippet1)]

メソッドは次のように動作します。`DeleteMessageAsync` メソッドは `1`の `recId` を渡して実行されます。

[!code-csharp[](razor-pages-tests/samples/2.x/tests/RazorPagesTestSample.Tests/UnitTests/DataAccessLayerTest.cs?name=snippet2)]

最後に、メソッドはコンテキストから `Messages` を取得し、`expectedMessages` と比較して、2 つが等しいことをアサートします。

[!code-csharp[](razor-pages-tests/samples/2.x/tests/RazorPagesTestSample.Tests/UnitTests/DataAccessLayerTest.cs?name=snippet3)]

2 つの `List<Message>` が同じであることを比較するために:

* メッセージは `Id` 順に並べ替えられます。
* メッセージのペアは、`Text` プロパティで比較されます。

類似のテスト メソッド `DeleteMessageAsync_NoMessageIsDeleted_WhenMessageIsNotFound` では、存在しないメッセージを削除しようとした結果を確認します。 この場合、データベース内の予期されるメッセージは、`DeleteMessageAsync` メソッドが実行された後の実際のメッセージと等しくなるはずです。 データベースのコンテンツへの変更はないはずです。

[!code-csharp[](razor-pages-tests/samples/2.x/tests/RazorPagesTestSample.Tests/UnitTests/DataAccessLayerTest.cs?name=snippet4)]

## <a name="unit-tests-of-the-page-model-methods"></a>ページ モデル メソッドの単体テスト

別の単体テスト セットは、ページ モデル メソッドのテストを担当します。 メッセージ アプリで、インデックス ページ モデルは *src/RazorPagesTestSample/Pages/Index.cshtml.cs* の `IndexModel` クラスにあります。

| ページ モデル メソッド | 関数 |
| ----------------- | -------- |
| `OnGetAsync` | `GetMessagesAsync` メソッドを使用して、UI の DAL からメッセージを取得します。 |
| `OnPostAddMessageAsync` | [ModelState](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary) が有効な場合、`AddMessageAsync` を呼び出して、データベースにメッセージを追加します。 |
| `OnPostDeleteAllMessagesAsync` | データベース内のすべてのメッセージを削除するには、`DeleteAllMessagesAsync` を呼び出します。 |
| `OnPostDeleteMessageAsync` | `Id` を指定してメッセージを削除するには、`DeleteMessageAsync` を実行します。 |
| `OnPostAnalyzeMessagesAsync` | データベースに 1 つ以上のメッセージが含まれている場合、メッセージあたりの平均単語数を計算します。 |

ページ モデル メソッドは、`IndexPageTests` クラス (*tests/RazorPagesTestSample.Tests/UnitTests/IndexPageTests.cs*) の 7 つのテストを使用してテストされます。 テストでは、おなじみの Arrange-Assert-Act パターンを使用します。 これらのテストでは次に焦点を合わせています。

* [ModelState](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary) が無効な場合に、メソッドが正しい動作に従うかどうかを判断します。
* メソッドによって正しい <xref:Microsoft.AspNetCore.Mvc.IActionResult> が生成されることを確認します。
* プロパティ値の割り当てが正しく行われていることを確認します。

この一連のテストでは、多くの場合に、ページ モデル メソッドが実行される Act ステップ用に予期されるデータを生成するために、DAL のメソッドのモックを作成します。 たとえば、`AppDbContext` の `GetMessagesAsync` メソッドは、出力を生成するためにモックが作成されます。 ページ モデル メソッドでこのメソッドを実行すると、モックによって結果が返されます。 データはデータベースから取得されません。 これにより、ページ モデル テストで DAL を使用するための、予測可能で信頼性の高いテスト条件が作成されます。

`OnGetAsync_PopulatesThePageModel_WithAListOfMessages` テストでは、ページ モデルに対して `GetMessagesAsync` メソッドのモックを作成する方法を示しています。

[!code-csharp[](razor-pages-tests/samples/2.x/tests/RazorPagesTestSample.Tests/UnitTests/IndexPageTests.cs?name=snippet1&highlight=3-4)]

Act ステップで `OnGetAsync` メソッドが実行されると、ページ モデルの `GetMessagesAsync` メソッドが呼び出されます。

単体テストの Act ステップ (*tests/RazorPagesTestSample.Tests/UnitTests/IndexPageTests.cs*):

[!code-csharp[](razor-pages-tests/samples/2.x/tests/RazorPagesTestSample.Tests/UnitTests/IndexPageTests.cs?name=snippet2)]

`IndexPage` ページ モデルの `OnGetAsync` メソッド (*src/RazorPagesTestSample/Pages/Index.cshtml.cs*):

[!code-csharp[](razor-pages-tests/samples/2.x/src/RazorPagesTestSample/Pages/Index.cshtml.cs?name=snippet1&highlight=3)]

DAL の `GetMessagesAsync` メソッドでは、このメソッド呼び出しの結果を返しません。 メソッドのモック バージョンで、結果を返します。

`Assert` ステップでは、実際のメッセージ (`actualMessages`) がページ モデルの `Messages` プロパティから割り当てられます。 メッセージが割り当てられるときに、型チェックも実行されます。 予期されるメッセージと実際のメッセージが、それらの `Text` プロパティによって比較されます。 テストでは、2 つの `List<Message>` インスタンスに同じメッセージが含まれていることをアサートします。

[!code-csharp[](razor-pages-tests/samples/2.x/tests/RazorPagesTestSample.Tests/UnitTests/IndexPageTests.cs?name=snippet3)]

このグループの他のテストでは、<xref:Microsoft.AspNetCore.Http.DefaultHttpContext>、<xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary>、`PageContext` を確立するための <xref:Microsoft.AspNetCore.Mvc.ActionContext>、`ViewDataDictionary`、および `PageContext` を含むページ モデル オブジェクトが作成されます。 これらは、テストの実施に役立ちます。 たとえば、メッセージ アプリでは、`OnPostAddMessageAsync` の実行時に有効な <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageResult> が返されることを確認するために、<xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.AddModelError*> を使用して `ModelState` エラーを作成します。

[!code-csharp[](razor-pages-tests/samples/2.x/tests/RazorPagesTestSample.Tests/UnitTests/IndexPageTests.cs?name=snippet4&highlight=11,26,29,32)]

## <a name="additional-resources"></a>その他の技術情報

* [dotnet テストと xUnit を使用した .NET Core での単体テスト C#](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)
* <xref:mvc/controllers/testing>
* [コードの単体テスト](/visualstudio/test/unit-test-your-code) (Visual Studio)
* <xref:test/integration-tests>
* [xUnit.net](https://xunit.github.io/)
* [Visual Studio for Mac を使用した macOS での完全な .NET Core ソリューションの構築](/dotnet/core/tutorials/using-on-mac-vs-full-solution)
* [XUnit.net の概要:.NET SDK コマンド ラインでの .Net Core の使用](https://xunit.github.io/docs/getting-started-dotnet-core)
* [Moq](https://github.com/moq/moq4)
* [Moq クイック スタート](https://github.com/Moq/moq4/wiki/Quickstart)
* [JustMockLite](https://github.com/telerik/JustMockLite):.NET 開発者向けのモック フレームワーク。 ("*Microsoft では保守管理もサポートも行っていません。* ")

::: moniker-end
