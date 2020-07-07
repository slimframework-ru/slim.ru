---
title: PSR 7
---

Slim поддерживает интерфейс [PSR-7](https://github.com/php-fig/http-message) для объектов запроса и ответа.
Это делает Slim гибче, потому что он может использовать _любую_ реализацию PSR-7.
Например, вы можете вернуть экземпляр `GuzzleHttp\Psr7\CachingStream` или любой другой объект, 
возвращаемый функцией `GuzzleHttp\Psr7\stream_for()`.

Slim предоставляет собственную реализацию PSR-7, поэтому он работает "из коробки". Тем не менее, 
вы можете заменить стандартные объекты PSR-7 сторонней реализацией.
Просто переопределите сервисы `request` и `response` в контейнере приложения, чтобы они возвращали 
экземпляр `Psr\Http\Message\ServerRequestInterface` и `Psr\Http\Message\ResponseInterface` соответственно.

## Объекты-значения

Объекты `Request` и `Response` являются [_неизменяемыми объектами-значениями_](http://en.wikipedia.org/wiki/Value_object).
Их можно "изменить", только клонировав версию с обновленными значениями свойств.
Эти объекты имеют номинальные издержки, но эти издержки никак не влияют на производительность.

Вы можете запросить копию объекта-значения, вызвав любой из его методов интерфейса PSR-7 
(обычно эти методы имеют префикс `with`). Например, у объекта Response есть метод `withHeader($name, $value)`, 
который возвращает клонированный объект значения с новым заголовком HTTP.

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->get('/foo', function (Request $request, Response $response, array $args) {
    $payload = json_encode(['hello' => 'world'], JSON_PRETTY_PRINT);
    $response->getBody()->write($payload);
    return $response->withHeader('Content-Type', 'application/json');
});

$app->run();
```

Интерфейс PSR-7 предоставляет следующие методы для преобразования объектов запроса и ответа:

* `withProtocolVersion($version)`
* `withHeader($name, $value)`
* `withAddedHeader($name, $value)`
* `withoutHeader($name)`
* `withBody(StreamInterface $body)`

Помимо общих, интерфейс запроса имеет следующие методы:

* `withMethod($method)`
* `withUri(UriInterface $uri, $preserveHost = false)`
* `withCookieParams(array $cookies)`
* `withQueryParams(array $query)`
* `withUploadedFiles(array $uploadedFiles)`
* `withParsedBody($data)`
* `withAttribute($name, $value)`
* `withoutAttribute($name)`

Интерфейс ответа, помимо общих, имеет методы:

* `withStatus($code, $reasonPhrase = '')`

Обратитесь к [документации PSR-7] (http://www.php-fig.org/psr/psr-7/) для получения дополнительной информации об этих методах.
