# labaiorio_4 - HAPROXY
## Ansible.cfg
Это конфигурационный файл, который содержит несколько параметров для управления настройками работы с удаленными компьютерами.

Например, параметр "inventory" указывает на файл, в котором хранится список удаленных компьютеров, с которыми можно работать. Параметр ```"remote_user"``` указывает имя пользователя, которое будет использоваться для доступа к удаленным компьютерам.

Параметр ```"host_key_checking"``` отключает проверку ключей хостов при подключении к удаленным компьютерам, что может быть полезно для автоматического подключения без ввода паролей.

Параметр ```"transport"``` указывает на метод передачи данных между компьютерами, который будет использоваться по умолчанию.

## Iventory
Первая часть [web_servers] содержит список веб-серверов, которые нужно настроить. Каждый сервер имеет имя и IP-адрес: **web1 с IP-адресом 192.168.11.121** и **web2** с IP-адресом **192.168.11.122**.

Вторая часть [haproxy] содержит список серверов, на которых работает балансировщик нагрузки HAProxy. Каждый сервер также имеет имя и IP-адрес: haproxy1 с IP-адресом **192.168.11.123** и приоритетом 101, и haproxy2 с IP-адресом **192.168.11.124** и приоритетом 100.

Информация об IP-адресах и именах серверов используется Ansible для установки и настройки ПО на этих серверах, а также для управления ими в дальнейшем. Приоритет указывает на порядок запуска серверов в случае сбоя в работе.

## Роль - selina 
### Tasks 
Первая задача **(Install the latest version of libselinux-python3)** устанавливает последнюю версию библиотеки libselinux-python3 через менеджер пакетов yum.

Вторая задача **(Copy config)** копирует файл конфигурации config на сервер в директорию ```/etc/selinux/``` с правами владельца root и группы root. Также уведомляется задача (Reboot machine), которая перезагрузит сервер в случае необходимости.

Обе задачи помечены тегом selina, который позволяет запускать только эти задачи при необходимости. Это может быть полезно, если нужно настроить только определенные аспекты конфигурации сервера.

## Роль - haproxi 
- Первый обработчик **(Start and enabled haproxy)** запускает сервис **HAProxy через системный менеджер systemd** и включает его автозапуск при старте сервера (enabled: true). Если сервис уже запущен, то он будет перезапущен (state: restarted).
- Второй обработчик **(Start and enabled keepalived)** запускает сервис **Keepalived** через системный менеджер systemd и включает его автозапуск при старте сервера (enabled: true). Если сервис уже запущен, то он будет перезапущен (state: restarted).

Обработчики используются в Ansible для выполнения действий при наступлении определенных условий, например, при изменении конфигурации или при выполнении других задач. В этом примере обработчики вызываются при выполнении задач из других файлов, которые уведомляют об изменениях в конфигурации сервисов.

### Tasks 
Этот файл содержит задачи для автоматизации установки и настройки программного обеспечения **HAProxy и Keepalived** на сервере. **HAProxy - это открытый балансировщик нагрузки, который позволяет распределять трафик между несколькими серверами, а Keepalived - это программа для обеспечения отказоустойчивости и высокой доступности сервисов, работающих на нескольких серверах.**

- Первая задача устанавливает последнюю версию HAProxy, Keepalived и необходимый пакет для работы SELinux на сервере, используя управление пакетами Yum.
- Вторая задача создает конфигурационный файл для HAProxy, **используя шаблон haproxy.cfg.j2 и сохраняет его в /etc/haproxy/haproxy.cfg.** Также, после создания файла конфигурации, выполняется уведомление, чтобы запустить и включить HAProxy.
- Третья задача настраивает параметры ядра Linux, чтобы HAProxy мог использовать не локальные адреса для своей работы.
- Четвертая задача создает конфигурационный файл для Keepalived, используя шаблон ```keepalived.conf.j2``` и сохраняет его в ```/etc/keepalived/keepalived.conf.``` Также, после создания файла конфигурации, выполняется уведомление, чтобы запустить и включить Keepalived.

## haproxy.cfg.j2
Первые строки определяют настройки логирования, позволяя указать адрес, на котором будут записываться логи, а также где хранится файл с PID-ом процесса.
Затем устанавливаются некоторые общие настройки, такие как режим работы (http), опции для логирования, ограничения времени ожидания, количество максимальных соединений и т.д.
Далее настраивается frontend (точка входа), который будет слушать все запросы на порту 80 и направлять их на backend (точку выхода) с именем ***"app".***
Backend использует метод балансировки **нагрузки roundrobin** (каждый новый запрос будет отправлен на следующий сервер в очереди). Далее следует цикл, который настраивает каждый сервер, используя информацию о каждом сервере в группе ***"web_servers".*** Используя параметры, определенные в каждом хосте (IP-адрес и nginx_port), настройки сервера **генерируются динамически.**

## Keepalived
Cодержит настройки для использования протокола VRRP. Cекция vrrp_instance, которая настраивает параметры VRRP. В этой секции устанавливается интерфейс, на котором будет работать VRRP, **уникальный идентификатор виртуального маршрутизатора (virtual_router_id), приоритет (priority) и виртуальный IP-адрес (virtual_ipaddress). 

## Итог 
***HaProxy и Keepalived являются популярными инструментами для реализации балансировки нагрузки и обеспечения высокой доступности сервисов.

![1524885102581](https://user-images.githubusercontent.com/113093880/231297855-f44932fd-ff0d-4101-8f85-4c12cb7c2e18.jpg)
