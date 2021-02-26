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
ms.openlocfilehash: 34e12afd6bc7015e4609fbdf773259ed5545809b
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551843"
---
# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. 新しいプロジェクトを作成します。
1. **[ワーカー サービス]** を選択します。 **[次へ]** を選択します。
1. **[プロジェクト名]** フィールドにプロジェクト名を入力するか、既定のプロジェクト名をそのまま使用します。 **［作成］** を選択します
1. **[新しい Worker サービスを作成します]** ダイアログで、**[作成]** を選択します。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

1. 新しいプロジェクトを作成します。
1. サイドバーの **[.NET Core]** の下で **[アプリ]** を選択します。
1. **[ASP.NET Core]** の下の **[ワーカー]** を選択します。 **[次へ]** を選択します。
1. **[ターゲット フレームワーク]** で **[.NET Core 3.0]** またはそれ以降を選択します。 **[次へ]** を選択します。
1. **[プロジェクト名]** フィールドに名前を指定します。 **［作成］** を選択します

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

コマンド シェルから [dotnet new](/dotnet/core/tools/dotnet-new) コマンドと共にワーカー サービス (`worker`) テンプレートを使用します。 次の例では、`ContosoWorker` という名前のワーカー サービス アプリが作成されます。 このコマンドが実行されると、`ContosoWorker` アプリ用のフォルダーが自動的に作成されます。

```dotnetcli
dotnet new worker -o ContosoWorker
```

---
