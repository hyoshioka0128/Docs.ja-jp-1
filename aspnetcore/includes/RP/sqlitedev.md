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
ms.openlocfilehash: 9ca3b6ef5f0d4c68aaaf54ef3b458ba66fc43e97
ms.sourcegitcommit: 1f35de0ca9ba13ea63186c4dc387db4fb8e541e0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2021
ms.locfileid: "104711617"
---
<a name="sqlite-dev"></a>
### <a name="use-sqlite-for-development-sql-server-for-production"></a>開発用に SQLite を、運用環境に SQL Server を使用する

SQLite が選択されている場合、テンプレートで生成されたコードは開発の準備ができています。 次のコードは、`Startup` に <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> を挿入する方法を示しています。 `IWebHostEnvironment` が挿入されているので、`ConfigureServices` では開発用に SQLite を、運用環境に SQL Server を使用できます。

[!code-csharp[](~/includes/RP/code/StartupDevProd.cs?name=snippet&highlight=5,10,14)]
