---
title: ASP.NET Core のロード テスト/ストレス テスト
author: Jeremy-Meng
description: ASP.NET Core アプリのロード テストとストレス テストを行うために、いくつかの注目すべきツールとアプローチについて説明します。
ms.author: riande
ms.custom: mvc
ms.date: 4/05/2019
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
uid: test/loadtests
ms.openlocfilehash: 5a860fe398dff2e12671b6854a80bbab3120499b
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102109482"
---
# <a name="aspnet-core-loadstress-testing"></a>ASP.NET Core のロード テスト/ストレス テスト

ロード テストとストレス テストは、Web アプリのパフォーマンスと拡張性を確保するために重要です。 これらの目標は、よく似たテストを共有する場合でも異なります。

**ロード テスト**:アプリが応答の目標を満たしつつ、特定のシナリオで、指定されたユーザー負荷を処理できるかどうかをテストします。 アプリは通常の状態で実行されます。

**ストレス テスト**:厳しい条件下で、多くの場合長い時間実行したときのアプリの安定性をテストします。 このテストでは、急激な負荷または徐々に増加する負荷によって、アプリに高いユーザー負荷をかけます。または、アプリのコンピューティング リソースを制限します。

ストレス テストでは、負荷がかかっているアプリが障害から復旧し、期待される動作に適切に戻ることができるかどうかを判断します。 ストレスをかけるアプリは、通常の状況では実行されません。

Visual Studio 2019 では、[ロード テストを廃止](https://devblogs.microsoft.com/devops/cloud-based-load-testing-service-eol/)する計画が発表されました。 対応する Azure DevOps のクラウドベースのロード テスト サービスは終了しています。

## <a name="third-party-tools"></a>サードパーティ製のツール

次の一覧には、さまざまな機能セットを備えたサードパーティ製の Web パフォーマンス ツールが含まれています。

* [Apache JMeter](https://jmeter.apache.org/)
* [ApacheBench (ab)](https://httpd.apache.org/docs/2.4/programs/ab.html)
* [Gatling](https://gatling.io/)
* [k6](https://k6.io)
* [Locust](https://locust.io/)
* [West Wind WebSurge](https://websurge.west-wind.com/)
* [Netling](https://github.com/hallatore/Netling)
* [Vegeta](https://github.com/tsenart/vegeta)
* [NBomber](https://github.com/PragmaticFlow/NBomber)
