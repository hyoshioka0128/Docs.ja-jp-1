---
title: 'チュートリアル: CRUD 機能を実装する - ASP.NET MVC と EF Core'
description: このチュートリアルでは、MVC スキャフォールディングがコントローラーとビュー用に自動的に作成する CRUD (作成、読み取り、更新、削除) コードを確認およびカスタマイズします。
author: rick-anderson
ms.author: riande
ms.custom: mvc
ms.date: 02/04/2019
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
uid: data/ef-mvc/crud
ms.openlocfilehash: c3cfee107541a20a089809193d4bf7fa08730bd2
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586398"
---
# <a name="tutorial-implement-crud-functionality---aspnet-mvc-with-ef-core"></a>チュートリアル: CRUD 機能を実装する - ASP.NET MVC と EF Core

前のチュートリアルでは、Entity Framework と SQL Server LocalDB を使ってデータを保存して表示する MVC アプリケーションを作成しました。 このチュートリアルでは、MVC スキャフォールディングがコントローラーとビュー用に自動的に作成する CRUD (作成、読み取り、更新、削除) コードを確認およびカスタマイズします。

> [!NOTE]
> コントローラーとデータ アクセス層の間に抽象化レイヤーを作成するためにリポジトリ パターンを実装することは、よく行われることです。 この一連のチュートリアルが複雑にならないようにし、Entity Framework 自体の使い方に集中できるように、チュートリアルではリポジトリは使われていません。 EF でのリポジトリについては、[このシリーズの最後のチュートリアル](advanced.md)をご覧ください。

このチュートリアルでは、次の作業を行いました。

> [!div class="checklist"]
> * Details ページをカスタマイズする
> * Create ページを更新する
> * [編集] ページを更新する
> * [削除] ページを更新する
> * データベース接続を閉じる

## <a name="prerequisites"></a>必須コンポーネント

* [EF Core と ASP.NET Core MVC の概要](intro.md)

## <a name="customize-the-details-page"></a>Details ページをカスタマイズする

Students/Index ページのスキャフォールディングされたコードでは、`Enrollments` プロパティが省略されています。これは、このプロパティがコレクションを保持しているためです。 **Details** ページでは、コレクションの内容を HTML テーブルで表示します。

*Controllers/StudentsController.cs* に含まれる Details ビューのアクション メソッドでは、`SingleOrDefaultAsync` メソッドを使って 1 つの `Student` エンティティを取得しています。 `Include`、 `ThenInclude`、および `AsNoTracking` メソッドを呼び出すコードを、次の強調表示された部分で示すように追加します。

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_Details&highlight=8-12)]

`Include` メソッドと `ThenInclude` メソッドにより、コンテキストは `Student.Enrollments` ナビゲーション プロパティと、各登録内の `Enrollment.Course` ナビゲーション プロパティを読み込みます。  これらのメソッドについては、[関連データの読み取り](read-related-data.md)チュートリアルをご覧ください。

`AsNoTracking` メソッドを使うと、返されたエンティティが現在のコンテキストの有効期間内に更新されないシナリオで、パフォーマンスが向上します。 詳細については、このチュートリアルの最後の `AsNoTracking` をご覧ください。

### <a name="route-data"></a>ルート データ

`Details` メソッドに渡すキー値は、"*ルート データ*" から取得します。 ルート データは、モデル バインダーが URL のセグメント内で検出したデータです。 たとえば、既定のルートでは、controller、action、id の各セグメントが指定されています。

[!code-csharp[](intro/samples/cu/Startup.cs?name=snippet_Route&highlight=5)]

次の URL では、既定のルートは、Instructor を controller として、Index を action として、1 を id としてマップします。これらがルート データの値です。

```
http://localhost:1230/Instructor/Index/1?courseID=2021
```

URL の最後の部分 ("?courseID=2021") は、クエリ文字列の値です。 ID をクエリ文字列の値として渡した場合、モデル バインダーは `Index` メソッドの `id` パラメーターにも ID 値を渡します。

