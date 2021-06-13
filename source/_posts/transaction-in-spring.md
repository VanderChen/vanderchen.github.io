---
title: Transactions in Spring
date: 2021-05-06
updated: {{date}}
tags: 
- Spring
---

## Basic concepts

- **Atomicity** It says that transaction is atomic in nature *i.e.* it is either full or it isn’t.
- **Consistency** It says that after the end of the transaction the system should be in a be a valid state.
- **Isolation** It says that all the transactions of the system should be done in isolation.
- **Durability** It says that a transaction should be durable when all the changes made to the system are permanent.

## Spring Isolation Level

- [ISOLATION_DEFAULT](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/TransactionDefinition.html#ISOLATION_DEFAULT)

  ```java
  if (isolationLevel != ISOLATION_DEFAULT) {
      if (currentTransactionIsolationLevel() != isolationLevel) {
          throw IllegalTransactionStateException
      }
  }
  ```

- [ISOLATION_READ_UNCOMMITTED](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/TransactionDefinition.html#ISOLATION_READ_COMMITTED) 一个事务要等另一个事务提交后才能读取数据，解决脏读问题。会出现不可重复读、幻读问题（锁定正在读取的行，适用于大多数系统，Oracle默认级别）

  ```java
  @Transactional(isolation = Isolation.READ_COMMITTED)
  public void log(String message){
      // ...
  }
  ```

- [ISOLATION_READ_COMMITTED](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/TransactionDefinition.html#ISOLATION_READ_UNCOMMITTED) 一个事务可以读取另一个未提交事务的数据。会出现脏读、不可重复读、幻读（隔离级别最低，但并发性高）

  ```java
  @Transactional(isolation = Isolation.READ_UNCOMMITTED)
  public void log(String message) {
      // ...
  }
  ```
  
- [ISOLATION_REPEATABLE_READ](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/TransactionDefinition.html#ISOLATION_REPEATABLE_READ) 在开始读取数据（事务开启）时，不再允许修改操作，解决不可重复读问题。会出现幻读问题（锁定所读的所有行，MYSQL默认级别）

  ```java
  @Transactional(isolation = Isolation.REPEATABLE_READ) 
  public void log(String message){
      // ...
  }
  ```
  
- [ISOLATION_SERIALIZABLE](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/TransactionDefinition.html#ISOLATION_SERIALIZABLE) 最高的事务隔离级别，在该级别下，事务串行化顺序执行，可以避免脏读、不可重复读与幻读。但是这种事务隔离级别效率低下，比较耗数据库性能，一般不使用。（锁整表）

  ```java
  @Transactional(isolation = Isolation.SERIALIZABLE)
  public void log(String message){
      // ...
  }
  ```

### Mapping between level & effects

|                  | dirty reads | non-repeatable reads | phantom reads |
| ---------------- | ----------- | -------------------- | ------------- |
| READ_UNCOMMITTED | Y           | Y                    | Y             |
| READ_COMMITTED   | N           | Y                    | Y             |
| REPEATABLE_READ  | N           | N                    | Y             |
| SERIALIZABLE     | N           | N                    | N             |

- **Dirty read:** read the uncommitted change of a concurrent transaction
- **Nonrepeatable read**: get different value on re-read of a row if a concurrent transaction updates the same row and commits
- **Phantom read:** get different rows after re-execution of a range query if another transaction adds or removes some rows in the range and commits

## Transaction Propagation

- PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。是Spring默认的传播行为。
- PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
- PROPAGATION_MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
- PROPAGATION_NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价PROPAGATION_REQUIRED。嵌套事务使用数据库中的保存点来实现，即嵌套事务回滚不影响外部事务，但外部事务回滚将导致嵌套事务回滚。

用例分析https://juejin.cn/post/6844903600943022088

