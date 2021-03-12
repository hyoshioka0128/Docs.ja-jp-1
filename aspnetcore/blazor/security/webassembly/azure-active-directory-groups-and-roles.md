---
title: Azure Active Directory のグループとロールを使用する ASP.NET Core Blazor WebAssembly
author: guardrex
description: Azure Active Directory のグループとロールを使用するように Blazor WebAssembly を構成する方法を説明します。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: devx-track-csharp, mvc
ms.date: 01/24/2021
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
ms.openlocfilehash: b725a60a310be23f7ceb626d4c543d0df6fadf62
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/06/2021
ms.locfileid: "102394760"
---
# <a name="azure-active-directory-aad-groups-administrator-roles-and-app-roles"></a>Azure Active Directory (AAD) グループ、管理者ロール、およびアプリ ロール

Azure Active Directory (AAD) には、ASP.NET Core Identity と組み合わせることができる承認方法がいくつか用意されています。

* グループ
  * セキュリティ
  * Microsoft 365
  * 配布
* 役割
  * AAD 管理者ロール
  * アプリ ロール

この記事のガイダンスは、次のトピックで説明されている Blazor WebAssembly AAD デプロイ シナリオに適用されます。

* [Microsoft アカウントによるスタンドアロン](xref:blazor/security/webassembly/standalone-with-microsoft-accounts)
* [AAD によるスタンドアロン](xref:blazor/security/webassembly/standalone-with-azure-active-directory)
* [AAD によるホスティング](xref:blazor/security/webassembly/hosted-with-azure-active-directory)

この記事のガイダンスでは、クライアントとサーバーのアプリについて説明します。

* **クライアント**:スタンドアロン Blazor WebAssembly アプリ、またはホストされた Blazor ソリューションの **`Client`** アプリ。
* **サーバー**:スタンドアロン ASP.NET Core サーバー API または Web API アプリ、またはホストされた Blazor ソリューションの **`Server`** アプリ。

## <a name="scopes"></a>スコープ

ユーザー プロファイル、ロールの割り当て、およびグループ メンバーシップ データに対する [Microsoft Graph API](/graph/use-the-api) 呼び出しを許可するために、**クライアント** は Azure portal で (`https://graph.microsoft.com/User.Read`) [Graph API アクセス許可 (スコープ)](/graph/permissions-reference) を使用して構成されます。

ロールおよびグループ メンバーシップ データに対して Graph API を呼び出す **サーバー** アプリには、Azure portal で `GroupMember.Read.All` (`https://graph.microsoft.com/GroupMember.Read.All`) [Graph API アクセス許可 (スコープ)](/graph/permissions-reference) が付与されます。

この記事の最初のセクションに記載したトピックで説明されている AAD デプロイ シナリオに必要なスコープに加えて、これらのスコープが必要になります。

> [!NOTE]
> "アクセス許可" と "スコープ" という語は、Azure portal や、Microsoft および外部のさまざまなドキュメントのセットで同じように使用されています。 この記事では、Azure portal でアプリに割り当てられたアクセス許可に対して、"スコープ" という語を使用します。

## <a name="group-membership-claims-attribute"></a>グループ メンバーシップ要求の属性

