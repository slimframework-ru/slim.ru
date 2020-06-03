---
title: Веб-серверы
---

Типичное использование патерна front-controller для перебора соответствующих HTTP-запросов, 
полученных вашим веб-сервером, в один файл PHP. В приведенных ниже инструкциях объясняется, как сообщить веб-серверу 
отправлять HTTP-запросы в ваш файл front-controller PHP.

## Встроенный сервер PHP

Запустите следующую команду в терминале, чтобы запустить веб-сервер localhost, предполагая, что 
 `./public/` это общедоступный каталог с `index.php` файлом:

```bash
php -S localhost:8888 -t public public/index.php
```

Если вы не используете `index.php` свою точку входа, измените ее соответствующим образом.

## Конфигурация Apache

Убедитесь, что ваши файлы `.htaccess` и `index.php` файлы находятся в том же общедоступном каталоге. 
Файл `.htaccess`  должен содержать следующий код:

```apacheconfig
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.php [QSA,L]
```

Убедитесь, что ваш виртуальный хост Apache настроен с параметром `AllowOverride` 
чтобы `.htaccess` можно было использовать правила перезаписи:

```apacheconfig
AllowOverride All
```


## Конфигурация Nginx

Это пример конфигурации виртуального хоста Nginx для домена `example.com`.
Он слушает для входящих HTTP соединений на порт 80. Это предполагает сервер PHP-FPM работает на порту 9000. 
Вы должны обновить `server_name`, `error_log`,
`access_log`, и `root` директиву с вашими собственными значениями. Директива `root` путь к общественности документ 
корневой директории вашего приложения; ваш
`index.php` файл front-controller Slim должен находиться в этом каталоге.

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

Ваш файл конфигурации виртуальной машины HipHop должен содержать этот код (вместе с другими параметрами, которые могут 
вам понадобиться). Убедитесь, что вы изменили  `SourceRoot` настройку, чтобы указать на корневой каталог документа Slim app.

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

Убедитесь, что файлы `Web.config` и `index.php` файлы находятся в том же общедоступном каталоге.
 Файл `Web.config` должен содержать следующий код:


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

Ваш файл конфигурации lighttpd должен содержать этот код (вместе с другими параметрами, которые могут вам понадобиться). 
Этот код требует lighttpd> = 1.4.24

```text
url.rewrite-if-not-file = ("(.*)" => "/index.php/$0")
```

Это предполагает, что Slim  `index.php` находится в корневой папке вашего проекта (www root).
