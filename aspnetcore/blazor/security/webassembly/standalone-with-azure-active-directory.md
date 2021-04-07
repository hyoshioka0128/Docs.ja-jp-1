---
title: Azure Active Directory を使用して、ASP.NET Core Blazor WebAssembly スタンドアロン アプリをセキュリティで保護する
author: guardrex
description: Azure Active Directory を使用して、ASP.NET Core Blazor WebAssembly スタンドアロン アプリをセキュリティで保護する方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: devx-track-csharp, mvc
ms.date: 10/27/2020
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
uid: blazor/security/webassembly/standalone-with-azure-active-directory
ms.openlocfilehash: 4f3c07f2698dd68fbb80c5f34a3f3df62225e3b3
ms.sourcegitcommit: 4bbc69f51c59bed1a96aa46f9f5dca2f2a2634cb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/25/2021
ms.locfileid: "105554682"
---
# <a name="secure-an-aspnet-core-blazor-webassembly-standalone-app-with-azure-active-directory"></a>Azure Active Directory を使用して、ASP.NET Core Blazor WebAssembly スタンドアロン アプリをセキュリティで保護する

この記事では、認証用に [Azure Active Directory (AAD)](https://azure.microsoft.com/services/active-directory/) を使用する[スタンドアロン Blazor WebAssembly アプリ](xref:blazor/hosting-models#blazor-webassembly)を作成する方法について説明します。

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> AAD 組織ディレクトリのアカウントをサポートするように構成されている Visual Studio で作成された Blazor WebAssembly アプリの場合、現在、プロジェクトの生成時に Visual Studio によって Azure portal アプリの登録が正しく構成されません。 これは、Visual Studio の将来のリリースで対処される予定です。
>
> この記事では、.NET CLI `dotnet new` コマンドを使用してソリューションと Azure アプリ ポータルの登録を作成する方法と、Azure portal でアプリの登録を手動で作成する方法について説明します。
>
> IDE が更新される前に Visual Studio を使用してソリューションと Azure アプリの登録を作成する場合は、**_この記事の各セクション_** を参照し、Visual Studio によってソリューションが作成された後、アプリの構成とアプリの登録を確認または更新します。

AAD アプリを登録する:

1. Azure portal で **[Azure Active Directory]** に移動します。 サイドバーで **[アプリの登録]** を選択します。 **[新規登録]** ボタンを選択します。
1. アプリの **名前** を指定します (例: **Blazor スタンドアロン AAD**)。
1. **[サポートされているアカウントの種類]** を選択します。 このエクスペリエンスでは、 **[この組織のディレクトリ内のアカウントのみ]** を選択できます。
1. **[リダイレクト URI]** ドロップダウンを **[シングルページ アプリケーション (SPA)]** に設定し、次のリダイレクト URI を指定します: `https://localhost:{PORT}/authentication/login-callback`。 Kestrel で実行されているアプリの既定のポートは 5001 です。 アプリが別の Kestrel ポートで実行されている場合は、アプリのポートを使用します。 IIS Express の場合、アプリのランダムに生成されたポートは、 **[デバッグ]** パネルのアプリのプロパティで確認できます。 この時点ではアプリは存在せず、IIS Express ポートは不明であるため、アプリが作成された後にこの手順に戻り、リダイレクト URI を更新してください。 このトピックの後半で、IIS Express ユーザーにリダイレクト URI を更新するよう促す注意が表示されます。
1. [未確認の発行元ドメイン](/azure/active-directory/develop/howto-configure-publisher-domain)を使用している場合は、 **[アクセス許可]**  >  **[openid と offline_access アクセス許可に対して管理者の同意を付与します]** チェック ボックスをオフにします。 発行元ドメインが検証済みの場合、このチェック ボックスは表示されません。
1. **[登録]** を選択します。

次の情報を記録しておきます。

* アプリケーション (クライアント) ID (例: `41451fa7-82d9-4673-8fa5-69eff5a761fd`)
* ディレクトリ (テナント) ID (例: `e86c78e2-8bb4-4c41-aefd-918e0565a45e`)

**[認証]** > **[プラットフォーム構成]** > **[シングルページ アプリケーション (SPA)]** で次の操作を行います。

1. **[リダイレクト URI]** が`https://localhost:{PORT}/authentication/login-callback` であることを確認します。
1. **[暗黙の付与]** で、 **[アクセス トークン]** と **[ID トークン]** のチェック ボックスが **オフ** になっていることを確認します。
1. アプリの残りの既定値は、このエクスペリエンスで使用可能です。
1. **[保存]** ボタンを選択します。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

AAD アプリを登録する:

1. Azure portal で **[Azure Active Directory]** に移動します。 サイドバーで **[アプリの登録]** を選択します。 **[新規登録]** ボタンを選択します。
1. アプリの **名前** を指定します (例: **Blazor スタンドアロン AAD**)。
1. **[サポートされているアカウントの種類]** を選択します。 このエクスペリエンスでは、 **[この組織のディレクトリ内のアカウントのみ]** を選択できます。
1. **[リダイレクト URI]** ドロップ ダウンの設定を **[Web]** のままとし、次のリダイレクト URI を指定します: `https://localhost:{PORT}/authentication/login-callback`。 Kestrel で実行されているアプリの既定のポートは 5001 です。 アプリが別の Kestrel ポートで実行されている場合は、アプリのポートを使用します。 IIS Express の場合、アプリのランダムに生成されたポートは、 **[デバッグ]** パネルのアプリのプロパティで確認できます。 この時点ではアプリは存在せず、IIS Express ポートは不明であるため、アプリが作成された後にこの手順に戻り、リダイレクト URI を更新してください。 このトピックの後半で、IIS Express ユーザーにリダイレクト URI を更新するよう促す注意が表示されます。
1. [未確認の発行元ドメイン](/azure/active-directory/develop/howto-configure-publisher-domain)を使用している場合は、 **[アクセス許可]**  >  **[openid と offline_access アクセス許可に対して管理者の同意を付与します]** チェック ボックスをオフにします。 発行元ドメインが検証済みの場合、このチェック ボックスは表示されません。
1. **[登録]** を選択します。

次の情報を記録しておきます。

* アプリケーション (クライアント) ID (例: `41451fa7-82d9-4673-8fa5-69eff5a761fd`)
* ディレクトリ (テナント) ID (例: `e86c78e2-8bb4-4c41-aefd-918e0565a45e`)

**[認証]** > **[プラットフォーム構成]** > **[Web]** で次の操作を行います。

1. **[リダイレクト URI]** が`https://localhost:{PORT}/authentication/login-callback` であることを確認します。
1. **[暗黙の付与]** で、 **[アクセス トークン]** と **[ID トークン]** のチェック ボックスをオンにします。
1. アプリの残りの既定値は、このエクスペリエンスで使用可能です。
1. **[保存]** ボタンを選択します。

::: moniker-end

空のフォルダーにアプリを作成します。 次のコマンドのプレースホルダーを、前に記録した情報に置き換え、コマンド シェルでこのコマンドを実行します。

```dotnetcli
dotnet new blazorwasm -au SingleOrg --client-id "{CLIENT ID}" -o {APP NAME} --tenant-id "{TENANT ID}"
```

| プレースホルダー   | Azure portal での名前       | 例                                |
| ------------- | ----------------------- | -------------------------------------- |
| `{APP NAME}`  | &mdash;                 | `BlazorSample`                         |
| `{CLIENT ID}` | アプリケーション (クライアント) ID | `41451fa7-82d9-4673-8fa5-69eff5a761fd` |
| `{TENANT ID}` | ディレクトリ (テナント) ID   | `e86c78e2-8bb4-4c41-aefd-918e0565a45e` |

`-o|--output` オプションで指定した出力場所にプロジェクト フォルダーが存在しない場合は作成されて、アプリの名前の一部になります。

> [!NOTE]
> Azure portal では、アプリのプラットフォーム構成の **[リダイレクト URI]** は、既定の設定の Kestrel サーバーで実行されるアプリの場合、ポート 5001 に構成されます。
>
> アプリがランダム IIS Express ポートで実行されている場合、アプリのポートは **[デバッグ]** パネルのアプリのプロパティで確認できます。
>
> ポートがアプリの既知のポートで事前に構成されていない場合は、Azure portal でアプリの登録に戻り、正しいポートでリダイレクト URI を更新します。

::: moniker range=">= aspnetcore-5.0"

[!INCLUDE[](~/blazor/includes/security/additional-scopes-standalone-AAD.md)]

::: moniker-end

アプリを作成すると、次のことができるようになります。

* AAD ユーザー アカウントを使用してアプリにログインする。
* Microsoft API のアクセス トークンを要求する。 詳細については次を参照してください:
  * [アクセス トークン スコープ](#access-token-scopes)
  * [クイック スタート:Web API を公開するようにアプリケーションを構成する](/azure/active-directory/develop/quickstart-configure-app-expose-web-apis)
  * [AAD によるホスティング: アクセス トークン スコープ (AAD `App ID URI` スコープ形式に関するガイダンスを含む)](xref:blazor/security/webassembly/hosted-with-azure-active-directory#access-token-scopes)
  * [Microsoft Graph API のアクセス トークン スコープ](xref:blazor/security/webassembly/graph-api)

## <a name="authentication-package"></a>認証パッケージ

職場または学校アカウント (`SingleOrg`) を使用するようにアプリを作成すると、アプリは [Microsoft Authentication Library](/azure/active-directory/develop/msal-overview) ([`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal)) のパッケージ参照を自動的に受け取ります。 このパッケージには、アプリでユーザーを認証し、保護された API を呼び出すためのトークンを取得するのに役立つ一連のプリミティブが用意されています。

アプリに認証を追加する場合は、アプリのプロジェクト ファイルにパッケージを手動で追加します。

```xml
<PackageReference Include="Microsoft.Authentication.WebAssembly.Msal" 
  Version="{VERSION}" />
```

プレースホルダー `{VERSION}` では、[NuGet.org](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal) のパッケージの **バージョン履歴** にある、アプリの共有フレームワークのバージョンに一致するパッケージの安定した最新バージョンを確認できます。

[`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal) パッケージによって、[`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) パッケージがアプリに推移的に追加されます。

## <a name="authentication-service-support"></a>認証サービスのサポート

ユーザーの認証に対するサポートは、[`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal) パッケージによって提供される <xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A> 拡張メソッドを使用して、サービス コンテナーに登録されます。 このメソッドでは、アプリが IdentityID プロバイダー (IP) とやり取りするために必要なサービスが設定されます。

`Program.cs`:

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    builder.Configuration.Bind("AzureAd", options.ProviderOptions.Authentication);
});
```

<xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A> メソッドでは、アプリを認証するために必要なパラメーターを構成するためのコールバックを受け入れます。 アプリを構成するために必要な値は、アプリを登録するときに AAD 構成から取得できます。

構成は `wwwroot/appsettings.json` ファイルによって提供されます。

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/{TENANT ID}",
    "ClientId": "{CLIENT ID}",
    "ValidateAuthority": true
  }
}
```

例:

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/e86c78e2-...-918e0565a45e",
    "ClientId": "41451fa7-82d9-4673-8fa5-69eff5a761fd",
    "ValidateAuthority": true
  }
}
```

## <a name="access-token-scopes"></a>アクセス トークン スコープ

Blazor WebAssembly テンプレートでは、セキュリティで保護された API のアクセス トークンを要求するようにアプリが自動的に構成されるわけではありません。 サインイン フローの一部としてアクセス トークンをプロビジョニングするには、<xref:Microsoft.Authentication.WebAssembly.Msal.Models.MsalProviderOptions> の既定のアクセス トークン スコープにスコープを追加します。

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...
    options.ProviderOptions.DefaultAccessTokenScopes.Add("{SCOPE URI}");
});
```

