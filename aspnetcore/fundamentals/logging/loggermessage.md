---
title: ASP.NET Core での LoggerMessage による高パフォーマンスのログ記録
author: rick-anderson
description: 高パフォーマンスのログ記録シナリオにおいて、必要とするオブジェクト割り当ての数が少ない、キャッシュ可能なデリゲートを LoggerMessage を使用して作成する方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/26/2019
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
uid: fundamentals/logging/loggermessage
ms.openlocfilehash: d6fae8d2e056c0a578b64322bb82027523904254
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587399"
---
# <a name="high-performance-logging-with-loggermessage-in-aspnet-core"></a>ASP.NET Core での LoggerMessage による高パフォーマンスのログ記録

::: moniker range=">= aspnetcore-3.0"

<xref:Microsoft.Extensions.Logging.LoggerMessage> 機能では、<xref:Microsoft.Extensions.Logging.LoggerExtensions.LogInformation*> や <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogDebug*> のような[ロガー拡張メソッド](xref:Microsoft.Extensions.Logging.LoggerExtensions)と比較して、必要なオブジェクト割り当ての数が少なくコンピューティング オーバーヘッドが小さいキャッシュ可能なデリゲートが作成されます。 高パフォーマンスのログ記録シナリオの場合は、<xref:Microsoft.Extensions.Logging.LoggerMessage> パターンを使用します。

<xref:Microsoft.Extensions.Logging.LoggerMessage> には、ロガー拡張メソッドに比べて次のようなパフォーマンス上の利点があります。

* ロガー拡張メソッドでは、`int` などの値の型を `object` に "ボックス化" (変換) する必要があります。 <xref:Microsoft.Extensions.Logging.LoggerMessage> パターンでは、静的な <xref:System.Action> フィールドと、厳密に型指定されたパラメーターを持つ拡張メソッドを使用してボックス化を回避します。
* ロガー拡張メソッドでは、ログ メッセージが書き込まれるたびにメッセージ テンプレート (名前付きの書式文字列) を解析する必要があります。 <xref:Microsoft.Extensions.Logging.LoggerMessage> では、メッセージを定義するときに、一度テンプレートを解析する必要があるだけです。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/logging/loggermessage/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

サンプル アプリでは、基本的な見積もり追跡システムで <xref:Microsoft.Extensions.Logging.LoggerMessage> 機能を実演します。 アプリでは、メモリ内のデータベースを使用して見積もりの追加および削除を行います。 これらの操作が実行されると、<xref:Microsoft.Extensions.Logging.LoggerMessage> パターンによってログ メッセージが生成されます。

## <a name="loggermessagedefine"></a>LoggerMessage.Define

[Define(LogLevel, EventId, String)](xref:Microsoft.Extensions.Logging.LoggerMessage.Define*) では、メッセージをログ記録するための <xref:System.Action> デリゲートが作成されます。 <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> オーバー ロードでは、名前付きの書式文字列 (テンプレート) に対して最大 6 個の型パラメーターを渡すことを許可します。

<xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> メソッドに指定された文字列はテンプレートであり、補間文字列ではありません。 プレースホルダーは、型が指定された順に入力されます。 テンプレート内のプレースホルダー名はわかりやすく、テンプレート間で一貫している必要があります。 それらは構造化されたログ データ内でプロパティ名としての役割を果たします。 プレースホルダー名には [Pascal 形式](/dotnet/standard/design-guidelines/capitalization-conventions)を推奨します。 たとえば、`{Count}`、`{FirstName}` のようになります。

各ログ メッセージは、[LoggerMessage.Define](xref:Microsoft.Extensions.Logging.LoggerMessage.Define*) によって作成された静的フィールドに保持される <xref:System.Action> です。 たとえば、サンプル アプリでは、インデックス ページに対する GET 要求に関するログ メッセージを記述するフィールドを作成します (*Internal/LoggerExtensions.cs*)。

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet1)]

<xref:System.Action> の場合は、次を指定します。

