---
title: ASP.NET Core Blazor のホスティング モデル
author: guardrex
description: Blazor WebAssembly と Blazor Server のホスティング モデルを理解します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/07/2020
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
uid: blazor/hosting-models
ms.openlocfilehash: ea39235e51fc430dc2c67b4f2cf281c416705c0b
ms.sourcegitcommit: 1f35de0ca9ba13ea63186c4dc387db4fb8e541e0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2021
ms.locfileid: "104711075"
---
# <a name="aspnet-core-blazor-hosting-models"></a>ASP.NET Core Blazor のホスティング モデル

Blazor は、[WebAssembly](https://webassembly.org/) ベースの .NET ランタイム ( *Blazor WebAssembly* ) 上のブラウザーのクライアント側で、または ASP.NET Core ( *Blazor Server* ) のサーバー側で実行されるように設計された Web フレームワークです。 ホスティング モデルに関係なく、アプリ モデルとコンポーネント モデルは "*同じ*" です。

## Blazor WebAssembly

Blazor のプライマリ ホスティング モデルは、WebAssembly 上のブラウザーのクライアント側で実行されます。 Blazor アプリ、その依存関係、.NET ランタイムがブラウザーにダウンロードされます。 アプリがブラウザー UI スレッド上で直接実行されます。 UI の更新とイベントの処理は、同じプロセス内で行われます。 アプリの資産は、静的コンテンツをクライアントに提供できる Web サーバーまたはサービスに静的ファイルとして展開されます。

![Blazor WebAssembly:Blazor アプリは、ブラウザー内の UI スレッドで実行されます。](hosting-models/_static/blazor-webassembly.png)

Blazor WebAssembly アプリは、該当するファイルを提供するバックエンド ASP.NET Core アプリなしの展開用として作成されると、"*スタンドアロン*" Blazor WebAssembly アプリと呼ばれます。 そのアプリが、バックエンド アプリを使用して該当するファイルを提供する展開用として作成されると、"*ホストされている*" Blazor WebAssembly アプリと呼ばれます。 ホストされている Blazor WebAssembly **`Client`** アプリは、通常、Web API 呼び出し、または [SignalR](xref:signalr/introduction) を使用するネットワークを経由して、バックエンド **`Server`** アプリとのやり取りを行います (「<xref:tutorials/signalr-blazor>」を参照)。

`blazor.webassembly.js` スクリプトは、フレームワークによって提供され、次の処理を行います。

* .NET ランタイム、アプリ、およびアプリの依存関係のダウンロード。
* アプリを実行するランタイムの初期化。

Blazor WebAssembly ホスティング モデルには、次のいくつかの利点があります。

* .NET サーバー側の依存関係がありません。 アプリはクライアントにダウンロードされた後、完全に機能します。
* クライアントのリソースと機能が完全に活用されます。
* 作業がサーバーからクライアントにオフロードされます。
* アプリをホストするために ASP.NET Core Web サーバーが必要ありません。 Content Delivery Network (CDN) からアプリを提供するなどのサーバーレス展開シナリオが可能です。

Blazor WebAssembly ホスティング モデルには、次の制限があります。

* アプリがブラウザーの機能に制限されます。
* サポートされているクライアント ハードウェアおよびソフトウェア (たとえば、WebAssembly サポート) が必要です。
* ダウンロード サイズが大きくなり、アプリの読み込みに時間がかかります。
* .NET ランタイムとツールのサポートが十分ではありません。 たとえば、[.NET Standard](/dotnet/standard/net-standard) のサポートとデバッグには制限があります。

Blazor WebAssembly アプリを作成するには、「<xref:blazor/tooling>」を参照してください。

ホストされている Blazor アプリ モデルでは、[Docker コンテナー](/dotnet/standard/microservices-architecture/container-docker-introduction/index)がサポートされています。 Visual Studio での Docker のサポートについては、ホストされている Blazor WebAssembly ソリューションの **`Server`** プロジェクトを右クリックして、 **[追加]**  >  **[Docker のサポート]** の順に選択します。

## Blazor Server

Blazor Server ホスティング モデルを使用すると、アプリは ASP.NET Core アプリ内からサーバー上で実行されます。 UI の更新、イベント処理、JavaScript の呼び出しは、[SignalR](xref:signalr/introduction) 接続経由で処理されます。

![ブラウザーとサーバー上のアプリ (ASP.NET Core アプリ内でホストされている) とのやりとりは、SignalR 接続を介して行われます。](hosting-models/_static/blazor-server.png)

ASP.NET Core アプリにより、次の項目を追加するためにアプリの `Startup` クラスが参照されます。

* サーバー側サービス。
* 要求処理パイプラインに対するアプリ。

クライアントでは、`blazor.server.js` スクリプトによって、サーバーとの SignalR 接続が確立されます。 スクリプトは、ASP.NET Core 共有フレームワークの埋め込みリソースから、クライアント側のアプリに提供されます。 クライアント側のアプリは、必要に応じて、アプリの状態を永続化および復元する役割を担います。 

Blazor Server ホスティング モデルには、次のいくつかの利点があります。

* ダウンロード サイズが Blazor WebAssembly アプリよりもかなり小さく、アプリの読み込み時間が大幅に短縮されます。
* このアプリでは、.NET Core と互換性のあるすべての API の使用を含め、サーバーの機能を最大限に活用できます。
* サーバー上の .NET Core はアプリの実行に使用されるため、デバッグなどの既存の .NET ツールは想定どおりに動作します。
* シン クライアントがサポートされています。 たとえば、Blazor Server アプリは、WebAssembly がサポートされていないブラウザーや、リソースが制限されたデバイスで動作します。
* アプリのコンポーネント コードを含め、アプリの .NET/C# コード ベースがクライアントに提供されません。

> [!IMPORTANT]
> 最初のクライアント要求への応答として、Blazor Server アプリによってプリレンダリングされます。これにより、サーバー上で UI の状態が作成されます。 クライアントで SignalR 接続の作成が再試行される際は、**クライアントを同じサーバーに再接続する必要があります**。 複数のバックエンド サーバーを使用する Blazor Server アプリでは、SignalR 接続に "*スティッキー セッション*" を実装する必要があります。 詳細については、「[サーバーへの接続](#connection-to-the-server)」セクションを参照してください。

Blazor Server ホスティング モデルには、次の制限があります。

* 通常、遅延時間が長くなります。 すべてのユーザーの操作にネットワーク ホップが関与します。
* オフライン サポートがありません。 クライアント接続が失敗すると、アプリの動作が停止します。
* 多くのユーザーがいるアプリで、スケーラビリティが課題になります。 サーバーでは、複数のクライアント接続を管理し、クライアントの状態を処理する必要があります。
* アプリを提供するのに ASP.NET Core サーバーが必要です。 Content Delivery Network (CDN) からアプリを提供するなどのサーバーレス展開シナリオは不可能です。

Blazor Server アプリを作成するには、「<xref:blazor/tooling>」を参照してください。

Blazor Server アプリ モデルでは、[Docker コンテナー](/dotnet/standard/microservices-architecture/container-docker-introduction/index)がサポートされています。 Visual Studio での Docker のサポートについては、Visual Studio のプロジェクトを右クリックし、 **[追加]**  >  **[Docker のサポート]** の順に選択します。

### <a name="comparison-to-server-rendered-ui"></a>サーバーでレンダリングされる UI との比較

Blazor Server アプリを理解する方法の 1 つは、Razor ビューまたは Razor Pages を使用して ASP.NET Core アプリで UI をレンダリングするための従来のモデルとの違いを理解することです。 どちらのモデルでも、[Razor 言語](xref:mvc/views/razor)を使用してレンダリング用の HTML コンテンツが記述されますが、マークアップのレンダリング "*方法*" が大きく異なります。

Razor ページまたはビューがレンダリングされると、Razor コードのすべての行で HTML がテキスト形式で出力されます。 レンダリング後、サーバーでは、生成されたすべての状態を含むページ インスタンスまたはビュー インスタンスが破棄されます。 たとえば、サーバーの検証に失敗して検証の概要が表示される場合など、ページに対する別の要求が発生すると、次の処理が行われます。

* ページ全体が HTML テキストに再びレンダリングされます。
* ページがクライアントに送信されます。

Blazor アプリは、"*コンポーネント*" と呼ばれる UI の再利用可能な要素で構成されています。 コンポーネントには、C# コード、マークアップ、およびその他のコンポーネントが含まれています。 コンポーネントがレンダリングされると、Blazor により、HTML または XML ドキュメント オブジェクト モデル (DOM) に似ている、含まれるコンポーネントのグラフが生成されます。 このグラフには、プロパティとフィールドに保持されているコンポーネントの状態が含まれます。 Blazor により、コンポーネント グラフが評価されて、マークアップのバイナリ表現が生成されます。 バイナリ形式は次のように処理できます。

* HTML テキストに変換できます (プリレンダリング時&dagger;)。
* 通常のレンダリング時にマークアップを効率的に更新するために使用できます。

&dagger;*プリレンダリング*:要求された Razor コンポーネントはサーバーで静的 HTML にコンパイルされ、クライアントに送信されて、そこでユーザーに対してレンダリングされます。 クライアントとサーバーの間で接続が確立されると、コンポーネントのプリレンダリング済みの静的な要素が対話型要素に置き換えられます。 プリレンダリングによって、ユーザーに対するアプリの応答性が向上します。

Blazor の UI の更新は、次の方法でトリガーされます。

* ユーザー操作 (ボタンの選択など)。
* タイマーなどのアプリ トリガー。

コンポーネント グラフが再レンダリングされ、UI *diff* (相違) が計算されます。 この diff は、クライアントで UI を更新するために必要な DOM 編集の最小セットです。 diff はバイナリ形式でクライアントに送信され、ブラウザーによって適用されます。

コンポーネントは、ユーザーがクライアント上でコンポーネントから移動すると破棄されます。 ユーザーがコンポーネントを操作している間、コンポーネントの状態 (サービス、リソース) はサーバーのメモリに保持されている必要があります。 多くのコンポーネントの状態はサーバーによって同時に維持される場合があるため、メモリ不足の問題に対処する必要があります。 サーバー メモリを最大限に活用できるよう Blazor Server アプリを作成する方法については、「<xref:blazor/security/server/threat-mitigation>」を参照してください。

### <a name="circuits"></a>回線

Blazor Server アプリは、[ASP.NET Core SignalR](xref:signalr/introduction) 上に構築されています。 各クライアントは、"*回線*" と呼ばれる 1 つ以上の SignalR 接続を介してサーバーと通信します。 回線は、一時的なネットワーク中断が許容される SignalR 接続を介した Blazor の抽象化です。 Blazor クライアントで、SignalR 接続が切断されていることが確認されると、新しい SignalR 接続を使用してサーバーへの再接続が試行されます。

Blazor Server アプリに接続されている各ブラウザー画面 (ブラウザー タブまたは iframe) では、SignalR 接続が使用されます。 これは、サーバーでレンダリングされる一般的なアプリとの、もう 1 つの重要な違いです。 サーバーでレンダリングされるアプリでは、複数のブラウザー画面で同じアプリを開いても、通常、サーバー上で追加のリソースは要求されません。 Blazor Server アプリでは、ブラウザー画面ごとに、個別の回線とコンポーネント状態の個別のインスタンスをサーバーで管理する必要があります。

Blazor では、ブラウザー タブを閉じるか、または外部 URL に移動して "*正常に終了*" することが検討されます。 正常な終了が行われた場合は、回線と関連リソースが直ちに解放されます。 ネットワークの中断などにより、クライアントが正常に切断されないこともあります。 Blazor Server では、クライアントが再接続できるように、切断された回線が格納されます (その間隔は設定できます)。

Blazor Server を使用すると、コードで "*回線ハンドラー*" を定義できます。これにより、ユーザーの回線の状態の変更時にコードを実行できます。 詳細については、「<xref:blazor/fundamentals/signalr?pivots=server#blazor-server-circuit-handler>」を参照してください。

### <a name="ui-latency"></a>UI 遅延時間

UI 遅延時間とは、アクションが開始されてから UI が更新されるまでにかかる時間のことです。 ユーザーに対するアプリの応答性を高めるには、UI 遅延時間の値を小さくすることが不可欠です。 Blazor Server アプリでは、各アクションがサーバーに送信され、処理されて、UI diff が返されます。 したがって、UI 遅延時間は、アクションの処理時のネットワーク遅延時間とサーバー遅延時間の合計になります。

企業のプライベート ネットワークに限定されたビジネス アプリでは、通常、ネットワーク遅延時間によってユーザーが遅延を感じる度合いはわずかです。 インターネット経由で展開されたアプリでは、特にユーザーが地理的に広く分散している場合、ユーザーが遅延を感じる可能性があります。

メモリ使用量も、アプリ遅延時間の一因となる場合があります。 メモリ使用量が増加すると、ガベージ コレクションまたはディスクへのメモリのページングが頻繁に発生します。どちらの場合も、アプリのパフォーマンスが低下し、その結果、UI 遅延時間が長くなります。

Blazor Server アプリは、ネットワーク遅延時間とメモリ使用量を低減することで、UI 遅延時間を最小限に抑えるように最適化する必要があります。 ネットワーク遅延時間を測定する方法については、「<xref:blazor/host-and-deploy/server#measure-network-latency>」を参照してください。 SignalR と Blazor の詳細については、以下を参照してください。

* <xref:blazor/host-and-deploy/server>
* <xref:blazor/security/server/threat-mitigation>

### <a name="connection-to-the-server"></a>サーバーへの接続

Blazor Server アプリには、サーバーへのアクティブな SignalR 接続が必要です。 接続が失われた場合、アプリではサーバーへの再接続が試行されます。 クライアントの状態がサーバーのメモリ内に保持されている限り、クライアント セッションは状態を失うことなく再開されます。

最初のクライアント要求への応答として、Blazor Server アプリによってプリレンダリングされます。これにより、サーバー上で UI の状態が作成されます。 クライアントで SignalR 接続の作成が再試行される際は、クライアントを同じサーバーに再接続する必要があります。 複数のバックエンド サーバーを使用する Blazor Server アプリでは、SignalR 接続に "*スティッキー セッション*" を実装する必要があります。

Blazor Server アプリには [Azure SignalR Service](/azure/azure-signalr) を使用することをお勧めします。 このサービスでは、多数の同時 SignalR 接続に対して Blazor Server アプリをスケールアップできます。 Azure SignalR サービスでは、サービスの `ServerStickyMode` オプションまたは構成値を `Required` に設定することにより、スティッキー セッションが有効になります。 詳細については、「<xref:blazor/host-and-deploy/server#signalr-configuration>」を参照してください。

IIS を使用すると、スティッキー セッションは "*アプリケーション要求ルーティング*" によって有効になります。 詳しくは、「[アプリケーション要求ルーティングを使用した HTTP 負荷分散](/iis/extensions/configuring-application-request-routing-arr/http-load-balancing-using-application-request-routing)」をご覧ください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:blazor/tooling>
* <xref:blazor/project-structure>
* <xref:signalr/introduction>
* <xref:blazor/fundamentals/signalr>
* <xref:tutorials/signalr-blazor>
