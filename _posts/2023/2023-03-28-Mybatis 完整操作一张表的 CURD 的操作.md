---
layout: post
title: Mybatis 完整操作一张表的 CURD 的操作
categories: Mybatis
description: Mybatis 完整操作一张表的 CURD 的操作
keywords: CURD, Mybatis
---

下面是一个使用 MyBatis 完整操作一张表的 CURD 的操作的示例：

1. 创建一张表并插入数据：

```sql
CREATE TABLE user (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(50),
  age INT,
  email VARCHAR(50)
);

INSERT INTO user (name, age, email) VALUES ('Alice', 20, 'alice@example.com');
INSERT INTO user (name, age, email) VALUES ('Bob', 25, 'bob@example.com');
```

2. 创建一个 User 实体类，对应数据库表中的一行数据：

```java
public class User {

    private Integer id;
    private String name;
    private Integer age;
    private String email;

    // getter and setter methods

}
```

3. 编写 MyBatis Mapper 接口，定义对 user 表的操作：

```java
public interface UserMapper {

    @Select("SELECT * FROM user")
    List<User> getAllUsers();

    @Select("SELECT * FROM user WHERE id = #{id}")
    User getUserById(Integer id);

    @Insert("INSERT INTO user (name, age, email) VALUES (#{name}, #{age}, #{email})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    void insertUser(User user);

    @Update("UPDATE user SET name = #{name}, age = #{age}, email = #{email} WHERE id = #{id}")
    void updateUser(User user);

    @Delete("DELETE FROM user WHERE id = #{id}")
    void deleteUser(Integer id);

}
```

- `@Select` 注解用于查询操作，其中的 SQL 语句对应数据库的查询语句；
- `@Insert` 注解用于插入操作，其中的 SQL 语句对应数据库的插入语句；
- `@Options` 注解用于指定插入操作的主键生成方式；
- `@Update` 注解用于更新操作，其中的 SQL 语句对应数据库的更新语句；
- `@Delete` 注解用于删除操作，其中的 SQL 语句对应数据库的删除语句。

4. 编写 MyBatis Mapper XML 文件，定义对 user 表的操作：

```xml
<mapper namespace="com.example.mapper.UserMapper">

    <select id="getAllUsers" resultType="com.example.model.User">
        SELECT * FROM user
    </select>

    <select id="getUserById" resultType="com.example.model.User" parameterType="java.lang.Integer">
        SELECT * FROM user WHERE id = #{id}
    </select>

    <insert id="insertUser" parameterType="com.example.model.User" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO user (name, age, email) VALUES (#{name}, #{age}, #{email})
    </insert>

    <update id="updateUser" parameterType="com.example.model.User">
        UPDATE user SET name = #{name}, age = #{age}, email = #{email} WHERE id = #{id}
    </update>

    <delete id="deleteUser" parameterType="java.lang.Integer">
        DELETE FROM user WHERE id = #{id}
    </delete>

</mapper>
```

- `<mapper>` 标签用于定义 MyBatis Mapper；
- `<select>` 标签用于查询操作，其中的 SQL 语句对应数据库的查询语句；
- `<insert>` 标签用于插入操作，其中的 SQL 语句对应数据库的插入语句；
- `<update>` 标签用于更新操作，其中的 SQL 语句对应数据库的更新语句；
- `<delete>` 标签用于删除操作，其中的 SQL 语句对应数据库的删除语句；
- `resultType` 属性用于指定查询结果的类型；
- `parameterType` 属性用于指定参数的类型；
- `useGeneratedKeys` 属性用于指定插入操作的主键生成方式；
- `keyProperty` 属性用于指定插入操作的主键属性。

5. 编写 MyBatis 配置文件，指定 Mapper 文件的位置和数据库连接信息：

```xml
<configuration>

    <mappers>
        <mapper resource="com/example/mapper/UserMapper.xml"/>
    </mappers>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis_example"/>
                <property name="username" value="root"/>
                <property name="password" value="password"/>
            </dataSource>
        </environment>
    </environments>

</configuration>
```

- `<mappers>` 标签用于指定 Mapper 文件的位置；
- `<mapper>` 标签的 `resource` 属性用于指定 Mapper 文件的路径；
- `<environments>` 标签用于指定数据库连接信息；
- `<transactionManager>` 标签用于指定事务管理器的类型；
- `<dataSource>` 标签用于指定数据源的类型和连接信息；
- `<property>` 标签用于指定连接信息的具体值。

6. 在 Spring Boot 中配置 MyBatis，指定 MyBatis 配置文件的位置和 Mapper 扫描的包：

```java
@Configuration
@MapperScan("com.example.mapper")
public class MyBatisConfig {

    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource) {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        ResourcePatternResolver resourcePatternResolver = new PathMatchingResourcePatternResolver();
        try {
            sqlSessionFactoryBean.setMapperLocations(resourcePatternResolver.getResources("classpath*:com/example/mapper/*.xml"));
        } catch (IOException e) {
            e.printStackTrace();
        }
        return sqlSessionFactoryBean;
    }

}
```

