# @SpringBootApplication注解

## 启动方法:

````java
@Controller
@SpringBootApplication
public class PersonController {
    public static void main(String[] args) {
        SpringApplication.run(PersonController.class, args);
    }
````

***

## 作用

​	就是以下三个注解的总和

​	@Configuration:	定义一个配置类

​	@EnableAutoConfiguration:	springBoot根据你的jar包的依赖自动配置

​	@ComponentScan:	 自动扫描  并且装入bean容器。 

​	