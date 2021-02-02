---
title: ASP.NET Core の静的ファイル
author: rick-anderson
description: 静的ファイルを提供したり、それをセキュリティで保護したりする方法、および ASP.NET Core Web アプリで静的ファイルをホストするミドルウェアの動作を構成する方法について説明します。
ms.author: riande
ms.custom: mvc
ms.date: 6/23/2020
no-loc:
- appsettings.json
- ASP.NET Core Identity
- cookie
- Cookie
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: fundamentals/static-files
ms.openlocfilehash: d97caeffc6e8beebddb01a5bd126d61ba988de65
ms.sourcegitcommit: ebc5beccba5f3f7619de20baa58ad727d2a3d18c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/22/2021
ms.locfileid: "98689293"
---
# <a name="static-files-in-aspnet-core"></a>ASP.NET Core の静的ファイル

::: moniker range=">= aspnetcore-3.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT) および [Kirk Larkin](https://twitter.com/serpent5)

HTML、CSS、画像、JavaScript などの静的ファイルは、既定では ASP.NET Core アプリにより直接クライアントに提供される資産です。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/static-files/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="serve-static-files"></a>静的ファイルの提供

静的ファイルは、プロジェクトの [Web ルート](xref:fundamentals/index#web-root) ディレクトリ内に格納されています。 既定のディレクトリは `{content root}/wwwroot` ですが、<xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseWebRoot%2A> メソッドを使用して変更できます。 詳細については、「[コンテンツ ルート](xref:fundamentals/index#content-root)」および「[Web ルート](xref:fundamentals/index#web-root)」を参照してください。

<xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder%2A> メソッドでは、コンテンツのルートが現在のディレクトリに設定されます。

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/Program2.cs?name=snippet_Program)]

上記のコードは、Web アプリ テンプレートを使用して作成されました。

静的ファイルには、[Web ルート](xref:fundamentals/index#web-root)に対する相対パスを使用してアクセスできます。 たとえば、**Web アプリケーション** プロジェクト テンプレートでは、`wwwroot` フォルダー内に次のいくつかのフォルダーが含まれています。

* `wwwroot`
  * `css`
  * `js`
  * `lib`

*wwwroot/images* フォルダーを作成し、*wwwroot/images/MyImage.jpg* ファイルを追加する場合を考えてみます。 `images` フォルダー内のファイルにアクセスするための URI 形式は `https://<hostname>/images/<image_file_name>` です。 たとえば、`https://localhost:5001/images/MyImage.jpg`

### <a name="serve-files-in-web-root"></a>Web ルート内のファイルの提供

既定の Web アプリ テンプレートでは、`Startup.Configure` 内で <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A> メソッドが呼び出されます。これにより、静的ファイルを提供できるようになります。

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/Startup.cs?name=snippet_Configure&highlight=15)]

`UseStaticFiles` メソッドのパラメーターなしのオーバーロードによって、[Web ルート](xref:fundamentals/index#web-root)内のファイルが提供可能とマークされます。 次のマークアップは、*wwwroot/images/MyImage.jpg* を参照します。

```html
<img src="~/images/MyImage.jpg" class="img" alt="My image" />
```

上記のコードでは、チルダ文字 `~/` が [Web ルート](xref:fundamentals/index#web-root)を指します。

### <a name="serve-files-outside-of-web-root"></a>Web ルート外のファイルの提供

提供する静的ファイルが [Web ルート](xref:fundamentals/index#web-root)の外にあるディレクトリ階層について考えます。

* `wwwroot`
  * `css`
  * `images`
  * `js`
* `MyStaticFiles`
  * `images`
    * `red-rose.jpg`

静的ファイル ミドルウェアを次のように構成すると、要求で `red-rose.jpg` ファイルにアクセスできます。

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupRose.cs?name=snippet_Configure&highlight=15-22)]

前述のコードでは、*MyStaticFiles* ディレクトリ階層は、*StaticFiles* URI セグメントでパブリックに公開されています。 `https://<hostname>/StaticFiles/images/red-rose.jpg` の要求は、*red-rose.jpg* ファイルを提供します。

次のマークアップは、*MyStaticFiles/images/red-rose.jpg* を参照します。

```html
<img src="~/StaticFiles/images/red-rose.jpg" class="img" alt="A red rose" />
```

### <a name="set-http-response-headers"></a>HTTP 応答ヘッダーの設定

<xref:Microsoft.AspNetCore.Builder.StaticFileOptions> オブジェクトを使用すると、HTTP 応答ヘッダーを設定できます。 [Web ルート](xref:fundamentals/index#web-root)から提供される静的ファイルの構成に加えて、次のコードで `Cache-Control` ヘッダーを設定します。

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupAddHeader.cs?name=snippet_Configure&highlight=15-24)]

