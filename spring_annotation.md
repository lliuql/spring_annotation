Spring 注解驱动开发



## 注解基础

### 注解的概念与创建

1. 理解annotation

    -  从 JDK 5.0 开始, Java 增加了对元数据(MetaData) 的支持, 也就是 Annotation(注解) 
    -  Annotation 其实就是代码里的特殊标记, 这些标记可以在编译, 类加载, 运行时被读取, 并执行相应的处理。通过使用 Annotation,程序员可以在不改变原有逻辑的情况下, 在源文件中嵌入一些补充信息。
    - Annotation 可以像修饰符一样被使用, 可用于修饰包,类, 构造器, 方 法, 成员变量, 参数, 局部变量的声明, 这些信息被保存在Annotation 的 “name=value” 对中
    - 在JavaSE中，注解的使用目的比较简单，例如标记过时的功能，忽略警告等。在JavaEE/Android中注解占据了更重要的角色，例如用来配置应用程序的任何切面，代替JavaEE旧版中所遗留的繁冗代码和XML配置等。
    -  未来的开发模式都是基于注解的，JPA是基于注解的，Spring2.5以 上都是基于注解的，Hibernate3.x以后也是基于注解的，现在的 Struts2有一部分也是基于注解的了，注解是一种趋势，一定程度上 可以说：框架 = 注解 + 反射 + 设计模式。

 2. Annocation的使用示例

     - 示例一：生成文档相关的注解
     - 示例二：在编译时进行格式检查(JDK内置的三个基本注解)
         @Override: 限定重写父类方法, 该注解只能用于方法
         @Deprecated: 用于表示所修饰的元素(类, 方法等)已过时。通常是因为所修饰的结构危险或存在更好的选择
         @SuppressWarnings: 抑制编译器警告
      - 示例三：跟踪代码依赖性，实现替代配置文件功能

  3. 如何自定义注解：参照@SuppressWarnings定义

     -  定义新的 Annotation 类型使用 @interface 关键字 

     -  自定义注解自动继承了java.lang.annotation.Annotation接口 

     -  Annotation 的成员变量在 Annotation 定义中以无参数方法的形式来声明。其 方法名和返回值定义了该成员的名字和类型。我们称为配置参数。类型只能 是八种基本数据类型、String类型、Class类型、enum类型、Annotation类型、 以上所有类型的数组。 

     -  可以在定义 Annotation 的成员变量时为其指定初始值, 指定成员变量的初始 值可使用 default 关键字 

       例3-1：
     
       ```java
       public @interface MyAnnotation{
           String value() default "hello";
       }
       ```
     
       
     
- 如果只有一个参数成员，建议使用参数名为value 
  
     -  如果定义的注解含有配置参数，那么使用时必须指定参数值，除非它有默认 值。格式是“参数名 = 参数值”，如果只有一个参数成员，且名称为value， 可以省略“value=” 
     
     ​		
     
     -  没有成员定义的 Annotation 称为标记; 包含成员变量的 Annotation 称为元数 据 Annotation 
     
       例：
     
       ```java
       @Target(ElementType.METHOD)
       @Retention(RetentionPolicy.SOURCE)
       public @interface Override {
       }
       ```
     
       注意：
     
       - 如果注解有成员，在使用注解时，需要指明成员的值。如3-1 使用时`@MyAnnotaion(value= 'xxx') `有默认值可以不需要。value可以省略
     
       - 自定义注解必须配上注解的信息处理流程(使用反射)才有意义。
       - 自定义注解通过都会指明两个元注解：Retention、Target
     
       
     
4. jdk 提供的4种元注解

     - 元注解概念： 对现有的注解进行解释说明的注解

     - Retention：指定所修饰的 Annotation 的生命周期：
     
     @Rentention 包含一个 RetentionPolicy 类型的成员变量, 使用 @Rentention 时必须为该 value 成员变量指定值: 
     
     - RetentionPolicy.SOURCE:在源文件中有效（即源文件保留），编译器直接丢弃这种策略的 注释 
          - RetentionPolicy.CLASS:在class文件中有效（即class保留） ， 当运行 Java 程序时, JVM 不会保留注解。 这是默认值 
          - RetentionPolicy.RUNTIME:在运行时有效（即运行时保留），当运行 Java 程序时, JVM 会 保留注释。程序可以通过反射获取该注释。
     
          例如：
     
          ``` java
          @Target(ElementType.METHOD)
          @Retention(RetentionPolicy.SOURCE)
          public @interface Override {
          }
          ```
     
          
     
     - Target:用于指定被修饰的 Annotation 能用于修饰哪些程序元素
     
          | 取值(ElementType) | 作用                                   |
          | ----------------- | -------------------------------------- |
          | CONSTRUCTOR       | 用于描述构造器                         |
          | PACKAGE           | 用于描述包                             |
          | FIELD             | 用于描述域                             |
          | PARAMETER         | 用于描述参数                           |
          | LOCAL_VARIABLE    | 描述局部变量                           |
          | TYPE              | 描述类、接口（包括注解类型）或enum生命 |
          | METHOD            | 描述方法                               |
     
          
     
      *******出现的频率较低*******
     
     - Documented:表示所修饰的注解在被javadoc解析时，保留下来。
     
     - Inherited:被它修饰的 Annotation 将具有继承性。如果某个类使用了被 @Inherited 修饰的 Annotation, 则其子类将自动具有该注解。 
       - 比如：如果把标有@Inherited注解的自定义的注解标注在类级别上，子类则可以 继承父类类级别的注解 
       - 在实际应用中，使用较少
     
