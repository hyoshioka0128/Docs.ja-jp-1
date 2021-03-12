---
title: ASP.NET Core Blazor 用のリンカーを構成する
author: guardrex
description: Blazor アプリを構築するときに、中間言語 (IL) リンカーを制御する方法について説明します。
monikerRange: '>= aspnetcore-3.1 < aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 05/19/2020
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
uid: blazor/host-and-deploy/configure-linker
ms.openlocfilehash: af3c059e7192d6b0d2b0a902b6e3a6121fdf6709
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/06/2021
ms.locfileid: "102395033"
---
# <a name="configure-the-linker-for-aspnet-core-blazor"></a>ASP.NET Core Blazor 用のリンカーを構成する

Blazor WebAssembly では、ビルド中に[中間言語 (IL)](/dotnet/standard/managed-code#intermediate-language--execution) のリンクが実行されて、アプリの出力アセンブリから不要な IL がトリミングされます。 デバッグ構成でビルドすると、リンカーは無効になります。 リンカーを有効にするには、アプリをリリース構成でビルドする必要があります。 Blazor WebAssembly アプリを配置する場合は、リリースでビルドすることをお勧めします。 

アプリをリンクするとサイズが最適化されますが、悪影響を及ぼす可能性があります。 リフレクションや関連する動的機能を使用するアプリは、トリミングされたときに中断する可能性があります。リンカーがこの動的な動作を認識せず、通常は実行時にリフレクションに必要な型を特定できないためです。 そのようなアプリをトリミングするには、コードと、アプリが依存しているパッケージまたはフレームワークのリフレクションで必要なすべての型を、リンカーに通知する必要があります。

トリミングされたアプリが配置後に正しく動作するには、開発中にアプリのリリース ビルドを頻繁にテストすることが重要です。

Blazor アプリのリンクは、次の MSBuild 機能を使用して構成できます。

* [MSBuild プロパティ](#control-linking-with-an-msbuild-property)を使ってリンクをグローバルに構成する。
* [構成ファイル](#control-linking-with-a-configuration-file)を使ってアセンブリごとにリンクを制御する。

## <a name="control-linking-with-an-msbuild-property"></a>MSBuild プロパティを使ってリンクを制御する

リンクは、アプリが `Release` 構成でビルドされると有効になります。 これを変更するには、プロジェクト ファイルで `BlazorWebAssemblyEnableLinking` の MSBuild プロパティを構成します。

```xml
<PropertyGroup>
  <BlazorWebAssemblyEnableLinking>false</BlazorWebAssemblyEnableLinking>
</PropertyGroup>
```

## <a name="control-linking-with-a-configuration-file"></a>構成ファイルを使ってリンクを制御する

XML の構成ファイルを用意してそのファイルをプロジェクト ファイル内で MSBuild 項目として指定することで、アセンブリごとにリンクを制御します。

```xml
<ItemGroup>
  <BlazorLinkerDescriptor Include="LinkerConfig.xml" />
</ItemGroup>
```

`LinkerConfig.xml`:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--
  This file specifies which parts of the BCL or Blazor packages must not be
  stripped by the IL Linker even if they aren't referenced by user code.
-->
<linker>
  <assembly fullname="mscorlib">
    <!--
      Preserve the methods in WasmRuntime because its methods are called by 
      JavaScript client-side code to implement timers.
      Fixes: https://github.com/dotnet/blazor/issues/239
    -->
    <type fullname="System.Threading.WasmRuntime" />
  </assembly>
  <assembly fullname="System.Core">
    <!--
      System.Linq.Expressions* is required by Json.NET and any 
      expression.Compile caller. The assembly isn't stripped.
    -->
    <type fullname="System.Linq.Expressions*" />
  </assembly>
  <!--
    In this example, the app's entry point assembly is listed. The assembly
    isn't stripped by the IL Linker.
  -->
  <assembly fullname="MyCoolBlazorApp" />
</linker>
```

詳細と例については、[データ形式 (mono/linker GitHub リポジトリ)](https://github.com/mono/linker/blob/main/docs/data-formats.md) を参照してください。

## <a name="add-an-xml-linker-configuration-file-to-a-library"></a>XML リンカー構成ファイルをライブラリに追加する

特定のライブラリ用にリンカーを構成するには、XML リンカー構成ファイルを埋め込みリソースとしてライブラリに追加します。 埋め込みリソースの名前は、アセンブリと同じにする必要があります。

次の例では、`LinkerConfig.xml` ファイルが、ライブラリのアセンブリと同じ名前を持つ埋め込みリソースとして指定されています。

```xml
<ItemGroup>
  <EmbeddedResource Include="LinkerConfig.xml">
    <LogicalName>$(MSBuildProjectName).xml</LogicalName>
  </EmbeddedResource>
</ItemGroup>
```

### <a name="configure-the-linker-for-internationalization"></a>国際化用にリンカーを構成する

既定では、Blazor WebAssembly アプリに対する Blazor のリンカー構成により、明示的に要求されたロケールを除き、国際化情報は除去されます。 これらのアセンブリを削除すると、アプリのサイズが最小限に抑えられます。

保持される I18N アセンブリを制御するには、プロジェクト ファイルで MSBuild のプロパティ `<BlazorWebAssemblyI18NAssemblies>` を設定します。

```xml
<PropertyGroup>
  <BlazorWebAssemblyI18NAssemblies>{all|none|REGION1,REGION2,...}</BlazorWebAssemblyI18NAssemblies>
</PropertyGroup>
```

| リージョンの値     | Mono のリージョン アセンブリ    |
| ---------------- | ----------------------- |
| `all`            | すべてのアセンブリが含まれます |
| `cjk`            | `I18N.CJK.dll`          |
| `mideast`        | `I18N.MidEast.dll`      |
| `none` (既定値) | None                    |
| `other`          | `I18N.Other.dll`        |
| `rare`           | `I18N.Rare.dll`         |
| `west`           | `I18N.West.dll`         |

複数の値を区切るにはコンマを使用します (例: `mideast,west`)。

詳しくは、「[I18N: Pnetlib 国際化フレームワーク ライブラリ (mono/mono GitHub リポジトリ)](https://github.com/mono/mono/tree/master/mcs/class/I18N)」をご覧ください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:blazor/webassembly-performance-best-practices#intermediate-language-il-linking>
