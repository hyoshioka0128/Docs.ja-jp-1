---
title: ASP.NET Core 3.0 の新機能
author: rick-anderson
description: ASP.NET Core 3.0 の新機能について説明します。
ms.author: riande
ms.custom: mvc
ms.date: 12/05/2019
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
uid: aspnetcore-3.0
ms.openlocfilehash: fe68c54ff16751058a3eeee3880a11657344c40a
ms.sourcegitcommit: acfe51c35497a204f75c2a61125c9408c04493e6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102605673"
---
# <a name="whats-new-in-aspnet-core-30"></a>ASP.NET Core 3.0 の新機能

この記事では、ASP.NET Core 3.0 の最も大きな変更点について説明します。また、その変更点のドキュメントへのリンクも示します。

## Blazor

Blazor は、.NET を使って対話型のクライアント側 Web UI を構築するための ASP.NET Core の新しいフレームワークです。

* JavaScript の代わりに C# を使って、優れた対話型 UI を作成します。
* .NET で記述された、サーバー側とクライアント側のアプリのロジックを共有します。
* モバイル ブラウザーを含めた広範なブラウザーのサポートのために、HTML および CSS として UI をレンダリングします。

Blazor フレームワークでサポートされるシナリオ:

* 再利用可能な UI コンポーネント (Razor コンポーネント)
* クライアント側のルーティング
* コンポーネントのレイアウト
* 依存関係の挿入のサポート
* フォームと検証
* Razor クラス ライブラリを使用したコンポーネント ライブラリの構築
* JavaScript 相互運用

詳細については、「<xref:blazor/index>」を参照してください。

### Blazor Server

Blazor では、UI の更新プログラムを適用する方法からコンポーネントのレンダリング ロジックが分離されます。 Blazor Server では、ASP.NET Core アプリでサーバー上の Razor コンポーネントをホストするためのサポートが提供されます。 UI の更新は SignalR 接続を介して処理されます。 Blazor Server は ASP.NET Core 3.0 でサポートされています。

### <a name="blazor-webassembly-preview"></a>Blazor WebAssembly (プレビュー)

Blazor アプリは、WebAssembly ベースの .NET ランタイムを使用してブラウザーで直接実行することもできます。 Blazor WebAssembly はプレビュー段階であり、ASP.NET Core 3.0 ではサポートされて "*いません*"。 Blazor WebAssembly は、ASP.NET Core の今後のリリースでサポートされる予定です。

### <a name="razor-components"></a>Razor のコンポーネント

Blazor アプリはコンポーネントから構築されています。 コンポーネントは、ページ、ダイアログ、フォームなどのユーザー インターフェイス (UI) の自己完結型チャンクです。 コンポーネントは、UI レンダリング ロジックとクライアント側のイベント ハンドラーを定義する通常の .NET クラスです。 JavaScript を使用せずに、機能豊富な対話型 Web アプリを作成できます。

通常、Blazor のコンポーネントは、HTML と C# が自然に融合している Razor 構文を使用して作成されます。 Razor コンポーネントは、両方とも Razor を使用するという点で Razor Pages および MVC ビューに似ています。 要求 - 応答モデルに基づくページやビューとは異なり、コンポーネントは特に UI コンポジションを処理するために使われます。

## <a name="grpc"></a>gRPC

