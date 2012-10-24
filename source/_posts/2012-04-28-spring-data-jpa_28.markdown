---
layout: post
title: "Собственная реализация методов в Spring Data JPA"
date: 2012-04-28T15:03:00+07:00
categories:
 - java
 - jpa
 - разработка
 - spring data
---

Очевидно, что мы не всегда можем воспользоваться автоматической генерацией кода, предоставляемой [Spring Data JPA](http://www.springsource.org/spring-data/jpa). Например, у нас слишком сложный запрос, или нам необходимо вызвать процедуру в базе данных, либо у нас сложная бизнес-логика.

Рассмотрим следующий пример - например, нам нужна функциональность уникального счётчика, который мы решили реализовать с помощью последовательности (sequence).

Сначала определим интерфейс, в котором опишем все методы, которые мы будем реализовывать самостоятельно. В нашем случае, это будет только один метод:

{% codeblock lang:java %}
public interface UserRepositoryCustom {
    /**
     * Returns next unique id.
     *
     * @return next unique id.
     */
    Integer getNextUniqueId();
}

{% endcodeblock %}

Затем обновим объявление репозитория, чтобы он унаследовал новый интерфейс <tt>UserRepositoryCustom</tt>

{% codeblock lang:java %}

public interface UserRepository extends JpaRepository<User, Integer>, UserRepositoryCustom {
   ...
}

{% endcodeblock %}

Теперь напишем реализацию метода:

{% codeblock lang:java %}
public class UserRepositoryImpl implements UserRepositoryCustom {

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public Integer getNextUniqueId() {

        // When using Hibernate via JPA native queries fails with mapping exception, so just use Hibernate directly:
        Session session = (Session) entityManager.getDelegate();
        SQLQuery nativeQuery = session.createSQLQuery("SELECT \"nextval\"('unique_id_seq') ");
        List<BigInteger> list = nativeQuery.list();
        if (list.isEmpty()) {
            throw new IncorrectResultSizeDataAccessException(1);
        }

        BigInteger result = list.get(0);

        return result.intValue();
    }
}
{% endcodeblock %}

И, наконец, укажем Spring Data JPA, чтобы в качестве класса для прокси использовался наш класс с реализацией собственных методов. Для этого нам нужна ещё одна секция <tt>repositories</tt> в конфигурационном файле:

{% codeblock lang:xml %}
    <repositories base-package="[base.repository.package]"/>

    <repositories base-package="[base.repository.package]">
        <repository id="userRepository" custom-impl-ref="userRepositoryImpl"/>
    </repositories>

    <beans:bean id="userRepositoryImpl" class="...UserRepositoryImpl"/>
{% endcodeblock %}

Вот и всё.
