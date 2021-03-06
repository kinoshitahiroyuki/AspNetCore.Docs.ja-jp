---
title: .NET Core での gRPC クライアント ファクトリの統合
author: jamesnk
description: クライアント ファクトリを使用して gRPC クライアントを作成する方法について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 11/12/2019
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: grpc/clientfactory
ms.openlocfilehash: 42b786b9a4d9b422ccf92d7a329979894a35b275
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/04/2020
ms.locfileid: "82774718"
---
# <a name="grpc-client-factory-integration-in-net-core"></a>.NET Core での gRPC クライアント ファクトリの統合

gRPC と `HttpClientFactory` の統合により、gRPC クライアントを一元的に作成する方法が提供されています。 これは、[スタンドアロンの gRPC クライアント インスタンスを構成する](xref:grpc/client)ための代替手段として使用できます。 ファクトリの統合は、[Grpc.Net.ClientFactory](https://www.nuget.org/packages/Grpc.Net.ClientFactory) NuGet パッケージで提供されています。

ファクトリには以下のような利点があります。

* 論理 gRPC クライアント インスタンスの構成を一元管理する場所となります
* 基になる `HttpClientMessageHandler` の存続期間を管理します
* ASP.NET Core gRPC サービスで期限とキャンセルを自動伝達

## <a name="register-grpc-clients"></a>gRPC クライアントを登録する

gRPC クライアントを登録するには、ジェネリック `AddGrpcClient` 拡張メソッドを `Startup.ConfigureServices` 内で使用して、gRPC の型指定されたクライアント クラスとサービス アドレスを指定します。

```csharp
services.AddGrpcClient<Greeter.GreeterClient>(o =>
{
    o.Address = new Uri("https://localhost:5001");
});
```

GRPC クライアントの種類は、依存関係の挿入 (DI) がある一時的なものとして登録されます。 これで、DI によって作成される種類でクライアントを直接挿入し、使用できるようになります。 ASP.NET Core MVC コントローラー、SignalR ハブ、および gRPC サービスは、gRPC クライアントを自動的に挿入できる場所です。

```csharp
public class AggregatorService : Aggregator.AggregatorBase
{
    private readonly Greeter.GreeterClient _client;

    public AggregatorService(Greeter.GreeterClient client)
    {
        _client = client;
    }

    public override async Task SayHellos(HelloRequest request,
        IServerStreamWriter<HelloReply> responseStream, ServerCallContext context)
    {
        // Forward the call on to the greeter service
        using (var call = _client.SayHellos(request))
        {
            await foreach (var response in call.ResponseStream.ReadAllAsync())
            {
                await responseStream.WriteAsync(response);
            }
        }
    }
}
```

## <a name="configure-httpclient"></a>HttpClient を構成する

`HttpClientFactory` では、gRPC クライアントによって使用される `HttpClient` が作成されます。 標準の `HttpClientFactory` メソッドを使用して、送信要求ミドルウェアを追加したり、`HttpClientHandler` の基になる `HttpClient` を構成したりすることができます。

```csharp
services
    .AddGrpcClient<Greeter.GreeterClient>(o =>
    {
        o.Address = new Uri("https://localhost:5001");
    })
    .ConfigurePrimaryHttpMessageHandler(() =>
    {
        var handler = new HttpClientHandler();
        handler.ClientCertificates.Add(LoadCertificate());
        return handler;
    });
```

詳細については、[IHttpClientFactory を使用した HTTP 要求の作成](xref:fundamentals/http-requests)に関するページを参照してください。

## <a name="configure-channel-and-interceptors"></a>チャネルとインターセプターを構成する

gRPC 固有のメソッドは、以下のために使用できます。

* gRPC クライアントの基になるチャネルを構成します。
* gRPC 呼び出しを行うときにクライアントが使用する `Interceptor` インスタンスを追加します。

```csharp
services
    .AddGrpcClient<Greeter.GreeterClient>(o =>
    {
        o.Address = new Uri("https://localhost:5001");
    })
    .AddInterceptor(() => new LoggingInterceptor())
    .ConfigureChannel(o =>
    {
        o.Credentials = new CustomCredentials();
    });
```

## <a name="deadline-and-cancellation-propagation"></a>期限とキャンセルの伝達

gRPC サービスでファクトリによって作成された gRPC クライアントは、`EnableCallContextPropagation()` を使用して、期限とキャンセル トークンが子の呼び出しに自動伝達されるように構成できます。 `EnableCallContextPropagation()` 拡張メソッドは [Grpc.AspNetCore.Server.ClientFactory](https://www.nuget.org/packages/Grpc.AspNetCore.Server.ClientFactory) NuGet パッケージで提供されています。

呼び出しコンテキストの伝達は、現在の gRPC 要求コンテキストから期限とキャンセル トークンを読み取り、それらを、gRPC クライアントによって行われた送信呼び出しに自動的に伝達することによって機能します。 呼び出しコンテキストの伝達は、複雑で入れ子になった gRPC のシナリオで、常に期限とキャンセルが確実に伝達されるようにする優れた方法です。

```csharp
services
    .AddGrpcClient<Greeter.GreeterClient>(o =>
    {
        o.Address = new Uri("https://localhost:5001");
    })
    .EnableCallContextPropagation();
```

期限と RPC のキャンセルの詳細については、「[RPC ライフサイクル](https://www.grpc.io/docs/guides/concepts/#rpc-life-cycle)」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:grpc/client>
* <xref:fundamentals/http-requests>
