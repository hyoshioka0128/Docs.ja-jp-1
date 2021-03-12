---
title: パート 4、データベースの操作
author: rick-anderson
description: Razor ページのチュートリアル シリーズのパート 4。
ms.author: riande
ms.date: 01/05/2021
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
uid: tutorials/razor-pages/sql
ms.openlocfilehash: 3133ff7a192f5e3bdec4f6f1e6e6d55f966d7e64
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589765"
---
# <a name="part-4-of-tutorial-series-on-razor-pages"></a>Razor ページのチュートリアル シリーズのパート 4。

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT) および [Joe Audette](https://twitter.com/joeaudette)

::: moniker range=">= aspnetcore-5.0"

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

`RazorPagesMovieContext` オブジェクトは、データベースへの接続と、データベース レコードへの `Movie` オブジェクトのマッピングのタスクを処理します。 データベース コンテキストは、*Startup.cs* の `ConfigureServices` メソッドで [依存性の注入](xref:fundamentals/dependency-injection)コンテナーに登録されます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie50/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie50/Startup.cs?name=snippet_UseSqlite&highlight=5-6)]

---

ASP.NET Core の[構成](xref:fundamentals/configuration/index)システムでは `ConnectionString` キーが読み取られます。 ローカルで開発する場合は、構成によって *appsettings.json* ファイルから接続文字列が取得されます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

生成される接続文字列は、次の JSON のようになります。

[!code-json[](razor-pages-start/sample/RazorPagesMovie50/appsettings.json?highlight=10-12)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/appsettings_SQLite.json?highlight=10-12)]

---

アプリがテスト サーバーまたは運用サーバーにデプロイされると、環境変数を使用して接続文字列をテストまたは運用データベース サーバーに設定できます。 詳細については、[構成](xref:fundamentals/configuration/index)に関するページを参照してください。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

## <a name="sql-server-express-localdb"></a>SQL Server Express LocalDB

