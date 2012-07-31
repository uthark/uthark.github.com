---
layout: post
title: "Упрощаем работу с JPA при помощи Spring Data JPA"
date: 2012-02-23T02:17:00+07:00
categories:
 - spring
 - разработка
 - spring data
---

<div class='post'>
<h3>Введение</h3>
<p>Уже прошло несколько лет с тех пор, как появился JPA. Работа с <a href="http://docs.oracle.com/javaee/5/api/javax/persistence/EntityManager.html">EntityManager</a> увлекательна, но разработчики пишут красивый API, а подробности работы с базой данных скрывают. При этом частая проблема - дублирование имплементации, когда из одного DAO в другой у нас плавно перекочёвывает один и тот же код, в лучшем случае этот код переносится в абстрактный базовый DAO. <a href="http://www.springsource.org/spring-data/jpa">Spring Data</a>  коренным образом решает проблему - при  его использовании остаётся только API на уровне интерфейсов, вся имплементация создаётся автоматически с использованием <a href="http://en.wikipedia.org/wiki/Aspect-oriented_programming">AOP</a>.</p>
<h3>История Spring Data</h3>

Несмотря на то, что проект только недавно достиг версии 1.0, у него достаточно богатая история - раньше он развивался в рамках проекта <a href="http://redmine.synyx.org/projects/show/hades">Hades</a>.

<h3>Объявление DAO-интерфейса</h3>

Итак, для начала нам необходимо объявить DAO-интерфейс, в котором мы будем объявлять методы для работы с сущностью.
<pre class="brush:java">
public interface UserRepository extends CrudRepository&lt;User, Long&gt; {
}
</pre>

Данного кода достаточно для обычного DAO с <a href="http://en.wikipedia.org/wiki/Create,_read,_update_and_delete">CRUD</a>-методами. 

<ul>
  <li><b>save</b> - сохраняет или обновляет переданную сущность.</li>
  <li><b>findOne</b> - ищет сущность по первичному ключу.</li>
  <li><b>findAll</b> - возвращает коллекцию всех сущностей</li>
  <li><b>count</b> - возвращает количество сущностей</li>
  <li><b>delete</b> - удаляет сущность</li>
  <li><b>exists</b> - проверяет, существует ли сущность с данным первичным ключом</li>
 
</ul>

Полный список методов, объявленный в CrudRepository можно посмотреть в <a href="http://static.springsource.org/spring-data/data-commons/docs/current/api/org/springframework/data/repository/CrudRepository.html">javadoc</a>.

В случае, если нам нужны не все методы, то есть возможность произвести наследование от интерфейса <a href="http://static.springsource.org/spring-data/data-commons/docs/current/api/org/springframework/data/repository/Repository.html">Repository</a> и перенести в наследника только те методы из интерфейса CrudRepository, которые нужны.

<h3>Поддержка сортировки и постраничного просмотра</h3>
Очень часто требующаяся функциональность - это возможность возвращать только часть сущностей из БД, например, для реализации постраничного просмотра в пользовательском интерфейсе. Spring Data и тут хорош и предоставляет нам возможность добавить такую функциональность в наш DAO. Для этого достаточно добавить объявление следующего метода в наш DAO-интерфейс:
<pre class="brush:java">
 Page&lt;User&gt; findAll(Pageable pageable);
</pre>
Интерфейс <a href="http://static.springsource.org/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Pageable.html">Pageable</a> инкапсулирует в себе сведения о номере запрашиваемой страницы, размере страницы, а также требуемой сортировке.

<h3>Ищем данные</h3>
Как правило, на обычных CRUD-ах DAO не заканчиваются и часто требуются дополнительные методы, которые возвращают только те сущности, которые удовлетворяют заданным условиям. На мой взгляд, Spring Data сильно упрощает жизнь в данной области.

Например, нам нужен методы для поиска пользователя по логину и по его e-mail адресу:

<pre class="brush:java">
 User findByLogin(String login);
 User findByEmail(String email);
</pre>

Все просто.

В случае, если нужны более сложные условия для поиска, то и это тоже реализовано.

Spring Data поддерживает следующие операторы:

<ul>
  <li>Between</li>
  <li>IsNotNull</li>
  <li>NotNull</li>
  <li>IsNull</li>
  <li>Null</li>
  <li>LessThan</li>
  <li>LessThanEqual</li>
  <li>GreaterThan</li>
  <li>GreaterThanEqual</li>
  <li>NotLike</li>
  <li>Like</li>
  <li>NotIn</li>
  <li>In</li>
  <li>Near</li>
  <li>Within</li>
  <li>Regex</li>
  <li>Exists</li>
  <li>IsTrue</li>
  <li>True</li>
  <li>IsFalse</li>
<li>False</li>
<li>Not</li>

</ul>
Такой внушительный список открывает простор для фантазии, так что можно составить сколь угодно сложный запрос. 