<!-- Q: The preceding code sets max-age to 604800 seconds (7 days), so what does the following mean? -->

以下のように、静的ファイルは 600 秒間パブリックにキャッシュ可能です。

![Cache-Control ヘッダーが追加された応答ヘッダー](static-files/_static/add-header.png)

## <a name="static-file-authorization"></a>静的ファイルの承認

ASP.NET Core テンプレートでは、<xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization%2A> を呼び出す前に <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A> が呼び出されます。 ほとんどのアプリがこのパターンに従います。 認可ミドルウェアの前に静的ファイル ミドルウェアが呼び出される場合:

  * 静的ファイルに対して認可チェックは実行されません。
  * 静的ファイル ミドルウェアによって提供される静的ファイル (`wwwroot` 下にあるものなど) には、パブリックにアクセスできます。
  
認可に基づいて静的ファイルを提供するには:

  * それらを `wwwroot` の外部に保存します。
  * `UseAuthorization` を呼び出した後に、パスを指定して `UseStaticFiles` を呼び出します。
  * [フォールバック認可ポリシー](xref:Microsoft.AspNetCore.Authorization.AuthorizationOptions.FallbackPolicy)を設定します。

  [!code-csharp[](static-files/samples/3.x/StaticFileAuth/Startup.cs?name=snippet2&highlight=24-29)]
  
  [!code-csharp[](static-files/samples/3.x/StaticFileAuth/Startup.cs?name=snippet1&highlight=20-25)]

  前のコードでは、フォールバック認可ポリシーによって "***すべての** _" ユーザーを認証することが要求されます。 独自の認可要件を指定する、コントローラー、Razor Pages などのエンドポイントでは、フォールバック認可ポリシーは使用されません。 たとえば、`[AllowAnonymous]` や `[Authorize(PolicyName="MyPolicy")]` を使用する Razor Pages、コントローラー、またはアクション メソッドでは、フォールバック認可ポリシーではなく適用された認可属性が使用されます。

  <xref:Microsoft.AspNetCore.Authorization.AuthorizationPolicyBuilder.RequireAuthenticatedUser%2A> により、現在のインスタンスに <xref:Microsoft.AspNetCore.Authorization.Infrastructure.DenyAnonymousAuthorizationRequirement> が追加されます。これにより、現在のユーザーが認証されます。

  `UseAuthentication` の前に既定の静的ファイル ミドルウェア (`app.UseStaticFiles();`) が呼び出されるため、`wwwroot` 下の静的資産にはパブリックにアクセスできます。 _MyStaticFiles* フォルダー内の静的資産には認証が必要です。 これは[サンプル コード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/static-files/samples)で示されています。

認可に基づいてファイルを提供する別の方法:

  * `wwwroot` や静的ファイル ミドルウェアがアクセスできる任意のディレクトリの外にファイルを配置します。
  * 承認が適用されるアクション メソッドを使用して提供し、<xref:Microsoft.AspNetCore.Mvc.FileResult> オブジェクトを返します。

  [!code-csharp[](static-files/samples/3.x/StaticFilesSample/Controllers/HomeController.cs?name=snippet_BannerImage)]

## <a name="directory-browsing"></a>ディレクトリ参照

ディレクトリ参照を使用すると、指定したディレクトリ内のディレクトリを一覧表示できます。

ディレクトリ参照は、セキュリティ上の理由から既定で無効になっています。 詳細については、「[注意事項](#considerations)」を参照してください。

ディレクトリ参照を有効にするには、次のメソッドを使用します。

* `Startup.ConfigureServices` の <xref:Microsoft.Extensions.DependencyInjection.DirectoryBrowserServiceExtensions.AddDirectoryBrowser%2A>
* `Startup.Configure` の <xref:Microsoft.AspNetCore.Builder.DirectoryBrowserExtensions.UseDirectoryBrowser%2A>

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupBrowse.cs?name=snippet_ClassMembers&highlight=4,21-35)]

