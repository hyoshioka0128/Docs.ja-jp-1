---
title: ASP.NET Core で証明書認証を構成する
author: blowdart
description: IIS および HTTP.sys の ASP.NET Core で証明書認証を構成する方法について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: bdorrans
ms.date: 07/16/2020
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
uid: security/authentication/certauth
ms.openlocfilehash: c862bc8bff6c4cc80696d92067e814889d6e7782
ms.sourcegitcommit: 7e394a8527c9818caebb940f692ae4fcf2f1b277
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/31/2021
ms.locfileid: "99217532"
---
# <a name="configure-certificate-authentication-in-aspnet-core"></a>ASP.NET Core で証明書認証を構成する

`Microsoft.AspNetCore.Authentication.Certificate` ASP.NET Core の [証明書認証](https://tools.ietf.org/html/rfc5246#section-7.4.4) に似た実装が含まれています。 証明書の認証は、ASP.NET Core に到達するずっと前に TLS レベルで実行されます。 より正確には、これは証明書を検証し、その証明書をに解決できるイベントを提供する認証ハンドラーです `ClaimsPrincipal` 。 

証明書認証用に[サーバーを構成](#configure-your-server-to-require-certificates)します。これは、IIS、Kestrel、Azure Web Apps など、使用している他の任意のものです。

## <a name="proxy-and-load-balancer-scenarios"></a>プロキシとロードバランサーのシナリオ

証明書認証は、主に、プロキシまたはロードバランサーがクライアントとサーバー間のトラフィックを処理しない場合に使用されるステートフルシナリオです。 プロキシまたはロードバランサーが使用されている場合、証明書の認証はプロキシまたはロードバランサーの場合にのみ機能します。

* 認証を処理します。
* 認証情報に対して動作するユーザー認証情報をアプリに渡します (要求ヘッダーなど)。

プロキシとロードバランサーを使用する環境での証明書認証の代わりに、OpenID Connect (OIDC) を使用したフェデレーションサービス (ADFS) Active Directory ます。

## <a name="get-started"></a>開始

HTTPS 証明書を取得して適用し、証明書を要求するように [サーバーを構成](#configure-your-server-to-require-certificates) します。

Web アプリで、 [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Certificate) パッケージへの参照を追加します。 次に、 `Startup.ConfigureServices` メソッドで、オプションを指定してを呼び出し `services.AddAuthentication(CertificateAuthenticationDefaults.AuthenticationScheme).AddCertificate(...);` 、 `OnCertificateValidated` 要求と共に送信されるクライアント証明書に対して補助的な検証を実行するためのデリゲートを提供します。 その情報をに変換 `ClaimsPrincipal` し、プロパティに設定し `context.Principal` ます。

認証が失敗した場合、このハンドラーは、 `403 (Forbidden)` 予期したとおりに応答を返し `401 (Unauthorized)` ます。 これは、最初の TLS 接続中に認証が行われるということです。 ハンドラーに到達するまでには遅すぎます。 匿名接続から証明書を使用して接続をアップグレードする方法はありません。

また `app.UseAuthentication();` 、メソッドにを追加 `Startup.Configure` します。 それ以外の場合、は `HttpContext.User` `ClaimsPrincipal` 証明書から作成されるように設定されません。 例:

::: moniker range=">= aspnetcore-5.0"

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(
        CertificateAuthenticationDefaults.AuthenticationScheme)
        .AddCertificate()
        // Adding an ICertificateValidationCache results in certificate auth caching the results.
        // The default implementation uses a memory cache.
        .AddCertificateCache();

    // All other service configuration
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseAuthentication();

    // All other app configuration
}
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(
        CertificateAuthenticationDefaults.AuthenticationScheme)
        .AddCertificate();

    // All other service configuration
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseAuthentication();

    // All other app configuration
}
```

::: moniker-end

前の例では、証明書認証を追加する既定の方法を示しています。 ハンドラーは、共通の証明書のプロパティを使用してユーザープリンシパルを構築します。

## <a name="configure-certificate-validation"></a>証明書の検証の構成

ハンドラーには、 `CertificateAuthenticationOptions` 証明書に対して実行する必要のある検証の最小値である組み込みの検証がいくつかあります。 これらの各設定は既定で有効になっています。

### <a name="allowedcertificatetypes--chained-selfsigned-or-all-chained--selfsigned"></a>AllowedCertificateTypes = チェーン、自己署名、またはすべて (チェーン |自己署名済み)

既定値:`CertificateTypes.Chained`

このチェックでは、適切な証明書の種類のみが許可されていることが検証されます。 アプリで自己署名証明書を使用している場合は、このオプションをまたはに設定する必要があり `CertificateTypes.All` `CertificateTypes.SelfSigned` ます。

### <a name="validatecertificateuse"></a>ValidateCertificateUse

既定値:`true`

このチェックでは、クライアントが提示した証明書にクライアント認証の拡張キー使用法 (EKU) があること、またはまったく Eku がないことを検証します。 仕様として、EKU が指定されていない場合は、すべての Eku が有効と見なされます。

### <a name="validatevalidityperiod"></a>ValidateValidityPeriod

既定値:`true`

このチェックでは、証明書が有効期間内であることを検証します。 要求が発生するたびに、ハンドラーは、提示されたときに有効だった証明書が現在のセッション中に期限切れにならないようにします。

### <a name="revocationflag"></a>RevocationFlag

既定値:`X509RevocationFlag.ExcludeRoot`

チェーン内のどの証明書の失効を確認するかを指定するフラグ。

失効確認は、証明書がルート証明書にチェーンされている場合にのみ実行されます。

### <a name="revocationmode"></a>RevocationMode

既定値:`X509RevocationMode.Online`

失効確認の実行方法を指定するフラグ。

オンラインチェックを指定すると、証明機関への接続中に長い遅延が発生する可能性があります。

失効確認は、証明書がルート証明書にチェーンされている場合にのみ実行されます。

### <a name="can-i-configure-my-app-to-require-a-certificate-only-on-certain-paths"></a>特定のパスでのみ証明書を要求するようにアプリを構成することはできますか。

これはできません。 証明書の交換は、HTTPS メッセージ交換の開始時に行われます。これは、その接続で最初の要求を受信する前にサーバーによって行われるため、要求フィールドに基づいてスコープを設定することはできません。

## <a name="handler-events"></a>ハンドラーイベント

ハンドラーには、次の2つのイベントがあります。

* `OnAuthenticationFailed`: 認証中に例外が発生し、対応できる場合に呼び出されます。
* `OnCertificateValidated`: 証明書が検証され、検証が渡され、既定のプリンシパルが作成された後に呼び出されます。 このイベントを使用すると、独自の検証を実行したり、プリンシパルを補強したり置き換えることができます。 例を次に示します。
  * 証明書がサービスに対して認識されているかどうかを確認しています。
  * 独自のプリンシパルを構築します。 `Startup.ConfigureServices` での次の例を検討してください。

    ```csharp
    services.AddAuthentication(
        CertificateAuthenticationDefaults.AuthenticationScheme)
        .AddCertificate(options =>
        {
            options.Events = new CertificateAuthenticationEvents
            {
                OnCertificateValidated = context =>
                {
                    var claims = new[]
                    {
                        new Claim(
                            ClaimTypes.NameIdentifier, 
                            context.ClientCertificate.Subject,
                            ClaimValueTypes.String, 
                            context.Options.ClaimsIssuer),
                        new Claim(ClaimTypes.Name,
                            context.ClientCertificate.Subject,
                            ClaimValueTypes.String, 
                            context.Options.ClaimsIssuer)
                    };
    
                    context.Principal = new ClaimsPrincipal(
                        new ClaimsIdentity(claims, context.Scheme.Name));
                    context.Success();
    
                    return Task.CompletedTask;
                }
            };
        });
    ```

受信証明書が追加の検証を満たしていない場合は、 `context.Fail("failure reason")` 失敗の理由でを呼び出します。

実際の機能の場合は、データベースまたはその他の種類のユーザーストアに接続する依存関係の挿入に登録されているサービスを呼び出す必要があります。 デリゲートに渡されたコンテキストを使用してサービスにアクセスします。 `Startup.ConfigureServices` での次の例を検討してください。

```csharp
services.AddAuthentication(
    CertificateAuthenticationDefaults.AuthenticationScheme)
    .AddCertificate(options =>
    {
        options.Events = new CertificateAuthenticationEvents
        {
            OnCertificateValidated = context =>
            {
                var validationService =
                    context.HttpContext.RequestServices
                        .GetRequiredService<ICertificateValidationService>();
                
                if (validationService.ValidateCertificate(
                    context.ClientCertificate))
                {
                    var claims = new[]
                    {
                        new Claim(
                            ClaimTypes.NameIdentifier, 
                            context.ClientCertificate.Subject, 
                            ClaimValueTypes.String, 
                            context.Options.ClaimsIssuer),
                        new Claim(
                            ClaimTypes.Name, 
                            context.ClientCertificate.Subject, 
                            ClaimValueTypes.String, 
                            context.Options.ClaimsIssuer)
                    };

                    context.Principal = new ClaimsPrincipal(
                        new ClaimsIdentity(claims, context.Scheme.Name));
                    context.Success();
                }                     

                return Task.CompletedTask;
            }
        };
    });
```

概念的には、証明書の検証は承認に関する問題です。 内部ではなく、承認ポリシーの発行者や拇印などのチェックを追加すること `OnCertificateValidated` は、まったく可能です。

## <a name="configure-your-server-to-require-certificates"></a>証明書を要求するようにサーバーを構成する

### <a name="kestrel"></a>Kestrel

*Program.cs* で、次のように Kestrel を構成します。

```csharp
public static void Main(string[] args)
{
    CreateHostBuilder(args).Build().Run();
}

public static IHostBuilder CreateHostBuilder(string[] args)
{
    return Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
            webBuilder.ConfigureKestrel(o =>
            {
                o.ConfigureHttpsDefaults(o => 
            o.ClientCertificateMode = 
                ClientCertificateMode.RequireCertificate);
            });
        });
}
```

> [!NOTE]
> <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults*> を呼び出す **前に** <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> を呼び出すことで作成されるエンドポイントには既定値が適用されません。

### <a name="iis"></a>IIS

IIS マネージャーで、次の手順を実行します。

1. [ **接続** ] タブからサイトを選択します。
1. [**機能ビュー** ] ウィンドウで、[ **SSL 設定**] オプションをダブルクリックします。
1. [ **SSL が必要**] チェックボックスをオンにし、[**クライアント証明書**] セクションの [**必須**] オプションを選択します。

![IIS でのクライアント証明書の設定](README-IISConfig.png)

### <a name="azure-and-custom-web-proxies"></a>Azure とカスタム web プロキシ

証明書転送ミドルウェアを構成する方法については、 [ホストを参照し、ドキュメントを展開](xref:host-and-deploy/proxy-load-balancer#certificate-forwarding) してください。

### <a name="use-certificate-authentication-in-azure-web-apps"></a>Azure Web Apps で証明書認証を使用する

Azure では、転送構成は必要ありません。 これは既に証明書転送ミドルウェアで設定されています。

> [!NOTE]
> これを行うには、CertificateForwardingMiddleware が存在している必要があります。

### <a name="use-certificate-authentication-in-custom-web-proxies"></a>カスタム web プロキシで証明書認証を使用する

メソッドを使用して、 `AddCertificateForwarding` 次のように指定します。

* クライアントヘッダー名。
* (プロパティを使用して) 証明書を読み込む方法 `HeaderConverter` 。

カスタム web プロキシでは、証明書はカスタム要求ヘッダーとして渡されます。たとえば、のように `X-SSL-CERT` なります。 これを使用するには、次のように証明書の転送を構成し `Startup.ConfigureServices` ます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddCertificateForwarding(options =>
    {
        options.CertificateHeader = "X-SSL-CERT";
        options.HeaderConverter = (headerValue) =>
        {
            X509Certificate2 clientCertificate = null;

            if(!string.IsNullOrWhiteSpace(headerValue))
            {
                byte[] bytes = StringToByteArray(headerValue);
                clientCertificate = new X509Certificate2(bytes);
            }

            return clientCertificate;
        };
    });
}

private static byte[] StringToByteArray(string hex)
{
    int NumberChars = hex.Length;
    byte[] bytes = new byte[NumberChars / 2];

    for (int i = 0; i < NumberChars; i += 2)
    {
        bytes[i / 2] = Convert.ToByte(hex.Substring(i, 2), 16);
    }

    return bytes;
}
```

