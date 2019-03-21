

### 一、什么是 Java 的反射机制？

反射（Reflection）是 Java 的高级特性之一，是框架实现的基础，定义：Java 反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为 Java 语言的反射机制。

一般而言，当用户使用一个类的时候，应该获取这个类，而后通过这个类实例化对象，但是使用反射则可以相反的通过对象获取类中的信息。

通俗的讲反射就是可以在程序运行的时候动态装载类，查看类的信息，生成对象，或操作生成的对象。它允许运行中的 Java 程序获取自身的信息，自己能看到自己，就像照镜子一样。

### 二、Java 反射机制常见方法介绍

#### 1. Java 反射实现的关键点之 Class（字节码文件对象）

Class 类的实例表示正在运行的 Java 应用程序中的类和接口。JVM 中有 N 多的实例，每个类的实例都有 Class 对象。（包括基本数据类型）

Class 没有公共构造方法。Class 对象是在加载类时由 Java 虚拟机以及通过调用类加载器中的 defineClass 方法自动构造的。也就是这不需要我们自己去处理创建，JVM 已经帮我们创建好了。

如果知道一个实例，那么可以通过实例的“getClass()”方法获得运行实例的 Class（该类型的字节码文件对象），如果你知道一个类型，那么你也可以使用“.class”的方法获得运行实例的 Class。

**方法 2~17 都是类 Class 的方法。**

java.lang.Class 继承自 java.lang.Object。

#### 2. getName() 方法

String getName()；返回此 Member 表示的底层成员或构造方法的简单名称。

#### 3. forName() 方法

```
public static Class <？> forName(String className)
                        throws ClassNotFoundException

```

返回与带有给定字符串名的类或接口相关联的 Class 对象。

> 参数：className - **所需类的完全限定名**

#### 4. getSuperclass() 方法

```
public Class<？ super T> getSuperclass()

```

返回：此对象所表示的类的超类。

#### 5. getInterfaces() 方法

```
public Class< ? >[] getInterfaces()

```

返回：该类所实现的接口的一个数组

#### 6. getConstructors() 方法

```
public Constructor< ? >[] getConstructors() throws SecurityException

```

返回：表示此类公共构造方法的 Constructor 对象数组

#### 7. newInstance() 方法

```
public T newInstance() throws InstantiationException, IllegalAccessException

```

创建此 Class 对象所表示的类的一个新实例。如同用一个带有一个空参数列表的 new 表达式实例化该类。如果该类尚未初始化，则初始化这个类。

返回：此对象所表示的类的一个新分配的实例。

#### 8. getDeclaredConstructors() 方法

```
public Constructor<?>[] getDeclaredConstructors() throws SecurityException

```

返回 Constructor 对象的一个数组，这些对象反映此 Class 对象表示的类声明的所有构造方法。它们是公共、保护、默认（包）访问和私有构造方法。返回数组中的元素没有排序，也没有任何特定的顺序。如果该类存在一个默认构造方法，则它包含在返回的数组中。 如果此 Class 对象表示一个接口、一个基本类型、一个数组类或 void，则此方法返回一个长度为 0 的数组。

返回：表示此类所有已声明的构造方法的 Constructor 对象的数组

#### 9. getFields() 方法

```
public Field[] getFields()  throws SecurityException`

```

返回：表示公共字段的 Field 对象的数组

#### 10. getDeclaredFields() 方法

```
public Field[] getDeclaredFields() throws SecurityException

```

返回：表示此类所有已声明字段的 Field 对象的数组,与 getFields() 方法的区别是 getFields() 方法只返回公有属性，而 getDeclaredFields() 方法返回所有属性。

#### 11. getMethods() 方法

```
public Method[] getMethods() throws SecurityException

```

返回：表示此类中公共方法的 Method 对象的数组

#### 12. getDeclaredMethods() 方法

```
public Method[] getDeclaredMethods()  throws SecurityException

```

返回：表示此类所有声明方法的 Method 对象的数组

#### 13. getClassLoader() 方法

```
public ClassLoader getClassLoader()

