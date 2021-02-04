---
title: ASP.NET Core を使用してネイティブ モバイル アプリのバックエンド サービスを作成する
author: ardalis
description: ASP.NET Core MVC を使用してネイティブ モバイル アプリをサポートするバックエンド サービスを作成する方法について説明します。
ms.author: riande
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
uid: mobile/native-mobile-backend
ms.openlocfilehash: 4e86241771e884ba9079bcdf9a09eebc6acd62c8
ms.sourcegitcommit: e311cfb77f26a0a23681019bd334929d1aaeda20
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99530217"
---
# <a name="create-backend-services-for-native-mobile-apps-with-aspnet-core"></a>ASP.NET Core を使用してネイティブ モバイル アプリのバックエンド サービスを作成する

作成者: [Steve Smith](https://ardalis.com/)

モバイル アプリは、ASP.NET Core バックエンド サービスと通信できます。 iOS シミュレーターと Android エミュレーターからローカル Web サービスに接続する手順については、「[Connect to Local Web Services from iOS Simulators and Android Emulators](/xamarin/cross-platform/deploy-test/connect-to-local-web-services)」 (iOS シミュレーターと Android エミュレーターからローカル Web サービスに接続する) を参照してください。

[サンプル バックエンド サービス コードの表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mobile/native-mobile-backend/sample)

## <a name="the-sample-native-mobile-app"></a>サンプル ネイティブ モバイル アプリ

このチュートリアルでは、ASP.NET Core MVC を使用してネイティブ モバイル アプリをサポートするバックエンド サービスを作成する方法について説明します。 [Xamarin Forms ToDoRest アプリ](/xamarin/xamarin-forms/data-cloud/consuming/rest)をネイティブ クライアントとして使用します。これには、Android、iOS、Windows Universal、Window Phone デバイス用に個別のネイティブ クライアントが含まれています。 リンクされているチュートリアルに従って、ネイティブ アプリを作成し (必要な無料の Xamarin ツールをインストールし)、Xamarin サンプル ソリューションをダウンロードすることができます。 Xamarin サンプルには、ASP.NET Web API 2 サービス プロジェクトが含まれています。この記事の ASP.NET Core アプリで、このプロジェクトを置き換えます (クライアント側に変更は必要ありません)。

![Android スマートフォン上で動作する To Do Rest アプリケーション](native-mobile-backend/_static/todo-android.png)

### <a name="features"></a>特徴

この ToDoRest アプリは、To-Do 項目の一覧表示、追加、削除、更新をサポートしています。 各項目には、ID、名前、メモ、完了したかどうかを示すプロパティがあります。

項目のメイン ビューには、上記のように各項目の名前が表示され、完了しているかどうかがチェックマークで示されます。

`+` アイコンをタップすると、項目の追加ダイアログが開きます。

![項目の追加ダイアログ](native-mobile-backend/_static/todo-android-new-item.png)

メイン リスト画面の項目をタップすると、編集ダイアログが開き、項目の名前、メモ、完了の設定を変更したり、項目を削除したりすることができます。

![項目の編集ダイアログ](native-mobile-backend/_static/todo-android-edit-item.png)

このサンプルは、既定で、developer.xamarin.com でホストされている読み取り専用操作を許可するバックエンド サービスを使用するように構成されています。 次のセクションで作成する ASP.NET Core アプリをコンピューター上で実行して、自分自身をテストするには、アプリの `RestUrl` 定数を更新する必要があります。 `ToDoREST` プロジェクトに移動し、*Constants.cs* ファイルを開きます。 `RestUrl` は、マシンの IP アドレスを含む URL に置き換えます (このアドレスはマシンからではなくデバイス エミュレーターから使用されるため、localhost または 127.0.0.1 には置き換えないでください)。 ポート番号 (5000) も含めます。 このサービスがデバイスで動作することをテストするために、アクティブなファイアウォールがこのポートへのアクセスをブロックしていないことを確認します。

```csharp
// URL of REST service (Xamarin ReadOnly Service)
//public static string RestUrl = "http://developer.xamarin.com:8081/api/todoitems{0}";

// use your machine's IP address
public static string RestUrl = "http://192.168.1.207:5000/api/todoitems/{0}";
```

## <a name="creating-the-aspnet-core-project"></a>ASP.NET Core プロジェクトの作成

Visual Studio で新しい ASP.NET Core Web アプリケーションを作成します。 Web API テンプレートと [No Authentication]\(認証なし\) を選択します。 プロジェクトに *ToDoApi* という名前を付けます。

![Web API プロジェクト テンプレートが選択されている [新しい ASP.NET Core Web アプリケーション] ダイアログ](native-mobile-backend/_static/web-api-template.png)

アプリケーションは、ポート 5000 に対して行われたすべての要求に応答する必要があります。 これを達成するために、`.UseUrls("http://*:5000")` を含むように *Program.cs* を更新します。

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Program.cs?range=10-16&highlight=3)]

