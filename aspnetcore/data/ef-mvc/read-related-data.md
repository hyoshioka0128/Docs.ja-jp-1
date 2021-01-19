---
title: 'チュートリアル: 関連データを読み取る - ASP.NET MVC と EF Core'
description: このチュートリアルでは、関連データ (Entity Framework がナビゲーション プロパティに読み込むデータ) の読み取りと表示を行います。
author: rick-anderson
ms.author: riande
ms.date: 09/28/2019
ms.topic: tutorial
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
uid: data/ef-mvc/read-related-data
ms.openlocfilehash: b1adca435b48db22a176d33a216c07d6647a6695
ms.sourcegitcommit: 97243663fd46c721660e77ef652fe2190a461f81
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2021
ms.locfileid: "98058312"
---
# <a name="tutorial-read-related-data---aspnet-mvc-with-ef-core"></a>チュートリアル: 関連データを読み取る - ASP.NET MVC と EF Core

前のチュートリアルでは、School データ モデルを作成しました。 このチュートリアルでは、関連データ (Entity Framework がナビゲーション プロパティに読み込むデータ) の読み取りと表示を行います。

以下の図は、使用するページを示しています。

![Courses/Index ページ](read-related-data/_static/courses-index.png)

![Instructors/Index ページ](read-related-data/_static/instructors-index.png)

このチュートリアルでは、次の作業を行いました。

> [!div class="checklist"]
> * 関連データを読み込む方法を学習する
> * Courses ページを作成する
> * Instructors ページを作成する
> * 明示的読み込みについて学習する

## <a name="prerequisites"></a>必須コンポーネント

* [複合データ モデルの作成](complex-data-model.md)

## <a name="learn-how-to-load-related-data"></a>関連データを読み込む方法を学習する

オブジェクト リレーショナル マッピング (ORM) ソフトウェア (Entity Framework など) では、次のように関連データをエンティティのナビゲーション プロパティに読み込むことができる方法がいくつかあります。

* 一括読み込み。 エンティティが読み取られるときに、関連データがエンティティと共に取得されます。 これは通常、必要なデータをすべて取得する 1 つの結合クエリになります。 `Include` メソッドと `ThenInclude` メソッドを使用して、Entity Framework Core で一括読み込みを指定します。

  ![一括読み込みの例](read-related-data/_static/eager-loading.png)

  分離したクエリのデータの一部を取得することができ、EF によってナビゲーション プロパティが "修正されます"。  つまり、EF で、前に取得したエンティティのナビゲーション プロパティに属する、個別に取得したエンティティを自動的に追加します。 関連データを取得するクエリの場合、リストやオブジェクトを返すメソッド (`ToList`、`Single` など) の代わりに、`Load` メソッドを使用できます。

  ![分離したクエリの例](read-related-data/_static/separate-queries.png)

* 明示的読み込み。 エンティティが最初に読み込まれるときに、関連データは取得されません。 必要な場合に、関連データを取得するコードを記述します。 分離したクエリによる一括読み込みのように、明示的読み込みでは、複数のクエリがデータベースに送信されます。 明示的読み込みを使用すると、コードで読み込まれるナビゲーション プロパティを指定する点が異なります。 Entity Framework Core 1.1 では、`Load` メソッドを使用して、明示的読み込みを実行できます。 次に例を示します。

  ![明示的読み込みの例](read-related-data/_static/explicit-loading.png)

* 遅延読み込み。 エンティティが最初に読み込まれるときに、関連データは取得されません。 ただし、ナビゲーション プロパティに初めてアクセスしようとすると、そのナビゲーション プロパティに必要なデータが自動的に取得されます。 初めてナビゲーション プロパティからデータを取得しようとするたびに、クエリがデータベースに送信されます。 Entity Framework Core 1.0 では、遅延読み込みはサポートされません。

### <a name="performance-considerations"></a>パフォーマンスに関する考慮事項

取得したすべてのエンティティの関連データが必要な場合は、通常、データベースに送信された 1 つのクエリの方が、取得した各エンティティに対する分離したクエリよりも効率的なため、一括読み込みを使用すると、より最適なパフォーマンスが得られます。 たとえば、各部門に関連コースが 10 個あるとします。 すべての関連データの一括読み込みは、1 つの (結合) クエリと 1 回のデータベースとのラウンド トリップだけになります。 部門ごとのコースへの分離したクエリでは、データベースとのラウンド トリップが 11 回行われることになります。 データベースとの余分なラウンド トリップは、特に待ち時間が長いときのパフォーマンスに悪影響をもたらします。

