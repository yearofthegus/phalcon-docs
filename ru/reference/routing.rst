Маршрутизация (Routing, Роутинг)
================================
Компонент маршрутизации позволяет определять маршруты, которые будут привязаны к контроллерам, или обработчикам для получения
запроса. Маршрутизатор просто разбирает URI для определения информации. Маршрутизатор имеет два режима: MVC
режим и режим совпадения. Первый режим идеально подходит для работы с MVC приложениями.

Определение маршрутов
---------------------
:doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>` предоставляет расширенные возможности маршрутизации. В MVC режиме вы
можете определить маршруты и направить их на контроллеры/действия, которые вам требуются. Маршруты определяются следующим образом:

.. code-block:: php

    <?php

    // Создание маршрутизатора
    $router = new \Phalcon\Mvc\Router();

    // Определение правила маршрутизации
    $router->add(
        "/admin/users/my-profile",
        array(
            "controller" => "users",
            "action"     => "profile",
        )
    );

    // Еще одно правило
    $router->add(
        "/admin/users/change-password",
        array(
            "controller" => "users",
            "action"     => "changePassword",
        )
    );

    $router->handle();

Метод add() принимает в качестве первого параметра шаблон ссылки, вторым параметром настройки этого маршрута.
В этом случае, если URI соответствует /admin/users/my-profile, и будет выполнен контроллер "users", а в нём действие "profile".
Маршрутизатор не выполняет действие контроллера, он только собирает эту информацию, чтобы сообщить правильные параметры в компонент
:doc:`Phalcon\\Mvc\\Dispatcher <../api/Phalcon_Mvc_Dispatcher>`.

Приложение может иметь множество маршрутов, определения их по одному может быть достаточно трудоемкой задачей. В таких случаях мы можем
создавать более гибкие маршруты:

.. code-block:: php

    <?php

    // Создание маршрутизатора
    $router = new \Phalcon\Mvc\Router();

    // Определение правила маршрутизации
    $router->add(
        "/admin/:controller/a/:action/:params",
        array(
            "controller" => 1,
            "action"     => 2,
            "params"     => 3,
        )
    );

В примере, приведенном выше, с помощью подстановочных элементов мы делаем маршрут подходящим для множества ссылок. Например, при получении
URL (/admin/users/a/delete/dave/301), маршрутизатор разберёт его в:

+-------------------------+----------+
| Контроллер / Controller | users    |
+-------------------------+----------+
| Действие / Action       | delete   |
+-------------------------+----------+
| Параметр / Parameter    | dave     |
+-------------------------+----------+
| Параметр / Parameter    | 301      |
+-------------------------+----------+

Метод add() принимает шаблон, который по желанию может быть написан с использованием регулярных выражений. Все
маршруты должны начинаться с косой черты (/). Регулярные выражения должны соответствовать формату  `регулярных выражений PCRE`_.
Отметим, что это не нужно добавлять в регулярные выражения разделители. Все маршруты не зависят от регистра.

Второй параметр определяет, какие из подходящих частей следует "привязать" к controller/action/parameters. Соответствующие
части берутся из "заполнителей" или по маскам ограничивающимися круглыми скобками. В примере, приведенном выше,
первой части соответствует контроллеру (:controller), второй действию и так далее.

Заполнители помогают написанию регулярных выражений, они более читабельны для разработчиков и проще
для понимания. Существуют такие заполнители:

+--------------+-----------------------+--------------------------------------------------------------------------------------------------------+
| Placeholder  | Регулярное вырожение  | Использование                                                                                          |
+==============+=======================+========================================================================================================+
| /:module     | /([a-zA-Z0-9\_\-]+)   | Проверяет соответствие названия модуля алфавитно-цифровым символам                                     |
+--------------+-----------------------+--------------------------------------------------------------------------------------------------------+
| /:controller | /([a-zA-Z0-9\_\-]+)   | Проверяет соответствие названия контроллера алфавитно-цифровым символам                                |
+--------------+-----------------------+--------------------------------------------------------------------------------------------------------+
| /:action     | /([a-zA-Z0-9\_]+)     | Проверяет соответствие названия действия алфавитно-цифровым символам                                   |
+--------------+-----------------------+--------------------------------------------------------------------------------------------------------+
| /:params     | (/.*)*                | Проверяет список дополнительных частей, разделенных косыми чертами. Использовать только в конце ссылок |
+--------------+-----------------------+--------------------------------------------------------------------------------------------------------+
| /:namespace  | /([a-zA-Z0-9\_\-]+)   | Проверяет пространство имен                                                                            |
+--------------+-----------------------+--------------------------------------------------------------------------------------------------------+
| /:int        | /([0-9]+)             | Проверяет соответствие цифровому формату                                                               |
+--------------+-----------------------+--------------------------------------------------------------------------------------------------------+

Названия контроллеров "camelized", это означает, что символы (-) и (_) удаляются, и следующий после них символ
преобразуется в верхний регистр. Например, some_controller преобразуется в SomeController.

Поскольку вы можете использовать множество маршрутов, добавляя их методом add(), порядок, в котором маршруты добавляются указывает
их актуальность, последние добавленные маршруты имеют больший приоритет, чем добавленные ранее. Внутри все определенные маршруты
перемещаются в обратном порядке, пока :doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>` не найдёт
тот, который соответствует данному URI и использует его, игнорируя остальные.

