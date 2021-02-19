---
title: クラス ライブラリで ASP.NET Core API を使用する
author: scottaddie
description: クラス ライブラリで ASP.NET Core API を使用する方法を説明します。
ms.author: scaddie
ms.custom: mvc
ms.date: 12/16/2019
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
uid: fundamentals/target-aspnetcore
ms.openlocfilehash: c012658a6f48247af60c8bfd56a7d987f6aa8a68
ms.sourcegitcommit: c1839f2992b003c92cd958244a2e0771ae928786
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/05/2021
ms.locfileid: "93061510"
---
# <a name="use-aspnet-core-apis-in-a-class-library"></a>クラス ライブラリで ASP.NET Core API を使用する

作成者: [Scott Addie](https://github.com/scottaddie)

このドキュメントでは、クラス ライブラリで ASP.NET Core API を使用するためのガイダンスを示します。 他のすべてのライブラリ ガイダンスについては、[オープン ソース ライブラリのガイダンス](/dotnet/standard/library-guidance/)を参照してください。

## <a name="determine-which-aspnet-core-versions-to-support"></a>サポートする ASP.NET Core のバージョンを決定する

ASP.NET Core は、[.NET Core サポート ポリシー](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)に準拠しています。 ライブラリでサポートする ASP.NET Core のバージョンを決定する際は、このサポート ポリシーを参照してください。 ライブラリは次の条件を満たす必要があります。

* 可能な限り、*長期サポート* (LTS) として分類されている ASP.NET Core バージョンをすべてサポートします。
* *サポート終了* (EOL) として分類されている ASP.NET Core バージョンをサポートする必要はありません。

ASP.NET Core のプレビュー リリースが利用可能になると、破壊的変更が [aspnet/Announcements](https://github.com/aspnet/Announcements/issues) GitHub リポジトリに投稿されます。 ライブラリの互換性テストは、フレームワーク機能の開発中に実施できます。

## <a name="use-the-aspnet-core-shared-framework"></a>ASP.NET Core 共有フレームワークの使用

.NET Core 3.0 のリリースから、多数の ASP.NET Core アセンブリがパッケージとして NuGet に公開されなくなりました。 代わりに、アセンブリは `Microsoft.AspNetCore.App` 共有フレームワークに含まれ、.NET Core SDK およびランタイム インストーラーと共にインストールされます。 公開されなくなったパッケージの一覧については、「[古いパッケージ参照の削除](xref:migration/22-to-30#remove-obsolete-package-references)」を参照してください。

.NET Core 3.0 から、`Microsoft.NET.Sdk.Web` MSBuild SDK を使用するプロジェクトは、共有フレームワークを暗黙的に参照します。 `Microsoft.NET.Sdk` SDK または `Microsoft.NET.Sdk.Razor` SDK を使用するプロジェクトで共有フレームワーク内の ASP.NET Core API を使用するには、ASP.NET Core を参照する必要があります。

ASP.NET Core を参照するには、次の `<FrameworkReference>` 要素をプロジェクト ファイルに追加します。

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-basic-library.csproj?highlight=8)]

ASP.NET Core を参照するためのこの方法は、.NET Core 3.x を対象とするプロジェクトでのみサポートされます。

## <a name="include-blazor-extensibility"></a>Blazor 拡張機能を含める

Blazor は、WebAssembly (WASM) [ホスティング モデル](xref:blazor/hosting-models)と Server ホスティング モデルをサポートします。 特別な理由がない限り、[Razor コンポーネント](xref:blazor/components/index)は両方のホスティング モデルをサポートする必要があります。 Razor コンポーネント ライブラリでは、[Microsoft.NET.Sdk.Razor SDK](xref:razor-pages/sdk) を使用する必要があります。

### <a name="support-both-hosting-models"></a>両方のホスティング モデルをサポートする

[Blazor Server](xref:blazor/hosting-models#blazor-server) プロジェクトと [Blazor WASM](xref:blazor/hosting-models#blazor-webassembly) プロジェクトの両方で Razor コンポーネントの使用をサポートするには、ご使用のエディターに応じて次の手順を使用します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

**Razor クラス ライブラリ** プロジェクト テンプレートを使用します。 このテンプレートの **[Support pages and views]\(ページとビューのサポート\)** チェックボックスの選択を解除する必要があります。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

統合ターミナルで次のコマンドを実行します。

```dotnetcli
dotnet new razorclasslib
```

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

**Razor クラス ライブラリ** プロジェクト テンプレートを使用します。

---

このテンプレートから生成されるプロジェクトは、次の操作を実行します。

* .NET Standard 2.0 を対象とします。
* `RazorLangVersion` プロパティを `3.0` に設定します。 `3.0` は、.NET Core 3.x の既定値です。
* 次のパッケージ参照を追加します。
  * [Microsoft.AspNetCore.Components](https://www.nuget.org/packages/Microsoft.AspNetCore.Components)
  * [Microsoft.AspNetCore.Components.Web](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.Web)

次に例を示します。

[!code-xml[](target-aspnetcore/samples/single-tfm/netstandard2.0-razor-components-library.csproj)]

### <a name="support-a-specific-hosting-model"></a>特定のホスティング モデルをサポートする

1 つの Blazor ホスティング モデルだけをサポートするのは、あまり一般的ではありません。 たとえば、[Blazor Server](xref:blazor/hosting-models#blazor-server) プロジェクトでのみ Razor コンポーネントの使用をサポートするには、次の操作を行います。

* .NET Core 3.x を対象とします。
* 共有フレームワークの `<FrameworkReference>` 要素を追加します。

次に例を示します。

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-razor-components-library.csproj)]

Razor コンポーネントを含むライブラリの詳細については、「[ASP.NET Core Razor コンポーネント クラス ライブラリ](xref:blazor/components/class-libraries)」を参照してください。

## <a name="include-mvc-extensibility"></a>MVC 拡張機能を含める

このセクションでは、次のものを含むライブラリの推奨事項について概要を説明します。

* [Razor ビューまたは Razor Pages](#razor-views-or-razor-pages)
* [タグ ヘルパー](#tag-helpers)
* [ビューのコンポーネント](#view-components)

このセクションでは、MVC の複数のバージョンをサポートするマルチターゲット機能については説明しません。 複数の ASP.NET Core バージョンのサポートに関するガイダンスについては、「[複数のバージョンの ASP.NET Core をサポートする](#support-multiple-aspnet-core-versions)」を参照してください。

### <a name="razor-views-or-razor-pages"></a>Razor ビューまたは Razor Pages

[Razor ビュー](xref:mvc/views/overview)または [Razor Pages](xref:razor-pages/index) を含むプロジェクトでは、[Microsoft.NET.Sdk.Razor SDK](xref:razor-pages/sdk) を使用する必要があります。

プロジェクトが .NET Core 3.x を対象とする場合、次のものが必要です。

* `true` に設定された `AddRazorSupportForMvc` MSBuild プロパティ。
* 共有フレームワークの `<FrameworkReference>` 要素。

**Razor クラス ライブラリ** プロジェクト テンプレートは、.NET Core 3.x を対象とするプロジェクトについて、前述の要件を満たしています。 ご使用のエディターに応じて、次の手順を使用します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

**Razor クラス ライブラリ** プロジェクト テンプレートを使用します。 このテンプレートの **[Support pages and views]\(ページとビューのサポート\)** チェックボックスをオンにする必要があります。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

統合ターミナルで次のコマンドを実行します。

```dotnetcli
dotnet new razorclasslib -s
```

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

現時点では、サポートされているプロジェクト テンプレートはありません。

---

次に例を示します。

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-razor-views-pages-library.csproj)]

プロジェクトが代わりに NET Standard を対象とする場合、[Microsoft.AspNetCore.Mvc](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc) パッケージ参照が必要です。 `Microsoft.AspNetCore.Mvc` パッケージは ASP.NET Core 3.0 で共有フレームワークに移動されたため、公開されなくなりました。 次に例を示します。

[!code-xml[](target-aspnetcore/samples/single-tfm/netstandard2.0-razor-views-pages-library.csproj?highlight=8)]

### <a name="tag-helpers"></a>タグ ヘルパー

[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)を含むプロジェクトでは、`Microsoft.NET.Sdk` SDK を使用する必要があります。 .NET Core 3.x を対象とする場合、共有フレームワークの `<FrameworkReference>` 要素を追加します。 次に例を示します。

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-basic-library.csproj)]

.NET Standard を対象とする場合 (ASP.NET Core 3.x より前のバージョンをサポートするため)、[Microsoft.AspNetCore.Mvc.Razor](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Razor) へのパッケージ参照を追加します。 `Microsoft.AspNetCore.Mvc.Razor` パッケージは共有フレームワークに移動されたため、公開されなくなりました。 次に例を示します。

[!code-xml[](target-aspnetcore/samples/single-tfm/netstandard2.0-tag-helpers-library.csproj)]

### <a name="view-components"></a>ビュー コンポーネント

[ビュー コンポーネント](xref:mvc/views/view-components)を含むプロジェクトでは、`Microsoft.NET.Sdk` SDK を使用する必要があります。 .NET Core 3.x を対象とする場合、共有フレームワークの `<FrameworkReference>` 要素を追加します。 次に例を示します。

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-basic-library.csproj)]

