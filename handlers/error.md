---
title: Обработчик системных ошибок
---

Все идет не так. Вы не можете предсказать ошибки, но можете их предвидеть. Каждое приложение Slim Framework имеет обработчик ошибок, который получает все исключенные исключения PHP. Этот обработчик ошибок также получает текущие объекты HTTP-запроса и ответа. Обработчик ошибок должен подготовить и вернуть соответствующий объект Response, который будет возвращен HTTP-клиенту.

## Обработчик ошибок по умолчанию

Обработчик ошибок по умолчанию очень простой. Он устанавливает код состояния ответа 500, устанавливает тип содержимого 
ответа `text/html` и добавляет общее сообщение об ошибке в тело ответа.

Это, _вероятно_, не подходит для разработки приложений. Вам настоятельно рекомендуется реализовать собственный обработчик ошибок Slim.

Обработчик ошибок по умолчанию также может содержать подробную информацию диагностики ошибок. Чтобы включить это, 
вам необходимо установить `displayErrorDetails` значение true:

```php
$configuration = [
    'settings' => [
        'displayErrorDetails' => true,
    ],
];
$c = new \Slim\Container($configuration);
$app = new \Slim\App($c);
```

## Пользовательский обработчик ошибок

Обработчик ошибок приложения Slim Framework - это услуга Pimple. Вы можете заменить свой собственный обработчик 
ошибок, указав собственный заводский метод Pimple с контейнером приложения.

Существует два способа ввода обработчиков:

### Предварительное приложение

```php
$c = new \Slim\Container();
$c['errorHandler'] = function ($c) {
    return function ($request, $response, $exception) use ($c) {
        return $c['response']->withStatus(500)
                             ->withHeader('Content-Type', 'text/html')
                             ->write('Something went wrong!');
    };
};
$app = new \Slim\App($c);
```

### Почтовое приложение

```php
$app = new \Slim\App();
$c = $app->getContainer();
$c['errorHandler'] = function ($c) {
    return function ($request, $response, $exception) use ($c) {
        return $c['response']->withStatus(500)
                             ->withHeader('Content-Type', 'text/html')
                             ->write('Something went wrong!');
    };
};
```

В этом примере мы определяем новый `errorHandler` завод, который возвращает вызываемый. Возвращаемый вызов допускает три аргумента:

1.  экземпляр `\Psr\Http\Message\ServerRequestInterface` 
2.  экземпляр `\Psr\Http\Message\ResponseInterface` 
3.  экземпляр `\Exception` 

Вызываемый **ДОЛЖЕН** возвращать новый `\Psr\Http\Message\ResponseInterface` экземпляр, подходящий для данного исключения

### Обработчик ошибок на основе классов

Обработчики ошибок также могут быть определены как вызываемый класс.

```php
class CustomHandler {
   public function __invoke($request, $response, $exception) {
        return $response
            ->withStatus(500)
            ->withHeader('Content-Type', 'text/html')
            ->write('Something went wrong!');
   }
}
```

и прилагается так:

```php
$app = new \Slim\App();
$c = $app->getContainer();
$c['errorHandler'] = function ($c) {
    return new CustomHandler();
};
```

Это позволяет нам определять более сложные обработчики или расширять / отменять встроенные `Slim\Handlers\*` классы.

### Обработка других ошибок

**Обратите внимание** : следующие четыре типа исключений не будут обрабатываться обычаем `errorHandler`:

- `Slim\Exception\MethodNotAllowedException`: Это можно обрабатывать с помощью пользовательского [`notAllowedHandler`](/docs/handlers/not-allowed.html).
- `Slim\Exception\NotFoundException`: Это можно обрабатывать с помощью пользовательского [`notFoundHandler`](/docs/handlers/not-found.html).
- Ошибки PHP Runtime (только для PHP 7+):  Это можно обрабатывать с помощью пользовательского [`phpErrorHandler`](/docs/handlers/php-error.html).
- `Slim\Exception\SlimException`: Этот тип исключения является внутренним для Slim, и его обработка не может быть переопределена.

### Отключение

Чтобы полностью отключить обработку ошибок Slim, просто удалите обработчик ошибок из контейнера:

```php
unset($app->getContainer()['errorHandler']);
unset($app->getContainer()['phpErrorHandler']);
```

Теперь вы отвечаете за обработку любых исключений, которые возникают в вашем приложении, поскольку Slim не будет обрабатываться
