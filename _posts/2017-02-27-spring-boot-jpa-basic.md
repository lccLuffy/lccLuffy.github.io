---
layout: post
permalink: /spring-boot-jpa-basic/
title: Spring Boot JPA - 基本使用
date: 2017-02-27T23:02:00
lang: zh-CN
---

## 什么是 JPA ？

JPA （The Java Persistence API）是用于访问，持久化和管理 Java 对象/类与关系型数据库之间的数据交互的 Java 规范。JPA 被定义为EJB （Enterprise JavaBeans） 3.0规范的一部分，作为 EJB 2 CMP 实体 Bean 规范的替代。

注意，JPA 只是一个标准，只定义了一系列接口，而没有具体的实现。很多企业级框架提供了对 JPA 的实现，如 Spring 。因此 Spring 本身与 JPA 无关，只是提供了对 JPA 的支持，因此在 Spring 中你也会看到很多注解都是属于 `javax.persistence` 包的。

JPA 允许 [POJO](https://en.wikipedia.org/wiki/POJO)（Plain Old Java Objects）轻松地持久化，而不需要类来实现 EJB 2 CM P规范所需的任何接口或方法。 JPA 还允许通过**注解**或 XML 定义对象的关系映射，定义 Java 类如何映射到关系数据库表。 JPA 还定义了一个运行时 `EntityManager`  API，用于处理对象的查询和管理事务。 同时，JPA 定义了对象级查询语言 **JPQL**，以允许从数据库中查询对象，实现了对数据库的解耦合，提高了程序的可移植性，而不具体依赖某一底层数据库。

JPA 是 Java 持久化规范中的一个最新版本。第一个版本是 OMG 持久性服务 Java 绑定，但这个一个失败的产品，甚至没有任何商业产品支持它。接下来的版本是 EJB 1.0 CMP Entity Beans，它已经非常成功地被大型 Java EE 提供程序（BEA，IBM）采用，但是它复杂性太高而且性能比较差。EJB 2.0 CMP 试图通过引入本地接口来减少 Entity Bean 的一些复杂性，但是大多数复杂性仍然存在,而且缺乏可移植性。

历史总是要向前发展的，种种的这些使得 EJB 3.0 规范将降低复杂性作为主要目标，这导致规范委员会沿着 JPA 的路径前进。 JPA 旨在统一 EJB 2 CMP，JDO，Hibernate，从目前来看，JPA 的确取得了成功。

目前大多数持久化供应商已经发布了 JPA 的实现，并被行业和用户采用。这些包括 Hibernate（由 JBoss 和 Red Hat 收购），TopLink（由 Oracle 收购）和 Kodo JDO（由 BEA 和 Oracle 收购）。其他支持 JPA 的产品包括 Cocobase（由 Thought Inc. 收购）和 JPOX。

## 定义实体类

#### 依赖

在 Spring Boot 中引入 Spring Data JPA 很简单，在 `pom.xml` 中加入依赖即可：
```
<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

#### Model
如果具体的 SQL 语句来创建数据库表，就会和具体的底层数据库产生耦合。在 Springl Boot 中，我们可以定义一系列 Entity，来实现 Java 对象到 数据表的映射。

加入我们的数据表有一些公用的属性，我们可以定义一个超类，来声明这些公用属性。`@MappedSuperclass` 注解表明实体是一个超类，保证不会被 Spring 实例化 （保险起见加上`abstract`）：
```
@MappedSuperclass
public abstract class BaseModel implements Serializable {
	/**
	 *
	 */
	private static final long serialVersionUID = 1782474744437162148L;
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	@Column(name = "ID")
	private Long id;

	public Long getId() {
		return id;
	}

	public void setId(Long id) {
		this.id = id;
	}

	@Override
	public boolean equals(Object o) {
		if (this == o) return true;
		if (o == null || getClass() != o.getClass()) return false;

		BaseModel baseModel = (BaseModel) o;

		return id != null ? id.equals(baseModel.id) : baseModel.id == null;
	}

	@Override
	public int hashCode() {
		return id != null ? id.hashCode() : 0;
	}
}
```
`@Id` 表明实体主键为整数类型，`@GeneratedValue` 表明主键值生成策略，`strategy` 可选值为 `TABLE`， `SEQUENCE`， `IDENTITY`，`AUTO` 。如果我们想对列加上更多的限制，可以使用 `@Column` 注解，`@Column` 可以制定列的属性有：`name`，列名；`unique` ,是否为一；`nullable` ，是否为空；`length` ,长度；`columnDefinition`，列定义，如 `TEXT` 类型；等等。

注意，列名和表名到数据库的映射策略，定义在 `org.hibernate.boot.model.naming` 包下，你也可以定义自己的映射策略，然后修改 application.yml  `spring.jpa.hibernate.naming.physical-strategy` 的属性即可。

#### 关系
以一个简单博客系统来说明吧，包括用户，文章，分类，标签四个表。用户与文章是一对多，一个用户可以有多篇文章；分类与文章也是一对多，文章与标签是多对多：
`User` 表：
```
@Entity
@Table
public class User extends BaseModel {
    private String name;
    private String email;
    private String address;

    @OneToMany(mappedBy = "user")
    private List<Post> posts;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public List<Post> getPosts() {
        return posts;
    }

    public void setPosts(List<Post> posts) {
        this.posts = posts;
    }
}
``` 
`@OneToMany` 声明了 `User` 和 `Post` 的一对多关系，`mappedBy ` 指明了 `Post` 类中维持该关系的字段名称是 `user` ，即 `Post` 表中的 `user` 字段：
`Post` 表：
```
@Entity
@Table
public class Post extends BaseModel {
	private String title;

	@Column(columnDefinition = "TEXT")
	private String content;

	@ManyToOne
	@JoinColumn(name = "category_id")
	private Category category;

	@ManyToMany
	@JoinTable(name = "post_tag", joinColumns = @JoinColumn(name = "post_id"), inverseJoinColumns = @JoinColumn(name = "tag_id"))
	private Set<Tag> tags;

	private Date createdAt;

	@ManyToOne
	@JoinColumn(name = "user_id")
	private User user;

	public String getTitle() {
		return title;
	}

	public void setTitle(String title) {
		this.title = title;
	}

	public String getContent() {
		return content;
	}

	public void setContent(String content) {
		this.content = content;
	}

	public Category getCategory() {
		return category;
	}

	public void setCategory(Category category) {
		this.category = category;
	}

	public User getUser() {
		return user;
	}

	public void setUser(User user) {
		this.user = user;
	}

	public Date getCreatedAt() {
		return createdAt;
	}

	public void setCreatedAt(Date createdAt) {
		this.createdAt = createdAt;
	}

	public Set<Tag> getTags() {
		return tags;
	}

	public void setTags(Set<Tag> tags) {
		this.tags = tags;
	}
}
```
`Post` 类中，因为 `User` 和 `Post` 的一对多关系，所以 `user` 字段加上了 `@JoinColumn(name = "user_id")`，表明会在 `Post` 表中新增加一列 `user_id` ，因为一对多关系大多数会在“多”的一方加入“一”那一方的主键来做外键（如果不加 `@JoinColumn(name = "user_id")` ，JPA 会默认新建一个表来维持这种关系）。
`Tag` 表：
```
@Entity
@Table
public class Tag extends BaseModel {
	private String name;

	@ManyToMany
	@JoinTable(name = "post_tag", joinColumns = @JoinColumn(name = "tag_id"), inverseJoinColumns = @JoinColumn(name = "post_id"))
	private Set<Post> users = new HashSet<>();

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public Set<Post> getUsers() {
		return users;
	}

	public void setUsers(Set<Post> users) {
		this.users = users;
	}
}
```
来说一下多对多，多对多需要一个额外的表来保持关系，所以 `Post` 和 `Tag` 表都使用了 `@JoinTable` 注解。其中 `name` 指的是额外表的名称，当前是 `post_tag`，`joinColumns` 指的是当前当前表的主键在额外的表中的名称，如 `tag_id` 表明  `Tag`  的 ID 在 `post_tag` 中名称为  `tag_id` 。`inverseJoinColumns` 与 `joinColumns` 正好相反，`Post` 和 `Tag` 中他们的属性正好相反。

## Repository

#### 核心概念
Spring Data 中最核心的一个接口是 `Repository` 接口，这是一个空接口，就像 `Serializable` 接口一样只提供一个标识，供 Spring 动态代理其方法（或者路由到接口的实现上）。实质上，`Repository` 只是为了获取 Domain Class 和  Domain Class 的 ID 类型等信息。

但是只提供给开发者一个空接口太不负责了！所以 Spring 内置提供了 `CrudRepository` （继承自 `Repository`，`Crud` 的意思是增删改查等基本操作）,这样我们可以对数据库进行基本的操作。每个方法的意思不用注释，应该能根据它的方法名，参数，返回类型来判断吧：
```
public interface CrudRepository<T, ID extends Serializable>
    extends Repository<T, ID> {

    <S extends T> S save(S entity); 

    T findOne(ID primaryKey);       

    Iterable<T> findAll();          

    Long count();                   

    void delete(T entity);          

    boolean exists(ID primaryKey);  

    // … more functionality omitted.
}
```

当然继承自 `CrudRepository` 的有一个 `PagingAndSortingRepository` ，提供了基本的分页操作。
```
public interface PagingAndSortingRepository<T, ID extends Serializable>
  extends CrudRepository<T, ID> {

  Iterable<T> findAll(Sort sort);

  Page<T> findAll(Pageable pageable);
}
```

因此在项目中我们可以根据需要继承 `Repository` ，`CrudRepository` ，`PagingAndSortingRepository` 或者 `JpaRepository` （停供了一些批量操作，如批量删除、增加）。如果你想为项目中所有 `Repository` 创建一个自定义的基 `Repository` 来让所有继承自该接口的接口共享方法，可以使用 `@NoRepositoryBean` 注解，内置的 `PagingAndSortingRepository` 或者 `JpaRepository` 也都加了  `@NoRepositoryBean` `@NoRepositoryBean` 注解，这表明 Spring 不会在运行时动态生成该接口的实例：
```
@NoRepositoryBean
interface MyBaseRepository<T, ID extends Serializable> extends Repository<T, ID> {

  T findOne(ID id);

  T save(T entity);
	
  T customSharedMethod(String arg);
}

interface UserRepository extends MyBaseRepository<User, Long> {
  User findByEmailAddress(EmailAddress emailAddress);
}
```
注意到 `findOne(…)` 和 `save(…)` 方法 Spring 在 `SimpleJpaRepository` 类中已经有了实现，所以调用这些方法时会路由到 `SimpleJpaRepository` 中对用的方法，而自定义的  `SimpleJpaRepository` 中不存在方法如 `T customSharedMethod(String arg)` 会由 Spring 动态代理来执行。

因为 Spring 内置的方法 `Repository`s 不可能满足我们所有的需求，所以在 `Repository` 中自定义自己的方法是必不可少的。

那么问题来了，我们自定义的方法， Spring 怎么知道如何去执行呢？Spring 有一套自己的查找策略。

如果你的方法签名比较简单，像 `findOne(…)` 或者 `save(…)` ，这些方法的执行会被路由到 `SimpleJpaRepository` 中对应的方法去执行。而其他的 Spring 则利用自己的机制来解析。

机制首先找到方法名的 `find…By`, `read…By`, `query…By`, `count…By`, 或者 `get…By` 前缀，接着解析剩下的部分。`...` 的位置亦可以插入一些高级的表达式，如 `Distinct` 。注意，`By` 是前缀和真正实体属性的分隔符，`By` 后面可以添加多个属性，用 `And` 或者 `Or` 连接。
例如：
``` java
public interface PersonRepository extends Repository<User, Long> {

  // 可以直接根据另一个关联实体进行查找
  List<Person> findByEmailAddressAndLastname(EmailAddress emailAddress, String lastname);

  // 查找非重复行
  List<Person> findDistinctPeopleByLastnameOrFirstname(String lastname, String firstname);
  List<Person> findPeopleDistinctByLastnameOrFirstname(String lastname, String firstname);

  // 忽略大小写
  List<Person> findByLastnameIgnoreCase(String lastname);
  List<Person> findByLastnameAndFirstnameAllIgnoreCase(String lastname, String firstname);

  // 按 Firstname 排序
  List<Person> findByLastnameOrderByFirstnameAsc(String lastname);
  List<Person> findByLastnameOrderByFirstnameDesc(String lastname);
}
```

#### 属性表达式

属性表达式（Property Expressions）只能指向被 `Repository` 管理的实体的属性，但是也可以指向实体的属性的嵌套属性：假设 `Person` 有一个 `Address` 属性，`Address` 有一个 `ZipCode` 属性，那么我们可以根据 `ZipCode` 来查找 `Person`：
``` java
List<Person> findByAddressZipCode(ZipCode zipCode);
```
这会判断 `x.address.zipCode` 是否和 `zipCode` 相等（根据 `ZipCode` 的主键来判断 ）。具体的解析策略是：Spring 检测到整个属性表达式 `AddressZipCode` ( `By` 后面)  ，然后判断实体类是否有 `addressZipCode` 这个属性，如果有，则算法停止，使用该属性；否则，算法根据驼峰命名规则，从右边将该属性分为“头”和“尾”两部分，例如 `AddressZip` 和 `Code`。如果实体类拥有“头”所指示的属性，然后 Spring 会从此处开始，按照刚才的方法将“尾”拆分，来递归判断是否属于“头”的属性。如果第一次拆分实体类没有“头”所指示的属性，则算法左移一步，拆分为 `Address` 和 `ZipCode` 然后继续。

大所述情况下该算法都会正常工作，但也有例外。比如 `Person` 有 `addressZip` 属性，但是 `AddressZip` 没有 `code` 属性，因此就会在第一次分割后失败。这种情况下，我们可以使用 `_` 显示分割：
``` java
List<Person> findByAddress_ZipCode(ZipCode zipCode);
```

#### 其他

我们还可以传 `Pageable` 和 `Sort` 参数来实现分页和排序：

``` java
Page<User> findByLastname(String lastname, Pageable pageable);

Slice<User> findByLastname(String lastname, Pageable pageable);

List<User> findByLastname(String lastname, Sort sort);

List<User> findByLastname(String lastname, Pageable pageable);
```

还可以使用 `first` 或者 `top` 限制返回结果的个数：

``` java
User findFirstByOrderByLastnameAsc();

User findTopByOrderByAgeDesc();

Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);

