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
ms.openlocfilehash: ddc6e2abf60577644a38b0b6ad0c74a734205ef5
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551770"
---
Forwarded Headers Middleware は、他のミドルウェアの前に実行する必要があります。 この順序により、転送されるヘッダー情報に依存するミドルウェアが処理にヘッダー値を使用できます。 診断およびエラー処理ミドルウェアの後に Forwarded Headers Middleware を実行する方法については、「[Forwarded Headers Middleware の順序](xref:host-and-deploy/proxy-load-balancer#fhmo)」を参照してください。