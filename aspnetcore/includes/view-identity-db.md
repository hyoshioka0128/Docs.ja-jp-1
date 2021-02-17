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
ms.openlocfilehash: ff0502f68546213d76fe33f45574b5d8654b522b
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552300"
---
### <a name="view-the-identity-database"></a>データベースを表示する Identity

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio) 

* [ **表示** ] メニューの [ **SQL Server オブジェクトエクスプローラー** (ssox)] を選択します。
* **(Localdb) MSSQLLocalDB (SQL Server 13)** に移動します。 Dbo を右クリックし **ます。AspNetUsers**  >  **View データ**:

![SQL Server オブジェクトエクスプローラーの AspNetUsers テーブルのコンテキストメニュー](~/security/authentication/accconfirm/_static/ssox.png)

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

Sqlite データベースを管理および表示するためにダウンロードできるサードパーティ製のツールは多数あります。たとえば、 [sqlite 用の DB Browser](https://sqlitebrowser.org/)などです。

---