Slice<User> findTop3ByLastname(String lastname, Pageable pageable);

List<User> findFirst10ByLastname(String lastname, Sort sort);

List<User> findTop10ByLastname(String lastname, Pageable pageable);
```

可以返回 Java 8 的 `Stream` 对象：
```
@Query("select u from User u")
Stream<User> findAllByCustomQueryAndStream();

Stream<User> readAllByFirstnameNotNull();

@Query("select u from User u")
Stream<User> streamAllPaged(Pageable pageable);
```
注意，使用 `Stream` 之后必须手动关闭，或者这样写：
```
try (Stream<User> stream = repository.findAllByCustomQueryAndStream()) {
  stream.forEach(…);
}
```

还可以返回异步加载的对象：

``` java
//使用 java.util.concurrent.Future 作为返回对象.
@Async
Future<User> findByFirstname(String firstname);               

//使用 Java 8 java.util.concurrent.CompletableFuture 作为返回对象.
@Async
CompletableFuture<User> findOneByFirstname(String firstname); 

//使用 org.springframework.util.concurrent.ListenableFuture 作为返回对象.
@Async
ListenableFuture<User> findOneByLastname(String lastname);    
```

## 附录
#### 方法名支持的关键字
| Keyword           | Sample                                                  | JPQL snippet                                                  |
|-------------------|---------------------------------------------------------|---------------------------------------------------------------|
| And               | findByLastnameAndFirstname                              | ? where x.lastname = ?1 and x.firstname = ?2                  |
| Or                | findByLastnameOrFirstname                               | ? where x.lastname = ?1 or x.firstname = ?2                   |
| Is,Equals         | findByFirstname,findByFirstnameIs,findByFirstnameEquals | ? where x.firstname = ?1                                      |
| Between           | findByStartDateBetween                                  | ? where x.startDate between ?1 and ?2                         |
| LessThan          | findByAgeLessThan                                       | ? where x.age < ?1                                            |
| LessThanEqual     | findByAgeLessThanEqual                                  | ? where x.age <= ?1                                           |
| GreaterThan       | findByAgeGreaterThan                                    | ? where x.age > ?1                                            |
| GreaterThanEqual  | findByAgeGreaterThanEqual                               | ? where x.age >= ?1                                           |
| After             | findByStartDateAfter                                    | ? where x.startDate > ?1                                      |
| Before            | findByStartDateBefore                                   | ? where x.startDate < ?1                                      |
| IsNull            | findByAgeIsNull                                         | ? where x.age is null                                         |
| IsNotNull,NotNull | findByAge(Is)NotNull                                    | ? where x.age not null                                        |
| Like              | findByFirstnameLike                                     | ? where x.firstname like ?1                                   |
| NotLike           | findByFirstnameNotLike                                  | ? where x.firstname not like ?1                               |
| StartingWith      | findByFirstnameStartingWith                             | ? where x.firstname like ?1(parameter bound with appended˙%)  |
| EndingWith        | findByFirstnameEndingWith                               | ? where x.firstname like ?1(parameter bound with prepended˙%) |
| Containing        | findByFirstnameContaining                               | ? where x.firstname like ?1(parameter bound wrapped in˙%)     |
| OrderBy           | findByAgeOrderByLastnameDesc                            | ? where x.age = ?1 order by x.lastname desc                   |
| Not               | findByLastnameNot                                       | ? where x.lastname <> ?1                                      |
| In                | findByAgeIn(Collection<Age> ages)                       | ? where x.age in ?1                                           |
| NotIn             | findByAgeNotIn(Collection<Age> age)                     | ? where x.age not in ?1                                       |
| TRUE              | findByActiveTrue()                                      | ? where x.active = true                                       |
| FALSE             | findByActiveFalse()                                     | ? where x.active = false                                      |
| IgnoreCase        | findByFirstnameIgnoreCase                               | ? where UPPER(x.firstame) = UPPER(?1)                         |

#### 方法支持的返回值
| Return type          | Description                                                                                                                                                                                                                                |
|----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| void                 | 无返回值.                                                                                                                                                                                                                   |
| Primitives           | Java primitives.                                                                                                                                                                                                                           |
| Wrapper types        | Java wrapper types.                                                                                                                                                                                                                        |
| T                    | 返回最多一个实体(否咋抛出 `IncorrectResultSizeDataAccessException`)，返回 `null` 代表未找到.                                         |
| Iterator<T>          | Java Iterator.                                                                                                                                                                                                                               |
| Collection<T>        | Java Collection.                                                                                                                                                                                                                              |
| List<T>              | Java List.                                                                                                                                                                                                                                    |
| Optional<T>          | Java 8 Optional， 返回最多一个实体(否咋抛出 `IncorrectResultSizeDataAccessException`)|
| Stream<T>            | Java 8 Stream.                                                                                                                                                                                                                           |
| Future<T>            | Java Future，需要加 `@Async` 注解，并且Spring 开启异步  |
| CompletableFuture<T> | Java CompletableFuture，需要加 `@Async` 注解，并且Spring 开启异步.                                                                                             |
| ListenableFuture     | `org.springframework.util.concurrent.ListenableFuture`，需要加 `@Async` 注解，并且Spring 开启异步                                                                |
| Slice                | 一组固定大小的数据，可以判断是否有更多数据，需要 `Pageable` 参数                                                                                                                         |
| Page<T>              | 分页，包括一页的数据，元素总数，总共页数等，需要 `Pageable` 参数                                                                                                                                |


## 最后
说了这么多，Spring  提供的方法已经够强大了，基本满足日常需求。但对于更复杂的需求，如投影、关联等高级的查询操作，Spring 就显得心有余而力不足了。下篇介绍[Querydsl](http://www.querydsl.com/)，看这里[Spring Boot JPA - Querydsl](https://lufficc.com/blog/spring-boot-jpa-querydsl)，做个引子，说明如果使用 Spring 进行复杂的数据操作。

## 参考
1. [Java Persistence/What is JPA?](https://en.wikibooks.org/wiki/Java_Persistence/What_is_JPA%3F)
2. [Spring Data JPA - Reference Documentation](http://docs.spring.io/spring-data/jpa/docs/2.0.0.M1/reference/html/)
3. [Querydsl Reference Guide](http://www.querydsl.com/static/querydsl/latest/reference/html_single/)
