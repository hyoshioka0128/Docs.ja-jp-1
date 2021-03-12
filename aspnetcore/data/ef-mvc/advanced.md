---
title: 'チュートリアル: 高度なシナリオについて学習する - ASP.NET MVC と EF Core'
description: このチュートリアルでは、Entity Framework Core を使用するより高度な ASP.NET Core Web アプリを開発する際に、活用できるいくつかのトピックを紹介します。
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
uid: data/ef-mvc/advanced
ms.openlocfilehash: 135881bfb4137bc81bd0f7250f1ce191cab13dcd
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586840"
---
# <a name="tutorial-learn-about-advanced-scenarios---aspnet-mvc-with-ef-core"></a>チュートリアル: 高度なシナリオについて学習する - ASP.NET MVC と EF Core

前のチュートリアルでは、Table-Per-Hierarchy 継承を実装しました。 このチュートリアルでは、Entity Framework Core を使用するより高度な ASP.NET Core Web アプリケーションを開発する際に、注意すべきいくつかのトピックを紹介します。

このチュートリアルでは、次の作業を行いました。

> [!div class="checklist"]
> * 生 SQL クエリを実行する
> * エンティティを返すクエリを呼び出す
> * その他の型を返すクエリを呼び出す
> * 更新クエリを呼び出す
> * SQL クエリを調べる
> * 抽象化レイヤーを作成する
> * 変更の自動検出について学習する
> * EF Core のソース コードと開発計画について学習する
> * 動的な LINQ を使ってコードを簡略化する方法を学習する

## <a name="prerequisites"></a>必須コンポーネント

* [継承の実装](inheritance.md)

## <a name="perform-raw-sql-queries"></a>生 SQL クエリを実行する

Entity Framework を使用する利点の 1 つは、データを格納する特定のメソッドにコードを過度に接近させなくてもよい点です。 SQL クエリとコマンドが生成されるため、自分でこれらを記述する必要がなくなります。 ただし、例外的なシナリオがあります。手動で作成した特定の SQL クエリを実行する必要がある場合です。 このようなシナリオでは、Entity Framework Code First API には、SQL コマンドをデータベースに直接渡せるメソッドが含まれています。 EF Core 1.0 には次のオプションがあります。

* エンティティ型を返すクエリに対して `DbSet.FromSql` メソッドを使用します。 返されたオブジェクトは、`DbSet` オブジェクトで想定されている型である必要があります。また、[追跡をオフにしている場合を除き](crud.md#no-tracking-queries)、データベース コンテキストによって自動的に追跡されます。

* 非クエリ コマンドに対して `Database.ExecuteSqlCommand` を使用します。

エンティティではない型を返すクエリを実行する必要がある場合は、ADO.NET と EF で提供されるデータベース接続を使用できます。 このメソッドを使用してエンティティ型を取得する場合でも、返されるデータはデータベース コンテキストによって追跡されません。

Web アプリケーションで SQL コマンドを実行する場合は常に、SQL インジェクション攻撃から自身のサイトを保護する対策を講じる必要があります。 これを行う 1 つの方法として、パラメーター化されたクエリを使用して、Web ページによって送信された文字列が SQL コマンドとして解釈できないことを確認します。 このチュートリアルでは、ユーザー入力をクエリに統合するときに、パラメーター化されたクエリを使用します。

## <a name="call-a-query-to-return-entities"></a>エンティティを返すクエリを呼び出す

`DbSet<TEntity>` クラスは、`TEntity` 型のエンティティを返すクエリの実行に使用できるメソッドを提供します。 このしくみを確認するため、Department (部門) コントローラーの `Details` メソッド内のコードを変更します。

*DepartmentsController.cs* の `Details` メソッドで、次の強調表示されたコードに示されているように、部門を取得するコードを `FromSql` メソッドの呼び出しに置き換えます。

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_RawSQL&highlight=8,9,10)]

新しいコードが正しく動作することを確認するには、 **[Departments]\(部門\)** タブを選択し、いずれかの部門の **[Details]\(詳細\)** を選択します。

![部門の詳細](advanced/_static/department-details.png)

## <a name="call-a-query-to-return-other-types"></a>その他の型を返すクエリを呼び出す

