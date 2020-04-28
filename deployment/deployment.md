---
title: Развертывание
---
Поздравления! если вы сделали это так далеко, это означает, что вы успешно создали что-то потрясающее, используя Slim. 
Однако время для вечеринки еще не наступило. 
Нам все равно придется подталкивать наше приложение на производственный сервер.

Существует много способов сделать это, выходящие за рамки этой документации. 
В этом разделе мы приводим несколько заметок для различных настроек.

### Отключить отображение ошибок в процессе производства

Первое, что нужно сделать, это настроить настройки  (`src/settings.php` в приложении скелета) 
и убедиться, что вы не отображаете полную информацию об ошибке для публики.

<figure class="highlight"><pre><code class="language-php" data-lang="php">  'displayErrorDetails' =&gt; false, // set to false in production</code></pre></figure>

Вы также должны убедиться, что ваша установка PHP настроена так, чтобы не отображать ошибки с `php.ini` настройкой:

<figure class="highlight"><pre><code class="language-ini" data-lang="ini">
<span class="py">display_errors</span> <span class="p">= </span><span class="s"> 0</span></code></pre>
</figure>


## Развертывание на собственный сервер

Если вы управляете своим сервером, вам следует настроить процесс развертывания с использованием 
любой из многих систем развертывания, таких как:

* Deploybot
* Capistrano
* Сценарий, управляемый Phing, Make, Ant, etc.


Просмотрите документацию [Web Servers](/docs/start/web-servers.html) для настройки вашего веб-сервера.


## Развертывание на общий сервер

Если ваш общий сервер работает Apache, то вам нужно создать`.htaccess` файл в веб - сервера корневой каталог
 (обычно называется `htdocs`, `public`, `public_html` или `www`) со следующим содержанием:

<figure class="highlight"><pre><code class="language-apache" data-lang="apache"><span class="p">&lt;</span><span class="nl">IfModule</span><span class="sr"> mod_rewrite.c</span><span class="p">&gt;
</span>   <span class="nc">RewriteEngine</span> <span class="ss">on</span>
   <span class="nc">RewriteRule</span> ^$ public/     [L]
   <span class="nc">RewriteRule</span> (.*) public/$1 [L]
<span class="p">&lt;/</span><span class="nl">IfModule</span><span class="p">&gt;</span></code></pre></figure>

(замените `public` на правильное имя)

Теперь загрузите все файлы, составляющие ваш проект Slim, на веб-сервер. Поскольку вы находитесь на общем хостинге, 
это, вероятно, выполняется через FTP, и вы можете использовать любой FTP-клиент, такой как Filezilla, для этого.