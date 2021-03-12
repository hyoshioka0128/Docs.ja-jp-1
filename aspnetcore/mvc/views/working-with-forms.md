---
title: ASP.NET Core のフォームのタグ ヘルパー
author: rick-anderson
description: フォームで使用される組み込みのタグ ヘルパーについて説明します。
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
uid: mvc/views/working-with-forms
ms.openlocfilehash: 51e5f2f74493e7f4c18273c8589ed0424a1f2cac
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102585982"
---
# <a name="tag-helpers-in-forms-in-aspnet-core"></a>ASP.NET Core のフォームのタグ ヘルパー

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)、[N. Taylor Mullen](https://github.com/NTaylorMullen)、[Dave Paquette](https://twitter.com/Dave_Paquette)、[Jerrie Pelser](https://github.com/jerriep)

このドキュメントでは、フォームとフォームでよく使用される HTML 要素の使用方法について説明します。 HTML の [Form](https://www.w3.org/TR/html401/interact/forms.html) 要素には、Web アプリケーションからサーバーにデータをポスト バックするために使用する主要なメカニズムがあります。 このドキュメントでは、[タグ ヘルパー](tag-helpers/intro.md)と、タグ ヘルパーを利用して堅牢な HTML フォームを生産的に作成する方法について主に説明します。 このドキュメントを読む前に、[タグ ヘルパーの概要](tag-helpers/intro.md)に関するページを読むことをお勧めします。

多くの場合、HTML ヘルパーでは特定のタグ ヘルパーの代替方法が提供されますが、タグ ヘルパーを HTML ヘルパーの代わりに使用することはできないことと、各 HTML ヘルパーに対応するタグ ヘルパーがないことを認識することが重要です。 HTML ヘルパーの代替が存在する場合は記載します。

<a name="my-asp-route-param-ref-label"></a>

## <a name="the-form-tag-helper"></a>フォーム タグ ヘルパー

[フォーム](https://www.w3.org/TR/html401/interact/forms.html)タグヘルパー:

* [\<FORM>](https://www.w3.org/TR/html401/interact/forms.html) `action` MVC コントローラーアクションまたは名前付きルートの HTML 属性値を生成します

* クロスサイト リクエスト フォージェリを防ぐために、非表示の[要求検証トークン](/aspnet/mvc/overview/security/xsrfcsrf-prevention-in-aspnet-mvc-and-web-pages)を生成します (HTTP POST アクション メソッドで `[ValidateAntiForgeryToken]` 属性と共に使用する場合)

* `asp-route-<Parameter Name>` 属性を提供します (`<Parameter Name>` がルート値に追加される場合)。 `Html.BeginForm` および `Html.BeginRouteForm` に `routeValues` パラメーターを指定すると、同様の機能が提供されます。

* HTML ヘルパーの代替の `Html.BeginForm` と `Html.BeginRouteForm` があります

サンプル:

[!code-cshtml[](working-with-forms/sample/final/Views/Demo/RegisterFormOnly.cshtml)]

上記のフォーム タグ ヘルパーで、次の HTML が生成されます。

```html
<form method="post" action="/Demo/Register">
    <!-- Input and Submit elements -->
    <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
</form>
```

MVC ランタイムで、フォーム タグ ヘルパーの属性 `asp-controller` と `asp-action` から `action` 属性値が生成されます。 また、フォーム タグ ヘルパーも、クロスサイト リクエスト フォージェリを防ぐために、非表示の[要求検証トークン](/aspnet/mvc/overview/security/xsrfcsrf-prevention-in-aspnet-mvc-and-web-pages)を生成します (HTTP POST アクション メソッドで `[ValidateAntiForgeryToken]` 属性と共に使用する場合)。 純粋な HTML フォームをクロスサイト リクエスト フォージェリから保護することは難しいため、フォーム タグ ヘルパーが提供するサービスを利用してください。

### <a name="using-a-named-route"></a>名前付きのルートの使用

`asp-route` タグ ヘルパー属性で、HTML `action` 属性のマークアップを生成することもできます。 `register` という名前の[ルート](../../fundamentals/routing.md)を持つアプリケーションは、登録ページに次のマークアップを使用できます。

[!code-cshtml[](../../mvc/views/working-with-forms/sample/final/Views/Demo/RegisterRoute.cshtml)]

*Views/Account* フォルダー (*個々のユーザー アカウント* を使用して新しい Web アプリケーションを作成するときに生成されるフォルダー) のビューの多くには、[asp-route-returnurl](xref:mvc/views/working-with-forms) 属性が含まれています。

```cshtml
<form asp-controller="Account" asp-action="Login"
     asp-route-returnurl="@ViewData["ReturnUrl"]"
     method="post" class="form-horizontal" role="form">
```

>[!NOTE]
>承認済みで認証されていないリソースまたは承認されていないリソースにアクセスしようとすると、組み込みのテンプレートを使用して、`returnUrl` のみが自動的に設定されます。 未承認のアクセスを試行すると、セキュリティ ミドルウェアによって、`returnUrl` が設定されたログイン ページにリダイレクトされます。

## <a name="the-form-action-tag-helper"></a>フォーム アクション タグ ヘルパー

フォーム アクション タグ ヘルパーにより、生成された`<button ...>` または `<input type="image" ...>` タグ上に `formaction` 属性が生成されます。 `formaction` 属性では、フォームがそのデータを送信する場所を制御します。 [\<input>](https://www.w3.org/wiki/HTML/Elements/input)型および要素の要素にバインドさ `image` [\<button>](https://www.w3.org/wiki/HTML/Elements/button) れます。 フォーム アクション タグ ヘルパーにより、[AnchorTagHelper の ](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) `asp-` 属性を複数使うことが可能になり、対応する要素に向けて何の `formaction` リンクが生成されるかを制御できます。

`formaction` の値を制御するためにサポートされている [AnchorTagHelper](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) 属性:

|属性|説明|
|---|---|
|[asp-controller](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-controller)|コントローラーの名前。|
|[asp-action](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-action)|アクション メソッドの名前です。|
|[asp-area](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-area)|領域の名前です。|
|[asp-page](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-page)|ページの名前 Razor 。|
|[asp-page-handler](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-page-handler)|ページハンドラーの名前 Razor 。|
|[asp-route](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-route)|ルートの名前。|
|[asp-route-{value}](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-route-value)|単一の URL ルート値です。 たとえば、「 `asp-route-id="1234"` 」のように入力します。|
|[asp-all-route-data](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-all-route-data)|すべてのルート値です。|
|[asp-fragment](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-fragment)|URL フラグメントです。|

### <a name="submit-to-controller-example"></a>コントローラーに送信する例

次のマークアップでは、入力またはボタンが選択されたときに、`HomeController` の `Index` アクションにフォームを送信します。

```cshtml
<form method="post">
    <button asp-controller="Home" asp-action="Index">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" asp-controller="Home" 
                                asp-action="Index">
</form>
```

以前のマークアップでは次の HTML が生成されます。

```html
<form method="post">
    <button formaction="/Home">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" formaction="/Home">
</form>
```

### <a name="submit-to-page-example"></a>ページに送信する例

次のマークアップは、フォームをページに送信し `About` Razor ます。

```cshtml
<form method="post">
    <button asp-page="About">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" asp-page="About">
</form>
```

以前のマークアップでは次の HTML が生成されます。

```html
<form method="post">
    <button formaction="/About">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" formaction="/About">
</form>
```

### <a name="submit-to-route-example"></a>ルートに送信する例

`/Home/Test` エンドポイントを検討します。

```csharp
public class HomeController : Controller
{
    [Route("/Home/Test", Name = "Custom")]
    public string Test()
    {
        return "This is the test page";
    }
}
```

次のマークアップでは、`/Home/Test` エンドポイントにフォームを送信します。

```cshtml
<form method="post">
    <button asp-route="Custom">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" asp-route="Custom">
</form>
```

以前のマークアップでは次の HTML が生成されます。

```html
<form method="post">
    <button formaction="/Home/Test">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" formaction="/Home/Test">
</form>
```

## <a name="the-input-tag-helper"></a>入力タグ ヘルパー

入力タグヘルパーは、 [\<input>](https://www.w3.org/wiki/HTML/Elements/input) razor ビューのモデル式に HTML 要素をバインドします。

構文:

```cshtml
<input asp-for="<Expression Name>">
```

入力タグ ヘルパー:

* `asp-for` 属性で指定された式の名前の `id` および `name` HTML 属性を生成します。 `asp-for="Property1.Property2"` は `m => m.Property1.Property2` に相当します。 式の名前は、`asp-for` 属性値に使用されるものです。 詳細については、「[式の名前](#expression-names)」セクションを参照してください。

* モデル プロパティに適用されているモデル型と[データ注釈](/dotnet/api/microsoft.aspnetcore.mvc.dataannotations.iattributeadapter)に基づいて HTML `type` 属性値を設定します

* HTML `type` 属性値が指定されている場合は、上書きしません

* モデル プロパティに適用された[データ注釈](/dotnet/api/microsoft.aspnetcore.mvc.dataannotations.iattributeadapter)属性から [HTML5](https://developer.mozilla.org/docs/Web/Guide/HTML/HTML5) 検証属性を生成します

* `Html.TextBoxFor` および `Html.EditorFor` と重複する HTML ヘルパー機能があります。 詳細については、「**入力タグ ヘルパーの代替となる HTML ヘルパー**」セクションを参照してください。

* 厳密な型指定を提供します。 プロパティの名前が変更され、タグ ヘルパーを更新しない場合は、次のようなエラーが表示されます。

  ```
  An error occurred during the compilation of a resource required to process
  this request. Please review the following specific error details and modify
  your source code appropriately.

  Type expected
   'RegisterViewModel' does not contain a definition for 'Email' and no
   extension method 'Email' accepting a first argument of type 'RegisterViewModel'
   could be found (are you missing a using directive or an assembly reference?)
  ```

`Input` タグ ヘルパーは、.NET 型に基づいて HTML `type` 属性を設定します。 次の表は、一般的な.NET 型と生成される HTML 型の一部をまとめたものです (すべての .NET 型を網羅した一覧ではありません)。

|.NET の種類|入力タイプ|
|---|---|
|Bool|type="checkbox"|
|String|type="text"|
|DateTime|type=["datetime-local"](https://developer.mozilla.org/docs/Web/HTML/Element/input/datetime-local)|
|Byte|type="number"|
|int|type="number"|
|Single、Double|type="number"|

次の表は、入力タグ ヘルパーが特定の入力の型にマップする一般的な[データ注釈](/dotnet/api/microsoft.aspnetcore.mvc.dataannotations.iattributeadapter)属性の一部をまとめたものです (すべての検証属性を網羅した一覧ではありません)。

|属性|入力タイプ|
|---|---|
|[EmailAddress]|type="email"|
|[Url]|type="url"|
|[HiddenInput]|type="hidden"|
|[Phone]|type="tel"|
|[DataType(DataType.Password)]|type="password"|
|[DataType(DataType.Date)]|type="date"|
|[DataType(DataType.Time)]|type="time"|

サンプル:

[!code-csharp[](working-with-forms/sample/final/ViewModels/RegisterViewModel.cs)]

[!code-cshtml[](working-with-forms/sample/final/Views/Demo/RegisterInput.cshtml)]

上記のコードで、次の HTML が生成されます。

```html
  <form method="post" action="/Demo/RegisterInput">
      Email:
      <input type="email" data-val="true"
             data-val-email="The Email Address field is not a valid email address."
             data-val-required="The Email Address field is required."
             id="Email" name="Email" value=""><br>
      Password:
      <input type="password" data-val="true"
             data-val-required="The Password field is required."
             id="Password" name="Password"><br>
      <button type="submit">Register</button>
      <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
   </form>
```

`Email` および `Password` プロパティに適用されたデータ注釈によって、モデルに関するメタデータが生成されます。 入力タグ ヘルパーはモデルのメタデータを使用し、[HTML5 の ](https://developer.mozilla.org/docs/Web/Guide/HTML/HTML5) `data-val-*` 属性を生成します ([モデルの検証](../models/validation.md)に関するページを参照してください)。 これらの属性に、入力フィールドにアタッチする検証コントロールを記述します。 これで、控えめな HTML5 と [jQuery](https://jquery.com/) の検証機能を提供します。 控えめな属性の形式はです `data-val-rule="Error Message"` 。ここで、rule は検証規則の名前 (、、など `data-val-required` `data-val-email` `data-val-maxlength` ) です。属性にエラーメッセージが指定されている場合は、属性の値として表示され `data-val-rule` ます。 `data-val-maxlength-max="1024"` など、ルールに関する追加の詳細情報を提供するフォームの属性 `data-val-ruleName-argumentName="argumentValue"` もあります。

### <a name="html-helper-alternatives-to-input-tag-helper"></a>入力タグ ヘルパーの代替となる HTML ヘルパー

`Html.TextBox`、`Html.TextBoxFor`、`Html.Editor`、`Html.EditorFor` には、入力タグ ヘルパーと重複する機能があります。 入力タグ ヘルパーでは `type` 属性が自動的に設定されますが、`Html.TextBox` と `Html.TextBoxFor` では自動設定されません。 `Html.Editor` と `Html.EditorFor` はコレクション、複雑なオブジェクト、テンプレートを処理しますが、入力タグ ヘルパーは処理しません。 入力タグ ヘルパーの `Html.EditorFor` と `Html.TextBoxFor` は厳密に型指定されていますが (ラムダ式を使用します)、`Html.TextBox` と `Html.Editor` は厳密に型指定されていません (式の名前を使用します)。

### <a name="htmlattributes"></a>HtmlAttributes

`@Html.Editor()` と `@Html.EditorFor()` は、既定のテンプレートを実行するときに `htmlAttributes` という名前の特殊な `ViewDataDictionary` エントリを使用します。 この動作は、必要に応じて `additionalViewData` パラメーターを使用して拡張されます。 キー "htmlAttributes" は大文字と小文字が区別されません。 キー "htmlAttributes" は、`htmlAttributes` のような入力ヘルパーに渡される `@Html.TextBox()` オブジェクトと同様に処理されます。

```cshtml
@Html.EditorFor(model => model.YourProperty, 
  new { htmlAttributes = new { @class="myCssClass", style="Width:100px" } })
```

### <a name="expression-names"></a>式の名前

`asp-for` 属性値は `ModelExpression` であり、ラムダ式の右辺です。 そのため、生成されるコードで `asp-for="Property1"` は `m => m.Property1` になります。したがって、`Model` をプレフィックスとして付加する必要はありません。 "\@" 文字を使用してインライン式を開始し、`m.` の前に移動できます。

```cshtml
@{
  var joe = "Joe";
}

<input asp-for="@joe">
```

以下が生成されます。

```html
<input type="text" id="joe" name="joe" value="Joe">
```

`i` の値が `23` の場合、`asp-for="CollectionProperty[23].Member"` はコレクションのプロパティを使用して、`asp-for="CollectionProperty[i].Member"` と同じ名前を生成します。

ASP.NET Core MVC で `ModelExpression` の値が計算されるとき、`ModelState` を含む、いくつかのソースが検査されます。 `<input type="text" asp-for="@Name">` を検討します。 計算された `value` 属性は、次のうちの最初の非 null 値です。

* キー "Name" を持つ `ModelState` エントリ。
* 式 `Model.Name` の結果。

### <a name="navigating-child-properties"></a>子プロパティの移動

ビュー モデルのプロパティ パスを使用して、子プロパティに移動することもできます。 子 `Address` プロパティを含むより複雑なモデル クラスを考えてみましょう。

[!code-csharp[](../../mvc/views/working-with-forms/sample/final/ViewModels/AddressViewModel.cs?highlight=1,2,3,4&range=5-8)]

[!code-csharp[](../../mvc/views/working-with-forms/sample/final/ViewModels/RegisterAddressViewModel.cs?highlight=8&range=5-13)]

このビューでは、`Address.AddressLine1` にバインドしています。

[!code-cshtml[](../../mvc/views/working-with-forms/sample/final/Views/Demo/RegisterAddress.cshtml?highlight=6)]

`Address.AddressLine1` に対して次の HTML が生成されます。

```html
<input type="text" id="Address_AddressLine1" name="Address.AddressLine1" value="">
```

### <a name="expression-names-and-collections"></a>式の名前とコレクション

`Colors` の配列を含むサンプル モデル:

[!code-csharp[](../../mvc/views/working-with-forms/sample/final/ViewModels/Person.cs?highlight=3&range=5-10)]

アクション メソッド:

```csharp
public IActionResult Edit(int id, int colorIndex)
{
    ViewData["Index"] = colorIndex;
    return View(GetPerson(id));
}
```

特定の Razor 要素にアクセスする方法を次に示し `Color` ます。

[!code-cshtml[](working-with-forms/sample/final/Views/Demo/EditColor.cshtml)]

*Views/Shared/EditorTemplates/String.cshtml* テンプレート:

[!code-cshtml[](working-with-forms/sample/final/Views/Shared/EditorTemplates/String.cshtml)]

`List<T>` を使用するサンプル:

[!code-csharp[](working-with-forms/sample/final/ViewModels/ToDoItem.cs?range=3-8)]

Razorコレクションを反復処理する方法を次に示します。

[!code-cshtml[](working-with-forms/sample/final/Views/Demo/Edit.cshtml)]

*Views/Shared/EditorTemplates/ToDoItem.cshtml* テンプレート:

[!code-cshtml[](working-with-forms/sample/final/Views/Shared/EditorTemplates/ToDoItem.cshtml)]

値を `asp-for` または `Html.DisplayFor` と同じコンテキストで使用する場合は、可能であれば `foreach` を使用する必要があります。 一般に、反復子を割り当てる必要がないため、(使用可能なシナリオでは) `for` の方が `foreach` よりも優れています。ただし、LINQ 式内でのインデクサーの評価はコストが高くなる可能性があるため、最小限に抑える必要があります。

&nbsp;

>[!NOTE]
>上記のコメント付きサンプル コードは、ラムダ式を `@` 演算子に置き換えて、リスト内の各 `ToDoItem` にアクセスする方法を示しています。

## <a name="the-textarea-tag-helper"></a>Textarea タグ ヘルパー

`Textarea Tag Helper` タグ ヘルパーは、入力タグ ヘルパーと似ています。

* `id` `name` 要素の属性と属性、およびモデルからのデータ検証属性を生成し [\<textarea>](https://www.w3.org/wiki/HTML/Elements/textarea) ます。

* 厳密な型指定を提供します。

* HTML ヘルパーの代替: `Html.TextAreaFor`

サンプル:

[!code-csharp[](working-with-forms/sample/final/ViewModels/DescriptionViewModel.cs)]

[!code-cshtml[](../../mvc/views/working-with-forms/sample/final/Views/Demo/RegisterTextArea.cshtml?highlight=4)]

次の HTML が生成されます。

```html
<form method="post" action="/Demo/RegisterTextArea">
  <textarea data-val="true"
   data-val-maxlength="The field Description must be a string or array type with a maximum length of &#x27;1024&#x27;."
   data-val-maxlength-max="1024"
   data-val-minlength="The field Description must be a string or array type with a minimum length of &#x27;5&#x27;."
   data-val-minlength-min="5"
   id="Description" name="Description">
  </textarea>
  <button type="submit">Test</button>
  <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
</form>
```

## <a name="the-label-tag-helper"></a>ラベル タグ ヘルパー

* `for` [\<label>](https://www.w3.org/wiki/HTML/Elements/label) 式名の要素にラベルのキャプションと属性を生成します

* HTML ヘルパーの代替: `Html.LabelFor`。

`Label Tag Helper` は、純粋な HTML の label 要素よりも次の点で優れています。

* `Display` 属性からわかりやすいラベル値が自動的に取得されます。 意図した表示名は時間と共に変化する可能性があります。また、`Display` 属性とラベル タグ ヘルパーの組み合わせによって、使用されているあらゆる場所に `Display` が適用されます。

* ソース コードのマークアップが少ない

* モデルのプロパティを使用した厳密な型指定。

サンプル:

[!code-csharp[](working-with-forms/sample/final/ViewModels/SimpleViewModel.cs)]

[!code-cshtml[](../../mvc/views/working-with-forms/sample/final/Views/Demo/RegisterLabel.cshtml?highlight=4)]

`<label>` 要素に対して次の HTML が生成されます。

```html
<label for="Email">Email Address</label>
```

ラベル タグ ヘルパーから "Email" の属性値 `for` が生成されました。これは、`<input>` に関連付けられた ID です。 タグ ヘルパーでは一貫性のある `id` および `for` 要素が生成されるので、正しく関連付けられます。 このサンプルのキャプションは `Display` 属性に由来します。 モデルに `Display` 属性を含めなかった場合、キャプションは式のプロパティ名になります。

## <a name="the-validation-tag-helpers"></a>検証タグ ヘルパー

2 つの検証タグ ヘルパーがあります。 `Validation Message Tag Helper` (モデルの単一のプロパティに関する検証メッセージを表示する) と `Validation Summary Tag Helper` (検証エラーの概要を表示する) です。 `Input Tag Helper` は、モデル クラスのデータ注釈属性に基づいて、HTML5 のクライアント側検証属性を input 要素に追加します。 検証はサーバー側でも実行されます。 検証エラーが発生すると、検証タグ ヘルパーによってこれらのエラー メッセージが表示されます。

### <a name="the-validation-message-tag-helper"></a>検証メッセージ タグ ヘルパー

* [HTML5 の ](https://developer.mozilla.org/docs/Web/Guide/HTML/HTML5)  `data-valmsg-for="property"` 属性を [span](https://developer.mozilla.org/docs/Web/HTML/Element/span) 要素に追加します。これによって、指定されたモデル プロパティの入力フィールドに検証エラー メッセージがアタッチされます。 クライアント側の検証エラーが発生すると、[jQuery](https://jquery.com/) によって `<span>` 要素のエラー メッセージが表示されます。

* 検証はサーバー側でも実行されます。 クライアントで JavaScript が無効にされている場合や、一部の検証をサーバー側でのみ実行できる場合があります。

* HTML ヘルパーの代替: `Html.ValidationMessageFor`

`Validation Message Tag Helper` は、HTML の [span](https://developer.mozilla.org/docs/Web/HTML/Element/span) 要素で `asp-validation-for` 属性と共に使用されます。

```cshtml
<span asp-validation-for="Email"></span>
```

検証メッセージ タグ ヘルパーから、次の HTML が生成されます。

```html
<span class="field-validation-valid"
  data-valmsg-for="Email"
  data-valmsg-replace="true"></span>
```

一般的に、同じプロパティの場合は、`Input` タグ ヘルパーの後に `Validation Message Tag Helper` を使用します。 こうすることで、エラーの原因となった入力の近くで検証エラー メッセージが表示されます。

> [!NOTE]
> クライアント側の検証のために、正しい JavaScript および [jQuery](https://jquery.com/) のスクリプト参照を使用したビューを用意する必要があります。 詳細については、「 [モデルの検証](../models/validation.md) 」を参照してください。

サーバー側の検証エラーが発生した場合 (カスタムのサーバー側の検証がある場合や、クライアント側の検証が無効な場合など)、MVC はそのエラー メッセージを `<span>` 要素の本文として配置します。

```html
<span class="field-validation-error" data-valmsg-for="Email"
            data-valmsg-replace="true">
   The Email Address field is required.
</span>
```

### <a name="the-validation-summary-tag-helper"></a>検証概要タグ ヘルパー

* `asp-validation-summary` 属性を持つ `<div>` 要素をターゲットとします

* HTML ヘルパーの代替: `@Html.ValidationSummary`

`Validation Summary Tag Helper` は、検証メッセージの概要を表示するために使用されます。 `asp-validation-summary` 属性値には、次のいずれかを指定できます。

|asp-validation-summary|検証メッセージが表示されます|
|--- |--- |
|ValidationSummary.All|プロパティとモデル レベル|
|ValidationSummary.ModelOnly|モデル|
|ValidationSummary.None|なし|

### <a name="sample"></a>サンプル

次の例のデータ モデルは、`DataAnnotation` 属性が設定されており、`<input>` 要素に関する検証エラー メッセージを生成します。  検証エラーが発生すると、検証タグ ヘルパーはエラー メッセージを表示します。

[!code-csharp[](working-with-forms/sample/final/ViewModels/RegisterViewModel.cs)]

[!code-cshtml[](../../mvc/views/working-with-forms/sample/final/Views/Demo/RegisterValidation.cshtml?highlight=4,6,8&range=1-10)]

生成される HTML (モデルが有効な場合):

```html
<form action="/DemoReg/Register" method="post">
  <div class="validation-summary-valid" data-valmsg-summary="true">
  <ul><li style="display:none"></li></ul></div>
  Email:  <input name="Email" id="Email" type="email" value=""
   data-val-required="The Email field is required."
   data-val-email="The Email field is not a valid email address."
   data-val="true"><br>
  <span class="field-validation-valid" data-valmsg-replace="true"
   data-valmsg-for="Email"></span><br>
  Password: <input name="Password" id="Password" type="password"
   data-val-required="The Password field is required." data-val="true"><br>
  <span class="field-validation-valid" data-valmsg-replace="true"
   data-valmsg-for="Password"></span><br>
  <button type="submit">Register</button>
  <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
</form>
```

## <a name="the-select-tag-helper"></a>選択タグ ヘルパー

* モデルのプロパティについて、[select](https://www.w3.org/wiki/HTML/Elements/select) 要素と、関連する [option](https://www.w3.org/wiki/HTML/Elements/option) 要素を生成します。

* HTML ヘルパーの代替の `Html.DropDownListFor` と `Html.ListBoxFor` があります

`Select Tag Helper` `asp-for` は [select](https://www.w3.org/wiki/HTML/Elements/select) 要素のモデル プロパティ名を指定し、`asp-items` は [option](https://www.w3.org/wiki/HTML/Elements/option) 要素を指定します。  次に例を示します。

[!code-cshtml[](working-with-forms/sample/final/Views/Home/Index.cshtml?range=4)]

サンプル:

[!code-csharp[](working-with-forms/sample/final/ViewModels/CountryViewModel.cs)]

`Index` メソッドは `CountryViewModel` を初期化し、選択された国を設定し、それを `Index` ビューに渡します。

[!code-csharp[](working-with-forms/sample/final/Controllers/HomeController.cs?range=8-13)]

HTTP POST `Index` メソッドによって選択内容が表示されます。

[!code-csharp[](working-with-forms/sample/final/Controllers/HomeController.cs?range=15-27)]

`Index` ビュー:

[!code-cshtml[](working-with-forms/sample/final/Views/Home/Index.cshtml?highlight=4)]

次の HTML が生成されます ("CA" が選択されている場合)。

```html
<form method="post" action="/">
     <select id="Country" name="Country">
       <option value="MX">Mexico</option>
       <option selected="selected" value="CA">Canada</option>
       <option value="US">USA</option>
     </select>
       <br /><button type="submit">Register</button>
     <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
   </form>
```

> [!NOTE]
> 選択タグ ヘルパーで `ViewBag` または `ViewData` を使用することはお勧めしません。 ビュー モデルは、MVC メタデータを提供する場合に堅牢性が高くなり、一般的にあまり問題にはなりません。

`asp-for` 属性値は特殊なケースであり、(`asp-items` などの) 他のタグ ヘルパー属性とは異なり、`Model` プレフィックスは必須ではありません

[!code-cshtml[](working-with-forms/sample/final/Views/Home/Index.cshtml?range=4)]

### <a name="enum-binding"></a>Enum バインディング

`<select>` を `enum` プロパティと組み合わせて使用し、`enum` 値から `SelectListItem` 要素を生成すると便利な場合がよくあります。

サンプル:

[!code-csharp[](working-with-forms/sample/final/ViewModels/CountryEnumViewModel.cs?range=3-7)]

[!code-csharp[](working-with-forms/sample/final/ViewModels/CountryEnum.cs)]

`GetEnumSelectList` メソッドは列挙型の場合に `SelectList` オブジェクトを生成します。

[!code-cshtml[](../../mvc/views/working-with-forms/sample/final/Views/Home/IndexEnum.cshtml?highlight=5)]

より高機能な UI にするために、列挙子リストを `Display` 属性でマークできます。

[!code-csharp[](working-with-forms/sample/final/ViewModels/CountryEnum.cs?highlight=5,7)]

次の HTML が生成されます。

```html
  <form method="post" action="/Home/IndexEnum">
         <select data-val="true" data-val-required="The EnumCountry field is required."
                 id="EnumCountry" name="EnumCountry">
             <option value="0">United Mexican States</option>
             <option value="1">United States of America</option>
             <option value="2">Canada</option>
             <option value="3">France</option>
             <option value="4">Germany</option>
             <option selected="selected" value="5">Spain</option>
         </select>
         <br /><button type="submit">Register</button>
         <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
    </form>
```

### <a name="option-group"></a>オプション グループ

HTML  [\<optgroup>](https://www.w3.org/wiki/HTML/Elements/optgroup) 要素は、ビューモデルに1つ以上のオブジェクトが含まれている場合に生成され `SelectListGroup` ます。

`CountryViewModelGroup` は `SelectListItem` 要素を "北米" グループと "ヨーロッパ" グループに分けます。

[!code-csharp[](../../mvc/views/working-with-forms/sample/final/ViewModels/CountryViewModelGroup.cs?highlight=5,6,14,20,26,32,38,44&range=6-56)]

この 2 つのグループを次に示します。

![オプション グループの例](working-with-forms/_static/grp.png)

生成される HTML:

```html
 <form method="post" action="/Home/IndexGroup">
      <select id="Country" name="Country">
          <optgroup label="North America">
              <option value="MEX">Mexico</option>
              <option value="CAN">Canada</option>
              <option value="US">USA</option>
          </optgroup>
          <optgroup label="Europe">
              <option value="FR">France</option>
              <option value="ES">Spain</option>
              <option value="DE">Germany</option>
          </optgroup>
      </select>
      <br /><button type="submit">Register</button>
      <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
 </form>
```

### <a name="multiple-select"></a>複数選択

`asp-for` 属性に指定されているプロパティが `IEnumerable` の場合、選択タグ ヘルパーは [multiple = "multiple"](https://w3c.github.io/html-reference/select.html) 属性を自動的に生成します。 たとえば、次のようなモデルがあるとします。

[!code-csharp[](../../mvc/views/working-with-forms/sample/final/ViewModels/CountryViewModelIEnumerable.cs?highlight=6)]

次のビューを対象にします。

[!code-cshtml[](../../mvc/views/working-with-forms/sample/final/Views/Home/IndexMultiSelect.cshtml?highlight=4)]

次の HTML が生成されます。

```html
<form method="post" action="/Home/IndexMultiSelect">
    <select id="CountryCodes"
    multiple="multiple"
    name="CountryCodes"><option value="MX">Mexico</option>
<option value="CA">Canada</option>
<option value="US">USA</option>
<option value="FR">France</option>
<option value="ES">Spain</option>
<option value="DE">Germany</option>
</select>
    <br /><button type="submit">Register</button>
  <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
</form>
```

### <a name="no-selection"></a>選択なし

複数のページで "未指定" オプションを使用しているとわかった場合は、テンプレートを作成して HTML の繰り返しを除去することができます。

[!code-cshtml[](../../mvc/views/working-with-forms/sample/final/Views/Home/IndexEmptyTemplate.cshtml?highlight=5)]

*Views/Shared/EditorTemplates/CountryViewModel.cshtml* テンプレート:

[!code-cshtml[](working-with-forms/sample/final/Views/Shared/EditorTemplates/CountryViewModel.cshtml)]

HTML [\<option>](https://www.w3.org/wiki/HTML/Elements/option) 要素の追加は、選択されて *いない* 場合にのみ制限されません。 たとえば、次のビューおよびアクション メソッドで、上記のコードのような HTML が生成されます。

[!code-csharp[](working-with-forms/sample/final/Controllers/HomeController.cs?name=snippetNone)]

[!code-cshtml[](working-with-forms/sample/final/Views/Home/IndexOption.cshtml)]

現在の `Country` 値に応じて、(`selected="selected"` 属性を含む) 正しい `<option>` 要素が選択されます。

[!code-csharp[](working-with-forms/sample/final/Controllers/HomeController.cs?range=114-119)]

```html
 <form method="post" action="/Home/IndexEmpty">
      <select id="Country" name="Country">
          <option value="">&lt;none&gt;</option>
          <option value="MX">Mexico</option>
          <option value="CA" selected="selected">Canada</option>
          <option value="US">USA</option>
      </select>
      <br /><button type="submit">Register</button>
   <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
 </form>
 ```

## <a name="additional-resources"></a>その他のリソース

* <xref:mvc/views/tag-helpers/intro>
* [HTML の Form 要素](https://www.w3.org/TR/html401/interact/forms.html)
* [要求検証トークン](/aspnet/mvc/overview/security/xsrfcsrf-prevention-in-aspnet-mvc-and-web-pages)
* <xref:mvc/models/model-binding>
* <xref:mvc/models/validation>
* [IAttributeAdapter インターフェイス](/dotnet/api/Microsoft.AspNetCore.Mvc.DataAnnotations.IAttributeAdapter)
* [このドキュメントのコード スニペット](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/working-with-forms/sample/final)