上記のコードでは、URL `https://<hostname>/MyImages` が使用され、各ファイルおよびフォルダーへのリンクを含む *wwwroot/images* フォルダーのディレクトリが参照できるようになります。

![ディレクトリ参照](static-files/_static/dir-browse.png)

## <a name="serve-default-documents"></a>既定のドキュメントの提供

既定のページを設定すると、サイトのビジターの開始点になります。 完全修飾 URI を使用せずに `wwwroot` から既定のページを提供するには、<xref:Owin.DefaultFilesExtensions.UseDefaultFiles%2A> メソッドを呼び出します。

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupEmpty.cs?name=snippet_Configure&highlight=15)]

既定のファイルを提供するには、`UseStaticFiles` の前に `UseDefaultFiles` が呼び出される必要があります。 `UseDefaultFiles` は、ファイルを提供しない URL リライターです。

`UseDefaultFiles` を使用すると、`wwwroot` 内のフォルダーの要求では以下のファイルが検索されます。

* *default.htm*
* *default.html*
* *index.htm*
* *index.html*

一覧で見つかった最初のファイルは、要求で完全修飾 URI が使用されたかのように提供されます。 ブラウザー URL は、要求された URI を反映し続けます。

次のコードによって、既定のファイル名が *mydefault.html* に変更されます。

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupDefault.cs?name=snippet_DefaultFiles)]

次のコードでは、上記のコードを含めた `Startup.Configure` を示します。

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupDefault.cs?name=snippet_Configure&highlight=15-19)]

### <a name="usefileserver-for-default-documents"></a>既定のドキュメントの UseFileServer

<xref:Microsoft.AspNetCore.Builder.FileServerExtensions.UseFileServer*> は、`UseStaticFiles`、`UseDefaultFiles`、およびオプションとして `UseDirectoryBrowser` の機能を兼ね備えています。

`app.UseFileServer` を呼び出すと、静的ファイルと既定ファイルが提供できるようになります。 ディレクトリ参照は有効にしません。 次のコードでは、`UseFileServer` を含めた `Startup.Configure` を示します。

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupEmpty2.cs?name=snippet_Configure&highlight=15)]

次のコードによって、静的ファイルの提供、既定ファイル、ディレクトリ参照が有効になります。

```csharp
app.UseFileServer(enableDirectoryBrowsing: true);
```

次のコードでは、上記のコードを含めた `Startup.Configure` を示します。

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupEmpty3.cs?name=snippet_Configure&highlight=15)]

次のディレクトリ階層があるとします。

* `wwwroot`
  * `css`
  * `images`
  * `js`
* `MyStaticFiles`
  * `images`
    * `MyImage.jpg`
  * `default.html`

次のコードによって、`MyStaticFiles` の静的ファイルの提供、既定ファイル、ディレクトリ参照が有効になります。

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupFileServer.cs?name=snippet_ClassMembers&highlight=4,21-31)]

`EnableDirectoryBrowsing` プロパティの値が `true` であるときは、<xref:Microsoft.Extensions.DependencyInjection.DirectoryBrowserServiceExtensions.AddDirectoryBrowser%2A> を呼び出す必要があります。

このファイル階層と前述のコードを使用すると、URL は次のように解決されます。

| URI            |      応答  |
| ------- | ------|
| `https://<hostname>/StaticFiles/images/MyImage.jpg` | *MyStaticFiles/images/MyImage.jpg* |
| `https://<hostname>/StaticFiles` | *MyStaticFiles/default.html* |

*MyStaticFiles* ディレクトリに既定の名前のファイルが存在しない場合、`https://<hostname>/StaticFiles` によってクリック可能なリンクを含むディレクトリの一覧が返されます。

![静的ファイルの一覧](static-files/_static/db2.png)

<xref:Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles*> および <xref:Microsoft.AspNetCore.Builder.DirectoryBrowserExtensions.UseDirectoryBrowser*> では、末尾の `/` がないターゲット URI から末尾の `/` があるターゲット URI へのクライアント側リダイレクトが実行されます。 たとえば、`https://<hostname>/StaticFiles` は `https://<hostname>/StaticFiles/` になります。 *StaticFiles* ディレクトリ内の相対 URL は、末尾のスラッシュ (`/`) がないと無効です。

