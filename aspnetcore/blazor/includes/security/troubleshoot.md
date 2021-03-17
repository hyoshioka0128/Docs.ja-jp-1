---
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
ms.openlocfilehash: f58da78475d65cbb70b0b177e1b0443535e97d55
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/06/2021
ms.locfileid: "102402303"
---
## <a name="troubleshoot"></a>トラブルシューティング

### <a name="common-errors"></a>一般的なエラー

* アプリまたは Identity プロバイダー (IP) の構成の誤り

  最も一般的なエラーの原因は、構成の誤りです。 以下に例を示します。
  
  * シナリオの要件によっては、権限、インスタンス、テナント ID、テナント ドメイン、クライアント ID、またはリダイレクト URI の欠落または誤りによって、アプリによるクライアントの認証ができなくなります。
  * アクセス トークン スコープが正しくないと、クライアントがサーバー Web API エンドポイントにアクセスできなくなります。
  * サーバー API のアクセス許可が正しくないか、存在しないと、クライアントがサーバー Web API エンドポイントにアクセスできなくなります。
  * Identity プロバイダーのアプリ登録のリダイレクト URI で構成されているものとは異なるポートでアプリが実行されています。
  
  この記事のガイダンスの構成セクションに、正しい構成の例を示します。 記事の各セクションを慎重に確認して、アプリと IP の構成の誤りを探してください。
  
  構成が正しい場合:
  
  * アプリケーション ログを分析します。
  * ブラウザーの開発者ツールを使用して、クライアント アプリと IP またはサーバー アプリの間のネットワーク トラフィックを確認します。 多くの場合、要求を行った後、IP またはサーバー アプリによって、問題の原因を特定する手掛かりを含む正確なエラー メッセージまたはメッセージがクライアントに返されます。 開発者ツールのガイダンスは、次の記事にあります。

    * [Google Chrome](https://developers.google.com/web/tools/chrome-devtools/network) (Google ドキュメント)
    * [Microsoft Edge](/microsoft-edge/devtools-guide-chromium/network/)
    * [Mozilla Firefox](https://developer.mozilla.org/docs/Tools/Network_Monitor) (Mozilla ドキュメント)

  * 問題が発生している場所に応じて、クライアントの認証またはサーバー Web API へのアクセスに使用される JSON Web Token (JWT) の内容をデコードします。 詳細については、「[JSON Web トークン (JWT) の内容を検査する](#inspect-the-content-of-a-json-web-token-jwt)」を参照してください。
  
  ドキュメント チームは、ドキュメントのフィードバックと記事のバグについては対応します (**こちらのページ** のフィードバック セクションからイシューを作成してください) が、製品サポートを提供することはできません。 アプリのトラブルシューティングに役立つ、いくつかのパブリック サポート フォーラムが用意されています。 次をお勧めします。
  
  * [Stack Overflow (タグ: `blazor`)](https://stackoverflow.com/questions/tagged/blazor)
  * [ASP.NET Core Slack Team](http://tattoocoder.com/aspnet-slack-sign-up/)
  * [Blazor Gitter](https://gitter.im/aspnet/Blazor)
  
  セキュリティで保護されておらず、機密でも社外秘でもない再現可能なフレームワークのバグ レポートについては、[ASP.NET Core 製品単位でイシューを作成してください](https://github.com/dotnet/aspnetcore/issues)。 問題の原因を徹底的に調査し、パブリック サポート フォーラムのコミュニティの助けを借りてもお客様自身で解決できない場合にのみ、製品単位でイシューを作成してください。 単純な構成の誤りやサードパーティのサービスに関連するユース ケースによって破損した個々のアプリのトラブルシューティングは、製品単位で行うことはできません。 レポートが機密性の高い性質のものでる場合や、攻撃者が悪用するおそれのある製品の潜在的なセキュリティ上の欠陥が記述されている場合は、[セキュリティの問題とバグの報告 (dotnet/Aspnetcore GitHub リポジトリ)](https://github.com/dotnet/aspnetcore/blob/main/CONTRIBUTING.md#reporting-security-issues-and-bugs) を参照してください。

::: moniker range=">= aspnetcore-5.0"

* AAD で承認されないクライアント

  > 情報:Microsoft.AspNetCore.Authorization.DefaultAuthorizationService[2] 承認に失敗しました。 次の要件が満たされていません。DenyAnonymousAuthorizationRequirement:認証済みユーザーが必要です。

  AAD からのログイン コールバック エラー:

  * エラー: `unauthorized_client`
  * 説明: `AADB2C90058: The provided application is not configured to allow public clients.`

  このエラーを解決するには:

  1. Azure portal で、[アプリのマニフェスト](/azure/active-directory/develop/reference-app-manifest)にアクセスします。
  1. [`allowPublicClient` 属性](/azure/active-directory/develop/reference-app-manifest#allowpublicclient-attribute)を `null` または `true` に設定します。

::: moniker-end

### <a name="cookies-and-site-data"></a>Cookie とサイト データ

Cookie とサイト データは、アプリが更新されても保持され、テストやトラブルシューティングに影響する可能性があります。 アプリ コードの変更、プロバイダーによるユーザー アカウントの変更、プロバイダー アプリの構成変更を行うときは、次のものをクリアしてください。

* ユーザー サインイン cookie
* アプリ cookie
* キャッシュおよび保存されたサイト データ

残った cookie とサイト データがテストとトラブルシューティングに影響しないようにする方法を、次に示します。

* ブラウザーを構成する
  * ブラウザーが閉じるたびに cookie とサイト データをすべて削除するように構成できることをテストするために、ブラウザーを使用します。
  * アプリ、テスト ユーザー、プロバイダー構成が変更されるたびにブラウザーが手動で、または IDE によって閉じられていることを確認します。
* カスタム コマンドを使用して、Visual Studio でブラウザーをシークレットまたはプライベート モードで開く。
  * Visual Studio の **[実行]** ボタンをクリックして **[ブラウザーの選択]** ダイアログボックスを開きます。
  * **[追加]** ボタンを選びます。
  * **[プログラム]** フィールドでブラウザーのパスを指定します。 次の実行可能パスが、Windows 10 の一般的なインストール場所です。 ブラウザーが別の場所にインストールされている場合、または Windows 10 を使用していない場合は、ブラウザーの実行可能ファイルのパスを指定してください。
    * Microsoft Edge: `C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe`
    * Google Chrome: `C:\Program Files (x86)\Google\Chrome\Application\chrome.exe`
    * Mozilla Firefox: `C:\Program Files\Mozilla Firefox\firefox.exe`
  * **[引数]** フィールドに、ブラウザーをシークレットまたはプライベート モードで開くために使用するコマンドライン オプションを指定します。 ブラウザーによっては、アプリの URL が必要になる場合があります。
    * Microsoft Edge:`-inprivate` を使用してください。
    * Google Chrome:`--incognito --new-window {URL}` を使用します。プレースホルダー `{URL}` は開く URL (`https://localhost:5001` など) です。
    * Mozilla Firefox:`-private -url {URL}` を使用します。プレースホルダー `{URL}` は開く URL (`https://localhost:5001` など) です。
  * **[フレンドリ名]** フィールドに名前を指定します。 たとえば、`Firefox Auth Testing` のようにします。
  * **[OK]** ボタンを選択します。
  * アプリでテストを繰り返すたびにブラウザー プロファイルを選択する必要がないようにするには、 **[既定値として設定]** ボタンでプロファイルを既定値として設定します。
  * アプリ、テスト ユーザー、またはプロバイダー構成が変更されるたびに、ブラウザーが IDE によって閉じられていることを確認します。

### <a name="app-upgrades"></a>アプリのアップグレード

開発マシンで .NET Core SDK をアップグレードしたり、アプリ内のパッケージ バージョンを変更したりした直後に、機能しているアプリが失敗することがあります。 場合によっては、パッケージに統一性がないと、メジャー アップグレード実行時にアプリが破壊されることがあります。 これらの問題のほとんどは、次の手順で解決できます。

1. コマンド シェルから [`dotnet nuget locals all --clear`](/dotnet/core/tools/dotnet-nuget-locals) を実行して、ローカル システムの NuGet パッケージ キャッシュをクリアします。
1. プロジェクトのフォルダー `bin` と `obj` を削除します。
1. プロジェクトを復元してリビルドします。
1. アプリを再展開する前に、サーバー上の展開フォルダー内のすべてのファイルを削除します。

> [!NOTE]
> アプリのターゲット フレームワークと互換性のないパッケージ バージョンの使用はサポートされていません。 パッケージの詳細については、[NuGet ギャラリー](https://www.nuget.org)または [FuGet パッケージ エクスプローラー](https://www.fuget.org)を使用してください。

### <a name="run-the-server-app"></a>Server アプリを実行する

ホステッド Blazor ソリューションのテストとトラブルシューティングを行うときは、 **`Server`** プロジェクトからアプリを実行していることをご確認ください。 たとえば、Visual Studio で、次のいずれかの方法を使用してアプリを起動する前に、Server プロジェクトが **ソリューション エクスプローラー** で強調表示されていることを確認します。

* **[実行]** ボタンを選択します。
* メニューの、 **[デバッグ]**  >  **[デバッグ開始]** を使用します。
* <kbd>F5</kbd>キーを押します。

### <a name="inspect-the-content-of-a-json-web-token-jwt"></a>JSON Web トークン (JWT) の内容を検査する

JSON Web トークン (JWT) をデコードするには、Microsoft の [jwt.ms](https://jwt.ms/) ツールを使用します。 UI の値がブラウザーに残ることはありません。

エンコードされた JWT の例 (表示用に短縮されています):

> eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Ilg1ZVhrNHh5b2pORnVtMWtsMll0djhkbE5QNC1j ... bQdHBHGcQQRbW7Wmo6SWYG4V_bU55Ug_PW4pLPr20tTS8Ct7_uwy9DWrzCMzpD-EiwT5IjXwlGX3IXVjHIlX50IVIydBoPQtadvT7saKo1G5Jmutgq41o-dmz6-yBMKV2_nXA25Q

Azure AAD B2C に対して認証するアプリのツールによってデコードされた JWT の例:

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "X5eXk4xyojNFum1kl2Ytv8dlNP4-c57dO6QGTVBwaNk"
}.{
  "exp": 1610059429,
  "nbf": 1610055829,
  "ver": "1.0",
  "iss": "https://mysiteb2c.b2clogin.com/5cc15ea8-a296-4aa3-97e4-226dcc9ad298/v2.0/",
  "sub": "5ee963fb-24d6-4d72-a1b6-889c6e2c7438",
  "aud": "70bde375-fce3-4b82-984a-b247d823a03f",
  "nonce": "b2641f54-8dc4-42ca-97ea-7f12ff4af871",
  "iat": 1610055829,
  "auth_time": 1610055822,
  "idp": "idp.com",
  "tfp": "B2C_1_signupsignin"
}.[Signature]
```
