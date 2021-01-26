---
title: ASP.NET Core Blazor の認証と承認
author: guardrex
description: Blazor の認証と承認のシナリオについて説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/19/2020
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
uid: blazor/security/index
ms.openlocfilehash: 35620299b1138ca45a6ebf1d789a9f4aa9b29518
ms.sourcegitcommit: 610936e4d3507f7f3d467ed7859ab9354ec158ba
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/25/2021
ms.locfileid: "98751606"
---
# <a name="aspnet-core-no-locblazor-authentication-and-authorization"></a>ASP.NET Core Blazor の認証と承認

作成者: [Steve Sanderson](https://github.com/SteveSandersonMS)、[Luke Latham](https://github.com/guardrex)

ASP.NET Core は、Blazor アプリのセキュリティの構成と管理をサポートしています。

Blazor Server アプリと Blazor WebAssembly アプリでは、セキュリティに関するシナリオが異なります。 Blazor Server アプリはサーバー上で実行するため、承認チェックでは以下のことを決定できます。

* ユーザーに表示される UI オプション (たとえば、ユーザーが利用できるメニュー エントリ)。
* アプリとコンポーネントの領域に対するアクセス規則。

Blazor WebAssembly アプリはクライアント上で実行されます。 承認は、表示する UI オプションを決定するために "*のみ*" 使用されます。 クライアント側のチェックはユーザーによって変更またはバイパスされる可能性があるため、Blazor WebAssembly アプリでは承認アクセス規則を適用できません。

[Razor Pages の承認規則](xref:security/authorization/razor-pages-authorization)は、ルーティング可能な Razor コンポーネントには適用されません。 ルーティング不可能な Razor コンポーネントが[ページに埋め込まれている](xref:blazor/components/prerendering-and-integration)場合、ページの承認規則は、Razor コンポーネントと、ページのコンテンツの残りの部分に間接的に影響します。

> [!NOTE]
> <xref:Microsoft.AspNetCore.Identity.SignInManager%601> と <xref:Microsoft.AspNetCore.Identity.UserManager%601> は、Razor コンポーネントではサポートされていません。

## <a name="authentication"></a>認証

Blazor は、既存の ASP.NET Core 認証メカニズムを使用してユーザーの ID を証明します。 詳細なメカニズムは、Blazor アプリのホスティング方法、Blazor WebAssembly サーバーか Blazor Server かによって異なります。

### <a name="no-locblazor-webassembly-authentication"></a>Blazor WebAssembly 認証

Blazor WebAssembly アプリでは、ユーザーがすべてのクライアント側コードを変更できるため、認証チェックがバイパスされる可能性があります。 JavaScript SPA フレームワークや任意のオペレーティング システム用のネイティブ アプリを含め、すべてのクライアント側アプリのテクノロジにも同じことが当てはまります。

以下を追加します。

* アプリのプロジェクト ファイルに、[`Microsoft.AspNetCore.Components.Authorization`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.Authorization) のパッケージ参照。
* アプリの `_Imports.razor` ファイルに、`Microsoft.AspNetCore.Components.Authorization` 名前空間。

認証を処理するために、組み込みまたはカスタムの <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> サービスの使用について、以降のセクションで説明します。

アプリの作成と構成の詳細については、「<xref:blazor/security/webassembly/index>」を参照してください。

### <a name="no-locblazor-server-authentication"></a>Blazor Server 認証

Blazor Server アプリは、SignalR を使用して作成されたリアルタイム接続を介して動作します。 [SignalR ベースのアプリの認証](xref:signalr/authn-and-authz)は、接続が確立したときに処理されます。 認証は、cookie または他のベアラー トークンに基づいている場合があります。

Blazor Server アプリの組み込みの <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> サービスでは、ASP.NET Core の `HttpContext.User` から認証状態データが取得されます。 このようにして、認証状態は既存の ASP.NET Core 認証メカニズムと統合されます。

アプリの作成と構成の詳細については、「<xref:blazor/security/server/index>」を参照してください。

## <a name="authenticationstateprovider-service"></a>AuthenticationStateProvider サービス

<xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> は、認証状態を取得するために <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> コンポーネントと <xref:Microsoft.AspNetCore.Components.Authorization.CascadingAuthenticationState> コンポーネントによって使用される基となるサービスです。

通常、<xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> は直接使用しません。 この記事で後述する [`AuthorizeView` コンポーネント](#authorizeview-component)または [`Task<AuthenticationState>`](#expose-the-authentication-state-as-a-cascading-parameter) のアプローチを使用します。 <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> を直接使用することの主な欠点は、基となる認証状態データが変更されてもコンポーネントに自動的に通知されないことです。

次の例に示すように、<xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> サービスから現在のユーザーの <xref:System.Security.Claims.ClaimsPrincipal> データを提供できます。

```razor
@page "/"
@using System.Security.Claims
@using Microsoft.AspNetCore.Components.Authorization
@inject AuthenticationStateProvider AuthenticationStateProvider

<h3>ClaimsPrincipal Data</h3>

<button @onclick="GetClaimsPrincipalData">Get ClaimsPrincipal Data</button>

<p>@_authMessage</p>

@if (_claims.Count() > 0)
{
    <ul>
        @foreach (var claim in _claims)
        {
            <li>@claim.Type: @claim.Value</li>
        }
    </ul>
}

<p>@_surnameMessage</p>

@code {
    private string _authMessage;
    private string _surnameMessage;
    private IEnumerable<Claim> _claims = Enumerable.Empty<Claim>();

    private async Task GetClaimsPrincipalData()
    {
        var authState = await AuthenticationStateProvider.GetAuthenticationStateAsync();
        var user = authState.User;

        if (user.Identity.IsAuthenticated)
        {
            _authMessage = $"{user.Identity.Name} is authenticated.";
            _claims = user.Claims;
            _surnameMessage = 
                $"Surname: {user.FindFirst(c => c.Type == ClaimTypes.Surname)?.Value}";
        }
        else
        {
            _authMessage = "The user is NOT authenticated.";
        }
    }
}
```

`user.Identity.IsAuthenticated` が `true` である場合、ユーザーが <xref:System.Security.Claims.ClaimsPrincipal> であるため、要求を列挙し、役割のメンバーシップを評価できます。

依存関係の注入 (DI) とサービスの詳細については、<xref:blazor/fundamentals/dependency-injection> と <xref:fundamentals/dependency-injection> を参照してください。

## <a name="implement-a-custom-authenticationstateprovider"></a>カスタム AuthenticationStateProvider の実装

アプリでカスタム プロバイダーを必要とする場合は、<xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> を実装し、`GetAuthenticationStateAsync` をオーバーライドします。

```csharp
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.Authorization;

public class CustomAuthStateProvider : AuthenticationStateProvider
{
    public override Task<AuthenticationState> GetAuthenticationStateAsync()
    {
        var identity = new ClaimsIdentity(new[]
        {
            new Claim(ClaimTypes.Name, "mrfibuli"),
        }, "Fake authentication type");

        var user = new ClaimsPrincipal(identity);

        return Task.FromResult(new AuthenticationState(user));
    }
}
```

Blazor WebAssembly アプリでは、`CustomAuthStateProvider` サービスは `Program.cs` の `Main` に登録されています。

```csharp
using Microsoft.AspNetCore.Components.Authorization;

...

builder.Services.AddScoped<AuthenticationStateProvider, CustomAuthStateProvider>();
```

Blazor Server アプリでは、`CustomAuthStateProvider` サービスは `Startup.ConfigureServices` に登録されています。

```csharp
using Microsoft.AspNetCore.Components.Authorization;

...

services.AddScoped<AuthenticationStateProvider, CustomAuthStateProvider>();
```

前の例で `CustomAuthStateProvider` を使用すると、すべてのユーザーはユーザー名 `mrfibuli` で認証されます。

## <a name="expose-the-authentication-state-as-a-cascading-parameter"></a>認証状態をカスケード パラメーターとして公開する

ユーザーによってトリガーされたアクションを実行する場合など、認証状態データが手続き型ロジックに必要な場合は、型 `Task<`<xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationState>`>` のカスケード パラメーターを定義して認証状態データを取得します。

```razor
@page "/"

<button @onclick="LogUsername">Log username</button>

<p>@_authMessage</p>

@code {
    [CascadingParameter]
    private Task<AuthenticationState> authenticationStateTask { get; set; }

    private string _authMessage;

    private async Task LogUsername()
    {
        var authState = await authenticationStateTask;
        var user = authState.User;

        if (user.Identity.IsAuthenticated)
        {
            _authMessage = $"{user.Identity.Name} is authenticated.";
        }
        else
        {
            _authMessage = "The user is NOT authenticated.";
        }
    }
}
```

`user.Identity.IsAuthenticated` が `true` である場合、要求を列挙し、役割のメンバーシップを評価できます。

`App` コンポーネント (`App.razor`) 内の <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeRouteView> および <xref:Microsoft.AspNetCore.Components.Authorization.CascadingAuthenticationState> コンポーネントを使用して、`Task<`<xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationState>`>` カスケード パラメーターを設定します。

```razor
<CascadingAuthenticationState>
    <Router AppAssembly="@typeof(Program).Assembly">
        <Found Context="routeData">
            <AuthorizeRouteView RouteData="@routeData" 
                DefaultLayout="@typeof(MainLayout)" />
        </Found>
        <NotFound>
            <LayoutView Layout="@typeof(MainLayout)">
                <p>Sorry, there's nothing at this address.</p>
            </LayoutView>
        </NotFound>
    </Router>
</CascadingAuthenticationState>
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

Blazor WebAssembly アプリでは、オプションと承認のためのサービスを `Program.Main` に追加します。

```csharp
builder.Services.AddOptions();
builder.Services.AddAuthorizationCore();
```

Blazor Server アプリでは、オプションと承認のためのサービスが既に存在するため、これ以上の操作は必要ありません。

## <a name="authorization"></a>承認

ユーザーが認証されると、ユーザーが実行できる操作を制御する "*承認*" 規則が適用されます。

通常、アクセスは以下の条件に基づいて許可または拒否されます。

* ユーザーが認証されている (サインインしている)。
* ユーザーに "*ロール*" が割り当てられている。
* ユーザーに "*要求*" がある。
* "*ポリシー*" が満たされている。

これらの各概念は、ASP.NET Core MVC または Razor Pages アプリと同じです。 ASP.NET Core のセキュリティの詳細については、[ASP.NET Core のセキュリティと Identity](xref:security/index) の記事を参照してください。

## <a name="authorizeview-component"></a>AuthorizeView コンポーネント

<xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> コンポーネントでは、ユーザーを承認するかどうかに応じて UI コンテンツが選択的に表示されます。 このアプローチは、ユーザーに対してデータを "*表示する*" だけで済み、手続き型ロジックでユーザーの ID を使用する必要がない場合に便利です。

このコンポーネントでは、型 <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationState> の `context` 変数が公開されており、これを使用して、サインインしたユーザーに関する情報にアクセスできます。

```razor
<AuthorizeView>
    <h1>Hello, @context.User.Identity.Name!</h1>
    <p>You can only see this content if you're authenticated.</p>
</AuthorizeView>
```

ユーザーが承認されていない場合は、表示用に異なるコンテンツを指定することもできます。

```razor
<AuthorizeView>
    <Authorized>
        <h1>Hello, @context.User.Identity.Name!</h1>
        <p>You can only see this content if you're authorized.</p>
        <button @onclick="SecureMethod">Authorized Only Button</button>
    </Authorized>
    <NotAuthorized>
        <h1>Authentication Failure!</h1>
        <p>You're not signed in.</p>
    </NotAuthorized>
</AuthorizeView>

@code {
    private void SecureMethod() { ... }
}
```

`<Authorized>` および `<NotAuthorized>` タグには、他の対話型コンポーネントなど、任意の項目を含めることができます。

前の例の `<button>` 要素の `SecureMethod` メソッドなど、承認された要素の既定のイベント ハンドラーは、承認されたユーザーのみが呼び出すことができます。

UI オプションまたはアクセスを制御するロールやポリシーなどの承認条件については、「[承認](#authorization)」セクションを参照してください。

承認条件が指定されていない場合、<xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> では既定のポリシーが使用され、以下が扱われます。

* 承認済みの認証された (サインインした) ユーザー。
* 未承認の認証されていない (サインアウトした) ユーザー。

<xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> コンポーネントは、`NavMenu` コンポーネント (`Shared/NavMenu.razor`) で使用して [`NavLink` コンポーネント](xref:blazor/fundamentals/routing#navlink-and-navmenu-components) (<xref:Microsoft.AspNetCore.Components.Routing.NavLink>) のリスト項目 (`<li>...</li>`) を表示できますが、この方法では、表示された出力からリスト項目が削除されるだけであることに注意してください。 ユーザーがコンポーネントに移動するのを防ぐことはできません。

### <a name="role-based-and-policy-based-authorization"></a>ロールベースとリソースベースの承認

<xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> コンポーネントは、"*ロールベース*" または "*ポリシーベース*" の承認をサポートしています。

ロールベースの承認の場合は、<xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView.Roles> パラメーターを使用します。

```razor
<AuthorizeView Roles="admin, superuser">
    <p>You can only see this if you're an admin or superuser.</p>
</AuthorizeView>
```

詳細については、「<xref:security/authorization/roles>」を参照してください。

ポリシーベースの承認の場合は、<xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView.Policy> パラメーターを使用します。

```razor
<AuthorizeView Policy="content-editor">
    <p>You can only see this if you satisfy the "content-editor" policy.</p>
</AuthorizeView>
```

要求ベースの承認は、ポリシーベースの承認の特殊なケースです。 たとえば、ユーザーが特定の要求持つことを必須にするポリシーを定義できます。 詳細については、「<xref:security/authorization/policies>」を参照してください。

これらの API は、Blazor Server アプリまたは Blazor WebAssembly アプリで使用できます。

<xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView.Roles> も <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView.Policy> も指定されていない場合、<xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> には既定のポリシーが使用されます。

### <a name="content-displayed-during-asynchronous-authentication"></a>非同期認証中に表示されるコンテンツ

Blazor では、認証状態を "*非同期的に*" 決定することができます。 このアプローチの主なシナリオは、認証のために外部エンドポイントに要求を送信する Blazor WebAssembly アプリです。

認証が進行中の間、<xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> には既定でコンテンツが表示されません。 認証が行われている間にコンテンツを表示するには、`<Authorizing>` タグを使用します。

```razor
<AuthorizeView>
    <Authorized>
        <h1>Hello, @context.User.Identity.Name!</h1>
        <p>You can only see this content if you're authenticated.</p>
    </Authorized>
    <Authorizing>
        <h1>Authentication in progress</h1>
        <p>You can only see this content while authentication is in progress.</p>
    </Authorizing>
</AuthorizeView>
```

通常、このアプローチは Blazor Server アプリには適用されません。 Blazor Server アプリでは、状態が確立されるとすぐに認証状態が認識されます。 <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeViewCore.Authorizing> コンテンツは Blazor Server アプリの <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> コンポーネントで提供できますが、コンテンツは表示されません。

## <a name="authorize-attribute"></a>[Authorize] 属性

[`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) 属性は Razor コンポーネント内で使用できます。

```razor
@page "/"
@attribute [Authorize]

You can only see this if you're signed in.
```

> [!IMPORTANT]
> Blazor ルーター経由で到達した `@page` コンポーネントにのみ [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) を使用してください。 承認はルーティングの一面としてのみ実行され、ページ内にレンダリングされた子コンポーネントに対しては実行され "*ません*"。 ページ内の特定部分の表示を承認するには、代わりに <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> を使用します。

[`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) 属性は、ロールベースまたはポリシーベースの承認もサポートしています。 ロールベースの承認の場合は、<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute.Roles> パラメーターを使用します。

```razor
@page "/"
@attribute [Authorize(Roles = "admin, superuser")]

<p>You can only see this if you're in the 'admin' or 'superuser' role.</p>
```

ポリシーベースの承認の場合は、<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute.Policy> パラメーターを使用します。

```razor
@page "/"
@attribute [Authorize(Policy = "content-editor")]

<p>You can only see this if you satisfy the 'content-editor' policy.</p>
```

<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute.Roles> も <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute.Policy> も指定されていない場合、[`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) には既定のポリシーが使用されます。これは既定で以下が扱われます。

* 承認済みの認証された (サインインした) ユーザー。
* 未承認の認証されていない (サインアウトした) ユーザー。

## <a name="customize-unauthorized-content-with-the-router-component"></a>Router コンポーネントを使用して承認されていないコンテンツをカスタマイズする

<xref:Microsoft.AspNetCore.Components.Routing.Router> コンポーネントを <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeRouteView> コンポーネントとともに使用すると、以下の場合にアプリがカスタム コンテンツを指定できます。

* コンテンツが見つからない。
* ユーザーはコンポーネントに適用されている [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) 条件に失敗します。 [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) 属性については、「[`[Authorize]` 属性](#authorize-attribute)」セクションを参照してください。
* 非同期認証が実行中です。

既定の Blazor Server プロジェクト テンプレートでは、`App` コンポーネント (`App.razor`) によりカスタム コンテンツの設定方法が示されます。

```razor
<CascadingAuthenticationState>
    <Router AppAssembly="@typeof(Program).Assembly">
        <Found Context="routeData">
            <AuthorizeRouteView RouteData="@routeData" 
                DefaultLayout="@typeof(MainLayout)">
                <NotAuthorized>
                    <h1>Sorry</h1>
                    <p>You're not authorized to reach this page.</p>
                    <p>You may need to log in as a different user.</p>
                </NotAuthorized>
                <Authorizing>
                    <h1>Authentication in progress</h1>
                    <p>Only visible while authentication is in progress.</p>
                </Authorizing>
            </AuthorizeRouteView>
        </Found>
        <NotFound>
            <LayoutView Layout="@typeof(MainLayout)">
                <h1>Sorry</h1>
                <p>Sorry, there's nothing at this address.</p>
            </LayoutView>
        </NotFound>
    </Router>
</CascadingAuthenticationState>
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

`<NotFound>`、`<NotAuthorized>`、および `<Authorizing>` タグには、他の対話型コンポーネントなど、任意の項目を含めることができます。

`<NotAuthorized>` タグが指定されていない場合、<xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeRouteView> には次のフォールバック メッセージが使用されます。

```html
Not authorized.
```

## <a name="notification-about-authentication-state-changes"></a>認証状態の変更に関する通知

基となる認証状態データが変わったとアプリに判断された場合 (たとえば、ユーザーがサインアウトした、または別のユーザーがロールを変更したなど)、[カスタムの `AuthenticationStateProvider`](#implement-a-custom-authenticationstateprovider) では必要に応じて <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> 基底クラスのメソッド <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider.NotifyAuthenticationStateChanged%2A> を呼び出すことができます。 これにより、新しいデータを使用して再表示するように、認証状態データ (<xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> など) がコンシューマーに通知されます。

## <a name="procedural-logic"></a>手続き型ロジック

手続き型ロジックの一部としてアプリで承認規則をチェックする必要がある場合、型 `Task<`<xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationState>`>` のカスケード パラメーターを使用してユーザーの <xref:System.Security.Claims.ClaimsPrincipal> を取得します。 ポリシーを評価するために、`Task<`<xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationState>`>` を `IAuthorizationService` などの他のサービスと組み合わせることができます。

```razor
@using Microsoft.AspNetCore.Authorization
@inject IAuthorizationService AuthorizationService

<button @onclick="@DoSomething">Do something important</button>

@code {
    [CascadingParameter]
    private Task<AuthenticationState> authenticationStateTask { get; set; }

    private async Task DoSomething()
    {
        var user = (await authenticationStateTask).User;

        if (user.Identity.IsAuthenticated)
        {
            // Perform an action only available to authenticated (signed-in) users.
        }

        if (user.IsInRole("admin"))
        {
            // Perform an action only available to users in the 'admin' role.
        }

        if ((await AuthorizationService.AuthorizeAsync(user, "content-editor"))
            .Succeeded)
        {
            // Perform an action only available to users satisfying the 
            // 'content-editor' policy.
        }
    }
}
```

> [!NOTE]
> Blazor WebAssembly アプリ コンポーネントで、<xref:Microsoft.AspNetCore.Authorization> と <xref:Microsoft.AspNetCore.Components.Authorization> の名前空間を追加します。
>
> ```razor
> @using Microsoft.AspNetCore.Authorization
> @using Microsoft.AspNetCore.Components.Authorization
> ```
>
> これらの名前空間は、アプリの `_Imports.razor` ファイルに追加することで、グローバルに提供できます。

## <a name="troubleshoot-errors"></a>エラーのトラブルシューティング

一般的なエラー:

* **承認には、型 `Task\<AuthenticationState>` のカスケード パラメーターが必要です。これを実行するには `CascadingAuthenticationState` の使用を検討します。**

* `authenticationStateTask` **に対して**`null` 値を受け取ります

認証が有効な Blazor Server テンプレートを使用してプロジェクトが作成されなかった可能性があります。 UI ツリーの一部に `<CascadingAuthenticationState>` をラップします。たとえば、`App` コンポーネント (`App.razor`) で次のようにします。

```razor
<CascadingAuthenticationState>
    <Router AppAssembly="@typeof(Program).Assembly">
        ...
    </Router>
</CascadingAuthenticationState>
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

<xref:Microsoft.AspNetCore.Components.Authorization.CascadingAuthenticationState> には `Task<`<xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationState>`>` カスケード パラメーターが用意されており、次に基となる <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> DI サービスから受け取ります。

## <a name="additional-resources"></a>その他の技術情報

* <xref:security/index>
* <xref:security/authentication/windowsauth>
* [Authentication.MSAL JavaScript ライブラリのカスタム バージョンをビルドする](xref:blazor/security/webassembly/additional-scenarios#build-a-custom-version-of-the-authenticationmsal-javascript-library)
* [すばらしい Blazor: 認証](https://github.com/AdrienTorris/awesome-blazor#authentication) コミュニティのサンプル リンク