**クライアント** および **サーバー** アプリに対する Azure portal のアプリ マニフェストで、[`groupMembershipClaims` 属性](/azure/active-directory/develop/reference-app-manifest#groupmembershipclaims-attribute)を `All` に設定します。 値を `All` にすると、サインインしているユーザーが属しているすべてのセキュリティ グループ、配布グループ、およびロールが取得されます。

1. アプリの Azure portal の登録を開きます。
1. サイド バーで **[管理]**  >  **[マニフェスト]** を選択します。
1. `groupMembershipClaims` 属性を見つけます。
1. この値を `All` に設定します。
1. **[保存]** を選択します。

```json
"groupMembershipClaims": "All",
```

## <a name="custom-user-account"></a>カスタム ユーザー アカウント

Azure portal で AAD セキュリティ グループと AAD 管理者ロールにユーザーを割り当てます。

この記事の例では:

* サーバー API データへのアクセスを認可するために、ユーザーが Azure portal AAD テナントの AAD "*課金管理者*" ロールに割り当てられているとします。
* [認可ポリシー](xref:security/authorization/policies)を使用して、**クライアント** および **サーバー** アプリ内でアクセスを制御します。

**クライアント** アプリで、<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> を拡張して次のプロパティを含めます。

* `Roles`: AAD アプリ ロールの配列 (「[アプリ ロール](#app-roles)」セクションで説明します)
* `Wids`: [既知の ID 要求 (`wids`)](/azure/active-directory/develop/access-tokens#payload-claims) の AAD 管理者ロール
* `Oid`: 変更不可能な[オブジェクト識別子要求 (`oid`)](/azure/active-directory/develop/id-tokens#payload-claims) (テナント内およびテナント間でユーザーを一意に識別します)

各配列プロパティには空の配列を代入します。これにより、これらのプロパティを `foreach` ループ内で使用するときに `null` を確認しなくても済むようになります。

`CustomUserAccount.cs`:

```csharp
using System;
using System.Text.Json.Serialization;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class CustomUserAccount : RemoteUserAccount
{
    [JsonPropertyName("roles")]
    public string[] Roles { get; set; } = Array.Empty<string>();

    [JsonPropertyName("wids")]
    public string[] Wids { get; set; } = Array.Empty<string>();

    [JsonPropertyName("oid")]
    public string Oid { get; set; }
}
```

**クライアント** アプリのプロジェクト ファイルに [`Microsoft.Graph`](https://www.nuget.org/packages/Microsoft.Graph) のパッケージ参照を追加します。

<xref:blazor/security/webassembly/graph-api#graph-sdk> の記事のセクション「*Graph SDK*」にある Graph SDK ユーティリティ クラスと構成を追加します。 `GraphClientExtensions` クラスの `AuthenticateRequestAsync` メソッドで、アクセス トークンの `User.Read` スコープを指定します。

```csharp
var result = await TokenProvider.RequestAccessToken(
    new AccessTokenRequestOptions()
    {
        Scopes = new[] { "https://graph.microsoft.com/User.Read" }
    });
```

次のカスタム ユーザー アカウント ファクトリを **クライアント** アプリに追加します。 カスタム ユーザー ファクトリは、以下を確立するために使用されます。

* アプリ ロールの要求 (`appRole`) (「[アプリ ロール](#app-roles)」セクションで説明します)
* AAD 管理者ロールの要求 (`directoryRole`)
* ユーザーの携帯電話番号に対するユーザー プロファイル データの要求の例 (`mobilePhone`)
* AAD グループ要求 (`directoryGroup`)

`CustomAccountFactory.cs`:

```csharp
using System;
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
                userIdentity.AddClaim(new Claim("appRole", role));
            }

            foreach (var wid in account.Wids)
            {
                userIdentity.AddClaim(new Claim("directoryRole", wid));
            }

            try
            {
                var graphClient = ActivatorUtilities
                    .CreateInstance<GraphServiceClient>(serviceProvider);

                var requestMe = graphClient.Me.Request();
                var user = await requestMe.GetAsync();

                if (user != null)
                {
                    userIdentity.AddClaim(new Claim("mobilePhone",
                        user.MobilePhone));
                }

                var requestMemberOf = graphClient.Users[account.Oid].MemberOf;
                var memberships = await requestMemberOf.Request().GetAsync();

                if (memberships != null)
                {
                    foreach (var entry in memberships)
                    {
                        if (entry.ODataType == "#microsoft.graph.group")
                        {
                            userIdentity.AddClaim(
                                new Claim("directoryGroup", entry.Id));
                        }
                    }
                }
            }
            catch (ServiceException exception)
            {
                logger.LogError("Graph API service failure: {Message}",
                    exception.Message);
            }
        }

        return initialUser;
    }
}
```

上記のコードには、推移的なメンバーシップは含まれていません。 アプリで直接的および推移的なグループ メンバーシップの要求が必要である場合は、`MemberOf` プロパティ (`IUserMemberOfCollectionWithReferencesRequestBuilder`) を `TransitiveMemberOf` (`IUserTransitiveMemberOfCollectionWithReferencesRequestBuilder`) に置き換えます。

上記のコードでは、AAD 管理者ロール (`#microsoft.graph.directoryRole` の種類) であるグループ メンバーシップの要求 (`groups`) が無視されます。これは、Microsoft Identity プラットフォーム 2.0 によって返される GUID 値が、[**ロール テンプレート ID**](/azure/active-directory/roles/permissions-reference#role-template-ids) ではなく AAD 管理者ロールの **エンティティ ID** であるためです。 エンティティ ID は Microsoft Identity プラットフォーム 2.0 のテナント間で一定ではないため、アプリでユーザーの認可ポリシーを作成するために使用しないでください。 **`wids` 要求によって提供される** AAD 管理者ロールの **ロール テンプレート ID** を常に使用してください。

**クライアント** アプリの `Program.Main` で、カスタム ユーザー アカウント ファクトリを使用するように MSAL 認証を構成します。

`Program.cs`:

```csharp
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.Extensions.Configuration;

...

builder.Services.AddMsalAuthentication<RemoteAuthenticationState,
    CustomUserAccount>(options =>
{
    ...
})
.AddAccountClaimsPrincipalFactory<RemoteAuthenticationState, CustomUserAccount,
    CustomAccountFactory>();

...

builder.Services.AddGraphClient();
```

## <a name="authorization-configuration"></a>承認の構成

**クライアント** アプリの `Program.Main` で、各 [アプリ ロール](#app-roles)、AAD 管理者ロール、またはセキュリティ グループに対して [ポリシー](xref:security/authorization/policies)を作成します。 次の例では、AAD の "*課金管理者*" ロールに対するポリシーを作成します。

```csharp
builder.Services.AddAuthorizationCore(options =>
{
    options.AddPolicy("BillingAdministrator", policy => 
        policy.RequireClaim("directoryRole", 
            "b0f54661-2d74-4c50-afa3-1ec803f12efe"));
});
```

AAD 管理者ロールの ID の完全な一覧については、Azure ドキュメントの「[ロール テンプレート ID](/azure/active-directory/roles/permissions-reference#role-template-ids)」を参照してください。 認可ポリシーの詳細については、「<xref:security/authorization/policies>」を参照してください。

次の例では、**クライアント** アプリで前述のポリシーを使用してユーザーを承認します。

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

`Pages/CheckPolicy.razor`:

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

## <a name="authorize-server-apiweb-api-access"></a>サーバー API または Web API のアクセスを承認する

**サーバー** API アプリでは、アクセス トークンに `groups`、`wids`、`http://schemas.microsoft.com/ws/2008/06/identity/claims/role` の要求が含まれている場合に、セキュリティ グループ、AAD 管理者ロール、およびアプリ ロールに対する [認可ポリシー](xref:security/authorization/policies)を使用して、セキュリティで保護された API エンドポイントにアクセスすることをユーザーに許可できます。 次の例では、`Startup.ConfigureServices` で `wids` (既知の ID またはロール テンプレート ID) 要求を使用して、AAD "*課金管理者*" ロールに対するポリシーを作成しています。

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy("BillingAdministrator", policy => 
        policy.RequireClaim("wids", "b0f54661-2d74-4c50-afa3-1ec803f12efe"));
});
```

AAD 管理者ロールの ID の完全な一覧については、Azure ドキュメントの「[ロール テンプレート ID](/azure/active-directory/roles/permissions-reference#role-template-ids)」を参照してください。 認可ポリシーの詳細については、「<xref:security/authorization/policies>」を参照してください。

**サーバー** アプリのコントローラーへのアクセスは、[`[Authorize]` 属性](xref:security/authorization/simple)とポリシー名の使用に基づいて行うことができます (API ドキュメント:<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute>)。

次の例では、`BillingDataController` からの課金データへのアクセスを、`BillingAdministrator` というポリシー名を持つ Azure 課金管理者に制限しています。

```csharp
...
using Microsoft.AspNetCore.Authorization;

[Authorize(Policy = "BillingAdministrator")]
[ApiController]
[Route("[controller]")]
public class BillingDataController : ControllerBase
{
    ...
}
```

詳細については、「<xref:security/authorization/policies>」を参照してください。

## <a name="app-roles"></a>アプリ ロール

アプリ ロールのメンバーシップ要求を提供するように Azure portal でアプリを構成するには、「[方法: アプリケーションにアプリ ロールを追加してトークンで受け取る](/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps)」を Azure ドキュメントで参照してください。

以下の例では、**クライアント** と **サーバー** アプリが 2 つのロールを使用して構成されていること、およびロールがテスト ユーザーに割り当てられていることを前提としています。

* `admin`
* `developer`

> [!NOTE]
> ホストされている Blazor WebAssembly アプリ、またはスタンドアロン アプリのクライアントとサーバーのペア (スタンドアロンの Blazor WebAssembly アプリとスタンドアロンの ASP.NET Core サーバー API または Web API アプリ) を開発する場合、クライアントとサーバー両方の Azure portal アプリ登録の `appRoles` マニフェスト プロパティに、構成済みの同じロールを含める必要があります。 クライアント アプリのマニフェストでロールを確立したら、それら全体をサーバー アプリのマニフェストにコピーします。 クライアントとサーバーのアプリ登録の間でマニフェスト `appRoles` をミラー化しないと、アクセス トークンに適切なロール要求があっても、サーバー API または Web API の認証済みユーザーに対してロール要求が確立されません。

> [!NOTE]
> Azure AD Premium アカウントがないとグループにロールを割り当てることはできませんが、Standard Azure アカウントを使用して、ユーザーにロールを割り当て、ユーザーに対するロール要求を受け取ることができます。 このセクションのガイダンスでは、AAD Premium アカウントは必要ありません。
>
> Azure portal で各追加ロール割り当てに対して **_ユーザーを再追加する_** ことにより、複数のロールを割り当てます。

「[カスタム ユーザー アカウント](#custom-user-account)」セクションで示されている `CustomAccountFactory` は、JSON 配列値を持つ `roles` 要求に対して動作するように設定されています。 「[カスタム ユーザー アカウント](#custom-user-account)」セクションで示されているように、**クライアント** アプリに `CustomAccountFactory` を追加して登録します。 元の `roles` 要求はフレームワークによって自動的に削除されるため、それを削除するためのコードを提供する必要はありません。

**クライアント** アプリの `Program.Main` で、<xref:System.Security.Claims.ClaimsPrincipal.IsInRole%2A?displayProperty=nameWithType> のチェック用にロール要求として "`appRole`" という名前の要求を指定します。

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...

    options.UserOptions.RoleClaim = "appRole";
});
```

> [!NOTE]
> `directoryRoles` 要求 (管理者ロールの追加) を使用したい場合は、"`directoryRoles`" を <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticationUserOptions.RoleClaim?displayProperty=nameWithType> に代入します。

**サーバー** アプリの `Startup.ConfigureServices` で、<xref:System.Security.Claims.ClaimsPrincipal.IsInRole%2A?displayProperty=nameWithType> のチェック用にロール要求として "`http://schemas.microsoft.com/ws/2008/06/identity/claims/role`" という名前の要求を指定します。

```csharp
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(options =>
    {
        Configuration.Bind("AzureAd", options);
        options.TokenValidationParameters.RoleClaimType = 
            "http://schemas.microsoft.com/ws/2008/06/identity/claims/role";
    },
    options => { Configuration.Bind("AzureAd", options); });
```

> [!NOTE]
> `wids` 要求 (管理者ロールの追加) を使用したい場合は、"`wids`" を <xref:Microsoft.IdentityModel.Tokens.TokenValidationParameters.RoleClaimType?displayProperty=nameWithType> に代入します。

この時点でコンポーネントの承認方法が機能しています。 **クライアント** アプリのコンポーネント内のすべての認可メカニズムで、`admin` ロールを使用してユーザーを承認できます。

* [`AuthorizeView` コンポーネント](xref:blazor/security/index#authorizeview-component)

  ```razor
  <AuthorizeView Roles="admin">
  ```

* [`[Authorize]` 属性ディレクティブ](xref:blazor/security/index#authorize-attribute) (<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute>)

  ```razor
  @attribute [Authorize(Roles = "admin")]
  ```

* [手続き型ロジック](xref:blazor/security/index#procedural-logic)

  ```csharp
  var authState = await AuthenticationStateProvider.GetAuthenticationStateAsync();
  var user = authState.User;

  if (user.IsInRole("admin")) { ... }
  ```

複数のロール テストがサポートされています。

* `AuthorizeView` コンポーネントを使用して、ユーザーが `admin` **または** `developer` ロールの **いずれか** であることを要求します。

  ```razor
  <AuthorizeView Roles="admin, developer">
      ...
  </AuthorizeView>
  ```

* `AuthorizeView` コンポーネントを使用して、ユーザーが `admin` **および** `developer` ロールの **両方** であることを要求します。

  ```razor
  <AuthorizeView Roles="admin">
      <AuthorizeView Roles="developer">
          ...
      </AuthorizeView>
  </AuthorizeView>
  ```

* `[Authorize]` 属性を使用して、ユーザーが `admin` **または** `developer` ロールの **いずれか** であることを要求します。

  ```razor
  @attribute [Authorize(Roles = "admin, developer")]
  ```

* `[Authorize]` 属性を使用して、ユーザーが `admin` **および** `developer` ロールの **両方** であることを要求します。

  ```razor
  @attribute [Authorize(Roles = "admin")]
  @attribute [Authorize(Roles = "developer")]
  ```

* 手続き型のコードを使用して、ユーザーが `admin` **または** `developer` ロールの **いずれか** であることを要求します。

  ```razor
  @code {
      private async Task DoSomething()
      {
          var authState = await AuthenticationStateProvider
              .GetAuthenticationStateAsync();
          var user = authState.User;

          if (user.IsInRole("admin") || user.IsInRole("developer"))
          {
              ...
          }
          else
          {
              ...
          }
      }
  }
  ```

* 手続き型のコードを使用して、ユーザーが `admin` **および** `developer` ロールの **両方** であることを要求します。それには、前の例の [条件付き OR (`||`)](/dotnet/csharp/language-reference/operators/boolean-logical-operators) を [条件付き AND (`&&`)](/dotnet/csharp/language-reference/operators/boolean-logical-operators) に変更します。

  ```csharp
  if (user.IsInRole("admin") && user.IsInRole("developer"))
  ```

**サーバー** アプリのコントローラー内のすべての認可メカニズムで、`admin` ロールを使用してユーザーを承認できます。

* [`[Authorize]` 属性ディレクティブ](xref:blazor/security/index#authorize-attribute) (<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute>)

  ```csharp
  [Authorize(Roles = "admin")]
  ```

* [手続き型ロジック](xref:blazor/security/index#procedural-logic)

  ```csharp
  if (User.IsInRole("admin")) { ... }
  ```

複数のロール テストがサポートされています。

* `[Authorize]` 属性を使用して、ユーザーが `admin` **または** `developer` ロールの **いずれか** であることを要求します。

  ```csharp
  [Authorize(Roles = "admin, developer")]
  ```

* `[Authorize]` 属性を使用して、ユーザーが `admin` **および** `developer` ロールの **両方** であることを要求します。

  ```csharp
  [Authorize(Roles = "admin")]
  [Authorize(Roles = "developer")]
  ```

* 手続き型のコードを使用して、ユーザーが `admin` **または** `developer` ロールの **いずれか** であることを要求します。

  ```csharp
  static readonly string[] scopeRequiredByApi = new string[] { "API.Access" };

  ...

  [HttpGet]
  public IEnumerable<ReturnType> Get()
  {
      HttpContext.VerifyUserHasAnyAcceptedScope(scopeRequiredByApi);

      if (User.IsInRole("admin") || User.IsInRole("developer"))
      {
          ...
      }
      else
      {
          ...
      }

      return ...
  }
  ```

* 手続き型のコードを使用して、ユーザーが `admin` **および** `developer` ロールの **両方** であることを要求します。それには、前の例の [条件付き OR (`||`)](/dotnet/csharp/language-reference/operators/boolean-logical-operators) を [条件付き AND (`&&`)](/dotnet/csharp/language-reference/operators/boolean-logical-operators) に変更します。

  ```csharp
  if (User.IsInRole("admin") && User.IsInRole("developer"))
  ```

## <a name="additional-resources"></a>その他のリソース

* [ロール テンプレート ID (Azure ドキュメント)](/azure/active-directory/roles/permissions-reference#role-template-ids)
* [`groupMembershipClaims` 属性 (Azure ドキュメント)](/azure/active-directory/develop/reference-app-manifest#groupmembershipclaims-attribute)
* [方法: アプリケーションにアプリ ロールを追加してトークンで受け取る (Azure ドキュメント)](/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps)
* [アプリケーション ロール (Azure ドキュメント)](/azure/architecture/multitenant-identity/app-roles)
* <xref:security/authorization/claims>
* <xref:security/authorization/roles>
* <xref:blazor/security/index>
