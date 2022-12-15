# spring 源码整体分析 - 告别时光 - 博客园
spring 架构原理图
------------

![](https://img-blog.csdnimg.cn/47c73d11e9ae4d318bd4671ffdfb71e5.jpg?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ZGK5Yir5pe25YWJ,size_20,color_FFFFFF,t_70,g_se,x_16)

* * *

核心注解
----

### 常用注解

![](https://img-blog.csdnimg.cn/c767f9f5c39b4b5d9c01a821f508f598.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ZGK5Yir5pe25YWJ,size_20,color_FFFFFF,t_70,g_se,x_16)

![](https://img-blog.csdnimg.cn/a7bb71494cb447588d081115842f0a46.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ZGK5Yir5pe25YWJ,size_20,color_FFFFFF,t_70,g_se,x_16)

### @Bean

使用 @Bean + @Configuration 的形式可以替代 xml 配置文件的形式

### @Import

@Import：指示要导入的一个或多个组件类

Spring 提供了很多方式来定义 bean 的信息，包括 xml 配置文件，注解，网络，磁盘等，通过资源加载器加载这些资源中的 bean 信息到 BeanDefinitionRegistrar 中，BeanDefinitionRegistrar 就相当于 bean 的图纸（除此之外还有 SimpleAliasRegistry 和SingletonBeanRegistry 等其他保存 bean 定义信息的图纸），无论是 xml 还是注解方式定义的 bean 信息，最终进入的都是 BeanDefinitionRegistrar 这种类型中保存，在这其中定义的只是 bean 的类型、名称信息，bean 的属性值等信息不在其中定义。

在 @Import 中可以直接传入 bean 的 class，也可以传入 BeanDefinitionRegistry 加载其中所有的 bean 定义信息，使用如下

```null
@Import({Person.class, Config.MyBeanDefinitionRegistry.class})

```

```null
static class MyBeanDefinitionRegistry implements ImportBeanDefinitionRegistrar {
	@Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) 	{
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition();
        
        
        rootBeanDefinition.setBeanClass(Person.class);
        registry.registerBeanDefinition("person01", rootBeanDefinition);
    }
}

```

### @Lookup

当单例的组件中需要依赖非单例的组件时，使用此注解实现，注意，一旦使用此注解，则对应的非单实例属性不能使用 @Autowired 注入且 @Lookup 需要标注在对应属性的 getXxx() 方法上，使用如下

```null
public class MainTest {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
        Person p1 = context.getBean(Person.class);
        Person p2 = context.getBean(Person.class);

        System.out.println(p1.getCat() == p2.getCat()); 
    }
}

```

Cat 类

```null
@Scope(scopeName = ConfigurableBeanFactory.SCOPE_PROTOTYPE) 
@Component
public class Cat {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

```

Person 依赖 Cat

```null
package com.dhj.demo.bean;

import org.springframework.beans.factory.annotation.Lookup;
import org.springframework.stereotype.Component;


@Component
public class Person {
    private String name;

    private Cat cat;

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person{" +
            "name='" + name + '\'' +
            '}';
    }

    @Lookup
    public Cat getCat() {
        return cat;
    }

    public void setCat(Cat cat) {
        this.cat = cat;
    }

```

spring 大致容器类关系图
---------------

![](https://img-blog.csdnimg.cn/7600d1656c094825951c482b8a32998f.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ZGK5Yir5pe25YWJ,size_20,color_FFFFFF,t_70,g_se,x_16)

* * *

spring 核心组件&接口
--------------

### org.springframework.core.io.Resource

此接口代表的是一个资源的抽象，Spring 将不同资源（磁盘，网络，xml 配置等）加载为一个 Resource

### org.springframework.core.io.ResourceLoader

ResourceLoader 用于加载 Resource 中的资源，ResourceLoader 使用了策略模式

**策略模式简介**：

> 策略（Strategy）模式的定义：该模式定义了一系列算法，并将每个算法封装起来，使它们可以相互替换，且算法的变化不会影响使用算法的客户。策略模式属于对象行为模式，它通过对算法进行封装，把使用算法的责任和算法的实现分割开来，并委派给不同的对象对这些算法进行管理

**策略模式的主要角色如下**：

> 抽象策略（Strategy）类：定义了一个公共接口，各种不同的算法以不同的方式实现这个接口，环境角色使用这个接口调用不同的算法，一般使用接口或抽象类实现。
> 
> 具体策略（Concrete Strategy）类：实现了抽象策略定义的接口，提供具体的算法实现。
> 
> 环境（Context）类：持有一个策略类的引用，最终给客户端调用

ResourceLoader 接口中定义了对资源的各种抽象策略，ResourceLoader 的具体实现类中则是对抽象策略的各种具体实现。用以应对不同资源的加载，例如默认的实现类 DefaultResourceLoader 就可以对 url 资源和 classpath下的资源进行加载。

此时还缺少一个环境类，这个环境类就是 AbstractApplicationContext，在该环境类中，引用了抽象策略 ResourceLoader 且在类创建时，就指向了一个具体的策略实例

```null

public AbstractApplicationContext() {
    this.resourcePatternResolver = getResourcePatternResolver();
}

```

### org.springframework.beans.factory.BeanFactory

用于访问Spring bean容器的根接口，该接口的主要类图如下

![](https://img-blog.csdnimg.cn/c9763cd8078741d9b13e353441fd9bf6.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ZGK5Yir5pe25YWJ,size_20,color_FFFFFF,t_70,g_se,x_16)

主要是一些工厂和各种 ioc 容器，而由此牵涉出来的其他类图如下

![](https://img-blog.csdnimg.cn/a6d999eeede24fd689b3a4b65a63a160.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ZGK5Yir5pe25YWJ,size_20,color_FFFFFF,t_70,g_se,x_16)

其中 BeanDefinitionRegistry，SingletonBeanRegistry 等都相当于是 bean 的图纸，保存 bean 的定义信息，最终通过层层的继承，实现，组合等方式，得到了一个 DefaultListableBeanFactory（总档案馆，与多个档案馆、工厂都有联系），该类中就保存了 ioc 中的核心信息（包括所有 bean 的定义信息等），部分相关属性如下

```null


private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);



private final Map<Class<?>, String[]> allBeanNamesByType = new ConcurrentHashMap<>(64);


private final Map<Class<?>, String[]> singletonBeanNamesByType = new ConcurrentHashMap<>(64);


private volatile List<String> beanDefinitionNames = new ArrayList<>(256);


private volatile Set<String> manualSingletonNames = new LinkedHashSet<>(16);


@Nullable
private volatile String[] frozenBeanDefinitionNames;

```

BeanFactory 的另一个接口实现 AutowireCapableBeanFactory 则定义了 bean 自动装配的能力，而我们常用的对注解进行支持的 AnnotationConfigApplicationContext 的父类 GenericApplicationContext 中，就通过组合的方式（合成复用原则）组合了上面提到的总档案馆 DefaultListableBeanFactory， 部分源码如下

```null
public class GenericApplicationContext extends AbstractApplicationContext implements BeanDefinitionRegistry {

	private final DefaultListableBeanFactory beanFactory;
	
	....
}

```

这也就相当于 AnnotationConfigApplicationContext 拥有了自动装配的能力

### spring 资源加载 & bean 信息定义流程

以常用的 ClassPathXmlApplicationContext 为例，该类的实例就是一个 spring 容器

```null
 ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");

```

> 从类路径下的 bean.xml 中加载资源，最终构造成一个 ioc 容器

构造器底层调用如下

```null
public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
		this(new String[] {configLocation}, true, null);
	}

```

继续深入，如下

```null
public ClassPathXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {

		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
            
			refresh();
		}
	}

```

> 传入资源名 configLocations 也就是 bean.xml，且 refresh 为 true，这代表在构建 spring 容器时就会刷新

refresh() 方法部分如下

```null
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        
        prepareRefresh();

        
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        
        prepareBeanFactory(beanFactory);

        ....

```

最主要是 obtainFreshBeanFactory() 方法，通过 debug 堆栈继续跟踪，其底层调用如下

```null
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
		refreshBeanFactory();
		return getBeanFactory();
	}

```

继续跟踪，调用的是 refreshBeanFactory()，依旧是刷新 bean 工厂，源码如下

```null
@Override
protected final void refreshBeanFactory() throws BeansException {
    if (hasBeanFactory()) {
        destroyBeans();
        closeBeanFactory();
    }
    try {
        
        DefaultListableBeanFactory beanFactory = createBeanFactory();
        beanFactory.setSerializationId(getId());
        customizeBeanFactory(beanFactory);

        
        loadBeanDefinitions(beanFactory);
        synchronized (this.beanFactoryMonitor) {
            this.beanFactory = beanFactory;
        }
    }
    catch (IOException ex) {
        throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
    }
}

```

> DefaultListableBeanFactory 内部的档案馆定义如下
> 
> ```null
> 
> private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
> 
> ```
> 
> 使用 bean 的名字作为 key

创建了档案馆之后，才会将 bean 的定义信息加载进档案馆，调用的是 loadBeanDefinitions() 方法，底层如下

```null
@Override
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
    
    XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

    
    
    beanDefinitionReader.setEnvironment(this.getEnvironment());
    beanDefinitionReader.setResourceLoader(this);
    beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

    
    
    initBeanDefinitionReader(beanDefinitionReader);
    
    
    loadBeanDefinitions(beanDefinitionReader);
}

```

> 创建读取器后，还为该读取器设置了一个资源加载器
> 
> ```null
> beanDefinitionReader.setResourceLoader(this);
> 
> ```
> 
> 这说明读取器就是通过资源加载器来加载 xml 配置文件资源的

继续堆栈跟踪如下

```null
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
		Resource[] configResources = getConfigResources();
		if (configResources != null) {
			reader.loadBeanDefinitions(configResources);
		}
    
        
		String[] configLocations = getConfigLocations();
		if (configLocations != null) {
            
            
			reader.loadBeanDefinitions(configLocations);
		}
	}

```

堆栈跟踪

```null
@Override
public int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException {
	Assert.notNull(locations, "Location array must not be null");
	int count = 0;
	for (String location : locations) {
        
		count += loadBeanDefinitions(location);
	}
	return count;
}

```

堆栈跟踪，部分源码如下

```null

if (resourceLoader instanceof ResourcePatternResolver) {
    
    try {
        
        Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
        
        int count = loadBeanDefinitions(resources);
        if (actualResources != null) {
            Collections.addAll(actualResources, resources);
        }
        if (logger.isTraceEnabled()) {
            logger.trace("Loaded " + count + " bean definitions from location pattern [" + location + "]");
        }
        return count;
    }
    catch (IOException ex) {
        throw new BeanDefinitionStoreException(
            "Could not resolve bean definition resource pattern [" + location + "]", ex);
    }
}

```

堆栈跟踪

```null
try {
    InputStream inputStream = encodedResource.getResource().getInputStream();
    try {
        InputSource inputSource = new InputSource(inputStream);
        if (encodedResource.getEncoding() != null) {
            inputSource.setEncoding(encodedResource.getEncoding());
        }
        
        return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
    }
    finally {
        inputStream.close();
    }
}
catch (IOException ex) {
    throw new BeanDefinitionStoreException(
        "IOException parsing XML document from " + encodedResource.getResource(), ex);
}
finally {
    currentResources.remove(encodedResource);
    if (currentResources.isEmpty()) {
        this.resourcesCurrentlyBeingLoaded.remove();
    }
}

```

堆栈跟踪

```null
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
    throws BeanDefinitionStoreException {

    try {
        
        Document doc = doLoadDocument(inputSource, resource);

        
        int count = registerBeanDefinitions(doc, resource);
        if (logger.isDebugEnabled()) {
            logger.debug("Loaded " + count + " bean definitions from " + resource);
        }
        return count;
    }

```

堆栈跟踪

```null
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
    BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
    int countBefore = getRegistry().getBeanDefinitionCount();
    
    
    documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
    return getRegistry().getBeanDefinitionCount() - countBefore;
}

```

堆栈跟踪

```null
@Override
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
    this.readerContext = readerContext;
    
    doRegisterBeanDefinitions(doc.getDocumentElement());
}

```

堆栈跟踪

```null
preProcessXml(root);


parseBeanDefinitions(root, this.delegate);
postProcessXml(root);

this.delegate = parent;

```

> 在解析 dom 对象时，还传入了一个 delegate 对象，这实际上是一个代理类，规定了 spring 配置文件中能被识别的标签，其源码部分如下
> 
> ```null
> public static final String MULTI_VALUE_ATTRIBUTE_DELIMITERS = ",; ";
> 
> 	
> 	public static final String TRUE_VALUE = "true";
> 
> 	public static final String FALSE_VALUE = "false";
> 
> 	public static final String DEFAULT_VALUE = "default";
> 
> 	public static final String DESCRIPTION_ELEMENT = "description";
> 
> 	public static final String AUTOWIRE_NO_VALUE = "no";
> 
> 	public static final String AUTOWIRE_BY_NAME_VALUE = "byName";
> 
> 	public static final String AUTOWIRE_BY_TYPE_VALUE = "byType";
> 
> 	public static final String AUTOWIRE_CONSTRUCTOR_VALUE = "constructor";
> 
> 	public static final String AUTOWIRE_AUTODETECT_VALUE = "autodetect";
> 
> 	public static final String NAME_ATTRIBUTE = "name";
> 
> 	public static final String BEAN_ELEMENT = "bean";
> 
> ```

继续堆栈跟踪

```null
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
    if (delegate.isDefaultNamespace(root)) {
        NodeList nl = root.getChildNodes();
        for (int i = 0; i < nl.getLength(); i++) {
            Node node = nl.item(i);
            if (node instanceof Element) {
                Element ele = (Element) node;
                if (delegate.isDefaultNamespace(ele)) {
                    
                    parseDefaultElement(ele, delegate);
                }
                else {
                    
                    delegate.parseCustomElement(ele);
                }
            }
        }
    }
    else {
        delegate.parseCustomElement(root);
    }
}

```

> 这里就开始真正的解析 dom 对象了，根据传入的 delegate 对象作为标准来解析 dom 对象中的元素，

堆栈跟踪

```null
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
    if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
        importBeanDefinitionResource(ele);
    }
    else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
        processAliasRegistration(ele);
    }

    
    else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
        processBeanDefinition(ele, delegate);
    }
    else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
        
        doRegisterBeanDefinitions(ele);
    }
}

```

堆栈跟踪

```null
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
    
    BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
    if (bdHolder != null) {
        bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
        try {
            
            BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
        }
        catch (BeanDefinitionStoreException ex) {
            getReaderContext().error("Failed to register bean definition with name '" +
                                     bdHolder.getBeanName() + "'", ele, ex);
        }
        
        getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
    }
}

```

> BeanDefinitionHolder 部分源码如下
> 
> ```null
> public class BeanDefinitionHolder implements BeanMetadataElement {
> 
> 	private final BeanDefinition beanDefinition;
> 
> 	private final String beanName;
> 
> 	@Nullable
> 	private final String[] aliases;
> 
> 	.....
> 
> ```
> 
> dom 元素最近会被解析为 beanName + BeanDefinition 的形式

继续调用 `BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());` 将 bean 信息注册到档案馆中，registerBeanDefinition() 方法实现如下

```null
public static void registerBeanDefinition(
    BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
    throws BeanDefinitionStoreException {

    
    String beanName = definitionHolder.getBeanName();
    
    
    registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

    
    String[] aliases = definitionHolder.getAliases();
    if (aliases != null) {
        for (String alias : aliases) {
            registry.registerAlias(beanName, alias);
        }
    }
}

```

> registerBeanDefinition() 方法有不同的实现，这里使用的是 DefaultListableBeanFactory 中的实现

在 DefaultListableBeanFactory 的 registerBeanDefinition() 方法中，正常情况下，最终会将 bean 信息放入 Map 结构的档案馆中

```null
this.beanDefinitionMap.put(beanName, beanDefinition);

```

无论是 DefaultListableBeanFactory 还是其他的一些实现类，他们都实现的是 BeanDefinitionRegistry，可以说 BeanDefinitionRegistry 就是所有 bean 定义信息的档案馆，至此，完成了整个的从 spring xml 配置文件最终被加载为 bean 定义信息的流程。

流程图总结

![](https://img-blog.csdnimg.cn/7566e1de2fb04bdf9b93477cf6d0bbf7.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ZGK5Yir5pe25YWJ,size_20,color_FFFFFF,t_70,g_se,x_16)

### bean 实例化 & 属性自动装配过程

前面的流程依然是一样的，调用 spring 容器的构造器，先刷新创建 bean 工厂

```null
public ClassPathXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {

		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}

```

完成 bean 工厂的初始化

```null

finishBeanFactoryInitialization(beanFactory);

```

跟踪堆栈，finishBeanFactoryInitialization() 方法内部继续调用

```null

beanFactory.preInstantiateSingletons();

```

跟踪 preInstantiateSingletons() 源码

```null
@Override
public void preInstantiateSingletons() throws BeansException {
    if (logger.isTraceEnabled()) {
        logger.trace("Pre-instantiating singletons in " + this);
    }

    
    
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

    
    for (String beanName : beanNames) {
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
            
            if (isFactoryBean(beanName)) {
                Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
                if (bean instanceof FactoryBean) {
                    final FactoryBean<?> factory = (FactoryBean<?>) bean;
                    boolean isEagerInit;
                    if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                        isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
                                                                    ((SmartFactoryBean<?>) factory)::isEagerInit,
                                                                    getAccessControlContext());
                    }
                    else {
                        isEagerInit = (factory instanceof SmartFactoryBean &&
                                       ((SmartFactoryBean<?>) factory).isEagerInit());
                    }
                    if (isEagerInit) {
                        getBean(beanName);
                    }
                }
            }
            else {
                
                getBean(beanName);
            }
        }
    }

```

> 关于工厂 bean，以下就是一个工厂 bean
> 
> ```null
> public class HelloFactoryBean implements FactoryBean<Cat> {
>  @Override
>  public Cat getObject() throws Exception {
>      return new Cat();
>  }
> 
>  @Override
>  public Class<?> getObjectType() {
>      return Cat.class;
>  }
> }
> 
> ```
> 
> 相比较一般的普通 bean，例如 Person 这种直接将自身注册到 spring 容器中的，工厂 bean 注册到容器中的并非自身，而是 getObject() 方法的返回值，注册组件的类型就是 getObjectType 的返回值，实现了 FactoryBean 接口的类，就能通过重写这两个方法来自定义注册组件到 spring 容器中

在 getBean() 中，走的便是 bean 工厂中 doGetBean，createBean() 那一套流程

```null
....

if (mbd.isSingleton()) {
    sharedInstance = getSingleton(beanName, () -> {
        try {
            
            return createBean(beanName, mbd, args);
        }
        catch (BeansException ex) {
            
            
            
            destroySingleton(beanName);
            throw ex;
        }
    });
    bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
}
.....

```

跟踪 createBean() 源码

```null
.....
try {
    
    Object beanInstance = doCreateBean(beanName, mbdToUse, args);
    if (logger.isTraceEnabled()) {
        logger.trace("Finished creating instance of bean '" + beanName + "'");
    }
    return beanInstance;
}
.....

```

继续跟踪 doCreateBean()

```null
if (instanceWrapper == null) {
    
	instanceWrapper = createBeanInstance(beanName, mbd, args);
}
.......

```

跟踪 createBeanInstance()

```null

ctors = mbd.getPreferredConstructors();
if (ctors != null) {
    return autowireConstructor(beanName, mbd, ctors, null);
}


return instantiateBean(beanName, mbd);

```

> 继续跟踪 instantiateBean()，默认使用的是无参构造创建对象，最底层使用的是反射（也可以使用 cglib），如果有构造参数注入的方式，则使用 autowireConstructor()，底层使用的是 BeanUtils 来创建默认的空参构造的实例，相关源码如下
> 
> ```null
> public static <T> T instantiateClass(Constructor<T> ctor, Object... args) throws BeanInstantiationException {
>  Assert.notNull(ctor, "Constructor must not be null");
>  try {
>      ReflectionUtils.makeAccessible(ctor);
>      if (KotlinDetector.isKotlinReflectPresent() && KotlinDetector.isKotlinType(ctor.getDeclaringClass())) {
>          return KotlinDelegate.instantiateClass(ctor, args);
>      }
>      else {
>          Class<?>[] parameterTypes = ctor.getParameterTypes();
>          Assert.isTrue(args.length <= parameterTypes.length, "Can't specify more arguments than constructor parameters");
>          Object[] argsWithDefaultValues = new Object[args.length];
>          for (int i = 0 ; i < args.length; i++) {
>              if (args[i] == null) {
>                  Class<?> parameterType = parameterTypes[i];
>                  argsWithDefaultValues[i] = (parameterType.isPrimitive() ? DEFAULT_TYPE_VALUES.get(parameterType) : null);
>              }
>              else {
>                  argsWithDefaultValues[i] = args[i];
>              }
>          }
>          
>          return ctor.newInstance(argsWithDefaultValues);
>      }
>  }
>  catch (InstantiationException ex) {
>      throw new BeanInstantiationException(ctor, "Is it an abstract class?", ex);
>  }
>  catch (IllegalAccessException ex) {
>      throw new BeanInstantiationException(ctor, "Is the constructor accessible?", ex);
>  }
>  catch (IllegalArgumentException ex) {
>      throw new BeanInstantiationException(ctor, "Illegal arguments for constructor", ex);
>  }
>  catch (InvocationTargetException ex) {
>      throw new BeanInstantiationException(ctor, "Constructor threw exception", ex.getTargetException());
>  }
> }
> 
> ```

通过上述流程 Person 对象的实例已经被创建，如果有属性的自动装配，那么将继续执行 doCreateBean() 方法，也就是如下逻辑

```null

Object exposedObject = bean;
try {
    
    populateBean(beanName, mbd, instanceWrapper);
    exposedObject = initializeBean(beanName, exposedObject, mbd);
}
catch (Throwable ex) {
    if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
        throw (BeanCreationException) ex;
    }
    else {
        throw new BeanCreationException(
            mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
    }
}
......

```

跟踪 populateBean() 方法

```null
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    .......

	
    PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

    .......

    if (pvs != null) {
        
        applyPropertyValues(beanName, mbd, bw, pvs);
    }
}

```

以上是 xml 版本的 bean 实例化和属性赋值，注解方式调用的则是 populateBean() 中的另外一段逻辑，如下

```null

for (BeanPostProcessor bp : getBeanPostProcessors()) {
    if (bp instanceof InstantiationAwareBeanPostProcessor) {
        InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
        
        PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
        if (pvsToUse == null) {
            if (filteredPds == null) {
                filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
            }
            pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
            if (pvsToUse == null) {
                return;
            }
        }
        pvs = pvsToUse;
    }
}

```

跟踪堆栈

```null
@Override
public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
    
    InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);
    try {
        metadata.inject(bean, beanName, pvs);
    }
    catch (BeanCreationException ex) {
        throw ex;
    }
    catch (Throwable ex) {
        throw new BeanCreationException(beanName, "Injection of autowired dependencies failed", ex);
    }
    return pvs;
}

```

至此，spring 中基本的 bean 实例化和属性赋值的流程就完成了

### org.springframework.context.ApplicationContext

ApplicationContext 类关系图如下

![](https://img-blog.csdnimg.cn/7c5132c724304d08af12604bc39fb05a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ZGK5Yir5pe25YWJ,size_20,color_FFFFFF,t_70,g_se,x_16)

通过关系图可知，ApplicationContext 既包含了 bean 定义信息（ListableBeanFactory > DefaultListableBeanFactory），也包含了资源加载器（ResourceLoader）还包括国际化器（MessageSource）以及 Bean 工厂（通过组合的方式获得了自动装配功能 > AutowireCapableBeanFactory）

### org.springframework.beans.factory.Aware & org.springframework.beans.factory.config.BeanPostProcessor

Aware

> 一个标记超级接口，指示 bean 有资格通过回调样式的方法被特定框架对象的 Spring 容器通知。实际的方法签名由各个子接口确定，但通常应仅由一个接受单个参数的返回空值的方法组成。简单来说，我们想要在代码中得到 Spring 管理的一些组件甚至是 Spring 容器时，除了 @Autoware 自动注入外，对应一些特定的组件，就可以实现其对应的接口，重写抽象方法，通过回调的方式得到对应组件的实例

BeanPostProcessor

> 后置增强组件，对 bean 组件进行后置增强，Aware 的回调注入依靠该接口的实现完成

例如通过 ApplicationContextAware 获取 ApplicationContext 如下

```null
public class Person implements ApplicationContextAware {
    ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        context = applicationContext;
    }
.....

```

相关的 Aware 类图

![](https://img-blog.csdnimg.cn/91007cff71c844f492cb0943690d8224.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ZGK5Yir5pe25YWJ,size_20,color_FFFFFF,t_70,g_se,x_16)

下面通过 Aware 来追踪 spring 加载和实例化 bean 以及 BeanPostProcessor 运作的时机的过程，依旧使用 ClassPathXmlApplicationContext 来进行调试

```null
 ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");

```

堆栈跟踪

```null
public ClassPathXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {

		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}

```

> 这部分就是上面提到过的创建 bean 工厂，加载 bean 的定义信息到档案馆

```null

finishBeanFactoryInitialization(beanFactory);

```

```null

beanFactory.preInstantiateSingletons();

```

preInstantiateSingletons() 部分源码如下

```null

for (String beanName : beanNames) {
    RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
    if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
        if (isFactoryBean(beanName)) {
            Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
            if (bean instanceof FactoryBean) {
                final FactoryBean<?> factory = (FactoryBean<?>) bean;
                boolean isEagerInit;
                if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                    isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
                                                                ((SmartFactoryBean<?>) factory)::isEagerInit,
                                                                getAccessControlContext());
                }
                else {
                    isEagerInit = (factory instanceof SmartFactoryBean &&
                                   ((SmartFactoryBean<?>) factory).isEagerInit());
                }
                if (isEagerInit) {
                    
                    getBean(beanName);
                }
            }
        }
        else {
            
            getBean(beanName);
        }
    }
}

```

> 可以看出，在 spring 底层，实际上早就开始获取 bean 了

getBean() 底层调用的实际上是 doGetBean() 方法， 部分源码如下

```null
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
                          @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

    final String beanName = transformedBeanName(name);
    Object bean;

    
    Object sharedInstance = getSingleton(beanName);
    .....

```

继续追踪 getSingleton() 底层，调用到了 org.springframework.beans.factory.support.DefaultSingletonBeanRegistry 中的如下属性

```null

private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

```

回到 doGetBean() 方法，

```null

if (mbd.isSingleton()) {
    
    sharedInstance = getSingleton(beanName, () -> {
        try {
            
            return createBean(beanName, mbd, args);
        }
        catch (BeansException ex) {
            
            
            
            destroySingleton(beanName);
            throw ex;
        }
    });
    bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
}

```

createBean() 是一个 lambda 表达式，在外部继续调用 getSingleton() 方法，追踪关键源码如下

```null
try {
    
    singletonObject = singletonFactory.getObject();
    newSingleton = true;
}

```

这里的 singletonFactory 实际就是传入的 createBean Lambda 表达式，跟踪堆栈，调用的实际上是 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory 中的 createBean() 方法，部分源码如下

```null
try {
   Object beanInstance = doCreateBean(beanName, mbdToUse, args);
   if (logger.isTraceEnabled()) {
      logger.trace("Finished creating instance of bean '" + beanName + "'");
   }
   return beanInstance;
}

```

跟踪 doCreateBean() 方法，关键部分源码如下

```null

BeanWrapper instanceWrapper = null;
if (mbd.isSingleton()) {
    instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
}
if (instanceWrapper == null) {
    
    instanceWrapper = createBeanInstance(beanName, mbd, args);
}

```

跟踪 instantiateBean() 方法，关键源码如下

```null
if (resolved) {
    if (autowireNecessary) {
        return autowireConstructor(beanName, mbd, null, null);
    }
    else {
        
        return instantiateBean(beanName, mbd);
    }
}

```

instantiateBean() 完整源码如下

```null
protected BeanWrapper instantiateBean(final String beanName, final RootBeanDefinition mbd) {
		try {
			Object beanInstance;
			final BeanFactory parent = this;
			if (System.getSecurityManager() != null) {
				beanInstance = AccessController.doPrivileged((PrivilegedAction<Object>) () ->
						getInstantiationStrategy().instantiate(mbd, beanName, parent),
						getAccessControlContext());
			}
			else {
                
				beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, parent);
			}
			BeanWrapper bw = new BeanWrapperImpl(beanInstance);
			initBeanWrapper(bw);
			return bw;
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					mbd.getResourceDescription(), beanName, "Instantiation of bean failed", ex);
		}
	}

```

> spring 将创建 bean 的不同方式封装为不同的策略（jdk 反射创建，cglib 创建子类实例等（代理对象）），根据当前环境的不同使用不同的策略来创建 bean

这里使用的是 SimpleInstantiationStrategy 中的 bean 创建策略，调用的 instantiate() 方法，源码如下

```null
@Override
public Object instantiate(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner) {
    
    if (!bd.hasMethodOverrides()) {
        Constructor<?> constructorToUse;
        synchronized (bd.constructorArgumentLock) {
            constructorToUse = (Constructor<?>) bd.resolvedConstructorOrFactoryMethod;
            if (constructorToUse == null) {
                final Class<?> clazz = bd.getBeanClass();
                if (clazz.isInterface()) {
                    throw new BeanInstantiationException(clazz, "Specified class is an interface");
                }
                try {
                    if (System.getSecurityManager() != null) {
                        constructorToUse = AccessController.doPrivileged(
                            (PrivilegedExceptionAction<Constructor<?>>) clazz::getDeclaredConstructor);
                    }
                    else {
                        
                        constructorToUse = clazz.getDeclaredConstructor();
                    }
                    bd.resolvedConstructorOrFactoryMethod = constructorToUse;
                }
                catch (Throwable ex) {
                    throw new BeanInstantiationException(clazz, "No default constructor found", ex);
                }
            }
        }
        
        return BeanUtils.instantiateClass(constructorToUse);
    }
    else {
        
        return instantiateWithMethodInjection(bd, beanName, owner);
    }
}

```

到此实际上就创建了 bean 的实例，只不过内部还有很多细节，但我们更加应着眼于全局

对于实现了 Aware 相关接口的 bean，其属性回调赋值的流程如下

回到 doCreateBean() 方法处，在 bean 实例创建后，跟踪堆栈，继续调用了 initializeBean() 方法对 bean 进行初始化

跟踪堆栈，在初始化时，使用到了 ApplicationContextAwareProcessor，这就是 BeanPostProcessor 的实现类，在该类中对实现了 Aware 的 bean 进行处理（后置处理），调用了 invokeAwareInterfaces() 方法，源码如下

```null
private void invokeAwareInterfaces(Object bean) {
    if (bean instanceof EnvironmentAware) {
        ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
    }
    if (bean instanceof EmbeddedValueResolverAware) {
        ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
    }
    if (bean instanceof ResourceLoaderAware) {
        ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
    }
    if (bean instanceof ApplicationEventPublisherAware) {
        ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
    }
    if (bean instanceof MessageSourceAware) {
        ((MessageSourceAware) bean).setMessageSource(this.applicationContext);
    }
    if (bean instanceof ApplicationContextAware) {
        ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
    }
}

```

> 判断实现了Aware 接口的 bean 类型，根据不同类型来执行对应的回调传入对应的实例

流程图总结如下

![](https://img-blog.csdnimg.cn/d250d0bb7f3f4141947f8d172edbf085.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ZGK5Yir5pe25YWJ,size_20,color_FFFFFF,t_70,g_se,x_16)

### 生命周期主要接口

#### org.springframework.beans.factory.config.BeanPostProcessor

> 后置增强组件，其每一个子接口增强器在合适的时机运行，在于改变实例化后的 bean 组件，例如对实现了 Aware 接口的组件进行回调属性赋值

#### org.springframework.beans.factory.config.BeanFactoryPostProcessor

> 后置增强工厂，对 BeanFactory 进行后置增强

#### org.springframework.beans.factory.InitializingBean

> 初始化 bean 组件后，对组件进行后续的设置，在于 bean 组件实例化后的额外的处理而不是改变 bean 组件

后面我们将通过上面几个关键接口及其子接口来研究 spring 中 bean 的整个生命周期中被怎样的干预，改变的，通过掌握这整个流程，加深对 spring 运行机制的理解。

* * *

bean 生命周期
---------

### 工厂后置处理接口解析之 BeanFactoryPostProcessor

继续通过 debug 的方式来解析上面三个接口的执行过程，实现上述三个接口，放入容器中，启动加载 spring 容器，debug 跟踪源码

首先，依旧是刷新创建工厂

```null
public ClassPathXmlApplicationContext(
    String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
    throws BeansException {

    super(parent);
    setConfigLocations(configLocations);
    if (refresh) {
        refresh();
    }
}

```

工厂刷新创建成功后，就开始调用相关后置增强处理方法了，可以看出，最先调用的实际上是 bean 工厂的后置增强处理：BeanDefinitionRegistryPostProcessor，这也很好理解，bean 工厂是造 bean 实例的，要是想要对 bean 工厂进行一些定制化的增强，肯定要在 bean 实例化之前或者说最初就进行增强逻辑的调用，以便保证这种增强在可能对 bean 实例化有影响的情况下起作用，倘若在 bean 实例化后再增强 bean 工厂，那这种影响就不会生效

```null
....

ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();


prepareBeanFactory(beanFactory);

try {
    
    postProcessBeanFactory(beanFactory);

    
    invokeBeanFactoryPostProcessors(beanFactory);
    .....

```

跟踪堆栈，invokeBeanFactoryPostProcessors() 调用的源码如下

```null
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    	
		PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());

		
		
		if (beanFactory.getTempClassLoader() == null && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}
	}

```

下面，开始真正的调用 bean 工厂后置处理的逻辑（重要部分通过注释标注）

```null
public static void invokeBeanFactoryPostProcessors(
			ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {

    	
		
		Set<String> processedBeans = new HashSet<>();

		if (beanFactory instanceof BeanDefinitionRegistry) {
			BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
			List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();
            
            
			List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();
			for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
				if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
					BeanDefinitionRegistryPostProcessor registryProcessor =
							(BeanDefinitionRegistryPostProcessor) postProcessor;
					registryProcessor.postProcessBeanDefinitionRegistry(registry);
					registryProcessors.add(registryProcessor);
				}
				else {
					regularPostProcessors.add(postProcessor);
				}
			}

			
			
			
			
            
			List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();

			
			String[] postProcessorNames =
					beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
			for (String ppName : postProcessorNames) {
                
                
				if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					processedBeans.add(ppName);
				}
			}
			sortPostProcessors(currentRegistryProcessors, beanFactory);
			registryProcessors.addAll(currentRegistryProcessors);
            
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
			currentRegistryProcessors.clear();

			
			postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
			for (String ppName : postProcessorNames) {
                
                
				if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					processedBeans.add(ppName);
				}
			}
			sortPostProcessors(currentRegistryProcessors, beanFactory);
			registryProcessors.addAll(currentRegistryProcessors);
            
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
			currentRegistryProcessors.clear();

			
            
			boolean reiterate = true;
			while (reiterate) {
				reiterate = false;
				postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
				for (String ppName : postProcessorNames) {
					if (!processedBeans.contains(ppName)) {
                        
						currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
						processedBeans.add(ppName);
						reiterate = true;
					}
				}
                
				sortPostProcessors(currentRegistryProcessors, beanFactory);
				registryProcessors.addAll(currentRegistryProcessors);
                
				invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
				currentRegistryProcessors.clear();
			}

			
			invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
			invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
		}

		else {
			
			invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
		}

		
		
    
		String[] postProcessorNames =
				beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);

		
		
    
		List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
		List<String> orderedPostProcessorNames = new ArrayList<>();
		List<String> nonOrderedPostProcessorNames = new ArrayList<>();
		for (String ppName : postProcessorNames) {
			if (processedBeans.contains(ppName)) {
				
			}
			else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
				priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
			}
			else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
				orderedPostProcessorNames.add(ppName);
			}
			else {
				nonOrderedPostProcessorNames.add(ppName);
			}
		}

		
    	
		sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
    	
		invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);

		
    	
		List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
		for (String postProcessorName : orderedPostProcessorNames) {
			orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
		sortPostProcessors(orderedPostProcessors, beanFactory);
    	
		invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);

		
    	
		List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
		for (String postProcessorName : nonOrderedPostProcessorNames) {
			nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
    	
		invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);

		
		
		beanFactory.clearMetadataCache();
	}

```

> 整个过程无非就是获取实现了 BeanFactoryPostProcessor 接口的实例，调用其中的 postProcessBeanDefinitionRegistry() 或postProcessBeanFactory() 方法，调用之前进行优先级接口的一些排序
> 
> 我们需要知道的是最先加载的是实现了 BeanDefinitionRegistryPostProcessor 接口的组件（实际上 BeanDefinitionRegistryPostProcessor 实现的也是 BeanFactoryPostProcessor 这里 spring 通过 BeanDefinitionRegistryPostProcessor 将直接实现了 BeanFactoryPostProcessor 的实例进行区分），在 bean 工厂创建完成后就被加载了，先执行了其中的 postProcessBeanDefinitionRegistry()，再执行了 postProcessBeanFactory()
> 
> 然后执行的才是直接实现了 BeanFactoryPostProcessor 接口的实例中的 postProcessBeanFactory()

bean 工厂的典型应用就是 Configuration 配置类的后置处理，被 @Configuration 标注的配置类，会被 ConfigurationClassPostProcessor 进行后置处理，解析其中的配置，这也是属于 bean 工厂的后置处理，在工厂刷新创建成功后就先解析配置类中的配置，保证后续 bean 实例的准确创建，这其中的后置处理逻辑，调用的就是 ConfigurationClassPostProcessor 中的 postProcessBeanDefinitionRegistry() 方法（对 bean 工厂实行后置处理，加载配置）；

### 生命周期接口解析之 BeanPostProcessor 的注册

关于 bean 工厂的后置增强，前面已经说过了，接下来便是 bean 的后置增强过程

依旧从刷新工厂开始

```null
public ClassPathXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {

		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}

```

刷新工厂时，便进入后置增强逻辑

```null
.....
try {
    
    postProcessBeanFactory(beanFactory);

    
    invokeBeanFactoryPostProcessors(beanFactory);

    
    registerBeanPostProcessors(beanFactory);
    .....

```

> 关于 bean 的后置处理器的 **注册**，紧接在 bean 工厂后置处理逻辑执行完毕后调用

registerBeanPostProcessors() 源码如下，其 api 文档很关键

```null

	protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
		PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
	}

```

其中 PostProcessorRegistrationDelegate 也很关键，它代理执行了所有的后置增强器的功能，在 bean 工厂的后置处理中，也是使用的 PostProcessorRegistrationDelegate 来代理执行的

继续追踪 PostProcessorRegistrationDelegate 中的 registerBeanPostProcessors() 方法，大体上依然是遍历所有的 bean 后置处理，根据其实现的 PriorityOrdered 或 Ordered 接口进行排序

```null
public static void registerBeanPostProcessors(
    ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {

    
    String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);

    
    
    
    int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
    beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));

    
    
    List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
    List<BeanPostProcessor> internalPostProcessors = new ArrayList<>();
    List<String> orderedPostProcessorNames = new ArrayList<>();
    List<String> nonOrderedPostProcessorNames = new ArrayList<>();
    
    
    for (String ppName : postProcessorNames) {
        if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
            BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
            priorityOrderedPostProcessors.add(pp);
            if (pp instanceof MergedBeanDefinitionPostProcessor) {
                
                internalPostProcessors.add(pp);
            }
        }
        else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
            orderedPostProcessorNames.add(ppName);
        }
        else {
            
            nonOrderedPostProcessorNames.add(ppName);
        }
    }

    
    sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);

    
    List<BeanPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
    for (String ppName : orderedPostProcessorNames) {
        
        BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
        
        orderedPostProcessors.add(pp);
        if (pp instanceof MergedBeanDefinitionPostProcessor) {
            internalPostProcessors.add(pp);
        }
    }
    sortPostProcessors(orderedPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, orderedPostProcessors);

    
    List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
    for (String ppName : nonOrderedPostProcessorNames) {
        BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
        nonOrderedPostProcessors.add(pp);
        if (pp instanceof MergedBeanDefinitionPostProcessor) {
            internalPostProcessors.add(pp);
        }
    }
    registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);

    
    sortPostProcessors(internalPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, internalPostProcessors);

    
    
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));
}

```

registerBeanPostProcessors 源码如下

```null
private static void registerBeanPostProcessors(
			ConfigurableListableBeanFactory beanFactory, List<BeanPostProcessor> postProcessors) {
    	
		for (BeanPostProcessor postProcessor : postProcessors) {
			beanFactory.addBeanPostProcessor(postProcessor);
		}
	}

```

以上就是 bean 后置处理被注册的流程，所谓的 bean 后置处理器的注册，实际上就是将 bean 后置处理器进行分类排序然后保存到 bean 工厂中。

### 生命周期接口解析之 SmartInstantiationAwareBeanPostProcessor （一）

以上是 BeanPostProcesser 的注册，下面继续解析 BeanPostProcesser 中后置处理逻辑的执行时机

首先是常规流程

```null

ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beans2.xml");

```

```null

public ClassPathXmlApplicationContext(
    String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
    throws BeansException {

    super(parent);
    setConfigLocations(configLocations);
    if (refresh) {
        refresh();
    }
}

```

refresh() 内部

```null
.....
try {
    
    postProcessBeanFactory(beanFactory);

    
    invokeBeanFactoryPostProcessors(beanFactory);

    
    registerBeanPostProcessors(beanFactory);

    
    initMessageSource();

    
    initApplicationEventMulticaster();

    
    onRefresh();

    
    registerListeners();

    
    finishBeanFactoryInitialization(beanFactory);

    
    finishRefresh();
}
.......

```

跟踪 registerListeners() 源码

```null

protected void registerListeners() {
    
    for (ApplicationListener<?> listener : getApplicationListeners()) {
        getApplicationEventMulticaster().addApplicationListener(listener);
    }

    
    
    String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
    for (String listenerBeanName : listenerBeanNames) {
        getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
    }

    
    Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
    this.earlyApplicationEvents = null;
    if (earlyEventsToProcess != null) {
        for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
            getApplicationEventMulticaster().multicastEvent(earlyEvent);
        }
    }
}

```

深入追踪 getBeanNamesForType() 方法源码，如下

```null
private String[] doGetBeanNamesForType(ResolvableType type, boolean includeNonSingletons, boolean allowEagerInit) {
    List<String> result = new ArrayList<>();

    
    for (String beanName : this.beanDefinitionNames) {
        
        
        if (!isAlias(beanName)) {
            try {
                
                RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
                
                if (!mbd.isAbstract() && (allowEagerInit ||
                                          (mbd.hasBeanClass() || !mbd.isLazyInit() || isAllowEagerClassLoading()) &&
                                          !requiresEagerInitForType(mbd.getFactoryBeanName()))) {
                    boolean isFactoryBean = isFactoryBean(beanName, mbd);
                    BeanDefinitionHolder dbd = mbd.getDecoratedDefinition();
                    boolean matchFound = false;
                    boolean allowFactoryBeanInit = allowEagerInit || containsSingleton(beanName);
                    boolean isNonLazyDecorated = dbd != null && !mbd.isLazyInit();
                    if (!isFactoryBean) {
                        if (includeNonSingletons || isSingleton(beanName, mbd, dbd)) {
                            
                            matchFound = isTypeMatch(beanName, type, allowFactoryBeanInit);
                        }
                    }
                    else  {
                        if (includeNonSingletons || isNonLazyDecorated ||
                            (allowFactoryBeanInit && isSingleton(beanName, mbd, dbd))) {
                            matchFound = isTypeMatch(beanName, type, allowFactoryBeanInit);
                        }
                        if (!matchFound) {
                            
                            beanName = FACTORY_BEAN_PREFIX + beanName;
                            matchFound = isTypeMatch(beanName, type, allowFactoryBeanInit);
                        }
                    }
                    if (matchFound) {
                        
                        result.add(beanName);
                    }
                }
            }
            catch (CannotLoadBeanClassException | BeanDefinitionStoreException ex) {
                if (allowEagerInit) {
                    throw ex;
                }
                
                LogMessage message = (ex instanceof CannotLoadBeanClassException) ?
                    LogMessage.format("Ignoring bean class loading failure for bean '%s'", beanName) :
                LogMessage.format("Ignoring unresolvable metadata in bean definition '%s'", beanName);
                logger.trace(message, ex);
                onSuppressedException(ex);
            }
        }
    }

```

追踪 isTypeMatch() 部分源码

```null
.....
if (predictedType == null) {
    
    predictedType = predictBeanType(beanName, mbd, typesToMatch);
    if (predictedType == null) {
        return false;
    }
}
......

```

跟踪 predictBeanType()

```null
@Override
@Nullable
protected Class<?> predictBeanType(String beanName, RootBeanDefinition mbd, Class<?>... typesToMatch) {
    Class<?> targetType = determineTargetType(beanName, mbd, typesToMatch);
    if (targetType != null && !mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        boolean matchingOnlyFactoryBean = typesToMatch.length == 1 && typesToMatch[0] == FactoryBean.class;
        
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            
            if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
                
                Class<?> predicted = ibp.predictBeanType(targetType, beanName);
                if (predicted != null &&
                    (!matchingOnlyFactoryBean || FactoryBean.class.isAssignableFrom(predicted))) {
                    return predicted;
                }
            }
        }
    }
    return targetType;
}

```

在这里，执行了 BeanPostProcesser 中的第一个处理器方法，为 SmartInstantiationAwareBeanPostProcessor 中的 predictBeanType() 方法

predictBeanType() 的 bean 类型的预测只针对还没有完成实例化创建的组件，主要作用在于组件实例化创建之前，还能够决定组件的类型，详细流程解析如下

主要逻辑在 doGetBeanNamesForType() 方法中， 部分源码如下

```null
.....

if (!isFactoryBean) {
    if (includeNonSingletons || isSingleton(beanName, mbd, dbd)) {
        matchFound = isTypeMatch(beanName, type, allowFactoryBeanInit);
    }
}
else  {
    
    if (includeNonSingletons || isNonLazyDecorated ||
        (allowFactoryBeanInit && isSingleton(beanName, mbd, dbd))) {
        matchFound = isTypeMatch(beanName, type, allowFactoryBeanInit);
    }
    if (!matchFound) {
        
        beanName = FACTORY_BEAN_PREFIX + beanName;
        matchFound = isTypeMatch(beanName, type, allowFactoryBeanInit);
    }
}
if (matchFound) {
    result.add(beanName);
}
......

```

追踪 isSingleton()

```null
private boolean isSingleton(String beanName, RootBeanDefinition mbd, @Nullable BeanDefinitionHolder dbd) {
    return (dbd != null ? mbd.isSingleton() : isSingleton(beanName));
}

```

上面的三目运算无非就是通过 bean 定义信息或者 beanName 来作为判断依据，因此继续深入 isSingleton()，首先是空参的 isSingleton()

```null

@Override
public boolean isSingleton() {
    return SCOPE_SINGLETON.equals(this.scope) || SCOPE_DEFAULT.equals(this.scope);
}

```

然后是有参的 isSingleton()

```null
@Override
public boolean isSingleton(String name) throws NoSuchBeanDefinitionException {
    String beanName = transformedBeanName(name);

    
    Object beanInstance = getSingleton(beanName, false);
    if (beanInstance != null) {
        if (beanInstance instanceof FactoryBean) {
            return (BeanFactoryUtils.isFactoryDereference(name) || ((FactoryBean<?>) beanInstance).isSingleton());
        }
        else {
            return !BeanFactoryUtils.isFactoryDereference(name);
        }
    }

    
    BeanFactory parentBeanFactory = getParentBeanFactory();
    if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
        
        return parentBeanFactory.isSingleton(originalBeanName(name));
    }

    RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);

    
    if (mbd.isSingleton()) {
        if (isFactoryBean(beanName, mbd)) {
            if (BeanFactoryUtils.isFactoryDereference(name)) {
                return true;
            }
            FactoryBean<?> factoryBean = (FactoryBean<?>) getBean(FACTORY_BEAN_PREFIX + beanName);
            return factoryBean.isSingleton();
        }
        else {
            return !BeanFactoryUtils.isFactoryDereference(name);
        }
    }
    else {
        return false;
    }
}

```

以上就说明了，predictBeanType() 只会对当前未实例化的组件进行类型预测

总结

> SmartInstantiationAwareBeanPostProcessor 中的 predictBeanType() 方法在 bean 的生命周期中被首次调用，且会被调用多次，因为后续 getBeanNamesForType() 方法还会执行且只要 getBeanNamesForType() 执行，predictBeanType() 就会被调用

### 生命周期接口解析之 InstantiationAwareBeanPostProcessor（一）

SmartInstantiationAwareBeanPostProcessor 执行后，下一个就是 InstantiationAwareBeanPostProcessor，继续追踪源码

```null

public ClassPathXmlApplicationContext(
    String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
    throws BeansException {

    super(parent);
    setConfigLocations(configLocations);
    if (refresh) {
        refresh();
    }
}

```

来到熟悉的刷新容器十二大步

```null
......
try {
    
    postProcessBeanFactory(beanFactory);

    
    invokeBeanFactoryPostProcessors(beanFactory);

    
    registerBeanPostProcessors(beanFactory);

    
    initMessageSource();

    
    initApplicationEventMulticaster();

    
    onRefresh();

    
    registerListeners();

    
    finishBeanFactoryInitialization(beanFactory);

    
    finishRefresh();
}
......

```

这次的堆栈点停留到了 finishBeanFactoryInitialization() 处，继续追踪

```null

beanFactory.preInstantiateSingletons();

```

之后就是根据 bean 名称获取 bean 定义信息，对 bean 定义信息进行一系列判断，最后调用 getBean() 创建 bean 实例等逻辑，相关源码如下

```null


for (String beanName : beanNames) {
    
    RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
    
    if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
        
        if (isFactoryBean(beanName)) {
            Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
            if (bean instanceof FactoryBean) {
                final FactoryBean<?> factory = (FactoryBean<?>) bean;
                boolean isEagerInit;
                if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                    isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
                                                                ((SmartFactoryBean<?>) factory)::isEagerInit,
                                                                getAccessControlContext());
                }
                else {
                    isEagerInit = (factory instanceof SmartFactoryBean &&
                                   ((SmartFactoryBean<?>) factory).isEagerInit());
                }
                if (isEagerInit) {
                    getBean(beanName);
                }
            }
        }
        else {
            
            getBean(beanName);
        }
    }
}

```

在 getBean() 的调用过程中，一定会在 getSingleton() 的 lambda 表达式中调用 createBean() 而 InstantiationAwareBeanPostProcessor 后置处理器起作用的关键逻辑也在 createBean() 中

```null
@Override
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
    throws BeanCreationException {

    if (logger.isTraceEnabled()) {
        logger.trace("Creating instance of bean '" + beanName + "'");
    }
    RootBeanDefinition mbdToUse = mbd;

    
    
    
    Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
    if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
        mbdToUse = new RootBeanDefinition(mbd);
        mbdToUse.setBeanClass(resolvedClass);
    }

    
    try {
        mbdToUse.prepareMethodOverrides();
    }
    catch (BeanDefinitionValidationException ex) {
        throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(),
                                               beanName, "Validation of method overrides failed", ex);
    }

    try {
        
        Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
        
        if (bean != null) {
            return bean;
        }
    }
    catch (Throwable ex) {
        throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
                                        "BeanPostProcessor before instantiation of bean failed", ex);
    }

    try {
        
        Object beanInstance = doCreateBean(beanName, mbdToUse, args);
        if (logger.isTraceEnabled()) {
            logger.trace("Finished creating instance of bean '" + beanName + "'");
        }
        return beanInstance;
    }
    catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
        
        
        throw ex;
    }
    catch (Throwable ex) {
        throw new BeanCreationException(
            mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", ex);
    }
}

```

跟踪 resolveBeforeInstantiation()

```null
@Nullable
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
    Object bean = null;
    if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
        
        if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
            Class<?> targetType = determineTargetType(beanName, mbd);
            
            if (targetType != null) {
                
                bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
                if (bean != null) {
                    
                    bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
                }
            }
        }
        mbd.beforeInstantiationResolved = (bean != null);
    }
    return bean;
}

```

继续深入 applyBeanPostProcessorsBeforeInstantiation()

```null
@Nullable
protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
    
    for (BeanPostProcessor bp : getBeanPostProcessors()) {
        if (bp instanceof InstantiationAwareBeanPostProcessor) {
            
            
            
            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
            Object result = ibp.postProcessBeforeInstantiation(beanClass, beanName);
            if (result != null) {
                return result;
            }
        }
    }
    return null;
}

```

以上，分析出了 BeanPostProcesser 系列中，第二个执行的后置处理方法，也就是 InstantiationAwareBeanPostProcessor 中的 postProcessBeforeInstantiation()

### 生命周期接口解析之 SmartInstantiationAwareBeanPostProcessor（二）

跟 SmartInstantiationAwareBeanPostProcessor 接口相关的后置处理已经是多次出现了，第一次通过调用 predictBeanType() 进行了类型预测，后续在调用 doGetBeanNamesForType() 方法时，predictBeanType() 也会继续调用

而这次，SmartInstantiationAwareBeanPostProcessor 中的 determineCandidateConstructors() 在 InstantiationAwareBeanPostProcessor 的 postProcessBeforeInstantiation() 执行后，被调用，成为了 BeanPostProcesser 后置处理逻辑中，第三个执行的逻辑，下面对该逻辑的执行流程进行梳理

首先，构造容器实例，刷新工厂

```null
public ClassPathXmlApplicationContext(
    String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
    throws BeansException {

    super(parent);
    setConfigLocations(configLocations);
    if (refresh) {
        refresh();
    }
}

```

执行工厂刷新的十二大步

```null

@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        
        prepareRefresh();

        
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        
        prepareBeanFactory(beanFactory);

        try {
            
            postProcessBeanFactory(beanFactory);

            
            invokeBeanFactoryPostProcessors(beanFactory);

            
            registerBeanPostProcessors(beanFactory);

            
            initMessageSource();

            
            initApplicationEventMulticaster();

            
            onRefresh();

            
            registerListeners();

            
            finishBeanFactoryInitialization(beanFactory);

            
            finishRefresh();
        }
        ......

```

这次的堆栈点任然停留到了 finishBeanFactoryInitialization()，往后就是创建 bean 实例的过程，getBean() >>> doGetBean() >>> getSingleton() >>> lambad 表达式调用 createBean()

在 createBean() 中，又主要通过 doCreateBean() 来获取 bean 实例，doCreateBean() 中，继续调用 createBeanInstance()，第三个后置处理逻辑也在 createBeanInstance() 被执行

```null
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
    
    Class<?> beanClass = resolveBeanClass(mbd, beanName);

    
    if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
        throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                        "Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
    }

    
    
    Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
    if (instanceSupplier != null) {
        return obtainFromSupplier(instanceSupplier, beanName);
    }

    
    if (mbd.getFactoryMethodName() != null) {
        return instantiateUsingFactoryMethod(beanName, mbd, args);
    }

    
    boolean resolved = false;
    boolean autowireNecessary = false;
    if (args == null) {
        synchronized (mbd.constructorArgumentLock) {
            if (mbd.resolvedConstructorOrFactoryMethod != null) {
                resolved = true;
                autowireNecessary = mbd.constructorArgumentsResolved;
            }
        }
    }
    if (resolved) {
        if (autowireNecessary) {
            return autowireConstructor(beanName, mbd, null, null);
        }
        else {
            return instantiateBean(beanName, mbd);
        }
    }

    
    Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
    if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
        mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
        
        return autowireConstructor(beanName, mbd, ctors, args);
    }

    
    ctors = mbd.getPreferredConstructors();
    if (ctors != null) {
        return autowireConstructor(beanName, mbd, ctors, null);
    }

    
    return instantiateBean(beanName, mbd);
}

```

堆栈停留在了 determineConstructorsFromBeanPostProcessors() 处，其内部源码如下

```null
@Nullable
protected Constructor<?>[] determineConstructorsFromBeanPostProcessors(@Nullable Class<?> beanClass, String beanName)
    throws BeansException {

    if (beanClass != null && hasInstantiationAwareBeanPostProcessors()) {
        
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            
            if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
                
                Constructor<?>[] ctors = ibp.determineCandidateConstructors(beanClass, beanName);
                if (ctors != null) {
                    return ctors;
                }
            }
        }
    }
    return null;
}

```

以上就是 bean 后置处理的第三次执行，为 SmartInstantiationAwareBeanPostProcessor 中的 determineCandidateConstructors()，主要用于在 bean 调用构造器实例化之前，通过 determineCandidateConstructors() 来使用自定义的构造器创建 bean 实例

### 生命周期接口解析之 MergedBeanDefinitionPostProcessor（一）

下面进入 bean 生命周期过程中，bean 后置处理的第四次执行流程分析

前面的流程和上面 SmartInstantiationAwareBeanPostProcessor 第三次执行的流程过程一致，直到 doCreateBean() 中，第四次 bean 后置处理逻辑被执行，它是在 createBeanInstance() 调用，也就是 bean 实例化之后执行的

```null
........

BeanWrapper instanceWrapper = null;
if (mbd.isSingleton()) {
    instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
}
if (instanceWrapper == null) {
    instanceWrapper = createBeanInstance(beanName, mbd, args);
}
final Object bean = instanceWrapper.getWrappedInstance();
Class<?> beanType = instanceWrapper.getWrappedClass();
if (beanType != NullBean.class) {
    mbd.resolvedTargetType = beanType;
}


synchronized (mbd.postProcessingLock) {
    if (!mbd.postProcessed) {
        try {
            
            applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
        }
        catch (Throwable ex) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                            "Post-processing of merged bean definition failed", ex);
        }
        mbd.postProcessed = true;
    }
}
......

```

跟踪 applyMergedBeanDefinitionPostProcessors()

```null
protected void applyMergedBeanDefinitionPostProcessors(RootBeanDefinition mbd, Class<?> beanType, String beanName) {
    for (BeanPostProcessor bp : getBeanPostProcessors()) {
        if (bp instanceof MergedBeanDefinitionPostProcessor) {
            MergedBeanDefinitionPostProcessor bdp = (MergedBeanDefinitionPostProcessor) bp;
            bdp.postProcessMergedBeanDefinition(mbd, beanType, beanName);
        }
    }
}

```

还是一样的逻辑，遍历获取 bean 后置处理器，判断类型为 MergedBeanDefinitionPostProcessor，调用其中的 postProcessMergedBeanDefinition()，目的在于再次修改 bean 的定义信息

### 生命周期接口解析之 InstantiationAwareBeanPostProcessor（二）

bean 后置处理的第五次调用，继续来到了 InstantiationAwareBeanPostProcessor 中，调用的位置任然在 doCreateBean() 中，且紧接着上面的 MergedBeanDefinitionPostProcessor 调用，堆栈点位于 populateBean() 处，说明本次 bean 后置处理的调用发生在 populateBean() 内部，和 bean 属性的赋值有关，堆栈点如下

```null

Object exposedObject = bean;
try {
    
    populateBean(beanName, mbd, instanceWrapper);
    exposedObject = initializeBean(beanName, exposedObject, mbd);
}
catch (Throwable ex) {
    if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
        throw (BeanCreationException) ex;
    }
    else {
        throw new BeanCreationException(
            mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
    }
}

```

追踪堆栈点，部分源码如下

```null

boolean continueWithPropertyPopulation = true;

if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
    for (BeanPostProcessor bp : getBeanPostProcessors()) {
        if (bp instanceof InstantiationAwareBeanPostProcessor) {
            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
            if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                continueWithPropertyPopulation = false;
                break;
            }
        }
    }
}
if (!continueWithPropertyPopulation) {
			return;
}

```

遍历所有的 beam 后置处理，判断其中的 InstantiationAwareBeanPostProcessor，调用其 postProcessAfterInstantiation() 方法，完成 bean 后置处理逻辑的第五次调用，目的在于通过 postProcessAfterInstantiation() 在 bean 实例化之后，spring 属性赋值之前，可以自定义的对 bean 实例执行一些操作

需要注意的是，如果我们在 postProcessAfterInstantiation() 规定的返回值为 false，spring 自己的后续的属性赋值逻辑将不会执行，populateBean() 方法直接退出

### 生命周期接口解析之 InstantiationAwareBeanPostProcessor（三）

如果上面 postProcessAfterInstantiation 返回了 true，则会继续执行 InstantiationAwareBeanPostProcessor 中的 postProcessProperties() 方法，这是 bean 后置处理逻辑的第六次执行且紧接上面的 postProcessAfterInstantiation() 执行，部分源码如下

```null
........
PropertyDescriptor[] filteredPds = null;
if (hasInstAwareBpps) {
    if (pvs == null) {
        
        pvs = mbd.getPropertyValues();
    }
    
    for (BeanPostProcessor bp : getBeanPostProcessors()) {
        
        if (bp instanceof InstantiationAwareBeanPostProcessor) {
            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
            
            PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
            if (pvsToUse == null) {
                if (filteredPds == null) {
                    filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
                }
                pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
                if (pvsToUse == null) {
                    return;
                }
            }
            pvs = pvsToUse;
        }
    }
}
if (needsDepCheck) {
    if (filteredPds == null) {
        filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
    }
    checkDependencies(beanName, mbd, filteredPds, pvs);
}

if (pvs != null) {
    
    applyPropertyValues(beanName, mbd, bw, pvs);
	}
}

```

本次调用的 bean 后置处理 postProcessProperties() 方法，主要作用在于：在工厂将给定的属性值应用于给定的 bean 之前对给定的属性值进行处理；其中，方法参数 PropertyValues 类型的 pvs 封装了所有的属性信息

@Autowired 的自动装配也发生于此，主要由 AutowiredAnnotationBeanPostProcessor 在其 postProcessProperties() 方法中实现，注意

@Autowired 自动装配在 postProcessProperties() 不是直接给 bean 实例的属性直接赋值，而是会返回一个 PropertyValues，这是属性键值对的集合，后续在 applyPropertyValues() 中才是对 bean 实例赋值的操作，而这个赋值操作，依靠的是 bean 实例被包装后的 org.springframework.beans.BeanWrapper 实现

### 生命周期接口解析之 BeanPostProcessor（一）

经过前面几次的调用，后置处理逻辑已经覆盖到了 bean 实例的创建到 bean 实例的属性赋值方面，在属性赋值结束后，紧接了执行了第七次的 bean 后置处理逻辑，直接来自于 BeanPostProcessor 中的 postProcessBeforeInitialization() 方法

该后置处理逻辑的执行，依然在 doCreateBean() 中，部分源码如下

```null
.....
    
    Object exposedObject = bean;
try {
    populateBean(beanName, mbd, instanceWrapper);
    
    exposedObject = initializeBean(beanName, exposedObject, mbd);
}
catch (Throwable ex) {
    if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
        throw (BeanCreationException) ex;
    }
    else {
        throw new BeanCreationException(
            mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
    }
}
.....

```

追踪 initializeBean()

```null
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
		if (System.getSecurityManager() != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
				invokeAwareMethods(beanName, bean);
				return null;
			}, getAccessControlContext());
		}
		else {
            
			invokeAwareMethods(beanName, bean);
		}

		Object wrappedBean = bean;
		if (mbd == null || !mbd.isSynthetic()) {
            
			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
		}

		try {
            
			invokeInitMethods(beanName, wrappedBean, mbd);
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					(mbd != null ? mbd.getResourceDescription() : null),
					beanName, "Invocation of init method failed", ex);
		}
		if (mbd == null || !mbd.isSynthetic()) {
            
			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
		}

		return wrappedBean;
	}

```

其中，还会执行 invokeInitMethods() 方法，凡是实现了 InitializingBean 接口或者指定了自定义的初始化方法的 bean，其中的初始化方法就在 invokeInitMethods() 逻辑中被调用，invokeInitMethods() 源码如下

```null

protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd)
    throws Throwable {

    boolean isInitializingBean = (bean instanceof InitializingBean);
    if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
        if (logger.isTraceEnabled()) {
            logger.trace("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
        }
        if (System.getSecurityManager() != null) {
            try {
                AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
                    ((InitializingBean) bean).afterPropertiesSet();
                    return null;
                }, getAccessControlContext());
            }
            catch (PrivilegedActionException pae) {
                throw pae.getException();
            }
        }
        else {
            
            ((InitializingBean) bean).afterPropertiesSet();
        }
    }

    
    if (mbd != null && bean.getClass() != NullBean.class) {
        String initMethodName = mbd.getInitMethodName();
        if (StringUtils.hasLength(initMethodName) &&
            !(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
            !mbd.isExternallyManagedInitMethod(initMethodName)) {
            
            invokeCustomInitMethod(beanName, bean, mbd);
        }
    }
}

```

> 自定义方法的设置如下
> 
> ```null
> 
> rootBeanDefinition.setInitMethodName("xxx");
> 
> ```
> 
> 或者通过 @Bean 注解的 initMethod 属性进行设置

最后执行 applyBeanPostProcessorsAfterInitialization() 方法，其源码如下

```null
@Override
public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
    throws BeansException {

    Object result = existingBean;
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
        Object current = processor.postProcessBeforeInitialization(result, beanName);
        if (current == null) {
            return result;
        }
        result = current;
    }
    return result;
}

```

执行了 BeanPostProcessor 中的 postProcessBeforeInitialization() 方法，作用在于，即使当前 bean 已经实例化且属性被赋值，仍然可以通过该方法，返回一个自定义的 bean 实例，也就是说，在 bean 初始化的环节中，之前的 bean 又可能被替代或者改变。

之后，还会执行 applyBeanPostProcessorsAfterInitialization()，调用的是 BeanPostProcessor 中的 postProcessAfterInitialization() 方法，这两个方法，分别在 invokeInitMethods() 调用前后被调用，因此一个叫初始化之前（BeforeInitialization），一个叫初始化之后（AfterInitialization）。

以上流程都在 initializeBean() 方法中执行完毕，最终会返回一个 exposedObject 实例，这就是最终被创建成功的 bean 实例对象，部分源码如下

```null
......
try {
    populateBean(beanName, mbd, instanceWrapper);
    exposedObject = initializeBean(beanName, exposedObject, mbd);
}
.....

```

### spring bean 生命周期总结

以上的这些整个逻辑大部分都在 doCreateBean() 中被执行，doCreateBean() 实际上被 createBean() 调用，createBean 又在 getSingleton() 的 lambda 表达式中调用，而这一切，都只是因为在 DefaultListableBeanFactory 的 preInstantiateSingletons() 方法中，遍历 bean 的定义信息，获取 bean 实例时，调用的 getBean() 方法引起的；preInstantiateSingletons() 又是被 spring 容器创建时，刷新工厂的十二大步中的 finishBeanFactoryInitialization() 调用，最终，工厂完成刷新，spring 容器被创建，我们才可以直接使用其中的 bean 实例。

至此，spring 中，bean 创建的整个生命周期的执行流程就完毕了，对于 spring bean 的整个流程，大致上可分为

> 实例化，主要方法：createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object\[\] args)
> 
> 属性填充，主要方法：populateBean(beanName, mbd, instanceWrapper)
> 
> 初始化，主要方法：initializeBean(beanName, exposedObject, mbd);
> 
> 销毁，spring 容器关闭时，会调用销毁方法，该方法可以通过标注注解，实现接口，或者自定义来规定，可参考相关博客：[https://cloud.tencent.com/developer/article/1730097](https://cloud.tencent.com/developer/article/1730097)
> 
> 在这几个生命周期阶段中，各种后置处理器就穿插其间，影响 bean 的整个创建和初始化过程

spring 其他功能的实现，也是基于此生命周期来的，例如 spring 的事务，那就一定有一个事务相关的后置处理器，会检查方法是否标注了 @Transactional 注解，在处理器的后置处理逻辑中，进行一些事务的操作；又或者是 @Controller 注解，那也是在被标注了 @Controller 注解的组件在实例化时，通过执行生命周期中的某些环节的逻辑来实现客户端请求到指定目标方法的功能的。

流程图总结

![](https://img-blog.csdnimg.cn/b8639aa2468c47f0a2de3f54de245d9a.jpg?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ZGK5Yir5pe25YWJ,size_20,color_FFFFFF,t_70,g_se,x_16)

补充

> 想要查看 spring 中注册了那些组件，可以直接给 RootBeanDefinition 的构造器打上断点
> 
> 又或者是给刷新容器方法 refresh() 的最后一行打上断点来分析 spring 的整个流程

* * *

bean 初始化流程梳理
------------

下面，对 bean 的初始化流程，从整体角度进行一个梳理。

spring bean 的初始化流程，发生在刷新工厂十二步的 finishBeanFactoryInitialization() 方法处，最终在 preInstantiateSingletons() 方法中被执行，源码如下

```null

for (String beanName : beanNames) {
    
    RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
    
    if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
         
        if (isFactoryBean(beanName)) {
            Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
            if (bean instanceof FactoryBean) {
                final FactoryBean<?> factory = (FactoryBean<?>) bean;
                boolean isEagerInit;
                if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                    isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
                                                                ((SmartFactoryBean<?>) factory)::isEagerInit,
                                                                getAccessControlContext());
                }
                else {
                    isEagerInit = (factory instanceof SmartFactoryBean &&
                                   ((SmartFactoryBean<?>) factory).isEagerInit());
                }
                if (isEagerInit) {
                    getBean(beanName);
                }
            }
        }
        else {
            getBean(beanName);
        }
    }
}

```

这里先研究工厂 bean 的初始化流程，在 bean 的定义信息满足了单实例，非抽象类，非懒加载的条件后，紧接着就是判断是否工厂 bean 的流程，追踪源码如下

```null
@Override
public boolean isFactoryBean(String name) throws NoSuchBeanDefinitionException {
    String beanName = transformedBeanName(name);
    
    Object beanInstance = getSingleton(beanName, false);
    if (beanInstance != null) {
        
        return (beanInstance instanceof FactoryBean);
    }
    
    if (!containsBeanDefinition(beanName) && getParentBeanFactory() instanceof ConfigurableBeanFactory) {
        
        return ((ConfigurableBeanFactory) getParentBeanFactory()).isFactoryBean(name);
    }
    
    return isFactoryBean(beanName, getMergedLocalBeanDefinition(beanName));
}

```

isFactoryBean() 源码如下

```null
protected boolean isFactoryBean(String beanName, RootBeanDefinition mbd) {
    
    Boolean result = mbd.isFactoryBean;
    if (result == null) {
        
        Class<?> beanType = predictBeanType(beanName, mbd, FactoryBean.class);
        
        
        result = (beanType != null && FactoryBean.class.isAssignableFrom(beanType));
        mbd.isFactoryBean = result;
    }
    return result;
}

```

继续回到 preInstantiateSingletons() 中

```null

for (String beanName : beanNames) {
    
    RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
    
    if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
         
        if (isFactoryBean(beanName)) {
            
            
            
            Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
            if (bean instanceof FactoryBean) {
                final FactoryBean<?> factory = (FactoryBean<?>) bean;
                boolean isEagerInit;
                if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                    isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
                                                                ((SmartFactoryBean<?>) factory)::isEagerInit,
                                                                getAccessControlContext());
                }
                else {
                    
                    isEagerInit = (factory instanceof SmartFactoryBean &&
                                   ((SmartFactoryBean<?>) factory).isEagerInit());
                }
                if (isEagerInit) {
                    getBean(beanName);
                }
            }
        }
        else {
            getBean(beanName);
        }
    }
}

```

工厂 bean 被创建后，其 getObject() 方法并不会立即执行，而是从容器中获取 getObject() 方法对应返回值类型的 bean 实例时，调用 getObject() 创建 bean 实例，这也是 FactoryBean 的作用；当我们从容器中获取 bean 实例，调用的是 getBean()，此时 FactoryBean 返回对应实例的 getObject() 逻辑会在 getBean() 中执行；下面，通过 debug 调试源码来追踪其完整流程

首先，从 spring 容器中获取一个 Person.class 类型的实例，该实例通过 FactoryBean 来得到

```null
Person bean = applicationContext.getBean(Person.class);

```

追踪 getBean() 源码

```null
@Override
public <T> T getBean(Class<T> requiredType) throws BeansException {
    assertBeanFactoryActive();
    return getBeanFactory().getBean(requiredType);
}

```

> 从工厂中获取 bean

追踪源码

```null
@Override
public <T> T getBean(Class<T> requiredType) throws BeansException {
    return getBean(requiredType, (Object[]) null);
}

```

> 通过类型从工厂中获取 bean 实例

```null
@Override
public <T> T getBean(Class<T> requiredType, @Nullable Object... args) throws BeansException {
    Assert.notNull(requiredType, "Required type must not be null");
    
    Object resolved = resolveBean(ResolvableType.forRawClass(requiredType), args, false);
    if (resolved == null) {
        throw new NoSuchBeanDefinitionException(requiredType);
    }
    return (T) resolved;
}

```

追踪 resolveBean()

```null
private <T> T resolveBean(ResolvableType requiredType, @Nullable Object[] args, boolean nonUniqueAsNull) {
    
    NamedBeanHolder<T> namedBean = resolveNamedBean(requiredType, args, nonUniqueAsNull);
    if (namedBean != null) {
        
        return namedBean.getBeanInstance();
    }
    BeanFactory parent = getParentBeanFactory();
    if (parent instanceof DefaultListableBeanFactory) {
        return ((DefaultListableBeanFactory) parent).resolveBean(requiredType, args, nonUniqueAsNull);
    }
    else if (parent != null) {
        ObjectProvider<T> parentProvider = parent.getBeanProvider(requiredType);
        if (args != null) {
            return parentProvider.getObject(args);
        }
        else {
            return (nonUniqueAsNull ? parentProvider.getIfUnique() : parentProvider.getIfAvailable());
        }
    }
    return null;
}

```

追踪 resolveNamedBean(requiredType, args, nonUniqueAsNull)，部分源码如下

```null
private <T> NamedBeanHolder<T> resolveNamedBean(
			ResolvableType requiredType, @Nullable Object[] args, boolean nonUniqueAsNull) throws BeansException {

		Assert.notNull(requiredType, "Required type must not be null");
		String[] candidateNames = getBeanNamesForType(requiredType);
		.....

```

又来到了 getBeanNamesForType() 根据类型获取 beanName 的环节，在在这里面会遍历得到每一个 bean 定义信息；此时若存在工厂 bean，则会判断工厂 bean 类中，getObject() 方法的返回值类型，该类型又由工厂 bean 类中的 getObjectType() 方法获取到，将其 beanName 加上 & 前缀作为返回值；如果是工厂 bean，在 getBeanNamesForType() 方法中，会调用 isTypeMatch() 方法来判断是否符合方法参数中的 requiredType 类型，isTypeMatch() 部分源码如下

```null
protected boolean isTypeMatch(String name, ResolvableType typeToMatch, boolean allowFactoryBeanInit)
    throws NoSuchBeanDefinitionException {

    String beanName = transformedBeanName(name);
    boolean isFactoryDereference = BeanFactoryUtils.isFactoryDereference(name);

    
    Object beanInstance = getSingleton(beanName, false);
    if (beanInstance != null && beanInstance.getClass() != NullBean.class) {
        
        if (beanInstance instanceof FactoryBean) {
            if (!isFactoryDereference) {
                
                Class<?> type = getTypeForFactoryBean((FactoryBean<?>) beanInstance);
                
                return (type != null && typeToMatch.isAssignableFrom(type));
            }
            else {
                return typeToMatch.isInstance(beanInstance);
            }
        }
        .......

```

> 如果工厂 bean 的返回类型符合目标 bean 的类型，那么最终 isTypeMatch() 会返回 true，这也代表 getBeanNamesForType() 方法的返回值中，会有对应的工厂 bean 的 beanName【工厂 bean 前缀(&) + beanName】

退回到 resolveNamedBean() 方法中

```null
private <T> NamedBeanHolder<T> resolveNamedBean(
			ResolvableType requiredType, @Nullable Object[] args, boolean nonUniqueAsNull) throws BeansException {

		Assert.notNull(requiredType, "Required type must not be null");
		String[] candidateNames = getBeanNamesForType(requiredType);
.....
    
    if (candidateNames.length == 1) {
        String beanName = candidateNames[0];
        
        return new NamedBeanHolder<>(beanName, (T) getBean(beanName, requiredType.toClass(), args));
    }
.....

```

调用 getBean() 方法

```null
public <T> T getBean(String name, @Nullable Class<T> requiredType, @Nullable Object... args)
    throws BeansException {

    return doGetBean(name, requiredType, args, false);
}

```

继续追踪

```null
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
                          @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

    final String beanName = transformedBeanName(name);
    Object bean;

    
    Object sharedInstance = getSingleton(beanName);
    if (sharedInstance != null && args == null) {
        if (logger.isTraceEnabled()) {
            if (isSingletonCurrentlyInCreation(beanName)) {
                logger.trace("Returning eagerly cached instance of singleton bean '" + beanName +
                             "' that is not fully initialized yet - a consequence of a circular reference");
            }
            else {
                logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
            }
        }
        
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
    }
    ......

```

继续追踪

```null




if (!(beanInstance instanceof FactoryBean)) {
    return beanInstance;
}

Object object = null;
if (mbd != null) {
    mbd.isFactoryBean = true;
}
else {
    
    object = getCachedObjectForFactoryBean(beanName);
}

if (object == null) {
    
    FactoryBean<?> factory = (FactoryBean<?>) beanInstance;
    
    if (mbd == null && containsBeanDefinition(beanName)) {
        mbd = getMergedLocalBeanDefinition(beanName);
    }
    boolean synthetic = (mbd != null && mbd.isSynthetic());
    
    object = getObjectFromFactoryBean(factory, beanName, !synthetic);
}
return object;
}

```

追踪 getObjectFromFactoryBean()，部分源码如下

```null
protected Object getObjectFromFactoryBean(FactoryBean<?> factory, String beanName, boolean shouldPostProcess) {
    if (factory.isSingleton() && containsSingleton(beanName)) {
        synchronized (getSingletonMutex()) {
            
            Object object = this.factoryBeanObjectCache.get(beanName);
            if (object == null) {
                
                object = doGetObjectFromFactoryBean(factory, beanName);
                
                
                Object alreadyThere = this.factoryBeanObjectCache.get(beanName);
.......
    	
    	if (containsSingleton(beanName)) {
							this.factoryBeanObjectCache.put(beanName, object);
		}

```

追踪 doGetObjectFromFactoryBean()，部分源码如下

```null
private Object doGetObjectFromFactoryBean(final FactoryBean<?> factory, final String beanName)
    throws BeanCreationException {

    Object object;
    try {
        if (System.getSecurityManager() != null) {
            AccessControlContext acc = getAccessControlContext();
            try {
                object = AccessController.doPrivileged((PrivilegedExceptionAction<Object>) factory::getObject, acc);
            }
            catch (PrivilegedActionException pae) {
                throw pae.getException();
            }
        }
        else {
            
            object = factory.getObject();
        }
    }

```

以上，整个工厂 bean 获取实例的大致流程就结束了。

工厂 bean 的流程完成后，剩下的就是普通的单实例 bean 的初始化流程；经过 refresh() >>> finishBeanFactoryInitialization() >>> beanFactory.preInstantiateSingletons() 又来到了下面熟悉的流程

```null
@Override
public void preInstantiateSingletons() throws BeansException {
    if (logger.isTraceEnabled()) {
        logger.trace("Pre-instantiating singletons in " + this);
    }

    
    
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

    
    for (String beanName : beanNames) {
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
            
            if (isFactoryBean(beanName)) {
                Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
                if (bean instanceof FactoryBean) {
                    final FactoryBean<?> factory = (FactoryBean<?>) bean;
                    boolean isEagerInit;
                    if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                        isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
                                                                    ((SmartFactoryBean<?>) factory)::isEagerInit,
                                                                    getAccessControlContext());
                    }
                    else {
                        
                        isEagerInit = (factory instanceof SmartFactoryBean &&
                                       ((SmartFactoryBean<?>) factory).isEagerInit());
                    }
                    if (isEagerInit) {
                        getBean(beanName);
                    }
                }
            }
            else {
                
                getBean(beanName);
            }
        }
    }

    
    for (String beanName : beanNames) {
        Object singletonInstance = getSingleton(beanName);
        if (singletonInstance instanceof SmartInitializingSingleton) {
            final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
            if (System.getSecurityManager() != null) {
                AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
                    smartSingleton.afterSingletonsInstantiated();
                    return null;
                }, getAccessControlContext());
            }
            else {
                smartSingleton.afterSingletonsInstantiated();
            }
        }
    }
}

```

getBean() 底层继续调用了 doGetBean()，其源码如下

```null
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
                          @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

    final String beanName = transformedBeanName(name);
    Object bean;

    
    Object sharedInstance = getSingleton(beanName);
    if (sharedInstance != null && args == null) {
        if (logger.isTraceEnabled()) {
            if (isSingletonCurrentlyInCreation(beanName)) {
                logger.trace("Returning eagerly cached instance of singleton bean '" + beanName +
                             "' that is not fully initialized yet - a consequence of a circular reference");
            }
            else {
                logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
            }
        }
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
    }
    else {
        
        if (isPrototypeCurrentlyInCreation(beanName)) {
            throw new BeanCurrentlyInCreationException(beanName);
        }

        
        
        BeanFactory parentBeanFactory = getParentBeanFactory();
        if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
            
            String nameToLookup = originalBeanName(name);
            if (parentBeanFactory instanceof AbstractBeanFactory) {
                return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
                    nameToLookup, requiredType, args, typeCheckOnly);
            }
            else if (args != null) {
                
                return (T) parentBeanFactory.getBean(nameToLookup, args);
            }
            else if (requiredType != null) {
                
                return parentBeanFactory.getBean(nameToLookup, requiredType);
            }
            else {
                return (T) parentBeanFactory.getBean(nameToLookup);
            }
        }

        
        if (!typeCheckOnly) {
            
            markBeanAsCreated(beanName);
        }

        try {
            
            final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
            checkMergedBeanDefinition(mbd, beanName, args);

            
            String[] dependsOn = mbd.getDependsOn();
            if (dependsOn != null) {
                for (String dep : dependsOn) {
                    if (isDependent(beanName, dep)) {
                        throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                                        "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
                    }
                    registerDependentBean(dep, beanName);
                    try {
                        
                        getBean(dep);
                    }
                    catch (NoSuchBeanDefinitionException ex) {
                        throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                                        "'" + beanName + "' depends on missing bean '" + dep + "'", ex);
                    }
                }
            }

            
            if (mbd.isSingleton()) {
                sharedInstance = getSingleton(beanName, () -> {
                    try {
                        return createBean(beanName, mbd, args);
                    }
                    catch (BeansException ex) {
                        
                        
                        
                        destroySingleton(beanName);
                        throw ex;
                    }
                });
                bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
            }

            
            else if (mbd.isPrototype()) {
                
                Object prototypeInstance = null;
                try {
                    beforePrototypeCreation(beanName);
                    prototypeInstance = createBean(beanName, mbd, args);
                }
                finally {
                    afterPrototypeCreation(beanName);
                }
                bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
            }

            
            else {
                String scopeName = mbd.getScope();
                final Scope scope = this.scopes.get(scopeName);
                if (scope == null) {
                    throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
                }
                try {
                    Object scopedInstance = scope.get(beanName, () -> {
                        beforePrototypeCreation(beanName);
                        try {
                            return createBean(beanName, mbd, args);
                        }
                        finally {
                            afterPrototypeCreation(beanName);
                        }
                    });
                    bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
                }
                catch (IllegalStateException ex) {
                    throw new BeanCreationException(beanName,
                                                    "Scope '" + scopeName + "' is not active for the current thread; consider " +
                                                    "defining a scoped proxy for this bean if you intend to refer to it from a singleton",
                                                    ex);
                }
            }
        }
        catch (BeansException ex) {
            cleanupAfterBeanCreationFailure(beanName);
            throw ex;
        }
    }

    
    if (requiredType != null && !requiredType.isInstance(bean)) {
        try {
            T convertedBean = getTypeConverter().convertIfNecessary(bean, requiredType);
            if (convertedBean == null) {
                throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
            }
            return convertedBean;
        }
        catch (TypeMismatchException ex) {
            if (logger.isTraceEnabled()) {
                logger.trace("Failed to convert bean '" + name + "' to required type '" +
                             ClassUtils.getQualifiedName(requiredType) + "'", ex);
            }
            throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
        }
    }
    return (T) bean;
}

```

其中，最开始调用的 getSingleton() 底层继续调用了一个双参的 getSingleton(beanName, true)，该方法的第二个布尔参数代表【是否允许早期引用】，该方法源码如下

```null

protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    
    Object singletonObject = this.singletonObjects.get(beanName); 
    
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return singletonObject;
}

```

按照正常流程，就会执行 lambda 表达式形式的单实例 bean 创建流程，首先，外部的 getSingleton() 调用

```null
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(beanName, "Bean name must not be null");
		synchronized (this.singletonObjects) {
            
			Object singletonObject = this.singletonObjects.get(beanName);
			if (singletonObject == null) {
				if (this.singletonsCurrentlyInDestruction) {
					throw new BeanCreationNotAllowedException(beanName,
							"Singleton bean creation not allowed while singletons of this factory are in destruction " +
							"(Do not request a bean from a BeanFactory in a destroy method implementation!)");
				}
				if (logger.isDebugEnabled()) {
					logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
				}
                
				beforeSingletonCreation(beanName);
				boolean newSingleton = false;
				boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
				if (recordSuppressedExceptions) {
					this.suppressedExceptions = new LinkedHashSet<>();
				}
				try {
                    
					singletonObject = singletonFactory.getObject();
					newSingleton = true;
				}
				catch (IllegalStateException ex) {
					
					
					singletonObject = this.singletonObjects.get(beanName);
					if (singletonObject == null) {
						throw ex;
					}
				}
				catch (BeanCreationException ex) {
					if (recordSuppressedExceptions) {
						for (Exception suppressedException : this.suppressedExceptions) {
							ex.addRelatedCause(suppressedException);
						}
					}
					throw ex;
				}
				finally {
					if (recordSuppressedExceptions) {
						this.suppressedExceptions = null;
					}
					afterSingletonCreation(beanName);
				}
				if (newSingleton) {
					addSingleton(beanName, singletonObject);
				}
			}
			return singletonObject;
		}
	}

```

> 如果单例池中没有单实例缓存，则会创建单实例，首先就进入 beforeSingletonCreation()
> 
> ```null
> protected void beforeSingletonCreation(String beanName) {
>  
>  
>  if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
>      throw new BeanCurrentlyInCreationException(beanName);
>  }
> }
> 
> ```
> 
> 其中，singletonsCurrentlyInCreation 会把当前要创建的对象名称保存起来，这个很关键

最后，getSingleton() 会调用 createBean() ，这其中经过 bean 的完整生命周期（实例化，后置处理器，属性赋值，初始化等）；最终创建出 bean 对象实例。

bean 初始化的流程图总结如下

![](https://img-blog.csdnimg.cn/17198a195140446ca12d8d6266673865.jpg?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5ZGK5Yir5pe25YWJ,size_20,color_FFFFFF,t_70,g_se,x_16)

* * *

spring 容器刷新流程梳理
---------------

正常情况下，使用 `AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(Config.class);`创建一个 spring 容器，之后便是容器刷新的十二大步

首先是手动通过 new 的方式创建了 spring 容器

```null
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
    
    this();
    
    register(componentClasses);
    
    refresh();
}

```

首先看 this() 空参调用环节

```null
public AnnotationConfigApplicationContext() {
    
    this.reader = new AnnotatedBeanDefinitionReader(this);
    this.scanner = new ClassPathBeanDefinitionScanner(this);
}

```

AnnotatedBeanDefinitionReader 的构造器底层调用的时另一个重写的构造器，如下

```null
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
    Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
    Assert.notNull(environment, "Environment must not be null");
    this.registry = registry;
    this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
    
    AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}

```

追踪 registerAnnotationConfigProcessors()

```null
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
    BeanDefinitionRegistry registry, @Nullable Object source) {

    DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
    if (beanFactory != null) {
        if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
            beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
        }
        if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
            beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
        }
    }

    Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(8);

    
    
    
    if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    
    if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
    }
    
    
    
    if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    
    
    if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition();
        try {
            def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,
                                                AnnotationConfigUtils.class.getClassLoader()));
        }
        catch (ClassNotFoundException ex) {
            throw new IllegalStateException(
                "Cannot load optional framework class: " + PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME, ex);
        }
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    
    if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
    }

    
    if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
    }

    return beanDefs;
}

```

> 以上是创建读取器的过程，主要是注册一些底层的核心组件 bean 定义信息

紧接着，又创建了一个扫描器 `this.scanner = new ClassPathBeanDefinitionScanner(this);`，用于准备环境变量等信息

以上是空参构造器 this() 的执行流程，接着便是 register(componentClasses) 的流程

先跟踪第一次 register()，参数直接传入了主配置类，使用 this() 中构建好的 reader 读取主配置类

```null
@Override
public void register(Class<?>... componentClasses) {
    Assert.notEmpty(componentClasses, "At least one component class must be specified");
    
    this.reader.register(componentClasses);
}

```

继续跟踪，底层调用的是 doRegisterBean()

```null
private <T> void doRegisterBean(Class<T> beanClass, @Nullable String name,
                                @Nullable Class<? extends Annotation>[] qualifiers, @Nullable Supplier<T> supplier,
                                @Nullable BeanDefinitionCustomizer[] customizers) {

    
    AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(beanClass);
    if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
        return;
    }

    abd.setInstanceSupplier(supplier);
    ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
    abd.setScope(scopeMetadata.getScopeName());
    String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));

    
    AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
    if (qualifiers != null) {
        for (Class<? extends Annotation> qualifier : qualifiers) {
            if (Primary.class == qualifier) {
                abd.setPrimary(true);
            }
            else if (Lazy.class == qualifier) {
                abd.setLazyInit(true);
            }
            else {
                abd.addQualifier(new AutowireCandidateQualifier(qualifier));
            }
        }
    }
    if (customizers != null) {
        for (BeanDefinitionCustomizer customizer : customizers) {
            customizer.customize(abd);
        }
    }

    BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
    definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
    
    
    BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
}

```

> 以上，register() 流程就结束了

下面进入容器刷新的十二大步

```null

@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        
        prepareRefresh();

        
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        
        prepareBeanFactory(beanFactory);

        try {
            
            postProcessBeanFactory(beanFactory);

            
            invokeBeanFactoryPostProcessors(beanFactory);

            
            registerBeanPostProcessors(beanFactory);

            
            initMessageSource();

            
            initApplicationEventMulticaster();

            
            onRefresh();

            
            registerListeners();

            
            finishBeanFactoryInitialization(beanFactory);

            
            finishRefresh();
        }
        ......

```

最终，spring 容器刷新的详细流程图总结如下

![](https://img-blog.csdnimg.cn/fdd6f8b236204d7e888ecc13a460c4ce.jpg?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5ZGK5Yir5pe25YWJ,size_20,color_FFFFFF,t_70,g_se,x_16)

* * *

循环依赖
----

循环依赖指的是，A 组件中依赖了 B 组件，同时 B 组件又依赖了 A 组件，如下

```null
@Component
public class B {
    private A a;

    public B() {
        System.out.println("B 构造方法 ...");
    }

    @Autowired
    public void setA(A a) {
        this.a = a;
    }
}

```

```null
@Component
public class A {
    private B b;

    public A(){
        System.out.println("A 构造方法 ...");
    }

    @Autowired
    public void setB(B b) {
        this.b = b;
    }
}

```

又或者是组件自己依赖自己

```null
@Component
public class Self {

    private Self self;

    public Self() {
        System.out.println("Self 构造方法 ...");
    }

    @Autowired
    public void setSelf(Self self) {
        this.self = self;
    }
}

```

下面，通过 A 依赖 B 的方式来剖析 spring 应对循环依赖的过程，通过 debug 发现 A 组件先进入实例化的过程，直接通过 getBean() 获取 A 实例，通过 getBean() 又继续来到 doGetBean()，首先检查缓存中是否有已经手动实例化的单实例

```null
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

		final String beanName = transformedBeanName(name);
		Object bean;

		
		Object sharedInstance = getSingleton(beanName);
    .... 

```

深入 getSingleton() 方法

```null
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    
    Object singletonObject = this.singletonObjects.get(beanName);
    
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return singletonObject;
}

```

以上如果单例池中没有且不是正在创建中，则会正常执行 getSingleton() 逻辑，在这其中，又会第二次检查单例池中是否有 A 组件

```null
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(beanName, "Bean name must not be null");
    synchronized (this.singletonObjects) {
        Object singletonObject = this.singletonObjects.get(beanName);
        .....

```

此时单例池中依旧没有 A 组件的实例，继续执行 A 组件创建逻辑，而在 A 创建之前，会执行`beforeSingletonCreation(beanName);`如下

```null
protected void beforeSingletonCreation(String beanName) {
    
    if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
        throw new BeanCurrentlyInCreationException(beanName);
    }
}

```

之后又继续执行组件实例化的逻辑，来到 createBean() 中，调用一系列的后置处理器，然后来到真正创建实例的地方

```null
Object beanInstance = doCreateBean(beanName, mbdToUse, args);

```

至此，A 组件的实例创建完成，组件的属性赋值还没有完成；接下来，在 doCreateBean() 方法中的属性赋值操作`populateBean()`执行之前，会先执行如下逻辑

```null


boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                                  isSingletonCurrentlyInCreation(beanName));
if (earlySingletonExposure) {
    if (logger.isTraceEnabled()) {
        logger.trace("Eagerly caching bean '" + beanName +
                     "' to allow for resolving potential circular references");
    }
    
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
}

```

深入`addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));`逻辑

```null
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized (this.singletonObjects) {
        
        if (!this.singletonObjects.containsKey(beanName)) {
            
            this.singletonFactories.put(beanName, singletonFactory);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }
}

```

> lambda 表达式的具体逻辑如下
> 
> ```null
> protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
>  Object exposedObject = bean;
>  
>  if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
>      for (BeanPostProcessor bp : getBeanPostProcessors()) {
>          
>          if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
>              SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
>              exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
>          }
>      }
>  }
>  
>  return exposedObject;
> }
> 
> ```
> 
> 以上 lambda 逻辑后续都会在通过 singletonFactories 获取 bean 实例时被调用

继续执行，来到 A 组件的赋值逻辑，这里面，针对循环依赖的情况，A 需要将 B 组件的实例注入到自己的属性中；深入 populateBean() 部分关键逻辑

```null
.....
PropertyDescriptor[] filteredPds = null;
if (hasInstAwareBpps) {
    if (pvs == null) {
        pvs = mbd.getPropertyValues();
    }
	
    for (BeanPostProcessor bp : getBeanPostProcessors()) {
        if (bp instanceof InstantiationAwareBeanPostProcessor) {
            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
            PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
            if (pvsToUse == null) {
                if (filteredPds == null) {
                    filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
                }
                pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
                if (pvsToUse == null) {
                    return;
                }
            }
            pvs = pvsToUse;
        }
    }
}
....

```

AutowiredAnnotationBeanPostProcessor 后置处理器的 postProcessProperties() 中，执行以下逻辑

```null
public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
    	
		InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);
		try {
            
			metadata.inject(bean, beanName, pvs);
		}
		catch (BeanCreationException ex) {
			throw ex;
		}
		catch (Throwable ex) {
			throw new BeanCreationException(beanName, "Injection of autowired dependencies failed", ex);
		}
		return pvs;
	}

```

对于 bean 属性的注入，通过 injec() 最终会调用到如下逻辑

```null
public Object resolveCandidate(String beanName, Class<?> requiredType, BeanFactory beanFactory)
    throws BeansException {
    return beanFactory.getBean(beanName);
}

```

依旧会通过 getBean() 来获取属性注入需要的实例，此时 A 组件依赖的 B 组件也进入了实例的创建流程且创建流程和 A 组件一致，因为 B 目前来说也是第一次创建，直到 B 组件的 populateBean() 执行时，情况发生了一些微妙的变化；因为 B 组件是依赖了 A 组件的，这一点在 AutowiredAnnotationBeanPostProcessor 的 postProcessProperties() 会被 findAutowiringMetadata() 解析出来，接下来，B 组件的属性元数据 `InjectionMetadata metadata` 开始执行 `inject(bean, beanName, pvs)` 注入属性；这其中就又会执行 getBean() 来获取 A 组件，前面的流程依然一样 getBean() > doGetBean()，注意，在 doGetBean() 中依然会先执行`Object sharedInstance = getSingleton(beanName);`检查工厂中是否有属性对应的 bean 实例，参照源码

```null
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    
    Object singletonObject = this.singletonObjects.get(beanName);
    
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
       
        synchronized (this.singletonObjects) {
            
            singletonObject = this.earlySingletonObjects.get(beanName);
            
            if (singletonObject == null && allowEarlyReference) {
                 
                
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    
                    singletonObject = singletonFactory.getObject();
                    
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    
    return singletonObject;
}

```

到此，B 得了 A 组件，B 的属性赋值结束，后续就执行 B 组件的初始化逻辑，注意，此时 A 组件仍然停留在属性赋值阶段，它需要等 B 组件创建完成，B 的初始化逻辑完成后，B 组件的创建也就完成了，B 创建完成之后，继续执行初始化逻辑，然后又会进入如下逻辑

```null
.....

if (earlySingletonExposure) {
    
    Object earlySingletonReference = getSingleton(beanName, false);
    if (earlySingletonReference != null) {
        if (exposedObject == bean) {
            exposedObject = earlySingletonReference;
        }
        else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
            String[] dependentBeans = getDependentBeans(beanName);
            Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
            for (String dependentBean : dependentBeans) {
                if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
                    actualDependentBeans.add(dependentBean);
                }
            }
            if (!actualDependentBeans.isEmpty()) {
                throw new BeanCurrentlyInCreationException(beanName,
                                                           "Bean with name '" + beanName + "' has been injected into other beans [" +
                                                           StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
                                                           "] in its raw version as part of a circular reference, but has eventually been " +
                                                           "wrapped. This means that said other beans do not use the final version of the " +
                                                           "bean. This is often the result of over-eager type matching - consider using " +
                                                           "'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
            }
        }
    }
}
.....

```

之后，在外部的 doCreateBean() 的调用者，getSingleton() 方法的 finally 块中，还会执行一段关键逻辑，如下

```null
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(beanName, "Bean name must not be null");
    synchronized (this.singletonObjects) {
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null) {
            if (this.singletonsCurrentlyInDestruction) {
                throw new BeanCreationNotAllowedException(beanName,
                                                          "Singleton bean creation not allowed while singletons of this factory are in destruction " +
                                                          "(Do not request a bean from a BeanFactory in a destroy method implementation!)");
            }
            if (logger.isDebugEnabled()) {
                logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
            }
            beforeSingletonCreation(beanName);
            boolean newSingleton = false;
            boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
            if (recordSuppressedExceptions) {
                this.suppressedExceptions = new LinkedHashSet<>();
            }
            try {
                singletonObject = singletonFactory.getObject();
                newSingleton = true;
            }
            catch (IllegalStateException ex) {
                
                
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject == null) {
                    throw ex;
                }
            }
            catch (BeanCreationException ex) {
                if (recordSuppressedExceptions) {
                    for (Exception suppressedException : this.suppressedExceptions) {
                        ex.addRelatedCause(suppressedException);
                    }
                }
                throw ex;
            }
            finally {
                if (recordSuppressedExceptions) {
                    this.suppressedExceptions = null;
                }
                
                afterSingletonCreation(beanName);
            }
            if (newSingleton) {
                
                addSingleton(beanName, singletonObject);
            }
        }
        return singletonObject;
    }
}

```

深入 afterSingletonCreation(beanName);

```null
protected void afterSingletonCreation(String beanName) {
    
    if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.remove(beanName)) {
        throw new IllegalStateException("Singleton '" + beanName + "' isn't currently in creation");
    }
}

```

深入 addSingleton(beanName, singletonObject)

```null
protected void addSingleton(String beanName, Object singletonObject) {
    synchronized (this.singletonObjects) {
         
        this.singletonObjects.put(beanName, singletonObject);
        this.singletonFactories.remove(beanName);
        this.earlySingletonObjects.remove(beanName);
        
        this.registeredSingletons.add(beanName);
    }
}

```

至此，B 组件创建完成，提升到了一级缓存中，而 B 组件循环依赖的 A 组件目前还在二级缓存中；我们来梳理一下当前 A，B 组件在三级缓存中的变动情况：

> 最初，A 组件刚刚创建完实例时，作为一个早期单实例，通过 addSingletonFactory() 被放入了三级缓存 singletonFactories 中
> 
> 接着 A 组件在注入属性时，触发了 B 组件的实例化创建和属性注入，B 组件的早期单实例也被放入了三级缓存中
> 
> 之后 B 组件的属性注入通过三级缓存得到了 A 组件没有注入属性的早期单实例，然后将 A 组件由三级缓存提升到了二级缓存
> 
> 此时 B 组件实例化，属性注入，初始化的逻辑全部完成，作为一个合格的单实例，B 组件被直接放入一级缓存的单例池中（singletonObjects）同时从三级缓存和二级缓存中移除，但实际上当前整个流程 B 组件并没有进入过二级缓存，只是执行了一个移除操作

目前，A 组件还在属性注入阶段，当 B 组件完全创建成功后，A 的属性注入会继续执行，此时 B 已经在一级缓存的单例池中存在了，A 在属性注入时调用的 getBean() 就直接可以从容器中得到 B 单实例，完成属性的注入，接着又是 A 的初始化逻辑执行，初始化之后又来到了早期单实例暴露逻辑，此时的 A 还在二级缓存中，那么双检锁的 getSingleton() 就能够在二级缓存中获取到 A 组件。

最终 A 组件也创建完成，依然会执行`afterSingletonCreation()`和`addSingleton()`两个逻辑，将 A 从正在创建的缓存集合中移除，从二三级缓存中移除，将 A 组件放入一级缓存中。

需要注意的是，以上的整个所有流程，实际上我们实在获取 A 组件，目前 A 组件也获取完成了；B 组件虽然在 A 组件的获取过程中被完整创建了，但实际上顶层的真正获取 B 组件的逻辑并没有调用，当`preInstantiateSingletons()` 中遍历 bean 定义信息准备获取 B 组件时，才算是真正获取 B 组件，而此时 B 组件已经在一级缓存中了，直接 getBean() 就能从单例池中得到。

但实际上，就算没有三级缓存，只有二级，甚至是一级都能完成整个循环依赖的解决，因为三级缓存无非就是说将半成品的 bean 实例和完整的 bean 实例进行一个区分，我就是直接使用一级缓存，通过规定一些 key 的特定标识来区分半成品和完全品的 bean 实例，然后执行具体的逻辑

循环依赖流程图总结如下

![](https://img-blog.csdnimg.cn/56d7b1e224f649288903f28578004bb5.jpg?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5ZGK5Yir5pe25YWJ,size_20,color_FFFFFF,t_70,g_se,x_16)

* * *

spring aop
----------

AOP（Aspect OrientedProgramming）：面向切面编程，是 Spring 框架中的一个重要内容。利用 AOP 可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

为什么使用 aop

> 面向对象编程（OOP）的好处是显而易见的，缺点也同样明显。当需要为多个不具有继承关系的对象添加一个公共的方法的时候，例如日志记录、性能监控等，如果采用面向对象编程的方法，需要在每个对象里面都添加相同的方法，这样就产生了较大的重复工作量和大量的重复代码，不利于维护。面向切面编程（AOP）是面向对象编程的补充，简单来说就是统一处理某一“切面”的问题的编程思想。如果使用AOP的方式进行日志的记录和处理，所有的日志代码都集中于一处，不需要再每个方法里面都去添加，极大减少了重复代码。

aop 的使用简单高效，但是对其内部执行原理的掌握仍然具备一定的必要性，下面针对 spring aop 的源码进行分析

首先准备前置环境，在 spring boot 环境下，导入 aop 的 starter

```null

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>

```

配置类中开启 spring aop

```null
@EnableAspectJAutoProxy 
@Configuration
@ComponentScan(basePackages = "com.dhj.demo")
public class Config {
}

```

准备业务类

```null
@Component  
public class HelloService {

    public HelloService() {
        System.out.println("....");
    }

    public String sayHello(String name) {
        String result = "你好：" + name;
        System.out.println(result);
        int length = name.length();
        return result + "---" + length;
    }
}

```

准备 aop 切面

```null
@Component  
@Aspect 
public class LogAspect {

    public LogAspect() {
        System.out.println("LogAspect 构造方法...");
    }

    
    @Before("execution(* com.dhj.demo.aop.HelloService.sayHello(..))")
    public void logStart(JoinPoint joinPoint) {
        String name = joinPoint.getSignature().getName();
        System.out.println("前置通知：logStart()==>" + name + "....【args: " + Arrays.asList(joinPoint.getArgs()) + "】");
    }

    
    @AfterReturning(value = "execution(* com.dhj.demo.aop.HelloService.sayHello(..))", returning = "result")
    public void logReturn(JoinPoint joinPoint, Object result) {
        String name = joinPoint.getSignature().getName();
        System.out.println("返回通知：logReturn()==>" + name + "....【args: " + Arrays.asList(joinPoint.getArgs()) + "】【result: " + result + "】");
    }

    
    @After("execution(* com.dhj.demo.aop.HelloService.sayHello(..))")
    public void logEnd(JoinPoint joinPoint) {
        String name = joinPoint.getSignature().getName();
        System.out.println("后置通知：logEnd()==>" + name + "....【args: " + Arrays.asList(joinPoint.getArgs()) + "】");
    }


    
    @AfterThrowing(value = "execution(* com.dhj.demo.aop.HelloService.sayHello(..))", throwing = "e")
    public void logError(JoinPoint joinPoint, Exception e) {
        String name = joinPoint.getSignature().getName();
        System.out.println("异常通知：logError()==>" + name + "....【args: " + Arrays.asList(joinPoint.getArgs()) + "】【exception: " + e + "】");
    }
}

```

要分析 aop 的实现原理，需要明确两点

> 向 spring 容器中放入了那些组件
> 
> 这些组件干了什么事情

因为要使用 aop 就一定需要`@EnableAspectJAutoProxy`来开启 aop 功能，那么就可以直接从`@EnableAspectJAutoProxy`注解入手，其源码如下

```null
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {

	
	boolean proxyTargetClass() default false;

	
	boolean exposeProxy() default false;

}

```

在`AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);`处打上断点，开启 debug 从全局角度分析`@EnableAspectJAutoProxy`注解的执行逻辑

首先，创建 spring 容器，在十二大步的`invokeBeanFactoryPostProcessors(beanFactory);`中通过`org.springframework.context.annotation.ConfigurationClassPostProcessor`加载解析配置类，内部调用`processConfigBeanDefinitions(registry);`在这其中会解析配置类；通过`@EnableAspectJAutoProxy`注解的源码可以知道，它通过`@Import`往容器中导入了一个组件：`@Import(AspectJAutoProxyRegistrar.class)`，那么在解析配置类时，就会解析到`@EnableAspectJAutoProxy`中的`@Import`注解，解析`@Import`的关键源码如下

```null

loadBeanDefinitionsFromRegistrars(configClass.getImportBeanDefinitionRegistrars());

```

最终解析到`@EnableAspectJAutoProxy`中`@Import`注解的`Class`参数为`org.springframework.context.annotation.AspectJAutoProxyRegistrar`，该类实现了`ImportBeanDefinitionRegistrar`且重写了其中的`registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry)`方法，该方法会向`BeanDefinitionRegistry`中注册 bean 定义信息，AspectJAutoProxyRegistrar 源码如下

```null
import org.springframework.aop.config.AopConfigUtils;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.core.annotation.AnnotationAttributes;
import org.springframework.core.type.AnnotationMetadata;

class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {

	
	@Override
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

		AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);

		AnnotationAttributes enableAspectJAutoProxy =
				AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
		if (enableAspectJAutoProxy != null) {
			if (enableAspectJAutoProxy.getBoolean("proxyTargetClass")) {
				AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
			}
			if (enableAspectJAutoProxy.getBoolean("exposeProxy")) {
				AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
			}
		}
	}

}

```

通过`AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);`注册 bean 定义信息，追踪该方法，有如下源码

```null
public static BeanDefinition registerAspectJAnnotationAutoProxyCreatorIfNecessary(
    BeanDefinitionRegistry registry, @Nullable Object source) {

    return registerOrEscalateApcAsRequired(AnnotationAwareAspectJAutoProxyCreator.class, registry, source);
}

```

其中，`AnnotationAwareAspectJAutoProxyCreator.class`便是最终要注册 bean 定义信息的 class 类型，检查该类的类图，如下

![](https://img-blog.csdnimg.cn/f9291ccd3246488c9adf2c5ab7f134e6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5ZGK5Yir5pe25YWJ,size_20,color_FFFFFF,t_70,g_se,x_16)

发现`AnnotationAwareAspectJAutoProxyCreator`实际上是一个`BeanPostProcesser`后置处理器，那么整个 aop 功能的实现就是依靠该后置处理器在 bean 创建时完成的。

接下来，就详细分析`AnnotationAwareAspectJAutoProxyCreator`在 aop 过程中的执行逻辑，首先依旧是创建 spring 容器实例，进入刷新容器十二大步，因为`AnnotationAwareAspectJAutoProxyCreator`是一个后置处理器，因此其具体逻辑的执行在容器刷新十二大步中的`registerBeanPostProcessors(beanFactory);`中；

之后便会获取后置处理器的实例，进入后置处理的`getBean()`流程：`beanFactory.getBean(ppName, BeanPostProcessor.class);`

因为`AnnotationAwareAspectJAutoProxyCreator`实现了`BeanFactoryAware`接口，则在其初始化时会调用`invokeAwareMethods()`，根据其实现的`Aware`接口类型来调用对应的接口方法，这里调用的是`((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);`，相关源码如下

```null
@Override
public void setBeanFactory(BeanFactory beanFactory) {
    
    super.setBeanFactory(beanFactory);
    if (!(beanFactory instanceof ConfigurableListableBeanFactory)) {
        throw new IllegalArgumentException(
            "AdvisorAutoProxyCreator requires a ConfigurableListableBeanFactory: " + beanFactory);
    }
    
    initBeanFactory((ConfigurableListableBeanFactory) beanFactory);
}

```

子类中的`initBeanFactory()`实现如下

```null
@Override
protected void initBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    
    super.initBeanFactory(beanFactory);
    if (this.aspectJAdvisorFactory == null) {
        
        this.aspectJAdvisorFactory = new ReflectiveAspectJAdvisorFactory(beanFactory);
    }
    
    this.aspectJAdvisorsBuilder =
        
        new BeanFactoryAspectJAdvisorsBuilderAdapter(beanFactory, this.aspectJAdvisorFactory);
}

```

以上，`AnnotationAwareAspectJAutoProxyCreator`初始化结束，整个创建就完成了，主要是创建了`aspectJAdvisorFactory`和`aspectJAdvisorsBuilder`，它们负责保存和产生功能增强器

整个就是`AnnotationAwareAspectJAutoProxyCreator`中第一个被调用方法：`initBeanFactory()`的执行过程，都是在`AnnotationAwareAspectJAutoProxyCreator`实例化时执行的。

继续断点调试，来到了第二个执行的方法：`isInfrastructureClass(Class<?> beanClass)`，源码如下

```null
protected boolean isInfrastructureClass(Class<?> beanClass) {
    
    return (super.isInfrastructureClass(beanClass) ||
            
            (this.aspectJAdvisorFactory != null && this.aspectJAdvisorFactory.isAspect(beanClass)));
}

```

注意，在这之前，`AnnotationAwareAspectJAutoProxyCreator`这个后置处理器以及创建完成了，那么在其他组件创建时，该后置处理器就会起作用，以上就是该后置处理器产生作用而执行的一段逻辑

执行的时机则是在十二大步的`finishBeanFactoryInitialization()`处，正是组件实例化的地方，具体的执行时机实在 createBean() 中的如下源码处

```null
.....
try {
    
    Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
    if (bean != null) {
        return bean;
    }
}
catch (Throwable ex) {
    throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
                                    "BeanPostProcessor before instantiation of bean failed", ex);
}
.....

```

之后则是调用后置处理逻辑，执行上面的`isInfrastructureClass(Class<?> beanClass)`方法，该方法主要就是判断其他组件是否需要被 aop 增强。

继续下一个断点，来到`AnnotationAwareAspectJAutoProxyCreator`中的`findCandidateAdvisors()`方法，

```null
@Override
protected List<Advisor> findCandidateAdvisors() {
    
    List<Advisor> advisors = super.findCandidateAdvisors();
    
    if (this.aspectJAdvisorsBuilder != null) {
        advisors.addAll(this.aspectJAdvisorsBuilder.buildAspectJAdvisors());
    }
    return advisors;
}

```

`findCandidateAdvisors()`也是后置增强逻辑，倒推其执行逻辑，来到后置增强器调用处

```null
public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) {
    Object cacheKey = getCacheKey(beanClass, beanName);

    if (!StringUtils.hasLength(beanName) || !this.targetSourcedBeans.contains(beanName)) {
        if (this.advisedBeans.containsKey(cacheKey)) {
            return null;
        }
        
        if (isInfrastructureClass(beanClass) || shouldSkip(beanClass, beanName)) {
            
            this.advisedBeans.put(cacheKey, Boolean.FALSE);
            return null;
        }
    }

```

`shouldSkip()`源码如下

```null
@Override
protected boolean shouldSkip(Class<?> beanClass, String beanName) {
    
    List<Advisor> candidateAdvisors = findCandidateAdvisors();
    for (Advisor advisor : candidateAdvisors) {
        if (advisor instanceof AspectJPointcutAdvisor &&
            ((AspectJPointcutAdvisor) advisor).getAspectName().equals(beanName)) {
            return true;
        }
    }
    return super.shouldSkip(beanClass, beanName);
}

```

以上方法，最初调用的是`postProcessBeforeInstantiation()`后置增强逻辑，这其中调用到了`isInfrastructureClass(beanClass)`和`shouldSkip(beanClass, beanName)`，而在`shouldSkip(beanClass, beanName)`中又调用了`findCandidateAdvisors()`方法，`findCandidateAdvisors()`的特点在于，无论其他组件是否有增强器，都会执行构建增强器的逻辑，这一切都是在`AnnotationAwareAspectJAutoProxyCreator`等后置处理器创建后，后面的其他组件进入创建阶段时，第一个被创建的其他组件创建逻辑执行时开始执行的；

也就是说，每当其他组件创建一次，哪怕该组件没有标注`@Aspect`注解，也不需要增强，都会执行`findCandidateAdvisors()`中的：`查找增强器`和`构建增强器逻辑`，这一切都是在其他组件实例化之前，通过`AnnotationAwareAspectJAutoProxyCreator`后置处理器的调用来实现，而后置处理逻辑的实现，在`AnnotationAwareAspectJAutoProxyCreator`的父类`org.springframework.aop.framework.autoproxy.AbstractAdvisorAutoProxyCreator`中实现，只不过在父类的后置处理方法中，调用了子类重写的对应方法，如`findCandidateAdvisors()`或`shouldSkip()`

既然无论怎样，`findCandidateAdvisors()`中的构建增强器逻辑：`advisors.addAll(this.aspectJAdvisorsBuilder.buildAspectJAdvisors());`都会执行，那就来看看其内部实现逻辑，借此，能够解析一出 spring 是如何感知到切面组件的

```null
public List<Advisor> buildAspectJAdvisors() {
    
    List<String> aspectNames = this.aspectBeanNames;
    
    if (aspectNames == null) {
        synchronized (this) {
            aspectNames = this.aspectBeanNames;
            if (aspectNames == null) {
                List<Advisor> advisors = new ArrayList<>();
                aspectNames = new ArrayList<>();
                
                String[] beanNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
                    this.beanFactory, Object.class, true, false);
                
                for (String beanName : beanNames) {
                    
                    if (!isEligibleBean(beanName)) {
                        continue;
                    }
                    
                    
                    Class<?> beanType = this.beanFactory.getType(beanName);
                    if (beanType == null) {
                        continue;
                    }
                    
                    if (this.advisorFactory.isAspect(beanType)) {
                        
                        aspectNames.add(beanName);
                        AspectMetadata amd = new AspectMetadata(beanType, beanName);
                        if (amd.getAjType().getPerClause().getKind() == PerClauseKind.SINGLETON) {
                            MetadataAwareAspectInstanceFactory factory =
                                new BeanFactoryAspectInstanceFactory(this.beanFactory, beanName);
                            
                            List<Advisor> classAdvisors = this.advisorFactory.getAdvisors(factory);
                            if (this.beanFactory.isSingleton(beanName)) {
                                this.advisorsCache.put(beanName, classAdvisors);
                            }
                            else {
                                this.aspectFactoryCache.put(beanName, factory);
                            }
                            advisors.addAll(classAdvisors);
                        }
                        else {
                            
                            if (this.beanFactory.isSingleton(beanName)) {
                                throw new IllegalArgumentException("Bean with name '" + beanName +
                                                                   "' is a singleton, but aspect instantiation model is not singleton");
                            }
                            MetadataAwareAspectInstanceFactory factory =
                                new PrototypeAspectInstanceFactory(this.beanFactory, beanName);
                            this.aspectFactoryCache.put(beanName, factory);
                            advisors.addAll(this.advisorFactory.getAdvisors(factory));
                        }
                    }
                }
                
                this.aspectBeanNames = aspectNames;
                return advisors;
            }
        }
    }

    
    if (aspectNames.isEmpty()) {
        return Collections.emptyList();
    }
    List<Advisor> advisors = new ArrayList<>();
    for (String aspectName : aspectNames) {
        List<Advisor> cachedAdvisors = this.advisorsCache.get(aspectName);
        if (cachedAdvisors != null) {
            advisors.addAll(cachedAdvisors);
        }
        else {
            MetadataAwareAspectInstanceFactory factory = this.aspectFactoryCache.get(aspectName);
            advisors.addAll(this.advisorFactory.getAdvisors(factory));
        }
    }
    
    return advisors;
}

```

综合来看，spring aop 在其对应的后置处理逻辑第一次运行时，就完成了切面及其增强器（通知）的加载且放入了缓存，仅管后续的 bean 创建每次都会调用 aop 的后置处理器，但内部通过缓存加判断实现了幂等调用，不会重复获取和缓存 aop 切面信息

小总结

> spring aop 的初始化和切面加载就是围绕`AnnotationAwareAspectJAutoProxyCreator`后置处理来实现的，首先在`AnnotationAwareAspectJAutoProxyCreator`初始化时，其`aspectJAdvisorFactory`和`aspectJAdvisorsBuilder`属性被初始化
> 
> 之后，当其他 bean 创建时，`AnnotationAwareAspectJAutoProxyCreator`中后置处理相关逻辑被调用，但后置处理逻辑的调用处是其父类及其父类的父类，采取的是组合调用的形式，在后置处理被首次调用时，切面的相关信息就被加载到缓存中且后续每次后置处理逻辑被调用时，都不会重复加载切面数据，而且这个后置处理逻辑在其他 bean 创建之前和初始化时都会调用一次。