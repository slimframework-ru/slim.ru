---
title: Руководство по обновлению
---

Если вы обновляетесь с версии 2 до версии 3, это существенные изменения, о которых вам нужно знать.

## Новая версия PHP
Для Slim 3 требуется PHP 5.5+

## Класс \ Slim \ Slim переименован \ Slim \ App
Slim 3 использует `\Slim\App` для [Application](/docs/objects/application.html) объект обычно называется `$app`.

```php
$app = new \Slim\App();
```

## Подпись новой функции маршрута

```php
$app->get('/', function (Request $req,  Response $res, $args = []) {
    return $res->withStatus(400)->write('Bad Request');
});
```


## Объекты запроса и ответа больше не доступны через объект приложения
Как упоминалось выше, Slim 3 передает объекты `Request` и `Response` объекты в качестве аргументов функции обработки 
маршрута. Поскольку они теперь доступны непосредственно в теле функции маршрута `request` и `response` больше не 
являются объектами экземпляра `/Slim/App` ([Application](/docs/objects/application.html) object).

## Получение переменных _GET и _POST

```php
$app->get('/', function (Request $req,  Response $res, $args = []) {
    $myvar1 = $req->getParam('myvar'); //checks both _GET and _POST [NOT PSR-7 Compliant]
    $myvar2 = $req->getParsedBody()['myvar']; //checks _POST  [IS PSR-7 compliant]
    $myvar3 = $req->getQueryParams()['myvar']; //checks _GET [IS PSR-7 compliant]
});
```

