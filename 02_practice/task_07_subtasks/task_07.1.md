# Kubernetes. Play with Kubernetes

В качестве первой части рассмотрим работу уже развёрнутого Kubernetes. Наша задача заключается только в том, чтобы объединить сервера в кластер и запустить простое приложение.

## Объединение серверов в кластер

Эта часть работы не требует локально установленного софта.

1. Перейдите на сайт [https://labs.play-with-k8s.com/](https://labs.play-with-k8s.com/) и залогиньтесь при помощи учётной записи gitHub или dockerHub.

2. После того, как вы нажмёте на кнопку "Start" для вас будет создана 4-х часовая сессия, в пределах которой можно будет создать до 5-ти серверов (node) с уже установленными компонентами kubernetes.  
   Создайте максимум нод при помощи кнопки "Add new instance".

3. Каждую ноду можно воспринимать как отдельный сервер подключённый к подсети 192.168.0.0. Несмотря на то, что это локальная сеть (т.е. недоступная из интернета) каждой ноде присвоено доменное имя (url) на поддомене `direct.labs.play-with-k8s.com` по которому уже можно будет достучаться до сервера.

4. На каждой ноде вы увидите одинаковое приветствие с предложением выполнить несколько команд, которые (1) инициируют на машине мастер-ноду, (2) создадут сеть внутри кластера и (3) установят приложение "nginx".

5. В данном случае мы будем создавать кластер с одной мастер-нодай (хотя можно и больше) и 4-мя рабочими нодами.  Поэтому:

   - На первой ноде запустите команду (`Ctrl + Shift + С` для вставки):
     
     ```bash
     kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr 10.5.0.0/16
     ```
     
     Это приведёт к [инициализации](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/) на текущей машине управляющих компонент кластера ([Control Plane](Control Plane)). Т.е. эта машина станет мастером и, в дальнейшем, с неё можно будет управлять всем кластером. В качестве ip-адреса для доступа к API кластера назначается ip-адрес текущей машины (ключ `--apiserver-advertise-address`). Кроме того, указывается (при помощи ключа `--pod-network-cidr`), что всем созданным в кластере подам следует выдавать ip-адреса из подсети (`10.5.0.0/16`).
     
   - После завершения процесса инициализации вас попросят указать путь к конфиг-файлу кластера. По умолчанию, kubernetes ищет конфиг в рабочем каталоге пользователя в папке `~/.kube/config` или в переменной окружения `KUBECONFIG`.  
     Мы пропускаем этот этап, т.к. kubernetes сам создал в нашем рабочем каталоге файл с конфигом, но это произошло потому, что мы запустили процедуру установки из под этого пользователя, и если бы мы хотели передать возможность работать с нашим кластером другому пользователю, то конфиг нужно было бы скопировать ему в рабочий каталог. Т.е. под выбранным для управления кластером пользователем нужно было бы выполнить команды:  
     
     ```bash
     mkdir -p $HOME/.kube
     sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
     sudo chown $(id -u):$(id -g) $HOME/.kube/config
     ```
     
   - Здесь же будет показана строка которую нужно будет запускать на рабочих узлах для подключения к кластеру. Найдите её и скопируйте в какой ни будь текстовый файл, она нам ещё пригодится. В моём случае она выглядит так:  
     
     ```bash
     kubeadm join 192.168.0.13:6443 --token ixsi7m.i6cnbvip7489gku1 \
         --discovery-token-ca-cert-hash sha256:73d00e6775ed529cfb35992629bb2b4fc11a81e6dbe1404232095900b6fe7648 
     ```
     
     Здесь просто указан ip-адрес управляющего сервера, токен доступа к кластеру и хэш корневого сертификата.
     
   - На мастер-ноде запустите команду:
     
     ```bash
     kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml
     ```
     
     Она приведёт к инициализации сети внутри кластера. На самом деле будет создан ряд элементов кластера прописанных в yaml-манифесте указанном в файле по [ссылке](https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml).

   Теперь у нас есть полностью готовая к работе мастер-нода.

6. Выполните команду:  
   ```bash
   kubectl get nodes
   ```

   В результате вы увидите список подключённых к кластеру нод. В нашем случае, в списке, пока-что доступна только одна нода.

7. Выполните команду:

   ```bash
   kubectl get pods -A
   а затем
   kubectl get pods
   ```

   В результате вы увидите полный (`-A`) список подов запущенных в кластере и только те поды, которые находятся в пространстве имён "default". В первом приближении поды можно воспринимать как аналог процессов на обычном компьютере.  
   Все поды распределяются по пространствам имён (namespaces) полный список которых можно посмотреть при помощи команды: `kubectl get namespaces`. Если команды выполняется без явного указания пространства имён (ключ `--namespace`), то подразумевается пространство имён "default".

8. Подключим остальные сервера к кластеру.  
   Для этого на оставшихся 4-х нодах запустите команду `kubeadm join` с параметрами, которые вы копировали ранее в текстовый файл. В моём случае команда выглядит так:  

   ```bash
   kubeadm join 192.168.0.13:6443 --token ixsi7m.i6cnbvip7489gku1 \
       --discovery-token-ca-cert-hash sha256:73d00e6775ed529cfb35992629bb2b4fc11a81e6dbe1404232095900b6fe7648 
   ```

9. Перейдите на мастер-ноду и снова выполните команду:  
   ```bash
   kubectl get nodes
   ```

   В результате вы должны увидеть, что в списке теперь 5 нод.

10. Выполните команду:   
    ```bash
    kubectl describe nodes node1
    ```
    
    Данная команда позволит вам получить подробную информацию по ноде с именем "node1".


11. Выполните команду:  
    ```bash
    kubectl get pods -o wide -A
    ```

    В результате вы должны получить список всех подов с расширенной информацией по каждому. Обратите внимание, что некоторые поды автоматически запустились на новых нодах.

12. Выполните команду:  
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/application/nginx-app.yaml
    ```

    В результате будет запущено 3 пода с сервером "nginx". Проверьте это при помощи команды `kubectl get pods -o wide`. В столбце "Status" должно быть указано "Running".

13. Обычно поды "заперты" во внутренней сети кластера и снаружи к ним доступа нет. Но командой выше мы не только запустили 3 пода с  "nginx", но и создали элемент кластера "Service", который пробросил некоторый внешний порт на 80й порт внутри кластера. Т.к. "nginx" - это веб-сервер то он слушает как раз 80й порт. Определим, какой же порт проброшен.  
    Для этого введите команду:  

    ```bash
    kubectl get svc
    ```

    Эта команда покажет вам список элементов типа "Service". Найдите в нём сервис типа "LoadBalancer" с названием "my-nginx-svc" и в столбце "Port" определите внешний порт (у меня это 31515).

14. Проверим, что "nginx" работает при помощи утилиты `curl`:  
    ```bash
    curl 192.168.0.18:31515
    ```

    Здесь `192.168.0.18` - это ip-адрес моей мастер-ноды, а 31515 - проброшенный внутрь порт.  
    В результате вы должны увидеть код html-страницы с приветствием от "nginx".

15. Отключим одну или несколько нод на которых работают поды с "nginx".  Процесс корректного отключения ноды от реального кластера может содержать больше шагов, чем приведены здесь:

    - Определите на каких нодах запущены поды с "nginx" (у меня это node4, node5, node2).

    - Введите команду (со своим именем ноды):  
      ```bash
      kubectl drain node4 --ignore-daemonsets --delete-local-data
      ```

      Данная команда проинформирует ноду о том, что нужно завершить все свои поды и в результате они будут автоматически запущены на оставшихся нодах.

    - Проверьте список доступных нод в кластере.

16. Остановим запущенное приложение "nginx". Для этого выполните команду запуска, но вместо `applay` укажите `delete`, т.е.:  
    ```bash
    kubectl delete -f https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/application/nginx-app.yaml
    ```

17. Проверьте список запущенных подов - он должен быть пустым.

<br>

## Полезные ссылки

1. [Введение в Kubernetes на примере Minikube](https://youtu.be/sLQefhPfwWE) - полноценная лекция по kubernetes. Теор. материал чередуется с практикой.
2. [Kubernetes Уроки](https://youtube.com/playlist?list=PL3SzV1_k2H1VDePbSWUqERqlBXIk02wCQ) - довольно большой плей-лист по kubernetes. Небольшие видео в основном состоящие из практики;
3. [Шпаргалка по kubectl](https://kubernetes.io/ru/docs/reference/kubectl/cheatsheet/) - примеры команды по управлению кластером kubernetes.

