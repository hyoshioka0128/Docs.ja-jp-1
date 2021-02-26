---
title: パート 3、ASP.NET Core MVC アプリへのビューの追加
author: rick-anderson
description: ASP.NET Core MVC のチュートリアル シリーズのパート 3。
ms.author: riande
ms.date: 01/28/2021
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
uid: tutorials/first-mvc-app/adding-view
ms.custom: contperf-fy21q3
ms.openlocfilehash: 1542e44fcc6d0ae22fb1a759ea3a3ed1d866cbc7
ms.sourcegitcommit: 19a004ff2be73876a9ef0f1ac44d0331849ad159
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/07/2021
ms.locfileid: "99804570"
---
# <a name="part-3-add-a-view-to-an-aspnet-core-mvc-app"></a>パート 3、ASP.NET Core MVC アプリへのビューの追加

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

このセクションでは、[Razor](xref:mvc/views/razor) ビュー ファイルを使用するよう、`HelloWorldController` クラスを変更します。 これにより、クライアントへの HTML 応答を生成するプロセスが完全にカプセル化されます。

ビュー テンプレートは Razor を使用して作成されます。 Razor ベースのビュー テンプレートは次のとおりです。

* ファイル拡張子は *.cshtml* です。
* C# を使用して HTML 出力を作成する洗練された方法が提供されます。

現在、`Index` メソッドは、コントローラー クラスにメッセージを含め文字列を返します。 `HelloWorldController` クラスでは、`Index` メソッドを次のコードで置き換えます。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_4)]

上記のコードでは、次のことが行われます。

* コントローラーの <xref:Microsoft.AspNetCore.Mvc.Controller.View*> メソッドを呼び出します。
* ビュー テンプレートを使用し HTML 応答を生成します。

コントローラー メソッドは次のとおりです。

* "*アクション メソッド*" と呼ばれます。  たとえば、前のコードの `Index` アクション メソッドです。
* 通常、`string` のような型ではなく <xref:Microsoft.AspNetCore.Mvc.IActionResult> または <xref:Microsoft.AspNetCore.Mvc.ActionResult> から派生したクラスを返します。

## <a name="add-a-view"></a>ビューを追加する

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

*Views* フォルダーを右クリックし、 **[追加]、[新しいフォルダー]** の順に選択し、フォルダーに *HelloWorld* という名前を付けます。

*Views/HelloWorld* フォルダーを右クリックし、 **[追加]、[新しい項目]** の順に選択します。

**[新しい項目の追加 - MvcMovie]** ダイアログで次を行います。

* 右上の検索ボックスに「*view*」と入力します。
* **[Razor ビュー]** を選択します
* **[名前]** ボックスの値、*Index.cshtml* を維持します。
* **[追加]** を選択します。

![[新しい項目の追加] ダイアログ](adding-view/_static/add_view50.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

次のように `HelloWorldController` に `Index` ビューを追加します。

* *Views/HelloWorld* という名前の新しいフォルダーを追加します。
* *Views/HelloWorld* フォルダー名 *Index.cshtml* に新しいファイルを追加します。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

*Views* フォルダーを Control キーを押しながらクリックし、 **[追加]、[新しいフォルダー]** の順に選択し、フォルダーに *HelloWorld* という名前を付けます。

*Views/HelloWorld* フォルダーを Control キーを押しながらクリックし、 **[追加]、[新しいファイル]** の順に選択します。

**[新しいファイル]** ダイアログで次を実行します。

* 左側のウィンドウで、 **[ASP.NET Core]** を選択します。
* 中央のウィンドウで **[Razor View - Empty]** \(Razor ビュー - 空\) を選択します。
* **[名前]** ボックスに「*Index*」と入力します。
* **[新規]** を選択します。

  ![[新しい項目の追加] ダイアログ](adding-view/_static/add_view_macVSM8.9.png)

---

*Views/HelloWorld/Index.cshtml* Razor ビュー ファイルを次の内容に置き換えます。

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/HelloWorld/Index1.cshtml?highlight=7)]