## <a name="fileextensioncontenttypeprovider"></a>FileExtensionContentTypeProvider

<xref:Microsoft.AspNetCore.StaticFiles.FileExtensionContentTypeProvider> クラスには、MIME コンテンツ タイプへのファイル拡張子のマッピングを行う `Mappings` プロパティが含まれます。 次の例では、いくつかのファイル拡張子が、既知の MIME タイプにマッピングされています。 *.rtf* 拡張子は置換され、 *.mp4* は削除されています。

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupFileExtensionContentTypeProvider.cs?name=snippet_Provider)]

次のコードでは、上記のコードを含めた `Startup.Configure` を示します。

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupFileExtensionContentTypeProvider.cs?name=snippet_Configure&highlight=15-43)]

「[MIME content types](https://www.iana.org/assignments/media-types/media-types.xhtml)」 (MIME コンテンツ タイプ) を参照してください。

## <a name="non-standard-content-types"></a>非標準のコンテンツ タイプ

静的ファイル ミドルウェアでは、約 400 の既知のファイル コンテンツ タイプが認識されています。 ユーザーがファイルの種類が不明なファイルを要求した場合、静的ファイル ミドルウェアでその要求がパイプラインの次のミドルウェアに渡されます。 ミドルウェアで要求が処理されない場合、*404 見つかりません* という応答が返されます。 ディレクトリ参照が有効になっている場合、ディレクトリ一覧にファイルへのリンクが表示されます。

次のコードによって、不明なタイプを提供できるようにし、不明なファイルをイメージとしてレンダリングします。

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupServeUnknownFileTypes.cs?name=snippet_UseStaticFiles)]

次のコードでは、上記のコードを含めた `Startup.Configure` を示します。

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupServeUnknownFileTypes.cs?name=snippet_Configure&highlight=15-19)]

上記のコードでは、不明なコンテンツ タイプ ファイルに対する要求は、イメージとして返されます。

