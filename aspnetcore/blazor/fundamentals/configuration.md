---
title: ASP.NET Core Blazor の構成
author: guardrex
description: アプリの設定、認証、ログの構成など、Blazor アプリの構成について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/10/2020
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
uid: blazor/fundamentals/configuration
ms.openlocfilehash: fd2a9784e92b132d679b2065b37cb37b7dd89cde
ms.sourcegitcommit: 07e7ee573fe4e12be93249a385db745d714ff6ae
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/12/2021
ms.locfileid: "103413406"
---
# <a name="aspnet-core-blazor-configuration"></a>ASP.NET Core Blazor の構成

> [!NOTE]
> このトピックの対象は、Blazor WebAssembly です。 ASP.NET Core アプリの構成に関する一般的なガイダンスについては、「<xref:fundamentals/configuration/index>」を参照してください。

Blazor WebAssembly を使用すると、既定で次のアプリ設定ファイルから構成が読み込まれます。

* `wwwroot/appsettings.json`.
* `wwwroot/appsettings.{ENVIRONMENT}.json`。ここで、`{ENVIRONMENT}` プレースホルダーはアプリの[ランタイム環境](xref:fundamentals/environments)です。

アプリによって登録されたその他の構成プロバイダーで構成を指定することもできますが、すべてのプロバイダーやプロバイダーの機能が Blazor WebAssembly アプリに適しているわけではありません。

* [Azure Key Vault 構成プロバイダー](xref:security/key-vault-configuration): クライアント シークレットのシナリオでは、プロバイダーはマネージド ID およびアプリケーション ID (クライアント ID) に対してはサポートされていません。 クライアント シークレットを使用したアプリケーション ID は、ASP.NET Core アプリ、特に Blazor WebAssembly アプリでは推奨されません。これは、クライアント シークレットを Azure Key Vault サービスにアクセスするためにクライアント側ではセキュリティで保護できないためです。
* [Azure App Configuration プロバイダー](/azure/azure-app-configuration/quickstart-aspnet-core-app): このプロバイダーは Blazor WebAssembly アプリに適していません。これは、Blazor WebAssembly アプリは Azure 内のサーバー上では実行されないためです。

> [!WARNING]
> Blazor WebAssembly アプリでの構成は、ユーザーに表示されます。 **アプリのシークレット、資格情報、その他の機微なデータを Blazor WebAssembly アプリの構成に格納しないでください。**

構成プロバイダーの詳細については、「<xref:fundamentals/configuration/index>」を参照してください。

## <a name="app-settings-configuration"></a>アプリ設定の構成

アプリ設定ファイルの構成は既定で読み込まれます。 次の例では、UI 構成値がアプリ設定ファイルに格納され、Blazor フレームワークによって自動的に読み込まれます。 その値はコンポーネントによって読み取られます。

`wwwroot/appsettings.json`:

```json
{
  "h1FontSize": "50px"
}
```

構成データにアクセスするために、コンポーネントに <xref:Microsoft.Extensions.Configuration.IConfiguration> インスタンスを挿入します。

`Pages/ConfigurationExample.razor`:

```razor
@page "/configuration-example"
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration

<h1 style="font-size:@Configuration["h1FontSize"]">
    Configuration example
</h1>
```

その他の構成ファイルを `wwwroot` フォルダーから構成に読み取るには、<xref:System.Net.Http.HttpClient> を使用してファイルの内容を取得します。 次の例では、構成ファイル (`cars.json`) をアプリの構成に読み取ります。

`wwwroot/cars.json`:

```json
{
    "size": "tiny"
}
```

`Program.cs` に <xref:Microsoft.Extensions.Configuration?displayProperty=fullName> の名前空間を追加します。

```csharp
using Microsoft.Extensions.Configuration;
```

`Program.cs` の `Program.Main` で、クライアントを使用してファイルを読み取るように、既存の <xref:System.Net.Http.HttpClient> サービス登録を変更します。

```csharp
var http = new HttpClient()
{
    BaseAddress = new Uri(builder.HostEnvironment.BaseAddress)
};

builder.Services.AddScoped(sp => http);

using var response = await http.GetAsync("cars.json");
using var stream = await response.Content.ReadAsStreamAsync();

builder.Configuration.AddJsonStream(stream);
```

## <a name="memory-configuration-source"></a>メモリ構成のソース

次の例では、`Program.Main` で <xref:Microsoft.Extensions.Configuration.Memory.MemoryConfigurationSource> を使用して追加の構成を指定します。

`Program.cs` に <xref:Microsoft.Extensions.Configuration.Memory?displayProperty=fullName> の名前空間を追加します。