次のように `https://localhost:{PORT}/HelloWorld` に移動します。

* `HelloWorldController` の `Index` メソッドにより、ビュー テンプレート ファイルを使用して、ブラウザーに応答をレンダリングするようメソッドを指定する、ステートメント `return View();` が実行されました。
* ビュー テンプレートのファイル名が指定されていないため、MVC で既定のビュー ファイルが使われました。 ビュー ファイル名を指定しない場合は、既定のビューが返されます。 この例では、既定のビューの名前は、アクション メソッド `Index` と同じ名前が付けられています。 ビュー テンプレート */Views/HelloWorld/Index.cshtml* が使用されています。
* 次のイメージは、ビューにハード コーディングされた "Hello from our View Template!" という文字列を示しています。

  ![ブラウザー ウィンドウ](~/tutorials/first-mvc-app/adding-view/_static/hell_template.png)

## <a name="change-views-and-layout-pages"></a>ビューとレイアウト ページを変更する

メニューのリンク、 **[MvcMovie]** 、 **[ホーム]** 、 **[プライバシー]** を選択します。 各ページには同じメニューのレイアウトが表示されます。 メニューのレイアウトは、*Views/Shared/_Layout.cshtml* ファイルに実装されています。

*Views/Shared/_Layout.cshtml* ファイルを開きます。

[レイアウト](xref:mvc/views/layout) テンプレートでは、次のことが可能です。

* 1 か所でサイトの HTML コンテナー レイアウトを指定できます。
* サイト内の複数のページに HTML コンテナー レイアウトを適用できます。

`@RenderBody()` という行を見つけます。 `RenderBody` は、作成したビュー固有のページがすべて表示されるプレースホルダーで、レイアウト ページに *ラップ* されます。 たとえば、 **[プライバシー]** リンクを選択した場合、`RenderBody` メソッド内で **Views/Home/Privacy.cshtml** ビューがレンダリングされます。

## <a name="change-the-title-footer-and-menu-link-in-the-layout-file"></a>レイアウト ファイルでのタイトル、フッター、およびメニュー リンクの変更

*Views/Shared/_Layout.cshtml* ファイルの内容を次のマークアップに置き換えます。 変更が強調表示されています。
::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Views/Shared/_Layout.cshtml?highlight=6,14,40)]

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie5/Views/Shared/_Layout.cshtml?highlight=6,14,40)]

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

上記のマークアップでは、次の変更が加えられています。

* `MvcMovie` から `Movie App` が 3 回発生します。
* アンカー要素 `<a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">MvcMovie</a>` が `<a class="navbar-brand" asp-controller="Movies" asp-action="Index">Movie App</a>` になりました。

上記のマークアップでは、このアプリでは [Area](xref:mvc/controllers/areas) が使用されていないため、`asp-area=""` [アンカー タグ ヘルパー属性](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper)と属性値が省略されています。

**注:** まだ `Movies` コントローラーは実装されていません。 この時点では、`Movie App` リンクは機能していません。

変更を保存し、 **[プライバシー]** リンクを選択します。 ブラウザー タブのタイトルが、**Privacy Policy - Mvc Movie** ではなく、**Privacy Policy - Movie App** になっていることに注目してください。

![[プライバシー] タブ](~/tutorials/first-mvc-app/adding-view/_static/privacy50.png)

**[ホーム]** リンクを選択します。

タイトルとアンカー テキストに **Movie App** と表示されていることに着目してください。 レイアウト テンプレートが一度変更され、この新しいリンク テキストと新しいタイトルがサイト上のすべてのページに反映されました。

*Views/_ViewStart.cshtml* ファイルを確認します。

```cshtml
@{
    Layout = "_Layout";
}
```