```

返回该类的类加载器。有些实现可能使用 null 来表示引导类加载器。如果该类由引导类加载器加载，则此方法在这类实现中将返回 null。如果存在安全管理器，并且调用者的类加载器不是 null，也不同于或是请求其类加载器的类的类加载器的祖先，则此方法通过 RuntimePermission("getClassLoader") 权限调用此安全管理器checkPermission 方法，以确保可以访问该类的类加载器。如果此对象表示一个基本类型或 void，则返回 null。

返回：加载此对象所表示的类或接口的类加载器。

#### 14. getInterfaces() 方法

```
public Class<?>[] getInterfaces()

```

返回：该类所实现的接口的一个数组

#### 15. getComponentType() 方法

```
public Class<?> getComponentType()

```

返回表示数组组件类型的 Class。如果此类不表示数组类，则此方法返回 null。

返回：如果此类是数组，则返回表示此类组件类型的 Class

#### 16. getResourceAsStream() 方法

```
public InputStream getResourceAsStream(String name)

```

查找具有给定名称的资源。查找与给定类相关的资源的规则是通过定义类的 class loader 实现的。此方法委托此对象的类加载器。如果此对象通过引导类加载器加载，则此方法将委托给 ClassLoader.getSystemResourceAsStream(java.lang.String)。

> 参数：name - 所需资源的名称

返回：一个 InputStream 对象；如果找不到带有该名称的资源，则返回 null

#### 17. isInstance() 方法

```
public boolean isInstance(Object obj)

```

> 参数：obj - 要检查的对象

返回：如果 obj 是此类的实例，则返回 true

一般地，我们用 instanceof 关键字来判断是否为某个类的实例。同时我们也可以借助反射中 Class 对象的 isInstance() 方法来判断是否为某个类的实例，它是一个 Native 方法。

#### 18. invoke() 方法

```
public Object invoke(Object obj, Object... args)  throws IllegalAccessException,  IllegalArgumentException,  InvocationTargetException

```

参数：

- obj - 从中调用底层方法的对象
- args - 用于方法调用的参数

返回：使用参数 args 在 obj 上指派该对象所表示方法的结果

```
java.lang.reflect 
类 Method
java.lang.Object
  继承者 java.lang.reflect.AccessibleObject
      继承者 java.lang.reflect.Method

```

#### 19. newProxyInstance() 方法

```
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException

```

返回一个指定接口的代理类实例，该接口可以将方法调用指派到指定的调用处理程序。此方法相当于：

```
Proxy.getProxyClass(loader, interfaces). getConstructor(new Class[] { InvocationHandler.class }). newInstance(new Object[] { handler });

```

参数：

- loader - 定义代理类的类加载器
- interfaces - 代理类要实现的接口列表
- h - 指派方法调用的调用处理程序

返回：一个带有代理类的指定调用处理程序的代理实例，它由指定的类加载器定义，并实现指定的接口

```
java.lang.reflect 
类 Proxy
java.lang.Object
  继承者 java.lang.reflect.Proxy

```

### 三、Java 反射机制常见用法举例

#### 1. 使用 Java 反射通过一个对象获取某个类的完整包名和类名

通过实例的 getClass() 方法获得运行实例的字节码文件对象，然后通过 getName() 方法获得类的完整包名和类名。

```
    public static void test1(){
        ReflectTest testReflect = new ReflectTest();
        System.out.println(testReflect.getClass().getName());
    }
    //结果:com.channelsoft.ssm.controller.ReflectTest

```

#### 2. 使用 Java 反射获取 Class 对象（三种方式）

- 方式1：通过 Class 类的静态方法获取 Class 类对象 (**推荐**)
- 方式2：因为所有类都继承 Object 类。因而可通过调用 Object 类中的 getClass 方法来获取
- 方式3：任何数据类型（包括基本数据类型）都有一个“静态”的 class 属性

```
    public static void test2() throws Exception{
            Class<?> class1 = null;
            Class<?> class2 = null;
            Class<?> class3 = null;
            class1 =  Class.forName("com.channelsoft.ssm.controller.ReflectTest");
            //这一new 产生一个ReflectTest对象，一个Class对象。 
            class2 = new ReflectTest().getClass();
            class3 = ReflectTest.class;
            System.out.println("类名称 1：   " + class1.getName());
            System.out.println("类名称 2：  " + class2.getName());
            System.out.println("类名称 3：  " + class3.getName());
    }

