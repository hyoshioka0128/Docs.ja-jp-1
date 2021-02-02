---
title: ASP.NET Core Blazor WebAssembly をセキュリティで保護する
author: guardrex
description: シングル ページ アプリケーション (SPA) として Blazor WebAssemlby アプリをセキュリティで保護する方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
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
uid: blazor/security/webassembly/index
ms.openlocfilehash: 0b555ad7befe882c4ffd06e2505a9edc1263eee2
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057084"
---
# <a name="secure-aspnet-core-blazor-webassembly"></a>ASP.NET Core Blazor WebAssembly をセキュリティで保護する

作成者: [Javier Calvarro Jeannine](https://github.com/javiercn)

Blazor WebAssembly アプリは、シングル ページ アプリケーション (SPA) と同じ方法でセキュリティ保護されます。 ユーザーを SPA で認証する方法はいくつかありますが、最も一般的で包括的な方法は、[OpenID Connect (OIDC)](https://openid.net/connect/) などの [OAuth 2.0 プロトコル](https://oauth.net/)に基づく実装を使用することです。

## <a name="authentication-library"></a>認証ライブラリ

Blazor WebAssembly では、[`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) ライブラリを介して OIDC を使用したアプリの認証と承認がサポートされています。 ライブラリには、ASP.NET Core バックエンドに対してシームレスに認証を行うための一連のプリミティブが用意されています。 ライブラリによって ASP.NET Core Identity が、[Identity Server](https://identityserver.io/) 上に構築された API 認可サポートと統合されます。 このライブラリは、OpenID Provider (OP) と呼ばれる OIDC をサポートするすべてのサードパーティ Identity プロバイダー (IP) に対して認証できます。

Blazor WebAssembly の認証のサポートは、基になる認証プロトコルの詳細を処理するために使用される、`oidc-client.js` ライブラリに基づいて構築されています。

SameSite cookie の使用など、SPA を認証するためのその他のオプションも存在します。 ただし、Blazor WebAssembly のエンジニアリング設計は、Blazor WebAssembly アプリでの認証に最適なオプションとして OAuth および OIDC にも採用されています。 [JSON Web トークン (JWT)](https://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) に基づく[トークンベースの認証](xref:security/anti-request-forgery#token-based-authentication)は、機能とセキュリティ上の理由により、[cookie ベースの認証](xref:security/anti-request-forgery#cookie-based-authentication)に基づいて選択されています。

* トークンベースのプロトコルを使用すると、トークンがすべての要求で送信されないため、攻撃対象領域が小さくなります。
* サーバー エンドポイントでは、トークンが明示的に送信されるため、[クロスサイト リクエスト フォージェリ (CSRF)](xref:security/anti-request-forgery) に対する保護が必要ありません。 これにより、MVC または Razor ページ アプリと共に、Blazor WebAssembly アプリをホストすることができます。
* トークンの権限は cookie よりも狭くなります。 たとえば、該当する機能が明示的に実装されていない限り、トークンを使用してユーザー アカウントを管理したり、ユーザーのパスワードを変更したりできません。
* トークンの有効期間は短く、既定では 1 時間です。これにより、攻撃時間が制限されます。 トークンは、いつでも取り消すことができます。
* 自己完結型の JWT は、クライアントとサーバーの認証プロセスを保証します。 たとえば、クライアントには、受信したトークンが正当であり、指定した認証プロセスの一環として出力されたことを検出して検証する手段があります。 サード パーティが認証プロセスの途中でトークンを切り替えようとすると、クライアントは切り替えられたトークンを検出して使用しないようにすることができます。
* OAuth および OIDC を使用したトークンは、アプリが安全であることを確認するために、正しく動作しているユーザー エージェントに依存しません。
* OAuth や OIDC などのトークンベースのプロトコルでは、同じセキュリティ特性のセットを使用して、ホストされたアプリとスタンドアロンのアプリの認証と認可を行うことができます。

## <a name="authentication-process-with-oidc"></a>OIDC を使用した認証プロセス

[`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) ライブラリには、OIDC を使用した認証と認可を実装するためのいくつかのプリミティブが用意されています。 大まかにいうと、認証は次のようにして行われます。

* 匿名ユーザーがログイン ボタンを選択する、または [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) 属性が適用されたページを要求すると、そのユーザーがアプリのログイン ページ (`/authentication/login`) にリダイレクトされます。
* ログイン ページで、認証ライブラリが承認エンドポイントへのリダイレクトを準備します。 承認エンドポイントは、Blazor WebAssembly アプリの外部にあり、別のオリジンでホストすることができます。 エンドポイントは、ユーザーが認証されているかどうかを判断し、応答として 1 つ以上のトークンを発行する役割を担います。 認証ライブラリは、認証応答を受信するためのログイン コールバックを提供します。
  * ユーザーが認証されていない場合、そのユーザーは基になる認証システム (通常は ASP.NET Core Identity) にリダイレクトされます。
  * ユーザーが既に認証されている場合、承認エンドポイントは適切なトークンを生成し、ブラウザーをログイン コールバック エンドポイント (`/authentication/login-callback`) にリダイレクトします。
* Blazor WebAssembly アプリでログイン コールバック エンドポイント (`/authentication/login-callback`) が読み込まれると、認証応答が処理されます。
  * 認証プロセスが正常に完了した場合、ユーザーは認証され、必要に応じてユーザーが要求した元の保護された URL に戻されます。
  * 何らかの理由で認証プロセスが失敗した場合、ユーザーはログイン失敗ページ (`/authentication/login-failed`) に送信され、エラーが表示されます。

## <a name="authentication-component"></a>`Authentication` コンポーネント

`Authentication` コンポーネント (`Pages/Authentication.razor`) はリモート認証操作を処理し、次のことをアプリに許可します。

* 認証状態のアプリのルートを構成する。
* 認証状態の UI コンテンツを設定する。
* 認証状態を管理する。

ユーザーの登録やサインインなどの認証アクションは、Blazor フレームワークの <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorViewCore%601> コンポーネントに渡されます。ここでは認証操作全体で状態が保持され、制御されます。

使用例を含む詳細については、「<xref:blazor/security/webassembly/additional-scenarios>」を参照してください。

## <a name="authorization"></a>承認

Blazor WebAssembly アプリでは、すべてのクライアント側コードがユーザーによって変更される可能性があるため、承認チェックがバイパスされる可能性があります。 JavaScript SPA フレームワークや任意のオペレーティング システム用のネイティブ アプリを含め、すべてのクライアント側アプリのテクノロジにも同じことが当てはまります。

**常に、クライアント側アプリからアクセスされるすべての API エンドポイント内のサーバー上で承認チェックを実行します。**

## <a name="require-authorization-for-the-entire-app"></a>アプリ全体での承認を要求する

次のいずれかの方法を使用して、[`[Authorize]` 属性](xref:blazor/security/index#authorize-attribute) ([API ドキュメント](xref:System.Web.Mvc.AuthorizeAttribute)) をアプリの各 Razor コンポーネントに適用します。

* `_Imports.razor` ファイルの [`@attribute`](xref:mvc/views/razor#attribute) ディレクティブを使用します。

  ```razor
  @using Microsoft.AspNetCore.Authorization
  @attribute [Authorize]
  ```

* `Pages` フォルダー内の各 Razor コンポーネントに属性を追加します。

> [!NOTE]
> <xref:Microsoft.AspNetCore.Authorization.AuthorizationPolicyBuilder.RequireAuthenticatedUser%2A> を持つポリシーに <xref:Microsoft.AspNetCore.Authorization.AuthorizationOptions.FallbackPolicy?displayProperty=nameWithType> を設定することはサポートされて **いません**。

## <a name="refresh-tokens"></a>更新トークン

更新トークンは、Blazor WebAssembly アプリのクライアント側でセキュリティで保護することはできません。 そのため、更新トークンを直接使用するためにアプリに送信するべきではありません。

更新トークンは、ホストされている Blazor WebAssembly ソリューションのサーバー側アプリで、サード パーティの API にアクセスするために保持および使用することができます。 詳細については、「<xref:blazor/security/webassembly/additional-scenarios#authenticate-users-with-a-third-party-provider-and-call-protected-apis-on-the-host-server-and-the-third-party>」を参照してください。

## <a name="establish-claims-for-users"></a>ユーザーに対するクレームを確立する

アプリでは、サーバーへの Web API 呼び出しに基づくユーザーへのクレームが必要になることがよくあります。 たとえば、クレームは、アプリで[承認を確立する](xref:blazor/security/index#authorization)ためによく使用されます。 このようなシナリオにおいて、アプリでは、サービスにアクセスするためのアクセス トークンを要求し、そのトークンを使用してクレームに対するユーザー データを取得します。 例については、次のリソースを参照してください。

* [その他のシナリオ: ユーザーをカスタマイズする](xref:blazor/security/webassembly/additional-scenarios#customize-the-user)
* <xref:blazor/security/webassembly/aad-groups-roles>

## <a name="azure-app-service-on-linux-with-identity-server"></a>Identity Server を使用した Azure App Service on Linux

Identity Server を使用して Azure App Service on Linux にデプロイするときに、発行者を明示的に指定します。 詳細については、「<xref:security/authentication/identity/spa#azure-app-service-on-linux>」を参照してください。

## <a name="windows-authentication"></a>Windows 認証

Blazor WebAssembly またはその他の SPA フレームワークでは、Windows 認証を使用しないことをお勧めします。 Windows 認証の代わりにトークンベースのプロトコルを使用することをお勧めします (Active Directory フェデレーション サービス (AD FS) を使用する OIDC など)。

Windows 認証が Blazor WebAssembly またはその他の SPA フレームワークで使用されている場合、クロスサイト リクエスト フォージェリ (CSRF) トークンからアプリを保護するために追加の手段が必要になります。 cookie に当てはまるのと同じ問題が Windows 認証にも当てはまります。これに加えて、Windows 認証では、オリジン間での認証コンテキストの共有を防ぐメカニズムは提供されません。 CSRF からの追加の保護なしで Windows 認証を使用するアプリは、少なくとも組織のイントラネットに限定する必要があり、インターネットでは使用できません。

詳細については、「<xref:security/anti-request-forgery>」を参照してください。

## <a name="implementation-guidance"></a>実装ガイダンス

この「*概要*」の記事では、特定のプロバイダーに対して Blazor WebAssembly アプリのユーザーを認証する方法について説明します。

スタンドアロン Blazor WebAssembly アプリ:

* [OIDC プロバイダーと WebAssembly 認証ライブラリ向けの一般的なガイダンス](xref:blazor/security/webassembly/standalone-with-authentication-library)
* [Microsoft アカウント](xref:blazor/security/webassembly/standalone-with-microsoft-accounts)
* [Azure Active Directory (AAD)](xref:blazor/security/webassembly/standalone-with-azure-active-directory)
* [Azure Active Directory (AAD) B2C](xref:blazor/security/webassembly/standalone-with-azure-active-directory-b2c)

ホストされている Blazor WebAssembly アプリ:

* [Azure Active Directory (AAD)](xref:blazor/security/webassembly/hosted-with-azure-active-directory)
* [Azure Active Directory (AAD) B2C](xref:blazor/security/webassembly/hosted-with-azure-active-directory-b2c)
* [Identity サーバー](xref:blazor/security/webassembly/hosted-with-identity-server)

::: moniker range=">= aspnetcore-5.0"

構成に関するその他のガイダンスについては、次の記事を参照してください。

* <xref:blazor/security/webassembly/additional-scenarios>
* <xref:blazor/security/webassembly/graph-api>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

構成に関するその他のガイダンスについては、「<xref:blazor/security/webassembly/additional-scenarios>」をご覧ください。

::: moniker-end

## <a name="additional-resources"></a>その他のリソース

* <xref:host-and-deploy/proxy-load-balancer>: 次のガイダンスが含まれます。
  * 転送されたヘッダー ミドルウェアを使用した、プロキシ サーバーと内部ネットワークの間での HTTPS スキーム情報の保持。
  * 追加のシナリオとユース ケース (手動スキーム構成を含む)、正しい要求ルーティングのための要求パスの変更、Linux および IIS 以外のリバース プロキシのための要求スキームの転送。
