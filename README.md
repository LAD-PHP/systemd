# Systemd
___
**Systemd** — это менеджер системы и служб Linux. Он инициализирует другие демоны в системе при загрузке. Основная цель systemd — полный контроль над запуском и выполнением процессов, описанных в его конфигурации.

>При старте Systemd запускает параллельно столько служб, сколько возможно в зависимости от стадии загрузки. Это заметно ускоряет запуск системы. Если замерить время от включения до входа, то systemd использует его гораздо меньше, чем предшественники — например, SystemV.

Systemd — универсальный менеджер. Он управляет практически всей работающей системой. Systemd отвечает за работу запущенных служб и может рассказать все об их состоянии. Он также управляет монтированием файловой системы, аппаратной частью, процессами.

Systemd использует скомпилированные бинарные файлы. Все конфигурационные файлы открыты. Их можно менять через командную строку или GUI. При необходимости можно добавить свои конфигурационные файлы.
___

## **Архитектура systemd**

На этой схеме изображены компоненты, которые входят в systemd. Это упрощенное представление, которое показывает только основную архитектуру. В нем размещены не все файлы и службы. Здесь также нет информации о сложном потоке данных.

![arch](/arch.png)
 
 > Подсистема оперирует специально оформленными файлами конфигурации — модулями (англ. unit). Каждый модуль отвечает за отдельно взятую службу, точку монтирования, подключаемое устройство, файл подкачки, виртуальную машину и тому подобные ресурсы. Существуют специальные типы модулей, которые не несут функциональной нагрузки, но позволяют задействовать дополнительные возможности systemd, к ним относятся модули типа target, slice, automount и ряд других.
___
**Возможности systemd**

В зависимости от используемых при компиляции параметров systemd включает до 69 файлов. С их помощью выполняются следующие задачи:

#### Systemd как PID 1 осуществляет запуск стольких служб в параллельном режиме, сколько ему нужно. Благодаря этому загрузка ускоряется.

> В каждой Unix системе есть один процесс с специальным идентификатором 1. Он запускается ядром перед всеми последующими процессами, и он является также родительским процессом для всех последующих/остальных процессов кто не смог найти себе родителя. Вследствие, он может делать много того чего не могут позволить себе другие процессы. Также он несет ответственность за некоторые вещи за которые не отвечают другие процессы, например, за поднятие и поддержку userspace во время загрузки системы.
>> **User Space** - адресное пространство, где выполняются обычные пользовательские процессы (то есть все, кроме ядра). Роль ядра состоит в том,чтобы управлять приложениями,работающими в этом пространстве.  
**Kernel Space** - это пространство памяти, где хранится и выполнятеся код ядра.

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

ПРИМЕРЫ: 

>* **Большой раздел**. Если есть большой раздел, вы можете разрешить службам, которые не обращаются к нему, запускаться в то время, как он проверяется программой fsck. Для этого добавьте следующие параметры монтирования в запись /etc/fstab для соответствующего раздела:  
 **noauto,x-systemd.automount**  
При этом процедура проверки и монтирования раздела будет запущена только при первой попытке доступа, и ядро будет держать в ожидании все потоки ввода-вывода, связанные с этим разделом, пока он не будет смонтирован. Это полезно в случае, например, очень большого раздела /home.  
*Примечание: Разделу будет присвоен тип файловой системы autofs, который по умолчанию игнорируется mlocate. Используйте эту возможность с осторожностью.

>* ***Удалённая файловая система***  
Автоматическое монтирование может аналогичным образом использоваться и для монтирования удаленных файловых систем. В дополнение, вы можете использовать параметр x-systemd.device-timeout=# для указания времени ожидания удаленной файловой системы при перебоях в соединении. Также опция _netdev даёт systemd понять, что файловая система зависит от сети.  
noauto,x-systemd.automount,x-systemd.mount-timeout=30,_netdev  

>* ***Автоматическое размонтирование***  
С помощью флага x-systemd.idle-timeout можно указать таймаут бездействия. Например:  
noauto,x-systemd.automount,x-systemd.idle-timeout=1min  
В таком случае systemd размонтирует раздел спустя одну минуту неактивности.

