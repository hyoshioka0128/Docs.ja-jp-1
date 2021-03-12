---
title: ASP.NET Core での SMS による2要素認証
author: rick-anderson
description: ASP.NET Core アプリで2要素認証 (2FA) を設定する方法について説明します。
monikerRange: < aspnetcore-2.0
ms.author: riande
ms.date: 09/22/2018
ms.custom: mvc, seodec18
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
uid: security/authentication/2fa
ms.openlocfilehash: 1f77f3f4b7e9dd558e9869992e2f1f4d185e5b10
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586853"
---
# <a name="two-factor-authentication-with-sms-in-aspnet-core"></a>ASP.NET Core での SMS による2要素認証

By [Rick Anderson](https://twitter.com/RickAndMSFT) および [スイス-開発者](https://github.com/Swiss-Devs)

>[!WARNING]
> 2要素認証 (2FA) 認証アプリは、時間ベースのワンタイムパスワードアルゴリズム (TOTP) を使用して、2FA に業界で推奨されるアプローチです。 TOTP を使用した2FA は、SMS 2FA に適しています。 詳細については、ASP.NET Core 2.0 以降の [ASP.NET Core での TOTP authenticator アプリの QR コード生成の有効化](xref:security/authentication/identity-enable-qrcodes) に関する説明を参照してください。

このチュートリアルでは、SMS を使用して2要素認証 (2FA) を設定する方法について説明します。 手順については、 [twilio](https://www.twilio.com/) と [aspsms](https://www.aspsms.com/asp.net/identity/core/testcredits/)を参照してください。ただし、他の sms プロバイダーを使用することもできます。 このチュートリアルを開始する前に [、アカウントの確認とパスワードの回復](xref:security/authentication/accconfirm) を完了することをお勧めします。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/2fa/sample/Web2FA)します。 [ダウンロード方法](xref:index#how-to-download-a-sample)。

## <a name="create-a-new-aspnet-core-project"></a>新しい ASP.NET Core プロジェクトを作成する

`Web2FA`個々のユーザーアカウントを使用して、という名前の新しい ASP.NET Core web アプリを作成します。 「」の手順に従っ <xref:security/enforcing-ssl> て、HTTPS を設定し、要求します。

### <a name="create-an-sms-account"></a>SMS アカウントの作成

たとえば、 [twilio](https://www.twilio.com/) や [ASPSMS](https://www.aspsms.com/asp.net/identity/core/testcredits/)などの sms アカウントを作成します。 認証資格情報 (ASPSMS の場合は twilio: accountSid および authToken の場合はユーザーキーとパスワード) を記録します。

#### <a name="figuring-out-sms-provider-credentials"></a>SMS プロバイダーの資格情報の確認

**Twilio**

Twilio アカウントの [ダッシュボード] タブで、 **アカウント SID** と **認証トークン** をコピーします。

**ASPSMS:**

アカウントの設定から **ユーザーキー** に移動し、 **パスワード** と共にコピーします。

これらの値は、後で、キーと内の secret manager ツールを使用してに格納し `SMSAccountIdentification` `SMSAccountPassword` ます。

#### <a name="specifying-senderid--originator"></a>SenderID/オリジネータの指定

**Twilio:** [数値] タブで、Twilio **電話番号** をコピーします。

**Aspsms:** [発信元のロック解除] メニュー内で、1つ以上の発信者をロック解除するか、英数字の発信者を選択します (すべてのネットワークでサポートされていません)。

この値は、後でキー内にシークレットマネージャーツールで格納し `SMSAccountFrom` ます。

### <a name="provide-credentials-for-the-sms-service"></a>SMS サービスの資格情報を指定する

[オプションパターン](xref:fundamentals/configuration/options)を使用して、ユーザーアカウントとキーの設定にアクセスします。

* セキュリティで保護された SMS キーを取得するクラスを作成します。 このサンプルでは、 `SMSoptions` *サービス/smsoptions .cs* ファイルにクラスを作成します。

[!code-csharp[](2fa/sample/Web2FA/Services/SMSoptions.cs)]

、、 `SMSAccountIdentification` `SMSAccountPassword` およびを `SMSAccountFrom` [secret manager ツール](xref:security/app-secrets)を使用して設定します。 次に例を示します。

```none
C:/Web2FA/src/WebApp1>dotnet user-secrets set SMSAccountIdentification 12345
info: Successfully saved SMSAccountIdentification = 12345 to the secret store.
```

* SMS プロバイダーの NuGet パッケージを追加します。 パッケージマネージャーコンソール (PMC) から次のように実行します。

**Twilio**

`Install-Package Twilio`

**ASPSMS:**

`Install-Package ASPSMS`

* *サービス/MessageServices .cs* ファイルにコードを追加して、SMS を有効にします。 Twilio または ASPSMS セクションのいずれかを使用します。

**Twilio**  
[!code-csharp[](2fa/sample/Web2FA/Services/MessageServices_twilio.cs)]

**ASPSMS:**  
[!code-csharp[](2fa/sample/Web2FA/Services/MessageServices_ASPSMS.cs)]

### <a name="configure-startup-to-use-smsoptions"></a>使用するスタートアップの構成 `SMSoptions`

`SMSoptions`Startup.cs のメソッドで、サービスコンテナーにを追加し `ConfigureServices` ます。 

[!code-csharp[](2fa/sample/Web2FA/Startup.cs?name=snippet1&highlight=4)]

### <a name="enable-two-factor-authentication"></a>2 要素認証を有効にする

*Views/Manage/Index. cshtml* Razor ビューファイルを開き、コメント文字を削除します (マークアップがコメントアウトされることはありません)。

## <a name="log-in-with-two-factor-authentication"></a>2要素認証を使用してログインする

* アプリを実行して新しいユーザーを登録する

![Microsoft Edge で開いている Web アプリケーション登録ビュー](2fa/_static/login2fa1.png)

* [コントローラーの管理] でアクションメソッドをアクティブにするユーザー名をタップし `Index` ます。 次に、[電話番号の **追加** ] リンクをタップします。

![[管理] の [追加] リンク](2fa/_static/login2fa2.png)

* 確認コードを受信する電話番号を追加し、[ **確認コードの送信**] をタップします。

![[電話番号の追加] ページ](2fa/_static/login2fa3.png)

* 確認コードを含むテキストメッセージが表示されます。 入力し、[**送信**] をタップします。

![[電話番号の確認] ページ](2fa/_static/login2fa4.png)

テキストメッセージが表示されない場合は、「twilio log page」を参照してください。

* 管理ビューには、電話番号が正常に追加されたことが表示されます。

![ビューの管理-電話番号が正常に追加されました](2fa/_static/login2fa5.png)

* [ **有効** ] をタップして、2要素認証を有効にします。

![ビューの管理-2 要素認証の有効化](2fa/_static/login2fa6.png)

### <a name="test-two-factor-authentication"></a>2要素認証をテストする

* ログオフします。

* ログインします。

* ユーザーアカウントで2要素認証が有効になっているため、2番目の認証要素を指定する必要があります。 このチュートリアルでは、電話による検証を有効にしました。 組み込みのテンプレートでは、2番目の要素として電子メールを設定することもできます。 QR コードなどの認証については、さらに2つ目の要素を設定できます。 [ **Submit (送信**)」をタップします。

![確認コードビューの送信](2fa/_static/login2fa7.png)

* SMS メッセージで取得したコードを入力します。

* [ **このブラウザーを記憶** する] チェックボックスをオンにすると、同じデバイスとブラウザーを使用しているときに2fa を使用してログオンする必要がありません。 2FA を有効にして [ **このブラウザーを記憶** する] をオンにすると、悪意のあるユーザーが自分のデバイスにアクセスできなくても、アカウントへのアクセスを試みる悪意のあるユーザーからの強力な2fa 保護が提供されます。 この操作は、定期的に使用するプライベートデバイスで行うことができます。 [  **このブラウザーを記憶** する] を設定すると、定期的に使用しないデバイスから2fa のセキュリティが強化され、自分のデバイスで2fa を通過する必要がなくなります。

![ビューの確認](2fa/_static/login2fa8.png)

## <a name="account-lockout-for-protecting-against-brute-force-attacks"></a>ブルートフォース攻撃から保護するためのアカウントロックアウト

アカウントのロックアウトは2FA で推奨されます。 ユーザーがローカルアカウントまたはソーシャルアカウントを使用してサインインすると、2FA に失敗した各操作が格納されます。 失敗したアクセス試行の最大回数に達した場合、ユーザーはロックアウトされます (既定では5分間のロックアウトが失敗し、アクセス試行が5回失敗します)。 認証が成功すると、失敗したアクセス試行回数がリセットされ、時計がリセットされます。 失敗したアクセス試行の最大回数とロックアウト時間は、 [Maxfailed Accessattempts](/dotnet/api/microsoft.aspnetcore.identity.lockoutoptions.maxfailedaccessattempts) と [Defaultlockouttimespan](/dotnet/api/microsoft.aspnetcore.identity.lockoutoptions.defaultlockouttimespan)で設定できます。 次の例では、10分間に失敗したアクセス試行の10分後にアカウントのロックアウトを構成します。

[!code-csharp[](2fa/sample/Web2FA/Startup.cs?name=snippet2&highlight=13-17)]

[PasswordSignInAsync](/dotnet/api/microsoft.aspnetcore.identity.signinmanager-1.passwordsigninasync)がに設定されていること `lockoutOnFailure` を確認し `true` ます。

```csharp
var result = await _signInManager.PasswordSignInAsync(
                 Input.Email, Input.Password, Input.RememberMe, lockoutOnFailure: true);
```
