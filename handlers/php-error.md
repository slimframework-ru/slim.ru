---
title: Обработчик ошибок PHP
---

Если ваше приложение Slim Framework выдает ошибку [PHP Runtime error](http://php.net/manual/en/class.error.php)
 (только для PHP 7+), приложение вызывает обработчик ошибок PHP и возвращает 
 `HTTP/1.1 500 Internal Server Error` ответ HTTP-клиенту.

## Обработчик ошибок PHP по умолчанию

Каждое приложение Slim Framework имеет обработчик ошибок PHP по умолчанию. Этот обработчик устанавливает 
статус ответа `500`, он устанавливает тип содержимого `text/html` и записывает простое объяснение телу Response.

## Пользовательский обработчик ошибок PHP

Обработчик ошибок PHP Slim Framework - это служба Pimple. Вы можете заменить свой 
собственный обработчик ошибок PHP, указав собственный заводский метод Pimple с контейнером приложения.

```php
// Create Slim
$app = new \Slim\App();
// get the app's di-container
$c = $app->getContainer();
$c['phpErrorHandler'] = function ($c) {
    return function ($request, $response, $error) use ($c) {
        return $c['response']
            ->withStatus(500)
            ->withHeader('Content-Type', 'text/html')
            ->write('Something went wrong!');
    };
};
```

> **N.B** Проверьте [Not Found](/docs/handlers/not-found.html) документы для метода предварительного тонкого 
> создания, используя новый экземпляр `\Slim\Container`

В этом примере мы определяем новый `phpErrorHandler` завод, который возвращает вызываемый. 
Возвращаемый вызов допускает три аргумента:

1. `\Psr\Http\Message\ServerRequestInterface` экземпляр
2. `\Psr\Http\Message\ResponseInterface` экземпляр
3. `\Throwable` экземпляр

Вызываемый **ДОЛЖЕН** вернуть новый `\Psr\Http\Message\ResponseInterface` 
экземпляр, подходящий для данной ошибки.