Именованные параметры
^^^^^^^^^^^^^^^^^^^^^
В примере ниже показано, как определить имена для параметров маршрутов:

.. code-block:: php

    <?php

    $router->add(
        "/news/([0-9]{4})/([0-9]{2})/([0-9]{2})/:params",
        array(
            "controller" => "posts",
            "action"     => "show",
            "year"       => 1, // ([0-9]{4})
            "month"      => 2, // ([0-9]{2})
            "day"        => 3, // ([0-9]{2})
            "params"     => 4, // :params
        )
    );

В приведенном выше примере, в маршруте не определены части для "контроллера" или "действия". Эти параметры заменяются
фиксированными значениями ("posts" и "show"). Пользователь не будет видеть вызванный контроллер.
Внутри контроллера именованные параметры можно получить следующим образом:

.. code-block:: php

    <?php

    class PostsController extends \Phalcon\Mvc\Controller
    {

        public function indexAction()
        {

        }

        public function showAction()
        {

            // Возвращает параметр "year"
            $year = $this->dispatcher->getParam("year");

            // Возвращает параметр "month"
            $month = $this->dispatcher->getParam("month");

            // Возвращает параметр "day"
            $day = $this->dispatcher->getParam("day");

        }

    }

Обратите внимание, что значения параметров получаются из диспетчера. Это происходит потому, что это
компонент, который, непосредственно запускает в работу ваше приложение.
Кроме того, существует и другой способ создавать именованные параметры, например, как часть правила маршрутизации:

.. code-block:: php

    <?php

    $router->add(
        "/documentation/{chapter}/{name}.{type:[a-z]+}",
        array(
            "controller" => "documentation",
            "action"     => "show"
        )
    );

Вы можете получить доступ к их значениям так же, как раньше:

.. code-block:: php

    <?php

    class DocumentationController extends \Phalcon\Mvc\Controller
    {

        public function showAction()
        {

            // Возвращает параметр "name"
            $name = $this->dispatcher->getParam("name");

            // Возвращает параметр "type"
            $type = $this->dispatcher->getParam("type");

        }

    }

Краткий синтаксис
^^^^^^^^^^^^^^^^^
Если вам не нравится использование массивов для определения правил маршрута, альтернативный синтаксис также доступен.
Следующие примеры дают одинаковый результат:

