---
title: .NET での gRPC でサポートされているプラットフォーム
author: jamesnk
description: .NET での gRPC でサポートされているプラットフォームについて説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 3/11/2021
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
uid: grpc/supported-platforms
ms.openlocfilehash: c2bd808d16f11077e39aada829d79e8aedf2755b
ms.sourcegitcommit: 07e7ee573fe4e12be93249a385db745d714ff6ae
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/12/2021
ms.locfileid: "103413419"
---
# <a name="grpc-on-net-supported-platforms"></a>.NET での gRPC でサポートされているプラットフォーム

作成者: [James Newton-King](https://twitter.com/jamesnk)

この記事では、.NET で gRPC を使用するための要件と、サポートされているプラットフォームについて説明します。 次の 2 つの主な gRPC ワークロードにはさまざまな要件があります。

* [ASP.NET Core で gRPC サービスをホストする](#aspnet-core-grpc-server-requirements)
* [.NET クライアント アプリから gRPC を呼び出す](#net-grpc-client-requirements)

## <a name="wire-formats"></a>ワイヤ形式

gRPC では、HTTP/2 で利用できる高度な機能が活用されます。 HTTP/2 はどこでもサポートされているわけではありませんが、gRPC では、HTTP/1.1 を使用した 2 番目のワイヤ形式を利用できます。

* [`application/grpc`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md) - HTTP/2 による gRPC は、一般的な gRPC の使用方法です。
* [`application/grpc-web`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-WEB.md) - gRPC-Web では、HTTP/1.1 と互換性を持つように gRPC プロトコルが変更されます。 gRPC-Web はより多くの場所で使用できます。 gRPC-Web は、HTTP/2 を完全にサポートしていなくても、ブラウザー アプリやネットワークで使用できます。 2 つの高度な gRPC 機能がサポートされなくなりました。クライアント ストリーミングと双方向ストリーミングです。

.NET での gRPC では、両方のワイヤ形式がサポートされています。 `application/grpc` が既定で使用されます。 gRPC-Web の呼び出しを正常に行うには、クライアントとサーバーで gRPC-Web を構成する必要があります。 gRPC-Web の設定方法について詳しくは、「<xref:grpc/browser>」を参照してください。

## <a name="aspnet-core-grpc-server-requirements"></a>ASP.NET Core gRPC サーバーの要件

ASP.NET Core で gRPC サービスをホストするには、.NET Core 3.x 以降が必要です。

> [!div class="checklist"]
>
> * .NET 5 以降
> * .NET Core 3

ASP.NET Core gRPC サービスは、.NET Core でサポートされるすべてのオペレーティング システムでホストできます。

> [!div class="checklist"]
>
> * Windows
> * Linux
> * macOS&dagger;

&dagger;[macOS では、HTTPS を使用した ASP.NET Core アプリのホストがサポートされていません](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos)。

### <a name="supported-aspnet-core-servers"></a>サポートされる ASP.NET Core サーバー

すべての組み込み ASP.NET Core サーバーがサポートされています。

> [!div class="checklist"]
>
> * Kestrel
> * TestServer
> * IIS&dagger;
> * HTTP.sys&Dagger;

&dagger;IIS では、.NET 5 および Windows 10 ビルド 20241 以降が必要です。

&Dagger;HTTP.sys では、.NET 5 および Windows 10 ビルド 19529 以降が必要です。

gRPC を実行するために ASP.NET Core サーバーを構成する方法の詳細については、<xref:grpc/aspnetcore#server-options> に関する記事を参照してください。

### <a name="azure-services"></a>Azure サービス

> [!div class="checklist"]
>
> * [Azure Kubernetes Service (AKS)](https://azure.microsoft.com/services/kubernetes-service/)
> * [Azure App Service](https://azure.microsoft.com/services/app-service/)&dagger;

&dagger;Azure App Service では、HTTP/2 による gRPC のホストがサポートされていません。 gRPC-Web は互換性のある代替手段です。

Azure App Service では、HTTP/2 を使用した gRPC のサポートを向上させるための作業が進行中です。 詳細については、次を参照してください。[この GitHub の問題](https://github.com/dotnet/AspNetCore/issues/9020)します。

## <a name="net-grpc-client-requirements"></a>.NET gRPC クライアントの要件

[Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client/) パッケージでは、.NET Core 3 と .NET 5 以降で HTTP/2 を経由した gRPC 呼び出しをサポートしています。

.NET Framework では gRPC over HTTP/2 に制限付きサポートを利用できます。 UWP、Xamarin、Unity などのその他の .NET バージョンには必要な HTTP/2 サポートはなく、gRPC-Web を使用する必要があります。

次の表に、.NET の実装と、その gRPC クライアントのサポートを示します。

| .NET 実装                          | gRPC over HTTP/2   | gRPC-Web   |
|----------------------------------------------|--------------------|------------|
| .NET 5 以降                              | ✔️                | ✔️         |
| .NET Core 3                                  | ✔️                | ✔️         |
| .NET Core 2.1                                | ❌                | ✔️         |
| .NET Framework 4.6.1                         | ⚠️&dagger;        | ✔️         |
| Blazor WebAssembly                           | ❌                | ✔️         |
| Mono 5.4                                     | ❌                | ✔️         |
| Xamarin.iOS 10.14                            | ❌                | ✔️         |
| Xamarin.Android 8.0                          | ❌                | ✔️         |
| ユニバーサル Windows プラットフォーム 10.0.16299        | ❌                | ✔️         |
| Unity 2018.1                                 | ❌                | ✔️         |

&dagger;.NET Framework の場合、<xref:System.Net.Http.WinHttpHandler> を構成する必要があり、Windows 10 ビルド 19622 以降が必要です。

.NET Framework または gRPC-Web で `Grpc.Net.Client` を使用するには、追加の構成が必要です。 詳細については、「<xref:grpc/netstandard>」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:grpc/netstandard>
* [gRPC C# コアライブラリ](https://grpc.io/docs/languages/csharp/quickstart/)
