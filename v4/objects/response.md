---
title: Ответ
---

Маршруты и промежуточное ПО вашего приложения Slim получают объект ответа PSR-7, представляющий текущий HTTP-ответ, который должен быть возвращен клиенту. Объект ответа реализует [PSR-7 ResponseInterface](https://www.php-fig.org/psr/psr-7/#33-psrhttpmessageresponseinterface), с помощью которого вы можете проверять и управлять состоянием ответа HTTP, заголовками и телом.

## Как получить объект Response

Объект ответа PSR-7 внедряется в маршруты вашего приложения Slim в качестве второго аргумента для обратного вызова маршрута, например:

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->get('/hello', function (ServerRequest $request, Response $response) {
    $response->getBody()->write('Hello World');
    return $response;
});

$app->run();
```

<div id="the-response-status"></div>

## Статус ответа

Каждый HTTP-ответ имеет числовой [код состояния](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html). Код состояния идентифицирует _тип_ ответа HTTP, который будет возвращен клиенту. По умолчанию, код состояния объекта ответа PSR-7 - `200` (ОК). Вы можете получить код состояния объекта PSR-7 Response с помощью метода `getStatusCode()` следующим образом.

```php
$status = $response->getStatusCode();
```

Вы можете скопировать объект ответа PSR-7 и назначить новый код состояния, например:

```php
$newResponse = $response->withStatus(302);
```

<div id="the-response-headers"></div>

## Заголовки ответа

Каждый HTTP-ответ имеет заголовки. Это метаданные, которые описывают HTTP-ответ, но не видны в теле ответа. Объект Response PSR-7 предоставляет несколько методов для проверки и манипулирования своими заголовками.

### Получение всех заголовков

Вы можете извлечь все заголовки HTTP-ответа в виде ассоциативного массива с помощью метода `getHeaders()` объекта PSR-7 ответа.
Ключи этого массива будут названиями заголовков, а их значения - нумерованными массивами строк, содержащими значения заголовка.

```php
$headers = $response->getHeaders();
foreach ($headers as $name => $values) {
    echo $name . ": " . implode(", ", $values);
}
```

### Получение одного заголовка

Вы можете получить значения одного заголовка с помощью метода `getHeader($name)` объекта ответа PSR-7. 
Метод возвращает массив значений для данного имени заголовка. 
Помните, что _один заголовок HTTP может иметь более одного значения!_

```php
$headerValueArray = $response->getHeader('Vary');
```

Вы также можете получить разделенную запятыми строку со всеми значениями для данного заголовка с помощью метода `getHeaderLine($name)` объекта ответа PSR-7. В отличие от метода `getHeader($name)`, этот метод возвращает строку объединенных через запятую значений.

```php
$headerValueString = $response->getHeaderLine('Vary');
```

### Определение наличия заголовка

Вы можете проверить наличие заголовка с помощью метода `hasHeader($name)` объекта ответа PSR-7.

```php
if ($response->hasHeader('Vary')) {
    // Do something
}
```

### Установка заголовка

Вы можете задать значение заголовка объекту ответа с помощью метода `withHeader($name, $value)`.

```php
$newResponse = $oldResponse->withHeader('Content-type', 'application/json');
```

<div class="alert alert-info">
    <div><strong>Напоминание</strong></div>
    <div>Объект Response является неизменным. Этот метод возвращает <em>копию</em> объекта Response с новым значением заголовка. <strong>Этот метод является деструктивным</strong> и <em>заменяет</em> существующие значения заголовка, уже связанные с тем же именем заголовка.</div>
</div>

### Добавление значения заголовка

Вы можете добавить значение заголовка объекту PSR-7 ответа с помощью метода `withAddedHeader($name, $value)`.

```php
$newResponse = $oldResponse->withAddedHeader('Allow', 'PUT');
```

<div class="alert alert-info">
    <div><strong>Напоминание</strong></div>
    <div>В отличие от метода <code>withHeader()</code>, этот метод <em>добавляет</em> новое значение к набору значений, которые уже существуют для того же имени заголовка. Объект Response является неизменным. Этот метод возвращает <em>копию</em> объекта Response с добавленным значением заголовка.</div>
</div>

### Удаление заголовка

Вы можете удалить заголовк из объекта PSR-7 ответа с помощью метода `withoutHeader($name)`.

```php
$newResponse = $oldResponse->withoutHeader('Allow');
```

<div class="alert alert-info">
    <div><strong>Напоминание</strong></div>
    <div>Объект Response является неизменным. Этот метод возвращает <em>копию</em> объекта Response без удаленного заголовка.</div>
</div>

<div id="the-response-body"></div>

## Тело ответа

Обычно HTTP-ответ имеет тело.

Как и объект запроса PSR-7, объект ответа PSR-7 реализует тело как экземпляр `Psr\Http\Message\StreamInterface`. 
Вы можете получить экземпляр тела ответа HTTP `StreamInterface` с помощью метода `getBody()` объекта PSR-7 Response.
Метод `getBody()` предпочтителен, если длина исходящего HTTP-ответа неизвестна или слишком велика для доступной памяти.

```php
$body = $response->getBody();
```

Результирующий экземпляр `Psr\Http\Message\StreamInterface` предоставляет следующие методы для чтения и итерации своего базового PHP-ресурса:

* getSize()
* tell()
* eof()
* isSeekable()
* seek()
* rewind()
* isWritable()
* write($string)
* isReadable()
* read($length)
* getContents()
* getMetadata($key = null)

Чаще всего вам нужно писать в объект ответа PSR-7. Вы можете записать содержимое в экземпляр `StreamInterface` с помощью его метода `write()` следующим образом:

```php
$body = $response->getBody();
$body->write('Hello');
```

Вы также можете _заменить_ тело объекта PSR-7 Response совершенно новым экземпляром `StreamInterface`. 
Это особенно полезно, когда вы хотите передать содержимое из удаленного места назначения 
(например, файловой системы или удаленного API) в ответ HTTP. 
Вы можете заменить тело объекта PSR-7 Response его методом `withBody(StreamInterface $body)`. 
Его аргумент **ДОЛЖЕН** быть экземпляром `Psr\Http\Message\StreamInterface`.

```php
use GuzzleHttp\Psr7\LazyOpenStream;

$newStream = new LazyOpenStream('/path/to/file', 'r');
$newResponse = $oldResponse->withBody($newStream);
```

<div class="alert alert-info">
    <div><strong>Напоминание</strong></div>
    <div>Объект Response является неизменным. Этот метод возвращает <em>копию</em> объекта Response с новым телом.</div>
</div>

<div id="returning-json"></div>

## Возврат JSON

В простейшей форме данные JSON могут быть возвращены с кодом состояния HTTP по умолчанию 200.

```php
$data = array('name' => 'Bob', 'age' => 40);
$payload = json_encode($data);

$response->getBody()->write($payload);
return $response
          ->withHeader('Content-Type', 'application/json');
```

Так же мы можем возвращать JSON-данные с другими кодами состояния.

```php
$data = array('name' => 'Rob', 'age' => 40);
$payload = json_encode($data);

$response->getBody()->write($payload);
return $response
          ->withHeader('Content-Type', 'application/json')
          ->withStatus(201);
```

<div class="alert alert-info">
    <div><strong>Напоминание</strong></div>
    <div>Объект Response является неизменным. Этот метод возвращает <em>копию</em> объекта Response с новым заголовком Content-Type. <strong>Этот метод является деструктивным</strong> и <em>заменяет</em> существующие значения заголовка Content-Type header.</div>
</div>

<div id="returning-redirect"></div>

## Ответ с перенаправлением

Вы можете перенаправить клиента на другой ресурс, используя HTTP ответ с заголовком `Location`

```php
return $response
  ->withHeader('Location', 'https://www.example.com')
  ->withStatus(302);
```
