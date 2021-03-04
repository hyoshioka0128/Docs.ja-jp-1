---
title: ASP.NET Core での Google 外部ログインのセットアップ
author: rick-anderson
description: このチュートリアルでは、既存の ASP.NET Core アプリに Google アカウントユーザー認証を統合する方法について説明します。
ms.author: riande
ms.custom: mvc, seodec18
ms.date: 02/18/2021
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
uid: security/authentication/google-logins
ms.openlocfilehash: 181ce87e8085839e0fcc0d77010c588ef7a290b1
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102110132"
---
# <a name="google-external-login-setup-in-aspnet-core"></a>ASP.NET Core での Google 外部ログインのセットアップ

作成者: [Valeriy Novytskyy](https://github.com/01binary)、[Rick Anderson](https://twitter.com/RickAndMSFT)

このチュートリアルでは、 [前のページ](xref:security/authentication/social/index)で作成した ASP.NET Core 3.0 プロジェクトを使用して、ユーザーが自分の Google アカウントでサインインできるようにする方法について説明します。

## <a name="create-a-google-api-console-project-and-client-id"></a>Google API コンソールプロジェクトとクライアント ID を作成する

* アプリに [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Google) NuGet パッケージを追加します。
* [「Google Sign-In を web アプリに統合する](https://developers.google.com/identity/sign-in/web/sign-in)(google ドキュメント)」のガイダンスに従ってください。
* [Google コンソール](https://console.developers.google.com/apis/credentials)の [**資格情報**] ページで、[**資格情報の作成**] [  >  **OAuth クライアント ID**] を選択します。
* [ **アプリケーションの種類** ] ダイアログボックスで、[ **Web アプリケーション**] を選択します。 アプリの **名前** を指定します。
* [承認された **リダイレクト uri** ] セクションで、[ **uri の追加** ] を選択してリダイレクト uri を設定します。 リダイレクト URI の例: `https://localhost:{PORT}/signin-google` 、 `{PORT}` プレースホルダーはアプリのポートです。
* [ **作成** ] ボタンを選択します。
* アプリの構成で使用するために、 **クライアント ID** と **クライアントシークレット** を保存します。
* サイトを展開するときは、次のいずれかの方法を実行します。
  * **Google Console** のアプリのリダイレクト uri をアプリのデプロイ済みリダイレクト uri に更新します。
  * 運用環境のリダイレクト URI を使用して、 **Google コンソール** で、実稼働アプリの新しい google API 登録を作成します。

## <a name="store-the-google-client-id-and-secret"></a>Google クライアント ID とシークレットを保存する

Google クライアント ID やシークレット値などの機微な設定を [Secret Manager](xref:security/app-secrets)に保存します。 このサンプルでは、次の手順を使用します。

1. 「 [シークレットストレージを有効にする](xref:security/app-secrets#enable-secret-storage)」の手順に従って、シークレットストレージのプロジェクトを初期化します。
1. 秘密キーとシークレットキーを使用して、機密設定をローカルシークレットストアに保存 `Authentication:Google:ClientId` し `Authentication:Google:ClientSecret` ます。

    ```dotnetcli
    dotnet user-secrets set "Authentication:Google:ClientId" "<client-id>"
    dotnet user-secrets set "Authentication:Google:ClientSecret" "<client-secret>"
    ```

[!INCLUDE[](~/includes/environmentVarableColon.md)]

Api [コンソール](https://console.developers.google.com/apis/dashboard)で api の資格情報と使用状況を管理できます。

## <a name="configure-google-authentication"></a>Google 認証を構成する

Google サービスをに追加し `Startup.ConfigureServices` ます。

[!code-csharp[](~/security/authentication/social/social-code/3.x/StartupGoogle3x.cs?highlight=11-19)]

[!INCLUDE [default settings configuration](includes/default-settings2-2.md)]

## <a name="sign-in-with-google"></a>Google でサインイン

* アプリを実行し、[ **ログイン**] をクリックします。 Google でサインインするためのオプションが表示されます。
* [ **Google** ] ボタンをクリックします。これにより、認証のために google にリダイレクトされます。
* Google の資格情報を入力すると、web サイトにリダイレクトされます。

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

[!INCLUDE[](includes/chain-auth-providers.md)]

<xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions>Google 認証でサポートされる構成オプションの詳細については、「API リファレンス」を参照してください。 これは、ユーザーに関するさまざまな情報を要求するために使用できます。

## <a name="change-the-default-callback-uri"></a>既定のコールバック URI を変更する

URI セグメント `/signin-google` は、Google 認証プロバイダーの既定のコールバックとして設定されます。 クラスの "継承された" プロパティを使用して Google authentication ミドルウェアを構成するときに、既定のコールバック URI を変更でき <xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.CallbackPath?displayProperty=nameWithType> <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions> ます。

## <a name="troubleshooting"></a>トラブルシューティング

* サインインが機能せず、エラーが発生しない場合は、開発モードに切り替えて、問題を簡単にデバッグできるようにします。
* Identityでを呼び出すことによってが構成されていない場合 `services.AddIdentity` 、ArgumentException で結果を認証しようとしています。 `ConfigureServices` *' SignInScheme ' オプションを指定する必要があり* ます。 このチュートリアルで使用するプロジェクトテンプレートによって、この処理が確実に行われます。
* 初期移行を適用してサイトデータベースが作成されていない場合は、 *要求エラーの処理中にデータベース操作が失敗* します。 [ **移行の適用** ] を選択してデータベースを作成し、ページを更新してエラーを解消します。

## <a name="next-steps"></a>次のステップ

* この記事では、Google で認証する方法について説明しました。 同様のアプローチに従って、 [前のページ](xref:security/authentication/social/index)に一覧表示されている他のプロバイダーとの認証を行うことができます。
* アプリを Azure に発行したら、 `ClientSecret` GOOGLE API コンソールでをリセットします。
* とを `Authentication:Google:ClientId` 、 `Authentication:Google:ClientSecret` Azure portal のアプリケーション設定として設定します。 構成システムは、環境変数からキーを読み取るように設定されています。
