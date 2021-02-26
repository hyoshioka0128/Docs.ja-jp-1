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
ms.openlocfilehash: 7caea4089d3624d4c02db4b8adbe9edb73f3d31a
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552468"
---
> [!NOTE]
> ASP.NET Core 3.0 より前の Web アプリでは、要求されたカルチャがサポートされていない場合、要求ごとに `LogLevel.Warning` の種類のログが 1 つ書き込まれます。 要求ごとに 1 つの `LogLevel.Warning` をログに記録する場合、冗長な情報を含む大きなログ ファイルが作成される可能性があります。 この動作は、ASP.NET 3.0 で変更されています。 `RequestLocalizationMiddleware` によって `LogLevel.Debug` の種類のログが書き込まれるため、運用ログのサイズが縮小されます。
