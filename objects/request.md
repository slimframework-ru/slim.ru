---
title: Запрос - Request
---

Вашим маршрутам Slone и промежуточному программному обеспечению предоставляется объект запроса PSR 7, 
который представляет текущий HTTP-запрос, полученный вашим веб-сервером. Объект запроса реализует 
[PSR 7 ServerRequestInterface][psr7] с помощью которого вы можете проверять и обрабатывать 
метод HTTP-запросов, заголовки и тело.
                                     


[psr7]: http://www.php-fig.org/psr/psr-7/#3-2-1-psr-http-message-serverrequestinterface

## Как получить объект Request

Объект запроса PSR 7 вводится в ваши маршруты Slim-приложений в качестве первого 
аргумента для обратного вызова маршрута следующим образом:

```php
<?php
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\ResponseInterface;

$app = new \Slim\App;
$app->get('/foo', function (ServerRequestInterface $request, ResponseInterface $response) {
    // Use the PSR 7 $request object

    return $response;
});
$app->run();
```

<figcaption>Figure 1: Запрос Inject PSR 7 в обратный вызов маршрута приложения.</figcaption>


Объект запроса PSR 7 вводится в ваше _middleware_ Slim-приложения в качестве первого 
аргумента промежуточного программного обеспечения, вызываемого следующим образом:


```php
<?php
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\ResponseInterface;

$app = new \Slim\App;
$app->add(function (ServerRequestInterface $request, ResponseInterface $response, callable $next) {
    // Use the PSR 7 $request object

    return $next($request, $response);
});
// Define app routes...
$app->run();
```
<figcaption>Figure 2: Запрос Inject PSR 7 в промежуточное программное обеспечение приложения.</figcaption>


<div id='the-request-method'>
</div>

## Метод запроса

Каждый HTTP-запрос имеет метод, который обычно является одним из следующих:

* GET
* POST
* PUT
* DELETE
* HEAD
* PATCH
* OPTIONS

Вы можете проверить метод HTTP-запроса с соответствующим методом имени объекта Request  `getMethod()`.

```php
    $method = $request->getMethod();
```

Поскольку это обычная задача, встроенная реализация PSR 7 Slim также предоставляет эти запатентованные методы, которые возвращают
`true` или `false`.

* `$request->isGet()`
* `$request->isPost()`
* `$request->isPut()`
* `$request->isDelete()`
* `$request->isHead()`
* `$request->isPatch()`
* `$request->isOptions()`

Это возможно подменить или _переопределить_ метод HTTP запроса. Это полезно, если например, вам нужно подражать `PUT` запросу, используя традиционный веб-браузер, который поддерживает `GET` или `POST` запроса.

Существует два способа переопределить метод HTTP-запроса. Вы можете включить 
`_METHOD` параметр в `POST` тело запроса. HTTP-запрос должен использовать
`application/x-www-form-urlencoded` тип содержимого.

```php
POST /path HTTP/1.1
Host: example.com
Content-type: application/x-www-form-urlencoded
Content-length: 22

data=value&_METHOD=PUT
```
<figure>
<figcaption>Figure 3: Переопределить метод HTTP с параметром _METHOD.</figcaption>
</figure>

Вы также можете переопределить метод HTTP-запроса с помощью настраиваемого
`X-Http-Method-Override` заголовка HTTP-запроса. Это работает с любым типом содержимого HTTP-запроса.

```php
POST /path HTTP/1.1
Host: example.com
Content-type: application/json
Content-length: 16
X-Http-Method-Override: PUT

{"data":"value"}
```
<figure>
<figcaption>Figure 4: Переопределить метод HTTP с заголовком X-Http-Method-Override.</figcaption>
</figure>

Вы можете получить _оригинальный_ (не переопределенный) HTTP метод с помощью метода объекта запроса PSR 7  `getOriginalMethod()`.

## URI запроса

Каждый HTTP-запрос имеет URI, который идентифицирует запрашиваемый ресурс приложения. URI запроса HTTP имеет несколько частей:

