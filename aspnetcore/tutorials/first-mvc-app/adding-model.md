---
title: パート 4、ASP.NET Core MVC アプリにモデルを追加する
author: rick-anderson
description: ASP.NET Core MVC のチュートリアル シリーズのパート 4。
ms.author: riande
ms.date: 11/16/2020
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
uid: tutorials/first-mvc-app/adding-model
ms.openlocfilehash: 8b3e88ef42914c409a7c3319945e6bc086e6b85a
ms.sourcegitcommit: 1f35de0ca9ba13ea63186c4dc387db4fb8e541e0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2021
ms.locfileid: "104711633"
---
# <a name="part-4-add-a-model-to-an-aspnet-core-mvc-app"></a>パート 4、ASP.NET Core MVC アプリにモデルを追加する

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT) および [Tom Dykstra](https://github.com/tdykstra)

このセクションでは、データベースのムービーを管理するクラスを追加します。 これらのクラスは、**M** VC アプリの "**モ** デル" 部分です。

[Entity Framework Core](/ef/core) (EF Core) でこれらのクラスを使用して、データベースを操作します。 EF Core は、記述する必要があるデータ アクセス コードを簡略化するオブジェクト リレーショナル マッピング (ORM) フレームワークです。

作成するモデル クラスは、EF Core に対する依存関係がないために、POCO クラス (**P** lain-**O** ld **C** LR **O** bjects から) と呼ばれます。 これらは単に、データベースに格納されるデータのプロパティを定義します。

このチュートリアルでは、まずモデル クラスを記述し、EF コアによってデータベースが作成されます。

::: moniker range=">= aspnetcore-5.0"

## <a name="add-a-data-model-class"></a>データ モデル クラスの追加

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

*Models* フォルダーを右クリックし、 **[追加]**  >  **[クラス]** の順に選択します。 ファイルに *Movie.cs* という名前を付けます。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

*Movie.cs* という名前のファイルを *Models* フォルダーに追加します。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

*Models* フォルダーを右クリックし、 **[追加]**  >  **[新しいクラス]**  >  **[空のクラス]** の順に選択します。 ファイルに *Movie.cs* という名前を付けます。

---

次のコードで *Movie.cs* ファイルを更新します。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Models/Movie.cs)]

`Movie` クラスには、データベースで主キー用に必要となる `Id` フィールドが含まれています。

`ReleaseDate` の <xref:System.ComponentModel.DataAnnotations.DataType> 属性により、データの型 (`Date`) が指定されます。 この属性を使用する場合:

* ユーザーは日付フィールドに時刻の情報を入力する必要はありません。
* 日付のみが表示され、時刻の情報は表示されません。

[DataAnnotations](/dotnet/api/system.componentmodel.dataannotations) は、後のチュートリアルで説明されます。

## <a name="add-nuget-packages"></a>NuGet パッケージを追加する

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

**[ツール]** メニューで、 **[NuGet パッケージ マネージャー]** > **[パッケージ マネージャー コンソール]** (PMC) の順に選択します。

![PMC メニュー](~/tutorials/first-mvc-app/adding-model/_static/pmc.png)

PMC で次のコマンドを実行します。

```powershell
Install-Package Microsoft.EntityFrameworkCore.SqlServer
```

上記のコマンドによって EF Core SQL Server プロバイダーが追加されます。 プロバイダー パッケージによって、依存関係として EF Core パッケージがインストールされます。 追加のパッケージは、このチュートリアルの後半で、スキャフォールディングの手順で自動的にインストールされます。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/add-EF-NuGet-SQLite-CLI-5.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

**[プロジェクト]** メニューから、 **[NuGet パッケージの管理]** を選択します。

右上の **[検索]** フィールドに「`Microsoft.EntityFrameworkCore.SQLite`」と入力し、**Return** キーを押して検索します。 一致する NuGet パッケージを選択し、 **[パッケージの追加]** ボタンをクリックします。

![Entity Framework Core NuGet パッケージを追加する](~/tutorials/first-mvc-app-mac/adding-model/_static/add-nuget-packages.png)

**[プロジェクトの選択]** ダイアログが、`MvcMovie` プロジェクトが選択された状態で表示されます。 **[OK]** ボタンをクリックします。

**[ライセンスの同意]** ダイアログが表示されます。 必要に応じてライセンスを確認し、 **[同意する]** ボタンをクリックします。

上記の手順を繰り返して、次の NuGet パッケージをインストールします。

* `Microsoft.VisualStudio.Web.CodeGeneration.Design`
* `Microsoft.EntityFrameworkCore.SqlServer`
* `Microsoft.EntityFrameworkCore.Design`

次の .NET CLI コマンドを実行します。

```dotnetcli
dotnet tool install --global dotnet-aspnet-codegenerator
```

上記のコマンドを実行すると、[aspnet-codegenerator スキャフォールディング ツール](xref:fundamentals/tools/dotnet-aspnet-codegenerator)が追加されます。

---

<a name="dc"></a>

## <a name="create-a-database-context-class"></a>データベース コンテキスト クラスを作成する

データベース コンテキスト クラスは、`Movie` モデルの EF Core 機能 (作成、読み取り、更新、削除) を調整するために必要です。 データベース コンテキストは [Microsoft.EntityFrameworkCore.DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext) から派生し、データ モデルに含めるエンティティを指定します。

*Data* フォルダーを作成します。

次のコードで *Data/MvcMovieContext.cs* ファイルを追加します。 

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/zDocOnly/MvcMovieContext.cs?name=snippet)]

上記のコードによって、エンティティ セットの [DbSet\<Movie>](/dotnet/api/microsoft.entityframeworkcore.dbset-1) プロパティが作成されます。 Entity Framework の用語では、エンティティ セットは通常はデータベース テーブルに対応します。 エンティティはテーブル内の行に対応します。

<a name="reg"></a>

## <a name="register-the-database-context"></a>データベース コンテキストの登録

ASP.NET Core には、[依存関係挿入 (DI)](xref:fundamentals/dependency-injection) が組み込まれています。 サービス (EF Core DB コンテキストなど) は、アプリケーションの起動時に DI に登録する必要があります。 これらのサービスを必要とするコンポーネント (Razor Pages など) には、コンストラクターのパラメーターを介してこれらのサービスが指定されます。 DB コンテキスト インスタンスを取得するコンストラクター コードは、チュートリアルの後半で示します。 このセクションでは、DI コンテナーにデータベース コンテキストを登録します。

*Startup.cs* の先頭に次の `using` ステートメントを追加します。

```csharp
using MvcMovie.Data;
using Microsoft.EntityFrameworkCore;
```

`Startup.ConfigureServices` で次の強調表示されたコードを追加します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Startup.cs?name=snippet_UseSqlite&highlight=5-6)]

---

[DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions) オブジェクトでメソッドが呼び出され、接続文字列の名前がコンテキストに渡されます。 ローカル開発の場合、[ASP.NET Core 構成システム](xref:fundamentals/configuration/index)によって *appsettings.json* ファイルから接続文字列が読み取られます。

<a name="cs"></a>

## <a name="add-a-database-connection-string"></a>データベース接続文字列の追加

接続文字列を *appsettings.json* ファイルに追加します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-json[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/appsettings.json?highlight=10-12)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-json[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/appsettings_SQLite.json?highlight=10-12)]

---

コンパイラ エラーのチェックとしてプロジェクトをビルドします。

## <a name="scaffold-movie-pages"></a>ムービー ページのスキャフォールディング

スキャフォールディング ツールを使用し、ムービー モデルの作成、読み取り、更新、削除の (CRUD) ページを生成します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

*ソリューション エクスプローラー* で、**Controllers** フォルダーを右クリックし、 **[追加]、[スキャフォールディングされた新しい項目]** の順に選択します。

![前述の手順を参照](adding-model/_static/add_controller21.png)

**[スキャフォールディングを追加]** ダイアログで、 **[Entity Framework を使用したビューがある MVC コントローラー]、[追加]** の順に選択します。

![[スキャフォールディングを追加] ダイアログ](adding-model/_static/add_scaffold21.png)

**[コントローラーの追加]** ダイアログ ボックスを完了します。

* **モデル クラス:** *Movie (MvcMovie.Models)*
* **データ コンテキスト クラス:** *MvcMovieContext (MvcMovie.Data)*

![[データの追加] コンテキスト](adding-model/_static/dc5.png)

* **ビュー:** 各オプションの既定値をオンにします。
* **コントローラー名:** 既定の *MoviesController* のままにします。
* **[追加]** を選択します。

Visual Studio では、次が作成されます。

* ムービー コントローラー (*Controllers/MoviesController.cs*)
* 作成、削除、詳細、編集、およびインデックス ページ用の Razor ビュー ファイル (*Views/Movies/\*.cshtml*)

このようなファイルの自動作成は、"*スキャフォールディング*" と呼ばれます。

### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code) 

* プロジェクト ディレクトリ (*Program.cs*、*Startup.cs*、および *.csproj* ファイルを含むディレクトリ) でコマンド ウィンドウを開きます。

* Linux で、スキャフォールディング ツールのパスをエクスポートします。

  ```console
  export PATH=$HOME/.dotnet/tools:$PATH
  ```

* 次のコマンドを実行します。

  ```dotnetcli
  dotnet aspnet-codegenerator controller -name MoviesController -m Movie -dc MvcMovieContext --relativeFolderPath Controllers --useDefaultLayout --referenceScriptLibraries
  ```

  [!INCLUDE [explains scaffold generated params](~/includes/mvc-intro/model4.md)]

### <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* プロジェクト ディレクトリ (*Program.cs*、*Startup.cs*、および *.csproj* ファイルを含むディレクトリ) でコマンド ウィンドウを開きます。

* スキャフォールディング ツールのパスをエクスポートします。

  ```console
  export PATH=$HOME/.dotnet/tools:$PATH
  ```

* 次のコマンドを実行します。

  ```dotnetcli
  dotnet aspnet-codegenerator controller -name MoviesController -m Movie -dc MvcMovieContext --relativeFolderPath Controllers --useDefaultLayout --referenceScriptLibraries
  ```

  [!INCLUDE [explains scaffold generated params](~/includes/mvc-intro/model4.md)]

---

<!-- End of tabs                  -->

データベースが存在しないため、スキャフォールディング ページをまだ使用できません。 アプリを実行し、 **[Movie App]** リンクをクリックすると、 *[データベースを開けません]* または *[そのようなテーブルはありません:Movie*] というエラー メッセージが表示されます。

<a name="migration"></a>

## <a name="initial-migration"></a>最初の移行

EF Core [移行](xref:data/ef-mvc/migrations)機能を使用し、データベースを作成します。 移行は、データ モデルに合わせてデータベースを作成したり、更新したりできる一連のツールです。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

**[ツール]** メニューで、 **[NuGet パッケージ マネージャー]** > **[パッケージ マネージャー コンソール]** (PMC) の順に選択します。

PMC で、次のコマンドを入力します。

```powershell
Add-Migration InitialCreate
Update-Database
```

* `Add-Migration InitialCreate`:*Migrations/{timestamp}_InitialCreate.cs* 移行ファイルが生成されます。 `InitialCreate` 引数は、移行の名前です。 任意の名前を使用できますが、慣例により、移行を説明する名前が選択されます。 これは最初の移行であるため、生成されたクラスには、データベース スキーマを作成するコードが含まれています。 データベース スキーマは、`MvcMovieContext` クラスで指定されたモデルに基づきます。

* `Update-Database`:前のコマンドで作成された最新の移行にデータベースを更新します。 このコマンドにより *Migrations/{time-stamp}_InitialCreate.cs* ファイルで `Up` メソッドが実行され、データベースが作成されます。

  データベース更新コマンドにより、次の警告が生成されます。 

  > エンティティ型 'Movie' の decimal 列 'Price' に型が指定されていません。 これにより、値が既定の有効桁数と小数点以下桁数に収まらない場合、自動的に切り捨てられます。 'HasColumnType()' を使用してすべての値に適合する SQL server 列の型を明示的に指定します。

  この警告は無視して構いません。後のチュートリアルで修正されます。

[!INCLUDE [more information on the PMC tools for EF Core](~/includes/ef-pmc.md)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!INCLUDE [more information on the CLI for EF Core](~/includes/ef-cli.md)]

次の .NET Core CLI コマンドを実行します。

```dotnetcli
dotnet ef migrations add InitialCreate
dotnet ef database update
```

* `ef migrations add InitialCreate`:*Migrations/{timestamp}_InitialCreate.cs* 移行ファイルが生成されます。 `InitialCreate` 引数は、移行の名前です。 任意の名前を使用できますが、慣例により、移行を説明する名前が選択されます。 これは最初の移行であるため、生成されたクラスには、データベース スキーマを作成するコードが含まれています。 データベース スキーマは、`MvcMovieContext` クラスで指定されたモデルに基づきます (*Data/MvcMovieContext.cs* ファイル内)。

* `ef database update`:前のコマンドで作成された最新の移行にデータベースを更新します。 このコマンドにより *Migrations/{time-stamp}_InitialCreate.cs* ファイルで `Up` メソッドが実行され、データベースが作成されます。

---

### <a name="the-initialcreate-class"></a>InitialCreate クラス

*Migrations/{timestamp}_InitialCreate.cs* 移行ファイルを調べます。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Migrations/20190805165915_InitialCreate.cs?name=snippet)]

`Up` メソッドにより Movie テーブルが作成され、主キーとして `Id` が構成されます。 `Down` メソッドにより、`Up` 移行で行われたスキーマ変更が元に戻ります。

