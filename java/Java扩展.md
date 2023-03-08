# 1. Java日志

## 1.1 日志介绍

* JDK自带日志(Java.util.logging)将日志等级分为7种, 可以设置日志级别控制日志输出

```java
java.util.logging.ConsoleHandler.level = INFO

private static Logger log = Logger.getLogger(Test.class.toString());

        // 级别依次升高，后面的日志级别会屏蔽之前的级别
        log.finest("finest");
        log.finer("finer");
        log.fine("fine");
        log.config("config");
        log.info("info");
        log.warning("warning");
        log.severe("server");


```

JDK自带日志相对于其他的日志框架，无论易用性，功能还是扩展性都要稍逊一筹，所以在商业系统中很少直接使用。

---



* 日志接口+日志实现

常用的日志**接口**(slf4j---静态绑定为日志实现类, common-logging---动态查找日志实现类)

常见的日志**实现**: logback, log4j

## 1.2 使用日志

先引入依赖

```xml
  <!--slf4j依赖-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.32</version>
        </dependency>
        <!-- logback 依赖 -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>
```

支持格式化字符串, 共5个日志级别

```java
private static final Logger LOGGER = LoggerFactory.getLogger(PmsBrandController.class);

logger.info("slf4j print info msg:{}",msg);
logger.debug("slf4j print debug msg:{}",msg);
```

可以配置日志级别和日志输出格式(以spring为例)

```yml
# spring日志设置
logging:
  level:
    root: debug #在不同package下可以设置不同的日志级别
```

# 2. Swagger-API文档生成

swagger是API文档生成/展示的工具, 用于前后端分离时后端api的文档生成

springfox时spring社区维护的一个项目, 用来集成swagger3

```xml
        <!--springfox-->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-boot-starter</artifactId>
            <version>3.0.0</version>
        </dependency>
```

## 2.1 SpringBoot 2.6.X 兼容问题

Swagger很久没有更新导致了与SpringMvc的兼容问题

### 2.1.1 问题原因

> Springboot 2.6.X 后, SpringMVC默认的路径匹配器由 `ant_path_matcher` 改为 `path_pattern_parser`(路径匹配器的使用者包括 `AbstractUrlHandlerMapping` ...)

* ant_path_matcher匹配规则

```
匹配1个字符; * 匹配0个或多个字符; ** 匹配路径中的0个或多个目录;
```

### 2.1.2 问题解决方法

1. 修改路径匹配规则

```yaml
spring:
  mvc:
    pathmatch:
      matching-strategy: ant_path_matcher # Swagger兼容Springboot-2.6.x
```

2. 在Swagger配置类中添加Bean

```java
// Swagger兼容Springboot-2.6.x
@Bean
public static BeanPostProcessor springfoxHandlerProviderBeanPostProcessor() {
    return new BeanPostProcessor() {

        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
            if (bean instanceof WebMvcRequestHandlerProvider || bean instanceof WebFluxRequestHandlerProvider) {
                customizeSpringfoxHandlerMappings(getHandlerMappings(bean));
            }
            return bean;
        }

        private <T extends RequestMappingInfoHandlerMapping> void customizeSpringfoxHandlerMappings(List<T> mappings) {
            List<T> copy = mappings.stream()
                    .filter(mapping -> mapping.getPatternParser() == null)
                    .collect(Collectors.toList());
            mappings.clear();
            mappings.addAll(copy);
        }

        @SuppressWarnings("unchecked")
        private List<RequestMappingInfoHandlerMapping> getHandlerMappings(Object bean) {
            try {
                Field field = ReflectionUtils.findField(bean.getClass(), "handlerMappings");
                field.setAccessible(true);
                return (List<RequestMappingInfoHandlerMapping>) field.get(bean);
            } catch (IllegalArgumentException | IllegalAccessException e) {
                throw new IllegalStateException(e);
            }
        }
    };
}

```

## 2.2 配置

配置类中设置API分组的元数据和摘要信息, 以及安全认证