.. code-block:: php

    <?php

    // Краткий синтаксис
    $router->add("/posts/{year:[0-9]+}/{title:[a-z\-]+}", "Posts::show");

    // Использование массива
    $router->add(
        "/posts/([0-9]+)/([a-z\-]+)",
        array(
           "controller" => "posts",
           "action"     => "show",
           "year"       => 1,
           "title"      => 2,
        )
    );

Совмещение массивов и краткого синтаксиса
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Массив и краткий синтаксис может быть смешанным, в данном случае, обратите внимание, что именованные параметры автоматически
добавляются в маршрут в соответствии с положением, на котором они были определены:

.. code-block:: php

    <?php

    // В качестве первой позиции выступает параметр 'country'
    $router->add('/news/{country:[a-z]{2}}/([a-z+])/([a-z\-+])',
        array(
            'section' => 2, // Это уже позиция номер 2
            'article' => 3
        )
    );

Маршрутизация модулей
^^^^^^^^^^^^^^^^^^^^^
Вы можете определить маршруты, пути которых включают в себя модули. Это особенно подходит для мульти-модульных приложений.
Возможно так же определить маршрут по умолчанию, который включает в себя модуль шаблона:

.. code-block:: php

    <?php

    $router = new Phalcon\Mvc\Router(false);

    $router->add('/:module/:controller/:action/:params', array(
        'module' => 1,
        'controller' => 2,
        'action' => 3,
        'params' => 4
    ));

В этом случае маршрут всегда должен иметь имя модуля в качестве части URL-адреса. Например, в следующем
URL: /admin/users/edit/sonny, будут обработан как:

+------------+---------------+
| Модуль     | admin         |
+------------+---------------+
| Контроллер | users         |
+------------+---------------+
| Действие   | edit          |
+------------+---------------+
| Параметр   | sonny         |
+------------+---------------+

Или вы можете привязать конкретные маршруты к конкретным модулям:

.. code-block:: php

    <?php

    $router->add("/login", array(
        'module' => 'backend',
        'controller' => 'login',
        'action' => 'index',
    ));

    $router->add("/products/:action", array(
        'module' => 'frontend',
        'controller' => 'products',
        'action' => 1,
    ));

Или привязать к конкретному пространству имен:

.. code-block:: php

    <?php

    $router->add("/:namespace/login", array(
        'namespace' => 1,
        'controller' => 'login',
        'action' => 'index'
    ));

Пространства имён и названия классов должны передаваться раздельно:

.. code-block:: php

    <?php

    $router->add("/login", array(
        'namespace' => 'Backend\Controllers',
        'controller' => 'login',
        'action' => 'index'
    ));

Разделение по HTTP методам
^^^^^^^^^^^^^^^^^^^^^^^^^^
При добавлении маршрута, используя метод add(), маршрут будет активен для любого HTTP-метода. Иногда можно использовать маршрут для
конкретного метода, это особенно полезно при создании RESTful приложений:

.. code-block:: php

    <?php

    // Маршрут соответствует только HTTP методу GET
    $router->addGet("/products/edit/{id}", "Products::edit");

    // Маршрут соответствует только HTTP методу POST
    $router->addPost("/products/save", "Products::save");

    // Маршрут соответствует сразу двум HTTP методам POST и PUT
    $router->add("/products/update")->via(array("POST", "PUT"));

Использование преобразований
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Метод convert позволяет трансформировать параметры маршрута до передачи их диспетчеру, следующий пример показывает вариант использования:

.. code-block:: php

    <?php

    // Название действия разрешает использование "-": /products/new-ipod-nano-4-generation
    $router
        ->add('/products/{slug:[a-z\-]+}', array(
            'controller' => 'products',
            'action' => 'show'
        ))
        ->convert('slug', function($slug) {
            // Удаляем тире из выбранного параметра
            return str_replace('-', '', $slug);
        });

