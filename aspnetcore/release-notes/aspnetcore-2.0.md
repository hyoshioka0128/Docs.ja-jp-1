---
title: ASP.NET Core 2.0 の新機能
author: rick-anderson
description: ASP.NET Core 2.0 の新機能について説明します。
ms.author: riande
ms.custom: mvc
ms.date: 12/05/2019
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
uid: aspnetcore-2.0
ms.openlocfilehash: b7515bcd8b15199770a4245469d00d10da5566f8
ms.sourcegitcommit: acfe51c35497a204f75c2a61125c9408c04493e6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102605687"
---
# <a name="whats-new-in-aspnet-core-20"></a>ASP.NET Core 2.0 の新機能

この記事では、ASP.NET Core 2.0 の最も大きな変更点について説明します。また、その変更点のドキュメントへのリンクも示します。

## <a name="razor-pages"></a>Razor Pages

Razor Pages は、ページ コーディングに重点を置いたシナリオをより簡略化し、生産性を高める ASP.NET Core MVC の新機能です。

詳細については、概要とチュートリアルを参照してください。

* [Razor Pages の概要](xref:razor-pages/index)
* [Razor Pages の使用を開始する](xref:tutorials/razor-pages/razor-pages-start)

## <a name="aspnet-core-metapackage"></a>ASP.NET Core メタパッケージ

