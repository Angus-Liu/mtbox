> [Spring Data Elasticsearch](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/)

# Spring Data Elasticsearch

# 前言

Spring Data Elasticsearch项目使用Elasticsearch搜索引擎将核心Spring概念应用于解决方案的开发。我们提供了一个“模板”作为存储，查询，排序和分面文档的高级抽象。您会注意到Spring Framework中Spring数据solr和mongodb支持的相似之处。

## 项目元数据

- 版本控制 - <https://github.com/spring-projects/spring-data-elasticsearch>
- Bugtracker - <https://jira.spring.io/browse/DATAES>
- 发布存储库 - <https://repo.spring.io/libs-release>
- 里程碑存储库 - <https://repo.spring.io/libs-milestone>
- 快照存储库 - <https://repo.spring.io/libs-snapshot>

## 要求

如果您使用嵌入式节点客户端，则需要[Elasticsearch](http://www.elasticsearch.org/download/) 0.20.2及更高版本或可选依赖项

## 1.使用Spring Data Repositories

Spring Data存储库抽象的目标是显着减少为各种持久性存储实现数据访问层所需的样板代码量。

> Spring Data存储库文档和您的模块
>
> 本章介绍Spring Data存储库的核心概念和接口。本章中的信息来自Spring Data Commons模块。它使用Java Persistence API（JPA）模块的配置和代码示例。您应该将XML名称空间声明和要扩展的类型调整为您使用的特定模块的等效项。“ [命名空间参考](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.namespace-reference) ”涵盖XML配置，支持存储库API的所有Spring Data模块都支持XML配置。“ [存储库查询关键字](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repository-query-keywords) ”涵盖了存储库抽象支持的查询方法关键字。有关模块特定功能的详细信息，请参阅本文档该模块的章节。

### 1.1 核心概念

Spring Data存储库抽象中的中央接口是`Repository`。它将域类以及域类的ID类型作为类型参数进行管理。此接口主要用作标记接口，用于捕获要使用的类型，并帮助您发现扩展此接口的接口。该`CrudRepository`规定对于正在管理的实体类复杂的CRUD功能。

例1 `CrudRepository` 接口

```java
public interface CrudRepository<T, ID extends Serializable>
  extends Repository<T, ID> {

  // 保存给定的实体
  <S extends T> S save(S entity);

  // 返回由给定 ID 标识的实体
  Optional<T> findById(ID primaryKey); 

  // 返回所有实体
  Iterable<T> findAll();               

  // 返回实体数量
  long count();                        

  // 删除给定的实体
  void delete(T entity);               

  // 指示是否存在具有给定 ID 的实体
  boolean existsById(ID primaryKey);   

  // ... 更多的功能省略
}
```

> 我们还提供持久性技术特定的抽象，例如`JpaRepository`或`MongoRepository`。`CrudRepository`除了相当通用的持久性技术无关的接口之外，这些接口还扩展和公开了底层持久性技术的功能`CrudRepository`。

最重要的是`CrudRepository`，有一个`PagingAndSortingRepository`抽象添加了额外的方法来简化对实体的分页访问：

例2 `PagingAndSortingRepository` 接口

```java
public interface PagingAndSortingRepository<T, ID extends Serializable> extends CrudRepository<T, ID> {

  Iterable<T> findAll(Sort sort);

  Page<T> findAll(Pageable pageable);
}
```

要访问`User`页面大小为20 的第二页，您可以执行以下操作：

```java
PagingAndSortingRepository<User, Long> repository = // … 获得对bean的访问权限
Page<User> users = repository.findAll(PageRequest.of(1, 20));
```

除了查询方法之外，还可以使用计数和删除查询的查询派生。以下列表显示派生计数查询的接口定义：

例3 派生计数查询

```java
interface UserRepository extends CrudRepository<User, Long> {

  long countByLastname(String lastname);
}
```

以下列表显示了派生删除查询的接口定义：

例4 派生的删除查询

```java
interface UserRepository extends CrudRepository<User, Long> {

  long deleteByLastname(String lastname);

  List<User> removeByLastname(String lastname);
}
```

### 1.2 查询方法

标准CRUD功能存储库通常对基础数据存储区进行查询。使用Spring Data，声明这些查询将分为四个步骤：

1. 声明扩展Repository或其子接口之一的接口，并将其键入应处理的域类和ID类型，如以下示例所示：

   ```java
   interface PersonRepository extends Repository<Person, Long> { … }
   ```

2. 在接口上声明查询方法。

   ```java
   interface PersonRepository extends Repository<Person, Long> {
     List<Person> findByLastname(String lastname);
   }
   ```

3. 设置Spring以使用[JavaConfig](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.create-instances.java-config)或[XML配置](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.create-instances)为这些接口创建代理实例。

   a.要使用Java配置，请创建类似于以下内容的类：

   ```java
   import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
   
   @EnableJpaRepositories
   class Config {}
   ```

   b.要使用XML配置，请定义类似于以下内容的bean：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:jpa="http://www.springframework.org/schema/data/jpa"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/data/jpa
        http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">
   
      <jpa:repositories base-package="com.acme.repositories"/>
   
   </beans>
   ```
   在此示例中使用JPA名称空间。如果对任何其他商店使用存储库抽象，则需要将其更改为商店模块的相应名称空间声明。换句话说，你应该换一个jpa赞成，例如，mongodb。
   
   另外，请注意JavaConfig变体不会显式配置包，因为默认情况下使用带注释的类的包。要自定义要扫描的包，请使用特定于数据存储库的@Enable${store}Repositories 注解的属性之一 basePackage…。

4. 注入存储库实例并使用它，如以下示例所示：

   ```java
   class SomeClient {
   
     private final PersonRepository repository;
   
     SomeClient(PersonRepository repository) {
       this.repository = repository;
     }
   
     void doSomething() {
       List<Person> persons = repository.findByLastname("Matthews");
     }
   }
   ```

以下部分详细说明了每个步骤：

- [定义存储库接口](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.definition)
- [定义查询方法](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.query-methods.details)
- [创建存储库实例](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.create-instances)
- [Spring数据存储库的自定义实现](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.custom-implementations)

### 1.3 定义存储库接口

首先，定义特定于域类的存储库接口。接口必须扩展`Repository`并键入域类和ID类型。如果要为该域类型公开CRUD方法，请扩展`CrudRepository`而不是`Repository`。

#### 1.3.1 微调存储库定义

通常情况下，你的 repository 接口扩展`Repository`，`CrudRepository`或`PagingAndSortingRepository`。或者，如果您不想扩展Spring Data接口，还可以使用`@RepositoryDefinition`注解来注释您的 repository  接口。扩展`CrudRepository`公开了一整套方法来操纵您的实体。如果您希望对要公开的方法有选择性，请将要公开的方法从`CrudRepository`复制到你的领域 repository  中。

> 这样做可以让您在提供的Spring Data Repositories功能之上定义自己的抽象。

下面的示例示出了如何以选择性地露出CRUD方法（`findById`和`save`，在这种情况下）：

例5 有选择地暴露CRUD方法

```java
@NoRepositoryBean
interface MyBaseRepository<T, ID extends Serializable> extends Repository<T, ID> {

  Optional<T> findById(ID id);

  <S extends T> S save(S entity);
}

interface UserRepository extends MyBaseRepository<User, Long> {
  User findByEmailAddress(EmailAddress emailAddress);
}
```

在前面的例子，你为所有 repositor 定义了一个共同的基础接口，暴露了`findById(…)`，以及`save(…)` 方法。这些方法被发送到基础信息库实现你所选择的由Spring提供的数据（例如，如果使用JPA商店，实现是`SimpleJpaRepository`），因为它们匹配方法签名`CrudRepository`。因此，`UserRepository`现在可以保存用户，按ID查找单个用户，并触发查询以`Users`通过电子邮件地址查找。

> 中间存储库接口使用注释`@NoRepositoryBean`。确保将该注解添加到Spring Data不应在运行时创建实例的所有存储库接口。

#### 1.3.2 存储库方法的空处理

从Spring Data 2.0开始，返回单个聚合实例的存储库CRUD方法使用Java 8 `Optional`来指示可能缺少值。除此之外，Spring Data支持在查询方法上返回以下包装类型：

- `com.google.common.base.Optional`
- `scala.Option`
- `io.vavr.control.Option`
- `javaslang.control.Option` （不推荐使用Javaslang）

或者，查询方法可以选择根本不使用包装类型。然后通过返回指示缺少查询结果`null`。保证返回集合，集合替代，包装器和流的存储库方法永远不会返回`null`，而是相应的空表示。有关详细信息，请参阅“ [存储库查询返回类型](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repository-query-return-types) ”。

**可空性注释**

您可以使用[Spring Framework的可空性注释](https://docs.spring.io/spring/docs/5.1.7.RELEASE/spring-framework-reference/core.html#null-safety)来表达存储库方法的可空性约束。它们`null`在运行时提供了一种工具友好的方法和选择性检查，如下所示：

- [`@NonNullApi`](https://docs.spring.io/spring/docs/5.1.7.RELEASE/javadoc-api/org/springframework/lang/NonNullApi.html)：在包级别上使用，以声明参数和返回值的默认行为是不接受或生成`null`值。
- [`@NonNull`](https://docs.spring.io/spring/docs/5.1.7.RELEASE/javadoc-api/org/springframework/lang/NonNull.html)：用于参数或返回值不能为`null` （参数不需要，返回值`@NonNullApi`适用）。
- [`@Nullable`](https://docs.spring.io/spring/docs/5.1.7.RELEASE/javadoc-api/org/springframework/lang/Nullable.html)：用于参数或返回值可以为`null`。

Spring注释是使用[JSR 305](https://jcp.org/en/jsr/detail?id=305)注释进行元注释的（一种休眠但广泛传播的JSR）。JSR 305元注释允许[IDEA](https://www.jetbrains.com/help/idea/nullable-and-notnull-annotations.html)，[Eclipse](https://help.eclipse.org/oxygen/index.jsp?topic=/org.eclipse.jdt.doc.user/tasks/task-using_external_null_annotations.htm)和[Kotlin](https://kotlinlang.org/docs/reference/java-interop.html#null-safety-and-platform-types)等工具供应商以通用方式提供空安全支持，而无需对Spring注释进行硬编码支持。要为查询方法启用运行时检查可空性约束，需要使用Spring的`@NonNullApi`in 来激活包级别的非可空性`package-info.java`，如以下示例所示：

例6 声明中的非可空性 `package-info.java`

```java
@org.springframework.lang.NonNullApi
package com.acme;
```

一旦存在非null默认值，就会在运行时验证存储库查询方法调用的可空性约束。如果查询执行结果违反了定义的约束，则抛出异常。当方法返回`null`但声明为非可空时（默认情况下，存储库所在的包中定义了注释）会发生这种情况。如果您想再次选择加入可空的结果，请有选择地使用`@Nullable`单个方法。使用本节开头提到的结果包装器类型将继续按预期工作：空结果将转换为表示缺席的值。

以下示例显示了刚才描述的一些技术：

例7 使用不同的可空性约束

```java
// 存储库驻留在我们已定义非空行为的包（或子包）中。
package com.acme;                                                       

import org.springframework.lang.Nullable;

interface UserRepository extends Repository<User, Long> {

  // 抛出一个EmptyResultDataAccessException在执行时不产生结果的查询。抛出一个IllegalArgumentException当emailAddress交给该方法null。
  User getByEmailAddress(EmailAddress emailAddress);                    

  // null执行查询时不返回结果。也接受null作为的值emailAddress。
  @Nullable
  User findByEmailAddress(@Nullable EmailAddress emailAdress);          

  // Optional.empty()执行查询时不返回结果。抛出一个IllegalArgumentException当emailAddress交给该方法null。
  Optional<User> findOptionalByEmailAddress(EmailAddress emailAddress); 
}
```

**基于Kotlin的存储库中的可空性**

Kotlin 对语言中的可[空性约束进行](https://kotlinlang.org/docs/reference/null-safety.html)了定义。Kotlin代码编译为字节码，它不通过方法签名表达可空性约束，而是通过编译元数据表达。确保`kotlin-reflect`在项目中包含JAR，以便对Kotlin的可空性约束进行内省。Spring Data存储库使用语言机制来定义这些约束以应用相同的运行时检查，如下所示：

例8 对Kotlin存储库使用可空性约束

```kotlin
interface UserRepository : Repository<User, String> {

  // 该方法将参数和结果都定义为非可空（Kotlin默认值）。Kotlin编译器拒绝传递null给该方法的方法调用。如果查询执行产生空结果，EmptyResultDataAccessException则抛出a
  fun findByUsername(username: String): User     

  // 该方法接受null的firstname参数，如果查询执行不产生结果，返回null
  fun findByFirstname(firstname: String?): User? 
}
```

#### 1.3.3 使用具有多个Spring数据模块的存储库

在应用程序中使用唯一的Spring Data模块会使事情变得简单，因为定义范围内的所有存储库接口都绑定到Spring Data模块。有时，应用程序需要使用多个Spring Data模块。在这种情况下，存储库定义必须区分持久性技术。当它在类路径上检测到多个存储库工厂时，Spring Data进入严格的存储库配置模式。严格配置使用存储库或域类的详细信息来确定存储库定义的Spring Data模块绑定：

1. 如果存储库定义[扩展了特定于模块的存储库](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.multiple-modules.types)，那么它是特定Spring Data模块的有效候选者。
2. 如果[使用特定于模块的类型注释](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.multiple-modules.annotations)对域类进行[注释](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.multiple-modules.annotations)，则它是特定Spring Data模块的有效候选者。Spring Data模块接受第三方注释（例如JPA `@Entity`）或提供自己的注释（例如`@Document`Spring Data MongoDB和Spring Data Elasticsearch）。

以下示例显示了使用特定于模块的接口的存储库（在本例中为JPA）：

例9 使用模块特定接口的存储库定义

```java
// MyRepository 和 UserRepository  在其类型层次结构中继承 JpaRepository。
// 它们是Spring Data JPA模块的有效候选者。
interface MyRepository extends JpaRepository<User, Long> { }

@NoRepositoryBean
interface MyBaseRepository<T, ID extends Serializable> extends JpaRepository<T, ID> {
  …
}

interface UserRepository extends MyBaseRepository<User, Long> {
  …
}
```

以下示例显示了使用通用接口的存储库：

例10 使用通用接口的存储库定义

```java
// AmbiguousRepository 和 AmbiguousUserRepository 在他们的类型层次仅继承 Repository，
// 和 CrudRepository。虽然这在使用独特的Spring Data模块时非常好，
// 但是多个模块无法区分这些存储库应该绑定到哪个特定的Spring Data。
interface AmbiguousRepository extends Repository<User, Long> {
 …
}

@NoRepositoryBean
interface MyBaseRepository<T, ID extends Serializable> extends CrudRepository<T, ID> {
  …
}

interface AmbiguousUserRepository extends MyBaseRepository<User, Long> {
  …
}
```

以下示例显示了使用带注释的域类的存储库：

例11 使用带注释的域类的存储库定义

```java
// PersonRepository引用了使用JPA @Entity进行注解的 Person，因此该存储库显然属于Spring Data JPA
// UserRepository引用User，User 使用Spring Data MongoDB的@Document注释进行注释。
interface PersonRepository extends Repository<Person, Long> {
 …
}

@Entity
class Person {
  …
}

interface UserRepository extends Repository<User, Long> {
 …
}

@Document
class User {
  …
}
```

以下错误示例显示了使用具有混合注释的域类的存储库：

例12 使用具有混合注释的域类的存储库定义

```java
// 此示例显示了使用JPA和Spring Data MongoDB注释的域类。
// 它定义了两个存储库，JpaPersonRepository和MongoDBPersonRepository。
// 一个用于JPA，另一个用于MongoDB用法。Spring Data不再能够将存储库分开，从而导致未定义的行为。
interface JpaPersonRepository extends Repository<Person, Long> {
 …
}

interface MongoDBPersonRepository extends Repository<Person, Long> {
 …
}

@Entity
@Document
class Person {
  …
}
```

[存储库类型详细信息](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.multiple-modules.types)和[区分域类注释](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.multiple-modules.annotations)用于严格存储库配置，以识别特定Spring数据模块的存储库候选。在同一域类型上使用多个持久性技术特定的注释是可能的，并允许跨多种持久性技术重用域类型。但是，Spring Data不再能够确定用于绑定存储库的唯一模块。

区分存储库的最后一种方法是通过确定存储库基础包的范围。基础包定义了扫描存储库接口定义的起点，这意味着将存储库定义放在相应的包中。默认情况下，注释驱动的配置使用配置类的包。[基于XML的配置中](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.create-instances.spring)的[基础包](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.create-instances.spring)是必需的。

以下示例显示了基本包的注释驱动配置：

示例13.基础包的注释驱动配置

```java
@EnableJpaRepositories(basePackages = "com.acme.repositories.jpa")
@EnableMongoRepositories(basePackages = "com.acme.repositories.mongo")
interface Configuration { }
```

### 1.4 定义查询方法

存储库代理有两种方法可以从方法名称派生特定于存储库的查询：

- 通过直接从方法名称派生查询。
- 通过使用手动定义的查询。

可用选项取决于实际存储库。但是，必须有一个策略来决定创建实际查询的内容。下一节将介绍可用选项。

#### 1.4.1 查询查找策略

存储库基础结构可以使用以下策略来解析查询。使用XML配置，您可以通过`query-lookup-strategy`属性在命名空间配置策略。对于Java配置，您可以使用注释的`queryLookupStrategy`属性`Enable${store}Repositories`。特定数据存储可能不支持某些策略。

- `CREATE`尝试从查询方法名称构造特定于存储的查询。一般方法是从方法名称中删除一组已知的前缀，并解析方法的其余部分。您可以在“ [查询创建](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.query-methods.query-creation) ”中阅读有关查询构造的更多信息。
- `USE_DECLARED_QUERY`尝试查找声明的查询，如果找不到，则抛出异常。查询可以通过某处的注释来定义，也可以通过其他方式声明。查阅特定商店的文档以查找该商店的可用选项。如果存储库基础结构在引导时未找到该方法的声明查询，则它将失败。
- `CREATE_IF_NOT_FOUND`（默认）组合`CREATE`和`USE_DECLARED_QUERY`。它首先查找声明的查询，如果没有找到声明的查询，它会创建一个基于自定义方法名称的查询。这是默认的查找策略，因此，如果您未明确配置任何内容，则使用此策略。它允许通过方法名称进行快速查询定义，还可以根据需要引入声明的查询来自定义这些查询。

#### 1.4.2 查询创建

Spring Data存储库基础结构中内置的查询构建器机制对于构建对存储库实体的约束查询非常有用。该机制条前缀`find…By`，`read…By`，`query…By`，`count…By`，和`get…By`从所述方法和开始分析它的其余部分。introduction子句可以包含更多表达式，例如`Distinct`在要创建的查询上设置不同的标志。但是，第一个`By`用作分隔符以指示实际标准的开始。在最基本的层面上，您可以在实体属性上定义条件，并将它们与`And`和它们连接起来`Or`。以下示例显示了如何创建大量查询：

示例14.从方法名称创建查询

```java
interface PersonRepository extends Repository<User, Long> {

  List<Person> findByEmailAddressAndLastname(EmailAddress emailAddress, String lastname);

  // Enables the distinct flag for the query
  List<Person> findDistinctPeopleByLastnameOrFirstname(String lastname, String firstname);
  List<Person> findPeopleDistinctByLastnameOrFirstname(String lastname, String firstname);

  // Enabling ignoring case for an individual property
  List<Person> findByLastnameIgnoreCase(String lastname);
  // Enabling ignoring case for all suitable properties
  List<Person> findByLastnameAndFirstnameAllIgnoreCase(String lastname, String firstname);

  // Enabling static ORDER BY for a query
  List<Person> findByLastnameOrderByFirstnameAsc(String lastname);
  List<Person> findByLastnameOrderByFirstnameDesc(String lastname);
}
```

解析方法的实际结果取决于您为其创建查询的持久性存储。但是，有一些一般要注意的事项：

- 表达式通常是属性遍历与可以连接的运算符相结合。您可以将属性表达式与`AND`和组合使用`OR`。您还可以得到这样的运营商为支撑`Between`，`LessThan`，`GreaterThan`，和`Like`该属性的表达式。支持的运算符可能因数据存储而异，因此请参阅参考文档的相应部分。
- 方法解析器支持`IgnoreCase`为各个属性（例如`findByLastnameIgnoreCase(…)`）或支持忽略大小写的类型（`String`例如，通常为实例）的所有属性设置标志`findByLastnameAndFirstnameAllIgnoreCase(…)`。是否支持忽略大小写可能因商店而异，因此请参阅参考文档中有关特定于商店的查询方法的相关章节。
- 您可以通过`OrderBy`在引用属性的查询方法中附加子句并提供排序方向（`Asc`或`Desc`）来应用静态排序。要创建支持动态排序的查询方法，请参阅“ [特殊参数处理](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.special-parameters) ”。

#### 1.4.3 属性表达

属性表达式只能引用被管实体的直接属性，如前面的示例所示。在创建查询时，您已确保已解析的属性是托管域类的属性。但是，您也可以通过遍历嵌套属性来定义约束。请考虑以下方法签名：

```java
List<Person> findByAddressZipCode(ZipCode zipCode);
```

假设 `Person`有`Address`和一个`ZipCode`。在这种情况下，该方法创建属性遍历`x.address.zipCode`。解析算法首先将整个part（`AddressZipCode`）解释为属性，并检查域类中是否具有该名称的属性（未大写）。如果算法成功，它将使用该属性。如果没有，算法将来自右侧的驼峰部分的源分成头部和尾部，并尝试找到相应的属性 - 在我们的示例中，`AddressZip`和`Code`。如果算法找到具有该头部的属性，则它采用尾部并继续从那里构建树，以刚刚描述的方式将尾部分开。如果先拆不匹配，则算法移动分割点左侧（`Address`，`ZipCode`）并继续。

虽然这应该适用于大多数情况，但算法可能会选择错误的属性。假设`Person`该类也具有`addressZip`属性。算法将在第一个拆分轮中匹配，选择错误的属性，并且失败（因为类型`addressZip`可能没有`code`属性）。

要解决这种歧义，您可以`_`在方法名称中使用手动定义遍历点。所以我们的方法名称如下：

```java
List<Person> findByAddress_ZipCode(ZipCode zipCode);
```

因为我们将下划线字符视为保留字符，所以我们强烈建议遵循标准Java命名约定（即，不在属性名称中使用下划线，而是使用camel case）。

#### 1.4.4 特殊参数处理

要处理查询中的参数，请定义方法参数，如前面示例中所示。除此之外，基础设施承认某些特定的类型，如`Pageable`和`Sort`，动态地应用分页和排序，以查询。以下示例演示了这些功能：

例15 使用`Pageable`, `Slice`和`Sort`查询方法

```java
Page<User> findByLastname(String lastname, Pageable pageable);

Slice<User> findByLastname(String lastname, Pageable pageable);

List<User> findByLastname(String lastname, Sort sort);

List<User> findByLastname(String lastname, Pageable pageable);
```

第一种方法允许您将`org.springframework.data.domain.Pageable`实例传递给查询方法，以动态地将分页添加到静态定义的查询中。`Page`知道可用元素和页面的总数。它通过基础设施触发计数查询来计算总数来实现。由于这可能很昂贵（取决于所使用的存储），您可以改为返回`Slice`。`Slice`只知道下一个`Slice`是否可用，这在遍历更大的结果集时可能就足够了。

排序选项也通过`Pageable`实例处理。如果只需要排序，`org.springframework.data.domain.Sort`请在方法中添加参数。如您所见，返回`List`也是可能的。在这种情况下，`Page`不会创建构建实际实例所需的其他元数据（这反过来意味着不会发出必要的附加计数查询）。相反，它限制查询仅查找给定范围的实体。

> 要了解整个查询的页数，必须触发其他计数查询。默认情况下，此查询是从您实际触发的查询派生的。

#### 1.4.5 限制查询结果

查询方法的结果可以通过使用`first`或`top`关键字来限制，可以互换使用。可以附加一个可选的数值，`top`或者`first`指定要返回的最大结果大小。如果省略该数字，则假定结果大小为1。以下示例显示如何限制查询大小：

例16 使用`Top`和`First`限制查询的结果大小

```java
User findFirstByOrderByLastnameAsc();

User findTopByOrderByAgeDesc();

Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);