#### Можно создавать временные файлы и управлять ими, в том числе удалять.
> **systemd-tmpfiles** - этот инструмент предоставляет структурированный и настраиваемый метод для управления временными каталогами и файлами.Когда сервисный модуль systemd-tmpfiles-setup запускается, он запускает команду   systemd-tmpfiles –create –remove.  
Команда проверяет файлы конфигурации из:  
**/usr/lib/tmpfiles.d/.conf**  
**/run/tmpfiles.d/.conf**  
**/etc/tmpfiles.d/*.conf**  
Если в указанных выше файлах конфигурации есть файлы и каталоги, помеченные для удаления, они будут удалены.
Файлы и каталоги, помеченные для создания создаются с правильными разрешениями, если это необходимо.

#### Можно запускать скрипты при подключении или отключении устройств.  
> Например создать .target который запустится после multi-user.target и запустит скрипт через ExecStart=


#### Можно через анализ последовательности загрузки **systemd-analyze** найти службы, запуск которых отнимает больше всего ресурсов или вызывает сбои.  
> **systemd-analyze** - Для просмотра информации о количестве времени, которое затрачивается при загрузке системы с разбивкой на ядро и пользовательское пространство.
```
root@os3:~# systemd-analyze
Startup finished in 4.489s (kernel) + 16.468s (userspace) = 20.957s
graphical.target reached after 14.005s in userspace
```    
**systemd-analyze blame** - просмотреть подробную информацию о затраченном времени каждым блоком (процессом) в отдельности:  
```
root@os3:~# systemd-analyze blame
5.403s vboxadd.service
5.266s man-db.service
4.438s fwupd-refresh.service
4.171s snapd.service
4.171s logrotate.service
3.700s e2scrub_all.service
2.152s snap.lxd.activate.service
1.829s systemd-networkd-wait-online.service
1.803s cloud-config.service
1.671s dev-mapper-ubuntu\x2d\x2dvg\x2dubuntu\x2d\x2dlv.device
1.518s systemd-udev-settle.service
1.415s cloud-final.service
1.259s networkd-dispatcher.service
1.206s cloud-init-local.service
1.164s udisks2.service
...
```
**systemd-analyze critical-chain** - просмотреть подробную информацию в виде дерева критической по времени цепочки событий:   
```
root@os3:~# systemd-analyze critical-chain
The time when unit became active or started is printed after the "@" character.
The time the unit took to start is printed after the "+" character.

graphical.target @14.005s
└─multi-user.target @14.005s
  └─snapd.seeded.service @12.731s +512ms
    └─basic.target @7.437s
      └─sockets.target @7.431s
        └─snapd.socket @7.366s +36ms
          └─sysinit.target @7.213s
            └─cloud-init.service @6.127s +1.059s
              └─systemd-networkd-wait-online.service @4.291s +1.829s
                └─systemd-networkd.service @4.178s +109ms
                  └─network-pre.target @4.174s
                    └─cloud-init-local.service @2.966s +1.206s
                      └─systemd-remount-fs.service @715ms +106ms
                        └─systemd-journald.socket @582ms
                          └─system.slice @474ms
                            └─-.slice @474ms
```
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
> * UNIT — название юнита.  
> * LOAD — информация об успешной загрузке конфигурации.  
> * ACTIVE — сообщение о статусе. Может быть также INACTIVE, LOADED и другие.  
> * SUB — детальная системная информация об юните.  
> * DESCRIPTION — краткое описание юнита.  

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

> * Type — описывает, как запустится демон. Есть несколько вариантов:  
   **simple** (по умолчанию) — systemd ожидает, что служба запустится незамедлительно. Процесс не должен разветвляться.  
   **forking** — после запуска демон ответвляется (делает форк), родительский процесс завершается. Такой подход используется для запуска классических демонов.    
   **one-shot** — одноразовое выполнение. Используется для скриптов, которые запускаются и завершаются после выполнения.  
   **notify** — аналог simple, но в этом случае сам процесс сообщит systemd о том, что он закончил загрузку и готов к работе.
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

Пример оформления секции Service:

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

*  poweroff – отключение системы;
*  rescue – режим восстановления, однопользовательский (init 1);
*  multi-user – сетевой режим без графической оболочки, (init 3);
*  graphical – сетевой режим с графической оболочкой (init 5);
*  reboot – перезагрузка;
*  emergency – аварийная командная строка, минимальный функционал.
* default.target — режим, который будет загружаться по умолчанию, является символической ссылкой на один из boot tatget.  

Посмотреть target(цель) по умолчанию
> systemctl get-default  

или
> ls -l /usr/lib/systemd/system/default.target
```
/usr/lib/systemd/system/default.target -> multi-user.target
```

Так же все цели можно посмотреть:
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

* Перейти в нужную цель (например – загрузится из сетевого режима (multi-user.target) в графический)
> systemctl isolate graphical.target

* Выбрать target(цель) по умолчанию
>systemctl set-default multi-user.target

## **Редактирование юнитов**

Юниты нельзя редактировать напрямую. Для этого используется команда edit:

>systemctl edit --full nginx.service

Теперь можно добавить в описание юнита несколько переменных. Например, измените секцию Service. Чтобы сервис автоматически перезапускался в случае сбоя через 60 секунд.  
*RestartSec=* - позволяет указать время простоя в секундах, после которого systemd перезапускает службу после аварии (по умолчанию 5 секунд).  
*Restart=* -  определяет, при каких условиях необходимо перезапускать службы. Значение по умолчанию *- -on-failure*. Это означает, что службу необходимо перезапустить, если она завершила работу с ненулевым кодом выхода.


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
все команды можно посмотреть в man systemctl
* systemctl или systemctl list-units - список запущенных юнитов
* systemctl list-units --type service --all – отображение статуса всех сервисов 
* systemctl list-unit-files --type service – отображает все сервисы и проверяет, какие из них активированы  
* systemctl start name.service – запуск сервиса.  
* systemctl stop name.service — остановка сервиса  
* systemctl restart name.service — перезапуск сервиса  
* systemctl try-restart name.service — перезапуск сервиса только, если он запущен  
* systemctl reload name.service — перезагрузка конфигурации сервиса  
* systemctl status name.service — проверка, запущен ли сервис с детальным выводом состояния сервиса  
* systemctl status pid - информация о процессе по его PID
> systemctl status 766 - PID процесса
* systemctl is–enabled name.service – проверяет, активирован ли сервис 
* systemctl is-active name.service — проверка, запущен ли сервис с простым ответом: active или inactive  
* systemctl enable name.service – активирует сервис (позволяет стартовать во время запуска системы)  
* systemctl enable --now name.service - включить юнит в автозагрузку и сразу запустить
* systemctl disable name.service – деактивирует сервис  
* systemctl reenable name.service – деактивирует сервис и сразу активирует его  
* systemctl mask name.service – заменяет файл сервиса симлинком на /dev/null, делая юнит недоступным для systemd  
> /dev/null — специальный файл в системах класса UNIX, представляющий собой так называемое «пустое устройство». Запись в него происходит успешно, независимо от объёма «записанной» информации.
* systemctl unmask name.service – возвращает файл сервиса, делая юнит доступным для systemd  
* systemctl cat name.service - отобразить содержимое юнита
* systemctl edit --full name.service - редактирование файл сервиса
* systemctl show name.service - показывает все параметры systemd, которые можно настроить в юните
* systemctl help unit - страница руководства юнита
* systemctl daemon-reload - Перезагрузить настройки systemd. Сканировать систему на наличие новых или изменённых юнитов

## По подробней разберём *systemctl status name.service*

```
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled) 
     Active: active (running) since Sat 2022-11-26 15:42:17 UTC; 8min ago
       Docs: man:nginx(8)
    Process: 667 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 742 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 766 (nginx)
      Tasks: 3 (limit: 2274)
     Memory: 7.2M
     CGroup: /system.slice/nginx.service
             ├─766 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             ├─768 nginx: worker process
             └─769 nginx: worker process

Nov 26 15:42:16 os3 systemd[1]: Starting A high performance web server and a reverse proxy server...
Nov 26 15:42:17 os3 systemd[1]: Started A high performance web server and a reverse proxy server.
```

1.	Название и описание службы
2.	**"Loaded** :" в выходных данных покажет "loaded", если устройство было загружено в память.   
Другие возможные значения для "Loaded :" включает:   
•	**"error"**, если возникла проблема с загрузкой  
•	**"not-found"**, если для этого модуля не был найден файл модуля  
•	**"bad-setting"**, если не удалось проанализировать настройку файла модуля  
•	**"masked"**, если файл модуля был замаскирован.   
•	**"enabled/disabled"** - так же тут отображается путь к файлю модуля\сервиса и включён или нет сервис в автозагрузку
3.	**Active** active(running) – сервис активен и запущен.   
Другие возможные состояния:    
•	**active (exited)**	- служба успешно запущена из файла конфигурации. Обычно однократно считывается конфигурация служб перед выходом из службы. Например, служба AppArmor или Firewall.  
•	**active (waiting)** - наша служба запущена но ожидает событие такое как CPUS/printing event  
•	**inactive** - либо одноразовая конфигурация не выполнилась, либо еще не выполнена. (Сервис может быть в статусе inactive(dead), но это не значит что он не работает, возможно он запускается по таймеру.  
•	**"6 days ago"** - Так же здесь можно увидеть с какого времени служба запущена
4.	**Docs** - файлы документации,в нашем случае man- страницы, также может быть указан url на документацию.
5.	**Main PID** – уникальный номер процесса
6.	**Tasks** – количество заданий и их лимит
7.	**Memory** – информация об используемой памяти(не всегда можно положиться на эту информацию)
8.	**CGroup** - Контрольная группа (cgroup), к которой принадлежит процесс данного подразделения. Группа - это подмножество процессов, ресурсы которых могут быть ограничены и обрабатываться совместно. В этом случае sshd является единственным процессом в этой группе, но в случае, когда он совместно используется другими процессами, их pid также будут перечислены здесь.
> Контрольная группа (англ. control group, cgroups, cgroup) — группа процессов в Linux, для которой механизмами ядра наложена изоляция и установлены ограничения на некоторые вычислительные ресурсы (процессорные, сетевые, ресурсы памяти, ресурсы ввода-вывода). Механизм позволяет образовывать иерархические группы процессов с заданными ресурсными свойствами и обеспечивает программное управление ими.  
**Механизм предоставляет следующие возможности:**  
> * ограничение ресурсов (англ. resource limiting): использование памяти, в том числе виртуальной;  
> * приоритизацию: разным группам можно выделить разное количество процессорного ресурса и пропускной способности подсистемы ввода-вывода;  
> * учёт: подсчёт затрат тех либо иных ресурсов группой;  
> * изоляцию: разделение пространств имён для групп таким образом, что одной группе недоступны процессы, сетевые соединения и файлы другой;  
> * управление: приостановку (freezing) групп, создание контрольных точек (checkpointing) и их перезагрузку.  
![systemctl status](/cgroups.png)  
9. последние строки логов
#


# Journald #  
**systemd** использует журнал (journal), собственную систему ведения логов, в связи с чем больше не требуется запускать отдельный демон логирования. Для чтения логов используется команда **journalctl**  

**Journald** — системный демон журналов systemd. Systemd спроектирован так, чтобы централизованно управлять системными логами от процессов, приложений и т.д. Все такие события обрабатываются демоном journald, он собирает логи со всей системы и сохраняет их в бинарных файлах.

Преимуществ централизованного логирования событий в бинарном формате достаточно много, например системные логи можно транслировать в различные форматы, такие как простой текст, или в JSON, при необходимости. Так же довольно просто можно отследить лог вплоть до одиночного события используя фильтры даты и времени.

Файлы логов journald могут собирать тысячи событий и они обновляются с каждым новым событием, поэтому если ваша Linux-система работает достаточно долго — размер файлов с логами может достигать несколько гигабайт и более. Поэтому анализ таких логов может происходить с задержками, в таком случае, при анализе логов можно фильтровать вывод, чтобы ускорить работу.

> **Journalctl**  — отличный инструмент для анализа логов, обычно один из первых с которым знакомятся начинающие администраторы linux систем. Встроенные возможности ротации, богатые возможности фильтрации и возможность просматривать логи всех systemd unit-сервисов одним инструментом очень удобны и заметно облегчают работу системным администраторам.

### Файл конфигурации journald

Файл конфигурации можно найти по следующему пути: **/etc/systemd/journald.conf**, он содержит различные настройки journald.

