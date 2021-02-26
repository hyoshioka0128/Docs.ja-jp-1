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
ms.openlocfilehash: b2519eaab7a83d66e60be05b1c2deeb8094b016f
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551596"
---
Index ページ (`wwwroot/index.html`) ページには、JavaScript で `AuthenticationService` を定義するスクリプトが含まれています。 `AuthenticationService` によって、OIDC プロトコルの下位レベルの詳細が処理されます。 アプリは、認証操作を実行するために、スクリプトで定義されているメソッドを内部的に呼び出します。

```html
<script src="_content/Microsoft.AspNetCore.Components.WebAssembly.Authentication/
    AuthenticationService.js"></script>
```
