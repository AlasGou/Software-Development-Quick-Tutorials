# Domain Driven Design

Domain Driven Design (DDD) is a technique for building software with complex
business domains. The core idea is to put the focus on building a shared software
model that both software developers, and domain experts can understand. This
software model, called the *domain* should model the business problem, but
include no technical issues like database persistence or talking to 
web services. 

When combined with Onion Architecture it can be a clean way of separating the
business issues from the technical issues. Onion Architecture is similar to
Ports & Adapters and Hexagonal Architecture to the point where they are
otherwise the same. You can learn more about Onion Architecture at the following
[article](../Onion-Architecture).

## Ubiquitous Language

One of the higher level principles of DDD is Ubiquitous Language, it is
something that can improve your code even when you are not using domain driven design.
Ubiquitous language is one the most underused parts of DDD with people preferring to
dive straight into DDD modeling techniques like aggregates and entities.

Ubiquitous Language allows domain experts and software developers to understand
each other, and is core to making sure the software model matches the reality
of the domain. When discussing the domain, software developers and domain experts
should collaboratively develop a language for use when discussing
the domain. This language should identify and explain standard business terms and 
phrases related to the domain in a way that both parties can understand. It is 
useful to document this language in a internal wiki that anyone can reference.

Any specifications, user stories, acceptance criteria should use the
standard terms and phrases identified in the ubiquitous language.
When writing the software for the domain model, it should be
implemented using ubiquitous language to the point where the domain expert 
should be able to understand the code(within reason).

Imagine we are working on some software for administrating flu shots. On our
first attempt we may attempt to use the following code to record a flu shot:

```csharp
var shot = new Shot
{
    ShotType = ShotTypes.TypeFlu
    Dosage = 15
    Nurse = nurse
    Patient = patient
}
ShotRepository.Add(shot)
```

Lets re-imagine this code using ubiquitous language

```csharp
Vaccine vaccine = vaccines.StandardAdultFluDose();
nurse.AdministerFluVaccine(patient, vaccine);
```

Notice this is almost readable without being programming expert, and much
easier for the software developer and domain expert to see if this is matching the
reality of the business domain.

## Domain Driven Design Modeling Techniques

### Value Object

A value object is an object that is defined by the value of its attributes
rather than having a unique identity. A money object would be a value object.

```csharp
public class Money
{
    public Decimal Value { get; set;}
    public CurrencyType Currency  {get; set;}
}
```

### Entity

An entity is an object that is unique and has an identity. A purchase order is
an entity since it has a unique id.

```csharp
public class PurchaseOrder
{
    public PurchaseOrderId PurchaseOrderId { get; set; }

    //Various properties and methods
}
```

### Aggregate

An aggregate is a collection of objects that should act as a single unit. For example a purchase order aggregate
could contain a purchase order entity, an address value object, a person entity, and line item objects.

Enforcing the *invariants*(a term DDD proponents like to use) otherwise known as business rules
is the job of the aggregates. This is because aggregates are groups of objects that 
should remain consistent with each other according to a set of business rules. 

For the purchase order aggregate we may say that a purchase order can not exist without at least 1 line item,
can not have more than 20 line items, only certain people can create purchase orders,
and that a purchase order over a certain amount can not be automatically approved. 
To enforce these rules, the purchase order entity, person entity, and the line item value objects(to calculate
purchase order total) must work together.

Group together domain objects as aggregates, if they need to work together to enforce
the business rules of the application. For example if the value of a line item changes,
then the value of a purchase order may change which means it may no longer be able
to be automatically approved. This means that these two objects are good candidates 
to grouped together as an aggregate.
 
#### Aggregate Root

The aggregate root is at the top of the hierarchy of an aggregate, it directly
references or indirectly references all the objects inside of an aggregate.
To perform an action on a aggregate, you must you use the aggregate root.
This makes it the primary interface to an aggregate, and enforces
all the invariants.

Any object outside of the aggregate is not allowed change any object inside of the aggregate except for calling for
calling behavior on the aggregate root. If an object inside an aggregate was directly changed,
it could skip some of the business rules applied by the aggregate root.

To create a purchase order aggregate root with a method for approval from different managers each with different
monetary approval limits, we could end up with code that looks like this:

```csharp
public class PurchaseOrder
{
    public PurchaseOrderId PurchaseOrderId { get; set; }

    private List<LineItem> LineItems { get; set; }

    private bool Approved {get; set;}

    //Other properties and methods

    public void Approve(Manager manager)
    {
        if(Approved)
            return;

        if(TotalLineItemAmount() <= manager.AmountAllowedToApprove())
        {
            Approved = true;
            DomainEvents.Raise(new PurchaseOrderApproved(this, manager))
        }
        else
        {
            DomainEvents.Raise(new PurchaseOrderNotApproved(this, manager))
        }
    }
}

```

### Repository

A repository is a method of retrieving aggregate roots for use
by the application services. Do not retrieve any other object using 
a repository when using DDD as this would bypass the 
invarients enforced by the aggregate root. When modelling the domain the repository interfaces are 
part of the domain, but the implementations are not.
This is because the interfaces define business issues
like "retrive all unapproved purchase orders", but 
implementations are more concerned with the details of 
retriving or persisting to a specific persistance technology
like MSSQL. The implementations of repositories are
placed in a separate layer, such as the Infrastructure layer and injected in at runtime.

For example:

Create the repository interface in the domain:

```csharp
namespace App.Domain.PurchaseOrders
{
    public class IPurchaseOrderRepository
    {
        PurchaseOrder PurchaseOrderWithId(PurchaseOrderId id)
    }
}
```
Create the repository implementation in the Infrastructure layer: 

```csharp
namespace App.Infrastructure.Persistence.MSSQL
{
    public class PurchaseOrderRepository : IPurchaseOrderRepository
    {
        public PurchaseOrder PurchaseOrderWithId(PurchaseOrderId id)
        {
            var purchaseOrder == //map from MSSQL database
            return purchaseOrder;
        }
    }
}
```
In the future the requirements may change, and DocumentDb is now
required instead of MSSQL. Change the persistance by creating new
persistence implementations using DocumentDb in the Infrastructure
layer.

```csharp
namespace App.Infrastructure.Persistence.DocumentDb
{
    public class PurchaseOrderRepository : IPurchaseOrderRepository
    {
        public PurchaseOrder PurchaseOrderWithId(PurchaseOrderId id)
        {
            var purchaseOrder == //map from document db
            return purchaseOrder;
        }
    }
}
```
Injecting the new documentdb implementations should now change the application
to persist on DocumentDb without changing domain or any other code except
for the Infrastructure layer.


#### Ubiquitous Language and Repositories

Since repository interfaces are part of the domain they should use ubiquitous language like any other
object in the domain. The example code in the `Ubiquitous Language` section showed the use
 ubiquitous language on a repository:

```csharp
Vaccine vaccine = vaccines.StandardAdultFluDose();
nurse.AdministerFluVaccine(patient, vaccine);
```

One thing to highlight is that the `vaccines` object that the method
`standardAdultFluDose` is called on is a repository but it reads like English. This is
important to highlight since a common pattern to use is generic repositories to expose CRUD 
like schematics. Usually with an interface such as:

```csharp
interface IRepository<T>
{
    Create(T)
    Delete(T)
    Add(T)
    Update(T)
    T Get(int id)
}
```

These generic repositories are tempting since they can reduce the amount of
repository code required but it does make it difficult to use ubiquitous language to make the domain self explanatory.
One way around this to have a internal data access layer that repository
implementations can use to implement methods using ubiquitous language. It won't be as concise as generic repositories but
will reduce the amount of internal implementation.

```csharp
public VaccineRepository : IVaccineRepository
{
    private DataAccessLayer<Vaccine> DataAccess {get;}

    public Vaccine StandardAdultFluDose()
    {
        return DataAccess.FindBy(//some expression to find standard adult flu dose)
    }
}

```

Since repository implementations are not classed as being inside the
domain layer, it does not matter that we are not using ubiquitous language
internally. External code operating on the domain layer won't see the internals of
repositories, but they will see their interfaces!

### Domain Events

A domain event is an event that tells the world that something has happened.
For example we publish a PurchaseOrderApproved event which other objects may
listen for, for example an object listening for PurchaseOrderApproved event may
notify a manager that a purchase order has been approved via email. It is good way of
linking up parts of the domain, and external systems without tightly coupling
objects.

```csharp
public class PurchaseOrder
{
    public PurchaseOrderId PurchaseOrderId { get; set; }

    private List<LineItem> LineItems { get; set; }

    private Money Total {get; set;}

    private bool Approved {get; set;}

    public void Approve(Manager manager)
    {
        if(Approved)
            return;

        if(total <= manager.AmountAllowedToApprove())
        {
            Approved = true;
            DomainEvents.Raise(new PurchaseOrderApproved(this, manager))
        }
        else
        {
            DomainEvents.Raise(new PurchaseOrderNotApproved(this, manager))
        }
    }
}
```

