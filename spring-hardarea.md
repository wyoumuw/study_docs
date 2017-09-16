#关于Spring4的@Bean

我们观察spring boot的源码的时候进场发现有类似于下面的写法
        
        @Configuration
        class Config{
        
            @Bean
            A a(){
               return new A();
            }
            
            @Bean
            B b(){
               return new B(a());
            }
        }
        org.springframework.context.annotation.ConfigurationClassPostProcessor
看到上面的代码我们很容易发现一个new B(a())难道这个a()和上面那个a()不是独立的方法调用生成了2个对象吗？
其实我们打debug的时候就发现，这个Config类被放到spring 容器的时候会被CGLIB给代理，enhancer为**org.springframework.context.annotation.ConfigurationClassEnhancer**。
我们看一下源码：


        1        private static class BeanMethodInterceptor implements MethodInterceptor, ConditionalCallback {
        2        		@Override
        2        		public Object intercept(Object enhancedConfigInstance, Method beanMethod, Object[] beanMethodArgs,
        3        					MethodProxy cglibMethodProxy) throws Throwable {
        4                            //这个就是拦截方法的方法拦截器的拦截主要逻辑
        5                            //获取BeanFactory也就是容器
        6        			ConfigurableBeanFactory beanFactory = getBeanFactory(enhancedConfigInstance);
        7        			//获取其bean的名字，因为拦截的是方法，会吧方法名当成bean名字
        8        			String beanName = BeanAnnotationHelper.determineBeanNameFor(beanMethod);
        9        
        10        			// 检测他的scope
        11        			Scope scope = AnnotatedElementUtils.findMergedAnnotation(beanMethod, Scope.class);
        12        			if (scope != null && scope.proxyMode() != ScopedProxyMode.NO) {
        13        				String scopedBeanName = ScopedProxyCreator.getTargetBeanName(beanName);
        14        				if (beanFactory.isCurrentlyInCreation(scopedBeanName)) {
        15        					beanName = scopedBeanName;
        16        				}
        17        			}
        18        
        19                            //判断是不是工厂bean---不是关键
        20        			if (factoryContainsBean(beanFactory, BeanFactory.FACTORY_BEAN_PREFIX + beanName) &&
        21        					factoryContainsBean(beanFactory, beanName)) {
        22        				Object factoryBean = beanFactory.getBean(BeanFactory.FACTORY_BEAN_PREFIX + beanName);
        23        				if (factoryBean instanceof ScopedProxyFactoryBean) {
        24        				    //如果这个bean是ScopedProxyFactoryBean代理工厂的bean的话不处理
        25        				}
        26        				else {
        27        					//否则生成一个代理对象返回
        28        					return enhanceFactoryBean(factoryBean, beanMethod.getReturnType(), beanFactory, beanName);
        29        				}
        30        			}
        31                            //判断是否调用当前的方法
        32        			if (isCurrentlyInvokedFactoryMethod(beanMethod)) {
        33        				// The factory is calling the bean method in order to instantiate and register the bean
        34        				// (i.e. via a getBean() call) -> invoke the super implementation of the method to actually
        35        				// create the bean instance.
        36        				if (logger.isWarnEnabled() &&
        37        						BeanFactoryPostProcessor.class.isAssignableFrom(beanMethod.getReturnType())) {
        38        					logger.warn(String.format("@Bean method %s.%s is non-static and returns an object " +
        39        									"assignable to Spring's BeanFactoryPostProcessor interface. This will " +
        40        									"result in a failure to process annotations such as @Autowired, " +
        41        									"@Resource and @PostConstruct within the method's declaring " +
        42        									"@Configuration class. Add the 'static' modifier to this method to avoid " +
        43        									"these container lifecycle issues; see @Bean javadoc for complete details.",
        44        							beanMethod.getDeclaringClass().getSimpleName(), beanMethod.getName()));
        45        				}
        46        				//直接调用方法上面示例中生成A对象a()这次是直接调用
        47        				return cglibMethodProxy.invokeSuper(enhancedConfigInstance, beanMethodArgs);
        48        			}
        49                           //从工厂中绑定实例返回，比如上例中的B生成时调用a()
        50        			return obtainBeanInstanceFromFactory(beanMethod, beanMethodArgs, beanFactory, beanName);
                        }
                
                        ....
                    }
        
这里说明下，上面的a()方法和b()方法都会被代理掉成为代理方法。
然后我们从上面可以理一下思路：
上例中顺序执行会先调用a()生成A对象到容器中对应47行，然后生成B对象b()方法的时候也会调用47行代码，但是调用到构造方法需要a()的时候会直接调用50行代码进行返回，这个时候返回值为从容器里返回的值。
上面的方法我们可以写个简单的伪码就是:

       Object proxyMethod(Method currentMethod){
            beanName=获取BeanName(currentMethod)
            if 容器里有这个bean吗
                return 容器的bean
             else
                return 调用当前的方法currentMethod
       
       }
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        