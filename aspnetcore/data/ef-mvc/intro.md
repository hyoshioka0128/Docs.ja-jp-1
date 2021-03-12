---
title: 'チュートリアル: ASP.NET MVC Web アプリでの EF Core の概要'
description: このページは、Contoso University のサンプル EF/MVC アプリを作成する方法を説明するチュートリアル シリーズの 1 回目です
author: rick-anderson
ms.author: riande
ms.custom: mvc
ms.date: 11/06/2020
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
uid: data/ef-mvc/intro
ms.openlocfilehash: 5bb1a05f5e81730d0785c1d6a69a8cafb68f85ab
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586684"
---
# <a name="tutorial-get-started-with-ef-core-in-an-aspnet-mvc-web-app"></a>チュートリアル: ASP.NET MVC Web アプリでの EF Core の概要

作成者: [Tom Dykstra](https://github.com/tdykstra)、[Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-5.0"

[!INCLUDE [RP better than MVC](~/includes/RP-EF/rp-over-mvc.md)]

Contoso University のサンプル Web アプリでは、Entity Framework (EF) Core と Visual Studio を使用して ASP.NET Core MVC Web アプリを作成する方法を示します。

このサンプル アプリは架空の Contoso University の Web サイトです。 学生の受け付け、講座の作成、講師の割り当てなどの機能が含まれています。 これは、Contoso University のサンプル アプリを作成する方法を説明するチュートリアル シリーズの 1 回目です。

## <a name="prerequisites"></a>前提条件

* ASP.NET Core MVC を初めて使用する場合は、これを開始する前に、「[ASP.NET Core MVC の概要](xref:tutorials/first-mvc-app/start-mvc)」チュートリアル シリーズをご覧ください。

[!INCLUDE[VS prereqs](~/includes/net-core-prereqs-vs-5.0.md)]

## <a name="database-engines"></a>データベース エンジン

Visual Studio の手順では、[SQL Server LocalDB](/sql/database-engine/configure-windows/sql-server-2016-express-localdb) を使用します。これは、Windows 上でのみ実行される SQL Server Express のバージョンです。

<!--
The Visual Studio Code instructions use [SQLite](https://www.sqlite.org/), a cross-platform database engine.

If you choose to use SQLite, download and install a third-party tool for managing and viewing a SQLite database, such as [DB Browser for SQLite](https://sqlitebrowser.org/).
-->

## <a name="solve-problems-and-troubleshoot"></a>問題の解決とトラブルシューティング

解決できない問題に遭遇した場合、通常、[完成済みのプロジェクト](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples)と自分のコードを比較することで解決策がわかります。 一般的なエラーとその解決方法の一覧については、[このシリーズの最後のチュートリアルにあるトラブルシューティングのセクション](advanced.md#common-errors)をご覧ください。 そこで必要な答えが見つからない場合、StackOverflow.com で [ASP.NET Core](https://stackoverflow.com/questions/tagged/asp.net-core) または [EF Core](https://stackoverflow.com/questions/tagged/entity-framework-core) に関する質問を投稿できます。

> [!TIP]
> これは 10 回のチュートリアルからなるシリーズであり、いずれの回も前のチュートリアルを基盤にしています。 チュートリアルが完了したら、毎回、プロジェクトのコピーを保存するようお勧めします。 問題に遭遇したとき、前のチュートリアルから始めることができます。シリーズ全体の始めまで戻る必要がありません。

## <a name="contoso-university-web-app"></a>Contoso University Web アプリ

このチュートリアル シリーズで作成するアプリは、大学向けの基本的な Web サイトです。

ユーザーは学生、講座、講師の情報を見たり、更新したりできます。 アプリの画面のいくつかを次に示します。

![Students インデックス ページ](intro/_static/students-index.png)

![Students 編集ページ](intro/_static/student-edit.png)

## <a name="create-web-app"></a>Web アプリを作成する

1. Visual Studio を開始し、 **[新しいプロジェクトの作成]** を選択します。
1. **[新しいプロジェクトの作成]** ダイアログで、 **[ASP.NET Core Web アプリケーション]** > **[次へ]** の順に選択します。
1. **[新しいプロジェクトの構成]** ダイアログで、 **[プロジェクト名]** に「`ContosoUniversity`」と入力します。 コードをコピーするときに各`namespace`が一致するように、この正確な名前 (大文字と小文字を含む) を使用することが重要です。
1. **［作成］** を選択します
1. **[新しい ASP.NET Core Web アプリケーションの作成]** ダイアログで、次のものを選択します。
    1. ドロップダウンで **[.NET Core]** と **[ASP.NET Core 5.0]**
    1. **ASP.NET Core Web アプリ (Model-View-Controller)**
    1. **[作成]** 
      ![[新しい ASP.NET Core プロジェクト] ダイアログ](~/data/ef-mvc/intro/_static/new-aspnet5.png)

## <a name="set-up-the-site-style"></a>サイトのスタイルを設定する

基本的な変更をいくつか行い、サイトのメニュー、レイアウト、ホーム ページを決めます。

*Views/Shared/_Layout.cshtml* を開き、次のように変更します。

* 各 `ContosoUniversity` を `Contoso University` に変更します。 これは 3 回出てきます。
* メニュー エントリとして「**About**」、「**Students**」、「**Courses**」、「**Instructors**」、「**Departments**」を追加し、「**Privacy**」メニュー エントリを削除します。

次のコードでは上の変更が強調表示されています。

[!code-cshtml[](intro/samples/5cu/Views/Shared/_Layout.cshtml?highlight=6,24-38,52)]

*Views/Home/Index.cshtml* で、ファイルの内容を次のマークアップに置き換えます。

[!code-cshtml[](intro/samples/5cu/Views/Home/Index.cshtml)]

CTRL を押しながら F5 を押してプロジェクトを実行するか、メニューで **[デバッグ]、[デバッグなしで開始]** の順に選択します。 ホーム ページに、このチュートリアルで作成されるページのタブが表示されます。

![Contoso University のホーム ページ](intro/_static/home-page5.png)

## <a name="ef-core-nuget-packages"></a>EF Core の NuGet パッケージ

このチュートリアルでは SQL Server を使用します。プロバイダー パッケージは [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer/) です。

EF SQL Server パッケージとその依存関係 `Microsoft.EntityFrameworkCore` と `Microsoft.EntityFrameworkCore.Relational` により、EF のランタイム サポートが提供されます。

[Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore) NuGet パッケージと [Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore) NuGet パッケージを追加します。 パッケージ マネージャー コンソール (PMC) で、次のコマンドを入力して NuGet パッケージを追加します。

```powershell
Install-Package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore
Install-Package Microsoft.EntityFrameworkCore.SqlServer
```

`Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore` NuGet パッケージには EF Core のエラー ページ用の ASP.NET Core ミドルウェアが用意されています。 このミドルウェアは、EF Core の移行に関するエラーを検出して診断するのに役立ちます。

EF Core で利用できるその他のデータベース プロバイダーに関しては、「[データベース プロバイダー](/ef/core/providers/)」を参照してください。

## <a name="create-the-data-model"></a>データ モデルを作成する

このアプリ用に次のエンティティ クラスを作成します。

![Course、Enrollment、Student からなるデータ モデルの図](intro/_static/data-model-diagram.png)

上のエンティティには、次のリレーションシップがあります。

* `Student` エンティティと `Enrollment` エンティティの間の一対多リレーションシップ。 1 人の学生を任意の数の講座に登録できます。
* `Course` エンティティと `Enrollment` エンティティの間の一対多リレーションシップ。 1 つの講座に任意の数の学生を登録できます。

次のセクションでは、これらの各エンティティに対してクラスを作成します。

### <a name="the-student-entity"></a>Student エンティティ

![Student エンティティの図](intro/_static/student-entity.png)

以下のコードを使用して、*Models* フォルダーに `Student` クラスを作成します。

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_Intro)]

`ID` プロパティは、このクラスに相当するデータベース テーブルの主キー (**PK**) 列です。 既定では、EF は、`ID` または `classnameID` という名前のプロパティを主キーとして解釈します。 たとえば、PK には `ID` ではなく `StudentID` という名前を付けることができます。

`Enrollments` プロパティは[ナビゲーション プロパティ](/ef/core/modeling/relationships)です。 ナビゲーション プロパティには、このエンティティに関連する他のエンティティが含まれます。 `Student` エンティティの `Enrollments` プロパティ:

* その `Student` エンティティに関連するすべての `Enrollment` エンティティが含まれます。
* データベースの特定の `Student` 行に、関連する `Enrollment` 行が 2 つある場合:
  * その `Student` エンティティの `Enrollments` ナビゲーション プロパティには、これら 2 つの `Enrollment` エンティティが含まれます。
  
`Enrollment` 行には、`StudentID` 外部キー (**FK**) 列の学生の PK 値が含まれます。

ナビゲーション プロパティが複数のエンティティを保持できる場合:

 * 型は、`ICollection<T>`、`List<T>`、`HashSet<T>` などのリストである必要があります。
 * エンティティの追加、削除、更新を行うことができます。

多対多および一対多のナビゲーション リレーションシップは、複数のエンティティを含むことができます。 `ICollection<T>` を使用する場合、EF によって `HashSet<T>` コレクションが既定で作成されます。

### <a name="the-enrollment-entity"></a>Enrollment エンティティ

![Enrollment エンティティの図](intro/_static/enrollment-entity.png)

以下のコードを使用して、*Models* フォルダーに `Enrollment` クラスを作成します。

[!code-csharp[](intro/samples/cu/Models/Enrollment.cs?name=snippet_Intro)]

`EnrollmentID` プロパティは PK です。 このエンティティには、`ID` 自体ではなく `classnameID` パターンが使用されます。 `Student` エンティティには `ID` パターンが使用されていました。 データ モデル全体で 1 つのパターンを使用することを好む開発者もいます。 このチュートリアルのバリエーションでは、どちらのパターンも使用できることが示されています。 [後のチュートリアル](inheritance.md)では、クラス名なしの `ID` を使用して、データ モデルに継承を簡単に実装する方法を示します。

`Grade` プロパティは `enum` です。 `Grade` の型宣言の後にある `?` は、`Grade` プロパティが [Null 許容](/dotnet/csharp/language-reference/builtin-types/nullable-value-types)であることを示します。 `null` という成績は、成績が 0 であるということではありません。 `null` は、成績がわからないこと、またはまだ割り当てられていないことを意味します。

`StudentID` プロパティは外部キー (FK) であり、それに対応するナビゲーション プロパティは `Student` です。 `Enrollment` エンティティは 1 つの `Student` エンティティに関連付けられるので、プロパティで保持できる `Student` エンティティは 1 つだけです。 これは、複数の `Enrollment` エンティティを保持できる `Student.Enrollments` ナビゲーション プロパティとは異なります。

`CourseID` プロパティは FK で、それに対応するナビゲーション プロパティは `Course` です。 `Enrollment` エンティティは 1 つの `Course` エンティティに関連付けられます。

プロパティの名前が `<`ナビゲーション プロパティ名`><`主キー プロパティ名`>` になっている場合、それは Entity Framework により FK プロパティとして解釈されます。 たとえば、`Student` ナビゲーション プロパティに対する `StudentID` (`Student` エンティティの PK は `ID` であるため)。 FK プロパティは、`<`主キー プロパティ名`>` という名前にすることもできます。 たとえば、`CourseID` (`Course` エンティティの PK が `CourseID` であるため)。

### <a name="the-course-entity"></a>Course エンティティ

![Course エンティティの図](intro/_static/course-entity.png)

以下のコードを使用して、*Models* フォルダーに `Course` クラスを作成します。

[!code-csharp[](intro/samples/cu/Models/Course.cs?name=snippet_Intro)]

`Enrollments` プロパティはナビゲーション プロパティです。 1 つの `Course` エンティティにたくさんの `Enrollment` エンティティを関連付けることができます。

[DatabaseGenerated](xref:System.ComponentModel.DataAnnotations.DatabaseGeneratedAttribute) 属性については、[後のチュートリアル](complex-data-model.md)で説明されています。 この属性を使用すると、講座の PK を、データベースによって生成するのではなく、自分で入力できます。

## <a name="create-the-database-context"></a>データベース コンテキストの作成

定められたデータ モデルの EF 機能を調整するメイン クラスは、<xref:Microsoft.EntityFrameworkCore.DbContext> データベース コンテキスト クラスです。 このクラスは `Microsoft.EntityFrameworkCore.DbContext` クラスから派生させて作成します。 `DbContext` 派生クラスによって、データ モデルに含めるエンティティが指定されます。 EF の一部の動作はカスタマイズできます。 このプロジェクトでは、クラスに `SchoolContext` という名前が付けられています。

プロジェクト フォルダーで、`Data` という名前のフォルダーを作成します。

*Data* フォルダーに、次のコードで `SchoolContext` クラスを作成します。

[!code-csharp[](intro/samples/cu/Data/SchoolContext.cs?name=snippet_Intro)]

上記のコードにより、各エンティティ セットに対する `DbSet` プロパティが作成されます。 EF の用語では:

* エンティティ セットは通常、データベース テーブルに対応します。
* エンティティはテーブル内の行に対応します。

`DbSet<Enrollment>` ステートメントと `DbSet<Course>` ステートメントは省略することができ、動作は同じです。 EF には、次の理由により暗黙的にこれらが含まれます。

* `Student` エンティティでは、`Enrollment` エンティティが参照されます。
* `Enrollment` エンティティでは、`Course` エンティティが参照されます。

データベースが作成されると、EF によって、`DbSet` プロパティと同じ名前を持つテーブルが作成されます。 コレクションのプロパティ名は通常、複数形になります。 たとえば、`Student` ではなく `Students`。 テーブル名を複数形にするかどうかについては、開発者の間で意見が分かれるでしょう。 これらのチュートリアルでは、`DbContext` で単数形のテーブル名を指定することにより、既定の動作をオーバーライドします。 そのために、最後の DbSet プロパティの後に、次の強調表示されているコードを追加します。

[!code-csharp[](intro/samples/cu/Data/SchoolContext.cs?name=snippet_TableNames&highlight=16-21)]

## <a name="register-the-schoolcontext"></a>`SchoolContext` を登録する

ASP.NET Core には、[依存関係挿入](../../fundamentals/dependency-injection.md)が含まれています。 EF データベース コンテキストなどのサービスは、アプリの起動時に依存関係の挿入で登録されます。 これらのサービス (MVC コント ローラーなど) を必要とするコンポーネントには、コンストラクターのパラメーターを介してこれらのサービスが提供されます。 コンテキスト インスタンスを取得するコントローラー コンストラクターのコードは、このチュートリアルで後ほど示します。

`SchoolContext` をサービスとして登録するには、*Startup.cs* を開き、強調表示されている行を `ConfigureServices` メソッドに追加します。

[!code-csharp[](intro/samples/5cu-snap/Startup.cs?name=snippet&highlight=1-2,22-23)]

`DbContextOptionsBuilder` オブジェクトでメソッドが呼び出され、接続文字列の名前がコンテキストに渡されます。 ローカル開発の場合、[ASP.NET Core 構成システム](xref:fundamentals/configuration/index)によって *appsettings.json* ファイルから接続文字列が読み取られます。

*appsettings.json* ファイルを開き、次のマークアップで示されているように接続文字列を追加します。

[!code-json[](./intro/samples/5cu/appsettings1.json?highlight=2-4)]

### <a name="add-the-database-exception-filter"></a>データベース例外フィルターを追加する

次のコードに示すように、`ConfigureServices` に <xref:Microsoft.Extensions.DependencyInjection.DatabaseDeveloperPageExceptionFilterServiceExtensions.AddDatabaseDeveloperPageExceptionFilter%2A> を追加します。

[!code-csharp[](intro/samples/5cu/Startup.cs?name=snippet&highlight=6)]

`AddDatabaseDeveloperPageExceptionFilter` により、[開発環境](xref:fundamentals/environments)で役に立つエラー情報が提供されます。

### <a name="sql-server-express-localdb"></a>SQL Server Express LocalDB

この接続文字列によって [SQL Server LocalDB](/sql/database-engine/configure-windows/sql-server-2016-express-localdb) が指定されます。 LocalDB は SQL Server Express データベース エンジンの軽量版であり、実稼働ではなく、アプリの開発を意図して設計されています。 LocalDB は要求時に開始され、ユーザー モードで実行されるため、複雑な構成はありません。 既定では、LocalDB は `C:/Users/<user>` ディレクトリに *.mdf* DB ファイルを作成します。

## <a name="initialize-db-with-test-data"></a>テスト データで DB を初期化する

EF により、空のデータベースが作成されます。 このセクションでは、データベースが作成された後でそれにテスト データを設定するために呼び出されるメソッドを追加します。

`EnsureCreated` メソッドを使用して、データベースを自動的に作成します。 [後のチュートリアル](migrations.md)では、モデル変更の処理方法について説明します。データベースを削除し、再作成するのではなく、Code First Migrations を利用してデータベース スキーマを変更します。

*Data* フォルダーに、次のコードで `DbInitializer` という名前の新しいクラスを作成します。

[!code-csharp[DbInitializer](intro/samples/5cu-snap/DbInitializer.cs)]

上記のコードでは、データベースが存在するかどうかを確認します。

* データベースが見つからない場合
  * 作成されて、テスト データが読み込まれます。 `List<T>` コレクションではなく配列にテスト データを読み込み、パフォーマンスを最適化します。
* データベースが見つかった場合は、何も行われません。

次のコードを使用して *Program.cs* を更新します。

[!code-csharp[Program file](intro/samples/5cu-snap/Program.cs?highlight=1-2,14-18,21-37)]

アプリの起動時に、*Program.cs* によって次のことが行われます。

* 依存関係挿入コンテナーからデータベース コンテキスト インスタンスを取得します。
* `DbInitializer.Initialize` メソッドを呼び出します。
* 次のコードで示すように、`Initialize` メソッドが完了したらコンテキストを破棄します。

[!code-csharp[](intro/samples/cu/Program.cs?name=snippet_Seed&highlight=5-18)]

アプリが初めて実行されたときは、データベースが作成され、テスト データが読み込まれます。 データ モデルが変更されるたび:

* データベースを削除します。
* シード メソッドを更新し、新しいデータベースで新たに開始します。

 後のチュートリアルでは、データ モデルが変更されたら、データベースを削除して作成し直すのではなく、それを更新します。 データ モデルが変更されても、データは失われません。

## <a name="create-controller-and-views"></a>コントローラーとビューを作成する

Visual Studio のスキャフォールディング エンジンを使用して、MVC コントローラーとビューを追加します。これらにより、EF を使用してクエリが実行され、データが保存されます。

[CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) アクションのメソッドとビューの自動作成は、スキャフォールディングと呼ばれます。

* **ソリューション エクスプローラー** で、`Controllers` フォルダーを右クリックし、 **[追加] > [新規スキャフォールディング アイテム]** を選択します。
* **[スキャフォールディングを追加]** ダイアログ ボックスで:
  * **[Entity Framework を使用したビューがある MVC コントローラー]** を選択します。
  * **[追加]** をクリックします。 **[Add MVC Controller with views, using Entity Framework]\(Entity Framework を使用したビューがある MVC コントローラーを追加する\)** ダイアログ ボックスが表示されます。![Student のスキャフォールディング](intro/_static/scaffold-student2.png)
  * **[モデル クラス]** で **Student** を選択します。
  * **[データ コンテキスト クラス]** で **SchoolContext** を選択します。
  * 名前は **StudentsController** をそのまま選択します。
  * **[追加]** をクリックします。

Visual Studio スキャフォールディング エンジンにより、`StudentsController.cs` ファイルと、コントローラーで動作するビューのセット (`*.cshtml` ファイル) が作成されます。

コントローラーによりコンストラクター パラメーターとして `SchoolContext` が受け取られることに注意してください。

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_Context&highlight=5,7,9)]

