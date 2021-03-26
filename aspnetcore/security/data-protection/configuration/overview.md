---
title: ASP.NET Core データ保護の構成
author: rick-anderson
description: ASP.NET Core でデータ保護を構成する方法について説明します。
ms.author: riande
ms.custom: mvc
ms.date: 11/02/2020
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
uid: security/data-protection/configuration/overview
ms.openlocfilehash: 8c00deb49709bd2ec2a5824c841fa3600f99b204
ms.sourcegitcommit: 4bbc69f51c59bed1a96aa46f9f5dca2f2a2634cb
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/25/2021
ms.locfileid: "105554669"
---
# <a name="configure-aspnet-core-data-protection"></a>ASP.NET Core データ保護の構成

データ保護システムが初期化されると、運用環境に基づいて [既定の設定](xref:security/data-protection/configuration/default-settings) が適用されます。 これらの設定は、通常、1台のコンピューターで実行されるアプリに適しています。 開発者が既定の設定を変更する場合があります。

* アプリは複数のコンピューターに分散されています。
* コンプライアンス上の理由で。

これらのシナリオでは、データ保護システムに豊富な構成 API が用意されています。

> [!WARNING]
> 構成ファイルと同様に、データ保護キーリングは適切なアクセス許可を使用して保護する必要があります。 保存時のキーの暗号化は選択できますが、攻撃者によって新しいキーが作成されるのを防ぐことはできません。 その結果、アプリのセキュリティが影響を受けます。 データ保護を使用して構成されたストレージの場所は、構成ファイルを保護する場合と同様に、アプリ自体に限定されます。 たとえば、キーリングをディスクに保存する場合は、ファイルシステムのアクセス許可を使用します。 Web アプリが実行されている id のみが、そのディレクトリへの読み取り、書き込み、および作成のアクセス権を持っていることを確認します。 Azure Blob Storage を使用する場合は、web アプリのみが Blob ストア内の新しいエントリの読み取り、書き込み、または作成を行うことができます。
>
> 拡張メソッド [Adddataprotection](/dotnet/api/microsoft.extensions.dependencyinjection.dataprotectionservicecollectionextensions.adddataprotection) は [IDataProtectionBuilder](/dotnet/api/microsoft.aspnetcore.dataprotection.idataprotectionbuilder)を返します。 `IDataProtectionBuilder` データ保護オプションを構成するために連結できる拡張メソッドを公開します。

::: moniker range=">= aspnetcore-3.0"

この記事で使用するデータ保護拡張機能には、次の NuGet パッケージが必要です。

