---
title: Документация
---

<div class="alert alert-info">
    <p>
        Эта документация предназначена для <strong>Slim 4</strong>. Ищете <a href="/v3">документацию по Slim 3</a>?
    </p>
</div>



## Добро пожаловать

Slim - это PHP микрофреймворк, который поможет вам
быстро создавать простые, но мощные веб-приложения и API. По своей сути, Slim просто
диспетчер, который получает HTTP-запрос, вызывает соответствующий обработчик и возвращает HTTP-ответ. Вот и все.

## В чем смысл?

Slim - это идеальный инструмент для создания API-интерфейсов, которые используют, обрабатывают или публикуют данные. 
Также Slim отлично подходит для быстрого прототипирования. Черт, вы даже можете создать полнофункциональный веб
приложения с пользовательскими интерфейсами. Что еще более важно, Слим очень быстрый
и имеет очень мало кода. Фактически, вы можете читать и понимать его исходный код
всего за день!

> По своей сути, Slim просто
  диспетчер, который получает HTTP-запрос, вызывает соответствующий обработчик и возвращает HTTP-ответ. Вот и все.

Вам не всегда нужен кухонный комбайн вроде [Symfony][symfony] или [Laravel][laravel].
Это несомненно отличные инструменты, но они сликом переполнены излишним функционалом.
Slim же, наоборот, предоставляет минимальный набор нужных инструментов, и не более того.

## Как это работает?

Во-первых, вам нужен веб-сервер, такой как Nginx или Apache. Вы должны [настроить свой веб-сервер](/v4/start/web-servers)
так, чтобы он отправлял все соответствующие запросы в один файл PHP «фронт-контроллер».
Вы создаете экземпляр приложения Slim и запускаете свое его в этом файле.

Приложение Slim содержит маршруты, отвечающие определенным HTTP-запросам.
Каждый маршрут вызывает обработчик и возвращает HTTP-ответ. 
Чтобы начать работу, сначала создайте экземпляр приложения и настройте приложение Slim. 
Затем вы определяете маршруты своего приложения. Наконец, вы запускаете приложение. Всё просто. Вот пример приложения:

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

/**
 * Instantiate App
 *
 * In order for the factory to work you need to ensure you have installed
 * a supported PSR-7 implementation of your choice e.g.: Slim PSR-7 and a supported
 * ServerRequest creator (included with Slim PSR-7)
 */
$app = AppFactory::create();

// Add Routing Middleware
$app->addRoutingMiddleware();

/**
 * Add Error Handling Middleware
 *
 * @param bool $displayErrorDetails -> Should be set to false in production
 * @param bool $logErrors -> Parameter is passed to the default ErrorHandler
 * @param bool $logErrorDetails -> Display error details in error log
 * which can be replaced by a callable of your choice.
 
 * Note: This middleware should be added last. It will not handle any exceptions/errors
 * for middleware added after it.
 */
$errorMiddleware = $app->addErrorMiddleware(true, true, true);

// Define app routes
$app->get('/hello/{name}', function (Request $request, Response $response, $args) {
    $name = $args['name'];
    $response->getBody()->write("Hello, $name");
    return $response;
});

// Run app
$app->run();
```

## Запрос и ответ

Когда вы создаете приложение Slim, вы часто работаете непосредственно с объектами запроса и ответа. 
Эти объекты представляют собой фактический HTTP-запрос, полученный веб-сервером, и возможный HTTP-ответ, 
возвращаемый клиенту.

Каждому маршруту Slim-приложения присваиваются текущие объекты запроса и ответа в качестве аргументов 
его обработчика. Эти объекты реализуют популярные интерфейсы [PSR-7](/v4/concepts/value-objects). 
Маршрут Slim-приложения может проверять или манипулировать этими объектами по мере необходимости. 
В конечном счете, каждый маршрут приложения Slim **ДОЛЖЕН** вернуть объект PSR-7 совместимого ответа.

## Принесите свои собственные компоненты

Slim также хорошо работает с другими компонентами PHP. Вы можете зарегистрировать дополнительные сторонние 
компоненты, такие как [Slim-Csrf][csrf], [Slim-HttpCache][httpcache] или [Slim-Flash][flash],
которые основываются на функциональных возможностях Slim по умолчанию. 
Также легко интегрировать сторонние компоненты, найденные в [Packagist](https://packagist.org/).

## Как читать эту документацию

Новичкам в Slim я рекомендую читать эту документацию от начала до конца. 
Если вы уже знакомы с фреймворком, то можете сразу перейти в соответствующий раздел.

Прежде чем вникать в конкретные темы (такие, как обработка запросов и ответов, маршрутизация и обработка ошибок), 
данная документация объясняет концепцию и архитектуру Slim.

## Лицензия документации
<p style="text-align: left;">
    Этот сайт и документация лицензированы <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License</a>.
    <br />
    <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">
        <img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" />
    </a>
</p>

[symfony]: https://symfony.com/
[laravel]: https://laravel.com/
[csrf]: https://github.com/slimphp/Slim-Csrf/
[httpcache]: https://github.com/slimphp/Slim-HttpCache
[flash]: https://github.com/slimphp/Slim-Flash