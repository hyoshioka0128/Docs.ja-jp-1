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
ms.openlocfilehash: 6435d39b6d442f0d4643d77d9111d11b50781544
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100536301"
---
`:` の区切り記号は、すべてのプラットフォームの環境変数階層キーには対応していません。 `__`（ダブルアンダースコア）は、

* すべてのプラットフォームに対応しています。 たとえば、[Bash](https://linuxhint.com/bash-environment-variables/) は`:` の区切り記号には対応していませんが、`__` には対応しています。
* 自動で `:` に置換されます。