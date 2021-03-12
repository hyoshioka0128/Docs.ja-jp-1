---
title: ASP.NET Core での HTTP.sys Web サーバーの実装
author: rick-anderson
description: Windows 上の ASP.NET Core 用 Web サーバーである HTTP.sys について説明します。 HTTP.sys は、Http.sys カーネル モード ドライバーに基づいて構築された、IIS なしで直接インターネットに接続するために使用できる Kestrel の代替製品です。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/07/2020
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
uid: fundamentals/servers/httpsys
ms.openlocfilehash: e44cdcb7e427c1ae2531c452a7c8b49e104b3d11
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586073"
---
# <a name="httpsys-web-server-implementation-in-aspnet-core"></a>ASP.NET Core での HTTP.sys Web サーバーの実装

作成者: [Tom Dykstra](https://github.com/tdykstra)、[Chris Ross](https://github.com/Tratcher)

::: moniker range=">= aspnetcore-3.1"

[HTTP.sys](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture#hypertext-transfer-protocol-stack-httpsys) は、Windows 上でのみ動作する [ASP.NET Core 用 Web サーバー](xref:fundamentals/servers/index)です。 HTTP.sys は [Kestrel](xref:fundamentals/servers/kestrel) サーバーの代替製品であり、Kestrel では提供されていない機能がいくつか用意されています。

> [!IMPORTANT]
> HTTP.sys は [ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)と互換性がなく、IIS や IIS Express で使用することはできません。

HTTP.sys は、次の機能をサポートします。

* [Windows 認証](xref:security/authentication/windowsauth)
* ポート共有
* SNI を使用する HTTPS
* HTTP/2 over TLS (Windows 10 以降)
* 直接ファイル伝送
* 応答キャッシュ
* WebSocket (Windows 8 以降)

サポートされている Windows バージョン:

* Windows 7 以降
* Windows Server 2008 R2 以降

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/httpsys/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="when-to-use-httpsys"></a>HTTP.sys を使用するタイミング

HTTP.sys は、次のような展開に適しています。

* IIS を使用せず、インターネットに直接サーバーを公開する必要がある。

  ![インターネットと直接通信する HTTP.sys](httpsys/_static/httpsys-to-internet.png)

* 内部の展開から Kestrel では使用できない機能が要求されている。 詳細については、このドキュメントの「[Kestrel と HTTP.sys](xref:fundamentals/servers/index#kestrel-vs-httpsys)

  ![内部ネットワークと直接通信する HTTP.sys](httpsys/_static/httpsys-to-internal.png)

HTTP.sys は、さまざまな種類の攻撃を防ぎ、フル機能の Web サーバーとして堅牢性、セキュリティ、スケーラビリティを提供する、成熟したテクノロジです。 IIS 自体が、HTTP.sys 上で HTTP リスナーとして実行されています。

## <a name="http2-support"></a>HTTP/2 のサポート

[Http/2](https://httpwg.org/specs/rfc7540.html) は、次の基本要件が満たされている場合に、ASP.NET Core アプリに対して有効になります。

* Windows Server 2016/Windows 10 以降
* [アプリケーション レイヤー プロトコル ネゴシエーション (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) 接続
* TLS 1.2 以降の接続

Http/2 接続が確立されると、[HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) が `HTTP/2` を報告します。

HTTP/2 は既定で有効になっています。 Http/2 接続が確立されない場合、接続は http/1.1 にフォールバックします。 Windows の今後のリリースで、HTTP.sys で HTTP/2 を無効にする機能を含む HTTP/2 構成フラグが使用可能になる予定です。

## <a name="kernel-mode-authentication-with-kerberos"></a>Kerberos を使用したカーネル モード認証

HTTP.sys では、Kerberos 認証プロトコルを使用したカーネル モード認証に処理が委任されます。 Kerberos および HTTP.sys ではユーザー モード認証がサポートされていません。 Active Directory から取得され、クライアントによって、ユーザーを認証するサーバーに転送される Kerberos トークン/チケットを暗号化解除するには、コンピューター アカウントを使用する必要があります。 アプリのユーザーではなく、ホストのサービス プリンシパル名 (SPN) を登録します。

## <a name="how-to-use-httpsys"></a>HTTP.sys の使用方法

### <a name="configure-the-aspnet-core-app-to-use-httpsys"></a>HTTP.sys を使用するように ASP.NET Core アプリを構成する

ホストを構築するときに <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderHttpSysExtensions.UseHttpSys*> 拡張メソッドを呼び出し、必要な <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions> を指定します。 次の例では、既定値にオプションを設定しています。

[!code-csharp[](httpsys/samples/3.x/SampleApp/Program.cs?name=snippet1&highlight=5-13)]

HTTP.sys の追加の構成は、[レジストリ設定](https://support.microsoft.com/help/820129/http-sys-registry-settings-for-windows)を通じて処理されます。

**HTTP.sys オプション**

| プロパティ | 説明 | Default |
| -------- | ----------- | :-----: |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.AllowSynchronousIO> | `HttpContext.Request.Body` および `HttpContext.Response.Body` に対して、入力/出力の同期を許可するかどうかを制御します。 | `false` |
| [Authentication.AllowAnonymous](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.AllowAnonymous) | 匿名要求を許可します。 | `true` |
| [Authentication.Schemes](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.Schemes) | 許可される認証方式を指定します。 リスナーを破棄する前ならいつでも変更できます。 値は [AuthenticationSchemes 列挙型](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationSchemes) (`Basic`、`Kerberos`、`Negotiate`、`None`、および `NTLM`) によって指定します。 | `None` |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.EnableResponseCaching> | 対象となるヘッダーを持つ応答に対して、[カーネル モード](/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode)のキャッシュを試行します。 `Set-Cookie`、`Vary`、または `Pragma` ヘッダーを含む応答は対象外です。 応答は、`public` である `Cache-Control` ヘッダーと `shared-max-age` または `max-age` の値のいずれかを含むか、または `Expires` ヘッダーを含む必要があります。 | `true` |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.Http503Verbosity> | 調整の条件によって要求を拒否する場合の HTTP.sys の動作。 | [Http503VerbosityLevel.<br>Basic](xref:Microsoft.AspNetCore.Server.HttpSys.Http503VerbosityLevel) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxAccepts> | 同時受け入れの最大数です。 | 5 &times;[Environment.<br>ProcessorCount](xref:System.Environment.ProcessorCount) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxConnections> | 受け入れるコンカレント接続の最大数です。 無限にするには、`-1` を使用します。 コンピューター全体のレジストリ設定を使用するには、`null` を使用します。 | `null`<br>(コンピューター全体の<br>設定) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> | 「<a href="#maxrequestbodysize">MaxRequestBodySize</a>」セクションを参照してください。 | 30000000 バイト<br>(~28.6 MB) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.RequestQueueLimit> | キューに置くことができる要求の最大数。 | 1000 |
| `RequestQueueMode` | これは、サーバーで要求キューの作成と構成を行う必要があるかどうか、または既存のキューにアタッチする必要があるかを示します。<br>既存のキューにアタッチする場合、既存の構成オプションのほとんどは適用されません。 | `RequestQueueMode.Create` |
| `RequestQueueName` | HTTP.sys 要求キューの名前。 | `null` (匿名キュー) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.ThrowWriteExceptions> | 応答本文の書き込みがクライアントの接続の切断によって失敗した場合、例外をスローするか、または正常に完了するかどうかを指定します。 | `false`<br>(正常に完了する) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.Timeouts> | HTTP.sys <xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager> 構成を公開します。これはレジストリでも構成できます。 各設定に関する既定値などの詳細については、API のリンクを参照してください。<ul><li><xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.DrainEntityBody?displayProperty=nameWithType>:HTTP サーバー API が Keep-Alive 接続でエンティティ本体をドレインするまでに許容される時間です。</li><li><xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.EntityBody?displayProperty=nameWithType>:要求のエンティティ本体が到着するまでに許容される時間。</li><li><xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.HeaderWait?displayProperty=nameWithType>:HTTP サーバー API が要求ヘッダーを解析するまでに許容される時間。</li><li><xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.IdleConnection?displayProperty=nameWithType>:アイドル接続で許容される時間。</li><li><xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.MinSendBytesPerSecond?displayProperty=nameWithType>:応答の最小送信レート。</li><li><xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.RequestQueue?displayProperty=nameWithType>:要求が、アプリにピック アップされるまでに要求キューの中に留まっていられる時間。</li></ul> |  |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> | HTTP.sys に登録する <xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection> を指定します。 最も便利なのは <xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection.Add%2A?displayProperty=nameWithType> です。これを使用して、コレクションにプレフィックスを追加できます。 これらは、リスナーを破棄する前ならいつでも変更できます。 |  |

<a name="maxrequestbodysize"></a>

**MaxRequestBodySize**

要求本文の最大許容サイズ (バイト単位) です。 `null` に設定する場合、要求本文の最大サイズは制限されません。 この制限は、アップグレード済みの接続 (常に無制限) には影響しません。

1 つの `IActionResult` に対する ASP.NET Core MVC アプリの制限をオーバーライドする方法として、アクション メソッドに対して <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> 属性を使用することをお勧めします。

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

アプリが要求の読み取りを開始した後に、アプリが要求に対する制限を構成しようとすると、例外がスローされます。 `IsReadOnly` プロパティを使用して、`MaxRequestBodySize` プロパティが読み取り専用状態にあるかどうか、つまり制限を構成するには遅すぎるかどうかを示すことができます。

要求ごとにアプリで <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> をオーバーライドする必要がある場合は、<xref:Microsoft.AspNetCore.Http.Features.IHttpMaxRequestBodySizeFeature> を使います。

[!code-csharp[](httpsys/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=6-7)]

Visual Studio を使用する場合は、アプリが IIS または IIS Express を実行するように構成されていないことを確認します。

Visual Studio では、既定の起動プロファイルは IIS Express 用です。 プロジェクトをコンソール アプリとして実行するには、次のスクリーン ショットに示すように、選択したプロファイルを手動で変更します。

![コンソール アプリのプロファイルを選択する](httpsys/_static/vs-choose-profile.png)

### <a name="configure-windows-server"></a>Windows Server を構成する

1. アプリに対して開くポートを決めたら、[Windows ファイアウォール](/windows/security/threat-protection/windows-firewall/create-an-inbound-port-rule)か [New-NetFirewallRule](/powershell/module/netsecurity/new-netfirewallrule) PowerShell コマンドレットを使用して、トラフィックが HTTP.sys に到達できるようにファイアウォールのポートを開きます。 次のコマンドとアプリの構成では、ポート 443 を使用します。

1. Azure VM に展開する場合は、[ネットワーク セキュリティ グループ](/azure/virtual-machines/windows/nsg-quickstart-portal)内でポートを開きます。 次のコマンドとアプリの構成では、ポート 443 を使用します。

1. 必要に応じて、X.509 証明書を取得してインストールします。

   Windows の場合は、[New-SelfSignedCertificate PowerShell コマンドレット](/powershell/module/pkiclient/new-selfsignedcertificate)を使用して自己署名証明書を作成します。 サポート対象外の例については、[UpdateIISExpressSSLForChrome.ps1](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/includes/make-x509-cert/UpdateIISExpressSSLForChrome.ps1) を参照してください。

   自己署名証明書か CA 署名証明書のいずれかをサーバーの **Local Machine**>**Personal** ストアにインストールします。

1. アプリが[フレームワークに依存する展開](/dotnet/core/deploying/#framework-dependent-deployments-fdd)である場合は、.NET Core、.NET Framework、またはその両方 (アプリが .NET Framework をターゲットとする .NET Core アプリである場合) をインストールします。

   * **.NET Core**: アプリで .NET Core が必要な場合は、[.NET Core のダウンロード](https://dotnet.microsoft.com/download)から **.NET Core Runtime** インストーラーを取得して実行します。 サーバーに SDK 全体をインストールしないでください。
   * **.NET Framework**:アプリで .NET Framework が必要な場合は、[.NET Framework のインストール ガイド](/dotnet/framework/install/)を参照してください。 必要な .NET Framework をインストールします。 最新の .NET Framework のインストーラーは [.NET Core のダウンロード](https://dotnet.microsoft.com/download) ページから入手できます。

   アプリが[自己完結型の展開](/dotnet/core/deploying/#self-contained-deployments-scd)の場合、アプリの展開内にランタイムが含まれています。 サーバーにフレームワークをインストールする必要はありません。

1. アプリに URL とポートを構成します。

   既定では、ASP.NET Core は `http://localhost:5000` にバインドされます。 URL プレフィックスとポートを構成するには、次のオプションがあります。

   * <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
   * `urls` コマンド ライン引数
   * `ASPNETCORE_URLS` 環境変数
   * <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes>

   次のコード例は、サーバーのローカル IP アドレス `10.0.0.4` を使ってポート 443 上で <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> を使う方法を示しています。

   [!code-csharp[](httpsys/samples_snapshot/3.x/Program.cs?highlight=7)]

   `UrlPrefixes` の利点は、プレフィックスの形式が正しくなかった場合、すぐにエラー メッセージが生成されることです。

   `UrlPrefixes` の設定は `UseUrls`/`urls`/`ASPNETCORE_URLS` の設定をオーバーライドします。 したがって、`UseUrls`、`urls`、および `ASPNETCORE_URLS` 環境変数の利点は、Kestrel と HTTP.sys を簡単に切り替えられることです。

   HTTP.sys では、[HTTP サーバー API の UrlPrefix 文字列形式](/windows/win32/http/urlprefix-strings)が使用されます。

   > [!WARNING]
   > 最上位のワイルドカードのバインド ( `http://*:80/` と `http://+:80` ) は使用しては **いけません** 。 最上位のワイルドカードのバインドを使用すると、アプリにセキュリティの脆弱性が生じます。 これは、強力と脆弱の両方のワイルドカードに適用されます。 ワイルドカードではなく、明示的なホスト名か IP アドレスを使用してください。 親ドメイン全体を制御する場合、サブドメインのワイルドカードのバインド (たとえば、`*.mysub.com`) がセキュリティ リスクになることはありません (脆弱である `*.com` とは対照的)。 詳細については、[RFC 7230:セクション 5.4:ホスト](https://tools.ietf.org/html/rfc7230#section-5.4)に関するページを参照してください。

1. サーバーで URL プレフィックスを事前登録します。

   HTTP.sys を構成するための組み込みツールは、*netsh.exe* です。 *netsh.exe* を使用して、URL プレフィックスを予約し、X.509 証明書を割り当てることができます。 ツールを使用するには管理者特権が必要です。

   *netsh.exe* ツールを使用して、アプリ用に URL を登録します。

   ```console
   netsh http add urlacl url=<URL> user=<USER>
   ```

   * `<URL>`:完全修飾 URL (Uniform Resource Locator)。 ワイルドカードのバインドは使用しないでください。 有効なホスト名かローカル IP アドレスを使用してください。 "*URL の末尾にはスラッシュが必要です。* "
   * `<USER>`:ユーザーまたはユーザー グループの名前を指定します。

   次の例では、サーバーのローカル IP アドレスは `10.0.0.4` です。

   ```console
   netsh http add urlacl url=https://10.0.0.4:443/ user=Users
   ```

   URL が登録されると、ツールから `URL reservation successfully added` という応答があります。

   登録済みの URL を削除するには、`delete urlacl` コマンドを使用します。

   ```console
   netsh http delete urlacl url=<URL>
   ```

1. サーバーで X.509 証明書を登録します。

   *netsh.exe* ツールを使用して、アプリ用の証明書を登録します。

   ```console
   netsh http add sslcert ipport=<IP>:<PORT> certhash=<THUMBPRINT> appid="{<GUID>}"
   ```

   * `<IP>`:バインド用のローカル IP アドレスを指定します。 ワイルドカードのバインドは使用しないでください。 有効な IP アドレスを使用してください。
   * `<PORT>`:バインド用のポートを指定します。
   * `<THUMBPRINT>`:X.509 証明書の拇印です。
   * `<GUID>`:情報提供を目的として開発者によって生成された、アプリを表す GUID です。

   参照用に、この GUID をパッケージ タグとしてアプリに格納します。

   * Visual Studio:
     * **ソリューション エクスプローラー** 内でアプリを右クリックし、 **[プロパティ]** をクリックして、アプリのプロジェクト プロパティを開きます。
     * **[パッケージ]** タブを選択します。
     * 作成した GUID を **[タグ]** フィールドに入力します。
   * Visual Studio を使用しない場合:
     * アプリのプロジェクト ファイルを開きます。
     * 作成した GUID を指定した `<PackageTags>` プロパティを、新規または既存の `<PropertyGroup>` に追加します。

       ```xml
       <PropertyGroup>
         <PackageTags>9412ee86-c21b-4eb8-bd89-f650fbf44931</PackageTags>
       </PropertyGroup>
       ```

   次に例を示します。

   * サーバーのローカル IP アドレスは `10.0.0.4` です。
   * オンラインのランダム GUID ジェネレーターによって、`appid` の値が提供されます。

   ```console
   netsh http add sslcert 
       ipport=10.0.0.4:443 
       certhash=b66ee04419d4ee37464ab8785ff02449980eae10 
       appid="{9412ee86-c21b-4eb8-bd89-f650fbf44931}"
   ```

   証明書が登録されると、ツールから `SSL Certificate successfully added` という応答があります。

   証明書の登録を削除するには、`delete sslcert` コマンドを使用します。

   ```console
   netsh http delete sslcert ipport=<IP>:<PORT>
   ```

   以下は、*netsh.exe* のリファレンス ドキュメントです。

   * [Netsh Commands for Hypertext Transfer Protocol (HTTP)](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc725882(v=ws.10)) (ハイパーテキスト転送プロトコル (HTTP) 用の Netsh コマンド)
   * [UrlPrefix Strings](/windows/win32/http/urlprefix-strings) (UrlPrefix 文字列)

1. アプリを実行します。

   1024 より大きいポート番号で (HTTPS ではなく) HTTP を使用して localhost にバインドする場合、アプリの実行に管理者権限は必要ありません。 その他の構成の場合 (たとえば、ローカル IP アドレスを使用する場合やポート 443 にバインドする場合)、管理者権限でアプリを実行します。

   サーバーのパブリック IP アドレスでアプリが応答します。 この例では、サーバーは自身のパブリック IP アドレス `104.214.79.47` でインターネットからアクセスされます。

   この例では開発証明書が使用されています。 証明書が信頼できないというブラウザーの警告がバイパスされた後に、ページが安全に読み込まれます。

   ![読み込まれたアプリのインデックス ページを表示するブラウザー ウィンドウ](httpsys/_static/browser.png)

## <a name="proxy-server-and-load-balancer-scenarios"></a>プロキシ サーバーとロード バランサーのシナリオ

インターネットや企業ネットワークからの要求とやりとりする HTTP.sys でホストされるアプリの場合、プロキシ サーバーやロード バランサーの背後でホストするとき、追加の構成が必要になることがあります。 詳細については、「[プロキシ サーバーとロード バランサーを使用するために ASP.NET Core を構成する](xref:host-and-deploy/proxy-load-balancer)」を参照してください。

## <a name="advanced-http2-features-to-support-grpc"></a>gRPC をサポートする高度な HTTP/2 機能

HTTP.sys 内の追加の HTTP/2 機能によって、gRPC がサポートされています。これには、応答トレーラーやリセット フレームの送信に関するサポートが含まれます。

HTTP.sys で gRPC を実行するための要件:

* Windows 10、OS Build 19041.508 以降
* TLS 1.2 以降の接続

### <a name="trailers"></a>予告編

[!INCLUDE[](~/includes/trailers.md)]

### <a name="reset"></a>Reset

[!INCLUDE[](~/includes/reset.md)]

## <a name="additional-resources"></a>その他の技術情報

* [HTTP.sys を使用して Windows 認証を有効にする](xref:security/authentication/windowsauth#httpsys)
* [HTTP サーバー API](/windows/win32/http/http-api-start-page)
* [aspnet/HttpSysServer GitHub リポジトリ (ソース コード)](https://github.com/aspnet/HttpSysServer/)
* [ホスト](xref:fundamentals/index#host)
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="= aspnetcore-3.0"

[HTTP.sys](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture#hypertext-transfer-protocol-stack-httpsys) は、Windows 上でのみ動作する [ASP.NET Core 用 Web サーバー](xref:fundamentals/servers/index)です。 HTTP.sys は [Kestrel](xref:fundamentals/servers/kestrel) サーバーの代替製品であり、Kestrel では提供されていない機能がいくつか用意されています。

> [!IMPORTANT]
> HTTP.sys は [ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)と互換性がなく、IIS や IIS Express で使用することはできません。

HTTP.sys は、次の機能をサポートします。

* [Windows 認証](xref:security/authentication/windowsauth)
* ポート共有
* SNI を使用する HTTPS
* HTTP/2 over TLS (Windows 10 以降)
* 直接ファイル伝送
* 応答キャッシュ
* WebSocket (Windows 8 以降)

サポートされている Windows バージョン:

* Windows 7 以降
* Windows Server 2008 R2 以降

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/httpsys/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="when-to-use-httpsys"></a>HTTP.sys を使用するタイミング

HTTP.sys は、次のような展開に適しています。

* IIS を使用せず、インターネットに直接サーバーを公開する必要がある。

  ![インターネットと直接通信する HTTP.sys](httpsys/_static/httpsys-to-internet.png)

* 内部の展開で、[Windows 認証](xref:security/authentication/windowsauth)などの、Kestrel では使用できない機能が要求されている。

  ![内部ネットワークと直接通信する HTTP.sys](httpsys/_static/httpsys-to-internal.png)

HTTP.sys は、さまざまな種類の攻撃を防ぎ、フル機能の Web サーバーとして堅牢性、セキュリティ、スケーラビリティを提供する、成熟したテクノロジです。 IIS 自体が、HTTP.sys 上で HTTP リスナーとして実行されています。

## <a name="http2-support"></a>HTTP/2 のサポート

[Http/2](https://httpwg.org/specs/rfc7540.html) は、次の基本要件が満たされている場合に、ASP.NET Core アプリに対して有効になります。

* Windows Server 2016/Windows 10 以降
* [アプリケーション レイヤー プロトコル ネゴシエーション (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) 接続
* TLS 1.2 以降の接続

Http/2 接続が確立されると、[HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) が `HTTP/2` を報告します。

HTTP/2 は既定で有効になっています。 Http/2 接続が確立されない場合、接続は http/1.1 にフォールバックします。 Windows の今後のリリースで、HTTP.sys で HTTP/2 を無効にする機能を含む HTTP/2 構成フラグが使用可能になる予定です。

## <a name="kernel-mode-authentication-with-kerberos"></a>Kerberos を使用したカーネル モード認証

HTTP.sys では、Kerberos 認証プロトコルを使用したカーネル モード認証に処理が委任されます。 Kerberos および HTTP.sys ではユーザー モード認証がサポートされていません。 Active Directory から取得され、クライアントによって、ユーザーを認証するサーバーに転送される Kerberos トークン/チケットを暗号化解除するには、コンピューター アカウントを使用する必要があります。 アプリのユーザーではなく、ホストのサービス プリンシパル名 (SPN) を登録します。

## <a name="how-to-use-httpsys"></a>HTTP.sys の使用方法

### <a name="configure-the-aspnet-core-app-to-use-httpsys"></a>HTTP.sys を使用するように ASP.NET Core アプリを構成する

ホストを構築するときに <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderHttpSysExtensions.UseHttpSys*> 拡張メソッドを呼び出し、必要な <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions> を指定します。 次の例では、既定値にオプションを設定しています。

[!code-csharp[](httpsys/samples/3.x/SampleApp/Program.cs?name=snippet1&highlight=5-13)]

HTTP.sys の追加の構成は、[レジストリ設定](https://support.microsoft.com/help/820129/http-sys-registry-settings-for-windows)を通じて処理されます。

**HTTP.sys オプション**

| プロパティ | 説明 | Default |
| -------- | ----------- | :-----: |
| [AllowSynchronousIO](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.AllowSynchronousIO) | `HttpContext.Request.Body` および `HttpContext.Response.Body` に対して、入力/出力の同期を許可するかどうかを制御します。 | `false` |
| [Authentication.AllowAnonymous](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.AllowAnonymous) | 匿名要求を許可します。 | `true` |
| [Authentication.Schemes](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.Schemes) | 許可される認証方式を指定します。 リスナーを破棄する前ならいつでも変更できます。 値は [AuthenticationSchemes 列挙型](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationSchemes) (`Basic`、`Kerberos`、`Negotiate`、`None`、および `NTLM`) によって指定します。 | `None` |
| [EnableResponseCaching](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.EnableResponseCaching) | 対象となるヘッダーを持つ応答に対して、[カーネル モード](/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode)のキャッシュを試行します。 `Set-Cookie`、`Vary`、または `Pragma` ヘッダーを含む応答は対象外です。 応答は、`public` である `Cache-Control` ヘッダーと `shared-max-age` または `max-age` の値のいずれかを含むか、または `Expires` ヘッダーを含む必要があります。 | `true` |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxAccepts> | 同時受け入れの最大数です。 | 5 &times;[Environment.<br>ProcessorCount](xref:System.Environment.ProcessorCount) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxConnections> | 受け入れるコンカレント接続の最大数です。 無限にするには、`-1` を使用します。 コンピューター全体のレジストリ設定を使用するには、`null` を使用します。 | `null`<br>(コンピューター全体の<br>設定) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> | 「<a href="#maxrequestbodysize">MaxRequestBodySize</a>」セクションを参照してください。 | 30000000 バイト<br>(~28.6 MB) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.RequestQueueLimit> | キューに置くことができる要求の最大数。 | 1000 |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.ThrowWriteExceptions> | 応答本文の書き込みがクライアントの接続の切断によって失敗した場合、例外をスローするか、または正常に完了するかどうかを指定します。 | `false`<br>(正常に完了する) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.Timeouts> | HTTP.sys <xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager> 構成を公開します。これはレジストリでも構成できます。 各設定に関する既定値などの詳細については、API のリンクを参照してください。<ul><li>[TimeoutManager.DrainEntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.DrainEntityBody):HTTP サーバー API が Keep-Alive 接続でエンティティ本体をドレインするまでに許容される時間です。</li><li>[TimeoutManager.EntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.EntityBody):要求のエンティティ本体が到着するまでに許容される時間。</li><li>[TimeoutManager.HeaderWait](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.HeaderWait):HTTP サーバー API が要求ヘッダーを解析するまでに許容される時間。</li><li>[TimeoutManager.IdleConnection](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.IdleConnection):アイドル接続で許容される時間。</li><li>[TimeoutManager.MinSendBytesPerSecond](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.MinSendBytesPerSecond):応答の最小送信レート。</li><li>[TimeoutManager.RequestQueue](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.RequestQueue):要求が、アプリにピック アップされるまでに要求キューの中に留まっていられる時間。</li></ul> |  |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> | HTTP.sys に登録する <xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection> を指定します。 最も便利なのは [UrlPrefixCollection.Add](xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection.Add*) です。これを使用して、コレクションにプレフィックスを追加できます。 これらは、リスナーを破棄する前ならいつでも変更できます。 |  |

<a name="maxrequestbodysize"></a>

**MaxRequestBodySize**

要求本文の最大許容サイズ (バイト単位) です。 `null` に設定する場合、要求本文の最大サイズは制限されません。 この制限は、アップグレード済みの接続 (常に無制限) には影響しません。

1 つの `IActionResult` に対する ASP.NET Core MVC アプリの制限をオーバーライドする方法として、アクション メソッドに対して <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> 属性を使用することをお勧めします。

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

アプリが要求の読み取りを開始した後に、アプリが要求に対する制限を構成しようとすると、例外がスローされます。 `IsReadOnly` プロパティを使用して、`MaxRequestBodySize` プロパティが読み取り専用状態にあるかどうか、つまり制限を構成するには遅すぎるかどうかを示すことができます。

要求ごとにアプリで <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> をオーバーライドする必要がある場合は、<xref:Microsoft.AspNetCore.Http.Features.IHttpMaxRequestBodySizeFeature> を使います。

[!code-csharp[](httpsys/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=6-7)]

Visual Studio を使用する場合は、アプリが IIS または IIS Express を実行するように構成されていないことを確認します。

Visual Studio では、既定の起動プロファイルは IIS Express 用です。 プロジェクトをコンソール アプリとして実行するには、次のスクリーン ショットに示すように、選択したプロファイルを手動で変更します。

![コンソール アプリのプロファイルを選択する](httpsys/_static/vs-choose-profile.png)

### <a name="configure-windows-server"></a>Windows Server を構成する

1. アプリに対して開くポートを決めたら、[Windows ファイアウォール](/windows/security/threat-protection/windows-firewall/create-an-inbound-port-rule)か [New-NetFirewallRule](/powershell/module/netsecurity/new-netfirewallrule) PowerShell コマンドレットを使用して、トラフィックが HTTP.sys に到達できるようにファイアウォールのポートを開きます。 次のコマンドとアプリの構成では、ポート 443 を使用します。

1. Azure VM に展開する場合は、[ネットワーク セキュリティ グループ](/azure/virtual-machines/windows/nsg-quickstart-portal)内でポートを開きます。 次のコマンドとアプリの構成では、ポート 443 を使用します。

1. 必要に応じて、X.509 証明書を取得してインストールします。

   Windows の場合は、[New-SelfSignedCertificate PowerShell コマンドレット](/powershell/module/pkiclient/new-selfsignedcertificate)を使用して自己署名証明書を作成します。 サポート対象外の例については、[UpdateIISExpressSSLForChrome.ps1](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/includes/make-x509-cert/UpdateIISExpressSSLForChrome.ps1) を参照してください。

   自己署名証明書か CA 署名証明書のいずれかをサーバーの **Local Machine**>**Personal** ストアにインストールします。

1. アプリが[フレームワークに依存する展開](/dotnet/core/deploying/#framework-dependent-deployments-fdd)である場合は、.NET Core、.NET Framework、またはその両方 (アプリが .NET Framework をターゲットとする .NET Core アプリである場合) をインストールします。

   * **.NET Core**: アプリで .NET Core が必要な場合は、[.NET Core のダウンロード](https://dotnet.microsoft.com/download)から **.NET Core Runtime** インストーラーを取得して実行します。 サーバーに SDK 全体をインストールしないでください。
   * **.NET Framework**:アプリで .NET Framework が必要な場合は、[.NET Framework のインストール ガイド](/dotnet/framework/install/)を参照してください。 必要な .NET Framework をインストールします。 最新の .NET Framework のインストーラーは [.NET Core のダウンロード](https://dotnet.microsoft.com/download) ページから入手できます。

   アプリが[自己完結型の展開](/dotnet/core/deploying/#self-contained-deployments-scd)の場合、アプリの展開内にランタイムが含まれています。 サーバーにフレームワークをインストールする必要はありません。

1. アプリに URL とポートを構成します。

   既定では、ASP.NET Core は `http://localhost:5000` にバインドされます。 URL プレフィックスとポートを構成するには、次のオプションがあります。

   * <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
   * `urls` コマンド ライン引数
   * `ASPNETCORE_URLS` 環境変数
   * <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes>

   次のコード例は、サーバーのローカル IP アドレス `10.0.0.4` を使ってポート 443 上で <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> を使う方法を示しています。

   [!code-csharp[](httpsys/samples_snapshot/3.x/Program.cs?highlight=7)]

   `UrlPrefixes` の利点は、プレフィックスの形式が正しくなかった場合、すぐにエラー メッセージが生成されることです。

   `UrlPrefixes` の設定は `UseUrls`/`urls`/`ASPNETCORE_URLS` の設定をオーバーライドします。 したがって、`UseUrls`、`urls`、および `ASPNETCORE_URLS` 環境変数の利点は、Kestrel と HTTP.sys を簡単に切り替えられることです。

   HTTP.sys では、[HTTP サーバー API の UrlPrefix 文字列形式](/windows/win32/http/urlprefix-strings)が使用されます。

   > [!WARNING]
   > 最上位のワイルドカードのバインド ( `http://*:80/` と `http://+:80` ) は使用しては **いけません** 。 最上位のワイルドカードのバインドを使用すると、アプリにセキュリティの脆弱性が生じます。 これは、強力と脆弱の両方のワイルドカードに適用されます。 ワイルドカードではなく、明示的なホスト名か IP アドレスを使用してください。 親ドメイン全体を制御する場合、サブドメインのワイルドカードのバインド (たとえば、`*.mysub.com`) がセキュリティ リスクになることはありません (脆弱である `*.com` とは対照的)。 詳細については、[RFC 7230:セクション 5.4:ホスト](https://tools.ietf.org/html/rfc7230#section-5.4)に関するページを参照してください。

1. サーバーで URL プレフィックスを事前登録します。

   HTTP.sys を構成するための組み込みツールは、*netsh.exe* です。 *netsh.exe* を使用して、URL プレフィックスを予約し、X.509 証明書を割り当てることができます。 ツールを使用するには管理者特権が必要です。

   *netsh.exe* ツールを使用して、アプリ用に URL を登録します。

   ```console
   netsh http add urlacl url=<URL> user=<USER>
   ```

   * `<URL>`:完全修飾 URL (Uniform Resource Locator)。 ワイルドカードのバインドは使用しないでください。 有効なホスト名かローカル IP アドレスを使用してください。 "*URL の末尾にはスラッシュが必要です。* "
   * `<USER>`:ユーザーまたはユーザー グループの名前を指定します。

   次の例では、サーバーのローカル IP アドレスは `10.0.0.4` です。

   ```console
   netsh http add urlacl url=https://10.0.0.4:443/ user=Users
   ```

   URL が登録されると、ツールから `URL reservation successfully added` という応答があります。

   登録済みの URL を削除するには、`delete urlacl` コマンドを使用します。

   ```console
   netsh http delete urlacl url=<URL>
   ```

1. サーバーで X.509 証明書を登録します。

   *netsh.exe* ツールを使用して、アプリ用の証明書を登録します。

   ```console
   netsh http add sslcert ipport=<IP>:<PORT> certhash=<THUMBPRINT> appid="{<GUID>}"
   ```

   * `<IP>`:バインド用のローカル IP アドレスを指定します。 ワイルドカードのバインドは使用しないでください。 有効な IP アドレスを使用してください。
   * `<PORT>`:バインド用のポートを指定します。
   * `<THUMBPRINT>`:X.509 証明書の拇印です。
   * `<GUID>`:情報提供を目的として開発者によって生成された、アプリを表す GUID です。

   参照用に、この GUID をパッケージ タグとしてアプリに格納します。

   * Visual Studio:
     * **ソリューション エクスプローラー** 内でアプリを右クリックし、 **[プロパティ]** をクリックして、アプリのプロジェクト プロパティを開きます。
     * **[パッケージ]** タブを選択します。
     * 作成した GUID を **[タグ]** フィールドに入力します。
   * Visual Studio を使用しない場合:
     * アプリのプロジェクト ファイルを開きます。
     * 作成した GUID を指定した `<PackageTags>` プロパティを、新規または既存の `<PropertyGroup>` に追加します。

       ```xml
       <PropertyGroup>
         <PackageTags>9412ee86-c21b-4eb8-bd89-f650fbf44931</PackageTags>
       </PropertyGroup>
       ```

   次に例を示します。

   * サーバーのローカル IP アドレスは `10.0.0.4` です。
   * オンラインのランダム GUID ジェネレーターによって、`appid` の値が提供されます。

   ```console
   netsh http add sslcert 
       ipport=10.0.0.4:443 
       certhash=b66ee04419d4ee37464ab8785ff02449980eae10 
       appid="{9412ee86-c21b-4eb8-bd89-f650fbf44931}"
   ```

   証明書が登録されると、ツールから `SSL Certificate successfully added` という応答があります。

   証明書の登録を削除するには、`delete sslcert` コマンドを使用します。

   ```console
   netsh http delete sslcert ipport=<IP>:<PORT>
   ```

   以下は、*netsh.exe* のリファレンス ドキュメントです。

   * [Netsh Commands for Hypertext Transfer Protocol (HTTP)](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc725882(v=ws.10)) (ハイパーテキスト転送プロトコル (HTTP) 用の Netsh コマンド)
   * [UrlPrefix Strings](/windows/win32/http/urlprefix-strings) (UrlPrefix 文字列)

1. アプリを実行します。

   1024 より大きいポート番号で (HTTPS ではなく) HTTP を使用して localhost にバインドする場合、アプリの実行に管理者権限は必要ありません。 その他の構成の場合 (たとえば、ローカル IP アドレスを使用する場合やポート 443 にバインドする場合)、管理者権限でアプリを実行します。

   サーバーのパブリック IP アドレスでアプリが応答します。 この例では、サーバーは自身のパブリック IP アドレス `104.214.79.47` でインターネットからアクセスされます。

   この例では開発証明書が使用されています。 証明書が信頼できないというブラウザーの警告がバイパスされた後に、ページが安全に読み込まれます。

   ![読み込まれたアプリのインデックス ページを表示するブラウザー ウィンドウ](httpsys/_static/browser.png)

## <a name="proxy-server-and-load-balancer-scenarios"></a>プロキシ サーバーとロード バランサーのシナリオ

インターネットや企業ネットワークからの要求とやりとりする HTTP.sys でホストされるアプリの場合、プロキシ サーバーやロード バランサーの背後でホストするとき、追加の構成が必要になることがあります。 詳細については、「[プロキシ サーバーとロード バランサーを使用するために ASP.NET Core を構成する](xref:host-and-deploy/proxy-load-balancer)」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* [HTTP.sys を使用して Windows 認証を有効にする](xref:security/authentication/windowsauth#httpsys)
* [HTTP サーバー API](/windows/win32/http/http-api-start-page)
* [aspnet/HttpSysServer GitHub リポジトリ (ソース コード)](https://github.com/aspnet/HttpSysServer/)
* [ホスト](xref:fundamentals/index#host)
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[HTTP.sys](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture#hypertext-transfer-protocol-stack-httpsys) は、Windows 上でのみ動作する [ASP.NET Core 用 Web サーバー](xref:fundamentals/servers/index)です。 HTTP.sys は [Kestrel](xref:fundamentals/servers/kestrel) サーバーの代替製品であり、Kestrel では提供されていない機能がいくつか用意されています。

> [!IMPORTANT]
> HTTP.sys は [ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)と互換性がなく、IIS や IIS Express で使用することはできません。

HTTP.sys は、次の機能をサポートします。

* [Windows 認証](xref:security/authentication/windowsauth)
* ポート共有
* SNI を使用する HTTPS
* HTTP/2 over TLS (Windows 10 以降)
* 直接ファイル伝送
* 応答キャッシュ
* WebSocket (Windows 8 以降)

サポートされている Windows バージョン:

* Windows 7 以降
* Windows Server 2008 R2 以降

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/httpsys/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="when-to-use-httpsys"></a>HTTP.sys を使用するタイミング

HTTP.sys は、次のような展開に適しています。

* IIS を使用せず、インターネットに直接サーバーを公開する必要がある。

  ![インターネットと直接通信する HTTP.sys](httpsys/_static/httpsys-to-internet.png)

* 内部の展開で、[Windows 認証](xref:security/authentication/windowsauth)などの、Kestrel では使用できない機能が要求されている。

  ![内部ネットワークと直接通信する HTTP.sys](httpsys/_static/httpsys-to-internal.png)

HTTP.sys は、さまざまな種類の攻撃を防ぎ、フル機能の Web サーバーとして堅牢性、セキュリティ、スケーラビリティを提供する、成熟したテクノロジです。 IIS 自体が、HTTP.sys 上で HTTP リスナーとして実行されています。

## <a name="http2-support"></a>HTTP/2 のサポート

[Http/2](https://httpwg.org/specs/rfc7540.html) は、次の基本要件が満たされている場合に、ASP.NET Core アプリに対して有効になります。

* Windows Server 2016/Windows 10 以降
* [アプリケーション レイヤー プロトコル ネゴシエーション (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) 接続
* TLS 1.2 以降の接続

Http/2 接続が確立されると、[HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) が `HTTP/2` を報告します。

HTTP/2 は既定で有効になっています。 Http/2 接続が確立されない場合、接続は http/1.1 にフォールバックします。 Windows の今後のリリースで、HTTP.sys で HTTP/2 を無効にする機能を含む HTTP/2 構成フラグが使用可能になる予定です。

## <a name="kernel-mode-authentication-with-kerberos"></a>Kerberos を使用したカーネル モード認証

HTTP.sys では、Kerberos 認証プロトコルを使用したカーネル モード認証に処理が委任されます。 Kerberos および HTTP.sys ではユーザー モード認証がサポートされていません。 Active Directory から取得され、クライアントによって、ユーザーを認証するサーバーに転送される Kerberos トークン/チケットを暗号化解除するには、コンピューター アカウントを使用する必要があります。 アプリのユーザーではなく、ホストのサービス プリンシパル名 (SPN) を登録します。

## <a name="how-to-use-httpsys"></a>HTTP.sys の使用方法

### <a name="configure-the-aspnet-core-app-to-use-httpsys"></a>HTTP.sys を使用するように ASP.NET Core アプリを構成する

[Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app) ([nuget.org](https://www.nuget.org/packages/Microsoft.AspNetCore.App/)) を使用する場合は、プロジェクト ファイルのパッケージ参照は必要ありません。 `Microsoft.AspNetCore.App` メタパッケージを使用しない場合は、[Microsoft.AspNetCore.Server.HttpSys](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.HttpSys/) にパッケージ参照を追加します。

ホストを構築するときに <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderHttpSysExtensions.UseHttpSys*> 拡張メソッドを呼び出し、必要な <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions> を指定します。 次の例では、既定値にオプションを設定しています。

[!code-csharp[](httpsys/samples/2.x/SampleApp/Program.cs?name=snippet1&highlight=4-12)]

HTTP.sys の追加の構成は、[レジストリ設定](https://support.microsoft.com/help/820129/http-sys-registry-settings-for-windows)を通じて処理されます。

**HTTP.sys オプション**

| プロパティ | 説明 | Default |
| -------- | ----------- | :-----: |
| [AllowSynchronousIO](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.AllowSynchronousIO) | `HttpContext.Request.Body` および `HttpContext.Response.Body` に対して、入力/出力の同期を許可するかどうかを制御します。 | `true` |
| [Authentication.AllowAnonymous](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.AllowAnonymous) | 匿名要求を許可します。 | `true` |
| [Authentication.Schemes](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.Schemes) | 許可される認証方式を指定します。 リスナーを破棄する前ならいつでも変更できます。 値は [AuthenticationSchemes 列挙型](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationSchemes) (`Basic`、`Kerberos`、`Negotiate`、`None`、および `NTLM`) によって指定します。 | `None` |
| [EnableResponseCaching](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.EnableResponseCaching) | 対象となるヘッダーを持つ応答に対して、[カーネル モード](/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode)のキャッシュを試行します。 `Set-Cookie`、`Vary`、または `Pragma` ヘッダーを含む応答は対象外です。 応答は、`public` である `Cache-Control` ヘッダーと `shared-max-age` または `max-age` の値のいずれかを含むか、または `Expires` ヘッダーを含む必要があります。 | `true` |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxAccepts> | 同時受け入れの最大数です。 | 5 &times;[Environment.<br>ProcessorCount](xref:System.Environment.ProcessorCount) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxConnections> | 受け入れるコンカレント接続の最大数です。 無限にするには、`-1` を使用します。 コンピューター全体のレジストリ設定を使用するには、`null` を使用します。 | `null`<br>(コンピューター全体の<br>設定) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> | 「<a href="#maxrequestbodysize">MaxRequestBodySize</a>」セクションを参照してください。 | 30000000 バイト<br>(~28.6 MB) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.RequestQueueLimit> | キューに置くことができる要求の最大数。 | 1000 |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.ThrowWriteExceptions> | 応答本文の書き込みがクライアントの接続の切断によって失敗した場合、例外をスローするか、または正常に完了するかどうかを指定します。 | `false`<br>(正常に完了する) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.Timeouts> | HTTP.sys <xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager> 構成を公開します。これはレジストリでも構成できます。 各設定に関する既定値などの詳細については、API のリンクを参照してください。<ul><li>[TimeoutManager.DrainEntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.DrainEntityBody):HTTP サーバー API が Keep-Alive 接続でエンティティ本体をドレインするまでに許容される時間です。</li><li>[TimeoutManager.EntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.EntityBody):要求のエンティティ本体が到着するまでに許容される時間。</li><li>[TimeoutManager.HeaderWait](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.HeaderWait):HTTP サーバー API が要求ヘッダーを解析するまでに許容される時間。</li><li>[TimeoutManager.IdleConnection](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.IdleConnection):アイドル接続で許容される時間。</li><li>[TimeoutManager.MinSendBytesPerSecond](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.MinSendBytesPerSecond):応答の最小送信レート。</li><li>[TimeoutManager.RequestQueue](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.RequestQueue):要求が、アプリにピック アップされるまでに要求キューの中に留まっていられる時間。</li></ul> |  |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> | HTTP.sys に登録する <xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection> を指定します。 最も便利なのは [UrlPrefixCollection.Add](xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection.Add*) です。これを使用して、コレクションにプレフィックスを追加できます。 これらは、リスナーを破棄する前ならいつでも変更できます。 |  |

<a name="maxrequestbodysize"></a>

**MaxRequestBodySize**

要求本文の最大許容サイズ (バイト単位) です。 `null` に設定する場合、要求本文の最大サイズは制限されません。 この制限は、アップグレード済みの接続 (常に無制限) には影響しません。

1 つの `IActionResult` に対する ASP.NET Core MVC アプリの制限をオーバーライドする方法として、アクション メソッドに対して <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> 属性を使用することをお勧めします。

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

アプリが要求の読み取りを開始した後に、アプリが要求に対する制限を構成しようとすると、例外がスローされます。 `IsReadOnly` プロパティを使用して、`MaxRequestBodySize` プロパティが読み取り専用状態にあるかどうか、つまり制限を構成するには遅すぎるかどうかを示すことができます。

要求ごとにアプリで <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> をオーバーライドする必要がある場合は、<xref:Microsoft.AspNetCore.Http.Features.IHttpMaxRequestBodySizeFeature> を使います。

[!code-csharp[](httpsys/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=6-7)]

Visual Studio を使用する場合は、アプリが IIS または IIS Express を実行するように構成されていないことを確認します。

Visual Studio では、既定の起動プロファイルは IIS Express 用です。 プロジェクトをコンソール アプリとして実行するには、次のスクリーン ショットに示すように、選択したプロファイルを手動で変更します。

![コンソール アプリのプロファイルを選択する](httpsys/_static/vs-choose-profile.png)

### <a name="configure-windows-server"></a>Windows Server を構成する

1. アプリに対して開くポートを決めたら、[Windows ファイアウォール](/windows/security/threat-protection/windows-firewall/create-an-inbound-port-rule)か [New-NetFirewallRule](/powershell/module/netsecurity/new-netfirewallrule) PowerShell コマンドレットを使用して、トラフィックが HTTP.sys に到達できるようにファイアウォールのポートを開きます。 次のコマンドとアプリの構成では、ポート 443 を使用します。

1. Azure VM に展開する場合は、[ネットワーク セキュリティ グループ](/azure/virtual-machines/windows/nsg-quickstart-portal)内でポートを開きます。 次のコマンドとアプリの構成では、ポート 443 を使用します。

1. 必要に応じて、X.509 証明書を取得してインストールします。

   Windows の場合は、[New-SelfSignedCertificate PowerShell コマンドレット](/powershell/module/pkiclient/new-selfsignedcertificate)を使用して自己署名証明書を作成します。 サポート対象外の例については、[UpdateIISExpressSSLForChrome.ps1](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/includes/make-x509-cert/UpdateIISExpressSSLForChrome.ps1) を参照してください。

   自己署名証明書か CA 署名証明書のいずれかをサーバーの **Local Machine**>**Personal** ストアにインストールします。

1. アプリが[フレームワークに依存する展開](/dotnet/core/deploying/#framework-dependent-deployments-fdd)である場合は、.NET Core、.NET Framework、またはその両方 (アプリが .NET Framework をターゲットとする .NET Core アプリである場合) をインストールします。

   * **.NET Core**: アプリで .NET Core が必要な場合は、[.NET Core のダウンロード](https://dotnet.microsoft.com/download)から **.NET Core Runtime** インストーラーを取得して実行します。 サーバーに SDK 全体をインストールしないでください。
   * **.NET Framework**:アプリで .NET Framework が必要な場合は、[.NET Framework のインストール ガイド](/dotnet/framework/install/)を参照してください。 必要な .NET Framework をインストールします。 最新の .NET Framework のインストーラーは [.NET Core のダウンロード](https://dotnet.microsoft.com/download) ページから入手できます。

   アプリが[自己完結型の展開](/dotnet/core/deploying/#self-contained-deployments-scd)の場合、アプリの展開内にランタイムが含まれています。 サーバーにフレームワークをインストールする必要はありません。

1. アプリに URL とポートを構成します。

   既定では、ASP.NET Core は `http://localhost:5000` にバインドされます。 URL プレフィックスとポートを構成するには、次のオプションがあります。

   * <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
   * `urls` コマンド ライン引数
   * `ASPNETCORE_URLS` 環境変数
   * <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes>

   次のコード例は、サーバーのローカル IP アドレス `10.0.0.4` を使ってポート 443 上で <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> を使う方法を示しています。

   [!code-csharp[](httpsys/samples_snapshot/2.x/Program.cs?highlight=6)]

   `UrlPrefixes` の利点は、プレフィックスの形式が正しくなかった場合、すぐにエラー メッセージが生成されることです。

   `UrlPrefixes` の設定は `UseUrls`/`urls`/`ASPNETCORE_URLS` の設定をオーバーライドします。 したがって、`UseUrls`、`urls`、および `ASPNETCORE_URLS` 環境変数の利点は、Kestrel と HTTP.sys を簡単に切り替えられることです。

   HTTP.sys では、[HTTP サーバー API の UrlPrefix 文字列形式](/windows/win32/http/urlprefix-strings)が使用されます。

   > [!WARNING]
   > 最上位のワイルドカードのバインド ( `http://*:80/` と `http://+:80` ) は使用しては **いけません** 。 最上位のワイルドカードのバインドを使用すると、アプリにセキュリティの脆弱性が生じます。 これは、強力と脆弱の両方のワイルドカードに適用されます。 ワイルドカードではなく、明示的なホスト名か IP アドレスを使用してください。 親ドメイン全体を制御する場合、サブドメインのワイルドカードのバインド (たとえば、`*.mysub.com`) がセキュリティ リスクになることはありません (脆弱である `*.com` とは対照的)。 詳細については、[RFC 7230:セクション 5.4:ホスト](https://tools.ietf.org/html/rfc7230#section-5.4)に関するページを参照してください。

1. サーバーで URL プレフィックスを事前登録します。

   HTTP.sys を構成するための組み込みツールは、*netsh.exe* です。 *netsh.exe* を使用して、URL プレフィックスを予約し、X.509 証明書を割り当てることができます。 ツールを使用するには管理者特権が必要です。

   *netsh.exe* ツールを使用して、アプリ用に URL を登録します。

   ```console
   netsh http add urlacl url=<URL> user=<USER>
   ```

   * `<URL>`:完全修飾 URL (Uniform Resource Locator)。 ワイルドカードのバインドは使用しないでください。 有効なホスト名かローカル IP アドレスを使用してください。 "*URL の末尾にはスラッシュが必要です。* "
   * `<USER>`:ユーザーまたはユーザー グループの名前を指定します。

   次の例では、サーバーのローカル IP アドレスは `10.0.0.4` です。

   ```console
   netsh http add urlacl url=https://10.0.0.4:443/ user=Users
   ```

   URL が登録されると、ツールから `URL reservation successfully added` という応答があります。

   登録済みの URL を削除するには、`delete urlacl` コマンドを使用します。

   ```console
   netsh http delete urlacl url=<URL>
   ```

1. サーバーで X.509 証明書を登録します。

   *netsh.exe* ツールを使用して、アプリ用の証明書を登録します。

   ```console
   netsh http add sslcert ipport=<IP>:<PORT> certhash=<THUMBPRINT> appid="{<GUID>}"
   ```

   * `<IP>`:バインド用のローカル IP アドレスを指定します。 ワイルドカードのバインドは使用しないでください。 有効な IP アドレスを使用してください。
   * `<PORT>`:バインド用のポートを指定します。
   * `<THUMBPRINT>`:X.509 証明書の拇印です。
   * `<GUID>`:情報提供を目的として開発者によって生成された、アプリを表す GUID です。

   参照用に、この GUID をパッケージ タグとしてアプリに格納します。

   * Visual Studio:
     * **ソリューション エクスプローラー** 内でアプリを右クリックし、 **[プロパティ]** をクリックして、アプリのプロジェクト プロパティを開きます。
     * **[パッケージ]** タブを選択します。
     * 作成した GUID を **[タグ]** フィールドに入力します。
   * Visual Studio を使用しない場合:
     * アプリのプロジェクト ファイルを開きます。
     * 作成した GUID を指定した `<PackageTags>` プロパティを、新規または既存の `<PropertyGroup>` に追加します。

       ```xml
       <PropertyGroup>
         <PackageTags>9412ee86-c21b-4eb8-bd89-f650fbf44931</PackageTags>
       </PropertyGroup>
       ```

   次に例を示します。

   * サーバーのローカル IP アドレスは `10.0.0.4` です。
   * オンラインのランダム GUID ジェネレーターによって、`appid` の値が提供されます。

   ```console
   netsh http add sslcert 
       ipport=10.0.0.4:443 
       certhash=b66ee04419d4ee37464ab8785ff02449980eae10 
       appid="{9412ee86-c21b-4eb8-bd89-f650fbf44931}"
   ```

   証明書が登録されると、ツールから `SSL Certificate successfully added` という応答があります。

   証明書の登録を削除するには、`delete sslcert` コマンドを使用します。

   ```console
   netsh http delete sslcert ipport=<IP>:<PORT>
   ```

   以下は、*netsh.exe* のリファレンス ドキュメントです。

   * [Netsh Commands for Hypertext Transfer Protocol (HTTP)](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc725882(v=ws.10)) (ハイパーテキスト転送プロトコル (HTTP) 用の Netsh コマンド)
   * [UrlPrefix Strings](/windows/win32/http/urlprefix-strings) (UrlPrefix 文字列)

1. アプリを実行します。

   1024 より大きいポート番号で (HTTPS ではなく) HTTP を使用して localhost にバインドする場合、アプリの実行に管理者権限は必要ありません。 その他の構成の場合 (たとえば、ローカル IP アドレスを使用する場合やポート 443 にバインドする場合)、管理者権限でアプリを実行します。

   サーバーのパブリック IP アドレスでアプリが応答します。 この例では、サーバーは自身のパブリック IP アドレス `104.214.79.47` でインターネットからアクセスされます。

   この例では開発証明書が使用されています。 証明書が信頼できないというブラウザーの警告がバイパスされた後に、ページが安全に読み込まれます。

   ![読み込まれたアプリのインデックス ページを表示するブラウザー ウィンドウ](httpsys/_static/browser.png)

## <a name="proxy-server-and-load-balancer-scenarios"></a>プロキシ サーバーとロード バランサーのシナリオ

インターネットや企業ネットワークからの要求とやりとりする HTTP.sys でホストされるアプリの場合、プロキシ サーバーやロード バランサーの背後でホストするとき、追加の構成が必要になることがあります。 詳細については、「[プロキシ サーバーとロード バランサーを使用するために ASP.NET Core を構成する](xref:host-and-deploy/proxy-load-balancer)」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* [HTTP.sys を使用して Windows 認証を有効にする](xref:security/authentication/windowsauth#httpsys)
* [HTTP サーバー API](/windows/win32/http/http-api-start-page)
* [aspnet/HttpSysServer GitHub リポジトリ (ソース コード)](https://github.com/aspnet/HttpSysServer/)
* [ホスト](xref:fundamentals/index#host)
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="< aspnetcore-2.2"

[HTTP.sys](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture#hypertext-transfer-protocol-stack-httpsys) は、Windows 上でのみ動作する [ASP.NET Core 用 Web サーバー](xref:fundamentals/servers/index)です。 HTTP.sys は [Kestrel](xref:fundamentals/servers/kestrel) サーバーの代替製品であり、Kestrel では提供されていない機能がいくつか用意されています。

> [!IMPORTANT]
> HTTP.sys は [ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)と互換性がなく、IIS や IIS Express で使用することはできません。

HTTP.sys は、次の機能をサポートします。

* [Windows 認証](xref:security/authentication/windowsauth)
* ポート共有
* SNI を使用する HTTPS
* HTTP/2 over TLS (Windows 10 以降)
* 直接ファイル伝送
* 応答キャッシュ
* WebSocket (Windows 8 以降)

サポートされている Windows バージョン:

* Windows 7 以降
* Windows Server 2008 R2 以降

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/httpsys/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="when-to-use-httpsys"></a>HTTP.sys を使用するタイミング

HTTP.sys は、次のような展開に適しています。

* IIS を使用せず、インターネットに直接サーバーを公開する必要がある。

  ![インターネットと直接通信する HTTP.sys](httpsys/_static/httpsys-to-internet.png)

* 内部の展開で、[Windows 認証](xref:security/authentication/windowsauth)などの、Kestrel では使用できない機能が要求されている。

  ![内部ネットワークと直接通信する HTTP.sys](httpsys/_static/httpsys-to-internal.png)

HTTP.sys は、さまざまな種類の攻撃を防ぎ、フル機能の Web サーバーとして堅牢性、セキュリティ、スケーラビリティを提供する、成熟したテクノロジです。 IIS 自体が、HTTP.sys 上で HTTP リスナーとして実行されています。

## <a name="http2-support"></a>HTTP/2 のサポート

[Http/2](https://httpwg.org/specs/rfc7540.html) は、次の基本要件が満たされている場合に、ASP.NET Core アプリに対して有効になります。

* Windows Server 2016/Windows 10 以降
* [アプリケーション レイヤー プロトコル ネゴシエーション (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) 接続
* TLS 1.2 以降の接続

Http/2 接続が確立されると、[HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) が `HTTP/1.1` を報告します。

HTTP/2 は既定で有効になっています。 Http/2 接続が確立されない場合、接続は http/1.1 にフォールバックします。 Windows の今後のリリースで、HTTP.sys で HTTP/2 を無効にする機能を含む HTTP/2 構成フラグが使用可能になる予定です。

## <a name="kernel-mode-authentication-with-kerberos"></a>Kerberos を使用したカーネル モード認証

HTTP.sys では、Kerberos 認証プロトコルを使用したカーネル モード認証に処理が委任されます。 Kerberos および HTTP.sys ではユーザー モード認証がサポートされていません。 Active Directory から取得され、クライアントによって、ユーザーを認証するサーバーに転送される Kerberos トークン/チケットを暗号化解除するには、コンピューター アカウントを使用する必要があります。 アプリのユーザーではなく、ホストのサービス プリンシパル名 (SPN) を登録します。

## <a name="how-to-use-httpsys"></a>HTTP.sys の使用方法

### <a name="configure-the-aspnet-core-app-to-use-httpsys"></a>HTTP.sys を使用するように ASP.NET Core アプリを構成する

[Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app) ([nuget.org](https://www.nuget.org/packages/Microsoft.AspNetCore.App/)) を使用する場合は、プロジェクト ファイルのパッケージ参照は必要ありません。 `Microsoft.AspNetCore.App` メタパッケージを使用しない場合は、[Microsoft.AspNetCore.Server.HttpSys](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.HttpSys/) にパッケージ参照を追加します。

ホストを構築するときに <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderHttpSysExtensions.UseHttpSys*> 拡張メソッドを呼び出し、必要な <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions> を指定します。 次の例では、既定値にオプションを設定しています。

[!code-csharp[](httpsys/samples/2.x/SampleApp/Program.cs?name=snippet1&highlight=4-12)]

HTTP.sys の追加の構成は、[レジストリ設定](https://support.microsoft.com/help/820129/http-sys-registry-settings-for-windows)を通じて処理されます。

**HTTP.sys オプション**

| プロパティ | 説明 | Default |
| -------- | ----------- | :-----: |
| [AllowSynchronousIO](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.AllowSynchronousIO) | `HttpContext.Request.Body` および `HttpContext.Response.Body` に対して、入力/出力の同期を許可するかどうかを制御します。 | `true` |
| [Authentication.AllowAnonymous](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.AllowAnonymous) | 匿名要求を許可します。 | `true` |
| [Authentication.Schemes](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.Schemes) | 許可される認証方式を指定します。 リスナーを破棄する前ならいつでも変更できます。 値は [AuthenticationSchemes 列挙型](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationSchemes) (`Basic`、`Kerberos`、`Negotiate`、`None`、および `NTLM`) によって指定します。 | `None` |
| [EnableResponseCaching](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.EnableResponseCaching) | 対象となるヘッダーを持つ応答に対して、[カーネル モード](/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode)のキャッシュを試行します。 `Set-Cookie`、`Vary`、または `Pragma` ヘッダーを含む応答は対象外です。 応答は、`public` である `Cache-Control` ヘッダーと `shared-max-age` または `max-age` の値のいずれかを含むか、または `Expires` ヘッダーを含む必要があります。 | `true` |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxAccepts> | 同時受け入れの最大数です。 | 5 &times;[Environment.<br>ProcessorCount](xref:System.Environment.ProcessorCount) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxConnections> | 受け入れるコンカレント接続の最大数です。 無限にするには、`-1` を使用します。 コンピューター全体のレジストリ設定を使用するには、`null` を使用します。 | `null`<br>(コンピューター全体の<br>設定) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> | 「<a href="#maxrequestbodysize">MaxRequestBodySize</a>」セクションを参照してください。 | 30000000 バイト<br>(~28.6 MB) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.RequestQueueLimit> | キューに置くことができる要求の最大数。 | 1000 |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.ThrowWriteExceptions> | 応答本文の書き込みがクライアントの接続の切断によって失敗した場合、例外をスローするか、または正常に完了するかどうかを指定します。 | `false`<br>(正常に完了する) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.Timeouts> | HTTP.sys <xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager> 構成を公開します。これはレジストリでも構成できます。 各設定に関する既定値などの詳細については、API のリンクを参照してください。<ul><li>[TimeoutManager.DrainEntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.DrainEntityBody):HTTP サーバー API が Keep-Alive 接続でエンティティ本体をドレインするまでに許容される時間です。</li><li>[TimeoutManager.EntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.EntityBody):要求のエンティティ本体が到着するまでに許容される時間。</li><li>[TimeoutManager.HeaderWait](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.HeaderWait):HTTP サーバー API が要求ヘッダーを解析するまでに許容される時間。</li><li>[TimeoutManager.IdleConnection](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.IdleConnection):アイドル接続で許容される時間。</li><li>[TimeoutManager.MinSendBytesPerSecond](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.MinSendBytesPerSecond):応答の最小送信レート。</li><li>[TimeoutManager.RequestQueue](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.RequestQueue):要求が、アプリにピック アップされるまでに要求キューの中に留まっていられる時間。</li></ul> |  |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> | HTTP.sys に登録する <xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection> を指定します。 最も便利なのは [UrlPrefixCollection.Add](xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection.Add*) です。これを使用して、コレクションにプレフィックスを追加できます。 これらは、リスナーを破棄する前ならいつでも変更できます。 |  |

<a name="maxrequestbodysize"></a>

**MaxRequestBodySize**

要求本文の最大許容サイズ (バイト単位) です。 `null` に設定する場合、要求本文の最大サイズは制限されません。 この制限は、アップグレード済みの接続 (常に無制限) には影響しません。

1 つの `IActionResult` に対する ASP.NET Core MVC アプリの制限をオーバーライドする方法として、アクション メソッドに対して <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> 属性を使用することをお勧めします。

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

アプリが要求の読み取りを開始した後に、アプリが要求に対する制限を構成しようとすると、例外がスローされます。 `IsReadOnly` プロパティを使用して、`MaxRequestBodySize` プロパティが読み取り専用状態にあるかどうか、つまり制限を構成するには遅すぎるかどうかを示すことができます。

要求ごとにアプリで <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> をオーバーライドする必要がある場合は、<xref:Microsoft.AspNetCore.Http.Features.IHttpMaxRequestBodySizeFeature> を使います。

[!code-csharp[](httpsys/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=6-7)]

Visual Studio を使用する場合は、アプリが IIS または IIS Express を実行するように構成されていないことを確認します。

Visual Studio では、既定の起動プロファイルは IIS Express 用です。 プロジェクトをコンソール アプリとして実行するには、次のスクリーン ショットに示すように、選択したプロファイルを手動で変更します。

![コンソール アプリのプロファイルを選択する](httpsys/_static/vs-choose-profile.png)

### <a name="configure-windows-server"></a>Windows Server を構成する

1. アプリに対して開くポートを決めたら、[Windows ファイアウォール](/windows/security/threat-protection/windows-firewall/create-an-inbound-port-rule)か [New-NetFirewallRule](/powershell/module/netsecurity/new-netfirewallrule) PowerShell コマンドレットを使用して、トラフィックが HTTP.sys に到達できるようにファイアウォールのポートを開きます。 次のコマンドとアプリの構成では、ポート 443 を使用します。

1. Azure VM に展開する場合は、[ネットワーク セキュリティ グループ](/azure/virtual-machines/windows/nsg-quickstart-portal)内でポートを開きます。 次のコマンドとアプリの構成では、ポート 443 を使用します。

1. 必要に応じて、X.509 証明書を取得してインストールします。

   Windows の場合は、[New-SelfSignedCertificate PowerShell コマンドレット](/powershell/module/pkiclient/new-selfsignedcertificate)を使用して自己署名証明書を作成します。 サポート対象外の例については、[UpdateIISExpressSSLForChrome.ps1](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/includes/make-x509-cert/UpdateIISExpressSSLForChrome.ps1) を参照してください。

   自己署名証明書か CA 署名証明書のいずれかをサーバーの **Local Machine**>**Personal** ストアにインストールします。

1. アプリが[フレームワークに依存する展開](/dotnet/core/deploying/#framework-dependent-deployments-fdd)である場合は、.NET Core、.NET Framework、またはその両方 (アプリが .NET Framework をターゲットとする .NET Core アプリである場合) をインストールします。

   * **.NET Core**: アプリで .NET Core が必要な場合は、[.NET Core のダウンロード](https://dotnet.microsoft.com/download)から **.NET Core Runtime** インストーラーを取得して実行します。 サーバーに SDK 全体をインストールしないでください。
   * **.NET Framework**:アプリで .NET Framework が必要な場合は、[.NET Framework のインストール ガイド](/dotnet/framework/install/)を参照してください。 必要な .NET Framework をインストールします。 最新の .NET Framework のインストーラーは [.NET Core のダウンロード](https://dotnet.microsoft.com/download) ページから入手できます。

   アプリが[自己完結型の展開](/dotnet/core/deploying/#self-contained-deployments-scd)の場合、アプリの展開内にランタイムが含まれています。 サーバーにフレームワークをインストールする必要はありません。

1. アプリに URL とポートを構成します。

   既定では、ASP.NET Core は `http://localhost:5000` にバインドされます。 URL プレフィックスとポートを構成するには、次のオプションがあります。

   * <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
   * `urls` コマンド ライン引数
   * `ASPNETCORE_URLS` 環境変数
   * <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes>

   次のコード例は、サーバーのローカル IP アドレス `10.0.0.4` を使ってポート 443 上で <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> を使う方法を示しています。

   [!code-csharp[](httpsys/samples_snapshot/2.x/Program.cs?highlight=6)]

   `UrlPrefixes` の利点は、プレフィックスの形式が正しくなかった場合、すぐにエラー メッセージが生成されることです。

   `UrlPrefixes` の設定は `UseUrls`/`urls`/`ASPNETCORE_URLS` の設定をオーバーライドします。 したがって、`UseUrls`、`urls`、および `ASPNETCORE_URLS` 環境変数の利点は、Kestrel と HTTP.sys を簡単に切り替えられることです。

   HTTP.sys では、[HTTP サーバー API の UrlPrefix 文字列形式](/windows/win32/http/urlprefix-strings)が使用されます。

   > [!WARNING]
   > 最上位のワイルドカードのバインド ( `http://*:80/` と `http://+:80` ) は使用しては **いけません** 。 最上位のワイルドカードのバインドを使用すると、アプリにセキュリティの脆弱性が生じます。 これは、強力と脆弱の両方のワイルドカードに適用されます。 ワイルドカードではなく、明示的なホスト名か IP アドレスを使用してください。 親ドメイン全体を制御する場合、サブドメインのワイルドカードのバインド (たとえば、`*.mysub.com`) がセキュリティ リスクになることはありません (脆弱である `*.com` とは対照的)。 詳細については、[RFC 7230:セクション 5.4:ホスト](https://tools.ietf.org/html/rfc7230#section-5.4)に関するページを参照してください。

1. サーバーで URL プレフィックスを事前登録します。

   HTTP.sys を構成するための組み込みツールは、*netsh.exe* です。 *netsh.exe* を使用して、URL プレフィックスを予約し、X.509 証明書を割り当てることができます。 ツールを使用するには管理者特権が必要です。

   *netsh.exe* ツールを使用して、アプリ用に URL を登録します。

   ```console
   netsh http add urlacl url=<URL> user=<USER>
   ```

   * `<URL>`:完全修飾 URL (Uniform Resource Locator)。 ワイルドカードのバインドは使用しないでください。 有効なホスト名かローカル IP アドレスを使用してください。 "*URL の末尾にはスラッシュが必要です。* "
   * `<USER>`:ユーザーまたはユーザー グループの名前を指定します。

   次の例では、サーバーのローカル IP アドレスは `10.0.0.4` です。

   ```console
   netsh http add urlacl url=https://10.0.0.4:443/ user=Users
   ```

   URL が登録されると、ツールから `URL reservation successfully added` という応答があります。

   登録済みの URL を削除するには、`delete urlacl` コマンドを使用します。

   ```console
   netsh http delete urlacl url=<URL>
   ```

1. サーバーで X.509 証明書を登録します。

   *netsh.exe* ツールを使用して、アプリ用の証明書を登録します。

   ```console
   netsh http add sslcert ipport=<IP>:<PORT> certhash=<THUMBPRINT> appid="{<GUID>}"
   ```

   * `<IP>`:バインド用のローカル IP アドレスを指定します。 ワイルドカードのバインドは使用しないでください。 有効な IP アドレスを使用してください。
   * `<PORT>`:バインド用のポートを指定します。
   * `<THUMBPRINT>`:X.509 証明書の拇印です。
   * `<GUID>`:情報提供を目的として開発者によって生成された、アプリを表す GUID です。

   参照用に、この GUID をパッケージ タグとしてアプリに格納します。

   * Visual Studio:
     * **ソリューション エクスプローラー** 内でアプリを右クリックし、 **[プロパティ]** をクリックして、アプリのプロジェクト プロパティを開きます。
     * **[パッケージ]** タブを選択します。
     * 作成した GUID を **[タグ]** フィールドに入力します。
   * Visual Studio を使用しない場合:
     * アプリのプロジェクト ファイルを開きます。
     * 作成した GUID を指定した `<PackageTags>` プロパティを、新規または既存の `<PropertyGroup>` に追加します。

       ```xml
       <PropertyGroup>
         <PackageTags>9412ee86-c21b-4eb8-bd89-f650fbf44931</PackageTags>
       </PropertyGroup>
       ```

   次に例を示します。

   * サーバーのローカル IP アドレスは `10.0.0.4` です。
   * オンラインのランダム GUID ジェネレーターによって、`appid` の値が提供されます。

   ```console
   netsh http add sslcert 
       ipport=10.0.0.4:443 
       certhash=b66ee04419d4ee37464ab8785ff02449980eae10 
       appid="{9412ee86-c21b-4eb8-bd89-f650fbf44931}"
   ```

   証明書が登録されると、ツールから `SSL Certificate successfully added` という応答があります。

   証明書の登録を削除するには、`delete sslcert` コマンドを使用します。

   ```console
   netsh http delete sslcert ipport=<IP>:<PORT>
   ```

   以下は、*netsh.exe* のリファレンス ドキュメントです。

   * [Netsh Commands for Hypertext Transfer Protocol (HTTP)](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc725882(v=ws.10)) (ハイパーテキスト転送プロトコル (HTTP) 用の Netsh コマンド)
   * [UrlPrefix Strings](/windows/win32/http/urlprefix-strings) (UrlPrefix 文字列)

1. アプリを実行します。

   1024 より大きいポート番号で (HTTPS ではなく) HTTP を使用して localhost にバインドする場合、アプリの実行に管理者権限は必要ありません。 その他の構成の場合 (たとえば、ローカル IP アドレスを使用する場合やポート 443 にバインドする場合)、管理者権限でアプリを実行します。

   サーバーのパブリック IP アドレスでアプリが応答します。 この例では、サーバーは自身のパブリック IP アドレス `104.214.79.47` でインターネットからアクセスされます。

   この例では開発証明書が使用されています。 証明書が信頼できないというブラウザーの警告がバイパスされた後に、ページが安全に読み込まれます。

   ![読み込まれたアプリのインデックス ページを表示するブラウザー ウィンドウ](httpsys/_static/browser.png)

## <a name="proxy-server-and-load-balancer-scenarios"></a>プロキシ サーバーとロード バランサーのシナリオ

インターネットや企業ネットワークからの要求とやりとりする HTTP.sys でホストされるアプリの場合、プロキシ サーバーやロード バランサーの背後でホストするとき、追加の構成が必要になることがあります。 詳細については、「[プロキシ サーバーとロード バランサーを使用するために ASP.NET Core を構成する](xref:host-and-deploy/proxy-load-balancer)」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* [HTTP.sys を使用して Windows 認証を有効にする](xref:security/authentication/windowsauth#httpsys)
* [HTTP サーバー API](/windows/win32/http/http-api-start-page)
* [aspnet/HttpSysServer GitHub リポジトリ (ソース コード)](https://github.com/aspnet/HttpSysServer/)
* [ホスト](xref:fundamentals/index#host)
* <xref:test/troubleshoot>

::: moniker-end
