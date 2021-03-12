---
title: ASP.NET Core でホステッド サービスを使用するバックグラウンド タスク
author: rick-anderson
description: ASP.NET Core でホステッド サービスを使用するバックグラウンド タスクの実装方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/10/2020
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
uid: fundamentals/host/hosted-services
ms.openlocfilehash: c0492c0c5b660e1387b0d0a4f6be405ded49ee92
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587477"
---
# <a name="background-tasks-with-hosted-services-in-aspnet-core"></a>ASP.NET Core でホステッド サービスを使用するバックグラウンド タスク

作成者: [Jeow Li Huan](https://github.com/huan086)

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core では、バックグラウンド タスクを *ホステッド サービス* として実装することができます。 ホストされるサービスは、<xref:Microsoft.Extensions.Hosting.IHostedService> インターフェイスを実装するバックグラウンド タスク ロジックを持つクラスです。 このトピックでは、3 つのホステッド サービスの例について説明します。

* タイマーで実行されるバックグラウンド タスク。
* [スコープ サービス](xref:fundamentals/dependency-injection#service-lifetimes)をアクティブ化するホステッド サービス。 スコープ サービスは[依存関係の挿入 (DI)](xref:fundamentals/dependency-injection) を使用できます。
* 連続して実行される、キューに格納されたバックグラウンド タスク。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/hosted-services/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="worker-service-template"></a>ワーカー サービス テンプレート

ASP.NET Core ワーカー サービス テンプレートは、実行時間が長いサービス アプリを作成する場合の出発点として利用できます。 ワーカー サービス テンプレートから作成されたアプリで、そのプロジェクト ファイル内のワーカー SDK が指定されます。

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

ホステッド サービス アプリの基礎としてテンプレートを使用するには:

[!INCLUDE[](~/includes/worker-template-instructions.md)]

## <a name="package"></a>Package

ワーカー サービス テンプレートに基づくアプリは `Microsoft.NET.Sdk.Worker` SDK を使用し、[Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting) パッケージへの明示的なパッケージ参照を含んでいます。 たとえば、サンプル アプリのプロジェクト ファイル (*BackgroundTasksSample.csproj*) を参照してください。

`Microsoft.NET.Sdk.Web` SDK を使用する Web アプリの場合、[Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting) パッケージは共有フレームワークから暗黙的に参照されます。 アプリのプロジェクト ファイル内の明示的なパッケージ参照は必要ありません。

## <a name="ihostedservice-interface"></a>IHostedService インターフェイス

<xref:Microsoft.Extensions.Hosting.IHostedService> インターフェイスは、ホストによって管理されるオブジェクトの 2 つのメソッドを定義します。

* [StartAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync*): `StartAsync` には、バックグラウンド タスクを開始するロジックが含まれています。 `StartAsync` は、以下よりも "*前に*" 呼び出されます。

  * アプリの要求処理パイプラインが構成される (`Startup.Configure`)。
  * サーバーが起動して、[IApplicationLifetime.ApplicationStarted](xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime.ApplicationStarted*) がトリガーされる。

  既定の動作を変更して、アプリのパイプラインが構成されて `ApplicationStarted` が呼び出された後で、ホステッド サービスの `StartAsync` が実行するようにできます。 既定の動作を変更するには、`ConfigureWebHostDefaults` を呼び出した後でホステッド サービス (次の例では `VideosWatcher`) を追加します。

  ```csharp
  using Microsoft.AspNetCore.Hosting;
  using Microsoft.Extensions.DependencyInjection;
  using Microsoft.Extensions.Hosting;

  public class Program
  {
      public static void Main(string[] args)
      {
          CreateHostBuilder(args).Build().Run();
      }

      public static IHostBuilder CreateHostBuilder(string[] args) =>
          Host.CreateDefaultBuilder(args)
              .ConfigureWebHostDefaults(webBuilder =>
              {
                  webBuilder.UseStartup<Startup>();
              })
              .ConfigureServices(services =>
              {
                  services.AddHostedService<VideosWatcher>();
              });
  }
  ```

* [StopAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*):ホストが正常なシャットダウンを実行しているときにトリガーされます。 `StopAsync` には、バックグラウンド タスクを終了するロジックが含まれています。 アンマネージ リソースを破棄するには、<xref:System.IDisposable> と[ファイナライザー (デストラクター)](/dotnet/csharp/programming-guide/classes-and-structs/destructors) を実装します。

  キャンセル トークンには、シャットダウン プロセスが正常に行われないことを示す、既定の 5 秒間のタイムアウトが含まれています。 キャンセルがトークンに要求された場合:

  * アプリで実行されている残りのバックグラウンド操作が中止します。
  * `StopAsync` で呼び出されたすべてのメソッドが速やかに戻ります。

  ただし、キャンセルが要求された後もタスクは破棄されません&mdash;呼び出し元がすべてのタスクの完了を待機します。

  アプリが予期せずシャットダウンした場合 (たとえば、アプリのプロセスが失敗した場合)、`StopAsync` は呼び出されないことがあります。 そのため、`StopAsync` で呼び出されたメソッドや行われた操作が実行されない可能性があります。

  既定の 5 秒のシャットダウン タイムアウトを延長するには、次を設定します。

  * <xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*> (汎用ホストを使用するとき) 詳細については、「<xref:fundamentals/host/generic-host#shutdown-timeout>」を参照してください。
  * シャットダウン タイムアウトのホスト構成設定 (Web ホストを使用するとき) 詳細については、「<xref:fundamentals/host/web-host#shutdown-timeout>」を参照してください。

ホステッド サービスは、アプリの起動時に一度アクティブ化され、アプリのシャットダウン時に正常にシャットダウンされます。 バックグラウンド タスクの実行中にエラーがスローされた場合、`StopAsync` が呼び出されていなくても `Dispose` を呼び出す必要があります。

## <a name="backgroundservice-base-class"></a>BackgroundService 基底クラス

<xref:Microsoft.Extensions.Hosting.BackgroundService> は、長期 <xref:Microsoft.Extensions.Hosting.IHostedService> を実装するための基底クラスです。

[ExecuteAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.BackgroundService.ExecuteAsync*) は、バックグラウンド サービスを実行するために呼び出されます。 この実装では、バックグラウンド サービスの有効期間全体を表す <xref:System.Threading.Tasks.Task> が返されます。 `await` を呼び出すなどして [ExecuteAsync が非同期になる](https://github.com/dotnet/extensions/issues/2149)まで、以降のサービスは開始されません。 `ExecuteAsync` では、長時間の初期化作業を実行しないようにしてください。 ホストは [StopAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.BackgroundService.StopAsync*) でブロックされ、`ExecuteAsync` の完了を待機します。

[IHostedService.StopAsync](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*) が呼び出されると、キャンセル トークンがトリガーされます。 サービスを正常にシャットダウンするためのキャンセル トークンが起動すると、`ExecuteAsync` の実装はすぐに終了します。 それ以外の場合は、シャットダウンのタイムアウト時にサービスが強制的にシャットダウンします。 詳細については、「[IHostedService インターフェイス](#ihostedservice-interface)」のセクションを参照してください。

## <a name="timed-background-tasks"></a>時間が指定されたバックグラウンド タスク

時間が指定されたバックグラウンド タスクは、[System.Threading.Timer](xref:System.Threading.Timer) クラスを利用します。 このタイマーはタスクの `DoWork` メソッドをトリガーします。 タイマーは `StopAsync` で無効になり、`Dispose` でサービス コンテナーが破棄されたときに破棄されます。

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/TimedHostedService.cs?name=snippet1&highlight=16-17,34,41)]

前の `DoWork` の実行が完了するまで <xref:System.Threading.Timer> は待機されないため、ここで示したアプローチはすべてのシナリオに適しているとは限りません。 [Interlocked.Increment](xref:System.Threading.Interlocked.Increment*) は、アトミック操作として実行カウンターをインクリメントするために使用されされます。これにより、複数のスレッドによって `executionCount` が同時に更新されなくなります。

サービスは、`AddHostedService` 拡張メソッドを使用して `IHostBuilder.ConfigureServices` (*Program.cs*) に登録されます。

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet1)]

