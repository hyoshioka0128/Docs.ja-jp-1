---
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
ms.openlocfilehash: 05c94351ee4747813cfa8dc2318a6fc02c3a46bf
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552120"
---
```console
npm run release
```

このコマンドは、アプリの実行中に提供するクライアント側資産を生成します。 資産は、*wwwroot* フォルダーに配置されます。

Webpack は、次のタスクを完了しました。

* *wwwroot* ディレクトリの内容を消去しました。
* "*トランスコンパイル*" と呼ばれるプロセスで TypeScript を JavaScript に変換しました。
* "*縮小*" と呼ばれるプロセスで、生成後の JavaScript ファイルのサイズを縮小しました。
* 処理済みの JavaScript、CSS、および HTML ファイルを *src* から *wwwroot* ディレクトリにコピーしました。
* 次の要素を *wwwroot/index.html* ファイルに挿入しました。
  * *wwwroot/main.\<hash\>.css* ファイルを参照している `<link>` タグ。 このタグは、終了 `</head>` タグの直前に置かれます。
  * 縮小された *wwwroot/main.\<hash\>.js* ファイルを参照している `<script>` タグ。 このタグは、終了 `</body>` タグの直前に置かれます。