以前に、登録日ごとの学生数を示す About ページ用に、学生の統計グリッドを作成しました。 Students エンティティ セット (`_context.Students`) からデータを取得し、LINQ を使用して結果を `EnrollmentDateGroup` ビュー モデル オブジェクトに投影しました。 LINQ を使用するのではなく、SQL そのものを記述するとします。 これを行うには、エンティティ オブジェクト以外のものを返す SQL クエリを実行する必要があります。 EF Core 1.0 では、これを行う方法の 1 つとして、ADO.NET コードを記述し、EF からデータベース接続を取得します。

*HomeController.cs* で、`About` メソッドを次のコードで置き換えます。

[!code-csharp[](intro/samples/cu/Controllers/HomeController.cs?name=snippet_UseRawSQL&highlight=3-32)]

using ステートメントを追加します。

[!code-csharp[](intro/samples/cu/Controllers/HomeController.cs?name=snippet_Usings2)]

アプリを実行して [About] ページに移動します。 以前に行ったのと同じデータが表示されます。

![About ページ](advanced/_static/about.png)

## <a name="call-an-update-query"></a>更新クエリを呼び出す

Contoso University の管理者が、すべてのコースの単位数を変更するなどの、データベースでグローバルな変更を実行するとします。 大学に多くのコースがある場合は、それらすべてをエンティティとして取得し、それらを個別に変更するのは非効率的です。 このセクションでは、すべてのコースの単位数を変更する係数をユーザーが指定できるようにする Web ページを実装し、SQL UPDATE ステートメントを実行することによって変更を行います。 Web ページは次の図のようになります。

![Course Credits ページを更新する](advanced/_static/update-credits.png)

*CoursesController.cs* で、HttpGet および HttpPost に UpdateCourseCredits メソッドを追加します。

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_UpdateGet)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_UpdatePost)]

コントローラーが HttpGet 要求を処理するときに、`ViewData["RowsAffected"]` では何も返されず、前の図に示されているように、ビューに空のテキスト ボックスと、[送信] ボタンが表示されます。

**[更新]** ボタンをクリックすると、HttpPost メソッドが呼び出され、乗数がテキスト ボックスに入力した値になります。 このコードは次に、コースを更新し、影響を受けた行の数を`ViewData` のビューに返す SQL を実行します。 ビューが `RowsAffected` 値を取得すると、更新された行の数を表示します。

**ソリューション エクスプローラー** で、*Views/Courses* フォルダーを右クリックし、 **[追加] > [新しい項目]** の順にクリックします。

**[新しい項目の追加]** ダイアログで、左側のウィンドウの **[インストール済み]** の下の **[ASP.NET Core]** をクリックし、 **[Razor ビュー]** をクリックして、新しいビューに *UpdateCourseCredits.cshtml* という名前を付けます。

*Views/Courses/UpdateCourseCredits.cshtml* で、テンプレート コードを次のコードに置き換えます。

[!code-cshtml[](intro/samples/cu/Views/Courses/UpdateCourseCredits.cshtml)]

**[Courses]\(コース\)** タブを選択してから、ブラウザーのアドレス バーで URL の末尾に "/UpdateCourseCredits" を追加して (例: `http://localhost:5813/Courses/UpdateCourseCredits`)、`UpdateCourseCredits` メソッドを実行します。 テキスト ボックスに数値を入力します。

![Course Credits ページを更新する](advanced/_static/update-credits.png)

**[更新]** をクリックします。 影響を受けた行の数が表示されます。

![Course Credits ページの更新で影響を受けた行](advanced/_static/update-credits-rows-affected.png)

**[リストに戻る]** をクリックして、単位数が変更されたコースの一覧を表示します。

実稼働コードでは、更新の結果が常に有効なデータになることが保証される点に注意してください。 ここに示した簡略化されたコードは、5 より大きい数値になるように単位数を乗算できます (`Credits` プロパティには `[Range(0, 5)]` 属性があります)。更新クエリは機能しますが、無効なデータによって、5 以下の単位数を想定していたシステムの他の部分で予期しない結果が発生する可能性があります。