* ログ レベル。
* 静的な拡張メソッドの名前を持つ一意のイベント識別子です (<xref:Microsoft.Extensions.Logging.EventId>)。
* メッセージ テンプレート (名前付き書式文字列)。 

サンプル アプリのインデックス ページに対する要求では、次の設定を行います。

* ログ レベルを `Information` に設定します。
* イベント ID を `IndexPageRequested` メソッドの名前を含む `1` に設定します。
* メッセージ テンプレート (名前付き書式文字列) を文字列に設定します。

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet5)]

構造化されたログ ストアは、イベント ID によって指定されているとき、ログ記録を強化するために、イベント名を使用する場合があります。 たとえば、[Serilog](https://github.com/serilog/serilog-extensions-logging) はイベント名を使用します。

<xref:System.Action> は厳密に型指定された拡張メソッドによって呼び出されます。 `IndexPageRequested` メソッドは、サンプル アプリにおけるインデックス ページの GET 要求に関するメッセージを記録します。

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet9)]

`IndexPageRequested` は、*Pages/Index.cshtml.cs* に含まれる `OnGetAsync` メソッドのロガー上で呼び出されます。

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet2&highlight=3)]

アプリのコンソール出力を検査する:

```console
info: LoggerMessageSample.Pages.IndexModel[1]
      => RequestId:0HL90M6E7PHK4:00000001 RequestPath:/ => /Index
      GET request for Index page
```

ログ メッセージにパラメーターを渡すには、静的フィールドを作成するときに最大 6 個の型を定義します。 サンプル アプリでは、<xref:System.Action> フィールドに対して `string` 型を定義して見積もりを追加すると、文字列が記録されます。

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet2)]

デリゲートのログ メッセージ テンプレートは、そのプレースホルダー値を指定された型で受け取ります。 サンプル アプリでは、見積もりを追加するためのデリゲートを定義します。ここで見積もりパラメーターは `string` となります。

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet6)]

見積もりを追加するための静的な拡張メソッドである `QuoteAdded` は、見積もり引数の値を受け取って、<xref:System.Action> デリゲートに渡します。

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet10)]

インデックス ページのページ モデル (*Pages/Index.cshtml.cs*) では、メッセージを記録するために `QuoteAdded` が呼び出されます。

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet3&highlight=6)]

アプリのコンソール出力を検査する:

```console
info: LoggerMessageSample.Pages.IndexModel[2]
      => RequestId:0HL90M6E7PHK5:0000000A RequestPath:/ => /Index
      Quote added (Quote = 'You can avoid reality, but you cannot avoid the 
          consequences of avoiding reality. - Ayn Rand')
```

サンプル アプリでは、見積もり削除用の [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) パターンを実装します。 削除操作に成功した場合、情報メッセージが記録されます。 例外がスローされた場合は、削除操作に対してエラー メッセージが記録されます。 削除操作が失敗した場合のログ メッセージには、例外スタック トレースが含まれます (*Internal/LoggerExtensions.cs*)。

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet3)]

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet7)]

`QuoteDeleteFailed` 内のデリゲートに例外が渡される方法に注目してください。

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet11)]

インデックス ページのページ モデルでは、見積もりの削除に成功すると、ロガー上で `QuoteDeleted` メソッドが呼び出されます。 削除対象の見積もりが見つからない場合は、<xref:System.ArgumentNullException> がスローされます。 例外は [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) ステートメントによってトラップされ、[catch](/dotnet/csharp/language-reference/keywords/try-catch) ブロック (*Pages/Index.cshtml.cs*) 内のロガーで `QuoteDeleteFailed` メソッドを呼び出すことにより記録されます。

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet5&highlight=9,13)]

見積もりが正常に削除されたら、アプリのコンソール出力を検査します。

```console
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:00000016 RequestPath:/ => /Index
      Quote deleted (Quote = 'You can avoid reality, but you cannot avoid the 
          consequences of avoiding reality. - Ayn Rand' Id = 1)
```

見積もりの削除が失敗したら、アプリのコンソール出力を検査します。 例外はログ メッセージに含められます。

