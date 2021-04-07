---
title: 'チュートリアル: 複合データ モデルを作成する - ASP.NET MVC と EF Core'
description: このチュートリアルでは、エンティティとリレーションシップをさらに追加し、書式設定、検証、マッピングの規則を指定してデータ モデルをカスタマイズします。
author: rick-anderson
ms.author: riande
ms.custom: mvc
ms.date: 03/27/2019
ms.topic: tutorial
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
uid: data/ef-mvc/complex-data-model
ms.openlocfilehash: bbb783b8547ffec0cfad124f7c0e8f1a21e1e727
ms.sourcegitcommit: 4bbc69f51c59bed1a96aa46f9f5dca2f2a2634cb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/25/2021
ms.locfileid: "105555072"
---
# <a name="tutorial-create-a-complex-data-model---aspnet-mvc-with-ef-core"></a>チュートリアル: 複合データ モデルを作成する - ASP.NET MVC と EF Core

前のチュートリアルでは、3 つのエンティティで構成された単純なデータ モデルを使用して作業を行いました。 このチュートリアルでは、エンティティとリレーションシップをさらに追加し、書式設定、検証、データベース マッピングの規則を指定してデータ モデルをカスタマイズします。

完了すると、エンティティ クラスは、以下の図のように完成したデータ モデルを構成します。

![エンティティ図](complex-data-model/_static/diagram.png)

このチュートリアルでは、次の作業を行いました。

> [!div class="checklist"]
> * データ モデルをカスタマイズする
> * Student エンティティに変更を加える
> * Instructor エンティティを作成する
> * OfficeAssignment エンティティを作成する
> * Course エンティティを変更する
> * Department エンティティを作成する
> * Enrollment エンティティを変更する
> * データベース コンテキストを更新する
> * テスト データを使ってデータベースをシードする
> * 移行を追加する
> * 接続文字列を変更する
> * データベースを更新する

## <a name="prerequisites"></a>必須コンポーネント

* [EF Core 移行を使用する](migrations.md)

## <a name="customize-the-data-model"></a>データ モデルをカスタマイズする

このセクションでは、書式設定、検証、データベース マッピング規則を指定する属性を使用して、データ モデルをカスタマイズする方法を示します。 その後、次のいくつかのセクションで、作成済みのクラスに属性を追加し、モデルの残りのエンティティ型に対して新しいクラスを作成して、完全な School データ モデルを作成します。

### <a name="the-datatype-attribute"></a>DataType 属性

学生の登録日について、すべての Web ページでは現在、日付と共に時刻が表示されていますが、このフィールドでは日付が重要になります。 データ注釈属性を使用すれば、1 つのコードを変更するだけで、データを表示するすべてのビューの表示形式を修正できます。 その方法例を表示するには、`Student` クラスの `EnrollmentDate` プロパティに属性を追加します。

*Models/Student.cs* で、次の例のように、`System.ComponentModel.DataAnnotations` 名前空間の `using` ステートメントを追加し、`DataType` および `DisplayFormat` 属性を `EnrollmentDate` プロパティに追加します。

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_DataType&highlight=3,12-13)]

`DataType` 属性は、データベースの組み込み型よりも具体的なデータ型を指定するために使用されます。 この例では、追跡する必要があるのは、日付と時刻ではなく、日付のみです。 `DataType` 列挙型は、Date、Time、PhoneNumber、Currency、EmailAddress など、多くの型のために用意されています。 また、`DataType` 属性を使用して、アプリケーションで型固有の機能を自動的に提供することもできます。 たとえば、`mailto:` リンクを `DataType.EmailAddress` に作成したり、HTML5 をサポートするブラウザーで `DataType.Date` に日付セレクターを提供したりできます。 `DataType` 属性は、HTML 5 ブラウザーが認識できる HTML 5 `data-` ("データ ダッシュ" と読みます) 属性を出力します。 `DataType` 属性では検証が提供されません。

`DataType.Date` は、表示される日付の書式を指定しません。 既定では、データ フィールドはサーバーの CultureInfo に基づき、既定の書式に従って表示されます。

`DisplayFormat` 属性は、日付の書式を明示的に指定するために使用されます。

```csharp
[DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
```

`ApplyFormatInEditMode` の設定では、編集用にテキスト ボックスに値を表示するときにも適用する必要がある書式設定を指定します (フィールドによっては適用したくないこともあります。たとえば、通貨値では、編集用テキスト ボックスには通貨記号が必要でない場合があります)。

