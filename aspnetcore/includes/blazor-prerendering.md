---
---
Blazor サーバー アプリをプリレンダリングしている間、ブラウザーとの接続が確立されていないため、JavaScript への呼び出しなどの特定のアクションは実行できません。 コンポーネントは、プリレンダリング時に異なるレンダリングが必要になる場合があります。

ブラウザーとの接続が確立されるまで JavaScript の相互運用呼び出しを遅らせるために、[OnAfterRenderAsync コンポーネント ライフサイクル イベント](xref:blazor/lifecycle#after-component-render)を使用できます。 このイベントは、アプリが完全にレンダリングされ、クライアント接続が確立された後にのみ呼び出されます。

```cshtml
@using Microsoft.JSInterop
@inject IJSRuntime JSRuntime

<div @ref="divElement">Text during render</div>

@code {
    private ElementReference divElement;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            await JSRuntime.InvokeVoidAsync(
                "setElementText", divElement, "Text after render");
        }
    }
}
```

前のコード例では、*wwwroot/index.html* (Blazor WebAssembly) または *Pages/_Host.cshtml* (Blazor サーバー) の `<head>` 要素内で、`setElementText` JavaScript 関数を提供します。 関数は `IJSRuntime.InvokeVoidAsync` を指定して呼び出され、値を返しません。

```html
<script>
  window.setElementText = (element, text) => element.innerText = text;
</script>
```

> [!WARNING]
> 前の例では、デモンストレーションのみを目的として、ドキュメント オブジェクト モデル (DOM) を直接変更しています。 ほとんどのシナリオにおいて、JavaScript で DOM を直接変更することはお勧めできません。JavaScript が Blazor の変更追跡に影響する可能性があるためです。

次のコンポーネントは、プリレンダリングと互換性のある方法で、コンポーネントの初期化ロジックの一部として JavaScript の相互運用を使用する方法を示しています。 コンポーネントには、`OnAfterRenderAsync` 内からレンダリングの更新をトリガーできることが示されています。 このシナリオでは、開発者が無限ループを作成しないようにする必要があります。

`JSRuntime.InvokeAsync` が呼び出されるとき、`ElementRef` は、以前のライフサイクル メソッドではなく `OnAfterRenderAsync` でのみ使用されます。コンポーネントがレンダリングされるまで JavaScript 要素が存在しないためです。

[StateHasChanged](xref:blazor/lifecycle#state-changes) は、JavaScript の相互運用呼び出しから取得された新しい状態でコンポーネントを再度レンダリングするために呼び出されます。 `StateHasChanged` は `infoFromJs`が `null` である場合にのみ呼び出されるため、このコードで無限ループが作成されることはありません。

```cshtml
@page "/prerendered-interop"
@using Microsoft.AspNetCore.Components
@using Microsoft.JSInterop
@inject IJSRuntime JSRuntime

<p>
    Get value via JS interop call:
    <strong id="val-get-by-interop">@(infoFromJs ?? "No value yet")</strong>
</p>

Set value via JS interop call:
<div id="val-set-by-interop" @ref="divElement"></div>

@code {
    private string infoFromJs;
    private ElementReference divElement;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender && infoFromJs == null)
        {
            infoFromJs = await JSRuntime.InvokeAsync<string>(
                "setElementText", divElement, "Hello from interop call!");

            StateHasChanged();
        }
    }
}
```

前のコード例では、*wwwroot/index.html* (Blazor WebAssembly) または *Pages/_Host.cshtml* (Blazor サーバー) の `<head>` 要素内で、`setElementText` JavaScript 関数を提供します。 関数は `IJSRuntime.InvokeAsync` を指定して呼び出され、次の値を返します。

```html
<script>
  window.setElementText = (element, text) => {
    element.innerText = text;
    return text;
  };
</script>
```

> [!WARNING]
> 前の例では、デモンストレーションのみを目的として、ドキュメント オブジェクト モデル (DOM) を直接変更しています。 ほとんどのシナリオにおいて、JavaScript で DOM を直接変更することはお勧めできません。JavaScript が Blazor の変更追跡に影響する可能性があるためです。
