---
title: 詳細な構成
author: rick-anderson
description: ASP.NET Core モジュールとインターネット インフォメーション サービス (IIS) に関する詳細な構成。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 5/7/2020
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
uid: host-and-deploy/iis/advanced
ms.openlocfilehash: de3abb7b5f89e3b84da38a8eabb183f9551cc1f9
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588920"
---
# <a name="advanced-configuration-of-the-aspnet-core-module-and-iis"></a>ASP.NET Core モジュールと IIS の詳細な構成

この記事では、ASP.NET Core モジュールと IIS の詳細な構成のオプションとシナリオについて説明します。

## <a name="modify-the-stack-size"></a>スタック サイズを変更する

"*インプロセス ホスティング モデルを使用している場合にのみ適用されます。* "

マネージド スタックのサイズは、`web.config` ファイルで `stackSize` の設定を使用して構成します (バイト単位)。 既定のサイズは 1,048,576 バイト (1 MB) です。 次の例では、スタック サイズを 2 MB (2,097,152 バイト) に変更します。

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="false"
    stdoutLogFile="\\?\%home%\LogFiles\stdout"
    hostingModel="inprocess">
  <handlerSettings>
    <handlerSetting name="stackSize" value="2097152" />
  </handlerSettings>
</aspNetCore>
```

## <a name="proxy-configuration-uses-http-protocol-and-a-pairing-token"></a>プロキシの構成で HTTP プロトコルとペアリング トークンを使用する

*アウト プロセス ホスティングにのみ適用されます。*

ASP.NET Core モジュールと Kestrel の間に作成されるプロキシは、HTTP プロトコルを使います。 モジュールと Kestrel の間のトラフィックが、サーバーから離れた場所で傍受される危険はありません。

ペアリング トークンを使用すると、Kestrel によって受信される要求が IIS によってプロキシされたものであり、他のソースからのものでないことを保証できます。 ペアリング トークンは、モジュールによって作成されて、環境変数 (`ASPNETCORE_TOKEN`) に設定されます。 ペアリング トークンはまた、プロキシされたすべての要求のヘッダー (`MS-ASPNETCORE-TOKEN`) にも設定されます。 IIS ミドルウェアは、受信した各要求をチェックし、ペアリング トークン ヘッダーの値が環境変数の値と一致することを確認します。 トークンの値が一致しない場合、要求はログに記録され、拒否されます。 ペアリング トークン環境変数およびモジュールと Kestrel の間のトラフィックには、サーバーから離れた場所からアクセスすることはできません。 ペアリング トークンの値がわからなければ、攻撃者は IIS ミドルウェアのチェックをバイパスする要求を送信できません。

## <a name="aspnet-core-module-with-an-iis-shared-configuration"></a>IIS 共有構成での ASP.NET Core モジュール

ASP.NET Core モジュールのインストーラーは、`TrustedInstaller` アカウントのアクセス許可を使って実行します。 ローカル システム アカウントには、IIS 共有構成によって使われる共有パスに対する変更アクセス許可がないため、インストーラーが共有上の `applicationHost.config` ファイル内のモジュール設定を構成しようとすると、アクセス拒否エラーがスローされます。

IIS がインストールされている同じコンピューターで IIS 共有抗生を使用するとき、`OPT_NO_SHARED_CONFIG_CHECK` パラメーターを `1` に設定して ASP.NET Core Hosting Bundle インストーラーを実行します。

```console
dotnet-hosting-{VERSION}.exe OPT_NO_SHARED_CONFIG_CHECK=1
```

共有構成のパスが IIS をインストールしている同じコンピューター上にないときは、次の手順を実行します。

1. IIS 共有構成を無効にします。
1. インストーラーを実行します。
1. 更新された `applicationHost.config` ファイルをファイル共有にエクスポートします。
1. IIS 共有構成を再び有効にします。

## <a name="data-protection"></a>データの保護

[ASP.NET Core データ保護スタック](xref:security/data-protection/introduction)は、認証で使用されるミドルウェアを含め、いくつかの [ASP.NET Core](xref:fundamentals/middleware/index) ミドルウェアによって使用されます。 データ保護 API がユーザーのコードから呼び出されない場合でも、配置スクリプトを使用するかユーザー コードで、永続的な暗号化[キー ストア](xref:security/data-protection/implementation/key-management)を作成するようにデータ保護を構成する必要があります。 データ保護を構成しない場合、既定でキーはメモリ内に保持され、アプリが再起動すると破棄されます。

データ保護キー リングがメモリに格納されている場合、アプリを再起動すると次のようになります。

* すべての cookie ベースの認証トークンは無効になります。 
* ユーザーは、次回の要求時に再度サインインする必要があります。 
* キーリングで保護されているデータは、いずれも復号化できなくなります。 これには、[CSRF トークン](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration)と [ASP.NET Core MVC TempData cookie](xref:fundamentals/app-state#tempdata) が含まれます。

キーリングを保持するために IIS でのデータ保護を構成するには、次の **いずれか** の方法を使用する必要があります。

* **データ保護のレジストリ キーを作成する**

  ASP.NET Core アプリで使用されるデータ保護キーは、アプリの外部のレジストリに格納されます。 特定のアプリのキーを保持するには、アプリ プール用のレジストリ キーを作成する必要があります。

  非 Web ファーム IIS のスタンドアロン インストールの場合、ASP.NET Core アプリで使用するアプリ プールごとに、[データ保護の PowerShell スクリプト Provision-AutoGenKeys.ps1](https://github.com/dotnet/AspNetCore/blob/main/src/DataProtection/Provision-AutoGenKeys.ps1) を使用できます。 このスクリプトを使用すると、HKLM レジストリに、アプリのアプリ プールのワーカー プロセス アカウントのみがアクセスできるレジストリ キーが作成されます。 キーは保存時に、DPAPI を使用して、コンピューター全体に適用するキーによって暗号化されます。

  Web ファームのシナリオの場合は、UNC パスを使用してデータ保護キー リングを格納するようにアプリを構成できます。 既定では、キーは暗号化されません。 ネットワーク共有に対するファイルのアクセス許可が、アプリが実行される Windows アカウントに確実に限定されているようにしてください。 保存時のキーを保護するために、X509 証明書を使用することができます。 ユーザーが証明書をアップロードできるメカニズムを検討します。 ユーザーの信頼できる証明書ストアに証明書を配置し、ユーザーのアプリが実行されるすべてのコンピューターで証明書を利用できるようにします。 詳細については、「<xref:security/data-protection/configuration/overview>」を参照してください。

* **ユーザー プロファイルを読み込むための IIS アプリケーション プールを構成する**

  この設定は、アプリ プールの **[詳細設定]** の **[プロセス モデル]** セクションにあります。 **[ユーザー プロファイルの読み込み]** を `True` に設定します。 `True` に設定すると、キーはユーザー プロファイル ディレクトリに格納され、ユーザー アカウントに固有のキーと共にデータ保護 API を使って保護されます。 キーは `%LOCALAPPDATA%/ASP.NET/DataProtection-Keys` フォルダーに保存されます。

  アプリ プールの [`setProfileEnvironment` 属性](/iis/configuration/system.applicationhost/applicationpools/add/processmodel#configuration)も有効にする必要があります。 `setProfileEnvironment` の既定値は `true`です。 一部のシナリオ (たとえば、Windows OS) では、`setProfileEnvironment` は `false` に設定されます。 キーが期待どおりにユーザー プロファイル ディレクトリに格納されていない場合:

  1. `%windir%/system32/inetsrv/config` フォルダーに移動します。
  1. `applicationHost.config` ファイルを開きます。
  1. `<system.applicationHost><applicationPools><applicationPoolDefaults><processModel>` 要素を探します。
  1. `setProfileEnvironment` 属性 (その規定値は `true` です) が存在しないことを確認するか、属性の値を明示的に `true` に設定します。

* **ファイル システムをキー リング ストアとして使用する**

  [ファイル システムをキー リング ストアとして使用](xref:security/data-protection/configuration/overview)するようにアプリ コードを調整します。 X509 証明書を使用してキー リングを保護し、その証明書が信頼された証明書であることを確認します。 証明書が自己署名されている場合は、信頼されたルート ストアにその証明書を配置します。

  Web ファームで IIS を使用する場合:

  * すべてのコンピューターがアクセスできるファイル共有を使用します。
  * X509 証明書を各コンピューターに配置します。 [コードでデータ保護](xref:security/data-protection/configuration/overview)を構成します。

* **データ保護に対してコンピューター全体のポリシーを設定する**

  データ保護システムによる、Data Protection API を使用するすべてのアプリに対する、既定で[コンピューター全体に適用されるポリシー](xref:security/data-protection/configuration/machine-wide-policy)の設定のサポートは限定的です。 詳細については、「<xref:security/data-protection/introduction>」を参照してください。

## <a name="iis-configuration"></a>IIS 構成

**Windows Server オペレーティング システム**

**Web サーバー (IIS)** サーバーの役割を有効にし、役割のサービスを設定します。

1. **[管理]** メニューから **役割と機能の追加** ウィザードを使用するか、**サーバー マネージャー** にあるリンクを使用します。 **[サーバーの役割]** のステップで、 **[Web サーバー (IIS)]** チェック ボックスをオンにします。

   ![[サーバーの役割の選択] のステップで Web サーバー IIS の役割を選択します。](index/_static/server-roles-ws2016.png)

1. **[機能]** のステップの後に、 **[役割サービスの]** のステップで Web サーバー (IIS) を読み込みます。 希望する IIS の役割サービスを選択するか、既定の役割サービスをそのまま使用します。

   ![[役割サービスの選択] のステップで既定の役割サービスを選択します。](index/_static/role-services-ws2016.png)

   **Windows 認証 (任意)**  
   Windows 認証を有効にするには、 **[Web サーバー]**  >  **[セキュリティ]** ノードを展開します。 **[Windows 認証]** 機能を選択します。 詳細については、「[Windows 認証 `<windowsAuthentication>`](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/)」および [Windows 認証の構成](xref:security/authentication/windowsauth)に関するページを参照してください。

   **Websocket (省略可能)**  
   WebSockets は、ASP.NET Core 1.1 以降でサポートされています。 WebSocket を有効にするには、 **[Web サーバー]**  >  **[アプリケーション開発]** ノードを展開します。 **[WebSocket プロトコル]** 機能を選択します。 詳細については、[WebSockets](xref:fundamentals/websockets) に関するページを参照してください。

1. **[確認]** のステップを経て Web サーバーの役割とサービスをインストールします。 **Web サーバー (IIS)** の役割をインストールした後、サーバーと IIS の再起動は必要ありません。

**Windows デスクトップ オペレーティング システム**

**[IIS 管理コンソール]** と **[World Wide Web サービス]** を有効にします。

1. **[コントロール パネル]**  >  **[プログラム]**  >  **[プログラムと機能]**  >  **[Windows の機能の有効化または無効化]** (画面の左側) に移動します。

1. **インターネット インフォメーション サービス (IIS)** ノードを開きます。 **[Web 管理ツール]** ノードを開きます。

1. **[IIS 管理コンソール]** チェック ボックスをオンにします。

1. **[World Wide Web サービス]** チェック ボックスをオンにします。

1. **[World Wide Web サービス]** の既定の機能をそのまま使用するか、IIS 機能をカスタマイズします。

   **Windows 認証 (任意)**  
   Windows 認証を有効にするには、 **[World Wide Web サービス]**  >  **[セキュリティ]** ノードを展開します。 **[Windows 認証]** 機能を選択します。 詳細については、「[Windows 認証 `<windowsAuthentication>`](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/)」および [Windows 認証の構成](xref:security/authentication/windowsauth)に関するページを参照してください。

   **Websocket (省略可能)**  
   WebSockets は、ASP.NET Core 1.1 以降でサポートされています。 WebSocket を有効にするには、 **[World Wide Web サービス]**  >  **[アプリケーション開発機能]** ノードを展開します。 **[WebSocket プロトコル]** 機能を選択します。 詳細については、[WebSockets](xref:fundamentals/websockets) に関するページを参照してください。

1. IIS のインストールに再起動が必要な場合は、システムを再起動します。

![[Windows の機能] で [IIS 管理コンソール] と [World Wide Web サービス] を選択します。](index/_static/windows-features-win10.png)

## <a name="virtual-directories"></a>仮想ディレクトリ

ASP.NET Core アプリでは [IIS 仮想ディレクトリ](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#virtual-directories)はサポートされません。 アプリは[サブアプリケーション](#sub-applications)としてホスティングできます。

## <a name="sub-applications"></a>サブアプリケーション

ASP.NET Core アプリは [IIS サブアプリケーション (サブアプリ)](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#applications) としてホスティングできます。 サブアプリのパスは、ルート アプリの URL の一部になります。

サブアプリ内にある静的資産のリンクでは、チルダとスラッシュの表記 (`~/`) を使う必要があります。 チルダとスラッシュの表記により[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)がトリガーされ、作成される相対リンクにサブアプリのパスベースが付加されます。 `/subapp_path` にあるサブアプリの場合、`src="~/image.png"` を使ってリンクされる画像は `src="/subapp_path/image.png"` として作成されます。 ルート アプリの静的ファイル ミドルウェアでは、静的ファイル要求は処理されません。 この要求は、サブアプリの静的ファイル ミドルウェアによって処理されます。

静的資産の `src` 属性が絶対パス (たとえば `src="/image.png"`) に設定されている場合、リンクはサブアプリのパスベースなしで作成されます。 ルート アプリの静的ファイル ミドルウェアでは、ルート アプリの [Web ルート](xref:fundamentals/index#web-root)から資産を提供しようとしますが、ルート アプリから静的資産を利用できる場合を除いて *404 - Not Found* 応答が返されます。

ある ASP.NET Core アプリを別の ASP.NET Core アプリの下でサブアプリとしてホスティングするには:

1. サブアプリ用のアプリ プールを確立します。 デスクトップ CLR (.NET CLR) ではなく、.NET Core 用の Core 共通言語ランタイム (CoreCLR) が起動されてワーカー プロセスでアプリがホストされるため、 **[.NET CLR バージョン]** を **[マネージド コードなし]** に設定します。

1. ルート サイトを IIS マネージャーに追加し、サブアプリをルート サイトの下のフォルダー内に置きます。

1. IIS マネージャーでサブアプリのフォルダーを右クリックし、 **[アプリケーションへの変換]** を選択します。

1. **[アプリケーションの追加]** ダイアログ ボックスで、 **[アプリケーション プール]** に対して **[選択]** ボタンを使い、作成したアプリケーション プールをサブアプリ用に割り当てます。 **[OK]** を選択します。

サブアプリに対して個別のアプリ プールを割り当てることは、インプロセス ホスティング モデルを使用する場合必須となります。

インプロセス ホスティング モデルと ASP.NET Core モジュールの構成について詳しくは、<xref:host-and-deploy/aspnet-core-module> をご覧ください。

## <a name="application-pools"></a>アプリケーション プール

アプリケーション プールの分離は、ホスティング モデルによって決定されます。

* インプロセス ホスティング: 別のアプリケーション プールでアプリケーションを実行する必要があります。
* アウトプロセス ホスティング: アプリケーションをそれぞれ専用のアプリケーション プールで実行して、アプリケーションを相互に分離することをお勧めします。

IIS の **[Web サイトの追加]** ダイアログの既定では、アプリケーションごとに 1 つのアプリケーション プールです。 **[サイト名]** を指定すると、入力したテキストが自動的に **[アプリケーション プール]** テキストボックスに設定されます。 サイトが追加されるときに、そのサイト名を使用して新しいアプリ プールが作成されます。

## <a name="application-pool-identity"></a>アプリケーション プール Identity

アプリ プール ID アカウントを使用すると、ドメインやローカル アカウントを作成して管理する必要なく、一意のアカウントでアプリを実行できます。 IIS 8.0 以降の IIS 管理者ワーカー プロセス (WAS) は、新しいアプリ プールの名前で仮想アカウントを作成し、既定によってアプリ プールのワーカー プロセスをこのアカウントで実行します。 IIS 管理コンソールの、アプリ プールに対する **[詳細設定]** で、確実に **Identity** で `ApplicationPoolIdentity` が使用されるように設定します。

![アプリケーション プールの [詳細設定] ダイアログ](index/_static/apppool-identity.png)

IIS 管理プロセスは、Windows セキュリティ システムでのアプリ プールの名前を使用して、セキュリティで保護された識別子を作成します。 この ID を使用してリソースを保護することができます。 ただし、この ID は実際のユーザー アカウントではなく、Windows ユーザーの管理コンソールに表示されません。

アプリに対する IIS ワーカー プロセスのアクセス許可を昇格させる必要がある場合は、アプリを含むディレクトリのアクセス制御リスト (ACL) を変更します。

1. エクスプローラーを開き、そのディレクトリに移動します。

1. そのディレクトリを右クリックし、 **[プロパティ]** を選択します。

1. **[セキュリティ]** タブの **[編集]** ボタンを選択し、 **[追加]** ボタンをクリックします。

1. **[場所]** ボタンを選択し、システムが選択されていることを確認します。

1. **[選択するオブジェクト名を入力してください]** 領域に、`IIS AppPool\{APP POOL NAME}` という形式で入力します。プレースホルダー `{APP POOL NAME}` はアプリ プール名です。 **[名前の確認]** を選択します。 *DefaultAppPool* で、`IIS AppPool\DefaultAppPool` を使用して名前を確認します。 **[名前の確認]** ボタンが選択されているときには、`DefaultAppPool` の値が、オブジェクト名領域に表示されます。 オブジェクト名の領域に直接アプリケーション プール名を入力することはできません。 `IIS AppPool\{APP POOL NAME}` 形式を使用します。ここで、プレースホルダー `{APP POOL NAME}` は、オブジェクト名を確認するときのアプリ プール名です。

   ![アプリ フォルダーのユーザーまたはグループのダイアログを選択します。[名前の確認] を選択する前に、オブジェクト名領域で "DefaultAppPool" のアプリ プール名が "IIS AppPool\" に適用されます。](index/_static/select-users-or-groups-1.png)

1. **[OK]** を選択します。

   ![アプリ フォルダーのユーザーまたはグループのダイアログを選択します。[名前の確認] を選択した後に、オブジェクト名 "DefaultAppPool" がオブジェクト名領域に表示されます。](index/_static/select-users-or-groups-2.png)

1. 読み取り &amp; 実行アクセス許可は、既定で付与される必要があります。 必要に応じて、追加のアクセス許可を提供します。

**ICACLS** ツールを使用してコマンド プロンプトでアクセス許可を付与することもできます。 たとえば、*DefaultAppPool* を使用する場合、次のコマンドを使用します。

```console
ICACLS C:\sites\MyWebApp /grant "IIS AppPool\DefaultAppPool":F
```

詳細については、「[icacls](/windows-server/administration/windows-commands/icacls)」のトピックを参照してください。

## <a name="http2-support"></a>HTTP/2 のサポート

[HTTP/2](https://httpwg.org/specs/rfc7540.html) は、次の IIS 展開シナリオでの ASP.NET Core でサポートされます。

* インプロセス
  * Windows Server 2016/Windows 10 以降、IIS 10 以降
  * TLS 1.2 以降の接続
* アウトプロセス
  * Windows Server 2016/Windows 10 以降、IIS 10 以降
  * 一般向けのエッジ サーバーでは HTTP/2 を使用しますが、[Kestrel サーバー](xref:fundamentals/servers/kestrel)へのリバース プロキシ接続では HTTP/1.1 を使用します。
  * TLS 1.2 以降の接続

インプロセスの展開の場合、HTTP/2 接続が確立されると、[`HttpRequest.Protocol`](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) によって `HTTP/2` が報告されます。 アウトプロセスの展開の場合、HTTP/2 接続が確立されると、[`HttpRequest.Protocol`](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) によって `HTTP/1.1` が報告されます。

インプロセスおよびアウトプロセス ホスティング モデルの詳細については、<xref:host-and-deploy/aspnet-core-module> をご覧ください。

HTTP/2 は既定で有効になっています。 HTTP/2 接続が確立されない場合、接続は HTTP/1.1 にフォールバックします。 IIS 展開での HTTP/2 構成の詳細については、[IIS での HTTP/2](/iis/get-started/whats-new-in-iis-10/http2-on-iis) に関するページを参照してください。

## <a name="cors-preflight-requests"></a>CORS プレフライト要求

"*このセクションは、.NET Framework をターゲットにした ASP.NET Core アプリにのみ適用されます。* "

.NET Framework をターゲットにした ASP.NET Core アプリの場合、IIS では既定で OPTIONS 要求がアプリに渡されません。 OPTIONS 要求を渡すように `web.config` でアプリの IIS のハンドラーを構成する方法については、[ASP.NET Web API 2 でのクロスオリジン要求の有効化:CORS のしくみ](/aspnet/web-api/overview/security/enabling-cross-origin-requests-in-web-api#how-cors-works)に関する記事をご覧ください。

## <a name="application-initialization-module-and-idle-timeout"></a>Application Initialization モジュールとアイドル タイムアウト

IIS 内で ASP.NET Core モジュール バージョン 2 によってホストされている場合:

* [Application Initialization モジュール](#application-initialization-module): アプリのホストされている[インプロセス](xref:host-and-deploy/iis/in-process-hosting)または[アウトプロセス](xref:host-and-deploy/iis/out-of-process-hosting)は、ワーカー プロセスの再起動時またはサーバーの再起動時に自動的に起動するように構成できます。
* [アイドル タイムアウト](#idle-timeout): アプリのホストされている[インプロセス](xref:host-and-deploy/iis/in-process-hosting)は、非アクティブ期間中にタイムアウトしないように構成できます。

### <a name="application-initialization-module"></a>Application Initialization モジュール

"*アプリのホストされているインプロセスとアウトプロセスに適用されます。* "

[IIS Application Initialization](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization) は、アプリ プールが開始するときまたはリサイクルされるときに、アプリに HTTP 要求を送信する IIS 機能です。 要求によってアプリの起動がトリガーされます。 既定では、IIS ではアプリのルート URL (`/`) に対して要求が発行され、アプリが初期化されます (構成の詳細については[その他の技術情報](#application-initialization-module-and-idle-timeout-additional-resources)を参照)。

IIS Application Initialization 役割の機能が有効になっていることを確認します。

Windows 7 以降のデスクトップ システム上で、IIS をローカルで使う場合:

1. **[コントロール パネル]**  >  **[プログラム]**  >  **[プログラムと機能]**  >  **[Windows の機能の有効化または無効化]** (画面の左側) に移動します。
1. **[インターネット インフォメーション サービス]**  >  **[World Wide Web サービス]**  >  **[アプリケーション開発機能]** を開きます。
1. **[Application Initialization]** のチェック ボックスをオンにします。

Windows Server 2008 R2 以降の場合:

1. **[役割と機能の追加ウィザード]** を開きます。
1. **[役割サービスの選択]** パネルで、 **[アプリケーション開発]** ノードを開きます。
1. **[Application Initialization]** のチェック ボックスをオンにします。

次の方法のいずれかを使って、サイトの Application Initialization モジュールを有効にします。

* IIS マネージャーを使う場合:

  1. **[接続]** パネルの **[アプリケーション プール]** を選択します。
  1. 一覧でアプリのアプリ プールを右クリックして、 **[詳細設定]** を選択します。
  1. 既定の **[開始モード]** は `OnDemand` です。 **[開始モード]** を `AlwaysRunning` に設定します。 **[OK]** を選択します。
  1. **[接続]** パネルの **[サイト]** ノードを開きます。
  1. アプリを右クリックして、 **[Web サイトの管理]**  >  **[詳細設定]** の順に選択します。
  1. 既定の **[有効化されたプリロード]** 設定は `False` です。 **[有効化されたプリロード]** を `True` に設定します。 **[OK]** を選択します。

* `web.config` を使う場合は、`doAppInitAfterRestart` を `true` に設定した `<applicationInitialization>` 要素を、アプリの `web.config` ファイルの `<system.webServer>` 要素に追加します。

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <configuration>
    <location path="." inheritInChildApplications="false">
      <system.webServer>
        <applicationInitialization doAppInitAfterRestart="true" />
      </system.webServer>
    </location>
  </configuration>
  ```

