# 1. Mybatis入门

Maven依赖

```xml
<!-- mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.6</version>
</dependency>

<!-- mybatis-spring-boot-starter -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.3</version>
</dependency>

<!-- mysql-connector-java -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.28</version>
</dependency>

<!-- druid -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.8</version>
</dependency>



```

## 1.1 核心配置文件

mybatis有两种配置方式, 一种使用xml文件, 一种使用注解. xml配置文件分为一个核心配置文件和多个mapper配置文件.

核心配置文件约定名称为 `mybatis-config.xml`, 核心配置文件用于指明数据源信息和mapper映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<!--    1. 导入 .properties 数据源配置文件中的变量-->
    <properties resource="db.properties"></properties>

<!--    2. 配置各种环境下的数据源和事务管理器-->
    <environments default="development">

<!--     开发环境-->
        <environment id="development">
<!--            事务管理器-->
            <transactionManager type="JDBC"></transactionManager>
<!--            数据源-->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"></property>
                <property name="url" value="${jdbc.url}"></property>
                <property name="username" value="${jdbc.username}"></property>
                <property name="password" value="${jdbc.password}"></property>
            </dataSource>
        </environment>
    </environments>


    <!--3. 导入Mapper文件-->
    <mappers>
        <mapper resource="UserMapper.xml"></mapper>
    </mappers>
</configuration>
```

## 1.2 Mapper配置文件和Mapper接口

一个XxxMapper接口对应一个实体类Xxx, 接口包含CRUD方法, mybatis根据Mapper配置文件(约定名称为XxxMapper.xml)来代理接口类实现对应的CRUD方法.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace为XxxMapper的全类名-->
<mapper namespace="xzy.wsh.mapper.UserMapper">
    <!--id为XxxMapper接口中的方法名, 标签内容为方法对应的sql-->
    <insert id="addUser">
        insert into mall VALUES (1, "spider");
    </insert>
</mapper>
```

## 1.3 获取数据库连接对象SqlSession

在mybatis核心配置文件中导入Mapper配置文件后, 通过配置文件创建 `SqlSessionFactory`

对象, 获取 `SqlSession`连接对象

```java
InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
SqlSession sqlSession = sqlSessionFactory.openSession();
```

之后获取XxxMapper接口的代理类并使用

```
UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
userMapper.getUser();
sqlSession.commit();
sqlSession.close();
```

## 1.4 Mapper配置文件中的CUD写法

CUD方法返回值均为int, 所以只需要关心SQL语句中如何使用Mapper方法的参数

**方法无参数时**

```xml
<insert id="addUser">
    insert into mall VALUES (1, "spider");
</insert>
```

**方法有基本类型/String参数时**,  会按照arg0, arg1为键依次对应参数列表中的参数, 形成Map

`#{键名}预编译占位符, 防止SQL注入`

`${键名}字符串拼接`

```xml
    <!--带基本类型参数写法:  int updateUser(int id, String name); -->
    <update id="updateUser">
        update t_user set  uname='king', pwd='123456'
        where id=#{arg0} and uname=#{arg1}
    </update>
```

@Param指定参数键名

```java
int updateUser(@Param("id") int id, @Param("uname") String uname);
```

**实体类型/Map类型参数**

实体类型的一个属性名看作一个键名, Map直接使用键名

```xml
    <!--int deleteUser(User user);-->
    <!--int deleteUser(Map<String, Object> user);-->
    <delete id="deleteUser">
        delete from t_user when id=#{id}
    </delete>
```

## 1.5 查询结果集的映射

将SQL结果每一行映射成特定类型时, 常用map或实体类的全类名

```xml
    <!--map, int等常用类-->
    <select id="getUser" resultType="map">
        select * from t_user;
    </select>

    <!--实体类-->
    <select id="getUser" resultType="xyz.wsh.mall.entity.User">
        select * from t_user;
    </select>
```

返回结果有多行时使用List

```java
    List<User> getUsers();
    List<Map<String, Object>> getUsers();
```

## 1.6 一些特殊的使用场景

插入一条数据到包含自增主键表格中后, 需要获取插入后自增主键

```xml
    <!--int addUser(User user);-->
    <!--插入数据后的对应自增主键会赋值给对象的id属性-->
    <insert id="addUser" useGeneratedKeys="true" keyProperty="id">
        insert into user(id, name) VALUES (null, #{uname});
    </insert>
```

模糊查询使用concat函数

```xml
    <select id="getUser" resultType="xyz.wsh.mall.entity.User">
        select * from t_user where name like concat("%", #{pattern}, "%");
    </select>
```

## 1.7 自定义字段-属性映射

使用ResultMap自定义一个映射

```xml
<mapper namespace="xzy.wsh.mapper.UserMapper">
    <resultMap id="user-map" type="xyz.wsh.mall.entity.User">
        <!--主键-->
        <id property="id" column="c_id"></id>
        <!--基本类型字段-->
        <result property="name" column="c_name"></result>
        <!--多个字段名对应一个引用类型的属性-->
        <association property="dept" javaType="xyz.wsh.mall.entity.Department">
            <id property="did" column="c_did"></id>
            <result property="dname" column="c_dname"></result>
        </association>
        <!--多个字段名映射成集合类型属性中的一个引用类型-->
        <collection property="deps" ofType="xyz.wsh.mall.entity.User">
            <id property="id" column="c_id"></id>
            <result property="name" column="c_name"></result>
        </collection>
    </resultMap>
    <select id="getUser" resultMap="user-map">
        select * from t_user where name like concat("%", #{pattern}, "%");
    </select>
</mapper>
```

