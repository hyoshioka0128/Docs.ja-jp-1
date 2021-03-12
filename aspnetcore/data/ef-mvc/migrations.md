---
title: チュートリアル パート 5 - Contoso University のサンプルに移行を適用する
description: Contoso University チュートリアル シリーズのパート 5。 ASP.NET Core MVC アプリでのデータ モデルの変更を管理するために EF Core の移行機能を使用します。
author: rick-anderson
ms.author: riande
ms.custom: contperf-fy21q2
ms.date: 11/13/2020
ms.topic: tutorial
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
uid: data/ef-mvc/migrations
ms.openlocfilehash: aebbc3f29b0356c7993abd83869ab21d3613bf61
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589349"
---
# <a name="tutorial-part-5-apply-migrations-to-the-contoso-university-sample"></a>チュートリアル: パート 5 - Contoso University のサンプルに移行を適用する

このチュートリアルでは、データ モデルの変更を管理するための EF Core の移行機能の使用を開始します。 チュートリアルの後半では、データ モデルを変更するときに移行を追加します。

このチュートリアルでは、次の作業を行いました。

> [!div class="checklist"]
> * 移行について学習する
> * 初期移行を作成する
> * Up および Down メソッドを確認する
> * データ モデルのスナップショットについて学習する
> * 移行を適用する

## <a name="prerequisites"></a>必須コンポーネント

* [並べ替え、フィルター処理、ページング](sort-filter-page.md)

## <a name="about-migrations"></a>移行について

新しいアプリケーションを開発して、データ モデルが頻繁に変更される場合、モデルが変更されるたびに、モデルはデータベースと同期されなくなります。 データベースが存在しない場合にデータベースを作成するように Entity Framework を構成して、これらのチュートリアルを開始しました。 そのため、データ モデルを変更する (エンティティ クラスを追加、削除、または変更するか、DbContext クラスを変更する) たびに、データベースを削除することができ、EF でモデルに一致する新しいデータベースを作成し、テスト データを使ってシードを設定します。

このメソッドは、実稼働環境にアプリケーションを展開するまで、データベースとデータ モデルの同期の維持がうまく機能します。 実稼働環境でアプリケーションを実行している場合、通常、保持する必要があるデータが保存され、新しい列の追加などの変更を加えるたびにすべてが失われないようにする必要があります。 EF Core の移行機能は、新しいデータベースを作成する代わりに、EF でデータベース スキーマを更新できるようにすることで、この問題を解決します。

