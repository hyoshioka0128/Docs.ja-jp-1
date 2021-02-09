---
title: ASP.NET Core のグローバリゼーションおよびローカリゼーション
author: rick-anderson
description: ASP.NET Core がコンテンツをさまざまな言語と文化にローカライズするために提供するサービスとミドルウェアについて説明します。
ms.author: riande
ms.date: 11/30/2019
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
uid: fundamentals/localization
ms.openlocfilehash: 67f245b7f4e4aa97b30c5318c73732617aea44c7
ms.sourcegitcommit: 7e394a8527c9818caebb940f692ae4fcf2f1b277
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/31/2021
ms.locfileid: "99217571"
---
# <a name="globalization-and-localization-in-aspnet-core"></a>ASP.NET Core のグローバリゼーションおよびローカリゼーション

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)、[Damien Bowden](https://twitter.com/damien_bod)、[Bart Calixto](https://twitter.com/bartmax)、[Nadeem Afana](https://afana.me/)、[Hisham Bin Ateya](https://twitter.com/hishambinateya)

多言語の Web サイトにすると、サイトはより幅広い対象ユーザーにリーチできます。 ASP.NET Core は、さまざまな言語と文化にローカライズするためのサービスとミドルウェアを提供します。

国際化には、[グローバリゼーション](/dotnet/api/system.globalization)と[ローカリゼーション](/dotnet/standard/globalization-localization/localization)が含まれます。 グローバリゼーションとは異なるカルチャをサポートするアプリを設計するプロセスです。 グローバリゼーションによって、特定の地域に関連する定義済みの言語セットの入出力と表示のサポートが追加されます。

ローカリゼーションとは、ローカライズのために既に処理されているグローバル化されたアプリを特定のカルチャ/ロケールに適合させるプロセスです。 詳細については、本ドキュメントの末尾にある「**グローバリゼーションとローカリゼーションの用語**」を参照してください。

アプリのローカリゼーションには、以下のものが関与します。

1. アプリのコンテンツをローカライズできるようにする
1. サポートする言語およびカルチャのローカライズされたリソースを提供する
1. 要求ごとに言語/カルチャを選択するための戦略を実装する

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/localization/sample/3.x/Localization)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="make-the-apps-content-localizable"></a>アプリのコンテンツをローカライズできるようにする

<xref:Microsoft.Extensions.Localization.IStringLocalizer> と <xref:Microsoft.Extensions.Localization.IStringLocalizer%601> は、ローカライズされたアプリの開発時に生産性が向上するように設計されています。 `IStringLocalizer` では、<xref:System.Resources.ResourceManager> と <xref:System.Resources.ResourceReader> を使用して、実行時にカルチャ固有のリソースを提供します。 インターフェイスには、ローカライズされた文字列を返すためのインデクサーと `IEnumerable` があります。 `IStringLocalizer` では、既定の言語文字列をリソース ファイルに格納する必要はありません。 ローカリゼーションの対象となるアプリを開発することができ、開発の早い段階でリソース ファイルを作成する必要はありません。 次のコードは、ローカリゼーションのために文字列 "About Title" をラップする方法を示しています。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/AboutController.cs)]

前のコードの `IStringLocalizer<T>` の実装は、[依存関係の挿入](dependency-injection.md)で説明したものです。 "About Title" のローカライズされた値が見つからない場合、インデクサーのキー、つまり文字列 "About Title" が返されます。 アプリで、既定の言語のリテラル文字列をそのままにし、ローカライザーにそれらをラップすることができるので、アプリの開発に専念することができます。 既定の言語でアプリを開発し、既定のリソース ファイルを最初に作成せずに、ローカリゼーション手順用に準備します。 または、従来のアプローチを使用し、既定の言語文字列を取得するキーを提供できます。 既定の言語の *.resx* ファイルを使用せずに、文字列リテラルをラップするだけの新しいワークフローは、多くの開発者にとって、アプリをローカライズする際のオーバーヘッドの削減になります。 他の開発者は、長い文字列リテラルの操作が容易で、ローカライズされた文字列を更新しやすい従来のワークフローを好みます。

HTML を格納しているリソースには、`IHtmlLocalizer<T>` の実装を使用します。 `IHtmlLocalizer` HTML は、リソース文字列でフォーマットされた引数をエンコードしますが、リソース文字列自体は HTML エンコードしません。 下の強調表示されたサンプルでは、`name` パラメーターの値のみが HTML エンコードされます。

[!code-csharp[](~/fundamentals/localization/sample/3.x/Localization/Controllers/BookController.cs?highlight=3,5,20&start=1&end=24)]

> [!NOTE]
> 一般に、HTML ではなく、テキストのみをローカライズします。

最下位のレベルでは、[依存関係の挿入](dependency-injection.md)から `IStringLocalizerFactory` を取得できます。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/TestController.cs?start=9&end=26&highlight=7-13)]

上記のコードは、2 つの各ファクトリ作成メソッドを示しています。

ローカライズされた文字列は、コントローラーまたは領域で仕切るか、1 つのコンテナーにすることができます。 サンプル アプリでは、共有されるリソースのために `SharedResource` というダミー クラスを使用しています。

[!code-csharp[](localization/sample/3.x/Localization/SharedResource.cs)]

一部の開発者は、`Startup` クラスを使用してグローバル文字列または共有文字列を格納します。 下のサンプルでは、`InfoController` と `SharedResource` のローカライザーが使用されています。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/InfoController.cs?range=9-26)]

## <a name="view-localization"></a>ビューのローカリゼーション

`IViewLocalizer` サービスは、[ビュー](xref:mvc/views/overview)のローカライズされた文字列を提供します。 `ViewLocalizer` クラスは、このインターフェイスを実装し、ビューのファイル パスからリソースの場所を見つけます。 次のコードは、`IViewLocalizer` の既定の実装の使用方法を示しています。

[!code-cshtml[](localization/sample/3.x/Localization/Views/Home/About.cshtml)]

`IViewLocalizer` の既定の実装は、ビューのファイル名に基づいてリソース ファイルを見つけます。 グローバルな共有リソース ファイルを使用するオプションはありません。 `ViewLocalizer` は、`IHtmlLocalizer` を使用してローカライザーを実装するので、Razor は、ローカライズされた文字列を HTML エンコードしません。 リソース文字列をパラメーター化することができます。`IViewLocalizer` は、パラメーターを HTML エンコードしますが、リソース文字列は HTML エンコードしません。 次の Razor マークアップがあるとします。

```cshtml
@Localizer["<i>Hello</i> <b>{0}!</b>", UserManager.GetUserName(User)]
```

フランス語のリソース ファイルには次の値が含まれます。

| Key | [値] |
| --- | ----- |
| `<i>Hello</i> <b>{0}!</b>` | `<i>Bonjour</i> <b>{0} !</b>` |

描画されたビューには、リソース ファイルからの HTML マークアップが含まれます。

> [!NOTE]
> 一般に、HTML ではなく、テキストのみをローカライズします。

ビュー内の共有リソース ファイルを使用するには、`IHtmlLocalizer<T>` を挿入します。

[!code-cshtml[](~/fundamentals/localization/sample/3.x/Localization/Views/Test/About.cshtml?highlight=5,12)]

## <a name="dataannotations-localization"></a>DataAnnotations のローカリゼーション

DataAnnotations エラー メッセージは `IStringLocalizer<T>` を使用してローカライズします。 オプション `ResourcesPath = "Resources"` を使用して、`RegisterViewModel` のエラー メッセージを次のいずれかのパスに格納できます。

* *Resources/ViewModels.Account.RegisterViewModel.fr.resx*
* *Resources/ViewModels/Account/RegisterViewModel.fr.resx*

[!code-csharp[](localization/sample/3.x/Localization/ViewModels/Account/RegisterViewModel.cs?start=9&end=26)]

ASP.NET Core MVC 1.1.0 以上では、非検証属性がローカライズされます。 ASP.NET Core MVC 1.0 は、非検証属性のローカライズされた文字列を検索 **しません**。

<a name="one-resource-string-multiple-classes"></a>

### <a name="using-one-resource-string-for-multiple-classes"></a>複数のクラスに 1 つのリソース文字列を使用する