```
http://localhost:1230/Instructor/Index?id=1&CourseID=2021
```

Index ページでは、Razor ビューのタグ ヘルパーのステートメントによって、ハイパーリンクの URL が作成されます。 次の Razor コードでは、`id` パラメーターが既定のルートと一致するため、`id` がルート データに追加されます。

```html
<a asp-action="Edit" asp-route-id="@item.ID">Edit</a>
```

これにより、`item.ID` が 6 のときは次の HTML が生成されます。

```html
<a href="/Students/Edit/6">Edit</a>
```

次の Razor コードでは、`studentID` は既定ルートのパラメーターと一致しないため、クエリ文字列として追加されます。

```html
<a asp-action="Edit" asp-route-studentID="@item.ID">Edit</a>
```

これにより、`item.ID` が 6 のときは次の HTML が生成されます。

```html
<a href="/Students/Edit?studentID=6">Edit</a>
```

タグ ヘルパーの詳細については、「<xref:mvc/views/tag-helpers/intro>」を参照してください。

### <a name="add-enrollments-to-the-details-view"></a>Details ビューに登録を追加する

*Views/Students/Details.cshtml* を開きます。 次の例で示すように、`DisplayNameFor` および `DisplayFor` ヘルパーを使って各フィールドが表示されます。

[!code-cshtml[](intro/samples/cu/Views/Students/Details.cshtml?range=13-18&highlight=2,5)]

最後のフィールドの後、終了タグ `</dl>` の直前に、登録の一覧を表示する次のコードを追加します。

[!code-cshtml[](intro/samples/cu/Views/Students/Details.cshtml?range=31-52)]

コードを貼り付けた後でコードのインデントが乱れた場合は、Ctrl + D + K キーを押して修正します。

このコードは、`Enrollments` ナビゲーション プロパティ内のエンティティをループ処理します。 登録ごとに、コースのタイトルとグレードが表示されます。 コース タイトルは、Enrollments エンティティの `Course` ナビゲーション プロパティに格納されている Course エンティティから取得されます。

アプリを実行し、 **[Students]** タブを選んで、受講者の **[Details]** リンクをクリックします。 選んだ受講者のコースとグレードの一覧が表示されます。

![Student/Details ページ](crud/_static/student-details.png)

## <a name="update-the-create-page"></a>Create ページを更新する

*StudentsController.cs* で、HttpPost の `Create` メソッドを変更し、try-catch ブロックを追加して、`Bind` 属性から ID を削除します。

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_Create&highlight=4,6-7,14-21)]

このコードは、ASP.NET Core MVC モデル バインダーによって作成された Student エンティティを Students エンティティ セットに追加した後、変更をデータベースに保存します (モデル バインダーとは、フォームによって送信されたデータの操作を容易にする ASP.NET Core MVC の機能です。モデル バインダーは、ポストされたフォーム値を CLR 型に変換して、パラメーター内のアクション メソッドに渡します。 この例のモデル バインダーは、Form コレクションからのプロパティ値を使って、Student エンティティを自動的にインスタンス化します)。

`ID` は行が挿入されるときに SQL Server によって自動的に設定される主キー値であるため、`Bind` 属性から ID を削除しました。 ユーザーからの入力によって ID 値が設定されることはありません。

`Bind` 属性以外では、スキャフォールディングされたコードに対して行った変更は try-catch ブロックだけです。 変更を保存するときに、`DbUpdateException` から派生した例外がキャッチされた場合は、汎用的なエラー メッセージが表示されます。 `DbUpdateException` 例外は、プログラミング エラーではなくアプリケーション外の何かが原因で発生する場合があるので、再試行することをお勧めします。 このサンプルでは実装されていませんが、運用品質のアプリケーションでは例外をログに記録します。 詳細については、「[Monitoring and Telemetry (Building Real-World Cloud Apps with Azure)](/aspnet/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/monitoring-and-telemetry)」(監視とテレメトリ (Azure での実際のクラウド アプリの構築)) の「**Log for insight**」(洞察のためのログ) セクションをご覧ください。