ASP.NET Core 依存関係挿入では、`SchoolContext` のインスタンスがコントローラーに渡されます。 `Startup` クラスでそれを構成しました。

コントローラーには `Index` アクション メソッドが含まれます。これはデータベースにあるすべての学生を表示します。 このメソッドはデータベース コンテキスト インスタンスの `Students` プロパティを読み取り、Students エンティティ セットから学生の一覧を取得します。

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_ScaffoldedIndex&highlight=3)]

このコードの非同期プログラミング要素については、チュートリアルで後ほど説明します。

*Views/Students/Index.cshtml* ビューには、この一覧で表形式で表示されます。

[!code-cshtml[](intro/samples/cu/Views/Students/Index1.cshtml)]

CTRL を押しながら F5 を押してプロジェクトを実行するか、メニューで **[デバッグ]、[デバッグなしで開始]** の順に選択します。

[Students] タブをクリックすると、`DbInitializer.Initialize` メソッドによって挿入されたテスト データが表示されます。 ブラウザーのウィンドウ幅によって決まることですが、`Students` タブ リンクはページの一番上に表示されるか、右上隅のナビゲーション アイコンをクリックしないと表示されません。

![Contoso University のホーム ページ (ウィンドウ幅が狭いとき)](intro/_static/home-page-narrow.png)