<a name="test"></a>

## <a name="test-the-app"></a>アプリのテスト

* アプリを実行し、 **[Movie App]** リンクをクリックします。

  次のような例外が表示された場合:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

  ```console
  SqlException: Cannot open database "MvcMovieContext-1" requested by the login. The login failed.
  ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

  ```console
  SqliteException: SQLite Error 1: 'no such table: Movie'.
  ```

---
  [移行手順](#migration)を実行しなかった可能性があります。

* **Create** ページをテストします。 データを入力して送信します。

  > [!NOTE]
  > `Price` フィールドに小数点のコンマを入力できない場合があります。 小数点にコンマ (",") を使う英語以外のロケール、および英語 (米国) 以外の日付形式で、[jQuery 検証](https://jqueryvalidation.org/)をサポートするには、アプリをグローバル化する必要があります。 グローバル化の手順については、[この GitHub の記事](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420)をご覧ください。

* **[編集]** 、 **[詳細]** 、 **[削除]** の各ページをテストします。

## <a name="dependency-injection-in-the-controller"></a>コントローラーの依存関係挿入

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

*Controllers/MoviesController.cs* ファイルを開いて、コンストラクターを調べます。

<!-- l.. Make copy of Movies controller (or use the old one as I did in the 3.0 upgrade) because we comment out the initial index method and update it later  -->

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_1)]

コンストラクターでは、[依存性の注入](xref:fundamentals/dependency-injection)を使ってデータベース コンテキスト (`MvcMovieContext`) がコントローラーに挿入されています。 データベース コンテキストは、コントローラーの各 [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) メソッドで使用されます。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_1)]

コンストラクターでは、[依存性の注入](xref:fundamentals/dependency-injection)を使ってデータベース コンテキスト (`MvcMovieContext`) がコントローラーに挿入されています。 データベース コンテキストは、コントローラーの各 [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) メソッドで使用されます。

<a name="sqlite-dev"></a>

### <a name="use-sqlite-for-development-sql-server-for-production"></a>開発用に SQLite を、運用環境に SQL Server を使用する

SQLite が選択されている場合、テンプレートで生成されたコードは開発の準備ができています。 次のコードでは、スタートアップに <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> を挿入する方法を示します。 `IWebHostEnvironment` が挿入されているので、`ConfigureServices` では開発用に SQLite を、運用環境に SQL Server を使用できます。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/StartupDevProd.cs?name=snippet_StartupClass&highlight=5,10,16-28)]

---
<!-- end of tabs --->

<a name="strongly-typed-models-keyword-label"></a>
<a name="strongly-typed-models-and-the--keyword"></a>

## <a name="strongly-typed-models-and-the-model-keyword"></a>厳密に型指定されたモデルと @model キーワード

コントローラーで `ViewData` ディクショナリを使ってビューにデータまたはオブジェクトを渡す方法を前に示しました。 `ViewData` ディクショナリは動的オブジェクトであり、ビューに情報を渡すための便利な遅延バインディングの方法を提供します。

MVC にも、厳密に型指定されたモデル オブジェクトをビューに渡す機能があります。 この厳密に型指定された方法では、コンパイル時にコードを確認できます。 スキャフォールディング メカニズムでは、`MoviesController` のクラスとビューで、この手法 (つまり、厳密に型指定されたモデルを渡しました) が使用されました。

*Controllers/MoviesController.cs* ファイルで生成された `Details` メソッドを調べてください。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_details)]

通常、`id` パラメーターはルート データとして渡されます。 たとえば、`https://localhost:5001/movies/details/1` は次のように設定します。

* コントローラーを `movies` コントローラーに (最初の URL セグメント)。
* アクションを `details` に (2 番目の URL セグメント)。
* ID を 1 に (最後の URL セグメント)。

次のようにクエリ文字列で `id` を渡すこともできます。

`https://localhost:5001/movies/details?id=1`

ID 値が指定されない場合、`id` パラメーターは [null 許容型](/dotnet/csharp/programming-guide/nullable-types/index) (`int?`) として定義されます。

ルート データまたはクエリ文字列の値と一致するムービー エンティティを選択するため、[ラムダ式](/dotnet/articles/csharp/programming-guide/statements-expressions-operators/lambda-expressions)が `FirstOrDefaultAsync` に渡されます。

```csharp
var movie = await _context.Movie
    .FirstOrDefaultAsync(m => m.Id == id);
```

ムービーが見つかった場合、`Movie` モデルのインスタンスが `Details` ビューに渡されます。

```csharp
return View(movie);
```

*Views/Movies/Details.cshtml* ファイルの内容を確認してください。

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/DetailsOriginal.cshtml)]

ビュー ファイルの一番上にある `@model` ステートメントにより、ビューで求められるオブジェクトの型が指定されます。 ムービー コントローラーが作成されたとき、次の `@model` ステートメントが含まれました。

```cshtml
@model MvcMovie.Models.Movie
```

この `@model` ディレクティブにより、コントローラーでビューに渡されたムービーにアクセスできます。 `Model` オブジェクトは厳密に型指定されます。 たとえば、*Details.cshtml* ビューでは、コードで厳密に型指定された `Model` オブジェクトを使って、`DisplayNameFor` および `DisplayFor` HTML ヘルパーに各ムービー フィールドを渡しています。 `Create` および `Edit` のメソッドとビューも、`Movie` モデル オブジェクトを渡します。

Movies コントローラーの *Index.cshtml* ビューと `Index` メソッドを確認してください。 コードで `View` メソッドを呼び出すときの `List` オブジェクトの作成方法に注意してください。 コードでは、この `Movies` リストを `Index` アクション メソッドからビューに渡しています。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_index)]

ムービー コントローラーが作成されたとき、スキャフォールディングにより、*Index.cshtml* ファイルの一番上に次の `@model` ステートメントが含まれました。

<!-- Copy Index.cshtml to IndexOriginal.cshtml -->

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/IndexOriginal.cshtml?range=1)]

`@model` ディレクティブにより、厳密に型指定された `Model` オブジェクトを使って、コントローラーがビューに渡したムービーのリストにアクセスできます。 たとえば、*Index.cshtml* ビューのコードでは、`foreach` ステートメントを使って厳密に型指定された `Model` オブジェクトのムービーをループ処理しています。

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/IndexOriginal.cshtml?highlight=1,31,34,37,40,43,46-48)]

`Model` オブジェクトは厳密に型指定されているので (`IEnumerable<Movie>` オブジェクトとして)、ループ内の各項目は `Movie` として型指定されます。 これには、コンパイル時にコードを確認できるなどの利点があります。

## <a name="additional-resources"></a>その他の技術情報

* [タグ ヘルパー](xref:mvc/views/tag-helpers/intro)
* [グローバライズとローカライズ](xref:fundamentals/localization)

