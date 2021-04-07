---
title: 'チュートリアル: コンカレンシーの処理 - ASP.NET MVC と EF Core'
description: このチュートリアルでは、複数のユーザーが同じエンティティを同時に更新するときの競合の処理方法について説明します。
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
uid: data/ef-mvc/concurrency
ms.openlocfilehash: 013527216adb799a32370a85e048fbb06792338d
ms.sourcegitcommit: 7b6781051d341a1daaf46c6a4368fa8a5701db81
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/28/2021
ms.locfileid: "105638810"
---
# <a name="tutorial-handle-concurrency---aspnet-mvc-with-ef-core"></a>チュートリアル: コンカレンシーの処理 - ASP.NET MVC と EF Core

先のチュートリアルでは、データを更新する方法について学習しました。 このチュートリアルでは、複数のユーザーが同じエンティティを同時に更新するときの競合の処理方法について説明します。

`Department` エンティティを使用する Web ページを作成し、コンカレンシー エラーを処理します。 次の図は Edit ページと Delete ページのものです。コンカレンシーで競合が発生すると、メッセージが表示されます。

![Department Edit ページ](concurrency/_static/edit-error.png)

![Department Delete ページ](concurrency/_static/delete-error.png)

このチュートリアルでは、次の作業を行いました。

> [!div class="checklist"]
> * コンカレンシーの競合について学習する
> * トラッキング プロパティを追加する
> * Departments のコントローラーとビューを作成する
> * Index ビューを更新する
> * Edit メソッドを更新する
> * Edit ビューを更新する
> * コンカレンシーの競合をテストする
> * [削除] ページを更新する
> * Details ビューと Create ビューの更新

## <a name="prerequisites"></a>必須コンポーネント

* [関連データの更新](update-related-data.md)

## <a name="concurrency-conflicts"></a>コンカレンシーの競合

あるユーザーがあるエンティティのデータを編集目的で表示したとき、別のユーザーが同じエンティティのデータを最初のユーザーの変更がデータベースに書き込まれる前に更新すると、コンカレンシーの競合が発生します。 このような競合の検出を有効にしないと、最後にデータベースを更新したユーザーが他のユーザーの変更を上書きすることになります。 多くのアプリケーションでは、このリスクが許容されています。ユーザーや更新がわずかであれば、あるいは変更が一部上書きされても大きな問題なければ、コンカレンシーのプログラミングにかかるコストが利点よりも重視されることがあります。 その場合、コンカレンシーの競合を処理するようにアプリケーションを構成する必要はありません。

### <a name="pessimistic-concurrency-locking"></a>ペシミスティック コンカレンシー (ロック)

コンカレンシーで偶発的にデータが失われる事態をアプリケーションで回避する必要があれば、その方法としてデータベース ロックがあります。 これはペシミスティック コンカレンシーと呼ばれています。 たとえば、データベースから行を読む前に、読み取り専用か更新アクセスでロックを要求します。 更新アクセスで行をロックすると、他のユーザーはその行を読み取り専用または更新アクセスでロックできなくなります。変更中のデータのコピーが与えられるためです。 読み取り専用で行をロックすると、他のユーザーはその行を読み取り専用でロックできますが、更新アクセスではロックできません。

ロックの利用には短所があります。 プログラムが複雑になります。 相当なデータベース管理リソースが必要になります。アプリケーションの利用者数が増えると、パフォーマンス上の問題を引き起こすことがあります。 そのような理由から、一部のデータベース管理システムはペシミスティック コンカレンシーに対応していません。 Entity Framework Core にも組み込まれておらず、このチュートリアルでは実装方法を説明しません。

### <a name="optimistic-concurrency"></a>オプティミスティック コンカレンシー

ペシミスティック コンカレンシーの代わりとなるのがオプティミスティック コンカレンシーです。 オプティミスティック コンカレンシーでは、コンカレンシーの競合の発生を許し、発生したら適切に対処します。 たとえば、Jane が Department Edit ページにアクセスし、English 部署の予算を $350,000.00 から $0.00 に変更するとします。

![予算を 0 に変更する](concurrency/_static/change-budget.png)

Jane が **[保存]** をクリックする前に John が同じページにアクセスし、[開始日] フィールドを 9/1/2007 から 9/1/2013 に変更します。

![開始日を 2013 に変更する](concurrency/_static/change-date.png)

Jane が **[保存]** を先にクリックすると、ブラウザーが Index ページに戻ったとき、Jane の変更が反映されています。

