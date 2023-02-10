---
title: Маршрутизация
---
Маршрутизация была реализована как промежуточное программное обеспечение. Мы по-прежнему используем Fast Route в качестве маршрутизатора по умолчанию, но он не тесно связан с ним. Если бы вы хотели реализовать другую библиотеку маршрутизации, вы могли бы это сделать, создав свои собственные реализации интерфейсов маршрутизации. DispatcherInterface, RouteCollectorInterface, RouteParserInterface и RouteResolverInterface, которые создают мост между компонентами Slim и библиотекой маршрутизации. Если вы использовали промежуточное программное обеспечение define Route Before App, вам необходимо добавить промежуточное программное обеспечение Middleware\Routing Middleware в ваше приложение непосредственно перед вызовом run(), чтобы сохранить предыдущее поведение.

### Применение

```php
<?php
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

// Add Routing Middleware
$app->addRoutingMiddleware();

// ...

$app->run();
```