次のコードは、複数のクラスに検証属性の 1 つのリソース文字列を使用する方法を示しています。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddDataAnnotationsLocalization(options => {
            options.DataAnnotationLocalizerProvider = (type, factory) =>
                factory.Create(typeof(SharedResource));
        });
}
```

上記のコードでは、`SharedResource` は、resx に対応するクラスであり、ここに検証メッセージが格納されます。 この方法では、DataAnnotations は、各クラスのリソースではなく、`SharedResource` のみを使用します。

## <a name="provide-localized-resources-for-the-languages-and-cultures-you-support"></a>サポートする言語およびカルチャのローカライズされたリソースを提供する

### <a name="supportedcultures-and-supporteduicultures"></a>SupportedCultures と SupportedUICultures

ASP.NET Core では、<xref:Microsoft.AspNetCore.Builder.RequestLocalizationOptions.SupportedCultures> と <xref:Microsoft.AspNetCore.Builder.RequestLocalizationOptions.SupportedUICultures> という 2 つのカルチャ値を指定できます。 日付、数値、および通貨の書式設定など、カルチャに依存する関数の結果は、`SupportedCultures` の <xref:System.Globalization.CultureInfo> オブジェクトによって決まります。 テキストの並べ替え順序、大文字と小文字の表記規則、文字列比較も `SupportedCultures` によって決まります。 サーバーによるカルチャの取得方法の詳細については、<xref:System.Globalization.CultureInfo.CurrentCulture?displayProperty=nameWithType> と <xref:System.Globalization.CultureInfo.CurrentUICulture?displayProperty=nameWithType> に関する記事を参照してください。 `SupportedUICultures` は、 `.resx` ファイルからのどの翻訳文字列が <xref:System.Resources.ResourceManager> によって検索されるかを決定します。 `ResourceManager` では、`CurrentUICulture` によって決定されるカルチャ固有の文字列が検索されます。 .NET のすべてのスレッドに `CurrentCulture` オブジェクトと `CurrentUICulture`オブジェクトがあります。 フレームワークでは、カルチャに依存する関数を表示するときに、これらの値が検査されます。 現在のスレッドのカルチャが `en-US` (英語、米国) に設定されている場合は、`DateTime.Now.ToLongDateString()` によって `Thursday, February 18, 2016` が表示されます。ただし、`CurrentCulture` が `es-ES` (スペイン語、スペイン) に設定されている場合は、出力は `jueves, 18 de febrero de 2016` になります。

## <a name="resource-files"></a>リソース ファイル

リソース ファイルは、ローカライズ可能な文字列をコードから分離するために役立つメカニズムです。 既定以外の言語の翻訳された文字列は、 *.resx* リソース ファイルで分離されています。 たとえば、翻訳された文字列を含む *Welcome.es.resx* という名前のスペイン語のリソース ファイルを作成したい場合があります。 "es" は、スペイン語の言語コードです。 Visual Studio でこのリソース ファイルを作成するには、次の手順を実行します。

1. **ソリューション エクスプローラー** で、リソース ファイルが格納されているフォルダーを右クリックし、 **[追加]** > **[新しい項目]** を選択します。

   ![入れ子になったコンテキスト メニュー:ソリューション エクスプローラーで、リソースのコンテキスト メニューが開かれます。 [追加] の 2 つ目のコンテキスト メニューが開き、[新しい項目] コマンドが強調表示されます。](localization/_static/newi.png)

1. **インストールされているテンプレートの検索** ボックスに、「resource」と入力し、ファイルに名前を付けます。

   ![[新しい項目の追加] ダイアログ](localization/_static/res.png)

1. キーの値 (ネイティブの文字列) を **[名前]** 列に入力し、翻訳された文字列を **[値]** 列に入力します。

   ![Welcome.es.resx ファイル (スペイン語の Welcome リソース ファイル) と、[名前] 列の Hello という単語と、[値] 列の Hola という単語 (スペイン語のこんにちは)](localization/_static/hola.png)

   Visual Studio に *Welcome.es.resx* ファイルが表示されます。

   ![Welcome スペイン語リソース ファイルを表示しているソリューション エクスプローラー](localization/_static/se.png)

## <a name="resource-file-naming"></a>リソース ファイルの名前付け

リソースの名前は、クラスの完全な型名からアセンブリ名を除いたものになります。 たとえば、メイン アセンブリが `LocalizationWebsite.Web.dll` であるプロジェクト内のクラス `LocalizationWebsite.Web.Startup` のフランス語のリソースは、*Startup.fr.resx* という名前になります。 クラス `LocalizationWebsite.Web.Controllers.HomeController` のリソースは、*Controllers.HomeController.fr.resx* という名前になります。 対象となるクラスの名前空間が、アセンブリ名と同じでない場合は、完全な型名が必要です。 たとえば、同じプロジェクトで、型 `ExtraNamespace.Tools` のリソースは、*ExtraNamespace.Tools.fr.resx* という名前になります。

サンプル プロジェクトで、`ConfigureServices` メソッドは `ResourcesPath` を "Resources" に設定するので、ホーム コントローラーのフランス語のりソース ファイルの相対パスは、*Resources/Controllers.HomeController.fr.resx* です。 あるいは、フォルダーを使用してリソース ファイルを整理することもできます。 Home コント ローラーのパスは、*Resources/Controllers/HomeController.fr.resx* です。 `ResourcesPath` オプションを使用しない場合、 *.resx* ファイルは、プロジェクトの基本ディレクトリに置かれます。 `HomeController` のリソース ファイルは、*Controllers.HomeController.fr.resx* という名前になります。 ドットまたはパスの名前付け規則の選択は、リソース ファイルを整理する方法によって決まります。

| リソース名 | ドットまたはパスの名前付け |
| ------------   | ------------- |
| Resources/Controllers.HomeController.fr.resx | ドット  |
| Resources/Controllers/HomeController.fr.resx  | パス |

Razor ビューの `@inject IViewLocalizer` を使用するリソース ファイルは同様のパターンに従います。 ビューのリソース ファイルには、ドットの名前付けまたはパスの名前付けを使用して名前を付けることができます。 Razor ビューのリソース ファイルは、関連するビュー ファイルのパスを模倣します。 `ResourcesPath` を "Resources" に設定すると仮定すると、*Views/Home/About.cshtml* ビューに関連付けられるフランス語のリソース ファイルは、次のいずれかになります。

* Resources/Views/Home/About.fr.resx

* Resources/Views.Home.About.fr.resx

`ResourcesPath` オプションを使用しない場合、ビューの *.resx* ファイルは、ビューと同じフォルダーに配置されます。

### <a name="rootnamespaceattribute"></a>RootNamespaceAttribute 

アセンブリのルート名前空間がアセンブリ名と異なると、<xref:Microsoft.Extensions.Localization.RootNamespaceAttribute> 属性によってアセンブリのルート名前空間が提供されます。 

> [!WARNING]
> これは、プロジェクト名が有効な .NET 識別子でない場合に発生する可能性があります。 たとえば、`my-project-name.csproj` ではルート名前空間 `my_project_name` が使用されるので、アセンブリ名 `my-project-name` はこのエラーにつながります。 

アセンブリのルート名前空間がアセンブリ名と異なる場合:

* 既定では、ローカライズが機能しません。
* アセンブリ内でのリソースの検索方法が原因で、ローカライズが失敗します。 `RootNamespace` はビルド時の値で、実行中のプロセスでは使用できません。 

`RootNamespace` が `AssemblyName` と異なる場合、*AssemblyInfo.cs* (パラメーターの値は実際の値に置き換えられます) に次を含めてください。

```csharp
using System.Reflection;
using Microsoft.Extensions.Localization;

[assembly: ResourceLocation("Resource Folder Name")]
[assembly: RootNamespace("App Root Namespace")]
```

上記のコードにより、resx ファイルが正常に解決できるようになります。

## <a name="culture-fallback-behavior"></a>カルチャ フォールバック動作

リソースを探すとき、ローカリゼーションは、"カルチャ フォールバック" を行います。 要求されたカルチャが存在しない場合、そのカルチャの親カルチャに戻ります。 なお、[CultureInfo.Parent](/dotnet/api/system.globalization.cultureinfo.parent) プロパティは親カルチャを示します。 つまり、通常は (必ずではありませんが) ISO から国の識別子が削除されます。 たとえば、メキシコで使われているスペイン語は、"es-MX" です。 これには、どの国にも固有でないスペイン語の "es" が親として付いています。

カルチャ "fr-CA" が使用された、"ようこそ" リソースの要求をサイトで受信するとします。 ローカリゼーション システムでは、次の順番でリソースを検索し、最初の一致を選択します。

* *Welcome.fr-CA.resx*
* *Welcome.fr.resx*
* *Welcome.resx* (`NeutralResourcesLanguage` が "fr-CA" の場合)

例として、".fr" カルチャ指定子を削除し、カルチャを French に設定した場合、既定のリソース ファイルは read で文字列がローカライズされます。 リソース マネージャーは、要求されたカルチャを満たすものがない場合、既定またはフォールバックのリソースを指定します。 要求されたカルチャのリソースが見つからないときにキーのみを返す場合は、既定のリソース ファイルを使用することはできません。

### <a name="generate-resource-files-with-visual-studio"></a>Visual Studio でリソース ファイルを生成する

Visual Studio で、ファイル名にカルチャを指定せずにリソース ファイルを作成する場合 (たとえば、*Welcome.resx*)、Visual Studio は各文字列のプロパティを使用して、C# クラスを作成します。 通常、ASP.NET Core ではこれを行いません。 一般的に既定の *.resx* リソース ファイル (カルチャ名のない *.resx* ファイル) は使用しません。 カルチャ名を含む *.resx* ファイル (たとえば *Welcome.fr.resx*) を作成することをお勧めします。 カルチャ名を含む *.resx* ファイルを作成すると、Visual Studio はクラス ファイルを生成しません。

### <a name="add-other-cultures"></a>その他のカルチャを追加する

各言語とカルチャの組み合わせ (既定の言語以外) ごとに一意のリソース ファイルが必要です。 ISO 言語コードがファイル名の一部となるリソース ファイル (たとえば、**en-us**、**fr-ca**、**en-gb**) を新規作成することで、異なるカルチャとロケールのリソース ファイルを作成することができます。 *Welcome.es-MX.resx* (スペイン語/メキシコ) のように、ファイル名と *.resx* ファイル拡張子の間にこれらの ISO コードが置かれます。

## <a name="implement-a-strategy-to-select-the-languageculture-for-each-request"></a>要求ごとに言語/カルチャを選択するための戦略を実装する

### <a name="configure-localization"></a>ローカリゼーションを構成する

ローカリゼーションは、`Startup.ConfigureServices` メソッドで構成されます。

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet1)]

* `AddLocalization` は、ローカリゼーション サービスをサービス コンテナーに追加します。 上記のコードは、リソース パスを "Resources" に設定します。

* `AddViewLocalization` は、ローカライズされたビュー ファイルのサポートを追加します。 このサンプル ビューでは、ローカリゼーションは、ビュー ファイルのサフィックスに基づいています。 たとえば、*Index.fr.cshtml* ファイルの "fr" です。

* `AddDataAnnotationsLocalization` は、`IStringLocalizer` 抽象化を介してローカライズされた `DataAnnotations` 検証メッセージのサポートを追加します。

### <a name="localization-middleware"></a>ローカリゼーション ミドルウェア

要求時に現在のカルチャが、ローカリゼーション [ミドルウェア](xref:fundamentals/middleware/index)で設定されます。 ローカリゼーション ミドルウェアは、`Startup.Configure` メソッドで有効になります。 要求のカルチャをチェックする可能性があるすべてのミドルウェア (たとえば `app.UseMvcWithDefaultRoute()`) の前に、ローカリゼーション ミドルウェアを構成する必要があります。

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet2)]

[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

`UseRequestLocalization` は `RequestLocalizationOptions` オブジェクトを初期化します。 すべての要求で、`RequestLocalizationOptions` の `RequestCultureProvider` のリストが列挙され、要求のカルチャを正常に決定できる最初のプロバイダーが使用されます。 既定のプロバイダーは `RequestLocalizationOptions` クラスから派生します。

1. `QueryStringRequestCultureProvider`
1. `CookieRequestCultureProvider`
1. `AcceptLanguageHeaderRequestCultureProvider`

既定のリストは、最も具体的なものから最も具体的でないものの順番になります。 記事の後半では、この順序を変更する方法、さらにカスタム カルチャ プロバイダーを追加する方法も説明します。 要求のカルチャを判断できるプロバイダーがない場合、`DefaultRequestCulture` が使用されます。

### <a name="querystringrequestcultureprovider"></a>QueryStringRequestCultureProvider

いくつかのアプリでは、クエリ文字列を使用して、<https://docs.microsoft.com/dotnet/api/system.globalization.cultureinfo?view=netcore-3.1>を設定します。 cookie または Accept-language ヘッダーのアプローチを使用するアプリの場合、URL にクエリ文字列を追加すると、デバッグおよびコードのテストに役立ちます。 既定では、`QueryStringRequestCultureProvider` が、`RequestCultureProvider` リストの最初のローカリゼーション プロバイダーとして登録されます。 クエリ文字列パラメーター `culture` と `ui-culture` を渡します。 次の例では、特定のカルチャ (言語および地域) をスペイン語/メキシコに設定します。

   `http://localhost:5000/?culture=es-MX&ui-culture=es-MX`

2 つのいずれかのみ (`culture` または `ui-culture`) を渡した場合、クエリ文字列プロバイダーは、渡した 1 つのパラメーターを使用して両方の値を設定します。 たとえば、カルチャだけを設定すると、`Culture` と `UICulture` の両方が設定されます。

```
http://localhost:5000/?culture=es-MX
```

### <a name="cookierequestcultureprovider"></a>CookieRequestCultureProvider

多くの場合、実稼働アプリケーションは、ASP.NET Core カルチャ cookie を使用してカルチャを設定するためのメカニズムを提供します。 cookie を作成するには、`MakeCookieValue` メソッドを使用します。