次に、 `Startup.Configure` メソッドはミドルウェアを追加します。 `UseCertificateForwarding` は、およびの呼び出しの前に呼び出され `UseAuthentication` `UseAuthorization` ます。

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    ...

    app.UseRouting();

    app.UseCertificateForwarding();
    app.UseAuthentication();
    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```

別のクラスを使用して、検証ロジックを実装できます。 この例では同じ自己署名証明書が使用されているため、使用できるのは証明書のみであることを確認してください。 クライアント証明書とサーバー証明書の両方の拇印が一致しているかどうかを検証します。それ以外の場合は、証明書を使用して、認証に十分なものになります。 これは、メソッドの内部で使用され `AddCertificate` ます。 中間証明書または子証明書を使用している場合は、ここでサブジェクトまたは発行者を検証することもできます。

```csharp
using System.IO;
using System.Security.Cryptography.X509Certificates;

namespace AspNetCoreCertificateAuthApi
{
    public class MyCertificateValidationService
    {
        public bool ValidateCertificate(X509Certificate2 clientCertificate)
        {
            // Do not hardcode passwords in production code
            // Use thumbprint or key vault
            var cert = new X509Certificate2(
                Path.Combine("sts_dev_cert.pfx"), "1234");

            if (clientCertificate.Thumbprint == cert.Thumbprint)
            {
                return true;
            }

            return false;
        }
    }
}
```

#### <a name="implement-an-httpclient-using-a-certificate-and-the-httpclienthandler"></a>証明書と HttpClientHandler を使用して HttpClient を実装する

は、 `HttpClientHandler` クラスのコンストラクターに直接追加でき `HttpClient` ます。 のインスタンスを作成するときは注意が必要 `HttpClient` です。 次に、は、 `HttpClient` 要求ごとに証明書を送信します。

```csharp
private async Task<JsonDocument> GetApiDataUsingHttpClientHandler()
{
    var cert = new X509Certificate2(Path.Combine(_environment.ContentRootPath, "sts_dev_cert.pfx"), "1234");
    var handler = new HttpClientHandler();
    handler.ClientCertificates.Add(cert);
    var client = new HttpClient(handler);
     
    var request = new HttpRequestMessage()
    {
        RequestUri = new Uri("https://localhost:44379/api/values"),
        Method = HttpMethod.Get,
    };
    var response = await client.SendAsync(request);
    if (response.IsSuccessStatusCode)
    {
        var responseContent = await response.Content.ReadAsStringAsync();
        var data = JsonDocument.Parse(responseContent);
        return data;
    }
 
    throw new ApplicationException($"Status code: {response.StatusCode}, Error: {response.ReasonPhrase}");
}
```

#### <a name="implement-an-httpclient-using-a-certificate-and-a-named-httpclient-from-ihttpclientfactory"></a>IHttpClientFactory から証明書と名前付き HttpClient を使用して HttpClient を実装する 

次の例では、 `HttpClientHandler` ハンドラーのプロパティを使用して、クライアント証明書をに追加し `ClientCertificates` ます。 このハンドラーは、 `HttpClient` メソッドを使用して、の名前付きインスタンスで使用でき `ConfigurePrimaryHttpMessageHandler` ます。 次のように設定されてい `Startup.ConfigureServices` ます。

```csharp
var clientCertificate = 
    new X509Certificate2(
      Path.Combine(_environment.ContentRootPath, "sts_dev_cert.pfx"), "1234");
 