```

运行结果：

![enter image description here](http://images.gitbook.cn/0c0e1320-75cb-11e8-a2a8-5d5dd52f3532)

#### 3. 使用 Java 反射获取一个对象的父类和实现的接口

```
public static void test3() throws ClassNotFoundException{
        Class<?> clazz = Class.forName("com.channelsoft.ssm.controller.ReflectTest");
        // 取得父类
        Class<?> parentClass = clazz.getSuperclass();
        System.out.println("父类为：" + parentClass.getName());
        // 获取所有的接口
        Class<?> intes[] = clazz.getInterfaces();
        System.out.println("实现的接口有：");
        for (int i = 0; i < intes.length; i++) {
            System.out.println((i + 1) + "：" + intes[i].getName());
        }
}

```

> 注：ReflectTest 实现了 Serializable 接口

运行结果:

![enter image description here](http://images.gitbook.cn/69e84240-75cb-11e8-b54c-0d200546486d)

#### 4. 使用 Java 反射获取某个类的全部构造函数并调用私有构造方法

Student 类：

```
package com.channelsoft.ssm.domain;

public class Student {

 public Student(){  
        System.out.println("调用了公有、无参构造方法。。。");  
    }  

    public Student(char name){  
        System.out.println("姓名：" + name);  
    }  

    public Student(String name ,int age){  
        System.out.println("姓名："+name+"年龄："+ age);
    }  

    //受保护的构造方法  
    protected Student(boolean b){  
        System.out.println("受保护的构造方法 b = " + b);  
    }  

    //私有构造方法  
    private Student(int age){  
        System.out.println("私有的构造方法，年龄："+ age);  
    }  
}

```

获取 Student 类的全部构造函数，并使用 Class 对象的 newInstance() 方法来创建 Class 对象对应类 Student 的实例来调用私有构造方法。

```
package com.channelsoft.ssm.controller;

import java.lang.reflect.Constructor;

public class Constructors {
    public static void main(String[] args) throws Exception {  
        //1.加载Class对象  
        Class clazz = Class.forName("com.channelsoft.ssm.domain.Student");      
        //2.获取所有公有构造方法  
        System.out.println("**********************所有公有构造方法****************************");  
        Constructor[] conArray = clazz.getConstructors();  
        for(Constructor c : conArray){  
            System.out.println(c);  
        }  

        System.out.println("************所有的构造方法(包括：私有、受保护、默认、公有)***************");  
        conArray = clazz.getDeclaredConstructors();  
        for(Constructor c : conArray){  
            System.out.println(c);  
        }  

        System.out.println("*****************获取公有、无参的构造方法***************************"); 
        //1、因为是无参的构造方法所以类型是一个null,不写也可以：这里需要的是一个参数的***类型***，
        //2、返回的是描述这个无参构造函数的类对象。  
        Constructor con = clazz.getConstructor(null);  
        System.out.println("con = " + con);  
        //调用构造方法  
        Object obj = con.newInstance();  
        System.out.println("obj = " + obj);  

        System.out.println("******************获取私有构造方法，并调用**************************");  
        con = clazz.getDeclaredConstructor(int.class);  
        System.out.println(con);  
        //调用构造方法  
        con.setAccessible(true);//暴力访问(忽略掉访问修饰符)  
        obj = con.newInstance(26);  
    }  
}

```

运行结果：

![enter image description here](http://images.gitbook.cn/841b5790-75cd-11e8-a2a8-5d5dd52f3532)

#### 5. 使用 Java 反射获取某个类的全部属性

Student 类：

```
package com.channelsoft.ssm.domain;

public class Student {

    public String name;  
    protected int age;  
    char sex;  
    private String phoneNum;

