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
ms.openlocfilehash: df4a9a7db8097ea765b2d0f404305b771f994ddf
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552473"
---
* 次のコマンドを実行し、HTTPS 開発証明書を信頼します。

  ```dotnetcli
  dotnet dev-certs https --trust
  ```
  
  上記のコマンドは、Linux では機能しません。 証明書を信頼する方法については、Linux ディストリビューションのドキュメントを参照してください。

  上記のコマンドでは、次のダイアログが表示されます。

  ![セキュリティ警告のダイアログ](~/getting-started/_static/cert.png)

* 開発証明書を信頼することに同意する場合は、 **[はい]** を選択します。

  詳細については、[ASP.NET Core HTTPS 開発証明書の信頼](xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos)に関する記事をご覧ください
  
[!INCLUDE[trust FF](~/includes/trust-ff.md)]