> [!div class="step-by-step"]
> [前のチュートリアル ビューの追加](adding-view.md)
> [次のチュートリアル SQL の使用](working-with-sql.md)

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

## <a name="add-a-data-model-class"></a>データ モデル クラスの追加

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

*Models* フォルダーを右クリックし、 **[追加]**  >  **[クラス]** の順に選択します。 ファイルに *Movie.cs* という名前を付けます。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

*Movie.cs* という名前のファイルを *Models* フォルダーに追加します。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

*Models* フォルダーを右クリックし、 **[追加]**  >  **[新しいクラス]**  >  **[空のクラス]** の順に選択します。 ファイルに *Movie.cs* という名前を付けます。

---

次のコードで *Movie.cs* ファイルを更新します。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Models/Movie.cs)]

`Movie` クラスには、データベースで主キー用に必要となる `Id` フィールドが含まれています。

`ReleaseDate` の <xref:System.ComponentModel.DataAnnotations.DataType> 属性により、データの型 (`Date`) が指定されます。 この属性を使用する場合:

* ユーザーは日付フィールドに時刻の情報を入力する必要はありません。
* 日付のみが表示され、時刻の情報は表示されません。

[DataAnnotations](/dotnet/api/system.componentmodel.dataannotations) は、後のチュートリアルで説明されます。

## <a name="add-nuget-packages"></a>NuGet パッケージを追加する

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

**[ツール]** メニューで、 **[NuGet パッケージ マネージャー]** > **[パッケージ マネージャー コンソール]** (PMC) の順に選択します。

![PMC メニュー](~/tutorials/first-mvc-app/adding-model/_static/pmc.png)

PMC で次のコマンドを実行します。

```powershell
Install-Package Microsoft.EntityFrameworkCore.SqlServer
```

上記のコマンドによって EF Core SQL Server プロバイダーが追加されます。 プロバイダー パッケージによって、依存関係として EF Core パッケージがインストールされます。 追加のパッケージは、このチュートリアルの後半で、スキャフォールディングの手順で自動的にインストールされます。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/add-EF-NuGet-SQLite-CLI.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

**[プロジェクト]** メニューから、 **[NuGet パッケージの管理]** を選択します。

右上の **[検索]** フィールドに「`Microsoft.EntityFrameworkCore.SQLite`」と入力し、**Return** キーを押して検索します。 一致する NuGet パッケージを選択し、 **[パッケージの追加]** ボタンをクリックします。

![Entity Framework Core NuGet パッケージを追加する](~/tutorials/first-mvc-app-mac/adding-model/_static/add-nuget-packages.png)

**[プロジェクトの選択]** ダイアログが、`MvcMovie` プロジェクトが選択された状態で表示されます。 **[OK]** ボタンをクリックします。

**[ライセンスの同意]** ダイアログが表示されます。 必要に応じてライセンスを確認し、 **[同意する]** ボタンをクリックします。

上記の手順を繰り返して、次の NuGet パッケージをインストールします。

* `Microsoft.VisualStudio.Web.CodeGeneration.Design`
* `Microsoft.EntityFrameworkCore.SqlServer`
* `Microsoft.EntityFrameworkCore.Design`

---

<a name="dc"></a>

## <a name="create-a-database-context-class"></a>データベース コンテキスト クラスを作成する

データベース コンテキスト クラスは、`Movie` モデルの EF Core 機能 (作成、読み取り、更新、削除) を調整するために必要です。 データベース コンテキストは [Microsoft.EntityFrameworkCore.DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext) から派生し、データ モデルに含めるエンティティを指定します。

*Data* フォルダーを作成します。

次のコードで *Data/MvcMovieContext.cs* ファイルを追加します。 

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/zDocOnly/MvcMovieContext.cs?name=snippet)]

上記のコードによって、エンティティ セットの [DbSet\<Movie>](/dotnet/api/microsoft.entityframeworkcore.dbset-1) プロパティが作成されます。 Entity Framework の用語では、エンティティ セットは通常はデータベース テーブルに対応します。 エンティティはテーブル内の行に対応します。

<a name="reg"></a>

## <a name="register-the-database-context"></a>データベース コンテキストの登録

ASP.NET Core には、[依存関係挿入 (DI)](xref:fundamentals/dependency-injection) が組み込まれています。 サービス (EF Core DB コンテキストなど) は、アプリケーションの起動時に DI に登録する必要があります。 これらのサービスを必要とするコンポーネント (Razor Pages など) には、コンストラクターのパラメーターを介してこれらのサービスが指定されます。 DB コンテキスト インスタンスを取得するコンストラクター コードは、チュートリアルの後半で示します。 このセクションでは、DI コンテナーにデータベース コンテキストを登録します。

*Startup.cs* の先頭に次の `using` ステートメントを追加します。

```csharp
using MvcMovie.Data;
using Microsoft.EntityFrameworkCore;
```

`Startup.ConfigureServices` で次の強調表示されたコードを追加します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Startup.cs?name=snippet_ConfigureServices&highlight=6-7)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Startup.cs?name=snippet_UseSqlite&highlight=6-7)]

---

[DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions) オブジェクトでメソッドが呼び出され、接続文字列の名前がコンテキストに渡されます。 ローカル開発の場合、[ASP.NET Core 構成システム](xref:fundamentals/configuration/index)によって *appsettings.json* ファイルから接続文字列が読み取られます。

<a name="cs"></a>

## <a name="add-a-database-connection-string"></a>データベース接続文字列の追加

接続文字列を *appsettings.json* ファイルに追加します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-json[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/appsettings.json?highlight=10-13)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-json[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/appsettings_SQLite.json?highlight=10-13)]

---

コンパイラ エラーのチェックとしてプロジェクトをビルドします。

## <a name="scaffold-movie-pages"></a>ムービー ページのスキャフォールディング

スキャフォールディング ツールを使用し、ムービー モデルの作成、読み取り、更新、削除の (CRUD) ページを生成します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

*ソリューション エクスプローラー* で、**Controllers** フォルダーを右クリックし、 **[追加]、[スキャフォールディングされた新しい項目]** の順に選択します。

![前述の手順を参照](adding-model/_static/add_controller21.png)

**[スキャフォールディングを追加]** ダイアログで、 **[Entity Framework を使用したビューがある MVC コントローラー]、[追加]** の順に選択します。

![[スキャフォールディングを追加] ダイアログ](adding-model/_static/add_scaffold21.png)

**[コントローラーの追加]** ダイアログ ボックスを完了します。

* **モデル クラス:** *Movie (MvcMovie.Models)*
* **データ コンテキスト クラス:** *MvcMovieContext (MvcMovie.Data)*

![[データの追加] コンテキスト](adding-model/_static/dc3.png)

* **ビュー:** 各オプションの既定値をオンにします。
* **コントローラー名:** 既定の *MoviesController* のままにします。
* **[追加]** を選択します。

