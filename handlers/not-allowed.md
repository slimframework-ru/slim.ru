---
title: 405 обработчик Not Allowed
---

Если ваше приложение Slim Framework имеет маршрут, соответствующий текущему URI-запросу HTTP, 
**но не метод HTTP-запроса** , приложение вызывает обработчик Not Allowed и возвращает 
`HTTP/1.1 405 Not Allowed` ответ HTTP-клиенту.

## Обработчик Not Allowed по умолчанию

Каждое приложение Slim Framework имеет обработчик Not Allowed по умолчанию. Этот обработчик устанавливает 
статус ответа на него `405`, он устанавливает тип содержимого `text/html`, он добавляет `Allowed:` HTTP-заголовок 
с разделенным запятыми списком разрешенных HTTP-методов, и он пишет простое объяснение телу Response.

## Обработчик Not Allowed пользовательский

Не разрешенный обработчик приложения Slim Framework - это служба Pimple. Вы можете заменить свой собственный 
обработчик не разрешенным, указав собственный заводский метод Pimple с контейнером приложения.

```php
// Create Slim
$app = new \Slim\App();
// get the app's di-container
$c = $app->getContainer();
$c['notAllowedHandler'] = function ($c) {
    return function ($request, $response, $methods) use ($c) {
        return $c['response']
            ->withStatus(405)
            ->withHeader('Allow', implode(', ', $methods))
            ->withHeader('Content-type', 'text/html')
            ->write('Method must be one of: ' . implode(', ', $methods));
    };
};
```

> **N.B** Проверьте [Not Found](/docs/handlers/not-found.html) документы для метода предварительного тонкого создания, используя новый экземпляр `\Slim\Container`

В этом примере мы определяем новый `notAllowedHandler` завод, который возвращает вызываемый. Возвращаемый вызов допускает 
три аргумента:
In this example, we define a new `notAllowedHandler` factory that returns a callable. The returned callable accepts 
three arguments:

1. `\Psr\Http\Message\ServerRequestInterface` экземпляр
2. `\Psr\Http\Message\ResponseInterface` экземпляр
3. Числовой массив разрешенных имен методов HTTP

Вызываемый **ДОЛЖЕН** вернуть соответствующий  `\Psr\Http\Message\ResponseInterface` экземпляр.