SQL クエリの詳細については、「[Raw SQL Queries](/ef/core/querying/raw-sql)」 (生 SQL クエリ) を参照してください。

## <a name="examine-sql-queries"></a>SQL クエリを調べる

データベースに送信される実際の SQL クエリを確認できると役立つ場合があります。 ASP.NET Core の組み込みのログ記録機能は、クエリと更新の SQL を含むログを書き込むため、EF Core によって自動的に使用されます。 このセクションでは、SQL ログの例をいくつか紹介します。

*StudentsController.cs* を開き、`Details` メソッドで `if (student == null)` ステートメントにブレークポイントを設定します。

デバッグ モードでアプリを実行して、学生の [Details] ページに移動します。

デバッグの出力を示す **[出力]** ウィンドウに移動して、クエリを確認します。

```
Microsoft.EntityFrameworkCore.Database.Command:Information: Executed DbCommand (56ms) [Parameters=[@__id_0='?'], CommandType='Text', CommandTimeout='30']
SELECT TOP(2) [s].[ID], [s].[Discriminator], [s].[FirstName], [s].[LastName], [s].[EnrollmentDate]
FROM [Person] AS [s]
WHERE ([s].[Discriminator] = N'Student') AND ([s].[ID] = @__id_0)
ORDER BY [s].[ID]
Microsoft.EntityFrameworkCore.Database.Command:Information: Executed DbCommand (122ms) [Parameters=[@__id_0='?'], CommandType='Text', CommandTimeout='30']
SELECT [s.Enrollments].[EnrollmentID], [s.Enrollments].[CourseID], [s.Enrollments].[Grade], [s.Enrollments].[StudentID], [e.Course].[CourseID], [e.Course].[Credits], [e.Course].[DepartmentID], [e.Course].[Title]
FROM [Enrollment] AS [s.Enrollments]
INNER JOIN [Course] AS [e.Course] ON [s.Enrollments].[CourseID] = [e.Course].[CourseID]
INNER JOIN (
    SELECT TOP(1) [s0].[ID]
    FROM [Person] AS [s0]
    WHERE ([s0].[Discriminator] = N'Student') AND ([s0].[ID] = @__id_0)
    ORDER BY [s0].[ID]
) AS [t] ON [s.Enrollments].[StudentID] = [t].[ID]
ORDER BY [t].[ID]
```

驚くかもしれませんが、SQL は Person テーブルから最大 2 つの行 (`TOP(2)`) を選択します。 `SingleOrDefaultAsync` メソッドは、サーバー上の 1 行に解決されません。 その理由を説明します。

* クエリが複数の行を返すと、メソッドは null を返します。
* クエリが複数の行を返すかどうかを判断するため、EF はクエリが少なくとも 2 を返すかどうかを確認する必要があります。

**[出力]** ウィンドウでログ出力を取得するには、デバッグ モードを使用してブレークポイントで停止する必要はありません。 これは単に、出力を見たいポイントでログを停止する便利な方法です。 これを行わないと、ログ記録は続行され、関心がある部分までスクロールで戻る必要があります。

## <a name="create-an-abstraction-layer"></a>抽象化レイヤーを作成する

多くの開発者は、Entity Framework で動作するコードをラップするラッパーとして、Repository パターンと Unit of Work パターンを実装するためのコードを記述します。 これらのパターンは、アプリケーションのデータ アクセス層とビジネス ロジック層の間に抽象化レイヤーを作成するためのものです。 これらのパターンを実装すると、データ ストアの変更からアプリケーションを隔離でき、自動化された単体テストやテスト駆動開発 (TDD) を円滑化できます。 ただし、次の複数の理由により、追加のコードを記述してこれらのパターンを実装することが、EF を使用するアプリケーションにとって最善の選択肢ではない場合もあります。

* EF コンテキスト クラス自体が、コードをデータ ストア固有のコードから隔離します。

* EF コンテキスト クラスは、EF を使用して行っているデータベースの更新の unit-of-work クラスとして動作できます。

* リポジトリ コードを記述しなくても、EF には TDD を実装するための機能が含まれています。