その一方で、一部のシナリオでは、分離したクエリがより効率的です。 1 つのクエリ内のすべての関連データの一括読み込みでは、SQL Server で効率的に処理できない、複雑な結合が生成される場合があります。 または、処理しているエンティティのセットのサブセットのためだけにエンティティのナビゲーション プロパティにアクセスする必要がある場合、事前のすべてを取得する一括読み込みでは、必要以上にデータを取得するため、分離したクエリの方が適切に実行される可能性があります。 パフォーマンスが重要な場合、最適な選択を行うために、両方の方法でパフォーマンスをテストすることをお勧めします。

## <a name="create-a-courses-page"></a>Courses ページを作成する

Course エンティティには、コースが割り当てられている部門の Dpartment エンティティを含む、ナビゲーション プロパティが含まれます。 コースのリストに割り当てられた部門の名前を表示するには、`Course.Department` ナビゲーション プロパティにある Department エンティティから Name プロパティを取得する必要があります。

次の図に示すように、Students コントローラーに対して前に実行した **Entity Framework のスキャフォールディング機能を使用して、ビューと共に MVC コントローラー** に同じオプションを使用して、Course エンティティ型の CoursesController という名前のコントローラーを作成します。

![Courses コントローラーを追加する](read-related-data/_static/add-courses-controller.png)

*CoursesController.cs* を開いて、`Index` メソッドを調べます。 自動スキャフォールディングでは、`Include` メソッドを使って、`Department` ナビゲーション プロパティに一括読み込みを指定しています。

`Index` メソッドを、Course エンティティ (`schoolContext` の代わりに `courses`) を返す `IQueryable` により適切な名前を使用する次のコードに置き換えます。

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_RevisedIndexMethod)]

*Views/Courses/Index.cshtml* を開いて、テンプレート コードを次のコードに置き換えます。 変更が強調表示されています。

[!code-cshtml[](intro/samples/cu/Views/Courses/Index.cshtml?highlight=4,7,15-17,34-36,44)]

スキャフォールディング コードに、次の変更を行いました。

* 見出しが Index から Courses に変更されました。

* `CourseID` プロパティ値を示す **Number** 列が追加されました。 既定では、主キーは、通常、エンド ユーザーにとって意味がないため、スキャフォールディングされません。 ただし、このケースでは、主キーは意味があり、表示する必要があります。

* 部門名が表示されるように、**Department** 列を変更しました。 コードは、`Department` ナビゲーション プロパティに読み込まれる Department エンティティの `Name` プロパティを表示します。

  ```html
  @Html.DisplayFor(modelItem => item.Department.Name)
  ```

アプリを実行し、 **[Courses]** タブを選択して部門名のリストを表示します。

![Courses/Index ページ](read-related-data/_static/courses-index.png)

## <a name="create-an-instructors-page"></a>Instructors ページを作成する

このセクションでは、Instructors ページを表示するために、Instructor エンティティのコントローラーとビューを作成します。

![Instructors/Index ページ](read-related-data/_static/instructors-index.png)

このページは、次の方法で関連データを読み取って表示します。

* インストラクターのリストには、OfficeAssignment エンティティからの関連データが表示されます。 Instructor エンティティと OfficeAssignment エンティティは、一対ゼロまたは一対一リレーションシップです。 OfficeAssignment エンティティに一括読み込みを使用します。 前述のように、通常、一括読み込みは、主テーブルで取得したすべての行の関連データが必要なときにより効率的です。 このケースでは、割り当てられたすべてのインストラクターのオフィスの割り当てを表示する必要があります。

* ユーザーがインストラクターを選択すると、関連する Course エンティティが表示されます。 Instructor エンティティと Course エンティティは、多対多リレーションシップです。 Course エンティティとそれに関連する Department エンティティに一括読み込みを使用します。 このケースでは、選択したインストラクターのコースのみが必要なため、分離したクエリの方が効率的な可能性があります。 ただし、この例では、ナビゲーション プロパティにあるエンティティ内のナビゲーション プロパティに一括読み込みを使用する方法を示します。

* ユーザーがコースを選択すると、Enrollments エンティティ セットからの関連データが表示されます。 Course エンティティと Enrollment エンティティは、一対多リレーションシップです。 Enrollment エンティティとそれに関連する Student エンティティに分離したクエリを使用します。

### <a name="create-a-view-model-for-the-instructor-index-view"></a>Instructor インデックス ビューのビュー モデルを作成する

Instructors ページには、3 つの異なるテーブルからデータが表示されます。 そのため、テーブルの 1 つにデータを保持するごとに、3 つのプロパティを含むビュー モデルを作成します。

*SchoolViewModels* フォルダー内に *InstructorIndexData.cs* を作成し、既存のコードを次のコードで置き換えます。

[!code-csharp[](intro/samples/cu/Models/SchoolViewModels/InstructorIndexData.cs)]

### <a name="create-the-instructor-controller-and-views"></a>Instructor コントローラーとビューを作成する

