---
title: ASP.NET Core Blazor ライフサイクル
author: guardrex
description: ASP.NET Core Blazor アプリで Razor コンポーネント ライフサイクル メソッドを使用する方法について学習します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/07/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/lifecycle
ms.openlocfilehash: e4fcd86b6e6a84d9e34a83688f9fb80c6907e5f3
ms.sourcegitcommit: e20653091c30e0768c4f960343e2c3dd658bba13
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/16/2020
ms.locfileid: "83438916"
---
# <a name="aspnet-core-blazor-lifecycle"></a>ASP.NET Core Blazor ライフサイクル

著者: [Luke Latham](https://github.com/guardrex)、[Daniel Roth](https://github.com/danroth27)

Blazor フレームワークには、同期と非同期のライフサイクル メソッドが含まれています。 コンポーネントの初期化およびレンダリング中にコンポーネントで追加の操作を実行するには、ライフサイクル メソッドをオーバーライドします。

## <a name="lifecycle-methods"></a>ライフサイクル メソッド

### <a name="component-initialization-methods"></a>コンポーネントの初期化メソッド

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> および <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> は、コンポーネントが、その親コンポーネントから初期パラメーターを受け取った後で初期化されるときに呼び出されます。 コンポーネントが非同期操作を実行し、操作の完了時に更新する必要がある場合は、`OnInitializedAsync` を使用します。

同期操作の場合は、`OnInitialized` をオーバーライドします。

```csharp
protected override void OnInitialized()
{
    ...
}
```

非同期操作を実行するには、`OnInitializedAsync` をオーバーライドし、操作で `await` キーワードを使用します。

```csharp
protected override async Task OnInitializedAsync()
{
    await ...
}
```

[コンテンツをプリレンダリングする ](xref:blazor/hosting-model-configuration#render-mode)Blazor サーバー アプリは、`OnInitializedAsync` を "**_2 回_**" 呼び出します。

* コンポーネントが最初にページの一部として静的にレンダリングされるときに 1 回。
* ブラウザーがサーバーへの接続を確立するときに 2 回目。

`OnInitializedAsync` 内で開発者コードが 2 回実行されないようにするには、「[プリレンダリング後のステートフル再接続](#stateful-reconnection-after-prerendering)」セクションを参照してください。

Blazor サーバー アプリをプリレンダリングしている間、ブラウザーとの接続が確立されていないため、JavaScript への呼び出しなどの特定のアクションは実行できません。 コンポーネントは、プリレンダリング時に異なるレンダリングが必要になる場合があります。 詳細については、「[アプリがプリレンダリングされていることを検出する](#detect-when-the-app-is-prerendering)」セクションを参照してください。

イベント ハンドラーが設定されている場合は、破棄時にそれらをアンフックします。 詳細については、「[IDisposable を使用したコンポーネントの破棄](#component-disposal-with-idisposable)」セクションを参照してください。

### <a name="before-parameters-are-set"></a>パラメーターが設定される前

<xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> は、レンダリング ツリーのコンポーネントの親によって指定されたパラメーターを設定します。

```csharp
public override async Task SetParametersAsync(ParameterView parameters)
{
    await ...

    await base.SetParametersAsync(parameters);
}
```

<xref:Microsoft.AspNetCore.Components.ParameterView> には、`SetParametersAsync` が呼び出されるたびに、パラメーター値のセット全体が含まれます。

`SetParametersAsync` の既定の実装では、対応する値が `ParameterView` 内にある `[Parameter]` または `[CascadingParameter]` 属性を使用して、各プロパティの値を設定します。 対応する値が `ParameterView` 内にないパラメーターは、変更されないままになります。

`base.SetParametersAync` が呼び出されない場合、カスタム コードでは、必要に応じて受信パラメーター値を解釈できます。 たとえば、受信したパラメーターをクラスのプロパティに割り当てる必要はありません。

イベント ハンドラーが設定されている場合は、破棄時にそれらをアンフックします。 詳細については、「[IDisposable を使用したコンポーネントの破棄](#component-disposal-with-idisposable)」セクションを参照してください。

### <a name="after-parameters-are-set"></a>パラメーターが設定された後

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> と <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSet%2A> が呼び出されます。

* コンポーネントが初期化され、その親コンポーネントからパラメーターの最初のセットを受け取ったとき。
* 親コンポーネントが再レンダリングし、次のものを提供するとき:
  * 少なくとも 1 つのパラメーターが変更された既知のプリミティブ不変型のみ。
  * 任意の複合型のパラメーター。 フレームワークは、複合型のパラメーターの値が内部で変更されているかどうかを認識できないため、パラメーター セットは変更済みとして扱われます。

```csharp
protected override async Task OnParametersSetAsync()
{
    await ...
}
```

> [!NOTE]
> パラメーターとプロパティ値を適用するときの非同期処理は、`OnParametersSetAsync` ライフサイクル イベント中に発生する必要があります。

```csharp
protected override void OnParametersSet()
{
    ...
}
```

イベント ハンドラーが設定されている場合は、破棄時にそれらをアンフックします。 詳細については、「[IDisposable を使用したコンポーネントの破棄](#component-disposal-with-idisposable)」セクションを参照してください。

### <a name="after-component-render"></a>コンポーネントのレンダリング後

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> および <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> は、コンポーネントのレンダリングが完了した後に呼び出されます。 この時点で、要素およびコンポーネント参照が設定されます。 レンダリングされた DOM 要素を操作するサードパーティ製の JavaScript ライブラリをアクティブ化するなど、レンダリングされたコンテンツを使用して追加の初期化手順を行うには、この段階を使用します。

`OnAfterRenderAsync` と `OnAfterRender` の `firstRender` パラメーター:

* コンポーネント インスタンスを初めて表示するときに `true` に設定されます。
* 初期化作業が確実に 1 回だけ実行されるように使用できます。

```csharp
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
    {
        await ...
    }
}
```

> [!NOTE]
> `OnAfterRenderAsync` ライフサイクル イベント中に、レンダリング直後の非同期作業が発生する必要があります。
>
> `OnAfterRenderAsync` から <xref:System.Threading.Tasks.Task> を返した場合でも、フレームワークでは、そのタスクが完了しても、コンポーネントに対してさらにレンダリング サイクルがスケジュールされることはありません。 これは、無限のレンダリング ループを回避するためです。 返されたタスクが完了すると、さらにレンダリング サイクルをスケジュールする他のライフサイクル メソッドとは異なります。

```csharp
protected override void OnAfterRender(bool firstRender)
{
    if (firstRender)
    {
        ...
    }
}
```

`OnAfterRender` と `OnAfterRenderAsync` は、"*サーバー上でプリレンダリングするときには呼び出されません。* "

イベント ハンドラーが設定されている場合は、破棄時にそれらをアンフックします。 詳細については、「[IDisposable を使用したコンポーネントの破棄](#component-disposal-with-idisposable)」セクションを参照してください。

### <a name="suppress-ui-refreshing"></a>UI 更新の抑制

<xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> をオーバーライドして、UI の更新を抑制します。 実装によって `true` が返された場合は、UI が更新されます。

```csharp
protected override bool ShouldRender()
{
    var renderUI = true;

    return renderUI;
}
```

`ShouldRender` は、コンポーネントがレンダリングされるたびに呼び出されます。

`ShouldRender` がオーバーライドされる場合でも、コンポーネントは常に最初にレンダリングされます。

詳細については、「<xref:performance/blazor/webassembly-best-practices#avoid-unnecessary-component-renders>」を参照してください。

## <a name="state-changes"></a>状態変更

<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> は、状態が変更されたことをコンポーネントに通知します。 必要に応じて、`StateHasChanged` を呼び出すと、コンポーネントが再レンダリングされます。

## <a name="handle-incomplete-async-actions-at-render"></a>レンダリング時の不完全な非同期アクションを処理する

ライフサイクル イベントで実行される非同期アクションは、コンポーネントがレンダリングされる前に完了していない可能性があります。 ライフサイクル メソッドの実行中に、オブジェクトが `null` またはデータが不完全に設定されている可能性があります。 オブジェクトが初期化されていることを確認するレンダリング ロジックを提供します。 オブジェクトが `null` の間、プレースホルダー UI 要素 (読み込みメッセージなど) をレンダリングします。

Blazor テンプレートの `FetchData` コンポーネントでは、`OnInitializedAsync` はオーバーライドされ、予測データを非同期に受信します (`forecasts`)。 `forecasts` が `null` の場合、読み込みメッセージがユーザーに表示されます。 `OnInitializedAsync` によって返された `Task` が完了すると、コンポーネントは更新された状態で再レンダリングされます。

Blazor サーバー テンプレート内の *Pages/FetchData.razor*:

[!code-razor[](lifecycle/samples_snapshot/3.x/FetchData.razor?highlight=9,21,25)]

## <a name="component-disposal-with-idisposable"></a>IDisposable を使用したコンポーネントの破棄

コンポーネントが <xref:System.IDisposable> を実装している場合は、コンポーネントが UI から削除されると、[Dispose メソッド](/dotnet/standard/garbage-collection/implementing-dispose)が呼び出されます。 次のコンポーネントでは、`@implements IDisposable` および `Dispose` メソッドが使用されます。

```razor
@using System
@implements IDisposable

...

@code {
    public void Dispose()
    {
        ...
    }
}
```

> [!NOTE]
> `Dispose` では、<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> の呼び出しはサポートされていません。 `StateHasChanged` は、レンダラーの破棄の一部として呼び出されることがあるため、その時点での UI 更新の要求はサポートされていません。

.NET イベントからイベント ハンドラーのサブスクライブを解除します。 次の [Blazor フォーム](xref:blazor/forms-validation)の例は、`Dispose` メソッドでイベント ハンドラーをアンフックする方法を示しています。

* プライベート フィールドとラムダのアプローチ

  [!code-razor[](lifecycle/samples_snapshot/3.x/event-handler-disposal-1.razor?highlight=23,28)]

* プライベート メソッドのアプローチ

  [!code-razor[](lifecycle/samples_snapshot/3.x/event-handler-disposal-2.razor?highlight=16,26)]

## <a name="handle-errors"></a>エラーの処理

ライフサイクル メソッド実行中のエラー処理の詳細については、「<xref:blazor/handle-errors#lifecycle-methods>」を参照してください。

## <a name="stateful-reconnection-after-prerendering"></a>プリレンダリング後のステートフル再接続

Blazor サーバー アプリで `RenderMode` が `ServerPrerendered` の場合、コンポーネントは最初にページの一部として静的にレンダリングされます。 ブラウザーがサーバーへの接続を確立すると、コンポーネントが "*再度*" レンダリングされ、コンポーネントがやりとりできるようになります。 コンポーネントを初期化するための [OnInitialized{Async}](#component-initialization-methods) ライフサイクル メソッドが存在する場合、メソッドは "*2 回*" 実行されます。

* コンポーネントが静的にプリレンダリングされたとき。
* サーバー接続が確立された後。

これにより、コンポーネントが最終的にレンダリングされるときに、UI に表示されるデータが大幅に変わる可能性があります。

Blazor サーバー アプリ内の二重レンダリングのシナリオを回避するには、次の手順を行います。

* プリレンダリング中に状態をキャッシュし、アプリの再起動後に状態を取得するために使用できる識別子を渡します。
* 識別子をプリレンダリング中に使用して、コンポーネントの状態を保存します。
* 識別子をプリレンダリング後に使用して、キャッシュされた状態を取得します。

次のコードは、二重レンダリングを回避するテンプレートベースの Blazor サーバー アプリ内で更新される `WeatherForecastService` を示しています。

```csharp
public class WeatherForecastService
{
    private static readonly string[] summaries = new[]
    {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild",
        "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };
    
    public WeatherForecastService(IMemoryCache memoryCache)
    {
        MemoryCache = memoryCache;
    }
    
    public IMemoryCache MemoryCache { get; }

    public Task<WeatherForecast[]> GetForecastAsync(DateTime startDate)
    {
        return MemoryCache.GetOrCreateAsync(startDate, async e =>
        {
            e.SetOptions(new MemoryCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = 
                    TimeSpan.FromSeconds(30)
            });

            var rng = new Random();

            await Task.Delay(TimeSpan.FromSeconds(10));

            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = startDate.AddDays(index),
                TemperatureC = rng.Next(-20, 55),
                Summary = summaries[rng.Next(summaries.Length)]
            }).ToArray();
        });
    }
}
```

`RenderMode` の詳細については、「<xref:blazor/hosting-model-configuration#render-mode>」を参照してください。

## <a name="detect-when-the-app-is-prerendering"></a>アプリがプリレンダリングされていることを検出する

[!INCLUDE[](~/includes/blazor-prerendering.md)]

## <a name="cancelable-background-work"></a>取り消し可能なバックグラウンド作業

ネットワーク呼び出し (<xref:System.Net.Http.HttpClient>) の実行やデータベースとの対話など、コンポーネントによって実行時間の長いバックグラウンド作業が実行されることがよくあります。 いくつかの状況でシステム リソースを節約するために、バックグラウンド作業を停止することをお勧めします。 たとえば、ユーザーがコンポーネントの操作を止めても、バックグラウンドの非同期操作は自動的に停止しません。

バックグラウンド作業項目の取り消しが必要になるその他の理由には、次のようなものがあります。

* 実行中のバックグラウンド タスクは、不完全な入力データまたは処理パラメーターを使用して開始されました。
* 現在実行中のバックグラウンド作業項目のセットを、新しい作業項目のセットに置き換える必要があります。
* 現在実行中のタスクの優先度を変更する必要があります。
* アプリをサーバーに再展開するには、シャットダウンする必要があります。
* サーバー リソースが制限され、バックグラウンド作業項目の再スケジュールが必要になりました。

コンポーネントに取り消し可能なバックグラウンド作業パターンを実装するには:

* <xref:System.Threading.CancellationTokenSource> と <xref:System.Threading.CancellationToken> を使用します。
* [コンポーネントの破棄](#component-disposal-with-idisposable)時と、任意の時点で手動でトークンを取り消すことで取り消しが望まれた場合は、[CancellationTokenSource.Cancel](xref:System.Threading.CancellationTokenSource.Cancel%2A) を呼び出して、バックグラウンド作業を取り消す必要があることを通知します。
* 非同期呼び出しが返された後、トークンに対して <xref:System.Threading.CancellationToken.ThrowIfCancellationRequested%2A> を呼び出します。

次に例を示します。

* `await Task.Delay(5000, cts.Token);` は、実行時間が長い非同期のバックグラウンド作業を表します。
* `BackgroundResourceMethod` は、メソッドが呼び出される前に `Resource` が破棄された場合に開始されない、実行時間が長いバックグラウンド メソッドを表します。

```razor
@implements IDisposable
@using System.Threading

<button @onclick="LongRunningWork">Trigger long running work</button>

@code {
    private Resource resource = new Resource();
    private CancellationTokenSource cts = new CancellationTokenSource();

    protected async Task LongRunningWork()
    {
        await Task.Delay(5000, cts.Token);

        cts.Token.ThrowIfCancellationRequested();
        resource.BackgroundResourceMethod();
    }

    public void Dispose()
    {
        cts.Cancel();
        cts.Dispose();
        resource.Dispose();
    }

    private class Resource : IDisposable
    {
        private bool disposed;

        public void BackgroundResourceMethod()
        {
            if (disposed)
            {
                throw new ObjectDisposedException(nameof(Resource));
            }
            
            ...
        }
        
        public void Dispose()
        {
            disposed = true;
        }
    }
}
```
