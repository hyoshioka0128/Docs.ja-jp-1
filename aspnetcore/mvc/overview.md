---
title: ASP.NET Core MVC の概要
author: ardalis
description: ASP.NET Core MVC が、モデル ビュー コントローラー デザイン パターンを使用して、Web アプリと API をビルドするための豊富なフレームワークであることについて説明します。
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
uid: mvc/overview
ms.openlocfilehash: 642a7b26a3ba0a79a35437a5c19ee0be63a367af
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057318"
---
# <a name="overview-of-aspnet-core-mvc"></a>ASP.NET Core MVC の概要

作成者: [Steve Smith](https://ardalis.com/)

ASP.NET Core MVC は、モデル ビュー コントローラー デザイン パターンを使用して、Web アプリと API をビルドするための豊富なフレームワークです。

## <a name="what-is-the-mvc-pattern"></a>MVC パターンとは何ですか?

モデル ビュー コントローラー (MVC) アーキテクチャ パターンでは、アプリケーションをモデル、ビュー、コントローラーという 3 つの主要なコンポーネントのグループに分けます。 このパターンは、[関心の分離](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns)を実現するのに役立ちます。 このパターンを使用すると、ユーザーの要求が、ユーザーの操作を実行したり、クエリの結果を取得したりするためにモデルを操作する役割がある、コントローラーにルーティングされます。 コントローラーでは、ユーザーに表示するビューを選び、必要なモデル データと共に提供します。

次の図は、3 つの主要なコンポーネントと、どのコンポーネントがどのコンポーネントを参照するかを示しています。

![MVC パターン](overview/_static/mvc.png)

この責任の概念は、複雑さの観点からアプリケーションを計るために役立ちます。それは、単一のジョブを持つもの (モデル、ビュー、コントローラー) の方がコーディング、デバッグ、およびテストしやすいためです。 これらの 3 つの領域の 2 つ以上に依存関係が広がるコードを更新、テスト、デバッグする方が難しいです。 たとえば、ユーザー インターフェイス ロジックは、ビジネス ロジックよりも頻繁に変更される傾向があります。 プレゼンテーション コードとビジネス ロジックが 1 つのオブジェクトに結合されている場合は、ユーザー インターフェイスを変更するたびに、ビジネス ロジックを含むオブジェクトは変更されます。 これにより、エラーが発生することが多く、最小限のユーザー インターフェイスの変更後に毎回ビジネス ロジックのテストを再実行する必要があります。

> [!NOTE]
> ビューとコントローラーは両方ともモデルに依存します。 ただし、モデルはビューにもコントローラーにも依存しません。 これは、分離の主な利点の 1 つです。 この分離によって、モデルをビルドし、視覚表示の独立をテストすることができます。

### <a name="model-responsibilities"></a>モデルの責任

MVC アプリケーションのモデルは、アプリケーションの状態、およびアプリケーションによって実行されるビジネス ロジックや操作を表します。 ビジネス ロジックは、アプリケーションの状態を保持するための実装ロジックと共に、モデルにカプセル化される必要があります。 通常、厳密に型指定されたビューでは、そのビューに表示するデータを含めるようにデザインされた ViewModel 型を使用します。 コントローラーは、モデルからこれらの ViewModel インスタンスを作成します。

### <a name="view-responsibilities"></a>ビューの責任

ビューは、ユーザー インターフェイスを介してコンテンツを表示する役割があります。 これらは、 [ Razor ビューエンジン](#razor-view-engine)を使用して、.NET コードを HTML マークアップに埋め込みます。 ビュー内のロジックは最小限にする必要があり、そこに含まれるロジックはすべて、コンテンツの表示に関連する必要があります。 複雑なモデルからデータを表示するために、ビュー ファイルで多くのロジックを実行する必要がある場合、ビューを簡略化するために、[ビューのコンポーネント](views/view-components.md)、ViewModel、テンプレートの表示を使用することを検討してください。

### <a name="controller-responsibilities"></a>コントローラーの責任

コントローラーは、ユーザーの操作を処理し、モデルを操作し、最終的にレンダリングするビューを選択するコンポーネントです。 MVC アプリケーションでは、ビューは情報のみを表示し、コントローラーがユーザーの入力と操作を処理して応答します。 MVC パターンでは、コントローラーは最初のエントリ ポイントであり、処理するモデルの型およびレンダリングするビューを選択する役割があります (そのため、このような名前で呼ばれており、アプリが指定した要求に応答する方法を制御します)。

> [!NOTE]
> コントローラーは責任が非常に多いので、過度に複雑にしないでください。 コントローラー ロジックが過度に複雑にならないように、ビジネス ロジックをコントローラーから排除し、ドメイン モデルに含めます。

>[!TIP]
> コントローラーのアクションで、頻繁に同じ種類のアクションを実行することがわかった場合、これらの一般的なアクションを[フィルター](#filters)に移動させます。

## <a name="what-is-aspnet-core-mvc"></a>ASP.NET Core MVC とは何ですか?

ASP.NET Core MVC フレームワークは、ASP.NET Core と共に使用するために最適化された、ライトウェイトかつオープン ソースのテストしやすいプレゼンテーション フレームワークです。

ASP.NET Core MVC では、明確な関心の分離を可能にする動的な Web サイトをビルドするためのパターン ベースの方法を提供します。 ここでは、マークアップのフル コントロールが提供され、TDD 向けの開発をサポートし、最新の Web 標準を使用することができます。

## <a name="features"></a>特徴

ASP.NET Core MVC には、以下が含まれます。

* [ルーティング](#routing)
* [モデル バインド](#model-binding)
* [モデルの検証](#model-validation)
* [依存関係の挿入](../fundamentals/dependency-injection.md)
* [フィルター](#filters)
* [領域](#areas)
* [Web API](#web-apis)
* [Testability](#testability)
* [Razor ビューエンジン](#razor-view-engine)
* [厳密に型指定されたビュー](#strongly-typed-views)
* [タグ ヘルパー](#tag-helpers)
* [コンポーネントの表示](#view-components)

### <a name="routing"></a>ルーティング

ASP.NET Core MVC は、[ASP.NET Core のルーティング](../fundamentals/routing.md)の上にビルドされます。このルーティングは強力な URL マッピング コンポーネントで、わかりやすく検索可能な URL のアプリケーションをビルドできます。 これにより、Web サーバー上のファイルを整理する方法に関係なく、検索エンジンの最適化 (SEO) およびリンクの作成に適しているアプリケーションの URL の名前付けパターンを定義できます。 ルート値制約、既定値、省略可能な値をサポートする、便利なルート テンプレートの構文を使用して、ルートを定義できます。

*規約ベースのルーティング* を使用すると、アプリケーションで受け入れられる URL 形式と、各形式を特定のコントローラー上の特定のアクションメソッドにマップする方法をグローバルに定義できます。 受信要求を受け取ると、ルーティング エンジンは URL を解析し、定義された URL 形式のいずれかに合わせて、関連するコントローラーのアクション メソッドを呼び出します。

```csharp
routes.MapRoute(name: "Default", template: "{controller=Home}/{action=Index}/{id?}");
```

*属性のルーティング* では、アプリケーションのルートを定義する属性でコントローラーとアクションを装飾して、ルーティング情報を指定できます。 つまり、ルート定義は、関連付けられているコントローラーとアクションの横に配置されるということです。

```csharp
[Route("api/[controller]")]
public class ProductsController : Controller
{
    [HttpGet("{id}")]
    public IActionResult GetProduct(int id)
    {
      ...
    }
}
```

### <a name="model-binding"></a>モデル バインド

ASP.NET Core MVC の[モデル バインド](models/model-binding.md)は、クライアント要求データ (フォームの値、ルート データ、クエリ文字列のパラメーター、HTTP ヘッダー) をコントローラーが処理できるオブジェクトに変換します。 その結果、コントローラー ロジックは、受信要求データを判断する作業を行う必要はありません。そのアクション メソッドに対するパラメーターとしてデータを持つだけです。

```csharp
public async Task<IActionResult> Login(LoginViewModel model, string returnUrl = null) { ... }
```

### <a name="model-validation"></a>モデルの検証

ASP.NET Core MVC では、データ注釈の検証属性でモデル オブジェクトを装飾することで、[検証](models/validation.md)をサポートします。 検証属性は、値をサーバーにポストする前に、クライアント側でチェックされ、コントローラー アクションが呼び出される前に、サーバー側でチェックされます。

```csharp
using System.ComponentModel.DataAnnotations;
public class LoginViewModel
{
    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Required]
    [DataType(DataType.Password)]
    public string Password { get; set; }

    [Display(Name = "Remember me?")]
    public bool RememberMe { get; set; }
}
```

コントローラー アクション:

```csharp
public async Task<IActionResult> Login(LoginViewModel model, string returnUrl = null)
{
    if (ModelState.IsValid)
    {
      // work with the model
    }
    // At this point, something failed, redisplay form
    return View(model);
}
```

フレームワークでは、クライアントとサーバーの両方で要求データの検証を処理します。 モデルの型に指定された検証ロジックは、控えめな注釈としてレンダリングされたビューに追加され、[jQuery Validation](https://jqueryvalidation.org/) を使ってブラウザーに適用されます。

### <a name="dependency-injection"></a>依存関係の挿入

ASP.NET Core には、[依存関係の挿入 (DI)](../fundamentals/dependency-injection.md) の組み込みのサポートがあります。 ASP.NET Core MVC では、[コントローラー](controllers/dependency-injection.md)は、[明示的な依存関係の原則](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies)に従うことを許可して、コンストラクターを介して必要なサービスを要求できます。

また、自分のアプリで、`@inject` ディレクティブを使用して、[ビュー ファイルに依存関係の挿入](views/dependency-injection.md)を使用することもできます。

```cshtml
@inject SomeService ServiceName

<!DOCTYPE html>
<html lang="en">
<head>
    <title>@ServiceName.GetTitle</title>
</head>
<body>
    <h1>@ServiceName.GetTitle</h1>
</body>
</html>
```

### <a name="filters"></a>フィルタ

[フィルター](controllers/filters.md)は、開発者が例外処理や承認など、横断的関心事をカプセル化するのに役立ちます。 フィルターでは、アクション メソッドの前処理と後処理ロジックを実行できるようにします。また、指定した要求の実行パイプライン内のある時点で実行するように構成することもできます。 フィルターは、属性としてコントローラーまたはアクションに適用できます (または、グローバルに実行できます)。 いくつかのフィルター (`Authorize` など) は、フレームワークに含まれます。 `[Authorize]` は MVC 承認フィルターの作成のために使用される属性です.

```csharp
[Authorize]
public class AccountController : Controller
```

### <a name="areas"></a>Areas

[区分](controllers/areas.md) を使用すると、大規模な ASP.NET Core MVC Web アプリをより小さな機能グループに分割することができます。 区分は、アプリケーション内の MVC 構造体となります。 MVC プロジェクトでは、モデル、コント ローラー、ビューなどの論理コンポーネントが異なるフォルダーに保持され、MVC では名前付け規則を使用して、これらのコンポーネントの関係を作成します。 大きなアプリでは、アプリを機能の個別の高レベル区分に分割すると便利な場合があります。 たとえば、チェックアウト、請求、検索などの複数のビジネスユニットを含む e コマースアプリです。これらの各ユニットには、それぞれ独自の論理コンポーネントビュー、コントローラー、およびモデルがあります。

### <a name="web-apis"></a>Web API

Web サイトのビルドに最適なプラットフォームというだけでなく、ASP.NET Core MVC には Web API のビルドに適したサポートがあります。 ブラウザーやモバイル デバイスなど、さまざまなクライアントに提供されるサービスをビルドできます。

フレームワークには、JSON または XML として [書式のデータ](xref:web-api/advanced/formatting)の組み込みサポートと共に、HTTP コンテンツ ネゴシエーションのサポートが含まれます。 [カスタム フォーマッタ](xref:web-api/advanced/custom-formatters)を作成して、独自の書式のサポートを追加します。

リンクの生成を使って、ハイパーメディアのサポートを有効にします。 自分の Web API を複数の Web アプリケーションで共有できるように、[クロス オリジン リソース共有 (CORS)](https://www.w3.org/TR/cors/) のサポートを簡単に有効にできます。

### <a name="testability"></a>Testability

フレームワークでインターフェイスと依存関係の挿入を使用して、単体テストに適するようにします。また、フレームワークには、[統合テスト](xref:test/integration-tests)もすばやく簡単にする機能 (Entity Framework の TestHost と InMemory プロバイダーなど) が含まれます。 詳細については、[コントローラー ロジックのテスト方法](controllers/testing.md)に関するページを参照してください。

### <a name="no-locrazor-view-engine"></a>Razor ビューエンジン

[MVC ビュー ASP.NET Core](views/overview.md)ビューを表示するために[ Razor ビューエンジン](views/razor.md)を使用します。 Razor は、埋め込み C# コードを使用してビューを定義するための、コンパクトで表現力豊かな滑らかなテンプレートマークアップ言語です。 Razor は、サーバー上の web コンテンツを動的に生成するために使用されます。 サーバー コードを、クライアント側のコンテンツとコードにクリーンに混在させることができます。

```cshtml
<ul>
    @for (int i = 0; i < 5; i++) {
        <li>List item @i</li>
    }
</ul>
```

Razorビューエンジンを使用すると、[レイアウト](views/layout.md)、[部分ビュー](views/partial.md) 、置き換え可能なセクションを定義できます。

### <a name="strongly-typed-views"></a>厳密に型指定されたビュー

Razor MVC のビューは、モデルに基づいて厳密に型指定できます。 コントローラーは、ビューの型チェックと IntelliSense サポートを有効にして、厳密に型指定されたモデルをビューに渡すことができます。

たとえば、次のビューでは、モデルの型 `IEnumerable<Product>` をレンダリングします。

```cshtml
@model IEnumerable<Product>
<ul>
    @foreach (Product p in Model)
    {
        <li>@p.Name</li>
    }
</ul>
```

### <a name="tag-helpers"></a>タグ ヘルパー

[タグヘルパー](views/tag-helpers/intro.md) を使用すると、サーバー側コードがファイル内の HTML 要素の作成とレンダリングに参加できるようになり Razor ます。 タグ ヘルパーを使って、カスタム タグ (例: `<environment>`) を定義したり、既存のタグ (例: `<label>`) の動作を変更したりすることができます。 タグ ヘルパーは、要素名とその属性に基づいて特定の要素をバインドします。 タグ ヘルパーでは、HTML の編集操作を保持しながら、サーバー側のレンダリングの利点を提供します。

フォームやリンクの作成、資産の読み込みなど、一般的なタスクの組み込みのタグ ヘルパーは数多くあります。パブリック GitHub リポジトリで NuGet パッケージとして使用することもできます。 タグ ヘルパーは C# で作成され、要素名、属性名、または親タグに基づく HTML 要素をターゲットとします。 たとえば、組み込みの LinkTagHelper を使用して、`AccountsController` の `Login` へのリンクを作成することができます。

```cshtml
<p>
    Thank you for confirming your email.
    Please <a asp-controller="Account" asp-action="Login">Click here to Log in</a>.
</p>
```

`EnvironmentTagHelper` を使用して、開発、ステージング、運用など、ランタイム環境に基づいて、ビュー (未加工、縮小など) にさまざまなスクリプトを含めることができます。

```cshtml
<environment names="Development">
    <script src="~/lib/jquery/dist/jquery.js"></script>
</environment>
<environment names="Staging,Production">
    <script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-2.1.4.min.js"
            asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
            asp-fallback-test="window.jQuery">
    </script>
</environment>
```

タグヘルパーは、HTML とマークアップを作成するための、HTML に適した開発エクスペリエンスと豊富な IntelliSense 環境を提供し Razor ます。 組み込みのタグ ヘルパーのほとんどは既存の HTML 要素をターゲットとし、要素に対してサーバー側の属性を提供します。

### <a name="view-components"></a>ビュー コンポーネント

[ビュー コンポーネント](views/view-components.md)を使用すると、レンダリング ロジックをパッケージ化し、アプリケーション全体で再利用することができます。 これは[部分ビュー](views/partial.md)と似ていますが、関連するロジックを含みます。

## <a name="compatibility-version"></a>互換バージョン

<xref:Microsoft.Extensions.DependencyInjection.MvcCoreMvcBuilderExtensions.SetCompatibilityVersion*> メソッドを使用すると、ASP.NET Core MVC 2.1 以降に導入されている、互換性に影響する重大な変更をオプトインまたはオプトアウトすることができます。

詳細については、「<xref:mvc/compatibility-version>」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* [ASP.NET CORE mvc 用の AspNetCore-Fluent テストライブラリ](https://github.com/ivaylokenov/MyTested.AspNetCore.Mvc): 厳密に型指定された単体テストライブラリ。 mvc および web API アプリをテストするための fluent インターフェイスを提供します。 ("*Microsoft では保守管理もサポートも行っていません。* ")
* <xref:blazor/components/prerendering-and-integration>