```csharp
using Microsoft.Extensions.Configuration.Memory;
```

`Program.cs` の `Program.Main`:

```csharp
var vehicleData = new Dictionary<string, string>()
{
    { "color", "blue" },
    { "type", "car" },
    { "wheels:count", "3" },
    { "wheels:brand", "Blazin" },
    { "wheels:brand:type", "rally" },
    { "wheels:year", "2008" },
};

var memoryConfig = new MemoryConfigurationSource { InitialData = vehicleData };

builder.Configuration.Add(memoryConfig);
```

構成データにアクセスするために、コンポーネントに <xref:Microsoft.Extensions.Configuration.IConfiguration> インスタンスを挿入します。

`Pages/MemoryConfig.razor`:

```razor
@page "/memory-config"
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration

<h1>Memory configuration example</h1>

<h2>General specifications</h2>

<ul>
    <li>Color: @Configuration["color"]</li>
    <li>Type: @Configuration["type"]</li>
</ul>

<h2>Wheels</h2>

<ul>
    <li>Count: @Configuration["wheels:count"]</li>
    <li>Brand: @Configuration["wheels:brand"]</li>
    <li>Type: @Configuration["wheels:brand:type"]</li>
    <li>Year: @Configuration["wheels:year"]</li>
</ul>
```

<xref:Microsoft.Extensions.Configuration.IConfiguration.GetSection%2A?displayProperty=nameWithType>で C# コード内の構成セクションを取得します。 次の例では、前の例の構成の `wheels` セクションを取得します。

```razor
@code {
    protected override void OnInitialized()
    {
        var wheelsSection = Configuration.GetSection("wheels");

        ...
    }
}
```

## <a name="authentication-configuration"></a>認証の構成

アプリ設定ファイルで認証構成を指定します。

`wwwroot/appsettings.json`:

```json
{
  "Local": {
    "Authority": "{AUTHORITY}",
    "ClientId": "{CLIENT ID}"
  }
}
```

`Program.Main` で <xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Bind%2A?displayProperty=nameWithType> を使用して、Identity プロバイダーの構成を読み込みます。 次の例では、[OIDC プロバイダー](xref:blazor/security/webassembly/standalone-with-authentication-library)の構成を読み込みます。

`Program.cs`:

```csharp
builder.Services.AddOidcAuthentication(options =>
    builder.Configuration.Bind("Local", options.ProviderOptions));
```

## <a name="logging-configuration"></a>ログの構成

アプリのプロジェクト ファイルに、[`Microsoft.Extensions.Logging.Configuration`](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Configuration) のパッケージ参照を追加します。

```xml
<PackageReference Include="Microsoft.Extensions.Logging.Configuration" Version="{VERION}" />
```

前の例では、プレースホルダー `{VERSION}` はパッケージのバージョンです。 パッケージのバージョンについては [NuGet.org](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Configuration) を参照してください。

アプリ設定ファイルで、ログ構成を指定します。 ログ構成は `Program.Main` に読み込まれます。

`wwwroot/appsettings.json`:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  }
}
```

`Program.cs` に <xref:Microsoft.Extensions.Logging?displayProperty=fullName> の名前空間を追加します。

```csharp
using Microsoft.Extensions.Logging;
```

`Program.cs` の `Program.Main`:

```csharp
builder.Logging.AddConfiguration(
    builder.Configuration.GetSection("Logging"));
```

## <a name="host-builder-configuration"></a>ホスト ビルダーの構成

`Program.Main` で <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder.Configuration?displayProperty=nameWithType> のホスト ビルダーの構成を読み取ります。

`Program.cs` の `Program.Main`:

```csharp
var hostname = builder.Configuration["HostName"];
```

## <a name="cached-configuration"></a>キャッシュされた構成

構成ファイルは、オフラインで使用できるようにキャッシュされます。 [プログレッシブ Web アプリケーション (PWA)](xref:blazor/progressive-web-app) では、新しい展開を作成するときにのみ構成ファイルを更新できます。 次の理由により、展開間で構成ファイルを編集しても意味がありません。

* ユーザーには、引き続き使用するファイルのキャッシュされたバージョンがあります。
* PWA の `service-worker.js` と `service-worker-assets.js` のファイルは、コンパイル時に再構築される必要があります。これにより、ユーザーの次回のオンライン アクセス時に、アプリが再展開されたことが通知されます。

PWA によるバックグラウンド更新の処理方法の詳細については、「<xref:blazor/progressive-web-app#background-updates>」を参照してください。