![予算が 0 に変更された](concurrency/_static/budget-zero.png)

次に John が Edit ページの **[保存]** をクリックします。このとき、予算は $350,000.00 と表示されています。 この後の動作は、コンカレンシーの競合の処理方法によって決定します。

次のようなオプションがあります。

* ユーザーが変更したプロパティを追跡記録し、それに該当する列だけをデータベースで更新できます。

     例のシナリオでは、2 人のユーザーが異なるプロパティを更新したため、データは失われません。 今度誰かが English 部署を閲覧すると、Jane の変更と John の変更が両方とも表示されます。開始日が 9/1/2013 で予算が 0 ドルです。 この更新方法では、データの損失につながる可能性がある競合の数を減らすことができますが、あるエンティティの同じプロパティに対して行われた変更が競合する場合、データの損失は避けられません。 Entity Framework がこのように動作するかどうかは、更新コードの実装方法に依存します。 これは Web アプリケーションの場合、実用的ではない場合が多いです。あるエンティティの新しい値に加え、元のプロパティ値もすべて追跡記録するため、大量のステータスを更新することになるからです。 大量のステータスを更新するとなると、サーバー リソースが必要になるか、Web ページ自体 (非表示フィールドなど) や cookie に含める必要があるため、アプリケーションのパフォーマンスに影響が出ます。

* John の変更で Jane の変更を上書きするように指定できます。

     今度誰かが English 部署を閲覧すると、日付は 9/1/2013 ですが、金額が $350,000.00 に戻っています。 これは *Client Wins* (クライアント側に合わせる) シナリオまたは *Last in Wins* (最終書き込み者優先) シナリオと呼ばれています。 (クライアントからの値がすべて、データ ストアの値より優先されます。)このセクションの冒頭でお伝えしたように、コンカレンシー処理について何のコーディングもしない場合、これが自動的に行われます。

* データベースで John の変更が更新されないようにできます。

     通常、エラー メッセージが表示され、John にデータの現在の状態が伝えられます。John は望むなら変更を再適用できます。 これは *Store Wins* (ストア側に合わせる) シナリオと呼ばれています。 (クライアントが送信した値よりデータストアの値が優先されます。)このチュートリアルでは、Store Wins シナリオを実装します。 この手法では、変更が上書きされるとき、それが必ずユーザーに警告されます。

### <a name="detecting-concurrency-conflicts"></a>コンカレンシーの競合の検出

Entity Framework がスローする `DbConcurrencyException` 例外を処理することで競合を解決できます。 このような例外がスローされるタイミングを認識する目的で、Entity Framework は競合を検出できなければなりません。 そのため、データベースとデータ モデルを適宜構成する必要があります。 競合検出を有効にするためのオプションには次のようなものがあります。

* 行が変更されたタイミングを判断するトラッキング列をデータベース テーブルに追加します。 その後、Entity Framework を構成し、SQL の Update または Delete コマンドの Where 句にその列を追加できます。

     トラッキング列のデータ型は通常、`rowversion` です。 `rowversion` 値は連続番号であり、行が更新されるたびに増えます。 Update または Delete コマンドでは、Where 句にトラッキング列の元の値が含まれます (元の行バージョン)。 更新中の行が別のユーザーによって変更された場合、`rowversion` 列の値は元の値とは異なります。Where 句に起因し、Update または Delete ステートメントは更新する行を見つけられません。 Update または Delete コマンドによって行が更新されていないことを Entity Framework が確認すると (影響を受けた行の数が 0 のとき)、それをコンカレンシーの競合として解釈します。

* Entity Framework を構成し、テーブルで Update コマンドと Delete コマンドの Where 句にすべての列の元の値を追加します。

     最初のオプションと同様に、行が最初に読み取られてから行に変更があった場合、Where 句は更新する行を返さず、Entity Framework はそれをコンカレンシーの競合として解釈します。 データベース テーブルに列がたくさんある場合、この手法では結果的に大量の Where 句が出現し、大量のステータスを保守管理しなければならなくなります。 先に述べたように、大量のステータスを保守管理することになると、アプリケーションのパフォーマンスに影響が出ます。 そのため、この手法は一般的には推奨されません。このチュートリアルでも利用しません。

     コンカレンシーにこの手法を導入する場合、コンカレンシーを追跡記録するエンティティのすべての主キーではないプロパティに `ConcurrencyCheck` 属性を追加し、印を付ける必要があります。 これで、Entity Framework では、Update ステートメントと Delete ステートメントの SQL Where 句にすべての列を含めることができます。