5. jdk 8 中注解的新特性： 可重复注解、类型注解

     - 可重复注解：① 在MyAnnotation上声明@Repeatable，成员值为MyAnnotations.class
                           ② MyAnnotation的Target和Retention等元注解与MyAnnotations相同。

                       例子：
                               
                       ![1567345145708](./images/1567345145708.png)
                       
     - 类型注解：
     
        - ElementType.TYPE_PARAMETER 表示该注解能写在类型变量的声明语句中（如：泛型声明）。
        
        - ElementType.TYPE_USE 表示该注解能写在使用类型的任何语句中。
        
          可以解决下面3个地方不能添加注解
        
        ![1567344862022](./images/1567344862022.png)





### 注解的使用

通过反射获取注解



## 组件添加

```
/**
 * 给容器中注册组件；
 * 1）、包扫描+组件标注注解（@Controller/@Service/@Repository/@Component）[自己写的类]
 * 2）、@Bean[导入的第三方包里面的组件]
 * 3）、@Import[快速给容器中导入一个组件]
 *        1）、@Import(要导入到容器中的组件)；容器中就会自动注册这个组件，id默认是全类名
 *        2）、ImportSelector:返回需要导入的组件的全类名数组；
 *        3）、ImportBeanDefinitionRegistrar:手动注册bean到容器中
 * 4）、使用Spring提供的 FactoryBean（工厂Bean）;
 *        1）、默认获取到的是工厂bean调用getObject创建的对象
 *        2）、要获取工厂Bean本身，我们需要给id前面加一个&
 *           &colorFactoryBean
 */
```



### @ComponentScan

检出需要的组件（扫描出需要的组件）

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Repeatable(ComponentScans.class)
public @interface ComponentScan {
    @AliasFor("basePackages")
    String[] value() default {};

    @AliasFor("value")
    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};

    Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

    Class<? extends ScopeMetadataResolver> scopeResolver() default AnnotationScopeMetadataResolver.class;

    ScopedProxyMode scopedProxy() default ScopedProxyMode.DEFAULT;

    String resourcePattern() default "**/*.class";

    boolean useDefaultFilters() default true;

    ComponentScan.Filter[] includeFilters() default {};

    ComponentScan.Filter[] excludeFilters() default {};

    boolean lazyInit() default false;

    @Retention(RetentionPolicy.RUNTIME)
    @Target({})
    public @interface Filter {
        FilterType type() default FilterType.ANNOTATION;

        @AliasFor("classes")
        Class<?>[] value() default {};

        @AliasFor("value")
        Class<?>[] classes() default {};

        String[] pattern() default {};
    }
}
```



使用：

```
@ComponentScans(
      value = {
            @ComponentScan(value="com.atguigu",includeFilters = {
/*                @Filter(type=FilterType.ANNOTATION,classes={Controller.class}),
                  @Filter(type=FilterType.ASSIGNABLE_TYPE,classes={BookService.class}),*/
                  @Filter(type=FilterType.CUSTOM,classes={MyTypeFilter.class})
            },useDefaultFilters = false)   
      }
      )
```

```
//@ComponentScan  value:指定要扫描的包
//excludeFilters = Filter[] ：指定扫描的时候按照什么规则排除那些组件
//includeFilters = Filter[] ：指定扫描的时候只需要包含哪些组件
//FilterType.ANNOTATION：按照注解
//FilterType.ASSIGNABLE_TYPE：按照给定的类型；
//FilterType.ASPECTJ：使用ASPECTJ表达式
//FilterType.REGEX：使用正则指定
//FilterType.CUSTOM：使用自定义规则
```

- FilterType.CUSTOM：使用自定义规则 需要自己定义Filter

```java
public class MyTypeFilter implements TypeFilter {

