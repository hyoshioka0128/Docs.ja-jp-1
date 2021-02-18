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
ms.openlocfilehash: 5fb658d165ac679be232e983f5a4da5ebf3422b3
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552077"
---
ASP.NET Core Identity では、ASP.NET Core Web アプリにユーザー インターフェイス (UI) ログイン機能が追加されます。 Web API と SPA をセキュリティで保護するには、次のいずれかを使用します。

* [Azure Active Directory](/azure/api-management/api-management-howto-protect-backend-with-aad)
* [Azure Active Directory B2C](/azure/active-directory-b2c/active-directory-b2c-custom-rest-api-netfw) (Azure AD B2C)
* [IdentityServer4](https://identityserver.io)

IdentityServer4 は、ASP.NET Core 用の OpenID Connect および OAuth 2.0 フレームワークです。 IdentityServer4 により、次のセキュリティ機能が有効になります。

* サービスとしての認証 (AaaS)
* 複数のアプリケーションの種類でのシングル サインオン/オフ (SSO)
* API のアクセス制御
* Federation Gateway

詳細については、「[ようこそ! IdentityServer4](https://docs.identityserver.io/en/latest/index.html)」を参照してください。
