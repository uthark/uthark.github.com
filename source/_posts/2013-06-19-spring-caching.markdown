---
layout: post
title: "Использование memcached в качестве backend для Spring Caching Abstraction"
date: 2013-06-19 19:00
comments: true
sharing: true
published: true
categories:
 - cache
 - caching
 - spring framework
 - spring caching
 - memcached
---

В [Spring 3.1](http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/html/new-in-3.1.html) появился замечательный модуль - [Spring Cache](http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/html/cache.html), который является абстракцией над кэшированием, что позволяет декларативно реализовывать кэширование в приложение.

Я не буду вдаваться в подробности работы, их можно прочитать в документации, но опишу, каким образом можно настроить memcached в качестве бэкэнда для работы.

## Подключаем зависимости

{% codeblock lang:xml %}
<dependency>
    <groupId>com.google.code.simple-spring-memcached</groupId>
    <artifactId>spymemcached-provider</artifactId>
    <version>3.1.0</version>
</dependency>
<dependency>
    <groupId>com.google.code.simple-spring-memcached</groupId>
    <artifactId>spring-cache</artifactId>
    <version>3.1.0</version>
</dependency>
{% endcodeblock %}

`spymemcached-provider` - это библиотечка для работы с `memcached` из Java, а `spring-cache` - модуль интеграции со Spring.

## Включаем кэширование

{% codeblock lang:java %}

@Configuration("serviceConfiguration")
@EnableCaching(proxyTargetClass = true, mode = AdviceMode.PROXY)
@Import(CacheConfiguration.class)
public class ServiceConfiguration {
  // bean declarations goes here.
}
{% endcodeblock %}

Аннотация `@EnableCaching` включает кэширование. Конфигурацию бинов для кэширования мы выносим в отдельный класс, `CacheConfiguration`, чтобы не смешивать бины, отвечающие за кэширование с бинами, отвечающими за бизнес-логику.

Объявляем необходимые для работы бины:

{% codeblock lang:java %}
@Configuration
public class CacheConfiguration {

    @Value("${memcached.url}")
    private String memcachedUrl;

    @Bean
    public CacheManager cacheManager() throws Exception {
        SSMCacheManager result = new SSMCacheManager();
        result.setCaches(Arrays.asList(
                new SSMCache(defaultCacheFactory().getObject(), 45)
        ));

        return result;
    }

    @Bean
    public CacheFactory defaultCacheFactory() {

        CacheFactory factory = new CacheFactory();
        factory.setCacheName("defaultCache");
        factory.setAddressProvider(addressProvider());
        factory.setCacheClientFactory(cacheClientFactory());
        factory.setConfiguration(cacheConfiguration());

        return factory;
    }

    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE) CacheClientFactory cacheClientFactory() {
        return new MemcacheClientFactoryImpl();
    }

    @Bean
    AddressProvider addressProvider() {
        return new DefaultAddressProvider(memcachedUrl);
    }

    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    com.google.code.ssm.providers.CacheConfiguration cacheConfiguration() {
        com.google.code.ssm.providers.CacheConfiguration configuration =
                new com.google.code.ssm.providers.CacheConfiguration();
        configuration.setConsistentHashing(true);
        return configuration;
    }
}
{% endcodeblock %}

## Использование

{% codeblock lang:java %}
@Cacheable("defaultCache")
public List<User> getUser(@NotNull String username) throws IOException {
    // ...
}
{% endcodeblock %}