*Views/_ViewStart.cshtml* ファイルは *Views/Shared/_Layout.cshtml* ファイルに取り込まれ、各ビューに適用されます。 `Layout` プロパティを使用すれば、別のレイアウト ビューを設定することも、`null` に設定してレイアウト ファイルが使用されないようにすることもできます。

*Views/HelloWorld/Index.cshtml* ビュー ファイルを開きます。

タイトルと `<h2>` 要素を、以下の強調表示どおりに変更します。

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Views/HelloWorld/Index2.cshtml?highlight=2,5)]

タイトルと `<h2>` 要素は若干違うので、コードのどの部分によって表示が変更されるのかを確認できます。

上のコードの `ViewData["Title"] = "Movie List";` では、`Title` ディクショナリの `ViewData` プロパティを "Movie List" に設定します。 `Title` プロパティは、次のように、レイアウト ページの `<title>` HTML 要素で使用されます。

```cshtml
<title>@ViewData["Title"] - Movie App</title>
```

変更内容を保存して、`https://localhost:{PORT}/HelloWorld` に移動します。

次が変更されたことを確認します。

* ブラウザー タイトル。
* 1 番目の見出し。
* 2 番目の見出し。

ブラウザーに変更がない場合は、キャッシュされたコンテンツが表示されている場合があります。 ブラウザーで Ctrl + F5 キーを押して、サーバーからの応答が強制的に読み込まれるようにします。 ブラウザーのタイトルは、*Index.cshtml* ビュー テンプレートで設定した `ViewData["Title"]` で作成されます。レイアウト ファイルには "- Movie App" が追加されます。

*Index.cshtml* ビュー テンプレート内のコンテンツは、*Views/Shared/_Layout.cshtml* ビュー テンプレートにマージされます。 1 つの HTML 応答がブラウザーに送信されます。 レイアウト テンプレートを使用すれば、アプリのすべてのページに適用される変更を簡単に行うことができます。 詳細については、「[Layout](xref:mvc/views/layout)」(レイアウト) を参照してください。

![ムービー リスト ビュー](~/tutorials/first-mvc-app/adding-view/_static/hell50.png)

ここでは、"データ" のごく一部 (この場合は "Hello from our View Template!" という メッセージ) をハード コーディングしました。 MVC アプリケーションには "V" (ビュー) があり、"C" (コントローラー) もありますが、"M" (モデル) はまだありません。

## <a name="passing-data-from-the-controller-to-the-view"></a>コントローラーからビューへのデータの受け渡し

コントローラー アクションは、受信 URL 要求への応答として呼び出されます。 コントローラー クラスでは、受信ブラウザー要求を処理するコードが記述されます。 コントローラーはデータ ソースからデータを取得し、ブラウザーに返す応答の種類を決定します。 ビュー テンプレートを使用すれば、コントローラーからブラウザーへの HTML 応答を生成して書式を設定できます。

コントローラーは、ビュー テンプレートで応答をレンダリングするために必要なデータを提供します。

ビュー テンプレートでは、次を **行いません**。

* ビジネス ロジックの実行
* データベースとの直接のやりとり

ビュー テンプレートは、コントローラーから提供されるデータのみを処理する必要があります。 この "関心の分離" を維持すれば、コードを次のように保つことができます。

* クリーン。
* テスト可能。
* 保守しやすい。

現時点では、`HelloWorldController` クラスの `Welcome` メソッドは `name` と `ID` パラメーターを受け取ってから、ブラウザーに直接値を出力します。

コントローラーにこの応答を文字列としてレンダリングさせるのではなく、代わりにビュー テンプレートを使用するようにコントローラーを変更します。 このビュー テンプレートは、応答を動的に生成します。つまり、応答の生成には、コントローラーからビューに適切なデータが渡される必要があります。 そのためには、ビュー テンプレートが必要とする動的データ (パラメーター) を、コントローラーに `ViewData` ディレクトリに配置させます。 これにより、ビュー テンプレートが動的データにアクセスできるようになります。

