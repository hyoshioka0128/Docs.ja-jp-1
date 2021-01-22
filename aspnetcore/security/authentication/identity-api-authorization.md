---
title: ASP.NET Core でのシングルページアプリの認証の概要
author: javiercn
description: IdentityASP.NET Core アプリ内でホストされるシングルページアプリで使用します。
monikerRange: '>= aspnetcore-3.0'
ms.author: scaddie
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
uid: security/authentication/identity/spa
ms.openlocfilehash: 5a6c160ebdda3ec600980aa839770f4f22a9c2fc
ms.sourcegitcommit: cc405f20537484744423ddaf87bd1e7d82b6bdf0
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/21/2021
ms.locfileid: "98658665"
---
# <a name="authentication-and-authorization-for-spas"></a>SPAs の認証と承認

ASP.NET Core 3.1 以降のテンプレートでは、API 承認のサポートを使用して、シングルページアプリ (spa) で認証を提供します。 ASP.NET Core Identity認証と保存のために、ユーザーを[ Identity サーバー](https://identityserver.io/)と組み合わせて OpenID connect を実装します。

認証パラメーターが、 **Web アプリケーション (モデルビューコントローラー)** (MVC) および **web アプリケーション**(ページ) プロジェクトテンプレートの認証パラメーターに似た **角度** で、**応答** するプロジェクトテンプレートに追加されました Razor 。 許可されるパラメーター値は、 **None** および **個人** です。 **React.js と Redux** プロジェクトテンプレートでは、現時点では認証パラメーターがサポートされていません。

## <a name="create-an-app-with-api-authorization-support"></a>API authorization サポートを使用してアプリを作成する

ユーザーの認証と承認は、両方の角度で使用でき、SPAs として対応します。 コマンドシェルを開き、次のコマンドを実行します。

**角度**:

```dotnetcli
dotnet new angular -o <output_directory_name> -au Individual
```

**反応**:

```dotnetcli
dotnet new react -o <output_directory_name> -au Individual
```

前のコマンドは、 *ClientApp* ディレクトリに SPA を含む ASP.NET Core アプリを作成します。

## <a name="general-description-of-the-aspnet-core-components-of-the-app"></a>アプリの ASP.NET Core コンポーネントの一般的な説明

次のセクションでは、認証のサポートが含まれている場合のプロジェクトへの追加について説明します。

### <a name="startup-class"></a>スタートアップ クラス

次のコード例は、AspNetCore に依存して [い Identity ます。サーバー](https://www.nuget.org/packages/Microsoft.AspNetCore.ApiAuthorization.IdentityServer) NuGet パッケージ。 この例では、および拡張メソッドを使用して、API の認証と承認を構成し <xref:Microsoft.Extensions.DependencyInjection.IdentityServerBuilderConfigurationExtensions.AddApiAuthorization%2A> <xref:Microsoft.AspNetCore.ApiAuthorization.IdentityServer.ApiResourceCollection.AddIdentityServerJwt%2A> ます。 認証を使用して、応答または角速度の SPA プロジェクトテンプレートを使用するプロジェクトには、このパッケージへの参照が含まれます。

クラスには `Startup` 、次の追加機能があります。

* メソッド内 `Startup.ConfigureServices` :
  * Identity 既定の UI では、次のようになります。

    ```csharp
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlite(Configuration.GetConnectionString("DefaultConnection")));

    services.AddDefaultIdentity<ApplicationUser>()
        .AddEntityFrameworkStores<ApplicationDbContext>();
    ```

  * Identityサーバーに追加のヘルパーメソッドを使用して、 `AddApiAuthorization` サーバー上に既定の ASP.NET Core 規則を設定する Identity 。

    ```csharp
    services.AddIdentityServer()
        .AddApiAuthorization<ApplicationUser, ApplicationDbContext>();
    ```

  * IdentityServer によって生成された JWT トークンを検証するようにアプリを構成する `AddIdentityServerJwt` ヘルパー メソッドが追加された Authentication:

    ```csharp
    services.AddAuthentication()
        .AddIdentityServerJwt();
    ```

* メソッド内 `Startup.Configure` :
  * 要求の資格情報を検証し、要求コンテキストでユーザーを設定する認証ミドルウェア。

    ```csharp
    app.UseAuthentication();
    ```

  * IdentityOpenID connect エンドポイントを公開するサーバーミドルウェアは次のとおりです。

    ```csharp
    app.UseIdentityServer();
    ```

### <a name="azure-app-service-on-linux"></a>Azure App Service on Linux

Linux での Azure App Service デプロイについては、次のように発行者を明示的に指定し `Startup.ConfigureServices` ます。

```csharp
services.Configure<JwtBearerOptions>(
    IdentityServerJwtConstants.IdentityServerJwtBearerScheme, 
    options =>
    {
        options.Authority = "{AUTHORITY}";
    });
```

前のコードでは、 `{AUTHORITY}` <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerOptions.Authority> OpenID connect 呼び出しを行うときに、プレースホルダーが使用されます。

例:

```csharp
options.Authority = "https://contoso-service.azurewebsites.net";
```

### <a name="addapiauthorization"></a>AddApiAuthorization

このヘルパーメソッド Identity は、サポートされている構成を使用するようにサーバーを構成します。 IdentityServer は、アプリのセキュリティの問題を処理するための強力で拡張可能なフレームワークです。 同時に、最も一般的なシナリオでは不必要な複雑さを公開します。 そのため、適切な出発点と見なされる一連の規則と構成オプションが用意されています。 認証を変更する必要がある場合でも、サーバーの能力を最大限に活用して、 Identity 必要に応じて認証をカスタマイズできます。

### <a name="addno-locidentityserverjwt"></a>AddIdentityServerJwt

このヘルパーメソッドは、アプリのポリシースキームを既定の認証ハンドラーとして構成します。 このポリシーは、 Identity Identity URL スペース "/" のサブパスにルーティングされるすべての要求を処理できるように構成されてい Identity ます。 それ以外のすべての要求は、`JwtBearerHandler` で処理されます。 さらに、このメソッドは `<<ApplicationName>>API` API リソースを Identity の既定のスコープと共にサーバーに登録 `<<ApplicationName>>API` し、アプリケーションのサーバーによって発行されたトークンを検証するように JWT ベアラートークンミドルウェアを構成し Identity ます。

### <a name="weatherforecastcontroller"></a>WeatherForecastController

*Controllers\WeatherForecastController.cs* ファイルで、属性がクラスに適用されていることを確認します。これは、 `[Authorize]` リソースにアクセスするための既定のポリシーに基づいてユーザーを承認する必要があることを示します。 既定の承認ポリシーは、前に説明したポリシースキームによって設定される既定の認証スキームを使用するように構成され `AddIdentityServerJwt` 、 `JwtBearerHandler` このようなヘルパーメソッドによって構成されたは、アプリに対する要求の既定のハンドラーになります。

### <a name="applicationdbcontext"></a>ApplicationDbContext

*Data\ApplicationDbContext.cs* ファイルで `DbContext` は、が拡張し Identity た例外 `ApiAuthorizationDbContext` (から派生したクラス) を使用して、 `IdentityDbContext` サーバーのスキーマを含めるという点に注意して Identity ください。

データベーススキーマを完全に制御するには、使用可能なクラスの1つを継承 Identity `DbContext` し、メソッドでを呼び出してスキーマを含めるようにコンテキストを構成し Identity `builder.ConfigurePersistedGrantContext(_operationalStoreOptions.Value)` `OnModelCreating` ます。

### <a name="oidcconfigurationcontroller"></a>OidcConfigurationController

*Controllers\OidcConfigurationController.cs* ファイルで、クライアントが使用する必要のある oidc パラメーターを提供するためにプロビジョニングされたエンドポイントを確認します。

### appsettings.json

*appsettings.json* プロジェクトルートのファイルには、 `IdentityServer` 構成されたクライアントの一覧を説明する新しいセクションがあります。 次の例には、1 つのクライアントがあります。 クライアント名はアプリケーション名に対応し、規則によって OAuth の `ClientId` パラメーターにマップされます。 構成対象のアプリの種類は、プロファイルによって示されています。 サーバーの構成プロセスを簡略化する規則を実現するために、内部的に使用されます。 「 [アプリケーションプロファイル](#application-profiles) 」セクションで説明されているように、使用可能なプロファイルがいくつかあります。

```json
"IdentityServer": {
  "Clients": {
    "angularindividualpreview3final": {
      "Profile": "IdentityServerSPA"
    }
  }
}
```

### <a name="appsettingsdevelopmentjson"></a>appsettings.Development.js

プロジェクトルートの *appsettings.Development.js* のファイルには、 `IdentityServer` トークンの署名に使用されるキーについて説明するセクションがあります。 運用環境にデプロイする場合は、「 [運用環境にデプロイする](#deploy-to-production) 」セクションで説明されているように、アプリと共にキーをプロビジョニングしてデプロイする必要があります。

```json
"IdentityServer": {
  "Key": {
    "Type": "Development"
  }
}
```

## <a name="general-description-of-the-angular-app"></a>角度アプリの一般的な説明

角度テンプレートでの認証と API 承認のサポートは、独自の角度モジュールの *Clientapp-authorization* ディレクトリに存在します。 モジュールは、次の要素で構成されています。

* 3個のコンポーネント:
  * *login. component. ts*: アプリのログインフローを処理します。
  * *logout*: アプリのログアウトフローを処理します。
  * *login-menu. component. ts*: 次のリンクのセットのいずれかを表示するウィジェット。
    * ユーザーが認証されると、ユーザープロファイルの管理とログアウトのリンクがあります。
    * ユーザーが認証されていない場合の登録とログインリンク。
* ルートに追加できるルートガード `AuthorizeGuard` 。ルートにアクセスする前にユーザーを認証する必要があります。
* `AuthorizeInterceptor`ユーザーが認証されるときに、API を対象とする発信 http 要求にアクセストークンを関連付ける http インターセプター。
* `AuthorizeService`認証プロセスの下位レベルの詳細を処理し、認証されたユーザーに関する情報を、使用するアプリの残りの部分に公開するサービス。
* アプリの認証部分に関連付けられているルートを定義する角度モジュール。 ログインメニューコンポーネント、インターセプター、ガード、およびアプリの残りの部分から使用するためのサービスを公開します。

## <a name="general-description-of-the-react-app"></a>反応アプリの一般的な説明

応答テンプレートでの認証と API 承認のサポートは、 *ClientApp\src\components\api-authorization* ディレクトリにあります。 これは、次の要素で構成されています。

* 4つのコンポーネント:
  * *Login.js*: アプリのログインフローを処理します。
  * *Logout.js*: アプリのログアウトフローを処理します。
  * *LoginMenu.js*: 次のリンクのセットのいずれかを表示するウィジェット。
    * ユーザーが認証されると、ユーザープロファイルの管理とログアウトのリンクがあります。
    * ユーザーが認証されていない場合の登録とログインリンク。
  * *AuthorizeRoute.js*: パラメーターに示されているコンポーネントを表示する前に、ユーザーを認証する必要があるルートコンポーネント。 `Component`
* `authService` `AuthorizeService` 認証プロセスの下位レベルの詳細を処理し、認証されたユーザーに関する情報をアプリの残りの部分に公開する、クラスのエクスポートされたインスタンス。

これで、ソリューションの主要なコンポーネントを確認できました。次は、アプリの個々のシナリオについて詳しく見ていきましょう。

## <a name="require-authorization-on-a-new-api"></a>新しい API での承認が必要

既定では、システムは新しい Api の承認を要求するように構成されています。 これを行うには、新しいコントローラーを作成し、コントローラー `[Authorize]` クラスまたはコントローラー内の任意のアクションに属性を追加します。

## <a name="customize-the-api-authentication-handler"></a>API 認証ハンドラーをカスタマイズする

API の JWT ハンドラーの構成をカスタマイズするには、そのインスタンスを構成し <xref:Microsoft.AspNetCore.Builder.JwtBearerOptions> ます。

```csharp
services.AddAuthentication()
    .AddIdentityServerJwt();

services.Configure<JwtBearerOptions>(
    IdentityServerJwtConstants.IdentityServerJwtBearerScheme,
    options =>
    {
        ...
    });
```

API の JWT ハンドラーは、を使用して認証プロセスを制御できるようにするイベントを発生させ `JwtBearerEvents` ます。 は、API 承認のサポートを提供するために、 `AddIdentityServerJwt` 独自のイベントハンドラーを登録します。

イベントの処理をカスタマイズするには、必要に応じて追加のロジックを使用して既存のイベントハンドラーをラップします。 次に例を示します。

```csharp
services.Configure<JwtBearerOptions>(
    IdentityServerJwtConstants.IdentityServerJwtBearerScheme,
    options =>
    {
        var onTokenValidated = options.Events.OnTokenValidated;       
        
        options.Events.OnTokenValidated = async context =>
        {
            await onTokenValidated(context);
            ...
        }
    });
```

前のコードでは、 `OnTokenValidated` イベントハンドラーはカスタム実装に置き換えられています。 この実装は次のとおりです。

1. API 承認サポートによって提供される元の実装を呼び出します。
1. 独自のカスタムロジックを実行します。

## <a name="protect-a-client-side-route-angular"></a>クライアント側のルートを保護する (角度)

クライアント側ルートの保護は、ルートを構成するときに実行するガードのリストに承認ガードを追加することによって行われます。 例として、主要なアプリの `fetch-data` 角度モジュール内でルートがどのように構成されているかを確認できます。

```typescript
RouterModule.forRoot([
  // ...
  { path: 'fetch-data', component: FetchDataComponent, canActivate: [AuthorizeGuard] },
])
```

ルートを保護しても実際のエンドポイントが保護されないことに注意してください (まだ属性が適用されている必要があり `[Authorize]` ます) が、ユーザーが認証されていないときに、特定のクライアント側ルートに移動できないようにします。

## <a name="authenticate-api-requests-angular"></a>API 要求の認証 (角度)

アプリと共にホストされる Api に対する要求の認証は、アプリによって定義された HTTP クライアントインターセプターを使用することによって自動的に行われます。

## <a name="protect-a-client-side-route-react"></a>クライアント側のルートを保護する (応答)

プレーンコンポーネントの代わりにコンポーネントを使用して、クライアント側のルートを保護し `AuthorizeRoute` `Route` ます。 たとえば、 `fetch-data` コンポーネント内でルートがどのように構成されているかに注目して `App` ください。

```jsx
<AuthorizeRoute path='/fetch-data' component={FetchData} />
```

ルートの保護:

* は実際のエンドポイントを保護しません (まだ `[Authorize]` 属性が適用されている必要があります)。
* は、ユーザーが認証されていないときに、特定のクライアント側ルートに移動できないようにします。

## <a name="authenticate-api-requests-react"></a>API 要求の認証 (応答)

を使用して要求を認証するには `authService` 、まずからインスタンスをインポートし `AuthorizeService` ます。 アクセストークンはから取得され、 `authService` 次に示すように要求にアタッチされます。 コンポーネントの反応では、通常、この作業は `componentDidMount` ライフサイクルメソッドで実行されるか、一部のユーザー操作の結果として行われます。

### <a name="import-the-authservice-into-your-component"></a>コンポーネントに authService をインポートする

```javascript
import authService from './api-authorization/AuthorizeService'
```

### <a name="retrieve-and-attach-the-access-token-to-the-response"></a>アクセストークンを取得して応答にアタッチする

```javascript
async populateWeatherData() {
  const token = await authService.getAccessToken();
  const response = await fetch('api/SampleData/WeatherForecasts', {
    headers: !token ? {} : { 'Authorization': `Bearer ${token}` }
  });
  const data = await response.json();
  this.setState({ forecasts: data, loading: false });
}
```

## <a name="deploy-to-production"></a>運用環境への配置

アプリを運用環境にデプロイするには、次のリソースをプロビジョニングする必要があります。

* Identityユーザーアカウントとサーバー権限を格納するデータベース Identity 。
* トークンの署名に使用する実稼働証明書。
  * この証明書には特定の要件はありません。自己署名証明書、または CA 証明機関を通じてプロビジョニングされた証明書を使用できます。
  * PowerShell や OpenSSL などの標準ツールを使用して生成できます。
  * ターゲットコンピューターの証明書ストアにインストールすることも、強力なパスワードを使用して *.pfx* ファイルとして展開することもできます。

### <a name="example-deploy-to-a-non-azure-web-hosting-provider"></a>例: Azure 以外の web ホスティングプロバイダーへのデプロイ

Web ホスティングパネルで、証明書を作成または読み込みます。 次に、アプリの *appsettings.json* ファイルで、セクションを変更し `IdentityServer` てキーの詳細を含めます。 次に例を示します。

```json
"IdentityServer": {
  "Key": {
    "Type": "Store",
    "StoreName": "WebHosting",
    "StoreLocation": "CurrentUser",
    "Name": "CN=MyApplication"
  }
}
```

前の例の場合:

* `StoreName` 証明書が格納されている証明書ストアの名前を表します。 この例では、web ホスティングストアを参照しています。
* `StoreLocation` 証明書の読み込み元 ( `CurrentUser` この場合は) を表します。
* `Name` 証明書の識別サブジェクトに対応します。

### <a name="example-deploy-to-azure-app-service"></a>例: Azure App Service に配置する

このセクションでは、証明書ストアに格納されている証明書を使用して Azure App Service するためのアプリの展開について説明します。 証明書ストアから証明書を読み込むようにアプリを変更するには、後の手順で Azure portal でアプリを構成するときに、Standard レベルのサービスプラン以上が必要です。

アプリのファイルで *appsettings.json* 、セクションを変更し `IdentityServer` て、キーの詳細を含めます。

```json
"IdentityServer": {
  "Key": {
    "Type": "Store",
    "StoreName": "My",
    "StoreLocation": "CurrentUser",
    "Name": "CN=MyApplication"
  }
}
```

* ストア名は、証明書が格納されている証明書ストアの名前を表します。 この場合、個人ユーザーストアを指します。
* ストアの場所は、証明書を読み込む場所 ( `CurrentUser` または) を表し `LocalMachine` ます。
* Certificate の name プロパティは、証明書の識別サブジェクトに対応します。

Azure App Service にデプロイするには、「 [azure へのアプリのデプロイ](xref:tutorials/publish-to-azure-webapp-using-vs#deploy-the-app-to-azure)」の手順に従って、必要な azure リソースを作成し、アプリを運用環境にデプロイする方法を説明します。

前述の手順に従うと、アプリは Azure にデプロイされますが、まだ機能していません。 アプリによって使用される証明書は、Azure portal で構成する必要があります。 証明書のサムプリントを見つけて、「 [証明書の読み込み](/azure/app-service/app-service-web-ssl-cert-load#load-the-certificate-in-code)」で説明されている手順に従います。

これらの手順では SSL について説明していますが、Azure portal には、アプリで使用するプロビジョニング済みの証明書をアップロードできる **プライベート証明書** セクションがあります。

Azure portal でアプリとアプリの設定を構成した後、ポータルでアプリを再起動します。

## <a name="other-configuration-options"></a>その他の構成オプション

API 承認のサポートは、 Identity 一連の規則、既定値、および拡張機能を使用してサーバー上に構築され、SPAs のエクスペリエンスが簡単になります。 言うまで Identity もありませんが、ASP.NET Core 統合によってシナリオがカバーされていない場合は、サーバーの全機能をバックグラウンドで利用できます。 ASP.NET Core サポートは、すべてのアプリが組織によって作成および展開される "ファーストパーティ" アプリに重点を置いています。 そのため、同意やフェデレーションなどのサポートは提供されていません。 これらのシナリオでは、Server を使用 Identity して、そのドキュメントに従ってください。

### <a name="application-profiles"></a>アプリケーションプロファイル

アプリケーションプロファイルは、そのパラメーターをさらに定義するアプリの事前定義された構成です。 現時点では、次のプロファイルがサポートされています。

* `IdentityServerSPA`: サーバーと共にホストされる SPA を Identity 1 つの単位として表します。
  * の `redirect_uri` 既定値は `/authentication/login-callback` です。
  * の `post_logout_redirect_uri` 既定値は `/authentication/logout-callback` です。
  * スコープのセットには、 `openid` `profile` アプリ内の api に対して定義されている、、、およびすべてのスコープが含まれます。
  * 許可される OIDC 応答の種類のセットは、 `id_token token` それぞれ個別に ( `id_token` 、 `token` ) です。
  * 許可される応答モードは `fragment` です。
* `SPA`: サーバーでホストされていない SPA を表し Identity ます。
  * スコープのセットには、 `openid` `profile` アプリ内の api に対して定義されている、、、およびすべてのスコープが含まれます。
  * 許可される OIDC 応答の種類のセットは、 `id_token token` それぞれ個別に ( `id_token` 、 `token` ) です。
  * 許可される応答モードは `fragment` です。
* `IdentityServerJwt`: サーバーと共にホストされる API を表し Identity ます。
  * アプリは、アプリ名を既定とする1つのスコープを持つように構成されています。
* `API`: サーバーでホストされていない API を表し Identity ます。
  * アプリは、アプリ名を既定とする1つのスコープを持つように構成されています。

### <a name="configuration-through-appsettings"></a>AppSettings を使用した構成

構成システムを使用して、またはの一覧にアプリを追加して、アプリを構成し `Clients` `Resources` ます。

`redirect_uri` `post_logout_redirect_uri` 次の例に示すように、各クライアントのおよびプロパティを構成します。

```json
"IdentityServer": {
  "Clients": {
    "MySPA": {
      "Profile": "SPA",
      "RedirectUri": "https://www.example.com/authentication/login-callback",
      "LogoutUri": "https://www.example.com/authentication/logout-callback"
    }
  }
}
```

リソースを構成するときは、次に示すように、リソースのスコープを構成できます。

```json
"IdentityServer": {
  "Resources": {
    "MyExternalApi": {
      "Profile": "API",
      "Scopes": "a b c"
    }
  }
}
```

### <a name="configuration-through-code"></a>コードを使用した構成

また、 `AddApiAuthorization` オプションを構成するためのアクションを実行するのオーバーロードを使用して、コードを使用してクライアントとリソースを構成することもできます。

```csharp
AddApiAuthorization<ApplicationUser, ApplicationDbContext>(options =>
{
    options.Clients.AddSPA(
        "My SPA", spa =>
        spa.WithRedirectUri("http://www.example.com/authentication/login-callback")
           .WithLogoutRedirectUri(
               "http://www.example.com/authentication/logout-callback"));

    options.ApiResources.AddApiResource("MyExternalApi", resource =>
        resource.WithScopes("a", "b", "c"));
});
```

## <a name="additional-resources"></a>その他のリソース

* <xref:spa/angular>
* <xref:spa/react>
* <xref:security/authentication/scaffold-identity>
