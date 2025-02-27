---
title: .NET Aspire Azure Blob Storage integration
description: This article describes the .NET Aspire Azure Blob Storage integration features and capabilities.
ms.topic: how-to
ms.date: 08/12/2024
---

# .NET Aspire Azure Blob Storage integration

In this article, you learn how to use the .NET Aspire Azure Blob Storage integration. The `Aspire.Azure.Storage.Blobs` library is used to register a <xref:Azure.Storage.Blobs.BlobServiceClient> in the DI container for connecting to Azure Blob Storage. It also enables corresponding health checks, logging and telemetry.

## Get started

To get started with the .NET Aspire Azure Blob Storage integration, install the [📦 Aspire.Azure.Storage.Blobs](https://www.nuget.org/packages/Aspire.Azure.Storage.Blobs) NuGet package in the client-consuming project, i.e., the project for the application that uses the Azure Blob Storage client.

### [.NET CLI](#tab/dotnet-cli)

```dotnetcli
dotnet add package Aspire.Azure.Storage.Blobs
```

### [PackageReference](#tab/package-reference)

```xml
<PackageReference Include="Aspire.Azure.Storage.Blobs"
                  Version="*" />
```

---

For more information, see [dotnet add package](/dotnet/core/tools/dotnet-add-package) or [Manage package dependencies in .NET applications](/dotnet/core/tools/dependencies).

## Example usage

In the _:::no-loc text="Program.cs":::_ file of your client-consuming project, call the <xref:Microsoft.Extensions.Hosting.AspireBlobStorageExtensions.AddAzureBlobClient%2A> extension to register a `BlobServiceClient` for use via the dependency injection container.

```csharp
builder.AddAzureBlobClient("blobs");
```

You can then retrieve the `BlobServiceClient` instance using dependency injection. For example, to retrieve the client from a service:

```csharp
public class ExampleService(BlobServiceClient client)
{
    // Use client...
}
```

## App host usage

To add Azure Storage hosting support to your <xref:Aspire.Hosting.IDistributedApplicationBuilder>, install the [📦 Aspire.Hosting.Azure.Storage](https://www.nuget.org/packages/Aspire.Hosting.Azure.Storage) NuGet package in the [app host](xref:dotnet/aspire/app-host) project.

### [.NET CLI](#tab/dotnet-cli)

```dotnetcli
dotnet add package Aspire.Hosting.Azure.Storage
```

### [PackageReference](#tab/package-reference)

```xml
<PackageReference Include="Aspire.Hosting.Azure.Storage"
                  Version="*" />
```

---

In your app host project, register the Azure Blob Storage integration and consume the service using the following methods, such as <xref:Aspire.Hosting.AzureStorageExtensions.AddAzureStorage%2A>:

```csharp
var builder = DistributedApplication.CreateBuilder(args);

var blobs = builder.AddAzureStorage("storage")
                   .AddBlobs("blobs");

var exampleProject = builder.AddProject<Projects.ExampleProject>()
                            .WithReference(blobs);
```

The <xref:Aspire.Hosting.AzureStorageExtensions.AddBlobs%2A> method will read connection information from the AppHost's configuration (for example, from "user secrets") under the `ConnectionStrings:blobs` config key. The <xref:Aspire.Hosting.ResourceBuilderExtensions.WithReference%2A> method passes that connection information into a connection string named blobs in the `ExampleProject` project. In the _:::no-loc text="Program.cs":::_ file of `ExampleProject`, the connection can be consumed using:

```csharp
builder.AddAzureBlobClient("blobs");
```

## Configuration

The .NET Aspire Azure Blob Storage integration provides multiple options to configure the `BlobServiceClient` based on the requirements and conventions of your project.

### Use a connection string

When using a connection string from the `ConnectionStrings` configuration section, you can provide the name of the connection string when calling `builder.AddAzureBlobClient`:

```csharp
builder.AddAzureBlobClient("blobs");
```

And then the connection string will be retrieved from the `ConnectionStrings` configuration section. Two connection formats are supported:

#### Service URI

The recommended approach is to use a `ServiceUri`, which works with the <xref:Aspire.Azure.Storage.Blobs.AzureStorageBlobsSettings.Credential?displayProperty=nameWithType> property to establish a connection. If no credential is configured, the <xref:Azure.Identity.DefaultAzureCredential?displayProperty=fullName> is used.

```json
{
  "ConnectionStrings": {
    "blobsConnectionName": "https://{account_name}.blob.core.windows.net/"
  }
}
```

#### Connection string

Alternatively, an [Azure Storage connection string](/azure/storage/common/storage-configure-connection-string) can be used.

```json
{
  "ConnectionStrings": {
    "blobsConnectionName": "AccountName=myaccount;AccountKey=myaccountkey"
  }
}
```

### Use configuration providers

The .NET Aspire Azure Blob Storage integration supports <xref:Microsoft.Extensions.Configuration?displayProperty=fullName>. It loads the <xref:Aspire.Azure.Storage.Blobs.AzureStorageBlobsSettings> and <xref:Azure.Storage.Blobs.BlobClientOptions> from configuration by using the `Aspire:Azure:Storage:Blobs` key. Example _:::no-loc text="appsettings.json":::_ that configures some of the options:

```json
{
  "Aspire": {
    "Azure": {
      "Storage": {
        "Blobs": {
          "DisableHealthChecks": true,
          "DisableTracing": false,
          "ClientOptions": {
            "Diagnostics": {
              "ApplicationId": "myapp"
            }
          }
        }
      }
    }
  }
}
```

### Use inline delegates

You can also pass the `Action<AzureStorageBlobsSettings> configureSettings` delegate to set up some or all the options inline, for example to configure health checks:

```csharp
builder.AddAzureBlobClient(
    "blobs",
    static settings => settings.DisableHealthChecks  = true);
```

You can also set up the `BlobClientOptions` using `Action<IAzureClientBuilder<BlobServiceClient, BlobClientOptions>> configureClientBuilder` delegate, the second parameter of the `AddAzureBlobClient` method. For example, to set the first part of user-agent headers for all requests issues by this client:

```csharp
builder.AddAzureBlobClient(
    "blobs",
    static configureClientBuilder: clientBuilder =>
        clientBuilder.ConfigureOptions(
            static options => options.Diagnostics.ApplicationId = "myapp"));
```

[!INCLUDE [integration-health-checks](../includes/integration-health-checks.md)]

The .NET Aspire Azure Blob Storage integration handles the following:

- Adds the `AzureBlobStorageHealthCheck` health check, which attempts to connect to and query blob storage
- Integrates with the `/health` HTTP endpoint, which specifies all registered health checks must pass for app to be considered ready to accept traffic

[!INCLUDE [integration-observability-and-telemetry](../includes/integration-observability-and-telemetry.md)]

### Logging

The .NET Aspire Azure Blob Storage integration uses the following log categories:

- `Azure.Core`
- `Azure.Identity`

### Tracing

The .NET Aspire Azure Blob Storage integration will emit the following tracing activities using OpenTelemetry:

- "Azure.Storage.Blobs.BlobContainerClient"

### Metrics

The .NET Aspire Azure Blob Storage integration currently does not support metrics by default due to limitations with the Azure SDK.

## See also

- [Azure Blob Storage docs](/azure/storage/blobs/)
- [.NET Aspire integrations](../fundamentals/integrations-overview.md)
- [.NET Aspire GitHub repo](https://github.com/dotnet/aspire)