Если необходимо, чтобы в результатах поиска было несколько сущностей, то необходимо называть метод find<b>All</b>ByBlahBlah

<h3>Поддержка Spring MVC</h3>
<i>Это часть основана на официальной документации.</i>
Представьте, что вы разрабатываете веб-приложение с использованием <a href="http://static.springsource.org/spring/docs/current/spring-framework-reference/html/mvc.html">Spring MVC</a>. Тогда вам необходимо будет загружать сущность из базы данных используя параметры HTTP-запроса. Это может выглядеть следующим образом:
<pre class="brush:java">
@Controller
@RequestMapping("/users")
public class UserController {

  private final UserRepository userRepository;

  public UserController(UserRepository userRepository) {
    userRepository = userRepository;
  }

  @RequestMapping("/{id}")
  public String showUserForm(@PathVariable("id") Long id, Model model) {
    
    // Do null check for id
    User user = userRepository.findOne(id);
    // Do null check for user
    // Populate model
    return "user";
  }
}</pre>

Во-первых, вы объявляете зависимость на DAO, а во-вторых всегда вызываете метод <tt>findOne()</tt> для загрузки сущности. К счастью, Spring позволяет нам преобразовывать строковые значения из HTTP-запросов в любой нужный тип используя либо <tt><a href="http://docs.oracle.com/javase/6/docs/api/java/beans/PropertyEditor.html">PropertyEditor</a></tt>, либо <tt><a href="http://static.springsource.org/spring/docs/current/javadoc-api/org/springframework/core/convert/ConversionService.html">ConversionService</a></tt>.

Если вы используете Spring версии 3.0 и выше, то вам необходимо добавить следующую конфигурацию:
<pre class="brush:xml">
&lt;mvc:annotation-driven conversion-service="conversionService" /&gt;
&lt;bean id="conversionService" class="&#8230;.context.support.ConversionServiceFactoryBean"&gt;
  &lt;property name="converters"&gt;
    &lt;list&gt;
      &lt;bean class="org.springframework.data.repository.support.DomainClassConverter"&gt;
        &lt;constructor-arg ref="conversionService" /&gt;
      &lt;/bean&gt;
    &lt;/list&gt;
  &lt;/property&gt;
&lt;/bean&gt;
</pre>

Если же вы используете Spring более старой версии, то вам необходима вот такая конфигурация:

<pre class="brush:xml">
&lt;bean class="&#8230;.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter"&gt;
  &lt;property name="webBindingInitializer"&gt;
    &lt;bean class="&#8230;.web.bind.support.ConfigurableWebBindingInitializer"&gt;
      &lt;property name="propertyEditorRegistrars"&gt;
        &lt;bean class="org.springframework.data.repository.support.DomainClassPropertyEditorRegistrar" /&gt;
      &lt;/property&gt;
    &lt;/bean&gt;
  &lt;/property&gt;
&lt;/bean&gt;
</pre>

После данных изменений в конфигурации можно переписать контроллер следующим образом:

<pre class="brush:java">

@Controller
@RequestMapping("/users")
public class UserController {

  @RequestMapping("/{id}")
  public String showUserForm(@PathVariable("id") User user, Model model) {

    // Do null check for user
    // Populate model
    return "userForm";
  }
}
</pre>

Обратите внимание на то, как упростился код и как мы красиво избавились от его дублирования.


<h3>Документация</h3>
На данный момент документации по проекту не так уж и много, но, тем не менее, она есть:

<ul>
<li><a href="http://static.springsource.org/spring-data/data-jpa/docs/current/reference/html/">Spring Data JPA - Reference Documentation</a></li>
<li><a href="http://static.springsource.org/spring-data/data-commons/docs/current/reference/html/">Spring Data Commons - Reference Documentation</a></li>
<li><a href="https://github.com/SpringSource/spring-data-jpa">исходники на github</a></li>

</ul>
<h3>Заключение</h3>
Spring Data значительно упрощает жизнь при использовании JPA. Рекомендуется к использованию в своих проектах.</div>
<h2>Comments</h2>
<div class='comments'>
<div class='comment'>
<div class='author'>Oleg Atamanenko</div>
<div class='content'>
1. Разве в SQL есть операторы Near, Regex, Within?<br />2. Для монги есть отдельный проект - Spring Data MongoDB, который поддерживает вышеперечисленный операторы.<br />В документации всё описано - http://static.springsource.org/spring-data/data-mongodb/docs/current/reference/html/#mongo.query</div>
</div>
<div class='comment'>
<div class='author'>php-coder</div>
<div class='content'>
Где можно найти описание операторов Near, Within и Regex? В доке по spring-data-jpa я их не вижу. Не там смотрю? Или они не поддерживаются в JPA и могут быть использованы только с NoSQL БД, типа MongoDb?</div>
</div>
</div>
