---
title: HttpRepl テレメトリ
author: rick-anderson
description: HttpRepl によって収集されたテレメトリについて説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.date: 11/11/2020
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
uid: web-api/http-repl/telemetry
ms.openlocfilehash: 267a1233c216e823f6ffc086e94c198b4eb103bc
ms.sourcegitcommit: 0abfe496fed8e9470037c8128efa8a50069ccd52
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/07/2021
ms.locfileid: "106563612"
---
# <a name="httprepl-telemetry"></a>HttpRepl テレメトリ

[HttpRepl](xref:web-api/http-repl)には、使用状況データを収集するテレメトリ機能が含まれています。 HttpRepl チームは、ツールがどのように使用されているかを理解し、改善できるようにすることが重要です。

## <a name="how-to-opt-out"></a>オプトアウトする方法

HttpRepl テレメトリ機能は、既定で有効になっています。 製品利用統計情報機能をオプトアウトするには、`DOTNET_HTTPREPL_TELEMETRY_OPTOUT` 環境変数を `1` または `true` に設定します。

## <a name="disclosure"></a>開示

HttpRepl では、ツールを初めて実行したときに、次のようなテキストが表示されます。 テキストは、実行しているツールのバージョンによって多少異なる場合があります。 この "最初の実行" の際に、Microsoft がデータ回収に関して通知する方法が示されます。

```console
Telemetry
---------
The .NET tools collect usage data in order to help us improve your experience. It is collected by Microsoft and shared with the community. You can opt-out of telemetry by setting the DOTNET_HTTPREPL_TELEMETRY_OPTOUT environment variable to '1' or 'true' using your favorite shell.
```

"最初の実行" のエクスペリエンステキストを抑制するに `DOTNET_HTTPREPL_SKIP_FIRST_TIME_EXPERIENCE` は、環境変数をまたはに設定し `1` `true` ます。

## <a name="data-points"></a>データ ポイント

テレメトリ機能は次のものではありません。

* ユーザー名、電子メールアドレス、Url などの個人データを収集します。
* HTTP 要求または応答をスキャンします。

データは、Microsoft サーバーに安全に送信され、制限付きアクセスで保持されます。

ユーザーのプライバシー保護は Microsoft にとって重要です。 テレメトリ機能が機密データを収集していると思われる場合、またはデータの安全でまたは不適切な処理が行われていると思われる場合は、次のいずれかの操作を実行します。

* [Dotnet/httprepl](https://github.com/dotnet/httprepl/issues)リポジトリに問題を記載してください。
* 調査のために電子メールを送信 [dotnet@microsoft.com](mailto:dotnet@microsoft.com) します。

テレメトリ機能は、次のデータを収集します。

| .NET SDK のバージョン | Data |
|--------------|------|
| >= 5.0        | 呼び出しのタイムスタンプ。 |
| >= 5.0        | 地理的な場所を決定するために使用される3オクテットの IP アドレス。 |
| >= 5.0        | オペレーティング システムとバージョン。 |
| >= 5.0        | ツールが実行されているランタイム ID (RID)。 |
| >= 5.0        | ツールがコンテナーで実行されているかどうか。 |
| >= 5.0        | ハッシュされたメディア Access Control (MAC) アドレス: コンピューターの暗号 (SHA256) ハッシュされた一意の ID。 |
| >= 5.0        | カーネル バージョン。 |
| >= 5.0        | HttpRepl のバージョン。 |
| >= 5.0        | ツールが、、のいずれかの引数を使用して開始されたかどうか `help` `run` `connect` 。 実際の引数値は収集されません。 |
| >= 5.0        | 呼び出されたコマンド (など `get` ) と、成功したかどうか。 |
| >= 5.0        | コマンドの場合、 `connect` `root` 、、のいずれかの `base` 引数が指定されているかどうか `openapi` 。 実際の引数値は収集されません。 |
| >= 5.0        | コマンドの場合 `pref` `get` は、またはが発行されたかどうか、 `set` およびアクセスされた設定があるかどうか。 よく知られていない場合は、名前がハッシュ化されます。 値は収集されません。 |
| >= 5.0        | コマンドの場合 `set header` 、設定されているヘッダー名。 Well-known ヘッダーでない場合は、名前がハッシュ化されます。 値は収集されません。 |
| >= 5.0        | コマンドの場合は `connect` 、の特殊なケースが使用されているかどうか、 `dotnet new webapi` および、優先順位によってバイパスされたかどうかを指定します。 |
| >= 5.0        | すべての HTTP コマンド (GET、POST、PUT など) について、各オプションが指定されているかどうかを示します。 オプションの値は収集されません。 |

## <a name="additional-resources"></a>その他の技術情報

* [.NET Core SDK 製品利用統計情報](/dotnet/core/tools/telemetry)
* [テレメトリデータを .NET Core CLI する](https://dotnet.microsoft.com/platform/telemetry)
