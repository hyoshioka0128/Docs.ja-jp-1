---
title: パート 7、新しいフィールドの追加
author: rick-anderson
description: Razor ページのチュートリアル シリーズのパート 7。
ms.author: riande
ms.custom: mvc, contperf-fy21q2
ms.date: 09/28/2020
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
uid: tutorials/razor-pages/new-field
ms.openlocfilehash: cd637507e19735f020b4c28e6f22de0e7e772040
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588322"
---
# <a name="part-7-add-a-new-field-to-a-razor-page-in-aspnet-core"></a>パート 7、ASP.NET Core で Razor ページでの新しいフィールドの追加

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-5.0"

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

このセクションでは [Entity Framework](/ef/core/get-started/aspnetcore/new-db) Code First Migrations を次の目的で使用します。

* モデルに新しいフィールドを追加する。
* 新しいフィールド スキーマの変更をデータベースに移行する。

EF Code First を使用してデータベースを自動的に作成する場合、Code First では次が実行されます。

* データベースに [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) テーブルが追加され、データベースのスキーマが生成元のモデル クラスと同期しているかどうかが追跡されます。
* モデル クラスがデータベースと同期されていない場合、EF から例外がスローされます。

スキーマとモデルが同期中であることが自動的に検証されるようにすると、整合性のないデータベース コードの問題を発見しやすくなります。

## <a name="adding-a-rating-property-to-the-movie-model"></a>ムービー モデルへの評価プロパティの追加

1. *Models/Movie.cs* ファイルを開き、`Rating` プロパティを追加します。

   [!code-csharp[](razor-pages-start/sample/RazorPagesMovie50/Models/MovieDateRating.cs?highlight=13&name=snippet)]

1. アプリをビルドします。

1. *Pages/Movies/Index.cshtml* を編集し、`Rating` フィールドを追加します。

   <a name="addrat"></a>

   [!code-cshtml[](razor-pages-start/sample/RazorPagesMovie50/SnapShots/IndexRating.cshtml?highlight=40-42,62-64)]

1. 次のページを更新します。
   1. [削除] と [詳細] ページに、`Rating` フィールドを追加します。
   1. [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Pages/Movies/Create.cshtml) を `Rating` フィールドを使用して更新します。
   1. [編集] ページに、`Rating` フィールドを追加します。

データベースを更新して新しいフィールドが含まれるようになるまでアプリは動作しません。 データベースを更新せずにアプリを実行すると、次の `SqlException` がスローされます。

`SqlException: Invalid column name 'Rating'.`

`SqlException` 例外が発生するのは、更新された Movie モデル クラスが、データベースの Movie テーブルのスキーマと異なるためです。 データベース テーブルに `Rating` 列はありません。

このエラーを解決するための手法がいくつかあります。

1. Entity Framework に、新しいモデル クラス スキーマを使用してデータベースを自動的にドロップさせ、再作成させます。 この手法は、開発周期の早い段階で便利です。モデルとデータベース スキーマを一緒に短期間で発展させることができます。 これの欠点は、データベースで既存のデータが失われることです。 実稼働データベースでこの方法は使用しないでください。 スキーマの変更時にデータベースを削除し、初期化子を使用して、データベースにテスト データを自動的にシードすることは、多くの場合、アプリケーションを開発する上で有益な方法となります。

2. モデル クラスに一致するように、既存のデータベースのスキーマを明示的に変更します。 この手法の長所は、データが維持されることです。 この変更は手動で行うか、データベース変更スクリプトを作成して行います。

3. Code First Migrations を使用して、データベース スキーマを更新します。

このチュートリアルでは、Code First Migrations を利用します。

新しい列に値を提供するように、`SeedData` クラスを更新します。 下に変更のサンプルがありますが、`new Movie` ブロックごとにこの変更を行ってください。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie50/Models/SeedDataRating.cs?name=snippet1&highlight=8)]

[完成した SeedData.cs ファイル](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Models/SeedDataRating.cs)を参照してください。

ソリューションをビルドします。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

<a name="pmc"></a>

### <a name="add-a-migration-for-the-rating-field"></a>評価フィールドの移行を追加する

1. **[ツール]** メニューで、 **[NuGet パッケージ マネージャー]、[パッケージ マネージャー コンソール]** の順に選択します。
2. PMC で、次のコマンドを入力します。

   ```powershell
   Add-Migration Rating
   Update-Database
   ```

