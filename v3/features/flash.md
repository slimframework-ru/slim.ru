---
title: Флеш-Сообщения
---

## Установка

через Composer

``` bash
$ composer require slim/flash
```

Требуется Slim 3.0.0 или новее.

## Применение

```php
// Начало сессии в PHP
session_start(); // по умолчанию не требует хранения сессии

$app = new \Slim\App();

// Fetch DI Container
$container = $app->getContainer();

// Провайдер регистрации
$container['flash'] = function () {
    return new \Slim\Flash\Messages();
};

$app->get('/foo', function ($req, $res, $args) {
    // Установка флэш-сообщения для следующего запроса
    $this->flash->addMessage('Test', 'This is a message');

    // Redirect
    return $res->withStatus(302)->withHeader('Location', '/bar');
});

$app->get('/bar', function ($req, $res, $args) {
    // Получение флэш-сообщений из предыдущего запроса
    $messages = $this->flash->getMessages();
    print_r($messages);
});

$app->run();
```
Обратите внимание, что сообщение может быть строкой, объектом или массивом. Проверьте, что может хранить ваше хранилище.
