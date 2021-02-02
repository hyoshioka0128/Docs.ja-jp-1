---
title: ASP.NET Core Kestrel Web サーバーのエンドポイントを構成する
author: rick-anderson
description: ASP.NET Core 用のクロスプラットフォーム Web サーバーである Kestrel でエンドポイントを構成する方法について説明します。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 05/04/2020
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
uid: fundamentals/servers/kestrel/endpoints
ms.openlocfilehash: 5fec573013da5bcb5039b7a189fd84d964349b3a
ms.sourcegitcommit: cc405f20537484744423ddaf87bd1e7d82b6bdf0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/21/2021
ms.locfileid: "98658743"
---
# <a name="configure-endpoints-for-the-aspnet-core-kestrel-web-server"></a>ASP.NET Core Kestrel Web サーバーのエンドポイントを構成する

既定では、ASP.NET Core は以下にバインドされます。

* `http://localhost:5000`
* `https://localhost:5001` (ローカル開発証明書が存在する場合)

以下を使用して URL を指定します。

* `ASPNETCORE_URLS` 環境変数。
* `--urls` コマンド ライン引数。
* `urls` ホスト構成キー。
* `UseUrls` 拡張メソッド。

これらの方法を使うと、1 つまたは複数の HTTP エンドポイントおよび HTTPS エンドポイント (既定の証明書が使用可能な場合は HTTPS) を指定できます。 セミコロン区切りのリストとして値を構成します (例: `"Urls": "http://localhost:8000;http://localhost:8001"`)。

これらの方法について詳しくは、「[サーバーの URL](xref:fundamentals/host/web-host#server-urls)」および「[構成のオーバーライド](xref:fundamentals/host/web-host#override-configuration)」をご覧ください。

開発証明書が作成されます。

* [.NET Core SDK](/dotnet/core/sdk) がインストールされるとき。
* 証明書を作成するには [dev-certs ツール](xref:aspnetcore-2.1#https)を使います。

一部のブラウザーでは、ローカル開発証明書を信頼するには、明示的にアクセス許可を付与する必要があります。

プロジェクト テンプレートでは、アプリを HTTPS で実行するように既定で構成され、[HTTPS のリダイレクトと HSTS のサポート](xref:security/enforcing-ssl)が含まれます。

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> で <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen%2A> または <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket%2A> メソッドを呼び出して、Kestrel 用に URL プレフィックスおよびポートを構成します。

`UseUrls`、`--urls` コマンドライン引数、`urls` ホスト構成キー、`ASPNETCORE_URLS` 環境変数も機能しますが、このセクションで後述する制限があります (既定の証明書が、HTTPS エンドポイントの構成に使用できる必要があります)。

`KestrelServerOptions` 構成:

## <a name="configureendpointdefaultsactionlistenoptions"></a>ConfigureEndpointDefaults(Action\<ListenOptions>)

指定した各エンドポイントに対して実行するように、構成の `Action` を指定します。 `ConfigureEndpointDefaults` を複数回呼び出すと、前の `Action` が最後に指定した `Action` で置き換えられます。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureEndpointDefaults(listenOptions =>
    {
        // Configure endpoint defaults
    });
});
```

> [!NOTE]
> <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureEndpointDefaults%2A> を呼び出す **前に** <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen%2A> を呼び出すことで作成されるエンドポイントには既定値が適用されません。

## <a name="configurehttpsdefaultsactionhttpsconnectionadapteroptions"></a>ConfigureHttpsDefaults(Action\<HttpsConnectionAdapterOptions>)

各 HTTPS エンドポイントに対して実行するように、構成の `Action` を指定します。 `ConfigureHttpsDefaults` を複数回呼び出すと、前の `Action` が最後に指定した `Action` で置き換えられます。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        // certificate is an X509Certificate2
        listenOptions.ServerCertificate = certificate;
    });
});
```