`ValidateAntiForgeryToken` 属性は、クロスサイト リクエスト フォージェリ (CSRF) 攻撃を防ぐのに役立ちます。 トークンは、[FormTagHelper](xref:mvc/views/working-with-forms#the-form-tag-helper) によってビューに自動的に挿入され、ユーザーがフォームを送信するときに追加されます。 トークンは、`ValidateAntiForgeryToken` 属性によって検証されます。 詳細については、「<xref:security/anti-request-forgery>」を参照してください。

<a id="overpost"></a>

### <a name="security-note-about-overposting"></a>過剰ポスティングに関するセキュリティの注意事項

スキャフォールディングされたコードによって `Create` メソッドに追加される `Bind` 属性は、作成シナリオで過剰ポスティングを防ぐ 1 つの方法です。 たとえば、Student エンティティに、この Web ページで設定したくない `Secret` プロパティが含まれているものとします。

```csharp
public class Student
{
    public int ID { get; set; }
    public string LastName { get; set; }
    public string FirstMidName { get; set; }
    public DateTime EnrollmentDate { get; set; }
    public string Secret { get; set; }
}
```

Web ページに `Secret` フィールドを作らなくても、ハッカーは、Fiddler などのツールを使うか、何らかの JavaScript を作成して、`Secret` フォーム値をポストすることできます。 モデル バインダーが Student インスタンスを作成するときに使うフィールドを制限する `Bind` 属性がないと、モデル バインダーはその `Secret` フォーム値を取得し、それを使って Student インスタンスを作成します。 その場合、`Secret` フォーム フィールドに対してハッカーが指定した値はすべて、データベースで更新されます。 次の図では、Fiddler ツールを使用して、ポストされたフォームの値に `Secret` フィールド (値 "OverPost" を含む) が追加されています。

![Fiddler による Secret フィールドの追加](crud/_static/fiddler.png)

値 "OverPost" は挿入される行の `Secret` プロパティに正常に追加されますが、Web ページがそのプロパティを設定できることは意図したものではありません。

最初にデータベースからエンティティを読み取り、`TryUpdateModel` を呼び出して明示的に許可されたプロパティ リストを渡すことにより、編集シナリオでの過剰ポスティングを防ぐことができます。 これらのチュートリアルではその方法が使われています。

多くの開発者に好まれている、過剰ポスティングを防ぐためのもう 1 つの方法は、エンティティ クラスではなくビュー モデルをモデル バインドで使うことです。 更新するプロパティのみをビュー モデルに含めます。 MVC モデル バインダーが終了したら、必要に応じて AutoMapper などのツールを使って、ビュー モデルのプロパティをエンティティ インスタンスにコピーします。 エンティティのインスタンスで `_context.Entry` を使ってその状態を `Unchanged` に設定した後、ビュー モデルに含まれる各エンティティ プロパティで `Property("PropertyName").IsModified` を true に設定します。 この方法は、編集シナリオと作成シナリオの両方で利用できます。

### <a name="test-the-create-page"></a>Create ページをテストする

*Views/Students/Create.cshtml* 内のコードでは、各フィールドに対して `label`、`input`、`span` (検証メッセージ用) の各タグ ヘルパーが使われています。

アプリを実行し、 **[Students]** タブを選んで、 **[Create New]** をクリックします。

名前と日付を入力します。 お使いのブラウザーでできる場合は、無効な日付を入力してみてください (ブラウザーによっては、日付選択機能が強制的に使われます)。その後、 **[Create]** をクリックしてエラー メッセージを確認します。

![データ検証エラー](crud/_static/date-error.png)

これは、既定で作成されるサーバー側の検証です。後のチュートリアルでは、クライアント側検証用コードも生成する属性を追加する方法を示します。 次の強調表示されたコードは、`Create` メソッドでのモデル検証チェックの部分です。

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_Create&highlight=8)]

