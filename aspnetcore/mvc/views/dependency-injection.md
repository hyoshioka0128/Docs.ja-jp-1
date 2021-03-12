---
title: ASP.NET Core でのビューへの依存関係の挿入
author: ardalis
description: ASP.NET Core で MVC ビューへの依存関係の挿入をサポートする方法について説明します。
ms.author: riande
ms.date: 10/14/2016
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
uid: mvc/views/dependency-injection
ms.openlocfilehash: 9d4920a7b572163b048bfb06029d913cee3f9e5a
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587254"
---
# <a name="dependency-injection-into-views-in-aspnet-core"></a>ASP.NET Core でのビューへの依存関係の挿入

作成者: [Steve Smith](https://ardalis.com/)

ASP.NET Core では、ビューへの[依存関係の挿入](xref:fundamentals/dependency-injection)をサポートしています。 この機能は、ビュー要素を作成するためだけに必要なローカライズやデータなど、ビュー固有のサービスに役立ちます。 コントローラーとビューの間で[関心の分離](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns)を保持する必要があります。 ビューに表示されるデータのほとんどは、コントローラーから渡されます。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/dependency-injection/sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="configuration-injection"></a>構成の挿入

*appsettings.json* 値は、ビューに直接挿入できます。

ファイルの例 *appsettings.json* :

```json
{
   "root": {
      "parent": {
         "child": "myvalue"
      }
   }
}
```

`@inject`: `@inject <type> <name>` の構文

`@inject` を使用した例:

```csharp
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration
@{
   string myValue = Configuration["root:parent:child"];
   ...
}
```

## <a name="service-injection"></a>サービスの挿入

`@inject` ディレクティブを使用して、サービスをビューに挿入することができます。 `@inject` は、ビューにプロパティを追加したり、DI を使用してプロパティを設定したりするものと考えることができます。

[!code-cshtml[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Views/ToDo/Index.cshtml?highlight=4,5,15,16,17)]

このビューでは、全体的な統計を示す概要と共に、`ToDoItem` インスタンスのリストが表示されます。 概要は、挿入された `StatisticsService` から作成されます。 このサービスは、*Startup.cs* 内の `ConfigureServices` の依存関係の挿入に登録されます。

[!code-csharp[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Startup.cs?highlight=6,7&range=15-22)]

`StatisticsService` では、`ToDoItem` インスタンスのセットでいくつかの計算を実行します。これには、次のリポジトリを使用してアクセスします。

[!code-csharp[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Model/Services/StatisticsService.cs?highlight=15,20,25)]

サンプルのリポジトリでは、メモリ内のコレクションを使用します。 上記の (メモリ内のすべてのデータを操作する) 実装は、大規模なリモートでアクセスされるデータ セットにはお勧めしません。

サンプルには、ビューとビューに挿入されたサービスにバインドされたモデルからのデータが表示されています。

![合計項目数、完了した項目数、優先度の平均、および優先度と完了を示すブール値を含むタスクのリストを一覧する To Do ビュー。](dependency-injection/_static/screenshot.png)

## <a name="populating-lookup-data"></a>ルックアップ データの作成

ビューの挿入は、ドロップダウン リストなど、UI 要素のオプションの作成に役立ちます。 性別、状態、およびその他の基本設定を指定するオプションを含む、ユーザー プロファイル フォームを検討します。 標準の MVC アプローチを使用するフォームのようなレンダリングには、これらのオプションのセットごとにデータ アクセス サービスを要求し、バインドするオプションの各セットを含むモデルまたは `ViewBag` を作成するために、コントローラーが必要です。

別のアプローチでは、オプションを取得するために、ビューに直接サービスを挿入します。 これにより、このビュー要素の構築ロジックをビュー自体に移動して、コントローラーから要求されるコードの量を最小限にします。 プロファイルの編集フォームを表示するコントローラー アクションは、フォームにプロファイル インスタンスを渡すためにのみ必要です。

[!code-csharp[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Controllers/ProfileController.cs?highlight=9,19)]

これらの基本設定を更新するために使用される HTML フォームには、3 つのプロパティのドロップダウン リストが含まれます。

![名前、性別、状態、好きな色の入力を許可するフォームを含む、[Update Profile] (プロファイルの更新) ビュー。](dependency-injection/_static/updateprofile.png)

これらのリストは、ビューに挿入されたサービスによって作成されます。

[!code-cshtml[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Views/Profile/Index.cshtml?highlight=4,16,17,21,22,26,27)]

`ProfileOptionsService` は、このフォームに必要なデータだけを提供する UI レベルのサービスです。

[!code-csharp[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Model/Services/ProfileOptionsService.cs?highlight=7,13,24)]

> [!IMPORTANT]
> `Startup.ConfigureServices` での依存関係の挿入を介して要求する型を登録するのを忘れないでください。 サービス プロバイダーは [GetRequiredService](/dotnet/api/microsoft.extensions.dependencyinjection.serviceproviderserviceextensions.getrequiredservice) を介して内部的にクエリが実行されるため、未登録の型は実行時に例外をスローします。

## <a name="overriding-services"></a>サービスのオーバーライド

新しいサービスを挿入するだけでなく、この技術はページ上の以前に挿入されたサービスをオーバーライドするためにも使用できます。 以下の図は、最初の例で使用されたページで利用できるすべてのフィールドを示しています。

![@ 記号を入力して、Html、Component、StatsService、Url フィールドが一覧された IntelliSense コンテキスト メニュー](dependency-injection/_static/razor-fields.png)

ご覧のように、既定のフィールドには、`Html`、`Component`、`Url` (さらに、挿入した `StatsService`) が含まれます。 たとえば、既定の HTML ヘルパーを自分のヘルパーに置き換える場合は、`@inject` を使用して簡単に置き換えることができます。

[!code-cshtml[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Views/Helper/Index.cshtml?highlight=3,11)]

既存のサービスを拡張する場合、独自に既存の実装から継承したり、ラップしたりするときに、この技術を使用できます。

## <a name="see-also"></a>参照

* Simon Timms のブログ: [ルックアップ データをビューに取り込む](https://blog.simontimms.com/2015/06/09/getting-lookup-data-into-you-view/)
