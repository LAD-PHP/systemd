<<<<<<< HEAD
# systemd

Systemd — это менеджер системы и служб Linux. Он инициализирует другие демоны в системе при загрузке. Основная цель systemd — полный контроль над запуском и выполнением процессов, описанных в его конфигурации.
=======
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