> [!WARNING]
> <xref:Microsoft.AspNetCore.Builder.StaticFileOptions.ServeUnknownFileTypes> を有効にすると、セキュリティ上リスクとなります。 これは既定では無効で、使用は推奨されていません。 非標準の拡張子のファイルを提供する場合、より安全な代替となるのは、[FileExtensionContentTypeProvider](#fileextensioncontenttypeprovider) です。

## <a name="serve-files-from-multiple-locations"></a>複数の場所からファイルを提供する

`UseStaticFiles` と `UseFileServer` の既定では、`wwwroot` をポイントするファイル プロバイダーが作成されます。 `UseStaticFiles` および `UseFileServer` の追加インスタンスを作成して他のファイル プロバイダーを使用すると、他の場所からファイルを提供することができます。 詳細については、次を参照してください。[この GitHub の問題](https://github.com/dotnet/AspNetCore.Docs/issues/15578)します。

<a name="sc"></a>

### <a name="security-considerations-for-static-files"></a>静的ファイルのセキュリティに関する注意点

> [!WARNING]
> `UseDirectoryBrowser` と `UseStaticFiles` では、機密データが漏洩することがあります。 本番では、ディレクトリ参照を無効にすることが、強く推奨されます。 `UseStaticFiles` や `UseDirectoryBrowser` でどのディレクトリが有効になっているか、慎重にご確認ください。 ディレクトリ全体とそのサブディレクトリが、パブリックにアクセス可能になります。 ファイルは、パブリックに提供するのに適した、`<content_root>/wwwroot` などの専用ディレクトリに格納します。 これらのファイルは、MVC ビュー、Razor Pages、構成ファイルなどとは別にします。

* `UseDirectoryBrowser` と `UseStaticFiles` で公開されるコンテンツの URL では、大文字と小文字が区別され、基になるファイル システムの文字制限の影響を受けます。 たとえば、Windows では大文字と小文字が区別されませんが、macOS と Linux では区別されます。

* IIS でホストされている ASP.NET Core アプリは、[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)を使用して、静的ファイルの要求を含む、すべての要求をアプリに転送します。 IIS の静的ファイル ハンドラーは使用されず、要求を処理することはできません。

* IIS マネージャーで次の手順を実行し、サーバーまたは Web サイト レベルで IIS の静的ファイル ハンドラーを削除します。
    1. **[モジュール]** 機能に移動します。
    1. 一覧の **[StaticFileModule]** を選択します。
    1. **[アクション]** サイドバーで、 **[削除]** をクリックします。

> [!WARNING]
> IIS の静的ファイル ハンドラーが有効になっており、**かつ**、ASP.NET Core モジュールが正しく構成されていない場合、静的ファイルにサービスが提供されます。 これは、たとえば、*web.config* ファイルが配置されていない場合などで発生します。

* アプリ プロジェクトの [Web ルート](xref:fundamentals/index#web-root)の外に、コード ファイル ( *.cs* と *.cshtml* を含む) を配置します。 これにより、アプリのクライアント側コンテンツとサーバー ベースのコードの間で、論理的な分離が作成されます。 これによって、サーバー側のコードが漏洩するのを防ぎます。

## <a name="additional-resources"></a>その他の技術情報

* [ミドルウェア](xref:fundamentals/middleware/index)
* [ASP.NET Core の概要](xref:index)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)、[Scott Addie](https://twitter.com/Scott_Addie)

HTML、CSS、画像、JavaScript などの静的ファイルは、ASP.NET Core アプリにより直接クライアントに提供される資産です。 これらのファイルを提供するには、いくつかの構成が必要です。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/static-files/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="serve-static-files"></a>静的ファイルの提供

静的ファイルは、プロジェクトの [Web ルート](xref:fundamentals/index#web-root) ディレクトリ内に格納されています。 既定のディレクトリは *{content root}/wwwroot* ですが、<xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseWebRoot%2A> メソッドを使用して変更できます。 詳細については、「[コンテンツ ルート](xref:fundamentals/index#content-root)」および「[Web ルート](xref:fundamentals/index#web-root)」を参照してください。

アプリの Web ホストでは、コンテンツのルート ディレクトリが把握されている必要があります。

`WebHost.CreateDefaultBuilder` メソッドでは、コンテンツのルートが現在のディレクトリに設定されます。

[!code-csharp[](../common/samples/WebApplication1DotNetCore2.0App/Program.cs?name=snippet_Main&highlight=9)]

静的ファイルには、[Web ルート](xref:fundamentals/index#web-root)に対する相対パスを使用してアクセスできます。 たとえば、**Web アプリケーション** プロジェクト テンプレートでは、`wwwroot` フォルダー内に次のいくつかのフォルダーが含まれています。

* `wwwroot`
  * `css`
  * `images`
  * `js`

*images* サブフォルダーのファイルにアクセスするための URI は、*http://\<server_address>/images/\<image_file_name>* です。 たとえば、 *http://localhost:9189/images/banner3.svg* です。

.NET Framework を対象としている場合、プロジェクトに [Microsoft.AspNetCore.StaticFiles](https://www.nuget.org/packages/Microsoft.AspNetCore.StaticFiles/) パッケージを追加します。 .NET Core をターゲットとする場合、[Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)にこのパッケージが含まれます。

静的ファイル サービスの提供を可能にする、[ミドルウェア](xref:fundamentals/middleware/index)を構成します。

### <a name="serve-files-inside-of-web-root"></a>Web ルート内のファイルの提供

`Startup.Configure` 内で <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A> メソッドを呼び出します。

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupStaticFiles.cs?name=snippet_ConfigureMethod&highlight=3)]

`UseStaticFiles` メソッドのパラメーターなしのオーバーロードによって、[Web ルート](xref:fundamentals/index#web-root)内のファイルが提供可能とマークされます。 次のマークアップは、*wwwroot/images/banner1.svg* を参照します。

[!code-cshtml[](static-files/samples/1.x/StaticFilesSample/Views/Home/Index.cshtml?name=snippet_static_file_wwwroot)]

上記のコードでは、チルダ文字 `~/` が [Web ルート](xref:fundamentals/index#web-root)を指します。

### <a name="serve-files-outside-of-web-root"></a>Web ルート外のファイルの提供

提供する静的ファイルが [Web ルート](xref:fundamentals/index#web-root)の外にあるディレクトリ階層について考えます。

* `wwwroot`
  * `css`
  * `images`
  * `js`
* `MyStaticFiles`
  * `images`
    * `banner1.svg`

静的ファイル ミドルウェアを次のように構成すると、要求で *banner1.svg* ファイルにアクセスできます。

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupTwoStaticFiles.cs?name=snippet_ConfigureMethod&highlight=5-10)]

前述のコードでは、*MyStaticFiles* ディレクトリ階層は、*StaticFiles* URI セグメントでパブリックに公開されています。 *http://\<server_address>/StaticFiles/images/banner1.svg* に対する要求によって、*banner1.svg* ファイルが提供されます。

次のマークアップは、*MyStaticFiles/images/banner1.svg* を参照します。

[!code-cshtml[](static-files/samples/1.x/StaticFilesSample/Views/Home/Index.cshtml?name=snippet_static_file_outside)]

### <a name="set-http-response-headers"></a>HTTP 応答ヘッダーの設定

<xref:Microsoft.AspNetCore.Builder.StaticFileOptions> オブジェクトを使用すると、HTTP 応答ヘッダーを設定できます。 [Web ルート](xref:fundamentals/index#web-root)から提供される静的ファイルの構成に加えて、次のコードで `Cache-Control` ヘッダーを設定します。

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupAddHeader.cs?name=snippet_ConfigureMethod)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

<xref:Microsoft.AspNetCore.Http.HeaderDictionaryExtensions.Append%2A?displayProperty=nameWithType> メソッドは、[Microsoft.AspNetCore.Http](https://www.nuget.org/packages/Microsoft.AspNetCore.Http/) パッケージにあります。

これらのファイルは、開発環境で 10 分間 (600 秒) パブリックにキャッシュ可能でした。

![Cache-Control ヘッダーが追加された応答ヘッダー](static-files/_static/add-header.png)

## <a name="static-file-authorization"></a>静的ファイルの承認

静的ファイル ミドルウェアでは、承認の確認は行いません。 *wwwroot* の下にあるものも含め、このミドルウェアによって提供されるすべてのファイルは、一般に公開されます。 承認に基づいてファイルを提供するには:

* *wwwroot* や静的ファイル ミドルウェアがアクセスできる任意のディレクトリの外にファイルを配置します。
* 承認が適用されるアクション メソッドを使用して提供するようにします。 <xref:Microsoft.AspNetCore.Mvc.FileResult> オブジェクトを返します。

  [!code-csharp[](static-files/samples/1.x/StaticFilesSample/Controllers/HomeController.cs?name=snippet_BannerImageAction)]

## <a name="enable-directory-browsing"></a>ディレクトリ参照の有効化

ディレクトリ参照では、Web アプリのユーザーが、ディレクトリ一覧および、指定したディレクトリ内のファイルを参照できるようになります。 ディレクトリ参照は、セキュリティ上の理由から既定で無効になっています (「[注意点](#considerations)」を参照)。 `Startup.Configure` で <xref:Microsoft.AspNetCore.Builder.DirectoryBrowserExtensions.UseDirectoryBrowser%2A> メソッドを呼び出すことで、ディレクトリの参照を有効にします。

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupBrowse.cs?name=snippet_ConfigureMethod&highlight=12-17)]

`Startup.ConfigureServices` から <xref:Microsoft.Extensions.DependencyInjection.DirectoryBrowserServiceExtensions.AddDirectoryBrowser%2A> メソッドを呼び出すと、必要なサービスを追加することができます。

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupBrowse.cs?name=snippet_ConfigureServicesMethod&highlight=3)]

上記のコードでは、URL *http://\<server_address>/MyImages* が使用され、各ファイルおよびフォルダーへのリンクを含む *wwwroot/images* フォルダーのディレクトリが参照できるようになります。

![ディレクトリ参照](static-files/_static/dir-browse.png)

参照を有効にした場合のセキュリティ上のリスクについては、「[注意点](#considerations)」を参照してください。

次の例の 2 つの `UseStaticFiles` 呼び出しを確認してください。 最初の呼び出しにより、*wwwroot* フォルダー内の静的ファイルが提供されます。 2 番目の呼び出しにより、URL *http://\<server_address>/MyImages* が使用され、*wwwroot/images* フォルダーのディレクトリ参照が有効になります。

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupBrowse.cs?name=snippet_ConfigureMethod&highlight=3,5)]

## <a name="serve-a-default-document"></a>既定のドキュメントの提供

設定された既定のホーム ページは、サイトを訪問するユーザーの論理的な開始点となります。 ユーザーが URI を完全修飾しない場合に既定のページを提供するには、`Startup.Configure` から <xref:Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles%2A> メソッドを呼び出します。

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupEmpty.cs?name=snippet_ConfigureMethod&highlight=3)]

> [!IMPORTANT]
> 既定のファイルを提供するには、`UseStaticFiles` の前に `UseDefaultFiles` が呼び出される必要があります。 `UseDefaultFiles` は、ファイルを実際には提供しない URL リライターです。 ファイルを提供するには、`UseStaticFiles` を使用して静的ファイル ミドルウェアを有効にします。

`UseDefaultFiles` を使用し、以下のフォルダーを検索します。

* *default.htm*
* *default.html*
* *index.htm*
* *index.html*

一覧で見つかった最初のファイルは、要求で完全修飾 URI が使用されたかのように提供されます。 ブラウザー URL は、要求された URI を反映し続けます。

次のコードによって、既定のファイル名が *mydefault.html* に変更されます。

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupDefault.cs?name=snippet_ConfigureMethod)]