Slice<User> findTop3ByLastname(String lastname, Pageable pageable);

List<User> findFirst10ByLastname(String lastname, Sort sort);

List<User> findTop10ByLastname(String lastname, Pageable pageable);
```

限制表达式也支持`Distinct`关键字。此外，对于将结果集限制为一个实例的查询，支持将结果包装到`Optional`关键字中。

如果将分页或切片应用于限制查询分页（以及可用页数的计算），则将其应用于有限结果中。

> 通过使用`Sort`参数将结果与动态排序结合使用，可以表达“K”最小元素和“K”元素的查询方法。

#### 1.4.6 流式查询结果

可以使用Java 8 `Stream<T>`作为返回类型以递增方式处理查询方法的结果。而不是将查询结果包装在`Stream`数据存储中的特定方法用于执行流式传输，如以下示例所示：

例17 使用Java 8流式传输查询结果 `Stream<T>`

```java
@Query("select u from User u")
Stream<User> findAllByCustomQueryAndStream();

Stream<User> readAllByFirstnameNotNull();

@Query("select u from User u")
Stream<User> streamAllPaged(Pageable pageable);
```

> `Stream`潜在包装底层数据存储专用资源，因此必须在使用之后被关闭。您可以`Stream`使用该`close()`方法或使用Java 7 `try-with-resources`块手动关闭，如以下示例所示：

示例18. `Stream<T>`在try-with-resources块中使用结果

```java
try (Stream<User> stream = repository.findAllByCustomQueryAndStream()) {
  stream.forEach(…);
}
```

> 并非所有Spring Data模块当前都支持`Stream<T>`返回类型。

#### 1.4.7 异步查询结果

可以使用[Spring的异步方法执行功能异步](https://docs.spring.io/spring/docs/5.1.7.RELEASE/spring-framework-reference/integration.html#scheduling)运行存储库查询。这意味着该方法在调用时立即返回，而实际的查询执行发生在已提交给Spring的任务中`TaskExecutor`。异步查询执行与响应式查询执行不同，不应混合使用。有关响应支持的更多详细信息，请参阅特定于存储的文档。以下示例显示了许多异步查询：

```java
// 使用java.util.concurrent.Future作为返回类型。
@Async
Future<User> findByFirstname(String firstname);               

