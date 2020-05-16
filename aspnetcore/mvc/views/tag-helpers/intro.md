---
title: ASP.NET Core のタグ ヘルパー
author: rick-anderson
description: タグ ヘルパーと、ASP.NET Core でのその使用方法について説明します。
ms.author: riande
ms.custom: H1Hack27Feb2017
ms.date: 03/18/2019
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: mvc/views/tag-helpers/intro
ms.openlocfilehash: 7298e514938e8285a1e42afad0f2f71375e2781d
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/04/2020
ms.locfileid: "82776475"
---
# <a name="tag-helpers-in-aspnet-core"></a>ASP.NET Core のタグ ヘルパー

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

## <a name="what-are-tag-helpers"></a>タグ ヘルパーとは

タグヘルパーを使用すると、サーバー側コードはファイル内のRazor HTML 要素の作成とレンダリングに参加できます。 たとえば、組み込みの `ImageTagHelper` では、イメージ名にバージョン番号を追加することができます。 イメージが変更されるたびに、サーバーはイメージに対して新しい一意のバージョンを生成するため、クライアントは (キャッシュされた古いイメージではなく) 現在のイメージを確実に取得できます。 フォームやリンクの作成、アセットの読み込みなど、一般的なタスクの組み込みのタグ ヘルパーは数多くあります。パブリック GitHub リポジトリで NuGet パッケージとして使用することもできます。 タグ ヘルパーは C# で作成され、要素名、属性名、または親タグに基づく HTML 要素をターゲットとします。 たとえば、`LabelTagHelper` 属性が適用されている場合、組み込みの `LabelTagHelper` では HTML `<label>` 要素をターゲットとすることができます。 [Html ヘルパー](https://stephenwalther.com/archive/2009/03/03/chapter-6-understanding-html-helpers)を使い慣れている場合、タグヘルパーを使用すると、ビューでRazorの html と C# の間の明示的な切り替えを減らすことができます。 多くの場合、HTML ヘルパーでは特定のタグ ヘルパーの代替方法が提供されますが、タグ ヘルパーを HTML ヘルパーの代わりに使用することはできないことと、各 HTML ヘルパーに対応するタグ ヘルパーがないことを認識することが重要です。 2 つの違いの詳細については、「[タグ ヘルパーと HTML ヘルパーの比較](#tag-helpers-compared-to-html-helpers)」を参照してください。

## <a name="what-tag-helpers-provide"></a>タグ ヘルパーで提供されるもの

**HTML を使いやすい開発エクスペリエンス**ほとんどの場合、タグRazorヘルパーを使用したマークアップは、標準 HTML のように見えます。 HTML/CSS/JavaScript で精通するフロントエンドデザイナーは、 Razor C# Razorの構文を学習せずに編集できます。

