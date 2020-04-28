---
title: Кэширование HTTP
---

В Slim 3 используется дополнительный компонент PHP [slimphp/Slim-HttpCache](https://github.com/slimphp/Slim-HttpCache) 
для HTTP-кеширования. Вы можете использовать этот компонент для создания и возвращать ответы , которые содержат 
`Cache`, `Expires`, `ETag` и `Last-Modified` заголовки, которые управляют временем хранения выходных 
данных приложения, кэшами на стороне клиента.  Возможно, вам нужно установить для параметра php.ini 
«session.cache_limiter» пустую строку.

## Установка

Выполните эту команду bash из корневого каталога вашего проекта:

```
composer require slim/http-cache
```

## Применение

Компонент `slimphp/Slim-HttpCache` содержит поставщик услуг и middleware. 
Вы должны добавить оба приложения в это приложение:

```php
// Регистрация поставщика услуг с контейнером
$container = new \Slim\Container;
$container['cache'] = function () {
    return new \Slim\HttpCache\CacheProvider();
};

// Добавить middleware к приложению
$app = new \Slim\App($container);
$app->add(new \Slim\HttpCache\Cache('public', 86400));

// Создайте свой маршрутов приложение...

// Запуск приложения
$app->run();
```

## ETag

Используйте метод поставщика услуг `withEtag()` для создания объекта Response с нужным `ETag` заголовком. 
Этот метод принимает объект ответа PSR7 и возвращает клонированный ответ PSR7 с новым HTTP-заголовком.

```php
$app->get('/foo', function ($req, $res, $args) {
    $resWithEtag = $this->cache->withEtag($res, 'abc');

    return $resWithEtag;
});
```

## Expires

Используйте метод поставщика услуг `withExpires()` для создания объекта Response с нужным `Expires` заголовком. 
Этот метод принимает объект ответа PSR7 и возвращает клонированный ответ PSR7 с новым HTTP-заголовком.

```php
$app->get('/bar',function ($req, $res, $args) {
    $resWithExpires = $this->cache->withExpires($res, time() + 3600);

    return $resWithExpires;
});
```

## Last-Modified

Используйте метод поставщика услуг `withLastModified()` для создания объекта Response с нужным `Last-Modified` 
заголовком. 
Этот метод принимает объект ответа PSR7 и возвращает клонированный ответ PSR7 с новым HTTP-заголовком.

```php
//Пример маршрута с помощью LastModified
$app->get('/foobar',function ($req, $res, $args) {
    $resWithLastMod = $this->cache->withLastModified($res, time() - 3600);

    return $resWithLastMod;
});
```