* Схема (например. `http` or `https`)
* Хост (например. `example.com`)
* Порт (например. `80` or `443`)
* Путь (например. `/users/1`)
* Строка запроса (например. `sort=created&dir=asc`)

Вы можете получить объект [URI объекта][psr7_uri]  запроса PSR 7 с помощью его `getUri()` метода:

[psr7_uri]: http://www.php-fig.org/psr/psr-7/#3-5-psr-http-message-uriinterface

```php
$uri = $request->getUri();
```

URI объекта запроса PSR 7 сам по себе является объектом, который предоставляет следующие методы для проверки URL-адресов URL-адреса HTTP-запроса:

* `getScheme()`
* `getAuthority()`
* `getUserInfo()`
* `getHost()`
* `getPort()`
* `getPath()`
* `getBasePath()`
* `getQuery()` <small>(возвращает полную строку запроса, например `a=1&b=2`)</small>
* `getFragment()`
* `getBaseUrl()`

Вы можете получить параметры запроса как ассоциативный массив при использовании объекта Request `getQueryParams()`.

Вы также можете получить одно значение параметра запроса, с необязательным значением по умолчанию, если параметр отсутствует, используя `getQueryParam($key, $default = null)`.

<div class="alert alert-info">
    <div><strong>Базовый путь</strong></div>
    Если ваш front-controller вашего Slim-приложения находится в физическом подкаталоге под корневым каталогом вашего документа, вы можете получить физический базовый путь HTTP-запроса (относительно корня документа) с помощью <code>getBasePath()</code>
    метода объекта Uri . Это будет пустая строка, если приложение Slim установлено в самый верхний каталог корневого каталога документа.
</div>

<div id='the-request-headers'>
</div>

## Заголовки запросов

Каждый HTTP-запрос имеет заголовки. Это метаданные, которые описывают HTTP-запрос, но не отображаются в теле запроса. Объект запроса PSR 7 Slim предоставляет несколько методов для проверки своих заголовков.

### Получить все заголовки

Вы можете получить все заголовки HTTP-запросов в качестве ассоциативного массива с помощью `getHeaders()` метода объекта запроса PSR 7 . Результирующие ключи ассоциативного массива - это имена заголовков, и его значения сами представляют собой числовой массив строковых значений для их соответствующего заголовка.

```php
$headers = $request->getHeaders();
foreach ($headers as $name => $values) {
    echo $name . ": " . implode(", ", $values);
}
```
<figure>
<figcaption>Figure 5: Извлечение и повторение всех заголовков HTTP-запросов в качестве ассоциативного массива.</figcaption>
</figure>

### Получить один заголовок

Вы можете получить значения одного заголовка с помощью `getHeader($name)` метода объекта PSR 7 Request. Это возвращает массив значений для данного заголовка. Помните, _что один HTTP-заголовок может иметь более одного значения!_

```php
$headerValueArray = $request->getHeader('Accept');
```
<figure>
<figcaption>Figure 6: Получить значения для определенного HTTP-заголовка.</figcaption>
</figure>

Вы также можете получить строку, разделенную запятыми, со всеми значениями для данного заголовка с помощью `getHeaderLine($name)` метода объекта запроса PSR 7 . В отличие от `getHeader($name)` метода, этот метод возвращает строку, разделенную запятыми.

```php
$headerValueString = $request->getHeaderLine('Accept');
```
<figure>
<figcaption>Figure 7: Get single header's values as comma-separated string.</figcaption>
</figure>

### Обнаружение заголовка

Вы можете проверить наличие заголовка с помощью `hasHeader($name)`метода объекта PSR 7 Request .

```php
if ($request->hasHeader('Accept')) {
    // Do something
}
```
<figure>
<figcaption>Figure 8: Обнаружение присутствия определенного заголовка HTTP-запроса.</figcaption>
</figure>

<div id='the-request-body'>
</div>

## Тело запроса

Каждый запрос HTTP имеет тело. Если вы создаете приложение Slim, которое использует данные JSON или XML, вы можете использовать
`getParsedBody()` метод объекта PSR 7 Request для анализа тела запроса HTTP в собственный PHP-формат. Slim может анализировать данные JSON, XML и URL-кодированные данные.

