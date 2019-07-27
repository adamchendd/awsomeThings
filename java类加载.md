## java类加载问题 ##
java类加载相关在大学的时候就有接触，但当时只想着做项目，对于java内存模型，jvm之类的基础知识基本没有了解。趁着工作之余，赶紧补补读书时遗留的功课。

当然这次再次了解java类加载相关知识也是源于最近学习dubbo相关知识，读到dubbo关于增强型SPI的使用，里面关于ServiceLoader的介绍，想着把这块捡起来，也算是为以后学习jvm开个头。

### 源码初探

    public class Main {
        public static void main(String[] args)  {
            System.out.println("hello world!");
        }
    }

对于这样一个简单的类，在ide中，直接点击run java application，控制台就能打印“hello world”这个结果。如果手动执行，控制台输入：

- **javac Main.java** 编译.java文件得到Main.class文件
- **java Main**加载Main.class文件，控制台打印“hello world”

类加载器将java字节码（.class文件）加载到jvm中，类加载器负责读取 Java 字节代码，并转换成 java.lang.Class类的一个实例。每个这样的实例用来表示一个 Java 类。通过此实例的newInstance()方法就可以创建出该类的一个对象**（这段抄的）**。今天网上查资料找文档主要还是了解了类加载过程，对于java源码编译过程留在后面详细了解。

##### 1. ClassLoader类
源码很多，挑简单的说，ClassLoader中与类加载相关的方法

|方法名|描述|
|:-----|:----|
|getParent()|获取类加载器的父类加载器|
|loadClass(String name)|加载名为name的类，返回java.lang.Class类的实例|
|findClass(String name)|查找名称为 name的类，返回的结果是 java.lang.Class类的实例|
|findLoadedClass(String name)|查找名称为 name的已经被加载过的类，返回的结果是 java.lang.Class类的实例|
|defineClass(String name, byte[] b, int off, int len)|把字节数组 b中的内容转换成 Java 类，返回的结果是 java.lang.Class类的实例。这个方法被声明为 final|
|resolveClass(Class<?> c)|链接指定的Java类|


##### 2. Launcher类
jdk中的ExtClassLoader和AppClassLoader定义在launcher类中，Launcher类构造方法伪代码：


    public Launcher() {
        Launcher.ExtClassLoader var1;
        try {
            var1 = Launcher.ExtClassLoader.getExtClassLoader();
        } catch (IOException var10) {
            throw new InternalError("Could not create extension class loader", var10);
        }

        try {
            this.loader = Launcher.AppClassLoader.getAppClassLoader(var1);
        } catch (IOException var9) {
            throw new InternalError("Could not create application class loader", var9);
        }

        Thread.currentThread().setContextClassLoader(this.loader);
        ……
        ……
    }

首先加载了两个类加载器ExtClassLoader和AppClassLoader（下节介绍）。下一步将系统类加载器设置为线程上下文类加载器，如果没有通过setContextClassLoader(ClassLoader cl)方法进行设置的话，线程将继承其父线程的上下文类加载器，Java应用运行的初始线程的上下文类加载器是系统类加载器。在线程中运行的代码可以通过此类加载器来加载类和资源。**（后续介绍）**

Launcher类在ClassLoader的getSystemClassLoader()方法中被调用：


    public static ClassLoader getSystemClassLoader() {
        initSystemClassLoader();
        if (scl == null) {
            return null;
        }
        ...
        ...
    }
    
    private static synchronized void initSystemClassLoader() {
        if (!sclSet) {
            if (scl != null)
                throw new IllegalStateException("recursive invocation");
            // 加载类加载器，得到的是系统类加载器
            sun.misc.Launcher l = sun.misc.Launcher.getLauncher();
            if (l != null) {
                Throwable oops = null;
                scl = l.getClassLoader();
                ...
                ...
            }
            sclSet = true;
        }
    }


### 类加载器结构

类加载器可以粗略分为系统提供和开发人员自定义两种：系统加载器有如下三类：
- 引导类加载器（Bootstrap Classloader）：引导类加载器是使用C++语言实现的，负责加载JVM虚拟机运行时所需的基本系统级别的类，如java.lang.String, java.lang.Object等等；
- 扩展类加载器（extensions class loader）：即ExtClassLoader，它用来加载 Java 的扩展库，Java 虚拟机的实现会提供一个扩展库目录，该类加载器在此目录里面查找并加载 Java 类；
- 系统类加载器（system class loader）：即AppClassLoader，它根据 Java 应用的类路径（CLASSPATH）来加载 Java 类。一般来说，Java 应用的类都是由它来完成加载的。可以通过 ClassLoader.getSystemClassLoader()来获取它。

引导类加载器没有父类加载器，可以认为是所有加载器的祖先类加载器，扩展类加载器是系统类加载器的父类加载器，系统类加载器是开发人员自定义加载器的父类加载器，可通过ClassLoader的getParent()方法获取到加载器对应的父类加载器。

    public static void main(String[] args) {

        System.out.println(ClassLoader.getSystemClassLoader());
        System.out.println(P11Util.class.getClassLoader());
        System.out.println(P11Util.class.getClassLoader().getParent());

    }

    console output：
    ------------------------------
    sun.misc.Launcher$AppClassLoader@18b4aac2--当前类加载器
    sun.misc.Launcher$ExtClassLoader@29453f44--P11Util是java扩展库中的类
    null--代表引导类加载器，不能被java引用
    ------------------------------


### 测试

    public static void main(String[] args) throws Exception {
        
        Launcher launcher = Launcher.getLauncher();
        ClassLoader classLoader = launcher.getClassLoader();
        Class<Loader> loaderClass = (Class<Loader>) classLoader.loadClass("packageName.className");
        Object instance = loaderClass.newInstance();
        Method method = loaderClass.getMethod("methodName", null);
        method.invoke(instance);

    }

测试代码中，首先通过Launcher类加载ExtClassLoader和AppClassLoader，利用loadClass方法加载已经被编译的.class文件，得到java.lang.Class类的实例，再通过newInstance()方法生成对应的对象，最后利用反射调用方法并执行。

## DONE！

文章中还有很多需要补充的知识点，后面继续！