`CookieRequestCultureProvider` `DefaultCookieName` は、ユーザーの優先するカルチャ情報を追跡するために使用される既定の cookie 名を返します。 既定の cookie 名は `.AspNetCore.Culture` です。

cookie の形式は `c=%LANGCODE%|uic=%LANGCODE%` です。ここで、`c` は `Culture` であり、`uic` は `UICulture` です。たとえば、次のようになります。

```
c=en-UK|uic=en-US
```

カルチャ情報と UI カルチャの 1 つのみを指定した場合、指定したカルチャが、カルチャ情報と UI カルチャの両方に使用されます。

### <a name="the-accept-language-http-header"></a>Accept-Language HTTP ヘッダー

[Accept-language ヘッダー](https://www.w3.org/International/questions/qa-accept-lang-locales)は、ほとんどのブラウザーで設定可能であり、当初はユーザーの言語を指定するためのものでした。 この設定は、ブラウザーが何を送信するように設定されているか、基になるオペレーティング システムから何を継承するかを示します。 ブラウザーの要求からの Accept Language HTTP ヘッダーは、ユーザーの優先言語を検出するための確実な方法ではありません (「[Setting language preferences in a browser](https://www.w3.org/International/questions/qa-lang-priorities.en.php)」(ブラウザーの優先言語を設定する) を参照してください)。 実稼働アプリには、ユーザーがカルチャの選択をカスタマイズする方法を含める必要があります。

### <a name="set-the-accept-language-http-header-in-ie"></a>IE で Accept-Language HTTP ヘッダーを設定する

1. 歯車アイコンから、 **[インターネット オプション]** をタップします。

1. **[言語]** をタップします。

   ![インターネット オプション](localization/_static/lang.png)

1. **[言語の優先順位の設定]** をタップします。

1. **[言語の追加]** をタップします。

1. 言語を追加します。

1. 言語をタップして、 **[上へ移動]** をタップします。

### <a name="use-a-custom-provider"></a>カスタム プロバイダーを使用する

お客様がデータベースに言語とカルチャを格納できるようにしたいとします。 ユーザーのためにこれらの値を検索するプロバイダーを記述することもできます。 次のコードは、カスタム プロバイダーを追加する方法を示しています。

```csharp
private const string enUSCulture = "en-US";

services.Configure<RequestLocalizationOptions>(options =>
{
    var supportedCultures = new[]
    {
        new CultureInfo(enUSCulture),
        new CultureInfo("fr")
    };

    options.DefaultRequestCulture = new RequestCulture(culture: enUSCulture, uiCulture: enUSCulture);
    options.SupportedCultures = supportedCultures;
    options.SupportedUICultures = supportedCultures;

    options.AddInitialRequestCultureProvider(new CustomRequestCultureProvider(async context =>
    {
        // My custom request culture logic
        return new ProviderCultureResult("en");
    }));
});
```

ローカリゼーション プロバイダーを追加または削除するには、`RequestLocalizationOptions` を使用します。

### <a name="set-the-culture-programmatically"></a>プログラムでカルチャを設定する

この [GitHub](https://github.com/aspnet/entropy) のサンプル **Localization.StarterWeb** プロジェクトには、`Culture` を設定するための UI が含まれています。 *Views/Shared/_SelectLanguagePartial.cshtml* ファイルを使用して、サポートされているカルチャの一覧からカルチャを選択することができます。

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_SelectLanguagePartial.cshtml)]

*Views/Shared/_SelectLanguagePartial.cshtml* ファイルは、レイアウト ファイルの `footer` セクションに追加されるので、すべてのビューで使用できます。

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_Layout.cshtml?range=43-56&highlight=10)]

`SetLanguage` メソッドはカルチャ cookie を設定します。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/HomeController.cs?range=57-67)]

*_SelectLanguagePartial.cshtml* をこのプロジェクトのサンプル コードに接続することはできません。 [GitHub](https://github.com/aspnet/entropy) の **Localization.StarterWeb** プロジェクトには、[依存関係の挿入](dependency-injection.md)コンテナーを介して Razor 部分に `RequestLocalizationOptions` をフローするコードがあります。

## <a name="model-binding-route-data-and-query-strings"></a>モデル バインド ルート データとクエリ文字列

「[モデル バインド ルート データとクエリ文字列のグローバリゼーション動作](xref:mvc/models/model-binding#glob)」を参照してください。

## <a name="globalization-and-localization-terms"></a>グローバリゼーションとローカリゼーションの用語

アプリをローカライズするプロセスでは、最新のソフトウェア開発でよく使用される関連する文字セットの基本的な理解と、それらに関連した問題の理解も必要です。 すべてのコンピューターは、テキストを数値 (コード) として格納しますが、さまざまなシステムが異なる数値を使用して同じテキストを格納します。 ローカリゼーション プロセスとは、特定のカルチャ/ロケール用にアプリのユーザー インターフェイス (UI) を変換することを指します。

[ローカライズ化](/dotnet/standard/globalization-localization/localizability-review)は、グローバル化されたアプリのローカリゼーションの準備ができていることを確認するための中間プロセスです。

カルチャ名の [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) 形式は `<languagecode2>-<country/regioncode2>` です。ここで、`<languagecode2>` は言語コードであり、`<country/regioncode2>` はサブカルチャ コードです。 たとえば、スペイン語 (チリ) は `es-CL`、英語 (米国) は `en-US`、英語 (オーストラリア) は `en-AU` です。 [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) は、言語に関連付けられた ISO 639 の 2 文字の小文字カルチャ コードと、国または地域に関連付けられた ISO 3166 の 2 文字の大文字サブカルチャ コードの組み合わせです。 以下を参照してください。<https://docs.microsoft.com/previous-versions/commerce-server/ee825488(v=cs.20)>

多くの場合、国際化は "I18N" に省略されます。 省略形は、最初と最後の文字およびそれらの間の文字の数になります。したがって、18 は、最初の "I" と最後の "N" の間の文字の数を表します。 同じことが、グローバリゼーション (G11N) とローカリゼーション (L10N) にも当てはまります。

用語:

* グローバリゼーション (G11N):アプリが別の言語と地域をサポートするようにするプロセス。
* ローカリゼーション (L10N):特定の言語と地域向けにアプリをカスタマイズするプロセス。
* 国際化 (I18N):グローバリゼーションとローカリゼーションの両方を示します。
* カルチャ:言語および必要に応じて地域です。
* ニュートラル カルチャ:指定した言語のみを含み、地域は含まないカルチャ。 (例: "en"、"es")。
* 特定のカルチャ:指定した言語と地域を含むカルチャ。 (例: "en-US"、"en-GB"、"es-CL")。
* 親カルチャ:特定のカルチャを含むニュートラル カルチャ。 (たとえば、"en" は "en-US" および "en-GB" の親カルチャです)。
* ロケール:ロケールはカルチャと同じです。

[!INCLUDE[](~/includes/localization/currency.md)]

[!INCLUDE[](~/includes/localization/unsupported-culture-log-level.md)]

## <a name="additional-resources"></a>その他の技術情報

* <xref:fundamentals/troubleshoot-aspnet-core-localization>
* 記事で使用されている [Localization.StarterWeb プロジェクト](https://github.com/aspnet/Entropy/tree/master/samples/Localization.StarterWeb)。
* [.NET アプリケーションのグローバライズとローカライズ](/dotnet/standard/globalization-localization/index)
* [.resx ファイル内のリソース](/dotnet/framework/resources/working-with-resx-files-programmatically)
* [Microsoft 多言語アプリ ツールキット](https://marketplace.visualstudio.com/items?itemName=MultilingualAppToolkit.MultilingualAppToolkit-18308)
* [ローカリゼーションとジェネリック](http://hishambinateya.com/localization-and-generics)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)、[Damien Bowden](https://twitter.com/damien_bod)、[Bart Calixto](https://twitter.com/bartmax)、[Nadeem Afana](https://afana.me/)、[Hisham Bin Ateya](https://twitter.com/hishambinateya)

多言語の Web サイトにすると、サイトはより幅広い対象ユーザーにリーチできます。 ASP.NET Core は、さまざまな言語と文化にローカライズするためのサービスとミドルウェアを提供します。

国際化には、[グローバリゼーション](/dotnet/api/system.globalization)と[ローカリゼーション](/dotnet/standard/globalization-localization/localization)が含まれます。 グローバリゼーションとは異なるカルチャをサポートするアプリを設計するプロセスです。 グローバリゼーションによって、特定の地域に関連する定義済みの言語セットの入出力と表示のサポートが追加されます。

ローカリゼーションとは、ローカライズのために既に処理されているグローバル化されたアプリを特定のカルチャ/ロケールに適合させるプロセスです。 詳細については、本ドキュメントの末尾にある「**グローバリゼーションとローカリゼーションの用語**」を参照してください。

アプリのローカリゼーションには、以下のものが関与します。

1. アプリのコンテンツをローカライズできるようにする
1. サポートする言語およびカルチャのローカライズされたリソースを提供する
1. 要求ごとに言語/カルチャを選択するための戦略を実装する

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/localization/sample/Localization)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="make-the-apps-content-localizable"></a>アプリのコンテンツをローカライズできるようにする

<xref:Microsoft.Extensions.Localization.IStringLocalizer> と <xref:Microsoft.Extensions.Localization.IStringLocalizer%601> は、ローカライズされたアプリの開発時に生産性が向上するように設計されています。 `IStringLocalizer` では、<xref:System.Resources.ResourceManager> と <xref:System.Resources.ResourceReader> を使用して、実行時にカルチャ固有のリソースを提供します。 インターフェイスには、ローカライズされた文字列を返すためのインデクサーと `IEnumerable` があります。 `IStringLocalizer` では、既定の言語文字列をリソース ファイルに格納する必要はありません。 ローカリゼーションの対象となるアプリを開発することができ、開発の早い段階でリソース ファイルを作成する必要はありません。 次のコードは、ローカリゼーションのために文字列 "About Title" をラップする方法を示しています。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/AboutController.cs)]

前のコードの `IStringLocalizer<T>` の実装は、[依存関係の挿入](dependency-injection.md)で説明したものです。 "About Title" のローカライズされた値が見つからない場合、インデクサーのキー、つまり文字列 "About Title" が返されます。 アプリで、既定の言語のリテラル文字列をそのままにし、ローカライザーにそれらをラップすることができるので、アプリの開発に専念することができます。 既定の言語でアプリを開発し、既定のリソース ファイルを最初に作成せずに、ローカリゼーション手順用に準備します。 または、従来のアプローチを使用し、既定の言語文字列を取得するキーを提供できます。 既定の言語の *.resx* ファイルを使用せずに、文字列リテラルをラップするだけの新しいワークフローは、多くの開発者にとって、アプリをローカライズする際のオーバーヘッドの削減になります。 他の開発者は、長い文字列リテラルの操作が容易で、ローカライズされた文字列を更新しやすい従来のワークフローを好みます。

HTML を格納しているリソースには、`IHtmlLocalizer<T>` の実装を使用します。 `IHtmlLocalizer` HTML は、リソース文字列でフォーマットされた引数をエンコードしますが、リソース文字列自体は HTML エンコードしません。 下の強調表示されたサンプルでは、`name` パラメーターの値のみが HTML エンコードされます。

[!code-csharp[](~/fundamentals/localization/sample/3.x/Localization/Controllers/BookController.cs?highlight=3,5,20&start=1&end=24)]

> [!NOTE]
> 一般に、HTML ではなく、テキストのみをローカライズします。

最下位のレベルでは、[依存関係の挿入](dependency-injection.md)から `IStringLocalizerFactory` を取得できます。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/TestController.cs?start=9&end=26&highlight=7-13)]