**HTML およびマークアップを作成するためRazorの豊富な IntelliSense 環境**は、html ヘルパーとは対照的に、ビューでRazorのマークアップのサーバー側作成に対する従来のアプローチです。 2 つの違いの詳細については、「[タグ ヘルパーと HTML ヘルパーの比較](#tag-helpers-compared-to-html-helpers)」を参照してください。 IntelliSense 環境については、「[Intellisense でのタグ ヘルパーのサポート](#intellisense-support-for-tag-helpers)」を参照してください。 C# の構文にRazor慣れている開発者であっても、タグRazorヘルパーを使用して c# マークアップを記述するよりも生産性が向上します。

**生産性を高め、サーバーでのみ使用可能な情報を使用することで、より堅牢で信頼できる保守しやすいコードを生成できる方法** たとえば、これまでは、イメージを変更する際にそのイメージ名を変更することがイメージ更新の理念でした。 パフォーマンス上の理由から、イメージを積極的にキャッシュする必要があり、イメージの名前を変更しない限り、クライアントが古いコピーを取得する危険性があります。 これまでは、イメージの編集後に、名前を変更する必要があり、Web アプリでのイメージへの各参照を更新する必要がありました。 非常に手間がかかるだけでなく、エラーも発生しやすくなります (参照を見逃したり、間違った文字列を誤って入力したりする可能性があります)。組み込みのは、 `ImageTagHelper`自動的に行うことができます。 `ImageTagHelper` ではイメージ名にバージョン番号を追加できるため、イメージが変更されるたびに、サーバーはそのイメージに対して新しい一意のバージョンを自動的に生成します。 クライアントは現在のイメージを確実に取得できます。 `ImageTagHelper` を使用することで、このような堅牢性と省力化を基本的に自由に実現できます。

ほとんどの組み込みタグ ヘルパーは、標準の HTML 要素をターゲットとし、要素に対してサーバー側の属性を提供します。 たとえば、*Views/Account* フォルダーの多くのビューで使用される `<input>` 要素には、`asp-for` 属性が含まれて この属性は、指定されたモデル プロパティの名前をレンダリングされる HTML に抽出します 次のRazorモデルのビューを考えてみましょう。

```csharp
public class Movie
{
    public int ID { get; set; }
    public string Title { get; set; }
    public DateTime ReleaseDate { get; set; }
    public string Genre { get; set; }
    public decimal Price { get; set; }
}
```

次Razorのマークアップ:

```cshtml
<label asp-for="Movie.Title"></label>
```

次の HTML が生成されます。

```html
<label for="Movie_Title">Title</label>
```

`asp-for` 属性は、[LabelTagHelper](/dotnet/api/microsoft.aspnetcore.mvc.taghelpers.labeltaghelper?view=aspnetcore-2.0) の `For` プロパティで使用できます。 詳しくは、[タグ ヘルパーの作成](xref:mvc/views/tag-helpers/authoring)に関するページをご覧ください。

## <a name="managing-tag-helper-scope"></a>タグ ヘルパーのスコープの管理

タグ ヘルパーのスコープは、`@addTagHelper`、`@removeTagHelper`、"!" オプトアウト文字の組み合わせによって制御されます。

<a name="add-helper-label"></a>

### <a name="addtaghelper-makes-tag-helpers-available"></a>`@addTagHelper` でタグ ヘルパーを使用可能にする

*AuthoringTagHelpers* という名前の新しい ASP.NET Core Web アプリを作成する場合、以下の *Views/_ViewImports.cshtml* ファイルがプロジェクトに追加されます。

[!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/_ViewImportsCopy.cshtml?highlight=2&range=2-3)]

`@addTagHelper` ディレクティブにより、ビューでタグ ヘルパーが使用可能になります。 この場合、ビュー ファイルは *Pages/_ViewImports.cshtml* であり、既定では *Pages* フォルダーとサブフォルダーのすべてのファイルによって継承され、タグ ヘルパーが使用可能になります。 上記のコードではワイルドカード構文 ("\*") を使用して、指定されたアセンブリ (*Microsoft.AspNetCore.Mvc.TagHelpers*) 内のすべてのタグ ヘルパーが、*Views* ディレクトリまたはサブディレクトリ内のすべてのビュー ファイルで使用可能になるように指定します。 `@addTagHelper` の後の最初のパラメーターでは読み込むタグ ヘルパーを指定し (すべてのタグ ヘルパーに対して "\*" を使用)、2 番目のパラメーター "Microsoft.AspNetCore.Mvc.TagHelpers" ではタグ ヘルパーを含むアセンブリを指定します。 *Microsoft.AspNetCore.Mvc.TagHelpers* は、組み込み ASP.NET Core タグ ヘルパーのアセンブリです。

すべてのタグ ヘルパーをこのプロジェクト (*AuthoringTagHelpers* という名前のアセンブリを作成する) で公開するには、以下を使用します。

[!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/_ViewImportsCopy.cshtml?highlight=3)]

