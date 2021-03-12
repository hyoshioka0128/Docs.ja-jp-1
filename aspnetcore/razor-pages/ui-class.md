---
title: ASP.NET Core のクラス ライブラリの再利用可能 Razor UI
author: Rick-Anderson
description: ASP.NET Core のクラス ライブラリで部分ビューを使用して、再利用可能な Razor UI を作成する方法について説明します。
ms.author: riande
ms.date: 01/19/2021
ms.custom: mvc, seodec18
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
uid: razor-pages/ui-class
ms.openlocfilehash: 88b3bae5d52e334dea7ffcfe245f49d2816047c3
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588179"
---
# <a name="create-reusable-ui-using-the-razor-class-library-project-in-aspnet-core"></a>ASP.NET Core の Razor クラス ライブラリ プロジェクトを使用した再利用可能 UI の作成

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

Razor ビュー、ページ、コントローラー、ページ モデル、[Razor コンポーネント](xref:blazor/components/class-libraries)、[ビュー コンポーネント](xref:mvc/views/view-components)、データ モデルは、Razor クラス ライブラリ (RCL) に組み込むことが可能です。 RCL はパッケージ化し、再利用できます。 アプリケーションには RCL を含めることができます。また、それに含まれるビューやページをオーバーライドできます。 Web アプリと RCL の両方にビュー、部分ビュー、Razor ページがあるとき、Web アプリの Razor マークアップ ( *.cshtml* ファイル) が優先されます。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/ui-class/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="create-a-class-library-containing-razor-ui"></a>Razor UI を含むクラス ライブラリの作成

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Visual Studio から **[新しいプロジェクトの作成]** を選択します。
* **[Razor クラス ライブラリ]** > **[次へ]** の順に選択します。
* ライブラリに名前を付け ("RazorClassLib" など)、 **[作成]** を選択します。 生成されたビュー ライブラリとファイル名の競合を避けるため、ライブラリ名の末尾が `.Views` ではないことを確認します。
* ビューをサポートする必要がある場合は、 **[ページとビューのサポート]** を選択します。 既定では、Razor ページのみがサポートされています。 **[作成]** を選択します。

Razor クラス ライブラリ (RCL) テンプレートは Razor コンポーネント開発での既定です。 **[ページとビューのサポート]** オプションによって、ページとビューがサポートされます。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

コマンド ラインから `dotnet new razorclasslib` を実行します。 次に例を示します。

```dotnetcli
dotnet new razorclasslib -o RazorUIClassLib
```

Razor クラス ライブラリ (RCL) テンプレートは Razor コンポーネント開発での既定です。 `--support-pages-and-views` オプションを渡す (`dotnet new razorclasslib --support-pages-and-views`) と、ページとビューがサポートされます。

詳細については、「[dotnet new](/dotnet/core/tools/dotnet-new)」を参照してください。 生成されたビュー ライブラリとファイル名の競合を避けるため、ライブラリ名の末尾が `.Views` ではないことを確認します。

---

RCL に Razor ファイルを追加します。

