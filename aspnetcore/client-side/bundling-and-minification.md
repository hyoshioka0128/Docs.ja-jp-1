---
title: ASP.NET Core での静的資産のバンドルと縮小
author: scottaddie
description: バンドルと縮小の手法を適用して、ASP.NET Core Web アプリケーションの静的リソースを最適化する方法について説明します。
ms.author: scaddie
ms.custom: mvc
ms.date: 03/14/2021
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
uid: client-side/bundling-and-minification
ms.openlocfilehash: d594bbf277907e22b0299b0451e480e9d533d506
ms.sourcegitcommit: 00368bb6a5420983beaced5b62dabc1f94abdeba
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/16/2021
ms.locfileid: "103557804"
---
# <a name="bundle-and-minify-static-assets-in-aspnet-core"></a>ASP.NET Core での静的資産のバンドルと縮小

作成者: [Scott Addie](https://twitter.com/Scott_Addie)、[David Pine](https://twitter.com/davidpine7)

この記事では、ASP.NET Core Web アプリでこれらの機能を使用する方法など、バンドルと縮小を適用する利点について説明します。

## <a name="what-is-bundling-and-minification"></a>バンドルと縮小とは

バンドルと縮小は、Web アプリに適用できる 2 種類のパフォーマンス最適化です。 バンドルと縮小を併用すると、サーバー要求の数が減り、要求される静的資産のサイズが小さくなるため、パフォーマンスが向上します。

バンドルと縮小を使用すると、主に最初のページ要求の読み込み時間が短縮されます。 Web ページが要求されると、ブラウザーによって静的資産 (JavaScript、CSS、および画像) がキャッシュされます。 そのため、同じ資産を要求する同じサイト上で、同じページまたは複数のページを要求する場合、バンドルと縮小を使用してもパフォーマンスは改善されません。 expires ヘッダーが資産に正しく設定されておらず、バンドルと縮小が使用されていない場合、ブラウザーの更新の間隔ヒューリスティックにより、数日後には資産が古くなっているとマークされます。 さらに、ブラウザーでは、各資産に対する検証要求が必要です。 この場合、最初のページ要求の後でも、バンドルと縮小によりパフォーマンスが向上します。

### <a name="bundling"></a>バンドル

バンドルでは、複数のファイルを単一のファイルに連結します。 バンドルを使用すると、Web ページなどの Web 資産のレンダリングに必要なサーバー要求の数を減らすことができます。 CSS、JavaScript など専用の個別のバンドルをいくつでも作成できます。ファイルが少なくなると、ブラウザーからサーバーへ、またはアプリケーションを提供するサービスからの HTTP 要求が少なくなることを意味します。 その結果、最初のページ読み込みのパフォーマンスが向上します。

### <a name="minification"></a>縮小

縮小を使用すると、機能を変更せずにコードから不要な文字を削除できます。 その結果、要求される資産 (CSS、画像、JavaScript ファイルなど) のサイズが大幅に減ります。 縮小の一般的な副作用として、変数名を 1 文字に短縮する機能や、コメントと不要な空白を削除する機能があります。

次の JavaScript 関数を考えてみます。

```javascript
AddAltToImg = function (imageTagAndImageID, imageContext) {
    ///<signature>
    ///<summary> Adds an alt tab to the image
    // </summary>
    //<param name="imgElement" type="String">The image selector.</param>
    //<param name="ContextForImage" type="String">The image context.</param>
    ///</signature>
    var imageElement = $(imageTagAndImageID, imageContext);
    imageElement.attr('alt', imageElement.attr('id').replace(/ID/, ''));
}
```

縮小により、関数が次のように短縮されます。

```javascript
AddAltToImg=function(t,a){var r=$(t,a);r.attr("alt",r.attr("id").replace(/ID/,""))};
```

コメントと不要な空白の削除に加えて、次のパラメーターと変数名は次のように名前が変更されました。

元 | 名前の変更
--- | :---:
`imageTagAndImageID` | `t`
`imageContext` | `a`
`imageElement` | `r`

## <a name="impact-of-bundling-and-minification"></a>バンドルと縮小の影響

次の表は、個別に資産を読み込む場合とバンドルと縮小を使用する場合の違いを示しています。

アクション | バンドルと縮小あり | バンドルと縮小なし | 変更
--- | :---: | :---: | :---:
ファイル要求  | 7   | 18     | 157%
転送済み (KB) | 156 | 264.68 | 70%
読み込み時間 (ミリ秒) | 885 | 2360   | 167%

ブラウザーは、HTTP 要求ヘッダーに関してかなり冗長です。 送信された合計バイト数メトリックは、バンドル時に大幅に減りました。 読み込み時間は大幅に改善されていますが、この例はローカルで実行されています。 ネットワークを経由で転送される資産にバンドルと縮小を使用すると、パフォーマンスが大幅に向上します。

## <a name="choose-a-bundling-and-minification-strategy"></a>バンドルと縮小の戦略を選択する

ASP.NET Core は、オープンソースのバンドルと縮小ソリューションである WebOptimizer と互換性があります。 セットアップ手順とサンプル プロジェクトについては、[WebOptimizer](https://github.com/ligershark/WebOptimizer) を参照してください。 ASP.NET Core には、ネイティブのバンドルと縮小ソリューションはありません。

[Gulp](https://gulpjs.com)、[Webpack](https://webpack.js.org) などのサードパーティ製ツールを利用すると、バンドルと縮小のワークフローを自動化したり、リンティングとイメージの最適化を行ったりすることができます。 設計時にバンドルと縮小を使用することで、アプリのデプロイ前に縮小されたファイルが作成されます。 デプロイ前のバンドルと縮小によって、サーバーの負荷が軽減されます。 ただし、設計時にバンドルと縮小を使用するとビルドの複雑さが増すので、静的ファイルでのみ機能することを認識することが重要です。

## <a name="environment-based-bundling-and-minification"></a>環境ベースのバンドルと縮小

ベスト プラクティスとして、アプリのバンドルおよび縮小されたファイルを運用環境で使用することをお勧めします。 開発中は、元のファイルがあるので、アプリのデバッグが容易になります。

ビューで [Environment Tag Helper](xref:mvc/views/tag-helpers/builtin-th/environment-tag-helper) を使用して、ページに含めるファイルを指定します。 Environment Tag Helper を使用すると、特定の[環境](xref:fundamentals/environments)で実行されている場合にのみコンテンツがレンダリングされます。

`Development` 環境で実行されている場合、次の `environment` タグを使用すると、未処理の CSS ファイルがレンダリングされます。

::: moniker range=">= aspnetcore-2.0"

```cshtml
<environment include="Development">
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
    <link rel="stylesheet" href="~/css/site.css" />
</environment>
```

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

```cshtml
<environment names="Staging,Production">
    <link rel="stylesheet" href="https://ajax.aspnetcdn.com/ajax/bootstrap/3.3.7/css/bootstrap.min.css"
          asp-fallback-href="~/lib/bootstrap/dist/css/bootstrap.min.css"
          asp-fallback-test-class="sr-only" asp-fallback-test-property="position" asp-fallback-test-value="absolute" />
    <link rel="stylesheet" href="~/css/site.min.css" asp-append-version="true" />
</environment>
```

::: moniker-end

`Development` 以外の環境で実行されている場合に、次の `environment` タグを使用すると、バンドルおよび縮小された CSS ファイルがレンダリングされます。 たとえば、`Production` または `Staging` で実行すると、これらのスタイルシートのレンダリングがトリガーされます。

::: moniker range=">= aspnetcore-2.0"

```cshtml
<environment exclude="Development">
    <link rel="stylesheet" href="https://ajax.aspnetcdn.com/ajax/bootstrap/3.3.7/css/bootstrap.min.css"
          asp-fallback-href="~/lib/bootstrap/dist/css/bootstrap.min.css"
          asp-fallback-test-class="sr-only" asp-fallback-test-property="position" asp-fallback-test-value="absolute" />
    <link rel="stylesheet" href="~/css/site.min.css" asp-append-version="true" />
</environment>
```

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

```cshtml
<environment names="Staging,Production">
    <link rel="stylesheet" href="https://ajax.aspnetcdn.com/ajax/bootstrap/3.3.7/css/bootstrap.min.css"
          asp-fallback-href="~/lib/bootstrap/dist/css/bootstrap.min.css"
          asp-fallback-test-class="sr-only" asp-fallback-test-property="position" asp-fallback-test-value="absolute" />
    <link rel="stylesheet" href="~/css/site.min.css" asp-append-version="true" />
</environment>
```

::: moniker-end

## <a name="additional-resources"></a>その他のリソース

* [複数の環境の使用](xref:fundamentals/environments)
* [タグ ヘルパー](xref:mvc/views/tag-helpers/intro)
