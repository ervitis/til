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

## Using Criteria Builder

In this example we will using generics too to implement a solution for criteria builder

We have the `SearchData` class

```java
@Data
@AllArguments
@NoArguments
public class SearchData {
	private String name;

	private Long telephone;
}
```

Having the interface

```java
@Repository
public interface FilterableSearchCustomRepository<T> {
    Page<T> getWithFilters(Pageable pageable, SearchData searchData);
}
```

Next, we implement our class where the logic of the search to the database will be. We will use generics so we can pass any type of class that it uses our filterable.

```java
public class FilterableSearch<T> {

	private final EntityManager em;

	private final Class<T> t;

	public FilterableSearch(EntityManager em, Class<T> t) {
		this.em = em;
		this.t = t;
	}

	public Page<T> getWithFilters(Pageable pageable, SearchData searchData) {
		CriteriaBuilder cb = this.em.getCriteriaBuilder();
		CriteriaQuery<T> cq = cb.createQuery(this.t);

		Root<T> er = cq.from(this.t);
		List<Predicate> predicates = new ArrayList<>();

		if (searchData.getName() != null && !searchData.getName().isEmpty()) {
			predicates.add(cb.equal(er.get("name"), searchData.getName()));
		}

		if (searchData.getTelephone() != null) {
			predicates.add(cb.equal(er.get("telephone"), searchData.getTelephone()));
		}

		if (predicates.size() > 0) {
			cq.where(predicates.toArray(new Predicate[0]));
		}

		TypedQuery<T> query = this.em.createQuery(cq);

		query.setFirstResult(pageable.getPageNumber() * pageable.getPageSize());
		query.setMaxResults(pageable.getPageSize());

		return new PageImpl<T>(query.getResultList(), pageable, query.getResultList().size());
	}
}
```

For example, let's have a class named `Student` with the following properties:

```java
@Data
@AllArguments
@NoArguments
public class Student {
	private String name;

	private Long telephone;

	private String surname;
}
```

The REST controller will handle the `query parameters` and so on, so let's implement our custom search repository:

```java
@Component
public class FilterableSearchStudentCustomRepositoryImpl implements FilterabFilterableSearchCustomRepository<Student> {

	@Autowired
	private EntityManager entityManager;

	@Override
	public Page<Student> getWithFilters(Pageable pageable, SearchData searchData) {
		FilterableSearch<Student> search = new FilterableSearch<>(this.entityManager, Student.class);

		return search.getWithFilters(pageable, searchData);
	}
}
```