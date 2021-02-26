---
title: 'パート 2: ASP.NET Core MVC アプリにコントローラーを追加する'
author: rick-anderson
description: ASP.NET Core MVC のチュートリアル シリーズのパート 2。
ms.author: riande
ms.date: 01/23/2021
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
uid: tutorials/first-mvc-app/adding-controller
ms.custom: contperf-fy21q3
ms.openlocfilehash: 47bb9b96bd5565a3a67f3cbdf9a4b6bc1f987447
ms.sourcegitcommit: ef8d8c79993a6608bf597ad036edcf30b231843f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2021
ms.locfileid: "99975262"
---
# <a name="part-2-add-a-controller-to-an-aspnet-core-mvc-app"></a>パート 2: ASP.NET Core MVC アプリにコントローラーを追加する

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

モデル ビュー コントローラー (MVC) アーキテクチャ パターンでは、アプリが 3 つの主要なコンポーネントに分けられます。**モデル**、**ビュー**、**コントローラー**。 MVC パターンでは、よりテスト可能で、従来のモノリシック アプリより更新しやすいアプリを作成できます。

MVC ベースのアプリには以下が含まれます。

* **モデル**: アプリのデータを表すクラス。 モデル クラスでは検証ロジックを使用して、そのデータにビジネス ルールを適用します。 通常、モデル オブジェクトはモデルの状態を取得して、データベースに格納します。 このチュートリアルでは、`Movie` モデルはデータベースからムービーデータを取得し、それをビューに提供するか、更新します。 更新されたデータはデータベースに書き込まれます。
* **ビュー**: ビューは、アプリのユーザー インターフェイス (UI) を表示するコンポーネントです。 一般に、この UI ではモデル データが表示されます。
* **コントローラー**: 次を行うクラスです。
  * ブラウザーの要求を処理する。
  * モデル データを取得する。
  * 応答を返すビュー テンプレートを呼び出す。

MVC アプリでは、ビューに情報のみが表示されます。 コントローラーによってユーザーの入力と操作が処理され応答が返されます。 たとえば、コントローラーによって URL セグメントとクエリ文字列の値が処理され、それらの値がモデルに渡されます。 モデルはこれらの値を使用して、データベースを照会する場合があります。 例:

* `Https://localhost:5001/Home/Privacy`: `Home` コントローラーと `Privacy` アクションを指定します。
* `Https://localhost:5001/Movies/Edit/5`: `Movies` コントローラーと `Edit` アクションを使用して ID = 5 のムービーを編集するための要求です。これについては、このチュートリアルで後ほど詳しく説明します。

ルート データについては、このチュートリアルで後ほど説明します。

アプリは、MVC アーキテクチャ パターンによって、モデル、ビュー、コントローラーという 3 つの主要なコンポーネントのグループに分けられます。 このパターンは、UI ロジックがビューに属している、という点で、関心の分離を実現するのに役立ちます。 入力ロジックはコントローラーに属しています。 ビジネス ロジックはモデルに属しています。 このように分離することで、他のコードに影響を与えることなく、一度に実装の 1 つの側面の作業に専念できるため、アプリを構築するときの複雑さが管理しやすくなります。 たとえば、ビジネス ロジック コードに依存することなく、ビュー コードに専念できます。

このチュートリアル シリーズでは、ムービー アプリを構築しながら、これらの概念について紹介しデモを行います。 MVC プロジェクトには、*Controllers* と *Views* の各フォルダーが含まれています。

## <a name="add-a-controller"></a>コントローラーの追加

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

**ソリューション エクスプローラー** で、 **[Controllers] を右クリックし、[追加]、[コントローラー]** の順に選択します。

![ソリューション エクスプローラーで、[Controllers] を右クリックし、[追加]、[コントローラー] の順に選択する](~/tutorials/first-mvc-app/adding-controller/_static/add_controllercopyVS19v16.9.png)

**[スキャフォールディングを追加]** ダイアログ ボックスで、 **[MVC コント ローラー - 空]** を選択します。

![MVC コント ローラーを追加し、名前を付けます](~/tutorials/first-mvc-app/adding-controller/_static/acCopyVS19v16.9.png)

**[新しい項目の追加 - MvcMovie]** ダイアログで、「**HelloWorldController.cs**」と入力し、 **[追加]** を選択します。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

**[エクスプローラー]** アイコンで、 **[コントローラー] を右クリックして [新しいファイル] を選択し**、新しいファイルに「*HelloWorldController.cs*」という名前を付けます。

