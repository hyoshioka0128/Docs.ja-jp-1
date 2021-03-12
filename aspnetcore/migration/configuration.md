---
title: 構成を ASP.NET Core に移行する
author: ardalis
description: ASP.NET MVC プロジェクトから ASP.NET Core MVC プロジェクトに構成を移行する方法について説明します。
ms.author: riande
ms.date: 10/14/2016
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
uid: migration/configuration
ms.openlocfilehash: c3957bf45dddcead24f7bb0f2702bf1a08950bdd
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587945"
---
# <a name="migrate-configuration-to-aspnet-core"></a>構成を ASP.NET Core に移行する

作成者: [Steve Smith](https://ardalis.com/)、[Scott Addie](https://scottaddie.com)

前の記事では、 [ASP.NET mvc プロジェクトの ASP.NET CORE mvc への移行](xref:migration/mvc)を開始しました。 この記事では、構成を移行します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/migration/configuration/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="setup-configuration"></a>セットアップの構成

ASP.NET Core は、以前のバージョンの ASP.NET が使用されていた *global.asax および* *web.config* ファイルを使用しなくなりました。 以前のバージョンの ASP.NET では、アプリケーションのスタートアップロジックは global.asax 内のメソッドに配置されていま `Application_StartUp` した。  その後、ASP.NET MVC では、 *Startup.cs* ファイルがプロジェクトのルートに含まれていました。また、アプリケーションの起動時に呼び出されました。 ASP.NET Core は、すべてのスタートアップロジックを *Startup.cs* ファイルに配置することによって、この方法を完全に採用しています。

*web.config* ファイルも ASP.NET Core で置き換えられました。 *Startup.cs* で説明されているアプリケーションのスタートアップ手順の一部として、構成自体を構成できるようになりました。 構成では引き続き XML ファイルを利用できますが、通常 ASP.NET Core プロジェクトは、などの JSON 形式のファイルに構成値を配置し *appsettings.json* ます。 ASP.NET Core の構成システムは、環境変数に簡単にアクセスすることもできます。これにより、環境固有の値に対して [より安全かつ堅牢な場所](xref:security/app-secrets) を提供できます。 これは、ソース管理にチェックインしない接続文字列や API キーなどのシークレットに特に当てはまります。 ASP.NET Core の構成の詳細については、「 [構成](xref:fundamentals/configuration/index) 」を参照してください。

この記事では、 [前の記事](xref:migration/mvc)の部分的に移行された ASP.NET Core プロジェクトから始めます。 構成を設定するには、次のコンストラクターとプロパティをプロジェクトのルートにある *Startup.cs* ファイルに追加します。

[!code-csharp[](configuration/samples/WebApp1/src/WebApp1/Startup.cs?range=11-16)]

この時点で、 *Startup.cs* ファイルはコンパイルされません。ただし、次のステートメントを追加する必要があるためです `using` 。

```csharp
using Microsoft.Extensions.Configuration;
```

*appsettings.json* 適切な項目テンプレートを使用して、プロジェクトのルートにファイルを追加します。

![AppSettings JSON の追加](configuration/_static/add-appsettings-json.png)

## <a name="migrate-configuration-settings-from-webconfig"></a>web.config から構成設定を移行する

ASP.NET MVC プロジェクトには、要素内の *web.config* に必要なデータベース接続文字列が含まれていました `<connectionStrings>` 。 この ASP.NET Core プロジェクトでは、この情報をファイルに格納し *appsettings.json* ます。 *appsettings.json* を開き、次のものが既に含まれていることに注意してください。

[!code-json[](../migration/configuration/samples/WebApp1/src/WebApp1/appsettings.json?highlight=4)]

上の強調表示された行で、データベースの名前を **_CHANGE_ME** からデータベースの名前に変更します。

## <a name="summary"></a>まとめ

ASP.NET Core によって、アプリケーションのすべてのスタートアップロジックが1つのファイルに配置されます。このファイルで、必要なサービスと依存関係を定義して構成することができます。 *web.config* ファイルは、JSON などのさまざまなファイル形式や環境変数を利用できる柔軟な構成機能に置き換えられます。
