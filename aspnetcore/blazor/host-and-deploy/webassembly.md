---
title: ASP.NET Core Blazor WebAssembly のホストと展開
author: guardrex
description: ASP.NET Core、Content Delivery Networks (CDN)、ファイル サーバー、GitHub ページを使用して、Blazor アプリをホストしデプロイする方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/12/2021
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
uid: blazor/host-and-deploy/webassembly
ms.openlocfilehash: 04eba2e004e920e9ca799b316781857f0b0b4ca3
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100279775"
---
# <a name="host-and-deploy-aspnet-core-blazor-webassembly"></a>ASP.NET Core Blazor WebAssembly のホストと展開

[Blazor WebAssembly ホスティング モデル](xref:blazor/hosting-models#blazor-webassembly)を使用する場合は以下のようになります。

* Blazor アプリ、その依存関係、.NET ランタイムが並行してブラウザーにダウンロードされます。
* アプリがブラウザー UI スレッド上で直接実行されます。

次の展開戦略がサポートされています。

* Blazor アプリは、ASP.NET Core アプリによって提供されます。 この戦略については、「[ASP.NET Core でのホストされた展開](#hosted-deployment-with-aspnet-core)」セクションで説明します。
* Blazor アプリは、Blazor アプリの提供に .NET が使用されていない静的ホスティング Web サーバーまたはサービス上に配置されます。 この戦略については、「[スタンドアロン展開](#standalone-deployment)」セクションで示されます。これには、Blazor WebAssembly アプリを IIS サブアプリとしてホストする方法についての情報が含まれています。

## <a name="compression"></a>[圧縮]

Blazor WebAssembly アプリが公開されると、公開中に出力が静的に圧縮されてアプリのサイズが縮小され、実行時の圧縮に必要なオーバーヘッドがなくなります。 次の圧縮アルゴリズムが使用されます。

* [Brotli](https://tools.ietf.org/html/rfc7932) (最高レベル)
* [Gzip](https://tools.ietf.org/html/rfc1952)

Blazor は、適切な圧縮ファイルにサービスを提供するため、ホストに依存します。 ASP.NET Core でホストするプロジェクトを使用するとき、ホスト プロジェクトでは、コンテント ネゴシエーションを実行したり、静的に圧縮されたファイルにサービスを提供したりできます。 Blazor WebAssembly スタンドアロン アプリをホストするとき、静的に圧縮されたファイルにサービスが提供されるよう、追加の作業が必要になることがあります。

* IIS の `web.config` の圧縮構成については、[IIS の「Brotli と Gzip の圧縮」](#brotli-and-gzip-compression)セクションを参照してください。 
* GitHub ページなど、静的に圧縮されたファイル コンテント ネゴシエーションをサポートしない静的ホスティング ソリューションでホストするとき、Brotli 圧縮ファイルをフェッチし、デコードするようにアプリを構成することを検討してください。

  * [google/brotli GitHub リポジトリ](https://github.com/google/brotli)から、JavaScript Brotli デコーダーを入手します。 デコーダー ファイルは、`decode.js` という名前で、リポジトリの [`js` フォルダー](https://github.com/google/brotli/tree/master/js)にあります。
  
    > [!NOTE]
    > 以前のバージョンは [google/brotli GitHub リポジトリ](https://github.com/google/brotli)の `decode.js` スクリプト (`decode.min.js`) の縮小バージョンにあります。 独自にスクリプトを縮小するか (例については、[BuildBundlerMinifier のバンドルと縮小](xref:client-side/bundling-and-minification#configure-bundling-and-minification)に関するページを参照してください)、[decode.min.js の TypeError (google/brotli #881)](https://github.com/google/brotli/issues/881) の問題が解決されるまで [npm package](https://www.npmjs.com/package/brotli) を使用します。 このセクションのサンプル コードでは、スクリプトの **未縮小** バージョンを使用しています。

  * そのデコーダーを使用するようにアプリを更新します。 `wwwroot/index.html` の終了タグ `<body>` に含まれるマークアップを次のように変更します。
  
    ```html
    <script src="decode.js"></script>
    <script src="_framework/blazor.webassembly.js" autostart="false"></script>
    <script>
      Blazor.start({
        loadBootResource: function (type, name, defaultUri, integrity) {
          if (type !== 'dotnetjs' && location.hostname !== 'localhost') {
            return (async function () {
              const response = await fetch(defaultUri + '.br', { cache: 'no-cache' });
              if (!response.ok) {
                throw new Error(response.statusText);
              }
              const originalResponseBuffer = await response.arrayBuffer();
              const originalResponseArray = new Int8Array(originalResponseBuffer);
              const decompressedResponseArray = BrotliDecode(originalResponseArray);
              const contentType = type === 
                'dotnetwasm' ? 'application/wasm' : 'application/octet-stream';
              return new Response(decompressedResponseArray, 
                { headers: { 'content-type': contentType } });
            })();
          }
        }
      });
    </script>
    ```
 
圧縮を無効にするには、アプリケーションのプロジェクト ファイルに `BlazorEnableCompression` MSBuild プロパティを追加し、値を `false` に設定します。

```xml
<PropertyGroup>
  <BlazorEnableCompression>false</BlazorEnableCompression>
</PropertyGroup>
```

`BlazorEnableCompression` プロパティは、コマンド シェルで次の構文を使用して [`dotnet publish`](/dotnet/core/tools/dotnet-publish) コマンドに渡すことができます。

```dotnetcli
dotnet publish -p:BlazorEnableCompression=false
```

## <a name="rewrite-urls-for-correct-routing"></a>正しいルーティングのために URL を書き換える

Blazor WebAssembly アプリ内のページ コンポーネントに対するルーティング要求は、Blazor Server (ホストされているアプリ) でのルーティング要求のように単純なものではありません。 次の 2 つのコンポーネントがある Blazor WebAssembly アプリについて考えてみます。

* `Main.razor`:アプリのルートで読み込まれ、`About` コンポーネントへのリンク (`href="About"`) が含まれています。
* `About.razor`: `About` コンポーネント。

アプリの既定のドキュメントがブラウザーのアドレス バー (例: `https://www.contoso.com/`) を使用して要求された場合:

1. ブラウザーにより要求が送信されます。
1. 既定のページ (通常は `index.html`) が返されます。
1. `index.html` によりアプリがブートストラップされます。
1. Blazor のルーターが読み込まれて、Razor `Main` コンポーネントが表示されます。

Main ページでは、`About` コンポーネントへのリンクの選択がクライアント上で動作します。Blazor のルーターにより、インターネット上で `www.contoso.com` に `About` を求めるブラウザーの要求が停止され、レンダリングされた `About` コンポーネント自体が提供されるためです。 " *Blazor WebAssembly アプリ内にある*" 内部エンドポイントへの要求は、すべて同じように動作します。要求によって、サーバーにホストされているインターネット上のリソースに対するブラウザーベースの要求がトリガーされることはありません。 要求は、ルーターによって内部的に処理されます。

ブラウザーのアドレス バーを使用して `www.contoso.com/About` の要求が行われた場合、その要求は失敗します。 アプリのインターネット ホスト上にそのようなリソースは存在しないため、"*404 見つかりません*" という応答が返されます。

ブラウザーではクライアント側ページの要求がインターネットベースのホストに対して行われるため、Web サーバーとホスティング サービスでは、サーバー上に物理的に存在しないリソースに対する `index.html` ページへのすべての要求を、書き換える必要があります。 `index.html` が返されると、アプリの Blazor ルーターがそれを受け取り、正しいリソースで応答します。

IIS サーバーに展開する場合は、アプリで発行される `web.config` ファイルで URL Rewrite Module を使用できます。 詳細については、「[IIS](#iis)」セクションを参照してください。

## <a name="hosted-deployment-with-aspnet-core"></a>ASP.NET Core でのホストされた展開

"*ホストされたデプロイ*" により、Blazor WebAssembly アプリが、Web サーバー上で実行されている [ASP.NET Core アプリ](xref:index)からブラウザーに提供されます。

クライアント Blazor WebAssembly アプリは、サーバー アプリの他の静的な Web アセットと共に、サーバー アプリの `/bin/Release/{TARGET FRAMEWORK}/publish/wwwroot` フォルダーに発行されます。 2 つのアプリが一緒に展開されます。 ASP.NET Core アプリをホストできる Web サーバーが必要です。 ホストされている展開の場合、Visual Studio には **Blazor WebAssembly アプリ** プロジェクト テンプレートが含まれており ([`dotnet new`](/dotnet/core/tools/dotnet-new) コマンドを使用する場合は `blazorwasm` テンプレート)、 **`Hosted`** オプションが選択されています (`dotnet new` コマンドを使用する場合は `-ho|--hosted`)。

ASP.NET Core アプリでのホストと展開の詳細については、「<xref:host-and-deploy/index>」を参照してください。

Azure App Service の展開については、「<xref:tutorials/publish-to-azure-webapp-using-vs>」を参照してください。

## <a name="hosted-deployment-with-multiple-blazor-webassembly-apps"></a>複数の Blazor WebAssembly アプリによるホストされた展開

### <a name="app-configuration"></a>アプリの構成

ホスティングされた Blazor ソリューションでは、複数の Blazor WebAssembly アプリに対応できます。

> [!NOTE]
> このセクションの例では、Visual Studio "*ソリューション*" の使用方法を参照していますが、複数のクライアント アプリがホスティングされた Blazor WebAssembly アプリのシナリオで動作するために、Visual Studio と Visual Studio ソリューションを使用する必要はありません。 Visual Studio を使用していない場合は、Visual Studio 用に作成された `{SOLUTION NAME}.sln` ファイルとその他のファイルは無視してください。

次に例を示します。

* 初期 (最初の) クライアント アプリは、Blazor WebAssembly プロジェクト テンプレートから作成されたソリューションの既定のクライアント プロジェクトです。 最初のクライアント アプリは、ブラウザーで、ポート 5001 または `firstapp.com` のホストで URL `/FirstApp` からアクセスできます。
* 2 つ目のクライアント アプリは、ソリューション `SecondBlazorApp.Client` に追加されます。 2 つ目クライアント アプリは、ブラウザーで、ポート 5002 または `secondapp.com` のホストで URL `/SecondApp` からアクセスできます。

既存のホスティングされた Blazor ソリューションを使用するか、Blazor のホスティングされたプロジェクト テンプレートから新しいソリューションを作成します。

* クライアント アプリのプロジェクト ファイルで、値が `FirstApp` の `<PropertyGroup>` に `<StaticWebAssetBasePath>` プロパティを追加して、プロジェクトの静的アセットの基本パスを設定します。

  ```xml
  <PropertyGroup>
    ...
    <StaticWebAssetBasePath>FirstApp</StaticWebAssetBasePath>
  </PropertyGroup>
  ```

* ソリューションに 2 つ目のクライアント アプリを追加します。

  * `SecondClient` という名前のフォルダーをソリューションのフォルダーに追加します。 プロジェクト テンプレートから作成されたソリューション フォルダーには、`SecondClient` フォルダーを追加した後の次のソリューション ファイルとフォルダーが含まれています。
  
    * `Client` (フォルダー)
    * `SecondClient` (フォルダー)
    * `Server` (フォルダー)
    * `Shared` (フォルダー)
    * `{SOLUTION NAME}.sln` (ファイル)
    
    プレースホルダー `{SOLUTION NAME}` は、ソリューションの名前です。

  * Blazor WebAssembly プロジェクト テンプレートから `SecondClient` フォルダーに `SecondBlazorApp.Client` という名前の Blazor WebAssembly アプリを作成します。

  * `SecondBlazorApp.Client` アプリのプロジェクト ファイル内で、次のようにします。

    * 値が `SecondApp` の `<PropertyGroup>` に `<StaticWebAssetBasePath>` プロパティを追加します。

      ```xml
      <PropertyGroup>
        ...
        <StaticWebAssetBasePath>SecondApp</StaticWebAssetBasePath>
      </PropertyGroup>
      ```

    * `Shared` プロジェクトにプロジェクト参照を追加します。

      ```xml
      <ItemGroup>
        <ProjectReference Include="..\Shared\{SOLUTION NAME}.Shared.csproj" />
      </ItemGroup>
      ```

      プレースホルダー `{SOLUTION NAME}` は、ソリューションの名前です。

* サーバー アプリのプロジェクト ファイルで、追加した `SecondBlazorApp.Client` クライアント アプリに対するプロジェクト参照を作成します。

  ```xml
  <ItemGroup>
    <ProjectReference Include="..\Client\{SOLUTION NAME}.Client.csproj" />
    <ProjectReference Include="..\SecondClient\SecondBlazorApp.Client.csproj" />
    <ProjectReference Include="..\Shared\{SOLUTION NAME}.Shared.csproj" />
  </ItemGroup>
  ```
  
  プレースホルダー `{SOLUTION NAME}` は、ソリューションの名前です。

* サーバー アプリの `Properties/launchSettings.json` ファイルで、ポート 5001 と 5002 でクライアント アプリにアクセスするように、Kestrel プロファイル (`{SOLUTION NAME}.Server`) の `applicationUrl` を構成します。

  ```json
  "applicationUrl": "https://localhost:5001;https://localhost:5002",
  ```

* サーバー アプリの `Startup.Configure` メソッド (`Startup.cs`) で、<xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection%2A> の呼び出しの後にある次の行を削除します。

  ```csharp
  app.UseBlazorFrameworkFiles();
  app.UseStaticFiles();

  app.UseRouting();

  app.UseEndpoints(endpoints =>
  {
      endpoints.MapRazorPages();
      endpoints.MapControllers();
      endpoints.MapFallbackToFile("index.html");
  });
  ```

  要求をクライアント アプリにマップするミドルウェアを追加します。 次の例では、ミドルウェアが以下のときに実行されるように構成します。

  * 要求のポートが、元のクライアント アプリの場合は 5001、追加されたクライアント アプリの場合は 5002 である。
  * 要求のホストが、元のクライアント アプリの場合は `firstapp.com`、追加されたクライアント アプリの場合は `secondapp.com` である。

    > [!NOTE]
    > このセクションで示されている例では、以下に対する追加構成が必要です。
    >
    > * 例のホスト ドメイン `firstapp.com` および `secondapp.com` でのアプリへのアクセス。
    > * クライアント アプリで TLS セキュリティ (HTTPS) を有効にするための証明書。
    >
    > 必要な構成はこの記事の範囲を超えており、ソリューションのホスト方法によって異なります。 詳細については、[ホストと展開に関する記事](xref:host-and-deploy/index)を参照してください。

  前に行を削除した場所に、次のコードを配置します。

  ```csharp
  app.MapWhen(ctx => ctx.Request.Host.Port == 5001 || 
      ctx.Request.Host.Equals("firstapp.com"), first =>
  {
      first.Use((ctx, nxt) =>
      {
          ctx.Request.Path = "/FirstApp" + ctx.Request.Path;
          return nxt();
      });

      first.UseBlazorFrameworkFiles("/FirstApp");
      first.UseStaticFiles();
      first.UseStaticFiles("/FirstApp");
      first.UseRouting();

      first.UseEndpoints(endpoints =>
      {
          endpoints.MapControllers();
          endpoints.MapFallbackToFile("/FirstApp/{*path:nonfile}", 
              "FirstApp/index.html");
      });
  });
  
  app.MapWhen(ctx => ctx.Request.Host.Port == 5002 || 
      ctx.Request.Host.Equals("secondapp.com"), second =>
  {
      second.Use((ctx, nxt) =>
      {
          ctx.Request.Path = "/SecondApp" + ctx.Request.Path;
          return nxt();
      });

      second.UseBlazorFrameworkFiles("/SecondApp");
      second.UseStaticFiles();
      second.UseStaticFiles("/SecondApp");
      second.UseRouting();

      second.UseEndpoints(endpoints =>
      {
          endpoints.MapControllers();
          endpoints.MapFallbackToFile("/SecondApp/{*path:nonfile}", 
              "SecondApp/index.html");
      });
  });
  ```

* サーバー アプリの天気予報コントローラー (`Controllers/WeatherForecastController.cs`) で、`WeatherForecastController` に対する既存のルート (`[Route("[controller]")]`) を次のルートに置き換えます。

  ```csharp
  [Route("FirstApp/[controller]")]
  [Route("SecondApp/[controller]")]
  ```

  前にサーバー アプリの `Startup.Configure` メソッドに追加したミドルウェアでは、`/WeatherForecast` に対する受信要求が、ポート (5001/5002) またはドメイン (`firstapp.com`/`secondapp.com`) に応じて、`/FirstApp/WeatherForecast` または `/SecondApp/WeatherForecast` に変更されます。 前のコントローラー ルートは、サーバー アプリからクライアント アプリに気象データを返すために必要です。

### <a name="static-assets-and-class-libraries"></a>静的アセットとクラス ライブラリ

静的アセットには、次の方法を使用します。

* アセットがクライアント アプリの `wwwroot` フォルダー内にある場合は、通常どおりにパスを指定します。

  ```razor
  <img alt="..." src="/{ASSET FILE NAME}" />
  ```

* アセットが [Razor クラス ライブラリ (RCL)](xref:blazor/components/class-libraries) の `wwwroot` フォルダーにある場合は、[RCL の記事](xref:razor-pages/ui-class#consume-content-from-a-referenced-rcl)のガイダンスに従って、クライアント アプリの静的アセットを参照します。

  ```razor
  <img alt="..." src="_content/{LIBRARY NAME}/{ASSET FILE NAME}" />
  ```

<!-- HOLD for reactivation at 5.x

::: moniker range=">= aspnetcore-5.0"

Components provided to a client app by a class library are referenced normally. If any components require stylesheets or JavaScript files, use either of the following approaches to obtain the static assets:

* The client app's `wwwroot/index.html` file can link (`<link>`) to the static assets.
* The component can use the framework's [`Link` component](xref:blazor/fundamentals/signalr#influence-html-head-tag-elements) to obtain the static assets.

The preceding approaches are demonstrated in the following examples.

::: moniker-end

::: moniker range="< aspnetcore-5.0"

-->

クラス ライブラリによってクライアント アプリに提供されるコンポーネントは、通常どおりに参照されます。 スタイルシートまたは JavaScript ファイルが必要なコンポーネントがある場合は、クライアント アプリの `wwwroot/index.html` ファイルに正しい静的アセットのリンクを含める必要があります。 次の例ではこれらの方法を示します。

<!-- HOLD for reactivation at 5.x

::: moniker-end

-->

次の `Jeep` コンポーネントをクライアント アプリのいずれかに追加します。 `Jeep` コンポーネントでは次のものが使用されています。

* クライアント アプリの `wwwroot` フォルダーにある画像 (`jeep-cj.png`)。
* [追加された Razor コンポーネント ライブラリ](xref:blazor/components/class-libraries) (`JeepImage`) の `wwwroot` フォルダーにある画像 (`jeep-yj.png`)。
* この例のコンポーネント (`Component1`) は、`JeepImage` ライブラリがソリューションに追加されると、RCL プロジェクト テンプレートによって自動的に作成されます。

```razor
@page "/Jeep"

<h1>1979 Jeep CJ-5&trade;</h1>

<p>
    <img alt="1979 Jeep CJ-5&trade;" src="/jeep-cj.png" />
</p>

<h1>1991 Jeep YJ&trade;</h1>

<p>
    <img alt="1991 Jeep YJ&trade;" src="_content/JeepImage/jeep-yj.png" />
</p>

<p>
    <em>Jeep CJ-5</em> and <em>Jeep YJ</em> are a trademarks of 
    <a href="https://www.fcagroup.com">Fiat Chrysler Automobiles</a>.
</p>

<JeepImage.Component1 />
```

> [!WARNING]
> 画像を所有している場合を除き、車両の画像を公開 **しないでください**。 そうしないと、著作権侵害のリスクがあります。

<!-- HOLD for reactivation at 5.x

::: moniker range=">= aspnetcore-5.0"

The library's `jeep-yj.png` image can also be added to the library's `Component1` component (`Component1.razor`). To provide the `my-component` CSS class to the client app's page, link to the library's stylesheet using the framework's [`Link` component](xref:blazor/fundamentals/signalr#influence-html-head-tag-elements):

```razor
<div class="my-component">
    <Link href="_content/JeepImage/styles.css" rel="stylesheet" />

    <h1>JeepImage.Component1</h1>

    <p>
        This Blazor component is defined in the <strong>JeepImage</strong> package.
    </p>

    <p>
        <img alt="1991 Jeep YJ&trade;" src="_content/JeepImage/jeep-yj.png" />
    </p>
</div>
```

An alternative to using the [`Link` component](xref:blazor/fundamentals/signalr#influence-html-head-tag-elements) is to load the stylesheet from the client app's `wwwroot/index.html` file. This approach makes the stylesheet available to all of the components in the client app:

```html
<head>
    ...
    <link href="_content/JeepImage/styles.css" rel="stylesheet" />
</head>
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

-->

ライブラリの `jeep-yj.png` 画像は、ライブラリの `Component1` コンポーネント (`Component1.razor`) に追加することもできます。

```razor
<div class="my-component">
    <h1>JeepImage.Component1</h1>

    <p>
        This Blazor component is defined in the <strong>JeepImage</strong> package.
    </p>

    <p>
        <img alt="1991 Jeep YJ&trade;" src="_content/JeepImage/jeep-yj.png" />
    </p>
</div>
```

クライアント アプリの `wwwroot/index.html` ファイルでは、次の `<link>` タグを追加してライブラリのスタイルシートを要求します。

```html
<head>
    ...
    <link href="_content/JeepImage/styles.css" rel="stylesheet" />
</head>
```

<!-- HOLD for reactivation at 5.x

::: moniker-end

-->

クライアント アプリの `NavMenu` コンポーネント (`Shared/NavMenu.razor`) に、`Jeep` コンポーネントへのナビゲーションを追加します。

```razor
<li class="nav-item px-3">
    <NavLink class="nav-link" href="Jeep">
        <span class="oi oi-list-rich" aria-hidden="true"></span> Jeep
    </NavLink>
</li>
```

RCL の詳細については、以下を参照してください。

* <xref:blazor/components/class-libraries>
* <xref:razor-pages/ui-class>

## <a name="standalone-deployment"></a>スタンドアロン展開

"*スタンドアロン デプロイ*" により、Blazor WebAssembly アプリが、クライアントによって直接要求される静的ファイルのセットとして提供されます。 任意の静的ファイル サーバーで Blazor アプリを提供できます。

スタンドアロンのデプロイ アセットは、`/bin/Release/{TARGET FRAMEWORK}/publish/wwwroot` フォルダーに発行されます。

### <a name="azure-app-service"></a>Azure App Service

Blazor WebAssembly アプリは、[IIS](#iis) 上でアプリをホストするために使用される Windows 上の Azure App Service にデプロイできます。

スタンドアロンの Blazor WebAssembly アプリを Azure App Service for Linux にデプロイすることは、現在サポートされていません。 現時点では、アプリをホストする Linux サーバー イメージは使用できません。 このシナリオを可能にするための取り組みが進行中です。

### <a name="iis"></a>IIS

IIS は、Blazor アプリ対応の静的ファイル サーバーです。 Blazor をホストするよう IIS を構成する方法については、「[IIS で静的 Web サイトを構築する](/iis/manage/creating-websites/scenario-build-a-static-website-on-iis)」を参照してください。

発行されたアセットは、`/bin/Release/{TARGET FRAMEWORK}/publish` フォルダーに作成されます。 `publish` フォルダーのコンテンツを、Web サーバーまたはホスティング サービス上でホストします。

#### <a name="webconfig"></a>web.config

Blazor プロジェクトが発行されると、`web.config` ファイルが以下の IIS 構成で作成されます。

* 各ファイル拡張子に対して設定される MIME の種類は次のとおりです。
  * `.dll`: `application/octet-stream`
  * `.json`: `application/json`
  * `.wasm`: `application/wasm`
  * `.woff`: `application/font-woff`
  * `.woff2`: `application/font-woff`
* 次の MIME の種類に対しては、HTTP 圧縮が有効にされます。
  * `application/octet-stream`
  * `application/wasm`
* URL Rewrite Module のルールが確立されます。
  * アプリの静的なアセットが存在するサブディレクトリ (`wwwroot/{PATH REQUESTED}`) が提供されます。
  * ファイル以外のアセットの要求が、アプリの静的アセット フォルダー内の既定のドキュメント (`wwwroot/index.html`) にリダイレクトされるように、SPA フォールバック ルーティングが作成されます。
  
#### <a name="use-a-custom-webconfig"></a>カスタム web.config を使用する

カスタムの `web.config` ファイルを使用するには、カスタムの `web.config` ファイルをプロジェクト フォルダーのルートに配置します。 アプリのプロジェクト ファイルで `PublishIISAssets` を使用して IIS 固有のアセットを発行するようにプロジェクトを構成し、プロジェクトを発行します。

```xml
<PropertyGroup>
  <PublishIISAssets>true</PublishIISAssets>
</PropertyGroup>
```

#### <a name="install-the-url-rewrite-module"></a>URL リライト モジュールをインストールする

[URL Rewrite Module](https://www.iis.net/downloads/microsoft/url-rewrite) は、URL の書き換えに必要となります。 このモジュールは既定ではインストールされていません。また、Web サーバー (IIS) の役割サービス機能としてインストールすることはできません。 モジュールは、IIS Web サイトからダウンロードする必要があります。 Web Platform Installer を使用してモジュールをインストールします。

1. ローカルで、[URL Rewrite Module のダウンロード ページ](https://www.iis.net/downloads/microsoft/url-rewrite#additionalDownloads)に移動します。 英語版については、**WebPI** を選択して WebPI インストーラーをダウンロードします。 その他の言語版については、サーバーの適切なアーキテクチャ (x86/x64) を選択して、インストーラーをダウンロードします。
1. インストーラーをサーバーにコピーします。 インストーラーを実行します。 **[インストール]** ボタンを選択して、ライセンス条項に同意します。 インストールが完了した後、サーバーの再起動は必要はありません。

#### <a name="configure-the-website"></a>Web サイトを構成する

Web サイトの **物理パス** をアプリのフォルダーに設定します。 フォルダーには次のものが含まれます。

* `web.config` ファイル。IIS ではこのファイルを使用して、必要なリダイレクト ルールやファイルのコンテンツの種類など、Web サイトの構成が行われます。
* アプリの静的なアセット フォルダー。

#### <a name="host-as-an-iis-sub-app"></a>IIS サブアプリとしてホストする

スタンドアロン アプリが IIS サブアプリとしてホストされている場合は、次のいずれかを実行します。

* 継承された ASP.NET Core モジュール ハンドラーを無効にします。

  Blazor アプリで発行された `web.config` ファイル内のハンドラーを、`<handlers>` セクションをファイルに追加することで削除します。

  ```xml
  <handlers>
    <remove name="aspNetCore" />
  </handlers>
  ```

* `inheritInChildApplications` が `false` に設定された `<location>` 要素を使用して、ルート (親) アプリの `<system.webServer>` セクションの継承を無効にします。

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <configuration>
    <location path="." inheritInChildApplications="false">
      <system.webServer>
        <handlers>
          <add name="aspNetCore" ... />
        </handlers>
        <aspNetCore ... />
      </system.webServer>
    </location>
  </configuration>
  ```

ハンドラーの削除または継承の無効化は、[アプリの基本パスの構成](xref:blazor/host-and-deploy/index#app-base-path)に加えて行われます。 IIS でサブアプリを構成するときに、アプリの `index.html` ファイル内のアプリのベース パスを、使用している IIS の別名に設定します。

#### <a name="brotli-and-gzip-compression"></a>Brotli と Gzip の圧縮

*このセクションは、スタンドアロンの Blazor WebAssembly アプリにのみ適用されます。ホストされている Blazor アプリでは、このセクションにリンクされているファイルではなく、既定の ASP.NET Core アプリ `web.config` ファイルが使用されます。*

`web.config` を使用して、スタンドアロンの Blazor WebAssembly 用に Brotli または Gzip で圧縮された Blazor アセットを提供するように IIS を構成できます。 構成ファイルの例については、[`web.config`](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/blazor/host-and-deploy/webassembly/_samples/web.config?raw=true) をご覧ください。

次のシナリオでは、サンプル `web.config` ファイルを追加で構成することが必要になる場合があります。

* このアプリの仕様では、次のいずれかが呼び出されます。
  * サンプル `web.config` ファイルで構成されていない使用中の圧縮ファイル。
  * サンプル `web.config` ファイルで非圧縮形式で構成されている使用中の圧縮ファイル。
* サーバーの IIS 構成 (`applicationHost.config` など) では、サーバーレベルの IIS の既定値が提供されています。 サーバーレベルの構成によっては、サンプル `web.config` ファイルに含まれるものとは異なる IIS 構成が必要になる場合があります。

#### <a name="troubleshooting"></a>トラブルシューティング

Web サイトの構成にアクセスしようとしたときに、"*500 - 内部サーバー エラー*" という応答が返され、IIS マネージャーによりエラーがスローされた場合は、URL リライト モジュールがインストールされていることを確認します。 モジュールがインストールされていない場合、IIS では `web.config` ファイルを解析できません。 これは、IIS マネージャーによる Web サイトの構成の読み込み、そして Web サイトによる Blazor の静的ファイルの提供を阻止するためのものです。

IIS への展開に関するトラブルシューティングの詳細については、「<xref:test/troubleshoot-azure-iis>」を参照してください。

### <a name="azure-storage"></a>Azure ストレージ

[Azure Storage](/azure/storage/) の静的ファイル ホスティングにより、サーバーレス Blazor アプリ ホスティングが可能になります。 カスタム ドメイン名の Azure Content Delivery Network (CDN) と HTTPS がサポートされています。

ストレージ アカウントでホスティングされている静的 Web サイトに Blob service サービスが有効になっているとき:

* **インデックス ドキュメント名** を `index.html` に設定します。
* **エラー ドキュメント パス** を `index.html` に設定します。 Razor コンポーネントとその他の非ファイル エンドポイントは、Blob service で保管される静的コンテンツの物理パスに置かれません。 このようなリソースの 1 つに対して受け取った要求を Blazor ルーターで処理しなければならないとき、Blob service によって生成された *404 - Not Found* エラーにより、要求が **エラー ドキュメント パス** に転送されます。 `index.html` BLOB が返され、Blazor ルーターでパスが読み込まれ、処理されます。

ファイルの `Content-Type` ヘッダーに不適切な MIME の種類があるために、実行時にファイルが読み込まれない場合は、次のいずれかの操作を実行します。

* ファイルの展開時に適切な MIME の種類 (`Content-Type` ヘッダー) を設定するようにツールを構成します。
* アプリの展開後に、ファイルの MIME の種類 (`Content-Type` ヘッダー) を変更します。

  Storage Explorer (Azure portal) で、ファイルごとに次のようにします。
  
  1. ファイルを右クリックし、 **[プロパティ]** を選択します。
  1. **ContentType** を設定し、 **[保存]** ボタンを選択します。

詳細については、「[Azure Storage での静的 Web サイト ホスティング](/azure/storage/blobs/storage-blob-static-website)」を参照してください。

### <a name="nginx"></a>Nginx

以下に示す `nginx.conf` ファイルは、Nginx が対応するファイルをディスク上で見つけられないときに `index.html` ファイルを送信するよう Nginx を構成する方法を示すために、簡略化されています。

```
events { }
http {
    server {
        listen 80;

        location / {
            root      /usr/share/nginx/html;
            try_files $uri $uri/ /index.html =404;
        }
    }
}
```

[NGINX バースト レート制限](https://www.nginx.com/blog/rate-limiting-nginx/#bursts)を [`limit_req`](https://nginx.org/docs/http/ngx_http_limit_req_module.html#limit_req) で設定するとき、アプリによって行われる比較的大量の要求を受け入れる目的で、場合によっては、Blazor WebAssembly アプリの `burst` パラメーター値を大きくする必要があります。 最初に、値を 60 以上に設定します。

```
http {
    server {
        ...

        location / {
            ...

            limit_req zone=one burst=60 nodelay;
        }
    }
}
```

"*503 - サービスを利用できません*" という状態コードを要求が受信していることがブラウザー開発者ツールまたはネットワーク トラフィック ツールで示されている場合、この値を増やします。

運用環境での Nginx Web サーバーの構成に関する詳細については、「[Creating NGINX Plus and NGINX Configuration Files](https://docs.nginx.com/nginx/admin-guide/basic-functionality/managing-configuration-files/)」 (NGINX Plus と NGINX 構成ファイルの作成) を参照してください。

### <a name="apache"></a>Apache

Blazor WebAssembly アプリを CentOS 7 以降にデプロイするには:

1. Apache 構成ファイルを作成します。 次の例は、簡略化された構成ファイル (`blazorapp.config`) です。

   ```
   <VirtualHost *:80>
       ServerName www.example.com
       ServerAlias *.example.com

       DocumentRoot "/var/www/blazorapp"
       ErrorDocument 404 /index.html

       AddType application/wasm .wasm
       AddType application/octet-stream .dll
   
       <Directory "/var/www/blazorapp">
           Options -Indexes
           AllowOverride None
       </Directory>

       <IfModule mod_deflate.c>
           AddOutputFilterByType DEFLATE text/css
           AddOutputFilterByType DEFLATE application/javascript
           AddOutputFilterByType DEFLATE text/html
           AddOutputFilterByType DEFLATE application/octet-stream
           AddOutputFilterByType DEFLATE application/wasm
           <IfModule mod_setenvif.c>
           BrowserMatch ^Mozilla/4 gzip-only-text/html
           BrowserMatch ^Mozilla/4.0[678] no-gzip
           BrowserMatch bMSIE !no-gzip !gzip-only-text/html
       </IfModule>
       </IfModule>

       ErrorLog /var/log/httpd/blazorapp-error.log
       CustomLog /var/log/httpd/blazorapp-access.log common
   </VirtualHost>
   ```

1. Apache 構成ファイルを `/etc/httpd/conf.d/` ディレクトリ (CentOS 7 の既定の Apache 構成ディレクトリ) に配置します。

1. アプリのファイルを `/var/www/blazorapp` ディレクトリ (構成ファイルで `DocumentRoot` に指定された場所) に配置します。

1. Apache サービスを再起動します。

詳細については、[`mod_mime`](https://httpd.apache.org/docs/2.4/mod/mod_mime.html) および [`mod_deflate`](https://httpd.apache.org/docs/current/mod/mod_deflate.html) を参照してください。

### <a name="github-pages"></a>GitHub ページ

URL の書き換えを処理するために、`wwwroot/404.html` ファイルを、要求を `index.html` ページにリダイレクトするスクリプトと共に追加します。 例については、[SteveSandersonMS/BlazorOnGitHubPages GitHub リポジトリ](https://github.com/SteveSandersonMS/BlazorOnGitHubPages)を参照してください。

* [`wwwroot/404.html`](https://github.com/SteveSandersonMS/BlazorOnGitHubPages/blob/master/wwwroot/404.html)
* [ライブ サイト](https://stevesandersonms.github.io/BlazorOnGitHubPages/)

組織のサイトではなくプロジェクトのサイトを使用しているときは、`wwwroot/index.html` 内の `<base>` タグを更新します。 `href` 属性の値を、GitHub リポジトリの名前の末尾にスラッシュを付けたもの (例: `/my-repository/`) に設定します。 [SteveSandersonMS/BlazorOnGitHubPages GitHub リポジトリ](https://github.com/SteveSandersonMS/BlazorOnGitHubPages)では、[`.github/workflows/main.yml` 構成ファイル](https://github.com/SteveSandersonMS/BlazorOnGitHubPages/blob/master/.github/workflows/main.yml)による発行時に、基本 `href` が更新されます。

> [!NOTE]
> [SteveSandersonMS/BlazorOnGitHubPages GitHub リポジトリ](https://github.com/SteveSandersonMS/BlazorOnGitHubPages)は、.NET Foundation または Microsoft では、所有、管理、またはサポートされていません。

## <a name="host-configuration-values"></a>ホストの構成値

[Blazor WebAssembly アプリ](xref:blazor/hosting-models#blazor-webassembly)では、開発環境での実行時に以下のホスト構成値をコマンドライン引数として受け入れることができます。

### <a name="content-root"></a>コンテンツ ルート

`--contentroot` 引数では、アプリのコンテンツ ファイルを含むディレクトリへの絶対パスが設定されます ([コンテンツ ルート](xref:fundamentals/index#content-root))。 次の例では、`/content-root-path` はアプリのコンテンツ ルート パスです。

* コマンド プロンプトでアプリをローカルに実行するときに、引数を渡します。 アプリのディレクトリから、次のコマンドを実行します。

  ```dotnetcli
  dotnet run --contentroot=/content-root-path
  ```

* **IIS Express** プロファイル内のアプリの `launchSettings.json` ファイルに、エントリを追加します。 この設定は、Visual Studio デバッガーを使用し、コマンド プロンプトから `dotnet run` でアプリが実行されるときに使用されます。

  ```json
  "commandLineArgs": "--contentroot=/content-root-path"
  ```

* Visual Studio の **[プロパティ]**  >  **[デバッグ]**  >  **[アプリケーションの引数]** で、引数を指定します。 Visual Studio のプロパティ ページで引数を設定すると、その引数が `launchSettings.json` ファイルに追加されます。

  ```console
  --contentroot=/content-root-path
  ```

### <a name="path-base"></a>パス ベース

`--pathbase` 引数により、ルート以外の相対 URL パスでローカルで実行されているアプリに対して、アプリのベース パスを設定することができます (ステージングと運用環境の場合、`<base>` タグ `href` は `/` 以外のパスに設定されます)。 次の例では、`/relative-URL-path` はアプリのパス ベースです。 詳細については、「[アプリのベースパス](xref:blazor/host-and-deploy/index#app-base-path)」を参照してください。

> [!IMPORTANT]
> `<base>` タグの `href` に指定するパスとは異なり、`--pathbase` 引数値を渡すときはスラッシュ (`/`) を末尾に含めないでください。 `<base>` タグでアプリのベース パスが `<base href="/CoolApp/">` と指定されている場合 (末尾にスラッシュあり)、コマンドライン引数値としては `--pathbase=/CoolApp` を渡してください (末尾にスラッシュなし)。

* コマンド プロンプトでアプリをローカルに実行するときに、引数を渡します。 アプリのディレクトリから、次のコマンドを実行します。

  ```dotnetcli
  dotnet run --pathbase=/relative-URL-path
  ```

* **IIS Express** プロファイル内のアプリの `launchSettings.json` ファイルに、エントリを追加します。 この設定は、Visual Studio デバッガーを使用し、コマンド プロンプトから `dotnet run` でアプリを実行するときに使用されます。

  ```json
  "commandLineArgs": "--pathbase=/relative-URL-path"
  ```

* Visual Studio の **[プロパティ]**  >  **[デバッグ]**  >  **[アプリケーションの引数]** で、引数を指定します。 Visual Studio のプロパティ ページで引数を設定すると、その引数が `launchSettings.json` ファイルに追加されます。

  ```console
  --pathbase=/relative-URL-path
  ```

### <a name="urls"></a>URL

`--urls` 引数では、要求をリッスンするポートとプロトコルを使用して、IP アドレスまたはホスト アドレスが設定されます。

* コマンド プロンプトでアプリをローカルに実行するときに、引数を渡します。 アプリのディレクトリから、次のコマンドを実行します。

  ```dotnetcli
  dotnet run --urls=http://127.0.0.1:0
  ```

* **IIS Express** プロファイル内のアプリの `launchSettings.json` ファイルに、エントリを追加します。 この設定は、Visual Studio デバッガーを使用し、コマンド プロンプトから `dotnet run` でアプリを実行するときに使用されます。

  ```json
  "commandLineArgs": "--urls=http://127.0.0.1:0"
  ```

* Visual Studio の **[プロパティ]**  >  **[デバッグ]**  >  **[アプリケーションの引数]** で、引数を指定します。 Visual Studio のプロパティ ページで引数を設定すると、その引数が `launchSettings.json` ファイルに追加されます。

  ```console
  --urls=http://127.0.0.1:0
  ```

::: moniker range=">= aspnetcore-5.0"

## <a name="configure-the-trimmer"></a>トリマーを構成する

Blazor では、出力アセンブリから不要な中間言語 (IL) を削除するために、IL トリミング設定が各リリース ビルド上で実行されます。 詳細については、「<xref:blazor/host-and-deploy/configure-trimmer>」を参照してください。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

## <a name="configure-the-linker"></a>リンカーを構成する

Blazor では、出力アセンブリから不要な中間言語 (IL) を削除するために、IL リンク設定が各リリース ビルド上で実行されます。 詳細については、「<xref:blazor/host-and-deploy/configure-linker>」を参照してください。

::: moniker-end

## <a name="custom-boot-resource-loading"></a>カスタム ブート リソースの読み込み

Blazor WebAssembly アプリを `loadBootResource` 関数で初期化して、組み込みのブート リソース読み込みメカニズムをオーバーライドできます。 次のシナリオで `loadBootResource` を使用します。

* ユーザーが、タイムゾーン データや `dotnet.wasm` などの静的なリソースを CDN から読み込むことができるようにする。
* HTTP 要求を使用して圧縮されたアセンブリを読み込み、サーバーからの圧縮コンテンツのフェッチをサポートしていないホストのクライアントに展開する。
* 各 `fetch` 要求を新しい名前にリダイレクトして、リソースを別の名前に設定する。

`loadBootResource` パラメーターは次の表に表示されます。

| パラメーター    | 説明 |
| ------------ | ----------- |
| `type`       | リソースの型。 許容される型: `assembly`、`pdb`、`dotnetjs`、`dotnetwasm`、`timezonedata` |
| `name`       | リソースの名前。 |
| `defaultUri` | リソースの相対 URI または絶対 URI。 |
| `integrity`  | 応答で予想されるコンテンツを表す整合性文字列。 |

`loadBootResource` は読み込みプロセスをオーバーライドするために、次のいずれかを返します。

* URI 文字列。 次の例 (`wwwroot/index.html`) では、`https://my-awesome-cdn.com/` の CDN から次のファイルが提供されます。

  * `dotnet.*.js`
  * `dotnet.wasm`
  * タイムゾーン データ

  ```html
  ...

  <script src="_framework/blazor.webassembly.js" autostart="false"></script>
  <script>
    Blazor.start({
      loadBootResource: function (type, name, defaultUri, integrity) {
        console.log(`Loading: '${type}', '${name}', '${defaultUri}', '${integrity}'`);
        switch (type) {
          case 'dotnetjs':
          case 'dotnetwasm':
          case 'timezonedata':
            return `https://my-awesome-cdn.com/blazorwebassembly/3.2.0/${name}`;
        }
      }
    });
  </script>
  ```

* `Promise<Response>`. ヘッダーに `integrity` パラメーターを渡して、既定の整合性チェックの動作を保持します。

  次の例 (`wwwroot/index.html`) は、カスタム HTTP ヘッダーを送信要求に追加し、`integrity` パラメーターを経由して `fetch` 呼び出しに渡します。
  
  ```html
  <script src="_framework/blazor.webassembly.js" autostart="false"></script>
  <script>
    Blazor.start({
      loadBootResource: function (type, name, defaultUri, integrity) {
        return fetch(defaultUri, { 
          cache: 'no-cache',
          integrity: integrity,
          headers: { 'MyCustomHeader': 'My custom value' }
        });
      }
    });
  </script>
  ```

* `null`/`undefined`。既定の読み込み動作になります。

外部ソースは、ブラウザーがクロスオリジンのリソースの読み込みを許可するために必要な CORS ヘッダーを返す必要があります。 通常、既定では、必要なヘッダーが CDN によって提供されます。

カスタム動作の型のみ指定する必要があります。 `loadBootResource` に指定されていない型は、既定の読み込み動作に従ってフレームワークによって読み込まれます。

## <a name="change-the-filename-extension-of-dll-files"></a>DLL ファイルのファイル名拡張子を変更する

アプリで発行されている `.dll` ファイルのファイル名拡張子を変更する必要がある場合は、このセクションのガイダンスに従ってください。

アプリを発行した後、シェル スクリプトまたは DevOps ビルド パイプラインを使用して、別のファイル拡張子を使用するように `.dll` ファイルの名前を変更します。 アプリで発行された出力の `wwwroot` ディレクトリ (たとえば、`{CONTENT ROOT}/bin/Release/netstandard2.1/publish/wwwroot`) 内の `.dll` ファイルをターゲットにします。

次の例では、`.dll` ファイルの名前が `.bin` ファイル拡張子を使用するように変更されています。

Windows の場合:

```powershell
dir .\_framework\_bin | rename-item -NewName { $_.name -replace ".dll\b",".bin" }
((Get-Content .\_framework\blazor.boot.json -Raw) -replace '.dll"','.bin"') | Set-Content .\_framework\blazor.boot.json
```

サービス ワーカー アセットも使用されている場合は、次のコマンドを追加します。

```powershell
((Get-Content .\service-worker-assets.js -Raw) -replace '.dll"','.bin"') | Set-Content .\service-worker-assets.js
```

Linux または macOS の場合:

```console
for f in _framework/_bin/*; do mv "$f" "`echo $f | sed -e 's/\.dll/.bin/g'`"; done
sed -i 's/\.dll"/.bin"/g' _framework/blazor.boot.json
```

サービス ワーカー アセットも使用されている場合は、次のコマンドを追加します。

```console
sed -i 's/\.dll"/.bin"/g' service-worker-assets.js
```
   
`.bin` とは異なるファイル拡張子を使用するには、前のコマンドで `.bin` を置き換えます。

圧縮された `blazor.boot.json.gz` と `blazor.boot.json.br` のファイルに対処するには、次のいずれかの方法を採用します。

* 圧縮された `blazor.boot.json.gz` と `blazor.boot.json.br` のファイルを削除します。 このアプローチでは、圧縮は無効になっています。
* 更新した `blazor.boot.json` ファイルを圧縮します。

上記のガイダンスは、サービス ワーカー アセットが使用されている場合にも適用されます。 `wwwroot/service-worker-assets.js.br` と `wwwroot/service-worker-assets.js.gz` を削除または再圧縮します。 そうしないと、ブラウザーでのファイルの整合性チェックが失敗します。

次の Windows の例では、プロジェクトのルートに配置された PowerShell スクリプトを使用しています。

`ChangeDLLExtensions.ps1:`:

```powershell
param([string]$filepath,[string]$tfm)
dir $filepath\bin\Release\$tfm\wwwroot\_framework\_bin | rename-item -NewName { $_.name -replace ".dll\b",".bin" }
((Get-Content $filepath\bin\Release\$tfm\wwwroot\_framework\blazor.boot.json -Raw) -replace '.dll"','.bin"') | Set-Content $filepath\bin\Release\$tfm\wwwroot\_framework\blazor.boot.json
Remove-Item $filepath\bin\Release\$tfm\wwwroot\_framework\blazor.boot.json.gz
```

サービス ワーカー アセットも使用されている場合は、次のコマンドを追加します。

```powershell
((Get-Content $filepath\bin\Release\$tfm\wwwroot\service-worker-assets.js -Raw) -replace '.dll"','.bin"') | Set-Content $filepath\bin\Release\$tfm\wwwroot\service-worker-assets.js
```

プロジェクト ファイルでは、アプリの発行後にスクリプトが実行されます。

```xml
<Target Name="ChangeDLLFileExtensions" AfterTargets="Publish" Condition="'$(Configuration)'=='Release'">
  <Exec Command="powershell.exe -command &quot;&amp; { .\ChangeDLLExtensions.ps1 '$(SolutionDir)' '$(TargetFramework)'}&quot;" />
</Target>
```

> [!NOTE]
> 同じアセンブリの名前変更と遅延読み込みについては、<xref:blazor/webassembly-lazy-load-assemblies#onnavigateasync-events-and-renamed-assembly-files> のガイダンスを参照してください。

## <a name="resolve-integrity-check-failures"></a>整合性チェックの失敗を解決する

Blazor WebAssembly によってアプリのスタートアップ ファイルがダウンロードされると、応答に対して整合性チェックを実行するようにブラウザーに指示が出されます。 `blazor.boot.json` ファイルの情報を使用して、`.dll`、`.wasm`、およびその他のファイルに対して想定される SHA-256 ハッシュ値が指定されます。 これには次のような理由で利点があります。

* たとえばユーザーがアプリケーション ファイルをダウンロードしているときに Web サーバーに新しい展開が適用された場合など、一貫性のないファイルのセットを読み込むリスクがなくなります。 不整合なファイルは未定義の動作につながる可能性があります。
* ユーザーのブラウザーが不整合または無効な応答をキャッシュしないようになります (その場合、ページを手動で更新してもアプリを起動できなくなる可能性があります)。
* 応答を安全にキャッシュできるようになり、想定される SHA-256 ハッシュ自体が変更されるまでサーバー側の変更がチェックされません。そのため、後続のページ読み込みに伴う要求の数が減り、完了までにかかる時間が大幅に短縮されます。

想定される SHA-256 ハッシュに一致しない応答が Web サーバーから返された場合、ブラウザーの開発者コンソールに次のようなエラーが表示されます。

> Failed to find a valid digest in the 'integrity' attribute for resource 'https://myapp.example.com/\_framework/MyBlazor App.dll' with computed SHA-256 integrity 'IIa70iwvmEg5WiDV17OpQ5eCztNYqL186J56852RpJY='. (計算された SHA-256 整合性 'IIa70iwvmEg5WiDV17OpQ5eCztNYqL186J56852RpJY=' を持つリソース 'https://myapp.example.com/\_framework/MyBlazor App.dll' の 'integrity' 属性に有効なダイジェストが見つかりませんでした。) The resource has been blocked. (リソースがブロックされています。)

ほとんどの場合、これは整合性チェック自体に関する問題では "*ありません*"。 代わりにこれは他の問題があることを意味し、整合性チェックによってその他の問題に関する警告が表示されます。

### <a name="diagnosing-integrity-problems"></a>整合性に関する問題の診断

アプリがビルドされると、生成された `blazor.boot.json` マニフェストによって、ビルド出力が生成された時点でのブート リソース (たとえば `.dll`、`.wasm`、およびその他のファイル) の SHA-256 ハッシュが記述されます。 整合性チェックは、`blazor.boot.json` の SHA-256 ハッシュがブラウザーに配信されるファイルと一致している限り成功します。

これに失敗する一般的な理由は次のとおりです。

 * Web サーバーからの応答が、ブラウザーが要求したファイルではなく、エラー (たとえば *404 - Not Found* や *500 - Internal Server Error*) である場合。 これはブラウザーによって、応答エラーとしてではなく整合性チェックの失敗として報告されます。
 * ファイルのビルドからブラウザーへの配信の間に、何らかによってファイルの内容が変更された場合。 次のような場合に発生します。
   * ユーザーまたはビルド ツールによって、ビルド出力が手動で変更された場合。
   * 展開プロセスの何らかの側面によってファイルが変更された場合。 たとえば、Git ベースの展開メカニズムを使用する場合は、Windows でファイルをコミットして Linux でチェックアウトすると、Git によって Windows スタイルの改行コードが Unix スタイルの改行コードに透過的に変換されることに注意してください。 ファイルの改行コードを変更すると、SHA-256 ハッシュが変更されます。 この問題を回避するには、[`.gitattributes` を使用してビルド成果物を `binary` ファイルとして扱う](https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes)ことを検討してください。
   * Web サーバーによるファイル提供の一環として、その内容が変更された場合。 たとえば、一部のコンテンツ配信ネットワーク (CDN) では、HTML の[縮小](xref:client-side/bundling-and-minification#minification)が自動的に試行され、それによって変更されます。 場合によっては、このような機能を無効にする必要があります。

これらのうちどれが自分のケースに当てはまるか診断するには:

 1. エラー メッセージを確認し、エラーをトリガーしているファイルをメモします。
 1. ブラウザーの開発者ツールを開き、 *[ネットワーク]* タブを確認します。必要に応じて、ページを再度読み込み、要求と応答の一覧を表示します。 その一覧でエラーをトリガーしているファイルを見つけます。
 1. 応答に含まれる HTTP 状態コードを確認します。 サーバーから *200 - OK* (または別の 2xx 状態コード) 以外のものが返された場合は、サーバー側に診断すべき問題があります。 たとえば、状態コード 403 は承認に関する問題があることを意味しますが、状態コード 500 は、サーバーが未指定のエラーで失敗していることを意味します。 サーバー側のログを参照してアプリを診断し、修正してください。
 1. リソースに対する状態コードが *200 - OK* の場合は、ブラウザーの開発者ツールで応答の内容を確認し、その内容が予想されるデータと一致していることを確認します。 たとえば、よくある問題は、他のファイルに対しても要求から `index.html` データが返されるように、ルーティングを誤って構成してしまうことです。 `.wasm` 要求への応答が WebAssembly であり、`.dll` 要求への応答が .NET アセンブリ バイナリであることを確認してください。 そうでない場合、サーバー側のルーティングに関する問題を診断する必要があります。
 1. 「[整合性 PowerShell スクリプトのトラブルシューティング](#troubleshoot-integrity-powershell-script)」で、アプリの発行および展開された出力を検証しようと試みます。

サーバーから正しいと思われるデータが返されていることを確認した場合は、ファイルのビルドと配布の間で何か別の原因によって内容が変更されています。 これを調査するには:

 * ファイルがビルドされた後にファイルが変更される場合に備えて、ビルド ツールチェーンと展開メカニズムを調べます。 この例としては、前に説明したように、Git によってファイルの改行コードが変換される場合が挙げられます。
 * 応答を動的に変更する (たとえば、HTML を縮小しようとする) ように設定されている場合があるため、Web サーバーまたは CDN の構成を確認します。 Web サーバーに HTTP 圧縮が実装されていても問題ありません (たとえば `content-encoding: br` や `content-encoding: gzip` が返される場合)。これは展開後の結果には影響しないためです。 ただし、Web サーバーによって圧縮されていないデータが変更される場合は "*問題があります*"。

### <a name="troubleshoot-integrity-powershell-script"></a>整合性 PowerShell スクリプトのトラブルシューティング

[`integrity.ps1`](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/blazor/host-and-deploy/webassembly/_samples/integrity.ps1?raw=true) PowerShell スクリプトを使用して、発行および展開された Blazor アプリを検証します。 このスクリプトは、アプリに Blazor フレームワークでは特定できない整合性の問題がある場合の出発点として提供されています。 使用するアプリに対してスクリプトのカスタマイズが必要になる場合があります。

スクリプトを実行すると、`publish` フォルダー内のファイルと展開されたアプリからダウンロードしたファイルがチェックされ、整合性ハッシュを含むさまざまなマニフェストの問題が検出されます。 これらのチェックにより、最も一般的な問題が検出されます。

* 発行された出力で、知らずにファイルを変更した。
* アプリが展開ターゲットに正しく展開されなかった、または展開ターゲットの環境内で何かが変更された。
* 展開されたアプリと、アプリの発行からの出力の間に違いがある。

PowerShell コマンド シェルで次のコマンドを使用してスクリプトを呼び出します。

```powershell
.\integrity.ps1 {BASE URL} {PUBLISH OUTPUT FOLDER}
```

プレースホルダー:

* `{BASE URL}`: 展開されたアプリの URL。
* `{PUBLISH OUTPUT FOLDER}`: アプリの `publish` フォルダー、またはアプリが展開のために発行される場所へのパス。

> [!NOTE]
> [Bitdefender](https://www.bitdefender.com) ウイルス検索プログラムを使用するシステムに `dotnet/AspNetCore.Docs` GitHub リポジトリを複製するには、`integrity.ps1` スクリプトの Bitdefender に例外を追加します。 リポジトリを複製する前に Bitdefender に例外を追加して、ウイルス検索プログラムによってスクリプトが検疫されないようにします。 次の例は、Windows システム上の複製されたリポジトリのスクリプトへの一般的なパスです。 必要に応じてパスを調整します。 プレースホルダー `{USER}` はユーザーのパスのセグメントです。
>
> ```
> C:\Users\{USER}\Documents\GitHub\AspNetCore.Docs\aspnetcore\blazor\host-and-deploy\webassembly\_samples\integrity.ps1
> ```

### <a name="disable-integrity-checking-for-non-pwa-apps"></a>非 PWA アプリの整合性チェックを無効にする

ほとんどの場合、整合性チェックを無効にしないでください。 整合性チェックを無効にしても、予期しない応答の原因となった根本的な問題は解決されず、前述の利点が失われる結果になります。

Web サーバーから一貫した応答が返されるとは限らないため、整合性チェックを無効にするしかないケースがあります。 整合性チェックを無効にするには、Blazor WebAssembly プロジェクトの `.csproj` ファイル内のプロパティ グループに次を追加します。

```xml
<BlazorCacheBootResources>false</BlazorCacheBootResources>
```

`BlazorCacheBootResources` により、SHA-256 ハッシュに基づいて `.dll`、`.wasm`、およびその他のファイルをキャッシュする Blazor の既定の動作も無効になります。このプロパティによって、SHA-256 ハッシュの正確性を信頼できないことが指定されるためです。 この設定を使用しても、ブラウザーの通常の HTTP キャッシュによってこれらのファイルがキャッシュされる可能性がありますが、このような状況が発生するかどうかは、Web サーバーの構成と、それによって提供される `cache-control` ヘッダーによって異なります。

> [!NOTE]
> `BlazorCacheBootResources` プロパティによって[プログレッシブ Web アプリケーション (PWA)](xref:blazor/progressive-web-app) の整合性チェックが無効になることはありません。 PWA に関連するガイダンスについては、「[PWA の整合性チェックを無効にする](#disable-integrity-checking-for-pwas)」セクションをご覧ください。

### <a name="disable-integrity-checking-for-pwas"></a>PWA の整合性チェックを無効にする

Blazor のプログレッシブ Web アプリケーション (PWA) テンプレートには、オフライン使用のためにアプリケーション ファイルをフェッチおよび格納するための推奨される `service-worker.published.js` ファイルが含まれています。 これは通常のアプリの起動メカニズムとは別のプロセスであり、独自の整合性チェック ロジックを備えています。

`service-worker.published.js` ファイル内に、次の行があります。

```javascript
.map(asset => new Request(asset.url, { integrity: asset.hash }));
```

整合性チェックを無効にするには、行を次のように変更して `integrity` パラメーターを削除します。

```javascript
.map(asset => new Request(asset.url));
```

ここでも、整合性チェックを無効にすることは、整合性チェックによって提供される安全性の保証が失われることを意味します。 たとえば、ユーザーのブラウザーでアプリをキャッシュしている瞬間に、新しいバージョンを展開した場合、古い展開から一部のファイルがキャッシュされ、新しい展開から別のファイルがキャッシュされるリスクがあります。 そのような場合、さらなる更新プログラムを展開するまで、アプリは破損した状態のままになります。
