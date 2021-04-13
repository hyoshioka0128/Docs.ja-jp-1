---
title: 'パート 2: モデルを追加する'
author: rick-anderson
description: Razor ページのチュートリアル シリーズのパート 2。 このセクションでは、モデル クラスを追加します。
ms.author: riande
ms.date: 03/10/2021
ms.custom: contperf-fy21q2
no-loc:
- Index
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
uid: tutorials/razor-pages/model
ms.openlocfilehash: 28d1a99e57a07a8d63cbbb7f0ecc7f84d1adf446
ms.sourcegitcommit: 0abfe496fed8e9470037c8128efa8a50069ccd52
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/07/2021
ms.locfileid: "106564300"
---
# <a name="part-2-add-a-model-to-a-razor-pages-app-in-aspnet-core"></a>パート 2: ASP.NET Core で Razor ページ アプリにモデルを追加する

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-5.0"

<!-- In the next update on the CLI version, let the scaffolder do the same work the VS driven scaffolder does. That is, create the DB context, etc -->

このセクションでは、データベースで映画を管理するクラスが追加されます。 アプリのモデル クラスでは、[Entity Framework Core (EF Core)](/ef/core) を使用して、データベースを操作します。 EF Core は、データ アクセスを簡略化するオブジェクト リレーショナル マッパー (O/RM) です。 モデル クラスを記述すると、EF Core によってデータベースが作成されます。

