---
title: ASP.NET Core Blazor レイアウト
author: guardrex
description: Blazor アプリの再利用可能なレイアウト コンポーネントを作成する方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/12/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/layouts
ms.openlocfilehash: 09cca9c4af23c35fdbc2ee92169913c960b0a68d
ms.sourcegitcommit: 69e1a79a572b0af17d08e81af12c594b7316f2e1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/15/2020
ms.locfileid: "83424331"
---
# <a name="aspnet-core-blazor-layouts"></a>ASP.NET Core Blazor レイアウト

作成者: [Rainer Stropek](https://www.timecockpit.com)、[Luke Latham](https://github.com/guardrex)

メニュー、著作権メッセージ、会社のロゴなどの一部のアプリ要素は、通常、アプリの全体のレイアウトの一部であり、アプリのすべてのコンポーネントで使用されます。 これらの要素のコードをアプリのすべてのコンポーネントにコピーするのは、効率的な方法ではありません。 要素の 1 つに更新が必要になるたびに、すべてのコンポーネントを更新する必要があります。 このような複製を維持することは困難であり、時間の経過と共にコンテンツの一貫性が失われる可能性があります。 *レイアウト*によって、この問題を解決します。

技術的に、レイアウトはもう 1 つのコンポーネントにすぎません。 レイアウトは Razor テンプレートまたは C# コードで定義され、[データ バインディング](xref:blazor/data-binding)、[依存関係の挿入](xref:blazor/dependency-injection)、その他のコンポーネント シナリオを使用できます。

*コンポーネント*を*レイアウト*に変えるには、コンポーネントが:

* レイアウト内のレンダリングされるコンテンツの `Body` プロパティを定義する `LayoutComponentBase` から継承している。
* Razor 構文 `@Body` を使用して、コンテンツがレンダリングされるレイアウト マークアップ内の場所を指定している。

次のコード サンプルに、レイアウト コンポーネント *MainLayout.razor* の Razor テンプレートを示します。 レイアウトは `LayoutComponentBase` を継承し、ナビゲーション バーとフッターの間に `@Body` を設定します。

[!code-razor[](layouts/sample_snapshot/3.x/MainLayout.razor?highlight=1,13)]

Blazor アプリ テンプレートのいずれかに基づくアプリでは、`MainLayout` コンポーネント (*MainLayout.razor*) は、アプリの*共有*フォルダーにあります。

## <a name="default-layout"></a>既定のレイアウト

アプリの *App.razor* ファイル内の `Router` コンポーネントに、既定のアプリ レイアウトを指定します。 既定の Blazor テンプレートによって提供される次の `Router` コンポーネントは、既定のレイアウトを `MainLayout` コンポーネントに設定します。

[!code-razor[](layouts/sample_snapshot/3.x/App1.razor?highlight=3)]

`NotFound` コンテンツの既定のレイアウトを指定するには、`NotFound` コンテンツの `LayoutView` を指定します。

[!code-razor[](layouts/sample_snapshot/3.x/App2.razor?highlight=6-9)]

`Router` コンポーネントの詳細については、「<xref:blazor/routing>」を参照してください。

ルーターでレイアウトを既定のレイアウトとして指定することは、コンポーネントごとまたはフォルダーごとにオーバーライドできるため、便利な方法です。 ルーターを使用してアプリの既定のレイアウトを設定することは、最も一般的な技法であるため、お勧めします。

## <a name="specify-a-layout-in-a-component"></a>コンポーネントにレイアウトを指定する

コンポーネントにレイアウトを適用するには、Razor ディレクティブ `@layout` を使用します。 コンパイラでは `@layout` を `LayoutAttribute` に変換します。これはコンポーネント クラスに適用されます。

次の `MasterList` コンポーネントのコンテンツは、`@Body` の位置にある `MasterLayout` に挿入されます。

[!code-razor[](layouts/sample_snapshot/3.x/MasterList.razor?highlight=1)]

コンポーネントに直接レイアウトを指定すると、ルーターに設定された*既定のレイアウトまたは *_Imports.razor* からインポートされた `@layout` ディレクティブ*がオーバーライドされます。

## <a name="centralized-layout-selection"></a>一元的なレイアウトの選択

アプリのすべてのフォルダーには、必要に応じて、 *_Imports.razor* という名前のテンプレート ファイルを格納できます。 コンパイラにより、インポート ファイルに指定されたディレクティブが、同じフォルダー内とそのすべてのサブフォルダー内で再帰的にすべての Razor テンプレートに含まれます。 そのため、`@layout MyCoolLayout` を含む *_Imports.razor* ファイルにより、フォルダー内のすべてのコンポーネントで `MyCoolLayout` が確実に使用されます。 フォルダーおよびサブフォルダー内のすべての *.razor* ファイルに `@layout MyCoolLayout` を繰り返し追加する必要はありません。 `@using` ディレクティブは、同じようにコンポーネントにも適用されます。

次の *_Imports.razor* ファイルでは、次のものをインポートします。

* `MyCoolLayout`。
* 同じフォルダーおよびサブフォルダー内のすべての Razor コンポーネント。
* `BlazorApp1.Data` 名前空間。
 
[!code-razor[](layouts/sample_snapshot/3.x/_Imports.razor)]

*_Imports.razor* ファイルは、[Razor ビューおよびページに対する _ViewImports.cshtml ファイル](xref:mvc/views/layout#importing-shared-directives)に似ていますが、Razor コンポーネント ファイルに限定して適用されます。

*_Imports.razor* にレイアウトを指定すると、ルーターの*既定のレイアウト*として指定されたレイアウトがオーバーライドされます。

## <a name="nested-layouts"></a>入れ子になったレイアウト

アプリは、入れ子になったレイアウトで構成できます。 コンポーネントでは、別のレイアウトを参照するレイアウトを参照できます。 たとえば、複数レベルのメニュー構造を作成するために、レイアウトの入れ子を使用します。

次の例に、入れ子になったレイアウトの使用方法を示しています。 *EpisodesComponent.razor* ファイルは、表示するコンポーネントです。 コンポーネントは `MasterListLayout` を参照しています。

[!code-razor[](layouts/sample_snapshot/3.x/EpisodesComponent.razor?highlight=1)]

*MasterListLayout. razor* ファイルは `MasterListLayout` を提供します。 このレイアウトは、それがレンダリングされる別のレイアウト `MasterLayout` を参照しています。 `EpisodesComponent` は、`@Body` が表示される場所にレンダリングされます。

[!code-razor[](layouts/sample_snapshot/3.x/MasterListLayout.razor?highlight=1,9)]

最後に、*MasterLayout.razor* 内の `MasterLayout` に、ヘッダー、メイン メニュー、フッターなどの最上位レイアウト要素が含まれます。 `EpisodesComponent` を含む `MasterListLayout` は、`@Body` が表示される場所にレンダリングされます。

[!code-razor[](layouts/sample_snapshot/3.x/MasterLayout.razor?highlight=6)]

## <a name="share-a-razor-pages-layout-with-integrated-components"></a>統合コンポーネントと Razor Pages レイアウトを共有する

ルーティング可能なコンポーネントが Razor Pages アプリに統合されている場合、コンポーネントでアプリの共有レイアウトを使用できます。 詳細については、「<xref:blazor/integrate-components>」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:mvc/views/layout>
