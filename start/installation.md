---
title: Установка
---

## Системные Требования

* Веб-сервер с переписыванием URL-адресов
* PHP 5.5 или новее

## Как установить Slim

Мы рекомендуем установить Slim с [Composer](https://getcomposer.org/).
Перейдите в корневой каталог вашего проекта и выполните команду bash, показанную ниже. 
Эта команда загружает Slim Framework и ее сторонние зависимости в `vendor/` каталог вашего проекта.

<figure class="highlight"><pre>
<code class="language-bash" data-lang="bash">composer require slim/slim <span class="s2">"^3.0"</span></code>
</pre></figure>

Установите автозагрузчик Composer в свой PHP-скрипт, и вы готовы начать использовать Slim.

<figure class="highlight"><pre><code class="language-php" data-lang="php"><span class="cp">&lt;?php</span>
<span class="k">require</span> <span class="s1">'vendor/autoload.php'</span><span class="p">;</span></code></pre></figure>

## Как установить Composer

У вас нет Composer? Его легко установить, следуя инструкциям на странице [загрузки](https://getcomposer.org/download/).