`Add-Migration` コマンドにより、フレームワークに次の指示があります。

* `Movie` モデルを、`Movie` データベースのスキーマと比較します。
* データベース スキーマを新しいモデルに移行するコードを作成します。

"Rating (評価)" という名前は任意です。移行ファイルに名前を付けるために利用されます。 移行ファイルには意味のある名前を使用すると便利です。

`Update-Database` コマンドを使うと、データベースにスキーマ変更を適用し、既存のデータを保持するようにフレームワークに指示できます。

<a name="ssox"></a>

データベース内のすべてのレコードを削除すると、初期化子はデータベースにデータを初期投入し、`Rating` フィールドを追加します。 これはブラウザーの削除リンクで行うか、[Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX) から行うことができます。

別のオプションとしては、データベースを削除してから、移行を使用することで、データベースを再作成することもできます。 SSOX 内でデータベースを削除するには:

1. SSOX でデータベースを選択します。
1. データベースを右クリックし、 **[削除]** を選択します。
1. **[既存の接続を閉じる]** をチェックします。
1. **[OK]** を選択します。
1. [PMC](xref:tutorials/razor-pages/new-field#pmc) でデータベースを更新します。

   ```powershell
   Update-Database
   ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

### <a name="drop-and-re-create-the-database"></a>データベースを削除して再作成する

> [!NOTE]
> このチュートリアルでは、可能な場合は Entity Framework Core の "*移行*" 機能を使います。 移行では、データ モデルの変更に合わせてデータベース スキーマが更新されます。 ただし、移行では EF Core プロバイダーでサポートされている種類の変更しか行うことができず、SQLite プロバイダーの機能は制限されています。 たとえば、列の追加はサポートされていますが、列の削除や変更はサポートされていません。 列を削除または変更するために移行を作成した場合、`ef migrations add` コマンドは成功しますが、`ef database update` コマンドは失敗します。 これらの制限により、このチュートリアルでは SQLite のスキーマの変更に対しては移行を使用しません。 代わりに、スキーマを変更するときは、データベースを削除して再作成します。
>
>SQLite の制限に対する回避策としては、テーブル内の何かが変更されたときにテーブルの再構築を実行する移行コードを、手動で作成します。 テーブルの再構築には次の作業が含まれます。
>
>* 新しいテーブルの作成。
>* 古いテーブルから新しいテーブルへのデータのコピー。
>* 古いテーブルの削除。
>* 新しいテーブルの名前変更。
>
>詳細については、次のリソースを参照してください。
>
> * [SQLite EF Core データベース プロバイダーの制限事項](/ef/core/providers/sqlite/limitations)
> * [移行コードをカスタマイズする](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [データのシード処理](/ef/core/modeling/data-seeding)
> * [SQLite ALTER TABLE ステートメント](https://sqlite.org/lang_altertable.html)

1. 移行フォルダーを削除します。  

1. 次のコマンドを使用して、データベースを再作成します。

   ```dotnetcli
   dotnet ef database drop
   dotnet ef migrations add InitialCreate
   dotnet ef database update
   ```

---

アプリを実行し、`Rating` フィールドでムービーを作成、編集、表示できることを確認します。 データベースがシード処理されない場合は、`SeedData.Initialize` メソッドでブレークポイントを設定します。

## <a name="additional-resources"></a>その他の技術情報

> [!div class="step-by-step"]
> [前へ:検索の追加](xref:tutorials/razor-pages/search)
> [次へ:検証の追加](xref:tutorials/razor-pages/validation)

::: moniker-end

::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

このセクションでは [Entity Framework](/ef/core/get-started/aspnetcore/new-db) Code First Migrations を次の目的で使用します。

* モデルに新しいフィールドを追加する。
* 新しいフィールド スキーマの変更をデータベースに移行する。

EF Code First を使用してデータベースを自動的に作成する場合、Code First では次が実行されます。

* データベースに [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) テーブルが追加され、データベースのスキーマが生成元のモデル クラスと同期しているかどうかが追跡されます。
* モデル クラスがデータベースと同期されていない場合、EF から例外がスローされます。

スキーマとモデルが同期中であることが自動的に検証されるようにすると、整合性のないデータベース コードの問題を発見しやすくなります。

## <a name="adding-a-rating-property-to-the-movie-model"></a>ムービー モデルへの評価プロパティの追加

1. *Models/Movie.cs* ファイルを開き、`Rating` プロパティを追加します。

   [!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/MovieDateRating.cs?highlight=13&name=snippet)]

1. アプリをビルドします。

1. *Pages/Movies/Index.cshtml* を編集し、`Rating` フィールドを追加します。

   <a name="addrat"></a>

   [!code-cshtml[](razor-pages-start/sample/RazorPagesMovie30/SnapShots/IndexRating.cshtml?highlight=40-42,62-64)]

1. 次のページを更新します。
   1. [削除] と [詳細] ページに、`Rating` フィールドを追加します。
   1. [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Create.cshtml) を `Rating` フィールドを使用して更新します。
   1. [編集] ページに、`Rating` フィールドを追加します。

データベースを更新して新しいフィールドが含まれるようになるまでアプリは動作しません。 データベースを更新せずにアプリを実行すると、次の `SqlException` がスローされます。

`SqlException: Invalid column name 'Rating'.`

`SqlException` 例外が発生するのは、更新された Movie モデル クラスが、データベースの Movie テーブルのスキーマと異なるためです。 データベース テーブルに `Rating` 列はありません。

このエラーを解決するための手法がいくつかあります。

1. Entity Framework に、新しいモデル クラス スキーマを使用してデータベースを自動的にドロップさせ、再作成させます。 この手法は、開発周期の早い段階で便利です。モデルとデータベース スキーマを一緒に短期間で発展させることができます。 これの欠点は、データベースで既存のデータが失われることです。 実稼働データベースでこの方法は使用しないでください。 スキーマの変更時にデータベースを削除し、初期化子を使用して、データベースにテスト データを自動的にシードすることは、多くの場合、アプリケーションを開発する上で有益な方法となります。

2. モデル クラスに一致するように、既存のデータベースのスキーマを明示的に変更します。 この手法の長所は、データが維持されることです。 この変更は手動で行うか、データベース変更スクリプトを作成して行います。

3. Code First Migrations を使用して、データベース スキーマを更新します。

このチュートリアルでは、Code First Migrations を利用します。

新しい列に値を提供するように、`SeedData` クラスを更新します。 下に変更のサンプルがありますが、`new Movie` ブロックごとにこの変更を行ってください。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/SeedDataRating.cs?name=snippet1&highlight=8)]

[完成した SeedData.cs ファイル](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Models/SeedDataRating.cs)を参照してください。

ソリューションをビルドします。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

<a name="pmc"></a>

### <a name="add-a-migration-for-the-rating-field"></a>評価フィールドの移行を追加する

1. **[ツール]** メニューで、 **[NuGet パッケージ マネージャー]、[パッケージ マネージャー コンソール]** の順に選択します。
2. PMC で、次のコマンドを入力します。

   ```powershell
   Add-Migration Rating
   Update-Database
   ```

`Add-Migration` コマンドにより、フレームワークに次の指示があります。

* `Movie` モデルを、`Movie` データベースのスキーマと比較します。
* データベース スキーマを新しいモデルに移行するコードを作成します。

"Rating (評価)" という名前は任意です。移行ファイルに名前を付けるために利用されます。 移行ファイルには意味のある名前を使用すると便利です。

`Update-Database` コマンドを使うと、データベースにスキーマ変更を適用し、既存のデータを保持するようにフレームワークに指示できます。

<a name="ssox"></a>

データベース内のすべてのレコードを削除すると、初期化子はデータベースにデータを初期投入し、`Rating` フィールドを追加します。 これはブラウザーの削除リンクで行うか、[Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX) から行うことができます。

別のオプションとしては、データベースを削除してから、移行を使用することで、データベースを再作成することもできます。 SSOX 内でデータベースを削除するには:

* SSOX でデータベースを選択します。
* データベースを右クリックし、 **[削除]** を選択します。
* **[既存の接続を閉じる]** をチェックします。
* **[OK]** を選択します。
* [PMC](xref:tutorials/razor-pages/new-field#pmc) でデータベースを更新します。

  ```powershell
  Update-Database
  ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

### <a name="drop-and-re-create-the-database"></a>データベースを削除して再作成する

> [!NOTE]
> このチュートリアルでは、可能な場合は Entity Framework Core の "*移行*" 機能を使います。 移行では、データ モデルの変更に合わせてデータベース スキーマが更新されます。 ただし、移行では EF Core プロバイダーでサポートされている種類の変更しか行うことができず、SQLite プロバイダーの機能は制限されています。 たとえば、列の追加はサポートされていますが、列の削除や変更はサポートされていません。 列を削除または変更するために移行を作成した場合、`ef migrations add` コマンドは成功しますが、`ef database update` コマンドは失敗します。 これらの制限により、このチュートリアルでは SQLite のスキーマの変更に対しては移行を使用しません。 代わりに、スキーマを変更するときは、データベースを削除して再作成します。
>
>SQLite の制限に対する回避策としては、テーブル内の何かが変更されたときにテーブルの再構築を実行する移行コードを、手動で作成します。 テーブルの再構築には次の作業が含まれます。
>
>* 新しいテーブルの作成。
>* 古いテーブルから新しいテーブルへのデータのコピー。
>* 古いテーブルの削除。
>* 新しいテーブルの名前変更。
>
>詳細については、次のリソースを参照してください。
>
> * [SQLite EF Core データベース プロバイダーの制限事項](/ef/core/providers/sqlite/limitations)
> * [移行コードをカスタマイズする](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [データのシード処理](/ef/core/modeling/data-seeding)
> * [SQLite ALTER TABLE ステートメント](https://sqlite.org/lang_altertable.html)

1. 移行フォルダーを削除します。  

1. 次のコマンドを使用して、データベースを再作成します。

   ```dotnetcli
   dotnet ef database drop
   dotnet ef migrations add InitialCreate
   dotnet ef database update
   ```

---

アプリを実行し、`Rating` フィールドでムービーを作成、編集、表示できることを確認します。 データベースがシード処理されない場合は、`SeedData.Initialize` メソッドでブレークポイントを設定します。

## <a name="additional-resources"></a>その他の技術情報

> [!div class="step-by-step"]
> [前へ:検索の追加](xref:tutorials/razor-pages/search)
> [次へ:検証の追加](xref:tutorials/razor-pages/validation)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

このセクションでは [Entity Framework](/ef/core/get-started/aspnetcore/new-db) Code First Migrations を次の目的で使用します。

* モデルに新しいフィールドを追加する。
* 新しいフィールド スキーマの変更をデータベースに移行する。

EF Code First を使用してデータベースを自動的に作成する場合、Code First では次が実行されます。

* データベースに [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) テーブルが追加され、データベースのスキーマが生成元のモデル クラスと同期しているかどうかが追跡されます。
* モデル クラスがデータベースと同期されていない場合、EF から例外がスローされます。

スキーマとモデルが同期中であることが自動的に検証されるようにすると、整合性のないデータベース コードの問題を発見しやすくなります。

## <a name="adding-a-rating-property-to-the-movie-model"></a>ムービー モデルへの評価プロパティの追加

*Models/Movie.cs* ファイルを開き、`Rating` プロパティを追加します。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/MovieDateRating.cs?highlight=13&name=snippet)]

アプリをビルドします。

*Pages/Movies/Index.cshtml* を編集し、`Rating` フィールドを追加します。

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/IndexRating.cshtml?highlight=40-42,61-63)]

