---
title: C# を使用した gRPC サービス
author: juntaoluo
description: C# を使用した gRPC サービスを作成する際の基本的な概念について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: johluo
ms.date: 07/09/2020
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
uid: grpc/basics
ms.openlocfilehash: 37f3b0c902d272cf8970265c8e964a1b2a7ddf00
ms.sourcegitcommit: 7e593abc85602429d94a20603936a4725838aeea
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/04/2021
ms.locfileid: "106293438"
---
# <a name="grpc-services-with-c"></a>C\# を使用した gRPC サービス

このドキュメントでは、C# で [gRPC](https://grpc.io/docs/guides/) アプリを作成するために必要な概念を説明します。 ここで取り上げるトピックは、[C-core](https://grpc.io/blog/grpc-stacks)ベースと ASP.NET Core ベースの両方の gRPC アプリに適用されます。

[!INCLUDE[](~/includes/gRPCazure.md)]

## <a name="proto-file"></a>proto ファイル

gRPC では、API 開発に対してコントラクト優先のアプローチが使われます。 プロトコル バッファー (protobuf) は、既定でインターフェイス定義言語 (IDL) として使用されます。 *\*.proto* ファイルには次のものが含まれます。

* gRPC サービスの定義。
* クライアントとサーバー間で送信されるメッセージ。

protobuf ファイルの構文の詳細については、「<xref:grpc/protobuf>」を参照してください。

たとえば、[gRPC サービスの概要に関するページ](xref:tutorials/grpc/grpc-start)で使用されている *greet.proto* ファイルについて考えてみます。

* `Greeter` サービスを定義します。
* `Greeter` サービスで `SayHello` 呼び出しを定義します。
* `SayHello` では、`HelloRequest` メッセージを送信し、`HelloReply` メッセージを受信します。

[!code-protobuf[](~/tutorials/grpc/grpc-start/sample/GrpcGreeter/Protos/greet.proto)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

## <a name="add-a-proto-file-to-a-c-app"></a>C\# アプリに .proto ファイルを追加する

*\*.proto* ファイルは、`<Protobuf>` 項目グループにそれを追加することで、プロジェクトに含まれます。

[!code-xml[](~/tutorials/grpc/grpc-start/sample/GrpcGreeter/GrpcGreeter.csproj?highlight=2&range=7-9)]

既定で、`<Protobuf>` 参照によって、具象クライアントとサービス基本クラスが生成されます。 参照要素の `GrpcServices` 属性を使用して、C# アセットの生成を制限できます。 有効な `GrpcServices` オプションは次のとおりです。

* `Both` (存在しない場合の既定値)
* `Server`
* `Client`
* `None`

## <a name="c-tooling-support-for-proto-files"></a>.proto ファイルに対する C# ツール サポート

*\*.proto* ファイルから C# アセットを生成するために、ツール パッケージ [Grpc.Tools](https://www.nuget.org/packages/Grpc.Tools/) が必要です。 生成されるアセット (ファイル) は:

* プロジェクトがビルドされるたびに、必要に応じて生成されます。
* プロジェクトに追加されないか、ソース管理にチェックインされません。
* *obj* ディレクトリに格納されるビルド成果物です。

このパッケージは、サーバー プロジェクトとクライアント プロジェクトの両方で必要です。 `Grpc.AspNetCore` メタパッケージには、`Grpc.Tools` への参照が含まれます。 サーバー プロジェクトでは、Visual Studio の Package Manager を使用するか、プロジェクト ファイルに `<PackageReference>` を追加することによって、`Grpc.AspNetCore` を追加できます。

[!code-xml[](~/tutorials/grpc/grpc-start/sample/GrpcGreeter/GrpcGreeter.csproj?highlight=1&range=12)]

クライアント プロジェクトでは、gRPC クライアントを使用するために必要なその他のパッケージと共に、`Grpc.Tools` を直接参照する必要があります。 ツール パッケージは実行時に不要であるため、依存関係に `PrivateAssets="All"` でマークが付けられます。

[!code-xml[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/GrpcGreeterClient.csproj?highlight=3&range=9-14)]

## <a name="generated-c-assets"></a>生成された C# アセット

ツール パッケージは、含まれている *\*.proto* ファイルに定義されたメッセージを表す C# 型を生成します。

サーバー側アセットの場合、抽象サービスの基本型が生成されます。 基本型には、 *.proto* ファイルに含まれるすべての gRPC 呼び出しの定義が含まれます。 この基本型から派生し、gRPC 呼び出しのロジックを実装する具象サービス実装を作成します。 `greet.proto` の場合、前述の例では、仮想 `SayHello` メソッドを含む抽象 `GreeterBase` 型が生成されます。 具象実装 `GreeterService` は、メソッドをオーバーライドし、gRPC 呼び出しを処理するロジックを実装します。

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeter/Services/GreeterService.cs?name=snippet)]

クライアント側アセットの場合、具象クライアント型が生成されます。 *.proto* ファイル内の gRPC 呼び出しは、具象型のメソッドに変換され、これを呼び出すことができます。 `greet.proto` の場合、前述の例では、具象 `GreeterClient` 型が生成されます。 `GreeterClient.SayHelloAsync` を呼び出して、サーバーへの gRPC 呼び出しを開始します。

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/Program.cs?name=snippet)]

既定で、`<Protobuf>` 項目グループに含まれている各 *\*.proto* ファイルに対して、サーバーとクライアントのアセットが生成されます。 サーバー プロジェクトでサーバー アセットのみが生成されるようにするには、`GrpcServices` 属性を `Server` に設定します。

[!code-xml[](~/tutorials/grpc/grpc-start/sample/GrpcGreeter/GrpcGreeter.csproj?highlight=2&range=7-9)]

同様に、クライアントプロジェクトでは属性を `Client` に設定します。

## <a name="additional-resources"></a>その他の技術情報

* <xref:grpc/index>
* <xref:tutorials/grpc/grpc-start>
* <xref:grpc/aspnetcore>
* <xref:grpc/client>
