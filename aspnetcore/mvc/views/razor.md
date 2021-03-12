---
title: ASP.NET Core の razor 構文リファレンス
author: rick-anderson
description: RazorWeb ページにサーバーベースのコードを埋め込むためのマークアップ構文について説明します。
ms.author: riande
ms.date: 02/12/2020
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
uid: mvc/views/razor
ms.openlocfilehash: 60471232b3373039404b27c4afd1a1725d4d21eb
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586866"
---
# <a name="razor-syntax-reference-for-aspnet-core"></a>ASP.NET Core の Razor 構文リファレンス

[Rick Anderson](https://twitter.com/RickAndMSFT)、 [Taylor Mullen](https://twitter.com/ntaylormullen)、 [Dan Vicarel](https://github.com/Rabadash8820)

Razor は、web ページにサーバーベースのコードを埋め込むためのマークアップ構文です。 構文は、 Razor Razor マークアップ、C#、および HTML で構成されています。 に含まれるファイル Razor には、通常、拡張子が付いています。 Razorは、 [ Razor コンポーネント](xref:blazor/components/index)ファイル (*razor*) にもあります。

## <a name="rendering-html"></a>HTML のレンダリング

既定の Razor 言語は HTML です。 マークアップから html をレンダリング Razor することは、html ファイルから html をレンダリングする場合と同じです。 ファイル *内の* HTML マークアップ Razor は、サーバーによって変更されずに表示されます。

## <a name="razor-syntax"></a>Razor の構文

Razor C# をサポートし、シンボルを使用して `@` HTML から c# に移行します。 Razor C# の式を評価し、HTML 出力に表示します。

記号の `@` 後に[ Razor 予約済みのキーワード](#razor-reserved-keywords)がある場合は、 Razor 固有のマークアップに遷移します。 それ以外の場合は、普通の C# に移行します。

マークアップでシンボルをエスケープするには `@` Razor 、2番目のシンボルを使用し `@` ます。

```cshtml
<p>@@Username</p>
```

HTML では、コードは 1 つの `@` 記号でレンダリングされます。

```html
<p>@Username</p>
```

メール アドレスを含む HTML の属性とコンテンツは、`@` 記号を遷移文字として扱いません。 次の例の電子メールアドレスは、解析によってそのまま残り Razor ます。

```cshtml
<a href="mailto:Support@contoso.com">Support@contoso.com</a>
```

## <a name="implicit-razor-expressions"></a>暗黙的な Razor 式

暗黙的 Razor な式の先頭に `@` は、次の C# コードが続きます。

```cshtml
<p>@DateTime.Now</p>
<p>@DateTime.IsLeapYear(2016)</p>
```

C# の `await` キーワードを除き、暗黙的な式にスペースを含めることはできません。 C# のステートメントに明確な終わりがある場合は、スペースを混在させることができます。

```cshtml
<p>@await DoSomething("hello", "world")</p>
```

暗黙的な式では、山かっこ (`<>`) の内側の文字は HTML タグとして解釈されるため、C# ジェネリックを含めることは **できません**。 次のコードは有効では **ありません**。

```cshtml
<p>@GenericMethod<int>()</p>
```

上記のコードでは、次のいずれかのようなコンパイラ エラーが生成されます。

* The "int" element wasn't closed. All elements must be either self-closing or have a matching end tag. ("int" 要素が閉じられませんでした。すべての要素は、自己終了するか、対応する終了タグが存在する必要があります。)
* メソッド グループ 'GenericMethod' を非デリゲート型 'object' に変換することはできません。 このメソッドを呼び出しますか?

ジェネリックメソッドの呼び出しは、[明示的な Razor 式](#explicit-razor-expressions)または[ Razor コードブロック](#razor-code-blocks)でラップする必要があります。

## <a name="explicit-razor-expressions"></a>明示的な Razor 式

明示的な Razor 式は、 `@` 均衡かっこ付きの記号で構成されます。 先週の時間を表示するには、次の Razor マークアップを使用します。

```cshtml
<p>Last week this time: @(DateTime.Now - TimeSpan.FromDays(7))</p>
```

`@()` のかっこ内の内容が評価されて、出力にレンダリングされます。

前のセクションで説明した暗黙的な式は、一般に、スペースを含むことはできません。 次のコードでは、現在時刻から 1 週間は減算されません。

[!code-cshtml[](razor/sample/Views/Home/Contact.cshtml?range=17)]

このコードでは、次のような HTML がレンダリングされます。

```html
<p>Last week: 7/7/2016 4:39:52 PM - TimeSpan.FromDays(7)</p>
```

明示的な式を使うと、テキストと式の結果を連結できます。

```cshtml
@{
    var joe = new Person("Joe", 33);
}

<p>Age@(joe.Age)</p>
```

明示的な式を使わないと、`<p>Age@joe.Age</p>` はメール アドレスとして扱われ、`<p>Age@joe.Age</p>` がレンダリングされます。 明示的な式として記述すると、`<p>Age33</p>` がレンダリングされます。

明示的な式を使うと、ジェネリック メソッドから *.cshtml* ファイルに出力できます。 次のマークアップは、C# ジェネリックの山かっこによって発生した前述のエラーを修正する方法について示します。 コードは明示的な式として書き込まれます。

```cshtml
<p>@(GenericMethod<int>())</p>
```

## <a name="expression-encoding"></a>式のエンコード

文字列として評価される C# の式は、HTML でエンコードされます。 `IHtmlContent` として評価される C# の式は、`IHtmlContent.WriteTo` によって直接レンダリングされます。 `IHtmlContent` として評価されない C# の式は、`ToString` によって文字列に変換され、レンダリングされる前にエンコードされます。

```cshtml
@("<span>Hello World</span>")
```

上記のコードは、次の HTML をレンダリングします。

```html
&lt;span&gt;Hello World&lt;/span&gt;
```

HTML は、プレーンテキストとしてブラウザーに表示されます。

&lt;スパン &gt; Hello World &lt; /span&gt;

`HtmlHelper.Raw` の出力はエンコードされませんが、HTML マークアップとしてレンダリングされません。

> [!WARNING]
> サニタイズされていないユーザー入力で `HtmlHelper.Raw` を使うと、セキュリティ上のリスクがあります。 ユーザー入力には、悪意のある JavaScript または他の攻撃が含まれる可能性があります。 ユーザー入力をサニタイズすることは困難です。 ユーザー入力では `HtmlHelper.Raw` を使わないでください。

```cshtml
@Html.Raw("<span>Hello World</span>")
```

このコードでは、次のような HTML がレンダリングされます。

```html
<span>Hello World</span>
```

## <a name="razor-code-blocks"></a>Razor コードブロック

Razor コードブロックはで始まり `@` 、で囲まれて `{}` います。 式とは異なり、コード ブロック内の C# コードはレンダリングされません。 ビュー内のコード ブロックと式は同じスコープを共有し、次の順序で定義されます。

```cshtml
@{
    var quote = "The future depends on what you do today. - Mahatma Gandhi";
}

<p>@quote</p>

@{
    quote = "Hate cannot drive out hate, only love can do that. - Martin Luther King, Jr.";
}

<p>@quote</p>
```

このコードでは、次のような HTML がレンダリングされます。

```html
<p>The future depends on what you do today. - Mahatma Gandhi</p>
<p>Hate cannot drive out hate, only love can do that. - Martin Luther King, Jr.</p>
```

::: moniker range=">= aspnetcore-3.0"

コード ブロックで、る[ローカル関数](/dotnet/csharp/programming-guide/classes-and-structs/local-functions)をマークアップで宣言し、テンプレート メソッドとしてのサービスを提供します。

```cshtml
@{
    void RenderName(string name)
    {
        <p>Name: <strong>@name</strong></p>
    }

    RenderName("Mahatma Gandhi");
    RenderName("Martin Luther King, Jr.");
}
```

このコードでは、次のような HTML がレンダリングされます。

```html
<p>Name: <strong>Mahatma Gandhi</strong></p>
<p>Name: <strong>Martin Luther King, Jr.</strong></p>
```

::: moniker-end

### <a name="implicit-transitions"></a>暗黙の移行

コードブロックの既定の言語は C# ですが、 Razor ページは HTML に戻ることができます。

```cshtml
@{
    var inCSharp = true;
    <p>Now in HTML, was in C# @inCSharp</p>
}
```

### <a name="explicit-delimited-transition"></a>明示的に区切られた遷移

HTML を表示する必要があるコードブロックのサブセクションを定義するには、タグを使用して、表示する文字を囲み Razor `<text>` ます。

```cshtml
@for (var i = 0; i < people.Length; i++)
{
    var person = people[i];
    <text>Name: @person.Name</text>
}
```

HTML タグによって囲まれていない HTML をレンダリングするには、この方法を使います。 HTML またはタグがないと Razor 、 Razor ランタイムエラーが発生します。

`<text>` タグは、内容をレンダリングするときに空白文字を制御するのに便利です。

* `<text>` タグの間の内容だけがレンダリングされます。
* `<text>` タグの前後にある空白文字は HTML の出力には表示されません。

### <a name="explicit-line-transition"></a>明示的な行の遷移

残りの行全体をコード ブロック内に HTML としてレンダリングするには、`@:` 構文を使います。

```cshtml
@for (var i = 0; i < people.Length; i++)
{
    var person = people[i];
    @:Name: @person.Name
}
```

コードにを指定しないと `@:` 、 Razor ランタイムエラーが生成されます。

`@`ファイルに余分な文字が含まれ Razor ていると、ステートメントの後のブロックで、コンパイラエラーが発生する可能性があります。 これらのコンパイラ エラーは、実際のエラーは報告されたエラーより前で発生しているため、理解するのが難しい場合があります。 このエラーは、複数の暗黙的/明示的な式を 1 つのコード ブロックに結合した後で発生することがよくあります。

## <a name="control-structures"></a>制御構造

制御構造は、コード ブロックの拡張機能です。 コード ブロックのすべての側面 (マークアップへの遷移、インライン C# ) が、次の構造にも適用されます。

### <a name="conditionals-if-else-if-else-and-switch"></a>条件 `@if, else if, else, and @switch`

`@if` は、いつコードを実行するかを制御します。

```cshtml
@if (value % 2 == 0)
{
    <p>The value was even.</p>
}
```

`else` および `else if` には、`@` 記号は必要ありません。

```cshtml
@if (value % 2 == 0)
{
    <p>The value was even.</p>
}
else if (value >= 1337)
{
    <p>The value is large.</p>
}
else
{
    <p>The value is odd and small.</p>
}
```

次のマークアップでは、switch ステートメントの使い方を示します。

```cshtml
@switch (value)
{
    case 1:
        <p>The value is 1!</p>
        break;
    case 1337:
        <p>Your number is 1337!</p>
        break;
    default:
        <p>Your number wasn't 1 or 1337.</p>
        break;
}
```

### <a name="looping-for-foreach-while-and-do-while"></a>サウンド `@for, @foreach, @while, and @do while`

ループ制御ステートメントを使って、テンプレート化された HTML をレンダリングできます。 人の一覧をレンダリングするには:

```cshtml
@{
    var people = new Person[]
    {
          new Person("Weston", 33),
          new Person("Johnathon", 41),
          ...
    };
}
```

以下のループ ステートメントがサポートされています。

`@for`

```cshtml
@for (var i = 0; i < people.Length; i++)
{
    var person = people[i];
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>
}
```

`@foreach`

```cshtml
@foreach (var person in people)
{
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>
}
```

`@while`

```cshtml
@{ var i = 0; }
@while (i < people.Length)
{
    var person = people[i];
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>

    i++;
}
```

`@do while`

```cshtml
@{ var i = 0; }
@do
{
    var person = people[i];
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>

    i++;
} while (i < people.Length);
```

### <a name="compound-using"></a>複合 `@using`

C# では、オブジェクトを確実に破棄するために `using` オブジェクトが使われています。 で Razor は、同じメカニズムを使用して、追加のコンテンツを含む HTML ヘルパーを作成します。 次のコードの HTML ヘルパーは、`@using` ステートメントを含む `<form>` タグをレンダリングします。

```cshtml
@using (Html.BeginForm())
{
    <div>
        Email: <input type="email" id="Email" value="">
        <button>Register</button>
    </div>
}
```

### `@try, catch, finally`

例外処理は C# に似ています。

[!code-cshtml[](razor/sample/Views/Home/Contact7.cshtml)]

### `@lock`

Razor には、次のように、重要なセクションを lock ステートメントで保護する機能があります。

```cshtml
@lock (SomeLock)
{
    // Do critical section work
}
```

### <a name="comments"></a>コメント

Razor C# および HTML コメントをサポートしています。

```cshtml
@{
    /* C# comment */
    // Another C# comment
}
<!-- HTML comment -->
```

このコードでは、次のような HTML がレンダリングされます。

```html
<!-- HTML comment -->
```

Razor コメントは、web ページが表示される前にサーバーによって削除されます。 Razor は `@*  *@` 、を使用してコメントを区切ります。 次のコードはコメント化されているため、サーバーはどのマークアップもレンダリングしません。

```cshtml
@*
    @{
        /* C# comment */
        // Another C# comment
    }
    <!-- HTML comment -->
*@
```

## <a name="directives"></a>ディレクティブ

Razor ディレクティブは、記号の後に予約済みのキーワードを持つ暗黙的な式で表され `@` ます。 通常、ディレクティブは、ビューの解析方法を変更したり、異なる機能を有効にしたりします。

が Razor ビューのコードを生成する方法を理解すると、ディレクティブのしくみを理解しやすくなります。

[!code-cshtml[](razor/sample/Views/Home/Contact8.cshtml)]

上のコードでは、次のようなクラスが生成されます。

```csharp
public class _Views_Something_cshtml : RazorPage<dynamic>
{
    public override async Task ExecuteAsync()
    {
        var output = "Getting old ain't for wimps! - Anonymous";

        WriteLiteral("/r/n<div>Quote of the Day: ");
        Write(output);
        WriteLiteral("</div>");
    }
}
```

この記事の後半で、「 [ Razor ビューに対して生成された C# クラスを調べる](#inspect-the-razor-c-class-generated-for-a-view) 」セクションでは、この生成されたクラスを表示する方法について説明します。

### `@attribute`

`@attribute` ディレクティブでは、指定された属性が生成されたページまたはビューのクラスに追加されます。 次の例では、`[Authorize]` 属性が追加されます。

```cshtml
@attribute [Authorize]
```

::: moniker range=">= aspnetcore-3.0"

### `@code`

*このシナリオは、 Razor コンポーネント (razor) にのみ適用されます。*

この `@code` ブロックにより、 [ Razor コンポーネント](xref:blazor/components/index)は、コンポーネントに C# のメンバー (フィールド、プロパティ、およびメソッド) を追加できます。

```razor
@code {
    // C# members (fields, properties, and methods)
}
```

コンポーネントの場合 Razor 、 `@code` はのエイリアスであり、よりも優先され [`@functions`](#functions) `@functions` ます。 複数の `@code` ブロックが許容されます。

::: moniker-end

### `@functions`

`@functions` ディレクティブでは、生成されたクラスに C# メンバー (フィールド、プロパティ、メソッド) を追加できます。

```cshtml
@functions {
    // C# members (fields, properties, and methods)
}
```

::: moniker range=">= aspnetcore-3.0"

[ [ Razor コンポーネント](xref:blazor/components/index)] で、を使用して `@code` `@functions` C# メンバーを追加します。

::: moniker-end

次に例を示します。

[!code-cshtml[](razor/sample/Views/Home/Contact6.cshtml)]

このコードは、次の HTML マークアップを生成します。

```html
<div>From method: Hello</div>
```

次のコードは、生成された Razor C# クラスです。

[!code-csharp[](razor/sample/Classes/Views_Home_Test_cshtml.cs?range=1-19)]

::: moniker range=">= aspnetcore-3.0"

`@functions` メソッドは、マークアップが与えられているとき、テンプレート メソッドとしてサービスを提供します。

```cshtml
@{
    RenderName("Mahatma Gandhi");
    RenderName("Martin Luther King, Jr.");
}

@functions {
    private void RenderName(string name)
    {
        <p>Name: <strong>@name</strong></p>
    }
}
```

このコードでは、次のような HTML がレンダリングされます。

```html
<p>Name: <strong>Mahatma Gandhi</strong></p>
<p>Name: <strong>Martin Luther King, Jr.</strong></p>
```

### `@implements`

`@implements` ディレクティブでは、生成されたクラスのインターフェイスが実装されます。

次の例では、<xref:System.IDisposable.Dispose*> メソッドを呼び出せるように、<xref:System.IDisposable?displayProperty=fullName> が実装されます。

```cshtml
@implements IDisposable

<h1>Example</h1>

@functions {
    private bool _isDisposed;

    ...

    public void Dispose() => _isDisposed = true;
}
```

::: moniker-end

### `@inherits`

`@inherits` ディレクティブは、ビューが継承するクラスの完全な制御を提供します。

```cshtml
@inherits TypeNameOfClassToInheritFrom
```

次のコードは、カスタム Razor ページの種類です。

[!code-csharp[](razor/sample/Classes/CustomRazorPage.cs)]

`CustomText` がビューに表示されます。

[!code-cshtml[](razor/sample/Views/Home/Contact10.cshtml)]

このコードでは、次のような HTML がレンダリングされます。

```html
<div>
    Custom text: Gardyloo! - A Scottish warning yelled from a window before dumping
    a slop bucket on the street below.
</div>
```

 `@model` と `@inherits` は同じビューで使うことができます。 `@inherits` は、ビューがインポートする *_ViewImports.cshtml* ファイルで指定できます。

[!code-cshtml[](razor/sample/Views/_ViewImportsModel.cshtml)]

次のコードは、厳密に型指定されたビューの例です。

[!code-cshtml[](razor/sample/Views/Home/Login1.cshtml)]

モデルに rick@contoso.com が渡された場合、ビューは次の HTML マークアップを生成します。

```html
<div>The Login Email: rick@contoso.com</div>
<div>
    Custom text: Gardyloo! - A Scottish warning yelled from a window before dumping
    a slop bucket on the street below.
</div>
```

### `@inject`

ディレクティブを使用すると、サービス `@inject` Razor [コンテナー](xref:fundamentals/dependency-injection) からビューにサービスを挿入できます。 詳しくは、「[ビューへの依存関係の挿入](xref:mvc/views/dependency-injection)」をご覧ください。

::: moniker range=">= aspnetcore-3.0"

### `@layout`

*このシナリオは、 Razor コンポーネント (razor) にのみ適用されます。*

ディレクティブは、 `@layout` ディレクティブを持つルーティング可能なコンポーネントのレイアウトを指定し Razor [`@page`](#page) ます。 レイアウト コンポーネントは、コードの重複や不整合を回避するために使用されます。 詳細については、「<xref:blazor/layouts>」を参照してください。

::: moniker-end

### `@model`

*このシナリオは、MVC ビューおよび Razor ページ (cshtml) にのみ適用されます。*

`@model` ディレクティブにより、ビューまたはページに渡されるモデルの型が指定されます。

```cshtml
@model TypeNameOfModel
```

Razor個々のユーザーアカウントで作成された ASP.NET CORE MVC またはページアプリでは、 *Views/Account/Login. cshtml* には次のモデル宣言が含まれています。

```cshtml
@model LoginViewModel
```

生成されるクラスは、`RazorPage<dynamic>` を継承します。

```csharp
public class _Views_Account_Login_cshtml : RazorPage<LoginViewModel>
```

Razor`Model`ビューに渡されるモデルにアクセスするためのプロパティを公開します。

```cshtml
<div>The Login Email: @Model.Email</div>
```

`@model` ディレクティブにより、`Model` プロパティの型が指定されます。 ディレクティブでは、ビューが派生する生成されたクラスの `T` を `RazorPage<T>` で指定します。 `@model` ディレクティブが指定されていない場合、`Model` プロパティは `dynamic` 型になります。 詳細については、「 [厳密に型指定されたモデルと @model キーワード](xref:tutorials/first-mvc-app/adding-model#strongly-typed-models-and-the--keyword)」を参照してください。

### `@namespace`

`@namespace` ディレクティブ:

* 生成された Razor ページ、MVC ビュー、またはコンポーネントのクラスの名前空間を設定し Razor ます。
* ページ、ビュー、またはコンポーネントクラスのルート派生名前空間を、ディレクトリツリー内の最も近いインポートファイル、 *_ViewImports* (ビューまたはページ)、または *_Imports razor* ( Razor コンポーネント) に設定します。

```cshtml
@namespace Your.Namespace.Here
```

次の表に示すページの例については、「」を参照して Razor ください。

* 各ページで *Pages/_ViewImports.cshtml* がインポートされます。
* *Pages/_ViewImports.cshtml* に `@namespace Hello.World` が含まれます。
* 各ページには、その名前空間のルートとして `Hello.World` が含まれます。

| ページ                                        | 名前空間                             |
| ------------------------------------------- | ------------------------------------- |
| *Pages/Index. cshtml*                        | `Hello.World`                         |
| *Pages/MorePages/Page.cshtml*               | `Hello.World.MorePages`               |
| *Pages/MorePages/EvenMorePages/Page.cshtml* | `Hello.World.MorePages.EvenMorePages` |

前のリレーションシップは、MVC ビューおよびコンポーネントで使用されるファイルのインポートに適用され Razor ます。

複数のインポート ファイルに `@namespace` ディレクティブがあるとき、ディレクトリ ツリーでページ、ビュー、またはコンポーネントに最も近いファイルがルート名前空間の設定に使用されます。

前の例の *EvenMorePages* フォルダーに `@namespace Another.Planet` が含まれるインポート ファイルがある場合 (または、*Pages/MorePages/EvenMorePages/Page.cshtml* ファイルに `@namespace Another.Planet` が含まれる場合)、結果は次の表のようになります。

| ページ                                        | 名前空間               |
| ------------------------------------------- | ----------------------- |
| *Pages/Index. cshtml*                        | `Hello.World`           |
| *Pages/MorePages/Page.cshtml*               | `Hello.World.MorePages` |
| *Pages/MorePages/EvenMorePages/Page.cshtml* | `Another.Planet`        |

### `@page`

::: moniker range=">= aspnetcore-3.0"

`@page` ディレクティブには、それが表示されるファイルの型によって、さまざまな効果があります。 ディレクティブ:

* は、ファイルがページであることを示し *ます。* Razor 詳細については、「[カスタム ルート](xref:razor-pages/index#custom-routes)」と「<xref:razor-pages/index>」を参照してください。
* コンポーネントが要求を直接処理する必要があることを指定し Razor ます。 詳細については、「<xref:blazor/fundamentals/routing>」を参照してください。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

`@page`ファイルの先頭行にあるディレクティブは、ファイルがページであることを示し *ます。* Razor 詳細については、「<xref:razor-pages/index>」を参照してください。

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

### `@preservewhitespace`

*このシナリオは、コンポーネント () にのみ適用さ Razor `.razor` れます。*

`false`(既定値) に設定した場合、コンポーネント () から表示されるマークアップ内の空白は、次の場合に削除され Razor `.razor` ます。

* 要素内の先頭または末尾。
* `RenderFragment` パラメーター内の先頭または末尾。 別のコンポーネントに渡される子コンテンツなどです。
* `@if` または `@foreach` のような、C# コード ブロックの前か後にある。

::: moniker-end

### `@section`

*このシナリオは、MVC ビューおよび Razor ページ (cshtml) にのみ適用されます。*

ディレクティブは、 `@section` ビューまたはページが HTML ページのさまざまな部分でコンテンツを表示できるようにするために、 [MVC および Razor ページレイアウト](xref:mvc/views/layout) と組み合わせて使用されます。 詳細については、「<xref:mvc/views/layout>」を参照してください。

### `@using`

`@using` ディレクティブは、生成されるビューに C# の `using` ディレクティブを追加します。

[!code-cshtml[](razor/sample/Views/Home/Contact9.cshtml)]

::: moniker range=">= aspnetcore-3.0"

[ Razor コンポーネント](xref:blazor/components/index)で `@using` は、スコープ内のコンポーネントも制御します。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

## <a name="directive-attributes"></a>ディレクティブ属性

Razor ディレクティブ属性は、記号の後に予約済みのキーワードを使用した暗黙的な式によって表され `@` ます。 通常、ディレクティブ属性は、要素の解析方法を変更したり、さまざまな機能を有効にしたりします。

### `@attributes`

*このシナリオは、 Razor コンポーネント (razor) にのみ適用されます。*

`@attributes` では、非宣言属性のレンダリングがコンポーネントに許可されます。 詳細については、「<xref:blazor/components/index#attribute-splatting-and-arbitrary-parameters>」を参照してください。

### `@bind`

*このシナリオは、 Razor コンポーネント (razor) にのみ適用されます。*

コンポーネントのデータ バインドは、`@bind` 属性によって実現されます。 詳細については、「<xref:blazor/components/data-binding>」を参照してください。

### `@on{EVENT}`

*このシナリオは、 Razor コンポーネント (razor) にのみ適用されます。*

Razor コンポーネントのイベント処理機能を提供します。 詳細については、「<xref:blazor/components/event-handling>」を参照してください。

::: moniker-end

::: moniker range=">= aspnetcore-3.1"

### `@on{EVENT}:preventDefault`

*このシナリオは、 Razor コンポーネント (razor) にのみ適用されます。*

イベントの既定のアクションを禁止します。

### `@on{EVENT}:stopPropagation`

*このシナリオは、 Razor コンポーネント (razor) にのみ適用されます。*

イベントのイベント伝達を停止します。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

### `@key`

*このシナリオは、 Razor コンポーネント (razor) にのみ適用されます。*

`@key` ディレクティブ属性により、コンポーネントの比較アルゴリズムは、キーの値に基づいて要素またはコンポーネントの保存を保証します。 詳細については、「<xref:blazor/components/index#use-key-to-control-the-preservation-of-elements-and-components>」を参照してください。

### `@ref`

*このシナリオは、 Razor コンポーネント (razor) にのみ適用されます。*

コンポーネント参照 (`@ref`) からは、コンポーネント インスタンスにコマンドを発行できるように、そのインスタンスを参照する方法が与えられます。 詳細については、「<xref:blazor/components/index#capture-references-to-components>」を参照してください。

### `@typeparam`

*このシナリオは、 Razor コンポーネント (razor) にのみ適用されます。*

`@typeparam` ディレクティブによって、生成されるコンポーネント クラスのジェネリック型パラメーターを宣言します。 詳細については、「<xref:blazor/components/templated-components>」を参照してください。

::: moniker-end

## <a name="templated-razor-delegates"></a>テンプレート化 Razor デリゲート

Razor テンプレートを使用すると、次の形式で UI スニペットを定義できます。

```cshtml
@<tag>...</tag>
```

次の例は、テンプレート化されたデリゲートをとして指定する方法を示してい Razor <xref:System.Func%602> ます。 デリゲートによってカプセル化されるメソッドのパラメーターに対しては、[dynamic 型](/dotnet/csharp/programming-guide/types/using-type-dynamic)を指定します。 デリゲートの戻り値としては、[object 型](/dotnet/csharp/language-reference/keywords/object)を指定します。 テンプレートは、`Name` プロパティを持つ `Pet` の <xref:System.Collections.Generic.List%601> で使用されます。

```csharp
public class Pet
{
    public string Name { get; set; }
}
```

```cshtml
@{
    Func<dynamic, object> petTemplate = @<p>You have a pet named <strong>@item.Name</strong>.</p>;

    var pets = new List<Pet>
    {
        new Pet { Name = "Rin Tin Tin" },
        new Pet { Name = "Mr. Bigglesworth" },
        new Pet { Name = "K-9" }
    };
}
```

テンプレートは、`foreach` ステートメントによって提供される `pets` で表示されます。

```cshtml
@foreach (var pet in pets)
{
    @petTemplate(pet)
}
```

表示される出力:

```html
<p>You have a pet named <strong>Rin Tin Tin</strong>.</p>
<p>You have a pet named <strong>Mr. Bigglesworth</strong>.</p>
<p>You have a pet named <strong>K-9</strong>.</p>
```

Razorメソッドの引数としてインラインテンプレートを指定することもできます。 次の例では、 `Repeat` メソッドがテンプレートを受け取り Razor ます。 メソッドは、テンプレートを使用して、リストから提供される項目の繰り返しで HTML コンテンツを生成します。

```cshtml
@using Microsoft.AspNetCore.Html

@functions {
    public static IHtmlContent Repeat(IEnumerable<dynamic> items, int times,
        Func<dynamic, IHtmlContent> template)
    {
        var html = new HtmlContentBuilder();

        foreach (var item in items)
        {
            for (var i = 0; i < times; i++)
            {
                html.AppendHtml(template(item));
            }
        }

        return html;
    }
}
```

前の例のペットのリストを使用して、次のように `Repeat` メソッドを呼び出します。

* <xref:System.Collections.Generic.List%601> の `Pet`。
* 各ペットを繰り返す回数。
* 順不同のリストのリスト項目に対して使用するインライン テンプレート。

```cshtml
<ul>
    @Repeat(pets, 3, @<li>@item.Name</li>)
</ul>
```

表示される出力:

```html
<ul>
    <li>Rin Tin Tin</li>
    <li>Rin Tin Tin</li>
    <li>Rin Tin Tin</li>
    <li>Mr. Bigglesworth</li>
    <li>Mr. Bigglesworth</li>
    <li>Mr. Bigglesworth</li>
    <li>K-9</li>
    <li>K-9</li>
    <li>K-9</li>
</ul>
```

## <a name="tag-helpers"></a>タグ ヘルパー

*このシナリオは、MVC ビューおよび Razor ページ (cshtml) にのみ適用されます。*

[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)に関する 3 つのディレクティブがあります。

| ディレクティブ | 機能 |
| --------- | -------- |
| [`@addTagHelper`](xref:mvc/views/tag-helpers/intro#add-helper-label) | ビューでタグ ヘルパーを使えるようにします。 |
| [`@removeTagHelper`](xref:mvc/views/tag-helpers/intro#remove-razor-directives-label) | 前に追加したタグ ヘルパーをビューから削除します。 |
| [`@tagHelperPrefix`](xref:mvc/views/tag-helpers/intro#prefix-razor-directives-label) | タグ プレフィックスを指定して、タグ ヘルパーのサポートを有効にしたり、タグ ヘルパーの使用を明示的にしたりします。 |

## <a name="razor-reserved-keywords"></a>Razor 予約済みキーワード

### <a name="razor-keywords"></a>Razor keywords

* `page` (ASP.NET Core 2.1 以降が必要)
* `namespace`
* `functions`
* `inherits`
* `model`
* `section`
* `helper` (現在 ASP.NET Core ではサポートされていません)

Razor キーワードは、でエスケープされ `@(Razor Keyword)` ます (たとえば、 `@(functions)` )。

### <a name="c-razor-keywords"></a>C# の Razor キーワード

* `case`
* `do`
* `default`
* `for`
* `foreach`
* `if`
* `else`
* `lock`
* `switch`
* `try`
* `catch`
* `finally`
* `using`
* `while`

C# の Razor キーワードは、を使用してダブルエスケープする必要があり `@(@C# Razor Keyword)` ます (例: `@(@case)` )。 最初のは、 `@` パーサーをエスケープし Razor ます。 2 番目の `@` は、C# パーサーをエスケープします。

### <a name="reserved-keywords-not-used-by-razor"></a>予約済みキーワードがで使用されていません Razor

* `class`

## <a name="inspect-the-razor-c-class-generated-for-a-view"></a>Razorビューに対して生成された C# クラスを検査する

::: moniker range=">= aspnetcore-2.1"

.NET Core SDK 2.1 以降では、 [ Razor SDK](xref:razor-pages/sdk)によってファイルのコンパイルが処理さ Razor れます。 プロジェクトをビルドすると、SDK によって、 Razor プロジェクトルートに *obj/<build_configuration>Razor /<target_framework_moniker/* ディレクトリが生成されます。 ディレクトリ内のディレクトリ構造は、 *Razor* プロジェクトのディレクトリ構造をミラー化します。

.NET Core 2.1 を対象とする ASP.NET Core 2.1 ページプロジェクトでは、次のディレクトリ構造について考えてみ Razor ます。

```
 Areas/
   Admin/
     Pages/
       Index.cshtml
       Index.cshtml.cs
 Pages/
   Shared/
     _Layout.cshtml
   _ViewImports.cshtml
   _ViewStart.cshtml
   Index.cshtml
   Index.cshtml.cs
  ```

*Debug* 構成でプロジェクトを作成すると、次の *obj* ディレクトリが生成されます。

```
 obj/
   Debug/
     netcoreapp2.1/
       Razor/
         Areas/
           Admin/
             Pages/
               Index.g.cshtml.cs
         Pages/
           Shared/
             _Layout.g.cshtml.cs
           _ViewImports.g.cshtml.cs
           _ViewStart.g.cshtml.cs
           Index.g.cshtml.cs
```

*Pages/Index. cshtml* の生成されたクラスを表示するには、 *obj/Debug/netcoreapp 2.1/ Razor /Pages/Index.g.cshtml.cs* を開きます。

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

次のクラスを ASP.NET Core MVC プロジェクトに追加します。

[!code-csharp[](razor/sample/Utilities/CustomTemplateEngine.cs)]

`Startup.ConfigureServices` で、MVC によって追加された `RazorTemplateEngine` を `CustomTemplateEngine` クラスでオーバーライドします。

[!code-csharp[](razor/sample/Startup.cs?highlight=4&range=10-14)]

`CustomTemplateEngine` の `return csharpDocument;` ステートメントにブレークポイントを設定します。 プログラムの実行がブレークポイントで停止したら、`generatedCode` の値を表示します。

![generatedCode のテキスト ビジュアライザーの表示](razor/_static/tvr.png)

::: moniker-end

## <a name="view-lookups-and-case-sensitivity"></a>ビューの参照と大文字/小文字の区別

Razorビューエンジンは、ビューに対して大文字と小文字を区別して検索を実行します。 ただし、実際の参照は、基になるファイル システムによって決定されます。

* ファイル ベースのソース:
  * 大文字と小文字が区別されないファイル システムを使っているオペレーティング システム (Windows など) では、物理的なファイル プロバイダーの参照は大文字と小文字を区別しません。 たとえば、`return View("Test")` は、*/Views/Home/Test.cshtml*、*/Views/home/test.cshtml*、その他のすべての大文字と小文字のバリエーションと一致します。
  * 大文字と小文字が区別されるファイル システム (たとえば、Linux、OSX、および `EmbeddedFileProvider`) では、参照は大文字と小文字を区別します。 たとえば、`return View("Test")` は */Views/Home/Test.cshtml* だけと一致します。
* プリコンパイル済みのビュー: ASP.NET Core 2.0 以降では、プリコンパイル済みのビューの参照は、すべてのオペレーティング システムで大文字と小文字を区別しません。 動作は、Windows での物理ファイル プロバイダーの動作と同じです。 2 つのプリコンパイル済みビューの相違点が大文字と小文字の使い分けだけの場合、参照の結果はどちらになるかわかりません。

開発者には、ファイル名とディレクトリ名の大文字/小文字の使い分けを、次のものと一致させることをお勧めします。

* 領域、コントローラー、アクションの名前。
* Razor トピック.

大文字と小文字の使い分けを一致させると、展開は基になっているファイル システムに関係なくビューを検索できます。

## <a name="additional-resources"></a>その他のリソース

を[使用した ASP.NET Web プログラミング Razor の概要構文には、](/aspnet/web-pages/overview/getting-started/introducing-razor-syntax-c)構文を使用したプログラミングの多くのサンプルが用意されて Razor います。
