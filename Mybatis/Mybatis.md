# MyBatis

## 1. MyBatis工作原理

（1）读取 MyBatis 配置文件 mybatis-config .xml mybatis-config.xml 作为 MyBatis 的全局配置文件，配置了 MyBatis 的运行环境等信息，其中主要内容是获取数据库连接。

```java
public class MybatisUtils {
    private static SqlSessionFactory sqlSessionFactory = null;
    static {
        //用resources加载mybatis-config文件
        try {
            Reader reader = Resources.getResourceAsReader("mybatis-config.xml");
            //构建工厂
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
    
    //封装opensession
    public static SqlSession openSession() {
        return sqlSessionFactory.openSession();
    }
}
```

（2）加载映射文件 Mapper.xml。 Mapper.xml 文件即 SOL 映射文件，该文件中配置了操作数据库的 SOL 语句，需要在 mybatis-config .xml 中加载才能执行 mybatis-config .xml 可以加载多个配置文件，每个配置文件对应数据库中的一张表。

```xml
<mappers>
    <mapper resource="com/xd/mapper/CustomerMapper.xml"/>
    <mapper resource="com/xd/mapper/UserMapper.xml"/>
</mappers>
```

（3）构建会话 通过 MyBatis 的环境等配置信息构建会话工厂 SqlSessionFactory。

```java
InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml") ; 
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

（4）创 SqlSession 对象由会话工厂创建 SqlSession 对象，该对象中包含了执行 SOL的所有方法。

```java
SqlSession sqlSession = sqlSessionFactory.openSession();
try { 
	// 此处执行持久化操作
} finally { 
	sqlSession.close();
}
```

（5）MyBatis 底层定义了一个 Executor 接口来操作数据库，它会根据 SqlSession 传递的参数动态地生成需要执行的 SOL 语句，同时负责查询缓存的维护。

（6）在 Executor 接口的执行方法中，包含一个 MappedStatement 类型的参数， 该参数是对映射信息的封装 用于存储要映射的 SOL 语句的 id 、参数等 Mapper.xml 文件中一个 SOL对应一个 MappedStatement 对象 SOL的id 即是 MappedStatement的id。

（7）输入参数映射在执行方法时， MappedStatement 对象会对用户执行 SOL 语句的输入参数进行定义(可以定义为 Map 、Li st 类型、基本类型和 POJO 类型 Executor 执行器会通过MappedStatement 对象在执行 SOL 前，将输入的 Java 对象映射到 SOL 语句中 这里对输入参数的映射过程就类似于 JDBC 编程中对 preparedStatement 对象设置参数的过程。

（8）输出结果映射在数据库中执行完 SOL 语句后， MappedStatement 对象会对 SOL执行输出的结果进行定义(可以定义为 Map 和List 类型、基本类型、 POJO 类型 Executor 执行器会通过 MappedStatement 对象在执行 SOL 语句后，将输出结果映射至 Java 对象中 这种将输出结果映射到 Java 对象的过程就类似于 JDBC 编程中对结果的解析处理过程。

## 2. MyBatis基本的CRUD

resultType参数值在mybatis-config.xml配置文件中，以通过typeAliases标签取别名

```xml
<!-- 取别名 -->
<typeAliases>
    <!-- 别名不区分大小写 -->
    <package name="com.xd.po"/>
</typeAliases>
```

### 2.1 查询客户

（1） 根据客户编号获取客户信息

```xml
<!-- 根据客户编号获取客户信息 -->
<select id="findCustomerByld" parameterType="Integer" resultType="customer"> 
	select * from t custæ where id = #{id} 
</select>
```

（2）根据用户名模糊查询客户，并使用SQL元素替换SQL语句

```xml
<!-- 定义前缀名 -->
<sql id="tablename">
    ${prefix}customer
</sql>

<!-- 定义要查询的表 -->
<sql id="someinclude">
    from <include refid="${include_target}"/>
</sql>

<!-- 要查询的列 -->
<sql id="customerColumns">
    id,username,jobs,phone
</sql>

