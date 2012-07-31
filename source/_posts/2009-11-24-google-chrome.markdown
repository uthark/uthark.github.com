---
layout: post
title: "Создание страницы настроек для расширений Google Chrome"
date: 2009-11-24T23:09:00+06:00
categories:
 - extension
 - chromium
---

<div class='post'>
В продолжение <a href="http://atamanenko.blogspot.com/2009/11/delicious-bookmarks-google.html">предыдущей заметки</a><br />
<br />
Логично предположить, что у расширений могут быть настройки. В Google Chrome/Chromium для этого есть специальный API.<br />
<br />
Для того, чтобы создать собственную страницу настроек необходимо сделать следующее:<br />
<br />
1. Объявить в манифесте страницу настроек<br />
<pre class="brush:js; highlight: [5]">{
  "name": "Delicious plugin", 
  "version": "0.2", 
  "background_page": "background.html", 
  "options_page": "options.html"
}
</pre><br />
2. Реализовать страницу с настройками.<br />
<br />
Страница с настройками - это обычная HTML-страничка. Для доступа к настройкам Google Chrome предоставляет объект <tt>localStorage</tt>, который умеет сохранять и возвращать значения.<br />
<br />
С объектом <tt>localStorage</tt> работа идёт как с обычным hash. <br />
<br />
<pre class="brush:js; html-script: true; highlight:[09, 21, 20, 33, 41]">&lt;html&gt;
&lt;head&gt;
&lt;title&gt;Delicious Bookmarks Options&lt;/title&gt;
&lt;/head&gt;
&lt;script type=&quot;text/javascript&quot;&gt;
    // Saves options to localStorage.
    function saveOptions() {
        var share = document.getElementById(&quot;share&quot;);
        localStorage[&quot;markPrivate&quot;] = share.checked;

        // Update status to let user know options were saved.
        var status = document.getElementById(&quot;status&quot;);
        status.innerHTML = &quot;Options Saved.&quot;;
        setTimeout(function() {
            status.innerHTML = &quot;&quot;;
        }, 1500);
    }

    // Restores select box state to saved value from localStorage.
    function restoreOptions() {
        var share = localStorage[&quot;markPrivate&quot;];

        if (!share) {
            return;
        }
        var shareCheckbox = document.getElementById(&quot;share&quot;);

        shareCheckbox.checked = share;
        
    }
&lt;/script&gt;

&lt;body onload=&quot;restoreOptions()&quot;&gt;

&lt;label for=&quot;share&quot;&gt;Mark as Private&lt;/label&gt;
&lt;input type=&quot;checkbox&quot; class=&quot;checkbox&quot; name=&quot;share&quot; id=&quot;share&quot; /&gt;


&lt;br&gt;
&lt;button onclick=&quot;saveOptions();&quot;&gt;Save&lt;/button&gt;
&lt;div id=&quot;status&quot;&gt;&amp;nbsp;&lt;/div&gt;
&lt;/body&gt;
&lt;/html&gt;
</pre><br />
<h2>Чтение настроек</h2>При инициализации страницы настроек необходимо проставить актуальные значения. Для этого на событе <tt>onload</tt> навешивается функция <tt>restoreOptions()</tt>, которая проставляет в пользовательском интерфейсе текущие настройки. <br />
<br />
<h2>Запись настроек</h2>Для сохранения настроек навешивается обработчик <tt>onclick</tt> для кнопки save - метод <tt>saveOptions</tt>.<br />
<br />
Вот таким простым способом можно сохранять настройки расширения. <br />
<br />
<h1>Документация</h1><ul><li><a href="http://code.google.com/chrome/extensions/options.html">Google Chrome Extensions Options</a><br />
<li><a href="http://www.w3.org/TR/2009/WD-webstorage-20091029/#the-localstorage-attribute">Документация по localStorage</a></li><br />
<br />
</ul></div>
<h2>Comments</h2>
<div class='comments'>
<div class='comment'>
<div class='author'>karri</div>
<div class='content'>
Мне вот тут понадобилось поставить Flash player для Chromium&#39;a, перерыла кучу ссылок в инете и наткнулась на классный блог http://www.ubuntugeek.com/<br />Он, судя по всему, для таких чайников как я(все было понятно даже на английском), поставила player без проблем и терь сижу довольная до невозможности :) Оч рекомендую начинающим линуксоидам, может кому пригодится :)</div>
</div>
</div>