* [Azure.Extensions.AspNetCore.DataProtection.Blobs](https://www.nuget.org/packages/Azure.Extensions.AspNetCore.DataProtection.Blobs)
* [Azure.Extensions.AspNetCore.DataProtection.Keys](https://www.nuget.org/packages/Azure.Extensions.AspNetCore.DataProtection.Keys)

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

## <a name="protectkeyswithazurekeyvault"></a>ProtectKeysWithAzureKeyVault

CLI を使用して Azure にログインします。次に例を示します。

```azurecli
az login
``` 

[Azure Key Vault](https://azure.microsoft.com/services/key-vault/)にキーを格納するには、クラスで[ProtectKeysWithAzureKeyVault](/dotnet/api/microsoft.aspnetcore.dataprotection.azuredataprotectionbuilderextensions.protectkeyswithazurekeyvault)を使用してシステムを構成し `Startup` ます。 `blobUriWithSasToken` キーファイルを格納する必要がある完全な URI を指定します。 URI には、クエリ文字列パラメーターとして SAS トークンを含める必要があります。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToAzureBlobStorage(new Uri("<blobUriWithSasToken>"))
        .ProtectKeysWithAzureKeyVault(new Uri("<keyIdentifier>"), new DefaultAzureCredential());
}
```

キーリングのストレージの場所を設定します (たとえば、 [Persistkeystoazureblobstorage](/dotnet/api/microsoft.aspnetcore.dataprotection.azuredataprotectionbuilderextensions.persistkeystoazureblobstorage))。 を呼び出すと、 `ProtectKeysWithAzureKeyVault` キーリングストレージの場所を含む自動データ保護設定を無効にする [IXmlEncryptor](/dotnet/api/microsoft.aspnetcore.dataprotection.xmlencryption.ixmlencryptor) が実装されるため、場所を設定する必要があります。 前の例では、Azure Blob Storage を使用してキーリングを永続化しています。 詳細については、「 [キー記憶域プロバイダー: Azure Storage](xref:security/data-protection/implementation/key-storage-providers#azure-storage)」を参照してください。 キーリングを [Persistkeystofilesystem](xref:security/data-protection/implementation/key-storage-providers#file-system)でローカルに永続化することもできます。

はキーの `keyIdentifier` 暗号化に使用される key vault キー識別子です。 たとえば、内のという名前の key vault で作成されたキーには、 `dataprotection` `contosokeyvault` キー識別子があり `https://contosokeyvault.vault.azure.net/keys/dataprotection/` ます。 キーコンテナーへの **ラップ解除キー** と **ラップキー** のアクセス許可をアプリに提供します。

`ProtectKeysWithAzureKeyVault` オーバーライド

* [ProtectKeysWithAzureKeyVault (IDataProtectionBuilder, uri, tokencredential)](/dotnet/api/microsoft.aspnetcore.dataprotection.azuredataprotectionkeyvaultkeybuilderextensions.protectkeyswithazurekeyvault#Microsoft_AspNetCore_DataProtection_AzureDataProtectionKeyVaultKeyBuilderExtensions_ProtectKeysWithAzureKeyVault_Microsoft_AspNetCore_DataProtection_IDataProtectionBuilder_System_Uri_Azure_Core_TokenCredential_) では、keyidentifier Uri と tokencredential を使用して、データ保護システムが key vault を使用できるようにします。
* [ProtectKeysWithAzureKeyVault (IDataProtectionBuilder, string, IKeyEncryptionKeyResolver)](/dotnet/api/microsoft.aspnetcore.dataprotection.azuredataprotectionkeyvaultkeybuilderextensions.protectkeyswithazurekeyvault#Microsoft_AspNetCore_DataProtection_AzureDataProtectionKeyVaultKeyBuilderExtensions_ProtectKeysWithAzureKeyVault_Microsoft_AspNetCore_DataProtection_IDataProtectionBuilder_System_Uri_Azure_Core_TokenCredential_) では、keyidentifier 文字列と IKeyEncryptionKeyResolver を使用して、データ保護システムが key vault を使用できるようにします。

アプリが以前の Azure パッケージ (と) を使用していて、 [`Microsoft.AspNetCore.DataProtection.AzureStorage`](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.AzureStorage) [`Microsoft.AspNetCore.DataProtection.AzureKeyVault`](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.AzureKeyVault) Azure Key Vault と Azure Storage を組み合わせてキーを格納および保護している場合、 <xref:System.UriFormatException?displayProperty=nameWithType> キーストレージの blob が存在しないと、がスローされます。 Azure portal でアプリを実行する前に blob を手動で作成することも、次の手順を使用することもできます。

1. 最初の実行でのへの呼び出しを削除して、 `ProtectKeysWithAzureKeyVault` blob を適切に作成します。
1. 後続の実行では、への呼び出しを追加し `ProtectKeysWithAzureKeyVault` ます。

`ProtectKeysWithAzureKeyVault`最初の実行に対してを削除することをお勧めします。これは、適切なスキーマと値が設定された状態でファイルが作成されることを保証するためです。 

提供される API によって blob が存在しない場合に自動的に作成されるため、 [AspNetCore](https://www.nuget.org/packages/Azure.Extensions.AspNetCore.DataProtection.Blobs) および [AspNetCore](https://www.nuget.org/packages/Azure.Extensions.AspNetCore.DataProtection.Keys) パッケージにアップグレードすることをお勧めします。

```csharp
services.AddDataProtection()
    //This blob must already exist before the application is run
    .PersistKeysToAzureBlobStorage("<storage account connection string", "<key store container name>", "<key store blob name>")
    //Removing this line below for an initial run will ensure the file is created correctly
    .ProtectKeysWithAzureKeyVault(new Uri("<keyIdentifier>"), new DefaultAzureCredential());
```

::: moniker-end

## <a name="persistkeystofilesystem"></a>PersistKeysToFileSystem

*% Localappdata%* の既定の場所ではなく UNC 共有にキーを格納するには、 [Persistkeystofilesystem](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.persistkeystofilesystem)を使用してシステムを構成します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToFileSystem(new DirectoryInfo(@"\\server\share\directory\"));
}
```

> [!WARNING]
> キーの保存場所を変更すると、DPAPI が適切な暗号化メカニズムであるかどうかがわからないため、保存されているキーは自動的に暗号化されなくなります。

## <a name="protectkeyswith"></a>ProtectKeysWith\*

[ProtectKeysWith \* ](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions)構成 api のいずれかを呼び出すことにより、保存時のキーを保護するようにシステムを構成できます。 次の例では、UNC 共有にキーを格納し、特定の x.509 証明書を使用して保存中のキーを暗号化します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToFileSystem(new DirectoryInfo(@"\\server\share\directory\"))
        .ProtectKeysWithCertificate(Configuration["Thumbprint"]);
}
```

::: moniker range=">= aspnetcore-2.1"

ASP.NET Core 2.1 以降では、ファイルから読み込まれた証明書などの[ProtectKeysWithCertificate](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.protectkeyswithcertificate)に[X509Certificate2](/dotnet/api/system.security.cryptography.x509certificates.x509certificate2)を提供できます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToFileSystem(new DirectoryInfo(@"\\server\share\directory\"))
        .ProtectKeysWithCertificate(
            new X509Certificate2("certificate.pfx", Configuration["Thumbprint"]));
}
```

::: moniker-end

その他の例と組み込みのキー暗号化メカニズムの詳細については、「保存 [時のキーの暗号化](xref:security/data-protection/implementation/key-encryption-at-rest) 」を参照してください。

::: moniker range=">= aspnetcore-2.1"

## <a name="unprotectkeyswithanycertificate"></a>UnprotectKeysWithAnyCertificate

ASP.NET Core 2.1 以降では、 [UnprotectKeysWithAnyCertificate](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.unprotectkeyswithanycertificate)で[X509Certificate2](/dotnet/api/system.security.cryptography.x509certificates.x509certificate2)証明書の配列を使用して、証明書をローテーションし、保存時のキーを復号化することができます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToFileSystem(new DirectoryInfo(@"\\server\share\directory\"))
        .ProtectKeysWithCertificate(
            new X509Certificate2("certificate.pfx", Configuration["MyPasswordKey"));
        .UnprotectKeysWithAnyCertificate(
            new X509Certificate2("certificate_old_1.pfx", Configuration["MyPasswordKey_1"),
            new X509Certificate2("certificate_old_2.pfx", Configuration["MyPasswordKey_2"));
}
```

::: moniker-end

## <a name="setdefaultkeylifetime"></a>SetDefaultKeyLifetime

既定の90日ではなく、14日間のキー有効期間を使用するようにシステムを構成するには、 [Setdefaultkeylifetime](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.setdefaultkeylifetime)を使用します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .SetDefaultKeyLifetime(TimeSpan.FromDays(14));
}
```

## <a name="setapplicationname"></a>SetApplicationName

既定では、データ保護システムは、同じ物理キーリポジトリを共有している場合でも、 [コンテンツのルート](xref:fundamentals/index#content-root) パスに基づいて、アプリを相互に分離します。 これにより、アプリは互いの保護されたペイロードを認識できなくなります。

保護されたペイロードをアプリ間で共有するには:

* <xref:Microsoft.AspNetCore.DataProtection.DataProtectionBuilderExtensions.SetApplicationName*>各アプリで同じ値を使用してを構成します。
* アプリ全体で同じバージョンのデータ保護 API スタックを使用します。 アプリのプロジェクトファイルで、次の **いずれか** を実行します。
  * [Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)を介して、同じ共有フレームワークのバージョンを参照します。
  * 同じ [データ保護パッケージ](xref:security/data-protection/introduction#package-layout) のバージョンを参照します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .SetApplicationName("shared app name");
}
```

## <a name="disableautomatickeygeneration"></a>DisableAutomaticKeyGeneration

場合によっては、有効期限が近づいたときにキーを自動的にロールする (新しいキーを作成する) 必要はありません。 この例の1つは、プライマリとセカンダリの関係で設定されたアプリです。プライマリアプリのみがキー管理の問題を担い、セカンダリアプリはキーリングの読み取り専用ビューを持っているだけです。 次のようにシステムを構成することによって、キーリングを読み取り専用として扱うように、セカンダリアプリを構成できます <xref:Microsoft.AspNetCore.DataProtection.DataProtectionBuilderExtensions.DisableAutomaticKeyGeneration*> 。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .DisableAutomaticKeyGeneration();
}
```

## <a name="per-application-isolation"></a>アプリケーションごとの分離

データ保護システムが ASP.NET Core ホストによって提供されると、アプリが同じワーカープロセスアカウントで実行され、同じマスターキーマテリアルを使用している場合でも、アプリが相互に自動的に分離されます。 これは、System.web の要素の IsolateApps 修飾子と少し似てい `<machineKey>` ます。

分離メカニズムは、ローカルコンピューター上の各アプリを一意のテナントと見なすことによって機能します。したがって、 <xref:Microsoft.AspNetCore.DataProtection.IDataProtector> 特定のアプリのルートルートには、アプリ ID が識別子として自動的に含まれます。 アプリの一意の ID は、アプリの物理パスです。

* IIS でホストされているアプリの場合、一意の ID はアプリの IIS 物理パスです。 アプリが web ファーム環境に配置されている場合、IIS 環境が web ファーム内のすべてのコンピューターで同じように構成されていることを前提として、この値は安定しています。
* [Kestrel サーバー](xref:fundamentals/servers/index#kestrel)で実行されている自己ホスト型アプリの場合、一意の ID はディスク上のアプリへの物理パスです。

一意の識別子は、 &mdash; 個別のアプリとコンピューター自体の両方を再設定するように設計されています。

この分離メカニズムは、アプリが悪意のあるものではないことを前提としています。 悪意のあるアプリは、同じワーカープロセスアカウントで実行されている他のアプリに常に影響を与える可能性があります。 アプリが相互に信頼されていない共有ホスティング環境では、ホスティングプロバイダーはアプリ間の OS レベルの分離を保証するための手順を実行する必要があります。これには、アプリの基になるキーリポジトリの分離も含まれます。

データ保護システムが ASP.NET Core ホストによって提供されない場合 (具象型でインスタンス化する場合など `DataProtectionProvider` )、アプリの分離は既定で無効になります。 アプリの分離が無効になっている場合、同じキーマテリアルによってサポートされるすべてのアプリは、適切な [目的](xref:security/data-protection/consumer-apis/purpose-strings)で提供されている限り、ペイロードを共有できます。 この環境でアプリの分離を提供するには、構成オブジェクトで [Setapplicationname](#setapplicationname) メソッドを呼び出し、アプリごとに一意の名前を指定します。

## <a name="changing-algorithms-with-usecryptographicalgorithms"></a>UseCryptographicAlgorithms を使用したアルゴリズムの変更

データ保護スタックでは、新しく生成されたキーによって使用される既定のアルゴリズムを変更することができます。 これを行う最も簡単な方法は、構成コールバックから [UseCryptographicAlgorithms](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.usecryptographicalgorithms) を呼び出すことです。

::: moniker range=">= aspnetcore-2.0"

```csharp
services.AddDataProtection()
    .UseCryptographicAlgorithms(
        new AuthenticatedEncryptorConfiguration()
    {
        EncryptionAlgorithm = EncryptionAlgorithm.AES_256_CBC,
        ValidationAlgorithm = ValidationAlgorithm.HMACSHA256
    });
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
services.AddDataProtection()
    .UseCryptographicAlgorithms(
        new AuthenticatedEncryptionSettings()
    {
        EncryptionAlgorithm = EncryptionAlgorithm.AES_256_CBC,
        ValidationAlgorithm = ValidationAlgorithm.HMACSHA256
    });
```

::: moniker-end

既定の EncryptionAlgorithm は AES-256-CBC、既定の ValidationAlgorithm は HMACSHA256 です。 既定のポリシーは、 [コンピューター全体のポリシー](xref:security/data-protection/configuration/machine-wide-policy)を使用してシステム管理者が設定できますが、を明示的に呼び出すと、 `UseCryptographicAlgorithms` 既定のポリシーが上書きされます。

を呼び出す `UseCryptographicAlgorithms` と、定義済みの組み込みリストから目的のアルゴリズムを指定できます。 アルゴリズムの実装について心配する必要はありません。 上記のシナリオでは、Windows で実行されている場合、データ保護システムは AES の CNG 実装を使用しようとします。 それ以外の場合は [、管理された](/dotnet/api/system.security.cryptography.aes) system.object クラスにフォールバックします。

[UseCustomCryptographicAlgorithms](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.usecustomcryptographicalgorithms)の呼び出しを使用して、手動で実装を指定できます。

> [!TIP]
> アルゴリズムを変更しても、キーリング内の既存のキーには影響しません。 新たに生成されたキーにのみ影響します。

### <a name="specifying-custom-managed-algorithms"></a>カスタムマネージアルゴリズムの指定

::: moniker range=">= aspnetcore-2.0"

カスタムマネージアルゴリズムを指定するには、次の実装の種類を指す [Managed認証 Ated暗号化の構成](/dotnet/api/microsoft.aspnetcore.dataprotection.authenticatedencryption.configurationmodel.managedauthenticatedencryptorconfiguration) インスタンスを作成します。

```csharp
serviceCollection.AddDataProtection()
    .UseCustomCryptographicAlgorithms(
        new ManagedAuthenticatedEncryptorConfiguration()
    {
        // A type that subclasses SymmetricAlgorithm
        EncryptionAlgorithmType = typeof(Aes),

        // Specified in bits
        EncryptionAlgorithmKeySize = 256,

        // A type that subclasses KeyedHashAlgorithm
        ValidationAlgorithmType = typeof(HMACSHA256)
    });
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

カスタムマネージアルゴリズムを指定するには、実装の種類を指す [Managedauthenticatedencryptionsettings](/dotnet/api/microsoft.aspnetcore.dataprotection.authenticatedencryption.managedauthenticatedencryptionsettings) インスタンスを作成します。

```csharp
serviceCollection.AddDataProtection()
    .UseCustomCryptographicAlgorithms(
        new ManagedAuthenticatedEncryptionSettings()
    {
        // A type that subclasses SymmetricAlgorithm
        EncryptionAlgorithmType = typeof(Aes),

        // Specified in bits
        EncryptionAlgorithmKeySize = 256,

        // A type that subclasses KeyedHashAlgorithm
        ValidationAlgorithmType = typeof(HMACSHA256)
    });
```

::: moniker-end

一般に、 \* 型プロパティは [Symmetricalgorithm](/dotnet/api/system.security.cryptography.symmetricalgorithm) と [KeyedHashAlgorithm](/dotnet/api/system.security.cryptography.keyedhashalgorithm)のインスタンス化された (パブリックのパラメーター化されていない) 実装を指す必要がありますが、システムによって特別なケースによって、便宜上の値が使用さ `typeof(Aes)` れます。

> [!NOTE]
> SymmetricAlgorithm のキーの長さは、≥128ビットおよびブロックサイズ≥64ビットである必要があります。また、PKCS #7 パディングによる CBC モードの暗号化をサポートする必要があります。 KeyedHashAlgorithm のダイジェストサイズは >= 128 ビットである必要があり、ハッシュアルゴリズムのダイジェストの長さと同じ長さのキーをサポートする必要があります。 KeyedHashAlgorithm は、必ずしも HMAC である必要はありません。

### <a name="specifying-custom-windows-cng-algorithms"></a>カスタム Windows CNG アルゴリズムの指定

::: moniker range=">= aspnetcore-2.0"

HMAC 検証で CBC モード暗号化を使用してカスタム Windows CNG アルゴリズムを指定するには、アルゴリズム情報を含む [CngCbcAuthenticatedEncryptorConfiguration](/dotnet/api/microsoft.aspnetcore.dataprotection.authenticatedencryption.configurationmodel.cngcbcauthenticatedencryptorconfiguration) インスタンスを作成します。

```csharp
services.AddDataProtection()
    .UseCustomCryptographicAlgorithms(
        new CngCbcAuthenticatedEncryptorConfiguration()
    {
        // Passed to BCryptOpenAlgorithmProvider
        EncryptionAlgorithm = "AES",
        EncryptionAlgorithmProvider = null,

        // Specified in bits
        EncryptionAlgorithmKeySize = 256,

        // Passed to BCryptOpenAlgorithmProvider
        HashAlgorithm = "SHA256",
        HashAlgorithmProvider = null
    });
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

HMAC 検証で CBC モード暗号化を使用してカスタム Windows CNG アルゴリズムを指定するには、アルゴリズム情報を含む [CngCbcAuthenticatedEncryptionSettings](/dotnet/api/microsoft.aspnetcore.dataprotection.authenticatedencryption.cngcbcauthenticatedencryptionsettings) インスタンスを作成します。

```csharp
services.AddDataProtection()
    .UseCustomCryptographicAlgorithms(
        new CngCbcAuthenticatedEncryptionSettings()
    {
        // Passed to BCryptOpenAlgorithmProvider
        EncryptionAlgorithm = "AES",
        EncryptionAlgorithmProvider = null,

        // Specified in bits
        EncryptionAlgorithmKeySize = 256,

        // Passed to BCryptOpenAlgorithmProvider
        HashAlgorithm = "SHA256",
        HashAlgorithmProvider = null
    });
```

::: moniker-end

> [!NOTE]
> 対称ブロック暗号アルゴリズムのキーの長さは >= 128 ビット、ブロックサイズ >= 64 ビットである必要があります。また、PKCS #7 パディングによる CBC モードの暗号化をサポートしている必要があります。 ハッシュアルゴリズムのダイジェストサイズは >= 128 ビットである必要があり、BCRYPT \_ ALG \_ HANDLE \_ HMAC フラグフラグを使用して開くことがサポートされている必要があり \_ ます。 \*プロバイダーのプロパティを null に設定すると、指定したアルゴリズムの既定のプロバイダーを使用できます。 詳細については、 [BCryptOpenAlgorithmProvider](/windows/win32/api/bcrypt/nf-bcrypt-bcryptopenalgorithmprovider) のドキュメントを参照してください。

::: moniker range=">= aspnetcore-2.0"

検証で Galois/カウンタモードの暗号化を使用してカスタム Windows CNG アルゴリズムを指定するには、アルゴリズム情報を含む [CngGcmAuthenticatedEncryptorConfiguration](/dotnet/api/microsoft.aspnetcore.dataprotection.authenticatedencryption.configurationmodel.cnggcmauthenticatedencryptorconfiguration) インスタンスを作成します。

```csharp
services.AddDataProtection()
    .UseCustomCryptographicAlgorithms(
        new CngGcmAuthenticatedEncryptorConfiguration()
    {
        // Passed to BCryptOpenAlgorithmProvider
        EncryptionAlgorithm = "AES",
        EncryptionAlgorithmProvider = null,

        // Specified in bits
        EncryptionAlgorithmKeySize = 256
    });
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

検証で Galois/カウンタモードの暗号化を使用してカスタム Windows CNG アルゴリズムを指定するには、アルゴリズム情報を含む [CngGcmAuthenticatedEncryptionSettings](/dotnet/api/microsoft.aspnetcore.dataprotection.authenticatedencryption.cnggcmauthenticatedencryptionsettings) インスタンスを作成します。

```csharp
services.AddDataProtection()
    .UseCustomCryptographicAlgorithms(
        new CngGcmAuthenticatedEncryptionSettings()
    {
        // Passed to BCryptOpenAlgorithmProvider
        EncryptionAlgorithm = "AES",
        EncryptionAlgorithmProvider = null,

        // Specified in bits
        EncryptionAlgorithmKeySize = 256
    });
```

::: moniker-end

> [!NOTE]
> 対称ブロック暗号アルゴリズムのキーの長さは >= 128 ビット、ブロックサイズは正確に128ビットである必要があり、GCM 暗号化をサポートしている必要があります。 指定されたアルゴリズムの既定のプロバイダーを使用するには、 [EncryptionAlgorithmProvider](/dotnet/api/microsoft.aspnetcore.dataprotection.authenticatedencryption.configurationmodel.cngcbcauthenticatedencryptorconfiguration.encryptionalgorithmprovider) プロパティを null に設定します。 詳細については、 [BCryptOpenAlgorithmProvider](/windows/win32/api/bcrypt/nf-bcrypt-bcryptopenalgorithmprovider) のドキュメントを参照してください。

### <a name="specifying-other-custom-algorithms"></a>その他のカスタムアルゴリズムの指定

データ保護システムは、ファーストクラスの API として公開されていませんが、ほとんどすべての種類のアルゴリズムを指定できるように拡張されています。 たとえば、ハードウェアセキュリティモジュール (HSM) に含まれるすべてのキーを保持し、コア暗号化および復号化ルーチンのカスタム実装を提供することができます。 詳細については、「[コア暗号化機能拡張](xref:security/data-protection/extensibility/core-crypto)の[IAuthenticatedEncryptor](/dotnet/api/microsoft.aspnetcore.dataprotection.authenticatedencryption.iauthenticatedencryptor)」を参照してください。

## <a name="persisting-keys-when-hosting-in-a-docker-container"></a>Docker コンテナーでホストするときのキーの永続化

[Docker](/dotnet/standard/microservices-architecture/container-docker-introduction/)コンテナーでホストする場合は、次のいずれかの方法でキーを管理する必要があります。

* 共有ボリュームやホストによってマウントされたボリュームなど、コンテナーの有効期間を超えて永続化される Docker ボリュームのフォルダー。
* [Azure Key Vault](https://azure.microsoft.com/services/key-vault/)や[Redis](https://redis.io/)などの外部プロバイダー。

## <a name="persisting-keys-with-redis"></a>Redis でのキーの永続化

キーを格納するには、 [Redis データの永続](/azure/azure-cache-for-redis/cache-how-to-premium-persistence) 化をサポートする redis のバージョンのみを使用する必要があります。 [Azure Blob storage](/azure/storage/blobs/storage-blobs-introduction) は永続的で、キーを格納するために使用できます。 詳細については、次を参照してください。[この GitHub の問題](https://github.com/dotnet/AspNetCore/issues/13476)します。

## <a name="additional-resources"></a>その他の技術情報

* <xref:security/data-protection/configuration/non-di-scenarios>
* <xref:security/data-protection/configuration/machine-wide-policy>
* <xref:host-and-deploy/web-farm>
* <xref:security/data-protection/implementation/key-storage-providers>