![コンテキスト メニュー](~/tutorials/first-mvc-app-xplat/adding-controller/_static/new_fileVSC1.51.png)

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

**ソリューション エクスプローラー** で、 **[Controllers] を右クリックし、[追加]、[新しいファイル]** の順に選択します。

![コンテキスト メニュー](~/tutorials/first-mvc-app-mac/adding-controller/_static/add_controller.png)

**[ASP.NET Core]** と **[コントローラー クラス]** を選択します。

コントローラーに「**HelloWorldController**」という名前を付けます。

![MVC コント ローラーを追加し、名前を付けます](~/tutorials/first-mvc-app-mac/adding-controller/_static/ac.png)

---

*Controllers/HelloWorldController.cs* の内容を次のように置き換えます。

  [!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_1)]

コントローラーのすべての `public` メソッドが、HTTP エンドポイントとして呼び出されます。 上のサンプルでは、両方のメソッドが文字列を返します。 各メソッドの前のコメントに注意してください。

HTTP エンドポイント:

* Web アプリケーションのターゲット設定可能な URL です (`https://localhost:5001/HelloWorld` など)。
* 組み合わせ:
  * 使用されるプロトコル: `HTTPS`。
  * TCP ポートなど、Web サーバーのネットワークの場所: `localhost:5001`。
  * ターゲット URI: `HelloWorld`

1 番目のコメントは、これが [HTTP GET](https://developer.mozilla.org/docs/Web/HTTP/Methods/GET) メソッドであり、ベース URL に `/HelloWorld/` を追加することによって呼び出されることを示しています。

2 番目のコメントでは、URL に `/HelloWorld/Welcome/` を追加することによって呼び出される [HTTP GET](https://developer.mozilla.org/docs/Web/HTTP/Methods) メソッドが示されています。 このチュートリアルではこの後、スキャフォールディング エンジンを使用して、データを更新する `HTTP POST` メソッドを生成します。

デバッガーなしでアプリを実行します。

アドレス バーのパスに "HelloWorld" を追加します。 `Index` メソッドが文字列を返します。

!["This is my default action" というアプリの応答が表示されているブラウザー ウィンドウ](~/tutorials/first-mvc-app/adding-controller/_static/hell1.png)

着信 URL に応じて、コントローラー クラスおよびそれらに含まれるアクション メソッドが MVC によって呼び出されます。 MVC によって使用される既定の [URL ルーティング ロジック](xref:mvc/controllers/routing)では、次のような形式を使用して、呼び出すコードが決定されます。

`/[Controller]/[ActionName]/[Parameters]`

ルーティングの形式は、*Startup.cs* ファイル内の`Configure` メソッドで設定されます。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Startup.cs?name=snippet_1&highlight=5)]

URL セグメントを指定しないでアプリを参照すると、既定では、"Home" コントローラーと "Index" メソッドが、上の強調表示されている template 行で指定されます。  上記の URL セグメントでは、次のようになります。

* 1 番目の URL セグメントでは、実行するコントローラー クラスが決定されます。 そのため、`localhost:5001/HelloWorld` は、**HelloWorld** コントローラー クラスにマップされます。
* URL セグメントの 2 番目の部分では、クラスのアクション メソッドが決定されます。 したがって、`localhost:5001/HelloWorld/Index` を使用すると、`HelloWorldController` クラスの `Index` メソッドが実行されます。 参照する必要があるのは `localhost:5001/HelloWorld` だけであり、`Index` メソッドは既定で呼び出されることに注意してください。 `Index` はメソッド名が明示的に指定されていない場合にコントローラーで呼び出される既定のメソッドです。
* URL セグメントの 3 番目の部分 (`id`) はルート データ用です。 ルート データについては、このチュートリアルで後ほど説明します。

`https://localhost:{PORT}/HelloWorld/Welcome` を参照します。 `{PORT}` は実際のポート番号に置き換えます。

`Welcome` メソッドが実行され、文字列 `This is the Welcome action method...` が返されます。 この URL では、コントローラーは `HelloWorld` で、`Welcome` がアクション メソッドです。 URL の `[Parameters]` の部分はまだ使っていません。

!["This is the Welcome action method" というアプリケーションの応答が表示されているブラウザー ウィンドウ](~/tutorials/first-mvc-app/adding-controller/_static/welcome.png)

URL からコントローラーにいくつかのパラメーター情報を渡すように、コードを変更します。 たとえば、`/HelloWorld/Welcome?name=Rick&numtimes=4` のようにします。

次のコードで示すように、2 つのパラメーターを含むように `Welcome` メソッドを変更します。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_2)]

