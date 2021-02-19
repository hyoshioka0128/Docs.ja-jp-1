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
ms.openlocfilehash: 6808c71b1ca43755eea4958ff9409f40e8685694
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551247"
---
このチュートリアルでは、ASP.NET Core MVC と Entity Framework Core のコントローラーとビューについて説明します。 [Razor Pages](xref:razor-pages/index) は代替プログラミング モデルです。 新しい開発では、コントローラーやビューを使う MVC よりも Razor Pages を使うことをお勧めします。 このチュートリアルの [Razor Pages](xref:data/ef-rp/intro) バージョンを参照してください。 それぞれのチュートリアルには、もう一方では説明されない内容が含まれています。

この MVC のチュートリアルに含まれ、Razor Pages のチュートリアルには含まれていないこと:

* データ モデルで継承を実装する
* 生 SQL クエリを実行する
* 動的な LINQ を使ってコードを簡略化する

Razor Pages のチュートリアルに含まれ、このチュートリアルには含まれていないこと:

* Select メソッドを使って関連データを読み込む
* EF のベスト プラクティス。