> [!NOTE]
> 既定でローカル以外の要求を無視する IIS Express の背後ではなく、アプリケーションを直接実行するようにします。 コマンド プロンプトから [dotnet run](/dotnet/core/tools/dotnet-run) を実行するか、Visual Studio ツールバーの [デバッグ ターゲット] ドロップダウンからアプリケーション名のプロファイルを選択します。

To-Do 項目を表すモデル クラスを追加します。 必須フィールドを `[Required]` 属性でマークします。

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Models/ToDoItem.cs)]

API メソッドには、データを操作する何らかの方法が必要です。 元の Xamarin サンプルで使用されているものと同じ `IToDoRepository` インターフェイスを使用します。

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Interfaces/IToDoRepository.cs)]

このサンプルの実装では、項目のプライベート コレクションを使用します。

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Services/ToDoRepository.cs)]

*Startup.cs* で実装を構成します。

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Startup.cs?highlight=6&range=29-35)]

この時点で、*ToDoItemsController* を作成する準備が整いました。

> [!TIP]
> Web API の作成方法については、[ASP.NET Core MVC と Visual Studio での最初の Web API の構築](../tutorials/first-web-api.md)に関するページを参照してください。

## <a name="creating-the-controller"></a>コントローラーの作成

*ToDoItemsController* プロジェクトに新しいコントローラーを追加します。 Microsoft.AspNetCore.Mvc.Controller から継承する必要があります。 `api/todoitems` で始まるパスに対する要求をコントローラーが処理することを示す `Route` 属性を追加します。 ルート内の `[controller]` トークンはコントローラーの名前に置き換えられます (`Controller` サフィックスは省略されます)。これは特にグローバル ルートの場合に役立ちます。 詳細については、[ルーティング](../fundamentals/routing.md)に関するページを参照してください。

コントローラーを使用するには `IToDoRepository` が機能する必要があります。コントローラーのコンストラクターを介してこの種類のインスタンスを要求します。 実行時に、[依存関係の挿入](../fundamentals/dependency-injection.md)用のフレームワークのサポートを使用して、このインスタンスが提供されます。

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=1-17&highlight=9,14)]

この API は、データ ソースに対して CRUD (Create (作成)、Read (読み取り)、Update (更新)、Delete (削除)) 操作を実行する 4 つの異なる HTTP 動詞をサポートしています。 この中で最も単純な操作は読み取りです。HTTP GET 要求に対応します。

### <a name="reading-items"></a>項目の読み取り

項目一覧の要求は、`List` メソッドに対する GET 要求で行われます。 `List` メソッドの `[HttpGet]` 属性は、このアクションが GET 要求のみを処理する必要があることを示します。 このアクションのルートは、コントローラーで指定されたルートです。 ルートの一部としてアクション名を使用する必要はありません。 各アクションに一意で明確なルートを持たせる必要があります。 ルーティング属性をコントローラー レベルとメソッド レベルの両方で適用して、特定のルートを構築することができます。

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=19-23)]

`List` メソッドは、200 OK 応答コードとすべての ToDo 項目を JSON としてシリアル化して返します。

