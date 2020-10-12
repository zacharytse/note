[TOC]
# IOC

依赖注入。用于解耦，对于一个类来说，只要持有某个对象，而不用在初始化的时候去new这个对象。

IOC的一个简单实现

基本原理：通过xml文件，写入对象的相关信息(对象id，对象的包名以及该对象的属性)，然后读取该xml文件，并通过java反射技术构造出该对象，并将其保存到map中。接着某一个类只需要通过该对象id就可以获取到这个对象了

```java
//SimpleIOC.java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package com.xcq;

import java.io.FileInputStream;
import java.io.InputStream;
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.Map;
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.Node;
import org.w3c.dom.NodeList;

public class SimpleIOC {
    private Map<String, Object> beanMap = new HashMap();

    public SimpleIOC(String location) throws Exception {
        this.loadBeans(location);
    }

    private void loadBeans(String location) throws Exception {
        InputStream inputStream = new FileInputStream(location);
        DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
        DocumentBuilder documentBuilder = factory.newDocumentBuilder();
        Document doc = documentBuilder.parse(inputStream);
        Element root = doc.getDocumentElement();
        NodeList nodes = root.getChildNodes();

        for(int i = 0; i < nodes.getLength(); ++i) {
            Node node = nodes.item(i);
            if (node instanceof Element) {
                Element ele = (Element)node;
                String id = ele.getAttribute("id");
                String className = ele.getAttribute("class");
                Class beanClass = null;

                try {
                    beanClass = Class.forName(className);
                } catch (ClassNotFoundException var23) {
                    var23.printStackTrace();
                    return;
                }

                Object bean = beanClass.newInstance();
                NodeList propertyNodes = ele.getElementsByTagName("property");

                for(int j = 0; j < propertyNodes.getLength(); ++j) {
                    Node propertyNode = propertyNodes.item(j);
                    if (propertyNode instanceof Element) {
                        Element propertyElement = (Element)propertyNode;
                        String name = propertyElement.getAttribute("name");
                        String value = propertyElement.getAttribute("value");
                        Field declaredField = bean.getClass().getDeclaredField(name);
                        declaredField.setAccessible(true);
                        if (value != null && value.length() > 0) {
                            declaredField.set(bean, value);
                        } else {
                            String ref = propertyElement.getAttribute("ref");
                            if (ref == null || ref.length() == 0) {
                                throw new IllegalArgumentException("ref config error");
                            }

                            declaredField.set(bean, this.getBean(ref));
                        }

                        this.registerBean(id, bean);
                    }
                }
            }
        }

    }

    private void registerBean(String id, Object bean) {
        this.beanMap.put(id, bean);
    }

    Object getBean(String name) {
        Object bean = this.beanMap.get(name);
        if (bean == null) {
            throw new IllegalArgumentException("there is no bean with name " + name);
        } else {
            return bean;
        }
    }
}

```

```java
//Car.java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package com.xcq;

public class Car {
    private String name;
    private String length;
    private String width;
    private String height;
    private Wheel wheel;

    public Car() {
    }
}
```

```java
//Wheel.java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package com.xcq;

public class Wheel {
    private String brand;
    private String specification;

    public Wheel() {
    }
}

```

```java
//测试文件
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package com.xcq;

public class Main {
    public Main() {
    }

    public static void test_IOC() throws Exception {
        String location = SimpleIOC.class.getClassLoader().getResource("ioc.xml").getFile();
        SimpleIOC bf = new SimpleIOC(location);
        Wheel wheel = (Wheel)bf.getBean("wheel");
        System.out.println(wheel);
        Car car = (Car)bf.getBean("car");
        System.out.println(car);
    }

    public static void main(String[] args) throws Exception {
        test_IOC();
    }
}

```

```xml
//ioc.xml 配置文件
<beans>
    <bean id="wheel" class="com.titizz.simulation.toyspring.Wheel">
        <property name="brand" value="Michelin" />
        <property name="specification" value="265/60 R18" />
    </bean>

    <bean id="car" class="com.titizz.simulation.toyspring.Car">
        <property name="name" value="Mercedes Benz G 500"/>
        <property name="length" value="4717mm"/>
        <property name="width" value="1855mm"/>
        <property name="height" value="1949mm"/>
        <property name="wheel" ref="wheel"/>
    </bean>
</beans>
```

# AOP

AOP基于代理模式实现

## 代理模式

用一个类来代表另一个类的功能。主要用于为其他对象提供一种代理以控制对这个对象的访问。有些对象由于某些原因（比如对象创建开销很大，或者某些操作需要安全控制，或者需要进程外的访问），直接访问会给使用者或者系统结构带来很多麻烦，我们可以在访问此对象时加上一个对此对象的访问层。