```console
LoggerMessageSample.Pages.IndexModel: Error: Quote delete failed (Id = 999)

System.NullReferenceException: Object reference not set to an instance of an object.
   at lambda_method(Closure , ValueBuffer )
   at System.Linq.Enumerable.SelectEnumerableIterator`2.MoveNext()
   at Microsoft.EntityFrameworkCore.InMemory.Query.Internal.InMemoryShapedQueryCompilingExpressionVisitor.AsyncQueryingEnumerable`1.AsyncEnumerator.MoveNextAsync()
   at Microsoft.EntityFrameworkCore.Query.ShapedQueryCompilingExpressionVisitor.SingleOrDefaultAsync[TSource](IAsyncEnumerable`1 asyncEnumerable, CancellationToken cancellationToken)
   at Microsoft.EntityFrameworkCore.Query.ShapedQueryCompilingExpressionVisitor.SingleOrDefaultAsync[TSource](IAsyncEnumerable`1 asyncEnumerable, CancellationToken cancellationToken)
   at LoggerMessageSample.Pages.IndexModel.OnPostDeleteQuoteAsync(Int32 id) in c:\Users\guard\Documents\GitHub\Docs\aspnetcore\fundamentals\logging\loggermessage\samples\3.x\LoggerMessageSample\Pages\Index.cshtml.cs:line 77
```

## <a name="loggermessagedefinescope"></a>LoggerMessage.DefineScope

[DefineScope(String)](xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*) は、[ログ スコープ](xref:fundamentals/logging/index#log-scopes)を定義するための <xref:System.Func%601> デリゲートを作成します。 <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> オーバー ロードでは、名前付きの書式文字列 (テンプレート) に対して最大 3 個の型パラメーターを渡すことを許可します。

<xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> メソッドの場合と同様に、<xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> メソッドに指定された文字列はテンプレートであり、補間文字列ではありません。 プレースホルダーは、型が指定された順に入力されます。 テンプレート内のプレースホルダー名はわかりやすく、テンプレート間で一貫している必要があります。 それらは構造化されたログ データ内でプロパティ名としての役割を果たします。 プレースホルダー名には [Pascal 形式](/dotnet/standard/design-guidelines/capitalization-conventions)を推奨します。 たとえば、`{Count}`、`{FirstName}` のようになります。

<xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> メソッドを使用して、一連のログ メッセージに適用する[ログ スコープ](xref:fundamentals/logging/index#log-scopes)を定義します。

サンプル アプリには、データベース内のすべての見積もりを削除するための **[すべてクリア]** ボタンがあります。 見積もりを削除するには、一度に 1 つ削除します。 見積もりを削除するたびに、ロガーで `QuoteDeleted` メソッドが呼び出されます。 ログ スコープは、これらのログ メッセージに追加されます。

*appsettings.json* のコンソール ロガーのセクションで、`IncludeScopes` を有効にします。

[!code-json[](loggermessage/samples/3.x/LoggerMessageSample/appsettings.json?highlight=3-5)]

ログ スコープを作成するには、スコープに対して <xref:System.Func%601> デリゲートを保持するフィールドを追加します。 サンプル アプリでは、`_allQuotesDeletedScope` と呼ばれるフィールドを作成します (*Internal/LoggerExtensions.cs*)。

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet4)]

<xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> を使用してデリゲートを作成します。 デリゲートを呼び出すとき、テンプレート引数として使用するように最大 3 つの型を指定することができます。 サンプル アプリでは、削除された見積もりの数 (`int` 型) を含むメッセージ テンプレートが使用されます。

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet8)]

ログ メッセージ用の静的な拡張メソッドを指定します。 メッセージ テンプレートに表示される名前付きプロパティの任意の型パラメーターを含めます。 サンプル アプリでは、削除する見積もりの `count` を取得し、`_allQuotesDeletedScope` を返します。

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet12)]

スコープは、[using](/dotnet/csharp/language-reference/keywords/using-statement) ブロックでログ拡張呼び出しをラップします。

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet4&highlight=5-6,14)]

アプリのコンソール出力のログ メッセージを検査します。 次の結果には、削除された 3 つの見積もりに加えて、ログ スコープ メッセージが表示されています。