日付を有効な値に変更し、 **[Create]** をクリックして、新しい学生が **[Index]** ページに表示されることを確認します。

## <a name="update-the-edit-page"></a>Edit ページを更新する

*StudentController.cs* に含まれる HttpGet の `Edit` メソッド (`HttpPost` 属性がないもの) は、`SingleOrDefaultAsync` メソッドを使って、選ばれた Student エンティティを取得します (`Details` メソッドと同様)。 このメソッドを変更する必要はありません。

### <a name="recommended-httppost-edit-code-read-and-update"></a>推奨される HttpPost Edit のコード:読み取りと更新

HttpPost の Edit アクション メソッドを、次のコードに置き換えます。

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_ReadFirst)]

これらの変更は、過剰ポスティングを防ぐためのセキュリティのベスト プラクティスを実装します。 スキャフォルダーは、`Bind` 属性を生成し、モデル バインダーによって作成されたエンティティを、`Modified` フラグが設定されたエンティティ セットに追加していました。 そのコードは、`Include` パラメーターにリストされていないフィールド内の既存のデータを `Bind` 属性がクリアするため、多くのシナリオでは推奨されません。

新しいコードは、既存のエンティティを読み取り、`TryUpdateModel` を呼び出して、取得されたエンティティのフィールドを、[ポストされたフォーム データでのユーザー入力に基づいて](xref:mvc/models/model-binding)更新します。 Entity Framework の自動変更追跡は、フォーム入力によって変更されるフィールドの `Modified` フラグを設定します。 `SaveChanges` メソッドが呼び出されると、Entity Framework はデータベースの行を更新する SQL ステートメントを作成します。 コンカレンシーの競合は無視され、ユーザーによって更新されたテーブルの列のみが、データベースで更新されます。 (コンカレンシーの競合の処理方法は後のチュートリアルで示します。)

過剰ポスティングを防ぐ方法のベスト プラクティスとしては、 **[編集]** ページによって更新可能にするフィールドを、`TryUpdateModel` パラメーターで宣言します。 (パラメーター リストでフィールドのリストの前にある空の文字列は、フォーム フィールド名で使うプレフィックス用です)。現在、他に保護しているフィールドはありませんが、モデル バインダーでバインドしたいフィールドをリストに入れておくと、後でデータ モデルにフィールドを追加した場合に、ここでフィールドを明示的に追加するまで、自動的にフィールドを保護できます。

これらの変更の結果として、HttpPost の `Edit` メソッドのシグネチャは、HttpGet の `Edit` メソッドと同じになります。したがって、`EditPost` メソッドの名前を変更してあります。

### <a name="alternative-httppost-edit-code-create-and-attach"></a>代わりの HttpPost Edit コード:作成とアタッチ

HttpPost の Edit の推奨されるコードでは、変更された列のみが更新され、モデル バインドに含めたくないプロパティのデータは維持されます。 ただし、読み取り優先アプローチではデータベースの余分な読み取りが必要であり、コンカレンシーの競合を処理するためのコードが複雑になる可能性があります。 代わりの方法としては、モデル バインダーによって作成されたエンティティを EF コンテキストにアタッチし、変更済みとしてマークします (次のコードはオプションのアプローチを示すためだけに掲載してあるので、このコードでプロジェクトを更新しないでください)。

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_CreateAndAttach)]

この方法は、Web ページの UI にエンティティのすべてのフィールドが含まれ、そのどれでも更新できる場合に使うことができます。

スキャフォールディングされたコードは、作成とアタッチの方法を使いますが、`DbUpdateConcurrencyException` 例外のみをキャッチし、404 エラー コードを返します。  例では、すべてのデータベース更新例外をキャッチし、エラー メッセージを表示する方法が示されています。

