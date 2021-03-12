---
title: ASP.NET Core プロジェクトでのユーザーデータの追加、ダウンロード、および削除 Identity
author: rick-anderson
description: ASP.NET Core プロジェクト内のにカスタムユーザーデータを追加する方法について説明 Identity します。 GDPR ごとのデータを削除します。
ms.author: riande
ms.date: 03/26/2020
ms.custom: mvc, seodec18
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
uid: security/authentication/add-user-data
ms.openlocfilehash: 2d921a0c72fb7c03cd88966077e2d33e4b19ffa1
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102585917"
---
# <a name="add-download-and-delete-custom-user-data-to-identity-in-an-aspnet-core-project"></a>カスタムユーザーデータを Identity ASP.NET Core プロジェクトに追加、ダウンロード、および削除する

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

この記事では、次の方法について説明します。

* カスタムユーザーデータを ASP.NET Core web アプリに追加します。
* カスタムユーザーデータモデルを属性でマークして、 <xref:Microsoft.AspNetCore.Identity.PersonalDataAttribute> 自動的にダウンロードおよび削除できるようにします。 データをダウンロードして削除できるようにすると、 [GDPR](xref:security/gdpr) の要件を満たすことができます。

プロジェクトサンプルは、ページ web アプリから作成され Razor ますが、手順は ASP.NET CORE MVC web アプリの場合と似ています。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/add-user-data)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="prerequisites"></a>必須コンポーネント

::: moniker range=">= aspnetcore-3.0"

[!INCLUDE [](~/includes/3.0-SDK.md)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!INCLUDE [](~/includes/2.2-SDK.md)]

::: moniker-end

## <a name="create-a-razor-web-app"></a>Web アプリを作成する Razor

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

::: moniker range=">= aspnetcore-3.0"

* Visual Studio の **[ファイル]** メニューから、**[新規作成]** > **[プロジェクト]** の順に選択します。 [ダウンロードするサンプル](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/authentication/add-user-data)コードの名前空間と一致させる場合は、プロジェクトに **WebApp1** という名前を付けます。
* **ASP.NET Core Web アプリケーション** の選択- > **OK**
* ドロップダウンで **ASP.NET Core 3.0** を選択します。
* **Web アプリケーション** の選択- > **OK**
* プロジェクトをビルドして実行します。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

* Visual Studio の **[ファイル]** メニューから、**[新規作成]** > **[プロジェクト]** の順に選択します。 [ダウンロードするサンプル](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/authentication/add-user-data)コードの名前空間と一致させる場合は、プロジェクトに **WebApp1** という名前を付けます。
* **ASP.NET Core Web アプリケーション** の選択- > **OK**
* ドロップダウンで **ASP.NET Core 2.2** を選択します。
* **Web アプリケーション** の選択- > **OK**
* プロジェクトをビルドして実行します。

::: moniker-end


# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

```dotnetcli
dotnet new webapp -o WebApp1
```

---

## <a name="run-the-identity-scaffolder"></a>Scaffolder を実行します。 Identity

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* **ソリューションエクスプローラー** で、プロジェクトを右クリックし、[ **Add**  >  **New スキャフォールディング Item**] > ます。
* [**スキャフォールディングの追加**] ダイアログボックスの左ペインで、[追加] を選択し **Identity**  >  ます。
* [**追加 Identity** ] ダイアログで、次のオプションを選択します。
  * 既存のレイアウト _Layout ファイルを選択し  *ます。*
  * 上書きする以下のファイルを選択してください:
    * **アカウント/登録**
    * **アカウント/管理/インデックス**
  * **+** 新しい **データコンテキストクラス** を作成するには、このボタンをクリックします。 型 (プロジェクトの名前が **WebApp1** の場合は **WebApp1Context** ) をそのまま使用します。
  * **+** 新しい **ユーザークラス** を作成するには、このボタンを選択します。 型を受け入れます (プロジェクトの名前が **WebApp1** の場合は **WebApp1User** ) >**追加**] を使用します。
* **[追加]** を選択します。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