```java
@Configuration
@EnableOpenApi//开启swagger
@Slf4j
public class SwaggerConfig {
    @Value("${swagger.enable}")
    private boolean isEnable;
    @Value("${jwt.httpHeaderKeyName}")
    private String jwtHttpHeaderKeyName;

    // Swagger兼容Springboot-2.6.x+
    @Bean
    public static BeanPostProcessor springfoxHandlerProviderBeanPostProcessor() {
        return new BeanPostProcessor() {

            @Override
            public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
                if (bean instanceof WebMvcRequestHandlerProvider || bean instanceof WebFluxRequestHandlerProvider) {
                    customizeSpringfoxHandlerMappings(getHandlerMappings(bean));
                }
                return bean;
            }

            private <T extends RequestMappingInfoHandlerMapping> void customizeSpringfoxHandlerMappings(List<T> mappings) {
                List<T> copy = mappings.stream().filter(mapping -> mapping.getPatternParser() == null).collect(Collectors.toList());
                mappings.clear();
                mappings.addAll(copy);
            }

            @SuppressWarnings("unchecked")
            private List<RequestMappingInfoHandlerMapping> getHandlerMappings(Object bean) {
                try {
                    Field field = ReflectionUtils.findField(bean.getClass(), "handlerMappings");
                    field.setAccessible(true);
                    return (List<RequestMappingInfoHandlerMapping>) field.get(bean);
                } catch (IllegalArgumentException | IllegalAccessException e) {
                    throw new IllegalStateException(e);
                }
            }
        };
    }

    //配置Api元信息
    private ApiInfo apiInfoOfMall() {
        return new ApiInfoBuilder().title("Mall APIs").version("1.0").description("测试环境地址：http://localhost:80").contact(new Contact("By 开发团队", "#", null)).build();
    }

    //配置单个分组的摘要信息
    @Bean
    public Docket mallApi() {
        return new Docket(DocumentationType.OAS_30).enable(isEnable).apiInfo(this.apiInfoOfMall())
                //选择需要被Swagger展示的API
                .select()
                .apis(RequestHandlerSelectors.basePackage("xyz.mall.controller"))
                //过滤出特定的请求路径
                .paths(PathSelectors.any())
                //结束选择
                .build().groupName("Mall-APIs")
                //访问控制
                .securitySchemes(securitySchemes()).securityContexts(securityContexts());
    }

    //SecurityScheme--使用什么方式进行安全认证
    private List<SecurityScheme> securitySchemes() {
        List<SecurityScheme> schemes = new ArrayList<>();
        //ApiKey(...) 第一个参数指定SecurityScheme名字, 便于SecurityContext引用, 并作为键名
        //ApiKey(...)  第三个参数为apiKey位置
        log.debug("jwtHttpHeaderKeyName: {}", jwtHttpHeaderKeyName);
        schemes.add(new ApiKey(jwtHttpHeaderKeyName, null, "header"));
        return schemes;
    }

    //配置 SecurityScheme.name和接口的映射关系
    private List<SecurityContext> securityContexts() {
        List<SecurityReference> references = new ArrayList<>();
        references.add(new SecurityReference(jwtHttpHeaderKeyName, new AuthorizationScope[0]));
        List<SecurityContext> contexts = new ArrayList<>();
        SecurityContextBuilder securityContextBuilder = SecurityContext.builder().securityReferences(references).operationSelector(o -> {
            return true;//分组下的所有接口都默认使用JWT认证
        });
        contexts.add(securityContextBuilder.build());
        return contexts;
    }
}
```

## 2.4 使用注解添加API信息

使用注解对相关的Controller类/实体类进行说明

**@Api**  相同tag的Controller里的ApiOperation会在同一个栏显示

```java
@RestController
@RequestMapping("/brand")
@Api(tags = {"品牌相关API"})
public class PmsBrandController {
//...
}
```

**@ApiOperation**

```java
    @ApiOperation(value = "创建品牌", notes = "开发中")
    public Result createBrand(@RequestBody PmsBrand pmsBrand) {
	//...
    }
```

**@ApiImplicitParam**(s) 请求参数

```java
    @RequestMapping(value = "/update/{id}")
    @ApiOperation("更新品牌信息")
    @ApiImplicitParams(
            @ApiImplicitParam(name = "id", value = "品牌id", required = true)
    )
```

**@ApiModelProperty**

实体类的属性描述

```java
public class Result<T>{
    @ApiModelProperty("返回的数据")
    T data;
    @ApiModelProperty("Http响应码")
    Integer code;
    @ApiModelProperty("错误信息")
    String msg;
}

public class PmsBrand implements Serializable {
    @ApiModelProperty("品牌id")
    private Long id;
}
```

