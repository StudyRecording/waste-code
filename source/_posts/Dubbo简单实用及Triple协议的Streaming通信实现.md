---
title: Dubbo简单实用及Triple协议的Streaming通信实现
tags:
  - Dubbo
  - Triple
  - 实践
categories:
  - 分布式
excerpt: Dubbo的简单使用，Triple的协议和实现
thumbnail: https://pan.mwm.moe/f/PJgTM/33.WEBP
cover: https://t.mwm.moe/pc
sticky: 1
date: 2022-08-28 19:36:30
---

## 文章概览
- 项目模块  
    1. common模块——实现实体类以及声明暴露的api接口
    2. provider模块——暴露的api接口的业务实现
    3. consumer模块——请求接口的实现，将会待用暴露的api接口
- GITHUB: [Dubbo的简单使用以及Triple协议的Streaming通信的实现](https://github.com/StudyRecording/dubbo-sample)
- 官方文档: [Triple协议](https://dubbo.apache.org/zh/docs3-v2/java-sdk/reference-manual/protocol/triple/)
- Blog目的: 记录实现过程及出现的问题

## Dubbo的简单使用
1. 在common模块中定义实体类User
2. 在common模块中声明暴露出的接口，实现接口UserService
    ```java
    public interface UserService {

        /**
        * 获取用户信息
        * @param name
        * @return
        */
        User getUserInfo(String name);
    }
    ```

3. 在provider和consumer模块中引入相关依赖
    ```xml
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-spring-boot-starter</artifactId>
        <version>3.0.7</version>
    </dependency>

    <!--   下面这个包必须引用，服务注册到zookeeper中使用，之前没有引用这个包，结果应用起不来         -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-registry-zookeeper</artifactId>
        <version>3.0.7</version>
    </dependency>

    <dependency>
        <groupId>com.sample</groupId>
        <artifactId>common</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>
    ```
4. 在provider和consumer模块中创建application.yml文件并编写相关配置
    ```yaml
    server:
        port: 8082 # 这里填写端口号，provider和consumer不同，
    spring:
        application:
            name: consumer

    dubbo:
        protocol:
            name: dubbo # 选择通信协议
            port: -1
        registry:
            id: zk-zookeeper
            address: zookeeper://127.0.0.1:2181
    ```

5. 在provider和consumer中编写启动类，这里以consumer模块为例，这里要加上EnableDubbo注解
    ```java
    @SpringBootApplication
    @EnableDubbo
    public class ConsumerApplication {

        public static void main(String[] args) {
            SpringApplication.run(ConsumerApplication.class, args);
        }
    }
    ```
5. 在provider中对UserService进行实现
    ```java
    // 这里注意使用注解@DubboService，时dubbo中的Service注解，主要在对外提供服务的实现类上
    @DubboService
    public class UserServiceImpl implements UserService {
        @Override
        public User getUserInfo(String name) {
            User user = new User();
            user.setName("dubbo");
            user.setAge(12);
            return user;
        }

    }

    ```   

5. 在consumer中实现请求接口, 引用provider模块暴露出的接口要使用DubboReference注解
    ```java
    @RestController
    @RequestMapping("/user")
    public class UserController {

        @DubboReference
        private UserService userService;

        @GetMapping("/info")
        public User getUserInfo() {
            return userService.getUserInfo("xxx");
        }

    
    }
    ```

编写完成代码后，启动provider和consumer模块，然后通过Postman工具调用接口，发现可以正常使用就完成了

## Triple协议的Streaming通信实现

Triple协议的Stream通信主要分为三种：服务端流、客户端流、双向流
### 应用场景
- 接口需要发送大量数据，无法被放到一次请求中，需要分批次发送
- 流式场景，数据需要按照发送顺序处理, 数据本身是没有确定边界的
- 推送类场景，多个消息在同一个调用的上下文中被发送和处理
### 流的语义保证(优点)
- 提供消息边界，可以方便的对消息进行单独处理
- 严格有序，发送端的顺序和接收端的顺序是一致的
- 全双工，发送不需要等待
- 支持取消和超时

### Streaming流通信实现
#### 服务端流(SERVER_STREAM)请求流程
<img src="https://studyrecording.github.io/waste-code/images/Dubbo%E7%AE%80%E5%8D%95%E5%AE%9E%E7%94%A8%E5%8F%8ATriple%E5%8D%8F%E8%AE%AE%E7%9A%84Streaming%E9%80%9A%E4%BF%A1%E5%AE%9E%E7%8E%B0/image-1661691742134.png" alt="服务端流(SERVER_STREAM)" width="400">

#### 服务端流(SERVER_STREAM)的Java实现

1. 在provider和consumer模块中添加相关依赖
    ```xml
    <dependency>
        <groupId>com.google.protobuf</groupId>
        <artifactId>protobuf-java</artifactId>
    </dependency>
    ```

2. 修改provider和consumer模块中的相关配置
    ```yaml
    dubbo: #此处仅截取需要变更的配置，其他配置默认为有原有的就行
        protocol:
            name: tri # 修改dubbo的通信协议，当然triple协议同样支持之前的dubbo的简单使用
    ```
3. 在common模块的UserService中声明相关api接口
    ```java
    /**
     * 服务端流
     * @param name
     * @param response
     */
    void sayHelloServerStream(String name, StreamObserver<String> response)
        throws InterruptedException;
    ```
4. 在provider模块中实现相关功能
    ```java
    //StreamObserver是接收消息的观察者，
    //在onNext方法调用后，consumer模块中的消费者会获取相关的数据，
    //当onCompleted方法调用后，consumer模块进行最后的处理后，整个服务端流才会结束
    @Override
    public void sayHelloServerStream(String name, StreamObserver<String> response)
        throws InterruptedException {
    
        response.onNext("Hallo, " + name);
        
        // 这里延迟10s，主要测试，provider模块接收数据会不会有10s的延时
        Thread.sleep(10 * 1000);

        response.onNext("Hallo, " + name + ", 第二次");

        response.onCompleted();
    }
    ```

5. 在consumer模块编写请求方法
    ```java
    
    /**
     * 测试服务端流
     * @param name
     * @return
     * @throws InterruptedException
     */
    @GetMapping("/sayHallo/{name}")
    public List<String> sayHallo(@PathVariable("name") String name) throws InterruptedException {

        List<String> list = new ArrayList<>();
        userService.sayHelloServerStream(name, new StreamObserver<String>() {
            
            // 每次provider模块调用一次onNext时，该方法会执行一次
            @Override
            public void onNext(String data) {
                System.out.println("onNext:" + data);
                list.add(data);
            }

            @Override
            public void onError(Throwable throwable) {
                System.out.println("报错了");
            }
            // 当provider模块的onCompleted方法调用后，执行该方法
            @Override
            public void onCompleted() {
                System.out.println("结束");
            }
        });
        return list;
    }
    ```
#### 客户端流(CLIENT_STREAM)请求流程
<img src="https://studyrecording.github.io/waste-code/images/Dubbo%E7%AE%80%E5%8D%95%E5%AE%9E%E7%94%A8%E5%8F%8ATriple%E5%8D%8F%E8%AE%AE%E7%9A%84Streaming%E9%80%9A%E4%BF%A1%E5%AE%9E%E7%8E%B0/image-1661692170936.png" alt="客户端流(CLIENT_STREAM)" width="400">

#### 双向流(BIDIRECTIONAL_STREAM)请求流程
<img src="https://studyrecording.github.io/waste-code/images/Dubbo%E7%AE%80%E5%8D%95%E5%AE%9E%E7%94%A8%E5%8F%8ATriple%E5%8D%8F%E8%AE%AE%E7%9A%84Streaming%E9%80%9A%E4%BF%A1%E5%AE%9E%E7%8E%B0/image-1661692263642.png" alt="双向流(BIDIRECTIONAL_STREAM)" width="400">

#### 客户端流(CLIENT_STREAM)/双向流(BIDIRECTIONAL_STREAM)的Java实现
1. 客户端流和双向流在Java中的实现方式是同一种
2. 引用pom和修改配置与服务端流相同
3. 在common模块中声明相关接口
    ```java
    /**
     * 客户端流/双向流, 这里返回的StreamObserver类里的处理实在provider模块中实现，
     * 而参数StreamObserver则是在consumer模块中实现，虽然是consumer调用该方法
     * @param response
     * @return
     */
    StreamObserver<String> sayHelloStream(StreamObserver<String> response);
    ```

4. 在provider模块中实现相关方法
    ```java
    @Override
    public StreamObserver<String> sayHelloStream(StreamObserver<String> response) {
        return new StreamObserver<String>() {
            @Override
            public void onNext(String data) {
                System.out.println("服务端请求参数:" + data);
                response.onNext("Hello, " + data);
            }

            @Override
            public void onError(Throwable throwable) {

            }

            @Override
            public void onCompleted() {
                System.out.println("provider关闭");
                response.onCompleted();
            }
        };
    }
    ```
5. 在consumer模块中实现方法的调用
    ```java
    
    @PostMapping("/sayHallo")
    public List<String> sayHallo(@RequestBody List<String> names) {
        List<String> list = new ArrayList<>();
        StreamObserver<String> request = userService.sayHelloStream(new StreamObserver<String>() {
            @Override
            public void onNext(String data) {
                System.out.println("说了啥？" + data);
                list.add(data);
            }

            @Override
            public void onError(Throwable throwable) {

            }

            @Override
            public void onCompleted() {
                System.out.println("结束了");
            }
        });

        // 上面定义了StreamObserver并调用了方法后，在下边通过onNext方法调用发送请求
        names.forEach(item -> {
            request.onNext(item);
            try {
                Thread.sleep(10 * 1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        });
        request.onCompleted();

        return list;
    }
    ```    