## 1.8 动态SQL语句 *

动态Sql语句可以避免JDBC中的拼接SQL语句

常用标签if, where, trim, foreach, **了解即可**, 只有复杂需求的时候用得到

```xml
    <select id="getUser" resultMap="user-map">
        select * from t_user
        <where>
            <if test="uname != null and uname != ''">
                uname=#{uname}
            </if>
            <if test="age >= 0">
                and age=#{age}
            </if>
        </where>
    </select>
```

## 1.9 Mybatis缓存机制

一级缓存: SqlSession对于一条查询语句的查询结果会进行缓存, 当查询语句对应的表发生修改时, 缓存失效

二级缓存: SqlSessionFactory会在其产生的SqlSession关闭后收集一级缓存

一级缓存强制开启, 二级缓存默认开启, , 二级缓存可以在核心配置文件中配置, 查询时优先使用二级缓存

# 2. Mybatis插件

## 2.1 Mybatis generator

```
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.4.0</version>
</dependency>
```

[mybatis generator](https://mybatis.org/generator/index.html)是一个工具, 只需要一个配置文件就可以通过数据库自动生成Mapper代码

### 2.1.1 MBG配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <context id="mysql_mall" targetRuntime="MyBatis3" defaultModelType="flat">
        <property name="javaFileEncoding" value="UTF-8"/>

        <!--mbg增强插件-->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"/>
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin"/>

        <!--注释生成器-->
        <commentGenerator>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>

        <!-- jdbc datasource -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai&useSSL=false&allowPublicKeyRetrieval=true"
                        userId="root"
                        password="123456">
            <!--MySQL由于没有catalog和schema, 需要添加-->
            <property name="nullCatalogMeansCurrent" value="true" />
        </jdbcConnection>


        <!-- entity类生成器 -->
        <javaModelGenerator targetPackage="xyz.mall.entity" targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <!-- mapper.xml生成器 -->
        <sqlMapGenerator targetPackage="mapper"  targetProject=".\src\main\resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <!-- mapper类生成器 -->
        <javaClientGenerator type="ANNOTATEDMAPPER" targetPackage="xyz.mall.mapper"  targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <!--表, 表名为%表示所有表-->
        <table tableName="%">
        </table>
    </context>
</generatorConfiguration>
```

### 2.1.2 Java代码启动MBG

```java
        List<String> warnings = new ArrayList<String>();
        InputStream configFileIs = Generator.class.getResourceAsStream("/mbg-config.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFileIs);
        DefaultShellCallback callback = new DefaultShellCallback(true);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        myBatisGenerator.generate(null);
        warnings.forEach(log::warn);
        log.info("mbg completed!");
```

### 2.1.3 使用Mapper

```
        UserExample userExample = new UserExample();
        userExample.createCriteria().andNameEqualTo(name).andPasswordEqualTo(password);
        List<User> users = userMapper.selectByExample(userExample);
```

## 2.2 PageHelper

```xml
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>1.4.1</version>
        </dependency>
```

### 2.2.1 使用方法

```java
//Mapper接口方式的调用，推荐这种使用方式。
PageHelper.startPage(1, 10);
//紧跟的第一个mybatis查询会使用分页查询
List<Country> list = countryMapper.selectIf(1);

//实际返回的list类型是Page<T>, 是ArrayList<T>的子类

//用PageInfo对结果进行包装
PageInfo page = new PageInfo(list);
//测试PageInfo全部属性
//PageInfo包含了非常全面的分页属性
assertEquals(1, page.getPageNum());
```

# 3. Springboot整合

```
<!-- mybatis-spring-boot-starter -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.3</version>
</dependency>
```

在application.yml中添加数据源

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    username: root
    password: 123456
    url: jdbc:mysql://localhost:3306/exp6?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai&useSSL=false

```

**@MapperScan**   扫描某个包下的Mapper接口，Mybatis将生成Mapper接口的代理类, 然后交给Spring进行管理

```java
@Configuration
@MapperScan("seu.db.exp6.mapper")
public class MybatisConfig {
}

```

**@Mapper** 显式地告诉Mybatis这是一个Mapper接口(然后做和MapperScan的相同的事)

查询注解, 用于Mapper接口的方法, 添加这些注解就不用写XXXMapper.xml

**@Insert @Delete @Update @Select** @MapKey、@Options、@SelelctKey、**@Param**、
@InsertProvider、@UpdateProvider、@DeleteProvider、@SelectProvider

结果映射, 用于Mapper接口的方法,  添加这些注解就不用写resultMap@Results、@Result、**@ResultMap、@ResultType**、@ConstructorArgs、@Arg、@One、@Many、
@TypeDiscriminator、@Case