Группы маршрутов
^^^^^^^^^^^^^^^^
Если наборы маршрутов имеют общие пути, они могут быть сгруппированы для легкой поддержки:

.. code-block:: php

    <?php

    $router = new \Phalcon\Mvc\Router();

    // Создаётся группа с общим модулем и контроллером
    $blog = new \Phalcon\Mvc\Router\Group(array(
        'module' => 'blog',
        'controller' => 'index'
    ));

    // Маршруты начинаются с /blog
    $blog->setPrefix('/blog');

    // Добавление маршрута в группу
    $blog->add('/save', array(
        'action' => 'save'
    ));

    // Еще один маршрут
    $blog->add('/edit/{id}', array(
        'action' => 'edit'
    ));

    // Маршрут для действия по умолчанию
    $blog->add('/blog', array(
        'controller' => 'blog',
        'action' => 'index'
    ));

    // Добавление группы в общие правила маршрутизации
    $router->mount($blog);

Вы можете размещать группы маршрутов в разных файлах приложения, добиваясь оптимальной структуры и чистоты кода:

.. code-block:: php

    <?php

    class BlogRoutes extends Phalcon\Mvc\Router\Group
    {
        public function initialize()
        {
            // Параметры по умолчанию
            $this->setPaths(array(
                'module' => 'blog',
                'namespace' => 'Blog\Controllers'
            ));

            // Маршруты начинаются с преффикса /blog
            $this->setPrefix('/blog');

            // Добавляем маршрут
            $this->add('/save', array(
                'action' => 'save'
            ));

            // Еще маршрут
            $this->add('/edit/{id}', array(
                'action' => 'edit'
            ));

            // Данные для маршрута по умолчанию
            $this->add('/blog', array(
                'controller' => 'blog',
                'action' => 'index'
            ));

        }
    }

Созданную группу надо подмонтировать в маршрутизатор:

.. code-block:: php

    <?php

    // Добавляем маршруты в общий маршрутизатор:
    $router->mount(new BlogRoutes());

Соответствие маршрутов
----------------------
Текущий URI передаётся маршрутизатору для сопоставления его маршруту. По умолчанию, URI для обработки берется из
переменной $_GET['_url'], полученной с использованием mod_rewrite.
Для Phalcon подходят очень простые правила mod_rewrite:

.. code-block:: apacheconf

    RewriteEngine On
    RewriteCond   %{REQUEST_FILENAME} !-d
    RewriteCond   %{REQUEST_FILENAME} !-f
    RewriteRule   ^(.*)$ index.php?_url=/$1 [QSA,L]

В следующем примере показано, как использовать этот компонент автономно:

.. code-block:: php

    <?php

    // Создание маршрутизатора
    $router = new \Phalcon\Mvc\Router();

    // Тут устанавливаются правила маршрутизации
    // ...

    // Будет использован $_GET["_url"]
    $router->handle();

    // Можно указать параметр самостоятельно
    $router->handle("/employees/edit/17");

    // Получаем выбранный контроллер
    echo $router->getControllerName();

    // .. и соответсвющее действие
    echo $router->getActionName();

    // Получаем сам выбранный для ссылки маршрут
    $route = $router->getMatchedRoute();

Именованные маршруты
--------------------
Каждый маршрут, добавленный в маршрутизатор, хранится как объект :doc:`Phalcon\\Mvc\\Router\\Route <../api/Phalcon_Mvc_Router_Route>`.
Этот класс включает в себя все детали каждого маршрута. Например, мы можем дать ему имя и однозначно идентифицировать в нашем приложении.
Это особенно полезно, если вы хотите создать ссылки для него.

.. code-block:: php

    <?php

    $route = $router->add("/posts/{year}/{title}", "Posts::show");

    $route->setName("show-posts");

    // или проще

    $router->add("/posts/{year}/{title}", "Posts::show")->setName("show-posts");

