---
title: パート 8、ASP.NET Core Razor ページへの検証の追加
author: rick-anderson
description: Razor ページのチュートリアル シリーズのパート 8。
ms.author: riande
ms.custom: mvc
ms.date: 7/23/2019
no-loc:
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: tutorials/razor-pages/validation
ms.openlocfilehash: 1bc9b2b081c58392bd2c226ebedc804b43810bc2
ms.sourcegitcommit: d65a027e78bf0b83727f975235a18863e685d902
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/26/2020
ms.locfileid: "85408852"
---
# <a name="part-8-add-validation-to-an-aspnet-core-razor-page"></a>パート 8、ASP.NET Core Razor ページへの検証の追加

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

ここでは、`Movie` モデルに検証ロジックを追加します。 検証規則は、ユーザーがムービーを作成または編集するときに、いつでも適用できます。

## <a name="validation"></a>検証

ソフトウェア開発には、[DRY](https://wikipedia.org/wiki/Don%27t_repeat_yourself) ("**D**on't **R**epeat **Y**ourself" (繰り返しを避けること)) という重要な理念があります。 Razor ページでは、機能を一度規定したら、それをアプリ全体に反映する開発を推奨しています。 DRY は次の場合に役立ちます。

* アプリのコード量を減らす。
* コードのエラーが発生する可能性を低くし、テストと保守を簡単にする。

Razor ページと Entity Framework が提供している検証のサポートは、DRY 原則の好例です。 検証規則は、1 つの場所 (モデル クラス内) で宣言的に規定され、アプリの任意の場所で適用されます。

## <a name="add-validation-rules-to-the-movie-model"></a>検証規則をムービー モデルに追加する

DataAnnotations 名前空間には、クラスまたはプロパティに宣言的に適用される一連の組み込みの検証属性があります。 また、DataAnnotations には、書式設定を支援し、どの検証を行わない `DataType` のような書式設定属性もあります。

組み込みの `Required`、`StringLength`、`RegularExpression`、および `Range` 検証属性を利用するように、`Movie` クラスを更新します。

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Models/MovieDateRatingDA.cs?name=snippet1)]

検証属性では、適用対象のモデル プロパティに適用する動作が指定されます。

* `Required` および `MinimumLength` 属性は、プロパティに値が必要であることを示します。ただし、この検証を満たすためにユーザーが空白を入力することは禁止されていません。
* `RegularExpression` 属性は、入力できる文字を制限するために使用されます。 上のコード "Genre" では:

  * 文字のみを使用する必要があります。
  * 最初の文字は大文字にする必要があります。 空白、数字、特殊文字は使用できません。

* `RegularExpression` "評価":

  * 最初の文字が大文字である必要があります。
  * 後続のスペースでは、特殊文字と数字が使用できます。 "PG-13" は評価に対して有効ですが、"Genre" に対しては失敗します。

* `Range` 属性は、指定した範囲内に値を制限します。
* `StringLength` 属性では、文字列プロパティの最大長を設定でき、オプションとして最小長も設定できます。
* 値の型 (`decimal`、`int`、`float`、`DateTime` など) は本質的に必須ではなく、`[Required]` 属性を必要としません。

ASP.NET Core によって検証規則が自動的に適用されるようにすると、アプリをより堅牢にできます。 また、ユーザーが何かを検証することを忘れてしまい、データベースに不適切なデータが誤って格納されることもなくなります。

### <a name="validation-error-ui-in-razor-pages"></a>Razor ページの検証エラー UI

アプリを実行し、[Pages\\Movies]\(ページ\\ムービー\) に移動します。

**[Create New]\(新規作成\)** リンクを選択します。 いくつか無効な値を入力してフォームを設定します。 jQuery クライアント側検証がエラーを検出すると、エラー メッセージが表示されます。

![複数 jQuery クライアント側検証エラーが表示されたムービー ビュー フォーム](validation/_static/val.png)

[!INCLUDE[](~/includes/localization/currency.md)]

無効な値を含む各フィールドに、検証エラー メッセージが自動的に表示されることがわかります。 エラーは、(JavaScript と jQuery を使用している) クライアント側とサーバー側 (ユーザーが JavaScript を無効にしている場合) の両方に適用されます。

大きな利点は、[Create]/(作成/) ページまたは [Edit]/(編集/) ページの変更が**必要ない**ことです。 一度 DataAnnotations がモデルに適用されると、検証 UI は有効化されます。 このチュートリアルで作成した Razor ページは、(`Movie` モデル クラスのプロパティの検証属性を使用して) 検証規則を自動的に抽出しました。 [Edit]/(編集/) ページを使用して検証をテストすると、同じ検証が適用されます。

クライアント側の検証エラーがなくなるまで、フォーム データはサーバーにポストされません。 次のうち 1 つまたは複数の方法で、フォーム データがポストされていないことを確認します。

