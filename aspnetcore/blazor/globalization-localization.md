---
title: ASP.NET Core Blazor のグローバリゼーションおよびローカライズ
author: guardrex
description: 複数のカルチャと言語でユーザーが Razor コンポーネントにアクセスできるようにする方法について学習します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 06/04/2020
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
uid: blazor/globalization-localization
ms.openlocfilehash: 8baee9bc0a8e569174f33dac6a406b2162d09552
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100279918"
---
# <a name="aspnet-core-blazor-globalization-and-localization"></a>ASP.NET Core Blazor のグローバリゼーションおよびローカライズ

複数のカルチャと言語でユーザーが Razor コンポーネントにアクセスできるようにすることができます。 利用できる .NET のグローバリゼーションおよびローカライズのシナリオは、次のとおりです。

* .NET のリソース システム
* カルチャ固有の数値と日付の書式

現在、サポートされている ASP.NET Core のローカライズ シナリオは限られています。

* <xref:Microsoft.Extensions.Localization.IStringLocalizer> と <xref:Microsoft.Extensions.Localization.IStringLocalizer%601> は、Blazor アプリで "*サポートされています*"。
* <xref:Microsoft.AspNetCore.Mvc.Localization.IHtmlLocalizer>、<xref:Microsoft.AspNetCore.Mvc.Localization.IViewLocalizer>、データ注釈のローカライズは ASP.NET Core の MVC シナリオであり、Blazor アプリでは "**サポートされていません**"。

詳細については、「<xref:fundamentals/localization>」を参照してください。

## <a name="globalization"></a>グローバリゼーション

