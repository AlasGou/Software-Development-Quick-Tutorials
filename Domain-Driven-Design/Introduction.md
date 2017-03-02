# Domain Driven Design

Domain Driven Design (DDD) is a technique for building software with complex business domains. The core idea is 
to put the focus on building a shared software model that both software developers, and domain experts 
can understand. The software model should be only concerned with modeling the business issues in the
domain and include no technical issues like database persistence or handling requests.

When combined with Onion Architecture it can be a clean way of separating the business issues from the 
technical issues. Onion Architecture is similar to both Ports & Adapters and Hexagonal Architecture to the 
point where they are otherwise the same. You can learn more about Onion Architecture at the following
[article](../Onion-Architecture).

## Ubiquitous Language

One of the higher level principles of DDD is Ubiquitous Language, it is something that can improve your code even when you are 
not using domain driven design. Although it is one the most underused parts of DDD with people preferring to dive straight into 
DDD modeling techniques like aggregates and entities. 

Ubiquitous Language allows domain experts and software developers to understand each other, and is core to making sure the software model matches the reality of the domain.
When discussing the domain software developers, and domain experts should share a language in order to understand each other. Discussion and collaboration
between software developers and domain experts is necessary to develop this language. It useful to document this
language in a internal wiki anyone can reference. The software model should also be implemented using ubiquitous language as 
much as possible to the point where the domain expert should be able to understand the code.

Imagine we are working on some software for administrating flu shots. On our first attempt we may attempt to use the following code to record a flu shot:

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

Notice this is almost readable without being programming expert, and much easier for the software developer and domain expert to
see if this is matching the reality of the business domain.

One thing to highlight is that the `vaccines` object that the method `standardAdultFluDose` is called on is a repository 
but it reads like English. This is important to highlight since a common pattern to use is generic repositories. Usually
with an interface such as:

```csharp
interface IRepository<T>
{
    Add(T)
    Delete(T)
    T Get(int id)
}
```

These generic repositories are tempting since they can reduce the amount of repository code required but it does make it difficult to 
use Ubiquitous Language. One way around this to have internal data access layer that all repository implementations can use, it won't be as concise
as generic repositories but will reduce the amount of internal implementation. 

```csharp
public VaccineRepository : IVaccineRepository
{
    private DataAccessLayer<Vaccine> DataAccess {get; set;}

    public Vaccine StandardAdultFluDose()
    {
        var standardVaccineId = //get vaccine id
        return DataAccess.Get(standardVaccineId)
    }
}

```
Since repository implementations are not normally classed as being inside the domain layer, it doesn't matter that we are not using 
Ubiquitous Language internally. External code operating on the domain layer won't see the internals of repositories, but they will see their interfaces!

## Domain Driven Design Modeling Techniques

### Value Object

A value object is an object that is defined by the value of its attributes rather than having a unique identity. A money object would be a value object.

```csharp
public class Money
{
    public Decimal Value { get; set;}
    public CurrencyType Currency  {get; set;}
}
```

### Entity

An entity is an object that is unique and has an identity. A purchase order is an entity since it has a unique id.

```csharp
public class PurchaseOrder
{
    public PurchaseOrderId PurchaseOrderId { get; set; }

    //Various properties and methods
}
```

### Aggregate

An aggregate are a collection of objects that should act as a single unit. A purchase order aggregate may include a Purchase Order Entity, Address, a person, and line items. 
The aggregates are where the *invariants*(a fancy term DDD proponents like to use) or business rules are enforced. For our purchase order example, we may say that a 
purchase order can not exist without at least 1 line item, and that a purchase order over a certain amount can not be automatically approved.

A good check to see if some objects should be grouped together as an aggregate is that objects within that group change together when some business operation is 
applied. For example if the value of a line item changes, and the purchase order entity can no longer be automatically approved. 

An aggregate doesn't need to be explicitly modeled in software, it can be implicit by what the aggregate root references.

```csharp
public class PurchaseOrder
{
    public PurchaseOrderId PurchaseOrderId { get; set; }

    private List<LineItem> LineItems { get; set; }

    private Money Total { get; set; }

    public void AddLineItem(LineItem lineItem)
    {
        LineItems.Add(lineItem);
        Total = Total + lineItem.Amount;

        if(Total > 5000)
        {
            DomainEvents.
        }
    }
}

```

#### Aggregate Root

The aggregate root is the entity at the top of an aggregate, it directly references or indirectly references all the objects inside of an aggregate. It is the primary interface to an
aggregate, and all actions on a aggregate are performed via the aggregate root so that the invariants can be enforced. Any other objects outside of an aggeragate are not allowed
to directly reference any object inside the aggregate except for the aggregate root.

### Repository

A repository is the method of retrieving aggregate roots, it should not be used to retrieve any other type of object in DDD. The repository interfaces are normally considered a part of the
 domain because they can contain business issues such as "retrieve all unapproved purchase orders". The repository implementations are considered outside of the domain, and usually placed within a Infrastructure project and are injected
in at runtime. This is an example dependency inversion.

### Domain Events

A domain event is an event that tells the world that something has happened. For example we publish a PurchaseOrderApproved event which other objects may listen for, for example
an object listening for PurchaseOrderApproved event may notify a manager that a purchase order has been approved via email. It is good way of linking up parts of the domain, 
and external systems without tightly coupling objects.

### Bounded Context

A large project may have multiple domain models, and to stop the domain models becoming blurred we should explicitly define the business areas where the model applies. This is a bounded
context. For example when talking about purchase order, we could assume that this domain model is part of the purchasing domain which may include other entities like suppliers.

When using DDD in combination with microservices, the bounded context is a good way to specify the responsibilities of a service.

### Context Map

### Domain Services

### Application Services

## DDD Anti-Patterns

## Tutorial

You know may know the terms of domain driven design, but it maybe hard to visualise how to apply this to an actual project. For this reason we going to apply DDD to an example
project. Lets apply domain driven design to a domain that a developer could understand both the domain, and the implementation. We are going to use DDD to model
the Scrum development framework. 

## Further Reading

Domain Driven Design is a complicated subject and I can only explain the basics of modelling with DDD within such a small article. For further information please
see the following.
