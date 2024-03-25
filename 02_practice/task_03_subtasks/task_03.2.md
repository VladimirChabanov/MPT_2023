# Docker - многоконтейнерные приложения

![](./task_03.2_img/docker_compose.jpg)

## Начинаем работать с Docker compose

Docker compose - это инструмент для создания и запуска многоконтейнерных приложений Docker. Приложение описывается в декларативном стиле в файле формата YAML, а затем запускается одной командой. Загрузкой контейнеров и их работой в дальнейшем занимается docker compose.

1. Удалите все образы (кроме nginx) и все контейнеры которые у вас могли остаться с прошлой работы.

2. Выполните команду:

   ```bash
   docker compose --help
   ```

   Изучите список команд и опций доступных для управления docker compose. Для каждой команды из списка тоже можно вызвать справку, чтобы уточнить уже её список подкоманд и опций.
   Большинство команд docker compose аналогичны командам docker, но вместо *имени контейнера* в них нужно использовать *имя сервиса*.

3. Изучите структуру формата [yaml](https://learnxinyminutes.com/docs/ru-ru/yaml-ru/);

4. В домашнем каталоге создайте каталог "compose" и перейдите в него;

5. Создайте файл "compose.yaml" содержащий:

   ```yaml
   version: "3.9"
   services:
     web:
       image: nginx
   ```

   Здесь указаны 2 ключа верхнего уровня `version` и `services`:

   - Ключ [`version`](https://docs.docker.com/compose/compose-file/04-version-and-name/) указывает, версию спецификации compose-файла. В разных версиях может меняться список доступных ключей, их название или список параметров. Этот ключ рекомендует compose выбрать указанную версию, но не принуждает к этому.  
     На данный момент считается **устаревшим** (deprecated), поэтому в дальнейшем мы не будем его указывать.
   - Ключ [`services`](https://docs.docker.com/compose/compose-file/05-services/) - это раздел который содержит описание сервисов. Каждый сервис - это один контейнер. В данном примере присутствует только один сервис который назван "web".  
     Ключ `services` является **обязательным**.
   - Параметры сервиса "web" тоже представлены в виде набора ключей (здесь только один): `image` - базовый образ для контейнера.

   В этом файле описаны действия аналогичное консольной команде:

   ```bash
   docker run nginx
   ```

6. Запустите приложение командой:

   ```bash
   docker compose up
   ```

   В данном случае compose будет искать в текущем каталоге файл с именем: "compose.yaml" (предпочтительно) или "compose.yml" или "docker-compose.yaml" или "docker-compose.yml, после чего запустит приложение.
   Перед запуском compose проверит наличие всех образов и, при необходимости, скачает их, затем создаст необходимые сети и тома, а затем выполнит `docker run` для каждого контейнера указанного в разделе `services`.

7. В данном случае мы запустили сервис "web" в attached mode, поэтому он захватил ввод и вывод нашего терминала.  
   Остановите контейнер <kbd>Ctrl</kbd>+<kbd>C</kbd> и посмотрите список всех контейнеров `docker ps -a`.  Как видно в списке присутствует один контейнер построенный на базе образа nginx.

8. Выполните команду:

   ```bash
   docker compose down
   ```

   Эта команда остановит приложение (у нас оно уже остановлено) и удалит всё что было ранее создано compose кроме томов.

9. Проверьте, что теперь список контейнеров пуст.

10. Модифицируйте "compose.yaml" следующим образом:

    ```yaml
    services:
      web:
        image: nginx
        ports:
          - "80:80"
    ```

    и запустите приложение командой:

    ```bash
    docker compose up -d
    ```

    Здесь мы добавили ещё один ключ в список параметров контейнера - это ключ `ports`. Это массив пробрасываемых портов.  
    Опция `-d` в команде запуска приложения говорит, что контейнеры нужно запустить в detached mode. Т.е. то что мы сделали аналогично консольной команде:

    ```bash
    docker run -d -p 80:80 nginx
    ```

11. Перейдите в браузер в основной ОС и введите в строку адреса ip виртуальной машины. Вы должны увидеть приветственную страницу от nginx.

12. Остановите приложение командой:

    ```bash
    docker compose stop
    ```

    Эта команда выполнит `stop` для всех конвейеров приложения, но НЕ удали их, сети и остальное.

13. Запустите приложение командой:

    ```bash
    docker compose start
    ```

    Эта команда выполнит `start` для всех контейнеров приложения. Убедитесь, что через браузер по прежнему можно получить ответ от nginx.

14. Остановите приложение при помощи команды:

    ```bash
    docker compose down
    ```

15. Модифицируйте "compose.yaml" следующим образом:

    ```yaml
    services:
      web:
        container_name: web
        image: nginx
        ports:
          - "80:80"
        volumes:
          - ./content:/usr/share/nginx/html
    ```

    Здесь, у сервиса "web", добавлено два новых ключа:

    - [`container_name`](https://docs.docker.com/compose/compose-file/05-services/#container_name) - который устанавливает для контейнера созданного сервисом "web" имя "web". По умолчанию имя контейнера генерируется compose на основании имени текущего каталога, имени сервиса и цифры, чтобы предотвратить конфликт имён;
    - [`volumes`](https://docs.docker.com/compose/compose-file/05-services/#volumes) - массив томов и каталогов которые будут подмонтированы к контейнеру. В данном случае мы монтируем локальный каталог "content" из текущего каталога (будет создан автоматически) в "/usr/share/nginx/html" внутри контейнера.

    Файл аналогичен консольной команде:

    ```bash
    docker run -d --name=web -v ./content:/usr/share/nginx/html -p 80:80 nginx
    ```

16. Запустите приложение и проверьте, что контейнеру присвоено имя "web" и в текущем каталоге появился каталог "content".

17. Как и ожидалось каталог пустой и принадлежит root. Поменяйте владельца каталога на своего пользователя и создайте в нём файл "index.html" содержащий:

    ```html
    <h1>Hello from compose</h1>
    ```

    Перейдите в браузер и проверьте какую страницу теперь присылает nginx.

18. Остановите приложение (`down`) и убедитесь, что каталог "content" по прежнему существует.

19. Модифицируйте "compose.yaml" следующим образом:

    ```yaml
    services:
      web:
        image: nginx
        container_name: web
        ports:
          - "80:80"
        volumes:
          - data:/usr/share/nginx/html
    
    volumes:
      data:
    ```

    Здесь в разделе `volumes` сервиса "web" теперь монтируется том "data". Т.к. тома можно создавать с различными настройками, поэтому у нас добавился ключ *верхнего* уровня [`volumes`](https://docs.docker.com/compose/compose-file/07-volumes/) создающий раздел в котором нужно перечислить тома используемые сервисами.  
    В этом примере мы создаём том "data" со стандартными настройками, поэтому он указан без параметров (двоеточие в конце - не опечатка).  
    Файл аналогичен консольным командам:

    ```bash
    docker volume create data
    docker run -d --name=web -v data:/usr/share/nginx/html -p 80:80 nginx
    ```

20. Запустите приложение и проверьте список томов.  
    Как видно там появился том, название которого состоит из названия текущего каталога и названия тома в yaml файле.  
    Если том с таким именем уже существует и он создан не compose, то вы получите предупреждение. Чтобы намеренно использовать существующий том, нужно создавать его таким образом:

    ```yaml
    volumes:
      data:
        external: true
    ```

    Т.к. указано `external: true`, то compose будет искать существующий том ровно с именем "data" и если не найдёт, то приложение не запустится.

21. Остановите приложение и убедитесь, что том по прежнему существует.

22. Модифицируйте "compose.yaml" следующим образом:

    ```yaml
    services:
      web:
        image: nginx
        container_name: web
        ports:
          - "80:80"
        volumes:
          - data:/usr/share/nginx/html
        networks:
          - local
    
      tools:
        image: avenga/net-tools
        command: "sleep 10m"
        networks:
          - local
    
    volumes:
      data:
    
    networks:
      local:
    ```

    Теперь приложение состоит из двух сервисов: "web" и "tools". В параметрах сервиса "web" появился новый ключ [`networks`](https://docs.docker.com/compose/compose-file/05-services/#networks) в котором нужно перечислить все сети к которым должен быть подключён контейнер. Т.к. сети могут быть созданы с разными настройками, поэтому у нас добавился ключ *верхнего* уровня [`networks`](https://docs.docker.com/compose/compose-file/06-networks/) создающий раздел в котором нужно перечислить сети используемые сервисами.

    Второй сервис "tools" основан на образе "avenga/net-tools" и подключается к той же сети, что и сервис "web". Кроме того у "tools" присутствует ключ [`command`](https://docs.docker.com/compose/compose-file/05-services/#command) который используется для указания какую команду нужно выполнить после старта контейнера. В нашем случае указана команда "sleep 10m", это нужно для того, чтобы контейнер не закрывался сразу после старта и мы успели в нём поработать.  
    Файл аналогичен консольным командам:

    ```bash
    docker volume create data
    docker network create local
    docker run -d --network=local --name=web -v data:/usr/share/nginx/html -p 80:80 nginx
    docker run -d --network=local avenga/net-tools sleep 10m
    ```

23. Запустите приложение и посмотрите список сетей.  
    Как видно там появилась сеть, название которой состоит из названия текущего каталога и названия сети в yaml файле.  
    Для подключения к существующей сети также как и для тома используйте ключ `external: true`.

24. Выполните команду:

    ```bash
    docker network inspect compose_local
    ```

    и найдите ip адрес контейнера "web".

25. Выполните команду:

    ```bash
    docker compose exec tools curl {ip_контейнера_web}
    ```

    Здесь мы используем команду `docker compose exec` при помощи которой запускаем команду `curl {ip_контейнера_web}` внутри сервиса `tools`.  
    Как результат вы должны получить приветствие от nginx, что доказывает, что контейнеры подключены к одной сети.

26. Остановите приложение и проверьте, что сети "compose_local" больше нет в списке.

27. В прошлой работе вы создавали приложение "var_keeper" и загружали его на свой аккаунт в DockerHub. Используем этот образ и развернём приложение на локальной машине при помощи compose.

28. Модифицируйте "compose.yaml" следующим образом:

    ```yaml
    services:
      keeper:
        image: {ваш_логин}/var_keeper
        ports:
          - "80:5000"
        networks:
          - net
    
      db:
        image: mysql
        environment:
          - MYSQL_ROOT_PASSWORD=123
        volumes:
          - vars:/var/lib/mysql
        networks:
          - net
    
    volumes:
      vars:
    
    networks:
      net:
    ```

    Как видно, здесь создаются два сервиса "keeper" и "db", том "vars" и сеть "net". Из новых ключей здесь добавился ключ [`environment`](https://docs.docker.com/compose/compose-file/05-services/#environment), который позволяет определить набор переменных окружения которые будут доступны в контейнере.  
    Файл аналогичен консольным командам:

    ```bash
    docker volume cleate vars
    docker network cleate net
    docker run -d --network=net -p 80:5000 {ваш_логин}/var_keeper
    docker run -d --network=net -v vars:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123 mysql
    ```

29. Запустите приложение и проверьте его работоспособность при помощи браузера. Чтобы послать POST-запрос сервису "keeper" введите в строке адреса:

    ```html
    data:text/html,<form action=http://{ip_виртуальной_машины}:80/var/a method=post><input name=value></form>
    ```

    На экране появится поле ввода. Введённое значение будет отправлено на `/var/a` и приложение должно запомнить новую переменную `a`.

30. В результате вы должны получить ошибку. Воспользуйтесь командой:

    ```
    docker compose logs
    ```

    Здесь вы увидите вывод *всех* контейнеров приложения, а если добавить имя сервиса, то только логи указанного контейнера.  
    Пролистав логи в начало вы найдёте сообщение о том, что не удалось подключится к базе данных. Всё нормально, контейнер с базой данных в полном порядке, просто compose запускает все контейнеры асинхронно и контейнер с базой данных не успел запуститься вовремя.

31. Приложение не работает, поэтому остановите его.

32. Чтобы указать порядок запуска сервисов в compose есть специальный ключ [`depends_on`](https://docs.docker.com/compose/compose-file/05-services/#depends_on), который позволяет указать список сервисов, которые должны быть запущены перед текущим.  
    Модифицируйте "compose.yaml" следующим образом:

    ```yaml
    services:
      keeper:
        image: {ваш_логин}/var_keeper
        ports:
          - "80:5000"
        networks:
          - net
        depends_on:
          - db
    
      db:
        image: mysql
        environment:
          - MYSQL_ROOT_PASSWORD=123
        volumes:
          - vars:/var/lib/mysql
        networks:
          - net
          
    volumes:
      vars:
      
    networks:
      net:
    ```

    Файл аналогичен консольным командам (база данных теперь запускается первой):  
    ```bash
    docker volume cleate vars
    docker network cleate net
    docker run -d --network=net -v vars:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123 mysql
    docker run -d --network=net -p 80:5000 {ваш_логин}/var_keeper
    ```

33. Запустите приложение и сразу проверьте логи.  
    Как вы обнаружите, логи контейнера с базой данных теперь в начале, но ошибка подключения всё равно сохранилась. Всё нормально `depends_on` отработал как положено, просто он гарантирует порядок запуска контейнеров, но не может проконтролировать готовность того, что внутри. Т.к. запуск базы данных - это довольно долгая процедура, то контейнер "keeper" успевает обогнать контейнер "db".

34. Приложение по прежнему не работает - остановите его.

35. Чтобы решить нашу задачу compose предоставляет возможность проверять "здоровье" (healthy) контейнера при помощи ключа [`healthcheck`](https://docs.docker.com/compose/compose-file/05-services/#healthcheck).  
    Модифицируйте "compose.yaml" следующим образом:

    ```yaml
    services:
      keeper:
        image: {ваш_логин}/var_keeper
        ports:
          - "80:5000"
        networks:
          - net
        depends_on:
          db:
            condition: service_healthy
    
      db:
        image: mysql
        environment:
          - MYSQL_ROOT_PASSWORD=123
        volumes:
          - vars:/var/lib/mysql
        networks:
          - net
        healthcheck:
          test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost"]
          timeout: 5s
          retries: 10
          
    volumes:
      vars:
      
    networks:
      net:
    ```

    Здесь мы добавили контейнеру "db" параметр `healthcheck`, который будет запускать команду `mysqladmin ping -h localhost` каждые 5 секунд (`timeout`) 10 раз (`retries`), а контейнеру "keeper" в раздел `depends_on.db` условие при котором нужно запустить контейнер `condition: service_healthy`.  
    Это приведёт к тому, что контейнер "keeper" будет ждать пока `healthcheck` контейнера "db" не станет успешным и только после этого запуститься.

36. Запустите приложение, дождитесь пока все контейнеры стартуют и затем проверьте логи.  
    Теперь вы должны обнаружить сообщение:

    ```bash
    Connection to db:ОК
    Create db:ОК
    Change db:ОК
    Create table:ОК
    ```

37. Если в процессе работы приложения один или несколько контейнеров остановятся, то приложение перестанет работать нормально. Чтобы поддерживать его работоспособность можно назначить политику перезапуска при помощи ключа [`restart`](https://docs.docker.com/compose/compose-file/05-services/#restart).

38. Остановите приложение и модифицируйте "compose.yaml" следующим образом:  
    ```yaml
    services:
      keeper:
        image: {ваш_логин}/var_keeper
        ports:
          - "80:5000"
        networks:
          - net
        depends_on:
          db:
            condition: service_healthy
        restart: always
    
      db:
        image: mysql
        environment:
          - MYSQL_ROOT_PASSWORD=123
        volumes:
          - vars:/var/lib/mysql
        networks:
          - net
        healthcheck:
          test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost"]
          timeout: 5s
          retries: 10
          
    volumes:
      vars:
      
    networks:
      net:
    ```

    Здесь мы добавили сервису "keeper" политику `restart: always`, т.е. всегда перезапускать контейнер до тех пор, пока его не удалят. Данная политика не отработает если мы остановим контейнер вручную (нужно будет перезапустить демон docker), поэтому, чтобы проверить работоспособность политики перезапуска мы сломаем его изнутри.

39. Запустите приложение и зайдите в контейнер сервиса "keeper" при помощи команды:

    ```bash
    docker compose exec keeper sh
    ```

    Затем найдите в списке процессов (`ps`) главный процесс контейнера у него будет такое имя: `/usr/local/bin/python /app/app.py` и завершите его при помощи команды `kill -9 {PID}`.  
    Завершение главного процесса контейнера приведёт к его остановке и вас из него выкинет.

40. Проверьте список запущенных контейнеров и вы должны обнаружить, что "keeper" по прежнему на месте, но в столбце `STATUS` указано, что он был запущен совсем недавно.

41. Остановите приложение.

42. Сейчас для сохранения и получения значений переменных нужно напрямую посылать приложению запросы, что не очень удобно. Поэтому добавим ему простенький фронтенд. Сборку контейнера с фронтендом поручим compose, но для этого ему нужен Docerfile и код фронтенда.  
    Выполните действия:

    - Создайте в директории "compose" ещё одну с именем "front" и перейдите туда;

    - Создайте файл "app.py" содержащий:

      ```python
      import requests
      from flask import Flask, request, render_template
       
      app = Flask(__name__)
       
      @app.route('/')
      def home():
        return render_template("index.html", name = "", value = "")
       
      @app.route('/var/<var_name>', methods=['GET'])
      def get(var_name):
        var_value = requests.get(f'http://keeper:5000/var/{var_name}')
        return var_value.text
       
      @app.route('/var/<var_name>', methods=['POST'])
      def set(var_name):
        var_value = request.form.get(f"{var_name}")
        resp = requests.post(f'http://keeper:5000/var/{var_name}', data={'value': var_value})
        return resp.text
       
      if __name__ == "__main__":
        app.run(debug=True, host='0.0.0.0', port=80)
      ```

      Здесь flask-приложение которое слушает на 80 порту и занимается двумя вещами:
      
      - На запрос по  `/` отдаёт файл "index.html" с интерфейсом пользователя;
      - На запрос по `/var/<var_name>` выполняется простая пересылка данных на сервис "keeper" который слушает на 5000.
      
    - Создайте каталог "templates" и в нём файл "index.html" содержащий:

      ```html
      <!DOCTYPE html>
      <html>
      <head>
        <title>Keep your vars</title>
      </head>
      <body>
        <label>Name: <input type="text" id="name" value=""></label><br><br>
        <label>Value: <input type="text" id="value" value=""></label><br><br>
      
        <button onclick="postValue()">Set value</button>
        <button onclick="getValue()">Get value</button>
        
        <div id="success" style="display: none; color: green;">OK</div>
        <div id="failure" style="display: none; color: red;">ERROR</div>
        <script>
          const successfulRequest = () =>{
      	  document.getElementById("success").style.display = "inline-block";
            setTimeout(function() {
              document.getElementById("success").style.display = "none";
            }, 3000);	    
      	};
            
          const failedRequest = () =>{
      	  document.getElementById("failure").style.display = "inline-block";
            setTimeout(function() {
              document.getElementById("failure").style.display = "none";
            }, 3000);	    
      	};
            
          const getValue = () => {
            const url = "var/" + document.getElementById("name").value;
            fetch(url).then(response => {
              if (!response.ok) throw new Error('Network response was not ok');
              return response.text();
            }).then(data => {
              document.getElementById("value").value = data;
              successfulRequest();
            }).catch(error => {
              failedRequest();
            });
          };
      
          const postValue = () => {
            const name = document.getElementById("name").value;
            const value = document.getElementById("value").value;
            let formData = new FormData();
            formData.append(name, value);
            fetch("var/"+name, {
              method: 'POST',
              body: formData
            }).then(response => {
              if (!response.ok) throw new Error('Network response was not ok');
              return response.text();
            }).then(data => {
              if (data !== "OK") throw new Error('Variable was not set');
              successfulRequest();
            }).catch(error => {
              failedRequest();
            });  
          };
        </script>
      </body>
      </html>
      ```

      Здесь два поля ввода и две кнопки. Первая кнопка выполняет установку значения переменной, а вторая её получение.  Для установки значения переменной выполняется POST запрос на наш фронтенд, который будет переслан дальше на keeper, а для получения, аналогичным образом, выполняется GET запрос.
      
    - Вернитесь в каталог "front" и чтобы получить список зависимостей выполните команды:

      ```bash
      python3 -m venv venv
      source venv/bin/activate
      pip install flask requests
      pip freeze > requirements.txt
      ```
      
      Здесь нам интересен только файл "requirements.txt".
      
    - Создайте Dockerfile (версию python укажите свою):

      ```python
      FROM python:3.10.12-alpine
      WORKDIR /app
      COPY / /app
      RUN pip install -r requirements.txt
      ENTRYPOINT [ "python" ]
      CMD [ "app.py" ]
      ```
      
      Файл точно такой же, как и для сервиса "keeper".

43. Теперь у нас есть готовый фронтенд и Dockerfile, чтобы упаковать его в контейнер.  
    Выйдите из каталога "front" и модифицируйте "compose.yaml" следующим образом:

    ```yaml
    services:
      front:
        build: ./front
        image: var_keeper_front
        ports:
          - "80:80"
        networks:
          - net
        depends_on:
          - keeper
        restart: always
    
      keeper:
        image: {ваш_логин}/var_keeper
        networks:
          - net
        depends_on:
          db:
            condition: service_healthy
        restart: always
    
      db:
        image: mysql
        environment:
          - MYSQL_ROOT_PASSWORD=123
        volumes:
          - vars:/var/lib/mysql
        networks:
          - net
        healthcheck:
          test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost"]
          timeout: 20s
          retries: 10
    
    volumes:
      vars:
    
    networks:
      net:
    ```

    Здесь мы добавляем ещё один сервис "front", который будет собираться из Dockerfile. Каталог сборки указываем при помощи ключа [`build`](https://docs.docker.com/compose/compose-file/05-services/#build), при этом собранный образ будет сохранён в системе под именем "var_keeper_front" (ключ `image`). При последующих запусках приложения он будет использоваться как обычный образ без повторной сборки.  
    Кроме того, из соображений безопасности, мы убираем у сервиса "keeper" проброс портов, чтобы он был недоступен из внешней сети (он доступен только внутри нашей сети `net`) и пробрасываем 80-й порт фронтэнда для доступа пользователей к приложению.

44. Выполните команду:

    ```bash
    docker compose build
    ```

    Эта команда найдёт в файле "compose.yaml" все сервисы которые нужно собрать и выполнит их сборку без запуска приложения. Её также можно использовать для повторной сборки образов если их исходники были изменены.

45. Проверьте список образов. Теперь в нём должен появится образ "var_keeper_front:latest".

46. Выполните команду:

    ```bash
    docker compose up -d
    ```

    В случае, если бы образа "var_keeper_front" не было в системе, то compose вместо скачивания его с DockerHub запустил бы процесс сборки. Т.к. в нашем случае образ уже существует, то произойдёт обычный запуск.

47. Перейдите в браузер и введите в строку адреса ip виртуальной машины. Воспользуйтесь приложением, чтобы сохранить две переменные, а затем проверьте, что их значения корректно воспроизводятся.

<br>

## Источники

1. [Кратко о YAML](https://learnxinyminutes.com/docs/ru-ru/yaml-ru/) - пример yaml-файла с подробными комментариями;
2. [JSON ⇆ YAML](https://www.json2yaml.com/) - позволяет преобразовать json в yaml и обратно. Может помочь разобрать с yaml, если вы знакомы с json;
3. [08-Docker-COMPOSE. Простой запуск контейнеров](https://youtu.be/p8tNcUIQzZU) - видео про docker compose. В начале немного теории, затем практика.
4. Как успешно реализовать проверку состояния контейнера в Docker Compose: [длинная ссылка](https://medium.com/nuances-of-programming/как-успешно-реализовать-проверку-состояния-контейнера-в-docker-compose-6e3b449018b7)