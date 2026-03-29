

## springbootapplication爆红

```xml
原因就是在pom。xml中没有导入相关的依赖 就是
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
```

## 一切的起源@SpringBootApplication

```java
@SpringBootConfiguration -》@Configuration &&@Component
@EnableAutoConfiguration 自动导入包
@ComponentScan 扫描当前主启动类同级的包
```



## @ConfigurationProperties

![QQ_1758351896772](F:\记录\QQ_1758351896772.png)

这个@ConfigurationProperties 可以让类与ymal中的同名类进行赋值 当然 ```前提是prefix 要相同```