[gRPC](https://grpc.io/):

* 広く普及している高パフォーマンス RPC (リモート プロシージャ コール) フレームワークです。
* API 開発に対して独特のコントラクト優先のアプローチを提供します。
* 次のような最新テクノロジを使用します。

  * HTTP/2 (転送用)。
  * プロトコル バッファー (インターフェイス記述言語として)。
  * バイナリ シリアル化形式。
* 次のような機能があります。

  * 認証
  * 双方向のストリーミングおよびフロー制御。
  * キャンセルおよびタイムアウト。

ASP.NET Core 3.0 の gRPC 機能には次のものが含まれます。

* [Grpc.AspNetCore](https://www.nuget.org/packages/Grpc.AspNetCore):gRPC サービスをホストするための ASP.NET Core フレームワーク。 ASP.NET Core 上の gRPC は、ログ記録、依存関係の挿入 (DI)、認証、承認など、標準の ASP.NET Core 機能と完全に統合されています。
* [Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client):使い慣れた `HttpClient` に基づいて構築される .NET Core 用の gRPC クライアント。
* [Grpc.Net.ClientFactory](https://www.nuget.org/packages/Grpc.Net.ClientFactory): `HttpClientFactory` と gRPC クライアントの統合。

詳細については、「<xref:grpc/index>」を参照してください。

## SignalR

移行の手順については、[SignalR コードの更新](xref:migration/22-to-30#signalr)に関するページを参照してください。 現在、SignalR は `System.Text.Json` を使用して JSON メッセージのシリアル化および逆シリアル化を行います。 `Newtonsoft.Json` ベースのシリアライザーを復元する手順については、「[Newtonsoft.Json に切り替える](xref:migration/22-to-30#switch-to-newtonsoftjson)」を参照してください。

SignalR 対応の JavaScript および .NET クライアントには、自動再接続のサポートが追加されました。 既定では、クライアントはすぐに再接続を試行し、必要に応じて 2 秒後、10 秒後、30 秒後に再試行します。 クライアントが正常に再接続すると、新しい接続 ID を受け取ります。 自動再接続はオプトインです。

```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .withAutomaticReconnect()
    .build();
```

再接続の間隔は、ミリ秒単位の間隔を配列で渡すことによって指定できます。

```javascript
.withAutomaticReconnect([0, 3000, 5000, 10000, 15000, 30000])
//.withAutomaticReconnect([0, 2000, 10000, 30000]) The default intervals.
```

カスタム実装を渡すと、再接続間隔を完全に制御することができます。

最後の再接続間隔の後で再接続に失敗した場合:

* クライアントは接続がオフラインであると見なします。
* クライアントは再接続の試行を停止します。

再接続の試行中に、再接続が試行されていることをユーザーに通知するようにアプリ UI を更新します。

接続が中断されたときに UI にフィードバックを提供するため、次のイベント ハンドラーを含むように SignalR クライアント API が拡張されました。

* `onreconnecting`:これによって、開発者が UI を無効にしたり、アプリがオフラインであることをユーザーに知らせたりすることができます。
* `onreconnected`:これによって、接続が再確立したときに開発者が UI を更新できます。

次のコードでは `onreconnecting` を使用して、接続の試行中に UI を更新します。

```javascript
connection.onreconnecting((error) => {
    const status = `Connection lost due to error "${error}". Reconnecting.`;
    document.getElementById("messageInput").disabled = true;
    document.getElementById("sendButton").disabled = true;
    document.getElementById("connectionStatus").innerText = status;
});
```

次のコードでは `onreconnected` を使用して、接続時に UI を更新します。

```javascript
connection.onreconnected((connectionId) => {
    const status = `Connection reestablished. Connected.`;
    document.getElementById("messageInput").disabled = false;
    document.getElementById("sendButton").disabled = false;
    document.getElementById("connectionStatus").innerText = status;
});
```

SignalR 3.0 以降では、ハブ メソッドが承認を必要とする場合に、承認ハンドラーにカスタム リソースが提供されます。 リソースは `HubInvocationContext` のインスタンスです。 `HubInvocationContext` には次のものが含まれます。

* `HubCallerContext`
* 呼び出されているハブ メソッドの名前。
* ハブ メソッドに対する引数。

複数の組織が Azure Active Directory を使用してサインインできるチャット ルーム アプリの例について考えてみます。 Microsoft アカウントを持つユーザーは誰でもサインインしてチャットできますが、ユーザーを利用禁止にしたりユーザーのチャット履歴を表示したりできるのは所有している組織のメンバーのみです。 アプリを使用すると、特定のユーザーに対して特定の機能を制限することができます。

```csharp
public class DomainRestrictedRequirement :
    AuthorizationHandler<DomainRestrictedRequirement, HubInvocationContext>,
    IAuthorizationRequirement
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context,
        DomainRestrictedRequirement requirement,
        HubInvocationContext resource)
    {
        if (context.User?.Identity?.Name == null)
        {
            return Task.CompletedTask;
        }

        if (IsUserAllowedToDoThis(resource.HubMethodName, context.User.Identity.Name))
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }

    private bool IsUserAllowedToDoThis(string hubMethodName, string currentUsername)
    {
        if (hubMethodName.Equals("banUser", StringComparison.OrdinalIgnoreCase))
        {
            return currentUsername.Equals("bob42@jabbr.net", StringComparison.OrdinalIgnoreCase);
        }

        return currentUsername.EndsWith("@jabbr.net", StringComparison.OrdinalIgnoreCase));
    }
}
```

上記のコードでは、`DomainRestrictedRequirement` がカスタムの `IAuthorizationRequirement` として機能します。 `HubInvocationContext` リソース パラメーターが渡されているため、内部ロジックで以下を行うことができます。

* ハブが呼び出されているコンテキストを調べます。
* 個々のハブ メソッドの実行をユーザーに許可するかどうかを決定します。

個々のハブ メソッドは、コードが実行時にチェックするポリシーの名前でマークできます。 クライアントが個々のハブ メソッドを呼び出そうとすると、`DomainRestrictedRequirement` ハンドラーが実行され、メソッドへのアクセスを制御します。 この方法に基づいて、`DomainRestrictedRequirement` が次のようにアクセスを制御します。

* ログインしているすべてのユーザーが `SendMessage` メソッドを呼び出すことができます。
* `@jabbr.net` 電子メール アドレスを使用してログインしたユーザーのみが、ユーザーの履歴を表示できます。
* `bob42@jabbr.net` のみが、ユーザーをチャット ルーム利用禁止にすることができます。

```csharp
[Authorize]
public class ChatHub : Hub
{
    public void SendMessage(string message)
    {
    }

    [Authorize("DomainRestricted")]
    public void BanUser(string username)
    {
    }

    [Authorize("DomainRestricted")]
    public void ViewUserHistory(string username)
    {
    }
}
```

`DomainRestricted` ポリシーの作成には以下が含まれる場合があります。

* *Startup.cs* に新しいポリシーを追加します。
* カスタムの `DomainRestrictedRequirement` 要件をパラメーターとして指定します。
* 承認ミドルウェアを使用して `DomainRestricted` を登録します。

```csharp
services
    .AddAuthorization(options =>
    {
        options.AddPolicy("DomainRestricted", policy =>
        {
            policy.Requirements.Add(new DomainRestrictedRequirement());
        });
    });
```

SignalR ハブでは[エンドポイント ルーティング](xref:fundamentals/routing)が使用されます。 SignalR ハブ接続は以前は明示的に実行されました。

```csharp
app.UseSignalR(routes =>
{
    routes.MapHub<ChatHub>("hubs/chat");
});
```

以前のバージョンでは、開発者はさまざまな場所でコントローラー、Razor ページ、ハブを関連付ける必要がありました。 明示的な接続によって、ほぼ同一のルーティング セグメントがいくつも生成されます。

```csharp
app.UseSignalR(routes =>
{
    routes.MapHub<ChatHub>("hubs/chat");
});

app.UseRouting(routes =>
{
    routes.MapRazorPages();
});
```

SignalR 3.0 ハブは、エンドポイント ルーティングを介してルーティングできます。 エンドポイント ルーティングでは、通常、すべてのルーティングを `UseRouting` に構成できます。

```csharp
app.UseRouting(routes =>
{
    routes.MapRazorPages();
    routes.MapHub<ChatHub>("hubs/chat");
});
```

ASP.NET Core 3.0 SignalR で追加されたもの:

クライアントとサーバー間のストリーミング。 クライアントとサーバー間のストリーミングでは、サーバー側メソッドが `IAsyncEnumerable<T>` または `ChannelReader<T>` のインスタンスを受け取ることができます。 次 C# の例では、ハブの `UploadStream` メソッドがクライアントから文字列のストリームを受け取ります。

```csharp
public async Task UploadStream(IAsyncEnumerable<string> stream)
{
    await foreach (var item in stream)
    {
        // process content
    }
}
```

.NET クライアント アプリは、`IAsyncEnumerable<T>` または `ChannelReader<T>` インスタンスを、上記の `UploadStream` Hub メソッドの `stream` 引数として渡すことができます。

`for` ループが完了し、ローカル関数が終了した後で、ストリームの完了が送信されます。

```csharp
async IAsyncEnumerable<string> clientStreamData()
{
    for (var i = 0; i < 5; i++)
    {
        var data = await FetchSomeData();
        yield return data;
    }
}

await connection.SendAsync("UploadStream", clientStreamData());
```

JavaScript クライアント アプリは、上記の `UploadStream` ハブ メソッドの `stream` 引数のために SignalR `Subject` (または [RxJS Subject](https://rxjs.dev/api/index/class/Subject)) を使用します。

```javascript
let subject = new signalR.Subject();
await connection.send("StartStream", "MyAsciiArtStream", subject);
```

JavaScript コードで `subject.next` メソッドを使用すると、文字列がキャプチャされてからサーバーへの送信準備が整うように文字列を処理できます。

```javascript
subject.next("example");
subject.complete();
```

直前の 2 つのスニペットのようなコードを使用して、リアルタイム ストリーミング エクスペリエンスを作成できます。

## <a name="new-json-serialization"></a>新しい JSON シリアル化

現在、ASP.NET Core 3.0 では、JSON シリアル化のために既定で <xref:System.Text.Json> が使用されます。

* 非同期で JSON の読み取りと書き込みを行います。
* UTF-8 テキスト用に最適化されています。
* 通常、`Newtonsoft.Json` よりパフォーマンスが向上します。

Json.NET を ASP.NET Core 3.0 に追加するには、「[Newtonsoft.Json ベースの JSON 形式のサポートを追加する](xref:web-api/advanced/formatting#add-newtonsoftjson-based-json-format-support)」を参照してください。

## <a name="new-razor-directives"></a>新しい Razor ディレクティブ

次の一覧には、新しい Razor ディレクティブが含まれます。

* [`@attribute`](xref:mvc/views/razor#attribute):`@attribute` ディレクティブでは、指定された属性が生成されたページまたはビューのクラスに適用されます。 たとえば、`@attribute [Authorize]` のようにします。
* [`@implements`](xref:mvc/views/razor#implements):`@implements` ディレクティブでは、生成されたクラスのインターフェイスが実装されます。 たとえば、`@implements IDisposable` のようにします。

## <a name="identityserver4-supports-authentication-and-authorization-for-web-apis-and-spas"></a>IdentityServer4 では、Web API と SPA の認証と承認がサポートされています

ASP.NET Core 3.0 では、Web API 認証のサポートの使用により、シングルページ アプリ (SPA) での認証が提供されます。 ユーザーを認証および格納するための ASP.NET Core Identity が [IdentityServer4](https://identityserver.io/) と組み合わされ、OpenID Connect が実装されます。

IdentityServer4 は、ASP.NET Core 3.0 用の OpenID Connect および OAuth 2.0 フレームワークです。 これにより、次のセキュリティ機能が有効になります。

* サービスとしての認証 (AaaS)
* 複数のアプリケーションの種類でのシングル サインオン/オフ (SSO)
* API のアクセス制御
* Federation Gateway

詳細については、[IdentityServer4 のドキュメント](http://docs.identityserver.io/en/latest/index.html)または「[SPAs の認証と承認](xref:security/authentication/identity/spa)」を参照してください。

## <a name="certificate-and-kerberos-authentication"></a>証明書および Kerberos 認証

証明書認証には以下が必要です。

* 証明書を受け入れるようにサーバーを構成します。
* `Startup.Configure` に認証ミドルウェアを追加します。
* `Startup.ConfigureServices` に証明書認証サービスを追加します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(
        CertificateAuthenticationDefaults.AuthenticationScheme)
            .AddCertificate();
    // Other service configuration removed.
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseAuthentication();
    // Other app configuration removed.
}
```

証明書認証のオプションには次の機能が含まれます。

* 自己署名された証明書を受け入れる。
* 証明書の失効を確認する。
* 提供された証明書に適切な使用フラグが含まれていることを確認する。

既定のユーザー プリンシパルは、証明書のプロパティで構成されます。 ユーザー プリンシパルには、プリンシパルの補完または置換を可能にするイベントが含まれています。 詳細については、「<xref:security/authentication/certauth>」を参照してください。

[Windows 認証](/windows-server/security/windows-authentication/windows-authentication-overview)は、Linux および macOS に拡張されています。 以前のバージョンでは、Windows 認証は [IIS](xref:host-and-deploy/iis/index) と [HTTP.sys](xref:fundamentals/servers/httpsys) に限定されていました。 ASP.NET Core 3.0 では、[Kestrel](xref:fundamentals/servers/kestrel) は、Windows、Linux、および macOS 上で、Windows ドメインに参加しているホストに対して Negotiate、[Kerberos](/windows-server/security/kerberos/kerberos-authentication-overview)、および [NTLM](/windows-server/security/kerberos/ntlm-overview) を使用できます。 これらの認証スキームの Kestrel サポートは、[Microsoft.AspNetCore.Authentication.Negotiate NuGet](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Negotiate) パッケージによって提供されます。 他の認証サービスと同様に、認証アプリ全体を構成してからサービスを構成します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(NegotiateDefaults.AuthenticationScheme)
        .AddNegotiate();
    // Other service configuration removed.
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseAuthentication();
    // Other app configuration removed.
}
```

ホストの要件:

* Windows ホストでは、アプリをホストするユーザー アカウントに[サービス プリンシパル名](/windows/win32/ad/service-principal-names) (SPN) を追加する必要があります。
* Linux および macOS マシンは、ドメインに参加している必要があります。
  * Web プロセス用に SPN を作成する必要があります。
  * [キータブ ファイル](/archive/blogs/pie/all-you-need-to-know-about-keytab-files)をホスト マシン上で生成して構成する必要があります。

詳細については、「<xref:security/authentication/windowsauth>」を参照してください。

## <a name="template-changes"></a>テンプレートの変更

Web UI テンプレート (Razor Pages、コントローラーとビューを含む MVC) では、以下が削除されています。

* cookie 同意 UI は含まれなくなりました。 ASP.NET Core 3.0 テンプレートで生成されるアプリで cookie 同意機能を有効にするには、「<xref:security/gdpr>」を参照してください。
* スクリプトと関連する静的アセットは、CDN を使用する代わりに、ローカル ファイルとして参照されるようになりました。 詳細については、「[スクリプトおよび関連する静的アセットが、現在の環境に基づいた CDN を使用するのではなく、ローカル ファイルとして参照されるようになりました (dotnet/AspNetCore.Docs #14350)](https://github.com/dotnet/AspNetCore.Docs/issues/14350)」を参照してください。

Angular テンプレートは、Angular 8 を使用するように更新されました。

Razor クラス ライブラリ (RCL) テンプレートは Razor コンポーネント開発での既定です。 Visual Studio の新しいテンプレート オプションによって、ページとビューのテンプレート サポートが提供されます。 コマンド シェルでテンプレートから RCL を作成するときは、`--support-pages-and-views` オプション (`dotnet new razorclasslib --support-pages-and-views`) を渡します。

## <a name="generic-host"></a>汎用ホスト

ASP.NET Core 3.0 テンプレートでは <xref:fundamentals/host/generic-host> が使用されます。 以前のバージョンでは <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder> が使用されていました。 .NET Core 汎用ホスト (<xref:Microsoft.Extensions.Hosting.HostBuilder>) を使用すると、ASP.NET Core アプリと、Web 固有ではない他のサーバー シナリオの統合が強化されます。 詳細については、「[HostBuilder による WebHostBuilder の置き換え](xref:migration/22-to-30?view=aspnetcore-2.2#hostbuilder-replaces-webhostbuilder)」を参照してください。

### <a name="host-configuration"></a>ホストの構成

ASP.NET Core 3.0 リリースよりも前には、`ASPNETCORE_` というプレフィックスが付いた環境変数が Web ホストのホスト構成用に読み込まれました。 3\.0 では、`AddEnvironmentVariables` が使用され、`DOTNET_` というプレフィックスが付いた環境変数が `CreateDefaultBuilder` でのホスト構成用に読み込まれます。

### <a name="changes-to-startup-constructor-injection"></a>Startup コンストラクター挿入の変更

汎用ホストでは、`Startup` コンストラクター挿入で次の型しかサポートされません。

* <xref:Microsoft.Extensions.Hosting.IHostEnvironment>
* `IWebHostEnvironment`
* <xref:Microsoft.Extensions.Configuration.IConfiguration>

すべてのサービスを `Startup.Configure` メソッドへの引数として直接挿入することは引き続きできます。 詳細については、「[汎用ホストによる Startup コンストラクター挿入の制限 (aspnet/Announcements #353)](https://github.com/aspnet/Announcements/issues/353)」を参照してください。

## <a name="kestrel"></a>Kestrel

* Kestrel 構成は、汎用ホストへの移行に対応するように更新されました。 3\.0 では、Kestrel は `ConfigureWebHostDefaults` によって提供される Web ホスト ビルダー上で構成されます。
* 接続アダプターは Kestrel から削除され、接続ミドルウェアで置き換えられました。これは ASP.NET Core パイプラインの HTTP ミドルウェアに似ていますが下位レベルの接続用です。
* Kestrel トランスポート層は、`Connections.Abstractions` のパブリック インターフェイスとして公開されています。
* ヘッダーとトレーラーのあいまいさは、末尾のヘッダーを新しいコレクションに移動することによって解決されました。
* 同期 I/O の API (`HttpRequest.Body.Read` など) は、アプリのクラッシュにつながるスレッドの不足の一般的な原因です。 3\.0 では、`AllowSynchronousIO` は既定で無効になっています。

詳細については、「<xref:migration/22-to-30#kestrel>」を参照してください。

## <a name="http2-enabled-by-default"></a>既定で有効な HTTP/2

HTTP/2 は、Kestrel では HTTPS エンドポイントに対して既定で有効です。 IIS または HTTP.sys での HTTP/2 サポートは、オペレーティング システムでサポートされる場合に有効です。

## <a name="eventcounters-on-request"></a>要求時の EventCounter

ホスティング EventSource `Microsoft.AspNetCore.Hosting` は、受信要求に関連する次の新しい種類の <xref:System.Diagnostics.Tracing.EventCounter> を出力します。

* `requests-per-second`
* `total-requests`
* `current-requests`
* `failed-requests`

## <a name="endpoint-routing"></a>エンドポイント ルーティング

エンドポイント ルーティングによってフレームワーク (MVC など) がミドルウェアとうまく連携できるようになりますが、これが次のように強化されています。

* ミドルウェアとエンドポイントの順序を `Startup.Configure` の要求処理パイプラインで構成できます。
* エンドポイントとミドルウェアは、正常性チェックなどの他の ASP.NET Core ベースのテクノロジに対して適切に構成できます。
* エンドポイントは、ミドルウェアと MVC の両方に、CORS や承認などのポリシーを実装できます。
* フィルターおよび属性をコントローラーのメソッドに設定できます。

詳細については、「<xref:fundamentals/routing#routing-basics>」を参照してください。

## <a name="health-checks"></a>正常性チェック

正常性チェックでは、汎用ホストとのエンドポイント ルーティングを使用します。 `Startup.Configure` で、エンドポイントの URL または相対パスを使用して、エンドポイント ビルダーで `MapHealthChecks` を呼び出します。

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health");
});
```

正常性チェックのエンドポイントは以下を行うことができます。

* 1 つまたは複数の許可されたホスト/ポートを指定する。
* 承認を必要とする。
* CORS を必要とする。

詳細については、次の記事を参照してください。

* <xref:migration/22-to-30#health-checks>
* <xref:host-and-deploy/health-checks>

## <a name="pipes-on-httpcontext"></a>HttpContext のパイプ

現在、<xref:System.IO.Pipelines> API を使用して、要求本文の読み取りと応答本文の書き込みを行うことができます。 次に、 <!-- <xref:Microsoft.AspNetCore.Http.HttpRequest.BodyReader> --> `HttpRequest.BodyReader` プロパティは、要求本文を読み取るために使用できる <xref:System.IO.Pipelines.PipeReader> を提供します。 次に、 <!-- <xref:Microsoft.AspNetCore.Http.> --> `HttpResponse.BodyWriter` プロパティは、応答本文を書き込むために使用できる <xref:System.IO.Pipelines.PipeWriter> を提供します。 `HttpRequest.BodyReader` は、`HttpRequest.Body` ストリームに類似しています。 `HttpResponse.BodyWriter` は `HttpResponse.Body` ストリームに類似しています。

<!-- indirectly related, https://github.com/dotnet/docs/pull/14414 won't be published by 9/23  -->

## <a name="improved-error-reporting-in-iis"></a>IIS でのエラー報告の向上

IIS で ASP.NET Core アプリをホストする際の起動エラーで生成される診断データが豊富になりました。 これらのエラーは、該当する場合は常にスタック トレースと共に Windows イベント ログに報告されます。 さらに、すべての警告、エラー、および未処理の例外が、Windows イベント ログに記録されます。

## <a name="worker-service-and-worker-sdk"></a>ワーカー サービスとワーカー SDK

.NET Core 3.0 では、新しいワーカー サービス アプリ テンプレートが導入されました。 このテンプレートは、.NET Core で長期サービスを作成する場合の出発点として利用できます。

詳細については次を参照してください:

* [Windows サービスとしての .NET Core ワーカー](https://devblogs.microsoft.com/aspnet/net-core-workers-as-windows-services/)
* <xref:fundamentals/host/hosted-services>
* <xref:host-and-deploy/windows-service>

## <a name="forwarded-headers-middleware-improvements"></a>Forwarded Headers Middleware の機能強化

ASP.NET Core の以前のバージョンでは、Azure Linux にデプロイした場合、または IIS 以外のリバース プロキシの背後にデプロイした場合に、<xref:Microsoft.AspNetCore.Builder.HstsBuilderExtensions.UseHsts*> と <xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection*> の呼び出しで問題が発生していました。 以前のバージョンの修正方法は、「[Linux および非 IIS のリバース プロキシのスキームを転送する](xref:host-and-deploy/proxy-load-balancer#forward-the-scheme-for-linux-and-non-iis-reverse-proxies)」に記載されてます。

このシナリオは ASP.NET Core 3.0 では修正されています。 ホストによって [Forwarded Headers Middleware](xref:host-and-deploy/proxy-load-balancer#forwarded-headers-middleware-options) が有効化されるのは、`ASPNETCORE_FORWARDEDHEADERS_ENABLED` 環境変数が `true` に設定されているときです。 `ASPNETCORE_FORWARDEDHEADERS_ENABLED` はコンテナー イメージで `true` に設定されています。

## <a name="performance-improvements"></a>パフォーマンスの向上

ASP.NET Core 3.0 には、メモリ使用量を減らしてスループットを向上させる多くの機能強化が含まれています。

* スコープ サービスで組み込み依存関係挿入コンテナーを使用する場合のメモリ使用量の削減。
* ミドルウェアのシナリオやルーティングを含む、フレームワーク全体での割り当ての削減。
* WebSocket 接続のメモリ使用量の削減。
* HTTPS 接続でのメモリの削減とスループットの向上。
* 最適化された完全非同期の新 JSON シリアライザー。
* フォーム解析でのメモリ使用量の削減とスループットの向上。

## <a name="aspnet-core-30-only-runs-on-net-core-30"></a>.NET Core 3.0 のみで実行する ASP.NET Core 3.0

ASP.NET Core 3.0 以降、.NET Framework はサポートされるターゲット フレームワークではなくなりました。 .NET Framework をターゲットとするプロジェクトは、[.NET Core 2.1 LTS リリース](https://dotnet.microsoft.com/download/dotnet-core/2.1)を使用して完全にサポートされた状態で続行できます。 ほとんどの ASP.NET Core 2.1.x 関連パッケージは、.NET Core 2.1 の 3 年の LTS 期間が終わっても無期限にサポートされます。

移行の詳細については、「[.NET Framework から .NET Core にコードを移植する](/dotnet/core/porting/)」を参照してください。

## <a name="use-the-aspnet-core-shared-framework"></a>ASP.NET Core 共有フレームワークの使用

[Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)に含まれる ASP.NET Core 3.0 共有フレームワークでは、プロジェクト ファイル内の明示的な `<PackageReference />` 要素を必要としなくなりました。 プロジェクト ファイル内で `Microsoft.NET.Sdk.Web` SDK を使用すると、共有フレームワークが自動的に参照されます。

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

## <a name="assemblies-removed-from-the-aspnet-core-shared-framework"></a>ASP.NET Core 共有フレームワークから削除されたアセンブリ

ASP.NET Core 3.0 共有フレームワークから削除された重要なアセンブリは次のとおりです。

* [Newtonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json/) (Json.NET)。 Json.NET を ASP.NET Core 3.0 に追加するには、「[Newtonsoft.Json ベースの JSON 形式のサポートを追加する](xref:web-api/advanced/formatting#add-newtonsoftjson-based-json-format-support)」を参照してください。 ASP.NET Core 3.0 では JSON の読み取りと書き込みのために `System.Text.Json` が導入されています。 詳細については、このドキュメントの「[新しい JSON シリアル化](#new-json-serialization)」を参照してください。
* [Entity Framework Core](/ef/core/)

共有フレームワークから削除されたすべてのアセンブリの一覧については、「[Microsoft.AspNetCore.App 3.0 から削除されるアセンブリ](https://github.com/dotnet/AspNetCore/issues/3755)」を参照してください。 この変更の動機については、「[Microsoft.AspNetCore.App 3.0 に対する重大な変更](https://github.com/aspnet/Announcements/issues/325)」および「[ASP.NET Core 3.0 の変更点を初めて見て](https://devblogs.microsoft.com/aspnet/a-first-look-at-changes-coming-in-asp-net-core-3-0/)」を参照してください。

<!-- 
## Additional information
For the complete list of changes, see the [ASP.NET Core 3.0 Release Notes](WHERE IS THIS????).
-->