次の図に示すように、EF の読み取り/書き込みアクションで、Instructors コントローラーを作成します。

![Instructors コントローラーを追加する](read-related-data/_static/add-instructors-controller.png)

*InstructorsController.cs* を開いて、ViewModels 名前空間に対する using ステートメントを追加します。

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_Using)]

Index メソッドを次のコードに置き換えて、関連データの一括読み込みを行い、ビュー モデルに配置します。

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_EagerLoading)]

メソッドでは、選択したインストラクターと選択したコースの ID 値を指定する、オプションのルート データ (`id`) とクエリ文字列パラメーター (`courseID`) を受け入れます。 パラメーターは、ページの **Select** ハイパーリンクによって指定されます。

このコードは、ビュー モデルのインスタンスを作成し、インストラクターのリストに配置することから始めます。 コードでは、`Instructor.OfficeAssignment` と `Instructor.CourseAssignments` ナビゲーション プロパティに一括読み込みを指定します。 `CourseAssignments` プロパティ内で `Course` プロパティが読み込まれ、そのプロパティ内で `Enrollments` と `Department` プロパティが読み込まれ、各 `Enrollment` エンティティ内で `Student` プロパティが読み込まれます。

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ThenInclude)]

ビューには常に OfficeAssignment エンティティが必要なため、同じクエリでフェッチする方が効率的です。 インストラクターが Web ページで選択されたときに、Course エンティティが必要なため、1 つのクエリが複数のクエリよりも適しているのは、ページに選択したコースを含めないよりも、含めて表示することの方が多い場合のみです。

`Course` から 2 つのプロパティが必要なため、コードでは `CourseAssignments` と `Course` を繰り返します。 `ThenInclude` 呼び出しの最初の文字列では、`CourseAssignment.Course`、`Course.Enrollments`、および `Enrollment.Student` を取得します。

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ThenInclude&highlight=3-6)]

コードのその時点で、もう 1 つの `ThenInclude` は、必要としない `Student` のナビゲーション プロパティ用になります。 ただし、`Include` を呼び出すと、`Instructor` プロパティを使ってやり直されるため、もう一度チェーンを順に移動する必要があります。今回は `Course.Enrollments` の代わりに `Course.Department` を指定しています。

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ThenInclude&highlight=7-9)]

次のコードは、インストラクターが選択されたときに実行されます。 選択されたインストラクターがビュー モデルのインストラクターのリストから取得されます。 ビュー モデルの `Courses` プロパティが Course エンティティと共にそのインストラクターの `CourseAssignments` ナビゲーション プロパティから読み込まれます。

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?range=56-62)]

`Where` メソッドはコレクションを返しますが、このケースでは、そのメソッドに渡された条件は、返されている Instructor エンティティの 1 つのみになります。 `Single` メソッドでは、コレクションがエンティティの `CourseAssignments` プロパティへのアクセス権を付与する、1 つの Instructor エンティティに変換されます。 `CourseAssignments` プロパティには、関連する `Course` エンティティのみを必要とする、`CourseAssignment` エンティティが含まれます。

コレクションに項目が 1 つのみであることがわかっている場合、コレクションで `Single` メソッドを使用します。 コレクションが空になる場合、または複数の項目がある場合、Single メソッドは例外をスローします。 代わりに、コレクションが空の場合に既定値 (この場合は null) を返す `SingleOrDefault` を使用します。 ただし、(null 参照で `Courses` プロパティを見つけようとして) 引き続き例外となる場合は、例外メッセージでは問題の原因があまり明確に示されません。 `Single` メソッドを呼び出す場合、個別に `Where` メソッドを呼び出す代わりに、Where 条件で渡すこともできます。

```csharp
.Single(i => i.ID == id.Value)
```

これは次のコードの代わりに使用します。

```csharp
.Where(i => i.ID == id.Value).Single()
```

次に、コースが選択された場合、選択したコースはビュー モデルのコースのリストから取得されます。 次に、ビュー モデルの `Enrollments` プロパティが Enrollment エンティティと共にそのコースの `Enrollments` ナビゲーション プロパティから読み込まれます。

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?range=64-69)]

### <a name="modify-the-instructor-index-view"></a>Instructor インデックス ビューを変更する

*Views/Instructors/Index.cshtml* で、テンプレート コードを次のコードに置き換えます。 変更が強調表示されています。

::: moniker range=">= aspnetcore-2.2"

[!code-cshtml[](intro/samples/5cu-snap/Views/Instructors/Index.cshtml?range=1-62&highlight=1,3-7,15-19,24,26-31,41-52,54)]

::: moniker-end

::: moniker range="<= aspnetcore-2.1"

[!code-cshtml[](intro/samples/cu/Views/Instructors/Index1.cshtml?range=1-62&highlight=1,3-7,15-19,24,26-31,41-52,54)]