Visual Studio では、次が作成されます。

* ムービー コントローラー (*Controllers/MoviesController.cs*)
* 作成、削除、詳細、編集、およびインデックス ページ用の Razor ビュー ファイル (*Views/Movies/\*.cshtml*)

このようなファイルの自動作成は、"*スキャフォールディング*" と呼ばれます。

### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code) 

* プロジェクト ディレクトリ (*Program.cs*、*Startup.cs*、および *.csproj* ファイルを含むディレクトリ) でコマンド ウィンドウを開きます。

* Linux で、スキャフォールディング ツールのパスをエクスポートします。

  ```console
  export PATH=$HOME/.dotnet/tools:$PATH
  ```

* 次のコマンドを実行します。

  ```dotnetcli
  dotnet aspnet-codegenerator controller -name MoviesController -m Movie -dc MvcMovieContext --relativeFolderPath Controllers --useDefaultLayout --referenceScriptLibraries
  ```

  [!INCLUDE [explains scaffold generated params](~/includes/mvc-intro/model4.md)]

### <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* プロジェクト ディレクトリ (*Program.cs*、*Startup.cs*、および *.csproj* ファイルを含むディレクトリ) でコマンド ウィンドウを開きます。

* 次のコマンドを実行します。

  ```dotnetcli
  dotnet aspnet-codegenerator controller -name MoviesController -m Movie -dc MvcMovieContext --relativeFolderPath Controllers --useDefaultLayout --referenceScriptLibraries
  ```

  [!INCLUDE [explains scaffold generated params](~/includes/mvc-intro/model4.md)]

---

<!-- End of tabs                  -->

データベースが存在しないため、スキャフォールディング ページをまだ使用できません。 アプリを実行し、 **[Movie App]** リンクをクリックすると、 *[データベースを開けません]* または *[そのようなテーブルはありません:Movie*] というエラー メッセージが表示されます。

<a name="migration"></a>

## <a name="initial-migration"></a>最初の移行

EF Core [移行](xref:data/ef-mvc/migrations)機能を使用し、データベースを作成します。 移行は、データ モデルに合わせてデータベースを作成したり、更新したりできる一連のツールです。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

**[ツール]** メニューで、 **[NuGet パッケージ マネージャー]** > **[パッケージ マネージャー コンソール]** (PMC) の順に選択します。

PMC で、次のコマンドを入力します。

```powershell
Add-Migration InitialCreate
Update-Database
```

* `Add-Migration InitialCreate`:*Migrations/{timestamp}_InitialCreate.cs* 移行ファイルが生成されます。 `InitialCreate` 引数は、移行の名前です。 任意の名前を使用できますが、慣例により、移行を説明する名前が選択されます。 これは最初の移行であるため、生成されたクラスには、データベース スキーマを作成するコードが含まれています。 データベース スキーマは、`MvcMovieContext` クラスで指定されたモデルに基づきます。

* `Update-Database`:前のコマンドで作成された最新の移行にデータベースを更新します。 このコマンドにより *Migrations/{time-stamp}_InitialCreate.cs* ファイルで `Up` メソッドが実行され、データベースが作成されます。

  データベース更新コマンドにより、次の警告が生成されます。 

  > エンティティ型 'Movie' の decimal 列 'Price' に型が指定されていません。 これにより、値が既定の有効桁数と小数点以下桁数に収まらない場合、自動的に切り捨てられます。 'HasColumnType()' を使用してすべての値に適合する SQL server 列の型を明示的に指定します。

  この警告は無視して構いません。後のチュートリアルで修正されます。

[!INCLUDE [more information on the PMC tools for EF Core](~/includes/ef-pmc.md)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!INCLUDE [more information on the CLI for EF Core](~/includes/ef-cli.md)]

次の .NET Core CLI コマンドを実行します。

```dotnetcli
dotnet ef migrations add InitialCreate
dotnet ef database update
```

* `ef migrations add InitialCreate`:*Migrations/{timestamp}_InitialCreate.cs* 移行ファイルが生成されます。 `InitialCreate` 引数は、移行の名前です。 任意の名前を使用できますが、慣例により、移行を説明する名前が選択されます。 これは最初の移行であるため、生成されたクラスには、データベース スキーマを作成するコードが含まれています。 データベース スキーマは、`MvcMovieContext` クラスで指定されたモデルに基づきます (*Data/MvcMovieContext.cs* ファイル内)。

* `ef database update`:前のコマンドで作成された最新の移行にデータベースを更新します。 このコマンドにより *Migrations/{time-stamp}_InitialCreate.cs* ファイルで `Up` メソッドが実行され、データベースが作成されます。

---

### <a name="the-initialcreate-class"></a>InitialCreate クラス

*Migrations/{timestamp}_InitialCreate.cs* 移行ファイルを調べます。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Migrations/20190805165915_InitialCreate.cs?name=snippet)]

`Up` メソッドにより Movie テーブルが作成され、主キーとして `Id` が構成されます。 `Down` メソッドにより、`Up` 移行で行われたスキーマ変更が元に戻ります。

<a name="test"></a>

## <a name="test-the-app"></a>アプリのテスト