```console
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:0000002E RequestPath:/ => /Index => 
          All quotes deleted (Count = 3)
      Quote deleted (Quote = 'Quote 1' Id = 2)
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:0000002E RequestPath:/ => /Index => 
          All quotes deleted (Count = 3)
      Quote deleted (Quote = 'Quote 2' Id = 3)
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:0000002E RequestPath:/ => /Index => 
          All quotes deleted (Count = 3)
      Quote deleted (Quote = 'Quote 3' Id = 4)
```

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<xref:Microsoft.Extensions.Logging.LoggerMessage> 機能では、<xref:Microsoft.Extensions.Logging.LoggerExtensions.LogInformation*> や <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogDebug*> のような[ロガー拡張メソッド](xref:Microsoft.Extensions.Logging.LoggerExtensions)と比較して、必要なオブジェクト割り当ての数が少なくコンピューティング オーバーヘッドが小さいキャッシュ可能なデリゲートが作成されます。 高パフォーマンスのログ記録シナリオの場合は、<xref:Microsoft.Extensions.Logging.LoggerMessage> パターンを使用します。

<xref:Microsoft.Extensions.Logging.LoggerMessage> には、ロガー拡張メソッドに比べて次のようなパフォーマンス上の利点があります。

* ロガー拡張メソッドでは、`int` などの値の型を `object` に "ボックス化" (変換) する必要があります。 <xref:Microsoft.Extensions.Logging.LoggerMessage> パターンでは、静的な <xref:System.Action> フィールドと、厳密に型指定されたパラメーターを持つ拡張メソッドを使用してボックス化を回避します。
* ロガー拡張メソッドでは、ログ メッセージが書き込まれるたびにメッセージ テンプレート (名前付きの書式文字列) を解析する必要があります。 <xref:Microsoft.Extensions.Logging.LoggerMessage> では、メッセージを定義するときに、一度テンプレートを解析する必要があるだけです。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/logging/loggermessage/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

サンプル アプリでは、基本的な見積もり追跡システムで <xref:Microsoft.Extensions.Logging.LoggerMessage> 機能を実演します。 アプリでは、メモリ内のデータベースを使用して見積もりの追加および削除を行います。 これらの操作が実行されると、<xref:Microsoft.Extensions.Logging.LoggerMessage> パターンによってログ メッセージが生成されます。

## <a name="loggermessagedefine"></a>LoggerMessage.Define

[Define(LogLevel, EventId, String)](xref:Microsoft.Extensions.Logging.LoggerMessage.Define*) では、メッセージをログ記録するための <xref:System.Action> デリゲートが作成されます。 <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> オーバー ロードでは、名前付きの書式文字列 (テンプレート) に対して最大 6 個の型パラメーターを渡すことを許可します。

<xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> メソッドに指定された文字列はテンプレートであり、補間文字列ではありません。 プレースホルダーは、型が指定された順に入力されます。 テンプレート内のプレースホルダー名はわかりやすく、テンプレート間で一貫している必要があります。 それらは構造化されたログ データ内でプロパティ名としての役割を果たします。 プレースホルダー名には [Pascal 形式](/dotnet/standard/design-guidelines/capitalization-conventions)を推奨します。 たとえば、`{Count}`、`{FirstName}` のようになります。

各ログ メッセージは、[LoggerMessage.Define](xref:Microsoft.Extensions.Logging.LoggerMessage.Define*) によって作成された静的フィールドに保持される <xref:System.Action> です。 たとえば、サンプル アプリでは、インデックス ページに対する GET 要求に関するログ メッセージを記述するフィールドを作成します (*Internal/LoggerExtensions.cs*)。

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet1)]

<xref:System.Action> の場合は、次を指定します。

* ログ レベル。
* 静的な拡張メソッドの名前を持つ一意のイベント識別子です (<xref:Microsoft.Extensions.Logging.EventId>)。
* メッセージ テンプレート (名前付き書式文字列)。 

サンプル アプリのインデックス ページに対する要求では、次の設定を行います。

