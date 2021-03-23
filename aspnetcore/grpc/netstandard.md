---
title: .NET Standard 2.0 での gRPC クライアントの使用
author: jamesnk
description: .NET Standard 2.0 をサポートするアプリとライブラリで .NET gRPC クライアントを使用する方法について説明します。
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
uid: grpc/netstandard
ms.openlocfilehash: a6b066979dcdcdf648b8b0326bef47fe0e466266
ms.sourcegitcommit: 07e7ee573fe4e12be93249a385db745d714ff6ae
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/12/2021
ms.locfileid: "103422485"
---
# <a name="use-grpc-client-with-net-standard-20"></a>.NET Standard 2.0 での gRPC クライアントの使用

作成者: [James Newton-King](https://twitter.com/jamesnk)

この記事では、[.NET Standard 2.0](/dotnet/standard/net-standard) をサポートする .NET 実装で .NET gRPC クライアントを使用する方法について説明します。

## <a name="net-implementations"></a>.NET 実装

次の .NET 実装 (またはそれ以降) では [Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client/) をサポートしていますが、HTTP/2 の完全サポートは提供していません。

* .NET Core 2.1
* .NET Framework 4.6.1
* Mono 5.4
* Xamarin.iOS 10.14
* Xamarin.Android 8.0
* ユニバーサル Windows プラットフォーム 10.0.16299
* Unity 2018.1

.NET gRPC クライアントでは、追加の構成を使用して、これらの .NET 実装からサービスを呼び出すことができます。

## <a name="httphandler-configuration"></a>HttpHandler の構成

HTTP プロバイダーは `GrpcChannelOptions.HttpHandler` を使用して構成する必要があります。 ハンドラーが構成されていない場合、エラーがスローされます。

> `System.PlatformNotSupportedException`: gRPC over HTTP/2 をサポートしていない .NET 実装で RPC 呼び出しを正常に行うには、gRPC に追加の構成が必要です。 HTTP プロバイダーは `GrpcChannelOptions.HttpHandler` を使用して指定する必要があります。 構成した HTTP プロバイダーでは、HTTP/2 をサポートするか、gRPC-Web を使用するように構成する必要があります。

UWP、Xamarin、Unity など、HTTP/2 をサポートしていない .NET 実装では、代わりに gRPC-Web を使用できます。

```csharp
var channel = GrpcChannel.ForAddress("https://localhost:5001", new GrpcChannelOptions
    {
        HttpHandler = new GrpcWebHandler(new HttpClientHandler())
    });

var client = new Greeter.GreeterClient(channel);
var response = await client.SayHelloAsync(new HelloRequest { Name = ".NET" });
```

詳細については、「[.NET gRPC クライアントを使用して gRPC-Web を構成する](xref:grpc/browser#configure-grpc-web-with-the-net-grpc-client)」を参照してください。

## <a name="net-framework"></a>.NET Framework

.NET Framework では gRPC over HTTP/2 を制限付きでサポートしています。 .NET Framework で gRPC over HTTP/2 を有効にするには、<xref:System.Net.Http.WinHttpHandler> を使用するようにチャネルを構成してください。

`WinHttpHandler` を使用するための要件と制限事項を以下に示します。

* Windows 10 ビルド 19622 以降。
* [System.Net.Http.WinHttpHandler](https://www.nuget.org/packages/System.Net.Http.WinHttpHandler/) NuGet パッケージへの参照。
* 単項およびサーバー ストリーミングの gRPC 呼び出しのみがサポートされています。
* TLS 経由の gRPC 呼び出しのみがサポートされています。

```csharp
var channel = GrpcChannel.ForAddress("https://localhost:5001", new GrpcChannelOptions
    {
        HttpHandler = new WinHttpHandler()
    });

var client = new Greeter.GreeterClient(channel);
var response = await client.SayHelloAsync(new HelloRequest { Name = ".NET" });
```

> [!NOTE]
> .NET Framework のサポートは初期段階で、プレリリースのソフトウェアを使用する必要があります。
> * Windows 10 ビルド 19622 以降は [Windows Insider](https://insider.windows.com/) ビルドとして利用できます。
> * 必要なバージョンの `System.Net.Http.WinHttpHandler` は、現在 NuGet.org にありません。[この NuGet フィード](https://pkgs.dev.azure.com/dnceng/public/_packaging/dotnet6/nuget/v3/index.json)で提供されている最新のプレリリース バージョンを使用する必要があります。

## <a name="grpc-c-core-library"></a>gRPC C# コアライブラリ

.NET Framework と Xamarin 向けの代替オプションは、[gRPC C# コアライブラリ](https://grpc.io/docs/languages/csharp/quickstart/)を使用して、gRPC 呼び出しを行うことです。 これは、.NET Framework と Xamarin で HTTP/2 を経由した gRPC の呼び出しをサポートしているサードパーティのライブラリです。 gRPC C-コアは、Microsoft によってサポートされていません。

## <a name="additional-resources"></a>その他の技術情報

* <xref:grpc/client>
* <xref:grpc/browser>
* [gRPC C# コアライブラリ](https://grpc.io/docs/languages/csharp/quickstart/)
