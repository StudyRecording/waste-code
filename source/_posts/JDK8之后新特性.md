---
title: JDK8之后新特性
tags:
  - Java
  - 新特性
categories:
  - Java
excerpt: 查询其他网上的文章, 记录有误, 因此记录一下关于Java8之后新的语法
thumbnail: https://t.mwm.moe/ycy
cover: https://t.mwm.moe/pc
sticky: 1
date: 2023-05-22 21:19:52
---



## 静态工厂方法

都是不可变的方法，也就是一旦实例化后，不可再新增或删除数据

```java
// 创建只有一个值的可读list，底层不使用数组
static <E> List<E> of(E e1) {
   return new ImmutableCollections.List12<>(e1);
}
// 创建有多个值的可读list，底层使用数组
static <E> List<E> of(E e1, E e2, E e3) {
   return new ImmutableCollections.List12<>(e1, e2,e3);
}
// 创建单例长度为0的Set结合
static <E> Set<E> of() {
   return ImmutableCollections.emptySet();
}
static <E> Set<E> of(E e1) {
   return new ImmutableCollections.Set12<>(e1);
}

```





## 新的Stream Api

```java
// 如果第一个元素不符合条件则返回为空，
// 如果第一个元素符合条件，则从第一个元素开始继续向下获取，直到获取不符合条件元素所在的上一个元素为止
default Stream takeWhile(Predicate predicate);

// eg:
List.of(2, 1, 3, 6, 9, 5, 2, 4, 1, 0, 9).stream()
                .takeWhile(item -> item < 3)
                .forEach(System.out::println);
// 最终输出是2和1


```



```java
// 如果第一元素不符合条件则返回全部元素
// 如果第一个元素符合条件, 则从第一个不符合条件的元素开始，直到最后一个元素为止
default Stream dropWhile(Predicate predicate);


// eg:
List.of(2, 1, 3, 6, 9, 5, 2, 4, 1, 0, 9).stream()
                .dropWhile(item -> item < 3)
                .forEach(System.out::println);
// 最终输出为3、6、9、5、2、4、1、0、9
```



```java
// 根据表达式生成符合条件的值
static Stream iterate(T seed, Predicate hasNext, UnaryOperator next);

// eg:
Stream.iterate(1, item -> item < 5, item -> item * 2)
                .forEach(System.out::println);
// 最终输出为1、2、4
```



```java
// 使用空值创建空的Stream,避免空指针
static Stream ofNullable(T t);

// eg:
Stream.ofNullable(list)
                .forEach(System.out::println);
// 不输出，不报错

Stream.of(list)
                .forEach(System.out::println);
// 输出字符串"null"
```



## 接口中定义方法

```java
public interface TestInterface {
    private void openComputer() {
        System.out.println("打开电脑");
    }
    default void code() {
        openComputer();
        coding();
    }

    void coding();
}
```



## 类型推断

```java
var list = List.of(1,2,3,4,5);
for (var integer : list) {
    System.out.println(integer);
}
// 输出：1、2、3、4、5
```



## Http请求使用



### 同步请求

```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://www.baidu.com"))
        .GET()
        .build();
HttpResponse<String> send = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(send.body());
```



### 异步请求并保存到文件

```java
HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://www.baidu.com"))
        .GET()
        .build();
HttpClient.newHttpClient()
        .sendAsync(request, HttpResponse.BodyHandlers.ofFile(Paths.get("/Users/hpc/jdkTest.txt")))
        .join();
```



## `instanceof` 语法改进

```java
    public static void main(String[] args) {
        Object obj = get();
        if (obj instanceof Entity entity) {
            entity = (Entity) obj;
            System.out.println(entity);
        }
    }

    private static Object get() {
        return new Entity(1, "xx");
    }
```



## ## Switch和Yield

yield在switch中可以当做return的意思，只不过yield的作用于仅限于switch中

```java
    public static void main(String[] args) {
        System.out.println(get(8));
        // 输出"xxxxxxxxx"

        System.out.println(getByFormat(1L));
        // 输出"long 1"

        System.out.println(getByYield(4));
        // 输出"xxxx"
    }

    private static String get(Integer num) {
        return switch (num) {
            case 1 -> "x";
            case 2 -> "xx";
            case 3 -> "xxx";
            case 4,5,6 -> "xxxx";
            case 7,8,9 -> {
                StringBuilder sb = new StringBuilder();
                for (int i = 0; i < 9; i++) {
                    sb.append("x");
                }
                yield sb.toString();
            }
            default -> "";
        };
    }
    
    private static String getByYield(Integer num) {
        return switch (num) {
            case 1: yield "x";
            case 2: yield "xx";
            case 3: yield "xxx";
            case 4,5,6: yield "xxxx";
            case 7:
            case 8:
            case 9:
                StringBuilder sb = new StringBuilder();
                for (int i = 0; i < 9; i++) {
                    sb.append("x");
                }
                yield sb.toString();
            default: yield  "";
        };
    }

    private static String getByFormat(Object o) {
        return switch (o) {
            case Integer i -> String.format("int %d", i);
            case Long l -> String.format("long %d", l);
            default -> o.toString();
        };
    }
```
