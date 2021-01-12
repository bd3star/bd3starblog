---
title: '看完此文,别说你不懂IoC是什么?'
date: 2021-01-11 22:41:14
categories: 重学Spring
tags: [Spring, SpringIoC]
description: 这是显示在首页的概述，正文内容均会被隐藏。
---

> 盆友圈都在晒十八岁，
> 真羡慕你们可以发18岁的照片，我还要再等两年才能发。哎~惆怅..

## 前言

很多小伙伴会对Spring的IoC有些疑惑,什么是控制反转?
我为何要使用IoC 把控制权交给容器这样对我有什么好处?
书上只讲理论, 我现在都不能体会Spring的IoC用于不用相比有什么好处,
由spring托管有什么好处呢？
我现在感觉用spring 的set注入就是看起来代码NB点，
完全不理解到底有什么优势啊……  
基于如上让你彻底搞定理解IoC,从此不再困惑

废话不多说(正文依旧有),我们开始吧!


## 原生Servlet

我们先从原生Servlet时代的三层架构(MVC)开始切入

### 创建maven工程 引入依赖

``` xml
 <dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>

```

### 创建测试

``` java
@WebServlet(urlPatterns = "/poXing")
public class PoXingServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        response.getWriter().println("PoXingServlet run ......");
    }

}
```

