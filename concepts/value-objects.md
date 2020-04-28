---
title: PSR 7 и объекты Value
---

Slim поддерживает [PSR-7](https://github.com/php-fig/http-message) интерфейсы для объектов запроса и ответа. 
Это делает Slim гибким, поскольку он может использовать любую реализацию PSR-7. Например, для Slim-приложения не 
нужно возвращать экземпляр  `\Slim\Http\Response`. Например, он может вернуть экземпляр 
`\GuzzleHttp\Psr7\CachingStream` или любой экземпляр, возвращаемый `\GuzzleHttp\Psr7\stream_for()` функцией.

Slim обеспечивает собственную реализацию PSR-7, так что она работает из коробки. Тем не менее, вы можете заменить 
объекты PSR 7 по умолчанию Slim сторонней реализацией. Просто переопределите контейнер приложения 
`request` и `response` службы, чтобы они возвращали экземпляр `\Psr\Http\Message\ServerRequestInterface` и
`\Psr\Http\Message\ResponseInterface`, соответственно.

## Value objects

Объекты запроса и ответа Slim являются объектами [_неизменяемых значений _](http://en.wikipedia.org/wiki/Value_object).
Их можно «изменить» только путем запроса клонированной версии, которая обновила значения свойств. Объекты Value имеют 
номинальные накладные расходы, потому что они должны быть клонированы, когда их свойства обновляются. Эти накладные 
расходы не влияют на производительность каким-либо значимым образом.

Вы можете запросить копию объекта значения, вызвав любой из его методов интерфейса PSR 7 (эти методы обычно имеют 
`with` префикс). Например, объект ответа PSR 7 имеет `withHeader($name, $value)` метод, который возвращает 
объект клонированного значения с новым HTTP-заголовком.

```php
<?php
$app = new \Slim\App;
$app->get('/foo', function ($req, $res, $args) {
    return $res->withHeader(
        'Content-Type',
        'application/json'
    );
});
$app->run();
```

Интерфейс PSR 7 предоставляет эти методы для преобразования объектов запроса и ответа:

* `withProtocolVersion($version)`
* `withHeader($name, $value)`
* `withAddedHeader($name, $value)`
* `withoutHeader($name)`
* `withBody(StreamInterface $body)`

Интерфейс PSR 7 предоставляет эти методы для преобразования объектов Request (запросов):

* `withMethod($method)`
* `withUri(UriInterface $uri, $preserveHost = false)`
* `withCookieParams(array $cookies)`
* `withQueryParams(array $query)`
* `withUploadedFiles(array $uploadedFiles)`
* `withParsedBody($data)`
* `withAttribute($name, $value)`
* `withoutAttribute($name)`

Интерфейс PSR 7 предоставляет эти методы для преобразования объектов Response:

* `withStatus($code, $reasonPhrase = '')`

Для получения дополнительной информации об этих методах обратитесь к [PSR-7 documentation](http://www.php-fig.org/psr/psr-7/) 
