# JPA Fetch Join & Paging

ToMany 관계에서 Fetch Join과 페이징을 함께 사용하면 OutOfMemoryError 발생 할 수 있음

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(mappedBy = "product", cascade = CascadeType.PERSIST)
    private List categories = new ArrayList();

    // ... 중략
}

@Entity
@Getter
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
class ProductCategory {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    private Product product;

    @ManyToOne(fetch = FetchType.LAZY)
    private Category category;
}

interface ProductJpaRepository extends JpaRepository {

    @Query("""
                 select p
                 from Product p
                 join fetch p.categories pc
                 join fetch pc.category c
                 order by p.id desc
            """)
    Slice findProductWithSlice(Pageable pageable);
}
```

`findProductWithSlice` 호출 시 경고 메시지 표시

```bash
firstResult/maxResults specified with collection fetch; applying in memory
```

실행되는 쿼리를 확인해도 페이징 쿼리를 실행하지 않음

```sql
select
        p.id,
        pc.product_id,
        pc.id,
        c.id,
        c.name 
    from
       product p 
    join
       product_category pc 
       on p.id = pc.product_id 
    join
       category c 
       on c.id = pc.category_id 
    order by p.id desc
```

ProductCategory를 조인하면 Product의 결과도 함께 증가(카티션 프로덕트)

페이징을 위해 설정한 값이 의도한 대로 동작하기 어려워,

JPA는 전체 결과를 메모리에 적재한 다음 가공하여 페이징 수행

이때, 수많은 데이터가 메모리에 적재된다면 OOM 발생 가능

### 해결법

Fetch Join을 사용하지 않으면 되지만,

그러면 ProductCategory 리스트를 조회하기 위해서 N + 1 쿼리 발생 가능

```sql
Slice result = productJpaRepository.findProductWithSlice(pageRequest);
result.forEach(product -> System.out.println(product.getCategories())); // N + 1
```

`@BatchSize`와 default_batch_fetch_size 옵션 사용 가능

Parent Entity(Product)의 Child Entity 컬렉션(ProductCategory)을 조회할 때,

영속성 컨텍스트에서 관리하는 Parent의 식별자를 IN 절에 추가하여 Child 조회하는 기능

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @BatchSize(size = 10)
    @OneToMany(mappedBy = "product", cascade = CascadeType.PERSIST)
    private List categories = new ArrayList();

    // ... 중략
}
```

쿼리 실행 시

```sql
select
        pc.product_id,
        pc.id,
        pc.category_id 
    from
        product_category pc 
    where
        pc.product_id in (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
```