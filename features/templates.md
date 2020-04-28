---
title: Шаблоны
---

У Slim нет слоя представления, такого как традиционные рамки MVC. "view" Slim _является ответом HTTP_. 
Каждый тонкий маршрут приложения отвечает за подготовку и возвращение соответствующего объекта ответа PSR 7.

> "view" Slim - это ответ HTTP.

Это, как говорится, Slim проект предусматривает [Twig-View](#the-slimtwig-view-component) и
[PHP-View](#the-slimphp-view-component) компоненты, которые помогут вам создать шаблоны для объекта PSR7 Response.  
components to help you render templates to a PSR7
Response object.

## slim/twig-view Компонент

Компонент [Twig-View][twigview] PHP помогает построить [Twig][twig]
шаблоны в вашем приложении. Этот компонент доступер в Packagist, и
его можно установить в Composer следующим образом:

[twigview]: https://github.com/slimphp/Twig-View
[twig]: http://twig.sensiolabs.org/

```php
composer require slim/twig-view
```
<figure>
<figcaption>Figure 1: Установка slim/twig-view компонента.</figcaption>
</figure>

Затем вам необходимо зарегистрировать компонент в качестве службы в контейнере Slim, например:

```php
<?php
// Create app
$app = new \Slim\App();

// Get container
$container = $app->getContainer();

// Register component on container
$container['view'] = function ($container) {
    $view = new \Slim\Views\Twig('path/to/templates', [
        'cache' => 'path/to/cache'
    ]);
    
    // Instantiate and add Slim specific extension
    $basePath = rtrim(str_ireplace('index.php', '', $container['request']->getUri()->getBasePath()), '/');
    $view->addExtension(new Slim\Views\TwigExtension($container['router'], $basePath));

    return $view;
};
```
<figure>
<figcaption>Figure 2: Зарегистрируйте компонент slim / twig-view с контейнером.</figcaption>
</figure>

Примечание: для кеша может быть установлено значение false, чтобы отключить его, см. Также параметр «auto_reload», 
полезный в среде разработки. Дополнительные сведения см. 
В разделе  [Twig environment options](http://twig.sensiolabs.org/doc/2.x/api.html#environment-options)

Теперь вы можете использовать `slim/twig-view` службу компонента внутри маршрута приложения 
для визуализации шаблона и записать его в объект PSR 7 Response следующим образом:

```php
// Render Twig template in route
$app->get('/hello/{name}', function ($request, $response, $args) {
    return $this->view->render($response, 'profile.html', [
        'name' => $args['name']
    ]);
})->setName('profile');

// Run app
$app->run();
```
<figure>
<figcaption>Figure 3:  Шаблон Render с тонким / twig-view контейнером.</figcaption>
</figure>

В этом примере, `$this->view` вызываемый внутри callback маршрута, является 
ссылкой на `\Slim\Views\Twig` экземпляр, возвращаемый службой `view` контейнера. 
Метод `\Slim\Views\Twig` экземпляра `render()` принимает объект ответа PSR 7 как его 
первый аргумент, путь шаблона Twig как его второй аргумент и массив переменных 
шаблона в качестве его последнего аргумента. Метод `render()` возвращает новый объект 
PSR 7 Response , чье тело создано шаблоном Twig.


### Метод path_for()

Компонент`slim/twig-view` представляет пользовательскую функцию `path_for()` шаблона Twig. 
 Вы можете использовать эту функцию для создания полных URL-адресов для любого именованного 
 маршрута в своем Slim-приложении. Функция `path_for()` принимает два аргумента:

1. Имя маршрута
2. Хэш имен заполнителей маршрута и значений замены.

Второй аргумент ключевой должен соответствовать записям шаблонов выбранного маршрута. 
Это пример шаблона Twig, который рисует URL-адрес ссылки для "profile" с именем route, 
показанного в примере Slim, описанном выше.
```html

{% raw %}
{% extends "layout.html" %}

{% block body %}
<h1>User List</h1>
<ul>
    <li><a href="{{ path_for('profile', { 'name': 'josh' }) }}">Josh</a></li>
</ul>
{% endblock %}
{% endraw %}
```

## Компонент slim/php-view

[PHP-View][phpview] компонент PHP помогает визуализации шаблонов PHP. Этот компонент доступен 
в Packagist и может быть установлен с помощью Composer следующим образом:

[phpview]: https://github.com/slimphp/PHP-View

```php
composer require slim/php-view
```
<figure>
<figcaption>Figure 4: Установите компонент slim / php-view.</figcaption>
</figure>

Чтобы зарегистрировать этот компонент в качестве службы в контейнере Slim App, выполните следующие действия:

```php
<?php
// Create app
$app = new \Slim\App();

// Get container
$container = $app->getContainer();

// Register component on container
$container['view'] = function ($container) {
    return new \Slim\Views\PhpRenderer('path/to/templates/with/trailing/slash/');
};
```
```
<figure>
<figcaption>Figure 5: Регистрация компонента slim/php-view с контейнером.</figcaption>
</figure>

Используйте компонент вида для визуализации представления PHP следующим образом:

```php
```php

// Render PHP template in route
$app->get('/hello/{name}', function ($request, $response, $args) {
    return $this->view->render($response, 'profile.html', [
        'name' => $args['name']
    ]);
})->setName('profile');

// Run app
$app->run();
```
<figure>
<figcaption>Figure 6: Создайте шаблон с помощью сервиса slim/php-view.</figcaption>
</figure>

## Другие системы шаблонов

Вы не ограничены `Twig-View` и `PHP-View` компонентами. Вы можете использовать любую систему 
шаблонов PHP, при условии, что вы в конечном итоге напишите вывод обработанного шаблона в тело объекта PSR 7 Response.