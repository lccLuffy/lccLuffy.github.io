---
layout: post
permalink: /spring-boot-jpa-querydsl/
title: Spring Boot JPA - Querydsl
date: 2017-03-27T23:06:00
lang: zh-CN
---

[Querydsl](http://www.querydsl.com) 是一个类型安全的 Java 查询框架，支持 JPA, JDO, JDBC, Lucene, Hibernate Search 等标准。类型安全（Type safety）和一致性（Consistency）是它设计的两大准则。在 Spring Boot 中可以很好的弥补 JPA 的不灵活，实现更强大的逻辑。

## 依赖
```xml
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
</dependency>
```

因为是类型安全的，所以还需要加上Maven APT plugin，使用 APT 自动生成一些类:
```xml
<project>
  <build>
  <plugins>
    ...
    <plugin>
    <groupId>com.mysema.maven</groupId>
    <artifactId>apt-maven-plugin</artifactId>
    <version>1.1.3</version>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>process</goal>
            </goals>
            <configuration>
                <outputDirectory>target/generated-sources</outputDirectory>
                <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
            </configuration>
        </execution>
    </executions>
</plugin>
    ...
  </plugins>
  </build>
</project>
```
## 基本概念
每一个 Model (使用 `@javax.persistence.Entity` 注解的)，Querydsl 都会在同一个包下生成一个以 `Q` 开头（默认，可配置）的类，来实现便利的查询操作。
如：
```java
JPAQueryFactory queryFactory = new JPAQueryFactory(entityManager);
// 基本查询
List<Person> persons = queryFactory.selectFrom(person)
  .where(
    person.firstName.eq("John"),
    person.lastName.eq("Doe"))
  .fetch();

// 排序
List<Person> persons = queryFactory.selectFrom(person)
  .orderBy(person.lastName.asc(), 
           person.firstName.desc())
  .fetch();

// 子查询
List<Person> persons = queryFactory.selectFrom(person)
  .where(person.children.size().eq(
    JPAExpressions.select(parent.children.size().max())
                  .from(parent)))
  .fetch();

// 投影
List<Tuple> tuples = queryFactory.select(
    person.lastName, person.firstName, person.yearOfBirth)
  .from(person)
  .fetch();
```
有没有很强大？ 。。。。。 来看一个具体一点的例子吧：

## 实例
接着上一章的几个表，来一点高级的查询操作：

Spring 提供了一个便捷的方式使用 Querydsl，只需要在 `Repository` 中继承 `QueryDslPredicateExecutor` 即可：
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long>, QueryDslPredicateExecutor<User> {
}
```
然后就可以使用 `UserRepository` 无缝和 Querydsl 连接：
```java
userRepository.findAll(user.name.eq("lufifcc")); // 所有用户名为 lufifcc 的用户

userRepository.findAll(
        user.email.endsWith("@gmail.com")
                .and(user.name.startsWith("lu"))
                .and(user.id.in(Arrays.asList(520L, 1314L, 1L, 2L, 12L)))
); // 所有 Gmail 用户，且名字以 lu 开始，并且 ID 在520L, 1314L, 1L, 2L, 12L中

userRepository.count(
        user.email.endsWith("@outlook.com")
                .and(user.name.containsIgnoreCase("a"))
); // Outlook 用户，且名字以包含 a (不区分大小写)的用户数量

userRepository.findAll(
        user.email.endsWith("@outlook.com")
                .and(user.posts.size().goe(5))
); // Outlook 用户，且文章数大于等于5

userRepository.findAll(
        user.posts.size().eq(JPAExpressions.select(user.posts.size().max()).from(user))
);// 文章数量最多的用户
```
另外， Querydsl 还可以采用 SQL 模式查询，加入依赖：
```xml
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-sql</artifactId>
</dependency>
```
#### 1. 然后，获取所有用户邮箱：

```java
@GetMapping("users/emails")
public Object userEmails() {
    QUser user = QUser.user;
    return queryFactory.selectFrom(user)
            .select(user.email)
            .fetch();
}
// 返回
[
  "lufficc@qq.com",
  "allen@qq.com",
  "mike@qq.com",
  "lucy@qq.com"
]
```

#### 2. 获取所有用户名和邮箱：
```java
@GetMapping("users/names-emails")
public Object userNamesEmails() {
    QUser user = QUser.user;
    return queryFactory.selectFrom(user)
            .select(user.email, user.name)
            .fetch()
            .stream()
            .map(tuple -> {
                Map<String, String> map = new LinkedHashMap<>();
                map.put("name", tuple.get(user.name));
                map.put("email", tuple.get(user.email));
                return map;
            }).collect(Collectors.toList());
}
// 返回
[
  {
    "name": "Lufficc",
    "email": "lufficc@qq.com"
  },
  {
    "name": "Allen",
    "email": "allen@qq.com"
  },
  {
    "name": "Mike",
    "email": "mike@qq.com"
  },
  {
    "name": "Lucy",
    "email": "lucy@qq.com"
  }
]
```

注意到投影的时候，我们可以直接利用查询时用到的表达式来获取类型安全的值，如获取 `name` : `tuple.get(user.name)`，返回值是 String 类型的。

#### 3. 获取所有用户ID，名称，和他们的文章数量：
```java
@GetMapping("users/posts-count")
public Object postCount() {
    QUser user = QUser.user;
    QPost post = QPost.post;
    return queryFactory.selectFrom(user)
            .leftJoin(user.posts, post)
            .select(user.id, user.name, post.count())
            .groupBy(user.id)
            .fetch()
            .stream()
            .map(tuple -> {
                Map<String, Object> map = new LinkedHashMap<>();
                map.put("id", tuple.get(user.id));
                map.put("name", tuple.get(user.name));
                map.put("posts_count", tuple.get(post.count()));
                return map;
            }).collect(Collectors.toList());
}
// 返回
[
  {
    "id": 1,
    "name": "Lufficc",
    "posts_count": 9
  },
  {
    "id": 2,
    "name": "Allen",
    "posts_count": 6
  },
  {
    "id": 3,
    "name": "Mike",
    "posts_count": 6
  },
  {
    "id": 4,
    "name": "Lucy",
    "posts_count": 6
  }
]
```

#### 4. 获取所有用户名，以及对应用户的 `Java` 、`Python` 分类下的文章数量：
```java
@GetMapping("users/category-count")
public Object postCategoryMax() {
    QUser user = QUser.user;
    QPost post = QPost.post;
    NumberExpression<Integer> java = post.category
            .name.lower().when("java").then(1).otherwise(0);
    NumberExpression<Integer> python = post.category
            .name.lower().when("python").then(1).otherwise(0);
    return queryFactory.selectFrom(user)
            .leftJoin(user.posts, post)
            .select(user.name, user.id, java.sum(), python.sum(), post.count())
            .groupBy(user.id)
            .orderBy(user.name.desc())
            .fetch()
            .stream()
            .map(tuple -> {
                Map<String, Object> map = new LinkedHashMap<>();
                map.put("username", tuple.get(user.name));
                map.put("java_count", tuple.get(java.sum()));
                map.put("python_count", tuple.get(python.sum()));
                map.put("total_count", tuple.get(post.count()));
                return map;
            }).collect(Collectors.toList());
}

// 返回
[
  {
    "username": "Mike",
    "java_count": 3,
    "python_count": 1,
    "total_count": 5
  },
  {
    "username": "Lufficc",
    "java_count": 2,
    "python_count": 4,
    "total_count": 9
  },
  {
    "username": "Lucy",
    "java_count": 1,
    "python_count": 1,
    "total_count": 5
  },
  {
    "username": "Allen",
    "java_count": 2,
    "python_count": 1,
    "total_count": 5
  }
]
```
注意这里用到了强大的 [Case 表达式（Case expressions）](http://www.querydsl.com/static/querydsl/4.1.3/reference/html_single/#d0e2105) 。

#### 5. 统计每一年发文章数量，包括 `Java` 、`Python` 分类下的文章数量：
```java
@GetMapping("posts-summary")
public Object postsSummary() {
    QPost post = QPost.post;
    ComparableExpressionBase<?> postTimePeriodsExp = post.createdAt.year();
    NumberExpression<Integer> java = post.category
            .name.lower().when("java").then(1).otherwise(0);
    NumberExpression<Integer> python = post.category
            .name.lower().when("python").then(1).otherwise(0);
    return queryFactory.selectFrom(post)
            .groupBy(postTimePeriodsExp)
            .select(postTimePeriodsExp, java.sum(), python.sum(), post.count())
            .orderBy(postTimePeriodsExp.asc())
            .fetch()
            .stream()
            .map(tuple -> {
                Map<String, Object> map = new LinkedHashMap<>();
                map.put("time_period", tuple.get(postTimePeriodsExp));
                map.put("java_count", tuple.get(java.sum()));
                map.put("python_count", tuple.get(python.sum()));
                map.put("total_count", tuple.get(post.count()));
                return map;
            }).collect(Collectors.toList());
}

// 返回
[
  {
    "time_period": 2015,
    "java_count": 1,
    "python_count": 3,
    "total_count": 6
  },
  {
    "time_period": 2016,
    "java_count": 4,
    "python_count": 2,
    "total_count": 14
  },
  {
    "time_period": 2017,
    "java_count": 3,
    "python_count": 2,
    "total_count": 7
  }
]
```

## 补充一点
Spring 参数支持解析 `com.querydsl.core.types.Predicate`，根据用户请求的参数自动生成 `Predicate`，这样搜索方法不用自己写啦，比如：
```java
@GetMapping("posts")
public Object posts(@QuerydslPredicate(root = Post.class) Predicate predicate) {
    return postRepository.findAll(predicate);
}
// 或者顺便加上分页
@GetMapping("posts")
public Object posts(@QuerydslPredicate(root = Post.class) Predicate predicate, Pageable pageable) {
    return postRepository.findAll(predicate, pageable);
}
```
然后请求：
```
/posts?title=title01 // 返回文章 title 为 title01 的文章

/posts?id=2 // 返回文章 id 为 2 的文章

/posts?category.name=Python // 返回分类为 Python 的文章（你没看错，可以嵌套，访问关系表中父类属性）

/posts?user.id=2&category.name=Java // 返回用户 ID 为 2 且分类为 Java 的文章
```
注意，这样不会产生 `SQL` 注入问题的，因为不存在的属性写了是不起效果的，Spring 已经进行了判断。
再补充一点，你还可以修改默认行为，继承 `QueryDslPredicateExecutor` 接口：
```java
@Repository
public interface PostRepository extends JpaRepository<Post, Long>, QueryDslPredicateExecutor<Post>, QuerydslBinderCustomizer<QPost> {
    default void customize(QuerydslBindings bindings, QPost post) {
        bindings.bind(post.title).first(StringExpression::containsIgnoreCase);
    }
}
```
这样你再访问 `/posts?title=title01` 时，返回的是文章标题包含 title01 ，而不是仅仅等于的所有文章啦!

## 总结
本文知识抛砖引玉， Querydsl 的强大之处并没有完全体现出来，而且 Spring Boot 官方也提供了良好的支持，所以，掌握了 Querydsl，真的不愁 Java 写不出来的 `Sql`，重要的是类型安全（没有强制转换），跨数据库（Querydsl 底层还是 JPA 这样的技术，所有只要不写原生 Sql，基本关系型数据库通用）。对了，源代码在这里 [example-jpa](https://github.com/lufficc/example-jpa)，可以直接运行的~~
大家有什么好的经验也可以在评论里提出来啊。