移行の作業を行うには、**パッケージ マネージャー コンソール** (PMC) または CLI を使用できます。  このチュートリアルでは、CLI コマンドを使用する方法を示します。 PMC については、[このチュートリアルの最後](#pmc)に説明します。

## <a name="drop-the-database"></a>データベースを削除します。

EF Core ツールを[グローバル ツール](/ef/core/miscellaneous/cli/dotnet)としてインストールし、データベースを削除します。

 ```dotnetcli
 dotnet tool install --global dotnet-ef
 dotnet ef database drop
 ```

次のセクションでは、CLI コマンドを実行する方法について説明します。

## <a name="create-an-initial-migration"></a>初期移行を作成する

変更を保存し、プロジェクトをビルドします。 次に、コマンド ウィンドウを開き、プロジェクト フォルダーに移動します。 この操作を行う簡単な方法を次に示します。

* **[ソリューション エクスプローラー]** で、プロジェクトを右クリックし、コンテキスト メニューの **[エクスプローラーでフォルダーを開く]** を選びます。

  ![[エクスプローラーで開く] メニュー項目](migrations/_static/open-in-file-explorer.png)

* アドレス バーに「cmd」と入力して、Enter キーを押します。

  ![コマンド ウィンドウを開く](migrations/_static/open-command-window.png)

コマンド ウィンドウで次のコマンドを入力します。

```dotnetcli
dotnet ef migrations add InitialCreate
```

上のコマンドでは、次のような出力が表示されます。

```console
info: Microsoft.EntityFrameworkCore.Infrastructure[10403]
      Entity Framework Core initialized 'SchoolContext' using provider 'Microsoft.EntityFrameworkCore.SqlServer' with options: None
Done. To undo this action, use 'ef migrations remove'
```

"*別のプロセスによって使用されているため、ContosoUniversity.dll ファイルにアクセスできません。* " というエラー メッセージが表示された場合は、Windows のシステム トレイの IIS Express アイコンを見つけて右クリックし、 **[ContosoUniversity]、[サイトの停止]** の順にクリックします。

## <a name="examine-up-and-down-methods"></a>Up および Down メソッドを確認する

`migrations add` コマンドを実行したときに、EF では最初からデータベースを作成するコードが生成されています。 このコードは、 *\<timestamp>_InitialCreate.cs* という名前のファイルにある *[Migrations]* フォルダー内にあります。 `InitialCreate` クラスの `Up` メソッドでは、データ モデルのエンティティ セットに対応するデータベース テーブルを作成し、`Down` メソッドでは、次の例に示すようにそれらを削除します。

[!code-csharp[](intro/samples/cu/Migrations/20170215220724_InitialCreate.cs?range=92-118)]

移行は、`Up` メソッドを呼び出して、移行のためのデータ モデルの変更を実装します。 更新をロールバックするためのコマンドを入力すると、移行が `Down` メソッドを呼び出します。

このコードは、`migrations add InitialCreate` コマンドを入力したときに作成された初期移行向けのものです。 移行の name パラメーター (この例では "InitialCreate") は、ファイル名に使用され、どのような名前でも付けることができます。 移行で行われている処理をまとめた単語または語句を選択することをお勧めします。 たとえば、後で移行に "AddDepartmentTable" という名前を付ける可能性があります。

データベースが既に存在するときに、初期移行を作成した場合、データベースの作成コードが生成されますが、データベースは既にデータと一致しているため、作成コードを実行する必要はありません。 データベースがまだ存在しない別の環境にアプリを展開する場合、データベースを作成するために、このコードが実行されるため、最初にテストを行うことをお勧めします。 これが、移行で新しいデータベースを最初から作成できるように、前の接続文字列でデータベースの名前を変更した理由です。

## <a name="the-data-model-snapshot"></a>データ モデルのスナップショット

移行は、現在のデータベース スキーマの *スナップショット* を *Migrations/SchoolContextModelSnapshot.cs* 内に作成します。 移行を追加するときに、EF は、スナップショット ファイルとデータ モデルを比較することによって変更内容を判断します。

移行を削除するには、[dotnet ef migrations remove](/ef/core/miscellaneous/cli/dotnet#dotnet-ef-migrations-remove) コマンドを使用します。 `dotnet ef migrations remove` によって移行が削除され、スナップショットが正しくリセットされたことが確認されます。 `dotnet ef migrations remove` が失敗した場合は、`dotnet ef migrations remove -v` を使用して、エラーに関する詳細情報を確認してください。

スナップショット ファイルの使用方法の詳細については、[チーム環境での EF Core 移行](/ef/core/managing-schemas/migrations/teams)に関するページを参照してください。

## <a name="apply-the-migration"></a>移行を適用する

コマンド ウィンドウで、次のコマンドを入力してデータベースとデータベース内のテーブルを作成します。

```dotnetcli
dotnet ef database update
```

コマンドからの出力は、データベースを設定する SQL コマンドのログを表示する以外は、`migrations add` コマンドと同様です。 次のサンプル出力では、ログのほとんどは省略されています。 ログ メッセージの詳細レベルを表示しない場合は、*appsettings.Development.json* ファイルでログ レベルを変更できます。 詳細については、「<xref:fundamentals/logging/index>」を参照してください。

```text
info: Microsoft.EntityFrameworkCore.Infrastructure[10403]
      Entity Framework Core initialized 'SchoolContext' using provider 'Microsoft.EntityFrameworkCore.SqlServer' with options: None
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (274ms) [Parameters=[], CommandType='Text', CommandTimeout='60']
      CREATE DATABASE [ContosoUniversity2];
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (60ms) [Parameters=[], CommandType='Text', CommandTimeout='60']
      IF SERVERPROPERTY('EngineEdition') <> 5
      BEGIN
          ALTER DATABASE [ContosoUniversity2] SET READ_COMMITTED_SNAPSHOT ON;
      END;
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (15ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      CREATE TABLE [__EFMigrationsHistory] (
          [MigrationId] nvarchar(150) NOT NULL,
          [ProductVersion] nvarchar(32) NOT NULL,
          CONSTRAINT [PK___EFMigrationsHistory] PRIMARY KEY ([MigrationId])
      );

<logs omitted for brevity>

info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (3ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      INSERT INTO [__EFMigrationsHistory] ([MigrationId], [ProductVersion])
      VALUES (N'20190327172701_InitialCreate', N'5.0-rtm');
Done.
```

**SQL Server オブジェクト エクスプローラー** を使用して、最初のチュートリアルで行ったように、データベースを調べます。  データベースに適用されている移行を記録する \_\_EFMigrationsHistory テーブルに追加があることがわかります。 テーブルのデータを表示すると、最初の移行の 1 行が表示されます。 (前の CLI の出力例の最後のログは、この行を作成する INSERT ステートメントを示しています)。

アプリケーションを実行して、すべてが以前と同じように動作することを確認します。

![Students インデックス ページ](migrations/_static/students-index.png)

<a id="pmc"></a>

## <a name="compare-cli-and-pmc"></a>CLI と PMC を比較する

移行を管理するための EF ツールは、.NET Core CLI コマンドから、または Visual Studio **パッケージ マネージャー コンソール** (PMC) ウィンドウの PowerShell コマンドレットから利用できます。 このチュートリアルでは、CLI の使用方法を示しますが、好みに応じて PMC を使用できます。

PMC コマンドの EF コマンドは、[Microsoft.EntityFrameworkCore.Tools](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools) パッケージ内にあります。 このパッケージは [Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)に含まれているので、アプリに `Microsoft.AspNetCore.App` のパッケージ参照がある場合、パッケージ参照を追加する必要はありません。

**重要:** これは、 *.csproj* ファイルを編集して CLI 用にインストールするものと同じパッケージではありません。 `Tools.DotNet` で終わる CLI パッケージ名とは異なり、このパッケージの名前は `Tools` で終わります。

CLI コマンドの詳細については、「[.NET Core CLI](/ef/core/miscellaneous/cli/dotnet)」を参照してください。

PMC コマンドの詳細については、「[パッケージ マネージャー コンソール (Visual Studio)](/ef/core/miscellaneous/cli/powershell)」を参照してください。

## <a name="get-the-code"></a>コードを取得する

[完成したアプリケーションをダウンロードまたは表示する。](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples)

## <a name="next-step"></a>次のステップ

データ モデルの展開に関するより高度なトピックを確認するには、次のチュートリアルに進んでください。 その途中で、追加の移行を作成して適用することになります。

> [!div class="nextstepaction"]
> [追加の移行を作成して適用する](complex-data-model.md)