## <a name="usefileserver"></a>UseFileServer

<xref:Microsoft.AspNetCore.Builder.FileServerExtensions.UseFileServer*> は、`UseStaticFiles`、`UseDefaultFiles`、およびオプションとして `UseDirectoryBrowser` の機能を兼ね備えています。

次のコードによって、静的ファイルと既定ファイルが提供できるようになります。 ディレクトリ参照は有効にしません。

```csharp
app.UseFileServer();
```

次のコードは、ディレクトリ参照が有効になった、パラメーターなしのオーバーロード時に構築されます。

```csharp
app.UseFileServer(enableDirectoryBrowsing: true);
```

次のディレクトリ階層があるとします。

* **wwwroot**
  * **css**
  * **images**
  * **js**
* **MyStaticFiles**
  * **images**
    * *banner1.svg*
  * *default.html*

次のコードにより、`MyStaticFiles` への静的ファイル、既定ファイル、ディレクトリ参照が有効になります。

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupUseFileServer.cs?name=snippet_ConfigureMethod&highlight=5-11)]

`EnableDirectoryBrowsing` プロパティの値が `true` であるとき、`AddDirectoryBrowser` を呼び出す必要があります。

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupUseFileServer.cs?name=snippet_ConfigureServicesMethod)]

このファイル階層と前述のコードを使用すると、URL は次のように解決されます。