LocalDB は、プログラム開発を対象にした、SQL Server Express データベース エンジンの軽量版です。 LocalDB は要求時に開始され、ユーザー モードで実行されるため、複雑な構成はありません。 LocalDB データベースでは、既定で `C:\Users\<user>\` ディレクトリに `*.mdf` ファイルが作成されます。

<a name="ssox"></a>
1. **[表示]** メニューの **[SQL Server オブジェクト エクスプローラー]** (SSOX) を開きます。

   ![[View] メニュー](sql/_static/5/ssox.png)

1. `Movie` テーブルを右クリックし、 **[デザイナーの表示]** を選択します。

   ![Movie テーブルに対して開かれたコンテキスト メニュー](sql/_static/5/design.png)

   ![デザイナーに開かれた Movie テーブル](sql/_static/dv.png)

   `ID` の横のキー アイコンに注意してください。 既定では、EF で主キーに `ID` という名前のプロパティが作成されます。

1. `Movie` テーブルを右クリックし、 **[データの表示]** を選択します。

   ![開いた Movie テーブルにテーブル データが表示されています](sql/_static/vd22.png)

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

## <a name="sqlite"></a>SQLite

[SQLite](https://www.sqlite.org/) の Web サイトは次のようなものです。

> SQLite は埋め込みの全機能を備えた、自己完結型かつ高信頼性でパブリック ドメインの SQL データベース エンジンです。 SQLite は、世界で最も使用されているデータベース エンジンです。

SQLite データベースを管理および表示するためにダウンロードできるサードパーティ製ツールは多数あります。 以下の画像は [DB Browser for SQLite](https://sqlitebrowser.org/) のものです。 お気に入りの SQLite ツールがございましたら、そのツールの気に入っている点についてコメントしてください。

![Movie データベースを示している DB Browser for SQLite](~/tutorials/first-mvc-app-xplat/working-with-sql/_static/dbb.png)

> [!NOTE]
> このチュートリアルでは、可能な場合は Entity Framework Core の "*移行*" 機能が使用されます。 移行では、データ モデルの変更に合わせてデータベース スキーマが更新されます。 ただし、移行では EF Core プロバイダーでサポートされている種類の変更しか行うことができず、SQLite プロバイダーの機能は制限されています。 たとえば、列の追加はサポートされていますが、列の削除や変更はサポートされていません。 列を削除または変更するために移行を作成した場合、`ef migrations add` コマンドは成功しますが、`ef database update` コマンドは失敗します。 これらの制限により、このチュートリアルでは SQLite のスキーマの変更に対しては移行を使用しません。 代わりに、スキーマが変更されると、データベースが削除され、再作成されます。
>
>SQLite の制限に対する回避策としては、テーブル内の何かが変更されたときにテーブルの再構築を実行する移行コードを、手動で作成します。 テーブルの再構築には次の作業が含まれます。
>
>* 新しいテーブルの作成。
>* 古いテーブルから新しいテーブルへのデータのコピー。
>* 古いテーブルの削除。
>* 新しいテーブルの名前変更。
>
>詳細については、次のリソースを参照してください。
> * [SQLite EF Core データベース プロバイダーの制限事項](/ef/core/providers/sqlite/limitations)
> * [移行コードをカスタマイズする](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [データのシード処理](/ef/core/modeling/data-seeding)
> * [SQLite ALTER TABLE ステートメント](https://sqlite.org/lang_altertable.html)

---

## <a name="seed-the-database"></a>データベースのシード

次のコードを使用して、*Models* フォルダーに `SeedData` という名前の新しいクラスを作成します。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/SeedData.cs?name=snippet_1)]

データベースにムービーがある場合、シード初期化子が返され、ムービーは追加されません。

```csharp
if (context.Movie.Any())
{
    return;
}
```

<a name="si"></a>

### <a name="add-the-seed-initializer"></a>シード初期化子の追加

*Program.cs* の内容を次のコードに置き換えます。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie50/Program.cs)]

以前のコードでは、`Main` メソッドが次のように変更されています。

* 依存関係挿入コンテナーからデータベース コンテキスト インスタンスを取得します。
* `seedData.Initialize` メソッドを呼び出すと、データベース コンテキスト インスタンスに渡されます。
* seed メソッドが完了したら、コンテキストを破棄します。 [using ステートメント](/dotnet/csharp/language-reference/keywords/using-statement)を使用すると、コンテキストが確実に破棄されます。

`Update-Database` が実行されていなかった場合、次の例外が発生します。

> `SqlException: Cannot open database "RazorPagesMovieContext-" requested by the login. The login failed.`
> `Login failed for user 'user name'.`

### <a name="test-the-app"></a>アプリのテスト

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. データベース内のすべてのレコードを削除します。 ブラウザーの削除リンクを使用するか、[SSOX](xref:tutorials/razor-pages/new-field#ssox) から行います。

1. `Startup` クラスでメソッドを呼び出してアプリを強制的に初期化して、シード メソッドが実行されるようにします。 強制的に初期化するには、IIS Express を停止してから再起動する必要があります。 次の方法のいずれかを使用して、IIS を停止してから再起動します。

   1. 通知領域にある IIS Express システム トレイ アイコンを右クリックし、 **[終了]** または **[サイトの停止]** を選択します。

      ![IIS Express システム トレイ アイコン](../first-mvc-app/working-with-sql/_static/iisExIcon.png)

      ![コンテキスト メニュー](sql/_static/stopIIS.png)

   1. 非デバッグ モードでアプリが実行されている場合は、<kbd>F5</kbd> キーを押してデバッグ モードで実行します。
   1. デバッグ モードでアプリが実行されている場合は、デバッガーを停止して、<kbd>F5</kbd> キーを押します。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

データベース内のすべてのレコードを削除します (そのため Seed メソッドが実行されます)。 アプリを停止および起動して、データベースをシードします。

---

アプリにシードされたデータが表示されます。

![ムービー データが表示されたブラウザーで開いている Movie アプリケーション](sql/_static/5/m55.png)

## <a name="additional-resources"></a>その他のリソース

> [!div class="step-by-step"]
> [前へ:スキャフォールディングされた Razor Pages](xref:tutorials/razor-pages/page)
> [次:ページの更新](xref:tutorials/razor-pages/da1)

::: moniker-end

::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

`RazorPagesMovieContext` オブジェクトは、データベースへの接続と、データベース レコードへの `Movie` オブジェクトのマッピングのタスクを処理します。 データベース コンテキストは、*Startup.cs* の `ConfigureServices` メソッドで [依存性の注入](xref:fundamentals/dependency-injection)コンテナーに登録されます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_UseSqlite&highlight=5-6)]

---

ASP.NET Core の[構成](xref:fundamentals/configuration/index)システムでは `ConnectionString` キーが読み取られます。 ローカルで開発する場合は、構成によって *appsettings.json* ファイルから接続文字列が取得されます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

生成される接続文字列は、次のようになります。

[!code-json[](razor-pages-start/sample/RazorPagesMovie30/appsettings.json?highlight=10-12)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/appsettings_SQLite.json?highlight=10-12)]

---

アプリがテスト サーバーまたは運用サーバーにデプロイされると、環境変数を使用して接続文字列をテストまたは運用データベース サーバーに設定できます。 詳細については、[構成](xref:fundamentals/configuration/index)に関するページを参照してください。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

## <a name="sql-server-express-localdb"></a>SQL Server Express LocalDB

LocalDB は、プログラム開発を対象にした、SQL Server Express データベース エンジンの軽量版です。 LocalDB は要求時に開始され、ユーザー モードで実行されるため、複雑な構成はありません。 LocalDB データベースでは、既定で `C:\Users\<user>\` ディレクトリに `*.mdf` ファイルが作成されます。

<a name="ssox"></a>
* **[表示]** メニューの **[SQL Server オブジェクト エクスプローラー]** (SSOX) を開きます。

  ![[View] メニュー](sql/_static/ssox.png)

* `Movie` テーブルを右クリックし、 **[デザイナーの表示]** を選択します。

  ![Movie テーブルに対して開かれたコンテキスト メニュー](sql/_static/design.png)

  ![デザイナーに開かれた Movie テーブル](sql/_static/dv.png)

`ID` の横のキー アイコンに注意してください。 既定では、EF で主キーに `ID` という名前のプロパティが作成されます。

* `Movie` テーブルを右クリックし、 **[データの表示]** を選択します。

  ![開いた Movie テーブルにテーブル データが表示されています](sql/_static/vd22.png)

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

## <a name="sqlite"></a>SQLite

[SQLite](https://www.sqlite.org/) の Web サイトは次のようなものです。

> SQLite は埋め込みの全機能を備えた、自己完結型かつ高信頼性でパブリック ドメインの SQL データベース エンジンです。 SQLite は、世界で最も使用されているデータベース エンジンです。

SQLite データベースを管理および表示するためにダウンロードできるサードパーティ製ツールは多数あります。 以下の画像は [DB Browser for SQLite](https://sqlitebrowser.org/) のものです。 お気に入りの SQLite ツールがございましたら、そのツールの気に入っている点についてコメントしてください。

![Movie データベースを示している DB Browser for SQLite](~/tutorials/first-mvc-app-xplat/working-with-sql/_static/dbb.png)

> [!NOTE]
> このチュートリアルでは、可能な場合は Entity Framework Core の "*移行*" 機能が使用されます。 移行では、データ モデルの変更に合わせてデータベース スキーマが更新されます。 ただし、移行では EF Core プロバイダーでサポートされている種類の変更しか行うことができず、SQLite プロバイダーの機能は制限されています。 たとえば、列の追加はサポートされていますが、列の削除や変更はサポートされていません。 列を削除または変更するために移行を作成した場合、`ef migrations add` コマンドは成功しますが、`ef database update` コマンドは失敗します。 これらの制限により、このチュートリアルでは SQLite のスキーマの変更に対しては移行を使用しません。 代わりに、スキーマが変更されると、データベースが削除され、再作成されます。
>
>SQLite の制限に対する回避策としては、テーブル内の何かが変更されたときにテーブルの再構築を実行する移行コードを、手動で作成します。 テーブルの再構築には次の作業が含まれます。
>
>* 新しいテーブルの作成。
>* 古いテーブルから新しいテーブルへのデータのコピー。
>* 古いテーブルの削除。
>* 新しいテーブルの名前変更。
>
>詳細については、次のリソースを参照してください。
> * [SQLite EF Core データベース プロバイダーの制限事項](/ef/core/providers/sqlite/limitations)
> * [移行コードをカスタマイズする](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [データのシード処理](/ef/core/modeling/data-seeding)
> * [SQLite ALTER TABLE ステートメント](https://sqlite.org/lang_altertable.html)

---

## <a name="seed-the-database"></a>データベースのシード

次のコードを使用して、*Models* フォルダーに `SeedData` という名前の新しいクラスを作成します。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/SeedData.cs?name=snippet_1)]

データベースにムービーがある場合、シード初期化子が返され、ムービーは追加されません。

```csharp
if (context.Movie.Any())
{
    return;
}
```

<a name="si"></a>

### <a name="add-the-seed-initializer"></a>シード初期化子の追加

*Program.cs* の内容を次のコードに置き換えます。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Program.cs)]

以前のコードでは、`Main` メソッドが次のように変更されています。

* 依存関係挿入コンテナーからデータベース コンテキスト インスタンスを取得します。
* `seedData.Initialize` メソッドを呼び出すと、データベース コンテキスト インスタンスに渡されます。
* seed メソッドが完了したら、コンテキストを破棄します。 [using ステートメント](/dotnet/csharp/language-reference/keywords/using-statement)を使用すると、コンテキストが確実に破棄されます。

`Update-Database` が実行されていなかった場合、次の例外が発生します。

> `SqlException: Cannot open database "RazorPagesMovieContext-" requested by the login. The login failed.`
> `Login failed for user 'user name'.`

### <a name="test-the-app"></a>アプリのテスト

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* データベース内のすべてのレコードを削除します。 ブラウザーの削除リンクを使用するか、[SSOX](xref:tutorials/razor-pages/new-field#ssox) から行います。
* `Startup` クラスでメソッドを呼び出してアプリを強制的に初期化して、シード メソッドが実行されるようにします。 強制的に初期化するには、IIS Express を停止してから再起動する必要があります。 次の方法のいずれかを使用して、IIS を停止してから再起動します。

  * 通知領域にある IIS Express システム トレイ アイコンを右クリックし、 **[終了]** または **[サイトの停止]** をタップします。

    ![IIS Express システム トレイ アイコン](../first-mvc-app/working-with-sql/_static/iisExIcon.png)

    ![コンテキスト メニュー](sql/_static/stopIIS.png)

    * 非デバッグ モードでアプリが実行されている場合は、<kbd>F5</kbd> キーを押してデバッグ モードで実行します。
    * デバッグ モードでアプリが実行されている場合は、デバッガーを停止して、<kbd>F5</kbd> キーを押します。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

データベース内のすべてのレコードを削除します (そのため Seed メソッドが実行されます)。 アプリを停止および起動して、データベースをシードします。

---

アプリにシードされたデータが表示されます。

![ムービー データが表示された、Chrome で開かれているムービー アプリケーション](sql/_static/m55https.png)

## <a name="additional-resources"></a>その他のリソース

> [!div class="step-by-step"]
> [前へ:スキャフォールディングされた Razor Pages](xref:tutorials/razor-pages/page)
> [次:ページの更新](xref:tutorials/razor-pages/da1)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

`RazorPagesMovieContext` オブジェクトは、データベースへの接続と、データベース レコードへの `Movie` オブジェクトのマッピングのタスクを処理します。 データベース コンテキストは、*Startup.cs* の `ConfigureServices` メソッドで [依存性の注入](xref:fundamentals/dependency-injection)コンテナーに登録されます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_ConfigureServices&highlight=15-17)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_UseSqlite&highlight=11-12)]

---

`ConfigureServices` で使用されているメソッドの詳細については、以下を参照してください。

* [ASP.NET Core での `CookiePolicyOptions` 用の EU の一般データ保護規制 (GDPR) のサポート](xref:security/gdpr)
* [SetCompatibilityVersion](xref:mvc/compatibility-version)

ASP.NET Core の[構成](xref:fundamentals/configuration/index)システムでは `ConnectionString` キーが読み取られます。 ローカルで開発する場合は、構成によって *appsettings.json* ファイルから接続文字列が取得されます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

生成される接続文字列は、次のようになります。

[!code-json[](razor-pages-start/sample/RazorPagesMovie22/appsettings.json?highlight=8-10)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/appsettings_SQLite.json?highlight=8-10)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/appsettings_SQLite.json?highlight=8-10)]

---

アプリがテスト サーバーまたは運用サーバーにデプロイされると、環境変数を使用して接続文字列をテストまたは運用データベース サーバーに設定できます。 詳細については、[構成](xref:fundamentals/configuration/index)に関するページを参照してください。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

## <a name="sql-server-express-localdb"></a>SQL Server Express LocalDB

LocalDB は、プログラム開発を対象にした、SQL Server Express データベース エンジンの軽量版です。 LocalDB は要求時に開始され、ユーザー モードで実行されるため、複雑な構成はありません。 LocalDB データベースでは、既定で `C:/Users/<user/>` ディレクトリに `*.mdf` ファイルが作成されます。

<a name="ssox"></a>
* **[表示]** メニューの **[SQL Server オブジェクト エクスプローラー]** (SSOX) を開きます。

  ![[View] メニュー](sql/_static/ssox.png)

* `Movie` テーブルを右クリックし、 **[デザイナーの表示]** を選択します。

  ![Movie テーブルで開かれたコンテキスト メニュー](sql/_static/design.png)

  ![デザイナーに開かれた Movie テーブル](sql/_static/dv.png)

`ID` の横のキー アイコンに注意してください。 既定では、EF で主キーに `ID` という名前のプロパティが作成されます。

* `Movie` テーブルを右クリックし、 **[データの表示]** を選択します。

  ![開いた Movie テーブルにテーブル データが表示されています](sql/_static/vd22.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/rp/sqlite.md)]
[!INCLUDE[](~/includes/RP-mvc-shared/sqlite-warn.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/rp/sqlite.md)]
[!INCLUDE[](~/includes/RP-mvc-shared/sqlite-warn.md)]

---

## <a name="seed-the-database"></a>データベースのシード

次のコードを使用して、*Models* フォルダーに `SeedData` という名前の新しいクラスを作成します。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/SeedData.cs?name=snippet_1)]

データベースにムービーがある場合、シード初期化子が返され、ムービーは追加されません。

```csharp
if (context.Movie.Any())
{
    return;
}
```

<a name="si"></a>

### <a name="add-the-seed-initializer"></a>シード初期化子の追加

*Program.cs* の内容を次のコードに置き換えます。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Program.cs)]

以前のコードでは、`Main` メソッドが次のように変更されています。

* 依存関係挿入コンテナーからデータベース コンテキスト インスタンスを取得します。
* `seedData.Initialize` メソッドを呼び出すと、データベース コンテキスト インスタンスに渡されます。
* seed メソッドが完了したら、コンテキストを破棄します。 [using ステートメント](/dotnet/csharp/language-reference/keywords/using-statement)を使用すると、コンテキストが確実に破棄されます。

運用アプリは `Database.Migrate` を呼び出しません。 これは、`Update-Database` が実行されていないとき、前述のコードに追加され、次の例外を阻止します。

SqlException:Cannot open database "RazorPagesMovieContext-21" requested by the login. (SqlException: ログインで要求されている "RazorPagesMovieContext-21" データベースを開くことができませんでした。) The login failed.\(ログインに失敗しました。\)
Login failed for user 'user name'.\(ユーザー 'ユーザー名' はログインできませんでした。\)

### <a name="test-the-app"></a>アプリのテスト

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* データベース内のすべてのレコードを削除します。 これはブラウザーの削除リンクで行うか、[SSOX](xref:tutorials/razor-pages/new-field#ssox) から行うことができます。
* `Startup` クラスでメソッドを呼び出してアプリを強制的に初期化して、シード メソッドが実行されるようにします。 強制的に初期化するには、IIS Express を停止してから再起動する必要があります。 これは次の方法のいずれかを使用して行うことができます。

  * 通知領域にある IIS Express システム トレイ アイコンを右クリックし、 **[終了]** または **[サイトの停止]** をタップします。

    ![IIS Express システム トレイ アイコン](../first-mvc-app/working-with-sql/_static/iisExIcon.png)

    ![コンテキスト メニュー](sql/_static/stopIIS.png)

    * 非デバッグ モードでアプリが実行されている場合は、<kbd>F5</kbd> キーを押してデバッグ モードで実行します。
    * デバッグ モードでアプリが実行されている場合は、デバッガーを停止して、<kbd>F5</kbd> キーを押します。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

データベース内のすべてのレコードを削除します (そのため Seed メソッドが実行されます)。 アプリを停止および起動して、データベースをシードします。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

データベース内のすべてのレコードを削除します (そのため Seed メソッドが実行されます)。 アプリを停止および起動して、データベースをシードします。

---

アプリにシードされたデータが表示されます。

![ムービー データが表示された、Chrome で開かれているムービー アプリケーション](sql/_static/m55https.png)

次のチュートリアルでは、データの表示をクリーンアップします。

## <a name="additional-resources"></a>その他の技術情報

* [このチュートリアルの YouTube バージョン](https://youtu.be/A_5ff11sDHY)

> [!div class="step-by-step"]
> [前へ:スキャフォールディングされた Razor Pages](xref:tutorials/razor-pages/page)
> [次:ページの更新](xref:tutorials/razor-pages/da1)

::: moniker-end