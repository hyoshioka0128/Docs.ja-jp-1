---
title: ASP.NET Core Blazor Server と Entity Framework Core (EFCore)
author: JeremyLikness
description: Blazor Server アプリで EF Core を使用するためのガイダンスです。
monikerRange: '>= aspnetcore-3.1'
ms.author: jeliknes
ms.custom: mvc
ms.date: 08/14/2020
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
uid: blazor/blazor-server-ef-core
ms.openlocfilehash: 53d276db996304852d69566584e43d47aa73f921
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586554"
---
# <a name="aspnet-core-blazor-server-with-entity-framework-core-efcore"></a>ASP.NET Core Blazor Server と Entity Framework Core (EFCore)

:::moniker range=">= aspnetcore-5.0"

Blazor Server はステートフル アプリ フレームワークです。 アプリではサーバーへの継続的な接続が維持され、ユーザーの状態は "*回線*" 内のサーバーのメモリに保持されます。 ユーザー状態の一例として、回線に範囲が設定されている[依存関係の挿入 (DI)](xref:fundamentals/dependency-injection) サービス インスタンスに保存されているデータがあります。 Blazor Server が提供する独自のアプリケーション モデルでは、Entity Framework Core を使用するための特別なアプローチが必要です。

> [!NOTE]
> この記事では、Blazor Server アプリでの EF Core について説明します。 Blazor WebAssembly アプリは、ほとんどの直接データベース接続が防止される WebAssembly サンドボックス内で実行されます。 Blazor WebAssembly での EF Core の実行については、この記事では扱いません。

<h2 id="sample-app-5x">サンプル アプリ</h2>

このサンプル アプリは、EF Core を使用する Blazor Server アプリのリファレンスとして作成されました。 サンプル アプリには、並べ替えとフィルター処理、削除、追加、更新の各操作を行うグリッドが含まれています。 このサンプルは、EF Core を使用してオプティミスティック同時実行制御を処理する方法を示しています。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/blazor/common/samples/5.x/BlazorServerEFCoreSample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

