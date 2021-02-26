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
ms.openlocfilehash: c96c5e66d2e15db03c321d239a64b98119a7543a
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552107"
---
<!-- USED in RP and MVC tutorial -->

## <a name="add-validation-rules-to-the-movie-model"></a>検証規則をムービー モデルに追加する

DataAnnotations 名前空間には、クラスまたはプロパティに宣言的に適用される一連の組み込みの検証属性があります。 また、DataAnnotations には、書式設定を支援し、どの検証を行わない `DataType` のような書式設定属性もあります。

組み込みの `Required`、`StringLength`、`RegularExpression`、および `Range` 検証属性を利用するように、`Movie` クラスを更新します。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Models/MovieDateRatingDA.cs?name=snippet1)]

検証属性では、適用対象のモデル プロパティに適用する動作が指定されます。

* `Required` および `MinimumLength` 属性は、プロパティに値が必要であることを示します。ただし、この検証を満たすためにユーザーが空白を入力することは禁止されていません。
* `RegularExpression` 属性は、入力できる文字を制限するために使用されます。 上のコード "Genre" では:

  * 文字のみを使用する必要があります。
  * 最初の文字は大文字にする必要があります。 空白、数字、特殊文字は使用できません。

* `RegularExpression` "評価":

  * 最初の文字が大文字である必要があります。
  * 後続のスペースでは、特殊文字と数字が使用できます。 "PG-13" は評価に対して有効ですが、"Genre" に対しては失敗します。

* `Range` 属性は、指定した範囲内に値を制限します。
* `StringLength` 属性では、文字列プロパティの最大長を設定でき、オプションとして最小長も設定できます。
* 値の型 (`decimal`、`int`、`float`、`DateTime` など) は本質的に必須ではなく、`[Required]` 属性を必要としません。

ASP.NET Core によって検証規則が自動的に適用されるようにすると、アプリをより堅牢にできます。 また、ユーザーが何かを検証することを忘れてしまい、データベースに不適切なデータが誤って格納されることもなくなります。
