---
title: ASP.NET Core のタグ ヘルパー作成
author: rick-anderson
description: ASP.NET Core でのタグ ヘルパーの作成方法を説明します。
ms.author: riande
ms.custom: mvc
ms.date: 12/05/2019
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
uid: mvc/views/tag-helpers/authoring
ms.openlocfilehash: e0c9f450e4eded49694cbbb0e9fa2614a221ab14
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586827"
---
# <a name="author-tag-helpers-in-aspnet-core"></a>ASP.NET Core のタグ ヘルパー作成

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/tag-helpers/authoring/sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="get-started-with-tag-helpers"></a>タグ ヘルパーの概要

このチュートリアルでは、タグ ヘルパーのプログラミングの概要を提供します。 [タグ ヘルパーの概要](intro.md)に関するページでは、タグ ヘルパーが提供する利点について説明しています。

タグ ヘルパーは、`ITagHelper` インターフェイス実装する任意のクラスです。 ただし、タグ ヘルパーを作成する場合は、通常は `TagHelper` から派生させます。こうすることで、`Process` メソッドにアクセスできるようになります。

1. **AuthoringTagHelpers** という新しい ASP.NET Core プロジェクトを作成します。 このプロジェクトに認証は必要ありません。

1. タグ ヘルパーを保持する *TagHelpers* というフォルダーを作成します。 *TagHelpers* フォルダーは必須では *ありません* が、あると便利です。 それでは、いくつか単純なタグ ヘルパーを記述してみましょう。

## <a name="a-minimal-tag-helper"></a>最小のタグ ヘルパー

このセクションでは、電子メール タグを更新するタグ ヘルパーを記述します。 次に例を示します。

```html
<email>Support</email>
```

サーバーは、電子メール タグ ヘルパーを使用して、そのマークアップを以下に変換します。

```html
<a href="mailto:Support@contoso.com">Support@contoso.com</a>
```

つまり、これを電子メール リンクにするアンカー タグです。 ブログ エンジンを記述していて、マーケティング、サポート、およびその他の連絡先の電子メールをすべて同じドメインに送信するためにそれが必要な場合に、これを行うことができます。

1. 次の `EmailTagHelper` クラスを *TagHelpers* フォルダーに追加します。

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1EmailTagHelperCopy.cs)]

   * タグ ヘルパーは、ルート クラス名の要素 (クラス名部分から *TagHelper* 部分を引いたもの) をターゲットとする名前付け規則です。 この例では、 **Emailtaghelper** のルート名は *email* であるため、タグは対象になり `<email>` ます。 この命名規則は、ほとんどのタグ ヘルパーで機能します。オーバーライドの方法については、後述します。

   * `EmailTagHelper` クラスは、`TagHelper` から派生したものです。 `TagHelper` クラスはタグ ヘルパーを記述するためのメソッドとプロパティを提供します。

   * オーバーライドされた `Process` メソッドは、実行時のタグ ヘルパーの動作を制御します。 `TagHelper` クラスには、同じパラメーターを使用する非同期バージョン (`ProcessAsync`) も用意されています。

   * `Process` (および `ProcessAsync`) のコンテキスト パラメーターには、現在の HTML タグの実行に関連付けられている情報が含まれています。

   * `Process` (および `ProcessAsync`) の出力パラメーターには、HTML タグとコンテンツの生成に使用された元のソースを代表するステートフル HTML 要素が含まれています。

   * クラス名には **TagHelper** のサフィックスが付けられています。これは必須 *ではありません* が、ベスト プラクティスです。 クラスは、次のように宣言できます。

   ```csharp
   public class Email : TagHelper
   ```

1. クラスをすべてのビューで使用できるようにするには `EmailTagHelper` Razor 、 `addTagHelper` ディレクティブを *views/_ViewImports cshtml* ファイルに追加します。

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/_ViewImportsCopyEmail.cshtml?highlight=2,3)]

   上記のコードでは、ワイルドカードの構文を使用して、アセンブリ内のすべてのタグ ヘルパーが使用可能になるように指定しています。 `@addTagHelper` の後の最初の文字列は、読み込むタグ ヘルパーを指定します (すべてのタグ ヘルパーを指定するには、"*" を使用します)。2 番目の文字列 "AuthoringTagHelpers" は、タグ ヘルパーが存在するアセンブリを指定します。 また、2番目の行は、ワイルドカード構文を使用して ASP.NET Core MVC タグヘルパーを取り込むことに注意してください (これらのヘルパーについては、「 [タグヘルパーの概要](intro.md)」で説明されています)。これは、 `@addTagHelper` タグヘルパーをビューで使用できるようにするディレクティブです Razor 。 または、次に示すように、タグ ヘルパーの完全修飾名 (FQN) を指定することもできます。

