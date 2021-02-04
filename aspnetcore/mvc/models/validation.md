---
title: ASP.NET Core MVC でのモデルの検証
author: rick-anderson
description: ASP.NET Core MVC とページでのモデルの検証について説明し Razor ます。
ms.author: riande
ms.custom: mvc
ms.date: 12/15/2019
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
uid: mvc/models/validation
ms.openlocfilehash: d6fa7e4524a8afdc23d4ad46354d9d8b395656a3
ms.sourcegitcommit: e311cfb77f26a0a23681019bd334929d1aaeda20
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99530191"
---
# <a name="model-validation-in-aspnet-core-mvc-and-razor-pages"></a>MVC とページ ASP.NET Core でのモデルの検証 Razor

::: moniker range=">= aspnetcore-3.0"

作成者: [Kirk Larkin](https://github.com/serpent5)

この記事では、ASP.NET Core MVC またはページアプリでユーザー入力を検証する方法について説明し Razor ます。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/validation/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="model-state"></a>モデルの状態

モデルの状態では、モデル バインドとモデル検証の 2 つのサブシステムで発生したエラーが表されます。 [モデルバインド](model-binding.md)から発生するエラーは、通常、データ変換エラーです。 たとえば、"x" は整数フィールドに入力されます。 モデルの検証は、モデル バインド後に行われ、データがビジネス ルールに準拠していないエラーを報告します。 たとえば、1 から 5 の評価を想定したフィールドに 0 が入力されたとします。

モデルバインドとモデル検証は、どちらもコントローラーアクションまたはページハンドラーメソッドの実行前に行わ Razor れます。 Web アプリでは、`ModelState.IsValid` を調べて適切に対処するのはアプリの責任です。 通常、Web アプリではエラー メッセージを含むページを再表示します。

[!code-csharp[](validation/samples/3.x/ValidationSample/Pages/Movies/Create.cshtml.cs?name=snippet_OnPostAsync&highlight=3-6)]

Web API コントローラーでは、`[ApiController]` 属性が設定されている場合は、`ModelState.IsValid` を確認する必要はありません。 その場合、モデルが無効な状態のときは、エラーの詳細を含む HTTP 400 応答が自動的に返されます。 詳細については、「[自動的な HTTP 400 応答](xref:web-api/index#automatic-http-400-responses)」を参照してください。

## <a name="rerun-validation"></a>検証を再実行する

検証は自動的に行われますが、手動で繰り返したい場合があります。 たとえば、プロパティの値を計算し、プロパティを計算値に設定した後で検証を再実行するような場合です。 検証を再実行するには、次に示すように `TryValidateModel` メソッドを呼び出します。

[!code-csharp[](validation/samples/3.x/ValidationSample/Pages/Movies/Create.cshtml.cs?name=snippet_TryValidate&highlight=3-6)]

## <a name="validation-attributes"></a>検証属性

検証属性を使用して、モデルのプロパティの検証規則を指定できます。 サンプル アプリの次の例では、検証属性で注釈されたモデル クラスが示されています。 `[ClassicMovie]` 属性はカスタム検証属性であり、その他は組み込み属性です。 `[ClassicMovieWithClientValidator]` は表示されません。 `[ClassicMovieWithClientValidator]` は、カスタム属性を実装するもう 1 つの方法を示しています。

[!code-csharp[](validation/samples/3.x/ValidationSample/Models/Movie.cs?name=snippet_Class)]

## <a name="built-in-attributes"></a>組み込みの属性

組み込み検証属性の一部を次に示します。

* `[CreditCard]`: プロパティにクレジットカード形式があることを検証します。 [JQuery 検証の追加のメソッド](https://cdnjs.cloudflare.com/ajax/libs/jquery-validate/1.19.1/additional-methods.min.js)が必要です。
* `[Compare]`: モデル内の2つのプロパティが一致することを検証します。
* `[EmailAddress]`: プロパティが電子メール形式であることを検証します。
* `[Phone]`: プロパティに電話番号の書式が設定されていることを検証します。
* `[Range]`: プロパティ値が指定した範囲内にあることを検証します。
* `[RegularExpression]`: プロパティ値が指定した正規表現に一致することを検証します。
* `[Required]`: フィールドが null でないことを検証します。 この属性の動作の詳細については、「 [ `[Required]` 属性](#required-attribute)」を参照してください。
* `[StringLength]`: 文字列プロパティの値が、指定された長さの制限を超えていないことを検証します。
* `[Url]`: プロパティに URL 形式があることを検証します。
* `[Remote]`: サーバーでアクションメソッドを呼び出すことによって、クライアントの入力を検証します。 この属性の動作の詳細については、「 [ `[Remote]` 属性](#remote-attribute)」を参照してください。

検証属性の完全な一覧については、[System.ComponentModel.DataAnnotations](xref:System.ComponentModel.DataAnnotations) 名前空間で確認できます。

### <a name="error-messages"></a>エラー メッセージ

検証属性では、無効な入力に対して表示されるエラー メッセージを指定できます。 次に例を示します。

```csharp
[StringLength(8, ErrorMessage = "Name length can't be more than 8.")]
```

内部的には、属性ではフィールド名のプレースホルダーを指定して `String.Format` が呼び出され、場合によっては追加のプレースホルダーが指定されます。 次に例を示します。

```csharp
[StringLength(8, ErrorMessage = "{0} length must be between {2} and {1}.", MinimumLength = 6)]
```

`Name` プロパティに適用すると、上記のコードによって作成されるエラー メッセージは "Name length must be between 6 and 8" になります。

特定の属性のエラー メッセージに対して `String.Format` に渡されるパラメーターを確認するには、[DataAnnotations のソース コード](https://github.com/dotnet/runtime/tree/master/src/libraries/System.ComponentModel.Annotations/src/System/ComponentModel/DataAnnotations)をご覧ください。

## <a name="non-nullable-reference-types-and-required-attribute"></a>Null 非許容の参照型と [Required] 属性

検証システムは、属性を持っているかのように、null 非許容パラメーターまたはバインドされたプロパティを扱い `[Required]` ます。 [ `Nullable` コンテキストを有効にする](/dotnet/csharp/nullable-references#nullable-contexts)ことで、MVC は属性で属性が付けられているかのように、null 非許容のプロパティまたはパラメーターの検証を暗黙的に開始し `[Required]` ます。 次のコードについて考えてみます。

```csharp
public class Person
{
    public string Name { get; set; }
}
```

アプリがでビルドされた場合 `<Nullable>enable</Nullable>` 、 `Name` JSON またはフォームの post での欠損値が検証エラーになります。 Null 値を許容する参照型を使用して、プロパティに null 値または欠損値を指定できるようにし `Name` ます。

```csharp
public class Person
{
    public string? Name { get; set; }
}
```

. `Startup.ConfigureServices` で <xref:Microsoft.AspNetCore.Mvc.MvcOptions.SuppressImplicitRequiredAttributeForNonNullableReferenceTypes> を次のように構成することで、この動作を無効にできます。

```csharp
services.AddControllers(options => options.SuppressImplicitRequiredAttributeForNonNullableReferenceTypes = true);
```

### <a name="required-validation-on-the-server"></a>サーバーでの [Required] の検証

サーバーでは、プロパティが null の場合、必須の値が不足しているものと見なされます。 null 非許容型フィールドは常に有効であり、`[Required]` 属性のエラー メッセージが表示されることはありません。

ただし、null 非許容型プロパティに対するモデル バインドは失敗する場合があり、`The value '' is invalid` などのエラー メッセージが表示されます。 null 非許容型のサーバー側検証に対してカスタム エラー メッセージを指定するには、次のオプションがあります。

* フィールドを null 許容型にします (たとえば、`decimal` の代わりに `decimal?` を使用)。 [Null \<T> ](/dotnet/csharp/programming-guide/nullable-types/)値を許容値型は、標準の null 許容型と同様に扱われます。
* 次の例に示すように、モデル バインドで使用される既定のエラー メッセージを指定します。

  [!code-csharp[](validation/samples/3.x/ValidationSample/Startup.cs?name=snippet_Configuration&highlight=5-6)]

  既定のメッセージを設定できるモデル バインド エラーに関して詳しくは、<xref:Microsoft.AspNetCore.Mvc.ModelBinding.Metadata.DefaultModelBindingMessageProvider#methods> をご覧ください。

### <a name="required-validation-on-the-client"></a>クライアントでの [Required] の検証

クライアントでの null 非許容型と文字列の処理は、サーバーとは異なります。 クライアント側 :

* 値は、入力が行われた場合にのみ存在するものと見なされます。 そのため、クライアント側の検証では、null 非許容型は null 許容型と同じように処理されます。
* 文字列フィールド内の空白文字は、jQuery Validation の [required](https://jqueryvalidation.org/required-method/) メソッドでは有効な入力と見なされます。 サーバー側の検証では、空白文字のみが入力された場合は、必須文字列フィールドが無効であるものと見なされます。

前述のように、null 非許容型は `[Required]` 属性を持つものとして処理されます。 つまり、`[Required]` 属性を適用していない場合でも、クライアント側の検証が行われます。 ただし、属性を使用しない場合は、既定のエラー メッセージが表示されます。 カスタム エラー メッセージを指定するには、属性を使用します。

## <a name="remote-attribute"></a>[Remote] 属性

`[Remote]` 属性では、フィールド入力が有効かどうかを判断するためにサーバーでメソッドを呼び出す必要があるクライアント側検証が実装されます。 たとえば、アプリでは、ユーザー名が既に使用されているかどうかを確認することが必要な場合があります。

リモート検証を実装するには:

1. JavaScript で呼び出すアクション メソッドを作成します。  JQuery Validation [リモート](https://jqueryvalidation.org/remote-method/) メソッドは JSON 応答を想定しています。

   * `true` は、入力データが有効であることを意味します。
   * `false`、`undefined`、または `null` は、入力が有効ではないことを意味します。 既定のエラー メッセージを表示します。
   * その他の文字列は、入力が無効であることを意味します。 カスタム エラー メッセージとして文字列を表示します。

   カスタム エラー メッセージを返すアクション メソッドの例を次に示します。

   [!code-csharp[](validation/samples/3.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyEmail)]

1. 次の例に示すように、モデル クラスで、検証アクション メソッドを指し示す `[Remote]` 属性を使用してプロパティに注釈を付けます。

   [!code-csharp[](validation/samples/3.x/ValidationSample/Models/User.cs?name=snippet_Email)]
 
   `[Remote]` 属性は `Microsoft.AspNetCore.Mvc` 名前空間にあります。
   
### <a name="additional-fields"></a>追加フィールド

`[Remote]` 属性の `AdditionalFields` プロパティでは、サーバー上のデータに対してフィールドの組み合わせを検証できます。 たとえば、`User` モデルに `FirstName` プロパティと `LastName` プロパティがある場合、その名前のペアを使用する既存ユーザーがいないことを確認したいことがあります。 `AdditionalFields` を使用する方法を次の例に示します。

[!code-csharp[](validation/samples/3.x/ValidationSample/Models/User.cs?name=snippet_Name&highlight=1,5)]

`AdditionalFields` を文字列 FirstName および LastName に明示的に設定することもできますが、[nameof](/dotnet/csharp/language-reference/keywords/nameof) 演算子を使用すると、後のリファクタリングが容易になります。 この検証のアクション メソッドは、`firstName` と `lastName` の両方の引数を受け入れる必要があります。

[!code-csharp[](validation/samples/3.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyName)]

ユーザーが名または姓を入力すると、JavaScript はリモート呼び出しを行って、その名前のペアが取得されているかどうかを確認します。

複数の追加フィールドを検証するには、それらをコンマ区切りのリストとして提供します。 たとえば、`MiddleName` プロパティをモデルに追加するには、`[Remote]` 属性を次の例のように設定します。

```csharp
[Remote(action: "VerifyName", controller: "Users", AdditionalFields = nameof(FirstName) + "," + nameof(LastName))]
public string MiddleName { get; set; }
```

他の属性引数と同じように、`AdditionalFields` も定数式である必要があります。 したがって、[補間文字列](/dotnet/csharp/language-reference/keywords/interpolated-strings)を使用したり、<xref:System.String.Join*> を呼び出して `AdditionalFields` を初期化したりしないでください。

## <a name="alternatives-to-built-in-attributes"></a>組み込み属性に代わる方法

組み込み属性によって提供されない検証が必要な場合は、次のようにできます。

* [カスタム属性を作成する](#custom-attributes)。
* [IValidatableObject を実装する](#ivalidatableobject)。

## <a name="custom-attributes"></a>カスタム属性

組み込みの検証属性で処理されないシナリオの場合は、カスタム検証属性を作成できます。 <xref:System.ComponentModel.DataAnnotations.ValidationAttribute> を継承するクラスを作成し、<xref:System.ComponentModel.DataAnnotations.ValidationAttribute.IsValid*> メソッドをオーバーライドします。

`IsValid` メソッドは、*value* という名前のオブジェクトを受け取ります。これは、検証対象の入力です。 オーバーロードは `ValidationContext` オブジェクトも受け取ります。これは、モデル バインドによって作成されたモデル インスタンスなどの追加情報を提供します。

次の例では、*Classic* ジャンルの映画の公開日が指定した年より後ではないことを検証します。 `[ClassicMovie]` 属性:

* サーバー上でのみ実行されます。
* クラシック映画の場合は、公開日が検証されます。

[!code-csharp[](validation/samples/3.x/ValidationSample/Validation/ClassicMovieAttribute.cs?name=snippet_Class)]

前の例の `movie` 変数は、フォーム送信からのデータを格納している `Movie` オブジェクトを表します。 検証に失敗すると、エラー メッセージを含む `ValidationResult` が返されます。

## <a name="ivalidatableobject"></a>IValidatableObject

前の例は、`Movie` 型でのみ動作します。 クラス レベルの検証に対するもう 1 つのオプションは、次の例に示すように、`IValidatableObject` をモデル クラスに実装することです。

[!code-csharp[](validation/samples/3.x/ValidationSample/Models/ValidatableMovie.cs?name=snippet_Class&highlight=1,26-34)]

## <a name="top-level-node-validation"></a>最上位ノードの検証

最上位ノードには次が含まれています。

* アクションのパラメーター
* コントローラーのプロパティ
* ページ ハンドラーのパラメーター
* ページ モデルのプロパティ

モデルのパラメーター検証に加え、モデルが関連付けられた最上位ノードが検証されます。 サンプル アプリからの次の例の `VerifyPhone` メソッドでは、<xref:System.ComponentModel.DataAnnotations.RegularExpressionAttribute> を使用して `phone` アクションパラメーターが検証されています。

[!code-csharp[](validation/samples/3.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyPhone)]

最上位ノードでは、検証属性と共に <xref:Microsoft.AspNetCore.Mvc.ModelBinding.BindRequiredAttribute> を使用できます。 サンプル アプリからの次の例では、`CheckAge` メソッドによって、フォームの送信時、クエリ文字列から `age` パラメーターを関連付ける必要があることが指定されます。

[!code-csharp[](validation/samples/3.x/ValidationSample/Controllers/UsersController.cs?name=snippet_CheckAgeSignature)]

[年齢確認] ページ (*CheckAge.cshtml*) には 2 つのフォームがあります。 最初のフォームでは、`Age` の値 `99` がクエリ文字列パラメーター `https://localhost:5001/Users/CheckAge?Age=99` として送信されます。

クエリ文字列の正しく書式設定された `age` パラメーターが送信されると、フォームの有効性が確認されます。

[年齢確認] ページの 2 番目のフォームでは、要求本文で `Age` 値が送信され、検証は不合格となります。 `age` パラメーターはクエリ文字列で渡される必要があるため、バインドが失敗します。

## <a name="maximum-errors"></a>最大エラー数

エラーの最大数 (既定では 200) に達すると、検証は停止します。 この数は、`Startup.ConfigureServices` の次のコードを使用して構成します。

[!code-csharp[](validation/samples/3.x/ValidationSample/Startup.cs?name=snippet_Configuration&highlight=4)]

## <a name="maximum-recursion"></a>最大再帰

<xref:Microsoft.AspNetCore.Mvc.ModelBinding.Validation.ValidationVisitor> では、検証対象のモデルのオブジェクト グラフが走査されます。 深いモデルまたは無限に再帰するモデルでは、検証でスタック オーバーフローが発生する可能性があります。 [MvcOptions.MaxValidationDepth](xref:Microsoft.AspNetCore.Mvc.MvcOptions.MaxValidationDepth) では、ビジターの再帰が構成されている深さを超えた場合、早い段階で検証を停止する方法が提供されています。 `MvcOptions.MaxValidationDepth` の既定値は 32 です。

## <a name="automatic-short-circuit"></a>自動省略

モデル グラフの検証が必要ない場合、検証は自動的に省略 (スキップ) されます。 ランタイムで検証がスキップされるオブジェクトとしては、プリミティブのコレクション (`byte[]`、`string[]`、`Dictionary<string, string>` など) や、検証コントロールを何も持たない複雑なオブジェクト グラフなどがあります。

## <a name="disable-validation"></a>検証を無効にする

検証を無効にするには:

1. どのフィールドも無効としてマークしない `IObjectModelValidator` の実装を作成します。

   [!code-csharp[](validation/samples/3.x/ValidationSample/Validation/NullObjectModelValidator.cs?name=snippet_Class)]

1. 依存関係挿入コンテナーで、次のコードを `Startup.ConfigureServices` に追加し、既定の `IObjectModelValidator` の実装を置き換えます。

   [!code-csharp[](validation/samples/3.x/ValidationSample/Startup.cs?name=snippet_DisableValidation)]

その場合でも、モデル バインドから発生するモデル状態エラーが表示される可能性があります。

## <a name="client-side-validation"></a>クライアント側の検証

クライアント側検証は、フォームが有効になるまで送信を許可しません。 送信ボタンをクリックすると、フォームの送信またはエラー メッセージの表示を行う JavaScript が実行されます。

クライアント側の検証を使用すると、フォームに入力エラーがある場合に、サーバーへの不要なラウンドトリップを回避できます。 *_Layout.cshtml* および *_ValidationScriptsPartial.cshtml* の次のスクリプト参照では、クライアント側の検証がサポートされています。

[!code-cshtml[](validation/samples/3.x/ValidationSample/Views/Shared/_Layout.cshtml?name=snippet_Scripts)]

[!code-cshtml[](validation/samples/3.x/ValidationSample/Views/Shared/_ValidationScriptsPartial.cshtml?name=snippet_Scripts)]

[Jquery の控えめな検証](https://github.com/aspnet/jquery-validation-unobtrusive)スクリプトは、広く使われている[jquery validation](https://jqueryvalidation.org/)プラグインを基盤とするカスタム Microsoft フロントエンドライブラリです。 jQuery Unobtrusive Validation を使用しないと、同じ検証ロジックを 2 か所でコーディングする必要があります。1 つはモデル プロパティでのサーバー側検証属性で、もう 1 つはクライアント側スクリプトです。 代わりに、[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)および [HTML ヘルパー](xref:mvc/views/overview)では、モデル プロパティの検証属性と型メタデータを使用して、検証の必要なフォーム要素に対する HTML 5 の `data-` 属性がレンダリングされます。 jQuery の控えめな検証では、属性を解析し、その `data-` ロジックを jQuery の検証に渡します。これにより、サーバー側の検証ロジックがクライアントに効果的に "コピー" されます。 次に示すように、タグ ヘルパーを使用して、クライアントで検証エラーを表示できます。

[!code-cshtml[](validation/samples/3.x/ValidationSample/Pages/Movies/Create.cshtml?name=snippet_ReleaseDate&highlight=3-4)]

上記のタグ ヘルパーでは、次の HTML がレンダリングされます。

```html
<div class="form-group">
    <label class="control-label" for="Movie_ReleaseDate">Release Date</label>
    <input class="form-control" type="date" data-val="true"
        data-val-required="The Release Date field is required."
        id="Movie_ReleaseDate" name="Movie.ReleaseDate" value="">
    <span class="text-danger field-validation-valid"
        data-valmsg-for="Movie.ReleaseDate" data-valmsg-replace="true"></span>
</div>
```

HTML 出力の `data-` 属性が、`Movie.ReleaseDate` プロパティの検証属性に対応していることに注意してください。 `data-val-required` 属性には、ユーザーが公開日フィールドを入力していない場合に表示されるエラー メッセージが含まれています。 jQuery の控えめな検証では、この値を jQuery Validation [required ()](https://jqueryvalidation.org/required-method/) メソッドに渡します。これにより、そのメッセージが付随する要素に表示され **\<span>** ます。

`[DataType]` 属性によってオーバーライドされていない限り、データ型の検証はプロパティの .NET 型に基づいて行われます。 ブラウザーには独自の既定のエラー メッセージがありますが、jQuery Validation Unobtrusive Validation パッケージでそれらのメッセージをオーバーライドできます。 `[DataType]` 属性と `[EmailAddress]` などのサブクラスを使用して、エラー メッセージを指定できます。

## <a name="unobtrusive-validation"></a>控えめな検証

控えめな検証については、[こちらの GitHub のイシュー](https://github.com/dotnet/AspNetCore.Docs/issues/1111)をご覧ください。

### <a name="add-validation-to-dynamic-forms"></a>動的なフォームに検証を追加する

jQuery の控えめな検証は、ページが最初に読み込まれるときに検証ロジックとパラメーターを jQuery 検証に渡します。 したがって、動的に生成されるフォームでは、検証は自動的には機能しません。 検証を有効にするには、作成直後に動的フォームを解析するよう、jQuery Unobtrusive Validation に指示します。 たとえば、次のコードでは、AJAX によって追加されるフォームでクライアント側検証が設定されます。

```javascript
$.get({
    url: "https://url/that/returns/a/form",
    dataType: "html",
    error: function(jqXHR, textStatus, errorThrown) {
        alert(textStatus + ": Couldn't add form. " + errorThrown);
    },
    success: function(newFormHTML) {
        var container = document.getElementById("form-container");
        container.insertAdjacentHTML("beforeend", newFormHTML);
        var forms = container.getElementsByTagName("form");
        var newForm = forms[forms.length - 1];
        $.validator.unobtrusive.parse(newForm);
    }
})
```

`$.validator.unobtrusive.parse()` メソッドには、その引数の 1 つで jQuery セレクターを指定します。 このメソッドは、そのセレクター内のフォームの `data-` 属性を解析するよう jQuery Unobtrusive Validation に指示します。 これらの属性の値は、jQuery Validation プラグインに渡されます。

### <a name="add-validation-to-dynamic-controls"></a>動的なコントロールに検証を追加する

`$.validator.unobtrusive.parse()` メソッドは、`<input>` や `<select/>` などの動的に生成される個々のコントロールではなく、フォーム全体に対して動作します。 フォームを再解析するには、次の例に示すように、フォームが前に解析されたときに追加された検証データを削除します。

```javascript
$.get({
    url: "https://url/that/returns/a/control",
    dataType: "html",
    error: function(jqXHR, textStatus, errorThrown) {
        alert(textStatus + ": Couldn't add control. " + errorThrown);
    },
    success: function(newInputHTML) {
        var form = document.getElementById("my-form");
        form.insertAdjacentHTML("beforeend", newInputHTML);
        $(form).removeData("validator")    // Added by jQuery Validation
               .removeData("unobtrusiveValidation");   // Added by jQuery Unobtrusive Validation
        $.validator.unobtrusive.parse(form);
    }
})
```

## <a name="custom-client-side-validation"></a>カスタム クライアント側検証

カスタムのクライアント側検証を行うには `data-` 、カスタム JQuery 検証アダプターを使用する HTML 属性を生成します。 次のサンプルのアダプター コードは、この記事で前に導入した `[ClassicMovie]` および `[ClassicMovieWithClientValidator]` 属性用に記述されたものです。

[!code-javascript[](validation/samples/3.x/ValidationSample/wwwroot/js/classicMovieValidator.js)]

アダプターの作成方法の詳細については、 [jQuery の検証](https://jqueryvalidation.org/documentation/)に関するドキュメントを参照してください。

特定のフィールドに対するアダプターの使用は、次のような `data-` 属性によってトリガーされます。

* 検証対象としてフィールドにフラグを設定します (`data-val="true"`)。
* 検証規則名とエラー メッセージ テキストを示します (例: `data-val-rulename="Error message."`)。
* 検証コントロールで必要なその他のパラメーターを提供します (例: `data-val-rulename-param1="value"`)。

次の例では、サンプル アプリの `ClassicMovie` 属性に対する `data-` 属性を示します。

```html
<input class="form-control" type="date"
    data-val="true"
    data-val-classicmovie="Classic movies must have a release year no later than 1960."
    data-val-classicmovie-year="1960"
    data-val-required="The Release Date field is required."
    id="Movie_ReleaseDate" name="Movie.ReleaseDate" value="">
```

前に説明したように、[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)と [HTML ヘルパー](xref:mvc/views/overview)では、検証属性からの情報を使用して `data-` 属性がレンダリングされます。 カスタム `data-` HTML 属性が作成されるようになるコードを記述するには、2 つのオプションがあります。

* `AttributeAdapterBase<TAttribute>` から派生するクラスと `IValidationAttributeAdapterProvider` を実装するクラスを作成し、属性とそのアダプターを DI に登録します。 この方法では[単一責任の原則](https://wikipedia.org/wiki/Single_responsibility_principle)に従って、サーバー関連の検証コードとクライアント関連の検証コードは別のクラスになります。 アダプターには、DI に登録されるため、必要な場合に DI 内の他のサービスがそれを使用できるという利点もあります。
* `ValidationAttribute` クラスで `IClientModelValidator` を実装します。 この方法は、属性でサーバー側の検証が何も行われず、DI からのサービスが必要ない場合に、適している可能性があります。

### <a name="attributeadapter-for-client-side-validation"></a>クライアント側検証用の AttributeAdapter

HTML に `data-` 属性をレンダリングするこの方法は、サンプル アプリの `ClassicMovie` 属性で使用されています。 この方法を使用してクライアント検証を追加するには、次のようにします。

1. カスタム検証属性の属性アダプター クラスを作成します。 <xref:Microsoft.AspNetCore.Mvc.DataAnnotations.AttributeAdapterBase%601>からクラスを派生させます。 次の例で示すように、レンダリングされた出力に `data-` 属性を追加する `AddValidation` メソッドを作成します。

   [!code-csharp[](validation/samples/3.x/ValidationSample/Validation/ClassicMovieAttributeAdapter.cs?name=snippet_Class)]

1. <xref:Microsoft.AspNetCore.Mvc.DataAnnotations.IValidationAttributeAdapterProvider> を実装するアダプター プロバイダー クラスを作成します。 次の例に示すように、`GetAttributeAdapter` メソッドで、カスタム属性をアダプターのコンストラクターに渡します。

   [!code-csharp[](validation/samples/3.x/ValidationSample/Validation/CustomValidationAttributeAdapterProvider.cs?name=snippet_Class)]

1. `Startup.ConfigureServices` でアダプター プロバイダーを DI に登録します。

   [!code-csharp[](validation/samples/3.x/ValidationSample/Startup.cs?name=snippet_Configuration&highlight=9-10)]

### <a name="iclientmodelvalidator-for-client-side-validation"></a>クライアント側検証用の IClientModelValidator

HTML に `data-` 属性をレンダリングするこの方法は、サンプル アプリの `ClassicMovieWithClientValidator` 属性で使用されています。 この方法を使用してクライアント検証を追加するには、次のようにします。

* カスタム検証属性で、`IClientModelValidator` インターフェイスを実装し、`AddValidation` メソッドを作成します。 次の例で示すように、`AddValidation` メソッドで、検証用の `data-` 属性を追加します。

  [!code-csharp[](validation/samples/3.x/ValidationSample/Validation/ClassicMovieWithClientValidatorAttribute.cs?name=snippet_Class)]

## <a name="disable-client-side-validation"></a>クライアント側検証を無効にする

次のコードは、ページ内のクライアント検証を無効にし Razor ます。

[!code-csharp[](validation/samples/3.x/ValidationSample/Startup.cs?name=snippet_DisableClientValidation&highlight=2-5)]

クライアント側検証を無効にするその他のオプション:

* すべての *.cshtml* ファイル内の `_ValidationScriptsPartial` への参照をコメントアウトします。
* *Pages\Shared\_ValidationScriptsPartial.cshtml* ファイルの内容を削除します。

上記の方法では、クライアント側でクラスライブラリを検証することはできません ASP.NET Core Identity Razor 。 詳細については、「<xref:security/authentication/scaffold-identity>」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* [System.ComponentModel.DataAnnotations 名前空間](xref:System.ComponentModel.DataAnnotations)
* [モデルバインド](model-binding.md)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

この記事では、ASP.NET Core MVC またはページアプリでユーザー入力を検証する方法について説明し Razor ます。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/validation/sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="model-state"></a>モデルの状態

モデルの状態では、モデル バインドとモデル検証の 2 つのサブシステムで発生したエラーが表されます。 [モデル バインド](model-binding.md)で発生するエラーは、一般に、データ変換エラーです (たとえば、整数が必要なフィールドに "x" が入力された場合)。 モデル検証は、モデル バインドの後で行われて、データがビジネス ルールに従っていないエラーが報告されます (たとえば、1 から 5 までのレーティングが必要なフィールドに 0 が入力された場合)。

モデルバインドと検証の両方が、コントローラーアクションまたはページハンドラーメソッドの実行前に行わ Razor れます。 Web アプリでは、`ModelState.IsValid` を調べて適切に対処するのはアプリの責任です。 通常、Web アプリではエラー メッセージを含むページを再表示します。

[!code-csharp[](validation/samples_snapshot/2.x/Create.cshtml.cs?name=snippet&highlight=3-6)]

Web API コントローラーでは、`[ApiController]` 属性が設定されている場合は、`ModelState.IsValid` を確認する必要はありません。 その場合、モデルが無効な状態のときは、エラーの詳細を含む HTTP 400 応答が自動的に返されます。 詳細については、「[自動的な HTTP 400 応答](xref:web-api/index#automatic-http-400-responses)」を参照してください。

## <a name="rerun-validation"></a>検証を再実行する

検証は自動的に行われますが、手動で繰り返したい場合があります。 たとえば、プロパティの値を計算し、プロパティを計算値に設定した後で検証を再実行するような場合です。 検証を再実行するには、次に示すように `TryValidateModel` メソッドを呼び出します。

[!code-csharp[](validation/samples/2.x/ValidationSample/Controllers/MoviesController.cs?name=snippet_TryValidateModel&highlight=11)]

## <a name="validation-attributes"></a>検証属性

検証属性を使用して、モデルのプロパティの検証規則を指定できます。 [サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/validation/sample)の次の例では、検証属性で注釈されたモデル クラスが示されています。 `[ClassicMovie]` 属性はカスタム検証属性であり、その他は組み込み属性です。 `[ClassicMovie2]` は示されていませんが、カスタム属性を実装するもう 1 つの方法を示します。

[!code-csharp[](validation/samples/2.x/ValidationSample/Models/Movie.cs?name=snippet_ModelClass)]

## <a name="built-in-attributes"></a>組み込みの属性

組み込みの検証属性には次のものがあります。

* `[CreditCard]`: プロパティにクレジットカード形式があることを検証します。
* `[Compare]`: モデル内の2つのプロパティが一致することを検証します。 たとえば、*Register.cshtml.cs* ファイルは `[Compare]` を使用して、入力された 2 つのパスワードが一致していることを検証します。 [スキャフォールディング Identity ](xref:security/authentication/scaffold-identity)を参照してください。
* `[EmailAddress]`: プロパティが電子メール形式であることを検証します。
* `[Phone]`: プロパティに電話番号の書式が設定されていることを検証します。
* `[Range]`: プロパティ値が指定した範囲内にあることを検証します。
* `[RegularExpression]`: プロパティ値が指定した正規表現に一致することを検証します。
* `[Required]`: フィールドが null でないことを検証します。 この属性の動作の詳細については、「 [ `[Required]` 属性](#required-attribute)」を参照してください。
* `[StringLength]`: 文字列プロパティの値が、指定された長さの制限を超えていないことを検証します。
* `[Url]`: プロパティに URL 形式があることを検証します。
* `[Remote]`: サーバーでアクションメソッドを呼び出すことによって、クライアントの入力を検証します。 この属性の動作の詳細については、「 [ `[Remote]` 属性](#remote-attribute)」を参照してください。

クライアント側の検証で `[RegularExpression]` 属性を使用する場合、regex はクライアントの JavaScript で実行されます。 これは、[ECMAScript](/dotnet/standard/base-types/regular-expression-options#ecmascript-matching-behavior) 一致の動作が使用されることを意味します。 詳細については、次を参照してください。[この GitHub の問題](https://github.com/dotnet/corefx/issues/42487)します。

検証属性の完全な一覧については、[System.ComponentModel.DataAnnotations](xref:System.ComponentModel.DataAnnotations) 名前空間で確認できます。

### <a name="error-messages"></a>エラー メッセージ

検証属性では、無効な入力に対して表示されるエラー メッセージを指定できます。 次に例を示します。

```csharp
[StringLength(8, ErrorMessage = "Name length can't be more than 8.")]
```

内部的には、属性ではフィールド名のプレースホルダーを指定して `String.Format` が呼び出され、場合によっては追加のプレースホルダーが指定されます。 次に例を示します。

```csharp
[StringLength(8, ErrorMessage = "{0} length must be between {2} and {1}.", MinimumLength = 6)]
```

`Name` プロパティに適用すると、上記のコードによって作成されるエラー メッセージは "Name length must be between 6 and 8" になります。

特定の属性のエラー メッセージに対して `String.Format` に渡されるパラメーターを確認するには、[DataAnnotations のソース コード](https://github.com/dotnet/corefx/tree/master/src/System.ComponentModel.Annotations/src/System/ComponentModel/DataAnnotations)をご覧ください。

## <a name="required-attribute"></a>[Required] 属性

既定では、検証システムでは null 非許容型のパラメーターまたはプロパティは `[Required]` 属性を持つものとして処理されます。 `decimal` や `int` などの[値の型](/dotnet/csharp/language-reference/keywords/value-types)は null 非許容型です。

### <a name="required-validation-on-the-server"></a>サーバーでの [Required] の検証

サーバーでは、プロパティが null の場合、必須の値が不足しているものと見なされます。 null 非許容型フィールドは常に有効であり、[Required] 属性のエラー メッセージが表示されることはありません。

ただし、null 非許容型プロパティに対するモデル バインドは失敗する場合があり、`The value '' is invalid` などのエラー メッセージが表示されます。 null 非許容型のサーバー側検証に対してカスタム エラー メッセージを指定するには、次のオプションがあります。

* フィールドを null 許容型にします (たとえば、`decimal` の代わりに `decimal?` を使用)。 [Null \<T> ](/dotnet/csharp/programming-guide/nullable-types/)値を許容値型は、標準の null 許容型と同様に扱われます。
* 次の例に示すように、モデル バインドで使用される既定のエラー メッセージを指定します。

  [!code-csharp[](validation/samples/2.x/ValidationSample/Startup.cs?name=snippet_MaxModelValidationErrors&highlight=4-5)]

  既定のメッセージを設定できるモデル バインド エラーに関して詳しくは、<xref:Microsoft.AspNetCore.Mvc.ModelBinding.Metadata.DefaultModelBindingMessageProvider#methods> をご覧ください。

### <a name="required-validation-on-the-client"></a>クライアントでの [Required] の検証

クライアントでの null 非許容型と文字列の処理は、サーバーとは異なります。 クライアント側 :

* 値は、入力が行われた場合にのみ存在するものと見なされます。 そのため、クライアント側の検証では、null 非許容型は null 許容型と同じように処理されます。
* 文字列フィールド内の空白文字は、jQuery Validation の [required](https://jqueryvalidation.org/required-method/) メソッドでは有効な入力と見なされます。 サーバー側の検証では、空白文字のみが入力された場合は、必須文字列フィールドが無効であるものと見なされます。

前述のように、null 非許容型は `[Required]` 属性を持つものとして処理されます。 つまり、`[Required]` 属性を適用していない場合でも、クライアント側の検証が行われます。 ただし、属性を使用しない場合は、既定のエラー メッセージが表示されます。 カスタム エラー メッセージを指定するには、属性を使用します。

## <a name="remote-attribute"></a>[Remote] 属性

`[Remote]` 属性では、フィールド入力が有効かどうかを判断するためにサーバーでメソッドを呼び出す必要があるクライアント側検証が実装されます。 たとえば、アプリでは、ユーザー名が既に使用されているかどうかを確認することが必要な場合があります。

リモート検証を実装するには:

1. JavaScript で呼び出すアクション メソッドを作成します。  JQuery Validate の [remote](https://jqueryvalidation.org/remote-method/) メソッドでは、JSON の応答が必要です。

   * `"true"` は、入力データが有効であることを意味します。
   * `"false"`、`undefined`、または `null` は、入力が有効ではないことを意味します。  既定のエラー メッセージを表示します。
   * その他の文字列は、入力が無効であることを意味します。 カスタム エラー メッセージとして文字列を表示します。

   カスタム エラー メッセージを返すアクション メソッドの例を次に示します。

   [!code-csharp[](validation/samples/2.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyEmail)]

1. 次の例に示すように、モデル クラスで、検証アクション メソッドを指し示す `[Remote]` 属性を使用してプロパティに注釈を付けます。

   [!code-csharp[](validation/samples/2.x/ValidationSample/Models/User.cs?name=snippet_UserEmailProperty)]
 
   `[Remote]` 属性は `Microsoft.AspNetCore.Mvc` 名前空間にあります。 `Microsoft.AspNetCore.App` または `Microsoft.AspNetCore.All` メタパッケージを使用していない場合は、[Microsoft.AspNetCore.Mvc.ViewFeatures](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.ViewFeatures) NuGet パッケージをインストールします。
   
### <a name="additional-fields"></a>追加フィールド

`[Remote]` 属性の `AdditionalFields` プロパティでは、サーバー上のデータに対してフィールドの組み合わせを検証できます。 たとえば、`User` モデルに `FirstName` プロパティと `LastName` プロパティがある場合、その名前のペアを使用する既存ユーザーがいないことを確認したいことがあります。 `AdditionalFields` を使用する方法を次の例に示します。

[!code-csharp[](validation/samples/2.x/ValidationSample/Models/User.cs?name=snippet_UserNameProperties)]

`AdditionalFields` を文字列 `"FirstName"` および `"LastName"` に明示的に設定することもできますが、[nameof](/dotnet/csharp/language-reference/keywords/nameof) 演算子を使用すると、後のリファクタリングが容易になります。 この検証のアクション メソッドは、名と姓の両方を引数として受け取る必要があります。

[!code-csharp[](validation/samples/2.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyName)]

ユーザーが名または姓を入力すると、JavaScript はリモート呼び出しを行って、その名前のペアが取得されているかどうかを確認します。

複数の追加フィールドを検証するには、それらをコンマ区切りのリストとして提供します。 たとえば、`MiddleName` プロパティをモデルに追加するには、`[Remote]` 属性を次の例のように設定します。

```csharp
[Remote(action: "VerifyName", controller: "Users", AdditionalFields = nameof(FirstName) + "," + nameof(LastName))]
public string MiddleName { get; set; }
```

他の属性引数と同じように、`AdditionalFields` も定数式である必要があります。 したがって、[補間文字列](/dotnet/csharp/language-reference/keywords/interpolated-strings)を使用したり、<xref:System.String.Join*> を呼び出して `AdditionalFields` を初期化したりしないでください。

## <a name="alternatives-to-built-in-attributes"></a>組み込み属性に代わる方法

組み込み属性によって提供されない検証が必要な場合は、次のようにできます。

* [カスタム属性を作成する](#custom-attributes)。
* [IValidatableObject を実装する](#ivalidatableobject)。

## <a name="custom-attributes"></a>カスタム属性

組み込みの検証属性で処理されないシナリオの場合は、カスタム検証属性を作成できます。 <xref:System.ComponentModel.DataAnnotations.ValidationAttribute> を継承するクラスを作成し、<xref:System.ComponentModel.DataAnnotations.ValidationAttribute.IsValid*> メソッドをオーバーライドします。

`IsValid` メソッドは、*value* という名前のオブジェクトを受け取ります。これは、検証対象の入力です。 オーバーロードは `ValidationContext` オブジェクトも受け取ります。これは、モデル バインドによって作成されたモデル インスタンスなどの追加情報を提供します。

次の例では、*Classic* ジャンルの映画の公開日が指定した年より後ではないことを検証します。 `[ClassicMovie2]` 属性では最初にジャンルがチェックされ、*Classic* である場合にのみ続行されます。 クラシックとして識別された映画については、公開日が属性のコンストラクターに渡された制限より後ではないことがチェックされます。

[!code-csharp[](validation/samples/2.x/ValidationSample/Attributes/ClassicMovieAttribute.cs?name=snippet_ClassicMovieAttribute)]

前の例の `movie` 変数は、フォーム送信からのデータを格納している `Movie` オブジェクトを表します。 `IsValid` メソッドでは、日付とジャンルがチェックされます。 検証に成功すると、`IsValid` によって `ValidationResult.Success` コードが返されます。 検証に失敗すると、エラー メッセージを含む `ValidationResult` が返されます。

## <a name="ivalidatableobject"></a>IValidatableObject

前の例は、`Movie` 型でのみ動作します。 クラス レベルの検証に対するもう 1 つのオプションは、次の例に示すように、`IValidatableObject` をモデル クラスに実装することです。

[!code-csharp[](validation/samples/2.x/ValidationSample/Models/MovieIValidatable.cs?name=snippet&highlight=1,26-34)]

## <a name="top-level-node-validation"></a>最上位ノードの検証

最上位ノードには次が含まれています。

* アクションのパラメーター
* コントローラーのプロパティ
* ページ ハンドラーのパラメーター
* ページ モデルのプロパティ

モデルのパラメーター検証に加え、モデルが関連付けられた最上位ノードが検証されます。 サンプル アプリからの次の例の `VerifyPhone` メソッドでは、<xref:System.ComponentModel.DataAnnotations.RegularExpressionAttribute> を使用して `phone` アクションパラメーターが検証されています。

[!code-csharp[](validation/samples/2.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyPhone)]

最上位ノードでは、検証属性と共に <xref:Microsoft.AspNetCore.Mvc.ModelBinding.BindRequiredAttribute> を使用できます。 サンプル アプリからの次の例では、`CheckAge` メソッドによって、フォームの送信時、クエリ文字列から `age` パラメーターを関連付ける必要があることが指定されます。

[!code-csharp[](validation/samples/2.x/ValidationSample/Controllers/UsersController.cs?name=snippet_CheckAge)]

[年齢確認] ページ (*CheckAge.cshtml*) には 2 つのフォームがあります。 最初のフォームでは、`Age` の値 `99` がクエリ文字列 `https://localhost:5001/Users/CheckAge?Age=99` として送信されます。

クエリ文字列の正しく書式設定された `age` パラメーターが送信されると、フォームの有効性が確認されます。

[年齢確認] ページの 2 番目のフォームでは、要求本文で `Age` 値が送信され、検証は不合格となります。 `age` パラメーターはクエリ文字列で渡される必要があるため、バインドが失敗します。

`CompatibilityVersion.Version_2_1` 以降で実行すると、最上位ノードの検証が既定で有効になります。 それ以外の場合、最上位ノードの検証は無効です。 次に示すように、`Startup.ConfigureServices` で <xref:Microsoft.AspNetCore.Mvc.MvcOptions.AllowValidatingTopLevelNodes*> プロパティを設定することにより、既定のオプションをオーバーライドできます。

[!code-csharp[](validation/samples_snapshot/2.x/Startup.cs?name=snippet_AddMvc&highlight=4)]

## <a name="maximum-errors"></a>最大エラー数

エラーの最大数 (既定では 200) に達すると、検証は停止します。 この数は、`Startup.ConfigureServices` の次のコードを使用して構成します。

[!code-csharp[](validation/samples/2.x/ValidationSample/Startup.cs?name=snippet_MaxModelValidationErrors&highlight=3)]

## <a name="maximum-recursion"></a>最大再帰

<xref:Microsoft.AspNetCore.Mvc.ModelBinding.Validation.ValidationVisitor> では、検証対象のモデルのオブジェクト グラフが走査されます。 非常に深いモデルまたは無限に再帰するモデルでは、検証でスタック オーバーフローが発生する可能性があります。 [MvcOptions.MaxValidationDepth](xref:Microsoft.AspNetCore.Mvc.MvcOptions.MaxValidationDepth) では、ビジターの再帰が構成されている深さを超えた場合、早い段階で検証を停止する方法が提供されています。 `CompatibilityVersion.Version_2_2` 以降で実行したときの `MvcOptions.MaxValidationDepth` の既定値は 32 です。 それより前のバージョンでは、値は null であり、深さの制約がないことを意味します。

## <a name="automatic-short-circuit"></a>自動省略

モデル グラフの検証が必要ない場合、検証は自動的に省略 (スキップ) されます。 ランタイムで検証がスキップされるオブジェクトとしては、プリミティブのコレクション (`byte[]`、`string[]`、`Dictionary<string, string>` など) や、検証コントロールを何も持たない複雑なオブジェクト グラフなどがあります。

## <a name="disable-validation"></a>検証を無効にする

検証を無効にするには:

1. どのフィールドも無効としてマークしない `IObjectModelValidator` の実装を作成します。

   [!code-csharp[](validation/samples/2.x/ValidationSample/Attributes/NullObjectModelValidator.cs?name=snippet_DisableValidation)]

1. 依存関係挿入コンテナーで、次のコードを `Startup.ConfigureServices` に追加し、既定の `IObjectModelValidator` の実装を置き換えます。

   [!code-csharp[](validation/samples/2.x/ValidationSample/Startup.cs?name=snippet_DisableValidation)]

その場合でも、モデル バインドから発生するモデル状態エラーが表示される可能性があります。

## <a name="client-side-validation"></a>クライアント側の検証

クライアント側検証は、フォームが有効になるまで送信を許可しません。 送信ボタンをクリックすると、フォームの送信またはエラー メッセージの表示を行う JavaScript が実行されます。

クライアント側の検証を使用すると、フォームに入力エラーがある場合に、サーバーへの不要なラウンドトリップを回避できます。 *_Layout.cshtml* および *_ValidationScriptsPartial.cshtml* の次のスクリプト参照では、クライアント側の検証がサポートされています。

[!code-cshtml[](validation/samples/2.x/ValidationSample/Views/Shared/_Layout.cshtml?name=snippet_ScriptTag)]

[!code-cshtml[](validation/samples/2.x/ValidationSample/Views/Shared/_ValidationScriptsPartial.cshtml?name=snippet_ScriptTags)]

[jQuery Unobtrusive Validation](https://github.com/aspnet/jquery-validation-unobtrusive) スクリプトは、人気のある [jQuery Validate](https://jqueryvalidation.org/) プラグインを基に作成された Microsoft のカスタム フロントエンド ライブラリです。 jQuery Unobtrusive Validation を使用しないと、同じ検証ロジックを 2 か所でコーディングする必要があります。1 つはモデル プロパティでのサーバー側検証属性で、もう 1 つはクライアント側スクリプトです。 代わりに、[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)および [HTML ヘルパー](xref:mvc/views/overview)では、モデル プロパティの検証属性と型メタデータを使用して、検証の必要なフォーム要素に対する HTML 5 の `data-` 属性がレンダリングされます。 jQuery Unobtrusive Validation では、`data-` 属性が解析され、ロジックが jQuery Validate に渡されて、サーバー側検証ロジックがクライアントに実質的に "コピー" されます。 次に示すように、タグ ヘルパーを使用して、クライアントで検証エラーを表示できます。

[!code-cshtml[](validation/samples/2.x/ValidationSample/Views/Movies/Create.cshtml?name=snippet_ReleaseDate&highlight=4-5)]

上記のタグ ヘルパーでは、次の HTML がレンダリングされます。

```html
<form action="/Movies/Create" method="post">
    <div class="form-horizontal">
        <h4>Movie</h4>
        <div class="text-danger"></div>
        <div class="form-group">
            <label class="col-md-2 control-label" for="ReleaseDate">ReleaseDate</label>
            <div class="col-md-10">
                <input class="form-control" type="datetime"
                data-val="true" data-val-required="The ReleaseDate field is required."
                id="ReleaseDate" name="ReleaseDate" value="">
                <span class="text-danger field-validation-valid"
                data-valmsg-for="ReleaseDate" data-valmsg-replace="true"></span>
            </div>
        </div>
    </div>
</form>
```

HTML 出力の `data-` 属性が、`ReleaseDate` プロパティの検証属性に対応していることに注意してください。 `data-val-required` 属性には、ユーザーが公開日フィールドを入力していない場合に表示されるエラー メッセージが含まれています。 jQuery の控えめな検証は、この値を jQuery Validate [required ()](https://jqueryvalidation.org/required-method/) メソッドに渡します。このメソッドは、付随する要素にそのメッセージを表示し **\<span>** ます。

`[DataType]` 属性によってオーバーライドされていない限り、データ型の検証はプロパティの .NET 型に基づいて行われます。 ブラウザーには独自の既定のエラー メッセージがありますが、jQuery Validation Unobtrusive Validation パッケージでそれらのメッセージをオーバーライドできます。 `[DataType]` 属性と `[EmailAddress]` などのサブクラスを使用して、エラー メッセージを指定できます。

### <a name="add-validation-to-dynamic-forms"></a>動的なフォームに検証を追加する

ページが初めて読み込まれるときに、jQuery Unobtrusive Validation によって検証ロジックとパラメーターが jQuery Validate に渡されます。 したがって、動的に生成されるフォームでは、検証は自動的には機能しません。 検証を有効にするには、作成直後に動的フォームを解析するよう、jQuery Unobtrusive Validation に指示します。 たとえば、次のコードでは、AJAX によって追加されるフォームでクライアント側検証が設定されます。

```javascript
$.get({
    url: "https://url/that/returns/a/form",
    dataType: "html",
    error: function(jqXHR, textStatus, errorThrown) {
        alert(textStatus + ": Couldn't add form. " + errorThrown);
    },
    success: function(newFormHTML) {
        var container = document.getElementById("form-container");
        container.insertAdjacentHTML("beforeend", newFormHTML);
        var forms = container.getElementsByTagName("form");
        var newForm = forms[forms.length - 1];
        $.validator.unobtrusive.parse(newForm);
    }
})
```

`$.validator.unobtrusive.parse()` メソッドには、その引数の 1 つで jQuery セレクターを指定します。 このメソッドは、そのセレクター内のフォームの `data-` 属性を解析するよう jQuery Unobtrusive Validation に指示します。 その後、これらの属性の値は、jQuery Validate プラグインに渡されます。

### <a name="add-validation-to-dynamic-controls"></a>動的なコントロールに検証を追加する

`$.validator.unobtrusive.parse()` メソッドは、`<input>` や `<select/>` などの動的に生成される個々のコントロールではなく、フォーム全体に対して動作します。 フォームを再解析するには、次の例に示すように、フォームが前に解析されたときに追加された検証データを削除します。

```javascript
$.get({
    url: "https://url/that/returns/a/control",
    dataType: "html",
    error: function(jqXHR, textStatus, errorThrown) {
        alert(textStatus + ": Couldn't add control. " + errorThrown);
    },
    success: function(newInputHTML) {
        var form = document.getElementById("my-form");
        form.insertAdjacentHTML("beforeend", newInputHTML);
        $(form).removeData("validator")    // Added by jQuery Validate
               .removeData("unobtrusiveValidation");   // Added by jQuery Unobtrusive Validation
        $.validator.unobtrusive.parse(form);
    }
})
```

## <a name="custom-client-side-validation"></a>カスタム クライアント側検証

カスタム クライアント側検証は、カスタム jQuery Validate アダプターで動作する `data-` HTML 属性を生成することによって行われます。 次のサンプルのアダプター コードは、この記事で前に導入した `ClassicMovie` および `ClassicMovie2` 属性用に記述されたものです。

[!code-javascript[](validation/samples/2.x/ValidationSample/wwwroot/js/classicMovieValidator.js?name=snippet_UnobtrusiveValidation)]

アダプターの作成方法については、[jQuery Validate のドキュメント](https://jqueryvalidation.org/documentation/)をご覧ください。

特定のフィールドに対するアダプターの使用は、次のような `data-` 属性によってトリガーされます。

* 検証対象としてフィールドにフラグを設定します (`data-val="true"`)。
* 検証規則名とエラー メッセージ テキストを示します (例: `data-val-rulename="Error message."`)。
* 検証コントロールで必要なその他のパラメーターを提供します (例: `data-val-rulename-parm1="value"`)。

次の例では、サンプル アプリの `ClassicMovie` 属性に対する `data-` 属性を示します。

```html
<input class="form-control" type="datetime"
    data-val="true"
    data-val-classicmovie1="Classic movies must have a release year earlier than 1960."
    data-val-classicmovie1-year="1960"
    data-val-required="The ReleaseDate field is required."
    id="ReleaseDate" name="ReleaseDate" value="">
```

前に説明したように、[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)と [HTML ヘルパー](xref:mvc/views/overview)では、検証属性からの情報を使用して `data-` 属性がレンダリングされます。 カスタム `data-` HTML 属性が作成されるようになるコードを記述するには、2 つのオプションがあります。

* `AttributeAdapterBase<TAttribute>` から派生するクラスと `IValidationAttributeAdapterProvider` を実装するクラスを作成し、属性とそのアダプターを DI に登録します。 この方法では[単一責任の原則](https://wikipedia.org/wiki/Single_responsibility_principle)に従って、サーバー関連の検証コードとクライアント関連の検証コードは別のクラスになります。 アダプターには、DI に登録されるため、必要な場合に DI 内の他のサービスがそれを使用できるという利点もあります。
* `ValidationAttribute` クラスで `IClientModelValidator` を実装します。 この方法は、属性でサーバー側の検証が何も行われず、DI からのサービスが必要ない場合に、適している可能性があります。

### <a name="attributeadapter-for-client-side-validation"></a>クライアント側検証用の AttributeAdapter

HTML に `data-` 属性をレンダリングするこの方法は、サンプル アプリの `ClassicMovie` 属性で使用されています。 この方法を使用してクライアント検証を追加するには、次のようにします。

1. カスタム検証属性の属性アダプター クラスを作成します。 <xref:Microsoft.AspNetCore.Mvc.DataAnnotations.AttributeAdapterBase%601>からクラスを派生させます。 次の例で示すように、レンダリングされた出力に `data-` 属性を追加する `AddValidation` メソッドを作成します。

   [!code-csharp[](validation/samples/2.x/ValidationSample/Attributes/ClassicMovieAttributeAdapter.cs?name=snippet_ClassicMovieAttributeAdapter)]

1. <xref:Microsoft.AspNetCore.Mvc.DataAnnotations.IValidationAttributeAdapterProvider> を実装するアダプター プロバイダー クラスを作成します。 次の例に示すように、`GetAttributeAdapter` メソッドで、カスタム属性をアダプターのコンストラクターに渡します。

   [!code-csharp[](validation/samples/2.x/ValidationSample/Attributes/CustomValidationAttributeAdapterProvider.cs?name=snippet_CustomValidationAttributeAdapterProvider)]

1. `Startup.ConfigureServices` でアダプター プロバイダーを DI に登録します。

   [!code-csharp[](validation/samples/2.x/ValidationSample/Startup.cs?name=snippet_MaxModelValidationErrors&highlight=8-10)]

### <a name="iclientmodelvalidator-for-client-side-validation"></a>クライアント側検証用の IClientModelValidator

HTML に `data-` 属性をレンダリングするこの方法は、サンプル アプリの `ClassicMovie2` 属性で使用されています。 この方法を使用してクライアント検証を追加するには、次のようにします。

* カスタム検証属性で、`IClientModelValidator` インターフェイスを実装し、`AddValidation` メソッドを作成します。 次の例で示すように、`AddValidation` メソッドで、検証用の `data-` 属性を追加します。

  [!code-csharp[](validation/samples/2.x/ValidationSample/Attributes/ClassicMovie2Attribute.cs?name=snippet_ClassicMovie2Attribute)]

## <a name="disable-client-side-validation"></a>クライアント側検証を無効にする

次のコードでは、MVC ビューのクライアント検証が無効になります。

[!code-csharp[](validation/samples_snapshot/2.x/Startup2.cs?name=snippet_DisableClientValidation)]

ページ内 Razor :

[!code-csharp[](validation/samples_snapshot/2.x/Startup3.cs?name=snippet_DisableClientValidation)]

クライアント検証を無効にするもう 1 つのオプションは、*.cshtml* ファイルで `_ValidationScriptsPartial` への参照をコメントにすることです。

## <a name="additional-resources"></a>その他のリソース

* [System.ComponentModel.DataAnnotations 名前空間](xref:System.ComponentModel.DataAnnotations)
* [モデルバインド](model-binding.md)

::: moniker-end