// 使用Java 8 java.util.concurrent.CompletableFuture作为返回类型。
@Async
CompletableFuture<User> findOneByFirstname(String firstname); 

// 使用org.springframework.util.concurrent.ListenableFuture作为返回类型。
@Async
ListenableFuture<User> findOneByLastname(String lastname);    
```

### 1.5 创建存储库实例

在本节中，您将为定义的存储库接口创建实例和bean定义。一种方法是使用随每个支持存储库机制的Spring Data模块一起提供的Spring命名空间，尽管我们通常建议使用Java配置。

#### 1.5.1 XML配置

每个Spring Data模块都包含一个`repositories`元素，允许您定义Spring为您扫描的基础包，如以下示例所示：

例19 通过XML启用Spring Data存储库

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns="http://www.springframework.org/schema/data/jpa"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/data/jpa
    http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

  <repositories base-package="com.acme.repositories" />

</beans:beans>
```

在前面的示例中，指示Spring扫描`com.acme.repositories`其所有子包以进行扩展`Repository`或其子接口之一。对于找到的每个接口，基础结构都会注册特定`FactoryBean`于持久性技术的内容，以创建处理查询方法调用的相应代理。每个bean都是在从接口名称派生的bean名称下注册的，因此`UserRepository`将在其下注册一个接口`userRepository`。该`base-package`属性允许使用通配符，以便您可以定义扫描包的模式。

