# <a name="aspnet-core-health-check-sample"></a>ASP.NET Core 正常性チェック サンプル

このサンプルでは、正常性チェック ミドルウェアと各種サービスの使用方法を示します。 このサンプルでは、「[Health checks in ASP.NET Core](https://docs.microsoft.com/aspnet/core/host-and-deploy/health-checks)」(ASP.NET Core の正常性チェック) というトピックで説明されているシナリオを実際に確認できます。

そのトピックで説明しているシナリオに対してサンプル アプリを実行するには、コマンド シェルでプロジェクトのフォルダーから [dotnet run](https://docs.microsoft.com/dotnet/core/tools/dotnet-run) コマンドを使用します。 これから試すシナリオのスイッチを渡します。 スイッチが `basic` に指定されない場合、アプリの設定は既定で `dotnet run` になります。

| シナリオ                                               | サンプル アプリ コマンド               | 説明 |
| ------------------------------------------------------ | -------------------------------- | ----------- |
| 基本的な正常性プローブ (既定)                           | `dotnet run --scenario basic`    | アプリで HTTP 要求を処理できることを確認します。 |
| データベース プローブ                                         | `dotnet run --scenario db`       | SQL Server データベース接続を確認します。 手順については、同トピックの「[Database probe](https://docs.microsoft.com/aspnet/core/host-and-deploy/health-checks#database-probe)」(データベース プローブ) セクションを参照してください。 |
| 対応性/活動性プローブ                              | `dotnet run --scenario liveness` | アプリの状態が稼働中 (*活動性*) か稼働準備中 (*対応性*) かを確認します。 |
| メトリックベースのプローブ (メモリ)/<br>カスタム応答ライター | `dotnet run --scenario writer`   | メモリ使用を確認し、正常性エンドポイントの確認時にカスタム JSON を書き出します。 |
| ポート別のフィルター処理                                         | `dotnet run --scenario port`     | 正常性チェックを指定のポートに絞り込みます。 手順については、同トピックの「[ポート別のフィルター処理](https://docs.microsoft.com/aspnet/core/host-and-deploy/health-checks#filter-by-port)」セクションを参照してください。 |

データベース プローブとポート フィルターのシナリオでは、追加設定が必要です。 詳細については、「[Health checks](https://docs.microsoft.com/aspnet/core/host-and-deploy/health-checks)」(正常性チェック) を参照してください。
