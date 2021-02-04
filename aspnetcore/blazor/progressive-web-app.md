---
title: ASP.NET Core Blazor WebAssembly を使用してプログレッシブ Web アプリケーションをビルドする
author: guardrex
description: 最新のブラウザー機能を使用してデスクトップ アプリのように動作する、Blazor ベースのプログレッシブ Web アプリケーション (PWA) をビルドする方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/11/2021
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
uid: blazor/progressive-web-app
ms.openlocfilehash: fcf06295deb41f304b92caa82535a1197c909898
ms.sourcegitcommit: 75db2f684a9302b0be7925eab586aa091c6bd19f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/01/2021
ms.locfileid: "99238238"
---
# <a name="build-progressive-web-applications-with-aspnet-core-blazor-webassembly"></a>ASP.NET Core Blazor WebAssembly を使用してプログレッシブ Web アプリケーションをビルドする

作成者: [Steve Sanderson](https://github.com/SteveSandersonMS)

プログレッシブ Web アプリケーション (PWA) は、通常、最新のブラウザーの API と機能を使用してデスクトップ アプリのように動作するシングル ページ アプリケーション (SPA) です。 Blazor WebAssembly は、標準ベースのクライアント側 Web アプリ プラットフォームであるため、次の機能に必要な任意のブラウザー API (PWA API を含む) を使用できます。

* ネットワーク速度に関係なく、オフラインで動作し、瞬時に読み込む。
* ブラウザー ウィンドウだけでなく、独自のアプリ ウィンドウでも実行できる。
* ホスト オペレーティング システムのスタート メニュー、ドッキング、またはホーム画面から起動される。
* ユーザーがアプリを使用していない場合でも、バックエンド サーバーからプッシュ通知を受信する。
* バックグラウンドで自動更新される。

このようなアプリを説明するために "*プログレッシブ*" という単語を使用しているのは、次の理由からです。

* ユーザーは、他の SPA と同じように、まず自分の Web ブラウザー内でアプリを検出して使用する可能性があります。
* その後、ユーザーはそれの OS へのインストールと、プッシュ通知の有効化に進みます。

## <a name="create-a-project-from-the-pwa-template"></a>PWA テンプレートからプロジェクトを作成する

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

**[新しいプロジェクトの作成]** ダイアログで新しい **Blazor WebAssembly アプリ** を作成するときに、 **[プログレッシブ Web アプリケーション]** チェック ボックスをオンにします。

![Visual Studio の [新しいプロジェクトの作成] ダイアログで [プログレッシブ Web アプリケーション] チェック ボックスが選択されている。](progressive-web-app/_static/image1.png)

<!--

# [Visual Studio for Mac](#tab/visual-studio-mac)

-->

# <a name="visual-studio-code--net-core-cli"></a>[Visual Studio Code / .NET Core CLI](#tab/visual-studio-code+netcore-cli)

次のコマンドを使用し、コマンド シェルで `--pwa` スイッチを使用して PWA プロジェクトを作成します。

```dotnetcli
dotnet new blazorwasm -o MyBlazorPwa --pwa
```

上記のコマンドでは、`-o|--output` オプションによって `MyBlazorPwa` という名前のアプリ用の新しいフォルダーが作成されます。

---

必要に応じて、ASP.NET Core でホストされるテンプレートから作成されたアプリに対して PWA を構成できます。 PWA シナリオは、ホスティング モデルに依存しません。

## <a name="convert-an-existing-blazor-webassembly-app-into-a-pwa"></a>既存の Blazor WebAssembly アプリを PWA に変換する

このセクションのガイダンスに従って、既存の Blazor WebAssembly アプリを PWA に変換します。

アプリのプロジェクト ファイル内で:

* 次の `ServiceWorkerAssetsManifest` プロパティを `PropertyGroup` に追加します。

  ```xml
    ...
    <ServiceWorkerAssetsManifest>service-worker-assets.js</ServiceWorkerAssetsManifest>
  </PropertyGroup>
   ```

* 次の `ServiceWorker` 項目を `ItemGroup` に追加します。

  ```xml
  <ItemGroup>
    <ServiceWorker Include="wwwroot\service-worker.js" 
      PublishedContent="wwwroot\service-worker.published.js" />
  </ItemGroup>
  ```

静的資産を取得するには、次の **いずれか** の方法を使用します。

::: moniker range=">= aspnetcore-5.0"

* コマンド シェルで [`dotnet new`](/dotnet/core/tools/dotnet-new) コマンドを使用して、新しい PWA プロジェクトを個別に作成します。

  ```dotnetcli
  dotnet new blazorwasm -o MyBlazorPwa --pwa
  ```
  
  上記のコマンドでは、`-o|--output` オプションによって `MyBlazorPwa` という名前のアプリ用の新しいフォルダーが作成されます。
  
  アプリを最新リリース用に変換しない場合は、`-f|--framework` オプションを渡します。 次の例は、ASP.NET Core バージョン 3.1 用のアプリを作成します。
  
  ```dotnetcli
  dotnet new blazorwasm -o MyBlazorPwa --pwa -f netcoreapp3.1
  ```

* 次の URL の ASP.NET Core GitHub リポジトリに移動します。これは、5.0 リリースの参照ソースと資産にリンクしています。 5\.0 リリースのアプリを変換しない場合は、アプリに適用する **[Switch branches or tags]\(分岐またはタグの切り替え\)** ドロップダウン リストから、使用しているリリースを選択します。

  [dotnet/aspnetcore (リリース 5.0) Blazor WebAssembly プロジェクト テンプレート `wwwroot` フォルダー](https://github.com/dotnet/aspnetcore/tree/release/5.0/src/ProjectTemplates/Web.ProjectTemplates/content/ComponentsWebAssembly-CSharp/Client/wwwroot)

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* コマンド シェルで [`dotnet new`](/dotnet/core/tools/dotnet-new) コマンドを使用して、新しい PWA プロジェクトを個別に作成します。 `-f|--framework` オプションを渡してバージョンを選択します。 次の例は、ASP.NET Core バージョン 3.1 用のアプリを作成します。
  
  ```dotnetcli
  dotnet new blazorwasm -o MyBlazorPwa --pwa -f netcoreapp3.1
  ```
  
  上記のコマンドでは、`-o|--output` オプションによって `MyBlazorPwa` という名前のアプリ用の新しいフォルダーが作成されます。

* 次の URL の ASP.NET Core GitHub リポジトリに移動します。これは、3.1 リリースの参照ソースと資産にリンクしています。

  [dotnet/aspnetcore (リリース 3.1) Blazor WebAssembly プロジェクト テンプレート `wwwroot` フォルダー](https://github.com/dotnet/aspnetcore/tree/release/3.1/src/ProjectTemplates/ComponentsWebAssembly.ProjectTemplates/content/ComponentsWebAssembly-CSharp/Client/wwwroot)

  > [!NOTE]
  > Blazor WebAssembly プロジェクト テンプレートの URL は、ASP.NET Core 3.1 のリリース後に変更されました。 5\.0 以降の参照資産は、次の URL から入手できます。
  >
  > [dotnet/aspnetcore (リリース 5.0) Blazor WebAssembly プロジェクト テンプレート `wwwroot` フォルダー](https://github.com/dotnet/aspnetcore/tree/release/5.0/src/ProjectTemplates/Web.ProjectTemplates/content/ComponentsWebAssembly-CSharp/Client/wwwroot)

::: moniker-end

作成したアプリのソース `wwwroot` フォルダーから、または `dotnet/aspnetcore` GitHub リポジトリの参照資産から、次のファイルをアプリの `wwwroot` フォルダーにコピーします。

* `icon-512.png`
* `manifest.json`
* `service-worker.js`
* `service-worker.published.js`

アプリの `wwwroot/index.html` ファイルで、次のことを行います。

* マニフェストとアプリのアイコンに `<link>` 要素を追加します。

  ```html
  <link href="manifest.json" rel="manifest" />
  <link rel="apple-touch-icon" sizes="512x512" href="icon-512.png" />
  ```

* `blazor.webassembly.js` スクリプト タグの直後に、次の `<script>` タグを終了タグ `</body>` の内側に追加します。

  ```html
      ...
      <script>navigator.serviceWorker.register('service-worker.js');</script>
  </body>
  ```

## <a name="installation-and-app-manifest"></a>インストールとアプリ マニフェスト

PWA テンプレートを使用して作成されたアプリにアクセスするときに、ユーザーはアプリを OS のスタート メニュー、ドッキング、またはホーム画面にインストールすることを選択できます。 このオプションがどのように表示されるかは、ユーザーのブラウザーによって異なります。 Microsoft Edge や Chrome などのデスクトップ Chromium ベースのブラウザーを使用している場合、URL バーに **[追加]** ボタンが表示されます。 ユーザーが **[追加]** ボタンを選択すると、確認のダイアログが表示されます。

![Google Chrome の確認ダイアログには、"MyBlazorPwa" アプリのユーザーのための [インストール] ボタンがあります。](progressive-web-app/_static/image2.png)

iOS では、Safari の **[共有]** ボタンとその **[ホーム画面に追加]** オプションを使用して、PWA をインストールできます。 Android 用の Chrome では、ユーザーは右上隅の **[メニュー]** ボタンを選択してから、 **[ホーム画面に追加]** を選択する必要があります。

インストールが完了すると、アプリはアドレス バーのない独自のウィンドウに表示されます。

!['MyBlazorPwa' アプリは、Google Chrome でアドレス バーなしで実行されます。](progressive-web-app/_static/image3.png)

ウィンドウのタイトル、配色、アイコン、またはその他の詳細をカスタマイズするには、プロジェクトの `wwwroot` ディレクトリにある `manifest.json` ファイルを参照してください。 このファイルのスキーマは、Web 標準によって定義されます。 詳細については、[MDN Web ドキュメント:Web アプリ マニフェスト](https://developer.mozilla.org/docs/Web/Manifest)を参照してください。

## <a name="offline-support"></a>オフライン サポート

既定では、PWA テンプレート オプションを使用して作成されたアプリでは、オフラインでの実行がサポートされています。 ユーザーのアプリへの最初のアクセスは、アプリがオンラインになっている間に行う必要があります。 オフラインでの操作に必要なすべてのリソースが、ブラウザーによって自動的にダウンロードされてキャッシュされます。

> [!IMPORTANT]
> 開発サポートにより、変更してテストするという通常の開発サイクルが妨げられる可能性があります。 そのため、オフライン サポートは、"*公開された*" アプリに対してのみ有効です。 

> [!WARNING]
> オフライン対応 PWA を配布することを予定している場合は、[重要な警告と注意事項](#caveats-for-offline-pwas)がいくつかあります。 これらのシナリオはオフラインの PWA に固有のものであり、Blazor に固有のものではありません。 オフライン対応アプリがどのように動作するかを想定する前に、これらの注意事項を読んで理解しておいてください。

オフライン サポートのしくみを確認するには、次の手順を行います。

1. アプリの発行 詳細については、「<xref:blazor/host-and-deploy/index#publish-the-app>」を参照してください。
1. HTTPS をサポートするサーバーにアプリを配置し、アプリのセキュリティで保護された HTTPS アドレスを使用して、ブラウザーでアプリにアクセスします。
1. ブラウザーの開発者ツールを開き、 **[アプリケーション]** タブで *サービス ワーカー* がホストに登録されていることを確認できます。

   ![サービス ワーカーがアクティブ化され、実行されていることが示されている Google Chrome 開発者ツールの [アプリケーション] タブ。](progressive-web-app/_static/image4.png)

1. ページを再度読み込み、 **[ネットワーク]** タブを確認します。**サービス ワーカー** または **メモリ キャッシュ** が、すべてのページのアセットのソースとして一覧表示されます。

   ![すべてのページのアセットのソースが表示されている Google Chrome 開発者ツールの [ネットワーク] タブ。](progressive-web-app/_static/image5.png)

1. アプリを読み込むためにブラウザーがネットワーク アクセスに依存していないことを確認するには、次のいずれかの方法を使用します。

   * Web サーバーをシャットダウンし、アプリが引き続き正常に機能することを確認します。これには、ページの再読み込みも含まれます。 低速のネットワーク接続がある場合も同様に、アプリは引き続き正常に機能します。
   * **[ネットワーク]** タブでオフライン モードをシミュレートするようにブラウザーに指示します。

   ![ブラウザー モードのドロップダウンが [オンライン] から [オフライン] に変更されている Google Chrome 開発者ツールの [ネットワーク] タブ。](progressive-web-app/_static/image6.png)

サービス ワーカーを使用したオフライン サポートは、Blazor 固有ではなく、Web 標準です。 サービス ワーカーの詳細については、[MDN Web ドキュメント:サービス ワーカー API](https://developer.mozilla.org/docs/Web/API/Service_Worker_API) を参照してください。 サービス ワーカーの一般的な使用パターンの詳細については、[Google Web:サービス ワーカーのライフサイクル](https://developers.google.com/web/fundamentals/primers/service-workers/lifecycle)を参照してください。

Blazor の PWA テンプレートでは、次の 2 つのサービス ワーカー ファイルが生成されます。

* `wwwroot/service-worker.js`: 開発中に使用されます。
* `wwwroot/service-worker.published.js`: アプリが発行された後に使用されます。

2 つのサービス ワーカー ファイル間でロジックを共有するには、次のアプローチを検討してください。

* 共通のロジックを保持する 3 番目の JavaScript ファイルを追加する。
* [`self.importScripts`](https://developer.mozilla.org/docs/Web/API/WorkerGlobalScope/importScripts) を使用して、共通のロジックを両方のサービス ワーカー ファイルに読み込む。

### <a name="cache-first-fetch-strategy"></a>キャッシュ優先のフェッチ戦略

組み込みの `service-worker.published.js` サービス ワーカーでは、"*キャッシュ優先*" 戦略を使用して要求が解決されます。 これは、ユーザーがネットワークにアクセスできるかどうか、またはサーバーで使用可能な新しいコンテンツがあるかどうかに関係なく、サービス ワーカーがキャッシュされたコンテンツを返すことを意味します。

キャッシュ優先の戦略が重要な理由は、次のとおりです。

* **信頼性が確保されます。** ネットワーク アクセスはブール型の状態ではありません。 ユーザーは単純にオンラインまたはオフラインではありません。

  * ユーザーのデバイスではオンラインと見なされていても、ネットワークの速度が遅すぎるため、待機するのが現実的ではない場合があります。
  * ネットワークが特定の URL に対して無効な結果を返す場合があります。たとえば、特定の要求を現在ブロックまたはリダイレクトしている固定 WIFI ポータルがある場合などです。
  
  このように、ブラウザーの `navigator.onLine` API は信頼できないため、依存しないようにする必要があります。

* **正確性が保証されます。** オフライン リソースのキャッシュを構築する場合、サービス ワーカーではコンテンツ ハッシュを使用して、リソースの完全かつ自己整合スナップショットが一度にフェッチされることが保証されます。 このキャッシュはアトミック ユニットとして使用されます。 必要な唯一のバージョンが既にキャッシュされているので、ネットワークに新しいリソースを要求する必要はありません。 その他のすべてのバージョンには、不整合と非互換性のリスクがあります (たとえば、一緒にコンパイルされていない .NET アセンブリのバージョンを使用しようとした場合など)。

### <a name="background-updates"></a>バックグラウンド更新

メンタル モデルとして、オフラインファーストの PWA は、インストール可能なモバイル アプリのように動作すると考えることができます。 アプリはネットワーク接続に関係なく、すぐに起動しますが、インストールされているアプリ ロジックは、最新バージョンではない可能性がある特定の時点のスナップショットから取得されます。

Blazor PWA テンプレートを使用すると、ユーザーがアクセスしてネットワークに接続するたびに、バックグラウンドで自動的に自己更新を試行するアプリが生成されます。 このしくみは次のとおりです。

* コンパイル中に、プロジェクトによって "*サービス ワーカー アセット マニフェスト*" が生成されます。 既定では、これは `service-worker-assets.js` と呼ばれます。 マニフェストには、アプリがオフラインで動作するために必要なすべての静的リソース (.NET アセンブリ、JavaScript ファイル、CSS など) が、そのコンテンツ ハッシュと共にリストされています。 リソース リストは、キャッシュするリソースを認識できるように、サービス ワーカーによって読み込まれます。
* ユーザーがアプリにアクセスするたびに、ブラウザーによって `service-worker.js` と `service-worker-assets.js` がバックグラウンドで再要求されます。 ファイルは、インストールされている既存のサービス ワーカーとバイト単位で比較されます。 サーバーがこれらのファイルのいずれかに対して変更されたコンテンツを返す場合、サービス ワーカーはそれ自体の新しいバージョンをインストールしようとします。
* サービス ワーカーでは、それ自体の新しいバージョンをインストールするときに、オフライン リソース用に新しい別のキャッシュが作成され、そのキャッシュに `service-worker-assets.js` にリストされたリソースの入力が開始されます。 このロジックは、`service-worker.published.js` 内の `onInstall` 関数に実装されています。
* すべてのリソースがエラーなしで読み込まれ、すべてのコンテンツ ハッシュが一致すると、プロセスは正常に完了します。 成功した場合、新しいサービス ワーカーは、"*アクティブ化を待機している*" 状態になります。 ユーザーがアプリを閉じると (アプリのタブやウィンドウが残っていない状態)、新しいサービス ワーカーが "*アクティブ*" になり、後続のアプリへのアクセスに使用されます。 古いサービス ワーカーとそのキャッシュは削除されます。
* プロセスが正常に完了しなかった場合、新しいサービス ワーカー インスタンスは破棄されます。 ユーザーが次にアクセスすると (このとき、要求を完了できるほどクライアントのネットワーク接続が良好であることが望ましい)、更新プロセスがもう一度試行されます。

サービス ワーカー ロジックを編集して、このプロセスをカスタマイズします。 上記の動作はいずれも Blazor 固有のものではなく、PWA テンプレート オプションによって提供される既定のエクスペリエンスにすぎません。 詳細については、[MDN Web ドキュメント:サービス ワーカー API](https://developer.mozilla.org/docs/Web/API/Service_Worker_API) を参照してください。

### <a name="how-requests-are-resolved"></a>要求の解決方法

「[キャッシュ優先のフェッチ戦略](#cache-first-fetch-strategy)」セクションで説明したように、既定のサービス ワーカーでは "*キャッシュ優先*" 戦略が使用されます。これは、キャッシュされたコンテンツがあればそれが提供されることを意味します。 特定の URL に対してキャッシュされたコンテンツがない場合 (バックエンド API からデータを要求する場合など)、サービス ワーカーは通常のネットワーク要求にフォールバックします。 サーバーに到達可能な場合は、ネットワーク要求は成功します。 このロジックは、`service-worker.published.js` 内の `onFetch` 関数内に実装されています。

アプリの Razor コンポーネントがバックエンド API からのデータの要求に依存しており、ネットワークが利用できないことが原因で失敗した要求に対してわかりやすいユーザー エクスペリエンスを提供する場合は、アプリのコンポーネント内にロジックを実装します。 たとえば、<xref:System.Net.Http.HttpClient> 要求について `try/catch` を使用します。

### <a name="support-server-rendered-pages"></a>サーバーでレンダリングされるページのサポート

ユーザーが最初に `/counter` などの URL、またはアプリ内の他のディープ リンクに移動すると、どうなるかを考えます。 このような場合は、`/counter` としてキャッシュされたコンテンツを返すのではなく、ブラウザーでキャッシュされたコンテンツを `/index.html` として読み込み、Blazor WebAssembly アプリを起動する必要があります。 これらの初期要求は、次の対語として、"*ナビゲーション*" 要求と呼ばれます。

* `subresource` からは、画像、スタイルシート、またはその他のファイルが要求されます。
* `fetch/XHR` からは API データが要求されます。

既定のサービス ワーカーには、ナビゲーション要求用の特殊なケースのロジックが含まれています。 サービス ワーカーは、要求された URL に関係なく、`/index.html` のキャッシュされたコンテンツを返すことで要求を解決します。 このロジックは、`service-worker.published.js` 内の `onFetch` 関数に実装されています。

サーバーでレンダリングされた HTML を返す (キャッシュからの `/index.html` を提供するのではなく) 必要がある特定の URL がアプリにある場合は、サービス ワーカーのロジックを編集する必要があります。 `/Identity/` を含むすべての URL を、サーバーに対する通常のオンライン専用の要求として処理する必要がある場合は、`service-worker.published.js` `onFetch` ロジックを変更します。 次のコードを見つけます。

```javascript
const shouldServeIndexHtml = event.request.mode === 'navigate';
```

コードを次のように変更します。

```javascript
const shouldServeIndexHtml = event.request.mode === 'navigate'
  && !event.request.url.includes('/Identity/');
```

この変更を行わないと、ネットワーク接続に関係なく、サービス ワーカーによってこのような URL に対する要求がインターセプトされ、`/index.html` を使用してそれらが解決されます。

外部認証プロバイダー用にその他のエンドポイントをチェックに追加します。 次の例では、Google 認証用の `/signin-google` がチェックに追加されています。

```javascript
const shouldServeIndexHtml = event.request.mode === 'navigate'
  && !event.request.url.includes('/Identity/')
  && !event.request.url.includes('/signin-google');
```

コンテンツが常にネットワークからフェッチされる開発環境では、必要な操作はありません。

### <a name="control-asset-caching"></a>アセット キャッシュの制御

プロジェクトで `ServiceWorkerAssetsManifest` MSBuild プロパティが定義されている場合、Blazor のビルド ツールによって、指定された名前でサービス ワーカー アセット マニフェストが生成されます。 既定の PWA テンプレートでは、次のプロパティを含むプロジェクト ファイルが生成されます。

```xml
<ServiceWorkerAssetsManifest>service-worker-assets.js</ServiceWorkerAssetsManifest>
```

ファイルは `wwwroot` 出力ディレクトリに配置されるため、ブラウザーでは `/service-worker-assets.js` を要求することによって、このファイルを取得できます。 このファイルの内容を表示するには、テキスト エディターで `/bin/Debug/{TARGET FRAMEWORK}/wwwroot/service-worker-assets.js` を開きます。 ただし、ファイルは各ビルドで再生成されるため、編集しないでください。

既定では、このマニフェストには次の項目がリストされています。

* .NET アセンブリや .NET WebAssembly アセンブリ ランタイム ファイルなど、オフラインで機能するために必要な、Blazor 管理対象リソース。
* アプリの `wwwroot` ディレクトリに公開するためのすべてのリソース (画像、スタイルシート、JavaScript ファイルなど)。外部プロジェクトや NuGet パッケージによって提供される静的な Web アセットが含まれます。

サービス ワーカーによってフェッチおよびキャッシュされるリソースを制御するには、`service-worker.published.js` 内の `onInstall` でロジックを編集します。 既定では、`.html`、`.css`、`.js`、`.wasm` などの一般的な Web ファイル名の拡張子に一致するファイルのほか、Blazor WebAssembly に固有のファイルの種類 (`.dll`、`.pdb`) がサービス ワーカーによってフェッチおよびキャッシュされます。

アプリの `wwwroot` ディレクトリに存在しない追加のリソースを含めるには、次の例に示すように、追加の MSBuild `ItemGroup` エントリを定義します。

```xml
<ItemGroup>
  <ServiceWorkerAssetsManifestItem Include="MyDirectory\AnotherFile.json"
    RelativePath="MyDirectory\AnotherFile.json" AssetUrl="files/AnotherFile.json" />
</ItemGroup>
```

`AssetUrl` メタデータにより、ブラウザーでキャッシュするリソースをフェッチするときに使用されるベース相対 URL が指定されます。 これは、ディスク上の元のソース ファイル名に依存しないものにすることができます。

> [!IMPORTANT]
> `ServiceWorkerAssetsManifestItem` を追加しても、ファイルはアプリの `wwwroot` ディレクトリに公開されません。 公開の出力は個別に制御する必要があります。 `ServiceWorkerAssetsManifestItem` では、サービス ワーカー アセット マニフェストに追加のエントリが表示されるだけです。

## <a name="push-notifications"></a>プッシュ通知

他の PWA と同様に、Blazor WebAssembly PWA では、バックエンド サーバーからプッシュ通知を受信できます。 サーバーは、ユーザーがアプリを積極的に使用していない場合でも、いつでもプッシュ通知を送信できます。 たとえば、別のユーザーが関連するアクションを実行したときに、プッシュ通知を送信できます。

プッシュ通知を送信するメカニズムは、任意のテクノロジを使用できるバックエンド サーバーによって実装されているため、Blazor WebAssembly から完全に独立しています。 ASP.NET Core サーバーからプッシュ通知を送信する場合は、[Blazing Pizza ワークショップで行ったのと同様の手法を使用する](https://github.com/dotnet-presentations/blazor-workshop/blob/master/docs/09-progressive-web-app.md#sending-push-notifications)ことを検討してください。

クライアントでプッシュ通知を受信して表示するメカニズムは、サービス ワーカー JavaScript ファイルに実装されているため、Blazor WebAssembly にも依存しません。 例として、[Blazing Pizza ワークショップで使用されている方法](https://github.com/dotnet-presentations/blazor-workshop/blob/master/docs/09-progressive-web-app.md#displaying-notifications)を参照してください。

## <a name="caveats-for-offline-pwas"></a>オフラインの PWA に関する注意事項

すべてのアプリでオフラインでの使用をサポートする必要はありません。 オフライン サポートは、複雑さが大幅に増すうえに、必要なユース ケースに常に関連しているとは限りません。

オフライン サポートは通常、次の場合にのみ関連性を持ちます。

* プライマリ データ ストアがブラウザーに対してローカルである場合。 たとえば、このアプローチは、`localStorage` または [IndexedDB](https://developer.mozilla.org/docs/Web/API/IndexedDB_API) にデータを格納する [IoT](https://en.wikipedia.org/wiki/Internet_of_things) デバイス用の UI があるアプリに関連しています。
* ユーザーがデータ内をオフラインで移動できるように、アプリが各ユーザーに関連するバックエンド API データをフェッチしてキャッシュする大量の作業を実行する場合。 アプリで編集がサポートされている必要がある場合は、変更を追跡し、バックエンドとデータを同期するためのシステムを構築する必要があります。
* ネットワークの状態に関係なく、直ちにアプリが読み込まれることを保証することが、目的の場合。 ネットワークが使用できないことが原因で要求が失敗した場合に、要求の進行状況を表示し、適切に動作するため、バックエンド API 要求を中心にした適切なユーザー エクスペリエンスを実装します。

さらに、オフライン対応の PWA では、さまざまな複雑さに対処する必要があります。 開発者は、以降のセクションの注意点をよく理解しておく必要があります。

### <a name="offline-support-only-when-published"></a>オフライン サポートは公開された場合のみ

開発時には、バックグラウンド更新プロセスを実行せずに、各変更がブラウザーにすぐに反映されることを確認したいことがよくあります。 そのため、Blazor の PWA テンプレートでは、公開された場合にのみオフライン サポートが有効になります。

オフライン対応アプリをビルドする場合、開発環境でアプリをテストするだけでは不十分です。 アプリがさまざまなネットワークの状態にどのように応答するかを理解するには、アプリが公開された状態でテストする必要があります。

### <a name="update-completion-after-user-navigation-away-from-app"></a>ユーザーがアプリから移動した後に更新が完了する

ユーザーがすべてのタブでアプリから移動するまで、更新は完了しません。 「[バックグラウンド更新](#background-updates)」セクションで説明したように、アプリに更新プログラムを配置すると、更新されたサービス ワーカー ファイルがブラウザーによってフェッチされ、更新プロセスが開始されます。

多くの開発者は、この更新が完了しても、ユーザーがすべてのタブでアプリケーションから移動しない限り、この更新が有効に **ならない** ことを意外に思うでしょう。 アプリを表示するタブを更新するだけでは、たとえそれがアプリを表示する唯一のタブであっても、十分では **ありません**。 アプリが完全に閉じられるまで、新しいサービス ワーカーは "*アクティブ化の待機中*" 状態のままになります。 **これは Blazor に固有のものではなく、標準的な Web プラットフォームの動作です。**

これは、サービス ワーカーまたはオフラインでキャッシュされたリソースに対する更新をテストしようとしている開発者にとって、共通の問題となります。 ブラウザーの開発者ツールをチェックインすると、次のような内容が表示されることがあります。

![Google Chrome の [アプリケーション] タブに、アプリのサービス ワーカーが "アクティブ化の待機中" と表示されます。](progressive-web-app/_static/image7.png)

"クライアント" のリスト (アプリを表示しているタブまたはウィンドウ) が空でない限り、ワーカーは待機を続けます。 サービス ワーカーがこれを行う理由は、整合性を保証するためです。 整合性とは、すべてのリソースが同じアトミック キャッシュからフェッチされることを意味します。

変更をテストする際には、前のスクリーンショットに示されているように、"skipWaiting" リンクをクリックし、ページを再度読み込むと便利な場合があります。 ["待機中" のフェーズをスキップして、更新時にすぐにアクティブ化](https://developers.google.com/web/fundamentals/primers/service-workers/lifecycle#skip_the_waiting_phase)するように、サービス ワーカーをコーディングすることで、すべてのユーザーに対してこの動作を自動化できます。 待機中のフェーズをスキップすると、リソースが常に同じキャッシュ インスタンスから一貫してフェッチされることを保証できなくなります。

### <a name="users-may-run-any-historical-version-of-the-app"></a>ユーザーはアプリのあらゆる履歴バージョンを実行する可能性がある

Web 開発者は、ユーザーが、配置された最新バージョンの Web アプリを実行するものと思いがちです。これは、従来の Web 配布モデルでは自然なことでした。 しかし、オフラインファーストの PWA は、ユーザーが必ずしも最新バージョンを実行しているとは限らないネイティブ モバイル アプリに似ています。

「[バックグラウンド更新](#background-updates)」セクションで説明したように、アプリに更新プログラムを配置すると、**既存の各ユーザーは、少なくともあと 1 回のアクセスで以前のバージョンを使用し続けます**。これは、更新がバックグラウンドで行われ、ユーザーがその後アプリから移動するまでアクティブ化されないからです。 また、以前に使用されていたバージョンが、以前に配置したものと必ずしも同じであるとは限りません。 以前のバージョンは、ユーザーが最後に更新を完了した日時に応じて、"*あらゆる*" 履歴バージョンが使用される可能性があります。

これは、アプリのフロントエンドとバックエンドの部分が API 要求のスキーマに関する合意を必要とする場合に問題になることがあります。 すべてのユーザーがアップグレードしたことが確認できるまで、下位互換性のない API スキーマの変更を配置しないでください。 または、ユーザーが互換性のない古いバージョンのアプリを使用できないようにします。 このシナリオ要件は、ネイティブ モバイル アプリの場合と同じです。 サーバー API に重大な変更を配置すると、まだ更新していないユーザーに対してクライアント アプリが中断されます。

可能であれば、バックエンド API に重大な変更を配置しないでください。 これを行う必要がある場合は、[ServiceWorkerRegistration などの標準のサービス ワーカー API](https://developer.mozilla.org/docs/Web/API/ServiceWorkerRegistration) を使用して、アプリが最新であるかどうかを判断し、最新でない場合は使用できないようにすることを検討してください。

### <a name="interference-with-server-rendered-pages"></a>サーバーでレンダリングされるページによる干渉

「[サーバーでレンダリングされるページのサポート](#support-server-rendered-pages)」セクションで説明したように、すべてのナビゲーション要求に対して `/index.html` コンテンツを返すサービス ワーカーの動作をバイパスする場合は、サービス ワーカーのロジックを編集します。

### <a name="all-service-worker-asset-manifest-contents-are-cached-by-default"></a>サービス ワーカー アセット マニフェストのすべてのコンテンツが既定でキャッシュされる

「[アセット キャッシュの制御](#control-asset-caching)」セクションで説明したように、ファイル `service-worker-assets.js` はビルド時に生成され、サービス ワーカーでフェッチおよびキャッシュする必要があるすべてのアセットがリストされます。

既定では、このリストには `wwwroot` に出力されるすべての項目 (外部のパッケージとプロジェクトによって提供されるコンテンツを含む) が含まれるので、コンテンツの数が多くなりすぎないように注意する必要があります。 `wwwroot` ディレクトリに数百万のイメージが含まれている場合、サービス ワーカーはそれらすべてをフェッチおよびキャッシュしようとするため、帯域幅が過剰に消費されて、正常に完了しない可能性が高くなります。

`service-worker.published.js` 内の `onInstall` 関数を編集することで、マニフェストのコンテンツのどのサブセットをフェッチおよびキャッシュするかを制御する任意のロジックを実装します。

### <a name="interaction-with-authentication"></a>認証との相互作用

PWA テンプレートは、認証と組み合わせて使用できます。 オフライン対応 PWA では、ユーザーが最初のネットワーク接続を行うときに認証をサポートすることもできます。

ユーザーがネットワークに接続していない場合は、アクセス トークンを認証したり取得したりすることはできません 既定では、ネットワーク アクセスなしでログイン ページにアクセスしようとすると、"ネットワーク エラー" メッセージが表示されます。 ユーザーを認証したりアクセス トークンを取得したりすることなく、ユーザーがオフラインで便利なタスクを実行できるようにする UI フローを設計する必要があります。 または、ネットワークを使用できないときに適切に失敗するようにアプリを設計することもできます。 このようなシナリオを処理するようにアプリを設計できない場合は、オフライン サポートを有効にしないことをお勧めします。

オンラインとオフラインで使用するように設計されたアプリが再びオンラインになった場合:

* 場合によっては、アプリで新しいアクセス トークンをプロビジョニングする必要があります。
* アプリでは、別のユーザーがサービスにサインインしているかどうかを検出して、オフライン中に行われた操作をユーザーのアカウントに適用できるようにする必要があります。

認証と対話するオフラインの PWA アプリを作成するには:

* <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccountClaimsPrincipalFactory%601> を、最後にサインインしたユーザーを格納し、アプリがオフラインのときに格納されたユーザーを使用するファクトリに置き換えます。
* アプリがオフラインのときに操作をキューに格納し、アプリがオンラインに戻ったときにそれらを適用します。
* サインアウト中に、格納されているユーザーをクリアします。

[`CarChecker`](https://github.com/SteveSandersonMS/CarChecker) サンプル アプリでは、前述の方法が示されています。 アプリの次の部分を参照してください。

* `OfflineAccountClaimsPrincipalFactory` (`Client/Data/OfflineAccountClaimsPrincipalFactory.cs`)
* `LocalVehiclesStore` (`Client/Data/LocalVehiclesStore.cs`)
* `LoginStatus` コンポーネント (`Client/Shared/LoginStatus.razor`)

## <a name="additional-resources"></a>その他の技術情報

* [整合性 PowerShell スクリプトのトラブルシューティング](xref:blazor/host-and-deploy/webassembly#troubleshoot-integrity-powershell-script)
* [認証のための SignalR のクロスオリジンネゴシエーション](xref:blazor/fundamentals/additional-scenarios#signalr-cross-origin-negotiation-for-authentication)