サンプルにはローカルの [SQLite](https://www.sqlite.org/index.html) データベースが使用されているため、どのプラットフォームでも使用できます。 このサンプルでは、生成された SQL クエリを表示するようにデータベースのログも構成されています。 これは `appsettings.Development.json` で構成されています。

[!code-json[](~/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/appsettings.Development.json?highlight=8)]

グリッド、追加、および表示の各コンポーネントでは、操作ごとにコンテキストが作成される "操作ごとのコンテキスト" パターンが使用されます。 編集コンポーネントでは、コンポーネントごとにコンテキストが作成される "コンポーネントごとのコンテキスト" パターンが使用されます。

> [!NOTE]
> このトピックのコード例には、表示されていない名前空間とサービスを必要とするものもあります。 Razor の例に必要な [`@using`](xref:mvc/views/razor#using) および [`@inject`](xref:mvc/views/razor#inject) ディレクティブを含む完全に機能するコードを検査するには、[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/blazor/common/samples/5.x/BlazorServerEFCoreSample)を参照してください。

<h2 id="database-access-5x">データベース アクセス</h2>

EF Core では、[データベース アクセスを構成](/ef/core/miscellaneous/configuring-dbcontext)し、"[*作業単位*](https://martinfowler.com/eaaCatalog/unitOfWork.html)" として機能する手段として <xref:Microsoft.EntityFrameworkCore.DbContext> を利用しています。 EF Core には、既定でコンテキストを "*スコープ*" サービスとして登録する ASP.NET Core アプリの <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> 拡張機能が用意されています。 Blazor Server アプリでは、インスタンスがユーザーの回線内のコンポーネント全体で共有されるため、スコープ サービスの登録が問題になる可能性があります。 <xref:Microsoft.EntityFrameworkCore.DbContext> はスレッド セーフではなく、同時に使用するように設計されていません。 次の理由により、既存の有効期間は不適切です。

* **[Singleton]\(シングルトン\)** の場合、アプリのすべてのユーザーで状態が共有され、不適切な同時使用につながります。
* **[範囲指定]** (既定値) の場合、同じユーザーのコンポーネント間で同様の問題が発生します。
* **[一時的]** の場合、要求ごとに新しいインスタンスが生成されます。ただし、コンポーネントの有効期間が長くなる可能性があるため、意図したよりも時間のかかるコンテキストになります。

以下の推奨事項は、Blazor Server アプリで EF Core を使用する上で一貫したアプローチを提供するように設計されています。

* 既定では、操作ごとに 1 つのコンテキストを使用することを検討してください。 コンテキストは、高速でオーバーヘッドの少ないインスタンス化を目的として設計されています。

  ```csharp
  using var context = new MyContext();

  return await context.MyEntities.ToListAsync();
  ```

* フラグを使用して、複数の同時操作を防止します。

  ```csharp
  if (Loading)
  {
      return;
  }

  try
  {
      Loading = true;

      ...
  }
  finally
  {
      Loading = false;
  }
  ```

  `try` ブロックの `Loading = true;` 行の後に操作を配置します。

* EF Core の[変更追跡](/ef/core/querying/tracking)または[同時実行制御](/ef/core/saving/concurrency)を利用する、より長期間の操作の場合は、[コンポーネントの有効期間にコンテキストの範囲を限定します](#scope-to-the-component-lifetime-5x)。

<h3 id="new-dbcontext-instances-5x">新しい DbContext インスタンス</h3>

新しい <xref:Microsoft.EntityFrameworkCore.DbContext> インスタンスを作成する最も簡単な方法は、`new` を使用して新しいインスタンスを作成することです。 ただし、追加の依存関係を解決する必要があるシナリオがいくつかあります。 たとえば、[`DbContextOptions`](/ef/core/miscellaneous/configuring-dbcontext#configuring-dbcontextoptions) を使用してコンテキストを構成することができます。

依存関係を持つ新しい <xref:Microsoft.EntityFrameworkCore.DbContext> を作成するには、ファクトリを使用することをお勧めします。 EF Core 5.0 以降には、新しいコンテキストを作成するためのファクトリが組み込まれています。

次の例では、[SQLite](https://www.sqlite.org/index.html) を構成し、データのログ記録を有効にします。 このコードでは、[拡張メソッド (`AddDbContextFactory`)](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Data/FactoryExtensions.cs) を使用して、DI のデータベース ファクトリを構成し、既定のオプションを指定しています。

[!code-csharp[](~/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Startup.cs?name=snippet1)]

ファクトリはコンポーネントに挿入され、新しいインスタンスを作成するために使用されます。 たとえば、`Pages/Index.razor` では次のようにします。

[!code-csharp[](~/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/Index.razor?name=snippet1)]

> [!NOTE]
> `Wrapper` は、`GridWrapper` コンポーネントへの[コンポーネント参照](xref:blazor/components/index#capture-references-to-components)です。 [サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/Index.razor)の `Index` コンポーネント (`Pages/Index.razor`) を参照してください。

新しい <xref:Microsoft.EntityFrameworkCore.DbContext> インスタンスは、[[ASP.NET Core の Identity モデル]](xref:security/authentication/customize_identity_model) を使用する場合など、`DbContext` ごとに接続文字列を構成できるファクトリを使用して作成できます。

```csharp
services.AddDbContextFactory<ApplicationDbContext>(options =>
{
    options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection"));
});

services.AddScoped<ApplicationDbContext>(p => 
    p.GetRequiredService<IDbContextFactory<ApplicationDbContext>>()
    .CreateDbContext());
```

<h3 id="scope-to-the-component-lifetime-5x">コンポーネントの有効期間の範囲</h3>

コンポーネントの有効期間中は存在する <xref:Microsoft.EntityFrameworkCore.DbContext> を作成することができます。 これにより、それを[作業単位](https://martinfowler.com/eaaCatalog/unitOfWork.html)として使用し、変更の追跡や同時実行の解決などの組み込み機能を利用することができます。
ファクトリーを使用してコンテキストを作成し、コンポーネントの有効期間中はそれを追跡できます。 まず <xref:System.IDisposable> を実装し、`Pages/EditContact.razor` に示すようにファクトリを挿入します。

```razor
@implements IDisposable
@inject IDbContextFactory<ContactContext> DbFactory
```

サンプル アプリでは、コンポーネントが破棄されるときに、コンテキストも確実に破棄されます。

[!code-csharp[](~/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/EditContact.razor?name=snippet1)]

最後に、[`OnInitializedAsync`](xref:blazor/components/lifecycle) は新しいコンテキストを作成するためにオーバーライドされます。 サンプル アプリでは、[`OnInitializedAsync`](xref:blazor/components/lifecycle) を使用して同じメソッドで連絡先を読み込んでいます。

[!code-csharp[](~/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/EditContact.razor?name=snippet2)]

<h3 id="enable-sensitive-data-logging">機密データのログ記録を有効にする</h3>

<xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> では、例外メッセージおよびフレームワークのログにアプリケーション データが含まれます。 ログに記録されるデータには、エンティティ インスタンスのプロパティに割り当てられた値と、データベースに送信されたコマンドのパラメーター値を含めることができます。 <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> を使用したデータのログ記録は **セキュリティ リスク** です。データベースに対して実行された SQL ステートメントをログに記録するときに、パスワードやその他の個人を特定できる情報 (PII) が公開される可能性があるためです。

<xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> は、開発とテストのためにのみ有効にすることをお勧めします。

```csharp
#if DEBUG
    services.AddDbContextFactory<ContactContext>(opt =>
        opt.UseSqlite($"Data Source={nameof(ContactContext.ContactsDb)}.db")
        .EnableSensitiveDataLogging());
#else
    services.AddDbContextFactory<ContactContext>(opt =>
        opt.UseSqlite($"Data Source={nameof(ContactContext.ContactsDb)}.db"));
#endif
```

:::moniker-end

:::moniker range="< aspnetcore-5.0"

Blazor Server はステートフル アプリ フレームワークです。 アプリではサーバーへの継続的な接続が維持され、ユーザーの状態は "*回線*" 内のサーバーのメモリに保持されます。 ユーザー状態の一例として、回線に範囲が設定されている[依存関係の挿入 (DI)](xref:fundamentals/dependency-injection) サービス インスタンスに保存されているデータがあります。 Blazor Server が提供する独自のアプリケーション モデルでは、Entity Framework Core を使用するための特別なアプローチが必要です。

> [!NOTE]
> この記事では、Blazor Server アプリでの EF Core について説明します。 Blazor WebAssembly アプリは、ほとんどの直接データベース接続が防止される WebAssembly サンドボックス内で実行されます。 Blazor WebAssembly での EF Core の実行については、この記事では扱いません。

<h2 id="sample-app-3x">サンプル アプリ</h2>

このサンプル アプリは、EF Core を使用する Blazor Server アプリのリファレンスとして作成されました。 サンプル アプリには、並べ替えとフィルター処理、削除、追加、更新の各操作を行うグリッドが含まれています。 このサンプルは、EF Core を使用してオプティミスティック同時実行制御を処理する方法を示しています。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

サンプルにはローカルの [SQLite](https://www.sqlite.org/index.html) データベースが使用されているため、どのプラットフォームでも使用できます。 このサンプルでは、生成された SQL クエリを表示するようにデータベースのログも構成されています。 これは `appsettings.Development.json` で構成されています。

[!code-json[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/appsettings.Development.json?highlight=8)]

グリッド、追加、および表示の各コンポーネントでは、操作ごとにコンテキストが作成される "操作ごとのコンテキスト" パターンが使用されます。 編集コンポーネントでは、コンポーネントごとにコンテキストが作成される "コンポーネントごとのコンテキスト" パターンが使用されます。

> [!NOTE]
> このトピックのコード例には、表示されていない名前空間とサービスを必要とするものもあります。 Razor の例に必要な [`@using`](xref:mvc/views/razor#using) および [`@inject`](xref:mvc/views/razor#inject) ディレクティブを含む完全に機能するコードを検査するには、[サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample)を参照してください。

<h2 id="database-access-3x">データベース アクセス</h2>

EF Core では、[データベース アクセスを構成](/ef/core/miscellaneous/configuring-dbcontext)し、"[*作業単位*](https://martinfowler.com/eaaCatalog/unitOfWork.html)" として機能する手段として <xref:Microsoft.EntityFrameworkCore.DbContext> を利用しています。 EF Core には、既定でコンテキストを "*スコープ*" サービスとして登録する ASP.NET Core アプリの <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> 拡張機能が用意されています。 Blazor Server アプリでは、インスタンスがユーザーの回線内のコンポーネント全体で共有されるため、これは問題になる可能性があります。 <xref:Microsoft.EntityFrameworkCore.DbContext> はスレッド セーフではなく、同時に使用するように設計されていません。 次の理由により、既存の有効期間は不適切です。

* **[Singleton]\(シングルトン\)** の場合、アプリのすべてのユーザーで状態が共有され、不適切な同時使用につながります。
* **[範囲指定]** (既定値) の場合、同じユーザーのコンポーネント間で同様の問題が発生します。
* **[一時的]** の場合、要求ごとに新しいインスタンスが生成されます。ただし、コンポーネントの有効期間が長くなる可能性があるため、意図したよりも時間のかかるコンテキストになります。

以下の推奨事項は、Blazor Server アプリで EF Core を使用する上で一貫したアプローチを提供するように設計されています。

* 既定では、操作ごとに 1 つのコンテキストを使用することを検討してください。 コンテキストは、高速でオーバーヘッドの少ないインスタンス化を目的として設計されています。

  ```csharp
  using var context = new MyContext();

  return await context.MyEntities.ToListAsync();
  ```

* フラグを使用して、複数の同時操作を防止します。

  ```csharp
  if (Loading)
  {
      return;
  }

  try
  {
      Loading = true;

      ...
  }
  finally
  {
      Loading = false;
  }
  ```

  `try` ブロックの `Loading = true;` 行の後に操作を配置します。

* EF Core の[変更追跡](/ef/core/querying/tracking)または[同時実行制御](/ef/core/saving/concurrency)を利用する、より長期間の操作の場合は、[コンポーネントの有効期間にコンテキストの範囲を限定します](#scope-to-the-component-lifetime-3x)。

<h3 id="new-dbcontext-instances-3x">新しい DbContext インスタンス</h3>

新しい <xref:Microsoft.EntityFrameworkCore.DbContext> インスタンスを作成する最も簡単な方法は、`new` を使用して新しいインスタンスを作成することです。 ただし、追加の依存関係を解決する必要があるシナリオがいくつかあります。 たとえば、[`DbContextOptions`](/ef/core/miscellaneous/configuring-dbcontext#configuring-dbcontextoptions) を使用してコンテキストを構成することができます。

依存関係を持つ新しい <xref:Microsoft.EntityFrameworkCore.DbContext> を作成するには、ファクトリを使用することをお勧めします。 このサンプル アプリでは、`Data/DbContextFactory.cs` で独自のファクトリを実装しています。

[!code-csharp[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Data/DbContextFactory.cs)]

前のファクトリは、次のようになっています。

* すべての依存関係は、サービス プロバイダーを介して <xref:Microsoft.Extensions.DependencyInjection.ActivatorUtilities.CreateInstance%2A?displayProperty=nameWithType> によって満たされています。
* `IDbContextFactory` は、EF Core ASP.NET Core 5.0 以降で使用できます。そのため、インターフェイスは、[ASP.NET Core 3.x 用のサンプル アプリに実装されます](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Data/IDbContextFactory.cs)。

次の例では、[SQLite](https://www.sqlite.org/index.html) を構成し、データのログ記録を有効にします。 このコードでは、拡張メソッドを使用して、DI のデータベース ファクトリを構成し、既定のオプションを指定しています。

[!code-csharp[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Startup.cs?name=snippet1)]

ファクトリはコンポーネントに挿入され、新しいインスタンスを作成するために使用されます。 たとえば、`Pages/Index.razor` では次のようにします。

[!code-csharp[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/Index.razor?name=snippet1)]

> [!NOTE]
> `Wrapper` は、`GridWrapper` コンポーネントへの[コンポーネント参照](xref:blazor/components/index#capture-references-to-components)です。 [サンプル アプリ](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/Index.razor)の `Index` コンポーネント (`Pages/Index.razor`) を参照してください。

新しい <xref:Microsoft.EntityFrameworkCore.DbContext> インスタンスは、[ASP.NET Core の Identity モデル])(xref:security/authentication/customize_identity_model) を使用する場合など、`DbContext` ごとに接続文字列を構成できるファクトリを使用して作成できます。

```csharp
services.AddDbContextFactory<ApplicationDbContext>(options =>
{
    options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection"));
});

services.AddScoped<ApplicationDbContext>(p => 
    p.GetRequiredService<IDbContextFactory<ApplicationDbContext>>()
    .CreateDbContext());
```

<h3 id="scope-to-the-component-lifetime-3x">コンポーネントの有効期間の範囲</h3>

コンポーネントの有効期間中は存在する <xref:Microsoft.EntityFrameworkCore.DbContext> を作成することができます。 これにより、それを[作業単位](https://martinfowler.com/eaaCatalog/unitOfWork.html)として使用し、変更の追跡や同時実行の解決などの組み込み機能を利用することができます。
ファクトリーを使用してコンテキストを作成し、コンポーネントの有効期間中はそれを追跡できます。 まず <xref:System.IDisposable> を実装し、`Pages/EditContact.razor` に示すようにファクトリを挿入します。

```razor
@implements IDisposable
@inject IDbContextFactory<ContactContext> DbFactory
```

サンプル アプリでは、コンポーネントが破棄されるときに、コンテキストも確実に破棄されます。

[!code-csharp[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/EditContact.razor?name=snippet1)]

最後に、[`OnInitializedAsync`](xref:blazor/components/lifecycle) は新しいコンテキストを作成するためにオーバーライドされます。 サンプル アプリでは、[`OnInitializedAsync`](xref:blazor/components/lifecycle) を使用して同じメソッドで連絡先を読み込んでいます。

[!code-csharp[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/EditContact.razor?name=snippet2)]

前の例の場合:

* `Busy` を `true` に設定すると、非同期操作が開始されることがあります。 `Busy` を `false` に戻すと、非同期操作は完了するはずです。
* 追加のエラー処理ロジックを `catch` ブロックに配置します。

<h3 id="enable-sensitive-data-logging">機密データのログ記録を有効にする</h3>

<xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> では、例外メッセージおよびフレームワークのログにアプリケーション データが含まれます。 ログに記録されるデータには、エンティティ インスタンスのプロパティに割り当てられた値と、データベースに送信されたコマンドのパラメーター値を含めることができます。 <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> を使用したデータのログ記録は **セキュリティ リスク** です。データベースに対して実行された SQL ステートメントをログに記録するときに、パスワードやその他の個人を特定できる情報 (PII) が公開される可能性があるためです。

<xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> は、開発とテストのためにのみ有効にすることをお勧めします。

```csharp
#if DEBUG
    services.AddDbContextFactory<ContactContext>(opt =>
        opt.UseSqlite($"Data Source={nameof(ContactContext.ContactsDb)}.db")
        .EnableSensitiveDataLogging());
#else
    services.AddDbContextFactory<ContactContext>(opt =>
        opt.UseSqlite($"Data Source={nameof(ContactContext.ContactsDb)}.db"));
#endif
```

:::moniker-end

## <a name="additional-resources"></a>その他のリソース

* [EF Core のドキュメント](/ef/)
