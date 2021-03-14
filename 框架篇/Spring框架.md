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

## 1.2 如何初始化Spring的Bean?