## <a name="consuming-a-scoped-service-in-a-background-task"></a>バックグラウンド タスクでスコープ サービスを使用する

[BackgroundService](#backgroundservice-base-class) 内で[スコープ サービス](xref:fundamentals/dependency-injection#service-lifetimes)を使用するには、スコープを作成します。 既定では、ホステッド サービスのスコープは作成されません。

バックグラウンド タスクのスコープ サービスには、バックグラウンド タスクのロジックが含まれています。 次に例を示します。

* サービスは非同期です。 `DoWork` メソッドは `Task` を返します。 デモンストレーションのために、10 秒の遅延が `DoWork` メソッドで待機されます。
* <xref:Microsoft.Extensions.Logging.ILogger> がサービスに挿入されます。

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/ScopedProcessingService.cs?name=snippet1)]

ホステッド サービスはスコープを作成してバックグラウンド タスクのスコープ サービスを解決し、`DoWork` メソッドを呼び出します。 `ExecuteAsync` で待機していた `DoWork` が `Task` を返します。

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/ConsumeScopedServiceHostedService.cs?name=snippet1&highlight=19,22-35)]

サービスは `IHostBuilder.ConfigureServices` (*Program.cs*) に登録されています。 ホステッド サービスは、`AddHostedService` 拡張メソッドを使用して登録されます。

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet2)]