Blazor の [`@bind`](xref:mvc/views/razor#bind) 機能は、ユーザーの現在のカルチャに基づいて、書式設定を実行し、表示する値を解析します。

現在のカルチャは、<xref:System.Globalization.CultureInfo.CurrentCulture?displayProperty=fullName> プロパティからアクセスできます。

<xref:System.Globalization.CultureInfo.InvariantCulture?displayProperty=nameWithType> は、次のフィールドの型 (`<input type="{TYPE}" />`) に使用されます。

* `date`
* `number`

前のフィールドの型は次のようになります。

* 適切なブラウザー ベースの書式ルールを使用して表示されます。
* 自由形式のテキストを含めることはできません。
* ブラウザーの実装に基づいてユーザー操作の特性を指定します。

次のフィールドの型には、特定の書式設定の要件がありますが、すべての主要なブラウザーでサポートされていないため、Blazor では現在サポートされていません。

* `datetime-local`
* `month`
* `week`

[`@bind`](xref:mvc/views/razor#bind) では、値を解析および書式設定するための <xref:System.Globalization.CultureInfo?displayProperty=fullName> を提供する `@bind:culture` パラメーターをサポートしています。 `date` および `number` のフィールドの型を使用する場合は、カルチャを指定しないことをお勧めします。 `date` および `number` には、必要なカルチャを提供する Blazor サポートが組み込まれています。

## <a name="localization"></a>ローカリゼーション

### Blazor WebAssembly

Blazor WebAssembly アプリでは、ユーザーの[言語設定](https://developer.mozilla.org/docs/Web/API/NavigatorLanguage/languages)を使用してカルチャが設定されます。

カルチャを明示的に構成するには、`Program.Main` で <xref:System.Globalization.CultureInfo.DefaultThreadCurrentCulture?displayProperty=nameWithType> と <xref:System.Globalization.CultureInfo.DefaultThreadCurrentUICulture?displayProperty=nameWithType> を設定します。

::: moniker range=">= aspnetcore-5.0"

既定では、Blazor WebAssembly には、ユーザーのカルチャで日付や通貨などの値を表示するために必要な最小限のグローバリゼーション リソースがあります。 カルチャの動的な変更をサポートする必要があるアプリケーションでは、プロジェクト ファイルで `BlazorWebAssemblyLoadAllGlobalizationData` を構成する必要があります。

```xml
<PropertyGroup>
  <BlazorWebAssemblyLoadAllGlobalizationData>true</BlazorWebAssemblyLoadAllGlobalizationData>
</PropertyGroup>
```

Blazor WebAssembly は、`Blazor.start` に渡されるオプションを使用して、特定のアプリケーション カルチャを使用して起動するように構成することもできます。 たとえば、次のサンプルでは、`en-GB` カルチャを使用して起動するように構成されたアプリが示されています。

```html
<script src="_framework/blazor.webassembly.js" autostart="false"></script>
<script>
  Blazor.start({
    applicationCulture: 'en-GB'
  });
</script>
```

`applicationCulture` の値は、[BCP-47 言語タグ形式](https://tools.ietf.org/html/bcp47)に準拠している必要があります。

アプリがローカライズを必要としない場合は、`en-US` カルチャに基づくインバリアント カルチャをサポートするようにアプリを構成することができます。

```xml
<PropertyGroup>
  <InvariantGlobalization>true</InvariantGlobalization>
</PropertyGroup>
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

既定では、Blazor WebAssembly アプリに対する中間言語 (IL) リンカーの構成によって、明示的に要求されたロケールを除き、国際化情報が削除されます。 詳細については、「<xref:blazor/host-and-deploy/configure-linker#configure-the-linker-for-internationalization>」を参照してください。

::: moniker-end

Blazor で既定で選択されるカルチャは、ほとんどのユーザーにとって十分と考えられますが、ユーザーが優先ロケールを指定する手段を提供することを検討してください。 カルチャ ピッカーを使用した Blazor WebAssembly サンプル アプリについては、[`LocSample`](https://github.com/pranavkm/LocSample) ローカライズのサンプル アプリを参照してください。

### Blazor Server

Blazor Server アプリは、[ローカライズ ミドルウェア](xref:fundamentals/localization#localization-middleware)を使用してローカライズされます。 ミドルウェアによって、アプリからリソースを要求するユーザーに対して適切なカルチャが選択されます。

カルチャは、次のいずれかの方法を使用して設定できます。

* [Cookie ](#cookies)
* [カルチャを選択するための UI を提供する](#provide-ui-to-choose-the-culture)

使用例を含む詳細については、「<xref:fundamentals/localization>」を参照してください。

#### <a name="cookies"></a>Cookies

ローカライズ カルチャの cookie では、ユーザーのカルチャが保持されます。 ローカライズ ミドルウェアでは、後続の要求で cookie を読み取り、ユーザーのカルチャを設定します。 

cookie を使用すると、WebSocket 接続によってカルチャを正しく伝達できることを確実にします。 ローカライズ スキームが URL パスまたはクエリ文字列に基づいている場合は、スキームが WebSocket を使用できない可能性があるため、カルチャを保持できません。 したがって、ローカライズ カルチャの cookie を使用することをお勧めします。

カルチャがローカライズ cookie で保持されている場合は、任意の手法を使用してカルチャを割り当てることができます。 アプリにサーバー側 ASP.NET Core 用に確立されたローカライズ スキームが既にある場合は、アプリの既存のローカライズ インフラストラクチャを引き続き使用し、アプリのスキーム内にローカライズ カルチャ cookie を設定します。

次の例では、ローカライズ ミドルウェアによって読み取ることができる cookie で現在のカルチャを設定する方法を示します。 `Pages/_Host.cshtml` ファイルの開いている `<body>` タグのすぐ下に、Razor 式を作成します。

```cshtml
@using System.Globalization
@using Microsoft.AspNetCore.Localization

...

<body>
    @{
        this.HttpContext.Response.Cookies.Append(
            CookieRequestCultureProvider.DefaultCookieName,
            CookieRequestCultureProvider.MakeCookieValue(
                new RequestCulture(
                    CultureInfo.CurrentCulture,
                    CultureInfo.CurrentUICulture)));
    }

    ...
</body>
```

ローカライズは、次の一連のイベントでアプリによって処理されます。

1. ブラウザーによって、アプリに最初の HTTP 要求が送信されます。
1. カルチャは、ローカライズ ミドルウェアによって割り当てられます。
1. `_Host` ページ (`_Host.cshtml`) の Razor 式によって、応答の一部として cookie にカルチャが保持されます。
1. ブラウザーによって、WebSocket 接続が開かれ、対話型の Blazor Server セッションが作成されます。
1. ローカライズ ミドルウェアによって、cookie が読み取られ、カルチャが割り当てられます。
1. Blazor Server セッションは、正しいカルチャで開始されます。

<xref:Microsoft.AspNetCore.Mvc.Razor.RazorPage> の操作時は <xref:Microsoft.AspNetCore.Mvc.Razor.RazorPage.Context> プロパティを使用します。

```razor
@{
    this.Context.Response.Cookies.Append(
        CookieRequestCultureProvider.DefaultCookieName,
        CookieRequestCultureProvider.MakeCookieValue(
            new RequestCulture(
                CultureInfo.CurrentCulture,
                CultureInfo.CurrentUICulture)));
}
```

#### <a name="provide-ui-to-choose-the-culture"></a>カルチャを選択するための UI を提供する

ユーザーがカルチャを選択できるように UI を提供するには、"*リダイレクト ベースのアプローチ*" をお勧めします。 このプロセスは、セキュリティで保護されたリソースにユーザーがアクセスしようとすると Web アプリで発生する処理に似ています。 ユーザーはサインイン ページにリダイレクトされ、元のリソースに再びリダイレクトされます。 

アプリでは、コントローラーへのリダイレクトによって、ユーザーが選択したカルチャが保持されます。 コントローラーによって、ユーザーが選択したカルチャが cookie に設定され、ユーザーは元の URI にリダイレクトされます。

cookie にユーザーが選択したカルチャを設定し、元の URI へのリダイレクトを実行するように、サーバー上に HTTP エンドポイントを確立します。

```csharp
[Route("[controller]/[action]")]
public class CultureController : Controller
{
    public IActionResult SetCulture(string culture, string redirectUri)
    {
        if (culture != null)
        {
            HttpContext.Response.Cookies.Append(
                CookieRequestCultureProvider.DefaultCookieName,
                CookieRequestCultureProvider.MakeCookieValue(
                    new RequestCulture(culture)));
        }

        return LocalRedirect(redirectUri);
    }
}
```

> [!WARNING]
> <xref:Microsoft.AspNetCore.Mvc.ControllerBase.LocalRedirect%2A> アクションの結果を使用して、オープン リダイレクト攻撃を防ぎます。 詳細については、「<xref:security/preventing-open-redirects>」を参照してください。

アプリがコントローラー アクションを処理するように構成されていない場合:

* 次のように `Startup.ConfigureServices` のサービス コレクションに MVC サービスを追加します。

  ```csharp
  services.AddControllers();
  ```

* 次のように `Startup.Configure` にコントローラーのエンドポイント ルーティングを追加します。

  ```csharp
  app.UseEndpoints(endpoints =>
  {
      endpoints.MapControllers();
      endpoints.MapBlazorHub();
      endpoints.MapFallbackToPage("/_Host");
  });
  ```

次のコンポーネントでは、ユーザーがカルチャを選択したときに、最初のリダイレクトを実行する方法の例を示します。

```razor
@inject NavigationManager NavigationManager

<h3>Select your language</h3>

<select @onchange="OnSelected">
    <option>Select...</option>
    <option value="en-US">English</option>
    <option value="fr-FR">Français</option>
</select>

@code {
    private void OnSelected(ChangeEventArgs e)
    {
        var culture = (string)e.Value;
        var uri = new Uri(NavigationManager.Uri)
            .GetComponents(UriComponents.PathAndQuery, UriFormat.Unescaped);
        var query = $"?culture={Uri.EscapeDataString(culture)}&" +
            $"redirectUri={Uri.EscapeDataString(uri)}";

        NavigationManager.NavigateTo("/Culture/SetCulture" + query, forceLoad: true);
    }
}
```

## <a name="additional-resources"></a>その他の技術情報

* <xref:fundamentals/localization>