> [!NOTE]
> <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults%2A> を呼び出す **前に** <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen%2A> を呼び出すことで作成されるエンドポイントには既定値が適用されません。

## <a name="configureiconfiguration"></a>Configure(IConfiguration)

入力として <xref:Microsoft.Extensions.Configuration.IConfiguration> を受け取る Kestrel を設定するための構成ローダーを作成します。 構成のスコープは、Kestrel の構成セクションにする必要があります。

`CreateDefaultBuilder` は既定で `Configure(context.Configuration.GetSection("Kestrel"))` を呼び出して Kestrel の構成を読み込みます。

```json
{
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5000"
      },
      "Https": {
        "Url": "https://localhost:5001",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      }
    }
  }
}
```

## <a name="listenoptionsusehttps"></a>ListenOptions.UseHttps

HTTPS を使用するように Kestrel を構成します。

`ListenOptions.UseHttps` 拡張機能:

* `UseHttps`:既定の証明書で HTTPS を使うように Kestrel を構成します。 既定の証明書が構成されていない場合は、例外がスローされます。
* `UseHttps(string fileName)`
* `UseHttps(string fileName, string password)`
* `UseHttps(string fileName, string password, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(StoreName storeName, string subject)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(X509Certificate2 serverCertificate)`
* `UseHttps(X509Certificate2 serverCertificate, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(Action<HttpsConnectionAdapterOptions> configureOptions)`

`ListenOptions.UseHttps` パラメーター:

* `filename` は、アプリのコンテンツ ファイルが格納されているディレクトリを基準とする、証明書ファイルのパスとファイル名です。
* `password` は、X.509 証明書データにアクセスするために必要なパスワードです。
* `configureOptions` は、`HttpsConnectionAdapterOptions` を構成するための `Action` です。 `ListenOptions` を返します。
* `storeName` は、証明書の読み込み元の証明書ストアです。
* `subject` は、証明書のサブジェクト名です。
* `allowInvalid` は、無効な証明書 (自己署名証明書など) を考慮する必要があるかどうかを示します。
* `location` は、証明書を読み込むストアの場所です。
* `serverCertificate` は、X.509 証明書です。

運用環境では、HTTPS を明示的に構成する必要があります。 少なくとも、既定の証明書を指定する必要があります。

サポートされている構成を次に説明します。

* 構成なし
* 構成から既定の証明書を置き換える
* コードで既定値を変更する

### <a name="no-configuration"></a>構成なし

Kestrel は、`http://localhost:5000` と `https://localhost:5001` (既定の証明書が使用可能な場合) でリッスンします。

<a name="configuration"></a>

### <a name="replace-the-default-certificate-from-configuration"></a>構成から既定の証明書を置き換える

Kestrel は、既定の HTTPS アプリ設定構成スキーマを使用できます。 ディスク上のファイルまたは証明書ストアから、URL や使用する証明書など、複数のエンドポイントを構成します。

次の *appsettings.json* の例では以下のようになります。

* `AllowInvalid` を `true` に設定し、の無効な証明書 (自己署名証明書など) の使用を許可します。
* 証明書 (この後の例では `HttpsDefaultCert`) が指定されていないすべての HTTPS エンドポイントは、`Certificates:Default` または開発証明書で定義されている証明書にフォールバックします。

```json
{
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5000"
      },
      "HttpsInlineCertFile": {
        "Url": "https://localhost:5001",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      },
      "HttpsInlineCertAndKeyFile": {
        "Url": "https://localhost:5002",
        "Certificate": {
          "Path": "<path to .pem/.crt file>",
          "KeyPath": "<path to .key file>",
          "Password": "<certificate password>"
        }
      },
      "HttpsInlineCertStore": {
        "Url": "https://localhost:5003",
        "Certificate": {
          "Subject": "<subject; required>",
          "Store": "<certificate store; required>",
          "Location": "<location; defaults to CurrentUser>",
          "AllowInvalid": "<true or false; defaults to false>"
        }
      },
      "HttpsDefaultCert": {
        "Url": "https://localhost:5004"
      }
    },
    "Certificates": {
      "Default": {
        "Path": "<path to .pfx file>",
        "Password": "<certificate password>"
      }
    }
  }
}
```