### 浏览器访问
访问路径  
<http://localhost:8080/spring-framework-ioc-learning/poXing>   
(http://ip:端口号/[context-path(默认是工程名称可以自行修改)]/uri)

浏览器可输出 `PoXingServlet run ......`  此时已完成基础Servlet

### 进一步优化 添加 Service 和 Dao


> 这里为演示简单 并未引入输入库相关依赖 故此处 Dao 只是模拟有数据库的存在,
不深入

工程目录创建以下类与接口

![目录截图](./01.png)

三层架构中组件与依赖应如下图所示

![关系图](./02.png)


#### 添加Dao

``` java
public interface IDemoDao {
    List<String> findAll();
}


```

``` java
public class DemoDaoImpl implements IDemoDao {
    @Override
    public List<String> findAll() {
        // 此处为演示方便不连接数据库 模拟返回数据库list结果
        return Arrays.asList("aaa", "bbb", "ccc");
    }

}

```


#### 添加Service

``` java
public interface IDemoService {
    List<String> findAll();
}

```

``` java
public class DemoServiceImpl implements IDemoService {

    private IDemoDao demoDao = new DemoDaoImpl();

    @Override
    public List<String> findAll() {
        return demoDao.findAll();
    }
}

```

#### 修改DemoServlet

将我们前面创建的 `PoXingServlet` 内容拷贝 并加以修改

``` java
@WebServlet(urlPatterns = "/demoServlet")
public class DemoServlet extends HttpServlet {

    IDemoService demoService = new DemoServiceImpl();

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        resp.getWriter().println(demoService.findAll().toString());
    }
}

```

#### 运行测试

访问路径  
<http://localhost:8080/spring-framework-ioc-learning/demoServlet>    
(http://ip:端口号/[context-path(默认是工程名称可以自行修改)]/uri)

浏览器可输出 `['aaa', 'bbb', 'ccc']`  此时已完成基础Servlet


基础搭建工作完成,下面才是**我们的重点! 我们的重点!! 我们的重点!!!**

### 好吧 需求开始变更

现在你已经完成了开发工作,数据库使用的Mysql
增删改查都以实现(你就当做已经实现了...虽然我并没有写增删改)
项目马上交付,此时你的领导(XX领导完全不懂技术)电话da来了.

> 那个谁谁谁  最新我听说大公司都用Oracle数据库
> 客户好像都觉得Oracle数据库更靠谱呢
> 这样吧, 你也给我换成Oracle数据库

此时你的内心应该是这样的 XX领导, XX不干了,


![不乐意](./03.png)  ![都是我的错行了吧](./04.jpg)   ![我滚了](./05.png) ![跪着走完自己选择的路](./06.jpg)

木头办法哎, 找下家太麻烦, 当前还是要恰饭的, 服从吧  咱小咱卑微 咱是干饭人 咱的错
咱改

#### 修改数据

对于数据库的切换  我们知道  不仅要修改数据库连接池等相关的配置 还要修改
相应的SQL语句(特定的SQL对于不同的数据库写法是不一样滴,分页就不一样), 咋办?
改Dao层吧 于是乎 开始修改项目里所有的Dao实现类 也就是DaoImpl

``` java
public class DemoDaoImpl implements IDemoDao {
    @Override
    public List<String> findAll() {
        // 此处为演示方便不连接数据库 模拟返回数据库list结果
//        return Arrays.asList("aaa", "bbb", "ccc");
        // 模拟修改成了Orcale的SQL的语句了
        return Arrays.asList("xx领导(Oracle数据库)", "xx领导(Oracle数据库)", "xx领导(Oracle数据库)");
    }

}


``` 

哈哈 全部检查一遍  自动化测试也执行完  没毛病  搜易贼...
可以摸鱼了...

### 嘟嘟嘟  需求再次变更

项目终于改完要交付了, 干了杯枸杞红枣茶, 摸鱼准备下班中...

> 那个那个那个啥...  怎么Oracle数据库要花钱？？？ 公司最近客户还没有回款
> 你把数据库还是换回MySQL吧 后面等客户给回款了我们再考虑用Oracle

此时的你...

![逐渐起了杀心](./07.jpg)

哎 再一再二  我想大概也会有再三吧 整吧 这次我要想想办法了
我把你的再三再四直接给你弄上  省的影响我摸鱼.

#### 引入静态工厂

我要创建一个静态工厂, 你想要啥数据库我给你整出来一个啥数据来
这里暂时起名`BeanFactory` 别问为啥 我就想起这个名 我卑微我任性

``` java
public class BeanFactory {
    public static IDemoDao getDemoDao() {
        // 返回 mysql 的 Dao
        // return new DemoDaoImpl();
        // 返回 oracle 的 Dao
        return new DemoOracleDaoImpl();
    }
}


```

#### 改造Service实现即 ServiceImpl
ServiceImpl 中引用的 Dao 不再可以是手动 new 出来的了,
而应该由 `BeanFactory` 的静态方法返回来获得

``` java
public class DemoServiceImpl implements IDemoService {

//    private IDemoDao demoDao = new DemoDaoImpl();
    private IDemoDao demoDao = BeanFactory.getDemoDao();

    @Override
    public List<String> findAll() {
        return demoDao.findAll();
    }
}

```

好了 即使你再发生变更(改回Oracle) 也不会影响我摸鱼了  
我只是改改**BeanFactory中的静态方法而已**

现在终于可以交付项目了, 终于.....

抱歉(并不是你心里所想的会出啥事) 一切顺利 老板满意 客户满意  
开会要给你先画个饼 鼓励一下咯

### 又出现了新的问题

系统已经上线运行  这个时候你已经着手进行下一个项目了,
可是..可是 领导觉得你的任务最近不多呀 提出让你对上一个项目进行优化
和扩展的需求, 这时候你再次打开了旧项目 居然发现项目编译不通过. 此处为了
演示编译无法通过,删除`DemoDaoImpl.java`

此时你的脑海里一定在翻滚,为么为么为么之前好好的,机器也没换啊,找到编译
出错的类`BeanFactory`,我x, 我的类去哪儿了,去哪儿了,去哪儿了,三连问之后
少了`DemoDaoImpl.java`  导致无法编译

#### 依赖关系导致紧耦合

``` java

public class BeanFactory {
    public static IDemoDao getDemoDao() {
        // 返回 mysql 的 Dao
        // 因为 DemoDaoImpl.java不存在导致编译失败
         return new DemoDaoImpl();
        // 返回 oracle 的 Dao
//        return new DemoOracleDaoImpl();
    }
}


```

`BeanFactory`类中因 `DemoDaoImpl.java` 的不存在 而报红, 导致编译没法通过
像当前 `BeanFactory` **强依赖于** `DemoDaoImpl` (没有`DemoDaoImpl`就报红给你看)
这就是**紧耦合**


#### 解决紧耦合

没有这个Java文件  我没法干活呀 要不我自己造一个空的？？？ 不行不行
万一明天 `DemoOracleDaoImpl` 这个又没有了呢, 我不能总造空的玩呀
突然此时 你灵光一现 好像 好像 反射大概似乎可能也许可以解决这个问题呢
**反射可以声明一个类的全限定类名,进而获取它的字节码,这样也可以构造一个对象**

于是乎 开干吧 `BeanFactory` 就成这样的了

``` java
public class BeanFactory {

    public static IDemoDao getDemoDao() {
        try {
            return (IDemoDao) Class.forName("com.huodd.dao.impl.DemoDaoImpl").newInstance();
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException("IDemoDao instantiation error, cause: " + e.getMessage());
        }
    }
}

```

现在编译问题得以解决,虽然 `DemoService`在初始化的时候 还有有问题 但是整个项目
起码不会无法编译了

#### 弱依赖的引入

当前使用了反射 编译通过 但是工程启动后 由于`BeanFactory` 要构造 `DemoDaoImpl`
时确实还没有该类,所以会抛出`ClassNotFoundException`
但是此时的`BeanFactory`对`DemoDaoImpl` 的依赖程度(你愿意丢你就丢 别影响我编译)
就算的上是 **弱依赖**了

#### 硬编码

弱依赖的完成 你找遍电脑的各个盘 终于把 `DemoDaoImpl.java` 该类给找回来了,这下一切OK
不影响编译了,不影响运行了,但是好像还哪里不大对呢, 要是再切换数据库呢, 我在工厂类里面
写死了全限定类名,切换数据库的时候我还得去重新编译一遍工程才可以正常运行,
应该还可以有办法. 可以考虑下.

#### 引入外部化的配置文件

机智如你呀,终究被你想到了**利用IO实现文件的存储配置**  每次`BeanFactory`
被初始化时 就让他去读取**配置文件**就好了 我下次就改改配置文件就行了
硬编码问题得以解决


##### factory.properties 创建

``` properties
demoDao=com.huodd.dao.impl.DemoDaoImpl

```
为方便取到全限定类名,前面是我们给类起的一个"**别名**"(类似于mybatis的xml中的alias思想)
这样后面可以直接通过 "**别名找到对应类的全限定名**"

##### BeanFactory 改造

``` java
public class BeanFactory {
    private static Properties properties;

    // 使用静态代码块初始化properties，加载factord.properties文件
    static {
        properties = new Properties();
        try {
            // 必须使用类加载器读取resource文件夹下的配置文件
            properties.load(BeanFactory.class.getClassLoader().getResourceAsStream("factory.properties"));
        } catch (IOException e) {
            // BeanFactory类的静态初始化都失败了，那后续也没有必要继续执行了
            throw new ExceptionInInitializerError("BeanFactory initialize error, cause: " + e.getMessage());
        }
    }
	
	public static IDemoDao getDemoDao() {
        String beanName = properties.getProperty("demoDao");
        try {
            Class<?> beanClazz = Class.forName(beanName);
            return (IDemoDao) beanClazz.newInstance();
        } catch (ClassNotFoundException e) {
            throw new RuntimeException("BeanFactory have not [" + beanName + "] bean!", e);
        } catch (IllegalAccessException | InstantiationException e) {
            throw new RuntimeException("[" + beanName + "] instantiation error!", e);
        }
    }

}



```

等等  等等  这里我们好像又把起的那个别名`demoDao`给写死了,得改 干脆**传参**吧
要哪个全限定类名参数传哪个全限定类名对应的别名 此时 `getDemoDao`这个名字得改还要加参数 此时
就叫`getBean`吧 根据别名获取相应的Bean对象

``` java
public static IDemoDao getBean(String beanAlias) {
        String beanName = properties.getProperty("beanAlias");
        try {
            Class<?> beanClazz = Class.forName(beanName);
            return (IDemoDao) beanClazz.newInstance();
        } catch (ClassNotFoundException e) {
            throw new RuntimeException("BeanFactory have not [" + beanName + "] bean!", e);
        } catch (IllegalAccessException | InstantiationException e) {
            throw new RuntimeException("[" + beanName + "] instantiation error!", e);
        }
    }

```


##### ServiceImpl 改造

`DemoServiceImpl` 中不能调用 `getDao` 方法(让我们给改成了`getBean(String beanAlias)`)了，

``` java
public class DemoServiceImpl implements IDemoService {

	IDemoDao demoDao = (IDemoDao) BeanFactory.getBean("demoDao");
    
	@Override
    public List<String> findAll() {
        return demoDao.findAll();
    }
}

``` 
到这里，你突然发现一个现象：
这下你可以把**所有**想抽取出来的**组件都可以做成外部化配置**了！

PS: 自行去`DemoServlet`里面把对`DemoServiceImpl`的紧耦合改正
> 算了 简单说一下  
> 在 `factory.properties` 外部配置化文件 加入 `demoService=com.huodd.service.impl.DemoServiceImpl`  
> `DemoServlet`获取`DemoServiceImpl`改成 `IDemoService demoService = (IDemoService) BeanFactory.getBean("DemoServiceImpl");`

#### 外部配置化

对于这种可能会变化的配置、属性等，通常不会直接硬编码在源代码中，
而是抽取为一些配置文件的形式（ properties 、xml 、json 、yml 等），
配合程序对配置文件的加载和解析，从而达到动态配置、降低配置耦合的目的。
由此大概我们可以知道 原来并不是那些老外拍脑袋随便就乱搞才出来的这个IoC思想


#### 多重构建问题

细心的小伙伴可能已经发现 项目还会存在问题 影响性能的问题 也是很大的问题
就是`BeanFactory#getBean(String beanAlias)` 会重复创建同一个对象的多个实例
而我们根本就不需要每次都给我去创建新的对象实例,旧的就够用了
因此处不是本文重点  小伙伴自行改进  这里给出一个大概的思路

> 可以考虑引入缓存 将创建出来的实例缓存起来 每次我们从缓存中读取  
> PS: 要多考虑一点哦 别忘了多线程并发问题


到这里,不知道小伙伴是否对IOC有了个全新的认识和理解呢

这里总结一下里面出现的几个关键点

- 静态工厂可将多处依赖进行抽取分离
- 外部化配置文件+反射可解决配置的硬编码问题
- 缓存可控制对象实例数(这里我们并没有具体去实现小伙伴自行动手哦)

接下来  是否解决了文章开始时小伙伴们的困惑呢？

## IOC的思想引入

``` java

private IDemoDao dao = new DemoDaoImpl();

private IDemoDao dao = (IDemoDao) BeanFactory.getBean("demoDao");
```

对比如上两种方法获取`dao`
- 前者强依赖/紧耦合 后者弱依赖/松散耦合
- 前者需保证`DemoDaoImpl`存在才能通过编译
  后者无需保证`DemoDaoImpl`存在就可以通过编译 倘若`factory.properties` 中声明的全限定类名出现错误，
  则会出现`ClassCastException `


仔细体会下面这种对象获取的方式，本来咱开发者可以使用上面的方式，
主动声明实现类，但如果选择下面的方式，
那就不再是咱自己去声明，而是将**获取对象的方式交给了**`BeanFactory` 。
这种将**控制权交给别人**的思想，就可以称作：**控制反转（ Inverse of Control , IOC ）**。
而 BeanFactory 根据指定的 beanName 去获取和创建对象的过程，
就可以称作：**依赖查找（ Dependency Lookup , DL ）**。


## 源码获取

本文源代码获取 可添加公众号"PoXing" 回复【IOC】进行获取
扫码添加一下吧~~~
![公众号二维码](./PoXing.jpg)





