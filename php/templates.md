# Шаблоны в PHP

Это перепечатка с сокращениями статьи, которая была опубликована на несуществующем более сайте `http://www.phpinfo.su/articles/practice/shablony_v_php.html` ([оригинал в веб-архиве](http://web.archive.org/web/20161119062218/http://www.phpinfo.su/articles/practice/shablony_v_php.html)), автор: phpinfo.su

В конце добавлен маленький обзор современных шаблонизаторов.

## Истоки шаблонизации

Давным-давно, после появления PHP, один умный человек сказал, что для того, чтобы программа на PHP оставалась легко модифицируемой и расширяемой, **нужно отделять код скрипта от кода шаблона**. Эту мудрую мысль подхватило сообщество php-программистов и началось: были написаны десятки книг, руководств и статей, авторы которых делились соображениями насчет того, как можно более-менее грамотно **отделить PHP от HTML**. Так появился известный [шаблонизатор](https://ru.wikipedia.org/wiki/%D0%A8%D0%B0%D0%B1%D0%BB%D0%BE%D0%BD%D0%B8%D0%B7%D0%B0%D1%82%D0%BE%D1%80) [Smarty](https://www.smarty.net/about_smarty), так появились и другие шаблонные решения. Заметьте, я не зря выделил жирным шрифтом две фразы — не смотря на схожесть, они несут разный смысл.

## Логика приложения и логика отображения

Давайте разберем мудрую мысль безымянного гения. Что имел в виду автор, когда сказал, что нужно отделять код скрипта от кода шаблона? Возьмем в пример банальную программу — скрипт, который складывает два значения:

```php
<?php
$result = $a + $b;
```

Эта незамысловатая операция по праву может называться **бизнес-логикой** или **логикой приложения**. Иначе говоря — это суть программы. Ничего больше от программы не требуется, кроме как вычислить сумму двух слагаемых. В конечном итоге данная программа (при условии, что значения переменных `$a` и `$b` определены) может получить два различных типа значения — либо ноль, либо отрицательное или положительное число.

Поскольку программа используется в web, то логично было бы отдавать результат её выполнения в виде HTML. При этом хотелось бы применить некоторую *логику* при выводе HTML-кода — если результат не равен нулю — вывести результат синеньким текстом, иначе вывести сообщение красным цветом, что мол извините, "бублик", тобишь ноль.

### Новичок

Рассмотрим решение данной задачи новичком. Новичок ничего не слышал об отделении php-кода скрипта от html-кода шаблона и наверняка напишет программу примерно так:

```php
<?php
echo "<!DOCTYPE HTML PUBLIC \"-//W3C//DTD HTML 4.0 Transitional//EN\">";
echo "<html>";
echo "<head>";
echo "   <title>Основной шаблон HTML-страницы</title>";
echo "</head>";
echo "<body>";

$a = isset($_GET['a']) && is_numeric($_GET['a']) ? $_GET['a'] : 0;
$b = isset($_GET['b']) && is_numeric($_GET['b']) ? $_GET['b'] : 0;

$result = $a + $b;

if ($result) {
    echo "<span style=\"color:blue; font-weight:bold\">Результат: $result</span>";
} else {
    echo "<span style=\"color:red; font-weight:bold\">Результат равен нулю!</span>";
}

echo "</body>";
echo "</html>";
```

Какие ошибки совершил новичок? Он смешал PHP код (логику приложения) и **логику отображения**. Что такое логика отображения? Это условие в управляющей конструкции if-else, выводящее HTML-код в зависимости от полученного результата. **Логика отображения никак не связана с логикой приложения**, она даже не знает, как был получен `$result` — через сложение двух переменных или через сложные математические алгоритмы, сопровождающиеся выборками из базы и запросами к стороннему серверу. Ей это всё равно, у логики отображения другая задача — показать пользователю результат работы программы.

> Примечание: помимо всего прочего новичок вывел HTML через echo заключив HTML в двойные кавычки, что привело к экранированию двойных кавычек в HTML-коде. Получилась смесь из HTML и PHP кода, трудно читаемая, трудно поддерживаемая и совершенно не красивая.

Теперь представим, что новичок написал целый интернет-магазин в подобном стиле, смешав воедино выборки из базы, алгоритмы и HTML. Потом верстальщику понадобилось изменить значительную часть HTML кода и вуаля — код поддерживать невозможно, не только верстальщиком, но и самим программистом. Бизнес-логика переплетена в логикой отображения, смешались в кучу кони, люди.

Но оставим новичка в покое. Все PHP-программисты так начинали и автор данной статьи тоже не исключение.

### Студент

Рано или поздно веб-программист начинает становиться опытнее и к нему приходит понимание, что отделять HTML-код от PHP-кода всё же нужно. Проштудировав форумы и некоторые руководства (а быть может и по собственной смекалке) программист пишет программу, которая в коде HTML-шаблона, на месте определенных меток типа `%var%` или `{var}`, подставляет значения, полученные от PHP-скрипта:

Скрипт script.php:

```php
<?php
$a = isset($_GET['a']) && is_numeric($_GET['a']) ? $_GET['a'] : 0;
$b = isset($_GET['b']) && is_numeric($_GET['b']) ? $_GET['b'] : 0;

$result = $a + $b;

if ($result) {
    $body = "<span style=\"color:blue; font-weight:bold\">Результат: $result</span>";
} else {
    $body = "<span style=\"color:red; font-weight:bold\">Результат равен нулю!</span>";
}

// загружаем содержимое файла шаблона в строку
$tpl = file_get_contents('template.html');
// меняем в шаблоне метку {body} на переменную $body
$tpl = str_replace('{body}', $body, $tpl);
echo $tpl;
```

Шаблон template.html:

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN">
<html>
<head>
    <title>Основной шаблон HTML-страницы</title>
</head>
<body>
{body}
</body>
</html>
```

Что получается? Основной каркас страницы лежит в отдельном файле и уже не связан с PHP-кодом, уже лучше. Но **в скрипте script.php по прежнему присутствует HTML-код и логика отображения**! Получается, променяли "шило на мыло". HTML теперь размазан как по скрипту, так и по основному шаблону, что не очень отличается от кода "Новичка" из примера выше.

### Умник

Следующий шаг — это терминальная стадия, когда программист в *попытке отделить PHP от HTML* начинает писать свой собственный шаблонизатор — набор правил для шаблона, которые могли бы выполнять хотя бы минимальные логические операции с данными, полученными из PHP-скрипта:

Скрипт script.php:

```php
<?php
$a = isset($_GET['a']) && is_numeric($_GET['a']) ? $_GET['a'] : 0;
$b = isset($_GET['b']) && is_numeric($_GET['b']) ? $_GET['b'] : 0;

$result = $a + $b;

// загружаем содержимое файла шаблона в строку
$tpl = file_get_contents('template.html');
// запускаем наш супер-мега самописный шаблонизатор и передаем в него данные из 
// php-скрипта в виде пар ключ => значение
$tpl = super_mega_template_engine( array('result' => $result) );
echo $tpl;
```

Шаблон теперь выглядит так:

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN">
<html>
<head>
    <title>Основной шаблон HTML-страницы</title>
</head>
<body>
    <if[$result] {<span style="color:blue; font-weight:bold">Результат: [$result] </span>}
                 {<span style="color:red; font-weight:bold">Результат равен нулю</span>}
    </if>
</body>
</html>
```

В шаблоне появляется IF-подобная конструкция, которую обрабатывает пользовательский шаблонизатор super_mega_template_engine. И вроде бы она работает — выводит значение в зависимости от значения переменной body. Но дальше начинается самое веселое — с усложнением логики отображения требуется создать в шаблонизаторе конструкции, обрабатывающие массивы — циклы. Тривиального IF-подобного синтаксиса начинает не хватать — нужны уровни вложенности и многое другое. В конечном итоге программист приходит на форум и задается вопросом — как написать свой собственный шаблонизатор на PHP.

## А все дело в...

А всё дело в упомянутой в начале статьи фразе об отделении кода скрипта PHP от кода HTML шаблона. Так получилось, что огромная программистская общественность в буквальном смысле слова не поняла посыл неизвестного автора — **отделять PHP от HTML не нужно! Нужно отделять логику приложения от логики отображения**, но это не значит, что в HTML-шаблоне мы не можем использовать PHP-код. PHP изначально задумывался как язык, позволяющий делать вставки кода в HTML страницы:

> PHP сконструирован специально для ведения Web-разработок и [его код может внедряться непосредственно в HTML](http://php.net/manual/ru/intro-whatis.php) — php.net.

Что из этого следует? **PHP — сам по себе является не только очень мощным языком программирования, но и самодостаточным шаблонизатором**, позволяющим делать качественные шаблоны без ущерба для логики приложения. Для этого надо соблюсти следующие условия:

- Не использовать в шаблонах логику приложения, передавать в шаблоны только данные, полученные из скрипта — скаляры, массивы, объекты. Никаких вызовов к базе, алгоритмов не связанных с логикой отображения и т.п.
- Использовать в шаблонах [структуры управления PHP](http://php.net/manual/ru/language.control-structures.php), необходимые для логики отображения - IF/ELSEIF/ELSE, FOR/FOREACH, INCLUDE/REQUIRE.
- Использовать для структур управления [альтернативный синтаксис](http://www.php.net/manual/ru/control-structures.alternative-syntax.php) — он **очень** упрощает чтение HTML-шаблонов.
- Стараться не использовать встроенные функции в шаблонах. Если даже вам нужно применить в шаблоне довольно часто используемую функцию [htmlspecialchars](http://php.net/manual/ru/function.htmlspecialchars.php), то не поленитесь обернуть её в статический метод класса-помощника или в функцию. Это в дальнейшем даст больший простор для рефакторинга и просто создаст единобразный стиль вашего API. 

Такой стиль шаблонизации на PHP называется pure-шаблонизация, т.е. чистая шаблонизация, основанная на возможностях самого PHP.

Используя pure-шаблонизацию код нашего скрипта и шаблона мог бы выглядеть так:

Скрипт:

```php
<?php
$a = isset($_GET['a']) && is_numeric($_GET['a']) ? $_GET['a'] : 0;
$b = isset($_GET['b']) && is_numeric($_GET['b']) ? $_GET['b'] : 0;

$result = $a + $b;

// загружаем шаблон 
include('template.html');
```

Шаблон:

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN">
<html>
<head>
    <title>Основной шаблон HTML-страницы</title>
</head>
<body>
    <?php if ($result): ?>
        <span style="color:blue; font-weight:bold">Результат: <?=$result?></span>
    <? else: ?>
        <span style="color:red; font-weight:bold">Результат равен нулю</span>
    <? endif; ?>
</body>
</html>
```

Согласитесь, красиво и совершенно просто! Мы разделили логику приложения и логику отображения. Теперь верстальщик, хоть немного знакомый с тривиальными управляющими конструкциями любого языка программирования, может с легкостью поддерживать HTML-код, а программисту нет необходимости что-либо знать о том, как и где будет выведен результат работы программы. Мы разделили обязанности и создали поддерживаемый код, который легко модифицировать.

Конечно, наш пример очень прост, но приемущества pure-шаблонизации очень заметны на реальных проектах.

### Пример шаблона посложнее — гостевая книга, вывод записей

В качестве примера посложнее можно привести шаблон гостевой книги, выводящей записи из заранее сформированного массива $guestbook_messages. Выводятся записи как зарегистрированных, так и незарегистрированных пользователей. Кроме того, возможен вывод сообщения администратора гостевой книги (если оно есть) под определенным сообщением пользователя.

В массиве присутствуют следующие ключи:

- $guestbook_messages['user_id'] — ID зарегистрированного пользователя. Если его нет, значит пользователь — анонимный.
- $guestbook_messages['user_name'] — Имя зарегистрированного пользователя. Если его нет, значит пользователь — анонимный.
- $guestbook_messages['user_ip'] — IP-адрес пользователя.
- $guestbook_messages['user_message'] — Сообщение пользователя.
- $guestbook_messages['date'] — Дата публикации сообщения.
- $guestbook_messages['admin_answer'] — Сообщение администратора, относящееся к записи пользователя.

```php
<!DOCTYPE html>
<html>
<head>
<title>Гостевая книга</title>
</head>
<body>

<?php if ($guestbook_messages): ?>
    <?php foreach ($guestbook_messages as $message): ?>
    
        <?php if ($message['user_id']): ?>
            <p class="register_user_info">Пользователь: 
            <a href="/users/<?=$message['user_id']?>.html">
                <?=htmlspecialchars($message['user_name'])?>
            </a>
            </p>
        <?php else: ?>
            <p class="anonim_user_info">Анонимный пользователь с IP <?=$message['user_ip']?></p>
        <?php endif; ?>
        
        <div class="message"><?=htmlspecialchars($message['user_message'])?></div>
        
        <div class="date"><?=date(DATE_W3C, $message['date'])?></div>
        
        <?php if ($message['admin_answer']): ?>
            <div class="answer"><?=htmlspecialchars($message['admin_answer'])?></div>
        <?php endif; ?>
        
    <?php endforeach; ?>
<?php else: ?>

    <p>В гостевую книгу ещё не добавлено ни одной записи</p>
    
<?php endif;?>

</body>
</html>
```

## Экранирование данных, пришедших извне

Отдельного внимания заслуживает тема экранирования данных. Хороший тон веб-программирования — записывать пришедшие от пользователя данные "как есть", а при выводе подвергать их обработке, в зависимости от требуемого формата вывода. Например, если это стандартное веб-приложение, то необходимо экранировать спецсимволы, использующиеся в HTML через функцию [htmlspecialchars](http://www.php.net/manual/ru/function.htmlspecialchars.php), что бы предотвратить [XSS-уязвимости](../security/xss.md). Наоборот, для Excel/Word формата нет необходимости применять htmlspecialchars для данных из базы веб-приложения. Т.е. каждый формат, в котором будут выводится данные, диктует свои правила обработки этих данных.

После выхода в свет этой статьи на форуме phpclub.ru было [обсуждение](http://phpclub.ru/talk/threads/Хорошие-статьи-по-шаблонам-в-php.72476/) — на каком этапе лучше форматировать данные, которые пришли в базу из пользовательского ввода (как пример — сообщения в гостевой книге, которые могут содержать HTML и JavaScript код). Львиная доля разработчиков согласилась с утверждением, что данные лучше форматировать не в php-скрипте, а в шаблоне. Причина тому в том, что форматов вывода данных может быть много, а php-скрипт, получающий данные из базы, как правило один общий. Соответственно знание о том, как форматировать данные должен принимать обработчик, который занимается отображением данных, а не общий PHP-скрипт, генерирующий эти данные.

Как пример — все те же сообщения из гостевой книги пользователя. Пользователь Хакер ввёл в текст сообщения JavaScript код, который бесконечно будет показывать alert-сообщение с надписью "Ты дурак!". Применив htmlspecialchars мы экранировали спецсимволы HTML, что в конечном итоге дало отображение HTML кода как текста. JavaScript код не сработал:

```php
<?php
// Сообщения нашей гостевой книги
$guestbook_messages = array(
    array('name' => 'Вася', 'message' => 'Хороший сайт!'),
    array('name' => 'Хакер', 'message' => '<script>while(1)alert("Ты дурак!")</script>'),
);
?>

<html>
<head>
<title>Моя гостевая книга</title>
</head>
<body>
    <?php foreach($guestbook_messages as $message): ?>
        <p><b><?=htmlspecialchars($message['name'])?></b>:</p>
        <p><?=htmlspecialchars($message['message'])?></p>
        <hr />
    <?php endforeach; ?>
</body>
</html>
```

Результат отображения в браузере:

> **Вася:**
>
> Хороший сайт!
> 
> ----------------------
> **Хакер:**
> 
> \<script\>while(1)alert("Ты дурак!")\</script\>
> 
> ------------------------

## Дополнение к оригинальной статье

Описанный выше подход - с использованием встроенных в PHP возможностей шаблонизации - хорошо работает в маленьких, простых скриптах, где не хочется подключать внешние библиотеки. На больших проектах сторонний шаблонизатор позволяет сделать код шаблонов более простым. На данный момент (2017 год) один из самых распространенных шаблонизаторов - [Twig](https://twig.symfony.com/), у него удобный синтаксис и много возможностей, которых нет во встроенном шаблонизаторе PHP: наследование шаблонов, автоэкранирование (не нужно вручную писать вызов htmlspecialchars или аналогичной функции), удаление лишних пробелов.

Вот пример простого шаблона на Twig, выводящего комментарии из примера гостевой книги выше: 

```twig
<body>
    {% for message in guestbook_messages %}
        <p><b>{{ message.name }}</b>:</p>
        <p>{{ message.message }}</p>
        <hr />
    {% else %}
        <p>Сообщений пока нету.</p>
    {% endfor %}
</body>
```

Также, некоторым нравится альтернативный синтаксис для HTML с отступами - [HAML](http://haml.info/tutorial.html), позволяющий генерировать HTML-разметку, используя меньший объем кода. HAML был придуман для использования с языком Руби, но позже и для других языков были сделаны похожие шаблонизаторы. На основе HAML был создан синтаксис шаблонов для JS под названием [Pug](https://github.com/pugjs/pug) (ранее он назывался Jade). 

Вот пример HAML-шаблона: 

```haml
%html
  %head
    %title Название страницы
  %body
    %h1 Заголовок страницы

    #content
      .left.column
        %p= print_information
      .right.column
        = render :partial => "sidebar"
```

И пример Pug-шаблона:

```pug
doctype html
html(lang="en")
  head
    title= pageTitle
  body
    h1 Pug - шаблонизатор для JS
    #container.col
      if youAreUsingPug
        p Вы молодец!
      else
        p Познакомьтесь с Pug!
      p.
        Pug - это лаконичный и простой язык шаблонов, уделяющий 
        внимание производительности и мощным возможностям.
```

Вместо тегов здесь используется CSS-подобный синтаксис, а вложенность элементов друг в друга задается отступами. Например, конструкция `#container.col` в коде выше развернется в HTML-код `<div id="container" class="col">...</div>`. Для PHP есть шаблонизаторы с поддержкой синтаксисов Pug и HAML:

- https://github.com/kylekatarnls/jade-php
- https://github.com/pug-php/pug
- https://github.com/arnaud-lb/MtHaml

К недостаткам HAML можно отнести неудобство работы с инлайновыми тегами, и то, что при отладке придется разбирать HTML-код, который сильно отличается по виду от шаблона. 

Также, довольно распространен синтаксис [Handlebars](http://handlebarsjs.com/)/[Mustache.js](https://mustache.github.io/). Эти шаблонизаторы изначально были придуманы для языка JavaScript, но есть их аналоги на PHP. Вот пример шаблона: 

```handlebars
<div class="post">
  <h1>Автор: {{fullName author}}</h1>
  <div class="body">{{body}}</div>

  <h1>Комментарии</h1>

  {{#each comments}}
  <h2>Автор: {{fullName author}}</h2>
  <div class="body">{{body}}</div>
  {{/each}}
</div>
```

Особенность этих шаблонизаторов в том, что в них очень сильно ограничен набор и возможности управляющих конструкций (вроде `if`), чтобы в шаблонах был минимум логики. Вот шаблонизаторы на PHP, поддерживающие этот синтаксис: 

- https://github.com/bobthecow/mustache.php
- https://github.com/zordius/lightncandy

Также ранее был популярен (но сейчас он используется реже) шаблонизатор на основе языка XSLT. [XSLT](https://ru.wikipedia.org/wiki/XSLT) - это язык, который позволяет преобразовать [XML](https://ru.wikipedia.org/wiki/XML)-документ в другой XML- или HTML-документ. Вот пример фрагмента XSLT-шаблона: 

```xslt
<xsl:for-each select="/guestbook-messages/message">
    <p>
        <b>Автор: <xsl:value-of select="@name" /></b>:
    </p>
    <p>
        <xsl:value-of select="@message" />
    </p>
    <hr />    
</xsl:for-each>
```

А вот пример XML-данных, которые нужно передать в этот шаблон: 

```xml
<guestbook-messages>
    <message name="Иван" message="Текст сообщения Ивана">
    <message name="Петр" message="Текст сообщения Петра">
</guestbook-messages>
```

В [статье Википедии про XSLT](https://ru.wikipedia.org/wiki/XSLT) есть гораздо больше примеров.

Преимуществом XSLT является строгость синтаксиса - он не пропустит незакрытые или несбалансированные HTML-теги в шаблоне. К недостаткам относится громоздкость синтаксиса, сложность расширения шаблонизатора пользовательскими функциями, необходимость преобразовывать все данные в XML вместо передачи напрямую в шаблонизатор. 

XSLT версии 1 поддерживается стандартным расширением к PHP: http://php.net/manual/ru/book.xsl.php

Для поддержки XSLT версии 2 придется устанавливать сторонние библиотеки: http://www.saxonica.com/html/saxon-c/

## Ссылки: 

- подробнее про встроенный в PHP шаблонизатор: http://php.net/manual/ru/language.basic-syntax.php , http://php.net/manual/ru/control-structures.alternative-syntax.php
- другие шаблонизаторы для PHP в репозитории packagist: https://packagist.org/?q=template&p=0
- поиск шаблонизаторов для PHP на phptrends: https://phptrends.com/top?q=template%20engine
