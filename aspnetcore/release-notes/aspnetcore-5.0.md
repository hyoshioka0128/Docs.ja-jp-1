---
title: ASP.NET Core 5.0 の新機能
author: rick-anderson
description: ASP.NET Core 5.0 の新機能について説明します。
ms.author: riande
ms.custom: mvc
ms.date: 10/29/2020
no-loc:
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
- Kestrel
uid: aspnetcore-5.0
ms.openlocfilehash: 5caa412773bf9c8e3bed5ebc529d48b886de6956
ms.sourcegitcommit: 063a06b644d3ade3c15ce00e72a758ec1187dd06
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98253073"
---
# <a name="whats-new-in-aspnet-core-50"></a>ASP.NET Core 5.0 の新機能

この記事では、ASP.NET Core 5.0 の最も大きな変更点について説明します。また、関連するドキュメントへのリンクも示します。

## <a name="aspnet-core-mvc-and-no-locrazor-improvements"></a>ASP.NET Core MVC と Razor の機能強化

### <a name="model-binding-datetime-as-utc"></a>UTC としての DateTime のモデル バインド

モデル バインドにおいて、UTC 時刻文字列の `DateTime` へのバインドがサポートされるようになりました。 要求に UTC 時刻文字列が含まれている場合は、モデル バインドによって UTC `DateTime` にバインドされます。 たとえば、次の時刻文字列は UTC `DateTime` にバインドされます: `https://example.com/mycontroller/myaction?time=2019-06-14T02%3A30%3A04.0576719Z`

### <a name="model-binding-and-validation-with-c-9-record-types"></a>C# 9 のレコード型を使用したモデル バインドと検証

