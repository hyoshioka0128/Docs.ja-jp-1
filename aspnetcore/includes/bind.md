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
ms.openlocfilehash: 30baab0649268f4abf0dbd6c99dfeef3f43d0054
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100536303"
---
関連する構成値を読み取る方法としては、[オプション パターン](xref:fundamentals/configuration/options)を使用することをお勧めします。 たとえば、以下の構成値を読み取るには、次のようにします:

```json
  "Position": {
    "Title": "Editor",
    "Name": "Joe Smith"
  }
```

次の `PositionOptions` クラスを作成します:

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Options/PositionOptions.cs?name=snippet)]

オプション クラス:

* パラメーターのないパブリック コンストラクターを持った非抽象でなければなりません。
* 型のパブリックな読み取り/書き込みプロパティは、すべてバインドされます。
* フィールドはバインド ***されません***。 上のコード `Position` はバインドされません。 `Position` プロパティは、クラスを構成プロバイダーにバインドするときに、アプリで文字列 `"Position"` をハードコーディングする必要をなくすために使用されます。

コード例を次に示します。

* [ConfigurationBinder.Bind](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Bind*) を呼び出して、`PositionOptions` クラスを `Position` セクションにバインドします。
* `Position` 構成データを表示します。

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Pages/Test22.cshtml.cs?name=snippet)]

上のコードでは、アプリが開始された後の JSON 構成ファイルへの変更が既定で読み取られます。

[`ConfigurationBinder.Get<T>`](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Get*) 指定された型をバインドして返します。 `ConfigurationBinder.Get<T>` は `ConfigurationBinder.Bind` を使用するよりも便利な場合があります。 次のコードは、`PositionOptions` クラスで `ConfigurationBinder.Get<T>` を使用する方法を示しています:

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Pages/Test21.cshtml.cs?name=snippet)]

上のコードでは、アプリが開始された後の JSON 構成ファイルへの変更が既定で読み取られます。

***オプション パターン** _ を使用する際の別の方法として、`Position` セクションをバインドし、[依存関係挿入サービスコンテナー](xref:fundamentals/dependency-injection)に追加する方法があります。 次のコードでは、`PositionOptions` は <xref:Microsoft.Extensions.DependencyInjection.OptionsConfigurationServiceCollectionExtensions.Configure_> でサービスコンテナーに追加され、構成にバインドされます:

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Startup.cs?name=snippet)]

下記のコードは、上記のコードを使用して位置オプションを読み取ります:

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Pages/Test2.cshtml.cs?name=snippet)]

上のコードでは、アプリが開始された後の JSON 構成ファイルへの変更は読み取られ ***ません***。 アプリの開始後に変更を読み取るには、[IOptionsSnapshot](xref:fundamentals/configuration/options#ios) を使用します。
