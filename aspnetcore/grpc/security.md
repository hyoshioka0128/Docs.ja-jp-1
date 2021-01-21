---
title: gRPC での ASP.NET Core のセキュリティに関する考慮事項
author: jamesnk
description: ASP.NET Core のセキュリティに関する gRPC での考慮事項について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.custom: mvc
ms.date: 07/07/2019
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
uid: grpc/security
ms.openlocfilehash: 45ac0916a368cf68f4d40e14298a7628446989ee
ms.sourcegitcommit: 063a06b644d3ade3c15ce00e72a758ec1187dd06
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98252813"
---
# <a name="security-considerations-in-grpc-for-aspnet-core"></a>gRPC での ASP.NET Core のセキュリティに関する考慮事項

作成者: [James Newton-King](https://twitter.com/jamesnk)

この記事では、.NET Core を使用する gRPC をセキュリティで保護することについて説明します。

## <a name="transport-security"></a>トランスポート セキュリティ

gRPC のメッセージは、HTTP/2 を使用して送受信されます。 以下のことが推奨されます。

* [トランスポート層セキュリティ (TLS)](https://tools.ietf.org/html/rfc5246) を使用して、運用環境にある gRPC アプリ内のメッセージをセキュリティで保護します。
* gRPC サービスは、セキュリティで保護されたポートでのみリッスンと応答を行う必要があります。

::: moniker range=">= aspnetcore-5.0"
TLS は Kestrel で構成されます。 Kestrel エンドポイントの構成について詳しくは、[Kestrel のエンドポイント構成](xref:fundamentals/servers/kestrel/endpoints)に関するページを参照してください。
::: moniker-end

::: moniker range="< aspnetcore-5.0"
TLS は Kestrel で構成されます。 Kestrel エンドポイントの構成について詳しくは、[Kestrel のエンドポイント構成](xref:fundamentals/servers/kestrel#endpoint-configuration)に関するページを参照してください。
::: moniker-end

## <a name="exceptions"></a>例外

例外メッセージは、通常、クライアントに公開してはいけない機密データと見なされます。 既定では、gRPC は、gRPC サービスによってスローされた例外の詳細をクライアントに送信しません。 クライアントはその代わりに、エラーが発生したことを示す一般的なメッセージを受信します。 クライアントへの例外メッセージの配信は、(開発やテストなどでは) [EnableDetailedErrors](xref:grpc/configuration#configure-services-options) を使用してオーバーライドできます。 例外メッセージは、運用環境のアプリではクライアントに公開しないでください。

## <a name="message-size-limits"></a>メッセージ サイズの制限

gRPC のクライアントやサービスへの受信メッセージはメモリに読み込まれます。 メッセージ サイズの制限は、gRPC で過剰なリソースが消費されないようにするためのメカニズムです。

gRPC では、メッセージごとのサイズ制限を利用して受信メッセージと送信メッセージを管理します。 既定で、gRPC では受信メッセージは 4 MB に制限されます。 送信メッセージに制限はありません。

サーバーでは、`AddGrpc` を使用して、アプリ内のすべてのサービスに対して gRPC メッセージの制限を構成できます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddGrpc(options =>
    {
        options.MaxReceiveMessageSize = 1 * 1024 * 1024; // 1 MB
        options.MaxSendMessageSize = 1 * 1024 * 1024; // 1 MB
    });
}
```

`AddServiceOptions<TService>` を使用して、個々のサービスに対する制限を構成することもできます。 メッセージ サイズ制限の構成の詳細については、[gRPC の構成](xref:grpc/configuration)に関するページを参照してください。

## <a name="client-certificate-validation"></a>クライアント証明書の検証

[クライアント証明書](https://tools.ietf.org/html/rfc5246#section-7.4.4)は、接続が確立されるときに最初に検証されます。 既定で、Kestrel では接続のクライアント証明書の追加の検証は実行されません。

クライアント証明書によって保護されている gRPC サービスでは、[Microsoft.AspNetCore.Authentication.Certificate](xref:security/authentication/certauth) パッケージの使用が推奨されます。 ASP.NET Core の証明書認証では、クライアント証明書に対して、以下を含む追加の検証が実行されます。

* 証明書に有効な拡張キー使用法 (EKU) がある
* 有効期間内である
* 証明書の失効について確認する