上記のコードでは次の操作が行われます。

* C# のオプション パラメーター機能を使って、`numTimes` パラメーターに値が渡されない場合の既定値が 1 であることを示します。
* `HtmlEncoder.Default.Encode` を使用して、JavaScript などによる悪意のある入力からアプリを保護します。
* `$"Hello {name}, NumTimes is: {numTimes}"` 内で[補間文字列](/dotnet/articles/csharp/language-reference/keywords/interpolated-strings)を使います。

アプリを実行して、`https://localhost:{PORT}/HelloWorld/Welcome?name=Rick&numtimes=4` を参照します。 `{PORT}` は実際のポート番号に置き換えます。

URL の `name` と `numtimes` に違う値を指定してみてください。 MVC の[モデル バインド](xref:mvc/models/model-binding) システムによって、名前付きパラメーターがアドレス バーのクエリ文字列からメソッドのパラメーターに自動的にマップされます。 詳しくは、「[モデル バインド](xref:mvc/models/model-binding)」をご覧ください。

!["Hello Rick, NumTimes is\: 4" というアプリケーションの応答が表示されているブラウザー ウィンドウ](~/tutorials/first-mvc-app/adding-controller/_static/rick4.png)

上図では、次のようになっています。

* URL セグメント `Parameters` は使用されません。
* `name` および `numTimes` パラメーターは、[クエリ文字列 ](https://wikipedia.org/wiki/Query_string) で渡されます。
* 上の URL の `?` (疑問符) は区切り記号であり、後にクエリ文字列が続きます。
* `&` 文字を使ってフィールドと値のペアを区切ります。

`Welcome` メソッドを次のコードで置き換えます。

  [!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_3)]

アプリを実行し、次の URL を入力します: `https://localhost:{PORT}/HelloWorld/Welcome/3?name=Rick`

上のコード URL では、次のようになります。

* 3 番目の URL セグメントがルート パラメーター `id` と一致しました。 
* `Welcome` メソッドには、`MapControllerRoute` メソッドの URL テンプレートと一致したパラメーター `id` が含まれます。
* 末尾の `?` (`id?`) は、`id` パラメーターが省略可能であることを示します。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie5/Startup.cs?name=snippet_route&highlight=5)]

前の例の場合:

* 3 番目の URL セグメントがルート パラメーター `id` と一致しました。
* `Welcome` メソッドには、`MapControllerRoute` メソッドの URL テンプレートと一致したパラメーター `id` が含まれます。
* 末尾の `?` (`id?`) は、`id` パラメーターが省略可能であることを示します。

> [!div class="step-by-step"]
> [前へ: 作業の開始](start-mvc.md)
> [次へ: ビューを追加する](adding-view.md)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

モデル ビュー コントローラー (MVC) アーキテクチャ パターンでは、アプリが 3 つの主要なコンポーネントに分けられます。**モデル**、**ビュー**、**コントローラー**。 MVC パターンでは、よりテスト可能で、従来のモノリシック アプリより更新しやすいアプリを作成できます。 MVC ベースのアプリには以下が含まれます。

* **モデル**: アプリのデータを表すクラス。 モデル クラスでは検証ロジックを使用して、そのデータにビジネス ルールを適用します。 通常、モデル オブジェクトはモデルの状態を取得して、データベースに格納します。 このチュートリアルでは、`Movie` モデルはデータベースからムービーデータを取得し、それをビューに提供するか、更新します。 更新されたデータはデータベースに書き込まれます。

* **ビュー**: ビューは、アプリのユーザー インターフェイス (UI) を表示するコンポーネントです。 一般に、この UI ではモデル データが表示されます。

* **コントローラー**: ブラウザー要求を処理するクラスです。 モデル データを取得し、応答を返すビュー テンプレートを呼び出します。 MVC アプリでは、ビューに情報のみが表示され、コントローラーがユーザーの入力と操作を処理して応答します。 たとえば、コントローラーはルート データとクエリ文字列の値を処理し、これらの値をモデルに渡します。 モデルはこれらの値を使用して、データベースを照会する場合があります。 たとえば、`https://localhost:5001/Home/About` には、`Home` (コントローラー) と `About` (ホーム コントローラーに対して呼び出すアクション メソッド) のルート データがあります。 `https://localhost:5001/Movies/Edit/5` は、ムービー コントローラーを使用して ID=5 のムービーを編集する要求です。 ルート データについては、このチュートリアルで後ほど説明します。

