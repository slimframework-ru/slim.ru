---
title: Руководство по обновлению
---

Если Вы обновляете свое приложение с 3 версии на 4, вы должны быть в курсе следующих значительных изменений.

## Требование к версии PHP
Slim 4 требует **PHP 7.2 и выше**.

## Изменения, ломающие обратную совместимость в конструкторе Slim\App constructor

Настройки приложения хранились в контейнере. Теперь они вынесены отдельно.

```php
/**
 * Slim 3 App::__construct($container = [])
 * Раньше настройки передавались в конструкторе
 */
$app = new App([
    'settings' => [...],
]);

/**
 * Slim 4 App::__constructor() метод принимает 1 обязательный и 4 опциональных параметра
 * 
 * @param ResponseFactoryInterface Любая реализация ResponseFactory
 * @param ContainerInterface|null Любая реализация  Container
 * @param CallableResolverInterface|null Любая реализация CallableResolver
 * @param RouteCollectorInterface|null Любая реализация RouteCollector
 * @param RouteResolverInterface|null Любая реализация RouteResolver
 */
$app = new App(...);
```

## Удаленные настройки приложения

- `addContentLengthHeader` См [Middleware размера ответа](/v4/middleware/content-length) для новой реализации данной настройки.
- `determineRouteBeforeAppMiddleware` Добавьте [Middleware маршрутизации](/docs/v4/middleware/routing) в правильную позицию вашего стека middleware, чтобы воспроизвести нужное поведение.
- `outputBuffering` См [Middleware буферизации вывода](/v4/middleware/output-buffering) для новой реализации данной настройки.
- `displayErrorDetails` См [Middleware обработки ошибок](/v4/middleware/error-handling) для новой реализации данной настройки.

## Изменения конейнера

Slim больше не включает в комплект пакет контейнера, вам нужно самостоятельно выбрать и установить любимый контейнер. Также был удален магический метод `App::__call()`, поэтому доступ к свойству контейнера через `$app->key_name()` больше не работает.

```php
/**
 * Slim 3.x поставляется с контейнером Pimple и включает следующий синтаксис
 */
$container = $app->getContainer();

// сервисы назначаются как в массиве
$container['view'] = function (\Psr\Container\ContainerInterface $container){
    return new \Slim\Views\Twig('');
};


/**
 * Slim 4.x не включает библиотеку контейнера. Он поддерживает любую реализацию PSR-11, например PHP-DI
 * Для установки PHP-DI выполните команду `composer require php-di/php-di`
 */

use Slim\Factory\AppFactory;

$container = new \DI\Container();

AppFactory::setContainer($container);
$app = AppFactory::create();

$container = $app->getContainer();
$container->set('view', function(\Psr\Container\ContainerInterface $container){
    return new \Slim\Views\Twig('');
});
```

## Изменения в обработке базового пути

