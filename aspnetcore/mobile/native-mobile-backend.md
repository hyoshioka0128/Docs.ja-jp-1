---
title: ASP.NET Core を使用してネイティブ モバイル アプリのバックエンド サービスを作成する
author: rick-anderson
description: ASP.NET Core MVC を使用してネイティブ モバイル アプリをサポートするバックエンド サービスを作成する方法について説明します。
ms.author: riande
ms.date: 2/12/2021
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
ms.openlocfilehash: e496b7811cc534b6f0f6dfdb857f6e462b38049e
ms.sourcegitcommit: f77a7467651bab61b24261da9dc5c1dd75fc1fa9
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/17/2021
ms.locfileid: "100564029"
---
# <a name="create-backend-services-for-native-mobile-apps-with-aspnet-core"></a>ASP.NET Core を使用してネイティブ モバイル アプリのバックエンド サービスを作成する

[James Montemagno](https://twitter.com/JamesMontemagno)

モバイル アプリは、ASP.NET Core バックエンド サービスと通信できます。 iOS シミュレーターと Android エミュレーターからローカル Web サービスに接続する手順については、「[Connect to Local Web Services from iOS Simulators and Android Emulators](/xamarin/cross-platform/deploy-test/connect-to-local-web-services)」 (iOS シミュレーターと Android エミュレーターからローカル Web サービスに接続する) を参照してください。

[サンプル バックエンド サービス コードの表示またはダウンロード](https://github.com/xamarin/xamarin-forms-samples/tree/master/WebServices/TodoREST)

## <a name="the-sample-native-mobile-app"></a>サンプル ネイティブ モバイル アプリ

このチュートリアルでは、ASP.NET Core を使用してネイティブモバイルアプリをサポートするバックエンドサービスを作成する方法について説明します。 これは、Android、iOS、Windows 用の個別のネイティブクライアントを含む、ネイティブクライアントとして [TodoRest アプリ](/xamarin/xamarin-forms/data-cloud/consuming/rest) を使用します。 リンクされているチュートリアルに従って、ネイティブ アプリを作成し (必要な無料の Xamarin ツールをインストールし)、Xamarin サンプル ソリューションをダウンロードすることができます。 Xamarin サンプルには ASP.NET Core Web API サービスプロジェクトが含まれています。このプロジェクトでは、この記事の ASP.NET Core アプリが置き換えられます (クライアントが変更する必要はありません)。

![Android スマートフォン上で動作する To Do Rest アプリケーション](native-mobile-backend/_static/todo-android.png)

### <a name="features"></a>特徴

[TodoREST アプリ](https://github.com/xamarin/xamarin-forms-samples/tree/master/WebServices/TodoREST)では、To-Do 項目の一覧表示、追加、削除、および更新がサポートされています。 各項目には、ID、名前、メモ、完了したかどうかを示すプロパティがあります。

項目のメイン ビューには、上記のように各項目の名前が表示され、完了しているかどうかがチェックマークで示されます。

`+` アイコンをタップすると、項目の追加ダイアログが開きます。

![項目の追加ダイアログ](native-mobile-backend/_static/todo-android-new-item.png)

メイン リスト画面の項目をタップすると、編集ダイアログが開き、項目の名前、メモ、完了の設定を変更したり、項目を削除したりすることができます。

![項目の編集ダイアログ](native-mobile-backend/_static/todo-android-edit-item.png)

お使いのコンピューターで実行されている次のセクションで作成した ASP.NET Core アプリに対してテストするには、アプリの定数を更新し [`RestUrl`](https://github.com/xamarin/xamarin-forms-samples/blob/master/WebServices/TodoREST/TodoREST/Constants.cs#L13) ます。

Android エミュレーターはローカルコンピューター上では実行されず、ループバック IP (10.0.2.2) を使用してローカルコンピューターと通信します。 [DeviceInfo](/xamarin/essentials/device-information/)を活用して、正しい URL を使用するためにシステムが実行されているオペレーティングシステムを検出します。

プロジェクトに移動 [`TodoREST`](https://github.com/xamarin/xamarin-forms-samples/tree/master/WebServices/TodoREST/TodoREST) し、ファイルを開き [`Constants.cs`](https://github.com/xamarin/xamarin-forms-samples/blob/master/WebServices/TodoREST/TodoREST/Constants.cs) ます。 *Constants.cs* ファイルには、次の構成が含まれています。

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoREST/Constants.cs" highlight="13":::

必要に応じて、web サービスを Azure などのクラウドサービスにデプロイし、を更新することができ `RestUrl` ます。

## <a name="creating-the-aspnet-core-project"></a>ASP.NET Core プロジェクトの作成

Visual Studio で新しい ASP.NET Core Web アプリケーションを作成します。 [Web API] テンプレートを選択します。 プロジェクトに *TodoAPI* という名前を指定します。

![Web API プロジェクト テンプレートが選択されている [新しい ASP.NET Core Web アプリケーション] ダイアログ](native-mobile-backend/_static/web-api-template.png)

アプリは、モバイルクライアントのクリアテキスト http トラフィックを含む、ポート5000に対して行われたすべての要求に応答する必要があります。 *Startup.cs* を更新して、 <xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection%2A> 開発で実行しないようにします。

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Startup.cs" id="snippet" highlight="7-11":::

> [!NOTE]
> IIS Express の背後ではなく、直接アプリを実行します。 既定では、IIS Express はローカル以外の要求を無視します。 コマンドプロンプトから [dotnet run](/dotnet/core/tools/dotnet-run) を実行するか、Visual Studio ツールバーの [デバッグターゲット] ドロップダウンからアプリ名プロファイルを選択します。

To-Do 項目を表すモデル クラスを追加します。 必須フィールドを `[Required]` 属性でマークします。

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Models/TodoItem.cs":::

API メソッドには、データを操作する何らかの方法が必要です。 元の Xamarin サンプルで使用されているものと同じ `ITodoRepository` インターフェイスを使用します。

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Interfaces/ITodoRepository.cs":::

このサンプルの実装では、項目のプライベート コレクションを使用します。

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Services/TodoRepository.cs":::

*Startup.cs* で実装を構成します。

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Startup.cs" id="snippet2" highlight="3":::

## <a name="creating-the-controller"></a>コントローラーの作成

新しいコントローラーをプロジェクト [TodoItemsController](https://github.com/xamarin/xamarin-forms-samples/tree/master/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs)に追加します。 このメソッドは、から継承する必要があり <xref:Microsoft.AspNetCore.Mvc.ControllerBase> ます。 `api/todoitems` で始まるパスに対する要求をコントローラーが処理することを示す `Route` 属性を追加します。 ルート内の `[controller]` トークンはコントローラーの名前に置き換えられます (`Controller` サフィックスは省略されます)。これは特にグローバル ルートの場合に役立ちます。 詳細については、[ルーティング](../fundamentals/routing.md)に関するページを参照してください。

コントローラーを使用するには `ITodoRepository` が機能する必要があります。コントローラーのコンストラクターを介してこの種類のインスタンスを要求します。 実行時に、[依存関係の挿入](../fundamentals/dependency-injection.md)用のフレームワークのサポートを使用して、このインスタンスが提供されます。

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippetDI":::

この API は、データ ソースに対して CRUD (Create (作成)、Read (読み取り)、Update (更新)、Delete (削除)) 操作を実行する 4 つの異なる HTTP 動詞をサポートしています。 この中で最も単純な操作は読み取りです。HTTP GET 要求に対応します。

### <a name="reading-items"></a>項目の読み取り

項目一覧の要求は、`List` メソッドに対する GET 要求で行われます。 `List` メソッドの `[HttpGet]` 属性は、このアクションが GET 要求のみを処理する必要があることを示します。 このアクションのルートは、コントローラーで指定されたルートです。 ルートの一部としてアクション名を使用する必要はありません。 各アクションに一意で明確なルートを持たせる必要があります。 ルーティング属性をコントローラー レベルとメソッド レベルの両方で適用して、特定のルートを構築することができます。

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippet":::

メソッドは、 `List` JSON としてシリアル化された 200 OK 応答コードとすべての Todo 項目を返します。

次のように、[Postman](https://www.getpostman.com/docs/) などのさまざまなツールを使用して、新しい API メソッドをテストできます。

![todoitems の GET 要求を表示する Postman コンソールと、返された 3 つの項目の JSON を示す応答の本文](native-mobile-backend/_static/postman-get.png)

### <a name="creating-items"></a>項目の作成

慣例により、新しいデータ項目の作成は HTTP POST 動詞にマッピングされます。 `Create` メソッドには `[HttpPost]` 属性が適用され、このメソッドは `TodoItem` インスタンスを受け入れます。 `item` 引数は POST の本文で渡されるため、このパラメーターは `[FromBody]` 属性を指定します。

メソッド内では、データ ストア内で項目が有効であることと事前に存在することが確認され、問題がなければ、リポジトリを使用して追加されます。 `ModelState.IsValid` の確認で[モデルの検証](../mvc/models/validation.md)が実行されます。この確認は、ユーザー入力を受け入れるすべての API メソッドで実行する必要があります。

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippetCreate":::

このサンプルでは、 `enum` モバイルクライアントに渡されるエラーコードを含むを使用します。

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippetErrorCode":::

Postman を使用して新しい項目の追加をテストします。このときに、要求の本文で JSON 形式の新しいオブジェクトを提供する POST 動詞を選択します。 また、`application/json` の `Content-Type` を指定する要求ヘッダーも追加する必要があります。

![POST と応答が表示される Postman コンソール](native-mobile-backend/_static/postman-post.png)

このメソッドは、新しく作成された項目を応答で返します。

### <a name="updating-items"></a>項目の更新

レコードの変更は、HTTP PUT 要求を使用して行われます。 `Edit` メソッドは、この変更以外は `Create` とほとんど同じです。 レコードが見つからない場合、`Edit` アクションから `NotFound` (404) 応答が返されます。

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippetEdit":::

Postman を使用してテストするには、動詞を PUT に変更します。 要求の本文で更新されたオブジェクト データを指定します。

![PUT と応答が表示される Postman コンソール](native-mobile-backend/_static/postman-put.png)

このメソッドが成功した場合は、既存の API との整合性を保つために、`NoContent` (204) 応答が返されます。

### <a name="deleting-items"></a>アイテムを削除する

レコードを削除するには、サービスに対して DELETE 要求を実行し、削除する項目の ID を渡します。 更新と同様に、存在しない項目に対する要求は `NotFound` 応答を受け取ります。 それ以外の成功した要求は `NoContent` (204) 応答を受け取ります。

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippetDelete":::

削除機能をテストする場合、要求の本文には何も指定する必要がありません。

![DELETE と応答が表示される Postman コンソール](native-mobile-backend/_static/postman-delete.png)

## <a name="prevent-over-posting"></a>過剰な投稿を防止する

現在、サンプル アプリでは `TodoItem` オブジェクト全体が公開されています。 通常、運用環境のアプリでは、モデルのサブセットを使用して入力されるデータおよび返されるデータが制限されています。 その背景には複数の理由があり、セキュリティは主なものです。 モデルのサブセットは、通常、データ転送オブジェクト (DTO)、入力モデル、またはビュー モデルと呼ばれます。 この記事では **DTO** を使用しています。

DTO は次の目的で使用できます。

* 過剰な投稿を防止する。
* クライアントが表示しないことになっているプロパティを非表示にする。
* ペイロード サイズを減らすために、いくつかのプロパティを省略する。
* 入れ子になったオブジェクトを含むオブジェクト グラフをフラット化する。 フラット化されたオブジェクト グラフは、クライアントにとってより便利になる可能性があります。

DTO アプローチの例については、「[過剰投稿の防止](xref:tutorials/first-web-api#prevent-over-posting)」を参照してください。

## <a name="common-web-api-conventions"></a>一般的な Web API 規約

アプリのバックエンド サービスを開発する場合は、横断的な懸案事項を処理するための一連の規約やポリシーが必要になります。 たとえば、前述のサービスでは、見つからなかった特定のレコードに対する要求は、`BadRequest` 応答ではなく `NotFound` 応答を受け取りました。 同様に、モデルにバインドされた種類で渡されたこのサービスに対するコマンドは、常に `ModelState.IsValid` を確認し、無効なモデルの種類の場合に `BadRequest` を返していました。

API の共通ポリシーを特定した場合、通常はそのポリシーを[フィルター](../mvc/controllers/filters.md)にカプセル化できます。 詳細については、[ASP.NET Core MVC アプリケーションで一般的な API ポリシーをカプセル化する方法](/archive/msdn-magazine/2016/august/asp-net-core-real-world-asp-net-core-mvc-filters)に関するページを参照してください。

## <a name="additional-resources"></a>その他のリソース

- [Xamarin. Forms: Web サービス認証](/xamarin/xamarin-forms/data-cloud/authentication/)
- [Xamarin. フォーム: RESTful Web サービスを使用する](/xamarin/xamarin-forms/data-cloud/web-services/rest)
- [Microsoft Learn: Xamarin アプリで REST web サービスを使用する](/learn/modules/consume-rest-services/)
- [Microsoft Learn:ASP.NET Core で Web API を作成する](/learn/modules/build-web-api-aspnet-core/)