Каталог с журналом journald располагается **/run/log/journal** (в том случае, если не настроено постоянное хранение журналов, но об этом позже).

Файлы хранятся в бинарном формате, поэтому нормально их просмотреть с помощью cat или nano, как привыкли многие администраторы — не получится.

### Использование journalctl для просмотра и анализа логов

Основная команда для просмотра:
> journalctl    
> journalctl --utc посмотреть логи в UTC

*Она выведет все записи из всех журналов, включая ошибки и предупреждения, начиная с того момента, когда система начала загружаться. Старые записи событий будут наверху, более новые — внизу, вы можете использовать PageUp и PageDown чтобы перемещаться по списку, Enter — чтобы пролистывать журнал построчно, G -чтобы переместится в конец журнала, / - для инициализации поиска и Q — чтобы выйти.*

### Фильтрация событий по важности

Система записывает события с различными уровнями важности, какие-то события могут быть предупреждением, которое можно проигнорировать, какие-то могут быть критическими ошибками. Если мы хотим просмотреть только ошибки, игнорируя другие сообщения, введем команду с указанием кода важности:

> journalctl -p 0

Для уровней важности, приняты следующие обозначения:

0: emergency (неработоспособность системы)  
1: alerts (предупреждения, требующие немедленного вмешательства)  
2: critical (критическое состояние)  
3: errors (ошибки)  
4: warning (предупреждения)  
5: notice (уведомления)  
6: info (информационные сообщения)  
7: debug (отладочные сообщения)  

