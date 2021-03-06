---
layout: post
title: "Простой чат на NodeJS и Socket.IO"
---

Одним из лучших способов понять работу WebSocket, это написание чата, который позволяет людям общаться в режиме реального времени. И именно это мы разберем в этой статье.

Все мы помним конструкции `document.write("Hello World");`, которые мы писали 10(или 15) лет назад. Конечно это очень простое выражение, и мы не будем забывать когда-то так популярное выражение `var today = new Date(); document.write(today);`. За это время JavaScript быстро развивалась в DOM манипулировании с помощью различных фреймворков и библиотек, таких как Prototype и jQuery. Используя эти методы, можно создавать AJAX запросы, кнопки добавления и удаления элементов интерфейса, карусели, лайт боксы... возможности бесконечны. И когда мы подумали что вот оно, JavaScript развивается и становится все лучше и лучше - в результате мы имеем NodeJS, AngularJS, BackboneJS и кучу новых фреймворков JavaScript, которые помогут достичь очень многого.

Когда я впервые услышал о NodeJS, мои мысли были: "Что за выдумка! Серверный JavaScript?". Я не мог себе этого представить, ведь мои знания ограничивались моим предыдущим, хоть и несколько ограниченным знанием этого языка. Однажды я начал работать с nodeJS и сразу все понял. Просто для того что бы дать Вам немного объяснений: nodeJS использует движок Google V8 JavaScript, который сам по себе невероятно быстр.

В базовом приложении nodeJS, есть возможность создать свой собственыый веб-сервер(это означает что больше не нужно использовать Apache или Nginx). Также мы добавим Socket.IO, который является отличным решением для создания приложений, которые работают в любом браузере, включая мобильные устройства. Socket.IO позволяет отправлять сообщения между клиентом(браузер) и сервером(WebSocket сервер). Он является одним из самым востребованным для создания приложений чата, где каждый пользователь, который подключается к клиенту, имеет возможность получаеть ответы от всех подключенных пользователей - идеальное решение для чата, не так ли?

Вот список главных особенностей, которые я собираюсь продемонстрировать:

- Люди могут входить от своего имени в чат
- Затем они подключаются к серверу WebSocket
- WebSocket сервер отправляет сообщение с подтверждением клиенту, а так же уведомляет других клиентов о том, что кто-то подключился
- Пользователи имеют возможность обмениваться сообщениями
- Уведомление подключенных клиенов когда кто-то вышел из чата
- Уведомлять подключенных клиентов, если сервер WebSocket выключается

В первую очередь мя создадим WebSocket сервер, который будет принимать соединения на `0.0.0.0:3000`. Мы так же будем хранить список подключенных пользователей:

    var io = require('socket.io');
    var socket = io.listen(3000, '0.0.0.0');
    var people = {};

Каждое событие, которое должно произойти, пока WebSocket работает, должно быть обернуто в специальную функцию: `socket.on('connection', function(client) {...});`. Итак наши события:

- подключение пользователя
- отправка сообщения
- отключение пользователя

Давайте опишим наши события:

    socket.on('connection', function (client) {
      client.on('join', function(name){
        people[client.id] = name;
        client.emit('update', 'Вы подключились к серверу.');
        socket.sockets.emit('update', name + ' подключился к серверу.');
        socket.sockets.emit('update-people', people);
      });

      client.on('send', function(msg){
        socket.sockets.emit('chat', people[client.id], msg);
      });

      client.on('disconnect', function(){
        socket.sockets.emit('update', people[client.id] + ' отключился от сервера.');
        delete people[client.id];
        socket.sockets.emit('update-people', people);
      });
    });

