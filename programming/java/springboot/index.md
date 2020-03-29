# SpringBoot

## AutoGenerate UUID for id

```java
@Entity
public class Book {

	@Id
	@GeneratedValue(generator = “UUID”)
	@GenericGenerator(
		name = “UUID”,
		strategy = “org.hibernate.id.UUIDGenerator”,
	)
	@Column(name = “id”, updatable = false, nullable = false)
	private UUID id; // private String id;
}
```

## Unique data

```java
@Column(unique = true, nullable = false)
private String slug;
```

## Use @PrePersist

```java
@PrePersist
private void sluggify() {
    this.slug = Normalizer.normalize(new String(name.getBytes(), StandardCharsets.UTF_8), Normalizer.Form.NFKC);
}
```

## Create or update dates automatically

```java
@CreationTimestamp
private Date createdAt;

@UpdateTimestamp
private Date updatedAt;
```

## SpringBoot configuration db

```yaml
jpa:
  hibernate:
    ddl-auto: create-drop
    naming:
      implicit-strategy: org.hibernate.boot.model.naming.ImplicitNamingStrategyJpaCompliantImpl
```

### ddl-auto values

> Source: https://stackoverflow.com/questions/42135114/how-does-spring-jpa-hibernate-ddl-auto-property-exactly-work-in-spring

### naming.implicit-strategy values

> Source: 
> - https://stackoverflow.com/questions/41267416/hibernate-5-naming-strategy-examples
> - https://www.baeldung.com/hibernate-field-naming-spring-boot