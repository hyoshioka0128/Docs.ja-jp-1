---
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
ms.openlocfilehash: 44cef0c1491cc609dd9b821910014ec88ecaad81
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552490"
---
::: moniker range=">= aspnetcore-3.0"

<a name="dc"></a>

*Data* フォルダーを作成します。

次の `MvcMovieContext` クラスを *Data* フォルダーに追加します。  

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/zDocOnly/MvcMovieContext.cs?name=snippet)]

上記のコードによって、エンティティ セットの `DbSet` プロパティが作成されます。 Entity Framework の用語では、エンティティ セットは通常はデータベース テーブルに対応し、エンティティはテーブルの行に対応します。

<a name="cs"></a>

### <a name="add-a-database-connection-string"></a>データベース接続文字列の追加

接続文字列を *appsettings.json* ファイルに追加します。

[!code-json[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/appsettings_SQLite.json?highlight=10-12)]

### <a name="add-nuget-packages-and-ef-tools"></a>NuGet パッケージと EF ツールを追加します。

[!INCLUDE[](~/includes/add-EF-NuGet-SQLite-CLI.md)]

<a name="reg"></a>

### <a name="register-the-database-context"></a>データベース コンテキストの登録

*Startup.cs* の先頭に次の `using` ステートメントを追加します。

```csharp
using MvcMovie.Data;
using Microsoft.EntityFrameworkCore;
```

`Startup.ConfigureServices` で[依存性の挿入](xref:fundamentals/dependency-injection)コンテナーを使用し、データベース コンテキストを登録します。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Startup.cs?name=snippet_UseSqlite&highlight=6-7)]

コンパイラ エラーのチェックとしてプロジェクトをビルドします。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

次の `MvcMovieContext` クラスを *Models* フォルダーに追加します。  

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Data/MvcMovieContext.cs)]

上記のコードによって、エンティティ セットの `DbSet` プロパティが作成されます。 Entity Framework の用語では、エンティティ セットは通常はデータベース テーブルに対応し、エンティティはテーブルの行に対応します。

<a name="cs"></a>

### <a name="add-a-database-connection-string"></a>データベース接続文字列の追加

接続文字列を *appsettings.json* ファイルに追加します。

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie/appsettings_SQLite.json?highlight=8-10)]

### <a name="add-required-nuget-packages"></a>必要な NuGet パッケージの追加

次の .NET Core CLI コマンドを実行し、SQLite と CodeGeneration.Design をプロジェクトに追加します。

```dotnetcli
dotnet add package Microsoft.EntityFrameworkCore.SQLite
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
```

スキャフォールディングには `Microsoft.VisualStudio.Web.CodeGeneration.Design` パッケージが必要です。

<a name="reg"></a>

### <a name="register-the-database-context"></a>データベース コンテキストの登録

*Startup.cs* の先頭に次の `using` ステートメントを追加します。

```csharp
using MvcMovie.Models;
using Microsoft.EntityFrameworkCore;
```

`Startup.ConfigureServices` で[依存性の挿入](xref:fundamentals/dependency-injection)コンテナーを使用し、データベース コンテキストを登録します。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Startup.cs?name=snippet_UseSqlite&highlight=11-12)]

エラー チェックとしてプロジェクトをビルドします。
::: moniker-end