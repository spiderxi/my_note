# 1. 服务注册中心

## 1.1 Eureka

### 1.1.1  什么是微服务?

### 1.1.2 Eureka-Client

### 1.1.3 Eureka-Server

### 1.1.4 Eureka集群

### 1.1.5 Eureka常用配置

## 1.2 CAP理论

**CAP理论说的就是：一个分布式系统，不可能同时做到这三点**

* Consistency（一致性）: 对访问分布式系统的客户端的一种承诺：要么我给您返回一个错误，要么我给你返回绝对一致的最新数据，不难看出，其强调的是数据正确。
* Availability（可用性）:  对访问分布式系统的客户端的另一种承诺：我一定会给您返回数据，不会给你返回错误，但不保证数据最新，强调的是不出错。
* Partition tolerance（分区容忍性）: 对访问分布式系统的客户端的再一种承诺：我会一直运行，不管我的内部出现何种数据同步问题，强调的是不挂掉。

![1673930705210](image/spring-cloud/1673930705210.png)

## 1.3 各大服务注册中心对比

# 2. 负载均衡

## 2.1 Nginx和Ribbon

* Ribbon(本地负载均衡组件):   在调用微服务的时候、会在[eureka](https://so.csdn.net/so/search?q=eureka&spm=1001.2101.3001.7020)注册中心上获取注册信息服务列表，获取到之后，缓存在jvm本地，使用本地实现RPC(remote procedure call)进行调用，即是本地实现负载均衡.
* Nginx(服务器负载均衡): 客户端所有请求都会交给nginx，然后由nginx实现转发请求，即[负载均衡](https://so.csdn.net/so/search?q=%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1&spm=1001.2101.3001.7020)是由服务端实现.

Ribbon本地负载均衡适合微服务rpc远程调用, nginx服务负载均衡适合于针对Tomcat服务器端.

## 2.2 Ribbon使用

### 2.2.1 RestTemplate使用

```
Eureka-Client间接依赖了Ribbon 
       <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
```

在springboot配置类中注册 `RestTemplate(spring对Http-Client的封装)`

加上注解 `@LoadBalanced`开启负载均衡

```
@Configuration
public class ApplicationContextConfig {
    @Bean
    @LoadBalanced //让这个RestTemplate在请求时拥有客户端负载均衡的能力
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

使用RestTemplate进行RPC

```
    @Resource
    private RestTemplate restTemplate;

    @GetMapping("/payment/get/{id}")
    public CommonResult<Payment> getPayment(@PathVariable("id") Long id) {
	//getObject方法发起get请求, 返回CommonResult.class对应的JSON(需要实现Serializable)
	//PAYMENT_URL为http://host:port, 当使用服务注册中心时, 可以使用http://服务名称
        return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id,CommonResult.class);
    }

    @GetMapping("/payment/getForEntity/{id}")
    public CommonResult<Payment> getPayment2(@PathVariable("id") Long id) {
	//getForEntity发起get请求, 返回包含响应头等信息的ResponseEntity类
        ResponseEntity<CommonResult> entity = restTemplate.getForEntity(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);

        if (entity.getStatusCode().is2xxSuccessful()) {
            return entity.getBody();
        } else {
            return new CommonResult<>(444, "操作失败");
        }
    }
```

### 2.2.2负载均衡自定义策略

使用RestTemplate进行RPC的时候，会自动使用负载均衡策略，它内部会在RestTemplate中加入LoadBalancerInterceptor这个拦截器，这个拦截器的作用就是使用负载均衡。**默认情况下会采用轮询策略**，通过往Spring容器中注入一个IRule接口的实现类, 可以改变Ribbon默认的负载均衡策略。

```
@Bean
public IRule ribbonRule() {
    return new BestAvailableRule();
}
```

Ribbon依赖包自带的IRule继承图如下

![1673933886644](image/spring-cloud/1673933886644.png)

# 3. 服务调用

## 3.1什么是OpenFeign

Feign是一个声明式的Web服务客户端（Web服务客户端就是Http客户端），让编写Web服务客户端变得非常容易，只需创建一个接口并在接口上添加注解即可。Java当中常见的Http客户端有很多，除了Feign，类似的还有Apache 的 `HttpClient` 以及 `OKHttp3`，还有SpringBoot自带的 `RestTemplate`这些都是Java当中常用的HTTP 请求工具。

OpenFeign是Spring Cloud 在Feign的基础上支持了SpringMVC的注解，如@RequesMapping等等。OpenFeign的@FeignClient可以解析SpringMVC的@RequestMapping注解下的接口，并通过**动态代理的方式产生实现类**，实现类中做负载均衡并调用其他服务。

## 3.1 OpenFeign基本使用

导入依赖

```
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

在启动类中激活Feign

```
@SpringBootApplication
@EnableFeignClients
public class CloudConsumerFeignOrder80Application {
    public static void main(String[] args) {
        SpringApplication.run(CloudConsumerFeignOrder80Application.class, args);
    }
}
```

写一个接口作为FeignClient, 绑定这个接口访问的微服务名/host:port

```
@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentFeignService {
    @GetMapping(value = "/payment/get/{id}")
    CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);

    @GetMapping(value = "/payment/feign/timeout")
    String paymentFeignTimeout();
}
```

Feign-client的使用

```
    @Resource
    private PaymentFeignService paymentFeignService;

    @GetMapping(value = "/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id) {
        return paymentFeignService.getPaymentById(id);
    }

    @GetMapping(value = "/payment/feign/timeout")
    public String paymentFeignTimeout() {
        // OpenFeign客户端一般默认等待1秒钟
        return paymentFeignService.paymentFeignTimeout();
    }
```


# 4. 服务熔断和降级

## 4.1 Hystrix

@EnableHystrix

@HystrixCommand

@HystrixProperty
