---
title: Middleware
---

Вы можете запускать код _до_ и _после_ вашего Slim-приложения, чтобы управлять объектами Request and Response по своему 
усмотрению. Это называется _middleware_ (промежуточным программным обеспечением). Зачем вам это делать? Возможно, вы хотите защитить 
свое приложение от подделки межсайтовых запросов. Возможно, вы хотите аутентифицировать запросы перед запуском приложения. 
Middleware идеально подходит для этих сценариев.


## Что такое middleware?

Технически говоря, middleware является _вызываемым_ которое принимает три аргумента:

1. `\Psr\Http\Message\ServerRequestInterface` - Объект запроса PSR7
2. `\Psr\Http\Message\ResponseInterface` - Объект ответа PSR7
3. `callable` - Следующее middleware (промежуточное программное обеспечение), подлежащее вызову

Он может делать все, что подходит этим объектам. Единственное жесткое требование заключается в том, что промежуточное 
ПО **ДОЛЖНО** возвращать экземпляр `\Psr\Http\Message\ResponseInterface`.
Каждое промежуточное ПО **СЛЕДУЕТ** вызывать следующее промежуточное программное обеспечение и передавать его 
объектам Request and Response в качестве аргументов.

## Как работает промежуточное программное обеспечение (middleware)?

Различные структуры используют промежуточное ПО по-разному. Slim добавляет промежуточное ПО как концентрические слои, 
окружающие ваше основное приложение. Каждый новый слой промежуточного программного обеспечения окружает любые 
существующие уровни промежуточного программного обеспечения. Концентрическая структура расширяется наружу, когда 
добавляются дополнительные слои промежуточного слоя.

Последний добавленный слой промежуточного программного обеспечения является первым, который будет выполнен.

Когда вы запускаете приложение Slim, объекты Request and Response пересекают структуру промежуточного программного 
обеспечения извне. Сначала они вводят самое внешнее промежуточное ПО, а затем следующее внешнее самое промежуточное ПО 
(и т. Д.), пока они в конечном итоге не прибудут к самому Slim-приложению. После того, как приложение Slim отправляет 
соответствующий маршрут, результирующий объект Response выходит из приложения Slim и пересекает структуру 
промежуточного программного обеспечения изнутри. В конечном итоге конечный объект Response выходит из самого 
промежуточного ПО, сериализуется в исходный HTTP-ответ и возвращается клиенту HTTP. Вот диаграмма, иллюстрирующая 
поток процесса промежуточного программного обеспечения:

<div style="padding: 2em 0; text-align: center">
    <img src="/md/images/middleware.png" alt="Middleware architecture" style="max-width: 80%;"/>
</div>

## Как написать промежуточное программное обеспечение?

Middleware является вызываемым, которое принимает три аргумента: объект Request, объект Response и следующее middleware. 
Каждое middleware **ДОЛЖНО** возвращать экземпляр  `\Psr\Http\Message\ResponseInterface`.

### Пример замыкание middleware.

Пример middleware замыкания.

```php
<?php
/**
 * Example middleware closure
 *
 * @param  \Psr\Http\Message\ServerRequestInterface $request  PSR7 request
 * @param  \Psr\Http\Message\ResponseInterface      $response PSR7 response
 * @param  callable                                 $next     Next middleware
 *
 * @return \Psr\Http\Message\ResponseInterface
 */
function ($request, $response, $next) {
    $response->getBody()->write('BEFORE');
    $response = $next($request, $response);
    $response->getBody()->write('AFTER');

    return $response;
};
```

### Пример вызываемого класса middleware

Это пример middleware вызываемого класс о который реализует магический метод `__invoke()`.

```php
<?php
class ExampleMiddleware
{
    /**
     * Example middleware invokable class
     *
     * @param  \Psr\Http\Message\ServerRequestInterface $request  PSR7 request
     * @param  \Psr\Http\Message\ResponseInterface      $response PSR7 response
     * @param  callable                                 $next     Next middleware
     *
     * @return \Psr\Http\Message\ResponseInterface
     */
    public function __invoke($request, $response, $next)
    {
        $response->getBody()->write('BEFORE');
        $response = $next($request, $response);
        $response->getBody()->write('AFTER');

        return $response;
    }
}
```