`DisplayFormat` 属性を単独で使用できますが、一般的に、`DataType` 属性も使用することをお勧めします。 `DataType` 属性は、画面でのレンダリング方法とは異なり、データのセマンティクスを伝達します。また、`DisplayFormat` にはない、次の利点があります。

* ブラウザーは HTML5 機能を有効にすることができます (たとえば、カレンダー コントロール、ロケールに適した通貨記号、メール リンク、一部のクライアント側の入力検証を表示するときなど)。

* ブラウザーの既定では、ロケールに基づいて正しい書式を使ってデータがレンダリングされます。

詳細については、[\<input> タグ ヘルパーに関するドキュメント](../../mvc/views/working-with-forms.md#the-input-tag-helper)を参照してください。

アプリを実行し、Students インデックス ページに移動すると、登録日の時刻が表示されなくなっていることがわかります。 Student モデルを使用するどのビューでも同様です。

![時刻なしの日付が表示されている Students インデックス ページ](complex-data-model/_static/dates-no-times.png)

### <a name="the-stringlength-attribute"></a>StringLength 属性

属性を使用して、データ検証規則と検証エラー メッセージを指定することもできます。 `StringLength` 属性では、データベース内の最大長が設定され、ASP.NET Core MVC に対するクライアント側とサーバー側の検証が提供されます。 この属性で最小長を指定することもできますが、最小値はデータベース スキーマに影響しません。

たとえば、ユーザーが 50 文字を超える名前を入力しないようにする必要があるとします。 この制限を追加するには、次の例のように、`StringLength` 属性を `LastName` および `FirstMidName` プロパティに追加します。

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_StringLength&highlight=10,12)]

`StringLength` 属性では、ユーザーが名前に空白を入力しないようにすることはできません。 `RegularExpression` 属性を使用して、入力に制限を適用することができます。 たとえば、次のコードでは、最初の文字を大文字にし、残りの文字をアルファベット順にすることを要求します。

```csharp
[RegularExpression(@"^[A-Z]+[a-zA-Z]*$")]
```

`MaxLength` 属性は、`StringLength` 属性と同様の機能を提供しますが、クライアント側の検証は提供しません。

データベース モデルは、現在、データベース スキーマでの変更が必要な方法で変更されています。 移行を使用すれば、アプリケーション UI を使用してデータベースに追加した可能性のあるデータを失うことなく、スキーマを更新できます。

変更を保存し、プロジェクトをビルドします。 次に、プロジェクト フォルダーでコマンド ウィンドウを開き、次のコマンドを入力します。

```dotnetcli
dotnet ef migrations add MaxLengthOnNames
```

```dotnetcli
dotnet ef database update
```

`migrations add` コマンドはデータ損失が発生する可能性があることを警告します。これは、変更により、2 つの列の最大長が短くなるためです。  移行時に *\<timeStamp>_MaxLengthOnNames.cs* という名前のファイルが作成されます。 このファイルには、現在のデータ モデルと一致するようにデータベースを更新する `Up` メソッドのコードが含まれます。 `database update` コマンドでそのコードが実行されました。

移行ファイル名の前に付けられる timestamp は、移行を並べ替えるために Entity Framework によって使用されます。 update-database コマンドを実行する前に複数の移行を作成できます。その後、すべての移行は作成順に適用されます。

アプリを実行し、 **[Students]** タブを選択して、 **[新規作成]** をクリックし、50 文字を超えるいずれかの名前を入力してみます。 アプリケーションにより、この操作が防止されます。 

### <a name="the-column-attribute"></a>Column 属性

属性を使用して、データベースへのクラスとプロパティのマッピング方法を制御することもできます。 フィールドにミドル ネームも含まれている場合があるため、名フィールドに対して `FirstMidName` という名前を使用したとします。 ただし、データベース列は `FirstName` という名前にする必要があります。これは、データベースに対するアドホック クエリを記述するユーザーがその名前に慣れているためです。 このマッピングを作成する場合、`Column` 属性を使用できます。

`Column` 属性は、データベースの作成時に、`FirstMidName` プロパティにマップする `Student` テーブルの列が `FirstName` という名前になるように指定します。 つまり、コードが `Student.FirstMidName` を参照したときに、データが `Student` テーブルの `FirstName` 列から取り込まれるか、更新されます。 列名を指定しない場合は、プロパティ名と同じ名前が付けられます。

