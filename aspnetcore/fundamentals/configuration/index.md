---
title: ASP.NET Core の構成
author: rick-anderson
description: 構成 API を使用して、ASP.NET Core アプリを構成する方法を説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 1/29/2021
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
uid: fundamentals/configuration/index
ms.openlocfilehash: fa7fbf1ae88b28cd36c5aece626240d849669a96
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586346"
---
# <a name="configuration-in-aspnet-core"></a>ASP.NET Core の構成

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT) および [Kirk Larkin](https://twitter.com/serpent5)

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core の構成は、1つまたは複数の[構成プロバイダー](#cp)を使用して実行されます。 構成プロバイダーは、以下のようなさまざまな構成ソースを使用して、キーと値のペアから構成データを読み取ります:

* *appsettings.json* などの設定ファイル
* 環境変数
* Azure Key Vault
* Azure App Configuration
* コマンド ライン引数
* インストール済みまたは作成済みのカスタム プロバイダー
* ディレクトリ ファイル
* メモリ内 .NET オブジェクト

このトピックでは ASP.NET Core の構成について説明します。 コンソール アプリでの構成の使用方法について詳しくは、[.NET 構成](/dotnet/core/extensions/configuration)に関する記事をご覧ください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/index/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

<a name="default"></a>

## <a name="default-configuration"></a>既定の構成

[dotnet new](/dotnet/core/tools/dotnet-new) または Visual Studio で作成された ASP.NET Core の web アプリが、次のコードを生成します:

[!code-csharp[](index/samples/3.x/ConfigSample/Program.cs?name=snippet&highlight=9)]

 <xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder*> により、次の順序でアプリの既定の構成が提供されます。

1. [ChainedConfigurationProvider](xref:Microsoft.Extensions.Configuration.ChainedConfigurationSource) は:既存の `IConfiguration` をソースとして追加します。 既定の構成では、[ホスト](#hvac)構成を追加し、_アプリ_ 構成の最初のソースとして設定します。
1. [JSON 構成プロバイダー](#file-configuration-provider)を使用する [appsettings.json](#appsettingsjson)。
1. [JSON 構成プロバイダー](#file-configuration-provider)を使用する *appsettings.* `Environment`*json*。 たとえば、*appsettings*.***Production**_._json* および *appsettings*.***Development** _._json*。
1. `Development` 環境でアプリが実行される際の [App シークレット](xref:security/app-secrets)。
1. [環境変数構成プロバイダー](#evcp)を使用する環境変数。
1. [コマンドライン構成プロバイダー](#command-line)を使用するコマンドライン引数。

後から追加される構成プロバイダーは、それ以前のキー設定をオーバーライドします。 たとえば、`MyKey` が *appsettings.json* と環境の両方で設定されている場合、環境の値が使用されます。 既定の構成プロバイダーを使用すると、[コマンドライン構成プロバイダー](#clcp) が他のすべてのプロバイダーをオーバーライドします。

`CreateDefaultBuilder` の詳細については、[既定のビルダー設定](xref:fundamentals/host/generic-host#default-builder-settings)を参照してください。

以下のコードでは、追加した順に、有効な構成プロバイダーが表示されます:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Index2.cshtml.cs?name=snippet)]

### appsettings.json

次の *appsettings.json* ファイルを考えてみます。

[!code-json[](index/samples/3.x/ConfigSample/appsettings.json)]

[サンプル ダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) の次のコードでは、上記の構成設定のいくつかが表示されます:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

既定の <xref:Microsoft.Extensions.Configuration.Json.JsonConfigurationProvider> は、以下の順序で構成を読み込みます:

1. *appsettings.json*
1. *appsettings.* `Environment` *.json*:たとえば、*appsettings*.***Production**_._json* および *appsettings*.***Development** _._json* ファイル。 ファイルの環境バージョンは、[IHostingEnvironment.EnvironmentName](xref:Microsoft.Extensions.Hosting.IHostingEnvironment.EnvironmentName*)に基づいて読み込まれます。 詳細については、「<xref:fundamentals/environments>」を参照してください。

*appsettings*.`Environment`.*json* の値によって、 *appsettings.json* 内のキーがオーバーライドされます。 たとえば、既定では次のようになります:

* 開発中は、*appsettings*.***Development** _._json* 構成によって、 *appsettings.json* で見つかった値が上書きされます。
* 運用環境では、*appsettings*.***Production** _._json* 構成によって、 *appsettings.json* で見つかった値が上書きされます。 たとえば、Azure にアプリをデプロイする場合。

構成値を保証する必要がある場合は、「[GetValue](#getvalue)」を参照してください。 前の例では文字列が読み取られるだけで、既定値はサポートされていません

<a name="optpat"></a>

### <a name="bind-hierarchical-configuration-data-using-the-options-pattern"></a>オプションパターンを使用して、階層型の構成データをバインドします

[!INCLUDE[](~/includes/bind.md)]

[既定](#default)の構成を利用する場合、[reloadOnChange: true](https://github.com/dotnet/extensions/blob/release/3.1/src/Hosting/Hosting/src/Host.cs#L74-L75) で *appsettings.json* と *appsettings.* `Environment` *.json* ファイルを有効にすることができます。 アプリの開始 ***後*** に *appsettings.json* と *appsettings.* `Environment` *.json* ファイルに加えられた変更は、[JSON 構成プロバイダー](#jcp)によって読み取られます。

追加の JSON 構成ファイルを追加する方法の詳細については、このドキュメント中の「[JSON 構成プロバイダー](#jcp)」を参照してください。

## <a name="combining-service-collection"></a>サービス コレクションの結合

[!INCLUDE[](~/includes/combine-di.md)]

<a name="security"></a>

## <a name="security-and-user-secrets"></a>セキュリティとユーザー シークレット

構成データのガイドライン:

* 構成プロバイダーのコードやプレーンテキストの構成ファイルには、パスワードなどの機密データを格納しないでください。 [Secret Manager](xref:security/app-secrets) ツールを使用すると、開発時にシークレットを格納できます。
* 開発環境やテスト環境では運用シークレットを使用しないでください。
* プロジェクトの外部にシークレットを指定してください。そうすれば、誤ってリソース コード リポジトリにコミットされることはありません。

[既定](#default)では、ユーザー シークレットの構成ソースは、JSON 構成ソースの後に登録されます。 このため、ユーザー シークレット キーは、 *appsettings.json* および *appsettings.* `Environment` *.json* よりも優先されます。

パスワードその他の機密データの格納については、次を参照してください：

* <xref:fundamentals/environments>
* <xref:security/app-secrets>:ここには、環境変数を使用して機密データを格納する際のアドバイスが記載されています。 Secret Manager ツールでは、[ファイル構成プロバイダー](#fcp)を使用して、ユーザーのシークレットがローカル システム上の JSON ファイルに格納されます。

[Azure Key Vault](https://azure.microsoft.com/services/key-vault/) では、ASP.NET Core アプリのアプリのシークレットが安全に保存されます。 詳細については、「<xref:security/key-vault-configuration>」を参照してください。

<a name="evcp"></a>

## <a name="environment-variables"></a>環境変数

[既定](#default)の構成を使用すると、<xref:Microsoft.Extensions.Configuration.EnvironmentVariables.EnvironmentVariablesConfigurationProvider> によって、 *appsettings.json* 、*appsettings.* `Environment` *.json*、および [ユーザー シークレット](xref:security/app-secrets)の読み取り後に、環境変数のキーと値のペアから構成が読み込まれます。 そのため、環境から読み取られたキー値によって、 *appsettings.json* 、*appsettings.* `Environment` *.json*、およびユーザー シークレットから読み取られた値がオーバーライドされます。

[!INCLUDE[](~/includes/environmentVarableColon.md)]

次の `set` コマンドは：

* Windows で[ 上記の例 ](#appsettingsjson)の環境キーと値を設定します。
* [サンプル ダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample)を使用する際に、設定をテストします。 `dotnet run` コマンドは、プロジェクト ディレクトリで実行する必要があります。

```dotnetcli
set MyKey="My key from Environment"
set Position__Title=Environment_Editor
set Position__Name=Environment_Rick
dotnet run
```

上記の環境設定は：

* コマンド ウィンドウから起動されたプロセスでのみ設定可能です。
* Visual Studio で起動されたブラウザーでは読み取れません。

次の [setx](/windows-server/administration/windows-commands/setx) コマンドで、Windows 上の環境キーと値を設定できます。 `set` とは異なり、`setx` 設定は保持されます。 `/M` は、システム環境で変数を設定します。 `/M` スイッチが使用されていない場合には、ユーザー環境変数が設定されます。

```console
setx MyKey "My key from setx Environment" /M
setx Position__Title Setx_Environment_Editor /M
setx Position__Name Environment_Rick /M
```

上記のコマンドによって、 *appsettings.json* および *appsettings.* `Environment` *.json* がオーバーライドされるのをテストするには:

* Visual Studio の場合:Visual Studio を終了して再起動します。
* CLI の場合:新しいコマンド ウィンドウを起動し、`dotnet run` を入力します。

環境変数のプレフィックスを指定する文字列を指定して <xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*> を呼び出します：

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Program.cs?name=snippet4&highlight=12)]

上のコードでは以下の操作が行われます。

* `config.AddEnvironmentVariables(prefix: "MyCustomPrefix_")` は[既定の構成プロバイダー](#default)の後に追加されます。 構成プロバイダーの順序付けの例については、「[JSON 構成プロバイダー](#jcp)」を参照してください。
* `MyCustomPrefix_` プレフィックスを使用して設定された環境変数は、[既定の構成プロバイダー](#default)をオーバーライドします。 これには、プレフィックスのない環境変数が含まれます。

構成のキーと値のペアの読み取り時に、プレフィックスは削除されます。

次のコマンドは、カスタム プレフィックスをテストします：

```dotnetcli
set MyCustomPrefix_MyKey="My key with MyCustomPrefix_ Environment"
set MyCustomPrefix_Position__Title=Editor_with_customPrefix
set MyCustomPrefix_Position__Name=Environment_Rick_cp
dotnet run
```

[既定の構成](#default)では、`DOTNET_` と `ASPNETCORE_` のプレフィックスが付いた環境変数とコマンド ライン引数を読み込みます。 `DOTNET_` と `ASPNETCORE_` のプレフィックスは ASP.NET Core によって[ホストとアプリの構成](xref:fundamentals/host/generic-host#host-configuration)に使用されますが、ユーザーの構成には使用されません。 ホストとアプリの構成の詳細については、「[.NET 汎用ホスト](xref:fundamentals/host/generic-host)」を参照してください。

[Azure App Service](https://azure.microsoft.com/services/app-service/) で、 **設定 > 構成** ページの **新しいアプリケーション設定** を選択します。 Azure App Service アプリケーションの設定は：

* 保存時に暗号化され、暗号化されたチャネルで送信されます。
* 環境変数として公開されます。

詳細については、「[Azure アプリ: Azure Portal を使用してアプリの構成をオーバーライドする](xref:host-and-deploy/azure-apps/index#override-app-configuration-using-the-azure-portal)」を参照してください。

Azure データベース接続文字列の詳細については、「[接続文字列のプレフィックス](#constr)」を参照してください。

### <a name="naming-of-environment-variables"></a>環境変数の名前付け

環境変数名には、 *appsettings.json* ファイルの構造が反映されます。 階層内の各要素は、二重アンダースコア (推奨) またはコロンによって区切られます。 要素構造に配列が含まれている場合は、配列のインデックスをこのパスの追加の要素名として扱う必要があります。 次の *appsettings.json* ファイルと、環境変数として表されたその同等の値を考えてみましょう。

**appsettings.json**

```json
{
    "SmtpServer": "smtp.example.com",
    "Logging": [
        {
            "Name": "ToEmail",
            "Level": "Critical",
            "Args": {
                "FromAddress": "MySystem@example.com",
                "ToAddress": "SRE@example.com"
            }
        },
        {
            "Name": "ToConsole",
            "Level": "Information"
        }
    ]
}
```

**環境変数**

```console
setx SmtpServer=smtp.example.com
setx Logging__0__Name=ToEmail
setx Logging__0__Level=Critical
setx Logging__0__Args__FromAddress=MySystem@example.com
setx Logging__0__Args__ToAddress=SRE@example.com
setx Logging__1__Name=ToConsole
setx Logging__1__Level=Information
```

### <a name="environment-variables-set-in-generated-launchsettingsjson"></a>生成された launchSettings.json に設定されている環境変数

*launchSettings.json* に設定されている環境変数で、システム環境に設定されているそれらがオーバーライドされます。 たとえば、ASP.NET Core Web テンプレートでは、エンドポイント構成を次のように設定する *launchSettings.json* ファイルが生成されます。

```json
"applicationUrl": "https://localhost:5001;http://localhost:5000"
```

`applicationUrl` を構成すると `ASPNETCORE_URLS` 環境変数が設定され、環境で設定されている値がオーバーライドされます。

### <a name="escape-environment-variables-on-linux"></a>Linux で環境変数をエスケープする

Linux では、`systemd` で解析できるように URL 環境変数の値をエスケープする必要があります。 Linux ツール `systemd-escape` を使用すると、`http:--localhost:5001` が生成されます
 
 ```cmd
 groot@terminus:~$ systemd-escape http://localhost:5001
 http:--localhost:5001
 ```

### <a name="display-environment-variables"></a>環境変数を表示する

次のコードでは、アプリケーションの起動時に環境変数と値が表示されます。これは環境設定をデバッグするときに役立つことがあります。

[!code-csharp[](~/fundamentals/configuration/index/samples_snippets/5.x/Program.cs?name=snippet)]

<a name="clcp"></a>

## <a name="command-line"></a>コマンド ライン

[既定](#default)の構成を使用して、<xref:Microsoft.Extensions.Configuration.CommandLine.CommandLineConfigurationProvider> は、以下の構成ソースの後にコマンド ライン引数のキーと値のペアから構成を読み込みます：

* *appsettings.json* および *appsettings*.`Environment`.*json* ファイル。
* 開発環境での[アプリ シークレット](xref:security/app-secrets)。
* 環境変数。

[既定](#default)では、コマンド ラインで設定された構成値は、他のすべての構成プロバイダーで設定された構成値をオーバーライドします。

### <a name="command-line-arguments"></a>コマンド ライン引数

次のコマンドは `=` を使用してキーと値を設定します：

```dotnetcli
dotnet run MyKey="My key from command line" Position:Title=Cmd Position:Name=Cmd_Rick
```

次のコマンドは `/` を使用してキーと値を設定します：

```dotnetcli
dotnet run /MyKey "Using /" /Position:Title=Cmd_ /Position:Name=Cmd_Rick
```

次のコマンドは `--` を使用してキーと値を設定します：

```dotnetcli
dotnet run --MyKey "Using --" --Position:Title=Cmd-- --Position:Name=Cmd--Rick
```

キーの値:

* `=` の後に続ける必要があります。または、値がスペースの後にある場合は、キーのプレフィックスが `--` または `/` である必要があります。
* `=` を使用する場合は必要ありません。 たとえば、`MySetting=` のようにします。

同じコマンド内では、`=` を使用するコマンド ライン引数のキーと値のペアを、スペースを使用するキーと値のペアと混在させないでください。

### <a name="switch-mappings"></a>スイッチ マッピング

スイッチ マッピングでは、**キー** 名の置換ロジックが許可されます。 <xref:Microsoft.Extensions.Configuration.CommandLineConfigurationExtensions.AddCommandLine*> メソッドにスイッチ置換するディクショナリを提供します。

スイッチ マッピング ディクショナリが使用されている場合、そのディレクトリで、コマンドライン引数によって指定されたキーと一致するキーが確認されます。 ディクショナリ中にコマンド ライン キーが見つかった場合は、そのディクショナリの値が返され、キーと値のペアがアプリの構成に設定されます。 スイッチ マッピングは、単一のダッシュ (`-`) が前に付いたすべてのコマンドライン キーに必要です。

スイッチ マッピング ディクショナリ キーの規則:

* スイッチは `-` または `--`で始める必要があります。
* スイッチ マッピング ディクショナリに重複キーを含めることはできません。

スイッチ マッピング ディクショナリを使用するには、それを `AddCommandLine` の呼び出しに渡します：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramSwitch.cs?name=snippet&highlight=10-18,23)]

次のコードは、置換されたキーのキー値を示しています：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test3.cshtml.cs?name=snippet)]

次のコマンドを実行して、キーの置換をテストできます：

```dotnetcli
dotnet run -k1 value1 -k2 value2 --alt3=value2 /alt4=value3 --alt5 value5 /alt6 value6
```

スイッチ マッピングを使用するアプリでは、`CreateDefaultBuilder` への呼び出しで引数を渡すことはできません。 `CreateDefaultBuilder` メソッドの `AddCommandLine` 呼び出しには、マップされたスイッチが含まれていないため、スイッチ マッピング ディクショナリを `CreateDefaultBuilder` に渡すことはできません。 解決策は、`CreateDefaultBuilder` に引数を渡す代わりに、`ConfigurationBuilder` メソッドの `AddCommandLine` メソッドに、引数とスイッチ マッピング ディクショナリの両方を処理させることです。

## <a name="hierarchical-configuration-data"></a>階層的な構成データ

構成 API では、構成キーの区切り記号を使用して階層データをフラット化することにより、階層型の構成データの読み取りが行われます。

[サンプル ダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) には、次の *appsettings.json* ファイルが含まれます:

[!code-json[](index/samples/3.x/ConfigSample/appsettings.json)]

[サンプル ダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) の次のコードでは、構成設定のいくつかが表示されます：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

階層型の構成データを読み取る方法としては、オプション パターンを使用することをお勧めします。 詳細については、このドキュメント中の「[階層型の構成データをバインドする](#optpat)」を参照してください。

<xref:Microsoft.Extensions.Configuration.ConfigurationSection.GetSection*> メソッドと <xref:Microsoft.Extensions.Configuration.IConfiguration.GetChildren*> メソッドを使用して、構成データ内のセクションとセクションの子を分離することができます。 これらのメソッドについては、後ほど「[GetSection、GetChildren、Exists](#getsection)」で説明します。

<!--
[Azure Key Vault configuration provider](xref:security/key-vault-configuration) implement change detection.
-->

## <a name="configuration-keys-and-values"></a>構成キーと値

構成キー:

* 構成キーでは、大文字と小文字は区別されません。 たとえば、`ConnectionString` と `connectionstring` は同等のキーとして扱われます。
* キーと値が複数の構成プロバイダーで設定されている場合は、最後に追加されたプロバイダーの値が使用されます。 詳細については、「[既定の構成](#default)」を参照してください。
* 階層キー
  * 構成 API 内では、すべてのプラットフォームでコロン (`:`) の区切りが機能します。
  * 環境変数内では、コロン区切りがすべてのプラットフォームでは機能しない場合があります。 ダブル アンダースコア (`__`) はすべてのプラットフォームでサポートされ、コロン `:` に自動で変換されます。
  * Azure Key Vault では、階層型のキーは区切り記号に `--` を使用します。 シークレットがアプリの構成に読み込まれると、[Azure Key Vault 構成プロバイダー](xref:security/key-vault-configuration)によって `--` が `:` に自動的に置き換えられます。
* <xref:Microsoft.Extensions.Configuration.ConfigurationBinder> は、構成キーで配列インデックスを使用して、オブジェクトに対する配列のバインドをサポートしています。 配列のバインドについては、「[配列をクラスにバインドする](#boa)」セクションで説明します。

構成値:

* 構成値は文字列です。
* Null 値を構成に格納したり、オブジェクトにバインドしたりすることはできません。

<a name="cp"></a>

## <a name="configuration-providers"></a>構成プロバイダー

ASP.NET Core アプリで使用できる構成プロバイダーを次の表に示します。

| プロバイダー | 以下から構成を提供します |
| -------- | ----------------------------------- |
| [Azure Key Vault 構成プロバイダー](xref:security/key-vault-configuration) | Azure Key Vault |
| [Azure App Configuration プロバイダー](/azure/azure-app-configuration/quickstart-aspnet-core-app) | Azure App Configuration |
| [コマンド ライン構成プロバイダー](#clcp) | コマンド ライン パラメーター |
| [カスタム構成プロバイダー](#custom-configuration-provider) | カスタム ソース |
| [環境変数構成プロバイダー](#evcp) | 環境変数 |
| [ファイル構成プロバイダー](#file-configuration-provider) | INI、JSON、および XML ファイル |
| [ファイルごとのキーの構成プロバイダー](#key-per-file-configuration-provider) | ディレクトリ ファイル |
| [メモリ構成プロバイダー](#memory-configuration-provider) | メモリ内コレクション |
| [ユーザー シークレット](xref:security/app-secrets) | ユーザー プロファイル ディレクトリ内のファイル |

構成ソースは、構成プロバイダーで指定された順序で読み取られます。 アプリで必要とされる、基になる構成ソースの優先順位に合わせるために、コード内で構成プロバイダーを並べ替えます。

一般的な一連の構成プロバイダーは次のとおりです。

1. *appsettings.json*
1. *appsettings*.`Environment`.*json*
1. [ユーザー シークレット](xref:security/app-secrets)
1. [環境変数構成プロバイダー](#evcp)を使用する環境変数。
1. [コマンドライン構成プロバイダー](#command-line-configuration-provider)を使用するコマンドライン引数。

コマンド ライン引数が他のプロバイダーによって設定された構成をオーバーライドできるようにするには、コマンド ラインの構成プロバイダーを一連のプロバイダーの最後に配置するのが一般的です。

上記の一連のプロバイダーは、[既定の構成](#default)で使用されます。

<a name="constr"></a>

### <a name="connection-string-prefixes"></a>接続文字列のプレフィックス

構成 API には、4つの接続文字列環境変数に対する特別なプロセスルールがあります。 これらの接続文字列は、アプリ環境用の Azure 接続文字列の構成に含まれています。 表に示されたプレフィックスを持つ環境変数は、[既定の構成](#default) を使用するとき、または `AddEnvironmentVariables` にプレフィックスが指定されていない場合に、アプリに読み込まれます。

| 接続文字列のプレフィックス | プロバイダー |
| ------------------------ | -------- |
| `CUSTOMCONNSTR_` | カスタム プロバイダー |
| `MYSQLCONNSTR_` | [MySQL](https://www.mysql.com/) |
| `SQLAZURECONNSTR_` | [Azure SQL Database](https://azure.microsoft.com/services/sql-database/) |
| `SQLCONNSTR_` | [SQL Server](https://www.microsoft.com/sql-server/) |

表に示す 4 つのプレフィックスのいずれかを使用して、環境変数が検出され構成に読み込まれた場合:

* 環境変数のプレフィックスを削除し、構成キーのセクション (`ConnectionStrings`) を追加することによって、構成キーが作成されます。
* データベースの接続プロバイダーを表す新しい構成のキーと値のペアが作成されます (示されたプロバイダーを含まない `CUSTOMCONNSTR_` を除く)。

| 環境変数キー | 変換された構成キー | プロバイダーの構成エントリ                                                    |
| ------------------------ | --------------------------- | ------------------------------------------------------------------------------- |
| `CUSTOMCONNSTR_{KEY} `   | `ConnectionStrings:{KEY}`   | 構成エントリは作成されません。                                                |
| `MYSQLCONNSTR_{KEY}`     | `ConnectionStrings:{KEY}`   | キー: `ConnectionStrings:{KEY}_ProviderName`:<br>値: `MySql.Data.MySqlClient` |
| `SQLAZURECONNSTR_{KEY}`  | `ConnectionStrings:{KEY}`   | キー: `ConnectionStrings:{KEY}_ProviderName`:<br>値: `System.Data.SqlClient`  |
| `SQLCONNSTR_{KEY}`       | `ConnectionStrings:{KEY}`   | キー: `ConnectionStrings:{KEY}_ProviderName`:<br>値: `System.Data.SqlClient`  |

<a name="fcp"></a>

## <a name="file-configuration-provider"></a>ファイル構成プロバイダー

<xref:Microsoft.Extensions.Configuration.FileConfigurationProvider> は、ファイル システムから構成を読み込むための基本クラスです。 以下の構成プロバイダーは `FileConfigurationProvider` から派生したものです：

* [INI 構成プロバイダー](#ini-configuration-provider)
* [JSON 構成プロバイダー](#jcp)
* [XML 構成プロバイダー](#xml-configuration-provider)

### <a name="ini-configuration-provider"></a>INI 構成プロバイダー

<xref:Microsoft.Extensions.Configuration.Ini.IniConfigurationProvider> では、実行時に INI ファイルのキーと値のペアから構成が読み込まれます。

次のコードは、すべての構成プロバイダーをクリアし、いくつかの構成プロバイダーを追加します：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramINI.cs?name=snippet&highlight=10-30)]

上記のコードでは、*MyIniConfig.ini* と *MyIniConfig*.`Environment`.*ini* ファイルの設定は、以下の設定によってオーバーライドされます：

* [環境変数構成プロバイダー](#evcp)
* [コマンドライン構成プロバイダー](#clcp)。

[サンプルダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) には、次の *MyIniConfig* ファイルが含まれます：

[!code-ini[](index/samples/3.x/ConfigSample/MyIniConfig.ini)]

[サンプル ダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) の次のコードでは、上記の構成設定のいくつかが表示されます:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

<a name="jcp"></a>

### <a name="json-configuration-provider"></a>JSON 構成プロバイダー

<xref:Microsoft.Extensions.Configuration.Json.JsonConfigurationProvider> では、JSON ファイルのキーと値のペアから構成が読み込みまれます。

オーバーロードでは、次の指定ができます：

* ファイルを省略可能かどうか。
* ファイルが変更された場合に構成を再度読み込むかどうか。

次のコードがあるとします。

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramJSON.cs?name=snippet&highlight=12-14)]

上記のコードでは次の操作が行われます。

* 次のオプションを使用して、*MyConfig.json* ファイルを読み込むように JSON 構成プロバイダーを構成します：
  * `optional: true`:ファイルは省略可能です。
  * `reloadOnChange: true` は、次のとおりです。変更が保存されると、ファイルが再読み込みされます。
* *MyConfig.json* ファイルの前に [既定の構成プロバイダー](#default)を読み取ります。 [環境変数構成プロバイダー](#evcp) および [コマンド ライン構成プロバイダー](#clcp)を含む、既定の構成プロバイダーでの *MyConfig.json* ファイルのオーバーライドの設定。

通常は、[環境変数構成プロバイダー](#evcp)および [コマンドライン構成プロバイダー](#clcp)で設定されている値をオーバーライドするカスタム JSON ファイルは ***必要ありません***。

次のコードは、すべての構成プロバイダーをクリアし、いくつかの構成プロバイダーを追加します：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramJSON2.cs?name=snippet)]

上記のコードでは、*MyConfig.json* と *MyConfig*.`Environment`.*json* ファイルの設定は：

* *appsettings.json* および *appsettings*.`Environment`.*json* ファイル内の設定をオーバーライドします。
* [環境変数の構成プロバイダー](#evcp)と[コマンドライン構成プロバイダー](#clcp)の設定によってオーバーライドされます。

[サンプル ダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample)には、次の *MyConfig.json* ファイルが含まれます：

[!code-json[](index/samples/3.x/ConfigSample/MyConfig.json)]

[サンプル ダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) の次のコードでは、上記の構成設定のいくつかが表示されます:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

### <a name="xml-configuration-provider"></a>XML 構成プロバイダー

<xref:Microsoft.Extensions.Configuration.Xml.XmlConfigurationProvider> では、実行時に XML ファイルのキーと値のペアから構成が読み込まれます。

次のコードは、すべての構成プロバイダーをクリアし、いくつかの構成プロバイダーを追加します：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramXML.cs?name=snippet)]

上記のコードでは、*MyXMLFile.xml* と *MyXMLFile*.`Environment`.*xml* ファイルの設定は、以下の設定によってオーバーライドされます：

* [環境変数構成プロバイダー](#evcp)
* [コマンドライン構成プロバイダー](#clcp)。

[サンプルダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) には、次の *MyXMLFile.xml* ファイルが含まれます：

[!code-xml[](index/samples/3.x/ConfigSample/MyXMLFile.xml)]

[サンプル ダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) の次のコードでは、上記の構成設定のいくつかが表示されます:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

同じ要素名を使用する要素の繰り返しは、要素を区別するために `name` 属性を使用する場合に機能します。

[!code-xml[](index/samples/3.x/ConfigSample/MyXMLFile3.xml)]

以下のコードでは、前の構成ファイルを読み取って、キーと値を表示します：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/XML/Index.cshtml.cs?name=snippet)]

値を指定するために属性を使用できます。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <key attribute="value" />
  <section>
    <key attribute="value" />
  </section>
</configuration>
```

前の構成ファイルでは、`value` を使用して次のキーが読み込まれます。

* key:attribute
* section:key:attribute

## <a name="key-per-file-configuration-provider"></a>ファイルごとのキーの構成プロバイダー

<xref:Microsoft.Extensions.Configuration.KeyPerFile.KeyPerFileConfigurationProvider> では、構成のキーと値のペアとしてディレクトリのファイルが使用されます。 キーはファイル名です。 値にはファイルのコンテンツが含まれます。 ファイルごとのキーの構成プロバイダーは、Docker ホスティングのシナリオで使用されます。

ファイルごとのキーの構成をアクティブにするには、<xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> のインスタンスの <xref:Microsoft.Extensions.Configuration.KeyPerFileConfigurationBuilderExtensions.AddKeyPerFile*> 拡張メソッドを呼び出します。 ファイルに対する `directoryPath` は、絶対パスである必要があります。

オーバーロードによって次のものを指定できます。

* ソースを構成する `Action<KeyPerFileConfigurationSource>` デリゲート。
* ディレクトリを省略可能かどうか、またディレクトリへのパス。

アンダースコア 2 つ (`__`) は、ファイル名で構成キーの区切り記号として使用されます。 たとえば、ファイル名 `Logging__LogLevel__System` では、構成キー `Logging:LogLevel:System` が生成されます。

ホストをビルドするときに `ConfigureAppConfiguration` を呼び出して、アプリの構成を指定します。

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    var path = Path.Combine(
        Directory.GetCurrentDirectory(), "path/to/files");
    config.AddKeyPerFile(directoryPath: path, optional: true);
})
```

<a name="mcp"></a>

## <a name="memory-configuration-provider"></a>メモリ構成プロバイダー

<xref:Microsoft.Extensions.Configuration.Memory.MemoryConfigurationProvider> では、構成のキーと値のペアとして、メモリ内コレクションが使用されます。

次のコードでは、構成システムにメモリコ レクションが追加されています：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramArray.cs?name=snippet6)]

[サンプルのダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) の以下のコードでは、上記の構成設定のいくつかが表示されます：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

上記のコードでは、[既定の構成プロバイダー](#default)の後に `config.AddInMemoryCollection(Dict)` が追加されています。 構成プロバイダーの順序付けの例については、「[JSON 構成プロバイダー](#jcp)」を参照してください。

`MemoryConfigurationProvider` を使用した別の例については、[配列をバインド](#boa)を参照してください。

::: moniker-end
::: moniker range=">= aspnetcore-5.0"

<a name="kestrel"></a>

## <a name="kestrel-endpoint-configuration"></a>Kestrel のエンドポイントの構成

Kestrel 固有のエンドポイント構成によって、すべての[サーバー間](xref:fundamentals/servers/index)のエンドポイント構成がオーバーライドされます。 サーバー間のエンドポイント構成には、次のものがあります。

  * [UseUrls](xref:fundamentals/host/web-host#server-urls)
  * [コマンド ライン](xref:fundamentals/configuration/index#command-line)での `--urls`
  * [環境変数](xref:fundamentals/configuration/index#environment-variables) `ASPNETCORE_URLS`

ASP.NET Core Web アプリで使用される次の *appsettings.json* ファイルについて考えてみましょう。

[!code-json[](~/fundamentals/configuration/index/samples_snippets/5.x/appsettings.json?highlight=2-8)]

上の強調表示されているマークアップが ASP.NET Core Web アプリで使用され、"***かつ***" 次のサーバー間エンドポイント構成を使用してコマンド ラインでアプリが起動される場合:

`dotnet run --urls="https://localhost:7777"`

Kestrel は、`https://localhost:7777` ではなく、 *appsettings.json* ファイルで Kestrel 専用に構成されたエンドポイント (`https://localhost:9999`) にバインドされます。

環境変数として構成された Kestrel 固有のエンドポイントを考えてみます。

`set Kestrel__Endpoints__Https__Url=https://localhost:8888`

上記の環境変数では、`Https` が Kestrel 固有のエンドポイントの名前です。 前の *appsettings.json* ファイルでも、`Https` という名前の Kestrel 固有のエンドポイントが定義されています。 [既定](#default-configuration)で、[環境変数構成プロバイダー](#evcp)を使用している環境変数は *appsettings.* `Environment` *.json* の後に読み取られます。したがって、上記の環境変数は `Https` エンドポイント用に使用されます。

::: moniker-end
::: moniker range=">= aspnetcore-3.0"

## <a name="getvalue"></a>GetValue

[`ConfigurationBinder.GetValue<T>`](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.GetValue*) 指定したキーを使用して、構成から単一の値を抽出し、それを指定した型に変換します：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/TestNum.cshtml.cs?name=snippet)]

上記のコードでは、`NumberKey` が構成中に見つからない場合には `99` の既定値が使用されます。

## <a name="getsection-getchildren-and-exists"></a>GetSection、GetChildren、Exists

以下の例では、次の *MySubsection.json* ファイルについて考えます：

[!code-json[](index/samples/3.x/ConfigSample/MySubsection.json)]

以下のコードでは、*MySubsection セクション* を構成プロバイダーに追加します：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramJSONsection.cs?name=snippet)]

### <a name="getsection"></a>GetSection

[IConfiguration.GetSection](xref:Microsoft.Extensions.Configuration.IConfiguration.GetSection*) は、指定されたサブセクションのキーを持つ構成サブセクションを返します。

次のコードは、`section1` の値を返します：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/TestSection.cshtml.cs?name=snippet)]

次のコードは、`section2:subsection0` の値を返します：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/TestSection2.cshtml.cs?name=snippet)]

`GetSection` が `null` を返すことはありません。 一致するセクションが見つからない場合は、空の `IConfigurationSection` が返されます。

`GetSection` で一致するセクションが返されると、<xref:Microsoft.Extensions.Configuration.IConfigurationSection.Value> は設定されません。 <xref:Microsoft.Extensions.Configuration.IConfigurationSection.Key> と <xref:Microsoft.Extensions.Configuration.IConfigurationSection.Path> はセクションが存在する場合に返されます。

### <a name="getchildren-and-exists"></a>GetChildren と Exists

次のコードは、[IConfiguration.GetChildren](xref:Microsoft.Extensions.Configuration.IConfiguration.GetChildren*) を呼び出し、`section2:subsection0` の値を返します：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/TestSection4.cshtml.cs?name=snippet)]

上記のコードは、[ConfigurationExtensions. Exists](xref:Microsoft.Extensions.Configuration.ConfigurationExtensions.Exists*) を呼び出し、セクションが存在することを確認します：

 <a name="boa"></a>

## <a name="bind-an-array"></a>配列をバインドする

[ConfigurationBinder. Bind](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Bind*) は、構成キーの配列インデックスを使用して、オブジェクトに対する配列のバインドをサポートしています。 数値のキー セグメントを公開する配列形式は、すべて [POCO](https://wikipedia.org/wiki/Plain_Old_CLR_Object) クラスの配列にバインドできます。

[サンプル ダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample)の *MyArray.json* について考えます：

[!code-json[](index/samples/3.x/ConfigSample/MyArray.json)]

次のコードでは、*MyArray.json* を構成プロバイダーに追加します：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramJSONarray.cs?name=snippet)]

次のコードでは、構成を読み取り、値を表示します：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Array.cshtml.cs?name=snippet)]

上記のコードは、次の出力を返します：

```text
Index: 0  Value: value00
Index: 1  Value: value10
Index: 2  Value: value20
Index: 3  Value: value40
Index: 4  Value: value50
```

上記の出力では、インデックス3の値が `value40` になります。これは *MyArray. json* の `"4": "value40",` に対応しています。 このバインドされた配列インデックスは連続的であり、構成キーインデックスにバインドされていません。 構成バインダーは、バインドされたオブジェクトに null 値をバインドしたり、null エントリを作成したりはできません

次のコードでは、<xref:Microsoft.Extensions.Configuration.MemoryConfigurationBuilderExtensions.AddInMemoryCollection*> 拡張メソッドで `array:entries` 構成を読み込みます：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramArray.cs?name=snippet)]

次のコードでは、`arrayDict` `Dictionary` の構成を読み取り、値を表示します：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Array.cshtml.cs?name=snippet)]

上記のコードは、次の出力を返します：

```text
Index: 0  Value: value0
Index: 1  Value: value1
Index: 2  Value: value2
Index: 3  Value: value4
Index: 4  Value: value5
```

バインドされたオブジェクトのインデックス &num;3 によって、`array:4` 構成キーの構成データと、その値 `value4` が保持されます。 配列を含む構成データがバインドされている場合、構成キーの配列インデックスは、オブジェクト作成時の構成データの反復のために使用されます。 構成データに null 値を保持することはできません。また、構成キーの配列が 1 つまたは複数のインデックスをスキップしても、バインドされたオブジェクトに null 値のエントリは作成されません。

インデックス&num;3 に不足している構成項目は、`ArrayExample` インスタンスにバインドする前に、インデックス&num;3のキーと値のペアを読み取る構成プロバイダーによってサプライできます。 サンプルダウンロードの、次の *Value3.json* ファイルについて考えます：

[!code-json[](index/samples/3.x/ConfigSample/Value3.json)]

次のコードには、*Value3.json* と `arrayDict` `Dictionary` の構成が含まれています：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramArray.cs?name=snippet2)]

次のコードは、上記の構成を読み取り、値を表示します：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Array.cshtml.cs?name=snippet)]

上記のコードは、次の出力を返します：

```text
Index: 0  Value: value0
Index: 1  Value: value1
Index: 2  Value: value2
Index: 3  Value: value3
Index: 4  Value: value4
Index: 5  Value: value5
```

カスタム構成プロバイダーが配列のバインドを実装する必要はありません。

## <a name="custom-configuration-provider"></a>カスタム構成プロバイダー

サンプル アプリでは、[Entity Framework (EF)](/ef/core/) を使用してデータベースから構成のキーと値のペアを読み取る、基本的な構成プロバイダーを作成する方法を示します。

プロバイダーの特徴は次のとおりです。

* EF のメモリ内データベースは、デモンストレーションのために使用されます。 接続文字列を必要とするデータベースを使用するには、第 2 の `ConfigurationBuilder` を実装して、別の構成プロバイダーからの接続文字列を指定します。
* プロバイダーは、起動時に、構成にデータベース テーブルを読み取ります。 プロバイダーは、キー単位でデータベースを照会しません。
* 変更時に再度読み込む機能は実装されていません。このため、アプリの起動後にデータベースを更新しても、アプリの構成には影響がありません。

データベースに構成値を格納するための `EFConfigurationValue` エンティティを定義します。

*Models/EFConfigurationValue.cs*:

[!code-csharp[](index/samples/3.x/ConfigurationSample/Models/EFConfigurationValue.cs?name=snippet1)]

構成した値を格納し、その値にアクセスするための `EFConfigurationContext` を追加します。

*EFConfigurationProvider/EFConfigurationContext.cs*:

[!code-csharp[](index/samples/3.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationContext.cs?name=snippet1)]

<xref:Microsoft.Extensions.Configuration.IConfigurationSource> を実装するクラスを作成します。

*EFConfigurationProvider/EFConfigurationSource.cs*:

[!code-csharp[](index/samples/3.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationSource.cs?name=snippet1)]

<xref:Microsoft.Extensions.Configuration.ConfigurationProvider> から継承して、カスタム構成プロバイダーを作成します。 データベースが空だった場合、構成プロバイダーはこれを初期化します。 [構成キーでは大文字と小文字が区別されない](#keys)ため、データベースの初期化に使用されるディクショナリは、大文字と小文字を区別しない比較子 ([StringComparer.OrdinalIgnoreCase](xref:System.StringComparer.OrdinalIgnoreCase)) を使用して作成されます。

*EFConfigurationProvider/EFConfigurationProvider.cs*:

[!code-csharp[](index/samples/3.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationProvider.cs?name=snippet1)]

`AddEFConfiguration` 拡張メソッドを使用すると、`ConfigurationBuilder` に構成ソースを追加できます。

*Extensions/EntityFrameworkExtensions.cs*:

[!code-csharp[](index/samples/3.x/ConfigurationSample/Extensions/EntityFrameworkExtensions.cs?name=snippet1)]

次のコードでは、*Program.cs* でカスタムの `EFConfigurationProvider` を使用する方法を示します。

[!code-csharp[](index/samples_snippets/3.x/ConfigurationSample/Program.cs?highlight=7-8)]

<a name="acs"></a>

## <a name="access-configuration-in-startup"></a>起動時の構成へのアクセス

次のコードでは `Startup` メソッドの構成データが表示されます：

[!code-csharp[](index/samples/3.x/ConfigSample/StartupKey.cs?name=snippet&highlight=13,18)]

起動時の簡易メソッドを使用して構成にアクセスする例については、[アプリ起動時の簡易メソッド](xref:fundamentals/startup#convenience-methods)に関連する記事をご覧ください。

## <a name="access-configuration-in-razor-pages"></a>Razor ページの構成にアクセスする

次のコードでは Razor ページの構成データが表示されます:

[!code-cshtml[](index/samples/3.x/ConfigSample/Pages/Test5.cshtml)]

次のコードでは、`MyOptions` は <xref:Microsoft.Extensions.DependencyInjection.OptionsConfigurationServiceCollectionExtensions.Configure*> でサービスコンテナーに追加され、構成にバインドされます:

[!code-csharp[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/Startup3.cs?name=snippet_Example2)]

次のマークアップは、[`@inject`](xref:mvc/views/razor#inject) Razor ディレクティブを使用して、オプションの値を解決して表示します。

[!code-cshtml[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/Pages/Test3.cshtml)]

## <a name="access-configuration-in-a-mvc-view-file"></a>MVC ビューファイルの構成へのアクセス

次のコードでは、MVC ビューの構成データが表示されます：

[!code-cshtml[](index/samples/3.x/ConfigSample/Views/Home2/Index.cshtml)]

## <a name="configure-options-with-a-delegate"></a>デリゲートでオプションを構成する

デリゲートで構成されたオプションは、構成プロバイダーで設定された値をオーバーライドします。

サンプル アプリの例 2 では、デリゲートでオプションを構成しています。

次のコードでは、<xref:Microsoft.Extensions.Options.IConfigureOptions%601> サービスがサービス コンテナーに追加されます。 デリゲートを利用して `MyOptions` の値が構成されます。

[!code-csharp[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/Startup2.cs?name=snippet_Example2)]

このコードには、次のオプションの値が表示されます。

[!code-csharp[](options/samples/3.x/OptionsSample/Pages/Test2.cshtml.cs?name=snippet)]

先の例では、値 `Option1` と `Option2` が *appsettings.json* で指定されてから、構成されているデリゲートによりオーバーライドされます。

<a name="hvac"></a>

## <a name="host-versus-app-configuration"></a>ホストとアプリの構成

アプリを構成して起動する前に、"*ホスト*" を構成して起動します。 ホストはアプリの起動と有効期間の管理を担当します。 アプリとホストは、両方ともこのトピックで説明する構成プロバイダーを使用して構成します。 ホスト構成のキーと値のペアも、アプリの構成に含まれます。 ホストをビルドするときの構成プロバイダーの使用方法、およびホストの構成に対する構成ソースの影響について詳しくは、「<xref:fundamentals/index#host>」をご覧ください。

<a name="dhc"></a>

## <a name="default-host-configuration"></a>既定のホスト構成

[Web ホスト](xref:fundamentals/host/web-host)を使用する場合の既定の構成の詳細については、[このトピックの ASP.NET Core 2.2 バージョン](?view=aspnetcore-2.2&preserve-view=true)を参照してください。

* ホストの構成は、次から提供されます。
  * [環境変数構成プロバイダー](#environment-variables)を使用する、プレフィックス `DOTNET_` (`DOTNET_ENVIRONMENT` など) が付いた環境変数。 構成のキーと値のペアが読み込まれるときに、プレフィックス (`DOTNET_`) は削除されます。
  * [コマンドライン構成プロバイダー](#command-line-configuration-provider)を使用するコマンドライン引数。
* Web ホストの既定の構成が確立されます (`ConfigureWebHostDefaults`)。
  * Kestrel は Web サーバーとして使用され、アプリの構成プロバイダーを使用して構成されます。
  * Host Filtering Middleware を追加します。
  * `ASPNETCORE_FORWARDEDHEADERS_ENABLED` 環境変数が `true` に設定されている場合は、Forwarded Headers Middleware を追加します。
  * IIS 統合を有効にします。

## <a name="other-configuration"></a>その他の構成

このトピックは、"*アプリの構成*" のみに関連しています。 ASP.NET Core アプリの実行とホストに関するその他の側面は、このトピックでは扱わない構成ファイルを使って構成されます。

* *launch.json*/*launchSettings.json* は、開発環境用のツール構成ファイルです。以下で説明されています。
  * <xref:fundamentals/environments#development>、
  * 開発シナリオ用の ASP.NET Core アプリを構成するためにこのファイルが使用されている、ドキュメント セット全体。
* *web.config* はサーバー構成ファイルです。次のトピックで説明されています。
  * <xref:host-and-deploy/iis/index>
  * <xref:host-and-deploy/aspnet-core-module>

*launchSettings.json* に設定されている環境変数で、システム環境に設定されているそれらがオーバーライドされます。

以前のバージョンの ASP.NET からアプリの構成を移行する方法について詳しくは、<xref:migration/proper-to-2x/index#store-configurations> をご覧ください。

## <a name="add-configuration-from-an-external-assembly"></a>外部アセンブリから構成を追加する

<xref:Microsoft.AspNetCore.Hosting.IHostingStartup> の実装により、アプリの `Startup` クラスの外部にある外部アセンブリから、起動時に拡張機能をアプリに追加できるようになります。 詳細については、「<xref:fundamentals/configuration/platform-specific-configuration>」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* [構成のソースコード](https://github.com/dotnet/runtime/tree/main/src/libraries/Microsoft.Extensions.Configuration)
* <xref:fundamentals/configuration/options>
* <xref:blazor/fundamentals/configuration>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core でのアプリの構成は、"*構成プロバイダー*" によって設定するキーと値のペアに基づいています。 構成プロバイダーは、さまざまな構成のソースから構成データを読み取り、キーと値のペアを作成します。

* Azure Key Vault
* Azure App Configuration
* コマンド ライン引数
* カスタム プロバイダー (インストール済みまたは作成済み)
* ディレクトリ ファイル
* 環境変数
* メモリ内 .NET オブジェクト
* 設定ファイル

一般的な構成プロバイダーのシナリオに向けた構成パッケージ ([Microsoft.Extensions.Configuration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration/)) は、[Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)に含まれます。

以下とサンプル アプリのコード例では、<xref:Microsoft.Extensions.Configuration> 名前空間を使います。

```csharp
using Microsoft.Extensions.Configuration;
```

"*オプション パターン*" は、このトピックで説明する構成の概念を拡張したものです。 オプションでは、クラスを使用して関連する設定のグループを表します。 詳細については、「<xref:fundamentals/configuration/options>」を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/index/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="host-versus-app-configuration"></a>ホストとアプリの構成

アプリを構成して起動する前に、"*ホスト*" を構成して起動します。 ホストはアプリの起動と有効期間の管理を担当します。 アプリとホストは、両方ともこのトピックで説明する構成プロバイダーを使用して構成します。 ホスト構成のキーと値のペアも、アプリの構成に含まれます。 ホストをビルドするときの構成プロバイダーの使用方法、およびホストの構成に対する構成ソースの影響について詳しくは、「<xref:fundamentals/index#host>」をご覧ください。

## <a name="other-configuration"></a>その他の構成

このトピックは、"*アプリの構成*" のみに関連しています。 ASP.NET Core アプリの実行とホストに関するその他の側面は、このトピックでは扱わない構成ファイルを使って構成されます。

* *launch.json*/*launchSettings.json* は、開発環境用のツール構成ファイルです。以下で説明されています。
  * <xref:fundamentals/environments#development>、
  * 開発シナリオ用の ASP.NET Core アプリを構成するためにこのファイルが使用されている、ドキュメント セット全体。
* *web.config* はサーバー構成ファイルです。次のトピックで説明されています。
  * <xref:host-and-deploy/iis/index>
  * <xref:host-and-deploy/aspnet-core-module>

以前のバージョンの ASP.NET からアプリの構成を移行する方法について詳しくは、<xref:migration/proper-to-2x/index#store-configurations> をご覧ください。

## <a name="default-configuration"></a>既定の構成

ASP.NET Core の [dotnet new](/dotnet/core/tools/dotnet-new) テンプレートに基づく Web アプリは、ホストの構築時に <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> を呼び出します。 `CreateDefaultBuilder` により、次の順序でアプリの既定の構成が提供されます。

[Web ホスト](xref:fundamentals/host/web-host)を使用するアプリには、以下が適用されます。 [汎用ホスト](xref:fundamentals/host/generic-host)を使用する場合の既定の構成の詳細については、[このトピックの最新バージョン](xref:fundamentals/configuration/index)を参照してください。

* ホストの構成は、次から提供されます。
  * [環境変数構成プロバイダー](#environment-variables-configuration-provider)を使用する、プレフィックス `ASPNETCORE_` (`ASPNETCORE_ENVIRONMENT` など) が付いた環境変数。 構成のキーと値のペアが読み込まれるときに、プレフィックス (`ASPNETCORE_`) は削除されます。
  * [コマンドライン構成プロバイダー](#command-line-configuration-provider)を使用するコマンドライン引数。
* アプリの構成は、次から提供されます。
  * [ファイル構成プロバイダー](#file-configuration-provider)を使用する *appsettings.json* 。
  * [ファイル構成プロバイダー](#file-configuration-provider)を使用する *appsettings.{Environment}.json*。
  * エントリ アセンブリを使用して `Development` 環境でアプリが実行される場合に使用される[ユーザー シークレット](xref:security/app-secrets)。
  * [環境変数構成プロバイダー](#environment-variables-configuration-provider)を使用する環境変数。
  * [コマンドライン構成プロバイダー](#command-line-configuration-provider)を使用するコマンドライン引数。

## <a name="security"></a>セキュリティ

機密性の高い構成データをセキュリティで保護するには、次の方法を採用します。

* 構成プロバイダーのコードやプレーンテキストの構成ファイルには、パスワードなどの機密データを格納しないでください。
* 開発環境やテスト環境では運用シークレットを使用しないでください。
* プロジェクトの外部にシークレットを指定してください。そうすれば、誤ってリソース コード リポジトリにコミットされることはありません。

詳細については、次のトピックを参照してください。

* <xref:fundamentals/environments>
* <xref:security/app-secrets>:ここには、環境変数を使用して機密データを格納する際のアドバイスが記載されています。 Secret Manager ツールでは、ファイル構成プロバイダーを使用して、ユーザー シークレットがローカル システム上の JSON ファイルに格納されます。 ファイル構成プロバイダーについては、このトピックの後半で説明します。

[Azure Key Vault](https://azure.microsoft.com/services/key-vault/) では、ASP.NET Core アプリのアプリのシークレットが安全に保存されます。 詳細については、「<xref:security/key-vault-configuration>」を参照してください。

## <a name="hierarchical-configuration-data"></a>階層的な構成データ

構成 API は、構成キー内の区切り記号を使用して階層データをフラット化することにより、階層的な構成データを管理することができます。

次の JSON ファイルでは、2 つのセクションの構造化階層に 4 つのキーが存在します。

```json
{
  "section0": {
    "key0": "value",
    "key1": "value"
  },
  "section1": {
    "key0": "value",
    "key1": "value"
  }
}
```

ファイルが構成に読み込まれると、構成ソースの元の階層データ構造を維持するために、一意なキーが作成されます。 セクションとキーは、元の構造を維持するために、コロン (`:`) を使用してフラット化されます。

* section0:key0
* section0:key1
* section1:key0
* section1:key1

<xref:Microsoft.Extensions.Configuration.ConfigurationSection.GetSection*> メソッドと <xref:Microsoft.Extensions.Configuration.IConfiguration.GetChildren*> メソッドを使用して、構成データ内のセクションとセクションの子を分離することができます。 これらのメソッドについては、後ほど「[GetSection、GetChildren、Exists](#getsection-getchildren-and-exists)」で説明します。

## <a name="conventions"></a>規約

### <a name="sources-and-providers"></a>ソースとプロバイダー

アプリの起動時に、各構成プロバイダーが指定されている順序で構成ソースが読み取られます。

変更の検出を実装する構成プロバイダーは、基になる設定が変更された場合に構成を再読み込みする機能を備えています。 たとえば、ファイル構成プロバイダー (このトピックで後から説明します) と[Azure Key Vault 構成プロバイダー](xref:security/key-vault-configuration)では、変更の検出を実装します。

<xref:Microsoft.Extensions.Configuration.IConfiguration> は、アプリの[依存関係の挿入 (DI)](xref:fundamentals/dependency-injection) コンテナーで使用できます。 <xref:Microsoft.Extensions.Configuration.IConfiguration> を Razor Pages <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> または MVC <xref:Microsoft.AspNetCore.Mvc.Controller> に挿入して、クラスの構成を取得することができます。

次の例では、構成値にアクセスするために `_config` フィールドが使用されています。

```csharp
public class IndexModel : PageModel
{
    private readonly IConfiguration _config;

    public IndexModel(IConfiguration config)
    {
        _config = config;
    }
}
```

```csharp
public class HomeController : Controller
{
    private readonly IConfiguration _config;

    public HomeController(IConfiguration config)
    {
        _config = config;
    }
}
```

構成プロバイダーでは DI を使用できません。ホストによって構成プロバイダーが設定されている場合、DI を使用できないためです。

### <a name="keys"></a>キー

構成キーでは、次の規則が採用されます。

* キーでは、大文字と小文字は区別されません。 たとえば、`ConnectionString` と `connectionstring` は同等のキーとして扱われます。
* 同じキーに対する値が、同じまたは別の構成プロバイダーによって設定された場合、最後にキーに設定された値が使用される値となります。 重複する JSON キーの詳細については、「[こちらの GitHub のイシュー](https://github.com/dotnet/extensions/issues/2381)」を参照してください。
* 階層キー
  * 構成 API 内では、すべてのプラットフォームでコロン (`:`) の区切りが機能します。
  * 環境変数内では、コロン区切りがすべてのプラットフォームでは機能しない場合があります。 二重のアンダースコア (`__`) はすべてのプラットフォームでサポートされ、コロンに自動的に変換されます。
  * Azure Key Vault では、階層キーは区切り記号として `--` (2 つのダッシュ) を使用します。 コードを指定して、アプリの構成にシークレットが読み込まれるときにダッシュをコロンで置き換えます。
* <xref:Microsoft.Extensions.Configuration.ConfigurationBinder> は、構成キーで配列インデックスを使用して、オブジェクトに対する配列のバインドをサポートしています。 配列のバインドについては、「[配列をクラスにバインドする](#bind-an-array-to-a-class)」セクションで説明します。

### <a name="values"></a>値

構成値では、次の規則が採用されます。

* 値は文字列です。
* Null 値を構成に格納したり、オブジェクトにバインドしたりすることはできません。

## <a name="providers"></a>プロバイダー

ASP.NET Core アプリで使用できる構成プロバイダーを次の表に示します。

| プロバイダー | &hellip; から構成を提供します。 |
| -------- | ----------------------------------- |
| [Azure Key Vault 構成プロバイダー](xref:security/key-vault-configuration) ("*セキュリティ*" トピック) | Azure Key Vault |
| [Azure App Configuration プロバイダー](/azure/azure-app-configuration/quickstart-aspnet-core-app) (Azure のドキュメント) | Azure App Configuration |
| [コマンド ライン構成プロバイダー](#command-line-configuration-provider) | コマンド ライン パラメーター |
| [カスタム構成プロバイダー](#custom-configuration-provider) | カスタム ソース |
| [環境変数構成プロバイダー](#environment-variables-configuration-provider) | 環境変数 |
| [ファイル構成プロバイダー](#file-configuration-provider) | ファイル (INI、JSON、XML) |
| [ファイルごとのキーの構成プロバイダー](#key-per-file-configuration-provider) | ディレクトリ ファイル |
| [メモリ構成プロバイダー](#memory-configuration-provider) | メモリ内コレクション |
| [ユーザー シークレット](xref:security/app-secrets) ("*セキュリティ*" トピック) | ユーザー プロファイル ディレクトリ内のファイル |

アプリの起動時に各構成プロバイダーが指定されている順序で構成ソースが読み取られます。 このトピックで説明する構成プロバイダーは、それらをコードで配置する順ではなく、アルファベット順で説明します。 アプリで必要とされる、基になる構成ソースの優先順位に合わせるために、コード内で構成プロバイダーを並べ替えます。

一般的な一連の構成プロバイダーは次のとおりです。

1. ファイル ( *appsettings.json* 、*appsettings.{Environment}.json*。ここで、`{Environment}` はアプリの現在のホスト環境です)
1. [Azure Key Vault](xref:security/key-vault-configuration)
1. [ユーザー シークレット](xref:security/app-secrets) (開発環境のみ)
1. 環境変数
1. コマンド ライン引数

コマンド ライン引数が他のプロバイダーによって設定された構成をオーバーライドできるようにするために、コマンド ラインの構成プロバイダーを一連のプロバイダーの最後に配置するのは、一般的な方法です。

この一連のプロバイダーは、`CreateDefaultBuilder` を使用して新しいホスト ビルダーを初期化するときに使用されます。 詳細については、「[既定の構成](#default-configuration)」セクションを参照してください。

## <a name="configure-the-host-builder-with-useconfiguration"></a>UseConfiguration を使用してホスト ビルダーを構成する

ホスト ビルダーを構成するには、構成を使用するホスト ビルダー上で <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseConfiguration*> を呼び出します。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args)
{
    var dict = new Dictionary<string, string>
    {
        {"MemoryCollectionKey1", "value1"},
        {"MemoryCollectionKey2", "value2"}
    };

    var config = new ConfigurationBuilder()
        .AddInMemoryCollection(dict)
        .Build();

    return WebHost.CreateDefaultBuilder(args)
        .UseConfiguration(config)
        .UseStartup<Startup>();
}
```

## <a name="configureappconfiguration"></a>ConfigureAppConfiguration

ホストをビルドするときに `ConfigureAppConfiguration` を呼び出し、`CreateDefaultBuilder` によって自動的に追加されるものに加え、アプリの構成プロバイダーを指定します。

[!code-csharp[](index/samples/2.x/ConfigurationSample/Program.cs?name=snippet_Program&highlight=20)]

### <a name="override-previous-configuration-with-command-line-arguments"></a>前の構成をコマンドライン引数でオーバーライドする

コマンドライン引数でオーバーライドできるアプリ構成を指定するには、最後に `AddCommandLine` を呼び出します。

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    // Call other providers here
    config.AddCommandLine(args);
})
```

### <a name="remove-providers-added-by-createdefaultbuilder"></a>CreateDefaultBuilder によって追加されたプロバイダーの削除

`CreateDefaultBuilder` によって追加されたプロバイダーを削除するには、最初に [IConfigurationBuilder.Sources](xref:Microsoft.Extensions.Configuration.IConfigurationBuilder.Sources) の [クリア](/dotnet/api/system.collections.generic.icollection-1.clear) を呼び出します。

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.Sources.Clear();
    // Add providers here
})
```

### <a name="consume-configuration-during-app-startup"></a>アプリの起動時に構成を使用する

`ConfigureAppConfiguration` のアプリに指定した構成は、`Startup.ConfigureServices` などのアプリの起動中に使用できます。 詳細については、「[起動中に構成にアクセスする](#access-configuration-during-startup)」のセクションを参照してください。

## <a name="command-line-configuration-provider"></a>コマンド ライン構成プロバイダー

<xref:Microsoft.Extensions.Configuration.CommandLine.CommandLineConfigurationProvider> では、実行時にコマンドライン引数のキーと値のペアから構成が読み込まれます。

コマンド ライン構成をアクティブにするために、<xref:Microsoft.Extensions.Configuration.CommandLineConfigurationExtensions.AddCommandLine*> 拡張メソッドが <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> のインスタンスで呼び出されます。

`CreateDefaultBuilder(string [])` が呼び出されると、`AddCommandLine` が自動的に呼び出されます。 詳細については、「[既定の構成](#default-configuration)」セクションを参照してください。

`CreateDefaultBuilder` では次のものも読み込まれます。

* *appsettings.json* および *appsettings.{Environment}.json* ファイルからの省略可能な構成。
* 開発環境での[ユーザー シークレット](xref:security/app-secrets)。
* 環境変数。

`CreateDefaultBuilder` はコマンド ライン構成プロバイダーを最後に追加します。 実行時に渡されるコマンド ライン引数によって、他のプロバイダーによって設定された構成がオーバーライドされます。

`CreateDefaultBuilder` は、ホストが作成されるときに機能します。 そのため、`CreateDefaultBuilder` によってアクティブ化されるコマンド ライン構成によって、ホストの構成方法に影響を与えることができます。

ASP.NET Core テンプレートに基づくアプリの場合、`AddCommandLine` は `CreateDefaultBuilder` によって既に呼び出されています。 さらに構成プロバイダーを追加し、コマンドライン引数を使用してそれらのプロバイダーの構成をオーバーライドする機能を維持するには、アプリの追加プロバイダーを `ConfigureAppConfiguration` で呼び出し、最後に `AddCommandLine` を呼び出します。

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    // Call other providers here
    config.AddCommandLine(args);
})
```

**例**

サンプル アプリでは、静的な簡易メソッド `CreateDefaultBuilder` を利用してホストをビルドします。これには <xref:Microsoft.Extensions.Configuration.CommandLineConfigurationExtensions.AddCommandLine*> の呼び出しが含まれます。

1. プロジェクトのディレクトリでコマンド プロンプトを開きます。
1. `dotnet run` コマンドにコマンドライン引数を指定します (`dotnet run CommandLineKey=CommandLineValue`)。
1. アプリを実行したら、アプリに対して `http://localhost:5000` でブラウザーを開きます。
1. 出力に、`dotnet run` に提供される構成のコマンド ライン引数のキーと値のペアが含まれていることを観察します。

### <a name="arguments"></a>引数

値は等号 (`=`) の後に続ける必要があります。または、値をスペースの後に続ける場合は、キーにプレフィックス (`--`または`/`) を付ける必要があります。 等号 (`CommandLineKey=` など) が使用されている場合、値は必要ありません。

| キーのプレフィックス               | 例                                                |
| ------------------------ | ------------------------------------------------------ |
| プレフィックスなし                | `CommandLineKey1=value1`                               |
| 2 つのダッシュ (`--`)        | `--CommandLineKey2=value2`, `--CommandLineKey2 value2` |
| スラッシュ (`/`)      | `/CommandLineKey3=value3`, `/CommandLineKey3 value3`   |

同じコマンド内のコマンド ライン引数で、等号を使用するキーと値のペアと、スペースを使用するキーと値のペアを混在させないでください。

コマンドの例:

```dotnetcli
dotnet run CommandLineKey1=value1 --CommandLineKey2=value2 /CommandLineKey3=value3
dotnet run --CommandLineKey1 value1 /CommandLineKey2 value2
dotnet run CommandLineKey1= CommandLineKey2=value2
```

### <a name="switch-mappings"></a>スイッチ マッピング

スイッチ マッピングでは、キー名の交換ロジックが許可されます。 <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> で構成を手動でビルドするときに、<xref:Microsoft.Extensions.Configuration.CommandLineConfigurationExtensions.AddCommandLine*> メソッドにスイッチ置換のディクショナリを指定します。

スイッチ マッピング ディクショナリが使用されている場合、そのディレクトリで、コマンドライン引数によって指定されたキーと一致するキーが確認されます。 コマンド ライン キーがディクショナリで見つかった場合は、アプリの構成にキーと値のペアを設定するためにディクショナリの値 (キー交換) が返されます。 スイッチ マッピングは、単一のダッシュ (`-`) が前に付いたすべてのコマンドライン キーに必要です。

スイッチ マッピング ディクショナリ キーの規則:

* スイッチはダッシュ (`-`) または二重ダッシュ (`--`) で開始する必要があります。
* スイッチ マッピング ディクショナリに重複キーを含めることはできません。

スイッチ マッピング ディクショナリを作成します。 次の例では、2 つのスイッチ マッピングが作成されます。

```csharp
public static readonly Dictionary<string, string> _switchMappings = 
    new Dictionary<string, string>
    {
        { "-CLKey1", "CommandLineKey1" },
        { "-CLKey2", "CommandLineKey2" }
    };
```

ホストが構築されたら、スイッチ マッピング ディクショナリを使用して `AddCommandLine` を呼び出します。

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddCommandLine(args, _switchMappings);
})
```

スイッチ マッピングを使用するアプリでは、`CreateDefaultBuilder` への呼び出しで引数を渡すことはできません。 `CreateDefaultBuilder` メソッドの `AddCommandLine` の呼び出しにはマップされたスイッチが含まれないため、スイッチ マッピング ディクショナリを `CreateDefaultBuilder` に渡す方法はありません。 ソリューションでは `CreateDefaultBuilder` に引数を渡す代わりに、`ConfigurationBuilder` メソッドの `AddCommandLine` メソッドに、引数とスイッチ マッピング ディクショナリの両方を処理させることができます。

スイッチ マッピング ディクショナリが作成されると、以下の表に示すデータが含まれます。

| Key       | 値             |
| --------- | ----------------- |
| `-CLKey1` | `CommandLineKey1` |
| `-CLKey2` | `CommandLineKey2` |

アプリの起動時にスイッチ マッピングされたキーを使用する場合、構成は、ディクショナリによって指定されたキーでの構成値を受け取ります。

```dotnetcli
dotnet run -CLKey1=value1 -CLKey2=value2
```

上記のコマンドを実行すると、次の表に示す値が構成に含まれます。

| Key               | [値]    |
| ----------------- | -------- |
| `CommandLineKey1` | `value1` |
| `CommandLineKey2` | `value2` |

## <a name="environment-variables-configuration-provider"></a>環境変数構成プロバイダー

<xref:Microsoft.Extensions.Configuration.EnvironmentVariables.EnvironmentVariablesConfigurationProvider> では、実行時に環境変数のキーと値のペアから構成が読み込まれます。

環境変数の構成をアクティブにするには、<xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> のインスタンスの <xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*> 拡張メソッドを呼び出します。

[!INCLUDE[](~/includes/environmentVarableColon.md)]

[Azure App Service](https://azure.microsoft.com/services/app-service/) を使用すると、環境変数構成プロバイダーを使用してアプリの構成をオーバーライドすることができる環境変数を、Azure portal で設定できます。 詳細については、「[Azure アプリ: Azure Portal を使用してアプリの構成をオーバーライドする](xref:host-and-deploy/azure-apps/index#override-app-configuration-using-the-azure-portal)」を参照してください。

新しいホスト ビルダーが [Web ホスト](xref:fundamentals/host/web-host)で初期化され、`CreateDefaultBuilder` が呼び出されると、`AddEnvironmentVariables` が使用され、[ホスト構成](#host-versus-app-configuration)の `ASPNETCORE_` で始まる環境変数が読み込まれます。 詳細については、「[既定の構成](#default-configuration)」セクションを参照してください。

`CreateDefaultBuilder` では次のものも読み込まれます。

* プレフィックスなしの `AddEnvironmentVariables` 呼び出しによる、プレフィックスの付いていない環境変数からのアプリの構成。
* *appsettings.json* および *appsettings.{Environment}.json* ファイルからの省略可能な構成。
* 開発環境での[ユーザー シークレット](xref:security/app-secrets)。
* コマンド ライン引数。

ユーザー シークレットと *appsettings* ファイルから構成が設定された後に、環境変数構成プロバイダーが呼び出されます。 この位置でプロバイダーを呼び出すことにより、実行時に読み込まれた環境変数が、ユーザー シークレットと *appsettings* ファイルによって設定された構成をオーバーライドすることができます。

追加の環境変数からアプリの構成を指定するには、`ConfigureAppConfiguration` のアプリの追加プロバイダーを呼び出し、次のプレフィックスを含む `AddEnvironmentVariables` を呼び出します。

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddEnvironmentVariables(prefix: "PREFIX_");
})
```

`AddEnvironmentVariables` を最後に呼び出して、指定されたプレフィックスを持つ環境変数が他のプロバイダーの値をオーバーライドできるようにします。

**例**

サンプル アプリでは、静的な簡易メソッド `CreateDefaultBuilder` を利用してホストをビルドします。これには `AddEnvironmentVariables` の呼び出しが含まれます。

1. サンプル アプリを実行します。 アプリに対して `http://localhost:5000` でブラウザーを開きます。
1. 出力に、環境変数 `ENVIRONMENT` のキーと値のペアが含まれていることを観察します。 値には、アプリを実行している環境が反映されます (ローカルで実行している場合は通常 `Development`)。

アプリによってレンダリングされる環境変数の一覧を短く保つために、アプリでは環境変数がフィルター処理されます。 サンプル アプリの *Pages/Index.cshtml.cs* ファイルを参照してください。

アプリで使用できるすべての環境変数を公開する場合は、*Pages/Index.cshtml.cs* の `FilteredConfiguration` を次のように変更します。

```csharp
FilteredConfiguration = _config.AsEnumerable();
```

### <a name="prefixes"></a>プレフィックス

アプリの構成に読み込まれる環境変数は、`AddEnvironmentVariables` メソッドにプレフィックスを指定すると、フィルター処理されます。 たとえば、プレフィックス `CUSTOM_` で環境変数をフィルター処理するには、構成プロバイダーにプレフィックスを指定します。

```csharp
var config = new ConfigurationBuilder()
    .AddEnvironmentVariables("CUSTOM_")
    .Build();
```

構成のキーと値のペアが作成されるときに、プレフィックスは削除されます。

ホストビルダーが作成されると、環境変数によってホスト構成が指定されます。 これらの環境変数に使用されるプレフィックスの詳細については、「[既定の構成](#default-configuration)」セクションを参照してください。

**接続文字列のプレフィックス**

構成 API には、アプリの環境に向けた Azure の接続文字列の構成に関係する、4 つの接続文字列環境変数のための特別な処理規則があります。 表に示されるプレフィックスを含む環境変数は、`AddEnvironmentVariables` にプレフィックスが指定されていない場合、アプリに読み込まれます。

| 接続文字列のプレフィックス | プロバイダー |
| ------------------------ | -------- |
| `CUSTOMCONNSTR_` | カスタム プロバイダー |
| `MYSQLCONNSTR_` | [MySQL](https://www.mysql.com/) |
| `SQLAZURECONNSTR_` | [Azure SQL Database](https://azure.microsoft.com/services/sql-database/) |
| `SQLCONNSTR_` | [SQL Server](https://www.microsoft.com/sql-server/) |

表に示す 4 つのプレフィックスのいずれかを使用して、環境変数が検出され構成に読み込まれた場合:

* 環境変数のプレフィックスを削除し、構成キーのセクション (`ConnectionStrings`) を追加することによって、構成キーが作成されます。
* データベースの接続プロバイダーを表す新しい構成のキーと値のペアが作成されます (示されたプロバイダーを含まない `CUSTOMCONNSTR_` を除く)。

| 環境変数キー | 変換された構成キー | プロバイダーの構成エントリ                                                    |
| ------------------------ | --------------------------- | ------------------------------------------------------------------------------- |
| `CUSTOMCONNSTR_{KEY} `   | `ConnectionStrings:{KEY}`   | 構成エントリは作成されません。                                                |
| `MYSQLCONNSTR_{KEY}`     | `ConnectionStrings:{KEY}`   | キー: `ConnectionStrings:{KEY}_ProviderName`:<br>値: `MySql.Data.MySqlClient` |
| `SQLAZURECONNSTR_{KEY}`  | `ConnectionStrings:{KEY}`   | キー: `ConnectionStrings:{KEY}_ProviderName`:<br>値: `System.Data.SqlClient`  |
| `SQLCONNSTR_{KEY}`       | `ConnectionStrings:{KEY}`   | キー: `ConnectionStrings:{KEY}_ProviderName`:<br>値: `System.Data.SqlClient`  |

**例**

サーバー上にカスタム接続文字列環境変数が作成されます。

* 名前: `CUSTOMCONNSTR_ReleaseDB`
* 値: `Data Source=ReleaseSQLServer;Initial Catalog=MyReleaseDB;Integrated Security=True`

`IConfiguration` が挿入され、`_config` という名前のフィールドに割り当てられた場合は、次の値を読み取ります。

```csharp
_config["ConnectionStrings:ReleaseDB"]
```

## <a name="file-configuration-provider"></a>ファイル構成プロバイダー

<xref:Microsoft.Extensions.Configuration.FileConfigurationProvider> は、ファイル システムから構成を読み込むための基本クラスです。 次の構成プロバイダーは、特定のファイルの種類専用です。

* [INI 構成プロバイダー](#ini-configuration-provider)
* [JSON 構成プロバイダー](#json-configuration-provider)
* [XML 構成プロバイダー](#xml-configuration-provider)

### <a name="ini-configuration-provider"></a>INI 構成プロバイダー

<xref:Microsoft.Extensions.Configuration.Ini.IniConfigurationProvider> では、実行時に INI ファイルのキーと値のペアから構成が読み込まれます。

INI ファイルの構成をアクティブにするには、<xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> のインスタンスの <xref:Microsoft.Extensions.Configuration.IniConfigurationExtensions.AddIniFile*> 拡張メソッドを呼び出します。

INI ファイルの構成では、セクションの区切り記号としてコロンを使用できます。

オーバーロードによって次のものを指定できます。

* ファイルを省略可能かどうか。
* ファイルが変更された場合に構成を再度読み込むかどうか。
* ファイルにアクセスするために <xref:Microsoft.Extensions.FileProviders.IFileProvider> が使用されます。

ホストをビルドするときに `ConfigureAppConfiguration` を呼び出して、アプリの構成を指定します。

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddIniFile(
        "config.ini", optional: true, reloadOnChange: true);
})
```

INI 構成ファイルの汎用的な例:

```ini
[section0]
key0=value
key1=value

[section1]
subsection:key=value

[section2:subsection0]
key=value

[section2:subsection1]
key=value
```

前の構成ファイルでは、`value` を使用して次のキーが読み込まれます。

* section0:key0
* section0:key1
* section1:subsection:key
* section2:subsection0:key
* section2:subsection1:key

### <a name="json-configuration-provider"></a>JSON 構成プロバイダー

<xref:Microsoft.Extensions.Configuration.Json.JsonConfigurationProvider> では、実行時に JSON ファイルのキーと値のペアから構成が読み込まれます。

JSON ファイルの構成をアクティブにするには、<xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> のインスタンスの <xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*> 拡張メソッドを呼び出します。

オーバーロードによって次のものを指定できます。

* ファイルを省略可能かどうか。
* ファイルが変更された場合に構成を再度読み込むかどうか。
* ファイルにアクセスするために <xref:Microsoft.Extensions.FileProviders.IFileProvider> が使用されます。

`CreateDefaultBuilder` を使用して新しいホスト ビルダーを初期化すると、`AddJsonFile` が自動的に 2 回呼び出されます。 このメソッドは、次から構成を読み込むために呼び出されます。

* *appsettings.json* : このファイルは最初に読み取られます。 ファイルの環境バージョンを使用すると、 *appsettings.json* ファイルによって指定された値をオーバーライドできます。
* *appsettings.{Environment}.json*:ファイルの環境バージョンは、[IHostingEnvironment.EnvironmentName](xref:Microsoft.Extensions.Hosting.IHostingEnvironment.EnvironmentName*)に基づいて読み込まれます。

詳細については、「[既定の構成](#default-configuration)」セクションを参照してください。

`CreateDefaultBuilder` では次のものも読み込まれます。

* 環境変数。
* 開発環境での[ユーザー シークレット](xref:security/app-secrets)。
* コマンド ライン引数。

JSON 構成プロバイダーが最初に確立されます。 このため、ユーザー シークレット、環境変数、およびコマンド ライン引数によって、*appsettings* ファイルによって設定された構成がオーバーライドされます。

ホストのビルド時に `ConfigureAppConfiguration` を呼び出して、 *appsettings.json* および *appsettings.{Environment}.json* 以外のファイルにアプリの構成を指定します。

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddJsonFile(
        "config.json", optional: true, reloadOnChange: true);
})
```

**例**

サンプル アプリでは、静的な簡易メソッド `CreateDefaultBuilder` を利用してホストをビルドします。これには `AddJsonFile` の 2 回の呼び出しが含まれます。

* `AddJsonFile` の最初の呼び出しを行うと、 *appsettings.json* から構成が読み込まれます。

  [!code-json[](index/samples/2.x/ConfigurationSample/appsettings.json)]

* `AddJsonFile` の 2 回目の呼び出しでは、*appsettings.{Environment}.json* から構成を読み込みます。 サンプル アプリの *appsettings.Development.json* では、次のファイルが読み込まれます。

  [!code-json[](index/samples/2.x/ConfigurationSample/appsettings.Development.json)]

1. サンプル アプリを実行します。 アプリに対して `http://localhost:5000` でブラウザーを開きます。
1. 出力には、アプリの環境に基づく構成のキーと値のペアが含まれています。 開発環境でアプリを実行する場合、キー `Logging:LogLevel:Default` のログ レベルは `Debug` です。
1. 運用環境でもう一度サンプル アプリを実行します。
   1. *Properties/launchSettings.json* ファイルを開きます。
   1. `ConfigurationSample` プロファイルで、`ASPNETCORE_ENVIRONMENT` 環境変数の値を `Production` に変更します。
   1. ファイルを保存し、コマンド シェルで `dotnet run` を使用してアプリを実行します。
1. *appsettings.Development.json* 内の設定によって、 *appsettings.json* 内の設定はオーバーライドされなくなりました。 キー `Logging:LogLevel:Default` のログ レベルは `Warning` です。

### <a name="xml-configuration-provider"></a>XML 構成プロバイダー

<xref:Microsoft.Extensions.Configuration.Xml.XmlConfigurationProvider> では、実行時に XML ファイルのキーと値のペアから構成が読み込まれます。

XML ファイルの構成をアクティブにするには、<xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> のインスタンスの <xref:Microsoft.Extensions.Configuration.XmlConfigurationExtensions.AddXmlFile*> 拡張メソッドを呼び出します。

オーバーロードによって次のものを指定できます。

* ファイルを省略可能かどうか。
* ファイルが変更された場合に構成を再度読み込むかどうか。
* ファイルにアクセスするために <xref:Microsoft.Extensions.FileProviders.IFileProvider> が使用されます。

構成ファイルのルート ノードは、構成のキーと値のペアを作成するときには無視されます。 ファイル内でドキュメント型定義 (DTD) または名前空間を指定しないでください。

ホストをビルドするときに `ConfigureAppConfiguration` を呼び出して、アプリの構成を指定します。

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddXmlFile(
        "config.xml", optional: true, reloadOnChange: true);
})
```

XML 構成ファイルでは、繰り返しのセクション用に個別の要素名を使用できます。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <section0>
    <key0>value</key0>
    <key1>value</key1>
  </section0>
  <section1>
    <key0>value</key0>
    <key1>value</key1>
  </section1>
</configuration>
```

前の構成ファイルでは、`value` を使用して次のキーが読み込まれます。

* section0:key0
* section0:key1
* section1:key0
* section1:key1

同じ要素名を使用する要素の繰り返しは、要素を区別するために `name` 属性を使用する場合に機能します。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <section name="section0">
    <key name="key0">value</key>
    <key name="key1">value</key>
  </section>
  <section name="section1">
    <key name="key0">value</key>
    <key name="key1">value</key>
  </section>
</configuration>
```

前の構成ファイルでは、`value` を使用して次のキーが読み込まれます。

* section:section0:key:key0
* section:section0:key:key1
* section:section1:key:key0
* section:section1:key:key1

値を指定するために属性を使用できます。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <key attribute="value" />
  <section>
    <key attribute="value" />
  </section>
</configuration>
```

前の構成ファイルでは、`value` を使用して次のキーが読み込まれます。

* key:attribute
* section:key:attribute

## <a name="key-per-file-configuration-provider"></a>ファイルごとのキーの構成プロバイダー

<xref:Microsoft.Extensions.Configuration.KeyPerFile.KeyPerFileConfigurationProvider> では、構成のキーと値のペアとしてディレクトリのファイルが使用されます。 キーはファイル名です。 値にはファイルのコンテンツが含まれます。 ファイルごとのキーの構成プロバイダーは、Docker ホスティングのシナリオで使用されます。

ファイルごとのキーの構成をアクティブにするには、<xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> のインスタンスの <xref:Microsoft.Extensions.Configuration.KeyPerFileConfigurationBuilderExtensions.AddKeyPerFile*> 拡張メソッドを呼び出します。 ファイルに対する `directoryPath` は、絶対パスである必要があります。

オーバーロードによって次のものを指定できます。

* ソースを構成する `Action<KeyPerFileConfigurationSource>` デリゲート。
* ディレクトリを省略可能かどうか、またディレクトリへのパス。

アンダースコア 2 つ (`__`) は、ファイル名で構成キーの区切り記号として使用されます。 たとえば、ファイル名 `Logging__LogLevel__System` では、構成キー `Logging:LogLevel:System` が生成されます。

ホストをビルドするときに `ConfigureAppConfiguration` を呼び出して、アプリの構成を指定します。

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    var path = Path.Combine(
        Directory.GetCurrentDirectory(), "path/to/files");
    config.AddKeyPerFile(directoryPath: path, optional: true);
})
```

## <a name="memory-configuration-provider"></a>メモリ構成プロバイダー

<xref:Microsoft.Extensions.Configuration.Memory.MemoryConfigurationProvider> では、構成のキーと値のペアとして、メモリ内コレクションが使用されます。

メモリ内コレクションの構成をアクティブにするには、<xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> のインスタンスの <xref:Microsoft.Extensions.Configuration.MemoryConfigurationBuilderExtensions.AddInMemoryCollection*> 拡張メソッドを呼び出します。

構成プロバイダーは、`IEnumerable<KeyValuePair<String,String>>` を使用して初期化できます。

ホストをビルドするときに `ConfigureAppConfiguration` を呼び出して、アプリの構成を指定します。

次の例では、構成ディクショナリが作成されます。

```csharp
public static readonly Dictionary<string, string> _dict = 
    new Dictionary<string, string>
    {
        {"MemoryCollectionKey1", "value1"},
        {"MemoryCollectionKey2", "value2"}
    };
```

ディクショナリは、構成を指定するために `AddInMemoryCollection` の呼び出しと共に使用されます。

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddInMemoryCollection(_dict);
})
```

## <a name="getvalue"></a>GetValue

[`ConfigurationBinder.GetValue<T>`](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.GetValue*) 指定したキーを持つ構成から単一の値を抽出し、指定したコレクション以外の型に変換します。 オーバーロードは、既定値を受け入れます。

次のような例です。

* キー `NumberKey` の文字列値を構成から抽出します。 `NumberKey` が構成キーに見つからない場合、既定値の `99` が使用されます。
* 値の型は `int` です。
* `NumberConfig` プロパティの値をページで使用するために格納します。

```csharp
public class IndexModel : PageModel
{
    public IndexModel(IConfiguration config)
    {
        _config = config;
    }

    public int NumberConfig { get; private set; }

    public void OnGet()
    {
        NumberConfig = _config.GetValue<int>("NumberKey", 99);
    }
}
```

## <a name="getsection-getchildren-and-exists"></a>GetSection、GetChildren、Exists

以下の例では、次の JSON ファイルについて考えます。 4 つのキーが 2 つのセクションに含まれています。そのうちの 1 つには、一組のサブセクションが含まれています。

```json
{
  "section0": {
    "key0": "value",
    "key1": "value"
  },
  "section1": {
    "key0": "value",
    "key1": "value"
  },
  "section2": {
    "subsection0" : {
      "key0": "value",
      "key1": "value"
    },
    "subsection1" : {
      "key0": "value",
      "key1": "value"
    }
  }
}
```

ファイルが構成に読み取られると、次の一意の階層キーが作成され、構成値が保持されます。

* section0:key0
* section0:key1
* section1:key0
* section1:key1
* section2:subsection0:key0
* section2:subsection0:key1
* section2:subsection1:key0
* section2:subsection1:key1

### <a name="getsection"></a>GetSection

[IConfiguration.GetSection](xref:Microsoft.Extensions.Configuration.IConfiguration.GetSection*) では、指定したサブセクションのキーを持つ構成のサブセクションが抽出されます。

`section1` のキーと値のペアのみを含む <xref:Microsoft.Extensions.Configuration.IConfigurationSection> を返すには、`GetSection` を呼び出してセクション名を指定します。

```csharp
var configSection = _config.GetSection("section1");
```

`configSection` には値はなく、キーとパスのみが含まれます。

同様に、`section2:subsection0` のキーの値を取得するには、`GetSection` を呼び出してセクションのパスを指定します。

```csharp
var configSection = _config.GetSection("section2:subsection0");
```

`GetSection` が `null` を返すことはありません。 一致するセクションが見つからない場合は、空の `IConfigurationSection` が返されます。

`GetSection` で一致するセクションが返されると、<xref:Microsoft.Extensions.Configuration.IConfigurationSection.Value> は設定されません。 <xref:Microsoft.Extensions.Configuration.IConfigurationSection.Key> と <xref:Microsoft.Extensions.Configuration.IConfigurationSection.Path> はセクションが存在する場合に返されます。

### <a name="getchildren"></a>GetChildren

`section2` での [IConfiguration.GetChildren](xref:Microsoft.Extensions.Configuration.IConfiguration.GetChildren*) の呼び出しでは、次を含む `IEnumerable<IConfigurationSection>` が取得されます。

* `subsection0`
* `subsection1`

```csharp
var configSection = _config.GetSection("section2");

var children = configSection.GetChildren();
```

### <a name="exists"></a>既存

[ConfigurationExtensions.Exists](xref:Microsoft.Extensions.Configuration.ConfigurationExtensions.Exists*) を使用して、構成セクションが存在するかどうかを判断します。

```csharp
var sectionExists = _config.GetSection("section2:subsection2").Exists();
```

例のデータの場合、構成データに `section2:subsection2` セクションが存在しないため、`sectionExists` は `false` となります。

## <a name="bind-to-an-object-graph"></a>オブジェクト グラフにバインドする

<xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Bind*> では、POCO オブジェクト グラフ全体をバインドすることができます。 単純なオブジェクトをバインドする場合と同様に、パブリックな読み取り/書き込みプロパティのみがバインドされます。

サンプルには、オブジェクト グラフに `Metadata` クラスと `Actors` クラスが含まれる `TvShow` モデルが含まれます (*Models/TvShow.cs*)。

[!code-csharp[](index/samples/2.x/ConfigurationSample/Models/TvShow.cs?name=snippet1)]

サンプル アプリには、構成データを含む *tvshow.xml* ファイルが含まれます。

[!code-xml[](index/samples/2.x/ConfigurationSample/tvshow.xml)]

構成は、`Bind` メソッドを使用して、`TvShow` オブジェクト グラフ全体にバインドされます。 バインドされたインスタンスは、表示のためにプロパティに割り当てられます。

```csharp
var tvShow = new TvShow();
_config.GetSection("tvshow").Bind(tvShow);
TvShow = tvShow;
```

[`ConfigurationBinder.Get<T>`](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Get*) 指定された型をバインドして返します。 `Get<T>` は `Bind` を使用するよりも便利です。 次のコードは、上記の例で `Get<T>` を使用する方法を示します：

[!code-csharp[](index/samples/2.x/ConfigurationSample/Pages/Index.cshtml.cs?name=snippet_tvshow)]

## <a name="bind-an-array-to-a-class"></a>配列をクラスにバインドする

*サンプル アプリは、このセクションで説明する概念を示しています。*

<xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Bind*> は、構成キーで配列インデックスを使用して、オブジェクトに対する配列のバインドをサポートしています。 数値のキー セグメント (`:0:`、`:1:`、&hellip; `:{n}:`) を公開する配列形式は、すべて POCO クラスの配列にバインドできます。

> [!NOTE]
> バインドは慣例に従って指定されます。 カスタム構成プロバイダーが配列のバインドを実装する必要はありません。

**メモリ内配列の処理**

次の表に示す構成のキーと値について考えます。

| Key             | [値]  |
| :-------------: | :----: |
| 配列:エントリ:0 | value0 |
| 配列:エントリ:1 | value1 |
| 配列:エントリ:2 | value2 |
| 配列:エントリ:4 | value4 |
| 配列:エントリ:5 | value5 |

これらのキーと値は、メモリ構成プロバイダーを使用してサンプル アプリに読み込まれます。

[!code-csharp[](index/samples/2.x/ConfigurationSample/Program.cs?name=snippet_Program&highlight=5-12,22)]

配列は、インデックス &num;3 の値をスキップします。 構成バインダーは、null 値をバインドしたり、バインドされたオブジェクトに null エントリを作成したりすることはできません。このことは、この配列をオブジェクトにバインドした結果によって明らかになります。

サンプル アプリでは、バインドされた構成データを保持するために POCO クラスを使用できます。

[!code-csharp[](index/samples/2.x/ConfigurationSample/Models/ArrayExample.cs?name=snippet1)]

構成データはオブジェクトにバインドされます。

```csharp
var arrayExample = new ArrayExample();
_config.GetSection("array").Bind(arrayExample);
```

[`ConfigurationBinder.Get<T>`](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Get*) 構文を使用することもできます。これにより、コードがよりコンパクトになります：

[!code-csharp[](index/samples/2.x/ConfigurationSample/Pages/Index.cshtml.cs?name=snippet_array)]

バインドされたオブジェクト (`ArrayExample` のインスタンス) は、構成から配列データを受け取ります。

| `ArrayExample.Entries` インデックス | `ArrayExample.Entries` 値 |
| :--------------------------: | :--------------------------: |
| 0                            | value0                       |
| 1                            | value1                       |
| 2                            | value2                       |
| 3                            | value4                       |
| 4                            | value5                       |

バインドされたオブジェクトのインデックス &num;3 によって、`array:4` 構成キーの構成データと、その値 `value4` が保持されます。 配列を含む構成データがバインドされると、構成キーの配列のインデックスは、オブジェクトを作成するときに構成データを反復処理するためだけに使用されます。 構成データに null 値を保持することはできません。また、構成キーの配列が 1 つまたは複数のインデックスをスキップしても、バインドされたオブジェクトに null 値のエントリは作成されません。

インデックス &num;3 の不足している構成項目は、`ArrayExample` インスタンスにバインドする前に、構成で適切なキーと値のペアを生成する構成プロバイダーによって指定できます。 不足しているキーと値のペアを含む JSON 構成プロバイダーがサンプルに含まれる場合、`ArrayExample.Entries` は完全な構成の配列と一致します。

*missing_value.json*:

```json
{
  "array:entries:3": "value3"
}
```

`ConfigureAppConfiguration`の場合:

```csharp
config.AddJsonFile(
    "missing_value.json", optional: false, reloadOnChange: false);
```

表に示すキーと値のペアが構成に読み込まれます。

| Key             | 値  |
| :-------------: | :----: |
| array:entries:3 | value3 |

JSON 構成プロバイダーにインデックス &num;3 のエントリが含まれた後に `ArrayExample` クラスのインスタンスがバインドされる場合、`ArrayExample.Entries` 配列に値が含まれます。

| `ArrayExample.Entries` インデックス | `ArrayExample.Entries` 値 |
| :--------------------------: | :--------------------------: |
| 0                            | value0                       |
| 1                            | value1                       |
| 2                            | value2                       |
| 3                            | value3                       |
| 4                            | value4                       |
| 5                            | value5                       |

**JSON 配列の処理**

JSON ファイルに配列が含まれる場合、配列要素の構成キーは、0 から始まるセクションのインデックスで作成されます。 次の構成ファイルにおいて、`subsection` は配列です。

[!code-json[](index/samples/2.x/ConfigurationSample/json_array.json)]

JSON 構成プロバイダーは、次のキーと値のペアに構成データを読み取ります。

| Key                     | 値  |
| ----------------------- | :----: |
| json_array:key          | valueA |
| json_array:subsection:0 | valueB |
| json_array:subsection:1 | valueC |
| json_array:subsection:2 | valueD |

サンプル アプリでは、構成のキーと値のペアをバインドするために、次の POCO クラスを使用できます。

[!code-csharp[](index/samples/2.x/ConfigurationSample/Models/JsonArrayExample.cs?name=snippet1)]

バインド後、`JsonArrayExample.Key` は値 `valueA` を保持します。 サブセクションの値は、POCO 配列のプロパティ `Subsection` に格納されます。

| `JsonArrayExample.Subsection` インデックス | `JsonArrayExample.Subsection` 値 |
| :---------------------------------: | :---------------------------------: |
| 0                                   | valueB                              |
| 1                                   | valueC                              |
| 2                                   | valueD                              |

## <a name="custom-configuration-provider"></a>カスタム構成プロバイダー

サンプル アプリでは、[Entity Framework (EF)](/ef/core/) を使用してデータベースから構成のキーと値のペアを読み取る、基本的な構成プロバイダーを作成する方法を示します。

プロバイダーの特徴は次のとおりです。

* EF のメモリ内データベースは、デモンストレーションのために使用されます。 接続文字列を必要とするデータベースを使用するには、第 2 の `ConfigurationBuilder` を実装して、別の構成プロバイダーからの接続文字列を指定します。
* プロバイダーは、起動時に、構成にデータベース テーブルを読み取ります。 プロバイダーは、キー単位でデータベースを照会しません。
* 変更時に再度読み込む機能は実装されていません。このため、アプリの起動後にデータベースを更新しても、アプリの構成には影響がありません。

データベースに構成値を格納するための `EFConfigurationValue` エンティティを定義します。

*Models/EFConfigurationValue.cs*:

[!code-csharp[](index/samples/2.x/ConfigurationSample/Models/EFConfigurationValue.cs?name=snippet1)]

構成した値を格納し、その値にアクセスするための `EFConfigurationContext` を追加します。

*EFConfigurationProvider/EFConfigurationContext.cs*:

[!code-csharp[](index/samples/2.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationContext.cs?name=snippet1)]

<xref:Microsoft.Extensions.Configuration.IConfigurationSource> を実装するクラスを作成します。

*EFConfigurationProvider/EFConfigurationSource.cs*:

[!code-csharp[](index/samples/2.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationSource.cs?name=snippet1)]

<xref:Microsoft.Extensions.Configuration.ConfigurationProvider> から継承して、カスタム構成プロバイダーを作成します。 データベースが空だった場合、構成プロバイダーはこれを初期化します。

*EFConfigurationProvider/EFConfigurationProvider.cs*:

[!code-csharp[](index/samples/2.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationProvider.cs?name=snippet1)]

`AddEFConfiguration` 拡張メソッドを使用すると、`ConfigurationBuilder` に構成ソースを追加できます。

*Extensions/EntityFrameworkExtensions.cs*:

[!code-csharp[](index/samples/2.x/ConfigurationSample/Extensions/EntityFrameworkExtensions.cs?name=snippet1)]

次のコードでは、*Program.cs* でカスタムの `EFConfigurationProvider` を使用する方法を示します。

[!code-csharp[](index/samples/2.x/ConfigurationSample/Program.cs?name=snippet_Program&highlight=29-30)]

## <a name="access-configuration-during-startup"></a>起動中に構成にアクセスする

`Startup` コンストラクターに `IConfiguration` を挿入して、`Startup.ConfigureServices` で構成値にアクセスします。 `Startup.Configure` で構成にアクセスするには、メソッドに直接 `IConfiguration` を挿入するか、コンストラクターからのインスタンスを使用します。

```csharp
public class Startup
{
    private readonly IConfiguration _config;

    public Startup(IConfiguration config)
    {
        _config = config;
    }

    public void ConfigureServices(IServiceCollection services)
    {
        var value = _config["key"];
    }

    public void Configure(IApplicationBuilder app, IConfiguration config)
    {
        var value = config["key"];
    }
}
```

起動時の簡易メソッドを使用して構成にアクセスする例については、[アプリ起動時の簡易メソッド](xref:fundamentals/startup#convenience-methods)に関連する記事をご覧ください。

## <a name="access-configuration-in-a-razor-pages-page-or-mvc-view"></a>Razor Pages ページまたは MVC ビューで構成にアクセスする

Razor Pages ページまたは MVC ビューで構成設定にアクセスするには、[Microsoft.Extensions.Configuration](xref:Microsoft.Extensions.Configuration) 名前空間に [using ディレクティブ](xref:mvc/views/razor#using) ([C# リファレンス: using ディレクティブ](/dotnet/csharp/language-reference/keywords/using-directive)) を追加して、<xref:Microsoft.Extensions.Configuration.IConfiguration> をページまたはビューに挿入します。

Razor Pages ページで、次のようにします。

```cshtml
@page
@model IndexModel
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration

<!DOCTYPE html>
<html lang="en">
<head>
    <title>Index Page</title>
</head>
<body>
    <h1>Access configuration in a Razor Pages page</h1>
    <p>Configuration value for 'key': @Configuration["key"]</p>
</body>
</html>
```

MVC ビュー:

```cshtml
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration

<!DOCTYPE html>
<html lang="en">
<head>
    <title>Index View</title>
</head>
<body>
    <h1>Access configuration in an MVC view</h1>
    <p>Configuration value for 'key': @Configuration["key"]</p>
</body>
</html>
```

## <a name="add-configuration-from-an-external-assembly"></a>外部アセンブリから構成を追加する

<xref:Microsoft.AspNetCore.Hosting.IHostingStartup> の実装により、アプリの `Startup` クラスの外部にある外部アセンブリから、起動時に拡張機能をアプリに追加できるようになります。 詳細については、「<xref:fundamentals/configuration/platform-specific-configuration>」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:fundamentals/configuration/options>

::: moniker-end