*HelloWorldController.cs* 内で、`Welcome` メソッドを変更して `Message` および `NumTimes` 値を `ViewData` ディクショナリに追加します。

`ViewData` ディクショナリは、すべての型を使用できる動的なオブジェクトです。 `ViewData` オブジェクトには、何かが追加されるまで、プロパティは定義されていません。 [MVC のモデル バインド システム](xref:mvc/models/model-binding)は、`name` と `numTimes` の名前付きパラメーターをクエリ文字列からメソッドのパラメーターに自動的にマップします。 完全な `HelloWorldController` は次のとおりです。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_5&highlight=13-19)]

`ViewData` ディクショナリ オブジェクトには、ビューに渡されるデータが含まれています。

*Views/HelloWorld/Welcome.cshtml* という名前の [ようこそ] ビュー テンプレートを作成します。

"Hello" `NumTimes` を表示する *Welcome.cshtml* ビュー テンプレートでループを作成します。 *Views/HelloWorld/Welcome.cshtml* の内容を次のコードに置き換えます。

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Views/HelloWorld/Welcome.cshtml)]

変更内容を保存し、次の URL を参照します。

`https://localhost:{PORT}/HelloWorld/Welcome?name=Rick&numtimes=4`

データは URL から取得され、[MVC モデル バインダー](xref:mvc/models/model-binding)を使用してコントローラーに渡されます。 コントローラーはデータを `ViewData` ディクショナリにパッケージ化し、そのオブジェクトをビューに渡します。 その後、ビューでブラウザーに HTML としてデータがレンダリングされます。

![[ようこそ] ラベルと、Hello Rick という語句が 4 つ示された [プライバシー] ビュー](~/tutorials/first-mvc-app/adding-view/_static/rick2_50.png)

上のサンプルでは、`ViewData` ディクショナリを使用して、コントローラーからビューにデータを渡しました。 チュートリアルの後半では、ビュー モデルを使用して、コントローラーからビューにデータを渡します。 データを渡すには、`ViewData` ディクショナリを使用する方法より、ビュー モデルを使用する方法が推奨されます。

次のチュートリアルでは、ムービーのデータベースを作成します。

> [!div class="step-by-step"]
> [前: コントローラーの追加](adding-controller.md)
> [次: モデルの追加](adding-model.md)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

このセクションでは、クライアントに対する HTML 応答を生成するプロセスを完全にカプセル化するために、[Razor](xref:mvc/views/razor) ビュー ファイルを使用して、`HelloWorldController` クラスを変更します。

ビュー テンプレート ファイルは Razor を使用して作成します。 Razor ベースのビュー テンプレートには *.cshtml* ファイル拡張子が含まれています。 C# を使用して HTML 出力を作成する洗練された方法が提供されます。

現在、`Index` メソッドは、コントローラー クラスでハード コーディングされるメッセージを含む文字列を返します。 `HelloWorldController` クラスでは、`Index` メソッドを次のコードで置き換えます。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_4)]

上のコードでは、コントローラーの <xref:Microsoft.AspNetCore.Mvc.Controller.View*> メソッドを呼び出します。 ビュー テンプレートを使用して、HTML 応答を生成します。 上記の `Index` メソッドなどのコントローラー メソッド (*アクション メソッド* ともいう) は、一般に、`string` などの型ではなく、<xref:Microsoft.AspNetCore.Mvc.IActionResult> (または <xref:Microsoft.AspNetCore.Mvc.ActionResult> から派生したクラス) を返します。

## <a name="add-a-view"></a>ビューを追加する

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* *Views* フォルダーを右クリックし、 **[追加]、[新しいフォルダー]** の順に選択し、フォルダーに *HelloWorld* という名前を付けます。

* *Views/HelloWorld* フォルダーを右クリックし、 **[追加]、[新しい項目]** の順に選択します。

