---
layout: post
title: "@Autowiring EJBs with Spring"
date: 2013-06-19 20:00
comments: true
sharing: true
published: true
categories:
 - autowiring
 - factory bean
 - spring framework
 - development
---
Предположим, что у нас есть проект на Spring, в котором необходимо использовать внешние EJB. Для получения бинов необходимо создавать [InitialContext](http://docs.oracle.com/javase/6/docs/api/javax/naming/InitialContext.html) и делать `lookup()` нужных ejb. Но эту задачу можно автоматизировать и пользоваться [@Autowired](http://static.springsource.org/spring/docs/3.2.x/javadoc-api/org/springframework/beans/factory/annotation/Autowired.html), то есть код будет выглядеть вот так:

{% codeblock lang:java %}
@Service
public class UserService {

  @Autowired
  private UserRemoteBean userRemoteBean;

}
{% endcodeblock %}

Для этого в Spring существует [FactoryBean](http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/htmlsingle/#beans-factory-extension-factorybean) - класс, который знает, как создавать бины нужного типа. Собственно, нам необходимо написать такой класс (код на [Scala](http://www.scala-lang.org/)):

{% codeblock lang:scala %}
@Component
class UserRemoteBeanFactoryBean extends FactoryBean[UserRemoteBean] {

  def ACCESS_BEAN_REMOTE_NAME: String = "our ejb name."

  private val log: Logger = LoggerFactory.getLogger(getObjectType)

  var ctx: Context = null

  def getObject: UserRemoteBean = {
    log.debug("Requesting new instance of {}", getObjectType)

    try {
      ctx.lookup(ACCESS_BEAN_REMOTE_NAME).asInstanceOf[UserRemoteBean]
    }
    catch {
      case e: NamingException => {
        log.error("Unable to get ejb: {}", Array[AnyRef](e.getMessage, e): _*)
        throw new RuntimeException("Unable to get UserRemoteBean bean " + e.getMessage, e)
      }

    }
  }

  @PostConstruct
  protected def init() {
    val jndiProperties: Hashtable[String, Object] = new Hashtable[String, Object]
    jndiProperties.put(Context.URL_PKG_PREFIXES, "org.jboss.ejb.client.naming")
    jndiProperties.put("jboss.naming.client.ejb.context", new java.lang.Boolean(true))
    try
      ctx = new InitialContext(jndiProperties)
    catch {
      case e: NamingException => log.error("Unable to create JNDI Context: {}", Array[AnyRef](e.getMessage, e): _*)
    }
  }

  def isSingleton: Boolean = {
    false
  }

  def getObjectType: Class[_] = classOf[UserRemoteBean]
}
{% endcodeblock %}