```php 
$parsedBody = $request->getParsedBody();
```
<figure>
<figcaption>Figure 9: Тело запроса HTTP-анализа в собственный PHP-формат</figcaption>
</figure>

* Запросы JSON преобразуются в ассоциативные массивы с помощью `json_decode($input, true)`.
* XML-запросы преобразуются в  `SimpleXMLElement` с `simplexml_load_string($input)`.
* Запросы с URL-кодированием преобразуются в массив PHP с `parse_str($input)`.

Для URL-кодированных запросов вы также можете получить одно значение параметра, с необязательным значением по умолчанию, если параметр отсутствует, используя `getParsedBodyParam($key, $default = null)`.

Технически говоря, объект запроса PSR 7 Slim представляет собой тело запроса HTTP как экземпляр `\Psr\Http\Message\StreamInterface`. Вы можете получить `StreamInterface` экземпляр тела запроса HTTP с помощью `getBody()` метода объекта запроса PSR 7. `getBody()` Способ является предпочтительным , если размер входящего запроса HTTP , неизвестен или слишком большой для доступной памяти.

```php
$body = $request->getBody();
```
<figure>
<figcaption>Figure 10: Получить тело запроса HTTP</figcaption>
</figure>

Получаемый `\Psr\Http\Message\StreamInterface` экземпляр предоставляет следующие методы для чтения и повторения его базового PHP `resource`.

* `getSize()`
* `tell()`
* `eof()`
* `isSeekable()`
* `seek()`
* `rewind()`
* `isWritable()`
* `write($string)`
* `isReadable()`
* `read($length)`
* `getContents()`
* `getMetadata($key = null)`

### Переопределение тела

При вызове `getParsedBody` объекта Request несколько раз тело обрабатывается только один раз, даже если тело запроса изменяется в то же время.

Чтобы обеспечить репарацию тела, `reparseBody` можно использовать метод объекта Request.

<div id='uploaded-files'>
</div>

## Загруженные файлы

Загрузка файлов `$_FILES` доступна из `getUploadedFiles()` метода объекта Request . Это возвращает массив с именем `<input>` элемента.

```php
$files = $request->getUploadedFiles();
```
<figure>
<figcaption>Figure 11: Загрузка файлов</figcaption>
</figure>

Каждый объект в `$files` массиве является экземпляром
`\Psr\Http\Message\UploadedFileInterface` и поддерживает следующие методы:

* `getStream()`
* `moveTo($targetPath)`
* `getSize()`
* `getError()`
* `getClientFilename()`
* `getClientMediaType()`

См. [cookbook](/cookbook/uploading-files) о том, как загружать файлы с помощью формы POST.

<div id='request-helpers'>
</div>

## Запрос помощников - Request Helpers

Реализация Slim's PSR 7 Request предоставляет эти дополнительные проприетарные методы, которые помогут вам дополнительно проверить HTTP-запрос.

### Обнаружение XHR запроса

Вы можете обнаружить запросы XHR с помощью `isXhr()` метода объекта Request. Этот метод обнаруживает наличие `X-Requested-With` заголовка HTTP-запроса и обеспечивает его значение `XMLHttpRequest`.

```php
POST /path HTTP/1.1
Host: example.com
Content-type: application/x-www-form-urlencoded
Content-length: 7
X-Requested-With: XMLHttpRequest

foo=bar
```
<figure>
<figcaption>Figure 13: Пример запроса XHR.</figcaption>
</figure>

```php
if ($request->isXhr()) {
    // Do something
}
```

### Тип содержимого

Вы можете получить тип содержимого HTTP-запроса с помощью `getContentType()`метода объекта Request . Это возвращает `Content-Type` полное значение заголовка, предоставленное клиентом HTTP.

```php
$contentType = $request->getContentType();
```

### Тип Media

