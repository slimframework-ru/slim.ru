---
title: Контейнер зависимости
---

Slim может использовать контейнер зависимостей для подготовки, управления и внедрения зависимостей приложения.
Slim поддерживает контейнеры, которые реализуют [PSR-11](http://www.php-fig.org/psr/psr-11/), например [PHP-DI](http://php-di.org/doc/frameworks /slim.html).

## Пример использования с PHP-DI

Вам _не нужно_ предоставлять контейнер зависимостей, но если намерены это сделать, вы должны передать экземпляр контейнера в `AppFactory` перед созданием `App`.

```php
<?php
use DI\Container;
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

// Создание контейнера PHP-DI
$container = new Container();

// Установка контейнера в создаваемый экземпляр приложения через фабрику
AppFactory::setContainer($container);
$app = AppFactory::create();
```

Добавление сервиса в контейнер:

```php
$container->set('myService', function () {
    $settings = [...];
    return new MyService($settings);
});
```

Вы можете получать сервисы из контейнера как явным образом, так и изнутри Slim, как здесь:

```php
/**
 * Пример маршрута GET
 *
 * @param  ServerRequestInterface $request  PSR-7 запрос
 * @param  ResponseInterface      $response  PSR-7 ответ
 * @param  array                  $args параметры маршрута
 *
 * @return ResponseInterface
 */
$app->get('/foo', function (Request $request, Response $response, $args) {
    $myService = $this->get('myService');

    // ...какие-то действия с $myService...

    return $response;
});
```

Для проверки наличия сервиса в контейнере перед использованием служит метод `has()`:

```php
/**
 * Пример маршрута GET
 *
 * @param  ServerRequestInterface $request  PSR-7 запрос
 * @param  ResponseInterface      $response  PSR-7 ответ
 * @param  array                  $args параметры маршрута
 *
 * @return ResponseInterface
 */
$app->get('/foo', function (Request $request, Response $response, $args) {
    if ($this->has('myService')) {
        $myService = $this->get('myService');
    }
    return $response;
});
```

## Конфигурирование приложения через контейнер

Если вы хотите создать `App` с зависимостями, уже определенными в вашем контейнере, вы можете использовать метод `AppFactory::createFromContainer()`.

**Пример**

```php
<?php

use App\Factory\MyResponseFactory;
use DI\Container;
use Psr\Container\ContainerInterface;
use Psr\Http\Message\ResponseFactoryInterface;
use Slim\Factory\AppFactory;

require_once __DIR__ . '/../vendor/autoload.php';

// Создание контейнера PHP-DI
$container = new Container();

// Добавление своей фабрики ответов
$container->set(ResponseFactoryInterface::class, function (ContainerInterface $container) {
    return new MyResponseFactory(...);
});

// Конфигурирование приложения через контейнер
$app = AppFactory::createFromContainer($container);

// ...

$app->run();
```

Поддерживаемые `App` зависимости:

* Psr\Http\Message\ResponseFactoryInterface
* Slim\Interfaces\CallableResolverInterface
* Slim\Interfaces\RouteCollectorInterface
* Slim\Interfaces\RouteResolverInterface
* Slim\Interfaces\MiddlewareDispatcherInterface