Затем, при помощи компонента :doc:`Phalcon\\Mvc\\Url <../api/Phalcon_Mvc_Url>` и названия маршрута можно создать ссылку:

.. code-block:: php

    <?php

    // возвратит /posts/2012/phalcon-1-0-released
    echo $url->get(array(
        "for" => "show-posts",
        "year" => "2012",
        "title" => "phalcon-1-0-released"
    ));

Примеры использования
---------------------
Ниже приведены примеры пользовательских маршрутов:

.. code-block:: php

    <?php

    // пример - "/system/admin/a/edit/7001"
    $router->add(
        "/system/:controller/a/:action/:params",
        array(
            "controller" => 1,
            "action"     => 2,
            "params"     => 3
        )
    );

    // пример - "/es/news"
    $router->add(
        "/([a-z]{2})/:controller",
        array(
            "controller" => 2,
            "action"     => "index",
            "language"   => 1
        )
    );

    // пример - "/es/news"
    $router->add(
        "/{language:[a-z]{2}}/:controller",
        array(
            "controller" => 2,
            "action"     => "index"
        )
    );

    // пример - "/admin/posts/edit/100"
    $router->add(
        "/admin/:controller/:action/:int",
        array(
            "controller" => 1,
            "action"     => 2,
            "id"         => 3
        )
    );

    // пример - "/posts/2010/02/some-cool-content"
    $router->add(
        "/posts/([0-9]{4})/([0-9]{2})/([a-z\-]+)",
        array(
            "controller" => "posts",
            "action"     => "show",
            "year"       => 1,
            "month"      => 2,
            "title"      => 4
        )
    );

    // пример - "/manual/en/translate.adapter.html"
    $router->add(
        "/manual/([a-z]{2})/([a-z\.]+)\.html",
        array(
            "controller" => "manual",
            "action"     => "show",
            "language"   => 1,
            "file"       => 2
        )
    );

    // пример - /feed/fr/le-robots-hot-news.atom
    $router->add(
        "/feed/{lang:[a-z]+}/{blog:[a-z\-]+}\.{type:[a-z\-]+}",
        "Feed::get"
    );

    // пример - /api/v1/users/peter.json
    $router->add('/api/(v1|v2)/{method:[a-z]+}/{param:[a-z]+}\.(json|xml)',
        array(
            'controller' => 'api',
            'version' => 1,
            'format' => 4
        )
    );

.. highlights::
    Остерегайтесь использования спецсимволов в регулярных выражениях для контроллеров и пространств имён. Эти
    параметры формируют имена классов и файлов, что, в свою очередь, взаимодействует с файловой системой, и может
    использоваться злоумышленником для чтения несанкционированных файлов. Безопасным является регулярное выражение: /([a-zA-Z0-9\_\-]+)

