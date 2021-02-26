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
ms.openlocfilehash: 5964554c36e2242b70faee390374828acd2bd860
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552940"
---
AAD に登録されているサーバー API を使用するときに、アプリの AAD 登録が[未確認の発行元ドメイン](/azure/active-directory/develop/howto-configure-publisher-domain)に依存するテナント内にある場合、サーバー API アプリのアプリ ID の URI は、`api://{SERVER API APP CLIENT ID OR CUSTOM VALUE}` ではなく `https://{TENANT}.onmicrosoft.com/{SERVER API APP CLIENT ID OR CUSTOM VALUE}` の形式になります。 このケースの場合、 *`Client`* アプリの `Program.Main` (`Program.cs`) の既定のアクセス トークン スコープは、次のようになります。

```csharp
options.ProviderOptions.DefaultAccessTokenScopes
    .Add("https://{TENANT}.onmicrosoft.com/{SERVER API APP CLIENT ID OR CUSTOM VALUE}/{DEFAULT SCOPE}");
```

一致する対象ユーザーに対してサーバー API アプリを構成するには、 *`Server`* API アプリ設定ファイル (`appsettings.json`) の `Audience` を、Azure portal によって提供されるアプリの対象ユーザーと一致するように設定します。

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/{TENANT ID}",
    "ClientId": "{SERVER API APP CLIENT ID}",
    "ValidateAuthority": true,
    "Audience": "https://{TENANT}.onmicrosoft.com/{SERVER API APP CLIENT ID OR CUSTOM VALUE}"
  }
}
```

上記の構成では、`Audience` 値の末尾に既定のスコープ `/{DEFAULT SCOPE}` が含まれて **いません**。

例:

*`Client`* アプリの `Program.Main` (`Program.cs`):

```csharp
options.ProviderOptions.DefaultAccessTokenScopes
    .Add("https://contoso.onmicrosoft.com/41451fa7-82d9-4673-8fa5-69eff5a761fd/API.Access");
```

一致する対象ユーザー (`Audience`) を使用して *`Server`* API アプリ設定ファイル (`appsettings.json`) を構成します。

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/e86c78e2-...-918e0565a45e",
    "ClientId": "41451fa7-82d9-4673-8fa5-69eff5a761fd",
    "ValidateAuthority": true,
    "Audience": "https://contoso.onmicrosoft.com/41451fa7-82d9-4673-8fa5-69eff5a761fd"
  }
}
```

上記の例では、`Audience` 値の末尾に既定のスコープ `/API.Access` が含まれて **いません**。