スキーマに関する注意事項:

* エンドポイント名は[大文字と小文字が区別されます](xref:fundamentals/configuration/index#configuration-keys-and-values)。 たとえば、 `HTTPS` and `Https` は同等です。
* `Url` パラメーターは、エンドポイントごとに必要です。 このパラメーターの形式は、1 つの値に制限されることを除き、最上位レベルの `Urls` 構成パラメーターと同じです。
* これらのエンドポイントは、最上位レベルの `Urls` 構成での定義に追加されるのではなく、それを置き換えます。 コードで `Listen` を使用して定義されているエンドポイントは、構成セクションで定義されているエンドポイントに累積されます。
* `Certificate` セクションは省略可能です。 `Certificate` セクションを指定しないと、`Certificates:Default` で定義した既定値が使用されます。 既定値を使用できない場合は、開発証明書が使用されます。 既定値がなく、開発証明書も存在しない場合、サーバーにより例外がスローされ、起動に失敗します。
* `Certificate` セクションでは複数の[証明書のソース](#certificate-sources)がサポートされています。
* ポートが競合しない限り、[構成](xref:fundamentals/configuration/index)で任意の数のエンドポイントを定義できます。

#### <a name="certificate-sources"></a>証明書のソース

さまざまなソースから証明書を読み込むように証明書ノードを構成することができます。

* *.pfx* ファイルを読み込むための `Path` と `Password`。
* *.pem*/ *.crt* および *.key* ファイルを読み込むための `Path`、`KeyPath`、および `Password`。
* 証明書ストアから読み込むための `Subject` と `Store`。

たとえば、 `Certificates:Default`   の証明書は次のように指定できます。

```json
"Default": {
  "Subject": "<subject; required>",
  "Store": "<cert store; required>",
  "Location": "<location; defaults to CurrentUser>",
  "AllowInvalid": "<true or false; defaults to false>"
}
```

#### <a name="configurationloader"></a>ConfigurationLoader

`options.Configure(context.Configuration.GetSection("{SECTION}"))` が `.Endpoint(string name, listenOptions => { })` メソッドで返す <xref:Microsoft.AspNetCore.Server.Kestrel.KestrelConfigurationLoader> を使用して、構成されているエンドポイントの設定を補足できます。

```csharp
webBuilder.UseKestrel((context, serverOptions) =>
{
    serverOptions.Configure(context.Configuration.GetSection("Kestrel"))
        .Endpoint("HTTPS", listenOptions =>
        {
            listenOptions.HttpsOptions.SslProtocols = SslProtocols.Tls12;
        });
});
```

`KestrelServerOptions.ConfigurationLoader` に直接アクセスして、既存のローダー (<xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder%2A> によって提供されるものなど) の反復を維持することができます。

* カスタム設定を読み取ることができるように、各エンドポイントの構成セクションを `Endpoint` メソッドのオプションで使用できます。
* 別のセクションで `options.Configure(context.Configuration.GetSection("{SECTION}"))` を再び呼び出すことにより、複数の構成を読み込むことができます。 前のインスタンスで `Load` を明示的に呼び出していない限り、最後の構成のみが使用されます。 既定の構成セクションを置き換えることができるように、メタパッケージは `Load` を呼び出しません。
* `KestrelConfigurationLoader` はミラー、`KestrelServerOptions` から API の `Listen` ファミリを `Endpoint` オーバーロードとしてミラーリングするので、コードと構成のエンドポイントを同じ場所で構成できます。 これらのオーバーロードでは名前は使用されず、構成からの既定の設定のみが使用されます。

### <a name="change-the-defaults-in-code"></a>コードで既定値を変更する

`ConfigureEndpointDefaults` および`ConfigureHttpsDefaults` を使用して、`ListenOptions` および `HttpsConnectionAdapterOptions` の既定の設定を変更できます (前のシナリオで指定した既定の証明書のオーバーライドなど)。 エンドポイントを構成する前に、`ConfigureEndpointDefaults` および `ConfigureHttpsDefaults` を呼び出す必要があります。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureEndpointDefaults(listenOptions =>
    {
        // Configure endpoint defaults
    });

    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        listenOptions.SslProtocols = SslProtocols.Tls12;
    });
});
```

## <a name="configure-endpoints-using-server-name-indication"></a>Server Name Indication を使用してエンドポイントを構成する

[Server Name Indication (SNI)](https://tools.ietf.org/html/rfc6066#section-3) を使用すると、同じ IP アドレスとポートで複数のドメインをホストできます。 SNI が機能するためには、サーバーが正しい証明書を提供できるように、クライアントは TLS ハンドシェイクの間にセキュリティで保護されたセッションのホスト名をサーバーに送信します。 クライアントは、TLS ハンドシェイクに続くセキュリティで保護されたセッション中に、提供された証明書をサーバーとの暗号化された通信に使用します。

Kestrel は、`ServerCertificateSelector` コールバックによって SNI をサポートします。 コールバックは接続ごとに 1 回呼び出されるので、アプリはそれを使って、ホスト名を検査し、適切な証明書を選択できます。 次のコールバック コードは、プロジェクトの *Program.cs* ファイルの `ConfigureWebHostDefaults` メソッドの呼び出しで使用できます。

```csharp
//using System.Security.Cryptography.X509Certificates;
//using Microsoft.AspNetCore.Server.Kestrel.Https;

webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ListenAnyIP(5005, listenOptions =>
    {
        listenOptions.UseHttps(httpsOptions =>
        {
            var localhostCert = CertificateLoader.LoadFromStoreCert(
                "localhost", "My", StoreLocation.CurrentUser,
                allowInvalid: true);
            var exampleCert = CertificateLoader.LoadFromStoreCert(
                "example.com", "My", StoreLocation.CurrentUser,
                allowInvalid: true);
            var subExampleCert = CertificateLoader.LoadFromStoreCert(
                "sub.example.com", "My", StoreLocation.CurrentUser,
                allowInvalid: true);
            var certs = new Dictionary<string, X509Certificate2>(StringComparer.OrdinalIgnoreCase)
            {
                { "localhost", localhostCert },
                { "example.com", exampleCert },
                { "sub.example.com", subExampleCert },
            };            

            httpsOptions.ServerCertificateSelector = (connectionContext, name) =>
            {
                if (name != null && certs.TryGetValue(name, out var cert))
                {
                    return cert;
                }

                return exampleCert;
            };
        });
    });
});
```

SNI のサポートには、次が必要です。

* ターゲット フレームワーク `netcoreapp2.1` 以降で実行される必要があります。 `net461` 以降、コールバックは呼び出されますが、`name` は常に `null` です。 クライアントが TLS ハンドシェイクでホスト名パラメーターを提供しない場合も、`name` は `null` です。
* すべての Web サイトは、同じ Kestrel インスタンスで実行されます。 Kestrel では、リバース プロキシは使用しない、複数のインスタンスでの IP アドレスとポートの共有はサポートしていません。

## <a name="connection-logging"></a>接続のログ記録

接続でのバイト レベルの通信に対してデバッグ レベルのログを出力するには、<xref:Microsoft.AspNetCore.Hosting.ListenOptionsConnectionLoggingExtensions.UseConnectionLogging%2A> を呼び出します。 接続のログ記録は、TLS 暗号化の間やプロキシの内側など、低レベルの通信での問題のトラブルシューティングに役立ちます。 `UseConnectionLogging` が `UseHttps` の前に配置されている場合は、暗号化されたトラフィックがログに記録されます。 `UseConnectionLogging` が `UseHttps` の後に配置されている場合は、暗号化解除されたトラフィックがログに記録されます。 これは、[接続ミドルウェア](#connection-middleware)に組み込まれています。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseConnectionLogging();
    });
});
```