上記のコードは、2 つの各ファクトリ作成メソッドを示しています。

ローカライズされた文字列は、コントローラーまたは領域で仕切るか、1 つのコンテナーにすることができます。 サンプル アプリでは、共有されるリソースのために `SharedResource` というダミー クラスを使用しています。

[!code-csharp[](localization/sample/3.x/Localization/SharedResource.cs)]

一部の開発者は、`Startup` クラスを使用してグローバル文字列または共有文字列を格納します。 下のサンプルでは、`InfoController` と `SharedResource` のローカライザーが使用されています。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/InfoController.cs?range=9-26)]

## <a name="view-localization"></a>ビューのローカリゼーション

`IViewLocalizer` サービスは、[ビュー](xref:mvc/views/overview)のローカライズされた文字列を提供します。 `ViewLocalizer` クラスは、このインターフェイスを実装し、ビューのファイル パスからリソースの場所を見つけます。 次のコードは、`IViewLocalizer` の既定の実装の使用方法を示しています。

[!code-cshtml[](localization/sample/3.x/Localization/Views/Home/About.cshtml)]

`IViewLocalizer` の既定の実装は、ビューのファイル名に基づいてリソース ファイルを見つけます。 グローバルな共有リソース ファイルを使用するオプションはありません。 `ViewLocalizer` は、`IHtmlLocalizer` を使用してローカライザーを実装するので、Razor は、ローカライズされた文字列を HTML エンコードしません。 リソース文字列をパラメーター化することができます。`IViewLocalizer` は、パラメーターを HTML エンコードしますが、リソース文字列は HTML エンコードしません。 次の Razor マークアップがあるとします。

```cshtml
@Localizer["<i>Hello</i> <b>{0}!</b>", UserManager.GetUserName(User)]
```

フランス語のリソース ファイルには次の値が含まれます。

| Key | [値] |
| --- | ----- |
| `<i>Hello</i> <b>{0}!</b>` | `<i>Bonjour</i> <b>{0} !</b>` |

描画されたビューには、リソース ファイルからの HTML マークアップが含まれます。

> [!NOTE]
> 一般に、HTML ではなく、テキストのみをローカライズします。

ビュー内の共有リソース ファイルを使用するには、`IHtmlLocalizer<T>` を挿入します。

[!code-cshtml[](~/fundamentals/localization/sample/3.x/Localization/Views/Test/About.cshtml?highlight=5,12)]

## <a name="dataannotations-localization"></a>DataAnnotations のローカリゼーション

DataAnnotations エラー メッセージは `IStringLocalizer<T>` を使用してローカライズします。 オプション `ResourcesPath = "Resources"` を使用して、`RegisterViewModel` のエラー メッセージを次のいずれかのパスに格納できます。

* *Resources/ViewModels.Account.RegisterViewModel.fr.resx*
* *Resources/ViewModels/Account/RegisterViewModel.fr.resx*

[!code-csharp[](localization/sample/3.x/Localization/ViewModels/Account/RegisterViewModel.cs?start=9&end=26)]

ASP.NET Core MVC 1.1.0 以上では、非検証属性がローカライズされます。 ASP.NET Core MVC 1.0 は、非検証属性のローカライズされた文字列を検索 **しません**。

<a name="one-resource-string-multiple-classes"></a>

### <a name="using-one-resource-string-for-multiple-classes"></a>複数のクラスに 1 つのリソース文字列を使用する

次のコードは、複数のクラスに検証属性の 1 つのリソース文字列を使用する方法を示しています。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddDataAnnotationsLocalization(options => {
            options.DataAnnotationLocalizerProvider = (type, factory) =>
                factory.Create(typeof(SharedResource));
        });
}
```

上記のコードでは、`SharedResource` は、resx に対応するクラスであり、ここに検証メッセージが格納されます。 この方法では、DataAnnotations は、各クラスのリソースではなく、`SharedResource` のみを使用します。

## <a name="provide-localized-resources-for-the-languages-and-cultures-you-support"></a>サポートする言語およびカルチャのローカライズされたリソースを提供する

### <a name="supportedcultures-and-supporteduicultures"></a>SupportedCultures と SupportedUICultures

ASP.NET Core では、`SupportedCultures` と `SupportedUICultures` という 2 つのカルチャ値を指定できます。 日付、数値、および通貨の書式設定など、カルチャに依存する関数の結果は、`SupportedCultures` の [CultureInfo](/dotnet/api/system.globalization.cultureinfo) オブジェクトによって決まります。 テキストの並べ替え順序、大文字と小文字の表記規則、文字列比較も `SupportedCultures` によって決まります。 サーバーがカルチャを取得する方法の詳細については、「[CultureInfo.CurrentCulture](/dotnet/api/system.stringcomparer.currentculture#System_StringComparer_CurrentCulture)」を参照してください。 `SupportedUICultures` は、 *.resx* ファイルからのどの翻訳文字列が [ResourceManager](/dotnet/api/system.resources.resourcemanager) によって検索されるかを決定します。 `ResourceManager` は、`CurrentUICulture` によって決定されるカルチャ固有の文字列を単に検索します。 .NET のすべてのスレッドに `CurrentCulture` オブジェクトと `CurrentUICulture`オブジェクトがあります。 ASP.NET Core は、カルチャに依存する関数を表示するときに、これらの値を検査します。 たとえば、現在のスレッドのカルチャが "en-US" (英語、米国) に設定されている場合は、`DateTime.Now.ToLongDateString()` は、"Thursday, February 18, 2016" を表示しますが、`CurrentCulture` が "es-ES" (スペイン語、スペイン) に設定されている場合、出力は "jueves, 18 de febrero de 2016" になります。

## <a name="resource-files"></a>リソース ファイル

リソース ファイルは、ローカライズ可能な文字列をコードから分離するために役立つメカニズムです。 既定以外の言語の翻訳された文字列は、 *.resx* リソース ファイルで分離されています。 たとえば、翻訳された文字列を含む *Welcome.es.resx* という名前のスペイン語のリソース ファイルを作成したい場合があります。 "es" は、スペイン語の言語コードです。 Visual Studio でこのリソース ファイルを作成するには、次の手順を実行します。

1. **ソリューション エクスプローラー** で、リソース ファイルが格納されているフォルダーを右クリックし、 **[追加]** > **[新しい項目]** を選択します。

   ![入れ子になったコンテキスト メニュー:ソリューション エクスプローラーで、リソースのコンテキスト メニューが開かれます。 [追加] の 2 つ目のコンテキスト メニューが開き、[新しい項目] コマンドが強調表示されます。](localization/_static/newi.png)

1. **インストールされているテンプレートの検索** ボックスに、「resource」と入力し、ファイルに名前を付けます。

   ![[新しい項目の追加] ダイアログ](localization/_static/res.png)

1. キーの値 (ネイティブの文字列) を **[名前]** 列に入力し、翻訳された文字列を **[値]** 列に入力します。

   ![Welcome.es.resx ファイル (スペイン語の Welcome リソース ファイル) と、[名前] 列の Hello という単語と、[値] 列の Hola という単語 (スペイン語のこんにちは)](localization/_static/hola.png)

   Visual Studio に *Welcome.es.resx* ファイルが表示されます。

   ![Welcome スペイン語リソース ファイルを表示しているソリューション エクスプローラー](localization/_static/se.png)

## <a name="resource-file-naming"></a>リソース ファイルの名前付け

リソースの名前は、クラスの完全な型名からアセンブリ名を除いたものになります。 たとえば、メイン アセンブリが `LocalizationWebsite.Web.dll` であるプロジェクト内のクラス `LocalizationWebsite.Web.Startup` のフランス語のリソースは、*Startup.fr.resx* という名前になります。 クラス `LocalizationWebsite.Web.Controllers.HomeController` のリソースは、*Controllers.HomeController.fr.resx* という名前になります。 対象となるクラスの名前空間が、アセンブリ名と同じでない場合は、完全な型名が必要です。 たとえば、同じプロジェクトで、型 `ExtraNamespace.Tools` のリソースは、*ExtraNamespace.Tools.fr.resx* という名前になります。

サンプル プロジェクトで、`ConfigureServices` メソッドは `ResourcesPath` を "Resources" に設定するので、ホーム コントローラーのフランス語のりソース ファイルの相対パスは、*Resources/Controllers.HomeController.fr.resx* です。 あるいは、フォルダーを使用してリソース ファイルを整理することもできます。 Home コント ローラーのパスは、*Resources/Controllers/HomeController.fr.resx* です。 `ResourcesPath` オプションを使用しない場合、 *.resx* ファイルは、プロジェクトの基本ディレクトリに置かれます。 `HomeController` のリソース ファイルは、*Controllers.HomeController.fr.resx* という名前になります。 ドットまたはパスの名前付け規則の選択は、リソース ファイルを整理する方法によって決まります。

| リソース名 | ドットまたはパスの名前付け |
| ------------   | ------------- |
| Resources/Controllers.HomeController.fr.resx | ドット  |
| Resources/Controllers/HomeController.fr.resx  | パス |

Razor ビューの `@inject IViewLocalizer` を使用するリソース ファイルは同様のパターンに従います。 ビューのリソース ファイルには、ドットの名前付けまたはパスの名前付けを使用して名前を付けることができます。 Razor ビューのリソース ファイルは、関連するビュー ファイルのパスを模倣します。 `ResourcesPath` を "Resources" に設定すると仮定すると、*Views/Home/About.cshtml* ビューに関連付けられるフランス語のリソース ファイルは、次のいずれかになります。

* Resources/Views/Home/About.fr.resx

* Resources/Views.Home.About.fr.resx

`ResourcesPath` オプションを使用しない場合、ビューの *.resx* ファイルは、ビューと同じフォルダーに配置されます。

### <a name="rootnamespaceattribute"></a>RootNamespaceAttribute 

アセンブリのルート名前空間がアセンブリ名と異なると、<xref:Microsoft.Extensions.Localization.RootNamespaceAttribute> 属性によってアセンブリのルート名前空間が提供されます。 

> [!WARNING]
> これは、プロジェクト名が有効な .NET 識別子でない場合に発生する可能性があります。 たとえば、`my-project-name.csproj` ではルート名前空間 `my_project_name` が使用されるので、アセンブリ名 `my-project-name` はこのエラーにつながります。 

アセンブリのルート名前空間がアセンブリ名と異なる場合:

* 既定では、ローカライズが機能しません。
* アセンブリ内でのリソースの検索方法が原因で、ローカライズが失敗します。 `RootNamespace` はビルド時の値で、実行中のプロセスでは使用できません。 

`RootNamespace` が `AssemblyName` と異なる場合、*AssemblyInfo.cs* (パラメーターの値は実際の値に置き換えられます) に次を含めてください。

```csharp
using System.Reflection;
using Microsoft.Extensions.Localization;

[assembly: ResourceLocation("Resource Folder Name")]
[assembly: RootNamespace("App Root Namespace")]
```

上記のコードにより、resx ファイルが正常に解決できるようになります。

## <a name="culture-fallback-behavior"></a>カルチャ フォールバック動作

リソースを探すとき、ローカリゼーションは、"カルチャ フォールバック" を行います。 要求されたカルチャが存在しない場合、そのカルチャの親カルチャに戻ります。 なお、[CultureInfo.Parent](/dotnet/api/system.globalization.cultureinfo.parent) プロパティは親カルチャを示します。 つまり、通常は (必ずではありませんが) ISO から国の識別子が削除されます。 たとえば、メキシコで使われているスペイン語は、"es-MX" です。 これには、どの国にも固有でないスペイン語の "es" が親として付いています。

カルチャ "fr-CA" が使用された、"ようこそ" リソースの要求をサイトで受信するとします。 ローカリゼーション システムでは、次の順番でリソースを検索し、最初の一致を選択します。

* *Welcome.fr-CA.resx*
* *Welcome.fr.resx*
* *Welcome.resx* (`NeutralResourcesLanguage` が "fr-CA" の場合)

例として、".fr" カルチャ指定子を削除し、カルチャを French に設定した場合、既定のリソース ファイルは read で文字列がローカライズされます。 リソース マネージャーは、要求されたカルチャを満たすものがない場合、既定またはフォールバックのリソースを指定します。 要求されたカルチャのリソースが見つからないときにキーのみを返す場合は、既定のリソース ファイルを使用することはできません。

### <a name="generate-resource-files-with-visual-studio"></a>Visual Studio でリソース ファイルを生成する

Visual Studio で、ファイル名にカルチャを指定せずにリソース ファイルを作成する場合 (たとえば、*Welcome.resx*)、Visual Studio は各文字列のプロパティを使用して、C# クラスを作成します。 通常、ASP.NET Core ではこれを行いません。 一般的に既定の *.resx* リソース ファイル (カルチャ名のない *.resx* ファイル) は使用しません。 カルチャ名を含む *.resx* ファイル (たとえば *Welcome.fr.resx*) を作成することをお勧めします。 カルチャ名を含む *.resx* ファイルを作成すると、Visual Studio はクラス ファイルを生成しません。

### <a name="add-other-cultures"></a>その他のカルチャを追加する

各言語とカルチャの組み合わせ (既定の言語以外) ごとに一意のリソース ファイルが必要です。 ISO 言語コードがファイル名の一部となるリソース ファイル (たとえば、**en-us**、**fr-ca**、**en-gb**) を新規作成することで、異なるカルチャとロケールのリソース ファイルを作成することができます。 *Welcome.es-MX.resx* (スペイン語/メキシコ) のように、ファイル名と *.resx* ファイル拡張子の間にこれらの ISO コードが置かれます。

## <a name="implement-a-strategy-to-select-the-languageculture-for-each-request"></a>要求ごとに言語/カルチャを選択するための戦略を実装する

### <a name="configure-localization"></a>ローカリゼーションを構成する

ローカリゼーションは、`Startup.ConfigureServices` メソッドで構成されます。

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet1)]

* `AddLocalization` は、ローカリゼーション サービスをサービス コンテナーに追加します。 上記のコードは、リソース パスを "Resources" に設定します。

* `AddViewLocalization` は、ローカライズされたビュー ファイルのサポートを追加します。 このサンプル ビューでは、ローカリゼーションは、ビュー ファイルのサフィックスに基づいています。 たとえば、*Index.fr.cshtml* ファイルの "fr" です。

* `AddDataAnnotationsLocalization` は、`IStringLocalizer` 抽象化を介してローカライズされた `DataAnnotations` 検証メッセージのサポートを追加します。

### <a name="localization-middleware"></a>ローカリゼーション ミドルウェア

要求時に現在のカルチャが、ローカリゼーション [ミドルウェア](xref:fundamentals/middleware/index)で設定されます。 ローカリゼーション ミドルウェアは、`Startup.Configure` メソッドで有効になります。 要求のカルチャをチェックする可能性があるすべてのミドルウェア (たとえば `app.UseMvcWithDefaultRoute()`) の前に、ローカリゼーション ミドルウェアを構成する必要があります。

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet2)]

[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

`UseRequestLocalization` は `RequestLocalizationOptions` オブジェクトを初期化します。 すべての要求で、`RequestLocalizationOptions` の `RequestCultureProvider` のリストが列挙され、要求のカルチャを正常に決定できる最初のプロバイダーが使用されます。 既定のプロバイダーは `RequestLocalizationOptions` クラスから派生します。

1. `QueryStringRequestCultureProvider`
1. `CookieRequestCultureProvider`
1. `AcceptLanguageHeaderRequestCultureProvider`

既定のリストは、最も具体的なものから最も具体的でないものの順番になります。 記事の後半では、この順序を変更する方法、さらにカスタム カルチャ プロバイダーを追加する方法も説明します。 要求のカルチャを判断できるプロバイダーがない場合、`DefaultRequestCulture` が使用されます。

### <a name="querystringrequestcultureprovider"></a>QueryStringRequestCultureProvider

いくつかのアプリでは、クエリ文字列を使用して、<https://docs.microsoft.com/dotnet/api/system.globalization.cultureinfo?view=netcore-3.1>を設定します。 cookie または Accept-language ヘッダーのアプローチを使用するアプリの場合、URL にクエリ文字列を追加すると、デバッグおよびコードのテストに役立ちます。 既定では、`QueryStringRequestCultureProvider` が、`RequestCultureProvider` リストの最初のローカリゼーション プロバイダーとして登録されます。 クエリ文字列パラメーター `culture` と `ui-culture` を渡します。 次の例では、特定のカルチャ (言語および地域) をスペイン語/メキシコに設定します。

```
http://localhost:5000/?culture=es-MX&ui-culture=es-MX
```

2 つのいずれかのみ (`culture` または `ui-culture`) を渡した場合、クエリ文字列プロバイダーは、渡した 1 つのパラメーターを使用して両方の値を設定します。 たとえば、カルチャだけを設定すると、`Culture` と `UICulture` の両方が設定されます。

```
http://localhost:5000/?culture=es-MX
```

### <a name="cookierequestcultureprovider"></a>CookieRequestCultureProvider

多くの場合、実稼働アプリケーションは、ASP.NET Core カルチャ cookie を使用してカルチャを設定するためのメカニズムを提供します。 cookie を作成するには、`MakeCookieValue` メソッドを使用します。

`CookieRequestCultureProvider` `DefaultCookieName` は、ユーザーの優先するカルチャ情報を追跡するために使用される既定の cookie 名を返します。 既定の cookie 名は `.AspNetCore.Culture` です。

cookie の形式は `c=%LANGCODE%|uic=%LANGCODE%` です。ここで、`c` は `Culture` であり、`uic` は `UICulture` です。たとえば、次のようになります。

```
c=en-UK|uic=en-US
```

カルチャ情報と UI カルチャの 1 つのみを指定した場合、指定したカルチャが、カルチャ情報と UI カルチャの両方に使用されます。

### <a name="the-accept-language-http-header"></a>Accept-Language HTTP ヘッダー

[Accept-language ヘッダー](https://www.w3.org/International/questions/qa-accept-lang-locales)は、ほとんどのブラウザーで設定可能であり、当初はユーザーの言語を指定するためのものでした。 この設定は、ブラウザーが何を送信するように設定されているか、基になるオペレーティング システムから何を継承するかを示します。 ブラウザーの要求からの Accept Language HTTP ヘッダーは、ユーザーの優先言語を検出するための確実な方法ではありません (「[Setting language preferences in a browser](https://www.w3.org/International/questions/qa-lang-priorities.en.php)」(ブラウザーの優先言語を設定する) を参照してください)。 実稼働アプリには、ユーザーがカルチャの選択をカスタマイズする方法を含める必要があります。

### <a name="set-the-accept-language-http-header-in-ie"></a>IE で Accept-Language HTTP ヘッダーを設定する

1. 歯車アイコンから、 **[インターネット オプション]** をタップします。

1. **[言語]** をタップします。

   ![インターネット オプション](localization/_static/lang.png)

1. **[言語の優先順位の設定]** をタップします。

1. **[言語の追加]** をタップします。

1. 言語を追加します。

1. 言語をタップして、 **[上へ移動]** をタップします。

### <a name="use-a-custom-provider"></a>カスタム プロバイダーを使用する

お客様がデータベースに言語とカルチャを格納できるようにしたいとします。 ユーザーのためにこれらの値を検索するプロバイダーを記述することもできます。 次のコードは、カスタム プロバイダーを追加する方法を示しています。

```csharp
private const string enUSCulture = "en-US";

