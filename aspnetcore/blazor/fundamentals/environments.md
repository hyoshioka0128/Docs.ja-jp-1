---
title: ASP.NET Core Blazor の環境
author: guardrex
description: Blazor WebAssembly アプリの環境を設定する方法など、Blazor の環境について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/11/2020
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
uid: blazor/fundamentals/environments
ms.openlocfilehash: a5ead59e467da331b585e8daefb1d7d259c7edba
ms.sourcegitcommit: 422e8444b9f5cedc373be5efe8032822db54fcaf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/18/2021
ms.locfileid: "101101049"
---
# <a name="aspnet-core-blazor-environments"></a>ASP.NET Core Blazor の環境

> [!NOTE]
> このトピックの対象は、Blazor WebAssembly です。 Blazor Server アプリに使用する方法について説明している ASP.NET Core アプリの構成に関する一般的なガイダンスについては、「<xref:fundamentals/environments>」を参照してください。

アプリをローカルで実行する場合、環境は既定で開発に設定されます。 アプリが発行されると、環境は既定で実稼働になります。

ホストされている Blazor WebAssembly ソリューションのクライアント側 Blazor アプリ ( *`Client`* ) によって、環境をブラウザーに伝えるミドルウェアを介するソリューションの *`Server`* アプリからの環境が決まります。 *`Server`* アプリによって、ヘッダーの値として "environment" (環境) を含む `blazor-environment` という名前のヘッダーが追加されます。 *`Client`* アプリによってそのヘッダーが読み取られます。 ソリューションの *`Server`* アプリは ASP.NET Core アプリであるため、環境の構成方法の詳細については「<xref:fundamentals/environments>」を参照してください。

ローカルで実行されているスタンドアロン Blazor WebAssembly アプリの場合、開発サーバーによって `blazor-environment` ヘッダーが追加され、開発環境が指定されます。 他のホスト環境の環境を指定するには、`blazor-environment` ヘッダーを追加します。

次の IIS の例では、発行された `web.config` ファイルにカスタム ヘッダー (`blazor-environment`) が追加されます。 `web.config` ファイルは `bin/Release/{TARGET FRAMEWORK}/publish` フォルダー内にあります。プレースホルダー `{TARGET FRAMEWORK}` はターゲット フレームワークです。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>

    ...

    <httpProtocol>
      <customHeaders>
        <add name="blazor-environment" value="Staging" />
      </customHeaders>
    </httpProtocol>
  </system.webServer>
</configuration>
```

> [!NOTE]
> アプリが `publish` フォルダーに発行されたときに上書きされない IIS 用のカスタム `web.config` ファイルを使用するには、「<xref:blazor/host-and-deploy/webassembly#use-a-custom-webconfig>」を参照してください。

<xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment> を挿入し、<xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.Environment> プロパティを読み取ることで、コンポーネント内のアプリの環境を取得します。

`Pages/ReadEnvironment.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/environments/ReadEnvironment.razor?highlight=3,7)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/environments/ReadEnvironment.razor?highlight=3,7)]

::: moniker-end

起動時に、<xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder> によって <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment> が <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder.HostEnvironment> プロパティを介して公開されます。これにより、ホスト ビルダー コードで環境固有のロジックが有効になります。

`Program.cs` の `Program.Main` で:

```csharp
if (builder.HostEnvironment.Environment == "Custom")
{
    ...
};
```

<xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions> を通じて提供された次の便利な拡張メソッドを使用すると、現在の環境で開発、運用、ステージング、およびカスタムの環境名を確認できます。

* <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions.IsDevelopment%2A>
* <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions.IsProduction%2A>
* <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions.IsStaging%2A>
* <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions.IsEnvironment%2A>

`Program.cs` の `Program.Main` で:

```csharp
if (builder.HostEnvironment.IsStaging())
{
    ...
};

if (builder.HostEnvironment.IsEnvironment("Custom"))
{
    ...
};
```

<xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> プロパティは、<xref:Microsoft.AspNetCore.Components.NavigationManager> サービスを利用できないときの起動時に使用できます。

## <a name="additional-resources"></a>その他の技術情報

* <xref:fundamentals/environments>
