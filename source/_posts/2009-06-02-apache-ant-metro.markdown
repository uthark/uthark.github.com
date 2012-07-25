---
layout: post
title: "Apache Ant и Metro"
date: 2009-06-02T23:31:00+07:00
categories:
 - apache ant
 - java
 - грабли
---

<div class='post'>
Столкнулся с такой проблемой - в некоторых случаях приложение, собранное антом не работает. Конкретно - не работает десереализация запросов на веб-сервис. Получаемый запрос содержит <tt>null</tt> в значениях полей.

Опытным путём было выяснено, что на это влияет используемая версия Java. Под 1.5 работает корректно, под 1.6 - не работает. Нашли <a href="https://jaxb.dev.java.net/guide/Runtime_Errors.html#Illegal_class_modifiers_for_package_info__0x1600">ошибку в метро</a>, связанную именно с версией JDK и на этом успокоились. Но было одно исключение из правил. У одного из разработчиков под 1.5 тоже не работало.

Как выяснилось позже, виноват был во всё Ant.
Metro создавал файл package-info.java, в котором было примерно следующее содержание:

<code>
@javax.xml.bind.annotation.XmlSchema(namespace = "http://z.y.x.com", elementFormDefault=javax.xml.bind.annotation.XmlNsForm.UNQUALIFIED)
package com.x.y.z;
</code>

Ант, в свою очередь, должен был этот файл компилировать. А <a href="http://ant.apache.org/manual/CoreTasks/javac.html">в реальности этого не происходило</a>, поэтому Metro не мог корректно отобразить XML-запрос на объекты доменной области.

Решение было немного кривым, но при этом поразительно простым - принудительный вызов javac на файлах package-info.java.</div>