var handler = new HttpClientHandler();
handler.ClientCertificates.Add(clientCertificate);
 
services.AddHttpClient("namedClient", c =>
{
}).ConfigurePrimaryHttpMessageHandler(() => handler);
```

その後、を使用して、 `IHttpClientFactory` ハンドラーと証明書を使用して名前付きインスタンスを取得できます。 `CreateClient` `Startup` インスタンスを取得するには、クラスで定義されているクライアントの名前を持つメソッドを使用します。 HTTP 要求は、クライアントを使用して必要に応じて送信できます。

```csharp
private readonly IHttpClientFactory _clientFactory;
 
public ApiService(IHttpClientFactory clientFactory)
{
    _clientFactory = clientFactory;
}
 
private async Task<JsonDocument> GetApiDataWithNamedClient()
{
    var client = _clientFactory.CreateClient("namedClient");
 
    var request = new HttpRequestMessage()
    {
        RequestUri = new Uri("https://localhost:44379/api/values"),
        Method = HttpMethod.Get,
    };
    var response = await client.SendAsync(request);
    if (response.IsSuccessStatusCode)
    {
        var responseContent = await response.Content.ReadAsStringAsync();
        var data = JsonDocument.Parse(responseContent);
        return data;
    }
 
    throw new ApplicationException($"Status code: {response.StatusCode}, Error: {response.ReasonPhrase}");
}
```

正しい証明書がサーバーに送信されると、データが返されます。 証明書または間違った証明書が送信されない場合は、HTTP 403 ステータスコードが返されます。

### <a name="create-certificates-in-powershell"></a>PowerShell での証明書の作成

証明書の作成は、このフローを設定するうえで最も難しい部分です。 ルート証明書は、PowerShell コマンドレットを使用して作成でき `New-SelfSignedCertificate` ます。 証明書を作成するときは、強力なパスワードを使用します。 ここで示すように、 `KeyUsageProperty` パラメーターとパラメーターを追加することが重要です `KeyUsage` 。

#### <a name="create-root-ca"></a>ルート CA の作成

```powershell
New-SelfSignedCertificate -DnsName "root_ca_dev_damienbod.com", "root_ca_dev_damienbod.com" -CertStoreLocation "cert:\LocalMachine\My" -NotAfter (Get-Date).AddYears(20) -FriendlyName "root_ca_dev_damienbod.com" -KeyUsageProperty All -KeyUsage CertSign, CRLSign, DigitalSignature

