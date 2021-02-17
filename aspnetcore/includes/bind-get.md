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
ms.openlocfilehash: c6880852f63b1e91789667506f01680608933226
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552458"
---
> [!WARNING]
> セキュリティ上の理由から、ページ モデルのプロパティに対して `GET` 要求データのバインドをオプトインする必要があります。 プロパティにマップする前に、ユーザー入力を確認してください。 `GET` バインドをオプトインするのは、クエリ文字列やルート値に依存するシナリオに対処する場合に便利です。
>
> `GET` 要求のプロパティをバインドするには、[`[BindProperty]`](xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute) 属性の `SupportsGet` プロパティを `true` に設定します。
>
> ```csharp
> [BindProperty(SupportsGet = true)]
> ```
>
> 詳細については、[ASP.NET Core コミュニティ スタンドアップ: GET のディスカッションでのバインド (YouTube)](https://www.youtube.com/watch?v=p7iHB9V-KVU&feature=youtu.be&t=54m27s) に関する動画を参照してください。
