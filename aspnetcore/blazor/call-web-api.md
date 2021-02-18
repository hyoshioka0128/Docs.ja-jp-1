---
title: ASP.NET Core Blazor WebAssembly から Web API を呼び出す
author: guardrex
description: クロス オリジン リソース共有 (CORS) 要求の作成など、JSON ヘルパーを使用して Blazor WebAssembly アプリから web API を呼び出す方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 06/24/2020
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
uid: blazor/call-web-api
ms.openlocfilehash: b3c783623252512621a0cee7a3607c69cb6d09bb
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280279"
---
# <a name="call-a-web-api-from-aspnet-core-blazor"></a>ASP.NET Core Blazor から Web API を呼び出す

> [!NOTE]
> このトピックの対象は、Blazor WebAssembly です。 [Blazor Server](xref:blazor/hosting-models#blazor-server) アプリは、<xref:System.Net.Http.HttpClient> インスタンスを使用して Web API を呼び出します。これは通常、<xref:System.Net.Http.IHttpClientFactory> を使用して作成されます。 Blazor Server に適用されるガイダンスについては、「<xref:fundamentals/http-requests>」を参照してください。

[Blazor WebAssembly](xref:blazor/hosting-models#blazor-webassembly) アプリは、事前に構成された <xref:System.Net.Http.HttpClient> サービスを使用して Web API を呼び出します。 JavaScript [Fetch API](https://developer.mozilla.org/docs/Web/API/Fetch_API) オプションを含めることができる要求は、Blazor JSON ヘルパーまたは <xref:System.Net.Http.HttpRequestMessage> を使用して作成ます。 Blazor WebAssembly アプリの <xref:System.Net.Http.HttpClient> サービスは、発行元のサーバーに要求を戻すことに重点を置いています。 このトピックのガイダンスは、Blazor WebAssembly アプリにのみ関連します。

[サンプル コードを表示またはダウンロードする](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) ([ダウンロード方法](xref:index#how-to-download-a-sample)):`BlazorWebAssemblySample` アプリを選択します。

`BlazorWebAssemblySample` サンプル アプリで次のコンポーネントを確認します。

* Web API (`Pages/CallWebAPI.razor`) を呼び出す
* HTTP 要求テスター (`Components/HTTPRequestTester.razor`)

## <a name="packages"></a>パッケージ

プロジェクト ファイルで [`System.Net.Http.Json`](https://www.nuget.org/packages/System.Net.Http.Json) NuGet パッケージを参照します。

## <a name="add-the-httpclient-service"></a>HttpClient サービスを追加する

`Program.Main` で、<xref:System.Net.Http.HttpClient> サービスがまだ存在しない場合は追加します。

```csharp
builder.Services.AddScoped(sp => 
    new HttpClient
    {
        BaseAddress = new Uri(builder.HostEnvironment.BaseAddress)
    });
```

## <a name="httpclient-and-json-helpers"></a>HttpClient と JSON ヘルパー

Blazor WebAssembly アプリでは、[`HttpClient`](xref:fundamentals/http-requests) は、要求を要求元のサーバーに返すための構成済みのサービスとして利用できます。

Blazor Server アプリには、既定では <xref:System.Net.Http.HttpClient> サービスが含まれません。 [`HttpClient` ファクトリ インフラストラクチャ](xref:fundamentals/http-requests)を使用して、アプリに <xref:System.Net.Http.HttpClient> を提供します。

<xref:System.Net.Http.HttpClient> と JSON のヘルパーは、サード パーティの Web API エンドポイントを呼び出すためにも使用されます。 <xref:System.Net.Http.HttpClient> は、ブラウザーの [Fetch API](https://developer.mozilla.org/docs/Web/API/Fetch_API) を使用して実装されており、同一生成元ポリシーの適用を含め、その制限に従います。

クライアントのベース アドレスは、生成元のサーバーのアドレスに設定されます。 <xref:System.Net.Http.HttpClient> インスタンスは [`@inject`](xref:mvc/views/razor#inject) ディレクティブを使用して挿入します。

```razor
@using System.Net.Http
@inject HttpClient Http
```

以降の例では、Todo Web API によって、(CRUD) 操作の作成、読み取り、更新、および削除が処理されます。 これらの例は、以下を格納する `TodoItem` クラスに基づいています。

* ID (`Id`、`long`):項目の一意の ID。
* 名前 (`Name`、`string`):項目の名前。
* 状態 (`IsComplete`、`bool`):Todo 項目が終了したかどうかを示します。

```csharp
private class TodoItem
{
    public long Id { get; set; }
    public string Name { get; set; }
    public bool IsComplete { get; set; }
}
```

JSON のヘルパー メソッドは、要求を URI (次の例では Web API) に送信し、応答を処理します。

* <xref:System.Net.Http.Json.HttpClientJsonExtensions.GetFromJsonAsync%2A>:HTTP GET 要求を送信し、JSON 応答本文を解析してオブジェクトを作成します。

  次のコードでは、コンポーネントによって `todoItems` が表示されます。 `GetTodoItems` メソッドは、コンポーネントのレンダリングが終了したとき ([`OnInitializedAsync`](xref:blazor/components/lifecycle#component-initialization-methods)) にトリガーされます。 完全な例についてはサンプル アプリを参照してください。

  ```razor
  @using System.Net.Http
  @inject HttpClient Http

  @code {
      private TodoItem[] todoItems;

      protected override async Task OnInitializedAsync() => 
          todoItems = await Http.GetFromJsonAsync<TodoItem[]>("api/TodoItems");
  }
  ```

* <xref:System.Net.Http.Json.HttpClientJsonExtensions.PostAsJsonAsync%2A>:JSON でエンコードされたコンテンツを含め、HTTP POST 要求を送信し、JSON 応答本文を解析してオブジェクトを作成します。

  次のコードでは、コンポーネントのバインドされた要素によって `newItemName` が提供されています。 `AddItem` メソッドは、`<button>` 要素を選択することにでトリガーされます。 完全な例についてはサンプル アプリを参照してください。

  ```razor
  @using System.Net.Http
  @inject HttpClient Http

  <input @bind="newItemName" placeholder="New Todo Item" />
  <button @onclick="@AddItem">Add</button>

  @code {
      private string newItemName;

      private async Task AddItem()
      {
          var addItem = new TodoItem { Name = newItemName, IsComplete = false };
          await Http.PostAsJsonAsync("api/TodoItems", addItem);
      }
  }
  ```
  
  <xref:System.Net.Http.Json.HttpClientJsonExtensions.PostAsJsonAsync%2A> を呼び出すと、<xref:System.Net.Http.HttpResponseMessage> が返されます。 応答メッセージから JSON コンテンツを逆シリアル化するには、`ReadFromJsonAsync<T>` 拡張メソッドを使用します。
  
  ```csharp
  var content = response.Content.ReadFromJsonAsync<WeatherForecast>();
  ```

* <xref:System.Net.Http.Json.HttpClientJsonExtensions.PutAsJsonAsync%2A>:JSON でエンコードされたコンテンツを含め、HTTP PUT 要求を送信します。

  次のコードでは、`Name` および `IsCompleted` の `editItem` 値が、コンポーネントのバインドされた要素によって提供されています。 項目の `Id` は、UI の別の部分で項目が選択され、`EditItem` が呼び出されたときに設定されます。 `SaveItem` メソッドは、Save `<button>` 要素を選択することでトリガーされます。 完全な例についてはサンプル アプリを参照してください。

  ```razor
  @using System.Net.Http
  @inject HttpClient Http

  <input type="checkbox" @bind="editItem.IsComplete" />
  <input @bind="editItem.Name" />
  <button @onclick="@SaveItem">Save</button>

  @code {
      private TodoItem editItem = new TodoItem();

      private void EditItem(long id)
      {
          editItem = todoItems.Single(i => i.Id == id);
      }

      private async Task SaveItem() =>
          await Http.PutAsJsonAsync($"api/TodoItems/{editItem.Id}, editItem);
  }
  ```
  
  <xref:System.Net.Http.Json.HttpClientJsonExtensions.PutAsJsonAsync%2A> を呼び出すと、<xref:System.Net.Http.HttpResponseMessage> が返されます。 応答メッセージから JSON コンテンツを逆シリアル化するには、<xref:System.Net.Http.Json.HttpContentJsonExtensions.ReadFromJsonAsync%2A> 拡張メソッドを使用します。
  
  ```csharp
  var content = response.Content.ReadFromJsonAsync<WeatherForecast>();
  ```

<xref:System.Net.Http> には、HTTP 要求を送信し、HTTP 応答を受信するための追加の拡張メソッドが含まれています。 <xref:System.Net.Http.HttpClient.DeleteAsync%2A?displayProperty=nameWithType> は、HTTP DELETE 要求を Web API に送信するために使用します。

次のコードでは、Delete `<button>` 要素で `DeleteItem` メソッドを呼び出しています。 バインドされた `<input>` 要素では、削除する項目の `id` が提供されます。 完全な例についてはサンプル アプリを参照してください。

```razor
@using System.Net.Http
@inject HttpClient Http

<input @bind="id" />
<button @onclick="@DeleteItem">Delete</button>

@code {
    private long id;

    private async Task DeleteItem() =>
        await Http.DeleteAsync($"api/TodoItems/{id}");
}
```

## <a name="named-httpclient-with-ihttpclientfactory"></a>IHttpClientFactory を使用する名前付き HttpClient

<xref:System.Net.Http.IHttpClientFactory> サービスおよび名前付き <xref:System.Net.Http.HttpClient> の構成がサポートされています。

プロジェクト ファイルで [`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) NuGet パッケージを参照します。

`Program.Main` (`Program.cs`):

```csharp
builder.Services.AddHttpClient("ServerAPI", client => 
    client.BaseAddress = new Uri(builder.HostEnvironment.BaseAddress));
```

`FetchData` コンポーネント (`Pages/FetchData.razor`):

```razor
@inject IHttpClientFactory ClientFactory

...

@code {
    private WeatherForecast[] forecasts;

    protected override async Task OnInitializedAsync()
    {
        var client = ClientFactory.CreateClient("ServerAPI");

        forecasts = await client.GetFromJsonAsync<WeatherForecast[]>(
            "WeatherForecast");
    }
}
```

## <a name="typed-httpclient"></a>型指定された HttpClient

型指定された <xref:System.Net.Http.HttpClient> では、1 つ以上のアプリの <xref:System.Net.Http.HttpClient> インスタンス (既定または名前付き) を使用して、1 つ以上の Web API エンドポイントからデータを返します。

`WeatherForecastClient.cs`:

```csharp
using System.Net.Http;
using System.Net.Http.Json;
using System.Threading.Tasks;

public class WeatherForecastHttpClient
{
    private readonly HttpClient http;

    public WeatherForecastHttpClient(HttpClient http)
    {
        this.http = http;
    }

    public async Task<WeatherForecast[]> GetForecastAsync()
    {
        var forecasts = new WeatherForecast[0];

        try
        {
            forecasts = await http.GetFromJsonAsync<WeatherForecast[]>(
                "WeatherForecast");
        }
        catch
        {
            ...
        }

        return forecasts;
    }
}
```

`Program.Main` (`Program.cs`):

```csharp
builder.Services.AddHttpClient<WeatherForecastHttpClient>(client => 
    client.BaseAddress = new Uri(builder.HostEnvironment.BaseAddress));
```

コンポーネントによって型指定された <xref:System.Net.Http.HttpClient> が挿入され、Web API が呼び出されます。

`FetchData` コンポーネント (`Pages/FetchData.razor`):

```razor
@inject WeatherForecastHttpClient Http

...

@code {
    private WeatherForecast[] forecasts;

    protected override async Task OnInitializedAsync()
    {
        forecasts = await Http.GetForecastAsync();
    }
}
```

## <a name="httpclient-and-httprequestmessage-with-fetch-api-request-options"></a>`HttpClient` および `HttpRequestMessage` と Fetch API 要求オプション

Blazor WebAssembly アプリで WebAssembly を実行するときに、[`HttpClient`](xref:fundamentals/http-requests) ([API ドキュメント](xref:System.Net.Http.HttpClient)) および <xref:System.Net.Http.HttpRequestMessage> を使用して要求をカスタマイズできます。 たとえば、HTTP メソッドや要求ヘッダーを指定できます。 次のコンポーネントでは、サーバー上の To Do List API エンドポイントに `POST` 要求を行い、応答本文を表示します。

```razor
@page "/todorequest"
@using System.Net.Http
@using System.Net.Http.Headers
@using System.Net.Http.Json
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@inject HttpClient Http
@inject IAccessTokenProvider TokenProvider

<h1>ToDo Request</h1>

<button @onclick="PostRequest">Submit POST request</button>

<p>Response body returned by the server:</p>

<p>@responseBody</p>

@code {
    private string responseBody;

    private async Task PostRequest()
    {
        var requestMessage = new HttpRequestMessage()
        {
            Method = new HttpMethod("POST"),
            RequestUri = new Uri("https://localhost:10000/api/TodoItems"),
            Content =
                JsonContent.Create(new TodoItem
                {
                    Name = "My New Todo Item",
                    IsComplete = false
                })
        };

        var tokenResult = await TokenProvider.RequestAccessToken();

        if (tokenResult.TryGetToken(out var token))
        {
            requestMessage.Headers.Authorization =
                new AuthenticationHeaderValue("Bearer", token.Value);

            requestMessage.Content.Headers.TryAddWithoutValidation(
                "x-custom-header", "value");

            var response = await Http.SendAsync(requestMessage);
            var responseStatusCode = response.StatusCode;

            responseBody = await response.Content.ReadAsStringAsync();
        }
    }

    public class TodoItem
    {
        public long Id { get; set; }
        public string Name { get; set; }
        public bool IsComplete { get; set; }
    }
}
```

.NET WebAssembly の <xref:System.Net.Http.HttpClient> の実装には、[WindowOrWorkerGlobalScope.fetch()](https://developer.mozilla.org/docs/Web/API/WindowOrWorkerGlobalScope/fetch) が使用されます。 フェッチを使用すると、いくつかの[要求固有のオプション](https://developer.mozilla.org/docs/Web/API/WindowOrWorkerGlobalScope/fetch#Parameters)を構成できます。 

HTTP フェッチ要求オプションは、次の表に示す <xref:System.Net.Http.HttpRequestMessage> 拡張メソッドを使用して構成できます。

| 拡張メソッド | フェッチ要求プロパティ |
| --- | --- |
| <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestCredentials%2A> | [`credentials`](https://developer.mozilla.org/docs/Web/API/Request/credentials) |
| <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestCache%2A> | [`cache`](https://developer.mozilla.org/docs/Web/API/Request/cache) |
| <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestMode%2A> | [`mode`](https://developer.mozilla.org/docs/Web/API/Request/mode) |
| <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestIntegrity%2A> | [`integrity`](https://developer.mozilla.org/docs/Web/API/Request/integrity) |

より汎用的な <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestOption%2A> 拡張メソッドを使用してその他のオプションを設定できます。
 
通常、HTTP 応答は、応答コンテンツに対する同期読み取りのサポートを有効にするために Blazor WebAssembly アプリでバッファリングされます。 応答ストリーミングのサポートを有効にするには、要求に対して <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserResponseStreamingEnabled%2A> 拡張メソッドを使用します。

クロスオリジン要求に資格情報を含めるには、<xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestCredentials%2A> 拡張メソッドを使用します。

```csharp
requestMessage.SetBrowserRequestCredentials(BrowserRequestCredentials.Include);
```

Fetch API オプションの詳細については、[MDN の Web ドキュメント、WindowOrWorkerGlobalScope.fetch():Parameters](https://developer.mozilla.org/docs/Web/API/WindowOrWorkerGlobalScope/fetch#Parameters) を参照してください。

## <a name="handle-errors"></a>エラーの処理

Web API の操作中にエラーが発生した場合は、開発者コードで処理できます。 たとえば、<xref:System.Net.Http.Json.HttpClientJsonExtensions.GetFromJsonAsync%2A> では、`Content-Type` が `application/json` のサーバー API からの JSON 応答が想定されています。 応答が JSON 形式ではない場合、コンテンツ検証により <xref:System.NotSupportedException> がスローされます。

次の例では、天気予報データ要求の URI エンドポイントのスペルが間違っています。 URI を `WeatherForecast` にする必要がありますが、呼び出しでは `WeatherForcast` と表示されています ("e" がありません)。

<xref:System.Net.Http.Json.HttpClientJsonExtensions.GetFromJsonAsync%2A> の呼び出しでは、JSON が返されることが想定されていますが、`Content-Type` が `text/html` のサーバー上の未処理の例外に対して、サーバーから HTML が返されます。 パスが見つからず、ミドルウェアから要求のページまたはビューを提供できないため、サーバー上で未処理の例外が発生します。

クライアント上の <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> では、応答コンテンツが非 JSON であると検証されると <xref:System.NotSupportedException> がスローされます。 例外は `catch` ブロックでキャッチされます。ここでカスタム ロジックを使用してエラーをログに記録したり、わかりやすいエラー メッセージをユーザーに表示したりすることができます。

```csharp
protected override async Task OnInitializedAsync()
{
    try
    {
        forecasts = await Http.GetFromJsonAsync<WeatherForecast[]>(
            "WeatherForcast");
    }
    catch (NotSupportedException exception)
    {
        ...
    }
}
```

> [!NOTE]
> 前の例は、デモンストレーションを目的としています。 エンドポイントが存在しない場合や、サーバー上でハンドルされない例外が発生した場合でも、JSON を返すように Web API サーバー アプリを構成できます。

詳細については、「<xref:blazor/fundamentals/handle-errors>」を参照してください。

## <a name="cross-origin-resource-sharing-cors"></a>クロスオリジン リソース共有 (CORS)

Web ページでは、ブラウザーのセキュリティにより、その Web ページを提供したドメインと異なるドメインに対して要求を行うことはできません。 この制限は、*同一オリジン ポリシー* と呼ばれます。 同一オリジン ポリシーにより、悪意のあるサイトが別のサイトから機密データを読み取れないようになっています。 オリジンが異なるエンドポイントへの要求をブラウザーから行うには、*エンドポイント* で [クロス オリジン リソース共有 (CORS)](https://www.w3.org/TR/cors/) を有効にする必要があります。

[Blazor WebAssembly サンプル アプリ (BlazorWebAssemblySample)](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) では、呼び出し Web API コンポーネント (`Pages/CallWebAPI.razor`) で CORS を使用する方法が示されています。

Blazor アプリにおけるセキュリティ保護された要求での CORS の詳細については、「<xref:blazor/security/webassembly/additional-scenarios#cross-origin-resource-sharing-cors>」を参照してください。

ASP.NET Core アプリでの CORS に関する一般的な情報については、「<xref:security/cors>」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:blazor/security/webassembly/additional-scenarios>:<xref:System.Net.Http.HttpClient> を使用してセキュリティで保護された Web API 要求を作成する方法について説明します。
* <xref:fundamentals/http-requests>
* <xref:security/enforcing-ssl>
::: moniker range=">= aspnetcore-5.0"
* [Kestrel HTTPS エンドポイントの構成](xref:fundamentals/servers/kestrel/endpoints)
::: moniker-end
::: moniker range="< aspnetcore-5.0"
* [Kestrel HTTPS エンドポイントの構成](xref:fundamentals/servers/kestrel#endpoint-configuration)
::: moniker-end
* [W3C におけるクロスオリジン リソース共有 (CORS)](https://www.w3.org/TR/cors/)