モデル クラスは EF Core に対する依存関係がないため、**P** lain-**O** ld **C** LR **O** bjects (単純な従来の CLR) の頭文字を取って POCO クラスと呼ばれます。 これらは、データベースに格納されるデータのプロパティを定義します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="add-a-data-model"></a>データ モデルの追加

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. **ソリューション エクスプローラー** で、 *RazorPagesMovie* プロジェクトを右クリックし、 **[追加]**  >  **[新しいフォルダー]** を選択します。 フォルダーに「*Models*」という名前を付けます。
1. *Models* フォルダーを右クリックします。 **[追加]**  >  **[クラス]** の順に選択します。 クラスに *Movie* と名前を付けます。
1. `Movie` クラスに次のプロパティを追加します。

   [!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

`Movie` クラスには次が含まれます。

* `ID` フィールドは、データベースで主キー用に必要です。
* `[DataType(DataType.Date)]`: [[DataType]](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) 属性では、データの種類 (`Date`) を指定します。 この属性を使用する場合:

  * ユーザーは日付フィールドに時刻の情報を入力する必要はありません。
  * 日付のみが表示され、時刻の情報は表示されません。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

1. *Models* という名前のフォルダーを追加します。
1. *Movie.cs* という名前の *Models* フォルダーにクラスを追加します。

`Movie` クラスに次のプロパティを追加します。

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

`Movie` クラスには次が含まれます。

* `ID` フィールドは、データベースで主キー用に必要です。
* `[DataType(DataType.Date)]`: [[DataType]](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) 属性では、データの種類 (`Date`) を指定します。 この属性を使用する場合:

  * ユーザーは日付フィールドに時刻の情報を入力する必要はありません。
  * 日付のみが表示され、時刻の情報は表示されません。

<a name="dc"></a>

### <a name="add-nuget-packages-and-ef-tools"></a>NuGet パッケージと EF ツールを追加します。

[!INCLUDE[](~/includes/add-EF-NuGet-SQLite-CLI-5.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

1. **ソリューション ツール ウィンドウ** で、Ctrl キーを押したまま *RazorPagesMovie* プロジェクトをクリックしてから、 **[追加]** > **[新しいフォルダー...]** を選択します。フォルダーに「*Models*」という名前を付けます。
1. Ctrl キーを押したまま *Models* フォルダーをクリックし、 **[追加]** > **[新しいファイル...]** を選択します。
1. **[新しいファイル]** ダイアログで次を実行します。
   1. 左側のウィンドウで **[全般]** を選択します。
   1. 中央ウィンドウで **[空のクラス]** を選択します。
   1. クラスに **Movie** という名前を付け、 **[新規]** を選択します。

1. `Movie` クラスに次のプロパティを追加します。

   [!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

`Movie` クラスには次が含まれます。

* `ID` フィールドは、データベースで主キー用に必要です。
* `[DataType(DataType.Date)]`: [[DataType]](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) 属性では、データの種類 (`Date`) を指定します。 この属性を使用する場合:

  * ユーザーは日付フィールドに時刻の情報を入力する必要はありません。
  * 日付のみが表示され、時刻の情報は表示されません。

---

[DataAnnotations](xref:System.ComponentModel.DataAnnotations) は、後のチュートリアルで説明されます。

プロジェクトをビルドして、コンパイル エラーがないことを確認します。

## <a name="scaffold-the-movie-model"></a>ムービー モデルのスキャフォールディング

このセクションでは、ムービー モデルがスキャフォールディングされます。 つまり、スキャフォールディング ツールにより、ムービー モデルの作成、読み取り、更新、削除の (CRUD) 操作用のページが生成されます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. *Pages/Movies* フォルダーを作成します。
   1. *Pages* フォルダーを右クリックし、 **[追加]** > **[新しいフォルダー]** を選択します。
   1. フォルダーに *Movies* という名前を付けます。

1. *Pages/Movies* フォルダーを右クリックし、 **[追加]** > **[スキャフォールディングされた新しい項目]** を選択します。

   ![前の手順からのイメージ。](model/_static/5/sca.png)

1. **[スキャフォールディングを追加]** ダイアログで、 **[Entity Framework を使用する Razor ページ (CRUD)]** > **[追加]** の順に選択します。

   ![前の手順からのイメージ。](model/_static/add_scaffold.png)

1. **[Add Razor Pages using Entity Framework (CRUD)]\(Entity Framework を使用して Razor Pages (CRUD) を追加する\)** ダイアログを完了します。
   1. **[モデル クラス]** ドロップ ダウンで、 **[Movie (RazorPagesMovie.Models)]** を選択します。
   1. **Data context class** 行で、 **+** (+) 記号を選択します。
      1. **[データ コンテキストの追加]** ダイアログで、クラス名 `RazorPagesMovie.Data.RazorPagesMovieContext` が生成されます。
   1. **[追加]** を選びます。

   ![前の手順からのイメージ。](model/_static/3/arp.png)

*appsettings.json* ファイルは、ローカル データベースへの接続に使用される接続文字列を使用して更新されます。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

<!--  Until https://github.com/aspnet/Scaffolding/issues/582 is fixed windows needs backslash or the namespace is namespace RazorPagesMovie.Pages_Movies rather than namespace RazorPagesMovie.Pages.Movies
-->

* *Program.cs*、*Startup.cs*、および *.csproj* ファイルが含まれているプロジェクト ディレクトリでコマンド シェルを開きます。

* **Windows の場合**:次のコマンドを実行します。

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages\Movies --referenceScriptLibraries
  ```

* **macOS および Linux の場合**:次のコマンドを実行します。

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages/Movies --referenceScriptLibraries
  ```

<a name="codegenerator"></a> 次の表で、ASP.NET Core コード ジェネレーターのオプションについて詳しく説明します。

| オプション               | 説明|
| ----------------- | ------------ |
| `-m`  | モデルの名前。 |
| `-dc`  | 使用する `DbContext` クラス。 |
| `-udl` | 既定のレイアウトを使用します。 |
| `-outDir` | ビューを作成するための相対出力フォルダー パス。 |
| `--referenceScriptLibraries` | [編集] および [作成] ページに `_ValidationScriptsPartial` を追加します。 |

`aspnet-codegenerator razorpage` コマンドに関するヘルプを取得するには、`-h` オプションを使用します。

```dotnetcli
dotnet aspnet-codegenerator razorpage -h
```

詳細については、「[dotnet aspnet-codegenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator)」を参照してください。

[!INCLUDE[](~/includes/RP/sqlitedev.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

1. *Pages/Movies* フォルダーを作成します。
   1. Ctrl キーを押したまま *Pages* フォルダーをクリックし、 **[追加]** > **[新しいフォルダー]** を選択します。
   1. フォルダーに *Movies* という名前を付けます。

1. Ctrl キーを押したまま *Pages/Movies* フォルダーをクリックし、 **[追加]** > **[新しいスキャフォールディング...]** を選択します。

   ![前の手順からのイメージ。](model/_static/scaMac.png)

1. **[新しいスキャフォールディング]** ダイアログで、 **[Entity Framework を使用する Razor Pages (CRUD)]** > **[次へ]** を選択します。

   ![前の手順からのイメージ。](model/_static/add_scaffoldMac.png)

1. **[Add Razor Pages using Entity Framework (CRUD)]\(Entity Framework を使用して Razor Pages (CRUD) を追加する\)** ダイアログを完了します。
   1. **[DbContext Class to use:]\(使用する DbContext クラス:\)** 行で、クラスに `RazorPagesMovie.Data.RazorPagesMovieContext` という名前を付けます。
   1. **[完了]** を選択します。

   ![前の手順からのイメージ。](model/_static/5/arpMac.png)

*appsettings.json* ファイルは、ローカル データベースへの接続に使用される接続文字列を使用して更新されます。

[!INCLUDE[](~/includes/RP/sqlitedev.md)]

---

### <a name="files-created"></a>作成されたファイル

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

スキャフォールディングのプロセスが作成され、次のファイルが更新されます。

* *Pages/Movies*: Create、Delete、Details、Edit、および Index。
* *Data/RazorPagesMovieContext.cs*

### <a name="updated"></a>更新済み

* *Startup.cs*

作成および更新されたファイルについては、次のセクションで説明します。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

スキャフォールディングのプロセスでは、次のファイルが作成されます。

* *Pages/Movies*: Create、Delete、Details、Edit、および Index。

作成されたファイルについては、次のセクションで説明します。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

スキャフォールディングのプロセスが作成され、次のファイルが更新されます。

* *Pages/Movies*: Create、Delete、Details、Edit、および Index。
* *Data/RazorPagesMovieContext.cs*

### <a name="updated"></a>更新済み

* *Startup.cs*

作成および更新されたファイルについては、次のセクションで説明します。

---

<a name="pmc"></a>

## <a name="create-the-initial-database-schema-using-efs-migration-feature"></a>EF の移行機能を使用して初期データベース スキーマを作成する

Entity Framework Core の移行機能を使用すると、次のことができます。

* 初期データベース スキーマを作成します。
* データベース スキーマを増分方式で更新して、アプリケーションのデータ モデルとの同期を維持する。  データベース内の既存のデータは保持されます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

このセクションでは、**パッケージ マネージャー コンソール** (PMC) ウィンドウを使用して、次の作業を行います。

* 初期移行を追加します。
* 初期移行でデータベースを更新します。

1. **[ツール]** メニューで、 **[NuGet パッケージ マネージャー]** > **[パッケージ マネージャー コンソール]** の順に選択します。

   ![PMC メニュー](model/_static/5/pmc.png)

1. PMC で、次のコマンドを入力します。

   ```powershell
   Add-Migration InitialCreate
   Update-Database
   ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!INCLUDE [more information on the CLI for EF Core](~/includes/ef-cli.md)]

* 次の .NET CLI コマンドを実行します。

  ```dotnetcli
  dotnet ef migrations add InitialCreate
  dotnet ef database update
  ```

---

上記のコマンドで次の警告が生成されます。"エンティティ型 'Movie' の decimal 列 'Price' に型が指定されていません。 これにより、値が既定の有効桁数と小数点以下桁数に収まらない場合、自動的に切り捨てられます。 'HasColumnType()' を使用してすべての値に適合する SQL server 列の型を明示的に指定します。"

この警告は、後の手順で解決されるため、無視してください。

`migrations` コマンドによって最初のデータベース スキーマを作成するコードが生成されます。 このスキーマは、`DbContext` で指定されたモデルに基づきます。 `InitialCreate` 引数は移行の命名に使用されます。 任意の名前を使用できますが、規則により、移行を説明する名前が選択されます。

`update` コマンドにより、適用されていない移行で `Up` メソッドが実行されます。 この場合、`update` により、*Migrations/\<time-stamp>_InitialCreate.cs* ファイルで `Up` メソッドが実行され、データベースが作成されます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

### <a name="examine-the-context-registered-with-dependency-injection"></a>依存関係挿入に登録されるコンテキストを調べる

ASP.NET Core には、[依存関係挿入](xref:fundamentals/dependency-injection)が組み込まれています。 アプリケーションの起動時に、EF Core データベース コンテキストなどのサービスが依存関係の挿入に登録されます。 これらのサービスを必要とするコンポーネント (Razor Pages など) には、コンストラクターのパラメーターを介してこれらのサービスが指定されます。 データベース コンテキスト インスタンスを取得するコンストラクター コードは、この後のチュートリアルで示します。

スキャフォールディング ツールによってデータベース コンテキストが自動的に作成され、依存関係挿入コンテナーに登録されました。

`Startup.ConfigureServices` メソッドを調べます。 強調表示された行は、スキャフォールダーによって追加されました。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

`RazorPagesMovieContext` によって、`Movie` モデル用の EF Core 機能 (作成、読み取り、更新、削除など) が調整されます。 データ コンテキスト (`RazorPagesMovieContext`) は [Microsoft.EntityFrameworkCore.DbContext](xref:Microsoft.EntityFrameworkCore.DbContext) から派生されます。 データ コンテキストによって、データ モデルに含めるエンティティが指定されます。

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Data/RazorPagesMovieContext.cs)]

上記のコードによって、エンティティ セットの [DbSet\<Movie>](xref:Microsoft.EntityFrameworkCore.DbSet%601) プロパティが作成されます。 Entity Framework の用語では、エンティティ セットは通常はデータベース テーブルに対応します。 エンティティはテーブル内の行に対応します。

[DbContextOptions](xref:Microsoft.EntityFrameworkCore.DbContextOptions) オブジェクトでメソッドが呼び出され、接続文字列の名前がコンテキストに渡されます。 ローカル開発の場合、[構成システム](xref:fundamentals/configuration/index)によって *appsettings.json* ファイルから接続文字列が読み取られます。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

`Up` メソッドを調べます。

---

<a name="test"></a>

## <a name="test-the-app"></a>アプリのテスト

1. アプリを実行し、ブラウザーで URL に `/Movies` を追加します ( `http://localhost:port/movies` )。

   次のエラー メッセージが表示される場合:

   ```console
   SqlException: Cannot open database "RazorPagesMovieContext-GUID" requested by the login. The login failed.
   Login failed for user 'User-name'.
   ```

   [移行手順](#pmc)を失敗しました。

1. **[作成]** リンクをテストします。

   ![[作成] ページ](model/_static/conan5.png)

   > [!NOTE]
   > `Price` フィールドに小数点のコンマを入力できない場合があります。 小数点にコンマ (",") を使う英語以外のロケール、および英語 (米国) 以外の日付形式で、[jQuery 検証](https://jqueryvalidation.org/)をサポートするには、アプリをグローバル化する必要があります。 グローバル化の手順については、[この GitHub の記事](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420)をご覧ください。

1. **[編集]** 、 **[詳細]** 、および **[削除]** の各リンクをテストします。

次のチュートリアルでは、スキャフォールディングによって作成されるファイルについて説明します。

## <a name="additional-resources"></a>その他の技術情報


> [!div class="step-by-step"]
> [前へ:はじめに](xref:tutorials/razor-pages/razor-pages-start)
> [次: スキャフォールディングされた Razor ページ](xref:tutorials/razor-pages/page)

::: moniker-end

<!--  ::: End of >= 5.0 moniker version   -->

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

<!-- In the next update on the CLI version, let the scaffolder do the same work the VS driven scaffolder does. That is, create the DB context, etc -->

このセクションでは、映画を管理するクラスが追加されます。 アプリのモデル クラスでは、[Entity Framework Core (EF Core)](/ef/core) を使用して、データベースを操作します。 EF Core は、データ アクセスを簡略化するオブジェクト リレーショナル マッパー (O/RM) です。

このモデル クラスは、EF Core に対する依存関係がないために、POCO クラス (plain-old CLR オブジェクト、つまり単純な従来の CLR) と呼ばれます。 これらは、データベースに格納されるデータのプロパティを定義します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="add-a-data-model"></a>データ モデルの追加

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

**RazorPagesMovie** プロジェクトを右クリックし、 **[追加]** 、 **[新しいフォルダー]** の順に選択します。 フォルダーに「*Models*」という名前を付けます。

*Models* フォルダーを右クリックします。 **[追加]**  >  **[クラス]** の順に選択します。 クラスに **Movie** と名前を付けます。

`Movie` クラスに次のプロパティを追加します。

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

`Movie` クラスには次が含まれます。

* `ID` フィールドは、データベースで主キー用に必要です。
* `[DataType(DataType.Date)]`: [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) 属性では、データの型 (`Date`) を指定します。 この属性を使用する場合:

  * ユーザーは日付フィールドに時刻の情報を入力する必要はありません。
  * 日付のみが表示され、時刻の情報は表示されません。

[DataAnnotations](xref:System.ComponentModel.DataAnnotations) は、後のチュートリアルで説明されます。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* *Models* という名前のフォルダーを追加します。
* *Movie.cs* という名前の *Models* フォルダーにクラスを追加します。

`Movie` クラスに次のプロパティを追加します。

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

`Movie` クラスには次が含まれます。

* `ID` フィールドは、データベースで主キー用に必要です。
* `[DataType(DataType.Date)]`: [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) 属性では、データの型 (`Date`) を指定します。 この属性を使用する場合:

  * ユーザーは日付フィールドに時刻の情報を入力する必要はありません。
  * 日付のみが表示され、時刻の情報は表示されません。

[DataAnnotations](xref:System.ComponentModel.DataAnnotations) は、後のチュートリアルで説明されます。

<a name="dc"></a>

### <a name="add-nuget-packages-and-ef-tools"></a>NuGet パッケージと EF ツールを追加します。

[!INCLUDE[](~/includes/add-EF-NuGet-SQLite-CLI.md)]

### <a name="add-a-database-context-class"></a>データベース コンテキスト クラスの追加

* *RazorPagesMovie* プロジェクト内に、*Data* という名前の新しいフォルダーを作成します。
* 次の `RazorPagesMovieContext` クラスを *Data* フォルダーに追加します。

  [!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Data/RazorPagesMovieContext.cs)]

上記のコードによって、エンティティ セットの `DbSet` プロパティが作成されます。 Entity Framework の用語では、エンティティ セットは通常はデータベース テーブルに対応し、エンティティはテーブルの行に対応します。 このコードは、後の手順で依存関係が追加されるまでコンパイルされません。

<a name="cs"></a>

### <a name="add-a-database-connection-string"></a>データベース接続文字列の追加

次の強調表示されたコードに示されているように、 *appsettings.json* ファイルに接続文字列を追加します。

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/appsettings_SQLite.json?highlight=10-12)]

<a name="reg"></a>

### <a name="register-the-database-context"></a>データベース コンテキストの登録

*Startup.cs* の先頭に次の `using` ステートメントを追加します。

```csharp
using RazorPagesMovie.Data;
using Microsoft.EntityFrameworkCore;
```

`Startup.ConfigureServices` で[依存性の挿入](xref:fundamentals/dependency-injection)コンテナーを使用し、データベース コンテキストを登録します。

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_UseSqlite&highlight=5-6)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* **ソリューション ツール ウィンドウ** で、Ctrl キーを押したまま **RazorPagesMovie** プロジェクトをクリックしてから、 **[追加]** > **[新しいフォルダー...]** を選択します。フォルダーに「*Models*」という名前を付けます。
* *Models* フォルダーを右クリックし、 **[追加]** > **[新しいファイル]** の順に選択します。
* **[新しいファイル]** ダイアログで次を実行します。

  * 左側のウィンドウで **[全般]** を選択します。
  * 中央ウィンドウで **[空のクラス]** を選択します。
  * クラスに **Movie** という名前を付け、 **[新規]** を選択します。

`Movie` クラスに次のプロパティを追加します。

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

`Movie` クラスには次が含まれます。

* `ID` フィールドは、データベースで主キー用に必要です。
* `[DataType(DataType.Date)]`: [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) 属性では、データの型 (`Date`) を指定します。 この属性を使用する場合:

  * ユーザーは日付フィールドに時刻の情報を入力する必要はありません。
  * 日付のみが表示され、時刻の情報は表示されません。

---

[DataAnnotations](xref:System.ComponentModel.DataAnnotations) は、後のチュートリアルで説明されます。

プロジェクトをビルドして、コンパイル エラーがないことを確認します。

## <a name="scaffold-the-movie-model"></a>ムービー モデルのスキャフォールディング

このセクションでは、ムービー モデルがスキャフォールディングされます。 つまり、スキャフォールディング ツールにより、ムービー モデルの作成、読み取り、更新、削除の (CRUD) 操作用のページが生成されます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

*Pages/Movies* フォルダーを作成します。

* *Pages* フォルダーを右クリックし、 **[追加]** > **[新しいフォルダー]** を選択します。
* フォルダーに *Movies* という名前を付けます。

*Pages/Movies* フォルダーを右クリックし、 **[追加]** > **[スキャフォールディングされた新しい項目]** を選択します。

![前の手順からのイメージ。](model/_static/sca.png)

**[スキャフォールディングを追加]** ダイアログで、 **[Entity Framework を使用する Razor ページ (CRUD)]** > **[追加]** の順に選択します。

![前の手順からのイメージ。](model/_static/add_scaffold.png)

**[Add Razor Pages using Entity Framework (CRUD)]\(Entity Framework を使用して Razor Pages (CRUD) を追加する\)** ダイアログを完了します。

* **[モデル クラス]** ドロップ ダウンで、 **[Movie (RazorPagesMovie.Models)]** を選択します。
* **データ コンテキスト クラス** 行で、 **+** (プラス) 記号を選択し、生成された名前 RazorPagesMovie.**Models**.RazorPagesMovieContext を RazorPagesMovie.**Data**.RazorPagesMovieContext に変更します。 [この変更](https://developercommunity.visualstudio.com/content/problem/652166/aspnet-core-ef-scaffolder-uses-incorrect-namespace.html)は必須ではありません。 これにより、正しい名前空間を使用してデータベース コンテキスト クラスが作成されます。
* **[追加]** を選びます。

![前の手順からのイメージ。](model/_static/3/arp.png)

*appsettings.json* ファイルは、ローカル データベースへの接続に使用される接続文字列を使用して更新されます。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

<!--  Until https://github.com/aspnet/Scaffolding/issues/582 is fixed windows needs backslash or the namespace is namespace RazorPagesMovie.Pages_Movies rather than namespace RazorPagesMovie.Pages.Movies
-->

* *Program.cs*、*Startup.cs*、および *.csproj* ファイルが含まれるプロジェクト ディレクトリでコマンド ウィンドウを開きます。

* **Windows の場合**:次のコマンドを実行します。

  ```dotnetcli
  dotnet-aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages\Movies --referenceScriptLibraries
  ```

* **macOS および Linux の場合**:次のコマンドを実行します。

  ```dotnetcli
  dotnet-aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages/Movies --referenceScriptLibraries
  ```

<a name="codegenerator"></a> 次の表で、ASP.NET Core コード ジェネレーターのオプションについて詳しく説明します。

| オプション               | 説明|
| ----------------- | ------------ |
| `-m`  | モデルの名前。 |
| `-dc`  | 使用する `DbContext` クラス。 |
| `-udl` | 既定のレイアウトを使用します。 |
| `-outDir` | ビューを作成するための相対出力フォルダー パス。 |
| `--referenceScriptLibraries` | [編集] および [作成] ページに `_ValidationScriptsPartial` を追加します。 |

`aspnet-codegenerator razorpage` コマンドに関するヘルプを取得するには、`-h` オプションを使用します。

```dotnetcli
dotnet-aspnet-codegenerator razorpage -h
```

詳細については、「[dotnet aspnet-codegenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator)」を参照してください。

### <a name="use-sqlite-for-development-sql-server-for-production"></a>開発用に SQLite を、運用環境に SQL Server を使用する

SQLite が選択されている場合、テンプレートで生成されたコードは開発の準備ができています。 次のコードでは、スタートアップに <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> を挿入する方法を示します。 `IWebHostEnvironment` が挿入されているので、`ConfigureServices` では開発用に SQLite を、運用環境に SQL Server を使用できます。

[!code-csharp[](~/includes/RP/code/StartupDevProd.cs?name=snippet&highlight=5,10,14)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

*Pages/Movies* フォルダーを作成します。

* *Pages* フォルダーを右クリックし、 **[追加]** > **[新しいフォルダー]** を選択します。
* フォルダーに *Movies* という名前を付けます。

*Pages/Movies* フォルダーを右クリックし、 **[追加]** > **[新しいスキャフォールディング...]** を選択します。

![前の手順からのイメージ。](model/_static/scaMac.png)

**[新しいスキャフォールディング]** ダイアログで、 **[Entity Framework を使用する Razor Pages (CRUD)]** > **[次へ]** を選択します。

![前の手順からのイメージ。](model/_static/add_scaffoldMac.png)

**[Add Razor Pages using Entity Framework (CRUD)]\(Entity Framework を使用して Razor Pages (CRUD) を追加する\)** ダイアログを完了します。

* **[モデル クラス]** ドロップ ダウンで、 **[Movie (RazorPagesMovie.Models)]** を選択するか、または入力します。
* **データ コンテキスト クラス** 行で、新しいクラスの名前「RazorPagesMovie.**Data**.RazorPagesMovieContext」と入力します。 [この変更](https://developercommunity.visualstudio.com/content/problem/652166/aspnet-core-ef-scaffolder-uses-incorrect-namespace.html)は必須ではありません。 これにより、正しい名前空間を使用してデータベース コンテキスト クラスが作成されます。
* **[追加]** を選びます。

![前の手順からのイメージ。](model/_static/arpMac.png)

*appsettings.json* ファイルは、ローカル データベースへの接続に使用される接続文字列を使用して更新されます。

### <a name="add-ef-tools"></a>EF ツールの追加

次の .NET Core CLI コマンドを実行します。

```dotnetcli
dotnet tool install --global dotnet-ef
```

上記のコマンドを実行すると、.NET Core CLI の Entity Framework Core ツールが追加されます。 詳細については、「[Entity Framework Core ツール リファレンス - .NET Core CLI](/ef/core/miscellaneous/cli/dotnet)」を参照してください。

### <a name="use-sqlite-for-development-sql-server-for-production"></a>開発用に SQLite を、運用環境に SQL Server を使用する

SQLite が選択されている場合、テンプレートで生成されたコードは開発の準備ができています。 次のコードでは、スタートアップに <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> を挿入する方法を示します。 `IWebHostEnvironment` が挿入されているので、`ConfigureServices` では開発用に SQLite を、運用環境に SQL Server を使用できます。

[!code-csharp[](~/includes/RP/code/StartupDevProd.cs?name=snippet&highlight=5,10,14)]

---

### <a name="files-created"></a>作成されたファイル

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

スキャフォールディングのプロセスが作成され、次のファイルが更新されます。

* *Pages/Movies*: Create、Delete、Details、Edit、および Index。
* *Data/RazorPagesMovieContext.cs*

### <a name="updated"></a>更新済み

* *Startup.cs*

作成および更新されたファイルについては、次のセクションで説明します。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

スキャフォールディングのプロセスが作成され、次のファイルが更新されます。

* *Pages/Movies*: Create、Delete、Details、Edit、および Index。
* *Data/RazorPagesMovieContext.cs*

### <a name="updated"></a>更新済み

* *Startup.cs*

作成および更新されたファイルについては、次のセクションで説明します。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

スキャフォールディングのプロセスでは、次のファイルが作成されます。

* *Pages/Movies*: Create、Delete、Details、Edit、および Index。

作成されたファイルについては、次のセクションで説明します。

---

<a name="pmc"></a>

## <a name="initial-migration"></a>最初の移行

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

このセクションでは、パッケージ マネージャー コンソール (PMC) を使用して、次の作業を行います。

* 初期移行を追加します。
* 初期移行でデータベースを更新します。

**[ツール]** メニューで、 **[NuGet パッケージ マネージャー]** > **[パッケージ マネージャー コンソール]** の順に選択します。

  ![PMC メニュー](../first-mvc-app/adding-model/_static/pmc.png)

PMC で、次のコマンドを入力します。

```powershell
Add-Migration InitialCreate
Update-Database
```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!INCLUDE [more information on the CLI for EF Core](~/includes/ef-cli.md)]

次の .NET Core CLI コマンドを実行します。

```dotnetcli
dotnet ef migrations add InitialCreate
dotnet ef database update
```

---

上記のコマンドで次の警告が生成されます。"エンティティ型 'Movie' の decimal 列 'Price' に型が指定されていません。 これにより、値が既定の有効桁数と小数点以下桁数に収まらない場合、自動的に切り捨てられます。 'HasColumnType()' を使用してすべての値に適合する SQL server 列の型を明示的に指定します。"

この警告は、後の手順で解決されるため、無視してください。

移行コマンドによって、最初のデータベース スキーマを作成するコードが生成されます。 このスキーマは、`DbContext` で指定されたモデルに基づきます。 `InitialCreate` 引数は移行の命名に使用されます。 任意の名前を使用できますが、規則により、移行を説明する名前が選択されます。

`update` コマンドにより、適用されていない移行で `Up` メソッドが実行されます。 この場合、`update` により、*Migrations/\<time-stamp>_InitialCreate.cs* ファイルで `Up` メソッドが実行され、データベースが作成されます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

### <a name="examine-the-context-registered-with-dependency-injection"></a>依存関係挿入に登録されるコンテキストを調べる

ASP.NET Core には、[依存関係挿入](xref:fundamentals/dependency-injection)が組み込まれています。 アプリケーションの起動時に、EF Core データベース コンテキスト context などのサービスが依存関係の挿入に登録されます。 これらのサービスを必要とするコンポーネント (Razor Pages など) には、コンストラクターのパラメーターを介してこれらのサービスが指定されます。 この後のチュートリアルで、データベース コンテキスト context インスタンスを取得するコンストラクター コードを示します。

スキャフォールディング ツールによってデータベース コンテキスト context が自動的に作成され、依存関係挿入コンテナーに登録されました。

`Startup.ConfigureServices` メソッドを調べます。 強調表示された行は、スキャフォールダーによって追加されました。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

`RazorPagesMovieContext` によって、`Movie` モデル用の EF Core 機能 (作成、読み取り、更新、削除など) が調整されます。 データ コンテキスト (`RazorPagesMovieContext`) は [Microsoft.EntityFrameworkCore.DbContext](xref:Microsoft.EntityFrameworkCore.DbContext) から派生されます。 データ コンテキストによって、データ モデルに含めるエンティティが指定されます。

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Data/RazorPagesMovieContext.cs)]

上記のコードによって、エンティティ セットの [DbSet\<Movie>](/dotnet/api/microsoft.entityframeworkcore.dbset-1) プロパティが作成されます。 Entity Framework の用語では、エンティティ セットは通常はデータベース テーブルに対応します。 エンティティはテーブル内の行に対応します。

[DbContextOptions](xref:Microsoft.EntityFrameworkCore.DbContextOptions) オブジェクトでメソッドが呼び出され、接続文字列の名前がコンテキストに渡されます。 ローカル開発の場合、[構成システム](xref:fundamentals/configuration/index)によって *appsettings.json* ファイルから接続文字列が読み取られます。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

`Up` メソッドを調べます。

---

<a name="test"></a>

### <a name="test-the-app"></a>アプリのテスト

* アプリを実行し、ブラウザーで URL に `/Movies` を追加します ( `http://localhost:port/movies` )。

エラーが発生した場合は、次のようにします。

```console
SqlException: Cannot open database "RazorPagesMovieContext-GUID" requested by the login. The login failed.
Login failed for user 'User-name'.
```

[移行手順](#pmc)を失敗しました。

* **[作成]** リンクをテストします。

  ![[作成] ページ](model/_static/conan5.png)

  > [!NOTE]
  > `Price` フィールドに小数点のコンマを入力できない場合があります。 小数点にコンマ (",") を使う英語以外のロケール、および英語 (米国) 以外の日付形式で、[jQuery 検証](https://jqueryvalidation.org/)をサポートするには、アプリをグローバル化する必要があります。 グローバル化の手順については、[この GitHub の記事](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420)をご覧ください。

* **[編集]** 、 **[詳細]** 、および **[削除]** の各リンクをテストします。

次のチュートリアルでは、スキャフォールディングによって作成されるファイルについて説明します。

## <a name="additional-resources"></a>その他の技術情報

> [!div class="step-by-step"]
> [前へ:はじめに](xref:tutorials/razor-pages/razor-pages-start)
> [次: スキャフォールディングされた Razor ページ](xref:tutorials/razor-pages/page)

::: moniker-end

<!--  ::: moniker previous version   -->
::: moniker range="< aspnetcore-3.0"

このセクションでは、クロスプラットフォーム [SQLite データベース](https://www.sqlite.org/index.html)でムービーを管理するためのクラスを追加します。 ASP.NET Core テンプレートから作成されたアプリでは、SQLite データベースが使用されます。 このアプリのモデル クラスは、データベースを操作するために [Entity Framework Core (EF Core)](/ef/core) ([SQLite EF Core データベース プロバイダー](/ef/core/providers/sqlite)) で使用されます。 EF Core は、データ アクセスを簡略化するオブジェクト リレーショナル マッピング (ORM) フレームワークです。

このモデル クラスは、EF Core に対する依存関係がないために、POCO クラス (plain-old CLR オブジェクト、つまり単純な従来の CLR) と呼ばれます。 これらは、データベースに格納されるデータのプロパティを定義します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="add-a-data-model"></a>データ モデルの追加

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

**RazorPagesMovie** プロジェクトを右クリックし、 **[追加]** 、 **[新しいフォルダー]** の順に選択します。 フォルダーに「*Models*」という名前を付けます。

*Models* フォルダーを右クリックします。 **[追加]**  >  **[クラス]** の順に選択します。 クラスに **Movie** と名前を付けます。

`Movie` クラスに次のプロパティを追加します。

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

`Movie` クラスには次が含まれます。

* `ID` フィールドは、データベースで主キー用に必要です。
* `[DataType(DataType.Date)]`: [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) 属性では、データの型 (`Date`) を指定します。 この属性を使用する場合:

  * ユーザーは日付フィールドに時刻の情報を入力する必要はありません。
  * 日付のみが表示され、時刻の情報は表示されません。

[DataAnnotations](xref:System.ComponentModel.DataAnnotations) は、後のチュートリアルで説明されます。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* *Models* という名前のフォルダーを追加します。
* *Movie.cs* という名前の *Models* フォルダーにクラスを追加します。

`Movie` クラスに次のプロパティを追加します。

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

`Movie` クラスには次が含まれます。

* `ID` フィールドは、データベースで主キー用に必要です。
* `[DataType(DataType.Date)]`: [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) 属性では、データの型 (`Date`) を指定します。 この属性を使用する場合:

  * ユーザーは日付フィールドに時刻の情報を入力する必要はありません。
  * 日付のみが表示され、時刻の情報は表示されません。

[DataAnnotations](xref:System.ComponentModel.DataAnnotations) は、後のチュートリアルで説明されます。

<a name="dc"></a>

### <a name="add-nuget-packages-and-ef-tools"></a>NuGet パッケージと EF ツールを追加します。

[!INCLUDE[](~/includes/add-EF-NuGet-SQLite-CLI.md)]

### <a name="add-a-database-context-class"></a>データベース コンテキスト クラスの追加

Razor PagesMovie プロジェクト内に、*Data* という名前の新しいフォルダーを作成します。 
次の `RazorPagesMovieContext` クラスを *Data* フォルダーに追加します。

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Data/RazorPagesMovieContext.cs)]

上記のコードによって、エンティティ セットの `DbSet` プロパティが作成されます。 Entity Framework の用語では、エンティティ セットは通常はデータベース テーブルに対応し、エンティティはテーブルの行に対応します。 このコードは、後の手順で依存関係が追加されるまでコンパイルされません。

<a name="cs"></a>

### <a name="add-a-database-connection-string"></a>データベース接続文字列の追加

次の強調表示されたコードに示されているように、 *appsettings.json* ファイルに接続文字列を追加します。

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/appsettings_SQLite.json?highlight=8-10)]

### <a name="add-required-nuget-packages"></a>必要な NuGet パッケージの追加

次の .NET Core CLI コマンドを実行し、SQLite と CodeGeneration.Design をプロジェクトに追加します。

```dotnetcli
dotnet add package Microsoft.EntityFrameworkCore.Sqlite --version 2.2.6
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design --version 2.2.4
dotnet add package Microsoft.EntityFrameworkCore.Design --version 2.2.6
```

スキャフォールディングには `Microsoft.VisualStudio.Web.CodeGeneration.Design` パッケージが必要です。

<a name="reg"></a>

### <a name="register-the-database-context"></a>データベース コンテキストの登録

*Startup.cs* の先頭に次の `using` ステートメントを追加します。

```csharp
using RazorPagesMovie.Models;
using Microsoft.EntityFrameworkCore;
```

`Startup.ConfigureServices` で[依存性の挿入](xref:fundamentals/dependency-injection)コンテナーを使用し、データベース コンテキストを登録します。

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_UseSqlite&highlight=11-12)]

エラー チェックとしてプロジェクトをビルドします。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* **ソリューション ツール ウィンドウ** で、Ctrl キーを押したまま *RazorPagesMovie* プロジェクトをクリックしてから、 **[追加]**  >  **[新しいフォルダー]** を選択します。 フォルダーに「*Models*」という名前を付けます。
* Ctrl キーを押したまま *Models* フォルダーをクリックし、 **[追加]** > **[新しいファイル]** を選択します。
* **[新しいファイル]** ダイアログで次を実行します。

  * 左側のウィンドウで **[全般]** を選択します。
  * 中央ウィンドウで **[空のクラス]** を選択します。
  * クラスに **Movie** という名前を付け、 **[新規]** を選択します。

`Movie` クラスに次のプロパティを追加します。

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

`Movie` クラスには次が含まれます。

* `ID` フィールドは、データベースで主キー用に必要です。
* `[DataType(DataType.Date)]`: [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) 属性では、データの型 (`Date`) を指定します。 この属性を使用する場合:

  * ユーザーは日付フィールドに時刻の情報を入力する必要はありません。
  * 日付のみが表示され、時刻の情報は表示されません。

[DataAnnotations](xref:System.ComponentModel.DataAnnotations) は、後のチュートリアルで説明されます。

---

プロジェクトをビルドして、コンパイル エラーがないことを確認します。

## <a name="scaffold-the-movie-model"></a>ムービー モデルのスキャフォールディング

このセクションでは、ムービー モデルがスキャフォールディングされます。 つまり、スキャフォールディング ツールにより、ムービー モデルの作成、読み取り、更新、削除の (CRUD) 操作用のページが生成されます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

*Pages/Movies* フォルダーを作成します。

* *Pages* フォルダーを右クリックし、 **[追加]** > **[新しいフォルダー]** を選択します。
* フォルダーに *Movies* という名前を付けます。

*Pages/Movies* フォルダーを右クリックし、 **[追加]** > **[スキャフォールディングされた新しい項目]** を選択します。

![前の手順からのイメージ。](model/_static/sca.png)

**[スキャフォールディングを追加]** ダイアログで、 **[Entity Framework を使用する Razor ページ (CRUD)]** > **[追加]** の順に選択します。

![前の手順からのイメージ。](model/_static/add_scaffold.png)

**[Add Razor Pages using Entity Framework (CRUD)]\(Entity Framework を使用して Razor Pages (CRUD) を追加する\)** ダイアログを完了します。
<!-- In the next section, change 
(plus) sign and accept the generated name 
to use Data, it should not use models. That will make the namespace the same for the VS version and the CLI version
-->

* **[モデル クラス]** ドロップ ダウンで、 **[Movie (RazorPagesMovie.Models)]** を選択します。
* **データ コンテキスト クラス** 行で、 **+** (プラス) 記号を選択し、生成された名前 **RazorPagesMovie.Models.RazorPagesMovieContext** を受け入れます。
* **[追加]** を選びます。

![前の手順からのイメージ。](model/_static/arp.png)

*appsettings.json* ファイルは、ローカル データベースへの接続に使用される接続文字列を使用して更新されます。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

<!--  Until https://github.com/aspnet/Scaffolding/issues/582 is fixed windows needs backslash or the namespace is namespace RazorPagesMovie.Pages_Movies rather than namespace RazorPagesMovie.Pages.Movies
-->

* *Program.cs*、*Startup.cs*、および *.csproj* ファイルが含まれるプロジェクト ディレクトリでコマンド ウィンドウを開きます。

* **Windows の場合**:次のコマンドを実行します。

  ```dotnetcli
  dotnet-aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages\Movies --referenceScriptLibraries
  ```

* **macOS および Linux の場合**:次のコマンドを実行します。

  ```dotnetcli
  dotnet-aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages/Movies --referenceScriptLibraries
  ```

<a name="codegenerator"></a> 次の表で、ASP.NET Core コード ジェネレーターのオプションについて詳しく説明します。

| オプション               | 説明|
| ----------------- | ------------ |
| `-m`  | モデルの名前。 |
| `-dc`  | 使用する `DbContext` クラス。 |
| `-udl` | 既定のレイアウトを使用します。 |
| `-outDir` | ビューを作成するための相対出力フォルダー パス。 |
| `--referenceScriptLibraries` | [編集] および [作成] ページに `_ValidationScriptsPartial` を追加します。 |

`aspnet-codegenerator razorpage` コマンドに関するヘルプを取得するには、`-h` オプションを使用します。

```dotnetcli
dotnet-aspnet-codegenerator razorpage -h
```

詳細については、「[dotnet aspnet-codegenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator)」を参照してください。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

*Pages/Movies* フォルダーを作成します。

* Ctrl キーを押したまま *Pages* フォルダーをクリックし、 **[追加]** > **[新しいフォルダー]** を選択します。
* フォルダーに *Movies* という名前を付けます。

Ctrl キーを押したまま *Pages/Movies* フォルダーをクリックし、 **[追加]** > **[スキャフォールディングされた新しい項目]** を選択します。

![前の手順からのイメージ。](model/_static/scaMac.png)

**[新しいスキャフォールディングの追加]** ダイアログで、 **[Entity Framework を使用する Razor Pages (CRUD)]** > **[追加]** を選択します。

![前の手順からのイメージ。](model/_static/add_scaffoldMac.png)

**[Add Razor Pages using Entity Framework (CRUD)]\(Entity Framework を使用して Razor Pages (CRUD) を追加する\)** ダイアログを完了します。

* **[モデル クラス]** ドロップ ダウンで、 **[Movie]** を選択するか、または入力します。
* **データ コンテキスト クラス** 行で、 **RazorPagesMovieContext** を選択します。これにより、新しいデータベース コンテキスト クラスが正しい名前空間で作成されます。 このクラスでは、 **RazorPagesMovie.Models.RazorPagesMovieContext** となります。
* **[追加]** を選びます。

![前の手順からのイメージ。](model/_static/arpMac.png)

*appsettings.json* ファイルは、ローカル データベースへの接続に使用される接続文字列を使用して更新されます。

---

スキャフォールディングのプロセスが作成され、次のファイルが更新されます。

### <a name="files-created"></a>作成されたファイル

* *Pages/Movies*: Create、Delete、Details、Edit、および Index。
* *Data/RazorPagesMovieContext.cs*

### <a name="file-updated"></a>更新されたファイル

* *Startup.cs*

作成および更新されたファイルについては、次のセクションで説明します。

<a name="pmc"></a>

## <a name="initial-migration"></a>最初の移行

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

このセクションでは、パッケージ マネージャー コンソール (PMC) を使用して、次の作業を行います。

* 初期移行を追加します。
* 初期移行でデータベースを更新します。

**[ツール]** メニューで、 **[NuGet パッケージ マネージャー]** > **[パッケージ マネージャー コンソール]** の順に選択します。

  ![PMC メニュー](../first-mvc-app/adding-model/_static/pmc.png)

PMC で、次のコマンドを入力します。

```powershell
Add-Migration Initial
Update-Database
```

`Add-Migration` コマンドによって最初のデータベース スキーマを作成するコードが生成されます。 スキーマは、 *RazorPagesMovieContext.cs* ファイルの `DbContext` に指定されたモデルに基づきます。 `InitialCreate` 引数は、移行の名前を指定するために使用されます。 任意の名前を使用できますが、規則により、移行を説明する名前が使用されます。 詳細については、「<xref:data/ef-mvc/migrations>」を参照してください。

`Update-Database` コマンドによって、*Migrations/\<time-stamp>_InitialCreate.cs* ファイルの `Up` メソッドが実行されます。 `Up` メソッドにより、データベースが作成されます。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!INCLUDE [more information on the CLI for EF Core](~/includes/ef-cli.md)]

