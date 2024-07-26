////
Goal of this doc:
Provide a complete reference of all available configuration options and where/how they can be set. (Any Elastic-specific configuration options are listed directly. General OpenTelemetry configuration options are linked.)

Assumptions we're comfortable making about the reader:
* They are familiar with Elastic
* They are familiar with OpenTelemetry
////

# Configuration options

:language: .NET
:language_lc: dotnet
:distro_name: Elastic Distribution for OpenTelemetry {language}

include::release-status.asciidoc[]

////
✅ How users set configuration options
////
[discrete]
[[configure-methods]]
=== Configuration methods

Configure the OpenTelemetry SDK using the mechanisms listed in the https://opentelemetry.io/docs/languages/net/automatic/configuration/[OpenTelemetry documentation],
including:

* <<configure-environment-variables,Setting environment variables>>
* <<configure-iconfiguration-integration,Using the `IConfiguration` integration>>
* <<configure-manual-configuration,Manually configuring the Elastic distribution>>

// Order of precedence
Configuration options set manually in the code take precedence over environment variables, and
environment variables take precedence over configuration options set using the `IConfiguration` system.

[discrete]
[[configure-environment-variables]]
==== Environment variables

// ✅ What and why
The distro can be configured using environment variables.
This is a cross-platform way to configure the distro and is especially useful in containerized environments.

// ✅ How
Environment variables are read at startup and can be used to configure the Elastic distribution.
For details of the various options available and their corresponding environment variable names,
see <<configure-configuration-options>>.


[discrete]
[[configure-iconfiguration-integration]]
==== `IConfiguration` integration

// ✅ What and why
In applications that use the "host" pattern, such as ASP.NET Core and worker service, the distro
can be configured using the `IConfiguration` integration.

// ✅ How
This is done by passing an `IConfiguration` instance to the `AddElasticOpenTelemetry` extension
method on the `IServiceCollection`.

When using an `IHostApplicationBuilder` such as modern ASP.NET Core applications, the current `IConfiguration`
can be accessed via the `Configuration` property on the builder:

[source,csharp]
----
var builder = WebApplication.CreateBuilder(args);
var currentConfig = builder.Configuration; <1>
----
<1> Access the current `IConfiguration` instance from the builder.

By default, at this stage, the configuration will be populated from the default configuration sources,
including the `appsettings.json` file(s) and command-line arguments. You may use these sources to define
the configuration for the Elastic Distribution for OpenTelemetry .NET.

// ✅ Example
For example, you can define the configuration for the Elastic Distribution for OpenTelemetry .NET in the `appsettings.json` file:

[source,json]
----
{
  "Elastic": {
    "OpenTelemetry": {
      "FileLogDirectory": "C:\\Logs" <1>
    }
  }
}
----
<1> This example sets the file log directory to `C:\Logs` which enables diagnostic file logging.

Configuration from the "Elastic:OpenTelemetry" section of the `IConfiguration` instance will be
bound to the `ElasticOpenTelemetryOptions` instance used to configure the Elastic distribution.

To learn more about the Microsoft configuration system, see
https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration[Configuration in ASP.NET Core].

[discrete]
[[configure-manual-configuration]]
==== Manual configuration

// ✅ What and why
In all other scenarios, you can configure the distro manually in code.

// ✅ How
Create an instance of `ElasticOpenTelemetryBuilderOptions` and pass it to the `ElasticOpenTelemetryBuilder`
constructor or an overload of the `AddElasticOpenTelemetry` extension method on the `IServiceCollection`.

// ✅ Example
For example, in traditional console applications, you can configure the
Elastic Distribution for OpenTelemetry .NET like this:

[source,csharp]
----
using Elastic.OpenTelemetry;
using Elastic.OpenTelemetry.Configuration;
using Elastic.OpenTelemetry.Extensions;
using Microsoft.Extensions.DependencyInjection;
using OpenTelemetry;

var services = new ServiceCollection();

var builderOptions = new ElasticOpenTelemetryBuilderOptions <1>
{
	DistroOptions = new ElasticOpenTelemetryOptions <2>
	{
		FileLogDirectory = "C:\\Logs", <3>
	}
};

await using var session = new ElasticOpenTelemetryBuilder(builderOptions) <4>
	.WithTracing(b => b.AddSource("MySource"))
	.Build();
----
<1> Create an instance of `ElasticOpenTelemetryBuilderOptions`.
<2> Create an instance of `ElasticOpenTelemetryOptions` and configure the file log directory by
setting the corresponding property.
<3> This example sets the file log directory to `C:\Logs` which enables diagnostic file logging.
<4> Pass the `ElasticOpenTelemetryBuilderOptions` instance to the `ElasticOpenTelemetryBuilder` constructor
to configure the Elastic Distribution for OpenTelemetry .NET.

////
✅ List all available configuration options
////
[discrete]
[[configure-configuration-options]]
=== Configuration options

Because the {distro_name} ("the distro") is an extension of the https://github.com/open-telemetry/opentelemetry-{language_lc}-instrumentation[OpenTelemetry {language} agent], it supports both:

* General OpenTelemetry SDK configuration options
* Elastic-specific configuration options that are only available when using the distro

[discrete]
[[configure-otel-sdk-options]]
==== OpenTelemetry SDK configuration options

The distro supports all configuration options listed in the https://opentelemetry.io/docs/languages/sdk-configuration/general/[OpenTelemetry General SDK Configuration documentation].