services.Configure<RequestLocalizationOptions>(options =>
{
    var supportedCultures = new[]
    {
        new CultureInfo(enUSCulture),
        new CultureInfo("fr")
    };

    options.DefaultRequestCulture = new RequestCulture(culture: enUSCulture, uiCulture: enUSCulture);
    options.SupportedCultures = supportedCultures;
    options.SupportedUICultures = supportedCultures;

    options.RequestCultureProviders.Insert(0, new CustomRequestCultureProvider(async context =>
    {
        // My custom request culture logic
        return new ProviderCultureResult("en");
    }));
});
```

ローカリゼーション プロバイダーを追加または削除するには、`RequestLocalizationOptions` を使用します。

### <a name="set-the-culture-programmatically"></a>プログラムでカルチャを設定する

この [GitHub](https://github.com/aspnet/entropy) のサンプル **Localization.StarterWeb** プロジェクトには、`Culture` を設定するための UI が含まれています。 *Views/Shared/_SelectLanguagePartial.cshtml* ファイルを使用して、サポートされているカルチャの一覧からカルチャを選択することができます。

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_SelectLanguagePartial.cshtml)]

*Views/Shared/_SelectLanguagePartial.cshtml* ファイルは、レイアウト ファイルの `footer` セクションに追加されるので、すべてのビューで使用できます。

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_Layout.cshtml?range=43-56&highlight=10)]

`SetLanguage` メソッドはカルチャ cookie を設定します。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/HomeController.cs?range=57-67)]

*_SelectLanguagePartial.cshtml* をこのプロジェクトのサンプル コードに接続することはできません。 [GitHub](https://github.com/aspnet/entropy) の **Localization.StarterWeb** プロジェクトには、[依存関係の挿入](dependency-injection.md)コンテナーを介して Razor 部分に `RequestLocalizationOptions` をフローするコードがあります。

## <a name="model-binding-route-data-and-query-strings"></a>モデル バインド ルート データとクエリ文字列

「[モデル バインド ルート データとクエリ文字列のグローバリゼーション動作](xref:mvc/models/model-binding#glob)」を参照してください。

## <a name="globalization-and-localization-terms"></a>グローバリゼーションとローカリゼーションの用語

アプリをローカライズするプロセスでは、最新のソフトウェア開発でよく使用される関連する文字セットの基本的な理解と、それらに関連した問題の理解も必要です。 すべてのコンピューターは、テキストを数値 (コード) として格納しますが、さまざまなシステムが異なる数値を使用して同じテキストを格納します。 ローカリゼーション プロセスとは、特定のカルチャ/ロケール用にアプリのユーザー インターフェイス (UI) を変換することを指します。

[ローカライズ化](/dotnet/standard/globalization-localization/localizability-review)は、グローバル化されたアプリのローカリゼーションの準備ができていることを確認するための中間プロセスです。

カルチャ名の [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) 形式は `<languagecode2>-<country/regioncode2>` です。ここで、`<languagecode2>` は言語コードであり、`<country/regioncode2>` はサブカルチャ コードです。 たとえば、スペイン語 (チリ) は `es-CL`、英語 (米国) は `en-US`、英語 (オーストラリア) は `en-AU` です。 [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) は、言語に関連付けられた ISO 639 の 2 文字の小文字カルチャ コードと、国または地域に関連付けられた ISO 3166 の 2 文字の大文字サブカルチャ コードの組み合わせです。 以下を参照してください。<https://docs.microsoft.com/previous-versions/commerce-server/ee825488(v=cs.20)>

多くの場合、国際化は "I18N" に省略されます。 省略形は、最初と最後の文字およびそれらの間の文字の数になります。したがって、18 は、最初の "I" と最後の "N" の間の文字の数を表します。 同じことが、グローバリゼーション (G11N) とローカリゼーション (L10N) にも当てはまります。

用語:

* グローバリゼーション (G11N):アプリが別の言語と地域をサポートするようにするプロセス。
* ローカリゼーション (L10N):特定の言語と地域向けにアプリをカスタマイズするプロセス。
* 国際化 (I18N):グローバリゼーションとローカリゼーションの両方を示します。
* カルチャ:言語および必要に応じて地域です。
* ニュートラル カルチャ:指定した言語のみを含み、地域は含まないカルチャ。 (例: "en"、"es")。
* 特定のカルチャ:指定した言語と地域を含むカルチャ。 (例: "en-US"、"en-GB"、"es-CL")。
* 親カルチャ:特定のカルチャを含むニュートラル カルチャ。 (たとえば、"en" は "en-US" および "en-GB" の親カルチャです)。
* ロケール:ロケールはカルチャと同じです。

[!INCLUDE[](~/includes/localization/currency.md)]

## <a name="additional-resources"></a>その他の技術情報

* <xref:fundamentals/troubleshoot-aspnet-core-localization>
* 記事で使用されている [Localization.StarterWeb プロジェクト](https://github.com/aspnet/Entropy/tree/master/samples/Localization.StarterWeb)。
* [.NET アプリケーションのグローバライズとローカライズ](/dotnet/standard/globalization-localization/index)
* [.resx ファイル内のリソース](/dotnet/framework/resources/working-with-resx-files-programmatically)
* [Microsoft 多言語アプリ ツールキット](https://marketplace.visualstudio.com/items?itemName=MultilingualAppToolkit.MultilingualAppToolkit-18308)
* [ローカリゼーションとジェネリック](http://hishambinateya.com/localization-and-generics)

::: moniker-end

<!-- ASP.NET Core 5.x starts here -->
::: moniker range="> aspnetcore-3.1"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)、[Damien Bowden](https://twitter.com/damien_bod)、[Bart Calixto](https://twitter.com/bartmax)、[Nadeem Afana](https://afana.me/)、[Hisham Bin Ateya](https://twitter.com/hishambinateya)

多言語の Web サイトにすると、サイトはより幅広い対象ユーザーにリーチできます。 ASP.NET Core は、さまざまな言語と文化にローカライズするためのサービスとミドルウェアを提供します。

国際化には、[グローバリゼーション](/dotnet/api/system.globalization)と[ローカリゼーション](/dotnet/standard/globalization-localization/localization)が含まれます。 グローバリゼーションとは異なるカルチャをサポートするアプリを設計するプロセスです。 グローバリゼーションによって、特定の地域に関連する定義済みの言語セットの入出力と表示のサポートが追加されます。

ローカリゼーションとは、ローカライズのために既に処理されているグローバル化されたアプリを特定のカルチャ/ロケールに適合させるプロセスです。 詳細については、本ドキュメントの末尾にある「**グローバリゼーションとローカリゼーションの用語**」を参照してください。

アプリのローカリゼーションには、以下のものが関与します。

1. アプリのコンテンツをローカライズできるようにする
1. サポートする言語およびカルチャのローカライズされたリソースを提供する
1. 要求ごとに言語/カルチャを選択するための戦略を実装する

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/localization/sample/2.x/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="make-the-apps-content-localizable"></a>アプリのコンテンツをローカライズできるようにする

<xref:Microsoft.Extensions.Localization.IStringLocalizer> と <xref:Microsoft.Extensions.Localization.IStringLocalizer%601> は、ローカライズされたアプリの開発時に生産性が向上するように設計されています。 `IStringLocalizer` では、<xref:System.Resources.ResourceManager> と <xref:System.Resources.ResourceReader> を使用して、実行時にカルチャ固有のリソースを提供します。 インターフェイスには、ローカライズされた文字列を返すためのインデクサーと `IEnumerable` があります。 `IStringLocalizer` では、既定の言語文字列をリソース ファイルに格納する必要はありません。 ローカリゼーションの対象となるアプリを開発することができ、開発の早い段階でリソース ファイルを作成する必要はありません。 次のコードは、ローカリゼーションのために文字列 "About Title" をラップする方法を示しています。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/AboutController.cs)]

前のコードの `IStringLocalizer<T>` の実装は、[依存関係の挿入](dependency-injection.md)で説明したものです。 "About Title" のローカライズされた値が見つからない場合、インデクサーのキー、つまり文字列 "About Title" が返されます。 アプリで、既定の言語のリテラル文字列をそのままにし、ローカライザーにそれらをラップすることができるので、アプリの開発に専念することができます。 既定の言語でアプリを開発し、既定のリソース ファイルを最初に作成せずに、ローカリゼーション手順用に準備します。 または、従来のアプローチを使用し、既定の言語文字列を取得するキーを提供できます。 既定の言語の *.resx* ファイルを使用せずに、文字列リテラルをラップするだけの新しいワークフローは、多くの開発者にとって、アプリをローカライズする際のオーバーヘッドの削減になります。 他の開発者は、長い文字列リテラルの操作が容易で、ローカライズされた文字列を更新しやすい従来のワークフローを好みます。

HTML を格納しているリソースには、`IHtmlLocalizer<T>` の実装を使用します。 `IHtmlLocalizer` HTML は、リソース文字列でフォーマットされた引数をエンコードしますが、リソース文字列自体は HTML エンコードしません。 下の強調表示されたサンプルでは、`name` パラメーターの値のみが HTML エンコードされます。

[!code-csharp[](~/fundamentals/localization/sample/3.x/Localization/Controllers/BookController.cs?highlight=3,5,20&start=1&end=24)]

> [!NOTE]
> 一般に、HTML ではなく、テキストのみをローカライズします。

最下位のレベルでは、[依存関係の挿入](dependency-injection.md)から `IStringLocalizerFactory` を取得できます。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/TestController.cs?start=9&end=26&highlight=7-13)]

上記のコードは、2 つの各ファクトリ作成メソッドを示しています。

ローカライズされた文字列は、コントローラーまたは領域で仕切るか、1 つのコンテナーにすることができます。 サンプル アプリでは、共有されるリソースのために `SharedResource` というダミー クラスを使用しています。

[!code-csharp[](localization/sample/3.x/Localization/SharedResource.cs)]

一部の開発者は、`Startup` クラスを使用してグローバル文字列または共有文字列を格納します。 下のサンプルでは、`InfoController` と `SharedResource` のローカライザーが使用されています。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/InfoController.cs?range=9-26)]

## <a name="view-localization"></a>ビューのローカリゼーション

`IViewLocalizer` サービスは、[ビュー](xref:mvc/views/overview)のローカライズされた文字列を提供します。 `ViewLocalizer` クラスは、このインターフェイスを実装し、ビューのファイル パスからリソースの場所を見つけます。 次のコードは、`IViewLocalizer` の既定の実装の使用方法を示しています。

[!code-cshtml[](localization/sample/3.x/Localization/Views/Home/About.cshtml)]

`IViewLocalizer` の既定の実装は、ビューのファイル名に基づいてリソース ファイルを見つけます。 グローバルな共有リソース ファイルを使用するオプションはありません。 `ViewLocalizer` は、`IHtmlLocalizer` を使用してローカライザーを実装するので、Razor は、ローカライズされた文字列を HTML エンコードしません。 リソース文字列をパラメーター化することができます。`IViewLocalizer` は、パラメーターを HTML エンコードしますが、リソース文字列は HTML エンコードしません。 次の Razor マークアップがあるとします。

```cshtml
@Localizer["<i>Hello</i> <b>{0}!</b>", UserManager.GetUserName(User)]
```

フランス語のリソース ファイルには次の値が含まれます。

| Key | [値] |
| --- | ----- |
| `<i>Hello</i> <b>{0}!</b>` | `<i>Bonjour</i> <b>{0} !</b>` |

描画されたビューには、リソース ファイルからの HTML マークアップが含まれます。

> [!NOTE]
> 一般に、HTML ではなく、テキストのみをローカライズします。

ビュー内の共有リソース ファイルを使用するには、`IHtmlLocalizer<T>` を挿入します。

[!code-cshtml[](~/fundamentals/localization/sample/3.x/Localization/Views/Test/About.cshtml?highlight=5,12)]

## <a name="dataannotations-localization"></a>DataAnnotations のローカリゼーション

DataAnnotations エラー メッセージは `IStringLocalizer<T>` を使用してローカライズします。 オプション `ResourcesPath = "Resources"` を使用して、`RegisterViewModel` のエラー メッセージを次のいずれかのパスに格納できます。

* *Resources/ViewModels.Account.RegisterViewModel.fr.resx*
* *Resources/ViewModels/Account/RegisterViewModel.fr.resx*

[!code-csharp[](localization/sample/3.x/Localization/ViewModels/Account/RegisterViewModel.cs?start=9&end=26)]

ASP.NET Core MVC 1.1.0 以上では、非検証属性がローカライズされます。 ASP.NET Core MVC 1.0 は、非検証属性のローカライズされた文字列を検索 **しません**。

<a name="one-resource-string-multiple-classes"></a>

### <a name="using-one-resource-string-for-multiple-classes"></a>複数のクラスに 1 つのリソース文字列を使用する

次のコードは、複数のクラスに検証属性の 1 つのリソース文字列を使用する方法を示しています。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddDataAnnotationsLocalization(options => {
            options.DataAnnotationLocalizerProvider = (type, factory) =>
                factory.Create(typeof(SharedResource));
        });
}
```