* アプリを実行し、 **[Movie App]** リンクをクリックします。

  次のような例外が表示された場合:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

  ```console
  SqlException: Cannot open database "MvcMovieContext-1" requested by the login. The login failed.
  ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

  ```console
  SqliteException: SQLite Error 1: 'no such table: Movie'.
  ```

---
  [移行手順](#migration)を実行しなかった可能性があります。

* **Create** ページをテストします。 データを入力して送信します。

  > [!NOTE]
  > `Price` フィールドに小数点のコンマを入力できない場合があります。 小数点にコンマ (",") を使う英語以外のロケール、および英語 (米国) 以外の日付形式で、[jQuery 検証](https://jqueryvalidation.org/)をサポートするには、アプリをグローバル化する必要があります。 グローバル化の手順については、[この GitHub の記事](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420)をご覧ください。

* **[編集]** 、 **[詳細]** 、 **[削除]** の各ページをテストします。

## <a name="dependency-injection-in-the-controller"></a>コントローラーの依存関係挿入

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

*Controllers/MoviesController.cs* ファイルを開いて、コンストラクターを調べます。

<!-- l.. Make copy of Movies controller (or use the old one as I did in the 3.0 upgrade) because we comment out the initial index method and update it later  -->

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_1)]

コンストラクターでは、[依存性の注入](xref:fundamentals/dependency-injection)を使ってデータベース コンテキスト (`MvcMovieContext`) がコントローラーに挿入されています。 データベース コンテキストは、コントローラーの各 [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) メソッドで使用されます。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_1)]

コンストラクターでは、[依存性の注入](xref:fundamentals/dependency-injection)を使ってデータベース コンテキスト (`MvcMovieContext`) がコントローラーに挿入されています。 データベース コンテキストは、コントローラーの各 [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) メソッドで使用されます。

### <a name="use-sqlite-for-development-sql-server-for-production"></a>開発用に SQLite を、運用環境に SQL Server を使用する

SQLite が選択されている場合、テンプレートで生成されたコードは開発の準備ができています。 次のコードでは、スタートアップに <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> を挿入する方法を示します。 `IWebHostEnvironment` が挿入されているので、`ConfigureServices` では開発用に SQLite を、運用環境に SQL Server を使用できます。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/StartupDevProd.cs?name=snippet_StartupClass&highlight=5,10,16-28)]

---
<!-- end of tabs --->

<a name="strongly-typed-models-keyword-label"></a>
<a name="strongly-typed-models-and-the--keyword"></a>

## <a name="strongly-typed-models-and-the-model-keyword"></a>厳密に型指定されたモデルと @model キーワード

コントローラーで `ViewData` ディクショナリを使ってビューにデータまたはオブジェクトを渡す方法を前に示しました。 `ViewData` ディクショナリは動的オブジェクトであり、ビューに情報を渡すための便利な遅延バインディングの方法を提供します。

MVC にも、厳密に型指定されたモデル オブジェクトをビューに渡す機能があります。 この厳密に型指定された方法では、コンパイル時にコードを確認できます。 スキャフォールディング メカニズムでは、`MoviesController` のクラスとビューで、この手法 (つまり、厳密に型指定されたモデルを渡しました) が使用されました。

*Controllers/MoviesController.cs* ファイルで生成された `Details` メソッドを調べてください。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_details)]

通常、`id` パラメーターはルート データとして渡されます。 たとえば、`https://localhost:5001/movies/details/1` は次のように設定します。

* コントローラーを `movies` コントローラーに (最初の URL セグメント)。
* アクションを `details` に (2 番目の URL セグメント)。
* ID を 1 に (最後の URL セグメント)。

次のようにクエリ文字列で `id` を渡すこともできます。

`https://localhost:5001/movies/details?id=1`

ID 値が指定されない場合、`id` パラメーターは [null 許容型](/dotnet/csharp/programming-guide/nullable-types/index) (`int?`) として定義されます。

ルート データまたはクエリ文字列の値と一致するムービー エンティティを選択するため、[ラムダ式](/dotnet/articles/csharp/programming-guide/statements-expressions-operators/lambda-expressions)が `FirstOrDefaultAsync` に渡されます。

```csharp
var movie = await _context.Movie
    .FirstOrDefaultAsync(m => m.Id == id);
```

ムービーが見つかった場合、`Movie` モデルのインスタンスが `Details` ビューに渡されます。

```csharp
return View(movie);
```

*Views/Movies/Details.cshtml* ファイルの内容を確認してください。

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/DetailsOriginal.cshtml)]

ビュー ファイルの一番上にある `@model` ステートメントにより、ビューで求められるオブジェクトの型が指定されます。 ムービー コントローラーが作成されたとき、次の `@model` ステートメントが含まれました。

```cshtml
@model MvcMovie.Models.Movie
```

この `@model` ディレクティブにより、コントローラーでビューに渡されたムービーにアクセスできます。 `Model` オブジェクトは厳密に型指定されます。 たとえば、*Details.cshtml* ビューでは、コードで厳密に型指定された `Model` オブジェクトを使って、`DisplayNameFor` および `DisplayFor` HTML ヘルパーに各ムービー フィールドを渡しています。 `Create` および `Edit` のメソッドとビューも、`Movie` モデル オブジェクトを渡します。

Movies コントローラーの *Index.cshtml* ビューと `Index` メソッドを確認してください。 コードで `View` メソッドを呼び出すときの `List` オブジェクトの作成方法に注意してください。 コードでは、この `Movies` リストを `Index` アクション メソッドからビューに渡しています。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_index)]

ムービー コントローラーが作成されたとき、スキャフォールディングにより、*Index.cshtml* ファイルの一番上に次の `@model` ステートメントが含まれました。

<!-- Copy Index.cshtml to IndexOriginal.cshtml -->

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/IndexOriginal.cshtml?range=1)]

`@model` ディレクティブにより、厳密に型指定された `Model` オブジェクトを使って、コントローラーがビューに渡したムービーのリストにアクセスできます。 たとえば、*Index.cshtml* ビューのコードでは、`foreach` ステートメントを使って厳密に型指定された `Model` オブジェクトのムービーをループ処理しています。

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/IndexOriginal.cshtml?highlight=1,31,34,37,40,43,46-48)]

`Model` オブジェクトは厳密に型指定されているので (`IEnumerable<Movie>` オブジェクトとして)、ループ内の各項目は `Movie` として型指定されます。 これには、コンパイル時にコードを確認できるなどの利点があります。

## <a name="additional-resources"></a>その他の技術情報

* [タグ ヘルパー](xref:mvc/views/tag-helpers/intro)
* [グローバライズとローカライズ](xref:fundamentals/localization)

> [!div class="step-by-step"]
> [前のチュートリアル ビューの追加](adding-view.md)
> [次のチュートリアル SQL の使用](working-with-sql.md)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

## <a name="add-a-data-model-class"></a>データ モデル クラスの追加

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

*Models* フォルダーを右クリックし、 **[追加]**  >  **[クラス]** の順に選択します。 クラスに **Movie** と名前を付けます。