| URI            |                             応答  |
| ------- | ------|
| *http://\<server_address>/StaticFiles/images/banner1.svg*    |      MyStaticFiles/images/banner1.svg |
| *http://\<server_address>/StaticFiles*             |     MyStaticFiles/default.html |

*MyStaticFiles* ディレクトリに既定の名前のファイルが存在しない場合、*http://\<server_address>/StaticFiles* によってクリック可能なリンクを含むディレクトリの一覧が返されます。

![静的ファイルの一覧](static-files/_static/db2.png)

> [!NOTE]
> <xref:Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles*> および <xref:Microsoft.AspNetCore.Builder.DirectoryBrowserExtensions.UseDirectoryBrowser*> では、`http://{SERVER ADDRESS}/StaticFiles` (末尾のスラッシュなし) から `http://{SERVER ADDRESS}/StaticFiles/` (末尾のスラッシュあり) へのクライアント側リダイレクトが実行されます。 *StaticFiles* ディレクトリ内の相対 URL は、末尾のスラッシュがないと無効です。

## <a name="fileextensioncontenttypeprovider"></a>FileExtensionContentTypeProvider

<xref:Microsoft.AspNetCore.StaticFiles.FileExtensionContentTypeProvider> クラスには、MIME コンテンツ タイプへのファイル拡張子のマッピングを行う `Mappings` プロパティが含まれます。 次の例では、いくつかのファイル拡張子は、既知の MIME タイプに登録されています。 *.rtf* 拡張子は置換され、 *.mp4* は削除されています。

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupFileExtensionContentTypeProvider.cs?name=snippet_ConfigureMethod&highlight=3-12,19)]