	/**
	 * metadataReader：读取到的当前正在扫描的类的信息
	 * metadataReaderFactory:可以获取到其他任何类信息的
	 */
	@Override
	public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
			throws IOException {
		// TODO Auto-generated method stub
		//获取当前类注解的信息
		AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
		//获取当前正在扫描的类的类信息
		ClassMetadata classMetadata = metadataReader.getClassMetadata();
		//获取当前类资源（类的路径）
		Resource resource = metadataReader.getResource();
		
		String className = classMetadata.getClassName();
		System.out.println("--->"+className);
		if(className.contains("er")){
			return true;
		}
		return false;
	}

}
```



### @Scope

@Scope 相当于组件的生命周期，填的值都可以在源码中找到 

```
* prototype：多实例的：ioc容器启动并不会去调用方法创建对象放在容器中。
*              每次获取的时候才会调用方法创建对象；
* singleton：单实例的（默认值）：ioc容器启动会调用方法创建对象放到ioc容器中。
*        以后每次获取就是直接从容器（map.get()）中拿，
* request：同一次请求创建一个实例
* session：同一个session创建一个实例
```

```
//  @Scope("prototype")
   @Lazy
   @Bean("person")
   public Person person(){
      System.out.println("给容器中添加Person....");
      return new Person("张三", 25);
   }
```



### @Lazy

```
懒加载：
*     单实例bean：默认在容器启动的时候创建对象；
*     懒加载：容器启动不创建对象。第一次使用(获取)Bean创建对象，并初始化；
```



### @Conditional

 按照一定的条件进行判断，满足条件给容器中注册bean



```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {
	Class<? extends Condition>[] value();
}
```



value填入Condition类数组

```
//判断是否linux系统
public class LinuxCondition implements Condition {

   /**
    * ConditionContext：判断条件能使用的上下文（环境）
    * AnnotatedTypeMetadata：注释信息
    */
   @Override
   public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
      // TODO是否linux系统
      //1、能获取到ioc使用的beanfactory
      ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
      //2、获取类加载器
      ClassLoader classLoader = context.getClassLoader();
      //3、获取当前环境信息
      Environment environment = context.getEnvironment();
      //4、获取到bean定义的注册类
      BeanDefinitionRegistry registry = context.getRegistry();
      
      String property = environment.getProperty("os.name");
      
      //可以判断容器中的bean注册情况，也可以给容器中注册bean
      boolean definition = registry.containsBeanDefinition("person");
      if(property.contains("linux")){
         return true;
      }
      
      return false;
   }

}
```





使用：

```
@Conditional(LinuxCondition.class)
@Bean("linus")
public Person person02(){
   return new Person("linus", 48);
}
```



使用2： 放到类上面

```
//类中组件统一设置。满足当前条件，这个类中配置的所有bean注册才能生效；
@Conditional({WindowsCondition.class})
@Configuration
public class MainConfig2 {}
```



### @Import

快速给容器导入组件

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {

   /**
    * {@link Configuration}, {@link ImportSelector}, {@link ImportBeanDefinitionRegistrar}
    * or regular component classes to import.
    */
   Class<?>[] value();

}
```

#### 普通使用：

id默认是组件的全类名

```
@Import({Color.class,Red.class)
//@Import导入组件，id默认是组件的全类名
public class MainConfig2 {}
```

#### 使用ImportSelector：

返回需要导入的组件的全类名数组

```
//自定义逻辑返回需要导入的组件
public class MyImportSelector implements ImportSelector {

   //返回值，就是到导入到容器中的组件全类名
   //AnnotationMetadata:当前标注@Import注解的类的所有注解信息
   @Override
   public String[] selectImports(AnnotationMetadata importingClassMetadata) {
      // TODO Auto-generated method stub
      //importingClassMetadata
      //方法不要返回null值
      return new String[]{"com.atguigu.bean.Blue","com.atguigu.bean.Yellow"};
   }

}
```



使用：

```java
@Import({Color.class,Red.class, MyImportSelector.class)
//@Import导入组件，id默认是组件的全类名
public class MainConfig2 {}
```





#### 使用ImportBeanDefinitionRegistrar

通过接口提供的BeanDefinitionRegistry对象来进行注册

```
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

   /**
    * AnnotationMetadata：当前类的注解信息
    * BeanDefinitionRegistry:BeanDefinition注册类；
    *        把所有需要添加到容器中的bean；调用
    *        BeanDefinitionRegistry.registerBeanDefinition手工注册进来
    */
   @Override
   public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
      
      boolean definition = registry.containsBeanDefinition("com.atguigu.bean.Red");
      boolean definition2 = registry.containsBeanDefinition("com.atguigu.bean.Blue");
      if(definition && definition2){
         //指定Bean定义信息；（Bean的类型，Bean。。。）
         RootBeanDefinition beanDefinition = new RootBeanDefinition(RainBow.class);
         //注册一个Bean，指定bean名
         registry.registerBeanDefinition("rainBow", beanDefinition);
      }
   }

}
```

### FactoryBean

 1）、默认获取到的是工厂bean调用getObject创建的对象