*Student.cs* ファイルで、以下の強調表示されているコードに示されているように、`System.ComponentModel.DataAnnotations.Schema` の `using` ステートメントを追加し、列名属性を `FirstMidName` プロパティに追加します。

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_Column&highlight=4,14)]

`Column` 属性を追加すると、`SchoolContext` をサポートするモデルが変更されるため、データベースと一致しなくなります。

変更を保存し、プロジェクトをビルドします。 次に、プロジェクト フォルダーでコマンド ウィンドウを開き、次のコマンドを入力して別の移行を作成します。

```dotnetcli
dotnet ef migrations add ColumnFirstName
```

```dotnetcli
dotnet ef database update
```

**SQL Server オブジェクト エクスプローラー** で、**Student** テーブルをダブルクリックして、Student テーブル デザイナーを開きます。

![移行後の SSOX の Students テーブル](complex-data-model/_static/ssox-after-migration.png)

最初の 2 つの移行を適用する前の名前列の型は nvarchar(MAX) でした。 現在は nvarchar(50) になっており、列名は FirstMidName から FirstName に変更されています。

> [!Note]
> 次のセクションですべてのエンティティ クラスの作成を完了する前にコンパイルしようとすると、コンパイラ エラーが発生する可能性があります。

## <a name="changes-to-student-entity"></a>Student エンティティに対する変更

![Student エンティティ](complex-data-model/_static/student-entity.png)

*Models/Student.cs* で、前の手順で追加したコードを以下のコードに置き換えます。 変更が強調表示されます。

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_BeforeInheritance&highlight=11,13,15,18,22,24-31)]

### <a name="the-required-attribute"></a>Required 属性

`Required` 属性では、名前プロパティの必須フィールドを作成します。 値の型 (DateTime、int、double、float など) などの null 非許容型では `Required` 属性は必要ありません。 null にできない型は自動的に必須フィールドとして扱われます。

`MinimumLength` を適用するには、`Required` 属性を `MinimumLength` と共に使用する必要があります。

```csharp
[Display(Name = "Last Name")]
[Required]
[StringLength(50, MinimumLength=2)]
public string LastName { get; set; }
```

### <a name="the-display-attribute"></a>Display 属性

`Display` 属性は、テキスト ボックスのキャプションを、各インスタンスのプロパティ名 (単語を区切るスペースがない) の代わりに、"First Name"、"Last Name"、"Full Name"、"Enrollment Date" にするよう指定します。

### <a name="the-fullname-calculated-property"></a>FullName 集計プロパティ

`FullName` は集計プロパティであり、2 つの別のプロパティを連結して作成される値を返します。 したがって、get アクセサーのみが存在し、データベースで `FullName` 列は生成されません。

## <a name="create-instructor-entity"></a>Instructor エンティティを作成する

![Instructor エンティティ](complex-data-model/_static/instructor-entity.png)

*Models/Instructor.cs* を作成し、テンプレート コードを以下のコードに置き換えます。

[!code-csharp[](intro/samples/cu/Models/Instructor.cs?name=snippet_BeforeInheritance)]

Student および Instructor エンティティに同じプロパティがいくつかあることに注目してください。 このシリーズの後半の[継承の実装](inheritance.md)に関するチュートリアルでは、冗長さをなくすため、このコードをリファクタリングします。

複数の属性 1 行に配置して、次のように `HireDate` 属性を記述することもできます。

```csharp
[DataType(DataType.Date),Display(Name = "Hire Date"),DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
```

### <a name="the-courseassignments-and-officeassignment-navigation-properties"></a>CourseAssignments と OfficeAssignment ナビゲーション プロパティ

`CourseAssignments` と `OfficeAssignment` プロパティはナビゲーション プロパティです。

講師は任意の数のコースを担当できるため、`CourseAssignments` はコレクションとして定義されます。

```csharp
public ICollection<CourseAssignment> CourseAssignments { get; set; }
```

ナビゲーション プロパティで複数のエンティティを保持できる場合は、その型を、エンティティを追加、削除、更新できるリストにする必要があります。  `ICollection<T>`、または `List<T>` や `HashSet<T>` などの型を指定することができます。 `ICollection<T>` を指定した場合、EF では既定で `HashSet<T>` コレクションが作成されます。

これらが `CourseAssignment` エンティティである理由は、多対多リレーションシップに関する以下のセクションで説明します。