次のページを更新します。

* [削除] と [詳細] ページに、`Rating` フィールドを追加します。
* [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Create.cshtml) を `Rating` フィールドを使用して更新します。
* [編集] ページに、`Rating` フィールドを追加します。

データベースを更新して新しいフィールドが含まれるようになるまでアプリは動作しません。 ここでアプリが実行されると、アプリによって `SqlException` がスローされます。

`SqlException: Invalid column name 'Rating'.`

このエラーが表示されるのは、更新された Movie モデル クラスがデータベースの Movie テーブルのスキーマと異なるためです。 データベース テーブルに `Rating` 列はありません。

このエラーを解決するための手法がいくつかあります。

1. Entity Framework に、新しいモデル クラス スキーマを使用してデータベースを自動的にドロップさせ、再作成させます。 この手法は、開発周期の早い段階で便利です。モデルとデータベース スキーマを一緒に短期間で発展させることができます。 これの欠点は、データベースで既存のデータが失われることです。 実稼働データベースでこの方法は使用しないでください。 スキーマの変更時にデータベースを削除し、初期化子を使用して、データベースにテスト データを自動的にシードすることは、多くの場合、アプリケーションを開発する上で有益な方法となります。

2. モデル クラスに一致するように、既存のデータベースのスキーマを明示的に変更します。 この手法の長所は、データが維持されることです。 この変更は手動で行うか、データベース変更スクリプトを作成して行います。