Когда вы указываете код важности, journalctl выведет все сообщения с этим кодом и выше. Например если мы укажем опцию -p 2, journalctl покажет все сообщения с уровнями 2, 1 и 0.

### Настройка хранения журналов

По умолчанию journald перезаписывает свои журналы логов при каждой перезагрузке, и вызов journalctl выведет журнал логов начиная с текущей загрузки системы.

Если необходимо настроить постоянное сохранение логов, потребуется отдельно это настроить, т.к. разработчики отказались от постоянного хранения всех журналов, чтобы не дублировать rsyslog.
> Rsyslog — это мощный сервис для управления логами в Linux.

Файл конфигурации находится по пути /etc/systemd/journald.conf и выглядит следующем образом:
```
Journald Configuration File
# See journald.conf(5) for details.
[Journal]
#Storage=auto
#Compress=yes
#Seal=yes
#SplitMode=uid
#SyncIntervalSec=5m
#RateLimitInterval=30s
#RateLimitBurst=1000
#SystemMaxUse=
#SystemKeepFree=
#SystemMaxFileSize=
#SystemMaxFiles=100
#RuntimeMaxUse=
#RuntimeKeepFree=
#RuntimeMaxFileSize=
#RuntimeMaxFiles=100
#MaxRetentionSec=
#MaxFileSec=1month
#ForwardToSyslog=yes
#ForwardToKMsg=no
#ForwardToConsole=no
#ForwardToWall=yes
#TTYPath=/dev/console
#MaxLevelStore=debug
#MaxLevelSyslog=debug
#MaxLevelKMsg=notice
#MaxLevelConsole=info
#MaxLevelWall=emerg
```

