---
title: 'チュートリアル: 関連データを更新する - ASP.NET MVC と EF Core'
description: このチュートリアルでは、外部キー フィールドとナビゲーション プロパティを更新することで関連データを更新します。
author: rick-anderson
ms.author: riande
ms.custom: mvc
ms.date: 03/27/2019
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
uid: data/ef-mvc/update-related-data
ms.openlocfilehash: c2fcc3730fa8bb117c5547b6168afa27ee039d52
ms.sourcegitcommit: 7b6781051d341a1daaf46c6a4368fa8a5701db81
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/28/2021
ms.locfileid: "105638745"
---
# <a name="tutorial-update-related-data---aspnet-mvc-with-ef-core"></a>チュートリアル: 関連データを更新する - ASP.NET MVC と EF Core

前のチュートリアルでは、関連データを表示しました。このチュートリアルでは、外部キー フィールドとナビゲーション プロパティを更新することで関連データを更新します。

以下の図は、使用するページの一部を示しています。

![Course/Edit ページ](update-related-data/_static/course-edit.png)

![Instructor ページを編集する](update-related-data/_static/instructor-edit-courses.png)

このチュートリアルでは、次の作業を行いました。

> [!div class="checklist"]
> * Courses ページをカスタマイズする
> * Instructors/Edit ページを追加する
> * Edit ページにコースを追加する
> * Delete ページを更新する
> * オフィスの場所とコースを Create ページに追加する

## <a name="prerequisites"></a>必須コンポーネント

* [関連データの読み取り](read-related-data.md)

## <a name="customize-courses-pages"></a>Courses ページをカスタマイズする

新しい `Course` エンティティが作成されると、既存の部門とのリレーションシップが必要になります。 これを容易にするため、スキャフォールディング コードには、コントローラーのメソッドと、部門を選択するためのドロップダウン リストを含む Create ビューと Edit ビューが含まれます。 ドロップダウン リストは、`Course.DepartmentID` 外部キー プロパティを設定します。これは、`Department` ナビゲーション プロパティを適切な `Department` エンティティとともに読み込むためにすべての Entity Framework で必要です。 このスキャフォールディング コードを使用しますが、エラー処理を追加し、ドロップダウン リストを並べ替えるために少し変更します。

*CoursesController.cs* で、4 つの Create メソッドと Edit メソッドを削除し、次のコードに置き換えます。

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_CreateGet)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_CreatePost)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_EditGet)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_EditPost)]

`Edit` HttpPost メソッドの後に、ドロップダウン リストに部門情報を読み込む新しいメソッドを作成します。

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_Departments)]

`PopulateDepartmentsDropDownList` メソッドはすべての部門を名前で並べ替えたリストを取得し、ドロップダウン リスト用に `SelectList` コレクションを作成し、そのコレクションを `ViewBag` でビューに渡します。 このメソッドは、ドロップダウン リストがレンダリングされるときに選択される項目を指定するためのコード呼び出しを許可する、省略可能な `selectedDepartment` パラメーターを受け取ります。 ビューが名前 "DepartmentID" を `<select>` タグ ヘルパーに渡すと、ヘルパーは `ViewBag` オブジェクト内で "DepartmentID" という名前の `SelectList` を探すようになります。

新しいコースには部門がまだ確立されていないため、HttpGet `Create` メソッドは、選択した項目を設定せずに `PopulateDepartmentsDropDownList` メソッドを呼び出します。

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?highlight=3&name=snippet_CreateGet)]

HttpGet `Edit` メソッドは、編集中のコースに既に割り当てられている部門の ID に基づいて、選択したアイテムを設定します。

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?highlight=15&name=snippet_EditGet)]

`Create` と `Edit` の両方の HttpPost メソッドには、エラーの発生後にページを再表示するときに選択した項目を設定するコードも含まれています。 これにより、エラー メッセージを表示するためにページを再表示するときに、選択されていた部門が選択されたままになることを保証します。

### <a name="add-asnotracking-to-details-and-delete-methods"></a>.AsNoTracking を Details メソッドと Delete メソッドに追加する

Course の Details ページと Delete ページのパフォーマンスを最適化するため、`AsNoTracking` 呼び出しを `Details` メソッドと HttpGet `Delete` メソッドに追加します。

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?highlight=10&name=snippet_Details)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?highlight=10&name=snippet_DeleteGet)]

### <a name="modify-the-course-views"></a>Course ビューを変更する