![Students インデックス ページ](intro/_static/students-index.png)

## <a name="view-the-database"></a>データベースを表示する

アプリが起動すると、`DbInitializer.Initialize` メソッドによって `EnsureCreated` が呼び出されます。 EF により、データベースが存在しないことが認識されました。

* そのため、データベースが作成されました。
* `Initialize` メソッドのコードにより、データベースにデータが設定されました。

Visual Studio でデータベースを表示するには、**SQL Server オブジェクト エクスプローラー** (SSOX) を使用します。

* Visual Studio の **[表示]** メニューで **[SQL Server オブジェクト エクスプローラー]** を選択します。
* SSOX で、 **(localdb)\MSSQLLocalDB > [データベース]** を選択します。
* `ContosoUniversity1` を選択します。これは、 *appsettings.json* ファイル内の接続文字列に含まれるデータベース名のエントリです。
* **[テーブル]** ノードを展開し、データベースのテーブルを表示します。

![SSOX のテーブル](intro/_static/ssox-tables.png)

**Student** テーブルを右クリックし、 **[データの表示]** をクリックして、テーブル内のデータを表示します。

![SSOX の Student テーブル](intro/_static/ssox-student-table.png)

`*.mdf` データベース ファイルと `*.ldf` データベース ファイルは *C:\Users\\\<username>* フォルダーにあります。