### <a name="entity-states"></a>エンティティの状態

データベース コンテキストは、メモリ内のエンティティがデータベースの対応する行と同期しているかどうかを追跡しており、この情報により、`SaveChanges` メソッドを呼び出したときの処理が決まります。 たとえば、新しいエンティティを `Add` メソッドに渡すと、そのエンティティの状態は `Added` に設定されます。 その後、`SaveChanges` メソッドを呼び出すと、データベース コンテキストは SQL の INSERT コマンドを発行します。

エンティティは、次のいずれかの状態になる可能性があります。

* `Added`. エンティティはデータベースにまだ存在しません。 `SaveChanges` メソッドは INSERT ステートメントを発行します。

* `Unchanged`. `SaveChanges` メソッドはこのエンティティに対し何も行う必要はありません。 データベースからエンティティを読み取ると、エンティティはこの状態で開始します。

* `Modified`. エンティティのプロパティ値の一部またはすべてが変更されています。 `SaveChanges` メソッドは UPDATE ステートメントを発行します。

* `Deleted`. エンティティには削除のマークが付けられています。 `SaveChanges` メソッドは DELETE ステートメントを発行します。

* `Detached`. エンティティはデータベース コンテキストによって追跡されていません。

デスクトップ アプリケーションにおいて、通常、状態の変更は自動的に設定されます。 エンティティを読み取って一部のプロパティの値を変更すると、 そのエンティティの状態は自動的に `Modified` に変更されます。 その後、`SaveChanges` を呼び出すと、Entity Framework は、変更された実際のプロパティのみを更新する SQL UPDATE ステートメントを生成します。

Web アプリでは、最初にエンティティを読み取って編集されるデータを表示する `DbContext` は、ページがレンダリングされた後で破棄されます。 HttpPost の `Edit` アクション メソッドが呼び出されると、新しい Web 要求が行われ、`DbContext` の新しいインスタンスが作成されます。 その新しいコンテキストでエンティティを再び読み取った場合、デスクトップの処理をシミュレートすることになります。

ただし、余分な読み取り操作を行いたくない場合は、モデル バインダーによって作成されるエンティティ オブジェクトを使う必要があります。  これを行う最も簡単な方法は、前に示した代替 HttpPost Edit コードと同じように、エンティティの状態を Modified に設定します。 その後、`SaveChanges` を呼び出すと、コンテキストには変更されたプロパティを特定する方法がないため、Entity Framework はデータベース行のすべての列を更新します。

読み取り優先アプローチを使わずに、ユーザーが実際に変更したフィールドのみを SQL UPDATE ステートメントで更新したい場合は、コードがさらに複雑になります。 HttpPost `Edit` メソッドが呼び出されたときに元の値を使用できるよう、何らかの方法 (非表示フィールドなど) を使って元の値を保存する必要があります。 その後、元の値を使って Student エンティティを作成し、エンティティの元のバージョンで `Attach` メソッドを呼び出して、エンティティの値を新しい値に更新した後、`SaveChanges` を呼び出します。

### <a name="test-the-edit-page"></a>Edit ページをテストする

アプリを実行し、 **[Students]** タブを選んで、 **[Edit]** ハイパーリンクをクリックします。

![Students/Edit ページ](crud/_static/student-edit.png)

データをいくつか変更し、 **[Save]** をクリックします。 **[Index]** ページが開き、変更したデータが表示されます。

## <a name="update-the-delete-page"></a>削除ページを更新する

*StudentController.cs* に含まれる HttpGet の `Delete` メソッドのテンプレート コードは、`SingleOrDefaultAsync` メソッドを使って、選ばれた Student エンティティを取得します (Details および Edit メソッドと同様です)。 ただし、`SaveChanges` の呼び出しが失敗したときのカスタム エラー メッセージを実装するには、何らかの機能とその対応するビューをこのメソッドに追加します。