このチュートリアルの残りの部分では、Department エンティティに `rowversion` トラッキング プロパティを追加し、コントローラーとビューを作成し、すべてが適切に動作することをテストで確認します。

## <a name="add-a-tracking-property"></a>トラッキング プロパティを追加する

*Models/Department.cs* で、RowVersion という名前のトラッキング プロパティを追加します。

[!code-csharp[](intro/samples/cu/Models/Department.cs?name=snippet_Final&highlight=26,27)]

`Timestamp` 属性によって、データベースに送信された Update コマンドと Delete コマンドの Where 句にこの列が追加されます。 前のバージョンの SQL Server では、SQL `rowversion` に取って代わられる前、SQL `timestamp` というデータ型が使用されていたため、この属性は `Timestamp` と呼ばれています。 `rowversion` の .NET 型はバイト配列です。

fluent API を利用する場合、次の例のように、`IsConcurrencyToken` メソッドを利用し (*Data/SchoolContext.cs* で)、トラッキング プロパティを指定できます。

```csharp
modelBuilder.Entity<Department>()
    .Property(p => p.RowVersion).IsConcurrencyToken();
```

プロパティを追加し、データベース モデルを変更したので、別の移行を行う必要があります。

変更内容を保存し、プロジェクトをビルドしてください。その後、コマンド ウィンドウに次のコマンドを入力します。

```dotnetcli
dotnet ef migrations add RowVersion
```

```dotnetcli
dotnet ef database update
```

## <a name="create-departments-controller-and-views"></a>Departments のコントローラーとビューを作成する

先に Students、Courses、Instructors に行ったように、Departments のコントローラーとビューをスキャフォールディングします。

![Department のスキャフォールディング](concurrency/_static/add-departments-controller.png)

*DepartmentsController.cs* ファイルに 4 回登場する "FirstMidName" をすべて "FullName" に変更します。それにより、部署の管理者のドロップダウン リストに講師の姓だけでなく、姓名が表示されます。

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_Dropdown)]

## <a name="update-index-view"></a>Index ビューを更新する

スキャフォールディング エンジンによりインデックス ビューに `RowVersion` 列が作成されましたが、このフィールドは表示すべきではありません。

*Views/Departments/Index.cshtml* のコードを次のコードに変更します。

[!code-cshtml[](intro/samples/cu/Views/Departments/Index.cshtml?highlight=4,7,44)]

これで見出しが "Departments" に変更され、`RowVersion` 列が削除され、管理者の名ではなく姓名が表示されます。

## <a name="update-edit-methods"></a>Edit メソッドを更新する

HttpGet `Edit` メソッドと `Details` メソッドの両方に `AsNoTracking` を追加します。 HttpGet `Edit` メソッドに Administrator の一括読み込みを追加します。

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_EagerLoading)]

HttpPost `Edit` メソッドの既存コードを次のコードに変更します。

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_EditPost)]

このコードはまず、更新する部署を読み込みます。 `FirstOrDefaultAsync` が null を返した場合、部署は別のユーザーが削除しています。 その場合、このコードは送信されたフォーム値を利用して `Department` エンティティを作成します。編集ページはエラー メッセージと共に再表示できます。 あるいは、部署フィールドを再表示せず、エラー メッセージのみを表示するのであれば、`Department` エンティティを再作成する必要はないでしょう。

ビューの非表示フィールドに元の `RowVersion` 値が保管されます。このメソッドは `rowVersion` パラメーターでその値を受け取ります。 `SaveChanges` を呼び出す前に、エンティティの `OriginalValues` コレクションにその元の `RowVersion` プロパティ値を置く必要があります。

```csharp
_context.Entry(departmentToUpdate).Property("RowVersion").OriginalValue = rowVersion;
```

次に、Entity Framework で SQL UPDATE コマンドが作成されるとき、元の `RowVersion` 値が含まれる行を探す WHERE 句がそのコマンドに含まれます。 UPDATE コマンドの影響を受ける行がない場合 (元の `RowVersion` 値が含まれる行がない)、Entity Framework は `DbUpdateConcurrencyException` 例外をスローします。

その例外の catch ブロックのコードによって、影響を受けた、例外オブジェクトの `Entries` プロパティから更新後の値が与えられた Department エンティティが取得されます。

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?range=164)]