### Bounded Context

A large project may have multiple domain models, and to stop the domain models
becoming blurred we should explicitly define the business areas where the model
applies. This is a bounded context. For example when talking about purchase
order, we could assume that this domain model is part of the purchasing domain which
may include other entities like suppliers.

When using DDD in combination with microservices, the bounded context is a good
way to specify the responsibilities of a service.

### Domain Services

When trying to model using DDD you may feel the need to model something inside the domain
that does not feel like it naturally fits inside of a domain entity. It could be that the business
logic spans multiple domain entities, and putting inside a single entity dilutes that entities focus, 
or it could be the domain needs to retrive something from an external source to process some 
business logic.

In general for something to be a domain service it needs to be:

* Stateless
* Its parameters and return types should be domain objects
* Model business logic that don't naturally fit inside domain entities or value objects

Since domain services are part of the domain, they should use ubiquitous language
like anything else in the domain.

Imagine you are writing some software in a financial domain. The domain
may require converting from different currencies, but to get the exchange
rate you need to interact with a 3rd party application. 

To do this we could introduce a domain service to model this interaction:

```csharp
public interface IRetriveExchangeRate
{
    ExchangeRate ExchangeRateBetween(Currency from, Currency to)
}
```

The domain service's interface should be part of the domain, since
it's addressing a business problem. The implementation 
for this will have to interact with a 3rd party over a REST API to get the
current exchange rate. The details of interacting over REST is a technical issue,
and so the implementation belongs in a infrastructure layer.

You can implement a domain service entirely inside the domain, if it uses
purely domain concepts and requires no infrastructure concerns.

### Application Services

Application services are the gateway to interacting with the domain,
the domain should not be directly exposed to the outside world since 
this can couple the domain with external code. This can make it
hard to change the domain without breaking the external code
depending on it. 

Applications services provide an api for interacting with the 
domain without directly interacting with the domain objects.
This means the domain can change, but the api can remain the 
same without breaking the outside services. No business logic 
is allowed inside the application services, it must belong
inside the domain.

Imagine you are writing the application services for a purchase order domain. You decide
to use the Command Handler pattern to implement your application services, and you end
up with the following code:

```csharp
public class PurchaseOrderCommandHandler : IHandleCommand<ApprovePurchaseOrderCommand>
{
    private IPurchaseOrderRepository PurchaseOrders {get;}

    //Constructor

    public void Handle(ApprovePurchaseOrderCommand command)
    {
        var purchaseOrder = PurchaseOrders.PurchaseOrderWithId(command.PurchaseOrderId)
        purchaseOrder.Approve();
    }

    //Other command handlers
}
```

Later on and the requirements change, and now the approval limits have to come
from a 3rd party service. A domain service is used to model this inside the
domain, and the approve method signature is modified the accept the 
new domain service.

Although the domain has changed, the `ApprovePurchaseOrderCommand` used to interact with
the domain does not. 

```csharp
public class PurchaseOrderCommandHandler : IHandleCommand<ApprovePurchaseOrderCommand>
{
    private IPurchaseOrderRepository PurchaseOrders {get;}

    private IApprovalLimitService ApprovalLimitService {get;}

    //Constructor...

    public void Handle(ApprovePurchaseOrderCommand command)
    {
        var purchaseOrder = PurchaseOrders.PurchaseOrderWithId(command.PurchaseOrderId)
        purchaseOrder.Approve(ApprovalLimitService);
    }

    //Other command handlers
}
```

This demonstrates how the use of application services can be used
to allow the domain to change, without breaking the applications
interacting with the domain.

## Further Reading

Domain Driven Design is a complicated subject and I can only explain the basics
of modelling with DDD within such a small article. For further information
please see the following:

The canonical text book for DDD - [Domain-driven Design: Tackling Complexity in the Heart of Software(Blue Book)](https://www.amazon.com/exec/obidos/ASIN/0321125215/domainlanguag-20) by Eric Evans

A good book for learning how to practically implement DDD - [Implementing Domain-Driven Design(Red Book) ](https://www.amazon.co.uk/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577) by Vaughn Vernon

A free chapter from Implementing Domain-Driven Design- [Implementing Domain-Driven Design: Aggregates ](https://www.infoq.com/minibooks/domain-driven-design-quickly)

https://www.infoq.com/minibooks/domain-driven-design-quickly
