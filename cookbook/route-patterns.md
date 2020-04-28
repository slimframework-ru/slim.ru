---
title: Завершение / в роутах
---

Slim обрабатывает шаблон URL с косой чертой в конце и без по разному. То есть, `/user` и `/user/` разные, и поэтому 
могут иметь разные callbacks вызовы.

Для запросов GET постоянное перенаправление выполняется нормально, но для других методов запроса, таких как POST 
или PUT, браузер отправит второй запрос с помощью метода GET. Чтобы этого избежать, вам просто нужно удалить конечную 
косую черту и передать управляемый URL-адрес следующему middleware.

Если вы хотите перенаправить/переписать все URL-адреса, которые заканчиваются `/` на не-trailing `/` эквивалент, 
вы можете добавить это middleware:

```php
use Psr\Http\Message\RequestInterface as Request;
use Psr\Http\Message\ResponseInterface as Response;

$app->add(function (Request $request, Response $response, callable $next) {
    $uri = $request->getUri();
    $path = $uri->getPath();
    if ($path != '/' && substr($path, -1) == '/') {
        // permanently redirect paths with a trailing slash
        // to their non-trailing counterpart
        $uri = $uri->withPath(substr($path, 0, -1));
        
        if($request->getMethod() == 'GET') {
            return $response->withRedirect((string)$uri, 301);
        }
        else {
            return $next($request->withUri($uri), $response);
        }
    }

    return $next($request, $response);
});
```

В качестве альтернативы рассмотрим 
[oscarotero/psr7-middlewares' TrailingSlash](//github.com/oscarotero/psr7-middlewares#trailingslash) 
которое также позволяет принудительно привязать конечную косую черту ко всем URL-адресам:

```php
use Psr7Middlewares\Middleware\TrailingSlash;

$app->add(new TrailingSlash(true)); // true adds the trailing slash (false removes it)
```