`Entries` コレクションには `EntityEntry` オブジェクトが 1 つだけ与えられます。  そのオブジェクトを利用し、ユーザーが入力した新しい値と現在のデータベース値を取得できます。

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?range=165-166)]

このコードにより、編集ページでユーザーが入力したものとデータベース値が異なる列ごとにカスタムのエラー メッセージが追加されます (簡潔にするため、ここでは 1 つだけフィールドを示しています)。

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?range=174-178)]

最後になりますが、このコードで `departmentToUpdate` の `RowVersion` 値がデータベースから取得された新しい値に設定されます。 Edit ページが再表示されるとき、この新しい `RowVersion` 値が非表示フィールドに保存されます。今度ユーザーが **[保存]** をクリックすると、Edit ページの再表示後に発生したコンカレンシー エラーのみが取得されます。

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?range=199-200)]

`ModelState` の `RowVersion` 値が古いため、`ModelState.Remove` ステートメントが必要になります。 このビューでは、フィールドの `ModelState` 値がモデル プロパティ値より優先されます。

## <a name="update-edit-view"></a>Edit ビューを更新する

*Views/Departments/Edit.cshtml* で、次の変更を行います。

* `DepartmentID` プロパティの非表示フィールドのすぐ後ろに `RowVersion` プロパティ値を保存する非表示フィールドを追加します。

* ドロップダウン リストに "Select Administrator" オプションを追加します。

[!code-cshtml[](intro/samples/cu/Views/Departments/Edit.cshtml?highlight=16,34-36)]

## <a name="test-concurrency-conflicts"></a>コンカレンシーの競合をテストする

アプリを実行し、Departments Index ページに移動します。 English 部署の **[編集]** ハイパーリンクを右クリックし、 **[新しいタブで開く]** を選択し、English 部署の **[編集]** ハイパーリンクをクリックします。 2 つのブラウザー タブに同じ情報が表示されます。

最初のブラウザー タブでフィールドを変更し、 **[保存]** をクリックします。

![変更後の Department Edit ページ 1](concurrency/_static/edit-after-change-1.png)

値が変更された Index ページがブラウザーに表示されます。

2 番目のブラウザー タブでフィールドを変更します。

![変更後の Department Edit ページ 2](concurrency/_static/edit-after-change-2.png)

**[保存]** をクリックします。 エラー メッセージが表示されます。

![Department Edit ページのエラー メッセージ](concurrency/_static/edit-error.png)

**[保存]** をもう一度クリックします。 2 番目のブラウザー タブで入力した値が保存されます。 Index ページが表示されると、保存した値を確認できます。

## <a name="update-the-delete-page"></a>[削除] ページを更新する

Delete ページの場合、Entity Framework は、同様の方法で部署を編集している他のユーザーが起こしたコンカレンシーの競合を検出します。 HttpGet `Delete` 値に確定ビューが表示されると、そのビューに非表示フィールドの元の `RowVersion` 値が含まれます。 その値は、ユーザーが削除を確定したときに呼び出される HttpPost `Delete` メソッドで利用されます。 Entity Framework で SQL DELETE コマンドが作成されるとき、WHERE 句と元の `RowVersion` 値が含まれます。 コマンドの結果、影響を受けた行が 0 であれば (削除確認ページの表示後に行が変更された)、コンカレンシー例外がスローされます。確定ページをエラー メッセージと共に再表示するために、エラー フラグが true に設定された状態で HttpGet `Delete` メソッドが呼び出されます。 別のユーザーが行を削除したため、影響を受けた行が 0 になることもあります。その場合、エラー メッセージは表示されません。

### <a name="update-the-delete-methods-in-the-departments-controller"></a>Departments コントローラーの Delete メソッドの更新

*DepartmentsController.cs* で、HttpGet `Delete` メソッドを次のコードで置き換えます。

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_DeleteGet&highlight=1,10,14-17,21-29)]

このメソッドは、コンカレンシー エラー後にページが再表示されたかどうかを示すオプション パラメーターを受け取ります。 このフラグが true のとき、指定の部署が現存していなければ、別のユーザーによって削除されています。 その場合、このコードは Index ページにリダイレクトされます。  このフラグが true のとき、その部署が存在すれば、別のユーザーが変更しています。 その場合、このコードは `ViewData` を利用してビューにエラー メッセージを送信します。

HttpPost `Delete` メソッドのコード (名前は `DeleteConfirmed`) を次のコードで置き換えます。

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_DeletePost&highlight=1,3,5-8,11-18)]