.NET Standard を対象とする場合 (ASP.NET Core 3.x より前のバージョンをサポートするため)、[Microsoft.AspNetCore.Mvc.ViewFeatures](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.ViewFeatures) へのパッケージ参照を追加します。 `Microsoft.AspNetCore.Mvc.ViewFeatures` パッケージは共有フレームワークに移動されたため、公開されなくなりました。 次に例を示します。

[!code-xml[](target-aspnetcore/samples/single-tfm/netstandard2.0-view-components-library.csproj)]

## <a name="support-multiple-aspnet-core-versions"></a>複数のバージョンの ASP.NET Core をサポートする

ASP.NET Core の複数のバリアントをサポートするライブラリを作成するには、マルチターゲット機能が必要です。 タグ ヘルパー ライブラリで ASP.NET Core の次のバリアントをサポートする必要があるシナリオについて考えてみましょう。

* .NET Framework 4.6.1 を対象とする ASP.NET Core 2.1
* .NET Core 2.x を対象とする ASP.NET Core 2.x
* .NET Core 3.x を対象とする ASP.NET Core 3.x

次のプロジェクト ファイルでは、`TargetFrameworks` プロパティを使用してこれらのバリアントをサポートします。

[!code-xml[](target-aspnetcore/samples/multi-tfm/recommended-tag-helpers-library.csproj)]