* **[新しい項目の追加 - MvcMovie]** ダイアログ

  * 右上の検索ボックスに「*view*」と入力します。

  * **[Razor ビュー]** を選択します

  * **[名前]** ボックスの値、*Index.cshtml* を維持します。

  * **[追加]** を選択します。

![[新しい項目の追加] ダイアログ](adding-view/_static/add_view.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

`HelloWorldController` の `Index` ビューを追加します。

* *Views/HelloWorld* という名前の新しいフォルダーを追加します。
* *Views/HelloWorld* フォルダー名 *Index.cshtml* に新しいファイルを追加します。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* *Views* フォルダーを右クリックし、 **[追加]、[新しいフォルダー]** の順に選択し、フォルダーに *HelloWorld* という名前を付けます。
* *Views/HelloWorld* フォルダーを右クリックし、 **[追加]、[新しいファイル]** の順に選択します。
* **[新しいファイル]** ダイアログで次を実行します。

  * 左側のウィンドウで **[Web]** を選択します。
  * 中央のウィンドウで **[空の HTML ファイル]** を選択します。
  * **[名前]** ボックスに「*Index.cshtml*」と入力します。
  * **[新規]** を選択します。

![[新しい項目の追加] ダイアログ](adding-view/_static/add_view_mac.png)

---

*Views/HelloWorld/Index.cshtml* Razor ビュー ファイルを次の内容に置き換えます。

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/HelloWorld/Index1.cshtml?highlight=7)]

`https://localhost:{PORT}/HelloWorld` に移動します。 `HelloWorldController` の `Index` メソッドでは多くのことは行いませんでした。つまり、ステートメント `return View();` を実行し、メソッドでビュー テンプレート ファイルを使用して、ブラウザーへの応答をレンダリングするよう指定しただけです。 ビュー テンプレートのファイル名が指定されていないため、MVC では既定のビュー ファイルが使われました。 既定のビュー ファイルはメソッド (`Index`) と同じ名前なので、 */Views/HelloWorld/Index.cshtml* が使われます。 次のイメージは、ビューにハード コーディングされた "Hello from our View Template!" という文字列を示しています。

![ブラウザー ウィンドウ](~/tutorials/first-mvc-app/adding-view/_static/hell_template.png)

## <a name="change-views-and-layout-pages"></a>ビューとレイアウト ページを変更する

メニューのリンク ( **[MvcMovie]** 、 **[ホーム]** 、 **[プライバシー]** ) を選択します。 各ページには同じメニューのレイアウトが表示されます。 メニューのレイアウトは、*Views/Shared/_Layout.cshtml* ファイルに実装されています。 *Views/Shared/_Layout.cshtml* ファイルを開きます。

[[レイアウト]](xref:mvc/views/layout) テンプレートでは、1 か所でサイトの HTML コンテナー レイアウトを指定し、それをサイト内の複数のページに適用できます。 `@RenderBody()` という行を見つけます。 `RenderBody` は、作成したビュー固有のページがすべて表示されるプレースホルダーで、レイアウト ページに *ラップ* されます。 たとえば、 **[プライバシー]** リンクを選択した場合、`RenderBody` メソッド内で **Views/Home/Privacy.cshtml** ビューがレンダリングされます。

## <a name="change-the-title-footer-and-menu-link-in-the-layout-file"></a>レイアウト ファイルでのタイトル、フッター、およびメニュー リンクの変更

* タイトル要素とフッター要素で、`MvcMovie` を `Movie App` に変更します。
* アンカー要素 `<a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">MvcMovie</a>` を `<a class="navbar-brand" asp-controller="Movies" asp-action="Index">Movie App</a>` に変更します。

次のマークアップには、強調表示された変更点が示されています。

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Shared/_Layout.cshtml?highlight=6,24,51)]