*Views/Courses/Create.cshtml* で、 **[Department]\(部門\)** ドロップダウン リストに [Select Department]\(部門を選択\) オプションを追加し、キャプションを **[DepartmentID]** から **[Department]\(部門\)** に変更し、検証メッセージを追加します。

[!code-cshtml[](intro/samples/cu/Views/Courses/Create.cshtml?highlight=2-6&range=29-35)]

*Views/Courses/Edit.cshtml* で、*Create.cshtml* で行ったのと同じ変更を [Department]\(部門\) フィールドに加えます。

また、*Views/Courses/Edit.cshtml* で、 **[Title]\(タイトル\)** フィールドの前にコース番号フィールドを追加します。 コース番号は主キーであるため表示されますが、変更することはできません。

[!code-cshtml[](intro/samples/cu/Views/Courses/Edit.cshtml?range=15-18)]

Edit ビューには、コース番号の隠しフィールド (`<input type="hidden">`) が既にあります。 `<label>` タグ ヘルパーを追加しても、ユーザーが **[Edit]** ページで **[保存]** をクリックしたときに、ポストされたデータにコース番号が含まれないため、隠しフィールドの必要性はなくなりません。

*Views/Courses/Delete.cshtml* で、上部にコース番号フィールドを追加し、部門 ID を部門名に変更します。

[!code-cshtml[](intro/samples/cu/Views/Courses/Delete.cshtml?highlight=14-19,36)]

*Views/Courses/Details.cshtml* で、*Delete.cshtml* に行ったのと同じ変更を行います。

### <a name="test-the-course-pages"></a>Course ページをテストする

アプリを実行して、 **[Courses]** タブを選択し、 **[新規作成]** をクリックして新しいコースのデータを入力します。

![Course/Create ページ](update-related-data/_static/course-create.png)

**[作成]** をクリックします。 Courses/Index ページには、リストに追加された新しいコースが表示されます。 Index ページのリストの部門名は、ナビゲーション プロパティから取得され、リレーションシップが正常に確立されていることを示しています。

Courses/Index ページのコースで **[Edit]** をクリックします。

![Course/Edit ページ](update-related-data/_static/course-edit.png)

ページ上のデータを変更し、 **[Save]\(保存\)** をクリックします。 Courses/Index ページには、更新されたコース データが表示されます。

## <a name="add-instructors-edit-page"></a>Instructors/Edit ページを追加する

インストラクター レコードを編集するときに、インストラクターのオフィスの割り当ての更新が必要な場合があります。 `Instructor` エンティティには、`OfficeAssignment` エンティティとの一対ゼロまたは一対一のリレーションシップがあります。これは、コードで次の状況を処理する必要があることを意味します。

* ユーザーが元は値のあったオフィスの割り当てをクリアする場合は、`OfficeAssignment` エンティティを削除する。

* ユーザーが元は空白だったオフィスの割り当ての値を入力する場合は、新しい `OfficeAssignment` エンティティを作成する。

* ユーザーがオフィスの割り当ての値を変更する場合は、既存の `OfficeAssignment` エンティティの値を変更する。

### <a name="update-the-instructors-controller"></a>Instructors コントローラーを更新する

*InstructorsController.cs* で、HttpGet `Edit` メソッド内のコードを、Instructor エンティティの `OfficeAssignment` ナビゲーション プロパティを読み込んで `AsNoTracking` を呼び出すように変更します。

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=8-11&name=snippet_EditGetOA)]

HttpPost `Edit` メソッドを次のコードで置き換え、オフィスの割り当ての更新を処理します。

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_EditPostOA)]

このコードは次のことを行います。

* 署名が HttpGet `Edit` メソッドと同じになっているため、メソッド名を `EditPost` に変更します (`ActionName` 属性は `/Edit/` URL が引き続き使用されることを指定します)。

* `OfficeAssignment` ナビゲーション プロパティの一括読み込みを使用して、現在の `Instructor` エンティティをデータベースから取得します。 これは、HttpGet `Edit` メソッドで行ったのと同じです。

* モデル バインダーからの値を使用して、取得した `Instructor` エンティティを更新します。 `TryUpdateModel` オーバーロードでは、含めたいプロパティを宣言できるようになります。 これにより、[2 番目のチュートリアル](crud.md)で説明したように、過剰ポスティングを防止します。

    <!-- Snippets don't play well with <ul> [!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?range=241-244)] -->

    ```csharp
    if (await TryUpdateModelAsync<Instructor>(
        instructorToUpdate,
        "",
        i => i.FirstMidName, i => i.LastName, i => i.HireDate, i => i.OfficeAssignment))
    ```

