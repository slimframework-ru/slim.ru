---
title: Action-Domain-Responder с Slim
---

Тут я покажу, как реорганизовать учебное приложение Slim, чтобы более внимательно следить за шаблоном 
[Action-Domain-Responder](http://pmjones.io/adr).

Одна приятная вещь о Slim (и большинстве других [HTTP пользовательского интерфейса фреймворка](http://paul-m-jones.com/archives/6627))) заключается в том, 
что они уже ориентированы на действия. То есть, их маршрутизаторы не предполагают класс контроллера со многими методами 
действий. Вместо этого они предполагают закрытие действия или однонаправленный вызывающий класс.

Таким образом, часть действия Action-Domain-Responder уже существует для Slim. Все, что необходимо, - вытащить 
посторонние биты из Actions, чтобы более четко разделить действия Action от Domain и поведения Responder.

## Extract Domain

Начнем с извлечения логики домена. В оригинальном учебном пособии, в действии, используются два источника данных, 
а также встроенная бизнес-логика. Мы можем создать класс Service Layer `TicketService` и переместить эти операции из 
Actions в Domain. Это дает нам этот класс:

```php
class TicketService
{
    protected $ticket_mapper;
    protected $component_mapper;

    public function __construct(
        TicketMapper $ticket_mapper,
        ComponentMapper $component_mapper
    ) {
        $this->ticket_mapper = $ticket_mapper;
        $this->component_mapper = $component_mapper;
    }

    public function getTickets()
    {
        return $this->ticket_mapper->getTickets();
    }

    public function getComponents()
    {
        return $this->component_mapper->getComponents();
    }

    public function getTicketById($ticket_id)
    {
        $ticket_id = (int) $ticket_id;
        return $this->ticket_mapper->getTicketById($ticket_id);
    }

    public function createTicket($data)
    {
        $component_id = (int) $data['component'];
        $component = $this->component_mapper->getComponentById($component_id);

        $ticket_data = [];
        $ticket_data['title'] = filter_var($data['title'], FILTER_SANITIZE_STRING);
        $ticket_data['description'] = filter_var($data['description'], FILTER_SANITIZE_STRING);
        $ticket_data['component'] = $component->getName();

        $ticket = new TicketEntity($ticket_data);
        $this->ticket_mapper->save($ticket);
        return $ticket;
    }
}
```

Мы создаем для него контейнерный объект `index.php`:

```php
$container['ticket_service'] = function ($c) {
    return new TicketService(
        new TicketMapper($c['db']),
        new ComponentMapper($c['db'])
    );
};
```
И теперь Actions может использовать `TicketService` вместо непосредственного выполнения логики домена:

```php
$app->get('/tickets', function (Request $request, Response $response) {
    $this->logger->addInfo("Ticket list");
    $tickets = $this->ticket_service->getTickets();
    $response = $this->view->render(
        $response,
        "tickets.phtml",
        ["tickets" => $tickets, "router" => $this->router]
    );
    return $response;
});

$app->get('/ticket/new', function (Request $request, Response $response) {
    $components = $this->ticket_service->getComponents();
    $response = $this->view->render(
        $response,
        "ticketadd.phtml",
        ["components" => $components]
    );
    return $response;
});

$app->post('/ticket/new', function (Request $request, Response $response) {
    $data = $request->getParsedBody();
    $this->ticket_service->createTicket($data);
    $response = $response->withRedirect("/tickets");
    return $response;
});

$app->get('/ticket/{id}', function (Request $request, Response $response, $args) {
    $ticket = $this->ticket_service->getTicketById($args['id']);
    $response = $this->view->render(
        $response,
        "ticketdetail.phtml",
        ["ticket" => $ticket]
    );
    return $response;
})->setName('ticket-detail');
```

Одно из преимуществ заключается в том, что мы теперь можем тестировать действия домена отдельно от действий. 
Мы можем начать делать нечто большее, как интеграционное тестирование, даже модульное тестирование, а не сквозное 
тестирование системы.

## Извлечение ответчика

В случае с учебным приложением работа с презентацией настолько проста, что для каждого действия не требуется 
отдельный ответчик. Расслабленная вариация слоя ответчика идеально подходит в этом простом случае, где каждое 
действие использует другой метод для общего ответчика.

Извлечение работы презентации отдельному ответчику, так что создание ответа полностью удаляется из Action, выглядит так:

```php
use Psr\Http\Message\ResponseInterface as Response;
use Slim\Views\PhpRenderer;

class TicketResponder
{
    protected $view;

    public function __construct(PhpRenderer $view)
    {
        $this->view = $view;
    }

    public function index(Response $response, array $data)
    {
        return $this->view->render(
            $response,
            "tickets.phtml",
            $data
        );
    }

    public function detail(Response $response, array $data)
    {
        return $this->view->render(
            $response,
            "ticketdetail.phtml",
            $data
        );
    }

    public function add(Response $response, array $data)
    {
        return $this->view->render(
            $response,
            "ticketadd.phtml",
            $data
        );
    }

    public function create(Response $response)
    {
        return $response->withRedirect("/tickets");
    }
}
```

Затем мы можем добавить `TicketResponder` объект в контейнер `index.php` :

```php
$container['ticket_responder'] = function ($c) {
    return new TicketResponder($c['view']);
};
```

И, наконец, мы можем ссылаться на ответчика, а не только на систему шаблонов, в Actions:

```php
$app->get('/tickets', function (Request $request, Response $response) {
    $this->logger->addInfo("Ticket list");
    $tickets = $this->ticket_service->getTickets();
    return $this->ticket_responder->index(
        $response,
        ["tickets" => $tickets, "router" => $this->router]
    );
});

$app->get('/ticket/new', function (Request $request, Response $response) {
    $components = $this->ticket_service->getComponents();
    return $this->ticket_responder->add(
        $response,
        ["components" => $components]
    );
});

$app->post('/ticket/new', function (Request $request, Response $response) {
    $data = $request->getParsedBody();
    $this->ticket_service->createTicket($data);
    return $this->ticket_responder->create($response);
});

$app->get('/ticket/{id}', function (Request $request, Response $response, $args) {
    $ticket = $this->ticket_service->getTicketById($args['id']);
    return $this->ticket_responder->detail(
        $response,
        ["ticket" => $ticket]
    );
})->setName('ticket-detail');
```

Теперь мы можем протестировать работу по созданию ответа отдельно от работы в домене.

Некоторые примечания:

Помещение всего здания ответа в одном классе с несколькими методами, особенно для простых случаев, таких как этот 
учебник, отлично подходит для начала. Для *этого* не обязательно иметь одного ответчика для каждого действия. Что это 
необходимо, чтобы извлечь ответ потенциал проблемы выхода из действий.
Putting all the response-building in a single class with multiple methods, especially for simple cases like this 
tutorial, is fine to start with. For ADR, is not strictly necessary to have one Responder for each Action. What *is* 
necessary is to extract the response-building concerns out of the Action.

Но по мере того как сложность логики представления возрастает (заголовки состояния согласования контента и т. Д.), 
И поскольку зависимости для каждого типа создаваемого ответа становятся разными, вам нужно иметь ответчика для каждого
 действия.

Кроме того, вы можете придерживаться одного ответчика, но уменьшите его интерфейс до одного метода. В этом случае 
вы можете обнаружить, что использование [полезной нагрузки домена](http://paul-m-jones.com/archives/6043) (вместо «голых» результатов домена) имеет некоторые 
существенные преимущества.

## Вывод

На данный момент приложение Slim tutorial было преобразовано в ADR. Мы разделили логику домена на a `TicketService` и 
логику представления на a `TicketResponder`. И легко понять, как каждое действие делает почти то же самое:

- Маршалы вводят и передают его в домен
- Возвращает результат из Домена и передает его ответчику
- Вызывает ответчика, чтобы он мог создавать и возвращать ответ

Теперь, для простого случая, подобного этому, использование ADR (или даже webbishy MVC) может показаться излишним. 
Но простые случаи быстро становятся сложными, и в этом простом случае показано, как разделение проблем ADR может 
быть применено как сложное приложение на основе Slim.
