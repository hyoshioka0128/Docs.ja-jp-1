---
title: ASP.NET Core Web API のカスタム フォーマッタ
author: rick-anderson
description: ASP.NET Core で Web API のカスタム フォーマッタを作成して使用する方法を説明します。
ms.author: riande
ms.date: 06/25/2020
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
uid: web-api/advanced/custom-formatters
ms.openlocfilehash: 91c9c6513d7c8df671e283508ecc276768d79539
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587828"
---
# <a name="custom-formatters-in-aspnet-core-web-api"></a>ASP.NET Core Web API のカスタム フォーマッタ

[Kirk Larkin](https://twitter.com/serpent5)と[Tom Dykstra](https://github.com/tdykstra)。

ASP.NET Core MVC は、入力と出力のフォーマッタを使用した Web API でのデータ交換をサポートしています。 入力フォーマッタは、[モデル バインド](xref:mvc/models/model-binding)によって使用されます。 出力フォーマッタは、 [応答を書式設定](xref:web-api/advanced/formatting)するために使用されます。

フレームワークには、JSON および XML 用の組み込みの入力および出力フォーマッタが用意されています。 プレーン テキスト用の組み込みの出力フォーマッタが用意されていますが、プレーン テキスト用の入力フォーマッタは用意されていません。

この記事では、カスタム フォーマッタを作成して、追加形式のサポートを追加する方法を示します。 カスタムプレーンテキスト入力フォーマッタの例については、GitHub の [TextPlainInputFormatter](https://github.com/aspnet/Entropy/blob/master/samples/Mvc.Formatters/TextPlainInputFormatter.cs) を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="when-to-use-custom-formatters"></a>カスタム フォーマッタを使用するタイミング

カスタムフォーマッタを使用して、組み込みのフォーマッタによって処理されないコンテンツの種類のサポートを追加します。

## <a name="overview-of-how-to-use-a-custom-formatter"></a>カスタム フォーマッタの使用方法の概要

カスタムフォーマッタを作成するには:

* クライアントに送信されるデータをシリアル化するには、出力フォーマッタクラスを作成します。
* クライアントから受信したデータを逆シリアル化するには、入力フォーマッタクラスを作成します。
* のコレクションおよびのコレクションに、フォーマッタクラスのインスタンスを追加 `InputFormatters` `OutputFormatters` <xref:Microsoft.AspNetCore.Mvc.MvcOptions> します。

## <a name="how-to-create-a-custom-formatter-class"></a>カスタム フォーマッタ クラスの作成方法

フォーマッタを作成する場合は、次のようにします。

* クラスを適切な基底クラスから派生させます。 このサンプルアプリは、およびから派生して <xref:Microsoft.AspNetCore.Mvc.Formatters.TextOutputFormatter> <xref:Microsoft.AspNetCore.Mvc.Formatters.TextInputFormatter> います。
* コンストラクターで有効なメディアの種類とエンコーディングを指定します。
* <xref:Microsoft.AspNetCore.Mvc.Formatters.InputFormatter.CanReadType%2A> および <xref:Microsoft.AspNetCore.Mvc.Formatters.OutputFormatter.CanWriteType%2A> メソッドをオーバーライドします。
* <xref:Microsoft.AspNetCore.Mvc.Formatters.InputFormatter.ReadRequestBodyAsync%2A> および `WriteResponseBodyAsync` メソッドをオーバーライドします。

次のコードは、サンプルのクラスを示してい `VcardOutputFormatter` ます。 [](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples)

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_Class)]
  
### <a name="derive-from-the-appropriate-base-class"></a>適切な基底クラスからの派生

テキストメディアの種類 (たとえば、vCard) の場合 <xref:Microsoft.AspNetCore.Mvc.Formatters.TextInputFormatter> は、またはの基底クラスから派生し <xref:Microsoft.AspNetCore.Mvc.Formatters.TextOutputFormatter> ます。

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_ClassDeclaration)]

バイナリ型の場合 <xref:Microsoft.AspNetCore.Mvc.Formatters.InputFormatter> は、またはの基底クラスから派生し <xref:Microsoft.AspNetCore.Mvc.Formatters.OutputFormatter> ます。

### <a name="specify-valid-media-types-and-encodings"></a>有効なメディアの種類とエンコーディングの指定