* オフィスの場所が空白の場合は、`OfficeAssignment` テーブル内の関連する行が削除されるように、`Instructor.OfficeAssignment` プロパティを null に設定します。

    <!-- Snippets don't play well with <ul>  "intro/samples/cu/Controllers/InstructorsController.cs"} -->

    ```csharp
    if (String.IsNullOrWhiteSpace(instructorToUpdate.OfficeAssignment?.Location))
    {
        instructorToUpdate.OfficeAssignment = null;
    }
    ```

* データベースへの変更を保存します。

### <a name="update-the-instructor-edit-view"></a>Instructors/Edit ビューを更新する

*Views/Instructors/Edit.cshtml* で、オフィスの場所を編集するための新しいフィールドを、 **[Save]\(保存\)** ボタンの直前に追加します。

[!code-cshtml[](intro/samples/cu/Views/Instructors/Edit.cshtml?range=30-34)]

アプリを実行し、 **[Instructors]\(インストラクター\)** タブを選択し、インストラクターで **[Edit]\(編集\)** をクリックします。 **[Office Location]\(オフィスの場所\)** を変更し、 **[Save]\(保存\)** をクリックします。

![Instructor/Edit ページ](update-related-data/_static/instructor-edit-office.png)

## <a name="add-courses-to-edit-page"></a>Edit ページにコースを追加する

インストラクターは、任意の数のコースを担当する場合があります。 次のスクリーン ショットに示すように、チェック ボックスのグループを使用して、コースの割り当てを変更する機能を追加して、Instructor/Edit ページを拡張します。

![コースが表示された Instructor/Edit ページ](update-related-data/_static/instructor-edit-courses.png)

`Course` と `Instructor` のエンティティ間のリレーションシップは、多対多です。 リレーションシップの追加と削除を行うには、`CourseAssignments` 結合エンティティ セットに対してエンティティを追加および削除します。

インストラクターに割り当てられるコースを変更できるようにする UI は、チェック ボックスのグループです。 データベース内のすべてのコースのチェック ボックスが表示され、インストラクターに現在割り当てられているコースが選択されます。 ユーザーは、チェック ボックスをオンまたはオフにしてコースの割り当てを変更できます。 コースの数が非常に多い場合は、ビューにデータを表示する別のメソッドを使用したいと思うかもしれませんが、結合エンティティを操作してリレーションシップを作成または削除するのと同じメソッドを使用します。

### <a name="update-the-instructors-controller"></a>Instructors コントローラーを更新する

チェック ボックスのリストのためにデータをビューに提供するには、ビュー モデル クラスを使用します。

*SchoolViewModels* フォルダー内に *AssignedCourseData.cs* を作成し、既存のコードを次のコードで置き換えます。

[!code-csharp[](intro/samples/cu/Models/SchoolViewModels/AssignedCourseData.cs)]

*InstructorsController.cs* で、HttpGet `Edit` メソッドを次のコードで置き換えます。 変更が強調表示されます。

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=10,17,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36&name=snippet_EditGetCourses)]

このコードは、`Courses` ナビゲーション プロパティに一括読み込みを追加し、新しい `PopulateAssignedCourseData` メソッドを呼び出して、`AssignedCourseData` ビュー モデル クラスを使用してチェック ボックス配列に情報を提供します。

`PopulateAssignedCourseData` メソッド内のコードは、ビュー モデル クラスを使用してコースのリストを読み込むため、すべての `Course` エンティティを読み取ります。 各コースに対し、コードはそのコースがインストラクターの `Courses` ナビゲーション プロパティ内に存在しているかどうかをチェックします。 コースがインストラクターに割り当てられているかどうかをチェックするときに、効率的な参照を作成するため、インストラクターに割り当てられているコースが `HashSet` コレクション内に配置されます。 インストラクターが割り当てられているコースに対し、`Assigned` プロパティが true に設定されます。 ビューは、このプロパティを使用して、どのチェック ボックスを選択済みとして表示する必要があるかを判断します。 最後に、リストは `ViewData` でビューに渡されます。

