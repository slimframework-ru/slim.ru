---
title: Router - маршрутизатор
---

Маршрутизатор Slim Framework построен поверх компонента [nikic/fastroute](https://github.com/nikic/FastRoute)  и он очень быстрый и стабильный.

<div id="how-to-create-routes"></div>

## Как создать маршруты

Вы можете определить маршруты приложений с помощью proxy methods (методов-посредников) в `\Slim\App` экземпляре. 
Slim Framework предоставляет методы для самых популярных HTTP-методов.

### GET Route

Вы можете добавить маршрут, который обрабатывает только `GET` HTTP-запросы с помощью `get()` метода Slim-приложения. 
Он принимает два аргумента:

1. Шаблон маршрута (с дополнительными именными заполнителями)
2. Обратный вызов маршрута (callback)

```php
$app = new \Slim\App();
$app->get('/books/{id}', function ($request, $response, $args) {
    // Show book identified by $args['id']
});
```

### Маршрут POST

Вы можете добавить маршрут, который обрабатывает только `POST` HTTP-запросы с помощью `post()` метода Slim-приложения. 
Он принимает два аргумента:

1. Шаблон маршрута (с дополнительными именными заполнителями)
2. Обратный вызов маршрута (callback)

```php
$app = new \Slim\App();
$app->post('/books', function ($request, $response, $args) {
    // Create new book
});
```

### PUT Route

Вы можете добавить маршрут, который обрабатывает только `PUT` HTTP-запросы с помощью `put()` метода Slim-приложения. 
Он принимает два аргумента:

1. Шаблон маршрута (с дополнительными именными заполнителями)
2. Обратный вызов маршрута (callback)

```php
$app = new \Slim\App();
$app->put('/books/{id}', function ($request, $response, $args) {
    // Update book identified by $args['id']
});
```

### DELETE Route

Вы можете добавить маршрут, который обрабатывает только `DELETE` HTTP-запросы с помощью `delete()` метода Slim-приложения. 
Он принимает два аргумента:

1. Шаблон маршрута (с дополнительными именными заполнителями)
2. Обратный вызов маршрута (callback)

```php
$app = new \Slim\App();
$app->delete('/books/{id}', function ($request, $response, $args) {
    // Delete book identified by $args['id']
});
```

### OPTIONS Route

Вы можете добавить маршрут, который обрабатывает только `OPTIONS` HTTP-запросы с помощью `options()` метода Slim-приложения. 
Он принимает два аргумента:

1. Шаблон маршрута (с дополнительными именными заполнителями)
2. Обратный вызов маршрута (callback)

```php
$app = new \Slim\App();
$app->options('/books/{id}', function ($request, $response, $args) {
    // Return response headers
});
```

### PATCH Route

Вы можете добавить маршрут, который обрабатывает только `PATCH` HTTP-запросы с помощью `patch()` метода Slim-приложения. 
Он принимает два аргумента:

1. Шаблон маршрута (с дополнительными именными заполнителями)
2. Обратный вызов маршрута (callback)

```php
$app = new \Slim\App();
$app->patch('/books/{id}', function ($request, $response, $args) {
    // Apply changes to book identified by $args['id']
});
```

### Любой Route

Вы можете добавить маршрут, который обрабатывает все методы HTTP-запроса с помощью `any()` метода Slim-приложения. 
Он принимает два аргумента:

1. Шаблон маршрута (с дополнительными именными заполнителями)
2. Обратный вызов маршрута (callback)

```php
$app = new \Slim\App();
$app->any('/books/[{id}]', function ($request, $response, $args) {
    // Apply changes to books or book identified by $args['id'] if specified.
    // To check which method is used: $request->getMethod();
});
```
Обратите внимание, что вторым параметром является обратный вызов. Вы можете указать класс (который нуждается в 
`__invoke()` реализации) вместо Closure. Затем вы можете выполнить сопоставление в другом месте:

```php
$app->any('/user', 'MyRestfulController');
```

### Пользовательский маршрут

Вы можете добавить маршрут, который обрабатывает несколько методов HTTP-запроса с помощью `map()` метода Slim-приложения. 
Он принимает три аргумента:

1. Массив методов HTTP
2. Шаблон маршрута (с дополнительными именными заполнителями)
3. The route callback

```php
$app = new \Slim\App();
$app->map(['GET', 'POST'], '/books', function ($request, $response, $args) {
    // Create new book or list all books
});
```

<div id="route-callbacks"></div>

## Route callbacks

Каждый описанный выше способ маршрутизации принимает в качестве последнего аргумента процедуру обратного вызова (callback). 
Этот аргумент может быть любым вызываемым PHP, и по умолчанию он принимает три аргумента.

**Request Запрос**
: Первый аргумент - это `Psr\Http\Message\ServerRequestInterface` объект, который представляет текущий HTTP-запрос.

**Response отклик**
: Второй аргумент - это `Psr\Http\Message\ResponseInterface` объект, который представляет текущий HTTP-ответ.

**аргументы**
: Третий аргумент представляет собой ассоциативный массив, который содержит значения для имен имен текущего маршрута.

### Написание контента для ответа

Существует два способа записи контента в ответ HTTP. Во-первых, вы можете просто `echo()` контент с _callback_ маршрута. 
Этот контент будет добавлен к текущему объекту ответа HTTP. Во-вторых, вы можете вернуть `Psr\Http\Message\ResponseInterface` объект.

###  Привязка Closure

Если вы используете `Closure` экземпляр в качестве обратного вызова маршрута, состояние закрытия привязывается к 
`Container` экземпляру. Это означает, что у вас будет доступ к экземпляру контейнера DI _внутри_ Closure с помощью `$this` 
ключевого слова:

```php
$app = new \Slim\App();
$app->get('/hello/{name}', function ($request, $response, $args) {
    // Use app HTTP cookie service
    $this->get('cookies')->set('name', [
        'value' => $args['name'],
        'expires' => '7 days'
    ]);
});
```

<div id="route-strategies"></div>

## Стратегии маршрута

Подпись обратного вызова маршрута определяется стратегией маршрута. По умолчанию Slim ожидает, что обратные вызовы 
маршрута будут принимать запрос, ответ и массив аргументов заполнителя маршрута. Это называется стратегией RequestResponse. 
Однако вы можете изменить ожидаемую подпись обратного вызова, просто используя другую стратегию. В качестве примера 
Slim предоставляет альтернативную стратегию, называемую RequestResponseArgs, которая принимает запрос и ответ, а также 
каждый заполнитель маршрута в качестве отдельного аргумента. Вот пример использования этой альтернативной стратегии; 
просто замените `foundHandler` зависимость, предоставленную по умолчанию `\Slim\Container`:

```php
$c = new \Slim\Container();
$c['foundHandler'] = function() {
    return new \Slim\Handlers\Strategies\RequestResponseArgs();
};

$app = new \Slim\App($c);
$app->get('/hello/{name}', function ($request, $response, $name) {
    return $response->write($name);
});
```

Вы можете предоставить свою собственную стратегию маршрута, выполнив ее  `\Slim\Interfaces\InvocationStrategyInterface`.

<div id="route-placeholders"></div>

## Заполнители маршрутов

Каждый описанный выше метод маршрутизации принимает шаблон URL, который сопоставляется с текущим URI запроса HTTP. 
Шаблоны маршрутов могут использовать именованные заполнители для динамического соответствия сегментам URI запроса HTTP.

### Формат

Заполнитель шаблона маршрута начинается с `{`, за которым следует имя заполнителя, заканчивающееся на `}`. Это пример заполнителя с именем `name`:

```php
$app = new \Slim\App();
$app->get('/hello/{name}', function ($request, $response, $args) {
    echo "Hello, " . $args['name'];
});
```

### Дополнительные сегменты

Чтобы сделать раздел необязательным, просто оберните в квадратные скобки:

```php
$app->get('/users[/{id}]', function ($request, $response, $args) {
    // responds to both `/users` and `/users/123`
    // but not to `/users/`
});
```

Несколько необязательных параметров поддерживаются вложением:

```php
$app->get('/news[/{year}[/{month}]]', function ($request, $response, $args) {
    // reponds to `/news`, `/news/2016` and `/news/2016/03`
});
```

Для необязательных параметров «Без ограничений» вы можете сделать это:

```php
$app->get('/news[/{params:.*}]', function ($request, $response, $args) {
    $params = explode('/', $request->getAttribute('params'));

    // $params is an array of all the optional segments
});
```

В этом примере, URI `/news/2016/03/20` приведет k `$params` массиву, содержащим три элемента: `['2016', '03', '20']`.


### Регулярное выражение

По умолчанию заполнители записываются внутри `{}` и могут принимать любые значения. Однако заполнители могут также 
требовать, чтобы URI HTTP-запроса соответствовал определенному регулярному выражению. Если текущий URI-запрос HTTP 
не соответствует регулярному выражению-заполнителю, маршрут не вызывается. Это пример имени заполнителя, `id` который 
требует одну или несколько цифр.

```php
$app = new \Slim\App();
$app->get('/users/{id:[0-9]+}', function ($request, $response, $args) {
    // Find user identified by $args['id']
});
```

<div id="route-names"></div>

## Названия маршрутов

Маршрутам приложения может быть присвоено имя. Это полезно, если вы хотите программно генерировать URL-адрес для 
определенного маршрута с помощью `pathFor()` метода маршрутизатора . Каждый описанный выше метод маршрутизации 
возвращает `\Slim\Route` объект, и этот объект предоставляет `setName()` метод.

```php
$app = new \Slim\App();
$app->get('/hello/{name}', function ($request, $response, $args) {
    echo "Hello, " . $args['name'];
})->setName('hello');
```

Вы можете создать URL-адрес для этого именованного маршрута с помощью `pathFor()` метода маршрутизатора приложения .

```php
echo $app->getContainer()->get('router')->pathFor('hello', [
    'name' => 'Josh'
]);
// Outputs "/hello/Josh"
```

Метод маршрутизатора `pathFor()` принимает два аргумента:

1. Название маршрута
2. Ассоциативный массив заполнителей шаблонов маршрутов и значения замещения

<div id="route-groups"></div>

## Группы маршрутов

Чтобы помочь организовать маршруты в логические группы, `\Slim\App` также предоставляется `group()` метод. Структура 
маршрута каждой группы добавляется к маршрутам или группам, содержащимся в ней, и любые аргументы-заполнители в 
групповом шаблоне в конечном итоге становятся доступными для вложенных маршрутов:

```php
$app = new \Slim\App();
$app->group('/users/{id:[0-9]+}', function () {
    $this->map(['GET', 'DELETE', 'PATCH', 'PUT'], '', function ($request, $response, $args) {
        // Find, delete, patch or replace user identified by $args['id']
    })->setName('user');
    $this->get('/reset-password', function ($request, $response, $args) {
        // Route for /users/{id:[0-9]+}/reset-password
        // Reset the password for user identified by $args['id']
    })->setName('user-password-reset');
});
```

Примечание внутри закрытия группы `$this` используется вместо `$app`. Slim связывает закрытие с экземпляром приложения 
для вас, точно так же, как в случае обратного вызова маршрута с экземпляром контейнера.

* внутри группового замыкания, `$this` связано с примером `Slim\App`
* закрытие внутри маршрута, `$this` связано с экземпляром `Slim\Container`

<div id="route-middleware"></div>

## Route middleware

Вы также можете подключить middleware к любому маршруту или группе маршрутов. [узнать больше](/concepts/middleware).

## Кэширование маршрутизатора

Кэш маршрутизатора можно включить, установив допустимое имя файла в настройках по умолчанию Slim. [узнать больше](/objects/application.html#slim-default-settings).

<div id="container-resolution"></div>

## Разрешение контейнера

Вы не ограничены определением функции для своих маршрутов. В Slim существует несколько разных способов определения ваших функций действия маршрута.

В дополнение к функции вы можете использовать:

 - `container_key:method`
 - `Class:method`
 - An invokable class
 - `container_key`
 
Эта функциональность включена Slim's Callable Resolver Class. Он преобразует строку в вызов функции. Пример:

```php
  $app->get('/', '\HomeController:home');
```

Кроме того, вы можете воспользоваться `::class` оператором PHP, который хорошо работает с системами поиска IDE и дает тот же результат:

```php
  $app->get('/', \HomeController::class . ':home');
```
В этом коде выше мы определяем `/` маршрут и сообщаем Slim о выполнении `home()` метода в `HomeController` классе.

Сначала Slim ищет запись `HomeController` в контейнере, если будет найден, он будет использовать этот экземпляр, иначе 
он будет называть его конструктором с контейнером в качестве первого аргумента. Когда экземпляр класса будет создан, 
он затем вызовет указанный метод, используя любую стратегию, которую вы определили.

### Регистрация контроллера с контейнером

Создайте контроллер с помощью `home` метода действий. Конструктор должен принимать зависимости, которые требуются. Например:

```php
class HomeController
{
    protected $view;
    
    public function __construct(\Slim\Views\Twig $view) {
        $this->view = $view;
    }
    public function home($request, $response, $args) {
      // your code here
      // use $this->view to render the HTML
      return $response;
    }
}
```

Создайте фабрику в контейнере, который запускает контроллер с зависимостями:

```php
$container = $app->getContainer();
$container['HomeController'] = function($c) {
    $view = $c->get("view"); // retrieve the 'view' from the container
    return new HomeController($view);
};
```

Это позволяет использовать контейнер для инъекций зависимостей, и поэтому вы можете 
вводить определенные зависимости в контроллер..


### Разрешить Slim создавать экземпляр контроллера

Альтернативно, если класс не имеет записи в контейнере, то Slim передает экземпляр контейнера в конструктор. 
Вы можете создавать контроллеры со многими действиями вместо вызываемого класса, который обрабатывает 
только одно действие.

```php
class HomeController 
{
   protected $container;

   // constructor receives container instance
   public function __construct(ContainerInterface $container) {
       $this->container = $container;
   }
   
   public function home($request, $response, $args) {
        // your code
        // to access items in the container... $this->container->get('');
        return $response;
   }
   
   public function contact($request, $response, $args) {
        // your code
        // to access items in the container... $this->container->get('');
        return $response;
   }
}
```

Вы можете использовать свои методы контроллера так.

```php
$app->get('/', \HomeController::class . ':home');
$app->get('/contact', \HomeController::class . ':contact');
```

### Использование вызываемого класса

Вам не нужно указывать метод в вашем маршруте, который может быть вызван, и может просто установить его как вызывающий класс, например:

```php
class HomeAction
{
   protected $container;
   
   public function __construct(ContainerInterface $container) {
       $this->container = $container;
   }
   
   public function __invoke($request, $response, $args) {
        // your code
        // to access items in the container... $this->container->get('');
        return $response;
   }
}
```

Вы можете использовать этот класс так.

```php
$app->get('/', \HomeAction::class);
```


Опять же, как и в случае с контроллерами, если вы зарегистрируете имя класса в контейнере, вы можете создать 
фабрику и ввести только конкретные зависимости, которые вам нужны, в ваш класс действий.