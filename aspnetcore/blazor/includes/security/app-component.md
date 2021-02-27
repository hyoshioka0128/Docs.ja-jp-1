---
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
ms.openlocfilehash: d632ab0604f81f7b6067d4535b0f5da0afe2e0ad
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551517"
---
`App` コンポーネント (`App.razor`) は、Blazor Server アプリにある `App` コンポーネントに似ています。

* <xref:Microsoft.AspNetCore.Components.Authorization.CascadingAuthenticationState> コンポーネントによって、アプリの残りの部分に <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationState> を公開する動作が管理されます。
* <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeRouteView> コンポーネントによって、現在のユーザーには所与のページへのアクセスが許可されます。それ以外では、`RedirectToLogin` コンポーネントがレンダリングされます。
* `RedirectToLogin` コンポーネントによって、承認されていないユーザーのログイン ページへのリダイレクトが管理されます。

```razor
<CascadingAuthenticationState>
    <Router AppAssembly="@typeof(Program).Assembly">
        <Found Context="routeData">
            <AuthorizeRouteView RouteData="@routeData" 
                DefaultLayout="@typeof(MainLayout)">
                <NotAuthorized>
                    @if (!context.User.Identity.IsAuthenticated)
                    {
                        <RedirectToLogin />
                    }
                    else
                    {
                        <p>
                            You are not authorized to access 
                            this resource.
                        </p>
                    }
                </NotAuthorized>
            </AuthorizeRouteView>
        </Found>
        <NotFound>
            <LayoutView Layout="@typeof(MainLayout)">
                <p>Sorry, there's nothing at this address.</p>
            </LayoutView>
        </NotFound>
    </Router>
</CascadingAuthenticationState>
```

[!include[](../prefer-exact-matches.md)]
