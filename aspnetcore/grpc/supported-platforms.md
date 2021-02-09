---
title: .NET での gRPC でサポートされているプラットフォーム
author: jamesnk
description: .NET での gRPC でサポートされているプラットフォームについて説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 01/22/2021
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
ms.openlocfilehash: 6e48a19027f79b75edeebde9c584419871fba533
ms.sourcegitcommit: e311cfb77f26a0a23681019bd334929d1aaeda20
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99530165"
---
# <a name="grpc-on-net-supported-platforms"></a>.NET での gRPC でサポートされているプラットフォーム

作成者: [James Newton-King](https://twitter.com/jamesnk)

この記事では、.NET で gRPC を使用するための要件と、サポートされているプラットフォームについて説明します。

gRPC では、HTTP/2 で利用できる高度な機能が活用されます。 HTTP/2 はどこでもサポートされているわけではありませんが、gRPC では、HTTP/1.1 を使用した 2 番目のワイヤ形式を利用できます。

* [`application/grpc`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md) - HTTP/2 による gRPC は、一般的な gRPC の使用方法です。
* [`application/grpc-web`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-WEB.md) - gRPC-Web では、HTTP/1.1 と互換性を持つように gRPC プロトコルが変更されます。 gRPC-Web はより多くの場所で使用できます。特に、ブラウザー アプリから呼び出すことができます。 2 つの高度な gRPC 機能がサポートされなくなりました。クライアント ストリーミングと双方向ストリーミングです。

.NET での gRPC では、両方のワイヤ形式がサポートされています。 既定では、HTTP/2 による gRPC が使用されます。 gRPC-Web の設定方法について詳しくは、「<xref:grpc/browser>」を参照してください。

## <a name="device-requirements"></a>デバイスの要件

.NET での gRPC では、.NET Core でサポートされているすべてのデバイスがサポートされます。

> [!div class="checklist"]
>
> * Windows
> * Linux
> * macOS&dagger;
> * ブラウザー&Dagger;

&dagger;[macOS では、HTTPS を使用した ASP.NET Core アプリのホストがサポートされていません](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos)。 macOS 上の gRPC クライアントでは、HTTPS を使用するリモート サービスを呼び出すことができます。

&Dagger;Blazor WebAssembly アプリでは、gRPC-Web を使用して gRPC サービスを呼び出すことができます。

## <a name="aspnet-core-server-requirements"></a>ASP.NET Core サーバーの要件

gRPC サービスは、すべての組み込み ASP.NET Core サーバー上でホストできます。

> [!div class="checklist"]
>
> * Kestrel
> * TestServer
> * IIS&dagger;
> * HTTP.sys&Dagger;

&dagger;IIS では、.NET 5 および Windows 10 ビルド 20241 以降が必要です。

&Dagger;HTTP.sys では、.NET 5 および Windows 10 ビルド 19529 以降が必要です。

gRPC を実行するために ASP.NET Core サーバーを構成する方法の詳細については、<xref:grpc/aspnetcore#server-options> に関する記事を参照してください。

## <a name="net-version-requirements"></a>.NET バージョンの要件

.NET での gRPC では、.NET Core 3 および .NET 5 以降がサポートされています。

> [!div class="checklist"]
>
> * .NET 5 以降
> * .NET Core 3

.NET での gRPC では、.NET Framework および Xamarin での実行はサポートされていません。 [gRPC C# コアライブラリ](https://grpc.io/docs/languages/csharp/quickstart/)は、.NET Framework と Xamarin がサポートされているサード パーティ製のライブラリです。 gRPC C-コアは、Microsoft によってサポートされていません。

## <a name="azure-services"></a>Azure サービス

> [!div class="checklist"]
>
> * [Azure Kubernetes Service (AKS)](https://azure.microsoft.com/services/kubernetes-service/)
> * [Azure App Service](https://azure.microsoft.com/services/app-service/)&dagger;

&dagger;Azure App Service では、HTTP/2 による gRPC のホストがサポートされていません。 gRPC-Web は互換性のある代替手段です。

Azure App Service では、HTTP/2 を使用した gRPC のサポートを向上させるための作業が進行中です。 詳細については、次を参照してください。[この GitHub の問題](https://github.com/dotnet/AspNetCore/issues/9020)します。

## <a name="additional-resources"></a>その他の技術情報

* [gRPC C# コアライブラリ](https://grpc.io/docs/languages/csharp/quickstart/)
