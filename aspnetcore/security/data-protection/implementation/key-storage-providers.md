---
title: ASP.NET Core のキー記憶域プロバイダー
author: rick-anderson
description: ASP.NET Core の主要な記憶域プロバイダーと、キーの格納場所を構成する方法について説明します。
ms.author: riande
ms.date: 12/05/2019
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
uid: security/data-protection/implementation/key-storage-providers
ms.openlocfilehash: 137cdabc12b7cd01b82f7fe9921c17a5be957eb7
ms.sourcegitcommit: acfe51c35497a204f75c2a61125c9408c04493e6
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102605530"
---
# <a name="key-storage-providers-in-aspnet-core"></a>ASP.NET Core のキー記憶域プロバイダー

データ保護システムでは、暗号化キーの保存先を決定するために、 [既定で検出メカニズム](xref:security/data-protection/configuration/default-settings) が使用されます。 開発者は、既定の検出メカニズムを上書きし、場所を手動で指定できます。

> [!WARNING]
> 明示的なキーの保存場所を指定した場合、データ保護システムは解除の既定のキー暗号化メカニズムを使用するので、保存時にキーが暗号化されなくなります。 運用環境のデプロイで [は、明示的なキー暗号化メカニズム](xref:security/data-protection/implementation/key-encryption-at-rest) を追加で指定することをお勧めします。

## <a name="file-system"></a>ファイル システム

ファイルシステムベースのキーリポジトリを構成するには、次に示すように、 [Persistkeystofilesystem](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.persistkeystofilesystem) 構成ルーチンを呼び出します。 キーを格納するリポジトリを指す [DirectoryInfo](/dotnet/api/system.io.directoryinfo) を指定します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToFileSystem(new DirectoryInfo(@"c:\temp-keys\"));
}
```

は、 `DirectoryInfo` ローカルコンピューター上のディレクトリを指すことも、ネットワーク共有上のフォルダーを指すこともできます。 ローカルコンピューター上のディレクトリを指している場合 (つまり、このリポジトリを使用するためにアクセスが必要なのはローカルコンピューター上のアプリのみです)、windows [DPAPI](xref:security/data-protection/implementation/key-encryption-at-rest) (windows) を使用して保存時のキーを暗号化することを検討してください。 それ以外の場合は、 [x.509 証明書](xref:security/data-protection/implementation/key-encryption-at-rest) を使用して保存時のキーを暗号化することを検討してください。

## <a name="azure-storage"></a>Azure Storage

[AspNetCore](https://www.nuget.org/packages/Azure.Extensions.AspNetCore.DataProtection.Blobs)パッケージを使用すると、Azure Blob Storage にデータ保護キーを格納できます。 キーは、web アプリの複数のインスタンス間で共有できます。 アプリは、認証 cookie s または CSRF 保護を複数のサーバーで共有できます。

Azure Blob Storage プロバイダーを構成するには、 [Persistkeystoazureblobstorage](/dotnet/api/microsoft.aspnetcore.dataprotection.azuredataprotectionbuilderextensions.persistkeystoazureblobstorage) オーバーロードのいずれかを呼び出します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToAzureBlobStorage(new Uri("<blob URI including SAS token>"));
}
```

Web アプリが Azure サービスとして実行されている場合は、接続文字列を使用して azure storage に対する認証を行うことができます。 [blob](/dotnet/api/azure.storage.blobs.blobcontainerclient)を使用します。

```csharp
string connectionString = "<connection_string>";
string containerName = "my-key-container";
BlobContainerClient container = new BlobContainerClient(connectionString, containerName);

// optional - provision the container automatically
await container.CreateIfNotExistsAsync();

services.AddDataProtection()
    .PersistKeysToAzureBlobStorage(container, "keys.xml");
```

> [!NOTE]
> ストレージアカウントへの接続文字列は、Azure Portal の [アクセスキー] セクションで確認するか、次の CLI コマンドを実行して確認できます。 
> ```bash
> az storage account show-connection-string --name <account_name> --resource-group <resource_group>
> ```

## <a name="redis"></a>Redis

::: moniker range=">= aspnetcore-2.2"

[StackExchangeRedis](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.StackExchangeRedis/)パッケージは、Redis cache にデータ保護キーを格納することを許可します。 キーは、web アプリの複数のインスタンス間で共有できます。 アプリは、認証 cookie s または CSRF 保護を複数のサーバーで共有できます。

::: moniker-end

::: moniker range="< aspnetcore-2.2"

[AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.Redis/)パッケージを使用すると、redis cache にデータ保護キーを格納できます。 キーは、web アプリの複数のインスタンス間で共有できます。 アプリは、認証 cookie s または CSRF 保護を複数のサーバーで共有できます。

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

Redis でを構成するには、 [PersistKeysToStackExchangeRedis](/dotnet/api/microsoft.aspnetcore.dataprotection.stackexchangeredisdataprotectionbuilderextensions.persistkeystostackexchangeredis) オーバーロードのいずれかを呼び出します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var redis = ConnectionMultiplexer.Connect("<URI>");
    services.AddDataProtection()
        .PersistKeysToStackExchangeRedis(redis, "DataProtection-Keys");
}
```

::: moniker-end

::: moniker range="< aspnetcore-2.2"

Redis でを構成するには、 [PersistKeysToRedis](/dotnet/api/microsoft.aspnetcore.dataprotection.redisdataprotectionbuilderextensions.persistkeystoredis) オーバーロードのいずれかを呼び出します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var redis = ConnectionMultiplexer.Connect("<URI>");
    services.AddDataProtection()
        .PersistKeysToRedis(redis, "DataProtection-Keys");
}
```

