---
title: Choose Driver or ORM
summary: Learn how to choose a driver or ORM framework to connect to TiDB.
---

# Choose Driver or ORM

> **Note:**
>
> TiDB provides the following two support levels for drivers and ORMs:
>
> - **Full**: indicates that using this driver or ORM does not have any known issues.
> - **Verified**: indicates that using this driver or ORM might get errors because of compatibility differences between TiDB and MySQL.

<CustomContent platform="tidb">

>
> For more information, refer to [Third-Party Tools Supported by TiDB](/develop/dev-guide-third-party-support.md).

</CustomContent>

TiDB is highly compatible with the MySQL protocol but some features are incompatible with MySQL.

For example:

- Features that are not supported by TiDB:

    - Stored procedures and functions
    - Triggers
    - `FOREIGN KEY` constraints

- Features that are different from MySQL:

    - Auto-increment ID: auto-incremental columns are globally unique in TiDB. They are incremental on a single TiDB server, but ***not*** necessarily incremental among multiple TiDB servers or allocated sequentially.

For a full list of compatibility differences, see [MySQL Compatibility](/mysql-compatibility.md).

## Java

This section describes how to use drivers and ORM frameworks in Java.

### Java drivers

<SimpleTab>
<div label="MySQL-JDBC">

Support level: **Full**

