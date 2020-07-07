---
title: Middleware (промежуточное ПО)
---

Вы можете запустить код _перед_ и _после_ вашего обработчика в приложении Slim, чтобы манипулировать 
объектами Request и Response по своему усмотрению. Это называется _промежуточное ПО_.
Зачем это делать? Возможно, вы хотите защитить свое приложение от подделки межсайтовых запросов.
Возможно, вы хотите аутентифицировать запросы до запуска вашего приложения. 
Промежуточное программное обеспечение идеально подходит для этих сценариев.

## Что такое промежуточное ПО?

Промежуточное ПО реализует [интерфейс PSR-15](https://www.php-fig.org/psr/psr-15/):

1. `Psr\Http\Message\ServerRequestInterface` - объект PSR-7 запроса
2. `Psr\Http\Server\RequestHandlerInterface` - объект PSR-15 обработчика запроса

Промежуточное ПО может делать что угодно с этими объектами. Единственным жестким требованием является то, 
что промежуточное ПО **ДОЛЖНО** возвращать экземпляр `Psr\Http\Message\ResponseInterface`.
Каждое промежуточное ПО вызовет следующее промежуточное ПО, передаст ему объект Request и получит от него объект Response.

## Как работает промежуточное ПО?

Различные фреймворки используют промежуточное ПО по-разному. Slim добавляет промежуточное ПО в виде концентрических 
слоев, окружающих ваше основное приложение. Каждый новый слой промежуточного ПО окружает любые существующие слои 
промежуточного ПО. Концентрическая структура расширяется наружу по мере добавления дополнительных слоев 
промежуточного ПО.

Последний добавленный слой промежуточного ПО выполняется первым.

Когда вы запускаете приложение Slim, объект Request пересекает структуру промежуточного программного обеспечения извне.
Сначала они вводят самое внешнее промежуточное ПО, затем следующее внешнее промежуточное ПО (и т.д.), Пока в конечном 
итоге не достигают обработчика приложения Slim. После того как приложение Slim обрабатывает соответствующий маршрут,
результирующий объект Response выходит из приложения Slim и пересекает структуру ПО в обратном порядке.
В завершение конечный объект Response выходит из самого внешнего промежуточного ПО, сериализуется в сырой HTTP-ответ
и возвращается HTTP-клиенту. Вот схема, которая иллюстрирует поток процесса промежуточного программного обеспечения:

![Архитектура промежуточного ПО](/images/middleware.png)

## Как писать промежуточное ПО?

Промежуточное ПО - это вызываемый объект, который принимает два аргумента: 
объект `Psr\Http\Message\ServerRequestInterface` и объект `Psr\Http\Server\RequestHandlerInterface`.
Каждое промежуточное ПО **ДОЛЖНО** возвращать экземпляр `Psr\Http\Message\ResponseInterface`.

### Пример промежуточного ПО как анонимная функция.

```php
<?php
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Server\RequestHandlerInterface as RequestHandler;
use Slim\Factory\AppFactory;
use Slim\Psr7\Response;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

/**
 * Пример промежуточного ПО как анонимная функция.
 *
 * @param  Request  $request PSR-7 запрос
 * @param  RequestHandler $handler PSR-15 обработчик запроса
 *
 * @return Response
 */
$beforeMiddleware = function (Request $request, RequestHandler $handler) {
    $response = $handler->handle($request);
    $existingContent = (string) $response->getBody();

    $response = new Response();
    $response->getBody()->write('BEFORE' . $existingContent);

    return $response;
};

$afterMiddleware = function ($request, $handler) {
    $response = $handler->handle($request);
    $response->getBody()->write('AFTER');
    return $response;
};

$app->add($beforeMiddleware);
$app->add($afterMiddleware);

// ...

$app->run();
```

### Пример промежуточного ПО как вызываемый класс.

Данный пример промежуточного ПО как класс, реализующий магический метод `__invoke()`.

```php
<?php
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Server\RequestHandlerInterface as RequestHandler;
use Slim\Psr7\Response;

class ExampleBeforeMiddleware
{
    /**
     * Пример промежуточного ПО как вызываемый класс.
     * Данное промежуточное ПО дописывает BEFORE в начало ответа
     *
     * @param  Request  $request PSR-7 запрос
     * @param  RequestHandler $handler PSR-15 обработчик запроса
     *
     * @return Response
     */
    public function __invoke(Request $request, RequestHandler $handler): Response
    {
        $response = $handler->handle($request);
        $existingContent = (string) $response->getBody();
    
        $response = new Response();
        $response->getBody()->write('BEFORE' . $existingContent);
    
        return $response;
    }
}
```

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Server\RequestHandlerInterface as RequestHandler;

class ExampleAfterMiddleware
{
    /**
     * Пример промежуточного ПО как вызываемый класс.
     * Данное промежуточное ПО дописывает AFTER в конец ответа
     *
     * @param  Request  $request PSR-7 запрос
     * @param  RequestHandler $handler PSR-15 обработчик запроса
     *
     * @return Response
     */
    public function __invoke(Request $request, RequestHandler $handler): Response
    {
        $response = $handler->handle($request);
        $response->getBody()->write('AFTER');
        return $response;
    }
}
```

Чтобы использовать эти классы в качестве промежуточного ПО, вы можете использовать цепочку вызовов метода **add(new ExampleMiddleware());** вашего объекта Slim или объектов Route, возвращаемых методами **get(), post(), put(), patch(), delete(), options(), any()** или **group()**, как в приведенном ниже коде.

```php
<?php
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

// Add Middleware On App
$app->add(new ExampleMiddleware());

// Add Middleware On Route
$app->get('/', function () { ... })->add(new ExampleMiddleware());

// Add Middleware On Group
$app->group('/', function () { ... })->add(new ExampleMiddleware());

// ...

$app->run();
```

## Как добавлять промежуточное ПО?

Вы можете добавить промежуточное ПО в приложение Slim, в отдельный маршрут приложения Slim или в группу маршрутов. Все сценарии принимают одно и то же промежуточное программное обеспечение и реализуют один и тот же интерфейс промежуточного программного обеспечения.

### Промежуточное ПО приложения

Промежуточное ПО приложения вызывается для каждого **входящего** HTTP-запроса не зависимо от маршрута. 
Добавьте промежуточное ПО приложения с помощью метода **add()** экземпляра приложения Slim. 
В этом примере добавлен пример промежуточного программного обеспечения `Closure`:

```php
<?php
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Server\RequestHandlerInterface as RequestHandler;
use Slim\Factory\AppFactory;
use Slim\Psr7\Response;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->add(function (Request $request, RequestHandler $handler) {
    $response = $handler->handle($request);
    $existingContent = (string) $response->getBody();

    $response = new Response();
    $response->getBody()->write('BEFORE ' . $existingContent);

    return $response;
});

$app->add(function (Request $request, RequestHandler $handler) {
    $response = $handler->handle($request);
    $response->getBody()->write(' AFTER');
    return $response;
});

$app->get('/', function (Request $request, Response $response, $args) {
    $response->getBody()->write('Hello World');
    return $response;
});

$app->run();
```

Это вывело бы это тело ответа HTTP:

```text
BEFORE Hello World AFTER
```

### Промежуточное ПО маршрута

Промежуточное ПО маршрута вызывается, _только если_ его маршрут соответствует текущему методу HTTP-запроса и URI. 
Промежуточное ПО для маршрутов указывается сразу после вызова любого из методов маршрутизации приложения Slim (
например, **get()** или **post()**). Каждый метод маршрутизации возвращает экземпляр `\Slim\Route`, 
и этот класс предоставляет тот же интерфейс промежуточного ПО, что и экземпляр приложения Slim.
Добавьте промежуточное ПО в маршрут с помощью метода **add()** экземпляра `Route`. В этом примере добавлен 
пример промежуточного программного обеспечения `Closure`:

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Server\RequestHandlerInterface as RequestHandler;
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$mw = function (Request $request, RequestHandler $handler) {
    $response = $handler->handle($request);
    $response->getBody()->write('World');

    return $response;
};

$app->get('/', function (Request $request, Response $response, $args) {
    $response->getBody()->write('Hello ');

    return $response;
})->add($mw);

$app->run();
```

Это вывело бы это тело ответа HTTP:

```text
Hello World
```

### Промежуточное ПО групп маршрутов

В дополнение ко всему, группы маршрутов, определяемые методом **group()** тоже позволяют использовать промежуточное ПО
для всех маршрутов группы. Для добавления промежуточного ПО группе, нужно вызвать метод **add()** объекта, возвращаемого методом **group()**.

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Server\RequestHandlerInterface as RequestHandler;
use Slim\Factory\AppFactory;
use Slim\Routing\RouteCollectorProxy;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->get('/', function (Request $request, Response $response) {
    $response->getBody()->write('Hello World');
    return $response;
});

$app->group('/utils', function (RouteCollectorProxy $group) {
    $group->get('/date', function (Request $request, Response $response) {
        $response->getBody()->write(date('Y-m-d H:i:s'));
        return $response;
    });
    
    $group->get('/time', function (Request $request, Response $response) {
        $response->getBody()->write((string)time());
        return $response;
    });
})->add(function (Request $request, RequestHandler $handler) use ($app) {
    $response = $handler->handle($request);
    $dateOrTime = (string) $response->getBody();

    $response = $app->getResponseFactory()->createResponse();
    $response->getBody()->write('It is now ' . $dateOrTime . '. Enjoy!');

    return $response;
});

$app->run();
```

При запросе **GET /utils/date** будет выведена примерно такая строка.

```text
It is now 2015-07-06 03:11:01. Enjoy!
```

Посещение **/utils/time** выведет что-то такое:

```text
It is now 1436148762. Enjoy!
```

Но посещение **/** *(корня домена)*, приведет к следующему выводу, так как промежуточное ПО не было назначено:

```text
Hello World
```

### Передача переменных из промежуточного ПО

Самый простой способ передать атрибуты из промежуточного программного обеспечения - это использовать атрибуты запроса.

Установка переменной в промежуточном ПО:

```php
$request = $request->withAttribute('foo', 'bar');
```

Получение переменной в обработчике:

```php
$foo = $request->getAttribute('foo');
```

## Поиск доступного промежуточного программного обеспечения

Возможно, вы уже нашли классы промежуточного ПО PSR-15, которые удовлетворяют ваши потребности.
Вот несколько неофициальных списков для поиска:

* [Промежуточное ПО для фреймворка Slim v4 (wiki проекта)](https://github.com/slimphp/Slim/wiki/Middleware-for-Slim-Framework-v4.x)
* [middlewares/awesome-psr15-middlewares](https://github.com/middlewares/awesome-psr15-middlewares)
