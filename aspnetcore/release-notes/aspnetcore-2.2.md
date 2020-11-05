---
title: ASP.NET Core 2.2 の新機能
author: rick-anderson
description: ASP.NET Core 2.2 の新機能について説明します。
ms.author: riande
ms.custom: mvc
ms.date: 12/05/2019
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
uid: aspnetcore-2.2
ms.openlocfilehash: 085a26d508af5bdce506ab923a3256046b3894ab
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/17/2020
ms.locfileid: "90722489"
---
# <a name="whats-new-in-aspnet-core-22"></a>ASP.NET Core 2.2 の新機能

この記事では、ASP.NET Core 2.2 の最も大きな変更点について説明します。また、その変更点のドキュメントへのリンクも示します。

## <a name="openapi-analyzers--conventions"></a>OpenAPI のアナライザーと規則

OpenAPI (旧称 Swagger) は、REST API を記述するために、言語に関係なく使用できる仕様です。 OpenAPI エコシステムには、この仕様を使用してクライアント コードの検出、テスト、および作成を行うことができるツールがあります。 ASP.NET Core MVC での OpenAPI ドキュメントの生成と視覚化のサポートは、[NSwag](https://github.com/RicoSuter/NSwag) や [Swashbuckle.AspNetCore](https://github.com/domaindrivendev/Swashbuckle.AspNetCore) などのコミュニティ主導のプロジェクトを介して提供されています。 ASP.NET Core 2.2 では、OpenAPI ドキュメントを作成するためのツールとランタイム エクスペリエンスが改善されています。

詳細については、次のリソースを参照してください。

* <xref:web-api/advanced/analyzers>
* <xref:web-api/advanced/conventions>
* [ASP.NET Core 2.2.0-preview1:OpenAPI のアナライザーと規則](https://blogs.msdn.microsoft.com/webdev/2018/08/23/asp-net-core-2-20-preview1-open-api-analyzers-conventions/)

## <a name="problem-details-support"></a>問題の詳細のサポート

ASP.NET Core 2.1 では、HTTP 応答でエラーの詳細を伝達するための [RFC 7807](https://tools.ietf.org/html/rfc7807) 仕様に基づいて、`ProblemDetails` が導入されました。 2\.2 で、`ProblemDetails` は、属性が `ApiControllerAttribute` のコントローラーのクライアント エラー コードに対する標準の応答です。 クライアント エラー状態コード (4xx) を返している `IActionResult` は、`ProblemDetails` の本文を返すようになりました。 この結果には、要求ログを使用してエラーを関連付けるために使用できる相関関係 ID も含まれています。 クライアント エラーの場合、`ProducesResponseType` の既定では `ProblemDetails` が応答の種類として使用されます。 これは、NSwag または Swashbuckle.AspNetCore を使用して生成される OpenAPI/Swagger 出力にドキュメント化されます。

## <a name="endpoint-routing"></a>エンドポイント ルーティング

ASP.NET Core 2.2 では、要求のディスパッチを改善するために新しい*エンドポイント ルーティング* システムを使用しています。 この変更には、新しいリンク生成 API メンバーとルート パラメーター変換機能が含まれています。

詳細については、次のリソースを参照してください。

* [2.2 のエンドポイント ルーティング](https://blogs.msdn.microsoft.com/webdev/2018/08/27/asp-net-core-2-2-0-preview1-endpoint-routing/)
* [ルート パラメーターの変換機能](https://www.hanselman.com/blog/ASPNETCore22ParameterTransformersForCleanURLGenerationAndSlugsInRazorPagesOrMVC.aspx) (**ルーティング**に関するセクションを参照してください)
* [IRouter ベースのルーティングとエンドポイントベースのルーティングの違い](xref:fundamentals/routing?view=aspnetcore-2.2#differences-from-earlier-versions-of-routing)

## <a name="health-checks"></a>正常性チェック

新しい正常性チェック サービスを使用すると、Kubernetes などの正常性チェックが必要な環境で ASP.NET Core を簡単に使用できます。 正常性チェックには、ミドルウェアと、`IHealthCheck` の抽象化とサービスを定義する一連のライブラリが含まれています。

正常性チェックは、システムが正常に要求に応答しているかどうかを迅速に判断するために、コンテナー オーケストレーターまたはロード バランサーによって使用されます。 正常性に問題が確認された場合、コンテナー オーケストレーターは実行中の展開を停止したり、コンテナーを再起動したりします。 ロード バランサーは、正常性チェックに対して、障害が発生したサービスのインスタンスからトラフィックをルーティングすることで応答する可能性があります。

正常性チェックは、監視システムから使用される HTTP エンドポイントとして、アプリケーションによって公開されています。 正常性チェックは、さまざまなリアルタイム監視シナリオや監視システムに合わせて構成できます。 正常性チェック サービスは、[BeatPulse プロジェクト](https://github.com/Xabaril/BeatPulse)と統合されています。 そのため、数多くの一般的なシステムや依存関係のチェックを簡単に追加することができます。

詳細については、「[Health checks in ASP.NET Core](xref:host-and-deploy/health-checks)」(ASP.NET Core の正常性チェック) を参照してください。

## <a name="http2-in-kestrel"></a>Kestrel の HTTP/2

ASP.NET Core 2.2 では HTTP/2 のサポートが追加されました。

HTTP/2 は HTTP プロトコルのメジャー リビジョンです。 HTTP/2 には次のような注目すべき機能があります。

* ヘッダーの圧縮をサポート。
* 単一の接続での完全に多重化されたストリーム。

HTTP/2 は HTTP のセマンティクス (HTTP ヘッダー、メソッドなど) を維持していますが、データのフレーム化方法やクライアントとサーバー間での送信方法に関して、HTTP/1.x からの破壊的変更があります。

フレーム化に関するこの変更の結果、サーバーとクライアントは、使用されているプロトコル バージョンのネゴシエートが必要になりました。 アプリケーション層プロトコル ネゴシエーション (ALPN) は、TLS の拡張機能です。これにより、サーバーとクライアントは、その TLS のハンドシェイクの一部として使用されているプロトコルのバージョンをネゴシエートすることができます。 プロトコルに関してサーバーとクライアント間に事前の情報を持たせることはできますが、すべての主要なブラウザーは、HTTP/2 接続を確立する唯一の方法として ALPN をサポートしています。

詳細については、「[HTTP/2 のサポート](xref:fundamentals/servers/index?view=aspnetcore-2.2#http2-support)」を参照してください。

## <a name="kestrel-configuration"></a>Kestrel の構成

以前のバージョンの ASP.NET Core では、`UseKestrel` を呼び出して Kestrel のオプションを構成します。 2\.2 で Kestrel のオプションを構成するには、ホスト ビルダー上で `ConfigureKestrel` を呼び出します。 この変更により、インプロセス ホスティングの `IServer` の登録順に関する問題が解決されます。 詳細については、次のリソースを参照してください。

* [UseIIS の競合を軽減する](https://github.com/aspnet/KestrelHttpServer/issues/2760)
* [ConfigureKestrel を使用して Kestrel サーバーのオプションを構成する](xref:fundamentals/servers/kestrel?view=aspnetcore-2.2#how-to-use-kestrel-in-aspnet-core-apps)

## <a name="iis-in-process-hosting"></a>IIS のインプロセス ホスティング

以前のバージョンの ASP.NET Core では、IIS はリバース プロキシとして機能しています。 2\.2 では、ASP.NET Core モジュールで CoreCLR を起動し、IIS worker プロセス (*w3wp.exe*) 内でアプリをホストすることができます。 インプロセス ホスティングを IIS で実行すると、パフォーマンスと診断機能が向上します。

詳細については、[IIS のインプロセス ホスティング](xref:host-and-deploy/aspnet-core-module?view=aspnetcore-2.2#in-process-hosting-model)に関する記事を参照してください。

## <a name="no-locsignalr-java-client"></a>SignalR Java クライアント

ASP.NET Core 2.2 には SignalR 用の Java クライアントが導入されました。 このクライアントは、Android アプリを含め、Java コードから ASP.NET Core SignalR サーバーへの接続をサポートしています。

詳細については、「[ASP.NET Core SignalR Java クライアント](../signalr/java-client.md?view=aspnetcore-2.2)」を参照してください。

## <a name="cors-improvements"></a>CORS の機能強化

以前のバージョンの ASP.NET Core では、`CorsPolicy.Headers` に構成されている値に関係なく、CORS ミドルウェアから `Accept`、`Accept-Language`、`Content-Language`、`Origin` ヘッダーを送信できます。 2\.2 では、CORS ミドルウェア ポリシーの一致は、`Access-Control-Request-Headers` で送信されたヘッダーが `WithHeaders` に示されているヘッダーと正確に一致する場合にのみ可能です。

詳細については、[CORS ミドルウェア](xref:security/cors?view=aspnetcore-2.2#set-the-allowed-request-headers)に関する記事を参照してください。

## <a name="response-compression"></a>応答圧縮

ASP.NET Core 2.2 は、[Brotli 圧縮形式](https://tools.ietf.org/html/rfc7932)で応答を圧縮できます。

詳細については、[応答圧縮ミドルウェアによる Brotli 圧縮のサポート](xref:performance/response-compression?view=aspnetcore-2.2#brotli-compression-provider)に関する記事を参照してください。

## <a name="project-templates"></a>プロジェクト テンプレート

ASP.NET Core Web プロジェクト テンプレートは、[Bootstrap 4](https://getbootstrap.com/docs/4.1/migration/) と [Angular 6](https://blog.angular.io/version-6-of-angular-now-available-cc56b0efa7a4) に更新されました。 新しい外観は視覚的にシンプルになり、アプリの重要な構造を簡単に把握できるようになりました。

![ホームまたはインデックス ページ](~/tutorials/razor-pages/razor-pages-start/_static/home2.2.png)

## <a name="validation-performance"></a>検証のパフォーマンス

MVC の検証システムは、拡張性と柔軟性を考慮して設計されており、特定のモデルにどの検証コントロールが適用されるかを要求単位で判断することができます。 そのため、複雑な検証プロバイダーを作成する場合に最適です。 ただし、最も一般的なケースでは、アプリケーションは組み込みの検証コントロールのみを使用し、このような追加の柔軟性は不要です。 組み込みの検証コントロールには、[Required]、[StringLength] などの DataAnnotations や、`IValidatableObject` が含まれています。

ASP.NET Core 2.2 では、特定のモデル グラフに検証が不要と判断された場合、MVC で検証を省略することができます。 検証をスキップすると、検証モデルに検証コントロールを含めることができない場合や含めない場合に大幅に改善されます。 これには、プリミティブ型のコレクション (`byte[]`、`string[]`、`Dictionary<string, string>` など) のオブジェクトや、多数の検証コントロールがない複雑なオブジェクト グラフが含まれます。

## <a name="http-client-performance"></a>HTTP クライアントのパフォーマンス

ASP.NET Core 2.2 では、接続プールのロック競合を減らすことで、`SocketsHttpHandler` のパフォーマンスが向上しました。 一部のマイクロサービス アーキテクチャなど、多くの送信 HTTP 要求を作成するアプリの場合、スループットが向上します。 読み込み時には、`HttpClient` のスループットが Linux では最大 60%、Windows では 20% 向上する可能性があります。

詳細については、[このような改善を実現したプルリクエスト](https://github.com/dotnet/corefx/pull/32568)に関する記事を参照してください。

## <a name="additional-information"></a>追加情報

変更の全一覧については、[ASP.NET Core 2.2 のリリース ノート](https://github.com/dotnet/aspnetcore/releases/tag/2.2.0)に関する記事を参照してください。