$mypwd = ConvertTo-SecureString -String "1234" -Force -AsPlainText

Get-ChildItem -Path cert:\localMachine\my\"The thumbprint..." | Export-PfxCertificate -FilePath C:\git\root_ca_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\"The thumbprint..." -FilePath root_ca_dev_damienbod.crt
```

> [!NOTE]
> `-DnsName`パラメーター値は、アプリの配置ターゲットと一致している必要があります。 たとえば、開発用の "localhost" などです。

#### <a name="install-in-the-trusted-root"></a>信頼されたルートにインストールする

ルート証明書は、ホストシステムで信頼されている必要があります。 証明機関によって作成されていないルート証明書は、既定では信頼されません。 次のリンクを使用して、Windows でこれを実現する方法について説明します。

https://social.msdn.microsoft.com/Forums/SqlServer/5ed119ef-1704-4be4-8a4f-ef11de7c8f34/a-certificate-chain-processed-but-terminated-in-a-root-certificate-which-is-not-trusted-by-the

#### <a name="intermediate-certificate"></a>中間証明機関の証明書

ルート証明書から中間証明書を作成できるようになりました。 これはすべてのユースケースで必須ではありませんが、多くの証明書を作成したり、証明書のグループをアクティブ化または無効にしたりする必要がある場合があります。 `TextExtension`パラメーターは、証明書の基本的な制約でパスの長さを設定するために必要です。

その後、中間証明書を Windows ホストシステムの信頼された中間証明書に追加できます。

```powershell
$mypwd = ConvertTo-SecureString -String "1234" -Force -AsPlainText