新しい ASP.NET Core メタパッケージには、ASP.NET Core および Entity Framework Core チームでサポートされているすべてのパッケージが、内部およびサードパーティの依存関係と共に含まれています。 これにより、パッケージの ASP.NET Core の個別機能を選択する必要がなくなりました。 すべての機能は、[Microsoft.AspNetCore.All](https://www.nuget.org/packages/Microsoft.AspNetCore.All) パッケージに含まれています。 このパッケージは、既定のテンプレートで使用されています。

詳細については、「[Microsoft.AspNetCore.All metapackage for ASP.NET Core 2.0](xref:fundamentals/metapackage)」 (ASP.NET Core 2.0 用の Microsoft.AspNetCore.All メタパッケージ) を参照してください。

## <a name="runtime-store"></a>ランタイム ストア

`Microsoft.AspNetCore.All` メタパッケージを使用するアプリケーションでは、新しい .NET Core ランタイム ストアが自動的に利用されます。 このストアには、ASP.NET Core 2.0 アプリケーションの実行に必要なすべてのランタイム アセットが含まれています。 `Microsoft.AspNetCore.All` メタパッケージを使用する場合、参照される ASP.NET Core NuGet パッケージの資産は、既にターゲット システム上にあるため、アプリケーションと共に配置されません。 ランタイム ストア内の資産は、アプリケーションの起動時間を向上させるためにプリコンパイルもされています。

詳細については、「[Runtime Store](/dotnet/core/deploying/runtime-store)」 (ランタイム ストア) を参照してください。

## <a name="net-standard-20"></a>.NET Standard 2.0

ASP.NET Core 2.0 パッケージは、.NET Standard 2.0 を対象としています。 このパッケージは、その他の .NET Standard 2.0 ライブラリから参照でき、.NET Core 2.0 と.NET Framework 4.6.1 を含む、.NET Standard 2.0 に準拠した実装で実行できます。 

`Microsoft.AspNetCore.All` メタパッケージは、.NET Core 2.0 ランタイム ストアで使用することを意図しており、.NET Core 2.0 のみを対象としています。

## <a name="configuration-update"></a>構成の更新

ASP.NET Core 2.0 では、既定で `IConfiguration` インスタンスがサービス コンテナーに追加されています。 サービス コンテナーの `IConfiguration` では、アプリケーションがコンテナーから構成値を取得するのを容易にします。

計画されているドキュメントの状態については、「[GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/3387)」 (GitHub の問題) を参照してください。

## <a name="logging-update"></a>ログ記録の更新

ASP.NET Core 2.0 には、既定で依存性の注入 (DI) システムにログ記録が組み込まれています。 *Startup.cs* ファイルではなく *Program.cs* ファイルに、プロバイダーを追加しフィルター処理を構成できます。 また、既定の `ILoggerFactory` では、プロバイダーをまたがるフィルター処理と特定のプロバイダーのフィルター処理の両方で、ある柔軟なアプローチを使用するフィルター処理をサポートしています。

詳細については、「[Introduction to Logging](xref:fundamentals/logging/index)」 (ログ記録の概要) を参照してください。

## <a name="authentication-update"></a>認証の更新

新しい認証モデルにより、DI を使用するアプリケーションでの認証の構成が容易になりました。

新しいテンプレートを使うと、[Azure AD B2C](https://azure.microsoft.com/services/active-directory-b2c/) を使って Web アプリと Web API に対する認証を構成できます。

計画されているドキュメントの状態については、「[GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/3054)」 (GitHub の問題) を参照してください。

## <a name="identity-update"></a>Identity の更新

ASP.NET Core 2.0 で Identity を使用し、簡単にセキュリティで保護された Web API をビルドできるようになりました。 [Microsoft 認証ライブラリ (MSAL)](https://www.nuget.org/packages/Microsoft.Identity.Client) を使用して Web API にアクセスするアクセス トークンを取得することができます。

2\.0 での認証の変更の詳細については、次のリソースを参照してください。

* [ASP.NET Core でのアカウントの確認とパスワードの回復](xref:security/authentication/accconfirm)
* [ASP.NET Core の認証アプリでの QR コードの生成の有効化](xref:security/authentication/identity-enable-qrcodes)
* [ASP.NET Core 2.0 への認証と Identity の移行](xref:migration/1x-to-2x/identity-2x)

## <a name="spa-templates"></a>SPA テンプレート

Angular、Aurelia、Knockout.js、React.js、Redux 用 React.js で、シングル ページ アプリケーション (SPA) プロジェクト テンプレートが利用できるようになりました。 Angular テンプレートは、Angular 4 に更新されました。 Angular と React テンプレートは既定で使用可能です。その他のテンプレートの入手方法については、[新しい SPA プロジェクトを作成する](xref:client-side/spa-services#create-a-new-project)方法に関するページを参照してください。 ASP.NET Core で SPA を構築する方法については、<xref:client-side/spa-services> を参照してください。

## <a name="kestrel-improvements"></a>Kestrel の機能強化

Kestrel Web サーバーを、インターネット接続用サーバーとして使用するのにより適切にする新機能が追加されました。 `KestrelServerOptions` クラスの新しい `Limits` プロパティに、多数のサーバー制約構成が追加されました。 次に制限を追加できるようになりました。

* クライアントの最大接続数
* 要求本文の最大サイズ
* 要求本文の最小レート

詳細については、「[Kestrel web server implementation in ASP.NET Core](xref:fundamentals/servers/kestrel)」 (ASP.NET Core への Kestrel Web サーバーの実装) を参照してください。

## <a name="weblistener-renamed-to-httpsys"></a>WebListener から HTTP.sys への名称変更

パッケージ、`Microsoft.AspNetCore.Server.WebListener` と `Microsoft.Net.Http.Server` が新しい `Microsoft.AspNetCore.Server.HttpSys` パッケージにマージされました。 一致するように名前空間が更新されました。

詳細については、「[HTTP.sys web server implementation in ASP.NET Core](xref:fundamentals/servers/httpsys)」 (ASP.NET Core への HTTP.sys Web サーバーの実装) を参照してください。

## <a name="enhanced-http-header-support"></a>HTTP ヘッダーのサポートの強化

MVC を使用して `FileStreamResult` または `FileContentResult` を送信する場合、送信するコンテンツに `ETag` または `LastModified` 日付を設定するオプションが追加されました。 次のようなコードで、返されるコンテンツにこれらの値を設定できます。

```csharp
var data = Encoding.UTF8.GetBytes("This is a sample text from a binary array");
var entityTag = new EntityTagHeaderValue("\"MyCalculatedEtagValue\"");
return File(data, "text/plain", "downloadName.txt", lastModified: DateTime.UtcNow.AddSeconds(-5), entityTag: entityTag);
```

ユーザーに返されるファイルには、`ETag` と `LastModified` 値に適切な HTTP ヘッダーが含まれます。

アプリケーションのユーザーが範囲要求ヘッダーを使用してコンテンツを要求する場合、ASP.NET Core は要求を認識して、そのヘッダーを処理します。 要求されたコンテンツを部分的に配信できる場合、ASP.NET Core は要求されたバイトのセットだけを適切に省略して返します。 この機能を適用または処理するために、メソッドに特別なハンドラーを記述する必要はありません。自動的に処理されます。

## <a name="hosting-startup-and-application-insights"></a>スタートアップのホストと Application Insights

ホスティング環境で、アプリケーションが明示的に依存関係を取得したり、任意のメソッドを呼び出したりせずに、余分なパッケージの依存関係を挿入して、アプリケーションの起動時にコードを実行することができるようになりました。 この機能は、特定の環境でその環境に固有の機能を、アプリケーションが事前に認識する必要なく、明らかにできるようにするために使用できます。 

ASP.NET Core 2.0 では、Visual Studio でのデバッグ時および Azure App Services での実行時 (有効化後に) に、Application Insights での診断を自動的に有効にするために、この機能が使用されています。 その結果、プロジェクト テンプレートでは Application Insights のパッケージとコードが既定で追加されなくなりました。

計画されているドキュメントの状態については、「[GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/3389)」 (GitHub の問題) を参照してください。

## <a name="automatic-use-of-anti-forgery-tokens"></a>偽造防止トークンの自動使用

ASP.NET Core では、常にコンテンツの HTML エンコードを既定で支援してきましたが、新しいバージョンでは、クロスサイト リクエスト フォージェリ (XSRF) 攻撃を防ぐための手段もさらに講じられています。 ASP.NET Core では、既定で偽造防止トークンが生成されるようになり、追加の構成をしなくても、フォーム POST アクションとページでそれらが検証されるようになりました。

詳細については、「<xref:security/anti-request-forgery>」を参照してください。

## <a name="automatic-precompilation"></a>自動でのプリコンパイル

Razor ビューのプリコンパイルは発行時に既定で有効になっています。これにより、発行の出力サイズが縮小され、アプリケーションの起動時間が削減されます。

詳細については、[ASP.NET Core での Razor ビューのコンパイルとプリコンパイル](xref:mvc/views/view-compilation)に関するページを参照してください。

## <a name="razor-support-for-c-71"></a>C# 7.1 での Razor サポート

Razor ビュー エンジンが更新され、新しい Roslyn コンパイラで使用できるようになりました。 これには、既定の式、推定タプル名、およびジェネリック パターン マッチングのような C# 7.1 の機能のサポートが含まれています。 プロジェクトで C# 7.1 を使用する場合、プロジェクト ファイルに次のプロパティを追加し、ソリューションを再読み込みします。

```xml
<LangVersion>latest</LangVersion>
```

C# 7.1 機能の状態については、「[Roslyn GitHub repository](https://github.com/dotnet/roslyn/blob/main/docs/Language%20Feature%20Status.md)」 (Roslyn GitHub リポジトリ) を参照してください。

## <a name="other-documentation-updates-for-20"></a>2\.0 のその他のドキュメントの更新

* [ASP.NET Core アプリ開発のための Visual Studio によるプロファイルの発行](xref:host-and-deploy/visual-studio-publish-profiles)
* [キーの管理](xref:security/data-protection/implementation/key-management)
* [Facebook 認証の構成](xref:security/authentication/facebook-logins)
* [Twitter 認証の構成](xref:security/authentication/twitter-logins)
* [Googler 認証の構成](xref:security/authentication/google-logins)
* [Microsoft アカウント認証の構成](xref:security/authentication/microsoft-logins)

## <a name="migration-guidance"></a>移行ガイダンス

ASP.NET Core 1.x アプリケーションを ASP.NET Core 2.0 に移行する方法の手順については、次のリソースを参照してください。

* [ASP.NET Core 1.x から ASP.NET Core 2.0 への移行](xref:migration/1x-to-2x/index)
* [ASP.NET Core 2.0 への認証と Identity の移行](xref:migration/1x-to-2x/identity-2x)

## <a name="additional-information"></a>追加情報

変更の全一覧については、「[ASP.NET Core 2.0 Release Notes](https://github.com/dotnet/aspnetcore/releases/tag/2.0.0)」 (ASP.NET Core 2.0 のリリース ノート) を参照してください。

ASP.NET Core 開発チームの進捗状況や計画とつながるには、週次の [ASP.NET Community Standup](https://live.asp.net/) にアクセスしてください。
