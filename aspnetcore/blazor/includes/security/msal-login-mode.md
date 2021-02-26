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
ms.openlocfilehash: 1b045d437d1a16eabc0ab41573c8b66d9c4bb77e
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552702"
---
フレームワークは、既定ではポップアップ ログイン モードになり、ポップアップを開くことができない場合はリダイレクト ログイン モードに戻ります。 <xref:Microsoft.Authentication.WebAssembly.Msal.Models.MsalProviderOptions> の `LoginMode` プロパティを `redirect` に設定して、リダイレクト ログイン モードを使用するように MSAL を構成します。

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...
    options.ProviderOptions.LoginMode = "redirect";
});
```

既定の設定は `popup` であり、文字列の値の大文字と小文字は区別されません。
