---
title: ASP.NET から ASP.NET Core への移行
author: isaac2004
description: 既存の ASP.NET MVC または Web API アプリを ASP.NET Core.web に移行するときのガイダンスをご覧ください
ms.author: scaddie
ms.date: 10/18/2019
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
uid: migration/proper-to-2x/index
ms.openlocfilehash: 7961890becc8f4513e0750f28341c9d4cf94e7ad
ms.sourcegitcommit: 07e7ee573fe4e12be93249a385db745d714ff6ae
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/12/2021
ms.locfileid: "103413336"
---
# <a name="migrate-from-aspnet-to-aspnet-core"></a>ASP.NET から ASP.NET Core への移行

著者: [Isaac Levin](https://isaaclevin.com)

この記事は、ASP.NET アプリを ASP.NET Core に移行するための参考ガイドです。 包括的な移植のガイドについては、「[既存の ASP.NET アプリを .NET Core に移植する](https://aka.ms/aspnet-porting-ebook)」の電子書籍版を参照してください。

## <a name="prerequisites"></a>必須コンポーネント

[.NET Core SDK 2.2 以降](https://dotnet.microsoft.com/download)

## <a name="target-frameworks"></a>ターゲット フレームワーク

ASP.NET Core プロジェクトを使うと、開発者は、.NET Core と .NET Framework のどちらか一方または両方を対象にして柔軟に開発できます。 最も適切なターゲット フレームワークの決定については、「[サーバー アプリ用 .NET Core と .NET Framework の選択](/dotnet/standard/choosing-core-framework-server)」をご覧ください。

.NET Framework を対象にする場合は、プロジェクトで個々の NuGet パッケージを参照する必要があります。

.NET Core を対象にすると、ASP.NET Core [メタパッケージ](xref:fundamentals/metapackage-app)のおかげで、さまざまな明示的パッケージ参照をしなくて済みます。 `Microsoft.AspNetCore.App` メタパッケージをプロジェクトにインストールします。

```xml
<ItemGroup>
   <PackageReference Include="Microsoft.AspNetCore.App" />
</ItemGroup>
```

メタパッケージを使うと、メタパッケージ内で参照されているパッケージはアプリでは展開されません。 .NET Core ランタイム ストアにはこれらのアセットが含まれており、パフォーマンス向上のためにプリコンパイルされています。 詳細については、[ASP.NET Core 用の Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)に関する記事を参照してください。

## <a name="project-structure-differences"></a>プロジェクトの構造の違い

*.csproj* ファイルの形式は、ASP.NET Core では簡素化されています。 いくつかの重要な変更は次のとおりです。

- ファイルがプロジェクトの一部と見なされるためにファイルを明示的に含める必要はありません。 これにより、大規模なチームで作業する場合に XML のマージが競合するリスクが軽減されます。
- 他のプロジェクトを GUID で参照することはなくなり、ファイルの読みやすさが向上します。
- Visual Studio でアンロードせずにファイルを編集することができます。

    ![Visual Studio 2017 の CSPROJ の編集コンテキスト メニュー オプション](_static/EditProjectVs2017.png)

## <a name="globalasax-file-replacement"></a>Global.asax ファイルの置換

ASP.NET Core では、アプリをブートストラップする新しいメカニズムが導入されました。 ASP.NET アプリケーションのエントリ ポイントは、*Global.asax* ファイルです。 ルート構成、フィルター、領域の登録などのタスクは、*Global.asax* ファイルで処理されます。

[!code-csharp[](samples/globalasax-sample.cs)]

このアプローチでは、アプリケーションとその展開先のサーバーが、実装を妨げるような方法で結合されます。 結合を切り離すため、複数のフレームワークを一緒に使うさらにクリーンな方法を提供する [OWIN](https://owin.org/) が導入されました。 OWIN は、必要なモジュールのみを追加するためのパイプラインを提供します。 ホスティング環境は、[Startup](xref:fundamentals/startup) 関数を取得して、サービスとアプリの要求パイプラインを構成します。 `Startup` は、ミドルウェアのセットをアプリケーションに登録します。 アプリケーションは、要求ごとに、既存のハンドラーのセットに対するリンク リストのヘッド ポインターを指定して、各ミドルウェア コンポーネントを呼び出します。 各ミドルウェア コンポーネントは、要求処理パイプラインに 1 つ以上のハンドラーを追加できます。 これは、新しいリストのヘッドであるハンドラーへの参照を返すことによって行われます。 各ハンドラーは、リスト内の次のハンドラーを記憶して呼び出します。 ASP.NET Core では、アプリケーションへのエントリ ポイントは `Startup` であり、*Global.asax* に依存する必要はなくなりました。 .NET Framework で OWIN を使うときは、パイプラインとして次のようなものを使います。

[!code-csharp[](samples/webapi-owin.cs)]

これにより既定のルートが構成され、既定では Json 経由の XmlSerialization です。 必要に応じて、このパイプラインに他のミドルウェアを追加します (サービスの読み込み、構成設定、静的ファイルなど)。

ASP.NET Core は同様のアプローチを使いますが、エントリを処理するために OWIN には依存しません。 代わりに、(コンソール アプリケーションと同じように) *Program.cs* の `Main` メソッドを通して行われ、そこから `Startup` が読み込まれます。

[!code-csharp[](samples/program.cs)]

`Startup` は、`Configure` メソッドを含む必要があります。 `Configure` では、必要なミドルウェアをパイプラインに追加します。 (既定の Web サイト テンプレートからの) 次の例では、拡張メソッドにより、以下をサポートするパイプラインが構成されます。

- エラー ページ
- HTTP Strict Transport Security
- HTTP への HTTP リダイレクト
- ASP.NET Core MVC

[!code-csharp[](samples/startup.cs)]

ホストとアプリケーションは切り離されており、将来別のプラットフォームに柔軟に移動できます。

> [!NOTE]
> ASP.NET Core のスタートアップとミドルウェアについて詳しくは、「[ASP.NET Core でのアプリケーションのスタートアップ](xref:fundamentals/startup)」をご覧ください

## <a name="store-configurations"></a>構成を保存する

ASP.NET では保存の設定がサポートされています。 これらの設定は、たとえば、アプリケーションが展開された環境のサポートに使われます。 一般的な方法は、すべてのカスタム キー/値ペアを、*Web.config* ファイルの `<appSettings>` セクションに保存するというものでした。

[!code-xml[](samples/webconfig-sample.xml)]

アプリケーションでは、`System.Configuration` 名前空間内の `ConfigurationManager.AppSettings` コレクションを使ってこれらの設定を読み取ります。

[!code-csharp[](samples/read-webconfig.cs)]

ASP.NET Core では、アプリケーションの構成データを任意のファイルに保存し、ミドルウェアのブートストラップの一部として読み込むことができます。 プロジェクト テンプレートで使用される既定のファイルは、 *appsettings.json* です。

[!code-json[](samples/appsettings-sample.json)]

アプリケーション内部の `IConfiguration` のインスタンスにこのファイルを読み込むには、*Startup.cs* が使われます。

[!code-csharp[](samples/startup-builder.cs)]

アプリでは、`Configuration` から読み取って設定を取得します。

[!code-csharp[](samples/read-appsettings.cs)]

このアプローチにはプロセスをより堅牢にする拡張機能があります。たとえば、[依存性の注入](xref:fundamentals/dependency-injection) (DI) を使ってサービスとこれらの値を読み込むことができます。 DI アプローチは、厳密に型指定された構成オブジェクトのセットを提供します。

```csharp
// Assume AppConfiguration is a class representing a strongly-typed version of AppConfiguration section
services.Configure<AppConfiguration>(Configuration.GetSection("AppConfiguration"));
```

> [!NOTE]
> ASP.NET Core の構成について詳しくは、「[ASP.NET Core の構成](xref:fundamentals/configuration/index)」をご覧ください。

## <a name="native-dependency-injection"></a>ネイティブな依存性の注入

大規模で拡張性の高いアプリケーションを構築するときの重要な目標は、コンポーネントとサービスの疎な結合です。 [依存性の注入](xref:fundamentals/dependency-injection)はこれを実現するための一般的な手法であり、ASP.NET Core のネイティブなコンポーネントです。

ASP.NET アプリでは、開発者はサードパーティのライブラリに依存して依存性の注入を実装します。 [Unity](https://github.com/unitycontainer/unity) はそのようなライブラリの 1 つであり、Microsoft Patterns & Practices によって提供されます。

Unity で依存性の注入を設定する例は、`UnityContainer` をラップする `IDependencyResolver` の実装です。

[!code-csharp[](samples/sample8.cs)]

`UnityContainer` のインスタンスを作成し、サービスを登録して、`HttpConfiguration` の依存関係リゾルバーをコンテナー用の `UnityResolver` の新しいインスタンスに設定します。

[!code-csharp[](samples/sample9.cs)]

必要な場所に `IProductRepository` を挿入します。

[!code-csharp[](samples/sample5.cs)]

依存性の注入は ASP.NET Core の一部であるため、*Startup.cs* の `ConfigureServices` メソッドに独自のサービスを追加できます。

[!code-csharp[](samples/configure-services.cs)]

Unity でそうであったように、リポジトリは任意の場所に挿入できます。

> [!NOTE]
> 依存関係の挿入について詳しくは、「[依存関係の挿入](xref:fundamentals/dependency-injection)」をご覧ください。

## <a name="serve-static-files"></a>静的ファイルの提供

Web 開発の重要な部分は、静的なクライアント側アセットを提供する機能です。 静的なファイルの最も一般的な例は、HTML、CSS、Javascript、およびイメージです。 これらのファイルは、アプリ (または CDN) の公開された場所に保存され、要求によって読み込めるように参照される必要があります。 このプロセスは、ASP.NET Core で変更されました。

ASP.NET では、静的ファイルはさまざまなディレクトリに保存され、ビューで参照されます。

ASP.NET Core では、構成が変更されていない限り、静的ファイルは "Web ルート" ( *&lt;コンテンツ ルート&gt;/wwwroot*) に保存されます。 ファイルは、`Startup.Configure` から `UseStaticFiles` 拡張メソッドを呼び出すことによって、要求パイプラインに読み込まれます。

[!code-csharp[](../../fundamentals/static-files/samples/1.x/StaticFilesSample/StartupStaticFiles.cs?highlight=3&name=snippet_ConfigureMethod)]

> [!NOTE]
> .NET Framework を対象にする場合は、NuGet パッケージ `Microsoft.AspNetCore.StaticFiles` をインストールします。

たとえば、*wwwroot/images* フォルダー内のイメージ アセットには、ブラウザーから `http://<app>/images/<imageFileName>` などの場所でアクセスできます。

> [!NOTE]
> ASP.NET Core での静的ファイルの提供について詳しくは、[静的ファイル](xref:fundamentals/static-files)に関するページをご覧ください。

## <a name="multi-value-cookies"></a>複数値の cookie

[複数値の cookie](xref:System.Web.HttpCookie.Values) は ASP.NET Core ではサポートされていません。 値ごとに cookie を 1 つ作成します。

## <a name="authentication-cookies-are-not-compressed-in-aspnet-core"></a>認証 cookie は ASP.NET Core では圧縮されません

[!INCLUDE[](~/includes/cookies-not-compressed.md)]

## <a name="partial-app-migration"></a>部分的なアプリの移行

部分的にアプリを移行する方法の 1 つは、IIS サブアプリケーションを作成し、アプリの URL 構造を維持しながら ASP.NET 4.x から ASP.NET Core に特定のルートのみを移動することです。 たとえば、*applicationHost.config* ファイルからのアプリの URL 構造について考えてみます。

```xml
<sites>
    <site name="Default Web Site" id="1" serverAutoStart="true">
        <application path="/">
            <virtualDirectory path="/" physicalPath="D:\sites\MainSite\" />
        </application>
        <application path="/api" applicationPool="DefaultAppPool">
            <virtualDirectory path="/" physicalPath="D:\sites\netcoreapi" />
        </application>
        <bindings>
            <binding protocol="http" bindingInformation="*:80:" />
            <binding protocol="https" bindingInformation="*:443:" sslFlags="0" />
        </bindings>
    </site>
    ...
</sites>
```

ディレクトリの構造:

```
.
├── MainSite
│   ├── ...
│   └── Web.config
└── NetCoreApi
    ├── ...
    └── web.config
```

## <a name="bind-and-input-formatters"></a>[BIND] と入力フォーマッタ

[以前のバージョンの ASP.NET](/aspnet/mvc/overview/getting-started/introduction/examining-the-edit-methods-and-edit-view) では、過剰ポスティング攻撃からの保護に `[Bind]` 属性が使用されていました。 [入力フォーマッタ](xref:mvc/models/model-binding#input-formatters)は ASP.NET Core では動作が異なります。 `[Bind]` 属性は、入力フォーマッタと共に使用して JSON または XML を解析するときに、過剰ポスティングを防ぐように設計されなくなりました。 データのソースが `x-www-form-urlencoded` コンテンツ タイプでポストされたフォーム データである場合、これらの属性はモデル バインドに影響します。

JSON 情報をコントローラーにポストし、JSON 入力フォーマッタを使用してデータを解析するアプリの場合は、`[Bind]` 属性を、`[Bind]` 属性で定義されるプロパティと一致するビュー モデルに置き換えることをお勧めします。

## <a name="additional-resources"></a>その他の技術情報

- [.NET Core にライブラリを移植する](/dotnet/core/porting/libraries)