    @Override  
    public String toString() {  
        return "Student [name=" + name + ", age=" + age + ", sex=" + sex  
                + ", phoneNum=" + phoneNum + "]";  
    }  
}

```

获取 Student 类的全部属性并调用：

```
package com.channelsoft.ssm.controller;

import java.lang.reflect.Field;

import com.channelsoft.ssm.domain.Student;

public class Fields {
    public static void main(String[] args) throws Exception {  
        //1.获取Class对象  
        Class stuClass = Class.forName("com.channelsoft.ssm.domain.Student");  
        //2.获取字段  
        System.out.println("************************获取所有公有的字段**************************************");  
        Field[] fieldArray = stuClass.getFields();  
        for(Field f : fieldArray){  
            System.out.println(f);  
        }  
        System.out.println();
        System.out.println("************************获取所有的字段(包括私有、受保护、默认的)**********************");  
        fieldArray = stuClass.getDeclaredFields();  
        for(Field f : fieldArray){  
            System.out.println(f);  
        }  
        System.out.println();
        System.out.println("*************************获取公有字段并调用**************************************");  
        Field f = stuClass.getField("name");  
        System.out.println(f);  
        //获取一个对象  
        Object obj = stuClass.getConstructor().newInstance();  
        f.set(obj, "laughitover");
        //验证  
        Student stu = (Student)obj;  
        System.out.println("验证姓名：" + stu.name);  

        System.out.println();
        System.out.println("**************************获取私有字段并调用**************************************");  
        f = stuClass.getDeclaredField("phoneNum");  
        System.out.println(f);  
        f.setAccessible(true);//暴力反射，解除私有限定  
        f.set(obj, "13812345678");  
        System.out.println("验证电话：" + stu);  
    }  
}

```

运行结果：

![enter image description here](http://images.gitbook.cn/8c2af0d0-75cd-11e8-bdae-ebb21c9b14c8)

#### 6. 使用 Java 反射获取某个类的全部方法

```
package com.channelsoft.ssm.domain;

public class Student {    
    public void show1(String s){  
        System.out.println("调用了：公有的，String参数的show1(): s = " + s);  
    }  
    protected void show2(){  
        System.out.println("调用了：受保护的，无参的show2()");  
    }  
    void show3(){  
        System.out.println("调用了：默认的，无参的show3()");  
    }  
    private String show4(int age){  
        System.out.println("调用了，私有的，并且有返回值的，int参数的show4(): age = " + age);  
        return "abcd";  
    } 
}

```

获取某个类的全部方法并且调用共有方法 show1() 和私有方法 show4()：

```
public static void test6() throws Exception {
        //1.获取Class对象
        Class stuClass = Class.forName("fanshe.method.Student");
        //2.获取所有公有方法
        System.out.println("***************获取所有的”公有“方法*******************");
        stuClass.getMethods();
        Method[] methodArray = stuClass.getMethods();
        for(Method m : methodArray){
            System.out.println(m);
        }
        System.out.println("***************获取所有的方法，包括私有的*******************");
        methodArray = stuClass.getDeclaredMethods();
        for(Method m : methodArray){
            System.out.println(m);
        }
        System.out.println("***************获取公有的show1()方法*******************");
        Method m = stuClass.getMethod("show1", String.class);
        System.out.println(m);
        //实例化一个Student对象
        Object obj = stuClass.getConstructor().newInstance();
        m.invoke(obj, "laughitover");

        System.out.println("***************获取私有的show4()方法******************");
        m = stuClass.getDeclaredMethod("show4", int.class);
        System.out.println(m);
        m.setAccessible(true);//解除私有限定
        Object result = m.invoke(obj, 20);//需要两个参数，一个是要调用的对象（获取有反射），一个是实参
        System.out.println("返回值：" + result);
}