Чтобы использовать этот класс в качестве middleware, вы можете использовать `->add( new ExampleMiddleware() );` 
цепочку функций после `$app`, `Route`,  или `group()`, что в приведенном ниже коде, любой из них, может 
представлять объект $ subject.

```php
$subject->add( new ExampleMiddleware() );
```

## Как добавить middleware?

Вы можете добавить промежуточное программное обеспечение в Slim-приложение, на индивидуальный маршрут Slim или в 
группу маршрутов. Все сценарии принимают одно и то же middleware и реализуют один и тот же интерфейс 
middleware.

### Application middleware

Для каждого *входящего* HTTP-запроса вызывается middleware. Добавьте промежуточное программное обеспечение 
приложения с помощью  `add()` метода экземпляра Slim. В этом примере добавляется пример Closure middleware:

```php
<?php
$app = new \Slim\App();

$app->add(function ($request, $response, $next) {
	$response->getBody()->write('BEFORE');
	$response = $next($request, $response);
	$response->getBody()->write('AFTER');

	return $response;
});

$app->get('/', function ($request, $response, $args) {
	$response->getBody()->write(' Hello ');

	return $response;
});

$app->run();
```

Это вывело бы это тело ответа HTTP:

    BEFORE Hello AFTER

### Route middleware

Route middleware вызывается _только в том случае, если_ его маршрут соответствует текущему методу HTTP-запроса и URI.
Route middleware указывается сразу же после вызова любого из методов маршрутизации Slim-приложения 
(например `get()` или `post()`). Каждый метод маршрутизации возвращает экземпляр `\Slim\Route`, 
и этот класс предоставляет тот же интерфейс middleware что и экземпляр приложения Slim. Добавьте middleware в 
маршрут с помощью `add()` метода экземпляра Route. В этом примере добавляется пример Closure middleware:

```php
<?php
$app = new \Slim\App();

$mw = function ($request, $response, $next) {
    $response->getBody()->write('BEFORE');
    $response = $next($request, $response);
    $response->getBody()->write('AFTER');

    return $response;
};

$app->get('/', function ($request, $response, $args) {
	$response->getBody()->write(' Hello ');

	return $response;
})->add($mw);

$app->run();
```

Это вывело бы это тело ответа HTTP:

    BEFORE Hello AFTER

### Групповое Middleware

В дополнение к общему приложению и стандартным маршрутам, способным принимать middleware, `group()` 
функция определения нескольких маршрутов, также позволяет индивидуальные маршруты внутри. 
Маршрутное групповое  middleware вызывается _только в том случае, если_ его маршрут соответствует одному из 
определенных методов HTTP-запроса и URI из группы. Чтобы добавить промежуточное программное обеспечение в обратном 
вызове и промежуточное ПО всей группы, можно установить путем цепочки `add()` после `group()` метода.

Пример приложения, используя callback middleware в группе обработчиков URL-адресов
```php
<?php

require_once __DIR__.'/vendor/autoload.php';

$app = new \Slim\App();

$app->get('/', function ($request, $response) {
    return $response->getBody()->write('Hello World');
});

$app->group('/utils', function () use ($app) {
    $app->get('/date', function ($request, $response) {
        return $response->getBody()->write(date('Y-m-d H:i:s'));
    });
    $app->get('/time', function ($request, $response) {
        return $response->getBody()->write(time());
    });
})->add(function ($request, $response, $next) {
    $response->getBody()->write('It is now ');
    $response = $next($request, $response);
    $response->getBody()->write('. Enjoy!');

    return $response;
});
```

При вызове  `/utils/date` метода будет выводиться строка, подобная приведенной ниже

    It is now 2015-07-06 03:11:01. Enjoy!

посещение `/utils/time` будет выводить строку, подобную приведенной ниже

    It is now 1436148762. Enjoy!

но посещение `/` *(domain-root)*, должно было бы генерировать следующий результат, поскольку не было назначено  middleware 

    Hello World

### Передача переменных из middleware

Самый простой способ передать атрибуты из промежуточного программного обеспечения - использовать атрибуты запроса.

Установка переменной в middleware:

```php
$request = $request->withAttribute('foo', 'bar');
```

Получение переменной в обратном вызове маршрута:

```php
$foo = $request->getAttribute('foo');
```
