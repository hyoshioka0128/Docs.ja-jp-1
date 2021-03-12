---
title: ASP.NET Core で Portable Object のローカライズを構成する
author: sebastienros
description: この記事では、Portable Object (PO) ファイルについて紹介します。また、Orchard Core フレームワークを使用する ASP.NET Core アプリケーションで PO ファイルを使用する手順について説明します。
ms.author: scaddie
ms.date: 09/26/2017
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
uid: fundamentals/portable-object-localization
ms.openlocfilehash: ed41657b364c7f845491b3e452db8a4d5c5aa389
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587178"
---
# <a name="configure-portable-object-localization-in-aspnet-core"></a>ASP.NET Core で Portable Object のローカライズを構成する

::: moniker range=">= aspnetcore-3.0"

作成者: [Sébastien Ros](https://github.com/sebastienros)、[Scott Addie](https://twitter.com/Scott_Addie)、[Hisham Bin Ateya](https://github.com/hishamco)

この記事では、[Orchard Core](https://github.com/OrchardCMS/OrchardCore) フレームワークを使用した ASP.NET Core アプリケーションで Portable Object (PO) ファイルを使用する手順について説明します。

**注:** Orchard Core は Microsoft 製品ではありません。 したがって、Microsoft はこの機能をサポートしていません。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/localization/sample/3.x/POLocalization)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="what-is-a-po-file"></a>PO ファイルとは

PO ファイルは、特定の言語の翻訳済み文字列を含むテキスト ファイルとして配布されます。 *.resx* ファイルの代わりに PO ファイルを使用すると、次のような利点があります。
- PO ファイルは複数形化をサポートしています。 *.resx* ファイルは複数形化をサポートしていません。
- PO ファイルは *.resx* ファイルのようにコンパイルされません。 そのため、特殊なツールやビルドの手順は必要ありません。
- PO ファイルは、オンラインの共同編集ツールでの利用に適しています。

### <a name="example"></a>例

フランス語の 2 つの文字列 (一方は複数形も含まれています) の翻訳が記載されたサンプル PO ファイルを次に示します。

*fr.po*

```text
#: Services/EmailService.cs:29
msgid "Enter a comma separated list of email addresses."
msgstr "Entrez une liste d'emails séparés par une virgule."

#: Views/Email.cshtml:112
msgid "The email address is \"{0}\"."
msgid_plural "The email addresses are \"{0}\"."
msgstr[0] "L'adresse email est \"{0}\"."
msgstr[1] "Les adresses email sont \"{0}\""
```

この例では、次の構文を使用します。

- `#:`:翻訳される文字列のコンテキストを示すコメント。 同じ文字列でも、使用される場所によって翻訳が変わることがあります。
- `msgid`:翻訳前の文字列。
- `msgstr`:翻訳後の文字列。

複数形化をサポートする場合は、その他のエントリも定義できます。

- `msgid_plural`:翻訳前の文字列の複数形。
- `msgstr[0]`:0 の場合の翻訳後の文字列。
- `msgstr[N]`:N の場合の翻訳後の文字列。

PO ファイルの仕様については、[こちら](https://www.gnu.org/savannah-checkouts/gnu/gettext/manual/html_node/PO-Files.html)を参照してください。

## <a name="configuring-po-file-support-in-aspnet-core"></a>ASP.NET Core で PO ファイルのサポートを構成する

この例は、Visual Studio 2017 プロジェクト テンプレートから生成された ASP.NET Core MVC アプリケーションに基づいています。

### <a name="referencing-the-package"></a>パッケージの参照

`OrchardCore.Localization.Core` NuGet パッケージの参照を追加します。

*.csproj* ファイルに次のような行が追加されました (バージョン番号は異なる可能性があります)。

[!code-xml[](localization/sample/3.x/POLocalization/POLocalization.csproj?range=8)]

### <a name="registering-the-service"></a>サービスの登録

*Startup.cs* の `ConfigureServices` メソッドに必要なサービスを追加します。

[!code-csharp[](localization/sample/3.x/POLocalization/Startup.cs?name=snippet_ConfigureServices&highlight=4-21)]

*Startup.cs* の `Configure` メソッドに必要なミドルウェアを追加します。

[!code-csharp[](localization/sample/3.x/POLocalization/Startup.cs?name=snippet_Configure&highlight=15)]

選択した Razor ビューに次のコードを追加します。 この例では *About.cshtml* を使用します。

[!code-cshtml[](localization/sample/3.x/POLocalization/Views/Home/About.cshtml)]

"Hello world!" というテキストを翻訳するために、`IViewLocalizer` インスタンスが挿入され、使用されます。

### <a name="creating-a-po-file"></a>PO ファイルの作成

アプリケーションのルート フォルダーに *\<culture code>.po* というファイルを作成します。 この例では、フランス語が使用されているため、ファイル名は *fr.po* です。

[!code-text[](localization/sample/3.x/POLocalization/fr.po)]

このファイルには、翻訳する文字列とフランス語で翻訳された文字列の両方が格納されています。 必要に応じて、翻訳は元の親カルチャに戻されます。 この例では、要求されるカルチャが `fr-FR` または `fr-CA` の場合、*fr.po* ファイルが使用されます。

### <a name="testing-the-application"></a>アプリケーションのテスト

アプリケーションを実行し、URL `/Home/About` に移動します。 テキスト **Hello world!** が表示されます。

URL `/Home/About?culture=fr-FR` に移動します。 テキスト **Bonjour le monde!** が表示されます。

## <a name="pluralization"></a>複数形化

PO ファイルは複数形化フォームをサポートしています。これは、カーディナリティに基づいて同じ文字列の翻訳を変える必要がある場合に便利です。 各言語で、カーディナリティに基づいて使用する文字列を選択するためのカスタム ルールが定義されているため、このタスクは複雑になります。

Orchard Localization パッケージには、このような異なる複数形を自動的に呼び出す API が用意されています。

### <a name="creating-pluralization-po-files"></a>複数形化 PO ファイルの作成

前述の *fr.po* ファイルに次の内容を追加します。

```text
msgid "There is one item."
msgid_plural "There are {0} items."
msgstr[0] "Il y a un élément."
msgstr[1] "Il y a {0} éléments."
```

この例の各エントリが表す内容の説明については、「[PO ファイルとは](#what-is-a-po-file)」を参照してください。

### <a name="adding-a-language-using-different-pluralization-forms"></a>異なる複数形化フォームを使用する言語の追加

前の例では、英語とフランス語の文字列が使用されていました。 英語とフランス語の複数形化フォームは 2 つのみであり、同じフォーム ルールを共有しています。つまり、1 のカーディナリティは最初の複数形にマップされます。 その他のカーディナリティは 2 番目の複数形にマップされます。

すべての言語が同じルールを共有するわけではありません。 たとえば、チェコ語の場合は 3 つの複数形があります。

`cs.po` ファイルを次のように作成し、複数形化に 3 つの異なる翻訳がどのように必要かを記述します。

[!code-text[](localization/sample/3.x/POLocalization/cs.po)]

チェコ語のローカライズを引き受ける場合は、`ConfigureServices` メソッドでサポートされるカルチャのリストに `"cs"` を追加します。

```csharp
var supportedCultures = new List<CultureInfo>
{
    new CultureInfo("en-US"),
    new CultureInfo("en"),
    new CultureInfo("fr-FR"),
    new CultureInfo("fr"),
    new CultureInfo("cs")
};
```

複数の基数についてローカライズされた複数形の文字列をレンダリングするように、*Views/Home/About.cshtml* ファイルを編集します。

```cshtml
<p>@Localizer.Plural(1, "There is one item.", "There are {0} items.")</p>
<p>@Localizer.Plural(2, "There is one item.", "There are {0} items.")</p>
<p>@Localizer.Plural(5, "There is one item.", "There are {0} items.")</p>
```

**注:** 実際のシナリオでは、変数を使用してカウントを表します。 ここでは、非常に特殊なケースを表すために、3 つの異なる値を使用して同じコードを繰り返しています。

カルチャを切り替えると、次のように表示されます。

`/Home/About`の場合:

```html
There is one item.
There are 2 items.
There are 5 items.
```

`/Home/About?culture=fr`の場合:

```html
Il y a un élément.
Il y a 2 éléments.
Il y a 5 éléments.
```

`/Home/About?culture=cs`の場合:

```html
Existuje jedna položka.
Existují 2 položky.
Existuje 5 položek.
```

チェコ語のカルチャの場合、3 つの翻訳が異なることに注意してください。 フランス語と英語のカルチャは、翻訳後の文字列のうち、後半 2 つは同じ構造を共有しています。

## <a name="advanced-tasks"></a>高度なタスク

### <a name="contextualizing-strings"></a>文字列のコンテキスト化

多くの場合、アプリケーションには複数の場所で翻訳される文字列が含まれています。 同じ文字列でも、アプリ内の場所 (Razor ビューやクラス ファイル) によっては翻訳が異なる場合があります。 PO ファイルは、表現されている文字列の分類に使用できるファイル コンテキストの概念をサポートしています。 ファイル コンテキストを使用すると、ファイル コンテキスト (またはファイル コンテキストの欠如) に応じて文字列の翻訳を変えることができます。

PO ローカライズ サービスは、文字列を翻訳するときに使用される完全クラスまたはビューの名前を使用します。 これは、`msgctxt` エントリに値を設定することによって行います。

前述の *fr.po* の例に少し追加してみましょう。 *Views/Home/About.cshtml* にある Razor ビューは、予約された `msgctxt` エントリの値を設定することでファイル コンテキストとして定義できます。

```text
msgctxt "Views.Home.About"
msgid "Hello world!"
msgstr "Bonjour le monde!"
```

`msgctxt` をこのように設定すると、`/Home/About?culture=fr-FR` に移動するときにテキストの翻訳が行われます。 `/Home/Contact?culture=fr-FR` に移動するときに翻訳は行われません。

特定のエントリが特定のファイル コンテキストと一致しない場合、Orchard Core のフォールバック メカニズムによって、コンテキストなしで適切な PO ファイルが検索されます。 *Views/Home/Contact.cshtml* に特定のファイル コンテキストが定義されていない場合、`/Home/Contact?culture=fr-FR` に移動すると、次のような PO ファイルが読み込まれます。

[!code-text[](localization/sample/3.x/POLocalization/fr.po)]

### <a name="changing-the-location-of-po-files"></a>PO ファイルの場所の変更

PO ファイルの既定の場所は、`ConfigureServices` で変更できます。

```csharp
services.AddPortableObjectLocalization(options => options.ResourcesPath = "Localization");
```

この例では、PO ファイルは *Localization* フォルダーから読み込まれます。

### <a name="implementing-a-custom-logic-for-finding-localization-files"></a>ローカライズ ファイルを検索するためのカスタム ロジックの実装

PO ファイルを特定するためにより複雑なロジックが必要な場合は、`OrchardCore.Localization.PortableObject.ILocalizationFileLocationProvider` インターフェイスを実装してサービスとして登録することができます。 これは、PO ファイルをさまざまな場所に格納できる場合や、複数のフォルダーがある 1 階層内でファイルを検出する必要がある場合に便利です。

### <a name="using-a-different-default-pluralized-language"></a>異なる既定の複数形化された言語の使用

このパッケージには、2 つの複数形に固有の `Plural` 拡張メソッドが含まれています。 より多くの複数形が必要な言語の場合は、拡張メソッドを作成します。 拡張メソッドを使用すると、既定の言語用にローカライズ ファイルを用意する必要はありません &mdash; 元の文字列はコード内でそのまま使用できます。

翻訳の文字列配列を受け付ける、より汎用的な `Plural(int count, string[] pluralForms, params object[] arguments)` オーバーロードを使用することができます。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作成者: [Sébastien Ros](https://github.com/sebastienros)、[Scott Addie](https://twitter.com/Scott_Addie)

この記事では、[Orchard Core](https://github.com/OrchardCMS/OrchardCore) フレームワークを使用した ASP.NET Core アプリケーションで Portable Object (PO) ファイルを使用する手順について説明します。

**注:** Orchard Core は Microsoft 製品ではありません。 したがって、Microsoft はこの機能をサポートしていません。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/localization/sample/2.x/POLocalization)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="what-is-a-po-file"></a>PO ファイルとは

PO ファイルは、特定の言語の翻訳済み文字列を含むテキスト ファイルとして配布されます。 *.resx* ファイルの代わりに PO ファイルを使用すると、次のような利点があります。
- PO ファイルは複数形化をサポートしています。 *.resx* ファイルは複数形化をサポートしていません。
- PO ファイルは *.resx* ファイルのようにコンパイルされません。 そのため、特殊なツールやビルドの手順は必要ありません。
- PO ファイルは、オンラインの共同編集ツールでの利用に適しています。

### <a name="example"></a>例

フランス語の 2 つの文字列 (一方は複数形も含まれています) の翻訳が記載されたサンプル PO ファイルを次に示します。

*fr.po*

```text
#: Services/EmailService.cs:29
msgid "Enter a comma separated list of email addresses."
msgstr "Entrez une liste d'emails séparés par une virgule."

#: Views/Email.cshtml:112
msgid "The email address is \"{0}\"."
msgid_plural "The email addresses are \"{0}\"."
msgstr[0] "L'adresse email est \"{0}\"."
msgstr[1] "Les adresses email sont \"{0}\""
```

この例では、次の構文を使用します。

- `#:`:翻訳される文字列のコンテキストを示すコメント。 同じ文字列でも、使用される場所によって翻訳が変わることがあります。
- `msgid`:翻訳前の文字列。
- `msgstr`:翻訳後の文字列。

複数形化をサポートする場合は、その他のエントリも定義できます。

- `msgid_plural`:翻訳前の文字列の複数形。
- `msgstr[0]`:0 の場合の翻訳後の文字列。
- `msgstr[N]`:N の場合の翻訳後の文字列。

PO ファイルの仕様については、[こちら](https://www.gnu.org/savannah-checkouts/gnu/gettext/manual/html_node/PO-Files.html)を参照してください。

## <a name="configuring-po-file-support-in-aspnet-core"></a>ASP.NET Core で PO ファイルのサポートを構成する

この例は、Visual Studio 2017 プロジェクト テンプレートから生成された ASP.NET Core MVC アプリケーションに基づいています。

### <a name="referencing-the-package"></a>パッケージの参照

`OrchardCore.Localization.Core` NuGet パッケージの参照を追加します。

*.csproj* ファイルに次のような行が追加されました (バージョン番号は異なる可能性があります)。

[!code-xml[](localization/sample/2.x/POLocalization/POLocalization.csproj?range=9)]

### <a name="registering-the-service"></a>サービスの登録

*Startup.cs* の `ConfigureServices` メソッドに必要なサービスを追加します。

[!code-csharp[](localization/sample/2.x/POLocalization/Startup.cs?name=snippet_ConfigureServices&highlight=4-21)]

*Startup.cs* の `Configure` メソッドに必要なミドルウェアを追加します。

[!code-csharp[](localization/sample/2.x/POLocalization/Startup.cs?name=snippet_Configure&highlight=15)]

選択した Razor ビューに次のコードを追加します。 この例では *About.cshtml* を使用します。

[!code-cshtml[](localization/sample/2.x/POLocalization/Views/Home/About.cshtml)]

"Hello world!" というテキストを翻訳するために、`IViewLocalizer` インスタンスが挿入され、使用されます。

### <a name="creating-a-po-file"></a>PO ファイルの作成

アプリケーションのルート フォルダーに *\<culture code>.po* というファイルを作成します。 この例では、フランス語が使用されているため、ファイル名は *fr.po* です。

[!code-text[](localization/sample/2.x/POLocalization/fr.po)]

このファイルには、翻訳する文字列とフランス語で翻訳された文字列の両方が格納されています。 必要に応じて、翻訳は元の親カルチャに戻されます。 この例では、要求されるカルチャが `fr-FR` または `fr-CA` の場合、*fr.po* ファイルが使用されます。

### <a name="testing-the-application"></a>アプリケーションのテスト

アプリケーションを実行し、URL `/Home/About` に移動します。 テキスト **Hello world!** が表示されます。

URL `/Home/About?culture=fr-FR` に移動します。 テキスト **Bonjour le monde!** が表示されます。

## <a name="pluralization"></a>複数形化

PO ファイルは複数形化フォームをサポートしています。これは、カーディナリティに基づいて同じ文字列の翻訳を変える必要がある場合に便利です。 各言語で、カーディナリティに基づいて使用する文字列を選択するためのカスタム ルールが定義されているため、このタスクは複雑になります。

Orchard Localization パッケージには、このような異なる複数形を自動的に呼び出す API が用意されています。

### <a name="creating-pluralization-po-files"></a>複数形化 PO ファイルの作成

前述の *fr.po* ファイルに次の内容を追加します。

```text
msgid "There is one item."
msgid_plural "There are {0} items."
msgstr[0] "Il y a un élément."
msgstr[1] "Il y a {0} éléments."
```

この例の各エントリが表す内容の説明については、「[PO ファイルとは](#what-is-a-po-file)」を参照してください。

### <a name="adding-a-language-using-different-pluralization-forms"></a>異なる複数形化フォームを使用する言語の追加

前の例では、英語とフランス語の文字列が使用されていました。 英語とフランス語の複数形化フォームは 2 つのみであり、同じフォーム ルールを共有しています。つまり、1 のカーディナリティは最初の複数形にマップされます。 その他のカーディナリティは 2 番目の複数形にマップされます。

すべての言語が同じルールを共有するわけではありません。 たとえば、チェコ語の場合は 3 つの複数形があります。

`cs.po` ファイルを次のように作成し、複数形化に 3 つの異なる翻訳がどのように必要かを記述します。

[!code-text[](localization/sample/2.x/POLocalization/cs.po)]

チェコ語のローカライズを引き受ける場合は、`ConfigureServices` メソッドでサポートされるカルチャのリストに `"cs"` を追加します。

```csharp
var supportedCultures = new List<CultureInfo>
{
    new CultureInfo("en-US"),
    new CultureInfo("en"),
    new CultureInfo("fr-FR"),
    new CultureInfo("fr"),
    new CultureInfo("cs")
};
```

複数の基数についてローカライズされた複数形の文字列をレンダリングするように、*Views/Home/About.cshtml* ファイルを編集します。

```cshtml
<p>@Localizer.Plural(1, "There is one item.", "There are {0} items.")</p>
<p>@Localizer.Plural(2, "There is one item.", "There are {0} items.")</p>
<p>@Localizer.Plural(5, "There is one item.", "There are {0} items.")</p>
```

**注:** 実際のシナリオでは、変数を使用してカウントを表します。 ここでは、非常に特殊なケースを表すために、3 つの異なる値を使用して同じコードを繰り返しています。

カルチャを切り替えると、次のように表示されます。

`/Home/About`の場合:

```html
There is one item.
There are 2 items.
There are 5 items.
```

`/Home/About?culture=fr`の場合:

```html
Il y a un élément.
Il y a 2 éléments.
Il y a 5 éléments.
```

`/Home/About?culture=cs`の場合:

```html
Existuje jedna položka.
Existují 2 položky.
Existuje 5 položek.
```

チェコ語のカルチャの場合、3 つの翻訳が異なることに注意してください。 フランス語と英語のカルチャは、翻訳後の文字列のうち、後半 2 つは同じ構造を共有しています。

## <a name="advanced-tasks"></a>高度なタスク

### <a name="contextualizing-strings"></a>文字列のコンテキスト化

多くの場合、アプリケーションには複数の場所で翻訳される文字列が含まれています。 同じ文字列でも、アプリ内の場所 (Razor ビューやクラス ファイル) によっては翻訳が異なる場合があります。 PO ファイルは、表現されている文字列の分類に使用できるファイル コンテキストの概念をサポートしています。 ファイル コンテキストを使用すると、ファイル コンテキスト (またはファイル コンテキストの欠如) に応じて文字列の翻訳を変えることができます。

PO ローカライズ サービスは、文字列を翻訳するときに使用される完全クラスまたはビューの名前を使用します。 これは、`msgctxt` エントリに値を設定することによって行います。

前述の *fr.po* の例に少し追加してみましょう。 *Views/Home/About.cshtml* にある Razor ビューは、予約された `msgctxt` エントリの値を設定することでファイル コンテキストとして定義できます。

```text
msgctxt "Views.Home.About"
msgid "Hello world!"
msgstr "Bonjour le monde!"
```

`msgctxt` をこのように設定すると、`/Home/About?culture=fr-FR` に移動するときにテキストの翻訳が行われます。 `/Home/Contact?culture=fr-FR` に移動するときに翻訳は行われません。

特定のエントリが特定のファイル コンテキストと一致しない場合、Orchard Core のフォールバック メカニズムによって、コンテキストなしで適切な PO ファイルが検索されます。 *Views/Home/Contact.cshtml* に特定のファイル コンテキストが定義されていない場合、`/Home/Contact?culture=fr-FR` に移動すると、次のような PO ファイルが読み込まれます。

[!code-text[](localization/sample/2.x/POLocalization/fr.po)]

### <a name="changing-the-location-of-po-files"></a>PO ファイルの場所の変更

PO ファイルの既定の場所は、`ConfigureServices` で変更できます。

```csharp
services.AddPortableObjectLocalization(options => options.ResourcesPath = "Localization");
```

この例では、PO ファイルは *Localization* フォルダーから読み込まれます。

### <a name="implementing-a-custom-logic-for-finding-localization-files"></a>ローカライズ ファイルを検索するためのカスタム ロジックの実装

PO ファイルを特定するためにより複雑なロジックが必要な場合は、`OrchardCore.Localization.PortableObject.ILocalizationFileLocationProvider` インターフェイスを実装してサービスとして登録することができます。 これは、PO ファイルをさまざまな場所に格納できる場合や、複数のフォルダーがある 1 階層内でファイルを検出する必要がある場合に便利です。

### <a name="using-a-different-default-pluralized-language"></a>異なる既定の複数形化された言語の使用

このパッケージには、2 つの複数形に固有の `Plural` 拡張メソッドが含まれています。 より多くの複数形が必要な言語の場合は、拡張メソッドを作成します。 拡張メソッドを使用すると、既定の言語用にローカライズ ファイルを用意する必要はありません &mdash; 元の文字列はコード内でそのまま使用できます。

翻訳の文字列配列を受け付ける、より汎用的な `Plural(int count, string[] pluralForms, params object[] arguments)` オーバーロードを使用することができます。

::: moniker-end