```csharp
@using AuthoringTagHelpers
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper AuthoringTagHelpers.TagHelpers.EmailTagHelper, AuthoringTagHelpers
```

<!--
the following snippet uses TagHelpers3 and should use TagHelpers (not the 3)
    [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/_ViewImports.cshtml?highlight=3&range=1-3)]
-->

FQN を使用してタグ ヘルパーをビューに追加するには、最初に FQN (`AuthoringTagHelpers.TagHelpers.EmailTagHelper`) を追加してから、**アセンブリ名** (*AuthoringTagHelpers*。`namespace` は不要) を追加します。 ほとんどの開発者は、ワイルドカードの構文を使用する方を選びます。 [タグ ヘルパーの概要](intro.md)に関するページでは、タグ ヘルパーの追加、削除、階層、およびワイルドカードの構文について詳しく説明されています。

1. これらの変更を加えて、*Views/Home/Contact.cshtml* ファイル内のマークアップを更新します。

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/Contact.cshtml?highlight=15,16&range=1-17)]

1. アプリを実行して、お好みのブラウザーを使用して HTML ソースを表示すると、電子メール タグがアンカー マークアップ (`<a>Support</a>` など) に置き換えられていることが確認できます。 *Support* と *Marketing* はリンクとして表示されますが、リンクを機能させる `href` 属性を持っていません。 次のセクションでこれを修正します。

## <a name="setattribute-and-setcontent"></a>SetAttribute と SetContent

このセクションでは、`EmailTagHelper` を更新して、電子メール用の有効なアンカー タグが作成できるようにします。 これを更新して、 Razor (属性の形式で) ビューから情報を取得 `mail-to` し、アンカーの生成に使用します。

次のように `EmailTagHelper` クラスを更新します。

[!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/EmailTagHelperMailTo.cs?range=6-22)]

* タグ ヘルパーのパスカルケースのクラス名とプロパティ名は、それぞれ[ケバブ ケース](https://stackoverflow.com/questions/11273282/whats-the-name-for-dash-separated-case/12273101)に変換されます。 そのため、`MailTo` 属性を使用するには、相当する `<email mail-to="value"/>` を使用します。

* 最後の行は、最低限機能するタグ ヘルパーの完成したコンテンツを設定します。

* 強調表示されている行は、属性を追加するための構文を示しています。

[!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/EmailTagHelperMailTo.cs?highlight=6&range=14-21)]

この方法は、属性 "href" が属性コレクションに現存していない場合に限り、属性 "href" に対して有効です。 `output.Attributes.Add` メソッドを使用してタグ ヘルパー属性をタグ属性のコレクションの末尾に追加することもできます。

1. これらの変更を加えて、*Views/Home/Contact.cshtml* ファイル内のマークアップを更新します。

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/ContactCopy.cshtml?highlight=15,16)]

1. アプリを実行し、適切なリンクが生成されることを確認します。

<a name="self-closing"></a>

   > [!NOTE]
   > 自己終了の電子メール タグ (`<email mail-to="Rick" />`) を記述すると、最終的な出力も自己終了になります。 開始タグのみを持つタグ (`<email mail-to="Rick">`) を記述する機能を有効にするには、次のようにクラスをマークする必要があります。
   >
   > [!code-csharp[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/EmailTagHelperMailVoid.cs?highlight=1&range=6-10)]

   自己終了の電子メール タグ ヘルパーでは、出力は `<a href="mailto:Rick@contoso.com" />` になります。 自己終了のアンカー タグは有効な HTML ではないため、作成することはないかもしれませんが、自己終了のタグ ヘルパーは必要な場合があります。 タグ ヘルパーは、タグを読み取った後に、`TagMode` プロパティの型を設定します。

### <a name="processasync"></a>ProcessAsync

このセクションでは非同期の電子メール ヘルパーを記述します。

