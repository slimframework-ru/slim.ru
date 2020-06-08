---
title: Веб серверы
---

Обычно шаблон фронт-контроллера используется для обработки всех полученных веб-сервером запросов одним файлом PHP.

## Встроенный в PHP сервер

Выполните следующую команду в терминале для запуска локального веб-сервера (предполагается, что файл `index.php` находится в директории `./public/`):

```bash
cd public/
php -S localhost:8888
```

Если вы используете не `index.php`, а какой-то другой файл, измените его соответствующим образом.

> **Предупреждение:** Встроенный в PHP веб сервер создан для облегчения разработки. 
Он может быть полезен в целях тестирования или демонстрации приложений, которые запускаются в контролируемых средах. Он не предназначен для полнофункционального веб-сервера. Он не должен использоваться в публичной сети.

## Конфигурация Apache

Убедитесь, что ваши файлы `.htaccess` и `index.php` находятся в одной публично доступной директории. Файл `.htaccess` должен содержать следующий код:

```apacheconfig
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.php [QSA,L]
```

Такой файл `.htaccess` требует перезаписи URL. Убедитесь, что модуль Apache mod_rewrite включен, и ваш виртуальный хост настроен с опцией `AllowOverride`, чтобы можно было использовать правила перезаписи:

```apacheconfig
AllowOverride All
```

## Конфигурация Nginx

Это пример конфигурации виртуального хоста Nginx для домена `example.com`.  
Он слушает входящие HTTP-соединения на 80 порту. В данном примере используется сервер PHP-FPM, слушающий порт 9000.
Вам нужно указать свои значения для директив `server_name`, `error_log`, `access_log` и `root` 

Директива `root` - это путь к общедоступному корню вашего приложения, в котором находится файл фронт-контроллера `index.php`.

```text
server {
    listen 80;
    server_name example.com;
    index index.php;
    error_log /path/to/example.error.log;
    access_log /path/to/example.access.log;
    root /path/to/public;

    location / {
        try_files $uri /index.php$is_args$args;
    }

    location ~ \.php {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param SCRIPT_NAME $fastcgi_script_name;
        fastcgi_index index.php;
        fastcgi_pass 127.0.0.1:9000;
    }
}
```

## Виртуальная машина HipHop

Файл конфигурации вашей виртуальной машины HipHop должен содержать этот код (наряду с другими необходимыми вам параметрами). Убедитесь, что вы изменили параметр `SourceRoot`, чтобы он указывал на корневой каталог документа приложения Slim.

```text
Server {
    SourceRoot = /path/to/public/directory
}

ServerVariables {
    SCRIPT_NAME = /index.php
}

VirtualHost {
    * {
        Pattern = .*
        RewriteRules {
            * {
                pattern = ^(.*)$
                to = index.php/$1
                qsa = true
            }
        }
    }
}
```

## IIS

Убедитесь, что файлы `Web.config` и` index.php` находятся в одном общедоступном каталоге. Файл `Web.config` должен содержать следующий код:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <rewrite>
            <rules>
                <rule name="slim" patternSyntax="Wildcard">
                    <match url="*" />
                    <conditions>
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
                    </conditions>
                    <action type="Rewrite" url="index.php" />
                </rule>
            </rules>
        </rewrite>
    </system.webServer>
</configuration>
```

## lighttpd

Ваш конфигурационный файл lighttpd должен содержать этот код (наряду с другими настройками, которые могут вам понадобиться). Этот код требует lighttpd> = 1.4.24.

```text
url.rewrite-if-not-file = ("(.*)" => "/index.php/$0")
```

Предполагается, что `index.php` находится в корневой папке вашего проекта (www root)..

<div id='run-from-a-sub-directory'></div>

## Запуск из подкаталога

Если вы хотите запустить приложение Slim из подкаталога в корне вашего сервера вместо создания виртуального хоста, вы можете настроить приложение `$app->setBasePath('path-to-your-app')` сразу после `AppFactory::Create ()`.
Предполагая, что корневым каталогом вашего сервера является `/var/www/html/`, а путь к вашему Slim-приложению - `/var/www/html/my-slim-app`, вы можете установить базовый путь `$app->setBasePath('/my-slim-app')`.

```php
<?php
use Slim\Factory\AppFactory;
use Slim\Middleware\OutputBufferingMiddleware;
// ...
$app = AppFactory::create();
$app->setBasePath('/my-slim-app');
// ...
$app->run();
```