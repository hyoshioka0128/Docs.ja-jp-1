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
ms.openlocfilehash: ed6de823b8b860c7d27e932e9ece40d8b43b0893
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552970"
---
Scaffolder を実行し Identity ます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* **ソリューションエクスプローラー** で、プロジェクトを右クリックし、[ **Add**  >  **New スキャフォールディング Item**] > ます。
* [ **Add New スキャフォールディング Item** ] ダイアログボックスの左ペインで、[追加] を選択し **Identity**  >  ます。
* [**追加 Identity** ] ダイアログで、必要なオプションを選択します。
  * 既存のレイアウトページを選択するか、正しくないマークアップでレイアウトファイルが上書きされます:
    * `~/Pages/Shared/_Layout.cshtml`ページの場合 Razor
    * `~/Views/Shared/_Layout.cshtml` MVC プロジェクトの場合
    * Blazor Server テンプレート () から作成されたアプリ Blazor Server `blazorserver` は、 Razor 既定ではページまたは MVC 用に構成されていません。 レイアウトページのエントリは空白のままにします。
  * **+** 新しい **データコンテキストクラス** を作成するには、このボタンをクリックします。 既定値をそのまま使用するか、クラス (など) を指定し `MyApplication.Data.ApplicationDbContext` ます。
* **[追加]** を選択します。

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

プロジェクトフォルダーで、 Identity 必要なオプションを指定して scaffolder を実行します。 たとえば、既定の UI とファイルの最小数で id を設定するには、次のコマンドを実行します。

```dotnetcli
dotnet aspnet-codegenerator identity --useDefaultUI
```

---