上記のコードでは、`SharedResource` は、resx に対応するクラスであり、ここに検証メッセージが格納されます。 この方法では、DataAnnotations は、各クラスのリソースではなく、`SharedResource` のみを使用します。

## <a name="provide-localized-resources-for-the-languages-and-cultures-you-support"></a>サポートする言語およびカルチャのローカライズされたリソースを提供する

### <a name="supportedcultures-and-supporteduicultures"></a>SupportedCultures と SupportedUICultures

ASP.NET Core では、`SupportedCultures` と `SupportedUICultures` という 2 つのカルチャ値を指定できます。 日付、数値、および通貨の書式設定など、カルチャに依存する関数の結果は、`SupportedCultures` の [CultureInfo](/dotnet/api/system.globalization.cultureinfo) オブジェクトによって決まります。 テキストの並べ替え順序、大文字と小文字の表記規則、文字列比較も `SupportedCultures` によって決まります。 サーバーがカルチャを取得する方法の詳細については、「[CultureInfo.CurrentCulture](/dotnet/api/system.stringcomparer.currentculture#System_StringComparer_CurrentCulture)」を参照してください。 `SupportedUICultures` は、 *.resx* ファイルからのどの翻訳文字列が [ResourceManager](/dotnet/api/system.resources.resourcemanager) によって検索されるかを決定します。 `ResourceManager` は、`CurrentUICulture` によって決定されるカルチャ固有の文字列を単に検索します。 .NET のすべてのスレッドに `CurrentCulture` オブジェクトと `CurrentUICulture`オブジェクトがあります。 ASP.NET Core は、カルチャに依存する関数を表示するときに、これらの値を検査します。 たとえば、現在のスレッドのカルチャが "en-US" (英語、米国) に設定されている場合は、`DateTime.Now.ToLongDateString()` は、"Thursday, February 18, 2016" を表示しますが、`CurrentCulture` が "es-ES" (スペイン語、スペイン) に設定されている場合、出力は "jueves, 18 de febrero de 2016" になります。

## <a name="resource-files"></a>リソース ファイル

リソース ファイルは、ローカライズ可能な文字列をコードから分離するために役立つメカニズムです。 既定以外の言語の翻訳された文字列は、 *.resx* リソース ファイルで分離されています。 たとえば、翻訳された文字列を含む *Welcome.es.resx* という名前のスペイン語のリソース ファイルを作成したい場合があります。 "es" は、スペイン語の言語コードです。 Visual Studio でこのリソース ファイルを作成するには、次の手順を実行します。

1. **ソリューション エクスプローラー** で、リソース ファイルが格納されているフォルダーを右クリックし、 **[追加]** > **[新しい項目]** を選択します。

   ![入れ子になったコンテキスト メニュー:ソリューション エクスプローラーで、リソースのコンテキスト メニューが開かれます。 [追加] の 2 つ目のコンテキスト メニューが開き、[新しい項目] コマンドが強調表示されます。](localization/_static/newi.png)

1. **インストールされているテンプレートの検索** ボックスに、「resource」と入力し、ファイルに名前を付けます。

   ![[新しい項目の追加] ダイアログ](localization/_static/res.png)

1. キーの値 (ネイティブの文字列) を **[名前]** 列に入力し、翻訳された文字列を **[値]** 列に入力します。

   ![Welcome.es.resx ファイル (スペイン語の Welcome リソース ファイル) と、[名前] 列の Hello という単語と、[値] 列の Hola という単語 (スペイン語のこんにちは)](localization/_static/hola.png)

   Visual Studio に *Welcome.es.resx* ファイルが表示されます。

   ![Welcome スペイン語リソース ファイルを表示しているソリューション エクスプローラー](localization/_static/se.png)

## <a name="resource-file-naming"></a>リソース ファイルの名前付け

リソースの名前は、クラスの完全な型名からアセンブリ名を除いたものになります。 たとえば、メイン アセンブリが `LocalizationWebsite.Web.dll` であるプロジェクト内のクラス `LocalizationWebsite.Web.Startup` のフランス語のリソースは、*Startup.fr.resx* という名前になります。 クラス `LocalizationWebsite.Web.Controllers.HomeController` のリソースは、*Controllers.HomeController.fr.resx* という名前になります。 対象となるクラスの名前空間が、アセンブリ名と同じでない場合は、完全な型名が必要です。 たとえば、同じプロジェクトで、型 `ExtraNamespace.Tools` のリソースは、*ExtraNamespace.Tools.fr.resx* という名前になります。

サンプル プロジェクトで、`ConfigureServices` メソッドは `ResourcesPath` を "Resources" に設定するので、ホーム コントローラーのフランス語のりソース ファイルの相対パスは、*Resources/Controllers.HomeController.fr.resx* です。 あるいは、フォルダーを使用してリソース ファイルを整理することもできます。 Home コント ローラーのパスは、*Resources/Controllers/HomeController.fr.resx* です。 `ResourcesPath` オプションを使用しない場合、 *.resx* ファイルは、プロジェクトの基本ディレクトリに置かれます。 `HomeController` のリソース ファイルは、*Controllers.HomeController.fr.resx* という名前になります。 ドットまたはパスの名前付け規則の選択は、リソース ファイルを整理する方法によって決まります。

| リソース名 | ドットまたはパスの名前付け |
| ------------   | ------------- |
| Resources/Controllers.HomeController.fr.resx | ドット  |
| Resources/Controllers/HomeController.fr.resx  | パス |

Razor ビューの `@inject IViewLocalizer` を使用するリソース ファイルは同様のパターンに従います。 ビューのリソース ファイルには、ドットの名前付けまたはパスの名前付けを使用して名前を付けることができます。 Razor ビューのリソース ファイルは、関連するビュー ファイルのパスを模倣します。 `ResourcesPath` を "Resources" に設定すると仮定すると、*Views/Home/About.cshtml* ビューに関連付けられるフランス語のリソース ファイルは、次のいずれかになります。

* Resources/Views/Home/About.fr.resx

* Resources/Views.Home.About.fr.resx

`ResourcesPath` オプションを使用しない場合、ビューの *.resx* ファイルは、ビューと同じフォルダーに配置されます。

### <a name="rootnamespaceattribute"></a>RootNamespaceAttribute 

アセンブリのルート名前空間がアセンブリ名と異なると、<xref:Microsoft.Extensions.Localization.RootNamespaceAttribute> 属性によってアセンブリのルート名前空間が提供されます。 

> [!WARNING]
> これは、プロジェクト名が有効な .NET 識別子でない場合に発生する可能性があります。 たとえば、`my-project-name.csproj` ではルート名前空間 `my_project_name` が使用されるので、アセンブリ名 `my-project-name` はこのエラーにつながります。 

アセンブリのルート名前空間がアセンブリ名と異なる場合:

* 既定では、ローカライズが機能しません。
* アセンブリ内でのリソースの検索方法が原因で、ローカライズが失敗します。 `RootNamespace` はビルド時の値で、実行中のプロセスでは使用できません。 

`RootNamespace` が `AssemblyName` と異なる場合、*AssemblyInfo.cs* (パラメーターの値は実際の値に置き換えられます) に次を含めてください。

```csharp
using System.Reflection;
using Microsoft.Extensions.Localization;

[assembly: ResourceLocation("Resource Folder Name")]
[assembly: RootNamespace("App Root Namespace")]
```

上記のコードにより、resx ファイルが正常に解決できるようになります。

## <a name="culture-fallback-behavior"></a>カルチャ フォールバック動作

リソースを探すとき、ローカリゼーションは、"カルチャ フォールバック" を行います。 要求されたカルチャが存在しない場合、そのカルチャの親カルチャに戻ります。 なお、[CultureInfo.Parent](/dotnet/api/system.globalization.cultureinfo.parent) プロパティは親カルチャを示します。 つまり、通常は (必ずではありませんが) ISO から国の識別子が削除されます。 たとえば、メキシコで使われているスペイン語は、"es-MX" です。 これには、どの国にも固有でないスペイン語の "es" が親として付いています。

カルチャ "fr-CA" が使用された、"ようこそ" リソースの要求をサイトで受信するとします。 ローカリゼーション システムでは、次の順番でリソースを検索し、最初の一致を選択します。

* *Welcome.fr-CA.resx*
* *Welcome.fr.resx*
* *Welcome.resx* (`NeutralResourcesLanguage` が "fr-CA" の場合)

例として、".fr" カルチャ指定子を削除し、カルチャを French に設定した場合、既定のリソース ファイルは read で文字列がローカライズされます。 リソース マネージャーは、要求されたカルチャを満たすものがない場合、既定またはフォールバックのリソースを指定します。 要求されたカルチャのリソースが見つからないときにキーのみを返す場合は、既定のリソース ファイルを使用することはできません。

### <a name="generate-resource-files-with-visual-studio"></a>Visual Studio でリソース ファイルを生成する

Visual Studio で、ファイル名にカルチャを指定せずにリソース ファイルを作成する場合 (たとえば、*Welcome.resx*)、Visual Studio は各文字列のプロパティを使用して、C# クラスを作成します。 通常、ASP.NET Core ではこれを行いません。 一般的に既定の *.resx* リソース ファイル (カルチャ名のない *.resx* ファイル) は使用しません。 カルチャ名を含む *.resx* ファイル (たとえば *Welcome.fr.resx*) を作成することをお勧めします。 カルチャ名を含む *.resx* ファイルを作成すると、Visual Studio はクラス ファイルを生成しません。

### <a name="add-other-cultures"></a>その他のカルチャを追加する

各言語とカルチャの組み合わせ (既定の言語以外) ごとに一意のリソース ファイルが必要です。 ISO 言語コードがファイル名の一部となるリソース ファイル (たとえば、**en-us**、**fr-ca**、**en-gb**) を新規作成することで、異なるカルチャとロケールのリソース ファイルを作成することができます。 *Welcome.es-MX.resx* (スペイン語/メキシコ) のように、ファイル名と *.resx* ファイル拡張子の間にこれらの ISO コードが置かれます。

## <a name="implement-a-strategy-to-select-the-languageculture-for-each-request"></a>要求ごとに言語/カルチャを選択するための戦略を実装する

### <a name="configure-localization"></a>ローカリゼーションを構成する

ローカリゼーションは、`Startup.ConfigureServices` メソッドで構成されます。

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet1)]

* `AddLocalization` は、ローカリゼーション サービスをサービス コンテナーに追加します。 上記のコードは、リソース パスを "Resources" に設定します。

* `AddViewLocalization` は、ローカライズされたビュー ファイルのサポートを追加します。 このサンプル ビューでは、ローカリゼーションは、ビュー ファイルのサフィックスに基づいています。 たとえば、*Index.fr.cshtml* ファイルの "fr" です。

* `AddDataAnnotationsLocalization` は、`IStringLocalizer` 抽象化を介してローカライズされた `DataAnnotations` 検証メッセージのサポートを追加します。

### <a name="localization-middleware"></a>ローカリゼーション ミドルウェア