アプリの起動時に実行される初期化メソッドで `EnsureCreated` が呼び出されるため、次のことができます。

* `Student` クラスに対する変更を行います。
* データベースを削除します。
* アプリを停止してから開始します。 変更に合わせてデータベースが自動的に再作成されます。

たとえば、`Student` クラスに `EmailAddress` プロパティが追加された場合、再作成されたテーブルには新しい `EmailAddress` 列が含まれます。 ビュー クラスには、新しい `EmailAddress` プロパティは表示されません。

## <a name="conventions"></a>規約

EF によって使用される規約のため、EF で完全なデータベースが作成されるために記述するコードの量は最小限に抑えられます。

* `DbSet` プロパティの名前がテーブル名として使用されます。 `DbSet` プロパティによって参照されないエンティティについては、エンティティ クラス名がテーブル名として使用されます。
* 列名には、エンティティ プロパティ名が使用されます。
* `ID` または `classnameID` という名前のエンティティ プロパティは、PK プロパティとして認識されます。
* 名前が `<`ナビゲーション プロパティ名`><`PK プロパティ名`>` であるプロパティは、FK プロパティとして解釈されます。 たとえば、`Student` ナビゲーション プロパティに対する `StudentID` (`Student` エンティティの PK は `ID` であるため)。 FK プロパティは、`<`主キー プロパティ名`>` という名前にすることもできます。 たとえば、`EnrollmentID` (`Enrollment` エンティティの PK が `EnrollmentID` であるため)。

従来の動作をオーバーライドできます。 たとえば、このチュートリアルで先に示したように、テーブル名を明示的に指定できます。 列名と任意のプロパティを、PK または FK として設定できます。

## <a name="asynchronous-code"></a>非同期コード

ASP.NET Core と EF Core では、非同期プログラミングが既定のモードです。

Web サーバーでは、利用できるスレッド数に限りがあります。負荷が高い状況では、利用できるスレッドが全部使われる可能性があります。 その場合、スレッドが解放されるまでサーバーは新しい要求を処理できません。 同期コードの場合、たくさんのスレッドが関連付けられていても、I/O の完了を待っているため、実際には何の作業も行っていないということがあります。 非同期コードの場合、あるプロセスが I/O の完了を待っているとき、そのスレッドは解放され、サーバーによって他の要求の処理に利用できます。 結果として、非同期コードの場合、サーバー リソースをより効率的に利用できます。サーバーは、より多くのトラフィックを遅延なく処理できます。

非同期コードは実行時に少量のオーバーヘッドを発生させるが、トラフィックが少ない場合、パフォーマンスに与える影響は無視して構わない程度です。トラフィックが多い場合、相当なパフォーマンス改善が見込まれます。

次のコードでは、`async`、`Task<T>`、`await`、`ToListAsync` により、コードは非同期的に実行されます。

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_ScaffoldedIndex)]

* キーワード `async` は、メソッド本文の一部にコールバックを生成し、返された `Task<IActionResult>` オブジェクトを自動作成するようにコンパイラに伝えます。
* 戻り値の型 `Task<IActionResult>` は、進行中の作業と型 `IActionResult` の結果を表します。
* キーワード `await` により、コンパイラはメソッドを 2 つに分割します。 最初の部分は、非同期で開始される操作で終わります。 2 つ目の部分は、操作の完了時に呼び出されるコールバック メソッドに入ります。
* `ToListAsync` は、`ToList` 拡張メソッドの非同期バージョンです。

EF を使用する非同期コードを記述するときに注意すべき点:

* クエリやコマンドをデータベースに送信するステートメントのみが非同期で実行されます。 たとえば、`ToListAsync`、`SingleOrDefaultAsync`、`SaveChangesAsync` などです。 `var students = context.Students.Where(s => s.LastName == "Davolio")` など、`IQueryable` を変更するだけのステートメントは含まれません。
* EF コンテキストはスレッド セーフではありません。複数の操作を並列実行しないでください。 非同期 EF メソッドを呼び出すとき、`await` キーワードを常に使用します。
* 非同期コードのパフォーマンス上の利点を利用するには、使用されているライブラリ パッケージで、クエリがデータベースに送信される EF のメソッドが呼び出されている場合は、それらでも非同期を使用するようにします。

.NET の非同期プログラミングについては、「[非同期の概要](/dotnet/articles/standard/async)」を参照してください。

## <a name="limit-entities-fetched"></a>フェッチされるエンティティを制限する

クエリから返される数値またはエンティティの制限については、「[パフォーマンスに関する考慮事項](xref:data/ef-rp/intro)」を参照してください。

基本的な CRUD (作成、読み取り、更新、削除) 操作の実行方法を学習するには、次のチュートリアルに進んでください。

> [!div class="nextstepaction"]
> [基本 CRUD 機能を実装する](crud.md)

::: moniker-end

::: moniker range="<= aspnetcore-3.1"

[!INCLUDE [RP better than MVC](~/includes/RP-EF/rp-over-mvc.md)]

Contoso University のサンプル Web アプリケーションでは、Entity Framework (EF) Core 2.2 と Visual Studio 2017 または 2019 を使用して ASP.NET Core 2.2 MVC Web アプリケーションを作成する方法を示します。

このチュートリアルは、ASP.NET Core 3.1 用に更新されていません。 [ASP.NET Core 5.0](xref:data/ef-mvc/intro?view=aspnetcore-5.0) 用に更新されています。

サンプル アプリケーションは架空の Contoso University の Web サイトです。 学生の受け付け、講座の作成、講師の割り当てなどの機能が含まれています。 これは、Contoso University のサンプル アプリケーションを一から作成する方法を説明するチュートリアル シリーズの 1 回目です。

## <a name="prerequisites"></a>必須コンポーネント

* [.NET Core SDK 2.2](https://dotnet.microsoft.com/download)
* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) と次のワークロード:
  * **ASP.NET および Web の開発** ワークロード
  * **.NET Core クロスプラットフォームの開発** ワークロード

## <a name="troubleshooting"></a>トラブルシューティング

解決できない問題に遭遇した場合、通常、[完成済みのプロジェクト](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples)と自分のコードを比較することで解決策がわかります。 一般的なエラーとその解決方法の一覧については、[このシリーズの最後のチュートリアルにあるトラブルシューティングのセクション](advanced.md#common-errors)をご覧ください。 そこで必要な答えが見つからない場合、StackOverflow.com で [ASP.NET Core](https://stackoverflow.com/questions/tagged/asp.net-core) または [EF Core](https://stackoverflow.com/questions/tagged/entity-framework-core) に関する質問を投稿できます。

> [!TIP]
> これは 10 回のチュートリアルからなるシリーズであり、いずれの回も前のチュートリアルを基盤にしています。 チュートリアルが完了したら、毎回、プロジェクトのコピーを保存するようお勧めします。 問題に遭遇したとき、前のチュートリアルから始めることができます。シリーズ全体の始めまで戻る必要がありません。

## <a name="contoso-university-web-app"></a>Contoso University Web アプリ

一連のチュートリアルで作成するアプリケーションは、簡単な大学向け Web サイトです。

ユーザーは学生、講座、講師の情報を見たり、更新したりできます。 次のような画面をこれから作成します。

![Students インデックス ページ](intro/_static/students-index.png)

![Students 編集ページ](intro/_static/student-edit.png)

## <a name="create-web-app"></a>Web アプリを作成する

* Visual Studio を開きます。

* **[ファイル]** メニューで **[新規作成]、[プロジェクト]** の順に選択します。

* 左側のウィンドウで、 **[インストール済み]、[Visual C#]、[Web]** の順に選択します。

* **[ASP.NET Core Web アプリケーション]** プロジェクト テンプレートを選択します。

* 名前に「**ContosoUniversity**」と入力し、 **[OK]** をクリックします。

  ![[新しいプロジェクト] ダイアログ](intro/_static/new-project2.png)

* **[新しい ASP.NET Core Web アプリケーション]** ダイアログが表示されるのを待ちます。

* **[.NET Core]** 、 **[ASP.NET Core 2.2]** 、および **[Web アプリケーション (モデル ビュー コントローラー)]** テンプレートを選択します。

* **[認証]** に **[認証なし]** が設定されていることを確認してください。

* **[OK]** を選択します。

  ![[新しい ASP.NET Core プロジェクト] ダイアログ](intro/_static/new-aspnet2.png)

## <a name="set-up-the-site-style"></a>サイトのスタイルを設定する

簡単な変更をいくつか行い、サイトのメニュー、レイアウト、ホーム ページを決めます。

*Views/Shared/_Layout.cshtml* を開き、次のように変更します。

* "ContosoUniversity" をすべて "Contoso University" に変更します。 これは 3 回出てきます。

* メニュー エントリとして「**About**」、「**Students**」、「**Courses**」、「**Instructors**」、「**Departments**」を追加し、「**Privacy**」メニュー エントリを削除します。

変更が強調表示されます。

[!code-cshtml[](intro/samples/cu/Views/Shared/_Layout.cshtml?highlight=6,34-48,63)]

*Views/Home/Index.cshtml* で、ファイルの中身を次のコードに変更し、ASP.NET と MVC に関するテキストをこのアプリケーションに関するテキストに変更します。

[!code-cshtml[](intro/samples/cu/Views/Home/Index.cshtml)]

CTRL を押しながら F5 を押してプロジェクトを実行するか、メニューで **[デバッグ]、[デバッグなしで開始]** の順に選択します。 一連のチュートリアルで作成するページのホーム ページとタブが表示されます。

![Contoso University のホーム ページ](intro/_static/home-page.png)

## <a name="about-ef-core-nuget-packages"></a>EF Core の NuGet パッケージについて

プロジェクトに EF Core サポートを追加するには、対象とするデータベース プロバイダーをインストールします。 このチュートリアルでは SQL Server を使用します。プロバイダー パッケージは [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer/) です。 このパッケージは [Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)に含まれているので、パッケージを参照する必要はありません。

EF SQL Server パッケージとその依存関係 (`Microsoft.EntityFrameworkCore` と `Microsoft.EntityFrameworkCore.Relational`) により、EF のランタイム サポートが提供されます。 後の[移行](migrations.md)チュートリアルでツール パッケージを追加します。

Entity Framework Core で利用できるその他のデータベース プロバイダーに関しては、「[データベース プロバイダー](/ef/core/providers/)」を参照してください。

## <a name="create-the-data-model"></a>データ モデルの作成

次に、Contoso University アプリケーションのエンティティ クラスを作成します。 次の 3 つのエンティティから始めます。

![講座、登録、学生からなるデータ モデルの図](intro/_static/data-model-diagram.png)

`Student` エンティティと `Enrollment` エンティティの間に一対多の関係があり、`Course` エンティティと `Enrollment` エンティティの間に一対多の関係があります。 言い換えると、1 人の学生をさまざまな講座に登録し、1 つの講座にたくさんの学生を登録できます。

次のセクションでは、エンティティごとにクラスを作成します。

### <a name="the-student-entity"></a>Student エンティティ

![Student エンティティの図](intro/_static/student-entity.png)

*[Models]* フォルダーで、*Student.cs* という名前のクラス ファイルを作成し、テンプレート コードを次のコードに変更します。

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_Intro)]

