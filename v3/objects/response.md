---
title: Ответ - Response
---

Вашим маршрутам Slone и промежуточному программному обеспечению предоставляется объект ответа PSR 7, 
который представляет текущий HTTP-ответ, который должен быть возвращен клиенту. Объект ответа реализует 
[PSR 7 ResponseInterface][psr7], с помощью которого вы можете проверять и обрабатывать статус ответа HTTP, заголовки и тело.

[psr7]: http://www.php-fig.org/psr/psr-7/#3-2-1-psr-http-message-responseinterface

## Как получить объект Response

Объект ответа PSR 7 вводится в ваши маршруты Slim-приложений в качестве второго аргумента для обратного вызова 
маршрута следующим образом:

```php
<?php
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\ResponseInterface;

$app = new \Slim\App;
$app->get('/foo', function (ServerRequestInterface $request, ResponseInterface $response) {
    // Use the PSR 7 $response object

    return $response;
});
$app->run();
```
<figure>
<figcaption>Figure 1:  Ввести ответ PSR 7 в обратный вызов маршрута приложения.</figcaption>
</figure>

Объект ответа PSR 7 вводится в ваше _middleware_
Slim-приложения в качестве второго аргумента middleware, вызываемого следующим образом:

```php
<?php
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\ResponseInterface;

$app = new \Slim\App;
$app->add(function (ServerRequestInterface $request, ResponseInterface $response, callable $next) {
    // Use the PSR 7 $response object

    return $next($request, $response);
});
// Define app routes...
$app->run();
```
<figure>
<figcaption>Figure 2: Ввести ответ PSR 7 в прикладное промежуточное ПО.</figcaption>
</figure>

<div id="the-response-status"></div>

## Статус ответа

Каждый ответ HTTP имеет числовой [status code][statuscodes]. Код состояния определяет _тип_ ответа HTTP, 
который должен быть возвращен клиенту. Код состояния объекта PSR 7 по умолчанию `200(OK)`. 
Вы можете получить код состояния объекта PSR 7 с помощью `getStatusCode()` метода, подобного этому.

[statuscodes]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html

```php
$status = $response->getStatusCode();
```
<figure>
<figcaption>Figure 3: Получить код состояния ответа.</figcaption>
</figure>

Вы можете скопировать объект ответа PSR 7 и присвоить новый код состояния следующим образом:

```php
$newResponse = $response->withStatus(302);
```
<figure>
<figcaption>Figure 4: Создать ответ с новым кодом состояния.</figcaption>
</figure>

<div id="the-response-headers"></div>

## Заголовки ответов

У каждого HTTP-ответа есть заголовки. Это метаданные, которые описывают HTTP-ответ, 
но не отображаются в теле ответа. Объект Slim's PSR 7 Response предоставляет несколько методов 
для проверки и управления его заголовками.

### Получить все заголовки

Вы можете получить все заголовки HTTP-ответа в качестве ассоциативного массива с помощью `getHeaders()` метода объекта 
PSR 7 Response . Результирующие ключи ассоциативного массива - это имена заголовков, и его значения 
сами представляют собой числовой массив строковых значений для их соответствующего заголовка.

```php
$headers = $response->getHeaders();
foreach ($headers as $name => $values) {
    echo $name . ": " . implode(", ", $values);
}
```
<figure>
<figcaption>Figure 5: Извлечение и повторение всех заголовков HTTP-ответов в качестве ассоциативного массива.</figcaption>
</figure>

### Получить один заголовок

Вы можете получить значения (ов) одного заголовка с помощью `getHeader($name)` метода объекта PSR 7 Response. 
Это возвращает массив значений для данного заголовка. Помните, что _один HTTP-заголовок может иметь более одного 
значения!_

```php
$headerValueArray = $response->getHeader('Vary');
```
<figure>
<figcaption>Figure 6: Получить значения для определенного HTTP-заголовка.</figcaption>
</figure>