プロジェクトに既定の名前空間がある `EmailTagHelper` が含まれている場合は (`AuthoringTagHelpers.TagHelpers.EmailTagHelper`)、タグ ヘルパーの完全修飾名 (FQN) を指定できます。

```cshtml
@using AuthoringTagHelpers
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper AuthoringTagHelpers.TagHelpers.EmailTagHelper, AuthoringTagHelpers
```

FQN を使用してビューにタグ ヘルパーを追加するには、最初に FQN (`AuthoringTagHelpers.TagHelpers.EmailTagHelper`) を追加してから、アセンブリ名 (*AuthoringTagHelpers*) を追加します。 ほとんどの開発者は、"\*" ワイルドカード構文を使用するほうを選びます。 ワイルドカード構文を使用することで、FQN のサフィックスとしてワイルドカード文字 "\*" を挿入できます。 たとえば、以下のディレクティブのいずれかで `EmailTagHelper` を取り込みます。

```cshtml
@addTagHelper AuthoringTagHelpers.TagHelpers.E*, AuthoringTagHelpers
@addTagHelper AuthoringTagHelpers.TagHelpers.Email*, AuthoringTagHelpers
```

前述のとおり、`@addTagHelper` ディレクティブを *Views/_ViewImports.cshtml* ファイルに追加すると、タグ ヘルパーを *Views* ディレクトリとサブディレクトリ内のすべてのビュー ファイルで使用できるようになります。 これらのビューにのみタグ ヘルパーを公開することを選択する場合は、特定のビュー ファイルで `@addTagHelper` ディレクティブを使用できます。

<a name="remove-razor-directives-label"></a>

### <a name="removetaghelper-removes-tag-helpers"></a>`@removeTagHelper` でタグ ヘルパーを削除する

`@removeTagHelper` には `@addTagHelper` と同じ 2 つのパラメーターがあり、以前に追加されたタグ ヘルパーを削除します。 たとえば、特定のビューに適用された `@removeTagHelper` では、指定されたタグ ヘルパーをビューから削除します。 *Views/Folder/_ViewImports.cshtml* ファイルで `@removeTagHelper` を使用すると、*Folder* 内のすべてのビューから指定されたタグ ヘルパーが削除されます。

### <a name="controlling-tag-helper-scope-with-the-_viewimportscshtml-file"></a>*_ViewImports.cshtml* ファイルによるタグ ヘルパー スコープの制御

*_ViewImports.cshtml* を任意のビュー フォルダーに追加することができ、ビュー エンジンではそのファイルと *Views/_ViewImports.cshtml* ファイルの両方のディレクティブが適用されます。 *Home* ビューに対して空の *Views/Home/_ViewImports.cshtml* ファイルを追加した場合、*_ViewImports.cshtml* ファイルは付加的なものであるため、何も変わりません。 *Views/Home/_ViewImports.cshtml* ファイルに追加する `@addTagHelper` ディレクティブ (既定の *Views/_ViewImports.cshtml* ファイルにはない) では、これらのタグ ヘルパーが *Home* フォルダー内のビューにのみ公開されます。

<a name="opt-out"></a>

### <a name="opting-out-of-individual-elements"></a>個々の要素の無効化

タグ ヘルパーのオプトアウト文字 ("!") を使用して、要素レベルでタグ ヘルパーを無効にすることができます。 たとえば、`Email` 検証は、タグ ヘルパーのオプトアウト文字を使用して `<span>` で無効にすることができます。

```cshtml
<!span asp-validation-for="Email" class="text-danger"></!span>
```

開始タグと終了タグにタグ ヘルパーのオプトアウト文字を適用する必要があります  (Visual Studio エディターでは、ユーザーがオプトアウト文字を開始タグに追加すると、自動的にオプトアウト文字が終了タグに追加されます)。 オプトアウト文字を追加した後、要素とタグ ヘルパーの属性は独特なフォントで表示されなくなります。

