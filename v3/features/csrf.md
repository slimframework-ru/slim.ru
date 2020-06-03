---
title: Защита CSRF
---

В Slim 3 используется дополнительный отдельный компонент PHP [slimphp/Slim-Csrf](https://github.com/slimphp/Slim-Csrf)
 для защиты вашего приложения от CSRF ( подпрограмма подбора сайтов). Этот компонент генерирует уникальный токен для 
каждого запроса, который проверяет последующие запросы POST из клиентских HTML-форм.

## Установка

Выполните эту команду bash из корневого каталога вашего проекта:

```
composer require slim/csrf
```

## Использование

Компонент `slimphp/Slim-Csrf` содержит middleware. Добавьте его в свое приложение следующим образом:

```php
// Добавление middleware в приложение
$app = new \Slim\App;
$app->add(new \Slim\Csrf\Guard);

// Create your application routes...

// Run application
$app->run();
```

## Получить имя и значение токена CSRF

Последнее имя и значение токена CSRF доступны как атрибуты объекта запроса PSR7. Имя и значение токена 
CSRF уникальны для каждого запроса. Вы можете выбрать текущее имя и значение токена CSRF, как это.

```php
$app->get('/foo', function ($req, $res, $args) {
    // Имя и значение токена CSRF
    $nameKey = $this->csrf->getTokenNameKey();
    $valueKey = $this->csrf->getTokenValueKey();
    $name = $req->getAttribute($nameKey);
    $value = $req->getAttribute($valueKey);

    // Форма отображения HTML, какие должности /бар с двумя скрытыми полями ввода для
    // имя и значение:
    // <input type="hidden" name="<?= $nameKey ?>" value="<?= $name ?>">
    // <input type="hidden" name="<?= $valueKey ?>" value="<?= $value ?>">
});

$app->post('/bar', function ($req, $res, $args) {
    // Защита CSRF успешно, если вы достигли так далеко.
});
```

Вы должны передать имя и значение токена CSRF в шаблон, чтобы они могли быть отправлены с 
запросами POST в форме HTML. Они часто хранятся как скрытое поле с HTML-формами.

Для получения дополнительных примеров использования и документации, пожалуйста, 
проверьте страницу [slimphp/Slim-Csrf](https://github.com/slimphp/Slim-Csrf).
