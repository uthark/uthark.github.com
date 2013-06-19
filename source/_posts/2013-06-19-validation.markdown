---
layout: post
title: "Валидация входных параметров с использованием Spring"
date: 2013-06-19 07:00
comments: true
sharing: true
published: true
categories:
 - beanvalidation
 - hibernate validator
 - spring framework
 - spring aop
 - validation
---

Очень часто возникает задача проверки входных параметров в сервис на корректность с точки зрения бизнес логики.

Эту задачу можно решить в лоб, написав вручную код валидации в каждом из методов сервиса, например, вот так:

{% codeblock lang:java %}
  public User save(User user) {
    if(user == null) {
        throw new IllegalArgumentException("User is null");
    }
    // other checks
    //...

    // business logic starts here...
    //...
    return savedUser;
  }
{% endcodeblock %}

Очевидно, что если в каждом методе делать такие проверки, то код бизнес-логики загрязняется проверками, что ухудшает читаемость кода. Решить эту проблему можно следующим образом:

1. Добавляем аннотации для валидации входных параметров.
{% codeblock lang:java %}
  public @NotNull User save(@NotNull User user) {
    // business logic starts here...
    //...
    return savedUser;
  }
{% endcodeblock %}

Чтобы заставить этот код работать, мы будем использовать возможности Spring Validation.

1. Spring предоставляет аннотацию [@Validated](http://static.springsource.org/spring/docs/3.2.x/javadoc-api/org/springframework/validation/annotation/Validated.html), которая в отличие от [@javax.validation.Valid](http://docs.oracle.com/javaee/7/api/javax/validation/Valid.html) позволяет определять группу валидации.
2. Этой аннотацией необходимо проаннотировать бин, методы которого необходимо валидировать.
{% codeblock lang:java %}
    @Validated
    @Service
    public class UserService {
        public @NotNull User save(@NotNull User user) {
            // business logic starts here...
            //...
            return savedUser;
        }
    }
{% endcodeblock %}

3. Для реализации самой валидации нам необходим валидатор. В качестве реализации [JSR-303](http://jcp.org/en/jsr/detail?id=303) можно использовать, например, [Hibernate Validator](http://www.hibernate.org/subprojects/validator.html). Для этого в `pom.xml` необходимо добавить требуемые зависимости:

{% codeblock lang:xml %}
    <dependency>
        <groupId>javax.validation</groupId>
        <artifactId>validation-api</artifactId>
        <version>1.0.0.GA</version>
    </dependency>

    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-validator</artifactId>
        <version>4.3.1.Final</version>
    </dependency>
{% endcodeblock %}

Обращаю внимание, что текущей на данный момент является [спецификация](http://beanvalidation.org/1.1/spec) [JSR-349 Bean Validation 1.1](http://jcp.org/en/jsr/detail?id=349), которая поддерживает валидацию методов из коробки, но текущая версия Spring 3.2 - не поддерживает новый API, поэтому необходимо использовать старую версию API, и, соответственно, реализацию. Релевантный баг в Spring [SPR-8199](https://jira.springsource.org/browse/SPR-8199), поддержка нового API ожидается в Spring Framework 4.0.

4. Необходимо объявить необходимые бины в конфигурации Spring:

{% codeblock lang:java %}
    @Bean
    public LocalValidatorFactoryBean validatorFactory() {
        LocalValidatorFactoryBean factoryBean = new LocalValidatorFactoryBean();
        factoryBean.afterPropertiesSet();
        return factoryBean;
    }

    @Bean
    public Validator validator() {
        return validatorFactory().getValidator();
    }

    @Bean
    public MethodValidationPostProcessor methodValidationPostProcessor() {
        return new MethodValidationPostProcessor();
    }

{% endcodeblock %}

5. На этом всё. Как видно, декларативная валидация входных параметров реализуется очень просто.
