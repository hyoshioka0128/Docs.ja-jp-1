---
title: .NET 構成のための gRPC
author: jamesnk
description: .NET アプリのために gRPC を構成する方法について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.custom: mvc
ms.date: 11/23/2020
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
uid: grpc/configuration
ms.openlocfilehash: 3f03d774a2223dc1fc08adcbc85cdc9a2b63dea0
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588894"
---
# <a name="grpc-for-net-configuration"></a>.NET 構成のための gRPC

## <a name="configure-services-options"></a>サービス オプションを構成する

gRPC サービスは、*Startup.cs* の `AddGrpc` によって構成されます。 次の表では、gRPC サービスを構成するためのオプションについて説明します。

| オプション | 既定値 | 説明 |
| ------ | ------------- | ----------- |
| MaxSendMessageSize | `null` | サーバーから送信できる最大メッセージ サイズ (バイト単位)。 構成された最大メッセージ サイズを超えるメッセージを送信しようとすると、例外が発生します。 `null` に設定する場合、メッセージ サイズは制限されません。 |
| MaxReceiveMessageSize | 4 MB | サーバーで受信できる最大メッセージ サイズ (バイト単位)。 サーバーでこの制限を超えるメッセージを受信すると、例外がスローされます。 この値を大きくすると、サーバーはより大きなメッセージを受け取れますが、メモリの消費に悪影響を与える可能性があります。 `null` に設定する場合、メッセージ サイズは制限されません。 |
| EnableDetailedErrors | `false` | `true` の場合、サービス メソッドで例外がスローされると、詳細な例外メッセージがクライアントに返されます。 既定値は、`false` です。 `EnableDetailedErrors` を `true` に設定すると、機密情報が漏洩する可能性があります。 |
| CompressionProviders | gzip | メッセージの圧縮と圧縮解除に使用される圧縮プロバイダーのコレクション。 カスタム圧縮プロバイダーを作成し、コレクションに追加することができます。 既定で構成されているプロバイダーは、**gzip** 圧縮をサポートしています。 |
| <span style="word-break:normal;word-wrap:normal">ResponseCompressionAlgorithm</span> | `null` | サーバーから送信されるメッセージの圧縮に使用される圧縮アルゴリズム。 このアルゴリズムは、`CompressionProviders` の圧縮プロバイダーと一致している必要があります。 アルゴリズムで応答を圧縮するには、そのアルゴリズムをサポートしていることをクライアントが **grpc-accept-encoding** ヘッダーで送信することによって、そのことを示す必要があります。 |
| ResponseCompressionLevel | `null` | サーバーから送信されるメッセージの圧縮に使用される圧縮レベル。 |
| インターセプター | None | 各 gRPC 呼び出しで実行されるインターセプターのコレクション。 インターセプターは、登録されている順序で実行されます。 グローバルに構成されたインターセプターは、1 つのサービスに対してインターセプターが構成される前に実行されます。 gRPC インターセプターの詳細については、「[gRPC インターセプターとミドルウェア](xref:grpc/migration#grpc-interceptors-vs-middleware)」を参照してください。 |
| IgnoreUnknownServices | `false` | `true` の場合、不明なサービスおよびメソッドへの呼び出しでは **UNIMPLEMENTED** 状態は返されず、要求は ASP.NET Core に登録された次のミドルウェアに渡されます。 |

`Startup.ConfigureServices` の `AddGrpc` 呼び出しに対して options デリゲートを指定することで、すべてのサービスに対してオプションを構成できます。

[!code-csharp[](~/grpc/configuration/sample/GrcpService/Startup.cs?name=snippet)]

1 つのサービスのオプションは、`AddGrpc` で提供されるグローバル オプションをオーバーライドします。また、`AddServiceOptions<TService>` を使用して構成することができます。

[!code-csharp[](~/grpc/configuration/sample/GrcpService/Startup2.cs?name=snippet)]

## <a name="configure-client-options"></a>クライアント オプションを構成する

gRPC のクライアント構成は `GrpcChannelOptions` で設定されます。 次の表では、gRPC チャネルを構成するためのオプションについて説明します。

| オプション | 既定値 | 説明 |
| ------ | ------------- | ----------- |
| HttpHandler | 新しいインスタンス | gRPC 呼び出しを行うために使用される `HttpMessageHandler`。 カスタム `HttpClientHandler` を構成したり、gRPC 呼び出しの HTTP パイプラインに追加のハンドラーを加えたりするために、クライアントを設定できます。 `HttpMessageHandler` が指定されていない場合、自動廃棄のチャネルのために新しい `HttpClientHandler` インスタンスが作成されます。 |
| HttpClient | `null` | gRPC 呼び出しを行うために使用される `HttpClient`。 この設定は、`HttpHandler` に代わる方法です。 |
| DisposeHttpClient | `false` | `true` に設定し、`HttpMessageHandler` または `HttpClient` が指定されている場合、`GrpcChannel` が廃棄されたときに (該当する) `HttpHandler` または `HttpClient` が廃棄されます。 |
| LoggerFactory | `null` | gRPC 呼び出しに関する情報をログに記録するため、クライアントによって使用される `LoggerFactory`。 `LoggerFactory` インスタンスは、依存関係の挿入から解決することも、`LoggerFactory.Create` を使用して作成することもできます。 ログ記録を構成する例については、「<xref:grpc/diagnostics#grpc-client-logging>」を参照してください。 |
| MaxSendMessageSize | `null` | クライアントから送信できる最大メッセージ サイズ (バイト単位)。 構成された最大メッセージ サイズを超えるメッセージを送信しようとすると、例外が発生します。 `null` に設定する場合、メッセージ サイズは制限されません。 |
| MaxReceiveMessageSize | 4 MB | クライアントで受信できる最大メッセージ サイズ (バイト単位)。 クライアントでこの制限を超えるメッセージを受信すると、例外がスローされます。 この値を大きくすると、クライアントはより大きなメッセージを受け取れますが、メモリの消費に悪影響を与える可能性があります。 `null` に設定する場合、メッセージ サイズは制限されません。 |
| 資格情報: | `null` | `ChannelCredentials` のインスタンス。 資格情報は、認証メタデータを gRPC 呼び出しに追加するために使用します。 |
| CompressionProviders | gzip | メッセージの圧縮と圧縮解除に使用される圧縮プロバイダーのコレクション。 カスタム圧縮プロバイダーを作成し、コレクションに追加することができます。 既定で構成されているプロバイダーは、**gzip** 圧縮をサポートしています。 |
| ThrowOperationCanceledOnCancellation | `false` | `true` に設定されている場合、呼び出しが取り消されたとき、または期限を超えたときに、クライアントから <xref:System.OperationCanceledException> がスローされます。 |
| MaxRetryAttempts | 5 | 最大再試行回数。 サービス構成で指定されている再試行回数と Hedging 試行回数が、この値によって制限されます。この値のみを設定しても、再試行は行われません。 再試行は、サービス構成で有効となり、`ServiceConfig` を使用して実行されます。 `null` 値を指定すると、最大再試行回数の制限が解除されます。 再試行の詳細については、「<xref:grpc/retries>」を参照してください。 |
| MaxRetryBufferSize | 16 MB | 再試行、または Hedging 呼び出しを実行するときに、送信済みメッセージを格納するために使用することができる、バイト単位の最大バッファー サイズ。 バッファーの制限を超過した場合、再試行はそれ以上実行されず、すべての Hedging 呼び出しは 1 回を除いてすべてキャンセルされます。 この制限は、チャネルを使用して実行されるすべての呼び出しに対して適用されます。 `null` 値を指定すると、再試行の最大バッファー サイズの制限が解除されます。 |
| <span style="word-break:normal;word-wrap:normal">MaxRetryBufferPerCallSize</span> | 1 MB | 再試行、または Hedging 呼び出しを実行するときに、送信済みメッセージを格納するために使用することができる、バイト単位の最大バッファー サイズ。 バッファーの制限を超過した場合、再試行はそれ以上実行されず、すべての Hedging 呼び出しは 1 回を除いてすべてキャンセルされます。 この制限は、1 回の呼び出しに適用されます。 `null` 値を指定すると、呼び出し 1 回あたりの、再試行の最大バッファー サイズの制限が解除されます。 |
| ServiceConfig | `null` | gRPC チャネルのサービス構成。 サービス構成を使用すると、[gRPC の再試行](xref:grpc/retries)を構成することができます。 |

コード例を次に示します。

* チャネルで送受信する最大メッセージ サイズを設定します。
* クライアントを作成します。

[!code-csharp[](~/grpc/configuration/sample/Program.cs?name=snippet&highlight=3-8)]

[!INCLUDE[](~/includes/gRPCazure.md)]

## <a name="additional-resources"></a>その他の技術情報

* <xref:grpc/aspnetcore>
* <xref:grpc/client>
* <xref:grpc/diagnostics>
* <xref:tutorials/grpc/grpc-start>
