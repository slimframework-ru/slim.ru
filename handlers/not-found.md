---
title: 404 Not Found Handler
---

Если ваше приложение Slim Framework не имеет маршрута, соответствующего текущему URI-запросу HTTP, приложение вызывает 
обработчик Not Found и возвращает `HTTP/1.1 404 Not Found` ответ HTTP-клиенту.

## Обработчик по умолчанию Not Found

Each Slim Framework application has a default Not Found handler. This handler sets the Response status to `404`, it sets the content type to `text/html`, and it writes a simple explanation to the Response body.

## Пользовательский обработчик Not Found

Обработчик Not Found для Slim Framework - это услуга Pimple. Вы можете заменить свой собственный обработчик Not Found, 
указав собственный заводский метод Pimple с контейнером приложения.


```php
$c = new \Slim\Container(); //Create Your container

//Override the default Not Found Handler
$c['notFoundHandler'] = function ($c) {
    return function ($request, $response) use ($c) {
        return $c['response']
            ->withStatus(404)
            ->withHeader('Content-Type', 'text/html')
            ->write('Page not found');
    };
};

//Create Slim
$app = new \Slim\App($c);

//... Your code
```

В этом примере мы определяем новый `notFoundHandler` фабрику, который возвращает вызываемый. Возвращаемый вызов 
допускает два аргумента:

1.  `\Psr\Http\Message\ServerRequestInterface` экземпляр
2.  `\Psr\Http\Message\ResponseInterface` экземпляр

Вызываемый **ДОЛЖЕН** вернуть соответствующий `\Psr\Http\Message\ResponseInterface` экземпляр.