* `OnPostAsync` メソッドにブレークポイントを設定します。 フォームを送信します ( **[Create]/(作成/)** または **[Save]/(保存/)** を選択します)。 ブレークポイントがヒットすることはありません。
* [Fiddler ツール](https://www.telerik.com/fiddler)を使用します。
* ブラウザー開発者向けツールを使用して、ネットワーク トラフィックを監視します。

### <a name="server-side-validation"></a>サーバー側の検証

ブラウザーで JavaScript が無効にされている場合、エラーがあるフォームを送信すると、サーバーにポストされます。

必要に応じて、サーバー側の検証をテストします。

* ブラウザーで JavaScript を無効にします。 ブラウザーの開発者ツールを使用して JavaScript を無効にすることができます。 ブラウザーで JavaScript を無効にすることができない場合は、別のブラウザーを試してください。
* [Create] または [Edit] ページの `OnPostAsync` メソッドにブレークポイントを設定します。
* 無効なデータを含むフォームを送信します。
* モデルの状態が無効であることを確認します。

  ```csharp
   if (!ModelState.IsValid)
   {
      return Page();
   }
  ```
  
または、[サーバーでクライアント側検証を無効にする](xref:mvc/models/validation#disable-client-side-validation)こともできます。

次のコードは、チュートリアルで前にスキャフォールディング処理した *Create.cshtml* ページの一部を示しています。 [Create] または [Edit] ページにおいて、最初のフォームの表示と、エラーイベント時におけるフォームの再表示のために使用されます。

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Create.cshtml?range=14-20)]

[入力タグ ヘルパー](xref:mvc/views/working-with-forms)は [DataAnnotations](/aspnet/mvc/overview/older-versions/mvc-music-store/mvc-music-store-part-6) 属性を使用し、クライアント側で jQuery 検証に必要な HTML 属性を生成します。 [検証タグ ヘルパー](xref:mvc/views/working-with-forms#the-validation-tag-helpers)には検証エラーが表示されます。 詳しくは、[検証に関する記事](xref:mvc/models/validation)をご覧ください。

[Create] ページと [Edit] ページ内に検証規則はありません。 検証規則とエラー文字列は、`Movie` クラスでのみ指定されています。 これらの検証規則は、`Movie` モデルを編集する Razor ページに自動的に適用されます。

検証ロジックの変更が必要な場合は、モデルでのみ変更します。 検証は常にアプリケーション全体に適用されます (検証ロジックは 1 か所で定義されます)。 検証が 1 か所であることは、コードが整理された状態を保つことを助け、保守と更新を簡単にします。

## <a name="using-datatype-attributes"></a>DataType 属性の使用

`Movie` クラスを調べます。 `System.ComponentModel.DataAnnotations` 名前空間には、組み込みの検証属性セットに加え、書式設定の属性もあります。 `DataType` 属性は、`ReleaseDate` および `Price` プロパティに適用されます。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie/Models/MovieDateRatingDA.cs?highlight=2,6&name=snippet2)]

`DataType` 属性は、ビュー エンジンに対して、データの書式設定のヒントのみを提供します (また、URL の場合に `<a>`、電子メールの場合に `<a href="mailto:EmailAddress.com">` などの属性を提供します)。 データの書式を検証するためには、`RegularExpression` 属性を使用してください。 `DataType` 属性は、データベースの組み込み型よりも具体的なデータ型を指定するために使用されます。 `DataType` 属性は、検証属性ではありません。 サンプル アプリケーションでは、日付のみが表示され、時刻は表示されません。

`DataType` 列挙型は、Date、Time、PhoneNumber、Currency、EmailAddress など、多くの型のために用意されています。 また、`DataType` 属性を使用して、アプリケーションで型固有の機能を自動的に提供することもできます。 たとえば、`DataType.EmailAddress` に対して `mailto:` リンクを作成できます。 HTML 5 をサポートしているブラウザーでは、`DataType.Date` に日付セレクターを提供できます。 `DataType` 属性は、HTML 5 ブラウザーが使用する HTML 5 `data-` ("データ ダッシュ" と読みます) 属性を出力します。 `DataType` 属性は、検証を**提供していません**。

`DataType.Date` は、表示される日付の書式を指定しません。 既定で、日付フィールドはサーバーの `CultureInfo` に基づき、既定の書式に従って表示されます。

`[Column(TypeName = "decimal(18, 2)")]` データ注釈は、Entity Framework Core がデータベースの通貨と `Price` を正しくマッピングできるようにするために必要です。 詳細については、「[Data Types](/ef/core/modeling/relational/data-types)」(データ型) を参照してください。

`DisplayFormat` 属性は、日付の書式を明示的に指定するために使用されます。

```csharp
[DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
public DateTime ReleaseDate { get; set; }
```

`ApplyFormatInEditMode` 設定では、編集対象の値を表示するときに適用する必要がある書式設定を指定します。 一部のフィールドでは、この動作が不要なことがあります。 たとえば、通貨値の場合、編集 UI で通貨記号が不要なことがあります。

