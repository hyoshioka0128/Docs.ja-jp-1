---
title: gRPC サービスの C-core から ASP.NET Core への移行
author: juntaoluo
description: 既存の C-core ベースの gRPC アプリを移動して ASP.NET Core スタック上で実行する方法について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: johluo
ms.date: 09/25/2019
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
uid: grpc/migration
ms.openlocfilehash: 27c53dd4b41d6c99e45fccb5af79bab1ed5dc1b9
ms.sourcegitcommit: 063a06b644d3ade3c15ce00e72a758ec1187dd06
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98253151"
---
# <a name="migrating-grpc-services-from-c-core-to-aspnet-core"></a>gRPC サービスの C-core から ASP.NET Core への移行

作成者: [John Luo](https://github.com/juntaoluo)

基になるスタックの実装のため、[C-core ベースの gRPC](https://grpc.io/blog/grpc-stacks) アプリと ASP.NET Core ベースのアプリ間で、すべての機能が同じように動作するわけではありません。 このドキュメントでは、2 つのスタック間で移行する場合の主な違いについて説明します。

## <a name="grpc-service-implementation-lifetime"></a>gRPC サービス実装の有効期間

ASP.NET Core スタックでは、既定で gRPC サービスは、[スコープが設定された有効期間](xref:fundamentals/dependency-injection#service-lifetimes)で作成されます。 これに対し、gRPC C-core は、既定で、[シングルトン有効期間](xref:fundamentals/dependency-injection#service-lifetimes)でサービスにバインドされます。

スコープが設定された有効期間により、サービス実装では、スコープが設定された有効期間を持つその他のサービスを解決できます。 たとえば、スコープが設定された有効期間では、コンストラクター インジェクションによって DI コンテナーから `DbContext` を解決することもできます。 スコープが設定された有効期間の使用:

* サービス実装の新しいインスタンスが、要求ごとに作成されます。
* 実装型のインスタンス メンバーを介して、要求間で状態を共有することはできません。
* シングルトン サービスでの共有状態は、DI コンテナーに格納することが期待されます。 格納された共有状態は、gRPC サービス実装のコンストラクターで解決されます。

サービス有効期間の詳細については、「<xref:fundamentals/dependency-injection#service-lifetimes>」を参照してください。

### <a name="add-a-singleton-service"></a>シングルトン サービスの追加

gRPC C-core 実装から ASP.NET Core への移行を容易にするために、サービス実装のサービス有効期間をスコープからシングルトンに変更することができます。 これには、サービス実装のインスタンスを DI コンテナーに追加する必要があります。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddGrpc();
    services.AddSingleton(new GreeterService());
}
```

ただし、シングルトン有効期間を持つサービス実装では、コンストラクター インジェクションによって、スコープ設定されたサービスを解決できなくなります。

## <a name="configure-grpc-services-options"></a>gRPC サービス オプションの構成

C-core ベースのアプリでは、`grpc.max_receive_message_length` や `grpc.max_send_message_length` などの設定は、[サーバー インスタンスの構築](https://grpc.io/grpc/csharp/api/Grpc.Core.Server.html#Grpc_Core_Server__ctor_System_Collections_Generic_IEnumerable_Grpc_Core_ChannelOption__)時に `ChannelOption` で構成します。

ASP.NET Core では、gRPC で `GrpcServiceOptions` 型による構成を提供しています。 たとえば、gRPC サービスの最大受信メッセージ サイズは、`AddGrpc` を使用して構成できます。 次の例では、既定の `MaxReceiveMessageSize` を 4 MB から 16 MB に変更しています。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddGrpc(options =>
    {
        options.MaxReceiveMessageSize = 16 * 1024 * 1024; // 16 MB
    });
}
```

構成の詳細については、「<xref:grpc/configuration>」を参照してください。

## <a name="logging"></a>ログの記録

C-core ベースのアプリでは、デバッグ目的で[ロガーを構成する](https://grpc.io/grpc/csharp/api/Grpc.Core.GrpcEnvironment.html?q=size#Grpc_Core_GrpcEnvironment_SetLogger_Grpc_Core_Logging_ILogger_)ために `GrpcEnvironment` に依存しています。 ASP.NET Core スタックでは、[ロギング API](xref:fundamentals/logging/index) によってこの機能を提供しています。 たとえば、コンストラクター インジェクションを使用して、gRPC サービスにロガーを追加できます。

```csharp
public class GreeterService : Greeter.GreeterBase
{
    public GreeterService(ILogger<GreeterService> logger)
    {
    }
}
```

## <a name="https"></a>HTTPS

::: moniker range=">= aspnetcore-5.0"
C-core ベースのアプリでは、[Server. Ports プロパティ](https://grpc.io/grpc/csharp/api/Grpc.Core.Server.html#Grpc_Core_Server_Ports)を使用して HTTPS を構成します。 同様の概念は、ASP.NET Core でサーバーを構成するために使用されます。 たとえば、Kestrel では、この機能のために[エンドポイント構成](xref:fundamentals/servers/kestrel/endpoints)を使用します。
::: moniker-end

::: moniker range="< aspnetcore-5.0"
C-core ベースのアプリでは、[Server. Ports プロパティ](https://grpc.io/grpc/csharp/api/Grpc.Core.Server.html#Grpc_Core_Server_Ports)を使用して HTTPS を構成します。 同様の概念は、ASP.NET Core でサーバーを構成するために使用されます。 たとえば、Kestrel では、この機能のために[エンドポイント構成](xref:fundamentals/servers/kestrel#endpoint-configuration)を使用します。
::: moniker-end

## <a name="grpc-interceptors-vs-middleware"></a>gRPC インターセプターとミドルウェア

ASP.NET Core [ミドルウェア](xref:fundamentals/middleware/index)は、C-core ベースの gRPC アプリのインターセプターと比較して、同様の機能を提供します。 ASP.NET Core ミドルウェアとインターセプターは概念的に似ています。 両方:

* gRPC 要求を処理するパイプラインを構築するために使用されます。
* パイプライン内の次のコンポーネントの前または後で作業が実行されます。
* `HttpContext` へのアクセスを提供します。
  * ミドルウェアでは、`HttpContext` はパラメーターです。
  * インターセプターでは、`ServerCallContext.GetHttpContext` 拡張メソッドで `ServerCallContext` パラメーターを使用して、`HttpContext` にアクセスできます。 この機能は、ASP.NET Core で実行されているインターセプターに固有です。

gRPC インターセプターと ASP.NET Core ミドルウェアの違い:

* インターセプター:
  * [ServerCallContext](https://grpc.io/grpc/csharp/api/Grpc.Core.ServerCallContext.html) を使用して、抽象化の gRPC レイヤーで動作します。
  * 次にアクセスできます。
    * 呼び出しに送信された逆シリアル化されたメッセージ。
    * シリアル化される前に、呼び出しから返されたメッセージ。
  * GRPC サービスからスローされた例外をキャッチして処理できます。
* ミドルウェア:
  * gRPC インターセプターの前に実行されます。
  * 基になる HTTP/2 メッセージを操作します。
  * 要求ストリームと応答ストリームからのバイトのみにアクセスできます。

## <a name="additional-resources"></a>その他の技術情報

* <xref:grpc/index>
* <xref:grpc/basics>
* <xref:grpc/aspnetcore>
* <xref:tutorials/grpc/grpc-start>
