### Настройка и использование сервиса rsyslog в ОС «Альт»
#### 1. Сервисы журнализации
В ОС «Альт» используется двухуровневая система журналирования, включающая systemd-journald и rsyslogd.

systemd-journald — служба, собирающая и хранящая журналы в структурированном бинарном формате. Она получает сообщения от ядра, от ранних этапов загрузки и от служб systemd, а также пересылает их в rsyslog для текстового хранения .

rsyslogd — демон текстовой журнализации, записывающий сообщения в текстовые файлы в каталоге /var/log. Он принимает сообщения от journald (при включённой пересылке) и от приложений, использующих syslog API .

Схема взаимодействия
```text
Ядро / Службы / Приложения
         │
         ▼
   systemd-journald
         │
         ▼ (ForwardToSyslog=yes)
      rsyslogd
         │
         ▼
  /var/log/*.log
```
Для включения пересылки сообщений из journald в rsyslog необходимо в файле /etc/systemd/journald.conf установить параметры :

```ini
ForwardToSyslog=yes
MaxLevelSyslog=debug
```
После изменения конфигурации перезапустить службу:

```bash
systemctl restart systemd-journald
```
Проверить работу можно, сгенерировав тестовое сообщение и найдя его в системном журнале :

```bash
logger -t TEST "ForwardToSyslog is now enabled"
grep TEST /var/log/messages
```
#### 2. Текстовая журнализация средствами rsyslogd
##### 2.1 Конфигурационные файлы
Обработка сообщений осуществляется на основании правил, описанных в конфигурационных файлах :

/etc/rsyslog.conf — основной конфигурационный файл, содержащий секции Modules, Configuration Directives, Templates и Rule Line;