Для постоянного хранения логов необходимо раскоментировать занчение Storage= и установить для него значение persistent (постоянный)
```
[Journal]
Storage=persistent
```

Это создаст каталог /var/log/journal, и все файлы журнала будут сохранены в эту директорию.


Полное описание:
> **Storage=** Указывает, где хранить журнал. Доступны следующие параметры:
> * **volatile** Журнал хранится в оперативной памяти, т.е. в каталоге /run/log/journal.    
> * **persistent** Данные хранятся на диске, т.е. в каталоге /var/log/journal  
> * **auto** используется по-умолчанию  
> * **none** Журнал не ведётся 

> **Compress=** Принимает значения "yes" или "no". Если включена (по-умолчанию) сообщения перед записью в журнал, будут сжиматься.  

> **Seal=** Принимает значения "yes" или "no". Если включена (по-умолчанию) будет включена защита Forward Secure Sealing (FSS), которая позволяет накладывать криптографические отпечатки на журнал системных логов.

> **SplitMode=** Определяет доступ к журналу пользователям. Доступны следующие параметры:
> * **uid** Все пользователи получают доступ к чтению системного журнала. Используется по-умочанию.  
> * **login** Каждый пользователь может читать только сообщения, относящиеся к его сеансу.  
> * **none** Пользователи не имеют доступа к системному журналу.

