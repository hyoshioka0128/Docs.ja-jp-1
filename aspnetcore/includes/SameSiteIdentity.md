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
ms.openlocfilehash: b6f6bc2e094c9070e0ea57b507f558313f19bc15
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552415"
---
[Identity](xref:security/authentication/identity)またはの統合のような高度なシナリオを除き、ASP.NET Core は[SameSite cookie s](xref:security/samesite)の主な影響を受け `IFrames` `OpenIdConnect` ません。

を使用する場合 `Identity` 、プロバイダーを追加し cookie たり、を呼び出したりしないで ` services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)` `Identity` ください。