* ログ レベルを `Information` に設定します。
* イベント ID を `IndexPageRequested` メソッドの名前を含む `1` に設定します。
* メッセージ テンプレート (名前付き書式文字列) を文字列に設定します。

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet5)]

構造化されたログ ストアは、イベント ID によって指定されているとき、ログ記録を強化するために、イベント名を使用する場合があります。 たとえば、[Serilog](https://github.com/serilog/serilog-extensions-logging) はイベント名を使用します。

<xref:System.Action> は厳密に型指定された拡張メソッドによって呼び出されます。 `IndexPageRequested` メソッドは、サンプル アプリにおけるインデックス ページの GET 要求に関するメッセージを記録します。

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet9)]

`IndexPageRequested` は、*Pages/Index.cshtml.cs* に含まれる `OnGetAsync` メソッドのロガー上で呼び出されます。

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet2&highlight=3)]

アプリのコンソール出力を検査する:

```console
info: LoggerMessageSample.Pages.IndexModel[1]
      => RequestId:0HL90M6E7PHK4:00000001 RequestPath:/ => /Index
      GET request for Index page
```

ログ メッセージにパラメーターを渡すには、静的フィールドを作成するときに最大 6 個の型を定義します。 サンプル アプリでは、<xref:System.Action> フィールドに対して `string` 型を定義して見積もりを追加すると、文字列が記録されます。

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet2)]

デリゲートのログ メッセージ テンプレートは、そのプレースホルダー値を指定された型で受け取ります。 サンプル アプリでは、見積もりを追加するためのデリゲートを定義します。ここで見積もりパラメーターは `string` となります。

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet6)]

見積もりを追加するための静的な拡張メソッドである `QuoteAdded` は、見積もり引数の値を受け取って、<xref:System.Action> デリゲートに渡します。

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet10)]

インデックス ページのページ モデル (*Pages/Index.cshtml.cs*) では、メッセージを記録するために `QuoteAdded` が呼び出されます。

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet3&highlight=6)]

アプリのコンソール出力を検査する:

```console
info: LoggerMessageSample.Pages.IndexModel[2]
      => RequestId:0HL90M6E7PHK5:0000000A RequestPath:/ => /Index
      Quote added (Quote = 'You can avoid reality, but you cannot avoid the 
          consequences of avoiding reality. - Ayn Rand')
```

サンプル アプリでは、見積もり削除用の [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) パターンを実装します。 削除操作に成功した場合、情報メッセージが記録されます。 例外がスローされた場合は、削除操作に対してエラー メッセージが記録されます。 削除操作が失敗した場合のログ メッセージには、例外スタック トレースが含まれます (*Internal/LoggerExtensions.cs*)。

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet3)]

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet7)]

`QuoteDeleteFailed` 内のデリゲートに例外が渡される方法に注目してください。

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet11)]

インデックス ページのページ モデルでは、見積もりの削除に成功すると、ロガー上で `QuoteDeleted` メソッドが呼び出されます。 削除対象の見積もりが見つからない場合は、<xref:System.ArgumentNullException> がスローされます。 例外は [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) ステートメントによってトラップされ、[catch](/dotnet/csharp/language-reference/keywords/try-catch) ブロック (*Pages/Index.cshtml.cs*) 内のロガーで `QuoteDeleteFailed` メソッドを呼び出すことにより記録されます。

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet5&highlight=14,18)]

見積もりが正常に削除されたら、アプリのコンソール出力を検査します。

```console
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:00000016 RequestPath:/ => /Index
      Quote deleted (Quote = 'You can avoid reality, but you cannot avoid the 
          consequences of avoiding reality. - Ayn Rand' Id = 1)
```

見積もりの削除が失敗したら、アプリのコンソール出力を検査します。 例外はログ メッセージに含められます。