ASP.NET Core scaffolder をまだインストールしていない場合は、ここでインストールします。

```dotnetcli
dotnet tool install -g dotnet-aspnet-codegenerator
```

[VisualStudio](https://www.nuget.org/packages/Microsoft.VisualStudio.Web.CodeGeneration.Design/)にパッケージ参照を追加し、プロジェクト (.csproj) ファイルに追加してください。 プロジェクトディレクトリで次のコマンドを実行します。

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet restore
```

次のコマンドを実行して、scaffolder オプションの一覧を表示し Identity ます。

```dotnetcli
dotnet aspnet-codegenerator identity -h
```

プロジェクトフォルダーで、scaffolder を実行し Identity ます。

```dotnetcli
dotnet aspnet-codegenerator identity -u WebApp1User -fi Account.Register;Account.Manage.Index
```

---

[移行、UseAuthentication、および layout](xref:security/authentication/scaffold-identity#efm)の指示に従って、次の手順を実行します。

* 移行を作成し、データベースを更新します。
* `UseAuthentication` を `Startup.Configure` に追加します。
* `<partial name="_LoginPartial" />`レイアウトファイルにを追加します。
* アプリをテストします。
  * ユーザーを登録する
  * [ **ログアウト** ] リンクの横にある新しいユーザー名を選択します。 場合によっては、ウィンドウを展開するか、ナビゲーションバーのアイコンを選択して、ユーザー名とその他のリンクを表示する必要があります。
  * [ **Personal Data** ] タブを選択します。
  * [ **ダウンロード** ] ボタンを選択し、ファイルの *PersonalData.js* を調べます。
  * [ **削除** ] ボタンをテストします。これにより、ログオンしているユーザーが削除されます。

## <a name="add-custom-user-data-to-the-identity-db"></a>DB にカスタムユーザーデータを追加する Identity

`IdentityUser`カスタムプロパティを使用して、派生クラスを更新します。 プロジェクトに WebApp1 という名前を付けた場合、ファイルの名前は *Areas/ Identity /Data/WebApp1User.cs* になります。 次のコードを使用して、ファイルを更新します。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Data/WebApp1User.cs)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Data/WebApp1User.cs)]

::: moniker-end

プロパティ [は、次のよう](/dotnet/api/microsoft.aspnetcore.identity.personaldataattribute) になります。

* *区分/ Identity /Pages/Account/Manage/DeletePersonalData.cshtml* ページがを呼び出したときに削除され Razor `UserManager.Delete` ます。
* ダウンロードされたデータには、[区分]、[ページ]、 *および [ Identity Manage/Download] の各* ページで含まれ Razor ます。

### <a name="update-the-accountmanageindexcshtml-page"></a>Account/Manage/Index. cshtml ページを更新する

次の `InputModel` 強調表示されたコードを使用して、 *区分/ Identity /Pages/Account/Manage/Index.cshtml.cs* のを更新します。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Pages/Account/Manage/Index.cshtml.cs?name=snippet&highlight=24-32,48-49,96-104,106)]

次の強調表示されているマークアップを使用して、 *区分//////////また Identity* は更新プログラムを更新します。

[!code-cshtml[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Pages/Account/Manage/Index.cshtml?highlight=18-25)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Pages/Account/Manage/Index.cshtml.cs?name=snippet&highlight=28-36,63-64,98-106,119)]

次の強調表示されているマークアップを使用して、 *区分//////////また Identity* は更新プログラムを更新します。

[!code-cshtml[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Pages/Account/Manage/Index.cshtml?highlight=35-42)]

::: moniker-end

### <a name="update-the-accountregistercshtml-page"></a>Account/Register. cshtml ページを更新する

次の `InputModel` 強調表示されたコードを使用して、 *区分/ Identity /Pages/Account/Register.cshtml.cs* のを更新します。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Pages/Account/Register.cshtml.cs?name=snippet&highlight=30-38,70-71)]

次の強調表示されたマークアップを使用して、 *区分//[///////////////また Identity* はの

[!code-cshtml[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Pages/Account/Register.cshtml?highlight=16-25)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Pages/Account/Register.cshtml.cs?name=snippet&highlight=28-36,67,66)]

次の強調表示されたマークアップを使用して、 *区分//[///////////////また Identity* はの

[!code-cshtml[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Pages/Account/Register.cshtml?highlight=16-25)]

::: moniker-end


プロジェクトをビルドします。

### <a name="add-a-migration-for-the-custom-user-data"></a>カスタムユーザーデータの移行を追加する

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Visual Studio **パッケージマネージャーコンソール** で次のようにします。

```powershell
Add-Migration CustomUserData
Update-Database
```

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

```dotnetcli
dotnet ef migrations add CustomUserData
dotnet ef database update
```

---

## <a name="test-create-view-download-delete-custom-user-data"></a>テストの作成、表示、ダウンロード、カスタムユーザーデータの削除

アプリをテストします。

* 新しいユーザーを登録します。
* ページ上のカスタムユーザーデータを表示し `/Identity/Account/Manage` ます。
* ページからユーザーの個人データをダウンロードして表示し `/Identity/Account/Manage/PersonalData` ます。

## <a name="add-claims-to-identity-using-iuserclaimsprincipalfactoryapplicationuser"></a>IdentityIUserClaimsPrincipalFactory を使用してにクレームを追加する<ApplicationUser>

> [!NOTE]
> このセクションは、前のチュートリアルの拡張機能ではありません。 チュートリアルを使用してビルドされたアプリに次の手順を適用するには、 [GitHub の問題](https://github.com/dotnet/AspNetCore.Docs/issues/18797)を参照してください。

インターフェイスを使用して、追加の要求をに追加でき ASP.NET Core Identity `IUserClaimsPrincipalFactory<T>` ます。 このクラスは、メソッドでアプリに追加でき `Startup.ConfigureServices` ます。 次のように、クラスのカスタム実装を追加します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

    services.AddScoped<IUserClaimsPrincipalFactory<ApplicationUser>, 
        AdditionalUserClaimsPrincipalFactory>();
```

デモコードでは、クラスを使用し `ApplicationUser` ます。 このクラス `IsAdmin` は、追加の要求を追加するために使用されるプロパティを追加します。

```csharp
public class ApplicationUser : IdentityUser
{
    public bool IsAdmin { get; set; }
}
```

`AdditionalUserClaimsPrincipalFactory` は、`UserClaimsPrincipalFactory` インターフェイスを実装します。 新しいロールクレームがに追加され `ClaimsPrincipal` ます。

```csharp
public class AdditionalUserClaimsPrincipalFactory 
        : UserClaimsPrincipalFactory<ApplicationUser, IdentityRole>
{
    public AdditionalUserClaimsPrincipalFactory( 
        UserManager<ApplicationUser> userManager,
        RoleManager<IdentityRole> roleManager, 
        IOptions<IdentityOptions> optionsAccessor) 
        : base(userManager, roleManager, optionsAccessor)
    {}

    public async override Task<ClaimsPrincipal> CreateAsync(ApplicationUser user)
    {
        var principal = await base.CreateAsync(user);
        var identity = (ClaimsIdentity)principal.Identity;

        var claims = new List<Claim>();
        if (user.IsAdmin)
        {
            claims.Add(new Claim(JwtClaimTypes.Role, "admin"));
        }
        else
        {
            claims.Add(new Claim(JwtClaimTypes.Role, "user"));
        }

        identity.AddClaims(claims);
        return principal;
    }
}
```

その後、追加の要求をアプリで使用できます。 ページでは、 Razor `IAuthorizationService` インスタンスを使用して要求値にアクセスできます。

```cshtml
@using Microsoft.AspNetCore.Authorization
@inject IAuthorizationService AuthorizationService

@if ((await AuthorizationService.AuthorizeAsync(User, "IsAdmin")).Succeeded)
{
    <ul class="mr-auto navbar-nav">
        <li class="nav-item">
            <a class="nav-link" asp-controller="Admin" asp-action="Index">ADMIN</a>
        </li>
    </ul>
}
```
