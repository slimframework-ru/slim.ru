---
title: Настройка CORS
---

CORS - совместное использование ресурсов между разными источниками

Хорошая блок-схема для реализации поддержки CORS. Ссылка:

[CORS server flowchart](http://www.html5rocks.com/static/images/cors_server_flowchart.png)

Вы можете проверить свою поддержку CORS здесь: http://www.test-cors.org/

Здесь вы можете ознакомиться со спецификацией: https://www.w3.org/TR/cors/


## Простое решение

Для простых запросов CORS серверу необходимо добавить в ответ следующий заголовок:

`Access-Control-Allow-Origin: <domain>, ... | *`

Следующий код должен включать ленивый CORS.

```php
$app->options('/{routes:.+}', function ($request, $response, $args) {
    return $response;
});

$app->add(function ($req, $res, $next) {
    $response = $next($req, $res);
    return $response
            ->withHeader('Access-Control-Allow-Origin', 'http://mysite')
            ->withHeader('Access-Control-Allow-Headers', 'X-Requested-With, Content-Type, Accept, Origin, Authorization')
            ->withHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
});

```



## Access-Control-Allow-Methods

Следующее middleware может использоваться для запроса маршрутизатора Slim и получения 
списка методов, которые реализует определенный шаблон.

Вот пример приложения:

```php
require __DIR__ . "/vendor/autoload.php";

// Эта настройка Slim требуется для того, чтобы middleware работало
$app = new Slim\App([
    "settings"  => [
        "determineRouteBeforeAppMiddleware" => true,
    ]
]);

// Это the middleware
// Он будет добавлять заголовок Access-Control-Allow-Methods для каждого запроса

$app->add(function($request, $response, $next) {
    $route = $request->getAttribute("route");

    $methods = [];

    if (!empty($route)) {
        $pattern = $route->getPattern();

        foreach ($this->router->getRoutes() as $route) {
            if ($pattern === $route->getPattern()) {
                $methods = array_merge_recursive($methods, $route->getMethods());
            }
        }
        // Методы содержат все HTTP-глаголы, которые обрабатывает конкретный маршрут.
    } else {
        $methods[] = $request->getMethod();
    }
    
    $response = $next($request, $response);


    return $response->withHeader("Access-Control-Allow-Methods", implode(",", $methods));
});

$app->get("/api/{id}", function($request, $response, $arguments) {
});

$app->post("/api/{id}", function($request, $response, $arguments) {
});

$app->map(["DELETE", "PATCH"], "/api/{id}", function($request, $response, $arguments) {
});

// Обратите внимание на это, когда вы используете фреймворк javascript front-end, и вы используете группы в slim php
$app->group('/api', function () {
    // Из-за поведения браузеров при отправке запроса PUT или DELETE вы должны добавить метод OPTIONS. Читайте о предполетном освещении.
    $this->map(['PUT', 'OPTIONS'], '/{user_id:[0-9]+}', function ($request, $response, $arguments) {
        // Ваш код здесь ...
    });
});

$app->run();
```

Большое спасибо [tuupola](https://github.com/tuupola) за это!