## <a name="queued-background-tasks"></a>キューに格納されたバックグラウンド タスク

バックグラウンド タスク キューは、.NET 4.x <xref:System.Web.Hosting.HostingEnvironment.QueueBackgroundWorkItem*> に基づいています。

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/BackgroundTaskQueue.cs?name=snippet1)]

次の `QueueHostedService` の例では以下のようになります。

* `ExecuteAsync` で待機していた `BackgroundProcessing` メソッドが `Task`を返します。
* `BackgroundProcessing` で、キュー内のバックグラウンド タスクがデキューされ、実行されます。
* 作業項目が待機されてから、`StopAsync` でサービスが停止します。

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/QueuedHostedService.cs?name=snippet1&highlight=28-29,33)]

`MonitorLoop` サービスは、`w` キーが入力デバイスで選択されると常に、ホステッド サービスのためにタスクのエンキューを処理します。

* `IBackgroundTaskQueue` が `MonitorLoop` サービスに挿入されます。
* `IBackgroundTaskQueue.QueueBackgroundWorkItem` が呼び出され、作業項目がエンキューされます。
* 作業項目により、実行時間の長いバックグラウンド タスクがシミュレートされます。
  * 5 秒間の遅延が 3 回実行されます (`Task.Delay`)。
  * タスクが取り消された場合、`try-catch` ステートメントによって <xref:System.OperationCanceledException> がトラップされます。

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/MonitorLoop.cs?name=snippet_Monitor&highlight=7,33)]

サービスは `IHostBuilder.ConfigureServices` (*Program.cs*) に登録されています。 ホステッド サービスは、`AddHostedService` 拡張メソッドを使用して登録されます。

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet3)]

`MonitorLoop` は `Program.Main` で開始されます。

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet4)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core では、バックグラウンド タスクを *ホステッド サービス* として実装することができます。 ホストされるサービスは、<xref:Microsoft.Extensions.Hosting.IHostedService> インターフェイスを実装するバックグラウンド タスク ロジックを持つクラスです。 このトピックでは、3 つのホステッド サービスの例について説明します。

* タイマーで実行されるバックグラウンド タスク。
* [スコープ サービス](xref:fundamentals/dependency-injection#service-lifetimes)をアクティブ化するホステッド サービス。 スコープ サービスは[依存関係の挿入 (DI)](xref:fundamentals/dependency-injection) を使用できます
* 連続して実行される、キューに格納されたバックグラウンド タスク。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/hosted-services/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="package"></a>Package

[Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)を参照するか、[Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting) パッケージへのパッケージ参照を追加します。

## <a name="ihostedservice-interface"></a>IHostedService インターフェイス

ホステッド サービスでは、<xref:Microsoft.Extensions.Hosting.IHostedService> インターフェイスを実装します。 このインターフェイスは、ホストによって管理されるオブジェクトの 2 つのメソッドを定義します。

* [StartAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync*): `StartAsync` には、バックグラウンド タスクを開始するロジックが含まれています。 [Web ホスト](xref:fundamentals/host/web-host) を使用する場合は、サーバーが起動し、[IApplicationLifetime.ApplicationStarted](xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime.ApplicationStarted*) がトリガーされた後で、`StartAsync` が呼び出されます。 [汎用ホスト](xref:fundamentals/host/generic-host) を使用する場合は、`ApplicationStarted` がトリガーされる前に `StartAsync` が呼び出されます。

* [StopAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*):ホストが正常なシャットダウンを実行しているときにトリガーされます。 `StopAsync` には、バックグラウンド タスクを終了するロジックが含まれています。 アンマネージ リソースを破棄するには、<xref:System.IDisposable> と[ファイナライザー (デストラクター)](/dotnet/csharp/programming-guide/classes-and-structs/destructors) を実装します。

  キャンセル トークンには、シャットダウン プロセスが正常に行われないことを示す、既定の 5 秒間のタイムアウトが含まれています。 キャンセルがトークンに要求された場合:

  * アプリで実行されている残りのバックグラウンド操作が中止します。
  * `StopAsync` で呼び出されたすべてのメソッドが速やかに戻ります。

  ただし、キャンセルが要求された後もタスクは破棄されません&mdash;呼び出し元がすべてのタスクの完了を待機します。

  アプリが予期せずシャットダウンした場合 (たとえば、アプリのプロセスが失敗した場合)、`StopAsync` は呼び出されないことがあります。 そのため、`StopAsync` で呼び出されたメソッドや行われた操作が実行されない可能性があります。

  既定の 5 秒のシャットダウン タイムアウトを延長するには、次を設定します。

  * <xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*> (汎用ホストを使用するとき) 詳細については、「<xref:fundamentals/host/generic-host#shutdown-timeout>」を参照してください。
  * シャットダウン タイムアウトのホスト構成設定 (Web ホストを使用するとき) 詳細については、「<xref:fundamentals/host/web-host#shutdown-timeout>」を参照してください。

ホステッド サービスは、アプリの起動時に一度アクティブ化され、アプリのシャットダウン時に正常にシャットダウンされます。 バックグラウンド タスクの実行中にエラーがスローされた場合、`StopAsync` が呼び出されていなくても `Dispose` を呼び出す必要があります。

## <a name="timed-background-tasks"></a>時間が指定されたバックグラウンド タスク

時間が指定されたバックグラウンド タスクは、[System.Threading.Timer](xref:System.Threading.Timer) クラスを利用します。 このタイマーはタスクの `DoWork` メソッドをトリガーします。 タイマーは `StopAsync` で無効になり、`Dispose` でサービス コンテナーが破棄されたときに破棄されます。

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/TimedHostedService.cs?name=snippet1&highlight=15-16,30,37)]

