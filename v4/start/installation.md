---
title: Установка
---

## Системные требования

* Веб-сервер с переписыванием URL-адресов
* PHP версии 7.2 или выше

## Шаг 1: Установка Composer

У вас нет Composer? Его легко установить, следуя инструкциям на [странице загрузки](https://getcomposer.org/download/).

## Шаг 2: Установка Slim

Мы рекомендуем устанавливать Slim с помощью [Composer](https://getcomposer.org/).
Перейдите в корневой каталог вашего проекта и выполните команду bash, приведенную ниже
Эта команда загружает фреймворк Slim и его зависимости в каталоге `vendor/` вашего проекта.

```bash
composer require slim/slim:"4.*"
```

## Шаг 3: Установка реализации PSR-7 и фабрики запроса

Прежде чем вы сможете начать работу со Slim, вам нужно выбрать наиболее подходящую вам реализацию PSR-7.
Чтобы автоопределение работало и позволяло вам использовать `AppFactory::create()` и `App::run()` без необходимости
вручную создавать `ServerRequest`, вам необходимо установить одну из следующих реализаций:

### [Slim PSR-7](https://github.com/slimphp/Slim-Psr7)
```bash
composer require slim/psr7
```

### [Nyholm PSR-7](https://github.com/Nyholm/psr7) and [Nyholm PSR-7 Server](https://github.com/Nyholm/psr7-server)
```bash
composer require nyholm/psr7 nyholm/psr7-server
```

### [Guzzle PSR-7](https://github.com/guzzle/psr7) and [Guzzle HTTP Factory](https://github.com/http-interop/http-factory-guzzle)
```bash
composer require guzzlehttp/psr7 http-interop/http-factory-guzzle
```

### [Laminas Diactoros](https://github.com/laminas/laminas-diactoros)
```bash
composer require laminas/laminas-diactoros
```

## Шаг 4: Hello World

Файл: `public/index.php`

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->get('/', function (Request $request, Response $response, $args) {
    $response->getBody()->write("Hello world!");
    return $response;
});

$app->run();
```