Contoso University のビジネス ルールには、講師は 1 つのオフィスのみを持つことができると示されているため、`OfficeAssignment` プロパティでは単一の OfficeAssignment エンティティが保持されます (オフィスが割り当てられていない場合は null である可能性がある)。

```csharp
public OfficeAssignment OfficeAssignment { get; set; }
```

## <a name="create-officeassignment-entity"></a>OfficeAssignment エンティティを作成する

![OfficeAssignment エンティティ](complex-data-model/_static/officeassignment-entity.png)

以下のコードを使用して、*Models/OfficeAssignment.cs* を作成します。

[!code-csharp[](intro/samples/cu/Models/OfficeAssignment.cs)]

### <a name="the-key-attribute"></a>Key 属性

`Instructor` エンティティと `OfficeAssignment` エンティティの間には一対ゼロまたは一対一のリレーションシップがあります。 オフィスが割り当てられている講師についてのみ、オフィス割り当てが存在します。したがって、その主キーは `Instructor` エンティティに対する外部キーでもあります。 ただし、名前が `ID` や `classnameID` 名前付け規則に従っていないため、Entity Framework では `InstructorID` をこのエンティティの主キーとして自動的に認識できません。 したがって、`Key` 属性はキーとして識別するために使用されます。

```csharp
[Key]
public int InstructorID { get; set; }
```

エンティティに独自の主キーはあっても、プロパティに classnameID や ID 以外の名前を付けたい場合は、`Key` 属性を使用することもできます。

列は依存リレーションシップに対するものであるため、既定では EF はキーを非データベース生成として扱います。

### <a name="the-instructor-navigation-property"></a>Instructor ナビゲーション プロパティ

Instructor エンティティには null 許容の `OfficeAssignment` ナビゲーション プロパティがあり (講師にオフィスが割り当てられていない場合があるため)、OfficeAssignment エンティティには null 非許容の `Instructor` ナビゲーション プロパティがあります (講師なしではオフィス割り当てが存在できない、つまり、`InstructorID` が null 非許容であるため)。 Instructor エンティティに関連する OfficeAssignment エンティティがある場合、各エンティティにはそのナビゲーション プロパティの別のエンティティへの参照があります。

Instructor ナビゲーション プロパティに `[Required]` 属性を配置して、関連する講師が存在するように指定できますが、`InstructorID` 外部キー (このテーブルに対するキーでもある) は null 非許容であるため、そうする必要はありません。

## <a name="modify-course-entity"></a>Course エンティティを変更する

![Course エンティティ](complex-data-model/_static/course-entity.png)

*Models/Course.cs* で、前の手順で追加したコードを以下のコードに置き換えます。 変更が強調表示されます。

[!code-csharp[](intro/samples/cu/Models/Course.cs?name=snippet_Final&highlight=2,10,13,16,19,21,23)]

Course エンティティには外部キー プロパティ `DepartmentID` (関連する Department エンティティを指す) があり、`Department` ナビゲーション プロパティがあります。

Entity Framework では、関連エンティティのナビゲーション プロパティがある場合、ユーザーがデータ モデルに外部キー プロパティを追加する必要はありません。  EF は必要に応じて、データベースで外部キーを自動的に作成し、[シャドウ プロパティ](/ef/core/modeling/shadow-properties)を作成します。 ただし、データ モデルに外部キーがある場合は、更新をより簡単かつ効率的に行うことができます。 たとえば、編集する `Course` エンティティをフェッチするときに読み込まないと、`Department` エンティティは null になります。したがって、`Course` エンティティを更新する場合は、まず、`Department` エンティティをフェッチする必要があります。 外部キー プロパティ `DepartmentID` がデータ モデルに含まれている場合は、更新前に `Department` エンティティをフェッチする必要はありません。

### <a name="the-databasegenerated-attribute"></a>DatabaseGenerated 属性

`CourseID` プロパティで `DatabaseGenerated` 属性と `None` パラメーターを使用すると、主キー値がデータベースによって生成されるのではなく、ユーザーによって提供されるように指定されます。

```csharp
[DatabaseGenerated(DatabaseGeneratedOption.None)]
[Display(Name = "Number")]
public int CourseID { get; set; }
```