`ID` プロパティは、このクラスに相当するデータベース テーブルの主キー列になります。 既定では、Entity Framework は、`ID` または `classnameID` という名前のプロパティを主キーとして解釈します。

`Enrollments` プロパティは[ナビゲーション プロパティ](/ef/core/modeling/relationships)です。 ナビゲーション プロパティには、このエンティティに関連する他のエンティティが含まれます。 この例では、`Student entity` の `Enrollments` プロパティで、その `Student` エンティティに関連するすべての `Enrollment` エンティティが保持されます。 つまり、データベースの `Student` 行に関連する `Enrollment` 行 (StudentID 外部キー列にその学生の主キー値が含まれる行) が 2 つある場合、その `Student` エンティティの `Enrollments` ナビゲーション プロパティには、それら 2 つの `Enrollment` エンティティが含まれます。

ナビゲーション プロパティに複数のエンティティが含まれる場合 (多対多または一対多の関係で)、その型はリストにする必要があります。`ICollection<T>` のように、エンティティを追加、削除、更新できるリストです。 `ICollection<T>`、または `List<T>` や `HashSet<T>` などの型を指定することができます。 `ICollection<T>` を指定した場合、EF では既定で `HashSet<T>` コレクションが作成されます。

### <a name="the-enrollment-entity"></a>Enrollment エンティティ

![Enrollment エンティティの図](intro/_static/enrollment-entity.png)

*[Models]* フォルダーで、*Enrollment.cs* を作成し、既存のコードを次のコードに変更します。

[!code-csharp[](intro/samples/cu/Models/Enrollment.cs?name=snippet_Intro)]

`EnrollmentID` プロパティは主キーになります。このエンティティは、`Student` エンティティと同様に、`ID` ではなく `classnameID` パターンを使用します。 通常、パターンを 1 つ選択し、データ モデル全体でそれを使用します。 ここのバリエーションから、いずれのパターンも利用できることがわかります。 [後のチュートリアル](inheritance.md)では、クラス名なしの ID を利用し、データ モデルに継承を簡単に実装する方法を学習します。

`Grade` プロパティは `enum` です。 `Grade` の型宣言の後の疑問符は、`Grade` プロパティが null 許容であることを示します。 null という成績は、0 の成績とは異なります。null は成績がわからないことか、まだ評価されていないことを意味します。

`StudentID` プロパティは外部キーです。それに対応するナビゲーション プロパティは `Student` です。 `Enrollment` エンティティは 1 つの `Student` エンティティに関連付けられており、1 つの `Student` エンティティだけを保持できます (先に見た、複数の `Enrollment` エンティティを保持できる `Student.Enrollments` ナビゲーション プロパティとは異なります)。

`CourseID` プロパティは外部キーです。それに対応するナビゲーション プロパティは `Course` です。 `Enrollment` エンティティは 1 つの `Course` エンティティに関連付けられます。

Entity Framework は `<navigation property name><primary key property name>` という名前が付いている場合、プロパティを外部キー プロパティとして解釈します。たとえば、`Student` ナビゲーション プロパティの `StudentID` です。`Student` エンティティの主キーが `ID` であるためです。 外部キーにも `<primary key property name>` という単純な名前を付けることができます。たとえば、`CourseID` です。`Course` エンティティの主キーが `CourseID` であるためです。

### <a name="the-course-entity"></a>Course エンティティ

![Course エンティティの図](intro/_static/course-entity.png)

*[Models]* フォルダーで、*Course.cs* を作成し、既存のコードを次のコードに変更します。

[!code-csharp[](intro/samples/cu/Models/Course.cs?name=snippet_Intro)]

`Enrollments` プロパティはナビゲーション プロパティです。 1 つの `Course` エンティティにたくさんの `Enrollment` エンティティを関連付けることができます。

`DatabaseGenerated` 属性については、このシリーズの[後のチュートリアル](complex-data-model.md)で詳しく学習します。 基本的に、この属性によって、講座の主キーをデータベースに生成させず、自分で入力できるようになります。

## <a name="create-the-database-context"></a>データベース コンテキストの作成

所与のデータ モデルの Entity Framework 機能を調整するメイン クラスは、データベース コンテキスト クラスです。 このクラスは、`Microsoft.EntityFrameworkCore.DbContext` クラスから派生させて作成します。 自分のコードでは、データ モデルに含めるエンティティを自分で指定します。 Entity Framework の特定の動作をカスタマイズすることもできます。 このプロジェクトでは、クラスに `SchoolContext` という名前が付けられています。

プロジェクト フォルダーで、*Data* という名前のフォルダーを作成します。

*[Data]* フォルダーで、*SchoolContext.cs* という名前の新しいクラス ファイルを作成し、テンプレート コードを次のコードに変更します。

[!code-csharp[](intro/samples/cu/Data/SchoolContext.cs?name=snippet_Intro)]

このコードによって、エンティティ セットごとに `DbSet` プロパティが作成されます。 Entity Framework の用語では、エンティティ セットは通常はデータベース テーブルに対応し、エンティティはテーブルの行に対応します。

`DbSet<Enrollment>` ステートメントと `DbSet<Course>` ステートメントは省略しても同じ動作をします。 Entity Framework にはそれらが暗黙的に含まれることがあります。`Student` エンティティが `Enrollment` エンティティを参照し、`Enrollment` エンティティが `Course` エンティティを参照するためです。

データベースが作成されると、EF によって、`DbSet` プロパティと同じ名前を持つテーブルが作成されます。 一般的にコレクションのプロパティ名は複数形 (Student ではなく、Students) ですが、テーブル名を複数にするかどうかについては、開発者の間で意見が分かれています。 このチュートリアル シリーズでは、DbContext に単数のテーブル名を指定して既定の動作をオーバーライドします。 そのために、最後の DbSet プロパティの後に、次の強調表示されているコードを追加します。