$parentcert = ( Get-ChildItem -Path cert:\LocalMachine\My\"The thumbprint of the root..." )

New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "intermediate_dev_damienbod.com" -Signer $parentcert -NotAfter (Get-Date).AddYears(20) -FriendlyName "intermediate_dev_damienbod.com" -KeyUsageProperty All -KeyUsage CertSign, CRLSign, DigitalSignature -TextExtension @("2.5.29.19={text}CA=1&pathlength=1")

Get-ChildItem -Path cert:\localMachine\my\"The thumbprint..." | Export-PfxCertificate -FilePath C:\git\AspNetCoreCertificateAuth\Certs\intermediate_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\"The thumbprint..." -FilePath intermediate_dev_damienbod.crt
```

#### <a name="create-child-certificate-from-intermediate-certificate"></a>中間証明書から子証明書を作成する

子証明書は、中間証明書から作成できます。 これは end エンティティであり、さらに子証明書を作成する必要はありません。

```powershell
$parentcert = ( Get-ChildItem -Path cert:\LocalMachine\My\"The thumbprint from the Intermediate certificate..." )

New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "child_a_dev_damienbod.com" -Signer $parentcert -NotAfter (Get-Date).AddYears(20) -FriendlyName "child_a_dev_damienbod.com"

$mypwd = ConvertTo-SecureString -String "1234" -Force -AsPlainText

