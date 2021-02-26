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
ms.openlocfilehash: 41d4fd2e746e08d32d9f666faab55acc56817be2
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551344"
---
<!-- Options common to Razor Pages and Controller -->
| オプション               | 説明|
| ----------------- | ------------ |
| --model または -m  | 使用するモデル クラス。 |
| --dataContext または -dc  | 使用する `DbContext` クラス。 |
| --bootstrapVersion または -b  | ブートストラップのバージョンを指定します。 有効な値は `3` または `4`です。 既定値は `4` です。 指定されたバージョンのブートストラップ ファイルを含む *wwwroot* ディレクトリが必要で存在しない場合は、作成されます。 |
| --referenceScriptLibraries または -scripts |  生成されたビューでスクリプト ライブラリを参照します。 [編集] および [作成] ページに `_ValidationScriptsPartial` を追加します。 |
| --layout または -l | 使用するカスタム レイアウト ページ。 |
| --useDefaultLayout または -udl | ビューの既定のレイアウトを使用します。 |
| --force または -f | 既存のファイルを上書きします。 |
| --relativeFolderPath または -outDir | ファイルが生成されるプロジェクトの相対出力フォルダー パス。 指定しない場合、ファイルはプロジェクト フォルダーに生成されます。 |