[!code-csharp[](intro/samples/cu/Data/SchoolContext.cs?name=snippet_TableNames&highlight=16-21)]

コンパイラ エラーのチェックとしてプロジェクトをビルドします。

## <a name="register-the-schoolcontext"></a>SchoolContext を登録する

ASP.NET Core は既定で[依存関係の挿入](../../fundamentals/dependency-injection.md)を実装します。 サービス (EF データベース コンテキストなど) は、アプリケーションの起動時に依存関係の挿入に登録されます。 これらのサービス (MVC コント ローラーなど) を必要とするコンポーネントには、コンストラクターのパラメーターを介してこれらのサービスが指定されます。 このチュートリアルの後半で、コンテキスト インスタンスを取得するコントローラー コンストラクター コードが登場します。

`SchoolContext` をサービスとして登録するには、*Startup.cs* を開き、強調表示されている行を `ConfigureServices` メソッドに追加します。

[!code-csharp[](intro/samples/cu/Startup.cs?name=snippet_SchoolContext&highlight=9-10)]

`DbContextOptionsBuilder` オブジェクトでメソッドが呼び出され、接続文字列の名前がコンテキストに渡されます。 ローカル開発の場合、[ASP.NET Core 構成システム](xref:fundamentals/configuration/index)によって *appsettings.json* ファイルから接続文字列が読み取られます。

名前空間の `ContosoUniversity.Data` と `Microsoft.EntityFrameworkCore` に対して `using` ステートメントを追加し、プロジェクトをビルドします。

[!code-csharp[](intro/samples/cu/Startup.cs?name=snippet_Usings)]

*appsettings.json* ファイルを開き、次の例で示されているように接続文字列を追加します。

[!code-json[](./intro/samples/cu/appsettings1.json?highlight=2-4)]

### <a name="sql-server-express-localdb"></a>SQL Server Express LocalDB

この接続文字列によって SQL Server LocalDB データベースが指定されます。 LocalDB は SQL Server Express データベース エンジンの軽量版であり、実稼働ではなく、アプリケーションの開発を意図して設計されています。 LocalDB は要求時に開始され、ユーザー モードで実行されるため、複雑な構成はありません。 既定では、LocalDB は `C:/Users/<user>` ディレクトリに *.mdf* データベース ファイルを作成します。

## <a name="initialize-db-with-test-data"></a>テスト データで DB を初期化する

Entity Framework によって空のデータベースが自動的に作成されます。 このセクションでは、テスト データを入力する目的で、データベースの作成後に呼び出されるメソッドを記述します。

ここでは、データベースを自動的に作成する `EnsureCreated` メソッドを使用します。 [後のチュートリアル](migrations.md)では、モデル変更の処理方法について学習します。データベースを削除し、再作成するのではなく、Code First Migrations を利用してデータベース スキーマを変更します。

*[データ]* フォルダーで *DbInitializer.cs* という名前の新しいクラス ファイルを作成し、テンプレート コードを次のコードに変更します。このコードにより、必要なときにデータベースが作成され、新しいデータベースにテスト データが読み込まれます。

[!code-csharp[](intro/samples/cu/Data/DbInitializer.cs?name=snippet_Intro)]

このコードはデータベースに学生が存在するかどうかを確認し、存在しない場合、そのデータベースは新しく、テスト データを入力する必要があると見なします。 `List<T>` コレクションではなく配列にテスト データを読み込み、パフォーマンスを最適化します。

*Program.cs* で、アプリケーションの起動時に次を実行するように `Main` メソッドを変更します。

* 依存関係挿入コンテナーからデータベース コンテキスト インスタンスを取得します。
* seed メソッドを呼び出し、コンテキストを渡します。
* seed メソッドが完了したら、コンテキストを破棄します。

[!code-csharp[](intro/samples/5cu-snap/Program.cs?highlight=1-2,14-18,21-38)]

アプリケーションを初めて実行すると、データベースが作成され、テスト データが設定されます。 データ モデルを変更するたび:

 * データベースを削除します。
 * シード メソッドを更新し、同じようにして新しいデータベースで新たに開始します。
 
後のチュートリアルでは、データ モデルが変わったとき、データベースを削除して作り直すのではなく、修正する方法について学習します。

## <a name="create-controller-and-views"></a>コントローラーとビューを作成する

このセクションでは、Visual Studio のスキャフォールディング エンジンを使用して、MVC のコントローラーとビューを追加します。それらにより、EF を使用して、クエリが実行され、データが保存されます。

CRUD アクションのメソッドとビューの自動作成は、スキャフォールディングと言います。 一般的には生成されたコードは修正しないのに対し、スキャフォールディングされたコードを開始点として独自の要件に合うように変更できるという点で、スキャフォールディングはコード生成と異なります。 生成されたコードをカスタマイズする必要があるとき、部分クラスを利用するか、状況が変わったときにコードを再生成します。

* **ソリューション エクスプローラー** の **Controllers** フォルダーを右クリックし、 **[追加]、[スキャフォールディングされた新しい項目]** の順に選択します。
* **[スキャフォールディングを追加]** ダイアログ ボックスで:
  * **[Entity Framework を使用したビューがある MVC コントローラー]** を選択します。
  * **[追加]** をクリックします。 **[Add MVC Controller with views, using Entity Framework]\(Entity Framework を使用したビューがある MVC コントローラーを追加する\)** ダイアログ ボックスが表示されます。![Student のスキャフォールディング](intro/_static/scaffold-student2.png)
  * **[モデル クラス]** で **[Student]** を選択します。
  * **[データ コンテキスト クラス]** で **[SchoolContext]** を選択します。
  * 名前は **StudentsController** をそのまま選択します。
  * **[追加]** をクリックします。

Visual Studio スキャフォールディング エンジンにより、*StudentsController.cs* ファイルと、コントローラーで動作するビューのセット ( *.cshtml* ファイル) が作成されます。

コントローラーによりコンストラクター パラメーターとして `SchoolContext` が受け取られることに注意してください。

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_Context&highlight=5,7,9)]

ASP.NET Core 依存関係挿入では、`SchoolContext` のインスタンスがコントローラーに渡されます。 *Startup.cs* ファイルでそれを構成しました。

コントローラーには `Index` アクション メソッドが含まれます。これはデータベースにあるすべての学生を表示します。 このメソッドはデータベース コンテキスト インスタンスの `Students` プロパティを読み取り、Students エンティティ セットから学生の一覧を取得します。

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_ScaffoldedIndex&highlight=3)]

このコードの非同期プログラミング要素については、チュートリアルで後ほど学習します。

*Views/Students/Index.cshtml* ビューには、この一覧で表形式で表示されます。