Get-ChildItem -Path cert:\localMachine\my\"The thumbprint..." | Export-PfxCertificate -FilePath C:\git\AspNetCoreCertificateAuth\Certs\child_a_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\"The thumbprint..." -FilePath child_a_dev_damienbod.crt
```

#### <a name="create-child-certificate-from-root-certificate"></a>ルート証明書から子証明書を作成する

子証明書は、ルート証明書から直接作成することもできます。 

```powershell
$rootcert = ( Get-ChildItem -Path cert:\LocalMachine\My\"The thumbprint from the root cert..." )

New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "child_a_dev_damienbod.com" -Signer $rootcert -NotAfter (Get-Date).AddYears(20) -FriendlyName "child_a_dev_damienbod.com"

$mypwd = ConvertTo-SecureString -String "1234" -Force -AsPlainText

Get-ChildItem -Path cert:\localMachine\my\"The thumbprint..." | Export-PfxCertificate -FilePath C:\git\AspNetCoreCertificateAuth\Certs\child_a_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\"The thumbprint..." -FilePath child_a_dev_damienbod.crt
```

#### <a name="example-root---intermediate-certificate---certificate"></a>ルート中間証明書の例-証明書

```powershell
$mypwdroot = ConvertTo-SecureString -String "1234" -Force -AsPlainText
$mypwd = ConvertTo-SecureString -String "1234" -Force -AsPlainText

New-SelfSignedCertificate -DnsName "root_ca_dev_damienbod.com", "root_ca_dev_damienbod.com" -CertStoreLocation "cert:\LocalMachine\My" -NotAfter (Get-Date).AddYears(20) -FriendlyName "root_ca_dev_damienbod.com" -KeyUsageProperty All -KeyUsage CertSign, CRLSign, DigitalSignature

Get-ChildItem -Path cert:\localMachine\my\0C89639E4E2998A93E423F919B36D4009A0F9991 | Export-PfxCertificate -FilePath C:\git\root_ca_dev_damienbod.pfx -Password $mypwdroot

Export-Certificate -Cert cert:\localMachine\my\0C89639E4E2998A93E423F919B36D4009A0F9991 -FilePath root_ca_dev_damienbod.crt

$rootcert = ( Get-ChildItem -Path cert:\LocalMachine\My\0C89639E4E2998A93E423F919B36D4009A0F9991 )

New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "child_a_dev_damienbod.com" -Signer $rootcert -NotAfter (Get-Date).AddYears(20) -FriendlyName "child_a_dev_damienbod.com" -KeyUsageProperty All -KeyUsage CertSign, CRLSign, DigitalSignature -TextExtension @("2.5.29.19={text}CA=1&pathlength=1")

Get-ChildItem -Path cert:\localMachine\my\BA9BF91ED35538A01375EFC212A2F46104B33A44 | Export-PfxCertificate -FilePath C:\git\AspNetCoreCertificateAuth\Certs\child_a_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\BA9BF91ED35538A01375EFC212A2F46104B33A44 -FilePath child_a_dev_damienbod.crt