前の `DoWork` の実行が完了するまで <xref:System.Threading.Timer> は待機されないため、ここで示したアプローチはすべてのシナリオに適しているとは限りません。

サービスは、`AddHostedService` 拡張メソッドを使用して `Startup.ConfigureServices` に登録されます。

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Startup.cs?name=snippet1)]

## <a name="consuming-a-scoped-service-in-a-background-task"></a>バックグラウンド タスクでスコープ サービスを使用する

`IHostedService` 内で[スコープ サービス](xref:fundamentals/dependency-injection#service-lifetimes)を使用するには、スコープを作成します。 既定では、ホステッド サービスのスコープは作成されません。

バックグラウンド タスクのスコープ サービスには、バックグラウンド タスクのロジックが含まれています。 次の例では、<xref:Microsoft.Extensions.Logging.ILogger> がサービスに挿入されています。

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/ScopedProcessingService.cs?name=snippet1)]

ホステッド サービスはスコープを作成してバックグラウンド タスクのスコープ サービスを解決し、`DoWork` メソッドを呼び出します。

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/ConsumeScopedServiceHostedService.cs?name=snippet1&highlight=29-36)]

サービスは `Startup.ConfigureServices` に登録されています。 `IHostedService` の実装は、`AddHostedService` 拡張メソッドで登録されます。

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Startup.cs?name=snippet2)]

## <a name="queued-background-tasks"></a>キューに格納されたバックグラウンド タスク

バックグラウンド タスク キューは、.NET Framework 4.x <xref:System.Web.Hosting.HostingEnvironment.QueueBackgroundWorkItem*> ([暫定的に ASP.NET Core に組み込まれる予定](https://github.com/aspnet/Hosting/issues/1280)) に基づいています。

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/BackgroundTaskQueue.cs?name=snippet1)]

`QueueHostedService` では、キュー内のバックグラウンド タスクは [BackgroundService](#backgroundservice-base-class) としてデキューされ、実行されます。これは、長時間実行する `IHostedService` を構成するための基本クラスです。

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/QueuedHostedService.cs?name=snippet1&highlight=21,25)]

サービスは `Startup.ConfigureServices` に登録されています。 `IHostedService` の実装は、`AddHostedService` 拡張メソッドで登録されます。

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Startup.cs?name=snippet3)]

インデックス ページ モデル クラスで:

* `IBackgroundTaskQueue` がコンストラクターに挿入され、`Queue` に割り当てられます。
* <xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory> が挿入され、`_serviceScopeFactory` に割り当てられます。 ファクトリは、スコープ内でサービス作成するための <xref:Microsoft.Extensions.DependencyInjection.IServiceScope> のインスタンス作成に使用されます。 スコープは、アプリの `AppDbContext` ([スコープ サービス](xref:fundamentals/dependency-injection#service-lifetimes)) を使用し、データベース レコードを `IBackgroundTaskQueue` (シングルトン サービス) に書き込むために作成されます。

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Pages/Index.cshtml.cs?name=snippet1)]

[インデックス] ページで **[タスクの追加]** ボタンを選択すると、`OnPostAddTask` メソッドが実行されます。 `QueueBackgroundWorkItem` が呼び出され、作業項目がエンキューされます。

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Pages/Index.cshtml.cs?name=snippet2)]

::: moniker-end

## <a name="additional-resources"></a>その他の技術情報

* [IHostedService と BackgroundService クラスを使ってマイクロサービスのバックグラウンド タスクを実装する](/dotnet/standard/microservices-architecture/multi-container-microservice-net-applications/background-tasks-with-ihostedservice)
* [Azure App Service で WebJobs を使用してバックグラウンド タスクを実行する](/azure/app-service/webjobs-create)
* <xref:System.Threading.Timer>