1. `EmailTagHelper` クラスを次のコードで置き換えます。

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/EmailTagHelper.cs?range=6-17)]

   **注:**

   * このバージョンでは、非同期の `ProcessAsync` メソッドを使用します。 非同期の `GetChildContentAsync` は `TagHelperContent` を含む `Task` を返します。

   * `output` パラメーターを使用して、HTML 要素のコンテンツを取得します。

1. *Views/Home/Contact.cshtml* ファイルに次の変更を行って、タグ ヘルパーがターゲットの電子メールを取得できるようにします。

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/Contact.cshtml?highlight=15,16&range=1-17)]

1. アプリを実行し、有効な電子メール リンクが生成されることを確認します。

### <a name="removeall-precontentsethtmlcontent-and-postcontentsethtmlcontent"></a>RemoveAll、PreContent.SetHtmlContent、PostContent.SetHtmlContent

1. 次の `BoldTagHelper` クラスを *TagHelpers* フォルダーに追加します。

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/BoldTagHelper.cs)]

   * `[HtmlTargetElement]` 属性は、"bold" という名前の HTML 属性を含むすべての HTML 要素を照合することを指定する属性パラメーターを渡し、クラス内で `Process` オーバーライド メソッドが実行されます。 このサンプルでは、`Process` メソッドによって "bold" 属性が削除され、中にあるマークアップが `<strong></strong>` で囲まれます。

   * 既存のタグ コンテンツは置き換えたくないので、`PreContent.SetHtmlContent` メソッドを使用して開始タグ `<strong>` を記述し、`PostContent.SetHtmlContent` メソッドを使用して終了タグ `</strong>` を記述する必要があります。

1. `bold` 属性値を含めるように *About.cshtml* ビューを変更します。 完成したコードを以下に示します。

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/AboutBoldOnly.cshtml?highlight=7)]

1. アプリを実行します。 お好みのブラウザーを使用して、ソースを検査し、マークアップを確認できます。

   上記の `[HtmlTargetElement]` 属性は、"bold" という属性名を提供する HTML マークアップのみをターゲットにしています。 `<bold>` 要素は、タグ ヘルパーによって変更されませんでした。

1. `[HtmlTargetElement]` 属性の行をコメント アウトすると、既定の `<bold>` タグ (`<bold>` 形式の HTML マークアップ) が再度ターゲットになります。 既定の名前付け規則は、クラス名 **Bold** TagHelper を `<bold>` タグと一致させることに注意してください。

1. アプリを実行して、`<bold>` タグがタグ ヘルパーによって処理されることを確認します。

複数の `[HtmlTargetElement]` 属性でクラスを修飾すると、結果はターゲットの論理 OR になります。 たとえば、次のコードを使用すると、bold タグまたは bold 属性が一致します。

[!code-csharp[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/zBoldTagHelperCopy.cs?highlight=1,2&range=5-15)]

同じステートメントに複数の属性を追加すると、ランタイムではそれらが論理 AND として扱われます。 たとえば、次のコードでは、一致させるために、"bold" という名前の属性 (`<bold bold />`) を使って HTML 要素に "bold" と名前を付ける必要があります。

```csharp
[HtmlTargetElement("bold", Attributes = "bold")]
   ```

また、`[HtmlTargetElement]` を使用してターゲット要素の名前を変更することもできます。 たとえば、`BoldTagHelper` で `<MyBold>` タグをターゲットにするには、次の属性を使用します。

```csharp
[HtmlTargetElement("MyBold")]
```

## <a name="pass-a-model-to-a-tag-helper"></a>タグ ヘルパーにモデルを渡す

1. *Models* フォルダーを追加します。

1. 次の `WebsiteContext` クラスを *Models* フォルダーに追加します。

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Models/WebsiteContext.cs)]

1. 次の `WebsiteInformationTagHelper` クラスを *TagHelpers* フォルダーに追加します。

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/WebsiteInformationTagHelper.cs)]

   * 前述したように、タグ ヘルパーは、パスカルケースの C# クラス名とタグ ヘルパーのプロパティを[ケバブ ケース](https://wiki.c2.com/?KebabCase)に変換します。 したがって、のを使用するには、を `WebsiteInformationTagHelper` Razor 記述 `<website-information />` します。

   * `[HtmlTargetElement]` 属性を使用してターゲット要素を明示的に識別しないため、`website-information` の既定がターゲットになります。 次の属性を適用した場合 (ケバブ ケースではありませんが、クラス名が一致します):

   ```csharp
   [HtmlTargetElement("WebsiteInformation")]
   ```

   ケバブ ケース タグ `<website-information />` は一致しません。 `[HtmlTargetElement]` 属性を使用する場合は、次に示すようにケバブ ケースを使用します。

   ```csharp
   [HtmlTargetElement("Website-Information")]
   ```

   * 自己終了の要素にはコンテンツがありません。 この例では、 Razor マークアップは自己終了タグを使用しますが、タグヘルパーは [section](https://www.w3.org/TR/html5/sections.html#the-section-element) 要素を作成します (これは自己終了ではなく、要素内にコンテンツを書き込んでい `section` ます)。 そのため、`TagMode` を `StartTagAndEndTag` に設定して出力を記述する必要があります。 または、`TagMode` を設定する行をコメント アウトして、終了タグを使ってマークアップを記述することもできます。 (サンプルのマークアップは、後ほどこのチュートリアルで提供します。)

   * 次の行の `$` (ドル記号) は、[挿入文字列](/dotnet/csharp/language-reference/keywords/interpolated-strings)を使用しています。

   ```cshtml
   $@"<ul><li><strong>Version:</strong> {Info.Version}</li>
   ```

1. *About.cshtml* ビューに次のマークアップを追加します。 強調表示されたマークアップは、Web サイトの情報を表示します。

   [!code-cshtml[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/About.cshtml?highlight=1,4-8, 18-999)]

   > [!NOTE]
   > Razor次に示すマークアップでは、
   >
   > [!code-html[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/About.cshtml?range=18-18)]
   >
   > Razor`info`属性が文字列ではなくクラスであることを認識し、C# コードを記述します。 文字列以外のタグ ヘルパー属性はすべて、`@` 文字を使わずに記述する必要があります。

1. アプリを実行し、[バージョン情報] ビューに移動して、Web サイトの情報を確認します。

   > [!NOTE]
   > 終了タグを持つ次のマークアップを使用して、タグ ヘルパー内の `TagMode.StartTagAndEndTag` を持つ行を削除できます。
   >
   > [!code-html[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/AboutNotSelfClosing.cshtml?range=20-21)]

## <a name="condition-tag-helper"></a>条件タグ ヘルパー

条件タグ ヘルパーは、true 値が渡されたときに出力をレンダリングします。

1. 次の `ConditionTagHelper` クラスを *TagHelpers* フォルダーに追加します。

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/ConditionTagHelper.cs)]

1. *Views/Home/Index.cshtml* ファイルの内容を次のマークアップに置き換えます。

   [!code-cshtml[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/Index.cshtml)]

1. `Home` コントローラーの `Index` メソッドを次のコードで置き換えます。

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Controllers/HomeController.cs?range=9-18)]

1. アプリを実行して、ホーム ページに移動します。 条件付き `div` ではマークアップはレンダリングされません。 クエリ文字列 `?approved=true` を URL に付加します (例: `http://localhost:1235/Home/Index?approved=true`)。 `approved` は true に設定され、条件付きマークアップが表示されます。

> [!NOTE]
> [nameof](/dotnet/csharp/language-reference/keywords/nameof) 演算子を使用して、太字タグ ヘルパーで行ったように文字列を指定するのではなく、ターゲットにする属性を指定します。
>
> [!code-csharp[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/zConditionTagHelperCopy.cs?highlight=1,2,5&range=5-18)]
>
> [nameof](/dotnet/csharp/language-reference/keywords/nameof) 演算子は、コードがリファクタリングされないように保護します (名前は `RedCondition` に変更できます)。

### <a name="avoid-tag-helper-conflicts"></a>タグ ヘルパーの競合を回避する

このセクションでは、自動リンク タグ ヘルパーのペアを記述します。 1 つ目のヘルパーは HTTP で始まる URL を含むマークアップを、同じ URL を含む HTML アンカー タグに置き換えます (この URL へのリンクも生成されます)。 2 つ目のヘルパーは WWW で始まる URL に対して同じことを行います。

これら 2 つのヘルパーは密接に関連しており、将来的にそれらをリファクターする可能性があるため、これらを同じファイルで保持します。

1. 次の `AutoLinkerHttpTagHelper` クラスを *TagHelpers* フォルダーに追加します。

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinker.cs?range=7-19)]

   >[!NOTE]
   >`AutoLinkerHttpTagHelper` クラスは `p` 要素をターゲットとし、[Regex](/dotnet/standard/base-types/regular-expression-language-quick-reference) を使用してアンカーを作成します。

