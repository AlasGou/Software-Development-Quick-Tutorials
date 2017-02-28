# Domain Driven Design

Domain Driven Design (DDD) is a technique for building software with complex business domains. The core idea is 
to put the focus on building a shared software model that both software developers, and domain experts 
can understand. The software model should be only concerned with modelling the business issues in the
domain and include no technical issues like database persistance or handling requests.

When combined with Onion Architecture it can be a clean way of seperating the business issues from the 
technical issues. Onion Architecture is similar to both Ports & Adapters and Hexagonal Architecture to the 
point where they are otherwise the same. You can learn more about Onion Architecture at the following
[article](../Onion-Architecture).

## Ubiquitous Language

## Entity

An entity is an object that is unique and has an identity. A purchase order is an entity since it has a unique id.

## Value Object

A value object is an object that is defined by its attributes rather than having a unique identity. A money object would be a value object.

## Aggregate

An aggregate are a collection of objects that should act as a single unit. A purchase order aggregate may include a Purchase Order Entity, Address, a person, and line items. 
The aggeragates are where the *invariants*(a fancy term DDD proponents like to use) or business rules are enforced. For our purchase order example, we may say that a 
purchase order can not exist without at least 1 line item, and that a purchase order over a certain amount can not be automatically approved.

A good check to see if some objects should be grouped together as an aggregate is that objects within that group change togother when some business operation is 
applied. For example if the value of a line item changes, and the purchase order entity can no longer be automatically approved. 

An aggregate doesn't need to be explictly modelled in software, it can be implict by what the aggregate root references.

### Aggregate Root

The aggregate root is the entity at the top of an aggregate, it directly references or indirectly references all the objects inside of an aggeragate. It is the primary interface to an
aggeragate, and all actions on a aggeragate are performed via the aggergate root so that the invariants can be enforced. Any other objects outside of an aggeragate are not allowed
to directly reference any object inside the aggregate except for the aggeragate root.

## Repository

A repository is the method of retriving aggregate roots, it should not be used to retrive any other type of object in DDD. The repository interfaces are normally considered a part of the
 domain because they can contain business issues such as "retrive all unapproved purchase orders". The repository implementations are considered outside of the domain, and usally placed within a Infrastructure project and are injected
in at runtime. This is an example dependency inversion.

## Domain Events

A domain event is an event that tells the world that something has happened. For example we publish a PurchaseOrderApproved event which other objects may listen for, for example
an object listenting for PurchaseOrderApproved event may notifiy a manager that a purchase order has been approved via email. It is good way of linking up parts of the domain, 
and external systems without tightly coupling objects.

## Bounded Context

A large project may have multiple domain models, and to stop the domain models becoming blurred we should explictly define the business areas where the model applies. This is a bounded
context. For example when takling about purchase order, we could assume that this domain model is part of the purchasing domain which may include other entities like suppliers.

When using DDD in combination with microservices, the bounded context is a good way to specifiy the responsibilities of a service.

## Context Map

## DDD Antipatterns

## Tutorial

You know may know the terms of domain driven design, but it maybe hard to visualise how to apply this to an actual project. For this reason we going to apply DDD to an example
project. Lets apply domain driven design to a domain that a developer could understand both the domain, and the implementation. We are going to use DDD to model
the Scrum development framework. 

## Further Reading

Domain Driven Design is a complicated subject and I can only explain the basics of modelling with DDD within such a small article. For further information please
see the following.