> **SyncIntervalSec=** Таймаут, после которого происходит синхронизация и запись журнала на диск. Относится только к уровням ERR, WARNING, NOTICE, INFO, DEBUG. Сообщения уровня CRIT, ALERT, EMERG записываются сразу на диск.

> **RateLimitInterval= и RateLimitBurst=** Настройки ограничения скорости генерации сообщений для каждой службы. Если в интервале времени, определяемого RateLimitInterval=, больше сообщений, чем указано в RateLimitBurst= регистрируются службой, все дальнейшие сообщения в интервале отбрасываются, пока интервал не закончится.При этом генерируется сообщение о количестве отброшенных сообщений. По умолчанию 1000 сообщений за 30 секунд. Единицы измерения: "s", "min", "h", "ms", "us". Чтобы выключить ограничение скорости, установите значения в 0.

> **MaxRetentionSec=** Максимальное время хранения записей журнала. Единицы измерения: year, month, week, day, h или m

> **MaxFileSec=** Максимальное время хранения записей в одном файле журнала, после которого он переводится в следующий.

> **ForwardToSyslog=, ForwardToKMsg=, ForwardToConsole=, ForwardToWall=** Определяют куда направлять сообщения: в традиционный системный журнал Syslog, в буфер журнала ядра (kmsg), на системную консоль, или на стену, чтобы было видно всем зарегистрированным пользователям. Эти опции принимают логические аргументы. Если переадресация на Syslog включен, но Syslog демон не работает, соответствующий параметр не имеет никакого эффекта. По умолчанию, только стена включена. Эти параметры могут быть переопределены во время загрузки с параметрами командной строки ядра systemd.journald.forward_to_syslog =, systemd.journald.forward_to_kmsg =, systemd.journald.forward_to_console = и systemd.journald.forward_to_wall =. При пересылке в консоль, должен быть установлен TTYPath =, как будет описано ниже.

> **TTYPath=** Назначает консоль TTY, для вывода сообщений, если установлен параметр ForwardToConsole=yes. По-умолчанию, используется /dev/console. Для того, чтобы вывести на 12 консоль, устанавливаем TTYPath=/dev/tty12. Для того, чтобы вывести на последовательный порт, устанавливаем TTYPath=/dev/ttySX, где X номер com-порта.

> **MaxLevelStore=, MaxLevelSyslog=, MaxLevelKMsg=, MaxLevelConsole=, MaxLevelWall=** Определяет максимальный уровень сообщений который сохраняется в журнал, выводится на традиционный системный журнал Syslog, буфер журнала ядра (kmsg), консоль или стену. Значения: emerg, alert, crit, err, warning, notice, info, debug или цифры от 0 до 7 (соответствуют уровням).

## Управление логированием

Определение текущего объёма логов

Со временем объём логов растёт, и они занимают всё больше места на жёстком диске. Узнать объём имеющихся на текущий момент логов можно с помощью команды:
```
journalctl --disk-usage
```
### Ротация логов

Настройка ротации логов осуществляется с помощью опций **−−vacuum-size** и **−−vacuum-time**. Первая из них устанавливает предельно допустимый размер для хранимых на диске логов (в нашем примере — 1 ГБ):
```
journalctl --vacuum-size=1G
```
>Как только объём логов превысит указанную цифру, лишние файлы будут автоматические удалены. Аналогичным образом работает опция −−vacuum-time. Она устанавливает для логов срок хранения, по истечении которого они будут автоматически удалены:
```
journalctl --vacuum-time=1years
```

>Настройка ротации в конфигурационном файле. Настройки ротации логов можно также прописать в конфигурационном файле **/еtc/systemd/journald.conf**, который включает в числе прочих следующие параметры:  
* **SystemMaxUse=** — максимальный объём, который логи могут занимать на диске;  
* **SystemKeepFree=** — объём свободного места, которое должно оставаться на диске после сохранения логов;  
* **SystemMaxFileSize=** — объём файла лога, по достижении которого он должен быть удален с диска;
* **RuntimeMaxUse=** — максимальный объём, который логи могут занимать в файловой системе /run;
* **RuntimeKeepFree=** — объём свободного места, которое должно оставаться в файловой системе /run после сохранения логов;  
* **RuntimeMaxFileSize=** — объём файла лога, по достижении которого он должен быть удален из файловой системы /run.  
> **Единицы измерения: K, M, G, T, P, E.**