---
type: infopage
acronym: aggregate-references-via-ids
title: Aggregate References via typed IDs
navLabel: Aggregate References via typed IDs
summary: >
  Aggregates need to be "transaction boundaries" in order to be decoupled of each other. 
  One essential rule is to reference other aggrates only by ID reference, not by object reference.
  This prevent the temptation to modify other aggregates in the same transaction. It also helps
  keeping queries small. 
---

## Why typed IDs?

For IDs instead of object references, see also 
the [ArchiLab infopage on the 4 main aggregate design rules](https://www.archi-lab.io/infopages/ddd/aggregate-design-rules-vernon.html)
for a more detailed motivation.

But why typed IDs, and not just `UUID`, `Long` etc.? With such generic base types, an accidental mismatch
in class properties or in method parameters is just easily possible. The compiler cannot distinguish
between `UUID productId` and `UUID customerId` - they are interchangeable, and can be confused. 
Therefore, we should use typed IDs (`ProductId` and `CustomerId` in our example).


## Abstract Parent Classes

It is useful to provide an abstract ID class: 
```java
@MappedSuperclass
@Getter
@EqualsAndHashCode(of = "id")
public abstract class GenericId {
    @Column(nullable = false, updatable = false)
    private final UUID id;

    protected GenericId() {
        this( UUID.randomUUID() );
    }
    protected GenericId( UUID id ) {
        this.id = id;
    }
}
```

In addition, we need a converter to ensures the proper DB mapping of our dedicated ID types. 
Here, we also provide an abstract parent class:
```java
public abstract class GenericIdConverter<T extends GenericId> implements AttributeConverter<T, UUID> {
    private final Function<UUID, T> factory;

    protected GenericIdConverter( Function<UUID, T> factory ) {
        this.factory = factory;
    }
    @Override
    public UUID convertToDatabaseColumn( T attribute ) {
        return attribute == null ? null : attribute.getId();
    }
    @Override
    public T convertToEntityAttribute( UUID dbData ) {
        return dbData == null ? null : factory.apply( dbData );
    }
}
```


## How To Use

For each of our entities, we need to implement a dedicated ID class ...
```java
@Embeddable
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AttributeOverride(name = "id", column = @Column(name = "customer_id"))
public class CustomerId extends GenericId {
    public CustomerId( UUID id ) {
        super( id );
    }
}
```

... and also a converter class:
```
@Converter(autoApply = true)
public class CustomerIdConverter extends GenericIdConverter<CustomerId> {
    public CustomerIdConverter() {
        super(CustomerId::new);
    }
}
```

**NOTE**: The converter is never directly used in your code! It just needs to be there
for the Spring framework to find.

Then, we can use the ID in our entity like this. Make sure that you 
**initialize the ID in each constructor** (see below)!
```
@Entity
public class Customer {
    @Setter(AccessLevel.PRIVATE)    // only for JPA
    @EmbeddedId
    private CustomerId id;
    
    public Customer(...) {
        // ...
        this.id = new CustomerId();
        // ...
    }
}    
```
