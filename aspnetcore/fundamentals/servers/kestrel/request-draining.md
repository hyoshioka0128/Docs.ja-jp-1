---
title: ASP.NET Core Kestrel Web サーバーでの要求のドレイン
author: rick-anderson
description: ASP.NET Core 用のクロスプラットフォーム Web サーバーである Kestrel での要求のドレインについて説明します。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 05/04/2020
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
uid: fundamentals/servers/kestrel/request-draining
ms.openlocfilehash: 41d517dae939ad0a83a3402e72eefc4e9db7b32e
ms.sourcegitcommit: 063a06b644d3ade3c15ce00e72a758ec1187dd06
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98253866"
---
# <a name="request-draining-with-aspnet-core-kestrel-web-server"></a>ASP.NET Core Kestrel Web サーバーでの要求のドレイン

HTTP 接続を開くには時間がかかります。 HTTPS の場合も、リソースが大量に消費されます。 そのため、Kestrel は、HTTP/1.1 プロトコルごとに接続を再利用しようとします。 接続を再利用できるようにするには、要求本文を完全に使用する必要があります。 サーバーがリダイレクトや 404 応答を返す HTTP POST 要求のように、アプリによって常に要求本文が使用されるわけではありません。 HTTP POST リダイレクトの場合は、次のようになります。

* クライアントによって、POST データの一部が既に送信されている可能性があります。
* サーバーは 301 応答を書き込みます。
* 以前の要求本文の POST データが完全に読み取られるまで、新しい要求に対して接続を使用することはできません。
* Kestrel は、要求本文のドレインを試行します。 要求本文のドレインは、データを処理せずに読み取りおよび破棄を行うことを意味します。

ドレイン プロセスでは、接続が再利用できるようになる代わりに、残りのデータをドレインするのに時間がかかります。

* ドレインのタイムアウトは 5 秒であり、この設定は構成できません。
* `Content-Length` または `Transfer-Encoding` ヘッダーで指定されたすべてのデータがタイムアウト前に読み取られていない場合、接続は閉じられます。

場合によっては、応答の書き込みの前後に要求をすぐに終了する必要があります。 たとえば、クライアントのデータ キャップが制限されている場合があります。 アップロードされたデータを制限することが優先される場合があります。 このような場合、要求を終了するには、コントローラー、Razor ページ、またはミドルウェアから [HttpContext.Abort](xref:Microsoft.AspNetCore.Http.HttpContext.Abort%2A) を呼び出します。

`Abort` を呼び出す際には、次の点に注意する必要があります。

* 新しい接続の作成には、時間とコストがかかることがあります。
* 接続が閉じる前にクライアントが応答を読み取ることは保証されません。
* `Abort` の呼び出しは最小限に抑え、一般的なエラーではなく重大なエラーが発生した場合のために予約する必要があります。
  * `Abort` は特定の問題を解決する必要がある場合にのみ、呼び出してください。 たとえば、悪意のあるクライアントによってデータの POST が試行されている場合や、クライアント コードに大量の、または複数の要求を引き起こすバグがある場合に `Abort` を呼び出します。
  * HTTP 404 (Not Found) などの一般的なエラー状況では、`Abort` を呼び出さないでください。

`Abort` を呼び出す前に [HttpResponse.CompleteAsync](xref:Microsoft.AspNetCore.Http.HttpResponse.CompleteAsync%2A) を呼び出すと、サーバーが応答の書き込みを完了したことが保証されます。 ただし、クライアントの動作は予測できないため、接続が中止される前に応答が読み取られない場合があります。

プロトコルでは、接続を閉じずに個々の要求ストリームを中止することがサポートされているため、HTTP/2 では、このプロセスが異なります。 5 秒のドレイン タイムアウトは適用されません。 応答の完了後に未読の要求本文データがある場合、サーバーは HTTP/2 RST フレームを送信します。 追加の要求本文データ フレームは無視されます。

可能なかぎり、クライアントが [Expect: 100-continue](https://developer.mozilla.org/docs/Web/HTTP/Status/100) の要求ヘッダーを利用し、要求本文の送信を開始する前にサーバーの応答を待機することをお勧めします。 これにより、クライアントは、不要なデータを送信する前に、応答を調べて中止することができます。