次の .NET Core CLI コマンドを実行します。

```dotnetcli
dotnet ef migrations add InitialCreate
dotnet ef database update
```

---

上記のコマンドで次の警告が生成されます。"エンティティ型 'Movie' の decimal 列 'Price' に型が指定されていません。 これにより、値が既定の有効桁数と小数点以下桁数に収まらない場合、自動的に切り捨てられます。 'HasColumnType()' を使用してすべての値に適合する SQL server 列の型を明示的に指定します。"

この警告は、後の手順で解決されるため、無視してください。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

### <a name="examine-the-context-registered-with-dependency-injection"></a>依存関係挿入に登録されるコンテキストを調べる

ASP.NET Core には、[依存関係挿入](xref:fundamentals/dependency-injection)が組み込まれています。 アプリケーションの起動時に、EF Core データベース コンテキスト context などのサービスが依存関係の挿入に登録されます。 これらのサービスを必要とするコンポーネント (Razor Pages など) には、コンストラクターのパラメーターを介してこれらのサービスが指定されます。 この後のチュートリアルで、データベース コンテキスト contextB コンテキスト インスタンスを取得するコンストラクター コードを示します。

スキャフォールディング ツールによってデータベース コンテキスト context が自動的に作成され、依存関係挿入コンテナーに登録されました。

