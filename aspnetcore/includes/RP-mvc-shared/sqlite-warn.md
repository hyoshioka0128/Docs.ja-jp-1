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
ms.openlocfilehash: e8c5bd00178aefb91ab0e7066c5490ceba315530
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552080"
---
> [!NOTE]
> このチュートリアルでは、可能な場合は Entity Framework Core の "*移行*" 機能を使います。 移行では、データ モデルの変更に合わせてデータベース スキーマが更新されます。 ただし、移行では EF Core プロバイダーでサポートされている種類の変更しか行うことができず、SQLite プロバイダーの機能は制限されています。 たとえば、列の追加はサポートされていますが、列の削除や変更はサポートされていません。 列を削除または変更するために移行を作成した場合、`ef migrations add` コマンドは成功しますが、`ef database update` コマンドは失敗します。 これらの制限により、このチュートリアルでは SQLite のスキーマの変更に対しては移行を使用しません。 代わりに、スキーマを変更するときは、データベースを削除して再作成します。
>
>SQLite の制限に対する回避策としては、テーブル内の何かが変更されたときにテーブルの再構築を実行する移行コードを、手動で作成します。 テーブルの再構築には次の作業が含まれます。
>
>* 新しいテーブルの作成。
>* 古いテーブルから新しいテーブルへのデータのコピー。
>* 古いテーブルの削除。
>* 新しいテーブルの名前変更。
>
>詳細については、次のリソースを参照してください。
>
> * [SQLite EF Core データベース プロバイダーの制限事項](/ef/core/providers/sqlite/limitations)
> * [移行コードをカスタマイズする](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [データのシード処理](/ef/core/modeling/data-seeding)
  * [SQLite ALTER TABLE ステートメント](https://sqlite.org/lang_altertable.html)