`DisplayFormat` 属性を単独で使用できますが、一般的に、`DataType` 属性を使用することが推奨されます。 `DataType` 属性は、画面でのレンダリング方法とは異なり、データのセマンティクスを伝達します。また、DisplayFormat にはない、次の利点があります。

* ブラウザーは HTML5 機能を有効にすることができます (たとえば、カレンダー コントロール、ロケールに適した通貨記号、電子メール リンクを表示するときなど)。
* ブラウザーの既定では、ロケールに基づいて正しい書式を使ってデータがレンダリングされます。
* `DataType` 属性を使用すると、ASP.NET Core フレームワークで、適切なフィールド テンプレートを選択し、データをレンダリングすることができます。 `DisplayFormat` を単独で使用する場合、文字列テンプレートが使用されます。

**注:** jQuery の検証は、`Range` 属性と `DateTime` では機能しません。 たとえば、次のコードでは、指定した範囲内の日付であっても、クライアント側の検証エラーが常に表示されます。

```csharp
[Range(typeof(DateTime), "1/1/1966", "1/1/2020")]
   ```

一般的に、モデルに日付をハードコーディングしてコンパイルすることは推奨されません。そのため、`Range` 属性と `DateTime` の使用は推奨されません。

次のコードは、1 行で複数の属性を組み合わせる例です。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/MovieDateRatingDAmult.cs?name=snippet1)]

[Razor ページと EF Core の概要](xref:data/ef-rp/intro)に関するページでは、Razor ページでの EF Core 操作についてより詳しく説明されています。

### <a name="apply-migrations"></a>移行を適用する

クラスに適用された DataAnnotations によって、スキーマが変更されます。 たとえば、`Title`フィールドに適用された DataAnnotations は次のようになります。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/MovieDateRatingDA.cs?name=snippet11)]

* 文字数を 60 に制限します。
* `null` 値を許可しません。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

`Movie` テーブルには現在、次のスキーマがあります。

```sql
CREATE TABLE [dbo].[Movie] (
    [ID]          INT             IDENTITY (1, 1) NOT NULL,
    [Title]       NVARCHAR (MAX)  NULL,
    [ReleaseDate] DATETIME2 (7)   NOT NULL,
    [Genre]       NVARCHAR (MAX)  NULL,
    [Price]       DECIMAL (18, 2) NOT NULL,
    [Rating]      NVARCHAR (MAX)  NULL,
    CONSTRAINT [PK_Movie] PRIMARY KEY CLUSTERED ([ID] ASC)
);
```

上記のスキーマ変更では、EF によって例外がスローされることはありません。 ただし、スキーマがモデルと一致するように、移行を作成してください。

**[ツール]** メニューで、 **[NuGet パッケージ マネージャー]、[パッケージ マネージャー コンソール]** の順に選択します。
PMC で、次のコマンドを入力します。

```powershell
Add-Migration New_DataAnnotations
Update-Database
```

`Update-Database` では、`New_DataAnnotations` クラスの `Up` メソッドが実行されます。 `Up` メソッドを検証します。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Migrations/20190724163003_New_DataAnnotations.cs?name=snippet)]

更新された `Movie` テーブルには、次のスキーマがあります。

```sql
CREATE TABLE [dbo].[Movie] (
    [ID]          INT             IDENTITY (1, 1) NOT NULL,
    [Title]       NVARCHAR (60)   NOT NULL,
    [ReleaseDate] DATETIME2 (7)   NOT NULL,
    [Genre]       NVARCHAR (30)   NOT NULL,
    [Price]       DECIMAL (18, 2) NOT NULL,
    [Rating]      NVARCHAR (5)    NOT NULL,
    CONSTRAINT [PK_Movie] PRIMARY KEY CLUSTERED ([ID] ASC)
);
```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

SQLite では、移行は必要ありません。

---

### <a name="publish-to-azure"></a>Azure に発行する

Azure へのデプロイの詳細については、「[チュートリアル: SQL Database を使用して Azure に ASP.NET Core アプリを作成する](/azure/app-service/app-service-web-tutorial-dotnetcore-sqldb)」をご覧ください。

このたびは、この Razor Pages の紹介を最後までお読みいただきありがとうございました。 このチュートリアルの後は、[Razor Pages と EF Core の概要](xref:data/ef-rp/intro)に関するページにお進みいただくことをお勧めします。

## <a name="additional-resources"></a>その他の技術情報

* <xref:mvc/views/working-with-forms>
* <xref:fundamentals/localization>
* <xref:mvc/views/tag-helpers/intro>
* <xref:mvc/views/tag-helpers/authoring>
* [このチュートリアルの YouTube バージョン](https://youtu.be/b63m66eu7us)

> [!div class="step-by-step"]
> [前へ:新しいフィールドの追加](xref:tutorials/razor-pages/new-field)