`Startup.ConfigureServices` メソッドを調べます。 強調表示された行は、スキャフォールダーによって追加されました。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_ConfigureServices&highlight=15-18)]

`RazorPagesMovieContext` によって、`Movie` モデル用の EF Core 機能 (作成、読み取り、更新、削除など) が調整されます。 データ コンテキスト (`RazorPagesMovieContext`) は [Microsoft.EntityFrameworkCore.DbContext](xref:Microsoft.EntityFrameworkCore.DbContext) から派生されます。 データ コンテキストによって、データ モデルに含めるエンティティが指定されます。

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Data/RazorPagesMovieContext.cs)]

上記のコードによって、エンティティ セットの [DbSet\<Movie>](/dotnet/api/microsoft.entityframeworkcore.dbset-1) プロパティが作成されます。 Entity Framework の用語では、エンティティ セットは通常はデータベース テーブルに対応します。 エンティティはテーブル内の行に対応します。

[DbContextOptions](xref:Microsoft.EntityFrameworkCore.DbContextOptions) オブジェクトでメソッドが呼び出され、接続文字列の名前がコンテキストに渡されます。 ローカル開発の場合、[構成システム](xref:fundamentals/configuration/index)によって *appsettings.json* ファイルから接続文字列が読み取られます。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

`Up` メソッドを調べます。