$parentcert = ( Get-ChildItem -Path cert:\LocalMachine\My\BA9BF91ED35538A01375EFC212A2F46104B33A44 )

New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "child_b_from_a_dev_damienbod.com" -Signer $parentcert -NotAfter (Get-Date).AddYears(20) -FriendlyName "child_b_from_a_dev_damienbod.com" 

Get-ChildItem -Path cert:\localMachine\my\141594A0AE38CBBECED7AF680F7945CD51D8F28A | Export-PfxCertificate -FilePath C:\git\AspNetCoreCertificateAuth\Certs\child_b_from_a_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\141594A0AE38CBBECED7AF680F7945CD51D8F28A -FilePath child_b_from_a_dev_damienbod.crt
```

ルート証明書、中間証明書、または子証明書を使用する場合は、必要に応じて拇印または PublicKey を使用して証明書を検証できます。

```csharp
using System.Collections.Generic;
using System.IO;
using System.Security.Cryptography.X509Certificates;

namespace AspNetCoreCertificateAuthApi
{
    public class MyCertificateValidationService 
    {
        public bool ValidateCertificate(X509Certificate2 clientCertificate)
        {
            return CheckIfThumbprintIsValid(clientCertificate);
        }

        private bool CheckIfThumbprintIsValid(X509Certificate2 clientCertificate)
        {
            var listOfValidThumbprints = new List<string>
            {
                "141594A0AE38CBBECED7AF680F7945CD51D8F28A",
                "0C89639E4E2998A93E423F919B36D4009A0F9991",
                "BA9BF91ED35538A01375EFC212A2F46104B33A44"
            };

            if (listOfValidThumbprints.Contains(clientCertificate.Thumbprint))
            {
                return true;
            }

            return false;
        }
    }
}
```

<a name="occ"></a>

::: moniker range=">= aspnetcore-5.0"

## <a name="certificate-validation-caching"></a>証明書の検証キャッシュ

ASP.NET Core 5.0 以降のバージョンでは、検証結果のキャッシュを有効にする機能がサポートされています。 キャッシュは、検証がコストのかかる操作であるため、証明書の認証のパフォーマンスを大幅に向上させます。

既定では、証明書の認証によってキャッシュが無効になります。 キャッシュを有効にするには、 `AddCertificateCache` 次のようにを呼び出し `Startup.ConfigureServices` ます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(
        CertificateAuthenticationDefaults.AuthenticationScheme)
            .AddCertificate()
            .AddCertificateCache(options =>
            {
                options.CacheSize = 1024;
                options.CacheEntryExpiration = TimeSpan.FromMinutes(2);
            });
}
```

既定のキャッシュ実装は、結果をメモリに格納します。 独自のキャッシュを提供するには、を実装 `ICertificateValidationCache` し、依存関係の挿入に登録します。 たとえば、「 `services.AddSingleton<ICertificateValidationCache, YourCache>()` 」のように入力します。

::: moniker-end

## <a name="optional-client-certificates"></a>オプションのクライアント証明書

このセクションでは、証明書を使用してアプリのサブセットを保護する必要があるアプリについて説明します。 たとえば、 Razor アプリ内のページまたはコントローラーにクライアント証明書が必要な場合があります。 これにより、クライアント証明書としての問題が示されます。
  
* は TLS 機能であり、HTTP 機能ではありません。
* は接続ごとにネゴシエートされます。 HTTP データを使用できるようにするには、接続の開始時にネゴシエートする必要があります。 接続の開始時には、Server Name Indication (SNI) のみ &dagger; が認識されます。 クライアント証明書とサーバー証明書は、接続の最初の要求の前にネゴシエートされ、通常、要求は再ネゴシエートできません。

