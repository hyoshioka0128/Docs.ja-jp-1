---
title: ASP.NET Core で JavaScript サービスを使用してシングル ページ アプリケーションを作成する
author: rick-anderson
description: ASP.NET Core と JavaScript サービスを使用してシングル ページ アプリケーション (SPA) を作成する利点について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: H1Hack27Feb2017, devx-track-js
ms.date: 09/06/2019
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
uid: client-side/spa-services
ms.openlocfilehash: c0f18c1466bb140c82c9dfb41d773a6c68181168
ms.sourcegitcommit: 0abfe496fed8e9470037c8128efa8a50069ccd52
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/07/2021
ms.locfileid: "106563803"
---
# <a name="use-javascript-services-to-create-single-page-applications-in-aspnet-core"></a>ASP.NET Core で JavaScript サービスを使用してシングル ページ アプリケーションを作成する

作成者: [Scott Addie](https://github.com/scottaddie)、[Fiyaz Hasan](https://fiyazhasan.me/)

シングル ページ アプリケーション (SPA) は、本質的に高度なユーザー エクスペリエンスを提供できることから、広く使われているタイプの Web アプリケーションです。 クライアント側の SPA フレームワークまたはライブラリ ([Angular](https://angular.io/) や [React](https://facebook.github.io/react/) など) と、ASP.NET Core などのサーバー側のフレームワークの統合は、困難な場合があります。 JavaScript サービスは、統合プロセスの手間を減らすために開発されました。 これにより、さまざまなクライアントやサーバーのテクノロジ スタック間でシームレスな操作を行うことができます。

::: moniker range=">= aspnetcore-3.0"

> [!WARNING]
> この記事で説明されている機能は、ASP.NET Core 3.0 時点で互換性のために残されてます。 より単純な SPA フレームワーク統合メカニズムが [Microsoft.AspNetCore.SpaServices.Extensions](https://www.nuget.org/packages/Microsoft.AspNetCore.SpaServices.Extensions) NuGet パッケージで利用できます。 詳細については、[Microsoft.AspNetCore.SpaServices と Microsoft.AspNetCore.NodeServices の廃止に関するお知らせのページ](https://github.com/dotnet/AspNetCore/issues/12890)を参照してください。

::: moniker-end

## <a name="what-is-javascript-services"></a>JavaScript サービスとは

JavaScript サービスは、ASP.NET Core のクライアント側テクノロジのコレクションです。 その目的は、ASP.NET Core を開発者の SPA 構築に有用なサーバー側プラットフォームとして位置付けることにあります。

JavaScript サービスは、次の 2 つの異なる NuGet パッケージで構成されています。

* [Microsoft.AspNetCore.NodeServices](https://www.nuget.org/packages/Microsoft.AspNetCore.NodeServices/) (NodeServices)
* [Microsoft.AspNetCore.SpaServices](https://www.nuget.org/packages/Microsoft.AspNetCore.SpaServices/) (SpaServices)

これらのパッケージは、次のシナリオで役立ちます。

* サーバーで JavaScript を実行する
* SPA フレームワークまたは SPA ライブラリを使用する
* Webpack を使用してクライアント側の資産を構築する

この記事では、SpaServices パッケージを使用することに重点を置いて説明します。

## <a name="what-is-spaservices"></a>SpaServices とは

SpaServices は、ASP.NET Core を開発者の SPA 構築に有用なサーバー側プラットフォームとして位置付けるために作成されました。 SpaServices は、ASP.NET Core で SPA を開発する際に必ず必要になるのものではありません。また、SpaServices を使っても、開発者が特定のクライアント フレームワークにロックされることはありません。

SpaServices は、次のような便利なインフラストラクチャを提供します。

* [サーバー側の事前レンダリング](#server-side-prerendering)
* [Webpack Dev ミドルウェア](#webpack-dev-middleware)
* [ホット モジュール置換](#hot-module-replacement)
* [ルーティング ヘルパー](#routing-helpers)

これらのインフラストラクチャ コンポーネントを集めると、開発ワークフローとランタイム エクスペリエンスの両方を強化することができます。 このコンポーネントは、個別に採用できます。

## <a name="prerequisites-for-using-spaservices"></a>SpaServices を使用するための前提条件

SpaServices を使用するには、以下をインストールします。

* [Node.js](https://nodejs.org/) (バージョン 6 以降) と npm

  * これらのコンポーネントがインストールされていて検出可能であることを確認するには、コマンド ラインから次のコマンドを実行します。

    ```console
    node -v && npm -v
    ```

  * Azure の Web サイトにデプロイする場合、操作は必要ありません &mdash; サーバー環境に Node.js がインストールされて使用できるようになります。

* [!INCLUDE [](~/includes/net-core-sdk-download-link.md)]

  * Visual Studio 2017 を使用している Windows では、 **.NET Core クロスプラットフォーム開発** ワークロードを選択すると SDK がインストールされます。

* [Microsoft.AspNetCore.SpaServices](https://www.nuget.org/packages/Microsoft.AspNetCore.SpaServices/) NuGet パッケージ

## <a name="server-side-prerendering"></a>サーバー側の事前レンダリング

ユニバーサル (アイソモーフィックとも呼ばれます) アプリケーションは、サーバーとクライアントの両方で実行できる JavaScript アプリケーションです。 Angular、React などの一般的なフレームワークは、このアプリケーション開発スタイル用のユニバーサル プラットフォームを提供します。 まず、サーバーで Node.js を使用してフレームワーク コンポーネントをレンダリングし、続く処理の実行をクライアントに委任します。

SpaServices が提供する ASP.NET Core [タグ ヘルパー](xref:mvc/views/tag-helpers/intro)は、サーバーで JavaScript 関数を呼び出すことで、サーバー側の事前レンダリングの実装が簡単にします。

### <a name="server-side-prerendering-prerequisites"></a>サーバー側の事前レンダリングの前提条件

[aspnet-prerendering](https://www.npmjs.com/package/aspnet-prerendering) npm パッケージをインストールします。

```console
npm i -S aspnet-prerendering
```

### <a name="server-side-prerendering-configuration"></a>サーバー側の事前レンダリングの構成

プロジェクトの *_ViewImports.cshtml* ファイルに名前空間を登録すると、タグ ヘルパーを検出できるようになります。

[!code-cshtml[](../client-side/spa-services/sample/SpaServicesSampleApp/Views/_ViewImports.cshtml?highlight=3)]

タグ ヘルパーは、Razor ビューで HTML に似た構文を利用することで、低レベルの API と直接通信する複雑な部分を抽象化します。

[!code-cshtml[](../client-side/spa-services/sample/SpaServicesSampleApp/Views/Home/Index.cshtml?range=5)]

### <a name="asp-prerender-module-tag-helper"></a>asp-prerender-module タグ ヘルパー

前のコード例で使用されている `asp-prerender-module` タグ ヘルパーは、サーバーの Node.js で *ClientApp/dist/main-server.js* を実行します。 わかりやすくするために、*main-server.js* ファイルは、[Webpack](https://webpack.github.io/) ビルド プロセスで TypeScript から JavaScript へのトランスパイル タスクの成果物とします。 Webpack では `main-server` のエントリ ポイントの別名が定義されています。また、この別名の依存関係グラフの走査は、*ClientApp/boot-server.ts* ファイルから開始します。

[!code-javascript[](../client-side/spa-services/sample/SpaServicesSampleApp/webpack.config.js?range=53)]

次の Angular の例では、*ClientApp/boot-server.ts* ファイルが `aspnet-prerendering` npm パッケージの `createServerRenderer` 関数と `RenderResult` 型を使用して Node.js によるサーバー レンダリングを構成しています。 サーバー側のレンダリング用の HTML マークアップは、resolve 関数呼び出しに渡されます。これは、厳密に型指定された JavaScript の `Promise` オブジェクトでラップされています。 `Promise` オブジェクトは、DOM のプレースホルダー要素に注入する HTML マークアップを非同期にページに渡すという重要な役割を果たしています。

[!code-typescript[](../client-side/spa-services/sample/SpaServicesSampleApp/ClientApp/boot-server.ts?range=6,10-34,79-)]

### <a name="asp-prerender-data-tag-helper"></a>asp-prerender-data タグ ヘルパー

`asp-prerender-data` タグ ヘルパーと `asp-prerender-module` タグ ヘルパーと組み合わせると、Razor ビューからサーバー側 JavaScript にコンテキスト情報を渡すことができます。 たとえば、次のマークアップは、ユーザー データを `main-server` モジュールに渡します。

[!code-cshtml[](../client-side/spa-services/sample/SpaServicesSampleApp/Views/Home/Index.cshtml?range=9-12)]

渡された `UserName` 引数は、組み込みの JSON シリアライザーによってシリアル化され、`params.data` オブジェクトに格納されます。 次の Angular の例では、このデータを使用して、`h1` の要素内に独自の挨拶文を作成します。

[!code-typescript[](../client-side/spa-services/sample/SpaServicesSampleApp/ClientApp/boot-server.ts?range=6,10-21,38-52,79-)]

タグ ヘルパーで渡されるプロパティ名は、**パスカルケース** 表記で表されます。 同じプロパティ名が **キャメルケース** で表される JavaScript とは対照的です。 既定の JSON シリアル化構成が、この違いに対応します。

上のコード例を発展させると、サーバーからビューにデータを渡すことができます。そのためには、`resolve` 関数に渡す `globals` プロパティを設定します。

[!code-typescript[](../client-side/spa-services/sample/SpaServicesSampleApp/ClientApp/boot-server.ts?range=6,10-21,57-77,79-)]

`globals` オブジェクト内で定義されている `postList` 配列は、ブラウザーのグローバル `window` オブジェクトにアタッチされます。 この変数はグローバル スコープに含められ、それによって作業の重複がなくなります。具体的に言えば、同じデータをサーバーで一度、クライアントで再度読み込むことがなくなるためです。

![window オブジェクトにアタッチされたグローバル postList 変数](spa-services/_static/global_variable.png)

## <a name="webpack-dev-middleware"></a>Webpack Dev ミドルウェア

[Webpack Dev ミドルウェア](https://webpack.js.org/guides/development/#using-webpack-dev-middleware) は、Webpack が必要に応じてリソースを構築することで、合理化された開発ワークフローを実現します。 ページがブラウザーに再読み込みされると、ミドルウェアはクライアント側のリソースを自動的にコンパイルして提供します。 別の方法として、サードパーティの依存関係またはカスタム コードが変更されたときに、プロジェクトの npm ビルド スクリプトを介して Webpack を手動で呼び出す方法もあります。 次の例では、*package.json* ファイル内の npm ビルド スクリプトを示しています。

```json
"build": "npm run build:vendor && npm run build:custom",
```

### <a name="webpack-dev-middleware-prerequisites"></a>Webpack Dev ミドルウェアの前提条件

[aspnet-webpack](https://www.npmjs.com/package/aspnet-webpack) npm パッケージをインストールします。

```console
npm i -D aspnet-webpack
```

### <a name="webpack-dev-middleware-configuration"></a>Webpack Dev ミドルウェアの構成

Webpack Dev ミドルウェアは、*Startup.cs* ファイルの `Configure` メソッドにある次のコードによって、HTTP 要求パイプラインに登録されます。

[!code-csharp[](../client-side/spa-services/sample/SpaServicesSampleApp/Startup.cs?name=snippet_WebpackMiddlewareRegistration&highlight=4)]

`UseStaticFiles` 拡張メソッドを使用して[静的ファイル ホスティングを登録](xref:fundamentals/static-files)する前に、`UseWebpackDevMiddleware` 拡張メソッドを呼び出す必要があります。 セキュリティ上の理由から、アプリが開発モードで実行されている場合にのみ、ミドルウェアを登録してください。

*webpack.config.js* ファイルの `output.publicPath` プロパティは、`dist` フォルダーの変更を監視するようにミドルウェアに指示します。

[!code-javascript[](../client-side/spa-services/sample/SpaServicesSampleApp/webpack.config.js?range=6,13-16)]

## <a name="hot-module-replacement"></a>ホット モジュール置換

Webpack の[ホット モジュール置換](https://webpack.js.org/concepts/hot-module-replacement/) (HMR) 機能は、[Webpack Dev ミドルウェア](#webpack-dev-middleware)が進化したものと考えることができます。 HMR は同じ利点をすべて実現しますが、変更をコンパイルした後にページ コンテンツを自動的に更新するので、開発ワークフローがさらに効率化されます。 これをブラウザーの更新と混同しないでください。ブラウザーの更新は、現在のメモリ内の状態と SPA のデバッグ セッションに影響します。 Webpack Dev ミドルウェア サービスとブラウザーの間には、ライブ リンクがあります。これは、変更がブラウザーにプッシュされることを意味します。

### <a name="hot-module-replacement-prerequisites"></a>ホット モジュール置換の前提条件

[webpack-hot-middleware](https://www.npmjs.com/package/webpack-hot-middleware) npm パッケージをインストールします。

```console
npm i -D webpack-hot-middleware
```

### <a name="hot-module-replacement-configuration"></a>ホット モジュール置換の構成

HMR コンポーネントは、`Configure` メソッドで MVC の HTTP 要求パイプラインに登録する必要があります。

```csharp
app.UseWebpackDevMiddleware(new WebpackDevMiddlewareOptions {
    HotModuleReplacement = true
});
```

[Webpack Dev ミドルウェア](#webpack-dev-middleware)の場合と同様に、`UseWebpackDevMiddleware` 拡張メソッドを `UseStaticFiles` 拡張メソッドの前に呼び出す必要があります。 セキュリティ上の理由から、アプリが開発モードで実行されている場合にのみ、ミドルウェアを登録してください。

*webpack.config.js* ファイルでは、`plugins` 配列を定義する必要があります。これは空のままでもかまいません。

[!code-javascript[](../client-side/spa-services/sample/SpaServicesSampleApp/webpack.config.js?range=6,25)]

ブラウザーでアプリを読み込むと、開発者ツールのコンソールのタブに HMR がアクティブ化されたことの確認が表示されます。

![ホット モジュール置換の接続メッセージ](spa-services/_static/hmr_connected.png)

## <a name="routing-helpers"></a>ルーティング ヘルパー

ほとんどの ASP.NET Core ベースの SPA では、サーバー側のルーティングに加えて、クライアント側のルーティングが必要になります。 SPA および MVC のルーティング システムは、干渉せずに個別に動作させることができます。 しかし、1 つのエッジケースでは、HTTP 応答 404 を認識するという課題があります。

拡張子のない `/some/page` というルートを使用するシナリオを考えてみましょう。 要求がサーバー側ルートとはパターン一致せず、クライアント側のルートと一致すると仮定します。 ここで、`/images/user-512.png` の受信要求について考えてみます。通常は、サーバー上のイメージ ファイルを検索する必要があります。 要求されたリソース パスがどのサーバー側ルートまたは静的ファイルとも一致しない場合、クライアント側アプリケーションがこれを処理することはほとんどありません &mdash; 通常は、HTTP 状態コード 404 が返されます。

### <a name="routing-helpers-prerequisites"></a>ルーティング ヘルパーの前提条件

クライアント側ルーティング npm パッケージをインストールします。 Angular を使用する場合の例を示します。

```console
npm i -S @angular/router
```

### <a name="routing-helpers-configuration"></a>ルーティング ヘルパーの構成

`Configure` メソッドで `MapSpaFallbackRoute` という名前の拡張メソッドを使用します。

[!code-csharp[](../client-side/spa-services/sample/SpaServicesSampleApp/Startup.cs?name=snippet_MvcRoutingTable&highlight=7-9)]

ルートは、構成された順序で評価されます。 そのため、上のコード例の `default` ルートは、最初にパターン マッチングに使用されます。

## <a name="create-a-new-project"></a>新しいプロジェクトを作成する

JavaScript サービスには、事前構成済みのアプリケーション テンプレートが用意されています。 これらのテンプレートでは、Angular、React、Redux などのさまざまなフレームワークやライブラリと共に SpaServices を使用します。

これらのテンプレートは、次のコマンドを実行して .NET Core CLI 経由でインストールできます。

```dotnetcli
dotnet new --install Microsoft.AspNetCore.SpaTemplates::*
```

使用可能な SPA テンプレートの一覧が表示されます。

| テンプレート                                 | 短い形式の名前 | 言語 | Tags        |
| ------------------------------------------| :--------: | :------: | :---------: |
| MVC ASP.NET Core with Angular             | angular    | [C#]     | Web/MVC/SPA |
| MVC ASP.NET Core with React.js            | react      | [C#]     | Web/MVC/SPA |
| MVC ASP.NET Core with React.js and Redux  | reactredux | [C#]     | Web/MVC/SPA |

SPA テンプレートの 1 つを使用して新しいプロジェクトを作成するには、[dotnet new](/dotnet/core/tools/dotnet-new) コマンドでテンプレートの **短い形式の名前** を指定します。 次のコマンドは、サーバー側用に構成された ASP.NET Core MVC を使用して、Angular アプリケーションを作成します。

```dotnetcli
dotnet new angular
```

### <a name="set-the-runtime-configuration-mode"></a>ランタイム構成モードの設定

2 つのプライマリ ランタイム構成モードがあります。

* **開発**:
  * デバッグを容易にするソース マップが含まれています。
  * パフォーマンス向上のためにクライアント側のコードを最適化することはしません。
* **実稼働**:
  * ソース マップを除外します。
  * バンドルと縮小を使用して、クライアント側のコードを最適化します。

ASP.NET Core では、`ASPNETCORE_ENVIRONMENT` という名前の環境変数を使用して構成モードを格納します。 詳細については、「[環境を設定する](xref:fundamentals/environments#set-the-environment)」を参照してください。

### <a name="run-with-net-core-cli"></a>.NET Core CLI で実行する

プロジェクト ルートで次のコマンドを実行して、必要な NuGet パッケージと npm パッケージを復元します。

```dotnetcli
dotnet restore && npm i
```

アプリケーションをビルドして実行します。

```dotnetcli
dotnet run
```

アプリケーションは、[ランタイム構成モード](#set-the-runtime-configuration-mode)に従って、localhost で開始されます。 ブラウザーで `http://localhost:5000` に移動すると、ランディング ページが表示されます。

### <a name="run-with-visual-studio-2017"></a>Visual Studio 2017 で実行する

[dotnet new](/dotnet/core/tools/dotnet-new) コマンドで生成された *.csproj* ファイルを開きます。 必要な NuGet および npm パッケージは、プロジェクトを開いたときに自動的に復元されます。 この復元プロセスには数分かかる場合があり、これが完了するとアプリケーションを実行する準備ができています。 緑色の実行ボタンをクリックするか `Ctrl + F5` キーを押すと、ブラウザーが開いてアプリケーションのランディング ページが表示されます。 アプリケーションは、[ランタイム構成モード](#set-the-runtime-configuration-mode)に従って、localhost で実行されます。

## <a name="test-the-app"></a>アプリのテスト

SpaServices テンプレートは、[Karma](https://karma-runner.github.io/1.0/index.html) と [Jasmine](https://jasmine.github.io/) を使用してクライアント側のテストを実行するように事前構成されています。 Jasmine は JavaScript 用の一般的な単体テスト フレームワークであるのに対し、Karma はそれらのテストのテスト ランナーです。 Karma は [Webpack Dev ミドルウェア](#webpack-dev-middleware)と連携するように構成されています。これにより、開発者は、変更が行われるたびにテストを停止して実行する必要がなくなります。 テスト ケースまたはテスト ケース自体に対してコードが実行されているかどうかにかかわらず、テストは自動的に実行されます。

例として、Angular アプリケーションを使用します。*counter.component.spec.ts* ファイルの `CounterComponent` に対し、2 つの Jasmine テスト ケースが既に提供されています。

[!code-typescript[](../client-side/spa-services/sample/SpaServicesSampleApp/ClientApp/app/components/counter/counter.component.spec.ts?range=15-28)]

*ClientApp* ディレクトリで、コマンド プロンプトを開きます。 次のコマンドを実行します。

```console
npm test
```

このスクリプトは、Karma テスト ランナーを起動します。これにより、*karma.conf.js* ファイルで定義されている設定が読み取られます。 その他の設定として、*karma.conf.js* の `files` 配列で実行するテスト ファイルを指定できます。

[!code-javascript[](../client-side/spa-services/sample/SpaServicesSampleApp/ClientApp/test/karma.conf.js?range=4-5,8-11)]

## <a name="publish-the-app"></a>アプリの発行

Azure への発行の詳細については、[こちらの GitHub の問題](https://github.com/dotnet/AspNetCore.Docs/issues/12474)のページを参照してください。

生成されたクライアント側アセットと発行された ASP.NET Core 成果物をすぐにデプロイ可能なパッケージにまとめると、煩雑になることがあります。 ありがたいことに、SpaServices は、`RunWebpack` という名前のカスタム MSBuild ターゲットを使用して、発行プロセス全体を調整します。

[!code-xml[](../client-side/spa-services/sample/SpaServicesSampleApp/SpaServicesSampleApp.csproj?range=31-45)]

この MSBuild ターゲットには、次の役割があります。

1. npm パッケージを復元する
1. サードパーティ製クライアント側アセットの実稼働レベルのビルドを作成する
1. カスタムのクライアント側アセットの実稼働レベルのビルドを作成する
1. Webpack が生成したアセットを発行フォルダーにコピーする

次を実行すると、MSBuild ターゲットが呼び出されます。

```dotnetcli
dotnet publish -c Release
```

## <a name="additional-resources"></a>その他の技術情報

* [Angular ドキュメント](https://angular.io/docs)