ASP.NET Core テンプレートでは、RCL コンテンツが *Areas* フォルダーにあるものとしています。 `~/Areas/Pages` ではなく `~/Pages` でコンテンツを公開する RCL を作成する場合は、「[RCL ページのレイアウト](#rcl-pages-layout)」を参照してください。

## <a name="reference-rcl-content"></a>RCL コンテンツを参照する

RCL は次によって参照できます。

* NuGet パッケージ。 「[Creating NuGet packages](/nuget/create-packages/creating-a-package)」 (NuGet パッケージの作成)、「[dotnet add package](/dotnet/core/tools/dotnet-add-package)」、「[Create and publish a NuGet package](/nuget/quickstart/create-and-publish-a-package-using-visual-studio)」 (NuGet パッケージの作成と公開) を参照してください。
* *{ProjectName}.csproj*. 「[dotnet-add reference](/dotnet/core/tools/dotnet-add-reference)」を参照してください。

## <a name="override-views-partial-views-and-pages"></a>ビュー、部分ビュー、ページのオーバーライド

Web アプリと RCL の両方にビュー、部分ビュー、Razor ページがあるとき、Web アプリの Razor マークアップ ( *.cshtml* ファイル) が優先されます。 たとえば、*WebApp1/Areas/MyFeature/Pages/Page1.cshtml* を WebApp1 に追加すると、WebApp1 の Page1 は RCL の Page1 よりも優先されます。

サンプル ダウンロードの *WebApp1/Areas/MyFeature2* の名前を *WebApp1/Areas/MyFeature* に変更し、優先設定をテストします。

*RazorUIClassLib/Areas/MyFeature/Pages/Shared/_Message.cshtml* 部分ビューを *WebApp1/Areas/MyFeature/Pages/Shared/_Message.cshtml* にコピーします。 新しい場所を示すようにマークアップを更新します。 アプリをビルドして実行し、アプリの部分ビューが使用されていることを確認します。

### <a name="rcl-pages-layout"></a>RCL ページのレイアウト

RCL コンテンツを Web アプリの *Pages* フォルダーの一部であるかのように参照するには、次のファイル構造で RCL プロジェクトを作成します。

* *RazorUIClassLib/Pages*
* *RazorUIClassLib/Pages/Shared*

*RazorUIClassLib/Pages/Shared* に 2 つの部分ファイル、 *_Header.cshtml* と *_Footer.cshtml* が含まれているとします。 これらの `<partial>` タグを *_Layout.cshtml* ファイルに追加できます。

```cshtml
<body>
  <partial name="_Header">
  @RenderBody()
  <partial name="_Footer">
</body>
```

*_ViewStart.cshtml* ファイルを、RCL プロジェクトの *Pages* フォルダーに追加して、ホスト Web アプリの *_Layout.cshtml* ファイルを使用します。

```cshtml
@{
    Layout = "_Layout";
}
```

## <a name="create-an-rcl-with-static-assets"></a>静的アセットを含む RCL を作成する

RCL では、RCL または RCL の使用アプリで参照できる静的なコンパニオン アセットが必要になる場合があります。 ASP.NET Core では、使用アプリで利用できる静的アセットを含む RCL の作成が可能です。

コンパニオン アセットを RCL の一部として含めるには、クラス ライブラリに *wwwroot* フォルダーを作成し、必要なファイルをすべてそのフォルダーに含めます。

RCL をパックすると、*wwwroot* フォルダー内のすべてのコンパニオン アセットがパッケージに自動的に組み込まれます。

Nuget.exe バージョン `nuget pack` ではなく、`dotnet pack` コマンドを使用します。

### <a name="exclude-static-assets"></a>静的アセットを除外する

静的アセットを除外するには、目的の除外パスをプロジェクトファイル内の `$(DefaultItemExcludes)` プロパティ グループに追加します。 各エントリは、セミコロン (`;`) で区切ります。

次の例では、*wwwroot* フォルダー内の *lib.css* スタイルシートは、静的アセットとは見なされず、公開された RCL には組み込まれていません。

```xml
<PropertyGroup>
  <DefaultItemExcludes>$(DefaultItemExcludes);wwwroot\lib.css</DefaultItemExcludes>
</PropertyGroup>
```

### <a name="typescript-integration"></a>Typescript の統合

TypeScript ファイルを RCL に含めるには、次の操作を行います。

1. TypeScript ファイル ( *.ts*) を *wwwroot* フォルダーの外側に配置します。 たとえば、ファイルを *Client* フォルダーに入れます。

1. *wwwroot* フォルダーの TypeScript ビルド出力を構成します。 プロジェクト ファイルの `PropertyGroup` の内側に `TypescriptOutDir` プロパティを設定します。

   ```xml
   <TypescriptOutDir>wwwroot</TypescriptOutDir>
   ```

1. プロジェクト ファイルの `PropertyGroup` の内側に次のターゲットを追加して、TypeScript ターゲットを `ResolveCurrentProjectStaticWebAssets` ターゲットの依存関係として含めます。

   ```xml
   <ResolveCurrentProjectStaticWebAssetsInputsDependsOn>
     CompileTypeScript;
     $(ResolveCurrentProjectStaticWebAssetsInputs)
   </ResolveCurrentProjectStaticWebAssetsInputsDependsOn>
   ```

### <a name="consume-content-from-a-referenced-rcl"></a>参照されている RCL からコンテンツを使用する

RCL の *wwwroot* フォルダーに含まれるファイルは、`_content/{LIBRARY NAME}/` プレフィックスに基づいて RCL または使用アプリのいずれかに公開されます。 たとえば、 *Razor.Class.Lib* という名前のライブラリを使用すると、`_content/Razor.Class.Lib/` にある静的コンテンツへのパスが生成されます。 NuGet パッケージを生成するときに、アセンブリ名がパッケージ ID と異なる場合は、`{LIBRARY NAME}` のパッケージ ID を使用します。

使用アプリは、ライブラリによって提供される静的アセットを `<script>`、`<style>`、`<img>`、およびその他の HTML タグ付きで参照します。 使用アプリの `Startup.Configure` で [静的ファイルのサポート](xref:fundamentals/static-files)が有効になっている必要があります。

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    ...

    app.UseStaticFiles();

    ...
}
```

使用アプリをビルド出力から実行 (`dotnet run`) すると、開発環境で、静的な Web アセットが既定で有効になります。 ビルド出力から実行するときに他の環境のアセットをサポートするには、*Program.cs* のホスト ビルダーで `UseStaticWebAssets` を呼び出します。

```csharp
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Hosting;

