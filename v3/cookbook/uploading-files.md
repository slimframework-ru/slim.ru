---
title: Загрузка файлов с использованием форм POST
---

Файлы, которые загружаются с использованием форм в POST-запросах, могут быть получены с помощью
[`getUploadedFiles`](/docs/objects/request.html#uploaded-files) метода `Request` объекта.

При загрузке файлов с использованием запроса POST убедитесь, что ваша форма загрузки файла имеет атрибут, 
`enctype="multipart/form-data"` иначе `getUploadedFiles()` будет возвращен пустой массив.

Если несколько файлов загружаются для одного имени ввода, добавьте скобки после имени ввода в HTML, иначе только 
один загруженный файл будет возвращен для имени ввода `getUploadedFiles()`.

Ниже приведен пример HTML-формы, содержащей как одиночную, так и множественную загрузку файлов.

```php
<!-- убедитесь, что enctype атрибут имеет значение multipart/form-data -->
<form method="post" enctype="multipart/form-data">
    <!-- загрузка одного файла -->
    <p>
        <label>Добавить файл (один): </label><br/>
        <input type="file" name="example1"/>
    </p>

    <!-- несколько полей ввода для одного и того же имени ввода, используйте квадратные скобки -->
    <p>
        <label>Добавить файлы (два): </label><br/>
        <input type="file" name="example2[]"/><br/>
        <input type="file" name="example2[]"/>
    </p>

    <!--одно поле ввода файла, позволяющее загружать несколько файлов, используйте квадратные скобки -->
    <p>
        <label>Добавить файлы (много): </label><br/>
        <input type="file" name="example3[]" multiple="multiple"/>
    </p>

    <p>
        <input type="submit"/>
    </p>
</form>
```
<figure>
<figcaption>Пример 1: Пример формы HTML для загрузки файлов</figcaption>
</figure>

Загруженные файлы можно перемещать в каталог с помощью `moveTo` метода. Ниже приведен пример приложения, который
обрабатывает загруженные файлы HTML-формы выше.

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';

use Slim\Http\Request;
use Slim\Http\Response;
use Slim\Http\UploadedFile;

$app = new \Slim\App();

$container = $app->getContainer();
$container['upload_directory'] = __DIR__ . '/uploads';

$app->post('/', function(Request $request, Response $response) {
    $directory = $this->get('upload_directory');

    $uploadedFiles = $request->getUploadedFiles();

    // обработчик одного input с одним файлом загрузки
    $uploadedFile = $uploadedFiles['example1'];
    if ($uploadedFile->getError() === UPLOAD_ERR_OK) {
        $filename = moveUploadedFile($directory, $uploadedFile);
        $response->write('uploaded ' . $filename . '<br/>');
    }


    // обработчик нескольких input с одним и тем же ключом
    foreach ($uploadedFiles['example2'] as $uploadedFile) {
        if ($uploadedFile->getError() === UPLOAD_ERR_OK) {
            $filename = moveUploadedFile($directory, $uploadedFile);
            $response->write('uploaded ' . $filename . '<br/>');
        }
    }

    // обработчик одного input с множественной загрузкой файлов
    foreach ($uploadedFiles['example3'] as $uploadedFile) {
        if ($uploadedFile->getError() === UPLOAD_ERR_OK) {
            $filename = moveUploadedFile($directory, $uploadedFile);
            $response->write('uploaded ' . $filename . '<br/>');
        }
    }
    
});

/**
 * Перемещает загруженный файл в каталог загрузки и назначает ему уникальное имя, 
 * чтобы избежать перезаписи существующего загруженного файла.
 *
 * @param string $directory каталог, в который перемещается файл
 * @param UploadedFile $uploadedFile загруженный файл для перемещения
 * @return string имя перемещенного файла
 */
function moveUploadedFile($directory, UploadedFile $uploadedFile)
{
    $extension = pathinfo($uploadedFile->getClientFilename(), PATHINFO_EXTENSION);
    $basename = bin2hex(random_bytes(8)); // see http://php.net/manual/en/function.random-bytes.php
    $filename = sprintf('%s.%0.8s', $basename, $extension);

    $uploadedFile->moveTo($directory . DIRECTORY_SEPARATOR . $filename);

    return $filename;
}

$app->run();
```
<figure>
<figcaption>Пример 2: Пример Slim application для обработки загруженных файлов</figcaption>
</figure>

Смотрите также
--------
* [https://akrabat.com/psr-7-file-uploads-in-slim-3/](https://akrabat.com/psr-7-file-uploads-in-slim-3/)