::: moniker-end

既存のコードに次の変更を行いました。

* モデル クラスが `InstructorIndexData` に変更されました。

* **Index** のページ タイトルが **Instructors** に変更されました。

* `item.OfficeAssignment` が null ではない場合にのみ `item.OfficeAssignment.Location` を表示する **Office** 列を追加しました。 (これは、一対ゼロまたは一対一のリレーションシップであるため、関連する OfficeAssignment エンティティがない場合があります。)

  ```cshtml
  @if (item.OfficeAssignment != null)
  {
      @item.OfficeAssignment.Location
  }
  ```

* インストラクターごとに担当したコースを表示する **Courses** 列を追加しました。 詳細については、Razor 構文記事の「[明示的な行の遷移](xref:mvc/views/razor#explicit-line-transition)」セクションを参照してください。

* 選択したインストラクターの `tr` 要素にブートストラップ CSS クラスを条件付きで追加するコードを追加しました。 このクラスにより、選択した行の背景色が設定されます。

* `Index` メソッドに送信される選択されたインストラクターの ID を発生させる、各行の他のリンクの直前に **Select** というラベルの新しいハイパーリンクを追加しました。

  ```cshtml
  <a asp-action="Index" asp-route-id="@item.ID">Select</a> |
  ```

アプリを実行し、 **[Instructors]** タブを選択します。関連する OfficeAssignment エンティティがない場合は、ページに関連する OfficeAssignment エンティティの Location プロパティと空の表のセルが表示されます。

![何も選択されていない Instructors/Index ページ](read-related-data/_static/instructors-index-no-selection.png)

*Views/Instructors/Index.cshtml* ファイルでは、テーブル要素を閉じた後に (ファイルの終わりに)、次のコードを追加します。 このコードでは、インストラクターが選択されたときに、インストラクターに関連するコースのリストを表示します。

[!code-cshtml[](intro/samples/cu/Views/Instructors/Index1.cshtml?range=66-101)]

このコードでは、ビュー モデルの `Courses` プロパティを読み取り、コースのリストを表示します。 また、選択したコースの ID を `Index` アクション メソッドに送信する、**Select** ハイパーリンクも指定します。

ページを更新し、インストラクターを選択します。 選択したインストラクターに割り当てられたコースを表示するグリッドを表示し、各コースに割り当てられた部門の名前を表示します。

![インストラクターが選択された Instructors/Index ページ](read-related-data/_static/instructors-index-instructor-selected.png)

追加したコード ブロックの後に、次のコードを追加します。 このコードは、コースを選択したときに、コースに登録されている受講者のリストを表示します。

[!code-cshtml[](intro/samples/cu/Views/Instructors/Index1.cshtml?range=103-125)]

このコードは、コースに登録された受講生のリストを表示するために、ビュー モデルの Enrollments プロパティを読み取ります。

もう一度ページを更新し、インストラクターを選択します。 次に、コースを選択して、登録済みの受講者とその成績のリストを表示します。

![インストラクターとコースが選択された Instructors/Index ページ](read-related-data/_static/instructors-index.png)

## <a name="about-explicit-loading"></a>明示的読み込みについて

*InstructorsController.cs* でインストラクターのリストを取得したときに、`CourseAssignments` ナビゲーション プロパティに一括読み込みを指定しました。

ユーザーは選択したインストラクターとコースの登録内容をほとんど表示する必要がないとします。 その場合は、要求された場合にのみ、登録データを読み取る必要がある可能性があります。 明示的読み込みを行う方法の例を表示するには、`Index` メソッドを次のコードに置き換えます。このコードは、Enrollments の一括読み込みを削除して、そのプロパティを明示的に読み込みます。 コードの変更が強調表示されています。

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ExplicitLoading&highlight=23-29)]

新しいコードでは、インストラクター エンティティを取得するコードから登録データを呼び出す *ThenInclude* メソッドを削除します。 `AsNoTracking` も削除されます。  インストラクターとコースが選択された場合、強調表示されたコードで選択されたコードの Enrollment エンティティ、および各 Enrollment の Student エンティティを取得します。

アプリを実行して、Instructors/Index ページに移動すると、データを取得する方法を変更しているにもかかわらず、ページ上で表示される内容に変わりがないことがわかります。

## <a name="get-the-code"></a>コードを取得する

[完成したアプリケーションをダウンロードまたは表示する。](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="next-steps"></a>次の手順

このチュートリアルでは、次の作業を行いました。

> [!div class="checklist"]
> * 関連データを読み込む方法を学習した
> * Courses ページを作成した
> * Instructors ページを作成した
> * 明示的読み込みについて学習した

関連データを更新する方法について学習するには、次のチュートリアルに進んでください。

> [!div class="nextstepaction"]
> [関連データの更新](update-related-data.md)