MVC パターンは、これらの要素間の疎結合を提供しながら、アプリのさまざまな側面 (入力ロジック、ビジネス ロジック、および UI ロジック) を分離するアプリを作成するのに役立ちます。 このパターンは、アプリで各種類のロジックを配置する必要がある場所を指定します。 UI ロジックはビューに属しています。 入力ロジックはコントローラーに属しています。 ビジネス ロジックはモデルに属しています。 このように分離することで、他のコードに影響を与えることなく、実装の 1 つの側面に専念できるため、アプリを構築するときの複雑さが管理しやすくなります。 たとえば、ビジネス ロジック コードに依存することなく、ビュー コードに専念できます。

このチュートリアルで示すこれらの概念を使用して、ムービー アプリを構築する方法を示します。 MVC プロジェクトには、*Controllers* と *Views* の各フォルダーが含まれています。

## <a name="add-a-controller"></a>コントローラーの追加

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* **ソリューション エクスプローラー** で、 **[Controllers] を右クリックし、[追加]、[新しいファイル]** の順に選択します。

  ![コンテキスト メニュー](~/tutorials/first-mvc-app/adding-controller/_static/add_controller.png)

* **[スキャフォールディングを追加]** ダイアログ ボックスで、 **[MVC コント ローラー - 空]** を選択します。

  ![MVC コント ローラーを追加し、名前を付けます](~/tutorials/first-mvc-app/adding-controller/_static/ac.png)

* **[Add Empty MVC Controller]\(空の MVC コント ローラーの追加\)** ダイアログで、「**HelloWorldController**」と入力して、 **[追加]** を選択します。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

**[エクスプローラー]** アイコンで、 **[コントローラー] を右クリックして [新しいファイル] を選択し**、新しいファイルに「*HelloWorldController.cs*」という名前を付けます。

  ![コンテキスト メニュー](~/tutorials/first-mvc-app-xplat/adding-controller/_static/new_file.png)

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

**ソリューション エクスプローラー** で、 **[Controllers] を右クリックし、[追加]、[新しいファイル]** の順に選択します。

![コンテキスト メニュー](~/tutorials/first-mvc-app-mac/adding-controller/_static/add_controller.png)

**[ASP.NET Core]** と **[MVC コントローラー クラス]** を選択します。

コントローラーに「**HelloWorldController**」という名前を付けます。

![MVC コント ローラーを追加し、名前を付けます](~/tutorials/first-mvc-app-mac/adding-controller/_static/ac.png)

---

*Controllers/HelloWorldController.cs* の内容を次のように置き換えます。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_1)]

コントローラーのすべての `public` メソッドが、HTTP エンドポイントとして呼び出されます。 上のサンプルでは、両方のメソッドが文字列を返します。 各メソッドの前のコメントに注意してください。

HTTP エンドポイントは、Web アプリケーション内のターゲット設定可能な URL (`https://localhost:5001/HelloWorld` など) であり、使われているプロトコル (`HTTPS`)、Web サーバーの (TCP ポートを含む) ネットワーク上の場所 (`localhost:5001`)、ターゲットの URI (`HelloWorld`) を組み合わせたものです。

1 番目のコメントは、これが [HTTP GET](https://www.w3schools.com/tags/ref_httpmethods.asp) メソッドであり、ベース URL に `/HelloWorld/` を追加することによって呼び出されることを示しています。 2 番目のコメントでは、URL に `/HelloWorld/Welcome/` を追加することによって呼び出される [HTTP GET](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) メソッドが示されています。 このチュートリアルではこの後、スキャフォールディング エンジンを使って、データを更新する `HTTP POST` メソッドを生成します。

非デバッグ モードでアプリを実行し、アドレス バーのパスに "HelloWorld" を追加します。 `Index` メソッドが文字列を返します。

!["This is my default action" というアプリケーションの応答が表示されているブラウザー ウィンドウ](~/tutorials/first-mvc-app/adding-controller/_static/hell1.png)

MVC は、着信 URL に応じてコントローラー クラス (およびそれらに含まれるアクション メソッド) を呼び出します。 MVC によって使われる既定の [URL ルーティング ロジック](xref:mvc/controllers/routing)では、次のような形式を使って呼び出すコードが決定されます。

`/[Controller]/[ActionName]/[Parameters]`

ルーティングの形式は、*Startup.cs* ファイル内の`Configure` メソッドで設定されます。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Startup.cs?name=snippet_1&highlight=5)]

<!-- 
Add link to explain lambda.
Remove link for simplified tutorial.
-->

