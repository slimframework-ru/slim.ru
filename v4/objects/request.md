---
title: Запрос
---

Маршруты и промежуточное ПО вашего приложения Slim получают объект запроса PSR-7, 
который представляет текущий HTTP-запрос, полученный вашим веб-сервером. Объект запроса реализует 
[PSR-7 ServerRequestInterface](https://www.php-fig.org/psr/psr-7/#321-psrhttpmessageserverrequestinterface), 
с помощью которого вы можете проверять и обрабатывать метод HTTP-запроса, заголовки и тело.

## Как получить объект запроса

Объект запроса PSR-7 внедряется в маршруты вашего приложения Slim в качестве первого аргумента обработчика, например:

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->get('/hello', function (Request $request, Response $response) {
    $response->getBody()->write('Hello World');
    return $response;
});

$app->run();
```

Объект запроса PSR-7 внедряется в _промежуточное ПО_ вашего приложения Slim в качестве первого аргумента обработчика промежуточного ПО, например:

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Server\RequestHandlerInterface as RequestHandler;
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->add(function (ServerRequestInterface $request, RequestHandler $handler) {
   return $handler->handle($request);
});

// ...определение маршрутов приложения...

$app->run();
```

<div id="the-request-method"></div>

## Метод запроса

Каждый HTTP-запрос включает метод. Обычно, один из:

* GET
* POST
* PUT
* DELETE
* HEAD
* PATCH
* OPTIONS

Вы можете проверить метод HTTP-запроса с помощью метода `getMethod()` объекта запроса.

```php
$method = $request->getMethod();
```

Можно подделать или _переопределить_ метод HTTP-запроса. Это полезно, если, например, вам нужно имитировать запрос `PUT`, используя традиционный веб-браузер, который поддерживает только запросы` GET` или `POST`.

<div class="alert alert-info">
    <div><strong>Предостережение!</strong></div>
    Для включения переопределения метода необходимо включить в приложении <a href="/v4/middleware/method-overriding">промежуточное ПО переопределения метода запроса</a>.
</div>

Есть два способа переопределить метод HTTP-запроса.

Вы можете включить параметр `_METHOD` в тело запроса `POST`. HTTP-запрос должен использовать тип содержимого `application/x-www-form-urlencoded`.

```text
POST /path HTTP/1.1
Host: example.com
Content-type: application/x-www-form-urlencoded
Content-length: 22

data=value&_METHOD=PUT
```

Так же вы можете переопределить метод запроса, передав нужный метод в заголовке запроса `X-Http-Method-Override`. Это сработает с любым типом содержимого запроса.

```text
POST /path HTTP/1.1
Host: example.com
Content-type: application/json
Content-length: 16
X-Http-Method-Override: PUT

{"data":"value"}
```

<div id="the-request-uri"></div>

## URI запроса

Каждый HTTP-запрос имеет URI, который идентифицирует запрошенный ресурс приложения. URI HTTP-запроса состоит из нескольких частей:

* Scheme (e.g. `http` or `https`)
* Host (e.g. `example.com`)
* Port (e.g. `80` or `443`)
* Path (e.g. `/users/1`)
* Query string (e.g. `sort=created&dir=asc`)

Вы можете извлечь объект [URI object](https://www.php-fig.org/psr/psr-7/#35-psrhttpmessageuriinterface) с помощью метода `getUri()` объекта запроса:

```php
$uri = $request->getUri();
```

объект URI запроса предоставляет следующие методы для проверки частей URL-адреса HTTP-запроса:

* getScheme()
* getAuthority()
* getUserInfo()
* getHost()
* getPort()
* getPath()
* getBasePath()
* getQuery() <small>(возвращает полную строку параметров запроса, например `a=1&b=2`)</small>
* getFragment()
* getBaseUrl()

Вы можете получить ассоциативный массив параметров запроса с помощью метода `getQueryParams()` объекта запроса.

<div class="alert alert-info">
    <div><strong>Базовый путь</strong></div>
    Если фронт-контроллер вашего приложения Slim находится в физическом подкаталоге ниже корневого каталога домена,
    вы можете получить физический базовый путь HTTP-запроса (относительно корня домена) 
    с помощью метода <code>getBasePath()</code> объекта Uri. 
    Если приложение Slim установлено в корне домена, данный метод вернет пустую строку.
</div>

<div id="the-request-headers"></div>

## Заголовки запроса

Каждый HTTP-запрос имеет заголовки. Это метаданные, которые описывают HTTP-запрос, но не видны в теле запроса. 
Объект запроса Slim PSR-7 предоставляет несколько методов для проверки его заголовков.

### Получение всех заголовков

Вы можете извлечь все заголовки HTTP-запроса в виде ассоциативного массива с помощью метода `getHeaders()` объекта запроса PSR-7. 
Ключи результирующего ассоциативного массива являются именами заголовков, а его значения сами являются массивом строковых значений для соответствующего имени заголовка.

```php
$headers = $request->getHeaders();
foreach ($headers as $name => $values) {
    echo $name . ": " . implode(", ", $values);
}
```

### Получение одного заголовка

Вы можете получить значения одного заголовка с помощью метода `getHeader($name)` объекта запроса PSR-7. 
Метод возвращает массив значений для данного имени заголовка. 
Помните, что _один заголовок HTTP может иметь более одного значения!_

```php
$headerValueArray = $request->getHeader('Accept');
```

Вы также можете получить разделенную запятыми строку со всеми значениями для данного заголовка с помощью метода `getHeaderLine($name)` объекта запроса PSR-7. В отличие от метода `getHeader($name)`, этот метод возвращает строку объединенных через запятую значений.

```php
$headerValueString = $request->getHeaderLine('Accept');
```

### Определение наличия заголовка

Вы можете проверить наличие заголовка с помощью метода `hasHeader($name)` объекта запроса PSR-7.

```php
if ($request->hasHeader('Accept')) {
    // Do something
}
```

<div id="the-request-body"></div>

## Тело запроса

Каждый HTTP-запрос имеет тело. Если вы создаете приложение Slim, которое использует данные JSON или XML, 
вы можете использовать метод `getParsedBody()` объекта запроса PSR-7 для получения десериализованного тела запроса.
Обратите внимание, что синтаксический анализ тела может отличаться в разных реализациях PSR-7.

Возможно, вам потребуется реализовать промежуточное ПО для анализа тела запроса в зависимости от выбранной вами реализации PSR-7.
Вот пример для десериализации тела запроса в формате `JSON`:

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface as RequestHandler;

class JsonBodyParserMiddleware implements MiddlewareInterface
{
    public function process(Request $request, RequestHandler $handler): Response
    {
        $contentType = $request->getHeaderLine('Content-Type');

        if (strstr($contentType, 'application/json')) {
            $contents = json_decode(file_get_contents('php://input'), true);
            if (json_last_error() === JSON_ERROR_NONE) {
                $request = $request->withParsedBody($contents);
            }
        }

        return $handler->handle($request);
    }
}
```

```php
$parsedBody = $request->getParsedBody();
```

Технически, объект запроса PSR-7 представляет тело HTTP-запроса как экземпляр `Psr\Http\Message\StreamInterface`. 
Вы можете получить экземпляр HTTP-запроса StreamInterface с помощью метода `getBody()` объекта запроса PSR-7.
Метод `getBody()` предпочтителен, если размер входящего HTTP-запроса неизвестен или слишком велик для доступной памяти.

```php
$body = $request->getBody();
```

Результирующий экземпляр `Psr\Http\Message\StreamInterface` предоставляет следующие методы для чтения и итерации своего базового PHP-ресурса:

* getSize()
* tell()
* eof()
* isSeekable()
* seek()
* rewind()
* isWritable()
* write($string)
* isReadable()
* read($length)
* getContents()
* getMetadata($key = null)

<div id="uploaded-files"></div>

## Загруженные файлы

Список загруженных файлы в `$ _FILES` доступен из метода `getUploadedFiles()` объекта Request. Метод возвращает массив с ключом по имени элемента `input`.

```php
$files = $request->getUploadedFiles();
```

Каждый элемент массива `$files` является экземпляром `Psr\Http\Message\UploadedFileInterface` и поддерживает методы:

* getStream()
* moveTo($targetPath)
* getSize()
* getError()
* getClientFilename()
* getClientMediaType()

См в [книге рецептов](/v4/cookbook/uploading-files), как загружать файлы с помощью POST-формы.

<div id="request-helpers"></div>

## Помощники запроса

Реализация запроса Slim в PSR-7 предоставляет дополнительные методы, которые помогут вам работать с HTTP-запросом.

### Определение XHR-запроса

Вы можете определить XHR-запрос, проверив значение `XMLHttpRequest` в заголовке `X-Requested-With`.

```text
POST /path HTTP/1.1
Host: example.com
Content-type: application/x-www-form-urlencoded
Content-length: 7
X-Requested-With: XMLHttpRequest

foo=bar
```

```php
if ($request->getHeaderLine('X-Requested-With') === 'XMLHttpRequest') {
    // Do something
}
```

### Тип содержимого

Вы можете узнать тип содержимого запроса с помощью метода `getHeaderLine()` объекта запроса.

```php
$contentType = $request->getHeaderLine('Content-Type');
```

### Размер содержимого

Вы можете узнать размер содержимого запроса с помощью метода `getHeaderLine()` объекта запроса.

```php
$length = $request->getHeaderLine('Content-Length');
```

### Параметры запроса

Получить значение одного параметра можно с помощью метода `getServerParams()`

```php
$params = $request->getServerParams();
$authorization = isset($params['HTTP_AUTHORIZATION']) ? $params['HTTP_AUTHORIZATION'] : null;
```

<div id="route-object"></div>

## параметры маршрута

Иногда в промежуточном ПО требуется параметр вашего маршрута.

В этом примере мы сначала проверяем, что пользователь вошел в систему, а затем, что у пользователя есть разрешения на просмотр конкретного видео, которое он пытается просмотреть.

```php
<?php
$app
  ->get('/course/{id}', Video::class.":watch")
  ->add(PermissionMiddleware::class);
```

```php
<?php
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Server\RequestHandlerInterface as RequestHandler;
use Slim\Routing\RouteContext;

class PermissionMiddleware {
    public function __invoke(Request $request, RequestHandler $handler) {
        $routeContext = RouteContext::fromRequest($request);
        $route = $routeContext->getRoute();
        
        $courseId = $route->getArgument('id');
        
        // ...логика проверки доступа...
        
        return $handler->handle($request);
    }
}
```

## Получение базового пути в обработчике маршрута

Чтобы получить базовый путь в обработчике маршрута, просто сделайте следующее:

```php
<?php
use Slim\Factory\AppFactory;
use Slim\Routing\RouteContext;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->get('/', function($request, $response) {
    $routeContext = RouteContext::fromRequest($request);
    $basePath = $routeContext->getBasePath();
    
    // ...
    
    return $response;
});
```

## Атрибуты запроса

С PSR-7 возможно ввести объекты-значения в объект запроса для дальнейшей обработки. В ваших приложениях промежуточному ПО часто требуется передавать какую-то информацию обработчику маршрута, и хорошим способом сделать это является добавление этой информации в атрибутах запроса.

```php
$app->add(function ($request, $handler) {
    // добавление хранилища сессии в запрос
    $request = $request->withAttribute('session', $_SESSION);
    return $handler->handle($request);
});
```

```php
$app->get('/test', function ($request, $response, $args) {
    $session = $request->getAttribute('session'); // получение хранилища сессии из запроса
    return $response->write('Yay, ' . $session['name']);
});
```