[C# 9 のレコード型](/dotnet/csharp/whats-new/csharp-9#record-types)を、MVC コントローラーまたは Razor ページのモデル バインドと共に使用することができます。 レコード型は、ネットワーク経由で転送されるデータをモデル化するための優れた方法です。

たとえば、次の `PersonController` では、モデル バインドとフォーム検証で `Person` レコード型が使用されます。

```csharp
public record Person([Required] string Name, [Range(0, 150)] int Age);

public class PersonController
{
   public IActionResult Index() => View();

   [HttpPost]
   public IActionResult Index(Person person)
   {
          // ...
   }
}
```

`Person/Index.cshtml` ファイルは以下の通りです。

```cshtml
@model Person

Name: <input asp-for="Model.Name" />
<span asp-validation-for="Model.Name" />

Age: <input asp-for="Model.Age" />
<span asp-validation-for="Model.Age" />
```

### <a name="improvements-to-dynamicroutevaluetransformer"></a>DynamicRouteValueTransformer の機能強化

ASP.NET Core 3.1 では、カスタム エンドポイントを使用して MVC コントローラーのアクションまたは Razor ページを動的に選択するための方法として、<xref:Microsoft.AspNetCore.Mvc.Routing.DynamicRouteValueTransformer> が導入されました。 ASP.NET Core 5.0 アプリでは、`DynamicRouteValueTransformer` に状態を渡して、選択されたエンドポイントのセットをフィルター処理できます。

### <a name="miscellaneous"></a>その他

* [[Compare]](xref:System.ComponentModel.DataAnnotations.CompareAttribute) 属性は Razor ページ モデルのプロパティに適用できます。
* 本文からバインドされたパラメーターとプロパティは、既定では必須と見なされます。 <!-- Review: How is this different from 3.1
The validation system in .NET Core 3.0 and later treats non-nullable parameters or bound properties as if they had a [Required] attribute.
see https://docs.microsoft.com/aspnet/core/mvc/models/validation?view=aspnetcore-3.1   
-->

## <a name="web-api"></a>Web API

### <a name="openapi-specification-on-by-default"></a>OpenAPI 仕様を規定で有効化

[OpenAPI 仕様](http://spec.openapis.org/oas/v3.0.3)は、HTTP API について記述し、それらを複雑なビジネス プロセスやサード パーティと統合するための業界標準です。 OpenAPI はすべてのクラウド プロバイダーと多くの API レジストリによって幅広くサポートされています。 アプリによって Web API から OpenAPI ドキュメントを出力できれば、これらの API を使用できるさまざまな新しい機会を得られます。 オープンソース プロジェクト [Swashbuckle.AspNetCore](https://www.nuget.org/packages/Swashbuckle.AspNetCore/) の保守管理者との連携により、ASP.NET Core API テンプレートには [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle.AspNetCore) への NuGet の依存関係が含まれています。 Swashbuckle は、OpenAPI ドキュメントを動的に生成する、広く普及しているオープンソース NuGet パッケージです。 Swashbuckle でこれを実行するには、実行時に API コントローラーを調べて OpenAPI ドキュメントを生成するか、または Swashbuckle CLI を使用してビルド時に生成します。

ASP.NET Core 5.0 では、Web API テンプレートによって既定で OpenAPI サポートが有効になります。 OpenAPI を無効にするには:

* コマンドラインから:

    ```dotnetcli
    dotnet new webapi --no-openapi true
    ```
* Visual Studio から: **[Enable OpenAPI support]\(OpenAPI サポートの有効化\)** をオフにします。

Web API プロジェクト用に作成されたすべての *.csproj* ファイルには、[Swashbuckle.AspNetCore](https://www.nuget.org/packages/Swashbuckle.AspNetCore/) NuGet パッケージ参照が含まれます。

```xml
<ItemGroup>
    <PackageReference Include="Swashbuckle.AspNetCore" Version="5.5.1" />
</ItemGroup>
```

テンプレートによって生成されたコードには、OpenAPI ドキュメントの生成をアクティブにするコードが `Startup.ConfigureServices` に含まれます。

[!code-csharp[](~/release-notes/sample/StartupSwagger.cs?name=snippet)]

`Startup.Configure` メソッドによって、次を有効にする Swashbuckle ミドルウェアが追加されます。

* ドキュメント生成プロセス。
* 開発モードでの既定の Swagger UI ページ。

運用環境への発行時に、テンプレートで生成されたコードによって API の説明が誤って公開されることはありません。

[!code-csharp[](~/release-notes/sample/StartupSwagger.cs?name=snippet2)]

#### <a name="azure-api-management-import"></a>Azure API Management のインポート

ASP.NET Core API プロジェクトで OpenAPI が有効になっている場合、Visual Studio 2019 バージョン 16.8 以降の発行では、発行フローに追加の手順が自動的に提示されます。 [Azure API Management](xref:tutorials/publish-to-azure-api-management-using-vs) を使用する開発者は、発行フロー中に、API を Azure API Management に自動的にインポートすることを選択できます。

![Azure API Management のインポートと発行](~/release-notes/static/publish-to-apim.png)

### <a name="better-launch-experience-for-web-api-projects"></a>Web API プロジェクトの起動エクスペリエンスの向上

OpenAPI が既定で有効になっている場合、Web API 開発者のアプリ起動エクスペリエンス (F5) が大幅に改善されます。 ASP.NET Core 5.0 では、Web API テンプレートが Swagger UI ページを読み込むように事前構成されています。 Swagger UI ページには、発行された API 用に追加されるドキュメントと、1 回のクリックで API をテストする機能の両方が用意されています。

![swagger/index.html の表示](~/release-notes/static/swagger-ui-page-rc1.png)

## Blazor

### <a name="performance-improvements"></a>パフォーマンスの向上

.NET 5 では、複雑な UI レンダリングと JSON シリアル化に特に焦点を当てて、Blazor WebAssembly ランタイムのパフォーマンスが大幅に改善されました。 Microsoft によるパフォーマンス テストでは、.NET 5 の Blazor WebAssembly はほとんどのシナリオで 2 倍から 3 倍高速になりました。 詳細については、[ASP.NET ブログ:.NET 5 リリース候補 1 における ASP.NET Core の更新](https://devblogs.microsoft.com/aspnet/asp-net-core-updates-in-net-5-release-candidate-1/#blazor-webassembly-performance-improvements)に関するページを参照してください。

### <a name="css-isolation"></a>CSS の分離

Blazor では、特定のコンポーネントにスコープ設定された CSS スタイルの定義がサポートされるようになりました。 コンポーネント固有の CSS スタイルを使用すると、アプリのスタイルを把握しやすくなり、グローバル スタイルにおける意図しない副作用を回避しやすくなります。 詳細については、 <xref:blazor/components/css-isolation> を参照してください。

### <a name="new-inputfile-component"></a>新しい `InputFile` コンポーネント

`InputFile` コンポーネントを使用すると、アップロード用にユーザーが選択した 1 つ以上のファイルを読み取ることができます。 詳細については、 <xref:blazor/file-uploads> を参照してください。

### <a name="new-inputradio-and-inputradiogroup-components"></a>新しい `InputRadio` および `InputRadioGroup` コンポーネント

Blazor に、ラジオ ボタン グループへのデータ バインドを簡略化する、統合された検証機能を備えた `InputRadio` および `InputRadioGroup` コンポーネントが組み込まれています。 詳細については、 <xref:blazor/forms-validation> を参照してください。

### <a name="component-virtualization"></a>コンポーネントの仮想化

Blazor フレームワークに組み込まれている仮想化サポートを使用して、コンポーネント レンダリングの認識されるパフォーマンスを向上させます。 詳細については、 <xref:blazor/components/virtualization> を参照してください。

### <a name="ontoggle-event-support"></a>`ontoggle` イベントのサポート

Blazor イベントで `ontoggle` DOM イベントがサポートされるようになりました。 詳細については、 <xref:blazor/components/event-handling#event-argument-types> を参照してください。

### <a name="set-ui-focus-in-no-locblazor-apps"></a>Blazor アプリで UI フォーカスを設定する

要素参照で便利なメソッド `FocusAsync` を使用して、UI フォーカスをその要素に設定できます。 詳細については、 <xref:blazor/components/event-handling#focus-an-element> を参照してください。

### <a name="custom-validation-class-attributes"></a>カスタム検証クラス属性

カスタム検証クラス名は、Bootstrap などの CSS フレームワークと統合する場合に便利です。 詳細については、 <xref:blazor/forms-validation#custom-validation-class-attributes> を参照してください。

### <a name="iasyncdisposable-support"></a>IAsyncDisposable のサポート

Blazor コンポーネントで、割り当てられたリソースの非同期リリース用に <xref:System.IAsyncDisposable> インターフェイスがサポートされるようになりました。

### <a name="javascript-isolation-and-object-references"></a> JavaScript の分離とオブジェクト参照

Blazor により、標準 [JavaScript モジュール](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules)で JavaScript の分離が有効にされます。 詳細については、 <xref:blazor/call-javascript-from-dotnet#blazor-javascript-isolation-and-object-references> を参照してください。

### <a name="form-components-support-display-name"></a>フォーム コンポーネントでの表示名のサポート

次の組み込みコンポーネントでは、`DisplayName` パラメーターを使用した表示名がサポートされています。

* `InputDate`
* `InputNumber`
* `InputSelect`

詳細については、「<xref:blazor/forms-validation#display-name-support>」を参照してください。

### <a name="catch-all-route-parameters"></a>キャッチオールのルート パラメーター

複数のフォルダー境界にまたがるパスをキャプチャするキャッチオール ルート パラメーターが、コンポーネントでサポートされます。 詳細については、 <xref:blazor/fundamentals/routing#catch-all-route-parameters> を参照してください。

### <a name="debugging-improvements"></a>デバッグの機能強化

ASP.NET Core 5.0 では Blazor WebAssembly アプリのデバッグが改善されました。 さらに、Visual Studio for Mac でデバッグがサポートされるようになりました。 詳細については、 <xref:blazor/debug> を参照してください。

### <a name="microsoft-no-locidentity-v20-and-msal-v20"></a>Microsoft Identity v2.0 および MSAL v2.0

Blazor セキュリティで Microsoft Identity v2.0 ([`Microsoft.Identity.Web`](https://www.nuget.org/packages/Microsoft.Identity.Web) と [`Microsoft.Identity.Web.UI`](https://www.nuget.org/packages/Microsoft.Identity.Web.UI)) および MSAL v2.0 が使用されるようになりました。 詳細については、[Blazor セキュリティと Identity ノード](xref:blazor/security/index)に関するトピックを参照してください。

### <a name="protected-browser-storage-for-no-locblazor-server"></a>Blazor Server の保護されたブラウザー ストレージ

Blazor Server アプリでは、ASP.NET Core データ保護を使用して改ざんから保護されているブラウザーにアプリの状態を格納するための、組み込みサポートを使用できるようになりました。 データは、ローカルのブラウザー ストレージまたはセッション ストレージに格納できます。 詳細については、 <xref:blazor/state-management> を参照してください。

### <a name="no-locblazor-webassembly-prerendering"></a>Blazor WebAssembly のプリレンダリング

コンポーネントの統合がホスティング モデル全体で向上し、Blazor WebAssembly アプリがサーバーで出力をプリレンダリングできるようになりました。 <!-- UNCOMMENT AFTER https://github.com/dotnet/AspNetCore.Docs/pull/19887 MERGES: For more information, see <xref:blazor/components/integrate-components-into-razor-pages-and-mvc-apps> and <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>. -->

### <a name="trimminglinking-improvements"></a>トリミングとリンクの機能強化

Blazor WebAssembly では、ビルド中に中間言語 (IL) のトリミングとリンクが実行され、アプリの出力アセンブリから不要な IL がトリミングされます。 ASP.NET Core 5.0 リリースでは、Blazor WebAssembly により、追加の構成オプションを備えた改善されたトリミングが実行されます。 詳細については、「<xref:blazor/host-and-deploy/configure-trimmer>」と「[トリミングのオプション](/dotnet/core/deploying/trimming-options)」を参照してください。

### <a name="browser-compatibility-analyzer"></a>ブラウザー互換性アナライザー

Blazor WebAssembly アプリは完全な .NET API 領域を対象としていますが、ブラウザー サンドボックスの制約により、すべての .NET API が WebAssembly でサポートされているわけではありません。 サポートされていない API は、WebAssembly で実行すると <xref:System.PlatformNotSupportedException> がスローされます。 開発者が、アプリのターゲット プラットフォームでサポートされていない API をアプリで使用すると、プラットフォーム互換性アナライザーから警告を受け取ります。 詳細については、 <xref:blazor/components/class-libraries#browser-compatibility-analyzer-for-blazor-webassembly> を参照してください。

### <a name="lazy-load-assemblies"></a>アセンブリの遅延読み込み

Blazor WebAssembly アプリの起動時のパフォーマンスは、一部のアプリケーション アセンブリの読み込みを必要になるまで延期することで改善できます。 詳細については、 <xref:blazor/webassembly-lazy-load-assemblies> を参照してください。

### <a name="updated-globalization-support"></a>グローバリゼーション サポートの更新

International Components for Unicode (ICU) に基づき Blazor WebAssembly でグローバリゼーション サポートを利用できます。 詳細については、 <xref:blazor/globalization-localization> を参照してください。

## <a name="grpc"></a>gRPC

[gRPC](https://grpc.io/) では、多くのパフォーマンス改善が行われています。 詳細については、「[.NET 5 での gRPC のパフォーマンスの向上](https://devblogs.microsoft.com/aspnet/grpc-performance-improvements-in-net-5/)」を参照してください。

gRPC の詳細については、「<xref:grpc/index>」を参照してください。

## SignalR

### <a name="no-locsignalr-hub-filters"></a>SignalR ハブ フィルター

SignalR ハブ フィルター (ASP.NET SignalR のハブ パイプラインと呼ばれます) は、ハブ メソッドが呼び出される前と後にコードを実行できる機能です。 ハブ メソッドが呼び出される前と後のコードの実行は、ミドルウェアが HTTP 要求の前と後にコードを実行する機能に似ています。 一般的な使用方法としては、ログ記録、エラー処理、引数の検証などがあります。

詳細については、「[ASP.NET Core SignalR のハブ フィルターを使用する](xref:signalr/hub-filters)」を参照してください。

### <a name="no-locsignalr-parallel-hub-invocations"></a>SignalR 並列ハブ呼び出し

ASP.NET Core SignalR では、並列ハブ呼び出しを処理できるようになりました。 既定の動作を変更して、クライアントが一度に複数のハブ メソッドを呼び出せるようにすることができます。

[!code-csharp[](~/release-notes/sample/StartupSignalRhubs.cs?name=snippet)]

### <a name="added-messagepack-support-in-no-locsignalr-java-client"></a>SignalR Java クライアントへの Messagepack サポートの追加

新しいパッケージ [com.microsoft.signalr.messagepack](https://mvnrepository.com/artifact/com.microsoft.signalr.messagepack) によって、SignalR Java クライアントに MessagePack サポートが追加されます。 MessagePack ハブ プロトコルを使用するには、接続ビルダーに `.withHubProtocol(new MessagePackHubProtocol())` を追加します。

```java
HubConnection hubConnection = HubConnectionBuilder.create(
                           "http://localhost:53353/MyHub")
               .withHubProtocol(new MessagePackHubProtocol())
               .build();
```

<!--
See [Update SignalR code](xref:migration/31-to-50#signalr) for migration instructions.
-->

## Kestrel

* 構成を使用して再読み込み可能なエンドポイント: Kestrel では、`reloadOnChange` パラメーターが `true` の場合、[KestrelServerOptions.Configure](xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Configure%2A) に渡された構成の変更を検出し、アプリケーションを再起動することなく、既存のエンドポイントからバインド解除して、新しいエンドポイントにバインドすることができます。 既定では、<xref:Microsoft.Extensions.Hosting.GenericHostBuilderExtensions.ConfigureWebHostDefaults%2A> または <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder%2A> を使用する場合、Kestrel は `reloadOnChange` が有効な ["Kestrel"](https://github.com/dotnet/aspnetcore/blob/7e9e03b70124784b1de5564c573bd65cdaccbfcc/src/DefaultBuilder/src/WebHost.cs#L226) 構成サブセクションにバインドされます。 アプリでは、`KestrelServerOptions.Configure` を手動で呼び出す際に `reloadOnChange: true` を渡して、再読み込み可能なエンドポイントを取得する必要があります。
* HTTP/2 応答ヘッダーの機能強化。 詳細については、次のセクションの「[パフォーマンスの向上](#performance-improvements)」を参照してください。
* ソケット転送における追加のエンドポイントの種類のサポート:<xref:System.Net.Sockets?displayProperty=nameWithType> に導入された新しい API に加えて、[Kestrel](xref:fundamentals/servers/kestrel) の既定のソケット転送で、既存のファイル ハンドルと Unix ドメイン ソケットの両方にバインドできるようになります。 既存のファイル ハンドルへのバインドのサポートによって、`libuv` 転送を必要とせずに、既存の `Systemd` 統合を使用できるようになります。
* Kestrel でのカスタム ヘッダーのデコード:アプリでは、既定の UTF-8 の代わりに、ヘッダー名に基づいて受信ヘッダーを解釈するために使用する <xref:System.Text.Encoding> を指定できます。 使用するエンコードを指定するには、 <!--<xref:Microsoft.AspNetCore.Server.Kestrel.KestrelServerOptions.RequestHeaderEncodingSelector> --> `Microsoft.AspNetCore.Server.Kestrel.KestrelServerOptions.RequestHeaderEncodingSelector` プロパティを設定します。
 
  ```csharp
  public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel(options =>
            {
                options.RequestHeaderEncodingSelector = encoding =>
                {
                      return encoding switch
                      {
                          "Host" => System.Text.Encoding.Latin1,
                          _ => System.Text.Encoding.UTF8,
                      };
                };
            });
            webBuilder.UseStartup<Startup>();
        });
  ```

### <a name="no-lockestrel-endpoint-specific-options-via-configuration"></a>構成を使用した Kestrel エンドポイント固有のオプション

[構成](xref:fundamentals/configuration/index)を使用して Kestrel のエンドポイント固有のオプションを構成するためのサポートが追加されました。 エンドポイント固有の構成には、次が含まれます。

* 使用される HTTP プロトコル
* 使用される TLS プロトコル
* 選択される証明書
* クライアント証明書モード

構成によって、指定したサーバー名に基づいてどの証明書が選択されるかを指定できます。 サーバー名は、クライアントによって示される、TLS プロトコルの Server Name Indication (SNI) 拡張機能の一部です。 Kestrel の構成では、ホスト名のワイルドカード プレフィックスもサポートされています。

構成ファイルを使用してエンドポイント固有の設定を指定する方法を次の例に示します。

```json
{
  "Kestrel": {
    "Endpoints": {
      "EndpointName": {
        "Url": "https://*",
        "Sni": {
          "a.example.org": {
            "Protocols": "Http1AndHttp2",
            "SslProtocols": [ "Tls11", "Tls12"],
            "Certificate": {
              "Path": "testCert.pfx",
              "Password": "testPassword"
            },
            "ClientCertificateMode" : "NoCertificate"
          },
          "*.example.org": {
            "Certificate": {
              "Path": "testCert2.pfx",
              "Password": "testPassword"
            }
          },
          "*": {
            // At least one sub-property needs to exist per
            // SNI section or it cannot be discovered via
            // IConfiguration
            "Protocols": "Http1",
          }
        }
      }
    }
  }
}
```

Server Name Indication (SNI) は、SSL ネゴシエーションの一部として仮想ドメインを含めるための TLS 拡張機能です。 これは実質的に、仮想ドメイン名 (またはホスト名) を使用してネットワーク エンドポイントを識別できることを意味します。

## <a name="performance-improvements"></a>パフォーマンスの向上

### <a name="http2"></a>HTTP/2

* HTTP/2 コード パスでの割り当ての大幅な削減。
* [Kestrel](xref:fundamentals/servers/kestrel)での HTTP/2 応答ヘッダーの [HPack 動的圧縮](https://tools.ietf.org/html/rfc7541)のサポート。 詳細については、「[ヘッダー テーブルのサイズ](xref:fundamentals/servers/kestrel/options#header-table-size)」と「[HPACK: HTTP/2 のサイレント機能](https://blog.cloudflare.com/hpack-the-silent-killer-feature-of-http-2/)」を参照してください。
* HTTP/2 PING フレームの送信:HTTP/2 には、アイドル状態の接続がまだ機能していることを確認するために PING フレームを送信するメカニズムがあります。 実行可能な接続であることの確認は、アイドル状態であることが多いが間欠的にアクティビティが発生する、長期間存在するストリームを使用する場合に特に便利です。たとえば、gRPC ストリームなどです。 アプリでは、<xref:Microsoft.AspNetCore.Server.Kestrel.KestrelServerOptions> に制限を設定することによって、[Kestrel](xref:fundamentals/servers/kestrel) で定期的に PING フレームを送信できます。

   ```csharp
   public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel(options =>
            {
                options.Limits.Http2.KeepAlivePingInterval =
                                               TimeSpan.FromSeconds(10);
                options.Limits.Http2.KeepAlivePingTimeout =
                                               TimeSpan.FromSeconds(1);
            });
            webBuilder.UseStartup<Startup>();
        });
   ```
   <!-- review: KeepAlivePingInterval not found in RC1. Try testing with RC1. See https://github.com/dotnet/aspnetcore/pull/22565/files see C:/Users/riande/source/repos/WebApplication128/WebApplication128 -->

### <a name="containers"></a>Containers

.NET 5.0 より前のバージョンでは、ASP.NET Core アプリの *Dockerfile* をビルドして発行するには、.NET Core SDK と ASP.NET Core のイメージ全体をプルする必要がありました。 このリリースでは、SDK イメージをプルするバイトが削減され、ASP.NET Core イメージ用にプルされるバイトの大部分が削除されます。 詳細については、[こちらの GitHub イシューのコメント](https://github.com/dotnet/dotnet-docker/issues/1814#issuecomment-625294750)を参照してください

## <a name="authentication-and-authorization"></a>認証と承認

### <a name="azure-active-directory-authentication-with-microsoftno-locidentityweb"></a>Microsoft.Identity.Web での Azure Active Directory 認証

ASP.NET Core のプロジェクト テンプレートが <xref:Microsoft.Identity.Web?displayProperty=fullName> と統合され、[Azure Active Directory](/azure/active-directory/fundamentals/active-directory-whatis) (Azure AD) での認証を処理できるようになりました。 [Microsoft.Identity.Web パッケージ](https://www.nuget.org/packages/Microsoft.Identity.Web/)には次が備わっています。

* Azure AD を使用した認証のエクスペリエンスが向上します。
* [Microsoft Graph](/graph/overview) など、ユーザーに代わって Azure リソースにアクセスするための簡単な方法。 [Microsoft.Identity.Web のサンプル](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2)を参照してください。これは基本的なログインから始まり、マルチテナント、Azure API の使用、Microsoft Graph の使用、独自の API の保護に進んでいきます。 `Microsoft.Identity.Web` は .NET 5 と共に使用できます。

### <a name="allow-anonymous-access-to-an-endpoint"></a>エンドポイントへの匿名アクセスを許可する

`AllowAnonymous` 拡張メソッドを使用すると、エンドポイントへの匿名アクセスを許可できます。

[!code-csharp[](~/release-notes/sample/StartupAnonEndpoint.cs?name=snippet)]

### <a name="custom-handling-of-authorization-failures"></a>認可失敗のカスタム処理

[認可](xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization%2A)[ミドルウェア](xref:fundamentals/middleware/index)によって呼び出される新しい [IAuthorizationMiddlewareResultHandler](https://github.com/dotnet/aspnetcore/blob/v5.0.0-rc.1.20451.17/src/Security/Authorization/Policy/src/IAuthorizationMiddlewareResultHandler.cs) インターフェイスにより、認可失敗のカスタム処理がより簡単になりました。 既定の実装は変わりませんが、カスタム ハンドラーを依存関係の挿入に登録できます。これにより、認可が失敗した理由に基づくカスタム HTTP 応答が可能になります。 `IAuthorizationMiddlewareResultHandler` の使用方法を示す[こちらのサンプル](https://github.com/dotnet/aspnetcore/blob/master/src/Security/samples/CustomAuthorizationFailureResponse/Authorization/SampleAuthorizationMiddlewareResultHandler.cs)を参照してください。

### <a name="authorization-when-using-endpoint-routing"></a>エンドポイント ルーティングを使用する場合の認可

エンドポイント ルーティングを使用する場合の認可では、エンドポイントのインスタンスではなく `HttpContext` を受信するようになりました。 これにより、認可ミドルウェアで、<xref:Microsoft.AspNetCore.Http.Endpoint> クラスではアクセスできなかった `RouteData` や `HttpContext` のその他のプロパティにアクセスできるようになります。 エンドポイントは、[context.GetEndpoint](xref:Microsoft.AspNetCore.Http.EndpointHttpContextExtensions.GetEndpoint%2A) を使用してコンテキストからフェッチできます。

### <a name="role-based-access-control-with-kerberos-authentication-and-ldap-on-linux"></a>Linux での Kerberos 認証と LDAP を使用したロールベースのアクセス制御

「[Kerberos 認証とロールベースのアクセス制御 (RBAC)](xref:security/authentication/windowsauth#rbac)」を参照してください

## <a name="api-improvements"></a>API の機能強化

### <a name="json-extension-methods-for-httprequest-and-httpresponse"></a>HttpRequest と HttpResponse 用の JSON 拡張メソッド

新しい <xref:System.Net.Http.Json.HttpContentJsonExtensions.ReadFromJsonAsync%2A> および `WriteAsJsonAsync` 拡張メソッドを使用して、`HttpRequest` および `HttpResponse` に対する JSON データの読み取りと書き込みを行うことができます。 これらの拡張メソッドでは、[System.Text.Json](xref:System.Text.Json) シリアライザーを使用して JSON データが処理されます。 新しい `HasJsonContentType` 拡張メソッドを使用すれば、要求に JSON コンテンツ タイプがあるかどうかを確認することもできます。

JSON 拡張メソッドを [エンドポイント ルーティング](xref:fundamentals/routing)と組み合わせることで、"***コードへのルーティング** _" と呼ばれるプログラミングのスタイルで JSON API を作成できます。 これは、基本的な JSON API を軽量な方法で作成したい開発者向けの新しいオプションです。 たとえば、少数のエンドポイントのみを持つ Web アプリでは、ASP.NET Core MVC の完全な機能ではなく、コードへのルーティングを使用することができます。

```csharp
endpoints.MapGet("/weather/{city:alpha}", async context =>
{
    var city = (string)context.Request.RouteValues["city"];
    var weather = GetFromDatabase(city);

    await context.Response.WriteAsJsonAsync(weather);
});
```

新しい JSON 拡張メソッドとコードへのルーティングについて詳しくは、[こちらの .NET ショー](https://channel9.msdn.com/Shows/On-NET/ASPNET-Core-Series-Route-to-Code)を参照してください。

### <a name="systemdiagnosticsactivity"></a>System.Diagnostics.Activity

<xref:System.Diagnostics.Activity?displayProperty=fullName> の既定の形式が W3C 形式になりました。 これにより、ASP.NET Core での分散トレースのサポートが、既定でより多くのフレームワークと相互運用できるようになります。

### <a name="frombodyattribute"></a>FromBodyAttribute

<xref:Microsoft.AspNetCore.Mvc.FromBodyAttribute> で、次のパラメーターまたはプロパティを省略可能と見なすことができるオプションを構成できるようになりました。

```csharp
public IActionResult Post([FromBody(EmptyBodyBehavior = EmptyBodyBehavior.Allow)]
                          MyModel model)
{
    ...
}
```

## <a name="miscellaneous-improvements"></a>その他の機能強化

ASP.NET Core アセンブリに [Null 許容の注釈](/dotnet/csharp/nullable-references#attributes-describe-apis)を適用する作業が開始されました。 .NET 5 フレームワークの一般的なパブリック API サーフェイスのほとんどに注釈が付けられる予定です。 <!-- Review: what's the impact of this? How does it work? Need more info.  Check the link I added -->

### <a name="control-startup-class-activation"></a>Startup クラスのアクティブ化を制御する

`Startup` クラスのアクティブ化を制御するためのファクトリ メソッドをアプリで提供できるようにする、新しい <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseStartup%2A> オーバーロードが追加されました。 `Startup` クラスのアクティブ化の制御は、ホストと共に初期化される `Startup` に追加のパラメーターを渡す場合に便利です。

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var logger = CreateLogger();
        var host = Host.CreateDefaultBuilder()
            .ConfigureWebHost(builder =>
            {
                builder.UseStartup(context => new Startup(logger));
            })
            .Build();

        await host.RunAsync();
    }
}
```

### <a name="auto-refresh-with-dotnet-watch"></a>dotnet watch での自動更新

.NET 5 では、ASP.NET Core プロジェクトで [dotnet watch](xref:tutorials/dotnet-watch) を実行すると、既定のブラウザーが起動され、コードに変更が加えられるとブラウザーが自動的に更新されます。 つまり、次のことが可能になります。

_ テキスト エディターで ASP.NET Core プロジェクトを開きます。
* `dotnet watch` を実行します。
* ツールによってアプリのリビルド、再起動、再読み込みを処理しながら、コードの変更に集中します。

### <a name="console-logger-formatter"></a>コンソール ロガー フォーマッタ

`Microsoft.Extensions.Logging` ライブラリのコンソール ログ プロバイダーが改善されました。 開発者は、カスタム `ConsoleFormatter` を実装して、コンソール出力の書式設定と色付けを完全に制御できるようになりました。 フォーマッタ API により、VT-100 エスケープ シーケンスのサブセットを実装することによって、豊富な書式設定が可能になります。 VT-100 は、最新のターミナルのほとんどでサポートされています。 コンソール ロガーでは、サポートされていないターミナルでエスケープ シーケンスを解析できるため、開発者はすべてのターミナル用に 1 つのフォーマッタを作成できます。

### <a name="json-console-logger"></a>JSON コンソール ロガー

カスタム フォーマッタのサポートに加えて、構造化された JSON ログをコンソールに出力する組み込みの JSON フォーマッタも追加されました。 次のコードは、既定のロガーから JSON に切り替える方法を示しています。

[!code-csharp[](~/release-notes/sample/ProgramJsonLog.cs?name=snippet)]

コンソールに出力されるログ メッセージは JSON 形式になります。

```json
{
  "EventId": 0,
  "LogLevel": "Information",
  "Category": "Microsoft.Hosting.Lifetime",
  "Message": "Now listening on: https://localhost:5001",
  "State": {
    "Message": "Now listening on: https://localhost:5001",
    "address": "https://localhost:5001",
    "{OriginalFormat}": "Now listening on: {address}"
  }
}
```
