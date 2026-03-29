## Nacos

在bin包下

输入

```cmd
startup.cmd -m standalone
就可以启动nacos服务
在http://localhost:8848/nacos下
```

## @EnableDiscoveryClient

在主类上卖弄使用这个注解 可以使服务发现

都是能够让**注册中心**能够发现，扫描到该服务。

## DiscoveryClient discoveryClient;

在discoveryClient可以得到

```java
@SpringBootTest
public class ProductApplicationTest {
    @Autowired
    DiscoveryClient discoveryClient;//nacos
    @Autowired
    NacosDiscoveryClient nacosDiscoveryClient;//二者效果一样，这个依赖nacos 只能由nacos使用

    @Test
    public void discoveryClientTest(){
        List<String> services = discoveryClient.getServices();//注册中心的微服务
        for (String service : services) {
            System.out.println("service = " + service);
            List<ServiceInstance> instances = discoveryClient.getInstances(service);//获得服务的实例List
            for (ServiceInstance instance : instances) {
                System.out.println("instance.getHost() = " + instance.getHost());//获得ip地址 和端口列表
                System.out.println("instance.getPort() = " + instance.getPort());
            }
        }
    }

    @Test
    public void nacosDiscoveryClientTest(){
        List<String> services = nacosDiscoveryClient.getServices();
        for (String service : services) {
            System.out.println("service = " + service);
            List<ServiceInstance> instances = nacosDiscoveryClient.getInstances(service);
            for (ServiceInstance instance : instances) {
                System.out.println("instance.getHost() = " + instance.getHost());
                System.out.println("instance.getPort() = " + instance.getPort());
            }
        }
    }
}

```





## @RestController

@RestController下的方法默认返回的是数据格式

 简而言之，`@RestController` 适用于构建 RESTful 风格的 API，其中每个方法的返回值会直接序列化为** JSON 或 XML** 数据并发送给客户端。

，@Controller注解标注的类下面的方法默认返回的就是以视图为格式



## RestTemplate

```java
restTemplate.getForObject(url, Product.class);
```