URL セグメントを指定しないでアプリを参照すると、既定では、"Home" コントローラーと "Index" メソッドが、上の強調表示されている template 行で指定されます。

1 番目の URL セグメントでは、実行するコントローラー クラスが決定されます。 したがって、`localhost:{PORT}/HelloWorld` は `HelloWorldController` クラスにマップします。 URL セグメントの 2 番目の部分では、クラスのアクション メソッドが決定されます。 したがって、`localhost:{PORT}/HelloWorld/Index` では `HelloWorldController` クラスの `Index` メソッドが実行されます。 参照する必要があるのは `localhost:{PORT}/HelloWorld` だけであり、`Index` メソッドは既定で呼び出されることに注意してください。 これは、`Index` はメソッド名が明示的に指定されていない場合にコントローラーで呼び出される既定のメソッドであるためです。 URL セグメントの 3 番目の部分 (`id`) はルート データ用です。 ルート データについては、このチュートリアルで後ほど説明します。

`https://localhost:{PORT}/HelloWorld/Welcome` を参照します。 `Welcome` メソッドが実行され、文字列 `This is the Welcome action method...` が返されます。 この URL では、コントローラーは `HelloWorld` で、`Welcome` がアクション メソッドです。 URL の `[Parameters]` の部分はまだ使っていません。

!["This is the Welcome action method" というアプリケーションの応答が表示されているブラウザー ウィンドウ](~/tutorials/first-mvc-app/adding-controller/_static/welcome.png)

URL からコントローラーにいくつかのパラメーター情報を渡すように、コードを変更します。 たとえば、`/HelloWorld/Welcome?name=Rick&numtimes=4` のようにします。 次のコードで示すように、2 つのパラメーターを含むように `Welcome` メソッドを変更します。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_2)]

上記のコードでは次の操作が行われます。

* C# のオプション パラメーター機能を使って、`numTimes` パラメーターに値が渡されない場合の既定値が 1 であることを示します。 <!-- remove for simplified -->
* `HtmlEncoder.Default.Encode` を使って、悪意のある入力 (つまり JavaScript) からアプリを保護します。
* `$"Hello {name}, NumTimes is: {numTimes}"` 内で[補間文字列](/dotnet/articles/csharp/language-reference/keywords/interpolated-strings)を使います。 <!-- remove for simplified -->

アプリを実行して次を参照します。

   `https://localhost:{PORT}/HelloWorld/Welcome?name=Rick&numtimes=4`

(`{PORT}` は実際のポート番号に置き換えます。)URL の `name` と `numtimes` に違う値を指定してみてください。 MVC の[モデル バインド](xref:mvc/models/model-binding) システムは、名前付きパラメーターを、アドレス バーのクエリ文字列からメソッドのパラメーターに自動的にマップします。 詳しくは、「[モデル バインド](xref:mvc/models/model-binding)」をご覧ください。

!["Hello Rick, NumTimes is\: 4" というアプリケーションの応答が表示されているブラウザー ウィンドウ](~/tutorials/first-mvc-app/adding-controller/_static/rick4.png)

上の図では、URL セグメント (`Parameters`) は使われておらず、`name` および `numTimes` パラメーターは[クエリ文字列](https://wikipedia.org/wiki/Query_string)で渡されています。 上の URL の `?` (疑問符) は区切り記号であり、後にクエリ文字列が続きます。 `&` 文字を使ってフィールドと値のペアを区切ります。

`Welcome` メソッドを次のコードで置き換えます。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_3)]

アプリを実行し、次の URL を入力します: `https://localhost:{PORT}/HelloWorld/Welcome/3?name=Rick`

今度は、3 番目の URL セグメントがルート パラメーター `id` と一致しました。 `Welcome` メソッドには、`MapRoute` メソッドの URL テンプレートと一致したパラメーター `id` が含まれます。 末尾の `?` (`id?`) は、`id` パラメーターが省略可能であることを示します。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Startup.cs?name=snippet_1&highlight=5)]

これらの例では、コントローラーによって MVC の "VC" 部分が実行されています。つまり、ビューとコントローラーが動作します。 コントローラーは HTML を直接返しています。 一般に、コントローラーが HTML を直接返すのは、コーディングと保守が非常に面倒になるので、望ましくありません。 代わりに、通常は、別の Razor ビュー テンプレート ファイルを使って、HTML 応答を生成できるようにします。 これは次のチュートリアルで行います。

> [!div class="step-by-step"]
> [前へ](start-mvc.md)
> [次へ](adding-view.md)

::: moniker-end