更新および作成操作で見たように、削除操作にも 2 つのアクション メソッドが必要です。 GET 要求に応答して呼び出されるメソッドは、ユーザーが削除操作を承認またはキャンセルできるビューを表示します。 ユーザーが操作を承認すると、POST 要求が作成されます。 その場合、HttpPost `Delete` メソッドが呼び出され、そのメソッドが実際の削除操作を実行します。

データベース更新時に発生する可能性のあるエラーを処理するには、try-catch ブロックを HttpPost の `Delete` メソッドに追加します。 エラーが発生した場合、HttpPost Delete メソッドは HttpGet Delete メソッドを呼び出して、エラーが発生したことを示すパラメーターを渡します。 HttpGet Delete メソッドは、確認ページとエラー メッセージを再び表示し、ユーザーがもう一度実行するか取り消すことができるようにします。

HttpGet の `Delete` アクション メソッドを、エラー報告を管理する次のコードに置き換えます。

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_DeleteGet&highlight=1,9,16-21)]

このコードは、変更保存の失敗後にメソッドが呼び出されたかどうかを示す省略可能なパラメーターを受け取ります。 HttpGet `Delete` メソッドが呼び出される前にエラーが発生していない場合、このパラメーターは false に設定されます。 データベース更新エラーに対して HttpPost の `Delete` メソッドがこのメソッドを呼び出した場合、パラメーターは true で、エラー メッセージがビューに渡されます。

### <a name="the-read-first-approach-to-httppost-delete"></a>HttpPost Delete の読み取り優先アプローチ

HttpPost の `Delete` アクション メソッド (名前は `DeleteConfirmed`) を、次のコードに置き換えます。このコードは、実際の削除操作を実行して、データベース更新エラーをキャッチします。

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_DeleteWithReadFirst&highlight=6-9,11-12,16-21)]

このコードは、選択されたエンティティを取得した後、`Remove` メソッドを呼び出して、エンティティの状態を `Deleted` に設定します。 `SaveChanges` が呼び出されると、SQL DELETE コマンドが生成されます。

### <a name="the-create-and-attach-approach-to-httppost-delete"></a>HttpPost Delete の作成とアタッチ アプローチ

大規模なアプリケーションでのパフォーマンス向上が優先される場合は、主キーの値のみを使って Student エンティティをインスタンス化し、エンティティの状態を `Deleted` に設定することによって、不必要な SQL クエリが実行されないようにすることができます。 エンティティを削除するために Entity Framework に必要なものは主キーの値だけです (このコードは、代替手段の説明のためにのみ示してあるので、プロジェクトに追加しないでください)。

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_DeleteWithoutReadFirst&highlight=7-8)]

エンティティに関連するデータも削除する必要がある場合は、連鎖削除をデータベースで構成します。 このエンティティ削除方法では、削除する関連エンティティがあることを EF が認識しない可能性があります。

### <a name="update-the-delete-view"></a>Delete ビューを更新する

*Views/Student/Delete.cshtml* で、次の例に示すように、h2 見出しと h3 見出しの間にエラー メッセージを追加します。

[!code-cshtml[](intro/samples/cu/Views/Students/Delete.cshtml?range=7-9&highlight=2)]

アプリを実行し、 **[Students]** タブを選んで、 **[Delete]** ハイパーリンクをクリックします。

![削除確認ページ](crud/_static/student-delete.png)

**[Delete]** をクリックします。 削除された学生を含まない [Index] ページが表示されます (アクションでのエラー処理コードの例は、コンカレンシーチュートリアルをご覧ください。)

## <a name="close-database-connections"></a>データベース接続を閉じる

データベース接続が保持しているリソースを解放するには、使い終わったコンテキスト インスタンスをできるだけ早く破棄する必要があります。 ASP.NET Core に組み込まれている[依存関係の挿入](../../fundamentals/dependency-injection.md)が、そのタスクを自動的に行います。