置き換えたスキャフォールディングされたコードで、このメソッドがレコード ID を 1 つだけ受け取りました。

```csharp
public async Task<IActionResult> DeleteConfirmed(int id)
```

このパラメーターをモデル バインダーによって作成された `Department` エンティティ インスタンスに変更しています。 それにより、EF はレコード キーに加え、RowVers`ion プロパティ値にアクセスできます。

```csharp
public async Task<IActionResult> Delete(Department department)
```

また、アクション メソッドの名前を `DeleteConfirmed` から `Delete` に変更しています。 スキャフォールディングされたコードは `DeleteConfirmed` という名前を利用し、HttpPost メソッドに一意のシグネチャを与えました。 (CLR では、さまざまなメソッド パラメーターを持つために、オーバーロードされたメソッドを必要とします。)シグネチャが一意になっているので、MVC 規則に準拠し、削除メソッドの HttpPost と HttpGet に同じ名前を利用できます。

部署が既に削除されている場合、`AnyAsync` メソッドは false を返します。アプリケーションは Index メソッドに戻ります。

コンカレンシー エラーがキャッチされた場合、このコードは削除確認ページを再表示し、コンカレンシー エラー メッセージを表示するかどうかを示すフラグを提供します。

### <a name="update-the-delete-view"></a>Delete ビューを更新する

*Views/Departments/Delete.cshtml* で、スキャフォールディングされたコードを次のコードで置き換えます。このコードは、DepartmentID プロパティと RowVersion プロパティのエラー メッセージ フィールドと非表示フィールドを追加します。 変更が強調表示されます。

[!code-cshtml[](intro/samples/cu/Views/Departments/Delete.cshtml?highlight=9,38,44,45,48)]

これにより、次の変更が行われます。

* 見出しの `h2` と `h3` の間にエラー メッセージが追加されます。

* **[管理者]** フィールドで FirstMidName が FullName に変更されます。

* RowVersion フィールドが削除されます。

* `RowVersion` プロパティの非表示フィールドが追加されます。

アプリを実行し、Departments Index ページに移動します。 English 部署の **[削除]** ハイパーリンクを右クリックし、 **[新しいタブで開く]** を選択します。次に最初のタブで English 部署の **[編集]** ハイパーリンクをクリックします。

最初のウィンドウで、いずれかの値を変更し、 **[保存]** をクリックします。

![変更後/削除前の Department Edit ページ](concurrency/_static/edit-after-change-for-delete.png)

2 番目のタブで **[削除]** をクリックします。 コンカレンシー エラー メッセージが表示されます。Department 値がデータベースの現在の内容で更新されています。

![Department 削除確認ページとコンカレンシー エラー](concurrency/_static/delete-error.png)

**[削除]** をもう一度クリックすると、Index ページにリダイレクトされます。Index ページには、部署が削除されていることが表示されます。

## <a name="update-details-and-create-views"></a>Details ビューと Create ビューの更新

Details ビューと Create ビューで、スキャフォールディングされたコードを任意でクリーンアップできます。

RowVersion 列を削除し、管理者の姓名を表示するように *Views/Departments/Details.cshtml* のコードを置換します。

[!code-cshtml[](intro/samples/cu/Views/Departments/Details.cshtml?highlight=35)]

ドロップダウン リストに Select オプションを追加するように *Views/Departments/Create.cshtml* のコードを置換します。

[!code-cshtml[](intro/samples/cu/Views/Departments/Create.cshtml?highlight=32-34)]

## <a name="get-the-code"></a>コードを取得する

[完成したアプリケーションをダウンロードまたは表示する。](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="additional-resources"></a>その他の技術情報

 EF Core のコンカレンシー処理の詳細については、[コンカレンシーの競合](/ef/core/saving/concurrency)に関するページを参照してください。

## <a name="next-steps"></a>次の手順

このチュートリアルでは、次の作業を行いました。

> [!div class="checklist"]
> * コンカレンシーの競合について学習した
> * トラッキング プロパティを追加した
> * Departments のコントローラーとビューを作成した
> * Index ビューを更新した
> * Edit メソッドを更新した
> * Edit ビューを更新した
> * コンカレンシーの競合をテストした
> * Delete ページを更新した
> * Details および Create ビューを更新した

Instructor および Student エンティティの Table-Per-Hierarchy 継承を実装する方法について学習するには、次のチュートリアルに進んでください。

> [!div class="nextstepaction"]
> [次へ: Table-Per-Hierarchy 継承を実装する](inheritance.md)