Repository パターンと Unit of Work パターンを実装する方法については、[このチュートリアル シリーズの Entity Framework 5 バージョン](/aspnet/mvc/overview/older-versions/getting-started-with-ef-5-using-mvc-4/implementing-the-repository-and-unit-of-work-patterns-in-an-asp-net-mvc-application)を参照してください。

Entity Framework Core は、テストに使用できる In-Memory データベース プロバイダーを実装します。 詳細については、[InMemory を使ったテスト](/ef/core/miscellaneous/testing/in-memory)に関するページを参照してください。

## <a name="automatic-change-detection"></a>変更の自動検出

Entity Framework では、エンティティの現在の値と元の値を比較して、エンティティがどのように変更されたか (およびそれによって、どの更新プログラムをデータベースに送信する必要があるか) を判断します。 元の値は、エンティティが照会されるかアタッチされるときに格納されます。 変更の自動検出を行うメソッドには、次のようなものがあります。

* DbContext.SaveChanges

* DbContext.Entry

* ChangeTracker.Entries

多数のエンティティを追跡していて、これらのいずれかのメソッドをループ内で何度も呼び出す場合、`ChangeTracker.AutoDetectChangesEnabled` プロパティを使用して変更の自動検出を一時的にオフにすると、パフォーマンスが大幅に向上する場合があります。 次に例を示します。

```csharp
_context.ChangeTracker.AutoDetectChangesEnabled = false;
```

## <a name="ef-core-source-code-and-development-plans"></a>EF Core のソース コードと開発計画

Entity Framework Core のソースは、[https://github.com/dotnet/efcore](https://github.com/dotnet/efcore) にあります。 EF Core リポジトリには、夜間ビルド、問題追跡、機能仕様、設計ミーティング メモ、および[将来の開発のためのロードマップ](https://github.com/dotnet/efcore/wiki/Roadmap)が含まれています。 バグを見つけて報告したり、投稿することができます。

ソース コードはオープンですが、Entity Framework Core はマイクロソフト製品として完全にサポートされています。 Microsoft Entity Framework チームは、各リリースの品質を保証するため、受け入れる投稿を管理し、すべてのコード変更をテストしています。

## <a name="reverse-engineer-from-existing-database"></a>既存のデータベースからのリバース エンジニアリング

既存のデータベースからエンティティ クラスを含むデータ モデルをリバース エンジニアリングするには、[scaffold-dbcontext](/ef/core/miscellaneous/cli/powershell#scaffold-dbcontext) コマンドを使用します。 [入門用チュートリアル](/ef/core/get-started/aspnetcore/existing-db)を参照してください。

<a id="dynamic-linq"></a>

## <a name="use-dynamic-linq-to-simplify-code"></a>動的な LINQ を使ってコードを簡略化する

[このシリーズの 3 番目のチュートリアル](sort-filter-page.md)では、`switch`ステートメントで列名をハード コーディングすることで、LINQ コードを記述する方法を示しています。 選択する列が 2 つの場合は正常に機能しますが、多数の列がある場合は、コードが冗長になる可能性があります。 この問題を解決するため、`EF.Property` メソッドを使用して、プロパティの名前を文字列として指定できます。 この方法を試すには、`StudentsController` の `Index` メソッドを次のコードで置き換えます。

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_DynamicLinq)]

## <a name="acknowledgments"></a>謝辞

チュートリアルを執筆してくださった、Tom Dykstra と Rick Anderson (twitter @RickAndMSFT)。 コードの確認をサポートし、チュートリアル用のコードの記述中に発生した問題のデバッグを支援してくれた、Rowan Miller、Diego Vega、およびその他の Entity Framework チームのメンバー。 ASP.NET Core 2.2 用にチュートリアルの更新作業を行ってくれた、John Parente と Paul Goldman。

<a id="common-errors"></a>

## <a name="troubleshoot-common-errors"></a>一般的なエラーのトラブルシューティング

### <a name="contosouniversitydll-used-by-another-process"></a>ContosoUniversity.dll が別のプロセスによって使用されている

エラー メッセージ

> '...bin\Debug\netcoreapp1.0\ContosoUniversity.dll' を書き込み用に開けません -- '別のプロセスで使用されているため、プロセスはファイル '...\bin\Debug\netcoreapp1.0\ContosoUniversity.dll' にアクセスできません。

