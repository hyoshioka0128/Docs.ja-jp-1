---
title: cookieASP.NET Core での SameSite s の使用
author: rick-anderson
description: を SameSite ASP.NET Core s に使用する方法について説明します。 cookie
ms.author: riande
ms.custom: mvc
ms.date: 12/03/2019
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
- Electron
uid: security/samesite
ms.openlocfilehash: e06c73edfc999053e0aa37f05d984a2b428f69a9
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102109937"
---
# <a name="work-with-samesite-cookies-in-aspnet-core"></a>cookieASP.NET Core での SameSite s の使用

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

SameSite は、クロスサイトリクエスト偽造 (CSRF) 攻撃に対して何らかの保護を提供するように設計された [IETF](https://ietf.org/about/) ドラフト標準です。 最初は [2016](https://tools.ietf.org/html/draft-west-first-party-cookies-07)でドラフトされていましたが、draft standard は [2019](https://tools.ietf.org/html/draft-west-cookie-incrementalism-00)で更新されました。 更新された標準は以前の標準との下位互換性はありませんが、次のように最も顕著な違いがあります。

* CookieSameSite ヘッダーのない s は、既定でとして扱われ `SameSite=Lax` ます。
* `SameSite=None` クロスサイトの使用を許可するには、を使用する必要があり cookie ます。
* Cookieアサートするは、 `SameSite=None` としてもマークされている必要があり `Secure` ます。
* を使用するアプリケーションは、 [`<iframe>`](https://developer.mozilla.org/docs/Web/HTML/Element/iframe) `sameSite=Lax` `sameSite=Strict` cookie `<iframe>` クロスサイトシナリオとして扱われるため、またはに問題が発生する可能性があります。
* 値 `SameSite=None` は [2016 標準](https://tools.ietf.org/html/draft-west-first-party-cookies-07) では許可されていないため、一部の実装ではとしてを処理し cookie `SameSite=Strict` ます。 このドキュメントの「 [古いブラウザーのサポート](#sob) 」を参照してください。

この `SameSite=Lax` 設定は、ほとんどのアプリケーションで機能し cookie ます。 [OpenID connect](https://openid.net/connect/) (oidc) や[ws-federation](https://auth0.com/docs/protocols/ws-fed)のような認証形式では、既定で POST ベースのリダイレクトが設定されます。 POST ベースのリダイレクトによって SameSite ブラウザーの保護がトリガーされるため、これらのコンポーネントの SameSite は無効になります。 ほとんどの [OAuth](https://oauth.net/) ログインは、要求フローの違いによって影響を受けません。

を出力する各 ASP.NET Core コンポーネント cookie は、SameSite が適切かどうかを判断する必要があります。

## <a name="samesite-and-identity"></a>SameSite と Identity

[!INCLUDE[](~/includes/SameSiteIdentity.md)]

## <a name="samesite-test-sample-code"></a>SameSite テストサンプルコード

::: moniker range=">= aspnetcore-2.1 < aspnetcore-3.0"

次のサンプルをダウンロードしてテストできます。

| サンプル               | ドキュメント |
| ----------------- | ------------ |
| [.NET Core MVC](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21MVC)  | <xref:security/samesite/mvc21> |
| [.NET Core Razor ページ](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21RazorPages)  | <xref:security/samesite/rp21> |

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

次のサンプルをダウンロードしてテストできます。

| サンプル               | ドキュメント |
| ----------------- | ------------ |
| [.NET Core Razor ページ](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore31RazorPages)  | <xref:security/samesite/rp31> |

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

## <a name="net-core-support-for-the-samesite-attribute"></a>SameSite 属性の .NET Core のサポート

.NET Core 2.2 以降では、年 12 2019 月の更新プログラムのリリース以降、SameSite の2019ドラフト標準がサポートされています。 開発者は、プロパティを使用して、sameSite 属性の値をプログラムで制御でき `HttpCookie.SameSite` ます。 `SameSite`プロパティを Strict、厳密でない、または None に設定すると、これらの値はと共にネットワーク上に書き込まれ cookie ます。 これをに設定すると `(SameSiteMode)(-1)` 、sameSite 属性をネットワークに含める必要があることを示します。 cookie

[!code-csharp[](samesite/snippets/Privacy.cshtml.cs?name=snippet)]

.NET Core 3.0 以降では、更新された SameSite 値がサポートされ、列挙型に追加の列挙値が追加され `SameSiteMode.Unspecified` `SameSiteMode` ます。
この新しい値は、で sameSite が送信されないことを示し cookie ます。

::: moniker-end

::: moniker range="= aspnetcore-2.1"

## <a name="december-patch-behavior-changes"></a>12月のパッチ動作の変更

.NET Framework と .NET Core 2.1 の特定の動作変更は、 `SameSite` プロパティが値を解釈する方法です `None` 。 パッチの前に、 `None` "属性をまったく出力しない" という値が適用される前に、パッチの後に "値を持つ属性を生成する" という意味になり `None` ます。 Patch の後に `SameSite` の値を指定する `(SameSiteMode)(-1)` と、属性は出力されません。

フォーム認証およびセッション状態 s の既定の SameSite 値は、 cookie からに変更されました `None` `Lax` 。

::: moniker-end

## <a name="api-usage-with-samesite"></a>SameSite を使用した API の使用

[HttpContext Cookie 。s. 追加](xref:Microsoft.AspNetCore.Http.IResponseCookies.Append*) の既定値はです。 `Unspecified` つまり、に SameSite 属性が追加されず、クライアントは既定の動作を使用します cookie (新しいブラウザーではなく、古いブラウザーではありません)。 次のコードは、SameSite の値をに変更する方法を示してい cookie `SameSiteMode.Lax` ます。

[!code-csharp[](samesite/sample/Pages/Index.cshtml.cs?name=snippet)]

S を出力するすべて cookie の ASP.NET Core コンポーネントは、前述の既定値よりも、そのシナリオに適した設定を上書きします。 オーバーライドされた前の既定値は変更されていません。

| コンポーネント | cookie | Default |
| ------------- | ------------- |
| <xref:Microsoft.AspNetCore.Http.CookieBuilder> | <xref:Microsoft.AspNetCore.Http.CookieBuilder.SameSite> | `Unspecified` |
| <xref:Microsoft.AspNetCore.Http.HttpContext.Session>  | [SessionOptions。Cookie](xref:Microsoft.AspNetCore.Builder.SessionOptions.Cookie) |`Lax` |
| <xref:Microsoft.AspNetCore.Mvc.ViewFeatures.CookieTempDataProvider>  | [CookieTempDataProviderOptions.Cookie](xref:Microsoft.AspNetCore.Mvc.CookieTempDataProviderOptions.Cookie) | `Lax` |
| <xref:Microsoft.AspNetCore.Antiforgery.IAntiforgery> | [アンチ Forgeryoptions。Cookie](xref:Microsoft.AspNetCore.Antiforgery.AntiforgeryOptions.Cookie)| `Strict` |
| [Cookie 認証](xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*) | [CookieAuthenticationOptions.Cookie](xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions.CookieName) | `Lax` |
| <xref:Microsoft.Extensions.DependencyInjection.TwitterExtensions.AddTwitter*> | [CookieTwitterOptions](xref:Microsoft.AspNetCore.Authentication.Twitter.TwitterOptions.StateCookie) | `Lax`  |
| <xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationHandler`1> | [RemoteAuthenticationOptions. 関連付けCookie](xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.CorrelationCookie)  | `None` |
| <xref:Microsoft.Extensions.DependencyInjection.OpenIdConnectExtensions.AddOpenIdConnect*> | [OpenIdConnectOptionsCookie](xref:Microsoft.AspNetCore.Authentication.OpenIdConnect.OpenIdConnectOptions.NonceCookie)| `None` |
| [CookieHttpContext。s. 追加](xref:Microsoft.AspNetCore.Http.IResponseCookies.Append*) | <xref:Microsoft.AspNetCore.Http.CookieOptions> | `Unspecified` |

::: moniker range=">= aspnetcore-3.1"

ASP.NET Core 3.1 以降では、次の SameSite サポートが提供されます。

* の動作を再定義して `SameSiteMode.None` を生成します。 `SameSite=None`
* 新しい値を追加し `SameSiteMode.Unspecified` て、SameSite 属性を省略します。
* すべてのの cookie api は、既定でに設定さ `Unspecified` れています。 S を使用するコンポーネントの中 cookie には、そのシナリオにより固有の値が設定されているものがあります。 例については、上の表を参照してください。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core 3.0 以降では、一貫性のないクライアントの既定値と競合しないように、SameSite の既定値が変更されました。 次の Api では、既定のをからに変更して、 `SameSiteMode.Lax ` `-1` これらのの SameSite 属性が出力されないようにしました cookie 。

* <xref:Microsoft.AspNetCore.Http.CookieOptions> HttpContext と共に使用されます。 [ Cookies. 追加](xref:Microsoft.AspNetCore.Http.IResponseCookies.Append*)
* <xref:Microsoft.AspNetCore.Http.CookieBuilder>  のファクトリとして使用されます。 `CookieOptions`
* [CookiePolicyOptions. MinimumSameSitePolicy](xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy)

::: moniker-end

## <a name="history-and-changes"></a>履歴と変更

SameSite サポートは、2.0 の ASP.NET Core で最初に [2016 ドラフト標準](https://tools.ietf.org/html/draft-west-first-party-cookies-07#section-4.1)を使用して実装されました。 2016標準はオプトインでした。 既定で複数のをに設定することによりオプトイン ASP.NET Core cookie `Lax` ます。 認証でいくつかの [問題](https://github.com/aspnet/Announcements/issues/318) が発生すると、ほとんどの SameSite の使用が [無効に](https://github.com/aspnet/Announcements/issues/348)なりました。

2016標準から2019標準に更新するために、2019年11月に[修正プログラム](https://devblogs.microsoft.com/dotnet/net-core-November-2019/)が発行されました。 [SameSite 仕様の2019ドラフト](https://github.com/aspnet/Announcements/issues/390):

* は、2016ドラフトとの下位互換性が **ありません** 。 詳細については、このドキュメントの「 [古いブラウザーのサポート](#sob) 」を参照してください。
* cookieは、既定でとして扱われることを指定し `SameSite=Lax` ます。
* cookieクロスサイト配信を有効にするために明示的にアサートするを指定し `SameSite=None` ます。これは、としてマークする必要があり `Secure` ます。 `None` は、オプトアウトする新しいエントリです。
* は、ASP.NET Core 2.1、2.2、および3.0 に対して発行された修正プログラムによってサポートされています。 ASP.NET Core 3.1 には、追加の SameSite サポートがあります。
* [2 月 2020](https://blog.chromium.org/2019/10/developers-get-ready-for-new.html)日に既定で[Chrome](https://chromestatus.com/feature/5088147346030592)によって有効になるようにスケジュールされています。 ブラウザーは2019でこの標準への移行を開始しました。

## <a name="apis-impacted-by-the-change-from-the-2016-samesite-draft-standard-to-the-2019-draft-standard"></a>2016 SameSite draft standard から2019ドラフト標準への変更によって影響を受ける Api

* [Http. SameSiteMode](xref:Microsoft.AspNetCore.Http.SameSiteMode)
* [CookieSameSite](xref:Microsoft.AspNetCore.Http.CookieOptions.SameSite)
* [CookieSameSite](xref:Microsoft.AspNetCore.Http.CookieBuilder.SameSite)
* [CookiePolicyOptions. MinimumSameSitePolicy](xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy)
* <xref:Microsoft.Net.Http.Headers.SameSiteMode?displayProperty=fullName>
* <xref:Microsoft.Net.Http.Headers.SetCookieHeaderValue.SameSite?displayProperty=fullName>

<a name="sob"></a>

## <a name="supporting-older-browsers"></a>古いブラウザーのサポート

2016 SameSite 標準では、不明な値を値として扱う必要があり `SameSite=Strict` ます。 2016 SameSite 標準をサポートする古いブラウザーからアクセスされるアプリは、値がの SameSite プロパティを取得すると破損する可能性があり `None` ます。 Web apps が古いブラウザーをサポートする予定の場合は、ブラウザーの検出を実装する必要があります。 User-Agents 値が非常に揮発性で頻繁に変更されるため、ASP.NET Core はブラウザーの検出を実装しません。 の拡張ポイントでは <xref:Microsoft.AspNetCore.CookiePolicy> User-Agent 特定のロジックにプラグインできます。

で `Startup.Configure` 、を <xref:Microsoft.AspNetCore.Builder.CookiePolicyAppBuilderExtensions.UseCookiePolicy*> 呼び出す前、 <xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*> または s を書き込む *任意* のメソッドを呼び出すコードを追加し cookie ます。

[!code-csharp[](samesite/sample/Startup.cs?name=snippet5&highlight=18-19)]

で `Startup.ConfigureServices` 、次のようなコードを追加します。

::: moniker range=">= aspnetcore-3.1"

[!code-csharp[](samesite/sample/Startup31.cs?name=snippet)]

::: moniker-end

::: moniker range="< aspnetcore-3.1"

[!code-csharp[](samesite/sample/Startup.cs?name=snippet)]

::: moniker-end

前の例では、は、ユーザー `MyUserAgentDetectionLib.DisallowsSameSiteNone` エージェントが SameSite をサポートしていないかどうかを検出する、ユーザーが指定したライブラリです `None` 。

[!code-csharp[](samesite/sample/Startup31.cs?name=snippet2)]

次のコードは、サンプルメソッドを示してい `DisallowsSameSiteNone` ます。

> [!WARNING]
> 次のコードは、デモンストレーションのみを対象としています。
> * 完全であるとは限りません。
> * 管理されていないか、サポートされていません。

[!code-csharp[](samesite/sample/Startup31.cs?name=snippetX)]

## <a name="test-apps-for-samesite-problems"></a>SameSite 問題のアプリをテストする

サードパーティのログインによってなどのリモートサイトと対話するアプリでは、次の操作を行う必要があります。

* 複数のブラウザーで相互作用をテストします。
* このドキュメントで説明され[ Cookie ているポリシーブラウザーの検出と軽減策](#sob)を適用します。

新しい SameSite 動作にオプトインできるクライアントバージョンを使用して、web アプリをテストします。 Chrome、Firefox、Chromium Edge には、テストに使用できる新しいオプトイン機能フラグがあります。 アプリが SameSite パッチを適用したら、古いクライアントバージョン (特に Safari) を使用してテストします。 詳細については、このドキュメントの「 [古いブラウザーのサポート](#sob) 」を参照してください。

### <a name="test-with-chrome"></a>Chrome を使用したテスト

Chrome 78 + には一時的な軽減策があるため、誤解を招く結果になります。 Chrome 78 + 一時的な軽減策を使用すると、が cookie 2 分未満になります。 適切なテストフラグが有効になっている Chrome 76 または77では、より正確な結果が得られます。 新しい SameSite の動作をテストするには、[ `chrome://flags/#same-site-by-default-cookies` **有効** に切り替えます。 新しい設定で失敗するように、Chrome の旧バージョン (75 以降) が報告され `None` ます。 このドキュメントの「 [古いブラウザーのサポート](#sob) 」を参照してください。

Google では、以前のバージョンの chrome は使用できません。 「 [Chromium のダウンロード](https://www.chromium.org/getting-involved/download-chromium) 」の手順に従って、Chrome の旧バージョンをテストします。 以前のバージョンの chrome を検索することによって提供されるリンクから Chrome **をダウンロードしないでください** 。

* [Chromium 76 Win64](https://commondatastorage.googleapis.com/chromium-browser-snapshots/index.html?prefix=Win_x64/664998/)
* [Chromium 74 Win64](https://commondatastorage.googleapis.com/chromium-browser-snapshots/index.html?prefix=Win_x64/638880/)

カナリアバージョン以降では、テストのために、厳密でない `80.0.3975.0` + POST の一時的な軽減策を無効にすることができます。この場合、新しいフラグを使用して、 `--enable-features=SameSiteDefaultChecksMethodRigorously` 軽減策が削除された機能の最終的な終了状態でサイトとサービスをテストできます。 詳細については、「Chromium Projects [SameSite Updates](https://www.chromium.org/updates/same-site) 」を参照してください。

### <a name="test-with-safari"></a>Safari を使用したテスト

Safari 12 では、以前のドラフトが厳密に実装されており、新しい `None` 値がに含まれていると失敗し cookie ます。 `None` このドキュメントでは、以前の [ブラウザーをサポート](#sob) するブラウザーの検出コードを使用して回避します。 MSAL、ADAL、使用している任意のライブラリを使用して、Safari 12、Safari 13、WebKit ベースの OS スタイルのログインをテストします。 この問題は、基盤の OS バージョンによって変わります。 OSX Mojave (10.14) と iOS 12 には、新しい SameSite の動作に互換性の問題があることがわかっています。 OS を OSX Catalina.properties (10.15) または iOS 13 にアップグレードすると、問題が解決されます。 Safari には、現在、新しい仕様動作をテストするオプトインフラグがありません。

### <a name="test-with-firefox"></a>Firefox でのテスト

新しい標準の Firefox サポートは、バージョン68以降で、機能フラグを使用してページをオンにすることでテストでき `about:config` `network.cookie.sameSite.laxByDefault` ます。 以前のバージョンの Firefox との互換性に関する問題は報告されていません。

### <a name="test-with-edge-browser"></a>Edge ブラウザーを使用したテスト

Edge では、古い SameSite 標準がサポートされています。 Edge バージョン44には、新しい標準との互換性に関する既知の問題はありません。

### <a name="test-with-edge-chromium"></a>Edge でテストする (Chromium)

ページに SameSite フラグが設定されてい `edge://flags/#same-site-by-default-cookies` ます。 Edge Chromium で互換性の問題は検出されませんでした。

### <a name="test-with-electron"></a>テストに使用する Electron

のバージョンに Electron は、古いバージョンの Chromium が含まれています。 たとえば、 Electron チームによって使用されるのバージョンは Chromium 66 であり、以前の動作を示しています。 使用している製品のバージョンで、独自の互換性テストを実行する必要があり Electron ます。 次のセクションの「 [古いブラウザーのサポート](#sob) 」を参照してください。

## <a name="additional-resources"></a>その他のリソース

* [Chromium ブログ: 開発者: 新しい SameSite の準備 = None;セキュリティで保護された Cookie 設定](https://blog.chromium.org/2019/10/developers-get-ready-for-new.html)
* [SameSite の cookie 説明](https://web.dev/samesite-cookies-explained/)
* [2019年11月の修正プログラム](https://devblogs.microsoft.com/dotnet/net-core-November-2019/)

 ::: moniker range=">= aspnetcore-2.1 < aspnetcore-3.0"

| サンプル               | ドキュメント |
| ----------------- | ------------ |
| [.NET Core MVC](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21MVC)  | <xref:security/samesite/mvc21> |
| [.NET Core Razor ページ](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21RazorPages)  | <xref:security/samesite/rp21> |

::: moniker-end

 ::: moniker range=">= aspnetcore-3.0"

| サンプル               | ドキュメント |
| ----------------- | ------------ |
| [.NET Core Razor ページ](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore31RazorPages)  | <xref:security/samesite/rp31> |

::: moniker-end