上記のプロジェクト ファイルでは、次のことが実行されます。

* すべてのコンシューマー向けに `Markdig` パッケージが追加されます。
* [Microsoft.AspNetCore.Mvc.Razor](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Razor) への参照 が、.NET Framework 4.6.1 以降または .NET Core 2.x を対象とするコンシューマー向けに追加されます。 パッケージのバージョン 2.1.0 は、下位互換性のために ASP.NET Core 2.2 で動作します。
* 共有フレームワークは、.NET Core 3.x を対象とするコンシューマー向けに参照されます。 `Microsoft.AspNetCore.Mvc.Razor` パッケージは、共有フレームワークに含まれます。

また、.NET Core 2.1 と .NET Framework 4.6.1 の両方を対象とする場合、代わりに .NET Standard 2.0 を対象とすることもできます。

[!code-xml[](target-aspnetcore/samples/multi-tfm/alternative-tag-helpers-library.csproj?highlight=4)]

前述のプロジェクト ファイルでは、次の点に注意する必要があります。

* ライブラリにはタグ ヘルパーのみが含まれているため、ASP.NET Core が実行される特定のプラットフォーム (.NET Core および .NET Framework) を対象とする方が簡単です。 タグ ヘルパーは、他の .NET Standard 2.0 準拠のターゲット フレームワーク (Unity、UWP、Xamarin など) で使用することはできません。
* .NET Framework から .NET Standard 2.0 を使用する場合、いくつかの問題がありますが、.NET Framework 4.7.2 で対処されました。 .NET Framework 4.6.1 から 4.7.1 までを使用するコンシューマーの場合、.NET Framework 4.6.1 を対象とすることにより、エクスペリエンスを向上させることができます。