```

运行结果：

![enter image description here](http://images.gitbook.cn/9161c790-75cd-11e8-b54c-0d200546486d)

#### 7. 使用 Java 反射调用某个类的方法

Java 反射获取 Class 对象并使用invoke()方法调用方法 reflect1() 和 reflect2()：

```
public static void test7() throws Exception{
        Class<?> clazz = Class.forName("com.channelsoft.ssm.controller.ReflectTest");
        Method method = clazz.getMethod("reflect1");
        method.invoke(clazz.newInstance());
        method = clazz.getMethod("reflect2", int.class, String.class);
        method.invoke(clazz.newInstance(), 20, "张三");
    }

    public void reflect1() {
        System.out.println("Java 反射机制 - 调用某个类的方法1.");
    }
    public void reflect2(int age, String name) {
        System.out.println("Java 反射机制 - 调用某个类的方法2.");
        System.out.println("age -> " + age + ". name -> " + name);
    }

```

运行结果:

![enter image description here](http://images.gitbook.cn/96def620-75cd-11e8-bdae-ebb21c9b14c8)

#### 8. 反射机制的动态代理实例

如果想获取实现 Subject() 接口的其它实例，只需新建类实现接口 Subject()，使用 MyInvocationHandler.bind() 方法获取即可实现调用。

```
 public static void test8(){
        MyInvocationHandler demo = new MyInvocationHandler();
        Subject sub = (Subject) demo.bind(new RealSubject());
        String info = sub.say("语文", 90);
        System.out.println(info);
    }

```

如果想要完成动态代理，首先需要定义一个 InvocationHandler 接口的子类，完成代理的具体操作。

```
package com.channelsoft.ssm.controller;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class MyInvocationHandler implements InvocationHandler {
    private Object obj = null;

    public Object bind(Object obj) {
        this.obj = obj;
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), this);
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object temp = method.invoke(this.obj, args);
        return temp;
    }
}

```

```
package com.channelsoft.ssm.service;

public interface Subject {
    public String say(String name, int score);
}

```

```
package com.channelsoft.ssm.controller;

import com.channelsoft.ssm.service.Subject;

public class RealSubject implements Subject {
    public String say(String name, int score) {
            return name + "  " + score;
    }
}

```

运行结果:

![enter image description here](http://images.gitbook.cn/b90de080-75cd-11e8-bdae-ebb21c9b14c8)

#### 9. 通过反射越过泛型检查

泛型作用在编译期，编译过后泛型擦除（消失掉）。所以是可以通过反射越过泛型检查的。

```
    public static void test10() throws Exception{
        ArrayList<Integer> list = new ArrayList<Integer>();
        list.add(100);
        list.add(200);
        Method method = list.getClass().getMethod("add", Object.class);
        method.invoke(list, "Java反射机制实例。");
        //遍历集合  
        for(Object obj : list){  
            System.out.println(obj);  
        }
    }

```

运行结果:

![enter image description here](http://images.gitbook.cn/befa99c0-75cd-11e8-8eb5-c76a9039c594)

#### 10. 通过反射机制获得数组信息并修改数组的大小和值

通过反射机制分别修改int和String类型的数组的大小并修改int数组的第一个值。

```
 public static void test12() throws Exception{
         int[] temp = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
         int[] newTemp = (int[]) arrayInc(temp, 15);
         print(newTemp);
         Array.set(newTemp, 0, 100);
         System.out.println("修改之后数组第一个元素为： " + Array.get(newTemp, 0));
         print(newTemp);
         String[] atr = { "a", "b", "c" };
         String[] str1 = (String[]) arrayInc(atr, 8);
         print(str1);
    }
    // 修改数组大小
    public static Object arrayInc(Object obj, int len) {
        Class<?> arr = obj.getClass().getComponentType();
        Object newArr = Array.newInstance(arr, len);
        int co = Array.getLength(obj);
        System.arraycopy(obj, 0, newArr, 0, co);
        return newArr;
    }
    // 打印
    public static void print(Object obj) {
        Class<?> c = obj.getClass();
        if (!c.isArray()) {
            return;
        }
        Class<?> arr = obj.getClass().getComponentType();
        System.out.println("数组类型： " + arr.getName());
        System.out.println("数组长度为： " + Array.getLength(obj));
        for (int i = 0; i < Array.getLength(obj); i++) {
            System.out.print(Array.get(obj, i) + " ");
        }
        System.out.println();
    }