*Startup.cs* で、[AddDbContext 拡張メソッド](https://github.com/aspnet/EntityFrameworkCore/blob/03bcb5122e3f577a84498545fcf130ba79a3d987/src/Microsoft.EntityFrameworkCore/EntityFrameworkServiceCollectionExtensions.cs)を呼び出して、ASP.NET Core DI コンテナー内の `DbContext` クラスをプロビジョニングします。 このメソッドは、サービスの有効期間を既定で `Scoped` に設定します。 `Scoped` はコンテキスト オブジェクトの有効期間が Web 要求の有効期間と一致することを意味し、Web 要求の最後に `Dispose` メソッドが自動的に呼び出されます。

## <a name="handle-transactions"></a>トランザクションを処理する

既定では、Entity Framework はトランザクションを暗黙的に実装します。 複数の行またはテーブルを変更してから `SaveChanges` を呼び出すシナリオでは、Entity Framework によって自動的に、すべての変更が成功するか、またはすべての変更が失敗することが保証されます。 一部の変更が完了した後でエラーが発生した場合、それらの変更は自動的にロールバックされます。 たとえば、Entity Framework の外部で行われる操作をトランザクションに含めたい場合など、より詳細な制御が必要なシナリオについては、「[Using Transactions](/ef/core/saving/transactions)」(トランザクションの使用) をご覧ください。

## <a name="no-tracking-queries"></a>追跡なしのクエリ

データベース コンテキストは、テーブルの行を取得してそれらを表すエンティティ オブジェクトを作成するとき、既定では、メモリ内のエンティティがデータベース内のデータと同期されているかどうかを追跡します。 メモリ内のデータはキャッシュとして機能し、エンティティを更新するときに使われます。 Web アプリケーションでは、一般にコンテキスト インスタンスの存続期間は短く (要求ごとに新しいインスタンスが作成されて破棄されます)、通常、エンティティを読み取るコンテキストはエンティティが再び使われる前に破棄されるので、多くの場合、このようなキャッシュは必要ありません。

`AsNoTracking` メソッドを呼び出すことで、メモリ内のエンティティ オブジェクトの追跡を無効にすることができます。 追跡を無効にした方がよい一般的なシナリオを以下に示します。

* コンテキストの有効期間中に、どのエンティティも更新する必要がなく、EF が[個別のクエリによって取得されるエンティティと共にナビゲーション プロパティを自動的に読み込む](read-related-data.md)必要がない場合。 コントローラーの HttpGet アクション メソッドでは、このような状況が頻繁に発生します。

* 大量のデータを取得するクエリを実行していて、返されるデータの小さな部分だけが更新される場合。 大規模なクエリでは、追跡をオフにして、更新が必要な少数のエンティティに対して後でクエリを実行する方が、効率的な場合があります。

* エンティティを更新するためにエンティティをアタッチしたいが、それより前に別の目的で同じエンティティを取得してある場合。 エンティティはデータベース コンテキストによって既に追跡されているため、変更するエンティティをアタッチできません。 このような状況に対処する方法の 1 つは、前のクエリで `AsNoTracking` を呼び出すことです。

詳細については、「[Tracking vs.No-Tracking Queries](/ef/core/querying/tracking)」(追跡ありクエリと追跡なしクエリ) をご覧ください。

## <a name="get-the-code"></a>コードを取得する

[完成したアプリケーションをダウンロードまたは表示する。](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="next-steps"></a>次の手順

このチュートリアルでは、次の作業を行いました。

> [!div class="checklist"]
> * Details ページをカスタマイズした
> * Create ページを更新した
> * Edit ページを更新した
> * Delete ページを更新した
> * データベース接続を閉じた

並べ替え、フィルター処理、ページングを追加して **Index** ページの機能を拡張する方法について学習するには、次のチュートリアルに進んでください。

> [!div class="nextstepaction"]
> [次へ: 並べ替え、フィルター処理、ページング](sort-filter-page.md)
