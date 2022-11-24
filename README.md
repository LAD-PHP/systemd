# Systemd
___
**Systemd** — это менеджер системы и служб Linux. Он инициализирует другие демоны в системе при загрузке. Основная цель systemd — полный контроль над запуском и выполнением процессов, описанных в его конфигурации.

>При старте Systemd запускает параллельно столько служб, сколько возможно в зависимости от стадии загрузки. Это заметно ускоряет запуск системы. Если замерить время от включения до входа, то systemd использует его гораздо меньше, чем предшественники — например, SystemV.

Systemd — универсальный менеджер. Он управляет практически всей работающей системой. Systemd отвечает за работу запущенных служб и может рассказать все об их состоянии. Он также управляет монтированием файловой системы, аппаратной частью, процессами.

Systemd использует скомпилированные бинарные файлы. Все конфигурационные файлы открыты. Их можно менять через командную строку или GUI. При необходимости можно добавить свои конфигурационные файлы.

Архитектура systemd
На этой схеме изображены компоненты, которые входят в systemd. Это упрощенное представление, которое показывает только основную архитектуру. В нем размещены не все файлы и службы. Здесь также нет информации о сложном потоке данных.

![alt text](https://highload.today/wp-content/uploads/2021/12/image1-5.png)
 

Возможности systemd
В зависимости от используемых при компиляции параметров systemd включает до 69 файлов. С их помощью выполняются следующие задачи:

Systemd как PID 1 осуществляет запуск стольких служб в параллельном режиме, сколько ему нужно. Благодаря этому загрузка ускоряется.

> В каждой Unix системе есть один процесс с специальным идентификатором 1. Он запускается ядром перед всеми последующими процессами, и он является также родительским процессом для всех последующих/остальных процессов кто не смог найти себе родителя. Вследствие, он может делать много того чего не могут позволить себе другие процессы. Также он несет ответственность за некоторые вещи за которые не отвечают другие процессы, например, за поднятие и поддержку юзерспейса во время загрузки системы.

Systemd создает журналы для хранения системных логов и дает инструменты для управления записями.
> Systemd предоставляет решение централизованного управления для регистрации всех процессов ядра и пользовательской области. Система, собирающая эти журналы и управляющая ими, называется журнальной системой.

Systemctl дает пользовательский интерфейс для управления службами.

Обеспечивается обратная совместимость благодаря поддержке SystemV и LSB.

Управление службами и логи дают информацию о состоянии служб.

Можно управлять сокетами.
> Сокеты — это прямая связь между двумя процессами.В некотором смысле сокеты — это сеть, полностью содержащаяся в ядре; вместо того, чтобы использовать сетевые интерфейсы и соответствующие накладные расходы для отправки данных, те же самые данные могут быть отправлены напрямую между программами.Сокеты просто предоставляют фактическое оборудование для перемещения данных.

Таймеры предоставляют расширенные возможности для планирования, включая запуск скриптов по времени от старта системы.
> Таймеры, как и задания cron, могут, в заданное время, вызывать выполнение различных действий в системе. Например — запуск скриптов командной оболочки или программ. Таймеры могут срабатывать, например, раз в день, причём — только по понедельникам. Ещё один пример — срабатывание таймера каждые 15 минут в рабочее время (с 8 утра до 6 вечера). Но таймеры systemd могут кое-что такое, что недоступно заданиям cron. Например, таймер может вызвать скрипт или программу через заданное время после некоего события. Таким событием может быть загрузка системы или запуск systemd, завершение предыдущей задачи или даже завершение работы сервиса, вызванного ранее по таймеру.

Можно монтировать и размонтировать файловые системы с иерархическим уведомлением для безопасного каскадирования.
> systemd.mount — монтирование
systemd полностью отвечает за монтирование разделов и файловых систем, описанных в файле /etc/fstab. systemd-fstab-generator(8) преобразует записи из /etc/fstab в юниты systemd; это выполняется при каждой загрузке системы, а также при перезагрузке конфигурации системного менеджера.
systemd расширяет возможности fstab и предлагает дополнительные опции монтирования. Они могут влиять на зависимости юнита монтирования: например, могут гарантировать, что монтирование выполняется только после подключения к сети или после монтирования другого раздела. 
Примером этих опций может быть автомонтирование (здесь имеется в виду не автоматическое монтирование во время загрузки, а монтирование при появлении запроса от устройства). 
ПРИВЕРЫ:
Если есть большой раздел, вы можете разрешить службам, которые не обращаются к нему, запускаться в то время, как он проверяется программой fsck. Для этого добавьте следующие параметры монтирования в запись /etc/fstab для соответствующего раздела:

> noauto,x-systemd.automount
При этом процедура проверки и монтирования раздела будет запущена только при первой попытке доступа, и ядро будет держать в ожидании все потоки ввода-вывода, связанные с этим разделом, пока он не будет смонтирован. Это полезно в случае, например, очень большого раздела /home.

> Примечание: Разделу будет присвоен тип файловой системы autofs, который по умолчанию игнорируется mlocate. Используйте эту возможность с осторожностью.
Удалённая файловая система
Автоматическое монтирование может аналогичным образом использоваться и для монтирования удаленных файловых систем. В дополнение, вы можете использовать параметр x-systemd.device-timeout=# для указания времени ожидания удаленной файловой системы при перебоях в соединении. Также опция _netdev даёт systemd понять, что файловая система зависит от сети.
noauto,x-systemd.automount,x-systemd.mount-timeout=30,_netdev
> Автоматическое размонтирование
> С помощью флага x-systemd.idle-timeout можно указать таймаут бездействия. Например:

> noauto,x-systemd.automount,x-systemd.idle-timeout=1min
> В таком случае systemd размонтирует раздел спустя одну минуту неактивности.

Можно создавать временные файлы и управлять ими, в том числе удалять.

Можно запускать скрипты при подключении или отключении устройств.

Можно через анализ последовательности загрузки найти службы, запуск которых отнимает больше всего ресурсов или вызывает сбои.
Эти и другие задачи решаются различными демонами, которые управляют программами и конфигурационными файлами в составе пакета systemd.

**Юниты systemd**

ЮНИТ — ЭТО ОПИСАНИЕ СЕРВИСА В ТЕКСТОВОМ ВИДЕ. В НЕМ УКАЗАНЫ ОПЕРАЦИИ, КОТОРЫЕ ВЫПОЛНЯЮТСЯ ДО И ПОСЛЕ ЗАПУСКА СЛУЖБЫ. ПО СУТИ, ЭТО ОПИСАНИЕ ПАРАМЕТРОВ ИНИЦИАЛИЗАЦИИ.
Все юниты разложены по трем каталогам:
1.	/usr/lib/systemd/system/ — юниты из установленных пакетов RPM. Например, Nginx, MySQL, Apache.
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

Юниты бывают разных типов. Например, юнит службы имеет тип *.service. Все виды:
> .service	Описывает, как управлять службой или приложением на сервере.

> .socket	Описывает сетевой, IPC-сокет или FIFO-буфер, который используется для активации сокета.

> .device	Описывает устройство, указанное как необходимое для управления systemd с помощью udev или файловой системы sysfs.

> .mount	Определяет точку монтирования в системе, которой управляет systemd.

> .automount	Настраивает точку монтирования, которая будет автоматически установлена.

> .swap	Описывает пространство подкачки в системе (swap-файл).

> .target	Обеспечивает точки синхронизации для других устройств при загрузке или изменении состояний.

> .path	Определяет путь, который может использоваться для активации на основе пути.

> .timer	Определяет таймер, который управляется systemd для задержки или активации по плану.

> .snapshot	Позволяет восстановить текущее состояние системы после изменений.

> .slice	Связан с узлами группы управления Linux, что позволяет ограничить ресурсы или назначить их для любых процессов, связанных с slice.

> .scope	Создается автоматически systemd из информации, которую получил от интерфейса шины.

Поработаем с сервисами *.service

Чтобы отфильтровать их необходимо выполнить команду > systemctl list-units --type=service
Также можно выбрать только неактивные службы
> systemctl list-units --all --state=inactive
Можно использовать другие статусы:
> active, inactive, running, exited, dead, loaded, not-found, plugged, mounted, waiting, listening.