## <a name="bind-to-a-tcp-socket"></a>TCP ソケットにバインドする

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen%2A> メソッドは TCP ソケットにバインドし、オプションのラムダにより X.509 証明書を構成できます。

[!code-csharp[](samples/5.x/KestrelSample/Program.cs?name=snippet_TCPSocket&highlight=12-18)]

例では、<xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions> を使用して、エンドポイントに対する HTTPS が構成されます。 同じ API を使用して、特定のエンドポイントに対する他の Kestrel 設定を構成します。

[!INCLUDE [How to make an X.509 cert](~/includes/make-x509-cert.md)]

## <a name="bind-to-a-unix-socket"></a>UNIX ソケットにバインドする

次の例に示すように、Nginx でのパフォーマンスの向上のために <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket%2A> を使って UNIX ソケットをリッスンします。

[!code-csharp[](samples/5.x/KestrelSample/Program.cs?name=snippet_UnixSocket)]

* Nginx 構成ファイルで `server` > `location` > `proxy_pass` エントリを `http://unix:/tmp/{KESTREL SOCKET}:/;` に設定します。 `{KESTREL SOCKET}` は <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket%2A> に提供されたソケットの名前です (たとえば、前の例の `kestrel-test.sock`)。
* ソケットが Nginx によって書き込み可能であることを確認します (たとえば、`chmod go+w /tmp/kestrel-test.sock`)。

## <a name="port-0"></a>ポート 0

ポート番号 `0` を指定すると、Kestrel は使用可能なポートに動的にバインドします。 次の例では、実行時に Kestrel によってバインドされたポートを特定する方法について説明します。

[!code-csharp[](samples/5.x/KestrelSample/Startup.cs?name=snippet_Configure&highlight=3-4,15-21)]

アプリを実行すると、コンソール ウィンドウの出力で、アプリがアクセスできる動的なポートが示されます。

```console
Listening on the following addresses: http://127.0.0.1:48508
```

## <a name="limitations"></a>制限事項

次の方法でエンドポイントを構成します。

* <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls%2A>
* `--urls` コマンド ライン引数
* `urls` ホスト構成キー
* `ASPNETCORE_URLS` 環境変数

コードを Kestrel 以外のサーバーで機能させるには、これらのメソッドが有用です。 ただし、次の使用制限があることに注意してください。

* HTTPS エンドポイントの構成 (`KestrelServerOptions` 構成、またはこの記事で前に示した構成ファイルなどを使用) で既定の証明書が指定されていない場合は、これらの方法で HTTPS を使用することはできません。
* `Listen` と `UseUrls` 両方の方法を同時に使用すると、`Listen` エンドポイントが `UseUrls` エンドポイントをオーバーライドします。

## <a name="iis-endpoint-configuration"></a>IIS エンドポイントの構成

IIS を使用するときは、IIS オーバーライド バインドに対する URL バインドを、`Listen` または `UseUrls` によって設定します。 詳細については、[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)に関するページを参照してください。

## <a name="listenoptionsprotocols"></a>ListenOptions.Protocols

`Protocols`プロパティでは、接続エンドポイントまたはサーバーに対して有効にされる HTTP プロトコル (`HttpProtocols`) が確定されます。 `HttpProtocols` 列挙型から `Protocols` プロパティに値を割り当てます。

