---
layout: post
title: "spring 三bean的生命周期以及作用域"
date: 2017-05-10
excerpt: "spring的生命周期问题"
tags: [java, spring]
comments: true
---

spring中的bean也是有生命周期的，就像一个人从出生到死亡，中间会经历10岁，20岁... 那么对于spring中的bean他们就会经过一系列的方法。说生命周期之前我们还是先看bean的作用域

***

### bean作用域
bean的作用域有几种类型，之前我们通过获取上下文的方式获取到applicationContext对象，在加载我们的spring的配置的时候我们就创建了bean，这种Bean的作用域是Singleton类型(没有在配置时更改作用域类型的bean)。除了这种方式可以去获取bean，我们还可以通过先获取beanFactory来获取bean，通过XmlBeanFactory()获取到beanFactory，但是这种方法在spring3.1之后就不用了，通过这种方式拿到的bean是propotype类型。除此之外还有request,session和global-session类型。后三个都是在java web中有效。
singleton是单例类型，就是在创建起容器时就自动创建了一个bean的对象，不管你是否使用，他都存在了，每次获取到的对象都是同一个对象。
propotype是原型类型，他在我们创建容器的时候并没有实例化，而是当我们获取bean的时候才会去创建一个对象。而且我们每次获取到的对象都不是同一个对象。
下面用例子来看看作用域，首先我们创建如下图的module取名为life
![life](http://upload-images.jianshu.io/upload_images/4638441-82449c28bbed7a77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
其中secondDemo，TypeTest是我们现在要用到的，其余的文件是下面生命周期用到的文件，secondDemo很简单,代码如下
```
public class SecondDemo {

    private String secondDemo;

    public SecondDemo() {
        System.out.println("SecondDemo被创建");
    }

    public String getSecondDemo() {
        return secondDemo;
    }

    public void setSecondDemo(String secondDemo) {
        this.secondDemo = secondDemo;
    }
}
```
然后我们在applicationContext文件中配置bean，显示默认配置，不对作用域配置
```
<bean id="secondDemo" class="cn.pwc.demo.SecondDemo">
    <property name="secondDemo" value="我是第二个demo"/>
</bean>
```
然后我们在typeTest中进行测试
```
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
System.out.println("----------------------begin-------------------------");
SecondDemo s1 = (SecondDemo) applicationContext.getBean("secondDemo");
System.out.println("----------------------end-------------------------");
```
运行之后
![singleton](http://upload-images.jianshu.io/upload_images/4638441-63493273398d2656.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
构造函数的secondDemo是不是在begin之前打出来了也就意味着我们在取得对象之前这个对象就已经创建好了，然后我们改变他的作用于，将bean的配置改为
![修改之后配置bean](http://upload-images.jianshu.io/upload_images/4638441-045c4856db2d5f42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
再次运行
![propertype](http://upload-images.jianshu.io/upload_images/4638441-076d57835cd85953.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们可以看到构造函数的secondDemo在begin和end之间打印出来了，这也就验证了对象的创建是在我们取得对象的时候创建的。对于多次取得的对象是否是同一个，我们可以在测试方法里稍加修改，propotype为
![propertypr对象验证](http://upload-images.jianshu.io/upload_images/4638441-e6dba8e83f0d2b35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
singleton测试方法一样，看打印出来的地址
![singleton对象验证](http://upload-images.jianshu.io/upload_images/4638441-ae80300c61018102.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
好了，作用于就到这里，下面看生命周期。

***

### bean的生命周期
对于bean生命周期的验证我们就看module目录中的FirstDemo,MyBeanPostProcessor和test文件。由于容器在创建的时候bean对象就产生了，所以我们的test文件就一句ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");就行了。
我们给出生命周期的流程图，然后去看她的每个过程。
![生命周期](http://upload-images.jianshu.io/upload_images/4638441-05bf2b9b2f2a01d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
创建不多说了，就是使用构造函数来创建起对象。
set，我们在配置bean的时候给对象注入了属性值，这里就是通过set方法注入属性值，是第二步
BeanNameAware，这是一个接口，实现该接口，我们需要实现setBeanName方法，该方法的唯一参数就是我们现在在创建的对象的对象名。
BeanFactoryAware，这也是一个接口，该接口的方法是setBeanFactory，该方法的参数将传入一个BeanFactory，该参数是bean工厂，里面有你配置的所有的bean。
ApplicationContextAware，接口，里面有setApplicationContext方法，这个方法传入的参数是上下文对象。
BeanPostProcessor（这里只执行before方法），接口，但是该接口不和其他接口一样，通过让firstDemo去继承实现它，而是新建一个类去继承BeanPostProcessor，然后通过去spring配置文件中配置这个新建的bean来关联其他bean来实现该接口。我们module中的MyBeanPostProcessor就实现了该接口。
```
public class MyBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object o, String s) throws BeansException {
        System.out.println("postProcessBeforeInitialization被调用");
        return o;
    }

    @Override
    public Object postProcessAfterInitialization(Object o, String s) throws BeansException {
        System.out.println("postProcessAfterInitialization被调用");
        return o;
    }
}
```
bean的配置
```
<bean id="firstDemo" class="cn.pwc.demo.FirstDemo" init-method="myInit" destroy-method="myDestroy">
    <property name="name" value="给firstDemo的值"/>
</bean>

<bean id="beanPost" class="cn.pwc.demo.MyBeanPostProcessor"/>
```
该接口有before和after两个方法，在调用before之后并不马上调用after。中间还有两个初始化的过程。实现该接口的类有点像拦截器，就是和他关联的每个类都要通过这里，这里又给我们有点aop的影子，我们在这里可以对每个对象可以进行统一的操作。aop后面介绍。
InitializingBean，接口，该接口有方法afterPropertiesSet，这里可以对对象进行一些初始化，除了这个接口可以初始化，下一步的方法也是初始化。
自定义init方法，看上图的bean配置是不是和之前的有点不同，多了init-method，这是制定对该bean的对象进行初始化，初始化的方法是myInit，在firstDemo中实现该方法。
BeanPostProcess（这里执行after方法）,这个接口在上面已经做了一些介绍。
接下去就是使用了。
使用完了容器销毁，释放资源。
DisposableBean，接口，该接口有destroy方法，该方法是bean生命中的最后一步，执行一些资源的释放，不识闲该方法，我们还可以用另外一种实现方式。
自定义的销毁方法，我们的firstDemo的配置除了多了init-method之外我们还多了destory-method，该属性指明我们在销毁的时候应该执行firstDemo中的哪个方法，这是bean的最后一步。
最后我们运行test，出现下图
![test](http://upload-images.jianshu.io/upload_images/4638441-3aa6a4d009e10994.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
销毁方法看不到是因为容器已经销毁了，但确实是执行了。

源码地址：[springDemo](https://github.com/pang-weichao/springDemo)