---

<a name="test"></a>

### <a name="test-the-app"></a>アプリのテスト

* アプリを実行し、ブラウザーで URL に `/Movies` を追加します ( `http://localhost:port/movies` )。

エラーが発生した場合は、次のようにします。

```console
SqlException: Cannot open database "RazorPagesMovieContext-GUID" requested by the login. The login failed.
Login failed for user 'User-name'.
```

[移行手順](#pmc)を失敗しました。

* **[作成]** リンクをテストします。

  ![[作成] ページ](model/_static/conan.png)

  > [!NOTE]
  > `Price` フィールドに小数点のコンマを入力できない場合があります。 小数点にコンマ (",") を使う英語以外のロケール、および英語 (米国) 以外の日付形式で、[jQuery 検証](https://jqueryvalidation.org/)をサポートするには、アプリをグローバル化する必要があります。 グローバル化の手順については、[この GitHub の記事](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420)をご覧ください。

* **[編集]** 、 **[詳細]** 、および **[削除]** の各リンクをテストします。

次のチュートリアルでは、スキャフォールディングによって作成されるファイルについて説明します。

## <a name="additional-resources"></a>その他の技術情報

> [!div class="step-by-step"]
> [前へ:はじめに](xref:tutorials/razor-pages/razor-pages-start)
> [次: スキャフォールディングされた Razor ページ](xref:tutorials/razor-pages/page)

::: moniker-end