/etc/rsyslog.d/*.conf — дополнительные правила. Рекомендуется размещать пользовательские правила именно в этом каталоге .

##### 2.2 Категории и уровни важности
Система журналирования syslog использует две ключевые категории :

Фасилити (facility) — источник сообщения:

Код	Название	Описание
0	kern	Сообщения ядра
1	user	Пользовательские процессы
2	mail	Почтовая система
3	daemon	Системные демоны
4	auth	Безопасность и авторизация
9	cron	Планировщик задач
10	authpriv	Приватные сообщения авторизации
16–23	local0–local7	Пользовательские категории

Уровень важности (priority):

Уровень	Приоритет	Описание
0	emerg	Система неработоспособна
1	alert	Требуется немедленное вмешательство
2	crit	Критическая ошибка
3	err	Ошибка
4	warning	Предупреждение
5	notice	Важное событие
6	info	Информационное сообщение
7	debug	Отладочная информация

При указании уровня обрабатываются все сообщения этого и более высокого уровня. Например, *.info обрабатывает сообщения info, notice, warning, err, crit, alert, emerg .

##### 2.3 Синтаксис правил
rsyslog поддерживает два типа синтаксиса :

Классический синтаксис: facility.priority action

RainerScript (рекомендуется для сложных правил):

```text
if $programname == 'sshd' then /var/log/sshd.log
```
Специальные операторы:

Оператор	Значение
*	Все фасилити или все приоритеты
.	Указанный уровень и выше
.=	Только указанный уровень
.!	Исключить указанный уровень
none	Отключить логирование
Примеры правил :

```text
# сообщения подсистемы local0
local0.*        /var/log/demo-local0.log

# предупреждения и ошибки подсистемы local1
local1.warning  /var/log/demo-warning.log

# только ошибки подсистемы local2
local2.err      /var/log/demo-error.log

# все сообщения, кроме debug
*.!debug        /var/log/all-but-debug.log
```
##### 2.4 Действия (Actions)
После обработки правила rsyslog выполняет действие :

Запись в файл: authpriv.* /var/log/secure (при указании абсолютного пути файл создаётся автоматически);

Асинхронная запись: authpriv.* - /var/log/secure (дефис означает без sync());

Пересылка на удалённый сервер: *.* @192.168.1.100:514 (UDP) или *.* @@192.168.1.100:514 (TCP);

Отбрасывание сообщений: kern.debug ~.

#### 3. Ротация логов
Журналы, создаваемые rsyslog, могут быстро увеличиваться, поэтому используется механизм ротации logrotate .

##### 3.1 Принцип работы
При ротации текущий лог-файл переименовывается (например, /var/log/syslog → /var/log/syslog.1), а новый файл создаётся на его месте. Архивные журналы обычно сжимаются в .gz для экономии дискового пространства .

##### 3.2 Конфигурационные файлы
/etc/logrotate.conf — основной файл, содержит глобальные настройки и подключает правила из каталога /etc/logrotate.d/ ;

/etc/logrotate.d/* — правила для отдельных служб (nginx, rsyslog, apt и др.).

##### 3.3 Основные параметры logrotate
Опция	Описание
daily / weekly / monthly	Частота ротации
size 100M	Ротация при превышении указанного размера
rotate 4	Хранить указанное количество архивных файлов
compress	Сжимать архивные журналы (gzip)
delaycompress	Отложить сжатие на один цикл ротации
create 640 root adm	Создавать новый файл с указанными правами
missingok	Игнорировать отсутствующие файлы
notifempty	Не ротировать пустые файлы
postrotate ... endscript	Выполнить команды после ротации

##### 3.4 Пример конфигурации
```bash
/var/log/*.log
{
    weekly
    rotate 4
    compress
    missingok
    notifempty
    create 0640 root adm
    postrotate
        systemctl reload rsyslog > /dev/null 2>&1 || true
    endscript
}
```
В данном примере журналы ротируются еженедельно, хранится 4 архивных файла, старые журналы сжимаются, после ротации перезагружается служба rsyslog .

##### 3.5 Проверка и запуск
```bash
# Проверка конфигурации (без выполнения ротации)
logrotate -d /etc/logrotate.conf

# Подробный запуск
logrotate -v /etc/logrotate.conf

# Принудительная ротация
logrotate -f /etc/logrotate.conf
```
По умолчанию logrotate запускается автоматически один раз в сутки через cron или таймер systemd. Информация о последних ротациях хранится в /var/lib/logrotate/status .

#### 4. Централизация сбора событий
Централизованное журналирование позволяет собирать журналы с нескольких узлов на одном сервере .

##### 4.1 Преимущества
Упрощение анализа событий;

Централизованный аудит безопасности;

Сохранение журналов даже при отказе узла;

Упрощение мониторинга инфраструктуры.

##### 4.2 Настройка сервера-приёмника
##### Шаг 1. В файле /etc/rsyslog.d/00_common.conf раскомментировать строки для приёма сообщений :

```text
# для UDP
module(load="imudp")
input(type="imudp" port="514")

# для TCP
module(load="imtcp")
input(type="imtcp" port="514")
```
##### Шаг 2. Создать шаблон для хранения журналов в /etc/rsyslog.d/myrules.conf :

```text
$template remote-incoming-logs,"/var/log/%HOSTNAME%/%PROGRAMNAME%.log"
*.* action(type="omfile" dynaFile="remote-incoming-logs" dirCreateMode="0755" fileCreateMode="0644")
```
Согласно этому шаблону все сообщения сохраняются в файлы /var/log/<имя_узла>/<приложение>.log.

##### Шаг 3. Перезапустить службу:

```bash
systemctl restart rsyslog
```
##### 4.3 Настройка клиента-отправителя
Создать файл /etc/rsyslog.d/all.conf :

```text
*.* @@192.168.0.111:514 # Отправка логов на сервер (TCP)
Обозначения:

@@ — передача по TCP;

@ — передача по UDP;

192.168.0.111 — IP-адрес сервера журналирования.
```
После перезапуска rsyslog на сервере в каталоге /var/log/ появится каталог с именем узла-отправителя :

```text
# ls -l /var/log/node02/
-rw------- 1 root adm  810 апр  3 12:30 crond.log
-rw------- 1 root adm 1194 апр  3 12:19 login.log
-rw------- 1 root adm 2509 апр  3 12:19 rsyslogd.log
```
#### 5. Практическая работа: Примеры
##### Пример 1. Настройка пользовательских правил журналирования
Цель: направить сообщения разных подсистем в отдельные файлы.

##### Шаг 1. Создать файл /etc/rsyslog.d/demo.conf :

```text
# сообщения подсистемы local0
local0.*        /var/log/demo-local0.log

# предупреждения и ошибки подсистемы local1
local1.warning  /var/log/demo-warning.log

# только ошибки подсистемы local2
local2.err      /var/log/demo-error.log
```
##### Шаг 2. Проверить корректность конфигурации:

```bash
rsyslogd -N1
```
##### Шаг 3. Перезапустить rsyslog:

```bash
systemctl restart rsyslog
```
##### Шаг 4. Сгенерировать тестовые сообщения :

```bash
logger -p local0.info "DEMO: info message from local0"
logger -p local1.warning "DEMO: warning message from local1"
logger -p local2.err "DEMO: error message from local2"
```
##### Шаг 5. Проверить наличие записей:

```bash
cat /var/log/demo-local0.log
cat /var/log/demo-warning.log
cat /var/log/demo-error.log
```
##### Пример 2. Настройка ротации журнала для конкретной службы
Цель: настроить ежедневную ротацию журналов с хранением 7 архивов.

##### Шаг 1. Создать файл /etc/logrotate.d/usbguard :

```text
/var/log/usbguard/*log {
    missingok
    notifempty
    rotate 7
    create 0600 root root
    delaycompress
    daily
    compress
}
```
##### Шаг 2. Проверить корректность конфигурации:

```bash
logrotate -d /etc/logrotate.d/usbguard
```
Ключ -d выполняет проверку без фактической ротации файлов .

##### Шаг 3. Для немедленного применения:

```bash
logrotate -f /etc/logrotate.d/usbguard
```
##### Пример 3. Централизованный сбор логов
Сценарий: сервер с IP 192.168.0.111 собирает журналы с клиентского узла node02.

На сервере:

Установить пакет rsyslog-classic (при необходимости):

```bash
apt-get install rsyslog-classic
```
В /etc/rsyslog.d/00_common.conf раскомментировать:

```text
module(load="imudp")
input(type="imudp" port="514")
module(load="imtcp")
input(type="imtcp" port="514")
```
Создать /etc/rsyslog.d/myrules.conf:

```text
$template remote-incoming-logs,"/var/log/%HOSTNAME%/%PROGRAMNAME%.log"
*.* action(type="omfile" dynaFile="remote-incoming-logs" dirCreateMode="0755" fileCreateMode="0644")
```
Перезапустить:

```bash
systemctl restart rsyslog
```
На клиенте (node02):

Создать /etc/rsyslog.d/all.conf:

```text
*.* @@192.168.0.111:514
```
Перезапустить:

```bash
systemctl restart rsyslog
```
Проверка на сервере:

```bash
ls -l /var/log/node02/
```
Должны появиться файлы crond.log, login.log, rsyslogd.log .

##### Пример 4. Проверка взаимодействия journald и rsyslog
Цель: убедиться, что сообщения из journald попадают в текстовые журналы rsyslog.

##### Шаг 1. В /etc/systemd/journald.conf установить:

```ini
ForwardToSyslog=yes
MaxLevelSyslog=debug
```
##### Шаг 2. Перезапустить journald:

```bash
systemctl restart systemd-journald
```
##### Шаг 3. Сгенерировать тестовое сообщение :

```bash
logger -t TEST "ForwardToSyslog is now enabled"
```
##### Шаг 4. Проверить наличие в системном журнале:

```bash
grep TEST /var/log/messages
```
##### 6. Дополнительные сведения
Проверка синтаксиса rsyslog перед перезапуском: rsyslogd -N1 .

Файл состояния logrotate: /var/lib/logrotate/status содержит информацию о последних ротациях .

Таймер logrotate: для автоматического запуска должен быть активирован logrotate.timer:

```bash
systemctl enable --now logrotate.timer
```
Разделение механизмов: systemd-journald и текстовые журналы в /var/log используют разные механизмы хранения и очистки. Настройка journald рассматривается отдельно от logrotate
