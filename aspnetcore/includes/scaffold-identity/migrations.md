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
ms.openlocfilehash: 1700adafb58cad57ea1becbf53cad45edd047962
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552968"
---
生成された Identity データベースコードには [Entity Framework Core 移行](/ef/core/managing-schemas/migrations/)が必要です。 移行を作成し、データベースを更新します。 たとえば、次のコマンドを実行します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Visual Studio **パッケージマネージャーコンソール** で次のようにします。

```powershell
Install-Package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore
Add-Migration CreateIdentitySchema
Update-Database
```

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

```dotnetcli
dotnet ef migrations add CreateIdentitySchema
dotnet ef database update
```

---

Identityコマンドの "Create Schema" name パラメーター `Add-Migration` は任意です。 `"CreateIdentitySchema"` 移行について説明します。