**使用过滤器**

默认情况下，基础结构会选择扩展`Repository`位于已配置的基本包下的特定于持久性技术的子接口的每个接口，并为其创建一个bean实例。但是，您可能希望对哪些接口为其创建bean实例进行更细粒度的控制。要做到这一点，使用`<include-filter />`和`<exclude-filter />`内部元素`<repositories />`的元素。语义完全等同于Spring的上下文命名空间中的元素。有关详细信息，请参阅这些元素的[Spring参考文档](https://docs.spring.io/spring/docs/5.1.7.RELEASE/spring-framework-reference/core.html#beans-scanning-filters)。

例如，要将某些接口从实例化中排除为存储库bean，可以使用以下配置：

例20 使用exclude-filter元素

```xml
<repositories base-package="com.acme.repositories">
  <context:exclude-filter type="regex" expression=".*SomeRepository" />
</repositories>
```

前面的示例排除了`SomeRepository`从实例化结束的所有接口。

#### 1.5.2 JavaConfig

还可以通过`@Enable${store}Repositories`在JavaConfig类上使用特定于存储的注解来触发存储库基础结构。有关Spring容器的基于Java的配置的介绍，请参阅[Spring参考文档中的JavaConfig](https://docs.spring.io/spring/docs/5.1.7.RELEASE/spring-framework-reference/core.html#beans-java)。

启用Spring Data存储库的示例配置类似于以下内容：

例21 基于样本注释的存储库配置

```java
@Configuration
@EnableJpaRepositories("com.acme.repositories")
class ApplicationConfiguration {

  @Bean
  EntityManagerFactory entityManagerFactory() {
    // …
  }
}
```

> 上面的示例使用JPA特定的注释，您可以根据实际使用的商店模块进行更改。这同样适用于`EntityManagerFactory`bean 的定义。请参阅涵盖特定于商店的配置的部分。

#### 1.5.3 独立使用

您还可以在Spring容器之外使用存储库基础结构 - 例如，在CDI环境中。您仍然需要在类路径中使用一些Spring库，但通常也可以通过编程方式设置存储库。提供存储库支持的Spring Data模块提供了特定`RepositoryFactory`于持久性技术的特性，您可以按如下方式使用：

示例22.存储库工厂的独立使用

```java
RepositoryFactorySupport factory = … // Instantiate factory here
UserRepository repository = factory.getRepository(UserRepository.class);
```

### 1.6 Spring数据存储库的自定义实现

本节介绍存储库自定义以及片段如何构成复合存储库。

当查询方法需要不同的行为或无法通过查询派生实现时，则需要提供自定义实现。Spring Data存储库允许您提供自定义存储库代码，并将其与通用CRUD抽象和查询方法功能集成。

#### 1.6.1 自定义单个存储库

要使用自定义功能丰富存储库，必须首先定义片段接口和自定义功能的实现，如以下示例所示：

例23 自定义存储库功能的接口

```java
interface CustomizedUserRepository {
  void someCustomMethod(User user);
}
```

然后，您可以让您的存储库接口从片段接口进一步扩展，如以下示例所示：

例24 自定义存储库功能的实现

```java
class CustomizedUserRepositoryImpl implements CustomizedUserRepository {

  public void someCustomMethod(User user) {
    // Your custom implementation
  }
}
```

> 对应于片段接口的类名最重要的部分是`Impl`后缀。

实现本身不依赖于Spring Data，可以是常规的Spring bean。因此，您可以使用标准依赖项注入行为将引用注入其他bean（例如一个`JdbcTemplate`），参与方面等。

您可以让存储库接口扩展片段接口，如以下示例所示：

例25 对存储库接口的更改

```java
interface UserRepository extends CrudRepository<User, Long>, CustomizedUserRepository {

  // Declare query methods here
}
```

使用存储库接口扩展片段接口可以组合CRUD和自定义功能，并使其可供客户端使用。

Spring Data存储库通过使用形成存储库组合的片段来实现。片段是基本存储库，功能方面（如[QueryDsl](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#core.extensions.querydsl)），自定义接口及其实现。每次向存储库界面添加接口时，都可以通过添加片段来增强组合。每个Spring Data模块都提供了基本存储库和存储库方面的实现。

以下示例显示了自定义接口及其实现：

例26 具有其实现的片段

```java
interface HumanRepository {
  void someHumanMethod(User user);
}

class HumanRepositoryImpl implements HumanRepository {

  public void someHumanMethod(User user) {
    // Your custom implementation
  }
}

interface ContactRepository {

  void someContactMethod(User user);

  User anotherContactMethod(User user);
}

class ContactRepositoryImpl implements ContactRepository {

  public void someContactMethod(User user) {
    // Your custom implementation
  }

  public User anotherContactMethod(User user) {
    // Your custom implementation
  }
}
```

以下示例显示了扩展的自定义存储库的接口`CrudRepository`：

例27 对存储库界面的更改

```java
interface UserRepository extends CrudRepository<User, Long>, HumanRepository, ContactRepository {

  // Declare query methods here
}
```

存储库可以由多个自定义实现组成，这些实现按其声明的顺序导入。自定义实现的优先级高于基本实现和存储库方面。如果两个片段提供相同的方法签名，则此排序允许您覆盖基本存储库和方面方法并解决歧义。存储库片段不限于在单个存储库接口中使用。多个存储库可以使用片段接口，允许您跨不同的存储库重用自定义。

以下示例显示了存储库片段及其实现：

例28 片段覆盖 `save(…)`

```java
interface CustomizedSave<T> {
  <S extends T> S save(S entity);
}

class CustomizedSaveImpl<T> implements CustomizedSave<T> {

  public <S extends T> S save(S entity) {
    // Your custom implementation
  }
}
```

以下示例显示了使用前面的存储库片段的存储库：

例29 定制的存储库接口

```java
interface UserRepository extends CrudRepository<User, Long>, CustomizedSave<User> {
}

interface PersonRepository extends CrudRepository<Person, Long>, CustomizedSave<Person> {
}
```

**配置**

如果使用命名空间配置，则存储库基础结构会尝试通过扫描其找到存储库的包下面的类来自动检测自定义实现片段。这些类需要遵循将namespace元素的`repository-impl-postfix`属性附加到片段接口名称的命名约定。此后缀默认为`Impl`。以下示例显示了使用默认后缀的存储库以及为后缀设置自定义值的存储库：

示例30.配置示例

```xml
<repositories base-package="com.acme.repository" />

<repositories base-package="com.acme.repository" repository-impl-postfix="MyPostfix" />
```

上一个示例中的第一个配置尝试查找一个名为“ `com.acme.repository.CustomizedUserRepositoryImpl`自定义存储库实现”的类。第二个示例尝试查找`com.acme.repository.CustomizedUserRepositoryMyPostfix`。

**解决歧义**

如果在不同的包中找到具有匹配类名的多个实现，则Spring Data使用bean名称来标识要使用的名称。

给定`CustomizedUserRepository`前面所示的以下两个自定义实现，使用第一个实现。它的bean名称`customizedUserRepositoryImpl`与片段interface（`CustomizedUserRepository`）和postfix的名称相匹配`Impl`。

例31 无差别实现的解决方案

```java
package com.acme.impl.one;

class CustomizedUserRepositoryImpl implements CustomizedUserRepository {

  // Your custom implementation
}
```

```java
package com.acme.impl.two;

@Component("specialCustomImpl")
class CustomizedUserRepositoryImpl implements CustomizedUserRepository {

  // Your custom implementation
}
```

如果使用bean 注释`UserRepository`接口`@Component("specialCustom")`，则bean名称加上`Impl`与为存储库实现定义的名称相匹配`com.acme.impl.two`，并使用它而不是第一个。

**手动连接**

如果您的自定义实现仅使用基于注释的配置和自动装配，则前面显示的方法效果很好，因为它被视为任何其他Spring bean。如果您的实现片段bean需要特殊连接，您可以声明bean并根据[前一节中](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.single-repository-behaviour.ambiguity)描述的约定命名它。然后，基础结构按名称引用手动定义的bean定义，而不是自己创建一个。以下示例显示如何手动连接自定义实现：

例32 自定义实现的手动连接

```xml
<repositories base-package="com.acme.repository" />

<beans:bean id="userRepositoryImpl" class="…">
  <!-- further configuration -->
</beans:bean>
```

#### 1.6.2 自定义Base Repository

当您要自定义基本存储库行为以便所有存储库都受到影响时，上[一节中](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.manual-wiring)描述的方法需要自定义每个存储库接口。要改为更改所有存储库的行为，可以创建一个扩展特定于持久性技术的存储库基类的实现。然后，此类充当存储库代理的自定义基类，如以下示例所示：

例33 自定义存储库基类

```java
class MyRepositoryImpl<T, ID extends Serializable>
  extends SimpleJpaRepository<T, ID> {

  private final EntityManager entityManager;

  MyRepositoryImpl(JpaEntityInformation entityInformation,
                          EntityManager entityManager) {
    super(entityInformation, entityManager);

    // Keep the EntityManager around to used from the newly introduced methods.
    this.entityManager = entityManager;
  }

  @Transactional
  public <S extends T> S save(S entity) {
    // implementation goes here
  }
}
```

> 该类需要具有特定于存储的存储库工厂实现所使用的超类的构造函数。如果存储库基类具有多个构造函数，则覆盖使用`EntityInformation`加号存储特定基础结构对象（例如，`EntityManager`模板类）的构造函数。

最后一步是使Spring Data Infrastructure了解自定义存储库基类。在Java配置中，您可以使用注释的`repositoryBaseClass`属性来执行此操作`@Enable${store}Repositories`，如以下示例所示：

例34 使用JavaConfig配置自定义存储库基类

```java
@Configuration
@EnableJpaRepositories(repositoryBaseClass = MyRepositoryImpl.class)
class ApplicationConfiguration { … }
```

XML命名空间中提供了相应的属性，如以下示例所示：

示例35.使用XML配置自定义存储库基类

```xml
<repositories base-package="com.acme.repository"
     base-class="….MyRepositoryImpl" />
```

### 1.7 从聚合根发布事件

由存储库管理的实体是聚合根。在域驱动设计应用程序中，这些聚合根通常会发布域事件。Spring Data提供了一个注释`@DomainEvents`，可以在聚合根的方法上使用，以使该发布尽可能简单，如以下示例所示：

例36 从聚合根公开事件域事件

```java
class AnAggregateRoot {

    // using方法@DomainEvents可以返回单个事件实例或事件集合。它不能采取任何论点。
    @DomainEvents 
    Collection<Object> domainEvents() {
        // … return events you want to get published here
    }

    // 在所有事件发布后，我们有一个注释的方法@AfterDomainEventPublication。
    // 它可用于潜在地清除要发布的事件列表（以及其他用途）。
    @AfterDomainEventPublication 
    void callbackMethod() {
       // … potentially clean up domain events list
    }
}
```

每次调用Spring Data存储库的`save(…)`方法时，都会调用这些方法。

### 1.8 Spring Data Extensions

本节介绍了一组Spring Data扩展，它们可以在各种上下文中使用Spring Data。目前，大多数集成都针对Spring MVC。

#### 1.8.1 Querydsl扩展

[Querydsl](http://www.querydsl.com/)是一个框架，可以通过其流畅的API构建静态类型的SQL类查询。

多个Spring Data模块提供与Querydsl的集成`QuerydslPredicateExecutor`，如以下示例所示：

例37 QuerydslPredicateExecutor接口

```java
public interface QuerydslPredicateExecutor<T> {

  // 查找并返回与之匹配的单个实体Predicate
  Optional<T> findById(Predicate predicate);  

  // 查找并返回与之匹配的所有实体Predicate
  Iterable<T> findAll(Predicate predicate);   

  // 返回匹配的实体数Predicate
  long count(Predicate predicate);            

  // 返回与Predicateexists 匹配的实体
  boolean exists(Predicate predicate);        

  // … more functionality omitted.
}
```

要使用Querydsl支持，请`QuerydslPredicateExecutor`在存储库界面上进行扩展，如以下示例所示

例38 存储库上的Querydsl集成

```java
interface UserRepository extends CrudRepository<User, Long>, QuerydslPredicateExecutor<User> {
}
```

上面的示例允许您使用Querydsl `Predicate`实例编写类型安全查询，如以下示例所示：

```java
Predicate predicate = user.firstname.equalsIgnoreCase("dave")
	.and(user.lastname.startsWithIgnoreCase("mathews"));

userRepository.findAll(predicate);
```

#### 1.8.2 网络支持

>  本节包含Spring Data Web支持的文档，因为它在Spring Data Commons的当前（及更高版本）版本中实现。由于新引入的支持更改了许多内容，因此我们在[[web.legacy\]中](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#web.legacy)保留了以前行为的文档。

支持存储库编程模型的Spring Data模块具有各种Web支持。Web相关组件要求Spring MVC JAR位于类路径上。其中一些甚至提供与[Spring HATEOAS的](https://github.com/SpringSource/spring-hateoas)集成。通常，通过`@EnableSpringDataWebSupport`在JavaConfig配置类中使用注释来启用集成支持，如以下示例所示：

例39 启用Spring Data Web支持

```java
@Configuration
@EnableWebMvc
@EnableSpringDataWebSupport
class WebConfiguration {}
```

该`@EnableSpringDataWebSupport`批注注册几个组件，我们将在一个位讨论。它还将检测类路径上的Spring HATEOAS，并为它注册集成组件（如果存在）。

或者，如果您使用XML配置，请注册`SpringDataWebConfiguration`或`HateoasAwareSpringDataWebConfiguration`作为Spring bean 注册，如以下示例所示（for `SpringDataWebConfiguration`）：

例40 在XML中启用Spring Data Web支持

```xml
<bean class="org.springframework.data.web.config.SpringDataWebConfiguration" />

<!-- If you use Spring HATEOAS, register this one *instead* of the former -->
<bean class="org.springframework.data.web.config.HateoasAwareSpringDataWebConfiguration" />
```

**基本Web支持**

[上一节中](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#core.web)显示的配置注册了一些基本组件：

- 一个[`DomainClassConverter`](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#core.web.basic.domain-class-converter)让Spring MVC的解决仓库管理域类的实例从请求参数或路径变量。
- [`HandlerMethodArgumentResolver`](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#core.web.basic.paging-and-sorting)实现让Spring MVC 从请求参数中解析`Pageable`和`Sort`实例。

**DomainClassConverter**

将`DomainClassConverter`让你在Spring MVC中的控制器方法签名使用域类型直接，让你不用通过储存手动查找的情况下，如在下面的例子：

例41 在方法签名中使用域类型的Spring MVC控制器

```java
@Controller
@RequestMapping("/users")
class UserController {

  @RequestMapping("/{id}")
  String showUserForm(@PathVariable("id") User user, Model model) {

    model.addAttribute("user", user);
    return "userForm";
  }
}
```

如您所见，该方法`User`直接接收实例，无需进一步查找。可以通过让Spring MVC首先将路径变量转换为`id`域类的类型来解析实例，并最终通过调用`findById(…)`为域类型注册的存储库实例来访问实例。

> 目前，存储库必须实现`CrudRepository`才有资格被发现进行转换。

**HandlerMethodArgumentResolvers for Pageable和Sort**

[上一节中](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#core.web.basic.domain-class-converter)显示的配置代码[段](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#core.web.basic.domain-class-converter)还注册了一个`PageableHandlerMethodArgumentResolver`以及一个实例`SortHandlerMethodArgumentResolver`。注册启用`Pageable`和`Sort`作为有效的控制器方法参数，如以下示例所示：

示例42.使用Pageable作为控制器方法参数

```java
@Controller
@RequestMapping("/users")
class UserController {

  private final UserRepository repository;

  UserController(UserRepository repository) {
    this.repository = repository;
  }

  @RequestMapping
  String showUsers(Model model, Pageable pageable) {

    model.addAttribute("users", repository.findAll(pageable));
    return "users";
  }
}
```

上述方法签名导致Spring MVC尝试`Pageable`使用以下默认配置从请求参数派生实例：
| 属性   | 作用                                                         |
| ------ | ------------------------------------------------------------ |
| `page` | 您要检索的页面。0索引，默认为0。                             |
| `size` | 要检索的页面大小。默认为20。                                 |
| `sort` | 应按格式排序的属性`property,property(,ASC|DESC)`。默认排序方向是升序。`sort`如果要切换方向，请使用多个参数 - 例如，`?sort=firstname&sort=lastname,asc`。 |

要自定义此行为，请分别注册实现`PageableHandlerMethodArgumentResolverCustomizer`接口或`SortHandlerMethodArgumentResolverCustomizer`接口的Bean 。`customize()`调用其方法，让您更改设置，如以下示例所示：

```java
@Bean SortHandlerMethodArgumentResolverCustomizer sortCustomizer() {
    return s -> s.setPropertyDelimiter("<-->");
}
```

如果设置现有属性`MethodArgumentResolver`不足以满足您的需要，请扩展其中一个`SpringDataWebConfiguration`或启用HATEOAS的等效项，重写`pageableResolver()`或`sortResolver()`方法，然后导入自定义配置文件而不是使用`@Enable`注释。

如果您需要从请求中解析多个`Pageable`或`Sort`实例（例如，对于多个表），则可以使用Spring的`@Qualifier`注释来区分彼此。然后请求参数必须以前缀为前缀`${qualifier}_`。以下示例显示了生成的方法签名：

```java
String showUsers(Model model,
      @Qualifier("thing1") Pageable first,
      @Qualifier("thing2") Pageable second) { … }
```

你有填充`thing1_page`和`thing2_page`等。

`Pageable`传递给方法的默认值相当于a，`PageRequest.of(0, 20)`但可以使用参数`@PageableDefault`上的注释进行自定义`Pageable`。

**对Pageables的超媒体支持**

Spring HATEOAS附带了一个表示模型类（`PagedResources`），它允许`Page`使用必要的`Page`元数据丰富实例的内容，以及让客户端轻松浏览页面的链接。将Page转换为a `PagedResources`是由Spring HATEOAS `ResourceAssembler`接口的实现完成的，称为`PagedResourcesAssembler`。以下示例显示如何使用a `PagedResourcesAssembler`作为控制器方法参数：

例43 使用PagedResourcesAssembler作为控制器方法参数

```java
@Controller
class PersonController {

  @Autowired PersonRepository repository;

  @RequestMapping(value = "/persons", method = RequestMethod.GET)
  HttpEntity<PagedResources<Person>> persons(Pageable pageable,
    PagedResourcesAssembler assembler) {

    Page<Person> persons = repository.findAll(pageable);
    return new ResponseEntity<>(assembler.toResources(persons), HttpStatus.OK);
  }
}
```

如上例所示启用配置可以将`PagedResourcesAssembler`其用作控制器方法参数。调用`toResources(…)`它具有以下效果：

- 内容`Page`成为`PagedResources`实例的内容。
- 该`PagedResources`对象获得一个`PageMetadata`附加实例，并使用来自`Page`和底层的信息填充`PageRequest`。
- 将`PagedResources`可能会`prev`和`next`连接链路，根据页面的状态。链接指向方法映射到的URI。添加到方法的分页参数与设置相匹配，`PageableHandlerMethodArgumentResolver`以确保稍后可以解析链接。

假设我们在数据库中有30个Person实例。您现在可以触发request（）并查看类似于以下内容的输出：`GEThttp://localhost:8080/persons`

```javascript
{ "links" : [ { "rel" : "next",
                "href" : "http://localhost:8080/persons?page=1&size=20 }
  ],
  "content" : [
     … // 20 Person instances rendered here
  ],
  "pageMetadata" : {
    "size" : 20,
    "totalElements" : 30,
    "totalPages" : 2,
    "number" : 0
  }
}
```

您会看到汇编程序生成了正确的URI，并且还选择了默认配置以将参数解析`Pageable`为即将发出的请求。这意味着，如果更改该配置，链接将自动遵循更改。默认情况下，汇编程序指向它所调用的控制器方法，但是可以通过交换自定义来自定义`Link`以构建分页链接，从而使`PagedResourcesAssembler.toResource(…)`方法重载。

**Web数据绑定支持**

Spring Data投影（在[[projection\]中](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#projections)描述）可以通过使用[JSONPath](http://goessner.net/articles/JsonPath/)表达式来绑定传入的请求有效负载（需要[Jayway JsonPath](https://github.com/json-path/JsonPath)或[XPath](https://www.w3.org/TR/xpath-31/)表达式（需要[XmlBeam](https://xmlbeam.org/)）），如以下示例所示：

示例44.使用JSONPath或XPath表达式的HTTP有效负载绑定

```java
@ProjectedPayload
public interface UserPayload {

  @XBRead("//firstname")
  @JsonPath("$..firstname")
  String getFirstname();

  @XBRead("/lastname")
  @JsonPath({ "$.lastname", "$.user.lastname" })
  String getLastname();
}
```

前面示例中显示的类型可以用作Spring MVC处理程序方法参数，也可以使用`ParameterizedTypeReference`其中一个`RestTemplate`方法。前面的方法声明将尝试查找`firstname`给定文档中的任何位置。该`lastname`XML查询是对输入文档的顶层进行。JSON变体`lastname`首先尝试顶级，但如果前者没有返回值，也会尝试`lastname`嵌套在`user`子文档中。这样，可以轻松地减轻源文档结构的变化，而无需客户端调用公开的方法（通常是基于类的有效负载绑定的缺点）。

如[[投影\]中](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#projections)所述，支持嵌套投影。如果该方法返回复杂的非接口类型，`ObjectMapper`则使用Jackson 来映射最终值。

对于Spring MVC，必要的转换器会`@EnableSpringDataWebSupport`在活动时自动注册，并且类路径上可以使用所需的依赖项。使用时`RestTemplate`，注册`ProjectingJackson2HttpMessageConverter`（JSON）或`XmlBeamHttpMessageConverter`手动。

有关更多信息，请参阅规范[Spring Data Examples存储库中](https://github.com/spring-projects/spring-data-examples)的[Web投影示例](https://github.com/spring-projects/spring-data-examples/tree/master/web/projection)。

**Querydsl Web支持**

对于那些具有[QueryDSL](http://www.querydsl.com/)集成的商店，可以从`Request`查询字符串中包含的属性派生查询。

请考虑以下查询字符串：

```
?firstname=Dave&lastname=Matthews
```

给定`User`前面示例中的对象，可以使用以下方法将查询字符串解析为以下值`QuerydslPredicateArgumentResolver`。

```java
QUser.user.firstname.eq("Dave").and(QUser.user.lastname.eq("Matthews"))
```

> `@EnableSpringDataWebSupport`当在类路径中找到Querydsl时 ，该功能会自动启用。

添加`@QuerydslPredicate`到方法签名提供了一个即用型`Predicate`，可以使用该方法运行`QuerydslPredicateExecutor`。

> 通常从方法的返回类型中解析类型信息。由于该信息不一定与域类型匹配，因此使用该`root`属性可能是个好主意`QuerydslPredicate`。

以下示例显示了如何`@QuerydslPredicate`在方法签名中使用：

```java
@Controller
class UserController {

  @Autowired UserRepository repository;

  // 将查询字符串参数解析为匹配Predicatefor User
  @RequestMapping(value = "/", method = RequestMethod.GET)
  String index(Model model, @QuerydslPredicate(root = User.class) Predicate predicate,    
          Pageable pageable, @RequestParam MultiValueMap<String, String> parameters) {

    model.addAttribute("users", repository.findAll(predicate, pageable));

    return "index";
  }
}
```

默认绑定如下：

- `Object`在简单的属性上`eq`。
- `Object`像集合一样的属性`contains`。
- `Collection`在简单的属性上`in`。

可以通过Java 8 的`bindings`属性`@QuerydslPredicate`或通过使用Java 8 `default methods`并将该`QuerydslBinderCustomizer`方法添加到存储库接口来定制这些绑定。

```java
// QuerydslPredicateExecutor提供对特定查找程序方法的访问权限Predicate
// QuerydslBinderCustomizer在存储库界面上定义的是自动拾取和快捷方式@QuerydslPredicate(bindings=…)
interface UserRepository extends CrudRepository<User, String>,
                                 QuerydslPredicateExecutor<User>,                
                                 QuerydslBinderCustomizer<QUser> {               

  @Override
  default void customize(QuerydslBindings bindings, QUser user) {
    // 将username属性的绑定定义为简单contains绑定。
    bindings.bind(user.username).first((path, value) -> path.contains(value)); 
    // 将String属性的默认绑定定义为不区分大小写的contains匹配
    bindings.bind(String.class)
      .first((StringPath path, String value) -> path.containsIgnoreCase(value));
    // password从Predicate解决方案中排除该属性
    bindings.excluding(user.password);                                           
  }
}
```

#### 1.8.3 存储库填充程序

如果您使用Spring JDBC模块，您可能熟悉`DataSource`使用SQL脚本填充的支持。虽然它不使用SQL作为数据定义语言，但它在存储库级别上可以使用类似的抽象，因为它必须与存储无关。因此，填充程序支持XML（通过Spring的OXM抽象）和JSON（通过Jackson）来定义用于填充存储库的数据。

假设您有一个`data.json`包含以下内容的文件：

例45 在JSON中定义的数据

```json
[ { "_class" : "com.acme.Person",
 "firstname" : "Dave",
  "lastname" : "Matthews" },
  { "_class" : "com.acme.Person",
 "firstname" : "Carter",
  "lastname" : "Beauford" } ]
```

您可以使用Spring Data Commons中提供的存储库命名空间的populator元素来填充存储库。要将前面的数据填充到PersonRepository，请声明类似于以下内容的populator：

示例46.声明Jackson存储库填充程序

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:repository="http://www.springframework.org/schema/data/repository"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/data/repository
    http://www.springframework.org/schema/data/repository/spring-repository.xsd">

  <repository:jackson2-populator locations="classpath:data.json" />

</beans>
```

前面的声明导致`data.json`文件被Jackson读取和反序列化`ObjectMapper`。

通过检查`_class`JSON文档的属性来确定解组JSON对象的类型。基础结构最终选择适当的存储库来处理反序列化的对象。

要使用XML来定义应该填充存储库的数据，您可以使用该`unmarshaller-populator`元素。您将其配置为使用Spring OXM中提供的XML marshaller选项之一。有关详细信息，请参阅[Spring参考文档](https://docs.spring.io/spring/docs/5.1.7.RELEASE/spring-framework-reference/data-access.html#oxm)。以下示例说明如何使用JAXB解组存储库填充程序：

例47 声明一个解组存储库populator（使用JAXB）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:repository="http://www.springframework.org/schema/data/repository"
  xmlns:oxm="http://www.springframework.org/schema/oxm"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/data/repository
    http://www.springframework.org/schema/data/repository/spring-repository.xsd
    http://www.springframework.org/schema/oxm
    http://www.springframework.org/schema/oxm/spring-oxm.xsd">

  <repository:unmarshaller-populator locations="classpath:data.json"
    unmarshaller-ref="unmarshaller" />

  <oxm:jaxb2-marshaller contextPath="com.acme" />

</beans>
```

# 参考文档

## 2.Elasticsearch存储库

本章包含Elasticsearch存储库实现的详细信息。

### 2.1 介绍

#### 2.1.1 Spring命名空间

Spring Data Elasticsearch模块包含一个自定义命名空间，允许定义存储库bean以及用于实例化a的元素`ElasticsearchServer`。

使用该`repositories`元素查找Spring Data存储库，如[创建存储库实例中所述](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.create-instances)。

示例48.使用命名空间设置Elasticsearch存储库

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:elasticsearch="http://www.springframework.org/schema/data/elasticsearch"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
http://www.springframework.org/schema/data/elasticsearch
http://www.springframework.org/schema/data/elasticsearch/spring-elasticsearch-1.0.xsd">

    <elasticsearch:repositories base-package="com.acme.repositories" />

</beans>
```

使用`Transport Client`or `Node Client`元素`Elasticsearch Server`在上下文中注册实例。

示例49.使用命名空间的传输客户端

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:elasticsearch="http://www.springframework.org/schema/data/elasticsearch"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
http://www.springframework.org/schema/data/elasticsearch
http://www.springframework.org/schema/data/elasticsearch/spring-elasticsearch-1.0.xsd">

    <elasticsearch:transport-client id="client" cluster-nodes="localhost:9300,someip:9300" />

</beans>
```

示例50.使用命名空间的节点客户端

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:elasticsearch="http://www.springframework.org/schema/data/elasticsearch"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
http://www.springframework.org/schema/data/elasticsearch
http://www.springframework.org/schema/data/elasticsearch/spring-elasticsearch-1.0.xsd">

    <elasticsearch:node-client id="client" local="true"" />

</beans>
```

#### 2.1.2 基于注释的配置

Spring Data Elasticsearch存储库支持不仅可以通过XML命名空间激活，还可以通过JavaConfig使用注释。

例51.使用JavaConfig的Spring Data Elasticsearch存储库

```java
@Configuration
@EnableElasticsearchRepositories(basePackages = "org/springframework/data/elasticsearch/repositories")
static class Config {

    @Bean
    public ElasticsearchOperations elasticsearchTemplate() {
        return new ElasticsearchTemplate(nodeBuilder().local(true).node().client());
    }
}
```

上面的配置设置了一个`Embedded Elasticsearch Server`使用的`ElasticsearchTemplate`。Spring Data Elasticsearch存储库使用`@EnableElasticsearchRepositories`注释激活，注释基本上具有与XML命名空间相同的属性。如果未配置基本软件包，则将使用配置类所在的基础软件包。

#### 2.1.3。Elasticsearch使用CDI重新存储

还可以使用CDI功能设置Spring Data Elasticsearch存储库。

示例52.使用JavaConfig的Spring Data Elasticsearch存储库

```java
class ElasticsearchTemplateProducer {

    @Produces
    @ApplicationScoped
    public ElasticsearchOperations createElasticsearchTemplate() {
        return new ElasticsearchTemplate(nodeBuilder().local(true).node().client());
    }
}

class ProductService {

    private ProductRepository repository;

    public Page<Product> findAvailableBookByName(String name, Pageable pageable) {
        return repository.findByAvailableTrueAndNameStartingWith(name, pageable);
    }

    @Inject
    public void setRepository(ProductRepository repository) {
        this.repository = repository;
    }
}
```

### 2.2 查询方法

#### 2.2.1 查询查找策略

Elasticsearch模块支持所有基本查询构建功能，如String，Abstract，Criteria或从方法名称派生。

##### 声明的查询

从方法名称派生查询并不总是足够和/或可能导致不可读的方法名称。在这种情况下，可以使用`@Query`注释（请参阅[使用@Query注释](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#elasticsearch.query-methods.at-query)）。

#### 2.2.2 查询创建

通常，Elasticsearch的查询创建机制的工作方式如[Query方法中所述](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.query-methods)。以下是Elasticsearch查询方法转换为的简短示例：

示例53.从方法名称创建查询

```java
public interface BookRepository extends Repository<Book, String>
{
    List<Book> findByNameAndPrice(String name, Integer price);
}
```

上面的方法名称将被转换为以下Elasticsearch json查询

```json
{ "bool" :
    { "must" :
        [
            { "field" : {"name" : "?"} },
            { "field" : {"price" : "?"} }
        ]
    }
}
```

Elasticsearch支持的关键字列表如下所示。

| 关键词                | 样品                                       | Elasticsearch查询字符串                                      |
| :-------------------- | :----------------------------------------- | :----------------------------------------------------------- |
| `And`                 | `findByNameAndPrice`                       | `{"bool" : {"must" : [ {"field" : {"name" : "?"}}, {"field" : {"price" : "?"}} ]}}` |
| `Or`                  | `findByNameOrPrice`                        | `{"bool" : {"should" : [ {"field" : {"name" : "?"}}, {"field" : {"price" : "?"}} ]}}` |
| `Is`                  | `findByName`                               | `{"bool" : {"must" : {"field" : {"name" : "?"}}}}`           |
| `Not`                 | `findByNameNot`                            | `{"bool" : {"must_not" : {"field" : {"name" : "?"}}}}`       |
| `Between`             | `findByPriceBetween`                       | `{"bool" : {"must" : {"range" : {"price" : {"from" : ?,"to" : ?,"include_lower" : true,"include_upper" : true}}}}}` |
| `LessThanEqual`       | `findByPriceLessThan`                      | `{"bool" : {"must" : {"range" : {"price" : {"from" : null,"to" : ?,"include_lower" : true,"include_upper" : true}}}}}` |
| `GreaterThanEqual`    | `findByPriceGreaterThan`                   | `{"bool" : {"must" : {"range" : {"price" : {"from" : ?,"to" : null,"include_lower" : true,"include_upper" : true}}}}}` |
| `Before`              | `findByPriceBefore`                        | `{"bool" : {"must" : {"range" : {"price" : {"from" : null,"to" : ?,"include_lower" : true,"include_upper" : true}}}}}` |
| `After`               | `findByPriceAfter`                         | `{"bool" : {"must" : {"range" : {"price" : {"from" : ?,"to" : null,"include_lower" : true,"include_upper" : true}}}}}` |
| `Like`                | `findByNameLike`                           | `{"bool" : {"must" : {"field" : {"name" : {"query" : "?*","analyze_wildcard" : true}}}}}` |
| `StartingWith`        | `findByNameStartingWith`                   | `{"bool" : {"must" : {"field" : {"name" : {"query" : "?*","analyze_wildcard" : true}}}}}` |
| `EndingWith`          | `findByNameEndingWith`                     | `{"bool" : {"must" : {"field" : {"name" : {"query" : "*?","analyze_wildcard" : true}}}}}` |
| `Contains/Containing` | `findByNameContaining`                     | `{"bool" : {"must" : {"field" : {"name" : {"query" : "**?**","analyze_wildcard" : true}}}}}` |
| `In`                  | `findByNameIn(Collection<String>names)`    | `{"bool" : {"must" : {"bool" : {"should" : [ {"field" : {"name" : "?"}}, {"field" : {"name" : "?"}} ]}}}}` |
| `NotIn`               | `findByNameNotIn(Collection<String>names)` | `{"bool" : {"must_not" : {"bool" : {"should" : {"field" : {"name" : "?"}}}}}}` |
| `Near`                | `findByStoreNear`                          | `Not Supported Yet !`                                        |
| `True`                | `findByAvailableTrue`                      | `{"bool" : {"must" : {"field" : {"available" : true}}}}`     |
| `False`               | `findByAvailableFalse`                     | `{"bool" : {"must" : {"field" : {"available" : false}}}}`    |
| `OrderBy`             | `findByAvailableTrueOrderByNameDesc`       | `{"sort" : [{ "name" : {"order" : "desc"} }],"bool" : {"must" : {"field" : {"available" : true}}}}` |

#### 2.2.3。使用@Query Annotation

例54.使用`@Query`注释在方法中声明查询。

```java
public interface BookRepository extends ElasticsearchRepository<Book, String> {
    @Query("{"bool" : {"must" : {"field" : {"name" : "?0"}}}}")
    Page<Book> findByName(String name,Pageable pageable);
}
```

## 3.其他Elasticsearch操作支持

本章介绍了对无法通过存储库接口直接访问的Elasticsearch操作的其他支持。建议将这些操作添加为自定义实现，如[Spring Data Repositories的](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.custom-implementations)自定义实现中所述。

### 3.1 过滤生成器

Filter Builder提高了查询速度。

```java
private ElasticsearchTemplate elasticsearchTemplate;

SearchQuery searchQuery = new NativeSearchQueryBuilder()
    .withQuery(matchAllQuery())
    .withFilter(boolFilter().must(termFilter("id", documentId)))
    .build();

Page<SampleEntity> sampleEntities =
    elasticsearchTemplate.queryForPage(searchQuery,SampleEntity.class);
```

### 3.2 使用滚动进行大结果集

Elasticsearch有一个滚动API，用于在块中获取大结果集。`ElasticsearchTemplate`有startScroll和continueScroll方法，可以使用如下。

例55.使用startScroll和continueScroll

```java
SearchQuery searchQuery = new NativeSearchQueryBuilder()
    .withQuery(matchAllQuery())
    .withIndices(INDEX_NAME)
    .withTypes(TYPE_NAME)
    .withFields("message")
    .withPageable(PageRequest.of(0, 10))
    .build();

Page<SampleEntity> scroll = elasticsearchTemplate.startScroll(1000, searchQuery, SampleEntity.class);

String scrollId = ((ScrolledPage) scroll).getScrollId();
List<SampleEntity> sampleEntities = new ArrayList<>();
while (scroll.hasContent()) {
    sampleEntities.addAll(scroll.getContent());
    scrollId = ((ScrolledPage) scroll).getScrollId();
    scroll = elasticsearchTemplate.continueScroll(scrollId, 1000, SampleEntity.class);
}
elasticsearchTemplate.clearScroll(scrollId);
```

`ElasticsearchTemplate` 另外还有stream方法，它将扫描和滚动操作包装到CloseableIterator中。

例56.使用流

```java
SearchQuery searchQuery = new NativeSearchQueryBuilder()
    .withQuery(matchAllQuery())
    .withIndices(INDEX_NAME)
    .withTypes(TYPE_NAME)
    .withFields("message")
    .withPageable(PageRequest.of(0, 10))
    .build();

CloseableIterator<SampleEntity> stream = elasticsearchTemplate.stream(searchQuery, SampleEntity.class);

List<SampleEntity> sampleEntities = new ArrayList<>();
while (stream.hasNext()) {
    sampleEntities.add(stream.next());
}
```

# 附录

## 附录A：命名空间参考

### `<repositories />`元素

该`<repositories />`元素触发Spring Data存储库基础结构的设置。最重要的属性是`base-package`，它定义了扫描Spring Data存储库接口的包。请参阅“ [XML配置](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.create-instances.spring) ”。下表描述了`<repositories />`元素的属性：

| 名称                           | 描述                                                         |
| :----------------------------- | :----------------------------------------------------------- |
| `base-package`                 | 定义要扫描的包，以便`*Repository`在自动检测模式下扩展（实际接口由特定Spring Data模块确定）的存储库接口。也会扫描配置包下面的所有包。允许使用通配符。 |
| `repository-impl-postfix`      | 定义后缀以自动检测自定义存储库实现。名称以配置的后缀结尾的类被视为候选。默认为`Impl`。 |
| `query-lookup-strategy`        | 确定用于创建查找程序查询的策略。有关详细信息，请参阅“ [查询查找策略](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#repositories.query-methods.query-lookup-strategies) ”。默认为`create-if-not-found`。 |
| `named-queries-location`       | 定义搜索包含外部定义查询的Properties文件的位置。             |
| `consider-nested-repositories` | 是否应考虑嵌套存储库接口定义。默认为`false`。                |

## 附录B：Populators命名空间参考

### `<populator />`元素

该`<populator />`元素允许通过Spring Data存储库基础结构填充数据存储。[ [1](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_1) ]

| 名称        | 描述                                       |
| :---------- | :----------------------------------------- |
| `locations` | 从哪里可以找到要从存储库中读取对象的文件。 |

## 附录C：存储库查询关键字

### 支持的查询关键字

下表列出了Spring Data存储库查询派生机制通常支持的关键字。但是，请查阅特定于商店的文档以获取支持的关键字的确切列表，因为此处列出的某些关键字可能在特定商店中不受支持。

| 逻辑关键字            | 关键字表达式                                   |
| :-------------------- | :--------------------------------------------- |
| `AND`                 | `And`                                          |
| `OR`                  | `Or`                                           |
| `AFTER`               | `After`， `IsAfter`                            |
| `BEFORE`              | `Before`， `IsBefore`                          |
| `CONTAINING`          | `Containing`，`IsContaining`，`Contains`       |
| `BETWEEN`             | `Between`， `IsBetween`                        |
| `ENDING_WITH`         | `EndingWith`，`IsEndingWith`，`EndsWith`       |
| `EXISTS`              | `Exists`                                       |
| `FALSE`               | `False`， `IsFalse`                            |
| `GREATER_THAN`        | `GreaterThan`， `IsGreaterThan`                |
| `GREATER_THAN_EQUALS` | `GreaterThanEqual`， `IsGreaterThanEqual`      |
| `IN`                  | `In`， `IsIn`                                  |
| `IS`                  | `Is`，`Equals`，（或无关键字）                 |
| `IS_EMPTY`            | `IsEmpty`， `Empty`                            |
| `IS_NOT_EMPTY`        | `IsNotEmpty`， `NotEmpty`                      |
| `IS_NOT_NULL`         | `NotNull`， `IsNotNull`                        |
| `IS_NULL`             | `Null`， `IsNull`                              |
| `LESS_THAN`           | `LessThan`， `IsLessThan`                      |
| `LESS_THAN_EQUAL`     | `LessThanEqual`， `IsLessThanEqual`            |
| `LIKE`                | `Like`， `IsLike`                              |
| `NEAR`                | `Near`， `IsNear`                              |
| `NOT`                 | `Not`， `IsNot`                                |
| `NOT_IN`              | `NotIn`， `IsNotIn`                            |
| `NOT_LIKE`            | `NotLike`， `IsNotLike`                        |
| `REGEX`               | `Regex`，`MatchesRegex`，`Matches`             |
| `STARTING_WITH`       | `StartingWith`，`IsStartingWith`，`StartsWith` |
| `TRUE`                | `True`， `IsTrue`                              |
| `WITHIN`              | `Within`， `IsWithin`                          |

## 附录D：存储库查询返回类型

### 支持的查询返回类型

下表列出了Spring Data存储库通常支持的返回类型。但是，请查阅特定于商店的文档以获取支持的返回类型的确切列表，因为此处列出的某些类型可能在特定商店中不受支持。

> 地理空间类型（如`GeoResult`，`GeoResults`，和`GeoPage`）是仅对支持地理空间查询数据存储可用。

| 返回类型               | 描述                                                         |
| :--------------------- | :----------------------------------------------------------- |
| `void`                 | 表示没有返回值。                                             |
| 基元                   | Java原语。                                                   |
| 包装类型               | Java包装器类型。                                             |
| `T`                    | 一个独特的实体。期望查询方法最多返回一个结果。如果未找到结果，`null`则返回。不止一个结果会触发一个`IncorrectResultSizeDataAccessException`。 |
| `Iterator<T>`          | 安`Iterator`。                                               |
| `Collection<T>`        | 一`Collection`。                                             |
| `List<T>`              | 一`List`。                                                   |
| `Optional<T>`          | Java 8或Guava `Optional`。期望查询方法最多返回一个结果。如果未找到结果，`Optional.empty()`或`Optional.absent()`返回结果。不止一个结果会触发一个`IncorrectResultSizeDataAccessException`。 |
| `Option<T>`            | Scala或Javaslang `Option`类型。语义上与`Optional`前面描述的Java 8相同。 |
| `Stream<T>`            | 一个Java 8 `Stream`。                                        |
| `Future<T>`            | 一`Future`。期望一个要注释的方法，`@Async`并要求启用Spring的异步方法执行功能。 |
| `CompletableFuture<T>` | 一个Java 8 `CompletableFuture`。期望一个要注释的方法，`@Async`并要求启用Spring的异步方法执行功能。 |
| `ListenableFuture`     | 一`org.springframework.util.concurrent.ListenableFuture`。期望一个要注释的方法，`@Async`并要求启用Spring的异步方法执行功能。 |
| `Slice`                | 一大块数据，指示是否有更多可用数据。需要`Pageable`方法参数。 |
| `Page<T>`              | A `Slice`附加信息，例如结果总数。需要`Pageable`方法参数。    |
| `GeoResult<T>`         | 带有附加信息的结果条目，例如到参考位置的距离。               |
| `GeoResults<T>`        | `GeoResult<T>`包含附加信息的列表，例如到参考位置的平均距离。 |
| `GeoPage<T>`           | 甲`Page`带`GeoResult<T>`，如到参考位置的平均距离。           |
| `Mono<T>`              | `Mono`使用反应性存储库发射零个或一个元素的Project Reactor 。期望查询方法最多返回一个结果。如果未找到结果，`Mono.empty()`则返回。不止一个结果会触发一个`IncorrectResultSizeDataAccessException`。 |
| `Flux<T>`              | `Flux`使用反应性存储库发出零个，一个或多个元素的Project Reactor 。返回的查询`Flux`也可以发出无限数量的元素。 |
| `Single<T>`            | `Single`使用反应式存储库发出单个元素的RxJava 。期望查询方法最多返回一个结果。如果未找到结果，`Mono.empty()`则返回。不止一个结果会触发一个`IncorrectResultSizeDataAccessException`。 |
| `Maybe<T>`             | `Maybe`使用反应式存储库发出零个或一个元素的RxJava 。期望查询方法最多返回一个结果。如果未找到结果，`Mono.empty()`则返回。不止一个结果会触发一个`IncorrectResultSizeDataAccessException`。 |
| `Flowable<T>`          | RxJava `Flowable`使用反应式存储库发出零个，一个或多个元素。返回的查询`Flowable`也可以发出无限数量的元素。 |