```console
fail: LoggerMessageSample.Pages.IndexModel[5]
      => RequestId:0HL90M6E7PHK5:00000010 RequestPath:/ => /Index
      Quote delete failed (Id = 999)
System.ArgumentNullException: Value cannot be null.
Parameter name: entity
   at Microsoft.EntityFrameworkCore.Utilities.Check.NotNull[T]
       (T value, String parameterName)
   at Microsoft.EntityFrameworkCore.DbContext.Remove[TEntity](TEntity entity)
   at Microsoft.EntityFrameworkCore.Internal.InternalDbSet`1.Remove(TEntity entity)
   at LoggerMessageSample.Pages.IndexModel.<OnPostDeleteQuoteAsync>d__14.MoveNext() 
      in <PATH>\sample\Pages\Index.cshtml.cs:line 87
```

## <a name="loggermessagedefinescope"></a>LoggerMessage.DefineScope

[DefineScope(String)](xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*) は、[ログ スコープ](xref:fundamentals/logging/index#log-scopes)を定義するための <xref:System.Func%601> デリゲートを作成します。 <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> オーバー ロードでは、名前付きの書式文字列 (テンプレート) に対して最大 3 個の型パラメーターを渡すことを許可します。

<xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> メソッドの場合と同様に、<xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> メソッドに指定された文字列はテンプレートであり、補間文字列ではありません。 プレースホルダーは、型が指定された順に入力されます。 テンプレート内のプレースホルダー名はわかりやすく、テンプレート間で一貫している必要があります。 それらは構造化されたログ データ内でプロパティ名としての役割を果たします。 プレースホルダー名には [Pascal 形式](/dotnet/standard/design-guidelines/capitalization-conventions)を推奨します。 たとえば、`{Count}`、`{FirstName}` のようになります。

<xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> メソッドを使用して、一連のログ メッセージに適用する[ログ スコープ](xref:fundamentals/logging/index#log-scopes)を定義します。

サンプル アプリには、データベース内のすべての見積もりを削除するための **[すべてクリア]** ボタンがあります。 見積もりを削除するには、一度に 1 つ削除します。 見積もりを削除するたびに、ロガーで `QuoteDeleted` メソッドが呼び出されます。 ログ スコープは、これらのログ メッセージに追加されます。

*appsettings.json* のコンソール ロガーのセクションで、`IncludeScopes` を有効にします。

[!code-json[](loggermessage/samples/2.x/LoggerMessageSample/appsettings.json?highlight=3-5)]

ログ スコープを作成するには、スコープに対して <xref:System.Func%601> デリゲートを保持するフィールドを追加します。 サンプル アプリでは、`_allQuotesDeletedScope` と呼ばれるフィールドを作成します (*Internal/LoggerExtensions.cs*)。

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet4)]

<xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> を使用してデリゲートを作成します。 デリゲートを呼び出すとき、テンプレート引数として使用するように最大 3 つの型を指定することができます。 サンプル アプリでは、削除された見積もりの数 (`int` 型) を含むメッセージ テンプレートが使用されます。

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet8)]

ログ メッセージ用の静的な拡張メソッドを指定します。 メッセージ テンプレートに表示される名前付きプロパティの任意の型パラメーターを含めます。 サンプル アプリでは、削除する見積もりの `count` を取得し、`_allQuotesDeletedScope` を返します。

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet12)]

スコープは、[using](/dotnet/csharp/language-reference/keywords/using-statement) ブロックでログ拡張呼び出しをラップします。

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet4&highlight=5-6,14)]

アプリのコンソール出力のログ メッセージを検査します。 次の結果には、削除された 3 つの見積もりに加えて、ログ スコープ メッセージが表示されています。

```console
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:0000002E RequestPath:/ => /Index => 
          All quotes deleted (Count = 3)
      Quote deleted (Quote = 'Quote 1' Id = 2)
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:0000002E RequestPath:/ => /Index => 
          All quotes deleted (Count = 3)
      Quote deleted (Quote = 'Quote 2' Id = 3)
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:0000002E RequestPath:/ => /Index => 
          All quotes deleted (Count = 3)
      Quote deleted (Quote = 'Quote 3' Id = 4)
```

::: moniker-end

## <a name="additional-resources"></a>その他の技術情報

* [ログ](xref:fundamentals/logging/index)
