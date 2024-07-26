[![Pull Request Validation](https://github.com/elastic/elastic-otel-dotnet/actions/workflows/ci.yml/badge.svg)](https://github.com/elastic/elastic-otel-dotnet/actions/workflows/ci.yml)

<!--
Goal of this section:
Provide all the information a user needs to determine if the product
is a good enough fit for their use case to merit further exploration.

Assumptions we're comfortable making about the reader:
* They are familiar with OpenTelemetry, Elastic, or both
-->

# Elastic Distribution for OpenTelemetry .NET

> [!IMPORTANT]
> The Elastic Distribution for OpenTelemetry .NET is not yet recommended for production use. Functionality may be changed or removed in future releases. Alpha releases are not subject to the support SLA of official GA features.
>
> We welcome your feedback! You can reach us by [opening a GitHub issue](https://github.com/elastic/elastic-otel-dotnet/issues) or starting a discussion thread on the [Elastic Discuss forum](https://discuss.elastic.co/tags/c/observability/apm/58/dotnet).

<!-- ✅ What is it? -->
The Elastic Distribution for OpenTelemetry .NET provides a zero code change extension
to [OpenTelemetry SDK for .NET](https://opentelemetry.io/docs/languages/net). These extensions ensure
a smooth and rich out of the box experience with [Elastic Observability](https://www.elastic.co/observability)
through strictly OpenTelemetry native means.

<!-- ✅ Why use it? -->
This ensures there are no new concepts to learn, and the full OpenTelemetry ecosystem remains at your
fingertips. Read more about the concept of [OpenTelemetry Distributions](https://opentelemetry.io/docs/concepts/distributions).

The Elastic Distribution for OpenTelemetry .NET includes some Elastic-specific processors to ensure the best
compatibility when exporting OpenTelemetry signal data [Elastic Observability](https://www.elastic.co/observability).
The distribution also preconfigures the collection of tracing, metrics and logs signals, applying
some opinionated defaults, such as which sources are collected by default. The distribution also
ensures that the OTLP exporter is enabled by default.

<!-- ✅ How to use it? -->
Use the distro to start the OpenTelemetry SDK with your .NET application to automatically capture tracing data, performance metrics, and logs. Traces, metrics, and logs are sent to any OTLP collector you choose.

Start with helpful defaults to begin collecting and exporting OpenTelemetry signals quickly. Then, further refine how you use the distro using extension methods that allow you to fully control the creation of the underlying tracer and metric providers.

After you start sending data to Elastic, use an [Elastic Observability](https://www.elastic.co/guide/en/observability/current/index.html) deployment -- hosted on Elastic Cloud or on-premises -- to monitor your applications, create alerts, and quickly identify root causes of service issues.

## Get started with the Elastic Distribution for OpenTelemetry .NET

This guide shows you how to use the Elastic Distribution for OpenTelemetry .NET to instrument your .NET application and start sending OpenTelemetry data to an Elastic Observability deployment.

**Already familiar with OpenTelemetry?** It's an explicit goal of this distribution to introduce **no new concepts** outside those defined by the wider OpenTelemetry community.

**New to OpenTelemetry?** This section will guide you through the _minimal_ configuration options to get the Elastic distro set up in your application. You do _not_ need any existing experience with OpenTelemetry to set up the Elastic distro initially. If you need more control over your configuration after getting set up, you can learn more in the [OpenTelemetry SDK documentation](https://opentelemetry.io/docs/languages/net).

### Prerequisites

**Check your .NET SDK version.** The current documentation and examples are written with .NET 6 and newer applications in mind. Before continuing, ensure that you have a supported [.NET SDK version](https://dotnet.microsoft.com/en-us/download/dotnet) installed locally.

**Set up Elastic Observability.** You'll need somewhere to send the gathered OpenTelemetry data so it can be viewed and analyzed. You can use an existing Elastic Observability cloud deployment or set up a new one.

<details>
<summary><strong>Expand for setup instructions</strong></summary>
To create your first Elastic Observability deployment:

1. Sign up for a [free Elastic Cloud trial](https://cloud.elastic.co/registration) or sign into an existing account.
1. Go to <https://cloud.elastic.co/home>.
1. Click **Create deployment**.
1. When the deployment is ready, click **Open** to visit your Kibana home page (for example, `https://{DEPLOYMENT_NAME}.kb.{REGION}.cloud.es.io/app/home#/getting_started`).
</details>

<!-- in case there are existing links to this section -->
<div id="installation"></div>

### Install

To get started with the Elastic OpenTelemetry Distribution for .NET, add the `Elastic.OpenTelemetry` NuGet package reference to your project file:

```xml
<PackageReference Include="Elastic.OpenTelemetry" Version="<LATEST>" />
```

Be sure to replace the `<LATEST>` placeholder with the latest available package from [NuGet.org](https://www.nuget.org/packages/Elastic.OpenTelemetry).

After adding the package reference, you can start using the distro in your application.

> [!NOTE]
> The distro includes a transitive dependency on the OpenTelemetry SDK, so you do not _need_ to add the
> OpenTelemetry SDK package to your project directly. However, you _can_ explicitly add the OpenTelemetry SDK
> as a dependency if you  want to opt into newer SDK versions. If you do this, the OpenTelemetry SDK dependency
> must be defined _before_ the Elastic OpenTelemetry Distribution for .NET is defined.

> [!NOTE]
> The distro is designed to be easy to use and integrate into your applications.
> This includes applications that have previously used the OpenTelemetry SDK directly.
> In situations where the OpenTelemetry SDK is already used, the only required change is
> to add the [`Elastic.OpenTelemetry`](https://www.nuget.org/packages/Elastic.OpenTelemetry)
> NuGet package to the project. Doing so will automatically switch to the opinionated configuration
> provided by the Elastic distro.

<!-- in case there are existing links to this section -->
<div id="aspnet-core-usage"></div>

### Send data from an ASP.NET Core application

The OpenTelemetry SDK and the Elastic Distribution for OpenTelemetry .NET provide extension
methods to enable observability features in your application with a few lines of code.

It's common to want to instrument ASP.NET Core applications based on the `Microsoft.Extensions.Hosting`
libraries, which provide dependency injection via an `IServiceProvider`.

If you want to instrument an ASP.NET Core minimal API application using the distro, follow the
steps below. Similar steps can also be used to instrument other ASP.NET Core workloads and other
host-based applications like [worker services](https://learn.microsoft.com/en-us/dotnet/core/extensions/workers).

> [!NOTE]
> The examples below assume your code uses the top-level statements feature introduced in C# 9.0
> and the default choice for applications created using the latest templates.

#### Add dependencies

To take advantage of the OpenTelemetry SDK instrumentation for ASP.NET Core:

1. add the following NuGet package to your project:

    ```xml
    <PackageReference Include="OpenTelemetry.Instrumentation.AspNetCore" Version="<LATEST>" />
    ```

    > [!NOTE]
    > Replace the `<LATEST>` placeholder with the latest available package from [NuGet.org](https://www.nuget.org/packages/OpenTelemetry.Instrumentation.AspNetCore).

    This package includes instrumentation to collect traces for requests handled by ASP.NET Core endpoints.

    > [!NOTE]
    > The ASP.NET Core instrumentation is not included by default in the distro.
    > As with all optional instrumentation libraries, you can choose to include them in your application by
    > adding a suitable package reference.


1. Inside the `Program.cs` file of the ASP.NET Core application, add the following two `using` directives:

    ```csharp
    using OpenTelemetry;
    using OpenTelemetry.Trace;
    ```

    The OpenTelemetry SDK provides extension methods on the `IServiceCollection` to enable the providers
    and configure the SDK. The distro overrides the default OpenTelemetry SDK registration, adding several
    opinionated defaults.

1. In the minimal API template, the `WebApplicationBuilder` exposes a `Services` property that can be used to register services with the dependency injection container. To enable tracing and metrics collection, ensure that the OpenTelemetry SDK is registered:

    ```csharp
    var builder = WebApplication.CreateBuilder(args);

    builder.Services
        /**
        The `AddHttpClient` method registers the `IHttpClientFactory`
        service with the dependency injection container. This is NOT
        required to enable OpenTelemetry, but the example endpoint will
        use it to send an HTTP request.
        */
	    .AddHttpClient()
        /**
        The `AddOpenTelemetry` method registers the OpenTelemetry SDK with
        the dependency injection container. When available, the Elastic
        Distribution for OpenTelemetry .NET will override this to add
        opinionated defaults.
        */
	    .AddOpenTelemetry()
        /** Configure tracing to instrument requests handled by ASP.NET Core. */
		    .WithTracing(t => t.AddAspNetCoreInstrumentation());
    ```

With these limited changes to the `Program.cs` file, the application is now configured to use the
OpenTelemetry SDK and the distro to collect traces and metrics, which are exported via the
OpenTelemetry protocol (OTLP).

#### Add a sample endpoint

To demonstrate the tracing capabilities, add a simple endpoint to the application:

```csharp
app.MapGet("/", async (IHttpClientFactory httpClientFactory) =>
{
	using var client = httpClientFactory.CreateClient();

	await Task.Delay(100);
  // Using this URL will require two redirects,
  // allowing us to see multiple spans in the trace.
	var response = await client.GetAsync("http://elastic.co");
	await Task.Delay(50);

	return response.StatusCode == System.Net.HttpStatusCode.OK ? Results.Ok() : Results.StatusCode(500);
});
```

The distro will automatically enable the exporting of signals via the OTLP exporter. This
exporter requires that endpoint(s) are configured. A common mechanism for configuring
endpoints is via environment variables.

#### Configure the distro

To configure the distro, at a minimum you'll need the deployment's OTLP endpoint and
authorization data to set the appropriate `OTLP_*` environment variables:

* `OTEL_EXPORTER_OTLP_ENDPOINT`: The full URL of the endpoint where data will be sent.
* `OTEL_EXPORTER_OTLP_HEADERS`: A comma-separated list of `key=value` pairs that will
be added to the headers of every request. This is typically this is used for
authentication information.

> [!NOTE]
> The steps below assume you are using an Elastic Cloud deployment as the destination
> for your observability data.

You can find the values of these variables in Kibana's APM tutorial.
In Kibana:

1. Go to **Setup guides**.
1. Select **Observability**.
1. Select **Monitor my application performance**.
1. Scroll down and select the **OpenTelemetry** option.
1. The appropriate values for `OTEL_EXPORTER_OTLP_ENDPOINT` and `OTEL_EXPORTER_OTLP_HEADERS` are shown there.

    ![Elastic Cloud OpenTelemetry configuration](https://raw.githubusercontent.com/elastic/elastic-otel-dotnet/main/docs/images/elastic-cloud-opentelemetry-configuration.png)

    For example:

    ```sh
    export OTEL_EXPORTER_OTLP_ENDPOINT=https://my-deployment.apm.us-west1.gcp.cloud.es.io
    export OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer P....l"
    ```

1. Configure environment variables for the application either in `launchSettings.json` or in the environment
where the application is running.
1. Once configured, run the application and make a request to the root endpoint. A trace will be generated
and exported to the OTLP endpoint.

### Send data from an application using Microsoft.Extensions.Hosting

For console applications, services, etc that are written against a builder that exposes an `IServiceCollection`
you can install this package:

```xml
<PackageReference Include="Elastic.OpenTelemetry" Version="<LATEST>" />
```
> **_NOTE:_** Replace the `<LATEST>` placeholder with the latest available package from
[NuGet.org](https://www.nuget.org/packages/Elastic.OpenTelemetry).

Ensure you call `AddOpenTelemetry` to enable OpenTelemetry just as you would when using OpenTelemetry directly.
Our package intercepts this call to set up our defaults, but can be further build upon as per usual:

```csharp
var builder = Host.CreateApplicationBuilder(args);

builder.Services.AddOpenTelemetry()
	.ConfigureResource(r => r.AddService(serviceName: "MyService"))
	.WithTracing(t => t.AddSource(Worker.ActivitySourceName).AddConsoleExporter())
	.WithMetrics(m => m.AddMeter(Worker.MeterName).AddConsoleExporter());
```

### Send data using manual instrumentation

In environments where an `IServiceCollection` is unavailable you may manually start instrumenting by creating
an instance of `ElasticOpenTelemetryBuilder`.

```csharp
await using var session = new ElasticOpenTelemetryBuilder()
    .WithTracing(b => b.AddSource(ActivitySourceName))
    .Build();
```

This will setup instrumentation for as long as `session` is not disposed. We would generally expect the `session`
to live for the life of the application.

`ElasticOpenTelemetryBuilder` is an implementation of [`IOpenTelemetryBuilder`](https://github.com/open-telemetry/opentelemetry-dotnet/blob/70657395b82ba00b8a1e848e8832b77dff94b6d2/src/OpenTelemetry.Api.ProviderBuilderExtensions/IOpenTelemetryBuilder.cs#L12).

This is important to know because any instrumentation configuration is automatically exposed by the base
OpenTelemetry package as extension methods on `IOpenTelemetryBuilder`. You will not lose functionality by
using our builder.

### Confirm that the distro is working

To confirm that the distro has successfully connected to Elastic:

1. In Kibana, go to **Observability** → **Applications** → **Traces**.
1. You should see the name of the service to which you just added the distro.
It can take several minutes after setting up the distro for the service to show up in this list.
1. Click on the name of the service in the list to see traces.

  ![Minimal API request trace sample in the Elastic APM UI](https://raw.githubusercontent.com/elastic/elastic-otel-dotnet/main/docs/images/trace-sample-minimal-api.png)

## Advanced Configuration

For a full reference of available configuration options, refer to [Configuration options](docs/configure.md).
