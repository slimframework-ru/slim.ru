---
title: Получение текущего маршрута
---

Если вам когда-либо понадобится доступ к текущему маршруту в вашем приложении, все, что вам нужно сделать, 
это вызвать метод класса запроса `getAttribute` с аргументом, `'route'` и он вернет текущий маршрут, который является 
экземпляром `Slim\Route` класса.

Оттуда вы можете получить имя маршрута, используя `getName()` или получить методы, поддерживаемые этим маршрутом, 
через `getMethods()` и т. Д.

Примечание. Если вам нужно получить доступ к маршруту из middleware вашего приложения, вы должны установить 
`'determineRouteBeforeAppMiddleware'` значение true в своей конфигурации, иначе оно `getAttribute('route')` вернет значение 
null. Также `getAttribute('route')` возвратит null на несуществующих маршрутах.

Пример:
```php
use Slim\App;
use Slim\Exception\NotFoundException;
use Slim\Http\Request;
use Slim\Http\Response;

$app = new App([
    'settings' => [
        // Установите это только в том случае, если вам нужен доступ к маршруту внутри middleware
        'determineRouteBeforeAppMiddleware' => true
    ]
]);

// routes...
$app->add(function (Request $request, Response $response, callable $next) {
    $route = $request->getAttribute('route');

    // return NotFound для несуществующего маршрута
    if (empty($route)) {
        throw new NotFoundException($request, $response);
    }

    $name = $route->getName();
    $groups = $route->getGroups();
    $methods = $route->getMethods();
    $arguments = $route->getArguments();

    // do something with that information

    return $next($request, $response);
});
```