次に、ユーザーが **[Save]\(保存\)** をクリックしたときに実行されるコードを追加します。 `EditPost` メソッドを次のコードで置き換え、Instructor エンティティの `Courses` ナビゲーション プロパティを更新する新しいメソッドを追加します。

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=1,3,12,13,25,39-40&name=snippet_EditPostCourses)]

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_UpdateCourses&highlight=1-31)]

現在、メソッドの署名は HttpGet `Edit`メソッドとは異なっているため、メソッド名を `EditPost` から `Edit` に戻します。

ビューには Course エンティティのコレクションがないため、モデル バインダーは `CourseAssignments` ナビゲーション プロパティを自動的に更新できません。 モデル バインダーを使用する代わりに、新しい `UpdateInstructorCourses` メソッドで `CourseAssignments` ナビゲーション プロパティを更新します。 そのため、モデル バインドから `CourseAssignments` プロパティを除外する必要があります。 これを行うために、`TryUpdateModel` を呼び出すコードを変更する必要はありません。これは、明示的な承認を必要とするオーバーロードを使用していて、`CourseAssignments` がインクルード リスト内にないためです。

チェック ボックスが選択されていない場合、`UpdateInstructorCourses` のコードは空のコレクションを使用して `CourseAssignments` ナビゲーション プロパティを初期化し、次を返します。

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_UpdateCourses&highlight=3-7)]

その後コードは、データベース内のすべてのコースをループ処理し、各コースを現在インストラクターに割り当てられているコースとビューで選択されているコースを比較してチェックします。 検索を効率化するため、最後の 2 つのコレクションが `HashSet` オブジェクトに格納されます。

コースのチェック ボックスが選択されたが、そのコースが `Instructor.CourseAssignments`ナビゲーション プロパティにない場合、そのコースがナビゲーション プロパティ内のコレクションに追加されます。

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=14-20&name=snippet_UpdateCourses)]

コースのチェック ボックスが選択さていないが、そのコースが `Instructor.CourseAssignments`ナビゲーション プロパティにある場合、そのコースがナビゲーション プロパティから削除されます。

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=21-29&name=snippet_UpdateCourses)]

### <a name="update-the-instructor-views"></a>Instructor ビューを更新する

*Views/Instructors/Edit.cshtml* で、次のコードを **Office** フィールドの `div` 要素の直後、**Save** ボタンの `div` 要素の前に追加することで、チェック ボックスの配列を持つ **Courses** フィールドを追加します。

<a id="notepad"></a>
> [!NOTE]
> Visual Studio にコードを貼り付けると、改行がコードを分割するように変更される場合があります。 貼り付けた後でコードが変化している場合は、Ctrl + Z キーを 1 回押して、自動書式設定を元に戻してください。 これにより、改行がここに示されているように修正されます。 インデントは完璧である必要はありませんが、`@:</tr><tr>`、`@:<td>`、`@:</td>`、および `@:</tr>` の行は、示されているようにそれぞれ 1 行にする必要があります。そうしないと、ランタイム エラーが発生します。 新しいコードのブロックを選択して、Tab キーを 3 回押して、新しいコードと既存のコードを並べます。 この問題は、Visual Studio 2019 で修正されます。

[!code-cshtml[](intro/samples/cu/Views/Instructors/Edit.cshtml?range=35-61)]

このコードは、3 つの列を含む HTML テーブルを作成します。 各列には、チェック ボックスとその後に続くキャプションがあります。キャプションは、コース番号とタイトルから構成されます。 チェック ボックスはすべて同じ名前 ("selectedCourses") を持ち、これらをグループとして扱うようにモデル バインダーに通知します。 各チェック ボックスの value 属性は `CourseID` の値に設定されます。 ページがポストされると、モデル バインダーは、選択されたチェック ボックスの `CourseID` 値のみで構成される配列をコントローラーに渡します。

チェック ボックスが最初にレンダリングされるときに、インストラクターに割り当てられるコースのチェック ボックスが checked 属性を持ち、選択されます (チェック ボックスがオンになった状態で表示されます)。

アプリを実行し、 **[Instructors]\(インストラクター\)** タブを選択し、インストラクターで **[Edit]\(編集\)** をクリックして **Edit** ページを表示します。

![コースが表示された Instructor/Edit ページ](update-related-data/_static/instructor-edit-courses.png)

一部のコース割り当てを変更し、[Save]\(保存\) をクリックします。 行った変更が Index ページに反映されます。

> [!NOTE]
> インストラクター コース データを編集するためにここで採用されている方法は、コースの数が限られている場合にはうまく機能します。 非常に大きいコレクションの場合、別の UI と別の更新方法が必要になる場合があります。