优点： 1、职责清晰。 2、高扩展性。 3、智能化。

缺点： 1、由于在客户端和真实主题之间增加了代理对象，因此有些类型的代理模式可能会造成请求的处理速度变慢。 2、实现代理模式需要额外的工作，有些代理模式的实现非常复杂。

**使用场景：**按职责来划分，通常有以下使用场景： 1、远程代理。 2、虚拟代理。 3、Copy-on-Write 代理。 4、保护（Protect or Access）代理。 5、Cache代理。 6、防火墙（Firewall）代理。 7、同步化（Synchronization）代理。 8、智能引用（Smart Reference）代理。

**注意事项：** 1、和适配器模式的区别：适配器模式主要改变所考虑对象的接口，而代理模式不能改变所代理类的接口。 2、和装饰器模式的区别：装饰器模式为了增强功能，而代理模式是为了加以控制。

总结：借助一个proxy对另一个对象的访问进行控制

```java
//Image.java
package com.xcq.proxy;

public interface Image {
    void display();
}

```

```java
//RealImage.java
package com.xcq.proxy;

public class RealImage implements Image{
    private String fileName;

    public RealImage(String fileName){
        this.fileName = fileName;
        loadFromDisk(fileName);
    }

    private void loadFromDisk(String fileName) {
        System.out.println("Loading " + fileName);
    }

    @Override
    public void display() {
        System.out.println("Displaying " + fileName);
    }
}

```

```java
//ProxyImage.java
package com.xcq.proxy;

public class ProxyImage implements Image{
    private RealImage realImage;
    private String fileName;

    public ProxyImage(String fileName){
        this.fileName = fileName;
    }
    @Override
    public void display() {
        if(realImage == null){
            realImage = new RealImage(fileName);
        }
        realImage.display();
    }
}

```

```java
//test
public static void test_proxy(){
        Image image = new ProxyImage("test_10mb.jpg");

        // 图像将从磁盘加载
        image.display();
        System.out.println("");
        // 图像不需要从磁盘加载
        image.display();
    }
```

![代理模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/proxy_pattern_uml_diagram.jpg)

ProxyImage代理了RealImage,外部程序在调用RealImage时，需要通过ProxyImage去调用。这样避免了每次都要创建RealImage所造成的巨大开销

## AOP的一些简单概念

通知(Advice)：抽取的共性功能组成的代码逻辑

```
通知定义了要织入目标对象的逻辑，以及执行时机。
Spring 中对应了 5 种不同类型的通知：
· 前置通知（Before）：在目标方法执行前，执行通知
· 后置通知（After）：在目标方法执行后，执行通知，此时不关系目标方法返回的结果是什么
· 返回通知（After-returning）：在目标方法执行后，执行通知
· 异常通知（After-throwing）：在目标方法抛出异常后执行通知
· 环绕通知（Around）: 目标方法被通知包裹，通知在目标方法执行前和执行后都被会调用
```

切点(PointCut)

```
如果说通知定义了在何时执行通知，那么切点就定义了在何处执行通知。所以切点的作用就是
通过匹配规则查找合适的连接点（Joinpoint），AOP 会在这些连接点上织入通知。
```

切面（Aspect）

```
切面包含了通知和切点，通知和切点共同定义了切面是什么，在何时，何处执行切面逻辑。
```

连接点(JoinPoint)

```
指类中的方法
```

引入(Introduction)(不重要)

```
将共性功能的成员进行加入
```

目标对象(target object)

```
缺少完整逻辑功能代码的对象(包含切入点)
```

AOP代理(aop proxy)

```
将通知类中的方法植入到目标对象中
```

## 最简单的实现版本

采用静态代理的方式

Hello和ProxyHello同时继承IHello的接口，ProxyHello接管Hello的控制权

```java
//public interface IHello.java
public interface IHello {
    void sayHello(String str);
}
```

```java
//实际业务类 Hello.java
public class Hello implements IHello{
    @Override
    public void sayHello(String str) {
        System.out.println("hello " + str);
    }
}
```

```java
//代理类，控制业务类的访问 ProxyHello.java
public class ProxyHello implements IHello{
    private IHello hello;

    public ProxyHello(IHello hello){
        super();
        this.hello = hello;
    }

    @Override
    public void sayHello(String str) {
        Logger.start();
        hello.sayHello(str);
        Logger.end();
    }
}
```

```java
//通知类
public class Logger {
    public static void start(){
        System.out.println(new Date() + " say hello start...");
    }

    public static void end(){
        System.out.println(new Date() + " say hello end");
    }
}
```

```java
//测试代码
public class Test {
    public static void main(String[] args) {
        IHello hello = new ProxyHello(new Hello());
        hello.sayHello("yes");
    }
}
```

采用动态代理