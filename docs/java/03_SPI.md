# ServiceLoader-SPI和类SPI机制的场景应用

## ServiceLoader

### 说明

原文："A simple service-provider loading facility."
解释：可以加载指定接口的实现类，最初主要用于加载厂商的实现类或插件使用。工程所依赖的jar包中在指定目录下添加实现类说明后，也可以被装载到JVM中；
理解：JDK内置的一种动态服务提供发现机制，运行时会装载具体的配置；除了spring之外，给我们提供一种原生的类装载和实例化的方式；


### ServiceLoader-代码实例

- Define Java Interface(Service)

    ```java
    public interface IHello {
        void sayHello();
    }
    ```

- Implement the Java Interface

    - Case1: HelloImpl1
        ```java
		package com.test.impl;
		import com.test.IHello;
		public class HelloImpl1 implements IHello {
		@Override
		public void sayHello() {
		      System.out.println("Hello1----HelloImpl1");
		   }
		}  
        ```
    - Case2: HelloImpl2
        ```java
        public class HelloImpl2 implements IHello {
            @Override
            public void sayHello() {
                System.out.println("Hello2----HelloImpl2");
            }
        }
		```

- Add service

在META-INF中建立services文件夹，并以service接口命名文件名，其内容为services实现类的全路径名，每一个类占一行，其配置内容如下：

```
com.hello.impl.HelloImpl1
com.hello.impl.HelloImpl2
```

- Call the service

```java

public class Test {
    public static void main(String[] args) {
        // case1:
        ServiceLoader<IHello> loader = ServiceLoader.load(IHello.class);

        // case2:
        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
        ServiceLoader<IHello> loader = ServiceLoader.load(IHello.class, classLoader);
        
        for(IHello hello:loader){
            System.out.println(hello.getClass());
            hello.sayHello();
        }
    }
}

```

### ServiceLoader总结

- 说明

SPI 是 Java 提供的一种服务加载方式，全名为 Service Provider Interface。
根据 Java 的 SPI 规范，我们可以定义一个服务接口，具体的实现由对应的实现者去提供，即服务提供者。
然后在使用的时候再根据 SPI 的规范去获取对应的服务提供者的服务实现。
通过 SPI 服务加载机制进行服务的注册和发现，可以有效的避免在代码中将具体的服务提供者写死。从而可以基于接口编程，实现模块间的解耦。

- 约定

1. 在 META-INF/services/ 目录中创建以接口全限定名命名的文件，该文件内容为API具体实现类的全限定名
2. 使用 ServiceLoader 类动态加载 META-INF 中的实现类
3. 如 SPI 的实现类为 Jar 则需要放在主程序 ClassPath 中
4. API 具体实现类必须有一个不带参数的构造方法

- 缺点

Java 中，可以通过 ServiceLoader 类比较方便的找到该类的所有子类实现。
META-INF/services 下的实现指定和实现子类实现完全可以和接口定义完全分开。
麻烦的地方: 每次都要手动创建实现指定文件，比较繁琐。


## AutoService


### 说明

解决手动编写 META-INF/services的问题。

```java
<dependencies>
    <dependency>
        <groupId>com.google.auto.service</groupId>
        <artifactId>auto-service</artifactId>
        <version>1.0-rc4</version>
    </dependency>
</dependencies>
```

### 实例说明


- 实现类说明

    ```java

     @AutoService(IHello.class)
 	public class HelloImpl1 implements IHello {
 	@Override
 	public void sayHello() {
 	      System.out.println("Hello1----HelloImpl1");
 	   }
 	}  
    ```

- 配置说明

删除 META-INF/services中相关定义。
