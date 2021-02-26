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
ms.openlocfilehash: cf20722e8c8669fb17af8db032d4064ca2be2f4c
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552183"
---
> [!NOTE]
> 
> **SQLite の制限事項**
>
> このチュートリアルでは、可能な場合は Entity Framework Core の *移行* 機能を使います。 移行では、データ モデルの変更に合わせてデータベース スキーマが更新されます。 しかし、移行ではデータベース エンジンによってサポートされる変更の種類のみが行われ、SQLite のスキーマ変更機能は制限されます。 たとえば、列の追加はサポートされていますが、列の削除はサポートされていません。 列を削除するために移行を作成した場合、`ef migrations add` コマンドは成功しますが、`ef database update` コマンドは失敗します。 
>
> SQLite の制限に対する回避策としては、テーブル内の何かが変更されたときにテーブルの再構築を実行する移行コードを、手動で作成します。 このコードは移行のために `Up` メソッドと `Down` メソッドに進み、次のようなものが含まれます。
>
> * 新しいテーブルの作成。
> * 古いテーブルから新しいテーブルへのデータのコピー。
> * 古いテーブルの削除。
> * 新しいテーブルの名前変更。
>
> この種類のデータベース固有のコードを記述することは、このチュートリアルの範囲外です。 代わりに、このチュートリアルでは、移行を適用しようとして失敗した場合に、データベースを削除して再作成します。 詳細については、次のリソースを参照してください。
>
> * [SQLite EF Core データベース プロバイダーの制限事項](/ef/core/providers/sqlite/limitations)
> * [移行コードをカスタマイズする](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [データのシード処理](/ef/core/modeling/data-seeding)
> * [SQLite ALTER TABLE ステートメント](https://sqlite.org/lang_altertable.html)