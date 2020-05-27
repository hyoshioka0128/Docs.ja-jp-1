---
title: ASP.NET Core の認証サンプル
author: rick-anderson
description: ASP.NET Core リポジトリの認証サンプルへのリンクを提供します。
ms.author: riande
ms.date: 01/31/2019
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/authentication/samples
ms.openlocfilehash: 7cd0fe60d7917abda7d8ac0e071deca13a4136ce
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/04/2020
ms.locfileid: "82776553"
---
# <a name="authentication-samples-for-aspnet-core"></a>ASP.NET Core の認証サンプル

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

[ASP.NET Core リポジトリ](https://github.com/dotnet/AspNetCore)には、 *AspNetCore/src/Security/samples*フォルダーに次の認証サンプルが含まれています。

* [要求の変換](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/ClaimsTransformation)
* [Cookie 認証](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/Cookies)
* [カスタムポリシープロバイダー-IAuthorizationPolicyProvider](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/CustomPolicyProvider)
* [動的な認証スキームとオプション](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/DynamicSchemes)
* [外部要求](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/Identity.ExternalClaims)
* [要求に基づいて cookie と別の認証スキームのどちらを選択するか](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/PathSchemeSelection)
* [静的ファイルへのアクセスを制限する](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/StaticFilesAuth)

## <a name="run-the-samples"></a>サンプルを実行する

* [ブランチ](https://github.com/dotnet/AspNetCore)を選択してください。 たとえば、`release/3.1` のように指定します。
* [ASP.NET Core リポジトリ](https://github.com/dotnet/AspNetCore)をクローンまたはダウンロードします。
* ASP.NET Core リポジトリのクローンに一致する[.NET Core SDK](https://dotnet.microsoft.com/download/dotnet-core)バージョンがインストールされていることを確認します。
* *AspNetCore/src/Security/samples*のサンプルに移動し、で`dotnet run`サンプルを実行します。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[ASP.NET Core リポジトリ](https://github.com/dotnet/AspNetCore)には、 *AspNetCore/src/Security/samples*フォルダーに次の認証サンプルが含まれています。

* [要求の変換](https://github.com/dotnet/AspNetCore/tree/release/2.2/src/Security/samples/ClaimsTransformation)
* [Cookie 認証](https://github.com/dotnet/AspNetCore/tree/release/2.2/src/Security/samples/Cookies)
* [カスタムポリシープロバイダー-IAuthorizationPolicyProvider](https://github.com/dotnet/AspNetCore/tree/release/2.2/src/Security/samples/CustomPolicyProvider)
* [動的な認証スキームとオプション](https://github.com/dotnet/AspNetCore/tree/release/2.2/src/Security/samples/DynamicSchemes)
* [外部要求](https://github.com/dotnet/AspNetCore/tree/release/2.2/src/Security/samples/Identity.ExternalClaims)
* [要求に基づいて cookie と別の認証スキームのどちらを選択するか](https://github.com/dotnet/AspNetCore/tree/release/2.2/src/Security/samples/PathSchemeSelection)
* [静的ファイルへのアクセスを制限する](https://github.com/dotnet/AspNetCore/tree/release/2.2/src/Security/samples/StaticFilesAuth)

## <a name="run-the-samples"></a>サンプルを実行する

* [ブランチ](https://github.com/dotnet/AspNetCore)を選択してください。 たとえば、`release/2.2` のように指定します。
* [ASP.NET Core リポジトリ](https://github.com/dotnet/AspNetCore)をクローンまたはダウンロードします。
* ASP.NET Core リポジトリのクローンに一致する[.NET Core SDK](https://dotnet.microsoft.com/download/dotnet-core)バージョンがインストールされていることを確認します。
* *AspNetCore/src/Security/samples*のサンプルに移動し、で`dotnet run`サンプルを実行します。

::: moniker-end