解決方法 :

IIS express でサイトを停止します。 Windows システム トレイに戻り、IIS Express を見つけてそのアイコンを右クリックし、Contoso University サイトを選択し、 **[サイトの停止]** をクリックします。

### <a name="migration-scaffolded-with-no-code-in-up-and-down-methods"></a>Up メソッドと Down メソッドでコードを使用せずに移行がスキャフォールディングされる

考えられる原因:

EF CLI コマンドは、コード ファイルを自動的に閉じて保存しません。 `migrations add` コマンドを実行するときに未保存の変更があると、EF は変更を検出しません。

解決方法 :

`migrations remove` コマンドを実行して、コードの変更を保存し、`migrations add` コマンドを再実行します。

### <a name="errors-while-running-database-update"></a>データベースの更新中のエラー

データが存在するデータベースでスキーマの変更を行っているときに、他のエラーが発生する場合があります。 解決できない移行エラーが発生した場合は、接続文字列のデータベース名を変更するか、データベースを削除できます。 新しいデータベースには移行するデータが存在しないため、update-database コマンドがエラーなしで完了する可能性が高くなります。

最も簡単な方法は、 *appsettings.json* でデータベースの名前を変更することです。 次に `database update` を実行したときに、新しいデータベースが作成されます。

SSOX でデータベースを削除するには、そのデータベースを右クリックして、 **[削除]** をクリックしてから、 **[データベースの削除]** ダイアログ ボックスで **[既存の接続を閉じる]** 、 **[OK]** の順にクリックします。

CLI を使用してデータベースを削除するには、`database drop` CLI コマンドを実行します。

```dotnetcli
dotnet ef database drop
```

### <a name="error-locating-sql-server-instance"></a>SQL Server インスタンスの位置を特定しているときのエラー

エラー メッセージ:

> SQL Server への接続を確立しているときに、ネットワーク関連またはインスタンス固有のエラーが発生しました。 サーバーが見つからないかアクセスできません。 インスタンス名が正しいこと、および SQL Server がリモート接続を許可するように構成されていることを確認してください。 (プロバイダー:SQL ネットワーク インターフェイス、エラー:26 - 指定されたサーバーまたはインスタンスの位置を特定しているときにエラーが発生しました)

解決方法 :

接続文字列を確認します。 データベース ファイルを手動で削除した場合は、構築文字列でデータベースの名前を変更して、新しいデータベースで最初からやり直します。

## <a name="get-the-code"></a>コードを取得する

[完成したアプリケーションをダウンロードまたは表示する。](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="additional-resources"></a>その他の技術情報

EF Core の詳細については、「[Entity Framework Core 概要](/ef/core)」を参照してください。 書籍「[Entity Framework Core in Action](https://www.manning.com/books/entity-framework-core-in-action)」もご利用いただけます。

Web アプリの展開方法については、「<xref:host-and-deploy/index>」を参照してください。

認証および承認などの、ASP.NET Core MVC に関連するその他のトピックについては、「<xref:index>」を参照してください。

## <a name="next-steps"></a>次の手順

このチュートリアルでは、次の作業を行いました。

> [!div class="checklist"]
> * 生 SQL クエリを実行した
> * エンティティを返すクエリを呼び出した
> * その他の型を返すクエリを呼び出した
> * 更新クエリを呼び出した
> * SQL クエリを調べた
> * 抽象化レイヤーを作成した
> * 変更の自動検出について学習した
> * EF Core のソース コードと開発計画について学習した
> * 動的な LINQ を使ってコードを簡略化する方法を学習した

これで、ASP.NET Core MVC アプリケーションでの Entity Framework Core の使用に関するチュートリアル シリーズは終了です。 このシリーズでは新しいデータベースが使用されました。別の方法としては、既存のデータベースからモデルをリバース エンジニアリングします。

> [!div class="nextstepaction"]
> [チュートリアル: EF Core と MVC、既存のデータベース](/ef/core/get-started/aspnetcore/existing-db?toc=/aspnet/core/toc.json&bc=/aspnet/core/breadcrumb/toc.json)