До версии 3, Slim извлекал базовый путь из директории, в которой был создан объект приложения. Теперь базовый путь должен быть вручную объявлен, [если приложение выполняется не из корня домена](/v4/start/web-servers#run-from-a-sub-directory):

```php
use Slim\Factory\AppFactory;
// ...
$app = AppFactory::create();
$app->setBasePath('/my-app-subpath');
// ...
$app->run();
```

## Изменения компонента маршрутизации

Компонент `Router` из Slim 3 разделен на несколько компонентов (`RouteCollector`, `RouteParser` и `RouteResolver`), 
чтобы отделить библиотеку FastRoute от ядра `App` и предоставить разработчикам большую гибкость.
Эти компоненты имеют свои интерфейсы, которые вы можете самостоятельно реализовать и добавить в конструктор  `App`.
Следующие пул-реквесты предоставят достаточно информации об интерфейсах новых компонентов:

- [Pull Request #2604](https://github.com/slimphp/Slim/pull/2604)
- [Pull Request #2622](https://github.com/slimphp/Slim/pull/2622)
- [Pull Request #2639](https://github.com/slimphp/Slim/pull/2639)
- [Pull Request #2640](https://github.com/slimphp/Slim/pull/2640)
- [Pull Request #2641](https://github.com/slimphp/Slim/pull/2641)
- [Pull Request #2642](https://github.com/slimphp/Slim/pull/2642)

Это значит, что [группы маршрутов](/v4/objects/routing#route-groups) тоже изменили свои сигнатуры:

```php
$app->group('/user', function(\Slim\Routing\RouteCollectorProxy $app){
    $app->get('', function() { /* ... */ });
    //...
});
```

## Новый подход Middleware

В Slim 4 мы хотели предоставить разработчикам больше гибкости и вынесли некоторые функциональные возможности в middleware. Это дает вам возможность использовать собственные реализации основных компонентов.

## Очередность выполнения Middleware

Очередность выполнения middleware не изменилась с версии 3 (`Последний вошёл, первый вышел (LIFO)`).

## Новая фабрика App

Компонент `AppFactory` был введен для упрощения некоторых изменений, связанных с отделением реализации PSR-7 от ядра.
Он определяет имеющуюся в проекте реализацию PSR-7 и PSR-17, что позволяет создать экземпляр приложения с помощью `AppFactory::create()` и использовать `App::run()` для его запуска без создания объекта `ServerRequest`.
Поддерживаются следующие реализации PSR-7 и создатели ServerRequest:
- [Slim PSR-7](https://github.com/slimphp/Slim-Psr7)
- [Nyholm PSR-7](https://github.com/Nyholm/psr7) and [Nyholm PSR-7 Server](https://github.com/Nyholm/psr7-server)
- [Guzzle PSR-7](https://github.com/guzzle/psr7) and [Guzzle HTTP Factory](https://github.com/http-interop/http-factory-guzzle)
- [Laminas Diactoros](https://github.com/laminas/laminas-diactoros)

## Новый Middleware маршрутизации

Маршрутизация теперь реализована как middleware. Мы всё ещё используем [FastRoute](https://github.com/nikic/FastRoute) для маршрутизации приложения.
Если вы использовали настройку `defineRouteBeforeAppMiddleware`, вам нужно добавить middleware `Middleware\RoutingMiddleware` в ваше приложение непосредственно перед вызовом`run ()` для обеспечения предыдущего поведения.
Подробнее см. в  [Pull Request #2288](https://github.com/slimphp/Slim/pull/2288).

```php
<?php
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

// Add Routing Middleware
$app->addRoutingMiddleware();

// ...

$app->run();
```

## Новый middleware обработки ошибок

Обработка ошибок так же реализована в качестве middleware.
Подробнее см. в [Pull Request #2398](https://github.com/slimphp/Slim/pull/2398).

```php
<?php
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

/**
 * Middleware маршрутизации должно быть добавлено до ErrorMiddleware
 * Иначе, исключения, брошенные из маршрутизатора не будут обработаны
 */
$app->addRoutingMiddleware();

/**
 * Add Error Handling Middleware
 *
 * @param bool $displayErrorDetails -> В продакшене должно быть установлено значение false
 * @param bool $logErrors -> Параметер передается в дефолтный ErrorHandler
 * @param bool $logErrorDetails -> Показывать подробности ошибок в логе ошибок,
 * который может быть заменен по вашему выбору.
 
 * Примечание: это middleware должно быть добавлено последним. Все ошибки и исключения из добавленных позже middlware не будут обработаны.
 */
$app->addErrorMiddleware(true, true, true);

// ...

$app->run();
```


## Новые Not Found- и Not Allowed обработчики

[Обработчик ошибки 404 Not Found](http://www.slimframework.com/docs/v3/handlers/not-found.html) и 
[Обработчик ошибки 405 Not Allowed](http://www.slimframework.com/docs/v3/handlers/not-allowed.html) из версии 3
могут быть перенесены как показано ниже: 

```php
<?php
use Psr\Http\Message\ServerRequestInterface;
use Slim\Factory\AppFactory;
use Slim\Exception\HttpNotFoundException;
use Slim\Psr7\Response;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$errorMiddleware = $app->addErrorMiddleware(true, true, true);

// Установка обработчика ошибки Not Found
$errorMiddleware->setErrorHandler(
    HttpNotFoundException::class,
    function (ServerRequestInterface $request, Throwable $exception, bool $displayErrorDetails) {
        $response = new Response();
        $response->getBody()->write('404 NOT FOUND');

        return $response->withStatus(404);
    });

// Установка обработчика ошибки Not Allowed
$errorMiddleware->setErrorHandler(
    HttpMethodNotAllowedException::class,
    function (ServerRequestInterface $request, Throwable $exception, bool $displayErrorDetails) {
        $response = new Response();
        $response->getBody()->write('405 NOT ALLOWED');

        return $response->withStatus(405);
    });
```

## Новый диспетчер маршрутизации и результаты маршутизатора

Мы создали обертку над FastRoute, которая добавляет обертку результатов и доступ к полному списку разрешенных методов маршрута вместо того, чтобы иметь доступ к ним только в случае возникновения исключения.
Атрибут запроса `routeInfo` помечен как устаревший и заменен на `routingResults`.
См подробнее в [Pull Request #2405](https://github.com/slimphp/Slim/pull/2405).

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;
use Slim\Routing\RouteContext;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->get('/hello/{name}', function (Request $request, Response $response) {
    $routeContext = RouteContext::fromRequest($request);
    $routingResults = $routeContext->getRoutingResults();
    
    // Получаем все аргументы маршрута, например ['name' => 'John']
    $routeArguments = $routingResults->getRouteArguments();
    
    // В отличие Slim 3, разрешенные методы маршрута теперь доступны всегда, не только в случаях ошибки.
    $allowedMethods = $routingResults->getAllowedMethods();
    
    return $response;
});

// ...

$app->run();
```

## Новый middleware переопределения метода

Если вы переопределяли HTTP-метод, используя собственный заголовок или параметр тела запроса, вам нужно добавить middleware `Middleware\MethodOverrideMiddleware`, чтобы иметь эту возможность, как и раньше.
Подробности в [Pull Request #2329](https://github.com/slimphp/Slim/pull/2329).

```php
<?php
use Slim\Factory\AppFactory;
use Slim\Middleware\MethodOverridingMiddleware;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$methodOverridingMiddleware = new MethodOverrideMiddleware();
$app->add($methodOverridingMiddleware);

// ...

$app->run();
```


## Новый middleware размера ответа

Этот middleware автоматически добавляет к ответу заголовок `Content-Length`. Он используется взамен удаленной из 3 версии настройки `addContentLengthHeader`. Этот middleware должен располагаться в стеке middleware, чтобы он выполнялся после всех изменений тела ответа.

```php
<?php
use Slim\Factory\AppFactory;
use Slim\Middleware\ContentLengthMiddleware;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$contentLengthMiddleware = new ContentLengthMiddleware();
$app->add($contentLengthMiddleware);

// ...

$app->run();
```

## Новый middleware буферизации вывода

Middleware буферизации вывода позволяет переключаться между двумя режимами буферизации вывода: `APPEND` (по умолчанию) и `PREPEND`. Режим `APPEND` будет использовать существующее тело ответа для добавления содержимого, а режим `PREPEND` создаст новое тело ответа и добавит его к существующему ответу. Этот middleware должен располагаться в стеке middleware, чтобы он выполнялся после всех изменений тела ответа.

```php
<?php
use Slim\Factory\AppFactory;
use Slim\Middleware\OutputBufferingMiddleware;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

/**
 * Доступно 2 режима
 * OutputBufferingMiddleware::APPEND (по умолчанию) - Добавляет к текущему телу ответа
 * OutputBufferingMiddleware::PREPEND - Создает новое тело ответа
 */
$mode = OutputBufferingMiddleware::APPEND;
$outputBufferingMiddleware = new OutputBufferingMiddleware($mode);

// ...

$app->run();
```