[discrete]
[[configure-distro-options]]
==== Elastic-specific configuration options

The distro supports the following Elastic-specific options.

[discrete]
[[configure-filelogdirectory]]
===== `FileLogDirectory`

A string specifying the directory where the Elastic Distribution for OpenTelemetry .NET will write diagnostic log files.
When not provided, no file logging will occur. Each new .NET process will create a new log file in the
specified directory.

[%header]
|===
| Environment variable name | `IConfiguration` key
| `ELASTIC_OTEL_FILE_LOG_DIRECTORY` | `Elastic:OpenTelemetry:FileLogDirectory`
|===

[%header]
|===
| Default | Type
| `string.Empty` | String
|===

[discrete]
[[configure-fileloglevel]]
===== `FileLogLevel`

Sets the logging level for the distribution.

Valid options: `Critical`, `Error`, `Warning`, `Information`, `Debug`, `Trace` and `None` (`None` disables the logging).

[%header]
|===
| Environment variable name | `IConfiguration` key
| `ELASTIC_OTEL_FILE_LOG_LEVEL` | `Elastic:OpenTelemetry:FileLogLevel`
|===

[%header]
|===
| Default | Type
| `Information` | String
|===

[discrete]
[[configure-skipotlpexporter]]
===== `SkipOtlpExporter`

Allows the distribution to used with its defaults, but without enabling the export of telemetry data to
an OTLP endpoint. This can be useful when you want to test applications without sending telemetry data.

[%header]
|===
| Environment variable name | `IConfiguration` key
| `ELASTIC_OTEL_SKIP_OTLP_EXPORTER` | `Elastic:OpenTelemetry:SkipOtlpExporter`
|===

[%header]
|===
| Default | Type
| `false` | Bool
|===

[discrete]
[[config-elasticdefaults]]
===== `ElasticDefaults`

A comma-separated list of Elastic defaults to enable. This can be useful when you want to enable
only some of the Elastic Distribution for OpenTelemetry .NET opinionated defaults.

Valid options: `None`, `Traces`, `Metrics`, `Logs`, `All`.

Except for the `None` option, all other options can be combined.

When this setting is not configured or the value is `string.Empty`, all Elastic Distribution for OpenTelemetry .NET defaults will be enabled.

When `None` is specified, no Elastic Distribution for OpenTelemetry .NET defaults will be enabled, and you will need to manually
configure the OpenTelemetry SDK to enable collection of telemetry signals. In this mode, the Elastic distribution
does not provide any opinionated defaults, nor register any processors, allowing you to start with the "vanilla"
OpenTelemetry SDK configuration. You may then choose to configure the various providers and register processors
as required.

In all other cases, the Elastic Distribution for OpenTelemetry .NET will enable the specified defaults. For example, to enable only
Elastic defaults only for tracing and metrics, set this value to `Traces,Metrics`.

[%header]
|===
| Environment variable name | `IConfiguration` key
| `ELASTIC_OTEL_DEFAULTS_ENABLED` | `Elastic:OpenTelemetry:ElasticDefaults` |
|===

[%header]
|===
| Default | Type
| `string.Empty` | String
|===

////
✅ List auth methods
////
[discrete]
[[configure-auth-methods]]
=== Authentication methods

When sending data to Elastic, there are two ways you can authenticate: using an APM agent key or using a secret token.

[discrete]
[[configure-api-key]]
==== Use an APM agent key (API key)

// ✅ What is this?
It is also possible to authenticate to an Elastic Observability endpoint using
an {observability-guide}/apm-api-key.html[APM agent key].
// ✅ Why would someone use it?
APM agent keys are revocable, you can have more than one of them, and
you can add or remove them without restarting APM Server.

// ✅ How do you authenticate using this method?
To create and manage APM agent keys in {kib}:

. Go to *APM Settings*.
. Select the *Agent Keys* tab.

When using an APM agent key, the `OTEL_EXPORTER_OTLP_HEADERS` is set using a
different auth schema (`ApiKey` rather than `Bearer`). For example:

// ✅ Code example
[source,sh]
----
export OTEL_EXPORTER_OTLP_ENDPOINT=https://my-deployment.apm.us-west1.gcp.cloud.es.io
export OTEL_EXPORTER_OTLP_HEADERS="Authorization=ApiKey TkpXUkx...dVZGQQ=="
----

[discrete]
[[configure-secret-token]]
==== Use a secret token

// ✅ What is this
{observability-guide}/apm-secret-token.html[Secret tokens] are used to authorize requests to the APM Server.
// ✅ Why use this
Both the distro and APM Server must be configured with the same secret token for the request to be accepted.

// ✅ How do you authenticate using this method?
// tag::get-auth[]
You can find the values of these variables in {kib}'s APM tutorial.
In {kib}:

. Go to *Setup guides*.
. Select *Observability*.
. Select *Monitor my application performance*.
. Scroll down and select the *OpenTelemetry* option.
. The appropriate values for `OTEL_EXPORTER_OTLP_ENDPOINT` and
`OTEL_EXPORTER_OTLP_HEADERS` are shown there.
+
image:images/elastic-cloud-opentelemetry-configuration.png[Elastic Cloud OpenTelemetry configuration]
+
For example:
+
[source,sh]
----
export OTEL_EXPORTER_OTLP_ENDPOINT=https://my-deployment.apm.us-west1.gcp.cloud.es.io
export OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer P....l"
----
// end::get-auth[]

:!language:
:!language_lc:
:!distro_name: