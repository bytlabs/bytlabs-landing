---
layout: post
title: "Getting Started with BytLabs.MicroserviceTemplate: API Project Setup"
description: "The BytLabs.MicroserviceTemplate provides a robust and feature-rich foundation for building scalable, multitenant, and observable microservices. This guide takes you through the configuration steps, focusing on the key components and services utilized by the template to get your microservice up and running."
author:
   - name: Shadman Kudchikar
     email: kudchikarsk@gmail.com
     website: https://www.linkedin.com/in/shadman-kudchikar/
---

The **BytLabs.MicroserviceTemplate** provides a robust and feature-rich foundation for building scalable, multitenant, and observable microservices. This guide takes you through the configuration steps, focusing on the key components and services utilized by the template to get your microservice up and running.


---

## **Getting Started with BytLabs.MicroserviceTemplate: Blog Series**

This post is part of the **"Getting Started with BytLabs.MicroserviceTemplate"** blog series. Explore the complete series to master building microservices using BytLabs.MicroserviceTemplate:

1. **[Getting Started with BytLabs.MicroserviceTemplate: API Project Setup](https://bytlabs.co/blog/getting-started-with-bytlabsmicroservicetemplate-api-project-setup)**  
2. **[Getting Started with BytLabs.MicroserviceTemplate: Domain and Application Setup](https://bytlabs.co/blog/getting-started-with-bytlabsmicroservicetemplate-domain-and-application-setup)**  

---

## Important Links

Check out these handy links to get you started:

- [BytLabs.MicroserviceTemplate GitHub Repo – **Use Template**, Fork, Star, and Contribute!](https://github.com/bytlabs/BytLabs.MicroserviceTemplate)
- [BytLabs.BackendPackages GitHub Repo – Fork, Star, and Contribute!](https://github.com/bytlabs/BytLabs.BackendPackages)
- [NuGet Package Page](https://www.nuget.org/profiles/bytlabs)

---

## Prerequisites

Before you jump in, make sure you’ve got these covered:  

- **.NET 8 SDK**: You’ll need the latest .NET 8 SDK (version 8.0 or higher). Grab it [here](https://dotnet.microsoft.com/en-us/download/dotnet/8.0).  
- **Visual Studio 2022 (v17.10+)**: Install Visual Studio 2022, and don’t forget to include the ASP.NET and web development workload. Check out my free guide for setting up the Community Edition.  
- **Learn Clean Architecture**: Familiarize yourself with Clean Architecture by reading this [detailed article](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures#clean-architecture).  
- **Understand CQRS and MediatR**: Once you’re good with Clean Architecture, dive into this [article](https://www.arunyadav.in/codehacks/blogs/post/28/use-mediatr-in-net-core-with-cqrs-implementation) on using MediatR with CQRS for .NET Core.  

Got everything set? Great, let’s get started!

---

## Folder Structure

The **BytLabs.MicroserviceTemplate.Api** project is organized to facilitate the integration of GraphQL with clean separation of concerns. Here’s the structure:

```
BytLabs.MicroserviceTemplate.Api
|-- Graphql
    |
    |-- Mutations
        |-- OrderMutations.cs
    |
    |-- Queries
        |-- OrderQueries.cs
|
|-- appsettings.json
|-- Program.cs
```

---

## Key Components in the API Project

The main API project leverages several libraries to enable a comprehensive solution:

- **[BytLabs.Api](https://www.nuget.org/packages/BytLabs.Api):** Provides essential APIs for building and managing microservices.
- **[BytLabs.Api.GraphQL](https://www.nuget.org/packages/BytLabs.Api.GraphQL):** Extends the API capabilities by integrating GraphQL support.

Here is the `program.cs` configuration for the API project:

```csharp
ApiServiceBuilder.CreateBuilder(builder)
 .WithHttpContextAccessor(userContextBuilder =>
 {
        userContextBuilder.AddResolver<HttpUserContextResolver>();
 })
 .WithMultiTenantContext(multitenancyBuilder =>
 {
        multitenancyBuilder.AddResolver<FromHeaderTenantIdResolver>();
 })
 .WithLogging()
 .WithMetrics()
 .WithTracing()
 .WithHealthChecks()
 .WithServiceConfiguration(services =>
 {
        services.AddInfrastructure(builder.Configuration);

        services
 .AddWebSockets(op => op.KeepAliveInterval = TimeSpan.FromSeconds(30));

        services.AddGraphQLService()
 .AddCommandTypes()
 .AddDtoTypes()
 .AddMutationType<Mutation>()
 .AddQueryType<Query>()
 .ModifyCostOptions(o => o.EnforceCostLimits = false)
 .ModifyOptions(o => o.RemoveUnreachableTypes = true);
 });
```

Let's break down the configuration steps in detail:

---

### Step 1: Configure `HttpContextAccessor` and Header Propagation

```csharp
.WithHttpContextAccessor(userContextBuilder =>
{
    userContextBuilder.AddResolver<HttpUserContextResolver>();
})
```

This step adds the **HttpContextAccessor** to the dependency injection container, which allows you to access the HTTP context across requests. In addition, header propagation is configured to automatically propagate specific headers, such as `Tenant` and `Authorization`, throughout the service collection.

For more details, you can refer to:

- [HttpConfiguration class](https://github.com/bytlabs/BytLabs.BackendPackages/blob/main/src/BytLabs.Api/BytLabs.Api/HttpConfiguration.cs)
- [ApiServiceBuilder class](https://github.com/bytlabs/BytLabs.BackendPackages/blob/c1b9d4c3e8a86c98301e9221e0caac9a299d1299/src/BytLabs.Api/BytLabs.Api/ApiServiceBuilder.cs#L60)

In addition to providing access to HTTP context data, this configuration enables you to resolve user context through the `IUserContextProvider` interface. This provider in turn depends on `IUserContextResolver` which can be registered like this:

```csharp

public class CustomUserContextResolver : IUserContextResolver
{
 // your implementation
}

userContextBuilder.AddResolver<CustomUserContextResolver>();
```

By default, the program.cs file has the `HttpUserContextResolver` registered as the default resolver, which retrieves the `userId` from the HTTP context. 

```csharp
userContextBuilder.AddResolver<HttpUserContextResolver>();
```
The default implementation of `HttpUserContextResolver` can be found here:

- [HttpUserContextResolver.cs](https://github.com/bytlabs/BytLabs.BackendPackages/blob/main/src/BytLabs.Api/BytLabs.Api/UserContextResolvers/HttpUserContextResolver.cs).

However, 

You can implement custom resolvers that adapt the user context resolution to your needs.

This flexibility allows for a tailored user context resolution mechanism in your microservice architecture.

---

### Step 2: Configure Multitenancy

```csharp
.WithMultiTenantContext(multitenancyBuilder =>
{
    multitenancyBuilder.AddResolver<FromHeaderTenantIdResolver>();
})
```

This step enables you to resolve tenant ID through the `ITenantIdProvider` interface. This provider in turn depends on `ITenantIdResolver` which can be registered like this:

```csharp
   multitenancyBuilder.AddResolver<FromHeaderTenantIdResolver>();
```

The default resolver is `FromHeaderTenantIdResolver`, which resolves the tenant identifier from the HTTP request header. 

You can learn more about the `FromHeaderTenantIdResolver` implemenation at 
- [FromHeaderTenantIdResolver.cs](https://github.com/bytlabs/BytLabs.BackendPackages/blob/main/src/BytLabs.Api/BytLabs.Api/TenantProvider/FromHeaderTenantIdResolver.cs)

But,

You can replace it with your custom resolver by implementing the `ITenantIdResolver` interface.
 
Example custom tenant resolution:

```csharp
public class CustomTenantIdResolver : ITenantIdResolver 
{
 // your implementation
}

multitenancyBuilder.AddResolver<CustomTenantIdResolver>();
```

This step utilizes the following package:
- [BytLabs.Multitenancy](https://www.nuget.org/packages/BytLabs.Multitenancy/)


---

### Step 3: Configure Logging

```csharp
.WithLogging(loggerConfiguration =>
{
 // Custom configuration here
})
```

Logging is handled by **Serilog**. This step sets up structured logging, enriching logs with additional information such as the environment name, application name, and trace/span details. Logs are written to the console, but you can also configure additional sinks, like external logging services.

If OpenTelemetry is enabled, logs will be exported to a collector for advanced monitoring and analysis.

You can learn more about the `LoggingRegistration` implemenation at 
- [LoggingRegistration.cs](https://github.com/bytlabs/BytLabs.BackendPackages/blob/main/src/BytLabs.Observability/BytLabs.Observability/LoggingRegistration.cs)

This step utilizes the following package:
- [BytLabs.Observability](https://www.nuget.org/packages/BytLabs.Observability/)

---

### Step 4: Configure Metrics

```csharp
.WithMetrics(configureMetrics => {
 // Custom configuration here
})
```

This step configures **OpenTelemetry** for capturing metrics. Metrics are used to monitor system health, performance, and resource consumption. By default, the configuration enables exporting metrics to an OpenTelemetry collector.

You can learn more about the `MetricsRegistration` implemenation at 
- [MetricsRegistration.cs](https://github.com/bytlabs/BytLabs.BackendPackages/blob/main/src/BytLabs.Observability/BytLabs.Observability/MetricsRegistration.cs)

This step utilizes the following package:
- [BytLabs.Observability](https://www.nuget.org/packages/BytLabs.Observability/)

---

### Step 5: Configure Tracing

```csharp
.WithTracing(configureTracing => {
 // Custom configuration here
})
```

Distributed tracing is enabled using **OpenTelemetry**, allowing you to track requests across different services. By default, this configuration tracks HTTP requests and records exceptions.

It configures an OTLP exporter with gRPC protocol for exporting traces, specifying the endpoint, export type (batch), and timeout configuration.

You can extend tracing to include other service interactions, like database queries or external HTTP calls, by modifying the tracing configuration.

You can learn more about the `TraceRegistration` implemenation at 
- [TraceRegistration.cs](https://github.com/bytlabs/BytLabs.BackendPackages/blob/main/src/BytLabs.Observability/BytLabs.Observability/TraceRegistration.cs)


This step utilizes the following package:
- [BytLabs.Observability](https://www.nuget.org/packages/BytLabs.Observability/)

---

### Step 6: Configure Health Checks

```csharp
.WithHealthChecks()
```

Health checks provide endpoints for verifying the health and readiness of the microservice. The default health check setup includes:

- `/healthz`: General health check.
- `/readyz`: Readiness check to indicate when the service is fully ready to handle the traffic.
- `/startupz`: Check for the service’s startup readiness.

Health check implementations can be extended for more specific checks, such as database or service dependencies.

You can learn more about the `HealthChecks` implemenation at 
- [ServiceExtensions.cs](https://github.com/bytlabs/BytLabs.BackendPackages/blob/main/src/BytLabs.Observability/BytLabs.Observability/HealthChecks/ServiceExtensions.cs)
- [DefaultApplicationHealthCheck.cs](https://github.com/bytlabs/BytLabs.BackendPackages/blob/main/src/BytLabs.Observability/BytLabs.Observability/HealthChecks/DefaultApplicationHealthCheck.cs)
- [DefaultApplicationReadyCheck.cs](https://github.com/bytlabs/BytLabs.BackendPackages/blob/main/src/BytLabs.Observability/BytLabs.Observability/HealthChecks/DefaultApplicationReadyCheck.cs)
- [DefaultApplicationStartupCheck.cs](https://github.com/bytlabs/BytLabs.BackendPackages/blob/main/src/BytLabs.Observability/BytLabs.Observability/HealthChecks/DefaultApplicationStartupCheck.cs)

This step utilizes the following package:
- [BytLabs.Observability](https://www.nuget.org/packages/BytLabs.Observability/)

---

### Step 7: Configure Application Services

```csharp
services.AddInfrastructure(builder.Configuration);

services
 .AddWebSockets(op => op.KeepAliveInterval = TimeSpan.FromSeconds(30));

services.AddGraphQLService()
 .AddCommandTypes()
 .AddDtoTypes()
 .AddMutationType<Mutation>()
 .AddQueryType<Query>()
 .ModifyCostOptions(o => o.EnforceCostLimits = false)
 .ModifyOptions(o => o.RemoveUnreachableTypes = true);
```

This step configures essential services:
- **`AddInfrastructure`**: Registers infrastructure services, including database connections and business logic components.
- **`AddWebSockets`**: Enables WebSocket communication with a customizable keep-alive interval.
- **`AddGraphQLService`**: Configures GraphQL support, adding types, mutations, and queries for your API.

#### Default Setup with `AddGraphQLService()`
The **`services.AddGraphQLService()`** method is part of the **BytLabs.Api.Graphql** package, which abstracts the setup of the **HotChocolate** GraphQL server in your API project. This method provides an out-of-the-box configuration for setting up the GraphQL server using **HotChocolate** and simplifies many of the steps required to configure a GraphQL API.

#### Customizing the GraphQL Service

Once the initial setup is complete using **`services.AddGraphQLService()`**, you can further customize the GraphQL server to fit your application's needs by chaining additional methods. Here are some examples of further configuration:

1. **`AddCommandTypes()`**:
   - Registers GraphQL command types, typically used for mutations. Command types define the operations that modify data in your system, like `CreateOrderCommand` or `UpdateCustomerCommand`. The BytLabs.MicroserviceTemplate.Infrastructure project implements `AddCommandTypes` at [`RequestExecutorBuilderExtensions.cs`](https://github.com/bytlabs/BytLabs.MicroserviceTemplate/blob/main/src/BytLabs.MicroserviceTemplate.Infrastructure/RequestExecutorBuilderExtensions.cs) where it registers commands, like `CreateOrderCommand` or `UpdateCustomerCommand` as Graphql Input Types in Hotchocolate server
   this is done to follow the standard naming convention where Command gets replaced with Input.

2. **`AddDtoTypes()`**:
   - Registers DTO (Data Transfer Object) types for use in the GraphQL schema. DTOs define the shape of the data that will be exposed by the GraphQL API. This is useful when you want to decouple the internal domain models from the external API contract (GraphQL). The BytLabs.MicroserviceTemplate.Infrastructure project implements `AddDtoTypes` at [`RequestExecutorBuilderExtensions.cs`](https://github.com/bytlabs/BytLabs.MicroserviceTemplate/blob/main/src/BytLabs.MicroserviceTemplate.Infrastructure/RequestExecutorBuilderExtensions.cs) where it registers dtos, like `OrderDto` and `OrderItemDto` as Graphql Object Types in Hotchocolate server

3. **`AddMutationType<Mutation>()`**:
   - Specifies the mutation type class to be used for the GraphQL mutations. The `Mutation` class contains all the GraphQL mutation definitions that can be performed in your application. You can define operations like creating or updating entities in the system here.

4. **`AddQueryType<Query>()`**:
   - Specifies the query type class to be used for the GraphQL queries. The `Query` class contains all the GraphQL query definitions that can be executed by clients to retrieve data. For example, `OrderQuery` or `CustomerQuery` would be added here to allow clients to fetch order or customer data.

5. **`ModifyCostOptions()`**:
   - **ModifyCostOptions** allows you to fine-tune the cost analysis of your GraphQL queries. The cost analysis helps prevent expensive or inefficient queries from being executed. In this case, the example disables cost enforcement with **`o.EnforceCostLimits = false`**, meaning that the server will not limit the complexity of queries, and all queries, regardless of their cost, will be executed.

6. **`ModifyOptions()`**:
   - **ModifyOptions** is used to modify additional **HotChocolate** options. In the example, **`o.RemoveUnreachableTypes = true`** is set, which ensures that any types in the schema that are not reachable via queries or mutations are removed from the schema. This helps to clean up the schema and prevent unnecessary exposure of types that aren't used.

---

## Conclusion

The **BytLabs.MicroserviceTemplate** provides a feature-rich starting point for building scalable, secure, and observable microservices. By following these configuration steps, you ensure that your service is ready to handle multitenancy, logging, monitoring, tracing, and health checks, along with application-specific features like WebSockets and GraphQL. You can easily extend and customize these components to suit the specific needs of your application.

This template simplifies the development of microservices, empowering you to focus on business logic while ensuring robust operational capabilities.

---

## Need Help?  
If you have any questions, need help, or face any confusion while setting up, feel free to reach out to me on [LinkedIn](https://www.linkedin.com/in/shadman-kudchikar/). I'm happy to assist!