1. 次のマークアップを *Views/Home/Contact.cshtml* ファイルの末尾に追加します。

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/Contact.cshtml?highlight=19)]

1. アプリを実行し、タグ ヘルパーがアンカーを正しくレンダリングすることを確認します。

1. `AutoLinker` クラスを更新して `AutoLinkerWwwTagHelper` を含めます。これは www テキストを元の www テキストも含むアンカー タグに変換します。 以下では、更新されたコードが強調表示されています。

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinker.cs?highlight=15-34&range=7-34)]

1. アプリを実行します。 www テキストはリンクとしてレンダリングされていますが、HTTP テキストはそのようになっていないことに注目してください。 両方のクラスにブレークポイントを配置すると、HTTP タグ ヘルパー クラスが最初に実行されることがわかります。 問題は、タグ ヘルパーの出力がキャッシュされ、WWW タグ ヘルパーを実行するときに、WWW タグ ヘルパーが HTTP タグ ヘルパーからのキャッシュされた出力を上書きすることです。 タグ ヘルパーの実行順序を制御する方法については、このチュートリアルで後ほど説明します。 コードを次のように修正します。

   [!code-csharp[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinkerCopy.cs?highlight=5,6,10,21,22,26&range=8-37)]

   > [!NOTE]
   > 自動リンク タグ ヘルパーの最初のエディションで、次のコードを使用してターゲットのコンテンツを取得しました。
   >
   > [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinker.cs?range=12)]
   >
   > つまり、`ProcessAsync` メソッドに渡された `TagHelperOutput` を使用して `GetChildContentAsync` を呼び出します。 前述したように、出力はキャッシュされるため、最後に実行されたタグ ヘルパーによって上書きされます。 次のコードを使用してこの問題を修正しました。
   >
   > [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z2AutoLinkerCopy.cs?range=34-35)]
   >
   > 上記のコードは、コンテンツが変更されているかどうかをチェックし、変更されている場合は、出力バッファーからコンテンツを取得します。

1. アプリを実行して、2 つのリンクが期待どおりに動作することを確認します。 自動リンカー タグ ヘルパーが正しく完全に機能しているように見えても、微妙な問題があります。 WWW タグ ヘルパーを最初に実行すると、www リンクが不正確になります。 `Order` オーバーロードを追加してコードを更新し、タグの実行順序を制御します。 `Order` プロパティを、同じ要素をターゲットとする他のタグ ヘルパーと比べることで実行順序が決まります。 既定の順序の値は 0 で、より低い値を持つインスタンスの方が先に実行されます。

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z2AutoLinkerCopy.cs?highlight=5,6,7,8&range=8-15)]

   上記のコードにより、WWW タグ ヘルパーの前に、HTTP タグ ヘルパーが実行されることが保証されます。 `Order` を `MaxValue` に変更し、WWW タグの生成されたマークアップが正しいことを確認します。

## <a name="inspect-and-retrieve-child-content"></a>子コンテンツを検査して取得する

タグ ヘルパーには、コンテンツを取得するための複数のプロパティが用意されています。

* `GetChildContentAsync` の結果を `output.Content` に付加できます。
* `GetContent` を使用して `GetChildContentAsync` の結果を検査できます。
* `output.Content` を変更すると、この自動リンカー サンプルのように、`GetChildContentAsync` を呼び出すまで、TagHelper ボディは実行またはレンダリングされません。

[!code-csharp[](../../views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinkerCopy.cs?highlight=5,6,10&range=8-21)]

* `GetChildContentAsync` を複数回呼び出すと、同じ値が返され、キャッシュされた結果を使用しないことを示す false パラメーターを渡さない限り、`TagHelper` ボディは再実行されません。

## <a name="load-minified-partial-view-taghelper"></a>縮小された部分ビュー TagHelper を読み込む

運用環境では、縮小された部分ビューを読み込むことでパフォーマンスを向上できます。 運用環境で縮小された部分ビューを活用するには:

* 部分ビューを縮小するビルド前プロセスを作成/設定します。
* 次のコードを使って、非開発環境に縮小された部分ビューを読み込みます。

[!code-csharp[](authoring/sample/AuthoringTagHelpers/src/MinifiedVersionTagHelper.cs?name=snippet)]
