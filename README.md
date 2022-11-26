# Systemd
___
**Systemd** — это менеджер системы и служб Linux. Он инициализирует другие демоны в системе при загрузке. Основная цель systemd — полный контроль над запуском и выполнением процессов, описанных в его конфигурации.

>При старте Systemd запускает параллельно столько служб, сколько возможно в зависимости от стадии загрузки. Это заметно ускоряет запуск системы. Если замерить время от включения до входа, то systemd использует его гораздо меньше, чем предшественники — например, SystemV.

Systemd — универсальный менеджер. Он управляет практически всей работающей системой. Systemd отвечает за работу запущенных служб и может рассказать все об их состоянии. Он также управляет монтированием файловой системы, аппаратной частью, процессами.

Systemd использует скомпилированные бинарные файлы. Все конфигурационные файлы открыты. Их можно менять через командную строку или GUI. При необходимости можно добавить свои конфигурационные файлы.
___

## **Архитектура systemd**

На этой схеме изображены компоненты, которые входят в systemd. Это упрощенное представление, которое показывает только основную архитектуру. В нем размещены не все файлы и службы. Здесь также нет информации о сложном потоке данных.

![alt text](https://highload.today/wp-content/uploads/2021/12/image1-5.png)
 
___
**Возможности systemd**

В зависимости от используемых при компиляции параметров systemd включает до 69 файлов. С их помощью выполняются следующие задачи:

#### Systemd как PID 1 осуществляет запуск стольких служб в параллельном режиме, сколько ему нужно. Благодаря этому загрузка ускоряется.

> В каждой Unix системе есть один процесс с специальным идентификатором 1. Он запускается ядром перед всеми последующими процессами, и он является также родительским процессом для всех последующих/остальных процессов кто не смог найти себе родителя. Вследствие, он может делать много того чего не могут позволить себе другие процессы. Также он несет ответственность за некоторые вещи за которые не отвечают другие процессы, например, за поднятие и поддержку userspace во время загрузки системы.
>> User Space - адресное пространство, где выполняются обычные пользовательские процессы (то есть все, кроме ядра)

#### Systemd создает журналы для хранения системных логов и дает инструменты для управления записями.
> Systemd предоставляет решение централизованного управления для регистрации всех процессов ядра и пользовательской области. Система, собирающая эти журналы и управляющая ими, называется журнальной системой.  
journald

#### Systemctl дает пользовательский интерфейс для управления службами.

#### Управление службами и логи дают информацию о состоянии служб.

#### Можно управлять сокетами.
* Сокеты — это прямая связь между двумя процессами.В некотором смысле сокеты — это сеть, полностью содержащаяся в ядре; вместо того, чтобы использовать сетевые интерфейсы и соответствующие накладные расходы для отправки данных, те же самые данные могут быть отправлены напрямую между программами.Сокеты просто предоставляют фактическое оборудование для перемещения данных.

#### Таймеры предоставляют расширенные возможности для планирования, включая запуск скриптов по времени от старта системы.
* Таймеры, как и задания cron, могут, в заданное время, вызывать выполнение различных действий в системе. Например — запуск скриптов командной оболочки или программ. Таймеры могут срабатывать, например, раз в день, причём — только по понедельникам. Ещё один пример — срабатывание таймера каждые 15 минут в рабочее время (с 8 утра до 6 вечера). Но таймеры systemd могут кое-что такое, что недоступно заданиям cron. Например, таймер может вызвать скрипт или программу через заданное время после некоего события. Таким событием может быть загрузка системы или запуск systemd, завершение предыдущей задачи или даже завершение работы сервиса, вызванного ранее по таймеру.

#### Можно монтировать и размонтировать файловые системы с иерархическим уведомлением для безопасного каскадирования.
* systemd.mount — монтирование. Systemd полностью отвечает за монтирование разделов и файловых систем, описанных в файле /etc/fstab. systemd-fstab-generator(8) преобразует записи из /etc/fstab в юниты systemd; это выполняется при каждой загрузке системы, а также при перезагрузке конфигурации системного менеджера.
systemd расширяет возможности fstab и предлагает дополнительные опции монтирования. Они могут влиять на зависимости юнита монтирования: например, могут гарантировать, что монтирование выполняется только после подключения к сети или после монтирования другого раздела. 
Примером этих опций может быть автомонтирование (здесь имеется в виду не автоматическое монтирование во время загрузки, а монтирование при появлении запроса от устройства). 
>ПРИМЕРЫ:  
Если есть большой раздел, вы можете разрешить службам, которые не обращаются к нему, запускаться в то время, как он проверяется программой fsck. Для этого добавьте следующие параметры монтирования в запись /etc/fstab для соответствующего раздела:  
 **noauto,x-systemd.automount**  
При этом процедура проверки и монтирования раздела будет запущена только при первой попытке доступа, и ядро будет держать в ожидании все потоки ввода-вывода, связанные с этим разделом, пока он не будет смонтирован. Это полезно в случае, например, очень большого раздела /home.  
*Примечание: Разделу будет присвоен тип файловой системы autofs, который по умолчанию игнорируется mlocate. Используйте эту возможность с осторожностью.*  
***Удалённая файловая система***  
Автоматическое монтирование может аналогичным образом использоваться и для монтирования удаленных файловых систем. В дополнение, вы можете использовать параметр x-systemd.device-timeout=# для указания времени ожидания удаленной файловой системы при перебоях в соединении. Также опция _netdev даёт systemd понять, что файловая система зависит от сети.  
noauto,x-systemd.automount,x-systemd.mount-timeout=30,_netdev  
***Автоматическое размонтирование***  
С помощью флага x-systemd.idle-timeout можно указать таймаут бездействия. Например:  
noauto,x-systemd.automount,x-systemd.idle-timeout=1min  
В таком случае systemd размонтирует раздел спустя одну минуту неактивности.

#### Можно создавать временные файлы и управлять ими, в том числе удалять.
> systemd-tmpfiles - этот инструмент предоставляет структурированный и настраиваемый метод для управления временными каталогами и файлами.Когда сервисный модуль systemd-tmpfiles-setup запускается, он запускает команду   systemd-tmpfiles –create –remove.  
Команда проверяет файлы конфигурации из:  
/usr/lib/tmpfiles.d/.conf  
/run/tmpfiles.d/.conf  
/etc/tmpfiles.d/*.conf  
Если в указанных выше файлах конфигурации есть файлы и каталоги, помеченные для удаления, они будут удалены.
Файлы и каталоги, помеченные для создания создаются с правильными разрешениями, если это необходимо.

#### Можно запускать скрипты при подключении или отключении устройств.  
> Например создать .target который запустится после multi-user.target и запустит скрипт через ExecStart=


Можно через анализ последовательности загрузки **systemd-analyze** найти службы, запуск которых отнимает больше всего ресурсов или вызывает сбои.
Эти и другие задачи решаются различными демонами, которые управляют программами и конфигурационными файлами в составе пакета systemd.

## **Юниты systemd**

ЮНИТ — ЭТО ОПИСАНИЕ СЕРВИСА В ТЕКСТОВОМ ВИДЕ. В НЕМ УКАЗАНЫ ОПЕРАЦИИ, КОТОРЫЕ ВЫПОЛНЯЮТСЯ ДО И ПОСЛЕ ЗАПУСКА СЛУЖБЫ. ПО СУТИ, ЭТО ОПИСАНИЕ ПАРАМЕТРОВ ИНИЦИАЛИЗАЦИИ.
Все юниты разложены по трем каталогам:
1.	/usr/lib/systemd/system/ — юниты из установленных пакетов RPM/DEB. Например, Nginx, MySQL, Apache.
2.	/run/systemd/system/ — юниты, созданные в рантайме.
3.	/etc/systemd/system/ — юниты, созданные системными администраторами.
Посмотреть список всех запущенных юнитов можно командой:
> systemctl

В терминале отобразится также статус каждой службы. Основные параметры:
> UNIT — название юнита.  
> LOAD — информация об успешной загрузке конфигурации.  
> ACTIVE — сообщение о статусе. Может быть также INACTIVE, LOADED и другие.  
> SUB — детальная системная информация об юните.  
> DESCRIPTION — краткое описание юнита.  

#### Юниты бывают разных типов. Например, юнит службы имеет тип *.service. 

Все виды:
> * .service -	Описывает, как управлять службой или приложением на сервере.
> * .socket -	Описывает сетевой, IPC-сокет или FIFO-буфер, который используется для активации сокета.
> * .device -	Описывает устройство, указанное как необходимое для управления systemd с помощью udev или файловой системы sysfs.
> * .mount -	Определяет точку монтирования в системе, которой управляет systemd.
> * .automount -	Настраивает точку монтирования, которая будет автоматически установлена.
> * .swap -	Описывает пространство подкачки в системе (swap-файл).
> * .target -	Обеспечивает точки синхронизации для других устройств при загрузке или изменении состояний.
> * .path -	Определяет путь, который может использоваться для активации на основе пути.
> * .timer -	Определяет таймер, который управляется systemd для задержки или активации по плану.
> * .snapshot -	Позволяет восстановить текущее состояние системы после изменений.
> * .slice -	Связан с узлами группы управления Linux, что позволяет ограничить ресурсы или назначить их для любых процессов, связанных с slice.
> * .scope -	Создается автоматически systemd из информации, которую получил от интерфейса шины.

### Рассмотрим *.service

Чтобы отфильтровать их необходимо выполнить команду 
> systemctl list-units --type=service  

Также можно выбрать только неактивные службы
> systemctl list-units --all --state=inactive

Посмотреть все возможные статусы можно:
> systemctl --state=help

## **Структура юнита systemd**

Структура может показаться сложной. Но на самом деле она строгая и очень логичная. Каждый юнит представляет собой текстовый файл. Внутри него — обязательные секции и переменные.

Для наглядности посмотрите на пример юнита sshd:

```[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.target
Wants=sshd-keygen.target
 
[Service]
Type=notify
EnvironmentFile=-/etc/crypto-policies/back-ends/opensshserver.config
EnvironmentFile=-/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS $CRYPTO_POLICY
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=42s
 
[Install]
WantedBy=multi-user.target
```

Давайте разберем все секции.

**[Unit]**


Первая обязательная секция, в которой описываются метаданные службы и правила взаимодействия с другими службами. В ней доступны следующие переменные:

> * Description — короткое описание демона.
> * Documentation — страница man, на которой расписан порядок взаимодействия со службой.
> * After — указание на то, после каких демонов и событий демон запускается. Например, юнит Nginx поднимается после запуска сетевых интерфейсов. Можно указать целую группу других сервисов.
> * Requires — какой сервис необходим для запуска юнита.
> * Wants — какой сервис желательно запустить перед стартом юнита.
Если указать сервис в Requires, но не прописать его в After, то он будет запущен параллельно с юнитом, который вы настраиваете.

Пример оформления секции Unit:

```[Unit]
Description=MyUnit
After=syslog.target
After=network.target
After=nginx.service
After=mysql.service
Requires=mysql.service
Wants=redis.service
```
**[Service]**

Вторая обязательная секция для описания конфигурации. Здесь вы указываете, какими командами и под каким пользователем запускать сервис.

> * Type — описывает, как запустится демон. Есть несколько вариантов: simple (по умолчанию) — systemd ожидает, что служба запустится незамедлительно. Процесс не должен разветвляться. forking — после запуска демон ответвляется (делает форк), родительский процесс завершается. Такой подход используется для запуска классических демонов. one-shot — одноразовое выполнение. Используется для скриптов, которые запускаются и завершаются после выполнения. notify — аналог simple, но в этом случае сам процесс сообщит systemd о том, что он закончил загрузку и готов к работе.
> * PIDFile — ссылка на основной процесс, который отслеживает systemd.
> * WorkingDirectory — рабочая директория. Становится текущей перед запуском стартап команд.
> * User — пользователь, под которым надо запускать сервис.
> * Group — группа, под которой надо запускать сервис.
> * Environment — переменная окружения.
> * OOMScoreAdjust — запрет на убийство сервиса из-за нехватки памяти или срабатывания механизма OOM (-1000 — полный запрет).
> * ExecStart — команда для старта сервиса.
> * ExecReload — команда для перезапуска сервиса.
> * ExecStop — команда для остановки сервиса.
> * TimeoutSec — время в секундах, которое сервис ожидает отработки старт- или стоп-команд.
> * Restart — настройки перезапуска.(например настройка перезапуска при непрдевиденном сбое сервиса)

*Пример оформления секции Service:

```[Service]
Type=forking
PIDFile=/work/www/myunit/shared/tmp/pids/service.pid
WorkingDirectory=/work/www/myunit/current
User=myunit
Group=myunit
Environment=RACK_ENV=production
OOMScoreAdjust=-1000
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/bundle exec service -C /work/www/myunit/shared/config/service.rb --daemon
ExecStop=/usr/local/bin/bundle exec service -S /work/www/myunit/shared/tmp/pids/service.state stop
ExecReload=/usr/local/bin/bundle exec service -S /work/www/myunit/shared/tmp/pids/service.state restart
TimeoutSec=300
```
**[Install]**

Третья обязательная секция. В ней описывается на каком уровне запуска стартует настраиваемый сервис.

Переменная WantedBy сообщает, как устройство включится. Например, multi-user.target означает, что при запуске в директории /etc/systemd/system появится каталог multi-user.target.wants. В нем будет ссылка на службу, которая удалится после остановки службы.
> Target(цели)  
На каждом уровне – запускаются или останавливаются определённые службы.  
Файлы целей systemd .target предназначены для группировки вместе других юнитов systemd через цепочку зависимостей. Например юнит graphical.target, использующийся для старта графической сессии, запускает системные сервисы GNOME Display Manager (gdm.service) и Accounts Service (accounts–daemon.service) и активирует multi–user.target. В свою очередь multi–user.target запускает другие системные сервисы, такие как Network Manager (NetworkManager.service)

Пример оформления секции Install:
```
[Install]
WantedBy=multi-user.target
```
Перечень доступных параметров указан в руководстве. Вызвать его можно командой:

> man systemd.unit  

Для определения того, какие сервисы и в каком порядке будут загружены, используется Target(цель). Его основные виды:

1.  poweroff – отключение системы;
2.  rescue – режим восстановления, однопользовательский (init 1);
3.  multi-user – сетевой режим без графической оболочки, (init 3);
4.  graphical – сетевой режим с графической оболочкой (init 5);
5.  reboot – перезагрузка;
6.  emergency – аварийная командная строка, минимальный функционал.

Так же цели можно посмотреть:
```
ls -l /usr/lib/systemd/system/runlevel*
/usr/lib/systemd/system/runlevel0.target -> poweroff.target
/usr/lib/systemd/system/runlevel1.target -> rescue.target
/usr/lib/systemd/system/runlevel2.target -> multi-user.target
/usr/lib/systemd/system/runlevel3.target -> multi-user.target
/usr/lib/systemd/system/runlevel4.target -> multi-user.target
/usr/lib/systemd/system/runlevel5.target -> graphical.target
/usr/lib/systemd/system/runlevel6.target -> reboot.target
```
Цели могут наследоваться. Пример взаимодействия с ними:

* Посмотреть список целей
> systemctl list-units --type=target

* Перейти в нужную цель (например – загрузится из сетевого режима в графический)
> systemctl isolate graphical.target

* Посмотреть target(цель) по умолчанию
> systemctl get-default

* Выбрать target(цель) по умолчанию
>systemctl set-default multi-user.target

## **Редактирование юнитов**

Юниты нельзя редактировать напрямую. Для этого используется команда edit:

>systemctl edit --full nginx.service

Теперь можно добавить в описание юнита несколько переменных. Например, измените секцию Service. Чтобы сервис автоматически перезапускался в случае сбоя через 60 секунд.  
RestartSec=60sпозволяет указать время простоя в секундах, после которого systemd перезапускает службу после аварии (по умолчанию 5 секунд).  
Restart определяет, при каких условиях необходимо перезапускать службы. Значение по умолчанию ―on-failure. Это означает, что службу необходимо перезапустить, если она завершила работу с ненулевым кодом выхода.


```
[Service]
Restart=on-failure 
RestartSec=60s
```

Чтобы применить изменения, необходимо перечитать файлы systemd:

> systemctl daemon-reload

При работе с файл-юнитами systemd вы рискуете оказаться перегруженными опциями.
Каждый файл-юнит может быть настроен с различными параметрами. Чтобы выяснить, какие параметры доступны для конкретного юнита, используйте команду:
> systemctl show unit_name.service

 Например, команда **systemctl show sshd** показывает все параметры systemd, которые можно настроить в юните sshd.service, включая их текущие значения по умолчанию.
```
Id=sshd.service
Names=sshd.service
Requires=basic.target
Wants=sshd-keygen.service system.slice
WantedBy=multi-user.target
ConsistsOf=sshd-keygen.service
Conflicts=shutdown.target
ConflictedBy=sshd.socket
Before=shutdown.target multi-user.target
After=network.target sshd-keygen.service systemd-journald.socket
basic.target system.slice
Description=OpenSSH server daemon
LoadState=loaded
ActiveState=active
SubState=running
FragmentPath=/usr/lib/systemd/system/sshd.service
UnitFileState=enabled
InactiveExitTimestamp=Sat 2015-05-02 11:06:02 EDT
InactiveExitTimestampMonotonic=2596332166
ActiveEnterTimestamp=Sat 2015-05-02 11:06:02 EDT
ActiveEnterTimestampMonotonic=2596332166
ActiveExitTimestamp=Sat 2015-05-02 11:05:22 EDT
ActiveExitTimestampMonotonic=2559916100
InactiveEnterTimestamp=Sat 2015-05-02 11:06:02 EDT
InactiveEnterTimestampMonotonic=2596331238
CanStart=yes
CanStop=yes
CanReload=yes
CanIsolate=no
StopWhenUnneeded=no
RefuseManualStart=no
RefuseManualStop=no
AllowIsolate=no
DefaultDependencies=yes
OnFailureIsolate=no
IgnoreOnIsolate=no 
```

## Создание юнитов  
Вы также можете создавать свои юниты. Например, вы написали приложение на Python и хотите добавить его в виде сервиса, которым будете управлять через systemd. В качестве примера создайте юнит item.service:

> vim /etc/systemd/system/test.service  

В описании юнита добавьте обязательные секции:
```
[Unit] 
Description=test service 
[Service] 
Type=oneshot ExecStart=/bin/echo "Hello World!" RemainAfterExit=yes
[Install]
WantedBy=multi-user.target 
```
Теперь нужно применить изменения в конфигурации:

> systemctl daemon-reload  

Следующий шаг — запуск юнита:

> systemctl start test.service  

Напоследок проверьте его статус:

> systemctl status test.service  

## КОМАНДЫ systemctl  

First Header  | Second Header
------------- | -------------
Content Cell  | Content Cell
Content Cell  | Content Cell