`AdditionalScopesToConsent` を使用して追加のスコープを指定します。

```csharp
options.ProviderOptions.AdditionalScopesToConsent.Add("{ADDITIONAL SCOPE URI}");
```

::: moniker range="< aspnetcore-5.0"

[!INCLUDE[](~/blazor/includes/security/azure-scope-3x.md)]

::: moniker-end

詳細については、"*その他のシナリオ*" に関する記事の次のセクションを参照してください。

* [追加のアクセス トークンを要求する](xref:blazor/security/webassembly/additional-scenarios#request-additional-access-tokens)
* [送信要求にトークンを添付する](xref:blazor/security/webassembly/additional-scenarios#attach-tokens-to-outgoing-requests)

::: moniker range=">= aspnetcore-5.0"

## <a name="login-mode"></a>ログイン モード

[!INCLUDE[](~/blazor/includes/security/msal-login-mode.md)]

::: moniker-end

## <a name="imports-file"></a>インポート ファイル

[!INCLUDE[](~/blazor/includes/security/imports-file-standalone.md)]

## <a name="index-page"></a>インデックス ページ

[!INCLUDE[](~/blazor/includes/security/index-page-msal.md)]

## <a name="app-component"></a>アプリ コンポーネント

[!INCLUDE[](~/blazor/includes/security/app-component.md)]

## <a name="redirecttologin-component"></a>RedirectToLogin コンポーネント

[!INCLUDE[](~/blazor/includes/security/redirecttologin-component.md)]

## <a name="logindisplay-component"></a>LoginDisplay コンポーネント

[!INCLUDE[](~/blazor/includes/security/logindisplay-component.md)]

## <a name="authentication-component"></a>認証コンポーネント

[!INCLUDE[](~/blazor/includes/security/authentication-component.md)]

[!INCLUDE[](~/blazor/includes/security/troubleshoot.md)]

## <a name="additional-resources"></a>その他の技術情報

* <xref:blazor/security/webassembly/additional-scenarios>
* [Authentication.MSAL JavaScript ライブラリのカスタム バージョンをビルドする](xref:blazor/security/webassembly/additional-scenarios#build-a-custom-version-of-the-authenticationmsal-javascript-library)
* [セキュリティで保護された既定のクライアントを使用する、アプリ内の認証または承認されていない Web API 要求](xref:blazor/security/webassembly/additional-scenarios#unauthenticated-or-unauthorized-web-api-requests-in-an-app-with-a-secure-default-client)
* <xref:blazor/security/webassembly/aad-groups-roles>
* <xref:security/authentication/azure-active-directory/index>
* [Microsoft ID プラットフォームのドキュメント](/azure/active-directory/develop/)