```

运行结果:

![enter image description here](http://images.gitbook.cn/c58e1a50-75cd-11e8-b54c-0d200546486d)

#### 11. 将反射机制应用于工厂模式

对于普通的工厂模式当我们在添加一个子类的时候，就需要对应的修改工厂类。 当我们添加很多的子类的时候，会很麻烦。

```
package com.channelsoft.ssm.service;

public interface Fruit {
    public abstract void eat();
}

```

```
package com.channelsoft.ssm.controller;

import com.channelsoft.ssm.service.Fruit;

public class Orange implements Fruit {
    public void eat() {
        System.out.println("Orange");
    }
}

```

```
package com.channelsoft.ssm.controller;

import com.channelsoft.ssm.service.Fruit;

public class Apple implements Fruit {
    public void eat() {
        System.out.println("Apple");
    }
}

```

```
package com.channelsoft.ssm.controller;

import com.channelsoft.ssm.service.Fruit;

public class Factory {
    public static Fruit getInstance(String ClassName) {
        Fruit f = null;
        try {
            f = (Fruit) Class.forName(ClassName).newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return f;
    }
}

```

```
    public static void test13(String fruit) throws Exception{
        Fruit f = Factory.getInstance(fruit);
        if (f != null) {
            f.eat();
        }
   }

```

还可以通过加载配置文件的方式：

```
    public static String getValue(String key) throws IOException{  
        Properties pro = new Properties();//获取配置文件的对象  
        InputStream in=new Demo().getClass().getResourceAsStream("/pro.txt");
        pro.load(in);//将流加载到配置文件对象中  
        in.close();  
        return pro.getProperty(key);//返回根据key获取的value值  
    } 

```

pro.txt 内容如下：

```
className = com.channelsoft.ssm.domain.Apple

```

运行结果:

![enter image description here](http://images.gitbook.cn/ca6f5700-75cd-11e8-8eb5-c76a9039c594)

#### 12. 通过 Java 反射机制获取 ClassLoader 类加载器

![enter image description here](http://images.gitbook.cn/c97ff150-75f6-11e8-a2a8-5d5dd52f3532)

```
package com.channelsoft.ssm.controller;

public class ReflectTest{

    public static void main(String[] args) throws Exception {
        testClassLoader();
    }

