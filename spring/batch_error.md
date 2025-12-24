# @StepScope + JpaItemReader EntityManager Null Pointer exception
```java
@Bean
@StepScope
public ItemReader<? extends Customer> batchReader() {
return new JpaPagingItemReaderBuilder<Customer>()
.name("partitionStepJpaReader")
.currentItemCount(0)
.entityManagerFactory(emf)
.maxItemCount(1000)
.queryString("select c from Customer c")
.build();
}
```
`JpaPagingItemReader`를 생성하는 구문에서 리턴 타입이 ItemReader로 되어 있음  
하지만 open() 메서드에서 ItemStream을 요구
때문에 `@StepScope`에 의해서 Proxy로 생성된 객체가 실행 시점에 실제 타겟 빈을 찾아 호출하게 되면,  
타겟 빈이 ItemStream 타입으로 구현되어 있는지 체크하고,  
open() 메소드를 실행하게 되는데,  
타겟 빈이 ItemReader 타입의 객체로만 생성되었기 때문에 EntityManager가 생성되지 못한 것  
```java
@Override
protected void doOpen() throws Exception {
   super.doOpen();

   entityManager = entityManagerFactory.createEntityManager(jpaPropertyMap);
   if (entityManager == null) {
      throw new DataAccessResourceFailureException("Unable to obtain an EntityManager");
   }
   // set entityManager to queryProvider, so it participates
   // in JpaPagingItemReader's managed transaction
   if (queryProvider != null) {
      queryProvider.setEntityManager(entityManager);
   }

}
```
`JpaPagingItemReader`를 생성하는 메서드의 리턴 타입을 `itemReader`와 `itemStream` 동시에 구현한 타입으로 리턴하면 됨  
```java
@Bean
@StepScope
public ItemStreamReader<? extends Customer> batchReader() {
    return new JpaPagingItemReaderBuilder<Customer>()
            .name("partitionStepJpaReader")
            .currentItemCount(0)
            .entityManagerFactory(emf)
            .maxItemCount(1000)
            .queryString("select c from Customer c")
            .build();
}
```