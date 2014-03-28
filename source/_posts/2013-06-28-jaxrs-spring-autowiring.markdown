---
layout: post
title: "@Autowired JAX-RS Client API"
date: 2013-06-28 19:05
comments: true
sharing: true
published: true
categories:
 - jax-rs client
 - spring
 - development
 - autowiring
---

Продолжая разговор о [JAX RS](http://www.jcp.org/en/jsr/detail?id=339) [Client API](https://jax-rs-spec.java.net/nonav/2.0/apidocs/javax/ws/rs/client/package-summary.html) - предположим, что мы уже [используем JAX-RS клиент](/blog/2013/06/28/jaxrs/)

У нас есть класс, который умеет создавать прокси для любого REST-интерфейса в проекте. Теперь мы хотим сделать так, чтобы эти интерфейсы можно было автоматически создавать в контексте Spring и связывать с другими бинами.

Первое решение, которое приходит в лоб - объявить бин в конфигурации для каждого интерфейса:

{% codeblock lang:java %}
@Configuration
public class SpringConfiguration {
    @Bean
    public UserRest userRest(){
        List<?> providers = Arrays.asList(new JsonMessageHandler(new ObjectMapper()));
        return JAXRSClientFactory.create("http://localhost:8080", UserRest.class, providers);
    }
}

{% endcodeblock %}

Написав объявление нескольких таких бинов можно задуматься - "Есть ли способ проще?".

Есть, и я вам сейчас его покажу.

Для того, чтобы добавлять собственные бины в Spring контекст, мы воспользуемся возможностью расширения Spring контекста с использованием [`BeanFactoryPostProcessor`](http://static.springsource.org/spring/docs/3.2.x/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html).
Метод `postProcessBeanFactory` данного класса позволяет выполнить дополнительную обработку фабрики бинов Spring, например, удалить, добавить, переопределить бин.
Это именно то, что нам нужно - автоматически добавить новые бины в фабрику.

Итак, что нам нужно сделать:

1. Найти все доступные REST интерфейсы в текущем classpath.
2. Для каждого из них создать прокси с использованием `JAXRSClientFactory` .
3. Зарегистрировать каждый прокси в фабрике бинов для дальнейшего использования (например, для @Autowired).

По спецификации JAX-RS, чтобы класс распознавался как REST-ресурс, у него должна быть аннотация [`@Path()`](https://jax-rs-spec.java.net/nonav/2.0/apidocs/javax/ws/rs/Path.html).

Для поиска всех возможных классов/интерфейсов воспользуемся классом [`ClassPathScanningCandidateComponentProvider`](http://static.springsource.org/spring/docs/3.2.x/javadoc-api/org/springframework/context/annotation/ClassPathScanningCandidateComponentProvider.html), который умеет сканировать классы в выбранном пакете и применять фильтры, чтобы собрать только нужные. Также нам нужно учесть, что по умолчанию `ClassPathScanningCandidateComponentProvider` пытается определить, может ли найденный класс быть и бином (например, проверяет, что это не абстрактный класс и не интерфейс), поэтому нам нужно написать подкласс, который позволит нам работать с интерфейсами.

Пишем класс:
{% codeblock lang:java %}
public class RestClientPostProcessor implements BeanFactoryPostProcessor {

    private static final Logger LOGGER = LoggerFactory.getLogger(RestClientPostProcessor.class);

    protected String endpoint = "http://localhost:8080";

    private Class<? extends Annotation> requiredAnnotation = Path.class;

    private String basePackage = "<base package>";

    private ObjectMapper objectMapper;

    // getters/setters omitted.

    protected Object createBean(Class<?> clazz) {
        List<?> providers = Arrays.asList(new JsonMessageHandler(objectMapper));
        return JAXRSClientFactory.create(endpoint, clazz, providers);
    }

    @Override 
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {

        ClassPathScanningCandidateComponentProvider provider = new ClasspathScanner();
        provider.addIncludeFilter(new AnnotationTypeFilter(requiredAnnotation));
        Set<BeanDefinition> components = provider.findCandidateComponents(basePackage);

        for (BeanDefinition component : components) {
            createAndRegisterBean(beanFactory, component);

        }
    }

    protected void createAndRegisterBean(ConfigurableListableBeanFactory beanFactory, BeanDefinition component) {
        try {
            String beanClassName = component.getBeanClassName();
            Class<?> clazz = Class.forName(beanClassName);

            Object o = createBean(clazz);

            beanFactory.registerResolvableDependency(clazz, o);
        } catch (ClassNotFoundException e) {
            LOGGER.warn("Unable to find class: {}", component.getBeanClassName(), e);
        }
    }

    private static class ClasspathScanner extends ClassPathScanningCandidateComponentProvider {

        public ClasspathScanner() {
            super(false);
        }

        @Override protected boolean isCandidateComponent(AnnotatedBeanDefinition beanDefinition) {
            // override this method, because our classes are interfaces, by default interfaces are
            // not allowed.
            return beanDefinition.getMetadata().isIndependent();
        }
    }
}

{% endcodeblock %}

Теперь этот процессор необходимо зарегистрировать в контексте Spring:
{% codeblock lang:java %}
@Configuration
class TestAutowiringConfiguration {

    @Bean
    public static RestClientPostProcessor autowiringRestApiProcessor() {
        return new RestClientPostProcessor();
    }
}
{% endcodeblock %}

И теперь можно использовать `UserRest` как обычный бин:
{% codeblock lang:java %}

    @Autowired
    private UserShowRest userShowRest;

{% endcodeblock %}

Если есть вопросы - [спрашивайте в Google+](https://plus.google.com/112372998073079463630/posts)