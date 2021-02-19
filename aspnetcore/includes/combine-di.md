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
ms.openlocfilehash: 08d212b48c3f97531bea34b11d5b8c9a9069ae34
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100536302"
---
<a name="csc"></a>

サービスを登録し、オプションを構成する次の `ConfigureServices` メソッドについて考えてみましょう。

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Startup2.cs?name=snippet)]

関連する登録グループは、サービスを登録するための拡張メソッドに移動できます。 たとえば、構成サービスは次のクラスに追加されます。

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Options/MyConfigServiceCollectionExtensions.cs)]

残りのサービスは、同様のクラスに登録されます。 次の `ConfigureServices` メソッドでは、新しい拡張メソッドを使用してサービスを登録します。

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Startup4.cs?name=snippet)]

**_注:_** 各 `services.Add{GROUP_NAME}` 拡張メソッドは、サービスを追加、場合によっては構成します。 たとえば、<xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddControllersWithViews%2A> ではビューを持つ MVC コントローラーで必要なサービスが追加され、<xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddRazorPages%2A> では Razor Pages で必要なサービスが追加されます。 アプリをこの名前付け規則に従わせることをお勧めします。 拡張メソッドを <xref:Microsoft.Extensions.DependencyInjection?displayProperty=fullName> 名前空間に配置して、サービス登録のグループをカプセル化します。
