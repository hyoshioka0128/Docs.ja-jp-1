---
title: ASP.NET Core の認証サンプル
author: rick-anderson
description: ASP.NET Core リポジトリの認証サンプルへのリンクを提供します。
ms.author: riande
ms.date: 02/21/2021
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
uid: security/authentication/samples
ms.openlocfilehash: e7fb2ac32f57cf4ecd3c5db294bd0df8e186b6c6
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102110119"
---
# <a name="authentication-samples-for-aspnet-core"></a>ASP.NET Core の認証サンプル

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

[ASP.NET Core リポジトリ](https://github.com/dotnet/aspnetcore)には、次の認証サンプル (分岐) が含まれてい `main` ます。

* [要求の変換](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/ClaimsTransformation)
* [Cookie 認証](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/Cookies)
* [カスタム承認エラー応答](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/CustomAuthorizationFailureResponse)
* [カスタムポリシープロバイダー-IAuthorizationPolicyProvider](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/CustomPolicyProvider)
* [動的な認証スキームとオプション](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/DynamicSchemes)
* [外部要求](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/Identity.ExternalClaims)
* [cookie要求に基づいて、別の認証スキームを選択する](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/PathSchemeSelection)
* [静的ファイルへのアクセスを制限する](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/StaticFilesAuth)

## <a name="obtain-and-run-the-samples"></a>サンプルを入手して実行する

この記事に記載されているサンプルリンクは、ASP.NET Core の今後のリリースのサンプルを提供します。 現在のリリースまたは以前のリリースのサンプルを取得するには、次の手順を実行します。

* [ASP.NET Core リポジトリ](https://github.com/dotnet/aspnetcore)のリリースブランチを選択します () https://github.com/dotnet/aspnetcore) 。 たとえば、分岐には `release/5.0` ASP.NET Core 5.0 リリースのサンプルが含まれています。
* ASP.NET Core リポジトリを複製またはダウンロードします。
* ローカルシステムで、ASP.NET Core リポジトリの複製に一致する [.NET Core SDK](https://dotnet.microsoft.com/download/dotnet-core) バージョンがインストールされていることを確認します。
* フォルダー内のサンプルに移動 `aspnetcore/src/Security/samples` し、 [ `dotnet run` コマンド](/dotnet/core/tools/dotnet-run)を使用してサンプルを実行します。