::: moniker-end

詳細については、次のトピックを参照してください。

* [StackExchange. Redis ConnectionMultiplexer](https://github.com/StackExchange/StackExchange.Redis/blob/main/docs/Basics.md)
* [Azure Redis Cache](/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache#connect-to-the-cache)
* [ASP.NET Core DataProtection のサンプル](https://github.com/dotnet/AspNetCore/tree/2.2.0/src/DataProtection/samples)

## <a name="registry"></a>レジストリ

**Windows の展開にのみ適用されます。**

場合によっては、アプリケーションにファイルシステムへの書き込みアクセス権がないことがあります。 アプリが仮想サービスアカウント ( *w3wp.exe* のアプリプール id など) として実行されているシナリオについて考えてみましょう。 このような場合、管理者は、サービスアカウント id によってアクセス可能なレジストリキーをプロビジョニングできます。 次に示すように、 [PersistKeysToRegistry](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.persistkeystoregistry) extension メソッドを呼び出します。 暗号化キーを格納する場所を指す [RegistryKey](/dotnet/api/microsoft.aspnetcore.dataprotection.repositories.registryxmlrepository.registrykey) を指定します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToRegistry(Registry.CurrentUser.OpenSubKey(@"SOFTWARE\Sample\keys", true));
}
```

> [!IMPORTANT]
> Rest でのキーの暗号化には [WINDOWS DPAPI](xref:security/data-protection/implementation/key-encryption-at-rest) を使用することをお勧めします。

::: moniker range=">= aspnetcore-2.2"

## <a name="entity-framework-core"></a>Entity Framework Core

[AspNetCore コア](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.EntityFrameworkCore/)パッケージは、Entity Framework Core を使用してデータベースにデータ保護キーを格納するためのメカニズムを提供します。 `Microsoft.AspNetCore.DataProtection.EntityFrameworkCore`NuGet パッケージは、プロジェクトファイルに追加する必要があります。これは、 [AspNetCore メタパッケージ](xref:fundamentals/metapackage-app)の一部ではありません。

このパッケージでは、web アプリの複数のインスタンス間でキーを共有できます。

EF Core プロバイダーを構成するには、 [Persistkeystodbcontext \<TContext> ](/dotnet/api/microsoft.aspnetcore.dataprotection.entityframeworkcoredataprotectionextensions.persistkeystodbcontext)メソッドを呼び出します。

[!code-csharp[Main](key-storage-providers/sample/Startup.cs?name=snippet&highlight=13-20)]

[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

ジェネリックパラメーターは、 `TContext` [dbcontext](/dotnet/api/microsoft.entityframeworkcore.dbcontext) から継承し、 [IDataProtectionKeyContext](/dotnet/api/microsoft.aspnetcore.dataprotection.entityframeworkcore.idataprotectionkeycontext)を実装する必要があります。

[!code-csharp[Main](key-storage-providers/sample/MyKeysContext.cs)]

`DataProtectionKeys` テーブルを作成します。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

**パッケージマネージャーコンソール**(PMC) ウィンドウで、次のコマンドを実行します。

```powershell
Add-Migration AddDataProtectionKeys -Context MyKeysContext
Update-Database -Context MyKeysContext
```

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

コマンドシェルで次のコマンドを実行します。

```dotnetcli
dotnet ef migrations add AddDataProtectionKeys --context MyKeysContext
dotnet ef database update --context MyKeysContext
```

---

`MyKeysContext` は、 `DbContext` 前のコードサンプルで定義されているです。 を別の名前で使用している場合は `DbContext` 、 `DbContext` の名前に置き換え `MyKeysContext` ます。

`DataProtectionKeys`クラス/エンティティは、次の表に示す構造を採用しています。

| プロパティ/フィールド | CLR 型 | SQL 型              |
| -------------- | -------- | --------------------- |
| `Id`           | `int`    | `int`、PK、 `IDENTITY(1,1)` 、null 以外   |
| `FriendlyName` | `string` | `nvarchar(MAX)`、null |
| `Xml`          | `string` | `nvarchar(MAX)`、null |

::: moniker-end

## <a name="custom-key-repository"></a>カスタムキーリポジトリ

インボックス機構が適切でない場合、開発者はカスタム [IXmlRepository](/dotnet/api/microsoft.aspnetcore.dataprotection.repositories.ixmlrepository)を提供することで、独自のキー永続化メカニズムを指定できます。