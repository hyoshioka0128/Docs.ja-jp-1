---
title: ASP.NET Core で .NET Core gRPC のクライアントとサーバーを作成する
author: juntaoluo
description: このチュートリアルでは、ASP.NET Core で gRPC サービスと gRPC クライアントを作成する方法を示します。 gRPC サービス プロジェクトの作成方法、proto ファイルの編集方法、二重ストリーミング呼び出しの追加方法について学習します。
ms.author: johluo
ms.date: 10/23/2020
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
uid: tutorials/grpc/grpc-start
ms.openlocfilehash: fed25eeacf57504810d41fcf002dcaa9927a21af
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588602"
---
# <a name="tutorial-create-a-grpc-client-and-server-in-aspnet-core"></a>チュートリアル: ASP.NET Core で gRPC のクライアントとサーバーを作成する

作成者: [John Luo](https://github.com/juntaoluo)

このチュートリアルでは、.NET Core [gRPC](https://grpc.io/docs/guides/) クライアントと ASP.NET Core gRPC サーバーを作成する方法を紹介します。

最終的に、gRPC あいさつサービスと通信する gRPC クライアントが与えられます。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/grpc/grpc-start/sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

このチュートリアルでは、次の作業を行いました。

> [!div class="checklist"]
> * gRPC サービスを作成する。
> * gRPC クライアントを作成します。
> * gRPC あいさつサービスで gRPC クライアント サービスをテストする。

## <a name="prerequisites"></a>必須コンポーネント

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.1.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* [Visual Studio for Mac バージョン 8.7 以降](/visualstudio/releasenotes/vs2019-mac-relnotes)
* [!INCLUDE [.NET Core 3.1 SDK](~/includes/3.1-SDK.md)]
---

## <a name="create-a-grpc-service"></a>gRPC サービスの作成

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Visual Studio を開始し、 **[新しいプロジェクトの作成]** を選択します。 または、Visual Studio の **[ファイル]** メニューから、 **[新規作成]**  >  **[プロジェクト]** の順に選択します。
* **[新しいプロジェクトの作成]** ダイアログで、 **[gRPC サービス]** を選択して、 **[次へ]** を選択します。

  ![Visual Studio の [新しいプロジェクトの作成] ダイアログ](~/tutorials/grpc/grpc-start/static/cnp.png)

* プロジェクトに **GrpcGreeter** という名前を付けます。 コードのコピーおよび貼り付けを行う際に名前空間が一致するように、プロジェクトに *GrpcGreeter* という名前を付けることが重要です。
* **[作成]** を選択します。
* **[Create a new gRPC service]\(新しい gPRC サービスの作成\)** ダイアログで、次のようにします。
  * **gRPC サービス** テンプレートが選択されています。
  * **[作成]** を選択します。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* [統合ターミナル](https://code.visualstudio.com/docs/editor/integrated-terminal)を開きます。
* ディレクトリ (`cd`) を、プロジェクトのフォルダーに変更します。
* 次のコマンドを実行します。

  ```dotnetcli
  dotnet new grpc -o GrpcGreeter
  code -r GrpcGreeter
  ```

  * `dotnet new` コマンドでは、*GrpcGreeter* フォルダー内に新しい gRPC サービスが作成されます。
  * `code` コマンドでは、Visual Studio Code の新しいインスタンス内に *GrpcGreeter* フォルダーが開かれます。

  "**ビルドとデバッグに必要な資産が 'GrpcGreeter' にありません。追加しますか?** " という内容のダイアログ ボックスが表示されたら、
* **[はい]** を選択します。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* Visual Studio for Mac を起動し、 **[新しいプロジェクトの作成]** を選択します。 または、Visual Studio の **[ファイル]** メニューから、 **[新規作成]**  >  **[プロジェクト]** の順に選択します。
* **[新しいプロジェクトの作成]** ダイアログで、 **[Web and Console]\(Web とコンソール\)** 、 **[アプリ]** 、 **[gRPC サービス]** の順に選択して **[次へ]** を選択します。

  ![macOS での [新しいプロジェクトの作成] ダイアログ](~/tutorials/grpc/grpc-start/static/cnp-mac.png)

* ターゲット フレームワークとして **[.NET Core 3.1]** を選択し、 **[次へ]** を選択します。
* プロジェクトに **GrpcGreeter** という名前を付けます。 コードのコピーおよび貼り付けを行う際に名前空間が一致するように、プロジェクトに *GrpcGreeter* という名前を付けることが重要です。
* **[作成]** を選択します。
---

### <a name="run-the-service"></a>サービスを実行する

  [!INCLUDE[](~/includes/run-the-app.md)]

サービスが `https://localhost:5001` でリッスンしていることがログに示されます。

```console
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: https://localhost:5001
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
```

> [!NOTE]
> gRPC テンプレートは[トランスポート層セキュリティ (TLS)](https://tools.ietf.org/html/rfc5246) を使用するように構成されています。 gRPC クライアントでは、HTTPS を使用してサーバーを呼び出す必要があります。
>
> macOS の場合、ASP.NET Core gRPC と TLS の組み合わせに対応していません。 macOS で gRPC サービスを正常に実行するには、追加の構成が必要です。 詳細については、[macOS で ASP.NET Core gRPC アプリを起動できない](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos)場合に関するページを参照してください。

### <a name="examine-the-project-files"></a>プロジェクト ファイルを確認する

*GrpcGreeter* プロジェクト ファイル:

* *greet.proto*:*Protos/greet.proto* ファイルは、`Greeter` gRPC を定義し、gRPC サーバー資産を生成するために使用されます。 詳細については、「[gRPC の概要](xref:grpc/index)」を参照してください。
* *Services* フォルダー:`Greeter` サービスの実装が含まれます。
* *appSettings.json*:Kestrel で使用されるプロトコルなどの構成データが含まれています。 詳細については、「<xref:fundamentals/configuration/index>」を参照してください。
* *Program.cs*:gRPC サービスのエントリ ポイントが含まれています。 詳細については、「<xref:fundamentals/host/generic-host>」を参照してください。
* *Startup.cs*:アプリの動作を構成するコードが含まれています。 詳細については、[アプリの Startup](xref:fundamentals/startup)に関するページを参照してください。

## <a name="create-the-grpc-client-in-a-net-console-app"></a>.NET コンソール アプリで gRPC クライアントを作成する

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Visual Studio のインスタンスをもう 1 つ開き、 **[新しいプロジェクトの作成]** を選択します。
* **[新しいプロジェクトの作成]** ダイアログで、 **[コンソール アプリ (.NET Core)]** を選択し、 **[次へ]** を選択します。
* **[プロジェクト名]** テキスト ボックスに「**GrpcGreeterClient**」を入力し、 **[作成]** を選択します。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* [統合ターミナル](https://code.visualstudio.com/docs/editor/integrated-terminal)を開きます。
* ディレクトリ (`cd`) を、プロジェクトのフォルダーに変更します。
* 次のコマンドを実行します。

  ```dotnetcli
  dotnet new console -o GrpcGreeterClient
  code -r GrpcGreeterClient
  ```

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

「[Visual Studio for Mac を使用した macOS での完全な .NET Core ソリューションの構築](/dotnet/core/tutorials/using-on-mac-vs-full-solution)」の手順に従って、*GrpcGreeterClient* という名前のコンソール アプリを作成します。

---

### <a name="add-required-packages"></a>必要なパッケージを追加する

gRPC クライアント プロジェクトには、次のパッケージが必要です。

* .NET Core のクライアントを含む [Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client)。
* [Google.Protobuf](https://www.nuget.org/packages/Google.Protobuf/)。これに C# の protobuf メッセージ API が含まれています。
* [Grpc.Tools](https://www.nuget.org/packages/Grpc.Tools/)。これには protobuf ファイルの C# ツール サポートが含まれています。 ツール パッケージは実行時に不要であり、依存関係には `PrivateAssets="All"` のマークが付きます。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

パッケージ マネージャー コンソール (PMC) または NuGet パッケージの管理を使用してパッケージをインストールします。

#### <a name="pmc-option-to-install-packages"></a>パッケージをインストールするための PMC オプション

* Visual Studio で **[ツール]**  >  **[NuGet パッケージ マネージャー]**  >  **[パッケージ マネージャー コンソール]** の順に選択します。
* **[パッケージ マネージャー コンソール]** ウィンドウから `cd GrpcGreeterClient` を実行し、*GrpcGreeterClient.csproj* ファイルが含まれるフォルダーにディレクトリを変更します。
* 次のコマンドを実行します。

  ```powershell
  Install-Package Grpc.Net.Client
  Install-Package Google.Protobuf
  Install-Package Grpc.Tools
  ```

#### <a name="manage-nuget-packages-option-to-install-packages"></a>パッケージをインストールするための [NuGet パッケージの管理] オプション

* **[ソリューション エクスプローラー]**  >  **[NuGet パッケージの管理]** でプロジェクトを右クリックします。
* **[参照]** タブを選択します。
* 検索ボックスに「**Grpc.Net.Client**」と入力します。
* **[参照]** タブから **Grpc.Net.Client** パッケージを選択し、 **[インストール]** を選択します。
* `Google.Protobuf` と `Grpc.Tools` に同じ手順を繰り返します。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

**統合ターミナル** から次のコマンドを実行します。

```dotnetcli
dotnet add GrpcGreeterClient.csproj package Grpc.Net.Client
dotnet add GrpcGreeterClient.csproj package Google.Protobuf
dotnet add GrpcGreeterClient.csproj package Grpc.Tools
```

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* **[Solution Pad]** で **[GrpcGreeterClient]** プロジェクトを右クリックし、 **[NuGet パッケージの管理]** を選択します。
* 検索ボックスに「**Grpc.Net.Client**」と入力します。
* 結果ペインから **Grpc.Net.Client** パッケージを選択し、 **[パッケージを追加]** を選択します。
* **[Accept License]\(ライセンスに同意する\)** ダイアログの **[承諾]** ボタンを選択します。
* `Google.Protobuf` と `Grpc.Tools` に同じ手順を繰り返します。

---

### <a name="add-greetproto"></a>greet.proto を追加する

* gRPC クライアント プロジェクトで *Protos* フォルダーを作成します。
* gRPC あいさつサービスから gRPC クライアント プロジェクトに *Protos\greet.proto* ファイルをコピーします。
* `greet.proto` ファイル内の名前空間を、プロジェクトの名前空間に更新します。

  ```
  option csharp_namespace = "GrpcGreeterClient";
  ```

* *GrpcGreeterClient.csproj* プロジェクト ファイルを編集します。

  # <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

  プロジェクトを右クリックし、 **[プロジェクト ファイルの編集]** を選択します。

  # <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

  *GrpcGreeterClient.csproj* ファイルを選択します。

  # <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

  プロジェクトを右クリックし、 **[プロジェクト ファイルの編集]** を選択します。

  ---

* *greet.proto* ファイルを参照する `<Protobuf>` 要素で項目グループを追加します。

  ```xml
  <ItemGroup>
    <Protobuf Include="Protos\greet.proto" GrpcServices="Client" />
  </ItemGroup>
  ```

### <a name="create-the-greeter-client"></a>Greeter クライアントを作成する

クライアント プロジェクトをビルドして、`GrpcGreeter` 名前空間内に型を作成します。 `GrpcGreeter` 型は、ビルド プロセスによって自動的に生成されます。

次のコードを使用して、gRPC クライアントの *Program.cs* ファイルを更新します。

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/Program.cs?name=snippet2)]

*Program.cs* には、gRPC クライアントのエントリ ポイントとロジックが含まれています。

Greeter クライアントは、次の方法で作成されます。

* gRPC サービスへの接続を作成するための情報が含まれている `GrpcChannel` をインスタンス化する。
* `GrpcChannel` を使用して、Greeter クライアントを構築します。

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/Program.cs?name=snippet&highlight=3-5)]

Greeter クライアントから非同期の `SayHello` メソッドが呼び出されます。 `SayHello` 呼び出しの結果が表示されます。

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/Program.cs?name=snippet&highlight=6-8)]

## <a name="test-the-grpc-client-with-the-grpc-greeter-service"></a>gRPC あいさつサービスで gRPC クライアントをテストする

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* あいさつサービスで、`Ctrl+F5` キーを押して、デバッガーなしでサーバーを起動します。
* `GrpcGreeterClient` プロジェクトで、`Ctrl+F5` 押してデバッガーなしでクライアントを起動します。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* あいさつサービスを開始します。
* クライアントを起動します。


# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* 前述の [macOS の HTTP/2 TLS 問題の回避策](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos)により、クライアントのチャンネル アドレスを "http://localhost:5000" に更新する必要があります。 **GrpcGreeterClient/Program.cs** の 13 行目を次のように更新します。
  ```csharp
  using var channel = GrpcChannel.ForAddress("http://localhost:5000");
  ``` 
* あいさつサービスを開始します。
* クライアントを起動します。

---

クライアントにより、その名前 *GreeterClient* が含まれるあいさつメッセージが、サービスに送信されます。 サービスから応答として "Hello GreeterClient" のメッセージが送信されます。 "Hello GreeterClient" の応答がコマンド プロンプトに表示されます。

```console
Greeting: Hello GreeterClient
Press any key to exit...
```

gRPC サービスにより、成功した呼び出しの詳細が、コマンド プロンプトに書き込まれるログに記録されます。

```console
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: https://localhost:5001
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: C:\GH\aspnet\docs\4\Docs\aspnetcore\tutorials\grpc\grpc-start\sample\GrpcGreeter
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/2 POST https://localhost:5001/Greet.Greeter/SayHello application/grpc
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint 'gRPC - /Greet.Greeter/SayHello'
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[1]
      Executed endpoint 'gRPC - /Greet.Greeter/SayHello'
info: Microsoft.AspNetCore.Hosting.Diagnostics[2]
      Request finished in 78.32260000000001ms 200 application/grpc
```

> [!NOTE]
> この記事のコードでは、gRPC サービスをセキュリティで保護するために、ASP.NET Core HTTPS 開発証明書が必要です。 .NET gRPC クライアントが `The remote certificate is invalid according to the validation procedure.` または `The SSL connection could not be established.` というメッセージで失敗する場合、その開発証明書は信頼されていません。 この問題を解決するには、「[信頼されていないか無効な証明書で gRPC サービスを呼び出す](xref:grpc/troubleshoot#call-a-grpc-service-with-an-untrustedinvalid-certificate)」を参照してください。

[!INCLUDE[](~/includes/gRPCazure.md)]

### <a name="next-steps"></a>次の手順

* <xref:grpc/index>
* <xref:grpc/basics>
* <xref:grpc/migration>