3. Code First Migrations を使用して、データベース スキーマを更新します。

このチュートリアルでは、Code First Migrations を利用します。

新しい列に値を提供するように、`SeedData` クラスを更新します。 下に変更のサンプルがありますが、`new Movie` ブロックごとにこの変更を行ってください。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/SeedDataRating.cs?name=snippet1&highlight=8)]

[完成した SeedData.cs ファイル](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/SeedDataRating.cs)を参照してください。

ソリューションをビルドします。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

<a name="pmc"></a>

### <a name="add-a-migration-for-the-rating-field"></a>評価フィールドの移行を追加する

**[ツール]** メニューで、 **[NuGet パッケージ マネージャー]、[パッケージ マネージャー コンソール]** の順に選択します。
PMC で、次のコマンドを入力します。

```powershell
Add-Migration Rating
Update-Database
```

`Add-Migration` コマンドにより、フレームワークに次の指示があります。

* `Movie` モデルを、`Movie` データベースのスキーマと比較します。
* データベース スキーマを新しいモデルに移行するコードを作成します。

"Rating (評価)" という名前は任意です。移行ファイルに名前を付けるために利用されます。 移行ファイルには意味のある名前を使用すると便利です。

フレームワークは、データベースにスキーマ変更を適用するように、`Update-Database` コマンドから指示されます。