[!code-cshtml[](intro/samples/cu/Views/Students/Index1.cshtml)]

CTRL を押しながら F5 を押してプロジェクトを実行するか、メニューで **[デバッグ]、[デバッグなしで開始]** の順に選択します。

[Students] タブをクリックすると、`DbInitializer.Initialize` メソッドによって挿入されたテスト データが表示されます。 ブラウザーのウィンドウ幅によって決まることですが、`Students` タブ リンクはページの一番上に表示されるか、右上隅のナビゲーション アイコンをクリックしないと表示されません。

![Contoso University のホーム ページ (ウィンドウ幅が狭いとき)](intro/_static/home-page-narrow.png)

![Students インデックス ページ](intro/_static/students-index.png)

## <a name="view-the-database"></a>データベースを表示する

アプリケーションを起動する葉、`DbInitializer.Initialize` メソッドが `EnsureCreated` を呼び出します。 EF はデータベースがないことを認識し、作成します。`Initialize` メソッド コードの残りの部分により、データベースにデータが入力されます。 **SQL Server Object Explorer** (SSOX) を利用し、Visual Studio でデータベースを表示できます。

ブラウザーを閉じます。

SSOX ウィンドウがまだ開いていない場合、Visual Studio の **[表示]** メニューから選択します。

SSOX で **(localdb)\MSSQLLocalDB > [データベース]** をクリックし、 *appsettings.json* ファイルの接続文字列にあるデータベース名のエントリをクリックします。

**[テーブル]** ノードを展開し、データベースのテーブルを表示します。

![SSOX のテーブル](intro/_static/ssox-tables.png)

**[Student]** テーブルを右クリックし、 **[データの表示]** をクリックすると、作成された列とテーブルに挿入された行が表示されます。

![SSOX の Student テーブル](intro/_static/ssox-student-table.png)

*.mdf* データベース ファイルと *.ldf* データベース ファイルは *C:\Users\\\<username>* フォルダーにあります。

アプリの起動時に実行される初期化子メソッドで `EnsureCreated` を呼び出すため、`Student` クラスを変更し、データベースを削除し、アプリケーションを再実行できます。変更に合わせてデータベースが自動的に再作成されます。 たとえば、`Student` クラスに `EmailAddress` プロパティを追加する場合、再作成されたテーブルに新しい `EmailAddress` 列が表示されます。

## <a name="conventions"></a>規約

規約を利用することや Entity Framework が想定を行うことにより、Entity Framework が完全なデータベースを自動作成するために記述しなければならないコードの量が最小限に抑えられます。

* `DbSet` プロパティの名前がテーブル名として使用されます。 `DbSet` プロパティによって参照されないエンティティについては、エンティティ クラス名がテーブル名として使用されます。
* 列名には、エンティティ プロパティ名が使用されます。
* ID または classnameID という名前が付けられているエンティティ プロパティは主キーのプロパティとして認識されます。
* *\<navigation property name>\<primary key property name>* という名前が付いている場合、プロパティは外部キー プロパティとして解釈されます。たとえば、`Student` ナビゲーション プロパティの `StudentID` です。`Student` エンティティの主キーが `ID` であるためです。 外部キーにも *\<primary key property name>* という単純な名前を付けることができます。たとえば、`EnrollmentID` です。`Enrollment` エンティティの主キーが `EnrollmentID` であるためです。

従来の動作をオーバーライドできます。 たとえば、このチュートリアルで先に見たように、テーブル名を明示的に指定できます。 また、列名を設定し、任意のプロパティを主キーまたは外部キーとして設定できます。これについては、このシリーズの[後のチュートリアル](complex-data-model.md)で学習します。

## <a name="asynchronous-code"></a>非同期コード

ASP.NET Core と EF Core では、非同期プログラミングが既定のモードです。

Web サーバーでは、利用できるスレッド数に限りがあります。負荷が高い状況では、利用できるスレッドが全部使われる可能性があります。 その場合、スレッドが解放されるまでサーバーは新しい要求を処理できません。 同期コードの場合、たくさんのスレッドが関連付けられていても、I/O の完了を待っているため、実際には何の作業も行っていないということがあります。 非同期コードの場合、あるプロセスが I/O の完了を待っているとき、そのスレッドは解放され、サーバーによって他の要求の処理に利用できます。 結果として、非同期コードの場合、サーバー リソースをより効率的に利用できます。サーバーは、より多くのトラフィックを遅延なく処理できます。

非同期コードは実行時に少量のオーバーヘッドを発生させるが、トラフィックが少ない場合、パフォーマンスに与える影響は無視して構わない程度です。トラフィックが多い場合、相当なパフォーマンス改善が見込まれます。

次のコードでは、キーワード `async`、戻り値 `Task<T>`、キーワード `await`、メソッド `ToListAsync` によりコードの実行が非同期になります。

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_ScaffoldedIndex)]

* キーワード `async` は、メソッド本文の一部にコールバックを生成し、返された `Task<IActionResult>` オブジェクトを自動作成するようにコンパイラに伝えます。
* 戻り値の型 `Task<IActionResult>` は、進行中の作業と型 `IActionResult` の結果を表します。
* キーワード `await` により、コンパイラはメソッドを 2 つに分割します。 最初の部分は、非同期で開始される操作で終わります。 2 つ目の部分は、操作の完了時に呼び出されるコールバック メソッドに入ります。
* `ToListAsync` は、`ToList` 拡張メソッドの非同期バージョンです。

Entity Framework を利用する非同期コードの記述で注意すべき点:

* クエリやコマンドをデータベースに送信するステートメントのみが非同期で実行されます。 たとえば、`ToListAsync`、`SingleOrDefaultAsync`、`SaveChangesAsync` などです。 `var students = context.Students.Where(s => s.LastName == "Davolio")` など、`IQueryable` を変更するだけのステートメントは含まれません。
* EF コンテキストはスレッド セーフではありません。複数の操作を並列実行しないでください。 非同期 EF メソッドを呼び出すとき、`await` キーワードを常に使用します。
* 非同期コードのパフォーマンス上の利点を最大限に活用する場合、(ページングなどのために) ライブラリ パッケージを利用しているのであれば、それがクエリをデータベースに送信させる Entity Framework メソッドを呼び出す場合、非同期を利用する必要があります。

.NET の非同期プログラミングについては、「[非同期の概要](/dotnet/articles/standard/async)」を参照してください。

## <a name="next-steps"></a>次のステップ

基本的な CRUD (作成、読み取り、更新、削除) 操作の実行方法を学習するには、次のチュートリアルに進んでください。

> [!div class="nextstepaction"]
> [基本 CRUD 機能を実装する](crud.md)

::: moniker-end
