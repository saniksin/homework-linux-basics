# Домашнє завдання №3. Процеси, пріоритети та моніторинг ресурсів

## Завдання 1. Огляд активних процесів

**Вивести список усіх процесів у системі за допомогою `ps`:**

```bash
ps aux
```

Перші рядки виводу (повний список — 134 процеси, обрізано для зручності):

```
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0 166112 11384 ?        Ss   Apr26   0:02 /sbin/init
root           2  0.0  0.0   2776  1928 ?        Sl   Apr26   0:00 /init
root          64  0.0  0.0  64220 14992 ?        S<s  Apr26   0:02 /lib/systemd/systemd-journald
root          87  0.0  0.0  22340  6128 ?        Ss   Apr26   0:00 /lib/systemd/systemd-udevd
postgres     375  0.0 10.6 5143792 3378088 ?     Ss   Apr26   0:07 /usr/lib/postgresql/18/bin/postgres
saniksin    2627  2.1  2.2 54992972 701960 pts/2 Sl+  Apr26  17:47 node (vscode-server)
saniksin    3307 56.1  1.6 1190904 511208 pts/0  Sl+  Apr26 474:12 python -m amm_orchestrator.app.main
...
```

Кількість процесів у системі:

```bash
ps aux | wc -l
```

Вивід:

```
135
```

(135 рядків з заголовком — отже, 134 активні процеси.)

**Запустити інтерактивний монітор `top` і знайти процес, який споживає найбільше пам'яті (RAM):**

Перевіряємо наявність утиліт:

```bash
which top
which htop
```

Вивід:

```
/usr/bin/top
/usr/bin/htop
```

В інтерактивному `top` сортування за пам'яттю — клавіша `Shift+M`. Для звіту використано пакетний режим (`-b -n 1`) з сортуванням за стовпцем `%MEM`:

```bash
top -b -n 1 -o %MEM | head -15
```

Вивід:

```
top - 10:41:46 up 14:06,  1 user,  load average: 0.97, 0.97, 0.95
Tasks: 134 total,   1 running, 133 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.8 us,  0.0 sy,  0.0 ni, 99.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :  30898.0 total,  16960.4 free,   4933.1 used,   9004.5 buff/cache
MiB Swap:   8192.0 total,   8192.0 free,      0.0 used.  22100.5 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
    375 postgres  20   0 5143792   3.2g   3.2g S   0.0  10.7   0:07.50 postgres
   2627 saniksin  20   0   52.4g 702688  65660 S   0.0   2.2  17:47.53 node
 156891 saniksin  20   0   52.3g 613572  65596 S   0.0   1.9   0:09.42 node
   3307 saniksin  20   0 1189880 511768  26808 S   0.0   1.6 474:18.01 python
   3163 saniksin  20   0   22.4g 478080  55440 S   0.0   1.5   0:47.79 node
   1763 root      20   0 1471896 360688  10932 S   0.0   1.1   0:13.46 starknet+
 157826 saniksin  20   0   70.4g 358312 111332 S   6.2   1.1   0:10.69 claude
```

**Найбільший споживач RAM:** процес `postgres` з PID **375** — використовує **10.7 % пам'яті** (приблизно **3,2 GB RES**).

**Знайти PID поточної оболонки (bash/zsh):**

```bash
echo $SHELL
echo $$
ps -p $$ -o pid,comm
```

Вивід:

```
/bin/bash
158956
    PID COMMAND
 159006 bash
```

Поточна оболонка — `bash`, PID = **158956** (значення `$$` саме під час запуску).

---

## Завдання 2. Робота у фоні та керування процесами

**Запустити довгу команду у фоновому режимі:**

```bash
sleep 1000 &
```

Вивід:

```
[1] 159356
```

**Перевірити список фонових завдань:**

```bash
jobs
```

Вивід:

```
[1]+  Running                 sleep 1000 &
```

```bash
ps -p 159356 -o pid,stat,cmd
```

Вивід:

```
    PID STAT CMD
 159356 S    sleep 1000
```

**Повернути процес із фону на передній план:**

```bash
fg %1
```

Вивід (процес знов виконується у foreground):

```
sleep 1000
```

**Зупинити процес (`Ctrl+Z`) і «примусово» завершити його через `kill`:**

```
^Z
[1]+  Stopped                 sleep 1000
```

```bash
kill -SIGTERM %1
jobs
```

Вивід:

```
[1]+  Terminated              sleep 1000
```

```bash
ps -p 159356 -o pid,stat,cmd 2>/dev/null || echo "процес завершено"
```

Вивід:

```
процес завершено
```

**Запустити команду через `nohup`, щоб вона працювала після закриття терміналу:**

```bash
nohup sleep 300 > nohup_demo.out 2>&1 &
disown
```

Вивід:

```
[1] 159410
```

Перевіряємо, що процес стартував і відчіпився від терміналу:

```bash
ps -p 159410 -o pid,ppid,stat,cmd
```

Вивід:

```
    PID    PPID STAT CMD
 159410       1 S    sleep 300
```

`PPID=1` означає, що процес «пере-усиновлений» процесом `init` (`systemd`) — закриття терміналу його не вб'є. `nohup` ігнорує сигнал `SIGHUP`, а потоки `stdout/stderr` перенаправлені у файл `nohup_demo.out` (за замовчуванням — у `nohup.out`).

---

## Завдання 3. Пріоритети та обмеження

**Запустити нову команду з підвищеним значенням `nice` (нижчий пріоритет):**

`nice` приймає значення в діапазоні `-20` (найвищий пріоритет) ... `+19` (найнижчий). Звичайний користувач може лише знижувати пріоритет (значення `0…19`).

```bash
nice -n 10 bash -c 'ps -o pid,ni,cmd -p $$'
```

Вивід:

```
    PID  NI CMD
 159476  10 ps -o pid,ni,cmd -p 159476
```

Стовпець `NI = 10` підтверджує, що процес запущено з підвищеним «niceness» (нижчим пріоритетом).

**Змінити пріоритет уже запущеного процесу:**

```bash
sleep 200 &
ps -o pid,ni,cmd -p $!
```

Вивід:

```
[1] 159477
    PID  NI CMD
 159477   0 sleep 200
```

```bash
renice -n 5 -p 159477
ps -o pid,ni,cmd -p 159477
```

Вивід:

```
159477 (process ID) old priority 0, new priority 5
    PID  NI CMD
 159477   5 sleep 200
```

Пріоритет змінився з `0` на `5`.

**Переглянути поточні обмеження ресурсів користувача:**

```bash
ulimit -a
```

Вивід:

```
real-time non-blocking time  (microseconds, -R) unlimited
core file size              (blocks, -c) 0
data seg size               (kbytes, -d) unlimited
scheduling priority                 (-e) 0
file size                   (blocks, -f) unlimited
pending signals                     (-i) 123564
max locked memory           (kbytes, -l) 65536
max memory size             (kbytes, -m) unlimited
open files                          (-n) 1000000
pipe size                (512 bytes, -p) 8
POSIX message queues         (bytes, -q) 819200
real-time priority                  (-r) 0
stack size                  (kbytes, -s) 8192
cpu time                   (seconds, -t) unlimited
max user processes                  (-u) 123564
virtual memory              (kbytes, -v) unlimited
file locks                          (-x) unlimited
```

Серед ключових обмежень: `open files = 1 000 000`, `max user processes = 123 564`, `stack size = 8 МБ`, `core file size = 0` (створення core-dump-файлів вимкнене).

---

## Завдання 4. Моніторинг ресурсів

**Інформація про використання дискового простору:**

```bash
df -h
```

Вивід (основні файлові системи):

```
Filesystem      Size  Used Avail Use% Mounted on
none             16G     0   16G   0% /usr/lib/modules/5.15.167.4-microsoft-standard-WSL2
none             16G  4.0K   16G   1% /mnt/wsl
drivers         1.9T  1.1T  771G  59% /usr/lib/wsl/drivers
/dev/sdc       1007G  228G  729G  24% /
none             16G  212K   16G   1% /mnt/wslg
rootfs           16G  2.4M   16G   1% /init
none             16G  1.1M   16G   1% /run
tmpfs           4.0M     0  4.0M   0% /sys/fs/cgroup
C:\             1.9T  1.1T  771G  59% /mnt/c
```

Кореневу систему `/` змонтовано на пристрої `/dev/sdc`, розмір — **1007 GB**, використано **228 GB (24 %)**, вільно **729 GB**.

**Обсяг вільної та використаної оперативної пам'яті:**

```bash
free -h
```

Вивід:

```
               total        used        free      shared  buff/cache   available
Mem:            30Gi       4.8Gi        16Gi       3.3Gi       8.8Gi        21Gi
Swap:          8.0Gi          0B       8.0Gi
```

- Усього RAM: **30 GiB**
- Зайнято: **4.8 GiB**
- Вільно: **16 GiB**
- Кеш та буфери: **8.8 GiB**
- Доступно для нових процесів: **21 GiB**
- Swap: **8 GiB** (повністю вільний).