<a name="prefix-razor-directives-label"></a>

### <a name="using-taghelperprefix-to-make-tag-helper-usage-explicit"></a>`@tagHelperPrefix` を使用してタグ ヘルパーの使用状況を明確にする

`@tagHelperPrefix` ディレクティブでは、タグ プレフィックス文字列を指定して、タグ ヘルパーのサポートを有効にしたり、タグ ヘルパーの使用状況を明確にすることができます。 たとえば、*Views/_ViewImports.cshtml* ファイルに次のマークアップを追加できます。

```cshtml
@tagHelperPrefix th:
```

以下のコード イメージでは、タグ ヘルパーのプレフィックスが `th:` に設定されているため、プレフィックス `th:` を使用する要素のみでタグ ヘルパーがサポートされます (タグ ヘルパーが有効な要素には独特なフォントがあります)。 `<label>` 要素と `<input>` 要素にはタグ ヘルパー プレフィックスがあり、タグ ヘルパーが有効ですが、`<span>` 要素にはありません。

![image](intro/_static/thp.png)

`@addTagHelper` に適用されるものと同じ階層規則が `@tagHelperPrefix` にも適用されます。

## <a name="self-closing-tag-helpers"></a>自己終了タグ ヘルパー

多くのタグ ヘルパーは、自己終了タグとして使用できません。 一部のタグ ヘルパーは、自己終了タグとして設計されています。 自己終了するようになっていないタグ ヘルパーを使用すると、表示される出力が抑制されます。 タグ ヘルパーを自己終了すると、表示される出力の中に自己終了タグが配置されます。 詳細については、[タグ ヘルパーの作成](xref:mvc/views/tag-helpers/authoring)に関するページの[こちらの注意](xref:mvc/views/tag-helpers/authoring#self-closing)をご覧ください。

## <a name="c-in-tag-helpers-attributedeclaration"></a>タグ ヘルパーの属性/宣言内の C# 

タグ ヘルパーでは要素の属性またはタグ宣言区分の C# は許可されません。 たとえば、次のようなコードは有効ではありません。

```cshtml
<input asp-for="LastName"  
       @(Model?.LicenseId == null ? "disabled" : string.Empty) />
```

上記のコードは、次のように記述できます。

```cshtml
<input asp-for="LastName" 
       disabled="@(Model?.LicenseId == null)" />
```

## <a name="intellisense-support-for-tag-helpers"></a>Intellisense でのタグ ヘルパーのサポート

Visual Studio で新しい ASP.NET Core web アプリを作成すると、NuGet パッケージ "AspNetCore" が追加されます。Razor.ツール " これは、タグ ヘルパー ツールを追加するパッケージです。

HTML `<label>` 要素を書き込むことを検討してください。 Visual Studio エディターで `<l` を入力するとすぐに、IntelliSense で一致する要素が表示されます。

![image](intro/_static/label.png)

HTML ヘルプだけでなく、アイコン ("@" symbol with "<>") も表示されます。

![image](intro/_static/tagSym.png)

これは、タグ ヘルパーのターゲットとなる要素を示します。 純粋な HTML 要素 (`fieldset` など) では "<>" アイコンが表示されます。

純粋な HTML `<label>` タグの場合、HTML タグ (既定の Visual Studio の配色テーマ) が茶色のフォントで、属性が赤で、属性値が青で表示されます。

![image](intro/_static/LableHtmlTag.png)

`<label` を入力した後、IntelliSense では次のように使用可能な HTML/CSS 属性とタグ ヘルパーのターゲットとなる属性がリストされます。

![image](intro/_static/labelattr.png)

IntelliSense ステートメント入力候補では、Tab キーを入力して、選択した値を使用してステートメントを完了することができます。

![image](intro/_static/stmtcomplete.png)

タグ ヘルパー属性が入力されるとすぐに、タグと属性のフォントが変わります。 既定の Visual Studio の "青" または "ライト" の配色テーマを使用すると、フォントが太字の紫になります。 "ダーク" テーマを使用する場合、フォントは太字の青緑になります。 このドキュメントのイメージは、既定のテーマを使用して取得されたものです。

![image](intro/_static/labelaspfor2.png)

Visual Studio の *CompleteWord* ショートカット ([既定](/visualstudio/ide/default-keyboard-shortcuts-in-visual-studio)は Ctrl + Space) を二重引用符 ("") 内に入力することができ、C# クラスの場合と同じように C# で使用できるようになりました。 IntelliSense では、ページ モデルですべてのメソッドとプロパティが表示されます。 プロパティの種類が `ModelExpression` であるため、メソッドとプロパティを使用できます。 次のイメージでは、`Register` ビューを編集するため、`RegisterViewModel` を使用できます。

![image](intro/_static/intellemail.png)

IntelliSense では、ページのモデルで使用可能なプロパティとメソッドがリストされます。 豊富な IntelliSense 環境は、CSS クラスを選択する場合に役立ちます。

![image](intro/_static/iclass.png)

![image](intro/_static/intel3.png)

## <a name="tag-helpers-compared-to-html-helpers"></a>タグ ヘルパーと HTML ヘルパーの比較

タグヘルパーはビュー内のRazor html 要素にアタッチされますが、 [html ヘルパー](https://stephenwalther.com/archive/2009/03/03/chapter-6-understanding-html-helpers)はビューでRazor html と共に混在するメソッドとして呼び出されます。 CSS クラス " Razor caption" を使用して HTML ラベルを作成する次のマークアップについて考えてみます。

```cshtml
@Html.Label("FirstName", "First Name:", new {@class="caption"})
```

At (`@`) 記号は、 Razorこれがコードの先頭であることを示します。 次の 2 つのパラメーター ("FirstName" と "First Name:") は文字列であるため、[IntelliSense](/visualstudio/ide/using-intellisense) では対応できません。 以下が最後の引数です。

```cshtml
new {@class="caption"}
```

これは属性を表すために使用される匿名オブジェクトです。 `class` は C# の予約済みのキーワードであるため、`@` シンボルを使用して、C# で強制的に `@class=` をシンボル (プロパティ名) として解釈するようにします。 フロントエンドデザイナー (HTML/CSS/JavaScript およびその他のクライアントテクノロジに慣れていても、C# やRazorには慣れていないユーザー) には、ほとんどの行が外部にあります。 IntelliSense に頼らずに行全体を作成する必要があります。

`LabelTagHelper` を使用すれば、以下と同じマークアップを書き込むことができます。

```cshtml
<label class="caption" asp-for="FirstName"></label>
```

タグ ヘルパー バージョンを使用して Visual Studio エディターで `<l` を入力するとすぐに、IntelliSense で一致する要素が表示されます。

![image](intro/_static/label.png)

IntelliSense は行全体を書き込む場合に役立ちます。

次のコードイメージは、Visual Studio に含まれている ASP.NET 4.5. x MVC テンプレートから生成された*Views/Account/Register. cshtml* Razorビューのフォーム部分を示しています。

![image](intro/_static/regCS.png)

Visual Studio エディターでは、背景が灰色の C# コードが表示されます。 たとえば、次の `AntiForgeryToken` HTML ヘルパーの場合、

```cshtml
@Html.AntiForgeryToken()
```

灰色の背景で表示されます。 Register ビューのマークアップのほとんどは C# です。 以下のタグ ヘルパーを使用する同じ方法と比較します。

![image](intro/_static/regTH.png)

HTML ヘルパーの方法よりも、マークアップがわかりやすく、読み取り、編集、保持が簡単になります。 C# コードは、サーバーで認識する必要がある最低限まで減っています。 Visual Studio エディターでは、独特なフォントでタグ ヘルパーのターゲットとなるマークアップが表示されます。

たとえば、以下の *Email* グループについて考えてみます。

[!code-csharp[](intro/sample/Register.cshtml?range=12-18)]

"asp-" 属性にはそれぞれ "Email" の値がありますが、"Email" は文字列ではありません。 このコンテキストでは、"Email" は `RegisterViewModel` の C# モデル式のプロパティとなります。

Visual Studio エディターは登録フォームのタグ ヘルパーの方法で**すべて**のマークアップを書き込む場合に役立ちますが、Visual Studio は HTML ヘルパーの方法ではほとんどのコードに対応できません。 Visual Studio エディターでのタグ ヘルパーの操作の詳細については、「[IntelliSense でのタグ ヘルパーのサポート](#intellisense-support-for-tag-helpers)」を参照してください。

## <a name="tag-helpers-compared-to-web-server-controls"></a>タグ ヘルパーと Web サーバー コントロールの比較

* タグ ヘルパーには関連付けられている要素はなく、要素とコンテンツのレンダリングに参加するだけです。 ASP.NET [Web サーバー コントロール](https://msdn.microsoft.com/library/7698y1f0.aspx)はページで宣言され、呼び出されます。

* [Web サーバー コントロール](https://msdn.microsoft.com/library/zsyt68f1.aspx)には複雑なライフサイクルがあり、これが開発とデバッグを困難にする場合があります。

* Web サーバー コントロールではクライアント コントロールを使用して、クライアントのドキュメント オブジェクト モデル (DOM) 要素に機能を追加することができます。 タグ ヘルパーには DOM がありません。

* Web サーバー コントロールには自動ブラウザー検出が含まれます。 タグ ヘルパーではブラウザーが認識されません。

* 複数のタグ ヘルパーを同じ要素で実行できますが ([タグ ヘルパーの競合の回避](xref:mvc/views/tag-helpers/authoring#avoid-tag-helper-conflicts)に関するページを参照)、通常は Web サーバー コントロールを構成できません。

* タグ ヘルパーではスコープとなる HTML 要素のタグとコンテンツを変更できますが、ページ上の他のものを直接変更しません。 Web サーバー コントロールではスコープが限定されないため、ページの他の部分に影響を与えるアクションを実行して、予想外の結果となる場合があります。

* Web サーバー コントロールでは型コンバーターを使用して、文字列をオブジェクトに変換します。 タグ ヘルパーの場合、C# でネイティブに作業するため、型を変換する必要はありません。

* Web サーバー コントロールでは [System.ComponentModel](/dotnet/api/system.componentmodel) 名前空間を使用して、コンポーネントやコントロールの実行時動作およびデザイン時動作を実装します。 `System.ComponentModel` には、属性と型コンバーターの実装、データ ソースへのバインド、コンポーネントのライセンス処理のための基底クラスと基底インターフェイスが含まれています。 通常は `TagHelper` から派生するタグ ヘルパーとは異なり、`TagHelper` 基底クラスでは `Process` と `ProcessAsync` の 2 つのメソッドのみを公開します。

## <a name="customizing-the-tag-helper-element-font"></a>タグ ヘルパー要素のフォントのカスタマイズ

[**ツール** > **] [オプション]** > [**環境** > ]**[フォントおよび色**] から、フォントと色付けをカスタマイズできます。

![image](intro/_static/fontoptions2.png)

[!INCLUDE[](~/includes/built-in-TH.md)]

## <a name="additional-resources"></a>その他のリソース

* [タグ ヘルパーの作成](xref:mvc/views/tag-helpers/authoring)
* [フォームの操作](xref:mvc/views/working-with-forms)
* [GitHub の TagHelperSamples](https://github.com/dpaquette/TagHelperSamples) には、[ブートストラップ](https://getbootstrap.com/)を操作するためのタグ ヘルパーのサンプルが含まれています。