「[MIME content types](https://www.iana.org/assignments/media-types/media-types.xhtml)」 (MIME コンテンツ タイプ) を参照してください。

カスタム <xref:Microsoft.AspNetCore.StaticFiles.FileExtensionContentTypeProvider> を使用する方法、または Blazor サーバー アプリで他の <xref:Microsoft.AspNetCore.Builder.StaticFileOptions> を構成する方法については、<xref:blazor/fundamentals/additional-scenarios#static-files> を参照してください。

## <a name="non-standard-content-types"></a>非標準のコンテンツ タイプ

静的ファイル ミドルウェアでは、約 400 個の既知のファイル コンテンツ タイプが認識されます。 ユーザーがファイルの種類が不明なファイルを要求した場合、静的ファイル ミドルウェアでその要求がパイプラインの次のミドルウェアに渡されます。 ミドルウェアで要求が処理されない場合、*404 見つかりません* という応答が返されます。 ディレクトリ参照が有効になっている場合、ディレクトリ一覧にファイルへのリンクが表示されます。

次のコードによって、不明なタイプを提供できるようにし、不明なファイルをイメージとしてレンダリングします。

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupServeUnknownFileTypes.cs?name=snippet_ConfigureMethod)]

上記のコードでは、不明なコンテンツ タイプ ファイルに対する要求は、イメージとして返されます。

> [!WARNING]
> <xref:Microsoft.AspNetCore.Builder.StaticFileOptions.ServeUnknownFileTypes> を有効にすると、セキュリティ上リスクとなります。 これは既定では無効で、使用は推奨されていません。 非標準の拡張子のファイルを提供する場合、より安全な代替となるのは、[FileExtensionContentTypeProvider](#fileextensioncontenttypeprovider) です。

## <a name="serve-files-from-multiple-locations"></a>複数の場所からファイルを提供する

`UseStaticFiles` と `UseFileServer` の既定では、*wwwroot* をポイントするファイル プロバイダーが作成されます。 他のファイル プロバイダーを使用する `UseStaticFiles` および `UseFileServer` の追加インスタンスを作成して、他の場所からファイルを提供することができます。 詳細については、次を参照してください。[この GitHub の問題](https://github.com/dotnet/AspNetCore.Docs/issues/15578)します。

### <a name="considerations"></a>注意事項

> [!WARNING]
> `UseDirectoryBrowser` と `UseStaticFiles` では、機密データが漏洩することがあります。 本番では、ディレクトリ参照を無効にすることが、強く推奨されます。 `UseStaticFiles` や `UseDirectoryBrowser` でどのディレクトリが有効になっているか、慎重にご確認ください。 ディレクトリ全体とそのサブディレクトリが、パブリックにアクセス可能になります。 ファイルは、パブリックに提供するのに適した、 *\<content_root>/wwwroot* などの専用ディレクトリに格納します。 これらのファイルは、MVC ビュー、Razor Pages (2.x のみ)、構成ファイルなどとは別にします。

* `UseDirectoryBrowser` と `UseStaticFiles` で公開されるコンテンツの URL では、大文字と小文字が区別され、基になるファイル システムの文字制限の影響を受けます。 たとえば、Windows では大文字小文字は区別されますが、macOS と Linux ではされません。

* IIS でホストされている ASP.NET Core アプリは、[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)を使用して、静的ファイルの要求を含む、すべての要求をアプリに転送します。 IIS 静的ファイル ハンドラーは使用されません。 モジュールによって処理される前に、これで要求が処理されることはありません。

* IIS マネージャーで次の手順を実行し、サーバーまたは Web サイト レベルで IIS の静的ファイル ハンドラーを削除します。
    1. **[モジュール]** 機能に移動します。
    1. 一覧の **[StaticFileModule]** を選択します。
    1. **[アクション]** サイドバーで、 **[削除]** をクリックします。

> [!WARNING]
> IIS の静的ファイル ハンドラーが有効になっており、**かつ**、ASP.NET Core モジュールが正しく構成されていない場合、静的ファイルにサービスが提供されます。 これは、たとえば、*web.config* ファイルが配置されていない場合などで発生します。

* アプリ プロジェクトの [Web ルート](xref:fundamentals/index#web-root)の外に、コード ファイル ( *.cs* と *.cshtml* を含む) を配置します。 これにより、アプリのクライアント側コンテンツとサーバー ベースのコードの間で、論理的な分離が作成されます。 これによって、サーバー側のコードが漏洩するのを防ぎます。

## <a name="additional-resources"></a>その他の技術情報

* [ミドルウェア](xref:fundamentals/middleware/index)
* [ASP.NET Core の概要](xref:index)

::: moniker-end