<a name="ssox"></a>

データベース内のすべてのレコードを削除すると、初期化子はデータベースにデータを初期投入し、`Rating` フィールドを追加します。 これはブラウザーの削除リンクで行うか、[Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX) から行うことができます。

別のオプションとしては、データベースを削除してから、移行を使用することで、データベースを再作成することもできます。 SSOX 内でデータベースを削除するには:

* SSOX でデータベースを選択します。
* データベースを右クリックし、 **[削除]** を選択します。
* **[既存の接続を閉じる]** をチェックします。
* **[OK]** を選択します。
* [PMC](xref:tutorials/razor-pages/new-field#pmc) でデータベースを更新します。

  ```powershell
  Update-Database
  ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

### <a name="drop-and-re-create-the-database"></a>データベースを削除して再作成する

> [!NOTE]
> このチュートリアルでは、可能な場合は Entity Framework Core の "*移行*" 機能を使います。 移行では、データ モデルの変更に合わせてデータベース スキーマが更新されます。 ただし、移行では EF Core プロバイダーでサポートされている種類の変更しか行うことができず、SQLite プロバイダーの機能は制限されています。 たとえば、列の追加はサポートされていますが、列の削除や変更はサポートされていません。 列を削除または変更するために移行を作成した場合、`ef migrations add` コマンドは成功しますが、`ef database update` コマンドは失敗します。 これらの制限により、このチュートリアルでは SQLite のスキーマの変更に対しては移行を使用しません。 代わりに、スキーマを変更するときは、データベースを削除して再作成します。
>
>SQLite の制限に対する回避策としては、テーブル内の何かが変更されたときにテーブルの再構築を実行する移行コードを、手動で作成します。 テーブルの再構築には次の作業が含まれます。
>
>* 新しいテーブルの作成。
>* 古いテーブルから新しいテーブルへのデータのコピー。
>* 古いテーブルの削除。
>* 新しいテーブルの名前変更。
>
>詳細については、次のリソースを参照してください。
>
> * [SQLite EF Core データベース プロバイダーの制限事項](/ef/core/providers/sqlite/limitations)
> * [移行コードをカスタマイズする](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [データのシード処理](/ef/core/modeling/data-seeding)
> * [SQLite ALTER TABLE ステートメント](https://sqlite.org/lang_altertable.html)

データベースを削除し、移行を使ってデータベースを再作成します。 データベースを削除するには、データベース ファイル (*MvcMovie.db*) を削除します。 次に、`ef database update` コマンドを実行します。

```dotnetcli
dotnet ef database update
```

---

アプリを実行し、`Rating` フィールドでムービーを作成、編集、表示できることを確認します。 データベースがシード処理されない場合は、`SeedData.Initialize` メソッドでブレークポイントを設定します。

## <a name="additional-resources"></a>その他の技術情報

* [このチュートリアルの YouTube バージョン](https://youtu.be/3i7uMxiGGR8)

> [!div class="step-by-step"]
> [前へ:検索の追加](xref:tutorials/razor-pages/search)
> [次へ:検証の追加](xref:tutorials/razor-pages/validation)

::: moniker-end