要求時に現在のカルチャが、ローカリゼーション [ミドルウェア](xref:fundamentals/middleware/index)で設定されます。 ローカリゼーション ミドルウェアは、`Startup.Configure` メソッドで有効になります。 要求のカルチャをチェックする可能性があるすべてのミドルウェア (たとえば `app.UseMvcWithDefaultRoute()`) の前に、ローカリゼーション ミドルウェアを構成する必要があります。

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet2)]

[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

`UseRequestLocalization` は `RequestLocalizationOptions` オブジェクトを初期化します。 すべての要求で、`RequestLocalizationOptions` の `RequestCultureProvider` のリストが列挙され、要求のカルチャを正常に決定できる最初のプロバイダーが使用されます。 既定のプロバイダーは `RequestLocalizationOptions` クラスから派生します。

1. `QueryStringRequestCultureProvider`
1. `CookieRequestCultureProvider`
1. `AcceptLanguageHeaderRequestCultureProvider`

既定のリストは、最も具体的なものから最も具体的でないものの順番になります。 記事の後半では、この順序を変更する方法、さらにカスタム カルチャ プロバイダーを追加する方法も説明します。 要求のカルチャを判断できるプロバイダーがない場合、`DefaultRequestCulture` が使用されます。

### <a name="querystringrequestcultureprovider"></a>QueryStringRequestCultureProvider

いくつかのアプリでは、クエリ文字列を使用して、<xref:System.Globalization.CultureInfo>を設定します。 cookie または Accept-language ヘッダーのアプローチを使用するアプリの場合、URL にクエリ文字列を追加すると、デバッグおよびコードのテストに役立ちます。 既定では、`QueryStringRequestCultureProvider` が、`RequestCultureProvider` リストの最初のローカリゼーション プロバイダーとして登録されます。 クエリ文字列パラメーター `culture` と `ui-culture` を渡します。 次の例では、特定のカルチャ (言語および地域) をスペイン語/メキシコに設定します。

```
http://localhost:5000/?culture=es-MX&ui-culture=es-MX
```

2 つのいずれかのみ (`culture` または `ui-culture`) を渡した場合、クエリ文字列プロバイダーは、渡した 1 つのパラメーターを使用して両方の値を設定します。 たとえば、カルチャだけを設定すると、`Culture` と `UICulture` の両方が設定されます。

```
http://localhost:5000/?culture=es-MX
```

### <a name="cookierequestcultureprovider"></a>CookieRequestCultureProvider

多くの場合、実稼働アプリケーションは、ASP.NET Core カルチャ cookie を使用してカルチャを設定するためのメカニズムを提供します。 cookie を作成するには、`MakeCookieValue` メソッドを使用します。

`CookieRequestCultureProvider` `DefaultCookieName` は、ユーザーの優先するカルチャ情報を追跡するために使用される既定の cookie 名を返します。 既定の cookie 名は `.AspNetCore.Culture` です。

cookie の形式は `c=%LANGCODE%|uic=%LANGCODE%` です。ここで、`c` は `Culture` であり、`uic` は `UICulture` です。たとえば、次のようになります。

```
c=en-UK|uic=en-US
```

カルチャ情報と UI カルチャの 1 つのみを指定した場合、指定したカルチャが、カルチャ情報と UI カルチャの両方に使用されます。

### <a name="the-accept-language-http-header"></a>Accept-Language HTTP ヘッダー

[Accept-language ヘッダー](https://www.w3.org/International/questions/qa-accept-lang-locales)は、ほとんどのブラウザーで設定可能であり、当初はユーザーの言語を指定するためのものでした。 この設定は、ブラウザーが何を送信するように設定されているか、基になるオペレーティング システムから何を継承するかを示します。 ブラウザーの要求からの Accept Language HTTP ヘッダーは、ユーザーの優先言語を検出するための確実な方法ではありません (「[Setting language preferences in a browser](https://www.w3.org/International/questions/qa-lang-priorities.en.php)」(ブラウザーの優先言語を設定する) を参照してください)。 実稼働アプリには、ユーザーがカルチャの選択をカスタマイズする方法を含める必要があります。

### <a name="set-the-accept-language-http-header-in-ie"></a>IE で Accept-Language HTTP ヘッダーを設定する

1. 歯車アイコンから、 **[インターネット オプション]** をタップします。

1. **[言語]** をタップします。

   ![インターネット オプション](localization/_static/lang.png)

1. **[言語の優先順位の設定]** をタップします。

1. **[言語の追加]** をタップします。

1. 言語を追加します。

1. 言語をタップして、 **[上へ移動]** をタップします。

### <a name="the-content-language-http-header"></a>Content-Language HTTP ヘッダー

[Content-Language](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Language) エンティティ ヘッダーは、

* 対象ユーザー用に想定した言語を説明するために使用されます。
* ユーザーが、そのユーザー独自の優先言語に従って区別することを可能にします。

エンティティ ヘッダーは、HTTP 要求と応答の両方で使用されます。

プロパティ `ApplyCurrentCultureToResponseHeaders` を設定することによって `Content-Language` ヘッダーを追加できます。

`Content-Language` ヘッダーを追加すると、

* RequestLocalizationMiddleware で `CurrentUICulture` を使って `Content-Language` を設定できるようになります。
* 応答ヘッダーの `Content-Language` を明示的に設定する必要がなくなります。

```csharp
app.UseRequestLocalization(new RequestLocalizationOptions
{
    ApplyCurrentCultureToResponseHeaders = true
});
```

### <a name="use-a-custom-provider"></a>カスタム プロバイダーを使用する

お客様がデータベースに言語とカルチャを格納できるようにしたいとします。 ユーザーのためにこれらの値を検索するプロバイダーを記述することもできます。 次のコードは、カスタム プロバイダーを追加する方法を示しています。

```csharp
private const string enUSCulture = "en-US";

services.Configure<RequestLocalizationOptions>(options =>
{
    var supportedCultures = new[]
    {
        new CultureInfo(enUSCulture),
        new CultureInfo("fr")
    };

    options.DefaultRequestCulture = new RequestCulture(culture: enUSCulture, uiCulture: enUSCulture);
    options.SupportedCultures = supportedCultures;
    options.SupportedUICultures = supportedCultures;

    options.AddInitialRequestCultureProvider(new CustomRequestCultureProvider(async context =>
    {
        // My custom request culture logic
        return new ProviderCultureResult("en");
    }));
});
```

ローカリゼーション プロバイダーを追加または削除するには、`RequestLocalizationOptions` を使用します。

### <a name="set-the-culture-programmatically"></a>プログラムでカルチャを設定する

この [GitHub](https://github.com/aspnet/entropy) のサンプル **Localization.StarterWeb** プロジェクトには、`Culture` を設定するための UI が含まれています。 *Views/Shared/_SelectLanguagePartial.cshtml* ファイルを使用して、サポートされているカルチャの一覧からカルチャを選択することができます。

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_SelectLanguagePartial.cshtml)]

*Views/Shared/_SelectLanguagePartial.cshtml* ファイルは、レイアウト ファイルの `footer` セクションに追加されるので、すべてのビューで使用できます。

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_Layout.cshtml?range=43-56&highlight=10)]

`SetLanguage` メソッドはカルチャ cookie を設定します。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/HomeController.cs?range=57-67)]

*_SelectLanguagePartial.cshtml* をこのプロジェクトのサンプル コードに接続することはできません。 [GitHub](https://github.com/aspnet/entropy) の **Localization.StarterWeb** プロジェクトには、[依存関係の挿入](dependency-injection.md)コンテナーを介して Razor 部分に `RequestLocalizationOptions` をフローするコードがあります。

## <a name="model-binding-route-data-and-query-strings"></a>モデル バインド ルート データとクエリ文字列

「[モデル バインド ルート データとクエリ文字列のグローバリゼーション動作](xref:mvc/models/model-binding#glob)」を参照してください。

## <a name="globalization-and-localization-terms"></a>グローバリゼーションとローカリゼーションの用語

アプリをローカライズするプロセスでは、最新のソフトウェア開発でよく使用される関連する文字セットの基本的な理解と、それらに関連した問題の理解も必要です。 すべてのコンピューターは、テキストを数値 (コード) として格納しますが、さまざまなシステムが異なる数値を使用して同じテキストを格納します。 ローカリゼーション プロセスとは、特定のカルチャ/ロケール用にアプリのユーザー インターフェイス (UI) を変換することを指します。

[ローカライズ化](/dotnet/standard/globalization-localization/localizability-review)は、グローバル化されたアプリのローカリゼーションの準備ができていることを確認するための中間プロセスです。

カルチャ名の [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) 形式は `<languagecode2>-<country/regioncode2>` です。ここで、`<languagecode2>` は言語コードであり、`<country/regioncode2>` はサブカルチャ コードです。 たとえば、スペイン語 (チリ) は `es-CL`、英語 (米国) は `en-US`、英語 (オーストラリア) は `en-AU` です。 [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) は、言語に関連付けられた ISO 639 の 2 文字の小文字カルチャ コードと、国または地域に関連付けられた ISO 3166 の 2 文字の大文字サブカルチャ コードの組み合わせです。 以下を参照してください。<https://docs.microsoft.com/previous-versions/commerce-server/ee825488(v=cs.20)>

多くの場合、国際化は "I18N" に省略されます。 省略形は、最初と最後の文字およびそれらの間の文字の数になります。したがって、18 は、最初の "I" と最後の "N" の間の文字の数を表します。 同じことが、グローバリゼーション (G11N) とローカリゼーション (L10N) にも当てはまります。

用語:

* グローバリゼーション (G11N):アプリが別の言語と地域をサポートするようにするプロセス。
* ローカリゼーション (L10N):特定の言語と地域向けにアプリをカスタマイズするプロセス。
* 国際化 (I18N):グローバリゼーションとローカリゼーションの両方を示します。
* カルチャ:言語および必要に応じて地域です。
* ニュートラル カルチャ:指定した言語のみを含み、地域は含まないカルチャ。 (例: "en"、"es")。
* 特定のカルチャ:指定した言語と地域を含むカルチャ。 (例: "en-US"、"en-GB"、"es-CL")。
* 親カルチャ:特定のカルチャを含むニュートラル カルチャ。 (たとえば、"en" は "en-US" および "en-GB" の親カルチャです)。
* ロケール:ロケールはカルチャと同じです。

[!INCLUDE[](~/includes/localization/currency.md)]

[!INCLUDE[](~/includes/localization/unsupported-culture-log-level.md)]

## <a name="additional-resources"></a>その他の技術情報

* <xref:fundamentals/troubleshoot-aspnet-core-localization>
* 記事で使用されている [Localization.StarterWeb プロジェクト](https://github.com/aspnet/Entropy/tree/master/samples/Localization.StarterWeb)。
* [.NET アプリケーションのグローバライズとローカライズ](/dotnet/standard/globalization-localization/index)
* [.resx ファイル内のリソース](/dotnet/framework/resources/working-with-resx-files-programmatically)
* [Microsoft 多言語アプリ ツールキット](https://marketplace.visualstudio.com/items?itemName=MultilingualAppToolkit.MultilingualAppToolkit-18308)
* [ローカリゼーションとジェネリック](http://hishambinateya.com/localization-and-generics)

::: moniker-end