Возможно, вам не нужен полный `Content-Type` заголовок. Что, если вместо этого вам нужен только тип мультимедиа? Вы можете получить тип медиафайла HTTP-запроса с помощью `getMediaType()` метода объекта Request .

```php
$mediaType = $request->getMediaType();
```

Вы можете получить добавленные параметры типа носителя в качестве ассоциативного массива с помощью `getMediaTypeParams()` метода объекта Request.

```php
$mediaParams = $request->getMediaTypeParams();
```

### Набор символов

Одним из наиболее распространенных параметров типа мультимедиа является набор символов HTTP-запроса. Объект Request предоставляет выделенный метод для получения этого параметра типа носителя.

```php
$charset = $request->getContentCharset();
```

### Длина содержимого

Вы можете получить длину содержимого запроса HTTP с помощью `getContentLength()` метода объекта Request.

```php
$length = $request->getContentLength();
```

### Параметр запроса

Для извлечения одного значения параметра запроса, использовать методы: `getParam()`, `getQueryParam()`, `getParsedBodyParam()`, `getCookieParam()`, `getServerParam()`, аналоги формы множественного числа PSR-7 get*Params() методы.

Например, чтобы получить один параметр сервера:

```php
$foo = $request->getServerParam('HTTP_NOT_EXIST', 'default_value_here');
```

<div id='route-object'>
</div>

## Объект роутера

Иногда в промежуточном программном обеспечении вам требуется параметр вашего маршрута.

В этом примере мы сначала проверяем, что пользователь вошел в систему, а во-вторых, у пользователя есть разрешения на просмотр конкретного видео, которое они пытаются просмотреть.

```php
    $app->get('/course/{id}', Video::class.":watch")->add(Permission::class)->add(Auth::class);

    //.. In the Permission Class's Invoke
    /** @var $route \Slim\Route */
    $route = $request->getAttribute('route');
    $courseId = $route->getArgument('id');
```

<div id='media-type-parsers'>
</div>

## Анализаторы типов мультимедиа

Slim выглядит как тип медиа-запроса и, если он его распознает, будет анализировать его на структурированные данные, доступные через `$request->getParsedBody()`. Обычно это массив, но является объектом для типов media XML.

Следующие типы носителей распознаются и анализируются:

* application/x-www-form-urlencoded
* application/json
* application/xml & text/xml

Если вы хотите, чтобы Slim анализировал контент с другого media type, вам нужно либо самостоятельно разобрать исходный media, либо зарегистрировать новый парсер. Парсеры media - это просто вызывающие элементы, которые принимают `$input` строку и возвращают анализируемый объект или массив.

Зарегистрируйте новый медиасервер в промежуточном программном приложении или маршруте. Обратите внимание, что вы должны зарегистрировать парсер, прежде чем пытаться получить доступ к анализируемому телу в первый раз.

Например, для автоматического разбора JSON, который отправляется с `text/javascript` типом контента, вы регистрируете парсер типа медиа в промежуточном программном обеспечении следующим образом:

```php
// Add the middleware
$app->add(function ($request, $response, $next) {
    // add media parser
    $request->registerMediaTypeParser(
        "text/javascript",
        function ($input) {
            return json_decode($input, true);
        }
    );

    return $next($request, $response);
});
```

## Атрибуты

С помощью PSR-7 можно добавлять объекты / значения в объект запроса для дальнейшей обработки. В ваших приложениях промежуточное программное обеспечение часто должно передавать информацию до вашего закрытия маршрута и способ сделать это заключается в том, чтобы добавить его в объект запроса через атрибут.

Пример. Установка значения для объекта запроса.

```php
$app->add(function ($request, $response, $next) {
    $request = $request->withAttribute('session', $_SESSION); //add the session storage to your request as [READ-ONLY]
    return $next($request, $response);
});
```


Пример, как получить значение.

```php
$app->get('/test', function ($request, $response, $args) {
    $session = $request->getAttribute('session'); //get the session from the request

    return $response->write('Yay, ' . $session['name']);
});
```

Объект запроса также имеет объемные функции. `$request->getAttributes()` и `$request->withAttributes()`
