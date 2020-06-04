---
title: Контейнер зависимости - Dependency Container
---

Slim использует контейнер зависимостей для подготовки, управления и внедрения зависимостей приложений. 
Slim поддерживает контейнеры, реализующие [PSR-11](http://www.php-fig.org/psr/psr-11/) или интерфейс 
[Container-Interop](https://github.com/container-interop/container-interop)  Вы можете использовать встроенный 
контейнер Slim (на основе [Pimple](http://pimple.sensiolabs.org/))
или сторонние контейнеры, такие как [Acclimate](https://github.com/jeremeamia/acclimate-container)
или [PHP-DI](http://php-di.org/doc/frameworks/slim.html).

## Как использовать контейнер

Вам не _нужно_ предоставлять контейнер зависимостей. Однако, если вы это сделаете, вы должны вставить 
экземпляр контейнера в конструктор Slim-приложения.

```php
$container = new \Slim\Container;
$app = new \Slim\App($container);
```

Добавьте сервис в контейнер Slim:

```php
$container = $app->getContainer();
$container['myService'] = function ($container) {
    $myService = new MyService();
    return $myService;
};
```

Вы можете явно или неявно получать службы из своего контейнера. Вы можете получить 
явную ссылку на экземпляр контейнера из внутреннего маршрута приложения Slim следующим образом:

```php
/**
 * Example GET route
 *
 * @param  \Psr\Http\Message\ServerRequestInterface $req  PSR7 request
 * @param  \Psr\Http\Message\ResponseInterface      $res  PSR7 response
 * @param  array                                    $args Route parameters
 *
 * @return \Psr\Http\Message\ResponseInterface
 */
$app->get('/foo', function ($req, $res, $args) {
    $myService = $this->get('myService');

    return $res;
});
```

Вы можете неявно получать службы из контейнера следующим образом:

```php
/**
 * Example GET route
 *
 * @param  \Psr\Http\Message\ServerRequestInterface $req  PSR7 request
 * @param  \Psr\Http\Message\ResponseInterface      $res  PSR7 response
 * @param  array                                    $args Route parameters
 *
 * @return \Psr\Http\Message\ResponseInterface
 */
$app->get('/foo', function ($req, $res, $args) {
    $myService = $this->myService;

    return $res;
});
```

Чтобы проверить, существует ли служба в контейнере перед ее использованием, используйте этот `has()` метод, например:

```php
/**
 * Example GET route
 *
 * @param  \Psr\Http\Message\ServerRequestInterface $req  PSR7 request
 * @param  \Psr\Http\Message\ResponseInterface      $res  PSR7 response
 * @param  array                                    $args Route parameters
 *
 * @return \Psr\Http\Message\ResponseInterface
 */
$app->get('/foo', function ($req, $res, $args) {
    if($this->has('myService')) {
        $myService = $this->myService;
    }

    return $res;
});
```


Slim использует `__get()` и `__isset()` магические методы, которые переносят в контейнер 
приложения все свойства, которые еще не существуют в экземпляре приложения.

## Необходимые услуги

Ваш контейнер ДОЛЖЕН реализовать эти необходимые услуги. Если вы используете встроенный контейнер Slim, 
они предоставляются для вас. Если вы выберете сторонний контейнер, вы должны определить эти необходимые 
службы самостоятельно.

настройки
:   Ассоциативный массив параметров приложения, включая ключи:
    
    * `httpVersion`
    * `responseChunkSize`
    * `outputBuffering`
    * `determineRouteBeforeAppMiddleware`.
    * `displayErrorDetails`.
    * `addContentLengthHeader`.
    * `routerCacheFile`.

Окружающая среда    
:   Экземпляр `\Slim\Interfaces\Http\EnvironmentInterface`.

запрос    
:   Экземпляр `\Psr\Http\Message\ServerRequestInterface`.

ответ    
:   Экземпляр `\Psr\Http\Message\ResponseInterface`.  


маршрутизатор

:   Экземпляр `\Slim\Interfaces\RouterInterface`.

foundHandler
:   Экземпляр `\Slim\Interfaces\InvocationStrategyInterface`.

phpErrorHandler
:   Вызывается вызываемая, если возникает ошибка PHP 7. Вызываемый **ДОЛЖЕН** возвращать экземпляр  
`\Psr\Http\Message\ResponseInterface` и принимать три аргумента:

1. `\Psr\Http\Message\ServerRequestInterface`
2. `\Psr\Http\Message\ResponseInterface`
3. `\Error`

errorHandler
:   Вызывается вызываемая, если вызывается исключение. Вызываемый **ДОЛЖЕН** возвращать экземпляр  
`\Psr\Http\Message\ResponseInterface` и принимать три аргумента:

1. `\Psr\Http\Message\ServerRequestInterface`
2. `\Psr\Http\Message\ResponseInterface`
3. `\Exception`

notFoundHandler
:   Вызывается вызываемая, если текущий URI-запрос HTTP не соответствует маршруту приложения. Вызываемый 
**ДОЛЖЕН** возвращать экземпляр `\Psr\Http\Message\ResponseInterface` и принимать два аргумента:

1. `\Psr\Http\Message\ServerRequestInterface`
2. `\Psr\Http\Message\ResponseInterface`

notAllowedHandler
:   Вызываемый вызов, если маршрут приложения соответствует текущему пути HTTP-запроса, но не его методу. Вызываемый  
**ДОЛЖЕН** возвращать экземпляр `\Psr\Http\Message\ResponseInterface` and accept three arguments:

1. `\Psr\Http\Message\ServerRequestInterface`
2. `\Psr\Http\Message\ResponseInterface`
3. Array of allowed HTTP methods

callableResolver
:   Экземпляр `\Slim\Interfaces\CallableResolverInterface`.
