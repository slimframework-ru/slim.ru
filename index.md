---
title: Документация
---

<div class="alert alert-info">
    <p>
        Эта документация предназначена для <strong>Slim 3</strong>. Документацию по Slim 2 можно найти на <a href="http://docs.slimframework.com/">docs.slimframework.com</a>.
    </p>
</div>

<p style="text-align: center;">
    <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">
        <img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" />
    </a>
    <br />
    Эта работа лицензируется в соответствии с  <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License</a>.
</p>

## Добро пожаловать

Slim - это микроструктура PHP, которая помогает быстро писать простые, но мощные веб-приложения и API. 
По сути, Slim - диспетчер, который получает HTTP-запрос, вызывает соответствующую процедуру обратного вызова и
 возвращает HTTP-ответ. Вот и все.

## В чем смысл?

Slim - идеальный инструмент для создания API-интерфейсов, которые потребляют, перенастраивают или публикуют данные. 
Slim - отличный инструмент для быстрого прототипирования. Черт, вы даже можете создавать полнофункциональные 
веб-приложения с пользовательскими интерфейсами. Что еще более важно, Slim очень быстрый и имеет очень мало кода. 
Фактически, вы можете читать и понимать его исходный код `только днем` (!in only an afternoon!)

> По сути, Slim - диспетчер, который получает HTTP-запрос, вызывает 
соответствующую процедуру обратного вызова и 
возвращает HTTP-ответ. Вот и все.

Вам не всегда нужно `kitchen-sink` решение , например, [Symfony][symfony] или [Laravel][laravel].
Это отличные инструменты, конечно. Но они часто переполняются. Вместо этого Slim предоставляет только 
минимальный набор инструментов, которые делают то, что вам нужно, и ничего больше.

## Как это работает?

Во-первых, вам нужен веб-сервер, такой как Nginx или Apache. Вы должны [настроить свой веб-сервер](/docs/start/web-servers.html)
 так, чтобы он отправлял все соответствующие запросы в один файл PHP «front-controller». Вы создаете экземпляр и 
 запускаете свое приложение Slim в этом файле PHP.

Приложение Slim содержит маршруты, отвечающие определенным HTTP-запросам. Каждый маршрут вызывает обратный 
вызов и возвращает HTTP-ответ. Чтобы начать работу, сначала создайте экземпляр и настройте приложение Slim. 
Затем вы определяете маршруты своего приложения. Наконец, вы запускаете приложение Slim. Это так просто. 
Вот пример приложения:

<figure>

  <figure class="highlight"><pre><code class="language-php" data-lang="php"><span class="cp">&lt;?php</span>
<span class="c1">// Create and configure Slim app
</span><span class="nv">$config</span> <span class="o">=</span> <span class="p">[</span><span
                class="s1">'settings'</span> <span class="o">=&gt;</span> <span class="p">[</span>
    <span class="s1">'addContentLengthHeader'</span> <span class="o">=&gt;</span> <span class="kc">false</span><span
                class="p">,</span>
<span class="p">]];</span>
<span class="nv">$app</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">\Slim\App</span><span
                class="p">(</span><span class="nv">$config</span><span class="p">);</span>

<span class="c1">// Define app routes
</span><span class="nv">$app</span><span class="o">-&gt;</span><span class="na">get</span><span class="p">(</span><span
                class="s1">'/hello/{name}'</span><span class="p">,</span> <span class="k">function</span> <span
                class="p">(</span><span class="nv">$request</span><span class="p">,</span> <span
                class="nv">$response</span><span
                class="p">,</span> <span class="nv">$args</span><span class="p">)</span> <span
                class="p">{</span>
    <span class="k">return</span> <span class="nv">$response</span><span class="o">-&gt;</span><span
                class="na">write</span><span class="p">(</span><span class="s2">"Hello "</span> <span
                class="o">.</span> <span class="nv">$args</span><span class="p">[</span><span
                class="s1">'name'</span><span class="p">]);</span>
<span class="p">});</span>

<span class="c1">// Run app
</span><span class="nv">$app</span><span class="o">-&gt;</span><span class="na">run</span><span
                class="p">();</span></code></pre>
  </figure>

  <figcaption>Пример 1: Пример Slim application</figcaption>
</figure>

## Запрос и ответ

Когда вы создаете Slim-приложение, вы часто работаете непосредственно с объектами Request and Response. 
Эти объекты представляют собой фактический HTTP-запрос, полученный веб-сервером, и возможный HTTP-ответ, 
возвращаемый клиенту.

Каждому маршруту Slim-приложения присваиваются текущие объекты Request and Response в качестве аргументов 
его подпрограммы обратного вызова. Эти объекты реализуют популярные интерфейсы [PSR 7](/docs/concepts/value-objects.html). 
Slim route приложения может проверять или манипулировать этими объектами по мере необходимости. 
 В конечном счете, каждый маршрут приложения Slim **MUST** возвращать ответ PSR 7  объекта.

## Принесите свои собственные компоненты

Slim также хорошо сочетается с другими компонентами PHP. Вы можете зарегистрировать дополнительные сторонние 
компоненты, такие как [Slim-Csrf][csrf], [Slim-HttpCache][httpcache],
или [Slim-Flash][flash] которые основываются на функциональных возможностях Slim по умолчанию. 
Также легко интегрировать сторонние компоненты, найденные в [Packagist](https://packagist.org/).

## Как читать эту документацию

Если вы новичок в Slim, я рекомендую вам прочитать эту документацию от начала до конца. 
Если вы уже знакомы с Slim, вы можете перейти прямо в соответствующий раздел.

Эта документация начинается с объяснения концепций и архитектуры Slim, прежде чем вникать в конкретные темы, 
такие как обработка запросов и ответов, маршрутизация и обработка ошибок.

[symfony]: http://symfony.com/
[laravel]: http://laravel.com/
[csrf]: https://github.com/slimphp/Slim-Csrf/
[httpcache]: https://github.com/slimphp/Slim-HttpCache
[flash]: https://github.com/slimphp/Slim-Flash
[eloquent]: http://laravel.com/docs/5.1/eloquent
[doctrine]: http://www.doctrine-project.org/projects/orm.html