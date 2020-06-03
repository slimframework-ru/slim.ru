---
title: Получение IP-адреса
---

Лучший способ получить текущий IP-адрес клиента - через middleware с использованием 
такого компонента, как [rka-ip-address-middleware](https://github.com/akrabat/rka-ip-address-middleware).

Этот компонент может быть установлен через composer:

```bash
composer require akrabat/rka-ip-address-middleware
```

To use it, register the middleware with the <code>App</code>, providing a list
of trusted proxies (e.g. varnish servers) if you are using them.:

```php
$checkProxyHeaders = true;
$trustedProxies = ['10.0.0.1', '10.0.0.2'];
$app->add(new RKA\Middleware\IpAddress($checkProxyHeaders, $trustedProxies));

$app->get('/', function ($request, $response, $args) {
    $ipAddress = $request->getAttribute('ip_address');

    return $response;
});
```

Middleware хранит IP-адрес клиента в атрибуте запроса, поэтому доступ осуществляется через 
`$request->getAttribute('ip_address')`.