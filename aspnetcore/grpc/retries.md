---
title: gRPC の再試行による一時的障害の処理
author: jamesnk
description: .NET で再試行を使用して、回復性がありフォールト トレラントな gRPC 呼び出しを行う方法について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 02/25/2021
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
uid: grpc/retries
ms.openlocfilehash: 613386d1fedd8b1b04081e3240b8a3aaf7b37012
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589788"
---
# <a name="transient-fault-handling-with-grpc-retries"></a>gRPC の再試行による一時的障害の処理

作成者: [James Newton-King](https://twitter.com/jamesnk)

gRPC 再試行の機能を使用すると、gRPC クライアントで失敗した呼び出しの再試行を自動的に行うことができます。 この記事では、.NET で再試行ポリシーを構成することにより、回復性がありフォールト トレラントな gRPC アプリを作成する方法について説明します。

gRPC の再試行には、[Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client) バージョン 2.36.0-pre1 以降が必要です。

## <a name="transient-fault-handling"></a>一時的な障害の処理

gRPC の呼び出しは、一時的な障害によって中断される場合があります。 一時的な障害には次のものがあります。

* ネットワーク接続の瞬間的な喪失。
* サービスの一時的な利用不能。
* サーバーの負荷によるタイムアウト。

gRPC の呼び出しが中断されると、エラーに関する詳細情報が含まれる `RpcException` がクライアントによってスローされます。 クライアント アプリでその例外をキャッチし、エラーの処理方法を選択する必要があります。

```csharp
var client = new Greeter.GreeterClient(channel);
try
{
    var response = await client.SayHelloAsync(
        new HelloRequest { Name = ".NET" });

    Console.WriteLine("From server: " + response.Message);
}
catch (RpcException ex)
{
    // Write logic to inspect the error and retry
    // if the error is from a transient fault.
}
```

アプリ全体に再試行ロジックを複製すると、冗長になり、エラーが発生しやすくなります。 さいわい、.NET の gRPC クライアントには自動再試行のサポートが組み込まれています。

## <a name="configure-a-grpc-retry-policy"></a>gRPC 再試行ポリシーを構成する

再試行ポリシーは、gRPC チャネルが作成されるときに 1 回構成されます。

```csharp
var defaultMethodConfig = new MethodConfig
{
    Names = { MethodName.Default },
    RetryPolicy = new RetryPolicy
    {
        MaxAttempts = 5,
        InitialBackoff = TimeSpan.FromSeconds(1),
        MaxBackoff = TimeSpan.FromSeconds(5),
        BackoffMultiplier = 1.5,
        RetryableStatusCodes = { StatusCode.Unavailable }
    }
};

var channel = GrpcChannel.ForAddress("https://localhost:5001", new GrpcChannelOptions
{
    ServiceConfig = new ServiceConfig { MethodConfigs = { defaultMethodConfig } }
});
```

上記のコードでは次の操作が行われます。

* `MethodConfig` を作成します。 再試行ポリシーはメソッドごとに構成でき、メソッドは `Names` プロパティを使用して照合されます。 このメソッドは `MethodName.Default` で構成されているため、このチャネルによって呼び出されるすべての gRPC メソッドに適用されます。
* 再試行ポリシーを構成します。 このポリシーにより、状態コード `Unavailable` で失敗した gRPC 呼び出しを自動的に再試行することがクライアントに指示されます。
* `GrpcChannelOptions.ServiceConfig` を設定することにより、作成されたチャネルで再試行ポリシーが使用されるように構成します。

そのチャネルで作成される gRPC クライアントは、失敗した呼び出しを自動的に再試行するようになります。

```csharp
var client = new Greeter.GreeterClient(channel);
var response = await client.SayHelloAsync(
    new HelloRequest { Name = ".NET" });

Console.WriteLine("From server: " + response.Message);
```

### <a name="grpc-retry-options"></a>gRPC 再試行のオプション

次の表では、gRPC 再試行ポリシーを構成するためのオプションについて説明します。

| オプション | 説明 |
| ------ | ----------- |
| `MaxAttempts` | 呼び出しの試行の最大回数 (元の試行を含みます)。 この値は `GrpcChannelOptions.MaxRetryAttempts` によって制限され、その既定値は 5 です。 値は必須で、1 より大きい必要があります。 |
| `InitialBackoff` | 再試行の間隔の初期バックオフ遅延。 0 から現在のバックオフまでの間のランダムな遅延により、次の再試行が行われるタイミングが決まります。 各試行の後で、現在のバックオフに `BackoffMultiplier` が乗算されます。 値は必須で、0 より大きい必要があります。 |
| `MaxBackoff` | 最大バックオフにより、エクスポネンシャル バックオフの増加の上限が設定されます。 値は必須で、0 より大きい必要があります。 |
| `BackoffMultiplier` | バックオフは、各再試行の後でこの値を乗算され、乗数が 1 より大きい場合は指数関数的に増加します。 値は必須で、0 より大きい必要があります。 |
| `RetryableStatusCodes` | 状態コードのコレクション。 一致する状態で失敗した gRPC 呼び出しは、自動的に再試行されます。 状態コードの詳細については、「[gRPC での状態コードとその使用](https://grpc.github.io/grpc/core/md_doc_statuscodes.html)」を参照してください。 再試行可能な状態コードが少なくとも 1 つ必要です。 |

## <a name="hedging"></a>ヘッジング

ヘッジングは、別の再試行戦略です。 ヘッジングを使用すると、1 つの gRPC 呼び出しの複数のコピーを、応答を待たずに積極的に送信できます。 ヘッジされた gRPC 呼び出しは、サーバー上で複数回実行される可能性があり、最初に成功した結果が使用されます。 ヘッジングは、複数回実行しても悪影響のない安全なメソッドに対してのみ有効にすることが重要です。

再試行と比較して、ヘッジングには長所と短所があります。 

* ヘッジングの利点は、より早く成功結果が返される可能性があることです。 複数の gRPC 呼び出しを同時に実行でき、最初の成功結果が得られた時点で完了します。 
* ヘッジングの欠点は、無駄になるおそれがあることです。 複数の呼び出しが行われ、すべてが成功する場合があります。 最初の結果のみが使用され、残りは破棄されます。

## <a name="configure-a-grpc-hedging-policy"></a>gRPC のヘッジング ポリシーを構成する

ヘッジング ポリシーは、再試行ポリシーのように構成されます。 ヘッジング ポリシーを再試行ポリシーと組み合わせることはできないことにご注意ください。

```csharp
var defaultMethodConfig = new MethodConfig
{
    Names = { MethodName.Default },
    HedgingPolicy = new HedgingPolicy
    {
        MaxAttempts = 5,
        NonFatalStatusCodes = { StatusCode.Unavailable }
    }
};

var channel = GrpcChannel.ForAddress("https://localhost:5001", new GrpcChannelOptions
{
    ServiceConfig = new ServiceConfig { MethodConfigs = { defaultMethodConfig } }
});
```

### <a name="grpc-hedging-options"></a>gRPC のヘッジング オプション

次の表では、gRPC ヘッジング ポリシーを構成するためのオプションについて説明します。

| オプション | 説明 |
| ------ | ----------- |
| `MaxAttempts` | ヘッジング ポリシーにより、最大この回数まで呼び出しが送信されます。 `MaxAttempts` は、すべての試行の合計回数を表します (元の試行を含みます)。 この値は `GrpcChannelOptions.MaxRetryAttempts` によって制限され、その既定値は 5 です。 値は必須で、2 以上である必要があります。 |
| `HedgingDelay` | 最初の呼び出しは直ちに送信されますが、後続のヘッジング呼び出しはこの値だけ遅延されます。 遅延を 0 または `null` に設定すると、ヘッジされたすべての呼び出しが直ちに送信されます。 既定値は 0 です。 |
| `NonFatalStatusCodes` | 他のヘッジの呼び出しがまだ成功する可能性があることを示す状態コードのコレクション。 サーバーから致命的ではない状態コードが返された場合、ヘッジされた呼び出しは続行されます。 それ以外の場合は、未処理の要求は取り消され、エラーがアプリに返されます。 状態コードの詳細については、「[gRPC での状態コードとその使用](https://grpc.github.io/grpc/core/md_doc_statuscodes.html)」を参照してください。 |

## <a name="additional-resources"></a>その他のリソース

* <xref:grpc/client>
* [再試行の一般的なガイダンス - クラウド アプリケーションのためのベスト プラクティス](/azure/architecture/best-practices/transient-faults)