既定では、Entity Framework は、主キー値がデータベースによって生成されることを前提とします。 これはほとんどのシナリオに該当します。 ただし、`Course` エンティティの場合、1 つの学科に 1000 シリーズ、別の学科に 2000 シリーズといったユーザー指定のコース番号を使用します。

行の作成日または更新日を記録するために使用されるデータベース列の場合のように、`DatabaseGenerated` 属性は既定値を生成するためにも使用できます。  詳細については、「[生成される値](/ef/core/modeling/generated-properties)」を参照してください。

### <a name="foreign-key-and-navigation-properties"></a>外部キー プロパティとナビゲーション プロパティ

`Course` エンティティの外部キー プロパティとナビゲーション プロパティには、以下のリレーションシップが反映されます。

コースは 1 つの学科に割り当てられます。したがって、前述の理由により、`DepartmentID` 外部キーと `Department` ナビゲーション プロパティが存在します。

```csharp
public int DepartmentID { get; set; }
public Department Department { get; set; }
```

コースには任意の数の学生が登録できるため、`Enrollments` ナビゲーション プロパティはコレクションとなります。

```csharp
public ICollection<Enrollment> Enrollments { get; set; }
```

複数の講師が 1 つのコースを担当する場合があるため、`CourseAssignments` ナビゲーション プロパティはコレクションとなります (`CourseAssignment` 型については、[後で](#many-to-many-relationships)説明します)。

```csharp
public ICollection<CourseAssignment> CourseAssignments { get; set; }
```

## <a name="create-department-entity"></a>Department エンティティを作成する

![Department エンティティ](complex-data-model/_static/department-entity.png)

以下のコードを使用して、*Models/Department.cs* を作成します。

[!code-csharp[](intro/samples/cu/Models/Department.cs?name=snippet_Begin)]

### <a name="the-column-attribute"></a>Column 属性

これまでは、`Column` 属性を使用して、列名のマッピングを変更しました。 `Department` エンティティのコードでは、`Column` 属性は SQL データ型のマッピングを変更するために使用されているため、列はデータベースの SQL Server `money` 型を使用して定義されます。

```csharp
[Column(TypeName="money")]
public decimal Budget { get; set; }
```

通常、列のマッピングは必要ありません。これは、Entity Framework が、プロパティに対して定義された CLR 型に基づいて、適切な SQL Server のデータ型を選択するためです。 CLR `decimal` 型は SQL Server の `decimal` 型にマップされます。 ただし、ここでは、列に通貨額が保持されることがわかっているため、money データ型がより適しています。

### <a name="foreign-key-and-navigation-properties"></a>外部キー プロパティとナビゲーション プロパティ

外部キーおよびナビゲーション プロパティには、次のリレーションシップが反映されます。

学科には管理者が存在する場合とそうでない場合があり、管理者は常に講師となります。 したがって、`InstructorID` プロパティは Instructor エンティティに対する外部キーとして含まれ、プロパティを null 許容とマークするために `int` 型の表記の後に疑問符が追加されます。 ナビゲーション プロパティは `Administrator` という名前ですが、Instructor エンティティを保持します。

```csharp
public int? InstructorID { get; set; }
public Instructor Administrator { get; set; }
```

学科には複数のコースがある場合があるため、Courses ナビゲーション プロパティがあります。

```csharp
public ICollection<Course> Courses { get; set; }
```

> [!NOTE]
> 規則により、Entity Framework では null 非許容の外部キーと多対多リレーションシップに対して連鎖削除が有効になります。 これにより、循環連鎖削除規則が適用される可能性があり、移行を追加しようとすると例外が発生します。 たとえば、`Department.InstructorID` プロパティを null 許容として定義しなかった場合、EF では、講師を削除したときに学科を削除するように連鎖削除規則が構成されます。これは、発生してほしくない動作です。 ビジネス ルールで `InstructorID` プロパティを null 非許容にすることが求められた場合、以下の fluent API ステートメントを使用して、リレーションシップで連鎖削除を無効にする必要がありました。
>
> ```csharp
> modelBuilder.Entity<Department>()
>    .HasOne(d => d.Administrator)
>    .WithMany()
>    .OnDelete(DeleteBehavior.Restrict)
> ```

## <a name="modify-enrollment-entity"></a>Enrollment エンティティを変更する

![Enrollment エンティティ](complex-data-model/_static/enrollment-entity.png)

*Models/Enrollment.cs* で、前の手順で追加したコードを以下のコードに置き換えます。

[!code-csharp[](intro/samples/cu/Models/Enrollment.cs?name=snippet_Final&highlight=1-2,16)]

### <a name="foreign-key-and-navigation-properties"></a>外部キー プロパティとナビゲーション プロパティ

外部キー プロパティとナビゲーション プロパティには、次のリレーションシップが反映されます。

登録レコードは単一のコースに対するものであるため、`CourseID` 外部キー プロパティと `Course` ナビゲーション プロパティがあります。

```csharp
public int CourseID { get; set; }
public Course Course { get; set; }
```

登録レコードは 1 人の学生に対するものであるため、`StudentID` 外部キー プロパティと `Student` ナビゲーション プロパティがあります。

```csharp
public int StudentID { get; set; }
public Student Student { get; set; }
```

## <a name="many-to-many-relationships"></a>多対多リレーションシップ

`Student` および `Course` エンティティの間には多対多リレーションシップがあり、`Enrollment` エンティティはデータベースで "*ペイロードがある*" 多対多の結合テーブルとして機能します。 "ペイロードがある" とは、`Enrollment` テーブルに統合テーブルの外部キー以外に追加データが含まれていることを意味します (この例では、主キーと `Grade` プロパティ)。

次の図は、エンティティ図でこれらのリレーションシップがどのようになるかを示しています (この図は、EF 6.x 用の Entity Framework Power Tools を使用して生成されたものです。このチュートリアルでは図は作成しません。ここでは例として使用するだけです)。

![Student と Course の多対多リレーションシップ](complex-data-model/_static/student-course.png)

各リレーションシップ線の一方の端に 1 が、もう一方の端にアスタリスク (*) があり、1 対多リレーションシップであることを示しています。

`Enrollment` テーブルに成績情報が含まれていない場合、含める必要があるのは 2 つの外部キー `CourseID` および `StudentID` のみです。 その場合、データベースにはペイロードがない多対多結合テーブル (純粋結合テーブル) が存在することになります。 `Instructor` および `Course` エンティティにはその種の多対多リレーションシップがあり、次の手順では、ペイロードがない結合テーブルとして機能するエンティティ クラスを作成します。

(EF 6.x では多対多リレーションシップの暗黙の結合テーブルがサポートされますが、EF Core ではサポートされません。 詳細については、[GitHub リポジトリの EF Core に関する記述](https://github.com/aspnet/EntityFramework/issues/1368)を参照してください)。

## <a name="the-courseassignment-entity"></a>CourseAssignment エンティティ

![CourseAssignment エンティティ](complex-data-model/_static/courseassignment-entity.png)

以下のコードを使用して、*Models/CourseAssignment.cs* を作成します。

[!code-csharp[](intro/samples/cu/Models/CourseAssignment.cs)]

### <a name="join-entity-names"></a>結合エンティティの名前

講師対コースの多対多リレーションシップのデータベースには結合テーブルが必要であり、エンティティ セットで表す必要があります。 結合テーブルには `EntityName1EntityName2` という名前 (ここでは `CourseInstructor`) を付けるのが一般的です。 ただし、リレーションシップを説明する名前を選択することをお勧めします。 データ モデルは始めは単純なものであっても、後からペイロードのない結合で頻繁にペイロードが取得されるようになるため大きくなっていきます。 最初にわかりやすいエンティティ名を付けておけば、後で名前を変更する必要はありません。 結合エンティティでは、ビジネス ドメインに独自の自然な (場合によっては 1 単語の) 名前を指定することが理想的です。 たとえば、Books と Customers は Ratings を通じてリンクできます。 このリレーションシップの場合は、`CourseInstructor` ではなく `CourseAssignment` を選択することをお勧めます。

### <a name="composite-key"></a>複合キー

外部キーが null 許容ではなく、テーブルの各行を一意に識別するために組み合わせて使用される場合、個別の主キーは必要ありません。 `InstructorID` および `CourseID` プロパティは複合主キーとして機能する必要があります。 EF に対する複合主キーを識別する唯一の方法は、*fluent API* を使用することです (属性を使用して行うことはできません)。 次のセクションでは、複合主キーの構成方法を示します。

複合キーを使用すると、1 つのコースに対して複数の行を、また 1 人の講師に対して複数の行を使用できても、同じ講師とコースに対しては複数の行を使用できなくなります。 `Enrollment` 結合エンティティでは独自の主キーを定義するため、このような重複が考えられます。 このような重複を防ぐために、外部キー フィールドで一意のインデックスを追加するか、`CourseAssignment` と同様の複合主キーを使用して `Enrollment` を構成することができます。 詳細については、「[インデックス](/ef/core/modeling/indexes)」を参照してください。

## <a name="update-the-database-context"></a>データベース コンテキストを更新する

次の強調表示されているコードを *Data/SchoolContext.cs* ファイルに追加します。

[!code-csharp[](intro/samples/cu/Data/SchoolContext.cs?name=snippet_BeforeInheritance&highlight=15-18,25-31)]

このコードでは新しいエンティティが追加され、CourseAssignment エンティティの複合主キーが構成されます。

## <a name="about-a-fluent-api-alternative"></a>代替手段 fluent API について

`DbContext` クラスの `OnModelCreating` メソッドのキーでは、*fluent API* を使用して EF の動作を構成します。 API は "fluent" と呼ばれます。これは、[EF Core のドキュメント](/ef/core/modeling/#use-fluent-api-to-configure-a-model)の例に示されているように、多くの場合、一連のメソッド呼び出しを単一のステートメントにまとめて使用されるためです。

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .Property(b => b.Url)
        .IsRequired();
}
```

このチュートリアルでは、属性では実行できないデータベース マッピングでのみ fluent API を使用します。 ただし、fluent API を使用して、属性で実行できる書式設定、検証、およびマッピング規則のほとんどを指定することもできます。 `MinimumLength` などの一部の属性は fluent API で適用できません。 前述のとおり、`MinimumLength` はスキーマを変更せず、クライアント側とサーバー側の検証規則のみを適用します。

一部の開発者は fluent API のみを使用することを選ぶため、エンティティ クラスを "クリーン" な状態に保つことができます。 必要に応じて、属性と fluent API を組み合わせて使用することができます。fluent API のみを使用して実行できるカスタマイズがいくつかありますが、一般的は 2 つの方法のいずれかを選択して、できるだけ一貫性を保つためにそれを使用することをお勧めします。 両方の使用時に競合が発生する場合は、Fluent API で属性がオーバーライドされることに注意してください。

属性と fluent API の詳細については、「[構成の方法](/ef/core/modeling/)」を参照してください。

## <a name="entity-diagram-showing-relationships"></a>リレーションシップを示すエンティティ図

次の図では、完成した School モデルに対して Entity Framework Power Tools で作成される図を示します。

![エンティティ図](complex-data-model/_static/diagram.png)

一対多リレーションシップの線 (1 対 \*) 以外にも、ここには`Instructor` および `OfficeAssignment` エンティティ間の一対ゼロまたは一リレーションシップの線 (1 対 0..1) と、Instructor および Department エンティティ間のゼロまたは一対多リレーションシップの線 (0..1 対 *) があります。

## <a name="seed-database-with-test-data"></a>テスト データを使ってデータベースをシードする

*Data/DbInitializer.cs* ファイルのコードを以下のコードに置き換えて、作成した新しいエンティティのシード データを提供します。

[!code-csharp[](intro/samples/cu/Data/DbInitializer.cs?name=snippet_Final)]

最初のチュートリアルの説明のとおり、このコードのほとんどでは単に新しいエンティティ オブジェクトを作成し、テストで必要な場合にサンプル データをプロパティに読み込みます。 多対多リレーションシップがどのように処理されているかに注目してください。このコードでは、`Enrollments` および `CourseAssignment` 結合エンティティ セットでエンティティを作成して、リレーションシップを作成しています。

## <a name="add-a-migration"></a>移行を追加する

変更を保存し、プロジェクトをビルドします。 プロジェクト フォルダーでコマンド ウィンドウを開いて、次のように `migrations add` コマンドを入力します (update-database コマンドはまだ入力しないでください)。

```dotnetcli
dotnet ef migrations add ComplexDataModel
```

考えられるデータ損失に関する警告が表示されます。

```text
An operation was scaffolded that may result in the loss of data. Please review the migration for accuracy.
Done. To undo this action, use 'ef migrations remove'
```

この時点で `database update` コマンドを実行しようとした場合は (まだ実行しないでください)、以下のようなエラーが表示されます。

> ALTER TABLE ステートメントは FOREIGN KEY 制約 "FK_dbo.Course_dbo.Department_DepartmentID" と競合しています。 競合が発生したのは、データベース "ContosoUniversity"、テーブル "dbo.Department"、列 'DepartmentID' です。

場合によっては、既存のデータで移行を実行するときに、外部キー制約を満たすためにデータベースにスタブ データを挿入する必要があります。 `Up` メソッドの生成されたコードでは、`Course` テーブルに null 非許容の `DepartmentID` 外部キーが追加されます。 コードの実行時に Course テーブルに既に行がある場合、`AddColumn` 操作は失敗します。これは、SQL Server では、null にできない列に配置される値が認識されないためです。 このチュートリアルでは、新しいデータベースで移行を実行しますが、運用アプリケーションでは、移行時に既存のデータを処理する必要があるため、以下の手順ではその方法例を示します。

既存のデータでこの移行を行うには、コードを変更して、新しい列に既定値を設定し、"Temp" という名前のスタブ学科を作成して、既定の学科として機能するようにする必要があります。 その結果、既存の Course 行が、`Up` メソッドの実行後に "Temp" 学科に関連付けられます。

* *{timestamp}_ComplexDataModel.cs* ファイルを開きます。

* DepartmentID 列を Course テーブルに追加するコードの行をコメントアウトします。

  [!code-csharp[](intro/samples/cu/Migrations/20170215234014_ComplexDataModel.cs?name=snippet_CommentOut&highlight=9-13)]

* Department テーブルを作成するコードの後に、次の強調表示されたコードを追加します。

  [!code-csharp[](intro/samples/cu/Migrations/20170215234014_ComplexDataModel.cs?name=snippet_CreateDefaultValue&highlight=22-32)]

運用アプリケーションでは、コードまたはスクリプトを記述して、Department 行を追加し、Course 行を新しい Department 行に関連付けます。 これで、`Course.DepartmentID` 列の既定値や "Temp" 学科は不要になります。

変更を保存し、プロジェクトをビルドします。

## <a name="change-the-connection-string"></a>接続文字列を変更する

これで、新しいエンティティのシード データを空のデータベースに追加する `DbInitializer` クラスの新しいコードが準備できました。 EF で新しい空のデータベースを作成するには、 *appsettings.json* の接続文字列のデータベース名を ContosoUniversity3 に変更するか、使用しているコンピューターではまだ使用していない別の名前に変更します。

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=ContosoUniversity3;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
```

*appsettings.json* の変更内容を保存します。

> [!NOTE]
> データベース名を変更する代わりに、データベースを削除することもできます。 **SQL Server オブジェクト エクスプローラー** (SSOX) または `database drop` CLI コマンドを使用します。
>
> ```dotnetcli
> dotnet ef database drop
> ```

## <a name="update-the-database"></a>データベースを更新する

データベース名を変更またはデータベースを削除した後、コマンド ウィンドウで `database update` コマンドを実行して、移行を実行します。

```dotnetcli
dotnet ef database update
```

アプリを実行すると、`DbInitializer.Initialize` メソッドが実行され、新しいデータベースが設定されます。

前の手順で行ったように SSOX でデータベースを開き、**Tables** ノードを展開して、テーブルがすべて作成されたことを確認します (前に開いた SSOX がそのままの状態の場合は、 **[更新]** ボタンをクリックします)。

![SSOX のテーブル](complex-data-model/_static/ssox-tables.png)

アプリを実行して、データベースをシードする初期化子コードをトリガーします。

**CourseAssignment** テーブルを右クリックして、 **[データの表示]** を選択し、テーブルにデータがあることを確認します。

![SSOX の CourseAssignment データ](complex-data-model/_static/ssox-ci-data.png)

## <a name="get-the-code"></a>コードを取得する

[完成したアプリケーションをダウンロードまたは表示する。](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="next-steps"></a>次の手順

このチュートリアルでは、次の作業を行いました。

> [!div class="checklist"]
> * データ モデルをカスタマイズした
> * Student エンティティに変更を加えた
> * Instructor エンティティを作成した
> * OfficeAssignment エンティティを作成した
> * Course エンティティを変更した
> * Department エンティティを作成した
> * Enrollment エンティティを変更した
> * データベース コンテキストを更新した
> * テスト データを使ってデータベースをシードした
> * 移行を追加した
> * 接続文字列を変更した
> * データベースを更新した

関連データにアクセスする方法について確認するには、次のチュートリアルに進んでください。

> [!div class="nextstepaction"]
> [次へ: 関連データにアクセスする](read-related-data.md)