Сервер WebSocket будет "слушать" функцию `join`, которая будет уведомлять подключенных клиентов(как мы увидим позднее, это вызывается через событие нажатия кнопки). Функция `join` отправит параметр `name`, который затем добавится в хэш `people` с уникальным идентификатором подключенного сокета. Как только он будет сохранен, WebSocket отправит уведомление(`update`), нашему клиенту(обратите внимание, что `client.emit()` отправит сообщение только клиенту, который его "слушает", тогда как `socket.sockets.emit()` отправит уведомление всем пользователям). Другими словами, уведомление "Вы подключились к серверу." увидет клиент, который только что подключился, в то время как два других, будут оправляться всем подключенным клиентам, включая подключенного в данный момент пользователя.

Следующая функция, это функция `send`, которая отправляет всем сведения об авторе и само сообщение.

`Disconnect` - обеспечивает уведомление всех пользователей, когда один из них отключился и обновляет хэш `people` у всех пользователей.

Давайте посмотрим как это выглядит на стороне клиента:

    <!DOCTYPE html>
    <html lang="en">
      <head>
        <script src="http://0.0.0.0:3000/socket.io/socket.io.js"></script>
        <script src="http://0.0.0.0/js/bootstrap.js"></script>
        <script src="http://0.0.0.0/js/jquery.js"></script>
        <link rel="stylesheet" type="text/css" href="http://0.0.0.0/css/bootstrap.css">
        <script>
          <!-- Добавим чуть позже -->
        </script>
      </head>
      <body>
        <div class="row">
          <div class="span2">
            <ul id="people" class="unstyled"></ul>
          </div>
          <div class="span4">
            <ul id="msgs" class="unstyled">
          </div>
        </div>
        <div class="row">
          <div class="span5 offset2" id="login">
            <form class="form-inline">
              <input type="text" class="input-small" placeholder="Ваше имя" id="name">
              <input type="button" name="join" id="join" value="Присоединиться" class="btn btn-primary">
            </form>
          </div>

          <div class="span5 offset2" id="chat">
            <form id="2" class="form-inline">
              <input type="text" class="input" placeholder="Ваше сообщение" id="msg">
              <input type="button" name="send" id="send" value="Отправить" class="btn btn-success">
            </form>
          </div>
        </div>
      </body>
    </html>

Тут нет ничего не обычного. Это просто HTML страница, которая использует Bootstrap библиотеку. А теперь давайте вставим этот JavaScript код между тегами `script`.

    $(document).ready(function(){
      var socket = io.connect('0.0.0.0:3000');

      $('#chat').hide();
      $('#name').focus();
      $('form').submit(function(event){
        event.preventDefault();
      });

      $('#join').click(function(){
        var name = $('#name').val();
        if (name != '') {
          socket.emit('join', name);
          $('#login').detach();
          $('#chat').show();
          $('#msg').focus();
          ready = true;
        }
      });

      $('#name').keypress(function(e){
        if(e.which == 13) {
          var name = $('#name').val();
          if (name != '') {
            socket.emit('join', name);
            ready = true;
            $('#login').detach();
            $('#chat').show();
            $('#msg').focus();
          }
        }
      });

      socket.on('update', function(msg) {
        if(ready) $('#msgs').append('<li>' + msg + '</li>');
      })

      socket.on('update-people', function(people){
        if(ready) {
          $('#people').empty();
          $.each(people, function(clientid, name) {
            $('#people').append('<li>' + name + '</li>');
          });
        }
      });

      socket.on('chat', function(who, msg){
        if(ready) {
          $('#msgs').append('<li>' + who + ' написал: ' + msg + '</li>');
        }
      });

      socket.on('disconnect', function(){
        $('#msgs').append('<li>Сервер не доступен</li>');
        $('#msg').attr('disabled', 'disabled');
        $('#send').attr('disabled', 'disabled');
      });


      $('#send').click(function(){
        var msg = $('#msg').val();
        socket.emit('send', msg);
        $('#msg').val('');
      });

      $('#msg').keypress(function(e){
        if(e.which == 13) {
          var msg = $('#msg').val();
          socket.emit('send', msg);
          $('#msg').val('');
        }
      });
    });