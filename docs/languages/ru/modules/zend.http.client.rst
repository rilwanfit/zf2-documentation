.. EN-Revision: none
.. _zend.http.client:

Zend\Http\Client - Введение
===========================

.. _zend.http.client.introduction:

Введение
--------

Zend\Http\Client предоставляет простой интерфейс для выполнения
HTTP-запросов. Zend\Http\Client поддерживает как большинство простых
возможностей, ожидаемых от любого HTTP-клиента, так и более
сложные функции, такие, как HTTP-аутентификация и выгрузка
файлов. При успешно выполненных запросах (и большинстве
неуспешно выполненных) возвращается объект Zend\Http\Response, который
предоставляет доступ к заголовкам и телу ответа (см. :ref:`
<zend.http.response>`).

Конструктор класса опционально принимает URL (может быть
строкой или объектом Zend\Uri\Http) и массив конфирурационных
параметров. Оба параметра могут быть опущены и установлены
позднее через методы *setUri()* и *setConfig()*.

   .. rubric:: Инстанцирование объекта Zend\Http\Client

   .. code-block:: php
      :linenos:

      <?php
          require_once 'Zend/Http/Client.php';

          $client = new Zend\Http\Client('http://example.org', array(
              'maxredirects' => 0,
              'timeout'      => 30));

          // Этот код делает то же самое:
          $client = new Zend\Http\Client();
          $client->setUri('http://example.org');
          $client->setConfig(array(
              'maxredirects' => 0,
              'timeout'      => 30));

      ?>


.. _zend.http.client.configuration:

Параметры конфигурации
----------------------

Конструктор и метод *setConfig()* принимают ассоциативный массив
параметров конфигурации. Установка этих параметров является
опциональной, поскольку все они имеют значения по умолчанию.

   .. table:: Параметры конфигурации Zend\Http\Client

      +---------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+---------------------------------+
      |Параметр       |Описание                                                                                                                                                                               |Тип               |Значение по умолчанию            |
      +===============+=======================================================================================================================================================================================+==================+=================================+
      |maxredirects   |Максимальное количество последующих перенаправлений (0 = ни одного перенаправления)                                                                                                    |integer           |5                                |
      +---------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+---------------------------------+
      |strictredirects|Строгое следование спецификации RFC при перенаправлениях (см. )                                                                                                                        |boolean           |false                            |
      +---------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+---------------------------------+
      |useragent      |Идентификатор агента пользователя (отправляется в заголовке запроса)                                                                                                                   |string            |'Zend\Http\Client'               |
      +---------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+---------------------------------+
      |timeout        |Таймаут соединения в секундах                                                                                                                                                          |integer           |10                               |
      +---------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+---------------------------------+
      |httpversion    |Версия протокола HTTP                                                                                                                                                                  |float (1.1 or 1.0)|1.1                              |
      +---------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+---------------------------------+
      |adapter        |Используемый класс адаптера соединения (см. )                                                                                                                                          |mixed             |'Zend\Http\Client\Adapter\Socket'|
      +---------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+---------------------------------+
      |keepalive      |Включение поддержки соединения keep-alive с сервером. Может быть полезно и повышает поизводительность, если выполняется несколько последовательных запросов к одному и тому же серверу.|boolean           |false                            |
      +---------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+---------------------------------+



.. _zend.http.client.basic-requests:

Выполнение базовых HTTP-запросов
--------------------------------

Выполнение простых HTTP-запросов с использованием метода *request()*
довольно простое, и редко требуется больше кода, чем в эти три
строчки:

   .. rubric:: Выполнение простого запроса GET

   .. code-block:: php
      :linenos:

      <?php
          require_once 'Zend/Http/Client.php';

          $client = new Zend\Http\Client('http://example.org');
          $response = $client->request();
      ?>
Метод *request()* принимает один необязательный параметр - метод
запроса. Это могут быть методы GET, POST, PUT, HEAD, DELETE, TRACE, OPTIONS или
CONNECT, определенные в протоколе HTTP. [#]_. Для удобства все они
определены как константы класса: Zend\Http\Request::GET, Zend\Http\Request::POST и
т.д.

Если метод запроса не был указан, то используемый метод
определяется последним вызовом *setMethod()*. Если *setMethod()* не был
вызван, то по умолчанию используется метод GET (см. пример выше).

   .. rubric:: Использование методов запроса, отличных от GET

   .. code-block:: php
      :linenos:

      <?php
          // Выполнение запроса POST
          $response = $client->request('POST');

          // Еще один способ сделать то же самое:
          $client->setMethod(Zend\Http\Client::POST);
          $response = $client->request();
      ?>


.. _zend.http.client.parameters:

Добавление параметров GET и POST
--------------------------------

Добавление параметров GET в HTTP-запрос довольно простое, это
может быть сделано посредством определения параметров как
часть URL или с использованием метода *setParameterGet()*. Этот метод
принимает имя параметра GET и его значение первый и второй
аргументы соответственно. Метод *setParameterGet()* может также
принимать ассоциативный массив пар имя => значение, что удобно,
если нужно установить несколько параметров GET.

   .. rubric:: Установка параметров GET

   .. code-block:: php
      :linenos:

      <?php
          // Установка параметра GET с использованием метода setParameterGet
          $client->setParameterGet('knight', 'lancelot');

          // Эвивалентный код с установкой через URL:
          $client->setUri('http://example.com/index.php?knight=lancelot');

          // Добавление нескольких параметров в одном вызове
          $client->setParameterGet(array(
              'first_name'  => 'Bender',
              'middle_name' => 'Bending'
              'made_in'     => 'Mexico',
          ));
      ?>


В то время как параметры GET могут отправляться с любыми
методами запроса, параметры POST могут отправляться только в
теле запроса POST. Добавление параметров POST к запросу очень
похоже на добавление параметров GET и выполняется через метод
*setParameterPost()*.

   .. rubric:: Установка параметров POST

   .. code-block:: php
      :linenos:

      <?php
          // Установка параметра POST
          $client->setParameterPost('language', 'fr');

          // Установка нескольких параметров POST,
          // один из них - с несколькими значениями
          $client->setParameterPost(array(
              'language'  => 'es',
              'country'   => 'ar',
              'selection' => array(45, 32, 80)
          ));
      ?>
Заметьте, что отправляя запрос POST, вы можете установить как
параметры POST, так и параметры GET. С другой стороны, хотя
установка параметров POST для не-POST запросов не вызывает ошибки,
она не имеет практического смысла. Если запрос не производится
по методу POST, то параметры POST просто игнорируются.

.. _zend.http.client.accessing_last:

Получение последних запроса и ответа
------------------------------------

Zend\Http\Client предоставляет методы для получения последнего
отправленного запроса и последнего ответа, полученного через
объект клиента. Метод *Zend\Http\Client->getLastRequest()* не требует
параметров и возвращает последний HTTP-запрос, отправленный
через объект клиента, в виде строки. Аналогично,
*Zend\Http\Client->getLastResponse()* возвращает последний HTTP-ответ, полученный
через объект клиента, в виде объекта :ref:`Zend\Http\Response <zend.http.response>`.



.. _`http://www.w3.org/Protocols/rfc2616/rfc2616.html`: http://www.w3.org/Protocols/rfc2616/rfc2616.html

.. [#] См. RFC 2616 -`http://www.w3.org/Protocols/rfc2616/rfc2616.html`_.