<!-- 根据用户名称模糊查询 -->
<select id="findCustomerLikeUsername" parameterType="String" resultType="customer">
    SELECT <include refid="customerColumns" />
    <include refid="someinclude">
        <property name="prefix" value="t_" />
        <property name="include_target" value="tablename" />
    </include>
    where username LIKE concat('%', #{value}, '%')
</select>
```

使用 MySQL 中的 **concat()** 函数可以在模糊查询中防止sql注入攻击。

### 2.2 添加客户

```xml
<!-- 添加客户信息(不带主键) -->
<insert id="addCustomer" keyProperty="id" useGeneratedKeys="true"  parameterType="customer">
    insert into t_customer(username,jobs,phone) values(#{username},#{jobs},#{phone})
</insert>
```

```xml
<!-- 添加客户信息（带主键） -->
<insert id="insertCustomer" parameterType="customer">
    <selectKey keyProperty="id" resultType="Integer" order="BEFORE">
        select if(max(id) is null,1,max(id)+1) as newId from t_customer
    </selectKey>
    insert into t_customer(username,jobs,phone)
    values(#{id},#{username},#{jobs},#{phone})
</insert>
```

### 2.3 更新客户

```xml
<!-- 更新客户信息 -->
<update id="updateCustomer" parameterType="customer">
    update t_customer set username=#{username},jobs=#{jobs},phone=#{phone}
    where id=#{id}
</update>
```

### 2.4 删除客户

```xml
<!-- 删除功能 -->
<delete id="deleteCustomer" parameterType="Integer">
    delete from t_customer where id=#{id}
</delete>
```

## 3. 配置文件主要元素

### 3.1 properties 元素

**properties**  是一个配置属性的元素，该元素通常用于将内部的配置外在化，即通过外部的配置来动态地替换内部定义的属性 例如，数据库的连接等属性，就可以通过典型的 Java 属性文件中的配置来替换，具体方式如下。

（1）Eclipse开发工具中，在项目的 src 目录下，添加一个全名为 db.properties 的配置文件，编辑后的代码如下所示

```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatisTest?useSSL=false&serverTimezone=UTC
jdbc.username=root
jdbc.password=123456
```

（2）MyBatis 配置文件 mybatis-config .xml 中配置<properties ... />属性

```xml
<!-- 数据库连接配置文件 -->
<properties resource="db.properties"/>
```

（3）修改配置文件中数据库连接的信息

```xml
<!-- 数据库连接池 -->
<dataSource type="POOLED">
    <property name="driver" value="${jdbc.driver}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</dataSource>
```

### 3.2  typeAliases 元素

**typeAliases** 元素用于为配置文件中的 Java 类型设置一个简短的名字，即设置别名。别名的设置与 XML 配置相关，其使用的意义在于减少全限定类名的冗余。

（1）使用  **typeAliases** 元素配置别名的方法如下

```xml
<!-- 定义别名 -->
<typeAliases> 
	<typeAlias alias="customer" type="com.xd.po.Customer"/>
</typeAliases>
```

（2）当POJO 类过多时，还可以通过自动扫描包的形式自定义别名

```xml
<typeAliases>
    <!-- 别名不区分大小写 -->
    <package name="com.xd.po"/>
</typeAliases>
```

需要注意的是，上述方式的别名只适用于没有使用注解的情况 如果在程序中使用了注解，则别名为其注解的值。

```java
@A1ias(value = "user") 
public class User { 
	//User 的属性和方法
    ...
}
```

### 3.3  **environments** 元素

​		在配置文件中， environments 元素用于对环境进行配置。 MyBatis 的环境配置实际上就是数据源的配置，我们可以通过 environments 元素配置多种数据源，即配置多种数据库。

```xml
<!-- 配置环境,m默认为mysql -->
<environments default="mysql">
    <environment id="mysql">
        <transactionManager type="JDBC"/>
        <!-- 数据库连接池 -->
        <dataSource type="POOLED">
            <property name="driver" value="${jdbc.driver}"/>
            <property name="url" value="${jdbc.url}"/>
            <property name="username" value="${jdbc.username}"/>
            <property name="password" value="${jdbc.password}"/>
        </dataSource>
    </environment>
</environments>
```

在MyBatis 可以配置两种类型的事务管理器，分别是 JDBC 和 MANAGED 关于这两个事务管理器的描述如下：

- DBC 此配置直接使 JDB 的提交和回滚设置 它依赖于从数据源得到的连接来管理事务的作用域。

- GED: 此配置从来不提交或回滚一个连接，而是让容器来管理事务的整个生命周期在默认情况下它会关闭连接，但一些容器并不希望这样。为此可以将 closeConnection 属性设置为 false 回来阻止它默认关闭行为。

**注意：**如果项目 使用的是 Spring+ MyBatis ，则没有 要在 MyBatis 中配置事务管理器，因为实际开发中，会使用 Spring 自带管理器来实现事务管理。

### 3.4 mappers 元素

在配置文件中， mappers 元素用于指定 MyBatis 映射文件的位置。

（1）使用类路径引入

```xml
<mappers>
    <mapper resource="com/xd/mapper/CustomerMapper.xml"/>
</mappers>
```

（2）使用本地文件路径引入

```xml
<mappers>
    <mapper resource="file:///D:/com/xd/mapper/CustomerMapper.xml"/>
</mappers>
```

（3）使用接口类引入

```xml
<mappers>
    <mapper class="com.xd.mapper.CustomerMapper"/>
</mappers>
```

（4）使用包名引入

```xml
<mappers> 
	<package name="com.xd.mapper"/> 
</mappers>
```