You can follow the [MySQL documentation](https://dev.mysql.com/doc/connector-j/8.0/en/) to download and configure a Java JDBC driver. It is recommended to use MySQL Connector/J 8.0.29 or later with TiDB v6.3.0 and newer.

> **Tip:**
>
> Since there is an [unresolved bug](https://bugs.mysql.com/bug.php?id=106252) in the current version 8.0.30, which might cause threads to hang when using TiDB versions earlier than v6.3.0. It is recommended that, if you are not yet using TiDB v6.3.0 or later versions, you do not upgrade to version 8.0 until MySQL Connector/J 8.0 fixes this bug, or use the TiDB version of MySQL Connector/J as that addresses this bug (see the *TiDB-JDBC* tab).

For an example of how to build a complete application, see [Build a Simple CRUD App with TiDB and JDBC](/develop/dev-guide-sample-application-java.md).

</div>
<div label="TiDB-JDBC">

Support level: **Full**

[TiDB-JDBC](https://github.com/pingcap/mysql-connector-j) is a customized Java driver based on MySQL 8.0.29. Compiled based on MySQL official version 8.0.29, TiDB-JDBC fixes the bug of multi-parameter and multi-field EOF in the prepare mode in the original JDBC, and adds features such as automatic TiCDC snapshot maintenance and the SM3 authentication plugin.

Using SM3-based authentication is only supported with the TiDB version of MySQL Connector/J.

If you use Maven, add the following content to the `<dependencies></dependencies>` section in the `pom.xml` file:

```xml
<dependency>
  <groupId>io.github.lastincisor</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>8.0.29-tidb-1.0.0</version>
</dependency>
```

If you need to enable SM3 authentication, add the following content to the `<dependencies></dependencies>` section in the `pom.xml` file:

```xml
<dependency>
  <groupId>io.github.lastincisor</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>8.0.29-tidb-1.0.0</version>
</dependency>
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.67</version>
</dependency>
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcpkix-jdk15on</artifactId>
    <version>1.67</version>
</dependency>
```

If you use Gradle, add the following content to `dependencies`:

```gradle
implementation group: 'io.github.lastincisor', name: 'mysql-connector-java', version: '8.0.29-tidb-1.0.0'
implementation group: 'org.bouncycastle', name: 'bcprov-jdk15on', version: '1.67'
implementation group: 'org.bouncycastle', name: 'bcpkix-jdk15on', version: '1.67'
```

</div>
</SimpleTab>

### Java ORM frameworks

> **Note:**
>
> - Currently, Hibernate does [not support nested transactions](https://stackoverflow.com/questions/37927208/nested-transaction-in-spring-app-with-jpa-postgres).
> - Since v6.2.0, TiDB supports [savepoint](/sql-statements/sql-statement-savepoint.md). To use the `Propagation.NESTED` transaction propagation option in `@Transactional`, that is, to set `@Transactional(propagation = Propagation.NESTED)`, make sure that your TiDB is v6.2.0 or later.

<SimpleTab>
<div label="Hibernate">

Support level: **Full**

To avoid manually managing complex relationships between different dependencies of an application, you can use [Gradle](https://gradle.org/install) or [Maven](https://maven.apache.org/install.html) to get all dependencies of your application, including those indirect ones. Note that only Hibernate `6.0.0.Beta2` or above supports the TiDB dialect.

If you are using **Maven**, add the following to your `<dependencies></dependencies>`:

{{< copyable "" >}}

```xml
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>6.0.0.CR2</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.49</version>
</dependency>
```

If you are using **Gradle**, add the following to your `dependencies`:

```gradle
implementation 'org.hibernate:hibernate-core:6.0.0.CR2'
implementation 'mysql:mysql-connector-java:5.1.49'
```

- For an example of using Hibernate to build a TiDB application by native Java, see [Build a Simple CRUD App with TiDB and Java](/develop/dev-guide-sample-application-java.md).
- For an example of using Spring Data JPA or Hibernate to build a TiDB application by Spring, see [Build a TiDB Application using Spring Boot](/develop/dev-guide-sample-application-spring-boot.md).

In addition, you need to specify the TiDB dialect in your [Hibernate configuration file](https://www.tutorialspoint.com/hibernate/hibernate_configuration.htm): `org.hibernate.dialect.TiDBDialect`, which is only supported by Hibernate `6.0.0.Beta2` or above. If your `Hibernate` version is earlier than `6.0.0.Beta2`, upgrade it first.

> **Note:**
>
> If you are unable to upgrade your `Hibernate` version, use the MySQL 5.7 dialect `org.hibernate.dialect.MySQL57Dialect` instead. However, this setting might cause unpredictable results and the absence of some TiDB-specific features, such as [sequences](/sql-statements/sql-statement-create-sequence.md).

</div>

<div label="MyBatis">

Support level: **Full**

To avoid manually managing complex relationships between different dependencies of an application, you can use [Gradle](https://gradle.org/install) or [Maven](https://maven.apache.org/install.html) to get all dependencies of your application, including those indirect dependencies.

If you are using Maven, add the following to your `<dependencies></dependencies>`:

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.9</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.49</version>
</dependency>
```

If you are using Gradle, add the following to your `dependencies`:

```gradle
implementation 'org.mybatis:mybatis:3.5.9'
implementation 'mysql:mysql-connector-java:5.1.49'
```

For an example of using MyBatis to build a TiDB application, see [Build a Simple CRUD App with TiDB and Java](/develop/dev-guide-sample-application-java.md).

</div>

</SimpleTab>

### Java client load balancing

**tidb-loadbalance**

Support level: **Full**

[tidb-loadbalance](https://github.com/pingcap/tidb-loadbalance) is a load balancing component on the application side. With tidb-loadbalance, you can automatically maintain the node information of TiDB server and distribute JDBC connections on the client using the tidb-loadbalance policies. Using a direct JDBC connection between the client application and TiDB server has higher performance than using the load balancing component.

Currently, tidb-loadbalance supports the following policies: roundrobin, random, and weight.

> **Note:**
>
> tidb-loadbalance must be used with [mysql-connector-j](https://github.com/pingcap/mysql-connector-j).

If you use Maven, add the following content to the element body of `<dependencies></dependencies>` in the `pom.xml` file:

```xml
<dependency>
  <groupId>io.github.lastincisor</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>8.0.29-tidb-1.0.0</version>
</dependency>
<dependency>
  <groupId>io.github.lastincisor</groupId>
  <artifactId>tidb-loadbalance</artifactId>
  <version>0.0.5</version>
</dependency>
```

If you use Gradle, add the following content to `dependencies`:

```gradle
implementation group: 'io.github.lastincisor', name: 'mysql-connector-java', version: '8.0.29-tidb-1.0.0'
implementation group: 'io.github.lastincisor', name: 'tidb-loadbalance', version: '0.0.5'
```

## Golang

This section describes how to use drivers and ORM frameworks in Golang.

### Golang drivers

**go-sql-driver/mysql**

Support level: **Full**

To download and configure a Golang driver, refer to the [go-sql-driver/mysql documentation](https://github.com/go-sql-driver/mysql).

For an example of how to build a complete application, see [Build a Simple CRUD App with TiDB and Golang](/develop/dev-guide-sample-application-golang.md).

### Golang ORM frameworks

**GORM**

Support level: **Full**

GORM is a popular ORM framework for Golang. To get all dependencies in your application, you can use the `go get` command.

```shell
go get -u gorm.io/gorm
go get -u gorm.io/driver/mysql
```

For an example of using GORM to build a TiDB application, see [Build a Simple CRUD App with TiDB and Golang](/develop/dev-guide-sample-application-golang.md).