[!INCLUDE [model 1b](~/includes/mvc-intro/model1b.md)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

* *Movie.cs* という名前の *Models* フォルダーにクラスを追加します。

[!INCLUDE [model 1b](~/includes/mvc-intro/model1b.md)]
[!INCLUDE [model 2](~/includes/mvc-intro/model2.md)]

---

## <a name="scaffold-the-movie-model"></a>ムービー モデルのスキャフォールディング

このセクションでは、ムービー モデルがスキャフォールディングされます。 つまり、スキャフォールディング ツールにより、ムービー モデルの作成、読み取り、更新、削除の (CRUD) 操作用のページが生成されます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

*ソリューション エクスプローラー* で、**Controllers** フォルダーを右クリックし、 **[追加]、[スキャフォールディングされた新しい項目]** の順に選択します。

![前述の手順を参照](adding-model/_static/add_controller21.png)

**[スキャフォールディングを追加]** ダイアログで、 **[Entity Framework を使用したビューがある MVC コントローラー]、[追加]** の順に選択します。

![[スキャフォールディングを追加] ダイアログ](adding-model/_static/add_scaffold21.png)

**[コントローラーの追加]** ダイアログ ボックスを完了します。

* **モデル クラス:** *Movie (MvcMovie.Models)*
* **データ コンテキスト クラス:** **+** アイコンを選択して、既定の **MvcMovie.Models.MvcMovieContext** を追加します。

![[データの追加] コンテキスト](adding-model/_static/dc.png)

* **ビュー:** 各オプションの既定値をオンにします。
* **コントローラー名:** 既定の *MoviesController* のままにします。
* **[追加]** を選択します。

![[コントローラーの追加] ダイアログ](adding-model/_static/add_controller2.png)

Visual Studio では、次が作成されます。

* Entity Framework Core の [データベース コンテキスト クラス](xref:data/ef-mvc/intro#create-the-database-context)(*Data/MvcMovieContext.cs*)
* ムービー コントローラー (*Controllers/MoviesController.cs*)
* 作成、削除、詳細、編集、およびインデックス ページ用の Razor ビュー ファイル (*Views/Movies/\*.cshtml*)

データベース コンテキストと [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) (作成、読み取り、更新、および削除) アクション メソッドとビューの自動作成は、*スキャフォールディング* と言います。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

<!--  Until https://github.com/aspnet/Scaffolding/issues/582 is fixed windows needs backslash or the namespace is namespace RazorPagesMovie.Pages_Movies rather than namespace RazorPagesMovie.Pages.Movies
-->

* プロジェクト ディレクトリ (*Program.cs*、*Startup.cs*、および *.csproj* ファイルを含むディレクトリ) でコマンド ウィンドウを開きます。
* スキャフォールディング ツールをインストールします。

  ```dotnetcli
   dotnet tool install --global dotnet-aspnet-codegenerator
   ```

* Linux で、スキャフォールディング ツールのパスをエクスポートします。

  ```console
    export PATH=$HOME/.dotnet/tools:$PATH
  ```

* 次のコマンドを実行します。

  ```dotnetcli
   dotnet aspnet-codegenerator controller -name MoviesController -m Movie -dc MvcMovieContext --relativeFolderPath Controllers --useDefaultLayout --referenceScriptLibraries
  ```

[!INCLUDE [explains scaffold generated params](~/includes/mvc-intro/model4.md)]

<!-- Mac -------------------------->

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* プロジェクト ディレクトリ (*Program.cs*、*Startup.cs*、および *.csproj* ファイルを含むディレクトリ) でコマンド ウィンドウを開きます。
* スキャフォールディング ツールをインストールします。

  ```dotnetcli
   dotnet tool install --global dotnet-aspnet-codegenerator
   ```

* 次のコマンドを実行します。

  ```dotnetcli
   dotnet aspnet-codegenerator controller -name MoviesController -m Movie -dc MvcMovieContext --relativeFolderPath Controllers --useDefaultLayout --referenceScriptLibraries
  ```

[!INCLUDE [explains scaffold generated params](~/includes/mvc-intro/model4.md)]

---

<!-- End of VS tabs                  -->

アプリを実行し、 **[MVC Movie]\(MVC ムービー\)** リンクをクリックすると、次のようなエラーが表示されます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

```
An unhandled exception occurred while processing the request.

SqlException: Cannot open database "MvcMovieContext-<GUID removed>" requested by the login. The login failed.
Login failed for user 'Rick'.

System.Data.SqlClient.SqlInternalConnectionTds..ctor(DbConnectionPoolIdentity identity, SqlConnectionString
```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

```
An unhandled exception occurred while processing the request.
SqliteException: SQLite Error 1: 'no such table: Movie'.
Microsoft.Data.Sqlite.SqliteException.ThrowExceptionForRC(int rc, sqlite3 db)

```

---

データベースを作成する必要があり、それには EF Core [移行](xref:data/ef-mvc/migrations)機能を使用します。 移行では、データ モデルに一致するデータベースを作成し、データ モデルの変更時にデータベース スキーマを更新することができます。

<a name="pmc"></a>

## <a name="initial-migration"></a>最初の移行

このセクションでは、次のタスクを完了しました。

* 初期移行を追加します。
* 初期移行でデータベースを更新します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. **[ツール]** メニューで、 **[NuGet パッケージ マネージャー]** > **[パッケージ マネージャー コンソール]** (PMC) の順に選択します。

   ![PMC メニュー](~/tutorials/first-mvc-app/adding-model/_static/pmc.png)

1. PMC で、次のコマンドを入力します。

   ```powershell
   Add-Migration Initial
   Update-Database
   ```

   `Add-Migration` コマンドによって最初のデータベース スキーマを作成するコードが生成されます。

   データベース スキーマは、`MvcMovieContext` クラスで指定されたモデルに基づきます。 `Initial` 引数は、移行の名前です。 任意の名前を使用できますが、慣例により、移行を説明する名前が使用されます。 詳細については、「<xref:data/ef-mvc/migrations>」を参照してください。

   `Update-Database` コマンドは、データベースを作成する、*Migrations/{time-stamp}_InitialCreate.cs* ファイルの `Up` メソッドを実行します。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!INCLUDE [initial migration](~/includes/RP/model3.md)]

`ef migrations add InitialCreate` コマンドによって最初のデータベース スキーマを作成するコードが生成されます。

データベース スキーマは、`MvcMovieContext` クラスで指定されたモデルに基づきます (*Data/MvcMovieContext.cs* ファイル内)。 `InitialCreate` 引数は、移行の名前です。 任意の名前を使用できますが、慣例により、移行を説明する名前が選択されます。

---

## <a name="examine-the-context-registered-with-dependency-injection"></a>依存関係挿入に登録されるコンテキストを調べる

ASP.NET Core には、[依存関係挿入 (DI)](xref:fundamentals/dependency-injection) が組み込まれています。 サービス (EF Core DB コンテキストなど) は、アプリケーションの起動時に DI に登録されます。 これらのサービスを必要とするコンポーネント (Razor Pages など) には、コンストラクターのパラメーターを介してこれらのサービスが指定されます。 DB コンテキスト インスタンスを取得するコンストラクター コードは、チュートリアルの後半で示します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

スキャフォールディング ツールが自動的に DB コンテキストを作成し、それを DI コンテナーに登録します。

次の `Startup.ConfigureServices` メソッドを調べます。 強調表示された行は、スキャフォールダーによって追加されました。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Startup.cs?name=snippet_ConfigureServices&highlight=14-15)]

`MvcMovieContext` では、`Movie` モデルのために EF Core 機能 (作成、読み取り、更新、削除など) が調整されます。 データ コンテキスト (`MvcMovieContext`) は [Microsoft.EntityFrameworkCore.DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext) から派生されます。 データ コンテキストによって、データ モデルに含めるエンティティが指定されます:

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Data/MvcMovieContext.cs)]

上記のコードによって、エンティティ セットの [DbSet\<Movie>](/dotnet/api/microsoft.entityframeworkcore.dbset-1) プロパティが作成されます。 Entity Framework の用語では、エンティティ セットは通常はデータベース テーブルに対応します。 エンティティはテーブル内の行に対応します。

[DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions) オブジェクトでメソッドが呼び出され、接続文字列の名前がコンテキストに渡されます。 ローカル開発の場合、[ASP.NET Core 構成システム](xref:fundamentals/configuration/index)によって *appsettings.json* ファイルから接続文字列が読み取られます。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

DB コンテキストを作成し、それを DI コンテナーに登録しました。

---

<a name="test"></a>

### <a name="test-the-app"></a>アプリのテスト

* アプリを実行し、ブラウザーで URL に `/Movies` を追加します ( `http://localhost:port/movies` )。

次のようなデータベースの例外が表示された場合:

```console
SqlException: Cannot open database "MvcMovieContext-GUID" requested by the login. The login failed.
Login failed for user 'User-name'.
```

[移行手順](#pmc)を失敗しました。

* **[作成]** リンクをテストします。 データを入力して送信します。

  > [!NOTE]
  > `Price` フィールドに小数点のコンマを入力できない場合があります。 小数点にコンマ (",") を使う英語以外のロケール、および英語 (米国) 以外の日付形式で、[jQuery 検証](https://jqueryvalidation.org/)をサポートするには、アプリをグローバル化する必要があります。 グローバル化の手順については、[この GitHub の記事](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420)をご覧ください。

* **[編集]** 、 **[詳細]** 、および **[削除]** の各リンクをテストします。

次の `Startup` クラスを調べます。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Startup.cs?name=snippet_ConfigureServices&highlight=13-99)]

上のコードで強調表示されている部分は、[依存性の注入](xref:fundamentals/dependency-injection)コンテナーに追加されているムービー データベース コンテキストを示します。

* `services.AddDbContext<MvcMovieContext>(options =>` では、使用するデータベースと接続文字列を指定します。
* `=>` は[ラムダ演算子](/dotnet/articles/csharp/language-reference/operators/lambda-operator)です。

*Controllers/MoviesController.cs* ファイルを開いて、コンストラクターを調べます。

<!-- l.. Make copy of Movies controller because we comment out the initial index method and update it later  -->

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_1)]

コンストラクターでは、[依存性の注入](xref:fundamentals/dependency-injection)を使ってデータベース コンテキスト (`MvcMovieContext`) がコントローラーに挿入されています。 データベース コンテキストは、コントローラーの各 [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) メソッドで使用されます。

<a name="strongly-typed-models-keyword-label"></a>
<a name="strongly-typed-models-and-the--keyword"></a>

## <a name="strongly-typed-models-and-the-model-keyword"></a>厳密に型指定されたモデルと @model キーワード

コントローラーで `ViewData` ディクショナリを使ってビューにデータまたはオブジェクトを渡す方法を前に示しました。 `ViewData` ディクショナリは動的オブジェクトであり、ビューに情報を渡すための便利な遅延バインディングの方法を提供します。

MVC にも、厳密に型指定されたモデル オブジェクトをビューに渡す機能があります。 この厳密に型指定された方法を使うと、コンパイル時のコードのチェックが向上します。 スキャフォールディング メカニズムは、メソッドとビューを作成するときに、`MoviesController` クラスとビューでこの方法 (つまり、厳密に型指定されたモデルを渡すこと) を使いました。

*Controllers/MoviesController.cs* ファイルで生成された `Details` メソッドを調べてください。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_details)]

通常、`id` パラメーターはルート データとして渡されます。 たとえば、`https://localhost:5001/movies/details/1` は次のように設定します。

* コントローラーを `movies` コントローラーに (最初の URL セグメント)。
* アクションを `details` に (2 番目の URL セグメント)。
* ID を 1 に (最後の URL セグメント)。

次のようにクエリ文字列で `id` を渡すこともできます。

`https://localhost:5001/movies/details?id=1`

ID 値が指定されない場合、`id` パラメーターは [null 許容型](/dotnet/csharp/programming-guide/nullable-types/index) (`int?`) として定義されます。

ルート データまたはクエリ文字列の値と一致するムービー エンティティを選択するため、[ラムダ式](/dotnet/articles/csharp/programming-guide/statements-expressions-operators/lambda-expressions)が `FirstOrDefaultAsync` に渡されます。

```csharp
var movie = await _context.Movie
    .FirstOrDefaultAsync(m => m.Id == id);
```

ムービーが見つかった場合、`Movie` モデルのインスタンスが `Details` ビューに渡されます。

```csharp
return View(movie);
   ```

*Views/Movies/Details.cshtml* ファイルの内容を確認してください。

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/DetailsOriginal.cshtml)]

ビュー ファイルの先頭に `@model` ステートメントを含めることで、ビューが期待するオブジェクトの型を指定することができます。 ムービー コントローラーを作成したとき、*Details.cshtml* ファイルの先頭に次の `@model` ステートメントが自動的に追加されています。

```cshtml
@model MvcMovie.Models.Movie
```

この `@model` ディレクティブにより、厳密に型指定された `Model` オブジェクトを使って、コントローラーがビューに渡したムービーにアクセスできます。 たとえば、*Details.cshtml* ビューでは、コードで厳密に型指定された `Model` オブジェクトを使って、`DisplayNameFor` および `DisplayFor` HTML ヘルパーに各ムービー フィールドを渡しています。 `Create` および `Edit` のメソッドとビューも、`Movie` モデル オブジェクトを渡します。

Movies コントローラーの *Index.cshtml* ビューと `Index` メソッドを確認してください。 コードで `View` メソッドを呼び出すときの `List` オブジェクトの作成方法に注意してください。 コードでは、この `Movies` リストを `Index` アクション メソッドからビューに渡しています。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_index)]

ムービー コントローラーを作成したとき、スキャフォールディングによって *Index.cshtml* ファイルの先頭に `@model` ステートメントが自動的に追加されています。

<!-- Copy Index.cshtml to IndexOriginal.cshtml -->

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/IndexOriginal.cshtml?range=1)]

`@model` ディレクティブにより、厳密に型指定された `Model` オブジェクトを使って、コントローラーがビューに渡したムービーのリストにアクセスできます。 たとえば、*Index.cshtml* ビューのコードでは、`foreach` ステートメントを使って厳密に型指定された `Model` オブジェクトのムービーをループ処理しています。

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/IndexOriginal.cshtml?highlight=1,31,34,37,40,43,46-48)]

`Model` オブジェクトは厳密に型指定されているので (`IEnumerable<Movie>` オブジェクトとして)、ループ内の各項目は `Movie` として型指定されます。 それ以外の利点としては、コンパイル時にコードのチェックが行われます。

## <a name="additional-resources"></a>その他の技術情報

* [タグ ヘルパー](xref:mvc/views/tag-helpers/intro)
* [グローバライズとローカライズ](xref:fundamentals/localization)

> [!div class="step-by-step"]
> [前のチュートリアル ビューの追加](adding-view.md)
> [次のチュートリアル データベースの使用](working-with-sql.md)

::: moniker-end
