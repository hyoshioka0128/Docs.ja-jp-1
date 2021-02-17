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
ms.openlocfilehash: ca743cdb39423d833902c330f6f0682b600c9833
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552769"
---
::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

> [!WARNING]
> ルーティングで [バグ](https://github.com/dotnet/aspnetcore/issues/18677)が原因で、**キャッチオール** パラメーターがルートと正しく一致しない可能性があります。 このバグの影響を受けるアプリには、次の特性があります。
>
> * キャッチオール ルート (たとえば、`{**slug}"`)
> * キャッチオール ルートが、一致すべき要求と一致しません。
> * 他のルートを削除すると、キャッチオール ルートが機能し始めます。
>
> このバグが発生するケースの例については、GitHub のバグ [18677](https://github.com/dotnet/aspnetcore/issues/18677) および [16579](https://github.com/dotnet/aspnetcore/issues/16579) を参照してください。
>
> このバグのオプトイン修正は [.NET Core 3.1.301 SDK 以降](https://dotnet.microsoft.com/download/dotnet-core/3.1)に含まれています。 次のコードにより、このバグを修正する内部スイッチが設定されます。
>
>```csharp
>public static void Main(string[] args)
>{
>    AppContext.SetSwitch("Microsoft.AspNetCore.Routing.UseCorrectCatchAllBehavior", 
>                          true);
>    CreateHostBuilder(args).Build().Run();
>}
>// Remaining code removed for brevity.
>```

::: moniker-end