public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStaticWebAssets();
                webBuilder.UseStartup<Startup>();
            });
}
```

発行された出力からアプリを実行する (`dotnet publish`) 場合、`UseStaticWebAssets` の呼び出しは必要ありません。

### <a name="multi-project-development-flow"></a>複数プロジェクトの開発フロー

使用アプリの実行時は、次のようになります。

* RCL のアセットは元のフォルダー内に保持されます。 これらのアセットは、使用アプリに移行されません。
* RCL の *wwwroot* フォルダー内の変更は、使用アプリをリビルドしなくても、RCL がリビルドされた後で、使用アプリに反映されます。

RCL がビルドされると、静的な Web アセットの場所を記述するマニフェストが生成されます。 使用アプリは、実行時にそのマニフェストを読み取って、参照されているプロジェクトおよびパッケージのアセットを使用します。 RCL に新しいアセットが追加された場合は、使用アプリがその新しいアセットにアクセスする前に、RCL をリビルドしてそのマニフェストを更新する必要があります。

### <a name="publish"></a>公開

アプリが公開されると、参照されているすべてのプロジェクトおよびパッケージのコンパニオン アセットが、`_content/{LIBRARY NAME}/` の下の公開済みアプリの *wwwroot* フォルダーにコピーされます。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Razor ビュー、ページ、コントローラー、ページ モデル、[Razor コンポーネント](xref:blazor/components/class-libraries)、[ビュー コンポーネント](xref:mvc/views/view-components)、データ モデルは、Razor クラス ライブラリ (RCL) に組み込むことが可能です。 RCL はパッケージ化し、再利用できます。 アプリケーションには RCL を含めることができます。また、それに含まれるビューやページをオーバーライドできます。 Web アプリと RCL の両方にビュー、部分ビュー、Razor ページがあるとき、Web アプリの Razor マークアップ ( *.cshtml* ファイル) が優先されます。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/ui-class/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="create-a-class-library-containing-razor-ui"></a>Razor UI を含むクラス ライブラリの作成

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Visual Studio の **[ファイル]** メニューから、 **[新規作成]** > **[プロジェクト]** の順に選択します。
* **[ASP.NET Core Web アプリケーション]** を選択します。
* ライブラリに名前を付け ("RazorClassLib" など)、 **[OK]** を選択します。 生成されたビュー ライブラリとファイル名の競合を避けるため、ライブラリ名の末尾が `.Views` ではないことを確認します。
* **ASP.NET Core 2.1** 以降が選択されていることを確認します。
* **[Razor クラス ライブラリ]** > **[OK]** の順に選択します。

RCL には次のプロジェクト ファイルがあります。

[!code-xml[](ui-class/samples/cli/RazorUIClassLib/RazorUIClassLib.csproj)]

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

コマンド ラインから `dotnet new razorclasslib` を実行します。 次に例を示します。

```dotnetcli
dotnet new razorclasslib -o RazorUIClassLib
```

詳細については、「[dotnet new](/dotnet/core/tools/dotnet-new)」を参照してください。 生成されたビュー ライブラリとファイル名の競合を避けるため、ライブラリ名の末尾が `.Views` ではないことを確認します。

---

RCL に Razor ファイルを追加します。

ASP.NET Core テンプレートでは、RCL コンテンツが *Areas* フォルダーにあるものとしています。 `~/Areas/Pages` ではなく `~/Pages` でコンテンツを公開する RCL を作成する場合は、「[RCL ページのレイアウト](#rcl-pages-layout)」を参照してください。

## <a name="reference-rcl-content"></a>RCL コンテンツを参照する

RCL は次によって参照できます。

* NuGet パッケージ。 「[Creating NuGet packages](/nuget/create-packages/creating-a-package)」 (NuGet パッケージの作成)、「[dotnet add package](/dotnet/core/tools/dotnet-add-package)」、「[Create and publish a NuGet package](/nuget/quickstart/create-and-publish-a-package-using-visual-studio)」 (NuGet パッケージの作成と公開) を参照してください。
* *{ProjectName}.csproj*. 「[dotnet-add reference](/dotnet/core/tools/dotnet-add-reference)」を参照してください。

## <a name="walkthrough-create-an-rcl-project-and-use-from-a-razor-pages-project"></a>チュートリアル: RCL プロジェクトを作成し、Razor Pages プロジェクトから使用する

作成しなくても、[完全なプロジェクト](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/ui-class/samples)をダウンロードしてテストできます。 サンプル ダウンロードには、プロジェクトのテストを簡単にする追加のコードやリンクが含まれています。 GitHub の問題に関するフィードバックは[こちら](https://github.com/dotnet/AspNetCore.Docs/issues/6098)で扱っています。ダウンロード サンプルと段階的指示の違いについてコメントを投稿できます。

### <a name="test-the-download-app"></a>ダウンロード アプリをテストする

完全なアプリをダウンロードしておらず、チュートリアル プロジェクトを作成する場合、[次のセクション](#create-an-rcl)に進んでください。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Visual Studio で *.sln* ファイルを開きます。 アプリを実行します。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

*cli* ディレクトリのコマンド プロンプトから、RCL と Web アプリをビルドします。

```dotnetcli
dotnet build
```

*WebApp1* ディレクトリに移動し、アプリを実行します。

```dotnetcli
dotnet run
```

---

[テスト WebApp1](#test-webapp1) の指示に従ってください。

## <a name="create-an-rcl"></a>RCL を作成する

このセクションでは、RCL が作成されます。 Razor ファイルが RCL に追加されます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

RCL プロジェクトの作成:

* Visual Studio の **[ファイル]** メニューから、 **[新規作成]** > **[プロジェクト]** の順に選択します。
* **[ASP.NET Core Web アプリケーション]** を選択します。
* アプリに **RazorUIClassLib** という名前を付け、 **[OK]** を選択します。
* **ASP.NET Core 2.1** 以降が選択されていることを確認します。
* **[Razor クラス ライブラリ]** > **[OK]** の順に選択します。
* *RazorUIClassLib/Areas/MyFeature/Pages/Shared/_Message.cshtml* という名前が付いた Razor 部分ビュー ファイルを追加します。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

コマンド ラインから次を実行します。

```dotnetcli
dotnet new razorclasslib -o RazorUIClassLib
dotnet new page -n _Message -np -o RazorUIClassLib/Areas/MyFeature/Pages/Shared
dotnet new viewstart -o RazorUIClassLib/Areas/MyFeature/Pages
```

上のコマンドでは以下の操作が行われます。

* `RazorUIClassLib` RCL が作成されます。
* Razor _Message ページが作成され、RCL に追加されます。 `-np` パラメーターによって、`PageModel` なしでページが作成されます。
* [_ViewStart.cshtml](xref:mvc/views/layout#running-code-before-each-view) ファイルが作成され、RCL に追加されます。

(次のセクションで追加される) RazorPages プロジェクトのレイアウトを使用するには、 *_ViewStart.cshtml* ファイルが必要です。

---

### <a name="add-razor-files-and-folders-to-the-project"></a>Razor ファイルとフォルダーをプロジェクトに追加する

* *RazorUIClassLib/Areas/MyFeature/Pages/Shared/_Message.cshtml* のマークアップを次のコードに変更します。

  [!code-cshtml[](ui-class/samples/cli/RazorUIClassLib/Areas/MyFeature/Pages/Shared/_Message.cshtml)]

* *RazorUIClassLib/Areas/MyFeature/Pages/Page1.cshtml* のマークアップを次のコードに変更します。

  [!code-cshtml[](ui-class/samples/cli/RazorUIClassLib/Areas/MyFeature/Pages/Page1.cshtml)]

  `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers` は部分ビュー (`<partial name="_Message" />`) を使用するために必要です。 `@addTagHelper` ディレクティブを含める代わりに、 *_ViewImports.cshtml* ファイルを追加できます。 次に例を示します。

  ```dotnetcli
  dotnet new viewimports -o RazorUIClassLib/Areas/MyFeature/Pages
  ```

  *_ViewImports.cshtml* について詳しくは、「[共有ディレクティブのインポート](xref:mvc/views/layout#importing-shared-directives)」を参照してください。

* クラス ライブラリをビルドし、コンパイラ エラーがないことを確認します。

  ```dotnetcli
  dotnet build RazorUIClassLib
  ```

ビルド出力には、 *RazorUIClassLib.dll* と *RazorUIClassLib.Views.dll* が含まれています。 *RazorUIClassLib.Views.dll* には、コンパイル済みの Razor コンテンツが含まれています。

### <a name="use-the-razor-ui-library-from-a-razor-pages-project"></a>Razor ページ プロジェクトから Razor UI ライブラリを使用します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Razor ページ Web アプリの作成:

* **ソリューション エクスプローラー** で、ソリューションを右クリックし、 **[追加]** > **[新しいプロジェクト]** の順に選択します。
* **[ASP.NET Core Web アプリケーション]** を選択します。
* アプリに **WebApp1** という名前を付けます。
* **ASP.NET Core 2.1** 以降が選択されていることを確認します。
* **[Web アプリケーション]** > **[OK]** の順に選択します。

* **ソリューション エクスプローラー** で、**WebApp1** を右クリックし、 **[スタートアップ プロジェクトに設定]** を選択します。
* **ソリューション エクスプローラー** で、**WebApp1** を右クリックし、 **[ビルド依存関係]** > **[プロジェクトの依存関係]** の順に選択します。
* **WebApp1** の依存関係として **RazorUIClassLib** を選択します。
* **ソリューション エクスプローラー** で、**WebApp1** を右クリックし、 **[追加]** > **[参照]** の順に選択します。
* **[参照マネージャー]** ダイアログで、 **[RazorUIClassLib]** をオンにして **[OK]** を選択します。

アプリを実行します。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

Razor Pages Web アプリと、Razor Pages アプリと RCL を含むソリューション ファイルを作成します。

```dotnetcli
dotnet new webapp -o WebApp1
dotnet new sln
dotnet sln add WebApp1
dotnet sln add RazorUIClassLib
dotnet add WebApp1 reference RazorUIClassLib
```

Web アプリをビルドし、実行します。

```dotnetcli
cd WebApp1
dotnet run
```

---

### <a name="test-webapp1"></a>テスト WebApp1

`/MyFeature/Page1` を参照して、Razor UI クラス ライブラリが使用されていることを確認します。

## <a name="override-views-partial-views-and-pages"></a>ビュー、部分ビュー、ページのオーバーライド

Web アプリと RCL の両方にビュー、部分ビュー、Razor ページがあるとき、Web アプリの Razor マークアップ ( *.cshtml* ファイル) が優先されます。 たとえば、*WebApp1/Areas/MyFeature/Pages/Page1.cshtml* を WebApp1 に追加すると、WebApp1 の Page1 は RCL の Page1 よりも優先されます。

サンプル ダウンロードの *WebApp1/Areas/MyFeature2* の名前を *WebApp1/Areas/MyFeature* に変更し、優先設定をテストします。

*RazorUIClassLib/Areas/MyFeature/Pages/Shared/_Message.cshtml* 部分ビューを *WebApp1/Areas/MyFeature/Pages/Shared/_Message.cshtml* にコピーします。 新しい場所を示すようにマークアップを更新します。 アプリをビルドして実行し、アプリの部分ビューが使用されていることを確認します。

### <a name="rcl-pages-layout"></a>RCL ページのレイアウト

RCL コンテンツを Web アプリの *Pages* フォルダーの一部であるかのように参照するには、次のファイル構造で RCL プロジェクトを作成します。

* *RazorUIClassLib/Pages*
* *RazorUIClassLib/Pages/Shared*

*RazorUIClassLib/Pages/Shared* に 2 つの部分ファイル、 *_Header.cshtml* と *_Footer.cshtml* が含まれているとします。 これらの `<partial>` タグを *_Layout.cshtml* ファイルに追加できます。

```cshtml
<body>
  <partial name="_Header">
  @RenderBody()
  <partial name="_Footer">
</body>
```

::: moniker-end

## <a name="additional-resources"></a>その他の技術情報

::: moniker range=">= aspnetcore-5.0"

* <xref:blazor/components/class-libraries>
* <xref:blazor/components/css-isolation#razor-class-library-rcl-support>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* <xref:blazor/components/class-libraries>

::: moniker-end