Поведение по умолчанию
----------------------
У компонента :doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>` есть поведение по умолчанию, при котором все URL
обрабатываются по простому шаблону: /:controller/:action/:params

Например, ссылку вида *http://phalconphp.com/documentation/show/about.html* маршрутизатор проанализирует как:

+------------+---------------+
| Контроллер | documentation |
+------------+---------------+
| Действие   | show          |
+------------+---------------+
| Параметр   | about.html    |
+------------+---------------+

Если вы не хотите использовать маршруты по умолчанию в вашем приложении, вы должны указать false в качестве параметра при создании объекта маршрутизатора:

.. code-block:: php

    <?php

    // Создания маршрутизатора без поддержки стандартной маршрутизации
    $router = new \Phalcon\Mvc\Router(false);

Указание маршрута по умолчанию
------------------------------
При обращению к главной странице приложения срабатывает маршрут '/', в нём надо указать что должно срабатывать:

.. code-block:: php

    <?php

    $router->add("/", array(
        'controller' => 'index',
        'action' => 'index'
    ));

404 страница
------------
Если ни один из указанных маршрутов в маршрутизаторе не совпадёт, вы можете определить действие для этого случая:

.. code-block:: php

    <?php

    // Указание действия для 404 страницы
    $router->notFound(array(
        "controller" => "index",
        "action" => "route404"
    ));

Установка параметров по умолчанию
---------------------------------
Можно определить значения по умолчанию для некоторых частей маршрута, таких как модуль, контроллер или действие. Когда в маршруте отсутствует любая из
указанных частей, они будут автоматически заполнены маршрутизатором из значений по умолчанию:

.. code-block:: php

    <?php

    // Установка по умолчанию
    $router->setDefaultModule('backend');
    $router->setDefaultNamespace('Backend\Controllers');
    $router->setDefaultController('index');
    $router->setDefaultAction('index');

    // Используя значения массива
    $router->setDefaults(array(
        'controller' => 'index',
        'action' => 'index'
    ));

Использование конечного /
-------------------------
Иногда обращение к маршруту может быть с дополнительной косой чертой (слэш) и в конце маршрута, это в отдельных случаях может привести
к несоответствию маршруту. Вы можете настроить маршрутизатор для автоматического удаления слэша из конца обрабатываемого маршрута:

.. code-block:: php

    <?php

    $router = new \Phalcon\Mvc\Router();

    // Конечные косые черты будут автоматически удалены
    $router->removeExtraSlashes(true);

Или вы можете изменить определенные маршруты, в которых необходимо использовать косые черты:

.. code-block:: php

    <?php

    $router->add(
        '/{language:[a-z]{2}}/:controller[/]{0,1}',
        array(
            'controller' => 2,
            'action'     => 'index'
        )
    );

Дополнительные условия
----------------------
Иногда требуется, чтобы перед выполнением маршрут удовлетворял определённым условиям.
Вы можете добавлять произвольные условия используя функцию обратного вызова (callback)
'beforeMatch'. Если эта функция вернёт false, то запрос не совпадёт с условием и
маршрут не выполнится:

.. code-block:: php

    <?php

    $router->add('/login', array(
        'module' => 'admin',
        'controller' => 'session'
    ))->beforeMatch(function($uri, $route) {
        // Проверим, что это был Ajax-запрос
        if ($_SERVER['HTTP_X_REQUESTED_WITH'] == 'xmlhttprequest') {
            return false;
        }
        return true;
    });

Вы можете повторно использовать эти дополнительные условия в классах:

.. code-block:: php

    <?php

    class AjaxFilter
    {
        public function check()
        {
            return $_SERVER['HTTP_X_REQUESTED_WITH'] == 'xmlhttprequest';
        }
    }

И использовать этот класс вместо анонимной функции:

.. code-block:: php

    <?php

    $router->add('/get/info/{id}', array(
        'controller' => 'products',
        'action' => 'info'
    ))->beforeMatch(array(new AjaxFilter(), 'check'));

Ограничение по имени хоста
--------------------------
Маршрутизатор позволяет вам выставлять ограничения по имени хоста. Это означает, что
конкретные маршруты или группы маршрутов могут быть привязаны к конкретным именам хостов:

.. code-block:: php

    <?php

    $router->add('/login', array(
        'module' => 'admin',
        'controller' => 'session',
        'action' => 'login'
    ))->setHostName('admin.company.com');

Имя хоста так же может быть регулярным выражением:

.. code-block:: php

    <?php

    $router->add('/login', array(
        'module' => 'admin',
        'controller' => 'session',
        'action' => 'login'
    ))->setHostName('([a-z+]).company.com');

В группах маршрутов вы можете установить ограничение по имени хоста, которое будет
применяться к каждому маршруту в группе:

.. code-block:: php

    <?php

    // Создаём группу с общим модулем и контроллером
    $blog = new \Phalcon\Mvc\Router\Group(array(
        'module' => 'blog',
        'controller' => 'posts'
    ));

    // Ограничиваем по имени хоста
    $blog->setHostName('blog.mycompany.com');

    // Все маршруты начинаются с /blog
    $blog->setPrefix('/blog');

    // Маршрут по умолчанию
    $blog->add('/', array(
        'action' => 'index'
    ));

    // Добавляем маршрут в группу
    $blog->add('/save', array(
        'action' => 'save'
    ));

    // Добавляем ещё один маршрут в группу
    $blog->add('/edit/{id}', array(
        'action' => 'edit'
    ));

    // Добавляем группу в маршрутизатор
    $router->mount($blog);

Источники URI
-------------
По умолчанию текущий URI для обработки берётся из переменной $_GET['_url'], так устроено внутри Phalcon и стандартных правилах mod-rewrite,
очень просто можно указать использование для этих целей переменную $_SERVER['REQUEST_URI']:

.. code-block:: php

    <?php

    $router->setUriSource(Router::URI_SOURCE_GET_URL); // использование $_GET['_url'] (по умолчанию)
    $router->setUriSource(Router::URI_SOURCE_SERVER_REQUEST_URI); // использование $_SERVER['REQUEST_URI'] (по умолчанию)

Или вы можете самостоятельно передавать URI в метод "handle":

.. code-block:: php

    <?php

    $router->handle('/some/route/to/handle');

Тестирование маршрутов
----------------------
Компонент маршрутизации не имеет внутренних зависимостей, вы можете создать файл, как показано ниже, для проверки свои маршрутов:

.. code-block:: php

    <?php

    // Маршруты для проверки
    $testRoutes = array(
        '/',
        '/index',
        '/index/index',
        '/index/test',
        '/products',
        '/products/index/',
        '/products/show/101',
    );

    $router = new Phalcon\Mvc\Router();
    $router->setDI(new Phalcon\DI\FactoryDefault());
    // Тут необходимо установить правила маршрутизации
    //...

    // Цикл проверки маршрутов
    foreach ($testRoutes as $testRoute) {

        // Обработка маршрута
        $router->handle($testRoute);

        echo 'Тестирование ', $testRoute, '<br>';

        // Проверка выбранного маршрута
        if ($router->wasMatched()) {
            echo 'Контроллер (Controller): ', $router->getControllerName(), '<br>';
            echo 'Действие (Action): ', $router->getActionName(), '<br>';
        } else {
            echo 'Маршрут не поддерживается<br>';
        }
        echo '<br>';

    }

Маршруты на аннотациях
----------------------
Компонент :doc:`Phalcon\\Mvc\\Router\\Annotations <../api/Phalcon_Mvc_Router_Annotations>` интегрирован с
компонентом :doc:`annotations <annotations>`, и позволяет получать информацию о маршрутах из doc-блоков внутри
кода контроллеров. Используя эту стратегию, вы можете указывать маршруты непосредственно в контроллерах, вместо
того, чтобы указывать их в отдельных правилах маршрутизации:

.. code-block:: php

    <?php

    $di['router'] = function() {

        // Используем маршрутизатор на аннотациях
        $router = new \Phalcon\Mvc\Router\Annotations(false);

        // Чтение аннотаций из контроллера ProductsController для ссылок начинающихся на /api/products
        $router->addResource('Products', '/api/products');

        return $router;
    };

Аннотации могут быть определены следующим образом:

.. code-block:: php

    <?php

    /**
     * @RoutePrefix("/api/products")
     */
    class ProductsController
    {

        /**
         * @Get("/")
         */
        public function indexAction()
        {

        }

        /**
         * @Get("/edit/{id:[0-9]+}", name="edit-robot")
         */
        public function editAction($id)
        {

        }

        /**
         * @Route("/save", methods={"POST", "PUT"}, name="save-robot")
         */
        public function saveAction()
        {

        }

        /**
         * @Route("/delete/{id:[0-9]+}", methods="DELETE",
         *      conversors={id="MyConversors::checkId"})
         */
        public function deleteAction($id)
        {

        }

        public function infoAction($id)
        {

        }

    }

Маршрутизатор поддерживает только строго определённые методы, вот список текущих поддерживаемых аннотации:

+--------------+--------------------------------------------------------------------------------------------------------+--------------------------------------------------------+
| Название     | Описание                                                                                               | Использование                                          |
+==============+========================================================================================================+========================================================+
| RoutePrefix  | Префикс добавляемый к каждому маршруту. Эта аннотация должна быть в комментариях класса (контроллера)  | @RoutePrefix("/api/products")                          |
+--------------+--------------------------------------------------------------------------------------------------------+--------------------------------------------------------+
| Route        | Эта аннотация создаёт маршрут для метода, она должна быть в комментариях метода                        | @Route("/api/products/show")                           |
+--------------+--------------------------------------------------------------------------------------------------------+--------------------------------------------------------+
| Get          | Эта аннотация создаёт маршрут для метода, разрешается только HTTP метод GET                            | @Get("/api/products/search")                           |
+--------------+--------------------------------------------------------------------------------------------------------+--------------------------------------------------------+
| Post         | Эта аннотация создаёт маршрут для метода, разрешается только HTTP метод POST                           | @Post("/api/products/save")                            |
+--------------+--------------------------------------------------------------------------------------------------------+--------------------------------------------------------+
| Put          | Эта аннотация создаёт маршрут для метода, разрешается только HTTP метод PUT                            | @Put("/api/products/save")                             |
+--------------+--------------------------------------------------------------------------------------------------------+--------------------------------------------------------+
| Delete       | Эта аннотация создаёт маршрут для метода, разрешается только HTTP метод DELETE                         | @Delete("/api/products/delete/{id}")                   |
+--------------+--------------------------------------------------------------------------------------------------------+--------------------------------------------------------+
| Options      | Эта аннотация создаёт маршрут для метода, разрешается только HTTP метод OPTIONS                        | @Option("/api/products/info")                          |
+--------------+--------------------------------------------------------------------------------------------------------+--------------------------------------------------------+

Для аннотации при добавлении маршрутов поддерживаются следующие параметры:

+--------------+--------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| Название     | Описание                                                                                   | Использование                                                      |
+==============+============================================================================================+====================================================================+
| methods      | Определяет HTTP метод доступа к маршруту                                                   | @Route("/api/products", methods={"GET", "POST"})                   |
+--------------+--------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| name         | Определяет название маршрута                                                               | @Route("/api/products", name="get-products")                       |
+--------------+--------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| paths        | Массив дополнительных частей пути Phalcon\\Mvc\\Router::add                                | @Route("/posts/{id}/{slug}", paths={module="backend"})             |
+--------------+--------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| conversors   | Метод преобразования для применения к параметрам                                           | @Route("/posts/{id}/{slug}", conversors={id="MyConversor::getId"}) |
+--------------+--------------------------------------------------------------------------------------------+--------------------------------------------------------------------+

Для формирования маршрутов из контроллеров модулей стоит использовать метод addModuleResource:

.. code-block:: php

    <?php

    $di['router'] = function() {

        // Используем маршрутизатор на аннотациях
        $router = new \Phalcon\Mvc\Router\Annotations(false);

        // Чтение аннотаций из контроллера Backend\Controllers\ProductsController для ссылок начинающихся на /api/products
        $router->addModuleResource('backend', 'Products', '/api/products');

        return $router;
    };

Создание собственного маршрутизатора
------------------------------------
Для создания адаптера необходимо реализовать интерфейс :doc:`Phalcon\\Mvc\\RouterInterface <../api/Phalcon_Mvc_RouterInterface>`.
Созданным классом надо подменить маршрутизатор ('router') в момент инициализации приложения.

.. _регулярных выражений PCRE: http://www.php.net/manual/en/book.pcre.php
