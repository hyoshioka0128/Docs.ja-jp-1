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
ms.openlocfilehash: 8772f383a830936881564ca95a7f034ba08a5798
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551589"
---
`FetchData` コンポーネントは次の方法を示します。

* アクセス トークンをプロビジョニングする。
* アクセス トークンを使用して、*Server* アプリで保護されたリソース API を呼び出す。

[`@attribute [Authorize]`](xref:mvc/views/razor#attribute) ディレクティブは、このコンポーネントにアクセスするためにユーザーを承認する必要があることを Blazor WebAssembly の承認システムに示します。 `Client` アプリに属性が存在しても、適切な資格情報を使用せずにサーバー上の API が呼び出されるのを防ぐことはできません。 また、`Server` アプリは、適切なエンドポイントでそれを正しく保護するために `[Authorize]` も使用する必要があります。

<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.IAccessTokenProvider.RequestAccessToken%2A?displayProperty=nameWithType> は、API を呼び出すために要求に追加できるアクセス トークンの要求を処理します。 トークンがキャッシュされている場合、またはサービスがユーザーの操作なしで新しいアクセス トークンをプロビジョニングできる場合、トークン要求は成功します。 それ以外の場合、トークン要求は、[`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) ステートメントでキャッチされた <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenNotAvailableException> により失敗します。

要求に含める実際のトークンを取得するには、アプリが [`tokenResult.TryGetToken(out var token)`](xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenResult.TryGetToken%2A) を呼び出して、要求が成功したことを確認する必要があります。

要求が成功すると、トークン変数にアクセス トークンが設定されます。 トークンの <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessToken.Value?displayProperty=nameWithType> プロパティは、`Authorization` 要求ヘッダーに含めるリテラル文字列を公開します。

ユーザーの操作なしでトークンをプロビジョニングできなかったために要求が失敗した場合、トークンの結果にリダイレクト URL が含まれます。 この URL に移動すると、認証が成功した後、ユーザーがログイン ページに移動してから、現在のページに戻ります。

```razor
@page "/fetchdata"
@using Microsoft.AspNetCore.Authorization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@using {APP NAMESPACE}.Shared
@attribute [Authorize]
@inject HttpClient Http

...

@code {
    private WeatherForecast[] forecasts;

    protected override async Task OnInitializedAsync()
    {
        try
        {
            forecasts = await Http.GetFromJsonAsync<WeatherForecast[]>("WeatherForecast");
        }
        catch (AccessTokenNotAvailableException exception)
        {
            exception.Redirect();
        }
    }
}
```
