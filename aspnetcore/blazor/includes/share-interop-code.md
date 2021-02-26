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
ms.openlocfilehash: 378c74c930f6e3f9565f408a2761a8ed867450d3
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551236"
---
## <a name="share-interop-code-in-a-class-library"></a>クラス ライブラリで相互運用コードを共有する

JS 相互運用コードは、クラス ライブラリに含めて NuGet パッケージ内でコードを共有することができます。

クラス ライブラリは、ビルドされたアセンブリでの JavaScript リソースの埋め込みを処理します。 JavaScript ファイルは、`wwwroot` フォルダーに配置されます。 ライブラリのビルド時にツールがリソースの埋め込みを行います。

ビルド済みの NuGet パッケージは、NuGet パッケージの参照と同じ方法で、アプリのプロジェクト ファイルで参照されます。 パッケージが復元されたら、アプリ コードを C# と同じように JavaScript に呼び出すことができます。

詳細については、「<xref:blazor/components/class-libraries>」を参照してください。
