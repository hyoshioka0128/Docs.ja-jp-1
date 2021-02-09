---
title: ASP.NET Core Blazor の CSS の分離
author: daveabrock
description: CSS の分離を使用して、コンポーネントに CSS のスコープを設定する方法について説明します。これにより、CSS が簡素化され、他のコンポーネントやライブラリとの競合を回避できます。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/20/2020
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
uid: blazor/components/css-isolation
ms.openlocfilehash: 0748f606314963788e6733ca2ae2ca2123d839b3
ms.sourcegitcommit: e311cfb77f26a0a23681019bd334929d1aaeda20
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/03/2021
ms.locfileid: "99529983"
---
# <a name="aspnet-core-blazor-css-isolation"></a>ASP.NET Core Blazor の CSS の分離

作成者: [Dave Brock](https://twitter.com/daveabrock)

CSS の分離により、グローバル スタイルへの依存を防ぐことで、アプリの CSS 占有領域を簡素化し、コンポーネントおよびライブラリ間のスタイルの競合を回避するのに役立ちます。

## <a name="enable-css-isolation"></a>CSS の分離を有効にする 

コンポーネント固有のスタイルを定義するには、同じフォルダー内のコンポーネントの `.razor` ファイルの名前と一致する `.razor.css` ファイルを作成します。 `.razor.css` ファイルは、"*スコープ付き CSS ファイル*" です。 

`Example.razor` ファイル内の `Example` コンポーネントの場合、コンポーネントと共に `Example.razor.css` という名前のファイルを作成します。 `Example.razor.css` ファイルは、`Example` コンポーネント (`Example.razor`) と同じフォルダー内に存在する必要があります。 ファイルの "`Example`" ベース名では、大文字と小文字が区別 **されません**。

`Pages/Example.razor`:

```razor
@page "/example"

<h1>Scoped CSS Example</h1>
```

`Pages/Example.razor.css`:

```css
h1 { 
    color: brown;
    font-family: Tahoma, Geneva, Verdana, sans-serif;
}
```

**`Example.razor.css` で定義されるスタイルは、`Example` コンポーネントのレンダリングされる出力にのみ適用されます。** CSS の分離は、一致する Razor ファイル内の HTML 要素に適用されます。 アプリ内の他の場所で定義されるどの `h1` CSS 宣言も、`Example` コンポーネントのスタイルと競合しません。

> [!NOTE]
> バンドルが発生したときにスタイルの分離を保証するために、Razor コード ブロックでの CSS のインポートはサポートされていません。

## <a name="css-isolation-bundling"></a>CSS の分離のバンドル

CSS の分離は、ビルド時に発生します。 このプロセス中、Blazor により、コンポーネントによってレンダリングされるマークアップと一致するように CSS セレクターが書き換えられます。 これらの書き換えられた CSS スタイルは、`{PROJECT NAME}.styles.css` でバンドルされ、静的アセットとして生成されます。ここで、プレースホルダー `{PROJECT NAME}` は、参照されるパッケージまたは製品名です。

既定では、これらの静的ファイルはアプリのルート パスから参照されます。 アプリでは、生成された HTML の `<head>` タグ内の参照を調べることにより、バンドルされたファイルを参照します。

```html
<link href="ProjectName.styles.css" rel="stylesheet">
```

バンドルされたファイル内で、各コンポーネントはスコープ識別子に関連付けられます。 スタイルが設定されるコンポーネントごとに、HTML 属性が形式 `b-<10-character-string>` を使用して追加されます。 識別子は一意であり、アプリごとに異なります。 レンダリングされる `Counter` コンポーネントでは、Blazor により、スコープ識別子が `h1` 要素に追加されます。

```html
<h1 b-3xxtam6d07>
```

`ProjectName.styles.css` ファイルは、スコープ識別子を使用してスタイル宣言をそのコンポーネントと共にグループ化します。 次の例では、前述の `<h1>` 要素のスタイルを示します。

```css
/* /Pages/Counter.razor.rz.scp.css */
h1[b-3xxtam6d07] {
    color: brown;
}
```

ビルド時、規則 `{STATIC WEB ASSETS BASE PATH}/Project.lib.scp.css` でプロジェクト バンドルが作成されます。ここで、プレースホルダー `{STATIC WEB ASSETS BASE PATH}` は静的な Web アセットのベース パスです。

NuGet パッケージや [Razor クラス ライブラリ](xref:blazor/components/class-libraries)などの他のプロジェクトを利用する場合、バンドル ファイルは次のようになります。

* CSS インポートを使用してスタイルを参照する。
* スタイルを使用するアプリの静的な Web アセットとして公開されない。

## <a name="child-component-support"></a>子コンポーネントのサポート

既定で、CSS の分離は形式 `{COMPONENT NAME}.razor.css` に関連付けられたコンポーネントにのみ適用されます。ここで、プレースホルダー `{COMPONENT NAME}` は、通常、コンポーネント名です。 子コンポーネントに変更を適用するには、親コンポーネントの `.razor.css` ファイル内の子孫要素に `::deep` 連結子を使用します。 `::deep` 連結子により、要素の生成されたスコープ識別子の "*子孫*" である要素が選択されます。 

次の例は、`Child` という名前の子コンポーネントを持つ `Parent` という名前の親コンポーネントを示します。

`Pages/Parent.razor`:

```razor
@page "/parent"

<div>
    <h1>Parent component</h1>

    <Child />
</div>
```

`Shared/Child.razor`:

```razor
<h1>Child Component</h1>
```

`::deep` 連結子を使用して `Parent.razor.css` 内の `h1` 宣言を更新し、`h1` スタイル宣言を親コンポーネントとその子に適用する必要があることを示します。

`Pages/Parent.razor.css`:

```css
::deep h1 { 
    color: red;
}
```

これで、子コンポーネント用の個別のスコープ付き CSS ファイルを作成する必要なく、`h1` スタイルは、`Parent` と `Child` の各コンポーネントに適用されます。

`::deep` 連結子は、子孫要素でのみ機能します。 次のマークアップでは、想定どおりに `h1` スタイルがコンポーネントに適用されます。 親コンポーネントのスコープ識別子が `div` 要素に適用されるため、ブラウザーでは、スタイルを親コンポーネントから継承することが認識されます。

`Pages/Parent.razor`:

```razor
<div>
    <h1>Parent</h1>

    <Child />
</div>
```

ただし、`div` 要素を除外すると、子孫関係が削除されます。 次の例では、スタイルは子コンポーネントに適用 **されません**。

`Pages/Parent.razor`:

```razor
<h1>Parent</h1>

<Child />
```

## <a name="css-preprocessor-support"></a>CSS プリプロセッサのサポート

CSS プリプロセッサは、変数、入れ子、モジュール、mixin、継承などの機能を利用することで CSS 開発を改善するのに役立ちます。 CSS の分離は、SASS や LESS などの CSS プリプロセッサをネイティブにサポートしていませんが、ビルド プロセス中に Blazor により CSS セレクターが書き換えられる前にプリプロセッサのコンパイルが行われる限り、CSS プリプロセッサの統合はシームレスです。 たとえば、Visual Studio を使用して、Visual Studio タスク ランナー エクスプローラーで既存のプリプロセッサ コンパイルを **ビルド前** タスクとして構成します。

[Delegate.SassBuilder](https://www.nuget.org/packages/Delegate.SassBuilder) などの多くのサードパーティ製 NuGet パッケージは、CSS の分離が発生する前に、ビルド プロセスの開始時に SASS または SCSS ファイルをコンパイルできます。追加の構成は必要ありません。

## <a name="css-isolation-configuration"></a>CSS の分離の構成

CSS の分離は、すぐに使用できるように設計されていますが、既存のツールやワークフローへの依存関係がある場合など、一部の高度なシナリオ用の構成も用意されています。

### <a name="customize-scope-identifier-format"></a>スコープ識別子の形式をカスタマイズする

既定では、スコープ識別子に形式 `b-<10-character-string>` が使用されます。 スコープ識別子の形式をカスタマイズするには、プロジェクト ファイルを目的のパターンに更新します。

```xml
<ItemGroup>
  <None Update="Pages/Example.razor.css" CssScope="my-custom-scope-identifier" />
</ItemGroup>
```

前の例では、`Example.Razor.css` 用に生成された CSS により、そのスコープ識別子は `b-<10-character-string>` から `my-custom-scope-identifier` に変更されます。

スコープ識別子を使用して、スコープ付き CSS ファイルでの継承を実現します。 次のプロジェクト ファイルの例では、`BaseComponent.razor.css` ファイルに、コンポーネント間の共通スタイルが含まれています。 `DerivedComponent.razor.css` ファイルでは、これらのスタイルが継承されます。

```xml
<ItemGroup>
  <None Update="Pages/BaseComponent.razor.css" CssScope="my-custom-scope-identifier" />
  <None Update="Pages/DerivedComponent.razor.css" CssScope="my-custom-scope-identifier" />
</ItemGroup>
```

ワイルドカード (`*`) 演算子を使用して、複数のファイル間でスコープ識別子を共有します。

```xml
<ItemGroup>
  <None Update="Pages/*.razor.css" CssScope="my-custom-scope-identifier" />
</ItemGroup>
```

### <a name="change-base-path-for-static-web-assets"></a>静的な Web アセットのベース パスを変更する

`scoped.styles.css` ファイルは、アプリのルートで生成されます。 プロジェクト ファイルでは、`StaticWebAssetBasePath` プロパティを使用して既定のパスを変更します。 次の例では、`scoped.styles.css` ファイルとアプリの残りのアセットを `_content` パスに配置します。

```xml
<PropertyGroup>
  <StaticWebAssetBasePath>_content/$(PackageId)</StaticWebAssetBasePath>
</PropertyGroup>
```

### <a name="disable-automatic-bundling"></a>自動バンドルを無効にする

Blazor でスコープ付きファイルを公開し、それを実行時に読み込む方法をオプトアウトするには、`DisableScopedCssBundling` プロパティを使用します。 このプロパティを使用する場合、`obj` ディレクトリからの CSS ファイルの分離と、それらの公開および実行時の読み込みを他のツールまたはプロセスが担当することを意味します。

```xml
<PropertyGroup>
  <DisableScopedCssBundling>true</DisableScopedCssBundling>
</PropertyGroup>
```

## <a name="razor-class-library-rcl-support"></a>Razor クラス ライブラリ (RCL) のサポート

[Razor クラス ライブラリ (RCL)](xref:razor-pages/ui-class) により分離スタイルが提供される場合、`<link>` タグの `href` 属性は `{STATIC WEB ASSET BASE PATH}/{ASSEMBLY NAME}.bundle.scp.css` を指します。ここで、プレースホルダーは次のとおりです。

* `{STATIC WEB ASSET BASE PATH}`: 静的な Web 資産のベース パス。
* `{ASSEMBLY NAME}`: クラス ライブラリのアセンブリ名。

次に例を示します。

* 静的な Web 資産のベース パスは `_content/ClassLib` です。
* クラス ライブラリのアセンブリ名は `ClassLib` です。

`wwwroot/index.html` (Blazor WebAssembly) または `Pages_Host.cshtml` (Blazor Server):

```html
<link href="_content/ClassLib/ClassLib.bundle.scp.css" rel="stylesheet">
```

RCL およびコンポーネント ライブラリの詳細については、以下を参照してください。

* <xref:razor-pages/ui-class>
* <xref:blazor/components/class-libraries>.
