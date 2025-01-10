---
layout: post
title: "Getting Started with BytLabs.MicroserviceTemplate: Domain and Application Setup"
description: "This blog post outlines how to set up the Domain, Application, and Infrastructure layers in BytLabs.MicroserviceTemplate. It covers DDD for domain models, CQRS with MediatR for handling commands and queries, MongoDB for persistence, and AutoMapper for object mapping, along with pipeline behaviors for logging and validation."
author:
   - name: Shadman Kudchikar
     email: kudchikarsk@gmail.com
     website: https://www.linkedin.com/in/shadman-kudchikar/
---

This blog post outlines how to use [BytLabs.MicroserviceTemplate](https://github.com/bytlabs/BytLabs.MicroserviceTemplate) to define your use cases in the Domain, Application, and Infrastructure layers. It covers how to set up aggregates and entities, define commands and queries to handle use cases, and finally, set up the infrastructure layer.

---

## Contents

- [Blog Series](#blog-series)
- [Important Links](#important-links)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Setting Up Domain Models](#setting-up-domain-models)
- [Setting Up Application Layer](#setting-up-application-layer)
- [Setting Up Infrastructure Layer](#setting-up-infrastructure-layer)
- [Conclusion](#conclusion)
- [Need Help?](#need-help)

---

## **Blog Series**

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

## Project Structure

We have respective projects for Domain, Application, and Infrastructure layers, with the dependency flow moving outward, following the principles of clean architecture:

- BytLabs.MicroserviceTemplate.Domain
- BytLabs.MicroserviceTemplate.Application
- BytLabs.MicroserviceTemplate.Infrastructure

Let's start with the core domain setup.

---

## Setting Up Domain Models

To set up domain models, use the **BytLabs.MicroserviceTemplate.Domain** project. The folder structure is organized to support Domain-Driven Design (DDD) principles, making it easier to manage aggregates, entities, and value objects.

### Folder Structure

Within the **Domain** project, the structure is organized as follows:

```
BytLabs.MicroserviceTemplate.Domain
|
|-- Aggregates
   |
   |-- OrderAggregate
   |-- Order.cs (Root Aggregate)
   |-- OrderItem.cs (Entity)
   |
   |-- Events
      |-- OrderCreatedEvent.cs
      |-- OrderShippedEvent.cs
```

Let's see how to set up the Domain layer, keeping the order example in mind. First, let's look at how to define aggregates and entities.

### Aggregate Structure

The `Order` class is the root aggregate for the `OrderAggregate` folder, representing the main entry point for managing the domain logic related to orders. It is designed to ensure consistency and enforce business rules across related entities, such as `OrderItem`.

#### `Order` Class Example

The `Order` class inherits from the base class `AggregateRootBase<Guid>`, which is required to work seamlessly with BytLabs packages. Below is an example:

```csharp
public class Order : AggregateRootBase<Guid>
{
    public DateTime OrderDate { get; private set; }
    public OrderStatus Status { get; private set; }
    public IReadOnlyCollection<OrderItem> Items { get; private set; }

    public Order(Guid id, DateTime orderDate, IEnumerable<OrderItem> items) : base(id)
 {
        if (!items.Any())
            throw new DomainException("An order must have at least one item.");

        Id = id;
        OrderDate = orderDate;
        Status = OrderStatus.Pending;
        Items = items.ToList();

        AddDomainEvent(new OrderCreatedEvent(Id));
 }

    public void MarkAsShipped()
 {
        if (Status != OrderStatus.Pending)
            throw new DomainException("Only pending orders can be marked as shipped.");

        Status = OrderStatus.Shipped;

        AddDomainEvent(new OrderShippedEvent(Id));
 }
}
```

### Key Concepts in the `Order` Class

1. **Inheritance from `AggregateRootBase<Guid>`:**

   - This ensures compatibility with BytLabs packages.
   - The `AggregateRootBase<T>` provides a base for defining aggregates, handling unique identifiers, and managing domain events.

2. **Properties:**

   - `OrderDate`: Tracks when the order was placed.
   - `Status`: Tracks the current status of the order (e.g., `Pending`, `Shipped`).
   - `Items`: A collection of `OrderItem` entities included in the order.

3. **Constructor:**

   - Validates the presence of at least one order item.
   - Initializes properties like `OrderDate` and `Status`.
   - Raises the `OrderCreatedEvent` domain event.

4. **Methods:**

   - `MarkAsShipped()`: Updates the order status to `Shipped` if it is currently `Pending`.
   - Raises the `OrderShippedEvent` domain event.

5. **Domain Events:**

   - The `AddDomainEvent` method records events like `OrderCreatedEvent` and `OrderShippedEvent`, enabling event-driven communication within the application.

### Domain Events

The `OrderAggregate` folder includes an `Events` folder containing domain events related to the `Order` aggregate. These events implement the `IDomainEvent` interface provided by BytLabs, enabling seamless integration with event-driven systems.

#### `OrderCreatedEvent` Example

```csharp
using BytLabs.Domain.DomainEvents;

namespace BytLabs.MicroserviceTemplate.Domain.Aggregates.OrderAggregate.Events
{
    public record class OrderCreatedEvent(Guid OrderId) : IDomainEvent;
}
```

#### `OrderShippedEvent` Example

```csharp
using BytLabs.Domain.DomainEvents;

namespace BytLabs.MicroserviceTemplate.Domain.Aggregates.OrderAggregate.Events
{
    public record class OrderShippedEvent(Guid OrderId) : IDomainEvent;
}
```

### Key Concepts in Domain Events

1. **Implementation of `IDomainEvent`:**
   
   - Both `OrderCreatedEvent` and `OrderShippedEvent` implement the `IDomainEvent` interface, which is required to work with BytLabs packages.

2. **Event-Driven Design:**
   
   - These events allow the domain layer to publish changes in the aggregate's state without directly coupling to external services.

3. **Usage in the Aggregate:**
   
   - The `Order` class adds these events using the `AddDomainEvent` method, ensuring they are captured during lifecycle changes.

### Entity Structure

#### `OrderItem` Class Example

The `OrderItem` class represents an entity that is part of the `Order` aggregate. Unlike the `Order` class, which is an aggregate root, `OrderItem` is a related entity within the aggregate. The class inherits from `Entity<Guid>`, ensuring it can be tracked and managed within the `Order` aggregate.

Here is the `OrderItem` class example:

```csharp
public class OrderItem : Entity<Guid>
{
    public Guid ProductId { get; private set; }
    public int Quantity { get; private set; }
    public decimal Price { get; private set; }

    public OrderItem(Guid productId, int quantity, decimal price) : base(Guid.NewGuid())
 {
        if (quantity <= 0)
            throw new DomainException("Quantity must be greater than zero.");

        if (price <= 0)
            throw new DomainException("Price must be greater than zero.");

        ProductId = productId;
        Quantity = quantity;
        Price = price;
 }
}
```

### Key Concepts in the `OrderItem` Class

1. **Inheritance from `Entity<Guid>`:**

   - The `OrderItem` class inherits from `Entity<Guid>`, which provides the ability to track entities by a unique identifier (`Guid` in this case).
   - This inheritance allows `OrderItem` to be part of the `Order` aggregate, and it ensures that the entity is tracked throughout the lifecycle of the aggregate.

2. **Properties:**

   - `ProductId`: Represents the identifier of the product in the order.
   - `Quantity`: The number of units of the product in the order.
   - `Price`: The price of a single unit of the product.

3. **Constructor:**

   - The constructor initializes the properties: `ProductId`, `Quantity`, and `Price`.
   - It includes validation to ensure the `Quantity` and `Price` are greater than zero. If either is invalid, a `DomainException` is thrown.

4. **Validation:**

   - The constructor checks the validity of the `Quantity` and `Price` before setting them, ensuring that only valid values are accepted, which maintains consistency in the domain model.

### How the `OrderItem` Works with the `Order` Aggregate

The `OrderItem` class is designed to be part of the `Order` aggregate, and it is used within the `Order` class to represent the items included in the order. 

The `Order` class contains a collection of `OrderItem` entities, and it can manage their consistency along with the aggregate's rules. For example, the `Order` class ensures that there is at least one `OrderItem` when an order is created and that each `OrderItem` is valid in terms of quantity and price.

---

## Setting Up Application Layer

In this section, we'll explore the **BytLabs.MicroserviceTemplate.Application** project, which serves as the application layer in the BytLabs microservice architecture. This layer is responsible for implementing business logic, handling commands and queries, and coordinating between the domain layer and external interfaces.

### Folder Structure

The **Application** project is organized to promote a clean separation of concerns and to facilitate the implementation of the CQRS (Command Query Responsibility Segregation) pattern. The typical folder structure is as follows:

```
BytLabs.MicroserviceTemplate.Application
|
|-- Commands
|   |-- CreateOrderCommand.cs
|   |-- CreateOrderCommandHandler.cs
|
|-- Queries
|
|-- DTOs
|   |-- OrderDto.cs
|   |-- OrderItemDto.cs

```

### Key Components

1. **Commands:**
   - Represent actions that change the state of the system.
   - Examples: `CreateOrderCommand`.

2. **Queries:**
   - Represent requests for data without modifying the system's state.

3. **Handlers:**
   - Handle business logic for commands and queries.
   - Examples: `CreateOrderCommandHandler`

4. **DTOs (Data Transfer Objects):**
   - Define the data structure used for communication between layers.
   - Examples: `OrderDto`, `OrderItemDto`.

#### `CreateOrderCommand` and `CreateOrderCommandHandler` Example

In the CQRS (Command Query Responsibility Segregation) pattern, commands represent requests to change the state of the system. In this case, the `CreateOrderCommand` is used to create a new order, and the `CreateOrderCommandHandler` handles the command and performs the required actions, such as inserting the order into the repository.

Here's how the `CreateOrderCommand` and `CreateOrderCommandHandler` are implemented:

```csharp
public record CreateOrderCommand(Guid OrderId, DateTime OrderDate, IEnumerable<OrderItem> Items) : ICommand<CreateOrderResult>;

public record CreateOrderResult(Guid OrderId);

public class CreateOrderCommandHandler : ICommandHandler<CreateOrderCommand, CreateOrderResult>
{
    private readonly IRepository<Order, Guid> orderRepository;

    public CreateOrderCommandHandler(IRepository<Order, Guid> orderRepository)
 {
        this.orderRepository = orderRepository;
 }

    public async Task<CreateOrderResult> Handle(CreateOrderCommand request, CancellationToken cancellationToken)
 {
        var order = new Order(request.OrderId, request.OrderDate, request.Items);
        await orderRepository.InsertAsync(order, cancellationToken);
        return new CreateOrderResult(request.OrderId);
 }
}
```

### Explanation

1. **`CreateOrderCommand`:**
   - This is a simple record that encapsulates the data required to create an order. It contains the `OrderId`, `OrderDate`, and a collection of `OrderItem` entities.
   - The `ICommand<T>` interface signifies that this command is responsible for changing the state of the system and returning a result (`CreateOrderResult` in this case).

2. **`CreateOrderResult`:**
   - This is a record that represents the result of handling the `CreateOrderCommand`. It returns the `OrderId` of the newly created order.

3. **`CreateOrderCommandHandler`:**
   - This class handles the command. It implements the `ICommandHandler<TCommand, TResult>` interface, where `TCommand` is the type of the command (`CreateOrderCommand`), and `TResult` is the type of the result (`CreateOrderResult`).
   - It contains a dependency on `IRepository<Order, Guid>`, which is used to insert the new `Order` into the database.
   - In the `Handle` method, it creates a new `Order` from the command data, inserts it into the repository, and returns a `CreateOrderResult`.

### How MediatR Handles Commands and Handlers

MediatR is used to handle commands and queries in a decoupled manner, eliminating the need for explicit dependencies on technologies like ASP.NET Web API or HotChocolate GraphQL. Here's how MediatR works:

1. **Triggering the Command:**
   - The command is triggered via different interfaces, such as REST APIs or GraphQL endpoints.
   - For example, in a REST API controller, you would call `mediator.Send(new CreateOrderCommand(...))`, which internally uses MediatR to route the command to the appropriate handler.

2. **Pipeline Behaviors:**
   - MediatR's pipeline behavior allows you to manipulate the request-handling process. You can use pipeline behaviors to add logic such as logging, validation, or caching before or after the command handler is executed.
   - In the case of the `CreateOrderCommandHandler`, if you want to add custom behavior (e.g., logging or validation), you can create a custom `IPipelineBehavior<TRequest, TResponse>` and register it with MediatR.
   - Pipeline behaviors run for both commands and queries, enabling cross-cutting concerns like authorization or logging to be handled separately from the command handlers themselves.

### Removing Technology Dependencies

Using MediatR allows you to decouple your business logic from the underlying web framework, whether it's ASP.NET Web API, HotChocolate GraphQL, or any other interface layer. The logic for handling commands and queries is kept separate from the specific technology used for the HTTP/GraphQL interface.

For instance, in the case of GraphQL using HotChocolate, you can define a mutation that triggers the `CreateOrderCommand` without depending on the ASP.NET Core Web API infrastructure. Instead, the mutation handler would send the command through MediatR to trigger the business logic encapsulated in the command handler.

### Example with HotChocolate (GraphQL)

```csharp
public class Mutation
{
    private readonly IMediator _mediator;

    public Mutation(IMediator mediator)
 {
        _mediator = mediator;
 }

    public async Task<CreateOrderResult> CreateOrder(CreateOrderCommand command)
 {
        return await _mediator.Send(command);
 }
}
```

In the GraphQL setup, the `Mutation` class defines a `CreateOrder` method, which receives the `CreateOrderCommand` and sends it via MediatR. The `CreateOrderCommandHandler` is executed, and the result is returned back as the response.

### IRepository Abstraction

```csharp
public class CreateOrderCommandHandler : ICommandHandler<CreateOrderCommand, CreateOrderResult>
{
    private readonly IRepository<Order, Guid> orderRepository;

    public CreateOrderCommandHandler(IRepository<Order, Guid> orderRepository)
 {
        this.orderRepository = orderRepository;
 }

 ...
}
```

The `IRepository<TAggregate, TId>` interface is a generic interface defined in the **BytLabs.Application** used within **BytLabs.MicroserviceTemplate.Application** project to abstract data access operations in the application layer. It provides basic methods like `InsertAsync`, `GetByIdAsync`, `UpdateAsync`, and `DeleteAsync` for interacting with the database.

How this is implemented is discussed in the next section, let's dive into it.

---

## Setting Up Infrastructure Layer

In this section, we'll delve into setting up the **Infrastructure** layer for the **BytLabs.MicroserviceTemplate** project, which is responsible for configuring essential infrastructure services such as the database, MediatR for CQRS (Command-Query Responsibility Segregation), and other cross-cutting concerns like logging, validation, and dependency injection. 

### Folder Structure

The **Infrastructure** project contains services that interact with external systems, like databases and messaging frameworks. The typical folder structure looks like this:

```
BytLabs.MicroserviceTemplate.Infrastructure
|
|-- ServiceExtensions.cs
```



### `ServiceExtensions` Setup

The `ServiceExtensions` class in the `BytLabs.MicroserviceTemplate.Infrastructure` project handles the registration of all critical services and middleware into the Dependency Injection (DI) container. Here’s a breakdown of what it sets up:

```csharp
public static class ServiceExtensions
{
    public static IServiceCollection AddInfrastructure(this IServiceCollection services, ConfigurationManager configuration)
 {
        if (services == null) throw new ArgumentNullException(nameof(services));
        if (configuration == null) throw new ArgumentNullException(nameof(configuration));

 // Set up CQS with MediatR
        services.AddCQS(new System.Reflection.Assembly[] { typeof(CreateOrderCommand).Assembly });

 // Add AutoMapper
        services.AddAutoMapper(typeof(OrderMappingProfile));

 // Set up MongoDB
        var mongoDatabaseConfiguration = configuration.GetConfiguration<MongoDatabaseConfiguration>();
        services.AddMongoDatabase(mongoDatabaseConfiguration)
 .RegisterMongoDBClassMaps()
 .AddMongoRepository<Order, Guid>();

        return services;
 }

    private static IServiceCollection RegisterMongoDBClassMaps(this IServiceCollection services)
 {
        BsonClassMap.TryRegisterClassMap<OrderItem>(cm =>
 {
            cm.AutoMap();
            cm.MapMember(c => c.ProductId)
 .SetSerializer(new GuidSerializer(BsonType.String));
 });

        return services;
 }
}
```
#### **AddCQS Setup for MediatR, Fluent Validation, and Pipeline Behaviors**

In the **BytLabs.MicroserviceTemplate.Infrastructure** project, the `AddCQS` method is used to configure **MediatR** for Command and Query handling, and it also incorporates essential middleware functionalities like **Fluent Validation** and **Request Logging** through pipeline behaviors.

Here’s a breakdown of how **AddCQS** helps set up these features:

1. **CQRS Setup with MediatR**:
   - The `AddCQS` method is responsible for setting up the **Command** and **Query** handling using **MediatR**. This facilitates the Command-Query Responsibility Segregation (CQRS) pattern, where commands (write operations) and queries (read operations) are processed separately, ensuring a clear separation of concerns.
   - MediatR ensures that for each Command (such as `CreateOrderCommand`) or Query, there are corresponding handlers (such as `CreateOrderCommandHandler`) that process them, which helps to maintain clean architecture.

2. **Fluent Validation Integration**:
   - **Fluent Validation** is automatically integrated through the pipeline behaviors. This ensures that commands and queries are validated before being handled by their respective handlers. For example, if a command requires a certain parameter to be non-null or within a valid range, Fluent Validation checks this before the request proceeds further.
   - This is part of the pipeline setup that validates requests as they flow through MediatR.

3. **Logging and Request Monitoring**:
   - The **Request Logging** behavior ensures that every incoming command or query request is logged, capturing essential information about the request, such as the type of command or query, its data, and any other relevant details. This helps monitor the flow of requests through the system, making it easier to trace issues and debug the application.
   - The logging behavior can be extended to log additional information such as execution times, error handling, etc.


#### **AddMongoDatabase Setup for MongoDB Integration**

Now let's discuss how `IRepository<Order, Guid> orderRepository` is resolved eventually in CreateOrderCommandHandler.

```csharp
public class CreateOrderCommandHandler : ICommandHandler<CreateOrderCommand, CreateOrderResult>
{
    private readonly IRepository<Order, Guid> orderRepository;

    public CreateOrderCommandHandler(IRepository<Order, Guid> orderRepository)
 {
        this.orderRepository = orderRepository;
 }

 ...
}
```

The `IRepository<TAggregate, TId>` interface is a generic interface defined in the **BytLabs.Application** used within **BytLabs.MicroserviceTemplate.Application** project to abstract data access operations in the application layer. It provides basic methods like `InsertAsync`, `GetByIdAsync`, `UpdateAsync`, and `DeleteAsync` for interacting with the database.

In the **BytLabs.MicroserviceTemplate.Infrastructure** layer, the `MongoRepository<Order>` class implements this interface, using the [`BytLabs.DataAccess.MongoDB`](https://www.nuget.org/packages/BytLabs.DataAccess.MongoDB/) package to interact with MongoDB. This is done in the infrastructure
layer with the below code:

```csharp
 services.
 ...
 ...
 .AddMongoRepository<Order, Guid>();
```

Through Dependency Injection (DI), the `MongoRepository<Order, Guid>` is injected into command handlers like `CreateOrderCommandHandler`, allowing the infrastructure layer's data access logic to be seamlessly integrated into the application without coupling the business logic to a specific technology like MongoDB. This makes the code more modular and testable.

---

## Conclusion

BytLabs.MicroserviceTemplate provides a robust foundation for developing microservices using DDD and CQRS. It ensures a clean architecture with a clear separation of concerns across the Domain, Application, and Infrastructure layers. By leveraging features like domain events, repository patterns, and MediatR, you can create scalable, maintainable, and testable applications.

This guide gives you a head start on setting up your microservice. Explore the template further to adapt it to your specific requirements and streamline your development process.

---

## Need Help?  

If you have any questions, need help, or face any confusion while setting up, feel free to reach out to me on [LinkedIn](https://www.linkedin.com/in/shadman-kudchikar/). I'm happy to assist!