2）、要获取工厂Bean本身，我们需要给id前面加一个&colorFactoryBean

```java
public interface FactoryBean<T> {

	T getObject() throws Exception;

	Class<?> getObjectType();

	boolean isSingleton();

}

```



使用：第一步创建工厂bean

```java
//创建一个Spring定义的FactoryBean
public class ColorFactoryBean implements FactoryBean<Color> {

	//返回一个Color对象，这个对象会添加到容器中
	@Override
	public Color getObject() throws Exception {
		System.out.println("ColorFactoryBean...getObject...");
		return new Color();
	}

	@Override
	public Class<?> getObjectType() {
		return Color.class;
	}

	//是单例？
	//true：这个bean是单实例，在容器中保存一份
	//false：多实例，每次获取都会创建一个新的bean；
	@Override
	public boolean isSingleton() {
		return false;
	}

}
```



第二步：配置文件上使用

```
@Configuration
public class MainConfig2 {
	@Bean
	public ColorFactoryBean colorFactoryBean(){
		return new ColorFactoryBean();
	}
}
```



第三步：获取对象使用

```
@Test
public void test1() {
    //工厂Bean获取的是调用getObject创建的对象
    Object bean2 = applicationContext.getBean("colorFactoryBean");
    Object bean3 = applicationContext.getBean("colorFactoryBean");
    System.out.println("bean的类型："+bean2.getClass());
    System.out.println(bean2 == bean3);
	
	// 获取工厂本身
    Object bean4 = applicationContext.getBean("&colorFactoryBean");
    System.out.println(bean4.getClass());
}

```





## bean 生命周期注解

- bean的生命周期：  bean创建---初始化----销毁的过程

  - 创建时机：
    -  单实例：在容器启动的时候创建对象（可以配置懒加载修改）
    - 多实例：在每次获取的时候创建对象
  - 销毁时机：
     *        单实例：容器关闭的时候
     *        多实例：容器不会管理这个bean；容器不会调用销毁方法；

  

- 自定义初始化和销毁方法：

  1）、指定初始化和销毁方法；

   	通过@Bean指定init-method和destroy-method；

  ```java
@Configuration
  public class MainConfigOfLifeCycle {
	
  	//@Scope("prototype")
  	@Bean(initMethod="init",destroyMethod="detory")
  	public Car car(){
  		return new Car();
  	}
  
  }

  public class Car {
	
  	public Car(){
		System.out.println("car constructor...");
  	}
  	
  	public void init(){
  		System.out.println("car ... init...");
  	}
  	
  	public void detory(){
  		System.out.println("car ... detory...");
  	}
  
  }
  ```
  
  
  
  
  
  2）、通过让Bean实现InitializingBean（定义初始化逻辑），DisposableBean（定义销毁逻辑）;
  
  ```java
  public interface InitializingBean {
  	void afterPropertiesSet() throws Exception;
  }
  public interface DisposableBean {
  	void destroy() throws Exception;
  }
  ```
  
  

  ```java
  public class Cat implements InitializingBean,DisposableBean {
  	
  	public Cat(){
  		System.out.println("cat constructor...");
  	}
  
  	@Override
  	public void destroy() throws Exception {
  		System.out.println("cat...destroy...");
  	}
  
  	@Override
  	public void afterPropertiesSet() throws Exception {
  		System.out.println("cat...afterPropertiesSet...");
  	}
  }
  
  @Configuration
  public class MainConfigOfLifeCycle {
  	
  	@Bean
  	public Car car(){
  		return new Car();
  	}
  
  }
  ```
  
  
  
  
  
  
  
  3）、可以使用JSR250；
           @PostConstruct：在bean创建完成并且属性赋值完成；来执行初始化方法
           @PreDestroy：在容器销毁bean之前通知我们进行清理工作
  
  ```
  public class Dog {
       
     public Dog(){
        System.out.println("dog constructor...");
     }
     
     //对象创建并赋值之后调用
     @PostConstruct
     public void init(){
        System.out.println("Dog....@PostConstruct...");
     }
     
     //容器移除对象之前
     @PreDestroy
     public void detory(){
        System.out.println("Dog....@PreDestroy...");
     }
  
     @Override
     public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
     }
  }
  ```
  
  
  
   4）、BeanPostProcessor【interface】：bean的后置处理器；
           在bean初始化前后进行一些处理工作；
  
  ```
  public interface BeanPostProcessor {
  	// 在初始化之前工作
     Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
  	// :在初始化之后工作
     Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
  
  }
  ```
  
  ```java
  public class MyBeanPostProcessor implements BeanPostProcessor {
  
     @Override
     public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization..."+beanName+"=>"+bean);
        return bean;
     }
  
     @Override
     public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization..."+beanName+"=>"+bean);
        return bean;
     }
  
  }
  ```
  
  
