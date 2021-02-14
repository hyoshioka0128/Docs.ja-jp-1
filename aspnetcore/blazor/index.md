---
title: ASP.NET Core Blazor の概要
author: guardrex
description: ASP.NET Core アプリ内に .NET を使った対話型のクライアント側 Web UI を構築する方法である、ASP.NET Core Blazor について調べます。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc, seoapril2019, devx-track-js
ms.date: 09/25/2020
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
uid: blazor/index
ms.openlocfilehash: 560fead9868bab9888c0d6a69cf09f135bbf39cc
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100279738"
---
# <a name="introduction-to-aspnet-core-blazor"></a>ASP.NET Core Blazor の概要

*Blazor へようこそ。*

Blazor は、[.NET](/dotnet/standard/tour) を使って対話型のクライアント側 Web UI を構築するためのフレームワークです。

* [JavaScript](https://www.javascript.com) の代わりに [C#](/dotnet/csharp/) を使って、優れた対話型 UI を作成します。
* .NET で記述された、サーバー側とクライアント側のアプリのロジックを共有します。
* モバイル ブラウザーを含めた広範なブラウザーのサポートのために、HTML および CSS として UI をレンダリングします。
* [Docker](/dotnet/standard/microservices-architecture/container-docker-introduction/index) などの最新のホスティング プラットフォームと統合します。

クライアント側の Web 開発に .NET を使用すると、次のような利点があります。

* JavaScript ではなく C# でコードを記述します。
* [.NET ライブラリ](/dotnet/standard/class-libraries)の既存の .NET エコシステムを活用します。
* サーバーとクライアント全体でアプリ ロジックを共有します。
* .NET のパフォーマンス、信頼性、およびセキュリティから利点が得られます。
* Windows、Linux、macOS 上の [Visual Studio](https://visualstudio.microsoft.com) を使って生産性を維持します。
* 多機能で使いやすい安定した言語、フレームワーク、およびツールの共通セットに基づいて構築します。

## <a name="components"></a>コンポーネント

Blazor アプリは、"*コンポーネント*" に基づいています。 Blazor のコンポーネントとは、ページ、ダイアログ、データ エントリ フォームなどの UI の要素です。

コンポーネントは、[.NET アセンブリ](/dotnet/standard/assembly/)に組み込まれている次のような .NET C# クラスです。

* 柔軟な UI のレンダリング ロジックを定義する。
* ユーザー イベントを処理する。
* 入れ子にしたり再利用したりできる。
* [Razor クラス ライブラリまたは ](xref:razor-pages/ui-class)[NuGet パッケージ](/nuget/what-is-nuget)として共有および配布できる。

通常、コンポーネント クラスは、`.razor` ファイル拡張子を持つ [Razor](xref:mvc/views/razor) マークアップ ページの形式で記述されます。 Blazor のコンポーネントは、正式には " *Razor コンポーネント*" と呼ばれます。 Razor とは、開発者の生産性のために設計された、C# コードに HTML マークアップを結合するための構文です。 Razor を使用すると、Visual Studio の [IntelliSense](/visualstudio/ide/using-intellisense) によるプログラミング サポートを利用して、同一ファイル内で HTML マークアップと C# を切り替えることができます。 また、Razor Pages と MVC でも Razor が使用されます。 要求/応答モデルを中心に構築される Razor Pages や MVC とは異なり、コンポーネントは特にクライアント側の UI ロジックとコンポジションに対して使用されます。

Blazor では、UI コンポジションにとって自然な HTML タグを使用します。 次の Razor マークアップは、ダイアログを表示し、ユーザーがボタンを選択したときにイベントを処理するコンポーネント (`Dialog.razor`) を示しています。

```razor
<div class="card" style="width:22rem">
    <div class="card-body">
        <h3 class="card-title">@Title</h3>
        <p class="card-text">@ChildContent</p>
        <button @onclick="OnYes">Yes!</button>
    </div>
</div>

@code {
    [Parameter]
    public RenderFragment ChildContent { get; set; }

    [Parameter]
    public string Title { get; set; }

    private void OnYes()
    {
        Console.WriteLine("Write to the console in C#! 'Yes' button selected.");
    }
}
```

前の例の `OnYes` は、ボタンの `onclick` イベントによってトリガーされる C# メソッドです。 ダイアログのテキスト (`ChildContent`) とタイトル (`Title`) は、UI にこのコンポーネントを使用する次のコンポーネントによって提供されます。

`Dialog` コンポーネントは、HTML タグを使用して、別のコンポーネント内に入れ子にされます。 次の例では、`Index` コンポーネント (`Pages/Index.razor`) で前の `Dialog` コンポーネントが使われています。 タグの `Title` 属性は、タイトルの値を `Dialog` コンポーネントの `Title` プロパティに渡します。  `Dialog` コンポーネントのテキスト (`ChildContent`) は、`<Dialog>` 要素の内容によって設定されます。 `Dialog` コンポーネントを `Index` に追加すると、[Visual Studio の IntelliSense](/visualstudio/ide/using-intellisense) の構文およびパラメーター補完により開発を迅速化できます。

```razor
@page "/"

<h1>Hello, world!</h1>

<p>
    Welcome to your new app.
</p>

<Dialog Title="Learn More">
    Do you want to <i>learn more</i> about Blazor?
</Dialog>
```

ブラウザー内で `Index` コンポーネントにアクセスすると、ダイアログがレンダリングされます。 ユーザーがボタンを選択すると、ブラウザーの開発者ツール コンソールに `OnYes` メソッドによって書き込まれたメッセージが表示されます。

![Index コンポーネントの内部に入れ子になっているブラウザーでレンダリングされたダイアログ コンポーネント。 ブラウザー開発者ツール コンソールには、ユーザーが UI の [はい] ボタンを選択したときに C# コードによって書き込まれるメッセージが 表示されます。](index/_static/dialog.png)

コンポーネントはレンダリングされると、"*レンダリング ツリー*" と呼ばれる、ブラウザーの [ドキュメント オブジェクト モデル (DOM)](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction) のメモリ内表現になります。これは、柔軟かつ効率的な方法で UI を更新するために使われます。

## Blazor WebAssembly

Blazor WebAssembly は、.NET を使って対話型のクライアント側 Web アプリを構築するための[シングルページ アプリ (SPA) フレームワーク](/dotnet/architecture/modern-web-apps-azure/choose-between-traditional-web-and-single-page-apps)です。 Blazor WebAssembly は、プラグインを使用せずに、または他の言語へのコードの再コンパイルなしに、オープンな Web 標準を使用します。 Blazor WebAssembly はモバイル ブラウザーも含めて、すべての最新の Web ブラウザーで動作します。

[WebAssembly](https://webassembly.org) (略称 `wasm`) によって、Web ブラウザー内で .NET コードを実行することが可能になります。 WebAssembly は、ダウンロードを高速化し実行速度を最大限に高めるために最適化されたコンパクトなバイトコード形式です。 WebAssembly はオープンな Web 標準であり、プラグインを使わずに Web ブラウザー内でサポートされます。

WebAssembly コードを使用すると、JavaScript を介してブラウザーの全機能にアクセスできます。これは、"*JavaScript の相互運用性*" と呼ばれますが、"*JavaScript 相互運用*" や "*JS 相互運用*" とよく省略されます。 ブラウザーの WebAssembly 経由で実行される .NET コードは、ブラウザーの JavaScript サンドボックス内で実行されます。その際、クライアント コンピューター上での悪意のある操作に対して、サンドボックスに備わった保護が適用されます。

![Blazor WebAssembly は WebAssembly を使用してブラウザーで .NET コードを実行します。](index/_static/blazor-webassembly.png)

Blazor WebAssembly アプリをビルドしてブラウザーで実行する場合:

* C# コード ファイルと Razor ファイルが .NET アセンブリにコンパイルされます。
* そのアセンブリと [.NET ランタイム](/dotnet/framework/get-started/overview)がブラウザーにダウンロードされます。
* Blazor WebAssembly により .NET ランタイムがブートストラップされ、アプリのアセンブリを読み込むようにそのランタイムが構成されます。 Blazor WebAssembly ランタイムでは、JavaScript 相互運用を使って DOM 操作とブラウザー API の呼び出しが処理されます。

公開されているアプリのサイズである "*ペイロードのサイズ*" は、アプリの使用性に関する重要なパフォーマンス要因になります。 大規模なアプリは、ブラウザーへのダウンロードに比較的長い時間がかかり、ユーザー エクスペリエンスが損なわれます。 Blazor WebAssembly では、ダウンロード時間を短縮するためにペイロードのサイズが最適化されます。

::: moniker range=">= aspnetcore-5.0"

* アプリが[中間言語 (IL) トリマー](xref:blazor/host-and-deploy/configure-trimmer)によって公開される場合、アプリから未使用コードが除去されます。
* HTTP 応答が圧縮されます。
* .NET ランタイムとアセンブリがブラウザーにキャッシュされます。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* アプリが[中間言語 (IL) リンカー](xref:blazor/host-and-deploy/configure-linker)によって公開される場合、アプリから未使用コードが除去されます。
* HTTP 応答が圧縮されます。
* .NET ランタイムとアセンブリがブラウザーにキャッシュされます。

::: moniker-end

## Blazor Server

Blazor では、UI の更新プログラムを適用する方法からコンポーネントのレンダリング ロジックが分離されます。 *Blazor Server* では、ASP.NET Core アプリでサーバー上の Razor コンポーネントをホストするためのサポートが提供されます。 UI の更新は [SignalR](xref:signalr/introduction) 接続を介して処理されます。

ランタイムはサーバー上にとどまり、次の処理を行います。

* アプリの C# コードを実行しています。
* ブラウザーからサーバーへの UI イベントの送信。
* サーバーによって返送される、レンダリングされたコンポーネントへの UI の更新の適用。

ブラウザーと通信するために Blazor Server で使われる接続は、JavaScript 相互運用の呼び出しを処理するためにも使われます。

![Blazor Server では、サーバー上で .NET コードが実行され、SignalR 接続を介してクライアント上のドキュメント オブジェクト モデルとのやりとりが行われます](index/_static/blazor-server.png)

## <a name="javascript-interop"></a>JavaScript 相互運用

サード パーティ製の JavaScript ライブラリや、ブラウザーの API へのアクセスを必要とするアプリのために、コンポーネントは JavaScript と相互運用します。 コンポーネントでは、JavaScript で使用できるライブラリまたは API はいずれも使用することができます。 C# コードによる [JavaScript コードの呼び出し](xref:blazor/call-javascript-from-dotnet)、および JavaScript コードによる [C# コードの呼び出し](xref:blazor/call-dotnet-from-javascript)を行うことができます。

## <a name="code-sharing-and-net-standard"></a>コードの共有と .NET Standard

Blazor は [.NET Standard](/dotnet/standard/net-standard) を実装します。これにより、Blazor プロジェクトは .NET Standard の仕様に準拠するライブラリを参照できます。 .NET Standard は、.NET 実装全体で共通した .NET API の標準仕様です。 .NET Standard のクラス ライブラリは、Blazor、.NET Framework、.NET Core、Xamarin、Mono、Unity など、さまざまな .NET プラットフォーム全体で共有することができます。

Web ブラウザー内で使用できない API (たとえば、ファイル システムへのアクセス、ソケットを開く機能、スレッドなど) からは、<xref:System.PlatformNotSupportedException> がスローされます。

## <a name="additional-resources"></a>その他の技術情報

* [WebAssembly](https://webassembly.org)
* <xref:blazor/hosting-models>
* <xref:tutorials/signalr-blazor>
* <xref:blazor/call-javascript-from-dotnet>
* <xref:blazor/call-dotnet-from-javascript>
* [C# のガイド](/dotnet/csharp/)
* [ASP.NET Core の Razor 構文リファレンス](xref:mvc/views/razor)
* [HTML](https://www.w3.org/html/)
* [Awesome Blazor](https://github.com/AdrienTorris/awesome-blazor) コミュニティへのリンク