- `@Configuration` 注解用于将该类注册为 Spring 配置类；
- `@MapperScan` 注解用于指定 Mapper 扫描的包；
- `SqlSessionFactoryBean` 用于创建 `SqlSessionFactory` 实例；
- `setDataSource` 方法用于指定数据源；
- `setMapperLocations` 方法用于指定 Mapper 文件的位置；
- `PathMatchingResourcePatternResolver` 和 `ResourcePatternResolver` 用于扫描 Mapper 文件。

7. 编写一个测试类，用于验证 MyBatis 的 CURD 操作：

```java
@SpringBootTest
class UserMapperTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    void testGetAllUsers() {
        List<User> userList = userMapper.getAllUsers();
        System.out.println(userList);
    }

    @Test
    void testGetUserById() {
        User user = userMapper.getUserById(1);
        System.out.println(user);
    }

    @Test
    void testInsertUser() {
        User user = new User();
        user.setName("Charlie");
        user.setAge(30);
        user.setEmail("charlie@example.com");
        userMapper.insertUser(user);
        System.out.println(user);
    }

    @Test
    void testUpdateUser() {
        User user = userMapper.getUserById(1);
        user.setName("Alice2");
        user.setAge(21);
        user.setEmail("alice2@example.com");
        userMapper.updateUser(user);
        System.out.println(userMapper.getUserById(1));
    }

    @Test
    void testDeleteUser() {
        userMapper.deleteUser(1);
        System.out.println(userMapper.getUserById(1));
    }

}
```

8. 运行测试类，观察控制台输出，可以看到查询结果和操作结果。

完整代码如下：

User.java：

```java
public class User {

    private Integer id;
    private String name;
    private Integer age;
    private String email;

    // getter and setter methods

}
```

UserMapper.java：

```java
public interface UserMapper {

    @Select("SELECT * FROM user")
    List<User> getAllUsers();

    @Select("SELECT * FROM user WHERE id = #{id}")
    User getUserById(Integer id);

    @Insert("INSERT INTO user (name, age, email) VALUES (#{name}, #{age}, #{email})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    void insertUser(User user);

    @Update("UPDATE user SET name = #{name}, age = #{age}, email = #{email} WHERE id = #{id}")
    void updateUser(User user);

    @Delete("DELETE FROM user WHERE id = #{id}")
    void deleteUser(Integer id);

}
```

UserMapper.xml：

```xml
<mapper namespace="com.example.mapper.UserMapper">

    <select id="getAllUsers" resultType="com.example.model.User">
        SELECT * FROM user
    </select>

    <select id="getUserById" resultType="com.example.model.User" parameterType="java.lang.Integer">
        SELECT * FROM user WHERE id = #{id}
    </select>

    <insert id="insertUser" parameterType="com.example.model.User" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO user (name, age, email) VALUES (#{name}, #{age}, #{email})
    </insert>

    <update id="updateUser" parameterType="com.example.model.User">
        UPDATE user SET name = #{name}, age = #{age}, email = #{email} WHERE id = #{id}
    </update>

    <delete id="deleteUser" parameterType="java.lang.Integer">
        DELETE FROM user WHERE id = #{id}
    </delete>

</mapper>
```

MyBatisConfig.java：

```java
@Configuration
@MapperScan("com.example.mapper")
public class MyBatisConfig {

    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource) {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        ResourcePatternResolver resourcePatternResolver = new PathMatchingResourcePatternResolver();
        try {
            sqlSessionFactoryBean.setMapperLocations(resourcePatternResolver.getResources("classpath*:com/example/mapper/*.xml"));
        } catch (IOException e) {
            e.printStackTrace();
        }
        return sqlSessionFactoryBean;
    }

}
```

UserMapperTest.java：

```java
@SpringBootTest
class UserMapperTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    void testGetAllUsers() {
        List<User> userList = userMapper.getAllUsers();
        System.out.println(userList);
    }

    @Test
    void testGetUserById() {
        User user = userMapper.getUserById(1);
        System.out.println(user);
    }

    @Test
    void testInsertUser() {
        User user = new User();
        user.setName("Charlie");
        user.setAge(30);
        user.setEmail("charlie@example.com");
        userMapper.insertUser(user);
        System.out.println(user);
    }

    @Test
    void testUpdateUser() {
        User user = userMapper.getUserById(1);
        user.setName("Alice2");
        user.setAge(21);
        user.setEmail("alice2@example.com");
        userMapper.updateUser(user);
        System.out.println(userMapper.getUserById(1));
    }

    @Test
    void testDeleteUser() {
        userMapper.deleteUser(1);
        System.out.println(userMapper.getUserById(1));
    }

}
```

