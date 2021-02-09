---
title: Blazor Todo リスト アプリを構築する
author: guardrex
description: Blazor アプリを段階的に構築します。
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 12/14/2020
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
uid: tutorials/build-a-blazor-app
ms.openlocfilehash: 106e1119db777074b5eae24f5d7e216e6127ca13
ms.sourcegitcommit: 75db2f684a9302b0be7925eab586aa091c6bd19f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/01/2021
ms.locfileid: "99238303"
---
# <a name="build-a-blazor-todo-list-app"></a>Blazor Todo リスト アプリを構築する

作成者: [Daniel Roth](https://github.com/danroth27)、[Luke Latham](https://github.com/guardrex)

このチュートリアルでは、Blazor アプリをビルドして変更する方法を示します。 以下の方法について説明します。

> [!div class="checklist"]
> * Todo リストの Blazor アプリ プロジェクトを作成する
> * Razor コンポーネントを変更する
> * コンポーネントでイベント処理とデータ バインディングを使用する
> * Blazor アプリでルーティングを使用する

このチュートリアルの最後には、動作する ToDo リスト アプリが完成します。

## <a name="prerequisites"></a>前提条件

::: moniker range=">= aspnetcore-5.0"

[!INCLUDE[](~/includes/5.0-SDK.md)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!INCLUDE[](~/includes/3.1-SDK.md)]

::: moniker-end

## <a name="create-a-todo-list-blazor-app"></a>ToDo リスト Blazor アプリを作成する

1. コマンド シェルで `TodoList` という名前の新しい Blazor アプリを作成します。

   ```dotnetcli
   dotnet new blazorserver -o TodoList
   ```

   上記のコマンドでは、アプリを保持するために `-o|--output` オプションを指定して、`TodoList` という名前のフォルダーを作成します。 `TodoList` フォルダーは、プロジェクトの *ルート フォルダー* です。 次のコマンドを使用して、ディレクトリを `TodoList` フォルダーに変更します。

   ```dotnetcli
   cd TodoList
   ```

1. 次のコマンドを使用して、新しい `Todo` Razor コンポーネントをアプリに追加します。

   ```dotnetcli
   dotnet new razorcomponent -n Todo -o Pages
   ```

   前のコマンドの `-n|--name` オプションで、新しい Razor コンポーネントの名前を指定します。 新しいコンポーネントは、`-o|--output` オプションを使用してプロジェクトの `Pages` フォルダーに作成されます。

   > [!IMPORTANT]
   > Razor コンポーネント ファイル名の先頭文字は、大文字である必要があります。 `Pages` フォルダーを開き、`Todo` コンポーネントのファイル名の先頭が大文字 `T` であることを確認します。 ファイル名は `Todo.razor` のはずです。

1. 任意のファイル エディターで `Todo` コンポーネントを開き、`/todo` の相対 URL を使用して、`@page` Razor ディレクティブをファイルの先頭に追加します。

   `Pages/Todo.razor`:

   [!code-razor[](build-a-blazor-app/samples_snapshot/Todo0.razor?highlight=1)]

   `Pages/Todo.razor` ファイルを保存します。

1. ナビゲーション バーに `Todo` コンポーネントを追加します。

   `NavMenu` コンポーネントはアプリのレイアウトで使用されます。 レイアウトは、アプリ内でのコンテンツの重複を回避できるようにするコンポーネントです。 アプリによってコンポーネントの URL が読み込まれるときに、`NavLink` コンポーネントによりそのアプリの UI でキューが指定されます。

   `NavMenu` コンポーネントの順不同のリスト (`<ul>...</ul>`) で、次のリスト項目 (`<li>...</li>`) と、`Todo` コンポーネントの `NavLink` コンポーネントを追加します。

   `Shared/NavMenu.razor`:

   [!code-razor[](build-a-blazor-app/samples_snapshot/NavMenu.razor?highlight=5-9)]

   `Shared/NavMenu.razor` ファイルを保存します。

1. `TodoList` フォルダーからコマンド シェルで [`dotnet watch run`](/aspnet/core/tutorials/dotnet-watch) コマンドを実行して、アプリをビルドして実行します。 アプリが実行された後、そのアプリのナビゲーション バーにある **[`Todo`]** リンクを選択して、新しい Todo ページにアクセスします。これにより、`/todo` にページが読み込まれます。

   アプリでコマンド シェルを実行したままにします。 ファイルが保存されるたびに、アプリが自動的に再構築されます。 コンパイルおよび再起動中は、ブラウザーからアプリへの接続が一時的に失われます。 接続が再確立されると、ブラウザーのページが自動的に再読み込みされます。

1. Todo アイテムを表すクラスを保持するために、プロジェクト (`TodoList` フォルダー) のルートに `TodoItem.cs` ファイルを追加します。 `TodoItem` クラス用に次の C# コードを使います。

   `TodoItem.cs`:

   [!code-csharp[](build-a-blazor-app/samples_snapshot/TodoItem.cs)]
   
   > [!NOTE]
   > Visual Studio を使用して `ToDoItem.cs` ファイルと `ToDoItem` クラスを作成する場合は、次のいずれかの方法を使用します。
   >
   > * Visual Studio によってクラス用に生成される名前空間を削除します。
   > * 前のコード ブロックの **[コピー]** ボタンを使用し、Visual Studio によって生成されるファイルの内容全体を置き換えます。

1. `Todo` コンポーネントに戻り、次のタスクを実行します。

   * Todo アイテム用のフィールドを `@code` ブロックに追加します。 `Todo` コンポーネントでは、このフィールドを使って ToDo リストの状態を維持します。
   * 各 Todo アイテムをリスト アイテム (`<li>`) としてレンダリングするために、順序のないリストのマークアップと `foreach` ループを追加します。

   `Pages/Todo.razor`:

   [!code-razor[](build-a-blazor-app/samples_snapshot/Todo2.razor?highlight=5-10,13)]

1. アプリには、リストに Todo 項目を追加するための UI 要素が必要です。 順序のないリスト (`<ul>...</ul>`) の下に、テキスト入力 (`<input>`) とボタン (`<button>`) を追加します。

   [!code-razor[](build-a-blazor-app/samples_snapshot/Todo3.razor?highlight=12-13)]

1. `TodoItem.cs` ファイルと更新された `Pages/Todo.razor` ファイルを保存します。 コマンド シェルでは、ファイルが保存されるとアプリが自動的に再構築されます。 ブラウザーからアプリへの接続は一時的に失われ、接続が再確立されるとページが再度読み込まれます。

1. **`Add todo`** ボタンを選択しても何も起こりません。ボタンにイベント ハンドラーがアタッチされていないためです。

1. `Todo` コンポーネントに `AddTodo` メソッドを追加し、`@onclick` 属性を使用して、そのメソッドをボタン用に登録します。 ボタンを選択すると C# のメソッド `AddTodo` が呼び出されます。

   [!code-razor[](build-a-blazor-app/samples_snapshot/Todo4.razor?highlight=2,7-10)]

1. 新しい Todo アイテムのタイトルを取得するには、`@code` ブロックの先頭に `newTodo` 文字列フィールドを追加します。

   [!code-razor[](build-a-blazor-app/samples_snapshot/Todo5.razor?highlight=3)]

   `@bind` 属性を使用して `newTodo` をバインドするようにテキストの `<input>` 要素を変更します。

   ```razor
   <input placeholder="Something todo" @bind="newTodo" />
   ```

1. 指定したタイトルを備えた `TodoItem` をリストに追加するように、`AddTodo` メソッドを更新します。 `newTodo` を空の文字列に設定して、テキスト入力の値をクリアします。

   [!code-razor[](build-a-blazor-app/samples_snapshot/Todo6.razor?highlight=19-26)]

1. `Pages/Todo.razor` ファイルを保存します。 このアプリは、コマンド シェルで自動的にリビルドされます。 ブラウザーからアプリに再接続された後、ページがブラウザーに再度読み込まれます。

1. 各 Todo アイテムのタイトルのテキストは編集可能にすることができます。また、チェック ボックスはユーザーが完了したアイテムを追跡するのに役立ちます。 各 Todo アイテムにチェック ボックス入力を追加し、その値を `IsDone` プロパティにバインドします。 `@todo.Title` を、`@bind` で `todo.Title` にバインドされた `<input>` 要素に変更します。

   [!code-razor[](build-a-blazor-app/samples_snapshot/Todo7.razor?highlight=4-7)]

1. `<h3>` ヘッダーを更新し、完了していない (`IsDone` が `false` の) Todo アイテムの数のカウントを表示するようにします。

   ```razor
   <h3>Todo (@todos.Count(todo => !todo.IsDone))</h3>
   ```

1. 完成した `Todo` コンポーネント (`Pages/Todo.razor`):

   [!code-razor[](build-a-blazor-app/samples_snapshot/Todo1.razor)]

1. `Pages/Todo.razor` ファイルを保存します。 このアプリは、コマンド シェルで自動的にリビルドされます。 ブラウザーからアプリに再接続された後、ページがブラウザーに再度読み込まれます。

1. アイテムを追加し、アイテムを編集し、Todo アイテムに完了のマークを付けて、コンポーネントをテストします。

1. 完了したら、コマンド シェルでアプリをシャットダウンします。 多くのコマンド シェルでは、<kbd>Ctrl</kbd>+<kbd>c</kbd> キーボード コマンドを使用してアプリを停止できます。

## <a name="next-steps"></a>次の手順

このチュートリアルでは、次の作業を行う方法を学びました。

> [!div class="checklist"]
> * Todo リストの Blazor アプリ プロジェクトを作成する
> * Razor コンポーネントを変更する
> * コンポーネントでイベント処理とデータ バインディングを使用する
> * Blazor アプリでルーティングを使用する

ASP.NET Core Blazor 用のツールについて学習します。

> [!div class="nextstepaction"]
> <xref:blazor/tooling>
