**TITLE:**  
Netdata: система моніторингу VPS та Linux-серверів — Хостинг-компанія FREEhost.UA

**Description:**  
Використання інструмента на Ubuntu 24.04 і Debian 13: встановлення, перший запуск, моніторинг CPU, RAM, I/O pressure, дисків, мережі, PHP-FPM, MySQL/MariaDB та сповіщення для VPS.

# **Netdata: швидкий моніторинг VPS та Linux-серверів за 5 хвилин**

Коли сайт працює на VPS, адміністратору потрібно бачити не тільки факт доступності сервера. Важливо швидко зрозуміти, що саме сповільнює сервер: CPU, RAM, диск, мережа, PHP-FPM чи база даних. Команди `top`, `free`, `df` і `ss` допомагають, але для нормальної діагностики потрібні графіки, історія метрик і сповіщення.

Інструмент добре підходить для такого старту. Це система моніторингу сервера, яку можна встановити за кілька хвилин і одразу отримати моніторинг VPS у веб-панелі. Рішення зручне для одного Linux-сервера, тестового стенда або невеликого production-проєкту. Якщо ви тільки запускаєте сайт, можна почати з сервера на [VPS-хостингу Freehost](https://freehost.com.ua/ukr/vps-hosting/) і відразу налаштувати базовий контроль стану сервера.

## **Навіщо моніторити сервер**

Моніторинг Linux-сервера потрібен для практичних завдань:

* чи справді процесор є вузьким місцем;
* чи не йде система у swap після релізу;
* чи не ростуть затримки дискових операцій;
* чи вистачає PHP-FPM workers під піковим трафіком;
* чи не забивається дисковий простір логами або бекапами;
* чи є проблеми з cron, backup або деплоєм.

Без історії метрик адміністратор бачить тільки поточний стан. Саме тому моніторинг продуктивності сервера має показувати не лише відсоток CPU, а й load average, iowait, steal time, swap, I/O pressure, мережевий трафік і стан сервісів. Якщо одночасно ростуть load average, iowait і latency диска, варто перевіряти диск або базу. Якщо CPU зайнятий, але iowait низький, шукайте процес, чергу PHP-FPM або важкі запити застосунку. Такий підхід робить моніторинг сервера корисним не тільки під час аварій, а й для планового контролю навантаження сервера.

## **Що таке Netdata**

Це інструмент і веб-інтерфейс моніторингу, який виконує збір метрик і показує серверні метрики майже в реальному часі. Після інсталяції одразу доступні системні показники: CPU, RAM, swap, диски, файлові системи, мережа, сокети, процеси, systemd-сервіси та Linux PSI.

PSI, або Pressure Stall Information, особливо корисний для VPS: він показує, скільки часу задачі втрачають через конкуренцію за CPU, пам'ять або I/O. Для віртуального сервера це часто точніший сигнал, ніж один графік завантаження.

Також є колектори для Nginx, Apache, PHP-FPM, MySQL, MariaDB, PostgreSQL, Redis і Docker. Частина графіків з'являється автоматично, а для прикладних сервісів зазвичай треба додати job у YAML-конфігурацію.

## **Встановлення на Ubuntu 24.04 / Debian 13**

Для встановлення Netdata на Ubuntu 24.04 або Netdata Debian 13 найпростіше використати офіційний `kickstart.sh`. Такий сценарій Netdata Ubuntu 24.04 підходить і для нового VPS, і для тестового стенда. Скрипт визначає дистрибутив і на підтримуваних системах ставить native packages. Для production-сервера краще явно обрати stable-канал.

<pre>
wget -O /tmp/netdata-kickstart.sh https://get.netdata.cloud/kickstart.sh
sudo sh /tmp/netdata-kickstart.sh --release-channel stable --non-interactive
</pre>

Перед запуском install-скрипт можна переглянути:

<pre>
less /tmp/netdata-kickstart.sh
</pre>

Після інсталяції перевірте сервіс і порт:

<pre>
systemctl status netdata
netdata -v
ss -lntp | grep 19999
</pre>

На публічному VPS не варто відкривати порт `19999` для всього інтернету. Для першого запуску безпечніше використати SSH-тунель:

<pre>
ssh -L 19999:127.0.0.1:19999 user@server
</pre>

Після цього відкрийте локально:

<pre>
http://localhost:19999
</pre>

Для невеликого сервера зазвичай достатньо stable-каналу й стандартних автооновлень. Якщо оновлення контролюються вручну, під час інсталяції можна додати `--no-updates`.

### **Пакети для тестового навантаження**

Щоб одразу побачити графіки продуктивності, встановіть базові сервіси й утиліти:

<pre>
sudo apt update
sudo apt install -y nginx php-fpm php-cli php-mysql mariadb-server
sudo apt install -y stress-ng fio sysbench apache2-utils iperf3
</pre>

Цього вистачить, щоб перевірити моніторинг ресурсів сервера, CPU, I/O pressure, мережу, моніторинг PHP-FPM і метрики MySQL/MariaDB.

## **Перший запуск**

Після входу в панель видно вузол, часовий діапазон, live-оновлення, алерти та розділи System, Network, Storage, Applications і Processes. Для початківців це зручно: основні метрики не треба збирати вручну з різних утиліт. Для адміністраторів цінність у деталях: можна швидко зіставити кілька графіків за один проміжок часу й оцінити навантаження на систему.

Перший тест:

<pre>
stress-ng --cpu 2 --vm 1 --vm-bytes 256M --timeout 90s --metrics-brief
</pre>

Через кілька секунд у панелі буде видно стрибок CPU, load average, активність RAM і pressure-метрики. Так працює моніторинг у реальному часі: навантаження створюється зараз, а графік реагує майже одразу.

**[МІСЦЕ ДЛЯ СКРІНШОТА 1: Overview або System -> CPU/RAM під час `stress-ng`]**

*Підпис до зображення:* Навантаження CPU та RAM під час `stress-ng`: видно стрибок використання процесора, load average, активність пам'яті та CPU pressure у реальному часі.

## **Огляд інтерфейсу**

Панель керування моніторингом побудована навколо інтерактивних графіків. У кожному блоці можна змінити часовий діапазон, зупинити live-оновлення, подивитися окремі dimensions і збільшити потрібний проміжок.

Після встановлення в першу чергу варто перевірити:

* CPU usage, load average, interrupts, softirqs;
* memory, swap, available RAM, page faults;
* disks, IOPS, bandwidth, utilization, latency;
* filesystem usage та inode usage;
* network traffic, drops, errors, TCP sockets;
* PSI для CPU, memory та I/O;
* alerts, які вже активні.

Такий веб-інтерфейс моніторингу не замінює логи, але скорочує шлях до пошуку проблеми: діагностика несправностей починається з графіків, а не з припущень. Якщо в логах Nginx видно повільні відповіді, у панелі можна одразу перевірити, що в цей момент було з PHP-FPM, диском, базою даних і RAM.

## **Моніторинг CPU, RAM, дисків та мережі**

Для CPU дивіться не тільки на "100%". Важливі user/system/steal/iowait, load average, runnable processes і CPU pressure. Для моніторингу ресурсів сервера це базові метрики, без яких контроль навантаження сервера буде неточним. На VPS окремо корисний steal time: якщо він росте, частину процесорного часу забирає гіпервізор або сусіднє навантаження на фізичному вузлі, а загальне навантаження на систему виглядає нестабільним.

Для RAM головні сигнали - available memory, завантаження оперативної пам'яті, swap activity, major page faults і memory pressure. Linux активно використовує кеш, тому "used memory" сам по собі не є проблемою. Проблема починається тоді, коли процеси чекають пам'ять або система активно працює зі swap.

Для дисків запустіть короткий `fio`:

<pre>
fio --name=netdata-disk-demo --filename=/tmp/netdata-fio-demo \
  --size=512M --rw=randrw --bs=4k --runtime=90 \
  --time_based --group_reporting
rm -f /tmp/netdata-fio-demo
</pre>

У Storage перевірте IOPS, read/write bandwidth, utilization, latency та I/O pressure. Якщо диск перевантажений, CPU може просто чекати завершення операцій, а продуктивність веб-сайту падатиме без очевидного процесорного піку.

**[МІСЦЕ ДЛЯ СКРІНШОТА 2: System -> Pressure або Storage -> Disks під час `fio`]**

*Підпис до зображення:* I/O pressure і дискові метрики під час `fio`: видно зростання затримок дискових операцій, IOPS та активності читання/запису. Такий екран допомагає оцінити, чи накопичувач не став вузьким місцем VPS.

Для мережі дивіться inbound/outbound traffic, packets, drops, errors, TCP sockets і retransmits. Якщо одночасно ростуть HTTP-запити, активні PHP-FPM connections і мережевий трафік, навантаження йде від реальних користувачів або ботів. Якщо мережа тиха, а CPU і диск зайняті, перевіряйте cron, backup, черги або фоновий імпорт.

## **Контроль навантаження PHP-FPM**

Моніторинг PHP-FPM потребує status page. У pool-конфігурації, зазвичай `/etc/php/*/fpm/pool.d/www.conf`, увімкніть:

<pre>
pm.status_path = /status
</pre>

Перезапустіть службу:

<pre>
sudo systemctl restart php*-fpm
</pre>

Endpoint краще віддати через Nginx тільки для localhost. Socket змініть під свою версію PHP:

<pre>
location ~ ^/(status|ping)$ {
    allow 127.0.0.1;
    deny all;
    fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
</pre>

Перевірка:

<pre>
curl http://127.0.0.1/status
</pre>

Далі відкрийте конфігурацію collector:

<pre>
cd /etc/netdata 2>/dev/null || cd /opt/netdata/etc/netdata
sudo ./edit-config go.d/phpfpm.conf
</pre>

Мінімальний job:

<pre>
jobs:
  - name: local
    url: http://localhost/status?full&json
</pre>

Перезапустіть агент і створіть HTTP-навантаження:

<pre>
sudo systemctl restart netdata
ab -n 5000 -c 40 http://127.0.0.1/
</pre>

У Applications -> PHP-FPM мають з'явитися active/idle connections, requests per second, slow requests, max children reached, request duration, request CPU і request memory. Для WordPress, Laravel або Symfony це швидкий спосіб зрозуміти, чи вистачає `pm.max_children`, чи потрібне кешування, чи проблема нижче - у SQL-запитах або диску.

**[МІСЦЕ ДЛЯ СКРІНШОТА 3: Applications -> PHP-FPM під час `ab`]**

*Підпис до зображення:* Графіки PHP-FPM під тестовим HTTP-навантаженням: видно активні підключення, requests per second, request duration і можливі спрацювання `max children reached`.

## **Моніторинг MySQL/MariaDB**

Моніторинг MySQL і моніторинг MariaDB працює через collector `mysql`. Він підключається через TCP або Unix socket і читає global status, InnoDB status, змінні сервера, replication status, connections, queries, handlers та locks.

Для production створіть окремого користувача. Для MySQL і MariaDB до 10.5.9:

<pre>
CREATE USER 'netdata'@'localhost';
GRANT USAGE, REPLICATION CLIENT, PROCESS ON *.* TO 'netdata'@'localhost';
FLUSH PRIVILEGES;
</pre>

Для MariaDB 10.5.9+ використовуйте `SLAVE MONITOR`:

<pre>
CREATE USER 'netdata'@'localhost';
GRANT USAGE, SLAVE MONITOR, PROCESS ON *.* TO 'netdata'@'localhost';
FLUSH PRIVILEGES;
</pre>

Конфігурація:

<pre>
cd /etc/netdata 2>/dev/null || cd /opt/netdata/etc/netdata
sudo ./edit-config go.d/mysql.conf
</pre>

TCP-приклад:

<pre>
jobs:
  - name: local
    dsn: netdata@tcp(127.0.0.1:3306)/
</pre>

Unix socket:

<pre>
jobs:
  - name: local
    dsn: netdata@unix(/run/mysqld/mysqld.sock)/
</pre>

Для тесту використайте `sysbench`:

<pre>
sudo mariadb -e "CREATE DATABASE IF NOT EXISTS sbtest;"
sudo sysbench oltp_read_write --db-driver=mysql \
  --mysql-socket=/run/mysqld/mysqld.sock --mysql-user=root \
  --mysql-db=sbtest --tables=2 --table-size=20000 prepare
sudo sysbench oltp_read_write --db-driver=mysql \
  --mysql-socket=/run/mysqld/mysqld.sock --mysql-user=root \
  --mysql-db=sbtest --tables=2 --table-size=20000 \
  --time=90 --threads=4 run
sudo sysbench oltp_read_write --db-driver=mysql \
  --mysql-socket=/run/mysqld/mysqld.sock --mysql-user=root \
  --mysql-db=sbtest --tables=2 cleanup
</pre>

Під час тесту дивіться не тільки queries per second. Корисніші зв'язки: InnoDB reads/writes, buffer pool, locks, temporary tables, threads connected/running і одночасна активність диска. Якщо база впирається в I/O, це буде видно і в графіках Storage.

**[МІСЦЕ ДЛЯ СКРІНШОТА 4: Applications -> MySQL/MariaDB під час `sysbench`]**

*Підпис до зображення:* Метрики MariaDB під час `sysbench`: видно активні підключення, SQL-запити, InnoDB-активність і зв'язок із дисковим навантаженням.

## **Сповіщення про проблеми**

Інструмент має вбудований health monitoring. Типові правила уже налаштовані з коробки, а власні alert definitions краще зберігати в `/etc/netdata/health.d`, щоб оновлення пакета не перезаписували зміни.

На старті перевірте алерти для таких ситуацій:

* заповнення диска та inode usage;
* RAM, swap і memory pressure;
* CPU load, iowait і steal time;
* disk latency та I/O utilization;
* недоступність сервісів;
* помилки collector jobs.

Автоматичні алерти не замінюють аналіз, але прискорюють виявлення перевантажень. Якщо система моніторингу сервера повідомляє про диск, пам'ять або latency, адмін реагує до того, як проблема стане інцидентом для користувачів. Для одного сервера достатньо залишити стандартні правила, налаштувати критичний канал повідомлень і протягом тижня подивитися, які warnings повторюються. Це простий спосіб підтримувати контроль навантаження сервера без постійного ручного перегляду графіків.

## **Переваги та недоліки**

### **Переваги**

* встановлення Netdata займає кілька хвилин;
* базовий моніторинг VPS з'являється без довгої конфігурації;
* є моніторинг у реальному часі з високою деталізацією;
* CPU, RAM, диски, мережа й PSI доступні одразу після запуску;
* можна додати PHP-FPM, MySQL, MariaDB та інші колектори;
* локальний доступ через SSH-тунель не вимагає відкривати порт.

### **Недоліки**

* прикладні сервіси часто потребують окремого collector job;
* велика кількість графіків спершу може перевантажувати;
* для командної роботи та централізованих нотифікацій зручніше підключати Cloud;
* на слабких VPS треба стежити за retention і дисковим простором;
* порт `19999` не можна залишати публічним без контролю доступу;
* це не повна observability-платформа для великого кластера.

## **Коли Netdata вже недостатньо**

Для одного VPS або кількох серверів цього інструменту зазвичай вистачає. Але якщо потрібно централізовано керувати десятками вузлів, будувати складні SLA-звіти, інтегрувати моніторинг із процесами NOC або детально контролювати інфраструктуру підприємства, варто дивитися в бік Zabbix. Якщо ж команда вже працює з Kubernetes, мікросервісами й time-series метриками, логічнішим вибором часто стає Prometheus + Grafana.

## **Що у підсумку?**

Це хороший варіант, коли потрібен швидкий моніторинг сервера без складної архітектури. За кілька хвилин можна отримати дані по серверу, побачити CPU, RAM, I/O pressure, диски, мережу, PHP-FPM і MariaDB, а потім поступово додати алерти та власні налаштування. Для невеликого проєкту це практична система моніторингу сервера, з якої можна починати моніторинг Linux-сервера без окремого стеку observability.

Цінність цього інструменту у простоті: встановив агент, відкрив панель, побачив графіки продуктивності. Для досвідчених адмінів користь у деталях: PSI, iowait, steal time, InnoDB, активні PHP-FPM connections, системні показники й швидка кореляція між подіями. Саме тому цей інструмент варто ставити одразу після запуску VPS.