## Хуки
Хуки больше не являются частью Slim по сравнению с v3. Вы должны рассмотреть реализовав какие - либо функциональные 
возможности, связанные с [default hooks in Slim v2](http://docs.slimframework.com/hooks/defaults/) как
 [middleware](/docs/concepts/middleware.html) вместо. Если вам нужна возможность применять пользовательские Хуки 
 в произвольных точках вашего кода (например, в пределах route),вам следует рассмотреть сторонний пакет, например
  [Symfony's EventDispatcher](http://symfony.com/doc/current/components/event_dispatcher/introduction.html) или
   [Zend Framework's EventManager](https://zend-eventmanager.readthedocs.org/en/latest/).

## Удаление HTTP-кеша
В Slim v3 мы удалили HTTP-кэширование в свой собственный модуль [Slim\Http\Cache](https://github.com/slimphp/Slim-HttpCache).

## Удаление остановки/остановки `Stop/Halt`
Slim Core удалил Stop / Halt. В ваших приложениях вы должны перейти к использованию методов withStatus () и withBody ().

## Удаление автозагрузчика
`Slim::registerAutoloader()` были удалены, мы полностью перешли к композитору.

## Изменения в контейнере
`$app->container->singleton(...)` теперь `$container = $app->getContainer(); $container['...'] = function () {};`
 читайте в Pimple docs для получения дополнительной информации

## Удаление configureMode()
`$app->configureMode(...)` был удален в v3.

## Удаление PrettyExceptions
PrettyExceptions вызывают множество проблем для многих людей, поэтому они были удалены.

## Route::setDefaultConditions(...) удален
Мы включили маршрутизаторы, которые позволяют вам сохранять регулярные выражения по умолчанию внутри шаблона маршрута.

## Изменения в перенаправлении
В Slim v2.x можно использовать вспомогательную функцию `$app->redirect();` для запуска запроса перенаправления.
В Slim v3.x можно сделать то же самое с использованием класса Response.

Пример:

```php
$app->get('/', function ($req, $res, $args) {
  return $res->withStatus(302)->withHeader('Location', 'your-new-uri');
});
``` 

Также, если вы хотите перенаправить маршрут без какой-либо другой обработки, вы можете использовать вспомогательную функцию `$app->redirect()`:

```php
$app->redirect('/', 'your-new-uri');
```

## Сигнатура `Middleware` промежуточного ПО 
Сигнатура middleware изменилась с класса на функцию.

Новая подпись:

```php
use Psr\Http\Message\RequestInterface as Request;
use Psr\Http\Message\ResponseInterface as Response;

$app->add(function (Request $req,  Response $res, callable $next) {
    // Do stuff before passing along
    $newResponse = $next($req, $res);
    // Do stuff after route is rendered
    return $newResponse; // continue
});
```

Вы все равно можете использовать класс:

```php
namespace My;

use Psr\Http\Message\RequestInterface as Request;
use Psr\Http\Message\ResponseInterface as Response;

class Middleware
{
    function __invoke(Request $req,  Response $res, callable $next) {
        // Do stuff before passing along
        $newResponse = $next($req, $res);
        // Do stuff after route is rendered
        return $newResponse; // continue
    }
}

// Register
$app->add(new My\Middleware());
// or
$app->add(My\Middleware::class);
```


## Выполнение Middleware
Приложение middleware выполняется как Last In First Executed (LIFE).

## Flash-сообщения 
Flash-сообщения больше не являются частью ядра Slim v3, а вместо этого перемещены в отдельный пакет
 [Slim Flash](/docs/features/flash.html).

## Cookies
В v3.0 файлы cookie удалены из ядра. См.  [FIG Cookies](https://github.com/dflydev/dflydev-fig-cookies) для файла 
cookie, совместимого с PSR-7

## Удаление Crypto
В v3.0 мы удалили зависимость для crypto в ядре.

## Новый маршрутизатор `Router`
Slim теперь использует [FastRoute](https://github.com/nikic/FastRoute), новый, более мощный маршрутизатор!

Это означает, что спецификация шаблонов маршрутов изменилась с именованными параметрами теперь в фигурных скобках и 
квадратных скобках, используемых для необязательных сегментов:

```php
// named parameter:
$app->get('/hello/{name}', /*...*/);

// optional segment:
$app->get('/news[/{year}]', /*...*/);

```

## Маршрутное промежуточное ПО `Route Middleware`
Синтаксис добавления промежуточного ПО маршрута несколько изменился. В версии 3.0:

```php
$app->get(…)->add($mw2)->add($mw1);
```

## Получение текущего маршрута
Маршрут является атрибутом объекта Request в v3.0:

```php
$request->getAttribute('route');
```

При получении текущего route в middleware, значение параметра
`determineRouteBeforeAppMiddleware` должно быть установлено `true` в конфигурации приложения, в противном случае
 возвращается вызов getAttribute `null`.

## urlFor() теперь pathFor() в маршрутизаторе

`urlFor()` был переименован `pathFor()` и может быть найден в `router` объекте:

```php
$app->get('/', function ($request, $response, $args) {
    $url = $this->router->pathFor('home');
    $response->write("<a href='$url'>Home</a>");
    return $response;
})->setName('home');
```

Также, `pathFor()` известен базовый путь.

## Контейнер и DI ... Построение
Slim использует Pimple в качестве контейнера для инъекций зависимостей.

```php
// index.php
$app = new Slim\App(
    new \Slim\Container(
        include '../config/container.config.php'
    )
);

// Slim will grab the Home class from the container defined below and execute its index method.
// If the class is not defined in the container Slim will still contruct it and pass the container as the first arugment to the constructor!
$app->get('/', Home::class . ':index');


// In container.config.php
// We are using the SlimTwig here
return [
    'settings' => [
        'viewTemplatesDirectory' => '../templates',
    ],
    'twig' => [
        'title' => '',
        'description' => '',
        'author' => ''
    ],
    'view' => function ($c) {
        $view = new Twig(
            $c['settings']['viewTemplatesDirectory'],
            [
                'cache' => false // '../cache'
            ]
        );

        // Instantiate and add Slim specific extension
        $view->addExtension(
            new TwigExtension(
                $c['router'],
                $c['request']->getUri()
            )
        );

        foreach ($c['twig'] as $name => $value) {
            $view->getEnvironment()->addGlobal($name, $value);
        }

        return $view;
    },
    Home::class => function ($c) {
        return new Home($c['view']);
    }
];
```

## Объекты PSR-7

### `Request, Response` Запрос, ответ,, Uri & UploadFile неизменяемы.
Это означает, что при изменении одного из этих объектов старый экземпляр не обновляется.

```php
// This is WRONG. The change will not pass through.
$app->add(function (Request $request, Response $response, $next) {
    $request->withAttribute('abc', 'def');
    return $next($request, $response);
});

// This is correct.
$app->add(function (Request $request, Response $response, $next) {
    $request = $request->withAttribute('abc', 'def');
    return $next($request, $response);
});
```

### Телами сообщений являются потоки

```php
// ...
$image = __DIR__ . '/huge_photo.jpg';
$body = new Stream($image);
$response = (new Response())
     ->withStatus(200, 'OK')
     ->withHeader('Content-Type', 'image/jpeg')
     ->withHeader('Content-Length', filesize($image))
     ->withBody($body);
// ...
```

Для текста:

```php
// ...
$response = (new Response())->getBody()->write('Hello world!')

// Or Slim specific: Not PSR-7 compliant.
$response = (new Response())->write('Hello world!');
// ...
```