TLS 再ネゴシエーションは、オプションのクライアント証明書を実装するための従来の方法でした。 これは、次の理由で推奨されなくなりました。
- HTTP/1.1 では、POST 要求中に renegotiating を実行すると、要求本文が TCP ウィンドウをいっぱいにして再ネゴシエーションパケットを受信できないデッドロックが発生する可能性があります。
- HTTP/2 は、再ネゴシエーションを [明示的に禁止](https://tools.ietf.org/html/rfc7540#section-9.2.1) します。
- TLS 1.3 では再ネゴシエーションのサポートが [削除され](https://tools.ietf.org/html/rfc8740#section-1) ました。

ASP.NET Core 5 preview 7 以降では、オプションのクライアント証明書の便利なサポートが追加されています。 詳細については、 [オプションの証明書のサンプル](https://github.com/dotnet/aspnetcore/tree/9ce4a970a21bace3fb262da9591ed52359309592/src/Security/Authentication/Certificate/samples/Certificate.Optional.Sample)を参照してください。

次の方法では、オプションのクライアント証明書がサポートされます。

::: moniker range=">= aspnetcore-5.0"

* ドメインとサブドメインのバインドを設定します。
  * たとえば、とでバインドを設定 `contoso.com` し `myClient.contoso.com` ます。 `contoso.com`ホストはクライアント証明書を必要としませんが、そう `myClient.contoso.com` です。
  * 詳細については、次を参照してください。
    * [Kestrel](/fundamentals/servers/kestrel):
      * [ListenOptions.UseHttps](xref:fundamentals/servers/kestrel/endpoints#listenoptionsusehttps)
      * <xref:Microsoft.AspNetCore.Server.Kestrel.Https.HttpsConnectionAdapterOptions.ClientCertificateMode>
      * 注 Kestrel では、現在、1つのバインドで複数の TLS 構成がサポートされていません。一意の Ip またはポートを持つ2つのバインドが必要です。 「https://github.com/dotnet/runtime/issues/31097」を参照してください。
    * IIS
      * [IIS のホスト](xref:host-and-deploy/iis/index#create-the-iis-site)
      * [IIS でのセキュリティの構成](/iis/manage/configuring-security/how-to-set-up-ssl-on-iis#configure-ssl-settings-2)
    * HTTP.sys: [Windows Server の構成](xref:fundamentals/servers/httpsys#configure-windows-server)

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* ドメインとサブドメインのバインドを設定します。
  * たとえば、とでバインドを設定 `contoso.com` し `myClient.contoso.com` ます。 `contoso.com`ホストはクライアント証明書を必要としませんが、そう `myClient.contoso.com` です。
  * 詳細については、次を参照してください。
    * [Kestrel](/fundamentals/servers/kestrel):
      * [ListenOptions.UseHttps](xref:fundamentals/servers/kestrel#listenoptionsusehttps)
      * <xref:Microsoft.AspNetCore.Server.Kestrel.Https.HttpsConnectionAdapterOptions.ClientCertificateMode>
      * 注 Kestrel では、現在、1つのバインドで複数の TLS 構成がサポートされていません。一意の Ip またはポートを持つ2つのバインドが必要です。 「https://github.com/dotnet/runtime/issues/31097」を参照してください。
    * IIS
      * [IIS のホスト](xref:host-and-deploy/iis/index#create-the-iis-site)
      * [IIS でのセキュリティの構成](/iis/manage/configuring-security/how-to-set-up-ssl-on-iis#configure-ssl-settings-2)
    * HTTP.sys: [Windows Server の構成](xref:fundamentals/servers/httpsys#configure-windows-server)

::: moniker-end

* クライアント証明書を必要としていない web アプリへの要求の場合:
  * クライアント証明書で保護されたサブドメインを使用して同じページにリダイレクトします。
  * たとえば、にリダイレクト `myClient.contoso.com/requestedPage` します。 への要求は `myClient.contoso.com/requestedPage` とは異なるホスト名であるため、 `contoso.com/requestedPage` クライアントは別の接続を確立し、クライアント証明書を提供します。
  * 詳細については、「<xref:security/authorization/introduction>」を参照してください。

[この GitHub ディスカッション](https://github.com/dotnet/AspNetCore.Docs/issues/18720)の問題では、オプションのクライアント証明書に関する質問、コメント、その他のフィードバックを残しておきます。

&dagger; Server Name Indication (SNI) は、SSL ネゴシエーションの一部として仮想ドメインを含めるための TLS 拡張機能です。 これは実質的に、仮想ドメイン名またはホスト名を使用してネットワークエンドポイントを識別できることを意味します。
