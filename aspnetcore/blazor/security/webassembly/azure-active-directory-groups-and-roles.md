---
title: Azure Active Directory のグループとロールを使用する ASP.NET Core Blazor WebAssembly
author: guardrex
description: Azure Active Directory のグループとロールを使用するように Blazor WebAssembly を構成する方法を説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: devx-track-csharp, mvc
ms.date: 10/27/2020
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
uid: blazor/security/webassembly/aad-groups-roles
ms.openlocfilehash: 96a7dde9a5a756e40125ffda4c54fbf24fdc616a
ms.sourcegitcommit: 97243663fd46c721660e77ef652fe2190a461f81
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2021
ms.locfileid: "98058260"
---
# <a name="azure-active-directory-aad-groups-administrator-roles-and-user-defined-roles"></a>Azure Active Directory (AAD) グループ、管理者ロール、およびユーザー定義ロール

作成者: [Luke Latham](https://github.com/guardrex)、[Javier Calvarro Nelson](https://github.com/javiercn)

> [!NOTE]
> この記事は、Microsoft Identity 1.0 を使用する Blazor ASP.NET Core アプリ バージョン 3.1 に適用され、まもなく Identity 2.0 を使用する 5.0 に更新される予定です。 詳しくは、次の GitHub イシューと pull request をご覧ください。 pull request の **[Files changed]** タブには、記事の更新に関する下書きのテキストと例が含まれています。 レビューと最終更新の後、pull request はライブ ドキュメント セットにマージされます。
>
> * 問題:[Blazor AAD グループとロールを使用した WASM (dotnet/AspNetCore.Docs #17683)](https://github.com/dotnet/AspNetCore.Docs/issues/17683)
> * Pull Request:[Blazor AAD グループとロールのトピック 5.0 (dotnet/AspNetCore #20856)](https://github.com/dotnet/AspNetCore.Docs/pull/20856)

Azure Active Directory (AAD) には、ASP.NET Core Identity と組み合わせることができる承認方法がいくつか用意されています。

* ユーザー定義グループ
  * セキュリティ
  * Microsoft 365
  * 配布
* 役割
  * AAD 管理者ロール
  * ユーザー定義ロール

この記事のガイダンスは、次のトピックで説明されている Blazor WebAssembly AAD デプロイ シナリオに適用されます。

* [Microsoft アカウントによるスタンドアロン](xref:blazor/security/webassembly/standalone-with-microsoft-accounts)
* [AAD によるスタンドアロン](xref:blazor/security/webassembly/standalone-with-azure-active-directory)
* [AAD によるホスティング](xref:blazor/security/webassembly/hosted-with-azure-active-directory)

## <a name="scopes"></a>スコープ

6 つ以上の AAD 管理者ロールおよびセキュリティ グループ メンバーシップを持つアプリ ユーザーの場合は、[Microsoft Graph API](/graph/use-the-api) の呼び出しが必須です。

Graph API の呼び出しを許可するには、Azure portal で、ホストされている Blazor ソリューションのスタンドアロンまたは *`Client`* アプリに次のいずれかの [Graph API アクセス許可 (スコープ)](/graph/permissions-reference) を付与します。

* `Directory.Read.All`
* `Directory.ReadWrite.All`
* `Directory.AccessAsUser.All`

`Directory.Read.All` は、最小特権スコープであり、この記事で説明する例に使用されているスコープです。

## <a name="user-defined-groups-and-administrator-roles"></a>ユーザー定義グループと管理者ロール

`groups` メンバーシップ要求を提供するように Azure portal でアプリを構成するには、次の Azure の記事を参照してください。 ユーザー定義 AAD グループと AAD 管理者ロールにユーザーを割り当てます。

* [Azure AD セキュリティ グループを使用したロール](/azure/architecture/multitenant-identity/app-roles#roles-using-azure-ad-security-groups)
* [`groupMembershipClaims` 属性](/azure/active-directory/develop/reference-app-manifest#groupmembershipclaims-attribute)

次の例では、ユーザーが AAD の "*課金管理者*" ロールに割り当てられているものとします。

AAD によって送信される単一の `groups` 要求では、ユーザーのグループとロールが JSON 配列内のオブジェクト ID (GUID) として示されます。 アプリでは、グループとロールの JSON 配列を、アプリで[ポリシー](xref:security/authorization/policies)を作成できる個々の `group` 要求に変換する必要があります。

割り当てられている AAD 管理者ロールとユーザー定義グループの数が 5 つを超えると、`groups` 要求を送信するのではなく、`true` 値を持つ `hasgroups` 要求が AAD から送信されます。 ユーザーに割り当てられているロールとグループが 5 つを超える可能性があるアプリでは、ユーザーのロールとグループを取得するために、個別の Graph API 呼び出しを行う必要があります。 この記事で提供している実装例は、このシナリオに対処するものです。 詳細については、`groups` および `hasgroups` 要求の情報について [Microsoft ID プラットフォーム アクセス トークン: ペイロードの要求](/azure/active-directory/develop/access-tokens#payload-claims)に関する記事を参照してください。

<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> を拡張して、グループとロールの配列プロパティを含めます。 各プロパティには空の配列を割り当てます。これにより、これらのプロパティを後で `foreach` ループ内で使用する場合に `null` を確認しなくても済むようになります。

`CustomUserAccount.cs`:

```csharp
using System.Text.Json.Serialization;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class CustomUserAccount : RemoteUserAccount
{
    [JsonPropertyName("groups")]
    public string[] Groups { get; set; } = new string[] { };

    [JsonPropertyName("roles")]
    public string[] Roles { get; set; } = new string[] { };
}
```

::: moniker range=">= aspnetcore-5.0"

次の **いずれか** の方法を使用して、AAD グループとロールの要求を作成します。

* [Graph SDK を使用する](#use-the-graph-sdk)
* [名前付き `HttpClient` を使用する](#use-a-named-httpclient)

### <a name="use-the-graph-sdk"></a>Graph SDK を使用する

[`Microsoft.Graph`](https://www.nuget.org/packages/Microsoft.Graph) 用のホストされている Blazor ソリューションのスタンドアロン アプリまたは *`Client`* アプリにパッケージ参照を追加します。

<xref:blazor/security/webassembly/graph-api#graph-sdk> の記事のセクション「*Graph SDK*」にある Graph SDK ユーティリティ クラスと構成を追加します。

ホストされている Blazor ソリューション (`CustomAccountFactory.cs`) のスタンドアロン アプリまたは *`Client`* アプリに次のカスタム ユーザー アカウント ファクトリを追加します。 カスタム ユーザー ファクトリは、ロールおよびグループの要求を処理するために使用されます。 `roles` 要求の配列については、「[ユーザー定義ロール](#user-defined-roles)」セクションで説明します。 `hasgroups` 要求が存在する場合、Graph SDK を使用して、ユーザーのロールとグループを取得するために Graph API に対する承認された要求を作成します。

```csharp
using System;
using System.Net.Http;
using System.Net.Http.Json;
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication.Internal;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.Graph;

public class CustomAccountFactory
    : AccountClaimsPrincipalFactory<CustomUserAccount>
{
    private readonly ILogger<CustomAccountFactory> logger;
    private readonly IServiceProvider serviceProvider;

    public CustomAccountFactory(IAccessTokenProviderAccessor accessor, 
        IServiceProvider serviceProvider,
        ILogger<CustomAccountFactory> logger)
        : base(accessor)
    {
        this.serviceProvider = serviceProvider;
        this.logger = logger;
    }

    public async override ValueTask<ClaimsPrincipal> CreateUserAsync(
        CustomUserAccount account,
        RemoteAuthenticationUserOptions options)
    {
        var initialUser = await base.CreateUserAsync(account, options);

        if (initialUser.Identity.IsAuthenticated)
        {
            var userIdentity = (ClaimsIdentity)initialUser.Identity;

            foreach (var role in account.Roles)
            {
                userIdentity.AddClaim(new Claim("role", role));
            }

            if (userIdentity.HasClaim(c => c.Type == "hasgroups"))
            {
                IUserMemberOfCollectionWithReferencesPage groupsAndAzureRoles = 
                    null;

                try
                {
                    var graphClient = ActivatorUtilities
                        .CreateInstance<GraphServiceClient>(serviceProvider);
                    var oid = userIdentity.Claims.FirstOrDefault(x => x.Type == "oid")?
                        .Value;

                    if (!string.IsNullOrEmpty(oid))
                    {
                        groupsAndAzureRoles = await graphClient.Users[oid].MemberOf
                            .Request().GetAsync();
                    }
                }
                catch (ServiceException serviceException)
                {
                    // Optional: Log the error
                }

                if (groupsAndAzureRoles != null)
                {
                    foreach (var entry in groupsAndAzureRoles)
                    {
                        userIdentity.AddClaim(new Claim("group", entry.Id));
                    }
                }

                var claim = userIdentity.Claims.FirstOrDefault(
                    c => c.Type == "hasgroups");

                userIdentity.RemoveClaim(claim);
            }
            else
            {
                foreach (var group in account.Groups)
                {
                    userIdentity.AddClaim(new Claim("group", group));
                }
            }
        }

        return initialUser;
    }
}
```

上記のコードには、推移的なメンバーシップは含まれていません。 アプリが直接的および推移的なグループ メンバーシップの要求を必要とする場合:

* `groupsAndAzureRoles` の `IUserMemberOfCollectionWithReferencesPage` 型を `IUserTransitiveMemberOfCollectionWithReferencesPage` に変更します。
* ユーザーのグループおよびロールを要求する場合、`MemberOf` プロパティを `TransitiveMemberOf` に置き換えます。

`Program.Main` (`Program.cs`) で、カスタム ユーザー アカウント ファクトリを使用するように MSAL 認証を構成します。<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> を拡張するカスタム ユーザー アカウント クラスがアプリで使用されている場合は、そのカスタム ユーザー アカウント クラスを次のコード内の <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> と入れ替えます。

```csharp
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.Extensions.Configuration;

...

builder.Services.AddMsalAuthentication<RemoteAuthenticationState, 
    CustomUserAccount>(options =>
{
    builder.Configuration.Bind("AzureAd", 
        options.ProviderOptions.Authentication);
    options.ProviderOptions.DefaultAccessTokenScopes.Add("...");

    options.ProviderOptions.AdditionalScopesToConsent.Add(
        "https://graph.microsoft.com/Directory.Read.All");
})
.AddAccountClaimsPrincipalFactory<RemoteAuthenticationState, CustomUserAccount, 
    CustomUserFactory>();
```

### <a name="use-a-named-httpclient"></a>名前付き `HttpClient` を使用する

::: moniker-end

ホストされている Blazor ソリューションのスタンドアロン アプリまたは *`Client`* アプリで、カスタム <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> クラスを作成します。 ロールおよびグループの情報を取得する Graph API 呼び出しには、正しいスコープを使用します。

`GraphAPIAuthorizationMessageHandler.cs`:

```csharp
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class GraphAPIAuthorizationMessageHandler : AuthorizationMessageHandler
{
    public GraphAPIAuthorizationMessageHandler(IAccessTokenProvider provider,
        NavigationManager navigationManager)
        : base(provider, navigationManager)
    {
        ConfigureHandler(
            authorizedUrls: new[] { "https://graph.microsoft.com" },
            scopes: new[] { "https://graph.microsoft.com/Directory.Read.All" });
    }
}
```

`Program.Main` (`Program.cs`) では、<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> 実装サービスを追加し、さらに Graph API 要求を行うための名前付き <xref:System.Net.Http.HttpClient> を追加します。 次の例では、クライアントに `GraphAPI` という名前を付けています。

```csharp
builder.Services.AddScoped<GraphAPIAuthorizationMessageHandler>();

builder.Services.AddHttpClient("GraphAPI",
        client => client.BaseAddress = new Uri("https://graph.microsoft.com"))
    .AddHttpMessageHandler<GraphAPIAuthorizationMessageHandler>();
```

Graph API 呼び出しから Open Data Protocol (OData) ロールおよびグループを受信するように AAD ディレクトリ オブジェクト クラスを作成します。 OData は JSON 形式で届きます。<xref:System.Net.Http.Json.HttpContentJsonExtensions.ReadFromJsonAsync%2A> の呼び出しによって `DirectoryObjects` クラスのインスタンスが作成されます。

`DirectoryObjects.cs`:

```csharp
using System.Collections.Generic;
using System.Text.Json.Serialization;

public class DirectoryObjects
{
    [JsonPropertyName("@odata.context")]
    public string Context { get; set; }

    [JsonPropertyName("value")]
    public List<Value> Values { get; set; }
}

public class Value
{
    [JsonPropertyName("@odata.type")]
    public string Type { get; set; }

    [JsonPropertyName("id")]
    public string Id { get; set; }
}
```

ロールとグループの要求を処理するカスタム ユーザー ファクトリを作成します。 次の実装例では、「[ユーザー定義ロール](#user-defined-roles)」セクションで説明する `roles` 要求の配列も処理します。 `hasgroups` 要求が存在する場合は、名前付きの <xref:System.Net.Http.HttpClient> を使用して、ユーザーのロールとグループを取得するために Graph API に承認された要求を送信します。 この実装では、Microsoft Identity Platform v1.0 エンドポイント `https://graph.microsoft.com/v1.0/me/memberOf` ([API ドキュメント](/graph/api/user-list-memberof)) を使用します。

`CustomAccountFactory.cs`:

```csharp
using System.Linq;
using System.Net.Http;
using System.Net.Http.Json;
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication.Internal;
using Microsoft.Extensions.Logging;

public class CustomUserFactory
    : AccountClaimsPrincipalFactory<CustomUserAccount>
{
    private readonly ILogger<CustomUserFactory> logger;
    private readonly IHttpClientFactory clientFactory;

    public CustomUserFactory(IAccessTokenProviderAccessor accessor, 
        IHttpClientFactory clientFactory, 
        ILogger<CustomUserFactory> logger)
        : base(accessor)
    {
        this.clientFactory = clientFactory;
        this.logger = logger;
    }

    public async override ValueTask<ClaimsPrincipal> CreateUserAsync(
        CustomUserAccount account,
        RemoteAuthenticationUserOptions options)
    {
        var initialUser = await base.CreateUserAsync(account, options);

        if (initialUser.Identity.IsAuthenticated)
        {
            var userIdentity = (ClaimsIdentity)initialUser.Identity;

            foreach (var role in account.Roles)
            {
                userIdentity.AddClaim(new Claim("role", role));
            }

            if (userIdentity.HasClaim(c => c.Type == "hasgroups"))
            {
                try
                {
                    var client = clientFactory.CreateClient("GraphAPI");

                    var response = await client.GetAsync("v1.0/me/memberOf");

                    if (response.IsSuccessStatusCode)
                    {
                        var userObjects = await response.Content
                            .ReadFromJsonAsync<DirectoryObjects>();

                        foreach (var obj in userObjects?.Values)
                        {
                            userIdentity.AddClaim(new Claim("group", obj.Id));
                        }

                        var claim = userIdentity.Claims.FirstOrDefault(
                            c => c.Type == "hasgroups");

                        userIdentity.RemoveClaim(claim);
                    }
                    else
                    {
                        logger.LogError("Graph API request failure: {REASON}", 
                            response.ReasonPhrase);
                    }
                }
                catch (AccessTokenNotAvailableException exception)
                {
                    logger.LogError("Graph API access token failure: {Message}", 
                        exception.Message);
                }
            }
            else
            {
                foreach (var group in account.Groups)
                {
                    userIdentity.AddClaim(new Claim("group", group));
                }
            }
        }

        return initialUser;
    }
}
```

元の `groups` 要求は (存在する場合) フレームワークによって自動的に削除されるため、それを削除するためのコードを提供する必要はありません。

> [!NOTE]
> この例のアプローチは次のとおりです。
>
> * カスタム <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> クラスを追加して、アクセス トークンを送信要求にアタッチします。
> * セキュリティで保護された外部 Web API エンドポイントに Web API 要求を送信するために、名前付きの <xref:System.Net.Http.HttpClient> を追加します。
> * 名前付きの <xref:System.Net.Http.HttpClient> を使用して、承認された要求を送信します。
>
> このアプローチの一般的な対象範囲については、記事 <xref:blazor/security/webassembly/additional-scenarios#custom-authorizationmessagehandler-class> を参照してください。

ホストされている Blazor ソリューションのスタンドアロン アプリまたは *`Client`* アプリの `Program.Main` (`Program.cs`) に、ファクトリを登録します。 アプリの追加のスコープとして `Directory.Read.All` スコープに同意します。

```csharp
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.Extensions.Configuration;

...

builder.Services.AddMsalAuthentication<RemoteAuthenticationState, 
    CustomUserAccount>(options =>
{
    builder.Configuration.Bind("AzureAd", 
        options.ProviderOptions.Authentication);
    options.ProviderOptions.DefaultAccessTokenScopes.Add("...");

    options.ProviderOptions.AdditionalScopesToConsent.Add(
        "https://graph.microsoft.com/Directory.Read.All");
})
.AddAccountClaimsPrincipalFactory<RemoteAuthenticationState, CustomUserAccount, 
    CustomUserFactory>();
```

## <a name="authorization-configuration"></a>承認の構成

`Program.Main` で、グループごとまたはロールごとに[ポリシー](xref:security/authorization/policies)を作成します。 次の例では、AAD の "*課金管理者*" ロールに対するポリシーを作成します。

```csharp
builder.Services.AddAuthorizationCore(options =>
{
    options.AddPolicy("BillingAdministrator", policy => 
        policy.RequireClaim("group", "69ff516a-b57d-4697-a429-9de4af7b5609"));
});
```

AAD ロール オブジェクト ID の完全な一覧については、「[AAD 管理者ロールのオブジェクト ID](#aad-administrator-role-object-ids)」セクションを参照してください。

次の例のアプリでは、前述のポリシーを使用してユーザーを承認します。

[`AuthorizeView` コンポーネント](xref:blazor/security/index#authorizeview-component)ではポリシーが使用されます。

```razor
<AuthorizeView Policy="BillingAdministrator">
    <Authorized>
        <p>
            The user is in the 'Billing Administrator' AAD Administrator Role
            and can see this content.
        </p>
    </Authorized>
    <NotAuthorized>
        <p>
            The user is NOT in the 'Billing Administrator' role and sees this
            content.
        </p>
    </NotAuthorized>
</AuthorizeView>
```

コンポーネント全体へのアクセスは、[`[Authorize]` 属性ディレクティブ](xref:blazor/security/index#authorize-attribute) (<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute>) を使用するポリシーを基にして行うことができます。

```razor
@page "/"
@using Microsoft.AspNetCore.Authorization
@attribute [Authorize(Policy = "BillingAdministrator")]
```

ログインしていないユーザーは、AAD のサインイン ページにリダイレクトされ、サインイン後にコンポーネントに戻されます。

ポリシー チェックは、[手続き型ロジックを使用してコードで実行する](xref:blazor/security/index#procedural-logic)こともできます。

```razor
@page "/checkpolicy"
@using Microsoft.AspNetCore.Authorization
@inject IAuthorizationService AuthorizationService

<h1>Check Policy</h1>

<p>This component checks a policy in code.</p>

<button @onclick="CheckPolicy">Check 'BillingAdministrator' policy</button>

<p>Policy Message: @policyMessage</p>

@code {
    private string policyMessage = "Check hasn't been made yet.";

    [CascadingParameter]
    private Task<AuthenticationState> authenticationStateTask { get; set; }

    private async void CheckPolicy()
    {
        var user = (await authenticationStateTask).User;

        if ((await AuthorizationService.AuthorizeAsync(user, 
            "BillingAdministrator")).Succeeded)
        {
            policyMessage = "Yes! The 'BillingAdministrator' policy is met.";
        }
        else
        {
            policyMessage = "No! 'BillingAdministrator' policy is NOT met.";
        }
    }
}
```

## <a name="authorize-server-api-access-for-user-defined-groups-and-administrator-roles"></a>ユーザー定義グループと管理者ロールに対してサーバー API アクセスを承認する

クライアント側 WebAssembly のユーザーにページとリソースへのアクセスを承認するだけでなく、サーバー API でユーザーにセキュリティで保護された API エンドポイントへのアクセスを承認することができます。 *Server* アプリによってユーザーのアクセス トークンが検証された後、次のようになります。

* サーバー API アプリでは、そのアクセス トークンからのユーザーの変更不可能な[オブジェクト識別子要求 (`oid`)](/azure/active-directory/develop/id-tokens#payload-claims) を使用して、Graph API のアクセス トークンを取得します。
* Graph API 呼び出しでは、ユーザーの [`memberOf`](/graph/api/user-list-memberof) を呼び出すことで、ユーザーの Azure ユーザー定義セキュリティ グループと管理者ロールのメンバーシップを取得します。
* メンバーシップは `group` 要求を確立するために使用されます。
* [認証ポリシー](xref:security/authorization/policies)を使用すると、アプリ全体でサーバー API エンドポイントへのユーザー アクセスを制限できます。

> [!NOTE]
> 現在、このガイドには、ユーザーの [AAD ユーザー定義ロール](#user-defined-roles)に基づく承認は含まれていません。

このセクションのガイダンスでは、サーバー API アプリを Microsoft Graph API 呼び出しの [*デーモン アプリ*](/azure/active-directory/develop/scenario-daemon-overview)として構成します。 この方法では、次のことを行い **ません**。

* `access_as_user` スコープを要求する。
* API 要求を行うユーザーまたはクライアントに代わって Graph API にアクセスする。

サーバー API アプリによる Graph API の呼び出しには、Azure portal 内の `Directory.Read.All` のサーバー API アプリ **アプリケーション** Graph API スコープのみが必要です。 この方法により、クライアント アプリは、サーバー API によって明示的に許可されていないディレクトリ データに完全にアクセスできなくなります。 クライアント アプリは、サーバー API アプリのコントローラー エンドポイントにのみアクセスできます。

### <a name="azure-configuration"></a>Azure の構成

* *Server* アプリの登録に、`Directory.Read.All` の (**委任** ではなく) **アプリケーション** Graph API スコープが付与されていることを確認します。これは、セキュリティ グループの最小特権アクセス レベルです。 スコープの割り当てを行った後、管理者の同意がスコープに適用されていることを確認します。
* 新しいクライアント シークレットを *Server* アプリに割り当てます。 「[アプリケーション設定](#app-settings)」セクションでのアプリの構成のために、シークレットをメモしておきます。

### <a name="app-settings"></a>アプリケーション設定

アプリ設定ファイル (`appsettings.json` または `appsettings.Production.json`) に、Azure portal からの *Server* アプリのクライアント シークレットを持つ `ClientSecret` エントリを作成します。

```json
"AzureAd": {
  "Instance": "https://login.microsoftonline.com/",
  "Domain": "XXXXXXXXXXXX.onmicrosoft.com",
  "TenantId": "{GUID}",
  "ClientId": "{GUID}",
  "ClientSecret": "{CLIENT SECRET}"
},
```

次に例を示します。

```json
"AzureAd": {
  "Instance": "https://login.microsoftonline.com/",
  "Domain": "contoso.onmicrosoft.com",
  "TenantId": "34bf0ec1-7aeb-4b5d-ba42-82b059b3abe8",
  "ClientId": "05d198e0-38c6-4efc-a67c-8ee87ed9bd3d",
  "ClientSecret": "54uE~9a.-wW91fe8cRR25ag~-I5gEq_92~"
},
```

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> テナント パブリッシャー ドメインが検証されていない場合、ユーザーまたはクライアント アクセスのサーバー API スコープでは、`https://` ベースの URI が使用されます。 このシナリオでは、サーバー API アプリで、`appsettings.json` ファイル内に `Audience` 構成が必要です。 次の構成では、`Audience` の末尾に、既定のスコープ `/{DEFAULT SCOPE}` が含まれて **いません**。ここで、プレースホルダー `{DEFAULT SCOPE}` は既定のスコープです。
>
> ```json
> {
>   "AzureAd": {
>     ...
>
>     "Audience": "https://{TENANT}.onmicrosoft.com/{SERVER API APP CLIENT ID OR CUSTOM VALUE}"
>   }
> }
>
> In the preceding configuration, the placeholder `{TENANT}` is the app's tenant, and the placeholder `{SERVER API APP CLIENT ID OR CUSTOM VALUE}` is the server API app's `ClientId` or custom value provided in the Azure portal's app registration.
>
> Example:
>
> ```json
> {
>   "AzureAd": {
>     ...
>
>     "Audience": "https://contoso.onmicrosoft.com/41451fa7-82d9-4673-8fa5-69eff5a761fd"
>   }
> }
> ```
>
> 上記の構成例では、次のようになっています。
>
> * テナント ドメインは、`contoso.onmicrosoft.com` です。
> * サーバー API アプリ `ClientId` は、`41451fa7-82d9-4673-8fa5-69eff5a761fd` です。
>
> > [!NOTE]
> > 通常、`api://` ベースの API スコープを持つ検証済みのパブリッシャー ドメインを持つアプリでは、通常、`Audience` を明示的に構成する必要は **ありません**。
>
> 詳細については、「<xref:blazor/security/webassembly/hosted-with-azure-active-directory#app-settings>」を参照してください。

::: moniker-end

### <a name="authorization-policies"></a>承認ポリシー

グループ オブジェクト ID と [AAD 管理者ロール オブジェクト ID](#aad-administrator-role-object-ids) に基づいて、*Server* アプリの `Startup.ConfigureServices` (`Startup.cs`) で、AAD セキュリティ グループと AAD 管理者ロールに対する [承認ポリシー](xref:security/authorization/policies)を作成します。

たとえば、Azure の課金管理者ロール ポリシーには、次の構成が含まれます。

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy("BillingAdmin", policy => 
        policy.RequireClaim("group", "69ff516a-b57d-4697-a429-9de4af7b5609"));
});
```

詳細については、<xref:security/authorization/policies> を参照してください。

### <a name="controller-access"></a>コントローラーのアクセス

*Server* アプリのコントローラーに対するポリシーが必要です。

次の例では、`BillingDataController` からの課金データへのアクセスを、「[承認ポリシー](#authorization-policies)」セクションで構成したように、`BillingAdmin` というポリシー名を持つ Azure 課金管理者に制限しています。

```csharp
[Authorize(Policy = "BillingAdmin")]
[ApiController]
[Route("[controller]")]
public class BillingDataController : ControllerBase
{
    ...
}
```

::: moniker range=">= aspnetcore-5.0"

### <a name="packages"></a>パッケージ

次のパッケージの *Server* アプリにパッケージ参照を追加します。

* [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph)
* [Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client)

### <a name="services"></a>サービス

*Server* アプリの `Startup.ConfigureServices` メソッドでは、*Server* アプリの `Startup` クラスのコードに追加の名前空間が必要です。 次の名前空間を `Startup.cs` に追加します。

```csharp
using System;
using System.IdentityModel.Tokens.Jwt;
using System.Net.Http.Headers;
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.Graph;
using Microsoft.Identity.Client;
using Microsoft.IdentityModel.Logging;
```

<xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerEvents> を構成する場合:

* 必要に応じて <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerEvents.OnAuthenticationFailed?displayProperty=nameWithType> の処理を含めます。 たとえば、アプリでは失敗した認証をログに記録できます。
* <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerEvents.OnTokenValidated?displayProperty=nameWithType> で、Graph API 呼び出しを行ってユーザーのグループとロールを取得します。

> [!WARNING]
> <xref:Microsoft.IdentityModel.Logging.IdentityModelEventSource.ShowPII?displayProperty=nameWithType> によって、メッセージのログ記録に個人を特定できる情報 (PII) が提供されます。 PII は、テスト ユーザー アカウントを使用したデバッグに対してのみアクティブにしてください。

`Startup.ConfigureServices`:

```csharp
JwtSecurityTokenHandler.DefaultMapInboundClaims = false;

#if DEBUG
IdentityModelEventSource.ShowPII = true;
#endif

var scopes = new string[] { "https://graph.microsoft.com/.default" };

var app = ConfidentialClientApplicationBuilder.Create(Configuration["AzureAd:ClientId"])
   .WithClientSecret(Configuration["AzureAd:ClientSecret"])
   .WithAuthority(new Uri(Configuration["AzureAd:Instance"] + Configuration["AzureAd:Domain"]))
   .Build();

services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(options =>
{
    Configuration.Bind("AzureAd", options);

    options.Events = new JwtBearerEvents()
    {
        OnTokenValidated = async context =>
        {
            var accessToken = context.SecurityToken as JwtSecurityToken;

            var oid = accessToken.Claims.FirstOrDefault(x => x.Type == "oid")?
                .Value;

            if (!string.IsNullOrEmpty(oid))
            {
                var userIdentity = (ClaimsIdentity)context.Principal.Identity;

                AuthenticationResult authResult = null;

                try
                {
                    authResult = await app.AcquireTokenForClient(scopes)
                        .ExecuteAsync();
                }
                catch (MsalUiRequiredException ex)
                {
                    // Optional: Log the exception
                }
                catch (MsalServiceException ex)
                {
                    // Optional: Log the exception
                }

                var graphClient = new GraphServiceClient(
                    new DelegateAuthenticationProvider(async requestMessage => {
                        requestMessage.Headers.Authorization =
                            new AuthenticationHeaderValue("Bearer", authResult.AccessToken);

                        await Task.CompletedTask;
                    }));

                IUserMemberOfCollectionWithReferencesPage groupsAndAzureRoles = 
                    null;

                try
                {
                    groupsAndAzureRoles = await graphClient.Users[oid].MemberOf.Request()
                        .GetAsync();
                }
                catch (ServiceException serviceException)
                {
                    // Optional: Log the exception
                }

                if (groupsAndAzureRoles != null)
                {
                    foreach (var entry in groupsAndAzureRoles)
                    {
                        userIdentity.AddClaim(new Claim("group", entry.Id));
                    }
                }
            }

            await Task.FromResult(0);
        }
    };
}, 
options =>
{
    Configuration.Bind("AzureAd", options);
});
```

前のコードでは、次のトークン エラーの処理を省略できます。

* `MsalUiRequiredException`: アプリには、十分なアクセス許可 (スコープ) がありません。
  * Azure portal 内のサーバー API アプリ スコープに、`Directory.Read.All` の **アプリケーション** アクセス許可が含まれているかどうかを確認します。
  * テナント管理者にアプリへのアクセス許可が付与されていることを確認します。
* `MsalServiceException` (`AADSTS70011`):スコープが `https://graph.microsoft.com/.default` であることを確認します。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

### <a name="packages"></a>パッケージ

次のパッケージの *Server* アプリにパッケージ参照を追加します。

* [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph)
* [Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages?q=Microsoft.IdentityModel.Clients.ActiveDirectory)

### <a name="service-configuration"></a>サービス構成

*Server* アプリの `Startup.ConfigureServices` メソッドに、Graph API 呼び出しを行い、ユーザーのセキュリティ グループとロールに対するユーザー `group` 要求を確立するためのロジックを追加します。

> [!NOTE]
> このセクションのコード例では、Microsoft Identity Platform v1.0 に基づく Active Directory 認証ライブラリ (ADAL) を使用しています。

*Server* アプリの `Startup` クラスのコードには、追加の名前空間が必要です。 次の一連の `using` ステートメントには、このセクションで後述するコードに必要な名前空間が含まれています。

```csharp
using System;
using System.IdentityModel.Tokens.Jwt;
using System.Linq;
using System.Net.Http.Headers;
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.AzureAD.UI;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Graph;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using Microsoft.IdentityModel.Logging;
```

<xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerEvents> を構成する場合:

* 必要に応じて <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerEvents.OnAuthenticationFailed?displayProperty=nameWithType> の処理を含めます。 たとえば、アプリでは失敗した認証をログに記録できます。
* <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerEvents.OnTokenValidated?displayProperty=nameWithType> で、Graph API 呼び出しを行ってユーザーのグループとロールを取得します。

> [!WARNING]
> <xref:Microsoft.IdentityModel.Logging.IdentityModelEventSource.ShowPII?displayProperty=nameWithType> によって、メッセージのログ記録に個人を特定できる情報 (PII) が提供されます。 PII は、テスト ユーザー アカウントを使用したデバッグに対してのみアクティブにしてください。

`Startup.ConfigureServices`:

```csharp
#if DEBUG
IdentityModelEventSource.ShowPII = true;
#endif

services.AddAuthentication(AzureADDefaults.BearerAuthenticationScheme)
    .AddAzureADBearer(options => Configuration.Bind("AzureAd", options));

services.Configure<JwtBearerOptions>(AzureADDefaults.JwtBearerAuthenticationScheme, 
    options =>
{
    options.Events = new JwtBearerEvents()
    {
        OnTokenValidated = async context =>
        {
            var accessToken = context.SecurityToken as JwtSecurityToken;
            var oid = accessToken.Claims.FirstOrDefault(x => x.Type == "oid")?
                .Value;

            if (!string.IsNullOrEmpty(oid))
            {
                var authContext = new AuthenticationContext(
                    Configuration["AzureAd:Instance"] +
                    Configuration["AzureAd:TenantId"]);
                AuthenticationResult authResult = null;

                try
                {
                    authResult = await authContext.AcquireTokenSilentAsync(
                        "https://graph.microsoft.com", 
                        Configuration["AzureAd:ClientId"]);
                }
                catch (AdalException adalException)
                {
                    if (adalException.ErrorCode == 
                        AdalError.FailedToAcquireTokenSilently || 
                        adalException.ErrorCode == 
                        AdalError.UserInteractionRequired)
                    {
                        var userAssertion = new UserAssertion(accessToken.RawData,
                            "urn:ietf:params:oauth:grant-type:jwt-bearer", oid);
                        var clientCredential = new ClientCredential(
                            Configuration["AzureAd:ClientId"],
                            Configuration["AzureAd:ClientSecret"]);
                        authResult = await authContext.AcquireTokenAsync(
                            "https://graph.microsoft.com", clientCredential, 
                            userAssertion);
                    }
                }

                var graphClient = new GraphServiceClient(
                    new DelegateAuthenticationProvider(async requestMessage => {
                        requestMessage.Headers.Authorization =
                            new AuthenticationHeaderValue("Bearer", 
                                authResult.AccessToken);

                        await Task.CompletedTask;
                    }));

                var userIdentity = (ClaimsIdentity)context.Principal.Identity;

                IUserMemberOfCollectionWithReferencesPage groupsAndAzureRoles = 
                    null;

                try
                {
                    groupsAndAzureRoles = await graphClient.Users[oid].MemberOf
                        .Request().GetAsync();
                }
                catch (ServiceException serviceException)
                {
                    // Optional: Log the error
                }

                if (groupsAndAzureRoles != null)
                {
                    foreach (var entry in groupsAndAzureRoles)
                    {
                        userIdentity.AddClaim(new Claim("group", entry.Id));
                    }
                }
            }

            await Task.FromResult(0);
        }
    };
});
```

前の例の場合:

* アクセス トークンが既に ADAL トークン キャッシュに格納されている可能性があるため、最初にサイレント トークンの取得 (<xref:Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext.AcquireTokenSilentAsync%2A>) が試行されます。 キャッシュからトークンを取得する方が、新しいトークンを要求するよりも高速です。
* アクセス トークンがキャッシュから取得されない場合 (<xref:Microsoft.IdentityModel.Clients.ActiveDirectory.AdalError.FailedToAcquireTokenSilently?displayProperty=nameWithType> または <xref:Microsoft.IdentityModel.Clients.ActiveDirectory.AdalError.UserInteractionRequired?displayProperty=nameWithType> がスローされた場合) は、クライアント資格情報 (<xref:Microsoft.IdentityModel.Clients.ActiveDirectory.ClientCredential>) を使用してユーザー アサーション (<xref:Microsoft.IdentityModel.Clients.ActiveDirectory.UserAssertion>) を作成し、ユーザーの代わりにトークンを取得します (<xref:Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext.AcquireTokenAsync%2A>)。 次に、`Microsoft.Graph.GraphServiceClient` により、トークンを使用して Graph API 呼び出しを行うことができます。 トークンは ADAL トークン キャッシュに格納されます。 その後同じユーザーに対して Graph API を呼び出す場合、トークンは <xref:Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext.AcquireTokenSilentAsync%2A> を使用してキャッシュからサイレントに取得されます。

::: moniker-end

<xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerEvents.OnTokenValidated> のコードでは、推移的なメンバーシップが取得されません。 直接的および推移的なメンバーシップを取得するようにコードを変更するには:

* 次のコード行に対して:

  ```csharp
  IUserMemberOfCollectionWithReferencesPage groupsAndAzureRoles = null;
  ```

  前の行を次のコードに置き換えます。

  ```csharp
  IUserTransitiveMemberOfCollectionWithReferencesPage groupsAndAzureRoles = null;
  ```

* 次のコード行に対して:

  ```csharp
  groupsAndAzureRoles = await graphClient.Users[oid].MemberOf.Request().GetAsync();
  ```

  前の行を次のコードに置き換えます。

  ```csharp
  groupsAndAzureRoles = await graphClient.Users[oid].TransitiveMemberOf.Request()
      .GetAsync();
  ```

<xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerEvents.OnTokenValidated> のコードでは、要求の作成時に AAD セキュリティ グループと AAD 管理者ロールが区別されません。 アプリでグループとロールを区別するには、グループとロールを反復処理するときに `entry.ODataType` をチェックします。 セキュリティ グループとロールの要求を別々に作成するには、次のようなコードを使用します。

```csharp
foreach (var entry in groupsAndAzureRoles)
{
    if (entry.ODataType == "#microsoft.graph.group")
    {
        userIdentity.AddClaim(new Claim("group", entry.Id));
    }
    else
    {
        // entry.ODataType == "#microsoft.graph.directoryRole"
        userIdentity.AddClaim(new Claim("role", entry.Id));
    }
}
```

## <a name="user-defined-roles"></a>ユーザー定義ロール

ユーザー定義ロールを使用するように、AAD 登録済みアプリを構成することもできます。

`roles` メンバーシップ要求を提供するように Azure portal でアプリを構成するには、「[方法: アプリケーションにアプリ ロールを追加してトークンで受け取る](/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps)」を Azure ドキュメントで参照してください。

以下の例では、次の 2 つのロールがアプリに構成されているものとします。

* `admin`
* `developer`

> [!NOTE]
> Azure AD Premium アカウントがないとセキュリティ グループにロールを割り当てることはできませんが、ユーザーをロールに割り当てて、Standard Azure アカウントを使用するユーザーに対する `roles` 要求を受け取ることができます。 このセクションのガイダンスでは、Azure AD Premium アカウントは必要ありません。
>
> Azure portal で各追加ロール割り当てに対して **_ユーザーを再追加する_** ことにより、複数のロールを割り当てます。

AAD によって送信される単一の `roles` 要求では、ユーザー定義のロールは JSON 配列内の `appRoles` の `value` として示されます。 アプリでは、ロールの JSON 配列を個々の `role` 要求に変換する必要があります。

[ユーザー定義グループと AAD 管理者ロール](#user-defined-groups-and-administrator-roles)に関するセクションで示されている `CustomUserFactory` は、JSON 配列値を持つ `roles` 要求に対して動作するように設定されます。 [ユーザー定義グループと AAD 管理者ロール](#user-defined-groups-and-administrator-roles)に関するセクションで示されているように、ホストされている Blazor ソリューションのスタンドアロン アプリまたは *`Client`* アプリに `CustomUserFactory` を追加して登録します。 元の `roles` 要求はフレームワークによって自動的に削除されるため、それを削除するためのコードを提供する必要はありません。

ホストされている Blazor ソリューションのスタンドアロン アプリまたは *`Client`* アプリの `Program.Main` で、ロール要求として "`role`" という名前の要求を指定します。

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...

    options.UserOptions.RoleClaim = "role";
});
```

この時点でコンポーネントの承認方法が機能しています。 コンポーネント内のすべての承認メカニズムで、`admin` ロールを使用してユーザーを承認できます。

* [`AuthorizeView` コンポーネント](xref:blazor/security/index#authorizeview-component) (例: `<AuthorizeView Roles="admin">`)
* [`[Authorize]` 属性ディレクティブ](xref:blazor/security/index#authorize-attribute) (<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute>) (例: `@attribute [Authorize(Roles = "admin")]`)
* [手続き型ロジック](xref:blazor/security/index#procedural-logic) (例: `if (user.IsInRole("admin")) { ... }`)

  複数のロール テストがサポートされています。

  ```csharp
  if (user.IsInRole("admin") && user.IsInRole("developer"))
  {
      ...
  }
  ```

## <a name="aad-administrator-role-object-ids"></a>AAD 管理者ロールのオブジェクト ID

次の表で示されているオブジェクト ID は、`group` 要求に対する[ポリシー](xref:security/authorization/policies)を作成するために使用されます。 ポリシーでは、アプリ内のさまざまなアクティビティについてユーザーを承認することをアプリに許可します。 詳細については、[ユーザー定義グループと AAD 管理者ロール](#user-defined-groups-and-administrator-roles)に関するセクションを参照してください。

AAD 管理者ロール | Object ID
--- | ---
アプリケーション管理者 | fa11557b-4f15-4ddd-85d5-313c7cd74047
アプリケーション開発者 | 68adcbb8-9504-44f6-89f2-5cd48dc74a2c
認証管理者 | 02d110a1-96b1-419e-af87-746461b60ed7
Azure DevOps 管理者 | a5311ace-ca41-44cd-b833-8d22caa0b34f
Azure Information Protection 管理者 | 18632dce-f9b5-4f01-abb5-37051f06860e
B2C IEF キーセット管理者 | 0c2e87e5-94f9-4adb-ae8c-bcafe11bd368
B2C IEF ポリシー管理者 | bfcab36c-10c6-4b13-b63c-4d8b62c0c44e
B2C ユーザー フロー管理者 | baa531b7-8cf0-44ad-8f98-eded88dae827
B2C ユーザー フロー属性管理者 | dd0baca0-a535-48c1-b871-8431abe16452
課金管理者 | 69ff516a-b57d-4697-a429-9de4af7b5609
クラウド アプリケーション管理者 | 250b5fe3-b553-458d-9a53-b782c13c34bf
クラウド デバイス管理者 | 26cd4b44-2636-4ddb-bdfa-27feae66f86d
コンプライアンス管理者 | 9d6e1dd0-c9f8-45f8-b558-b134f700116c
コンプライアンス データ管理者 | 4c0ca3a2-231e-416c-9411-4abe57d5cb9d
条件付きアクセス管理者 | 8f71a611-137d-49af-87ad-e97f1fd5da76
カスタマー ロックボックスのアクセス承認者 | c18d54a8-b13e-4954-a1a4-7deaf2e4f184
デスクトップ Analytics 管理者 | c62c4ac5-e4c6-4096-8a2f-1ee3cbaaae15
ディレクトリ閲覧者 | e1fc84a6-7762-4b9b-8e29-518b4adbc23b
Dynamics 365 管理者 | f20a9cfa-9fdf-49a8-a977-1afe446a1d6e
Exchange 管理者 | b2ec2cc0-d5c9-4864-ad9b-38dd9dba2652
外部 IdentityID プロバイダー管理者 | febfaeb4-e478-407a-b4b3-f4d9716618a2
全体管理者 | a45ba61b-44db-462c-924b-3b2719152588
グローバル閲覧者 | f6903b21-6aba-4124-b44c-76671796b9d5
グループ管理者 | 158b3e5a-d89d-460b-92b5-3b34985f0197
ゲスト招待元 | 4c730a1d-cc22-44af-8f9f-4eec635c7502
ヘルプデスク管理者 | 108678c8-6628-44e1-8d01-caf598a6a5f5
Intune 管理者 | 79950741-23fa-4189-b2cb-46640601c497
Kaizala 管理者 | d6322af2-48e7-42e0-8c68-0bbe31af3412
ライセンス管理者 | 3355458a-e423-44bf-8b98-4ac5e572cea5
メッセージ センターのプライバシー閲覧者 | 6395db95-9fb8-42b9-b1ed-30a2405eee6f
メッセージ センター閲覧者 | fd5d37b8-4e24-434b-9e63-70ed3b759a16
Office アプリ管理者 | 5f3870cd-b042-4f93-86d7-c9d77c664dc7
パスワード管理者 | 466e48b7-5d66-4ae5-8911-1a118de74941
Power BI 管理者 | 984e83b8-8337-4255-91a1-acb663175ab4
Power Platform 管理者 | 76d6f95e-9a15-4d7d-8d21-00de00faf9fd
特権認証管理者 | 0829f731-b46d-419f-9742-aeb122367d11
特権ロール管理者 | f20a725a-d1c8-4107-83ea-1171c97d00c7
レポート閲覧者 | 54635450-e8ed-4f2d-9632-07db2517b4de
Search 管理者 | c770a2f1-c9ba-4e60-9176-9f52b1eb1a31
Search エディター | 6a6858c6-5f0d-44ac-87c7-0190fbedd271
セキュリティ管理者 | 20fa50e3-6531-44d8-bd39-b251420568ad
セキュリティ オペレーター | 43aae017-8e51-4188-91ab-e6debd572800
セキュリティ閲覧者 | 45035cd3-fd97-4250-8197-3a53d3562d9b
サービス サポート管理者 | 2c92cf45-c914-48f8-9bf9-fc14b28818ab
SharePoint 管理者 | e1c32229-875e-461d-ae24-3cb99116e86c
Skype for Business 管理者 | 0a8cee12-e21d-43ef-abd9-f1ea85710e30
Teams 通信管理者 | 2393e455-6e13-4743-9f52-63fcec2b6a9c
Teams 通信サポート エンジニア | 802dd94e-d717-46f6-af98-b9167071e9fc
Teams 通信スペシャリスト | ef547281-cf46-4cc6-bcaa-f5eac3f030c9
Teams サービス管理者 | 8846a0be-197b-443a-b13c-11192691fa24
ユーザー管理者 | 1f6eed58-7dd3-460b-a298-666f975427a1

## <a name="additional-resources"></a>その他の技術情報

* <xref:security/authorization/claims>
* <xref:blazor/security/index>