ライブラリでプラットフォーム固有の API を呼び出す必要がある場合、.NET Standard の代わりに特定の .NET 実装を対象とします。 詳細については、「[マルチターゲット](/dotnet/standard/library-guidance/cross-platform-targeting#multi-targeting)」を参照してください。

## <a name="use-an-api-that-hasnt-changed"></a>変更されていない API を使用する

ミドルウェア ライブラリを .NET Core 2.2 から 3.0 にアップグレードするシナリオについて考えてみましょう。 ライブラリで使用される ASP.NET Core ミドルウェア API は、ASP.NET Core 2.2 と 3.0 間で変更されていません。 .NET Core 3.0 でミドルウェア ライブラリのサポートを継続するには、次の手順を行います。

* [標準ライブラリのガイダンス](/dotnet/standard/library-guidance/)に従います。
* 対応するアセンブリが共有フレームワークに存在しない場合は、各 API の NuGet へのパッケージ参照を追加します。

## <a name="use-an-api-that-changed"></a>変更された API を使用する

ライブラリを .NET Core 2.2 から 3.0 にアップグレードするシナリオについて考えてみましょう。 ライブラリで使用される ASP.NET Core API は、ASP.NET Core 3.0 で[破壊的変更](/dotnet/core/compatibility/breaking-changes)が行われています。 破壊された API をすべてのバージョンで使用しないようにライブラリを書き換えることができるかどうかを検討します。

ライブラリを書き換えることができる場合は書き換えて、パッケージ参照を使用して以前のターゲット フレームワーク (たとえば、.NET Standard 2.0 や .NET Framework 4.6.1 など) を引き続き対象とします。

ライブラリを書き換えることができない場合は、次の手順を行います。

* .NET Core 3.0 のターゲットを追加します。
* 共有フレームワークの `<FrameworkReference>` 要素を追加します。
* 条件付きでコードをコンパイルするには、適切なターゲット フレームワーク シンボルを設定した [#if プリプロセッサ ディレクティブ](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-if)を使用します。

たとえば、ASP.NET Core 3.0 から、HTTP 要求および応答ストリームでの同期読み取りと同期書き込みは既定で無効になっています。 ASP.NET Core 2.2 では、この同期動作が既定でサポートされています。 I/O の発生時に同期読み取りと同期書き込みを有効にする必要があるミドルウェア ライブラリについて検討します。 ライブラリで、同期機能を有効にするコードを適切なプリプロセッサ ディレクティブで囲む必要があります。 次に例を示します。

[!code-csharp[](target-aspnetcore/samples/middleware.cs?highlight=9-24)]

## <a name="use-an-api-introduced-in-30"></a>3\.0 で導入された API を使用する

ASP.NET Core 3.0 で導入された ASP.NET Core API を使用する必要がある場合について考えてみましょう。 以下の質問を検討します。

1. ライブラリの機能上、新しい API は必要ですか?
1. ライブラリで、この機能を別の方法で実装できますか?

ライブラリの機能上、新しい API が必要であり、それをダウンレベルで実装する方法がない場合:

* .NET Core 3.x のみを対象とします。
* 共有フレームワークの `<FrameworkReference>` 要素を追加します。

ライブラリでこの機能を別の方法で実装できる場合:

* NET Core 3.x をターゲット フレームワークとして追加します。
* 共有フレームワークの `<FrameworkReference>` 要素を追加します。
* 条件付きでコードをコンパイルするには、適切なターゲット フレームワーク シンボルを設定した [#if プリプロセッサ ディレクティブ](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-if)を使用します。

たとえば、次のタグ ヘルパーは、ASP.NET Core 3.0 で導入された <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> インターフェイスを使用します。 .NET Core 3.0 を対象とするコンシューマーは、`NETCOREAPP3_0` ターゲット フレームワーク シンボルで定義されたコード パスを実行します。 NET Core 2.1 および .NET Framework 4.6.1 のコンシューマーの場合、タグ ヘルパーのコンストラクター パラメーターの型が <xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment> に変更されます。 ASP.NET Core 3.0 で `IHostingEnvironment` が廃止としてマークされ、代わりに `IWebHostEnvironment` を推奨しているため、この変更が必要です。

```csharp
[HtmlTargetElement("script", Attributes = "asp-inline")]
public class ScriptInliningTagHelper : TagHelper
{
    private readonly IFileProvider _wwwroot;

#if NETCOREAPP3_0
    public ScriptInliningTagHelper(IWebHostEnvironment env)
#else
    public ScriptInliningTagHelper(IHostingEnvironment env)
#endif
    {
        _wwwroot = env.WebRootFileProvider;
    }

    // code omitted for brevity
}
```

次のマルチターゲット プロジェクト ファイルでは、このタグ ヘルパーのシナリオに対応しています。

[!code-xml[](target-aspnetcore/samples/multi-tfm/recommended-tag-helpers-library.csproj)]

## <a name="use-an-api-removed-from-the-shared-framework"></a>共有フレームワークから削除された API を使用する

共有フレームワークから削除された ASP.NET Core アセンブリを使用するには、適切なパッケージ参照を追加します。 ASP.NET Core 3.0 で共有フレームワークから削除されたパッケージの一覧については、「[古いパッケージ参照の削除](xref:migration/22-to-30#remove-obsolete-package-references)」を参照してください。

たとえば、Web API クライアントを追加するプロジェクト ファイルは、次のようになります。

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netcoreapp3.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <FrameworkReference Include="Microsoft.AspNetCore.App" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNet.WebApi.Client" Version="5.2.7" />
  </ItemGroup>

</Project>
```

## <a name="additional-resources"></a>その他の技術情報

* <xref:razor-pages/ui-class>
* <xref:blazor/components/class-libraries>
* [.NET 実装のサポート](/dotnet/standard/net-standard#net-implementation-support)
* [.NET サポート ポリシー](https://dotnet.microsoft.com/platform/support/policy)