次のように、[Postman](https://www.getpostman.com/docs/) などのさまざまなツールを使用して、新しい API メソッドをテストできます。

![todoitems の GET 要求を表示する Postman コンソールと、返された 3 つの項目の JSON を示す応答の本文](native-mobile-backend/_static/postman-get.png)

### <a name="creating-items"></a>項目の作成

慣例により、新しいデータ項目の作成は HTTP POST 動詞にマッピングされます。 `Create` メソッドには `[HttpPost]` 属性が適用され、このメソッドは `ToDoItem` インスタンスを受け入れます。 `item` 引数は POST の本文で渡されるため、このパラメーターは `[FromBody]` 属性を指定します。

メソッド内では、データ ストア内で項目が有効であることと事前に存在することが確認され、問題がなければ、リポジトリを使用して追加されます。 `ModelState.IsValid` の確認で[モデルの検証](../mvc/models/validation.md)が実行されます。この確認は、ユーザー入力を受け入れるすべての API メソッドで実行する必要があります。

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=25-46)]

このサンプルでは、​​モバイル クライアントに渡されるエラー コードを含む列挙型を使用しています。

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=91-99)]

Postman を使用して新しい項目の追加をテストします。このときに、要求の本文で JSON 形式の新しいオブジェクトを提供する POST 動詞を選択します。 また、`application/json` の `Content-Type` を指定する要求ヘッダーも追加する必要があります。

![POST と応答が表示される Postman コンソール](native-mobile-backend/_static/postman-post.png)

このメソッドは、新しく作成された項目を応答で返します。

### <a name="updating-items"></a>項目の更新

レコードの変更は、HTTP PUT 要求を使用して行われます。 `Edit` メソッドは、この変更以外は `Create` とほとんど同じです。 レコードが見つからない場合、`Edit` アクションから `NotFound` (404) 応答が返されます。

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=48-69)]

Postman を使用してテストするには、動詞を PUT に変更します。 要求の本文で更新されたオブジェクト データを指定します。

![PUT と応答が表示される Postman コンソール](native-mobile-backend/_static/postman-put.png)

このメソッドが成功した場合は、既存の API との整合性を保つために、`NoContent` (204) 応答が返されます。

### <a name="deleting-items"></a>アイテムを削除する

レコードを削除するには、サービスに対して DELETE 要求を実行し、削除する項目の ID を渡します。 更新と同様に、存在しない項目に対する要求は `NotFound` 応答を受け取ります。 それ以外の成功した要求は `NoContent` (204) 応答を受け取ります。

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=71-88)]

削除機能をテストする場合、要求の本文には何も指定する必要がありません。

![DELETE と応答が表示される Postman コンソール](native-mobile-backend/_static/postman-delete.png)

## <a name="common-web-api-conventions"></a>一般的な Web API 規約

アプリのバックエンド サービスを開発する場合は、横断的な懸案事項を処理するための一連の規約やポリシーが必要になります。 たとえば、前述のサービスでは、見つからなかった特定のレコードに対する要求は、`BadRequest` 応答ではなく `NotFound` 応答を受け取りました。 同様に、モデルにバインドされた種類で渡されたこのサービスに対するコマンドは、常に `ModelState.IsValid` を確認し、無効なモデルの種類の場合に `BadRequest` を返していました。

API の共通ポリシーを特定した場合、通常はそのポリシーを[フィルター](../mvc/controllers/filters.md)にカプセル化できます。 詳細については、[ASP.NET Core MVC アプリケーションで一般的な API ポリシーをカプセル化する方法](/archive/msdn-magazine/2016/august/asp-net-core-real-world-asp-net-core-mvc-filters)に関するページを参照してください。

## <a name="additional-resources"></a>その他のリソース

- [Xamarin. Forms: Web サービス認証](/xamarin/xamarin-forms/data-cloud/authentication/)
- [Xamarin. フォーム: RESTful Web サービスを使用する](/xamarin/xamarin-forms/data-cloud/web-services/rest)
- [Microsoft Learn: Xamarin アプリで REST web サービスを使用する](/learn/modules/consume-rest-services/)
- [Microsoft Learn: ASP.NET Core で web API を作成する](/learn/modules/build-web-api-aspnet-core/)