## 2.3 MBG注释生成器和实体类介绍

使用MBG生成的实体类代码需要手工添加@ApiModel和@ApiModelProperty

过于繁琐, 可以自定义MBG的注释生成器代替手工添加.

注释生成器有默认实现类 `DefaultCommentGenerator` , 继承并重写字段注释方法和文件注释方法即可

```java
public class ApiPropertyGenerator extends DefaultCommentGenerator {
    private static final boolean enabled = true;
    private static final String[] EXCLUDED_CLASS_PATTERNS = new String[]{"Example", "Provider"};
    private static final String API_MODEL_PROPERTY_FULL_CLASS_NAME="io.swagger.annotations.ApiModelProperty";

    /**
     * 给 Model Field 添加 @ApiModelProperty注解
     */
    @Override
    public void addFieldComment(Field field, IntrospectedTable introspectedTable,
                                IntrospectedColumn introspectedColumn) {
        String remarks = introspectedColumn.getRemarks();
        if(enabled && StringUtility.stringHasValue(remarks)){
            //数据库中"需要转为'
            if(remarks.contains("\"")){
                remarks = remarks.replace("\"","'");
            }
            //给model的字段添加swagger注解
            field.addJavaDocLine("@ApiModelProperty(value = \""+remarks+"\")");
        }
    }

    @Override
    public void addJavaFileComment(CompilationUnit compilationUnit) {
        super.addJavaFileComment(compilationUnit);
        //实体类需要添加文件注释 import io.swagger.annotations.ApiModelProperty;
        if(!compilationUnit.isJavaInterface()){
            for (String excludedClassPattern : EXCLUDED_CLASS_PATTERNS) {
                if (compilationUnit.getType().getShortName().contains(excludedClassPattern))
                    return;
            }
            compilationUnit.addImportedType(new FullyQualifiedJavaType(API_MODEL_PROPERTY_FULL_CLASS_NAME));
        }
    }
}
```

记得在MBG配置文件中使用自定义注释生成器

```xml
        <!--注释生成器-->
        <commentGenerator type="xyz.mall.mbg.ApiPropertyGenerator">
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
```

# 3. Java定时任务

## 3.1 定时任务介绍

定时任务使用业务场景

```
1. 购买商品后, 如果1个小时后不付款, 那么就将商品退还
2. 每隔24小时重置游戏的体力
```

定时任务实现方案

```
1. 线程等待实现: Thread.sleep(timeInterval)后执行任务
2. JDK自带的Timer API (缺点: 基于绝对时间, 操作系统时间改变会造成影响;  出现异常会终止任务, 没有异常处理)
3. JDK自带ScheduledExecutorService(基于线程池)
4. 单机开源框架: Quartz, SpringTask
5. 分布式任务调度框架: XXL-JOB Elastic-Job等框架
```

## 3.2 Spring Task

特点: 轻量简单/不支持分布式/单线程阻塞

### 3.2.1 cron表达式

cron表达式组成

```
Cron表达式是一个字符串，字符串以5或6个空格隔开，分为6或7个域，每一个域代表一个含义
Seconds Minutes Hours DayofMonth Month DayofWeek (Year)
```

每个域常用的特殊字符

```
：表示匹配任意值
- 表示范围
start/interval 表示起始时间开始触发，然后每隔固定时间触发一次 
? 在DayofMonth和DayofWeek中，用于匹配任意值
L 在DayofMonth和DayofWeek表示最后一个
A,B 枚举AB 
```

示例

```
0/20 * * * * ?  表示每20秒执行任务
0 15 10 ? * MON-FRI  表示周一到周五每天上午10:15执行任务
0 15 10 ? 6L 2002-2006  表示2002-2006年的每个月的最后一个星期五上午10:15执行任务
```

### 3.2.2 spring task的使用

springboot3+自带spring task依赖

配置类

```java
@Configuration
@EnableScheduling//开启spring task
public class SpringTaskConfig {

}
```

任务类

```java
@Component//需要注册为bean
@Slf4j
public class OrderTimeOutCancelTask {
    @Scheduled(cron = "0/10 * * ? * ?")
    private void cancelTimeoutOrder(){
        log.debug("每10s取消过期订单");
        //业务代码...
    }
}
```
