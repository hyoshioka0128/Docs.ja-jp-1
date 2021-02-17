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
ms.openlocfilehash: 30c4c469453f0202fe5310dbe00b3d7214f49b5a
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551627"
---
* `-F|--no-formatting`

  存在することで HTTP 応答の書式設定を抑制するフラグ。

* `-h|--header`

  HTTP 要求ヘッダーを設定します。 次の 2 つの値の形式がサポートされています。

  * `{header}={value}`
  * `{header}:{value}`

* `--response:body`

  HTTP 応答の本文を書き込むファイルを指定します。 たとえば、「 `--response:body "C:\response.json"` 」のように入力します。 ファイルが存在しない場合は作成されます。

* `--response:headers`

  HTTP 応答のヘッダーを書き込むファイルを指定します。 たとえば、「 `--response:headers "C:\response.txt"` 」のように入力します。 ファイルが存在しない場合は作成されます。

* `-s|--streaming`

  存在することで HTTP 応答のストリーミングを有効にするフラグ。