### <a name="idle-timeout"></a>アイドル タイムアウト

"*アプリのホストされているインプロセスにのみ適用されます。* "

アプリがアイドル状態にならないようにするには、IIS マネージャーを使ってアプリ プールのアイドル タイムアウトを設定します。

1. **[接続]** パネルの **[アプリケーション プール]** を選択します。
1. 一覧でアプリのアプリ プールを右クリックして、 **[詳細設定]** を選択します。
1. 既定の **[アイドル状態のタイムアウト (分)]** は `20` 分です。 **[アイドル状態のタイムアウト (分)]** を `0` (ゼロ) に設定します。 **[OK]** を選択します。
1. ワーカー プロセスをリサイクルします。

アプリのホストされている[アウトプロセス](xref:host-and-deploy/iis/out-of-process-hosting)がタイムアウトにならないようにするには、次の方法のいずれかを使います。

* 実行させ続けるために、外部サービスからアプリに ping を送信します。
* アプリでバックグラウンド サービスのみをホストしている場合は、IIS ホスティングを回避し、[ASP.NET Core アプリをホストするための Windows サービス](xref:host-and-deploy/windows-service)を使います。

### <a name="application-initialization-module-and-idle-timeout-additional-resources"></a>Application Initialization モジュールとアイドル タイムアウトに関するその他の技術情報