上記のマークアップでは、このアプリで[領域](xref:mvc/controllers/areas)が使用されていないため、`asp-area` [アンカー タグ ヘルパー属性](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper)は省略されました。

<!-- Routing has changed in 2.2, it's going to the last route.
>[!WARNING]
> We haven't implemented the `Movies` controller yet, so if you click the `Movie App` link, you get a 404 (Not found) error.
-->

**注**:`Movies` コントローラーは実装されていません。 この時点で、`Movie App`リンクは機能しません。

ご自分の変更を保存し、**プライバシー** リンクを選択します。 ブラウザー タブのタイトルが、**Privacy Policy - Mvc Movie** ではなく、**Privacy Policy - Movie App** になっていることに注目してください。

![[プライバシー] タブ](~/tutorials/first-mvc-app/adding-view/_static/privacy50.png)

**[ホーム]** リンクをタップし、タイトルとアンカー テキストにも **[Movie App]** と表示されていることを確認してください。 レイアウト テンプレートで一度変更しただけで、サイト上のすべてのページに新しいリンク テキストと新しいタイトルが反映できました。

*Views/_ViewStart.cshtml* ファイルを確認します。

```cshtml
@{
    Layout = "_Layout";
}
```

*Views/_ViewStart.cshtml* ファイルは *Views/Shared/_Layout.cshtml* ファイルに取り込まれ、各ビューに適用されます。 `Layout` プロパティを使用すれば、別のレイアウト ビューを設定することも、`null` に設定してレイアウト ファイルが使用されないようにすることもできます。

*Views/HelloWorld/Index.cshtml* ビュー ファイルのタイトルと `<h2>` 要素を次のように変更します。

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Views/HelloWorld/Index2.cshtml?highlight=2,5)]

タイトルと `<h2>` 要素は若干異なります。これにより、コードのどの部分によって表示が変更されるのかを確認できます。

上のコードの `ViewData["Title"] = "Movie List";` では、`Title` ディクショナリの `ViewData` プロパティを "Movie List" に設定します。 `Title` プロパティは、次のように、レイアウト ページの `<title>` HTML 要素で使用されます。

```cshtml
<title>@ViewData["Title"] - Movie App</title>
```

変更内容を保存して、`https://localhost:{PORT}/HelloWorld` に移動します。 ブラウザーのタイトル、プライマリ見出し、およびセカンダリ見出しが変更されていることに注意してください (ブラウザーに変更内容が表示されない場合は、キャッシュされたコンテンツを表示している可能性があります。 ブラウザーで Ctrl + F5 キーを押して、サーバーからの応答が強制的に読み込まれるようにしてください)。ブラウザーのタイトルは、*Index.cshtml* ビュー テンプレートで設定した `ViewData["Title"]` で作成されます。レイアウト ファイルには "- Movie App" が追加されます。

*Index.cshtml* ビュー テンプレートのコンテンツがどのように *Views/Shared/_Layout.cshtml* ビュー テンプレートにマージされ、1 つの HTML 応答がブラウザーに送信されたかにも注目してください。 レイアウト テンプレートを使用すれば、アプリケーションのすべてのページに適用される変更をとても簡単に行うことができます。 詳細については、「[Layout](xref:mvc/views/layout)」 (レイアウト) を参照してください。

![ムービー リスト ビュー](~/tutorials/first-mvc-app/adding-view/_static/hell3.png)

ここでは、"データ" のごく一部 (この場合は "Hello from our View Template!" というメッセージ) を ハード コーディングしました。 MVC アプリケーションには "V" (ビュー) があり、"C" (コントローラー) もありますが、"M" (モデル) はまだありません。

## <a name="passing-data-from-the-controller-to-the-view"></a>コントローラーからビューへのデータの受け渡し