| `HttpProtocols` 列挙値 | 許可される接続プロトコル |
| -------------------------- | ----------------------------- |
| `Http1`                    | HTTP/1.1 のみ。 TLS の有無にかかわらず使用できます。 |
| `Http2`                    | HTTP/2 のみ。 [予備知識モード](https://tools.ietf.org/html/rfc7540#section-3.4)がクライアントでサポートされている場合に限り、TLS なしで使用できます。 |
| `Http1AndHttp2`            | HTTP/1.1 および HTTP/2。 HTTP/2 では、クライアントが TLS [アプリケーション レイヤー プロトコル ネゴシエーション (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) ハンドシェイクで HTTP/2 を選択する必要があります。それ以外の場合、接続は既定で HTTP/1.1 となります。 |

任意のエンドポイントに対する `ListenOptions.Protocols` の既定値は `HttpProtocols.Http1AndHttp2` です。

HTTP/2 に対する TLS 制限事項:

* TLS バージョン 1.2 以降
* 再ネゴシエーションは無効
* 圧縮は無効
* 短期キー交換サイズの上限:
  * Elliptic curve Diffie-hellman (ECDHE) &lbrack;[RFC4492](https://www.ietf.org/rfc/rfc4492.txt)&rbrack;:224 ビット以上
  * Finite field Diffie-Hellman (DHE) &lbrack;`TLS12`&rbrack;:2048 ビット以上
* 暗号スイートは禁止されない 

`TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256` &lbrack;`TLS-ECDHE`&rbrack; と P-256 elliptic curve &lbrack;`FIPS186`&rbrack; の組み合わせは既定ではサポートされています。

次の例では、HTTP/1.1 接続と HTTP/2 接続がポート 8000 上で許可されています。 接続は、TLS と指定した証明書によってセキュリティで保護されます。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseHttps("testCert.pfx", "testPassword");
    });
});
```

Linux では、<xref:System.Net.Security.CipherSuitesPolicy> を使って、接続ごとに TLS ハンドシェイクをフィルター処理できます。

```csharp
// using System.Net.Security;
// using Microsoft.AspNetCore.Hosting;
// using Microsoft.AspNetCore.Server.Kestrel.Core;
// using Microsoft.Extensions.DependencyInjection;
// using Microsoft.Extensions.Hosting;

webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        listenOptions.OnAuthenticate = (context, sslOptions) =>
        {
            sslOptions.CipherSuitesPolicy = new CipherSuitesPolicy(
                new[]
                {
                    TlsCipherSuite.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,
                    TlsCipherSuite.TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,
                    // ...
                });
        };
    });
});
```

## <a name="connection-middleware"></a>接続ミドルウェア

カスタムの接続ミドルウェアを使用すると、必要に応じて、特定の暗号に対し、接続ごとに TLS ハンドシェイクをフィルター処理することができます。

次の例では、アプリでサポートされていないすべての暗号アルゴリズムに対して <xref:System.NotSupportedException> がスローされます。 または、[ITlsHandshakeFeature.CipherAlgorithm](xref:Microsoft.AspNetCore.Connections.Features.ITlsHandshakeFeature.CipherAlgorithm) を定義して、指定できる暗号スイートの一覧と比較します。

[CipherAlgorithmType.Null](xref:System.Security.Authentication.CipherAlgorithmType) 暗号アルゴリズムでは、暗号化は使用されません。

```csharp
// using System.Net;
// using Microsoft.AspNetCore.Connections;

webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseHttps("testCert.pfx", "testPassword");
        listenOptions.UseTlsFilter();
    });
});
```

```csharp
using System;
using System.Security.Authentication;
using Microsoft.AspNetCore.Connections.Features;

namespace Microsoft.AspNetCore.Connections
{
    public static class TlsFilterConnectionMiddlewareExtensions
    {
        public static IConnectionBuilder UseTlsFilter(
            this IConnectionBuilder builder)
        {
            return builder.Use((connection, next) =>
            {
                var tlsFeature = connection.Features.Get<ITlsHandshakeFeature>();

                if (tlsFeature.CipherAlgorithm == CipherAlgorithmType.Null)
                {
                    throw new NotSupportedException("Prohibited cipher: " +
                        tlsFeature.CipherAlgorithm);
                }

                return next();
            });
        }
    }
}
```

接続のフィルター処理は、<xref:Microsoft.AspNetCore.Connections.IConnectionBuilder> のラムダを使って構成することもできます。

```csharp
// using System;
// using System.Net;
// using System.Security.Authentication;
// using Microsoft.AspNetCore.Connections;
// using Microsoft.AspNetCore.Connections.Features;

webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseHttps("testCert.pfx", "testPassword");
        listenOptions.Use((context, next) =>
        {
            var tlsFeature = context.Features.Get<ITlsHandshakeFeature>();

            if (tlsFeature.CipherAlgorithm == CipherAlgorithmType.Null)
            {
                throw new NotSupportedException(
                    $"Prohibited cipher: {tlsFeature.CipherAlgorithm}");
            }

            return next();
        });
    });
});
```

## <a name="set-the-http-protocol-from-configuration"></a>HTTP プロトコルを構成から設定する

`CreateDefaultBuilder` は既定で `serverOptions.Configure(context.Configuration.GetSection("Kestrel"))` を呼び出して Kestrel の構成を読み込みます。

次の *appsettings.json* の例では、すべてのエンドポイントにおける既定の接続プロトコルとして HTTP/1.1 を確立しています。

```json
{
  "Kestrel": {
    "EndpointDefaults": {
      "Protocols": "Http1"
    }
  }
}
```

次の *appsettings.json* の例では、特定のエンドポイントにおける HTTP/1.1 接続プロトコルを確立しています。

```json
{
  "Kestrel": {
    "Endpoints": {
      "HttpsDefaultCert": {
        "Url": "https://localhost:5001",
        "Protocols": "Http1"
      }
    }
  }
}
```

構成で設定された値は、コード内で指定されたプロトコルによってオーバーライドされます。

## <a name="url-prefixes"></a>URL プレフィックス

`UseUrls`、`--urls` コマンドライン引数、`urls` ホスト構成キー、または `ASPNETCORE_URLS` 環境変数を使用する場合、URL プレフィックスは次のいずれかの形式となります。

HTTP URL プレフィックスのみが有効です。 `UseUrls` を使用して URL バインドを構成する場合、kestrel は HTTPS をサポートしません。

* IPv4 アドレスとポート番号

  ```
  http://65.55.39.10:80/
  ```

  `0.0.0.0` は、すべての IPv4 アドレスにバインドする特別なケースです。

* IPv6 アドレスとポート番号

  ```
  http://[0:0:0:0:0:ffff:4137:270a]:80/
  ```

  IPv6 の `[::]` は IPv4 の `0.0.0.0` に相当します。

* ホスト名とポート番号

  ```
  http://contoso.com:80/
  http://*:80/
  ```

  ホスト名、`*`、および `+` は特別ではありません。 有効な IP アドレスまたは `localhost` と認識されないものはすべて、IPv4 および IPv6 のすべての IP にバインドします。 さまざまなホスト名を、同じポート上のさまざまな ASP.NET Core アプリにバインドするには、[HTTP.sys](xref:fundamentals/servers/httpsys) かリバース プロキシ サーバーを使用します。 リバース プロキシ サーバーの例としては、IIS、Nginx、Apache などがあります。

  > [!WARNING]
  > リバース プロキシ構成でのホストには[ホストのフィルター処理](xref:fundamentals/servers/kestrel/host-filtering)が必要です。

* `localhost` 名とポート番号、またはループバック IP とポート番号

  ```
  http://localhost:5000/
  http://127.0.0.1:5000/
  http://[::1]:5000/
  ```

  `localhost` が指定された場合、Kestrel は IPv4 ループバック インターフェイスと IPv6 ループバック インターフェイスの両方へのバインドを試みます。 要求されたポートがいずれかのループバック インターフェイス上の別のサービスで使用中である場合、Kestrel は開始できません。 他の何らかの理由でいずれかのループバック インターフェイスが使用できない場合 (ほとんどの場合は IPv6 がサポートされていないことが理由)、Kestrel は警告をログに記録します。