* [IIS 8.0 Application Initialization](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization)
* [Application Initialization `<applicationInitialization>`](/iis/configuration/system.webserver/applicationinitialization/)
* [アプリケーション プールのプロセス モデルの設定 `<processModel>`](/iis/configuration/system.applicationhost/applicationpools/add/processmodel)。

## <a name="module-schema-and-configuration-file-locations"></a>モジュール、スキーマ、構成ファイルの場所

### <a name="module"></a>Module

**IIS (x86/amd64)** :

* `%windir%\System32\inetsrv\aspnetcore.dll`

* `%windir%\SysWOW64\inetsrv\aspnetcore.dll`

* `%ProgramFiles%\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll`

* `%ProgramFiles(x86)%\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll`

**IIS Express (x86/amd64)** :

* `%ProgramFiles%\IIS Express\aspnetcore.dll`

* `%ProgramFiles(x86)%\IIS Express\aspnetcore.dll`

* `%ProgramFiles%\IIS Express\Asp.Net Core Module\V2\aspnetcorev2.dll`

* `%ProgramFiles(x86)%\IIS Express\Asp.Net Core Module\V2\aspnetcorev2.dll`

### <a name="schema"></a>Schema

**IIS**

* `%windir%\System32\inetsrv\config\schema\aspnetcore_schema.xml`

* `%windir%\System32\inetsrv\config\schema\aspnetcore_schema_v2.xml`

**IIS Express**

* `%ProgramFiles%\IIS Express\config\schema\aspnetcore_schema.xml`

* `%ProgramFiles%\IIS Express\config\schema\aspnetcore_schema_v2.xml`

### <a name="configuration"></a>構成

**IIS**

* `%windir%\System32\inetsrv\config\applicationHost.config`

**IIS Express**

* Visual Studio: `{APPLICATION ROOT}\.vs\config\applicationHost.config`

* *iisexpress.exe* CLI: `%USERPROFILE%\Documents\IISExpress\config\applicationhost.config`

ファイルは、`applicationHost.config` ファイルで `aspnetcore` を検索することにより見つかります。