コントローラー アクションは、受信 URL 要求への応答として呼び出されます。 コントローラー クラスでは、受信ブラウザー要求を処理するコードが記述されます。 コントローラーはデータ ソースからデータを取得し、ブラウザーに返す応答の種類を決定します。 ビュー テンプレートを使用すれば、コントローラーからブラウザーへの HTML 応答を生成して書式を設定できます。

コントローラーは、ビュー テンプレートで応答をレンダリングするために必要なデータを提供します。 ベスト プラクティス: ビュー テンプレートでは、ビジネス ロジックを実行したり、データベースと直接やりとりしたり **しない** でください。 ビュー テンプレートでは、コントローラーによって提供されるデータのみを処理するようにしてください。 この "懸念事項の分離" を維持すれば、コードをクリーンでテスト可能な保守しやすい状態に保つことが楽になります。

現時点では、`HelloWorldController` クラスの `Welcome` メソッドは `name` と `ID` パラメーターを受け取ってから、ブラウザーに直接値を出力します。 コントローラーにこの応答を文字列としてレンダリングさせるのではなく、代わりにビュー テンプレートを使用するようにコントローラーを変更します。 このビュー テンプレートでは動的応答が生成されます。これは、応答を生成するために、コントローラーからビューに適量のデータを渡す必要があることを意味します。 そのためには、コントローラーで動的データ (パラメーター) を設定します。これは、ビュー テンプレートでアクセスできるようにするために `ViewData` ディレクトリに必要なデータです。

*HelloWorldController.cs* 内で、`Welcome` メソッドを変更して `Message` および `NumTimes` 値を `ViewData` ディクショナリに追加します。 `ViewData` ディクショナリは動的オブジェクトです。つまり、任意の型を使用することができます。`ViewData` オブジェクトでは、その内部に何かを設定するまでプロパティは定義されません。 [MVC のモデル バインド システム](xref:mvc/models/model-binding)は、名前付きパラメーター (`name` と `numTimes`) を、アドレス バーのクエリ文字列からメソッドのパラメーターに自動的にマップします。 完全な *HelloWorldController.cs* ファイルは次のようになります。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_5)]

`ViewData` ディクショナリ オブジェクトには、ビューに渡されるデータが含まれています。

*Views/HelloWorld/Welcome.cshtml* という名前の [ようこそ] ビュー テンプレートを作成します。

"Hello" `NumTimes` を表示する *Welcome.cshtml* ビュー テンプレートでループを作成します。 *Views/HelloWorld/Welcome.cshtml* の内容を次のコードに置き換えます。

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Views/HelloWorld/Welcome.cshtml)]

変更内容を保存し、次の URL を参照します。

`https://localhost:{PORT}/HelloWorld/Welcome?name=Rick&numtimes=4`

データは URL から取得され、[MVC モデル バインダー](xref:mvc/models/model-binding)を使用してコントローラーに渡されます。 コントローラーはデータを `ViewData` ディクショナリにパッケージ化し、そのオブジェクトをビューに渡します。 その後、ビューでブラウザーに HTML としてデータがレンダリングされます。

![[ようこそ] ラベルと、Hello Rick という語句が 4 つ示された [プライバシー] ビュー](~/tutorials/first-mvc-app/adding-view/_static/rick2.png)

上のサンプルでは、`ViewData` ディクショナリを使用して、コントローラーからビューにデータを渡しました。 チュートリアルの後半では、ビュー モデルを使用して、コントローラーからビューにデータを渡します。 一般には、`ViewData` ディクショナリを使用する方法より、ビュー モデルを使用してデータを渡す方法が推奨されます。 詳細については、[ViewBag、ViewData、または TempData を使用するタイミング](https://www.rachelappel.com/when-to-use-viewbag-viewdata-or-tempdata-in-asp-net-mvc-3-applications/)に関するページをご覧ください。

次のチュートリアルでは、ムービーのデータベースを作成します。

> [!div class="step-by-step"]
> [前へ](adding-controller.md)
> [次へ](adding-model.md)

::: moniker-end
