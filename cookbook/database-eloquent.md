---
title: Использование Eloquent вместе со Slim
---

Вы можете использовать ORM базы данных, например [Eloquent](https://laravel.com/docs/5.1/eloquent) для подключения 
вашего приложения SlimPHP к базе данных.

## Добавление Eloquent в ваше приложение

```
composer require illuminate/database "~5.1"
```
<figure>
<figcaption>Figure 1: Добвить Eloquent в ваше приложение.</figcaption>
</figure>

## Настройки Eloquent

Добавьте настройки базы данных в массив настроек Slim.

```php
<?php
return [
    'settings' => [
        // Slim Settings
        'determineRouteBeforeAppMiddleware' => false,
        'displayErrorDetails' => true,
        'db' => [
            'driver' => 'mysql',
            'host' => 'localhost',
            'database' => 'database',
            'username' => 'user',
            'password' => 'password',
            'charset'   => 'utf8',
            'collation' => 'utf8_unicode_ci',
            'prefix'    => '',
        ]
    ],
];
```
<figure>
    <figcaption>Figure 2: Массив настроек.</figcaption>
</figure>

В вашем `dependencies.php` или где вы добавляете свои Служебные Фабрики:

```php
// Service factory for the ORM
$container['db'] = function ($container) {
    $capsule = new \Illuminate\Database\Capsule\Manager;
    $capsule->addConnection($container['settings']['db']);

    $capsule->setAsGlobal();
    $capsule->bootEloquent();

    return $capsule;
};
```
<figure>
<figcaption>Figure 3: Конфигурация Eloquent.</figcaption>
</figure>

## Передайте контроллеру экземпляр вашей таблицы

```php
$container[App\WidgetController::class] = function ($c) {
    $view = $c->get('view');
    $logger = $c->get('logger');
    $table = $c->get('db')->table('table_name');
    return new \App\WidgetController($view, $logger, $table);
};
```
<figure>
<figcaption>Figure 4: Передайте объект таблицы в контроллер.</figcaption>
</figure>

## Запросить таблицу с контроллера

```php
<?php

namespace App;

use Slim\Views\Twig;
use Psr\Log\LoggerInterface;
use Illuminate\Database\Query\Builder;
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Message\ResponseInterface as Response;

class WidgetController
{
    private $view;
    private $logger;
    protected $table;

    public function __construct(
        Twig $view,
        LoggerInterface $logger,
        Builder $table
    ) {
        $this->view = $view;
        $this->logger = $logger;
        $this->table = $table;
    }

    public function __invoke(Request $request, Response $response, $args)
    {
        $widgets = $this->table->get();

        $this->view->render($response, 'app/index.twig', [
            'widgets' => $widgets
        ]);

        return $response;
    }
}
```
<figure>
<figcaption>Figure 5:  Пример контроллера, запрашивающего таблицу.</figcaption>
</figure>

### Запрос из таблицы с where

```php
...
$records = $this->table->where('name', 'like', '%foo%')->get();
...
```
<figure>
<figcaption>Figure 6: Поиск запросов для имен, соответствующих foo.</figcaption>
</figure>

### Запрос таблицы по идентификатору

```php
...
$record = $this->table->find(1);
...
```
<figure>
<figcaption>Figure 7:  Выбор строки на основе идентификатора.</figcaption>
</figure>

## Больше информации

[Eloquent](https://laravel.com/docs/5.1/eloquent) Documentation