    public static void testClassLoader() throws ClassNotFoundException {  
        //1、获取一个系统的类加载器  
        ClassLoader classLoader = ClassLoader.getSystemClassLoader();  
        System.out.println("系统的类加载器-->" + classLoader);  

        //2、获取系统类加载器的父类加载器(扩展类加载器（extensions classLoader）)  
        classLoader = classLoader.getParent();  
        System.out.println("扩展类加载器-->" + classLoader);  

        //3、获取扩展类加载器的父类加载器  
        //输出为Null,引导类加载器无法被Java程序直接引用  
        classLoader = classLoader.getParent();  
        System.out.println("启动类加载器-->" + classLoader);  

        //4、测试当前类由哪个类加载器进行加载 ,结果就是系统的类加载器  
        classLoader = Class.forName("com.channelsoft.ssm.controller.ReflectTest").getClassLoader();  
        System.out.println("当前类由哪个类加载器进行加载-->"+classLoader);  

        //5、测试JDK提供的Object类由哪个类加载器负责加载的  
        classLoader = Class.forName("java.lang.Object").getClassLoader();  
        System.out.println("JDK提供的Object类由哪个类加载器加载-->" + classLoader);  
    }  
}

```

![enter image description here](http://images.gitbook.cn/fd565be0-75f6-11e8-a2a8-5d5dd52f3532)

### 四、Java 反射机制相关面试题

**1. 反射创建类实例的三种方式是什么？**

> 答案参见：本篇文章：Java 反射机制常见用法举例2

**2. 反射中，Class.forName 和 ClassLoader 区别？**

> Class.forName(className)方法，其实调用的方法是 `Class.forName(className,true,classloader);` 第 2 个参数表示在 loadClass 后必须初始化。根据 JVM 加载类的知识，我们知道在执行过此方法后，目标对象的 static 块代码已经被执行，static 参数也已经被初始化。
>
> 而 ClassLoader.loadClass(className) 方法，实际调用的方法是 `ClassLoader.loadClass(className,false);` 第 2 个参数表示目标对象被装载后不进行链接，这就意味着不会去执行该类静态块中间的内容。
>
> 例如，在 JDBC 使用中，常看到这样的用法，`Class.forName("com.mysql.jdbc.Driver")` 如果换成了 `getClass().getClassLoader().loadClass("com.mysql.jdbc.Driver")`，就不行。根据 com.mysql.jdbc.Driver 的源代码，Driver 在 static 块中会注册自己到 java.sql.DriverManager。而 static 块就是在 Class 的初始化中被执行。所以这个地方就只能用 Class.forName(className)。
>
> **com.mysql.jdbc.Driver 的源代码：**
>
> ```
> // Register ourselves with the DriverManager
> static {
>     try {
>         java.sql.DriverManager.registerDriver(new Driver());
>     } catch (SQLException E) {
>         throw new RuntimeException("Can't register driver!");
>     }
> } 
>
> ```
>
> **Java 类的加载过程:**
>
> 在 Java 中，类装载器把一个类装入 Java 虚拟机中，要经过三个步骤来完成：装载、链接和初始化，其中链接又可以分成校验、准备和解析三步：
>
> - 装载：查找和导入类或接口的二进制数据；
> - 链接：执行下面的校验、准备和解析步骤，其中解析步骤是可以选择的；
> - 校验：检查导入类或接口的二进制数据的正确性；
> - 准备：给类的静态变量分配并初始化存储空间；
> - 解析：将符号引用转成直接引用；
> - 初始化：激活类的静态变量的初始化 Java 代码和静态 Java 代码块。

**3. 描述动态代理的几种实现方式，分别说出相应的优缺点**

JDK 动态代理和 CGLIB 动态代理。JDK 动态代理是由 Java 内部的反射机制来实现的，CGLIB 动态代理底层则是借助 ASM 框架来实现的，实现了无反射机制进行代理，利用空间来换取了时间，代理效率高于 JDK 。总的来说，反射机制在生成类的过程中比较高效，而 ASM 在生成类之后的相关执行过程中比较高效（可以通过将 ASM 生成的类进行缓存，这样解决 ASM 生成类过程低效问题）。还有一点必须注意：JDK 动态代理的应用前提，**必须是目标类基于统一的接口**。如果没有上述前提，JDK 动态代理不能应用。由此可以看出，JDK 动态代理有一定的局限性，CGLIB 这种第三方类库实现的动态代理应用更加广泛，且在效率上更有优势。

### 五、总结

很多人认为反射在实际的 Java 开发应用中并不广泛，其实不然。我们常用的 JDBC 中有一行代码：

```
Class.forName('com.MySQL.jdbc.Driver.class').newInstance();

```

这里用到的就是 Java 反射机制，除此之外很多框架也都用到反射机制。

实际上反射机制就是非常规模式，如果说方法的调用是 Java 正确的打开方式，那反射机制就是 Java 的后门。为什么会有这个后门呢？这涉及到了静态和动态的概念：

- 静态编译：在编译时确定类型，绑定对象
- 动态编译：运行时确定类型，绑定对象

两者的区别在于，动态编译可以最大程度地支持多态，多态最大的意义在于降低类的耦合性，而 Java 语言确是静态编译（如：int a=3；要指定类型），Java 反射很大程度上弥补了这一不足：解耦以及提高代码的灵活性，所以 Java 也被称为是准动态语言。

尽管反射有诸多优点，但是反射也有缺点：

1. 由于反射会额外消耗一定的系统资源，因此，反射操作的效率要比那些非反射操作低得多。
2. 由于反射允许代码执行一些在正常情况下不被允许的操作（比如访问私有的属性和方法），所以使用反射可能会导致意料之外的副作用（代码有功能上的错误）。所以能用其它方式实现的就尽量不去用反射。

