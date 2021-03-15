## 1.1 如何获取Spring容器对象?
> ①、实现BeanFactoryAware接口,其中接口关系Aware,通知的意思.然后重写setBeanFactory方法,就能从该方法中获取到spring容器对象.
```
public interface Aware {

}
```
```
public interface BeanFactoryAware extends Aware {

	/**
	 * Callback that supplies the owning factory to a bean instance.
	 * <p>Invoked after the population of normal bean properties
	 * but before an initialization callback such as
	 * {@link InitializingBean#afterPropertiesSet()} or a custom init-method.
	 * @param beanFactory owning BeanFactory (never {@code null}).
	 * The bean can immediately call methods on the factory.
	 * @throws BeansException in case of initialization errors
	 * @see BeanInitializationException
	 */
	void setBeanFactory(BeanFactory beanFactory) throws BeansException;

}
```
```
@Service
public class HomeBeanFactoryAwareService implements BeanFactoryAware {

    private BeanFactory beanFactory;

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
         this.beanFactory=beanFactory;
    }

    public void getBean(){
        Home bean = (Home) beanFactory.getBean("home");
        bean.test();
    }

}
```
> ②、实现ApplicationContextAware接口方式,实现ApplicationContextAware接口,然后重写setApplicationContext方法,也能从该方法中获取到spring容器对象.
```
@Service
public class HomeApplicationContextAwareService implements ApplicationContextAware {
    private ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
          this.applicationContext=applicationContext;
    }

    public void  getBean(){
        Home home = (Home)applicationContext.getBean("home");
        home.test();
    }
}
```
> ③、实现ApplicationListener接口,需要注意的是该接口接收的泛型是ContextRefreshedEvent类,然后重写onApplicationEvent方法,也能从该方法中获取到spring容器对象.
```
@Service
public class HomeApplicationListenerService implements ApplicationListener<ContextRefreshedEvent> {

    private ApplicationContext applicationContext;

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
         this.applicationContext=event.getApplicationContext();
    }

    public void  getBean(){
        Home home = (Home)applicationContext.getBean("home");
        home.test();
    }
}
```
> 此外,不得不提一下Aware接口,它其实是一个空接口,里面不包含任何方法。它表示已感知的意思，通过这类接口可以获取指定对象,比如：通过BeanFactoryAware获取BeanFactory.通过ApplicationContextAware获取ApplicationContext.通过BeanNameAware获取BeanName等.

## 1.2 谈谈Spring中的NoSuchBeanDefinitionException
> org.springframework.beans.factory.NoSuchBeanDefinitionException是很常见的异常,可以说绝大多数使用过Spring的人都曾遇到过它。本文旨在总结下NoSuchBeanDefinitionException(以下简称 NSBDE)的含义,哪些情况下可能抛出 NSBDE,和如何解决(文中配置均用JavaConfig)

> 情况1: No qualifying bean of type […] found for dependency
> 最常见的抛出 NSBDE 的情况就是在一个 BeanA 中注入 BeanB 时找不到 BeanB 的定义。例子如下
```
@Component
public class BeanA {
    @Autowired
    private BeanB dependency;
    //...
}
```
> 当在 BeanA 中注入BeanB 时,如果在 Spring 上下文中找不到BeanB的定义,就会抛出 NSBDE.异常信息如下:
> 抛异常的原因在异常信息中说的很清楚:expected at least 1 bean which qualifies as autowire candidate for this dependency。所以要么是BeanB不存在在 Spring 上下文中（比如没有标注 @Component,@Repository,@Service,@Controller等注解),要么就是 BeanB 所在的包没有被 Spring 扫描到.
> 解决办法就是先确认 BeanB 有没有被某些注解声明为 Bean:
```
package org.baeldung.packageB;
@Component
public class BeanB { ...}
```
> 如果BeanB已经被声明为一个 Bean,就再确认BeanB所在的包有没有被扫描.
```
@Configuration
@ComponentScan("org.baeldung.packageB")
public class ContextWithJavaConfig {
}
```
> 情况2: No qualifying bean of type […] is defined
> 还有一种可能抛出NSBDE的情况是在上下文中存在着两个Bean,比如有一个接口IBeanB,它有两个实现类BeanB1和BeanB2.
```
@Component
public class BeanB1 implements IBeanB {
    //
}
@Component
public class BeanB2 implements IBeanB {
    //
}
```
> 现在,如果BeanA按照下面的方式注入,那么Spring 将不知道要注入两个实现中的哪一个,就会抛出NSBDE
```
@Component
public class BeanA {
    @Autowired
    private IBeanB dependency;
}
```

> Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: 
  No qualifying bean of type
    [org.baeldung.packageB.IBeanB] is defined: 
      expected single matching bean but found 2: beanB1,beanB2
> 仔细看异常信息会发现，并不是直接抛出 NSBDE，而是它的子类 NoUniqueBeanDefinitionException，这是 Spring 3.2.1 之后引入的新异常，目的就是为了和第一种找不到 Bean Definition 的情况作区分。解决办法1就是利用 @Qualifier 注解，明确指定要注入的 Bean 的名字(BeanB2 默认的名字就是 beanB2).
```
@Component
public class BeanA {
    @Autowired
    @Qualifier("beanB2")
    private IBeanB dependency;
}
```
> 除了指定名字,我们还可以将其中一个 Bean 加上 @Primary的注解,这样会选择加了 Primary 注解的 Bean 来注入,而不会抛异常,这样Spring就能够知道到底应该注入哪个Bean了.
```
@Component
@Primary
public class BeanB1 implements IBeanB {
    //
}
```
> 情况3: No Bean Named […] is defined
> NSBDE 还可能在从 Spring 上下文中通过名字获取一个 Bean 时抛出.
```
@Component
public class BeanA implements InitializingBean {
    @Autowired
    private ApplicationContext context;
    @Override
    public void afterPropertiesSet() {
        context.getBean("someBeanName");
    }
}
```
> 在这种情况中,如果找不到指定名字Bean的 Definition，就会抛出如下异常:
> Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: 
  No bean named 'someBeanName' is defined