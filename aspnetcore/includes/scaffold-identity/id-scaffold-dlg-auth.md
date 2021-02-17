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
ms.openlocfilehash: 1161f7731898221e51a4c7f9f246269b83c6ae80
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552731"
---
::: moniker range=">= aspnetcore-3.0"

Scaffolder を実行し Identity ます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* **ソリューションエクスプローラー** で、プロジェクトを右クリックし、[ **Add** > **New スキャフォールディング Item**] > ます。
* [**スキャフォールディングの追加**] ダイアログボックスの左ペインで、[追加] を選択し **Identity** > ます。
* [**追加 Identity** ] ダイアログで、必要なオプションを選択します。
  * レイアウトファイルが正しくないマークアップで上書きされないように、既存のレイアウトページを選択します。 既存の *\_ Layout* ファイルが選択されている場合、そのファイルは上書きされ **ません**。 次に例を示します。
    * `~/Pages/Shared/_Layout.cshtml`Razor Blazor Server 既存の Razor ページインフラストラクチャを使用するページまたはプロジェクトの場合
    * `~/Views/Shared/_Layout.cshtml` mvc プロジェクトまたは Blazor Server 既存の mvc インフラストラクチャを使用するプロジェクトの場合
* 既存のデータコンテキストを使用するには、オーバーライドするファイルを少なくとも1つ選択します。 データコンテキストを追加するには、少なくとも1つのファイルを選択する必要があります。
  * データコンテキストクラスを選択します。
  * **[追加]** を選択します。
* 新しいユーザーコンテキストを作成し、場合によってはのカスタムユーザークラスを作成するには、次のようにし Identity ます。
  * **+** 新しい **データコンテキストクラス** を作成するには、このボタンをクリックします。 既定値をそのまま使用するか、クラス (など) を指定し `MyApplication.Data.ApplicationDbContext` ます。
  * **[追加]** を選択します。

注: 新しいユーザーコンテキストを作成している場合は、上書きするファイルを選択する必要はありません。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

ASP.NET Core scaffolder をまだインストールしていない場合は、ここでインストールします。

```dotnetcli
dotnet tool install -g dotnet-aspnet-codegenerator
```

必要な NuGet パッケージ参照をプロジェクトファイル (*.csproj*) に追加します。 プロジェクトディレクトリで次のコマンドを実行します。

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.AspNetCore.Identity.UI
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

次のコマンドを実行して、scaffolder オプションの一覧を表示し Identity ます。

```dotnetcli
dotnet aspnet-codegenerator identity -h
```

[!INCLUDE[](~/includes/scaffoldTFM.md)]

プロジェクトフォルダーで、 Identity 必要なオプションを指定して scaffolder を実行します。 たとえば、既定の UI とファイルの最小数で id を設定するには、次のコマンドを実行します。 DB コンテキストの正しい完全修飾名を使用します。

```dotnetcli
dotnet aspnet-codegenerator identity -dc MyApplication.Data.ApplicationDbContext --files "Account.Register;Account.Login"
```

PowerShell では、コマンドの区切り記号としてセミコロンを使用します。 PowerShell を使用する場合は、ファイルの一覧でセミコロンをエスケープするか、ファイルの一覧を二重引用符で囲みます。 次に例を示します。

```dotnetcli
dotnet aspnet-codegenerator identity -dc MyApplication.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.Logout"
```

Identityフラグやフラグを指定せずに scaffolder を実行すると `--files` `--useDefaultUI` 、使用可能なすべての Identity UI ページがプロジェクト内に作成されます。

---

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Scaffolder を実行し Identity ます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* **ソリューションエクスプローラー** で、プロジェクトを右クリックし、[ **Add** > **New スキャフォールディング Item**] > ます。
* [**スキャフォールディングの追加**] ダイアログボックスの左ペインで、[追加] を選択し **Identity** > ます。
* [**追加 Identity** ] ダイアログで、必要なオプションを選択します。
  * 既存のレイアウトページを選択するか、レイアウトファイルが正しくないマークアップで上書きされます。 既存の *\_ Layout* ファイルが選択されている場合、そのファイルは上書きされ **ません**。 次に例を示します。
    * `~/Pages/Shared/_Layout.cshtml`ページの場合 Razor
    * `~/Views/Shared/_Layout.cshtml` MVC プロジェクトの場合
* 既存のデータコンテキストを使用するには、オーバーライドするファイルを少なくとも1つ選択します。 データコンテキストを追加するには、少なくとも1つのファイルを選択する必要があります。
  * データコンテキストクラスを選択します。
  * **[追加]** を選択します。
* 新しいユーザーコンテキストを作成し、場合によってはのカスタムユーザークラスを作成するには、次のようにし Identity ます。
  * **+** 新しい **データコンテキストクラス** を作成するには、このボタンをクリックします。 既定値をそのまま使用するか、クラス (など) を指定し `MyApplication.Data.ApplicationDbContext` ます。
  * **[追加]** を選択します。

注: 新しいユーザーコンテキストを作成している場合は、上書きするファイルを選択する必要はありません。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

ASP.NET Core scaffolder をまだインストールしていない場合は、ここでインストールします。

```dotnetcli
dotnet tool install -g dotnet-aspnet-codegenerator
```

[VisualStudio](https://www.nuget.org/packages/Microsoft.VisualStudio.Web.CodeGeneration.Design/)にパッケージ参照を追加し、プロジェクトファイル (*.csproj*) に追加してください。 プロジェクトディレクトリで次のコマンドを実行します。

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet restore
```

次のコマンドを実行して、scaffolder オプションの一覧を表示し Identity ます。

```dotnetcli
dotnet aspnet-codegenerator identity -h
```

プロジェクトフォルダーで、 Identity 必要なオプションを指定して scaffolder を実行します。 たとえば、既定の UI とファイルの最小数で id を設定するには、次のコマンドを実行します。 DB コンテキストの正しい完全修飾名を使用します。

```dotnetcli
dotnet aspnet-codegenerator identity -dc MyApplication.Data.ApplicationDbContext --files "Account.Register;Account.Login"
```

PowerShell では、コマンドの区切り記号としてセミコロンを使用します。 PowerShell を使用する場合は、ファイルの一覧でセミコロンをエスケープするか、ファイルの一覧を二重引用符で囲みます。 次に例を示します。

```dotnetcli
dotnet aspnet-codegenerator identity -dc MyApplication.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.Logout"
```

Identityフラグやフラグを指定せずに scaffolder を実行すると `--files` `--useDefaultUI` 、使用可能なすべての Identity UI ページがプロジェクト内に作成されます。

---

::: moniker-end