Вы также можете получить строку с разделителями-запятыми со всеми значениями для данного заголовка 
с помощью `getHeaderLine($name)` метода объекта PSR 7 Response . В отличие от `getHeader($name)` метода, 
этот метод возвращает строку, разделенную запятыми.

```php
$headerValueString = $response->getHeaderLine('Vary');
```
<figure>
<figcaption>Figure 7:Получить значения одного заголовка в виде строки, разделенной запятыми.</figcaption>
</figure>

### Обнаружение заголовка

Вы можете проверить наличие заголовка с помощью `hasHeader($name)` метода объекта PSR 7 Response .

```php
if ($response->hasHeader('Vary')) {
    // Do something
}
```
<figure>
<figcaption>Figure 8: Обнаружение присутствия определенного HTTP-заголовка.</figcaption>
</figure>

### Установить заголовок

Вы можете установить значение заголовка с помощью `withHeader($name, $value)` метода объекта PSR 7 Response.

```php
$newResponse = $oldResponse->withHeader('Content-type', 'application/json');
```
<figure>
<figcaption>Figure 9:  Настройка заголовка HTTP</figcaption>
</figure>

<div class="alert alert-info">
    <div><strong>напоминание</strong></div>
    <div>
        Объект Response неизменен. Этот метод возвращает <em>копию</em> объекта Response с новым значением 
        заголовка. <strong>Этот метод является разрушительным</strong> и <em>заменяет</em> существующие значения заголовков, 
        уже связанные с тем же заголовком.
    </div>
</div>

### Добавить заголовок

Вы можете добавить значение заголовка с помощью `withAddedHeader($name, $value)` метода объекта PSR 7 Response 

```php
$newResponse = $oldResponse->withAddedHeader('Allow', 'PUT');
```
<figure>
<figcaption>Figure 10: Добавление заголовка HTTP</figcaption>
</figure>

<div class="alert alert-info">
    <div><strong>напоминание</strong></div>
    <div>
        В отличие от <code>withHeader()</code> метода, этот метод <em>добавляет</em> новое значение к набору 
        значений, которые уже существуют для одного и того же заголовка. Объект Response 
        неизменен. Этот метод возвращает <em>копию</em> объекта Response с добавленным значением заголовка.
    </div>
</div>

### Удалить заголовок

Вы можете удалить заголовок с помощью `withoutHeader($name)` метода объекта Response.

```php
$newResponse = $oldResponse->withoutHeader('Allow');
```
<figure>
<figcaption>Figure 11: Удалите HTTP-заголовок</figcaption>
</figure>

<div class="alert alert-info">
    <div><strong>напоминание</strong></div>
    <div>
        Объект Response неизменен. Этот метод возвращает <em>копию</em> объекта Response с 
        добавленным значением заголовка.
    </div>
</div>

<div id="the-response-body"></div>

## Тело ответа

Ответ HTTP обычно имеет тело. Slim предоставляет объект PSR 7 Response, с 
помощью которого вы можете проверять и обрабатывать тело ответа в конечном итоге.

Так же, как объект запроса PSR 7, объект ответа PSR 7 реализует тело как экземпляр 
`\Psr\Http\Message\StreamInterface`. Вы можете получить `StreamInterface` экземпляр 
тела ответа HTTP с помощью `getBody()` метода объекта PSR 7 Response. `getBody()` 
Способ является предпочтительным , если исходящей длина ответа HTTP , неизвестна 
или слишком большая для доступной памяти.

```php
$body = $response->getBody();
```
<figure>
<figcaption>Figure 12: Получить тело ответа HTTP</figcaption>
</figure>

Получаемый `\Psr\Http\Message\StreamInterface` экземпляр предоставляет следующие методы 
для чтения, итерации и записи в базовый PHP `resource`.

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

Чаще всего вам нужно будет записать объект PSR 7 Response. Вы можете записать 
содержимое в `StreamInterface` экземпляр с помощью его `write()` метода следующим образом:

```php
$body = $response->getBody();
$body->write('Hello');
```
<figure>
<figcaption>Figure 13:  Запись содержимого в тело ответа HTTP</figcaption>
</figure>

Вы также можете _заменить_ тело объекта PSR 7 Response совершенно новым `StreamInterface` экземпляром. 
Это особенно полезно, когда вы хотите передать контент из удаленного адресата (например, файловой 
системы или удаленного API) в ответ HTTP. Вы можете заменить тело объекта PSR 7 Response своим 
`withBody(StreamInterface $body)` методом. Его аргумент **ДОЛЖЕН** быть примером `\Psr\Http\Message\StreamInterface`.

```php
$newStream = new \GuzzleHttp\Psr7\LazyOpenStream('/path/to/file', 'r');
$newResponse = $oldResponse->withBody($newStream);
```
<figure>
    <figcaption>Figure 14: Замените тело ответа HTTP</figcaption>
</figure>

<div class="alert alert-info">
    <div><strong>напоминание</strong></div>
    <div>
        Объект Response неизменен. Этот метод возвращает <em>копию</em> объекта Response, который содержит новый элемент.
    </div>
</div>


<div id="returning-json"></div>

## Возвращение JSON

У объекта Slim Response есть собственный метод, `withJson($data, $status, $encodingOptions)` 
помогающий упростить процесс возвращения данных JSON.

Параметр `$data` содержит структуру данных, которые вы хотите возвратить как JSON. `$status` является 
необязательным и может использоваться для возврата пользовательского HTTP-кода. `$encodingOptions` 
является необязательным, и для него используются одни и те же параметры кодирования [`json_encode()`][json_encode].

В простейшей форме данные JSON могут быть возвращены с кодом состояния HTTP по умолчанию 200.

```php
$data = array('name' => 'Bob', 'age' => 40);
$newResponse = $oldResponse->withJson($data);
```
<figure>
<figcaption>Figure 15:  Возврат JSON с кодом состояния HTTP HTTP.</figcaption>
</figure>

Мы также можем вернуть данные JSON с пользовательским кодом состояния HTTP.

```php
$data = array('name' => 'Rob', 'age' => 40);
$newResponse = $oldResponse->withJson($data, 201);
```
<figure>
<figcaption>Figure 16: Возврат JSON с кодом статуса HTTP HTTP.</figcaption>
</figure>

`Content-Type` В ответ автоматически устанавливается `application/json;charset=utf-8`.

Если есть проблема с кодировкой данных в JSON, a `\RuntimeException($message, $code)` выбрано значение, 
содержащее значения [`json_last_error_msg()`][json_last_error_msg] как `$message` и [`json_last_error()`][json_last_error]
как `$code`.

<div class="alert alert-info">
    <div><strong>напоминание</strong></div>
    <div>
        Объект Response неизменен. Этот метод возвращает <em>копию </em> объекта Response с новым 
        заголовком Content-Type. <strong>Этот метод является разрушительным</strong>, и он <em>заменяет</em> 
        существующий заголовок Content-Type. Статус также заменяется, если статус $ был 
        передан при <code>withJson()</code> вызове.
    </div>
</div>

[json_encode]: http://php.net/manual/en/function.json-encode.php
[json_last_error]: http://php.net/manual/en/function.json-last-error.php
[json_last_error_msg]: http://php.net/manual/en/function.json-last-error-msg.php

## Возврат перенаправления
Объект ответа Slim имеет настраиваемый метод, `withRedirect($url, $status = null)` 
когда вы хотите вернуть перенаправление на другой URL. Вы указываете, `$url` куда 
вы хотите, чтобы клиент был перенаправлен вместе с дополнительным `$status` кодом.

```php
return $response->withRedirect('/new-url', 301);
```
<figure>
<figcaption>Figure 17: Возврат перенаправления с необязательным кодом состояния.</figcaption>
</figure>