## <a name="update-delete-page"></a>Delete ページを更新する

*InstructorsController.cs* で、`DeleteConfirmed` メソッドを削除し、その場所に次のコードを挿入します。

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=5-7,9-12&name=snippet_DeleteConfirmed)]

このコードにより、次の変更が行われます。

* `CourseAssignments` ナビゲーション プロパティに対して一括読み込みを行います。 これを含める必要があります。そうしないと、EF で関連 `CourseAssignment` エンティティが認識されず、削除されません。 ここでこれらを読み取らなくても済むようにするには、データベースで連鎖削除を構成します。

* 削除されるインストラクターが任意の部門の管理者として割り当てられている場合、インストラクターの割り当てをその部門から削除します。

## <a name="add-office-location-and-courses-to-create-page"></a>オフィスの場所とコースを Create ページに追加する

*InstructorsController.cs* で、HttpPost と HttpGet の `Create` メソッドを削除してから、その場所に次のコードを追加します。

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_Create&highlight=3-5,12,14-22,29)]

このコードは、`Edit` メソッドでご覧になったものと似ていますが、最初にコースが選択されていない点が異なります。 HttpGet `Create` メソッドは、`PopulateAssignedCourseData` メソッドを呼び出します。これはコースが選択されている可能性があるからではなく、ビュー内の `foreach` ループに空のコレクションを提供するためです (そうしないと、コードの表示で null 参照例外がスローされる場合があります)。

HttpPost `Create` メソッドは、選択した各コースを `CourseAssignments` ナビゲーション プロパティに追加してから、検証エラーをチェックし、データベースに新しいインストラクターを追加します。 モデル エラーが発生した (たとえばユーザーが無効な日付をキー指定した) 場合に、エラー メッセージとともにページが再表示され、行ったコースの選択がすべて自動的に復元されるように、コースはモデル エラーが発生しても追加されます。

コースを `CourseAssignments` ナビゲーション プロパティに追加できるようにするには、プロパティを空のコレクションとして初期化する必要があることに注意してください。

```csharp
instructor.CourseAssignments = new List<CourseAssignment>();
```

コントローラー コードでこれを行うための別の方法として、`Instructor` モデルでこれを行うことができます。このためには、プロパティ ゲッターを変更して、コレクションが存在しない場合に自動的に作成するようにします。次の例に示します。

```csharp
private ICollection<CourseAssignment> _courseAssignments;
public ICollection<CourseAssignment> CourseAssignments
{
    get
    {
        return _courseAssignments ?? (_courseAssignments = new List<CourseAssignment>());
    }
    set
    {
        _courseAssignments = value;
    }
}
```

`CourseAssignments` プロパティをこの方法で変更する場合、コントローラー内の明示的なプロパティの初期化コードを削除することができます。

*Views/Instructor/Create.cshtml* で、オフィスの場所のテキスト ボックスとチェック ボックスを [Submit]\(送信\) ボタンの前のコースに追加します。 Edit ページの場合と同様に、[コードを貼り付けたときに Visual Studio がコードを再フォーマットする場合は、書式設定を修正します](#notepad)。

[!code-cshtml[](intro/samples/cu/Views/Instructors/Create.cshtml?range=29-61)]

アプリを実行し、インストラクターを作成して、テストします。

## <a name="handling-transactions"></a>トランザクションの処理

[CRUD チュートリアル](crud.md)で説明したように、Entity Framework はトランザクションを暗黙的に実装します。 たとえば、Entity Framework の外部で行われる操作をトランザクションに含めたい場合など、より詳細な制御が必要なシナリオについては、「[Using Transactions](/ef/core/saving/transactions)」(トランザクションの使用) をご覧ください。

## <a name="get-the-code"></a>コードを取得する

[完成したアプリケーションをダウンロードまたは表示する。](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="next-steps"></a>次の手順

このチュートリアルでは、次の作業を行いました。

> [!div class="checklist"]
> * Courses ページをカスタマイズした
> * Instructors/Edit ページを追加した
> * Edit ページにコースを追加した
> * Delete ページを更新した
> * オフィスの場所とコースを Create ページに追加した

コンカレンシーの競合を処理する方法について学習するには、次のチュートリアルに進んでください。

> [!div class="nextstepaction"]
> [コンカレンシーの競合の処理](concurrency.md)