コンストラクターで、`SupportedMediaTypes` および `SupportedEncodings` コレクションに追加して、有効なメディアの種類とエンコーディングを指定します。

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_ctor)]

フォーマッタクラスは、依存関係にコンストラクターの挿入を使用でき **ません** 。 たとえば、を `ILogger<VcardOutputFormatter>` パラメーターとしてコンストラクターに追加することはできません。 サービスにアクセスするには、メソッドに渡されるコンテキストオブジェクトを使用します。 この記事のコード例と [サンプル](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples) では、この方法を示しています。

### <a name="override-canreadtype-and-canwritetype"></a>CanReadType と Canreadtype のオーバーライド

メソッドまたはメソッドをオーバーライドして、逆シリアル化またはシリアル化を行う型を指定し `CanReadType` `CanWriteType` ます。 たとえば、型から vCard テキストを作成 `Contact` し、その逆も同様にします。

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_CanWriteType)]

#### <a name="the-canwriteresult-method"></a>CanWriteResult メソッド

一部のシナリオでは、をで `CanWriteResult` はなくオーバーライドする必要があり `CanWriteType` ます。 次のすべての条件が満たされている場合は、`CanWriteResult` を使用します。

* アクションメソッドは、モデルクラスを返します。
* 実行時に返される可能性がある派生クラスがあります。
* アクションによって返される派生クラスは、実行時に既知である必要があります。

たとえば、次のようなアクションメソッドがあるとします。

* シグネチャは型を返し `Person` ます。
* `Student`は、 `Instructor` から派生した型または型を返すことができ `Person` ます。 

フォーマッタがオブジェクトのみを処理するようにするには、 `Student` <xref:Microsoft.AspNetCore.Mvc.Formatters.OutputFormatterCanWriteContext.Object> メソッドに提供されたコンテキストオブジェクトのの型を確認し `CanWriteResult` ます。 アクションメソッドがを返す場合 `IActionResult` :

* を使用する必要はありません `CanWriteResult` 。
* メソッドは、 `CanWriteType` ランタイム型を受け取ります。

<a id="read-write"></a>

### <a name="override-readrequestbodyasync-and-writeresponsebodyasync"></a>ReadRequestBodyAsync と WriteResponseBodyAsync のオーバーライド

逆シリアル化またはシリアル化は、またはで実行され `ReadRequestBodyAsync` `WriteResponseBodyAsync` ます。 次の例は、依存関係挿入コンテナーからサービスを取得する方法を示しています。 コンストラクターのパラメーターからサービスを取得できません。

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_WriteResponseBodyAsync)]

## <a name="how-to-configure-mvc-to-use-a-custom-formatter"></a>カスタム フォーマッタを使用するように MVC を構成する方法

カスタム フォーマッタを使用するには、`InputFormatters` または `OutputFormatters` コレクションにフォーマッタ クラスのインスタンスを追加します。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](custom-formatters/samples/2.x/CustomFormattersSample/Startup.cs?name=mvcoptions&highlight=3-4)]

::: moniker-end

フォーマッタは、挿入した順序で評価されます。 最初のものが優先されます。

## <a name="the-complete-vcardinputformatter-class"></a>完全な `VcardInputFormatter` クラス

次のコードは、サンプルのクラスを示してい `VcardInputFormatter` ます。 [](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples)

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardInputFormatter.cs?name=snippet_Class)]

## <a name="test-the-app"></a>アプリケーションをテストする

[この記事のサンプルアプリを実行](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples)します。これにより、基本的な vCard 入力フォーマッタと出力フォーマッタが実装されます。 アプリは次のような Vcard を読み取り、書き込みます。

```
BEGIN:VCARD
VERSION:2.1
N:Davolio;Nancy
FN:Nancy Davolio
END:VCARD
```

VCard の出力を表示するには、アプリを実行し、Accept ヘッダーを含む Get 要求をに送信し `text/vcard` `https://localhost:5001/api/contacts` ます。

アドレス帳のメモリ内コレクションに vCard を追加するには、次のようにします。

* `Post`Postman などのツールを使用してに要求を送信し `/api/contacts` ます。
* `Content-Type` ヘッダーを `text/vcard` に設定します。
* `vCard`本文には、前の例のように書式設定されたテキストを設定します。

## <a name="additional-resources"></a>その他のリソース

* <xref:web-api/advanced/formatting>
* <xref:grpc/dotnet-grpc>
